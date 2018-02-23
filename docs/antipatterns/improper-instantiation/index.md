---
title: "부적절한 인스턴스화 안티패턴"
description: "한 번 만든 다음 공유하는 개체의 새 인스턴스를 계속 만들지 마십시오."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 4d5ef9ad9e675b46df94b51e81d7a4bd4c1b25e9
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/23/2018
---
# <a name="improper-instantiation-antipattern"></a><span data-ttu-id="0e937-103">부적절한 인스턴스화 안티패턴</span><span class="sxs-lookup"><span data-stu-id="0e937-103">Improper Instantiation antipattern</span></span>

<span data-ttu-id="0e937-104">한 번 만든 다음 공유하는 개체의 새 인스턴스를 계속 만들면 성능이 떨어질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-104">It can hurt performance to continually create new instances of an object that is meant to be created once and then shared.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="0e937-105">문제 설명</span><span class="sxs-lookup"><span data-stu-id="0e937-105">Problem description</span></span>

<span data-ttu-id="0e937-106">많은 라이브러리가 외부 리소스의 추상화를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-106">Many libraries provide abstractions of external resources.</span></span> <span data-ttu-id="0e937-107">내부적으로 이러한 클래스는 대개 클라이언트가 리소스에 액세스하는 데 사용할 수 있는 broker 역할을 하는 리소스에 대한 자체 연결을 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-107">Internally, these classes typically manage their own connections to the resource, acting as brokers that clients can use to access the resource.</span></span> <span data-ttu-id="0e937-108">다음은 Azure 응용 프로그램과 관련된 broker 클래스의 몇 가지 예입니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-108">Here are some examples of broker classes that are relevant to Azure applications:</span></span>

- <span data-ttu-id="0e937-109">`System.Net.Http.HttpClient`</span><span class="sxs-lookup"><span data-stu-id="0e937-109">`System.Net.Http.HttpClient`.</span></span> <span data-ttu-id="0e937-110">HTTP를 사용하여 웹 서비스와 통신합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-110">Communicates with a web service using HTTP.</span></span>
- <span data-ttu-id="0e937-111">`Microsoft.ServiceBus.Messaging.QueueClient`</span><span class="sxs-lookup"><span data-stu-id="0e937-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span></span> <span data-ttu-id="0e937-112">Service Bus 큐에 메시지를 게시하고 수신합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-112">Posts and receives messages to a Service Bus queue.</span></span> 
- <span data-ttu-id="0e937-113">`Microsoft.Azure.Documents.Client.DocumentClient`</span><span class="sxs-lookup"><span data-stu-id="0e937-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span></span> <span data-ttu-id="0e937-114">Cosmos DB 인스턴스에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-114">Connects to a Cosmos DB instance</span></span>
- <span data-ttu-id="0e937-115">`StackExchange.Redis.ConnectionMultiplexer`</span><span class="sxs-lookup"><span data-stu-id="0e937-115">`StackExchange.Redis.ConnectionMultiplexer`.</span></span> <span data-ttu-id="0e937-116">Azure Redis Cache를 포함하여 Redis에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-116">Connects to Redis, including Azure Redis Cache.</span></span>

<span data-ttu-id="0e937-117">이러한 클래스는 한 번 인스턴스화되면 응용 프로그램의 수명 내내 다시 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-117">These classes are intended to be instantiated once and reused throughout the lifetime of an application.</span></span> <span data-ttu-id="0e937-118">그러나 이러한 클래스를 필요한 경우에만 확보하고 신속하게 릴리스해야 한다는 것은 일반적인 오해입니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-118">However, it's a common misunderstanding that these classes should be acquired only as necessary and released quickly.</span></span> <span data-ttu-id="0e937-119">(여기 나열된 항목은 .NET 라이브러리이지만 패턴은 .NET 고유 패턴이 아닙니다.) 다음 ASP.NET 예제는 원격 서비스와 통신하기 위해 `HttpClient` 인스턴스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-119">(The ones listed here happen to be .NET libraries, but the pattern is not unique to .NET.) The following ASP.NET example creates an instance of `HttpClient` to communicate with a remote service.</span></span> <span data-ttu-id="0e937-120">전체 샘플은 [여기][sample-app]에서 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-120">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

<span data-ttu-id="0e937-121">웹 응용 프로그램에서 이 기술은 확장성이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-121">In a web application, this technique is not scalable.</span></span> <span data-ttu-id="0e937-122">새 `HttpClient` 개체가 각 사용자 요청에 대해 만들어집니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-122">A new `HttpClient` object is created for each user request.</span></span> <span data-ttu-id="0e937-123">과부하 시, 웹 서버에서 사용 가능한 소켓 수가 소진되어 `SocketException` 오류가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-123">Under heavy load, the web server may exhaust the number of available sockets, resulting in `SocketException` errors.</span></span>

<span data-ttu-id="0e937-124">이 문제는 `HttpClient` 클래스에만 국한되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-124">This problem is not restricted to the `HttpClient` class.</span></span> <span data-ttu-id="0e937-125">리소스를 래핑하거나 만드는 비용이 높은 다른 클래스도 유사한 문제를 유발할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-125">Other classes that wrap resources or are expensive to create might cause similar issues.</span></span> <span data-ttu-id="0e937-126">다음 예제는 `ExpensiveToCreateService` 클래스에 대한 인스턴스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-126">The following example creates an instances of the `ExpensiveToCreateService` class.</span></span> <span data-ttu-id="0e937-127">여기서 문제는 소켓 소진이 아니라 각 인스턴스를 만드는 데 걸리는 시간입니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-127">Here the issue is not necessarily socket exhaustion, but simply how long it takes to create each instance.</span></span> <span data-ttu-id="0e937-128">이 클래스의 인스턴스를 계속 만들고 제거하면 시스템의 확장성에 부정적인 영향을 미칠 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-128">Continually creating and destroying instances of this class might adversely affect the scalability of the system.</span></span>

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="0e937-129">문제를 해결하는 방법</span><span class="sxs-lookup"><span data-stu-id="0e937-129">How to fix the problem</span></span>

<span data-ttu-id="0e937-130">외부 리소스를 래핑하는 클래스가 공유가 가능하고 스레드로부터 안전한 경우 공유 singleton 인스턴스 또는 재사용 가능한 클래스 인스턴스 풀을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-130">If the class that wraps the external resource is shareable and thread-safe, create a shared singleton instance or a pool of reusable instances of the class.</span></span>

<span data-ttu-id="0e937-131">다음 예제에는 정적 `HttpClient` 인스턴스를 사용하므로 모든 요청에서 연결을 공유합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-131">The following example uses a static `HttpClient` instance, thus sharing the connection across all requests.</span></span>

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient httpClient;

    static SingleHttpClientInstanceController()
    {
        httpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a><span data-ttu-id="0e937-132">고려 사항</span><span class="sxs-lookup"><span data-stu-id="0e937-132">Considerations</span></span>

- <span data-ttu-id="0e937-133">이 안티패턴의 핵심 요소는 *공유가 가능한* 개체 인스턴스를 반복적으로 만들고 제거하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-133">The key element of this antipattern is repeatedly creating and destroying instances of a *shareable* object.</span></span> <span data-ttu-id="0e937-134">클래스를 공유할 수 없는(스레드로부터 안전하지 않은) 경우에는 이 안티패턴이 적용되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-134">If a class is not shareable (not thread-safe), then this antipattern does not apply.</span></span>

- <span data-ttu-id="0e937-135">공유 리소스의 유형에 따라 singleton을 사용할지 풀을 만들지 여부가 결정될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-135">The type of shared resource might dictate whether you should use a singleton or create a pool.</span></span> <span data-ttu-id="0e937-136">`HttpClient` 클래스는 풀링되지 않고 공유되도록 설계되었습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-136">The `HttpClient` class is designed to be shared rather than pooled.</span></span> <span data-ttu-id="0e937-137">다른 개체는 풀링을 지원하기 때문에 시스템이 워크로드를 여러 인스턴스로 분산할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-137">Other objects might support pooling, enabling the system to spread the workload across multiple instances.</span></span>

- <span data-ttu-id="0e937-138">여러 요청에서 공유하는 개체는 *반드시* 스레드로부터 안전해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-138">Objects that you share across multiple requests *must* be thread-safe.</span></span> <span data-ttu-id="0e937-139">`HttpClient` 클래스는 이러한 방식으로 사용하도록 설계되었지만 다른 클래스는 동시 요청을 지원하지 않을 수 있으니 사용 가능한 설명서를 확인하십시오.</span><span class="sxs-lookup"><span data-stu-id="0e937-139">The `HttpClient` class is designed to be used in this manner, but other classes might not support concurrent requests, so check the available documentation.</span></span>

- <span data-ttu-id="0e937-140">일부 리소스는 희박하며 보유하지 말아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-140">Some resource types are scarce and should not be held onto.</span></span> <span data-ttu-id="0e937-141">데이터베이스 연결이 그 예입니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-141">Database connections are an example.</span></span> <span data-ttu-id="0e937-142">필요하지 않은 열린 데이터베이스 연결을 보유하면 다른 동시 사용자가 데이터베이스에 액세스하는 것을 막을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-142">Holding an open database connection that is not required may prevent other concurrent users from gaining access to the database.</span></span>

- <span data-ttu-id="0e937-143">.NET Framework에서는 외부 리소스에 대한 연결을 설정하는 많은 개체가 이러한 연결을 관리하는 다른 클래스의 정적 팩터리 메서드를 사용하여 만들어집니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-143">In the .NET Framework, many objects that establish connections to external resources are created by using static factory methods of other classes that manage these connections.</span></span> <span data-ttu-id="0e937-144">이런 팩터리 개체는 삭제되고 다시 만들어지기 보다 저장되고 다시 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-144">These factories  objects are intended to be saved and reused, rather than disposed and recreated.</span></span> <span data-ttu-id="0e937-145">예를 들어 Azure Service Bus에서 `QueueClient` 개체는 `MessagingFactory` 개체를 통해 만들어집니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-145">For example, in Azure Service Bus, the `QueueClient` object is created through a `MessagingFactory` object.</span></span> <span data-ttu-id="0e937-146">내부적으로 `MessagingFactory`가 연결을 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-146">Internally, the `MessagingFactory` manages connections.</span></span> <span data-ttu-id="0e937-147">자세한 내용은 [Service Bus 메시징을 사용한 성능 향상의 모범 사례][service-bus-messaging]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0e937-147">For more information, see [Best Practices for performance improvements using Service Bus Messaging][service-bus-messaging].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="0e937-148">문제를 감지하는 방법</span><span class="sxs-lookup"><span data-stu-id="0e937-148">How to detect the problem</span></span>

<span data-ttu-id="0e937-149">이 문제의 증상에는 처리량이 떨어지거나 오류 비율이 증가하는 경우와 함께 다음 중 하나 이상의 경우가 동반됩니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-149">Symptoms of this problem include a drop in throughput or an increased error rate, along with one or more of the following:</span></span> 

- <span data-ttu-id="0e937-150">소켓, 데이터베이스 연결, 파일 핸들 등과 같은 리소스가 소진된 것을 나타내는 예외가 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-150">An increase in exceptions that indicate exhaustion of resources such as sockets, database connections, file handles, and so on.</span></span> 
- <span data-ttu-id="0e937-151">메모리 사용 및 가비지 수집이 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-151">Increased memory use and garbage collection.</span></span>
- <span data-ttu-id="0e937-152">네트워크, 디스크 또는 데이터베이스 활동이 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-152">An increase in network, disk, or database activity.</span></span>

<span data-ttu-id="0e937-153">다음 단계를 수행하면 문제를 식별하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-153">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="0e937-154">프로덕션 시스템의 프로세스를 모니터링하여 응답 시간이 느려지거나 리소스 부족으로 인해 시스템에 장애가 발생하는 지점을 파악합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-154">Performing process monitoring of the production system, to identify points when response times slow down or the system fails due to lack of resources.</span></span>
2. <span data-ttu-id="0e937-155">이 지점에서 캡처된 원격 분석 데이터를 검사하여 어떤 작업이 리소스를 많이 소비하는 개체를 만들고 제거하는지 파악합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-155">Examine the telemetry data captured at these points to determine which operations might be creating and destroying resource-consuming objects.</span></span>
3. <span data-ttu-id="0e937-156">프로덕션 시스템이 아닌 통제된 테스트 환경에서 의심되는 각 작업의 부하를 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-156">Load test each suspected operation, in a controlled test environment rather than the production system.</span></span>
4. <span data-ttu-id="0e937-157">소스 코드를 검토하고 broker 개체가 관리되는 방식을 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-157">Review the source code and examine the how broker objects are managed.</span></span>

<span data-ttu-id="0e937-158">시스템에 부하가 걸렸을 때 느리게 실행되거나 예외를 생성하는 작업에 대한 스택 추적을 살펴봅니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-158">Look at stack traces for operations that are slow-running or that generate exceptions when the system is under load.</span></span> <span data-ttu-id="0e937-159">이 정보는 해당 작업이 리소스를 활용하는 방법을 파악하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-159">This information can help to identify how these operations are utilizing resources.</span></span> <span data-ttu-id="0e937-160">예외는 오류가 공유 자원이 소진되어 발생했는지를 파악하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-160">Exceptions can help to determine whether errors are caused by shared resources being exhausted.</span></span> 

## <a name="example-diagnosis"></a><span data-ttu-id="0e937-161">예제 진단</span><span class="sxs-lookup"><span data-stu-id="0e937-161">Example diagnosis</span></span>

<span data-ttu-id="0e937-162">다음 섹션에서는 이러한 단계를 앞에서 설명한 응용 프로그램 예제에 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-162">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slow-down-or-failure"></a><span data-ttu-id="0e937-163">느려지거나 실패한 지점 식별</span><span class="sxs-lookup"><span data-stu-id="0e937-163">Identify points of slow down or failure</span></span>

<span data-ttu-id="0e937-164">다음 이미지는 [New Relic APM][new-relic]을 사용하여 생성된 결과이며, 응답 시간이 열악한 작업을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-164">The following image shows results generated using [New Relic APM][new-relic], showing operations that have a poor response time.</span></span> <span data-ttu-id="0e937-165">이 경우 `NewHttpClientInstancePerRequest` 컨트롤러의 `GetProductAsync` 메서드는 보다 자세히 조사하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-165">In this case, the `GetProductAsync` method in the `NewHttpClientInstancePerRequest` controller is worth investigating further.</span></span> <span data-ttu-id="0e937-166">이러한 작업이 실행 중일 때 오류 비율도 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-166">Notice that the error rate also increases when these operations are running.</span></span> 

![각 요청에 대해 HttpClient 개체의 새 인스턴스를 만드는 응용 프로그램 예제를 보여주는 New Relic 모니터링 대시 보드][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="0e937-168">원격 분석 데이터 검사 및 상관 관계 찾기</span><span class="sxs-lookup"><span data-stu-id="0e937-168">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="0e937-169">다음 이미지는 이전 이미지와 동일한 기간에 스레드 프로파일링을 사용하여 캡처한 데이터입니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-169">The next image shows data captured using thread profiling, over the same period corresponding as the previous image.</span></span> <span data-ttu-id="0e937-170">시스템은 소켓 연결을 여는 데 많은 시간을 소비하며 소켓 연결을 닫고 소켓 예외를 처리하는 데 더 많은 시간을 소비합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-170">The system spends a significant time opening socket connections, and even more time closing them and handling socket exceptions.</span></span>

![각 요청에 대해 HttpClient 개체의 새 인스턴스를 만드는 응용 프로그램 예제를 보여주는 New Relic 스레드 프로파일러][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a><span data-ttu-id="0e937-172">부하 테스트 수행</span><span class="sxs-lookup"><span data-stu-id="0e937-172">Performing load testing</span></span>

<span data-ttu-id="0e937-173">부하 테스트를 사용하여 사용자가 수행할만한 일반적인 작업을 시뮬레이션합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-173">Use load testing to simulate the typical operations that users might perform.</span></span> <span data-ttu-id="0e937-174">이렇게 하면 다양한 부하에서 리소스 소진으로 인해 시스템의 어느 부분이 저하되는지 식별하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-174">This can help to identify which parts of a system suffer from resource exhaustion under varying loads.</span></span> <span data-ttu-id="0e937-175">이러한 테스트는 프로덕션 시스템보다는 통제된 환경에서 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-175">Perform these tests in a controlled environment rather than the production system.</span></span> <span data-ttu-id="0e937-176">다음 그래프는 사용자 로드가 동시 사용자 100명으로 증가하면서 `NewHttpClientInstancePerRequest` 컨트롤러가 처리하는 요청 처리량을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-176">The following graph shows the throughput of requests handled by the `NewHttpClientInstancePerRequest` controller as the user load increases to 100 concurrent users.</span></span>

![각 요청에 대해 HttpClient 개체의 새 인스턴스를 만드는 응용 프로그램 예제의 처리량][throughput-new-HTTPClient-instance]

<span data-ttu-id="0e937-178">우선 워크로드가 증가하면서 초당 처리되는 요청의 볼륨이 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-178">At first, the volume of requests handled per second increases as the workload increases.</span></span> <span data-ttu-id="0e937-179">하지만 사용자가 약 30명이 되면 성공한 요청의 볼륨이 한도에 도달하고 시스템은 예외를 생성하기 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-179">At about 30 users, however, the volume of successful requests reaches a limit, and the system starts to generate exceptions.</span></span> <span data-ttu-id="0e937-180">그 이후로 예외 볼륨이 사용자 로드에 따라 점차 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-180">From then on, the volume of exceptions gradually increases with the user load.</span></span> 

<span data-ttu-id="0e937-181">부하 테스트는 이러한 장애를 HTTP 500(내부 서버) 오류로 보고했습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-181">The load test reported these failures as HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="0e937-182">원격 분석 데이터를 검토하면 `HttpClient` 개체가 점점 더 많이 만들어지면서 시스템에 소켓 리소스가 소진되어 이러한 오류가 발생했다는 것을 알 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-182">Reviewing the telemetry showed that these errors were caused by the system running out of socket resources, as more and more `HttpClient` objects were created.</span></span>

<span data-ttu-id="0e937-183">다음 그래프는 사용자 지정 `ExpensiveToCreateService` 개체를 생성하는 컨트롤러에 대한 유사한 테스트를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-183">The next graph shows a similar test for a controller that creates the custom `ExpensiveToCreateService` object.</span></span>

![각 요청에 대해 ExpensiveToCreateService의 새 인스턴스를 만드는 응용 프로그램 예제의 처리량][throughput-new-ExpensiveToCreateService-instance]

<span data-ttu-id="0e937-185">이번에는 컨트롤러가 예외를 생성하지 않고 처리량이 안정기에 접어든 반면 평균 응답 시간은 20배 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-185">This time, the controller does not generate any exceptions, but throughput still reaches a plateau, while the average response time increases by a factor of 20.</span></span> <span data-ttu-id="0e937-186">(이 그래프는 응답 시간과 처리량에 대해 로그 눈금 간격을 사용합니다.) 원격 분석 데이터에 따르면 `ExpensiveToCreateService`의 새 인스턴스를 만드는 것이 문제의 주요 원인입니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-186">(The graph uses a logarithmic scale for response time and throughput.) Telemetry showed that creating new instances of the `ExpensiveToCreateService` was the main cause of the problem.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="0e937-187">솔루션 구현 및 결과 확인</span><span class="sxs-lookup"><span data-stu-id="0e937-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="0e937-188">`GetProductAsync` 메서드를 전환하여 단일 `HttpClient` 인스턴스를 공유한 후 2번째 부하 테스트에서 성능이 향상되었습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-188">After switching the `GetProductAsync` method to share a single `HttpClient` instance, a second load test showed improved performance.</span></span> <span data-ttu-id="0e937-189">오류는 보고되지 않았고 시스템은 초당 최대 500건의 요청에 달하는 증가한 부하를 처리할 수 있었습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-189">No errors were reported, and the system was able to handle an increasing load of up to 500 requests per second.</span></span> <span data-ttu-id="0e937-190">평균 응답 시간은 이전 테스트와 비교하여 절반이 되었습니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-190">The average response time was cut in half, compared with the previous test.</span></span>

![각 요청에 대해 HttpClient 개체의 동일한 인스턴스를 다시 사용하는 응용 프로그램 예제의 처리량][throughput-single-HTTPClient-instance]

<span data-ttu-id="0e937-192">비교를 위해 다음 이미지는 스택 추적 원격 분석 데이터를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-192">For comparison, the following image shows the stack trace telemetry.</span></span> <span data-ttu-id="0e937-193">이번에는 시스템이 소켓을 열고 닫는 대신 실제 작업을 수행하는 데 대부분의 시간을 소비합니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-193">This time, the system spends most of its time performing real work, rather than opening and closing sockets.</span></span>

![모든 요청에 대해 HttpClient 개체의 단일 인스턴스를 만드는 응용 프로그램 예제를 보여주는 New Relic 스레드 프로파일러][thread-profiler-single-HTTPClient-instance]

<span data-ttu-id="0e937-195">다음 그래프는 `ExpensiveToCreateService` 개체의 공유 인스턴스를 사용하는 유사한 부하 테스트를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-195">The next graph shows a similar load test using a shared instance of the `ExpensiveToCreateService` object.</span></span> <span data-ttu-id="0e937-196">처리된 요청의 볼륨은 사용자 로드에 따라 증가하지만 평균 응답 시간은 낮게 유지됩니다.</span><span class="sxs-lookup"><span data-stu-id="0e937-196">Again, the volume of handled requests increases in line with the user load, while the average response time remains low.</span></span> 

![각 요청에 대해 HttpClient 개체의 동일한 인스턴스를 다시 사용하는 응용 프로그램 예제의 처리량][throughput-single-ExpensiveToCreateService-instance]



[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
