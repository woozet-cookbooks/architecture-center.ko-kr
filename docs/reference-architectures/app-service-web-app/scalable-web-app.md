---
title: 확장 가능한 웹 응용 프로그램
description: Microsoft Azure에서 실행되는 웹 응용 프로그램의 확장성을 향상합니다.
author: MikeWasson
pnp.series.title: Azure App Service
pnp.series.prev: basic-web-app
pnp.series.next: multi-region-web-app
ms.date: 11/23/2016
cardTitle: Improve scalability
ms.openlocfilehash: 6459acebfa25491332e2118b9e8fe51d5fc79ff3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846511"
---
# <a name="improve-scalability-in-a-web-application"></a><span data-ttu-id="4fe9b-103">웹 응용 프로그램의 확장성 향상</span><span class="sxs-lookup"><span data-stu-id="4fe9b-103">Improve scalability in a web application</span></span>

<span data-ttu-id="4fe9b-104">이 참조 아키텍처는 Azure App Service 웹 응용 프로그램의 확장성과 성능 향상을 위한 검증된 사례를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-104">This reference architecture shows proven practices for improving scalability and performance in an Azure App Service web application.</span></span>

<span data-ttu-id="4fe9b-105">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="4fe9b-105">![[0]][0]</span></span>

<span data-ttu-id="4fe9b-106">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="4fe9b-106">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="4fe9b-107">건축</span><span class="sxs-lookup"><span data-stu-id="4fe9b-107">Architecture</span></span>  

<span data-ttu-id="4fe9b-108">이 아키텍처는 [기본 웹 응용 프로그램][basic-web-app]에 표시된 아키텍처를 기반으로 합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-108">This architecture builds on the one shown in [Basic web application][basic-web-app].</span></span> <span data-ttu-id="4fe9b-109">다음 구성 요소가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-109">It includes the following components:</span></span>

* <span data-ttu-id="4fe9b-110">**리소스 그룹**.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-110">**Resource group**.</span></span> <span data-ttu-id="4fe9b-111">[리소스 그룹][resource-group]은 Azure 리소스에 대한 논리적 컨테이너입니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-111">A [resource group][resource-group] is a logical container for Azure resources.</span></span>
* <span data-ttu-id="4fe9b-112">**[웹앱][app-service-web-app]** 및 **[API 앱][app-service-api-app]**.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-112">**[Web app][app-service-web-app]** and **[API app][app-service-api-app]**.</span></span> <span data-ttu-id="4fe9b-113">일반적인 최신 응용 프로그램에는 웹 사이트와 하나 이상의 RESTful 웹 API가 모두 포함되어 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-113">A typical modern application might include both a website and one or more RESTful web APIs.</span></span> <span data-ttu-id="4fe9b-114">AJAX를 통한 브라우저 클라이언트, 기본 클라이언트 응용 프로그램 또는 서버 쪽 응용 프로그램에서 웹 API를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-114">A web API might be consumed by browser clients through AJAX, by native client applications, or by server-side applications.</span></span> <span data-ttu-id="4fe9b-115">웹 API 디자인에 대한 고려 사항은 [API 디자인 지침][api-guidance]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-115">For considerations on designing web APIs, see [API design guidance][api-guidance].</span></span>    
* <span data-ttu-id="4fe9b-116">**WebJob**.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-116">**WebJob**.</span></span> <span data-ttu-id="4fe9b-117">백그라운드에서 장기 실행 작업을 실행하려면 [Azure WebJobs][webjobs]를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-117">Use [Azure WebJobs][webjobs] to run long-running tasks in the background.</span></span> <span data-ttu-id="4fe9b-118">WebJob은 예약에 따라, 연속적으로 또는 큐에 메시지를 넣는 등 트리거에 대한 응답으로 실행될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-118">WebJobs can run on a schedule, continously, or in response to a trigger, such as putting a message on a queue.</span></span> <span data-ttu-id="4fe9b-119">WebJob은 App Service 앱의 컨텍스트에서 백그라운드 프로세스로 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-119">A WebJob runs as a background process in the context of an App Service app.</span></span>
* <span data-ttu-id="4fe9b-120">**큐**.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-120">**Queue**.</span></span> <span data-ttu-id="4fe9b-121">여기에 표시된 아키텍처에서는 응용 프로그램이 [Azure Queue Storage][queue-storage] 큐에 메시지를 넣어 백그라운드 작업을 큐에 넣습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-121">In the architecture shown here, the application queues background tasks by putting a message onto an [Azure Queue storage][queue-storage] queue.</span></span> <span data-ttu-id="4fe9b-122">메시지가 WebJob의 함수를 트리거합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-122">The message triggers a function in the WebJob.</span></span> <span data-ttu-id="4fe9b-123">또는 Service Bus 큐를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-123">Alternatively, you can use Service Bus queues.</span></span> <span data-ttu-id="4fe9b-124">비교하려면 [Azure 큐 및 Service Bus 큐 - 비교 및 대조][queues-compared]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-124">For a comparison, see [Azure Queues and Service Bus queues - compared and contrasted][queues-compared].</span></span>
* <span data-ttu-id="4fe9b-125">**캐시**.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-125">**Cache**.</span></span> <span data-ttu-id="4fe9b-126">[Azure Redis Cache][azure-redis]의 반정적 데이터를 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-126">Store semi-static data in [Azure Redis Cache][azure-redis].</span></span>  
* <span data-ttu-id="4fe9b-127"><strong>CDN</strong>.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-127"><strong>CDN</strong>.</span></span> <span data-ttu-id="4fe9b-128">[Azure CDN(Content Delivery Network)][azure-cdn]을 사용하여 지연 시간을 단축하고 더 신속한 콘텐츠 배달을 위해 공개적으로 사용 가능한 콘텐츠를 캐시합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-128">Use [Azure Content Delivery Network][azure-cdn] (CDN) to cache publicly available content for lower latency and faster delivery of content.</span></span>
* <span data-ttu-id="4fe9b-129">**데이터 저장소**.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-129">**Data storage**.</span></span> <span data-ttu-id="4fe9b-130">관계형 데이터의 경우 [Azure SQL Database][sql-db]를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-130">Use [Azure SQL Database][sql-db] for relational data.</span></span> <span data-ttu-id="4fe9b-131">비관계형 데이터의 경우 [Cosmos DB][cosmosdb] 같은 NoSQL 저장소를 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-131">For non-relational data, consider a NoSQL store, such as [Cosmos DB][cosmosdb].</span></span>
* <span data-ttu-id="4fe9b-132">**Azure Search**.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-132">**Azure Search**.</span></span> <span data-ttu-id="4fe9b-133">[Azure Search][azure-search]를 사용하여 검색 제안, 유사 항목 검색 및 언어별 검색과 같은 검색 기능을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-133">Use [Azure Search][azure-search] to add search functionality such as search suggestions, fuzzy search, and language-specific search.</span></span> <span data-ttu-id="4fe9b-134">Azure Search는 일반적으로 다른 데이터 저장소와 함께 사용되는데, 특히 기본 데이터 저장소에 엄격한 일관성이 필요한 경우 그렇습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-134">Azure Search is typically used in conjunction with another data store, especially if the primary data store requires strict consistency.</span></span> <span data-ttu-id="4fe9b-135">이러한 접근 방식에서는 신뢰할 수 있는 데이터를 다른 데이터 저장소에 저장하고 검색 인덱스를 Azure Search에 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-135">In this approach, store authoritative data in the other data store and the search index in Azure Search.</span></span> <span data-ttu-id="4fe9b-136">또한 Azure Search는 여러 데이터 저장소의 단일 검색 인덱스를 통합하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-136">Azure Search can also be used to consolidate a single search index from multiple data stores.</span></span>  
* <span data-ttu-id="4fe9b-137">**메일/SMS**.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-137">**Email/SMS**.</span></span> <span data-ttu-id="4fe9b-138">SendGrid 또는 Twilio와 같은 타사 서비스를 사용하여 응용 프로그램에 직접 이 기능을 빌드하는 대신 메일이나 SMS 메시지를 전송합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-138">Use a third-party service such as SendGrid or Twilio to send email or SMS messages instead of building this functionality directly into the application.</span></span>
* <span data-ttu-id="4fe9b-139">**Azure DNS**.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-139">**Azure DNS**.</span></span> <span data-ttu-id="4fe9b-140">[Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-140">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="4fe9b-141">Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-141">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

## <a name="recommendations"></a><span data-ttu-id="4fe9b-142">권장 사항</span><span class="sxs-lookup"><span data-stu-id="4fe9b-142">Recommendations</span></span>

<span data-ttu-id="4fe9b-143">개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-143">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="4fe9b-144">이 섹션의 권장 사항을 시작점으로 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-144">Use the recommendations in this section as a starting point.</span></span>

### <a name="app-service-apps"></a><span data-ttu-id="4fe9b-145">App Service 앱</span><span class="sxs-lookup"><span data-stu-id="4fe9b-145">App Service apps</span></span>
<span data-ttu-id="4fe9b-146">웹 응용 프로그램과 웹 API를 별도의 App Service 앱으로 만드는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-146">We recommend creating the web application and the web API as separate App Service apps.</span></span> <span data-ttu-id="4fe9b-147">이렇게 디자인하면 이들을 별도의 App Service 계획에서 실행할 수 있으므로 독립적으로 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-147">This design lets you run them in separate App Service plans so they can be scaled independently.</span></span> <span data-ttu-id="4fe9b-148">처음에 이러한 수준의 확장성이 필요하지 않은 경우 앱을 동일한 계획에 배포하고 필요한 경우 나중에 별도의 계획으로 이동할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-148">If you don't need that level of scalability initially, you can deploy the apps into the same plan and move them into separate plans later if necessary.</span></span>

> [!NOTE]
> <span data-ttu-id="4fe9b-149">Basic, Standard 및 Premium 계획은 앱 단위가 아니라 계획의 VM 인스턴스에 대해 청구됩니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-149">For the Basic, Standard, and Premium plans, you are billed for the VM instances in the plan, not per app.</span></span> <span data-ttu-id="4fe9b-150">[App Service 가격 책정][app-service-pricing]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-150">See [App Service Pricing][app-service-pricing]</span></span>
> 
> 

<span data-ttu-id="4fe9b-151">App Service Mobile Apps의 *간편한 테이블* 또는 *간편한 API* 기능을 사용하려는 경우 이를 위해 별도의 App Service 앱을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-151">If you intend to use the *Easy Tables* or *Easy APIs* features of App Service Mobile Apps, create a separate App Service app for this purpose.</span></span>  <span data-ttu-id="4fe9b-152">이러한 기능은 특정 응용 프로그램 프레임워크에 의존하여 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-152">These features rely on a specific application framework to enable them.</span></span>

### <a name="webjobs"></a><span data-ttu-id="4fe9b-153">웹 작업</span><span class="sxs-lookup"><span data-stu-id="4fe9b-153">WebJobs</span></span>
<span data-ttu-id="4fe9b-154">별도의 App Service 계획 내에 있는 빈 App Service 앱에 리소스를 많이 사용하는 WebJob을 배포하는 것을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-154">Consider deploying resource intensive WebJobs to an empty App Service app within a separate App Service plan.</span></span> <span data-ttu-id="4fe9b-155">이렇게 하면 WebJob에 전용 인스턴스가 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-155">This provides dedicated instances for the WebJob.</span></span> <span data-ttu-id="4fe9b-156">[백그라운드 작업 지침][webjobs-guidance]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-156">See [Background jobs guidance][webjobs-guidance].</span></span>  

### <a name="cache"></a><span data-ttu-id="4fe9b-157">캐시</span><span class="sxs-lookup"><span data-stu-id="4fe9b-157">Cache</span></span>
<span data-ttu-id="4fe9b-158">[Azure Redis Cache][azure-redis]를 사용하여 일부 데이터를 캐시하면 성능과 확장성을 향상할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-158">You can improve performance and scalability by using [Azure Redis Cache][azure-redis] to cache some data.</span></span> <span data-ttu-id="4fe9b-159">다음에 대해 Redis Cache 사용을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-159">Consider using Redis Cache for:</span></span>

* <span data-ttu-id="4fe9b-160">반정적 트랜잭션 데이터</span><span class="sxs-lookup"><span data-stu-id="4fe9b-160">Semi-static transaction data.</span></span>
* <span data-ttu-id="4fe9b-161">세션 상태</span><span class="sxs-lookup"><span data-stu-id="4fe9b-161">Session state.</span></span>
* <span data-ttu-id="4fe9b-162">HTML 출력</span><span class="sxs-lookup"><span data-stu-id="4fe9b-162">HTML output.</span></span> <span data-ttu-id="4fe9b-163">이는 복잡한 HTML 출력을 렌더링하는 응용 프로그램에 유용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-163">This can be useful in applications that render complex HTML output.</span></span>

<span data-ttu-id="4fe9b-164">캐싱 전략 디자인에 대한 자세한 지침은 [캐싱 지침][caching-guidance]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-164">For more detailed guidance on designing a caching strategy, see [Caching guidance][caching-guidance].</span></span>

### <a name="cdn"></a><span data-ttu-id="4fe9b-165">CDN</span><span class="sxs-lookup"><span data-stu-id="4fe9b-165">CDN</span></span>
<span data-ttu-id="4fe9b-166">[Azure CDN][azure-cdn]을 사용하여 정적 콘텐츠를 캐시합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-166">Use [Azure CDN][azure-cdn] to cache static content.</span></span> <span data-ttu-id="4fe9b-167">CDN의 주요 이점은 사용자의 대기 시간이 줄어드는 것인데 그 이유는 콘텐츠가 사용자와 지리적으로 가까운 에지 서버에 캐시되기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-167">The main benefit of a CDN is to reduce latency for users, because content is cached at an edge server that is geographically close to the user.</span></span> <span data-ttu-id="4fe9b-168">또한 응용 프로그램에 의해 트래픽이 처리되지 않기 때문에 CDN은 응용 프로그램의 부하를 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-168">CDN can also reduce load on the application, because that traffic is not being handled by the application.</span></span>

<span data-ttu-id="4fe9b-169">앱이 대부분 정적 페이지로 구성된 경우 [전체 앱을 캐시하는 데 CDN][cdn-app-service] 사용을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-169">If your app consists mostly of static pages, consider using [CDN to cache the entire app][cdn-app-service].</span></span> <span data-ttu-id="4fe9b-170">그렇지 않은 경우에는 이미지, CSS, HTML 파일 등의 정적 콘텐츠를 [Azure Storage에 넣고 CDN을 사용하여 이러한 파일을 캐시합니다][cdn-storage-account].</span><span class="sxs-lookup"><span data-stu-id="4fe9b-170">Otherwise, put static content such as images, CSS, and HTML files, into [Azure Storage and use CDN to cache those files][cdn-storage-account].</span></span>

> [!NOTE]
> <span data-ttu-id="4fe9b-171">Azure CDN은 인증이 필요한 콘텐츠를 제공할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-171">Azure CDN cannot serve content that requires authentication.</span></span>
> 
> 

<span data-ttu-id="4fe9b-172">자세한 지침은 [CDN(콘텐츠 배달 네트워크) 지침][cdn-guidance]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-172">For more detailed guidance, see [Content Delivery Network (CDN) guidance][cdn-guidance].</span></span>

### <a name="storage"></a><span data-ttu-id="4fe9b-173">Storage</span><span class="sxs-lookup"><span data-stu-id="4fe9b-173">Storage</span></span>
<span data-ttu-id="4fe9b-174">최신 응용 프로그램은 종종 많은 양의 데이터를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-174">Modern applications often process large amounts of data.</span></span> <span data-ttu-id="4fe9b-175">클라우드에 대한 크기를 조정하려면 올바른 저장소 유형을 선택해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-175">In order to scale for the cloud, it's important to choose the right storage type.</span></span> <span data-ttu-id="4fe9b-176">다음은 몇 가지 기본 권장 사항입니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-176">Here are some baseline recommendations.</span></span> 

| <span data-ttu-id="4fe9b-177">저장 형식</span><span class="sxs-lookup"><span data-stu-id="4fe9b-177">What you want to store</span></span> | <span data-ttu-id="4fe9b-178">예</span><span class="sxs-lookup"><span data-stu-id="4fe9b-178">Example</span></span> | <span data-ttu-id="4fe9b-179">권장 저장소</span><span class="sxs-lookup"><span data-stu-id="4fe9b-179">Recommended storage</span></span> |
| --- | --- | --- |
| <span data-ttu-id="4fe9b-180">파일</span><span class="sxs-lookup"><span data-stu-id="4fe9b-180">Files</span></span> |<span data-ttu-id="4fe9b-181">이미지, 문서, PDF</span><span class="sxs-lookup"><span data-stu-id="4fe9b-181">Images, documents, PDFs</span></span> |<span data-ttu-id="4fe9b-182">Azure Blob Storage</span><span class="sxs-lookup"><span data-stu-id="4fe9b-182">Azure Blob Storage</span></span> |
| <span data-ttu-id="4fe9b-183">키/값 쌍</span><span class="sxs-lookup"><span data-stu-id="4fe9b-183">Key/Value pairs</span></span> |<span data-ttu-id="4fe9b-184">사용자 ID로 조회된 사용자 프로필 데이터</span><span class="sxs-lookup"><span data-stu-id="4fe9b-184">User profile data looked up by user ID</span></span> |<span data-ttu-id="4fe9b-185">Azure 테이블 저장소</span><span class="sxs-lookup"><span data-stu-id="4fe9b-185">Azure Table storage</span></span> |
| <span data-ttu-id="4fe9b-186">추가 처리를 트리거할 짧은 메시지</span><span class="sxs-lookup"><span data-stu-id="4fe9b-186">Short messages intended to trigger further processing</span></span> |<span data-ttu-id="4fe9b-187">주문 요청</span><span class="sxs-lookup"><span data-stu-id="4fe9b-187">Order requests</span></span> |<span data-ttu-id="4fe9b-188">Azure Queue Storage, Service Bus 큐 또는 Service Bus 항목</span><span class="sxs-lookup"><span data-stu-id="4fe9b-188">Azure Queue storage, Service Bus queue, or Service Bus topic</span></span> |
| <span data-ttu-id="4fe9b-189">기본 쿼리를 요구하는 유연한 스키마를 사용하는 비관계형 데이터</span><span class="sxs-lookup"><span data-stu-id="4fe9b-189">Non-relational data with a flexible schema requiring basic querying</span></span> |<span data-ttu-id="4fe9b-190">제품 카탈로그</span><span class="sxs-lookup"><span data-stu-id="4fe9b-190">Product catalog</span></span> |<span data-ttu-id="4fe9b-191">Azure Cosmos DB, MongoDB 또는 Apache CouchDB와 같은 문서 데이터베이스</span><span class="sxs-lookup"><span data-stu-id="4fe9b-191">Document database, such as Azure Cosmos DB, MongoDB, or Apache CouchDB</span></span> |
| <span data-ttu-id="4fe9b-192">보다 풍부한 쿼리 지원, 엄격한 스키마 및/또는 강력한 일관성이 필요한 관계형 데이터</span><span class="sxs-lookup"><span data-stu-id="4fe9b-192">Relational data requiring richer query support, strict schema, and/or strong consistency</span></span> |<span data-ttu-id="4fe9b-193">제품 인벤토리</span><span class="sxs-lookup"><span data-stu-id="4fe9b-193">Product inventory</span></span> |<span data-ttu-id="4fe9b-194">Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="4fe9b-194">Azure SQL Database</span></span> |

## <a name="scalability-considerations"></a><span data-ttu-id="4fe9b-195">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="4fe9b-195">Scalability considerations</span></span>

<span data-ttu-id="4fe9b-196">Azure App Service의 주요 이점은 부하에 따라 응용 프로그램을 확장할 수 있다는 점입니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-196">A major benefit of Azure App Service is the ability to scale your application based on load.</span></span> <span data-ttu-id="4fe9b-197">다음은 응용 프로그램 확장을 계획할 때 염두할 몇 가지 고려 사항입니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-197">Here are some considerations to keep in mind when planning to scale your application.</span></span>

### <a name="app-service-app"></a><span data-ttu-id="4fe9b-198">App Service 앱</span><span class="sxs-lookup"><span data-stu-id="4fe9b-198">App Service app</span></span>
<span data-ttu-id="4fe9b-199">솔루션에 여러 App Service 앱이 포함되어 있는 경우 App Service 계획이 분리되도록 배포하는 것을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-199">If your solution includes several App Service apps, consider deploying them to separate App Service plans.</span></span> <span data-ttu-id="4fe9b-200">이러한 방식을 사용하면 개별 인스턴스에서 실행되므로 개별적으로 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-200">This approach enables you to scale them independently because they run on separate instances.</span></span> 

<span data-ttu-id="4fe9b-201">마찬가지로, 백그라운드 작업이 HTTP 요청을 처리하는 동일한 인스턴스에서 실행되지 않도록 WebJob을 고유한 계획에 넣는 것을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-201">Similarly, consider putting a WebJob into its own plan so that background tasks don't run on the same instances that handle HTTP requests.</span></span>  

### <a name="sql-database"></a><span data-ttu-id="4fe9b-202">SQL Database</span><span class="sxs-lookup"><span data-stu-id="4fe9b-202">SQL Database</span></span>
<span data-ttu-id="4fe9b-203">데이터베이스를 *분할*하여 SQL데이터베이스의 확장성을 높입니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-203">Increase scalability of a SQL database by *sharding* the database.</span></span> <span data-ttu-id="4fe9b-204">분할은 데이터베이스를 가로로 분할합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-204">Sharding refers to partitioning the database horizontally.</span></span> <span data-ttu-id="4fe9b-205">분할을 사용하면 [Elastic Database 도구][sql-elastic]로 데이터베이스를 가로로 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-205">Sharding allows you to scale out the database horizontally using [Elastic Database tools][sql-elastic].</span></span> <span data-ttu-id="4fe9b-206">분할의 잠재적 이점은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-206">Potential benefits of sharding include:</span></span>

- <span data-ttu-id="4fe9b-207">트랜잭션 처리량이 늘어납니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-207">Better transaction throughput.</span></span>
- <span data-ttu-id="4fe9b-208">쿼리가 데이터의 하위 집합에서 더 빨리 실행될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-208">Queries can run faster over a subset of the data.</span></span>

### <a name="azure-search"></a><span data-ttu-id="4fe9b-209">Azure Search</span><span class="sxs-lookup"><span data-stu-id="4fe9b-209">Azure Search</span></span>
<span data-ttu-id="4fe9b-210">Azure Search는 주요 데이터 저장소에서 복잡한 데이터 검색을 수행하는 데 따른 오버헤드를 제거하므로 부하를 처리하도록 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-210">Azure Search removes the overhead of performing complex data searches from the primary data store, and it can scale to handle load.</span></span> <span data-ttu-id="4fe9b-211">[Azure Search의 쿼리 및 인덱싱 작업을 위한 리소스 수준 확장][azure-search-scaling]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-211">See [Scale resource levels for query and indexing workloads in Azure Search][azure-search-scaling].</span></span>

## <a name="security-considerations"></a><span data-ttu-id="4fe9b-212">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="4fe9b-212">Security considerations</span></span>
<span data-ttu-id="4fe9b-213">이 섹션에는 이 문서에 설명된 Azure 서비스와 관련된 보안 고려 사항이 나와 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-213">This section lists security considerations that are specific to the Azure services described in this article.</span></span> <span data-ttu-id="4fe9b-214">보안 모범 사례가 완전히 다 나와 있는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-214">It's not a complete list of security best practices.</span></span> <span data-ttu-id="4fe9b-215">몇 가지 추가 보안 고려 사항은 [Azure App Service에서 앱 보안][app-service-security]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-215">For some additional security considerations, see [Secure an app in Azure App Service][app-service-security].</span></span>

### <a name="cross-origin-resource-sharing-cors"></a><span data-ttu-id="4fe9b-216">CORS(크로스-원본 자원 공유)</span><span class="sxs-lookup"><span data-stu-id="4fe9b-216">Cross-Origin Resource Sharing (CORS)</span></span>
<span data-ttu-id="4fe9b-217">웹 사이트와 웹 API를 별도의 앱으로 만드는 경우 CORS를 사용하도록 설정하지 않으면 클라이언트 쪽 AJAX에서 API를 호출할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-217">If you create a website and web API as separate apps, the website cannot make client-side AJAX calls to the API unless you enable CORS.</span></span>

> [!NOTE]
> <span data-ttu-id="4fe9b-218">브라우저 보안은 웹 페이지에서 다른 도메인으로 AJAX 요청을 수행하지 못하도록 방지합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-218">Browser security prevents a web page from making AJAX requests to another domain.</span></span> <span data-ttu-id="4fe9b-219">이렇게 제한하는 것을 동일 원본 정책이라고 하며, 악성 사이트에서 다른 사이트의 중요한 데이터를 읽을 수 없도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-219">This restriction is called the same-origin policy, and prevents a malicious site from reading sentitive data from another site.</span></span> <span data-ttu-id="4fe9b-220">CORS는 서버에서 동일 원본 정책을 완화하고 크로스-원본 요청을 일부는 허용하고 일부는 거부할 수 있는 W3C표준입니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-220">CORS is a W3C standard that allows a server to relax the same-origin policy and allow some cross-origin requests while rejecting others.</span></span>
> 
> 

<span data-ttu-id="4fe9b-221">App Services에서는 응용 프로그램 코드를 작성할 필요 없이 기본적으로 CORS를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-221">App Services has built-in support for CORS, without needing to write any application code.</span></span> <span data-ttu-id="4fe9b-222">[CORS를 사용하여 JavaScript에서 API 앱 사용][cors]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-222">See [Consume an API app from JavaScript using CORS][cors].</span></span> <span data-ttu-id="4fe9b-223">API에 대해 허용되는 원본 목록에 웹 사이트를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-223">Add the website to the list of allowed origins for the API.</span></span>

### <a name="sql-database-encryption"></a><span data-ttu-id="4fe9b-224">SQL Database 암호화</span><span class="sxs-lookup"><span data-stu-id="4fe9b-224">SQL Database encryption</span></span>
<span data-ttu-id="4fe9b-225">데이터베이스에서 미사용 데이터를 암호화해야 하는 경우 [투명한 데이터 암호화][sql-encryption]를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-225">Use [Transparent Data Encryption][sql-encryption] if you need to encrypt data at rest in the database.</span></span> <span data-ttu-id="4fe9b-226">이 기능은 전체 데이터베이스(백업 및 트랜잭션 로그 파일 포함)의 실시간 암호화 및 암호 해독을 수행하며 응용 프로그램을 변경할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-226">This feature performs real-time encryption and decryption of an entire database (including backups and transaction log files) and requires no changes to the application.</span></span> <span data-ttu-id="4fe9b-227">암호화를 수행하면 대기 시간이 늘어나므로, 고유한 데이터베이스에 보호해야 하는 데이터를 분리하고 해당 데이터베이스에 대해서만 암호화를 사용하도록 설정하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="4fe9b-227">Encryption does add some latency, so it's a good practice to separate the data that must be secure into its own database and enable encryption only for that database.</span></span>  
  

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-dns]: /azure/dns/dns-overview
[azure-redis]: https://azure.microsoft.com/services/cache/
[azure-search]: https://azure.microsoft.com/documentation/services/search/
[azure-search-scaling]: /azure/search/search-capacity-planning
[background-jobs]: ../../best-practices/background-jobs.md
[basic-web-app]: basic-web-app.md
[basic-web-app-scalability]: basic-web-app.md#scalability-considerations
[caching-guidance]: ../../best-practices/caching.md
[cdn-app-service]: /azure/app-service-web/cdn-websites-with-cdn
[cdn-storage-account]: /azure/cdn/cdn-create-a-storage-account-with-cdn
[cdn-guidance]: ../../best-practices/cdn.md
[cors]: /azure/app-service-api/app-service-api-cors-consume-javascript
[cosmosdb]: /azure/cosmos-db/
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
[webjobs-guidance]: ../../best-practices/background-jobs.md
[webjobs]: /azure/app-service/app-service-webjobs-readme
[0]: ./images/scalable-web-app.png "확장성이 향상된 Azure의 웹 응용 프로그램"
