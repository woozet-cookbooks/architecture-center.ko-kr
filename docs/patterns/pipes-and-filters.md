---
title: "파이프 및 필터"
description: "복잡한 처리를 수행하는 작업을 재사용 가능한 일련의 별도 요소로 분류합니다."
keywords: "디자인 패턴"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- messaging
ms.openlocfilehash: b41f3e46ad5982a3a4ec6635918481cb440c5e02
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="pipes-and-filters-pattern"></a><span data-ttu-id="974b8-104">파이프 및 필터 패턴</span><span class="sxs-lookup"><span data-stu-id="974b8-104">Pipes and Filters pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="974b8-105">복잡한 처리를 수행하는 작업을 재사용 가능한 일련의 독립 요소로 나눕니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-105">Decompose a task that performs complex processing into a series of separate elements that can be reused.</span></span> <span data-ttu-id="974b8-106">이렇게 하면 처리를 수행하는 작업 요소를 독립적으로 배포하고 크기를 조정할 수 있으므로 성능, 확장성, 재사용성을 개선할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-106">This can improve performance, scalability, and reusability by allowing task elements that perform the processing to be deployed and scaled independently.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="974b8-107">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="974b8-107">Context and problem</span></span>

<span data-ttu-id="974b8-108">응용 프로그램은 처리되는 정보의 복잡도가 서로 다른 여러 작업을 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-108">An application is required to perform a variety of tasks of varying complexity on the information that it processes.</span></span> <span data-ttu-id="974b8-109">간단하면서도 유연성 없는 응용 프로그램 구현 방식은 이 처리를 모놀리식 모듈로 수행하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-109">A straightforward but inflexible approach to implementing an application is to perform this processing as a monolithic module.</span></span> <span data-ttu-id="974b8-110">그러나 이 접근 방식은 코드를 리팩터링, 최적화 또는 응용 프로그램 내의 다른 곳에 같은 처리의 일부가 필요한 경우 다시 사용할 기회를 감소시킬 가능성이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-110">However, this approach is likely to reduce the opportunities for refactoring the code, optimizing it, or reusing it if parts of the same processing are required elsewhere within the application.</span></span>

<span data-ttu-id="974b8-111">그림은 모노리식 방식을 사용해 처리할 때의 문제점을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-111">The figure illustrates the issues with processing data using the monolithic approach.</span></span> <span data-ttu-id="974b8-112">응용 프로그램은 두 가지 원본의 데이터를 받아서 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-112">An application receives and processes data from two sources.</span></span> <span data-ttu-id="974b8-113">각 원본의 데이터는 각각 다른 모듈에 의해 처리되며, 이 모듈은 응용 프로그램 비즈니스 논리로 전달하기 전에 이 데이터를 변환하는 일련의 작업을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-113">The data from each source is processed by a separate module that performs a series of tasks to transform this data, before passing the result to the business logic of the application.</span></span>

![그림 1 - 모놀리식 모듈을 사용하여 구현된 솔루션](./_images/pipes-and-filters-modules.png)

<span data-ttu-id="974b8-115">모노리식 모듈이 수행하는 작업 일부는 기능적인 측면에서 아주 유사하지만 서로 다르게 설계되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-115">Some of the tasks that the monolithic modules perform are functionally very similar, but the modules have been designed separately.</span></span> <span data-ttu-id="974b8-116">작업을 구현하는 코드는 모듈 내에 밀접하게 결합되어 있으며, 재사용 또는 확장성은 거의 고려하지 않고 개발되었습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-116">The code that implements the tasks is closely coupled in a module, and has been developed with little or no thought given to reuse or scalability.</span></span>

<span data-ttu-id="974b8-117">단, 각 모듈에서 수행하는 처리 작업 또는 각 작업에 대한 배포 요구사항은 비즈니스 요구사항이 업데이트될 때 변경될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-117">However, the processing tasks performed by each module, or the deployment requirements for each task, could change as business requirements are updated.</span></span> <span data-ttu-id="974b8-118">계산 집약적인 일부 작업은 강력한 하드웨어에서 실행하면 좋지만, 일부 작업은 그렇게 값비싼 리소스가 필요하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-118">Some tasks might be compute intensive and could benefit from running on powerful hardware, while others might not require such expensive resources.</span></span> <span data-ttu-id="974b8-119">또한, 향후 추가적인 처리가 필요하거나 해당 처리로 수행되는 작업 순서가 변경될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-119">Also, additional processing might be required in the future, or the order in which the tasks performed by the processing could change.</span></span> <span data-ttu-id="974b8-120">이러한 문제를 처리하고 코드 재사용 가능성을 높여 주는 솔루션이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-120">A solution is required that addresses these issues, and increases the possibilities for code reuse.</span></span>

## <a name="solution"></a><span data-ttu-id="974b8-121">해결 방법</span><span class="sxs-lookup"><span data-stu-id="974b8-121">Solution</span></span>

<span data-ttu-id="974b8-122">각 스트림에 필요한 처리를 각각 단일 작업을 수행하는 일련의 독립 구성 요소(또는 필터)로 나눕니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-122">Break down the processing required for each stream into a set of separate components (or filters), each performing a single task.</span></span> <span data-ttu-id="974b8-123">각 구성 요소가 받아서 보내는 데이터의 형식을 표준화하면, 이 필터를 하나의 파이프라인으로 결합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-123">By standardizing the format of the data that each component receives and sends, these filters can be combined together into a pipeline.</span></span> <span data-ttu-id="974b8-124">이렇게 하면 코드 중복을 방지할 수 있으며, 처리 요구 사항이 변경되는 경우 추가 구성 요소를 제거, 대체 또는 통합하는 것이 쉬워집니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-124">This helps to avoid duplicating code, and makes it easy to remove, replace, or integrate additional components if the processing requirements change.</span></span> <span data-ttu-id="974b8-125">다음 그림은 파이프 및 필터를 사용해 구현된 솔루션을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-125">The next figure shows a solution implemented using pipes and filters.</span></span>

![그림 2 - 파이프와 필터를 사용하여 구현된 솔루션](./_images/pipes-and-filters-solution.png)


<span data-ttu-id="974b8-127">단일 요청을 처리하는 데 소요되는 시간은 파이프라인에서 가장 느린 필터의 속도에 좌우됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-127">The time it takes to process a single request depends on the speed of the slowest filter in the pipeline.</span></span> <span data-ttu-id="974b8-128">한 개 이상의 필터로 인해 특히 특정 데이터 원본의 스트림에서 대량의 요청이 나타날 때 병목 현상이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-128">One or more filters could be a bottleneck, especially if a large number of requests appear in a stream from a particular data source.</span></span> <span data-ttu-id="974b8-129">파이프라인 구조의 주요 장점은 느린 필터의 병렬 인스턴스를 실행할 수 있어 시스템이 부하를 분배하고 처리량을 개선할 수 있다는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-129">A key advantage of the pipeline structure is that it provides opportunities for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span>

<span data-ttu-id="974b8-130">파이프라인을 구성하는 필터는 다양한 장비에서 실행할 수 있기 때문에, 독립적으로 크기를 조정할 수 있으며 여러 클라우드 환경이 제공하는 탄력성을 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-130">The filters that make up a pipeline can run on different machines, enabling them to be scaled independently and take advantage of the elasticity that many cloud environments provide.</span></span> <span data-ttu-id="974b8-131">계산 집약적인 필터는 고성능 하드웨어에서 실행할 수 있지만, 소모량이 낮은 필터는 그 보다 비용이 낮은 상용 하드웨어에 호스팅할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-131">A filter that is computationally intensive can run on high performance hardware, while other less demanding filters can be hosted on less expensive commodity hardware.</span></span> <span data-ttu-id="974b8-132">필터는 동일한 데이터 센터나 지리적 위치에 있을 필요는 없으므로, 같은 파이프라인의 요소들을 각각 필요한 리소스에 가까운 환경에서 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-132">The filters don't even have to be in the same data center or geographical location, which allows each element in a pipeline to run in an environment that is close to the resources it requires.</span></span>  <span data-ttu-id="974b8-133">다음 그림은 원본 1의 데이터에 대한 파이프라인에 적용되는 예시를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-133">The next figure shows an example applied to the pipeline for the data from Source 1.</span></span>

![그림 3은 원본 1의 데이터에 대한 파이프라인에 적용되는 예제를 보여 줍니다.](./_images/pipes-and-filters-load-balancing.png)

<span data-ttu-id="974b8-135">필터의 입력과 출력이 한 스트림으로 구성된 경우, 각 필터를 동시에 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-135">If the input and output of a filter are structured as a stream, it's possible to perform the processing for each filter in parallel.</span></span> <span data-ttu-id="974b8-136">파이프라인의 첫 번째 필터가 작업을 시작해서 결과를 출력하고, 이는 첫 번째 필터가 작업을 완료하기 전에 순서대로 그 다음 필터로 직접 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-136">The first filter in the pipeline can start its work and output its results, which are passed directly on to the next filter in the sequence before the first filter has completed its work.</span></span>

<span data-ttu-id="974b8-137">또 다른 장점은 이 모델이 제공할 수 있는 복원력입니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-137">Another benefit is the resiliency that this model can provide.</span></span> <span data-ttu-id="974b8-138">필터가 고장나거나 해당 장비를 더 이상 사용할 수 없는 경우, 파이프라인은 필터가 수행하던 작업의 일정을 조정하고 이 작업을 다른 구성 요소 인스턴스로 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-138">If a filter fails or the machine it's running on is no longer available, the pipeline can reschedule the work that the filter was performing and direct this work to another instance of the component.</span></span> <span data-ttu-id="974b8-139">단일 필터의 실패가 반드시 전체 파이프라인의 실패로 이어지지는 않습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-139">Failure of a single filter doesn't necessarily result in failure of the entire pipeline.</span></span>

<span data-ttu-id="974b8-140">파이프 및 필터 패턴을 [보상 트랜잭션 패턴](compensating-transaction.md)과 함께 사용하는 것은 분산 트랜잭션을 구현하는 또 다른 방식입니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-140">Using the Pipes and Filters pattern in conjunction with the [Compensating Transaction pattern](compensating-transaction.md) is an alternative approach to implementing distributed transactions.</span></span> <span data-ttu-id="974b8-141">분산 트랜잭션은 분리된 보상 가능 작업으로 나눌 수 있으며, 이 작업은 각각 보상 트랜잭션 패턴을 구현하는 필터를 사용해 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-141">A distributed transaction can be broken down into separate, compensable tasks, each of which can be implemented by using a filter that also implements the Compensating Transaction pattern.</span></span> <span data-ttu-id="974b8-142">파이프라인의 필터는 유지 관리되는 데이터에 근접해 실행되는 별도로 호스트된 작업으로 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-142">The filters in a pipeline can be implemented as separate hosted tasks running close to the data that they maintain.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="974b8-143">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="974b8-143">Issues and considerations</span></span>

<span data-ttu-id="974b8-144">이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-144">You should consider the following points when deciding how to implement this pattern:</span></span>
- <span data-ttu-id="974b8-145">**복잡성**.</span><span class="sxs-lookup"><span data-stu-id="974b8-145">**Complexity**.</span></span> <span data-ttu-id="974b8-146">이 패턴의 향상된 유연성으로 인해 파이프라인의 필터가 여러 서버에 걸쳐 분포되어 있는 경우 복잡성을 야기할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-146">The increased flexibility that this pattern provides can also introduce complexity, especially if the filters in a pipeline are distributed across different servers.</span></span>

- <span data-ttu-id="974b8-147">**신뢰성**.</span><span class="sxs-lookup"><span data-stu-id="974b8-147">**Reliability**.</span></span> <span data-ttu-id="974b8-148">파이프라인의 필터 사이에 흐르는 데이터가 손실되지 않는 인프라를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-148">Use an infrastructure that ensures that data flowing between filters in a pipeline won't be lost.</span></span>

- <span data-ttu-id="974b8-149">**멱등성**.</span><span class="sxs-lookup"><span data-stu-id="974b8-149">**Idempotency**.</span></span> <span data-ttu-id="974b8-150">메시지를 수신한 후 파이프라인의 필터에 오류가 나서 작업을 해당 필터의 다른 인스턴스로 조정한 경우, 작업 일부가 이미 완료되었을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-150">If a filter in a pipeline fails after receiving a message and the work is rescheduled to another instance of the filter, part of the work might have already been completed.</span></span> <span data-ttu-id="974b8-151">이 작업이 전역 상태의 일부 측면(데이터베이스에 저장된 정보 등)을 업데이트하는 경우, 동일한 업데이트가 반복될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-151">If this work updates some aspect of the global state (such as information stored in a database), the same update could be repeated.</span></span> <span data-ttu-id="974b8-152">파이프라인의 그 다음 필터에 결과를 게시한 후 작업을 성공적으로 완료했다는 표시를 하기 전 필터가 실패하는 경우 동일한 문제가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-152">A similar issue might occur if a filter fails after posting its results to the next filter in the pipeline, but before indicating that it's completed its work successfully.</span></span> <span data-ttu-id="974b8-153">이러한 경우, 동일한 작업을 필터의 다른 인스턴스에서 반복하여 같은 결과를 두 번 게시하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-153">In these cases, the same work could be repeated by another instance of the filter, causing the same results to be posted twice.</span></span> <span data-ttu-id="974b8-154">이로 인해 파이프라인의 그 다음 필터가 같은 데이터를 두 번 처리하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-154">This could result in subsequent filters in the pipeline processing the same data twice.</span></span> <span data-ttu-id="974b8-155">그러므로 파이프라인의 필터는 멱등성을 가지도록 설계해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-155">Therefore filters in a pipeline should be designed to be idempotent.</span></span> <span data-ttu-id="974b8-156">자세한 내용은 Jonathan Oliver에서 [멱등성 패턴](http://blog.jonathanoliver.com/idempotency-patterns/)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="974b8-156">For more information see [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>

- <span data-ttu-id="974b8-157">**반복 메시지**.</span><span class="sxs-lookup"><span data-stu-id="974b8-157">**Repeated messages**.</span></span> <span data-ttu-id="974b8-158">파이프라인의 다음 단계로 메시지를 게시한 후 파이프라인의 필터가 실패하는 경우, 해당 필터의 다른 인스턴스가 실행될 수 있으며 동일한 메시지 복사본을 파이프라인에 게시합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-158">If a filter in a pipeline fails after posting a message to the next stage of the pipeline, another instance of the filter might be run, and it'll post a copy of the same message to the pipeline.</span></span> <span data-ttu-id="974b8-159">이로 인해 같은 메시지의 인스턴스 두 개가 다음 필터로 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-159">This could cause two instances of the same message to be passed to the next filter.</span></span> <span data-ttu-id="974b8-160">이것을 방지하려면 파이프라인은 중복 메시지를 감지하여 제거해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-160">To avoid this, the pipeline should detect and eliminate duplicate messages.</span></span>

    >  <span data-ttu-id="974b8-161">메시지 큐(Microsoft Azure Service Bus 큐 등)를 사용해 파이프라인을 구현하는 경우, 메시지 큐 인프라에 자동 중복 메시지 감지 및 제거 기능이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-161">If you're implementing the pipeline by using message queues (such as Microsoft Azure Service Bus queues), the message queuing infrastructure might provide automatic duplicate message detection and removal.</span></span>

- <span data-ttu-id="974b8-162">**컨텍스트 및 상태**.</span><span class="sxs-lookup"><span data-stu-id="974b8-162">**Context and state**.</span></span> <span data-ttu-id="974b8-163">파이프라인에서 각 필터는 기본적으로 분리되어 실행되며 호출된 방식에 대한 가정을 해서는 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-163">In a pipeline, each filter essentially runs in isolation and shouldn't make any assumptions about how it was invoked.</span></span> <span data-ttu-id="974b8-164">즉, 각 필터에는 작업을 수행할 충분한 컨텍스트가 제공되어야 한다는 뜻입니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-164">This means that each filter should be provided with sufficient context to perform its work.</span></span> <span data-ttu-id="974b8-165">이 컨텍스트에는 대량의 상태 정보가 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-165">This context could include a large amount of state information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="974b8-166">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="974b8-166">When to use this pattern</span></span>

<span data-ttu-id="974b8-167">다음 경우에 이 패턴을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-167">Use this pattern when:</span></span>
- <span data-ttu-id="974b8-168">응용 프로그램에 필요한 처리는 일련의 독립된 단계로 쉽게 나눌 수 있는 경우</span><span class="sxs-lookup"><span data-stu-id="974b8-168">The processing required by an application can easily be broken down into a set of independent steps.</span></span>

- <span data-ttu-id="974b8-169">응용 프로그램에서 수행하는 처리 단계는 확장성 요구 사항이 서로 다른 경우</span><span class="sxs-lookup"><span data-stu-id="974b8-169">The processing steps performed by an application have different scalability requirements.</span></span>

    >  <span data-ttu-id="974b8-170">동일한 프로세스에서 같이 크기를 조정해야 하는 필터를 그룹화할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-170">It's possible to group filters that should scale together in the same process.</span></span> <span data-ttu-id="974b8-171">자세한 내용은 [계산 리소스 통합 패턴](compute-resource-consolidation.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="974b8-171">For more information, see the [Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span>

- <span data-ttu-id="974b8-172">응용 프로그램이 수행하는 처리 단계의 순서를 다시 매기려면 유연성이 필요하거나 단계를 추가 및 제거할 용량이 필요한 경우</span><span class="sxs-lookup"><span data-stu-id="974b8-172">Flexibility is required to allow reordering of the processing steps performed by an application, or the capability to add and remove steps.</span></span>

- <span data-ttu-id="974b8-173">여러 서버에 걸친 단계 처리를 배포함으로써 시스템이 장점을 취할 수 있는 경우</span><span class="sxs-lookup"><span data-stu-id="974b8-173">The system can benefit from distributing the processing for steps across different servers.</span></span>

- <span data-ttu-id="974b8-174">데이터가 처리되는 중에 어느 단계의 실패를 최소화할 수 있는 신뢰성 있는 솔루션이 필요한 경우</span><span class="sxs-lookup"><span data-stu-id="974b8-174">A reliable solution is required that minimizes the effects of failure in a step while data is being processed.</span></span>

<span data-ttu-id="974b8-175">다음의 경우에는 이 패턴이 유용하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-175">This pattern might not be useful when:</span></span>
- <span data-ttu-id="974b8-176">응용 프로그램에서 수행하는 처리 단계가 독립적이지 않거나 동일한 트랜잭션의 일부로 같이 수행해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="974b8-176">The processing steps performed by an application aren't independent, or they have to be performed together as part of the same transaction.</span></span>

- <span data-ttu-id="974b8-177">해당 단계에 필요한 텍스트 또는 상태 정보의 양으로 인해 이러한 접근의 효율성이 떨어지는 경우.</span><span class="sxs-lookup"><span data-stu-id="974b8-177">The amount of context or state information required by a step makes this approach inefficient.</span></span> <span data-ttu-id="974b8-178">대신 상태 정보를 데이터베이스에 유지할 수 있습니다. 단, 데이터베이스에 대한 추가 부하로 인해 경합이 과도해지는 경우 이 전략을 사용하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="974b8-178">It might be possible to persist state information to a database instead, but don't use this strategy if the additional load on the database causes excessive contention.</span></span>

## <a name="example"></a><span data-ttu-id="974b8-179">예</span><span class="sxs-lookup"><span data-stu-id="974b8-179">Example</span></span>

<span data-ttu-id="974b8-180">메시지 큐의 시퀀스를 사용해 파이프라인 구현에 필요한 인프라를 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-180">You can use a sequence of message queues to provide the infrastructure required to implement a pipeline.</span></span> <span data-ttu-id="974b8-181">초기 메시지 큐는 처리되지 않은 메시지를 수신합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-181">An initial message queue receives unprocessed messages.</span></span> <span data-ttu-id="974b8-182">필터 작업으로 구현된 구성 요소가 이 큐에서 메시지에 대한 수신 대기를 하고, 해당 작업을 수행한 다음 시퀀스의 다음 큐로 변환된 메시지를 게시합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-182">A component implemented as a filter task listens for a message on this queue, performs its work, and then posts the transformed message to the next queue in the sequence.</span></span> <span data-ttu-id="974b8-183">완전히 변환된 데이터가 큐의 최종 메시지에 표시될 때가지 다른 필터 작업이 이 큐에서 메시지에 대한 수신 대기를 하고, 이를 처리하며, 다른 큐로 결과를 게시할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-183">Another filter task can listen for messages on this queue, process them, post the results to another queue, and so on until the fully transformed data appears in the final message in the queue.</span></span> <span data-ttu-id="974b8-184">다음 그림은 메시지 큐를 사용한 파이프라인 구현을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-184">The next figure illustrates implementing a pipeline using message queues.</span></span>

![그림 4 - 메시지 큐를 사용하여 파이프라인 구현](./_images/pipes-and-filters-message-queues.png)


<span data-ttu-id="974b8-186">Azure에 솔루션을 빌드하는 경우, 서비스 버스 큐를 사용해 신뢰성 및 확장성 있는 큐 메커니즘을 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-186">If you're building a solution on Azure you can use Service Bus queues to provide a reliable and scalable queuing mechanism.</span></span> <span data-ttu-id="974b8-187">C#로 된 아래 `ServiceBusPipeFilter` 클래스는 큐로부터 입력 메시지를 수신하여, 이 메시지를 처리해 그 결과를 다른 큐에 게시하는 필터를 구현하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-187">The `ServiceBusPipeFilter` class shown below in C# demonstrates how you can implement a filter that receives input messages from a queue, processes these messages, and posts the results to another queue.</span></span>

>  <span data-ttu-id="974b8-188">`ServiceBusPipeFilter` 클래스는 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters)에서 사용할 수 있는 PipesAndFilters.Shared 프로젝트에 정의되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-188">The `ServiceBusPipeFilter` class is defined in the PipesAndFilters.Shared project available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>

```csharp
public class ServiceBusPipeFilter
{
  ...
  private readonly string inQueuePath;
  private readonly string outQueuePath;
  ...
  private QueueClient inQueue;
  private QueueClient outQueue;
  ...

  public ServiceBusPipeFilter(..., string inQueuePath, string outQueuePath = null)
  {
     ...
     this.inQueuePath = inQueuePath;
     this.outQueuePath = outQueuePath;
  }

  public void Start()
  {
    ...
    // Create the outbound filter queue if it doesn't exist.
    ...
    this.outQueue = QueueClient.CreateFromConnectionString(...);

    ...
    // Create the inbound and outbound queue clients.
    this.inQueue = QueueClient.CreateFromConnectionString(...);
  }

  public void OnPipeFilterMessageAsync(
    Func<BrokeredMessage, Task<BrokeredMessage>> asyncFilterTask, ...)
  {
    ...

    this.inQueue.OnMessageAsync(
      async (msg) =>
    {
      ...
      // Process the filter and send the output to the
      // next queue in the pipeline.
      var outMessage = await asyncFilterTask(msg);

      // Send the message from the filter processor
      // to the next queue in the pipeline.
      if (outQueue != null)
      {
        await outQueue.SendAsync(outMessage);
      }

      // Note: There's a chance that the same message could be sent twice
      // or that a message gets processed by an upstream or downstream
      // filter at the same time.
      // This would happen in a situation where processing of a message was
      // completed, it was sent to the next pipe/queue, and then failed
      // to complete when using the PeekLock method.
      // Idempotent message processing and concurrency should be considered
      // in a real-world implementation.
    },
    options);
  }

  public async Task Close(TimeSpan timespan)
  {
    // Pause the processing threads.
    this.pauseProcessingEvent.Reset();

    // There's no clean approach for waiting for the threads to complete
    // the processing. This example simply stops any new processing, waits
    // for the existing thread to complete, then closes the message pump
    // and finally returns.
    Thread.Sleep(timespan);

    this.inQueue.Close();
    ...
  }

  ...
}
```

<span data-ttu-id="974b8-189">`ServiceBusPipeFilter` 클래스의 `Start` 메서드는 한 쌍의 입력과 출력 큐로 연결되어 있으며, `Close` 메서드는 입력 큐와의 연결이 차단됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-189">The `Start` method in the `ServiceBusPipeFilter` class connects to a pair of input and output queues, and the `Close` method disconnects from the input queue.</span></span> <span data-ttu-id="974b8-190">`OnPipeFilterMessageAsync` 메서드는 실제 메시지 처리를 수행하며 이 메서드에 대한 `asyncFilterTask` 매개 변수는 수행할 처리를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-190">The `OnPipeFilterMessageAsync` method performs the actual processing of messages, the `asyncFilterTask` parameter to this method specifies the processing to be performed.</span></span> <span data-ttu-id="974b8-191">`OnPipeFilterMessageAsync` 메서드는 입력 큐에서 메시지 수신까지 대기하며, 메시지가 도착하면 각 메시지에서 `asyncFilterTask` 매개 변수가 지정한 코드를 실행하고, 그 결과를 출력 큐에 게시합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-191">The `OnPipeFilterMessageAsync` method waits for incoming messages on the input queue, runs the code specified by the `asyncFilterTask` parameter over each message as it arrives, and posts the results to the output queue.</span></span> <span data-ttu-id="974b8-192">큐 자체는 생성자에 의해 지정됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-192">The queues themselves are specified by the constructor.</span></span>

<span data-ttu-id="974b8-193">동일한 솔루션으로 일련의 작업자 역할에서 필터를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-193">The sample solution implements filters in a set of worker roles.</span></span> <span data-ttu-id="974b8-194">각 작업자 역할은 수행하는 비즈니스 처리의 복잡도 또는 처리에 필요한 리소스에 따라 독립적으로 크기를 조정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-194">Each worker role can be scaled independently, depending on the complexity of the business processing that it performs or the resources required for processing.</span></span> <span data-ttu-id="974b8-195">또한, 처리량을 늘리기 위해 각 작업자 역할의 인스턴스를 여러 개 동시에 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-195">Additionally, multiple instances of each worker role can be run in parallel to improve throughput.</span></span>

<span data-ttu-id="974b8-196">아래 코드는 이름이 `PipeFilterARoleEntry`인 Azure 작업자 역할로, 샘플 솔루션의 PipeFilterA 프로젝트에 정의되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-196">The following code shows an Azure worker role named `PipeFilterARoleEntry`, defined in the PipeFilterA project in the sample solution.</span></span>

```csharp
public class PipeFilterARoleEntry : RoleEntryPoint
{
  ...
  private ServiceBusPipeFilter pipeFilterA;

  public override bool OnStart()
  {
    ...
    this.pipeFilterA = new ServiceBusPipeFilter(
      ...,
      Constants.QueueAPath,
      Constants.QueueBPath);

    this.pipeFilterA.Start();
    ...
  }

  public override void Run()
  {
    this.pipeFilterA.OnPipeFilterMessageAsync(async (msg) =>
    {
      // Clone the message and update it.
      // Properties set by the broker (Deliver count, enqueue time, ...)
      // aren't cloned and must be copied over if required.
      var newMsg = msg.Clone();

      await Task.Delay(500); // DOING WORK

      Trace.TraceInformation("Filter A processed message:{0} at {1}",
        msg.MessageId, DateTime.UtcNow);

      newMsg.Properties.Add(Constants.FilterAMessageKey, "Complete");

      return newMsg;
    });

    ...
  }

  ...
}
```

<span data-ttu-id="974b8-197">이 역할에는 `ServiceBusPipeFilter`  개체가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-197">This role contains a `ServiceBusPipeFilter` object.</span></span> <span data-ttu-id="974b8-198">역할에서 `OnStart` 메서드는 입력 메시지 수신과 출력 메시지 게시를 위해 큐에 연결되어 있습니다(큐의 이름은 `Constants` 클래스에 정의되어 있음).</span><span class="sxs-lookup"><span data-stu-id="974b8-198">The `OnStart` method in the role connects to the queues for receiving input messages and posting output messages (the names of the queues are defined in the `Constants` class).</span></span> <span data-ttu-id="974b8-199">`Run` 메서드는 수신한 각 메시지에서 일부 처리를 수행하기 위해 `OnPipeFilterMessagesAsync` 메서드를 호출합니다(이 예제에서는 짧은 시간 동안 대기함으로써 처리를 시뮬레이트함).</span><span class="sxs-lookup"><span data-stu-id="974b8-199">The `Run` method invokes the `OnPipeFilterMessagesAsync` method to perform some processing on each message that's received (in this example, the processing is simulated by waiting for a short period of time).</span></span> <span data-ttu-id="974b8-200">처리가 완료되면, 결과를 포함한 새 메시지가 생성되고(이 경우, 입력 메시지에 사용자 지정 속성이 추가되어 있음), 이 메시지는 출력 큐에 게시됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-200">When processing is complete, a new message is constructed containing the results (in this case, the input message has a custom property added), and this message is posted to the output queue.</span></span>

<span data-ttu-id="974b8-201">샘플 코드에서는 PipeFilterB 프로젝트에 `PipeFilterBRoleEntry`라는 이름의 작업자 역할이 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-201">The sample code contains another worker role named `PipeFilterBRoleEntry` in the PipeFilterB project.</span></span> <span data-ttu-id="974b8-202">이 역할은 `PipeFilterARoleEntry`와 유사하지만 `Run` 메서드로 다른 처리를 수행한다는 점에서 차이가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-202">This role is similar to `PipeFilterARoleEntry` except that it performs different processing in the `Run` method.</span></span> <span data-ttu-id="974b8-203">예제 솔루션에서는 이 두 역할이 결합되어 한 파이프라인을 구성하고 있으며 `PipeFilterARoleEntry` 역할에 대한 출력 큐는 `PipeFilterBRoleEntry` 역할에 대한 입력 큐입니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-203">In the example solution, these two roles are combined to construct a pipeline, the output queue for the `PipeFilterARoleEntry` role is the input queue for the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="974b8-204">또한 샘플 솔루션은 이름이 `InitialSenderRoleEntry`(InitialSender 프로젝트) 및 `FinalReceiverRoleEntry`(FinalReceiver 프로젝트)인 두 개의 추가 역할을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-204">The sample solution also provides two additional roles named `InitialSenderRoleEntry` (in the InitialSender project) and `FinalReceiverRoleEntry` (in the FinalReceiver project).</span></span> <span data-ttu-id="974b8-205">`InitialSenderRoleEntry` 역할은 파이프라인의 초기 메시지를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-205">The `InitialSenderRoleEntry` role provides the initial message in the pipeline.</span></span> <span data-ttu-id="974b8-206">`OnStart` 메서드는 단일 큐에 연결되며 `Run` 메서드가 이 큐에 메서드를 게시합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-206">The `OnStart` method connects to a single queue and the `Run` method posts a method to this queue.</span></span> <span data-ttu-id="974b8-207">이 큐는 `PipeFilterARoleEntry` 역할이 사용하는 입력 큐이므로, 이 큐에 메시지를 게시하면 해당 메시지를 `PipeFilterARoleEntry` 역할이 수신하여 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-207">This queue is the input queue used by the `PipeFilterARoleEntry` role, so posting a message to it causes the message to be received and processed by the `PipeFilterARoleEntry` role.</span></span> <span data-ttu-id="974b8-208">그런 다음 처리된 메시지는 `PipeFilterBRoleEntry` 역할을 통해 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-208">The processed message then passes through the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="974b8-209">`FinalReceiveRoleEntry` 역할의 입력 큐는 `PipeFilterBRoleEntry` 역할의 출력 큐입니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-209">The input queue for the `FinalReceiveRoleEntry` role is the output queue for the `PipeFilterBRoleEntry` role.</span></span> <span data-ttu-id="974b8-210">아래 나타나 있는 `FinalReceiveRoleEntry` 역할의 `Run` 메시지를 수신하여 최종 처리 중 일부를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-210">The `Run` method in the `FinalReceiveRoleEntry` role, shown below, receives the message and performs some final processing.</span></span> <span data-ttu-id="974b8-211">그런 다음 파이프라인의 필터에서 추가한 사용자 지정 속성값을 출력 추적에 씁니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-211">Then it writes the values of the custom properties added by the filters in the pipeline to the trace output.</span></span>

```csharp
public class FinalReceiverRoleEntry : RoleEntryPoint
{
  ...
  // Final queue/pipe in the pipeline to process data from.
  private ServiceBusPipeFilter queueFinal;

  public override bool OnStart()
  {
    ...
    // Set up the queue.
    this.queueFinal = new ServiceBusPipeFilter(...,Constants.QueueFinalPath);
    this.queueFinal.Start();
    ...
  }

  public override void Run()
  {
    this.queueFinal.OnPipeFilterMessageAsync(
      async (msg) =>
      {
        await Task.Delay(500); // DOING WORK

        // The pipeline message was received.
        Trace.TraceInformation(
          "Pipeline Message Complete - FilterA:{0} FilterB:{1}",
          msg.Properties[Constants.FilterAMessageKey],
          msg.Properties[Constants.FilterBMessageKey]);

        return null;
      });
    ...
  }

  ...
}
```

##<a name="related-patterns-and-guidance"></a><span data-ttu-id="974b8-212">관련 패턴 및 지침</span><span class="sxs-lookup"><span data-stu-id="974b8-212">Related patterns and guidance</span></span>

<span data-ttu-id="974b8-213">이 패턴을 구현할 때 다음 패턴 및 지침도 관련이 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-213">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>
- <span data-ttu-id="974b8-214">이 패턴을 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters)에서 사용할 수 있음을 보여주는 샘플.</span><span class="sxs-lookup"><span data-stu-id="974b8-214">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>
- <span data-ttu-id="974b8-215">[경쟁 소비자 패턴](competing-consumers.md)</span><span class="sxs-lookup"><span data-stu-id="974b8-215">[Competing Consumers pattern](competing-consumers.md).</span></span> <span data-ttu-id="974b8-216">파이프라인에는 하나 이상의 필터의 여러 인스턴스가 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-216">A pipeline can contain multiple instances of one or more filters.</span></span> <span data-ttu-id="974b8-217">이 방법은 느린 필터의 병렬 인스턴스를 실행할 때 시스템이 부하를 분산시키고 처리량을 향상시킬 수 있도록 하므로 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-217">This approach is useful for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span> <span data-ttu-id="974b8-218">필터의 각 인스턴스는 다른 인스턴스의 입력과 경쟁하며, 한 필터의 두 인스턴스가 같은 데이터를 처리할 수는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-218">Each instance of a filter will compete for input with the other instances, two instances of a filter shouldn't be able to process the same data.</span></span> <span data-ttu-id="974b8-219">이 방법에 대한 설명이 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-219">Provides an explanation of this approach.</span></span>
- <span data-ttu-id="974b8-220">[계산 리소스 통합 패턴](compute-resource-consolidation.md)</span><span class="sxs-lookup"><span data-stu-id="974b8-220">[Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span> <span data-ttu-id="974b8-221">동일한 프로세스에서 같이 크기를 조정해야 하는 필터를 그룹화할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-221">It might be possible to group filters that should scale together into the same process.</span></span> <span data-ttu-id="974b8-222">이 전략의 장단점에 대한 자세한 내용을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-222">Provides more information about the benefits and tradeoffs of this strategy.</span></span>
- <span data-ttu-id="974b8-223">[보상 트랜잭션 패턴](compensating-transaction.md).</span><span class="sxs-lookup"><span data-stu-id="974b8-223">[Compensating Transaction pattern](compensating-transaction.md).</span></span> <span data-ttu-id="974b8-224">필터를 되돌릴 수 있거나, 오류가 발생할 때 상태를 이전 상태로 복원하는 보상 작업을 포함하는 작업으로 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-224">A filter can be implemented as an operation that can be reversed, or that has a compensating operation that restores the state to a previous version in the event of a failure.</span></span> <span data-ttu-id="974b8-225">이 기능을 구현하여 최종 일관성을 유지 관리하거나 달성하기 위한 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="974b8-225">Explains how this can be implemented to maintain or achieve eventual consistency.</span></span>
- <span data-ttu-id="974b8-226">Jonathan Oliver의 [멱등성 패턴](http://blog.jonathanoliver.com/idempotency-patterns/)</span><span class="sxs-lookup"><span data-stu-id="974b8-226">[Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>
