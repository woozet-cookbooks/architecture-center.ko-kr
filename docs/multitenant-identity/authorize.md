---
title: 다중 테넌트 응용 프로그램의 권한 부여
description: 다중 테넌트 응용 프로그램에서 권한 부여를 수행하는 방법
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 03c4d5fa10c75437a7b066534619ba9a123c350c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="role-based-and-resource-based-authorization"></a>역할 기반 및 리소스 기반 권한 부여

[![GitHub](../_images/github.png) 샘플 코드][sample application]

[참조 구현]은 ASP.NET Core 응용 프로그램입니다. 이 문서에서는 ASP.NET Core에 제공된 권한 부여 API를 사용하는 두 가지 일반적인 권한 부여 방법을 살펴보겠습니다.

* **역할 기반 권한 부여**. 사용자에게 할당된 역할에 따라 작업에 대한 권한을 부여합니다. 예를 들어 일부 작업에는 관리자 역할이 필요합니다.
* **리소스 기반 권한 부여**. 특정 리소스에 따라 작업에 대한 권한을 부여합니다. 예를 들어 모든 리소스에는 소유자가 있습니다. 소유자는 리소스를 삭제할 수 있지만 다른 사용자는 삭제할 수 없습니다.

일반적인 앱에서는 두 가지가 혼용됩니다. 예를 들어 리소스를 삭제하려면 사용자가 리소스 소유자 *또는* 관리자여야 합니다.

## <a name="role-based-authorization"></a>역할 기반 권한 부여
[Tailspin Surveys][Tailspin] 응용 프로그램은 다음 역할을 정의합니다.

* 관리자. 해당 테넌트에 속하는 모든 설문 조사에 대한 모든 CRUD 작업을 수행할 수 있습니다.
* 작성자. 새 설문 조사를 만들 수 있습니다.
* 읽기 권한자. 해당 테넌트에 속하는 모든 설문 조사를 읽을 수 있습니다.

역할은 응용 프로그램의 *사용자*에게 적용됩니다. Surveys 응용 프로그램에서 사용자는 관리자, 작성자 또는 읽기 권한자입니다.

역할을 정의하고 관리하는 방법에 대한 자세한 내용은 [응용 프로그램 역할]을 참조하세요.

역할을 관리하는 방법에 상관없이 인증 코드는 유사합니다. ASP.NET Core에는 [권한 부여 정책][policies]이라는 추상화가 있습니다. 이 기능을 통해 코드에서 권한 부여 정책을 정의한 후 컨트롤러 작업에 적용할 수 있습니다. 이 정책은 컨트롤러에서 분리됩니다.

### <a name="create-policies"></a>정책 만들기
정책을 정의하려면 먼저 `IAuthorizationRequirement`를 구현하는 클래스를 만듭니다. `AuthorizationHandler`에서 파생하는 것이 가장 간단합니다. `Handle` 메서드에서 관련 클레임을 검사합니다.

다음은 Tailspin Surveys 응용 프로그램의 예입니다.

```csharp
public class SurveyCreatorRequirement : AuthorizationHandler<SurveyCreatorRequirement>, IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SurveyCreatorRequirement requirement)
    {
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin) || 
            context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

이 클래스는 사용자가 새 설문 조사를 만들 수 있는 요구 사항을 정의합니다. 사용자는 SurveyAdmin 또는 SurveyCreator 역할을 가져야 합니다.

시작 클래스에서 하나 이상의 요구 사항을 포함하는 명명된 정책을 정의합니다. 여러 요구 사항이 있는 경우 사용자가 *모든* 요구 사항을 충족해야 권한이 부여됩니다. 다음 코드는 두 개의 정책을 정의합니다.

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy(PolicyNames.RequireSurveyCreator,
        policy =>
        {
            policy.AddRequirements(new SurveyCreatorRequirement());
            policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
            // By adding the CookieAuthenticationDefaults.AuthenticationScheme, if an authenticated
            // user is not in the appropriate role, they will be redirected to a "forbidden" page.
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });

    options.AddPolicy(PolicyNames.RequireSurveyAdmin,
        policy =>
        {
            policy.AddRequirements(new SurveyAdminRequirement());
            policy.RequireAuthenticatedUser();  
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });
});
```

또한 이 코드는 권한 부여에 실패한 경우 실행해야 하는 인증 미들웨어를 ASP.NET에 지시하는 인증 체계를 설정합니다. 이 예에서는 쿠키 인증 미들웨어가 사용자를 "사용할 수 없음" 페이지로 사용자를 리디렉션할 수 있으므로 쿠키 인증 미들웨어를 지정합니다. "사용할 수 없음" 페이지의 위치는 쿠키 미들웨어에 대한 `AccessDeniedPath` 옵션에 설정됩니다([인증 미들웨어 구성] 참조).

### <a name="authorize-controller-actions"></a>컨트롤러 작업 권한 부여
마지막으로 MVC 컨트롤러에서 작업에 대한 권한을 부여하려면 `Authorize` 특성에서 정책을 설정합니다.

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

이전 버전의 ASP.NET에서는 이 특성에서 **Roles** 속성을 설정했습니다.

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

이 기능은 ASP.NET Core에서도 지원되지만 권한 부여 정책에 비해 다음과 같은 몇 가지 단점이 있습니다.

* 특정 클레임 유형을 가정합니다. 정책은 모든 클레임 유형을 확인할 수 있습니다. 역할은 하나의 클레임 유형일 뿐입니다.
* 역할 이름이 특성에 하드 코드됩니다. 정책을 사용하면 권한 부여 논리가 모두 한 곳에서 유지되므로 업데이트하거나 구성 설정에서 로드하기가 보다 쉽습니다.
* 정책은 단순한 역할 멤버 자격으로 표현할 수 없는 보다 복잡한 권한 부여 의사 결정(예: 만 21세 이상의 나이)을 지원합니다.

## <a name="resource-based-authorization"></a>리소스 기반 권한 부여
*리소스 기반 권한 부여*는 작업의 영향을 받는 특정 리소스에 따라 권한 부여가 달라질 때마다 발생합니다. Tailspin Surveys 응용 프로그램의 모든 설문 조사에는 소유자와 참가자(0명~여러 명)가 있습니다.

* 소유자는 설문 조사를 읽고, 업데이트하고, 게시하고, 게시를 취소할 수 있습니다.
* 소유자는 설문 조사에 참가자를 할당할 수 있습니다.
* 참가자는 설문 조사를 읽고 업데이트할 수 있습니다.

"소유자"와 "참가자"는 응용 프로그램 역할이 아니고 응용 프로그램 데이터베이스에 설문 조사별로 저장됩니다. 예를 들어 사용자가 설문 조사를 삭제할 수 있는지 확인하기 위해 앱은 사용자가 해당 설문 조사의 소유자인지 확인합니다.

ASP.NET Core에서, **AuthorizationHandler**에서 파생하고 **Handle** 메서드를 재정의하여 리소스 기반 권한 부여를 구현합니다.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

이 클래스는 설문 조사 개체에 대한 강력한 형식입니다.  시작 시 DI에 클래스를 등록합니다.

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

권한 부여 확인을 수행하려면 컨트롤러에 삽입할 수 있는 **IAuthorizationService** 인터페이스를 사용합니다. 다음 코드는 사용자가 설문 조사를 읽을 수 있는지 여부를 확인합니다.

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

`Survey` 개체를 통해 전달하므로 이 호출은 `SurveyAuthorizationHandler`를 호출합니다.

인증 코드에서 사용자의 역할 기반 및 리소스 기반 사용 권한을 모두 집계한 다음 원하는 작업에 대해 집계 집합을 확인하는 것이 좋습니다.
다음은 Surveys 응용 프로그램의 예입니다. 이 응용 프로그램에서는 몇 가지 사용 권한 유형을 정의합니다.

* 관리자
* 참가자
* 작성자
* 소유자
* 판독기

또한 설문 조사에서 가능한 작업 집합을 정의합니다.

* 생성
* 읽기
* 업데이트
* 삭제
* 게시
* 게시 취소

다음 코드는 특정 사용자 및 설문 조사에 대한 사용 권한 목록을 만듭니다. 이 코드는 사용자의 응용 프로그램 역할과 설문 조사의 소유자/참가자 필드를 모두 확인합니다.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement requirement, Survey resource)
    {
        var permissions = new List<UserPermissionType>();
        int surveyTenantId = context.User.GetSurveyTenantIdValue();
        int userId = context.User.GetSurveyUserIdValue();
        string user = context.User.GetUserName();

        if (resource.TenantId == surveyTenantId)
        {
            // Admin can do anything, as long as the resource belongs to the admin's tenant.
            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin))
            {
                context.Succeed(requirement);
                return Task.FromResult(0);
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

        if (ValidateUserPermissions[requirement](permissions))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

다중 테넌트 응용 프로그램에서는 권한이 다른 테넌트의 데이터로 "누출"되지 않도록 해야 합니다. Surveys 앱에서 참가자 권한은 테넌트 전체에서 허용되므로 다른 테넌트의 사용자를 참가자로 할당할 수 있습니다. 다른 권한 유형은 해당 사용자의 테넌트에 속하는 리소스로 제한됩니다. 이 요구 사항을 적용하기 위해 코드에서 권한을 부여하기 전에 테넌트 ID를 확인합니다. `TenantId` 필드는 설문 조사를 만들 때 할당됩니다.

다음 단계에서는 사용 권한에 대해 작업(읽기, 업데이트, 삭제 등)을 확인합니다. Surveys 응용 프로그램은 함수의 조회 테이블을 사용하여 이 단계를 구현합니다.

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

[응용 프로그램 역할]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[참조 구현]: tailspin.md
[인증 미들웨어 구성]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
