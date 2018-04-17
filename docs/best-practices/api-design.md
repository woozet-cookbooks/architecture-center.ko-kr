---
title: API 디자인 지침
description: 잘 디자인된 Web API를 만드는 방법에 관한 지침입니다.
author: dragon119
ms.date: 01/12/2018
pnp.series.title: Best Practices
ms.openlocfilehash: a8c4a81835ebd3ebdba2fd2cec624a9a9d5646f5
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/17/2018
---
# <a name="api-design"></a><span data-ttu-id="73a47-103">API 디자인</span><span class="sxs-lookup"><span data-stu-id="73a47-103">API design</span></span>

<span data-ttu-id="73a47-104">대부분의 최신 웹 응용 프로그램은 클라이언트가 응용 프로그램과 상호 작용하는 데 사용할 수 있는 API를 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-104">Most modern web applications expose APIs that clients can use to interact with the application.</span></span> <span data-ttu-id="73a47-105">잘 디자인된 웹 API는 아래와 같은 특성을 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-105">A well-designed web API should aim to support:</span></span>

* <span data-ttu-id="73a47-106">**플랫폼 독립성**.</span><span class="sxs-lookup"><span data-stu-id="73a47-106">**Platform independence**.</span></span> <span data-ttu-id="73a47-107">모든 클라이언트는 내부에서 API가 구현되는 방법에 관계없이 API를 호출할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-107">Any client should be able to call the API, regardless of how the API is implemented internally.</span></span> <span data-ttu-id="73a47-108">그러려면 표준 프로토콜을 사용해야 하고, 클라이언트 및 웹 서비스가 교환할 데이터 형식에 동의할 수 있는 메커니즘이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-108">This requires using standard protocols, and having a mechanism whereby the client and the web service can agree on the format of the data to exchange.</span></span>

* <span data-ttu-id="73a47-109">**서비스 진화**.</span><span class="sxs-lookup"><span data-stu-id="73a47-109">**Service evolution**.</span></span> <span data-ttu-id="73a47-110">Web API는 클라이언트 응용 프로그램과 독립적으로 기능을 진화시키고 추가할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-110">The web API should be able to evolve and add functionality independently from client applications.</span></span> <span data-ttu-id="73a47-111">API가 진화해도 기존 클라이언트 응용 프로그램은 수정 없이 계속 작동할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-111">As the API evolves, existing client applications should continue to function without modification.</span></span> <span data-ttu-id="73a47-112">모든 기능은 클라이언트 응용 프로그램이 해당 기능을 완전히 이용할 수 있도록 검색이 가능해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-112">All functionality should be discoverable, so that client applications can fully utilize it.</span></span>

<span data-ttu-id="73a47-113">이 지침에서는 Web API를 디자인할 때 고려해야 하는 사항을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-113">This guidance describes issues that you should consider when designing a web API.</span></span>

## <a name="introduction-to-rest"></a><span data-ttu-id="73a47-114">REST 소개</span><span class="sxs-lookup"><span data-stu-id="73a47-114">Introduction to REST</span></span>

<span data-ttu-id="73a47-115">2000년에 Roy Fielding은 웹 서비스를 디자인하는 아키텍처 접근 방식으로 REST(Representational State Transfer)를 제안했습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-115">In 2000, Roy Fielding proposed Representational State Transfer (REST) as an architectural approach to designing web services.</span></span> <span data-ttu-id="73a47-116">REST는 하이퍼미디어 기반 분산 시스템을 구축하기 위한 아키텍처 스타일입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-116">REST is an architectural style for building distributed systems based on hypermedia.</span></span> <span data-ttu-id="73a47-117">REST는 어떤 기본 프로토콜과도 독립적이며 HTTP에 연결될 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-117">REST is independent of any underlying protocol and is not necessarily tied to HTTP.</span></span> <span data-ttu-id="73a47-118">그러나 대부분의 일반적인 REST 구현에서 응용 프로그램 프로토콜로 HTTP를 사용하고, 이 지침에서는 HTTP를 위한 REST API 디자인에 중점을 둡니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-118">However, most common REST implementations use HTTP as the application protocol, and this guide focuses on designing REST APIs for HTTP.</span></span>

<span data-ttu-id="73a47-119">REST가 HTTP보다 우수한 주요 장점은 개방형 표준을 사용하므로 API 또는 클라이언트 응용 프로그램의 구현이 특정 구현에 바인딩되지 않는다는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-119">A primary advantage of REST over HTTP is that it uses open standards, and does not bind the implementation of the API or the client applications any specific implementation.</span></span> <span data-ttu-id="73a47-120">예를 들어 REST 웹 서비스는 ASP.NET으로 작성할 수 있으며, 클라이언트 응용 프로그램은 HTTP 요청을 생성하고 HTTP 응답을 구문 분석할 수 있는 모든 언어 또는 도구 집합을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-120">For example, a REST web service could be written in ASP.NET, and client applications can use any language or toolset that can generate HTTP requests and parse HTTP responses.</span></span>

<span data-ttu-id="73a47-121">다음은 HTTP를 사용하는 RESTful API의 몇 가지 기본 디자인 원칙입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-121">Here are some of the main design principles of RESTful APIs using HTTP:</span></span>

- <span data-ttu-id="73a47-122">REST API는 *리소스*를 중심으로 디자인되며, 클라이언트에서 액세스할 수 있는 모든 종류의 개체, 데이터 또는 서비스가 리소스에 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-122">REST APIs are designed around *resources*, which are any kind of object, data, or service that can be accessed by the client.</span></span> 

- <span data-ttu-id="73a47-123">리소스마다 해당 리소스를 고유하게 식별하는 URI인 *식별자*가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-123">A resource has an *identifier*, which is a URI that uniquely identifies that resource.</span></span> <span data-ttu-id="73a47-124">예를 들어 특정 고객 주문의 URI는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-124">For example, the URI for a particular customer order might be:</span></span> 
 
    ```http
    http://adventure-works.com/orders/1
    ```
 
- <span data-ttu-id="73a47-125">클라이언트가 리소스의 *표현*을 교환하여 서비스와 상호 작용합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-125">Clients interact with a service by exchanging *representations* of resources.</span></span> <span data-ttu-id="73a47-126">많은 Web API가 교환 형식으로 JSON을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-126">Many web APIs use JSON as the exchange format.</span></span> <span data-ttu-id="73a47-127">예를 들어 위에 나열된 URI에 대한 GET 요청은 이 응답 본문을 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-127">For example, a GET request to the URI listed above might return this response body:</span></span>

    ```json
    {"orderId":1,"orderValue":99.90,"productId":1,"quantity":1}
    ```

- <span data-ttu-id="73a47-128">REST API는 균일한 인터페이스를 사용하므로 클라이언트와 서비스 구현을 분리하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-128">REST APIs use a uniform interface, which helps to decouple the client and service implementations.</span></span> <span data-ttu-id="73a47-129">HTTP를 기반으로 하는 REST API의 경우 리소스에 표준 HTTP 동사 수행 작업을 사용하는 것이 균일한 인터페이스에 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-129">For REST APIs built on HTTP, the uniform interface includes using standard HTTP verbs to perform operations on resources.</span></span> <span data-ttu-id="73a47-130">가장 일반적인 작업은 GET, POST, PUT, PATCH 및 DELETE입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-130">The most common operations are GET, POST, PUT, PATCH, and DELETE.</span></span> 

- <span data-ttu-id="73a47-131">REST API는 상태 비저장 요청 모델을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-131">REST APIs use a stateless request model.</span></span> <span data-ttu-id="73a47-132">HTTP 요청은 독립적이어야 하고 임의 순서로 발생할 수 있으므로, 요청 사이에 일시적인 상태 정보를 유지할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-132">HTTP requests should be independent and may occur in any order, so keeping transient state information between requests is not feasible.</span></span> <span data-ttu-id="73a47-133">정보는 리소스 자체에만 저장되며 각 요청은 자동 작업이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-133">The only place where information is stored is in the resources themselves, and each request should be an atomic operation.</span></span> <span data-ttu-id="73a47-134">이러한 제약 조건이 있기 때문에 웹 서비스의 확장성이 우수합니다. 클라이언트와 특정 서버 사이에 선호도를 유지할 필요가 없기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-134">This constraint enables web services to be highly scalable, because there is no need to retain any affinity between clients and specific servers.</span></span> <span data-ttu-id="73a47-135">모든 서버는 모든 클라이언트의 모든 요청을 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-135">Any server can handle any request from any client.</span></span> <span data-ttu-id="73a47-136">그렇긴 하지만, 다른 요소가 확장성을 제한할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-136">That said, other factors can limit scalability.</span></span> <span data-ttu-id="73a47-137">예를 들어 많은 웹 서비스가 백 엔드 데이터 저장소에 데이터를 쓰며, 이 경우 규모 확장이 어려울 수 있습니다. ([데이터 분할](./data-partitioning.md) 문서를 보시면 데이터 저장소의 규모 확장 전략에 대해 설명되어 있습니다.)</span><span class="sxs-lookup"><span data-stu-id="73a47-137">For example, many web services write to a backend data store, which may be hard to scale out. (The article [Data Partitioning](./data-partitioning.md) describes strategies to scale out a data store.)</span></span>

- <span data-ttu-id="73a47-138">REST API는 표현에 포함된 하이퍼미디어 링크에 따라 구동됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-138">REST APIs are driven by hypermedia links that are contained in the representation.</span></span> <span data-ttu-id="73a47-139">예를 들어 다음은 주문의 JSON 표현을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-139">For example, the following shows a JSON representation of an order.</span></span> <span data-ttu-id="73a47-140">주문과 관련된 고객을 가져오거나 업데이트하는 링크를 포함하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-140">It contains links to get or update the customer associated with the order.</span></span> 
 
    ```json
    {
        "orderID":3,
        "productID":2,
        "quantity":4,
        "orderValue":16.60,
        "links": [
            {"rel":"product","href":"http://adventure-works.com/customers/3", "action":"GET" },
            {"rel":"product","href":"http://adventure-works.com/customers/3", "action":"PUT" } 
        ]
    } 
    ```


<span data-ttu-id="73a47-141">2008년에 Leonard Richardson은 Web API에 대한 다음과 같은 [성숙도 모델](https://martinfowler.com/articles/richardsonMaturityModel.html)을 제안했습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-141">In 2008, Leonard Richardson proposed the following [maturity model](https://martinfowler.com/articles/richardsonMaturityModel.html) for web APIs:</span></span>

- <span data-ttu-id="73a47-142">수준 0: 한 URI를 정의합니다. 모든 작업은 이 URI에 대한 POST 요청입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-142">Level 0: Define one URI, and all operations are POST requests to this URI.</span></span>
- <span data-ttu-id="73a47-143">수준 1: 개별 리소스에 대한 별도의 URI를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-143">Level 1: Create separate URIs for individual resources.</span></span>
- <span data-ttu-id="73a47-144">수준 2: HTTP 메서드를 사용하여 리소스에 대한 작업을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-144">Level 2: Use HTTP methods to define operations on resources.</span></span>
- <span data-ttu-id="73a47-145">수준 3: 하이퍼미디어(HATEOAS, 아래에 설명)를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-145">Level 3: Use hypermedia (HATEOAS, described below).</span></span>

<span data-ttu-id="73a47-146">수준 3은 Fielding의 정의에 따르면 진정한 RESTful API에 해당합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-146">Level 3 corresponds to a truly RESTful API according to Fielding's definition.</span></span> <span data-ttu-id="73a47-147">실제로 게시된 여러 Web API가 수준 2의 어딘가에 해당합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-147">In practice, many  published web APIs fall somewhere around level 2.</span></span>  

## <a name="organize-the-api-around-resources"></a><span data-ttu-id="73a47-148">리소스를 중심으로 API 구성</span><span class="sxs-lookup"><span data-stu-id="73a47-148">Organize the API around resources</span></span>

<span data-ttu-id="73a47-149">즉, 웹 API가 표시하는 비즈니스 엔터티에 집중해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-149">Focus on the business entities that the web API exposes.</span></span> <span data-ttu-id="73a47-150">예를 들어 전자 상거래 시스템에서는 기본 엔터티가 고객과 주문입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-150">For example, in an e-commerce system, the primary entities might be customers and orders.</span></span> <span data-ttu-id="73a47-151">주문 정보가 포함된 HTTP POST 요청을 전송하여 주문 만들기를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-151">Creating an order can be achieved by sending an HTTP POST request that contains the order information.</span></span> <span data-ttu-id="73a47-152">HTTP 응답은 주문이 성공적으로 수행되었는지 여부를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-152">The HTTP response indicates whether the order was placed successfully or not.</span></span> <span data-ttu-id="73a47-153">가능하다면 리소스 URI는 동사(리소스에 대한 작업)가 아닌 명사(리소스)를 기반으로 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-153">When possible, resource URIs should be based on nouns (the resource) and not verbs (the operations on the resource).</span></span> 

```HTTP
http://adventure-works.com/orders // Good

http://adventure-works.com/create-order // Avoid
```

<span data-ttu-id="73a47-154">리소스는 단일 물리적 데이터 항목을 기반으로 할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-154">A resource does not have to be based on a single physical data item.</span></span> <span data-ttu-id="73a47-155">예를 들어 주문 리소스는 내부적으로 관계형 데이터베이스의 여러 테이블로 구현할 수 있지만 클라이언트에 대해서는 단일 엔터티로 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-155">For example, an order resource might be implemented internally as several tables in a relational database, but presented to the client as a single entity.</span></span> <span data-ttu-id="73a47-156">단순히 데이터베이스의 내부 구조를 반영하는 API를 만들지 마세요.</span><span class="sxs-lookup"><span data-stu-id="73a47-156">Avoid creating APIs that simply mirror the internal structure of a database.</span></span> <span data-ttu-id="73a47-157">REST의 목적은 엔터티 및 해당 엔터티에서 응용 프로그램이 수행할 수 있는 작업을 모델링하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-157">The purpose of REST is to model entities and the operations that an application can perform on those entities.</span></span> <span data-ttu-id="73a47-158">클라이언트는 내부 구현에 노출되면 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-158">A client should not be exposed to the internal implementation.</span></span>

<span data-ttu-id="73a47-159">엔터티는 종종 컬렉션(주문, 고객)으로 그룹화됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-159">Entities are often grouped together into collections (orders, customers).</span></span> <span data-ttu-id="73a47-160">컬렉션은 컬렉션 내 항목과는 별도의 리소스이며 고유한 URI가 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-160">A collection is a separate resource from the item within the collection, and should have its own URI.</span></span> <span data-ttu-id="73a47-161">예를 들어 다음 URI는 주문 컬렉션을 나타낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-161">For example, the following URI might represent the collection of orders:</span></span> 

```HTTP
http://adventure-works.com/orders
```

<span data-ttu-id="73a47-162">컬렉션 URI에 HTTP GET 요청을 보내면 컬렉션에 있는 항목 목록을 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-162">Sending an HTTP GET request to the collection URI retrieves a list of items in the collection.</span></span> <span data-ttu-id="73a47-163">또한 컬렉션의 항목마다 고유의 URI가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-163">Each item in the collection also has its own unique URI.</span></span> <span data-ttu-id="73a47-164">항목의 URI에 대한 HTTP GET 요청은 해당 항목의 세부 정보를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-164">An HTTP GET request to the item's URI returns the details of that item.</span></span> 

<span data-ttu-id="73a47-165">URI에 일관적인 명명 규칙을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-165">Adopt a consistent naming convention in URIs.</span></span> <span data-ttu-id="73a47-166">일반적으로 이렇게 하면 컬렉션을 참조하는 URI에 대해 복수 명사를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-166">In general, it helps to use plural nouns for URIs that reference collections.</span></span> <span data-ttu-id="73a47-167">컬렉션 및 항목에 대한 URI를 계층 구조로 구성하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-167">It's a good practice to organize URIs for collections and items into a hierarchy.</span></span> <span data-ttu-id="73a47-168">예를 들어 `/customers`는 고객 컬렉션의 경로이고, `/customers/5`는 ID가 5인 고객의 경로입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-168">For example, `/customers` is the path to the customers collection, and `/customers/5` is the path to the customer with ID equal to 5.</span></span> <span data-ttu-id="73a47-169">이 접근 방식을 사용하면 웹 API를 직관적으로 유지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-169">This approach helps to keep the web API intuitive.</span></span> <span data-ttu-id="73a47-170">또한 많은 Web API 프레임워크는 매개 변수가 있는 URI 경로를 기반으로 요청을 라우팅할 수 있으므로 개발자는 경로 `/customers/{id}`에 대한 경로를 정의할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-170">Also, many web API frameworks can route requests based on parameterized URI paths, so you could define a route for the path `/customers/{id}`.</span></span>

<span data-ttu-id="73a47-171">서로 다른 리소스 형식과 이러한 연결을 표시하는 방법 사이의 관계도 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-171">Also consider the relationships between different types of resources and how you might expose these associations.</span></span> <span data-ttu-id="73a47-172">예를 들어 `/customers/5/orders`는 고객 5에 대한 모든 주문을 나타낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-172">For example, the `/customers/5/orders` might represent all of the orders for customer 5.</span></span> <span data-ttu-id="73a47-173">반대 방향으로 이동하여 `/orders/99/customer` 같은 URI를 사용하여 주문에서 고객으로의 연결을 표시할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-173">You could also go in the other direction, and represent the association from an order back to a customer with a URI such as `/orders/99/customer`.</span></span> <span data-ttu-id="73a47-174">그러나 이 모델을 너무 많이 확장하면 구현이 어려울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-174">However, extending this model too far can become cumbersome to implement.</span></span> <span data-ttu-id="73a47-175">HTTP 응답 메시지의 본문에 연결된 리소스에 대한 탐색 가능한 링크를 제공하는 방법이 더 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-175">A better solution is to provide navigable links to associated resources in the body of the HTTP response message.</span></span> <span data-ttu-id="73a47-176">이 메커니즘은 [HATEOAS 접근 방식을 사용하여 관련 리소스 탐색 사용](#using-the-hateoas-approach-to-enable-navigation-to-related-resources) 섹션에 자세히 설명되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-176">This mechanism is described in more detail in the section [Using the HATEOAS Approach to Enable Navigation To Related Resources later](#using-the-hateoas-approach-to-enable-navigation-to-related-resources).</span></span>

<span data-ttu-id="73a47-177">좀 더 복잡한 시스템에서는 `/customers/1/orders/99/products`처럼 클라이언트가 여러 관계 수준을 탐색할 수 있는 URI를 제공하고 싶을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-177">In more complex systems, it can be tempting to provide URIs that enable a client to navigate through several levels of relationships, such as `/customers/1/orders/99/products`.</span></span> <span data-ttu-id="73a47-178">그러나 이 수준의 복잡성은 유지하기 어려울 수 있으며 나중에 리소스 사이의 관계가 변하면 유연성이 떨어집니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-178">However, this level of complexity can be difficult to maintain and is inflexible if the relationships between resources change in the future.</span></span> <span data-ttu-id="73a47-179">그 대신 URI를 비교적 간단하게 유지해 보세요.</span><span class="sxs-lookup"><span data-stu-id="73a47-179">Instead, try to keep URIs relatively simple.</span></span> <span data-ttu-id="73a47-180">응용 프로그램이 리소스 참조를 지정한 후에는 이 참조를 사용하여 해당 리소스와 관련된 항목을 찾을 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-180">Once an application has a reference to a resource, it should be possible to use this reference to find items related to that resource.</span></span> <span data-ttu-id="73a47-181">이전 쿼리를 `/customers/1/orders` URI로 바꿔서 고객 1의 모든 주문을 찾은 후 `/orders/99/products`로 바꿔서 이 주문의 제품을 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-181">The preceding query can be replaced with the URI `/customers/1/orders` to find all the orders for customer 1, and then `/orders/99/products` to find the products in this order.</span></span>

> [!TIP]
> <span data-ttu-id="73a47-182">리소스 URI를 *컬렉션/항목/컬렉션*보다 더 복잡하게 요구하지 않는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-182">Avoid requiring resource URIs more complex than *collection/item/collection*.</span></span>

<span data-ttu-id="73a47-183">또 다른 요소는 모든 웹 요청이 웹 서버의 부하를 높인다는 점입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-183">Another factor is that all web requests impose a load on the web server.</span></span> <span data-ttu-id="73a47-184">요청이 많을수록 부하가 커집니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-184">The more requests, the bigger the load.</span></span> <span data-ttu-id="73a47-185">따라서 다수의 작은 리소스를 표시하는 "번잡한" Web API를 피하도록 노력해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-185">Therefore, try to avoid "chatty" web APIs that expose a large number of small resources.</span></span> <span data-ttu-id="73a47-186">이러한 API를 사용하려면 클라이언트 응용 프로그램이 요구하는 모든 데이터를 찾을 수 있도록 여러 요청을 보내야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-186">Such an API may require a client application to send multiple requests to find all of the data that it requires.</span></span> <span data-ttu-id="73a47-187">그 대신, 데이터를 비정규화하고 단일 요청을 통해 관련 정보를 검색할 수 있는 더 큰 리소스로 결합하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-187">Instead, you might want to denormalize the data and combine related information into bigger resources that can be retrieved with a single request.</span></span> <span data-ttu-id="73a47-188">단, 이 접근 방식과 클라이언트에 필요 없는 데이터를 가져오는 오버헤드의 균형을 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-188">However, you need to balance this approach against the overhead of fetching data that the client doesn't need.</span></span> <span data-ttu-id="73a47-189">큰 개체를 검색하면 요청의 대기 시간이 증가하고 추가 대역폭 비용이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-189">Retrieving large objects can increase the latency of a request and incur additional bandwidth costs.</span></span> <span data-ttu-id="73a47-190">이러한 성능 안티패턴에 대한 자세한 내용은 [번잡한 I/O](../antipatterns/chatty-io/index.md) 및 [불필요한 가져오기](../antipatterns/extraneous-fetching/index.md)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="73a47-190">For more information about these performance antipatterns, see [Chatty I/O](../antipatterns/chatty-io/index.md) and [Extraneous Fetching](../antipatterns/extraneous-fetching/index.md).</span></span>

<span data-ttu-id="73a47-191">Web API와 기본 데이터 원본 사이에 종속성이 발생하지 않도록 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-191">Avoid introducing dependencies between the web API and the underlying data sources.</span></span> <span data-ttu-id="73a47-192">예를 들어 데이터가 관계형 데이터베이스에 저장되는 경우 Web API는 각 테이블을 리소스 컬렉션으로 표시할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-192">For example, if your data is stored in a relational database, the web API doesn't need to expose each table as a collection of resources.</span></span> <span data-ttu-id="73a47-193">사실 이것은 서투른 디자인입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-193">In fact, that's probably a poor design.</span></span> <span data-ttu-id="73a47-194">그 대신 Web API를 데이터베이스의 추상화라고 생각해 보세요.</span><span class="sxs-lookup"><span data-stu-id="73a47-194">Instead, think of the web API as an abstraction of the database.</span></span> <span data-ttu-id="73a47-195">필요하다면 데이터베이스와 Web API 사이에 매핑 계층을 도입합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-195">If necessary, introduce a mapping layer between the database and the web API.</span></span> <span data-ttu-id="73a47-196">이 방법을 사용하면 클라이언트 응용 프로그램이 기본 데이터베이스 스키마의 변경 내용으로부터 격리됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-196">That way, client applications are isolated from changes to the underlying database scheme.</span></span>

<span data-ttu-id="73a47-197">마지막으로, 웹 API에 의해 구현된 일부 작업을 특정 리소스에 매핑하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-197">Finally, it might not be possible to map every operation implemented by a web API to a specific resource.</span></span> <span data-ttu-id="73a47-198">HTTP GET 요청을 통해 기능을 호출하고 결과를 HTTP 응답 메시지로 반환하는 *리소스가 아닌* 시나리오를 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-198">You can handle such *non-resource* scenarios through HTTP requests that invoke a function and return the results as an HTTP response message.</span></span> <span data-ttu-id="73a47-199">예를 들어 더하기 및 빼기 같은 단순한 계산기 작업을 구현하는 Web API는 이러한 작업을 의사 리소스로 표시하고 쿼리 문자열을 사용하여 필요한 매개 변수를 지정하는 URI를 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-199">For example, a web API that implements simple calculator operations such as add and subtract could provide URIs that expose these operations as pseudo resources and use the query string to specify the parameters required.</span></span> <span data-ttu-id="73a47-200">예를 들어 URI */add?operand1=99&operand2=1*에 대한 GET 요청은 본문에 값 100이 포함된 응답 메시지를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-200">For example a GET request to the URI */add?operand1=99&operand2=1* would return a response message with the body containing the value 100.</span></span> <span data-ttu-id="73a47-201">그러나 이러한 형식의 URI는 제한적으로만 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-201">However, only use these forms of URIs sparingly.</span></span>

## <a name="define-operations-in-terms-of-http-methods"></a><span data-ttu-id="73a47-202">HTTP 메서드를 기준으로 작업 정의</span><span class="sxs-lookup"><span data-stu-id="73a47-202">Define operations in terms of HTTP methods</span></span>

<span data-ttu-id="73a47-203">HTTP 프로토콜은 요청에 의미 체계의미를 할당하는 다양한 메서드를 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-203">The HTTP protocol defines a number of methods that assign semantic meaning to a request.</span></span> <span data-ttu-id="73a47-204">대부분의 RESTful 웹 API에서 사용하는 일반적인 HTTP 메서드는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-204">The common HTTP methods used by most RESTful web APIs are:</span></span>

* <span data-ttu-id="73a47-205">**GET**은 지정된 URI에서 리소스의 표현을 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-205">**GET** retrieves a representation of the resource at the specified URI.</span></span> <span data-ttu-id="73a47-206">응답 메시지의 본문은 요청된 리소스의 세부 정보를 포함하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-206">The body of the response message contains the details of the requested resource.</span></span>
* <span data-ttu-id="73a47-207">**POST**는 지정된 URI에 새 리소스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-207">**POST** creates a new resource at the specified URI.</span></span> <span data-ttu-id="73a47-208">요청 메시지의 본문은 새 리소스의 세부 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-208">The body of the request message provides the details of the new resource.</span></span> <span data-ttu-id="73a47-209">참고로 POST를 사용하여 실제로 리소스를 만들지 않는 작업을 트리거할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-209">Note that POST can also be used to trigger operations that don't actually create resources.</span></span>
* <span data-ttu-id="73a47-210">**PUT**은 지정된 URI에 리소스를 만들거나 대체합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-210">**PUT** either creates or replaces the resource at the specified URI.</span></span> <span data-ttu-id="73a47-211">요청 메시지의 본문은 만들 또는 업데이트할 리소스를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-211">The body of the request message specifies the resource to be created or updated.</span></span>
* <span data-ttu-id="73a47-212">**PATCH**는 리소스의 부분 업데이트를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-212">**PATCH** performs a partial update of a resource.</span></span> <span data-ttu-id="73a47-213">요청 본문은 리소스에 적용할 변경 내용을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-213">The request body specifies the set of changes to apply to the resource.</span></span>
* <span data-ttu-id="73a47-214">**DELETE**는 지정된 URI의 리소스를 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-214">**DELETE** removes the resource at the specified URI.</span></span>

<span data-ttu-id="73a47-215">특정 요청의 효과는 리소스가 컬렉션인지 아니면 개별 항목인지에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-215">The effect of a specific request should depend on whether the resource is a collection or an individual item.</span></span> <span data-ttu-id="73a47-216">다음 표는 전자 상거래 예제를 사용 하여 대부분의 RESTful 구현에서 적용하는 일반적인 규칙을 요약합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-216">The following table summarizes the common conventions adopted by most RESTful implementations using the ecommerce example.</span></span> <span data-ttu-id="73a47-217">참고로 이 요청 중 일부는 구현되지 않을 수 있으며, 구현 여부는 특정 시나리오에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-217">Note that not all of these requests might be implemented; it depends on the specific scenario.</span></span>

| <span data-ttu-id="73a47-218">**리소스**</span><span class="sxs-lookup"><span data-stu-id="73a47-218">**Resource**</span></span> | <span data-ttu-id="73a47-219">**POST**</span><span class="sxs-lookup"><span data-stu-id="73a47-219">**POST**</span></span> | <span data-ttu-id="73a47-220">**GET**</span><span class="sxs-lookup"><span data-stu-id="73a47-220">**GET**</span></span> | <span data-ttu-id="73a47-221">**PUT**</span><span class="sxs-lookup"><span data-stu-id="73a47-221">**PUT**</span></span> | <span data-ttu-id="73a47-222">**DELETE**</span><span class="sxs-lookup"><span data-stu-id="73a47-222">**DELETE**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="73a47-223">/customers</span><span class="sxs-lookup"><span data-stu-id="73a47-223">/customers</span></span> |<span data-ttu-id="73a47-224">새 고객 만들기</span><span class="sxs-lookup"><span data-stu-id="73a47-224">Create a new customer</span></span> |<span data-ttu-id="73a47-225">모든 고객 검색</span><span class="sxs-lookup"><span data-stu-id="73a47-225">Retrieve all customers</span></span> |<span data-ttu-id="73a47-226">고객 대량 업데이트</span><span class="sxs-lookup"><span data-stu-id="73a47-226">Bulk update of customers</span></span> |<span data-ttu-id="73a47-227">모든 고객 제거</span><span class="sxs-lookup"><span data-stu-id="73a47-227">Remove all customers</span></span> |
| <span data-ttu-id="73a47-228">/customers/1</span><span class="sxs-lookup"><span data-stu-id="73a47-228">/customers/1</span></span> |<span data-ttu-id="73a47-229">오류</span><span class="sxs-lookup"><span data-stu-id="73a47-229">Error</span></span> |<span data-ttu-id="73a47-230">고객 1에 대한 세부 정보 검색</span><span class="sxs-lookup"><span data-stu-id="73a47-230">Retrieve the details for customer 1</span></span> |<span data-ttu-id="73a47-231">고객 1이 있는 경우 고객 1의 세부 정보 업데이트</span><span class="sxs-lookup"><span data-stu-id="73a47-231">Update the details of customer 1 if it exists</span></span> |<span data-ttu-id="73a47-232">고객 1 제거</span><span class="sxs-lookup"><span data-stu-id="73a47-232">Remove customer 1</span></span> |
| <span data-ttu-id="73a47-233">/customers/1/orders</span><span class="sxs-lookup"><span data-stu-id="73a47-233">/customers/1/orders</span></span> |<span data-ttu-id="73a47-234">고객 1에 대한 새 주문 만들기</span><span class="sxs-lookup"><span data-stu-id="73a47-234">Create a new order for customer 1</span></span> |<span data-ttu-id="73a47-235">고객 1에 대한 모든 주문 검색</span><span class="sxs-lookup"><span data-stu-id="73a47-235">Retrieve all orders for customer 1</span></span> |<span data-ttu-id="73a47-236">고객 1의 주문 대량 업데이트</span><span class="sxs-lookup"><span data-stu-id="73a47-236">Bulk update of orders for customer 1</span></span> |<span data-ttu-id="73a47-237">고객 1의 모든 주문 제거</span><span class="sxs-lookup"><span data-stu-id="73a47-237">Remove all orders for customer 1</span></span> |

<span data-ttu-id="73a47-238">POST, PUT 및 PATCH의 차이점을 구분하기 어려울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-238">The differences between POST, PUT, and PATCH can be confusing.</span></span>

- <span data-ttu-id="73a47-239">POST 요청은 리소스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-239">A POST request creates a resource.</span></span> <span data-ttu-id="73a47-240">서버는 새 리소스에 대한 URI를 할당하고 클라이언트에 해당 URI를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-240">The server assigns a URI for the new resource, and returns that URI to the client.</span></span> <span data-ttu-id="73a47-241">REST 모델에서는 컬렉션에 POST 요청을 자주 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-241">In the REST model, you frequently apply POST requests to collections.</span></span> <span data-ttu-id="73a47-242">새 리소스가 컬렉션에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-242">The new resource is added to the collection.</span></span> <span data-ttu-id="73a47-243">POST 요청은 새 리소스를 만들지 않고 기존 리소스에 처리할 데이터를 보내는데 사용할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-243">A POST request can also be used to submit data for processing to an existing resource, without any new resource being created.</span></span>

- <span data-ttu-id="73a47-244">PUT 요청은 리소스를 만들거나 *또는* 기존 리소스를 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-244">A PUT request creates a resource *or* updates an existing resource.</span></span> <span data-ttu-id="73a47-245">클라이언트는 리소스의 URI를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-245">The client specifies the URI for the resource.</span></span> <span data-ttu-id="73a47-246">요청 본문에는 리소스의 완전한 표현이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-246">The request body contains a complete representation of the resource.</span></span> <span data-ttu-id="73a47-247">이 URI를 사용하는 리소스가 이미 있으면 리소스가 대체됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-247">If a resource with this URI already exists, it is replaced.</span></span> <span data-ttu-id="73a47-248">아직 없고 서버에서 리소스 만들기를 지원하는 경우 새 리소스가 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-248">Otherwise a new resource is created, if the server supports doing so.</span></span> <span data-ttu-id="73a47-249">PUT 요청은 컬렉션보다는 특정 고객 같은 개별 항목인 리소스에 가장 자주 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-249">PUT requests are most frequently applied to resources that are individual items, such as a specific customer, rather than collections.</span></span> <span data-ttu-id="73a47-250">서버에서 PUT을 통한 업데이트를 지원하지만 만들기는 지원하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-250">A server might support updates but not creation via PUT.</span></span> <span data-ttu-id="73a47-251">PUT을 통한 만들기 지원 여부는 리소스가 존재하기 전에 클라이언트가 의미 있는 방법으로 리소스에 URI를 할당할 수 있는지 여부에 따라 결정됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-251">Whether to support creation via PUT depends on whether the client can meaningfully assign a URI to a resource before it exists.</span></span> <span data-ttu-id="73a47-252">할당할 수 없는 경우 POST를 사용하여 리소스를 만들고 PUT 또는 PATCH를 사용하여 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-252">If not, then use POST to create resources and PUT or PATCH to update.</span></span>

- <span data-ttu-id="73a47-253">PATCH 요청은 기존 리소스에 *부분 업데이트*를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-253">A PATCH request performs a *partial update* to an existing resource.</span></span> <span data-ttu-id="73a47-254">클라이언트는 리소스의 URI를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-254">The client specifies the URI for the resource.</span></span> <span data-ttu-id="73a47-255">요청 본문은 리소스에 적용할 *변경 내용*을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-255">The request body specifies a set of *changes* to apply to the resource.</span></span> <span data-ttu-id="73a47-256">클라이언트가 리소스의 전체 표현이 아닌 변경 내용만 보내기 때문에 PUT을 사용하는 것보다 이 방법이 더 효율적일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-256">This can be more efficient than using PUT, because the client only sends the changes, not the entire representation of the resource.</span></span> <span data-ttu-id="73a47-257">또한 서버에서 리소스 만들기를 지원하는 경우 기술적으로 PATCH는 새 리소스를 만들 수 있습니다("null" 리소스에 대한 업데이트를 지정하여).</span><span class="sxs-lookup"><span data-stu-id="73a47-257">Technically PATCH can also create a new resource (by specifying a set of updates to a "null" resource), if the server supports this.</span></span> 

<span data-ttu-id="73a47-258">PUT 요청은 idempotent여야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-258">PUT requests must be idempotent.</span></span> <span data-ttu-id="73a47-259">클라이언트가 동일한 PUT 요청을 여러 번 제출하는 경우 그 결과가 항상 같아야 합니다(같은 값을 사용하여 같은 리소스가 수정되므로).</span><span class="sxs-lookup"><span data-stu-id="73a47-259">If a client submits the same PUT request multiple times, the results should always be the same (the same resource will be modified with the same values).</span></span> <span data-ttu-id="73a47-260">POST 및 PATCH 요청이 반드시 idempotent가 된다는 보장은 없습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-260">POST and PATCH requests are not guaranteed to be idempotent.</span></span>

## <a name="conform-to-http-semantics"></a><span data-ttu-id="73a47-261">HTTP 의미 체계 준수</span><span class="sxs-lookup"><span data-stu-id="73a47-261">Conform to HTTP semantics</span></span>

<span data-ttu-id="73a47-262">이 섹션에서는 HTTP 사양을 준수하는 API 디자인에 대한 몇 가지 일반적인 고려 사항을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-262">This section describes some typical considerations for designing an API that conforms to the HTTP specification.</span></span> <span data-ttu-id="73a47-263">그러나 가능한 모든 세부 정보 또는 시나리오를 다루지는 않습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-263">However, it doesn't cover every possible detail or scenario.</span></span> <span data-ttu-id="73a47-264">궁금한 점은 HTTP 사양을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="73a47-264">When in doubt, consult the HTTP specifications.</span></span>

### <a name="media-types"></a><span data-ttu-id="73a47-265">미디어 유형</span><span class="sxs-lookup"><span data-stu-id="73a47-265">Media types</span></span>

<span data-ttu-id="73a47-266">앞서 언급했듯이, 클라이언트와 서버는 리소스 표현을 교환합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-266">As mentioned earlier, clients and servers exchange representations of resources.</span></span> <span data-ttu-id="73a47-267">예를 들어 POST 요청에서는 요청 본문에 만들 리소스의 표현이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-267">For example, in a POST request, the request body contains a representation of the resource to create.</span></span> <span data-ttu-id="73a47-268">GET 요청에서는 응답 본문에 가져온 리소스의 표현이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-268">In a GET request, the response body contains a representation of the fetched resource.</span></span>

<span data-ttu-id="73a47-269">HTTP 프로토콜에서 형식은 MIME 유형이라고도 하는 *미디어 유형*을 사용하여 지정됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-269">In the HTTP protocol, formats are specified through the use of *media types*, also called MIME types.</span></span> <span data-ttu-id="73a47-270">이진이 아닌 데이터의 경우 대부분의 Web API는 JSON(미디어 유형 = 응용 프로그램/json) 및 XML(미디어 유형 = 응용 프로그램/xml)을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-270">For non-binary data, most web APIs support JSON (media type = application/json) and possibly XML (media type = application/xml).</span></span> 

<span data-ttu-id="73a47-271">요청 또는 응답의 Content-Type 헤더는 표현 형식을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-271">The Content-Type header in a request or response specifies the format of the representation.</span></span> <span data-ttu-id="73a47-272">다음은 JSON 데이터를 포함하는 POST 요청의 예입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-272">Here is an example of a POST request that includes JSON data:</span></span>

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
Content-Type: application/json; charset=utf-8
Content-Length: 57

{"Id":1,"Name":"Gizmo","Category":"Widgets","Price":1.99}
```

<span data-ttu-id="73a47-273">서버에서 미디어 유형을 지원하지 않으면 HTTP 상태 코드 415(지원되지 않는 미디어 유형)를 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-273">If the server doesn't support the media type, it should return HTTP status code 415 (Unsupported Media Type).</span></span>

<span data-ttu-id="73a47-274">클라이언트 요청에는 클라이언트가 응답 메시지에서 서버로부터 받는 미디어 유형 목록을 포함하는 Accept 헤더가 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-274">A client request can include an Accept header that contains a list of media types the client will accept from the server in the response message.</span></span> <span data-ttu-id="73a47-275">예: </span><span class="sxs-lookup"><span data-stu-id="73a47-275">For example:</span></span>

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
Accept: application/json
```

<span data-ttu-id="73a47-276">서버가 나열된 미디어 유형 중 어떤 것도 일치시킬 수 없는 경우 HTTP 상태 코드 406(허용되지 않음)을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-276">If the server cannot match any of the media type(s) listed, it should return HTTP status code 406 (Not Acceptable).</span></span> 

### <a name="get-methods"></a><span data-ttu-id="73a47-277">GET 메서드</span><span class="sxs-lookup"><span data-stu-id="73a47-277">GET methods</span></span>

<span data-ttu-id="73a47-278">성공적인 GET 메서드는 일반적으로 HTTP 상태 코드 200(정상)를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-278">A successful GET method typically returns HTTP status code 200 (OK).</span></span> <span data-ttu-id="73a47-279">리소스를 찾을 수 없는 경우 메서드가 404(찾을 수 없음)를 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-279">If the resource cannot be found, the method should return 404 (Not Found).</span></span>

### <a name="post-methods"></a><span data-ttu-id="73a47-280">POST 메서드</span><span class="sxs-lookup"><span data-stu-id="73a47-280">POST methods</span></span>

<span data-ttu-id="73a47-281">POST 메서드는 새 리소스를 만드는 경우 HTTP 상태 코드 201(만들어짐)을 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-281">If a POST method creates a new resource, it returns HTTP status code 201 (Created).</span></span> <span data-ttu-id="73a47-282">새 리소스의 URI는 응답의 Location 헤더에 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-282">The URI of the new resource is included in the Location header of the response.</span></span> <span data-ttu-id="73a47-283">응답 본문은 리소스의 표현을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-283">The response body contains a representation of the resource.</span></span>

<span data-ttu-id="73a47-284">이 메서드가 일부 처리를 수행하지만 새 리소스를 만들지 않는 경우 메서드는 HTTP 상태 코드 200을 반환하고 작업의 결과를 응답 본문에 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-284">If the method does some processing but does not create a new resource, the method can return HTTP status code 200 and include the result of the operation in the response body.</span></span> <span data-ttu-id="73a47-285">또는 반환할 결과가 없으면 메서드가 응답 본문 없이 HTTP 상태 코드 204(내용 없음)를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-285">Alternatively, if there is no result to return, the method can return HTTP status code 204 (No Content) with no response body.</span></span>

<span data-ttu-id="73a47-286">클라이언트가 잘못된 데이터를 요청에 배치하면 서버에서 HTTP 상태 코드 400(잘못된 요청)을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-286">If the client puts invalid data into the request, the server should return HTTP status code 400 (Bad Request).</span></span> <span data-ttu-id="73a47-287">응답 본문에는 오류에 대한 추가 정보 또는 자세한 정보를 제공하는 URI 링크가 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-287">The response body can contain additional information about the error or a link to a URI that provides more details.</span></span>

### <a name="put-methods"></a><span data-ttu-id="73a47-288">PUT 메서드</span><span class="sxs-lookup"><span data-stu-id="73a47-288">PUT methods</span></span>

<span data-ttu-id="73a47-289">PUT 메서드는 POST 메서드와 마찬가지로 새 리소스를 만드는 경우 HTTP 상태 코드 201(만들어짐)을 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-289">If a PUT method creates a new resource, it returns HTTP status code 201 (Created), as with a POST method.</span></span> <span data-ttu-id="73a47-290">이 메서드는 기존 리소스를 업데이트할 경우 200(정상) 또는 204(내용 없음)를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-290">If the method updates an existing resource, it returns either 200 (OK) or 204 (No Content).</span></span> <span data-ttu-id="73a47-291">상황에 따라 기존 리소스를 업데이트할 수 없는 경우도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-291">In some cases, it might not be possible to update an existing resource.</span></span> <span data-ttu-id="73a47-292">이 경우 HTTP 상태 코드 409(충돌)를 반환하는 방안을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-292">In that case, consider returning HTTP status code 409 (Conflict).</span></span> 

<span data-ttu-id="73a47-293">컬렉션의 복수 리소스에 대한 업데이트를 일괄 처리할 수 있는 일괄 HTTP PUT 작업의 구현을 생각해 보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-293">Consider implementing bulk HTTP PUT operations that can batch updates to multiple resources in a collection.</span></span> <span data-ttu-id="73a47-294">PUT 요청은 컬렉션의 URI를 지정해야 하며, 요청 본문에 수정할 리소스의 세부 정보를 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-294">The PUT request should specify the URI of the collection, and the request body should specify the details of the resources to be modified.</span></span> <span data-ttu-id="73a47-295">이 접근 방식은 데이터 전송량을 줄이고 성능을 향상시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-295">This approach can help to reduce chattiness and improve performance.</span></span>

### <a name="patch-methods"></a><span data-ttu-id="73a47-296">PATCH 메서드</span><span class="sxs-lookup"><span data-stu-id="73a47-296">PATCH methods</span></span>

<span data-ttu-id="73a47-297">클라이언트는 PATCH 요청을 사용하여 업데이트를 *패치 문서*의 형태로 기존 리소스에 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-297">With a PATCH request, the client sends a set of updates to an existing resource, in the form of a *patch document*.</span></span> <span data-ttu-id="73a47-298">서버는 패치 문서를 처리하여 업데이트를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-298">The server processes the patch document to perform the update.</span></span> <span data-ttu-id="73a47-299">패치 문서는 리소스 전체가 아니라 적용할 변경 내용만 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-299">The patch document doesn't describe the whole resource, only a set of changes to apply.</span></span> <span data-ttu-id="73a47-300">PATCH 메서드에 대한 사양([RFC 5789](https://tools.ietf.org/html/rfc5789))은 패치 문서에 대한 특정 형식을 정의하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-300">The specification for the PATCH method ([RFC 5789](https://tools.ietf.org/html/rfc5789)) doesn't define a particular format for patch documents.</span></span> <span data-ttu-id="73a47-301">형식은 요청의 미디어 형식에서 유추해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-301">The format must be inferred from the media type in the request.</span></span>

<span data-ttu-id="73a47-302">Web API에 대한 가장 일반적인 데이터 형식은 JSON일 것입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-302">JSON is probably the most common data format for web APIs.</span></span> <span data-ttu-id="73a47-303">두 가지 주요 JSON 기반 패치 형식으로 *JSON 패치* 및 *JSON 병합 패치*가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-303">There are two main JSON-based patch formats, called *JSON patch* and *JSON merge patch*.</span></span>

<span data-ttu-id="73a47-304">JSON 병합 패치는 비교적 간단합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-304">JSON merge patch is somewhat simpler.</span></span> <span data-ttu-id="73a47-305">패치 문서는 원래 JSON 리소스와 동일한 구조를 갖지만 변경 또는 추가할 필드의 하위 집합만 포함하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-305">The patch document has the same structure as the original JSON resource, but includes just the subset of fields that should be changed or added.</span></span> <span data-ttu-id="73a47-306">또한 패치 문서에서 필드 값에 대해 `null`을 지정하여 필드를 삭제할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-306">In addition, a field can be deleted by specifying `null` for the field value in the patch document.</span></span> <span data-ttu-id="73a47-307">(즉, 원래 리소스가 명시적 null 값을 가질 수 있으면 병합 패치가 적합하지 않습니다.)</span><span class="sxs-lookup"><span data-stu-id="73a47-307">(That means merge patch is not suitable if the original resource can have explicit null values.)</span></span>

<span data-ttu-id="73a47-308">예를 들어 원래 리소스의 JSON 표현은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-308">For example, suppose the original resource has the following JSON representation:</span></span>

```json
{ 
    "name":"gizmo",
    "category":"widgets",
    "color":"blue",
    "price":10
}
```

<span data-ttu-id="73a47-309">이 리소스에 가능한 JSON 병합 패치는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-309">Here is a possible JSON merge patch for this resource:</span></span>

```json
{ 
    "price":12,
    "color":null,
    "size":"small"
}
```

<span data-ttu-id="73a47-310">이것은 서버에 "가격"을 업데이트하고, "색"을 삭제하고, "크기"를 추가하라고 알려줍니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-310">This tells the server to update "price", delete "color", and add "size".</span></span> <span data-ttu-id="73a47-311">"이름" 및 "범주"는 수정되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-311">"Name" and "category" are not modified.</span></span> <span data-ttu-id="73a47-312">JSON 병합 패치의 정확한 세부 정보는 [RFC 7396](https://tools.ietf.org/html/rfc7396)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="73a47-312">For the exact details of JSON merge patch, see [RFC 7396](https://tools.ietf.org/html/rfc7396).</span></span> <span data-ttu-id="73a47-313">JSON 병합 패치에 대한 미디어 유형은 "application/merge-patch+json"입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-313">The media type for JSON merge patch is "application/merge-patch+json".</span></span>

<span data-ttu-id="73a47-314">원래 리소스가 명시적 null 값을 포함할 수 있으면 패치 문서에서 `null`이 갖는 특별한 의미 때문에 병합 패치가 적합하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-314">Merge patch is not suitable if the original resource can contain explicit null values, due to the special meaning of `null` in the patch document.</span></span> <span data-ttu-id="73a47-315">또한 패치 문서는 서버에서 업데이트를 적용할 순서를 지정하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-315">Also, the patch document doesn't specify the order that the server should apply the updates.</span></span> <span data-ttu-id="73a47-316">데이터 및 도메인에 따라 이것이 중요할 수도 있고 중요하지 않을 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-316">That may or may not matter, depending on the data and the domain.</span></span> <span data-ttu-id="73a47-317">[RFC 6902](https://tools.ietf.org/html/rfc6902)에 정의된 JSON 패치는 좀 더 유연합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-317">JSON patch, defined in [RFC 6902](https://tools.ietf.org/html/rfc6902), is more flexible.</span></span> <span data-ttu-id="73a47-318">작업의 결과로 적용할 변경 내용을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-318">It specifies the changes as a sequence of operations to apply.</span></span> <span data-ttu-id="73a47-319">작업에는 추가, 제거, 바꾸기, 복사 및 테스트(값의 유효성 검사)가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-319">Operations include add, remove, replace, copy, and test (to validate values).</span></span> <span data-ttu-id="73a47-320">JSON 패치에 대한 미디어 유형은 "application/json-patch+json"입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-320">The media type for JSON patch is "application/json-patch+json".</span></span>

<span data-ttu-id="73a47-321">다음은 적절한 HTTP 상태 코드와 함께 PATCH 요청을 처리할 때 발생할 수 있는 몇 가지 일반적인 오류 조건입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-321">Here are some typical error conditions that might be encountered when processing a PATCH request, along with the appropriate HTTP status code.</span></span>

| <span data-ttu-id="73a47-322">오류 조건</span><span class="sxs-lookup"><span data-stu-id="73a47-322">Error condition</span></span> | <span data-ttu-id="73a47-323">HTTP 상태 코드</span><span class="sxs-lookup"><span data-stu-id="73a47-323">HTTP status code</span></span> |
|-----------|------------|
| <span data-ttu-id="73a47-324">지원되지 않는 패치 문서 형식입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-324">The patch document format isn't supported.</span></span> | <span data-ttu-id="73a47-325">415(지원되지 않는 미디어 형식)</span><span class="sxs-lookup"><span data-stu-id="73a47-325">415 (Unsupported Media Type)</span></span> |
| <span data-ttu-id="73a47-326">패치 문서의 형식이 잘못되었습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-326">Malformed patch document.</span></span> | <span data-ttu-id="73a47-327">400(잘못된 요청)</span><span class="sxs-lookup"><span data-stu-id="73a47-327">400 (Bad Request)</span></span> |
| <span data-ttu-id="73a47-328">패치 문서가 유효하지만 현재 상태에서는 변경 내용을 리소스에 적용할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-328">The patch document is valid, but the changes can't be applied to the resource in its current state.</span></span> | <span data-ttu-id="73a47-329">409(충돌)</span><span class="sxs-lookup"><span data-stu-id="73a47-329">409 (Conflict)</span></span>

### <a name="delete-methods"></a><span data-ttu-id="73a47-330">DELETE 메서드</span><span class="sxs-lookup"><span data-stu-id="73a47-330">DELETE methods</span></span>

<span data-ttu-id="73a47-331">삭제 작업이 성공하면 웹 서버는 프로세스가 성공적으로 처리되었지만 응답 본문에 추가 정보가 포함되지 않았음을 나타내는 HTTP 204 상태 코드로 응답해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-331">If the delete operation is successful, the web server should respond with HTTP status code 204, indicating that the process has been successfully handled, but that the response body contains no further information.</span></span> <span data-ttu-id="73a47-332">리소스가 없는 경우 웹 서버는 HTTP 404(찾을 수 없음)를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-332">If the resource doesn't exist, the web server can return HTTP 404 (Not Found).</span></span>

### <a name="asynchronous-operations"></a><span data-ttu-id="73a47-333">비동기 작업</span><span class="sxs-lookup"><span data-stu-id="73a47-333">Asynchronous operations</span></span>

<span data-ttu-id="73a47-334">경우에 따라 POST, PUT, PATCH 또는 DELETE 작업을 수행하려면 완료하는 데 약간 시간이 걸리는 처리 작업이 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-334">Sometimes a POST, PUT, PATCH, or DELETE operation might require processing that takes awhile to complete.</span></span> <span data-ttu-id="73a47-335">처리 작업이 완료될 때까지 기다렸다가 클라이언트에 응답을 보내는 경우 허용되지 않는 수준의 대기 시간이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-335">If you wait for completion before sending a response to the client, it may cause unacceptable latency.</span></span> <span data-ttu-id="73a47-336">이 경우 비동기 작업을 수행하는 방안을 고려해 보아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-336">If so, consider making the operation asynchronous.</span></span> <span data-ttu-id="73a47-337">요청 처리가 수락되었지만 아직 완료되지 않았음을 나타내는 HTTP 상태 코드 202(수락됨)를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-337">Return HTTP status code 202 (Accepted) to indicate the request was accepted for processing but is not completed.</span></span> 

<span data-ttu-id="73a47-338">클라이언트가 상태 엔드포인트를 폴링하여 상태를 모니터링할 수 있도록 비동기 요청의 상태를 반환하는 엔드포인트를 표시해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-338">You should expose an endpoint that returns the status of an asynchronous request, so the client can monitor the status by polling the status endpoint.</span></span> <span data-ttu-id="73a47-339">202 응답의 Location 헤더에 상태 엔드포인트의 URI를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-339">Include the URI of the status endpoint in the Location header of the 202 response.</span></span> <span data-ttu-id="73a47-340">예: </span><span class="sxs-lookup"><span data-stu-id="73a47-340">For example:</span></span>

```http
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

<span data-ttu-id="73a47-341">클라이언트가 이 엔드포인트에 GET 요청을 보내는 경우 응답에 요청의 현재 상태가 포함되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-341">If the client sends a GET request to this endpoint, the response should contain the current status of the request.</span></span> <span data-ttu-id="73a47-342">필요에 따라 예상 완료 시간 또는 작업 취소 링크를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-342">Optionally, it could also include an estimated time to completion or a link to cancel the operation.</span></span> 

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345"
}
```

<span data-ttu-id="73a47-343">비동기 작업에서 새 리소스를 만드는 경우 작업 완료 후 상태 엔드포인트에서 상태 코드 303(다른 항목 보기)을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-343">If the asynchronous operation creates a new resource, the status endpoint should return status code 303 (See Other) after the operation completes.</span></span> <span data-ttu-id="73a47-344">303 응답에 새 리소스의 URI를 제공하는 Location 헤더를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-344">In the 303 response, include a Location header that gives the URI of the new resource:</span></span>

```http
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

<span data-ttu-id="73a47-345">자세한 내용은 [REST의 비동기 작업](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="73a47-345">For more information, see [Asynchronous operations in REST](https://www.adayinthelifeof.nl/2011/06/02/asynchronous-operations-in-rest/).</span></span>

## <a name="filter-and-paginate-data"></a><span data-ttu-id="73a47-346">데이터 필터링 및 페이지 매기기</span><span class="sxs-lookup"><span data-stu-id="73a47-346">Filter and paginate data</span></span>

<span data-ttu-id="73a47-347">단일 URI를 통해 리소스 컬렉션을 표시하면 정보의 하위 집합만 필요할 때에도 응용 프로그램이 대량의 데이터를 가져올 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-347">Exposing a collection of resources through a single URI can lead to applications fetching large amounts of data when only a subset of the information is required.</span></span> <span data-ttu-id="73a47-348">예를 들어 클라이언트 응용 프로그램에서 비용이 특정 값을 초과하는 모든 주문을 찾아야 한다고 가정해 봅시다.</span><span class="sxs-lookup"><span data-stu-id="73a47-348">For example, suppose a client application needs to find all orders with a cost over a specific value.</span></span> <span data-ttu-id="73a47-349">클라이언트 응용 프로그램은 */orders* URI에서 모든 주문을 검색한 후 클라이언트 쪽에서 이러한 주문을 필터링할 것입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-349">It might retrieve all orders from the */orders* URI and then filter these orders on the client side.</span></span> <span data-ttu-id="73a47-350">이 프로세스는 매우 비효율적입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-350">Clearly this process is highly inefficient.</span></span> <span data-ttu-id="73a47-351">Web API를 호스팅하는 서버의 네트워크 대역폭 및 처리 성능이 낭비됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-351">It wastes network bandwidth and processing power on the server hosting the web API.</span></span>

<span data-ttu-id="73a47-352">이 방법 대신, */orders?minCost=n*처럼 API가 URI의 쿼리 문자열에서 필터 전달을 허용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-352">Instead, the API can allow passing a filter in the query string of the URI, such as */orders?minCost=n*.</span></span> <span data-ttu-id="73a47-353">그러면 Web API가 쿼리 문자열의 `minCost` 매개 변수를 구문 분석 및 처리하고 서버 쪽에서 필터링된 결과를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-353">The web API is then responsible for parsing and handling the `minCost` parameter in the query string and returning the filtered results on the sever side.</span></span> 

<span data-ttu-id="73a47-354">컬렉션 리소스에 대한 GET 요청은 다수의 항목을 반환할 가능성이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-354">GET requests over collection resources can potentially return a large number of items.</span></span> <span data-ttu-id="73a47-355">단일 요청에서 반환하는 데이터의 양이 제한되도록 Web API를 디자인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-355">You should design a web API to limit the amount of data returned by any single request.</span></span> <span data-ttu-id="73a47-356">검색할 최대 항목 수와 컬렉션의 시작 오프셋을 지정하는 쿼리 문자열을 지원하는 방안을 고려해 봅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-356">Consider supporting query strings that specify the maximum number of items to retrieve and a starting offset into the collection.</span></span> <span data-ttu-id="73a47-357">예: </span><span class="sxs-lookup"><span data-stu-id="73a47-357">For example:</span></span>

```
/orders?limit=25&offset=50
```

<span data-ttu-id="73a47-358">또한 서비스 거부 공격을 방지하기 위해 반환되는 항목 수를 제한하는 방안도 고려해 봅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-358">Also consider imposing an upper limit on the number of items returned, to help prevent Denial of Service attacks.</span></span> <span data-ttu-id="73a47-359">클라이언트 응용 프로그램을 돕기 위해, 페이지가 매겨진 데이터를 반환하는 GET 요청은 컬렉션의 사용할 수 있는 총 리소스 수를 나타내는 모종의 메타데이터 형식을 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-359">To assist client applications, GET requests that return paginated data should also include some form of metadata that indicate the total number of resources available in the collection.</span></span> <span data-ttu-id="73a47-360">다른 지능적인 페이지 매김 방법을 생각할 수도 있으며, 자세한 내용은 [API 설계 참고 사항: 스마트 페이지 매김](http://bizcoder.com/api-design-notes-smart-paging)</span><span class="sxs-lookup"><span data-stu-id="73a47-360">You might also consider other intelligent paging strategies; for more information, see [API Design Notes: Smart Paging](http://bizcoder.com/api-design-notes-smart-paging)</span></span>

<span data-ttu-id="73a47-361">필드 이름을 */orders?sort=ProductID* 같은 값으로 가져오는 정렬 매개 변수를 제공하여 데이터를 가져올 때 데이터를 정렬하는 비슷한 전략을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-361">You can use a similar strategy to sort data as it is fetched, by providing a sort parameter that takes a field name as the value, such as */orders?sort=ProductID*.</span></span> <span data-ttu-id="73a47-362">그러나 쿼리 문자열 매개 변수는 여러 캐시 구현에서 캐시된 데이터의 키로 사용되는 리소스 식별자의 일부를 구성하기 때문에 이 접근 방식은 캐싱에 나쁜 영향을 미칠 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-362">However, this approach can have a negative effect on caching, because query string parameters form part of the resource identifier used by many cache implementations as the key to cached data.</span></span>

<span data-ttu-id="73a47-363">각 항목에 대량의 데이터가 포함된 경우 각 항목에 대해 반환되는 필드를 제한하도록 이 접근 방식을 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-363">You can extend this approach to limit the fields returned for each item, if each item contains a large amount of data.</span></span> <span data-ttu-id="73a47-364">예를 들어 쉼표로 구분된 필드 목록을 수락하는 */orders?fields=ProductID,Quantity* 같은 쿼리 문자열 매개 변수를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-364">For example, you could use a query string parameter that accepts a comma-delimited list of fields, such as */orders?fields=ProductID,Quantity*.</span></span> 

<span data-ttu-id="73a47-365">쿼리 문자열의 모든 선택적 매개 변수에 의미 있는 기본값을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-365">Give all optional parameters in query strings meaningful defaults.</span></span> <span data-ttu-id="73a47-366">예를 들어 페이지 매김을 구현하는 경우 `limit` 매개 변수를 10으로, `offset` 매개 변수를 0으로 설정하고, 주문을 구현하는 경우 정렬 매개 변수를 리소스의 키로 설정하고, 프로젝션을 지원하는 경우 `fields` 매개 변수를 리소스의 모든 필드로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-366">For example, set the `limit` parameter to 10 and the `offset` parameter to 0 if you implement pagination, set the sort parameter to the key of the resource if you implement ordering, and set the `fields` parameter to all fields in the resource if you support projections.</span></span>

## <a name="support-partial-responses-for-large-binary-resources"></a><span data-ttu-id="73a47-367">대용량 이진 리소스에 대한 부분 응답 지원</span><span class="sxs-lookup"><span data-stu-id="73a47-367">Support partial responses for large binary resources</span></span>

<span data-ttu-id="73a47-368">리소스에 파일 또는 이미지 같은 대용량 이진 필드가 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-368">A resource may contain large binary fields, such as files or images.</span></span> <span data-ttu-id="73a47-369">신뢰할 수 없는 간헐적 연결에 의해 야기되는 문제를 해결하고 응답 시간을 개선하려면 이러한 리소스를 청크로 검색할 수 있게 하는 방안을 고려해 보세요.</span><span class="sxs-lookup"><span data-stu-id="73a47-369">To overcome problems caused by unreliable and intermittent connections and to improve response times, consider enabling such resources to be retrieved in chunks.</span></span> <span data-ttu-id="73a47-370">이렇게 하려면 Web API가 큰 리소스의 GET 요청에 대해 Accept-Ranges 헤더를 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-370">To do this, the web API should support the Accept-Ranges header for GET requests for large resources.</span></span> <span data-ttu-id="73a47-371">이 헤더는 GET 작업이 부분 요청을 지원한다는 것을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-371">This header indicates that the GET operation supports partial requests.</span></span> <span data-ttu-id="73a47-372">클라이언트 응용 프로그램은 바이트 범위로 지정된 리소스 하위 집합을 반환하는 GET 요청을 제출할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-372">The client application can submit GET requests that return a subset of a resource, specified as a range of bytes.</span></span> 

<span data-ttu-id="73a47-373">또한 이러한 리소스에 대해 HTTP HEAD 요청을 구현하는 방안을 고려해 봅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-373">Also, consider implementing HTTP HEAD requests for these resources.</span></span> <span data-ttu-id="73a47-374">HEAD 요청은 리소스에 대해 설명하는 HTTP 헤더만 반환하고 메시지 본문이 비어 있다는 점을 제외하면 GET 요청과 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-374">A HEAD request is similar to a GET request, except that it only returns the HTTP headers that describe the resource, with an empty message body.</span></span> <span data-ttu-id="73a47-375">클라이언트 응용 프로그램은 부분적인 GET 요청을 사용하여 리소스를 가져올지 여부를 결정하는 HEAD 요청을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-375">A client application can issue a HEAD request to determine whether to fetch a resource by using partial GET requests.</span></span> <span data-ttu-id="73a47-376">예: </span><span class="sxs-lookup"><span data-stu-id="73a47-376">For example:</span></span>

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
```

<span data-ttu-id="73a47-377">다음은 응답 메시지 예제입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-377">Here is an example response message:</span></span> 

```HTTP
HTTP/1.1 200 OK

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
```

<span data-ttu-id="73a47-378">Content-Length 헤더는 총 리소스 크기를 제공하고, Accept-Ranges 헤더는 해당 GET 작업이 일부 결과를 지원함을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-378">The Content-Length header gives the total size of the resource, and the Accept-Ranges header indicates that the corresponding GET operation supports partial results.</span></span> <span data-ttu-id="73a47-379">클라이언트 응용 프로그램은 이 정보를 사용하여 더 작은 청크에서 이미지를 검색할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-379">The client application can use this information to retrieve the image in smaller chunks.</span></span> <span data-ttu-id="73a47-380">첫 번째 요청은 범위 헤더를 사용하여 처음 2500 바이트를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-380">The first request fetches the first 2500 bytes by using the Range header:</span></span>

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
```

<span data-ttu-id="73a47-381">응답 메시지는 HTTP 상태 코드 206을 반환하여 이 응답이 부분 응답임을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-381">The response message indicates that this is a partial response by returning HTTP status code 206.</span></span> <span data-ttu-id="73a47-382">Content-Length 헤더는 메시지 본문에 반환된 실제 바이트 수(리소스의 크기가 아닌)를 지정하며, Content-Range 헤더는 해당 바이트가 리소스의 어느 부분인지(4580 바이트 중 바이트 0-2499)를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-382">The Content-Length header specifies the actual number of bytes returned in the message body (not the size of the resource), and the Content-Range header indicates which part of the resource this is (bytes 0-2499 out of 4580):</span></span>

```HTTP
HTTP/1.1 206 Partial Content

Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580

[...]
```

<span data-ttu-id="73a47-383">클라이언트 응용 프로그램의 후속 요청은 리소스의 나머지 부분을 검색할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-383">A subsequent request from the client application can retrieve the remainder of the resource.</span></span>

## <a name="use-hateoas-to-enable-navigation-to-related-resources"></a><span data-ttu-id="73a47-384">HATEOAS를 사용하여 관련 리소스 탐색</span><span class="sxs-lookup"><span data-stu-id="73a47-384">Use HATEOAS to enable navigation to related resources</span></span>

<span data-ttu-id="73a47-385">REST를 실행하는 기본적인 동기 중 하나는 URI 체계에 대해 미리 알고 있지 않아도 전체 리소스 집합을 탐색할 수 있어야 하기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-385">One of the primary motivations behind REST is that it should be possible to navigate the entire set of resources without requiring prior knowledge of the URI scheme.</span></span> <span data-ttu-id="73a47-386">각 HTTP GET 요청은 응답에 포함된 하이퍼링크를 통해 요청된 개체와 직접 관련된 리소스를 찾는 데 필요한 정보를 반환해야 하며, 이러한 각 리소스에 대해 사용할 수 있는 작업을 설명하는 정보도 제공되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-386">Each HTTP GET request should return the information necessary to find the resources related directly to the requested object through hyperlinks included in the response, and it should also be provided with information that describes the operations available on each of these resources.</span></span> <span data-ttu-id="73a47-387">이 원칙을 HATEOAS(Hypertext as the Engine of Application State)라 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-387">This principle is known as HATEOAS, or Hypertext as the Engine of Application State.</span></span> <span data-ttu-id="73a47-388">시스템은 실질적으로 유한 상태 시스템으로서, 각 요청에 대한 응답은 한 상태에서 다른 상태로 바꾸는 데 필요한 정보를 포함하고 있으며, 다른 정보는 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-388">The system is effectively a finite state machine, and the response to each request contains the information necessary to move from one state to another; no other information should be necessary.</span></span>

> [!NOTE]
> <span data-ttu-id="73a47-389">현재 HATEOAS 원칙을 모델링하는 방법을 정의하는 표준 또는 사양은 없습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-389">Currently there are no standards or specifications that define how to model the HATEOAS principle.</span></span> <span data-ttu-id="73a47-390">이 섹션에 나오는 예제에서는 한 가지 가능한 해결 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-390">The examples shown in this section illustrate one possible solution.</span></span>
>
>

<span data-ttu-id="73a47-391">예를 들어 주문과 고객 간의 관계를 처리하기 위해 주문 고객에게 사용 가능한 작업을 식별하는 링크를 주문 표현에 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-391">For example, to handle the relationship between an order and a customer, the representation of an order could include links that identify the available operations for the customer of the order.</span></span> <span data-ttu-id="73a47-392">다음은 가능한 표현입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-392">Here is a possible representation:</span></span> 

```json
{
  "orderID":3,
  "productID":2,
  "quantity":4,
  "orderValue":16.60,
  "links":[
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3", 
      "action":"GET",
      "types":["text/xml","application/json"] 
    },
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"customer",
      "href":"http://adventure-works.com/customers/3",
      "action":"DELETE",
      "types":[]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"GET",
      "types":["text/xml","application/json"]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"PUT",
      "types":["application/x-www-form-urlencoded"]
    },
    {
      "rel":"self",
      "href":"http://adventure-works.com/orders/3", 
      "action":"DELETE",
      "types":[]
    }]
}
```

<span data-ttu-id="73a47-393">이 예에서 `links` 배열에는 링크 집합이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-393">In this example, the `links` array has a set of links.</span></span> <span data-ttu-id="73a47-394">각 링크는 관련 엔터티에 대한 작업을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-394">Each link represents an operation on a related entity.</span></span> <span data-ttu-id="73a47-395">각 링크의 데이터에는 관계("고객"), URI(`http://adventure-works.com/customers/3`), HTTP 메서드 및 지원되는 MIME 형식이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-395">The data for each link includes the relationship ("customer"), the URI (`http://adventure-works.com/customers/3`), the HTTP method, and the supported MIME types.</span></span> <span data-ttu-id="73a47-396">이 모든 정보가 있어야 클라이언트 응용 프로그램이 작업을 호출할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-396">This is all the information that a client application needs to be able to invoke the operation.</span></span> 

<span data-ttu-id="73a47-397">또한 `links` 배열은 검색된 리소스 자체에 대한 자체 참조 정보를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-397">The `links` array also includes self-referencing information about the resource itself that has been retrieved.</span></span> <span data-ttu-id="73a47-398">이러한 관계가 *자체* 관계입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-398">These have the relationship *self*.</span></span>

<span data-ttu-id="73a47-399">리소스의 상태에 따라 반환되는 링크 집합이 달라질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-399">The set of links that are returned may change, depending on the state of the resource.</span></span> <span data-ttu-id="73a47-400">이것이 바로 "응용 프로그램 상태 엔진"이라는 하이퍼텍스트가 의미하는 바입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-400">This is what is meant by hypertext being the "engine of application state."</span></span>

## <a name="versioning-a-restful-web-api"></a><span data-ttu-id="73a47-401">RESTful 웹 API 버전 관리</span><span class="sxs-lookup"><span data-stu-id="73a47-401">Versioning a RESTful web API</span></span>

<span data-ttu-id="73a47-402">Web API가 정적 상태를 유지할 가능성은 매우 낮습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-402">It is highly unlikely that a web API will remain static.</span></span> <span data-ttu-id="73a47-403">비즈니스 요구 사항이 변경됨에 따라 자원의 새 컬렉션이 추가될 수 있으므로, 리소스 간의 관계가 변할 수 있으며 리소스 데이터의 구조가 수정될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-403">As business requirements change new collections of resources may be added, the relationships between resources might change, and the structure of the data in resources might be amended.</span></span> <span data-ttu-id="73a47-404">웹 API를 새로운 또는 서로 다른 요구 사항을 처리하도록 업데이트하는 동안 해당 변경이 웹 API를 사용하는 클라이언트 응용 프로그램에 미치는 영향을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-404">While updating a web API to handle new or differing requirements is a relatively straightforward process, you must consider the effects that such changes will have on client applications consuming the web API.</span></span> <span data-ttu-id="73a47-405">문제는 개발자가 해당 API를 완전히 제어할 수 있는 웹 API를 디자인 및 구현하더라도, 해당 개발자는 원격으로 작업하는 제3자 조직이 구축할 수 있는 클라이언트 응용 프로그램을 같은 정도로 제어하지 못한다는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-405">The issue is that although the developer designing and implementing a web API has full control over that API, the developer does not have the same degree of control over client applications which may be built by third party organizations operating remotely.</span></span> <span data-ttu-id="73a47-406">따라서 새 클라이언트 응용 프로그램이 새 기능과 리소스의 장점을 이용할 수 있도록 하면서도 기존 클라이언트 응용 프로그램이 변경되지 않고 계속 작동할 수 있도록 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-406">The primary imperative is to enable existing client applications to continue functioning unchanged while allowing new client applications to take advantage of new features and resources.</span></span>

<span data-ttu-id="73a47-407">버전 관리를 사용하면 웹 API는 자신이 표시하는 기능과 리소스를 나타낼 수 있으며, 클라이언트 응용 프로그램은 기능 또는 리소스의 특정 버전으로 지정된 요청을 제출할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-407">Versioning enables a web API to indicate the features and resources that it exposes, and a client application can submit requests that are directed to a specific version of a feature or resource.</span></span> <span data-ttu-id="73a47-408">다음 섹션에서는 각각 자체의 이점과 절충점을 가지고 있는 다양한 접근 방식을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-408">The following sections describe several different approaches, each of which has its own benefits and trade-offs.</span></span>

### <a name="no-versioning"></a><span data-ttu-id="73a47-409">버전 관리 없음</span><span class="sxs-lookup"><span data-stu-id="73a47-409">No versioning</span></span>
<span data-ttu-id="73a47-410">가장 간단한 방법이며 일부 내부 API에 대해 허용될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-410">This is the simplest approach, and may be acceptable for some internal APIs.</span></span> <span data-ttu-id="73a47-411">큰 변화는 새 리소스 또는 새 연결로 나타낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-411">Big changes could be represented as new resources or new links.</span></span>  <span data-ttu-id="73a47-412">기존 리소스에 콘텐츠를 추가해도 이 콘텐츠가 표시될 것으로 예상하지 않은 클라이언트 응용 프로그램은 해당 콘텐츠를 무시할 것이므로 주요 변경 내용이 표시되지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-412">Adding content to existing resources might not present a breaking change as client applications that are not expecting to see this content will simply ignore it.</span></span>

<span data-ttu-id="73a47-413">예를 들어 URI *http://adventure-works.com/customers/3*에 대한 요청은 클라이언트 응용 프로그램이 예상하는 `id`, `name` 및 `address` 필드가 포함된 단일 고객의 세부 정보를 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-413">For example, a request to the URI *http://adventure-works.com/customers/3* should return the details of a single customer containing `id`, `name`, and `address` fields expected by the client application:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> <span data-ttu-id="73a47-414">간단한 설명을 위해 이 섹션에 표시된 예제 응답은 HATEOAS 링크를 포함하고 있지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-414">For simplicity, the example responses shown in this section do not include HATEOAS links.</span></span>
>
>

<span data-ttu-id="73a47-415">`DateCreated` 필드가 고객 리소스의 체계에 추가되면 응답은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-415">If the `DateCreated` field is added to the schema of the customer resource, then the response would look like this:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="73a47-416">기존 클라이언트 응용 프로그램은 인식되지 않은 필드를 무시할 수 있으면 계속 올바르게 작동할 수 있으며, 한편 새 클라이언트 응용 프로그램을 새 필드를 처리하도록 디자인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-416">Existing client applications might continue functioning correctly if they are capable of ignoring unrecognized fields, while new client applications can be designed to handle this new field.</span></span> <span data-ttu-id="73a47-417">그러나 리소스가 더 크게 변경되거나(필드 제거 또는 이름 변경 등) 리소스 간의 관계가 변경된 경우에는 이러한 변화가 주요 변경 내용으로 인식되어 기존 클라이언트 응용 프로그램이 올바르게 작동하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-417">However, if more radical changes to the schema of resources occur (such as removing or renaming fields) or the relationships between resources change then these may constitute breaking changes that prevent existing client applications from functioning correctly.</span></span> <span data-ttu-id="73a47-418">이러한 상황에서는 다음 방법 중 하나를 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-418">In these situations you should consider one of the following approaches.</span></span>

### <a name="uri-versioning"></a><span data-ttu-id="73a47-419">URI 버전 관리</span><span class="sxs-lookup"><span data-stu-id="73a47-419">URI versioning</span></span>
<span data-ttu-id="73a47-420">웹 API를 수정하거나 리소스의 체계를 변경할 때마다 각 리소스의 URI에 버전 번호를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-420">Each time you modify the web API or change the schema of resources, you add a version number to the URI for each resource.</span></span> <span data-ttu-id="73a47-421">앞에서는 기존 URI가 전과 같이 계속 작동하여 원래 체계를 준수하는 리소스를 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-421">The previously existing URIs should continue to operate as before, returning resources that conform to their original schema.</span></span>

<span data-ttu-id="73a47-422">앞의 예제를 확장하여 `address` 필드가 주소의 각 구성 부분을 포함하고 있는 하위 필드(예: `streetAddress`, `city`, `state` 및 `zipCode`)로 재구성된다면, http://adventure-works.com/v2/customers/3:과 같은 버전 번호가 들어 있는 URI를 통해 리소스의 이 버전을 표시할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-422">Extending the previous example, if the `address` field is restructured into sub-fields containing each constituent part of the address (such as `streetAddress`, `city`, `state`, and `zipCode`), this version of the resource could be exposed through a URI containing a version number, such as http://adventure-works.com/v2/customers/3:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="73a47-423">이 버전 관리 메커니즘은 매우 간단하지만 요청을 적절한 끝점으로 라우팅하는 서버에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-423">This versioning mechanism is very simple but depends on the server routing the request to the appropriate endpoint.</span></span> <span data-ttu-id="73a47-424">그러나 여러 번 반복을 통해 웹 API가 성숙해짐에 따라 이 메커니즘을 다룰 수 없게 될 수 있으며 서버가 다양한 버전을 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-424">However, it can become unwieldy as the web API matures through several iterations and the server has to support a number of different versions.</span></span> <span data-ttu-id="73a47-425">또한 엄격히 말해서, 클라이언트 응용 프로그램이 같은 데이터(고객 3)를 가져오므로, URI가 버전에 따라 달라져서는 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-425">Also, from a purist’s point of view, in all cases the client applications are fetching the same data (customer 3), so the URI should not really be different depending on the version.</span></span> <span data-ttu-id="73a47-426">또한 이 체계는 모든 링크가 자신의 URI에 버전 번호를 포함해야 하므로 HATEOAS 구현을 복잡하게 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-426">This scheme also complicates implementation of HATEOAS as all links will need to include the version number in their URIs.</span></span>

### <a name="query-string-versioning"></a><span data-ttu-id="73a47-427">쿼리 문자열 버전 관리</span><span class="sxs-lookup"><span data-stu-id="73a47-427">Query string versioning</span></span>
<span data-ttu-id="73a47-428">복수 URI를 제공하는 대신에 HTTP 요청에 추가된 쿼리 문자열 내에 *http://adventure-works.com/customers/3?version=2* 같은 매개 변수를 사용하여 리소스의 버전을 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-428">Rather than providing multiple URIs, you can specify the version of the resource by using a parameter within the query string appended to the HTTP request, such as *http://adventure-works.com/customers/3?version=2*.</span></span> <span data-ttu-id="73a47-429">버전 매개 변수는 이전 클라이언트 응용 프로그램에서 생략했다면 기본적으로 1과 같은 의미 있는 값입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-429">The version parameter should default to a meaningful value such as 1 if it is omitted by older client applications.</span></span>

<span data-ttu-id="73a47-430">이 접근 방식은 같은 리소스가 언제나 같은 URI에서 검색된다는 의미 체계 장점이 있지만, 쿼리 문자열을 구문 분석하고 해당 HTTP 응답을 다시 보내기 위해 요청을 처리하는 코드에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-430">This approach has the semantic advantage that the same resource is always retrieved from the same URI, but it depends on the code that handles the request to parse the query string and send back the appropriate HTTP response.</span></span> <span data-ttu-id="73a47-431">또한 이 접근 방식은 HATEOAS를 URI 버전 관리 메커니즘으로 구현할 때와 같이 복잡합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-431">This approach also suffers from the same complications for implementing HATEOAS as the URI versioning mechanism.</span></span>

> [!NOTE]
> <span data-ttu-id="73a47-432">일부 구형 웹 브라우저와 웹 프록시는 URI에 쿼리 문자열을 포함하는 요청에 대한 응답을 캐싱하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-432">Some older web browsers and web proxies will not cache responses for requests that include a query string in the URI.</span></span> <span data-ttu-id="73a47-433">이는 웹 API를 사용하고 해당 웹 브라우저 내에서 실행되는 웹 응용 프로그램의 성능을 저하시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-433">This can have an adverse impact on performance for web applications that use a web API and that run from within such a web browser.</span></span>
>
>

### <a name="header-versioning"></a><span data-ttu-id="73a47-434">헤더 버전 관리</span><span class="sxs-lookup"><span data-stu-id="73a47-434">Header versioning</span></span>
<span data-ttu-id="73a47-435">버전 번호를 쿼리 문자열 매개 변수로 추가하지 않고 리소스의 버전을 나타내는 사용자 지정 헤더를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-435">Rather than appending the version number as a query string parameter, you could implement a custom header that indicates the version of the resource.</span></span> <span data-ttu-id="73a47-436">이 접근 방식을 사용하려면 클라이언트 응용 프로그램이 적절한 헤더를 요청에 추가해야 하지만, version 헤더가 생략된 경우 클라이언트 요청을 처리하는 코드가 기본값(버전 1)을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-436">This approach requires that the client application adds the appropriate header to any requests, although the code handling the client request could use a default value (version 1) if the version header is omitted.</span></span> <span data-ttu-id="73a47-437">다음 예제에서는 *Custom-Header*라는 사용자 지정 헤더를 이용합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-437">The following examples utilize a custom header named *Custom-Header*.</span></span> <span data-ttu-id="73a47-438">이 헤더의 값은 웹 API의 버전을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-438">The value of this header indicates the version of web API.</span></span>

<span data-ttu-id="73a47-439">버전 1:</span><span class="sxs-lookup"><span data-stu-id="73a47-439">Version 1:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=1
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="73a47-440">버전 2:</span><span class="sxs-lookup"><span data-stu-id="73a47-440">Version 2:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Custom-Header: api-version=2
```

```HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="73a47-441">참고로 이전의 두 방법 방식과 마찬가지로 HATEOAS를 구현하려면 모든 링크에 적절한 사용자 지정 헤더를 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-441">Note that as with the previous two approaches, implementing HATEOAS requires including the appropriate custom header in any links.</span></span>

### <a name="media-type-versioning"></a><span data-ttu-id="73a47-442">미디어 형식 버전 관리</span><span class="sxs-lookup"><span data-stu-id="73a47-442">Media type versioning</span></span>
<span data-ttu-id="73a47-443">이 지침의 앞부분에서 설명한 대로, 클라이언트 응용 프로그램은 웹 서버에 HTTP GET 요청을 보낼 때 Accept 헤더를 사용하여 처리할 수 있는 콘텐츠의 형식을 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-443">When a client application sends an HTTP GET request to a web server it should stipulate the format of the content that it can handle by using an Accept header, as described earlier in this guidance.</span></span> <span data-ttu-id="73a47-444">흔히 *Accept* 헤더의 목적은 클라이언트 응용 프로그램에서 응답 본문이 XML, JSON 또는 클라이언트가 구문 분석할 수 있는 몇몇 다른 일반적인 형식 중 어느 형식인지 지정할 수 있도록 하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-444">Frequently the purpose of the *Accept* header is to allow the client application to specify whether the body of the response should be XML, JSON, or some other common format that the client can parse.</span></span> <span data-ttu-id="73a47-445">그러나 클라이언트 응용 프로그램이 예상하는 리소스의 버전을 나타낼 수 있도록 하는 정보를 포함한 사용자 지정 미디어 형식을 정의할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-445">However, it is possible to define custom media types that include information enabling the client application to indicate which version of a resource it is expecting.</span></span> <span data-ttu-id="73a47-446">다음 예제는 *Accept* 헤더를 값 *application/vnd.adventure-works.v1+json*과 함께 지정하는 요청을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-446">The following example shows a request that specifies an *Accept* header with the value *application/vnd.adventure-works.v1+json*.</span></span> <span data-ttu-id="73a47-447">*vnd.adventure-works.v1* 요소는 웹 서버에 대해 리소스의 버전 1을 반환해야 한다는 것을 나타내며, 한편 *json* 요소는 응답 본문의 형식이 JSON이어야 함을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-447">The *vnd.adventure-works.v1* element indicates to the web server that it should return version 1 of the resource, while the *json* element specifies that the format of the response body should be JSON:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
Accept: application/vnd.adventure-works.v1+json
```

<span data-ttu-id="73a47-448">요청을 처리하는 코드는 *Accept* 헤더를 처리하고 가능하면 해당 헤더를 적용해야 합니다. 클라이언트 응용 프로그램은 *Accept* 헤더에 복수의 형식을 지정할 수 있으며, 이 경우 웹 서버는 응답 본문에 가장 적절한 형식을 선택할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-448">The code handling the request is responsible for processing the *Accept* header and honoring it as far as possible (the client application may specify multiple formats in the *Accept* header, in which case the web server can choose the most appropriate format for the response body).</span></span> <span data-ttu-id="73a47-449">웹 서버는 Content-Type 헤더를 사용하여 응답 본문에 있는 데이터의 형식을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-449">The web server confirms the format of the data in the response body by using the Content-Type header:</span></span>

```HTTP
HTTP/1.1 200 OK
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8

{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="73a47-450">Accept 헤더가 모든 알려진 미디어 형식을 지정하지 않은 경우, 웹 서버는 HTTP 406(승인 금지) 응답 메시지를 생성하거나 기본 미디어 형식이 포함된 메시지를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-450">If the Accept header does not specify any known media types, the web server could generate an HTTP 406 (Not Acceptable) response message or return a message with a default media type.</span></span>

<span data-ttu-id="73a47-451">이 접근 방식은 엄격히 말해서 버전 관리 메커니즘인지 여부에 대한 논란의 여지가 있으며 당연히 리소스 링크에 관련 데이터의 MIME 형식을 포함할 수 있는 HATEOAS에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-451">This approach is arguably the purest of the versioning mechanisms and lends itself naturally to HATEOAS, which can include the MIME type of related data in resource links.</span></span>

> [!NOTE]
> <span data-ttu-id="73a47-452">버전 관리 전략을 선택할 때에는 성능에 미치는 영향, 특히 웹 서버의 캐싱을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-452">When you select a versioning strategy, you should also consider the implications on performance, especially caching on the web server.</span></span> <span data-ttu-id="73a47-453">URI 버전 관리 및 쿼리 문자열 버전 관리 체계는 같은 URI/쿼리 문자열 조합이 매번 같은 데이터를 참조하므로 캐싱하기에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-453">The URI versioning and Query String versioning schemes are cache-friendly inasmuch as the same URI/query string combination refers to the same data each time.</span></span>
>
> <span data-ttu-id="73a47-454">일반적으로 헤더 버전 관리 및 미디어 형식 버전 관리 메커니즘에는 사용자 지정 헤더 또는 Accept 헤더의 값을 검사하기 위해 추가 논리가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-454">The Header versioning and Media Type versioning mechanisms typically require additional logic to examine the values in the custom header or the Accept header.</span></span> <span data-ttu-id="73a47-455">대규모 환경의 경우, 서로 다른 버전의 웹 API를 사용하는 많은 클라이언트가 서버 쪽 캐시에 상당한 양의 중복된 데이터를 발생시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-455">In a large-scale environment, many clients using different versions of a web API can result in a significant amount of duplicated data in a server-side cache.</span></span> <span data-ttu-id="73a47-456">클라이언트 응용 프로그램이 캐싱을 구현하는 프록시를 통해 웹 서버와 통신하는 경우 이 문제가 심각할 수 있으며, 현재 요청된 데이터의 복사본을 자체의 캐시에 저장하지 않은 경우에만 요청을 웹 서버에 전달해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-456">This issue can become acute if a client application communicates with a web server through a proxy that implements caching, and that only forwards a request to the web server if it does not currently hold a copy of the requested data in its cache.</span></span>
>
>

## <a name="open-api-initiative"></a><span data-ttu-id="73a47-457">Open API Initiative</span><span class="sxs-lookup"><span data-stu-id="73a47-457">Open API Initiative</span></span>
<span data-ttu-id="73a47-458">[Open API Initiative](https://www.openapis.org/)는 공급업체에서 REST API 설명을 표준화하기 위해 업계 컨소시엄에서 만들었습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-458">The [Open API Initiative](https://www.openapis.org/) was created by an industry consortium to standardize REST API descriptions across vendors.</span></span> <span data-ttu-id="73a47-459">이 이니셔티브의 일부로, Swagger 2.0 사양의 명칭이 OAS(Open API Specification)로 바뀐 후 Open API Initiative에 추가되었습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-459">As part of this initiative, the Swagger 2.0 specification was renamed the OpenAPI Specification (OAS) and brought under the Open API Initiative.</span></span>

<span data-ttu-id="73a47-460">사용하는 웹 API에 OpenAPI를 채택할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-460">You may want to adopt OpenAPI for your web APIs.</span></span> <span data-ttu-id="73a47-461">몇 가지 고려할 점은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-461">Some points to consider:</span></span>

- <span data-ttu-id="73a47-462">OpenAPI 사양에는 REST API 디자인 방식에 대한 독자적인 지침이 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-462">The OpenAPI Specification comes with a set of opinionated guidelines on how a REST API should be designed.</span></span> <span data-ttu-id="73a47-463">이 사양은 상호 운용성 측면에서는 이점이 있지만, 사양에 맞게 API를 디자인할 때는 좀 더 주의해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-463">That has advantages for interoperability, but requires more care when designing your API to conform to the specification.</span></span>
- <span data-ttu-id="73a47-464">OpenAPI는 구현 우선 방식이 아닌, 계약 우선 방식을 권장합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-464">OpenAPI promotes a contract-first approach, rather than an implementation-first approach.</span></span> <span data-ttu-id="73a47-465">계약 우선 방식에서는 API 계약(인터페이스)을 먼저 디자인한 후 계약을 구현하는 코드를 작성합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-465">Contract-first means you design the API contract (the interface) first and then write code that implements the contract.</span></span> 
- <span data-ttu-id="73a47-466">Swagger와 같은 도구는 API 계약에서 클라이언트 라이브러리 또는 문서를 생성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-466">Tools like Swagger can generate client libraries or documentation from API contracts.</span></span> <span data-ttu-id="73a47-467">예를 들어, [Swagger를 사용하는 ASP.NET 웹 API 도움말 페이지](/aspnet/core/tutorials/web-api-help-pages-using-swagger)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="73a47-467">For example, see [ASP.NET Web API Help Pages using Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).</span></span>

## <a name="more-information"></a><span data-ttu-id="73a47-468">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="73a47-468">More information</span></span>
* <span data-ttu-id="73a47-469">[Microsoft REST API 지침](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md).</span><span class="sxs-lookup"><span data-stu-id="73a47-469">[Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md).</span></span> <span data-ttu-id="73a47-470">공용 REST API 디자인에 대한 구체적인 권장 사항입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-470">Detailed recommendations for designing public REST APIs.</span></span>
* <span data-ttu-id="73a47-471">[REST Cookbook](http://restcookbook.com/).</span><span class="sxs-lookup"><span data-stu-id="73a47-471">[The REST Cookbook](http://restcookbook.com/).</span></span> <span data-ttu-id="73a47-472">RESTful API를 빌드하는 방법을 소개합니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-472">Introduction to building RESTful APIs.</span></span>
* <span data-ttu-id="73a47-473">[Web API 검사 목록](https://mathieu.fenniak.net/the-api-checklist/).</span><span class="sxs-lookup"><span data-stu-id="73a47-473">[Web API Checklist](https://mathieu.fenniak.net/the-api-checklist/).</span></span> <span data-ttu-id="73a47-474">Web API를 디자인 및 구현할 때 고려해야 하는 항목 목록입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-474">A useful list of items to consider when designing and implementing a web API.</span></span>
* <span data-ttu-id="73a47-475">[Open API Initiative](https://www.openapis.org/).</span><span class="sxs-lookup"><span data-stu-id="73a47-475">[Open API Initiative](https://www.openapis.org/).</span></span> <span data-ttu-id="73a47-476">Open API에 대한 설명서 및 구현 세부 정보입니다.</span><span class="sxs-lookup"><span data-stu-id="73a47-476">Documentation and implementation details on Open API.</span></span>
