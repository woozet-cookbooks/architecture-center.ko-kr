---
title: "데이터 관리 패턴"
description: "데이터 관리는 클라우드 응용 프로그램의 핵심 요소이며 대부분의 품질 특성에 영향을 줍니다. 일반적으로 데이터는 성능, 확장성, 가용성 등의 이유로 여러 위치의 여러 서버에 호스팅되며, 이로 인해 다양한 문제가 발생할 수 있습니다. 예를 들어 데이터 일관성을 유지해야 하며, 일반적으로 여러 위치 간에 데이터를 동기화해야 합니다."
keywords: "디자인 패턴"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: a009a06268f114ab7be4544dd81710612dabd8f4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="data-management-patterns"></a><span data-ttu-id="23629-106">데이터 관리 패턴</span><span class="sxs-lookup"><span data-stu-id="23629-106">Data Management patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="23629-107">데이터 관리는 클라우드 응용 프로그램의 핵심 요소이며 대부분의 품질 특성에 영향을 줍니다.</span><span class="sxs-lookup"><span data-stu-id="23629-107">Data management is the key element of cloud applications, and influences most of the quality attributes.</span></span> <span data-ttu-id="23629-108">일반적으로 데이터는 성능, 확장성, 가용성 등의 이유로 여러 위치의 여러 서버에 호스팅되며, 이로 인해 다양한 문제가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="23629-108">Data is typically hosted in different locations and across multiple servers for reasons such as performance, scalability or availability, and this can present a range of challenges.</span></span> <span data-ttu-id="23629-109">예를 들어 데이터 일관성을 유지해야 하며, 일반적으로 여러 위치 간에 데이터를 동기화해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="23629-109">For example, data consistency must be maintained, and data will typically need to be synchronized across different locations.</span></span>

| <span data-ttu-id="23629-110">패턴</span><span class="sxs-lookup"><span data-stu-id="23629-110">Pattern</span></span> | <span data-ttu-id="23629-111">요약</span><span class="sxs-lookup"><span data-stu-id="23629-111">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="23629-112">Cache-Aside</span><span class="sxs-lookup"><span data-stu-id="23629-112">Cache-Aside</span></span>](../cache-aside.md) | <span data-ttu-id="23629-113">필요할 때 데이터를 데이터 저장소에서 캐시로 로드</span><span class="sxs-lookup"><span data-stu-id="23629-113">Load data on demand into a cache from a data store</span></span> |
| [<span data-ttu-id="23629-114">CQRS</span><span class="sxs-lookup"><span data-stu-id="23629-114">CQRS</span></span>](../cqrs.md) | <span data-ttu-id="23629-115">별도의 인터페이스를 사용하여 데이터를 업데이트하는 작업과 데이터를 읽는 작업을 분리합니다.</span><span class="sxs-lookup"><span data-stu-id="23629-115">Segregate operations that read data from operations that update data by using separate interfaces.</span></span> |
| [<span data-ttu-id="23629-116">이벤트 소싱</span><span class="sxs-lookup"><span data-stu-id="23629-116">Event Sourcing</span></span>](../event-sourcing.md) | <span data-ttu-id="23629-117">추가 전용 저장소를 사용하여 도메인의 데이터에 대해 수행된 작업을 설명하는 일련의 이벤트 전체를 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="23629-117">Use an append-only store to record the full series of events that describe actions taken on data in a domain.</span></span> |
| [<span data-ttu-id="23629-118">인덱스 테이블</span><span class="sxs-lookup"><span data-stu-id="23629-118">Index Table</span></span>](../index-table.md) | <span data-ttu-id="23629-119">쿼리에서 자주 참조하는 데이터 저장소의 필드에 대한 인덱스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="23629-119">Create indexes over the fields in data stores that are frequently referenced by queries.</span></span> |
| [<span data-ttu-id="23629-120">구체화된 뷰</span><span class="sxs-lookup"><span data-stu-id="23629-120">Materialized View</span></span>](../materialized-view.md) | <span data-ttu-id="23629-121">데이터가 필요한 쿼리 작업에 대해 이상적으로 포맷되지 않은 경우 하나 이상의 데이터 저장소에 있는 데이터에 대한 미리 채워진 뷰를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="23629-121">Generate prepopulated views over the data in one or more data stores when the data isn't ideally formatted for required query operations.</span></span> |
| [<span data-ttu-id="23629-122">분할</span><span class="sxs-lookup"><span data-stu-id="23629-122">Sharding</span></span>](../sharding.md) | <span data-ttu-id="23629-123">데이터 저장소를 수평 파티션 또는 분할 집합으로 나눕니다.</span><span class="sxs-lookup"><span data-stu-id="23629-123">Divide a data store into a set of horizontal partitions or shards.</span></span> |
| [<span data-ttu-id="23629-124">정적 콘텐츠 호스팅</span><span class="sxs-lookup"><span data-stu-id="23629-124">Static Content Hosting</span></span>](../static-content-hosting.md) | <span data-ttu-id="23629-125">정적 콘텐츠를 클라이언트에 직접 제공할 수 있는 클라우드 기반 저장소 서비스에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="23629-125">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> |
| [<span data-ttu-id="23629-126">발레 키</span><span class="sxs-lookup"><span data-stu-id="23629-126">Valet Key</span></span>](../valet-key.md) | <span data-ttu-id="23629-127">클라이언트에 특정 리소스 또는 서비스에 대한 제한된 직접 액세스를 제공하는 토큰 또는 키를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="23629-127">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span> |