---
title: 프런트 엔드에 대한 백 엔드 패턴
description: 특정 프런트 엔드 응용 프로그램 또는 인터페이스에서 사용할 별도의 백 엔드 서비스를 만듭니다.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 2991d7a3e05b3ce6cd5148a552bae6d4ba8f7c4c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/30/2018
ms.locfileid: "30270022"
---
# <a name="backends-for-frontends-pattern"></a><span data-ttu-id="39200-103">프런트 엔드에 대한 백 엔드 패턴</span><span class="sxs-lookup"><span data-stu-id="39200-103">Backends for Frontends pattern</span></span>

<span data-ttu-id="39200-104">특정 프런트 엔드 응용 프로그램 또는 인터페이스에서 사용할 별도의 백 엔드 서비스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="39200-104">Create separate backend services to be consumed by specific frontend applications or interfaces.</span></span> <span data-ttu-id="39200-105">이 패턴은 단일 백 엔드를 여러 인터페이스에 맞게 사용자 지정하지 않으려는 경우에 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-105">This pattern is useful when you want to avoid customizing a single backend for multiple interfaces.</span></span> <span data-ttu-id="39200-106">이 패턴은 Sam Newman이 처음으로 설명했습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-106">This pattern was first described by Sam Newman.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="39200-107">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="39200-107">Context and problem</span></span>

<span data-ttu-id="39200-108">응용 프로그램은 처음에 데스크톱 웹 UI에서 대상으로 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-108">An application may initially be targeted at a desktop web UI.</span></span> <span data-ttu-id="39200-109">일반적으로 해당 UI에 필요한 기능을 제공하는 백 엔드 서비스는 병렬로 개발됩니다.</span><span class="sxs-lookup"><span data-stu-id="39200-109">Typically, a backend service is developed in parallel that provides the features needed for that UI.</span></span> <span data-ttu-id="39200-110">응용 프로그램의 사용자 기반이 확장되면 동일한 백 엔드와 상호 작용해야 하는 모바일 응용 프로그램이 개발됩니다.</span><span class="sxs-lookup"><span data-stu-id="39200-110">As the application's user base grows, a mobile application is developed that must interact with the same backend.</span></span> <span data-ttu-id="39200-111">이 백 엔드 서비스는 데스크톱 및 모바일 인터페이스의 요구 사항을 둘 다 처리하는 범용 백엔드가 됩니다.</span><span class="sxs-lookup"><span data-stu-id="39200-111">The backend service becomes a general-purpose backend, serving the requirements of both the desktop and mobile interfaces.</span></span>

<span data-ttu-id="39200-112">하지만 모바일 장치의 기능은 화면 크기, 성능 및 디스플레이 제한 측면에서 데스크톱 브라우저와 크게 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="39200-112">But the capabilities of a mobile device differ significantly from a desktop browser, in terms of screen size, performance, and display limitations.</span></span> <span data-ttu-id="39200-113">결과적으로 모바일 응용 프로그램 백 엔드에 대한 요구 사항은 데스크톱 웹 UI와 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="39200-113">As a result, the requirements for a mobile application backend differ from the desktop web UI.</span></span> 

<span data-ttu-id="39200-114">이러한 차이점 때문에 백 엔드에 대해 상충되는 요구 사항이 발생하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="39200-114">These differences result in competing requirements for the backend.</span></span> <span data-ttu-id="39200-115">백 엔드는 데스크톱 웹 UI 및 모바일 응용 프로그램을 둘 다 서비스하기 위해 정기적인 중요한 변경이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-115">The backend requires regular and significant changes to serve both the desktop web UI and the mobile application.</span></span> <span data-ttu-id="39200-116">종종 별도의 인터페이스 팀이 각 프런트 엔드에서 작업하므로, 백 엔드가 개발 프로세스의 병목 지점이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-116">Often, separate interface teams work on each frontend, causing the backend to become a bottleneck in the development process.</span></span> <span data-ttu-id="39200-117">업데이트 요구 사항이 충돌하고, 서비스가 프런트 엔드 둘 다에서 작동되도록 해야 하기 때문에 배포 가능한 단일 리소스에 많은 노력을 투입될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-117">Conflicting update requirements, and the need to keep the service working for both frontends, can result in spending a lot of effort on a single deployable resource.</span></span>

![](./_images/backend-for-frontend.png) 

<span data-ttu-id="39200-118">개발 활동이 백 엔드 서비스를 중심으로 진행되므로, 백 엔드를 관리하고 유지하기 위한 별도 팀을 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-118">As the development activity focuses on the backend service, a separate team may be created to manage and maintain the backend.</span></span> <span data-ttu-id="39200-119">결과적으로 이로 인해 인터페이스 및 백 엔드 개발 팀 사이에서 연결이 끊어지며, 백 엔드 팀은 여러 다른 UI 팀의 상충하는 요구 사항의 균형을 유지해야 하는 부담을 갖게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="39200-119">Ultimately, this results in a disconnect between the interface and backend development teams, placing a burden on the backend team to balance the competing requirements of the different UI teams.</span></span> <span data-ttu-id="39200-120">한 인터페이스 팀이 백 엔드를 변경해야 할 경우, 해당 변경 내용을 백 엔드에 통합하기 전에 먼저 다른 인터페이스 팀에서도 유효한지 확인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-120">When one interface team requires changes to the backend, those changes must be validated with other interface teams before they can be integrated into the backend.</span></span> 

## <a name="solution"></a><span data-ttu-id="39200-121">해결 방법</span><span class="sxs-lookup"><span data-stu-id="39200-121">Solution</span></span>

<span data-ttu-id="39200-122">사용자 인터페이스당 하나의 백 엔드를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="39200-122">Create one backend per user interface.</span></span> <span data-ttu-id="39200-123">다른 프런트 엔드 환경에 영향을 미치지 않고도 프런트 엔드 환경의 요구에 가장 잘 맞도록 각 백 엔드의 동작 및 성능을 미세 조정합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-123">Fine tune the behavior and performance of each backend to best match the needs of the frontend environment, without worrying about affecting other frontend experiences.</span></span>

![](./_images/backend-for-frontend-example.png) 

<span data-ttu-id="39200-124">각 백 엔드는 하나의 인터페이스에 맞춰지므로 해당 인터페이스에 대해 최적화할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-124">Because each backend is specific to one interface, it can be optimized for that interface.</span></span> <span data-ttu-id="39200-125">결과적으로 모든 인터페이스의 요구 사항을 충족하는 일반 백 엔드보다 더 작고, 덜 복잡하고, 더 빠를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-125">As a result, it will be smaller, less complex, and likely faster than a generic backend that tries to satisfy the requirements for all interfaces.</span></span> <span data-ttu-id="39200-126">각 인터페이스 팀은 자율적으로 자체 백 엔드를 제어할 수 있으며, 중앙 집중식 백 엔드 개발 팀에 의존하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-126">Each interface team has autonomy to control their own backend and doesn't rely on a centralized backend development team.</span></span> <span data-ttu-id="39200-127">이를 통해 인터페이스 팀은 언어 선택, 릴리스 일정, 작업 우선 순위, 백 엔드의 기능 통합 문제를 유연하게 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-127">This gives the interface team flexibility in language selection, release cadence, prioritization of workload, and feature integration in their backend.</span></span>

<span data-ttu-id="39200-128">자세한 내용은 [패턴: 프런트 엔드에 대한 백 엔드](http://samnewman.io/patterns/architectural/bff/)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="39200-128">For more information, see [Pattern: Backends For Frontends](http://samnewman.io/patterns/architectural/bff/).</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="39200-129">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="39200-129">Issues and considerations</span></span>

- <span data-ttu-id="39200-130">배포할 백 엔드 수를 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-130">Consider how many backends to deploy.</span></span>
- <span data-ttu-id="39200-131">여러 다른 인터페이스(예: 모바일 클라이언트)가 동일한 요청을 수행할 경우 각 인터페이스에 대해 백 엔드를 구현할 필요가 있는지 아니면 단일 백 엔드로 충분한지를 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-131">If different interfaces (such as mobile clients) will make the same requests, consider whether it is necessary to implement a backend for each interface, or if a single backend will suffice.</span></span>
- <span data-ttu-id="39200-132">이 패턴을 구현할 때는 서비스 간에 코드가 중복될 가능성이 높습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-132">Code duplication across services is highly likely when implementing this pattern.</span></span>
- <span data-ttu-id="39200-133">프런트 엔드에 중점을 둔 백 엔드 서비스는 클라이언트별 논리 및 동작만 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-133">Frontend-focused backend services should only contain client-specific logic and behavior.</span></span> <span data-ttu-id="39200-134">일반적인 비즈니스 논리 및 기타 전역 기능은 응용 프로그램의 다른 위치에서 관리되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-134">General business logic and other global features should be managed elsewhere in your application.</span></span>
- <span data-ttu-id="39200-135">이 패턴이 개발 팀의 업무에 어떻게 반영될 수 있는지 생각해 봅니다.</span><span class="sxs-lookup"><span data-stu-id="39200-135">Think about how this pattern might be reflected in the responsibilities of a development team.</span></span>
- <span data-ttu-id="39200-136">이 패턴을 구현하는 데 걸리는 시간을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-136">Consider how long it will take to implement this pattern.</span></span> <span data-ttu-id="39200-137">기존 일반 백 엔드를 계속 지원하면서, 새 백 엔드를 구축하면 기술적인 문제가 초래될까요?</span><span class="sxs-lookup"><span data-stu-id="39200-137">Will the effort of building the new backends incur technical debt, while you continue to support the existing generic backend?</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="39200-138">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="39200-138">When to use this pattern</span></span>

<span data-ttu-id="39200-139">다음 경우에 이 패턴을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-139">Use this pattern when:</span></span>

- <span data-ttu-id="39200-140">공유 또는 범용 백 엔드 서비스를 유지 관리하는 데 상당량의 개발 오버헤드가 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-140">A shared or general purpose backend service must be maintained with significant development overhead.</span></span>
- <span data-ttu-id="39200-141">특정 클라이언트 인터페이스의 요구 사항에 맞게 백 엔드를 최적화하려고 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-141">You want to optimize the backend for the requirements of specific client interfaces.</span></span>
- <span data-ttu-id="39200-142">범용 백 엔드는 여러 인터페이스를 수용할 수 있게 사용자 지정이 수행됩니다.</span><span class="sxs-lookup"><span data-stu-id="39200-142">Customizations are made to a general-purpose backend to accommodate multiple interfaces.</span></span>
- <span data-ttu-id="39200-143">대체 언어는 다른 사용자 인터페이스의 백 엔드에 좀 더 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="39200-143">An alternative language is better suited for the backend of a different user interface.</span></span>

<span data-ttu-id="39200-144">다음의 경우에는 이 패턴이 적합하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="39200-144">This pattern may not be suitable:</span></span>

- <span data-ttu-id="39200-145">인터페이스가 백 엔드에 대해 동일하거나 유사한 요청 수행하는 경우</span><span class="sxs-lookup"><span data-stu-id="39200-145">When interfaces make the same or similar requests to the backend.</span></span>
- <span data-ttu-id="39200-146">백 엔드와 상호 작용하는 데 하나의 인터페이스만 사용되는 경우</span><span class="sxs-lookup"><span data-stu-id="39200-146">When only one interface is used to interact with the backend.</span></span>

## <a name="related-guidance"></a><span data-ttu-id="39200-147">관련 지침</span><span class="sxs-lookup"><span data-stu-id="39200-147">Related guidance</span></span>

- [<span data-ttu-id="39200-148">게이트웨이 집계 패턴</span><span class="sxs-lookup"><span data-stu-id="39200-148">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="39200-149">게이트웨이 오프로딩 패턴</span><span class="sxs-lookup"><span data-stu-id="39200-149">Gateway Offloading pattern</span></span>](./gateway-offloading.md)
- [<span data-ttu-id="39200-150">게이트웨이 라우팅 패턴</span><span class="sxs-lookup"><span data-stu-id="39200-150">Gateway Routing pattern</span></span>](./gateway-routing.md)


