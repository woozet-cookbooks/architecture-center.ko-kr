---
title: 게이트웨이 집계 패턴
description: 게이트웨이를 사용하여 여러 개별 요청을 단일 요청으로 집계합니다.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: f59c8b8b02c6db28024d13621b782997e63a4e9e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
ms.locfileid: "26582737"
---
# <a name="gateway-aggregation-pattern"></a><span data-ttu-id="8c501-103">게이트웨이 집계 패턴</span><span class="sxs-lookup"><span data-stu-id="8c501-103">Gateway Aggregation pattern</span></span>

<span data-ttu-id="8c501-104">게이트웨이를 사용하여 여러 개별 요청을 단일 요청으로 집계합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-104">Use a gateway to aggregate multiple individual requests into a single request.</span></span> <span data-ttu-id="8c501-105">이 패턴은 클라이언트가 다른 백 엔드 시스템을 여러 번 호출하여 작업을 수행해야 할 경우에 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-105">This pattern is useful when a client must make multiple calls to different backend systems to perform an operation.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="8c501-106">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="8c501-106">Context and problem</span></span>

<span data-ttu-id="8c501-107">단일 작업을 수행하기 위해 클라이언트가 다양한 백 엔드 서비스를 여러 번 호출해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-107">To perform a single task, a client may have to make multiple calls to various backend services.</span></span> <span data-ttu-id="8c501-108">여러 서비스를 사용하여 작업을 수행하는 응용 프로그램은 각 요청마다 리소스를 확장해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-108">An application that relies on many services to perform a task must expend resources on each request.</span></span> <span data-ttu-id="8c501-109">새로운 기능이나 서비스가 응용 프로그램에 추가되면 추가 요청이 필요하고 리소스 요구 사항 및 네트워크 호출이 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-109">When any new feature or service is added to the application, additional requests are needed, further increasing resource requirements and network calls.</span></span> <span data-ttu-id="8c501-110">클라이언트와 백 엔드 사이에서 이 데이터 전송이 증가하면 응용 프로그램의 성능 및 규모에 악영향을 줄 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-110">This chattiness between a client and a backend can adversely impact the performance and scale of the application.</span></span>  <span data-ttu-id="8c501-111">여러 소규모 서비스로 빌드된 응용 프로그램은 자연히 서비스 간 호출이 많으므로 마이크로 서비스 아키텍처에서는 이 문제가 보다 일반적입니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-111">Microservice architectures have made this problem more common, as applications built around many smaller services naturally have a higher amount of cross-service calls.</span></span> 

<span data-ttu-id="8c501-112">다음 다이어그램에서 클라이언트가 각 서비스에 요청을 보냅니다(1,2,3).</span><span class="sxs-lookup"><span data-stu-id="8c501-112">In the following diagram, the client sends requests to each service (1,2,3).</span></span> <span data-ttu-id="8c501-113">각 서비스는 요청을 처리하고 응용 프로그램으로 다시 응답을 보냅니다(4,5,6).</span><span class="sxs-lookup"><span data-stu-id="8c501-113">Each service processes the request and sends the response back to the application (4,5,6).</span></span> <span data-ttu-id="8c501-114">일반적으로 대기 시간이 긴 셀룰러 네트워크에서 이러한 방식으로 개별 요청을 사용하는 것은 비효율적이며 연결이 끊어지거나 불완전한 요청이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-114">Over a cellular network with typically high latency, using individual requests in this manner is inefficient and could result in broken connectivity or incomplete requests.</span></span> <span data-ttu-id="8c501-115">각 요청을 병렬로 수행할 수 있지만 응용 프로그램이 각 요청 데이터를 모두 별도의 연결로 전송, 대기 및 처리해야 하므로 실패할 가능성이 높아집니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-115">While each request may be done in parallel, the application must send, wait, and process data for each request, all on separate connections, increasing the chance of failure.</span></span>

![](./_images/gateway-aggregation-problem.png) 

## <a name="solution"></a><span data-ttu-id="8c501-116">해결 방법</span><span class="sxs-lookup"><span data-stu-id="8c501-116">Solution</span></span>

<span data-ttu-id="8c501-117">게이트웨이를 사용하여 클라이언트와 서비스 간의 데이터 전송량을 줄입니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-117">Use a gateway to reduce chattiness between the client and the services.</span></span> <span data-ttu-id="8c501-118">게이트웨이는 클라이언트 요청을 받고, 다양한 백 엔드 시스템에 요청을 디스패치한 다음 결과를 집계하여 요청 클라이언트에 다시 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-118">The gateway receives client requests, dispatches requests to the various backend systems, and then aggregates the results and sends them back to the requesting client.</span></span>

<span data-ttu-id="8c501-119">이 패턴은 백 엔드 서비스에 대한 응용 프로그램 요청 수를 줄이고 대기 시간이 긴 네트워크에서 응용 프로그램 성능을 향상시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-119">This pattern can reduce the number of requests that the application makes to backend services, and improve application performance over high-latency networks.</span></span>

<span data-ttu-id="8c501-120">다음 다이어그램에서는 응용 프로그램이 게이트웨이에 요청을 보냅니다(1).</span><span class="sxs-lookup"><span data-stu-id="8c501-120">In the following diagram, the application sends a request to the gateway (1).</span></span> <span data-ttu-id="8c501-121">요청에는 추가 요청 패키지가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-121">The request contains a package of additional requests.</span></span> <span data-ttu-id="8c501-122">게이트웨이는 이를 분해하여 각 요청을 관련 서비스로 전송 처리합니다(2).</span><span class="sxs-lookup"><span data-stu-id="8c501-122">The gateway decomposes these and processes each request by sending it to the relevant service (2).</span></span> <span data-ttu-id="8c501-123">각 서비스는 게이트웨이에 응답을 반환합니다(3).</span><span class="sxs-lookup"><span data-stu-id="8c501-123">Each service returns a response to the gateway (3).</span></span> <span data-ttu-id="8c501-124">게이트웨이는 각 서비스의 응답을 조합하여 응용 프로그램에 응답을 보냅니다(4).</span><span class="sxs-lookup"><span data-stu-id="8c501-124">The gateway combines the responses from each service and sends the response to the application (4).</span></span> <span data-ttu-id="8c501-125">응용 프로그램은 단일 요청을 보내고 게이트웨이로부터 단일 응답만 수신합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-125">The application makes a single request and receives only a single response from the gateway.</span></span>

![](./_images/gateway-aggregation.png)

## <a name="issues-and-considerations"></a><span data-ttu-id="8c501-126">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="8c501-126">Issues and considerations</span></span>

- <span data-ttu-id="8c501-127">게이트웨이는 백 엔드 서비스를 결합해서는 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-127">The gateway should not introduce service coupling across the backend services.</span></span>
- <span data-ttu-id="8c501-128">게이트웨이는 가능한 대기 시간을 줄이기 위해 백 엔드 서비스 근처에 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-128">The gateway should be located near the backend services to reduce latency as much as possible.</span></span>
- <span data-ttu-id="8c501-129">게이트웨이 서비스에 단일 실패 지점이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-129">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="8c501-130">게이트웨이가 응용 프로그램의 가용성 요구 사항에 맞게 적절하게 디자인되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-130">Ensure the gateway is properly designed to meet your application's availability requirements.</span></span>
- <span data-ttu-id="8c501-131">게이트웨이에 병목 현상이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-131">The gateway may introduce a bottleneck.</span></span> <span data-ttu-id="8c501-132">게이트웨이 성능이 부하를 처리할 정도로 적당하고 예상되는 증가량에 맞게 크기를 조정할 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-132">Ensure the gateway has adequate performance to handle load and can be scaled to meet your anticipated growth.</span></span>
- <span data-ttu-id="8c501-133">게이트웨이 부하 테스트를 수행하여 서비스가 연속으로 실패하지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-133">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="8c501-134">[격벽][bulkhead], [회로 차단][circuit-breaker], [다시 시도][retry] 및 시간 제한 등의 기법을 사용하여 복원력 있는 디자인을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-134">Implement a resilient design, using techniques such as [bulkheads][bulkhead], [circuit breaking][circuit-breaker], [retry][retry], and timeouts.</span></span>
- <span data-ttu-id="8c501-135">하나 이상의 서비스를 호출하는 데 시간이 너무 오래 걸리는 경우 시간을 초과할 수 있기 때문에 데이터 일부 집합만 반환하는 것이 좋을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-135">If one or more service calls takes too long, it may be acceptable to timeout and return a partial set of data.</span></span> <span data-ttu-id="8c501-136">응용 프로그램에서 이 시나리오를 처리하는 방법을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-136">Consider how your application will handle this scenario.</span></span>
- <span data-ttu-id="8c501-137">비동기 I/O를 사용하여 백 엔드의 지연이 응용 프로그램에서 성능 문제를 야기하지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-137">Use asynchronous I/O to ensure that a delay at the backend doesn't cause performance issues in the application.</span></span>
- <span data-ttu-id="8c501-138">상관 관계 ID를 사용한 자동 분산 추적을 구현하여 각 개별 호출을 추적합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-138">Implement distributed tracing using correlation IDs to track each individual call.</span></span>
- <span data-ttu-id="8c501-139">요청 메트릭 및 응답 크기를 모니터링합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-139">Monitor request metrics and response sizes.</span></span>
- <span data-ttu-id="8c501-140">오류를 처리하기 위해 장애 조치 전략으로 캐시된 데이터를 반환하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-140">Consider returning cached data as a failover strategy to handle failures.</span></span>
- <span data-ttu-id="8c501-141">게이트웨이에 집계를 빌드하는 대신 게이트웨이 뒤에 집계 서비스를 배치하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-141">Instead of building aggregation into the gateway, consider placing an aggregation service behind the gateway.</span></span> <span data-ttu-id="8c501-142">요청 집계는 게이트웨이에서 다른 서비스보다 다양한 리소스 요구 사항이 있으므로 게이트웨이의 라우팅 및 오프로딩 기능에 영향을 줄 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-142">Request aggregation will likely have different resource requirements than other services in the gateway and may impact the gateway's routing and offloading functionality.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="8c501-143">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="8c501-143">When to use this pattern</span></span>

<span data-ttu-id="8c501-144">다음 경우에 이 패턴을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-144">Use this pattern when:</span></span>

- <span data-ttu-id="8c501-145">클라이언트가 작업을 수행하기 위해 여러 백 엔드 서비스와 통신해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="8c501-145">A client needs to communicate with multiple backend services to perform an operation.</span></span>
- <span data-ttu-id="8c501-146">클라이언트가 대기 시간이 상당한 네트워크(예: 셀룰러 네트워크)를 사용하는 경우</span><span class="sxs-lookup"><span data-stu-id="8c501-146">The client may use networks with significant latency, such as cellular networks.</span></span>

<span data-ttu-id="8c501-147">다음 경우에는 이 패턴이 적합하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-147">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="8c501-148">여러 작업에서 클라이언트와 단일 서비스 간에 호출 횟수를 줄이고 싶은 경우.</span><span class="sxs-lookup"><span data-stu-id="8c501-148">You want to reduce the number of calls between a client and a single service across multiple operations.</span></span> <span data-ttu-id="8c501-149">이 시나리오에서는 서비스에 일괄 처리 작업을 추가하는 것이 나을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-149">In that scenario, it may be better to add a batch operation to the service.</span></span>
- <span data-ttu-id="8c501-150">클라이언트 또는 응용 프로그램이 백 엔드 서비스 근처에 있고 대기 시간이 중요한 요소가 아닌 경우</span><span class="sxs-lookup"><span data-stu-id="8c501-150">The client or application is located near the backend services and latency is not a significant factor.</span></span>

## <a name="example"></a><span data-ttu-id="8c501-151">예</span><span class="sxs-lookup"><span data-stu-id="8c501-151">Example</span></span>

<span data-ttu-id="8c501-152">다음 예제는 Lua를 사용하여 간단한 게이트웨이 집계 NGINX 서비스를 만드는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="8c501-152">The following example illustrates how to create a simple a gateway aggregation NGINX service using Lua.</span></span>

```lua
worker_processes  4;

events {
  worker_connections 1024;
}

http {
  server {
    listen 80;

    location = /batch {
      content_by_lua '
        ngx.req.read_body()

        -- read json body content
        local cjson = require "cjson"
        local batch = cjson.decode(ngx.req.get_body_data())["batch"]

        -- create capture_multi table
        local requests = {}
        for i, item in ipairs(batch) do
          table.insert(requests, {item.relative_url, { method = ngx.HTTP_GET}})
        end

        -- execute batch requests in parallel
        local results = {}
        local resps = { ngx.location.capture_multi(requests) }
        for i, res in ipairs(resps) do
          table.insert(results, {status = res.status, body = cjson.decode(res.body), header = res.header})
        end

        ngx.say(cjson.encode({results = results}))
      ';
    }

    location = /service1 {
      default_type application/json;
      echo '{"attr1":"val1"}';
    }

    location = /service2 {
      default_type application/json;
      echo '{"attr2":"val2"}';
    }
  }
}
```

## <a name="related-guidance"></a><span data-ttu-id="8c501-153">관련 지침</span><span class="sxs-lookup"><span data-stu-id="8c501-153">Related guidance</span></span>

- [<span data-ttu-id="8c501-154">프런트 엔드에 대한 백 엔드 패턴</span><span class="sxs-lookup"><span data-stu-id="8c501-154">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="8c501-155">게이트웨이 오프로딩 패턴</span><span class="sxs-lookup"><span data-stu-id="8c501-155">Gateway Offloading pattern</span></span>](./gateway-offloading.md)
- [<span data-ttu-id="8c501-156">게이트웨이 라우팅 패턴</span><span class="sxs-lookup"><span data-stu-id="8c501-156">Gateway Routing pattern</span></span>](./gateway-routing.md)

[bulkhead]: ./bulkhead.md
[circuit-breaker]: ./circuit-breaker.md
[retry]: ./retry.md