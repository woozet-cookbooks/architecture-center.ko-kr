---
title: "번잡한 I/O 안티패턴"
description: "다수의 I/O 요청으로 인해 성능과 응답성이 저하될 수 있습니다."
author: dragon119
ms.openlocfilehash: 50001316939b56c9b57a119f6ae20f0878f54c0f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="chatty-io-antipattern"></a>번잡한 I/O 안티패턴

수많은 I/O 요청의 누적 효과는 성능 및 응답성에 상당한 영향을 미칠 수 있습니다.

## <a name="problem-description"></a>문제 설명

계산 작업에 비해 네트워크 호출 및 기타 I/O 작업은 본질적으로 느립니다. 일반적으로 각 I/O 요청에는 상당한 오버헤드가 있으며 수많은 I/O 작업의 누적 효과로 인해 시스템 속도가 느려질 수 있습니다. 번잡한 I/O의 일반적인 원인은 다음과 같습니다.

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a>개별 레코드를 별도의 요청으로 데이터베이스에 읽고 쓰기

다음 예제는 제품 데이터베이스에서 읽습니다. 여기에는 `Product`, `ProductSubcategory`, `ProductPriceListHistory`라는 3개의 테이블이 있습니다. 이 코드는 일련의 쿼리를 실행하여 가격 책정 정보와 함께 하위 카테고리의 모든 제품을 검색합니다.  

1. `ProductSubcategory` 테이블에서 하위 카테고리를 쿼리합니다.
2. `Product` 테이블을 쿼리하여 해당 하위 범주의 모든 제품을 찾습니다.
3. 각 제품에 대해 `ProductPriceListHistory` 테이블에서 가격 책정 데이터를 쿼리합니다.

응용 프로그램은 [Entity Framework][ef]를 사용하여 데이터베이스를 쿼리합니다. 전체 샘플은 [여기][code-sample]에서 찾을 수 있습니다. 

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

이 예제는 명시적으로 문제를 보여주지만 자식 레코드를 암시적으로 한 번에 하나씩 가져오는 경우 O/RM이 문제를 가릴 수 있습니다. 이것을 "N + 1 문제"라고 합니다. 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a>단일 논리 연산을 일련의 HTTP 요청으로 구현

개발자가 개체 지향 패러다임을 따르고 원격 개체를 메모리의 로컬 개체로 처리하려는 경우에 자주 발생합니다. 이로 인해 네트워크 왕복이 너무 많아질 수 있습니다. 예를 들어 다음 웹 API는 개별 HTTP GET 메서드를 통해 `User` 개체의 개별 속성을 노출합니다. 

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

이런 방식은 기술적으로 문제가 없지만 대부분의 클라이언트가 각 `User`에 대해 여러 속성을 가져와야 하기 때문에 클라이언트 코드가 다음과 같이 됩니다. 

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

### <a name="reading-and-writing-to-a-file-on-disk"></a>디스크의 파일 읽기 및 쓰기

파일 I/O는 데이터를 읽거나 쓰기 전에 파일을 열고 적절한 지점으로 이동하는 단계가 포함됩니다. 작업이 완료되면 운영 체제 리소스를 절약하기 위해 파일을 닫을 수 있습니다. 파일에 적은 소량의 정보를 지속적으로 읽고 쓰는 응용 프로그램은 상당한 I/O 오버헤드를 발생시킵니다. 작은 쓰기 요청은 파일 조각화로 이어져서 후속 I/O 작업이 더 느려질 수 있습니다. 

다음 예제는 `FileStream`을 사용하여 `Customer` 개체를 파일에 씁니다. `FileStream`을 만들면 파일이 열리고 삭제하면 파일이 닫힙니다. (`using` 문은 `FileStream` 개체를 자동으로 삭제합니다.) 새 고객이 추가될 때 응용 프로그램에서 이 메서드를 반복적으로 호출하면 I/O 오버헤드가 빠르게 누적될 수 있습니다.

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

## <a name="how-to-fix-the-problem"></a>문제를 해결하는 방법

데이터를 더 크고 적은 수의 요청으로 패키징하여 I/O 요청 수를 줄입니다.

데이터베이스에서 데이터를 여러 개의 작은 쿼리 대신 단일 쿼리로 가져옵니다. 다음은 제품 정보를 검색하는 수정된 코드 버전입니다.

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

웹 API에 대한 REST 디자인 원칙을 따릅니다. 다음은 이전 예제에서 수정된 웹 API 버전입니다. 각 속성에 대한 별도의 GET 메서드 대신 `User`를 반환하는 단일 GET 메서드가 있습니다. 결과적으로 요청당 응답 본문이 커지지만 각 클라이언트는 API 호출을 적게 생성합니다.

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

파일 I/O의 경우 데이터를 메모리에 버퍼링한 다음 버퍼링된 데이터를 단일 작업으로 파일에 기록하는 것이 좋습니다. 이 방식은 파일을 반복적으로 열고 닫아서 생기는 오버헤드를 줄이고 디스크의 파일 조각화를 줄이는 데 도움이 됩니다.

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

## <a name="considerations"></a>고려 사항

- 처음 두 예제는 I/O 호출을 *줄이*지만, 각각 더 *많은* 정보를 검색합니다. 이 두 요소 사이의 상충 관계를 고려해야 합니다. 정답은 실제 사용 패턴에 따라 달라집니다. 예를 들어 웹 API 예제에서 클라이언트에 사용자 이름만 자주 필요한 경우가 있을 수 있습니다. 이 경우 해당 데이터를 별도의 API 호출로 노출하는 것이 좋습니다. 자세한 내용은 [불필요한 가져오기][extraneous-fetching] 안티패턴을 참조하세요.

- 데이터를 읽을 때 I/O 요청을 너무 크게 만들지 마십시오. 응용 프로그램이 사용할만한 정보만 검색해야 합니다. 

- 대부분의 요청을 처리하는 *자주 액세스하는 데이터*와 거의 사용되지 않는 *드물게 액세스하는 데이터*라는 두 가지 청크로 개체의 정보를 분할하는 것이 도움이 되기도 합니다. 가장 자주 액세스하는 데이터가 개체의 전체 데이터에서 비교적 작은 부분을 차지하는 경우가 많기 때문에 해당 부분만 반환하면 상당한 I/O 오버헤드를 절약할 수 있습니다.

- 오래 걸리는 작업을 수행하는 동안 경합 가능성을 줄이기 위해, 데이터를 쓸 때 리소스를 필요 이상으로 오래 잠그지 않도록 하십시오. 쓰기 작업이 다수의 데이터 저장소, 파일 또는 서비스에 걸쳐있는 경우 최종적으로 일관된 방식을 채택합니다. [데이터 일관성 지침][data-consistency-guidance]을 참조하세요.

- 데이터를 쓰기 전에 메모리에 버퍼링하면 프로세스가 충돌하는 경우 데이터가 손상될 수 있습니다. 데이터 속도가 일반적으로 급격히 증가하거나 상대적으로 약한 경우 [Event Hubs](http://azure.microsoft.com/en-us/services/event-hubs/)와 같은 내구성이 있는 외부 큐에 데이터를 버퍼링하는 것이 더 안전할 수 있습니다.

- 서비스 또는 데이터베이스에서 검색하는 데이터를 캐시하는 것이 좋습니다. 이렇게 하면 동일한 데이터에 반복되는 요청을 피하여 I/O 볼륨을 줄이는 데 도움이 됩니다. 자세한 내용은 [캐싱 모범 사례][caching-guidance]를 참조하세요.

## <a name="how-to-detect-the-problem"></a>문제를 감지하는 방법

번잡한 I/O의 증상에는 높은 대기 시간과 낮은 처리량이 있습니다. I/O 리소스에 대한 경합 증가로 인해 응답 시간이 길어지거나 서비스 시간 초과하여 장애가 발생하면 최종 사용자가 보고할 가능성이 높습니다.

다음 단계를 수행하면 문제의 원인을 파악하는 데 도움이 될 수 있습니다.

1. 프로덕션 시스템의 프로세스 모니터링을 수행하여 응답 시간이 느린 작업을 식별합니다.
2. 이전 단계에서 확인된 각 작업의 부하 테스트를 수행합니다.
3. 부하 테스트 중 각 작업에 의해 생성된 데이터 액세스 요청에 대한 원격 분석 데이터를 수집합니다.
4. 데이터 저장소에 전송된 각 요청에 대한 자세한 통계를 수집합니다.
5. 테스트 환경에서 응용 프로그램을 프로파일링하여 잠재적으로 I/O 병목 현상이 발생할만한 위치를 밝힙니다. 

다음과 같은 증상을 찾습니다.

- 동일 파일에 대한 다수의 작은 I/O 요청.
- 응용 프로그램 인스턴스가 동일한 서비스에 대해 수행하는 다수의 작은 네트워크 요청.
- 응용 프로그램 인스턴스가 동일한 데이터 저장소에 대해 수행하는 다수의 작은 요청.
- I/O 바인딩이 되는 응용 프로그램 및 서비스.

## <a name="example-diagnosis"></a>예제 진단

다음 섹션에서는 이러한 단계를 데이터베이스를 쿼리하는 앞에서 설명한 예제에 적용합니다.

### <a name="load-test-the-application"></a>응용 프로그램 부하 테스트

이 그래프는 부하 테스트의 결과입니다. 중간 응답 시간은 요청당 10초 단위로 측정됩니다. 그래프는 매우 높은 대기 시간을 보여줍니다. 사용자가 1000명 로드되면 쿼리 결과를 보기 위해 거의 1분을 기다려야 할 수 있습니다. 

![번잡한 I/O 응용 프로그램 예제에 대한 핵심 지표 부하 테스트 결과][key-indicators-chatty-io]

> [!NOTE]
> 이 응용 프로그램은 Azure SQL Database를 사용하여 Azure App Service 웹앱으로 배포되었습니다. 부하 테스트에는 최대 1000명의 동시 사용자가 시뮬레이션된 단계 워크로드를 사용했습니다. 데이터베이스는 연결에 대한 경합이 결과에 영향을 줄 가능성을 줄이기 위해 최대 1000개의 동시 연결을 지원하는 연결 풀로 구성되었습니다. 

### <a name="monitor-the-application"></a>응용 프로그램 모니터링

APM(응용 프로그램 성능 모니터링) 패키지를 사용하여 번잡한 I/O를 식별할 수 있는 주요 메트릭을 캡처하고 분석할 수 있습니다. 어떤 메트릭이 중요한가는 I/O 워크로드에 따라 달라집니다. 이 예제의 경우 흥미로운 I/O 요청은 데이터베이스 쿼리입니다. 

다음 이미지는 [New Relic APM][new-relic]을 사용하여 생성된 결과입니다. 최대 워크로드 기간 동안 평균 데이터베이스 응답 시간은 요청당 약 5.6초에 최고점에 도달했습니다. 테스트 기간 동안 시스템은 분당 평균 410건의 요청을 지원할 수 있었습니다.

![AdventureWorks2012 데이터베이스를 방문하는 트래픽에 대한 개요][databasetraffic]

### <a name="gather-detailed-data-access-information"></a>상세 데이터 액세스 정보 수집

모니터링 데이터베이스를 자세히 살펴보면 응용 프로그램에서 3가지 다른 SQL SELECT 문을 실행하는 것이 보입니다. 이것은 `ProductListPriceHistory`, `Product` 및 `ProductSubcategory` 테이블에서 데이터를 가져오기 위해 Entity Framework에서 생성된 요청에 해당합니다.
또한 `ProductListPriceHistory` 테이블에서 데이터를 검색하는 쿼리는 규모 순으로 가장 자주 실행되는 SELECT 문입니다.

![테스트 중인 응용 프로그램 예제로 수행한 쿼리][queries]

앞서 살펴본 `GetProductsInSubCategoryAsync` 메서드는 45개의 SELECT 쿼리를 수행합니다. 각 쿼리는 응용 프로그램이 새 SQL 연결을 열도록 합니다.

![테스트 중인 응용 프로그램 예제에 대한 쿼리 통계][queries2]

> [!NOTE]
> 이 이미지는 부하 테스트에서 `GetProductsInSubCategoryAsync` 작업의 가장 느린 인스턴스에 대한 추적 정보를 보여줍니다. 프로덕션 환경에서는 가장 느린 인스턴스의 흔적을 조사하여 문제를 암시하는 패턴이 있는지 확인하는 것이 유용합니다. 평균값만 살펴보면 로드가 심하게 악화되는 문제를 간과할 수 있습니다.

다음 이미지는 실제 발급된 SQL 문을 보여줍니다. 가격 정보를 가져 오는 쿼리는 제품 하위 범주의 개별 제품에 대해 실행됩니다. 조인을 사용하면 데이터베이스 호출 수를 상당히 줄어 듭니다.

![테스트 중인 응용 프로그램 예제에 대한 쿼리 세부 정보][queries3]

Entity Framework와 같은 O/RM을 사용하는 경우, SQL 쿼리를 추적하면 O/RM이 프로그래밍 방식 호출을 SQL 문으로 변환하는 방식에 대한 통찰력을 얻을 수 있고 데이터 액세스가 최적화될 수 있는 영역을 알아낼 수 있습니다. 

### <a name="implement-the-solution-and-verify-the-result"></a>솔루션 구현 및 결과 확인

호출을 Entity Framework에 다시 쓰면 다음과 같은 결과가 나타납니다.

![번잡한 I/O 응용 프로그램 예제의 대규모 API에 대한 핵심 지표 부하 테스트 결과][key-indicators-chunky-io]

이 부하 테스트는 동일한 부하 프로필을 사용하여 동일한 배포에서 수행되었습니다. 이번에는 그래프에 표시된 대기 시간이 훨씬 낮습니다. 1000명의 사용자에 대한 평균 요청 시간은 5~6초 사이이며, 거의 1분에서 줄어 들었습니다.

이번에는 시스템이 분당 평균 3,970건(이전 테스트의 410건 대비)의 요청을 지원했습니다.

![대규모 API의 트랜잭션 개요][databasetraffic2]

SQL 문을 추적하면 모든 데이터를 단일 SELECT 문으로 가져온다는 것을 알 수 있습니다. 이 쿼리는 상당히 복잡하지만 작업당 한 번만 수행됩니다. 복잡한 조인은 비용이 높을 수 있지만 관계형 데이터베이스 시스템은 이러한 유형의 쿼리에 최적화되어 있습니다.  

![대규모 API에 대한 쿼리 세부 정보][queries4]

## <a name="related-resources"></a>관련 리소스

- [API 디자인 모범 사례][api-design]
- [캐싱 모범 사례][caching-guidance]
- [데이터 일관성 입문][data-consistency-guidance]
- [불필요한 가져오기 안티패턴][extraneous-fetching]
- [캐싱 없음 안티패턴][no-cache]

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

