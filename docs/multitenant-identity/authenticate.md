---
title: "다중 테넌트 응용 프로그램의 인증"
description: "다중 테넌트 응용 프로그램이 Azure AD에서 사용자를 인증하는 방법"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: tailspin
pnp.series.next: claims
ms.openlocfilehash: e85817626675cec4d126921c19a31a0983ecd62d
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/08/2017
---
# <a name="authenticate-using-azure-ad-and-openid-connect"></a>Azure AD 및 OpenID Connect를 사용하여 인증

[![GitHub](../_images/github.png) 샘플 코드][sample application]

Surveys 응용 프로그램은 Azure AD(Azure Active Directory)에 사용자를 인증하기 위해 OIDC(OpenID Connect) 프로토콜을 사용합니다. Surveys 응용 프로그램은 OIDC용 미들웨어를 기본 제공하는 ASP.NET Core를 사용합니다. 다음 다이어그램은 사용자가 로그인할 때 진행 과정을 개략적인 수준으로 보여 줍니다.

![인증 흐름](./images/auth-flow.png)

1. 사용자가 앱에서 "로그인" 단추를 클릭합니다. 이 작업은 MVC 컨트롤러에서 처리됩니다.
2. MVC 컨트롤러는 **ChallengeResult** 작업을 반환합니다.
3. 미들웨어는 **ChallengeResult** 를 가로채고 사용자를 Azure AD 로그인 페이지로 리디렉션하는 302 응답을 생성합니다.
4. 사용자는 Azure AD로 인증합니다.
5. Azure AD는 ID 토큰을 응용 프로그램으로 보냅니다.
6. 미들웨어는 ID 토큰의 유효성을 검사합니다. 이때 사용자가 응용 프로그램 내부에서 인증됩니다.
7. 미들웨어는 사용자를 응용 프로그램으로 다시 리디렉션합니다.

## <a name="register-the-app-with-azure-ad"></a>Azure AD에 앱 등록
OpenID Connect를 사용하도록 설정하려면 SaaS 공급자는 응용 프로그램을 자신의 Azure AD 테넌트 내에 등록합니다.

응용 프로그램을 등록하려면 [Azure Active Directory와 응용 프로그램 통합](/azure/active-directory/active-directory-integrating-applications/)에서 [응용 프로그램 추가](/azure/active-directory/active-directory-integrating-applications/#adding-an-application) 섹션의 단계를 따릅니다.

Surveys 응용 프로그램 관련 단계는 [Run the Surveys application](./run-the-app.md)(Surveys 응용 프로그램 실행)을 참조하세요. 다음 사항에 유의하세요.

- 다중 테넌트 응용 프로그램의 경우 다중 테넌트 옵션을 명시적으로 구성해야 합니다. 이렇게 하면 다른 조직이 응용 프로그램에 액세스할 수 있습니다.

- 회신 URL은 Azure AD에서 OAuth 2.0 응답을 보내는 URL입니다. ASP.NET Core를 사용할 경우 이 URL은 인증 미들웨어에서 구성하는 경로와 일치해야 합니다(다음 섹션 참조). 

## <a name="configure-the-auth-middleware"></a>인증 미들웨어 구성
이 섹션에서는 ASP.NET Core에서 OpenID Connect로 다중 테넌트 인증을 위한 인증 미들웨어를 구성하는 방법을 설명합니다.

[시작 클래스](/aspnet/core/fundamentals/startup)에서 OpenID Connect 미들웨어를 추가합니다.

```csharp
app.UseOpenIdConnectAuthentication(new OpenIdConnectOptions {
    ClientId = configOptions.AzureAd.ClientId,
    ClientSecret = configOptions.AzureAd.ClientSecret, // for code flow
    Authority = Constants.AuthEndpointPrefix,
    ResponseType = OpenIdConnectResponseType.CodeIdToken,
    PostLogoutRedirectUri = configOptions.AzureAd.PostLogoutRedirectUri,
    SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme,
    TokenValidationParameters = new TokenValidationParameters { ValidateIssuer = false },
    Events = new SurveyAuthenticationEvents(configOptions.AzureAd, loggerFactory),
});
```

일부 설정은 런타임 구성 옵션에서 가져옵니다. 다음은 미들웨어 옵션의 의미입니다.

* **ClientId**. Azure AD에 응용 프로그램을 등록할 때 얻은 응용 프로그램의 클라이언트 ID입니다.
* **Authority**. 다중 테넌트 응용 프로그램의 경우 `https://login.microsoftonline.com/common/`으로 설정합니다. Azure AD 공용 끝점에 대한 URL로, Azure AD 테넌트의 사용자가 이를 통해 로그인할 수 있습니다. 공용 끝점에 대한 자세한 내용은 [이 블로그 게시물](http://www.cloudidentity.com/blog/2014/08/26/the-common-endpoint-walks-like-a-tenant-talks-like-a-tenant-but-is-not-a-tenant/)을 참조하세요.
* **TokenValidationParameters**에서 **ValidateIssuer**를 false로 설정합니다. 이 경우 앱에서 ID 토큰의 발급자 값의 유효성을 검사해야 합니다. (토큰 자체의 유효성은 계속 미들웨어에서 검사합니다.) 발급자 유효성 검사에 대한 자세한 내용은 [발급자 확인](claims.md#issuer-validation)을 참조하세요.
* **PostLogoutRedirectUri**. 로그아웃한 후 사용자를 리디렉션할 URL을 지정합니다. 이 URL은 익명 요청을 허용하는 페이지여야 하며, 일반적으로 홈페이지입니다.
* **SignInScheme**. `CookieAuthenticationDefaults.AuthenticationScheme`로 설정합니다. 이 설정은 사용자가 인증된 후 사용자 클레임이 쿠키에 로컬로 저장됨을 의미합니다. 이 쿠키는 브라우저 세션 중에 사용자가 로그인을 유지하는 방법입니다.
* **이벤트.** 이벤트 콜백, [인증 이벤트](#authentication-events)입니다.

또한 파이프라인에 쿠키 인증 미들웨어를 추가합니다. 이 미들웨어는 쿠키에 사용자 클레임을 기록한 후 후속 페이지 로드 중에 쿠키를 읽는 역할을 합니다.

```csharp
app.UseCookieAuthentication(new CookieAuthenticationOptions {
    AutomaticAuthenticate = true,
    AutomaticChallenge = true,
    AccessDeniedPath = "/Home/Forbidden",
    CookieSecure = CookieSecurePolicy.Always,

    // The default setting for cookie expiration is 14 days. SlidingExpiration is set to true by default
    ExpireTimeSpan = TimeSpan.FromHours(1),
    SlidingExpiration = true
});
```

## <a name="initiate-the-authentication-flow"></a>인증 흐름 시작
ASP.NET MVC에서 인증 흐름을 시작하려면 컨트롤러에서 **ChallengeResult** 를 반환합니다.

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

그러면 미들웨어가 인증 끝점으로 리디렉션하는 302(Found) 응답을 반환합니다.

## <a name="user-login-sessions"></a>사용자 로그인 세션
언급했듯이 사용자가 처음 로그인할 때, 쿠키 인증 미들웨어는 사용자 클레임을 쿠키에 씁니다. 그 후에는 쿠키를 읽어 HTTP 요청을 인증합니다.

기본적으로 쿠키 미들웨어는 [세션 쿠키][session-cookie]를 쓰며, 이 쿠키는 사용자가 브라우저를 닫으면 삭제됩니다. 따라서 다음에 사용자가 사이트를 방문하는 경우 다시 로그인해야 합니다. 그러나 **ChallengeResult**에서 **IsPersistent**를 true로 설정하면 미들웨어가 영구 쿠키를 쓰므로 사용자가 브라우저를 닫은 후에도 로그인 상태로 유지됩니다. 쿠키 만료를 구성할 수 있습니다. [쿠키 옵션 제어][cookie-options]를 참조하세요. 영구 쿠키는 사용자에게 더 편리하지만 사용자에게 매번 로그인하도록 하는 일부 응용 프로그램(예: 뱅킹 응용 프로그램)에는 적합하지 않을 수 있습니다.

## <a name="about-the-openid-connect-middleware"></a>OpenID Connect 미들웨어 정보
ASP.NET의 OpenID Connect 미들웨어는 대부분의 프로토콜 정보를 숨깁니다. 이 섹션에는 프로토콜 흐름을 이해하는 데 유용할 수 있는 구현에 대 한 일부 정보가 들어 있습니다.

먼저, ASP.NET를 기준으로 하는 인증 흐름을 살펴보겠습니다(앱과 Azure AD 간의 OIDC 프로토콜 흐름에 대한 세부 정보는 생략). 다음 다이어그램은 이 프로세스를 보여 줍니다.

![로그인 흐름](./images/sign-in-flow.png)

이 다이어그램에는 두 개의 MVC 컨트롤러가 있습니다. Account 컨트롤러는 로그인 요청을 처리하고 Home 컨트롤러는 홈 페이지를 서비스합니다.

다음은 인증 프로세스입니다.

1. 사용자가 "로그인" 단추를 클릭하면 브라우저는 GET 요청을 보냅니다. 예: `GET /Account/SignIn/`
2. Account 컨트롤러는 `ChallengeResult`를 반환합니다.
3. OIDC 미들웨어는 Azure AD로 리디렉션하는 HTTP 302 응답을 반환합니다.
4. 브라우저가 Azure AD에 인증 요청을 보냅니다.
5. 사용자가 Azure AD에 로그인하고 Azure AD는 인증 응답을 다시 보냅니다.
6. OIDC 미들웨어는 클레임 주체를 만들고 이를 쿠키 인증 미들웨어에 전달합니다.
7. 쿠키 미들웨어는 클레임 주체를 직렬화하고 쿠키를 설정합니다.
8. OIDC 미들웨어는 응용 프로그램의 콜백 URL로 리디렉션됩니다.
9. 브라우저는 요청에 쿠키를 전송하는 리디렉션을 따릅니다.
10. 쿠키 미들웨어는 클레임 주체로 쿠키를 역직렬화하고 클레임 주체와 동일하게 `HttpContext.User` 를 설정합니다. 요청은 MVC 컨트롤러로 라우팅됩니다.

### <a name="authentication-ticket"></a>인증 티켓
인증이 성공하면 OIDC 미들웨어는 사용자의 클레임을 보유하는 클레임 주체를 포함하는 인증 티켓을 만듭니다. **AuthenticationValidated** 또는 **TicketReceived** 이벤트 내에서 티켓에 액세스할 수 있습니다.

> [!NOTE]
> 전체 인증 흐름이 완료될 때까지 `HttpContext.User`는 익명 보안 주체를 계속 보유하며 인증된 사용자가 **아닙니다**. 익명 보안 주체에는 빈 클레임 컬렉션이 있습니다. 인증이 완료되고 앱이 리디렉션된 후에는 쿠키 미들웨어가 인증 쿠키를 역직렬화하고 `HttpContext.User` 를 인증된 사용자를 나타내는 클레임 주체로 설정합니다.
> 
> 

### <a name="authentication-events"></a>인증 이벤트
인증 프로세스 중에 OpenID Connect 미들웨어는 일련의 이벤트를 발생시킵니다.

* **RedirectToIdentityProvider**. 미들웨어가 인증 끝점으로 리디렉션하기 직전에 호출됩니다. 이 이벤트를 사용하여 리디렉션 URL을 수정할 수 있습니다. 예를 들어 요청 매개변수를 추가할 수 있습니다. 예를 보려면 [관리자 동의 프롬프트 추가하기](signup.md#adding-the-admin-consent-prompt)를 참조하세요.
* **AuthorizationCodeReceived**. 인증 코드와 함께 호출됩니다.
* **TokenResponseReceived**. 미들웨어가 IDP에서 액세스 토큰을 가져오고 토큰의 유효성을 검사하기 전에 호출됩니다. 인증 코드 흐름에만 적용됩니다.
* **TokenValidated**. 미들웨어가 ID 토큰의 유효성을 검사한 후 호출됩니다. 이때 응용 프로그램에는 사용자에 대해 유효성 검증된 클레임 집합이 포함됩니다. 이 이벤트를 사용하여 클레임에 대한 추가 유효성 검사를 수행하거나 클레임을 변환할 수 있습니다. [클레임 작업](claims.md)을 참조하세요.
* **UserInformationReceived**. 미들웨어가 사용자 정보 끝점에서 사용자 프로필을 가져오는 경우 호출됩니다. 인증 코드 흐름에만 적용되며 미들웨어 옵션에서 `GetClaimsFromUserInfoEndpoint = true` 인 경우에만 적용됩니다.
* **TicketReceived**. 인증이 완료되면 호출됩니다. 인증이 성공했다고 가정할 경우 마지막 이벤트입니다. 이 이벤트가 처리된 후에는 사용자가 앱에 로그인됩니다.
* **AuthenticationFailed**. 인증에 실패하면 호출됩니다. 이 이벤트를 사용하여 인증 오류를 처리합니다. 예를 들어 오류 페이지로 리디렉션합니다.

이러한 이벤트에 대한 콜백을 제공하려면 미들웨어에서 **Events** 옵션을 설정합니다. 이벤트 처리기를 선언하는 방법은 람다를 사용하여 인라인으로 선언하거나 **OpenIdConnectEvents**에서 파생되는 클래스에서 선언하는 두 가지 방법이 있습니다. 이벤트 콜백에 많은 논리가 있는 경우 두 번째 방법을 사용하는 것이 좋은데, 시작 클래스를 복잡하게 만들지 않기 때문입니다. 여기의 참조 구현에서는 이 방법을 사용합니다.

### <a name="openid-connect-endpoints"></a>OpenID Connect 끝점
Azure AD는 [OpenID Connect 검색](https://openid.net/specs/openid-connect-discovery-1_0.html)을 지원하는데, 여기에서 IDP(ID 공급자)는 [잘 알려진 끝점](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig)에서 JSON 메타데이터 문서를 반환합니다. 메타데이터 문서에는 다음과 같은 정보가 들어 있습니다.

* 권한 부여 끝점의 URL. 사용자 인증을 위해 앱이 리디렉션되는 위치입니다.
* 앱이 사용자를 로그아웃하기 위해서 가는 "세션 종료" 끝점의 URL입니다.
* 클라이언트가 IDP에서 가져온 OIDC 토큰의 유효성을 검사하기 위해 사용하는 서명 키를 가져올 URL입니다.

기본적으로 OIDC 미들웨어는 이 메타데이터를 가져오는 방법을 알고 있습니다. 미들웨어에서 **Authority** 옵션을 설정하면 미들웨어는 메타데이터에 대한 URL을 구성합니다. (**MetadataAddress** 옵션을 설정하여 메타데이터 URL을 재정의할 수 있습니다.)

### <a name="openid-connect-flows"></a>OpenID connect 흐름
기본적으로 OIDC 미들웨어는 폼 게시 응답 모드로 하이브리드 흐름을 사용합니다.

* *하이브리드 흐름*은 클라이언트가 동일한 왕복에 있는 ID 토큰 및 인증 코드를 인증 서버로 가져올 수 있음을 의미합니다.
* *폼 게시 응답 모드*는 인증 서버가 HTTP POST 요청을 사용하여 ID 토큰 및 인증 코드를 앱으로 보내는 것을 의미합니다. 값은 form-urlencoded입니다(content type = "application/x-www-form-urlencoded").

OIDC 미들웨어가 권한 부여 끝점으로 리디렉션되면 리디렉션 URL은 OIDC에 필요한 모든 쿼리 문자열 매개 변수를 포함합니다. 하이브리드 흐름의 경우 다음과 같습니다.

* client_id. 이 값은 **ClientId** 옵션에서 설정됩니다.
* scope = "openid profile", OIDC 요청이며 사용자의 프로필을 원한다는 것을 의미합니다.
* response_type  = "code id_token". 하이브리드 흐름을 지정합니다.
* response_mode = "form_post". 폼 게시 응답을 지정합니다.

서로 다른 흐름을 지정하려면 옵션에서 **ResponseType** 속성을 설정합니다. 예: 

```csharp
app.UseOpenIdConnectAuthentication(options =>
{
    options.ResponseType = "code"; // Authorization code flow

    // Other options
}
```

[**다음**][claims]

[claims]: claims.md
[cookie-options]: /aspnet/core/security/authentication/cookie#controlling-cookie-options
[session-cookie]: https://en.wikipedia.org/wiki/HTTP_cookie#Session_cookie
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
