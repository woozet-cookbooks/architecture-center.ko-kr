---
title: 웹 큐 작업자 아키텍처 스타일
description: Azure에서 웹 큐 작업자 아키텍처의 이점, 과제 및 모범 사례를 설명합니다.
author: MikeWasson
ms.openlocfilehash: 545472e71ffcd43717ad24af0dc9218a221ca910
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/10/2018
ms.locfileid: "37958792"
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="1913f-103">웹 큐 작업자 아키텍처 스타일</span><span class="sxs-lookup"><span data-stu-id="1913f-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="1913f-104">이 아키텍처의 핵심 구성 요소는 클라이언트 요청을 처리하는 **웹 프런트 엔드**와 리소스 집약적 작업, 장기 실행 워크플로 또는 일괄 처리 작업을 수행하는 **작업자**입니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="1913f-105">웹 프런트 엔드는 **메시지 큐**를 통해 작업자와 통신합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-105">The web front end communicates with the worker through a **message queue**.</span></span>  

![](./images/web-queue-worker-logical.svg)

<span data-ttu-id="1913f-106">이 아키텍처에 공통적으로 통합되는 기타 구성 요소는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-106">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="1913f-107">하나 이상의 데이터베이스</span><span class="sxs-lookup"><span data-stu-id="1913f-107">One or more databases.</span></span> 
- <span data-ttu-id="1913f-108">빠른 읽기를 위해 데이터베이스의 값을 저장하는 캐시</span><span class="sxs-lookup"><span data-stu-id="1913f-108">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="1913f-109">정적 콘텐츠를 제공하기 위한 CDN</span><span class="sxs-lookup"><span data-stu-id="1913f-109">CDN to serve static content</span></span>
- <span data-ttu-id="1913f-110">원격 서비스(예: 전자 메일 또는 SMS 서비스)</span><span class="sxs-lookup"><span data-stu-id="1913f-110">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="1913f-111">종종 이러한 서비스는 타사에서 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-111">Often these are provided by third parties.</span></span>
- <span data-ttu-id="1913f-112">인증을 위한 ID 공급자</span><span class="sxs-lookup"><span data-stu-id="1913f-112">Identity provider for authentication.</span></span>

<span data-ttu-id="1913f-113">웹 및 작업자는 둘 다 상태 비저장입니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-113">The web and worker are both stateless.</span></span> <span data-ttu-id="1913f-114">세션 상태는 분산된 캐시에 저장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-114">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="1913f-115">모든 장기 실행 작업은 작업자에 의해 비동기적으로 수행됩니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-115">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="1913f-116">작업자는 큐의 메시지에 의해 트리거되거나, 일괄 처리 일정에 따라 실행될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-116">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="1913f-117">작업자는 선택적 구성 요소입니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-117">The worker is an optional component.</span></span> <span data-ttu-id="1913f-118">장기 실행 작업이 없는 경우 작업자를 생략할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-118">If there are no long-running operations, the worker can be omitted.</span></span>  

<span data-ttu-id="1913f-119">프런트 엔드는 웹 API로 구성될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-119">The front end might consist of a web API.</span></span> <span data-ttu-id="1913f-120">클라이언트 쪽에서 웹 API는 AJAX 호출을 하는 단일 페이지 응용 프로그램 또는 네이티브 클라이언트 응용 프로그램에서 사용될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-120">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="1913f-121">이 아키텍처를 사용하는 경우</span><span class="sxs-lookup"><span data-stu-id="1913f-121">When to use this architecture</span></span>

<span data-ttu-id="1913f-122">일반적으로 웹 큐 작업자 아키텍처는 관리되는 계산 서비스(Azure App Service 또는 Azure Cloud Services)를 사용하여 구현됩니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-122">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span> 

<span data-ttu-id="1913f-123">다음과 같은 경우 이 아키텍처 스타일을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-123">Consider this architecture style for:</span></span>

- <span data-ttu-id="1913f-124">비교적 간단한 도메인이 있는 응용 프로그램</span><span class="sxs-lookup"><span data-stu-id="1913f-124">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="1913f-125">장기 실행 워크플로 또는 일괄 처리 작업을 일부 포함하는 응용 프로그램</span><span class="sxs-lookup"><span data-stu-id="1913f-125">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="1913f-126">IaaS(Infrastructure as a Service)가 아닌 관리되는 서비스를 사용하려는 경우</span><span class="sxs-lookup"><span data-stu-id="1913f-126">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="1913f-127">이점</span><span class="sxs-lookup"><span data-stu-id="1913f-127">Benefits</span></span>

- <span data-ttu-id="1913f-128">이해하기 쉬운 비교적 간단한 아키텍처입니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-128">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="1913f-129">쉽게 배포 및 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-129">Easy to deploy and manage.</span></span>
- <span data-ttu-id="1913f-130">문제가 명확히 구분됩니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-130">Clear separation of concerns.</span></span>
- <span data-ttu-id="1913f-131">프런트 엔드는 비동기 메시징을 사용하여 작업자에서 분리됩니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-131">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="1913f-132">프런트 엔드 및 작업자 크기는 독립적으로 조정됩니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-132">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="1913f-133">과제</span><span class="sxs-lookup"><span data-stu-id="1913f-133">Challenges</span></span>

- <span data-ttu-id="1913f-134">주의깊게 디자인하지 않으면 프런트 엔드 및 작업자가 유지 관리 및 업데이트가 어려운 한 덩어리의 커다란 구성 요소로 변할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-134">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="1913f-135">프런트 엔드 및 작업자가 데이터 스키마 또는 코드 모듈을 공유하는 경우 숨겨진 종속성이 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-135">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span> 

## <a name="best-practices"></a><span data-ttu-id="1913f-136">모범 사례</span><span class="sxs-lookup"><span data-stu-id="1913f-136">Best practices</span></span>

- <span data-ttu-id="1913f-137">클라이언트에 잘 디자인된 API를 노출합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-137">Expose a well-designed API to the client.</span></span> <span data-ttu-id="1913f-138">[API 디자인 모범 사례][api-design]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="1913f-138">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="1913f-139">부하 변경을 처리하도록 자동으로 크기가 조정됩니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-139">Autoscale to handle changes in load.</span></span> <span data-ttu-id="1913f-140">[자동 크기 조정 모범 사례][autoscaling]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="1913f-140">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="1913f-141">반정적 데이터를 캐시합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-141">Cache semi-static data.</span></span> <span data-ttu-id="1913f-142">[캐싱 모범 사례][caching]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="1913f-142">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="1913f-143">CDN을 사용하여 정적 콘텐츠를 호스트합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-143">Use a CDN to host static content.</span></span> <span data-ttu-id="1913f-144">[CDN 모범 사례][cdn]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="1913f-144">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="1913f-145">적절한 경우 polyglot 지속성을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-145">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="1913f-146">[작업에 가장 적합한 데이터 저장소 사용][polyglot]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="1913f-146">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="1913f-147">데이터를 분할하여 확장성을 향상시키고 경합을 줄여 성능을 최적화합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-147">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="1913f-148">[데이터 분할 모범 사례][data-partition]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="1913f-148">See [Data partitioning best practices][data-partition].</span></span>


## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="1913f-149">Azure App Service의 웹 큐 작업자</span><span class="sxs-lookup"><span data-stu-id="1913f-149">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="1913f-150">이 섹션에서는 Azure App Service를 사용하는 권장 웹 큐 작업자 아키텍처에 대해 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-150">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span> 

![](./images/web-queue-worker-physical.png)

<span data-ttu-id="1913f-151">프런트 엔드는 Azure App Service 웹앱으로 구현되고, 작업자는 웹 작업으로 구현됩니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-151">The front end is implemented as an Azure App Service web app, and the worker is implemented as a WebJob.</span></span> <span data-ttu-id="1913f-152">웹앱과 웹 작업은 둘 다 VM 인스턴스를 제공하는 App Service 계획에 연결되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-152">The web app and the WebJob are both associated with an App Service plan that provides the VM instances.</span></span> 

<span data-ttu-id="1913f-153">메시지 큐에 대해 Azure Service Bus 또는 Azure Atorage 큐를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-153">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="1913f-154">(이 다이어그램은 Azure Storage 큐를 보여 줍니다.)</span><span class="sxs-lookup"><span data-stu-id="1913f-154">(The diagram shows an Azure Storage queue.)</span></span>

<span data-ttu-id="1913f-155">Azure Redis Cache는 낮은 대기 시간 액세스가 필요한 기타 데이터 및 세션 상태를 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-155">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

<span data-ttu-id="1913f-156">Azure CDN은 이미지, CSS 또는 HTML과 같은 정적 콘텐츠를 캐시하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-156">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

<span data-ttu-id="1913f-157">저장소의 경우, 응용 프로그램의 요구에 가장 잘 맞는 저장소 기술을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-157">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="1913f-158">여러 저장소 기술(polyglot 지속성)을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-158">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="1913f-159">이 아이디어를 설명하기 위해 이 다이어그램은 Azure SQL Database 및 Azure Cosmos DB를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-159">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>  

<span data-ttu-id="1913f-160">자세한 내용은 [App Service 웹 응용 프로그램 참조 아키텍처][scalable-web-app]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="1913f-160">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="1913f-161">추가 고려 사항</span><span class="sxs-lookup"><span data-stu-id="1913f-161">Additional considerations</span></span>

- <span data-ttu-id="1913f-162">모든 트랜잭션이 큐 및 작업자를 거쳐 저장소로 이동해야 하는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-162">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="1913f-163">웹 프런트 엔드는 간단한 읽기/쓰기 작업을 직접 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-163">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="1913f-164">작업자는 리소스 집약적인 작업 또는 장기 실행 워크플로를 위해 디자인되었습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-164">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="1913f-165">경우에 따라 작업자가 전혀 필요하지 않을 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-165">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="1913f-166">App Service의 기본 제공 자동 크기 조정 기능을 사용하여 VM 인스턴스 수를 스케일 아웃합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-166">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="1913f-167">응용 프로그램의 부하가 예측 가능한 패턴을 따를 경우 일정 기반 자동 크기 조정을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-167">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="1913f-168">부하를 예측할 수 없는 경우 메트릭 기반 자동 크기 조정 규칙을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-168">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>      

- <span data-ttu-id="1913f-169">웹앱 및 웹 작업을 별도의 App Service 계획에 포함하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-169">Consider putting the web app and the WebJob into separate App Service plans.</span></span> <span data-ttu-id="1913f-170">이런 방식으로 이러한 항목이 별도 VM 인스턴스에 호스트되고, 독립적으로 크기가 조정될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-170">That way, they are hosted on separate VM instances and can be scaled independently.</span></span> 

- <span data-ttu-id="1913f-171">프로덕션 및 테스트에 대해 별도 App Service 계획을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-171">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="1913f-172">그렇지 않고 프로덕션 및 테스트에 동일한 계획을 사용하는 것은 프로덕션 VM에서 테스트를 실행하는 것을 의미합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-172">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="1913f-173">배포 슬롯을 사용하여 배포를 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-173">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="1913f-174">이렇게 하면 업데이트된 버전을 스테이징 슬롯에 배포한 다음 새 버전으로 교체할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-174">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="1913f-175">또한 업데이트에 문제가 발생하면 이전 버전으로 다시 교체할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="1913f-175">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md