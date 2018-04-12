---
title: 메시지 패턴
description: 클라우드 응용 프로그램은 기본적으로 분산되기 때문에 구성 요소와 서비스를 연결하는 메시지 인프라가 필요하며, 가용성을 최대화할 수 있도록 느슨하게 결합된 인프라가 가장 이상적입니다. 널리 사용되는 비동기 메시지는 여러 가지 이점이 있지만 메시지 순서 지정, 포이즌 메시지 관리, 멱등성 같은 문제도 있습니다.
keywords: 디자인 패턴
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 8bf37903df3a6eb23f1581e0405358a7aee61f79
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="messaging-patterns"></a><span data-ttu-id="73454-105">메시지 패턴</span><span class="sxs-lookup"><span data-stu-id="73454-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="73454-106">클라우드 응용 프로그램은 기본적으로 분산되기 때문에 구성 요소와 서비스를 연결하는 메시지 인프라가 필요하며, 가용성을 최대화할 수 있도록 느슨하게 결합된 인프라가 가장 이상적입니다.</span><span class="sxs-lookup"><span data-stu-id="73454-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="73454-107">널리 사용되는 비동기 메시지는 여러 가지 이점이 있지만 메시지 순서 지정, 포이즌 메시지 관리, 멱등성 같은 문제도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73454-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>


|                            <span data-ttu-id="73454-108">패턴</span><span class="sxs-lookup"><span data-stu-id="73454-108">Pattern</span></span>                             |                                                                        <span data-ttu-id="73454-109">요약</span><span class="sxs-lookup"><span data-stu-id="73454-109">Summary</span></span>                                                                         |
|----------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
|        [<span data-ttu-id="73454-110">경쟁 소비자</span><span class="sxs-lookup"><span data-stu-id="73454-110">Competing Consumers</span></span>](../competing-consumers.md)        |                            <span data-ttu-id="73454-111">여러 동시 소비자가 동일한 메시징 채널에 수신된 메시지를 처리할 수 있게 해 줍니다.</span><span class="sxs-lookup"><span data-stu-id="73454-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span>                            |
|          [<span data-ttu-id="73454-112">파이프 및 필터</span><span class="sxs-lookup"><span data-stu-id="73454-112">Pipes and Filters</span></span>](../pipes-and-filters.md)          |                       <span data-ttu-id="73454-113">복잡한 처리를 수행하는 작업을 재사용 가능한 일련의 별도 요소로 분류합니다.</span><span class="sxs-lookup"><span data-stu-id="73454-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span>                        |
|             [<span data-ttu-id="73454-114">우선 순위 큐</span><span class="sxs-lookup"><span data-stu-id="73454-114">Priority Queue</span></span>](../priority-queue.md)             | <span data-ttu-id="73454-115">우선 순위가 높은 요청을 우선 순위가 낮은 요청보다 먼저 받아서 처리하도록 서비스로 전송된 요청의 우선 순위를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="73454-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
|  [<span data-ttu-id="73454-116">큐 기반 부하 평준화</span><span class="sxs-lookup"><span data-stu-id="73454-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md)  |              <span data-ttu-id="73454-117">작업 그리고 그 작업이 일시적인 높은 부하를 부드럽게 처리하기 위해 호출하는 서비스 사이에서 버퍼 역할을 하는 큐를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="73454-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span>               |
| [<span data-ttu-id="73454-118">Scheduler 에이전트 감독자</span><span class="sxs-lookup"><span data-stu-id="73454-118">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) |                              <span data-ttu-id="73454-119">서비스 및 기타 원격 리소스의 분산된 집합에서 일련의 작업을 조정합니다.</span><span class="sxs-lookup"><span data-stu-id="73454-119">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span>                              |

