---
title: "CQRS 아키텍처 스타일"
description: "CQRS 아키텍처의 이점, 과제 및 모범 사례를 설명합니다."
author: MikeWasson
ms.openlocfilehash: dd3da5886587159f57646ff1bfffa2094725f798
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="cqrs-architecture-style"></a><span data-ttu-id="e95b4-103">CQRS 아키텍처 스타일</span><span class="sxs-lookup"><span data-stu-id="e95b4-103">CQRS architecture style</span></span>

<span data-ttu-id="e95b4-104">CQRS(명령 및 쿼리 책임 분리)는 쓰기 작업에서 읽기 작업을 구분하는 아키텍처 스타일입니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-104">Command and Query Responsibility Segregation (CQRS) is an architecture style that separates read operations from write operations.</span></span> 

![](./images/cqrs-logical.svg)

<span data-ttu-id="e95b4-105">기존 아키텍처에서 데이터베이스를 쿼리하고 업데이트하는 데 동일한 데이터 모델을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-105">In traditional architectures, the same data model is used to query and update a database.</span></span> <span data-ttu-id="e95b4-106">그러면 간단하고 기본적인 CRUD 작업에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-106">That's simple and works well for basic CRUD operations.</span></span> <span data-ttu-id="e95b4-107">그러나 더 복잡한 응용 프로그램에서는 이 방법을 사용하기 어려울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-107">In more complex applications, however, this approach can become unwieldy.</span></span> <span data-ttu-id="e95b4-108">예를 들어 응용 프로그램은 읽기 쪽에서 다른 쿼리를 수행할 수 있습니다. 그러면 모양이 다른 DTO(데이터 전송 개체)를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-108">For example, on the read side, the application may perform many different queries, returning data transfer objects (DTOs) with different shapes.</span></span> <span data-ttu-id="e95b4-109">개체 매핑이 복잡해 질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-109">Object mapping can become complicated.</span></span> <span data-ttu-id="e95b4-110">모델은 쓰기 쪽에서 복잡한 유효성 검사 및 비즈니스 논리를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-110">On the write side, the model may implement complex validation and business logic.</span></span> <span data-ttu-id="e95b4-111">따라서 너무 많은 작업을 수행하는 과도하게 복잡한 모델이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-111">As a result, you can end up with an overly complex model that does too much.</span></span>

<span data-ttu-id="e95b4-112">읽기 및 쓰기 워크로드는 매우 다른 성능 및 확장성 요구 사항을 포함하여 비대칭이라는 점이 또 다른 잠재적인 문제입니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-112">Another potential problem is that read and write workloads are often asymmetrical, with very different performance and scale requirements.</span></span> 

<span data-ttu-id="e95b4-113">CQRS는 읽기 및 쓰기를 구분된 모델로 구분하여 이러한 문제를 해결합니다. 이 때 데이터를 업데이트하는 **명령** 및 데이터를 읽는 **쿼리**를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-113">CQRS addresses these problems by separating reads and writes into separate models, using **commands** to update data, and **queries** to read data.</span></span>

- <span data-ttu-id="e95b4-114">명령은 데이터 중심이 아닌 작업을 기반으로 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-114">Commands should be task based, rather than data centric.</span></span> <span data-ttu-id="e95b4-115">("ReservationStatus를 Reserved로 설정"하지 않고 "호텔 객실 예약")명령은 동기적으로 처리되지 않고 비동기 처리를 위해 큐에 배치될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-115">("Book hotel room," not "set ReservationStatus to Reserved.") Commands may be placed on a queue for asynchronous processing, rather than being processed synchronously.</span></span>

- <span data-ttu-id="e95b4-116">쿼리는 데이터베이스를 수정하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-116">Queries never modify the database.</span></span> <span data-ttu-id="e95b4-117">쿼리는 도메인 정보를 캡슐화하지 않는 DTO를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-117">A query returns a DTO that does not encapsulate any domain knowledge.</span></span>

<span data-ttu-id="e95b4-118">더 높은 격리 수준을 위해 쓰기 데이터에서 읽기 데이터를 물리적으로 구분할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-118">For greater isolation, you can physically separate the read data from the write data.</span></span> <span data-ttu-id="e95b4-119">이 경우에 읽기 데이터베이스는 쿼리에 대해 최적화된 고유한 데이터 스키마를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-119">In that case, the read database can use its own data schema that is optimized for queries.</span></span> <span data-ttu-id="e95b4-120">예를 들어 복잡한 조인이나 복잡한 O/RM 매핑을 방지하기 위해 데이터의 [구체화된 뷰][materialized-view]를 저장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-120">For example, it can store a [materialized view][materialized-view] of the data, in order to avoid complex joins or complex O/RM mappings.</span></span> <span data-ttu-id="e95b4-121">다른 유형의 데이터 저장소도 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-121">It might even use a different type of data store.</span></span> <span data-ttu-id="e95b4-122">예를 들어 쓰기 데이터베이스가 관계형일 수 있는 반면 읽기 데이터베이스는 문서 데이터베이스입니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-122">For example, the write database might be relational, while the read database is a document database.</span></span>

<span data-ttu-id="e95b4-123">별도 읽기 및 쓰기 데이터베이스를 사용하는 경우 동기화를 유지해야 합니다. 일반적으로 데이터베이스를 업데이트할 때마다 쓰기 모델에서 이벤트를 게시함으로써 수행됩니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-123">If separate read and write databases are used, they must be kept in sync. Typically this is accomplished by  having the write model publish an event whenever it updates the database.</span></span> <span data-ttu-id="e95b4-124">데이터베이스를 업데이트하고 이벤트를 게시하는 작업은 단일 트랜잭션에서 이루어져야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-124">Updating the database and publishing the event must occur in a single transaction.</span></span> 

<span data-ttu-id="e95b4-125">CQRS의 일부 구현에서는 [이벤트 소싱 패턴][event-sourcing]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-125">Some implementations of CQRS use the [Event Sourcing pattern][event-sourcing].</span></span> <span data-ttu-id="e95b4-126">이러한 패턴에서 응용 프로그램 상태는 이벤트의 시퀀스로 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-126">With this pattern, application state is stored as a sequence of events.</span></span> <span data-ttu-id="e95b4-127">각 이벤트는 데이터에 대한 변경 집합을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-127">Each event represents a set of changes to the data.</span></span> <span data-ttu-id="e95b4-128">현재 상태는 이벤트를 재생함으로써 구축됩니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-128">The current state is constructed by replaying the events.</span></span> <span data-ttu-id="e95b4-129">CQRS 컨텍스트에서 이벤트 소싱의 이점 중 하나는 다른 구성 요소 &mdash;를 알리는 데 동일한 이벤트를 사용할 수 있다는 점입니다. 특히 읽기 모델에 알립니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-129">In a CQRS context, one benefit of Event Sourcing is that the same events can be used to notify other components &mdash; in particular, to notify the read model.</span></span> <span data-ttu-id="e95b4-130">읽기 모델은 현재 상태의 스냅숏을 만드는 데 이벤트를 사용합니다. 이것이 쿼리에 보다 효과적입니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-130">The read model uses the events to create a snapshot of the current state, which is more efficient for queries.</span></span> <span data-ttu-id="e95b4-131">그러나 이벤트 소싱은 디자인에 복잡성을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-131">However, Event Sourcing adds complexity to the design.</span></span>

![](./images/cqrs-events.svg)

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="e95b4-132">이 아키텍처를 사용하는 경우</span><span class="sxs-lookup"><span data-stu-id="e95b4-132">When to use this architecture</span></span>

<span data-ttu-id="e95b4-133">읽기 및 쓰기 워크로드가 비대칭인 경우에 특히 많은 사용자가 동일한 데이터에 액세스하는 공동 작업 도메인에 CQRS를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-133">Consider CQRS for collaborative domains where many users access the same data, especially when the read and write workloads are asymmetrical.</span></span>

<span data-ttu-id="e95b4-134">CQRS는 전체 시스템에 적용되는 최상위 아키텍처가 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-134">CQRS is not a top-level architecture that applies to an entire system.</span></span> <span data-ttu-id="e95b4-135">읽기 및 쓰기를 구분하는 명확한 값이 있는 해당 하위 시스템에만 CQRS를 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-135">Apply CQRS only to those subsystems where there is clear value in separating reads and writes.</span></span> <span data-ttu-id="e95b4-136">그렇지 않으면 아무 이점 없이 복잡성만 추가하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-136">Otherwise, you are creating additional complexity for no benefit.</span></span>

## <a name="benefits"></a><span data-ttu-id="e95b4-137">이점</span><span class="sxs-lookup"><span data-stu-id="e95b4-137">Benefits</span></span>

- <span data-ttu-id="e95b4-138">**독립적인 크기 조정**</span><span class="sxs-lookup"><span data-stu-id="e95b4-138">**Independently scaling**.</span></span> <span data-ttu-id="e95b4-139">CQRS를 통해 읽기 및 쓰기 워크로드를 독립적으로 확장하고 더 적은 수의 잠금 경합이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-139">CQRS allows the read and write workloads to scale independently, and may result in fewer lock contentions.</span></span>
- <span data-ttu-id="e95b4-140">**최적화된 데이터 스키마**</span><span class="sxs-lookup"><span data-stu-id="e95b4-140">**Optimized data schemas.**</span></span>  <span data-ttu-id="e95b4-141">읽기 쪽에서는 쿼리에 최적화된 스키마를 사용하는 반면 쓰기 쪽에서는 업데이트에 최적화된 스키마를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-141">The read side can use a schema that is optimized for queries, while the write side uses a schema that is optimized for updates.</span></span>  
- <span data-ttu-id="e95b4-142">**보안**.</span><span class="sxs-lookup"><span data-stu-id="e95b4-142">**Security**.</span></span> <span data-ttu-id="e95b4-143">올바른 도메인 엔터티만 데이터에서 쓰기를 수행할 수 있는지 쉽게 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-143">It's easier to ensure that only the right domain entities are performing writes on the data.</span></span>
- <span data-ttu-id="e95b4-144">**문제 구분**</span><span class="sxs-lookup"><span data-stu-id="e95b4-144">**Separation of concerns**.</span></span> <span data-ttu-id="e95b4-145">읽기 및 쓰기 쪽을 구분하면 유지 가능하고 유연한 모델을 생성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-145">Segregating the read and write sides can result in models that are more maintainable and flexible.</span></span> <span data-ttu-id="e95b4-146">대부분의 복잡한 비즈니스 논리는 쓰기 모델로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-146">Most of the complex business logic goes into the write model.</span></span> <span data-ttu-id="e95b4-147">읽기 모델은 상대적으로 간단할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-147">The read model can be relatively simple.</span></span>
- <span data-ttu-id="e95b4-148">**단순한 쿼리**</span><span class="sxs-lookup"><span data-stu-id="e95b4-148">**Simpler queries**.</span></span> <span data-ttu-id="e95b4-149">읽기 데이터베이스에서 구체화된 뷰를 저장하여 쿼리할 때 응용 프로그램은 복잡한 조인을 방지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-149">By storing a materialized view in the read database, the application can avoid complex joins when querying.</span></span>

## <a name="challenges"></a><span data-ttu-id="e95b4-150">과제</span><span class="sxs-lookup"><span data-stu-id="e95b4-150">Challenges</span></span>

- <span data-ttu-id="e95b4-151">**복잡성**.</span><span class="sxs-lookup"><span data-stu-id="e95b4-151">**Complexity**.</span></span> <span data-ttu-id="e95b4-152">CQRS의 기본 개념은 간단합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-152">The basic idea of CQRS is simple.</span></span> <span data-ttu-id="e95b4-153">하지만 이벤트 소싱 패턴을 포함하는 경우에 특히 더 복잡한 응용 프로그램 디자인을 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-153">But it can lead to a more complex application design, especially if they include the Event Sourcing pattern.</span></span>

- <span data-ttu-id="e95b4-154">**메시징**</span><span class="sxs-lookup"><span data-stu-id="e95b4-154">**Messaging**.</span></span> <span data-ttu-id="e95b4-155">CQRS에 메시징이 필요하지 않지만 명령을 처리하고 업데이트 이벤트를 게시하는 데 공통적으로 메시징을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-155">Although CQRS does not require messaging, it's common to use messaging to process commands and publish update events.</span></span> <span data-ttu-id="e95b4-156">이 경우에 응용 프로그램은 메시지 오류 또는 중복 메시지를 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-156">In that case, the application must handle message failures or duplicate messages.</span></span> 

- <span data-ttu-id="e95b4-157">**결과적 일관성**</span><span class="sxs-lookup"><span data-stu-id="e95b4-157">**Eventual consistency**.</span></span> <span data-ttu-id="e95b4-158">읽기 및 쓰기 데이터베이스를 구분하는 경우 읽기 데이터는 기한이 경과되었을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-158">If you separate the read and write databases, the read data may be stale.</span></span> 

## <a name="best-practices"></a><span data-ttu-id="e95b4-159">모범 사례</span><span class="sxs-lookup"><span data-stu-id="e95b4-159">Best practices</span></span>

- <span data-ttu-id="e95b4-160">CQRS를 구현하는 방법에 대한 자세한 내용은 [CQRS 패턴][cqrs-pattern]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e95b4-160">For more information about implementing CQRS, see [CQRS Pattern][cqrs-pattern].</span></span>

- <span data-ttu-id="e95b4-161">[이벤트 소싱][event-sourcing] 패턴을 사용하여 업데이트 충돌을 방지하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-161">Consider using the [Event Sourcing][event-sourcing] pattern to avoid update conflicts.</span></span>

- <span data-ttu-id="e95b4-162">쿼리의 스키마를 최적화하기 위해 읽기 모델에 [구체화된 뷰 패턴][materialized-view]을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-162">Consider using the [Materialized View pattern][materialized-view] for the read model, to optimize the schema for queries.</span></span>

## <a name="cqrs-in-microservices"></a><span data-ttu-id="e95b4-163">마이크로 서비스의 CQRS</span><span class="sxs-lookup"><span data-stu-id="e95b4-163">CQRS in microservices</span></span>

<span data-ttu-id="e95b4-164">CQRS는 [마이크로 서비스 아키텍처][microservices]에서 특히 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-164">CQRS can be especially useful in a [microservices architecture][microservices].</span></span> <span data-ttu-id="e95b4-165">마이크로 서비스의 원리 중 하나는 서비스가 다른 서비스의 데이터 저장소에 직접 액세스할 수 없다는 점입니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-165">One of the principles of microservices is that a service cannot directly access another service's data store.</span></span>

![](./images/cqrs-microservices-wrong.png)

<span data-ttu-id="e95b4-166">다음 다이어그램에서 서비스 A는 데이터 저장소에 기록하고 서비스 B는 데이터의 구체화된 뷰를 유지합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-166">In the following diagram, Service A writes to a data store, and Service B keeps a materialized view of the data.</span></span> <span data-ttu-id="e95b4-167">서비스 A는 데이터 저장소에 기록할 때마다 이벤트를 게시합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-167">Service A publishes an event whenever it writes to the data store.</span></span> <span data-ttu-id="e95b4-168">서비스 B는 이벤트를 구독합니다.</span><span class="sxs-lookup"><span data-stu-id="e95b4-168">Service B subscribes to the event.</span></span>

![](./images/cqrs-microservices-right.png)


<!-- links -->

[cqrs-pattern]: ../../patterns/cqrs.md
[event-sourcing]: ../../patterns/event-sourcing.md
[materialized-view]: ../../patterns/materialized-view.md
[microservices]: ./microservices.md
