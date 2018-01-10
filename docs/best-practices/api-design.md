---
title: "API 디자인 지침"
description: "잘 디자인된 API를 만드는 방법에 관한 지침입니다."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 3ffadce1b0c4a4da808e52d61cff0b7f0b27de11
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="api-design"></a><span data-ttu-id="152e6-103">API 디자인</span><span class="sxs-lookup"><span data-stu-id="152e6-103">API design</span></span>
[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="152e6-104">많은 최신 웹 기반 솔루션은 웹 서버에서 호스트되는 웹 서비스를 사용하여 원격 클라이언트 응용 프로그램의 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-104">Many modern web-based solutions make the use of web services, hosted by web servers, to provide functionality for remote client applications.</span></span> <span data-ttu-id="152e6-105">웹 API는 웹 서비스에 표시되는 작업으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-105">The operations that a web service exposes constitute a web API.</span></span> <span data-ttu-id="152e6-106">잘 디자인된 웹 API는 아래와 같은 특성을 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-106">A well-designed web API should aim to support:</span></span>

* <span data-ttu-id="152e6-107">**플랫폼 독립성**.</span><span class="sxs-lookup"><span data-stu-id="152e6-107">**Platform independence**.</span></span> <span data-ttu-id="152e6-108">클라이언트 응용 프로그램은 API에 표시되는 데이터 또는 작업이 물리적으로 구현된 방법과 상관없이 웹 서비스가 제공하는 API를 이용할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-108">Client applications should be able to utilize the API that the web service provides without requiring how the data or operations that API exposes are physically implemented.</span></span> <span data-ttu-id="152e6-109">즉, API는 클라이언트 응용 프로그램과 웹 서비스가 사용할 데이터 형식 및 클라이언트 응용 프로그램과 웹 서비스 간에 교환되는 데이터의 구조를 일치시킬 수 있는 공통 표준을 적용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-109">This requires that the API abides by common standards that enable a client application and web service to agree on which data formats to use, and the structure of the data that is exchanged between client applications and the web service.</span></span>
* <span data-ttu-id="152e6-110">**서비스 진화**.</span><span class="sxs-lookup"><span data-stu-id="152e6-110">**Service evolution**.</span></span> <span data-ttu-id="152e6-111">웹 서비스는 클라이언트 응용 프로그램과 독립적으로 기능을 진화시키고 추가(또는 제거)할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-111">The web service should be able to evolve and add (or remove) functionality independently from client applications.</span></span> <span data-ttu-id="152e6-112">기존 클라이언트 응용 프로그램은 웹 서비스가 제공하는 기능이 변화하더라도 수정되지 않고 계속 작동할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-112">Existing client applications should be able to continue to operate unmodified as the features provided by the web service change.</span></span> <span data-ttu-id="152e6-113">또한 모든 기능은 클라이언트 응용 프로그램이 해당 기능을 완전히 이용할 수 있도록 검색이 가능해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-113">All functionality should also be discoverable, so that client applications can fully utilize it.</span></span>

<span data-ttu-id="152e6-114">이 지침에서는 웹 API를 디자인할 때 고려해야 하는 문제를 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-114">The purpose of this guidance is to describe the issues that you should consider when designing a web API.</span></span>

## <a name="introduction-to-representational-state-transfer-rest"></a><span data-ttu-id="152e6-115">REST(Representational State Transfer) 소개</span><span class="sxs-lookup"><span data-stu-id="152e6-115">Introduction to Representational State Transfer (REST)</span></span>
<span data-ttu-id="152e6-116">Roy Fielding은 2000년에 자신의 논문에서 웹 서비스에서 표시되는 작업의 구조를 지정하는 대체 아키텍처 접근 방식인 REST를 제안했습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-116">In his dissertation in 2000, Roy Fielding proposed an alternative architectural approach to structuring the operations exposed by web services; REST.</span></span> <span data-ttu-id="152e6-117">REST는 하이퍼미디어 기반 분산 시스템을 구축하기 위한 아키텍처 스타일입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-117">REST is an architectural style for building distributed systems based on hypermedia.</span></span> <span data-ttu-id="152e6-118">REST 모델의 주요 이점은 개방형 표준을 기반으로 하고 있어 해당 모델에 액세스하는 모델 또는 클라이언트 응용 프로그램의 구현이 특정 구현에 바인딩되지 않는다는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-118">A primary advantage of the REST model is that it is based on open standards and does not bind the implementation of the model or the client applications that access it to any specific implementation.</span></span> <span data-ttu-id="152e6-119">예를 들어 Microsoft ASP.NET Web API를 사용하여 REST 웹 서비스를 구현하거나 HTTP 요청을 생성하고 HTTP 응답을 구문 분석할 수 있는 어떤 언어와 도구 집합에 의해서도 클라이언트 응용 프로그램을 개발할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-119">For example, a REST web service could be implemented by using the Microsoft ASP.NET Web API, and client applications could be developed by using any language and toolset that can generate HTTP requests and parse HTTP responses.</span></span>

> [!NOTE]
> <span data-ttu-id="152e6-120">실제로 REST는 어떤 기본 프로토콜과도 독립적이며 HTTP에 연결될 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-120">REST is actually independent of any underlying protocol and is not necessarily tied to HTTP.</span></span> <span data-ttu-id="152e6-121">그러나 REST를 기반으로 하는 시스템의 가장 일반적인 구현은 요청을 보내고 받기 위한 응용 프로그램 프로토콜로 HTTP를 이용합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-121">However, most common implementations of systems that are based on REST utilize HTTP as the application protocol for sending and receiving requests.</span></span> <span data-ttu-id="152e6-122">이 문서에서는 HTTP를 사용하여 작동하도록 디자인된 시스템에 REST 원리를 매핑하는 데 집중합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-122">This document focuses on mapping REST principles to systems designed to operate using HTTP.</span></span>
>
>

<span data-ttu-id="152e6-123">REST 모델은 탐색 체계를 사용하여 네트워크를 통해 개체 및 서비스( *리소스*라고 함)를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-123">The REST model uses a navigational scheme to represent objects and services over a network (referred to as *resources*).</span></span> <span data-ttu-id="152e6-124">일반적으로 REST를 구현하는 많은 시스템은 HTTP 프로토콜을 사용하여 이러한 리소스에 액세스하기 위한 요청을 전송합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-124">Many systems that implement REST typically use the HTTP protocol to transmit requests to access these resources.</span></span> <span data-ttu-id="152e6-125">이러한 시스템에서 클라이언트 응용 프로그램은 리소스를 식별하는 URI 및 해당 리소스에 대해 수행할 작업을 나타내는 HTTP 메서드(GET, POST, PUT 또는 DELETE가 가장 일반적임) 형식으로 요청을 제출합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-125">In these systems, a client application submits a request in the form of a URI that identifies a resource, and an HTTP method (the most common being GET, POST, PUT, or DELETE) that indicates the operation to be performed on that resource.</span></span>  <span data-ttu-id="152e6-126">HTTP 요청의 본문은 작업을 수행하는 데 필요한 데이터를 포함하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-126">The body of the HTTP request contains the data required to perform the operation.</span></span> <span data-ttu-id="152e6-127">특히 REST는 상태 비저장 요청 모델을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-127">The important point to understand is that REST defines a stateless request model.</span></span> <span data-ttu-id="152e6-128">HTTP 요청은 독립적이어야 하고 임의 순서로 발생할 수 있으므로, 요청 사이의 일시적인 상태 정보를 유지하려고 시도하는 것은 타당하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-128">HTTP requests should be independent and may occur in any order, so attempting to retain transient state information between requests is not feasible.</span></span>  <span data-ttu-id="152e6-129">정보는 리소스 자체에만 저장되며 각 요청은 자동 작업이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-129">The only place where information is stored is in the resources themselves, and each request should be an atomic operation.</span></span> <span data-ttu-id="152e6-130">결국 REST 모델은 요청이 리소스를 잘 정의되고 일시적이지 않은 상태를 다른 상태로 전환하는 유한 상태 시스템을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-130">Effectively, a REST model implements a finite state machine where a request transitions a resource from one well-defined non-transient state to another.</span></span>

> [!NOTE]
> <span data-ttu-id="152e6-131">REST 모델의 개별 요청은 상태를 저장하지 않는 특성이 있으므로 이 원리에 따라 생성된 시스템은 확장성이 매우 높습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-131">The stateless nature of individual requests in the REST model enables a system constructed by following these principles to be highly scalable.</span></span> <span data-ttu-id="152e6-132">따라서 일련의 요청을 하는 클라이언트 응용 프로그램과 해당 요청을 처리하는 특정 웹 서버 사이에 선호도를 유지할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-132">There is no need to retain any affinity between a client application making a series of requests and the specific web servers handling those requests.</span></span>
>
>

<span data-ttu-id="152e6-133">또한 효과적인 REST 모델을 구현하려면 모델이 액세스를 제공하는 여러 리소스 사이의 관계를 잘 알아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-133">Another crucial point in implementing an effective REST model is to understand the relationships between the various resources to which the model provides access.</span></span> <span data-ttu-id="152e6-134">이러한 리소스는 일반적으로 컬렉션과 관계로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-134">These resources are typically organized as collections and relationships.</span></span> <span data-ttu-id="152e6-135">예를 들어 전자 상거래 시스템을 빨리 분석해 보니 클라이언트 응용 프로그램에서 관심을 가질 수 있는 주문(orders)과 고객(customers)이라는 두 컬렉션이 있다고 가정하겠습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-135">For example, suppose that a quick analysis of an ecommerce system shows that there are two collections in which client applications are likely to be interested: orders and customers.</span></span> <span data-ttu-id="152e6-136">식별을 위해 각 주문과 고객에 자체의 고유한 키를 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-136">Each order and customer should have its own unique key for identification purposes.</span></span> <span data-ttu-id="152e6-137">주문 컬렉션에 액세스하기 위한 URI는 단순히 */orders*이며, 마찬가지로 모든 고객을 검색하기 위한 URI는 */customers*일 것입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-137">The URI to access the collection of orders could be something as simple as */orders*, and similarly the URI for retrieving all customers could be */customers*.</span></span> <span data-ttu-id="152e6-138">*/orders* URI에 대해 HTTP GET 요청을 실행하면 HTTP 응답으로 인코딩된 컬렉션의 모든 주문을 나타내는 목록이 반환될 것입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-138">Issuing an HTTP GET request to the */orders* URI should return a list representing all orders in the collection encoded as an HTTP response:</span></span>

```HTTP
GET http://adventure-works.com/orders HTTP/1.1
...
```

<span data-ttu-id="152e6-139">아래와 같은 응답은 주문을 JSON 목록 구조로 인코딩합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-139">The response shown below encodes the orders as a JSON list structure:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
[{"orderId":1,"orderValue":99.90,"productId":1,"quantity":1},{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2},{"orderId":3,"orderValue":16.60,"productId":2,"quantity":4},{"orderId":4,"orderValue":25.90,"productId":3,"quantity":1},{"orderId":5,"orderValue":99.90,"productId":1,"quantity":1}]
```
<span data-ttu-id="152e6-140">개별 주문을 가져오려면 *orders* 리소스에서 */orders/2* 같은 순서 식별자를 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-140">To fetch an individual order requires specifying the identifier for the order from the *orders* resource, such as */orders/2*:</span></span>

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
```

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2}
```

> [!NOTE]
> <span data-ttu-id="152e6-141">간단히 하기 위해 이 예제에서는 반환되는 응답에서 정보를 JSON 텍스트 데이터로 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-141">For simplicity, these examples show the information in responses being returned as JSON text data.</span></span> <span data-ttu-id="152e6-142">그러나 리소스가 HTTP에서 지원되는 다른 데이터 형식(이진 또는 암호화된 형식 등)을 포함해도 상관없으며, HTTP 응답의 콘텐츠 형식에 해당 형식을 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-142">However, there is no reason why resources should not contain any other type of data supported by HTTP, such as binary or encrypted information; the content-type in the HTTP response should specify the type.</span></span> <span data-ttu-id="152e6-143">또한 REST 모델은 같은 데이터를 XML 또는 JSON 등의 다른 형식으로 반환할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-143">Also, a REST model may be able to return the same data in different formats, such as XML or JSON.</span></span> <span data-ttu-id="152e6-144">이 경우 웹 서비스는 요청을 하는 클라이언트와 콘텐츠 협상을 수행할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-144">In this case, the web service should be able to perform content negotiation with the client making the request.</span></span> <span data-ttu-id="152e6-145">요청은 클라이언트가 수신하고자 하는 기본 설정 형식을 지정하는 *Accept* 헤더를 포함할 수 있으며 웹 서비스는 가능하면 이 형식을 적용하려고 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-145">The request can include an *Accept* header which specifies the preferred format that the client would like to receive and the web service should attempt to honor this format if at all possible.</span></span>
>
>

<span data-ttu-id="152e6-146">참고로 REST 요청에 대한 응답은 표준 HTTP 상태 코드를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-146">Notice that the response from a REST request makes use of the standard HTTP status codes.</span></span> <span data-ttu-id="152e6-147">예를 들어 유효한 데이터를 반환하는 요청은 HTTP 응답 코드 200(정상)을 포함해야 하는 반면, 지정된 리소스를 찾거나 삭제하는 데 실패한 요청은 HTTP 상태 코드 404(찾을 수 없음)가 포함된 응답을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-147">For example, a request that returns valid data should include the HTTP response code 200 (OK), while a request that fails to find or delete a specified resource should return a response that includes the HTTP status code 404 (Not Found).</span></span>

## <a name="design-and-structure-of-a-restful-web-api"></a><span data-ttu-id="152e6-148">RESTful 웹 API의 디자인 및 구조</span><span class="sxs-lookup"><span data-stu-id="152e6-148">Design and structure of a RESTful web API</span></span>
<span data-ttu-id="152e6-149">성공적인 웹 API를 디자인하는 핵심 요소는 단순성과 일관성입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-149">The keys to designing a successful web API are simplicity and consistency.</span></span> <span data-ttu-id="152e6-150">이러한 두 요소를 보여 주는 웹 API를 이용하면 API를 사용해야 하는 클라이언트 응용 프로그램을 만들기 쉽습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-150">A Web API that exhibits these two factors makes it easier to build client applications that need to consume the API.</span></span>

<span data-ttu-id="152e6-151">RESTful 웹 API는 일련의 연결된 리소스 표시 및 응용 프로그램이 이러한 리소스를 조작하고 리소스 사이를 쉽게 탐색할 수 있는 코어 작업을 제공하는 데 집중합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-151">A RESTful web API is focused on exposing a set of connected resources, and providing the core operations that enable an application to manipulate these resources and easily navigate between them.</span></span> <span data-ttu-id="152e6-152">따라서 일반적인 RESTful 웹 API를 구성하는 URI는 표시하는 데이터를 지향해야 하며 HTTP가 이 데이터에 대해 작업하기 위해 제공하는 기능을 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-152">For this reason, the URIs that constitute a typical RESTful web API should be oriented towards the data that it exposes, and use the facilities provided by HTTP to operate on this data.</span></span> <span data-ttu-id="152e6-153">이 접근 방식을 채택하려면 개체와 클래스의 동작의 영향을 더 많이 받는 경향이 있는 개체 지향 API에 클래스 집합을 디자인할 때 일반적으로 채택되는 것과는 다른 사고방식을 가져야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-153">This approach requires a different mindset from that typically employed when designing a set of classes in an object-oriented API which tends to be more motivated by the behavior of objects and classes.</span></span> <span data-ttu-id="152e6-154">또한 RESTful 웹 API는 상태 비저장이어야 하며 특정 순서로 호출되는 작업에 종속되지 않아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-154">Additionally, a RESTful web API should be stateless and not depend on operations being invoked in a particular sequence.</span></span> <span data-ttu-id="152e6-155">다음 섹션에서는 RESTful 웹 API를 디자인할 때 고려해야 할 요소를 요약 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-155">The following sections summarize the points you should consider when designing a RESTful web API.</span></span>

### <a name="organizing-the-web-api-around-resources"></a><span data-ttu-id="152e6-156">웹 API 기반 리소스 구성</span><span class="sxs-lookup"><span data-stu-id="152e6-156">Organizing the web API around resources</span></span>
> [!TIP]
> <span data-ttu-id="152e6-157">REST 웹 서비스가 표시하는 URI는 동사(응용 프로그램이 데이터에 대해 수행할 수 있는 동작)가 아닌 명사(웹 API가 액세스를 제공하는 데이터)를 기반으로 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-157">The URIs exposed by a REST web service should be based on nouns (the data to which the web API provides access) and not verbs (what an application can do with the data).</span></span>
>
>

<span data-ttu-id="152e6-158">즉, 웹 API가 표시하는 비즈니스 엔터티에 집중해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-158">Focus on the business entities that the web API exposes.</span></span> <span data-ttu-id="152e6-159">예를 들어 앞에서 설명한 전자 상거래 시스템을 지원하도록 디자인된 웹 API의 경우, 기본 엔터티는 고객과 주문입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-159">For example, in a web API designed to support the ecommerce system described earlier, the primary entities are customers and orders.</span></span> <span data-ttu-id="152e6-160">주문을 수행하는 동작과 같은 프로세스는 주문 정보를 가져와서 고객의 주문 목록에 추가하는 HTTP POST 작업을 제공하면 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-160">Processes such as the act of placing an order can be achieved by providing an HTTP POST operation that takes the order information and adds it to the list of orders for the customer.</span></span> <span data-ttu-id="152e6-161">내부적으로 볼 때 이 POST 작업은 재고 수준 확인 및 고객에 대한 청구 등의 작업을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-161">Internally, this POST operation can perform tasks such as checking stock levels, and billing the customer.</span></span> <span data-ttu-id="152e6-162">HTTP 응답은 주문이 성공적으로 수행되었는지 여부를 나타낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-162">The HTTP response can indicate whether the order was placed successfully or not.</span></span> <span data-ttu-id="152e6-163">또한 리소스는 단일 물리적 데이터 항목을 기반으로 할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-163">Also note that a resource does not have to be based on a single physical data item.</span></span> <span data-ttu-id="152e6-164">한 예로, 관계형 데이터베이스의 여러 테이블에 분포된 많은 행에서 집계되지만 클라이언트에 대해서는 단일 엔터티로 표시되는 정보를 사용하여 내부적으로 주문 리소스를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-164">As an example, an order resource might be implemented internally by using information aggregated from many rows spread across several tables in a relational database but presented to the client as a single entity.</span></span>

> [!TIP]
> <span data-ttu-id="152e6-165">표시하는 데이터의 내부 구조를 미러링하거나 해당 구조에 따라 좌우되는 REST 인터페이스를 디자인하지 않는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-165">Avoid designing a REST interface that mirrors or depends on the internal structure of the data that it exposes.</span></span> <span data-ttu-id="152e6-166">REST는 관계형 데이터베이스에서 별도 테이블에 대해 간단한 CRUD(만들기, 검색, 업데이트, 삭제) 작업을 구현하는 것 이상의 많은 기능을 가지고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-166">REST is about more than implementing simple CRUD (Create, Retrieve, Update, Delete) operations over separate tables in a relational database.</span></span> <span data-ttu-id="152e6-167">REST의 목적은 비즈니스 엔터티와 응용 프로그램이 해당 엔터티에 대해 수행할 수 있는 작업을 매핑하는 것이지만, 이러한 물리적 세부사항을 클라이언트에 표시하지 않아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-167">The purpose of REST is to map business entities and the operations that an application can perform on these entities to the physical implementation of these entities, but a client should not be exposed to these physical details.</span></span>
>
>

<span data-ttu-id="152e6-168">개별 비즈니스 엔터티가 분리되어 존재하는 경우는 거의 없으며(일부 단일 항목 개체는 존재할 수 있지만) 컬렉션으로 함께 그룹화되는 경향이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-168">Individual business entities rarely exist in isolation (although some singleton objects may exist), but instead tend to be grouped together into collections.</span></span> <span data-ttu-id="152e6-169">REST과 관련해서는 각 엔터티와 각 컬렉션을 리소스라고 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-169">In REST terms, each entity and each collection are resources.</span></span> <span data-ttu-id="152e6-170">RESTful 웹 API에서 각 컬렉션은 웹 서비스 내에서 자체의 URI를 가지고 있으며 컬렉션의 URI에 대해 HTTP GET 요청을 수행하면 해당 컬렉션에서 항목 목록이 검색됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-170">In a RESTful web API, each collection has its own URI within the web service, and performing an HTTP GET request over a URI for a collection retrieves a list of items in that collection.</span></span> <span data-ttu-id="152e6-171">또한 각 개별 항목도 자체 URI를 가지고 있으며, 응용 프로그램은 이 URI를 사용하여 HTTP GET 요청을 제출하여 해당 항목의 세부 정보를 검색할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-171">Each individual item also has its own URI, and an application can submit another HTTP GET request using that URI to retrieve the details of that item.</span></span> <span data-ttu-id="152e6-172">컬렉션과 항목에 대해 계층적인 방법으로 URI를 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-172">You should organize the URIs for collections and items in a hierarchical manner.</span></span> <span data-ttu-id="152e6-173">전자 상거래 시스템에서 URI */customers*는 고객의 컬렉션을 나타내며, */customers/5*는 이 컬렉션에서 ID가 5인 단일 고객에 대한 세부 정보를 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-173">In the ecommerce system, the URI */customers* denotes the customer’s collection, and */customers/5* retrieves the details for the single customer with the ID 5 from this collection.</span></span> <span data-ttu-id="152e6-174">이 접근 방식을 사용하면 웹 API를 직관적으로 유지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-174">This approach helps to keep the web API intuitive.</span></span>

> [!TIP]
> <span data-ttu-id="152e6-175">URI에 일관성 있는 명명 규칙을 적용해야 하며, 일반적으로 이렇게 하면 컬렉션을 참조하는 URI에 대해 복수 명사를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-175">Adopt a consistent naming convention in URIs; in general it helps to use plural nouns for URIs that reference collections.</span></span>
>
>

<span data-ttu-id="152e6-176">또한 서로 다른 리소스 형식과 이러한 연결을 표시하는 방법 사이의 관계도 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-176">You also need to consider the relationships between different types of resources and how you might expose these associations.</span></span> <span data-ttu-id="152e6-177">예를 들어 고객은 0개 이상의 주문을 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-177">For example, customers may place zero or more orders.</span></span> <span data-ttu-id="152e6-178">*/customers/5/orders* 같은 URI를 통해 고객 5에 대한 모든 주문을 찾는 것은 이 관계를 나타내는 자연스러운 방법이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-178">A natural way to represent this relationship would be through a URI such as */customers/5/orders* to find all the orders for customer 5.</span></span> <span data-ttu-id="152e6-179">또한 */orders/99/customer* 같은 URI를 통해 주문에서 특정 고객에게 다시 돌아가는 연결을 나타냄으로써 주문 99에 대한 고객을 찾을 수도 있지만, 이 모델을 너무 크게 확장하면 구현하기가 번거로워질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-179">You might also consider representing the association from an order back to a specific customer through a URI such as */orders/99/customer* to find the customer for order 99, but extending this model too far can become cumbersome to implement.</span></span> <span data-ttu-id="152e6-180">더 나은 방법은 주문을 쿼리할 때 반환되는 HTTP 응답 메시지의 본문에 고객 같은 연결된 리소스에 대한 탐색 가능한 링크를 제공하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-180">A better solution is to provide navigable links to associated resources, such as the customer, in the body of the HTTP response message returned when the order is queried.</span></span> <span data-ttu-id="152e6-181">이 메커니즘은 이 지침의 후반부에 나오는 HATEOAS 접근 방식을 사용하여 관련 리소스 탐색 사용에 더 자세히 설명되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-181">This mechanism is described in more detail in the section Using the HATEOAS Approach to Enable Navigation To Related Resources later in this guidance.</span></span>

<span data-ttu-id="152e6-182">더 복잡한 시스템에는 더 많은 엔터티 형식이 있을 수 있으며, 클라이언트 응용 프로그램이 */customers/1/orders/99/products*와 같이 여러 수준의 관계를 통해 탐색하여 고객 1이 실행한 주문 99의 제품 목록을 가져올 수 있는 URI를 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-182">In more complex systems there may be many more types of entity, and it can be tempting to provide URIs that enable a client application to navigate through several levels of relationships, such as */customers/1/orders/99/products* to obtain the list of products in order 99 placed by customer 1.</span></span> <span data-ttu-id="152e6-183">그러나 이 수준의 복잡성은 유지하기 어려울 수 있으며 나중에 리소스 사이의 관계가 변하면 유연성이 떨어집니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-183">However, this level of complexity can be difficult to maintain and is inflexible if the relationships between resources change in the future.</span></span> <span data-ttu-id="152e6-184">가능하면 URI를 비교적 단순하게 유지하도록 노력해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-184">Rather, you should seek to keep URIs relatively simple.</span></span> <span data-ttu-id="152e6-185">응용 프로그램이 리소스 참조를 지정한 후에는 이 참조를 사용하여 해당 리소스와 관련된 항목을 찾을 수 있어야 한다는 점에 주의해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-185">Bear in mind that once an application has a reference to a resource, it should be possible to use this reference to find items related to that resource.</span></span> <span data-ttu-id="152e6-186">앞의 쿼리를 */customers/1/orders*로 바꾸어 고객 1에 대한 모든 주문을 찾은 다음 URI */orders/99/products*를 쿼리하여 이 주문의 제품을 찾을 수 있습니다(주문 99를 고객 1이 했다고 가정).</span><span class="sxs-lookup"><span data-stu-id="152e6-186">The preceding query can be replaced with the URI */customers/1/orders* to find all the orders for customer 1, and then query the URI */orders/99/products* to find the products in this order (assuming order 99 was placed by customer 1).</span></span>

> [!TIP]
> <span data-ttu-id="152e6-187">리소스 URI를 *컬렉션/항목/컬렉션*보다 더 복잡하게 요구하지 않는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-187">Avoid requiring resource URIs more complex than *collection/item/collection*.</span></span>
>
>

<span data-ttu-id="152e6-188">또한 모든 웹 요청은 웹 서버에 부담을 주며 요청 수가 많을수록 이 부담이 더 커진다는 점도 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-188">Another point to consider is that all web requests impose a load on the web server, and the greater the number of requests the bigger the load.</span></span> <span data-ttu-id="152e6-189">다수의 작은 리소스를 표시하는 "번잡한" 웹 API를 피하도록 리소스를 정의하기 위해 노력해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-189">You should attempt to define your resources to avoid “chatty” web APIs that expose a large number of small resources.</span></span> <span data-ttu-id="152e6-190">그러한 API를 사용하려면 클라이언트 응용 프로그램이 요구하는 모든 데이터를 찾기 위해 복수의 요청을 제출해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-190">Such an API may require a client application to submit multiple requests to find all the data that it requires.</span></span> <span data-ttu-id="152e6-191">데이터를 비정규화하고 관련 정보를 단일 요청에 의해 검색할 수 있는 더 큰 리소스로 함께 결합하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-191">It may be beneficial to denormalize data and combine related information together into bigger resources that can be retrieved by issuing a single request.</span></span> <span data-ttu-id="152e6-192">단, 이 접근 방식과 클라이언트가 자주 요구하지 않는 데이터를 가져오는 오버헤드의 균형을 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-192">However, you need to balance this approach against the overhead of fetching data that might not be frequently required by the client.</span></span> <span data-ttu-id="152e6-193">큰 개체를 검색하면 요청의 대기 시간이 증가하고 추가 데이터를 자주 사용하지 않는 경우 작은 이익을 얻기 위해 추가 대역폭 비용을 초래할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-193">Retrieving large objects can increase the latency of a request and incur additional bandwidth costs for little advantage if the additional data is not often used.</span></span>

<span data-ttu-id="152e6-194">웹 API와 기본 데이터 원본의 구조, 형식 또는 위치 사이에 종속성이 발생하지 않도록 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-194">Avoid introducing dependencies between the web API to the structure, type, or location of the underlying data sources.</span></span> <span data-ttu-id="152e6-195">예를 들어 데이터가 관계형 데이터베이스에 있는 경우, 웹 API는 각 테이블을 리소스 컬렉션으로 표시할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-195">For example, if your data is located in a relational database, the web API does not need to expose each table as a collection of resources.</span></span> <span data-ttu-id="152e6-196">웹 API를 데이터베이스의 추상화하고 생각하고 필요한 경우 데이터베이스와 웹 API 사이에 매핑 계층을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-196">Think of the web API as an abstraction of the database, and if necessary introduce a mapping layer between the database and the web API.</span></span> <span data-ttu-id="152e6-197">이러한 방식으로 데이터베이스의 디자인 또는 구현이 변경된 경우(예를 들어 정규화된 테이블의 컬렉션이 포함된 관계형 데이터베이스에서 문서 데이터베이스 같은 비정규화 NoSQL 저장소 시스템으로 이동한 경우) 클라이언트 응용 프로그램은 이러한 변경 내용을 인식하지 못하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-197">In this way, if the design or implementation of the database changes (for example, you move from a relational database containing a collection of normalized tables to a denormalized NoSQL storage system such as a document database) client applications are insulated from these changes.</span></span>

> [!TIP]
> <span data-ttu-id="152e6-198">웹 API를 바탕으로 하는 데이터의 원본은 데이터 저장소일 필요가 없으며, 다른 서비스 또는 기간 업무 응용 프로그램 또는 조직 내에서 온-프레미스로 실행되는 레거시 응용 프로그램일 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-198">The source of the data that underpins a web API does not have to be a data store; it could be another service or line-of-business application or even a legacy application running on-premises within an organization.</span></span>
>
>

<span data-ttu-id="152e6-199">마지막으로, 웹 API에 의해 구현된 일부 작업을 특정 리소스에 매핑하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-199">Finally, it might not be possible to map every operation implemented by a web API to a specific resource.</span></span> <span data-ttu-id="152e6-200">HTTP GET 요청을 통해 기능의 일부를 호출하고 결과를 HTTP 응답 메시지로 반환하는 *리소스가 아닌* 시나리오를 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-200">You can handle such *non-resource* scenarios through HTTP GET requests that invoke a piece of functionality and return the results as an HTTP response message.</span></span> <span data-ttu-id="152e6-201">더하기 및 빼기 같은 단순한 계산기 스타일의 작업을 구현하는 웹 API는 이러한 작업을 의사 리소스로 표시하고 쿼리 문자열을 사용하여 필요한 매개 변수를 지정하는 URI를 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-201">A web API that implements simple calculator-style operations such as add and subtract could provide URIs that expose these operations as pseudo resources and utilize the query string to specify the parameters required.</span></span> <span data-ttu-id="152e6-202">예를 들어 URI */add?operand1=99&operand2=1*에 대한 GET 요청은 본문에 값 100이 포함된 응답 메시지를 반환할 수 있으며, URI */subtract?operand1=50&operand2=20*에 대한 GET 요청은 본문에 값 30이 포함된 응답 메시지를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-202">For example a GET request to the URI */add?operand1=99&operand2=1* could return a response message with the body containing the value 100, and GET request to the URI */subtract?operand1=50&operand2=20* could return a response message with the body containing the value 30.</span></span> <span data-ttu-id="152e6-203">그러나 이러한 형식의 URI는 제한적으로만 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-203">However, only use these forms of URIs sparingly.</span></span>

### <a name="defining-operations-in-terms-of-http-methods"></a><span data-ttu-id="152e6-204">HTTP 메서드를 기준으로 작업 정의</span><span class="sxs-lookup"><span data-stu-id="152e6-204">Defining operations in terms of HTTP methods</span></span>
<span data-ttu-id="152e6-205">HTTP 프로토콜은 요청에 의미 체계의미를 할당하는 다양한 메서드를 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-205">The HTTP protocol defines a number of methods that assign semantic meaning to a request.</span></span> <span data-ttu-id="152e6-206">대부분의 RESTful 웹 API에서 사용하는 일반적인 HTTP 메서드는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-206">The common HTTP methods used by most RESTful web APIs are:</span></span>

* <span data-ttu-id="152e6-207">**GET**, 지정된 URI에서 리소스의 복사본을 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-207">**GET**, to retrieve a copy of the resource at the specified URI.</span></span> <span data-ttu-id="152e6-208">응답 메시지의 본문은 요청된 리소스의 세부 정보를 포함하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-208">The body of the response message contains the details of the requested resource.</span></span>
* <span data-ttu-id="152e6-209">**POST**, 지정된 URI에 새 리소스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-209">**POST**, to create a new resource at the specified URI.</span></span> <span data-ttu-id="152e6-210">요청 메시지의 본문은 새 리소스의 세부 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-210">The body of the request message provides the details of the new resource.</span></span> <span data-ttu-id="152e6-211">참고로 POST를 사용하여 실제로 리소스를 만들지 않는 작업을 트리거할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-211">Note that POST can also be used to trigger operations that don't actually create resources.</span></span>
* <span data-ttu-id="152e6-212">**PUT**, 지정된 URI의 리소스를 바꾸거나 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-212">**PUT**, to replace or update the resource at the specified URI.</span></span> <span data-ttu-id="152e6-213">요청 메시지의 본문은 수정할 리소스 및 적용할 값을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-213">The body of the request message specifies the resource to be modified and the values to be applied.</span></span>
* <span data-ttu-id="152e6-214">**DELETE**, 지정된 URI의 리소스를 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-214">**DELETE**, to remove the resource at the specified URI.</span></span>

> [!NOTE]
> <span data-ttu-id="152e6-215">또한 HTTP 프로토콜은 리소스에 대한 선택적 업데이트를 요청하는 데 사용하는 PATCH, 리소스 설명을 요청하는 데 사용하는 HEAD, 클라이언트가 서버에서 지원되는 통신 옵션에 관한 정보를 가져올 수 있는 OPTIONS 및 클라이언트가 테스트와 진단 목적으로 사용할 수 있는 정보를 가져올 수 있는 TRACE 등 일반적으로 사용되지 않는 다른 메서드도 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-215">The HTTP protocol also defines other less commonly-used methods, such as PATCH which is used to request selective updates to a resource, HEAD which is used to request a description of a resource, OPTIONS which enables a client information to obtain information about the communication options supported by the server, and TRACE which allows a client to request information that it can use for testing and diagnostics purposes.</span></span>
>
>

<span data-ttu-id="152e6-216">특정 요청의 효과는 적용되는 리소스가 컬렉션인지 아니면 개별 항목인지에 따라 달라져야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-216">The effect of a specific request should depend on whether the resource to which it is applied is a collection or an individual item.</span></span> <span data-ttu-id="152e6-217">다음 표는 전자 상거래 예제를 사용 하여 대부분의 RESTful 구현에서 적용하는 일반적인 규칙을 요약합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-217">The following table summarizes the common conventions adopted by most RESTful implementations using the ecommerce example.</span></span> <span data-ttu-id="152e6-218">참고로 이 요청 중 일부는 구현되지 않을 수 있으며, 구현 여부는 특정 시나리오에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-218">Note that not all of these requests might be implemented; it depends on the specific scenario.</span></span>

| <span data-ttu-id="152e6-219">**리소스**</span><span class="sxs-lookup"><span data-stu-id="152e6-219">**Resource**</span></span> | <span data-ttu-id="152e6-220">**POST**</span><span class="sxs-lookup"><span data-stu-id="152e6-220">**POST**</span></span> | <span data-ttu-id="152e6-221">**GET**</span><span class="sxs-lookup"><span data-stu-id="152e6-221">**GET**</span></span> | <span data-ttu-id="152e6-222">**PUT**</span><span class="sxs-lookup"><span data-stu-id="152e6-222">**PUT**</span></span> | <span data-ttu-id="152e6-223">**DELETE**</span><span class="sxs-lookup"><span data-stu-id="152e6-223">**DELETE**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="152e6-224">/customers</span><span class="sxs-lookup"><span data-stu-id="152e6-224">/customers</span></span> |<span data-ttu-id="152e6-225">새 고객 만들기</span><span class="sxs-lookup"><span data-stu-id="152e6-225">Create a new customer</span></span> |<span data-ttu-id="152e6-226">모든 고객 검색</span><span class="sxs-lookup"><span data-stu-id="152e6-226">Retrieve all customers</span></span> |<span data-ttu-id="152e6-227">고객에 대한 일괄 업데이트(*구현된 경우*)</span><span class="sxs-lookup"><span data-stu-id="152e6-227">Bulk update of customers (*if implemented*)</span></span> |<span data-ttu-id="152e6-228">모든 고객 제거</span><span class="sxs-lookup"><span data-stu-id="152e6-228">Remove all customers</span></span> |
| <span data-ttu-id="152e6-229">/customers/1</span><span class="sxs-lookup"><span data-stu-id="152e6-229">/customers/1</span></span> |<span data-ttu-id="152e6-230">오류</span><span class="sxs-lookup"><span data-stu-id="152e6-230">Error</span></span> |<span data-ttu-id="152e6-231">고객 1에 대한 세부 정보 검색</span><span class="sxs-lookup"><span data-stu-id="152e6-231">Retrieve the details for customer 1</span></span> |<span data-ttu-id="152e6-232">존재하는 경우 고객 1의 세부 정보 업데이트, 그렇지 않으면 오류 반환</span><span class="sxs-lookup"><span data-stu-id="152e6-232">Update the details of customer 1 if it exists, otherwise return an error</span></span> |<span data-ttu-id="152e6-233">고객 1 제거</span><span class="sxs-lookup"><span data-stu-id="152e6-233">Remove customer 1</span></span> |
| <span data-ttu-id="152e6-234">/customers/1/orders</span><span class="sxs-lookup"><span data-stu-id="152e6-234">/customers/1/orders</span></span> |<span data-ttu-id="152e6-235">고객 1에 대한 새 주문 만들기</span><span class="sxs-lookup"><span data-stu-id="152e6-235">Create a new order for customer 1</span></span> |<span data-ttu-id="152e6-236">고객 1에 대한 모든 주문 검색</span><span class="sxs-lookup"><span data-stu-id="152e6-236">Retrieve all orders for customer 1</span></span> |<span data-ttu-id="152e6-237">고객 1에 대한 주문 일괄 업데이트(*구현된 경우*)</span><span class="sxs-lookup"><span data-stu-id="152e6-237">Bulk update of orders for customer 1 (*if implemented*)</span></span> |<span data-ttu-id="152e6-238">고객 1에 대한 주문 제거(*구현된 경우*)</span><span class="sxs-lookup"><span data-stu-id="152e6-238">Remove all orders for customer 1(*if implemented*)</span></span> |

<span data-ttu-id="152e6-239">GET 및 DELETE 요청의 목적은 비교적 간단하지만, POST 및 PUT 요청의 목적과 효과에 대해서는 혼동을 일으키는 범위가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-239">The purpose of GET and DELETE requests are relatively straightforward, but there is scope for confusion concerning the purpose and effects of POST and PUT requests.</span></span>

<span data-ttu-id="152e6-240">POST 요청은 요청 본문에 제공된 데이터를 사용하여 새 리소스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-240">A POST request should create a new resource with data provided in the body of the request.</span></span> <span data-ttu-id="152e6-241">REST 모델에서는 흔히 POST 요청을 컬렉션인 리소스에 적용하며, 새 리소스는 컬렉션에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-241">In the REST model, you frequently apply POST requests to resources that are collections; the new resource is added to the collection.</span></span>

> [!NOTE]
> <span data-ttu-id="152e6-242">일부 기능을 트리거하는(그리고 데이터를 반환하지 않을 수도 있는) POST 요청을 정의할 수도 있으며, 이러한 요청 형식을 컬렉션에 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-242">You can also define POST requests that trigger some functionality (and that don't necessarily return data), and these types of request can be applied to collections.</span></span> <span data-ttu-id="152e6-243">예를 들어 POST 요청을 사용하여 급여 처리 서비스에 작업표를 전달하고 계산된 세금을 다시 응답으로 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-243">For example you could use a POST request to pass a timesheet to a payroll processing service and get the calculated taxes back as a response.</span></span>
>
>

<span data-ttu-id="152e6-244">PUT 요청은 기존 리소스를 수정하기 위한 것입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-244">A PUT request is intended to modify an existing resource.</span></span> <span data-ttu-id="152e6-245">지정된 리소스가 존재하지 않는 경우, PUT 요청은 오류를 반환할 수 있습니다(경우에 따라 실제로 리소스를 만들 수 있음).</span><span class="sxs-lookup"><span data-stu-id="152e6-245">If the specified resource does not exist, the PUT request could return an error (in some cases, it might actually create the resource).</span></span> <span data-ttu-id="152e6-246">PUT 요청은 컬렉션에 적용될 수 있지만(일반적으로 이렇게 구현하지 않지만) 개별 항목인 리소스(특정 고객 또는 주문)에 적용되는 경우가 가장 많습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-246">PUT requests are most frequently applied to resources that are individual items (such as a specific customer or order), although they can be applied to collections, although this is less-commonly implemented.</span></span> <span data-ttu-id="152e6-247">참고로 PUT 요청은 멱등원이지만 POST 요청은 그렇지 않습니다. 즉, 응용 프로그램이 같은 PUT 요청을 여러 번 제출하더라도 결과는 항상 같지만(같은 리소스가 같은 값으로 수정됨), 응용 프로그램이 같은 POST 요청을 반복하면 결과는 복수의 리소스가 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-247">Note that PUT requests are idempotent whereas POST requests are not; if an application submits the same PUT request multiple times the results should always be the same (the same resource will be modified with the same values), but if an application repeats the same POST request the result will be the creation of multiple resources.</span></span>

> [!NOTE]
> <span data-ttu-id="152e6-248">엄격히 말해서, HTTP PUT 요청은 기존 리소스를 요청 본문에 지정된 리소스로 바꿉니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-248">Strictly speaking, an HTTP PUT request replaces an existing resource with the resource specified in the body of the request.</span></span> <span data-ttu-id="152e6-249">리소스의 속성 선택을 수정하지만 다른 속성을 변경하지 않으려는 경우, HTTP PATCH 요청을 사용하여 이 기능을 구현해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-249">If the intention is to modify a selection of properties in a resource but leave other properties unchanged, then this should be implemented by using an HTTP PATCH request.</span></span> <span data-ttu-id="152e6-250">그러나 많은 RESTful 구현은 이 규칙을 완화하며 두 상황에 모두 PUT을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-250">However, many RESTful implementations relax this rule and use PUT for both situations.</span></span>
>
>

### <a name="processing-http-requests"></a><span data-ttu-id="152e6-251">HTTP 요청 처리</span><span class="sxs-lookup"><span data-stu-id="152e6-251">Processing HTTP requests</span></span>
<span data-ttu-id="152e6-252">많은 HTTP 요청에서 클라이언트 응용 프로그램이 포함시킨 데이터 및 웹 서버에서 오는 해당 응답 메시지를 다양한 형식(또는 미디어 형식)으로 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-252">The data included by a client application in many HTTP requests, and the corresponding response messages from the web server, could be presented in a variety of formats (or media types).</span></span> <span data-ttu-id="152e6-253">예를 들어, 고객 또는 주문에 대한 세부 정보를 지정하는 데이터를 XML, JSON 또는 다른 인코딩 및 압축된 형식으로 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-253">For example, the data that specifies the details for a customer or order could be provided as XML, JSON, or some other encoded and compressed format.</span></span> <span data-ttu-id="152e6-254">RESTful 웹 API는 요청을 제출하는 클라이언트 응용 프로그램에서 요청한 대로 서로 다양한 미디어 형식을 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-254">A RESTful web API should support different media types as requested by the client application that submits a request.</span></span>

<span data-ttu-id="152e6-255">클라이언트 응용 프로그램은 메시지 본문에 데이터를 반환하는 요청을 보낼 때 처리할 수 있는 미디어 형식을 요청의 Accept 헤더에 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-255">When a client application sends a request that returns data in the body of a message, it can specify the media types it can handle in the Accept header of the request.</span></span> <span data-ttu-id="152e6-256">다음 코드에서는 주문 2에 대한 세부 정보를 검색하고 반환할 결과를 JSON으로 요청하는 HTTP GET을 보여 줍니다. 클라이언트는 반환되는 데이터의 형식을 확인하기 위해 여전히 응답의 데이터의 미디어 형식을 검사해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-256">The following code illustrates an HTTP GET request that retrieves the details of order 2 and requests the result to be returned as JSON (the client should still examine the media type of the data in the response to verify the format of the data returned):</span></span>

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
Accept: application/json
...
```

<span data-ttu-id="152e6-257">웹 서버는 이 미디어 형식을 지원하는 경우 메시지 본문의 데이터 형식을 지정하는 Content-Type 헤더를 포함하고 있는 응답으로 회신할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-257">If the web server supports this media type, it can reply with a response that includes Content-Type header that specifies the format of the data in the body of the message:</span></span>

> [!NOTE]
> <span data-ttu-id="152e6-258">상호 운용성을 최대로 높이기 위해, Accept 및 Content-Type 헤더에 참조되는 미디어 형식은 일부 사용자 지정 미디어 형식이 아닌 MIME 형식으로 인식됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-258">For maximum interoperability, the media types referenced in the Accept and Content-Type headers should be recognized MIME types rather than some custom media type.</span></span>
>
>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

<span data-ttu-id="152e6-259">요청된 미디어 형식을 지원하지 않는 경우, 웹 서버는 데이터를 다른 형식으로 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-259">If the web server does not support the requested media type, it can send the data in a different format.</span></span> <span data-ttu-id="152e6-260">모든 경우 Content-Type 헤더에 미디어 형식을 지정해야 합니다(예: *application/json*).</span><span class="sxs-lookup"><span data-stu-id="152e6-260">IN all cases it must specify the media type (such as *application/json*) in the Content-Type header.</span></span> <span data-ttu-id="152e6-261">응답 메시지를 분석하고 메시지 본문의 결과를 적절히 해석하는 것은 클라이언트 응용 프로그램에서 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-261">It is the responsibility of the client application to parse the response message and interpret the results in the message body appropriately.</span></span>

<span data-ttu-id="152e6-262">참고로 이 예에서 웹 서버는 성공적으로 요청 데이터를 검색하며 응답 헤더에 상태 코드 200을 전달하여 성공을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-262">Note that in this example, the web server successfully retrieves the requested data and indicates success by passing back a status code of 200 in the response header.</span></span> <span data-ttu-id="152e6-263">일치하는 데이터가 없으면 상태 코드 404(찾을 수 없음)을 대신 반환해야 하며 응답 메시지의 본문에 추가 정보가 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-263">If no matching data is found, it should instead return a status code of 404 (not found) and the body of the response message can contain additional information.</span></span> <span data-ttu-id="152e6-264">이 정보의 형식은 다음 예제와 같이Content-type 헤더에 의해 지정됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-264">The format of this information is specified by the Content-Type header, as shown in the following example:</span></span>

```HTTP
GET http://adventure-works.com/orders/222 HTTP/1.1
...
Accept: application/json
...
```

<span data-ttu-id="152e6-265">주문 222는 존재하지 않으므로 응답 메시지는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-265">Order 222 does not exist, so the response message looks like this:</span></span>

```HTTP
HTTP/1.1 404 Not Found
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"message":"No such order"}
```

<span data-ttu-id="152e6-266">응용 프로그램은 리소스를 업데이트하는 HTTP PUT 요청을 보낼 때 리소스의 URI를 지정하고 수정할 데이터를 요청 메시지의 본문에 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-266">When an application sends an HTTP PUT request to update a resource, it specifies the URI of the resource and provides the data to be modified in the body of the request message.</span></span> <span data-ttu-id="152e6-267">또한 Content-Type 헤더를 사용하여 이 데이터의 형식을 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-267">It should also specify the format of this data by using the Content-Type header.</span></span> <span data-ttu-id="152e6-268">텍스트 기반 정보에 사용되는 일반적인 형식은 *application/x-www-form-urlencoded*이며, 이 형식은 & 문자로 구분되는 이름/값 쌍의 집합으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-268">A common format used for text-based information is *application/x-www-form-urlencoded*, which comprises a set of name/value pairs separated by the & character.</span></span> <span data-ttu-id="152e6-269">다음 예제에서는 주문 1의 정보를 수정하는 HTTP PUT 요청을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-269">The next example shows an HTTP PUT request that modifies the information in order 1:</span></span>

```HTTP
PUT http://adventure-works.com/orders/1 HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
ProductID=3&Quantity=5&OrderValue=250
```

<span data-ttu-id="152e6-270">수정에 성공할 경우, 프로세스가 성공적으로 처리되었지만 응답 본문에 추가 정보가 포함되지 않았음을 나타내는 HTTP 204 상태 코드로 응답하는 것이 최적입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-270">If the modification is successful, it should ideally respond with an HTTP 204 status code, indicating that the process has been successfully handled, but that the response body contains no further information.</span></span> <span data-ttu-id="152e6-271">응답의 Location 헤더에는 새로 업데이트된 리소스의 URI가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-271">The Location header in the response contains the URI of the newly updated resource:</span></span>

```HTTP
HTTP/1.1 204 No Content
...
Location: http://adventure-works.com/orders/1
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

> [!TIP]
> <span data-ttu-id="152e6-272">HTTP PUT 요청 메시지의 데이터가 날짜 및 시간 정보를 포함한 경우, 사용자의 웹 서비스는 ISO 8601 표준을 따르는 형식의 날짜 및 시간을 수락하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-272">If the data in an HTTP PUT request message includes date and time information, make sure that your web service accepts dates and times formatted following the ISO 8601 standard.</span></span>
>
>

<span data-ttu-id="152e6-273">업데이트할 리소스가 없는 경우, 웹 서버는 앞에서 설명한 대로 찾을 수 없음 응답으로 응답할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-273">If the resource to be updated does not exist, the web server can respond with a Not Found response as described earlier.</span></span> <span data-ttu-id="152e6-274">또는 서버가 개체 자체를 실제로 만드는 경우, 상태 코드 HTTP 200(정상) 또는 HTTP 201(생성됨)를 반환하며 응답 본문은 새 리소스에 대한 데이터를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-274">Alternatively, if the server actually creates the object itself it could return the status codes HTTP 200 (OK) or HTTP 201 (Created) and the response body could contain the data for the new resource.</span></span> <span data-ttu-id="152e6-275">요청의 Content-Type 헤더가 웹 서버에서 처리할 수 없는 데이터 형식을 지정한 경우, HTTP 상태 코드 415(지원되지 않는 미디어 형식)으로 응답해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-275">If the Content-Type header of the request specifies a data format that the web server cannot handle, it should respond with HTTP status code 415 (Unsupported Media Type).</span></span>

> [!TIP]
> <span data-ttu-id="152e6-276">컬렉션의 복수 리소스에 대한 업데이트를 일괄 처리할 수 있는 일괄 HTTP PUT 작업의 구현을 생각해 보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-276">Consider implementing bulk HTTP PUT operations that can batch updates to multiple resources in a collection.</span></span> <span data-ttu-id="152e6-277">PUT 요청은 컬렉션의 URI를 지정해야 하며, 요청 본문에 수정할 리소스의 세부 정보를 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-277">The PUT request should specify the URI of the collection, and the request body should specify the details of the resources to be modified.</span></span> <span data-ttu-id="152e6-278">이 접근 방식은 데이터 전송량을 줄이고 성능을 향상시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-278">This approach can help to reduce chattiness and improve performance.</span></span>
>
>

<span data-ttu-id="152e6-279">새 리소스를 만드는 HTTP POST 요청의 형식은 PUT 요청과 유사하며, 메시지 본문에 추가할 새 리소스의 세부 정보를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-279">The format of an HTTP POST requests that create new resources are similar to those of PUT requests; the message body contains the details of the new resource to be added.</span></span> <span data-ttu-id="152e6-280">그러나 URI는 일반적으로 리소스를 추가해야 할 컬렉션을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-280">However, the URI typically specifies the collection to which the resource should be added.</span></span> <span data-ttu-id="152e6-281">다음 예제에서는 새 주문을 만들어 주문 컬렉션에 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-281">The following example creates a new order and adds it to the orders collection:</span></span>

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
productID=5&quantity=15&orderValue=400
```

<span data-ttu-id="152e6-282">요청에 성공하면 HTTP 상태 코드 201(생성됨)이 포함된 메시지 코드로 응답해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-282">If the request is successful, the web server should respond with a message code with HTTP status code 201 (Created).</span></span> <span data-ttu-id="152e6-283">Location 헤더는 새로 만든 리소스의 URI를 포함해야 하며, 응답의 본문에 새 리소스의 복사본이 있어야 합니다. Content-type 헤더에는 이 데이터의 형식을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-283">The Location header should contain the URI of the newly created resource, and the body of the response should contain a copy of the new resource; the Content-Type header specifies the format of this data:</span></span>

```HTTP
HTTP/1.1 201 Created
...
Content-Type: application/json; charset=utf-8
Location: http://adventure-works.com/orders/99
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":99,"productID":5,"quantity":15,"orderValue":400}
```

> [!TIP]
> <span data-ttu-id="152e6-284">PUT 또는 POST 요청에 의해 제공되는 데이터가 유효하지 않은 경우, 웹 서버는 HTTP 상태 코드 400(잘못된 요청)이 포함된 메시지로 응답해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-284">If the data provided by a PUT or POST request is invalid, the web server should respond with a message with HTTP status code 400 (Bad Request).</span></span> <span data-ttu-id="152e6-285">이 메시지의 본문은 요청 및 예상되는 형식과 함께 문제에 대한 추가 정보를 포함하거나 더 자세한 정보를 제공하는 URL에 대한 링크를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-285">The body of this message can contain additional information about the problem with the request and the formats expected, or it can contain a link to a URL that provides more details.</span></span>
>
>

<span data-ttu-id="152e6-286">리소스를 제거하려면 HTTP DELETE 요청 단순히 삭제할 리소스의 URI를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-286">To remove a resource, an HTTP DELETE request simply provides the URI of the resource to be deleted.</span></span> <span data-ttu-id="152e6-287">다음 예제에서는 주문 99를 제거하려고 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-287">The following example attempts to remove order 99:</span></span>

```HTTP
DELETE http://adventure-works.com/orders/99 HTTP/1.1
...
```

<span data-ttu-id="152e6-288">삭제에 성공하면 웹 서버는 프로세스가 성공적으로 처리되었지만 응답 본문에 추가 정보가 없음을 나타내는 HTTP 상태 코드 204로 응답해야 합니다. 이는 성공한 PUT 작업에서 반환된 것과 같은 응답이지만, 리소스가 더 이상 존재하지 않으므로 Location 헤더가 없습니다. 또한 삭제가 비동기 방식으로 수행되는 경우 DELETE 요청이 HTTP 상태 코드 200(정상) 또는 202(수락)를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-288">If the delete operation is successful, the web server should respond with HTTP status code 204, indicating that the process has been successfully handled, but that the response body contains no further information (this is the same response returned by a successful PUT operation, but without a Location header as the resource no longer exists.) It is also possible for a DELETE request to return HTTP status code 200 (OK) or 202 (Accepted) if the deletion is performed asynchronously.</span></span>

```HTTP
HTTP/1.1 204 No Content
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

<span data-ttu-id="152e6-289">리소스가 없으면 웹 서버는 404(찾을 수 없음) 메시지를 대신 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-289">If the resource is not found, the web server should return a 404 (Not Found) message instead.</span></span>

> [!TIP]
> <span data-ttu-id="152e6-290">컬렉션의 모든 리소스를 삭제해야 하는 경우, 응용 프로그램이 컬렉션에서 각 리소스를 차례로 삭제하게 하는 대신 컬렉션의 URI에 대해 HTTP DELETE 요청을 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-290">If all the resources in a collection need to be deleted, enable an HTTP DELETE request to be specified for the URI of the collection rather than forcing an application to remove each resource in turn from the collection.</span></span>
>
>

### <a name="filtering-and-paginating-data"></a><span data-ttu-id="152e6-291">데이터 필터링 및 페이지 매김</span><span class="sxs-lookup"><span data-stu-id="152e6-291">Filtering and paginating data</span></span>
<span data-ttu-id="152e6-292">URI를 단순하고 직관적으로 유지하려고 노력해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-292">You should endeavor to keep the URIs simple and intuitive.</span></span> <span data-ttu-id="152e6-293">단일 URI를 통해 리소스 컬렉션을 표시하면 이렇게 하는 데 도움이 되지만, 응용 프로그램이 정보의 하위 집합만 필요한데도 대량의 데이터를 가져오게 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-293">Exposing a collection of resources through a single URI assists in this respect, but it can lead to applications fetching large amounts of data when only a subset of the information is required.</span></span> <span data-ttu-id="152e6-294">대량의 트래픽이 발생하면 웹 서버의 성능 및 확장성에도 나쁜 영향을 미칠 뿐만 아니라 데이터를 요청하는 클라이언트 응용 프로그램의 응답성도 저하됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-294">Generating a large volume of traffic impacts not only the performance and scalability of the web server but also adversely affect the responsiveness of client applications requesting the data.</span></span>

<span data-ttu-id="152e6-295">예를 들어 주문에 대해 지불한 가격이 주문에 포함된 경우, 비용이 특정 값 범위인 모든 주문을 검색해야 하는 클라이언트 응용 프로그램이 */orders* URI에서 모든 주문을 검색하고 이러한 주문을 로컬에서 필터링할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-295">For example, if orders contain the price paid for the order, a client application that needs to retrieve all orders that have a cost over a specific value might need to retrieve all orders from the */orders* URI and then filter these orders locally.</span></span> <span data-ttu-id="152e6-296">이 프로세스가 매우 비효율적인 것은 분명하며, 웹 API를 호스팅하는 서버의 네트워크 대역폭 및 처리 능력을 낭비합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-296">Clearly this process is highly inefficient; it wastes network bandwidth and processing power on the server hosting the web API.</span></span>

<span data-ttu-id="152e6-297">*/orders/ordervalue_greater_than_n*(단 *n*은 주문 가격) 같은 URI 체계를 제공하는 것이 한 가지 해결 방법이지만, 제한된 수의 가격을 제외한 모든 가격의 경우 이러한 접근 방식이 사실상 불가능합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-297">One solution may be to provide a URI scheme such as */orders/ordervalue_greater_than_n* where *n* is the order price, but for all but a limited number of prices such an approach is impractical.</span></span> <span data-ttu-id="152e6-298">또한 주문을 다른 기준에 따라 쿼리해야 하는 경우, 직관적이지 않을 수 있는 이름과 함께 URI의 긴 목록을 제공하게 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-298">Additionally, if you need to query orders based on other criteria, you can end up being faced with providing with a long list of URIs with possibly non-intuitive names.</span></span>

<span data-ttu-id="152e6-299">더 나은 데이터 필터링 방법은 웹 API에 전달되는 쿼리 문자열에 */orders?ordervaluethreshold=n* 같은 필터 조건을 제공하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-299">A better strategy to filtering data is to provide the filter criteria in the query string that is passed to the web API, such as */orders?ordervaluethreshold=n*.</span></span> <span data-ttu-id="152e6-300">이 예제에서는 웹 API의 해당 작업에서 쿼리 문자열의 `ordervaluethreshold` 매개 변수를 구문 분석 및 처리하고 필터링된 결과를 HTTP 응답에 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-300">In this example, the corresponding operation in the web API is responsible for parsing and handling the `ordervaluethreshold` parameter in the query string and returning the filtered results in the HTTP response.</span></span>

<span data-ttu-id="152e6-301">컬렉션 리소스에 대한 일부 간단한 HTTP GET 요청은 다수의 항목을 반환할 가능성이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-301">Some simple HTTP GET requests over collection resources could potentially return a large number of items.</span></span> <span data-ttu-id="152e6-302">이 문제 발생의 가능성에 대처하려면 웹 API를 단일 요청에서 반환되는 데이터의 양이 제한되도록 디자인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-302">To combat the possibility of this occurring you should design the web API to limit the amount of data returned by any single request.</span></span> <span data-ttu-id="152e6-303">사용자가 검색할 최대 항목 수(이 자체도 서비스 거부 공격을 방지하기 위해 상한값을 지정해야 할 수 있음)를 지정할 수 있는 쿼리 문자열과 컬렉션에 대한 시작 오프셋을 지원하면 이 목적을 달성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-303">You can achieve this by supporting query strings that enable the user to specify the maximum number of items to be retrieved (which could itself be subject to an upperbound limit to help prevent Denial of Service attacks), and a starting offset into the collection.</span></span> <span data-ttu-id="152e6-304">예를 들어 URI */orders?limit=25&offset=50*의 쿼리 문자열은 주문 컬렉션에서 발견된 50번째 주문부터 시작하여 주문 25개를 검색해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-304">For example, the query string in the URI */orders?limit=25&offset=50* should retrieve 25 orders starting with the 50th order found in the orders collection.</span></span> <span data-ttu-id="152e6-305">데이터 필터링과 마찬가지로 웹 API에서 GET 요청을 구현하는 작업도 쿼리 문자열의 `limit` 및 `offset` 매개 변수를 구문 분석 및 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-305">As with filtering data, the operation that implements the GET request in the web API is responsible for parsing and handling the `limit` and `offset` parameters in the query string.</span></span> <span data-ttu-id="152e6-306">클라이언트 응용 프로그램을 돕기 위해, 페이지가 매겨진 데이터를 반환하는 GET 요청은 컬렉션의 사용할 수 있는 총 리소스 수를 나타내는 모종의 메타데이터 형식을 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-306">To assist client applications, GET requests that return paginated data should also include some form of metadata that indicate the total number of resources available in the collection.</span></span> <span data-ttu-id="152e6-307">다른 지능적인 페이지 매김 방법을 생각할 수도 있으며, 자세한 내용은 [API 설계 참고 사항: 스마트 페이지 매김](http://bizcoder.com/api-design-notes-smart-paging)</span><span class="sxs-lookup"><span data-stu-id="152e6-307">You might also consider other intelligent paging strategies; for more information, see [API Design Notes: Smart Paging](http://bizcoder.com/api-design-notes-smart-paging)</span></span>

<span data-ttu-id="152e6-308">데이터를 가져올 때 정렬하는 경우에도 유사한 방법을 따를 수 있으며, 필드 이름을 */orders?sort=ProductID* 같은 값으로 가져오는 정렬 매개 변수를 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-308">You can follow a similar strategy for sorting data as it is fetched; you could provide a sort parameter that takes a field name as the value, such as */orders?sort=ProductID*.</span></span> <span data-ttu-id="152e6-309">그러나 이 접근 방식은 캐싱에 나쁜 영향을 미칠 수 있습니다(많은 캐시 구현에서 캐싱된 데이터에 대한 키로 사용하는 리소스 식별자의 쿼리 문자열 매개 변수 형식 부분).</span><span class="sxs-lookup"><span data-stu-id="152e6-309">However, note that this approach can have a deleterious effect on caching (query string parameters form part of the resource identifier used by many cache implementations as the key to cached data).</span></span>

<span data-ttu-id="152e6-310">이 접근 방식을 단일 리소스 항목에 대량의 데이터가 포함된 경우 반환되는 필드를 제한하도록 (프로젝트) 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-310">You can extend this approach to limit (project) the fields returned if a single resource item contains a large amount of data.</span></span> <span data-ttu-id="152e6-311">예를 들어 쉼표로 구분된 필드 목록을 수락하는 */orders?fields=ProductID,Quantity* 같은 쿼리 문자열 매개 변수를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-311">For example, you could use a query string parameter that accepts a comma-delimited list of fields, such as */orders?fields=ProductID,Quantity*.</span></span>

> [!TIP]
> <span data-ttu-id="152e6-312">쿼리 문자열의 모든 선택적 매개 변수에 의미 있는 기본값을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-312">Give all optional parameters in query strings meaningful defaults.</span></span> <span data-ttu-id="152e6-313">예를 들어 페이지 매김을 구현하는 경우 `limit` 매개 변수를 10으로, `offset` 매개 변수를 0으로 설정하고, 주문을 구현하는 경우 정렬 매개 변수를 리소스의 키로 설정하고, 프로젝션을 지원하는 경우 `fields` 매개 변수를 리소스의 모든 필드로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-313">For example, set the `limit` parameter to 10 and the `offset` parameter to 0 if you implement pagination, set the sort parameter to the key of the resource if you implement ordering, and set the `fields` parameter to all fields in the resource if you support projections.</span></span>
>
>

### <a name="handling-large-binary-resources"></a><span data-ttu-id="152e6-314">큰 이진 리소스 처리</span><span class="sxs-lookup"><span data-stu-id="152e6-314">Handling large binary resources</span></span>
<span data-ttu-id="152e6-315">파일 또는 이미지처럼 단일 리소스가 큰 이진 필드를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-315">A single resource may contain large binary fields, such as files or images.</span></span> <span data-ttu-id="152e6-316">신뢰할 수 없고 간헐적인 연결에서 야기되는 전송 문제를 해결하고 응답 시간을 개선하려면 클라이언트 응용 프로그램이 그러한 리소스를 청크로 검색할 수 있는 작업을 제공하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-316">To overcome the transmission problems caused by unreliable and intermittent connections and to improve response times, consider providing operations that enable such resources to be retrieved in chunks by the client application.</span></span> <span data-ttu-id="152e6-317">이렇게 하려면 웹 API 응용 프로그램이 큰 리소스의 GET 요청에 대해 Accept-Ranges 헤더를 지원해야 하며 이러한 리소스에 대해 HTTP HEAD 요청을 구현하는 것이 최적입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-317">To do this, the web API should support the Accept-Ranges header for GET requests for large resources, and ideally implement HTTP HEAD requests for these resources.</span></span> <span data-ttu-id="152e6-318">Accept-ranges 헤더는 GET 작업이 부분적인 결과를 지원하고 클라이언트 응용 프로그램이 바이트 범위로 지정된 리소스의 하위 집합을 반환하는 GET 요청을 제출할 수 있음을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-318">The Accept-Ranges header indicates that the GET operation supports partial results, and that a client application can submit GET requests that return a subset of a resource specified as a range of bytes.</span></span> <span data-ttu-id="152e6-319">HEAD 요청은 리소스 및 빈 메시지 본문을 설명하는 헤더만 반환하는 경우를 제외하고 GET 요청과 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-319">A HEAD request is similar to a GET request except that it only returns a header that describes the resource and an empty message body.</span></span> <span data-ttu-id="152e6-320">클라이언트 응용 프로그램은 부분적인 GET 요청을 사용하여 리소스를 가져올지 여부를 결정하는 HEAD 요청을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-320">A client application can issue a HEAD request to determine whether to fetch a resource by using partial GET requests.</span></span> <span data-ttu-id="152e6-321">다음 예제에서는 제품 이미지에 대 한 정보를 얻는 HEAD 요청을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-321">The following example shows a HEAD request that obtains information about a product image:</span></span>

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
...
```

<span data-ttu-id="152e6-322">응답 메시지는 리소스의 크기(4580 바이트)가 들어 있는 헤더 및 해당 GET 작업이 일부 결과를 지원하는 Accept-Ranges 헤더를 포함하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-322">The response message contains a header that includes the size of the resource (4580 bytes), and the Accept-Ranges header that the corresponding GET operation supports partial results:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
...
```

<span data-ttu-id="152e6-323">클라이언트 응용 프로그램은 이 정보를 사용하여 더 작은 청크에서 이미지를 검색하는 일련의 GET 작업을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-323">The client application can use this information to construct a series of GET operations to retrieve the image in smaller chunks.</span></span> <span data-ttu-id="152e6-324">첫 번째 요청은 범위 헤더를 사용하여 처음 2500 바이트를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-324">The first request fetches the first 2500 bytes by using the Range header:</span></span>

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
...
```

<span data-ttu-id="152e6-325">응답 메시지는 HTTP 상태 코드 206을 반환하여 이 응답이 부분 응답임을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-325">The response message indicates that this is a partial response by returning HTTP status code 206.</span></span> <span data-ttu-id="152e6-326">Content-Length 헤더는 메시지 본문에 반환된 실제 바이트 수(리소스의 크기가 아닌)를 지정하며, Content-Range 헤더는 해당 바이트가 리소스의 어느 부분인지(4580 바이트 중 바이트 0-2499)를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-326">The Content-Length header specifies the actual number of bytes returned in the message body (not the size of the resource), and the Content-Range header indicates which part of the resource this is (bytes 0-2499 out of 4580):</span></span>

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580
...
_{binary data not shown}_
```

<span data-ttu-id="152e6-327">클라이언트 응용 프로그램에서 오는 이후 요청은 해당 Range 헤더를 사용하여 리소스의 나머지 부분을 검색할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-327">A subsequent request from the client application can retrieve the remainder of the resource by using an appropriate Range header:</span></span>

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=2500-
...
```

<span data-ttu-id="152e6-328">해당하는 결과 메시지는 아래와 같아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-328">The corresponding result message should look like this:</span></span>

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2080
Content-Range: bytes 2500-4580/4580
...
```

## <a name="using-the-hateoas-approach-to-enable-navigation-to-related-resources"></a><span data-ttu-id="152e6-329">HATEOAS 접근 방식을 사용하여 관련 리소스 탐색 사용</span><span class="sxs-lookup"><span data-stu-id="152e6-329">Using the HATEOAS approach to enable navigation to related resources</span></span>
<span data-ttu-id="152e6-330">REST를 실행하는 기본적인 동기 중 하나는 URI 체계에 대해 미리 알고 있지 않아도 전체 리소스 집합을 탐색할 수 있어야 하기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-330">One of the primary motivations behind REST is that it should be possible to navigate the entire set of resources without requiring prior knowledge of the URI scheme.</span></span> <span data-ttu-id="152e6-331">각 HTTP GET 요청은 응답에 포함된 하이퍼링크를 통해 요청된 개체와 직접 관련된 리소스를 찾는 데 필요한 정보를 반환해야 하며, 이러한 각 리소스에 대해 사용할 수 있는 작업을 설명하는 정보도 제공되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-331">Each HTTP GET request should return the information necessary to find the resources related directly to the requested object through hyperlinks included in the response, and it should also be provided with information that describes the operations available on each of these resources.</span></span> <span data-ttu-id="152e6-332">이 원칙을 HATEOAS(Hypertext as the Engine of Application State)라 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-332">This principle is known as HATEOAS, or Hypertext as the Engine of Application State.</span></span> <span data-ttu-id="152e6-333">시스템은 실질적으로 유한 상태 시스템으로서, 각 요청에 대한 응답은 한 상태에서 다른 상태로 바꾸는 데 필요한 정보를 포함하고 있으며, 다른 정보는 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-333">The system is effectively a finite state machine, and the response to each request contains the information necessary to move from one state to another; no other information should be necessary.</span></span>

> [!NOTE]
> <span data-ttu-id="152e6-334">현재 HATEOAS 원칙을 모델링하는 방법을 정의하는 표준 또는 사양은 없습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-334">Currently there are no standards or specifications that define how to model the HATEOAS principle.</span></span> <span data-ttu-id="152e6-335">이 섹션에 나오는 예제에서는 한 가지 가능한 해결 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-335">The examples shown in this section illustrate one possible solution.</span></span>
>
>

<span data-ttu-id="152e6-336">한 예로, 고객과 주문 간의 관계를 처리하려면 특정 주문에 대한 응답에 반환된 데이터가 주문을 실행한 고객을 식별하는 하이퍼링크 형식의 URI 및 해당 고객에 대해 수행할 수 있는 작업을 포함하고 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-336">As an example, to handle the relationship between customers and orders, the data returned in the response for a specific order should contain URIs in the form of a hyperlink identifying the customer that placed the order, and the operations that can be performed on that customer.</span></span>

```HTTP
GET http://adventure-works.com/orders/3 HTTP/1.1
Accept: application/json
...
```

<span data-ttu-id="152e6-337">응답 메시지의 본문은 관계의 특성(*Customer*)을 지정하는 `links` 배열(코드 예제에 강조 표시됨), 고객의 URI(*http://adventure-works.com/customers/3*), 이 고객의 세부 정보를 검색하는 방법(*GET*) 및 웹 서버가 이 정보를 검색하기 위해 지원하는 MIME 형식(*text/xml* 및 *application/json*)을 포함하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-337">The body of the response message contains a `links` array (highlighted in the code example) that specifies the nature of the relationship (*Customer*), the URI of the customer (*http://adventure-works.com/customers/3*), how to retrieve the details of this customer (*GET*), and the MIME types that the web server supports for retrieving this information (*text/xml* and *application/json*).</span></span> <span data-ttu-id="152e6-338">이 정보가 모두 있어야 클라이언트 응용 프로그램이 고객의 세부 정보를 가져올 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-338">This is all the information that a client application needs to be able to fetch the details of the customer.</span></span> <span data-ttu-id="152e6-339">또한 링크 배열은 PUT(고객 수정, 웹 서버가 클라이언트에서 제공할 것으로 기대하는 형식 포함) 및 DELETE 등 수행할 수 있는 다른 작업에 대한 링크도 포함하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-339">Additionally, the Links array also includes links for the other operations that can be performed, such as PUT (to modify the customer, together with the format that the web server expects the client to provide), and DELETE.</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[(some links omitted){"rel":"customer","href":" http://adventure-works.com/customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":"
customer","href":" http://adventure-works.com /customers/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"customer","href":" http://adventure-works.com /customers/3","action":"DELETE","types":[]}]}
```

<span data-ttu-id="152e6-340">완전성을 위해, 링크 배열은 검색된 리소스와 관련된 자체 참조 정보도 포함하고 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-340">For completeness, the Links array should also include self-referencing information pertaining to the resource that has been retrieved.</span></span> <span data-ttu-id="152e6-341">이러한 링크는 이전 예제에서 생략되었지만, 다음 코드에는 강조 표시되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-341">These links have been omitted from the previous example, but are highlighted in the following code.</span></span> <span data-ttu-id="152e6-342">이러한 링크에서 관계 *self*를 사용하여 이것이 작업에서 반환되는 리소스에 대한 참조임을 나타냈습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-342">Notice that in these links, the relationship *self* has been used to indicate that this is a reference to the resource being returned by the operation:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[{"rel":"self","href":" http://adventure-works.com/orders/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" self","href":" http://adventure-works.com /orders/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"self","href":" http://adventure-works.com /orders/3", "action":"DELETE","types":[]},{"rel":"customer",
"href":" http://adventure-works.com /customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" customer" (customer links omitted)}]}
```

<span data-ttu-id="152e6-343">이 접근 방식이 효과적으로 적용되려면 이 추가 정보를 검색하고 구문 분석하도록 클라이언트 응용 프로그램을 준비해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-343">For this approach to be effective, client applications must be prepared to retrieve and parse this additional information.</span></span>

## <a name="versioning-a-restful-web-api"></a><span data-ttu-id="152e6-344">RESTful 웹 API 버전 관리</span><span class="sxs-lookup"><span data-stu-id="152e6-344">Versioning a RESTful web API</span></span>
<span data-ttu-id="152e6-345">가장 단순한 상황을 제외하고 웹 API가 정적으로 남아 있을 가능성은 거의 없습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-345">It is highly unlikely that in all but the simplest of situations that a web API will remain static.</span></span> <span data-ttu-id="152e6-346">비즈니스 요구 사항이 변경됨에 따라 자원의 새 컬렉션이 추가될 수 있으므로, 리소스 간의 관계가 변할 수 있으며 리소스 데이터의 구조가 수정될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-346">As business requirements change new collections of resources may be added, the relationships between resources might change, and the structure of the data in resources might be amended.</span></span> <span data-ttu-id="152e6-347">웹 API를 새로운 또는 서로 다른 요구 사항을 처리하도록 업데이트하는 동안 해당 변경이 웹 API를 사용하는 클라이언트 응용 프로그램에 미치는 영향을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-347">While updating a web API to handle new or differing requirements is a relatively straightforward process, you must consider the effects that such changes will have on client applications consuming the web API.</span></span> <span data-ttu-id="152e6-348">문제는 개발자가 해당 API를 완전히 제어할 수 있는 웹 API를 디자인 및 구현하더라도, 해당 개발자는 원격으로 작업하는 제3자 조직이 구축할 수 있는 클라이언트 응용 프로그램을 같은 정도로 제어하지 못한다는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-348">The issue is that although the developer designing and implementing a web API has full control over that API, the developer does not have the same degree of control over client applications which may be built by third party organizations operating remotely.</span></span> <span data-ttu-id="152e6-349">따라서 새 클라이언트 응용 프로그램이 새 기능과 리소스의 장점을 이용할 수 있도록 하면서도 기존 클라이언트 응용 프로그램이 변경되지 않고 계속 작동할 수 있도록 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-349">The primary imperative is to enable existing client applications to continue functioning unchanged while allowing new client applications to take advantage of new features and resources.</span></span>

<span data-ttu-id="152e6-350">버전 관리를 사용하면 웹 API는 자신이 표시하는 기능과 리소스를 나타낼 수 있으며, 클라이언트 응용 프로그램은 기능 또는 리소스의 특정 버전으로 지정된 요청을 제출할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-350">Versioning enables a web API to indicate the features and resources that it exposes, and a client application can submit requests that are directed to a specific version of a feature or resource.</span></span> <span data-ttu-id="152e6-351">다음 섹션에서는 각각 자체의 이점과 절충점을 가지고 있는 다양한 접근 방식을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-351">The following sections describe several different approaches, each of which has its own benefits and trade-offs.</span></span>

### <a name="no-versioning"></a><span data-ttu-id="152e6-352">버전 관리 없음</span><span class="sxs-lookup"><span data-stu-id="152e6-352">No versioning</span></span>
<span data-ttu-id="152e6-353">가장 간단한 방법이며 일부 내부 API에 대해 허용될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-353">This is the simplest approach, and may be acceptable for some internal APIs.</span></span> <span data-ttu-id="152e6-354">큰 변화는 새 리소스 또는 새 연결로 나타낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-354">Big changes could be represented as new resources or new links.</span></span>  <span data-ttu-id="152e6-355">기존 리소스에 콘텐츠를 추가해도 이 콘텐츠가 표시될 것으로 예상하지 않은 클라이언트 응용 프로그램은 해당 콘텐츠를 무시할 것이므로 주요 변경 내용이 표시되지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-355">Adding content to existing resources might not present a breaking change as client applications that are not expecting to see this content will simply ignore it.</span></span>

<span data-ttu-id="152e6-356">예를 들어 URI *http://adventure-works.com/customers/3*에 대한 요청은 클라이언트 응용 프로그램이 예상하는 `id`, `name` 및 `address` 필드가 포함된 단일 고객의 세부 정보를 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-356">For example, a request to the URI *http://adventure-works.com/customers/3* should return the details of a single customer containing `id`, `name`, and `address` fields expected by the client application:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [!NOTE]
> <span data-ttu-id="152e6-357">간단명료하게 하기 위해 이 섹션에 표시된 예제 응답은 HATEOAS 링크를 포함하고 있지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-357">For the purposes of simplicity and clarity, the example responses shown in this section do not include HATEOAS links.</span></span>
>
>

<span data-ttu-id="152e6-358">`DateCreated` 필드가 고객 리소스의 체계에 추가되면 응답은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-358">If the `DateCreated` field is added to the schema of the customer resource, then the response would look like this:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="152e6-359">기존 클라이언트 응용 프로그램은 인식되지 않은 필드를 무시할 수 있으면 계속 올바르게 작동할 수 있으며, 한편 새 클라이언트 응용 프로그램을 새 필드를 처리하도록 디자인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-359">Existing client applications might continue functioning correctly if they are capable of ignoring unrecognized fields, while new client applications can be designed to handle this new field.</span></span> <span data-ttu-id="152e6-360">그러나 리소스가 더 크게 변경되거나(필드 제거 또는 이름 변경 등) 리소스 간의 관계가 변경된 경우에는 이러한 변화가 주요 변경 내용으로 인식되어 기존 클라이언트 응용 프로그램이 올바르게 작동하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-360">However, if more radical changes to the schema of resources occur (such as removing or renaming fields) or the relationships between resources change then these may constitute breaking changes that prevent existing client applications from functioning correctly.</span></span> <span data-ttu-id="152e6-361">이러한 상황에서는 다음 방법 중 하나를 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-361">In these situations you should consider one of the following approaches.</span></span>

### <a name="uri-versioning"></a><span data-ttu-id="152e6-362">URI 버전 관리</span><span class="sxs-lookup"><span data-stu-id="152e6-362">URI versioning</span></span>
<span data-ttu-id="152e6-363">웹 API를 수정하거나 리소스의 체계를 변경할 때마다 각 리소스의 URI에 버전 번호를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-363">Each time you modify the web API or change the schema of resources, you add a version number to the URI for each resource.</span></span> <span data-ttu-id="152e6-364">앞에서는 기존 URI가 전과 같이 계속 작동하여 원래 체계를 준수하는 리소스를 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-364">The previously existing URIs should continue to operate as before, returning resources that conform to their original schema.</span></span>

<span data-ttu-id="152e6-365">앞의 예제를 확장하여 `address` 필드가 주소의 각 구성 부분을 포함하고 있는 하위 필드(예: `streetAddress`, `city`, `state` 및 `zipCode`)로 재구성된다면, http://adventure-works.com/v2/customers/3과 같은 버전 번호가 들어 있는 URI를 통해 리소스의 이 버전을 표시할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-365">Extending the previous example, if the `address` field is restructured into sub-fields containing each constituent part of the address (such as `streetAddress`, `city`, `state`, and `zipCode`), this version of the resource could be exposed through a URI containing a version number, such as http://adventure-works.com/v2/customers/3:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="152e6-366">이 버전 관리 메커니즘은 매우 간단하지만 요청을 적절한 끝점으로 라우팅하는 서버에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-366">This versioning mechanism is very simple but depends on the server routing the request to the appropriate endpoint.</span></span> <span data-ttu-id="152e6-367">그러나 여러 번 반복을 통해 웹 API가 성숙해짐에 따라 이 메커니즘을 다룰 수 없게 될 수 있으며 서버가 다양한 버전을 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-367">However, it can become unwieldy as the web API matures through several iterations and the server has to support a number of different versions.</span></span> <span data-ttu-id="152e6-368">또한 엄격히 말해서, 클라이언트 응용 프로그램이 같은 데이터(고객 3)를 가져오므로, URI가 버전에 따라 달라져서는 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-368">Also, from a purist’s point of view, in all cases the client applications are fetching the same data (customer 3), so the URI should not really be different depending on the version.</span></span> <span data-ttu-id="152e6-369">또한 이 체계는 모든 링크가 자신의 URI에 버전 번호를 포함해야 하므로 HATEOAS 구현을 복잡하게 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-369">This scheme also complicates implementation of HATEOAS as all links will need to include the version number in their URIs.</span></span>

### <a name="query-string-versioning"></a><span data-ttu-id="152e6-370">쿼리 문자열 버전 관리</span><span class="sxs-lookup"><span data-stu-id="152e6-370">Query string versioning</span></span>
<span data-ttu-id="152e6-371">복수 URI를 제공하는 대신에 HTTP 요청에 추가된 쿼리 문자열 내에 *http://adventure-works.com/customers/3?version=2* 같은 매개 변수를 사용하여 리소스의 버전을 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-371">Rather than providing multiple URIs, you can specify the version of the resource by using a parameter within the query string appended to the HTTP request, such as *http://adventure-works.com/customers/3?version=2*.</span></span> <span data-ttu-id="152e6-372">버전 매개 변수는 이전 클라이언트 응용 프로그램에서 생략했다면 기본적으로 1과 같은 의미 있는 값입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-372">The version parameter should default to a meaningful value such as 1 if it is omitted by older client applications.</span></span>

<span data-ttu-id="152e6-373">이 접근 방식은 같은 리소스가 언제나 같은 URI에서 검색된다는 의미 체계 장점이 있지만, 쿼리 문자열을 구문 분석하고 해당 HTTP 응답을 다시 보내기 위해 요청을 처리하는 코드에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-373">This approach has the semantic advantage that the same resource is always retrieved from the same URI, but it depends on the code that handles the request to parse the query string and send back the appropriate HTTP response.</span></span> <span data-ttu-id="152e6-374">또한 이 접근 방식은 HATEOAS를 URI 버전 관리 메커니즘으로 구현할 때와 같이 복잡합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-374">This approach also suffers from the same complications for implementing HATEOAS as the URI versioning mechanism.</span></span>

> [!NOTE]
> <span data-ttu-id="152e6-375">일부 구형 웹 브라우저와 웹 프록시는 URL에 쿼리 문자열을 포함하는 요청에 대한 응답을 캐싱하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-375">Some older web browsers and web proxies will not cache responses for requests that include a query string in the URL.</span></span> <span data-ttu-id="152e6-376">이는 웹 API를 사용하고 해당 웹 브라우저 내에서 실행되는 웹 응용 프로그램의 성능을 저하시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-376">This can have an adverse impact on performance for web applications that use a web API and that run from within such a web browser.</span></span>
>
>

### <a name="header-versioning"></a><span data-ttu-id="152e6-377">헤더 버전 관리</span><span class="sxs-lookup"><span data-stu-id="152e6-377">Header versioning</span></span>
<span data-ttu-id="152e6-378">버전 번호를 쿼리 문자열 매개 변수로 추가하지 않고 리소스의 버전을 나타내는 사용자 지정 헤더를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-378">Rather than appending the version number as a query string parameter, you could implement a custom header that indicates the version of the resource.</span></span> <span data-ttu-id="152e6-379">이 접근 방식을 사용하려면 클라이언트 응용 프로그램이 적절한 헤더를 요청에 추가해야 하지만, version 헤더가 생략된 경우 클라이언트 요청을 처리하는 코드가 기본값(버전 1)을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-379">This approach requires that the client application adds the appropriate header to any requests, although the code handling the client request could use a default value (version 1) if the version header is omitted.</span></span> <span data-ttu-id="152e6-380">다음 예제에서는 *Custom-Header*라는 사용자 지정 헤더를 이용합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-380">The following examples utilize a custom header named *Custom-Header*.</span></span> <span data-ttu-id="152e6-381">이 헤더의 값은 웹 API의 버전을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-381">The value of this header indicates the version of web API.</span></span>

<span data-ttu-id="152e6-382">버전 1:</span><span class="sxs-lookup"><span data-stu-id="152e6-382">Version 1:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=1
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="152e6-383">버전 2:</span><span class="sxs-lookup"><span data-stu-id="152e6-383">Version 2:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=2
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

<span data-ttu-id="152e6-384">참고로 이전의 두 방법 방식과 마찬가지로 HATEOAS를 구현하려면 모든 링크에 적절한 사용자 지정 헤더를 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-384">Note that as with the previous two approaches, implementing HATEOAS requires including the appropriate custom header in any links.</span></span>

### <a name="media-type-versioning"></a><span data-ttu-id="152e6-385">미디어 형식 버전 관리</span><span class="sxs-lookup"><span data-stu-id="152e6-385">Media type versioning</span></span>
<span data-ttu-id="152e6-386">이 지침의 앞부분에서 설명한 대로, 클라이언트 응용 프로그램은 웹 서버에 HTTP GET 요청을 보낼 때 Accept 헤더를 사용하여 처리할 수 있는 콘텐츠의 형식을 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-386">When a client application sends an HTTP GET request to a web server it should stipulate the format of the content that it can handle by using an Accept header, as described earlier in this guidance.</span></span> <span data-ttu-id="152e6-387">흔히 *Accept* 헤더의 목적은 클라이언트 응용 프로그램에서 응답 본문이 XML, JSON 또는 클라이언트가 구문 분석할 수 있는 몇몇 다른 일반적인 형식 중 어느 형식인지 지정할 수 있도록 하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-387">Frequently the purpose of the *Accept* header is to allow the client application to specify whether the body of the response should be XML, JSON, or some other common format that the client can parse.</span></span> <span data-ttu-id="152e6-388">그러나 클라이언트 응용 프로그램이 예상하는 리소스의 버전을 나타낼 수 있도록 하는 정보를 포함한 사용자 지정 미디어 형식을 정의할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-388">However, it is possible to define custom media types that include information enabling the client application to indicate which version of a resource it is expecting.</span></span> <span data-ttu-id="152e6-389">다음 예제는 *Accept* 헤더를 값 *application/vnd.adventure-works.v1+json*과 함께 지정하는 요청을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-389">The following example shows a request that specifies an *Accept* header with the value *application/vnd.adventure-works.v1+json*.</span></span> <span data-ttu-id="152e6-390">*vnd.adventure-works.v1* 요소는 웹 서버에 대해 리소스의 버전 1을 반환해야 한다는 것을 나타내며, 한편 *json* 요소는 응답 본문의 형식이 JSON이어야 함을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-390">The *vnd.adventure-works.v1* element indicates to the web server that it should return version 1 of the resource, while the *json* element specifies that the format of the response body should be JSON:</span></span>

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Accept: application/vnd.adventure-works.v1+json
...
```

<span data-ttu-id="152e6-391">요청을 처리하는 코드는 *Accept* 헤더를 처리하고 가능하면 해당 헤더를 적용해야 합니다. 클라이언트 응용 프로그램은 *Accept* 헤더에 복수의 형식을 지정할 수 있으며, 이 경우 웹 서버는 응답 본문에 가장 적절한 형식을 선택할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-391">The code handling the request is responsible for processing the *Accept* header and honoring it as far as possible (the client application may specify multiple formats in the *Accept* header, in which case the web server can choose the most appropriate format for the response body).</span></span> <span data-ttu-id="152e6-392">웹 서버는 Content-Type 헤더를 사용하여 응답 본문에 있는 데이터의 형식을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-392">The web server confirms the format of the data in the response body by using the Content-Type header:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

<span data-ttu-id="152e6-393">Accept 헤더가 모든 알려진 미디어 형식을 지정하지 않은 경우, 웹 서버는 HTTP 406(승인 금지) 응답 메시지를 생성하거나 기본 미디어 형식이 포함된 메시지를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-393">If the Accept header does not specify any known media types, the web server could generate an HTTP 406 (Not Acceptable) response message or return a message with a default media type.</span></span>

<span data-ttu-id="152e6-394">이 접근 방식은 엄격히 말해서 버전 관리 메커니즘인지 여부에 대한 논란의 여지가 있으며 당연히 리소스 링크에 관련 데이터의 MIME 형식을 포함할 수 있는 HATEOAS에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-394">This approach is arguably the purest of the versioning mechanisms and lends itself naturally to HATEOAS, which can include the MIME type of related data in resource links.</span></span>

> [!NOTE]
> <span data-ttu-id="152e6-395">버전 관리 전략을 선택할 때에는 성능에 미치는 영향, 특히 웹 서버의 캐싱을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-395">When you select a versioning strategy, you should also consider the implications on performance, especially caching on the web server.</span></span> <span data-ttu-id="152e6-396">URI 버전 관리 및 쿼리 문자열 버전 관리 체계는 같은 URI/쿼리 문자열 조합이 매번 같은 데이터를 참조하므로 캐싱하기에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-396">The URI versioning and Query String versioning schemes are cache-friendly inasmuch as the same URI/query string combination refers to the same data each time.</span></span>
>
> <span data-ttu-id="152e6-397">일반적으로 헤더 버전 관리 및 미디어 형식 버전 관리 메커니즘에는 사용자 지정 헤더 또는 Accept 헤더의 값을 검사하기 위해 추가 논리가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-397">The Header versioning and Media Type versioning mechanisms typically require additional logic to examine the values in the custom header or the Accept header.</span></span> <span data-ttu-id="152e6-398">대규모 환경의 경우, 서로 다른 버전의 웹 API를 사용하는 많은 클라이언트가 서버 쪽 캐시에 상당한 양의 중복된 데이터를 발생시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-398">In a large-scale environment, many clients using different versions of a web API can result in a significant amount of duplicated data in a server-side cache.</span></span> <span data-ttu-id="152e6-399">클라이언트 응용 프로그램이 캐싱을 구현하는 프록시를 통해 웹 서버와 통신하는 경우 이 문제가 심각할 수 있으며, 현재 요청된 데이터의 복사본을 자체의 캐시에 저장하지 않은 경우에만 요청을 웹 서버에 전달해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-399">This issue can become acute if a client application communicates with a web server through a proxy that implements caching, and that only forwards a request to the web server if it does not currently hold a copy of the requested data in its cache.</span></span>
>
>

## <a name="open-api-initiative"></a><span data-ttu-id="152e6-400">Open API Initiative</span><span class="sxs-lookup"><span data-stu-id="152e6-400">Open API Initiative</span></span>
<span data-ttu-id="152e6-401">[Open API Initiative](https://www.openapis.org/)는 공급업체에서 REST API 설명을 표준화하기 위해 업계 컨소시엄에서 만들었습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-401">The [Open API Initiative](https://www.openapis.org/) was created by an industry consortium to standardize REST API descriptions across vendors.</span></span> <span data-ttu-id="152e6-402">이 이니셔티브의 일부로, Swagger 2.0 사양의 명칭이 OAS(Open API Specification)로 바뀐 후 Open API Initiative에 추가되었습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-402">As part of this initiative, the Swagger 2.0 specification was renamed the OpenAPI Specification (OAS) and brought under the Open API Initiative.</span></span>

<span data-ttu-id="152e6-403">사용하는 웹 API에 OpenAPI를 채택할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-403">You may want to adopt OpenAPI for your web APIs.</span></span> <span data-ttu-id="152e6-404">몇 가지 고려할 점은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-404">Some points to consider:</span></span>

- <span data-ttu-id="152e6-405">OpenAPI 사양에는 REST API 디자인 방식에 대한 독자적인 지침이 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-405">The OpenAPI Specification comes with with a set of opinionated guidelines on how a REST API should be designed.</span></span> <span data-ttu-id="152e6-406">이 사양은 상호 운용성 측면에서는 이점이 있지만, 사양에 맞게 API를 디자인할 때는 좀 더 주의해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-406">That has advantages for interoperability, but requires more care when designing your API to conform to the specification.</span></span>
- <span data-ttu-id="152e6-407">OpenAPI는 구현 우선 방식이 아닌, 계약 우선 방식을 권장합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-407">OpenAPI promotes a contract-first approach, rather than an implementation-first approach.</span></span> <span data-ttu-id="152e6-408">계약 우선 방식에서는 API 계약(인터페이스)을 먼저 디자인한 후 계약을 구현하는 코드를 작성합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-408">Contract-first means you design the API contract (the interface) first and then write code that implements the contract.</span></span> 
- <span data-ttu-id="152e6-409">Swagger와 같은 도구는 API 계약에서 클라이언트 라이브러리 또는 문서를 생성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-409">Tools like Swagger can generate client libraries or documentation from API contracts.</span></span> <span data-ttu-id="152e6-410">예를 들어, [Swagger를 사용하는 ASP.NET 웹 API 도움말 페이지](/aspnet/core/tutorials/web-api-help-pages-using-swagger)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="152e6-410">For example, see [ASP.NET Web API Help Pages using Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).</span></span>

## <a name="more-information"></a><span data-ttu-id="152e6-411">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="152e6-411">More information</span></span>
* <span data-ttu-id="152e6-412">[Microsoft REST API 지침](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md)에는 공용 REST API 디자인에 대한 자세한 권장 사항이 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-412">The [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md) contain detailed recommendations for designing public REST APIs.</span></span>
* <span data-ttu-id="152e6-413">[RESTful Cookbook](http://restcookbook.com/) 은 RESTful API를 구축하는 방법을 소개합니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-413">The [RESTful Cookbook](http://restcookbook.com/) contains an introduction to building RESTful APIs.</span></span>
* <span data-ttu-id="152e6-414">[웹 API 검사 목록](https://mathieu.fenniak.net/the-api-checklist/)에는 웹 API를 디자인 및 구현할 때 고려할 유용한 항목 목록이 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-414">The [Web API Checklist](https://mathieu.fenniak.net/the-api-checklist/) contains a useful list of items to consider when designing and implementing a web API.</span></span>
* <span data-ttu-id="152e6-415">[Open API Initiative](https://www.openapis.org/) 사이트에는 Open API에 대한 모든 관련 설명서 및 구현 세부 정보가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="152e6-415">The [Open API Initiative](https://www.openapis.org/) site, contains all related documentation and implementation details on Open API.</span></span>
