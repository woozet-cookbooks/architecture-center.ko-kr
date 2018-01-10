---
title: "가용성 패턴"
description: "가용성은 시스템이 기능하고 작동하는 시간의 비율을 정의합니다. 시스템 오류, 인프라 문제, 악의적인 공격, 시스템 부하 등의 영향을 받습니다. 일반적으로 가동 시간의 백분율로 측정됩니다. 클라우드 응용 프로그램은 일반적으로 사용자에게 SLA(서비스 수준 계약)를 제공합니다. 이것은 응용 프로그램이 가용성을 최대화하는 방식으로 디자인 및 구현되어야 함을 의미합니다."
keywords: "디자인 패턴"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: f7eb6b0df388b2f1dab83e64ab540cc22f368e19
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="availability-patterns"></a><span data-ttu-id="eb708-107">가용성 패턴</span><span class="sxs-lookup"><span data-stu-id="eb708-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="eb708-108">가용성은 시스템이 기능하고 작동하는 시간의 비율을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="eb708-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="eb708-109">시스템 오류, 인프라 문제, 악의적인 공격, 시스템 부하 등의 영향을 받습니다.</span><span class="sxs-lookup"><span data-stu-id="eb708-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="eb708-110">일반적으로 가동 시간의 백분율로 측정됩니다.</span><span class="sxs-lookup"><span data-stu-id="eb708-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="eb708-111">클라우드 응용 프로그램은 일반적으로 사용자에게 SLA(서비스 수준 계약)를 제공합니다. 이것은 응용 프로그램이 가용성을 최대화하는 방식으로 디자인 및 구현되어야 함을 의미합니다.</span><span class="sxs-lookup"><span data-stu-id="eb708-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>

| <span data-ttu-id="eb708-112">패턴</span><span class="sxs-lookup"><span data-stu-id="eb708-112">Pattern</span></span> | <span data-ttu-id="eb708-113">요약</span><span class="sxs-lookup"><span data-stu-id="eb708-113">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="eb708-114">상태 엔드포인트 모니터링</span><span class="sxs-lookup"><span data-stu-id="eb708-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="eb708-115">외부 도구가 노출된 엔드포인트를 통해 주기적으로 액세스할 수 있는 기능 검사를 응용 프로그램 내부에 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="eb708-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
| [<span data-ttu-id="eb708-116">큐 기반 부하 평준화</span><span class="sxs-lookup"><span data-stu-id="eb708-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="eb708-117">작업 그리고 그 작업이 일시적인 높은 부하를 부드럽게 처리하기 위해 호출하는 서비스 사이에서 버퍼 역할을 하는 큐를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="eb708-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="eb708-118">제한</span><span class="sxs-lookup"><span data-stu-id="eb708-118">Throttling</span></span>](../throttling.md) | <span data-ttu-id="eb708-119">응용 프로그램 인스턴스, 개별 테넌트 또는 서비스 전체의 리소스 사용량을 제어합니다.</span><span class="sxs-lookup"><span data-stu-id="eb708-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span> |