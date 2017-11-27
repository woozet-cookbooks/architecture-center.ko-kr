---
title: "불필요한 가져오기 안티패턴"
description: "비즈니스 작업에 필요한 것보다 더 많은 데이터를 검색하면 불필요한 I/O 오버헤드가 발생하고 응답성이 감소할 수 있습니다."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 8a808dce62a1c80c126b7b1df536f74c46726ea1
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="extraneous-fetching-antipattern"></a><span data-ttu-id="63ec5-103">불필요한 가져오기 안티패턴</span><span class="sxs-lookup"><span data-stu-id="63ec5-103">Extraneous Fetching antipattern</span></span>

<span data-ttu-id="63ec5-104">비즈니스 작업에 필요한 것보다 더 많은 데이터를 검색하면 불필요한 I/O 오버헤드가 발생하고 응답성이 감소할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-104">Retrieving more data than needed for a business operation can result in unnecessary I/O overhead and reduce responsiveness.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="63ec5-105">문제 설명</span><span class="sxs-lookup"><span data-stu-id="63ec5-105">Problem description</span></span>

<span data-ttu-id="63ec5-106">이 안티패턴은 응용 프로그램이 필요*할 수 있는* 모든 데이터를 검색하여 I/O 요청의 최소화를 시도하는 경우 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-106">This antipattern can occur if the application tries to minimize I/O requests by retrieving all of the data that it *might* need.</span></span> <span data-ttu-id="63ec5-107">이는 흔히 [번잡한 I/O][chatty-io] 안티패턴을 과도하게 보정한 결과입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-107">This is often a result of overcompensating for the [Chatty I/O][chatty-io] antipattern.</span></span> <span data-ttu-id="63ec5-108">예를 들어 응용 프로그램이 데이터베이스의 모든 제품에 대한 세부정보를 가져올 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-108">For example, an application might fetch the details for every product in a database.</span></span> <span data-ttu-id="63ec5-109">그러나 사용자는 해당 세부정보의 일부만 필요하며(일부는 고객에게 유용하지 않을 수 있음) 아마도 *모든* 제품을 동시에 볼 필요가 없을 것입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-109">But the user may need just a subset of the details (some may not be relevant to customers), and probably doesn't need to see *all* of the products at once.</span></span> <span data-ttu-id="63ec5-110">사용자가 전체 카탈로그를 탐색한다 하더라도 결과를 한 번에 &mdash; 20개씩 보여 주는 페이지로 나누는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-110">Even if the user is browsing the entire catalog, it would make sense to paginate the results &mdash; showing 20 at a time, for example.</span></span>

<span data-ttu-id="63ec5-111">이 문제의 또 다른 출처는 좋지 않은 프로그래밍 또는 설계 관행을 따르는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-111">Another source of this problem is following poor programming or design practices.</span></span> <span data-ttu-id="63ec5-112">예를 들어 다음 코드는 Entity Framework를 사용하여 모든 제품에 전체 세부정보를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-112">For example, the following code uses Entity Framework to fetch the complete details for every product.</span></span> <span data-ttu-id="63ec5-113">그런 다음 결과를 필터링하여 필드의 일부만 반환하고 나머지는 버립니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-113">Then it filters the results to return only a subset of the fields, discarding the rest.</span></span> <span data-ttu-id="63ec5-114">전체 샘플은 [여기][sample-app]에서 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> GetAllFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Execute the query. This happens at the database.
        var products = await context.Products.ToListAsync();

        // Project fields from the query results. This happens in application memory.
        var result = products.Select(p => new ProductInfo { Id = p.ProductId, Name = p.Name });
        return Ok(result);
    }
}
```

<span data-ttu-id="63ec5-115">다음 예제에서 응용 프로그램은 데이터베이스 대신 수행할 수 있는 집계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-115">In the next example, the application retrieves data to perform an aggregation that could be done by the database instead.</span></span> <span data-ttu-id="63ec5-116">이 응용 프로그램은 판매한 모든 주문에 대한 모든 기록을 가져온 다음 해당 레코드에 대한 합계를 계산하여 총 매출을 계산합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-116">The application calculates total sales by getting every record for all orders sold, and then computing the sum over those records.</span></span> <span data-ttu-id="63ec5-117">전체 샘플은 [여기][sample-app]에서 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-117">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnClientAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Fetch all order totals from the database.
        var orderAmounts = await context.SalesOrderHeaders.Select(soh => soh.TotalDue).ToListAsync();

        // Sum the order totals in memory.
        var total = orderAmounts.Sum();
        return Ok(total);
    }
}
```

<span data-ttu-id="63ec5-118">다음 예제에서는 Entity Framework가 LINQ to Entities를 사용하는 방법 때문에 발생하는 미묘한 문제를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-118">The next example shows a subtle problem caused by the way Entity Framework uses LINQ to Entities.</span></span> 

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

<span data-ttu-id="63ec5-119">응용 프로그램은 `SellStartDate`주 이상 오래된 제품을 찾으려고 합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-119">The application is trying to find products with a `SellStartDate` more than a week old.</span></span> <span data-ttu-id="63ec5-120">대부분의 경우 LINQ to Entities는 `where` 절을 데이터베이스에서 실행되는 SQL 문으로 변환합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-120">In most cases, LINQ to Entities would translate a `where` clause to a SQL statement that is executed by the database.</span></span> <span data-ttu-id="63ec5-121">그러나 이 경우 LINQ to Entities는 `AddDays` 메서드를 SQL에 매핑할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-121">In this case, however, LINQ to Entities cannot map the `AddDays` method to SQL.</span></span> <span data-ttu-id="63ec5-122">그 대신에 `Product` 테이블의 모든 행이 반환되며 그 결과가 메모리에서 필터링됩니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-122">Instead, every row from the `Product` table is returned, and the results are filtered in memory.</span></span> 

<span data-ttu-id="63ec5-123">`AsEnumerable` 호출이 바로 문제가 있다는 힌트입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-123">The call to `AsEnumerable` is a hint that there is a problem.</span></span> <span data-ttu-id="63ec5-124">이 메서드는 결과를 `IEnumerable` 인터페이스로 변환합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-124">This method converts the results to an `IEnumerable` interface.</span></span> <span data-ttu-id="63ec5-125">`IEnumerable`은 필터링을 지원하지만 이 필터링은 데이터베이스가 아닌 *클라이언트* 쪽에서 수행됩니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-125">Although `IEnumerable` supports filtering, the filtering is done on the *client* side, not the database.</span></span> <span data-ttu-id="63ec5-126">기본적으로 LINQ to Entities는 `IQueryable`을 사용하여 필터링 책임을 데이터 원본에 넘깁니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-126">By default, LINQ to Entities uses `IQueryable`, which passes the responsibility for filtering to the data source.</span></span> 

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="63ec5-127">문제를 해결하는 방법</span><span class="sxs-lookup"><span data-stu-id="63ec5-127">How to fix the problem</span></span>

<span data-ttu-id="63ec5-128">금세 오래되거나 삭제될 수 있는 대량의 데이터를 가져오지 않고 수행 중인 작업에 필요한 데이터만 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-128">Avoid fetching large volumes of data that may quickly become outdated or might be discarded, and only fetch the data needed for the operation being performed.</span></span> 

<span data-ttu-id="63ec5-129">테이블의 모든 열을 가져와서 필터링하는 대신에 데이터베이스에서 필요한 열을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-129">Instead of getting every column from a table and then filtering them, select the columns that you need from the database.</span></span>

```csharp
public async Task<IHttpActionResult> GetRequiredFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Project fields as part of the query itself
        var result = await context.Products
            .Select(p => new ProductInfo {Id = p.ProductId, Name = p.Name})
            .ToListAsync();
        return Ok(result);
    }
}
```

<span data-ttu-id="63ec5-130">마찬가지로 응용 프로그램 메모리가 아닌 데이터베이스에서 집계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-130">Similarly, perform aggregation in the database and not in application memory.</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnDatabaseAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Sum the order totals as part of the database query.
        var total = await context.SalesOrderHeaders.SumAsync(soh => soh.TotalDue);
        return Ok(total);
    }
}
```

<span data-ttu-id="63ec5-131">Entity Framework를 사용하는 경우 LINQ 쿼리가 `IEnumerable`이 아닌 `IQueryable` 인터페이스를 사용하여 확인되도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-131">When using Entity Framework, ensure that LINQ queries are resolved using the `IQueryable`interface and not `IEnumerable`.</span></span> <span data-ttu-id="63ec5-132">데이터 원본에 매핑할 수 있는 함수만 사용하여 쿼리를 조정해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-132">You may need to adjust the query to use only functions that can be mapped to the data source.</span></span> <span data-ttu-id="63ec5-133">앞의 예제를 리팩터링하여 쿼리에서 `AddDays` 메서드를 제거하면 데이터베이스에서 필터링을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-133">The earlier example can be refactored to remove the `AddDays` method from the query, allowing filtering to be done by the database.</span></span>

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out. 
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a><span data-ttu-id="63ec5-134">고려 사항</span><span class="sxs-lookup"><span data-stu-id="63ec5-134">Considerations</span></span>

- <span data-ttu-id="63ec5-135">경우에 따라 데이터를 행 분할하여 성능을 개선할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-135">In some cases, you can improve performance by partitioning data horizontally.</span></span> <span data-ttu-id="63ec5-136">서로 다른 작업이 데이터의 서로 다른 특성에 액세스하는 경우 행 분할로 경합을 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-136">If different operations access different attributes of the data, horizontal partitioning may reduce contention.</span></span> <span data-ttu-id="63ec5-137">흔히 대부분의 작업은 데이터의 작은 하위 집합에 대해 실행되므로 이러한 부하 분산으로 성능이 개선될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-137">Often, most operations are run against a small subset of the data, so spreading this load may improve performance.</span></span> <span data-ttu-id="63ec5-138">[데이터 분할][data-partitioning]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="63ec5-138">See [Data partitioning][data-partitioning].</span></span>

- <span data-ttu-id="63ec5-139">바인딩되지 않은 쿼리를 지원해야 하는 작업의 경우 페이지 매김을 구현하고 한 번에 제한된 수의 엔터티만 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-139">For operations that have to support unbounded queries, implement pagination and only fetch a limited number of entities at a time.</span></span> <span data-ttu-id="63ec5-140">예를 들어 고객이 제품 카탈로그를 검색하는 경우 결과를 한 번에 한 페이지씩 표시할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-140">For example, if a customer is browsing a product catalog, you can show one page of results at a time.</span></span>

- <span data-ttu-id="63ec5-141">가능하면 데이터 저장소에 기본 제공되는 기능을 이용합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-141">When possible, take advantage of features built into the data store.</span></span> <span data-ttu-id="63ec5-142">예를 들어 SQL Database는 대개 집계 함수를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-142">For example, SQL databases typically provide aggregate functions.</span></span> 

- <span data-ttu-id="63ec5-143">집계 같은 특정 기능을 지원하지 않는 데이터 원본을 사용하는 경우 계산된 결과를 다른 곳에 저장하고 레코드가 추가되거나 업데이트될 때 값을 업데이트할 수 있으므로 응용 프로그램이 필요할 때마다 값을 다시 계산할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-143">If you're using a data store that doesn't support a particular function, such as aggregration, you could store the calculated result elsewhere, updating the value as records are added or updated, so the application doesn't have to recalculate the value each time it's needed.</span></span>

- <span data-ttu-id="63ec5-144">요청이 다수의 필드를 검색하는 것으로 보이면 소스 코드를 검토하여 이 필드가 모두 실제로 필요한지 여부를 결정합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-144">If you see that requests are retrieving a large number of fields, examine the source code to determine whether all of these fields are actually necessary.</span></span> <span data-ttu-id="63ec5-145">경우에 따라 이러한 요청은 잘못 설계된 `SELECT *` 쿼리의 결과입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-145">Sometimes these requests are the result of poorly designed `SELECT *` query.</span></span> 

- <span data-ttu-id="63ec5-146">마찬가지로 다수의 엔터티를 검색하는 요청은 응용 프로그램이 데이터를 올바르게 필터링하지 않는다는 신호일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-146">Similarly, requests that retrieve a large number of entities may be sign that the application is not filtering data correctly.</span></span> <span data-ttu-id="63ec5-147">이러한 엔터티가 모두 실제로 필요한지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-147">Verify that all of these entities are actually needed.</span></span> <span data-ttu-id="63ec5-148">가능하면 데이터베이스 쪽 필터링을 사용합니다. 예를 들어 SQL의 `WHERE` 절을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-148">Use database-side filtering if possible, for example, by using `WHERE` clauses in SQL.</span></span> 

- <span data-ttu-id="63ec5-149">처리를 데이터베이스에서 오프로드로 수행하는 방법이 항상 최선의 옵션인 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-149">Offloading processing to the database is not always the best option.</span></span> <span data-ttu-id="63ec5-150">이 전략은 데이터베이스가 그렇게 하도록 설계되거나 최적화된 경우에만 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-150">Only use this strategy when the database is designed or optimized to do so.</span></span> <span data-ttu-id="63ec5-151">대부분의 데이터베이스 시스템은 특정 기능에 대해 고도로 최적화되지만 범용 응용 프로그램 엔진으로 작동하도록 설계되어 있지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-151">Most database systems are highly optimized for certain functions, but are not designed to act as general-purpose application engines.</span></span> <span data-ttu-id="63ec5-152">자세한 내용은 [사용량이 많은 데이터베이스 안티패턴][BusyDatabase]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="63ec5-152">For more information, see the [Busy Database antipattern][BusyDatabase].</span></span>


## <a name="how-to-detect-the-problem"></a><span data-ttu-id="63ec5-153">문제를 감지하는 방법</span><span class="sxs-lookup"><span data-stu-id="63ec5-153">How to detect the problem</span></span>

<span data-ttu-id="63ec5-154">불필요한 가져오기의 증상에는 높은 대기 시간과 낮은 처리량이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-154">Symptoms of extraneous fetching include high latency and low throughput.</span></span> <span data-ttu-id="63ec5-155">데이터를 데이터 원본에서 검색하는 경우 경합이 증가할 확률도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-155">If the data is retrieved from a data store, increased contention is also probable.</span></span> <span data-ttu-id="63ec5-156">응답 시간이 길어지거나 서비스 시간 초과로 인해 장애가 발생하면 최종 사용자가 보고할 가능성이 높습니다. 이러한 장애는 HTTP 500(내부 서버) 오류 또는 HTTP 503(서비스를 사용할 수 없음) 오류를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-156">End users are likely to report extended response times or failures caused by services timing out. These failures could return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="63ec5-157">웹 서버의 이벤트 로그를 검사하십시오. 오류의 원인과 상황에 대한 자세한 정보가 포함되어 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-157">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="63ec5-158">이 안티패턴 및 가져온 일부 원격 분석 데이터의 증상은 [모놀리식 지속성 안티패턴][MonolithicPersistence]과 매우 유사할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-158">The symptoms of this antipattern and some of the telemetry obtained might be very similar to those of the [Monolithic Persistence antipattern][MonolithicPersistence].</span></span> 

<span data-ttu-id="63ec5-159">다음 단계를 수행하면 원인을 식별하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-159">You can perform the following steps to help identify the cause:</span></span>

1. <span data-ttu-id="63ec5-160">부하 테스트, 프로세스 모니터링 또는 다른 계측 데이터 수집 방법을 수행하여 느린 워크로드 또는 트랜잭션을 식별합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-160">Identify slow workloads or transactions by performing load-testing, process monitoring, or other methods of capturing instrumentation data.</span></span>
2. <span data-ttu-id="63ec5-161">시스템에서 보여 주는 동작 패턴을 관찰합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-161">Observe any behavioral patterns exhibited by the system.</span></span> <span data-ttu-id="63ec5-162">초당 트랜잭션 수 또는 사용자 볼륨 측면에서 특정 한계가 있습니까?</span><span class="sxs-lookup"><span data-stu-id="63ec5-162">Are there particular limits in terms of transactions per second or volume of users?</span></span>
3. <span data-ttu-id="63ec5-163">느린 워크로드의 인스턴스와 동작 패턴의 상관성을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-163">Correlate the instances of slow workloads with behavioral patterns.</span></span>
4. <span data-ttu-id="63ec5-164">사용 중인 데이터 저장소를 식별합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-164">Identify the data stores being used.</span></span> <span data-ttu-id="63ec5-165">각 데이터 원본에 대해 낮은 수준의 원격 분석을 실행하여 작업의 동작을 관찰합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-165">For each data source, run lower level telemetry to observe the behavior of operations.</span></span>
6. <span data-ttu-id="63ec5-166">이러한 데이터 원본을 참조하는 느리게 실행되는 쿼리를 식별합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-166">Identify any slow-running queries that reference these data sources.</span></span>
7. <span data-ttu-id="63ec5-167">느리게 실행되는 쿼리에 대해 리소스별 분석을 수행하고 데이터가 사용 및 소비되는 방법을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-167">Perform a resource-specific analysis of the slow-running queries and ascertain how the data is used and consumed.</span></span>

<span data-ttu-id="63ec5-168">다음과 같은 증상을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-168">Look for any of these symptoms:</span></span>

- <span data-ttu-id="63ec5-169">동일한 리소스 또는 데이터 저장소에 대한 대규모 I/O 요청이 자주 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-169">Frequent, large I/O requests made to the same resource or data store.</span></span>
- <span data-ttu-id="63ec5-170">공유 리소스 또는 데이터 저장소의 경합.</span><span class="sxs-lookup"><span data-stu-id="63ec5-170">Contention in a shared resource or data store.</span></span>
- <span data-ttu-id="63ec5-171">네트워크를 통해 대용량의 데이터를 자주 수신하는 작업.</span><span class="sxs-lookup"><span data-stu-id="63ec5-171">An operation that frequently receives large volumes of data over the network.</span></span>
- <span data-ttu-id="63ec5-172">I/O가 완료될 때까지 대기하면서 상당한 시간을 소비하는 응용 프로그램 및 서비스.</span><span class="sxs-lookup"><span data-stu-id="63ec5-172">Applications and services spending significant time waiting for I/O to complete.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="63ec5-173">예제 진단</span><span class="sxs-lookup"><span data-stu-id="63ec5-173">Example diagnosis</span></span>    

<span data-ttu-id="63ec5-174">다음 섹션은 이러한 단계를 앞의 예제에 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-174">The following sections apply these steps to the previous examples.</span></span>

### <a name="identify-slow-workloads"></a><span data-ttu-id="63ec5-175">느린 워크로드 식별</span><span class="sxs-lookup"><span data-stu-id="63ec5-175">Identify slow workloads</span></span>

<span data-ttu-id="63ec5-176">이 그래프는 앞에 나온 `GetAllFieldsAsync` 메서드를 실행하는 최대 400명의 동시 사용자를 시뮬레이션한 부하 테스트의 성능 결과를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-176">This graph shows performance results from a load test that simulated up to 400 concurrent users running the `GetAllFieldsAsync` method shown earlier.</span></span> <span data-ttu-id="63ec5-177">처리량은 부하가 증가함에 따라 서서히 줄어듭니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-177">Throughput diminishes slowly as the load increases.</span></span> <span data-ttu-id="63ec5-178">평균 응답 시간은 워크로드가 증가함에 따라 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-178">Average response time goes up as the workload increases.</span></span> 

![GetAllFieldsAsync 메서드에 대한 부하 테스트 결과][Load-Test-Results-Client-Side1]

<span data-ttu-id="63ec5-180">`AggregateOnClientAsync` 작업에 대한 부하 테스트도 유사한 패턴을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-180">A load test for the `AggregateOnClientAsync` operation shows a similar pattern.</span></span> <span data-ttu-id="63ec5-181">요청의 양은 비교적 안정적입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-181">The volume of requests is reasonably stable.</span></span> <span data-ttu-id="63ec5-182">평균 응답 시간은 워크로드에 따라 증가하지만 이전 그래프보다 더 서서히 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-182">The average response time increases with the workload, although more slowly than the previous graph.</span></span>

![AggregateOnClientAsync 메서드에 대한 부하 테스트 결과][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a><span data-ttu-id="63ec5-184">느린 워크로드와 동작 패턴의 상관성을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-184">Correlate slow workloads with behavioral patterns</span></span>

<span data-ttu-id="63ec5-185">높은 사용량의 일반적인 기간과 느린 성능 간의 상관성을 확인하면 문제 영역이 나타날 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-185">Any correlation between regular periods of high usage and slowing performance can indicate areas of concern.</span></span> <span data-ttu-id="63ec5-186">느린 실행이 의심되는 기능의 성능 프로필을 면밀히 검토하여 앞서 수행한 부하 테스트와 일치하는지 여부를 결정합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-186">Closely examine the performance profile of functionality that is suspected to be slow running, to determine whether it matches the load testing performed earlier.</span></span>

<span data-ttu-id="63ec5-187">단계별 사용자 부하를 사용하여 동일한 기능을 부하 테스트하여 성능이 크게 저하되거나 완전히 장애를 일으키는 지점을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-187">Load test the same functionality using step-based user loads, to find the point where performance drops significantly or fails completely.</span></span> <span data-ttu-id="63ec5-188">해당 지점이 예상한 실제 사용률의 경계 이내인 경우 기능이 구현된 방법을 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-188">If that point falls within the bounds of your expected real-world usage, examine how the functionality is implemented.</span></span>

<span data-ttu-id="63ec5-189">시스템이 스트레스를 받을 때 수행되지 않고 시간에 민감하지 않고 다른 중요한 작업의 성능에 부정적인 영향을 미치지 않는다면 느린 작업이 반드시 문제인 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-189">A slow operation is not necessarily a problem, if it is not being performed when the system is under stress, is not time critical, and does not negatively affect the performance of other important operations.</span></span> <span data-ttu-id="63ec5-190">예를 들어 월간 작업 통계 작성은 오래 실행되는 작업일 수 있지만 이 작업은 아마 일괄 처리 프로세스로 수행되고 낮은 우선 순위의 작업으로 실행될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-190">For example, generating monthly operational statistics might be a long-running operation, but it can probably be performed as a batch process and run as a low priority job.</span></span> <span data-ttu-id="63ec5-191">반면에 제품 카탈로그를 쿼리하는 고객은 중요한 비즈니스 작업입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-191">On the other hand, customers querying the product catalog is a critical business operation.</span></span> <span data-ttu-id="63ec5-192">이처럼 중요한 작업에서 생성된 원격 분석에 초점을 맞추고 사용량이 많은 기간 동안 성능이 어떻게 변화하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-192">Focus on the telemetry generated by these critical operations to see how the performance varies during periods of high usage.</span></span>

### <a name="identify-data-sources-in-slow-workloads"></a><span data-ttu-id="63ec5-193">느린 워크로드의 데이터 원본 식별</span><span class="sxs-lookup"><span data-stu-id="63ec5-193">Identify data sources in slow workloads</span></span>

<span data-ttu-id="63ec5-194">서비스가 데이터 검색 방법 때문에 성능이 저하되는 것으로 의심되는 경우 응용 프로그램이 사용하는 리포지토리와 어떻게 상호작용하는지 조사합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-194">If you suspect that a service is performing poorly because of the way it retrieves data, investigate how the application interacts with the repositories it uses.</span></span> <span data-ttu-id="63ec5-195">실시간 시스템을 모니터링하여 성능이 저하된 기간 동안 액세스되는 원본을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-195">Monitor the live system to see which sources are accessed during periods of poor performance.</span></span> 

<span data-ttu-id="63ec5-196">각 데이터 원본에 대해 시스템을 계측하여 다음 정보를 수집합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-196">For each data source, instrument the system to capture the following:</span></span>

- <span data-ttu-id="63ec5-197">각 데이터 저장소가 액세스되는 빈도.</span><span class="sxs-lookup"><span data-stu-id="63ec5-197">The frequency that each data store is accessed.</span></span>
- <span data-ttu-id="63ec5-198">데이터 저장소에 들어오고 나가는 데이터의 양.</span><span class="sxs-lookup"><span data-stu-id="63ec5-198">The volume of data entering and exiting the data store.</span></span>
- <span data-ttu-id="63ec5-199">이러한 작업의 타이밍, 특히 요청의 대기 시간.</span><span class="sxs-lookup"><span data-stu-id="63ec5-199">The timing of these operations, especially the latency of requests.</span></span>
- <span data-ttu-id="63ec5-200">전형적인 부하에서 각 데이터 저장소에 액세스하는 동안 발생하는 오류의 특성 및 비율.</span><span class="sxs-lookup"><span data-stu-id="63ec5-200">The nature and rate of any errors that occur while accessing each data store under typical load.</span></span>

<span data-ttu-id="63ec5-201">이 정보를 응용 프로그램에서 클라이언트에 반환하는 데이터의 양과 비교합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-201">Compare this information against the volume of data being returned by the application to the client.</span></span> <span data-ttu-id="63ec5-202">데이터 저장소에서 반환하는 데이터 양과 클라이언트에 반환되는 데이터 양의 비율을 추적합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-202">Track the ratio of the volume of data returned by the data store against the volume of data returned to the client.</span></span> <span data-ttu-id="63ec5-203">큰 차이가 있는 경우 응용 프로그램이 필요하지 않은 데이터를 가져오는지 여부를 조사하여 결정합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-203">If there is any large disparity, investigate to determine whether the application is fetching data that it doesn't need.</span></span>

<span data-ttu-id="63ec5-204">실시간 시스템을 관찰하고 각 사용자 요청의 수명 주기를 추적하면 이 데이터를 포착할 수 있습니다. 또는 일련의 합성 워크로드를 모델링하고 테스트 시스템에 대해 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-204">You may be able to capture this data by observing the live system and tracing the lifecycle of each user request, or you can model a series of synthetic workloads and run them against a test system.</span></span>

<span data-ttu-id="63ec5-205">다음 그래프는 `GetAllFieldsAsync` 메서드의 부하 테스트 중에 [새 Relic APM][new-relic]을 사용하여 수집한 원격 분석 데이터를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-205">The following graphs show telemetry captured using [New Relic APM][new-relic] during a load test of the `GetAllFieldsAsync` method.</span></span> <span data-ttu-id="63ec5-206">데이터베이스에서 수신된 데이터 양과 해당 HTTP 응답 간의 차이에 주목합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-206">Note the difference between the volumes of data received from the database and the corresponding HTTP responses.</span></span>

![`GetAllFieldsAsync` 메서드에 대한 원격 분석 데이터][TelemetryAllFields]

<span data-ttu-id="63ec5-208">각 요청에 대해 데이터는 80,503바이트를 반환했지만 클라이언트에 대한 응답은 19,855바이트만 포함했으며, 이는 데이터베이스 응답 크기의 약 25%에 해당합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-208">For each request, the database returned 80,503 bytes, but the response to the client only contained 19,855 bytes, about 25% of the size of the database response.</span></span> <span data-ttu-id="63ec5-209">클라이언트에서 반환되는 데이터의 크기는 형식에 따라 달라질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-209">The size of the data returned to the client can vary depending on the format.</span></span> <span data-ttu-id="63ec5-210">이 부하 테스트의 경우 클라이언트는 JSON 데이터를 요청했습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-210">For this load test, the client requested JSON data.</span></span> <span data-ttu-id="63ec5-211">XML(표시되지 않음)을 사용한 별도의 테스트에서는 응답 크기가 35,655바이트였으며, 이는 데이터베이스 응답 크기의 44%에 해당합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-211">Separate testing using XML (not shown) had a response size of 35,655 bytes, or 44% of the size of the database response.</span></span>

<span data-ttu-id="63ec5-212">`AggregateOnClientAsync` 메서드에 대한 부하 테스트는 더 극단적인 결과를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-212">The load test for the `AggregateOnClientAsync` method shows more extreme results.</span></span> <span data-ttu-id="63ec5-213">이 경우 각 테스트는 데이터베이스에서 280Kb가 넘는 데이터를 검색하는 쿼리를 수행했지만 JSON 응답은 불과 14바이트였습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-213">In this case, each test performed a query that retrieved over 280Kb of data from the database, but the JSON response was a mere 14 bytes.</span></span> <span data-ttu-id="63ec5-214">이처럼 큰 차이는 해당 메서드가 대량의 데이터에서 집계한 결과를 계산하기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-214">The wide disparity is because the method calculates an aggregated result from a large volume of data.</span></span>

![`AggregateOnClientAsync` 메서드에 대한 원격 분석][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a><span data-ttu-id="63ec5-216">느린 쿼리 식별 및 분석</span><span class="sxs-lookup"><span data-stu-id="63ec5-216">Identify and analyze slow queries</span></span>

<span data-ttu-id="63ec5-217">가장 많은 리소스를 사용하고 실행 시간이 가장 오래 걸리는 데이터베이스 쿼리를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-217">Look for database queries that consume the most resources and take the most time to execute.</span></span> <span data-ttu-id="63ec5-218">많은 데이터베이스 작업에 대한 시작 및 완료 시간을 찾는 계측을 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-218">You can add instrumentation to find the start and completion times for many database operations.</span></span> <span data-ttu-id="63ec5-219">또한 많은 데이터 저장소는 쿼리가 수행되고 최적화되는 방법에 관한 심층 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-219">Many data stores also provide in-depth information on how queries are performed and optimized.</span></span> <span data-ttu-id="63ec5-220">예를 들어 Azure SQL Database 관리 포털의 쿼리 성능 창에서 쿼리를 선택하고 자세한 런타임 성능 정보를 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-220">For example, the Query Performance pane in the Azure SQL Database management portal lets you select a query and view detailed runtime performance information.</span></span> <span data-ttu-id="63ec5-221">다음은 `GetAllFieldsAsync` 작업에서 생성된 쿼리입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-221">Here is the query generated by the `GetAllFieldsAsync` operation:</span></span>

![Windows Azure SQL Database 관리 포털의 쿼리 세부 정보 창][QueryDetails]


## <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="63ec5-223">솔루션 구현 및 결과 확인</span><span class="sxs-lookup"><span data-stu-id="63ec5-223">Implement the solution and verify the result</span></span>

<span data-ttu-id="63ec5-224">`GetRequiredFieldsAsync` 메서드를 데이터베이스 쪽의 SELECT 문을 사용하도록 변경하면 부하 테스트에서 다음과 같은 결과를 표시했습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-224">After changing the `GetRequiredFieldsAsync` method to use a SELECT statement on the database side, load testing showed the following results.</span></span>

![GetRequiredFieldsAsync 메서드에 대한 부하 테스트 결과][Load-Test-Results-Database-Side1]

<span data-ttu-id="63ec5-226">이 부하 테스트는 전과 동일한 배포 및 동시 사용자 수 400명의 동일한 시뮬레이션 워크로드를 사용했습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-226">This load test used the same deployment and the same simulated workload of 400 concurrent users as before.</span></span> <span data-ttu-id="63ec5-227">그래프가 훨씬 더 낮은 대기 시간을 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-227">The graph shows much lower latency.</span></span> <span data-ttu-id="63ec5-228">응답 시간은 부하에 따라 약 1.3초까지 증가하며, 이는 앞의 경우 4초와 비교하여 더 짧습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-228">Response time rises with load to approximately 1.3 seconds, compared to 4 seconds in the previous case.</span></span> <span data-ttu-id="63ec5-229">처리량도 이전의 초당 요청 100개에 비해 350개로 더 높습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-229">The throughput is also higher at 350 requests per second compared to 100 earlier.</span></span> <span data-ttu-id="63ec5-230">데이터베이스에 검색된 데이터 양은 이제 HTTP 응답 메시지의 크기와 거의 일치합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-230">The volume of data retrieved from the database now closely matches the size of the HTTP response messages.</span></span>

![`GetRequiredFieldsAsync` 메서드에 대한 원격 분석][TelemetryRequiredFields]

<span data-ttu-id="63ec5-232">`AggregateOnDatabaseAsync` 메서드를 사용한 부하 테스트에서 다음 결과를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-232">Load testing using the `AggregateOnDatabaseAsync` method generates the following results:</span></span>

![AggregateOnDatabaseAsync 메서드에 대한 부하 테스트 결과][Load-Test-Results-Database-Side2]

<span data-ttu-id="63ec5-234">이제 평균 응답 시간이 최소화되었습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-234">The average response time is now minimal.</span></span> <span data-ttu-id="63ec5-235">이는 주로 데이터베이스에서의 I/O가 대폭 감소했기 때문에 성능이 크게 개선된 결과입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-235">This is an order of magnitude improvement in performance, caused primarily by the large reduction in I/O from the database.</span></span> 

<span data-ttu-id="63ec5-236">다음은 `AggregateOnDatabaseAsync` 메서드에 대한 해당 원격 분석 데이터입니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-236">Here is the corresponding telemetry for the `AggregateOnDatabaseAsync` method.</span></span> <span data-ttu-id="63ec5-237">데이터베이스에서 검색한 데이터의 양은 트랜잭션당 280Kb 이상에서 53바이트로 대폭 감소했습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-237">The amount of data retrieved from the database was vastly reduced, from over 280Kb per transaction to 53 bytes.</span></span> <span data-ttu-id="63ec5-238">결과적으로 분당 최대 지속 요청 수는 약 2,000에서 25,000 이상으로 증가했습니다.</span><span class="sxs-lookup"><span data-stu-id="63ec5-238">As a result, the maximum sustained number of requests per minute was raised from around 2,000 to over 25,000.</span></span>

![`AggregateOnDatabaseAsync` 메서드에 대한 원격 분석 데이터][TelemetryAggregateInDatabaseAsync]


## <a name="related-resources"></a><span data-ttu-id="63ec5-240">관련 리소스</span><span class="sxs-lookup"><span data-stu-id="63ec5-240">Related resources</span></span>

- <span data-ttu-id="63ec5-241">[사용량이 많은 데이터베이스 안티패턴][BusyDatabase]</span><span class="sxs-lookup"><span data-stu-id="63ec5-241">[Busy Database antipattern][BusyDatabase]</span></span>
- <span data-ttu-id="63ec5-242">[번잡한 I/O 안티패턴][chatty-io]</span><span class="sxs-lookup"><span data-stu-id="63ec5-242">[Chatty I/O antipattern][chatty-io]</span></span>
- <span data-ttu-id="63ec5-243">[데이터 분할 모범 사례][data-partitioning]</span><span class="sxs-lookup"><span data-stu-id="63ec5-243">[Data partitioning best practices][data-partitioning]</span></span>


[BusyDatabase]: ../busy-database/index.md
[chatty-io]: ../chatty-io.md
[data-partitioning]: ../../best-practices/data-partitioning.md
[new-relic]: https://newrelic.com/application-monitoring

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ExtraneousFetching

[chatty-io]: ../chatty-io/index.md
[MonolithicPersistence]: ../monolithic-persistence/index.md
[Load-Test-Results-Client-Side1]:_images/LoadTestResultsClientSide1.jpg
[Load-Test-Results-Client-Side2]:_images/LoadTestResultsClientSide2.jpg
[Load-Test-Results-Database-Side1]:_images/LoadTestResultsDatabaseSide1.jpg
[Load-Test-Results-Database-Side2]:_images/LoadTestResultsDatabaseSide2.jpg
[QueryPerformanceZoomed]: _images/QueryPerformanceZoomed.jpg
[QueryDetails]: _images/QueryDetails.jpg
[TelemetryAllFields]: _images/TelemetryAllFields.jpg
[TelemetryAggregateOnClient]: _images/TelemetryAggregateOnClient.jpg
[TelemetryRequiredFields]: _images/TelemetryRequiredFields.jpg
[TelemetryAggregateInDatabaseAsync]: _images/TelemetryAggregateInDatabase.jpg
