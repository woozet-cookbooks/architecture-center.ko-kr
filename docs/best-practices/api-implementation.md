---
title: API 구현 지침
description: API를 구현하는 방법에 대한 지침입니다.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: cc28864de36afdeed2f8a7155a307e312c3a398e
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/05/2018
---
# <a name="api-implementation"></a><span data-ttu-id="6f4c2-103">API 구현</span><span class="sxs-lookup"><span data-stu-id="6f4c2-103">API implementation</span></span>

<span data-ttu-id="6f4c2-104">신중하게 설계된 RESTful Web API는 클라이언트 응용 프로그램에 액세스할 수 있는 리소스, 관계, 탐색 스키마를 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-104">A carefully-designed RESTful web API defines the resources, relationships, and navigation schemes that are accessible to client applications.</span></span> <span data-ttu-id="6f4c2-105">Web API를 구현하고 배포하는 경우 Web API를 호스팅하는 환경의 실제 요구 사항을 고려하고 데이터의 논리 구조보다는 Web API가 생성된 방식을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-105">When you implement and deploy a web API, you should consider the physical requirements of the environment hosting the web API and the way in which the web API is constructed rather than the logical structure of the data.</span></span> <span data-ttu-id="6f4c2-106">이 지침은 Web API를 구현하고 클라이언트 응용 프로그램에서 사용할 수 있도록 게시하는 것에 대한 모범 사례를 집중적으로 살펴봅니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-106">This guidance focusses on best practices for implementing a web API and publishing it to make it available to client applications.</span></span> <span data-ttu-id="6f4c2-107">Web API 디자인에 대한 자세한 정보는 [API 디자인 지침](/azure/architecture/best-practices/api-design)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-107">For detailed information about web API design, see [API Design Guidance](/azure/architecture/best-practices/api-design).</span></span>

## <a name="processing-requests"></a><span data-ttu-id="6f4c2-108">요청 처리</span><span class="sxs-lookup"><span data-stu-id="6f4c2-108">Processing requests</span></span>

<span data-ttu-id="6f4c2-109">요청을 처리하기 위해 코드를 구현하는 경우 다음 사항을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-109">Consider the following points when you implement the code to handle requests.</span></span>

### <a name="get-put-delete-head-and-patch-actions-should-be-idempotent"></a><span data-ttu-id="6f4c2-110">GET, PUT, DELETE, HEAD 및 PATCH 작업은 멱등원이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-110">GET, PUT, DELETE, HEAD, and PATCH actions should be idempotent</span></span>

<span data-ttu-id="6f4c2-111">이러한 요청을 구현하는 코드는 파생 효과를 부과하지 말아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-111">The code that implements these requests should not impose any side-effects.</span></span> <span data-ttu-id="6f4c2-112">동일한 리소스에 대해 동일한 요청이 반복되면 그 결과는 동일한 상태여야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-112">The same request repeated over the same resource should result in the same state.</span></span> <span data-ttu-id="6f4c2-113">예를 들어 동일한 URI로 여러 개의 DELETE 요청을 보내면 응답 메시지의 HTTP 상태 코드가 다를 수 있습지만 동일한 결과가 발생해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-113">For example, sending multiple DELETE requests to the same URI should have the same effect, although the HTTP status code in the response messages may be different.</span></span> <span data-ttu-id="6f4c2-114">첫 번째 DELETE 요청은 상태 코드 204(콘텐츠 없음)를 반환할 수 있습니다. 반면 후속 DELETE 요청은 상태 코드 404(찾을 수 없음)를 반환할 수 있습니다</span><span class="sxs-lookup"><span data-stu-id="6f4c2-114">The first DELETE request might return status code 204 (No Content), while a subsequent DELETE request might return status code 404 (Not Found).</span></span>

> [!NOTE]
> <span data-ttu-id="6f4c2-115">Jonathan Oliver의 블로그에 있는 [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) (멱등 패턴) 문서에는 멱등에 대한 개요 및 데이터 관리 옵션과의 관계가 제공되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-115">The article [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog provides an overview of idempotency and how it relates to data management operations.</span></span>
>

### <a name="post-actions-that-create-new-resources-should-not-have-unrelated-side-effects"></a><span data-ttu-id="6f4c2-116">새 리소스를 생성하는 POST 작업에는 무관한 파생 효과가 없어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-116">POST actions that create new resources should not have unrelated side-effects</span></span>

<span data-ttu-id="6f4c2-117">POST 요청이 새 리소스를 생성하기 위한 것이라면, 그 효과는 새 리소스(및 연계가 있는 경우에는 가급적 직접적으로 연관된 리소스)로 한정되어야 합니다. 예를 들어, 전자 상거래 시스템에서 고객을 위해 새 주문을 생성하는 POST 요청은 제고 수준을 변경하고 대금 청구 정보도 생성하지만 주문과 직접적인 관련이 없거나 시스템의 전반적인 상태에 다른 파생 효과를 미치는 정보는 수정하지 말아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-117">If a POST request is intended to create a new resource, the effects of the request should be limited to the new resource (and possibly any directly related resources if there is some sort of linkage involved) For example, in an ecommerce system, a POST request that creates a new order for a customer might also amend inventory levels and generate billing information, but it should not modify information not directly related to the order or have any other side-effects on the overall state of the system.</span></span>

### <a name="avoid-implementing-chatty-post-put-and-delete-operations"></a><span data-ttu-id="6f4c2-118">번잡한 POST, PUT 및 DELETE 작업을 구현하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-118">Avoid implementing chatty POST, PUT, and DELETE operations</span></span>

<span data-ttu-id="6f4c2-119">리소스 컬렉션을 통해 POST, PUT 및 DELETE 요청을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-119">Support POST, PUT and DELETE requests over resource collections.</span></span> <span data-ttu-id="6f4c2-120">POST 요청은 다수의 새로운 리소스에 대한 세부 정보를 포함할 수 있으며 그 모두를 동일한 컬렉션에 추가할 수 있고, PUT 요청은 컬렉션에 포함된 전체 리소스를 바꿀 수 있고, DELETE 요청은 컬렉션 전체를 제거할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-120">A POST request can contain the details for multiple new resources and add them all to the same collection, a PUT request can replace the entire set of resources in a collection, and a DELETE request can remove an entire collection.</span></span>

<span data-ttu-id="6f4c2-121">ASP.NET Web API 2에 포함된 OData 지원은 여러 요청을 일괄 처리할 수 있는 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-121">The OData support included in ASP.NET Web API 2 provides the ability to batch requests.</span></span> <span data-ttu-id="6f4c2-122">클라이언트 응용 프로그램은 여러 개의 Web API 요청을 패키지로 만들어서 단일 HTTP 요청으로 서버에 보낼 수 있고, 각 요청에 대한 응답을 포함하는 단일 HTTP 응답을 수신할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-122">A client application can package up several web API requests and send them to the server in a single HTTP request, and receive a single HTTP response that contains the replies to each request.</span></span> <span data-ttu-id="6f4c2-123">자세한 내용은 [Web API 및 Web API OData의 Batch 지원 소개](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-123">For more information, [Introducing Batch Support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx).</span></span>

### <a name="follow-the-http-specification-when-sending-a-response"></a><span data-ttu-id="6f4c2-124">응답을 보낼 때 HTTP 사양을 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-124">Follow the HTTP specification when sending a response</span></span> 

<span data-ttu-id="6f4c2-125">Web API는 클라이언트가 결과를 처리할 방법을 판단할 수 있도록 올바른 HTTP 상태 코드를 포함하고, 클라이언트가 결과의 특성을 이해할 수 있도록 적절한 HTTP 헤더를 포함하고, 클라이언트가 결과를 분석할 수 있도록 적절한 형식으로 구성된 본문을 포함하는 메시지를 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-125">A web API must return messages that contain the correct HTTP status code to enable the client to determine how to handle the result, the appropriate HTTP headers so that the client understands the nature of the result, and a suitably formatted body to enable the client to parse the result.</span></span> 

<span data-ttu-id="6f4c2-126">예를 들어 POST 작업은 상태 코드 201(생성됨)을 반환해야 하고 응답 메시지는 응답 메시지의 위치 헤더에 새로 생성된 리소스의 URI를 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-126">For example, a POST operation should return status code 201 (Created) and the response message should include the URI of the newly created resource in the Location header of the response message.</span></span>

### <a name="support-content-negotiation"></a><span data-ttu-id="6f4c2-127">콘텐츠 협상을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-127">Support content negotiation</span></span>

<span data-ttu-id="6f4c2-128">응답 메시지의 본문에는 다양한 형식의 데이터가 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-128">The body of a response message may contain data in a variety of formats.</span></span> <span data-ttu-id="6f4c2-129">예를 들어 HTTP GET 요청은 JSON 또는 XML 형식으로 데이터를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-129">For example, an HTTP GET request could return data in JSON, or XML format.</span></span> <span data-ttu-id="6f4c2-130">클라이언트가 요청을 제출할 때, 클라이언트가 처리할 수 있는 데이터 형식을 지정하는 Accept 헤더를 요청에 포함시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-130">When the client submits a request, it can include an Accept header that specifies the data formats that it can handle.</span></span> <span data-ttu-id="6f4c2-131">이러한 형식은 미디어 형식으로 지정됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-131">These formats are specified as media types.</span></span> <span data-ttu-id="6f4c2-132">예를 들어, 이미지를 검색하는 GET 요청을 생성하는 클라이언트는 클라이언트가 처리할 수 있는 미디어 유형을 나열하는(예: "image/jpeg, image/gif, image/png") Accept 헤더를 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-132">For example, a client that issues a GET request that retrieves an image can specify an Accept header that lists the media types that the client can handle, such as "image/jpeg, image/gif, image/png".</span></span>  <span data-ttu-id="6f4c2-133">Web API에서 결과를 반환할 때 이러한 미디어 유형 중 하나를 데이터 형식으로 사용해야 하며, 응답의 Content-Type 헤더에 해당 형식을 명시해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-133">When the web API returns the result, it should format the data by using one of these media types and specify the format in the Content-Type header of the response.</span></span>

<span data-ttu-id="6f4c2-134">클라이언트에서 Accept 헤더를 명시하지 않는 경우, 응답 본문에 대해 합당한 기본 형식을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-134">If the client does not specify an Accept header, then use a sensible default format for the response body.</span></span> <span data-ttu-id="6f4c2-135">한 예로, ASP.NET Web API 프레임워크는 텍스트 기반 데이터에 대해 JSON을 기본 형식으로 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-135">As an example, the ASP.NET Web API framework defaults to JSON for text-based data.</span></span>

### <a name="provide-links-to-support-hateoas-style-navigation-and-discovery-of-resources"></a><span data-ttu-id="6f4c2-136">HATEOAS 스타일 탐색 및 리소스 발견을 지원하는 링크를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-136">Provide links to support HATEOAS-style navigation and discovery of resources</span></span>

<span data-ttu-id="6f4c2-137">HATEOAS 접근 방식을 통해 클라이언트가 초기 시작 지점으로부터 리소스를 발견하고 탐색할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-137">The HATEOAS approach enables a client to navigate and discover resources from an initial starting point.</span></span> <span data-ttu-id="6f4c2-138">이것은 URI를 포함하는 링크를 사용하여 이루어 집니다. 클라이언트가 리소스를 확보하기 위해 HTTP GET 요청을 발행하는 경우, 직접적으로 연관된 리소스의 위치를 클라이언트 응용 프로그램이 신속하게 찾을 수 있도록 하는 URI가 응답에 포함되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-138">This is achieved by using links containing URIs; when a client issues an HTTP GET request to obtain a resource, the response should contain URIs that enable a client application to quickly locate any directly related resources.</span></span> <span data-ttu-id="6f4c2-139">예를 들어, 전자 상거래 솔루션을 지원하는 Web API에서 고객이 다수의 주문을 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-139">For example, in a web API that supports an e-commerce solution, a customer may have placed many orders.</span></span> <span data-ttu-id="6f4c2-140">클라이언트 응용 프로그램에서 고객의 세부 정보를 검색하는 경우, 클라이언트 응용 프로그램이 주문을 찾을 수 있는 HTTP GET 요청을 보낼 수 있도록 하는 링크가 응답에 포함되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-140">When a client application retrieves the details for a customer, the response should include links that enable the client application to send HTTP GET requests that can retrieve these orders.</span></span> <span data-ttu-id="6f4c2-141">또한 HATEOAS 스타일 링크는 각 요청을 수행하도록 해당 URI와 함께 연결된 리소스가 함께 지원하는 다른 작업(POST, PUT, DELETE 등)을 설명해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-141">Additionally, HATEOAS-style links should describe the other operations (POST, PUT, DELETE, and so on) that each linked resource supports together with the corresponding URI to perform each request.</span></span> <span data-ttu-id="6f4c2-142">이 방법은 [API 디자인][api-design]에 보다 자세히 설명되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-142">This approach is described in more detail in [API Design][api-design].</span></span>

<span data-ttu-id="6f4c2-143">현재 HATEOAS 구현을 제어하는 표준은 없으며 다음 예제에서 가능한 접근 방식을 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-143">Currently there are no standards that govern the implementation of HATEOAS, but the following example illustrates one possible approach.</span></span> <span data-ttu-id="6f4c2-144">이 예제에서 고객에 대한 세부 정보를 찾는 HTTP GET 요청은 해당 고객의 주문을 참조하는 HATEOAS 링크를 포함하는 응답을 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-144">In this example, an HTTP GET request that finds the details for a customer returns a response that include HATEOAS links that reference the orders for that customer:</span></span>

```HTTP
GET http://adventure-works.com/customers/2 HTTP/1.1
Accept: text/json
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"CustomerID":2,"CustomerName":"Bert","Links":[
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"PUT",
    "types":["application/x-www-form-urlencoded"]},
    {"rel":"self",
    "href":"http://adventure-works.com/customers/2",
    "action":"DELETE",
    "types":[]},
    {"rel":"orders",
    "href":"http://adventure-works.com/customers/2/orders",
    "action":"GET",
    "types":["text/xml","application/json"]},
    {"rel":"orders",
    "href":"http://adventure-works.com/customers/2/orders",
    "action":"POST",
    "types":["application/x-www-form-urlencoded"]}
]}
```

<span data-ttu-id="6f4c2-145">이 예제에서 고객 데이터는 다음 코드 조각에 표시된 `Customer` 클래스로 나타납니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-145">In this example, the customer data is represented by the `Customer` class shown in the following code snippet.</span></span> <span data-ttu-id="6f4c2-146">HATEOAS 링크는 `Links` 컬렉션 속성에 보관됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-146">The HATEOAS links are held in the `Links` collection property:</span></span>

```csharp
public class Customer
{
    public int CustomerID { get; set; }
    public string CustomerName { get; set; }
    public List<Link> Links { get; set; }
    ...
}

public class Link
{
    public string Rel { get; set; }
    public string Href { get; set; }
    public string Action { get; set; }
    public string [] Types { get; set; }
}
```

<span data-ttu-id="6f4c2-147">HTTP GET 작업은 저장소에서 고객 데이터를 가져오고 `Customer` 개체를 구성한 후 `Links` 컬렉션을 채웁니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-147">The HTTP GET operation retrieves the customer data from storage and constructs a `Customer` object, and then populates the `Links` collection.</span></span> <span data-ttu-id="6f4c2-148">결과는 JSON 응답 메시지 형식으로 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-148">The result is formatted as a JSON response message.</span></span> <span data-ttu-id="6f4c2-149">각 링크는 다음 필드를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-149">Each link comprises the following fields:</span></span>

* <span data-ttu-id="6f4c2-150">반환되는 개체와 링크로 설명되는 개체 사이의 관계.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-150">The relationship between the object being returned and the object described by the link.</span></span> <span data-ttu-id="6f4c2-151">이 경우 "self"는 링크가 개체 스스로를 참조한다는 것을 나타내며(다수의 개체 지향 언어에서 `this` 포인터와 유사한) "orders"는 관련된 주문 정보를 포함하는 컬렉션의 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-151">In this case "self" indicates that the link is a reference back to the object itself (similar to a `this` pointer in many object-oriented languages), and "orders" is the name of a collection containing the related order information.</span></span>
* <span data-ttu-id="6f4c2-152">URI 형태의 링크로 설명되는 개체에 대한 하이퍼링크(`Href`).</span><span class="sxs-lookup"><span data-stu-id="6f4c2-152">The hyperlink (`Href`) for the object being described by the link in the form of a URI.</span></span>
* <span data-ttu-id="6f4c2-153">이 URI에 전송될 수 있는 HTTP 요청 유형(`Action`).</span><span class="sxs-lookup"><span data-stu-id="6f4c2-153">The type of HTTP request (`Action`) that can be sent to this URI.</span></span>
* <span data-ttu-id="6f4c2-154">HTTP 요청에 제공되어야 하는 데이터 형식 또는 요청 유형에 따라 응답으로 반환될 수 있는 데이터의 형식(`Types`).</span><span class="sxs-lookup"><span data-stu-id="6f4c2-154">The format of any data (`Types`) that should be provided in the HTTP request or that can be returned in the response, depending on the type of the request.</span></span>

<span data-ttu-id="6f4c2-155">HTTP 응답 예제에 있는 HATEOAS 링크는 클라이언트 응용 프로그램이 다음 작업을 수행할 수 있다는 것을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-155">The HATEOAS links shown in the example HTTP response indicate that a client application can perform the following operations:</span></span>

* <span data-ttu-id="6f4c2-156">URI `http://adventure-works.com/customers/2`에 대한 HTTP GET 요청: 고객 세부 정보를 (다시) 가져오기 위한 요청입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-156">An HTTP GET request to the URI `http://adventure-works.com/customers/2` to fetch the details of the customer (again).</span></span> <span data-ttu-id="6f4c2-157">데이터는 XML 또는 JSON으로 반환될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-157">The data can be returned as XML or JSON.</span></span>
* <span data-ttu-id="6f4c2-158">URI `http://adventure-works.com/customers/2`에 대한 HTTP PUT 요청: 고객 세부 정보를 수정하기 위한 요청입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-158">An HTTP PUT request to the URI `http://adventure-works.com/customers/2` to modify the details of the customer.</span></span> <span data-ttu-id="6f4c2-159">요청 메시지에 x-www-form-urlencoded 형식의 새로운 데이터가 제공되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-159">The new data must be provided in the request message in x-www-form-urlencoded format.</span></span>
* <span data-ttu-id="6f4c2-160">URI `http://adventure-works.com/customers/2`에 대한 HTTP DELETE 요청: 고객을 삭제하기 위한 요청입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-160">An HTTP DELETE request to the URI `http://adventure-works.com/customers/2` to delete the customer.</span></span> <span data-ttu-id="6f4c2-161">이 요청은 추가적인 정보를 요구하지 않거나 응답 메시지 본문에 데이터를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-161">The request does not expect any additional information or return data in the response message body.</span></span>
* <span data-ttu-id="6f4c2-162">URI `http://adventure-works.com/customers/2/orders`에 대한 HTTP GET 요청: 고객에 대한 모든 주문을 찾기 위한 요청입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-162">An HTTP GET request to the URI `http://adventure-works.com/customers/2/orders` to find all the orders for the customer.</span></span> <span data-ttu-id="6f4c2-163">데이터는 XML 또는 JSON으로 반환될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-163">The data can be returned as XML or JSON.</span></span>
* <span data-ttu-id="6f4c2-164">URI `http://adventure-works.com/customers/2/orders`에 대한 HTTP PUT 요청: 이 고객에 대한 새로운 주문을 만들기 위한 요청입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-164">An HTTP PUT request to the URI `http://adventure-works.com/customers/2/orders` to create a new order for this customer.</span></span> <span data-ttu-id="6f4c2-165">요청 메시지에 x-www-form-urlencoded 형식의 데이터가 제공되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-165">The data must be provided in the request message in x-www-form-urlencoded format.</span></span>

## <a name="handling-exceptions"></a><span data-ttu-id="6f4c2-166">예외 처리</span><span class="sxs-lookup"><span data-stu-id="6f4c2-166">Handling exceptions</span></span>

<span data-ttu-id="6f4c2-167">작업이 확인할 수 없는 예외를 throw하는 경우 다음 사항을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-167">Consider the following points if an operation throws an uncaught exception.</span></span>

### <a name="capture-exceptions-and-return-a-meaningful-response-to-clients"></a><span data-ttu-id="6f4c2-168">예외 사항을 확인하고 클라이언트에 의미 있는 응답을 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-168">Capture exceptions and return a meaningful response to clients</span></span>

<span data-ttu-id="6f4c2-169">HTTP 작업을 구현하는 코드는 catch할 수 없는 예외가 프레임워크에 퍼지도록 하기 보다는 포괄적인 예외 처리를 제공해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-169">The code that implements an HTTP operation should provide comprehensive exception handling rather than letting uncaught exceptions propagate to the framework.</span></span> <span data-ttu-id="6f4c2-170">예외로 인해 작업을 성공적으로 완료하는 것이 불가능한 경우에는 예외 사항이 응답 메시지에 전달될 수 있습니다. 하지만 예외를 유발한 오류에 대하여 의미 있는 설명을 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-170">If an exception makes it impossible to complete the operation successfully, the exception can be passed back in the response message, but it should include a meaningful description of the error that caused the exception.</span></span> <span data-ttu-id="6f4c2-171">예외는 모든 상황에 대해 단순히 상태 코드 500만 반환하기 보다는 적절한 HTTP 상태 코드도 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-171">The exception should also include the appropriate HTTP status code rather than simply returning status code 500 for every situation.</span></span> <span data-ttu-id="6f4c2-172">예를 들어, 사용자 요청으로 인해 제약 조건에 위배되는 데이터베이스 업데이트(예: 주문량이 탁월한 고객을 삭제하려는 시도)가 발생한 경우 상태 코드 409(충돌)와 충돌 이유를 나타내는 메시지 본문을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-172">For example, if a user request causes a database update that violates a constraint (such as attempting to delete a customer that has outstanding orders), you should return status code 409 (Conflict) and a message body indicating the reason for the conflict.</span></span> <span data-ttu-id="6f4c2-173">다른 조건이 달성할 수 없는 요청을 렌더링하는 경우에는 상태 코드 400(잘못된 요청)을 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-173">If some other condition renders the request unachievable, you can return status code 400 (Bad Request).</span></span> <span data-ttu-id="6f4c2-174">HTTP 상태 코드의 전체 목록은 W3C 웹 사이트의 [Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)(상태 코드 정의) 페이지에서 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-174">You can find a full list of HTTP status codes on the [Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) page on the W3C website.</span></span>

<span data-ttu-id="6f4c2-175">코드 예제에서는 다른 조건을 트래핑하고 적절한 응답을 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-175">The code example traps different conditions and returns an appropriate response.</span></span>

```csharp
[HttpDelete]
[Route("customers/{id:int}")]
public IHttpActionResult DeleteCustomer(int id)
{
    try
    {
        // Find the customer to be deleted in the repository
        var customerToDelete = repository.GetCustomer(id);

        // If there is no such customer, return an error response
        // with status code 404 (Not Found)
        if (customerToDelete == null)
        {
                return NotFound();
        }

        // Remove the customer from the repository
        // The DeleteCustomer method returns true if the customer
        // was successfully deleted
        if (repository.DeleteCustomer(id))
        {
            // Return a response message with status code 204 (No Content)
            // To indicate that the operation was successful
            return StatusCode(HttpStatusCode.NoContent);
        }
        else
        {
            // Otherwise return a 400 (Bad Request) error response
            return BadRequest(Strings.CustomerNotDeleted);
        }
    }
    catch
    {
        // If an uncaught exception occurs, return an error response
        // with status code 500 (Internal Server Error)
        return InternalServerError();
    }
}
```

> [!TIP]
> <span data-ttu-id="6f4c2-176">API에 침투하려고 시도하는 공격자에게 도움이 될 수 있는 정보를 포함하지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-176">Do not include information that could be useful to an attacker attempting to penetrate your API.</span></span>
  
<span data-ttu-id="6f4c2-177">많은 웹 서버에서 Web API에 도달하기 전에 오류 조건을 자체 트래핑합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-177">Many web servers trap error conditions themselves before they reach the web API.</span></span> <span data-ttu-id="6f4c2-178">예를 들어, 웹 사이트에 대한 인증을 구성했는데 사용자가 올바른 인증 정보를 제공하지 못하면 웹 서버는 상태 코드 401(권한 없음)로 응답해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-178">For example, if you configure authentication for a web site and the user fails to provide the correct authentication information, the web server should respond with status code 401 (Unauthorized).</span></span> <span data-ttu-id="6f4c2-179">클라이언트가 인증된 후에는 이 클라이언트가 요청한 리소스에 액세스할 수 있는지를 검증하기 위하여 코드로 자체 확인을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-179">Once a client has been authenticated, your code can perform its own checks to verify that the client should be able access the requested resource.</span></span> <span data-ttu-id="6f4c2-180">권한 부여가 실패하면 상태 코드 403(사용 권한 없음)을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-180">If this authorization fails, you should return status code 403 (Forbidden).</span></span>
 
### <a name="handle-exceptions-consistently-and-log-information-about-errors"></a><span data-ttu-id="6f4c2-181">일관되게 예외를 처리하고 오류에 대한 정보를 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-181">Handle exceptions consistently and log information about errors</span></span>

<span data-ttu-id="6f4c2-182">일관된 방식으로 예외를 처리하려면 Web API에 대해 전체적인 오류 처리 전략을 구현하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-182">To handle exceptions in a consistent manner, consider implementing a global error handling strategy across the entire web API.</span></span> <span data-ttu-id="6f4c2-183">각 예외의 전체적인 세부 사항을 모두 파악하도록 오류 로깅을 통합해야 합니다. 이 오류 로그는 웹을 통해 클라이언트에 액세스할 수 있도록 만들지 않는 한 자세한 정보를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-183">You should also incorporate error logging which captures the full details of each exception; this error log can contain detailed information as long as it is not made accessible over the web to clients.</span></span> 

### <a name="distinguish-between-client-side-errors-and-server-side-errors"></a><span data-ttu-id="6f4c2-184">클라이언트쪽 오류와 서버쪽 오류를 구별합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-184">Distinguish between client-side errors and server-side errors</span></span>

<span data-ttu-id="6f4c2-185">HTTP 프로토콜은 클라이언트 응용 프로그램으로 인해 발생하는 오류(HTTP 4xx 상태 코드)와 서버의 문제 때문에 발생한 오류(HTTP 5xx 상태 코드)를 구분합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-185">The HTTP protocol distinguishes between errors that occur due to the client application (the HTTP 4xx status codes), and errors that are caused by a mishap on the server (the HTTP 5xx status codes).</span></span> <span data-ttu-id="6f4c2-186">모든 오류 응답 메시지에 대하여 이 규칙을 반드시 따라야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-186">Make sure that you respect this convention in any error response messages.</span></span>

## <a name="optimizing-client-side-data-access"></a><span data-ttu-id="6f4c2-187">클라이언트 쪽 데이터 액세스 최적화</span><span class="sxs-lookup"><span data-stu-id="6f4c2-187">Optimizing client-side data access</span></span>
<span data-ttu-id="6f4c2-188">웹 서버와 클라이언트 응용 프로그램을 포함하는 분산된 환경에서 주요 관심사 중 하나는 네트워크입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-188">In a distributed environment such as that involving a web server and client applications, one of the primary sources of concern is the network.</span></span> <span data-ttu-id="6f4c2-189">이것은 상당한 병목 지점이 될 수 있으며, 클라이언트 응용 프로그램이 빈번하게 요청을 보내거나 데이터를 받는 경우에는 특히 그렇습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-189">This can act as a considerable bottleneck, especially if a client application is frequently sending requests or receiving data.</span></span> <span data-ttu-id="6f4c2-190">따라서 네트워크를 통해 이동하는 트래픽 양을 최소화시킨다는 목표를 가져야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-190">Therefore you should aim to minimize the amount of traffic that flows across the network.</span></span> <span data-ttu-id="6f4c2-191">데이터를 가져오고 유지하기 위하여 코드를 구현하는 경우 다음 사항을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-191">Consider the following points when you implement the code to retrieve and maintain data:</span></span>

### <a name="support-client-side-caching"></a><span data-ttu-id="6f4c2-192">클라이언트쪽 캐싱을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-192">Support client-side caching</span></span>

<span data-ttu-id="6f4c2-193">HTTP 1.1 프로토콜은 클라이언트 및 Cache-Control 헤더를 사용하여 요청을 라우팅하는 중간 서버에서 캐싱을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-193">The HTTP 1.1 protocol supports caching in clients and intermediate servers through which a request is routed by the use of the Cache-Control header.</span></span> <span data-ttu-id="6f4c2-194">클라이언트 응용 프로그램이 Web API에 HTTP GET 요청을 보내면 응답은 본문에 포함된 데이터가 클라이언트 또는 중간 서버(요청이 라우팅되는)에 의해 안전하게 캐싱될 수 있는지와 얼마 만에 만료되는지, 오래된 것으로 간주되는 지를 나타내는 Cache-Control 헤더를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-194">When a client application sends an HTTP GET request to the web API, the response can include a Cache-Control header that indicates whether the data in the body of the response can be safely cached by the client or an intermediate server through which the request has been routed, and for how long before it should expire and be considered out-of-date.</span></span> <span data-ttu-id="6f4c2-195">다음 예제는 HTTP GET 요청 및 Cache-Control 헤더를 포함하는 해당 응답입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-195">The following example shows an HTTP GET request and the corresponding response that includes a Cache-Control header:</span></span>

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
```

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

<span data-ttu-id="6f4c2-196">이 예제에서 Cache-Control 헤더는 반환된 데이터가 600초 후에 만료되어야 하고, 단일 클라이언트에만 적합하며, 다른 클라이언트가 사용하는 공유 캐시에 저장되지 말아야 한다고 명시합니다(*private*).</span><span class="sxs-lookup"><span data-stu-id="6f4c2-196">In this example, the Cache-Control header specifies that the data returned should be expired after 600 seconds, and is only suitable for a single client and must not be stored in a shared cache used by other clients (it is *private*).</span></span> <span data-ttu-id="6f4c2-197">Cache-Control 헤더는 공유 캐시에 데이터를 저장할 수 있는 경우 *private*이 아닌 *public*을 지정합니다. 클라이언트에서 데이터를 캐싱하지 **말아야** 하는 경우 *no-store*로 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-197">The Cache-Control header could specify *public* rather than *private* in which case the data can be stored in a shared cache, or it could specify *no-store* in which case the data must **not** be cached by the client.</span></span> <span data-ttu-id="6f4c2-198">다음 코드 예제는 응답 메시지에서 Cache-Control 헤더를 구성하는 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-198">The following code example shows how to construct a Cache-Control header in a response message:</span></span>

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...
        // Create a Cache-Control header for the response
        var cacheControlHeader = new CacheControlHeaderValue();
        cacheControlHeader.Private = true;
        cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
        ...

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            CacheControlHeader = cacheControlHeader
        };
        return response;
    }
    ...
}
```

<span data-ttu-id="6f4c2-199">이 코드는 이름이 `OkResultWithCaching`인 사용자 지정 `IHttpActionResult` 클래스를 활용합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-199">This code makes use of a custom `IHttpActionResult` class named `OkResultWithCaching`.</span></span> <span data-ttu-id="6f4c2-200">이 클래스는 컨트롤러가 캐시 헤더 콘텐츠를 설정할 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-200">This class enables the controller to set the cache header contents:</span></span>

```csharp
public class OkResultWithCaching<T> : OkNegotiatedContentResult<T>
{
    public OkResultWithCaching(T content, ApiController controller)
        : base(content, controller) { }

    public OkResultWithCaching(T content, IContentNegotiator contentNegotiator, HttpRequestMessage request, IEnumerable<MediaTypeFormatter> formatters)
        : base(content, contentNegotiator, request, formatters) { }

    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }

    public override async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response;
        try
        {
            response = await base.ExecuteAsync(cancellationToken);
            response.Headers.CacheControl = this.CacheControlHeader;
            response.Headers.ETag = ETag;
        }
        catch (OperationCanceledException)
        {
            response = new HttpResponseMessage(HttpStatusCode.Conflict) {ReasonPhrase = "Operation was cancelled"};
        }
        return response;
    }
}
```

> [!NOTE]
> <span data-ttu-id="6f4c2-201">HTTP 프로토콜은 Cache-Control 헤더에 대해 *no-cache* 지시문을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-201">The HTTP protocol also defines the *no-cache* directive for the Cache-Control header.</span></span> <span data-ttu-id="6f4c2-202">다소 혼란스럽지만, 이 지시문은 "캐시하지 말라"는 의미가 아니고 "캐시된 정보를 반환하기 전에 서버에서 재검증하라"는 의미입니다. 데이터를 캐싱할 수 있지만 최신 상태인지 확인하기 위하여 사용될 때마다 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-202">Rather confusingly, this directive does not mean "do not cache" but rather "revalidate the cached information with the server before returning it"; the data can still be cached, but it is checked each time it is used to ensure that it is still current.</span></span>
>
>

<span data-ttu-id="6f4c2-203">캐시 관리는 클라이언트 응용 프로그램 또는 중간 서버의 책임입니다. 하지만 제대로 구현되면 대역폭을 절약할 수 있고 이미 최근에 검색한 데이터를 가져올 필요가 없기 때문에 성능을 향상시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-203">Cache management is the responsibility of the client application or intermediate server, but if properly implemented it can save bandwidth and improve performance by removing the need to fetch data that has already been recently retrieved.</span></span>

<span data-ttu-id="6f4c2-204">Cache-Control 헤더의 *max-age* 값은 가이드일 뿐이며 지정된 시간 동안 해당 데이터가 변경되지 않는다고 보장하는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-204">The *max-age* value in the Cache-Control header is only a guide and not a guarantee that the corresponding data won't change during the specified time.</span></span> <span data-ttu-id="6f4c2-205">Web API는 예상되는 데이터 변동성에 따라서 max-age를 적당한 값으로 설정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-205">The web API should set the max-age to a suitable value depending on the expected volatility of the data.</span></span> <span data-ttu-id="6f4c2-206">이 기간이 만료되면 클라이언트는 해당 개체를 캐시에서 삭제해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-206">When this period expires, the client should discard the object from the cache.</span></span>

> [!NOTE]
> <span data-ttu-id="6f4c2-207">설명한 대로, 대부분의 최신 웹 브라우저는 적절한 cache-control 헤더를 요청에 추가하고 결과의 헤더를 검토하는 방식으로 클라이언트쪽 캐싱을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-207">Most modern web browsers support client-side caching by adding the appropriate cache-control headers to requests and examining the headers of the results, as described.</span></span> <span data-ttu-id="6f4c2-208">하지만 일부 오래된 브라우저는 쿼리 문자열을 포함하는 URL로부터 반환되는 값을 캐시하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-208">However, some older browsers will not cache the values returned from a URL that includes a query string.</span></span> <span data-ttu-id="6f4c2-209">보통 이것은 여기에서 논의한 프로토콜을 기반으로 자체적인 캐시 관리 전략을 구현하는 사용자 지정 클라이언트 응용 프로그램에서는 문제가 되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-209">This is not usually an issue for custom client applications which implement their own cache management strategy based on the protocol discussed here.</span></span>
>
> <span data-ttu-id="6f4c2-210">일부 오래된 프록시는 동일한 행태를 보이며 URL을 기반으로 하는 요청을 쿼리 문자열로 캐시하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-210">Some older proxies exhibit the same behavior and might not cache requests based on URLs with query strings.</span></span> <span data-ttu-id="6f4c2-211">이것은 그러한 프록시를 통해 웹 서버에 연결하는 사용자 지정 클라이언트 응용 프로그램에서 문제가 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-211">This could be an issue for custom client applications that connect to a web server through such a proxy.</span></span>
>

### <a name="provide-etags-to-optimize-query-processing"></a><span data-ttu-id="6f4c2-212">쿼리 처리를 최적화하기 위해 ETag를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-212">Provide ETags to optimize query processing</span></span>

<span data-ttu-id="6f4c2-213">클라이언트 응용 프로그램이 개체를 검색하는 경우, 응답 메시지에 *ETag*(엔터티 태그)를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-213">When a client application retrieves an object, the response message can also include an *ETag* (Entity Tag).</span></span> <span data-ttu-id="6f4c2-214">ETag는 리소스의 버전을 나타내는 불투명 문자열입니다. 리소스가 변경될 때마다 Etag도 수정됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-214">An ETag is an opaque string that indicates the version of a resource; each time a resource changes the Etag is also modified.</span></span> <span data-ttu-id="6f4c2-215">ETag는 클라이언트 응용 프로그램에 의해 데이터의 일부로 캐시되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-215">This ETag should be cached as part of the data by the client application.</span></span> <span data-ttu-id="6f4c2-216">다음 코드 예제는 HTTP GET 요청에 대한 응답의 일부로 ETag를 추가하는 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-216">The following code example shows how to add an ETag as part of the response to an HTTP GET request.</span></span> <span data-ttu-id="6f4c2-217">이 코드는 개체를 식별하는 숫자 값을 생성하기 위해 개체의 `GetHashCode` 메서드를 사용합니다. (필요한 경우 이 메서드를 무효화하고 MD5와 같은 알고리즘을 사용하여 자체 해시를 생성할 수 있습니다.)</span><span class="sxs-lookup"><span data-stu-id="6f4c2-217">This code uses the `GetHashCode` method of an object to generate a numeric value that identifies the object (you can override this method if necessary and generate your own hash using an algorithm such as MD5) :</span></span>

```csharp
public class OrdersController : ApiController
{
    ...
    public IHttpActionResult FindOrderByID(int id)
    {
        // Find the matching order
        Order order = ...;
        ...

        var hashedOrder = order.GetHashCode();
        string hashedOrderEtag = $"\"{hashedOrder}\"";
        var eTag = new EntityTagHeaderValue(hashedOrderEtag);

        // Return a response message containing the order and the cache control header
        OkResultWithCaching<Order> response = new OkResultWithCaching<Order>(order, this)
        {
            ...,
            ETag = eTag
        };
        return response;
    }
    ...
}
```

<span data-ttu-id="6f4c2-218">Web API에서 게시한 응답 메시지는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-218">The response message posted by the web API looks like this:</span></span>

```HTTP
HTTP/1.1 200 OK
...
Cache-Control: max-age=600, private
Content-Type: text/json; charset=utf-8
ETag: "2147483648"
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

> [!TIP]
> <span data-ttu-id="6f4c2-219">보안을 위해 민감한 데이터 또는 인증된(HTTPS) 연결을 통해 반환되는 데이터가 캐시되도록 허용하지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-219">For security reasons, do not allow sensitive data or data returned over an authenticated (HTTPS) connection to be cached.</span></span>
>
>

<span data-ttu-id="6f4c2-220">클라이언트 응용 프로그램은 언제든 동일한 리소스를 검색하기 위하여 후속으로 GET 요청을 발행할 수 있습니다. 만약 리소스가 변경되면(ETag가 다르며) 캐시된 버전은 삭제되고 새 버전이 캐시에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-220">A client application can issue a subsequent GET request to retrieve the same resource at any time, and if the resource has changed (it has a different ETag) the cached version should be discarded and the new version added to the cache.</span></span> <span data-ttu-id="6f4c2-221">리소스가 크고 클라이언트로 전송하기 위해 상당한 양의 대역폭이 필요한 경우, 동일한 데이터를 가져오기 위한 반복 요청은 비능률적일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-221">If a resource is large and requires a significant amount of bandwidth to transmit back to the client, repeated requests to fetch the same data can become inefficient.</span></span> <span data-ttu-id="6f4c2-222">이 문제를 해결하기 위해, HTTP 프로토콜은 Web API에서 지원해야 하는 GET 요청을 최적화하기 위해 다음 프로세스를 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-222">To combat this, the HTTP protocol defines the following process for optimizing GET requests that you should support in a web API:</span></span>

* <span data-ttu-id="6f4c2-223">클라이언트는 If-None-Match HTTP 헤더에서 참조하는 리소스의 현재 캐시 버전에 대한 ETag를 포함하는 GET 요청을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-223">The client constructs a GET request containing the ETag for the currently cached version of the resource referenced in an If-None-Match HTTP header:</span></span>

    ```HTTP
    GET http://adventure-works.com/orders/2 HTTP/1.1
    If-None-Match: "2147483648"
    ```
* <span data-ttu-id="6f4c2-224">Web API의 GET 작업은 요청한 데이터(위 예제의 order 2)에 대한 현재 ETag를 확보하고 If-None-Match 헤더의 값과 비교합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-224">The GET operation in the web API obtains the current ETag for the requested data (order 2 in the above example), and compares it to the value in the If-None-Match header.</span></span>
* <span data-ttu-id="6f4c2-225">요청한 데이터의 현재 ETag가 요청에서 제공한 ETag와 부합하는 경우, 리소스는 변경되지 않은 것이며 Web API는 빈 메시지 본문과 상태 코드 304(수정되지 않음)로 HTTP 응답을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-225">If the current ETag for the requested data matches the ETag provided by the request, the resource has not changed and the web API should return an HTTP response with an empty message body and a status code of 304 (Not Modified).</span></span>
* <span data-ttu-id="6f4c2-226">요청한 데이터의 현재 ETag가 요청에서 제공한 ETag와 부합하지 않으면, 데이터는 변경된 것이며 Web API는 메시지 본문에 새로운 데이터를 넣고 상태 코드 200(OK)으로 HTTP 응답을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-226">If the current ETag for the requested data does not match the ETag provided by the request, then the data has changed and the web API should return an HTTP response with the new data in the message body and a status code of 200 (OK).</span></span>
* <span data-ttu-id="6f4c2-227">요청한 데이터가 더 이상 존재하지 않으면 Web API는 상태 코드 404(찾을 수 없음)로 HTTP 응답을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-227">If the requested data no longer exists then the web API should return an HTTP response with the status code of 404 (Not Found).</span></span>
* <span data-ttu-id="6f4c2-228">클라이언트는 캐시를 유지하기 위해 상태 코드를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-228">The client uses the status code to maintain the cache.</span></span> <span data-ttu-id="6f4c2-229">데이터가 변경되지 않은 경우(상태 코드 304)에 개체는 캐시된 상태로 남고 클라이언트 응용 프로그램은 이 버전의 개체를 계속 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-229">If the data has not changed (status code 304) then the object can remain cached and the client application should continue to use this version of the object.</span></span> <span data-ttu-id="6f4c2-230">데이터가 변경된 경우(상태 코드 200)에는 캐시된 개체를 삭제하고 새 개체를 삽입해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-230">If the data has changed (status code 200) then the cached object should be discarded and the new one inserted.</span></span> <span data-ttu-id="6f4c2-231">데이터를 더 이상 사용할 수 없는 경우(상태 코드 404)에는 개체를 캐시에서 제거해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-231">If the data is no longer available (status code 404) then the object should be removed from the cache.</span></span>

> [!NOTE]
> <span data-ttu-id="6f4c2-232">응답 헤더에 Cache-Control 헤더 no-store가 포함되면 HTTP 상태 코드에 상관없이 캐시에서 개체를 항상 제거해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-232">If the response header contains the Cache-Control header no-store then the object should always be removed from the cache regardless of the HTTP status code.</span></span>
>

<span data-ttu-id="6f4c2-233">아래 코드는 If-None-Match 헤더를 지원하기 위해 확장된 `FindOrderByID` 메서드입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-233">The code below shows the `FindOrderByID` method extended to support the If-None-Match header.</span></span> <span data-ttu-id="6f4c2-234">If-None-Match 헤더가 생략되면 지정된 order를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-234">Notice that if the If-None-Match header is omitted, the specified order is always retrieved:</span></span>

```csharp
public class OrdersController : ApiController
{
    [Route("api/orders/{id:int:min(0)}")]
    [HttpGet]
    public IHttpActionResult FindOrderByID(int id)
    {
        try
        {
            // Find the matching order
            Order order = ...;

            // If there is no such order then return NotFound
            if (order == null)
            {
                return NotFound();
            }

            // Generate the ETag for the order
            var hashedOrder = order.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Create the Cache-Control and ETag headers for the response
            IHttpActionResult response;
            var cacheControlHeader = new CacheControlHeaderValue();
            cacheControlHeader.Public = true;
            cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);
            var eTag = new EntityTagHeaderValue(hashedOrderEtag);

            // Retrieve the If-None-Match header from the request (if it exists)
            var nonMatchEtags = Request.Headers.IfNoneMatch;

            // If there is an ETag in the If-None-Match header and
            // this ETag matches that of the order just retrieved,
            // then create a Not Modified response message
            if (nonMatchEtags.Count > 0 &&
                String.CompareOrdinal(nonMatchEtags.First().Tag, hashedOrderEtag) == 0)
            {
                response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NotModified,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }
            // Otherwise create a response message that contains the order details
            else
            {
                response = new OkResultWithCaching<Order>(order, this)
                {
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag
                };
            }

            return response;
        }
        catch
        {
            return InternalServerError();
        }
    }
...
}
```

<span data-ttu-id="6f4c2-235">이 예제는 이름이 `EmptyResultWithCaching`인 사용자 지정 `IHttpActionResult` 클래스를 통합합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-235">This example incorporates an additional custom `IHttpActionResult` class named `EmptyResultWithCaching`.</span></span> <span data-ttu-id="6f4c2-236">이 클래스는 응답 본문을 포함하지 않는 `HttpResponseMessage` 개체의 래퍼 역할을 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-236">This class simply acts as a wrapper around an `HttpResponseMessage` object that does not contain a response body:</span></span>

```csharp
public class EmptyResultWithCaching : IHttpActionResult
{
    public CacheControlHeaderValue CacheControlHeader { get; set; }
    public EntityTagHeaderValue ETag { get; set; }
    public HttpStatusCode StatusCode { get; set; }
    public Uri Location { get; set; }

    public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
    {
        HttpResponseMessage response = new HttpResponseMessage(StatusCode);
        response.Headers.CacheControl = this.CacheControlHeader;
        response.Headers.ETag = this.ETag;
        response.Headers.Location = this.Location;
        return response;
    }
}
```

> [!TIP]
> <span data-ttu-id="6f4c2-237">이 예제에서 데이터에 대한 ETag는 기본 데이터 원본에서 가져온 데이터를 해시하여 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-237">In this example, the ETag for the data is generated by hashing the data retrieved from the underlying data source.</span></span> <span data-ttu-id="6f4c2-238">ETag를 다른 방법으로 계산할 수 있으면, 프로세스를 더 많이 최적화시킬 수 있고 데이터가 변경되면 데이터 원본으로부터 데이터만 가져오면 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-238">If the ETag can be computed in some other way, then the process can be optimized further and the data only needs to be fetched from the data source if it has changed.</span></span>  <span data-ttu-id="6f4c2-239">이 방법은 데이터가 크거나 데이터 원본에 액세스할 때 상당한 대기 시간이 발생하는 경우(예: 데이터 원본이 원격 데이터베이스인 경우)에 특히 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-239">This approach is especially useful if the data is large or accessing the data source can result in significant latency (for example, if the data source is a remote database).</span></span>
>

### <a name="use-etags-to-support-optimistic-concurrency"></a><span data-ttu-id="6f4c2-240">낙관적 동시성을 지원하기 위해 ETag를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-240">Use ETags to Support Optimistic Concurrency</span></span>

<span data-ttu-id="6f4c2-241">이전에 캐시한 데이터를 업데이트 할 수 있도록, HTTP 프로토콜은 낙관적 동시성 전략을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-241">To enable updates over previously cached data, the HTTP protocol supports an optimistic concurrency strategy.</span></span> <span data-ttu-id="6f4c2-242">리소스를 가져오고 캐시한 후에 클라이언트 응용 프로그램이 리소스를 변경하거나 제거하기 위해 PUT 또는 DELETE 요청을 계속해서 보내면 ETag를 참조하는 If-Match 헤더를 포함시켜야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-242">If, after fetching and caching a resource, the client application subsequently sends a PUT or DELETE request to change or remove the resource, it should include in If-Match header that references the ETag.</span></span> <span data-ttu-id="6f4c2-243">그러면 Web API는 이 정보를 사용하여 리소스를 가져온 후에 다른 사용자가 리소스를 변경했는지 여부를 판단하고, 클라이언트 응용 프로그램에 다음과 같이 적절한 응답을 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-243">The web API can then use this information to determine whether the resource has already been changed by another user since it was retrieved and send an appropriate response back to the client application as follows:</span></span>

* <span data-ttu-id="6f4c2-244">클라이언트는 리소스에 대해 새로운 세부 정보를 포함하는 PUT 요청과 If-Match HTTP 헤더에 참조되는 리소스의 현재 캐시 버전에 대한 ETag를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-244">The client constructs a PUT request containing the new details for the resource and the ETag for the currently cached version of the resource referenced in an If-Match HTTP header.</span></span> <span data-ttu-id="6f4c2-245">다음 예제는 order를 업데이트하는 PUT 요청입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-245">The following example shows a PUT request that updates an order:</span></span>

    ```HTTP
    PUT http://adventure-works.com/orders/1 HTTP/1.1
    If-Match: "2282343857"
    Content-Type: application/x-www-form-urlencoded
    Content-Length: ...
    productID=3&quantity=5&orderValue=250
    ```
* <span data-ttu-id="6f4c2-246">Web API의 PUT 작업은 요청된 데이터(위 예제의 order 1)에 대한 현재 ETag를 확보하여 If-Match 헤더에 포함된 값과 비교합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-246">The PUT operation in the web API obtains the current ETag for the requested data (order 1 in the above example), and compares it to the value in the If-Match header.</span></span>
* <span data-ttu-id="6f4c2-247">요청된 데이터의 현재 ETag가 요청에 의해 제공된 ETag와 부합하는 경우, 리소스는 변경되지 않았고 Web API는 업데이트를 수행해야 하며, 성공적으로 수행한 후에는 HTTP 상태 코드 204(내용 없음)를 포함하는 메시지를 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-247">If the current ETag for the requested data matches the ETag provided by the request, the resource has not changed and the web API should perform the update, returning a message with HTTP status code 204 (No Content) if it is successful.</span></span> <span data-ttu-id="6f4c2-248">응답은 업데이트된 리소스 버전에 대한 Cache-Control 및 ETag 헤더를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-248">The response can include Cache-Control and ETag headers for the updated version of the resource.</span></span> <span data-ttu-id="6f4c2-249">응답은 새롭게 업데이트된 리소스의 URI를 참조하는 Location 헤더를 항상 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-249">The response should always include the Location header that references the URI of the newly updated resource.</span></span>
* <span data-ttu-id="6f4c2-250">요청한 데이터의 현재 ETag가 요청에서 제공한 ETag와 부합하지 않으면, 데이터를 가져온 후 다른 사용자가 데이터를 변경한 것이며 Web API는 빈 메시지 본문과 상태 코드 412(전제 조건 실패)로 HTTP 응답을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-250">If the current ETag for the requested data does not match the ETag provided by the request, then the data has been changed by another user since it was fetched and the web API should return an HTTP response with an empty message body and a status code of 412 (Precondition Failed).</span></span>
* <span data-ttu-id="6f4c2-251">업데이트할 데이터가 더 이상 존재하지 않으면 Web API는 상태 코드 404(찾을 수 없음)와 함께 HTTP 응답을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-251">If the resource to be updated no longer exists then the web API should return an HTTP response with the status code of 404 (Not Found).</span></span>
* <span data-ttu-id="6f4c2-252">클라이언트는 캐시를 유지하기 위해 상태 코드와 응답 헤더를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-252">The client uses the status code and response headers to maintain the cache.</span></span> <span data-ttu-id="6f4c2-253">데이터가 업데이트된(상태 코드 204) 후에는 개체가 캐시된 상태로 남을 수 있지만(Cache-Control 헤더가 no-store를 명시하지 않는 한) ETag는 업데이트되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-253">If the data has been updated (status code 204) then the object can remain cached (as long as the Cache-Control header does not specify no-store) but the ETag should be updated.</span></span> <span data-ttu-id="6f4c2-254">데이터가 변경되거나(상태 코드 412) 찾을 수 없는(상태 코드 404) 다른 사용자에 의해 변경된 경우, 캐시된 개체는 삭제되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-254">If the data was changed by another user changed (status code 412) or not found (status code 404) then the cached object should be discarded.</span></span>

<span data-ttu-id="6f4c2-255">다음 코드 예제는 Orders 컨트롤러에 대한 PUT 작업의 구현입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-255">The next code example shows an implementation of the PUT operation for the Orders controller:</span></span>

```csharp
public class OrdersController : ApiController
{
    [HttpPut]
    [Route("api/orders/{id:int}")]
    public IHttpActionResult UpdateExistingOrder(int id, DTOOrder order)
    {
        try
        {
            var baseUri = Constants.GetUriFromConfig();
            var orderToUpdate = this.ordersRepository.GetOrder(id);
            if (orderToUpdate == null)
            {
                return NotFound();
            }

            var hashedOrder = orderToUpdate.GetHashCode();
            string hashedOrderEtag = $"\"{hashedOrder}\"";

            // Retrieve the If-Match header from the request (if it exists)
            var matchEtags = Request.Headers.IfMatch;

            // If there is an Etag in the If-Match header and
            // this etag matches that of the order just retrieved,
            // or if there is no etag, then update the Order
            if (((matchEtags.Count > 0 &&
                String.CompareOrdinal(matchEtags.First().Tag, hashedOrderEtag) == 0)) ||
                matchEtags.Count == 0)
            {
                // Modify the order
                orderToUpdate.OrderValue = order.OrderValue;
                orderToUpdate.ProductID = order.ProductID;
                orderToUpdate.Quantity = order.Quantity;

                // Save the order back to the data store
                // ...

                // Create the No Content response with Cache-Control, ETag, and Location headers
                var cacheControlHeader = new CacheControlHeaderValue();
                cacheControlHeader.Private = true;
                cacheControlHeader.MaxAge = new TimeSpan(0, 10, 0);

                hashedOrder = order.GetHashCode();
                hashedOrderEtag = $"\"{hashedOrder}\"";
                var eTag = new EntityTagHeaderValue(hashedOrderEtag);

                var location = new Uri($"{baseUri}/{Constants.ORDERS}/{id}");
                var response = new EmptyResultWithCaching()
                {
                    StatusCode = HttpStatusCode.NoContent,
                    CacheControlHeader = cacheControlHeader,
                    ETag = eTag,
                    Location = location
                };

                return response;
            }

            // Otherwise return a Precondition Failed response
            return StatusCode(HttpStatusCode.PreconditionFailed);
        }
        catch
        {
            return InternalServerError();
        }
    }
    ...
}
```

> [!TIP]
> <span data-ttu-id="6f4c2-256">If-Match 헤더 사용이 완전히 선택적이며 생략된 경우에, Web API는 지정된 order를 항상 업데이트하려고 시도할 것이고 다른 사용자에 의해 수행된 업데이트를 무작정 덮어쓸 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-256">Use of the If-Match header is entirely optional, and if it is omitted the web API will always attempt to update the specified order, possibly blindly overwriting an update made by another user.</span></span> <span data-ttu-id="6f4c2-257">업데이트 손실로 인한 문제를 피하려면 If-Match 헤더를 항상 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-257">To avoid problems due to lost updates, always provide an If-Match header.</span></span>
>
>

## <a name="handling-large-requests-and-responses"></a><span data-ttu-id="6f4c2-258">큰 요청 및 응답 처리</span><span class="sxs-lookup"><span data-stu-id="6f4c2-258">Handling large requests and responses</span></span>
<span data-ttu-id="6f4c2-259">클라이언트 응용 프로그램에서 크기가 수 MB(또는 그 이상)인 데이터를 보내거나 받는 요청을 발급해야 하는 경우가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-259">There may be occasions when a client application needs to issue requests that send or receive data that may be several megabytes (or bigger) in size.</span></span> <span data-ttu-id="6f4c2-260">이 정도 크기의 데이터가 전송되는 동안 대기하다가 클라이언트 응용 프로그램이 응답하지 않는 상태가 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-260">Waiting while this amount of data is transmitted could cause the client application to become unresponsive.</span></span> <span data-ttu-id="6f4c2-261">상당한 양의 데이터를 포함하는 요청을 처리해야 하는 경우 다음 사항을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-261">Consider the following points when you need to handle requests that include significant amounts of data:</span></span>

### <a name="optimize-requests-and-responses-that-involve-large-objects"></a><span data-ttu-id="6f4c2-262">큰 개체를 포함하는 요청 및 응답을 최적화합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-262">Optimize requests and responses that involve large objects</span></span>

<span data-ttu-id="6f4c2-263">일부 리소스는 큰 개체이거나 그래픽 이미지 또는 다른 형식의 이진 데이터와 같이 큰 필드를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-263">Some resources may be large objects or include large fields, such as graphics images or other types of binary data.</span></span> <span data-ttu-id="6f4c2-264">Web API는 이러한 리소스의 업로딩과 다운로딩을 최적화할 수 있도록 스트리밍을 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-264">A web API should support streaming to enable optimized uploading and downloading of these resources.</span></span>

<span data-ttu-id="6f4c2-265">HTTP 프로토콜은 큰 데이터 개체를 클라이언트로 다시 스트리밍하기 위하여 청크된 전송 인코딩 메커니즘을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-265">The HTTP protocol provides the chunked transfer encoding mechanism to stream large data objects back to a client.</span></span> <span data-ttu-id="6f4c2-266">클라이언트가 큰 개체에 대한 HTTP GET 요청을 보내면, Web API는 HTTP 연결을 통해 증분 *청크*로 회신을 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-266">When the client sends an HTTP GET request for a large object, the web API can send the reply back in piecemeal *chunks* over an HTTP connection.</span></span> <span data-ttu-id="6f4c2-267">회신에 포함된 데이터의 길이를 처음에는 모를 수 있기 때문에(생성될 수 있습니다) Web API를 호스팅하는 서버는 Content-Length 헤더가 아닌 Transfer-Encoding: Chunked 헤더를 지정하는 각각의 청크를 포함하는 응답 메시지를 보내야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-267">The length of the data in the reply may not be known initially (it might be generated), so the server hosting the web API should send a response message with each chunk that specifies the Transfer-Encoding: Chunked header rather than a Content-Length header.</span></span> <span data-ttu-id="6f4c2-268">클라이언트 응용 프로그램은 각각의 청크를 차례로 받아서 완전한 응답을 빌드할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-268">The client application can receive each chunk in turn to build up the complete response.</span></span> <span data-ttu-id="6f4c2-269">데이터 전송은 서버에서 크기가 0인 마지막 청크를 보내면 완료됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-269">The data transfer completes when the server sends back a final chunk with zero size.</span></span> 

<span data-ttu-id="6f4c2-270">단일 요청으로 인해 상당한 리소스를 사용하는 대규모 개체가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-270">A single request could conceivably result in a massive object that consumes considerable resources.</span></span> <span data-ttu-id="6f4c2-271">스트리밍 프로세스 중에 Web API가 요청에 포함된 데이터의 양이 허용할 수 있는 한도를 초과했다고 판단하면 작업을 중단하고 상태 코드 413(요청 엔터티 너무 큼)과 함께 응답 메시지를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-271">If, during the streaming process, the web API determines that the amount of data in a request has exceeded some acceptable bounds, it can abort the operation and return a response message with status code 413 (Request Entity Too Large).</span></span>

<span data-ttu-id="6f4c2-272">HTTP 압축을 사용하여 네트워크를 통해 전송되는 큰 개체의 크기를 최소화할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-272">You can minimize the size of large objects transmitted over the network by using HTTP compression.</span></span> <span data-ttu-id="6f4c2-273">이 방법은 Web API를 호스팅하는 서버와 클라이언트에서 추가적인 프로세스를 필요로 하지만 네트워크 트래픽의 양 및 그와 연관된 네트워크 대기 시간을 줄이는데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-273">This approach helps to reduce the amount of network traffic and the associated network latency, but at the cost of requiring additional processing at the client and the server hosting the web API.</span></span> <span data-ttu-id="6f4c2-274">예를 들어, 압축 데이터를 수신할 것으로 예상되는 클라이언트 응용 프로그램은 Accept-Encoding: gzip(다른 압축 알고리즘도 지정될 수 있습니다.) 요청 헤더를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-274">For example, a client application that expects to receive compressed data can include an Accept-Encoding: gzip request header (other data compression algorithms can also be specified).</span></span> <span data-ttu-id="6f4c2-275">압축을 지원하는 서버는 Content-Encoding: gzip 응답 헤더와 gzip 형식으로 포함된 콘텐츠를 메시지 본문에 넣어 응답해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-275">If the server supports compression it should respond with the content held in gzip format in the message body and the Content-Encoding: gzip response header.</span></span>

<span data-ttu-id="6f4c2-276">인코딩된 압축을 스트리밍과 결합할 수 있습니다. 데이터를 스트리밍하기 전에 우선 압축하고 메시지 헤더에 gzip 콘텐츠 인코딩과 청크된 전송 인코딩을 명시합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-276">You can combine encoded compression with streaming; compress the data first before streaming it, and specify the gzip content encoding and chunked transfer encoding in the message headers.</span></span> <span data-ttu-id="6f4c2-277">일부 웹 서버(예: Internet Information Server)는 Web API에서의 데이터 압축 여부와 상관 없이 HTTP 응답을 자동으로 압축하도록 구성될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-277">Also note that some web servers (such as Internet Information Server) can be configured to automatically compress HTTP responses regardless of whether the web API compresses the data or not.</span></span>

### <a name="implement-partial-responses-for-clients-that-do-not-support-asynchronous-operations"></a><span data-ttu-id="6f4c2-278">비동기 작업을 지원하지 않는 클라이언트에 대해 부분 응답을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-278">Implement partial responses for clients that do not support asynchronous operations</span></span>

<span data-ttu-id="6f4c2-279">비동기 스트리밍에 대한 대안으로 클라이언트 응용 프로그램은 큰 개체의 데이터를 명시적으로 청크로 요청할 수 있으며 이를 부분 응답이라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-279">As an alternative to asynchronous streaming, a client application can explicitly request data for large objects in chunks, known as partial responses.</span></span> <span data-ttu-id="6f4c2-280">클라이언트 응용 프로그램은 개체에 대한 정보를 입수하기 위해 HTTP HEAD 요청을 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-280">The client application sends an HTTP HEAD request to obtain information about the object.</span></span> <span data-ttu-id="6f4c2-281">Web API가 부분 응답을 지원하면 HEAD 요청에 대해 Accept-Ranges 헤더 및 개체의 총 크기를 나타내는 Content-Length 헤더를 포함하지만 메시지 본문은 빈 상태의 응답 메시지로 응답해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-281">If the web API supports partial responses if should respond to the HEAD request with a response message that contains an Accept-Ranges header and a Content-Length header that indicates the total size of the object, but the body of the message should be empty.</span></span> <span data-ttu-id="6f4c2-282">클라이언트 응용 프로그램은 이 정보를 사용하여 수신할 바이트의 범위를 지정하는 일련의 GET 요청을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-282">The client application can use this information to construct a series of GET requests that specify a range of bytes to receive.</span></span> <span data-ttu-id="6f4c2-283">Web API는 HTTP 상태 코드 206(부분 콘텐츠), 응답 메시지의 본문에 포함된 실제 데이터 양을 나타내는 Content-Length 헤더, 데이터가 개체의 어느 부분(예: 4000~ 8000바이트)에 해당하는지를 나타내는 Content-Range 헤더를 포함하는 응답 메시지를 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-283">The web API should return a response message with HTTP status 206 (Partial Content), a Content-Length header that specifies the actual amount of data included in the body of the response message, and a Content-Range header that indicates which part (such as bytes 4000 to 8000) of the object this data represents.</span></span>

<span data-ttu-id="6f4c2-284">HTTP HEAD 요청 및 부분 응답은 [API 디자인][api-design]에 보다 자세히 설명되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-284">HTTP HEAD requests and partial responses are described in more detail in [API Design][api-design].</span></span>

### <a name="avoid-sending-unnecessary-100-continue-status-messages-in-client-applications"></a><span data-ttu-id="6f4c2-285">클라이언트 응용 프로그램에서 불필요한 100(계속) 상태 메시지 전송을 자제합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-285">Avoid sending unnecessary 100-Continue status messages in client applications</span></span>

<span data-ttu-id="6f4c2-286">서버에 대량의 데이터를 보내려고 하는 클라이언트 응용 프로그램은 우선 서버에서 실제로 요청을 수신하려고 하는지를 판단합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-286">A client application that is about to send a large amount of data to a server may determine first whether the server is actually willing to accept the request.</span></span> <span data-ttu-id="6f4c2-287">데이터를 보내기 전에 클라이언트 응용 프로그램은 Expect: 100-Continue 헤더와 데이터의 크기를 나타내는 Content-Length 헤더를 포함시키고 메시지 본문은 빈 상태로 HTTP 요청을 제출할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-287">Prior to sending the data, the client application can submit an HTTP request with an Expect: 100-Continue header, a Content-Length header that indicates the size of the data, but an empty message body.</span></span> <span data-ttu-id="6f4c2-288">서버가 요청을 처리하려고 하는 경우, HTTP 상태 100(계속)을 나타내는 메시지로 응답합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-288">If the server is willing to handle the request, it should respond with a message that specifies the HTTP status 100 (Continue).</span></span> <span data-ttu-id="6f4c2-289">그러면 클라이언트 응용 프로그램은 메시지 본문에 데이터를 포함하는 완전한 요청을 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-289">The client application can then proceed and send the complete request including the data in the message body.</span></span>

<span data-ttu-id="6f4c2-290">IIS를 사용하여 서비스를 호스팅하는 경우, 웹 응용 프로그램에 요청을 전달하기 전에 HTTP.sys 드라이버가 Expect: 100-Continue 헤더를 자동으로 감지하여 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-290">If you are hosting a service by using IIS, the HTTP.sys driver automatically detects and handles Expect: 100-Continue headers before passing requests to your web application.</span></span> <span data-ttu-id="6f4c2-291">따라서 응용 프로그램 코드에서 이러한 헤더를 볼 가능성이 없으며, IIS가 적절하지 않거나 너무 큰 것으로 간주되는 모든 메시지를 이미 필터링 한 것으로 추정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-291">This means that you are unlikely to see these headers in your application code, and you can assume that IIS has already filtered any messages that it deems to be unfit or too large.</span></span>

<span data-ttu-id="6f4c2-292">.NET Framework를 사용하여 클라이언트 응용 프로그램을 빌드하는 경우에는 모든 POST 및 PUT 메시지가 기본적으로 Expect: 100-Continue 헤더를 포함하는 메시지를 우선 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-292">If you are building client applications by using the .NET Framework, then all POST and PUT messages will first send messages with Expect: 100-Continue headers by default.</span></span> <span data-ttu-id="6f4c2-293">서버쪽의 경우 .NET Framework에 의해 프로세스가 투명하게 처리됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-293">As with the server-side, the process is handled transparently by the .NET Framework.</span></span> <span data-ttu-id="6f4c2-294">하지만 이 프로세스는 POST 및 PUT 요청에 대해 (작은 요청에 대해서도) 서버와의 2회 왕복을 유발합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-294">However, this process results in each POST and PUT request causing two round-trips to the server, even for small requests.</span></span> <span data-ttu-id="6f4c2-295">응용 프로그램이 대량의 데이터를 포함하는 요청을 보내지 않는 경우, `ServicePointManager` 클래스를 사용하여 클라이언트 응용 프로그램에서 `ServicePoint` 개체를 생성하도록 하여 이 기능을 비활성화시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-295">If your application is not sending requests with large amounts of data, you can disable this feature by using the `ServicePointManager` class to create `ServicePoint` objects in the client application.</span></span> <span data-ttu-id="6f4c2-296">`ServicePoint` 개체는 서버의 리소스를 식별하는 URI의 호스트 조각과 스키마를 기반으로 클라이언트가 서버에 만드는 연결을 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-296">A `ServicePoint` object handles the connections that the client makes to a server based on the scheme and host fragments of URIs that identify resources on the server.</span></span> <span data-ttu-id="6f4c2-297">그 후 `ServicePoint` 개체의 `Expect100Continue`속성을 false로 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-297">You can then set the `Expect100Continue` property of the `ServicePoint` object to false.</span></span> <span data-ttu-id="6f4c2-298">`ServicePoint` 개체의 호스트 조각 및 스키마와 부합하는 URI를 통해 클라이언트에서 만드는 모든 후속 POST 및 PUT 요청은 Expect: 100-Continue 헤더 없이 전송됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-298">All subsequent POST and PUT requests made by the client through a URI that matches the scheme and host fragments of the `ServicePoint` object will be sent without Expect: 100-Continue headers.</span></span> <span data-ttu-id="6f4c2-299">다음 코드는 `http` 스키마와 `www.contoso.com` 호스트를 포함하는 URI로 전송되는 모든 요청을 구성하는 `ServicePoint` 개체를 구성하는 방법을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-299">The following code shows how to configure a `ServicePoint` object that configures all requests sent to URIs with a scheme of `http` and a host of `www.contoso.com`.</span></span>

```csharp
Uri uri = new Uri("http://www.contoso.com/");
ServicePoint sp = ServicePointManager.FindServicePoint(uri);
sp.Expect100Continue = false;
```

<span data-ttu-id="6f4c2-300">후속으로 생성되는 모든 `ServicePoint` 개체에 대해 이 속성의 기본값을 지정하기 위하여 `ServicePointManager` 클래스의 정적 `Expect100Continue` 속성을 설정할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-300">You can also set the static `Expect100Continue` property of the `ServicePointManager` class to specify the default value of this property for all subsequently created `ServicePoint` objects.</span></span> <span data-ttu-id="6f4c2-301">자세한 내용은 [ServicePoint 클래스](https://msdn.microsoft.com/library/system.net.servicepoint.aspx)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-301">For more information, see [ServicePoint Class](https://msdn.microsoft.com/library/system.net.servicepoint.aspx).</span></span>

### <a name="support-pagination-for-requests-that-may-return-large-numbers-of-objects"></a><span data-ttu-id="6f4c2-302">다수의 개체를 반환할 수 있는 요청에 대해 페이지 매김을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-302">Support pagination for requests that may return large numbers of objects</span></span>

<span data-ttu-id="6f4c2-303">컬렉션에 다수의 리소스가 포함되어 있는 경우 해당 URI에 GET 요청을 발급하면 Web API를 호스팅하는 서버에 상당한 양의 프로세스를 발생시켜서 성능에 영향을 미치고 상당한 양의 네트워크 트래픽이 발생하여 대기 시간을 증가시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-303">If a collection contains a large number of resources, issuing a GET request to the corresponding URI could result in significant processing on the server hosting the web API affecting performance, and generate a significant amount of network traffic resulting in increased latency.</span></span>

<span data-ttu-id="6f4c2-304">이런 경우를 처리하기 위하여 Web API는 클라이언트 응용 프로그램이 비교적 처리가 쉬운 블록(이나 페이지)에서 요청을 구체화하거나 데이터를 가져올 수 있도록 하는 쿼리 문자열을 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-304">To handle these cases, the web API should support query strings that enable the client application to refine requests or fetch data in more manageable, discrete blocks (or pages).</span></span> <span data-ttu-id="6f4c2-305">아래 코드는 `Orders` 컨트롤러의 `GetAllOrders` 메서드를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-305">The code below shows the `GetAllOrders` method in the `Orders` controller.</span></span> <span data-ttu-id="6f4c2-306">이 메서드는 order의 세부 정보를 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-306">This method retrieves the details of orders.</span></span> <span data-ttu-id="6f4c2-307">이 메서드에 제약이 없다면 아마도 대량의 데이터를 반환할 수 있을 것입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-307">If this method was unconstrained, it could conceivably return a large amount of data.</span></span> <span data-ttu-id="6f4c2-308">`limit` 및 `offset` 매개 변수는 데이터의 양을 보다 작은 하위 집합으로 줄이기 위한 것이며, 이 경우에는 기본적으로 처음 10개의 order만 해당합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-308">The `limit` and `offset` parameters are intended to reduce the volume of data to a smaller subset, in this case only the first 10 orders by default:</span></span>

```csharp
public class OrdersController : ApiController
{
    ...
    [Route("api/orders")]
    [HttpGet]
    public IEnumerable<Order> GetAllOrders(int limit=10, int offset=0)
    {
        // Find the number of orders specified by the limit parameter
        // starting with the order specified by the offset parameter
        var orders = ...
        return orders;
    }
    ...
}
```

<span data-ttu-id="6f4c2-309">클라이언트 응용 프로그램은 URI `http://www.adventure-works.com/api/orders?limit=30&offset=50`을 사용하여 오프셋 50에서 시작하여 30개의 주문을 가져오는 요청을 발급할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-309">A client application can issue a request to retrieve 30 orders starting at offset 50 by using the URI `http://www.adventure-works.com/api/orders?limit=30&offset=50`.</span></span>

> [!TIP]
> <span data-ttu-id="6f4c2-310">2000자 보다 긴 URI를 생성하는 쿼리 문자열을 지정하도록 클라이언트 응용 프로그램을 사용하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-310">Avoid enabling client applications to specify query strings that result in a URI that is more than 2000 characters long.</span></span> <span data-ttu-id="6f4c2-311">많은 웹 클라이언트 및 서버는 이렇게 긴 URI를 처리할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-311">Many web clients and servers cannot handle URIs that are this long.</span></span>
>
>

## <a name="maintaining-responsiveness-scalability-and-availability"></a><span data-ttu-id="6f4c2-312">응답성, 확장성 및 가용성 유지 관리</span><span class="sxs-lookup"><span data-stu-id="6f4c2-312">Maintaining responsiveness, scalability, and availability</span></span>
<span data-ttu-id="6f4c2-313">동일한 Web API를 전세계 어디에서나 실행되고 있는 많은 클라이언트 응용 프로그램에서 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-313">The same web API might be utilized by many client applications running anywhere in the world.</span></span> <span data-ttu-id="6f4c2-314">Web API는 부하가 큰 경우에도 응답성을 유지하도록, 변화가 매우 큰 워크로드를 지원하기 위해 축소와 확장이 가능하도록, 주요 비즈니스 작업을 수행하는 클라이언트에 대한 가용성을 보장하도록 구현하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-314">It is important to ensure that the web API is implemented to maintain responsiveness under a heavy load, to be scalable to support a highly varying workload, and to guarantee availability for clients that perform business-critical operations.</span></span> <span data-ttu-id="6f4c2-315">이러한 요구 사항을 충족하기 위한 방법을 결정할 때 다음 사항을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-315">Consider the following points when determining how to meet these requirements:</span></span>

### <a name="provide-asynchronous-support-for-long-running-requests"></a><span data-ttu-id="6f4c2-316">오래 실행되는 요청에 대해 비동기 지원을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-316">Provide asynchronous support for long-running requests</span></span>

<span data-ttu-id="6f4c2-317">처리에 긴 시간에 소요되는 요청은 요청을 제출한 클라이언트를 차단하지 말고 수행되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-317">A request that might take a long time to process should be performed without blocking the client that submitted the request.</span></span> <span data-ttu-id="6f4c2-318">Web API는 요청의 유효성을 검사하기 위한 초기 확인을 수행하고 업무를 수행할 개별 작업을 시작하고, HTTP 코드 202(수락됨)와 함께 응답 메시지를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-318">The web API can perform some initial checking to validate the request, initiate a separate task to perform the work, and then return a response message with HTTP code 202 (Accepted).</span></span> <span data-ttu-id="6f4c2-319">작업은 Web API 처리의 일부로 비동기적으로 실행되거나 백그라운드 작업에 오프로드될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-319">The task could run asynchronously as part of the web API processing, or it could be offloaded to a background task.</span></span>

<span data-ttu-id="6f4c2-320">Web API는 처리 결과를 클라이언트 응용 프로그램에 반환하기 위한 메커니즘을 제공해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-320">The web API should also provide a mechanism to return the results of the processing to the client application.</span></span> <span data-ttu-id="6f4c2-321">처리가 완료되었는지를 주기적으로 쿼리하고 결과를 입수하기 위하여 클라이언트 응용 프로그램에 대한 폴링 메커니즘을 제공하거나 작업이 완료되면 Web API가 알림을 보낼 수 있도록 하는 방법으로 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-321">You can achieve this by providing a polling mechanism for client applications to periodically query whether the processing has finished and obtain the result, or enabling the web API to send a notification when the operation has completed.</span></span>

<span data-ttu-id="6f4c2-322">다음 방법을 사용하여 가상 리소스 역할을 하는 *polling* URI를 제공하여 간단한 폴링 메커니즘을 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-322">You can implement a simple polling mechanism by providing a *polling* URI that acts as a virtual resource using the following approach:</span></span>

1. <span data-ttu-id="6f4c2-323">클라이언트 응용 프로그램이 Web API에 초기 요청을 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-323">The client application sends the initial request to the web API.</span></span>
2. <span data-ttu-id="6f4c2-324">Web API가 요청에 대한 정보를 Microsoft Azure Cache 또는 테이블 저장소에 있는 테이블에 저장하고 이 항목에 대한 고유 키를 가급적 GUID 형태로 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-324">The web API stores information about the request in a table held in table storage or Microsoft Azure Cache, and generates a unique key for this entry, possibly in the form of a GUID.</span></span>
3. <span data-ttu-id="6f4c2-325">Web API는 별도의 작업으로 처리를 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-325">The web API initiates the processing as a separate task.</span></span> <span data-ttu-id="6f4c2-326">Web API는 테이블에 작업 상태를 *Running*으로 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-326">The web API records the state of the task in the table as *Running*.</span></span>
4. <span data-ttu-id="6f4c2-327">Web API는 HTTP 상태 코드 202(수락됨)와 함께 메시지 본문에 테이블 항목의 GUID를 넣어서 응답 메시지를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-327">The web API returns a response message with HTTP status code 202 (Accepted), and the GUID of the table entry in the body of the message.</span></span>
5. <span data-ttu-id="6f4c2-328">작업이 완료되면 Web API는 결과를 테이블에 저장하고 작업의 상태를 *Complete*로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-328">When the task has completed, the web API stores the results in the table, and sets the state of the task to *Complete*.</span></span> <span data-ttu-id="6f4c2-329">작업이 실패하면 Web API는 실패에 대한 정보를 보관하고 상태를 *Failed*로 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-329">Note that if the task fails, the web API could also store information about the failure and set the status to *Failed*.</span></span>
6. <span data-ttu-id="6f4c2-330">작업이 실행되는 동안 클라이언트는 자체적인 프로세스를 계속 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-330">While the task is running, the client can continue performing its own processing.</span></span> <span data-ttu-id="6f4c2-331">URI */polling/{guid}* 에 주기적으로 요청을 보낼 수 있습니다. 여기서 *{guid}* 는 Web API에 의해 202 응답 메시지로 반환된 GUID입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-331">It can periodically send a request to the URI */polling/{guid}* where *{guid}* is the GUID returned in the 202 response message by the web API.</span></span>
7. <span data-ttu-id="6f4c2-332">*/polling/{guid}* URI의 Web API는 테이블에 있는 해당 작업의 상태를 쿼리하고 이 상태(*Running*, *Complete* 또는 *Failed*)를 포함하는 HTTP 상태 코드 200(정상)과 함께 응답 메시지를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-332">The web API at the */polling/{guid}* URI queries the state of the corresponding task in the table and returns a response message with HTTP status code 200 (OK) containing this state (*Running*, *Complete*, or *Failed*).</span></span> <span data-ttu-id="6f4c2-333">작업이 완료되거나 실패하면 응답 메시지에 처리 결과 또는 실패의 이유에 대한 정보를 포함시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-333">If the task has completed or failed, the response message can also include the results of the processing or any information available about the reason for the failure.</span></span>

<span data-ttu-id="6f4c2-334">알림을 구현하기 위한 옵션은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-334">Options for implementing notifications include:</span></span>

- <span data-ttu-id="6f4c2-335">Azure 알림 허브를 사용하여 클라이언트 응용 프로그램에 대한 비동기 응답을 푸시합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-335">Using an Azure Notification Hub to push asynchronous responses to client applications.</span></span> <span data-ttu-id="6f4c2-336">자세한 내용은 [Azure Notification Hubs 알릴 사용자](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-336">For more information, see [Azure Notification Hubs Notify Users](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/).</span></span>
- <span data-ttu-id="6f4c2-337">Comet 모델을 사용하여 클라이언트와 Web API를 호스팅하는 서버 사이의 영구적인 네트워크 연결을 유지하고, 이 연결을 사용하여 서버의 메시지를 클라이언트로 푸시합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-337">Using the Comet model to retain a persistent network connection between the client and the server hosting the web API, and using this connection to push messages from the server back to the client.</span></span> <span data-ttu-id="6f4c2-338">MSDN Magazine [Building a Simple Comet Application in the Microsoft .NET Framework](https://msdn.microsoft.com/magazine/jj891053.aspx) (Microsoft .NET Framework에서 간단한 Comet 응용 프로그램 빌드) 문서에 예제 솔루션 설명이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-338">The MSDN magazine article [Building a Simple Comet Application in the Microsoft .NET Framework](https://msdn.microsoft.com/magazine/jj891053.aspx) describes an example solution.</span></span>
- <span data-ttu-id="6f4c2-339">SignalR을 사용하여 영구적인 네트워크 연결을 통해 실시간으로 웹 서버에서 클라이언트로 데이터를 푸시합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-339">Using SignalR to push data in real-time from the web server to the client over a persistent network connection.</span></span> <span data-ttu-id="6f4c2-340">SignalR은 ASP.NET 웹 응용프로그램에서 NuGet 패키지로 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-340">SignalR is available for ASP.NET web applications as a NuGet package.</span></span> <span data-ttu-id="6f4c2-341">[ASP.NET SignalR](http://signalr.net/) 웹 페이지에서 자세한 내용을 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-341">You can find more information on the [ASP.NET SignalR](http://signalr.net/) website.</span></span>

### <a name="ensure-that-each-request-is-stateless"></a><span data-ttu-id="6f4c2-342">각 요청은 상태 비저장이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-342">Ensure that each request is stateless</span></span>

<span data-ttu-id="6f4c2-343">각 요청을 원자성으로 간주해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-343">Each request should be considered atomic.</span></span> <span data-ttu-id="6f4c2-344">클라이언트 응용 프로그램이 만든 요청과 동일한 클라이언트에서 제출한 후속 요청 사이에는 종속성이 없어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-344">There should be no dependencies between one request made by a client application and any subsequent requests submitted by the same client.</span></span> <span data-ttu-id="6f4c2-345">이 방법은 확장성을 지원합니다. 웹 서비스 인스턴스는 수많은 서버에 배포될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-345">This approach assists in scalability; instances of the web service can be deployed on a number of servers.</span></span> <span data-ttu-id="6f4c2-346">클라이언트 요청은 이들 중 어떤 인스턴스에도 전달될 수 있으며 결과는 항상 동일해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-346">Client requests can be directed at any of these instances and the results should always be the same.</span></span> <span data-ttu-id="6f4c2-347">비슷한 이유로 가용성을 향상시킵니다. 웹 서버가 실패하면 요청은 서버가 재시작되는 동안 클라이언트 응용 프로그램에 나쁜 효과를 미치지 않고 다른 인스턴스로(Azure Traffic Manager를 사용하여) 라우팅될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-347">It also improves availability for a similar reason; if a web server fails requests can be routed to another instance (by using Azure Traffic Manager) while the server is restarted with no ill effects on client applications.</span></span>

### <a name="track-clients-and-implement-throttling-to-reduce-the-chances-of-dos-attacks"></a><span data-ttu-id="6f4c2-348">클라이언트를 추적하고 제한을 구현하여 DOS 공격 가능성을 줄입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-348">Track clients and implement throttling to reduce the chances of DOS attacks</span></span>

<span data-ttu-id="6f4c2-349">특정 클라이언트가 일정 시간 안에 대단히 많은 수의 요청을 하면, 서비스를 독점할 수 있고 다른 클라이언트의 성능에 영향을 미칠 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-349">If a specific client makes a large number of requests within a given period of time it might monopolize the service and affect the performance of other clients.</span></span> <span data-ttu-id="6f4c2-350">이 문제를 완화하기 위하여 Web API는 들어오는 모든 요청의 IP 주소를 추적하거나 각각의 인증된 액세스를 기록하는 방식으로 클라이언트 응용 프로그램에서 나오는 호출을 모니터링할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-350">To mitigate this issue, a web API can monitor calls from client applications either by tracking the IP address of all incoming requests or by logging each authenticated access.</span></span> <span data-ttu-id="6f4c2-351">리소스 액세스를 제한하기 위해 이 정보를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-351">You can use this information to limit resource access.</span></span> <span data-ttu-id="6f4c2-352">클라이언트가 정해진 제한을 초과하면 Web API는 상태 503(서비스를 사용할 수 없음)과 함께 다음 요청을 언제 보낼 수 있는지(거부되지 않고)를 나타내는 Retry-After 헤더를 포함하는 응답 메시지를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-352">If a client exceeds a defined limit, the web API can return a response message with status 503 (Service Unavailable) and include a Retry-After header that specifies when the client can send the next request without it being declined.</span></span> <span data-ttu-id="6f4c2-353">이 전략은 시스템을 지연시키는 클라이언트로부터 DoS(서비스 거부) 공격의 가능성을 줄이는데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-353">This strategy can help to reduce the chances of a Denial Of Service (DOS) attack from a set of clients stalling the system.</span></span>

### <a name="manage-persistent-http-connections-carefully"></a><span data-ttu-id="6f4c2-354">영구적인 HTTP 연결을 신중하게 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-354">Manage persistent HTTP connections carefully</span></span>

<span data-ttu-id="6f4c2-355">HTTP 프로토콜은 영구적인 HTTP 연결(이 가능한 곳에서)을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-355">The HTTP protocol supports persistent HTTP connections where they are available.</span></span> <span data-ttu-id="6f4c2-356">HTTP 1.0 사양은 클라이언트 응용 프로그램이 새로 연결을 만들지 않고 동일한 연결을 사용하여 후속 요청을 보낼 수 있음을 서버에 지시할 수 있도록 하는 Connection:Keep-Alive 헤더를 추가했습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-356">The HTTP 1.0 specificiation added the Connection:Keep-Alive header that enables a client application to indicate to the server that it can use the same connection to send subsequent requests rather than opening new ones.</span></span> <span data-ttu-id="6f4c2-357">호스트가 정한 기간 내에 클라이언트가 연결을 재사용하지 않으면 연결이 자동으로 닫힙니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-357">The connection closes automatically if the client does not reuse the connection within a period defined by the host.</span></span> <span data-ttu-id="6f4c2-358">이러한 동작은 Azure 서비스에 의해 사용되며 HTTP 1.1의 기본값입니다. 따라서 메시지에 Keep-Alive 헤더를 포함시킬 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-358">This behavior is the default in HTTP 1.1 as used by Azure services, so there is no need to include Keep-Alive headers in messages.</span></span>

<span data-ttu-id="6f4c2-359">연결을 열린 상태로 유지하면 대기 시간과 네트워크 혼잡이 감소되어 응답성 향상에 도움이 되지만 불필요한 연결을 필요한 시간 보다 길게 열린 상태로 유지하고 다른 클라이언트의 동시간대 연결 성능을 제한하기 때문에 확장성에는 불리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-359">Keeping a connection open can help to improve responsiveness by reducing latency and network congestion, but it can be detrimental to scalability by keeping unnecessary connections open for longer than required, limiting the ability of other concurrent clients to connect.</span></span> <span data-ttu-id="6f4c2-360">클라이언트 응용 프로그램이 모바일 장치에서 실행되는 경우 배터리 수명에도 영향을 미칠 수 있습니다. 응용 프로그램이 서버에 간헐적인 요청을 보내는 경우, 연결을 열린 상태로 유지하면 배터리가 보다 빨리 방전될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-360">It can also affect battery life if the client application is running on a mobile device; if the application only makes occasional requests to the server, maintaining an open connection can cause the battery to drain more quickly.</span></span> <span data-ttu-id="6f4c2-361">HTTP 1.1 연결을 영구적으로 만들지 않으려면 클라이언트는 기본 동작을 재정의하는 메시지에 Connection:Close 헤더를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-361">To ensure that a connection is not made persistent with HTTP 1.1, the client can include a Connection:Close header with messages to override the default behavior.</span></span> <span data-ttu-id="6f4c2-362">마찬가지로 서버가 매우 많은 수의 클라이언트를 처리하는 경우 응답 메시지에 Connection:Close 헤더를 포함할 수 있으며 이렇게 하면 연결이 닫히고 서버 자원을 절약할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-362">Similarly, if a server is handling a very large number of clients it can include a Connection:Close header in response messages which should close the connection and save server resources.</span></span>

> [!NOTE]
> <span data-ttu-id="6f4c2-363">영구적인 HTTP 연결은 반복적으로 통신 채널을 수립하면서 발생하는 네트워크 오버헤드를 줄이기 위한 선택적인 기능입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-363">Persistent HTTP connections are a purely optional feature to reduce the network overhead associated with repeatedly establishing a communications channel.</span></span> <span data-ttu-id="6f4c2-364">Web API나 클라이언트 응용 프로그램 그 어느 쪽도 영구적인 HTTP 연결(을 사용할 수 있다고 해도)에 의존하지 말아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-364">Neither the web API nor the client application should depend on a persistent HTTP connection being available.</span></span> <span data-ttu-id="6f4c2-365">Comet 스타일 알림 시스템을 구현하기 위해 영구적인 HTTP 연결을 사용하지 마십시오. 대신 TCP 계층의 소켓(또는 가능하면 Websocket)을 활용하십시오.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-365">Do not use persistent HTTP connections to implement Comet-style notification systems; instead you should utilize sockets (or websockets if available) at the TCP layer.</span></span> <span data-ttu-id="6f4c2-366">클라이언트 응용 프로그램이 프록시를 통해 서버와 통신하는 경우 Keep-Alive 헤더는 별로 쓸모가 없습니다. 클라이언트와 프록시의 연결만 영구적입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-366">Finally, note Keep-Alive headers are of limited use if a client application communicates with a server via a proxy; only the connection with the client and the proxy will be persistent.</span></span>
>
>

## <a name="publishing-and-managing-a-web-api"></a><span data-ttu-id="6f4c2-367">Web API 게시 및 관리</span><span class="sxs-lookup"><span data-stu-id="6f4c2-367">Publishing and managing a web API</span></span>
<span data-ttu-id="6f4c2-368">Web API를 클라이언트 응용 프로그램에서 사용할 수 있도록 하려면 Web API가 호스트 환경에 배포되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-368">To make a web API available for client applications, the web API must be deployed to a host environment.</span></span> <span data-ttu-id="6f4c2-369">일반적으로 이런 환경은(다른 유형의 호스트 프로세스가 될 수도 있지만) 웹 서버입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-369">This environment is typically a web server, although it may be some other type of host process.</span></span> <span data-ttu-id="6f4c2-370">Web API를 게시하는 경우 다음 사항을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-370">You should consider the following points when publishing a web API:</span></span>

* <span data-ttu-id="6f4c2-371">모든 요청은 인증되고 권한이 부여되어야 하며, 적절한 액세스 제어 수준이 강제 적용되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-371">All requests must be authenticated and authorized, and the appropriate level of access control must be enforced.</span></span>
* <span data-ttu-id="6f4c2-372">상용 Web API는 응답 시간에 대한 다양한 품질 보장을 받을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-372">A commercial web API might be subject to various quality guarantees concerning response times.</span></span> <span data-ttu-id="6f4c2-373">시간 별로 부하가 상당히 많이 달라질 수 있다면, 호스트 환경의 확장성을 확인하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-373">It is important to ensure that host environment is scalable if the load can vary significantly over time.</span></span>
* <span data-ttu-id="6f4c2-374">경제적인 목적을 위해 요청을 계량 측정하는 것이 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-374">It may be necessary to meter requests for monetization purposes.</span></span>
* <span data-ttu-id="6f4c2-375">Web API에 대한 트래픽의 흐름을 규제하고 할당량이 소진된 클라이언트에 대해 제한을 구현하는 것이 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-375">It might be necessary to regulate the flow of traffic to the web API, and implement throttling for specific clients that have exhausted their quotas.</span></span>
* <span data-ttu-id="6f4c2-376">규제 요구 사항은 모든 요청과 응답의 기록 및 감사를 위임할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-376">Regulatory requirements might mandate logging and auditing of all requests and responses.</span></span>
* <span data-ttu-id="6f4c2-377">가용성을 보장하기 위하여 Web API를 호스팅하는 서버의 상태를 모니터링하고 필요하면 재시작해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-377">To ensure availability, it may be necessary to monitor the health of the server hosting the web API and restart it if necessary.</span></span>

<span data-ttu-id="6f4c2-378">Web API 구현과 관련하여, 이러한 문제를 기술적인 문제와 분리하는 것이 유용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-378">It is useful to be able to decouple these issues from the technical issues concerning the implementation of the web API.</span></span> <span data-ttu-id="6f4c2-379">따라서 별도 프로세스로 실행되면서 Web API에 요청을 라우팅하는 [외관](http://en.wikipedia.org/wiki/Facade_pattern) 생성을 고려하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-379">For this reason, consider creating a [façade](http://en.wikipedia.org/wiki/Facade_pattern), running as a separate process and that routes requests to the web API.</span></span> <span data-ttu-id="6f4c2-380">외관은 관리 작업을 제공하고 유효성을 검사한 요청을 Web API로 전달할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-380">The façade can provide the management operations and forward validated requests to the web API.</span></span> <span data-ttu-id="6f4c2-381">외관을 사용하면 다음과 같이 기능적인 이점이 많이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-381">Using a façade can also bring many functional advantages, including:</span></span>

* <span data-ttu-id="6f4c2-382">여러 Web API에 대한 통합 지점 역할을 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-382">Acting as an integration point for multiple web APIs.</span></span>
* <span data-ttu-id="6f4c2-383">메시지를 변환하고 다양한 기술을 사용하여 빌드한 클라이언트의 통신 프로토콜을 변환합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-383">Transforming messages and translating communications protocols for clients built by using varying technologies.</span></span>
* <span data-ttu-id="6f4c2-384">Web API를 호스팅하는 서버의 부하를 줄이기 위해 요청 및 응답을 캐시합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-384">Caching requests and responses to reduce load on the server hosting the web API.</span></span>

## <a name="testing-a-web-api"></a><span data-ttu-id="6f4c2-385">Web API 테스트</span><span class="sxs-lookup"><span data-stu-id="6f4c2-385">Testing a web API</span></span>
<span data-ttu-id="6f4c2-386">Web API는 다른 소프트웨어와 마찬가지로 철저히 테스트해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-386">A web API should be tested as thoroughly as any other piece of software.</span></span> <span data-ttu-id="6f4c2-387">Web API 특성의 기능이 고유한 추가 요구 사항을 가져오는지 확인하기 위해 단위 테스트를 만들어서 제대로 작동하는지 확인하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-387">You should consider creating unit tests to validate the functionality of The nature of a web API brings its own additional requirements to verify that it operates correctly.</span></span> <span data-ttu-id="6f4c2-388">다음과 같은 측면에 특히 주의를 기울여야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-388">You should pay particular attention to the following aspects:</span></span>

* <span data-ttu-id="6f4c2-389">올바른 작업을 호출하는지 확인하기 위하여 모든 경로를 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-389">Test all routes to verify that they invoke the correct operations.</span></span> <span data-ttu-id="6f4c2-390">예상치 않은 HTTP 상태 코드 405(메서드가 허용되지 않음) 반환에 특히 유의하십시오. 경로 및 경로에 디스패치할 수 있는 HTTP 메서드(GET, POST, PUT, DELETE) 사이의 불일치를 나타내는 것일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-390">Be especially aware of HTTP status code 405 (Method Not Allowed) being returned unexpectedly as this can indicate a mismatch between a route and the HTTP methods (GET, POST, PUT, DELETE) that can be dispatched to that route.</span></span>

    <span data-ttu-id="6f4c2-391">HTTP 요청을 지원하지 않는 경로에 HTTP 요청을 보냅니다. 예를 들어 특정 리소스에 POST 요청을 제출합니다. POST 요청은 리소스 컬렉션에만 전송해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-391">Send HTTP requests to routes that do not support them, such as submitting a POST request to a specific resource (POST requests should only be sent to resource collections).</span></span> <span data-ttu-id="6f4c2-392">이런 경우 유효한 응답은 상태 코드 405(허용되지 않음)여야 *합니다*.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-392">In these cases, the only valid response *should* be status code 405 (Not Allowed).</span></span>
* <span data-ttu-id="6f4c2-393">모든 경로가 적절하게 보호되는지 적절한 인증 및 권한에 속하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-393">Verify that all routes are protected properly and are subject to the appropriate authentication and authorization checks.</span></span>

  > [!NOTE]
  > <span data-ttu-id="6f4c2-394">사용자 인증과 같은 보안의 일부 측면은 Web API가 아닌 호스트 환경의 책임일 가능성이 높지만 배포 과정의 일부로 보안 테스트를 포함시킬 필요가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-394">Some aspects of security such as user authentication are most likely to be the responsibility of the host environment rather than the web API, but it is still necessary to include security tests as part of the deployment process.</span></span>
  >
  >
* <span data-ttu-id="6f4c2-395">각 작업에 의해 수행되는 예외 처리를 테스트한 후 적절하고 의미 있는 HTTP 응답이 클라이언트 응용 프로그램에 반환되는지를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-395">Test the exception handling performed by each operation and verify that an appropriate and meaningful HTTP response is passed back to the client application.</span></span>
* <span data-ttu-id="6f4c2-396">요청 및 응답 메시지가 올바르게 구성되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-396">Verify that request and response messages are well-formed.</span></span> <span data-ttu-id="6f4c2-397">예를 들어, HTTP POST 요청이 x-www-form-urlencoded 형식으로 된 새로운 리소스의 데이터를 포함하는 경우, 해당 작업이 데이터를 올바르게 분석하는지, 리소스를 생성하는지, 새 리소스의 세부 사항(올바른 Location 헤더를 포함하여)을 포함하는 응답을 반환하는지를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-397">For example, if an HTTP POST request contains the data for a new resource in x-www-form-urlencoded format, confirm that the corresponding operation correctly parses the data, creates the resources, and returns a response containing the details of the new resource, including the correct Location header.</span></span>
* <span data-ttu-id="6f4c2-398">응답 메시지의 모든 링크와 URI를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-398">Verify all links and URIs in response messages.</span></span> <span data-ttu-id="6f4c2-399">예를 들어, HTTP POST 메시지는 새로 생성된 리소스의 URI를 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-399">For example, an HTTP POST message should return the URI of the newly-created resource.</span></span> <span data-ttu-id="6f4c2-400">모든 HATEOAS 링크는 유효해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-400">All HATEOAS links should be valid.</span></span>

* <span data-ttu-id="6f4c2-401">각 작업이 다양한 입력 조합에 대해 올바른 상태 코드를 반환하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-401">Ensure that each operation returns the correct status codes for different combinations of input.</span></span> <span data-ttu-id="6f4c2-402">예: </span><span class="sxs-lookup"><span data-stu-id="6f4c2-402">For example:</span></span>

  * <span data-ttu-id="6f4c2-403">쿼리가 성공적이면 상태 코드 200(OK)을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-403">If a query is successful, it should return status code 200 (OK)</span></span>
  * <span data-ttu-id="6f4c2-404">리소스를 찾을 수 없으면 작업은 HTTP 상태 코드 404(찾을 수 없음)을 반환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-404">If a resource is not found, the operation should return HTTP status code 404 (Not Found).</span></span>
  * <span data-ttu-id="6f4c2-405">클라이언트가 요청을 전송하여 리소스 삭제를 완료한 경우, 상태 코드는 204(콘텐츠 없음)여야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-405">If the client sends a request that successfully deletes a resource, the status code should be 204 (No Content).</span></span>
  * <span data-ttu-id="6f4c2-406">클라이언트에서 새 리소스를 생성하는 요청을 보낸 경우 상태 코드는 201(생성됨)이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-406">If the client sends a request that creates a new resource, the status code should be 201 (Created)</span></span>

<span data-ttu-id="6f4c2-407">5xx 범위의 예상치 못한 응답 상태 코드에 유의하시기 바랍니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-407">Watch out for unexpected response status codes in the 5xx range.</span></span> <span data-ttu-id="6f4c2-408">이런 메시지는 보통 호스트 서버가 유효한 요청을 수행할 수 없다는 것을 나타내기 위해 보고됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-408">These messages are usually reported by the host server to indicate that it was unable to fulfill a valid request.</span></span>

* <span data-ttu-id="6f4c2-409">클라이언트 응용 프로그램에서 지정할 수 있는 다양한 조합의 헤더 요청을 테스트하고 Web API가 응답 메시지에 예상 정보를 반환하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-409">Test the different request header combinations that a client application can specify and ensure that the web API returns the expected information in response messages.</span></span>
* <span data-ttu-id="6f4c2-410">쿼리 문자열을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-410">Test query strings.</span></span> <span data-ttu-id="6f4c2-411">작업이 선택적인 매개 변수(예: 페이지 매김 요청)를 취할 수 있으면 매개 변수의 다른 조합과 순서를 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-411">If an operation can take optional parameters (such as pagination requests), test the different combinations and order of parameters.</span></span>
* <span data-ttu-id="6f4c2-412">비동기 작업이 완료되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-412">Verify that asynchronous operations complete successfully.</span></span> <span data-ttu-id="6f4c2-413">Web API가 큰 바이너리 개체를 반환하는 요청에 대해 스트리밍을 지원하는 경우 데이터가 스트리밍되는 동안 클라이언트 요청이 차단되지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-413">If the web API supports streaming for requests that return large binary objects (such as video or audio), ensure that client requests are not blocked while the data is streamed.</span></span> <span data-ttu-id="6f4c2-414">Web API가 오래 실행되는 데이터 수정 작업에 대해 폴링을 수행하는 경우, 작업이 진행되는 동안 상태를 제대로 보고하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-414">If the web API implements polling for long-running data modification operations, verify that that the operations report their status correctly as they proceed.</span></span>

<span data-ttu-id="6f4c2-415">Web API가 감금 하에서도 만족스럽게 작동하는지 확인하기 위하여 성능 테스트를 생성하고 실행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-415">You should also create and run performance tests to check that the web API operates satisfactorily under duress.</span></span> <span data-ttu-id="6f4c2-416">Visual Studio Ultimate을 사용하여 웹 성능 및 부하 테스트를 빌드할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-416">You can build a web performance and load test project by using Visual Studio Ultimate.</span></span> <span data-ttu-id="6f4c2-417">자세한 정보는 [릴리스 전에 응용 프로그램에서 성능 테스트 실행](https://msdn.microsoft.com/library/dn250793.aspx)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-417">For more information, see [Run performance tests on an application before a release](https://msdn.microsoft.com/library/dn250793.aspx).</span></span>

## <a name="using-azure-api-management"></a><span data-ttu-id="6f4c2-418">Azure API Management 사용</span><span class="sxs-lookup"><span data-stu-id="6f4c2-418">Using Azure API Management</span></span> 

<span data-ttu-id="6f4c2-419">Azure에서 [Azue API Management](https://azure.microsoft.com/documentation/services/api-management/)를 사용하여 Web API를 게시 및 관리하는 방안을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-419">On Azure, consider using [Azue API Management](https://azure.microsoft.com/documentation/services/api-management/) to publish and manage a web API.</span></span> <span data-ttu-id="6f4c2-420">이 기능을 사용하여 하나 이상의 Web API에 대해 외관 역할을 하는 서비스를 생성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-420">Using this facility, you can generate a service that acts as a façade for one or more web APIs.</span></span> <span data-ttu-id="6f4c2-421">서비스는 Azure 관리 포털을 사용하여 만들고 구성할 수 있는 축소/확장이 가능한 웹 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-421">The service is itself a scalable web service that you can create and configure by using the Azure Management portal.</span></span> <span data-ttu-id="6f4c2-422">이 서비스를 사용하여 다음과 같이 Web API를 게시 및 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-422">You can use this service to publish and manage a web API as follows:</span></span>

1. <span data-ttu-id="6f4c2-423">Web API를 웹 사이트, Azure 클라우드 서비스 또는 Azure 가상 머신에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-423">Deploy the web API to a website, Azure cloud service, or Azure virtual machine.</span></span>
2. <span data-ttu-id="6f4c2-424">API 관리 서비스를 Web API에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-424">Connect the API management service to the web API.</span></span> <span data-ttu-id="6f4c2-425">관리 API의 URL로 전송된 요청은 Web API의 URI로 매핑됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-425">Requests sent to the URL of the management API are mapped to URIs in the web API.</span></span> <span data-ttu-id="6f4c2-426">동일한 API 관리 서비스가 하나 이상의 Web API에 요청을 라우팅할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-426">The same API management service can route requests to more than one web API.</span></span> <span data-ttu-id="6f4c2-427">이를 통해 여러 개의 Web API를 단일 관리 서비스로 모을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-427">This enables you to aggregate multiple web APIs into a single management service.</span></span> <span data-ttu-id="6f4c2-428">마찬가지로, 다른 응용 프로그램에서 사용할 수 있는 기능을 제한하거나 분할하는 경우 동일한 Web API를 하나 이상의 API 관리 서비스에서 참조할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-428">Similarly, the same web API can be referenced from more than one API management service if you need to restrict or partition the functionality available to different applications.</span></span>

   > [!NOTE]
   > <span data-ttu-id="6f4c2-429">HTTP GET 요청에 대한 응답의 일부로 생성된 HATEOAS 링크의 URI는 Web API를 호스팅하는 웹 서버가 아닌 API 관리 서비스의 URL을 참조해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-429">The URIs in HATEOAS links generated as part of the response for HTTP GET requests should reference the URL of the API management service and not the web server hosting the web API.</span></span>
   >
   >
3. <span data-ttu-id="6f4c2-430">각각의 Web API에 대해 Web API가 작업에 입력 내용으로 취할 수 있는 다른 선택적인 매개 변수와 함께 노출하는 HTTP 작업을 명시합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-430">For each web API, specify the HTTP operations that the web API exposes together with any optional parameters that an operation can take as input.</span></span> <span data-ttu-id="6f4c2-431">API 관리 서비스가 동일한 데이터에 대해 반복되는 요청을 최적화하기 위해 Web API로부터 수신한 응답을 캐시해야 할지를 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-431">You can also configure whether the API management service should cache the response received from the web API to optimize repeated requests for the same data.</span></span> <span data-ttu-id="6f4c2-432">각각의 작업이 생성하는 HTTP 응답의 세부 사항을 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-432">Record the details of the HTTP responses that each operation can generate.</span></span> <span data-ttu-id="6f4c2-433">이 정보는 개발자를 위한 문서를 생성하는데 사용됩니다. 따라서 정확하고 완전한 정보를 기록하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-433">This information is used to generate documentation for developers, so it is important that it is accurate and complete.</span></span>

   <span data-ttu-id="6f4c2-434">Azure 관리 포털에서 제공하는 마법사를 사용하여 수동으로 작업을 정의하거나 WADL 또는 Swagger 형식의 정의를 포함하는 파일로부터 가져올 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-434">You can either define operations manually using the wizards provided by the Azure Management portal, or you can import them from a file containing the definitions in WADL or Swagger format.</span></span>
4. <span data-ttu-id="6f4c2-435">API 관리 서비스와 Web API를 호스팅하는 서버 사이의 통신에 대한 보안 설정을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-435">Configure the security settings for communications between the API management service and the web server hosting the web API.</span></span> <span data-ttu-id="6f4c2-436">API 관리 서비스는 OAuth 2.0 사용자 권한 부여 및 인증서를 사용하여 기본 인증 및 상호 인증을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-436">The API management service currently supports Basic authentication and mutual authentication using certificates, and OAuth 2.0 user authorization.</span></span>
5. <span data-ttu-id="6f4c2-437">제품을 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-437">Create a product.</span></span> <span data-ttu-id="6f4c2-438">제품은 게시 단위입니다. 이전에 관리 서비스에 연결한 Web API를 제품에 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-438">A product is the unit of publication; you add the web APIs that you previously connected to the management service to the product.</span></span> <span data-ttu-id="6f4c2-439">제품이 게시되면 Web API를 개발자들이 사용할 수 있게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-439">When the product is published, the web APIs become available to developers.</span></span>

   > [!NOTE]
   > <span data-ttu-id="6f4c2-440">제품 게시에 앞서, 제품에 액세스할 수 있는 사용자 그룹을 정의하고 그룹에 사용자를 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-440">Prior to publishing a product, you can also define user-groups that can access the product and add users to these groups.</span></span> <span data-ttu-id="6f4c2-441">이렇게 하면 Web API를 사용할 수 있는 개발자와 응용 프로그램을 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-441">This gives you control over the developers and applications that can use the web API.</span></span> <span data-ttu-id="6f4c2-442">승인을 받아야 하는 Web API에 대해 액세스 권한을 확보하려면 개발자는 제품 관리자에게 반드시 요청을 보내야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-442">If a web API is subject to approval, prior to being able to access it a developer must send a request to the product administrator.</span></span> <span data-ttu-id="6f4c2-443">관리자는 개발자의 액세스를 허용하거나 거부할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-443">The administrator can grant or deny access to the developer.</span></span> <span data-ttu-id="6f4c2-444">상황이 변하면 기존 개발자를 차단할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-444">Existing developers can also be blocked if circumstances change.</span></span>
   >
   >
6. <span data-ttu-id="6f4c2-445">각각의 Web API에 대한 정책을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-445">Configure policies for each web API.</span></span> <span data-ttu-id="6f4c2-446">정책은 도메인 간 호출이 허용되어야 하는지, 클라이언트를 어떻게 인증할지, XML과 JSON 데이터 형식 간의 변환을 투명하게 처리할지, 특정 IP 범위에서 들어오는 호출을 제한할지,사용 할당량, 호출 속도를 제한할지 등과 같은 측면을 제어합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-446">Policies govern aspects such as whether cross-domain calls should be allowed, how to authenticate clients, whether to convert between XML and JSON data formats transparently, whether to restrict calls from a given IP range, usage quotas, and whether to limit the call rate.</span></span> <span data-ttu-id="6f4c2-447">정책은 전체 프로젝트 포괄적으로 적용되거나 제품에 포함된 단일 Web API에 적용되거나 Web API에 포함된 개별 작업에 대해 적용될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-447">Policies can be applied globally across the entire product, for a single web API in a product, or for individual operations in a web API.</span></span>

<span data-ttu-id="6f4c2-448">자세한 내용은 [API Management 설명서](/azure/api-management/)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-448">For more information, see the [API Management Documentation](/azure/api-management/).</span></span> 

> [!TIP]
> <span data-ttu-id="6f4c2-449">Azure에는 장애 조치(failover) 및 부하 분산을 구현하고, 지리적으로 다른 위치에서 호스팅되는 웹 사이트의 복수 인스턴스에 대해 대기 시간을 줄일 수 있도록 하는 Azure Traffic Manager가 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-449">Azure provides the Azure Traffic Manager which enables you to implement failover and load-balancing, and reduce latency across multiple instances of a web site hosted in different geographic locations.</span></span> <span data-ttu-id="6f4c2-450">Azure Traffic Manager를 API Management 서비스와 결합하여 사용할 수 있습니다. API Management 서비스는 Azure Traffic Manager를 통해 웹 사이트의 인스턴스로 요청을 라우팅할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-450">You can use Azure Traffic Manager in conjunction with the API Management Service; the API Management Service can route requests to instances of a web site through Azure Traffic Manager.</span></span>  <span data-ttu-id="6f4c2-451">자세한 내용은 [Traffic Manager 라우팅 방법](/azure/traffic-manager/traffic-manager-routing-methods/)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-451">For more information, see [Traffic Manager routing Methods](/azure/traffic-manager/traffic-manager-routing-methods/).</span></span>
>
> <span data-ttu-id="6f4c2-452">이런 구조에서 웹 사이트에 사용자 지정 DNS 이름을 사용하면 Azure Traffic Manager 웹 사이트의 DNS 이름을 포인트하도록 각 웹 사이트에 대해 적절한 CNAME 레코드를 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-452">In this structure, if you are using custom DNS names for your web sites, you should configure the appropriate CNAME record for each web site to point to the DNS name of the Azure Traffic Manager web site.</span></span>
>

## <a name="supporting-client-side-developers"></a><span data-ttu-id="6f4c2-453">클라이언트 쪽 개발자 지원</span><span class="sxs-lookup"><span data-stu-id="6f4c2-453">Supporting client-side developers</span></span>
<span data-ttu-id="6f4c2-454">클라이언트 응용 프로그램을 구축하는 개발자들에게는 일반적으로 Web API 액세스 방법, 웹 서비스와 클라이언트 응용 프로그램 간의 다양한 요청과 응답을 명시하는 매개 변수, 데이터 유형, 반환 유형 및 반환 코드에 관한 문서가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-454">Developers constructing client applications typically require information on how to access the web API, and documentation concerning the parameters, data types, return types, and return codes that describe the different requests and responses between the web service and the client application.</span></span>

### <a name="document-the-rest-operations-for-a-web-api"></a><span data-ttu-id="6f4c2-455">Web API에 대한 REST 작업 문서화</span><span class="sxs-lookup"><span data-stu-id="6f4c2-455">Document the REST operations for a web API</span></span>
<span data-ttu-id="6f4c2-456">Azure API Management 서비스는 Web API에 의해 노출되는 REST 작업을 설명하는 개발자 포털을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-456">The Azure API Management Service includes a developer portal that describes the REST operations exposed by a web API.</span></span> <span data-ttu-id="6f4c2-457">제품이 게시되면 포털에 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-457">When a product has been published it appears on this portal.</span></span> <span data-ttu-id="6f4c2-458">개발자는 이 포털을 사용하여 액세스 등록을 할 수 있습니다. 그 후 관리자는 요청을 수락하거나 거부할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-458">Developers can use this portal to sign up for access; the administrator can then approve or deny the request.</span></span> <span data-ttu-id="6f4c2-459">승인된 개발자에게는 그들이 개발하는 클라이언트 응용 프로그램의 호출을 인증하는데 사용되는 구독 키가 할당됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-459">If the developer is approved, they are assigned a subscription key that is used to authenticate calls from the client applications that they develop.</span></span> <span data-ttu-id="6f4c2-460">이 키는 각각의 Web API에 대해 제공되어야 하며 그렇지 않으면 거부됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-460">This key must be provided with each web API call otherwise it will be rejected.</span></span>

<span data-ttu-id="6f4c2-461">이 포털에는 다음과 같은 내용도 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-461">This portal also provides:</span></span>

* <span data-ttu-id="6f4c2-462">제품이 노출하는 작업 목록, 필요한 매개 변수, 반환될 수 있는 다양한 응답 등을 을 포함하는 제품에 대한 문서.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-462">Documentation for the product, listing the operations that it exposes, the parameters required, and the different responses that can be returned.</span></span> <span data-ttu-id="6f4c2-463">이 정보는 Microsoft Azure API Management 서비스를 사용한 Web API 게시 섹션의 3단계에 제공되는 세부 내용으로부터 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-463">Note that this information is generated from the details provided in step 3 in the list in the Publishing a web API by using the Microsoft Azure API Management Service section.</span></span>
* <span data-ttu-id="6f4c2-464">JavaScript, C#, Java, Ruby, Python 및 PHP를 비롯한 언어로부터 작업을 호출하는 방법을 보여주는 코드 조각.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-464">Code snippets that show how to invoke operations from several languages, including JavaScript, C#, Java, Ruby, Python, and PHP.</span></span>
* <span data-ttu-id="6f4c2-465">개발자가 제품에 포함된 각각의 작업을 테스트하기 위해 HTTP 요청을 보내고 결과를 볼 수 있게 하는 개발자 콘솔.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-465">A developers' console that enables a developer to send an HTTP request to test each operation in the product and view the results.</span></span>
* <span data-ttu-id="6f4c2-466">개발자가 발견한 이슈나 문제를 보고할 수 있는 페이지.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-466">A page where the developer can report any issues or problems found.</span></span>

<span data-ttu-id="6f4c2-467">Azure 관리 포털에서 개발자는 스타일이나 레이아웃을 회사의 브랜딩에 맞게 변경하기 위하여 개발자 포털을 사용자 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-467">The Azure Management portal enables you to customize the developer portal to change the styling and layout to match the branding of your organization.</span></span>

### <a name="implement-a-client-sdk"></a><span data-ttu-id="6f4c2-468">클라이언트 SDK 구현</span><span class="sxs-lookup"><span data-stu-id="6f4c2-468">Implement a client SDK</span></span>
<span data-ttu-id="6f4c2-469">Web API를 액세스하기 위한 REST 요청을 호출하는 클라이언트 응용 프로그램을 빌드하려면 각각의 요청을 구성하여 적절한 형식을 갖추도록 하고, 웹 서비스를 호스팅하는 서버에 요청을 보내고, 응답의 구문을 해석하여 요청이 성공했는지 실패했는지를 파악하고 반환된 데이터를 추출하도록 하는 등의 코드를 상당히 많이 작성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-469">Building a client application that invokes REST requests to access a web API requires writing a significant amount of code to construct each request and format it appropriately, send the request to the server hosting the web service, and parse the response to work out whether the request succeeded or failed and extract any data returned.</span></span> <span data-ttu-id="6f4c2-470">클라이언트 응용 프로그램에 대해 이러한 염려를 해결하기 위하여 기능적인 메서드 세트 내부에 하위 수준의 세부 내용을 요약하고 REST 인터페이스를 요약해 놓은 SDK를 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-470">To insulate the client application from these concerns, you can provide an SDK that wraps the REST interface and abstracts these low-level details inside a more functional set of methods.</span></span> <span data-ttu-id="6f4c2-471">클라이언트 응용 프로그램은 이런 메서드를 사용하여 호출을 REST 요청으로 투명하게 변환한 후 응답을 다시 메서드 반환 값으로 변환합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-471">A client application uses these methods, which transparently convert calls into REST requests and then convert the responses back into method return values.</span></span> <span data-ttu-id="6f4c2-472">이것은 Azure SDK를 비롯한 많은 서비스에서 구현되는 일반적인 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-472">This is a common technique that is implemented by many services, including the Azure SDK.</span></span>

<span data-ttu-id="6f4c2-473">클라이언트쪽 SDK 생성은 일관적으로 구현되어야 하고 신중한 테스트를 거쳐야 하기 때문에 상당한 작업입니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-473">Creating a client-side SDK is a considerable undertaking as it has to be implemented consistently and tested carefully.</span></span> <span data-ttu-id="6f4c2-474">하지만, 이런 프로세스의 대부분은 기계적으로 생성될 수 있으며, 많은 공급 업체가 이런 작업을 자동화할 수 있는 도구를 제공하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-474">However, much of this process can be made mechanical, and many vendors supply tools that can automate many of these tasks.</span></span>

## <a name="monitoring-a-web-api"></a><span data-ttu-id="6f4c2-475">Web API 모니터링</span><span class="sxs-lookup"><span data-stu-id="6f4c2-475">Monitoring a web API</span></span>
<span data-ttu-id="6f4c2-476">Web API를 게시하고 배포한 방법에 따라서 Web API를 직접 모니터링하거나 API Management 서비스를 통과하는 트래픽을 분석하여 사용 정보와 상태 정보를 모을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-476">Depending on how you have published and deployed your web API you can monitor the web API directly, or you can gather usage and health information by analyzing the traffic that passes through the API Management service.</span></span>

### <a name="monitoring-a-web-api-directly"></a><span data-ttu-id="6f4c2-477">Web API 직접 모니터링</span><span class="sxs-lookup"><span data-stu-id="6f4c2-477">Monitoring a web API directly</span></span>
<span data-ttu-id="6f4c2-478">ASP.NET Web API 템플릿(Azure 클라우드 서비스의 Web API 프로젝트 또는 웹 역할로) 또는 Visual Studio 2013을 사용하여 Web API를 구현한 경우 ASP.NET Application Insights를 사용하여 가용성, 성능 및 사용 현황 데이터를 수집할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-478">If you have implemented your web API by using the ASP.NET Web API template (either as a Web API project or as a Web role in an Azure cloud service) and Visual Studio 2013, you can gather availability, performance, and usage data by using ASP.NET Application Insights.</span></span> <span data-ttu-id="6f4c2-479">Application Insights는 Web API가 클라우드에 배포될 때 요청과 응답에 대한 정보를 투명하게 추적하고 기록하는 패키지입니다. 패키지를 설치하고 구성한 후에는 사용을 위해 Web API의 코드를 수정할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-479">Application Insights is a package that transparently tracks and records information about requests and responses when the web API is deployed to the cloud; once the package is installed and configured, you don't need to amend any code in your web API to use it.</span></span> <span data-ttu-id="6f4c2-480">Azure 웹 사이트에 Web API를 배포하는 경우 모든 트래픽은 검사되며 다음과 같은 통계 자료가 수집됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-480">When you deploy the web API to an Azure web site, all traffic is examined and the following statistics are gathered:</span></span>

* <span data-ttu-id="6f4c2-481">서버 응답 시간.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-481">Server response time.</span></span>
* <span data-ttu-id="6f4c2-482">서버 요청의 수 및 각 요청의 세부 내용.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-482">Number of server requests and the details of each request.</span></span>
* <span data-ttu-id="6f4c2-483">평균 응답 시간 기준 가장 느린 요청.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-483">The top slowest requests in terms of average response time.</span></span>
* <span data-ttu-id="6f4c2-484">실패한 요청에 대한 세부 정보.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-484">The details of any failed requests.</span></span>
* <span data-ttu-id="6f4c2-485">다른 브라우저 및 사용자 에이전트에 의해 시작된 세션의 수.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-485">The number of sessions initiated by different browsers and user agents.</span></span>
* <span data-ttu-id="6f4c2-486">가장 많이 조회된 페이지(Web API보다는 주로 웹 응용 프로그램에 유용함).</span><span class="sxs-lookup"><span data-stu-id="6f4c2-486">The most frequently viewed pages (primarily useful for web applications rather than web APIs).</span></span>
* <span data-ttu-id="6f4c2-487">Web API에 액세스하는 다른 사용자 역할.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-487">The different user roles accessing the web API.</span></span>

<span data-ttu-id="6f4c2-488">이런 데이터를 Azure 관리 포털에서 실시간으로 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-488">You can view this data in real time from the Azure Management portal.</span></span> <span data-ttu-id="6f4c2-489">Web API 상태를 모니터링하는 웹 테스트를 생성할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-489">You can also create webtests that monitor the health of the web API.</span></span> <span data-ttu-id="6f4c2-490">웹 테스트는 Web API의 URI에 주기적으로 요청을 보내고 응답을 캡처합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-490">A webtest sends a periodic request to a specified URI in the web API and captures the response.</span></span> <span data-ttu-id="6f4c2-491">성공적인 응답의 정의(예: HTTP 상태 코드 200)를 명시할 수 있고, 요청이 이런 응답을 반환하지 않으면 관리자에게 경고를 보내도록 준비할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-491">You can specify the definition of a successful response (such as HTTP status code 200), and if the request does not return this response you can arrange for an alert to be sent to an administrator.</span></span> <span data-ttu-id="6f4c2-492">필요한 경우 관리자는 Web API에 오류가 발생하면 Web API를 호스팅하는 서버를 재시작할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-492">If necessary, the administrator can restart the server hosting the web API if it has failed.</span></span>

<span data-ttu-id="6f4c2-493">자세한 내용은 [Application Insights - ASP.NET 시작](/azure/application-insights/app-insights-asp-net/)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-493">For more information, see [Application Insights - Get started with ASP.NET](/azure/application-insights/app-insights-asp-net/).</span></span>

### <a name="monitoring-a-web-api-through-the-api-management-service"></a><span data-ttu-id="6f4c2-494">API Management 서비스를 통한 Web API 모니터링</span><span class="sxs-lookup"><span data-stu-id="6f4c2-494">Monitoring a web API through the API Management Service</span></span>
<span data-ttu-id="6f4c2-495">API Management 서비스를 사용하여 Web API를 게시한 경우 Azure 관리 포털의 API Management 페이지에 서비스의 전반적인 성능을 볼 수 있는 대시보드가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-495">If you have published your web API by using the API Management service, the API Management page on the Azure Management portal contains a dashboard that enables you to view the overall performance of the service.</span></span> <span data-ttu-id="6f4c2-496">분석 페이지에서는 제품이 어떻게 사용되는지를 자세히 드릴다운할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-496">The Analytics page enables you to drill down into the details of how the product is being used.</span></span> <span data-ttu-id="6f4c2-497">이 페이지는 다음과 같은 탭을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-497">This page contains the following tabs:</span></span>

* <span data-ttu-id="6f4c2-498">**사용 현황**.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-498">**Usage**.</span></span> <span data-ttu-id="6f4c2-499">이 탭은 실행된 API 호출의 수와 시간 별로 호출을 처리하는데 사용된 대역폭에 대한 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-499">This tab provides information about the number of API calls made and the bandwidth used to handle these calls over time.</span></span> <span data-ttu-id="6f4c2-500">제품, API, 작업 별로 사용 현황 세부 내용을 필터링할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-500">You can filter usage details by product, API, and operation.</span></span>
* <span data-ttu-id="6f4c2-501">**상태**.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-501">**Health**.</span></span> <span data-ttu-id="6f4c2-502">이 탭에서는 API 요청의 결과(반환된 HTTP 상태 코드), 캐시 정책의 효율성, API 응답 시간, 서비스 응답 시간을 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-502">This tab enables you to view the outcome of API requests (the HTTP status codes returned), the effectiveness of the caching policy, the API response time, and the service response time.</span></span> <span data-ttu-id="6f4c2-503">제품, API, 작업 별로 상태 데이터를 필터링할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-503">Again, you can filter health data by product, API, and operation.</span></span>
* <span data-ttu-id="6f4c2-504">**활동**.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-504">**Activity**.</span></span> <span data-ttu-id="6f4c2-505">이 탭에는 성공적인 호출, 실패한 호출, 차단된 호출의 수와 평균 응답 시간 그리고 각 제품, Web API 및 작업에 대한 응답 시간을 요약한 텍스트가 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-505">This tab provides a text summary of the numbers of successful calls, failed calls, blocked calls, average response time, and response times for each product, web API, and operation.</span></span> <span data-ttu-id="6f4c2-506">이 페이지에는 각 개발자가 생성한 호출의 수가 나열되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-506">This page also lists the number of calls made by each developer.</span></span>
* <span data-ttu-id="6f4c2-507">**개요**.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-507">**At a glance**.</span></span> <span data-ttu-id="6f4c2-508">이 탭에는 최다 API 호출, 제품, Web API의 생성에 책임이 있는 개발자 그리고 이런 호출을 수신하는 작업을 비롯한 성능 데이터에 대한 요약 정보가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-508">This tab displays a summary of the performance data, including the developers responsible for making the most API calls, and the products, web APIs, and operations that received these calls.</span></span>

<span data-ttu-id="6f4c2-509">이 정보를 사용하여 병목 현상을 유발하는 Web API나 작업이 있는지 호스트 환경을 축소/확장할 필요가 있는 서버를 더 추가할 필요가 있는지를 판단할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-509">You can use this information to determine whether a particular web API or operation is causing a bottleneck, and if necessary scale the host environment and add more servers.</span></span> <span data-ttu-id="6f4c2-510">하나 또는 그 이상의 응용 프로그램이 균형이 맞지 않는 양의 리소스를 사용하고 있는지를 확인하고 할당량을 설정하여 호출 속도를 제한하는 적절한 정책을 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-510">You can also ascertain whether one or more applications are using a disproportionate volume of resources and apply the appropriate policies to set quotas and limit call rates.</span></span>

> [!NOTE]
> <span data-ttu-id="6f4c2-511">게시된 제품의 세부 정보를 변경할 수 있고 변경 내용은 즉시 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-511">You can change the details for a published product, and the changes are applied immediately.</span></span> <span data-ttu-id="6f4c2-512">예를 들면, Web API를 포함하는 제품을 다시 게시할 필요 없이Web API의 작업을 추가하거나 삭제할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-512">For example, you can add or remove an operation from a web API without requiring that you republish the product that contains the web API.</span></span>
>
>

## <a name="more-information"></a><span data-ttu-id="6f4c2-513">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="6f4c2-513">More information</span></span>
* <span data-ttu-id="6f4c2-514">[ASP.NET Web API OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api)는 ASP.NET을 사용하여 OData 웹을 구현하는 자세한 정보와 예제를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-514">[ASP.NET Web API OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) contains examples and further information on implementing an OData web API by using ASP.NET.</span></span>
* <span data-ttu-id="6f4c2-515">Microsoft 웹 사이트의 [Introducing Batch Support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) (Web API 및 Web API OData의 배치 지원 소개) 페이지는 OData를 사용하여 Web API에 배치 작업을 구현하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-515">[Introducing Batch Support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) describes how to implement batch operations in a web API by using OData.</span></span>
* <span data-ttu-id="6f4c2-516">Jonathan Oliver의 블로그에 있는 [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/)에서는 idempotency 개요 및 데이터 관리 작업과 관련성을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-516">[Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog provides an overview of idempotency and how it relates to data management operations.</span></span>
* <span data-ttu-id="6f4c2-517">W3C 웹 사이트의 [상태 코드 정의](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)에는 HTTP 상태 코드의 전체 목록 및 해당 설명이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-517">[Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) on the W3C website contains a full list of HTTP status codes and their descriptions.</span></span>
* <span data-ttu-id="6f4c2-518">[WebJobs를 사용하여 배경 작업 실행](/azure/app-service-web/web-sites-create-web-jobs/)에서는 배경 작업을 실행하기 위해 WebJobs를 사용하는 방법에 대한 정보 및 예제를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-518">[Run background tasks with WebJobs](/azure/app-service-web/web-sites-create-web-jobs/) provides information and examples on using WebJobs to perform background operations.</span></span>
* <span data-ttu-id="6f4c2-519">[Azure Notification Hubs 알릴 사용자](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/)는 Azure 알림 허브를 사용하여 클라이언트 응용 프로그램에 비동기 응답을 푸시하는 방법을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-519">[Azure Notification Hubs Notify Users](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) shows how to use an Azure Notification Hub to push asynchronous responses to client applications.</span></span>
* <span data-ttu-id="6f4c2-520">[API Management](https://azure.microsoft.com/services/api-management/)는 Web API에 제어 및 보안 액세스를 제공하는 제품을 게시하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-520">[API Management](https://azure.microsoft.com/services/api-management/) describes how to publish a product that provides controlled and secure access to a web API.</span></span>
* <span data-ttu-id="6f4c2-521">[Azure API Management REST API 참조](https://msdn.microsoft.com/library/azure/dn776326.aspx)는 API Management REST API를 사용하여 사용자 지정 관리 응용 프로그램을 빌드하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-521">[Azure API Management REST API Reference](https://msdn.microsoft.com/library/azure/dn776326.aspx) describes how to use the API Management REST API to build custom management applications.</span></span>
* <span data-ttu-id="6f4c2-522">[Traffic Manager 라우팅 메서드](/azure/traffic-manager/traffic-manager-routing-methods/)는 Web API를 호스팅하는 웹 사이트의 복수 인스턴스에 대한 부하 분산 요청에 Azure Traffic Manager를 사용하는 방법을 요약합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-522">[Traffic Manager Routing Methods](/azure/traffic-manager/traffic-manager-routing-methods/) summarizes how Azure Traffic Manager can be used to load-balance requests across multiple instances of a website hosting a web API.</span></span>
* <span data-ttu-id="6f4c2-523">[Application Insights - ASP.NET 시작](/azure/application-insights/app-insights-asp-net/)은 ASP.NET Web API 프로젝트에서 Application Insights를 설치하고 구성하는 방법에 대한 자세한 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="6f4c2-523">[Application Insights - Get started with ASP.NET](/azure/application-insights/app-insights-asp-net/) provides detailed information on installing and configuring Application Insights in an ASP.NET Web API project.</span></span>


<!-- links -->

[api-design]: ./api-design.md
