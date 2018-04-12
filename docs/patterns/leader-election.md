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
# <a name="leader-election-pattern"></a><span data-ttu-id="6185c-104">리더 선택 패턴</span><span class="sxs-lookup"><span data-stu-id="6185c-104">Leader Election pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="6185c-105">인스턴스 중 하나를 다른 인스턴스를 관리하는 리더로 선택하여 분산된 응용 프로그램에서 공동 작업 인스턴스 컬렉션이 수행하는 작업을 조정합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-105">Coordinate the actions performed by a collection of collaborating instances in a distributed application by electing one instance as the leader that assumes responsibility for managing the others.</span></span> <span data-ttu-id="6185c-106">이렇게 하면 인스턴스가 서로 충돌하거나, 공유 리소스에 대한 경합을 일으키거나, 실수로 다른 인스턴스가 수행 중인 작업을 방해하지 않도록 방지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-106">This can help to ensure that instances don't conflict with each other, cause contention for shared resources, or inadvertently interfere with the work that other instances are performing.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="6185c-107">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="6185c-107">Context and problem</span></span>

<span data-ttu-id="6185c-108">기존 클라우드 응용 프로그램에는 조정된 방식으로 작동하는 많은 태스크가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-108">A typical cloud application has many tasks acting in a coordinated manner.</span></span> <span data-ttu-id="6185c-109">이러한 태스크는 모두 동일한 코드를 실행하고 동일한 리소스에 액세스해야 하는 인스턴스이거나, 병렬로 함께 작동하여 복잡한 계산의 개별 부분을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-109">These tasks could all be instances running the same code and requiring access to the same resources, or they might be working together in parallel to perform the individual parts of a complex calculation.</span></span>

<span data-ttu-id="6185c-110">태스크 인스턴스는 대부분의 시간 동안 개별적으로 실행될 수 있지만 각 인스턴스의 작업을 조정하여 충돌하거나, 공유 리소스에 대한 경합을 일으키거나, 실수로 다른 태스크 인스턴스가 수행 중인 작업을 방해하지 않도록 해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-110">The task instances might run separately for much of the time, but it might also be necessary to coordinate the actions of each instance to ensure that they don’t conflict, cause contention for shared resources, or accidentally interfere with the work that other task instances are performing.</span></span>

<span data-ttu-id="6185c-111">예: </span><span class="sxs-lookup"><span data-stu-id="6185c-111">For example:</span></span>

- <span data-ttu-id="6185c-112">수평적 크기 조정을 구현하는 클라우드 기반 시스템에서는 동일한 태스크의 여러 인스턴스가 동시에 실행되고 각 인스턴스가 다른 사용자에게 서비스를 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-112">In a cloud-based system that implements horizontal scaling, multiple instances of the same task could be running at the same time with each instance serving a different user.</span></span> <span data-ttu-id="6185c-113">이러한 인스턴스가 공유 리소스에 쓰는 경우 해당 작업을 조정하여 각 인스턴스가 다른 인스턴스의 변경 내용을 덮어쓰지 못하도록 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-113">If these instances write to a shared resource, it's necessary to coordinate their actions to prevent each instance from overwriting the changes made by the others.</span></span>
- <span data-ttu-id="6185c-114">태스크가 복잡한 계산의 개별 요소를 병렬로 수행하는 경우 모두 완료되면 결과를 집계해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-114">If the tasks are performing individual elements of a complex calculation in parallel, the results need to be aggregated when they all complete.</span></span>

<span data-ttu-id="6185c-115">태스크 인스턴스는 모두 피어이므로 코디네이터 또는 집계 역할을 할 수 있는 자연 리더가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-115">The task instances are all peers, so there isn't a natural leader that can act as the coordinator or aggregator.</span></span>

## <a name="solution"></a><span data-ttu-id="6185c-116">해결 방법</span><span class="sxs-lookup"><span data-stu-id="6185c-116">Solution</span></span>

<span data-ttu-id="6185c-117">리더 역할을 하도록 단일 태스크 인스턴스를 선택해야 하며, 이 인스턴스는 다른 하위 태스크 인스턴스의 작업을 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-117">A single task instance should be elected to act as the leader, and this instance should coordinate the actions of the other subordinate task instances.</span></span> <span data-ttu-id="6185c-118">모든 태스크 인스턴스가 동일한 코드를 실행하는 경우 각각 리더 역할을 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-118">If all of the task instances are running the same code, they are each capable of acting as the leader.</span></span> <span data-ttu-id="6185c-119">따라서 둘 이상의 인스턴스가 동시에 리더 역할을 맡지 않도록 선택 프로세스를 신중하게 관리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-119">Therefore, the election process must be managed carefully to prevent two or more instances taking over the leader role at the same time.</span></span>

<span data-ttu-id="6185c-120">시스템이 리더를 선택하는 강력한 메커니즘을 제공해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-120">The system must provide a robust mechanism for selecting the leader.</span></span> <span data-ttu-id="6185c-121">이 메서드는 네트워크 가동 중단 또는 프로세스 실패와 같은 이벤트를 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-121">This method has to cope with events such as network outages or process failures.</span></span> <span data-ttu-id="6185c-122">많은 솔루션에서 하위 태스크 인스턴스는 일부 유형의 하트비트 메서드 또는 폴링을 통해 리더를 모니터합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-122">In many solutions, the subordinate task instances monitor the leader through some type of heartbeat method, or by polling.</span></span> <span data-ttu-id="6185c-123">지정된 리더가 예기치 않게 종료되거나 네트워크 오류로 인해 하위 태스크 인스턴스에서 리더를 사용할 수 없는 경우 하위 태스크 인스턴스가 새 리더를 선택해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-123">If the designated leader terminates unexpectedly, or a network failure makes the leader unavailable to the subordinate task instances, it's necessary for them to elect a new leader.</span></span>

<span data-ttu-id="6185c-124">다음을 포함하여 분산 환경의 태스크 집합 중에서 리더를 선택하는 몇 가지 전략이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-124">There are several strategies for electing a leader among a set of tasks in a distributed environment, including:</span></span>
- <span data-ttu-id="6185c-125">순위가 가장 낮은 인스턴스 또는 프로세스 ID를 가진 태스크 인스턴스 선택.</span><span class="sxs-lookup"><span data-stu-id="6185c-125">Selecting the task instance with the lowest-ranked instance or process ID.</span></span>
- <span data-ttu-id="6185c-126">공유 분산 뮤텍스를 확보하기 위해 경합.</span><span class="sxs-lookup"><span data-stu-id="6185c-126">Racing to acquire a shared, distributed mutex.</span></span> <span data-ttu-id="6185c-127">뮤텍스를 확보하는 첫 번째 태스크 인스턴스가 리더가 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-127">The first task instance that acquires the mutex is the leader.</span></span> <span data-ttu-id="6185c-128">그러나 리더가 종료되거나 시스템의 나머지 부분에서 연결이 끊길 경우 뮤텍스가 해제되어 다른 태스크 인스턴스가 리더가 될 수 있도록 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-128">However, the system must ensure that, if the leader terminates or becomes disconnected from the rest of the system, the mutex is released to allow another task instance to become the leader.</span></span>
- <span data-ttu-id="6185c-129">[Bully 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) 또는 [Ring 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)과 같은 공용 리더 선택 알고리즘 중 하나 구현.</span><span class="sxs-lookup"><span data-stu-id="6185c-129">Implementing one of the common leader election algorithms such as the [Bully Algorithm](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html) or the [Ring Algorithm](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html).</span></span> <span data-ttu-id="6185c-130">이러한 알고리즘은 각 선택 후보에 고유 ID가 있으며 다른 후보와 안정적으로 통신할 수 있다고 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-130">These algorithms assume that each candidate in the election has a unique ID, and that it can communicate with the other candidates reliably.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="6185c-131">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="6185c-131">Issues and considerations</span></span>

<span data-ttu-id="6185c-132">이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="6185c-132">Consider the following points when deciding how to implement this pattern:</span></span>
- <span data-ttu-id="6185c-133">리더 선택 프로세스는 일시적 오류와 지속적 오류에 대한 복원력이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-133">The process of electing a leader should be resilient to transient and persistent failures.</span></span>
- <span data-ttu-id="6185c-134">리더가 실패했거나 달리 사용할 수 없게 되면(예: 통신 오류) 감지할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-134">It must be possible to detect when the leader has failed or has become otherwise unavailable (such as due to a communications failure).</span></span> <span data-ttu-id="6185c-135">필요한 감지 속도는 시스템에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-135">How quickly detection is needed is system dependent.</span></span> <span data-ttu-id="6185c-136">일부 시스템은 짧은 시간 동안 리더 없이 작동할 수 있으며, 일시적 결함은 이 시간 동안 해결될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-136">Some systems might be able to function for a short time without a leader, during which a transient fault might be fixed.</span></span> <span data-ttu-id="6185c-137">리더 오류를 즉시 감지하고 새 선택을 트리거해야 하는 경우도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-137">In other cases, it might be necessary to detect leader failure immediately and trigger a new election.</span></span>
- <span data-ttu-id="6185c-138">수평적 자동 크기 조정을 구현하는 시스템에서는 시스템이 다시 축소되고 일부 계산 리소스를 종료할 경우 리더가 종료될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-138">In a system that implements horizontal autoscaling, the leader could be terminated if the system scales back and shuts down some of the computing resources.</span></span>
- <span data-ttu-id="6185c-139">공유 분산 뮤텍스를 사용하면 뮤텍스를 제공하는 외부 서비스에 대한 종속성이 도입됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-139">Using a shared, distributed mutex introduces a dependency on the external service that provides the mutex.</span></span> <span data-ttu-id="6185c-140">서비스는 단일 실패 지점을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-140">The service constitutes a single point of failure.</span></span> <span data-ttu-id="6185c-141">어떤 이유로든 서비스를 사용할 수 없게 되면 시스템에서 리더를 선택할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-141">If it becomes unavailable for any reason, the system won't be able to elect a leader.</span></span>
- <span data-ttu-id="6185c-142">단일 전용 프로세스를 리더로 사용하는 것이 간단한 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-142">Using a single dedicated process as the leader is a straightforward approach.</span></span> <span data-ttu-id="6185c-143">그러나 프로세스가 실패할 경우 다시 시작하는 동안 상당한 지연이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-143">However, if the process fails there could be a significant delay while it's restarted.</span></span> <span data-ttu-id="6185c-144">리더가 작업을 조정할 때까지 기다리는 경우 그 결과로 인한 대기 시간이 다른 프로세스의 성능과 응답 시간에 영향을 미칠 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-144">The resulting latency can affect the performance and response times of other processes if they're waiting for the leader to coordinate an operation.</span></span>
- <span data-ttu-id="6185c-145">리더 선택 알고리즘 중 하나를 수동으로 구현하면 코드를 조정하고 최적화할 수 있는 유연성이 극대화됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-145">Implementing one of the leader election algorithms manually provides the greatest flexibility for tuning and optimizing the code.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="6185c-146">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="6185c-146">When to use this pattern</span></span>

<span data-ttu-id="6185c-147">클라우드에 호스트된 솔루션과 같은 분산 응용 프로그램의 태스크에 신중한 조정이 필요하고 자연 리더가 없는 경우 이 패턴을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-147">Use this pattern when the tasks in a distributed application, such as a cloud-hosted solution, need careful coordination and there's no natural leader.</span></span>

>  <span data-ttu-id="6185c-148">리더가 시스템의 병목 상태가 되지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-148">Avoid making the leader a bottleneck in the system.</span></span> <span data-ttu-id="6185c-149">리더의 목적은 하위 태스크의 작업을 조정하는 것이며 이 작업 자체에 반드시 참여해야 하는 것은 아닙니다. 단, 태스크가 리더로 선택되지 않은 경우에는 참여할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-149">The purpose of the leader is to coordinate the work of the subordinate tasks, and it doesn't necessarily have to participate in this work itself&mdash;although it should be able to do so if the task isn't elected as the leader.</span></span>

<span data-ttu-id="6185c-150">이 패턴은 다음과 같은 경우 유용하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-150">This pattern might not be useful if:</span></span>
- <span data-ttu-id="6185c-151">항상 리더 역할을 할 수 있는 자연 리더 또는 전용 프로세스가 있는 경우.</span><span class="sxs-lookup"><span data-stu-id="6185c-151">There's a natural leader or dedicated process that can always act as the leader.</span></span> <span data-ttu-id="6185c-152">예를 들어 태스크 인스턴스를 조정하는 단일 프로세스를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-152">For example, it might be possible to implement a singleton process that coordinates the task instances.</span></span> <span data-ttu-id="6185c-153">이 프로세스가 실패하거나 비정상 상태가 되면 시스템이 종료하고 다시 시작할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-153">If this process fails or becomes unhealthy, the system can shut it down and restart it.</span></span>
- <span data-ttu-id="6185c-154">더 경량 방법을 사용하여 태스크 간 조정을 수행할 수 있는 경우.</span><span class="sxs-lookup"><span data-stu-id="6185c-154">The coordination between tasks can be achieved using a more lightweight method.</span></span> <span data-ttu-id="6185c-155">예를 들어 단순히 여러 태스크 인스턴스에 공유 리소스에 대한 조정된 액세스가 필요한 경우 더 나은 솔루션은 낙관적 또는 비관적 잠금을 사용하여 액세스를 제어하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-155">For example, if several task instances simply need coordinated access to a shared resource, a better solution is to use optimistic or pessimistic locking to control access.</span></span>
- <span data-ttu-id="6185c-156">타사 솔루션이 더 적합한 경우.</span><span class="sxs-lookup"><span data-stu-id="6185c-156">A third-party solution is more appropriate.</span></span> <span data-ttu-id="6185c-157">예를 들어 Microsoft Azure HDInsight 서비스(Apache Hadoop 기반)는 Apache Zookeeper에서 제공하는 서비스를 사용하여 맵을 조정하고 데이터를 수집 및 요약하는 태스크를 줄입니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-157">For example, the Microsoft Azure HDInsight service (based on Apache Hadoop) uses the services provided by Apache Zookeeper to coordinate the map and reduce tasks that collect and summarize data.</span></span>

## <a name="example"></a><span data-ttu-id="6185c-158">예</span><span class="sxs-lookup"><span data-stu-id="6185c-158">Example</span></span>

<span data-ttu-id="6185c-159">LeaderElection 솔루션의 DistributedMutex 프로젝트([GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)에서 이 패턴을 보여 주는 샘플을 확인할 수 있음)는 Azure Storage Blob의 임대를 사용하여 공유 분산 뮤텍스를 구현하는 메커니즘을 제공하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-159">The DistributedMutex project in the LeaderElection solution (a sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)) shows how to use a lease on an Azure Storage blob to provide a mechanism for implementing a shared, distributed mutex.</span></span> <span data-ttu-id="6185c-160">이 뮤텍스를 사용하여 Azure 클라우드 서비스의 역할 인스턴스 그룹 중에서 리더를 선택할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-160">This mutex can be used to elect a leader among a group of role instances in an Azure cloud service.</span></span> <span data-ttu-id="6185c-161">임대를 획득한 첫 번째 역할 인스턴스가 리더로 선택되고, 임대를 해제하거나 임대를 갱신할 수 없을 때까지 리더로 유지됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-161">The first role instance to acquire the lease is elected the leader, and remains the leader until it releases the lease or isn't able to renew the lease.</span></span> <span data-ttu-id="6185c-162">다른 역할 인스턴스는 리더를 더 이상 사용할 수 없는 경우를 위해 Blob 임대를 계속 모니터할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-162">Other role instances can continue to monitor the blob lease in case the leader is no longer available.</span></span>

>  <span data-ttu-id="6185c-163">Blob 임대는 Blob에 대한 배타적 쓰기 잠금입니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-163">A blob lease is an exclusive write lock over a blob.</span></span> <span data-ttu-id="6185c-164">단일 Blob은 언제든지 한 임대의 주체만 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-164">A single blob can be the subject of only one lease at any point in time.</span></span> <span data-ttu-id="6185c-165">역할 인스턴스가 지정된 Blob에 대한 임대를 요청할 수 있으며, 동일한 Blob에 대한 임대를 보유한 다른 역할 인스턴스가 없는 경우 임대가 부여됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-165">A role instance can request a lease over a specified blob, and it'll be granted the lease if no other role instance holds a lease over the same blob.</span></span> <span data-ttu-id="6185c-166">다른 역할 인스턴스가 임대를 보유한 경우 요청에서 예외가 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-166">Otherwise the request will throw an exception.</span></span>
> 
> <span data-ttu-id="6185c-167">결함 있는 역할 인스턴스가 임대를 무기한 유지하는 것을 방지하려면 임대 수명을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-167">To avoid a faulted role instance retaining the lease indefinitely, specify a lifetime for the lease.</span></span> <span data-ttu-id="6185c-168">이 수명이 만료되면 임대를 사용할 수 있게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-168">When this expires, the lease becomes available.</span></span> <span data-ttu-id="6185c-169">그러나 역할 인스턴스가 임대를 보유하는 동안 임대가 갱신되도록 요청할 수 있으며, 추가 기간 동안 임대가 부여됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-169">However, while a role instance holds the lease it can request that the lease is renewed, and it'll be granted the lease for a further period of time.</span></span> <span data-ttu-id="6185c-170">역할 인스턴스가 임대를 유지하려는 경우 이 프로세스를 계속 반복할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-170">The role instance can continually repeat this process if it wants to retain the lease.</span></span>
> <span data-ttu-id="6185c-171">Blob 임대 방법에 대한 자세한 내용은 [Blob 임대(REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6185c-171">For more information on how to lease a blob, see [Lease Blob (REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx).</span></span>

<span data-ttu-id="6185c-172">아래 C# 예제의 `BlobDistributedMutex` 클래스에는 역할 인스턴스가 지정된 Blob에 대한 임대를 획득하려고 시도할 수 있게 하는 `RunTaskWhenMutexAquired` 메서드가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-172">The `BlobDistributedMutex` class in the C# example below contains the `RunTaskWhenMutexAquired` method that enables a role instance to attempt to acquire a lease over a specified blob.</span></span> <span data-ttu-id="6185c-173">`BlobDistributedMutex` 개체를 만들 때(이 개체는 샘플 코드에 포함된 단순 구조체임) Blob의 세부 정보(이름, 컨테이너, 저장소 계정)가 `BlobSettings` 개체의 생성자에게 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-173">The details of the blob (the name, container, and storage account) are passed to the constructor in a `BlobSettings` object when the `BlobDistributedMutex` object is created (this object is a simple struct that is included in the sample code).</span></span> <span data-ttu-id="6185c-174">또한 생성자는 Blob에 대한 임대를 획득하고 리더로 선택된 경우 역할 인스턴스가 실행해야 하는 코드를 참조하는 `Task`를 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-174">The constructor also accepts a `Task` that references the code that the role instance should run if it successfully acquires the lease over the blob and is elected the leader.</span></span> <span data-ttu-id="6185c-175">임대 획득의 하위 수준 세부 정보를 처리하는 코드는 `BlobLeaseManager`라는 별도의 도우미 클래스에서 구현됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-175">Note that the code that handles the low-level details of acquiring the lease is implemented in a separate helper class named `BlobLeaseManager`.</span></span>

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

<span data-ttu-id="6185c-176">위 코드 샘플의 `RunTaskWhenMutexAquired` 메서드는 실제로 임대를 획득하기 위해 다음 코드 샘플에 표시된 `RunTaskWhenBlobLeaseAcquired` 메서드를 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-176">The `RunTaskWhenMutexAquired` method in the code sample above invokes the `RunTaskWhenBlobLeaseAcquired` method shown in the following code sample to actually acquire the lease.</span></span> <span data-ttu-id="6185c-177">`RunTaskWhenBlobLeaseAcquired` 메서드는 비동기적으로 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-177">The `RunTaskWhenBlobLeaseAcquired` method runs asynchronously.</span></span> <span data-ttu-id="6185c-178">임대를 획득한 경우 역할 인스턴스가 리더로 선택된 것입니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-178">If the lease is successfully acquired, the role instance has been elected the leader.</span></span> <span data-ttu-id="6185c-179">`taskToRunWhenLeaseAcquired` 대리자의 목적은 다른 역할 인스턴스를 조정하는 작업을 수행하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-179">The purpose of the `taskToRunWhenLeaseAcquired` delegate is to perform the work that coordinates the other role instances.</span></span> <span data-ttu-id="6185c-180">임대를 획득하지 못한 경우 다른 역할 인스턴스가 리더로 선택된 것이며 현재 역할 인스턴스가 하위로 유지됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-180">If the lease isn't acquired, another role instance has been elected as the leader and the current role instance remains a subordinate.</span></span> <span data-ttu-id="6185c-181">`TryAcquireLeaseOrWait` 메서드는 `BlobLeaseManager` 개체를 사용하여 임대를 획득하는 도우미 메서드입니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-181">Note that the `TryAcquireLeaseOrWait` method is a helper method that uses the `BlobLeaseManager` object to acquire the lease.</span></span>

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

<span data-ttu-id="6185c-182">리더가 시작한 태스크도 비동기적으로 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-182">The task started by the leader also runs asynchronously.</span></span> <span data-ttu-id="6185c-183">이 태스크가 실행되는 동안 다음 코드 샘플에 표시된 `RunTaskWhenBlobLeaseAquired` 메서드는 정기적으로 임대를 갱신하려고 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-183">While this task is running, the `RunTaskWhenBlobLeaseAquired` method shown in the following code sample periodically attempts to renew the lease.</span></span> <span data-ttu-id="6185c-184">이렇게 하면 역할 인스턴스가 리더로 유지됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-184">This helps to ensure that the role instance remains the leader.</span></span> <span data-ttu-id="6185c-185">샘플 솔루션에서는 다른 역할 인스턴스가 리더로 선택되지 않도록 하기 위해 갱신 요청 사이의 지연이 임대 기간으로 지정된 시간보다 짧습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-185">In the sample solution, the delay between renewal requests is less than the time specified for the duration of the lease in order to prevent another role instance from being elected the leader.</span></span> <span data-ttu-id="6185c-186">어떤 이유로 인해 갱신에 실패하면 태스크가 취소됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-186">If the renewal fails for any reason, the task is canceled.</span></span>

<span data-ttu-id="6185c-187">임대가 갱신되지 않거나 태스크가 취소되면(역할 인스턴스 종료로 인해) 임대가 해제됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-187">If the lease fails to be renewed or the task is canceled (possibly as a result of the role instance shutting down), the lease is released.</span></span> <span data-ttu-id="6185c-188">이 시점에서 이 역할 인스턴스나 다른 역할 인스턴스가 리더로 선택될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-188">At this point, this or another role instance might be elected as the leader.</span></span> <span data-ttu-id="6185c-189">아래 추출된 코드는 이 프로세스 부분을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-189">The code extract below shows this part of the process.</span></span>

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

<span data-ttu-id="6185c-190">`KeepRenewingLease` 메서드는 `BlobLeaseManager` 개체를 사용하여 임대를 갱신하는 다른 도우미 메서드입니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-190">The `KeepRenewingLease` method is another helper method that uses the `BlobLeaseManager` object to renew the lease.</span></span> <span data-ttu-id="6185c-191">`CancelAllWhenAnyCompletes` 메서드는 처음 두 매개 변수로 지정된 태스크를 취소합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-191">The `CancelAllWhenAnyCompletes` method cancels the tasks specified as the first two parameters.</span></span> <span data-ttu-id="6185c-192">다음 다이어그램은 `BlobDistributedMutex` 클래스를 사용하여 리더를 선택하고 작업을 조정하는 태스크를 실행하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-192">The following diagram illustrates using the `BlobDistributedMutex` class to elect a leader and run a task that coordinates operations.</span></span>

![그림 1은 BlobDistributedMutex 클래스의 함수를 보여 줍니다.](./_images/leader-election-diagram.png)


<span data-ttu-id="6185c-194">다음 코드 예제는 작업자 역할에 `BlobDistributedMutex` 클래스를 사용하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-194">The following code example shows how to use the `BlobDistributedMutex` class in a worker role.</span></span> <span data-ttu-id="6185c-195">이 코드는 개발 저장소의 임대 컨테이너에 있는 `MyLeaderCoordinatorTask` Blob에 대한 임대를 획득하고, 역할 인스턴스가 리더로 선택된 경우 `MyLeaderCoordinatorTask` 메서드에 정의된 코드가 실행되도록 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-195">This code acquires a lease over a blob named `MyLeaderCoordinatorTask` in the lease's container in development storage, and specifies that the code defined in the `MyLeaderCoordinatorTask` method should run if the role instance is elected the leader.</span></span>

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

<span data-ttu-id="6185c-196">샘플 솔루션에 대한 다음 사항에 유의하세요.</span><span class="sxs-lookup"><span data-stu-id="6185c-196">Note the following points about the sample solution:</span></span>
- <span data-ttu-id="6185c-197">Blob은 잠재적인 단일 실패 지점입니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-197">The blob is a potential single point of failure.</span></span> <span data-ttu-id="6185c-198">Blob 서비스를 사용할 수 없거나 액세스할 수 없는 경우 리더가 임대를 갱신할 수 없으며 다른 역할 인스턴스도 임대를 획득할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-198">If the blob service becomes unavailable, or is inaccessible, the leader won't be able to renew the lease and no other role instance will be able to acquire the lease.</span></span> <span data-ttu-id="6185c-199">이 경우 리더 역할을 할 수 있는 역할 인스턴스가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-199">In this case, no role instance will be able to act as the leader.</span></span> <span data-ttu-id="6185c-200">그러나 Blob 서비스는 복원력이 있도록 디자인되었으므로 Blob 서비스가 완전히 실패할 가능성은 거의 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-200">However, the blob service is designed to be resilient, so complete failure of the blob service is considered to be extremely unlikely.</span></span>
- <span data-ttu-id="6185c-201">리더가 수행 중인 태스크가 중단되면 리더가 임대를 계속 갱신하여 다른 역할 인스턴스가 태스크를 조정하기 위해 임대를 획득하고 리더 역할을 맡지 못하도록 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-201">If the task being performed by the leader stalls, the leader might continue to renew the lease, preventing any other role instance from acquiring the lease and taking over the leader role in order to coordinate tasks.</span></span> <span data-ttu-id="6185c-202">프로덕션에서는 리더의 상태를 자주 확인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-202">In the real world, the health of the leader should be checked at frequent intervals.</span></span>
- <span data-ttu-id="6185c-203">선택 프로세스는 비결정적입니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-203">The election process is nondeterministic.</span></span> <span data-ttu-id="6185c-204">어떤 역할 인스턴스가 Blob 임대를 획득하고 리더가 될지 가정할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-204">You can't make any assumptions about which role instance will acquire the blob lease and become the leader.</span></span>
- <span data-ttu-id="6185c-205">Blob 임대의 대상으로 사용된 Blob은 다른 용도로 사용하면 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-205">The blob used as the target of the blob lease shouldn't be used for any other purpose.</span></span> <span data-ttu-id="6185c-206">역할 인스턴스가 이 Blob에 데이터를 저장하려고 시도할 경우 역할 인스턴스가 리더이고 Blob 임대를 보유한 경우가 아니면 이 데이터에 액세스할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-206">If a role instance attempts to store data in this blob, this data won't be accessible unless the role instance is the leader and holds the blob lease.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="6185c-207">관련 패턴 및 지침</span><span class="sxs-lookup"><span data-stu-id="6185c-207">Related patterns and guidance</span></span>

<span data-ttu-id="6185c-208">이 패턴을 구현할 때 다음 지침도 관련이 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-208">The following guidance might also be relevant when implementing this pattern:</span></span>
- <span data-ttu-id="6185c-209">이 패턴에는 다운로드 가능한 [샘플 응용 프로그램](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election)이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-209">This pattern has a downloadable [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/leader-election).</span></span>
- <span data-ttu-id="6185c-210">[자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx).</span><span class="sxs-lookup"><span data-stu-id="6185c-210">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="6185c-211">응용 프로그램의 부하가 변경됨에 따라 태스크 호스트 인스턴스를 시작 및 중지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-211">It's possible to start and stop instances of the task hosts as the load on the application varies.</span></span> <span data-ttu-id="6185c-212">자동 크기 조정을 사용하면 최고 처리 시간 동안 처리량과 성능을 유지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-212">Autoscaling can help to maintain throughput and performance during times of peak processing.</span></span>
- <span data-ttu-id="6185c-213">[Compute 분할 지침](https://msdn.microsoft.com/library/dn589773.aspx)</span><span class="sxs-lookup"><span data-stu-id="6185c-213">[Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx).</span></span> <span data-ttu-id="6185c-214">이 지침은 서비스의 확장성, 성능, 가용성 및 보안을 유지하면서 실행 비용을 최소화하는 데 도움이 되는 방식으로 클라우드 서비스의 호스트에 태스크를 할당하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="6185c-214">This guidance describes how to allocate tasks to hosts in a cloud service in a way that helps to minimize running costs while maintaining the scalability, performance, availability, and security of the service.</span></span>
- <span data-ttu-id="6185c-215">[태스크 기반 비동기 패턴](https://msdn.microsoft.com/library/hh873175.aspx).</span><span class="sxs-lookup"><span data-stu-id="6185c-215">The [Task-based Asynchronous Pattern](https://msdn.microsoft.com/library/hh873175.aspx).</span></span>
- <span data-ttu-id="6185c-216">[Bully 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html)을 보여 주는 예제.</span><span class="sxs-lookup"><span data-stu-id="6185c-216">An example illustrating the [Bully Algorithm](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/BullyExample.html).</span></span>
- <span data-ttu-id="6185c-217">[Ring 알고리즘](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html)을 보여 주는 예제.</span><span class="sxs-lookup"><span data-stu-id="6185c-217">An example illustrating the [Ring Algorithm](http://www.cs.colostate.edu/~cs551/CourseNotes/Synchronization/RingElectExample.html).</span></span>
- <span data-ttu-id="6185c-218">Microsoft Open Technologies 웹 사이트에 있는 [Microsoft Azure의 Apache Zookeeper](https://msopentech.com/opentech-projects/apache-zookeeper-on-windows-azure-2/) 문서.</span><span class="sxs-lookup"><span data-stu-id="6185c-218">The article [Apache Zookeeper on Microsoft Azure](https://msopentech.com/opentech-projects/apache-zookeeper-on-windows-azure-2/) on the Microsoft Open Technologies website.</span></span>
- <span data-ttu-id="6185c-219">Apache ZooKeeper용 클라이언트 라이브러리인 [Apache Curator](http://curator.apache.org/).</span><span class="sxs-lookup"><span data-stu-id="6185c-219">[Apache Curator](http://curator.apache.org/) a client library for Apache ZooKeeper.</span></span>
- <span data-ttu-id="6185c-220">MSDN에 있는 [Blob 임대(REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx) 문서.</span><span class="sxs-lookup"><span data-stu-id="6185c-220">The article [Lease Blob (REST API)](https://msdn.microsoft.com/library/azure/ee691972.aspx) on MSDN.</span></span>
