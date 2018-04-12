---
title: 리더 선택
description: 인스턴스 중 하나를 다른 인스턴스를 관리하는 리더로 선택하여 분산된 응용 프로그램의 공동 작업 인스턴스 컬렉션이 수행하는 작업을 조정합니다.
keywords: 디자인 패턴
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- resiliency
ms.openlocfilehash: 3e7d47f70f660f2507f0619e1c41bf9a32a25be4
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="leader-election-pattern"></a>리더 선택 패턴

[!INCLUDE [header](../_includes/header.md)]

인스턴스 중 하나를 다른 인스턴스를 관리하는 리더로 선택하여 분산된 응용 프로그램에서 공동 작업 인스턴스 컬렉션이 수행하는 작업을 조정합니다. 이렇게 하면 인스턴스가 서로 충돌하거나, 공유 리소스에 대한 경합을 일으키거나, 실수로 다른 인스턴스가 수행 중인 작업을 방해하지 않도록 방지할 수 있습니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제점

기존 클라우드 응용 프로그램에는 조정된 방식으로 작동하는 많은 태스크가 있습니다. 이러한 태스크는 모두 동일한 코드를 실행하고 동일한 리소스에 액세스해야 하는 인스턴스이거나, 병렬로 함께 작동하여 복잡한 계산의 개별 부분을 수행할 수 있습니다.

태스크 인스턴스는 대부분의 시간 동안 개별적으로 실행될 수 있지만 각 인스턴스의 작업을 조정하여 충돌하거나, 공유 리소스에 대한 경합을 일으키거나, 실수로 다른 태스크 인스턴스가 수행 중인 작업을 방해하지 않도록 해야 할 수도 있습니다.

예: 

- 수평적 크기 조정을 구현하는 클라우드 기반 시스템에서는 동일한 태스크의 여러 인스턴스가 동시에 실행되고 각 인스턴스가 다른 사용자에게 서비스를 제공할 수 있습니다. 이러한 인스턴스가 공유 리소스에 쓰는 경우 해당 작업을 조정하여 각 인스턴스가 다른 인스턴스의 변경 내용을 덮어쓰지 못하도록 해야 합니다.
- 태스크가 복잡한 계산의 개별 요소를 병렬로 수행하는 경우 모두 완료되면 결과를 집계해야 합니다.

태스크 인스턴스는 모두 피어이므로 코디네이터 또는 집계 역할을 할 수 있는 자연 리더가 없습니다.

## <a name="solution"></a>해결 방법

리더 역할을 하도록 단일 태스크 인스턴스를 선택해야 하며, 이 인스턴스는 다른 하위 태스크 인스턴스의 작업을 조정해야 합니다. 모든 태스크 인스턴스가 동일한 코드를 실행하는 경우 각각 리더 역할을 할 수 있습니다. 따라서 둘 이상의 인스턴스가 동시에 리더 역할을 맡지 않도록 선택 프로세스를 신중하게 관리해야 합니다.

시스템이 리더를 선택하는 강력한 메커니즘을 제공해야 합니다. 이 메서드는 네트워크 가동 중단 또는 프로세스 실패와 같은 이벤트를 처리해야 합니다. 많은 솔루션에서 하위 태스크 인스턴스는 일부 유형의 하트비트 메서드 또는 폴링을 통해 리더를 모니터합니다. 지정된 리더가 예기치 않게 종료되거나 네트워크 오류로 인해 하위 태스크 인스턴스에서 리더를 사용할 수 없는 경우 하위 태스크 인스턴스가 새 리더를 선택해야 합니다.

다음을 포함하여 분산 환경의 태스크 집합 중에서 리더를 선택하는 몇 가지 전략이 있습니다.
- 순위가 가장 낮은 인스턴스 또는 프로세스 ID를 가진 태스크 인스턴스 선택.
- 공유 분산 뮤텍스를 확보하기 위해 경합. 뮤텍스를 확보하는 첫 번째 태스크 인스턴스가 리더가 됩니다. 그러나 리더가 종료되거나 시스템의 나머지 부분에서 연결이 끊길 경우 뮤텍스가 해제되어 다른 태스크 인스턴스가 리더가 될 수 있도록 해야 합니다.
- [Bully 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) 또는 [Ring 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)과 같은 공용 리더 선택 알고리즘 중 하나 구현. 이러한 알고리즘은 각 선택 후보에 고유 ID가 있으며 다른 후보와 안정적으로 통신할 수 있다고 가정합니다.

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.
- 리더 선택 프로세스는 일시적 오류와 지속적 오류에 대한 복원력이 있어야 합니다.
- 리더가 실패했거나 달리 사용할 수 없게 되면(예: 통신 오류) 감지할 수 있어야 합니다. 필요한 감지 속도는 시스템에 따라 다릅니다. 일부 시스템은 짧은 시간 동안 리더 없이 작동할 수 있으며, 일시적 결함은 이 시간 동안 해결될 수 있습니다. 리더 오류를 즉시 감지하고 새 선택을 트리거해야 하는 경우도 있습니다.
- 수평적 자동 크기 조정을 구현하는 시스템에서는 시스템이 다시 축소되고 일부 계산 리소스를 종료할 경우 리더가 종료될 수 있습니다.
- 공유 분산 뮤텍스를 사용하면 뮤텍스를 제공하는 외부 서비스에 대한 종속성이 도입됩니다. 서비스는 단일 실패 지점을 구성합니다. 어떤 이유로든 서비스를 사용할 수 없게 되면 시스템에서 리더를 선택할 수 없습니다.
- 단일 전용 프로세스를 리더로 사용하는 것이 간단한 방법입니다. 그러나 프로세스가 실패할 경우 다시 시작하는 동안 상당한 지연이 발생할 수 있습니다. 리더가 작업을 조정할 때까지 기다리는 경우 그 결과로 인한 대기 시간이 다른 프로세스의 성능과 응답 시간에 영향을 미칠 수 있습니다.
- 리더 선택 알고리즘 중 하나를 수동으로 구현하면 코드를 조정하고 최적화할 수 있는 유연성이 극대화됩니다.

## <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

클라우드에 호스트된 솔루션과 같은 분산 응용 프로그램의 태스크에 신중한 조정이 필요하고 자연 리더가 없는 경우 이 패턴을 사용합니다.

>  리더가 시스템의 병목 상태가 되지 않도록 합니다. 리더의 목적은 하위 태스크의 작업을 조정하는 것이며 이 작업 자체에 반드시 참여해야 하는 것은 아닙니다. 단, 태스크가 리더로 선택되지 않은 경우에는 참여할 수 있어야 합니다.

이 패턴은 다음과 같은 경우 유용하지 않을 수 있습니다.
- 항상 리더 역할을 할 수 있는 자연 리더 또는 전용 프로세스가 있는 경우. 예를 들어 태스크 인스턴스를 조정하는 단일 프로세스를 구현할 수 있습니다. 이 프로세스가 실패하거나 비정상 상태가 되면 시스템이 종료하고 다시 시작할 수 있습니다.
- 더 경량 방법을 사용하여 태스크 간 조정을 수행할 수 있는 경우. 예를 들어 단순히 여러 태스크 인스턴스에 공유 리소스에 대한 조정된 액세스가 필요한 경우 더 나은 솔루션은 낙관적 또는 비관적 잠금을 사용하여 액세스를 제어하는 것입니다.
- 타사 솔루션이 더 적합한 경우. 예를 들어 Microsoft Azure HDInsight 서비스(Apache Hadoop 기반)는 Apache Zookeeper에서 제공하는 서비스를 사용하여 맵을 조정하고 데이터를 수집 및 요약하는 태스크를 줄입니다.

## <a name="example"></a>예

LeaderElection 솔루션의 DistributedMutex 프로젝트([GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)에서 이 패턴을 보여 주는 샘플을 확인할 수 있음)는 Azure Storage Blob의 임대를 사용하여 공유 분산 뮤텍스를 구현하는 메커니즘을 제공하는 방법을 보여 줍니다. 이 뮤텍스를 사용하여 Azure 클라우드 서비스의 역할 인스턴스 그룹 중에서 리더를 선택할 수 있습니다. 임대를 획득한 첫 번째 역할 인스턴스가 리더로 선택되고, 임대를 해제하거나 임대를 갱신할 수 없을 때까지 리더로 유지됩니다. 다른 역할 인스턴스는 리더를 더 이상 사용할 수 없는 경우를 위해 Blob 임대를 계속 모니터할 수 있습니다.

>  Blob 임대는 Blob에 대한 배타적 쓰기 잠금입니다. 단일 Blob은 언제든지 한 임대의 주체만 될 수 있습니다. 역할 인스턴스가 지정된 Blob에 대한 임대를 요청할 수 있으며, 동일한 Blob에 대한 임대를 보유한 다른 역할 인스턴스가 없는 경우 임대가 부여됩니다. 다른 역할 인스턴스가 임대를 보유한 경우 요청에서 예외가 발생합니다.
> 
> 결함 있는 역할 인스턴스가 임대를 무기한 유지하는 것을 방지하려면 임대 수명을 지정합니다. 이 수명이 만료되면 임대를 사용할 수 있게 됩니다. 그러나 역할 인스턴스가 임대를 보유하는 동안 임대가 갱신되도록 요청할 수 있으며, 추가 기간 동안 임대가 부여됩니다. 역할 인스턴스가 임대를 유지하려는 경우 이 프로세스를 계속 반복할 수 있습니다.
> Blob 임대 방법에 대한 자세한 내용은 [Blob 임대(REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx)를 참조하세요.

아래 C# 예제의 `BlobDistributedMutex` 클래스에는 역할 인스턴스가 지정된 Blob에 대한 임대를 획득하려고 시도할 수 있게 하는 `RunTaskWhenMutexAquired` 메서드가 포함되어 있습니다. `BlobDistributedMutex` 개체를 만들 때(이 개체는 샘플 코드에 포함된 단순 구조체임) Blob의 세부 정보(이름, 컨테이너, 저장소 계정)가 `BlobSettings` 개체의 생성자에게 전달됩니다. 또한 생성자는 Blob에 대한 임대를 획득하고 리더로 선택된 경우 역할 인스턴스가 실행해야 하는 코드를 참조하는 `Task`를 허용합니다. 임대 획득의 하위 수준 세부 정보를 처리하는 코드는 `BlobLeaseManager`라는 별도의 도우미 클래스에서 구현됩니다.

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

위 코드 샘플의 `RunTaskWhenMutexAquired` 메서드는 실제로 임대를 획득하기 위해 다음 코드 샘플에 표시된 `RunTaskWhenBlobLeaseAcquired` 메서드를 호출합니다. `RunTaskWhenBlobLeaseAcquired` 메서드는 비동기적으로 실행됩니다. 임대를 획득한 경우 역할 인스턴스가 리더로 선택된 것입니다. `taskToRunWhenLeaseAcquired` 대리자의 목적은 다른 역할 인스턴스를 조정하는 작업을 수행하는 것입니다. 임대를 획득하지 못한 경우 다른 역할 인스턴스가 리더로 선택된 것이며 현재 역할 인스턴스가 하위로 유지됩니다. `TryAcquireLeaseOrWait` 메서드는 `BlobLeaseManager` 개체를 사용하여 임대를 획득하는 도우미 메서드입니다.

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

리더가 시작한 태스크도 비동기적으로 실행됩니다. 이 태스크가 실행되는 동안 다음 코드 샘플에 표시된 `RunTaskWhenBlobLeaseAquired` 메서드는 정기적으로 임대를 갱신하려고 시도합니다. 이렇게 하면 역할 인스턴스가 리더로 유지됩니다. 샘플 솔루션에서는 다른 역할 인스턴스가 리더로 선택되지 않도록 하기 위해 갱신 요청 사이의 지연이 임대 기간으로 지정된 시간보다 짧습니다. 어떤 이유로 인해 갱신에 실패하면 태스크가 취소됩니다.

임대가 갱신되지 않거나 태스크가 취소되면(역할 인스턴스 종료로 인해) 임대가 해제됩니다. 이 시점에서 이 역할 인스턴스나 다른 역할 인스턴스가 리더로 선택될 수 있습니다. 아래 추출된 코드는 이 프로세스 부분을 보여 줍니다.

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

`KeepRenewingLease` 메서드는 `BlobLeaseManager` 개체를 사용하여 임대를 갱신하는 다른 도우미 메서드입니다. `CancelAllWhenAnyCompletes` 메서드는 처음 두 매개 변수로 지정된 태스크를 취소합니다. 다음 다이어그램은 `BlobDistributedMutex` 클래스를 사용하여 리더를 선택하고 작업을 조정하는 태스크를 실행하는 방법을 보여 줍니다.

![그림 1은 BlobDistributedMutex 클래스의 함수를 보여 줍니다.](./_images/leader-election-diagram.png)


다음 코드 예제는 작업자 역할에 `BlobDistributedMutex` 클래스를 사용하는 방법을 보여 줍니다. 이 코드는 개발 저장소의 임대 컨테이너에 있는 `MyLeaderCoordinatorTask` Blob에 대한 임대를 획득하고, 역할 인스턴스가 리더로 선택된 경우 `MyLeaderCoordinatorTask` 메서드에 정의된 코드가 실행되도록 지정합니다.

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

샘플 솔루션에 대한 다음 사항에 유의하세요.
- Blob은 잠재적인 단일 실패 지점입니다. Blob 서비스를 사용할 수 없거나 액세스할 수 없는 경우 리더가 임대를 갱신할 수 없으며 다른 역할 인스턴스도 임대를 획득할 수 없습니다. 이 경우 리더 역할을 할 수 있는 역할 인스턴스가 없습니다. 그러나 Blob 서비스는 복원력이 있도록 디자인되었으므로 Blob 서비스가 완전히 실패할 가능성은 거의 없습니다.
- 리더가 수행 중인 태스크가 중단되면 리더가 임대를 계속 갱신하여 다른 역할 인스턴스가 태스크를 조정하기 위해 임대를 획득하고 리더 역할을 맡지 못하도록 할 수 있습니다. 프로덕션에서는 리더의 상태를 자주 확인해야 합니다.
- 선택 프로세스는 비결정적입니다. 어떤 역할 인스턴스가 Blob 임대를 획득하고 리더가 될지 가정할 수 없습니다.
- Blob 임대의 대상으로 사용된 Blob은 다른 용도로 사용하면 안 됩니다. 역할 인스턴스가 이 Blob에 데이터를 저장하려고 시도할 경우 역할 인스턴스가 리더이고 Blob 임대를 보유한 경우가 아니면 이 데이터에 액세스할 수 없습니다.

## <a name="related-patterns-and-guidance"></a>관련 패턴 및 지침

이 패턴을 구현할 때 다음 지침도 관련이 있을 수 있습니다.
- 이 패턴에는 다운로드 가능한 [샘플 응용 프로그램](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)이 있습니다.
- [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx). 응용 프로그램의 부하가 변경됨에 따라 태스크 호스트 인스턴스를 시작 및 중지할 수 있습니다. 자동 크기 조정을 사용하면 최고 처리 시간 동안 처리량과 성능을 유지할 수 있습니다.
- [Compute 분할 지침](https://msdn.microsoft.com/library/dn589773.aspx) 이 지침은 서비스의 확장성, 성능, 가용성 및 보안을 유지하면서 실행 비용을 최소화하는 데 도움이 되는 방식으로 클라우드 서비스의 호스트에 태스크를 할당하는 방법을 설명합니다.
- [태스크 기반 비동기 패턴](https://msdn.microsoft.com/library/hh873175.aspx).
- [Bully 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html)을 보여 주는 예제.
- [Ring 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)을 보여 주는 예제.
- Microsoft Open Technologies 웹 사이트에 있는 [Microsoft Azure의 Apache Zookeeper](https://msopentech.com/opentech-projects/apache-zookeeper-on-windows-azure-2/) 문서.
- Apache ZooKeeper용 클라이언트 라이브러리인 [Apache Curator](http://curator.apache.org/).
- MSDN에 있는 [Blob 임대(REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx) 문서.
