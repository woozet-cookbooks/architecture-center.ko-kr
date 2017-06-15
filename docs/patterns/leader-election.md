---
title: Leader Election
description: Coordinate the actions performed by a collection of collaborating task instances in a distributed application by electing one instance as the leader that assumes responsibility for managing the other instances.
keywords: design pattern
author: dragon119
ms.service: guidance
ms.topic: article
ms.author: pnp
ms.date: 03/24/2017

pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: [design-implementation, resiliency]

---

# 리더 선정

[!INCLUDE [header](../_includes/header.md)]

하나의 인스턴스를 다른 인스턴스의 관리를 책임지는 리더로 선정해 배포 응용 프로그램 내에 있는 공동 작업 인스턴스 모음이 수행하는 동작을 조율합니다. 이렇게 하면 인스턴스가 서로 충돌하지 않고, 공유 리소스를 위한 경합을 초래하지 않으며, 다른 인스턴스가 수행 중인 작업을 실수로 방해하지 않는 데 도움을 줄 수 있습니다. 

## 배경 및 문제

일반적인 클라우드 응용 프로그램에는 조율 방식으로 동작하는 많은 작업이 있습니다. 이런 작업은 모두 동일한 코드를 실행하고 동일한 리소스에 액세스할 필요가 있는 인스턴스일 수 있고 또는 복잡한 계산의 개별 부분을 수행하기 위해 동시에 함께 실행될 수 있습니다.

작업 인스턴스는 대부분 개별적으로 실행되지만 인스턴스가 서로 충돌하지 않고, 공유 리소스를 위한 경합을 초래하지 않으며, 다른 작업 인스턴스가 수행 중인 작업을 실수로 방해하지 않도록 각 인스턴스의 동작을 조율할 필요가 있습니다.

예를 들어

- 수평 확장을 구현하는 클라우드 기반 시스템에서 동일한 작업의 여러 인스턴스는 각각의 인스턴스를 서로 다른 사용자가 사용함에 따라 동시에 실행될 수 있습니다. 이런 인스턴스가 공유 리소스에 쓰기를 수행하는 경우, 각 인스턴스가 다른 인스턴스에서 수행된 변경 사항을 덮어쓰지 않도록 동작을 조율할 필요가 있습니다.
- 복잡한 계산의 개별 요소를 동시에 수행하는 작업의 경우, 작업이 모두 완료되면 결과를 집계할 필요가 있습니다.

작업 인스턴스는 모두 동료이므로 조율자 또는 집계자로 작용할 수 있는 기본 리더가 없습니다.

## 해결책

리더로 동작할 단일 작업 인스턴스를 선정해야 하고, 선정된 리더 인스턴스는 다른 하위 작업 인스턴스의 동작을 조율해야 합니다. 동일한 코드를 실행하는 작업 인스턴스는 모두 리더로 동작할 수 있습니다. 따라서 둘 이상의 인스턴스가 동시에 리더 역할을 맡지 않도록 리더 선정 프로세스를 신중하게 관리해야 합니다.

시스템은 리더를 선정하는 탄탄한 방식을 제공해야 합니다. 이 방법은 네트워크 중단 또는 프로세스 실패와 같은 이벤트를 처리해야 합니다. 많은 솔루션에서 하위 작업 인스턴스는 특정 유형의 하트비트 방법 또는 폴링을 통해 리더를 모니터링합니다.  지정된 리더가 예기치 않게 종료되거나 네트워크 장애로 인해 리더와 하위 작업 인스턴스의 연결이 해제되는 경우, 새로운 리더를 선정할 필요가 있습니다.

분산 환경에서 작업 집합 중 리더를 선정하는 여러 전략은 다음과 같습니다.
- 최저 순위 인스턴스 또는 프로세스 ID를 포함하는 작업 인스턴스를 선정
- 공유되고 분산된 뮤텍스를 가져오기 위해 경쟁. 뮤텍스를 가져오는 첫 번째 작업 인스턴스는 리더입니다. 그러나 리더가 종료되거나 시스템의 나머지와 리더의 연결이 해제되는 경우 시스템은 다른 작업 인스턴스를 리더로 허용하도록 뮤텍스가 해제되는지 확인해야 합니다.
- [불리 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) 또는 [링 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)과 일반 리더 선정 알고리즘 중 하나를 구현. 이런 알고리즘은 각각의 리더 후보가 고유 ID를 보유하고 다른 후보와 안정적으로 통신할 수 있다고 가정합니다. 

## 문제 및 고려 사항

이 패턴의 구현 방법을 결정할 때는 다음 사항을 고려해야 합니다.
- 리더를 선정하는 프로세스는 일시적 및 지속적 실패에 대해 복원력이 있어야 합니다. 
- 리더가 고장 났거나 그 밖의 이유로 사용할 수 없게 되었을 때(예: 통신 장애) 이를 감지할 수 있어야 합니다. 얼마나 빨리 감지해야 하는가의 문제는 시스템에 따라 달라집니다. 일시적 오류를 해결하는 짧은 시간 동안 리더 없이 작동할 수 있는 시스템도 있지만 리더 장애를 즉시 감지하고 새로운 리더 선정을 트리거해야 하는 시스템도 있습니다.
- 수평 확장을 구현하는 시스템에서 시스템이 규모를 축소하고 계산 리소스 중 일부를 종료하면 리더가 종료될 수 있습니다.
- 공유되고 분산된 뮤텍스를 사용하면 뮤텍스를 제공하는 외부 서비스에 의존하게 됩니다. 서비스는 단일 실패 지점을 구성합니다. 어떤 이유로든 서비스를 사용할 수 없게 되면 시스템은 리더를 선정할 수 없게 됩니다.
- 단일 전용 프로세스를 리더로 사용하는 간단한 접근 방식도 있습니다. 그러나 이 경우 프로세스가 실패하면 다시 시작하는 동안 상당한 지연이 초래될 수 있습니다. 다른 프로세스가 작업을 조율할 리더를 기다리는 경우, 결과적인 대기 시간은 다른 프로세스의 성능과 응답 시간에 영향을 미칠 수 있습니다.
- 리더 선정 알고리즘 중 하나를 수동으로 구현하면 코드의 튜닝과 최적화에 최고의 유연성을 제공할 수 있습니다.

## 패턴 사용 사례

이 패턴은 클라우드 호스팅 솔루션과 같은 분산 응용 프로그램의 작업에 신중한 조율이 필요하고 기본 리더가 없을 때 사용합니다.

>  리더가 시스템에서 병목 현상을 일으키지 않도록 주의해야 합니다. 리더의 목적은 하위 작업의 동작을 조율하는 것입니다. 리더로 선정되지 않더라도 하위 작업의 동작에 참여할 수 있지만, 리더로 선정되든 아니든 하위 작업의 동작에 반드시 참여할 필요는 없습니다.

다음의 경우에는 이 패턴이 유용하지 않습니다.
- 항상 리더로 작용할 수 있는 기본 리더 또는 전용 프로세스가 있는 경우. 예를 들면 작업 인스턴스를 조율하는 단일 프로세스를 구현할 수 있습니다. 이 프로세스가 실패하거나 비정상이 되면 시스템은 프로세스를 종료하고 다시 시작할 수 있습니다.
- 더 간단한 방법을 사용해 작업을 조율할 수 있는 경우. 예를 들어 여러 작업 인스턴스가 단순히 공유 리소스에 대한 조율된 액세스를 필요로 하는 경우, 효율적인 솔루션은 낙관적 잠금 또는 최악 잠금을 사용해 액세스를 제어하는 것입니다.
- 타사 솔루션이 더 적절한 경우. 예를 들면 Microsoft Azure HDInsight 서비스(Apache Hadoop 기반)는 Apache Zookeeper가 제공하는 서비스를 사용해 지도를 조정하고 데이터 수집 및 요약 작업을 줄입니다.

## 예제

LeaderElection 솔루션의 DistributedMutex 프로젝트(이 패턴을 보여주는 샘플은 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)에서 다운로드 가능)는 Azure Storage Blob 임대를 사용해 공유되고 분산된 뮤텍스를 구현하기 위한 메커니즘을 제공하는 방법을 보여줍니다. 이런 뮤텍스는 Azure 클라우드 서비스에서 역할 인스턴스의 그룹 중 리더를 선정하는 데 사용할 수 있습니다. 임대를 가져오는 첫 번째 역할 인스턴스는 리더로 선정되고 임대를 해제하거나 임대를 갱신할 수 없을 때까지 리더로 유지됩니다. 다른 역할 인스턴스는 리더를 더 이상 사용할 수 없는 경우 Blob 임대를 계속 모니터링할 수 있습니다.

>  Blob 임대는 Blob에 대한 배타적 쓰기 잠금입니다. 단일 Blob은 특정 시점에 한 번뿐인 임대의 대상일 수 있습니다. 역할 인스턴스는 지정된 Blob에 대한 임대를 요청할 수 있고 다른 역할 인스턴스가 동일한 Blob에 대한 임대를 유지하지 않으면 임대가 허용됩니다. 그렇지 않으면 요청은 예외를 발생시킵니다.

> 오류가 발생한 역할 인스턴스가 임대를 무한정 유지하는 것을 방지하려면 임대의 수명을 지정합니다. 수명이 만료되면 임대는 사용할 수 있게 됩니다. 그러나 역할 인스턴스가 임대를 유지하는 동안에는 임대의 갱신을 요청할 수 있고 갱신되면 임대를 추가 기간 동안 유지하게 됩니다. 임대를 유지하길 원하는 경우 역할 인스턴스는 이런 프로세스를 계속 반복할 수 있습니다. Blob 임대 방법에 대한 추가 정보는 [Blob 임대(REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx)를 참조하시기 바랍니다.

다음 C# 예제의 `BlobDistributedMutex` 클래스는 역할 인스턴스가 지정된 Blob에 대한 임대를 획득하게 해주는 `RunTaskWhenMutexAquired` 메서드를 포함합니다. Blob의 세부 정보(이름, 컨테이너, 저장소 계정)는 `BlobDistributedMutex` 개체(샘플 코드에 포함된 간단한 구조체)를 생성할 때 `BlobSettings` 개체 내의 생성자에 전달됩니다. 또한 생성자는 blob에 대한 임대를 성공적으로 획득하고 리더로 선정된 경우 역할 인스턴스가 실행해야 하는 코드를 참조하는 `Task` 를 수락합니다. 임대 획득에 대한 낮은 수준의 세부 정보를 처리하는 코드는 `BlobLeaseManager` 이름의 별도 도우미 클래스 내에 구현됩니다.

```csharp
public class BlobDistributedMutex
{
  ...
  private readonly BlobSettings blobSettings;
  private readonly Func<CancellationToken, Task> taskToRunWhenLeaseAcquired;
  ...

  public BlobDistributedMutex(BlobSettings blobSettings,
           Func<CancellationToken, Task> taskToRunWhenLeaseAquired)
  {
    this.blobSettings = blobSettings;
    this.taskToRunWhenLeaseAquired = taskToRunWhenLeaseAquired;
  }

  public async Task RunTaskWhenMutexAcquired(CancellationToken token)
  {
    var leaseManager = new BlobLeaseManager(blobSettings);
    await this.RunTaskWhenBlobLeaseAcquired(leaseManager, token);
  }
  ...
```

위의 코드 샘플에 포함되어 있는 `RunTaskWhenMutexAquired` 메서드는 실제로 임대를 획득하기 위해 다음 코드 샘플에 제시된 `RunTaskWhenBlobLeaseAcquired` 메서드를 호출합니다. `RunTaskWhenBlobLeaseAcquired` 메서드는 비동기적으로 실행됩니다. 임대를 성공적으로 획득하면 역할 인스턴스는 리더로 선정된 상태입니다. `taskToRunWhenLeaseAcquired` 대리자의 목적은 다른 역할 인스턴스를 조율하는 동작을 수행하는 것입니다. 임대를 획득하지 못하면 다른 역할 인스턴스가 리더로 선정되고 현재 역할 인스턴스는 하위로 내려갑니다. `TryAcquireLeaseOrWait` 메서드는 `BlobLeaseManager` 개체를 사용해 임대를 획득하는 도우미 메서드입니다.

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (!token.IsCancellationRequested)
    {
      // Try to acquire the blob lease.
      // Otherwise wait for a short time before trying again.
      string leaseId = await this.TryAquireLeaseOrWait(leaseManager, token);

      if (!string.IsNullOrEmpty(leaseId))
      {
        // Create a new linked cancellation token source so that if either the
        // original token is canceled or the lease can't be renewed, the
        // leader task can be canceled.
        using (var leaseCts =
          CancellationTokenSource.CreateLinkedTokenSource(new[] { token }))
        {
          // Run the leader task.
          var leaderTask = this.taskToRunWhenLeaseAquired.Invoke(leaseCts.Token);
          ...
        }
      }
    }
    ...
  }
```

리더가 시작한 작업도 비동기적으로 실행됩니다. 이 작업이 실행되는 동안 다음 코드 샘플에 제시된 `RunTaskWhenBlobLeaseAquired` 메서드는 임대를 주기적으로 갱신합니다. 갱신은 역할 인스턴스를 리더로 유지하는 것을 도와줍니다. 샘플 솔루션에서 갱신 요청 사이의 지연은 임대의 지속을 위해 지정하는 시간보다 짧아야 합니다. 그래야 다른 역할 인스턴스가 리더로 선정되는 것을 막을 수 있습니다. 어떤 이유로든 갱신이 실패하면 작업이 취소됩니다.

임대가 갱신에 실패하거나 작업이 취소되면(역할 인스턴스 종료의 결과로 가능) 임대가 해제됩니다. 임대가 해제되면 갱신에 실패한 역할 인스턴스 또는 다른 역할 인스턴스는 리더로 선정될 수 있습니다.  다음 인용 코드는 프로세스 중 갱신 부분을 보여줍니다.

```csharp
  private async Task RunTaskWhenBlobLeaseAcquired(
    BlobLeaseManager leaseManager, CancellationToken token)
  {
    while (...)
    {
      ...
      if (...)
      {
        ...
        using (var leaseCts = ...)
        {
          ...
          // Keep renewing the lease in regular intervals.
          // If the lease can't be renewed, then the task completes.
          var renewLeaseTask =
            this.KeepRenewingLease(leaseManager, leaseId, leaseCts.Token);

          // When any task completes (either the leader task itself or when it
          // couldn't renew the lease) then cancel the other task.
          await CancelAllWhenAnyCompletes(leaderTask, renewLeaseTask, leaseCts);
        }
      }
    }
  }
  ...
}
```

`KeepRenewingLease` 메서드는 `BlobLeaseManager` 개체를 사용해 임대를 갱신하는 다른 도우미 메서드입니다. `CancelAllWhenAnyCompletes` 메서드는 처음 두 매개 변수로 지정된 작업을 취소합니다. 다음 다이어그램은 `BlobDistributedMutex` 클래스를 사용해 리더를 선정하고 동작을 조율하는 작업을 실행하는 과정을 보여줍니다.

![Figure 1 illustrates the functions of the BlobDistributedMutex class](./_images/leader-election-diagram.png)


다음 코드 예제는 작업자 역할에서 `BlobDistributedMutex` 클래스의 사용 방법을 보여줍니다. 이 코드는 개발 단계에서 임대의 컨테이너 내에 있는 `MyLeaderCoordinatorTask` 이름의 Blob에 대한 임대를 획득하고 역할 인스턴스가 리더로 선정된 경우 `MyLeaderCoordinatorTask` 메서드에 정의된 코드의 실행을 지정합니다.

```csharp
var settings = new BlobSettings(CloudStorageAccount.DevelopmentStorageAccount,
  "leases", "MyLeaderCoordinatorTask");
var cts = new CancellationTokenSource();
var mutex = new BlobDistributedMutex(settings, MyLeaderCoordinatorTask);
mutex.RunTaskWhenMutexAcquired(this.cts.Token);
...

// Method that runs if the role instance is elected the leader
private static async Task MyLeaderCoordinatorTask(CancellationToken token)
{
  ...
}
```

샘플 솔루션과 관련해 다음 사항에 유의하시기 바랍니다.
- Blob은 잠재적인 단일 실패 지점입니다. Blob 서비스를 사용할 수 없거나 액세스할 수 없게 되면 리더는 임대를 갱신할 수 없게 되고 다른 역할 인스턴스가 임대를 획득할 수 없게 됩니다. 이 경우 어떤 역할 인스턴스도 리더로 작용할 수 없게 됩니다. 그러나 Blob 서비스는 복원력이 있도록 설계되었으므로 Blob 서비스의 완전한 장애는 거의 가능성이 없다고 간주됩니다.
- 리더가 수행 중인 작업이 중지되면 리더는 다른 역할 인스턴스가 임대를 획득하지 못하게 하고 작업을 조율하는 리더 역할을 맡도록 임대를 계속 갱신할 수 있습니다. 실제로 리더의 상태는 자주 확인해야 합니다.
- 선정 프로세스는 비결정적입니다. 사용자는 어떤 역할 인스턴스가 Blob 임대를 획득해 리더가 될지를 가정할 수 없습니다.
- Blob 임대의 대상으로 사용되는 Blob은 다른 목적으로 사용되지 않아야 합니다. 역할 인스턴스가 데이터를 이 Blob에 저장하는 경우, 역할 인스턴스가 리더이고 Blob 임대를 유지하지 않으면 이 데이터에 액세스할 수 없습니다. 

## 관련 패턴 및 지침

이 패턴을 구현할 때 관련될 수 있는 지침은 다음과 같습니다.
- 이 패턴에는 다운로드할 수 있는 [샘플 응용 프로그램](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)이 포함되어 있습니다.
- [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx). 응용 프로그램 부하의 변동에 따라 작업 호스트의 인스턴스를 시작하고 정지할 수 있습니다. 자동 크기 조정은 최대 처리 시간 동안 처리량과 성능을 유지하는 데 도움을 줄 수 있습니다.
- [계산 분할 지침](https://msdn.microsoft.com/library/dn589773.aspx). 이 지침은 서비스의 확장성, 성능, 가용성 및 보안을 유지하면서 운영 비용을 최소화하는 데 도움을 주는 방식으로 클라우드 서비스에서 작업을 호스트에 할당하는 방법을 설명합니다.
- [작업 기반 비동기 패턴](https://msdn.microsoft.com/library/hh873175.aspx).
- [불리 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html)을 보여주는 예제
- [링 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)을 보여주는 예제
- Microsoft Open Technologies 웹 사이트에 게시된 [Microsoft Azure에서 Apache Zookeeper](https://msopentech.com/opentech-projects/apache-zookeeper-on-windows-azure-2/)
- MSDN에 게시된 [Blob 임대(REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx) 
