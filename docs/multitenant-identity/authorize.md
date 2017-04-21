---
title: Authorization in multitenant applications
description: How to perform authorization in a multitenant application
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 06/02/2016
ms.author: pnp

pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
---
# R역할 기반 및 리소스 기반 인증

[![GitHub](../_images/github.png) 샘플 코드][sample application]

우리의 [참조 구현](https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/tailspin)은 ASP.NET Core 1.0 응용 프로그램입니다. 이 문서에서는 ASP.NET Core 1.0에서 제공한 인증 API를 사용하여 인증에 대한 두 가지 일반적인 접근 방식을 살펴보겠습니다.

* **역할 기반 인증**.사용자에 지정된 역할을 기반으로 한 동작 인증. 예를 들면, 어떤 동작들은 관리자 역할을 요청합니다.

* **리소스 기반 인증**. 특정 리소스를 기반으로 한 동작 인증. 예를 들면, 모든 리소스에는 소유자가 있습니다. 소유자는 리소스를 삭제할 수 있지만, 다른 사용자들은 그럴 수 없습니다.

일반적인 앱은 둘을 혼합해서 이용합니다. 예를 들면, 어떤 리소스를 삭제하려면 사용자는 리소스 소유자이거나 관리자여야 합니다.

## 역할 기반 인증
[Tailspin Surveys][Tailspin] 응용 프로그램은 다음 역할을 정의합니다:

•	관리자. 해당 테넌트에 속한 설문조사에서 모든 CRUD 연산을 수행할 수 있습니다.

•	작성자. 새 설문조사를 만들 수 있습니다.

•	독자. 해당 테넌트에 속한 설문조사를 읽을 수 있습니다.


역할은 응용 프로그램의 *사용자*에게 적용됩니다. Surveys 응용 프로그램에서 사용자는 관리자, 작성자, 독자 중 하나입니다.

역할을 어떻게 정의하고 관리할지에 대한 논의는, [응용 프로그램 역할](https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/app-roles)을 참조하세요.

역할을 어떻게 관리하느냐와 관계없이, 인증 코드는 비슷해 보입니다. ASP.NET Core 1.0은 [인증 정책][policies]이라는 개념을 소개합니다. 이러한 특징과 더불어, 코드에서 인증 정책을 정의하고 그 정책을 컨트롤러 동작에 적용합니다. 정책은 컨트롤러에서 분리됩니다.

### 정책 만들기
어떤 정책을 정의하려면, 우선 `IAuthorizationRequirement`를 구현하는 클래스를 만듭니다. `AuthorizationHandler`에서 파생되는 것이 가장 쉽습니다. `Handle` 방법에서는 관련된 클레임을 검토합니다.

다음은 Tailspin Surveys 응용 프로그램에서 발췌한 예입니다:

```csharp
public class SurveyCreatorRequirement : AuthorizationHandler<SurveyCreatorRequirement>, IAuthorizationRequirement
{
    protected override void Handle(AuthorizationContext context, SurveyCreatorRequirement requirement)
    {
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin) ||
            context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            context.Succeed(requirement);
        }
    }
}
```

> [!참고]
> [SurveyCreatorRequirement.cs](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Security/Policy/SurveyCreatorRequirement.cs)를 참조하세요.
>
>

다음 클래스는 새로운 설문조사를 작성하기 위해서 사용자에 대한 요구 사항을 정의합니다. 사용자는 SurveyAdmin 또는 SurveyCreator 역할이어야 합니다.

시작 클래스에서, 요구 사항을 하나 이상 포함하는 명명된 정책을 정의합니다. 요구 사항이 여러 개인 경우, 사용자는 *모든* 요구 사항이 인증받도록 해야 합니다. 다음 코드는 두 가지 정책을 정의합니다:

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy(PolicyNames.RequireSurveyCreator,
        policy =>
        {
            policy.AddRequirements(new SurveyCreatorRequirement());
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });

    options.AddPolicy(PolicyNames.RequireSurveyAdmin,
        policy =>
        {
            policy.AddRequirements(new SurveyAdminRequirement());
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });
});
```

> [!참고]
> [Startup.cs](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Web/Startup.cs)를 참조하세요.
>
>

이 코드는 인증 체계도 설정하는데, 인증이 실패할 경우 어떤 인증 미들웨어가 실행되어야 하는지 ASP.NET에 알려줍니다. 이 경우, 쿠키 인증 미들웨어가 사용자를 "사용할 수 없음" 페이지로 리디렉션 할 수 있기 때문에 우리는 쿠키 인증 미들웨어를 지정합니다. 사용할 수 없음 페이지의 위치는 쿠키 미들웨어에 대한 AccessDeniedPath 옵션에서 설정됩니다; [인증 미들웨어 구성하기](https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/authenticate#configure-the-auth-middleware)를 참조하세요.

### 컨트롤러 동작 인증
마지막으로, MVC 컨트롤러에서 어떤 동작을 인증하려면 `Authorize` 특성에서 정책을 설정합니다.

```csharp
[Authorize(Policy = "SurveyCreatorRequirement")]
public IActionResult Create()
{
    // ...
}
```

ASP.NET의 초기 버전이라면, 특성에서 **Roles** 속성을 적절히 설정해야 합니다:

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]

```

이 코드는 여전히 ASP.NET Core 1.0에서 지원되지만, 인증 정책과 비교하면 몇 가지 결점이 있습니다:

•	특정한 클레임 유형을 가정합니다. 정책은 어떤 클레임 유형이라도 검사할 수 있습니다. 역할은 클레임의 한 유형일 뿐입니다.

•	역할 이름은 속성에 하드 코드로 작성됩니다. 정책이 있는 경우, 인증 로직이 모두 한 곳에 있어서 구성 설정을 업데이트하거나 그로부터 로드하기 더 쉽습니다.

•	정책은 단순한 역할 멤버십으로는 표현할 수 없는 더 복잡한 인증 결정을 가능하게 합니다(예: age >= 21).


## 리소스 기반 인증
*리소스 기반 인증*은 인증이 어떤 작업에 의해 영향을 받는 특정 리소스에 의존할 때마다 발생합니다.  Tailspin Surveys 응용 프로그램에서, 모든 설문조사는 한 명의 소유자와 영대다(zero-to-many)의 참가자를 보유합니다.

•	소유자는 설문조사를 읽기, 업데이트, 삭제, 게시, 게시 취소할 수 있습니다.

•	소유자는 설문조사에 참가자를 지정할 수 있습니다.

•	참가자들은 설문조사를 읽고 업데이트 할 수 있습니다.

"소유자"와 "참가자"는 응용 프로그램 역할이 아닙니다; 이들은 설문조사별로 응용 프로그램 데이터베이스에 저장됩니다. 예를 들면, 사용자가 설문조사를 삭제할 수 있는지 확인하기 위해서 앱은 사용자가 그 설문조사의 소유자인지 확인합니다.

ASP.NET Core 1.0에서는, **AuthorizationHandler**에서 파생하고 **Handle** 메서드를 재정의하는 방법으로 리소스 기반 인증을 구현합니다.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
     protected override void Handle(AuthorizationContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

이 클래스는 Survey 개체를 위한 강력한 형식(strongly typed)입니다. 시작 클래스에 DI에 대한 클래스를 등록합니다:

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

인증 확인을 위해서 **IAuthorizationService** 인터페이스를 사용합니다. 컨트롤러에 이 인터페이스를 삽입할 수 있습니다.  다음 코드는 사용자가 설문조사를 읽을 수 있는지 여부를 확인합니다:

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return new HttpStatusCodeResult(403);
}
```

우리가 `Survey` 개체를 전달했기 때문에, 이 호출은 `SurveyAuthorizationHandler`를 불러올 것입니다.

인증 코드에서 좋은 접근 방식은 사용자의 역할 기반 및 리소스 기반의 권한을 모두 집계한 다음, 원하는 작업에 대하여 그 집계를 확인하는 것입니다. 다음은 Surveys 응용 프로그램에서 발췌한 예입니다. 응용 프로그램은 몇 가지 권한 유형을 정의합니다:

•	관리자

•	참가자

•	작성자

•	소유자

•	독자


응용 프로그램은 설문조사에서 있을 수 있는 작업을 정의합니다:

•	작성

•	읽기

•	업데이트

•	삭제

•	게시

•	게시 취소


다음 코드는 특정 사용자와 설문조사에 대한 권한 목록을 만듭니다. 이 코드는 사용자의 앱 역할과 설문조사의 소유자/참가자 필드를 모두 볼 수 있습니다.

```csharp
protected override void Handle(AuthorizationContext context, OperationAuthorizationRequirement operation, Survey resource)
{
    var permissions = new List<UserPermissionType>();
    string userTenantId = context.User.GetTenantIdValue();
    int userId = ClaimsPrincipalExtensions.GetUserKey(context.User);
    string user = context.User.GetUserName();

    if (resource.TenantId == userTenantId)
    {
        // Admin can do anything, as long as the resource belongs to the admin's tenant.
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin))
        {
            context.Succeed(operation);
            return;
        }

        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            permissions.Add(UserPermissionType.Creator);
        }
        else
        {
            permissions.Add(UserPermissionType.Reader);
        }

        if (resource.OwnerId == userId)
        {
            permissions.Add(UserPermissionType.Owner);
        }
    }
    if (resource.Contributors != null && resource.Contributors.Any(x => x.UserId == userId))
    {
        permissions.Add(UserPermissionType.Contributor);
    }
    if (ValidateUserPermissions[operation](permissions))
    {
        context.Succeed(operation);
    }
}
```

> [!참고]
> [SurveyAuthorizationHandler.cs](https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Security/Policy/SurveyAuthorizationHandler.cs)를 참조하세요.
>
>

다중 테넌트 응용 프로그램에서, 사용 권한이 다른 테넌트의 데이터로 "누출"되지 않게 해야 합니다. Survey 응용 프로그램에서, 참가자 권한은 테넌트 간에 허용됩니다 - 사용자는 다른 테넌트에 있는 사람을 참가자로 지정할 수 있습니다. 다른 권한 유형은 그 사용자의 테넌트에 속한 리소스로 제한됩니다. 이 요구 사항을 적용하기 위해서 코드는 권한을 부여하기 전에 테넌트 ID를 확인합니다. (설문조사가 생성될 때 지정된 `TenantId` 필드)

다음 단계는 권한에 대한 작업을(읽기, 업데이트, 삭제 등) 확인하는 것입니다. Survey 응용 프로그램은 함수 조회 테이블을 사용하여 이 단계를 구현합니다:

```csharp
static readonly Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>> ValidateUserPermissions
    = new Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>>

    {
        { Operations.Create, x => x.Contains(UserPermissionType.Creator) },

        { Operations.Read, x => x.Contains(UserPermissionType.Creator) ||
                                x.Contains(UserPermissionType.Reader) ||
                                x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Update, x => x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Delete, x => x.Contains(UserPermissionType.Owner) },

        { Operations.Publish, x => x.Contains(UserPermissionType.Owner) },

        { Operations.UnPublish, x => x.Contains(UserPermissionType.Owner) }
    };
```


[**다음**][web-api]

<!-- Links -->
[Tailspin]: tailspin.md

[Application roles]: app-roles.md
[policies]: https://docs.asp.net/en/latest/security/authorization/policies.html
[rbac]: https://docs.asp.net/en/latest/security/authorization/resourcebased.html
[reference implementation]: tailspin.md
[SurveyCreatorRequirement.cs]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Security/Policy/SurveyCreatorRequirement.cs
[Startup.cs]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Web/Startup.cs
[Configuring the authentication middleware]: authenticate.md#configure-the-auth-middleware
[SurveyAuthorizationHandler.cs]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Security/Policy/SurveyAuthorizationHandler.cs
[sample application]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps
[web-api]: web-api.md
