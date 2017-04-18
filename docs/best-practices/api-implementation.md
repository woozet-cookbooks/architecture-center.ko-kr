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

    기본 라우팅 테이블은 RESTful web API에서 하위 리소스를 참조하는 요청(예: *http://www.adventure-works.com/api/customers/1/orders* (고객 1이 한 모든 주문의 세부 정보 검색))을 일치시키지 않습니다. 이러한 경우를 처리하기 위해 라우팅 테이블에 사용자 지정 경로를 추가할 수 있습니다.

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
  * The GET operation in the web API obtains the current ETag for the requested data (order 2 in the above example), and compares it to the value in the If-None-Match header.
  * If the current ETag for the requested data matches the ETag provided by the request, the resource has not changed and the web API should return an HTTP response with an empty message body and a status code of 304 (Not Modified).
  * If the current ETag for the requested data does not match the ETag provided by the request, then the data has changed and the web API should return an HTTP response with the new data in the message body and a status code of 200 (OK).
  * If the requested data no longer exists then the web API should return an HTTP response with the status code of 404 (Not Found).
  * The client uses the status code to maintain the cache. If the data has not changed (status code 304) then the object can remain cached and the client application should continue to use this version of the object. If the data has changed (status code 200) then the cached object should be discarded and the new one inserted. If the data is no longer available (status code 404) then the object should be removed from the cache.

    > [!NOTE]
    > If the response header contains the Cache-Control header no-store then the object should always be removed from the cache regardless of the HTTP status code.
    >
    >

    The code below shows the `FindOrderByID` method extended to support the If-None-Match header. Notice that if the If-None-Match header is omitted, the specified order is always retrieved:

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

    This example incorporates an additional custom `IHttpActionResult` class named `EmptyResultWithCaching`. This class simply acts as a wrapper around an `HttpResponseMessage` object that does not contain a response body:

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

    > [!TIP]
    > In this example, the ETag for the data is generated by hashing the data retrieved from the underlying data source. If the ETag can be computed in some other way, then the process can be optimized further and the data only needs to be fetched from the data source if it has changed.  This approach is especially useful if the data is large or accessing the data source can result in significant latency (for example, if the data source is a remote database).
    >
    >
* **Use ETags to Support Optimistic Concurrency**.

    To enable updates over previously cached data, the HTTP protocol supports an optimistic concurrency strategy. If, after fetching and caching a resource, the client application subsequently sends a PUT or DELETE request to change or remove the resource, it should include in If-Match header that references the ETag. The web API can then use this information to determine whether the resource has already been changed by another user since it was retrieved and send an appropriate response back to the client application as follows:

  * The client constructs a PUT request containing the new details for the resource and the ETag for the currently cached version of the resource referenced in an If-Match HTTP header. The following example shows a PUT request that updates an order:

    ```HTTP
    PUT http://adventure-works.com/orders/1 HTTP/1.1
    If-Match: "2282343857"
    Content-Type: application/x-www-form-urlencoded
    ...
    Date: Fri, 12 Sep 2014 09:18:37 GMT
    Content-Length: ...
    productID=3&quantity=5&orderValue=250
    ```
  * The PUT operation in the web API obtains the current ETag for the requested data (order 1 in the above example), and compares it to the value in the If-Match header.
  * If the current ETag for the requested data matches the ETag provided by the request, the resource has not changed and the web API should perform the update, returning a message with HTTP status code 204 (No Content) if it is successful. The response can include Cache-Control and ETag headers for the updated version of the resource. The response should always include the Location header that references the URI of the newly updated resource.
  * If the current ETag for the requested data does not match the ETag provided by the request, then the data has been changed by another user since it was fetched and the web API should return an HTTP response with an empty message body and a status code of 412 (Precondition Failed).
  * If the resource to be updated no longer exists then the web API should return an HTTP response with the status code of 404 (Not Found).
  * The client uses the status code and response headers to maintain the cache. If the data has been updated (status code 204) then the object can remain cached (as long as the Cache-Control header does not specify no-store) but the ETag should be updated. If the data was changed by another user changed (status code 412) or not found (status code 404) then the cached object should be discarded.

    The next code example shows an implementation of the PUT operation for the Orders controller:

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

    > [!TIP]
    > Use of the If-Match header is entirely optional, and if it is omitted the web API will always attempt to update the specified order, possibly blindly overwriting an update made by another user. To avoid problems due to lost updates, always provide an If-Match header.
    >
    >

<a name="considerations-for-handling-large"></a>

## Considerations for handling large requests and responses
There may be occasions when a client application needs to issue requests that send or receive data that may be several megabytes (or bigger) in size. Waiting while this amount of data is transmitted could cause the client application to become unresponsive. Consider the following points when you need to handle requests that include significant amounts of data:

* **Optimize requests and responses that involve large objects**.

    Some resources may be large objects or include large fields, such as graphics images or other types of binary data. A web API should support streaming to enable optimized uploading and downloading of these resources.

    The HTTP protocol provides the chunked transfer encoding mechanism to stream large data objects back to a client. When the client sends an HTTP GET request for a large object, the web API can send the reply back in piecemeal *chunks* over an HTTP connection. The length of the data in the reply may not be known initially (it might be generated), so the server hosting the web API should send a response message with each chunk that specifies the Transfer-Encoding: Chunked header rather than a Content-Length header. The client application can receive each chunk in turn to build up the complete response. The data transfer completes when the server sends back a final chunk with zero size.    You can implement chunking in the ASP.NET Web API by using the `PushStreamContent` class.

    The following example shows an operation that responds to HTTP GET requests for product images:

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

    In this example, `ConnectBlobToContainer` is a helper method that connects to a specified container (name not shown) in Azure Blob storage. `BlobExists` is another helper method that returns a Boolean value that indicates whether a blob with the specified name exists in the blob storage container.

    Each product has its own image held in blob storage. The `FileDownloadResult` class is a custom `IHttpActionResult` class that uses a `PushStreamContent` object to read the image data from appropriate blob and transmit it asynchronously as the content of the response message:

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

    You can also apply streaming to upload operations if a client needs to POST a new resource that includes a large object. The next example shows the Post method for the `ProductImages` controller. This method enables the client to upload a new product image:

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

    This code uses another custom `IHttpActionResult` class called `FileUploadResult`. This class contains the logic for uploading the data asynchronously:

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

  > [!TIP]
  > The volume of data that you can upload to a web service is not constrained by streaming, and a single request could conceivably result in a massive object that consumes considerable resources. If, during the streaming process, the web API determines that the amount of data in a request has exceeded some acceptable bounds, it can abort the operation and return a response message with status code 413 (Request Entity Too Large).
  >
  >

    You can minimize the size of large objects transmitted over the network by using HTTP compression. This approach helps to reduce the amount of network traffic and the associated network latency, but at the cost of requiring additional processing at the client and the server hosting the web API. For example, a client application that expects to receive compressed data can include an Accept-Encoding: gzip request header (other data compression algorithms can also be specified). If the server supports compression it should respond with the content held in gzip format in the message body and the Content-Encoding: gzip response header.

  > [!TIP]
  > You can combine encoded compression with streaming; compress the data first before streaming it, and specify the gzip content encoding and chunked transfer encoding in the message headers. Also note that some web servers (such as Internet Information Server) can be configured to automatically compress HTTP responses regardless of whether the web API compresses the data or not.
  >
  >
* **Implement partial responses for clients that do not support asynchronous operations**.

    As an alternative to asynchronous streaming, a client application can explicitly request data for large objects in chunks, known as partial responses. The client application sends an HTTP HEAD request to obtain information about the object. If the web API supports partial responses if should respond to the HEAD request with a response message that contains an Accept-Ranges header and a Content-Length header that indicates the total size of the object, but the body of the message should be empty. The client application can use this information to construct a series of GET requests that specify a range of bytes to receive. The web API should return a response message with HTTP status 206 (Partial Content), a Content-Length header that specifies the actual amount of data included in the body of the response message, and a Content-Range header that indicates which part (such as bytes 4000 to 8000) of the object this data represents.

    HTTP HEAD requests and partial responses are described in more detail in the API Design Guidance document.
* **Avoid sending unnecessary Continue status messages in client applications**.

    A client application that is about to send a large amount of data to a server may determine first whether the server is actually willing to accept the request. Prior to sending the data, the client application can submit an HTTP request with an Expect: 100-Continue header, a Content-Length header that indicates the size of the data, but an empty message body. If the server is willing to handle the request, it should respond with a message that specifies the HTTP status 100 (Continue). The client application can then proceed and send the complete request including the data in the message body.

    If you are hosting a service by using IIS, the HTTP.sys driver automatically detects and handles Expect: 100-Continue headers before passing requests to your web application. This means that you are unlikely to see these headers in your application code, and you can assume that IIS has already filtered any messages that it deems to be unfit or too large.

    If you are building client applications by using the .NET Framework, then all POST and PUT messages will first send messages with Expect: 100-Continue headers by default. As with the server-side, the process is handled transparently by the .NET Framework. However, this process results in each POST and PUT request causing 2 round-trips to the server, even for small requests. If your application is not sending requests with large amounts of data, you can disable this feature by using the `ServicePointManager` class to create `ServicePoint` objects in the client application. A `ServicePoint` object handles the connections that the client makes to a server based on the scheme and host fragments of URIs that identify resources on the server. You can then set the `Expect100Continue` property of the `ServicePoint` object to false. All subsequent POST and PUT requests made by the client through a URI that matches the scheme and host fragments of the `ServicePoint` object will be sent without Expect: 100-Continue headers. The following code shows how to configure a `ServicePoint` object that configures all requests sent to URIs with a scheme of `http` and a host of `www.contoso.com`.

    ```C#
    Uri uri = new Uri("http://www.contoso.com/");
    ServicePoint sp = ServicePointManager.FindServicePoint(uri);
    sp.Expect100Continue = false;
    ```

    You can also set the static `Expect100Continue` property of the `ServicePointManager` class to specify the default value of this property for all subsequently created `ServicePoint` objects. For more information, see the [ServicePoint Class](https://msdn.microsoft.com/library/system.net.servicepoint.aspx) page on the Microsoft website.
* **Support pagination for requests that may return large numbers of objects**.

    If a collection contains a large number of resources, issuing a GET request to the corresponding URI could result in significant processing on the server hosting the web API affecting performance, and generate a significant amount of network traffic resulting in increased latency.

    To handle these cases, the web API should support query strings that enable the client application to refine requests or fetch data in more manageable, discrete blocks (or pages). The ASP.NET Web API framework parses query strings and splits them up into a series of parameter/value pairs which are passed to the appropriate method, following the routing rules described earlier. The method should be implemented to accept these parameters using the same names specified in the query string. Additionally, these parameters should be optional (in case the client omits the query string from a request) and have meaningful default values. The code below shows the `GetAllOrders` method in the `Orders` controller. This method retrieves the details of orders. If this method was unconstrained, it could conceivably return a large amount of data. The `limit` and `offset` parameters are intended to reduce the volume of data to a smaller subset, in this case only the first 10 orders by default:

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

    A client application can issue a request to retrieve 30 orders starting at offset 50 by using the URI *http://www.adventure-works.com/api/orders?limit=30&offset=50*.

  > [!TIP]
  > Avoid enabling client applications to specify query strings that result in a URI that is more than 2000 characters long. Many web clients and servers cannot handle URIs that are this long.
  >
  >

<a name="considerations-for-maintaining-responsiveness"></a>

## Considerations for maintaining responsiveness, scalability, and availability
The same web API might be utilized by many client applications running anywhere in the world. It is important to ensure that the web API is implemented to maintain responsiveness under a heavy load, to be scalable to support a highly varying workload, and to guarantee availability for clients that perform business-critical operations. Consider the following points when determining how to meet these requirements:

* **Provide Asynchronous Support for Long-Running Requests**.

    A request that might take a long time to process should be performed without blocking the client that submitted the request. The web API can perform some initial checking to validate the request, initiate a separate task to perform the work, and then return a response message with HTTP code 202 (Accepted). The task could run asynchronously as part of the web API processing, or it could be offloaded to an Azure WebJob (if the web API is hosted by an Azure Website) or a worker role (if the web API is implemented as an Azure cloud service).

  > [!NOTE]
  > For more information about using WebJobs with Azure Website, visit the page [Use WebJobs to run background tasks in Microsoft Azure Websites](/azure/app-service-web/web-sites-create-web-jobs/) on the Microsoft website.
  >
  >

    The web API should also provide a mechanism to return the results of the processing to the client application. You can achieve this by providing a polling mechanism for client applications to periodically query whether the processing has finished and obtain the result, or enabling the web API to send a notification when the operation has completed.

    You can implement a simple polling mechanism by providing a *polling* URI that acts as a virtual resource using the following approach:

  1. The client application sends the initial request to the web API.
  2. The web API stores information about the request in a table held in table storage or Microsoft Azure Cache, and generates a unique key for this entry, possibly in the form of a GUID.
  3. The web API initiates the processing as a separate task. The web API records the state of the task in the table as *Running*.
  4. The web API returns a response message with HTTP status code 202 (Accepted), and the GUID of the table entry in the body of the message.
  5. When the task has completed, the web API stores the results in the table, and sets the state of the task to *Complete*. Note that if the task fails, the web API could also store information about the failure and set the status to *Failed*.
  6. While the task is running, the client can continue performing its own processing. It can periodically send a request to the URI */polling/{guid}* where *{guid}* is the GUID returned in the 202 response message by the web API.
  7. The web API at the */polling/{guid}* URI queries the state of the corresponding task in the table and returns a response message with HTTP status code 200 (OK) containing this state (*Running*, *Complete*, or *Failed*). If the task has completed or failed, the response message can also include the results of the processing or any information available about the reason for the failure.

     If you prefer to implement notifications, the options available include:
  8. Using an Azure Notification Hub to push asynchronous responses to client applications. The page [Azure Notification Hubs Notify Users](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) on the Microsoft website provides further details.
  9. Using the Comet model to retain a persistent network connection between the client and the server hosting the web API, and using this connection to push messages from the server back to the client. The MSDN magazine article [Building a Simple Comet Application in the Microsoft .NET Framework](https://msdn.microsoft.com/magazine/jj891053.aspx) describes an example solution.
  10. Using SignalR to push data in real-time from the web server to the client over a persistent network connection. SignalR is available for ASP.NET web applications as a NuGet package. You can find more information on the [ASP.NET SignalR](http://signalr.net/) website.

      > [!NOTE]
      > Comet and SignalR both utilize persistent network connections between the web server and the client application. This can affect scalability as a large number of clients may require an equally large number of concurrent connections.
      >
      >
* **Ensure that each request is stateless**.

    Each request should be considered atomic. There should be no dependencies between one request made by a client application and any subsequent requests submitted by the same client. This approach assists in scalability; instances of the web service can be deployed on a number of servers. Client requests can be directed at any of these instances and the results should always be the same. It also improves availability for a similar reason; if a web server fails requests can be routed to another instance (by using Azure Traffic Manager) while the server is restarted with no ill effects on client applications.
* **Track clients and implement throttling to reduce the chances of DOS attacks**.

    If a specific client makes a large number of requests within a given period of time it might monopolize the service and affect the performance of other clients. To mitigate this issue, a web API can monitor calls from client applications either by tracking the IP address of all incoming requests or by logging each authenticated access. You can use this information to limit resource access. If a client exceeds a defined limit, the web API can return a response message with status 503 (Service Unavailable) and include a Retry-After header that specifies when the client can send the next request without it being declined. This strategy can help to reduce the chances of a Denial Of Service (DOS) attack from a set of clients stalling the system.
* **Manage persistent HTTP connections carefully**.

    The HTTP protocol supports persistent HTTP connections where they are available. The HTTP 1.0 specificiation added the Connection:Keep-Alive header that enables a client application to indicate to the server that it can use the same connection to send subsequent requests rather than opening new ones. The connection closes automatically if the client does not reuse the connection within a period defined by the host. This behavior is the default in HTTP 1.1 as used by Azure services, so there is no need to include Keep-Alive headers in messages.

    Keeping a connection open can help to improve responsiveness by reducing latency and network congestion, but it can be detrimental to scalability by keeping unnecessary connections open for longer than required, limiting the ability of other concurrent clients to connect. It can also affect battery life if the client application is running on a mobile device; if the application only makes occasional requests to the server, maintaining an open connection can cause the battery to drain more quickly. To ensure that a connection is not made persistent with HTTP 1.1, the client can include a Connection:Close header with messages to override the default behavior. Similarly, if a server is handling a very large number of clients it can include a Connection:Close header in response messages which should close the connection and save server resources.

  > [!NOTE]
  > Persistent HTTP connections are a purely optional feature to reduce the network overhead associated with repeatedly establishing a communications channel. Neither the web API nor the client application should depend on a persistent HTTP connection being available. Do not use persistent HTTP connections to implement Comet-style notification systems; instead you should utilize sockets (or websockets if available) at the TCP layer. Finally, note Keep-Alive headers are of limited use if a client application communicates with a server via a proxy; only the connection with the client and the proxy will be persistent.
  >
  >

## Considerations for publishing and managing a web API
To make a web API available for client applications, the web API must be deployed to a host environment. This environment is typically a web server, although it may be some other type of host process. You should consider the following points when publishing a web API:

* All requests must be authenticated and authorized, and the appropriate level of access control must be enforced.
* A commercial web API might be subject to various quality guarantees concerning response times. It is important to ensure that host environment is scalable if the load can vary significantly over time.
* If may be necessary to meter requests for monetization purposes.
* It might be necessary to regulate the flow of traffic to the web API, and implement throttling for specific clients that have exhausted their quotas.
* Regulatory requirements might mandate logging and auditing of all requests and responses.
* To ensure availability, it may be necessary to monitor the health of the server hosting the web API and restart it if necessary.

It is useful to be able to decouple these issues from the technical issues concerning the implementation of the web API. For this reason, consider creating a [façade](http://en.wikipedia.org/wiki/Facade_pattern), running as a separate process and that routes requests to the web API. The façade can provide the management operations and forward validated requests to the web API. Using a façade can also bring many functional advantages, including:

* Acting as an integration point for multiple web APIs.
* Transforming messages and translating communications protocols for clients built by using varying technologies.
* Caching requests and responses to reduce load on the server hosting the web API.

## Considerations for testing a web API
A web API should be tested as thoroughly as any other piece of software. You should consider creating unit tests to validate the functionality of each operation, as you would with any other type of application. For more information, see the page [Verifying Code by Using Unit Tests](https://msdn.microsoft.com/library/dd264975.aspx) on the Microsoft website.

> [!NOTE]
> The sample web API available with this guidance includes a test project that shows how to perform unit testing over selected operations.
>
>

The nature of a web API brings its own additional requirements to verify that it operates correctly. You should pay particular attention to the following aspects:

* Test all routes to verify that they invoke the correct operations. Be especially aware of HTTP status code 405 (Method Not Allowed) being returned unexpectedly as this can indicate a mismatch between a route and the HTTP methods (GET, POST, PUT, DELETE) that can be dispatched to that route.

    Send HTTP requests to routes that do not support them, such as submitting a POST request to a specific resource (POST requests should only be sent to resource collections). In these cases, the only valid response *should* be status code 405 (Not Allowed).
* Verify that all routes are protected properly and are subject to the appropriate authentication and authorization checks.

  > [!NOTE]
  > Some aspects of security such as user authentication are most likely to be the responsibility of the host environment rather than the web API, but it is still necessary to include security tests as part of the deployment process.
  >
  >
* Test the exception handling performed by each operation and verify that an appropriate and meaningful HTTP response is passed back to the client application.
* Verify that request and response messages are well-formed. For example, if an HTTP POST request contains the data for a new resource in x-www-form-urlencoded format, confirm that the corresponding operation correctly parses the data, creates the resources, and returns a response containing the details of the new resource, including the correct Location header.
* Verify all links and URIs in response messages. For example, an HTTP POST message should return the URI of the newly-created resource. All HATEOAS links should be valid.

  > [!IMPORTANT]
  > If you publish the web API through an API Management Service, then these URIs should reflect the URL of the management service and not that of the web server hosting the web API.
  >
  >
* Ensure that each operation returns the correct status codes for different combinations of input. For example:

  * If a query is successful, it should return status code 200 (OK)
  * If a resource is not found, the operation should return HTTP status code 404 (Not Found).
  * If the client sends a request that successfully deletes a resource, the status code should be 204 (No Content).
  * If the client sends a request that creates a new resource, the status code should be 201 (Created)

Watch out for unexpected response status codes in the 5xx range. These messages are usually reported by the host server to indicate that it was unable to fulfill a valid request.

* Test the different request header combinations that a client application can specify and ensure that the web API returns the expected information in response messages.
* Test query strings. If an operation can take optional parameters (such as pagination requests), test the different combinations and order of parameters.
* Verify that asynchronous operations complete successfully. If the web API supports streaming for requests that return large binary objects (such as video or audio), ensure that client requests are not blocked while the data is streamed. If the web API implements polling for long-running data modification operations, verify that that the operations report their status correctly as they proceed.

You should also create and run performance tests to check that the web API operates satisfactorily under duress. You can build a web performance and load test project by using Visual Studio Ultimate. For more information, see the page [Run performance tests on an application before a release](https://msdn.microsoft.com/library/dn250793.aspx) on the Microsoft website.

## Publishing and managing a web API by using the Azure API Management Service
Azure provides the [API Management Service](https://azure.microsoft.com/documentation/services/api-management/) which you can use to publish and manage a web API. Using this facility, you can generate a service that acts a façade for one or more web APIs. The service is itself a scalable web service that you can create and configure by using the Azure Management portal. You can use this service to publish and manage a web API as follows:

1. Deploy the web API to a website, Azure cloud service, or Azure virtual machine.
2. Connect the API management service to the web API. Requests sent to the URL of the management API are mapped to URIs in the web API. The same API management service can route requests to more than one web API. This enables you to aggregate multiple web APIs into a single management service. Similarly, the same web API can be referenced from more than one API management service if you need to restrict or partition the functionality available to different applications.

   > [!NOTE]
   > The URIs in HATEOAS links generated as part of the response for HTTP GET requests should reference the URL of the API management service and not the web server hosting the web API.
   >
   >
3. For each web API, specify the HTTP operations that the web API exposes together with any optional parameters that an operation can take as input. You can also configure whether the API management service should cache the response received from the web API to optimize repeated requests for the same data. Record the details of the HTTP responses that each operation can generate. This information is used to generate documentation for developers, so it is important that it is accurate and complete.

    You can either define operations manually using the wizards provided by the Azure Management portal, or you can import them from a file containing the definitions in WADL or Swagger format.
4. Configure the security settings for communications between the API management service and the web server hosting the web API. The API management service currently supports Basic authentication and mutual authentication using certificates, and OAuth 2.0 user authorization.
5. Create a product. A product is the unit of publication; you add the web APIs that you previously connected to the management service to the product. When the product is published, the web APIs become available to developers.

   > [!NOTE]
   > Prior to publishing a product, you can also define user-groups that can access the product and add users to these groups. This gives you control over the developers and applications that can use the web API. If a web API is subject to approval, prior to being able to access it a developer must send a request to the product administrator. The administrator can grant or deny access to the developer. Existing developers can also be blocked if circumstances change.
   >
   >
6. Configure policies for each web API. Policies govern aspects such as whether cross-domain calls should be allowed, how to authenticate clients, whether to convert between XML and JSON data formats transparently, whether to restrict calls from a given IP range, usage quotas, and whether to limit the call rate. Policies can be applied globally across the entire product, for a single web API in a product, or for individual operations in a web API.

You can find full details describing how to perform these tasks on the [API Management](https://azure.microsoft.com/services/api-management/) page on the Microsoft website. The Azure API Management Service also provides its own REST interface, enabling you to build a custom interface for simplifying the process of configuring a web API. For more information, visit the [Azure API Management REST API Reference](https://msdn.microsoft.com/library/azure/dn776326.aspx) page on the Microsoft website.

> [!TIP]
> Azure provides the Azure Traffic Manager which enables you to implement failover and load-balancing, and reduce latency across multiple instances of a web site hosted in different geographic locations. You can use Azure Traffic Manager in conjunction with the API Management Service; the API Management Service can route requests to instances of a web site through Azure Traffic Manager.  For more information, visit the [Traffic Manager routing Methods](/azure/traffic-manager/traffic-manager-routing-methods/) page on the Microsoft website.
>
> In this structure, if you are using custom DNS names for your web sites, you should configure the appropriate CNAME record for each web site to point to the DNS name of the Azure Traffic Manager web site.
>
>

## Supporting developers building client applications
Developers constructing client applications typically require information on how to access the web API, and documentation concerning the parameters, data types, return types, and return codes that describe the different requests and responses between the web service and the client application.

### Documenting the REST operations for a web API
The Azure API Management Service includes a developer portal that describes the REST operations exposed by a web API. When a product has been published it appears on this portal. Developers can use this portal to sign up for access; the administrator can then approve or deny the request. If the developer is approved, they are assigned a subscription key that is used to authenticate calls from the client applications that they develop. This key must be provided with each web API call otherwise it will be rejected.

This portal also provides:

* Documentation for the product, listing the operations that it exposes, the parameters required, and the different responses that can be returned. Note that this information is generated from the details provided in step 3 in the list in the Publishing a web API by using the Microsoft Azure API Management Service section.
* Code snippets that show how to invoke operations from several languages, including JavaScript, C#, Java, Ruby, Python, and PHP.
* A developers' console that enables a developer to send an HTTP request to test each operation in the product and view the results.
* A page where the developer can report any issues or problems found.

The Azure Management portal enables you to customize the developer portal to change the styling and layout to match the branding of your organization.

### Implementing a client SDK
Building a client application that invokes REST requests to access a web API requires writing a significant amount of code to construct each request and format it appropriately, send the request to the server hosting the web service, and parse the response to work out whether the request succeeded or failed and extract any data returned. To insulate the client application from these concerns, you can provide an SDK that wraps the REST interface and abstracts these low-level details inside a more functional set of methods. A client application uses these methods, which transparently convert calls into REST requests and then convert the responses back into method return values. This is a common technique that is implemented by many services, including the Azure SDK.

Creating a client-side SDK is a considerable undertaking as it has to be implemented consistently and tested carefully. However, much of this process can be made mechanical, and many vendors supply tools that can automate many of these tasks.

## Monitoring a web API
Depending on how you have published and deployed your web API you can monitor the web API directly, or you can gather usage and health information by analyzing the traffic that passes through the API Management service.

### Monitoring a web API directly
If you have implemented your web API by using the ASP.NET Web API template (either as a Web API project or as a Web role in an Azure cloud service) and Visual Studio 2013, you can gather availability, performance, and usage data by using ASP.NET Application Insights. Application Insights is a package that transparently tracks and records information about requests and responses when the web API is deployed to the cloud; once the package is installed and configured, you don't need to amend any code in your web API to use it. When you deploy the web API to an Azure web site, all traffic is examined and the following statistics are gathered:

* Server response time.
* Number of server requests and the details of each request.
* The top slowest requests in terms of average response time.
* The details of any failed requests.
* The number of sessions initiated by different browsers and user agents.
* The most frequently viewed pages (primarily useful for web applications rather than web APIs).
* The different user roles accessing the web API.

You can view this data in real time from the Azure Management portal. You can also create webtests that monitor the health of the web API. A webtest sends a periodic request to a specified URI in the web API and captures the response. You can specify the definition of a successful response (such as HTTP status code 200), and if the request does not return this response you can arrange for an alert to be sent to an administrator. If necessary, the administrator can restart the server hosting the web API if it has failed.

The [Application Insights - Get started with ASP.NET](/azure/application-insights/app-insights-asp-net/) page on the Microsoft website provides more information.

### Monitoring a web API through the API Management Service
If you have published your web API by using the API Management service, the API Management page on the Azure Management portal contains a dashboard that enables you to view the overall performance of the service. The Analytics page enables you to drill down into the details of how the product is being used. This page contains the following tabs:

* **Usage**. This tab provides information about the number of API calls made and the bandwidth used to handle these calls over time. You can filter usage details by product, API, and operation.
* **Health**. This tab enables you view the outcome of API requests (the HTTP status codes returned), the effectiveness of the caching policy, the API response time, and the service response time. Again, you can filter health data by product, API, and operation.
* **Activity**. This tab provides a text summary of the numbers of successful calls, failed called, blocked calls, average response time, and response times for each product, web API, and operation. This page also lists the number of calls made by each developer.
* **At a glance**. This tab displays a summary of the performance data, including the developers responsible for making the most API calls, and the products, web APIs, and operations that received these calls.

You can use this information to determine whether a particular web API or operation is causing a bottleneck, and if necessary scale the host environment and add more servers. You can also ascertain whether one or more applications are using a disproportionate volume of resources and apply the appropriate policies to set quotas and limit call rates.

> [!NOTE]
> You can change the details for a published product, and the changes are applied immediately. For example, you can add or remove an operation from a web API without requiring that you republish the product that contains the web API.
>
>

## Related patterns
* The [façade](http://en.wikipedia.org/wiki/Facade_pattern) pattern describes how to provide an interface to a web API.

## More information
* The page [Learn About ASP.NET Web API](http://www.asp.net/web-api) on the Microsoft website provides a detailed introduction to building RESTful web services by using the Web API.
* The page [Routing in ASP.NET Web API](http://www.asp.net/web-api/overview/web-api-routing-and-actions/routing-in-aspnet-web-api) on the Microsoft website describes how convention-based routing works in the ASP.NET Web API framework.
* For more information on attribute-based routing, see the page [Attribute Routing in Web API 2](http://www.asp.net/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2) on the Microsoft website.
* The [Basic Tutorial](http://www.odata.org/getting-started/basic-tutorial/) page on the OData website provides an introduction to the features of the OData protocol.
* The [ASP.NET Web API OData](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api) page on the Microsoft website contains examples and further information on implementing an OData web API by using ASP.NET.
* The page [Introducing Batch Support in Web API and Web API OData](http://blogs.msdn.com/b/webdev/archive/2013/11/01/introducing-batch-support-in-web-api-and-web-api-odata.aspx) on the Microsoft website describes how to implement batch operations in a web API by using OData.
* The article [Idempotency Patterns](http://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog provides an overview of idempotency and how it relates to data management operations.
* The [Status Code Definitions](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) page on the W3C website contains a full list of HTTP status codes and their descriptions.
* For detailed information on handling HTTP exceptions with the ASP.NET Web API, visit the [Exception Handling in ASP.NET Web API](http://www.asp.net/web-api/overview/error-handling/exception-handling) page on the Microsoft website.
* The article [Web API Global Error Handling](http://www.asp.net/web-api/overview/error-handling/web-api-global-error-handling) on the Microsoft website describes how to implement a global error handling and logging strategy for a web API.
* The page [Run background tasks with WebJobs](/azure/app-service-web/web-sites-create-web-jobs/) on the Microsoft website provides information and examples on using WebJobs to perform background operations on an Azure Website.
* The page [Azure Notification Hubs Notify Users](/azure/notification-hubs/notification-hubs-aspnet-backend-windows-dotnet-wns-notification/) on the Microsoft website shows how you can use an Azure Notification Hub to push asynchronous responses to client applications.
* The [API Management](https://azure.microsoft.com/services/api-management/) page on the Microsoft website describes how to publish a product that provides controlled and secure access to a web API.
* The [Azure API Management REST API Reference](https://msdn.microsoft.com/library/azure/dn776326.aspx) page on the Microsoft website describes how to use the API Management REST API to build custom management applications.
* The [Traffic Manager Routing Methods](/azure/traffic-manager/traffic-manager-routing-methods/) page on the Microsoft website summarizes how Azure Traffic Manager can be used to load-balance requests across multiple instances of a website hosting a web API.
* The [Application Insights - Get started with ASP.NET](/azure/application-insights/app-insights-asp-net/) page on the Microsoft website provides detailed information on installing and configuring Application Insights in an ASP.NET Web API project.
* The page [Verifying Code by Using Unit Tests](https://msdn.microsoft.com/library/dd264975.aspx) on the Microsoft website provides detailed information on creating and managing unit tests by using Visual Studio.
* The page [Run performance tests on an application before a release](https://msdn.microsoft.com/library/dn250793.aspx) on the Microsoft website describes how to use Visual Studio Ultimate to create a web performance and load test project.
