---
title: 번잡한 I/O 안티패턴
description: 다수의 I/O 요청으로 인해 성능과 응답성이 저하될 수 있습니다.
author: dragon119
ms.openlocfilehash: 4f0e0e455ceb58317d3029d8ab4631d476802499
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/23/2018
---
# <a name="chatty-io-antipattern"></a><span data-ttu-id="20d98-103">번잡한 I/O 안티패턴</span><span class="sxs-lookup"><span data-stu-id="20d98-103">Chatty I/O antipattern</span></span>

<span data-ttu-id="20d98-104">수많은 I/O 요청의 누적 효과는 성능 및 응답성에 상당한 영향을 미칠 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-104">The cumulative effect of a large number of I/O requests can have a significant impact on performance and responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="20d98-105">문제 설명</span><span class="sxs-lookup"><span data-stu-id="20d98-105">Problem description</span></span>

<span data-ttu-id="20d98-106">계산 작업에 비해 네트워크 호출 및 기타 I/O 작업은 본질적으로 느립니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-106">Network calls and other I/O operations are inherently slow compared to compute tasks.</span></span> <span data-ttu-id="20d98-107">일반적으로 각 I/O 요청에는 상당한 오버헤드가 있으며 수많은 I/O 작업의 누적 효과로 인해 시스템 속도가 느려질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-107">Each I/O request typically has significant overhead, and the cumulative effect of numerous I/O operations can slow down the system.</span></span> <span data-ttu-id="20d98-108">번잡한 I/O의 일반적인 원인은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-108">Here are some common causes of chatty I/O.</span></span>

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a><span data-ttu-id="20d98-109">개별 레코드를 별도의 요청으로 데이터베이스에 읽고 쓰기</span><span class="sxs-lookup"><span data-stu-id="20d98-109">Reading and writing individual records to a database as distinct requests</span></span>

<span data-ttu-id="20d98-110">다음 예제는 제품 데이터베이스에서 읽습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-110">The following example reads from a database of products.</span></span> <span data-ttu-id="20d98-111">여기에는 `Product`, `ProductSubcategory`, `ProductPriceListHistory`라는 3개의 테이블이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-111">There are three tables, `Product`, `ProductSubcategory`, and `ProductPriceListHistory`.</span></span> <span data-ttu-id="20d98-112">이 코드는 일련의 쿼리를 실행하여 가격 책정 정보와 함께 하위 카테고리의 모든 제품을 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-112">The code retrieves all of the products in a subcategory, along with the pricing information, by executing a series of queries:</span></span>  

1. <span data-ttu-id="20d98-113">`ProductSubcategory` 테이블에서 하위 카테고리를 쿼리합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-113">Query the subcategory from the `ProductSubcategory` table.</span></span>
2. <span data-ttu-id="20d98-114">`Product` 테이블을 쿼리하여 해당 하위 범주의 모든 제품을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-114">Find all products in that subcategory by querying the `Product` table.</span></span>
3. <span data-ttu-id="20d98-115">각 제품에 대해 `ProductPriceListHistory` 테이블에서 가격 책정 데이터를 쿼리합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-115">For each product, query the pricing data from the `ProductPriceListHistory` table.</span></span>

<span data-ttu-id="20d98-116">응용 프로그램은 [Entity Framework][ef]를 사용하여 데이터베이스를 쿼리합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-116">The application uses [Entity Framework][ef] to query the database.</span></span> <span data-ttu-id="20d98-117">전체 샘플은 [여기][code-sample]에서 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-117">You can find the complete sample [here][code-sample].</span></span> 

```csharp
public async Task<IHttpActionResult> GetProductsInSubCategoryAsync(int subcategoryId)
{
    using (var context = GetContext())
    {
        // Get product subcategory.
        var productSubcategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subcategoryId)
                .FirstOrDefaultAsync();

        // Find products in that category.
        productSubcategory.Product = await context.Products
            .Where(p => subcategoryId == p.ProductSubcategoryId)
            .ToListAsync();

        // Find price history for each product.
        foreach (var prod in productSubcategory.Product)
        {
            int productId = prod.ProductId;
            var productListPriceHistory = await context.ProductListPriceHistory
                .Where(pl => pl.ProductId == productId)
                .ToListAsync();
            prod.ProductListPriceHistory = productListPriceHistory;
        }
        return Ok(productSubcategory);
    }
}
```

<span data-ttu-id="20d98-118">이 예제는 명시적으로 문제를 보여주지만 자식 레코드를 암시적으로 한 번에 하나씩 가져오는 경우 O/RM이 문제를 가릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-118">This example shows the problem explicitly, but sometimes an O/RM can mask the problem, if it implicitly fetches child records one at a time.</span></span> <span data-ttu-id="20d98-119">이것을 "N + 1 문제"라고 합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-119">This is known as the "N+1 problem".</span></span> 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a><span data-ttu-id="20d98-120">단일 논리 연산을 일련의 HTTP 요청으로 구현</span><span class="sxs-lookup"><span data-stu-id="20d98-120">Implementing a single logical operation as a series of HTTP requests</span></span>

<span data-ttu-id="20d98-121">개발자가 개체 지향 패러다임을 따르고 원격 개체를 메모리의 로컬 개체로 처리하려는 경우에 자주 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-121">This often happens when developers try to follow an object-oriented paradigm, and treat remote objects as if they were local objects in memory.</span></span> <span data-ttu-id="20d98-122">이로 인해 네트워크 왕복이 너무 많아질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-122">This can result in too many network round trips.</span></span> <span data-ttu-id="20d98-123">예를 들어 다음 웹 API는 개별 HTTP GET 메서드를 통해 `User` 개체의 개별 속성을 노출합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-123">For example, the following web API exposes the individual properties of `User` objects through individual HTTP GET methods.</span></span> 

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}/username")]
    public HttpResponseMessage GetUserName(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/gender")]
    public HttpResponseMessage GetGender(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/dateofbirth")]
    public HttpResponseMessage GetDateOfBirth(int id)
    {
        ...
    }
}
```

<span data-ttu-id="20d98-124">이런 방식은 기술적으로 문제가 없지만 대부분의 클라이언트가 각 `User`에 대해 여러 속성을 가져와야 하기 때문에 클라이언트 코드가 다음과 같이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-124">While there's nothing technically wrong with this approach, most clients will probably need to get several properties for each `User`, resulting in client code like the following.</span></span> 

```csharp
HttpResponseMessage response = await client.GetAsync("users/1/username");
response.EnsureSuccessStatusCode();
var userName = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/gender");
response.EnsureSuccessStatusCode();
var gender = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/dateofbirth");
response.EnsureSuccessStatusCode();
var dob = await response.Content.ReadAsStringAsync();
```

### <a name="reading-and-writing-to-a-file-on-disk"></a><span data-ttu-id="20d98-125">디스크의 파일 읽기 및 쓰기</span><span class="sxs-lookup"><span data-stu-id="20d98-125">Reading and writing to a file on disk</span></span>

<span data-ttu-id="20d98-126">파일 I/O는 데이터를 읽거나 쓰기 전에 파일을 열고 적절한 지점으로 이동하는 단계가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-126">File I/O involves opening a file and moving to the appropriate point before reading or writing data.</span></span> <span data-ttu-id="20d98-127">작업이 완료되면 운영 체제 리소스를 절약하기 위해 파일을 닫을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-127">When the operation is complete, the file might be closed to save operating system resources.</span></span> <span data-ttu-id="20d98-128">파일에 적은 소량의 정보를 지속적으로 읽고 쓰는 응용 프로그램은 상당한 I/O 오버헤드를 발생시킵니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-128">An application that continually reads and writes small amounts of information to a file will generate significant I/O overhead.</span></span> <span data-ttu-id="20d98-129">작은 쓰기 요청은 파일 조각화로 이어져서 후속 I/O 작업이 더 느려질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-129">Small write requests can also lead to file fragmentation, slowing subsequent I/O operations still further.</span></span> 

<span data-ttu-id="20d98-130">다음 예제는 `FileStream`을 사용하여 `Customer` 개체를 파일에 씁니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-130">The following example uses a `FileStream` to write a `Customer` object to a file.</span></span> <span data-ttu-id="20d98-131">`FileStream`을 만들면 파일이 열리고 삭제하면 파일이 닫힙니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-131">Creating the `FileStream` opens the file, and disposing it closes the file.</span></span> <span data-ttu-id="20d98-132">(`using` 문은 `FileStream` 개체를 자동으로 삭제합니다.) 새 고객이 추가될 때 응용 프로그램에서 이 메서드를 반복적으로 호출하면 I/O 오버헤드가 빠르게 누적될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-132">(The `using` statement automatically disposes the `FileStream` object.) If the application calls this method repeatedly as new customers are added, the I/O overhead can accumulate quickly.</span></span>

```csharp
private async Task SaveCustomerToFileAsync(Customer cust)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        byte [] data = null;
        using (MemoryStream memStream = new MemoryStream())
        {
            formatter.Serialize(memStream, cust);
            data = memStream.ToArray();
        }
        await fileStream.WriteAsync(data, 0, data.Length);
    }
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="20d98-133">문제를 해결하는 방법</span><span class="sxs-lookup"><span data-stu-id="20d98-133">How to fix the problem</span></span>

<span data-ttu-id="20d98-134">데이터를 더 크고 적은 수의 요청으로 패키징하여 I/O 요청 수를 줄입니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-134">Reduce the number of I/O requests by packaging the data into larger, fewer requests.</span></span>

<span data-ttu-id="20d98-135">데이터베이스에서 데이터를 여러 개의 작은 쿼리 대신 단일 쿼리로 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-135">Fetch data from a database as a single query, instead of several smaller queries.</span></span> <span data-ttu-id="20d98-136">다음은 제품 정보를 검색하는 수정된 코드 버전입니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-136">Here's a revised version of the code that retrieves product information.</span></span>

```csharp
public async Task<IHttpActionResult> GetProductCategoryDetailsAsync(int subCategoryId)
{
    using (var context = GetContext())
    {
        var subCategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subCategoryId)
                .Include("Product.ProductListPriceHistory")
                .FirstOrDefaultAsync();

        if (subCategory == null)
            return NotFound();

        return Ok(subCategory);
    }
}
```

<span data-ttu-id="20d98-137">웹 API에 대한 REST 디자인 원칙을 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-137">Follow REST design principles for web APIs.</span></span> <span data-ttu-id="20d98-138">다음은 이전 예제에서 수정된 웹 API 버전입니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-138">Here's a revised version of the web API from the earlier example.</span></span> <span data-ttu-id="20d98-139">각 속성에 대한 별도의 GET 메서드 대신 `User`를 반환하는 단일 GET 메서드가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-139">Instead of separate GET methods for each property, there is a single GET method that returns the `User`.</span></span> <span data-ttu-id="20d98-140">결과적으로 요청당 응답 본문이 커지지만 각 클라이언트는 API 호출을 적게 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-140">This results in a larger response body per request, but each client is likely to make fewer API calls.</span></span>

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}")]
    public HttpResponseMessage GetUser(int id)
    {
        ...
    }
}

// Client code
HttpResponseMessage response = await client.GetAsync("users/1");
response.EnsureSuccessStatusCode();
var user = await response.Content.ReadAsStringAsync();
```

<span data-ttu-id="20d98-141">파일 I/O의 경우 데이터를 메모리에 버퍼링한 다음 버퍼링된 데이터를 단일 작업으로 파일에 기록하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-141">For file I/O, consider buffering data in memory and then writing the buffered data to a file as a single operation.</span></span> <span data-ttu-id="20d98-142">이 방식은 파일을 반복적으로 열고 닫아서 생기는 오버헤드를 줄이고 디스크의 파일 조각화를 줄이는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-142">This approach reduces the overhead from repeatedly opening and closing the file, and helps to reduce fragmentation of the file on disk.</span></span>

```csharp
// Save a list of customer objects to a file
private async Task SaveCustomerListToFileAsync(List<Customer> customers)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        foreach (var cust in customers)
        {
            byte[] data = null;
            using (MemoryStream memStream = new MemoryStream())
            {
                formatter.Serialize(memStream, cust);
                data = memStream.ToArray();
            }
            await fileStream.WriteAsync(data, 0, data.Length);
        }
    }
}

// In-memory buffer for customers.
List<Customer> customers = new List<Customers>();

// Create a new customer and add it to the buffer
var cust = new Customer(...);
customers.Add(cust);

// Add more customers to the list as they are created
...

// Save the contents of the list, writing all customers in a single operation
await SaveCustomerListToFileAsync(customers);
```

## <a name="considerations"></a><span data-ttu-id="20d98-143">고려 사항</span><span class="sxs-lookup"><span data-stu-id="20d98-143">Considerations</span></span>

- <span data-ttu-id="20d98-144">처음 두 예제는 I/O 호출을 *줄이*지만, 각각 더 *많은* 정보를 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-144">The first two examples make *fewer* I/O calls, but each one retrieves *more* information.</span></span> <span data-ttu-id="20d98-145">이 두 요소 사이의 상충 관계를 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-145">You must consider the tradeoff between these two factors.</span></span> <span data-ttu-id="20d98-146">정답은 실제 사용 패턴에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-146">The right answer will depend on the actual usage patterns.</span></span> <span data-ttu-id="20d98-147">예를 들어 웹 API 예제에서 클라이언트에 사용자 이름만 자주 필요한 경우가 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-147">For example, in the web API example, it might turn out that clients often need just the user name.</span></span> <span data-ttu-id="20d98-148">이 경우 해당 데이터를 별도의 API 호출로 노출하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-148">In that case, it might make sense to expose it as a separate API call.</span></span> <span data-ttu-id="20d98-149">자세한 내용은 [불필요한 가져오기][extraneous-fetching] 안티패턴을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="20d98-149">For more information, see the [Extraneous Fetching][extraneous-fetching] antipattern.</span></span>

- <span data-ttu-id="20d98-150">데이터를 읽을 때 I/O 요청을 너무 크게 만들지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="20d98-150">When reading data, do not make your I/O requests too large.</span></span> <span data-ttu-id="20d98-151">응용 프로그램이 사용할만한 정보만 검색해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-151">An application should only retrieve the information that it is likely to use.</span></span> 

- <span data-ttu-id="20d98-152">대부분의 요청을 처리하는 *자주 액세스하는 데이터*와 거의 사용되지 않는 *드물게 액세스하는 데이터*라는 두 가지 청크로 개체의 정보를 분할하는 것이 도움이 되기도 합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-152">Sometimes it helps to partition the information for an object into two chunks, *frequently accessed data* that accounts for most requests, and *less frequently accessed data* that is used rarely.</span></span> <span data-ttu-id="20d98-153">가장 자주 액세스하는 데이터가 개체의 전체 데이터에서 비교적 작은 부분을 차지하는 경우가 많기 때문에 해당 부분만 반환하면 상당한 I/O 오버헤드를 절약할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-153">Often the most frequently accessed data is a relatively small portion of the total data for an object, so returning just that portion can save significant I/O overhead.</span></span>

- <span data-ttu-id="20d98-154">오래 걸리는 작업을 수행하는 동안 경합 가능성을 줄이기 위해, 데이터를 쓸 때 리소스를 필요 이상으로 오래 잠그지 않도록 하십시오.</span><span class="sxs-lookup"><span data-stu-id="20d98-154">When writing data, avoid locking resources for longer than necessary, to reduce the chances of contention during a lengthy operation.</span></span> <span data-ttu-id="20d98-155">쓰기 작업이 다수의 데이터 저장소, 파일 또는 서비스에 걸쳐있는 경우 최종적으로 일관된 방식을 채택합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-155">If a write operation spans multiple data stores, files, or services, then adopt an eventually consistent approach.</span></span> <span data-ttu-id="20d98-156">[데이터 일관성 지침][data-consistency-guidance]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="20d98-156">See [Data Consistency guidance][data-consistency-guidance].</span></span>

- <span data-ttu-id="20d98-157">데이터를 쓰기 전에 메모리에 버퍼링하면 프로세스가 충돌하는 경우 데이터가 손상될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-157">If you buffer data in memory before writing it, the data is vulnerable if the process crashes.</span></span> <span data-ttu-id="20d98-158">데이터 속도가 일반적으로 급격히 증가하거나 상대적으로 약한 경우 [Event Hubs](http://azure.microsoft.com/services/event-hubs/)와 같은 내구성이 있는 외부 큐에 데이터를 버퍼링하는 것이 더 안전할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-158">If the data rate typically has bursts or is relatively sparse, it may be safer to buffer the data in an external durable queue such as [Event Hubs](http://azure.microsoft.com/services/event-hubs/).</span></span>

- <span data-ttu-id="20d98-159">서비스 또는 데이터베이스에서 검색하는 데이터를 캐시하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-159">Consider caching data that you retrieve from a service or a database.</span></span> <span data-ttu-id="20d98-160">이렇게 하면 동일한 데이터에 반복되는 요청을 피하여 I/O 볼륨을 줄이는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-160">This can help to reduce the volume of I/O by avoiding repeated requests for the same data.</span></span> <span data-ttu-id="20d98-161">자세한 내용은 [캐싱 모범 사례][caching-guidance]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="20d98-161">For more information, see [Caching best practices][caching-guidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="20d98-162">문제를 감지하는 방법</span><span class="sxs-lookup"><span data-stu-id="20d98-162">How to detect the problem</span></span>

<span data-ttu-id="20d98-163">번잡한 I/O의 증상에는 높은 대기 시간과 낮은 처리량이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-163">Symptoms of chatty I/O include high latency and low throughput.</span></span> <span data-ttu-id="20d98-164">I/O 리소스에 대한 경합 증가로 인해 응답 시간이 길어지거나 서비스 시간 초과하여 장애가 발생하면 최종 사용자가 보고할 가능성이 높습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-164">End users are likely to report extended response times or failures caused by services timing out, due to increased contention for I/O resources.</span></span>

<span data-ttu-id="20d98-165">다음 단계를 수행하면 문제의 원인을 파악하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-165">You can perform the following steps to help identify the causes of any problems:</span></span>

1. <span data-ttu-id="20d98-166">프로덕션 시스템의 프로세스 모니터링을 수행하여 응답 시간이 느린 작업을 식별합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-166">Perform process monitoring of the production system to identify operations with poor response times.</span></span>
2. <span data-ttu-id="20d98-167">이전 단계에서 확인된 각 작업의 부하 테스트를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-167">Perform load testing of each operation identified in the previous step.</span></span>
3. <span data-ttu-id="20d98-168">부하 테스트 중 각 작업에 의해 생성된 데이터 액세스 요청에 대한 원격 분석 데이터를 수집합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-168">During the load tests, gather telemetry data about the data access requests made by each operation.</span></span>
4. <span data-ttu-id="20d98-169">데이터 저장소에 전송된 각 요청에 대한 자세한 통계를 수집합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-169">Gather detailed statistics for each request sent to a data store.</span></span>
5. <span data-ttu-id="20d98-170">테스트 환경에서 응용 프로그램을 프로파일링하여 잠재적으로 I/O 병목 현상이 발생할만한 위치를 밝힙니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-170">Profile the application in the test environment to establish where possible I/O bottlenecks might be occurring.</span></span> 

<span data-ttu-id="20d98-171">다음과 같은 증상을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-171">Look for any of these symptoms:</span></span>

- <span data-ttu-id="20d98-172">동일 파일에 대한 다수의 작은 I/O 요청.</span><span class="sxs-lookup"><span data-stu-id="20d98-172">A large number of small I/O requests made to the same file.</span></span>
- <span data-ttu-id="20d98-173">응용 프로그램 인스턴스가 동일한 서비스에 대해 수행하는 다수의 작은 네트워크 요청.</span><span class="sxs-lookup"><span data-stu-id="20d98-173">A large number of small network requests made by an application instance to the same service.</span></span>
- <span data-ttu-id="20d98-174">응용 프로그램 인스턴스가 동일한 데이터 저장소에 대해 수행하는 다수의 작은 요청.</span><span class="sxs-lookup"><span data-stu-id="20d98-174">A large number of small requests made by an application instance to the same data store.</span></span>
- <span data-ttu-id="20d98-175">I/O 바인딩이 되는 응용 프로그램 및 서비스.</span><span class="sxs-lookup"><span data-stu-id="20d98-175">Applications and services becoming I/O bound.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="20d98-176">예제 진단</span><span class="sxs-lookup"><span data-stu-id="20d98-176">Example diagnosis</span></span>

<span data-ttu-id="20d98-177">다음 섹션에서는 이러한 단계를 데이터베이스를 쿼리하는 앞에서 설명한 예제에 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-177">The following sections apply these steps to the example shown earlier that queries a database.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="20d98-178">응용 프로그램 부하 테스트</span><span class="sxs-lookup"><span data-stu-id="20d98-178">Load test the application</span></span>

<span data-ttu-id="20d98-179">이 그래프는 부하 테스트의 결과입니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-179">This graph shows the results of load testing.</span></span> <span data-ttu-id="20d98-180">중간 응답 시간은 요청당 10초 단위로 측정됩니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-180">Median response time is measured in 10s of seconds per request.</span></span> <span data-ttu-id="20d98-181">그래프는 매우 높은 대기 시간을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-181">The graph shows very high latency.</span></span> <span data-ttu-id="20d98-182">사용자가 1000명 로드되면 쿼리 결과를 보기 위해 거의 1분을 기다려야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-182">With a load of 1000 users, a user might have to wait for nearly a minute to see the results of a query.</span></span> 

![번잡한 I/O 응용 프로그램 예제에 대한 핵심 지표 부하 테스트 결과][key-indicators-chatty-io]

> [!NOTE]
> <span data-ttu-id="20d98-184">이 응용 프로그램은 Azure SQL Database를 사용하여 Azure App Service 웹앱으로 배포되었습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-184">The application was deployed as an Azure App Service web app, using Azure SQL Database.</span></span> <span data-ttu-id="20d98-185">부하 테스트에는 최대 1000명의 동시 사용자가 시뮬레이션된 단계 워크로드를 사용했습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-185">The load test used a simulated step workload of up to 1000 concurrent users.</span></span> <span data-ttu-id="20d98-186">데이터베이스는 연결에 대한 경합이 결과에 영향을 줄 가능성을 줄이기 위해 최대 1000개의 동시 연결을 지원하는 연결 풀로 구성되었습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-186">The database was configured with a connection pool supporting up to 1000 concurrent connections, to reduce the chance that contention for connections would affect the results.</span></span> 

### <a name="monitor-the-application"></a><span data-ttu-id="20d98-187">응용 프로그램 모니터링</span><span class="sxs-lookup"><span data-stu-id="20d98-187">Monitor the application</span></span>

<span data-ttu-id="20d98-188">APM(응용 프로그램 성능 모니터링) 패키지를 사용하여 번잡한 I/O를 식별할 수 있는 주요 메트릭을 캡처하고 분석할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-188">You can use an application performance monitoring (APM) package to capture and analyze the key metrics that might identify chatty I/O.</span></span> <span data-ttu-id="20d98-189">어떤 메트릭이 중요한가는 I/O 워크로드에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-189">Which metrics are important will depend on the I/O workload.</span></span> <span data-ttu-id="20d98-190">이 예제의 경우 흥미로운 I/O 요청은 데이터베이스 쿼리입니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-190">For this example, the interesting I/O requests were the database queries.</span></span> 

<span data-ttu-id="20d98-191">다음 이미지는 [New Relic APM][new-relic]을 사용하여 생성된 결과입니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-191">The following image shows results generated using [New Relic APM][new-relic].</span></span> <span data-ttu-id="20d98-192">최대 워크로드 기간 동안 평균 데이터베이스 응답 시간은 요청당 약 5.6초에 최고점에 도달했습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-192">The average database response time peaked at approximately 5.6 seconds per request during the maximum workload.</span></span> <span data-ttu-id="20d98-193">테스트 기간 동안 시스템은 분당 평균 410건의 요청을 지원할 수 있었습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-193">The system was able to support an average of 410 requests per minute throughout the test.</span></span>

![AdventureWorks2012 데이터베이스를 방문하는 트래픽에 대한 개요][databasetraffic]

### <a name="gather-detailed-data-access-information"></a><span data-ttu-id="20d98-195">상세 데이터 액세스 정보 수집</span><span class="sxs-lookup"><span data-stu-id="20d98-195">Gather detailed data access information</span></span>

<span data-ttu-id="20d98-196">모니터링 데이터베이스를 자세히 살펴보면 응용 프로그램에서 3가지 다른 SQL SELECT 문을 실행하는 것이 보입니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-196">Digging deeper into the monitoring data shows the application executes three different SQL SELECT statements.</span></span> <span data-ttu-id="20d98-197">이것은 `ProductListPriceHistory`, `Product` 및 `ProductSubcategory` 테이블에서 데이터를 가져오기 위해 Entity Framework에서 생성된 요청에 해당합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-197">These correspond to the requests generated by Entity Framework to fetch data from the `ProductListPriceHistory`, `Product`, and `ProductSubcategory` tables.</span></span>
<span data-ttu-id="20d98-198">또한 `ProductListPriceHistory` 테이블에서 데이터를 검색하는 쿼리는 규모 순으로 가장 자주 실행되는 SELECT 문입니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-198">Furthermore, the query that retrieves data from the `ProductListPriceHistory` table is by far the most frequently executed SELECT statement, by an order of magnitude.</span></span>

![테스트 중인 응용 프로그램 예제로 수행한 쿼리][queries]

<span data-ttu-id="20d98-200">앞서 살펴본 `GetProductsInSubCategoryAsync` 메서드는 45개의 SELECT 쿼리를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-200">It turns out that the `GetProductsInSubCategoryAsync` method, shown earlier, performs 45 SELECT queries.</span></span> <span data-ttu-id="20d98-201">각 쿼리는 응용 프로그램이 새 SQL 연결을 열도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-201">Each query causes the application to open a new SQL connection.</span></span>

![테스트 중인 응용 프로그램 예제에 대한 쿼리 통계][queries2]

> [!NOTE]
> <span data-ttu-id="20d98-203">이 이미지는 부하 테스트에서 `GetProductsInSubCategoryAsync` 작업의 가장 느린 인스턴스에 대한 추적 정보를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-203">This image shows trace information for the slowest instance of the `GetProductsInSubCategoryAsync` operation in the load test.</span></span> <span data-ttu-id="20d98-204">프로덕션 환경에서는 가장 느린 인스턴스의 흔적을 조사하여 문제를 암시하는 패턴이 있는지 확인하는 것이 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-204">In a production environment, it's useful to examine traces of the slowest instances, to see if there is a pattern that suggests a problem.</span></span> <span data-ttu-id="20d98-205">평균값만 살펴보면 로드가 심하게 악화되는 문제를 간과할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-205">If you just look at the average values, you might overlook problems that will get dramatically worse under load.</span></span>

<span data-ttu-id="20d98-206">다음 이미지는 실제 발급된 SQL 문을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-206">The next image shows the actual SQL statements that were issued.</span></span> <span data-ttu-id="20d98-207">가격 정보를 가져 오는 쿼리는 제품 하위 범주의 개별 제품에 대해 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-207">The query that fetches price information is run for each individual product in the product subcategory.</span></span> <span data-ttu-id="20d98-208">조인을 사용하면 데이터베이스 호출 수를 상당히 줄어 듭니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-208">Using a join would considerably reduce the number of database calls.</span></span>

![테스트 중인 응용 프로그램 예제에 대한 쿼리 세부 정보][queries3]

<span data-ttu-id="20d98-210">Entity Framework와 같은 O/RM을 사용하는 경우, SQL 쿼리를 추적하면 O/RM이 프로그래밍 방식 호출을 SQL 문으로 변환하는 방식에 대한 통찰력을 얻을 수 있고 데이터 액세스가 최적화될 수 있는 영역을 알아낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-210">If you are using an O/RM, such as Entity Framework, tracing the SQL queries can provide insight into how the O/RM translates programmatic calls into SQL statements, and indicate areas where data access might be optimized.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="20d98-211">솔루션 구현 및 결과 확인</span><span class="sxs-lookup"><span data-stu-id="20d98-211">Implement the solution and verify the result</span></span>

<span data-ttu-id="20d98-212">호출을 Entity Framework에 다시 쓰면 다음과 같은 결과가 나타납니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-212">Rewriting the call to Entity Framework produced the following results.</span></span>

![번잡한 I/O 응용 프로그램 예제의 대규모 API에 대한 핵심 지표 부하 테스트 결과][key-indicators-chunky-io]

<span data-ttu-id="20d98-214">이 부하 테스트는 동일한 부하 프로필을 사용하여 동일한 배포에서 수행되었습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-214">This load test was performed on the same deployment, using the same load profile.</span></span> <span data-ttu-id="20d98-215">이번에는 그래프에 표시된 대기 시간이 훨씬 낮습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-215">This time the graph shows much lower latency.</span></span> <span data-ttu-id="20d98-216">1000명의 사용자에 대한 평균 요청 시간은 5~6초 사이이며, 거의 1분에서 줄어 들었습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-216">The average request time at 1000 users is between 5 and 6 seconds, down from nearly a minute.</span></span>

<span data-ttu-id="20d98-217">이번에는 시스템이 분당 평균 3,970건(이전 테스트의 410건 대비)의 요청을 지원했습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-217">This time the system supported an average of 3,970 requests per minute, compared to 410 for the earlier test.</span></span>

![대규모 API의 트랜잭션 개요][databasetraffic2]

<span data-ttu-id="20d98-219">SQL 문을 추적하면 모든 데이터를 단일 SELECT 문으로 가져온다는 것을 알 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-219">Tracing the SQL statement shows that all the data is fetched in a single SELECT statement.</span></span> <span data-ttu-id="20d98-220">이 쿼리는 상당히 복잡하지만 작업당 한 번만 수행됩니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-220">Although this query is considerably more complex, it is performed only once per operation.</span></span> <span data-ttu-id="20d98-221">복잡한 조인은 비용이 높을 수 있지만 관계형 데이터베이스 시스템은 이러한 유형의 쿼리에 최적화되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="20d98-221">And while complex joins can become expensive, relational database systems are optimized for this type of query.</span></span>  

![대규모 API에 대한 쿼리 세부 정보][queries4]

## <a name="related-resources"></a><span data-ttu-id="20d98-223">관련 리소스</span><span class="sxs-lookup"><span data-stu-id="20d98-223">Related resources</span></span>

- <span data-ttu-id="20d98-224">[API 디자인 모범 사례][api-design]</span><span class="sxs-lookup"><span data-stu-id="20d98-224">[API Design best practices][api-design]</span></span>
- <span data-ttu-id="20d98-225">[캐싱 모범 사례][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="20d98-225">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="20d98-226">[데이터 일관성 입문][data-consistency-guidance]</span><span class="sxs-lookup"><span data-stu-id="20d98-226">[Data Consistency Primer][data-consistency-guidance]</span></span>
- <span data-ttu-id="20d98-227">[불필요한 가져오기 안티패턴][extraneous-fetching]</span><span class="sxs-lookup"><span data-stu-id="20d98-227">[Extraneous Fetching antipattern][extraneous-fetching]</span></span>
- <span data-ttu-id="20d98-228">[캐싱 없음 안티패턴][no-cache]</span><span class="sxs-lookup"><span data-stu-id="20d98-228">[No Caching antipattern][no-cache]</span></span>

[api-design]: ../../best-practices/api-design.md
[caching-guidance]: ../../best-practices/caching.md
[code-sample]:  https://github.com/mspnp/performance-optimization/tree/master/ChattyIO
[data-consistency-guidance]: http://https://msdn.microsoft.com/library/dn589800.aspx
[ef]: /ef/
[extraneous-fetching]: ../extraneous-fetching/index.md
[new-relic]: https://newrelic.com/application-monitoring
[no-cache]: ../no-caching/index.md

[key-indicators-chatty-io]: _images/ChattyIO.jpg
[key-indicators-chunky-io]: _images/ChunkyIO.jpg
[databasetraffic]: _images/DatabaseTraffic.jpg
[databasetraffic2]: _images/DatabaseTraffic2.jpg
[queries]: _images/DatabaseQueries.jpg
[queries2]: _images/DatabaseQueries2.jpg
[queries3]: _images/DatabaseQueries3.jpg
[queries4]: _images/DatabaseQueries4.jpg

