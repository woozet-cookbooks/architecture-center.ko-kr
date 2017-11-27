---
title: "동기 I/O 안티패턴"
description: "I/O가 완료하는 동안 호출 스레드를 차단하면 성능이 감소하고 수직 확장성에 영향을 미칠 수 있습니다."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: d5b3635565c6b71ef7716f54ee8cccc76093c3a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="synchronous-io-antipattern"></a><span data-ttu-id="06535-103">동기 I/O 안티패턴</span><span class="sxs-lookup"><span data-stu-id="06535-103">Synchronous I/O antipattern</span></span>

<span data-ttu-id="06535-104">I/O가 완료하는 동안 호출 스레드를 차단하면 성능이 감소하고 수직 확장성에 영향을 미칠 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-104">Blocking the calling thread while I/O completes can reduce performance and affect vertical scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="06535-105">문제 설명</span><span class="sxs-lookup"><span data-stu-id="06535-105">Problem description</span></span>

<span data-ttu-id="06535-106">동기 I/O 작업은 I/O가 완료하는 동안 호출 스레드를 차단합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-106">A synchronous I/O operation blocks the calling thread while the I/O completes.</span></span> <span data-ttu-id="06535-107">호출 스레드는 대기 상태로 전환하며 이 간격 동안 유용한 작업을 수행할 수 없어 처리 리소스를 낭비합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-107">The calling thread enters a wait state and is unable to perform useful work during this interval, wasting processing resources.</span></span>

<span data-ttu-id="06535-108">I/O의 일반적인 예제는 다음을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-108">Common examples of I/O include:</span></span>

- <span data-ttu-id="06535-109">데이터 검색 또는 데이터베이스나 임의 유형의 영구 저장소에 데이터 유지.</span><span class="sxs-lookup"><span data-stu-id="06535-109">Retrieving or persisting data to a database or any type of persistent storage.</span></span>
- <span data-ttu-id="06535-110">웹 서비스에 요청 보내기.</span><span class="sxs-lookup"><span data-stu-id="06535-110">Sending a request to a web service.</span></span>
- <span data-ttu-id="06535-111">메시지 게시 또는 큐에서 메시지 검색.</span><span class="sxs-lookup"><span data-stu-id="06535-111">Posting a message or retrieving a message from a queue.</span></span>
- <span data-ttu-id="06535-112">로컬 파일에 쓰기 또는 로컬 파일에서 읽기.</span><span class="sxs-lookup"><span data-stu-id="06535-112">Writing to or reading from a local file.</span></span>

<span data-ttu-id="06535-113">이런 안티패턴이 발생하는 일반적인 이유는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-113">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="06535-114">그러한 패턴이 작업을 수행하는 가장 직관적인 방법처럼 보입니다.</span><span class="sxs-lookup"><span data-stu-id="06535-114">It appears to be the most intuitive way to perform an operation.</span></span> 
- <span data-ttu-id="06535-115">응용 프로그램이 요청에서 응답을 요구합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-115">The application requires a response from a request.</span></span>
- <span data-ttu-id="06535-116">응용 프로그램이 동기 I/O 방법만 제공하는 라이브러리를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-116">The application uses a library that only provides synchronous methods for I/O.</span></span> 
- <span data-ttu-id="06535-117">외부 라이브러리가 동기 I/O 작업을 내부적으로 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-117">An external library performs synchronous I/O operations internally.</span></span> <span data-ttu-id="06535-118">단일 동기 I/O 호출이 전체 호출 체인을 차단할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-118">A single synchronous I/O call can block an entire call chain.</span></span>

<span data-ttu-id="06535-119">다음 코드는 Azure Blob Storage에 파일을 업로드합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-119">The following code uploads a file to Azure blob storage.</span></span> <span data-ttu-id="06535-120">코드 블록이 동기 I/O를 위해 대기하는 두 장소, `CreateIfNotExists` 메서드 및 `UploadFromStream` 메서드가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-120">There are two places where the code blocks waiting for synchronous I/O, the `CreateIfNotExists` method and the `UploadFromStream` method.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

<span data-ttu-id="06535-121">다음은 외부 서비스에서의 응답을 위해 대기하는 예제입니다.</span><span class="sxs-lookup"><span data-stu-id="06535-121">Here's an example of waiting for a response from an external service.</span></span> <span data-ttu-id="06535-122">`GetUserProfile` 메서드는 `UserProfile`을 반환하는 원격 서비스를 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-122">The `GetUserProfile` method calls a remote service that returns a `UserProfile`.</span></span>

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

<span data-ttu-id="06535-123">이 두 예제에 대한 전체 코드는 [여기][sample-app]서 찾아볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-123">You can find the complete code for both of these examples [here][sample-app].</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="06535-124">문제를 해결하는 방법</span><span class="sxs-lookup"><span data-stu-id="06535-124">How to fix the problem</span></span>

<span data-ttu-id="06535-125">동기 I/O 작업을 비동기 작업으로 바꿉니다.</span><span class="sxs-lookup"><span data-stu-id="06535-125">Replace synchronous I/O operations with asynchronous operations.</span></span> <span data-ttu-id="06535-126">그러면 현재 스레드가 해제되어 차단되지 않고 의미 있는 작업을 계속 수행할 수 있어 계산 리소스 사용률이 개선됩니다.</span><span class="sxs-lookup"><span data-stu-id="06535-126">This frees the current thread to continue performing meaningful work rather than blocking, and helps improve the utilization of compute resources.</span></span> <span data-ttu-id="06535-127">I/O를 비동기로 수행하는 방법은 클라이언트 응용 프로그램에서의 예기치 않은 요청 급증을 처리할 때 특히 효율적입니다.</span><span class="sxs-lookup"><span data-stu-id="06535-127">Performing I/O asynchronously is particularly efficient for handling an unexpected surge in requests from client applications.</span></span> 

<span data-ttu-id="06535-128">많은 라이브러리는 메서드의 동기 버전과 비동기 버전을 모두 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-128">Many libraries provide both synchronous and asynchronous versions of methods.</span></span> <span data-ttu-id="06535-129">가능하면 언제나 비동기 버전을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-129">Whenever possible, use the asynchronous versions.</span></span> <span data-ttu-id="06535-130">다음은 Azure Blob Storage에 파일을 업로드하는 앞의 예의 비동기 버전입니다.</span><span class="sxs-lookup"><span data-stu-id="06535-130">Here is the asynchronous version of the previous example that uploads a file to Azure blob storage.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

<span data-ttu-id="06535-131">`await` 연산자는 비동기 작업이 수행되는 동안 제어권을 호출 환경에 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-131">The `await` operator returns control to the calling environment while the asynchronous operation is performed.</span></span> <span data-ttu-id="06535-132">이 명령문 뒤의 코드는 비동기 작업이 완료되었을 때 실행하는 연속의 역할을 합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-132">The code after this statement acts as a continuation that runs when the asynchronous operation has completed.</span></span>

<span data-ttu-id="06535-133">잘 설계된 서비스는 비동기 작업도 제공해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-133">A well designed service should also provide asynchronous operations.</span></span> <span data-ttu-id="06535-134">다음은 사용자 프로필을 반환하는 웹 서비스의 비동기 버전입니다.</span><span class="sxs-lookup"><span data-stu-id="06535-134">Here is an asynchronous version of the web service that returns user profiles.</span></span> <span data-ttu-id="06535-135">`GetUserProfileAsync` 메서드를 사용하려면 사용자 프로필 서비스의 비동기 버전이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-135">The `GetUserProfileAsync` method depends on having an asynchronous version of the User Profile service.</span></span>

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

<span data-ttu-id="06535-136">작업의 비동기 버전을 제공하지 않는 버전의 경우 선택한 동기 방법의 주위에 비동기 래퍼를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-136">For libraries that don't provide asynchronous versions of operations, it may be possible to create asynchronous wrappers around selected synchronous methods.</span></span> <span data-ttu-id="06535-137">이 접근법은 주의해서 따라야 합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-137">Follow this approach with caution.</span></span> <span data-ttu-id="06535-138">이 접근법은 비동기 래퍼를 호출하는 스레드 응답성을 개선할 수 있지만 실제로는 더 많은 리소스를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-138">While it may improve responsiveness on the thread that invokes the asynchronous wrapper, it actually consumes more resources.</span></span> <span data-ttu-id="06535-139">추가 스레드가 만들어질 수 있으며 이 스레드가 수행한 작업의 동기화와 연결된 오버헤드가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-139">An extra thread may be created, and there is overhead associated with synchronizing the work done by this thread.</span></span> <span data-ttu-id="06535-140">다음 블로그 게시물에서 몇 가지 절충안을 설명합니다. [동기 메서드에 대해 비동기 래퍼를 노출해야 하나요?][async-wrappers]</span><span class="sxs-lookup"><span data-stu-id="06535-140">Some tradeoffs are discussed in this blog post: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]</span></span>

<span data-ttu-id="06535-141">다음은 동기 메서드 주위의 비동기 래퍼의 예제입니다.</span><span class="sxs-lookup"><span data-stu-id="06535-141">Here is an example of an asynchronous wrapper around a synchronous method.</span></span>

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

<span data-ttu-id="06535-142">이제 호출 코드는 래퍼상에서 대기할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-142">Now the calling code can await on the wrapper:</span></span>

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a><span data-ttu-id="06535-143">고려 사항</span><span class="sxs-lookup"><span data-stu-id="06535-143">Considerations</span></span>

- <span data-ttu-id="06535-144">매우 수명이 짧아서 경합을 야기할 가능성이 낮은 I/O 작업은 비동기 작업으로서 더 효율적일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-144">I/O operations that are expected to be very short lived and are unlikely to cause contention might be more performant as synchronous operations.</span></span> <span data-ttu-id="06535-145">SSD 드라이브에 있는 작은 파일을 읽는 경우가 그러한 예일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-145">An example might be reading small files on an SSD drive.</span></span> <span data-ttu-id="06535-146">작업이 완료될 때 작업을 다른 스레드에 디스패치하고 해당 스레드와 동기화하는 오버헤드가 비동기 I/O의 이점보다 클 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-146">The overhead of dispatching a task to another thread, and synchronizing with that thread when the task completes, might outweigh the benefits of asynchronous I/O.</span></span> <span data-ttu-id="06535-147">그러나 이러한 경우는 비교적 드물며 대부분의 I/O 작업은 비동기로 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-147">However, these cases are relatively rare, and most I/O operations should be done asynchronously.</span></span>

- <span data-ttu-id="06535-148">I/O 성능이 개선되면 시스템의 다른 부분에 병목이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-148">Improving I/O performance may cause other parts of the system to become bottlenecks.</span></span> <span data-ttu-id="06535-149">예를 들어 스레드를 차단 해제하면 동시 공유 리소스 요청이 더 많아지고 그에 따라 리소스 부족 또는 제한이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-149">For example, unblocking threads might result in a higher volume of concurrent requests to shared resources, leading in turn to resource starvation or throttling.</span></span> <span data-ttu-id="06535-150">이러한 상황이 문제가 된다면 경합을 줄이기 위해 웹 서버 수를 규모 확장하거나 데이터 저장소를 분할해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-150">If that becomes a problem, you might need to scale out the number of web servers or partition data stores to reduce contention.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="06535-151">문제를 감지하는 방법</span><span class="sxs-lookup"><span data-stu-id="06535-151">How to detect the problem</span></span>

<span data-ttu-id="06535-152">사용자 입장에서는 응용 프로그램이 응답하지 않거나 주기적으로 중지하는 것으로 보일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-152">For users, the application may seem unresponsive or appear to hang periodically.</span></span> <span data-ttu-id="06535-153">응용 프로그램은 시간 초과 예외 때문에 실패할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-153">The application might fail with timeout exceptions.</span></span> <span data-ttu-id="06535-154">이러한 장애는 HTTP 500(내부 서버) 오류를 반환할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-154">These failures could also return HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="06535-155">서버 쪽에서는 들어오는 클라이언트 요청이 스레드가 사용할 수 있게 될 때까지 차단되어 요청 큐 길이가 너무 커질 수 있으며 이는 HTTP 503(서비스를 사용할 수 없음) 오류로 명시됩니다.</span><span class="sxs-lookup"><span data-stu-id="06535-155">On the server, incoming client requests might be blocked until a thread becomes available, resulting in excessive request queue lengths, manifested as HTTP 503 (Service Unavailable) errors.</span></span>

<span data-ttu-id="06535-156">다음 단계를 수행하면 문제를 식별하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-156">You can perform the following steps to help identify the problem:</span></span>

1. <span data-ttu-id="06535-157">프로덕션 시스템을 모니터링하여 차단된 작업자 스레드가 처리량을 제약하는지 여부를 결정합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-157">Monitor the production system and determine whether blocked worker threads are constraining throughput.</span></span>

2. <span data-ttu-id="06535-158">요청이 스레드 부족 때문에 차단되는 경우 응용 프로그램을 검토하여 I/O를 동기식으로 수행하는 작업이 어느 것인지 결정합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-158">If requests are being blocked due to lack of threads, review the application to determine which operations may be performing I/O synchronously.</span></span>

3. <span data-ttu-id="06535-159">동기 I/O를 수행 중인 각 작업에 대해 제어된 부하 테스트를 수행하여 해당 작업이 시스템 성능에 영향을 미치는지 여부를 알아냅니다.</span><span class="sxs-lookup"><span data-stu-id="06535-159">Perform controlled load testing of each operation that is performing synchronous I/O, to find out whether those operations are affecting system performance.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="06535-160">예제 진단</span><span class="sxs-lookup"><span data-stu-id="06535-160">Example diagnosis</span></span>

<span data-ttu-id="06535-161">다음 섹션에서는 이러한 단계를 앞에서 설명한 응용 프로그램 예제에 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-161">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-web-server-performance"></a><span data-ttu-id="06535-162">웹 서버 성능 모니터링</span><span class="sxs-lookup"><span data-stu-id="06535-162">Monitor web server performance</span></span>

<span data-ttu-id="06535-163">Azure 웹 응용 프로그램 및 웹 역할의 경우 IIS 웹 서버의 성능을 모니터링하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-163">For Azure web applications and web roles, it's worth monitoring the performance of the IIS web server.</span></span> <span data-ttu-id="06535-164">특히 요청 큐 길이에 주의를 기울여 작업량이 많은 기간 동안 요청이 차단되어 사용할 수 있는 스레드를 위해 대기하는지 여부를 확인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-164">In particular, pay attention to the request queue length to establish whether requests are being blocked waiting for available threads during periods of high activity.</span></span> <span data-ttu-id="06535-165">Azure 진단을 사용하도록 설정하면 이 정보를 수집할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-165">You can gather this information by enabling Azure diagnostics.</span></span> <span data-ttu-id="06535-166">자세한 내용은 다음을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="06535-166">For more information, see:</span></span>

- <span data-ttu-id="06535-167">[Azure App Service에서 앱 모니터링][web-sites-monitor]</span><span class="sxs-lookup"><span data-stu-id="06535-167">[Monitor Apps in Azure App Service][web-sites-monitor]</span></span>
- <span data-ttu-id="06535-168">[Azure 응용 프로그램에서 성능 카운터 만들기 및 사용][performance-counters]</span><span class="sxs-lookup"><span data-stu-id="06535-168">[Create and use performance counters in an Azure application][performance-counters]</span></span>

<span data-ttu-id="06535-169">응용 프로그램을 계측하여 요청이 수용된 후 처리되는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="06535-169">Instrument the application to see how requests are handled once they have been accepted.</span></span> <span data-ttu-id="06535-170">요청 흐름을 추적하면 느리게 실행되는 호출을 수행하여 현재 스레드를 차단하는지 여부를 확인하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-170">Tracing the flow of a request can help to identify whether it is performing slow-running calls and blocking the current thread.</span></span> <span data-ttu-id="06535-171">또한 스레드 프로파일링을 수행하면 차단된 요청을 강조 표시할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-171">Thread profiling can also highlight requests that are being blocked.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="06535-172">응용 프로그램 부하 테스트</span><span class="sxs-lookup"><span data-stu-id="06535-172">Load test the application</span></span>

<span data-ttu-id="06535-173">다음 그래프는 동기 사용자 수가 최대 4000명인 변화하는 부하에서 앞에서 본 동기 `GetUserProfile` 메서드의 성능을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="06535-173">The following graph shows the performance of the synchronous `GetUserProfile` method shown earlier, under varying loads of up to 4000 concurrent users.</span></span> <span data-ttu-id="06535-174">응용 프로그램은 Azure Cloud Service 웹 역할에서 실행 중인 ASP.NET 응용 프로그램입니다.</span><span class="sxs-lookup"><span data-stu-id="06535-174">The application is an ASP.NET application running in an Azure Cloud Service web role.</span></span>

![비동기 I/O 작업을 수행하는 샘플 응용 프로그램에 대한 성능 차트][sync-performance]

<span data-ttu-id="06535-176">동기 작업은 2초 동안 휴면하고 동기 I/O를 시뮬레이션하도록 하드 코드되었으므로 최소 응답 시간은 2초를 조금 넘습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-176">The synchronous operation is hard-coded to sleep for 2 seconds, to simulate synchronous I/O, so the minimum response time is slightly over 2 seconds.</span></span> <span data-ttu-id="06535-177">부하가 대략 동시 사용자 2500명에 도달하면 평균 응답 시간은 안정기에 접어들지만 초당 요청 분량은 계속 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-177">When the load reaches approximately 2500 concurrent users, the average response time reaches a plateau, although the volume of requests per second continues to increase.</span></span> <span data-ttu-id="06535-178">참고로 이 두 측정값에 대한 비율 크기 조정은 로그값입니다.</span><span class="sxs-lookup"><span data-stu-id="06535-178">Note that the scale for these two measures is logarithmic.</span></span> <span data-ttu-id="06535-179">초당 요청 수는 이 점과 테스트 종료 시점 간에 두 배가 됩니다.</span><span class="sxs-lookup"><span data-stu-id="06535-179">The number of requests per second doubles between this point and the end of the test.</span></span>

<span data-ttu-id="06535-180">분리 상태에서는 이 테스트에서 동기 I/O가 문제인지 여부를 반드시 명확히 알 수 있는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="06535-180">In isolation, it's not necessarily clear from this test whether the synchronous I/O is a problem.</span></span> <span data-ttu-id="06535-181">더 큰 부하에서 응용 프로그램은 웹 서버가 더 이상 요청을 적시에 처리할 수 없어 클라이언트 응용 프로그램이 시간 초과 예외를 수신하게 하는 티핑 포인트에 도달할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="06535-181">Under heavier load, the application may reach a tipping point where the web server can no longer process requests in a timely manner, causing client applications to receive time-out exceptions.</span></span>

<span data-ttu-id="06535-182">들어오는 요청은 IIS 웹 서버에 의해 큐에 저장되며 ASP.NET 스레드 풀에서 실행 중인 스레드로 넘겨집니다.</span><span class="sxs-lookup"><span data-stu-id="06535-182">Incoming requests are queued by the IIS web server and handed to a thread running in the ASP.NET thread pool.</span></span> <span data-ttu-id="06535-183">각 작업이 I/O를 동기식으로 수행하므로 스레드는 작업이 완료될 때까지 차단됩니다.</span><span class="sxs-lookup"><span data-stu-id="06535-183">Because each operation performs I/O synchronously, the thread is blocked until the operation completes.</span></span> <span data-ttu-id="06535-184">워크로드가 증가함에 따라 결국 스레드 풀의 모든 ASP.NET 스레드가 할당되고 차단됩니다.</span><span class="sxs-lookup"><span data-stu-id="06535-184">As the workload increases, eventually all of the ASP.NET threads in the thread pool are allocated and blocked.</span></span> <span data-ttu-id="06535-185">그 시점에서 추가로 들어오는 요청은 사용할 수 있는 스레드를 위해 큐에서 대기해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-185">At that point, any further incoming requests must wait in the queue for an available thread.</span></span> <span data-ttu-id="06535-186">큐 길이가 커짐에 따라 요청이 시간 초과하기 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-186">As the queue length grows, requests start to time out.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="06535-187">솔루션 구현 및 결과 확인</span><span class="sxs-lookup"><span data-stu-id="06535-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="06535-188">다음 그래프는 코드의 비동기 버전에 대한 부하 테스트에서 나온 결과를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="06535-188">The next graph shows the results from load testing the asynchronous version of the code.</span></span>

![비동기 I/O 작업을 수행하는 샘플 응용 프로그램에 대한 성능 차트][async-performance]

<span data-ttu-id="06535-190">처리량이 훨씬 더 큽니다.</span><span class="sxs-lookup"><span data-stu-id="06535-190">Throughput is far higher.</span></span> <span data-ttu-id="06535-191">초당 요청 수로 측정했을 때, 이전 테스트와 같은 기간 동안 시스템은 거의 10배 증가한 처리량을 성공적으로 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="06535-191">Over the same duration as the previous test, the system successfully handles a nearly tenfold increase in throughput, as measured in requests per second.</span></span> <span data-ttu-id="06535-192">또한 평균 응답 시간이 비교적 일정하며 이전 테스트보다 약 25배 더 작게 유지됩니다.</span><span class="sxs-lookup"><span data-stu-id="06535-192">Moreover, the average response time is relatively constant and remains approximately 25 times smaller than the previous test.</span></span>


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: http://blogs.msdn.com/b/pfxteam/archive/2012/03/24/10287244.aspx
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg



