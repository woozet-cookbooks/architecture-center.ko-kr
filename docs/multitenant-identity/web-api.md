---
title: "다중 테넌트 응용 프로그램에서 백 엔드 웹 API 보안 유지"
description: "백 엔드 웹 API의 보안을 유지하는 방법"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authorize
pnp.series.next: token-cache
ms.openlocfilehash: 65529280c5849e36ed7ff23de08a0b485034d0d8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="secure-a-backend-web-api"></a>백 엔드 웹 API의 보안 유지

[![GitHub](../_images/github.png) 샘플 코드][sample application]

[Tailspin 설문 조사] 응용 프로그램은 백 엔드 웹 API를 사용하여 설문 조사에 대한 CRUD 작업을 관리합니다. 예를 들어 사용자가 "내 설문 조사"를 클릭하면 웹 응용 프로그램은 웹 API에 HTTP 요청을 보냅니다.

```
GET /users/{userId}/surveys
```

웹 API는 JSON 개체를 반환합니다.

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

웹앱은 OAuth 2 전달자 토큰을 사용하여 자신을 인증해야 하므로 웹 API는 익명 요청을 허용하지 않습니다.

> [!NOTE]
> 이는 서버 간 시나리오입니다. 응용 프로그램은 브라우저 클라이언트에서 API에 대한 AJAX 호출을 만들지 않습니다.
> 
> 

실행할 수 있는 두 가지 주요 방법이 있습니다.

* 위임된 사용자 ID. 웹 응용 프로그램은 사용자의 ID로 인증합니다.
* 응용 프로그램 ID. 웹 응용 프로그램은 OAuth2 클라이언트 자격 증명 흐름을 사용하여 해당 클라이언트 ID로 인증합니다.

Tailspin 응용 프로그램은 위임된 사용자 ID를 구현합니다. 주요 차이점은 다음과 같습니다.

**위임된 사용자 ID**

* 웹 API로 전송되는 전달자 토큰은 사용자 ID를 포함합니다.
* 웹 API는 사용자 ID에 따라 권한 부여를 결정합니다.
* 웹 응용 프로그램은 사용자가 작업을 수행할 수 있는 권한이 없는 경우 웹 API에서 403(사용할 수 없음) 오류를 처리해야 합니다.
* 일반적으로 웹 응용 프로그램은 UI 요소 표시 또는 숨기기와 같은 UI에 영향을 주는 몇 가지 권한 부여를 결정합니다.)
* 웹 API는 JavaScript 응용 프로그램 또는 네이티브 클라이언트 응용 프로그램과 같은 신뢰할 수 없는 클라이언트에서 잠재적으로 사용될 수 있습니다.

**응용 프로그램 ID**

* 웹 API는 사용자에 대한 정보를 가져오지 않습니다.
* 웹 API는 사용자 ID에 따라 권한 부여를 수행할 수 없습니다. 웹 응용 프로그램에서 모든 권한 부여를 결정합니다.  
* 웹 API는 신뢰할 수 없는 클라이언트(JavaScript 또는 네이티브 클라이언트 응용 프로그램)에서 사용될 수 없습니다.
* 웹 API에는 권한 부여 논리가 없으므로 이 방법은 구현하는 데 다소 수월할 수 있습니다.

방법 중 하나에서 웹 응용 프로그램은 웹 API를 호출하는 데 필요한 자격 증명인 액세스 토큰을 가져와야 합니다.

* 위임된 사용자 ID의 경우 토큰은 사용자를 대신하여 토큰을 발급할 수 있는 IDP에서 가져와야 합니다.
* 클라이언트 자격 증명의 경우 응용 프로그램은 해당 자체 토큰 서버인 IDP 또는 호스트에서 토큰을 가져올 수도 있습니다. 하지만 처음부터 토큰 서버를 작성하지 말고 [IdentityServer3]과 같이 테스트가 완료된 프레임워크를 사용합니다. Azure AD로 인증하는 경우 Azure AD의 액세스 토큰을 클라이언트 자격 증명 흐름과 함께 사용하는 것이 좋습니다.

이 문서의 나머지 부분에서는 응용 프로그램이 Azure AD로 인증하는 것을 가정합니다.

![액세스 토큰 가져오기](./images/access-token.png)

## <a name="register-the-web-api-in-azure-ad"></a>Azure AD에서 웹 API 등록
Azure AD에서 웹 API에 대한 전달자 토큰을 발급하려면 Azure AD에서 몇 가지를 구성해야 합니다.

1. Azure AD에서 웹 API를 등록합니다.

2. 웹앱의 클라이언트 ID를 `knownClientApplications` 속성의 웹 API 응용 프로그램 매니페스트에 추가합니다. [응용 프로그램 매니페스트 업데이트]를 참조하세요.

3. 웹 응용 프로그램에 웹 API를 호출하는 권한을 부여합니다. Azure 관리 포털에서 응용 프로그램 ID(클라이언트 자격 증명 흐름)에 대한 "응용 프로그램 권한" 또는 위임된 사용자 ID에 대한 "위임된 권한", 두 가지 유형의 사용 권한을 설정할 수 있습니다.
   
   ![위임된 권한](./images/delegated-permissions.png)

## <a name="getting-an-access-token"></a>액세스 토큰 가져오기
웹 API를 호출하기 전에 웹 응용 프로그램은 Azure AD에서 액세스 토큰을 가져옵니다. .NET 응용 프로그램에서는 [.NET용 ADAL(Azure AD 인증 라이브러리)][ADAL]을 사용합니다.

OAuth 2 권한 부여 코드 흐름에서 응용 프로그램은 액세스 토큰에 대한 권한 부여 코드를 교환합니다. 다음 코드는 ADAL을 사용하여 액세스 토큰을 가져옵니다. 이 코드는 `AuthorizationCodeReceived` 이벤트 중에 호출됩니다.

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

필요한 다양한 매개 변수는 다음과 같습니다.

* `authority`. 로그인한 사용자의 테넌트 ID에서 파생됨. (SaaS 공급자의 테넌트 ID가 아님)  
* `authorizationCode`. IDP에서 다시 가져온 인증 코드.
* `clientId`. 웹 응용 프로그램의 클라이언트 ID.
* `clientSecret`. 웹 응용 프로그램의 클라이언트 암호.
* `redirectUri`. OpenID Connect에 대해 설정한 리디렉션 URI. IDP가 토큰으로 다시 호출하는 위치입니다.
* `resourceID`. Azure AD에서 웹 API를 등록할 때 만든 웹 API의 앱 ID URI
* `tokenCache`. 액세스 토큰을 캐시하는 개체. [토큰 캐싱]을 참조하세요.

`AcquireTokenByAuthorizationCodeAsync` 이(가) 성공하는 경우 ADAL은 토큰을 캐시합니다. 나중에 AcquireTokenSilentAsync를 호출하여 캐시에서 토큰을 가져올 수 있습니다.

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

여기서 `userId`는 `http://schemas.microsoft.com/identity/claims/objectidentifier` 클레임에 있는 사용자의 개체 ID입니다.

## <a name="using-the-access-token-to-call-the-web-api"></a>액세스 토큰을 사용하여 웹 API 호출
토큰을 가지면 웹 API에 대한 HTTP 요청의 인증 헤더에 전송합니다.

```
Authorization: Bearer xxxxxxxxxx
```

설문 조사 응용 프로그램의 다음 확장 메서드는 **HttpClient** 클래스를 사용하여 HTTP 요청에 권한 부여 헤더를 설정합니다.

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

## <a name="authenticating-in-the-web-api"></a>웹 API에서 인증
웹 API는 전달자 토큰을 인증해야 합니다. ASP.NET Core에서는 [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] 패키지를 사용할 수 있습니다. 이 패키지는 응용 프로그램에서 OpenID Connect 전달자 토큰을 받을 수 있게 해 주는 미들웨어를 제공합니다.

웹 API `Startup` 클래스에서 미들웨어를 등록합니다.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ApplicationDbContext dbContext, ILoggerFactory loggerFactory)
{
    // ...

    app.UseJwtBearerAuthentication(new JwtBearerOptions {
        Audience = configOptions.AzureAd.WebApiResourceId,
        Authority = Constants.AuthEndpointPrefix,
        TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuer = false
        },
        Events= new SurveysJwtBearerEvents(loggerFactory.CreateLogger<SurveysJwtBearerEvents>())
    });
    
    // ...
}
```

* **Audience**. Azure AD로 웹 API를 등록할 때 만든 웹 API에 대한 앱 ID URI로 설정합니다.
* **Authority**. 다중 테넌트 응용 프로그램의 경우 `https://login.microsoftonline.com/common/`으로 설정합니다.
* **TokenValidationParameters**. 다중 테넌트 응용 프로그램의 경우 **ValidateIssuer**를 false로 설정합니다. 즉, 응용 프로그램이 발급자의 유효성을 검사합니다.
* **Events**는 **JwtBearerEvents**에서 파생된 클래스입니다.

### <a name="issuer-validation"></a>발급자 유효성 검사
**JwtBearerEvents.TokenValidated** 이벤트에서 토큰 발급자의 유효성을 검사합니다. 발급자는 "iss" 클레임에서 전송됩니다.

설문 조사 응용 프로그램에서 웹 API는 [테넌트 등록]을 처리하지 않습니다. 따라서 발급자가 이미 응용 프로그램 데이터베이스에 있는지만을 확인합니다. 그렇지 않으면 인증이 실패되는 예외를 throw합니다.

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.Ticket.Principal;
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue);

    if (tenant == null)
    {
        // The caller was not from a trusted issuer. Throw to block the authentication flow.
        throw new SecurityTokenValidationException();
    }

    var identity = principal.Identities.First();

    // Add new claim for survey_userid
    var registeredUser = await userManager.FindByObjectIdentifier(principal.GetObjectIdentifierValue());
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyUserIdClaimType, registeredUser.Id.ToString()));
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyTenantIdClaimType, registeredUser.TenantId.ToString()));

    // Add new claim for Email
    var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
    if (!string.IsNullOrWhiteSpace(email))
    {
        identity.AddClaim(new Claim(ClaimTypes.Email, email));
    }
}
```

이 예제에서처럼 **TokenValidated** 이벤트를 사용하여 클레임을 수정할 수도 있습니다. 클레임은 Azure AD에서 직접 가져온 것입니다. 웹 응용 프로그램에서 가져오는 클레임을 수정할 경우 해당 변경 내용이 웹 API를 수신하는 전달자 토큰에 나타나지 않습니다. 자세한 내용은 [클레임 변환][claims-transformation]을 참조하세요.

## <a name="authorization"></a>권한 부여
권한 부여에 대한 일반적 내용은 [역할 기반 및 리소스 기반 권한 부여][Authorization]를 참조하세요. 

JwtBearer 미들웨어는 인증 응답을 처리합니다. 예를 들어 컨트롤러 작업을 인증된 사용자로 제한하려면 **[Authorize]** 특성을 사용하고 **JwtBearerDefaults.AuthenticationScheme**을 인증 체계로 지정합니다.

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

사용자가 인증되지 않은 경우 401 상태 코드를 반환합니다.

권한 정책에 의한 컨트롤러 작업을 제한하려면 **[Authorize]** 특성에서 정책 이름을 지정합니다.

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

사용자가 인증되지 않은 경우 401 상태 코드를 반환하고 사용자가 인증됐지만 권한이 부여되지 않은 경우 403을 반환합니다. 시작 시 정책을 등록합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthorization(options =>
    {
        options.AddPolicy(PolicyNames.RequireSurveyCreator,
            policy =>
            {
                policy.AddRequirements(new SurveyCreatorRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
        options.AddPolicy(PolicyNames.RequireSurveyAdmin,
            policy =>
            {
                policy.AddRequirements(new SurveyAdminRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
    });
    
    // ...
}
```

[**다음**][token cache]

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Tailspin 설문 조사]: tailspin.md
[IdentityServer3]: https://github.com/IdentityServer/IdentityServer3
[응용 프로그램 매니페스트 업데이트]: ./run-the-app.md#update-the-application-manifests
[토큰 캐싱]: token-cache.md
[테넌트 등록]: signup.md
[claims-transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[token cache]: token-cache.md
