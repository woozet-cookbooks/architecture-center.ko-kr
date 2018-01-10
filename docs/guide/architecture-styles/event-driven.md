---
title: "이벤트 기반 아키텍처 스타일"
description: "Azure에서 이벤트 기반 아키텍처와 IoT 아키텍처의 혜택, 과제 및 모범 사례를 설명합니다."
author: MikeWasson
ms.openlocfilehash: ff7f936ceabefe7079a1ebbfa717ff4095bf133b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="event-driven-architecture-style"></a><span data-ttu-id="562ee-103">이벤트 기반 아키텍처 스타일</span><span class="sxs-lookup"><span data-stu-id="562ee-103">Event-driven architecture style</span></span>

<span data-ttu-id="562ee-104">이벤트 기반 아키텍처는 이벤트 스트림을 생성하는 **이벤트 생산자**와 이벤트를 수신 대기하는 **이벤트 소비자**로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-104">An event-driven architecture consists of **event producers** that generate a stream of events, and **event consumers** that listen for the events.</span></span> 

![](./images/event-driven.svg)

<span data-ttu-id="562ee-105">이벤트는 거의 실시간으로 전달되므로 이벤트가 발생하는 즉시 소비자가 이벤트에 응답할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-105">Events are delivered in near real time, so consumers can respond immediately to events as they occur.</span></span> <span data-ttu-id="562ee-106">생산자와 소비자가 분리되므로 생산자는 수신 대기 중인 소비자를 알 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-106">Producers are decoupled from consumers &mdash; a producer doesn't know which consumers are listening.</span></span> <span data-ttu-id="562ee-107">소비자도 서로 분리되며 각 소비자에게 모든 이벤트가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-107">Consumers are also decoupled from each other, and every consumer sees all of the events.</span></span> <span data-ttu-id="562ee-108">이는 소비자가 큐에서 메시지를 끌어오고 오류가 없다고 가정할 경우 메시지가 한 번만 처리되는 [경쟁 소비자][competing-consumers] 패턴과 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-108">This differs from a [Competing Consumers][competing-consumers] pattern, where consumers pull messages from a queue and a message is processed just once (assuming no errors).</span></span> <span data-ttu-id="562ee-109">IoT와 같은 일부 시스템에서는 매우 높은 볼륨으로 이벤트를 수집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-109">In some systems, such as IoT, events must be ingested at very high volumes.</span></span>

<span data-ttu-id="562ee-110">이벤트 기반 아키텍처는 게시자/구독자 모델 또는 이벤트 스트림 모델을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-110">An event driven architecture can use a pub/sub model or an event stream model.</span></span> 

- <span data-ttu-id="562ee-111">**게시자/구독자**: 메시징 인프라에서 구독을 추적합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-111">**Pub/sub**: The messaging infrastructure keeps track of subscriptions.</span></span> <span data-ttu-id="562ee-112">이벤트가 게시되면 각 구독자에게 이벤트를 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-112">When an event is published, it sends the event to each subscriber.</span></span> <span data-ttu-id="562ee-113">이벤트를 받은 후에는 재생할 수 없으며 새 구독자에게 이벤트가 표시되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-113">After an event is received, it cannot be replayed, and new subscribers do not see the event.</span></span> 

- <span data-ttu-id="562ee-114">**이벤트 스트리밍**: 이벤트가 로그에 기록됩니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-114">**Event streaming**: Events are written to a log.</span></span> <span data-ttu-id="562ee-115">이벤트가 파티션 내에 엄격하게 정렬되며 지속 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-115">Events are strictly ordered (within a partition) and durable.</span></span> <span data-ttu-id="562ee-116">클라이언트는 스트림을 구독하지 않고, 대신 스트림의 일부에서 읽을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-116">Clients don't subscribe to the stream, instead a client can read from any part of the stream.</span></span> <span data-ttu-id="562ee-117">클라이언트가 스트림에서 해당 위치를 진행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-117">The client is responsible for advancing its position in the stream.</span></span> <span data-ttu-id="562ee-118">즉, 클라이언트가 언제든지 연결하고 이벤트를 재생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-118">That means a client can join at any time, and can replay events.</span></span>

<span data-ttu-id="562ee-119">소비자 측면에서 다음과 같은 몇 가지 일반적인 변형이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-119">On the consumer side, there are some common variations:</span></span>

- <span data-ttu-id="562ee-120">**단순 이벤트 처리**.</span><span class="sxs-lookup"><span data-stu-id="562ee-120">**Simple event processing**.</span></span> <span data-ttu-id="562ee-121">이벤트가 즉시 소비자에서 작업을 트리거합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-121">An event immediately triggers an action in the consumer.</span></span> <span data-ttu-id="562ee-122">예를 들어 서비스 버스 토픽에 메시지를 게시할 때마다 함수가 실행되도록 서비스 버스 트리거와 함께 Azure Functions를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-122">For example, you could use Azure Functions with a Service Bus trigger, so that a function executes whenever a message is published to a Service Bus topic.</span></span>

- <span data-ttu-id="562ee-123">**복합 이벤트 처리**.</span><span class="sxs-lookup"><span data-stu-id="562ee-123">**Complex event processing**.</span></span> <span data-ttu-id="562ee-124">소비자가 Azure Stream Analytics 또는 Apache Storm과 같은 기술을 사용하여 이벤트 데이터에서 패턴을 찾으면서 일련의 이벤트를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-124">A consumer processes a series of events, looking for patterns in the event data, using a technology such as Azure Stream Analytics or Apache Storm.</span></span> <span data-ttu-id="562ee-125">예를 들어 포함된 장치에서 일정 기간 동안 읽은 값을 집계하고 이동 평균이 특정 임계값을 초과하면 알림을 생성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-125">For example, you could aggregate readings from an embedded device over a time window, and generate a notification if the moving average crosses a certain threshold.</span></span> 

- <span data-ttu-id="562ee-126">**이벤트 스트림 처리**.</span><span class="sxs-lookup"><span data-stu-id="562ee-126">**Event stream processing**.</span></span> <span data-ttu-id="562ee-127">Azure IoT Hub 또는 Apache Kafka와 같은 데이터 스트리밍 플랫폼을 파이프라인으로 사용하여 이벤트를 수집하고 스트림 프로세서에 공급합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-127">Use a data streaming platform, such as Azure IoT Hub or Apache Kafka, as a pipeline to ingest events and feed them to stream processors.</span></span> <span data-ttu-id="562ee-128">스트림 프로세서는 스트림을 처리하거나 변환합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-128">The stream processors act to process or transform the stream.</span></span> <span data-ttu-id="562ee-129">응용 프로그램의 각 하위 시스템에 사용되는 여러 스트림 프로세서가 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-129">There may be multiple stream processors for different subsystems of the application.</span></span> <span data-ttu-id="562ee-130">이 접근 방법은 IoT 워크로드에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-130">This approach is a good fit for IoT workloads.</span></span>

<span data-ttu-id="562ee-131">이벤트의 소스가 IoT 솔루션의 물리적 장치와 같이 시스템 외부에 있을 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-131">The source of the events may be external to the system, such as physical devices in an IoT solution.</span></span> <span data-ttu-id="562ee-132">이 경우 시스템이 데이터 원본에 필요한 볼륨 및 처리량으로 데이터를 수집할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-132">In that case, the system must be able to ingest the data at the volume and throughput that is required by the data source.</span></span>

<span data-ttu-id="562ee-133">위의 논리 다이어그램에서 각 소비자 유형은 단일 상자로 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-133">In the logical diagram above, each type of consumer is shown as a single box.</span></span> <span data-ttu-id="562ee-134">실제로는 소비자가 시스템의 단일 실패 지점이 되지 않도록 한 소비자의 여러 인스턴스가 있는 것이 일반적입니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-134">In practice, it's common to have multiple instances of a consumer, to avoid having the consumer become a single point of failure in system.</span></span> <span data-ttu-id="562ee-135">이벤트의 볼륨 및 빈도를 처리하기 위해 여러 인스턴스가 필요할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-135">Multiple instances might also be necessary to handle the volume and frequency of events.</span></span> <span data-ttu-id="562ee-136">또한 단일 소비자가 여러 스레드의 이벤트를 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-136">Also, a single consumer might process events on multiple threads.</span></span> <span data-ttu-id="562ee-137">이벤트가 순서대로 처리되어야 하거나 정확히 한 번 처리되어야 하는 경우 이것이 문제가 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-137">This can create challenges if events must be processed in order, or require exactly-once semantics.</span></span> <span data-ttu-id="562ee-138">[조정 최소화][minimize-coordination]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="562ee-138">See [Minimize Coordination][minimize-coordination].</span></span> 

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="562ee-139">이 아키텍처를 사용하는 경우</span><span class="sxs-lookup"><span data-stu-id="562ee-139">When to use this architecture</span></span>

- <span data-ttu-id="562ee-140">여러 하위 시스템이 동일한 이벤트를 처리해야 하는 경우.</span><span class="sxs-lookup"><span data-stu-id="562ee-140">Multiple subsystems must process the same events.</span></span> 
- <span data-ttu-id="562ee-141">최소 시간 지연의 실시간 처리.</span><span class="sxs-lookup"><span data-stu-id="562ee-141">Real-time processing with minimum time lag.</span></span>
- <span data-ttu-id="562ee-142">패턴 일치 또는 일정 기간의 집계와 같은 복합 이벤트 처리.</span><span class="sxs-lookup"><span data-stu-id="562ee-142">Complex event processing, such as pattern matching or aggregation over time windows.</span></span>
- <span data-ttu-id="562ee-143">높은 볼륨 및 높은 데이터 개발속도(예: IoT).</span><span class="sxs-lookup"><span data-stu-id="562ee-143">High volume and high velocity of data, such as IoT.</span></span>

## <a name="benefits"></a><span data-ttu-id="562ee-144">이점</span><span class="sxs-lookup"><span data-stu-id="562ee-144">Benefits</span></span>

- <span data-ttu-id="562ee-145">생산자와 소비자가 분리됩니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-145">Producers and consumers are decoupled.</span></span>
- <span data-ttu-id="562ee-146">지점 간 통합이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-146">No point-to point-integrations.</span></span> <span data-ttu-id="562ee-147">시스템에 새 소비자를 쉽게 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-147">It's easy to add new consumers to the system.</span></span>
- <span data-ttu-id="562ee-148">이벤트가 도착하는 즉시 소비자가 이벤트에 응답할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-148">Consumers can respond to events immediately as they arrive.</span></span> 
- <span data-ttu-id="562ee-149">확장성이 있고 배포 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-149">Highly scalable and distributed.</span></span> 
- <span data-ttu-id="562ee-150">하위 시스템에서 이벤트 스트림을 독립적으로 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-150">Subsystems have independent views of the event stream.</span></span>

## <a name="challenges"></a><span data-ttu-id="562ee-151">과제</span><span class="sxs-lookup"><span data-stu-id="562ee-151">Challenges</span></span>

- <span data-ttu-id="562ee-152">배달 보장.</span><span class="sxs-lookup"><span data-stu-id="562ee-152">Guaranteed delivery.</span></span> <span data-ttu-id="562ee-153">일부 시스템, 특히 IoT 시나리오에서는 이벤트가 배달되도록 보장하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-153">In some systems, especially in IoT scenarios, it's crucial to guarantee that events are delivered.</span></span>
- <span data-ttu-id="562ee-154">이벤트를 순서대로 또는 한 번만 처리.</span><span class="sxs-lookup"><span data-stu-id="562ee-154">Processing events in order or exactly once.</span></span> <span data-ttu-id="562ee-155">복원 및 확장성을 위해 일반적으로 각 소비자 유형이 여러 인스턴스에서 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-155">Each consumer type typically runs in multiple instances, for resiliency and scalability.</span></span> <span data-ttu-id="562ee-156">이벤트가 소비자 유형 내에서 순서대로 처리되어야 하거나 처리 논리가 비멱등적인 경우 이것이 문제가 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-156">This can create a challenge if the events must be processed in order (within a consumer type), or if the processing logic is not idempotent.</span></span>

## <a name="iot-architecture"></a><span data-ttu-id="562ee-157">IoT 아키텍처</span><span class="sxs-lookup"><span data-stu-id="562ee-157">IoT architecture</span></span>

<span data-ttu-id="562ee-158">이벤트 기반 아키텍처는 IoT 솔루션의 핵심입니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-158">Event-driven architectures are central to IoT solutions.</span></span> <span data-ttu-id="562ee-159">다음 다이어그램은 IoT의 가능한 논리 아키텍처를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-159">The following diagram shows a possible logical architecture for IoT.</span></span> <span data-ttu-id="562ee-160">이 다이어그램에서는 아키텍처의 이벤트 스트리밍 구성 요소가 강조 표시되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-160">The diagram emphasizes the event-streaming components of the architecture.</span></span>

![](./images/iot.png)

<span data-ttu-id="562ee-161">**클라우드 게이트웨이**는 안정적이고 대기 시간이 짧은 메시징 시스템을 사용하여 클라우드 경계에서 장치 이벤트를 수집합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-161">The **cloud gateway** ingests device events at the cloud boundary, using a reliable, low latency messaging system.</span></span>

<span data-ttu-id="562ee-162">장치는 클라우드 게이트웨이에 직접 또는 **필드 게이트웨이**를 통해 이벤트를 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-162">Devices might send events directly to the cloud gateway, or through a **field gateway**.</span></span> <span data-ttu-id="562ee-163">필드 게이트웨이는 이벤트를 수신하여 클라우드 게이트웨이에 전달하는 특수 장치 또는 소프트웨어이며 일반적으로 장치와 함께 배치됩니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-163">A field gateway is a specialized device or software, usually colocated with the devices, that receives events and forwards them to the cloud gateway.</span></span> <span data-ttu-id="562ee-164">필드 게이트웨이에서 필터링, 집계, 프로토콜 변환 등의 기능을 수행하여 원시 장치 이벤트를 전처리할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-164">The field gateway might also preprocess the raw device events, performing functions such as filtering, aggregation, or protocol transformation.</span></span>

<span data-ttu-id="562ee-165">수집된 이벤트는 데이터를 저장소 등으로 라우트하거나 분석 및 기타 처리를 수행할 수 있는 하나 이상의 **스트림 프로세서**를 통과합니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-165">After ingestion, events go through one or more **stream processors** that can route the data (for example, to storage) or perform analytics and other processing.</span></span>

<span data-ttu-id="562ee-166">다음은 몇 가지 일반적인 처리 유형입니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-166">The following are some common types of processing.</span></span> <span data-ttu-id="562ee-167">(전체 목록은 아닙니다.)</span><span class="sxs-lookup"><span data-stu-id="562ee-167">(This list is certainly not exhaustive.)</span></span>

- <span data-ttu-id="562ee-168">보관 또는 배치 분석을 위해 콜드 스토리지에 이벤트 데이터 기록.</span><span class="sxs-lookup"><span data-stu-id="562ee-168">Writing event data to cold storage, for archiving or batch analytics.</span></span>

- <span data-ttu-id="562ee-169">실행 부하 과다 경로 분석, (거의) 실시간으로 이벤트 스트림을 분석하여 이상 검색, 롤링 기간의 패턴 인식 또는 스트림에서 특정 조건이 발생할 때 경고 트리거.</span><span class="sxs-lookup"><span data-stu-id="562ee-169">Hot path analytics, analyzing the event stream in (near) real time, to detect anomalies, recognize patterns over rolling time windows, or trigger alerts when a specific condition occurs in the stream.</span></span> 

- <span data-ttu-id="562ee-170">알림, 경보 등 장치에서 수신된 특수 유형의 비원격 분석 메시지 처리.</span><span class="sxs-lookup"><span data-stu-id="562ee-170">Handling special types of non-telemetry messages from devices, such as notifications and alarms.</span></span> 

- <span data-ttu-id="562ee-171">기계 학습.</span><span class="sxs-lookup"><span data-stu-id="562ee-171">Machine learning.</span></span>

<span data-ttu-id="562ee-172">회색으로 표시된 상자는 이벤트 스트리밍과 직접 관련되지 않은 IoT 시스템의 구성 요소를 표시하지만 전체 표시를 위해 여기에 포함되었습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-172">The boxes that are shaded gray show components of an IoT system that are not directly related to event streaming, but are included here for completeness.</span></span>

- <span data-ttu-id="562ee-173">**장치 레지스트리**는 장치 ID와 일반적으로 장치 메타데이터(예: 위치)를 포함하는 프로비전된 장치의 데이터베이스입니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-173">The **device registry** is a database of the provisioned devices, including the device IDs and usually device metadata, such as location.</span></span>

- <span data-ttu-id="562ee-174">**프로비저닝 API**는 새 장치를 프로비전 및 등록하기 위한 공통 외부 인터페이스입니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-174">The **provisioning API** is a common external interface for provisioning and registering new devices.</span></span>

- <span data-ttu-id="562ee-175">일부 IoT 솔루션에서는 **명령 및 제어 메시지**를 장치에 전송할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-175">Some IoT solutions allow **command and control messages** to be sent to devices.</span></span>

> <span data-ttu-id="562ee-176">이 섹션에서는 IoT를 개괄적으로 설명했으며, 고려해야 할 여러 가지 세부 사항과 과제가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="562ee-176">This section has presented a very high-level view of IoT, and there are many subtleties and challenges to consider.</span></span> <span data-ttu-id="562ee-177">자세한 참조 아키텍처와 토론을 보려면 [Microsoft Azure IoT 참조 아키텍처][iot-ref-arch](PDF 다운로드)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="562ee-177">For a more detailed reference architecture and discussion, see the [Microsoft Azure IoT Reference Architecture][iot-ref-arch] (PDF download).</span></span>

 <!-- links -->

[competing-consumers]: ../../patterns/competing-consumers.md
[iot-ref-arch]: https://azure.microsoft.com/en-us/updates/microsoft-azure-iot-reference-architecture-available/
[minimize-coordination]: ../design-principles/minimize-coordination.md


