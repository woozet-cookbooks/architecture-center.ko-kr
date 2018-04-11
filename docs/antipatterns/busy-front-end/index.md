---
title: 붐비는 프런트 엔드 안태패턴
description: 다수의 백그라운드 스레드에서 비동기 작업을 수행하면 다른 포그라운드 작업에 결핍이 발생할 수 있습니다.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: cedb80ddac5ceb1eb901455df3165993fd28a138
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="busy-front-end-antipattern"></a>붐비는 프런트 엔드 안태패턴

다수의 백그라운드 스레드에서 비동기 작업을 수행하면 리소스의 다른 동시 포그라운드 작업에 결핍이 발생하여 응답 시간이 허용할 수 없는 수준으로 감소됩니다.

## <a name="problem-description"></a>문제 설명

리소스를 많이 사용하는 작업은 사용자 요청에 대한 응답 시간을 늘리고 긴 대기 시간을 유발할 수 있습니다. 응답 시간을 향상시키는 한 가지 방법은 리소스를 많이 사용하는 작업을 별도의 스레드에 오프로드하는 것입니다. 이 방법으로 백그라운드에서 처리가 진행되는 동안 응용 프로그램의 응답 속도를 유지할 수 있습니다. 하지만 백그라운드 스레드에서 실행되는 작업은 리소스를 여전히 소비합니다. 이 수가 너무 많으면 요청을 처리하는 스레드에 결핍이 발생할 수 있습니다.

> [!NOTE]
> *리소스*라는 용어는 CPU 사용률, 메모리 점유율 및 네트워크나 디스크 I/O와 같은 많은 요소를 포함합니다.

일반적으로 이런 문제는 응용 프로그램이 모놀리식 코드 조각으로 개발되고 모든 비즈니스 논리가 프레젠테이션 계층과 공유되는 단일 계층으로 결합되는 경우에 발생합니다.

다음은 문제를 보여주는 ASP.NET을 사용한 예제입니다. 전체 샘플은 [여기][code-sample]에서 찾을 수 있습니다.

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- `WorkInFrontEnd` 컨트롤러의 `Post` 메서드는 HTTP POST 작업을 구현합니다. 이 작업은 CPU를 많이 사용하는 장기 실행 작업을 시뮬레이션합니다. 이 작업은 POST 작업을 신속하게 완료할 수 있도록 별도의 스레드에서 수행됩니다.

- `UserProfile` 컨트롤러의 `Get` 메서드는 HTTP GET 작업을 구현합니다. 이 메서드는 훨씬 적은 CPU를 사용합니다.

주요 사안은 `Post` 메서드의 리소스 요구 사항입니다. 작업이 백그라운드 스레드에 배치되더라도 작업은 여전히 상당한 CPU 리소스를 소비합니다. 리소스는 다른 동시 사용자가 수행하는 다른 작업과 공유됩니다. 적당한 수의 사용자가 이 요청을 동시에 보내면 전반적인 성능이 저하되어 모든 작업이 느려질 수 있습니다. 예를 들면, `Get` 메서드에서 상당한 대기 시간이 발생할 수 있습니다.

## <a name="how-to-fix-the-problem"></a>문제를 해결하는 방법

상당한 리소스를 소비하는 프로세스를 별도의 백엔드로 이동합니다. 

이러한 방식을 통해 프런트 엔드는 리소스를 많이 사용하는 작업을 메시지 큐에 넣습니다. 백 엔드는 비동기 처리를 위한 작업을 선택합니다. 큐는 로드 레벨러(leveler)로 작동하여 백 엔드에 대한 요청을 버퍼링합니다. 큐 길이가 너무 길어지면 자동 크기 조정을 구성하여 백 엔드 규모를 확장할 수 있습니다.

다음은 이전 코드의 수정된 버전입니다. 이 버전에서 `Post` 메서드는 메시지를 Service Bus 큐에 배치합니다. 

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

백 엔드는 Service Bus 큐에서 메시지를 끌어와서 처리합니다.

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a>고려 사항

- 이 방식은 응용 프로그램에 몇 가지 복잡성을 더합니다. 오류 발생시 요청이 손실되지 않도록 큐 생성 및 큐 해제를 안전하게 처리해야 합니다.
- 응용 프로그램은 메시지 큐에 대한 추가 서비스에 종속됩니다.
- 처리 환경은 예상 워크로드를 처리하고 필요한 처리량 목표를 충족할 만큼 충분히 확장성이 있어야 합니다.
- 이 방식은 전반적인 응답성을 개선하는 하지만 백 엔드로 이동되는 작업은 완료하는 데 시간이 더 오래 걸릴 수 있습니다. 

## <a name="how-to-detect-the-problem"></a>문제를 감지하는 방법

붐비는 프런트 엔드의 증상에는 리소스를 많이 사용하는 작업을 수행할 때 대기 시간이 길어지는 것이 포함됩니다. 응답 시간이 길어지거나 서비스 시간 초과로 인해 장애가 발생하면 최종 사용자가 보고할 가능성이 높습니다. 이러한 장애는 HTTP 500(내부 서버) 오류 또는 HTTP 503(서비스를 사용할 수 없음) 오류를 반환할 수도 있습니다. 웹 서버의 이벤트 로그를 검사하십시오. 오류의 원인과 상황에 대한 자세한 정보가 포함되어 있을 수 있습니다.

다음 단계를 수행하면 문제를 식별하는 데 도움이 될 수 있습니다.

1. 프로덕션 시스템의 프로세스 모니터링을 수행하여 응답 시간이 느려지는 시점을 파악합니다.
2. 이 시점에 캡처된 원격 분석 데이터를 검사하여 수행되는 작업과 사용되는 리소스의 혼합을 결정합니다. 
3. 당시 수행되었던 작업의 볼륨 및 조합과 열악한 응답 시간 사이의 상관 관계를 찾습니다.
4. 의심되는 각 작업의 부하를 테스트하여 어떤 작업이 리소스를 소비하면서 다른 작업에 결핍을 유발하는지 파악합니다. 
5. 이러한 작업의 소스 코드를 검토하여 과도한 리소스 소비를 유발하는 원인을 알아냅니다.

## <a name="example-diagnosis"></a>예제 진단 

다음 섹션에서는 이러한 단계를 앞에서 설명한 응용 프로그램 예제에 적용합니다.

### <a name="identify-points-of-slowdown"></a>느려지는 지점 파악

각 요청이 소비하는 기간 및 자원을 추적하기 위해 각 메서드를 계측합니다. 그런 다음 프로덕션 환경에서 응용 프로그램을 모니터링합니다. 요청이 서로 경쟁하는 방식을 전반적으로 볼 수 있습니다. 스트레스 상황에서 느리게 실행되는 리소스가 부족한 요청은 다른 작업에 영향을 미칠 수 있으며 이러한 동작은 시스템을 모니터링하고 성능 저하에 주목함으로써 관찰할 수 있습니다.

다음 이미지는 모니터링 대시보드입니다. (테스트에는 [AppDynamics]가 사용되었습니다.) 초기에는 시스템 부하가 적습니다. 그런 다음 사용자가 `UserProfile` GET 메서드를 요청하기 시작합니다. 성능은 다른 사용자가 `WorkInFrontEnd` POST 메서드에 대한 요청 발급을 시작할 때가지 상당히 좋습니다. 이 시점에서 응답 시간이 크게 증가합니다(첫 번째 화살표). 응답 시간은 `WorkInFrontEnd` 컨트롤러에 대한 요청 볼륨이 줄어든 후에야 증가합니다(두 번째 화살표).

![WorkInFrontEnd 컨트롤러가 사용될 때 모든 요청의 응답 시간 효과를 보여주는 AppDynamics Business Transactions 창][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a>원격 분석 데이터 검사 및 상관 관계 찾기

다음 이미지는 동일한 간격으로 리소스 사용률을 모니터링하기 위해 모은 일부 메트릭입니다. 처음에는 시스템에 액세스하는 사용자가 거의 없습니다. 연결하는 사용자가 많아지면서 CPU 사용률이 매우 높아집니다(100 %). 또한 CPU 사용량이 증가함에 따라 네트워크 I/O 속도가 처음에는 올라갑니다. 하지만 CPU 사용량이 최고가 되면 네트워크 I/O가 실제로 내려갑니다. CPU가 용량에 도달하면 시스템이 비교적 적은 수의 요청만 처리할 수 있기 때문입니다. 사용자가 연결을 끊으면 CPU 로드가 끝납니다.

![CPU 및 네트워크 사용률을 보여주는 AppDynamics 메트릭][AppDynamics-Metrics-Front-End-Requests]

이 시점에서는 `WorkInFrontEnd` 컨트롤러의 `Post` 메서드가 정밀 조사의 주요 후보입니다. 가설을 확인하려면 통제된 환경에서 추가 작업이 필요합니다.

### <a name="perform-load-testing"></a>부하 테스트 수행 

다음 단계는 통제된 환경에서 테스트를 수행하는 것입니다. 예를 들어, 요청을 포함하는 일련의 부하 테스트를 실행한 다음 각 요청을 차례로 생략하면서 영향을 확인합니다.

아래 그래프는 이전 테스트에 사용한 클라우드 서비스의 동일한 배치에 수행된 부하 테스트의 결과입니다. 이 테스트에는 `WorkInFrontEnd` 컨트롤러에서 `Post` 작업을 수행하는 사용자의 단계 부하와 함께 `UserProfile` 컨트롤러에서 `Get` 작업을 수행하는 500명의 일정 부하를 사용했습니다. 

![WorkInFrontEnd 컨트롤러의 초기 부하 테스트 결과][Initial-Load-Test-Results-Front-End]

처음에는 단계 로드가 0이므로 활성 사용자만 `UserProfile` 요청을 수행합니다. 시스템은 초당 약 500개 요청에 응답할 수 있습니다. 60초 후, 100명의 추가 사용자가 `WorkInFrontEnd` 컨트롤러에 POST 요청을 보내기 시작합니다. 거의 즉시, `UserProfile` 컨트롤러에 전송된 워크로드가 초당 약 150개 요청으로 떨어집니다. 이것은 부하 테스트 러너(test runner)의 작동 방식 때문입니다. 다음 요청을 보내기 전에 응답을 기다리기 때문에 응답을 받는 시간이 오래 걸릴수록 요청 속도가 느려집니다.

`WorkInFrontEnd` 컨트롤러에 POST 요청을 보내는 사용자가 많아질수록 `UserProfile` 컨트롤러의 응답 속도는 계속 떨어집니다. 하지만 `WorkInFrontEnd` 컨트롤러가 처리하는 요청의 볼륨은 비교적 일정하게 유지됩니다. 두 요청의 전반적인 속도가 꾸준히 낮은 한도로 향하기 때문에 시스템의 포화 상태가 명백해집니다.


### <a name="review-the-source-code"></a>소스 코드 검토

마지막 단계에서는 소스 코드를 확인합니다. `Post` 메서드에 상당한 시간이 걸릴 수 있다는 것을 개발 팀이 알고 있었기 때문에 원래 구현에 별도의 스레드가 사용되었습니다. 이로 인해 `Post` 메서드가 장기 실행 작업이 완료되기를 기다리는 것을 차단하지 않았기 때문에 즉각적인 문제는 해결되었습니다.

하지만 이 메서드로 수행되는 작업은 여전히 CPU, 메모리 및 기타 리소스를 소비합니다. 이 프로세스를 비동기적으로 실행하면 통제되지 않은 방식으로 사용자가 동시에 많은 수의 작업을 트리거할 수 있기 때문에 실제로 성능이 저하될 수 있습니다. 서버가 실행할 수 있는 스레드 수에는 한도가 있습니다. 이 한도를 넘으면 새 스레드를 시작하려고 할 때 응용 프로그램에 예외가 발생할 수 있습니다.

> [!NOTE]
> 이것이 비동기 작업을 피해야 한다는 의미는 아닙니다. 권장되는 방법은 네트워크 호출에 비동기 await를 수행하는 것입니다. ([동기 I/O][sync-io] 안태패턴을 참조하세요.) 여기서 문제는 다른 스레드에서 CPU를 많이 사용하는 작업이 생성된다는 점입니다. 

### <a name="implement-the-solution-and-verify-the-result"></a>솔루션 구현 및 결과 확인

다음 이미지는 솔루션을 구현한 후 성능 모니터링입니다. 로드는 앞에 표시된 것과 비슷하지만 `UserProfile` 컨트롤러의 응답 시간이 훨씬 빨라졌습니다. 요청 볼륨은 같은 기간 동안 2,759에서 23,565로 증가했습니다. 

![WorkInBackground 컨트롤러가 사용될 때 모든 요청의 응답 시간 효과를 보여주는 AppDynamics Business Transactions 창][AppDynamics-Transactions-Background-Requests]

`WorkInBackground` 컨트롤러 역시 훨씬 더 많은 요청 볼륨을 처리했습니다. 하지만 이 경우 컨트롤러에서 수행되는 작업은 원래 코드와 매우 다르기 때문에 직접 비교할 수는 없습니다. 새 버전에서는 시간이 오래 걸리는 계산을 수행하는 것이 아니라 요청을 큐에 넣기만 합니다. 관건은 이 메서드가 전체 시스템을 부하 상태로 끌어 내리지 않는다는 점입니다.

CPU 및 네트워크 사용률 역시 향상된 성능을 보여줍니다. CPU 사용률이 100%에 도달하지 않았고 처리된 네트워크 요청의 볼륨이 이전보다 훨씬 커졌으며 작업 부하가 떨어질 때까지 감소되지 않았습니다.

![WorkInBackground 컨트롤러의 CPU 및 네트워크 사용률을 보여주는 AppDynamics 메트릭][AppDynamics-Metrics-Background-Requests]

다음 그래프는 부하 테스트의 결과입니다. 이전 테스트와 비교하여 처리된 요청의 전체적인 양이 크게 향상되었습니다.

![BackgroundImageProcessing 컨트롤러의 부하 테스트 결과][Load-Test-Results-Background]

## <a name="related-guidance"></a>관련 지침

- [자동 크기 조정 모범 사례][autoscaling]
- [백그라운드 작업 모범 사례][background-jobs]
- [큐 기반 부하 평준화 패턴][load-leveling]
- [웹 큐 작업자 아키텍처 스타일][web-queue-worker]

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg


