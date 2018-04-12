---
title: OLTP(온라인 트랜잭션 처리)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 07e7f680c8ee5e8589ff7cd2236ff95f6ee84f4c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/31/2018
---
# <a name="online-transaction-processing-oltp"></a><span data-ttu-id="7ff03-102">OLTP(온라인 트랜잭션 처리)</span><span class="sxs-lookup"><span data-stu-id="7ff03-102">Online transaction processing (OLTP)</span></span>

<span data-ttu-id="7ff03-103">컴퓨터 시스템을 사용하는 [트랜잭션 데이터](../concepts/transactional-data.md) 관리를 OLTP(온라인 트랜잭션 처리)라고 합니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-103">The management of [transactional data](../concepts/transactional-data.md) using computer systems is referred to as Online Transaction Processing (OLTP).</span></span> <span data-ttu-id="7ff03-104">OLTP 시스템은 조직의 일상적인 작업에서 발생하는 비즈니스 상호 작용을 기록하고, 이 데이터를 쿼리하여 유추할 수 있도록 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-104">OLTP systems record business interactions as they occur in the day-to-day operation of the organization, and support querying of this data to make inferences.</span></span>

![Azure의 OLTP](./images/oltp-data-pipeline.png)

## <a name="when-to-use-this-solution"></a><span data-ttu-id="7ff03-106">이 솔루션을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="7ff03-106">When to use this solution</span></span>

<span data-ttu-id="7ff03-107">비즈니스 트랜잭션을 효율적으로 처리 및 저장하고, 클라이언트 응용 프로그램에 일관된 방식에서 사용할 수 있게 하려는 경우 OLTP를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-107">Choose OLTP when you need to efficiently process and store business transactions and immediately make them available to client applications in a consistent way.</span></span> <span data-ttu-id="7ff03-108">또한 분명한 처리 지연이 비즈니스의 일상 작업에 부정적인 영향을 미칠 때 이 아키텍처를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-108">Use this architecture when any tangible delay in processing would have a negative impact on the day-to-day operations of the business.</span></span>

<span data-ttu-id="7ff03-109">OLTP 시스템은 트랜잭션을 효율적으로 처리 및 저장할 뿐만 아니라 트랜잭션 데이터를 쿼리하도록 디자인되었습니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-109">OLTP systems are designed to efficiently process and store transactions, as well as query transactional data.</span></span> <span data-ttu-id="7ff03-110">OLTP 시스템에서 개별 트랜잭션을 효율적으로 처리하고 저장한다는 목적은 데이터 정규화, 즉 데이터를 덜 중복되는 좀 더 작은 청크로 분할함으로써 어느 정도 달성됩니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-110">The goal of efficiently processing and storing individual transactions by an OLTP system is partly accomplished by data normalization &mdash; that is, breaking the data up into smaller chunks that are less redundant.</span></span> <span data-ttu-id="7ff03-111">이 경우 OLTP 시스템이 많은 수의 트랜잭션을 독립적으로 처리할 수 있도록 하고, 중복 데이터가 있을 때 데이터 무결성을 유지하기 위해 필요한 추가 처리가 해소되므로 효율성이 유지됩니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-111">This supports efficiency because it enables the OLTP system to process large numbers of transactions independently, and avoids extra processing needed to maintain data integrity in the presence of redundant data.</span></span>

## <a name="challenges"></a><span data-ttu-id="7ff03-112">과제</span><span class="sxs-lookup"><span data-stu-id="7ff03-112">Challenges</span></span>
<span data-ttu-id="7ff03-113">OLTP 시스템을 구현 및 사용할 경우 다음과 같은 몇 가지 해결 과제가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-113">Implementing and using an OLTP system can create a few challenges:</span></span>

- <span data-ttu-id="7ff03-114">잘 계획된 SQL Server 기반 솔루션과 같은 예외도 있지만, OLTP 시스템이 대량의 데이터에 대한 집계를 처리하는 데 항상 적절한 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-114">OLTP systems are not always good for handling aggregates over large amounts of data, although there are exceptions, such as a well-planned SQL Server-based solution.</span></span> <span data-ttu-id="7ff03-115">수백만 개의 개별 트랜잭션에 대한 집계 계산에 의존하는 데이터 분석은 OLTP 시스템의 리소스를 과도하게 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-115">Analytics against the data, that rely on aggregate calculations over millions of individual transactions, are very resource intensive for an OLTP system.</span></span> <span data-ttu-id="7ff03-116">따라서 실행이 느려질 수 있으며, 데이터베이스의 다른 트랜잭션을 차단하게 되어 속도 저하를 야기할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-116">They can be slow to execute and can cause a slow-down by blocking other transactions in the database.</span></span>
- <span data-ttu-id="7ff03-117">고도로 정규화된 데이터에 대해 분석 및 보고 작업을 수행할 경우, 조인을 사용해서 대부분의 쿼리를 비정규화해야 하므로 쿼리가 복잡해질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-117">When conducting analytics and reporting on data that is highly normalized, the queries tend to be complex, because most queries need to de-normalize the data by using joins.</span></span> <span data-ttu-id="7ff03-118">또한 OLTP 시스템에서 데이터베이스 개체에 대한 명명 규칙은 간결하고 간단한 편입니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-118">Also, naming conventions for database objects in OLTP systems tend to be terse and succinct.</span></span> <span data-ttu-id="7ff03-119">명명 규칙이 간결하고 정규화가 높아지면서 비즈니스 사용자가 DBA 또는 데이터 개발자의 도움 없이 OLTP 시스템을 쿼리하는 것이 어려워집니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-119">The increased normalization coupled with terse naming conventions makes OLTP systems difficult for business users to query, without the help of a DBA or data developer.</span></span>
- <span data-ttu-id="7ff03-120">트랜잭션 기록을 무기한 저장하고, 하나의 테이블에 너무 많은 데이터를 저장하게 되면 저장된 트랜잭션 수에 따라 쿼리 성능이 저하될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-120">Storing the history of transactions indefinitely and storing too much data in any one table can lead to slow query performance, depending on the number of transactions stored.</span></span> <span data-ttu-id="7ff03-121">일반적인 해결 방법은 OLTP 시스템에서 적절한 기간(예: 현재 회계 연도) 동안 두었다가 기록 데이터를 데이터 마트 또는 [데이터 웨어하우스](../technology-choices/data-warehouses.md) 등의 다른 시스템으로 오프로드하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-121">The common solution is to maintain a relevant window of time (such as the current fiscal year) in the OLTP system and offload historical data to other systems, such as a data mart or [data warehouse](../technology-choices/data-warehouses.md).</span></span>

## <a name="oltp-in-azure"></a><span data-ttu-id="7ff03-122">Azure의 OLTP</span><span class="sxs-lookup"><span data-stu-id="7ff03-122">OLTP in Azure</span></span>

<span data-ttu-id="7ff03-123">[App Service Web Apps](/azure/app-service/app-service-web-overview)에 호스트되는 웹 사이트, App Service에서 실행되는 REST API 등의 응용 프로그램이나 모바일 또는 데스크톱 응용 프로그램은 일반적으로 REST API를 매개자로 사용해서 OLTP 시스템과 통신합니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-123">Applications such as websites hosted in [App Service Web Apps](/azure/app-service/app-service-web-overview), REST APIs running in App Service, or mobile or desktop applications communicate with the OLTP system, typically via a REST API intermediary.</span></span>

<span data-ttu-id="7ff03-124">실제로 대부분의 워크로드는 순수한 OLTP가 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-124">In practice, most workloads are not purely OLTP.</span></span> <span data-ttu-id="7ff03-125">[분석 구성 요소](../scenarios/online-analytical-processing.md)의 역할을 하기도 합니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-125">There tends to be an [analytical component](../scenarios/online-analytical-processing.md) as well.</span></span> <span data-ttu-id="7ff03-126">또한, 운영 체제에 대한 보고서 실행 등, 실시간 보고에 대한 요구도 높아지고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-126">In addition, there is an increasing demand for real-time reporting, such as running reports against the operational system.</span></span> <span data-ttu-id="7ff03-127">이것을 HTAP(하이브리드 트랜잭션 및 분석 처리)라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="7ff03-127">This is also referred to as HTAP (Hybrid Transactional and Analytical Processing).</span></span> <span data-ttu-id="7ff03-128">자세한 내용은 [OLAP(온라인 분석 처리) 데이터 저장소](../technology-choices/olap-data-stores.md)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="7ff03-128">For more information, see [Online Analytical Processing (OLAP) data stores](../technology-choices/olap-data-stores.md).</span></span>

## <a name="technology-choices"></a><span data-ttu-id="7ff03-129">기술 선택</span><span class="sxs-lookup"><span data-stu-id="7ff03-129">Technology choices</span></span>

<span data-ttu-id="7ff03-130">데이터 저장소:</span><span class="sxs-lookup"><span data-stu-id="7ff03-130">Data storage:</span></span>

- [<span data-ttu-id="7ff03-131">Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="7ff03-131">Azure SQL Database</span></span>](/azure/sql-database/)
- [<span data-ttu-id="7ff03-132">Azure VM의 SQL Server</span><span class="sxs-lookup"><span data-stu-id="7ff03-132">SQL Server in an Azure VM</span></span>](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [<span data-ttu-id="7ff03-133">Azure Database for MySQL</span><span class="sxs-lookup"><span data-stu-id="7ff03-133">Azure Database for MySQL</span></span>](/azure/mysql/)
- [<span data-ttu-id="7ff03-134">Azure Database for PostgreSQL</span><span class="sxs-lookup"><span data-stu-id="7ff03-134">Azure Database for PostgreSQL</span></span>](/azure/postgresql/)

<span data-ttu-id="7ff03-135">자세한 내용은 [OLTP 데이터 저장소 선택](../technology-choices/oltp-data-stores.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="7ff03-135">For more information, see [Choosing an OLTP data store](../technology-choices/oltp-data-stores.md)</span></span>

<span data-ttu-id="7ff03-136">데이터 원본:</span><span class="sxs-lookup"><span data-stu-id="7ff03-136">Data sources:</span></span>

- [<span data-ttu-id="7ff03-137">App service</span><span class="sxs-lookup"><span data-stu-id="7ff03-137">App service</span></span>](/azure/app-service/)
- [<span data-ttu-id="7ff03-138">Mobile Apps</span><span class="sxs-lookup"><span data-stu-id="7ff03-138">Mobile Apps</span></span>](/azure/app-service-mobile/)

