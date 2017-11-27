---
title: "사용량이 많은 데이터베이스 안티패턴"
description: "처리를 데이터베이스 서버로 오프로드하면 성능 및 확장성 문제가 발생할 수 있습니다."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 9fdbde0731a1be570ef611894a9d23a1be87f4e7
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="busy-database-antipattern"></a><span data-ttu-id="814df-103">사용량이 많은 데이터베이스 안티패턴</span><span class="sxs-lookup"><span data-stu-id="814df-103">Busy Database antipattern</span></span>

<span data-ttu-id="814df-104">처리를 데이터베이스 서버로 오프로드하면 데이터 저장 및 검색 요청에 응답하는 대신 코드를 실행하는 데 상당한 시간이 소요될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-104">Offloading processing to a database server can cause it to spend a significant proportion of time running code, rather than responding to requests to store and retrieve data.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="814df-105">문제 설명</span><span class="sxs-lookup"><span data-stu-id="814df-105">Problem description</span></span>

<span data-ttu-id="814df-106">많은 데이터베이스 시스템에서 코드를 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-106">Many database systems can run code.</span></span> <span data-ttu-id="814df-107">저장 프로시저 및 트리거를 예로 들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-107">Examples include stored procedures and triggers.</span></span> <span data-ttu-id="814df-108">이런 처리를 데이터 가까이에서 수행하는 것이 처리를 위해 데이터를 클라이언트 응용 프로그램에 전송하는 것보다 효율적인 경우가 많습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-108">Often, it's more efficient to perform this processing close to the data, rather than transmitting the data to a client application for processing.</span></span> <span data-ttu-id="814df-109">하지만 이런 기능을 과도하게 사용하면 성능이 저하될 수 있으며 이유는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-109">However, overusing these features can hurt performance, for several reasons:</span></span>

- <span data-ttu-id="814df-110">데이터베이스 서버가 새 클라이언트 요청을 수락하고 데이터를 가져오기보다 처리하는 데 너무 많은 시간을 소비할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-110">The database server may spend too much time processing, rather than accepting new client requests and fetching data.</span></span>
- <span data-ttu-id="814df-111">데이터베이스는 대개 공유 리소스여서 많이 사용되는 기간에는 병목 상태가 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-111">A database is usually a shared resource, so it can become a bottleneck during periods of high use.</span></span>
- <span data-ttu-id="814df-112">데이터 저장소가 유료인 경우 런타임 비용이 과도해질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-112">Runtime costs may be excessive if the data store is metered.</span></span> <span data-ttu-id="814df-113">관리되는 데이터베이스 서비스의 경우 특히 그렇습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-113">That's particularly true of managed database services.</span></span> <span data-ttu-id="814df-114">예를 들어 Azure SQL Database는 DTU([데이터베이스 트랜잭션 단위][dtu])에 요금을 부과합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-114">For example, Azure SQL Database charges for [Database Transaction Units][dtu] (DTUs).</span></span>
- <span data-ttu-id="814df-115">데이터베이스는 확장 가능한 용량이 한정되어 있으며 데이터베이스를 수평 확장하기는 쉽지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-115">Databases have finite capacity to scale up, and it's not trivial to scale a database horizontally.</span></span> <span data-ttu-id="814df-116">따라서 쉽게 확장할 수 있는 VM이나 App Service 앱과 같은 컴퓨팅 리소스로 처리를 옮기는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-116">Therefore, it may be better to move processing into a compute resource, such as a VM or App Service app, that can easily scale out.</span></span>

<span data-ttu-id="814df-117">이런 안티패턴이 발생하는 일반적인 이유는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-117">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="814df-118">데이터베이스는 리포지토리가 아닌 서비스로 간주됩니다.</span><span class="sxs-lookup"><span data-stu-id="814df-118">The database is viewed as a service rather than a repository.</span></span> <span data-ttu-id="814df-119">응용 프로그램은 데이터베이스 서버를 사용하여 데이터 형식을 지정하거나(예: XML로 변환) 문자열 데이터를 조작하거나 복잡한 계산을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-119">An application might use the database server to format data (for example, converting to XML), manipulate string data, or perform complex calculations.</span></span>
- <span data-ttu-id="814df-120">개발자는 결과를 사용자에게 직접 표시할 수 있는 쿼리를 작성하려고 합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-120">Developers try to write queries whose results can be displayed directly to users.</span></span> <span data-ttu-id="814df-121">예를 들어, 쿼리는 필드를 결합하거나 로캘에 따라 날짜, 시간 및 통화 형식을 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-121">For example a query might combine fields, or format dates, times, and currency according to locale.</span></span>
- <span data-ttu-id="814df-122">개발자는 계산을 데이터베이스로 푸시하여 [불필요한 가져오기][ExtraneousFetching] 안티패턴을 수정하려고 합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-122">Developers are trying to correct the [Extraneous Fetching][ExtraneousFetching] antipattern by pushing computations to the database.</span></span>
- <span data-ttu-id="814df-123">저장 프로시저는 비즈니스 논리를 캡슐화하는 데 사용됩니다. 아마도 유지 관리 및 업데이트가 더 쉽다고 간주되기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="814df-123">Stored procedures are used to encapsulate business logic, perhaps because they are considered easier to maintain and update.</span></span>

<span data-ttu-id="814df-124">다음 예제는 지정된 판매 지역에서 가장 중요한 주문을 20건 검색하고 그 결과를 XML로 형식화합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-124">The following example retrieves the 20 most valuable orders for a specified sales territory and formats the results as XML.</span></span>
<span data-ttu-id="814df-125">Transact-SQL 함수를 사용하여 데이터를 구문 분석하고 결과를 XML로 변환합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-125">It uses Transact-SQL functions to parse the data and convert the results to XML.</span></span> <span data-ttu-id="814df-126">전체 샘플은 [여기][sample-app]에서 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-126">You can find the complete sample [here][sample-app].</span></span>

```SQL
SELECT TOP 20
  soh.[SalesOrderNumber]  AS '@OrderNumber',
  soh.[Status]            AS '@Status',
  soh.[ShipDate]          AS '@ShipDate',
  YEAR(soh.[OrderDate])   AS '@OrderDateYear',
  MONTH(soh.[OrderDate])  AS '@OrderDateMonth',
  soh.[DueDate]           AS '@DueDate',
  FORMAT(ROUND(soh.[SubTotal],2),'C')
                          AS '@SubTotal',
  FORMAT(ROUND(soh.[TaxAmt],2),'C')
                          AS '@TaxAmt',
  FORMAT(ROUND(soh.[TotalDue],2),'C')
                          AS '@TotalDue',
  CASE WHEN soh.[TotalDue] > 5000 THEN 'Y' ELSE 'N' END
                          AS '@ReviewRequired',
  (
  SELECT
    c.[AccountNumber]     AS '@AccountNumber',
    UPPER(LTRIM(RTRIM(REPLACE(
    CONCAT( p.[Title], ' ', p.[FirstName], ' ', p.[MiddleName], ' ', p.[LastName], ' ', p.[Suffix]),
    '  ', ' '))))         AS '@FullName'
  FROM [Sales].[Customer] c
    INNER JOIN [Person].[Person] p
  ON c.[PersonID] = p.[BusinessEntityID]
  WHERE c.[CustomerID] = soh.[CustomerID]
  FOR XML PATH ('Customer'), TYPE
  ),

  (
  SELECT
    sod.[OrderQty]      AS '@Quantity',
    FORMAT(sod.[UnitPrice],'C')
                        AS '@UnitPrice',
    FORMAT(ROUND(sod.[LineTotal],2),'C')
                        AS '@LineTotal',
    sod.[ProductID]     AS '@ProductId',
    CASE WHEN (sod.[ProductID] >= 710) AND (sod.[ProductID] <= 720) AND (sod.[OrderQty] >= 5) THEN 'Y' ELSE 'N' END
                        AS '@InventoryCheckRequired'

  FROM [Sales].[SalesOrderDetail] sod
  WHERE sod.[SalesOrderID] = soh.[SalesOrderID]
  ORDER BY sod.[SalesOrderDetailID]
  FOR XML PATH ('LineItem'), TYPE, ROOT('OrderLineItems')
  )

FROM [Sales].[SalesOrderHeader] soh
WHERE soh.[TerritoryId] = @TerritoryId
ORDER BY soh.[TotalDue] DESC
FOR XML PATH ('Order'), ROOT('Orders')
```

<span data-ttu-id="814df-127">이것은 분명히 복잡한 쿼리입니다.</span><span class="sxs-lookup"><span data-stu-id="814df-127">Clearly, this is complex query.</span></span> <span data-ttu-id="814df-128">나중에 살펴보겠지만 데이터베이스 서버에서 중요한 처리 리소스를 사용하는 것으로 나타납니다.</span><span class="sxs-lookup"><span data-stu-id="814df-128">As we'll see later, it turns out to use significant processing resources on the database server.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="814df-129">문제를 해결하는 방법</span><span class="sxs-lookup"><span data-stu-id="814df-129">How to fix the problem</span></span>

<span data-ttu-id="814df-130">처리를 데이터베이스 서버에서 다른 응용 프로그램 계층으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-130">Move processing from the database server into other application tiers.</span></span> <span data-ttu-id="814df-131">이상적으로는 데이터베이스가 RDBMS의 집계와 같이 최적화된 기능만을 사용하여 데이터 액세스 작업을 수행하도록 데이터베이스를 제한해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-131">Ideally, you should limit the database to performing data access operations, using only the capabilities that the database is optimized for, such as aggregation in an RDBMS.</span></span>

<span data-ttu-id="814df-132">예를 들어, 이전 Transact-SQL 코드는 처리할 데이터만 검색하는 명령문으로 대체될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-132">For example, the previous Transact-SQL code can be replaced with a statement that simply retrieves the data to be processed.</span></span>

```SQL
SELECT
soh.[SalesOrderNumber]  AS [OrderNumber],
soh.[Status]            AS [Status],
soh.[OrderDate]         AS [OrderDate],
soh.[DueDate]           AS [DueDate],
soh.[ShipDate]          AS [ShipDate],
soh.[SubTotal]          AS [SubTotal],
soh.[TaxAmt]            AS [TaxAmt],
soh.[TotalDue]          AS [TotalDue],
c.[AccountNumber]       AS [AccountNumber],
p.[Title]               AS [CustomerTitle],
p.[FirstName]           AS [CustomerFirstName],
p.[MiddleName]          AS [CustomerMiddleName],
p.[LastName]            AS [CustomerLastName],
p.[Suffix]              AS [CustomerSuffix],
sod.[OrderQty]          AS [Quantity],
sod.[UnitPrice]         AS [UnitPrice],
sod.[LineTotal]         AS [LineTotal],
sod.[ProductID]         AS [ProductId]
FROM [Sales].[SalesOrderHeader] soh
INNER JOIN [Sales].[Customer] c ON soh.[CustomerID] = c.[CustomerID]
INNER JOIN [Person].[Person] p ON c.[PersonID] = p.[BusinessEntityID]
INNER JOIN [Sales].[SalesOrderDetail] sod ON soh.[SalesOrderID] = sod.[SalesOrderID]
WHERE soh.[TerritoryId] = @TerritoryId
AND soh.[SalesOrderId] IN (
    SELECT TOP 20 SalesOrderId
    FROM [Sales].[SalesOrderHeader] soh
    WHERE soh.[TerritoryId] = @TerritoryId
    ORDER BY soh.[TotalDue] DESC)
ORDER BY soh.[TotalDue] DESC, sod.[SalesOrderDetailID]
```

<span data-ttu-id="814df-133">그런 다음 응용 프로그램이 .NET Framework `System.Xml.Linq` API를 사용하여 결과를 XML로 형식화합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-133">The application then uses the .NET Framework `System.Xml.Linq` APIs to format the results as XML.</span></span>

```csharp
// Create a new SqlCommand to run the Transact-SQL query
using (var command = new SqlCommand(...))
{
    command.Parameters.AddWithValue("@TerritoryId", id);

    // Run the query and create the initial XML document
    using (var reader = await command.ExecuteReaderAsync())
    {
        var lastOrderNumber = string.Empty;
        var doc = new XDocument();
        var orders = new XElement("Orders");
        doc.Add(orders);

        XElement lineItems = null;
        // Fetch each row in turn, format the results as XML, and add them to the XML document
        while (await reader.ReadAsync())
        {
            var orderNumber = reader["OrderNumber"].ToString();
            if (orderNumber != lastOrderNumber)
            {
                lastOrderNumber = orderNumber;

                var order = new XElement("Order");
                orders.Add(order);
                var customer = new XElement("Customer");
                lineItems = new XElement("OrderLineItems");
                order.Add(customer, lineItems);

                var orderDate = (DateTime)reader["OrderDate"];
                var totalDue = (Decimal)reader["TotalDue"];
                var reviewRequired = totalDue > 5000 ? 'Y' : 'N';

                order.Add(
                    new XAttribute("OrderNumber", orderNumber),
                    new XAttribute("Status", reader["Status"]),
                    new XAttribute("ShipDate", reader["ShipDate"]),
                    ... // More attributes, not shown.

                    var fullName = string.Join(" ",
                        reader["CustomerTitle"],
                        reader["CustomerFirstName"],
                        reader["CustomerMiddleName"],
                        reader["CustomerLastName"],
                        reader["CustomerSuffix"]
                    )
                   .Replace("  ", " ") //remove double spaces
                   .Trim()
                   .ToUpper();

               customer.Add(
                    new XAttribute("AccountNumber", reader["AccountNumber"]),
                    new XAttribute("FullName", fullName));
            }

            var productId = (int)reader["ProductID"];
            var quantity = (short)reader["Quantity"];
            var inventoryCheckRequired = (productId >= 710 && productId <= 720 && quantity >= 5) ? 'Y' : 'N';

            lineItems.Add(
                new XElement("LineItem",
                    new XAttribute("Quantity", quantity),
                    new XAttribute("UnitPrice", ((Decimal)reader["UnitPrice"]).ToString("C")),
                    new XAttribute("LineTotal", RoundAndFormat(reader["LineTotal"])),
                    new XAttribute("ProductId", productId),
                    new XAttribute("InventoryCheckRequired", inventoryCheckRequired)
                ));
        }
        // Match the exact formatting of the XML returned from SQL
        var xml = doc
            .ToString(SaveOptions.DisableFormatting)
            .Replace(" />", "/>");
    }
}
```

> [!NOTE]
> <span data-ttu-id="814df-134">이 코드는 다소 복잡합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-134">This code is somewhat complex.</span></span> <span data-ttu-id="814df-135">새 응용 프로그램의 경우 serialization 라이브러리를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-135">For a new application, you might prefer to use a serialization library.</span></span> <span data-ttu-id="814df-136">하지만 이 경우 개발 팀이 기존 응용 프로그램을 리팩터링한다고 가정하기 때문에 메서드가 원래 코드와 정확히 같은 형식을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-136">However, the assumption here is that the development team is refactoring an existing application, so the method needs to return the exact same format as the original code.</span></span>

## <a name="considerations"></a><span data-ttu-id="814df-137">고려 사항</span><span class="sxs-lookup"><span data-stu-id="814df-137">Considerations</span></span>

- <span data-ttu-id="814df-138">많은 데이터베이스 시스템은 특정 유형의 데이터 처리(예: 대규모 데이터 집합에 대한 집계 값 계산)를 수행하도록 고도로 최적화되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-138">Many database systems are highly optimized to perform certain types of data processing, such as calculating aggregate values over large datasets.</span></span> <span data-ttu-id="814df-139">이러한 유형의 처리를 데이터베이스 외부로 이동하지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="814df-139">Don't move those types of processing out of the database.</span></span>

- <span data-ttu-id="814df-140">처리를 재배치하면 데이터베이스가 네트워크를 통해 훨씬 더 많은 데이터를 전송하게 되므로 재배치하지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="814df-140">Do not relocate processing if doing so causes the database to transfer far more data over the network.</span></span> <span data-ttu-id="814df-141">[불필요한 가져오기 안티패턴][ExtraneousFetching]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="814df-141">See the [Extraneous Fetching antipattern][ExtraneousFetching].</span></span>

- <span data-ttu-id="814df-142">처리를 응용 프로그램 계층으로 이동하면 추가 작업을 처리하도록 해당 계층을 확장해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-142">If you move processing to an application tier, that tier may need to scale out to handle the additional work.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="814df-143">문제를 감지하는 방법</span><span class="sxs-lookup"><span data-stu-id="814df-143">How to detect the problem</span></span>

<span data-ttu-id="814df-144">사용량이 많은 데이터베이스의 증상에는 데이터베이스에 액세스하는 작업의 처리량과 응답 시간이 불균형적으로 감소하는 증상이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="814df-144">Symptoms of a busy database include a disproportionate decline in throughput and response times in operations that access the database.</span></span> 

<span data-ttu-id="814df-145">다음 단계를 수행하면 문제를 식별하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-145">You can perform the following steps to help identify this problem:</span></span> 

1. <span data-ttu-id="814df-146">성능 모니터링을 사용하여 프로덕션 시스템이 데이터베이스 활동을 수행하는 데 얼마나 많은 시간을 소비하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-146">Use performance monitoring to identify how much time the production system spends performing database activity.</span></span>

2. <span data-ttu-id="814df-147">이 기간 동안 데이터베이스가 수행한 작업을 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-147">Examine the work performed by the database during these periods.</span></span>

3. <span data-ttu-id="814df-148">특정 작업이 너무 많은 데이터베이스 활동을 유발한 것으로 의심되는 경우 통제된 환경에서 부하 테스트를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-148">If you suspect that particular operations might cause too much database activity, perform load testing in a controlled environment.</span></span> <span data-ttu-id="814df-149">각 테스트는 가변적인 사용자 로드와 의심스러운 작업을 혼합하여 실행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-149">Each test should run a mixture of the suspect operations with a variable user load.</span></span> <span data-ttu-id="814df-150">부하 테스트에서 원격 분석 데이터를 검사하여 데이터베이스가 어떻게 사용되는지 관찰합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-150">Examine the telemetry from the load tests to observe how the database is used.</span></span>

4. <span data-ttu-id="814df-151">데이터베이스 활동이 처리량은 많은 것으로 드러나지만 데이터 트래픽이 적다면 소스 코드를 검토하여 처리가 다른 곳에서 더 잘 수행될 수 있는지 판단합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-151">If the database activity reveals significant processing but little data traffic, review the source code to determine whether the processing can better be performed elsewhere.</span></span>

<span data-ttu-id="814df-152">데이터베이스 활동량이 작거나 응답 시간이 비교적 빠르다면 사용량이 많은 데이터베이스가 성능 문제일 가능성이 낮습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-152">If the volume of database activity is low or response times are relatively fast, then a busy database is unlikely to be a performance problem.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="814df-153">예제 진단</span><span class="sxs-lookup"><span data-stu-id="814df-153">Example diagnosis</span></span>

<span data-ttu-id="814df-154">다음 섹션에서는 이러한 단계를 앞에서 설명한 응용 프로그램 예제에 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-154">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-the-volume-of-database-activity"></a><span data-ttu-id="814df-155">데이터베이스 활동 볼륨 모니터링</span><span class="sxs-lookup"><span data-stu-id="814df-155">Monitor the volume of database activity</span></span>

<span data-ttu-id="814df-156">다음 그래프는 동시 사용자 최대 50명의 단계 부하를 사용하여 응용 프로그램 예제에 부하 테스트를 실행한 결과를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="814df-156">The following graph shows the results of running a load test against the sample application, using a step load of up to 50 concurrent users.</span></span> <span data-ttu-id="814df-157">요청 볼륨은 빠르게 한도에 도달하여 그 수준에 머무르고 평균 응답 시간은 꾸준히 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-157">The volume of requests quickly reaches a limit and stays at that level, while the average response time steadily increases.</span></span> <span data-ttu-id="814df-158">두 메트릭에 로그 눈금 간격이 사용되었습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-158">Note that a logarithmic scale is used for those two metrics.</span></span>

![데이터베이스에서 처리 수행에 대한 부하 테스트 결과][ProcessingInDatabaseLoadTest]

<span data-ttu-id="814df-160">다음 그래프는 CPU 사용률과 DTU를 서비스 할당량의 백분율로 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="814df-160">The next graph shows CPU utilization and DTUs as a percentage of service quota.</span></span> <span data-ttu-id="814df-161">DTU는 데이터베이스가 수행하는 처리량을 측정한 값을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-161">DTUs provides a measure of how much processing the database performs.</span></span> <span data-ttu-id="814df-162">이 그래프에서 CPU와 DTU 사용률 모두 100%에 신속하게 도달합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-162">The graph shows that CPU and DTU utilization both quickly reached 100%.</span></span>

![처리를 수행하는 동안 데이터베이스의 성능을 보여주는 Azure SQL Database 모니터링][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a><span data-ttu-id="814df-164">데이터베이스가 수행한 작업 검사</span><span class="sxs-lookup"><span data-stu-id="814df-164">Examine the work performed by the database</span></span>

<span data-ttu-id="814df-165">데이터베이스가 수행한 작업은 처리가 아니라 실제 데이터 액세스 작업이므로 데이터베이스를 사용하고 있을 때 실행 중인 SQL 문을 이해하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-165">It could be that the tasks performed by the database are genuine data access operations, rather than processing, so it is important to understand the SQL statements being run while the database is busy.</span></span> <span data-ttu-id="814df-166">시스템을 모니터링하여 SQL 트래픽을 캡처하고 SQL 작업과 응용 프로그램 요청의 상관 관계를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="814df-166">Monitor the system to capture the SQL traffic and correlate the SQL operations with application requests.</span></span>

<span data-ttu-id="814df-167">데이터베이스 작업이 처리가 포함되지 않은 순수한 데이터 액세스 작업인 경우에는 [불필요한 가져오기][ExtraneousFetching]가 문제일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-167">If the database operations are purely data access operations, without a lot of processing, then the problem might be [Extraneous Fetching][ExtraneousFetching].</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="814df-168">솔루션 구현 및 결과 확인</span><span class="sxs-lookup"><span data-stu-id="814df-168">Implement the solution and verify the result</span></span>

<span data-ttu-id="814df-169">다음 그래프는 업데이트된 코드를 사용한 부하 테스트를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="814df-169">The following graph shows a load test using the updated code.</span></span> <span data-ttu-id="814df-170">처리량이 초당 요청이 400건 이상으로 이전의 12보다 상당히 높습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-170">Throughput is significantly higher, over 400 requests per second versus 12 earlier.</span></span> <span data-ttu-id="814df-171">평균 응답 시간도 0.1초를 살짝 넘어 4초 이상에 비해 훨씬 적습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-171">The average response time is also much lower, just above 0.1 seconds compared to over 4 seconds.</span></span>

![데이터베이스에서 처리 수행에 대한 부하 테스트 결과][ProcessingInClientApplicationLoadTest]

<span data-ttu-id="814df-173">CPU 및 DTU 사용률을 보면 처리량 증가에도 불구하고 시스템이 포화 상태에 도달하는데 더 오래 걸렸습니다.</span><span class="sxs-lookup"><span data-stu-id="814df-173">CPU and DTU utilization shows that the system took longer to reach saturation, despite the increased throughput.</span></span>

![클라이언트 응용 프로그램에서 처리를 수행하는 동안 데이터베이스의 성능을 보여주는 Azure SQL Database 모니터링][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a><span data-ttu-id="814df-175">관련 리소스</span><span class="sxs-lookup"><span data-stu-id="814df-175">Related resources</span></span> 

- <span data-ttu-id="814df-176">[불필요한 가져오기 안티패턴][ExtraneousFetching]</span><span class="sxs-lookup"><span data-stu-id="814df-176">[Extraneous Fetching antipattern][ExtraneousFetching]</span></span>


[dtu]: /sql-database/sql-database-what-is-a-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
