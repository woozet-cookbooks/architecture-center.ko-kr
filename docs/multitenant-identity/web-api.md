---
title: Secure a backend web API in a multitenant application
description: How to secure a backend web API
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 06/02/2016
ms.author: pnp

pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authorize
pnp.series.next: token-cache
---
# 백 엔드 웹 API의 보안 처리

[![GitHub](../_images/github.png) 샘플 코드][sample application]

[Tailspin Surveys](https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/tailspin) 응용 프로그램은 백 엔드 웹 API를 사용하여 설문조사의 CRUD 작업을 관리합니다.  예를 들어, 사용자가 "내 설문조사"를 클릭하면, 웹 응용 프로그램은 웹 API로 HTTP 요청을 보냅니다:

```
GET /users/{userId}/surveys
```

웹 API가 JSON 개체를 반환합니다:

```
{
  "Published":[],
  "Own":[
    {"Id":1,"Title":"Survey 1"},
    {"Id":3,"Title":"Survey 3"},
    ],
  "Contribute": [{"Id":8,"Title":"My survey"}]
}
```


웹 API는 익명의 요청을 허용하지 않기 때문에, 웹 앱이 직접 OAuth 2 전달자 토큰을 사용해서 인증해야 합니다.

> [!참고]
> 이 작업은 서버 간 시나리오입니다. 응용 프로그램은 브라우저 클라이언트에서 API에 어떤 AJAZ 호출도 하지 않습니다.
> 
> 

취할 수 있는 접근 방식이 두 가지 있습니다:

•	위임된 사용자 ID. 웹 응용 프로그램은 사용자 ID를 인증합니다.

•	응용 프로그램 ID. 웹 응용 프로그램은 OAuth2 클라이언트 자격 증명 흐름을 사용하여 클라이언트 ID를 인증합니다.

Tailspin 응용 프로그램은 위임된 사용자 ID를 구현합니다. 주요 차이점은 다음과 같습니다:

**위임된 사용자 ID**

•	웹 API로 전송된 전달자 토큰은 사용자 ID를 포함합니다.

•	웹 API는 사용자 ID에 근거하여 권한 부여를 결정합니다.

•	사용자가 동작 수행의 권한을 부여받지 않은 경우, 웹 응용 프로그램은 웹 API에서 발생한 403(사용 권한 없음) 오류를 처리해야 합니다. 

•	일반적으로, 웹 응용 프로그램은 여전히 UI 표시 또는 숨기기와 같이 UI에 영향을 주는 권한 부여 결정을 내립니다. 

•	JavaScript 응용 프로그램 또는 네이티브 클라이언트 응용 프로그램과 같이, 웹 API는 잠재적으로 신뢰받지 않는 클라이언트에 의해 사용될 수 있습니다.


**응용 프로그램 ID**

•	웹 API는 사용자에 관한 정보를 가져오지 않습니다.

•	웹 API는 사용자 ID에 근거하여 어떤 권한 부여도 수행할 수 없습니다. 모든 권한 부여는 웹 응용 프로그램에 의해 결정됩니다. 

•	웹 API는 신뢰받지 않는 클라이언트에 의해 사용될 수 없습니다(JavaScript 또는 네이트브 클라이언트 응용 프로그램).

•	웹 API에는 권한 부여 로직이 없기 때문에, 이 접근 방식을 구현하기가 더 간단할 수 있습니다. 

어느 접근 방식이어도 웹 응용 프로그램은 웹 API 호출에 필요한 자격 증명인 액세스 토큰을 가져와야 합니다.

•	위임된 사용자 ID의 경우, 토큰은 사용자 대신 토큰을 발급할 수 있는 IDP에서 와야 합니다.

•	클라이언트 자격 증명의 경우, 응용 프로그램은 IDP에서 토큰을 가져오거나 자체의 토큰 서버를 호스팅할 수 있습니다. (그렇다고 사전 지식 없이 토큰 서버를 쓰지 마시고, 시험을 무난히 통과한  [IdentityServer3](https://github.com/IdentityServer/IdentityServer3)과 같은 프레임워크를 사용하세요.) Azure AD를 인증할 경우, 클라이언트 자격 증명 흐름으로 Azure AD에서 액세스 토큰을 가져올 것을 강력 추천합니다.

이 문서의 나머지 부분은 응용 프로그램이 Azure AD를 인증하는 것으로 가정합니다.

![Getting the access token](./images/access-token.png)

## Azure AD에서 웹 API 등록
Azure AD가 웹 API에 대한 전달자 토큰을 발급하려면, Azure AD에서 몇 가지 구성해야 합니다.

1. [Azure AD에서 웹 API 등록].
2. 웹 앱의 클라이언트 ID를 웹 API 응용 프로그램 매니페스트에 `knownClientApplications` 속성으로 추가합니다. [U응용 프로그램 매니페스트 업데이트]를 참조하세요.
3. [웹 응용 프로그램에 웹 API 호출 권한 부여](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/docs/running-the-app.md#give-the-web-app-permissions-to-call-the-web-api).
   
   Azure 관리 포털에서 두 가지 유형의 권한을 설정할 수 있습니다. 응용 프로그램 ID에 대한 "응용 프로그램 권한" (클라이언트 자격 증명 흐름) 또는 위임된 사용자 ID에 대한 "위임된 권한".
   
   ![Delegated permissions](./images/delegated-permissions.png)

## 액세스 토큰 가져오기
웹 응용 프로그램은 웹 API를 호출하기 전에 Azure AD에서 액세스 토큰을 가져옵니다. .NET 응용 프로그램에서는, [.NET용 Azure AD 인증 라이브러리(ADAL)][ADAL]를 사용합니다.

OAuth 2 인증 코드에서, 응용 프로그램은 인증 코드와 액세스 토큰을 교환합니다. 다음 코드는 ADAL을 사용하여 액세스 토큰을 가져옵니다. 이 코드는 `AuthorizationCodeReceived` 이벤트 중에 호출됩니다.

```csharp
// The OpenID Connect middleware sends this event when it gets the authorization code.   
public override async Task AuthorizationCodeReceived(AuthorizationCodeReceivedContext context)
{
    string authorizationCode = context.ProtocolMessage.Code;
    string authority = "https://login.microsoftonline.com/" + tenantID
    string resourceID = "https://tailspin.onmicrosoft.com/surveys.webapi" // App ID URI
    ClientCredential credential = new ClientCredential(clientId, clientSecret);

    AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
    AuthenticationResult authResult = await authContext.AcquireTokenByAuthorizationCodeAsync(
        authorizationCode, new Uri(redirectUri), credential, resourceID);

    // If successful, the token is in authResult.AccessToken
}
```

다음은 코드에 필요한 여러 가지 매개변수입니다:

* `authority`. 로그인된 사용자의 테넌트 ID에서 파생됨. (SaaS 공급자의 테넌트 ID가 아님) 

* `authorizationCode`. IDP에서 되돌려받은 인증 코드 .

* `clientId`. 웹 응용 프로그램의 클라이언트 ID.

* `clientSecret`. 웹 응용 프로그램의 클라이언트 암호.

* `redirectUri`. OpenID 연결에 설정한 리디렉션 URI. IDP가 토큰을 가지고 호출하는 곳입니다.

* `resourceID`. Azure AD에서 웹 API를 등록할 때 만든 웹 API의 앱 ID URI.
 
* `tokenCache`. 액세스 토큰을 캐시하는 개체. [토큰 캐싱](https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/token-cache)을 참조하세요.

`AcquireTokenByAuthorizationCodeAsync` 가 성공한 경우, ADAL은 토큰을 캐시합니다. 추후, AcquireTokenSilentAsync를 호출하여 캐시에서 토큰을 가져올 수 있습니다.

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

`userId`가 사용자의 개체 ID인 경우, `http://schemas.microsoft.com/identity/claims/objectidentifier` 클레임에 존재합니다.

## 액세스 토큰을 이용한 웹 API 호출
토큰이 있으면, HTTP 요청의 인증 헤더에 담아 웹 API로 전송합니다. 

```
Authorization: Bearer xxxxxxxxxx
```

다음과 같은 Surveys 응용 프로그램의 확장 메서드는 **HttpClient** 클래스를 사용하여 HTTP 요청에 있는 인증 헤더를 설정합니다.

```csharp
public static async Task<HttpResponseMessage> SendRequestWithBearerTokenAsync(this HttpClient httpClient, HttpMethod method, string path, object requestBody, string accessToken, CancellationToken ct)
{
    var request = new HttpRequestMessage(method, path);
    if (requestBody != null)
    {
        var json = JsonConvert.SerializeObject(requestBody, Formatting.None);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        request.Content = content;
    }

    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
    request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

    var response = await httpClient.SendAsync(request, ct);
    return response;
}
```

> [!참고]
> [HttpClientExtensions.cs](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Common/HttpClientExtensions.cs)를 참조하세요.
> 
> 

## 웹 API에서 인증하기
웹 API는 전달자 토큰을 인증해야 합니다. ASP.NET Core 1.0에서, [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] 패키지를 사용할 수 있습니다. 이 패키지는 OpenIN Connect 전달자 토큰을 받을 때 응용 프로그램을 사용하는 미들웨어를 제공합니다.

미들웨어를 웹 API `Startup` 클래스에 등록합니다.

```csharp
app.UseJwtBearerAuthentication(options =>
{
    options.Audience = "[app ID URI]";
    options.Authority = "https://login.microsoftonline.com/common/";
    options.TokenValidationParameters = new TokenValidationParameters
    {
        //Instead of validating against a fixed set of known issuers, we perform custom multi-tenant validation logic
        ValidateIssuer = false,
    };
    options.Events = new SurveysJwtBearerEvents();
});
```

> [!참고]
> [Startup.cs](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Common/HttpClientExtensions.cs)를 참조하세요.
> 
> 

* **Audience**. Azure AD에서 웹 API를 등록할 때 만든 웹 API의 앱 ID URI에 이 옵션을 등록합니다.

* **Authority**. 다중 테넌트 응용 프로그램의 경우, 이 옵션을 `https://login.microsoftonline.com/common/`으로 설정합니다.

* **TokenValidationParameters**. 다중 테넌트 응용 프로그램의 경우, **ValidateIssuer**를 false로 설정합니다. 응용 프로그램이 발급자를 확인할 것이라는 뜻입니다.
* **Events**는 **JwtBearerEvents**에서 파생된 클래스입니다.

### 발급자 확인
**JwtBearerEvents.ValidatedToken** 이벤트에서 토큰 발급자를 확인합니다.. 발급자가 "iss" 클레임에 전송됩니다.

Surveys 응용 프로그램에서, 웹 API는 [tenant sign-up](https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/signup)을 처리하지 않습니다.  따라서, 발급자가 응용 프로그램 데이터베이스에 이미 있는지 확인만 합니다. 없으면, 예외가 발생해서 인증이 실패한 것입니다.

```csharp
public override async Task ValidatedToken(ValidatedTokenContext context)
{
    var principal = context.AuthenticationTicket.Principal;
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue);

    if (tenant == null)
    {
        // the caller was not from a trusted issuer - throw to block the authentication flow
        throw new SecurityTokenValidationException();
    }
}
```

> [!참고]
> [SurveysJwtBearerEvents.cs](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.WebAPI/SurveyJwtBearerEvents.cs)를 참조하세요.
> 
> 

[클레임 변환](https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/claims#claims-transformations)을 할 때도 **ValidatedToken** 이벤트를 사용할 수 있습니다. 클레임은 Azure AD에서 직접 오는 것이고, 그래서 웹 응용 프로그램이 클레임 변환을 한 경우, 웹 API가 받는 전달자 토큰에 반영되지 않는다는 점을 기억하세요.

## 권한 부여
권한 부여에 대한 일반적인 논의는, [역할 기반 및 리소스 기반 권한 부여][Authorization]를 참조하세요. 

JwBearer 미들웨어는 권한 부여 응답을 처리합니다. 예를 들면, 컨트롤러 동작을 인증된 사용자로 제한하기 위해서, **[Authorize]** 특성을 사용하고 **JwtBearerDefaults.AuthenticationScheme**을 인증 체계로 지정합니다:

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

이 코드는 사용자가 인증되지 않은 경우, 401 상태 코드를 반환합니다.

인증 정책에 따라 컨트롤러 동작을 제한하려면, **[Authorize]** 특성에 정책 이름을 지정합니다:

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

사용자가 인증되지 않은 경우 401 상태 코드를, 사용자가 인증되었지만 권한이 없는 경우 403 상태 코드를 반환합니다. startup에 정책을 등록합니다:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthorization(options =>
    {
        options.AddPolicy(PolicyNames.RequireSurveyCreator,
            policy =>
            {
                policy.AddRequirements(new SurveyCreatorRequirement());
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
    });
}
```

[**다음**][token cache]

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Tailspin Surveys]: tailspin.md
[IdentityServer3]: https://github.com/IdentityServer/IdentityServer3
[Register the web API in Azure AD]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/docs/running-the-app.md#register-the-surveys-web-api
[Update the application manifests]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/docs/running-the-app.md#update-the-application-manifests
[Give the web application permission to call the web API]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/docs/running-the-app.md#give-the-web-app-permissions-to-call-the-web-api
[Token caching]: token-cache.md
[HttpClientExtensions.cs]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Common/HttpClientExtensions.cs
[Startup.cs]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.WebAPI/Startup.cs
[tenant sign-up]: signup.md
[SurveysJwtBearerEvents.cs]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.WebAPI/SurveyJwtBearerEvents.cs
[claims transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps
[token cache]: token-cache.md
