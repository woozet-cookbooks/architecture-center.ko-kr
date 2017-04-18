---
title: API implementation guidance
description: Guidance upon how to implement an API.
author: dragon119
ms.service: guidance
ms.topic: article
ms.date: 07/13/2016
ms.author: pnp

pnp.series.title: Best Practices
---
# API 구현
[!INCLUDE [header](../_includes/header.md)]

신중하게 설계된 RESTful web API는 클라이언트 응용 프로그램에 액세스 가능한 리소스, 관계, 탐색 스키마를 정의합니다.  웹 API를 구현 및 배치할 때 웹 API를 호스팅하는 환경의 물리적 요구사항과 데이터의 논리적 구조보다는 API가 구성되는 방식을 고려해야 합니다. 본 지침은 웹 API를 구현하고 클라이언트 응용 프로그램에서 사용할 수 있도록 게시하는 모범 사례에 초점을 맞춥니다. 보안 문제는 API 보안 지침 문서에서 별도로 설명합니다. API 설계 지침 문서에서 웹 API 설계에 대한 자세한 정보를 확인할 수 있습니다. 

## RESTful web API 구현 시 고려사항
다음 섹션에서는 RESTful web API 구축을 위한 ASP.NET Web API 템플릿 사용의 모범 사례를 설명합니다. 웹 API 템플릿 사용에 관한 자세한 정보는 Microsoft 웹 사이트의 [ASP.NET Web API 살펴보기](http://www.asp.net/web-api) 페이지를 확인하십시오.

## 요청 라우팅 구현 시 고려사항
ASP.NET Web API를 사용해 구현되는 서비스의 경우 각각의 요청은 *controller(컨트롤러)* 클래스의 메서드로 라우팅됩니다. Web API 프레임워크는 라우팅을 구현하기 위해 두 가지의 기본 옵션인 *convention-based(규칙 기반)* 라우팅 및 *attribute-based(특성 기반)* 라우팅을 제공합니다. 웹 API에서 요청을 라우팅할 최선의 방법을 결정할 때 다음 사항을 고려하십시오. 

* **약속 기반 라우팅의 한계 및 요구 사항을 이해하십시오.**.

    기본적으로 Web API 프레임워크는 규칙 기반 라우팅을 사용합니다. Web API 프레임워크는 다음 항목이 포함된 초기 라우팅 테이블을 생성합니다.

    ```C#
    config.Routes.MapHttpRoute(
          name: "DefaultApi",
          routeTemplate: "api/{controller}/{id}",
          defaults: new { id = RouteParameter.Optional }
    );
    ```

    경로는 일반적이고 *api* 와 같은 리터럴 및 *{controller}* 및 *{id}* 와 같은 변수로 구성될 수 있습니다. 규칙 기반 라우팅을 통해 경로의 일부 요소를 선택적으로 만들 수 있습니다. Web API 프레임워크는 요청의 HTTP 메서드와 API 메서드 이름의 최초 부분을 일치시킨 다음 선택적 매개 변수를 일치시켜 컨트롤러에서 어떤 메서드를 호출할지를 결정합니다. 예를 들어, *orders(주문)* 라는 이름의 컨트롤러에 메서드 *GetAllOrders()* 또는 *GetOrderByInt(int id)* 가 포함되어 있는 경우 GET 요청 *http://www.adventure-works.com/api/orders/* 은 메서드 *GetAlllOrders()* 로 지정되고 GET 요청 *http://www.adventure-works.com/api/orders/99* 은 메서드 *GetOrderByInt(int id)* 로 라우팅됩니다. 컨트롤러에 접두사 Get으로 시작하고 사용 가능한, 일치하는 메서드가 없는 경우 Web API 프레임워크는 HTTP 405(메서드 허용 안 함) 메시지로 회신합니다. 또한, 라우팅 테이블에 지정된 매개 변수의 이름(id)은 *GetOrderById* 메서드에 대한 매개 변수의 이름과 동일해야 합니다. 동일하지 않은 경우 Web API 프레임워크는 HTTP 404(찾을 수 없음) 응답으로 회신합니다.

    동일한 규칙이 POST, PUT, DELETE HTTP 요청에 적용됩니다. 주문 101의 세부 정보를 업데이트하는 PUT 요청은 URI *http://www.adventure-works.com/api/orders/101* 로 지정되고, 메시지 본문에는 주문의 새로운 세부 정보가 포함되며, 이 정보는 접두사 *Put*으로 시작하는 이름(예:*PutOrder*)을 포함해 주문 컨트롤러의 메서드에 매개 변수로 전달됩니다.

    기본 라우팅 테이블은 RESTful web API에서 하위 리소스를 참조하는 요청(예:*http://www.adventure-works.com/api/customers/1/orders*(고객 1이 한 모든 주문의 세부 정보 검색))을 일치시키지 않습니다. 이러한 경우를 처리하기 위해 라우팅 테이블에 사용자 지정 경로를 추가할 수 있습니다.

    ```C#
    config.Routes.MapHttpRoute(
        name: "CustomerOrdersRoute",
        routeTemplate: "api/customers/{custId}/orders",
        defaults: new { controller="Customers", action="GetOrdersForCustomer" })
    );
    ```

    이 경로는 URI와 일치하는 요청을 *Customers* 컨트롤러의 *GetOrdersForCustomer* 메서드에 지정합니다. 이 메서드는 *custI*라는 이름의 단일 매개 변수를 가져와야 합니다.

    ```C#
    public class CustomersController : ApiController
    {
        ...
        public IEnumerable<Order> GetOrdersForCustomer(int custId)
        {
            // Find orders for the specified customer
            var orders = ...
            return orders;
        }
        ...
    }
    ```

  > [!팁]
  > 가능한 한 기본 라우팅을 활용하고 복잡한 사용자 지정 경로를 많이 정의하지 마십시오. 이로 인해 불안정해지고 (메서드를 컨트롤러에 추가하기는 매우 쉽지만 그로 인해 경로가 모호해질 수 있음) 성능이 감소합니다(라우팅 테이블이 클수록 어떤 경로가 주어진 URI와 일치하는지를 확인하기 위해 Web API 프레임워크가 더 많은 작업을 수행해야 함). API 및 경로를 단순하게 유지하십시오. 자세한 내용은 API 설계 지침의 리소스 주변에 웹 API 구성 섹션을 참조하십시오. 사용자 지정 경로를 정의해야 하는 경우 본 섹션의 뒷부분에 설명되어 있는 특성 기반 라우팅을 사용하는 것이 좋습니다.
  >
  >

    규칙 기반 라우팅에 대한 자세한 정보는 Microsoft 웹 사이트의 [ASP.NET Web API에서의 라우팅](http://www.asp.net/web-api/overview/web-api-routing-and-actions/routing-in-aspnet-web-api) 페이지를 참조하십시오.
* **라우팅의 모호성을 방지하십시오**.

    컨트롤러의 여러 메서드가 동일한 경로와 일치하는 경우 규칙 기반 라우팅으로 인해 경로가 모호해질 수 있습니다. 이러한 경우 Web API 프레임워크는 다음 텍스트가 포함된 HTTP 500(Internal Server Error(내부 서버 오류)) 응답 메시지로 응답합니다. "Multiple actions were found that match the request(요청과 일치하는 여러 개의 동작이 검색되었습니다.)"
* **특성 기반 라우팅을 선택하십시오**.

    특성 기반 라우팅은 컨트롤러에서 경로를 메서드에 연결하기 위한 대안을 제공합니다. 규칙 기반 라우팅의 패턴 일치 기능에 의존하는 것이 아니라 메서드에 연결되어야 하는 경로의 세부 정보와 함께 컨트롤러의 메서드에 명시적으로 주석을 달 수 있습니다. 이 접근 방식은 모호성을 제거하는 데 도움이 됩니다. 더욱이, 명시적 경로가 디자인 타임에서 정의되므로 이 접근 방식이 런타임에서 규칙 기반 라우팅보다 효율적입니다. 다음 코드는 *Route* 특성을 고객 컨트롤러의 메서드에 적용하는 방법을 보여줍니다. 이러한 메서드 또한 *HTTP GET* 요청에 응답해야 함을 표시하기 위해 HttpGet 특성을 사용합니다. 이 특성을 통해 규칙 기반 라우팅에서 예상하는 것이 아니라 편리한 이름 지정 체계를 사용해 메서드 이름을 지정할 수 있습니다. 또한 다른 유형의 HTTP 요청에 응답하는 메서드를 정의하기 위해 메서드에 *HttpPost*, *HttpPut*, *HttpDelete* 특성으로 주석을 달 수 있습니다.

    ```C#
    public class CustomersController : ApiController
    {
        ...
        [Route("api/customers/{id}")]
        [HttpGet]
        public Customer FindCustomerByID(int id)
        {
            // Find the matching customer
            var customer = ...
            return customer;
        }
        ...
        [Route("api/customers/{id}/orders")]
        [HttpGet]
        public IEnumerable<Order> FindOrdersForCustomer(int id)
        {
            // Find orders for the specified customer
            var orders = ...
            return orders;
        }
        ...
    }
    ```

    특성 기반 라우팅은 앞으로 코드를 유지해야 할 필요가 있는 개발자에게 설명서 역할을 하는 유용한 부수적인 효과도 있습니다. 어떤 메서드가 어느 경로에 속하는지 바로 알 수 있고 *HttpGet* 특성은 메서드가 응답하는 HTTP 요청 유형을 명확하게 보여줍니다.

    특성 기반 라우팅을 사용해 매개 변수가 일치되는 방법을 제한하는 제약 조건을 정의할 수 있습니다. 제약 조건은 매개 변수 유형을 지정할 수 있으며, 일부 경우에는 매개 변수 값의 수용 가능 범위를 지정할 수도 있습니다. 다음 예에서 *FindCustomerByID* 메서드에 대한 id 매개 변수는 음수가 아닌 정수여야 합니다. 응용 프로그램에서 음수 고객이 포함된 HTTP GET 요청을 제출하는 경우 Web API 프레임워크는 HTTP 450(Method Not Allowed(메서드 허용 안 함)) 메시지로 응답합니다.

    ```C#
    public class CustomersController : ApiController
    {
        ...
        [Route("api/customers/{id:int:min(0)}")]
        [HttpGet]
        public Customer FindCustomerByID(int id)
        {
            // Find the matching customer
            var customer = ...
            return customer;
        }
        ...
    }
    ```

    특성 기반 라우팅에 대한 자세한 정보는 Microsoft 웹 사이트에 나와 있는 [Web API 2에서의 특성 라우팅](http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2)을 참조하십시오.
* **경로에서 유니코드 문자를 지원하십시오**.

    GET 요청에서 리소스를 식별하는 데 사용되는 키는 문자열이 될 수 있습니다. 따라서 전역 응용 프로그램에서 영어가 아닌 문자가 포함된 URI를 지원해야 할 수도 있습니다.
* **라우트되어서는 안 되는 메서드를 구별하십시오**.

    규칙 기반 라우팅을 사용하는 경우 메서드를 *NonAction* 특성으로 장식해 HTTP 동작에 해당하지 않는 메서드를 표시하십시오. 이는 일반적으로 컨트롤러 내 다른 메서드에서 사용하도록 정의되는 도우미 메서드에 적용되며, 이 특성은 이러한 메서드가 잘못된 HTTP 요청에 의해 일치되고 호출되는 것을 방지합니다.
* **하위 도메인에 API를 위치시키는 것에 따른 혜택과 단점을 고려하십시오**.

    기본적으로 ASP.NET web API는 API를 도메인의 */api* 디렉터리에 구성합니다(예: `http://www.adventure-works.com/api/orders`). 이 디렉터리는 동일한 호스트에서 표시하는 다른 서비스와 동일한 도메인에 위치합니다. 웹 API를 `http://api.adventure-works.com/orders`와 같은 URI를 사용해 별도의 호스트에서 실행되는 고유의 하위 도메인에 분할하는 것이 도움이 될 수 있습니다. 이러한 구분을 통해 *www.adventure-works.com* 도메인에서 실행되는 다른 웹 응용 프로그램이나 서비스에 영향을 미치지 않고 웹 API를 보다 효과적으로 파티션 및 확장/축소할 수 있습니다.

    그러나 웹 API를 다른 하위 도메인에 위치시킴으로 인해 보안 문제가 발생할 수 있습니다. 다른 어딘가에서 실행되는 웹 API를 호출하는 *www.adventure-works.com* 에서 호스팅 되는 웹 응용 프로그램 또는 서비스는 많은 웹 브라우저의 동일 원본 정책을 위반할 수 있습니다. 이러한 경우 호스트 간의 크로스-원본 리소스 공유(CORS)를 사용해야 합니다. 자세한 정보는 API 보안 지침 문서를 참조하십시오.

## 요청 처리 시 고려사항
클라이언트 응용 프로그램의 요청이 웹 API의 메서드에 성공적으로 라우트되면 가능한 효율적인 방법으로 요청을 처리해야 합니다. 요청을 처리할 코드를 구현할 때 다음 사항을 고려하십시오. 

* **GET, PUT, DELETE, HEAD, PATCH 동작은 idempotent가 되어야 합니다**.

    이러한 요청을 구현하는 코드는 파생 작업을 부과해서는 안 됩니다. 동일한 리소스에 대해 반복된 동일한 요청은 동일한 상태를 생성해야 합니다. 예를 들어 동일한 URI에 여러 번 DELETE 요청을 전송하면 후속 DELETE 요청이 상태 코드 404(Not Found(찾을 수 없음))를 반환하는 동안 응답 메시지의 HTTP 상태 코드가 다를 수 있지만(첫 번째 DELETE 요청은 상태 코드 204(No Content(콘텐츠 없음))을 반환할 수 있음), 동일한 효과를 가져와야 합니다.

> [!참고]
> 조나단 올리버(Jonathan Oliver)의 블로그에 게재된 [Idempotency 패턴](http://blog.jonathanoliver.com/idempotency-patterns/)에 관한 기사는 Idempotency 개요 및 Idempotency가 데이터 작업 관리와 어떻게 관련되는지를 설명합니다. 
>
>

* **새로운 리소스를 생성하는 POST 동작은 관련 없는 파생 작업을 부과하지 않고 새로운 리소스를 생성해야 합니다**.

    POST 요청의 목적이 새로운 리소스를 생성하는 것이라면 요청의 영향은 새로운 리소스 (및 몇 가지 연결이 관련되어 있는 경우 직접적으로 관련된 리소스)로 제한되어야 합니다. 예를 들어, Ecommerce 시스템에서 고객에 대한 새로운 주문을 생성하는 POST 요청은 재고 수준을 수정하고 결제 정보를 생성할 수 있지만, 주문과 직접적으로 관련되지 않은 정보를 수정하거나 전반적인 시스템 상태에 파생 작업을 부과해서는 안 됩니다.
* **수다스러운 POST, PUT, DELETE 작업을 구현하지 마십시오**.

    리소스 컬렉션에 대한 POST, PUT, DELETE 요청을 지원하십시오. POST 요청에는 여러 가지 새로운 리소스에 대한 세부 정보가 포함될 수 있고 세부 정보 모두를 동일한 컬렉션에 추가할 수 있고, PUT 요청은 컬렉션의 전체 리소스 세트를 대체할 수 있고, DELETE 요청은 전체 컬렉션을 제거할 수 있습니다.

    ASP.NET Web API 2에 포함된 OData 지원은 일괄 요청 기능을 제공한다는 점을 유념합니다. 클라이언트 응용 프로그램은 여러 가지 웹 API 요청을 패키지로 묶어 단일 HTTP 요청으로 서버에 전송할 수 있고 각 요청에 대한 회신이 포함된 단일 HTTP 응답을 수신합니다. 자세한 정보는 Microsoft 웹 페이지의 [웹 API 및 웹 API OData의 일괄 지원 소개](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) 페이지를 참조하십시오.
* **클라이언트 응용프로그램으로 다시 응답을 전송할 때 HTTP 프로토콜을 준수하십시오**.

    클라이언트가 결과를 처리하는 방법을 결정할 수 있으려면 웹 API는 올바른 HTTP 상태 코드가 포함된 메시지를 반환해야 하고, 클라이언트가 결과의 속성을 이해하려면 적절한 HTTP 헤더가 포함된 메시지를 반환해야 하고, 클라이언트가 결과를 구문 분석할 수 있으려면 적합하게 포맷된 본문이 포함된 메시지를 반환해야 합니다. ASP.NET Web API 템플릿을 사용하는 경우 HTTP POST 요청에 응답하는 메서드 구현을 위한 기본적인 전략은 다음 예에서 설명하는 것처럼 간단하게 새롭게 생성된 리소스 사본을 반환하는 것입니다.

    ```C#
    public class CustomersController : ApiController
    {
        ...
        [Route("api/customers")]
        [HttpPost]
        public Customer CreateNewCustomer(Customer customerDetails)
        {
            // Add the new customer to the repository
            // This method returns a customer with a unique ID allocated
            // by the repository
            var newCust = repository.Add(customerDetails);
            // Return the newly added customer
            return newCust;
        }
        ...
    }
    ```

    POST 작업이 성공하는 경우 Web API 프레임워크는 상태 코드 200(OK(확인)) 및 메시지 본문으로 고객 세부 정보가 포함된 HTTP 응답을 생성합니다. 그러나 이러한 경우 HTTP 프로토콜에 따르면, POST 작업은 상태 코드 201(Created(생성됨))을 반환해야 하고 응답 메시지에는 응답 메시지의 Location(위치) 헤더에 새롭게 생성된 리소스의 URI가 포함되어야 합니다.

    이러한 기능을 제공하려면 `IHttpActionResult` 인터페이스를 사용해 고유한 HTTP 응답 메시지를 반환하십시오. 이 접근 방식을 통해 다음 코드 예에서 확인할 수 있는 것처럼 HTTP 상태 코드, 응답 메시지의 헤더, 심지어는 응답 메시지 본문의 데이터 형식을 미세하게 제어할 수 있습니다. 이 버전의 `CreateNewCustomer` 메서드는 HTTP 프로토콜을 준수하는 클라이언트의 기대를 좀 더 철저하게 따릅니다. `ApiController` 클래스의 `Created` 메서드는 지정된 데이터에서 응답 메시지를 구성하고 결과에 Location(위치) 헤더를 추가합니다.

    ```C#
    public class CustomersController : ApiController
    {
        ...
        [Route("api/customers")]
        [HttpPost]
        public IHttpActionResult CreateNewCustomer(Customer customerDetails)
        {
            // Add the new customer to the repository
            var newCust = repository.Add(customerDetails);

            // Create a value for the Location header to be returned in the response
            // The URI should be the location of the customer including its ID,
            // such as http://adventure-works.com/api/customers/99
            var location = new Uri(...);

            // Return the HTTP 201 response,
            // including the location and the newly added customer
            return Created(location, newCust);
        }
        ...
    }
    ```
* **콘텐츠 협상을 지원하십시오**.

    응답 메시지의 본문에는 다양한 형식의 데이터가 포함될 수 있습니다. 예를 들어, HTTP GET 요청은 JSON 또는 XML 형식으로 데이터를 반환할 수 있습니다. 클라이언트가 요청을 제출하면 요청에는 클라이언트가 처리할 수 있는 데이터 형식을 지정하는 Accept(수락) 헤더가 포함될 수 있습니다. 이러한 형식은 미디어 유형으로 지정됩니다. 예를 들어, 이미지를 검색하는 GET 요청을 발행하는 클라이언트는 처리 가능한 미디어 유형(예: image/jpeg, image/gif, image/png)을 나열하는 Accept(수락) 헤더를 지정할 수 있습니다. 웹 API가 결과를 반환할 때 이러한 미디어 유형 중 하나를 사용해 데이터를 포맷해야 하고 응답의 Content-Type(콘텐츠 유형) 헤더에서 형식을 지정해야 합니다.

    클라이언트가 Accept(수락) 헤더를 지정하지 않는 경우 응답 본문에 대해 합리적인 기본 형식을 사용합니다. 가령 ASP.NET Web API 프레임워크는 텍스트 기반 데이터에 대해 JSON을 기본값으로 사용합니다.

  > [!참고]
  > ASP.NET Web API 프레임워크는 Accept(수락) 헤더의 일부 자동 검색을 수행하고 응답 메시지의 본문 데이터 유형을 근거로 처리합니다. 예를 들어 응답 메시지의 본문에 CLR(공용 언어 런타임) 개체가 포함되어 있는 경우 ASP.NET Web API는 클라이언트가 결과를 XML로 포맷해야 함을 표시하지 않는 한, 응답의 Content-Type(콘텐츠 유형) 헤더가 "application/json"로 설정된 상태에서 응답을 자동으로 JSON으로 포맷합니다. 결과를 XML로 포맷해야 함을 표시하는 경우 ASP.NET Web API 프레임워크는 응답을 XML로 포맷하고 응답의 Content-Type(콘텐츠 유형) 헤더를 "text/xml"로 설정합니다. 그러나 작업에 대한 구현 코드에서 명시적으로 다른 미디어 유형을 지정하는 Accept(수락) 헤더를 처리해야 합니다.
  >
  >
* **HATEOAS 스타일의 리소스 탐색 및 검색을 지원하기 위한 링크를 제공하십시오**.

    API 설계 지침에서는 HATEOAS 접근 방식을 따르면 클라이언트가 어떻게 초기 시작 지점에서 리소스를 탐색 및 검색할 수 있는지에 대해 설명합니다. 클라이언트가 리소스를 얻기 위한 HTTP GET 요청을 발행할 때 URI가 포함된 링크를 사용해 리소스를 시작 지점에서 탐색 및 검색할 수 있고, 응답에는 클라이언트 응용 프로그램이 직접적으로 관련된 리소스를 빠르게 찾을 수 있도록 하는 URI가 포함되어야 합니다. 예를 들어, E-commerce 솔루션을 지원하는 웹 API에서 고객은 많은 주문을 할 수 있습니다. 클라이언트 응용 프로그램이 고객에 대한 세부 정보를 검색할 때 응답에는 클라이언트 응용 프로그램이 이러한 주문을 검색할 수 있는 HTTP GET 요청을 전송하도록 지원하는 링크가 포함되어야 합니다. 뿐만 아니라 HATEOAS 스타일 링크는 각각 링크된 리소스가 해당 URI와 함께 각 요청 수행을 지원하는 다른 작업(POST, PUT, DELETE 등)을 설명해야 합니다. 이 접근 방식은 API 설계 지침 문서에 더 자세히 설명되어 있습니다.

    현재 HATEOAS 구현을 제어하는 표준은 없지만, 다음 예를 통해 한 가지 접근 방식을 설명합니다. 이 예에서 고객에 대한 세부 정보를 검색하는 HTTP GET 요청은 해당 고객에 대한 주문을 참조하는 HATEOAS 링크가 포함된 응답을 반환합니다.

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

    이 예에서 고객 데이터는 다음 코드 조각에서 확인할 수 있는 것처럼 Customer 클래스로 나타냅니다. HATEOAS 링크는 `Links` 컬렉션 속성에 보관됩니다.

    ```C#
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

    HTTP GET 작업은 저장소에서 고객 데이터를 검색하고 `Customer` 개체를 구성한 다음, `Links` 컬렉션을 채웁니다 . 결과는 JSON 응답 메시지로 포맷됩니다. 각각의 링크는 다음 필드로 구성됩니다.

  * 반환되는 개체와 링크를 통해 설명하는 개체 사이의 관계. 여기서 "self(셀프)"는 링크가 개체 자체를 다시 참조했음을 나타내며(상당수 개체 지향 언어에서 `this` 포인터와 유사), "orders(주문)"는 관련 주문 정보가 포함된 컬렉션의 이름을 의미합니다.
  * URI 양식의 링크를 통해 설명되는 개체에 대한 하이퍼링크(`Href`).
  * 이 URI로 전송될 수 있는 HTTP 요청 유형(`Action`).
  * 요청 유형에 따라 HTTP 요청에 제공되어야 하거나 응답으로 반환할 수 있는 데이터 형식(`Types`).

    HTTP 응답 예에 포함된 HATEOAS 링크는 클라이언트 응용 프로그램이 다음 작업을 수행할 수 있음을 나타냅니다.
  * 고객의 세부 정보를 (다시) 가져오기 위한 URI *http://adventure-works.com/customers/2* 에 대한 HTTP GET 요청. 데이터는 XML 또는 JSON으로 반환될 수 있습니다.
  * 고객의 세부 정보를 수정하기 위한 URI *http://adventure-works.com/customers/2* 에 대한 HTTP PUT 요청. 새로운 데이터는 x-www-form-urlencoded 형식으로 요청 메시지에 제공되어야 합니다.
  * 고객을 삭제하기 위한 URI *http://adventure-works.com/customers/2* 에 대한 HTTP DELETE 요청. 요청은 추가 정보를 기대하거나 응답 메시지 본문으로 데이터를 반환하지 않습니다.
  * 고객에 대한 모든 주문을 검색하기 위한 URI *http://adventure-works.com/customers/2/orders* 에 대한 HTTP GET 요청. 데이터는 XML 또는 JSON으로 반환될 수 있습니다.
  * 이 고객에 대한 새로운 주문을 생성하기 위한 URI *http://adventure-works.com/customers/2/orders* 에 대한 HTTP PUT 요청. 데이터는 x-www-form-urlencoded 형식으로 요청 메시지에 제공되어야 합니다.

## 예외 처리 시 고려사항
기본적으로 ASP.NET Web API 프레임워크에서 작업이 Catch되지 않은 예외를 발생시키는 경우 프레임워크는 HTTP 상태 코드 500(Internal Server Error(내부 서버 오류))이 포함된 응답 메시지를 반환합니다. 많은 경우에 가장 간단한 이 접근 방식은 고립된 상태에서 유용하지 않고, 예외의 원인을 확인하는 것을 어렵게 만듭니다. 따라서 다음 사항을 고려하여 예외를 처리하기 위한 보다 포괄적인 접근 방식을 채택해야 합니다. 

* **예외를 캡처하고 클라이언트에 의미 있는 응답을 반환하십시오**.

    HTTP 작업을 구현하는 코드는 Catch되지 않은 예외가 Web API 프레임워크에 적용되도록 두는 것이 아니라 포괄적인 예외 처리 방식을 제공해야 합니다. 예외로 인해 작업을 성공적으로 완료할 수 없다면 예외는 응답 메시지로 다시 전달될 수 있지만, 예외의 원인이 된 오류에 대한 의미 있는 설명을 포함해야 합니다. 또한, 간단하게 모든 상황에 대해 상태 코드 500을 반환하는 것이 아니라 예외에는 적절한 HTTP 상태 코드가 포함되어야 합니다. 예를 들어, 사용자 요청으로 제약 조건을 위반하는 데이터베이스 업데이트가 수행되는 경우(예: 처리되지 않은 주문이 있는 고객 삭제 시도) 상태 코드 409(Conflict(충돌)) 및 충돌에 대한 이유를 나타내는 메시지 본문을 반환해야 합니다. 일부 다른 조건으로 인해 요청이 수행 불가능한 경우 상태 코드 400(Bad Request(잘못된 요청))을 반환할 수 있습니다. W3C 웹사이트의 [상태 코드 정의](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)에서 HTTP 상태 코드 전체 목록을 확인할 수 있습니다.

    다음 코드는 다른 조건을 트래핑해 적절한 응답을 반환하는 예를 보여줍니다.

    ```C#
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

  > [!팁]
  > 웹 API를 해킹하려는 해커에게 유용한 정보를 포함해서는 안 됩니다. 자세한 정보는 Microsoft 웹 사이트에 나와 있는 [ASP.NET Web API에서의 예외 처리](http://www.asp.net/web-api/overview/error-handling/exception-handling) 페이지를 참조하십시오.
  >
  > [!참고]
  > 상당수 웹 서버가 에러 상태가 웹 API에 도달하기 전에 스스로 에러 상태를 트래핑합니다. 예를 들어 웹 사이트에 대한 인증을 구성했는데 사용자가 정확한 인증 정보를 제공하지 못하는 경우 웹 서버는 상태 코드 401(Unauthorized(권한이 없음))로 응답해야 합니다. 클라이언트가 인증되면 코드는 클라이언트가 요청한 리소스에 액세스할 수 있는지를 확인하기 위한 자체 확인을 수행할 수 있습니다. 권한 부여에 실패하면 상태 코드 403(Forbidden(사용할 수 없음))을 반환해야 합니다.
  >
  >
* **일관된 방식으로 예외를 처리하고 오류에 대한 정보를 로그하십시오**.

    일관된 방식으로 예외를 처리하려면 전체 웹 API 전반에서 전역 오류 처리 전략을 구현하는 것을 고려하십시오. `HttpResponseException` 예외가 아닌 처리되지 않은 예외를 컨트롤러가 발생시킬 때마다 실행되는 예외 필터를 생성해 전역 오류 처리 전략의 일부를 구현할 수 있습니다. 이 접근 방식은 Microsoft 웹 사이트 [ASP.NET Web API에서의 예외 처리](http://www.asp.net/web-api/overview/error-handling/exception-handling) 페이지에 설명되어 있습니다.

   그러나 다음과 같이 예외 필터가 예외를 캡처하지 않는 여러 상황이 있습니다.

  * 컨트롤러 생성자에서 발생한 예외.
  * 메시지 처리기에서 발생한 예외.
  * 라우팅 동안 발생한 예외.
  * 응답 메시지에 대한 콘텐츠를 직렬화하는 동안 발생한 예외

    이러한 경우를 처리하기 위해 더욱 사용자 지정된 접근 방식을 구현해야 합니다. 각각의 예외에 대한 모든 세부 정보를 캡처하는 오류 로깅을 포함시켜야 합니다. 이 오류 로그에는 웹을 통해 클라이언트가 액세스할 수 없는 한 자세한 정보를 포함할 수 있습니다. Microsoft 웹 사이트의 [웹 API 전역 오류 처리](http://www.asp.net/web-api/overview/error-handling/web-api-global-error-handling) 기사에서는 이 작업을 수행할 수 있는 단 한 가지 방법을 설명합니다.
* **클라이언트 쪽 오류와 서버 쪽 오류를 구별하십시오**.

    HTTP 프로토콜은 클라이언트 응용 프로그램 때문에 발생하는 오류(HTTP 4xx 상태 코드)와 서버 문제로 인해 발생하는 오류(HTTP 5xx 상태 코드)를 구별합니다. 오류 응답 메시지에서 이 규칙을 준수하고 있는지 확인하십시오.

<a name="considerations-for-optimizing"></a>

## 클라이언트 쪽 액세스 최적화 시 고려사항
웹 서버 및 클라이언트 응용 프로그램이 포함되는 것과 같은 분산 환경에서 주요 우려 사항 중 하나는 네트워크입니다. 네트워크는 특히 클라이언트 응용 프로그램이 자주 요청을 전송하거나 데이터를 수신하는 경우 상당한 병목 현상을 유발할 수 있습니다. 따라서 네트워크 전반을 흐르는 트래픽 양을 최소화하는 것을 목표로 삼아야 합니다. 데이터를 검색하고 유지할 코드를 구현할 때 다음 사항을 고려하십시오. 

* **클라이언트 쪽 캐싱을 지원하십시오**.

    HTTP 1.1 프로토콜은 클라이언트 및 중간 서버에서 캐싱을 지원하고 이를 통해 요청은 Cache-Control(캐시 제어) 헤더를 사용해 라우트됩니다. 클라이언트 응용 프로그램이 웹 API에 HTTP GET 요청을 전송하면 응답에는 응답 본문의 데이터가 클라이언트 또는 중간 서버에 의해 안전하게 캐시될 수 있는지를 표시하는 Cache-Control(캐시 제어) 헤더가 포함될 수 있습니다(이를 통해 요청이 라우팅 되고, 얼마 후 만료되고 유효하지 않은 요청으로 간주되는 시점을 알 수 있음). 다음 예는 HTTP GET 요청과 Cache-Control(캐시 제어) 헤더가 포함된 해당 응답을 보여줍니다.

    ```HTTP
    GET http://adventure-works.com/orders/2 HTTP/1.1
    ...
    ```

    ```HTTP
    HTTP/1.1 200 OK
    ...
    Cache-Control: max-age=600, private
    Content-Type: text/json; charset=utf-8
    Content-Length: ...
    {"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
    ```

    이 예에서 Cache-Control(캐시 제어) 헤더는 반환된 데이터가 600초 후에 만료되어야 하고, 단일 클라이언트에 대해서만 적합하고 다른 클라이언트가 사용하는 공유 캐시에 저장되어서는 안 된다고 지정합니다(*private(비공개)*입니다). Cache-Control(캐시 제어) 헤더는 *private(비공개)*이 아니라 *public(공개)*을 지정할 수 있고 이러한 경우 데이터는 공유 캐시에 저장되며, *no-store*로 지정하는 경우 데이터는 클라이언트에 의해 캐시되지 **않아야 합니다**. 다음 코드 예는 응답 메시지에서 Cache-Control(캐시 제어) 헤더를 구성하는 방법을 보여줍니다.

    ```C#
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

    이 코드는 `OkResultWithCaching`이라는 이름의 사용자 지정 `IHttpActionResult` 클래스를 사용합니다. 이 클래스를 통해 컨트롤러는 캐시 헤더 콘텐츠를 설정할 수 있습니다.

    ```C#
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
            HttpResponseMessage response = await base.ExecuteAsync(cancellationToken);

            response.Headers.CacheControl = this.CacheControlHeader;
            response.Headers.ETag = ETag;

            return response;
        }
    }
    ```

  > [!참고]
  > 또한, HTTP 프로토콜은 Cache-Control(캐시 제어) 헤더에 대한 no-cache 지시문을 정의합니다. 다소 혼란스럽지만, 이 지시문은 "do not cache(캐시 안 함)"가 아니라 "revalidate the cached information with the server before returning it(반환하기 전에 서버로 캐시된 정보 유효성 재검사)"을 의미하므로 데이터는 계속 캐시되지만 데이터가 여전히 최신의 것인지 데이터를 사용할 때마다 확인합니다.
  >
  >

    캐시 관리는 클라이언트 응용 프로그램 또는 중간 서버의 책임이지만 적절하게 구현된 경우 최근에 이미 검색된 데이터를 가져와야 할 필요성을 제거함으로써 대역폭을 저장하고 성능을 개선할 수 있습니다.

    Cache-Control(캐시 제어) 헤더의 *max-age* 값은 가이드일뿐 해당 데이터가 지정된 시간 동안 변경되지 않을 것을 보장하지 않습니다. 웹 API는 예상되는 데이터 휘발성에 따라 max-age(최대 기간)을 적절한 값으로 설정해야 합니다. 이 기간이 만료되면 클라이언트는 캐시에서 개체를 삭제해야 합니다.

  > [!참고]
  > 대부분의 현대식 웹 브라우저는 설명한 대로 적합한 Cache-Control(캐시 제어) 헤더를 요청에 추가하고 결과 헤더를 검사함으로써 클라이언트 쪽 캐싱을 지원합니다. 그러나 일부 이전 웹 브라우저는 쿼리 문자열이 포함된 URL에서 반환된 모든 값을 캐시하지 않습니다. 여기에서 논의하는 프로토콜을 기반으로 고유한 캐시 관리 전략을 구현하는 사용자 지정 클라이언트 응용 프로그램의 경우에는 일반적이지 않은 문제입니다.
  >
  > 일부 이전 프록시는 동일한 동작을 나타내고 쿼리 문자열이 포함된 URL을 기반으로 요청을 캐시하지 않을 수 있습니다. 이는 이러한 프록시를 통해 웹 서버에 연결하는 사용자 지정 클라이언트 응용 프로그램의 경우 문제가 될 수 있습니다.
  >
  >
* **쿼리 처리 최적화를 위한 ETag를 제공하십시오**.

    클라이언트 응용 프로그램이 개체를 검색하는 경우 응답 메시지에는 *ETag (엔터티 태그)*도 포함될 수 있습니다. Etag는 리소스 버전을 나타내는 불투명 문자열입니다. 리소스가 변경될 때마다 Etag도 수정됩니다. 이 Etag는 클라이언트 응용 프로그램에 의해 데이터의 일부로 캐시되어야 합니다. 다음 코드 예는 응답의 일부로 Etag를 HTTP GET 요청에 추가하는 방법을 보여줍니다. 이 코드는 개체를 식별하는 숫자 값을 생성하기 위해 개체의 `GetHashCode` 메서드를 사용합니다(필요한 경우 이 메서드를 다시 정의하고 MD5와 같은 알고리즘을 사용하여 고유한 해시를 생성할 수 있습니다.).

    ```C#
    public class OrdersController : ApiController
    {
        ...
        public IHttpActionResult FindOrderByID(int id)
        {
            // Find the matching order
            Order order = ...;
            ...

            var hashedOrder = order.GetHashCode();
            string hashedOrderEtag = String.Format("\"{0}\"", hashedOrder);
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

    웹 API에 의해 게시된 응답 메시지는 다음과 같습니다.

    ```HTTP
    HTTP/1.1 200 OK
    ...
    Cache-Control: max-age=600, private
    Content-Type: text/json; charset=utf-8
    ETag: "2147483648"
    Content-Length: ...
    {"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
    ```

  > [!팁]
  > 보안상의 이유로 중요한 데이터 또는 인증된(HTTPS) 연결을 통해 반환되는 데이터가 캐시되도록 하지 마십시오.
  >
  >

    클라이언트 응용 프로그램은 언제라도 동일한 리소스를 검색할 후속 GET 요청을 발행할 수 있고, 리소스가 변화하는 경우 (다른 Etag를 가지는 경우) 캐시된 버전은 삭제되고 새로운 버전이 캐시에 추가되어야 합니다. 리소스가 크고, 클라이언트에 다시 전송하려면 상당히 많은 양의 대역폭이 필요한 경우, 동일한 데이터를 가져오기 위한 반복된 요청은 비효율적일 수 있습니다. 이러한 문제를 해결하기 위해 HTTP 프로토콜은 웹 API에서 지원해야 할 GET 요청을 최적화하기 위한 다음과 같은 프로세스를 정의합니다.

  * 클라이언트는 If-None-Match HTTP 헤더에서 참조되는 현재 캐시된 리소스의 버전에 대한 Etag가 포함된 GET 요청을 구성합니다.

    ```HTTP
    GET http://adventure-works.com/orders/2 HTTP/1.1
    If-None-Match: "2147483648"
    ...
    ```
  * 웹 API에서 GET 작업은 요청한 데이터(위의 예에서 주문 2)에 대한 현재 ETag를 받아 If-None-Match 헤더에 있는 값과 비교합니다.
  * 요청한 데이터에 대한 현재 Etag가 요청에서 제공한 ETag와 일치하는 경우 리소스는 변경되지 않고 웹 API는 빈 메시지 본문과 상태 코드 304(Not Modified(수정되지 않음))가 포함된 HTTP 응답을 반환해야 합니다.
  * 요청한 데이터에 대한 현재 Etag가 요청에서 제공한 Etag와 일치하지 않는 경우 데이터는 변경되고 웹 API는 메시지 본문에 새로운 데이터와 상태 코드 200(OK(확인))이 포함된 HTTP응답을 반환해야 합니다.
  * 요청한 데이터가 더 이상 존재하지 않는 경우 웹 API는 상태 코드 404(Not Found(찾을 수 없음))가 포함된 HTTP 응답을 반환해야 합니다.
  * 클라이언트는 캐시를 유지하기 위해 상태 코드를 사용합니다. 데이터가 변경되지 않은 경우 (상태 코드 304) 개체는 캐시된 상태로 남아 있고 클라이언트 응용 프로그램은 계속 이 버전의 개체를 사용해야 합니다. 데이터가 변경된 경우 (상태 코드 200) 캐시된 개체는 삭제되고 새로운 개체가 삽입되어야 합니다. 데이터가 더 이상 사용 가능하지 않은 경우 (상태 코드 404) 해당 개체는 캐시에서 제거되어야 합니다.

    > [!참고]
    > 응답 헤더에 Cache-Control(캐시 제어) 헤더 no-store가 포함되어 있는 경우 해당 개체는 HTTP 상태 코드와 관계없이 캐시에서 항상 제거되어야 합니다.
    >
    >

    아래 코드는 If-None-Match 헤더를 지원하기 위해 확장된 `FindOrderByID` 메서드를 보여줍니다. If-None-Match 헤더가 생략된 경우 지정된 주문이 항상 검색된다는 점을 유념합니다.

    ```C#
    public class OrdersController : ApiController
    {
         ...
      [Route("api/orders/{id:int:min(0)}")]
      [HttpGet]
      public IHttpActionResult FindOrderById(int id)
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
              string hashedOrderEtag = String.Format("\"{0}\"", hashedOrder);

              // Create the Cache-Control and ETag headers for the response
              IHttpActionResult response = null;
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
                  String.Compare(nonMatchEtags.First().Tag, hashedOrderEtag) == 0)
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

    이 예에는 추가 사용자 지정 `IHttpActionResult` 클래스(이름: `EmptyResultWithCaching`)를 포함시켰습니다. 이 클래스는 간단하게 응답 본문이 포함되지 않은  `HttpResponseMessage` 개체 주변의 래퍼 역할을 합니다.

    ```C#
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

    > [!팁]
    > 이 예에서 데이터에 대한 Etag는 기본 데이터 원본에서 검색된 데이터를 해시함으로써 생성됩니다. Etag가 일부 다른 방식으로 컴퓨팅될 수 있으면 프로세스는 더욱 최적화될 수 있고 데이터는 변경된 경우 데이터 원본에서만 가져와야 합니다. 이 접근 방식은 데이터가 크거나 데이터 원본에 액세스하면 상당한 대기 시간이 발생하는 경우(예: 데이터 원본이 원격 데이터베이스인 경우) 매우 유용합니다.
    >
    >
* **낙관성 동시성을 지원하기 위해 Etag를 사용하십시오**.

    이전에 캐시된 데이터에 대한 업데이트를 사용하기 위해서는 HTTP 프로토콜이 낙관적 동시성 전략을 지원해야 합니다. 리소스를 가져와 캐시한 후 클라이언트 응용 프로그램에서 해당 리소스를 변경 또는 제거하기 위한 PUT 또는 DELETE 요청을 전송하는 경우, Etag를 참조하는 If-Match 헤더가 포함되어야 합니다. 웹 API는 이 정보를 사용해 리소스가 검색된 이후 다른 사용자에 의해 리소스가 이미 변경되었는지를 결정하고 다음과 같이 적절한 응답을 클라이언트 응용 프로그램에 다시 전송할 수 있습니다.

  * 클라이언트는 리소스에 대한 새로운 세부 정보 및 If-Match HTTP 헤더에서 참조된 현재 캐시된 리소스 버전에 대한 Etag가 포함된 PUT 요청을 구성합니다. 다음 예는 주문을 업데이트하는 PUT 요청을 보여줍니다.

    ```HTTP
    PUT http://adventure-works.com/orders/1 HTTP/1.1
    If-Match: "2282343857"
    Content-Type: application/x-www-form-urlencoded
    ...
    Date: Fri, 12 Sep 2014 09:18:37 GMT
    Content-Length: ...
    productID=3&quantity=5&orderValue=250
    ```
  * 웹 API에서 PUT 작업은 요청한 데이터(위의 예에서 주문 1)에 대한 현재 ETag를 받아 If-Match 헤더에 있는 값과 비교합니다.
  * 요청한 데이터에 대한 현재 Etag가 요청에서 제공한 ETag와 일치하는 경우 리소스는 변경되지 않고 웹 API는 업데이트를 수행하고 성공한 경우 HTTP 상태 코드 204(No Content(콘텐츠 없음))가 포함된 메시지를 반환해야 합니다. 응답에는 업데이트된 리소스 버전에 대한 Cache-Control(캐시 제어) 및 Etag 헤더가 포함될 수 있습니다. 응답은 새롭게 업데이트된 리소스의 URI를 참조하는 Location(위치) 헤더를 항상 포함해야 합니다.
  * 요청한 데이터에 대한 현재 Etag가 요청에서 제공한 Etag와 일치하지 않는 경우 데이터를 가져온 이후 다른 사용자에 의해 데이터가 변경된 것이고 웹 API는 빈 메시지 본문과 상태 코드 412(Precondition Failed(전제 조건이 실패함))가 포함된 HTTP 응답을 반환해야 합니다.
  * 업데이트할 리소스가 더 이상 존재하지 않는 경우 웹 API는 상태 코드 404(Not Found(찾을 수 없음))가 포함된 HTTP 응답을 반환해야 합니다.
  * 클라이언트는 캐시를 유지하기 위해 상태 코드와 응답 헤더를 사용합니다. 데이터가 업데이트된 경우(상태 코드 204) (Cach-Control(캐시 제어) 헤더가 no-store를 지정하지 않는 한) 개체는 캐시된 상태로 유지될 수 있지만 Etag는 업데이트되어야 합니다. 데이터가 변경된 다른 사용자에 의해 바뀌었거나(상태 코드 412) 검색할 수 없는 경우(상태 코드 404) 캐시된 개체를 삭제해야 합니다.

    다음 코드 예는 Order(주문) 컨트롤러에 대한 PUT 작업 구현을 보여줍니다.

    ```C#
    public class OrdersController : ApiController
    {
         ...
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
              string hashedOrderEtag = String.Format("\"{0}\"", hashedOrder);

              // Retrieve the If-Match header from the request (if it exists)
              var matchEtags = Request.Headers.IfMatch;

              // If there is an Etag in the If-Match header and
              // this etag matches that of the order just retrieved,
              // or if there is no etag, then update the Order
              if (((matchEtags.Count > 0 &&
                   String.Compare(matchEtags.First().Tag, hashedOrderEtag) == 0)) ||
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
                  hashedOrderEtag = String.Format("\"{0}\"", hashedOrder);
                  var eTag = new EntityTagHeaderValue(hashedOrderEtag);

                  var location = new Uri(string.Format("{0}/{1}/{2}", baseUri, Constants.ORDERS, id));
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

    > [!팁]
    > If-Match 헤더 사용이 완전히 선택적이고, 생략되어 있다면 웹 API는 항상 지정된 주문을 업데이트하려 시도하고, 다른 사용자에 의해 수행된 업데이트를 무조건 다시 정의할 가능성이 있습니다. 업데이트 손실로 인한 문제를 피하려면 If-Match 헤더를 항상 제공하십시오.
    >
    >

<a name="considerations-for-handling-large"></a>

## 큰 요청 및 응답 처리 시 고려사항
클라이언트 응용 프로그램은 수십 메가바이트 (또는 그 이상) 크기의 데이터를 전송 또는 수신하는 요청을 발행해야 하는 경우가 있습니다. 이러한 크기의 데이터가 전송되는 동안 대기할 때 클라이언트 응용 프로그램이 응답하지 않는 상태가 될 수 있습니다. 상당한 크기의 데이터가 포함되는 요청을 처리해야 하는 경우 다음 사항을 고려하십시오. 

* **큰 개체가 포함되는 요청 및 응답을 최적화하십시오**.

    일부 리소스는 큰 개체이거나 그래픽 이미지 또는 다른 유형의 바이너리 데이터와 같은 큰 필드를 포함할 수 있습니다. 웹 API는 이러한 리소스의 최적화된 업로드 및 다운로드를 위해 스트리밍을 지원해야 합니다.

    HTTP 프로토콜은 큰 데이터 개체를 클라이언트에 다시 스트리밍하기 위한 청크 분할 전송 인코딩 메커니즘을 제공합니다. 클라이언트가 큰 개체에 대한 HTTP GET 요청을 전송하는 경우 웹 API는 HTTP 연결을 통해 증분 *chunks(청크)*로 회신을 다시 전송할 수 있습니다. 회신 데이터의 길이는 처음부터 알 수 없으므로(생성될 수 있음) 웹 API를 호스팅하는 서버는 Transfer-Encoding(전송 인코딩): Chunked header rather than a Content-Length header(Content-Length 헤더가 아니라 Chunked 헤더)를 지정하는 각 청크가 포함된 응답 메시지를 전송해야 합니다. 클라이언트 응용 프로그램은 각각의 청크를 차례대로 받아 완벽한 응답을 구성할 수 있습니다. 서버에서 크기가 0인 마지막 청크를 전송하면 데이터 전송이 완료됩니다. `PushStreamContent` 클래스를 사용해 ASP.NET Web API에서 청크를 구현할 수 있습니다.

    다음 예는 제품 이미지에 대한 HTTP GET 요청에 응답하는 작업을 보여줍니다.

    ```C#
    public class ProductImagesController : ApiController
    {
        ...
        [HttpGet]
        [Route("productimages/{id:int}")]
        public IHttpActionResult Get(int id)
        {
            try
            {
                var container = ConnectToBlobContainer(Constants.PRODUCTIMAGESCONTAINERNAME);

                if (!BlobExists(container, string.Format("image{0}.jpg", id)))
                {
                    return NotFound();
                }
                else
                {
                    return new FileDownloadResult()
                    {
                        Container = container,
                        ImageId = id
                    };
                }
            }
            catch
            {
                return InternalServerError();
            }
        }
        ...
    }
    ```

    이 예에서 `ConnectBlobToContainer`는 도우미 메서드로 Azure Blob 저장소의 지정된 컨테이너(이름은 표시되지 않음)에 연결합니다. `BlobExists`는 또 다른 도우미 메서드로 Azure Blob 저장소 컨테이너에 지정된 이름이 포함된 Blob가 존재하는지 여부를 나타내는 부울 값을 반환합니다.

    각 제품에는 Blob 저장소에 보관되는 고유한 이미지가 있습니다. `FileDownloadResult` 클래스는 사용자 지정 `IHttpActionResult` 클래스로  `PushStreamContent` 개체를 사용해 적절한 Blob에서 데이터 이미지를 읽고 이를 응답 메시지의 콘텐츠로 비동기적으로 전송합니다.

    ```C#
    public class FileDownloadResult : IHttpActionResult
    {
        public CloudBlobContainer Container { get; set; }
        public int ImageId { get; set; }

        public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
        {
            var response = new HttpResponseMessage();
            response.Content = new PushStreamContent(async (outputStream, _, __) =>
            {
                try
                {
                    CloudBlockBlob blockBlob = Container.GetBlockBlobReference(String.Format("image{0}.jpg", ImageId));
                    await blockBlob.DownloadToStreamAsync(outputStream);
                }
                finally
                {
                    outputStream.Close();
                }
            });

            response.StatusCode = HttpStatusCode.OK;
            response.Content.Headers.ContentType = new MediaTypeHeaderValue("image/jpeg");
            return response;
        }
    }
    ```

    클라이언트가 큰 개체를 포함하고 있는 새로운 리소스를 POST 해야 하는 경우 작업을 업로드하는 데 스트리밍을 적용할 수도 있습니다. 다음 예는 `ProductImages` 컨트롤러에 대한 Post 메서드를 보여줍니다. 이 메서드를 사용해 클라이언트는 새로운 제품 이미지를 업로드할 수 있습니다.

    ```C#
    public class ProductImagesController : ApiController
    {
        ...
        [HttpPost]
        [Route("productimages")]
        public async Task<IHttpActionResult> Post()
        {
            try
            {
                if (!Request.Content.Headers.ContentType.MediaType.Equals("image/jpeg"))
                {
                    return StatusCode(HttpStatusCode.UnsupportedMediaType);
                }
                else
                {
                    var id = new Random().Next(); // Use a random int as the key for the new resource. Should probably check that this key has not already been used
                    var container = ConnectToBlobContainer(Constants.PRODUCTIMAGESCONTAINERNAME);
                    return new FileUploadResult()
                    {
                        Container = container,
                        ImageId = id,
                        Request = Request
                    };
                }
            }
            catch
            {
                return InternalServerError();
            }
        }
        ...
    }
    ```

    이 코드는 다른 사용자 지정 `IHttpActionResult` 클래스를 사용하며, 이 코드는 `FileUploadResult`라고 불립니다. 이 클래스에는 데이터를 비동기적으로 업로드하기 위한 논리가 포함되어 있습니다.

    ```C#
    public class FileUploadResult : IHttpActionResult
    {
        public CloudBlobContainer Container { get; set; }
        public int ImageId { get; set; }
        public HttpRequestMessage Request { get; set; }

        public async Task<HttpResponseMessage> ExecuteAsync(CancellationToken cancellationToken)
        {
            var response = new HttpResponseMessage();
            CloudBlockBlob blockBlob = Container.GetBlockBlobReference(String.Format("image{0}.jpg", ImageId));
            await blockBlob.UploadFromStreamAsync(await Request.Content.ReadAsStreamAsync());
            var baseUri = string.Format("{0}://{1}:{2}", Request.RequestUri.Scheme, Request.RequestUri.Host, Request.RequestUri.Port);
            response.Headers.Location = new Uri(string.Format("{0}/productimages/{1}", baseUri, ImageId));
            response.StatusCode = HttpStatusCode.OK;
            return response;
        }
    }
    ```

  > [!팁]
  > 웹 서비스에 업로드할 수 있는 데이터 볼륨은 스트리밍에 의해 제한을 받지 않으며, 단일 요청으로 인해 상당한 리소스를 사용하는 대규모 개체가 생성될 가능성이 있습니다. 스트리밍 프로세스 동안 웹 API가 요청의 데이터 양이 일부 수락 가능한 경계를 초과한다고 판단한 경우 해당 작업을 중단하고 상태 코드 413(Request Entity Too Large(요청 엔터티가 너무 큼))이 포함된 응답을 반환할 수 있습니다.
  >
  >

    HTTP 압축을 사용해 네트워크를 통해 전송되는 큰 개체의 크기를 최소화할 수 있습니다. 이 접근 방식은 네트워크 트래픽 양과 관련된 네트워크 지연 시간을 줄이는 데는 도움이 되지만 클라이언트와 웹 API를 호스팅하는 서버에서 추가 처리가 요구됩니다. 예를 들어, 압축 데이터를 수신할 것으로 예상하는 클라이언트 응용 프로그램에는 Accept-Encoding: gzip 요청 헤더가 포함될 수 있습니다(다른 데이터 압축 알고리즘도 지정할 수 있음). 서버가 압축을 지원하는 경우 서버는 메시지 본문에 gzip 형식으로 보관되는 콘텐츠와 Content-Encoding: gzip 응답 헤더로 반환해야 합니다.

  > [!팁]
  > 인코딩한 압축과 스트리밍을 결합할 수 있습니다. 데이터를 스트리밍하기 전에 먼저 압축하고 메시지 헤더에 gzip 콘텐츠 인코딩과 청크 분할 전송 인코딩을 지정하십시오. 일부 웹 서버(예: Internet Information Server)는 웹 API의 데이터 압축 여부와  관계없이 자동으로 HTTP 응답을 압축하도록 구성할 수 있습니다.
  >
  >
* **비동기 작업을 지원하지 않는 클라이언트에 대한 부분 응답을 구현하십시오**.

    비동기 스트리밍에 대한 대안으로 클라이언트 응용 프로그램은 부분 응답으로 알려진 청크로 큰 개체에 대한 데이터를 명시적으로 요청할 수 있습니다. 클라이언트 응용 프로그램은 해당 개체에 대한 정보를 얻기 위해 HTTP HEAD 요청을 전송합니다. 웹 API가 부분 응답을 지원하는 경우 웹 API는 Accept-Ranges(범위 수락) 헤더 및 개체의 총 크기를 나타내는 Content-Length(콘텐츠 길이) 헤더가 포함되어 있지만 메시지 본문은 비어 있어야 하는 응답 메시지로 HEAD 요청에 응답해야 합니다. 클라이언트 응용 프로그램은 이 정보를 사용해 수신하는 바이트의 범위를 지정하는 일련의 GET 요청을 구성할 수 있습니다. 웹 API는 HTTP 상태 206(Partial Content(일부 콘텐츠)), 응답 메시지의 본문에 포함된 데이터의 실제량을 지정하는 Content-Length(콘텐츠 길이) 헤더, 이 데이터가 나타내는 개체의 부분(예: 바이트 4000 ~ 8000)을 보여주는 Content-Range(콘텐츠 범위) 헤더가 포함된 응답 메시지를 반환해야 합니다.

    HTTP HEAD 요청 및 부분 응답은 API 설계 지침 문서에서 보다 자세하게 설명되어 있습니다.
* **클라이언트 응용 프로그램에서 불필요한 Continue(계속) 상태 메시지를 전송하지 마십시오**.

    많은 양의 데이터를 서버에 전송하려는 클라이언트 응용 프로그램은 서버가 실제적으로 해당 요청을 수락할 의지가 있는지를 먼저 결정할 수 있습니다. 데이터를 전송하기 전에 클라이언트 응용 프로그램은 Expect: 100-Continue 헤더(데이터의 크기를 나타내는 Content-Length(콘텐츠 길이) 헤더가 포함되어 있지만 메시지 본문은 비어 있음)가 포함된 HTTP 요청을 전송할 수 있습니다. 서버가 해당 요청을 처리할 의지가 있는 경우 서버는 HTTP 상태 100(Continue(계속))을 지정하는 메시지로 응답해야 합니다. 그 후 클라이언트 응용 프로그램은 메시지 본문에 데이터가 포함된 완벽한 요청을 전송할 수 있습니다.

    IIS를 사용해 서비스를 호스팅하는 경우 HTTP.sys 드라이버는 요청을 웹 응용 프로그램에 전달하기 전에 자동으로 Expect: 100-Continue 헤더를 감지하고 처리합니다. 이는 응용 프로그램 코드에서 이러한 헤더를 만날 가능성이 없음을 의미하고, IIS가 적합하지 않거나 너무 크다고 간주되는 모든 메시지를 이미 필터링한 것으로 가정할 수 있습니다.

    .NET Framework를 사용해 클라이언트 응용 프로그램을 구축하는 경우 모든 POST 및 PUT 메시지는 기본적으로 Expect: 100-Continue 헤더가 포함된 메시지를 먼저 전송합니다. 서버 쪽과 마찬가지로 프로세스는 .NET Framework에 의해 투명하게 처리됩니다. 그러나 이 프로세스로 인해 작은 요청이라도 서버를 2번 왕복하도록 하는 POST 및 PUT 요청이 각각 생성됩니다. 응용 프로그램이 많은 양의 데이터가 포함된 요청을 전송하지 않으면 `ServicePointManager` 클래스를 사용해 이 기능을 비활성화하는 방식으로 클라이언트 응용 프로그램에서 `ServicePoint` 개체를 생성할 수 있습니다. `ServicePoint` 개체는 서버의 리소스를 식별하는 URI의 호스트 조각과 스키마를 기반으로 클라이언트와 서버 간의 연결을 처리합니다. `ServicePoint` 개체의 `Expect100Continue` 속성을 false로 설정할 수 있습니다. `ServicePoint` 개체의 호스트 조각 및 스키마와 일치하는 URI를 통해 클라이언트가 하는 모든 후속 POST 및 PUT 요청은 Expect: 100-Continue 헤더 없이 전송됩니다. 다음 코드는 `http` 및 `www.contoso.com` 호스트 스키마로 URI에 전송된 모든 요청을 구성하는 `ServicePoint` 개체 구성 방법을 보여줍니다.

    ```C#
    Uri uri = new Uri("http://www.contoso.com/");
    ServicePoint sp = ServicePointManager.FindServicePoint(uri);
    sp.Expect100Continue = false;
    ```

    `ServicePointManager` 클래스의 정적 `Expect100Continue` 속성을 설정해 차후 생성되는 모든 `ServicePoint` 개체에 대해 이 속성의 기본값을 지정할 수도 있습니다. 자세한 정보는 Microsoft 웹 사이트의 [ServicePoint 클래스](https://msdn.microsoft.com/library/system.net.servicepoint.aspx) 페이지를 참조하십시오.
* **많은 수의 개체를 반환할 수 있는 요청에 대한 페이지 매김을 지원하십시오**.

    컬렉션에 많은 수의 리소스가 포함되어 있는 경우 해당 URI에 GET 요청을 발행하면 웹 API를 호스팅하는 서버에서 상당한 처리가 이루어져 성능에 영향을 미치고 상당한 양의 네트워크 트래픽이 생성되어 대기 시간이 증가할 수 있습니다.

    이러한 경우를 처리하기 위해서 웹 API는 클라이언트 응용 프로그램이 더욱 관리하기 쉬운 불연속 블록 (또는 페이지)으로 요청을 구체화하거나 데이터를 가져올 수 있도록 하는 쿼리 문자열을 지원해야 합니다. ASP.NET Web API 프레임워크는 앞서 설명한 것처럼 라우팅 규칙에 따라 쿼리 문자열을 구문 분석하고 적절한 메서드로 전달되는 일련의 매개 변수/값 쌍으로 분할합니다. 쿼리 문자열에 지정된 동일한 이름을 사용해 이러한 매개 변수를 수락하도록 메서드를 구현해야 합니다. 뿐만 아니라 이러한 매개 변수는 선택적이어야 하며 (클라이언트가 요청에서 쿼리 문자열을 생략한 경우) 의미 있는 기본값을 가져야 합니다. 아래 코드는 `Orders` 컨트롤러의 `GetAllOrders` 메서드를 보여줍니다. 이 메서드는 주문 상세 정보를 검색합니다. 이 메서드가 제한을 받지 않는다면 많은 양의 데이터를 반환하는 것이 가능합니다. `limit` 및 `offset` 매개 변수의 목적은 데이터 볼륨을 더 작은 하위 집합으로 줄이는 것이며, 이 경우에는 기본적으로 처음 10개 주문에만 해당합니다.

    ```C#
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

    클라이언트 응용 프로그램은 URI *http://www.adventure-works.com/api/orders?limit=30&offset=50* 를 사용해 오프셋 50에서 시작하는 30 주문을 검색하기 위한 요청을 발행할 수 있습니다.

  > [!팁]
  > 클라이언트 응용 프로그램이 길이가 2000자를 초과하는 URI를 생성하는 문자열을 지정하도록 하지 마십시오. 많은 웹 클라이언트 및 서버는 이렇게 긴 URI를 처리할 수 없습니다.
  >
  >

<a name="considerations-for-maintaining-responsiveness"></a>

## 응답성, 확장성, 가용성 유지 시 고려사항
전 세계 어디에서는 실행되는 많은 클라이언트 응용 프로그램이 동일한 웹 API를 활용할 수 있습니다. 웹 API는 부하가 높은 상황에서도 응답성을 유지하고, 매우 다양한 워크로드를 지원하도록 확장성을 가지며, 비즈니스에 매우 중요한 작업을 수행하는 클라이언트에 대해 가용성을 보장하도록 구현되는지를 확인하는 것이 매우 중요합니다. 이러한 요구 사항을 충족하는 방법을 결정할 때 다음 사항을 고려하십시오. 

* **장기 실행 요청에 대한 비동기 지원을 제공하십시오**.

    처리하는 데 오랜 시간이 소요될 수 있는 요청은 해당 요청을 전송한 클라이언트를 차단하지 않고 수행되어야 합니다. 웹 API는 요청 유효성을 검사하기 위한 일부 최초 검사를 수행하고 작업을 수행하기 위한 별도 작업을 시작한 다음 HTTP 코드 202(Accepted(수락됨))가 포함된 응답 메시지를 반환할 수 있습니다. 이 작업은 웹 API 처리의 일부로 비동기적으로 실행되거나 Azure WebJob(웹 API가 Azure 웹 사이트에 의해 호스팅 되는 경우) 또는 작업자 역할(웹 API가 Azure 클라우드 서비스로 구현되는 경우)에 오프로드될 수 있습니다.

  > [!참고]
  > Azure 웹사이트에서 WebJobs를 사용하는 방법에 대한 자세한 정보는 Microsoft 웹 사이트의 [Microsoft Azure 웹사이트에서 배경 작업을 실행하는 데 WebJobs 사용](/azure/app-service-web/web-sites-create-web-jobs/) 페이지를 참조하십시오.
  >
  >

    웹 API는 처리 결과를 클라이언트 응용 프로그램에 반환하는 메커니즘도 제공해야 합니다. 클라이언트 응용 프로그램이 처리가 완료되어 결과를 얻었는지를 정기적으로 쿼리하는 폴링 메커니즘을 제공하거나 웹 API가 작업이 완료되었을 때 알림을 전송하도록 해 이러한 메커니즘을 제공할 수 있습니다.

    다음과 같은 접근 방식을 사용해 가상 리소스 역할을 하는 *polling(폴링)* URI를 제공함으로써 간단한 폴링 메커니즘을 구현할 수 있습니다.

  1. 클라이언트 응용 프로그램은 웹 API에 초기 요청을 전송합니다.
  2. 웹 API는 테이블 저장소 또는 Microsoft Azure Cache에 보관된 테이블의 요청에 대한 정보를 저장하고 이러한 항목에 대해, GUID 양식으로 고유한 키를 생성합니다.
  3. 웹 API는 별도의 작업으로 처리를 시작합니다. 웹 API는 테이블에서 작업의 상태를 *Running(실행 중)*으로 기록합니다.
  4. 웹 API는 HTTP 상태 코드 202(Accepted(수락됨)) 및 메시지 본문의 테이블 항목 GUID가 포함된 응답 메시지를 반환합니다.
  5. 작업이 완료되면 웹 API는 테이블에 결과를 저장하고 작업 상태를 *Complete(완료)*로 설정합니다. 작업이 실패하면 웹 API는 실패에 대한 정보를 저장하고 상태를 *Failed(실패)*로 설정할 수도 있습니다.
  6. 작업이 실행되는 동안 클라이언트는 자체 처리를 계속 수행할 수 있습니다. URI */polling/{guid}*에 대한 요청을 정기적으로 전송할 수 있는데, 여기서 *{guid}*는 웹 API에 의해 202 응답 메시지로 반환된 GUID입니다.
  7. */polling/{guid}* URI에서 웹 API는 테이블의 해당 작업 상태를 쿼리하고 해당 상태(*Running(실행 중)*, *Complete(완료)* 또는 *Failed(실패)*)가 포함된 HTTP 상태 코드 200(OK(확인))이 포함된 응답 메시지를 반환합니다. 작업이 완료되거나 실패하면, 응답 메시지에는 처리 결과 또는 실패 원인에 관해 사용 가능한 정보가 포함될 수 있습니다.

     알림 구현을 선호하는 경우 사용 가능한 옵션에는 다음이 포함됩니다.
  8. Azure 알림 허브를 사용해 클라이언트 응용 프로그램에 비동기 응답 푸시. Microsoft 웹 사이트의[Azure 알림 허브 사용자 알림](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) 페이지에서 보다 자세한 상세 정보를 제공합니다.
  9. 클라이언트와 웹 API 호스팅 서버 간의 영구적 네트워크 연결을 유지하기 위해 Comet 모델을 사용하고 서버에서 클라이언트로 다시 메시지를 푸시하는 데 영구적 네트워크 연결 사용. MSDN 매거진 [Microsoft .NET Framework에서 단순한 Comet 응용 프로그램 구축](https://msdn.microsoft.com/magazine/jj891053.aspx) 기사에서 예로 한 가지 솔루션에 대해 설명합니다.
  10. 영구적 네트워크 연결을 통해 웹 서버에서 클라이언트로 실시간 데이터 푸시에 SignalR 사용. SignalR은 ASP.NET 웹 응용 프로그램에 대해 NuGet 패키지로 사용할 수 있습니다. [ASP.NET SignalR](http://signalr.net/) 웹 사이트에서 자세한 정보를 확인할 수 있습니다.

      > [!참고]
      > Comet 및 SignalR 모두 웹 서버 및 클라이언트 응용 프로그램 간에 영구적 네트워크 연결을 활용합니다. 이는 많은 수의 클라이언트에 동일하게 많은 수의 동시 연결이 필요할 수 있으므로 확장성에 영향을 미칠 수 있습니다.
      >
      >
* **각각의 요청이 상태 비저장인지 확인하십시오**.

    각각의 요청은 원자적으로 간주되어야 합니다. 클라이언트 응용 프로그램이 한 요청과 동일한 클라이언트가 제출한 후속 요청 간의 종속성이 없어야 합니다. 이 접근 방식은 확장성에 도움이 됩니다. 웹 서비스의 인스턴스는 많은 수의 서버에 배치될 수 있습니다. 클라이언트 요청은 이러한 인스턴스 어디에서나 지정될 수 있고 결과는 항상 같아야 합니다. 이는 비슷한 이유로 가용성을 개선합니다. 웹 서버가 실패하는 경우 요청은 (Azure 트래픽 관리자를 사용하여) 다른 인스턴스로 라우트될 수 있고 그 동안 서버는 클라이언트 응용 프로그램에 부정적인 영향을 미치지 않고 다시 시작됩니다.
* **DOS 공격 가능성을 줄이기 위해 클라이언트를 추적하고 제한을 구현하십시오**.

    특정 클라이언트가 지정된 기간 내에 많은 수의 요청을 하면 서비스를 독점하고 다른 클라이언트의 성능에 영향을 미칠 수 있습니다. 이러한 문제를 완화하기 위해 웹 API는 들어오는 모든 요청의 IP 주소를 추적하거나 각각의 인증된 액세스를 로깅함으로써 클라이언트 응용 프로그램의 호출을 모니터링할 수 있습니다. 이 정보를 사용해 리소스 액세스를 제한할 수 있습니다. 클라이언트가 정의된 제한을 초과하는 경우 웹 API는 상태 503(Service Unavailable(서비스를 사용할 수 없음))이 포함된 응답 메시지를 반환하고 클라이언트가 거부되지 않고 다음 요청을 전송할 수 있는 때를 지정하는 Retry-After 헤더를 포함시킬 수 있습니다. 이 전략은 시스템 작동 중단을 유발하는 일단의 클라이언트로부터 서비스 거부(DOS) 공격 가능성을 줄이는 데 도움이 될 수 있습니다.
* **영구적인 HTTP 연결을 신중하게 관리하십시오**.

    HTTP 프로토콜은 영구적인 HTTP 연결이 사용 가능한 경우 해당 연결을 지원합니다. HTTP 1.0 사양은 클라이언트 응용 프로그램이 새로운 연결을 여는 것이 아니라 후속 요청을 전송하는 데 동일한 연결을 사용할 수 있음을 서버에 표시하도록 하는 Connection:Keep-Alive 헤더를 추가했습니다. 클라이언트가 호스트에서 정의한 기간 내에 해당 연결을 다시 사용하지 않으면 해당 연결은 자동으로 닫힙니다. 이는 Azure 서비스에 의해 사용되는 HTTP 1.1에서는 기본 동작이므로 메시지에 Keep-Alive 헤더를 포함시킬 필요가 없습니다.

    연결을 열린 상태로 유지하는 것은 대기 시간 및 네트워크 정체를 줄여 응답성을 개선하는 데는 도움이 되지만, 필요한 시간보다 더 오래 불필요한 연결을 열린 상태로 유지해 확장성에 부정적인 영향을 미치기 때문에 다른 동시 클라이언트가 연결되는 기능을 제한할 수 있습니다. 이는 배터리 수명에도 영향을 미칠 수 있는데, 클라이언트 응용 프로그램이 모바일 장치에서 실행되는 경우, 클라이언트 응용 프로그램이 서버에 대해 간헐적으로만 요청을 하는 경우 열린 연결을 유지하는 것은 배터리를 더욱 빨리 닳게 할 수 있습니다. HTTP 1.1과 영구적으로 연결이 되지 않도록 클라이언트는 기본 동작을 다시 정의하기 위한 메시지와 함께 Connection:Close 헤더를 포함할 수 있습니다. 이와 마찬가지로 서버가 굉장히 많은 수의 클라이언트를 처리하는 경우 서버는 응답 메시지에 연결을 닫고 서버 리소스를 저장해야 하는 Connection:Close 헤더를 포함할 수 있습니다.

  > [!참고]
  > 영구적 HTTP 연결은 반복적으로 통신 채널을 설정하는 것과 관련된 네트워크 오버헤드를 줄이기 위한 완전히 선택적인 기능입니다. 웹 API나 클라이언트 응용 프로그램도 사용 가능한 영구적 HTTP 연결에 의존해서는 안 됩니다. Comet 스타일 알림 시스템을 구현하는 데 영구적 HTTP 연결을 사용하지 마십시오. 그 대신 TCP 레이어에서 소켓 (또는 사용 가능한 경우 웹소켓)을 활용해야 합니다. 마지막으로 클라이언트 응용 프로그램이 프록시를 통해 서버와 통신하는 경우 Keep-Alive 헤더는 사용이 제한적입니다. 클라이언트 및 프록시와의 연결만이 영구적입니다.
  >
  >

## 웹 API 게시 및 관리 시 고려사항
웹 API를 클라이언트 응용 프로그램에서 사용할 수 있도록 하려면 웹 API를 호스트 환경에 배치해야 합니다. 호스트 환경은 다른 유형의 호스트 프로세스일 수 있지만 일반적으로 웹 서버입니다. 웹 API를 게시할 때 다음 사항을 고려해야 합니다. 

* 모든 요청은 인증 및 권한을 부여 받아야 하며, 적절한 수준의 액세스 제한이 적용되어야 합니다.
* 상업적 웹 API는 응답 시간과 관련된 다양한 품질 보증의 적용을 받을 수 있습니다. 부하가 시간에 따라 상당 부분 변화하는 경우 호스트 환경이 확장성이 있는지 확인하는 것이 중요합니다.
* 영리 목적으로 요청을 측정해야 할 필요가 있습니다.
* 웹 API로의 트래픽 흐름을 규제하고 할당량을 모두 사용한 특정 클라이언트에 대한 제한을 구현해야 할 필요가 있습니다.
* 규정상의 요구 사항이 모든 요청 및 응답의 로깅 및 감사를 강제할 수도 있습니다.
* 가용성을 확보하기 위해 웹 API를 호스팅하는 서버 상태를 모니터링하고 필요한 경우 다시 시작해야 할 수도 있습니다.

이러한 문제를 웹 API 구현과 관련된 기술적인 문제와 구별하는 것이 유용합니다. 이러한 이유로 별도의 프로세스로 실행되고 요청을 웹 API에 라우트하는 [façade](http://en.wikipedia.org/wiki/Facade_pattern)를 생성하는 것을 고려해 보십시오. façade는 관리 작업을 제공하고 확인된 요청을 웹 API에 전달할 수 있습니다. façade 사용은 다음을 포함한 많은 기능적 이점을 제공할 수 있습니다. 

* 여러 웹 API에 대한 통합 포인트로서의 역할
* 다양한 기술을 사용해 구축된 클라이언트에 대한 메시지 전달 및 통신 프로토콜 변환
* 웹 API 호스팅 서버에 대한 부하를 줄이기 위해 요청 및 응답 캐싱

## 웹 API 테스트 시 고려사항
웹 API는 다른 소프트웨어와 마찬가지로 철저하게 테스트를 거쳐야 합니다. 다른 유형의 응용 프로그램과 마찬가지로 각 작업의 기능을 확인하기 위해 단위 테스트를 생성하는 것을 고려해야 합니다. 자세한 정보는 Microsoft 웹 사이트의 [단위 테스트를 사용하여 코드 확인](https://msdn.microsoft.com/library/dd264975.aspx) 페이지를 참조하십시오. 

> [!참고]
> 이 지침에서 사용 가능한 간단한 웹 API에는 선택한 작업에 대한 단위 테스트 수행 방법을 보여주는 테스트 프로젝트가 포함되어 있습니다. 
>
>

웹 API의 속성으로 인해 웹 API가 정확하게 작동하는지를 확인하기 위한 요구 사항이 추가됩니다. 다음 측면에 특히 더 주의를 기울여야 합니다. 

* 정확한 작업을 호출하는지 확인하기 위해 모든 경로를 테스트하십시오. 특히 예상치 못하게 반환되는 HTTP 상태 코드 405(Method Not Allowed(메서드 허용 안 함))에 주의를 기울여야 합니다. 이 상태 코드가 경로와 해당 경로에 디스패치될 수 있는 HTTP 메서드(GET, POST, PUT, DELETE) 간의 불일치를 나타낼 수 있기 때문입니다.

    특정 리소스에 POST 요청을 제출하는 것처럼(POST 요청은 리소스 컬렉션에만 전송될 수 있음) 지원하지 않는 경로에 HTTP 요청을 전송하십시오. 이러한 경우 유효한 응답은 *반드시* 상태 코드 405(Not Allowed(허용되지 않음))이어야 합니다.
* 모든 경로가 적절하게 보호되고 있는지, 모든 경로가 적절한 인증 및 권한 부여 확인 대상인지를 확인하십시오.

  > [!참고]
  > 사용자 인증과 같은 일부 보안 측면은 웹 API보다는 호스트 환경의 책임일 가능성이 가장 높지만, 배치 프로세스의 일환으로 보안 테스트를 포함시킬 필요가 있습니다.
  >
  >
* 각각의 작업에 의해 수행되는 예외 처리를 테스트하고 적절하고 의미 있는 HTTP 응답이 클라이언트 응용 프로그램에 다시 전달되는지를 확인하십시오.
* 요청 및 응답 메시지가 잘 구성되었는지 확인하십시오. 예를 들어, HTTP POST 요청에 x-www-form-urlencoded 형식의 새로운 리소스에 대한 데이터가 포함되어 있는 경우 해당 작업이 정확하게 데이터를 구문 분석하고, 리소스를 생성하고, 정확한 Location(위치) 헤더를 포함한 새로운 리소스에 대한 세부 정보를 포함하는 응답을 반환하는지 확인하십시오.
* 응답 메시지의 모든 링크와 URI를 확인하십시오. 예를 들어, HTTP POST 메시지는 새롭게 생성된 리소스의 URI를 반환해야 합니다. 모든 HATEOAS 링크가 유효해야 합니다.

  > [!중요]
  > API 관리 서비스를 통해 웹 API를 게시하는 경우 이러한 URI는 웹 API를 호스팅하는 웹 서버의 URI가 아니라 관리 서비스의 URI를 반영해야 합니다.
  >
  >
* 각 작업이 다양한 조합의 입력에 대해서 정확한 상태 코드를 반환하는지 확인하십시오. 예를 들어

  * 쿼리가 성공적인 경우 상태 코드 200(OK(확인))을 반환해야 합니다.
  * 리소스를 찾지 못한 경우 작업은 HTTP 상태 코드 404(Not Found(찾을 수 없음))을 반환해야 합니다.
  * 클라이언트가 성공적으로 리소스를 삭제하라는 요청을 전송하는 경우 상태 코드는 204(No Content(콘텐츠 없음))이어야 합니다.
  * 클라이언트가 새로운 리소스를 생성하라는 요청을 전송하는 경우 상태 코드는 201(Created(생성됨))이어야 합니다.

5xx 범위의 예상치 못한 응답 상태 코드가 생성되는지 주의하십시오. 이러한 메시지는 일반적으로 유효한 요청을 처리할 수 없음을 나타내기 위해 호스트 서버에 의해 보고됩니다. 

* 클라이언트 응용 프로그램이 지정할 수 있는 다양한 요청 헤더 조합을 테스트하고 웹 API가 응답 메시지에서 예상한 정보를 반환하는지 확인하십시오.
* 쿼리 문자열을 테스트하십시오. 작업에 (페이지 매김 요청과 같은) 선택적 매개 변수가 포함될 수 있는 경우 다양한 매개 변수 조합 및 순서를 테스트하십시오.
* 비동기 작업이 성공적으로 완료되는지 확인하십시오. 웹 API가 (비디오 또는 오디오 같은) 큰 바이너리 개체를 반환하는 요청에 대한 스트리밍을 지원하는 경우 데이터가 스트리밍되는 동안 클라이언트 요청이 차단되지 않는지 확인하십시오. 웹 API가 장기 실행 데이터 수정 작업에 대한 폴링을 구현하는 경우 해당 작업이 진행되는 대로 상태를 올바르게 보고하는지 확인하십시오.

웹 API가 압박을 받는 상태에서도 만족스러운 수준으로 작동하는지를 검사하기 위한 성능 테스트를 생성 및 실행해야 합니다. Visual Studio Ultimate을 사용해 웹 성능 및 부하 테스트 프로젝트를 구축할 수 있습니다. 자세한 정보는 Microsoft 웹 사이트의 [릴리스 전 응용 프로그램에 대한 성능 테스트 실행](https://msdn.microsoft.com/library/dn250793.aspx) 페이지를 참조하십시오. 

## Azure API 관리 서비스를 사용하여 웹 API 게시 및 관리
Azure는 웹 API를 게시 및 관리하는 데 사용할 수 있는 [API 관리 서비스](https://azure.microsoft.com/documentation/services/api-management/)를 제공합니다. 이 기능을 사용해 하나 이상의 웹 API에 대해서 façade 역할을 수행하는 서비스를 생성할 수 있습니다. 이 서비스는 서비스 자체가 확장 가능한 웹 서비스로 Azure 관리 포털을 사용해 생성하고 구성할 수 있습니다. 이 서비스를 사용해 다음과 같이 웹 API를 게시하고 관리할 수 있습니다. 

1. 웹 사이트, Azure 클라우드 서비스, Azure 가상 컴퓨터에 웹 API를 배치하십시오.
2. API 관리 서비스를 웹 API에 연결하십시오. 관리 API의 URL로 전송된 요청은 웹 API의 URI에 매핑됩니다. 동일한 API 관리 서비스가 하나 이상의 웹 API에 요청을 라우트할 수 있습니다. 이를 통해 여러 개의 웹 API를 단 하나의 관리 서비스에 집계할 수 있습니다. 이와 마찬가지로 다양한 응용 프로그램에 대해 사용 가능한 기능을 제한 또는 파티션해야 하는 경우 동일한 웹 API를 한 개 이상의 API 관리 서비스에서 참조할 수 있습니다.

   > [!참고]
   > HTTP GET 요청에 대한 응답의 일부로 생성된 HATEOAS 링크의 URI는 웹 API를 호스팅하는 웹 서버가 아니라 API 관리 서비스의 URI를 참조해야 합니다.
   >
   >
3. 각각의 웹 API에 대해서 작업이 입력으로 가져올 수 있는 선택적 매개 변수와 함께 웹 API가 표시하는 HTTP 작업을 지정하십시오. 동일한 데이터에 대한 반복된 요청을 최적화하기 위해 API 관리 서비스가 웹 API에서 수신한 응답을 캐시해야 할지를 구성할 수 있습니다. 각각의 작업이 생성할 수 있는 HTTP 응답의 세부 정보를 기록하십시오. 이 정보는 개발자용 설명서를 생성하는 데 사용되므로 정확하고 완전해야 합니다.

    Azure 관리 포털에서 제공하는 마법사를 사용해 수동으로 작업을 정의하거나 WADL 또는 Swagger 형식으로 정의가 포함된 파일에서 작업을 가져올 수 있습니다.
4. API 관리 서비스 및 웹 API를 호스팅하는 웹 서버 간의 통신에 대한 보안 설정을 구성하십시오. API 관리 서비스는 현재 인증서를 사용하는 기본 인증 및 상호 인증, 그리고 OAuth 2.0 사용자 권한 부여를 지원합니다.
5. 제품을 생성하십시오. 제품은 게시 단위입니다. 이전에 관리 서비스에 연결한 웹 API를 제품에 추가합니다. 제품이 게시되면 웹 API를 개발자들이 사용할 수 있습니다.

   > [!참고]
   > 제품을 게시하기 전에 제품에 액세스할 수 있는 사용자 그룹을 정의하고 사용자를 이러한 그룹에 추가할 수 있습니다. 이를 통해 해당 웹 API를 사용할 수 있는 개발자 및 응용 프로그램에 대한 권한이 수여됩니다. 웹 API가 승인 대상이라면 해당 웹 API에 액세스하기 전에 개발자는 제품 관리자에게 요청을 전송해야 합니다. 관리자는 해당 개발자에 대해 액세스를 수여하거나 거부할 수 있습니다. 환경이 변화하는 경우 기존 개발자도 차단할 수 있습니다.
   >
   >
6. 각각의 웹 API에 대한 정책을 구성하십시오. 정책은 도메인 간 호출을 허용할지 여부, 클라이언트 인증 방법, XML 및 JSON 데이터 형식을 투명하게 변환할지 여부, 주어진 IP 범위에서의 호출 제한 여부, 사용 할당량, 호출 속도 제한 여부와 같은 측면을 제어합니다. 정책은 제품의 단일 웹 API에 대해서 또는 웹 API의 개별적인 작업에 대해서 전체 제품 전반에서 전역으로 적용될 수 있습니다.

Microsoft 웹 사이트의 [API 관리](https://azure.microsoft.com/services/api-management/) 페이지에서 이러한 작업 수행 방법을 설명하는 모든 세부 정보를 확인할 수 있습니다. Azure API 관리 서비스는 고유한 REST 인터페이스를 제공하므로 사용자 지정 인터페이스를 구축해 웹 API를 구성하는 프로세스를 단순화할 수 있습니다. 자세한 정보는 Microsoft 웹 사이트의[Azure API Management REST API 리퍼런스](https://msdn.microsoft.com/library/azure/dn776326.aspx) 페이지를 참조하십시오. 

> [!팁]
> Azure는 Azure 트래픽 관리자를 제공합니다. 이를 통해 장애 조치 및 부하 분산을 구현할 수 있고 다양한 지리적 위치에서 호스팅되는 웹 사이트의 여러 인스턴스 전반의 대기 시간을 줄일 수 있습니다. Azure 트래픽 관리자를 API 관리 서비스와 함께 사용할 수 있습니다. API 관리 서비스는 Azure 트래픽 관리자를 통해 웹 사이트의 인스턴스에 요청을 라우트할 수 있습니다. 자세한 정보는 Microsoft 웹 사이트의 [트래픽 관리자 라우팅 메서드](/azure/traffic-manager/traffic-manager-routing-methods/) 페이지를 참조하십시오. 
>
> 이 구조에서 웹 사이트에 대해 사용자 지정 DNS 이름을 사용하는 경우 Azure 트래픽 관리자의 DNS 이름을 가리키도록 각각의 웹 사이트에 대한 적절한 CNAME 기록을 구성해야 합니다. 
>
>

## 클라이언트 응용 프로그램 구축 개발자 지원
클라이언트 응용 프로그램을 만드는 개발자는 일반적으로 웹 API 액세스 방법에 관한 정보, 그리고 매개 변수, 데이터 유형, 웹 서비스와 클라이언트 응용 프로그램 간의 다양한 요청 및 응답을 설명하는 반환 코드와 관련된 설명서를 필요로 합니다. 

### 웹 API에 대한 REST 작업 문서화
Azure API 관리 서비스에는 웹 API가 표시하는 REST 작업을 설명하는 개발자 포털이 포함됩니다. 제품이 게시되면 개발자 포털에 나타납니다. 개발자는 이 포털을 사용해 액세스를 위한 가입을 할 수 있고 관리자는 해당 요청을 승인 또는 거부할 수 있습니다. 개발자가 승인을 받는 경우 개발자에게는 그들이 개발한 클라이언트 응용 프로그램의 호출을 인증하는 데 사용되는 등록 키가 할당됩니다. 각각의 웹 API 호출에 대해서 등록 키가 제공되어야 하며, 제공되지 않으면 거부됩니다. 

개발자 포럼은 다음을 제공합니다. 

* 제품 설명서, 표시 작업 목록, 필요 매개 변수, 반환될 수 있는 다양한 응답. 참고로 이 정보는 Microsoft Azure API 관리 서비스를 사용하여 웹 API 게시 섹션의 목록 3단계에서 제공된 세부 정보에서 생성됩니다.
* JavaScript, C#, Java, Ruby, Python, PHP를 포함한 다양한 언어에서 작업을 호출하는 방법을 보여주는 코드 조각.
* 개발자가 제품에서 각각의 작업을 테스트하기 위해 HTTP 요청을 전송하고 결과를 확인할 수 있도록 하는 개발자 콘솔.
* 개발자가 발견한 이슈 또는 문제를 보고할 수 있는 페이지

Azure 관리 포털을 사용해 개발자 포털을 사용자 지정함으로써 조직의 브랜딩에 맞게 스타일과 레이아웃을 변경할 수 있습니다. 

### 클라이언트 SDK 구현
웹 API에 액세스하기 위한 REST 요청을 호출하는 클라이언트 응용 프로그램을 구축하려면 각각의 요청을 구성하고 요청을 적절하게 포맷하고, 요청을 웹 서비스를 호스팅하는 서버에 전송하고, 해당 요청이 성공 또는 실패했는지를 확인하기 위해 응답을 구문 분석하고 반환된 데이터를 추출하기 위해 상당한 양의 코드를 작성해야 합니다. 클라이언트 응용 프로그램은 이러한 영향을 받지 않도록 하기 위해 REST 인터페이스를 래핑하고 보다 기능적인 메서드 세트 내에서 낮은 수준의 세부 정보를 요약하는 SDK를 제공할 수 있습니다. 클라이언트 응용 프로그램은 투명하게 호출을 REST 요청으로 변환한 다음 응답을 다시 메서드 반환 값으로 변환하는 이러한 메서드를 사용합니다. 이는 Azure SDK를 포함해 많은 서비스에서 구현하는 공통적인 테크닉입니다. 

클라이언트 쪽 SDK 생성은 일관되게 구현되어야 하고 신중하게 테스트되어야 하기 때문에 상당히 힘든 작업입니다. 그러나 이 프로세스의 상당 부분을 기계적으로 만들 수 있고, 많은 벤더가 이러한 작업의 많은 부분을 자동화할 수 있는 도구를 제공합니다. 

## 웹 API 모니터링
웹 API를 게시 및 배치하는 방법에 따라 웹 API를 직접 모니터링할 수도 있고 API 관리 서비스를 통과하는 트래픽을 분석해 사용 및 상태 정보를 수집할 수도 있습니다. 

### 웹 API 직접 모니터링
(웹 API 프로젝트 또는 Azure 클라우드 서비스의 웹 역할로서) ASP.NET Web API 템플릿 및 Visual Studio 2013을 사용해 웹 API를 구현한 경우 ASP.NET 응용 프로그램 인사이트를 사용해 가용성, 성능, 사용 데이터를 수집할 수 있습니다. 응용 프로그램 인사이트는 웹 API가 클라우드에 배치될 때 요청 및 응답에 대한 정보를 투명하게 추적 및 기록하는 패키지입니다. 일단 패키지가 설치 및 구성되면 웹 API에서 사용할 코드를 수정할 필요가 없습니다. 웹 API를 Azure 웹 사이트에 배치하는 경우 모든 트래픽이 검사되고 다음 통계가 수집됩니다. 

* 서버 응답 시간.
* 서버 요청 수 및 각 요청의 세부 정보.
* 평균 응답 시간 관점에서 가장 느린 요청.
* 실패한 요청의 세부 정보.
* 다른 브라우저 및 사용자 에이전트에 의해 시작된 세션 수.
* 가장 자주 본 페이지(웹 API보다는 웹 응용 프로그램에 주로 유용함).
* 웹 API에 액세스하는 다양한 사용자 역할.

Azure 관리 포털에서 실시간으로 이러한 데이터를 확인할 수 있습니다. 웹 API의 상태를 모니터링하는 웹 테스트도 생성할 수 있습니다. 웹 테스트는 웹 API에서 지정된 URI에 정기적으로 요청을 전송하고 응답을 캡처합니다. 성공적인 응답의 정의(예: HTTP 상태 코드 200)를 지정할 수 있고 요청이 이 응답을 반환하지 않는 경우 관리자에게 전송할 경고를 만들 수 있습니다. 필요한 경우 관리자는 웹 API를 호스팅하는 서버를 다시 시작할 수 있습니다(실패한 경우). 

Microsoft 웹 사이트의 [응용 프로그램 인사이트 - ASP.NET으로 시작하기](/azure/application-insights/app-insights-asp-net/) 페이지에서 자세한 정보를 제공합니다. 

### API 관리 서비스를 통한 웹 API 모니터링
API 관리 서비스를 사용해 웹 API를 게시한 경우 Azure 관리 포털의 API 관리 페이지에는 서비스의 전반적인 성능을 확인할 수 있는 대시보드가 포함되어 있습니다. Analytics(애널리틱스) 페이지에서 제품이 어떻게 사용되고 있는지에 대한 세부 정보를 자세하게 확인할 수 있습니다. 이 페이지에는 다음과 같은 탭이 있습니다. 

* **Usage(사용)**. 이 탭은 이루어진 API 호출 횟수 및 시간에 따라 이러한 호출을 처리하기 위해 사용된 대역폭에 대한 정보를 제공합니다. 제품, API, 작업별로 사용 세부 정보를 필터링할 수 있습니다.
* **Health(상태)**. 이 탭을 사용해 API 요청 결과 (반환된 HTTP 상태 코드), 캐싱 정책 유효성, API 응답 시간, 서비스 응답 시간을 확인할 수 있습니다. 제품, API, 작업별로 상태 데이터를 필터링할 수 있습니다.
* **Activity(활동)**. 이 탭은 성공한 호출, 실패한 호출, 차단된 호출의 횟수, 평균 응답 시간, 각각의 제품에 대한 응답 시간, 웹 API, 작업 횟수에 대한 텍스트 요약을 제공합니다. 이 페이지는 각 개발자가 한 호출 횟수도 표시합니다.
* **At a glance(한 눈에 보기)**. 이 탭은 대부분의 API 호출을 할 책임이 있는 개발자, 모든 제품, 웹 API, 이러한 호출을 받은 작업을 포함해 성능 데이터에 대한 요약을 표시합니다.

특정 웹 API 또는 작업으로 인해 병목 현상이 야기되었는지를 확인하는 데 이 정보를 사용할 수 있고 필요한 경우 호스트 환경을 확장/축소하고 더 많은 서버를 추가할 수 있습니다. 한 개 이상의 응용 프로그램에서 불균형적으로 많은 리소스 볼륨을 사용하고 있는지를 확인하고 적절한 정책을 적용해 할당량을 설정하고 호출 속도를 제한할 수 있습니다. 

> [!참고]
> 게시된 제품의 세부 정보를 변경할 수 있으며, 변경은 바로 적용됩니다. 예를 들어 웹 API가 포함된 제품을 다시 게시할 필요 없이 웹 API에서 작업을 추가하거나 제거할 수 있습니다. 
>
>

## 관련 패턴
* [façade](http://en.wikipedia.org/wiki/Facade_pattern) 패턴은 웹 API에 대한 인터페이스 제공 방법을 설명합니다.

## 자세한 정보
* •	Microsoft 웹 사이트의 [ASP.NET Web API 살펴보기](http://www.asp.net/web-api) 페이지는 웹 API를 사용한 RESTful 웹 서비스 구축에 대한 자세한 소개를 제공합니다.
* •	Microsoft 웹 사이트의 [ASP.NET Web API에서의 라우팅](http://www.asp.net/web-api/overview/web-api-routing-and-actions/routing-in-aspnet-web-api) 페이지는 ASP.NET Web API 프레임워크에서 규칙 기반 라우팅이 어떻게 작동하는지를 설명합니다.
* •	특성 기반 라우팅에 대한 자세한 정보는 Microsoft 웹 사이트에 나와 있는 [Web API 2에서의 특성 라우팅](http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2) 을 참조하십시오.
* •	OData 웹 사이트의 [기본 자습서](http://www.odata.org/getting-started/basic-tutorial/) 페이지는 OData 프로토콜 기능에 대한 소개를 제공합니다.
* •	Microsoft 웹 사이트의[ASP.NET Web API OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) 페이지에는 ASP.NET을 사용하여 OData web API를 구현하는 예와 추가 정보가 포함되어 있습니다.
* •	Microsoft 웹 사이트의 [웹 API 및 웹 API OData에서의 일괄 지원 소개](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) 페이지는 OData를 사용해 웹 API에서 일괄 작업을 구현하는 방법을 설명합니다.
* •	조나단 올리버(Jonathan Oliver)의 블로그에 게재된 [Idempotency 패턴](http://blog.jonathanoliver.com/idempotency-patterns/) 에 관한 기사는 Idempotency 개요 및 Idempotency가 데이터 작업 관리와 어떻게 관련되는지를 설명합니다.
* •	W3C 웹사이트의 [상태 코드 정의](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) 페이지에는 HTTP 상태 코드 전체 목록 및 해당 설명이 포함되어 있습니다.
* •	ASP.NET Web API와 관련된 HTTP 예외 처리에 대한 자세한 정보는 Microsoft 웹 사이트의 [ASP.NET Web API에서의 예외 처리](http://www.asp.net/web-api/overview/error-handling/exception-handling) 페이지를 참조하십시오.
* •	Microsoft 웹 사이트의 [웹 API 전역 오류 처리](http://www.asp.net/web-api/overview/error-handling/web-api-global-error-handling) 기사에서 웹 API에 대한 전역 오류 처리 및 로깅 전략 구현 방법을 설명합니다.
* •	Microsoft 웹 사이트의 [배경 작업 실행에 WebJobs 사용](/azure/app-service-web/web-sites-create-web-jobs/) 페이지는 Azure 웹 사이트에서 배경 작업을 수행하기 위한 WebJobs 사용에 대한 정보 및 예를 제공합니다.
* •	Microsoft 웹 사이트의 [Azure 알림 허브 사용자 알림](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) 페이지는 클라이언트 응용 프로그램에 비동기 응답을 푸시하기 위한 Azure 알림 허브 사용 방법을 보여줍니다.
* •	Microsoft 웹 사이트의 [API 관리](https://azure.microsoft.com/services/api-management/) 페이지에서는 웹 API에 대한 안전하고 제한된 액세스를 제공하는 제품 게시 방법을 설명합니다.
* •	Microsoft 웹 사이트의 [Azure API Management REST API 리퍼런스](https://msdn.microsoft.com/library/azure/dn776326.aspx) 페이지에서는 사용자 지정 관리 응용 프로그램 구축을 위한 API Management REST API 사용 방법을 설명합니다.
* •	Microsoft 웹 사이트의 [트래픽 관리자 라우팅 메서드](/azure/traffic-manager/traffic-manager-routing-methods/) 페이지에서는 웹 API를 호스팅하는 웹 사이트의 여러 인스턴스 전반으로 요청을 부하 분산하는 데 Azure 트래픽 관리자를 사용하는 방법을 요약합니다.
* •	Microsoft 웹 사이트의 [응용 프로그램 인사이트 - ASP.NET으로 시작하기](/azure/application-insights/app-insights-asp-net/) 페이지는 ASP.NET Web API 프로젝트에서 응용 프로그램 인사이트 설치 및 구성에 대한 자세한 정보를 제공합니다.
* •	Microsoft 웹 사이트의 [단위 테스트를 사용하여 코드 확인](https://msdn.microsoft.com/library/dd264975.aspx) 페이지는 Visual Studio를 사용한 단위 테스트 생성 및 관리에 대한 자세한 정보를 제공합니다.
* •	Microsoft 웹 사이트의 [릴리스 전 응용 프로그램에 대한 성능 테스트 실행](https://msdn.microsoft.com/library/dn250793.aspx) 페이지는 웹 성능 및 부하 분산 테스트 프로젝트를 생성하기 위한 Visual Studio Ultimate 사용 방법을 설명합니다.
