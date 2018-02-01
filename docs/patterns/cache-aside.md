---
title: Cache-Aside
description: "필요할 때 데이터를 데이터 저장소에서 캐시로 로드"
keywords: "디자인 패턴"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: e0a6a91fda6ea43236f6eea552f7b8f8d31160ad
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="cache-aside-pattern"></a><span data-ttu-id="c6ffa-104">캐시 배제 패턴</span><span class="sxs-lookup"><span data-stu-id="c6ffa-104">Cache-Aside pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="c6ffa-105">필요할 때 데이터를 데이터 저장소에서 캐시로 로드합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-105">Load data on demand into a cache from a data store.</span></span> <span data-ttu-id="c6ffa-106">이렇게 하면 성능이 개선되고 캐시에 저장된 데이터 및 기본 데이터 저장소의 데이터 간 일관성을 유지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-106">This can improve performance and also helps to maintain consistency between data held in the cache and data in the underlying data store.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="c6ffa-107">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="c6ffa-107">Context and problem</span></span>

<span data-ttu-id="c6ffa-108">응용 프로그램에서 캐시를 사용하여 데이터 저장소에 저장된 정보에 반복되는 액세스를 개선합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-108">Applications use a cache to improve repeated access to information held in a data store.</span></span> <span data-ttu-id="c6ffa-109">그러나, 캐시된 데이터가 언제나 데이터 저장소에 저장된 데이터와 일관성을 완전히 유지하지는 것은 불가능합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-109">However, it's impractical to expect that cached data will always be completely consistent with the data in the data store.</span></span> <span data-ttu-id="c6ffa-110">응용 프로그램은 캐시에 보관된 데이터를 최신 상태로 유지할 수 있도록 지원하는 전략을 구현하는 동시에, 캐시에 보관된 오래된 데이터로 인해 발생하는 상황을 감지하고 처리할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-110">Applications should implement a strategy that helps to ensure that the data in the cache is as up-to-date as possible, but can also detect and handle situations that arise when the data in the cache has become stale.</span></span>

## <a name="solution"></a><span data-ttu-id="c6ffa-111">해결 방법</span><span class="sxs-lookup"><span data-stu-id="c6ffa-111">Solution</span></span>

<span data-ttu-id="c6ffa-112">많은 상업용 캐싱 시스템은 read-through 및 write-through/write-behind 작업을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-112">Many commercial caching systems provide read-through and write-through/write-behind operations.</span></span> <span data-ttu-id="c6ffa-113">이런 캐싱 시스템에서 응용 프로그램은 캐시를 참조해 데이터를 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-113">In these systems, an application retrieves data by referencing the cache.</span></span> <span data-ttu-id="c6ffa-114">캐시에 데이터가 없으면 데이터 저장소에서 데이터가 검색된 후 캐시에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-114">If the data isn't in the cache, it's retrieved from the data store and added to the cache.</span></span> <span data-ttu-id="c6ffa-115">캐시에 보관된 데이터의 모든 수정 사항도 나중 쓰기 방식으로 데이터 저장소에 자동 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-115">Any modifications to data held in the cache are automatically written back to the data store as well.</span></span>

<span data-ttu-id="c6ffa-116">이 기능을 제공하지 않는 캐시의 경우, 데이터 유지를 위해 캐시를 사용하는 것은 응용 프로그램의 책임입니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-116">For caches that don't provide this functionality, it's the responsibility of the applications that use the cache to maintain the data.</span></span>

<span data-ttu-id="c6ffa-117">응용 프로그램은 캐시 배제 전략을 구현해 read-through 캐싱의 기능을 에뮬레이트할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-117">An application can emulate the functionality of read-through caching by implementing the cache-aside strategy.</span></span> <span data-ttu-id="c6ffa-118">이 전략은 데이터를 주문형 캐시에 로드합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-118">This strategy loads data into the cache on demand.</span></span> <span data-ttu-id="c6ffa-119">아래 그림은 캐시 배제 패턴을 사용해 데이터를 캐시에 저장하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-119">The figure illustrates using the Cache-Aside pattern to store data in the cache.</span></span>

![캐시 배제 패턴을 사용해 데이터를 캐시에 저장](./_images/cache-aside-diagram.png)


<span data-ttu-id="c6ffa-121">정보를 업데이트한 응용 프로그램은 데이터 저장소에 수정 사항을 저장하고 캐시의 해당 항목을 무효화해 write-through 전략을 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-121">If an application updates information, it can follow the write-through strategy by making the modification to the data store, and by invalidating the corresponding item in the cache.</span></span>

<span data-ttu-id="c6ffa-122">다음에도 해당 항목이 필요한 경우, 캐시 배제 전략을 사용하면 데이터 저장소에서 업데이트된 데이터가 검색된 후 캐시에 다시 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-122">When the item is next required, using the cache-aside strategy will cause the updated data to be retrieved from the data store and added back into the cache.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="c6ffa-123">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="c6ffa-123">Issues and considerations</span></span>

<span data-ttu-id="c6ffa-124">이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-124">Consider the following points when deciding how to implement this pattern:</span></span> 

<span data-ttu-id="c6ffa-125">**캐시된 데이터의 수명**.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-125">**Lifetime of cached data**.</span></span> <span data-ttu-id="c6ffa-126">대부분의 캐시는 규정된 기간 동안 액세스하지 않은 경우 데이터를 무효화하고 캐시에서 삭제하는 만료 정책을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-126">Many caches implement an expiration policy that invalidates data and removes it from the cache if it's not accessed for a specified period.</span></span> <span data-ttu-id="c6ffa-127">캐시 배제가 효과적이려면 만료 정책이 데이터를 사용하는 응용 프로그램의 액세스 패턴과 일치해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-127">For cache-aside to be effective, ensure that the expiration policy matches the pattern of access for applications that use the data.</span></span> <span data-ttu-id="c6ffa-128">응용 프로그램은 데이터 저장소에서 데이터를 지속적으로 검색해서 캐시에 추가할 수 있기 때문에, 만료 기간이 너무 짧아서도 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-128">Don't make the expiration period too short because this can cause applications to continually retrieve data from the data store and add it to the cache.</span></span> <span data-ttu-id="c6ffa-129">마찬가지로 캐시된 데이터는 오래 유지될 수 있기 때문에 만료 기간이 너무 길어서도 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-129">Similarly, don't make the expiration period so long that the cached data is likely to become stale.</span></span> <span data-ttu-id="c6ffa-130">캐싱은 비교적 정적인 데이터 또는 자주 읽는 데이터에 가장 효과입니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-130">Remember that caching is most effective for relatively static data, or data that is read frequently.</span></span>

<span data-ttu-id="c6ffa-131">**데이터 제거**.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-131">**Evicting data**.</span></span> <span data-ttu-id="c6ffa-132">대부분의 캐시는 데이터를 가져오는 데이터 저장소에 비해 크기가 제한되기 때문에 필요한 경우 데이터를 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-132">Most caches have a limited size compared to the data store where the data originates, and they'll evict data if necessary.</span></span> <span data-ttu-id="c6ffa-133">대부분의 캐시는 오래 전에 사용한 항목 정책을 채택해 제거 대상 항목을 선택하지만, 정책을 사용자 지정할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-133">Most caches adopt a least-recently-used policy for selecting items to evict, but this might be customizable.</span></span> <span data-ttu-id="c6ffa-134">캐시의 전역 만료 속성과 기타 속성 및 각각의 캐시된 항목의 만료 속성을 구성하여 캐시의 경제성을 높입니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-134">Configure the global expiration property and other properties of the cache, and the expiration property of each cached item, to ensure that the cache is cost effective.</span></span> <span data-ttu-id="c6ffa-135">한편 캐시의 모든 항목에 전역 제거 정책을 적용하는 것이 항상 적절한 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-135">It isn't always appropriate to apply a global eviction policy to every item in the cache.</span></span> <span data-ttu-id="c6ffa-136">예를 들어 데이터 저장소에서 캐시된 항목을 검색하는 비용이 높은 경우에는 더 자주 액세스하지만 비용이 낮은 항목을 제거하고 비용이 높은 항목을 유지하는 것이 유리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-136">For example, if a cached item is very expensive to retrieve from the data store, it can be beneficial to keep this item in the cache at the expense of more frequently accessed but less costly items.</span></span>

<span data-ttu-id="c6ffa-137">**캐시 초기화**.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-137">**Priming the cache**.</span></span> <span data-ttu-id="c6ffa-138">대부분의 솔루션은 시작 처리 과정에서 응용 프로그램에 필요할 수 있는 데이터를 미리 캐시에 채워둡니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-138">Many solutions prepopulate the cache with the data that an application is likely to need as part of the startup processing.</span></span> <span data-ttu-id="c6ffa-139">시 배제 패턴은 이런 데이터 중 일부가 만료되거나 제거되는 경우에도 여전히 유용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-139">The Cache-Aside pattern can still be useful if some of this data expires or is evicted.</span></span>

<span data-ttu-id="c6ffa-140">**일관성**.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-140">**Consistency**.</span></span> <span data-ttu-id="c6ffa-141">캐시 배제 패턴의 구현이 데이터 저장소와 캐시의 일관성을 보증하는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-141">Implementing the Cache-Aside pattern doesn't guarantee consistency between the data store and the cache.</span></span> <span data-ttu-id="c6ffa-142">데이터 저장소의 항목은 외부 프로세스를 통해 언제든 변경될 수 있고, 이와 같은 변경은 해당 항목을 다시 로드할 때까지 캐시에 반영되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-142">An item in the data store can be changed at any time by an external process, and this change might not be reflected in the cache until the next time the item is loaded.</span></span> <span data-ttu-id="c6ffa-143">여러 데이터 저장소에 데이터를 복제하는 시스템에서 동기화가 자주 이루어질 경우 이와 같은 문제가 심각한 문제로 발전할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-143">In a system that replicates data across data stores, this problem can become serious if synchronization occurs frequently.</span></span>

<span data-ttu-id="c6ffa-144">**로컬(메모리 내) 캐싱**.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-144">**Local (in-memory) caching**.</span></span> <span data-ttu-id="c6ffa-145">캐시는 응용 프로그램 인스턴스에 대해 로컬일 수 있으며 메모리 내에 저장될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-145">A cache could be local to an application instance and stored in-memory.</span></span> <span data-ttu-id="c6ffa-146">캐시 배제는 응용 프로그램이 동일한 데이터에 반복적으로 액세스하는 경우 로컬 환경에서 유용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-146">Cache-aside can be useful in this environment if an application repeatedly accesses the same data.</span></span> <span data-ttu-id="c6ffa-147">그러나 로컬 캐시는 개인용이므로, 다양한 응용 프로그램 인스턴스 각각은 동일한 캐시된 데이터의 사본을 보유할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-147">However, a local cache is private and so different application instances could each have a copy of the same cached data.</span></span> <span data-ttu-id="c6ffa-148">이런 데이터는 캐시 사이에서 일관성을 빠르게 상실할 수 있으므로, 개인 캐시에 보관된 데이터는 만료시키고 더 자주 새로 고칠 필요가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-148">This data could quickly become inconsistent between caches, so it might be necessary to expire data held in a private cache and refresh it more frequently.</span></span> <span data-ttu-id="c6ffa-149">이런 시나리오에서는 공유 또는 분산 캐싱 메커니즘의 사용 여부 검토를 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-149">In these scenarios, consider investigating the use of a shared or a distributed caching mechanism.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="c6ffa-150">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="c6ffa-150">When to use this pattern</span></span>

<span data-ttu-id="c6ffa-151">다음의 경우에 이 패턴을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-151">Use this pattern when:</span></span>

- <span data-ttu-id="c6ffa-152">캐시가 기본 read-through 및 write-through 작업을 제공하지 않는 경우.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-152">A cache doesn't provide native read-through and write-through operations.</span></span>
- <span data-ttu-id="c6ffa-153">리소스 수요를 예측할 수 없는 경우.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-153">Resource demand is unpredictable.</span></span> <span data-ttu-id="c6ffa-154">이 패턴을 사용하면 응용 프로그램이 주문형 데이터를 로드할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-154">This pattern enables applications to load data on demand.</span></span> <span data-ttu-id="c6ffa-155">이 패턴은 응용 프로그램에 필요할 데이터에 대해 사전에 가정하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-155">It makes no assumptions about which data an application will require in advance.</span></span>

<span data-ttu-id="c6ffa-156">다음 경우에는 이 패턴이 적합하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-156">This pattern might not be suitable:</span></span>

- <span data-ttu-id="c6ffa-157">캐시된 데이터 집합이 정적인 경우.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-157">When the cached data set is static.</span></span> <span data-ttu-id="c6ffa-158">사용 가능한 캐시 공간에 데이터가 가득 차면 시작 시 데이터가 채워진 캐시를 초기화하고 데이터의 만료를 방지하는 정책을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-158">If the data will fit into the available cache space, prime the cache with the data on startup and apply a policy that prevents the data from expiring.</span></span>
- <span data-ttu-id="c6ffa-159">웹 팜에 호스트되는 웹 응용 프로그램 내 캐싱 세션 상태 정보.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-159">For caching session state information in a web application hosted in a web farm.</span></span> <span data-ttu-id="c6ffa-160">이런 환경에서는 클라이언트-서버 선호도를 기반으로 하는 종속성 도입을 피해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-160">In this environment, you should avoid introducing dependencies based on client-server affinity.</span></span>

## <a name="example"></a><span data-ttu-id="c6ffa-161">예</span><span class="sxs-lookup"><span data-stu-id="c6ffa-161">Example</span></span>

<span data-ttu-id="c6ffa-162">Microsoft Azure에서는 Azure Redis Cache를 사용해 응용 프로그램의 여러 인스턴스가 공유할 수 있는 분산 캐시를 생성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-162">In Microsoft Azure you can use Azure Redis Cache to create a distributed cache that can be shared by multiple instances of an application.</span></span> 

<span data-ttu-id="c6ffa-163">Azure Redis Cache 인스턴스에 연결하려면 정적 `Connect` 메서드를 호출하고 연결 문자열을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-163">To connect to an Azure Redis Cache instance, call the static `Connect` method and pass in the connection string.</span></span> <span data-ttu-id="c6ffa-164">이 메서드는 연결을 나타내는 `ConnectionMultiplexer`를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-164">The method returns a `ConnectionMultiplexer` that represents the connection.</span></span> <span data-ttu-id="c6ffa-165">응용 프로그램의 `ConnectionMultiplexer` 인스턴스를 공유하는 방법은 다음 예제와 비슷하게 연결된 인스턴스를 반환하는 정적 속성을 갖는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-165">One approach to sharing a `ConnectionMultiplexer` instance in your application is to have a static property that returns a connected instance, similar to the following example.</span></span> <span data-ttu-id="c6ffa-166">이 방법은 스레드가 안전하도록 단일 연결된 인스턴스를 초기화하는 방법을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-166">This approach provides a thread-safe way to initialize only a single connected instance.</span></span>

```csharp
private static ConnectionMultiplexer Connection;

// Redis Connection string info
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    string cacheConnection = ConfigurationManager.AppSettings["CacheConnection"].ToString();
    return ConnectionMultiplexer.Connect(cacheConnection);
});

public static ConnectionMultiplexer Connection => lazyConnection.Value;
```

<span data-ttu-id="c6ffa-167">다음 코드 예제의 `GetMyEntityAsync` 메서드는 Azure Redis Cache를 기반으로 하는 캐시 배제 패턴의 구현을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-167">The `GetMyEntityAsync` method in the following code example shows an implementation of the Cache-Aside pattern based on Azure Redis Cache.</span></span> <span data-ttu-id="c6ffa-168">이 메서드는 read-though 접근 방식을 사용해 캐시에서 개체를 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-168">This method retrieves an object from the cache using the read-though approach.</span></span>

<span data-ttu-id="c6ffa-169">개체 식별 키는 정수 ID입니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-169">An object is identified by using an integer ID as the key.</span></span> <span data-ttu-id="c6ffa-170">`GetMyEntityAsync` 메서드는 이 키를 포함한 항목을 캐시에서 검색하려고 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-170">The `GetMyEntityAsync` method tries to retrieve an item with this key from the cache.</span></span> <span data-ttu-id="c6ffa-171">일치하는 항목이 발견되면 해당 항목이 반환됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-171">If a matching item is found, it's returned.</span></span> <span data-ttu-id="c6ffa-172">캐시에 일치하는 항목이 없으면 `GetMyEntityAsync` 메서드는 개체를 데이터 저장소에서 검색해 캐시에 추가한 후 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-172">If there's no match in the cache, the `GetMyEntityAsync` method retrieves the object from a data store, adds it to the cache, and then returns it.</span></span> <span data-ttu-id="c6ffa-173">데이터 저장소의 종속성 때문에, 데이터 저장소에서 데이터를 실제로 읽는 코드는 여기에 표시되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-173">The code that actually reads the data from the data store is not shown here, because it depends on the data store.</span></span> <span data-ttu-id="c6ffa-174">캐시된 항목을 만료 방식으로 구성하여 다른 위치에서 업데이트되더라도 오래되지 않도록 조치해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-174">Note that the cached item is configured to expire to prevent it from becoming stale if it's updated elsewhere.</span></span>


```csharp
// Set five minute expiration as a default
private const double DefaultExpirationTimeInMinutes = 5.0;

public async Task<MyEntity> GetMyEntityAsync(int id)
{
  // Define a unique key for this method and its parameters.
  var key = $"MyEntity:{id}";
  var cache = Connection.GetDatabase();
  
  // Try to get the entity from the cache.
  var json = await cache.StringGetAsync(key).ConfigureAwait(false);
  var value = string.IsNullOrWhiteSpace(json) 
                ? default(MyEntity) 
                : JsonConvert.DeserializeObject<MyEntity>(json);
  
  if (value == null) // Cache miss
  {
    // If there's a cache miss, get the entity from the original store and cache it.
    // Code has been omitted because it's data store dependent.  
    value = ...;

    // Avoid caching a null value.
    if (value != null)
    {
      // Put the item in the cache with a custom expiration time that 
      // depends on how critical it is to have stale data.
      await cache.StringSetAsync(key, JsonConvert.SerializeObject(value)).ConfigureAwait(false);
      await cache.KeyExpireAsync(key, TimeSpan.FromMinutes(DefaultExpirationTimeInMinutes)).ConfigureAwait(false);
    }
  }

  return value;
}
```

>  <span data-ttu-id="c6ffa-175">위의 예제는 Azure Redis Cache API를 사용해 저장소에 액세스하고 캐시에서 정보를 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-175">The examples use the Azure Redis Cache API to access the store and retrieve information from the cache.</span></span> <span data-ttu-id="c6ffa-176">자세한 내용은 [Microsoft Azure Redis Cache 사용](https://docs.microsoft.com/en-us/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) 및 [Redis Cache를 사용하여 웹앱을 만드는 방법](https://docs.microsoft.com/en-us/azure/redis-cache/cache-web-app-howto)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-176">For more information, see [Using Microsoft Azure Redis Cache](https://docs.microsoft.com/en-us/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache) and [How to create a Web App with Redis Cache](https://docs.microsoft.com/en-us/azure/redis-cache/cache-web-app-howto)</span></span>

<span data-ttu-id="c6ffa-177">아래에 표시된 `UpdateEntityAsync` 메서드는 응용 프로그램에서 값이 변경될 때 캐시의 개체를 무효화하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-177">The `UpdateEntityAsync` method shown below demonstrates how to invalidate an object in the cache when the value is changed by the application.</span></span> <span data-ttu-id="c6ffa-178">다음 코드는 원래 데이터 저장소를 업데이트한 다음, 캐시에서 캐시된 항목을 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-178">The code updates the original data store and then removes the cached item from the cache.</span></span>

```csharp
public async Task UpdateEntityAsync(MyEntity entity)
{
    // Update the object in the original data store.
    await this.store.UpdateEntityAsync(entity).ConfigureAwait(false); 

    // Invalidate the current cache object.
    var cache = Connection.GetDatabase();
    var id = entity.Id;
    var key = $"MyEntity:{id}"; // The key for the cached object.
    await cache.KeyDeleteAsync(key).ConfigureAwait(false); // Delete this key from the cache.
}
```

> [!NOTE]
> <span data-ttu-id="c6ffa-179">이 시퀀스에서는 단계의 순서가 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-179">The order of the steps is important.</span></span> <span data-ttu-id="c6ffa-180">캐시에서 항목을 제거하기 *전에* 데이터 저장소를 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-180">Update the data store *before* removing the item from the cache.</span></span> <span data-ttu-id="c6ffa-181">캐시된 항목을 먼저 제거하면 데이터 저장소가 업데이트되기 전에 클라이언트 응용 프로그램이 데이터를 가져오는 시간이 짧아집니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-181">If you remove the cached item first, there is a small window of time when a client might fetch the item before the data store is updated.</span></span> <span data-ttu-id="c6ffa-182">이 경우 캐시에서 항목이 제거되었으므로 캐시 누락이 발생하며, 이전 버전의 항목을 데이터 저장소에서 가져온 후 캐시에 다시 추가하는 결과를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-182">That will result in a cache miss (because the item was removed from the cache), causing the earlier version of the item to be fetched from the data store and added back into the cache.</span></span> <span data-ttu-id="c6ffa-183">따라서 캐시는 오래된 데이터를 포함하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-183">The result will be stale cache data.</span></span>


## <a name="related-guidance"></a><span data-ttu-id="c6ffa-184">관련 지침</span><span class="sxs-lookup"><span data-stu-id="c6ffa-184">Related guidance</span></span> 

<span data-ttu-id="c6ffa-185">이 패턴의 구현과 관련된 정보는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-185">The following information may be relevant when implementing this pattern:</span></span>

- <span data-ttu-id="c6ffa-186">[캐싱 지침](https://docs.microsoft.com/en-us/azure/architecture/best-practices/caching)</span><span class="sxs-lookup"><span data-stu-id="c6ffa-186">[Caching Guidance](https://docs.microsoft.com/en-us/azure/architecture/best-practices/caching).</span></span> <span data-ttu-id="c6ffa-187">클라우드 솔루션에서 데이터를 캐시할 수 있는 방법에 대한 추가 정보뿐 아니라 캐시 구현 시 고려해야 하는 문제를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-187">Provides additional information on how you can cache data in a cloud solution, and the issues that you should consider when you implement a cache.</span></span>

- <span data-ttu-id="c6ffa-188">[데이터 일관성 입문서](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="c6ffa-188">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="c6ffa-189">보통 클라우드 응용 프로그램은 여러 데이터 저장소에 걸쳐 있는 데이터를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-189">Cloud applications typically use data that's spread across data stores.</span></span> <span data-ttu-id="c6ffa-190">이런 환경에서는 데이터 일관성의 관리와 유지가 시스템에서 중요한 측면으로 작용하는데, 구체적으로 동시성과 가용성 문제가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-190">Managing and maintaining data consistency in this environment is a critical aspect of the system, particularly the concurrency and availability issues that can arise.</span></span> <span data-ttu-id="c6ffa-191">이 입문서는 분산 데이터의 일관성 문제를 설명할 뿐 아니라 응용 프로그램에서 일관성을 구현해 데이터의 가용성을 유지하는 방법도 요약하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c6ffa-191">This primer describes issues about consistency across distributed data, and summarizes how an application can implement eventual consistency to maintain the availability of data.</span></span>
