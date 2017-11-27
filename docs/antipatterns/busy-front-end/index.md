---
title: "붐비는 프런트 엔드 안태패턴"
description: "다수의 백그라운드 스레드에서 비동기 작업을 수행하면 다른 포그라운드 작업에 결핍이 발생할 수 있습니다."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: cedb80ddac5ceb1eb901455df3165993fd28a138
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="busy-front-end-antipattern"></a><span data-ttu-id="00597-103">붐비는 프런트 엔드 안태패턴</span><span class="sxs-lookup"><span data-stu-id="00597-103">Busy Front End antipattern</span></span>

<span data-ttu-id="00597-104">다수의 백그라운드 스레드에서 비동기 작업을 수행하면 리소스의 다른 동시 포그라운드 작업에 결핍이 발생하여 응답 시간이 허용할 수 없는 수준으로 감소됩니다.</span><span class="sxs-lookup"><span data-stu-id="00597-104">Performing asynchronous work on a large number of background threads can starve other concurrent foreground tasks of resources, decreasing response times to unacceptable levels.</span></span>

## <a name="problem-description"></a><span data-ttu-id="00597-105">문제 설명</span><span class="sxs-lookup"><span data-stu-id="00597-105">Problem description</span></span>

<span data-ttu-id="00597-106">리소스를 많이 사용하는 작업은 사용자 요청에 대한 응답 시간을 늘리고 긴 대기 시간을 유발할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-106">Resource-intensive tasks can increase the response times for user requests and cause high latency.</span></span> <span data-ttu-id="00597-107">응답 시간을 향상시키는 한 가지 방법은 리소스를 많이 사용하는 작업을 별도의 스레드에 오프로드하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-107">One way to improve response times is to offload a resource-intensive task to a separate thread.</span></span> <span data-ttu-id="00597-108">이 방법으로 백그라운드에서 처리가 진행되는 동안 응용 프로그램의 응답 속도를 유지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-108">This approach lets the application stay responsive while processing happens in the background.</span></span> <span data-ttu-id="00597-109">하지만 백그라운드 스레드에서 실행되는 작업은 리소스를 여전히 소비합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-109">However, tasks that run on a background thread still consume resources.</span></span> <span data-ttu-id="00597-110">이 수가 너무 많으면 요청을 처리하는 스레드에 결핍이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-110">If there are too many of them, they can starve the threads that are handling requests.</span></span>

> [!NOTE]
> <span data-ttu-id="00597-111">*리소스*라는 용어는 CPU 사용률, 메모리 점유율 및 네트워크나 디스크 I/O와 같은 많은 요소를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-111">The term *resource* can encompass many things, such as CPU utilization, memory occupancy, and network or disk I/O.</span></span>

<span data-ttu-id="00597-112">일반적으로 이런 문제는 응용 프로그램이 모놀리식 코드 조각으로 개발되고 모든 비즈니스 논리가 프레젠테이션 계층과 공유되는 단일 계층으로 결합되는 경우에 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-112">This problem typically occurs when an application is developed as monolithic piece of code, with all of the business logic combined into a single tier shared with the presentation layer.</span></span>

<span data-ttu-id="00597-113">다음은 문제를 보여주는 ASP.NET을 사용한 예제입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-113">Here’s an example using ASP.NET that demonstrates the problem.</span></span> <span data-ttu-id="00597-114">전체 샘플은 [여기][code-sample]에서 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-114">You can find the complete sample [here][code-sample].</span></span>

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- <span data-ttu-id="00597-115">`WorkInFrontEnd` 컨트롤러의 `Post` 메서드는 HTTP POST 작업을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-115">The `Post` method in the `WorkInFrontEnd` controller implements an HTTP POST operation.</span></span> <span data-ttu-id="00597-116">이 작업은 CPU를 많이 사용하는 장기 실행 작업을 시뮬레이션합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-116">This operation simulates a long-running, CPU-intensive task.</span></span> <span data-ttu-id="00597-117">이 작업은 POST 작업을 신속하게 완료할 수 있도록 별도의 스레드에서 수행됩니다.</span><span class="sxs-lookup"><span data-stu-id="00597-117">The work is performed on a separate thread, in an attempt to enable the POST operation to complete quickly.</span></span>

- <span data-ttu-id="00597-118">`UserProfile` 컨트롤러의 `Get` 메서드는 HTTP GET 작업을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-118">The `Get` method in the `UserProfile` controller implements an HTTP GET operation.</span></span> <span data-ttu-id="00597-119">이 메서드는 훨씬 적은 CPU를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-119">This method is much less CPU intensive.</span></span>

<span data-ttu-id="00597-120">주요 사안은 `Post` 메서드의 리소스 요구 사항입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-120">The primary concern is the resource requirements of the `Post` method.</span></span> <span data-ttu-id="00597-121">작업이 백그라운드 스레드에 배치되더라도 작업은 여전히 상당한 CPU 리소스를 소비합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-121">Although it puts the work onto a background thread, the work can still consume considerable CPU resources.</span></span> <span data-ttu-id="00597-122">리소스는 다른 동시 사용자가 수행하는 다른 작업과 공유됩니다.</span><span class="sxs-lookup"><span data-stu-id="00597-122">These resources are shared with other operations being performed by other concurrent users.</span></span> <span data-ttu-id="00597-123">적당한 수의 사용자가 이 요청을 동시에 보내면 전반적인 성능이 저하되어 모든 작업이 느려질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-123">If a moderate number of users send this request at the same time, overall performance is likely to suffer, slowing down all operations.</span></span> <span data-ttu-id="00597-124">예를 들면, `Get` 메서드에서 상당한 대기 시간이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-124">Users might experience significant latency in the `Get` method, for example.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="00597-125">문제를 해결하는 방법</span><span class="sxs-lookup"><span data-stu-id="00597-125">How to fix the problem</span></span>

<span data-ttu-id="00597-126">상당한 리소스를 소비하는 프로세스를 별도의 백엔드로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-126">Move processes that consume significant resources to a separate back end.</span></span> 

<span data-ttu-id="00597-127">이러한 방식을 통해 프런트 엔드는 리소스를 많이 사용하는 작업을 메시지 큐에 넣습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-127">With this approach, the front end puts resource-intensive tasks onto a message queue.</span></span> <span data-ttu-id="00597-128">백 엔드는 비동기 처리를 위한 작업을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-128">The back end picks up the tasks for asynchronous processing.</span></span> <span data-ttu-id="00597-129">큐는 로드 레벨러(leveler)로 작동하여 백 엔드에 대한 요청을 버퍼링합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-129">The queue also acts as a load leveler, buffering requests for the back end.</span></span> <span data-ttu-id="00597-130">큐 길이가 너무 길어지면 자동 크기 조정을 구성하여 백 엔드 규모를 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-130">If the queue length becomes too long, you can configure autoscaling to scale out the back end.</span></span>

<span data-ttu-id="00597-131">다음은 이전 코드의 수정된 버전입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-131">Here is a revised version of the previous code.</span></span> <span data-ttu-id="00597-132">이 버전에서 `Post` 메서드는 메시지를 Service Bus 큐에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-132">In this version, the `Post` method puts a message on a Service Bus queue.</span></span> 

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

<span data-ttu-id="00597-133">백 엔드는 Service Bus 큐에서 메시지를 끌어와서 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-133">The back end pulls messages from the Service Bus queue and does the processing.</span></span>

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a><span data-ttu-id="00597-134">고려 사항</span><span class="sxs-lookup"><span data-stu-id="00597-134">Considerations</span></span>

- <span data-ttu-id="00597-135">이 방식은 응용 프로그램에 몇 가지 복잡성을 더합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-135">This approach adds some additional complexity to the application.</span></span> <span data-ttu-id="00597-136">오류 발생시 요청이 손실되지 않도록 큐 생성 및 큐 해제를 안전하게 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-136">You must handle queuing and dequeuing safely to avoid losing requests in the event of a failure.</span></span>
- <span data-ttu-id="00597-137">응용 프로그램은 메시지 큐에 대한 추가 서비스에 종속됩니다.</span><span class="sxs-lookup"><span data-stu-id="00597-137">The application takes a dependency on an additional service for the message queue.</span></span>
- <span data-ttu-id="00597-138">처리 환경은 예상 워크로드를 처리하고 필요한 처리량 목표를 충족할 만큼 충분히 확장성이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-138">The processing environment must be sufficiently scalable to handle the expected workload and meet the required throughput targets.</span></span>
- <span data-ttu-id="00597-139">이 방식은 전반적인 응답성을 개선하는 하지만 백 엔드로 이동되는 작업은 완료하는 데 시간이 더 오래 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-139">While this approach should improve overall responsiveness, the tasks that are moved to the back end may take longer to complete.</span></span> 

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="00597-140">문제를 감지하는 방법</span><span class="sxs-lookup"><span data-stu-id="00597-140">How to detect the problem</span></span>

<span data-ttu-id="00597-141">붐비는 프런트 엔드의 증상에는 리소스를 많이 사용하는 작업을 수행할 때 대기 시간이 길어지는 것이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="00597-141">Symptoms of a busy front end include high latency when resource-intensive tasks are being performed.</span></span> <span data-ttu-id="00597-142">응답 시간이 길어지거나 서비스 시간 초과로 인해 장애가 발생하면 최종 사용자가 보고할 가능성이 높습니다. 이러한 장애는 HTTP 500(내부 서버) 오류 또는 HTTP 503(서비스를 사용할 수 없음) 오류를 반환할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-142">End users are likely to report extended response times or failures caused by services timing out. These failures could also return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="00597-143">웹 서버의 이벤트 로그를 검사하십시오. 오류의 원인과 상황에 대한 자세한 정보가 포함되어 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-143">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="00597-144">다음 단계를 수행하면 문제를 식별하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-144">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="00597-145">프로덕션 시스템의 프로세스 모니터링을 수행하여 응답 시간이 느려지는 시점을 파악합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-145">Perform process monitoring of the production system, to identify points when response times slow down.</span></span>
2. <span data-ttu-id="00597-146">이 시점에 캡처된 원격 분석 데이터를 검사하여 수행되는 작업과 사용되는 리소스의 혼합을 결정합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-146">Examine the telemetry data captured at these points to determine the mix of operations being performed and the resources being used.</span></span> 
3. <span data-ttu-id="00597-147">당시 수행되었던 작업의 볼륨 및 조합과 열악한 응답 시간 사이의 상관 관계를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-147">Find any correlations between poor response times and the volumes and combinations of operations that were happening at those times.</span></span>
4. <span data-ttu-id="00597-148">의심되는 각 작업의 부하를 테스트하여 어떤 작업이 리소스를 소비하면서 다른 작업에 결핍을 유발하는지 파악합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-148">Load test each suspected operation to identify which operations are consuming resources and starving other operations.</span></span> 
5. <span data-ttu-id="00597-149">이러한 작업의 소스 코드를 검토하여 과도한 리소스 소비를 유발하는 원인을 알아냅니다.</span><span class="sxs-lookup"><span data-stu-id="00597-149">Review the source code for those operations to determine why they might cause excessive resource consumption.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="00597-150">예제 진단</span><span class="sxs-lookup"><span data-stu-id="00597-150">Example diagnosis</span></span> 

<span data-ttu-id="00597-151">다음 섹션에서는 이러한 단계를 앞에서 설명한 응용 프로그램 예제에 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-151">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slowdown"></a><span data-ttu-id="00597-152">느려지는 지점 파악</span><span class="sxs-lookup"><span data-stu-id="00597-152">Identify points of slowdown</span></span>

<span data-ttu-id="00597-153">각 요청이 소비하는 기간 및 자원을 추적하기 위해 각 메서드를 계측합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-153">Instrument each method to track the duration and resources consumed by each request.</span></span> <span data-ttu-id="00597-154">그런 다음 프로덕션 환경에서 응용 프로그램을 모니터링합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-154">Then monitor the application in production.</span></span> <span data-ttu-id="00597-155">요청이 서로 경쟁하는 방식을 전반적으로 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-155">This can provide an overall view of how requests compete with each other.</span></span> <span data-ttu-id="00597-156">스트레스 상황에서 느리게 실행되는 리소스가 부족한 요청은 다른 작업에 영향을 미칠 수 있으며 이러한 동작은 시스템을 모니터링하고 성능 저하에 주목함으로써 관찰할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-156">During periods of stress, slow-running resource-hungry requests will likely impact other operations, and this behavior can be observed by monitoring the system and noting the drop off in performance.</span></span>

<span data-ttu-id="00597-157">다음 이미지는 모니터링 대시보드입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-157">The following image shows a monitoring dashboard.</span></span> <span data-ttu-id="00597-158">(테스트에는 [AppDynamics]가 사용되었습니다.) 초기에는 시스템 부하가 적습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-158">(We used [AppDynamics] for our tests.) Initially, the system has light load.</span></span> <span data-ttu-id="00597-159">그런 다음 사용자가 `UserProfile` GET 메서드를 요청하기 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-159">Then users start requesting the `UserProfile` GET method.</span></span> <span data-ttu-id="00597-160">성능은 다른 사용자가 `WorkInFrontEnd` POST 메서드에 대한 요청 발급을 시작할 때가지 상당히 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-160">The performance is reasonably good until other users start issuing requests to the `WorkInFrontEnd` POST method.</span></span> <span data-ttu-id="00597-161">이 시점에서 응답 시간이 크게 증가합니다(첫 번째 화살표).</span><span class="sxs-lookup"><span data-stu-id="00597-161">At that point, response times increase dramatically (first arrow).</span></span> <span data-ttu-id="00597-162">응답 시간은 `WorkInFrontEnd` 컨트롤러에 대한 요청 볼륨이 줄어든 후에야 증가합니다(두 번째 화살표).</span><span class="sxs-lookup"><span data-stu-id="00597-162">Response times only improve after the volume of requests to the `WorkInFrontEnd` controller diminishes (second arrow).</span></span>

![WorkInFrontEnd 컨트롤러가 사용될 때 모든 요청의 응답 시간 효과를 보여주는 AppDynamics Business Transactions 창][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="00597-164">원격 분석 데이터 검사 및 상관 관계 찾기</span><span class="sxs-lookup"><span data-stu-id="00597-164">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="00597-165">다음 이미지는 동일한 간격으로 리소스 사용률을 모니터링하기 위해 모은 일부 메트릭입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-165">The next image shows some of the metrics gathered to monitor resource utilization during the same interval.</span></span> <span data-ttu-id="00597-166">처음에는 시스템에 액세스하는 사용자가 거의 없습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-166">At first, few users are accessing the system.</span></span> <span data-ttu-id="00597-167">연결하는 사용자가 많아지면서 CPU 사용률이 매우 높아집니다(100 %).</span><span class="sxs-lookup"><span data-stu-id="00597-167">As more users connect, CPU utilization becomes very high (100%).</span></span> <span data-ttu-id="00597-168">또한 CPU 사용량이 증가함에 따라 네트워크 I/O 속도가 처음에는 올라갑니다.</span><span class="sxs-lookup"><span data-stu-id="00597-168">Also notice that the network I/O rate initially goes up as CPU usage rises.</span></span> <span data-ttu-id="00597-169">하지만 CPU 사용량이 최고가 되면 네트워크 I/O가 실제로 내려갑니다.</span><span class="sxs-lookup"><span data-stu-id="00597-169">But once CPU usage peaks, network I/O actually goes down.</span></span> <span data-ttu-id="00597-170">CPU가 용량에 도달하면 시스템이 비교적 적은 수의 요청만 처리할 수 있기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-170">That's because the system can only handle a relatively small number of requests once the CPU is at capacity.</span></span> <span data-ttu-id="00597-171">사용자가 연결을 끊으면 CPU 로드가 끝납니다.</span><span class="sxs-lookup"><span data-stu-id="00597-171">As users disconnect, the CPU load tails off.</span></span>

![CPU 및 네트워크 사용률을 보여주는 AppDynamics 메트릭][AppDynamics-Metrics-Front-End-Requests]

<span data-ttu-id="00597-173">이 시점에서는 `WorkInFrontEnd` 컨트롤러의 `Post` 메서드가 정밀 조사의 주요 후보입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-173">At this point, it appears the `Post` method in the `WorkInFrontEnd` controller is a prime candidate for closer examination.</span></span> <span data-ttu-id="00597-174">가설을 확인하려면 통제된 환경에서 추가 작업이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-174">Further work in a controlled environment is needed to confirm the hypothesis.</span></span>

### <a name="perform-load-testing"></a><span data-ttu-id="00597-175">부하 테스트 수행</span><span class="sxs-lookup"><span data-stu-id="00597-175">Perform load testing</span></span> 

<span data-ttu-id="00597-176">다음 단계는 통제된 환경에서 테스트를 수행하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-176">The next step is to perform tests in a controlled environment.</span></span> <span data-ttu-id="00597-177">예를 들어, 요청을 포함하는 일련의 부하 테스트를 실행한 다음 각 요청을 차례로 생략하면서 영향을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-177">For example, run a series of load tests that include and then omit each request in turn to see the effects.</span></span>

<span data-ttu-id="00597-178">아래 그래프는 이전 테스트에 사용한 클라우드 서비스의 동일한 배치에 수행된 부하 테스트의 결과입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-178">The graph below shows the results of a load test performed against an identical deployment of the cloud service used in the previous tests.</span></span> <span data-ttu-id="00597-179">이 테스트에는 `WorkInFrontEnd` 컨트롤러에서 `Post` 작업을 수행하는 사용자의 단계 부하와 함께 `UserProfile` 컨트롤러에서 `Get` 작업을 수행하는 500명의 일정 부하를 사용했습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-179">The test used a constant load of 500 users performing the `Get` operation in the `UserProfile` controller, along with a step load of users performing the `Post` operation in the `WorkInFrontEnd` controller.</span></span> 

![WorkInFrontEnd 컨트롤러의 초기 부하 테스트 결과][Initial-Load-Test-Results-Front-End]

<span data-ttu-id="00597-181">처음에는 단계 로드가 0이므로 활성 사용자만 `UserProfile` 요청을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-181">Initially, the step load is 0, so the only active users are performing the `UserProfile` requests.</span></span> <span data-ttu-id="00597-182">시스템은 초당 약 500개 요청에 응답할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-182">The system is able to respond to approximately 500 requests per second.</span></span> <span data-ttu-id="00597-183">60초 후, 100명의 추가 사용자가 `WorkInFrontEnd` 컨트롤러에 POST 요청을 보내기 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-183">After 60 seconds, a load of 100 additional users starts sending POST requests to the `WorkInFrontEnd` controller.</span></span> <span data-ttu-id="00597-184">거의 즉시, `UserProfile` 컨트롤러에 전송된 워크로드가 초당 약 150개 요청으로 떨어집니다.</span><span class="sxs-lookup"><span data-stu-id="00597-184">Almost immediately, the workload sent to the `UserProfile` controller drops to about 150 requests per second.</span></span> <span data-ttu-id="00597-185">이것은 부하 테스트 러너(test runner)의 작동 방식 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-185">This is due to the way the load-test runner functions.</span></span> <span data-ttu-id="00597-186">다음 요청을 보내기 전에 응답을 기다리기 때문에 응답을 받는 시간이 오래 걸릴수록 요청 속도가 느려집니다.</span><span class="sxs-lookup"><span data-stu-id="00597-186">It waits for a response before sending the next request, so the longer it takes to receive a response, the lower the request rate.</span></span>

<span data-ttu-id="00597-187">`WorkInFrontEnd` 컨트롤러에 POST 요청을 보내는 사용자가 많아질수록 `UserProfile` 컨트롤러의 응답 속도는 계속 떨어집니다.</span><span class="sxs-lookup"><span data-stu-id="00597-187">As more users send POST requests to the `WorkInFrontEnd` controller, the response rate of the `UserProfile` controller continues to drop.</span></span> <span data-ttu-id="00597-188">하지만 `WorkInFrontEnd` 컨트롤러가 처리하는 요청의 볼륨은 비교적 일정하게 유지됩니다.</span><span class="sxs-lookup"><span data-stu-id="00597-188">But note that the volume of requests handled by the `WorkInFrontEnd`controller remains relatively constant.</span></span> <span data-ttu-id="00597-189">두 요청의 전반적인 속도가 꾸준히 낮은 한도로 향하기 때문에 시스템의 포화 상태가 명백해집니다.</span><span class="sxs-lookup"><span data-stu-id="00597-189">The saturation of the system becomes apparent as the overall rate of both requests tends towards a steady but low limit.</span></span>


### <a name="review-the-source-code"></a><span data-ttu-id="00597-190">소스 코드 검토</span><span class="sxs-lookup"><span data-stu-id="00597-190">Review the source code</span></span>

<span data-ttu-id="00597-191">마지막 단계에서는 소스 코드를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-191">The final step is to look at the source code.</span></span> <span data-ttu-id="00597-192">`Post` 메서드에 상당한 시간이 걸릴 수 있다는 것을 개발 팀이 알고 있었기 때문에 원래 구현에 별도의 스레드가 사용되었습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-192">The development team was aware that the `Post` method could take a considerable amount of time, which is why the original implementation used a separate thread.</span></span> <span data-ttu-id="00597-193">이로 인해 `Post` 메서드가 장기 실행 작업이 완료되기를 기다리는 것을 차단하지 않았기 때문에 즉각적인 문제는 해결되었습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-193">That solved the immediate problem, because the `Post` method did not block waiting for a long-running task to complete.</span></span>

<span data-ttu-id="00597-194">하지만 이 메서드로 수행되는 작업은 여전히 CPU, 메모리 및 기타 리소스를 소비합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-194">However, the work performed by this method still consumes CPU, memory, and other resources.</span></span> <span data-ttu-id="00597-195">이 프로세스를 비동기적으로 실행하면 통제되지 않은 방식으로 사용자가 동시에 많은 수의 작업을 트리거할 수 있기 때문에 실제로 성능이 저하될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-195">Enabling this process to run asynchronously might actually damage performance, as users can trigger a large number of these operations simultaneously, in an uncontrolled manner.</span></span> <span data-ttu-id="00597-196">서버가 실행할 수 있는 스레드 수에는 한도가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-196">There is a limit to the number of threads that a server can run.</span></span> <span data-ttu-id="00597-197">이 한도를 넘으면 새 스레드를 시작하려고 할 때 응용 프로그램에 예외가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-197">Past this limit, the application is likely to get an exception when it tries to start a new thread.</span></span>

> [!NOTE]
> <span data-ttu-id="00597-198">이것이 비동기 작업을 피해야 한다는 의미는 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="00597-198">This doesn't mean you should avoid asynchronous operations.</span></span> <span data-ttu-id="00597-199">권장되는 방법은 네트워크 호출에 비동기 await를 수행하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-199">Performing an asynchronous await on a network call is a recommended practice.</span></span> <span data-ttu-id="00597-200">([동기 I/O][sync-io] 안태패턴을 참조하세요.) 여기서 문제는 다른 스레드에서 CPU를 많이 사용하는 작업이 생성된다는 점입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-200">(See the [Synchronous I/O][sync-io] antipattern.) The problem here is that CPU-intensive work was spawned on another thread.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="00597-201">솔루션 구현 및 결과 확인</span><span class="sxs-lookup"><span data-stu-id="00597-201">Implement the solution and verify the result</span></span>

<span data-ttu-id="00597-202">다음 이미지는 솔루션을 구현한 후 성능 모니터링입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-202">The following image shows performance monitoring after the solution was implemented.</span></span> <span data-ttu-id="00597-203">로드는 앞에 표시된 것과 비슷하지만 `UserProfile` 컨트롤러의 응답 시간이 훨씬 빨라졌습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-203">The load was similar to that shown earlier, but the response times for the `UserProfile` controller are now much faster.</span></span> <span data-ttu-id="00597-204">요청 볼륨은 같은 기간 동안 2,759에서 23,565로 증가했습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-204">The volume of requests increased over the same duration, from 2,759 to 23,565.</span></span> 

![WorkInBackground 컨트롤러가 사용될 때 모든 요청의 응답 시간 효과를 보여주는 AppDynamics Business Transactions 창][AppDynamics-Transactions-Background-Requests]

<span data-ttu-id="00597-206">`WorkInBackground` 컨트롤러 역시 훨씬 더 많은 요청 볼륨을 처리했습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-206">Note that the `WorkInBackground` controller also handled a much larger volume of requests.</span></span> <span data-ttu-id="00597-207">하지만 이 경우 컨트롤러에서 수행되는 작업은 원래 코드와 매우 다르기 때문에 직접 비교할 수는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-207">However, you can't make a direct comparison in this case, because the work being performed in this controller is very different from the original code.</span></span> <span data-ttu-id="00597-208">새 버전에서는 시간이 오래 걸리는 계산을 수행하는 것이 아니라 요청을 큐에 넣기만 합니다.</span><span class="sxs-lookup"><span data-stu-id="00597-208">The new version simply queues a request, rather than performing a time consuming calculation.</span></span> <span data-ttu-id="00597-209">관건은 이 메서드가 전체 시스템을 부하 상태로 끌어 내리지 않는다는 점입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-209">The main point is that this method no longer drags down the entire system under load.</span></span>

<span data-ttu-id="00597-210">CPU 및 네트워크 사용률 역시 향상된 성능을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="00597-210">CPU and network utilization also show the improved performance.</span></span> <span data-ttu-id="00597-211">CPU 사용률이 100%에 도달하지 않았고 처리된 네트워크 요청의 볼륨이 이전보다 훨씬 커졌으며 작업 부하가 떨어질 때까지 감소되지 않았습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-211">The CPU utilization never reached 100%, and the volume of handled network requests was far greater than earlier, and did not tail off until the workload dropped.</span></span>

![WorkInBackground 컨트롤러의 CPU 및 네트워크 사용률을 보여주는 AppDynamics 메트릭][AppDynamics-Metrics-Background-Requests]

<span data-ttu-id="00597-213">다음 그래프는 부하 테스트의 결과입니다.</span><span class="sxs-lookup"><span data-stu-id="00597-213">The following graph shows the results of a load test.</span></span> <span data-ttu-id="00597-214">이전 테스트와 비교하여 처리된 요청의 전체적인 양이 크게 향상되었습니다.</span><span class="sxs-lookup"><span data-stu-id="00597-214">The overall volume of requests serviced is greatly improved compared to the the earlier tests.</span></span>

![BackgroundImageProcessing 컨트롤러의 부하 테스트 결과][Load-Test-Results-Background]

## <a name="related-guidance"></a><span data-ttu-id="00597-216">관련 지침</span><span class="sxs-lookup"><span data-stu-id="00597-216">Related guidance</span></span>

- <span data-ttu-id="00597-217">[자동 크기 조정 모범 사례][autoscaling]</span><span class="sxs-lookup"><span data-stu-id="00597-217">[Autoscaling best practices][autoscaling]</span></span>
- <span data-ttu-id="00597-218">[백그라운드 작업 모범 사례][background-jobs]</span><span class="sxs-lookup"><span data-stu-id="00597-218">[Background jobs best practices][background-jobs]</span></span>
- <span data-ttu-id="00597-219">[큐 기반 부하 평준화 패턴][load-leveling]</span><span class="sxs-lookup"><span data-stu-id="00597-219">[Queue-Based Load Leveling pattern][load-leveling]</span></span>
- <span data-ttu-id="00597-220">[웹 큐 작업자 아키텍처 스타일][web-queue-worker]</span><span class="sxs-lookup"><span data-stu-id="00597-220">[Web Queue Worker architecture style][web-queue-worker]</span></span>

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg


