---
title: 다중 테넌트 응용 프로그램에서 등록 및 테넌트 온보딩
description: 다중 테넌트 응용 프로그램에서 테넌트를 등록하는 방법
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: claims
pnp.series.next: app-roles
ms.openlocfilehash: dde577d5bab63fb436d52fb4548399d5bd8bb38f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
ms.locfileid: "26582744"
---
# <a name="tenant-sign-up-and-onboarding"></a>테넌트 등록 및 온보딩

[![GitHub](../_images/github.png) 샘플 코드][sample application]

이 문서는 고객이 응용 프로그램에 조직을 등록하도록 하는 다중 테넌트 응용 프로그램에서 *등록* 프로세스를 구현하는 방법을 설명합니다.
등록 프로세스를 구현하는 여러 가지 이유가 있습니다.

* AD 관리자가 고객의 전체 조직에 동의하여 응용 프로그램을 사용하도록 합니다.
* 신용 카드 지불 또는 기타 고객 정보를 수집합니다.
* 응용 프로그램에 필요한 모든 일회성 테넌트별 설치를 수행합니다.

## <a name="admin-consent-and-azure-ad-permissions"></a>관리자 동의 및 Azure AD 사용 권한
Azure AD를 인증하려면 응용 프로그램은 사용자의 디렉터리에 대한 액세스가 필요합니다. 최소한 응용 프로그램에서 사용자의 프로필을 읽을 수 있는 권한이 필요합니다. 사용자가 처음으로 로그인할 때 Azure AD는 요청되는 권한을 나열하는 동의 페이지를 보여 줍니다. **동의함**을 클릭하여 응용 프로그램에 권한을 부여합니다.

기본적으로 동의는 사용자 단위로 부여됩니다. 로그인하는 모든 사용자에게 동의 페이지가 표시됩니다. 그러나 Azure AD는 AD 관리자가 전체 조직에 대한 동의를 허용하는 *관리자 동의*를 지원합니다.

관리자 동의 흐름을 사용하면 동의 페이지는 AD 관리자가 전체 테넌트를 대신하여 사용 권한을 부여하는 것을 명시합니다.

![관리자 동의 프롬프트](./images/admin-consent.png)

관리자가 **동의함**을 클릭한 후 동일한 테넌트 내의 다른 사용자는 로그인할 수 있고 Azure AD는 동의 화면을 건너뜁니다.

전체 조직을 대신하여 권한을 부여하므로 AD 관리자만이 관리자 동의를 부여할 수 있습니다. 관리자가 아닌 사용자가 관리자 동의 흐름으로 인증하려고 하는 경우 Azure AD는 오류를 표시합니다.

![동의 오류](./images/consent-error.png)

응용 프로그램이 나중에 추가 권한이 필요한 경우 고객은 다시 등록하고 업데이트된 사용 권한에 동의해야 합니다.  

## <a name="implementing-tenant-sign-up"></a>테넌트 등록 구현
[Tailspin 설문 조사][Tailspin] 응용 프로그램의 경우 등록 프로세스에 대한 몇 가지 요구 사항을 정의했습니다.

* 테넌트는 사용자가 로그인하려면 등록해야 합니다.
* 등록은 관리자 동의 흐름을 사용합니다.
* 등록은 응용 프로그램 데이터베이스에 사용자의 테넌트를 추가합니다.
* 테넌트를 등록한 후 응용 프로그램은 온보딩 페이지를 보여 줍니다.

이 섹션에서 등록 프로세스의 구현을 안내합니다.
"등록" 및 "로그인"은 응용 프로그램 개념이라는 점을 이해해야 합니다. 인증 흐름 중 Azure AD는 사용자가 등록 프로세스에 있는지 여부를 본질적으로 알지 못합니다. 컨텍스트 추적은 응용 프로그램에 달려 있습니다.

익명 사용자가 설문 조사 응용 프로그램을 방문하면 하나는 로그인으로 하나는 "회사 등록"(등록)으로 두 개의 단추가 표시됩니다.

![응용 프로그램 등록 페이지](./images/sign-up-page.png)

이러한 단추는 `AccountController` 클래스에서 작업을 호출합니다.

`SignIn` 작업은 OpenID Connect 미들웨어를 인증 끝점으로 리디렉션할 수 있게 하는 **ChallegeResult**를 반환합니다. 이는 ASP.NET Core에서 인증을 트리거하는 기본 방법입니다.  

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

이제 `SignUp` 작업을 비교합니다.

```csharp
[AllowAnonymous]
public IActionResult SignUp()
{
    var state = new Dictionary<string, string> { { "signup", "true" }};
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties(state)
        {
            RedirectUri = Url.Action(nameof(SignUpCallback), "Account")
        });
}
```

`SignIn`과 마찬가지로 `SignUp` 작업도 `ChallengeResult`를 반환합니다. 하지만 이번에 `AuthenticationProperties` in the `ChallengeResult`에 상태 정보의 일부를 추가합니다.

* 등록: 사용자가 등록 프로세스를 시작했음을 나타내는 부울 플래그.

`AuthenticationProperties` 의 상태 정보는 인증 흐름 중 왕복하는 OpenID Connect [상태] 매개 변수에 추가됩니다.

![State 매개 변수](./images/state-parameter.png)

사용자가 Azure AD에서 인증하고 응용 프로그램으로 리디렉션된 후 인증 티켓은 상태를 포함합니다. 이 팩트를 사용하여 "등록" 값이 전체 인증 흐름에서 유지되도록 합니다.

## <a name="adding-the-admin-consent-prompt"></a>관리자 동의 프롬프트 추가
Azure AD에서 관리자 동의 흐름은 "prompt" 매개 변수를 인증 요청의 쿼리 문자열에 추가하여 트리거됩니다.

```
/authorize?prompt=admin_consent&...
```

설문 조사 응용 프로그램은 `RedirectToAuthenticationEndpoint` 이벤트 중 프롬프트를 추가합니다. 이 이벤트는 미들웨어가 인증 끝점에 리디렉션하기 직전에 호출됩니다.

```csharp
public override Task RedirectToAuthenticationEndpoint(RedirectContext context)
{
    if (context.IsSigningUp())
    {
        context.ProtocolMessage.Prompt = "admin_consent";
    }

    _logger.RedirectToIdentityProvider();
    return Task.FromResult(0);
}
```

설정` ProtocolMessage.Prompt` 은(는) "prompt" 매개 변수를 인증 요청에 추가하도록 미들웨어에 알려 줍니다.

프롬프트는 등록하는 동안만 필요합니다. 일반 로그인은 이를 포함하면 안됩니다. 서로 구분하려면 인증 상태에서 `signup` 값을 확인합니다. 다음 확장 메서드는 이러한 조건을 확인합니다.

```csharp
internal static bool IsSigningUp(this BaseControlContext context)
{
    Guard.ArgumentNotNull(context, nameof(context));

    string signupValue;
    // Check the HTTP context and convert to string
    if ((context.Ticket == null) ||
        (!context.Ticket.Properties.Items.TryGetValue("signup", out signupValue)))
    {
        return false;
    }

    // We have found the value, so see if it's valid
    bool isSigningUp;
    if (!bool.TryParse(signupValue, out isSigningUp))
    {
        // The value for signup is not a valid boolean, throw                
        throw new InvalidOperationException($"'{signupValue}' is an invalid boolean value");
    }

    return isSigningUp;
}
```

## <a name="registering-a-tenant"></a>테넌트 등록
설문 조사 응용 프로그램은 응용 프로그램 데이터베이스에 각 테넌트 및 사용자에 대한 정보를 저장합니다.

![테넌트 테이블](./images/tenant-table.png)

테넌트 테이블에서 IssuerValue는 테넌트에 대한 발급자 클레임의 값입니다. Azure AD의 경우 이는 `https://sts.windows.net/<tentantID>` 이고 테넌트별로 고유한 값을 제공합니다.

새 테넌트를 등록하면 설문 조사 응용 프로그램은 데이터베이스에 테넌트 레코드를 작성합니다. 이는 `AuthenticationValidated` 이벤트 내에서 발생합니다. (ID 토큰은 아직 확인되지 않았기 때문에 클레임 값을 신뢰할 수 없으므로 이 이벤트 전에 실행하지 마십시오. [인증]을 참조하세요.

설문 조사 응용 프로그램에서 관련 코드는 다음과 같습니다.

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.AuthenticationTicket.Principal;
    var userId = principal.GetObjectIdentifierValue();
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    _logger.AuthenticationValidated(userId, issuerValue);

    // Normalize the claims first.
    NormalizeClaims(principal);
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue)
        .ConfigureAwait(false);

    if (context.IsSigningUp())
    {
        if (tenant == null)
        {
            tenant = await SignUpTenantAsync(context, tenantManager)
                .ConfigureAwait(false);
        }

        // In this case, we need to go ahead and set up the user signing us up.
        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
    else
    {
        if (tenant == null)
        {
            _logger.UnregisteredUserSignInAttempted(userId, issuerValue);
            throw new SecurityTokenValidationException($"Tenant {issuerValue} is not registered");
        }

        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
}
```

이 코드는 다음을 수행합니다.

1. 테넌트의 발급자 값이 데이터베이스에 이미 있는지 확인합니다. 테넌트를 등록하지 않은 경우 `FindByIssuerValueAsync` 은(는) null을 반환합니다.
2. 사용자가 등록하는 경우:
   1. 데이터베이스(`SignUpTenantAsync`)에 테넌트를 추가합니다.
   2. 데이터베이스(`CreateOrUpdateUserAsync`)에 인증된 사용자를 추가합니다.
3. 그렇지 않은 경우 기본 로그인 흐름을 완료합니다.
   1. 테넌트의 발급자가 데이터베이스에 없는 경우 테넌트가 등록되지 않았으므로 고객이 등록해야 함을 의미합니다. 이 경우 인증에 실패하는 예외를 throw합니다.
   2. 그렇지 않은 경우 항목이 없으면 이 사용자에 대한 데이터베이스 레코드를 만듭니다(`CreateOrUpdateUserAsync`).

다음은 테넌트를 데이터베이스에 추가하는 `SignUpTenantAsync` 메서드입니다.

```csharp
private async Task<Tenant> SignUpTenantAsync(BaseControlContext context, TenantManager tenantManager)
{
    Guard.ArgumentNotNull(context, nameof(context));
    Guard.ArgumentNotNull(tenantManager, nameof(tenantManager));

    var principal = context.Ticket.Principal;
    var issuerValue = principal.GetIssuerValue();
    var tenant = new Tenant
    {
        IssuerValue = issuerValue,
        Created = DateTimeOffset.UtcNow
    };

    try
    {
        await tenantManager.CreateAsync(tenant)
            .ConfigureAwait(false);
    }
    catch(Exception ex)
    {
        _logger.SignUpTenantFailed(principal.GetObjectIdentifierValue(), issuerValue, ex);
        throw;
    }

    return tenant;
}
```

다음은 설문 조사 응용 프로그램에서 전체 등록 흐름에 대한 요약입니다.

1. 사용자는 **등록** 단추를 클릭합니다.
2. `AccountController.SignUp` 작업은 challege 결과를 반환합니다.  인증 상태는 "등록" 값을 포함합니다.
3. `RedirectToAuthenticationEndpoint` 이벤트에서 `admin_consent` 프롬프트를 추가합니다.
4. OpenID Connect 미들웨어는 Azure AD로 리디렉션하고 사용자는 인증합니다.
5. `AuthenticationValidated` 이벤트에서 “등록" 상태를 찾습니다.
6. 데이터베이스에 테넌트를 추가합니다.

[**다음**][app roles]

<!-- Links -->
[app roles]: app-roles.md
[Tailspin]: tailspin.md

[상태]: http://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[인증]: authenticate.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
