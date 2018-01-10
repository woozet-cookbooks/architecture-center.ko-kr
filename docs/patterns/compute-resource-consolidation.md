---
title: "계산 리소스 통합"
description: "여러 작업을 단일 계산 단위로 통합합니다."
keywords: "디자인 패턴"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: design-implementation
ms.openlocfilehash: 85191fc630549559f8a1395e5a8622a7a6140a2d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="compute-resource-consolidation-pattern"></a><span data-ttu-id="c6eee-104">계산 리소스 통합 패턴</span><span class="sxs-lookup"><span data-stu-id="c6eee-104">Compute Resource Consolidation pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="c6eee-105">여러 작업을 단일 계산 단위로 통합합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-105">Consolidate multiple tasks or operations into a single computational unit.</span></span> <span data-ttu-id="c6eee-106">따라서 계산 리소스 사용률을 높이고 클라우드 호스팅 응용 프로그램에서 계산 처리 수행에 관련된 비용과 관리 오버헤드를 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-106">This can increase compute resource utilization, and reduce the costs and management overhead associated with performing compute processing in cloud-hosted applications.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c6eee-107">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="c6eee-107">Context and problem</span></span>

<span data-ttu-id="c6eee-108">클라우드 응용 프로그램은 다양한 작업을 구현하는 경우가 많습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-108">A cloud application often implements a variety of operations.</span></span> <span data-ttu-id="c6eee-109">일부 솔루션에서는 처음부터 문제를 분리하는 디자인 원칙을 따르기도 하고 이런 작업을 개별적으로 호스트하고 배포하는 별도의 계산 단위(예: 별도의 App Service Web Apps, 별도의 Virtual Machines, 별도의 Cloud Service 역할)로 분할하기도 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-109">In some solutions it makes sense to follow the design principle of separation of concerns initially, and divide these operations into separate computational units that are hosted and deployed individually (for example, as separate App Service web apps, separate Virtual Machines, or separate Cloud Service roles).</span></span> <span data-ttu-id="c6eee-110">그러나 이 전략이 솔루션의 논리적 디자인 단순화에 도움이 되더라도 동일한 응용 프로그램의 일부로 많은 계산 단위를 배포하면 런타임 호스팅 비용이 증가하고 시스템의 관리가 더 복잡해질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-110">However, although this strategy can help simplify the logical design of the solution, deploying a large number of computational units as part of the same application can increase runtime hosting costs and make management of the system more complex.</span></span>

<span data-ttu-id="c6eee-111">다음 그림에 제시된 예제는 둘 이상의 계산 단위를 사용해 구현된 클라우드 호스팅 솔루션의 단순화된 구조를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-111">As an example, the figure shows the simplified structure of a cloud-hosted solution that is implemented using more than one computational unit.</span></span> <span data-ttu-id="c6eee-112">각 계산 단위는 자체 가상 환경에서 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-112">Each computational unit runs in its own virtual environment.</span></span> <span data-ttu-id="c6eee-113">각 기능은 자체 계산 단위에서 실행되는 별도의 작업(작업 A~작업 E)으로 구현되었습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-113">Each function has been implemented as a separate task (labeled Task A through Task E) running in its own computational unit.</span></span>

![전용 계산 단위 집합을 사용하여 클라우드 환경에서 작업 실행](./_images/compute-resource-consolidation-diagram.png)


<span data-ttu-id="c6eee-115">각 계산 단위는 사용하지 않거나 사용 시간이 적더라도 비용이 부과되는 리소스를 소모합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-115">Each computational unit consumes chargeable resources, even when it's idle or lightly used.</span></span> <span data-ttu-id="c6eee-116">따라서 이 솔루션이 항상 가장 경제적인 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-116">Therefore, this isn't always the most cost-effective solution.</span></span>

<span data-ttu-id="c6eee-117">Azure에서 이 문제는 자체 가상 환경에서 실행되는 Cloud Service, App Services, Virtual Machines에서의</span><span class="sxs-lookup"><span data-stu-id="c6eee-117">In Azure, this concern applies to roles in a Cloud Service, App Services, and Virtual Machines.</span></span> <span data-ttu-id="c6eee-118">역할에 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-118">These items run in their own virtual environment.</span></span> <span data-ttu-id="c6eee-119">잘 정의된 작업의 집합을 수행하도록 디자인되었지만 단일 솔루션의 일부로 통신하고 상호 작용해야 하는 별도의 역할, 웹 사이트 또는 가상 머신의 모음을 실행하면 리소스 사용이 비효율적일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-119">Running a collection of separate roles, websites, or virtual machines that are designed to perform a set of well-defined operations, but that need to communicate and cooperate as part of a single solution, can be an inefficient use of resources.</span></span>

## <a name="solution"></a><span data-ttu-id="c6eee-120">해결 방법</span><span class="sxs-lookup"><span data-stu-id="c6eee-120">Solution</span></span>

<span data-ttu-id="c6eee-121">여러 작업 또는 연산을 하나의 계산 단위에 통합하면 비용을 줄이고, 사용률을 높이며, 통신 속도를 개선하고, 관리를 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-121">To help reduce costs, increase utilization, improve communication speed, and reduce management it's possible to consolidate multiple tasks or operations into a single computational unit.</span></span>

<span data-ttu-id="c6eee-122">작업은 환경에 제공된 기능과 이러한 기능과 관련된 비용을 기준으로 그룹화할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-122">Tasks can be grouped according to criteria based on the features provided by the environment and the costs associated with these features.</span></span> <span data-ttu-id="c6eee-123">일반적인 접근 방식은 확장성, 수명, 처리 요구 사항이 비슷한 프로필을 갖는 작업을 찾는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-123">A common approach is to look for tasks that have a similar profile concerning their scalability, lifetime, and processing requirements.</span></span> <span data-ttu-id="c6eee-124">이러한 작업을 함께 그룹화하면 하나의 단위로 크기를 조정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-124">Grouping these together allows them to scale as a unit.</span></span> <span data-ttu-id="c6eee-125">많은 클라우드 환경에서 제공하는 탄력성을 통해 계산 단위의 추가 인스턴스를 워크로드에 따라 시작 및 중지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-125">The elasticity provided by many cloud environments enables additional instances of a computational unit to be started and stopped according to the workload.</span></span> <span data-ttu-id="c6eee-126">예를 들어 Azure는 Cloud Service, App Services, Virtual Machines에서의 역할에 적용할 수 있는 자동 크기 조정을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-126">For example, Azure provides autoscaling that you can apply to roles in a Cloud Service, App Services, and Virtual Machines.</span></span> <span data-ttu-id="c6eee-127">자세한 내용은 [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c6eee-127">For more information, see [Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span>

<span data-ttu-id="c6eee-128">어떤 작업을 함께 그룹화하지 않아야 하는지 결정하는 데 확장성을 사용할 수 있는 방법을 보여 주는 상반된 예제로 다음 두 가지 작업을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-128">As a counter example to show how scalability can be used to determine which operations shouldn't be grouped together, consider the following two tasks:</span></span>

- <span data-ttu-id="c6eee-129">작업 1은 큐에 전송되는, 자주 사용되지 않고 시간에 민감하지 않은 메시지를 폴링합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-129">Task 1 polls for infrequent, time-insensitive messages sent to a queue.</span></span>
- <span data-ttu-id="c6eee-130">작업 2는 네트워크 트래픽의 대규모 버스트를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-130">Task 2 handles high-volume bursts of network traffic.</span></span>

<span data-ttu-id="c6eee-131">두 번째 작업에는 계산 단위의 수많은 인스턴스를 시작 및 중지할 수 있는 탄력성이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-131">The second task requires elasticity that can involve starting and stopping a large number of instances of the computational unit.</span></span> <span data-ttu-id="c6eee-132">이와 동일한 크기 조정을 첫 번째 작업에 적용하면 동일한 큐에서 자주 발생하지 않는 메시지를 수신하는 작업이 더 많아져 리소스를 낭비하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-132">Applying the same scaling to the first task would simply result in more tasks listening for infrequent messages on the same queue, and is a waste of resources.</span></span>

<span data-ttu-id="c6eee-133">많은 클라우드 환경에서는 CPU 코어 개수, 메모리, 디스크 공간 등에 따라 계산 단위에 사용할 수 있는 리소스를 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-133">In many cloud environments it's possible to specify the resources available to a computational unit in terms of the number of CPU cores, memory, disk space, and so on.</span></span> <span data-ttu-id="c6eee-134">일반적으로 지정하는 리소스 수가 많을수록 비용이 늘어납니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-134">Generally, the more resources specified, the greater the cost.</span></span> <span data-ttu-id="c6eee-135">비용을 절감하려면 비싼 계산 단위가 수행하는 작업을 최대화하고 사용되지 않는 상태로 장시간 있지 않도록 조치하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-135">To save money, it's important to maximize the work an expensive computational unit performs, and not let it become inactive for an extended period.</span></span>

<span data-ttu-id="c6eee-136">짧은 버스트에서 많은 CPU 성능을 요구하는 작업이 있는 경우 필요한 성능을 제공하는 단일 계산 단위로 이러한 작업을 통합할 것을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-136">If there are tasks that require a great deal of CPU power in short bursts, consider consolidating these into a single computational unit that provides the necessary power.</span></span> <span data-ttu-id="c6eee-137">그러나 스트레스를 과도하게 받을 경우 발생할 수 있는 경합을 방지하는 동시에 값비싼 리소스를 계속 사용하도록 균형을 유지하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-137">However, it's important to balance this need to keep expensive resources busy against the contention that could occur if they are over stressed.</span></span> <span data-ttu-id="c6eee-138">예를 들어 장기 실행되는 계산 집약적 작업은 동일한 계산 단위를 공유하지 않아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-138">Long-running, compute-intensive tasks shouldn't share the same computational unit, for example.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="c6eee-139">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="c6eee-139">Issues and considerations</span></span>

<span data-ttu-id="c6eee-140">이 패턴을 구현할 때는 다음 사항을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-140">Consider the following points when implementing this pattern:</span></span>

<span data-ttu-id="c6eee-141">**확장성 및 탄력성**.</span><span class="sxs-lookup"><span data-stu-id="c6eee-141">**Scalability and elasticity**.</span></span> <span data-ttu-id="c6eee-142">많은 클라우드 솔루션은 계산 단위의 인스턴스를 시작 및 중지해 계산 단위 수준에서 확장성과 탄력성을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-142">Many cloud solutions implement scalability and elasticity at the level of the computational unit by starting and stopping instances of units.</span></span> <span data-ttu-id="c6eee-143">동일한 계산 단위에서 확장성 요구 사항이 충돌하는 작업을 그룹화해서는 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-143">Avoid grouping tasks that have conflicting scalability requirements in the same computational unit.</span></span>

<span data-ttu-id="c6eee-144">**수명**.</span><span class="sxs-lookup"><span data-stu-id="c6eee-144">**Lifetime**.</span></span> <span data-ttu-id="c6eee-145">클라우드 인프라는 계산 단위를 호스트하는 가상 환경을 주기적으로 재활용합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-145">The cloud infrastructure periodically recycles the virtual environment that hosts a computational unit.</span></span> <span data-ttu-id="c6eee-146">따라서 계산 단위 내에 장기 실행되는 작업이 많이 있으면 해당 작업이 완료될 때까지 계산 단위를 재활용하지 않도록 구성할 필요가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-146">When there are many long-running tasks inside a computational unit, it might be necessary to configure the unit to prevent it from being recycled until these tasks have finished.</span></span> <span data-ttu-id="c6eee-147">또는 검사점 접근을 사용해 작업을 완전히 중단하고 계산 단위를 다시 시작할 때는 중단된 지점에서 계속될 수 있도록 작업을 디자인합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-147">Alternatively, design the tasks by using a check-pointing approach that enables them to stop cleanly, and continue at the point they were interrupted when the computational unit is restarted.</span></span>

<span data-ttu-id="c6eee-148">**릴리스 주기**.</span><span class="sxs-lookup"><span data-stu-id="c6eee-148">**Release cadence**.</span></span> <span data-ttu-id="c6eee-149">작업의 구현 또는 구성을 자주 변경하는 경우 업데이트된 코드를 호스팅하는 계산 단위를 중지하고, 다시 구성하여 다시 배포한 다음, 다시 시작할 필요가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-149">If the implementation or configuration of a task changes frequently, it might be necessary to stop the computational unit hosting the updated code, reconfigure and redeploy the unit, and then restart it.</span></span> <span data-ttu-id="c6eee-150">이 프로세스에서는 동일한 계산 단위 내의 다른 모든 작업이 중지, 다시 배포 및 다시 시작되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-150">This process will also require that all other tasks within the same computational unit are stopped, redeployed, and restarted.</span></span>

<span data-ttu-id="c6eee-151">**보안**.</span><span class="sxs-lookup"><span data-stu-id="c6eee-151">**Security**.</span></span> <span data-ttu-id="c6eee-152">동일한 계산 단위 내의 작업은 동일한 보안 컨텍스트를 공유하고 동일한 리소스에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-152">Tasks in the same computational unit might share the same security context and be able to access the same resources.</span></span> <span data-ttu-id="c6eee-153">작업 간에는 신뢰도가 높아야 하고 하나의 작업이 다른 작업에 손상을 입히거나 악영향을 미치지 않는다는 확신이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-153">There must be a high degree of trust between the tasks, and confidence that one task isn't going to corrupt or adversely affect another.</span></span> <span data-ttu-id="c6eee-154">한편 계산 단위에서 실행되는 작업 수가 늘어나면 단위의 공격 표면이 늘어납니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-154">Additionally, increasing the number of tasks running in a computational unit increases the attack surface of the unit.</span></span> <span data-ttu-id="c6eee-155">가장 많은 취약성이 있는 작업이 안전해야 각 작업이 안전합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-155">Each task is only as secure as the one with the most vulnerabilities.</span></span>

<span data-ttu-id="c6eee-156">**내결함성**.</span><span class="sxs-lookup"><span data-stu-id="c6eee-156">**Fault tolerance**.</span></span> <span data-ttu-id="c6eee-157">계산 단위 내의 한 작업이 실패하거나 비정상적으로 동작하면 동일한 계산 내에서 실행되는 다른 작업에 영향을 미칠 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-157">If one task in a computational unit fails or behaves abnormally, it can affect the other tasks running within the same unit.</span></span> <span data-ttu-id="c6eee-158">예를 들어 올바르게 시작하는 데 실패한 작업은 전체 계산 단위의 시작 논리 실패를 초래하여 동일한 단위의 다른 작업 실행에 지장을 줄 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-158">For example, if one task fails to start correctly it can cause the entire startup logic for the computational unit to fail, and prevent other tasks in the same unit from running.</span></span>

<span data-ttu-id="c6eee-159">**경합**.</span><span class="sxs-lookup"><span data-stu-id="c6eee-159">**Contention**.</span></span> <span data-ttu-id="c6eee-160">동일한 계산 단위에서 리소스를 두고 경쟁하는 작업 간에 경합하지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-160">Avoid introducing contention between tasks that compete for resources in the same computational unit.</span></span> <span data-ttu-id="c6eee-161">동일한 계산 단위를 공유하는 작업이 서로 다른 리소스 사용률 특징을 나타내는 것이 가장 이상적입니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-161">Ideally, tasks that share the same computational unit should exhibit different resource utilization characteristics.</span></span> <span data-ttu-id="c6eee-162">예를 들어 2개의 계산 집약적 작업이 동일한 계산 단위 내에 있으면 안 되고 메모리를 많이 사용하는 2개의 작업도 동일한 계산 단위에 있으면 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-162">For example, two compute-intensive tasks should probably not reside in the same computational unit, and neither should two tasks that consume large amounts of memory.</span></span> <span data-ttu-id="c6eee-163">그러나 계산 집약적 작업과 메모리가 많이 필요한 작업은 혼합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-163">However, mixing a compute intensive task with a task that requires a large amount of memory is a workable combination.</span></span>

> [!NOTE]
>  <span data-ttu-id="c6eee-164">계산 리소스 통합은 일정 기간 동안 프로덕션에 존재해온 시스템에서만 고려합니다. 그럼으로써 운영자와 개발자는 시스템을 모니터링하고 각 작업이 다양한 리소스를 어떻게 사용하는지를 식별하는 _열 지도_를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-164">Consider consolidating compute resources only for a system that's been in production for a period of time so that operators and developers can monitor the system and create a _heat map_ that identifies how each task utilizes differing resources.</span></span> <span data-ttu-id="c6eee-165">이 지도는 계산 리소스를 공유하기 적합한 작업을 결정하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-165">This map can be used to determine which tasks are good candidates for sharing compute resources.</span></span>

<span data-ttu-id="c6eee-166">**복잡성**.</span><span class="sxs-lookup"><span data-stu-id="c6eee-166">**Complexity**.</span></span> <span data-ttu-id="c6eee-167">단일 계산 단위에 여러 작업을 조합하면 단위 코드에 복잡성이 추가되어 테스트, 디버그 및 유지 관리가 더 어려워질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-167">Combining multiple tasks into a single computational unit adds complexity to the code in the unit, possibly making it more difficult to test, debug, and maintain.</span></span>

<span data-ttu-id="c6eee-168">**안정적인 논리 아키텍처**.</span><span class="sxs-lookup"><span data-stu-id="c6eee-168">**Stable logical architecture**.</span></span> <span data-ttu-id="c6eee-169">작업의 코드를 각각 디자인하고 구현하여 작업을 실행하는 물리적 환경이 변하더라도 작업을 변경할 필요가 없도록 조치합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-169">Design and implement the code in each task so that it shouldn't need to change, even if the physical environment the task runs in does change.</span></span>

<span data-ttu-id="c6eee-170">**기타 전략**.</span><span class="sxs-lookup"><span data-stu-id="c6eee-170">**Other strategies**.</span></span> <span data-ttu-id="c6eee-171">여러 작업의 동시 실행에 관련된 비용을 줄이는 유일한 방법인 계산 리소스 통합이 효과적인 접근 방식으로 유지되려면</span><span class="sxs-lookup"><span data-stu-id="c6eee-171">Consolidating compute resources is only one way to help reduce costs associated with running multiple tasks concurrently.</span></span> <span data-ttu-id="c6eee-172">신중한 계획 수립과 모니터링이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-172">It requires careful planning and monitoring to ensure that it remains an effective approach.</span></span> <span data-ttu-id="c6eee-173">작업의 특성 및 작업을 실행하는 사용자가 있는 장소에 따라 다른 전략이 더 적절할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-173">Other strategies might be more appropriate, depending on the nature of the work and where the users these tasks are running are located.</span></span> <span data-ttu-id="c6eee-174">예를 들어 [계산 분할 지침](https://msdn.microsoft.com/library/dn589773.aspx)에서 설명하는 워크로드의 기능적 분해가 더 나은 옵션일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-174">For example, functional decomposition of the workload (as described by the [Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx)) might be a better option.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c6eee-175">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="c6eee-175">When to use this pattern</span></span>

<span data-ttu-id="c6eee-176">이 패턴은 자체 계산 단위에서 실행할 경우 경제적이지 않은 작업에 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-176">Use this pattern for tasks that are not cost effective if they run in their own computational units.</span></span> <span data-ttu-id="c6eee-177">거의 대부분의 시간 동안 사용되지 않는 작업을 전용 단위에서 실행하면 비용이 많이 소요될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-177">If a task spends much of its time idle, running this task in a dedicated unit can be expensive.</span></span>

<span data-ttu-id="c6eee-178">이 패턴은 중요한 내결함성 작업을 수행하는 작업 또는 매우 민감하거나 개인적인 데이터를 처리하고 자체 보안 컨텍스트가 필요한 작업에는 적합하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-178">This pattern might not be suitable for tasks that perform critical fault-tolerant operations, or tasks that process highly sensitive or private data and require their own security context.</span></span> <span data-ttu-id="c6eee-179">이러한 작업은 자체 격리된 환경의 별도 계산 단위에서 실행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-179">These tasks should run in their own isolated environment, in a separate computational unit.</span></span>

## <a name="example"></a><span data-ttu-id="c6eee-180">예</span><span class="sxs-lookup"><span data-stu-id="c6eee-180">Example</span></span>

<span data-ttu-id="c6eee-181">Azure에서 클라우드 서비스를 빌드할 때는 여러 작업이 수행하는 처리를 단일 역할에 통합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-181">When building a cloud service on Azure, it’s possible to consolidate the processing performed by multiple tasks into a single role.</span></span> <span data-ttu-id="c6eee-182">일반적으로 이 역할은 백그라운드 또는 비동기 처리 작업을 수행하는 작업자 역할입니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-182">Typically this is a worker role that performs background or asynchronous processing tasks.</span></span>

> <span data-ttu-id="c6eee-183">일부 경우 백그라운드 또는 비동기 처리 작업을 웹 역할에 포함시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-183">In some cases it's possible to include background or asynchronous processing tasks in the web role.</span></span> <span data-ttu-id="c6eee-184">이 기술이 웹 역할이 제공하는 공용 인터페이스의 확장성과 응답성에 영향을 미칠 수 있지만, 비용을 줄이고 배포를 단순화하는 데에는 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-184">This technique helps to reduce costs and simplify deployment, although it can impact the scalability and responsiveness of the public-facing interface provided by the web role.</span></span> <span data-ttu-id="c6eee-185">[Combining Multiple Azure Worker Roles into an Azure Web Role](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html)(여러 Azure 작업자 역할을 Azure 웹 역할에 조합) 문서에서는 웹 역할에서 백그라운드 또는 비동기 처리 작업 구현에 관해 자세하게 설명하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-185">The article [Combining Multiple Azure Worker Roles into an Azure Web Role](http://www.31a2ba2a-b718-11dc-8314-0800200c9a66.com/2012/02/combining-multiple-azure-worker-roles.html) contains a detailed description of implementing background or asynchronous processing tasks in a web role.</span></span>

<span data-ttu-id="c6eee-186">이 역할이 작업의 시작 및 중지를 담당합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-186">The role is responsible for starting and stopping the tasks.</span></span> <span data-ttu-id="c6eee-187">Azure 패브릭 컨트롤러가 역할을 로드하면 역할에 대한 `Start` 이벤트가 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-187">When the Azure fabric controller loads a role, it raises the `Start` event for the role.</span></span> <span data-ttu-id="c6eee-188">`WebRole` 또는 `WorkerRole` 클래스의 `OnStart` 메서드를 재정의하여 이 이벤트를 처리하고 이 메서드가 의존하는 작업에 필요한 데이터와 다른 리소스를 시작할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-188">You can override the `OnStart` method of the `WebRole` or `WorkerRole` class to handle this event, perhaps to initialize the data and other resources the tasks in this method depend on.</span></span>

<span data-ttu-id="c6eee-189">`OnStart ` 메서드가 완료되면 역할은 요청에 대한 응답을 시작할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-189">When the `OnStart `method completes, the role can start responding to requests.</span></span> <span data-ttu-id="c6eee-190">역할에서 `OnStart` 및 `Run` 메서드의 사용에 대한 자세한 내용과 지침은 패턴 및 사례 가이드인 [Moving Applications to the Cloud](https://msdn.microsoft.com/library/ff728592.aspx)(응용 프로그램에서 클라우드로 이동)의 [Application Startup Processes](https://msdn.microsoft.com/library/ff803371.aspx#sec16)(응용 프로그램 시작 프로세스) 섹션에서 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-190">You can find more information and guidance about using the `OnStart` and `Run` methods in a role in the [Application Startup Processes](https://msdn.microsoft.com/library/ff803371.aspx#sec16) section in the patterns & practices guide [Moving Applications to the Cloud](https://msdn.microsoft.com/library/ff728592.aspx).</span></span>

> <span data-ttu-id="c6eee-191">`OnStart` 메서드의 코드는 최대한 간결하게 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-191">Keep the code in the `OnStart` method as concise as possible.</span></span> <span data-ttu-id="c6eee-192">Azure는 이 메서드를 완료하는 데 소요되는 시간에 제한을 두지 않지만, 이 메서드가 완료되기 전까지는 역할이 전송된 네트워크 요청에 대해 응답할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-192">Azure doesn't impose any limit on the time taken for this method to complete, but the role won't be able to start responding to network requests sent to it until this method completes.</span></span>

<span data-ttu-id="c6eee-193">`OnStart` 메서드가 완료되면 역할은 `Run` 메서드를 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-193">When the `OnStart` method has finished, the role executes the `Run` method.</span></span> <span data-ttu-id="c6eee-194">이때 패브릭 컨트롤러는 역할에 요청 전송을 시작할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-194">At this point, the fabric controller can start sending requests to the role.</span></span>

<span data-ttu-id="c6eee-195">실제로 작업을 만드는 코드를 `Run` 메서드에 삽입합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-195">Place the code that actually creates the tasks in the `Run` method.</span></span> <span data-ttu-id="c6eee-196">`Run` 메서드는 역할 인스턴스의 수명을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-196">Note that the `Run` method defines the lifetime of the role instance.</span></span> <span data-ttu-id="c6eee-197">이 메서드가 완료되면 패브릭 컨트롤러는 종료할 역할을 정렬합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-197">When this method completes, the fabric controller will arrange for the role to be shut down.</span></span>

<span data-ttu-id="c6eee-198">역할이 종료되거나 재활용되면 패브릭 컨트롤러는 부하 분산 장치에서 추가로 들어오는 요청의 수신을 차단하고 `Stop` 이벤트를 발생시킵니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-198">When a role shuts down or is recycled, the fabric controller prevents any more incoming requests being received from the load balancer and raises the `Stop` event.</span></span> <span data-ttu-id="c6eee-199">역할의 `OnStop` 메서드를 재정의해 이 이벤트를 캡처하고 역할을 종료하기 전에 필요한 정리를 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-199">You can capture this event by overriding the `OnStop` method of the role and perform any tidying up required before the role terminates.</span></span>

> <span data-ttu-id="c6eee-200">`OnStop` 메서드 내에서 수행되는 모든 동작은 5분(또는 로컬 컴퓨터에서 Azure 에뮬레이터를 사용하는 경우는 30초) 이내에 완료되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-200">Any actions performed in the `OnStop` method must be completed within five minutes (or 30 seconds if you are using the Azure emulator on a local computer).</span></span> <span data-ttu-id="c6eee-201">그렇지 않으면 Azure 패브릭 컨트롤러는 역할이 멈추었다고 판단해 역할을 강제로 중지합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-201">Otherwise the Azure fabric controller assumes that the role has stalled and will force it to stop.</span></span>

<span data-ttu-id="c6eee-202">작업은 작업이 완료될 때까지 기다리는 `Run` 메서드를 통해 시작됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-202">The tasks are started by the `Run` method that waits for the tasks to complete.</span></span> <span data-ttu-id="c6eee-203">작업은 클라우드 서비스의 비즈니스 논리를 구현하며 Azure Load Balancer를 통해 역할에 게시된 메시지에 응답할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-203">The tasks implement the business logic of the cloud service, and can respond to messages posted to the role through the Azure load balancer.</span></span> <span data-ttu-id="c6eee-204">다음 그림은 Azure 클라우드 서비스에서 역할 내에 있는 작업과 리소스의 수명을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-204">The figure shows the lifecycle of tasks and resources in a role in an Azure cloud service.</span></span>

![Azure 클라우드 서비스에서 역할 내에 있는 작업과 리소스의 수명](./_images/compute-resource-consolidation-lifecycle.png)


<span data-ttu-id="c6eee-206">_ComputeResourceConsolidation.Worker_ 프로젝트에 포함되어 있는 _WorkerRole.cs_ 파일에서 Azure 클라우드 서비스에서 이 패턴을 구현하는 방법의 예제를 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-206">The _WorkerRole.cs_ file in the _ComputeResourceConsolidation.Worker_ project shows an example of how you might implement this pattern in an Azure cloud service.</span></span>

> <span data-ttu-id="c6eee-207">_ComputeResourceConsolidation.Worker_ 프로젝트는 _ComputeResourceConsolidation_ 솔루션의 일부로, [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation)에서 다운로드할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-207">The _ComputeResourceConsolidation.Worker_ project is part of the _ComputeResourceConsolidation_ solution available for download from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span></span>

<span data-ttu-id="c6eee-208">`MyWorkerTask1` 및 `MyWorkerTask2` 메서드는 동일한 작업자 역할 내에서 여러 작업을 수행하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-208">The `MyWorkerTask1` and the `MyWorkerTask2` methods illustrate how to perform different tasks within the same worker role.</span></span> <span data-ttu-id="c6eee-209">다음 코드는 `MyWorkerTask1` 메서드를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-209">The following code shows `MyWorkerTask1`.</span></span> <span data-ttu-id="c6eee-210">이 메서드는 30초 동안 일시 중지한 다음 추적 메시지를 출력하는 간단한 작업으로,</span><span class="sxs-lookup"><span data-stu-id="c6eee-210">This is a simple task that sleeps for 30 seconds and then outputs a trace message.</span></span> <span data-ttu-id="c6eee-211">작업이 취소될 때까지 이 프로세스를 반복합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-211">It repeats this process until the task is canceled.</span></span> <span data-ttu-id="c6eee-212">`MyWorkerTask2` 메서드의 코드도 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-212">The code in `MyWorkerTask2` is similar.</span></span>

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

> <span data-ttu-id="c6eee-213">샘플 코드는 백그라운드 프로세스의 일반적인 구현을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-213">The sample code shows a common implementation of a background process.</span></span> <span data-ttu-id="c6eee-214">취소 요청을 기다리는 루프의 본문에 자체 처리 논리를 삽입해야 한다는 점을 제외하면 실제 응용 프로그램에서도 이와 동일한 구조를 따를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-214">In a real world application you can follow this same structure, except that you should place your own processing logic in the body of the loop that waits for the cancellation request.</span></span>

<span data-ttu-id="c6eee-215">작업자 역할이 사용하는 리소스를 시작하면 `Run` 메서드가 위에서 예로 든 두 가지 작업을 동시에 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-215">After the worker role has initialized the resources it uses, the `Run` method starts the two tasks concurrently, as shown here.</span></span>

```csharp
/// <summary>
/// The cancellation token source use to cooperatively cancel running tasks
/// </summary>
private readonly CancellationTokenSource cts = new CancellationTokenSource();

/// <summary>
/// List of running tasks on the role instance
/// </summary>
private readonly List<Task> tasks = new List<Task>();

// RoleEntry Run() is called after OnStart().
// Returning from Run() will cause a role instance to recycle.
public override void Run()
{
  // Start worker tasks and add to the task list
  tasks.Add(MyWorkerTask1(cts.Token));
  tasks.Add(MyWorkerTask2(cts.Token));

  foreach (var worker in this.workerTasks)
  {
      this.tasks.Add(worker);
  }

  Trace.TraceInformation("Worker host tasks started");
  // The assumption is that all tasks should remain running and not return,
  // similar to role entry Run() behavior.
  try
  {
    Task.WaitAll(tasks.ToArray());
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

<span data-ttu-id="c6eee-216">이 예에서 `Run` 메서드는 작업이 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-216">In this example, the `Run` method waits for tasks to be completed.</span></span> <span data-ttu-id="c6eee-217">작업이 취소되면 `Run` 메서드는 역할이 종료되고 있다고 가정하고 완료 전에 나머지 작업이 취소될 때까지 대기(종료 전 최대 5분 대기)합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-217">If a task is canceled, the `Run` method assumes that the role is being shut down and waits for the remaining tasks to be canceled before finishing (it waits for a maximum of five minutes before terminating).</span></span> <span data-ttu-id="c6eee-218">예상되는 예외로 인해 작업이 실패하면 `Run` 메서드는 작업을 취소합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-218">If a task fails due to an expected exception, the `Run` method cancels the task.</span></span>

> <span data-ttu-id="c6eee-219">실패한 작업을 다시 시작하거나 역할이 개별 작업을 중지 및 시작할 수 있도록 하는 코드를 삽입하는 등 `Run` 메서드에서 더 포괄적인 모니터링과 예외 처리 전략을 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-219">You could implement more comprehensive monitoring and exception handling strategies in the `Run` method such as restarting tasks that have failed, or including code that enables the role to stop and start individual tasks.</span></span>

<span data-ttu-id="c6eee-220">다음 코드에 표시된 `Stop` 메서드는 패브릭 컨트롤러가 역할 인스턴스를 종료할 때 호출됩니다(`OnStop` 메서드에서 호출).</span><span class="sxs-lookup"><span data-stu-id="c6eee-220">The `Stop` method shown in the following code is called when the fabric controller shuts down the role instance (it's invoked from the `OnStop` method).</span></span> <span data-ttu-id="c6eee-221">다음 코드는 각 작업을 취소해 정상적으로 중지합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-221">The code stops each task gracefully by canceling it.</span></span> <span data-ttu-id="c6eee-222">작업 완료에 소요되는 시간이 5분을 초과하면 `Stop` 메서드의 취소 처리는 대기 상태를 중단하고 역할을 종료합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-222">If any task takes more than five minutes to complete, the cancellation processing in the `Stop` method ceases waiting and the role is terminated.</span></span>

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

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="c6eee-223">관련 패턴 및 지침</span><span class="sxs-lookup"><span data-stu-id="c6eee-223">Related patterns and guidance</span></span>

<span data-ttu-id="c6eee-224">이 패턴을 구현할 때 다음 패턴 및 지침도 관련이 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-224">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="c6eee-225">[자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx).</span><span class="sxs-lookup"><span data-stu-id="c6eee-225">[Autoscaling Guidance](https://msdn.microsoft.com/library/dn589774.aspx).</span></span> <span data-ttu-id="c6eee-226">자동 크기 조정은 예상되는 처리 요구에 따라 계산 리소스를 호스트하는 서비스의 인스턴스를 시작 및 중지하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-226">Autoscaling can be used to start and stop instances of service hosting computational resources, depending on the anticipated demand for processing.</span></span>

- <span data-ttu-id="c6eee-227">[계산 분할 지침](https://msdn.microsoft.com/library/dn589773.aspx).</span><span class="sxs-lookup"><span data-stu-id="c6eee-227">[Compute Partitioning Guidance](https://msdn.microsoft.com/library/dn589773.aspx).</span></span> <span data-ttu-id="c6eee-228">서비스의 확장성, 성능, 가용성, 보안을 유지하면서 운영 비용을 최소화하는 데 도움을 주는 방식으로, 클라우드 서비스에 서비스와 구성 요소를 할당하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-228">Describes how to allocate the services and components in a cloud service in a way that helps to minimize running costs while maintaining the scalability, performance, availability, and security of the service.</span></span>

- <span data-ttu-id="c6eee-229">이 패턴에는 다운로드할 수 있는 [샘플 응용 프로그램](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation)이 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6eee-229">This pattern includes a downloadable [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/compute-resource-consolidation).</span></span>
