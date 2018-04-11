---
title: 캐싱 없음 안티패턴
description: 동일한 데이터를 반복적으로 가져오면 성능과 확장성을 감소시킬 수 있습니다.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 8a2bc3b473a30536cc1bef9e1dcad87acb46c4a9
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/05/2018
---
# <a name="no-caching-antipattern"></a><span data-ttu-id="8a0dd-103">캐싱 없음 안티패턴</span><span class="sxs-lookup"><span data-stu-id="8a0dd-103">No Caching antipattern</span></span>

<span data-ttu-id="8a0dd-104">다수의 동시 요청을 처리하는 클라우드 응용 프로그램에서 같은 데이터를 반복적으로 가져오면 성능과 확장성을 감소시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-104">In a cloud application that handles many concurrent requests, repeatedly fetching the same data can reduce performance and scalability.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="8a0dd-105">문제 설명</span><span class="sxs-lookup"><span data-stu-id="8a0dd-105">Problem description</span></span>

<span data-ttu-id="8a0dd-106">데이터가 캐싱되지 않으면 다음과 같은 여러 가지 바람직하지 않은 동작이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-106">When data is not cached, it can cause a number of undesirable behaviors, including:</span></span>

- <span data-ttu-id="8a0dd-107">I/O 오버헤드 또는 대기 시간 측면에서 액세스 비용이 높은 리소스에서 동일한 정보를 반복적으로 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-107">Repeatedly fetching the same information from a resource that is expensive to access, in terms of I/O overhead or latency.</span></span>
- <span data-ttu-id="8a0dd-108">여러 요청에 대해 동일한 개체나 데이터 구조를 반복적으로 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-108">Repeatedly constructing the same objects or data structures for multiple requests.</span></span>
- <span data-ttu-id="8a0dd-109">서비스 할당량이 있는 원격 서비스를 과도하게 호출하고 특정 한도를 초과하여 클라이언트를 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-109">Making excessive calls to a remote service that has a service quota and throttles clients past a certain limit.</span></span>

<span data-ttu-id="8a0dd-110">결국, 이러한 문제로 인해 응답 시간이 저하되고, 데이터 저장소의 경합이 증가하고, 확장성이 떨어질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-110">In turn, these problems can lead to poor response times, increased contention in the data store, and poor scalability.</span></span>

<span data-ttu-id="8a0dd-111">다음 예제는 Entity Framework를 사용하여 데이터베이스에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-111">The following example uses Entity Framework to connect to a database.</span></span> <span data-ttu-id="8a0dd-112">여러 요청이 정확히 같은 데이터를 가져오더라도 모든 클라이언트 요청은 데이터베이스를 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-112">Every client request results in a call to the database, even if multiple requests are fetching exactly the same data.</span></span> <span data-ttu-id="8a0dd-113">반복되는 요청의 비용은 I/O 오버헤드 및 데이터 액세스 비용 측면에서, 신속하게 누적 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-113">The cost of repeated requests, in terms of I/O overhead and data access charges, can accumulate quickly.</span></span>

```csharp
public class PersonRepository : IPersonRepository
{
    public async Task<Person> GetAsync(int id)
    {
        using (var context = new AdventureWorksContext())
        {
            return await context.People
                .Where(p => p.Id == id)
                .FirstOrDefaultAsync()
                .ConfigureAwait(false);
        }
    }
}
```

<span data-ttu-id="8a0dd-114">전체 샘플은 [여기][sample-app]에서 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-114">You can find the complete sample [here][sample-app].</span></span>

<span data-ttu-id="8a0dd-115">이런 안티패턴이 발생하는 일반적인 이유는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-115">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="8a0dd-116">캐시를 사용하지 않는 것이 구현하기가 더 쉽고, 낮은 부하에서 정상적으로 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-116">Not using a cache is simpler to implement, and it works fine under low loads.</span></span> <span data-ttu-id="8a0dd-117">캐싱은 코드를 더욱 복잡하게 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-117">Caching makes the code more complicated.</span></span> 
- <span data-ttu-id="8a0dd-118">캐시 사용의 장단점을 명확하게 이해하기 힘듭니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-118">The benefits and drawbacks of using a cache are not clearly understood.</span></span>
- <span data-ttu-id="8a0dd-119">캐시된 데이터의 정확성과 최신 상태를 유지하는 오버헤드에 대한 우려가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-119">There is concern about the overhead of maintaining the accuracy and freshness of cached data.</span></span>
- <span data-ttu-id="8a0dd-120">네트워크 대기 시간이 문제가 되지 않는 온-프레미스 시스템에서 응용 프로그램이 마이그레이션되었고, 값 비싼 고성능 하드웨어에서 시스템이 실행되었으므로 원래 디자인에서는 캐싱이 고려되지 않았습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-120">An application was migrated from an on-premises system, where network latency was not an issue, and the system ran on expensive high-performance hardware, so caching wasn't considered in the original design.</span></span>
- <span data-ttu-id="8a0dd-121">주어진 시나리오에서 캐싱이 가능하다는 것을 개발자가 인식하지 못합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-121">Developers aren't aware that caching is a possibility in a given scenario.</span></span> <span data-ttu-id="8a0dd-122">예를 들어 웹 API를 구현할 때 개발자가 ETags 사용을 생각하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-122">For example, developers may not think of using ETags when implementing a web API.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="8a0dd-123">문제를 해결하는 방법</span><span class="sxs-lookup"><span data-stu-id="8a0dd-123">How to fix the problem</span></span>

<span data-ttu-id="8a0dd-124">가장 많이 사용되는 캐싱 전략은 *주문형* 또는 *캐시 배재* 전략입니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-124">The most popular caching strategy is the *on-demand* or *cache-aside* strategy.</span></span>

- <span data-ttu-id="8a0dd-125">읽기의 경우, 응용 프로그램은 캐시에서 데이터를 읽으려고 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-125">On read, the application tries to read the data from the cache.</span></span> <span data-ttu-id="8a0dd-126">데이터가 캐시에 없으면 응용 프로그램은 데이터 원본에서 데이터를 검색하여 캐시에 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-126">If the data isn't in the cache, the application retrieves it from the data source and adds it to the cache.</span></span>
- <span data-ttu-id="8a0dd-127">쓰기의 경우, 응용 프로그램은 변경 사항을 데이터 원본에 직접 쓰고 캐시에서 이전 값을 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-127">On write, the application writes the change directly to the data source and removes the old value from the cache.</span></span> <span data-ttu-id="8a0dd-128">다음에 필요할 때 검색되고 캐시에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-128">It will be retrieved and added to the cache the next time it is required.</span></span>

<span data-ttu-id="8a0dd-129">이 방식은 자주 변경되는 데이터에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-129">This approach is suitable for data that changes frequently.</span></span> <span data-ttu-id="8a0dd-130">다음은 [Cache-Aside][cache-aside] 패턴을 사용하도록 업데이트된 이전 예제입니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-130">Here is the previous example updated to use the [Cache-Aside][cache-aside] pattern.</span></span>  

```csharp
public class CachedPersonRepository : IPersonRepository
{
    private readonly PersonRepository _innerRepository;

    public CachedPersonRepository(PersonRepository innerRepository)
    {
        _innerRepository = innerRepository;
    }

    public async Task<Person> GetAsync(int id)
    {
        return await CacheService.GetAsync<Person>("p:" + id, () => _innerRepository.GetAsync(id)).ConfigureAwait(false);
    }
}

public class CacheService
{
    private static ConnectionMultiplexer _connection;

    public static async Task<T> GetAsync<T>(string key, Func<Task<T>> loadCache, double expirationTimeInMinutes)
    {
        IDatabase cache = Connection.GetDatabase();
        T value = await GetAsync<T>(cache, key).ConfigureAwait(false);
        if (value == null)
        {
            // Value was not found in the cache. Call the lambda to get the value from the database.
            value = await loadCache().ConfigureAwait(false);
            if (value != null)
            {
                // Add the value to the cache.
                await SetAsync(cache, key, value, expirationTimeInMinutes).ConfigureAwait(false);
            }
        }
        return value;
    }
}
```

<span data-ttu-id="8a0dd-131">`GetAsync` 메서드는 이제 데이터베이스를 직접 호출하지 않고 `CacheService` 클래스를 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-131">Notice that the `GetAsync` method now calls the `CacheService` class, rather than calling the database directly.</span></span> <span data-ttu-id="8a0dd-132">`CacheService` 클래스는 먼저 Azure Redis Cache에서 항목을 가져오려고 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-132">The `CacheService` class first tries to get the item from Azure Redis Cache.</span></span> <span data-ttu-id="8a0dd-133">Redis Cache에 값이 없으면 `CacheService`는 호출자가 전달한 람다 함수를 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-133">If the value isn't found in Redis Cache, the `CacheService` invokes a lambda function that was passed to it by the caller.</span></span> <span data-ttu-id="8a0dd-134">람다 함수는 데이터베이스에서 데이터를 가져오는 일을 담당합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-134">The lambda function is responsible for fetching the data from the database.</span></span> <span data-ttu-id="8a0dd-135">이런 구현은 특정 캐싱 솔루션에서 리포지토리를 분리하고 `CacheService`를 데이터베이스에서 분리합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-135">This implementation decouples the repository from the particular caching solution, and decouples the `CacheService` from the database.</span></span> 

## <a name="considerations"></a><span data-ttu-id="8a0dd-136">고려 사항</span><span class="sxs-lookup"><span data-stu-id="8a0dd-136">Considerations</span></span>

- <span data-ttu-id="8a0dd-137">일시적인 오류로 인해 캐시를 사용할 수 없는 경우 클라이언트에 오류를 반환하지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-137">If the cache is unavailable, perhaps because of a transient failure, don't return an error to the client.</span></span> <span data-ttu-id="8a0dd-138">대신 원래 데이터 원본에서 데이터를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-138">Instead, fetch the data from the original data source.</span></span> <span data-ttu-id="8a0dd-139">단, 캐시가 복구되는 동안 원래 데이터 저장소가 요청으로 가득 차서 제한 시간이 초과되고 연결되지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-139">However, be aware that while the cache is being recovered, the original data store could be swamped with requests, resulting in timeouts and failed connections.</span></span> <span data-ttu-id="8a0dd-140">(이것은 처음부터 캐시를 사용해야 하는 동기 중 하나입니다.) [회로 차단기 패턴][circuit-breaker]과 같은 기술을 사용하여 데이터 원본이 압도되지 않도록 하십시오.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-140">(After all, this is one of the motivations for using a cache in the first place.) Use a technique such as the [Circuit Breaker pattern][circuit-breaker] to avoid overwhelming the data source.</span></span>

- <span data-ttu-id="8a0dd-141">비정적 데이터를 캐시하는 응용 프로그램은 결과적 일관성을 지원하도록 설계되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-141">Applications that cache nonstatic data should be designed to support eventual consistency.</span></span>

- <span data-ttu-id="8a0dd-142">웹 API의 경우, 요청 및 응답 메시지에 Cache-Control 헤더를 포함시키고 ETag를 사용하여 개체 버전을 식별하면 클라이언트 쪽 캐싱을 지원할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-142">For web APIs, you can support client-side caching by including a Cache-Control header in request and response messages, and using ETags to identify versions of objects.</span></span> <span data-ttu-id="8a0dd-143">자세한 내용은 [API 구현][api-implementation]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-143">For more information, see [API implementation][api-implementation].</span></span>

- <span data-ttu-id="8a0dd-144">엔터티 전체를 캐시할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-144">You don't have to cache entire entities.</span></span> <span data-ttu-id="8a0dd-145">대부분의 엔터티가 정적이고 작은 부분만 자주 변경되는 경우에는 정적 요소를 캐시하고 동적 요소는 데이터 원본에서 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-145">If most of an entity is static but only a small piece changes frequently, cache the static elements and retrieve the dynamic elements from the data source.</span></span> <span data-ttu-id="8a0dd-146">이 방식은 데이터 원본에 대해 수행되는 I/O 볼륨을 줄이는 데 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-146">This approach can help to reduce the volume of I/O being performed against the data source.</span></span>

- <span data-ttu-id="8a0dd-147">경우에 따라 휘발성 데이터의 수명이 짧으면 캐시하는 것이 유용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-147">In some cases, if volatile data is short-lived, it can be useful to cache it.</span></span> <span data-ttu-id="8a0dd-148">예를 들어, 상태 업데이트를 지속적으로 보내는 장치의 경우</span><span class="sxs-lookup"><span data-stu-id="8a0dd-148">For example, consider a device that continually sends status updates.</span></span> <span data-ttu-id="8a0dd-149">정보가 도착하면 캐시하고 영구 저장소에 전혀 쓰지 않는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-149">It might make sense to cache this information as it arrives, and not write it to a persistent store at all.</span></span>  

- <span data-ttu-id="8a0dd-150">데이터가 부실해지는 것을 방지하기 위해 많은 캐싱 솔루션이 구성 가능한 만료 기간을 지원합니다. 따라서 지정된 간격이 지나면 데이터가 캐시에서 자동으로 제거됩니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-150">To prevent data from becoming stale, many caching solutions support configurable expiration periods, so that data is automatically removed from the cache after a specified interval.</span></span> <span data-ttu-id="8a0dd-151">시나리오의 만료 시간을 조정해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-151">You may need to tune the expiration time for your scenario.</span></span> <span data-ttu-id="8a0dd-152">매우 정적인 데이터는 휘발성 데이터(신속하게 부실한 상태가 되는 )보다 캐시에 오래 남아있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-152">Data that is highly static can stay in the cache for longer periods than volatile data that may become stale quickly.</span></span>

- <span data-ttu-id="8a0dd-153">캐싱 솔루션에 만료가 기본 제공되지 않는 경우, 캐시가 무제한으로 증가하는 것을 방지하기 위해 캐시를 가끔 비우는 백그라운드 프로세스를 구현해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-153">If the caching solution doesn't provide built-in expiration, you may need to implement a background process that occasionally sweeps the cache, to prevent it from growing without limits.</span></span> 

- <span data-ttu-id="8a0dd-154">외부 데이터 원본의 데이터를 캐싱하는 것 외에, 캐싱을 사용하여 복잡한 계산 결과를 저장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-154">Besides caching data from an external data source, you can use caching to save the results of complex computations.</span></span> <span data-ttu-id="8a0dd-155">단, 그렇게 하기 전에 응용 프로그램을 계측하여 응용 프로그램이 실제로 CPU 바운드인지 확인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-155">Before you do that, however, instrument the application to determine whether the application is really CPU bound.</span></span>

- <span data-ttu-id="8a0dd-156">응용 프로그램이 시작될 때 캐시를 준비하는 것이 유용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-156">It might be useful to prime the cache when the application starts.</span></span> <span data-ttu-id="8a0dd-157">가장 많이 사용될만한 데이터로 캐시를 채웁니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-157">Populate the cache with the data that is most likely to be used.</span></span>

- <span data-ttu-id="8a0dd-158">캐시 적중 및 캐시 누락을 감지하는 계측을 항상 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-158">Always include instrumentation that detects cache hits and cache misses.</span></span> <span data-ttu-id="8a0dd-159">이 정보를 사용하여 캐시할 데이터, 데이터가 만료되기 전에 캐시에 보유할 기간 등과 같은 캐싱 정책을 조정합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-159">Use this information to tune caching policies, such what data to cache, and how long to hold data in the cache before it expires.</span></span>

- <span data-ttu-id="8a0dd-160">캐싱이 부족하여 병목 상태가 되는 경우, 캐싱을 추가하면 요청 수가 너무 많아져서 웹 프런트 엔드에 과부하가 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-160">If the lack of caching is a bottleneck, then adding caching may increase the volume of requests so much that the web front end becomes overloaded.</span></span> <span data-ttu-id="8a0dd-161">클라이언트는 HTTP 503(서비스를 사용할 수 없음) 오류를 받기 시작할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-161">Clients may start to receive HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="8a0dd-162">이것은 프런트 엔드 규모를 확장해야 한다는 표시입니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-162">These are an indication that you should scale out the front end.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="8a0dd-163">문제를 감지하는 방법</span><span class="sxs-lookup"><span data-stu-id="8a0dd-163">How to detect the problem</span></span>

<span data-ttu-id="8a0dd-164">다음 단계를 수행하면 캐싱 부족으로 인해 성능 문제가 발생하는지 확인하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-164">You can perform the following steps to help identify whether lack of caching is causing performance problems:</span></span>

1. <span data-ttu-id="8a0dd-165">응용 프로그램 디자인을 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-165">Review the application design.</span></span> <span data-ttu-id="8a0dd-166">응용 프로그램에서 사용하는 모든 데이터 저장소의 인벤토리를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-166">Take an inventory of all the data stores that the application uses.</span></span> <span data-ttu-id="8a0dd-167">각각에 대해 응용 프로그램이 캐시를 사용하는지 여부를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-167">For each, determine whether the application is using a cache.</span></span> <span data-ttu-id="8a0dd-168">가능한 경우 데이터 변경 빈도를 파악합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-168">If possible, determine how frequently the data changes.</span></span> <span data-ttu-id="8a0dd-169">캐싱에 적합한 초기 후보로는 천천히 변하는 데이터와 빈번하게 읽혀지는 정적 참조 데이터가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-169">Good initial candidates for caching include data that changes slowly, and static reference data that is read frequently.</span></span> 

2. <span data-ttu-id="8a0dd-170">응용 프로그램을 계측하고 실시간 시스템을 모니터링하여 응용 프로그램이 데이터를 검색하거나 정보를 계산하는 빈도를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-170">Instrument the application and monitor the live system to find out how frequently the application retrieves data or calculates information.</span></span>

3. <span data-ttu-id="8a0dd-171">테스트 환경에서 응용 프로그램을 프로파일링하여 데이터 액세스 작업 또는 기타 자주 수행되는 계산과 관련된 오버 헤드에 대한 낮은 수준의 메트릭을 캡처합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-171">Profile the application in a test environment to capture low-level metrics about the overhead associated with data access operations or other frequently performed calculations.</span></span>

4. <span data-ttu-id="8a0dd-172">테스트 환경에서 부하 테스트를 수행하여 정상적인 워크로드 및 과부하 상태에서 시스템이 어떻게 반응하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-172">Perform load testing in a test environment to identify how the system responds under a normal workload and under heavy load.</span></span> <span data-ttu-id="8a0dd-173">부하 테스트는 실제 워크로드를 사용하여 프로덕션 환경에서 관찰되는 데이터 액세스 패턴을 시뮬레이션해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-173">Load testing should simulate the pattern of data access observed in the production environment using realistic workloads.</span></span> 

5. <span data-ttu-id="8a0dd-174">기본 데이터 저장소에 대한 데이터 액세스 통계를 검사하고 동일한 데이터 요청이 반복되는 빈도를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-174">Examine the data access statistics for the underlying data stores and review how often the same data requests are repeated.</span></span> 


## <a name="example-diagnosis"></a><span data-ttu-id="8a0dd-175">예제 진단</span><span class="sxs-lookup"><span data-stu-id="8a0dd-175">Example diagnosis</span></span>

<span data-ttu-id="8a0dd-176">다음 섹션에서는 이러한 단계를 앞에서 설명한 응용 프로그램 예제에 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-176">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-the-application-and-monitor-the-live-system"></a><span data-ttu-id="8a0dd-177">응용프로그램 계측 및 라이브 시스템 모니터링</span><span class="sxs-lookup"><span data-stu-id="8a0dd-177">Instrument the application and monitor the live system</span></span>

<span data-ttu-id="8a0dd-178">응용 프로그램을 계측하고 모니터링하여 응용 프로그램이 프로덕션 상태일 때 사용자가 생성하는 특정 요청에 대한 정보를 얻습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-178">Instrument the application and monitor it to get information about the specific requests that users make while the application is in production.</span></span> 

<span data-ttu-id="8a0dd-179">다음 이미지는 부하 테스트 중 [New Relic][NewRelic]으로 캡처한 모니터링 데이터를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-179">The following image shows monitoring data captured by [New Relic][NewRelic] during a load test.</span></span> <span data-ttu-id="8a0dd-180">이 경우 유일하게 수행된 HTTP GET 작업은 `Person/GetAsync`입니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-180">In this case, the only HTTP GET operation performed is `Person/GetAsync`.</span></span> <span data-ttu-id="8a0dd-181">그러나 실제 프로덕션 환경에서 각 요청이 수행되는 상대적 빈도를 알면 어떤 리소스를 캐시해야 하는지 파악할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-181">But in a live production environment, knowing the relative frequency that each request is performed can give you insight into which resources should be cached.</span></span>

![CachingDemo 응용 프로그램에 대한 서버 요청을 보여주는 New Relic][NewRelic-server-requests]

<span data-ttu-id="8a0dd-183">자세한 분석이 필요하면 프로파일러를 사용하여 프로덕션 시스템이 아닌 테스트 환경에서 저수준 성능 데이터를 캡처할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-183">If you need a deeper analysis, you can use a profiler to capture low-level performance data in a test environment (not the production system).</span></span> <span data-ttu-id="8a0dd-184">I/O 요청 속도, 메모리 사용량 및 CPU 사용률과 같은 메트릭을 살펴봅니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-184">Look at metrics such as I/O request rates, memory usage, and CPU utilization.</span></span> <span data-ttu-id="8a0dd-185">이러한 메트릭은 데이터 저장소나 서비스에 대한 많은 요청 또는 동일한 계산을 수행하는 반복되는 처리를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-185">These metrics may show a large number of requests to a data store or service, or repeated processing that performs the same calculation.</span></span> 

### <a name="load-test-the-application"></a><span data-ttu-id="8a0dd-186">응용 프로그램 부하 테스트</span><span class="sxs-lookup"><span data-stu-id="8a0dd-186">Load test the application</span></span>

<span data-ttu-id="8a0dd-187">다음 그래프는 응용 프로그램 예제 부하 테스트의 결과를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-187">The following graph shows the results of load testing the sample application.</span></span> <span data-ttu-id="8a0dd-188">부하 테스트는 일반적인 일련의 작업을 수행하는 사용자 최대 800명의 단계 부하를 시뮬레이션합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-188">The load test simulates a step load of up to 800 users performing a typical series of operations.</span></span> 

![캐시되지 않은 시나리오에 대한 성능 부하 테스트 결과][Performance-Load-Test-Results-Uncached]

<span data-ttu-id="8a0dd-190">매초 수행된 성공한 테스트 수가 안정기에 접어들고 결과적으로 추가 요청이 느려집니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-190">The number of successful tests performed each second reaches a plateau, and additional requests are slowed as a result.</span></span> <span data-ttu-id="8a0dd-191">평균 테스트 시간은 작업 부하에 따라 꾸준히 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-191">The average test time steadily increases with the workload.</span></span> <span data-ttu-id="8a0dd-192">사용자 로드가 정점에 도달하면 응답 시간 수준이 저하됩니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-192">The response time levels off once the user load peaks.</span></span>

### <a name="examine-data-access-statistics"></a><span data-ttu-id="8a0dd-193">데이터 액세스 통계 검사</span><span class="sxs-lookup"><span data-stu-id="8a0dd-193">Examine data access statistics</span></span>

<span data-ttu-id="8a0dd-194">데이터 액세스 통계 및 데이터 저장소에서 제공하는 기타 정보는 어떤 쿼리가 가장 자주 반복되는 지와 같은 유용한 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-194">Data access statistics and other information provided by a data store can give useful information, such as which queries are repeated most frequently.</span></span> <span data-ttu-id="8a0dd-195">예를 들어 Microsoft SQL Server의 경우 `sys.dm_exec_query_stats` 관리 뷰에 최근 실행한 쿼리에 대한 통계 정보가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-195">For example, in Microsoft SQL Server, the `sys.dm_exec_query_stats` management view has statistical information for recently executed queries.</span></span> <span data-ttu-id="8a0dd-196">각 쿼리에 대한 텍스트는 `sys.dm_exec-query_plan` 뷰에 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-196">The text for each query is available in the `sys.dm_exec-query_plan` view.</span></span> <span data-ttu-id="8a0dd-197">SQL Server Management Studio와 같은 도구를 사용하여 다음 SQL 쿼리를 실행하고 쿼리 수행 빈도를 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-197">You can use a tool such as SQL Server Management Studio to run the following SQL query and determine how frequently queries are performed.</span></span>

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

<span data-ttu-id="8a0dd-198">결과의 `UseCount` 열은 각 쿼리가 실행되는 빈도를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-198">The `UseCount` column in the results indicates how frequently each query is run.</span></span> <span data-ttu-id="8a0dd-199">다음 이미지는 3번째 쿼리가 다른 쿼리보다 훨씬 많은 250,000번 이상 실행되었음을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-199">The following image shows that the third query was run more than 250,000 times, significantly more than any other query.</span></span>

![SQL Server 관리 서버에서 동적 관리 뷰를 쿼리한 결과][Dynamic-Management-Views]

<span data-ttu-id="8a0dd-201">다음은 너무 많은 데이터베이스 요청을 유발하는 SQL 쿼리입니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-201">Here is the SQL query that is causing so many database requests:</span></span> 

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

<span data-ttu-id="8a0dd-202">앞에서 본 Entity Framework에서 `GetByIdAsync` 메서드로 생성한 쿼리입니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-202">This is the query that Entity Framework generates in `GetByIdAsync` method shown earlier.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="8a0dd-203">솔루션 구현 및 결과 확인</span><span class="sxs-lookup"><span data-stu-id="8a0dd-203">Implement the solution and verify the result</span></span>

<span data-ttu-id="8a0dd-204">캐시를 통합한 후 부하 테스트를 반복하고 그 결과를 캐시를 사용하지 않은 이전 부하 테스트와 비교합니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-204">After you incorporate a cache, repeat the load tests and compare the results to the earlier load tests without a cache.</span></span> <span data-ttu-id="8a0dd-205">다음은 응용 프로그램 예제에 캐시를 추가한 후 부하 테스트 결과입니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-205">Here are the load test results after adding a cache to the sample application.</span></span>

![캐시된 시나리오에 대한 성능 부하 테스트 결과][Performance-Load-Test-Results-Cached]

<span data-ttu-id="8a0dd-207">성공한 테스트 볼륨이 여전히 안정기에 접어들지만 사용자 로드는 더 높습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-207">The volume of successful tests still reaches a plateau, but at a higher user load.</span></span> <span data-ttu-id="8a0dd-208">이 로드에서 요청 속도는 이전보다 훨씬 높습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-208">The request rate at this load is significantly higher than earlier.</span></span> <span data-ttu-id="8a0dd-209">평균 테스트 시간은 부하에 따라 여전히 증가하지만 최대 응답 시간은 이전 1ms 대비 0.05ms로 &mdash;20배&times;가 향상되었습니다.</span><span class="sxs-lookup"><span data-stu-id="8a0dd-209">Average test time still increases with load, but the maximum response time is 0.05 ms, compared with 1ms earlier &mdash; a 20&times; improvement.</span></span> 

## <a name="related-resources"></a><span data-ttu-id="8a0dd-210">관련 리소스</span><span class="sxs-lookup"><span data-stu-id="8a0dd-210">Related resources</span></span>

- <span data-ttu-id="8a0dd-211">[API 구현 모범 사례][api-implementation]</span><span class="sxs-lookup"><span data-stu-id="8a0dd-211">[API implementation best practices][api-implementation]</span></span>
- <span data-ttu-id="8a0dd-212">[캐시 배제 패턴][cache-aside-pattern]</span><span class="sxs-lookup"><span data-stu-id="8a0dd-212">[Cache-Aside Pattern][cache-aside-pattern]</span></span>
- <span data-ttu-id="8a0dd-213">[캐싱 모범 사례][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="8a0dd-213">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="8a0dd-214">[회로 차단기 패턴][circuit-breaker]</span><span class="sxs-lookup"><span data-stu-id="8a0dd-214">[Circuit Breaker pattern][circuit-breaker]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/NoCaching
[cache-aside-pattern]: /azure/architecture/patterns/cache-aside
[caching-guidance]: ../../best-practices/caching.md
[circuit-breaker]: ../../patterns/circuit-breaker.md
[api-implementation]: ../../best-practices/api-implementation.md#optimizing-client-side-data-access
[NewRelic]: http://newrelic.com/azure
[NewRelic-server-requests]: _images/New-Relic.jpg
[Performance-Load-Test-Results-Uncached]:_images/InitialLoadTestResults.jpg
[Dynamic-Management-Views]: _images/SQLServerManagementStudio.jpg
[Performance-Load-Test-Results-Cached]: _images/CachedLoadTestResults.jpg