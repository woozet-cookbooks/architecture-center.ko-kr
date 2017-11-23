---
title: Authentication in multitenant applications
description: How a multitenant application can authenticate users from Azure AD
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 05/23/2016
ms.author: pnp

pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: tailspin
pnp.series.next: claims
---
# Azure AD와 OpenID Connect를 사용한 인증

[![GitHub](../_images/github.png) 샘플 코드][sample application]

Survey 응용 프로그램은 Azure Active Directory가 있는 사용자를 인증하기 위해서 OpenID Connect (OIDC) 프로토콜을 사용합니다. Survey 응용 프로그램은 OIDC에 대한 기본 미들웨어가 있는 ASP.NET Core 1.0으로 구성됩니다. 다음 다이어그램은 사용자가 상위 수준에서 로그인할 때 일어나는 작업을 보여줍니다. 

![Authentication flow](./images/auth-flow.png)

1.	사용자는 앱에서 "로그인" 단추를 클릭합니다. 이 동작은 MVC 컨트롤러에서 처리합니다.
2. MVC 컨트롤러는 **ChallengeResult** 동작을 반환합니다.
3. 미들웨어가 **ChallengeResult** 를 가로채어 302 응답을 만들어서 사용자를 Azure AD 로그인 페이지로 리디렉션합니다.
4.	사용자는 Azure AD로 인증합니다.
5.	Azure AD는 응용 프로그램에 ID 토큰을 전송합니다.
6.	미들웨어는 ID 토큰을 확인합니다. 이 시점에 사용자는 응용 프로그램 안에서 인증되었습니다.
7.	미들웨어는 사용자를 응용 프로그램으로 리디렉션합니다.


## Azure AD로 앱 등록
OpenID Connect를 사용하기 위해서, SaaS 공급자는 자체 Azure AD 테넌트 안에 응용 프로그램을 등록합니다.

응용 프로그램을 등록하려면 [응용 프로그램 추가하기](/azure/active-directory/active-directory-integrating-applications/#adding-an-application) 절에 있는 [응용 프로그램과 Azure Active Directory 통합하기](/azure/active-directory/active-directory-integrating-applications/)를 참조하세요.

**구성** 페이지에서:

* 클라이언트 ID 참조.

* **응용 프로그램이 다중 테넌트입니다**에서 **예**를 선택합니다.

* **회신 URL**을 Azure AD가 인증 응답을 보내개 될 URL로 설정합니다. 사용자 앱의 기본 URL을 사용할 수 있습니다.
  
  * 참고: 호스트 이름이 사용자가 배포한 앱과 일치하기만 하면 URL 경로는 무엇이든 가능합니다.
  
  * 사용자는 다중 회신 URL을 설정할 수 있습니다. 개발하는 동안, 로컬로 앱을 실행하기 위해서 사용자는 `로컬 호스트` 주소를 사용할 수 있습니다.

* 클라이언트 암호를 생성합니다: **키**에서 **기간 선택**이라는 드롭다운을 클릭하고 1 또는 2 년 중 하나를 선택합니다. **저장**을 클릭하면 키가 보입니다. 구성 페이지를 재로드하면 그 값이 다시 보이지 않기 때문에 반드시 복사해둡니다.

## 인증 미들웨어 구성
이 절은 다중 테넌트 인증에 대한 ASP.NET Core 1.0에서 OpenID Connect로 인증 미들웨어를 구성하는 방법에 대해 설명합니다.

사용자의 시작 클래스에 OpenID Connect 미들웨어를 추가합니다. 

```csharp
app.UseOpenIdConnectAuthentication(options =>
{
    options.AutomaticAuthenticate = true;
    options.AutomaticChallenge = true;
    options.ClientId = [client ID];
    options.Authority = "https://login.microsoftonline.com/common/";
    options.CallbackPath = [callback path];
    options.PostLogoutRedirectUri = [application URI];
    options.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = false
    };
    options.Events = [event callbacks];
});
```

> [!참고]
> [Startup.cs](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Web/Startup.cs)를 참조하세요.
> 
> 

시작 클래스에 대한 자세한 내용은 ASP.NET Core 1.0 문서에서 [응용 프로그램 시작](https://docs.asp.net/en/latest/fundamentals/startup.html)을 참조하세요.

다음의 미들웨어 옵션을 설정합니다:

* **ClientId**. Azure AD에서 응용 프로그램을 등록할 때 받은 응용 프로그램의 클라이언트 ID입니다.

* **Authority**. 다중 테넌트 응용 프로그램의 경우, 이 옵션을 https://login.microsoftonline.com/common/으로 설정합니다. Azure AD 공통 끝점에 대한 이 URL에서는, 어떤 Azure AD 테넌트의 사용자들도 로그인할 수 있습니다. 공통 끝점에 대한 자세한 정보는 [이 블로그 포스트](http://www.cloudidentity.com/blog/2014/08/26/the-common-endpoint-walks-like-a-tenant-talks-like-a-tenant-but-is-not-a-tenant/)를 참조하세요.

* **TokenValidationParameters**에서 **ValidateIssuer** 를 false로 설정합니다. 이것은 앱이 ID 토큰에서 발급자 값을 확인해야 함을 의미합니다. (미들웨어는 계속해서 토큰 그 자체를 확인합니다.) 발급자 확인에 관한 자세한 정보는 [발급자 확인](claims.md#issuer-validation)을 참조하세요.

* **CallbackPath**. 이 옵션을 Azure AD에 등록한 회신 URL에 있는 경로와 동일하게 설정합니다. 예를 들면, 회신 URL이 `http://contoso.com/aadsignin`인 경우, **CallbackPath**는 `aadsignin`이어야 합니다. 이 옵션을 설정하지 않은 경우, 기본값은 `signin-oidc`입니다.

* **PostLogoutRedirectUri**. 로그아웃 후 사용자를 리디렉션할 URL을 지정합니다. URL은 익명 요청을 허용하는 페이지여야 합니다 - 일반적으로 홈페이지.

* **SignInScheme**. 이 옵션을 `CookieAuthenticationDefaults.AuthenticationScheme`.으로 설정합니다. 이 설정은 사용자 인증 후 사용자 클레임이 쿠키에 로컬로 저장됨을 의미합니다. 이 쿠키 때문에 사용자가 브라우저 세션에서 로그인을 유지할 수 있습니다.

* **Events.** 이벤트 호출; [인증 이벤트](#authentication-events)를 참고하세요.

쿠키 인증 미들웨어 또한 파이프라인에 추가합니다. 이 미들웨어는 쿠키에 사용자 클레임을 쓴 다음, 후속 페이지가 로드될 때 쿠키를 읽는 일을 합니다.

```csharp
app.UseCookieAuthentication(options =>
{
    options.AutomaticAuthenticate = true;
    options.AutomaticChallenge = true;
    options.AccessDeniedPath = "/Home/Forbidden";
});
```

## 인증 흐름 초기화
ASP.NET MVC에서 인증 흐름을 시작하려면, 컨트롤러에서 **ChallengeResult**를 반환합니다:

```csharp
[AllowAnonymous]
public IActionResult SignIn()
{
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties
        {
            IsPersistent = true,
            RedirectUri = Url.Action("SignInCallback", "Account")
        });
}
```

이 코드는 미들웨어로 하여금 인증 끝점으로 리디렉션한 302(찾음) 응답을 반환하게 합니다.

## 사용자 로그인 세션
앞서 언급했듯이, 사용자가 처음 로그인하면 쿠키 인증 미들웨어가 사용자 클레임을 쿠키에 씁니다. 그런 다음, 쿠키 읽기에 의해 HTTP 요청이 인증됩니다.

기본값으로, 쿠키 미들웨어는 [세션 쿠키][session-cookie]에 쓰는데, 이 값은 사용자가 브라우저를 닫으면 지워집니다. 다음에 사이트를 방문할 때 사용자는 다시 로그인해야 합니다. 그러나, 사용자가 **ChallengeResult**에서 **IsPersistent** 를 true로 설정할 경우, 미들웨어는 영구적 쿠키에 쓰기 때문에 브라우저를 닫은 후에도 사용자는 로그인 상태를 유지합니다.  사용자는 쿠키 만료를 구성할 수 있습니다; [쿠키 옵션 제어하기][cookie-options]를 참조하세요. 영구적 쿠키는 사용자에게 더 편리하지만, 사용자가 매번 로그인하기를 원하는 일부 응용 프로그램에는(은행 응용 프로그램 같은) 부적절할 수 있습니다.

## OpenID Connect 미들웨어에 대하여
ASP.NET의 OpenID Connect 미들웨어는 대부분의 프로토콜 세부 정보를 숨깁니다. 이 절은 구현에 대한 참고 사항 일부를 포함하고 있으며, 이는 프로토콜 흐름을 이해하는 데 유용할 수 있습니다.

첫째, ASP.NET 관점에서(앱과 Azure AD 간에 OIDC 프로토콜 흐름에 관한 세부 정보를 무시하고) 인증 흐름을 검토해봅시다. 다음 다이어그램은 그 프로세스를 보여줍니다.

![Sign-in flow](./images/sign-in-flow.png)

다이어그램에는 두 개의 MVC 컨트롤러가 있습니다. 계정 컨트롤러는 로그인 요청을 처리하고, Home 컨트롤러는 홈페이지를 준비합니다.

다음은 인증 프로세스입니다:

1. 사용자가 "로그인" 단추를 클릭하면 브라우저는 GET 요청을 보냅니다. 예: `GET /Account/SignIn/`.
2. 계정 컨트롤러는 `ChallengeResult`를 반환합니다.
3.	OIDC 미들웨어는 Azure AD로 리디렉션하면서 HTTP 302 응답을 반환합니다. 
4.	브라우저는 Azure AD에 인증 요청을 보냅니다.
5.	사용자가 Azure AD에 로그인하고 Azure AD가 인증 응답을 되돌려보냅니다.
6.	OIDC 미들웨어는 클레임 주체를 만들어서 쿠키 인증 미들웨어에 전달합니다.
7.	쿠키 미들웨어는 클레임 주체를 직렬화하고 쿠키를 설정합니다.
8.	OIDC 미들웨어는 응용 프로그램의 호출 URL로 리디렉션합니다.
9.	브라우저는 리디렉션을 뒤따르고, 요청에 쿠키를 넣어 전송합니다.
10. 쿠키 미들웨어는 쿠키를 클레임 주체로 역직렬화하고 `HttpContext.User`를 클레임 주체와 동일하게 설정합니다. 요청이 MVC 컨트롤러로 경로 설정되었습니다.

### 인증 티켓
인증이 성공한 경우, OIDC 미들웨어는 인증 티켓을 만드는데, 인증 티켓은 사용자 클레임을 보유한 클레임 주체를 포함하고 있습니다. 사용자는 **AuthenticationValidated** 또는 **TicketReceived** 이벤트에서 티켓을 액세스할 수 있습니다.

> [!참고]
> 전체 인증 흐름이 완료될 때까지, `HttpContext.User` 는 인증된 사용자가 아닌 익명 계정을 계속 보유하고 있습니다. 익명 계정은 빈 클레임 컬렉션을 갖고 있습니다. 인증이 완료되고 앱이 리디렉션된 후, 쿠키 미들웨어는 인증 쿠키를 역직렬화하고 `HttpContext.User`를 인증된 사용자를 나타내는 클레임 주체로 설정합니다.
> 
> 

### 인증 이벤트
인증 프로세스 중에, OpenID Connect 미들웨어가 연속 이벤트를 발생시킵니다.

* **RedirectToAuthenticationEndpoint**. 미들웨어가 인증 끝점으로 리디렉션되기 직전에 호출됩니다. 리디렉션 URL을 수정하기 위해서 이 이벤트를 사용할 수 있습니다; 예를 들면 요청 매개변수를 추가하기 위해서. 예를 보려면 [관리자 동의 확인 프롬프트 추가하기](signup.md#adding-the-admin-consent-prompt)를 참조하세요.

* **AuthorizationResponseReceived**. 미들웨어가 ID 공급자에서(IDP) 인증 응답을 받은 후에, 단, 미들웨어가 그 응답을 확인하기 전에 호출됩니다. 

* **AuthorizationCodeReceived**. 인증 코드를 가지고 호출됩니다.

* **TokenResponseReceived**. 미들웨어가 IDP에서 액세스 토큰을 받은 후 호출됩니다. 인증 코드 흐름에만 적용됩니다.

* **AuthenticationValidated**. 미들웨어가 ID 토큰을 확인한 후 호출됩니다. 이 시점에 응용 프로그램은 사용자에 대해서 확인된 클레임 집합을 갖습니다. 사용자는 클레임을 추가로 확인하거나 클레임을 변환하는 데 이 이벤트를 사용할 수 있습니다. [클레임 작업](claims.md)을 참조하세요.

* **UserInformationReceived**. 미들웨어가 사용자 정보 끝점에서 사용자 프로필을 받은 경우 호출됩니다. 인증 코드 흐름에만 적용되고, 또 미들웨어 옵션에서 `GetClaimsFromUserInfoEndpoint = true`일 때만 적용됩니다.

* **TicketReceived**. 인증이 완료될 때 호출됩니다. 그 인증이 성공한다고 가정하면, 이것이 마지막 이벤트입니다. 이 이벤트가 처리되고 나면 사용자는 앱에 로그인됩니다.

* **AuthenticationFailed**. 인증이 실패할 경우 호출됩니다. 인증 실패를 처리하기 위해서 이 이벤트를 사용합니다 - 예를 들면, 오류 페이지으로 리디렉션 하는 방법으로.

이 이벤트에 대한 호출을 제공하기 위해서, 미들웨어에서 **Events** 옵션을 설정합니다. 이벤트 처리기를 선언하는 2가지 방법이 있습니다: 람다를 따라 선언하는 방법 또는 **OpenIdConnectEvents**에서 파생된 클래스에서 선언하는 방법.

람다를 따라:

```csharp
app.UseOpenIdConnectAuthentication(options =>
{
    // Other options not shown.

    options.Events = new OpenIdConnectEvents
    {
        OnTicketReceived = (context) =>
        {
             // Handle event
             return Task.FromResult(0);
        },
        // other events
    }
});
```

Deriving from **OpenIdConnectEvents**:

```csharp
public class SurveyAuthenticationEvents : OpenIdConnectEvents
{
    public override Task TicketReceived(TicketReceivedContext context)
    {
        // Handle event
        return base.TicketReceived(context);
    }
    // other events
}

// In Startup.cs:
app.UseOpenIdConnectAuthentication(options =>
{
    // Other options not shown.

    options.Events = new SurveyAuthenticationEvents();
});
```

두 번째 접근 방식은 이벤트 호출의 논리가 충분해서 시작 클래스를 어지럽히지 않을 경우에 추천합니다. 우리의 참조 구현은 이 접근 방식을 사용합니다; [SurveyAuthenticationEvents.cs](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Web/Security/SurveyAuthenticationEvents.cs)를 참조하세요.

### OpenID connect 끝점
Azure AD는 [OpenID Connect 검색](https://openid.net/specs/openid-connect-discovery-1_0.html)을 지원하는데, 여기에서 ID 공급자(IDP)는 [잘 알려진 끝점](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig)에서 JSON 메타데이터 문서를 반환합니다. 메타데이터 문서에는 다음과 같은 정보가 있습니다:

•	인증 끝점의 URL. 이 URL은 앱이 사용자를 인증하기 위해 리디렉션하는 곳입니다.

•	앱이 사용자를 로그아웃하기 위해서 가는 "세션 종료" 끝점의 URL입니다.

•	서명 키를 받는 URL로, 고객이 IDP로부터 받은 OIDC 토큰을 확인하기 위해서 사용합니다.


기본값으로, OIDC 미들웨어는 이 메타데이터를 가져오는 방법을 알고 있습니다. 미들웨어에서 **인증기관** 옵션을 설정하면, 미들웨어는 메타데이터에 대한 URL을 생성합니다. (사용자는 **MetadataAddress** 옵션을 설정하여 메타데이터 URL을 재정의할 수 있습니다.)

### OpenID connect 흐름
기본값으로, OIDC 미들웨어는 하이브리드 흐름을 폼 게시 응답 모드에서 사용합니다.

* *하이브리드 흐름*이란 클라이언트가 인증 서버와 같은 왕복 시간으로 ID 토큰과 인증 코드를 받는 것을 의미합니다.

* *폼 게시 응답 모드*란 인증 서버가 HTTP POST 요청을 사용하여 ID 토큰과 인증 코드를 앱에 전송하는 것을 의미합니다. 그 값은 form-urlencoded (콘텐츠 형식 = "application/x-www-form-urlencoded")입니다.

OIDC 미들웨어가 인증 끝점으로 리디렉션할 때, 리디렉션 URL은 OIDC에 필요한 모든 쿼리 문자열 매개변수를 포함합니다. 하이브리드 흐름의 경우:

•	client_id.. 이 값은 **ClientId** 옵션에서 설정됩니다.

•	scope = "openid profile"은 OIDC 요청이고 사용자 프로필을 원한다는 뜻입니다.

•	response_type = "code id_token". 하이브리드 흐름을 지정합니다.

•	response_mode = "form_post". 폼 게시 응답을 지정합니다.

다른 흐름을 지정하려면, 옵션에서 **ResponseType** 속성을 설정합니다.  예:

```csharp
app.UseOpenIdConnectAuthentication(options =>
{
    options.ResponseType = "code"; // Authorization code flow

    // Other options
}
```

[**다음**][claims]

[claims]: claims.md
[cookie-options]: https://docs.asp.net/en/latest/security/authentication/cookie.html#controlling-cookie-options
[session-cookie]: https://en.wikipedia.org/wiki/HTTP_cookie#Session_cookie
[sample application]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps
