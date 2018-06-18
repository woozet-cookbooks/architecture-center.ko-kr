---
title: 게이트웨이 라우팅 패턴
description: 단일 엔드포인트를 사용하여 요청을 여러 서비스에 라우팅합니다.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: ea0bc4d31b745043a7ac3afb277dfc46d87ff109
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252603"
---
# <a name="gateway-routing-pattern"></a><span data-ttu-id="b7fef-103">게이트웨이 라우팅 패턴</span><span class="sxs-lookup"><span data-stu-id="b7fef-103">Gateway Routing pattern</span></span>

<span data-ttu-id="b7fef-104">단일 엔드포인트를 사용하여 요청을 여러 서비스에 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-104">Route requests to multiple services using a single endpoint.</span></span> <span data-ttu-id="b7fef-105">이 패턴은 단일 끝점 및 경로의 여러 서비스를 요청에 따라 적절한 서비스에 노출하려는 경우에 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-105">This pattern is useful when you wish to expose multiple services on a single endpoint and route to the appropriate service based on the request.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="b7fef-106">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="b7fef-106">Context and problem</span></span>

<span data-ttu-id="b7fef-107">클라이언트가 여러 서비스를 이용해야 하는 경우 각 서비스에 대해 별도의 끝점을 설정하고 클라이언트를 통해 각 끝점을 관리하는 것이 어려울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-107">When a client needs to consume multiple services, setting up a separate endpoint for each service and having the client manage each endpoint can be challenging.</span></span> <span data-ttu-id="b7fef-108">예를 들어 전자 상거래 응용 프로그램은 검색, 리뷰, 카트, 체크 아웃 및 주문 기록과 같은 서비스를 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-108">For example, an e-commerce application might provide services such as search, reviews, cart, checkout, and order history.</span></span> <span data-ttu-id="b7fef-109">서비스마다 클라이언트가 조작해야 하는 다른 API가 있으며 클라이언트는 서비스에 연결하기 위해 각 끝점에 대해 알고 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-109">Each service has a different API that the client must interact with, and the client must know about each endpoint in order to connect to the services.</span></span> <span data-ttu-id="b7fef-110">API가 변경되면 클라이언트도 업데이트해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-110">If an API changes, the client must be updated as well.</span></span> <span data-ttu-id="b7fef-111">두 개 이상의 개별 서비스로 서비스를 리팩터링하는 경우 서비스와 클라이언트 둘 다에서 코드를 변경해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-111">If you refactor a service into two or more separate services, the code must change in both the service and the client.</span></span>

## <a name="solution"></a><span data-ttu-id="b7fef-112">해결 방법</span><span class="sxs-lookup"><span data-stu-id="b7fef-112">Solution</span></span>

<span data-ttu-id="b7fef-113">응용 프로그램, 서비스 또는 배포 집합 앞에 게이트웨이를 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-113">Place a gateway in front of a set of applications, services, or deployments.</span></span> <span data-ttu-id="b7fef-114">응용 프로그램 계층 7 라우팅을 사용하여 요청을 적절한 인스턴스로 라우트합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-114">Use application Layer 7 routing to route the request to the appropriate instances.</span></span>

<span data-ttu-id="b7fef-115">이 패턴을 사용하면 클라이언트 응용 프로그램이 단일 끝점만 알고 통신하면 됩니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-115">With this pattern, the client application only needs to know about and communicate with a single endpoint.</span></span> <span data-ttu-id="b7fef-116">서비스를 통합하거나 분해하는 경우 클라이언트를 업데이트할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-116">If a service is consolidated or decomposed, the client does not necessarily require updating.</span></span> <span data-ttu-id="b7fef-117">게이트웨이를 계속 요청할 수 있으며 라우팅만 변경됩니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-117">It can continue making requests to the gateway, and only the routing changes.</span></span>

<span data-ttu-id="b7fef-118">또한 게이트웨이를 통해 클라이언트의 백 엔드 서비스를 추상화하여 게이트웨이 뒤에 있는 백 엔드 서비스의 변경을 허용하는 동시에 클라이언트 호출을 단순하게 유지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-118">A gateway also lets you abstract backend services from the clients, allowing you to keep client calls simple while enabling changes in the backend services behind the gateway.</span></span> <span data-ttu-id="b7fef-119">예상 클라이언트 동작을 처리해야 하는 어떤 서비스로든 클라이언트 호출을 라우트할 수 있으므로 클라이언트를 변경하지 않고 게이트웨이 뒤에 서비스를 추가, 분할 및 재구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-119">Client calls can be routed to whatever service or services need to handle the expected client behavior, allowing you to add, split, and reorganize services behind the gateway without changing the client.</span></span>

![](./_images/gateway-routing.png)
 
<span data-ttu-id="b7fef-120">이 패턴은 사용자에게 업데이트를 롤아웃하는 방법을 관리할 수 있으므로 배포에도 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-120">This pattern can also help with deployment, by allowing you to manage how updates are rolled out to users.</span></span> <span data-ttu-id="b7fef-121">새 버전의 서비스를 배포할 때 기존 버전과 함께 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-121">When a new version of your service is deployed, it can be deployed in parallel with the existing version.</span></span> <span data-ttu-id="b7fef-122">라우팅을 통해 클라이언트에게 제공되는 서비스 버전을 제어할 수 있으므로 증분, 병렬 또는 전체 업데이트 롤아웃 등 다양한 릴리스 전략을 유연하게 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-122">Routing lets you control what version of the service is presented to the clients, giving you the flexibility to use various release strategies, whether incremental, parallel, or complete rollouts of updates.</span></span> <span data-ttu-id="b7fef-123">새 서비스를 배포한 후 문제가 발견될 경우 클라이언트에 영향을 주지 않고 게이트웨이에서 구성을 변경하여 빠르게 되돌릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-123">Any issues discovered after the new service is deployed can be quickly reverted by making a configuration change at the gateway, without affecting clients.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="b7fef-124">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="b7fef-124">Issues and considerations</span></span>

- <span data-ttu-id="b7fef-125">게이트웨이 서비스에 단일 실패 지점이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-125">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="b7fef-126">가용성 요구 사항을 충족하도록 올바르게 설계되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-126">Ensure it is properly designed to meet your availability requirements.</span></span> <span data-ttu-id="b7fef-127">구현 시 복원력 및 내결함성 기능을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-127">Consider resiliency and fault tolerance capabilities when implementing.</span></span>
- <span data-ttu-id="b7fef-128">게이트웨이 서비스로 인해 병목 상태가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-128">The gateway service may introduce a bottleneck.</span></span> <span data-ttu-id="b7fef-129">게이트웨이가 부하를 처리할 수 있는 적절한 성능을 갖추고 있고 예상 증가량에 맞게 쉽게 확장될 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-129">Ensure the gateway has adequate performance to handle load and can easily scale in line with your growth expectations.</span></span>
- <span data-ttu-id="b7fef-130">게이트웨이에 대해 부하 테스트를 수행하여 서비스 실패가 계단식으로 연속되지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-130">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="b7fef-131">게이트웨이 라우팅은 수준 7입니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-131">Gateway routing is level 7.</span></span> <span data-ttu-id="b7fef-132">IP, 포트, 헤더 또는 URL을 기반으로 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-132">It can be based on IP, port, header, or URL.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="b7fef-133">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="b7fef-133">When to use this pattern</span></span>

<span data-ttu-id="b7fef-134">다음 경우에 이 패턴을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-134">Use this pattern when:</span></span>

- <span data-ttu-id="b7fef-135">클라이언트가 게이트웨이 뒤에서 액세스할 수 있는 여러 서비스를 이용해야 하는 경우.</span><span class="sxs-lookup"><span data-stu-id="b7fef-135">A client needs to consume multiple services that can be accessed behind a gateway.</span></span>
- <span data-ttu-id="b7fef-136">단일 끝점을 사용하여 클라이언트 응용 프로그램을 간소화하려는 경우.</span><span class="sxs-lookup"><span data-stu-id="b7fef-136">You wish to simplify client applications by using a single endpoint.</span></span>
- <span data-ttu-id="b7fef-137">VM의 포트를 클러스터 가상 IP 주소에 노출하는 경우와 같이 외부에서 주소 지정 가능한 끝점에서 내부 가상 끝점으로 요청을 라우트해야 하는 경우.</span><span class="sxs-lookup"><span data-stu-id="b7fef-137">You need to route requests from externally addressable endpoints to internal virtual endpoints, such as exposing ports on a VM to cluster virtual IP addresses.</span></span>

<span data-ttu-id="b7fef-138">하나 또는 두 개의 서비스만 사용하는 단순 응용 프로그램을 가진 경우에는 이 패턴이 적합하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-138">This pattern may not be suitable when you have a simple application that uses only one or two services.</span></span>

## <a name="example"></a><span data-ttu-id="b7fef-139">예</span><span class="sxs-lookup"><span data-stu-id="b7fef-139">Example</span></span>

<span data-ttu-id="b7fef-140">Nginx를 라우터로 사용하는 다음 예는 다른 가상 디렉터리에 있는 응용 프로그램에 대한 요청을 백 엔드의 다른 컴퓨터로 라우트하는 서버에 대한 단순한 예제 구성 파일입니다.</span><span class="sxs-lookup"><span data-stu-id="b7fef-140">Using Nginx as the router, the following is a simple example configuration file for a server that routes requests for applications residing on different virtual directories to different machines at the back end.</span></span>

```
server {
    listen 80;
    server_name domain.com;

    location /app1 {
        proxy_pass http://10.0.3.10:80;
    }

    location /app2 {
        proxy_pass http://10.0.3.20:80;
    }

    location /app3 {
        proxy_pass http://10.0.3.30:80;
    }
}
```

## <a name="related-guidance"></a><span data-ttu-id="b7fef-141">관련 지침</span><span class="sxs-lookup"><span data-stu-id="b7fef-141">Related guidance</span></span>

- [<span data-ttu-id="b7fef-142">프런트 엔드에 대한 백 엔드 패턴</span><span class="sxs-lookup"><span data-stu-id="b7fef-142">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="b7fef-143">게이트웨이 집계 패턴</span><span class="sxs-lookup"><span data-stu-id="b7fef-143">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="b7fef-144">게이트웨이 오프로딩 패턴</span><span class="sxs-lookup"><span data-stu-id="b7fef-144">Gateway Offloading pattern</span></span>](./gateway-offloading.md)



