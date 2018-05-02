---
title: Azure 계산 서비스에 대한 의사 결정 트리
description: 계산 서비스를 선택하기 위한 순서도
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: f5703f4906ca2ea6f825b383710eb4bd335f5043
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/23/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="4da39-103">Azure 계산 서비스에 대한 의사 결정 트리</span><span class="sxs-lookup"><span data-stu-id="4da39-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="4da39-104">Azure는 응용 프로그램 코드를 호스트하는 다양한 방법을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="4da39-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="4da39-105">*계산*이라는 용어는 응용 프로그램이 실행되는 계산 리소스의 호스팅 모델을 말합니다.</span><span class="sxs-lookup"><span data-stu-id="4da39-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="4da39-106">다음 순서도는 응용 프로그램에 대한 계산 서비스를 선택하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="4da39-106">The following flowchart will help you to choose a compute service for your application.</span></span>
 
![](../images/compute-decision-tree.svg)

<span data-ttu-id="4da39-107">순서도는 권장 사항에 연결할 주요 의사 결정 기준의 집합을 안내합니다.</span><span class="sxs-lookup"><span data-stu-id="4da39-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> <span data-ttu-id="4da39-108">모든 응용 프로그램에는 고유한 요구 사항이 있으므로 권장 사항을 시작점으로 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="4da39-108">Every application has unique requirements, so you should treat the recommendation as a starting point.</span></span> <span data-ttu-id="4da39-109">그런 다음, 다음과 같은 측면을 살펴보는 보다 자세한 분석을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="4da39-109">Then perform a more detailed analysis, looking at aspects such as:</span></span>
 
- <span data-ttu-id="4da39-110">기능 집합</span><span class="sxs-lookup"><span data-stu-id="4da39-110">Feature set</span></span>
- [<span data-ttu-id="4da39-111">서비스 한도</span><span class="sxs-lookup"><span data-stu-id="4da39-111">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="4da39-112">비용</span><span class="sxs-lookup"><span data-stu-id="4da39-112">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="4da39-113">SLA</span><span class="sxs-lookup"><span data-stu-id="4da39-113">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="4da39-114">국가별 가용성</span><span class="sxs-lookup"><span data-stu-id="4da39-114">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="4da39-115">개발자 에코시스템 및 팀 기술</span><span class="sxs-lookup"><span data-stu-id="4da39-115">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="4da39-116">계산 비교 표</span><span class="sxs-lookup"><span data-stu-id="4da39-116">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="4da39-117">응용 프로그램이 여러 워크로드로 구성된 경우 각 워크로드를 개별적으로 평가합니다.</span><span class="sxs-lookup"><span data-stu-id="4da39-117">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="4da39-118">완벽한 솔루션은 두 개 이상의 계산 서비스를 통합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4da39-118">A complete solution may incorporate two or more compute services.</span></span>

