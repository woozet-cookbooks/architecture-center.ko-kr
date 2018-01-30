---
title: "부적절한 인스턴스화 안티패턴"
description: "한 번 만든 다음 공유하는 개체의 새 인스턴스를 계속 만들지 마십시오."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 4b217f7fc644901eb5c3e77319d151caed30eef1
ms.sourcegitcommit: cf207fd10110f301f1e05f91eeb9f8dfca129164
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/29/2018
---
# <a name="improper-instantiation-antipattern"></a>부적절한 인스턴스화 안티패턴

한 번 만든 다음 공유하는 개체의 새 인스턴스를 계속 만들면 성능이 떨어질 수 있습니다. 

## <a name="problem-description"></a>문제 설명

많은 라이브러리가 외부 리소스의 추상화를 제공합니다. 내부적으로 이러한 클래스는 대개 클라이언트가 리소스에 액세스하는 데 사용할 수 있는 broker 역할을 하는 리소스에 대한 자체 연결을 관리합니다. 다음은 Azure 응용 프로그램과 관련된 broker 클래스의 몇 가지 예입니다.

- `System.Net.Http.HttpClient` HTTP를 사용하여 웹 서비스와 통신합니다.
- `Microsoft.ServiceBus.Messaging.QueueClient` Service Bus 큐에 메시지를 게시하고 수신합니다. 
- `Microsoft.Azure.Documents.Client.DocumentClient` Cosmos DB 인스턴스에 연결합니다.
- `StackExchange.Redis.ConnectionMultiplexer` Azure Redis Cache를 포함하여 Redis에 연결합니다.

이러한 클래스는 한 번 인스턴스화되면 응용 프로그램의 수명 내내 다시 사용됩니다. 그러나 이러한 클래스를 필요한 경우에만 확보하고 신속하게 릴리스해야 한다는 것은 일반적인 오해입니다. (여기 나열된 항목은 .NET 라이브러리이지만 패턴은 .NET 고유 패턴이 아닙니다.)

다음 ASP.NET 예제는 원격 서비스와 통신하기 위해 `HttpClient` 인스턴스를 만듭니다. 전체 샘플은 [여기][sample-app]에서 찾을 수 있습니다.

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

웹 응용 프로그램에서 이 기술은 확장성이 없습니다. 새 `HttpClient` 개체가 각 사용자 요청에 대해 만들어집니다. 과부하 시, 웹 서버에서 사용 가능한 소켓 수가 소진되어 `SocketException` 오류가 발생할 수 있습니다.

이 문제는 `HttpClient` 클래스에만 국한되지 않습니다. 리소스를 래핑하거나 만드는 비용이 높은 다른 클래스도 유사한 문제를 유발할 수 있습니다. 다음 예제는 `ExpensiveToCreateService` 클래스에 대한 인스턴스를 만듭니다. 여기서 문제는 소켓 소진이 아니라 각 인스턴스를 만드는 데 걸리는 시간입니다. 이 클래스의 인스턴스를 계속 만들고 제거하면 시스템의 확장성에 부정적인 영향을 미칠 수 있습니다.

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a>문제를 해결하는 방법

외부 리소스를 래핑하는 클래스가 공유가 가능하고 스레드로부터 안전한 경우 공유 singleton 인스턴스 또는 재사용 가능한 클래스 인스턴스 풀을 만듭니다.

다음 예제에는 정적 `HttpClient` 인스턴스를 사용하므로 모든 요청에서 연결을 공유합니다.

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient HttpClient;

    static SingleHttpClientInstanceController()
    {
        HttpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await HttpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a>고려 사항

- 이 안티패턴의 핵심 요소는 *공유가 가능한* 개체 인스턴스를 반복적으로 만들고 제거하는 것입니다. 클래스를 공유할 수 없는(스레드로부터 안전하지 않은) 경우에는 이 안티패턴이 적용되지 않습니다.

- 공유 리소스의 유형에 따라 singleton을 사용할지 풀을 만들지 여부가 결정될 수 있습니다. `HttpClient` 클래스는 풀링되지 않고 공유되도록 설계되었습니다. 다른 개체는 풀링을 지원하기 때문에 시스템이 워크로드를 여러 인스턴스로 분산할 수 있습니다.

- 여러 요청에서 공유하는 개체는 *반드시* 스레드로부터 안전해야 합니다. `HttpClient` 클래스는 이러한 방식으로 사용하도록 설계되었지만 다른 클래스는 동시 요청을 지원하지 않을 수 있으니 사용 가능한 설명서를 확인하십시오.

- 일부 리소스는 희박하며 보유하지 말아야 합니다. 데이터베이스 연결이 그 예입니다. 필요하지 않은 열린 데이터베이스 연결을 보유하면 다른 동시 사용자가 데이터베이스에 액세스하는 것을 막을 수 있습니다.

- .NET Framework에서는 외부 리소스에 대한 연결을 설정하는 많은 개체가 이러한 연결을 관리하는 다른 클래스의 정적 팩터리 메서드를 사용하여 만들어집니다. 이런 팩터리 개체는 삭제되고 다시 만들어지기 보다 저장되고 다시 사용됩니다. 예를 들어 Azure Service Bus에서 `QueueClient` 개체는 `MessagingFactory` 개체를 통해 만들어집니다. 내부적으로 `MessagingFactory`가 연결을 관리합니다. 자세한 내용은 [Service Bus 메시징을 사용한 성능 향상의 모범 사례][service-bus-messaging]를 참조하세요.

## <a name="how-to-detect-the-problem"></a>문제를 감지하는 방법

이 문제의 증상에는 처리량이 떨어지거나 오류 비율이 증가하는 경우와 함께 다음 중 하나 이상의 경우가 동반됩니다. 

- 소켓, 데이터베이스 연결, 파일 핸들 등과 같은 리소스가 소진된 것을 나타내는 예외가 증가합니다. 
- 메모리 사용 및 가비지 수집이 증가합니다.
- 네트워크, 디스크 또는 데이터베이스 활동이 증가합니다.

다음 단계를 수행하면 문제를 식별하는 데 도움이 될 수 있습니다.

1. 프로덕션 시스템의 프로세스를 모니터링하여 응답 시간이 느려지거나 리소스 부족으로 인해 시스템에 장애가 발생하는 지점을 파악합니다.
2. 이 지점에서 캡처된 원격 분석 데이터를 검사하여 어떤 작업이 리소스를 많이 소비하는 개체를 만들고 제거하는지 파악합니다.
3. 프로덕션 시스템이 아닌 통제된 테스트 환경에서 의심되는 각 작업의 부하를 테스트합니다.
4. 소스 코드를 검토하고 broker 개체가 관리되는 방식을 검사합니다.

시스템에 부하가 걸렸을 때 느리게 실행되거나 예외를 생성하는 작업에 대한 스택 추적을 살펴봅니다. 이 정보는 해당 작업이 리소스를 활용하는 방법을 파악하는 데 도움이 될 수 있습니다. 예외는 오류가 공유 자원이 소진되어 발생했는지를 파악하는 데 도움이 됩니다. 

## <a name="example-diagnosis"></a>예제 진단

다음 섹션에서는 이러한 단계를 앞에서 설명한 응용 프로그램 예제에 적용합니다.

### <a name="identify-points-of-slow-down-or-failure"></a>느려지거나 실패한 지점 식별

다음 이미지는 [New Relic APM][new-relic]을 사용하여 생성된 결과이며, 응답 시간이 열악한 작업을 보여줍니다. 이 경우 `NewHttpClientInstancePerRequest` 컨트롤러의 `GetProductAsync` 메서드는 보다 자세히 조사하는 것이 좋습니다. 이러한 작업이 실행 중일 때 오류 비율도 증가합니다. 

![각 요청에 대해 HttpClient 개체의 새 인스턴스를 만드는 응용 프로그램 예제를 보여주는 New Relic 모니터링 대시 보드][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a>원격 분석 데이터 검사 및 상관 관계 찾기

다음 이미지는 이전 이미지와 동일한 기간에 스레드 프로파일링을 사용하여 캡처한 데이터입니다. 시스템은 소켓 연결을 여는 데 많은 시간을 소비하며 소켓 연결을 닫고 소켓 예외를 처리하는 데 더 많은 시간을 소비합니다.

![각 요청에 대해 HttpClient 개체의 새 인스턴스를 만드는 응용 프로그램 예제를 보여주는 New Relic 스레드 프로파일러][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a>부하 테스트 수행

부하 테스트를 사용하여 사용자가 수행할만한 일반적인 작업을 시뮬레이션합니다. 이렇게 하면 다양한 부하에서 리소스 소진으로 인해 시스템의 어느 부분이 저하되는지 식별하는 데 도움이 될 수 있습니다. 이러한 테스트는 프로덕션 시스템보다는 통제된 환경에서 수행합니다. 다음 그래프는 사용자 로드가 동시 사용자 100명으로 증가하면서 `NewHttpClientInstancePerRequest` 컨트롤러가 처리하는 요청 처리량을 보여줍니다.

![각 요청에 대해 HttpClient 개체의 새 인스턴스를 만드는 응용 프로그램 예제의 처리량][throughput-new-HTTPClient-instance]

우선 워크로드가 증가하면서 초당 처리되는 요청의 볼륨이 증가합니다. 하지만 사용자가 약 30명이 되면 성공한 요청의 볼륨이 한도에 도달하고 시스템은 예외를 생성하기 시작합니다. 그 이후로 예외 볼륨이 사용자 로드에 따라 점차 증가합니다. 

부하 테스트는 이러한 장애를 HTTP 500(내부 서버) 오류로 보고했습니다. 원격 분석 데이터를 검토하면 `HttpClient` 개체가 점점 더 많이 만들어지면서 시스템에 소켓 리소스가 소진되어 이러한 오류가 발생했다는 것을 알 수 있습니다.

다음 그래프는 사용자 지정 `ExpensiveToCreateService` 개체를 생성하는 컨트롤러에 대한 유사한 테스트를 보여줍니다.

![각 요청에 대해 ExpensiveToCreateService의 새 인스턴스를 만드는 응용 프로그램 예제의 처리량][throughput-new-ExpensiveToCreateService-instance]

이번에는 컨트롤러가 예외를 생성하지 않고 처리량이 안정기에 접어든 반면 평균 응답 시간은 20배 증가합니다. (이 그래프는 응답 시간과 처리량에 대해 로그 눈금 간격을 사용합니다.) 원격 분석 데이터에 따르면 `ExpensiveToCreateService`의 새 인스턴스를 만드는 것이 문제의 주요 원인입니다.

### <a name="implement-the-solution-and-verify-the-result"></a>솔루션 구현 및 결과 확인

`GetProductAsync` 메서드를 전환하여 단일 `HttpClient` 인스턴스를 공유한 후 2번째 부하 테스트에서 성능이 향상되었습니다. 오류는 보고되지 않았고 시스템은 초당 최대 500건의 요청에 달하는 증가한 부하를 처리할 수 있었습니다. 평균 응답 시간은 이전 테스트와 비교하여 절반이 되었습니다.

![각 요청에 대해 HttpClient 개체의 동일한 인스턴스를 다시 사용하는 응용 프로그램 예제의 처리량][throughput-single-HTTPClient-instance]

비교를 위해 다음 이미지는 스택 추적 원격 분석 데이터를 보여줍니다. 이번에는 시스템이 소켓을 열고 닫는 대신 실제 작업을 수행하는 데 대부분의 시간을 소비합니다.

![모든 요청에 대해 HttpClient 개체의 단일 인스턴스를 만드는 응용 프로그램 예제를 보여주는 New Relic 스레드 프로파일러][thread-profiler-single-HTTPClient-instance]

다음 그래프는 `ExpensiveToCreateService` 개체의 공유 인스턴스를 사용하는 유사한 부하 테스트를 보여줍니다. 처리된 요청의 볼륨은 사용자 로드에 따라 증가하지만 평균 응답 시간은 낮게 유지됩니다. 

![각 요청에 대해 HttpClient 개체의 동일한 인스턴스를 다시 사용하는 응용 프로그램 예제의 처리량][throughput-single-ExpensiveToCreateService-instance]



[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
