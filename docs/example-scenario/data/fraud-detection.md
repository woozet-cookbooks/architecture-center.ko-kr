---
title: 고급 분석 &mdash; 실시간 부정 행위 감지
description: Azure Event Hubs 및 Stream Analytics를 사용하여 부정 행위를 실시간으로 감지하는 입증된 솔루션입니다.
author: alexbuckgit
ms.date: 07/05/2018
ms.openlocfilehash: cf375445b38b0ff7d6fbc400902d5e97b34b4fed
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891324"
---
# <a name="real-time-fraud-detection-on-azure"></a><span data-ttu-id="35e3e-103">Azure에서 실시간 부정 행위 감지</span><span class="sxs-lookup"><span data-stu-id="35e3e-103">Real-time fraud detection on Azure</span></span>

<span data-ttu-id="35e3e-104">이 예제 시나리오는 데이터를 실시간으로 분석하여 부정 행위 거래 또는 기타 비정상적인 활동을 감지해야 하는 조직과 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-104">This example scenario is relevant to organizations that need to analyze data in real-time to detect fraudulent transactions or other anomalous activity.</span></span>

<span data-ttu-id="35e3e-105">잠재적인 응용 프로그램에는 신용 카드 부정 행위 또는 휴대폰 전화를 식별하는 기능이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-105">Potential applications include identifying fraudulent credit card activity or mobile phone calls.</span></span> <span data-ttu-id="35e3e-106">기존 온라인 분석 시스템에서 데이터를 변환하고 분석하여 비정상적인 활동을 식별하는 데 몇 시간이 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-106">Traditional online analytical systems might take hours to transform and analyze the data to identify anomalous activity.</span></span>

<span data-ttu-id="35e3e-107">Event Hubs 및 Stream Analytics와 같이 완전하게 관리되는 Azure 서비스를 사용하면, 개별 서버를 관리할 필요가 없으며, 비용을 줄이고, 클라우드 규모의 데이터 수집 및 실시간 분석에 대한 Microsoft의 전문 지식을 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-107">By using fully managed Azure services such as Event Hubs and Stream Analytics, companies can eliminate the need to manage individual servers, while reducing costs and leveraging Microsoft's expertise in cloud-scale data ingestion and real-time analytics.</span></span> <span data-ttu-id="35e3e-108">이 시나리오에서는 특히 부정 행위 감지에 대해 다루고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-108">This scenario specifically addresses the detection of fraudulent activity.</span></span> <span data-ttu-id="35e3e-109">데이터 분석에 대한 다른 요구 사항이 있으면 사용 가능한 [Azure Analytics 서비스][product-category] 목록을 검토해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-109">If you have other needs for data analytics, you should review the list of available [Azure Analytics services][product-category].</span></span>

<span data-ttu-id="35e3e-110">이 샘플은 광범위한 데이터 처리 아키텍처 및 전략의 한 부분을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-110">This sample represents one part of a broader data processing architecture and strategy.</span></span> <span data-ttu-id="35e3e-111">전체 아키텍처에서 이 측면에 대한 다른 옵션은 이 문서의 뒷부분에서 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-111">Other options for this aspect of an overall architecture are discussed later in this article.</span></span>
 
## <a name="potential-use-cases"></a><span data-ttu-id="35e3e-112">잠재적인 사용 사례</span><span class="sxs-lookup"><span data-stu-id="35e3e-112">Potential use cases</span></span>

<span data-ttu-id="35e3e-113">이 솔루션을 사용하는 데 적합한 사용 사례는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-113">Consider this solution for the following use cases:</span></span>

* <span data-ttu-id="35e3e-114">통신 시나리오에서 휴대폰의 부정 전화 행위 감지</span><span class="sxs-lookup"><span data-stu-id="35e3e-114">Detecting fraudulent mobile-phone calls in telecommunications scenarios.</span></span>
* <span data-ttu-id="35e3e-115">은행 기관에서 신용 카드의 부정 거래 행위 식별</span><span class="sxs-lookup"><span data-stu-id="35e3e-115">Identifying fraudulent credit card transactions for banking institutions.</span></span>
* <span data-ttu-id="35e3e-116">소매 또는 전자 상거래 시나리오에서 부정 구매 행위 식별</span><span class="sxs-lookup"><span data-stu-id="35e3e-116">Identifying fraudulent purchases in retail or e-commerce scenarios.</span></span>

## <a name="architecture"></a><span data-ttu-id="35e3e-117">아키텍처</span><span class="sxs-lookup"><span data-stu-id="35e3e-117">Architecture</span></span>

![실시간 부정 행위 감지 솔루션의 Azure 구성 요소 아키텍처에 대한 개요][architecture-diagram]

<span data-ttu-id="35e3e-119">이 솔루션에는 실시간 분석 파이프라인의 백 엔드 구성 요소가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-119">This solution covers the back-end components of a real-time analytics pipeline.</span></span> <span data-ttu-id="35e3e-120">솔루션을 통한 데이터 흐름은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-120">Data flows through the solution as follows:</span></span>

1. <span data-ttu-id="35e3e-121">휴대폰 호출 메타데이터는 원본 시스템에서 Azure Event Hubs 인스턴스로 보내집니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-121">Mobile phone call metadata is sent from the source system to an Azure Event Hubs instance.</span></span> 
2. <span data-ttu-id="35e3e-122">이벤트 허브 원본을 통해 데이터를 받는 Stream Analytics 작업이 시작됩니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-122">A Stream Analytics job is started, which receives data via the event hub source.</span></span>
3. <span data-ttu-id="35e3e-123">Stream Analytics 작업은 미리 정의된 쿼리를 실행하여 입력 스트림을 변환하고 부정 행위 트랜잭션 알고리즘에 따라 분석합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-123">The Stream Analytics job runs a predefined query to transform the input stream and analyze it based on a fraudulent-transaction algorithm.</span></span> <span data-ttu-id="35e3e-124">이 쿼리는 연속 창을 사용하여 스트림을 개별 시간 단위로 구분합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-124">This query uses a tumbling window to segment the stream into distinct temporal units.</span></span>
4. <span data-ttu-id="35e3e-125">Stream Analytics 작업은 감지된 부정 행위 호출을 나타내는 변환된 스트림을 Azure Blob 저장소의 출력 싱크에 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-125">The Stream Analytics job writes the transformed stream representing detected fraudulent calls to an output sink in Azure Blob storage.</span></span>

### <a name="components"></a><span data-ttu-id="35e3e-126">구성 요소</span><span class="sxs-lookup"><span data-stu-id="35e3e-126">Components</span></span>

* <span data-ttu-id="35e3e-127">[Azure Event Hubs][docs-event-hubs]는 초당 수백만 개의 이벤트를 받고 처리할 수 있는 실시간 스트리밍 플랫폼 및 이벤트 수집 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-127">[Azure Event Hubs][docs-event-hubs] is a real-time streaming platform and event ingestion service, capable of receiving and processing millions of events per second.</span></span> <span data-ttu-id="35e3e-128">Event Hubs는 분산된 소프트웨어와 장치에서 생성된 이벤트, 데이터 또는 원격 분석을 처리하고 저장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-128">Event Hubs can process and store events, data, or telemetry produced by distributed software and devices.</span></span> <span data-ttu-id="35e3e-129">이 솔루션에서 Event Hubs는 부정 행위로 분석될 모든 전화 호출 메타데이터를 받습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-129">In this solution, Event Hubs receives all phone call metadata to be analyzed for fraudulent activity.</span></span>
* <span data-ttu-id="35e3e-130">[Azure Stream Analytics][docs-stream-analytics]는 장치 및 다른 데이터 원본에서 스트림하는 대량의 데이터를 분석할 수 있는 이벤트 처리 엔진입니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-130">[Azure Stream Analytics][docs-stream-analytics] is an event-processing engine that can analyze high volumes of data streaming from devices and other data sources.</span></span> <span data-ttu-id="35e3e-131">또한 데이터 스트림에서 정보를 추출하여 패턴과 관계를 식별할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-131">It also supports extracting information from data streams to identify patterns and relationships.</span></span> <span data-ttu-id="35e3e-132">이러한 패턴은 다른 다운스트림 작업을 트리거할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-132">These patterns can trigger other downstream actions.</span></span> <span data-ttu-id="35e3e-133">이 솔루션에서는 Stream Analytics에서 Event Hubs로부터의 입력 스트림을 변환하여 부정 행위 호출을 식별합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-133">In this solution, Stream Analytics transforms the input stream from Event Hubs to identify fraudulent calls.</span></span>
* <span data-ttu-id="35e3e-134">[Blob 저장소][docs-blob-storage]는 이 솔루션에서 Stream Analytics 작업의 결과를 저장하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-134">[Blob storage][docs-blob-storage] is used in this solution to store the results of the Stream Analytics job.</span></span>

## <a name="considerations"></a><span data-ttu-id="35e3e-135">고려 사항</span><span class="sxs-lookup"><span data-stu-id="35e3e-135">Considerations</span></span>

### <a name="alternatives"></a><span data-ttu-id="35e3e-136">대안</span><span class="sxs-lookup"><span data-stu-id="35e3e-136">Alternatives</span></span>

<span data-ttu-id="35e3e-137">실시간 메시지 수집, 데이터 저장, 스트림 처리, 분석 데이터 저장, 분석 및 보고에 많은 기술을 선택하여 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-137">Many technology choices are available for real-time message ingestion, data storage, stream processing, storage of analytical data, and analytics and reporting.</span></span> <span data-ttu-id="35e3e-138">이러한 옵션, 기능 및 주요 선택 기준에 대한 개요는 Azure 데이터 아키텍처 가이드의 [빅 데이터 아키텍처: 실시간 처리](/azure/architecture/data-guide/technology-choices/real-time-ingestion)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="35e3e-138">For an overview of these options, their capabilities, and key selection criteria, see [Big data architectures: Real-time processing](/azure/architecture/data-guide/technology-choices/real-time-ingestion) in the Azure Data Architecture Guide.</span></span>

<span data-ttu-id="35e3e-139">또한 부정 행위 감지에 대해 더 복잡한 알고리즘은 Azure의 다양한 기계 학습 서비스에서 생성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-139">Additionally, more complex algorithms for fraud detection can be produced by various machine learning services in Azure.</span></span> <span data-ttu-id="35e3e-140">이러한 옵션에 대한 개요는 [Azure 데이터 아키텍처 가이드](../../data-guide/index.md)의 [기계 학습에 대한 기술 선택](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="35e3e-140">For an overview of these options, see [Technology choices for machine learning](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning) in the [Azure Data Architecture Guide](../../data-guide/index.md).</span></span>

### <a name="availability"></a><span data-ttu-id="35e3e-141">가용성</span><span class="sxs-lookup"><span data-stu-id="35e3e-141">Availability</span></span>

<span data-ttu-id="35e3e-142">Azure Monitor는 다양한 Azure 서비스를 모니터링하기 위한 통합된 사용자 인터페이스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-142">Azure Monitor provides unified user interfaces for monitoring across various Azure services.</span></span> <span data-ttu-id="35e3e-143">자세한 내용은 [Microsoft Azure에서 모니터링](/azure/monitoring-and-diagnostics/monitoring-overview)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="35e3e-143">For more information, see [Monitoring in Microsoft Azure](/azure/monitoring-and-diagnostics/monitoring-overview).</span></span> <span data-ttu-id="35e3e-144">Event Hubs 및 Stream Analytics는 모두 Azure Monitor와 통합됩니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-144">Event Hubs and Stream Analytics are both integrated with Azure Monitor.</span></span> 

<span data-ttu-id="35e3e-145">다른 가용성 고려 사항에 대해서는 Azure 아키텍처 센터의 [가용성 검사 목록][availability]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="35e3e-145">For other availability considerations, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="35e3e-146">확장성</span><span class="sxs-lookup"><span data-stu-id="35e3e-146">Scalability</span></span>

<span data-ttu-id="35e3e-147">이 솔루션의 구성 요소는 하이퍼스케일 수집 및 대규모 병렬 실시간 분석을 위해 설계되었습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-147">The components of this solution are designed for hyper-scale ingestion and massively parallel real-time analytics.</span></span> <span data-ttu-id="35e3e-148">Azure Event Hubs는 확장성이 뛰어나고, 대기 시간이 짧고 초당 수백만 개의 이벤트를 받고 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-148">Azure Event Hubs is highly scalable, capable of receiving and processing millions of events per second with low latency.</span></span>  <span data-ttu-id="35e3e-149">Event Hubs는 사용량 요구 사항에 맞게 처리량 단위 수를 [자동으로 확장](/azure/event-hubs/event-hubs-auto-inflate)할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-149">Event Hubs can [automatically scale up](/azure/event-hubs/event-hubs-auto-inflate) the number of throughput units to meet usage needs.</span></span> <span data-ttu-id="35e3e-150">Azure Stream Analytics는 많은 원본에서 대량의 스트리밍 데이터를 분석할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-150">Azure Stream Analytics is capable of analyzing high volumes of streaming data from many sources.</span></span> <span data-ttu-id="35e3e-151">스트리밍 작업을 실행하기 위해 할당된 [스트리밍 단위](/azure/stream-analytics/stream-analytics-streaming-unit-consumption)의 수를 늘려 Stream Analytics를 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-151">You can scale up Stream Analytics by increasing the number of [streaming units](/azure/stream-analytics/stream-analytics-streaming-unit-consumption) allocated to execute your streaming job.</span></span>

<span data-ttu-id="35e3e-152">확장 가능한 솔루션 설계에 대한 일반적인 지침은 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="35e3e-152">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="35e3e-153">보안</span><span class="sxs-lookup"><span data-stu-id="35e3e-153">Security</span></span>

<span data-ttu-id="35e3e-154">Azure Event Hubs는 SAS(공유 액세스 서명) 토큰과 이벤트 게시자의 조합에 기반한 [인증 및 보안 모델][docs-event-hubs-security-model]을 통해 데이터를 보호합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-154">Azure Event Hubs secures data through an [authentication and security model][docs-event-hubs-security-model] based on a combination of Shared Access Signature (SAS) tokens and event publishers.</span></span> <span data-ttu-id="35e3e-155">이벤트 게시자는 이벤트 허브에 대한 가상 끝점을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-155">An event publisher defines a virtual endpoint for an event hub.</span></span> <span data-ttu-id="35e3e-156">게시자는 이벤트 허브에 메시지를 보내는 데만 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-156">The publisher can only be used to send messages to an event hub.</span></span> <span data-ttu-id="35e3e-157">게시자에서 메시지를 받을 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-157">It is not possible to receive messages from a publisher.</span></span>

<span data-ttu-id="35e3e-158">보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="35e3e-158">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="35e3e-159">복원력</span><span class="sxs-lookup"><span data-stu-id="35e3e-159">Resiliency</span></span>

<span data-ttu-id="35e3e-160">복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 응용 프로그램 디자인][resiliency]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="35e3e-160">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="35e3e-161">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="35e3e-161">Deploy the solution</span></span>

<span data-ttu-id="35e3e-162">이 솔루션을 배포하려면 이 [단계별 자습서][tutorial]에 따라 솔루션의 각 구성 요소를 수동으로 배포하는 방법을 시연할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-162">To deploy this solution, you can follow this [step-by-step tutorial][tutorial] demonstrating how to manually deploy each component of the solution.</span></span> <span data-ttu-id="35e3e-163">또한 이 자습서에서는 .NET 클라이언트 응용 프로그램을 제공하여 샘플 전화 호출 메타데이터를 생성하고 해당 데이터를 이벤트 허브 인스턴스로 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-163">This tutorial also provides a .NET client application to generate sample phone call metadata and send that data to an event hub instance.</span></span> 

## <a name="pricing"></a><span data-ttu-id="35e3e-164">가격</span><span class="sxs-lookup"><span data-stu-id="35e3e-164">Pricing</span></span>

<span data-ttu-id="35e3e-165">이 솔루션을 실행하는 비용을 알아보기 위해 모든 서비스가 비용 계산기에서 미리 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-165">To explore the cost of running this solution, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="35e3e-166">특정 사용 사례에 대한 가격이 변경되는 정도를 확인하려면 필요한 데이터 양에 맞게 적절한 변수를 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-166">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected data volume.</span></span>

<span data-ttu-id="35e3e-167">가져오는 데 필요한 트래픽 양을 기준으로 다음 세 가지 샘플 비용 프로필을 제공했습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-167">We have provided three sample cost profiles based on amount of traffic you expect to get:</span></span>

* <span data-ttu-id="35e3e-168">[소량][small-pricing]: 매월 1개 표준 스트리밍 단위를 통해 백만 개의 이벤트를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-168">[Small][small-pricing]: process one million events through one standard streaming unit per month.</span></span>
* <span data-ttu-id="35e3e-169">[중간][medium-pricing]: 매월 5개 표준 스트리밍 단위를 통해 1억 개의 이벤트를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-169">[Medium][medium-pricing]: process 100M events through five standard streaming units per month.</span></span>
* <span data-ttu-id="35e3e-170">[대량][large-pricing]: 매월 20개 표준 스트리밍 단위를 통해 9억 9천 9백만 개의 이벤트를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-170">[Large][large-pricing]: process 999 million events through 20 standard streaming units per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="35e3e-171">관련 리소스</span><span class="sxs-lookup"><span data-stu-id="35e3e-171">Related resources</span></span>

<span data-ttu-id="35e3e-172">더 복잡한 부정 행위 감지 시나리오는 기계 학습 모델을 통해 이점을 누릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="35e3e-172">More complex fraud detection scenarios can benefit from a machine learning model.</span></span> <span data-ttu-id="35e3e-173">Machine Learning Server를 사용하여 구축된 솔루션은 [Machine Learning Server를 사용하여 부정 행위 감지][r-server-fraud-detection]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="35e3e-173">For solutions built using Machine Learning Server, see [Fraud detection using Machine Learning Server][r-server-fraud-detection].</span></span> <span data-ttu-id="35e3e-174">Machine Learning Server를 사용하는 다른 솔루션 템플릿은 [데이터 과학 시나리오 및 솔루션 템플릿][docs-r-server-sample-solutions]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="35e3e-174">For other solution templates using Machine Learning Server, see [Data science scenarios and solution templates][docs-r-server-sample-solutions].</span></span> <span data-ttu-id="35e3e-175">Azure Data Lake Analytics를 사용하는 예제 솔루션은 [부정 행위 감지에 Azure Data Lake 및 R 사용][technet-fraud-detection]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="35e3e-175">For an example solution using Azure Data Lake Analytics, see [Using Azure Data Lake and R for Fraud Detection][technet-fraud-detection].</span></span>  

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[tutorial]: /azure/stream-analytics/stream-analytics-real-time-fraud-detection
[small-pricing]: https://azure.com/e/74149ec312c049ccba79bfb3cfa67606
[medium-pricing]: https://azure.com/e/4fc94f7376de484d8ae67a6958cae60a
[large-pricing]: https://azure.com/e/7da8804396f9428a984578700003ba42
[architecture-diagram]: ./images/architecture-diagram-fraud-detection.png
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-event-hubs-security-model]: /azure/event-hubs/event-hubs-authentication-and-security-model-overview
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-blob-storage]: /azure/storage/blobs/storage-blobs-introduction
[docs-r-server-sample-solutions]: /machine-learning-server/r/sample-solutions
[r-server-fraud-detection]: https://microsoft.github.io/r-server-fraud-detection/
[technet-fraud-detection]: https://blogs.technet.microsoft.com/machinelearning/2017/06/28/using-azure-data-lake-and-r-for-fraud-detection/
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/

