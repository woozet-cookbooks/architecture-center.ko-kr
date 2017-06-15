---
title: Compute Resource Consolidation
description: Consolidate multiple tasks or operations into a single computational unit
keywords: design pattern
author: dragon119
ms.service: guidance
ms.topic: article
ms.author: pnp
ms.date: 03/24/2017

pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: [design-implementation]
---

# 계산 리소스 통합

[!INCLUDE [header](../_includes/header.md)]

여러 작업 또는 연산을 단일 계산 단위에 통합합니다. 따라서 계산 리소스 사용률을 높이고 클라우드 호스팅 응용 프로그램에서 계산 처리 수행에 관련된 비용과 관리 오버헤드를 줄일 수 있습니다.

## 배경 및 문제

클라우드 응용 프로그램은 다양한 작업을 구현하는 경우가 많습니다. 일부 솔루션에서는 처음부터 문제를 분리하는 설계 원리를 따르기도 하고 이런 작업을 개별적으로 호스팅하고 배포하는 별도의 계산 단위(예: 별도의 App Service 웹 앱, 별도의 가상 컴퓨터, 별도의 클라우드 서비스 역할)로 분할하기도 합니다. 그러나 이런 전략이 솔루션의 논리적 설계 단순화에 기여하더라도 동일한 응용 프로그램의 일부로 많은 계산 단위를 배포하면 런타임 호스팅 비용이 증가하고 시스템의 관리가 더 복잡해질 수 있습니다.

다음 그림에 제시된 사례는 하나 이상의 계산 단위를 사용해 구현한 클라우드 호스팅 솔루션의 단순화된 구조를 보여줍니다. 각 계산 단위는 자체 가상 환경에서 실행됩니다. 각 기능은 자체 계산 단위에서 실행되는 별도의 작업(작업 A~작업 E)으로 구현되었습니다.

![Running tasks in a cloud environment using a set of dedicated computational units](./_images/compute-resource-consolidation-diagram.png)


각 계산 단위는 사용하지 않거나 사용 시간이 적더라도 비용이 부과되는 리소스를 소모합니다. 따라서 이 솔루션이 항상 가장 경제적인 것은 아닙니다. 

Azure에서 이런 문제는 자체 가상 환경에서 실행되는 클라우드 서비스, App Service, 가상 컴퓨터에서의 역할에 적용됩니다. 잘 정의된 작업의 집합을 수행하도록 설계되었지만 단일 솔루션의 일부로 통신하고 협력할 필요가 있는 분리된 역할, 웹 사이트 또는 가상 컴퓨터의 모음을 실행하면 리소스 사용이 비효율적일 수 있습니다.

## 해결책

여러 작업 또는 연산을 하나의 계산 단위에 통합하면 비용을 줄이고, 사용률을 높이며, 통신 속도를 개선하고, 관리를 줄일 수 있습니다.

작업은 환경의 특징과 이런 특징에 관련된 비용을 기준으로 그룹으로 묶을 수 있습니다. 일반적인 접근 방식은 확장성, 수명, 처리 요구 사항이 비슷한 프로필을 갖는 작업을 찾는 것입니다. 이런 작업을 하나의 그룹으로 함께 묶으면 하나의 단위로 크기를 조정할 수 있습니다. 많은 클라우드 환경에서 제공하는 탄력성은 계산 단위의 추가 인스턴스를 워크로드에 따라 시작 및 중단할 수 있도록 지원합니다. 예를 들어 Azure는 클라우드 서비스, App Service, 가상 컴퓨터에서의 역할에 적용할 수 있는 자동 크기 조정을 제공합니다. 자세한 내용은 [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx) 을 참조하시기 바랍니다.

다음 두 가지 상반되는 사례는 확장성을 그룹으로 함께 묶어서는 안 되는 작업을 결정하는데 사용하는 방법을 보여줍니다.

- 작업 1은 큐에 전송되는, 자주 사용되지 않고 시간에 민감하지 않은 메시지를 폴링합니다.
- 작업 2는 네트워크 트래픽의 대규모 버스트를 처리합니다.

두 번째 작업에는 계산 단위의 수 많은 인스턴스를 시작 및 중단할 수 있는 탄력성이 필요합니다. 한편 이와 동일한 크기 조정을 첫 번째 작업에 적용하면 동일한 큐에서 자주 사용되지 않는 메시지를 수신하는 작업이 더 많아져 리소스를 낭비하게 됩니다.

많은 클라우드 환경에서는 CPU 코어 개수, 메모리, 디스크 공간 등과 같은 계산 단위에 사용할 수 있는 리소스를 지정할 수 있는데, 일반적으로 지정하는 리소스의 수가 늘어날수록 비용이 높아집니다. 비용을 절감하려면 비싼 계산 단위가 수행하는 작업의 양을 최대화하고 사용되지 않는 상태로 장시간 있지 않도록 조치하는 것이 중요합니다.

짧은 버스트에서 많은 CPU 성능을 요구하는 작업들은 필요한 CPU 성능을 제공하는 단일 계산 단위에 통합할 것을 고려해야 합니다. 그러나 중요한 것은 이와 같은 요구의 균형을 유지해 스트레스를 과도하게 받을 경우 발생할 수 있는 경합을 방지하는 동시에 비싼 리소스를 바쁜 상태로 유지하는 것입니다. 예를 들어 장기 실행되는 계산 집약적 작업은 동일한 계산 단위를 공유하지 않아야 합니다.

## 문제 및 고려 사항

이 패턴의 구현할 때는 다음 사항을 고려해야 합니다.

**확장성 및 탄력성**. 많은 클라우드 솔루션은 계산 단위의 인스턴스를 시작 및 중단해 계산 단위 수준에서 확장성과 탄력성을 구현합니다. 동일한 계산 단위에서 확장성 요구 사항이 충돌하는 작업을 그룹으로 묶어서는 안 됩니다.

**수명**. 클라우드 인프라는 계산 단위를 호스팅하는 가상 환경을 주기적으로 재활용합니다. 따라서 계산 단위 내에 장기 실행되는 작업이 많이 있으면 해당 작업이 완료될 때까지 계산 단위를 재활용하지 않도록 구성할 필요가 있습니다. 또는 체크 포인팅 접근을 이용해 작업을 완전히 중단하고 중단된 작업을 다시 시작할 때는 중단된 지점에서 시작할 수 있도록 설계해야 합니다.

**릴리스 주기**. 작업의 구현 또는 구성을 자주 변경하는 경우, 업데이트된 코드를 호스팅하는 계산 단위를 중단하고, 다시 구성하여 다시 배포한 다음, 다시 시작할 필요가 있습니다. 이런 프로세스를 활용하면 동일한 계산 단위 내의 모든 다른 작업도 중단하고, 다시 배포한 다음, 다시 시작할 필요가 있습니다.

**보안**. 동일한 계산 단위 내의 작업은 동일한 보안 컨텍스트를 공유하고 동일한 리소스에 액세스할 수 있습니다.  작업 사이에는 신뢰도가 높아야 하고 하나의 작업이 다른 작업에 손상을 입히거나 악영향을 미치지 않는다는 확신이 있어야 합니다. 한편 계산 단위에서 실행되는 작업의 숫자가 늘어나면 공격 대상이 되는 계산 단위의 범위가 넓어집니다. 보안 관점에서 각각의 작업은 가장 취약한 작업이 됩니다.

**내결함성**. 계산 단위 내의 한 작업이 실패하거나 비정상적으로 동작하면 동일한 계산 단위 내에서 실행되는 다른 작업에 영향을 미칠 수 있습니다. 예를 들어 올바르게 시작하는 데 실패한 작업은 전체 계산 단위의 시작 논리 실패를 초래하여 동일한 계산 단위 내의 다른 작업 실행에 지장을 줄 수 있습니다.

**경합**. 동일한 계산 단위에서 리소스를 두고 경쟁하는 작업들이 경합하지 않도록 주의해야 합니다. 동일한 계산 단위를 공유하는 작업들이 서로 다른 리소스 활용 특성을 나타내는 것이 가장 이상적입니다. 예를 들어 2개의 계산 집약적 작업이 동일한 계산 단위 내에 있으면 안 되고 메모리를 많이 사용하는 2개의 작업도 동일한 계산 단위에 있으면 안 됩니다. 그러나 계산 집약적 작업과 메모리가 많이 필요한 작업은 혼합할 수 있습니다. 

> [!참고]
>  계산 리소스 통합은 일정 기간 동안 프로덕션에 존재해온 시스템에서만 고려해야 합니다. 그럼으로써 운영자와 개발자는 시스템을 모니터링하고 각 작업이 다양한 리소스를 어떻게 사용하는지를 식별하는 _열 지도_ 를 생성할 수 있습니다. 이런 열 지도는 계산 리소스를 공유하기 적합한 작업을 결정하는 데 사용할 수 있습니다.

**복잡성**. 단일 계산 단위에 여러 작업을 조합하면 계산 단위 코드에 복잡성이 추가되어 테스트, 디버그 및 유지가 더 어려워질 수 있습니다.

**안정된 논리 아키텍처**. 작업의 코드를 각각 설계하고 구현하여 작업을 실행하는 물리적 환경이 변하더라도 작업을 변경할 필요가 없도록 조치합니다.

**기타 전략**. 여러 작업의 동시 실행에 관련된 비용을 줄이는 유일한 방법인 계산 리소스 통합이 효과적인 접근 방식으로 유지되려면 신중한 계획 수립과 모니터링이 필요합니다. 작업의 특성 및 작업을 실행하는 사용자가 있는 장소에 따라 다른 전략이 더 적절할 수 있습니다. 예를 들어 ([계산 분할 지침](https://msdn.microsoft.com/library/dn589773.aspx) 에서 설명하는) 워크로드의 기능적 분해가 더 나은 옵션일 수 있습니다.

## 패턴 사용 사례

이 패턴은 자체 계산 단위에서 실행할 경우 경제적이지 않은 작업에 사용합니다. 거의 대부분의 시간 동안 사용되지 않는 작업을 전용 단위에서 실행하면 비용이 많이 소요될 수 있습니다.

이런 패턴은 중요한 내결함성 작업을 수행하는 작업 또는 매우 민감하거나 개인적인 데이터를 처리하고 자체 보안 컨텍스트가 필요한 작업에는 적합하지 않습니다. 이런 작업은 분리된 환경인 별도의 계산 단위에서 실행해야 합니다.

## 예제

Azure에서 클라우드 서비스를 구축할 때는 여러 작업이 수행하는 처리를 단일 역할에 통합할 수 있습니다. 일반적으로 이것은 백그라운드 또는 비동기 처리 작업을 수행하는 작업자 역할입니다.

> 일부 경우 백그라운드 또는 비동기 처리 작업을 웹 역할에 포함시킬 수 있습니다. 이런 기법이 웹 역할이 제공하는 공용 인터페이스의 확장성과 응답성에 영향을 미칠 수 있지만, 비용을 줄이고 배포를 단순화하는 데에는 도움이 됩니다. [여러 Azure 작업자 역할을 Azure 웹 역할에 조합](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html) 은 웹 역할에서 백그라운드 또는 비동기 처리 작업 구현에 관해 자세하게 설명하고 있습니다.

이 역할은 작업의 시작 및 중단을 책임집니다. Azure 패브릭 컨트롤러가 역할을 로드하면 역할에 대한 `Start` 이벤트가 발생합니다. 사용자는 `WebRole` 또는 `WorkerRole` 클래스의 `OnStart` 메서드를 재정의해 이런 이벤트를 처리하고 OnStart 메서드가 의존하는 작업에 필요한 데이터와 다른 리소스를 시작합니다.

`OnStart ` 메서드가 완료되면 역할은 요청에 대한 응답을 시작할 수 있습니다. 역할에서 `OnStart` 및 `Run` 메서드의 사용에 대한 자세한 내용과 지침은 패턴 및 사례 가이드인 [응용 프로그램에서 클라우드로 이동](https://msdn.microsoft.com/library/ff728592.aspx) 의 [응용 프로그램 시작 프로세스](https://msdn.microsoft.com/library/ff803371.aspx#sec16) 섹션에서 확인할 수 있습니다.

> `OnStart` 메서드의 코드는 최대한 간결하게 유지합니다. Azure는 이 메서드를 완료하는 데 소요되는 시간에 제한을 두지 않지만, 이 메서드가 완료되기 전까지는 역할이 전송된 네트워크 요청에 대한 응답을 시작할 수 없습니다.

`OnStart` 메서드가 완료되면 역할은 `Run` 메서드를 실행합니다. 이때 패브릭 컨트롤러는 역할에 요청의 전송을 시작할 수 있습니다.

실제 작업을 생성하는 코드를 `Run` 메서드에 삽입합니다. `Run` 메서드는 역할 인스턴스의 수명을 정의합니다. 이 메서드가 완료되면 패브릭 컨트롤러는 종료할 역할을 정렬합니다.

역할이 종료되거나 재활용되면 패브릭 컨트롤러는 부하 분산 장치에서 추가로 들어오는 요청의 수신을 차단하고 `Stop` 이벤트를 발생시킵니다. 사용자는 역할의 `OnStop` 메서드를 재정의해 이런 이벤트를 캡처하고 역할을 종료하기 전에 필요한 정리를 수행할 수 있습니다.

> `OnStop` 메서드 내에서 수행되는 모든 동작은 5분 (또는 로컬 컴퓨터에서 Azure 에뮬레이터를 사용하는 경우는 30초) 이내에 완료되어야 합니다. 그렇지 않으면 Azure 패브릭 컨트롤러는 역할이 멈추었다고 판단해 역할을 강제로 중단합니다.

작업은 작업이 완료될 때까지 기다리는 `Run` 메서드를 통해 시작됩니다. 작업은 클라우드 서비스의 비즈니스 논리를 구현하며 Azure 부하 분산 장치를 통해 역할에 게시된 메시지에 응답할 수 있습니다. 다음 그림은 Azure 클라우드 서비스에서 역할 내에 있는 작업과 리소스의 수명을 보여줍니다.

![The lifecycle of tasks and resources in a role in a Azure cloud service](./_images/compute-resource-consolidation-lifecycle.png)


_ComputeResourceConsolidation.Worker_ 프로젝트에 포함되어 있는 _WorkerRole.cs_ 파일에서 Azure 클라우드 서비스에서 이런 패턴을 구현하는 방법의 예제를 확인할 수 있습니다.

> _ComputeResourceConsolidation.Worker_ 프로젝트는 _ComputeResourceConsolidation_ 솔루션의 일부로 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation) 에서 다운로드할 수 있습니다.

작업자 역할에서 역할을 시작할 때 실행되는 코드는 필요한 취소 토큰과 실행할 작업의 목록을 생성합니다.

```csharp
public class WorkerRole: RoleEntryPoint
{
  // The cancellation token source used to cooperatively cancel running tasks.
  private readonly CancellationTokenSource cts = new CancellationTokenSource ();

  // List of tasks running on the role instance.
  private readonly List<Task> tasks = new List<Task>();

  // List of worker tasks to run on this role.
  private readonly List<Func<CancellationToken, Task>> workerTasks
                        = new List<Func<CancellationToken, Task>>
    {
      MyWorkerTask1,
      MyWorkerTask2
    };

  ...
}
```

`MyWorkerTask1` 및 `MyWorkerTask2` 메서드는 동일한 작업자 역할 내에서 여러 작업을 수행하는 방법을 보여줍니다. 다음 코드는 `MyWorkerTask1` 메서드를 보여줍니다. 이 메서드는 30초 동안 일시 중지한 다음 추적 메시지를 출력하는 간단한 작업으로, 작업이 취소될 때까지 이 프로세스를 반복합니다. `MyWorkerTask2` 메서드의 코드도 비슷합니다.

```csharp
// A sample worker role task.
private static async Task MyWorkerTask1(CancellationToken ct)
{
  // Fixed interval to wake up and check for work and/or do work.
  var interval = TimeSpan.FromSeconds(30);

  try
  {
    while (!ct.IsCancellationRequested)
    {
      // Wake up and do some background processing if not canceled.
      // TASK PROCESSING CODE HERE
      Trace.TraceInformation("Doing Worker Task 1 Work");

      // Go back to sleep for a period of time unless asked to cancel.
      // Task.Delay will throw an OperationCanceledException when canceled.
      await Task.Delay(interval, ct);
    }
  }
  catch (OperationCanceledException)
  {
    // Expect this exception to be thrown in normal circumstances or check
    // the cancellation token. If the role instances are shutting down, a
    // cancellation request will be signaled.
    Trace.TraceInformation("Stopping service, cancellation requested");

    // Rethrow the exception.
    throw;
  }
}
```

> 샘플 코드는 백그라운드 프로세스의 일반적인 구현을 보여줍니다. 사용자의 특정한 처리 논리를 취소 요청을 기다리는 루프의 본문에 삽입해야 한다는 점을 제외하면 실제 응용 프로그램에서도 이와 동일한 구조를 따를 수 있습니다.

작업자 역할이 사용하는 리소스를 시작하면 `Run` 메서드가 위에서 예로 든 두 가지 작업을 동시에 시작합니다.

```csharp
// RoleEntry Run() is called after OnStart().
// Returning from Run() will cause a role instance to recycle.
public override void Run()
{
  // Start worker tasks and add them to the task list.
  foreach (var worker in workerTasks)
    tasks.Add(worker(cts.Token));

  Trace.TraceInformation("Worker host tasks started");
  // The assumption is that all tasks should remain running and not return,
  // similar to role entry Run() behavior.
  try
  {
    Task.WaitAny(tasks.ToArray());
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then re-throw the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }

  // If there wasn't a cancellation request, stop all tasks and return from Run()
  // An alternative to canceling and returning when a task exits would be to
  // restart the task.
  if (!cts.IsCancellationRequested)
  {
    Trace.TraceInformation("Task returned without cancellation request");
    Stop(TimeSpan.FromMinutes(5));
  }
}
...
```

이 예에서 `Run` 메서드는 작업이 완료될 때까지 기다립니다. 작업이 취소되면 `Run` 메서드는 역할이 종료 중이고 나머지 작업이 취소될 때까지 기다렸다가 종료한다고(종료 전 최대 5분 대기) 판단합니다. 예상되는 예외로 인해 작업이 실패하면 `Run` 메서드는 작업을 취소합니다.

> 사용자는 실패한 작업을 다시 시작하거나 역할이 개별 작업을 중단 및 시작할 수 있도록 지원하는 코드를 삽입하는 등 `Run` 메서드에서 더 포괄적인 모니터링과 예외 처리 전략을 구현할 수 있습니다.

다음 코드에 제시된 `Stop` 메서드는 패브릭 컨트롤러가 역할 인스턴스를 종료할 때 호출됩니다(`OnStop` 메서드에서 호출). 다음 코드는 각 작업을 취소해 정상적으로 중단합니다. 작업 완료에 소요되는 시간이 5분을 초과하면 `Stop` 메서드의 취소 처리는 대기 상태를 중단하고 역할을 종료합니다.

```csharp
// Stop running tasks and wait for tasks to complete before returning
// unless the timeout expires.
private void Stop(TimeSpan timeout)
{
  Trace.TraceInformation("Stop called. Canceling tasks.");
  // Cancel running tasks.
  cts.Cancel();

  Trace.TraceInformation("Waiting for canceled tasks to finish and return");

  // Wait for all the tasks to complete before returning. Note that the
  // emulator currently allows 30 seconds and Azure allows five
  // minutes for processing to complete.
  try
  {
    Task.WaitAll(tasks.ToArray(), timeout);
  }
  catch (AggregateException ex)
  {
    Trace.TraceError(ex.Message);

    // If any of the inner exceptions in the aggregate exception
    // are not cancellation exceptions then rethrow the exception.
    ex.Handle(innerEx => (innerEx is OperationCanceledException));
  }
}
```

## 관련 패턴 및 지침

이 패턴을 구현할 때 관련될 수 있는 패턴과 지침은 다음과 같습니다.

- [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx). 자동 크기 조정은 예상되는 처리 요구에 따라 계산 리소스를 호스팅하는 서비스의 인스턴스를 시작 및 중단하는 데 사용할 수 있습니다.

- [계산 분할 지침](https://msdn.microsoft.com/library/dn589773.aspx). 서비스의 확장성, 성능, 가용성, 보안을 유지하면서 운영 비용을 최소화하는 데 도움을 주는 방식으로, 클라우드 서비스에 서비스와 구성 요소를 할당하는 방법을 설명합니다.

- 이 패턴에는 다운로드할 수 있는 [샘플 응용 프로그램](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation) 이 포함되어 있습니다.
