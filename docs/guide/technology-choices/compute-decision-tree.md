---
title: Azure 계산 서비스에 대한 의사 결정 트리
description: 계산 서비스를 선택하기 위한 순서도
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: 60bb84d4bf210888d3d43498db043b6e452f6a80
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206724"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="44e05-103">Azure 계산 서비스에 대한 의사 결정 트리</span><span class="sxs-lookup"><span data-stu-id="44e05-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="44e05-104">Azure는 응용 프로그램 코드를 호스트하는 다양한 방법을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="44e05-105">*계산*이라는 용어는 응용 프로그램이 실행되는 계산 리소스의 호스팅 모델을 말합니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="44e05-106">다음 순서도는 응용 프로그램에 대한 계산 서비스를 선택하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="44e05-107">순서도는 권장 사항에 연결할 주요 의사 결정 기준의 집합을 안내합니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="44e05-108">**이 순서도를 시작점으로 처리합니다.**</span><span class="sxs-lookup"><span data-stu-id="44e05-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="44e05-109">모든 응용 프로그램에는 고유한 요구 사항이 있으므로 권장 사항을 시작점으로 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="44e05-110">그런 다음, 다음과 같은 측면을 살펴보는 보다 자세한 평가를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="44e05-111">기능 집합</span><span class="sxs-lookup"><span data-stu-id="44e05-111">Feature set</span></span>
- [<span data-ttu-id="44e05-112">서비스 한도</span><span class="sxs-lookup"><span data-stu-id="44e05-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="44e05-113">비용</span><span class="sxs-lookup"><span data-stu-id="44e05-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="44e05-114">SLA</span><span class="sxs-lookup"><span data-stu-id="44e05-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="44e05-115">국가별 가용성</span><span class="sxs-lookup"><span data-stu-id="44e05-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="44e05-116">개발자 에코시스템 및 팀 기술</span><span class="sxs-lookup"><span data-stu-id="44e05-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="44e05-117">계산 비교 표</span><span class="sxs-lookup"><span data-stu-id="44e05-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="44e05-118">응용 프로그램이 여러 워크로드로 구성된 경우 각 워크로드를 개별적으로 평가합니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="44e05-119">완벽한 솔루션은 두 개 이상의 계산 서비스를 통합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-119">A complete solution may incorporate two or more compute services.</span></span>

## <a name="flowchart"></a><span data-ttu-id="44e05-120">순서도</span><span class="sxs-lookup"><span data-stu-id="44e05-120">Flowchart</span></span>

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="44e05-121">정의</span><span class="sxs-lookup"><span data-stu-id="44e05-121">Definitions</span></span>

- <span data-ttu-id="44e05-122">**그린필드**는 완전히 새롭고 처음부터 빌드된 소프트웨어 프로젝트를 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-122">**Greenfield** describes a software project that is completely new and built from scratch.</span></span> <span data-ttu-id="44e05-123">레거시 코드는 포함되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-123">It does not include legacy code.</span></span> 

- <span data-ttu-id="44e05-124">**브라운필드**는 기존 응용 프로그램을 기반으로 하는 소프트웨어 프로젝트를 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-124">**Brownfield** describes a software project that builds on an existing application.</span></span> <span data-ttu-id="44e05-125">레거시 코드 또는 프레임워크를 상속할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-125">It may inherit legacy code or frameworks.</span></span>

- <span data-ttu-id="44e05-126">**리프트 앤 시프트**는 응용 프로그램을 다시 디자인하거나 코드를 변경하지 않고 워크로드를 클라우드로 마이그레이션하는 전략입니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-126">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="44e05-127">*재 호스팅*이라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-127">Also called *rehosting*.</span></span> <span data-ttu-id="44e05-128">자세한 내용은 [Azure 마이그레이션 센터](https://azure.microsoft.com/migration/)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="44e05-128">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="44e05-129">**클라우드 최적화**는 클라우드 고유 기능 및 성능을 활용하도록 응용 프로그램을 리팩터링하여 클라우드로 마이그레이션하는 전략입니다.</span><span class="sxs-lookup"><span data-stu-id="44e05-129">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="44e05-130">다음 단계</span><span class="sxs-lookup"><span data-stu-id="44e05-130">Next steps</span></span>

<span data-ttu-id="44e05-131">고려해야 할 추가 기준은 [Azure 계산 서비스를 선택하기 위한 조건](./compute-comparison.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="44e05-131">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
