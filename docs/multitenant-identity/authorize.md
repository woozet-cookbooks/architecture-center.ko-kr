---
title: "다중 테넌트 응용 프로그램의 권한 부여"
description: "다중 테넌트 응용 프로그램에서 권한 부여를 수행하는 방법"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 86c308d21f19bb3ac2a4a2240a9a03a504de5cf4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="role-based-and-resource-based-authorization"></a><span data-ttu-id="472b9-103">역할 기반 및 리소스 기반 권한 부여</span><span class="sxs-lookup"><span data-stu-id="472b9-103">Role-based and resource-based authorization</span></span>

<span data-ttu-id="472b9-104">[![GitHub](../_images/github.png) 샘플 코드][sample application]</span><span class="sxs-lookup"><span data-stu-id="472b9-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="472b9-105">[참조 구현]은 ASP.NET Core 응용 프로그램입니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-105">Our [reference implementation] is an ASP.NET Core application.</span></span> <span data-ttu-id="472b9-106">이 문서에서는 ASP.NET Core에 제공된 권한 부여 API를 사용하는 두 가지 일반적인 권한 부여 방법을 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-106">In this article we'll look at two general approaches to authorization, using the authorization APIs provided in ASP.NET Core.</span></span>

* <span data-ttu-id="472b9-107">**역할 기반 권한 부여**.</span><span class="sxs-lookup"><span data-stu-id="472b9-107">**Role-based authorization**.</span></span> <span data-ttu-id="472b9-108">사용자에게 할당된 역할에 따라 작업에 대한 권한을 부여합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-108">Authorizing an action based on the roles assigned to a user.</span></span> <span data-ttu-id="472b9-109">예를 들어 일부 작업에는 관리자 역할이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-109">For example, some actions require an administrator role.</span></span>
* <span data-ttu-id="472b9-110">**리소스 기반 권한 부여**.</span><span class="sxs-lookup"><span data-stu-id="472b9-110">**Resource-based authorization**.</span></span> <span data-ttu-id="472b9-111">특정 리소스에 따라 작업에 대한 권한을 부여합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-111">Authorizing an action based on a particular resource.</span></span> <span data-ttu-id="472b9-112">예를 들어 모든 리소스에는 소유자가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-112">For example, every resource has an owner.</span></span> <span data-ttu-id="472b9-113">소유자는 리소스를 삭제할 수 있지만 다른 사용자는 삭제할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-113">The owner can delete the resource; other users cannot.</span></span>

<span data-ttu-id="472b9-114">일반적인 앱에서는 두 가지가 혼용됩니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-114">A typical app will employ a mix of both.</span></span> <span data-ttu-id="472b9-115">예를 들어 리소스를 삭제하려면 사용자가 리소스 소유자 *또는* 관리자여야 합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-115">For example, to delete a resource, the user must be the resource owner *or* an admin.</span></span>

## <a name="role-based-authorization"></a><span data-ttu-id="472b9-116">역할 기반 권한 부여</span><span class="sxs-lookup"><span data-stu-id="472b9-116">Role-Based Authorization</span></span>
<span data-ttu-id="472b9-117">[Tailspin Surveys][Tailspin] 응용 프로그램은 다음 역할을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-117">The [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="472b9-118">관리자.</span><span class="sxs-lookup"><span data-stu-id="472b9-118">Administrator.</span></span> <span data-ttu-id="472b9-119">해당 테넌트에 속하는 모든 설문 조사에 대한 모든 CRUD 작업을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-119">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="472b9-120">작성자.</span><span class="sxs-lookup"><span data-stu-id="472b9-120">Creator.</span></span> <span data-ttu-id="472b9-121">새 설문 조사를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-121">Can create new surveys</span></span>
* <span data-ttu-id="472b9-122">읽기 권한자.</span><span class="sxs-lookup"><span data-stu-id="472b9-122">Reader.</span></span> <span data-ttu-id="472b9-123">해당 테넌트에 속하는 모든 설문 조사를 읽을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-123">Can read any surveys that belong to that tenant</span></span>

<span data-ttu-id="472b9-124">역할은 응용 프로그램의 *사용자*에게 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-124">Roles apply to *users* of the application.</span></span> <span data-ttu-id="472b9-125">Surveys 응용 프로그램에서 사용자는 관리자, 작성자 또는 읽기 권한자입니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-125">In the Surveys application, a user is either an administrator, creator, or reader.</span></span>

<span data-ttu-id="472b9-126">역할을 정의하고 관리하는 방법에 대한 자세한 내용은 [응용 프로그램 역할]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="472b9-126">For a discussion of how to define and manage roles, see [Application roles].</span></span>

<span data-ttu-id="472b9-127">역할을 관리하는 방법에 상관없이 인증 코드는 유사합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-127">Regardless of how you manage the roles, your authorization code will look similar.</span></span> <span data-ttu-id="472b9-128">ASP.NET Core에는 [권한 부여 정책][policies]이라는 추상화가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-128">ASP.NET Core has an abstraction called [authorization policies][policies].</span></span> <span data-ttu-id="472b9-129">이 기능을 통해 코드에서 권한 부여 정책을 정의한 후 컨트롤러 작업에 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-129">With this feature, you define authorization policies in code, and then apply those policies to controller actions.</span></span> <span data-ttu-id="472b9-130">이 정책은 컨트롤러에서 분리됩니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-130">The policy is decoupled from the controller.</span></span>

### <a name="create-policies"></a><span data-ttu-id="472b9-131">정책 만들기</span><span class="sxs-lookup"><span data-stu-id="472b9-131">Create policies</span></span>
<span data-ttu-id="472b9-132">정책을 정의하려면 먼저 `IAuthorizationRequirement`를 구현하는 클래스를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-132">To define a policy, first create a class that implements `IAuthorizationRequirement`.</span></span> <span data-ttu-id="472b9-133">`AuthorizationHandler`에서 파생하는 것이 가장 간단합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-133">It's easiest to derive from `AuthorizationHandler`.</span></span> <span data-ttu-id="472b9-134">`Handle` 메서드에서 관련 클레임을 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-134">In the `Handle` method, examine the relevant claim(s).</span></span>

<span data-ttu-id="472b9-135">다음은 Tailspin Surveys 응용 프로그램의 예입니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-135">Here is an example from the Tailspin Surveys application:</span></span>

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

<span data-ttu-id="472b9-136">이 클래스는 사용자가 새 설문 조사를 만들 수 있는 요구 사항을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-136">This class defines the requirement for a user to create a new survey.</span></span> <span data-ttu-id="472b9-137">사용자는 SurveyAdmin 또는 SurveyCreator 역할을 가져야 합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-137">The user must be in the SurveyAdmin or SurveyCreator role.</span></span>

<span data-ttu-id="472b9-138">시작 클래스에서 하나 이상의 요구 사항을 포함하는 명명된 정책을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-138">In your startup class, define a named policy that includes one or more requirements.</span></span> <span data-ttu-id="472b9-139">여러 요구 사항이 있는 경우 사용자가 *모든* 요구 사항을 충족해야 권한이 부여됩니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-139">If there are multiple requirements, the user must meet *every* requirement to be authorized.</span></span> <span data-ttu-id="472b9-140">다음 코드는 두 개의 정책을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-140">The following code defines two policies:</span></span>

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

<span data-ttu-id="472b9-141">또한 이 코드는 권한 부여에 실패한 경우 실행해야 하는 인증 미들웨어를 ASP.NET에 지시하는 인증 체계를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-141">This code also sets the authentication scheme, which tells ASP.NET which authentication middleware should run if authorization fails.</span></span> <span data-ttu-id="472b9-142">이 예에서는 쿠키 인증 미들웨어가 사용자를 "사용할 수 없음" 페이지로 사용자를 리디렉션할 수 있으므로 쿠키 인증 미들웨어를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-142">In this case, we specify the cookie authentication middleware, because the cookie authentication middleware can redirect the user to a "Forbidden" page.</span></span> <span data-ttu-id="472b9-143">"사용할 수 없음" 페이지의 위치는 쿠키 미들웨어에 대한 `AccessDeniedPath` 옵션에 설정됩니다([인증 미들웨어 구성] 참조).</span><span class="sxs-lookup"><span data-stu-id="472b9-143">The location of the Forbidden page is set in the `AccessDeniedPath` option for the cookie middleware; see [Configuring the authentication middleware].</span></span>

### <a name="authorize-controller-actions"></a><span data-ttu-id="472b9-144">컨트롤러 작업 권한 부여</span><span class="sxs-lookup"><span data-stu-id="472b9-144">Authorize controller actions</span></span>
<span data-ttu-id="472b9-145">마지막으로 MVC 컨트롤러에서 작업에 대한 권한을 부여하려면 `Authorize` 특성에서 정책을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-145">Finally, to authorize an action in an MVC controller, set the policy in the `Authorize` attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

<span data-ttu-id="472b9-146">이전 버전의 ASP.NET에서는 이 특성에서 **Roles** 속성을 설정했습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-146">In earlier versions of ASP.NET, you would set the **Roles** property on the attribute:</span></span>

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]

```

<span data-ttu-id="472b9-147">이 기능은 ASP.NET Core에서도 지원되지만 권한 부여 정책에 비해 다음과 같은 몇 가지 단점이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-147">This is still supported in ASP.NET Core, but it has some drawbacks compared with authorization policies:</span></span>

* <span data-ttu-id="472b9-148">특정 클레임 유형을 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-148">It assumes a particular claim type.</span></span> <span data-ttu-id="472b9-149">정책은 모든 클레임 유형을 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-149">Policies can check for any claim type.</span></span> <span data-ttu-id="472b9-150">역할은 하나의 클레임 유형일 뿐입니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-150">Roles are just a type of claim.</span></span>
* <span data-ttu-id="472b9-151">역할 이름이 특성에 하드 코드됩니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-151">The role name is hard-coded into the attribute.</span></span> <span data-ttu-id="472b9-152">정책을 사용하면 권한 부여 논리가 모두 한 곳에서 유지되므로 업데이트하거나 구성 설정에서 로드하기가 보다 쉽습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-152">With policies, the authorization logic is all in one place, making it easier to update or even load from configuration settings.</span></span>
* <span data-ttu-id="472b9-153">정책은 단순한 역할 멤버 자격으로 표현할 수 없는 보다 복잡한 권한 부여 의사 결정(예: 만 21세 이상의 나이)을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-153">Policies enable more complex authorization decisions (e.g., age >= 21) that can't be expressed by simple role membership.</span></span>

## <a name="resource-based-authorization"></a><span data-ttu-id="472b9-154">리소스 기반 권한 부여</span><span class="sxs-lookup"><span data-stu-id="472b9-154">Resource based authorization</span></span>
<span data-ttu-id="472b9-155">*리소스 기반 권한 부여*는 작업의 영향을 받는 특정 리소스에 따라 권한 부여가 달라질 때마다 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-155">*Resource based authorization* occurs whenever the authorization depends on a specific resource that will be affected by an operation.</span></span> <span data-ttu-id="472b9-156">Tailspin Surveys 응용 프로그램의 모든 설문 조사에는 소유자와 참가자(0명~여러 명)가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-156">In the Tailspin Surveys application, every survey has an owner and zero-to-many contributors.</span></span>

* <span data-ttu-id="472b9-157">소유자는 설문 조사를 읽고, 업데이트하고, 게시하고, 게시를 취소할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-157">The owner can read, update, delete, publish, and unpublish the survey.</span></span>
* <span data-ttu-id="472b9-158">소유자는 설문 조사에 참가자를 할당할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-158">The owner can assign contributors to the survey.</span></span>
* <span data-ttu-id="472b9-159">참가자는 설문 조사를 읽고 업데이트할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-159">Contributors can read and update the survey.</span></span>

<span data-ttu-id="472b9-160">"소유자"와 "참가자"는 응용 프로그램 역할이 아니고 응용 프로그램 데이터베이스에 설문 조사별로 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-160">Note that "owner" and "contributor" are not application roles; they are stored per survey, in the application database.</span></span> <span data-ttu-id="472b9-161">예를 들어 사용자가 설문 조사를 삭제할 수 있는지 확인하기 위해 앱은 사용자가 해당 설문 조사의 소유자인지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-161">To check whether a user can delete a survey, for example, the app checks whether the user is the owner for that survey.</span></span>

<span data-ttu-id="472b9-162">ASP.NET Core에서, **AuthorizationHandler**에서 파생하고 **Handle** 메서드를 재정의하여 리소스 기반 권한 부여를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-162">In ASP.NET Core, implement resource-based authorization by deriving from **AuthorizationHandler** and overriding the **Handle** method.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

<span data-ttu-id="472b9-163">이 클래스는 설문 조사 개체에 대한 강력한 형식입니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-163">Notice that this class is strongly typed for Survey objects.</span></span>  <span data-ttu-id="472b9-164">시작 시 DI에 클래스를 등록합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-164">Register the class for DI on startup:</span></span>

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

<span data-ttu-id="472b9-165">권한 부여 확인을 수행하려면 컨트롤러에 삽입할 수 있는 **IAuthorizationService** 인터페이스를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-165">To perform authorization checks, use the **IAuthorizationService** interface, which you can inject into your controllers.</span></span> <span data-ttu-id="472b9-166">다음 코드는 사용자가 설문 조사를 읽을 수 있는지 여부를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-166">The following code checks whether a user can read a survey:</span></span>

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

<span data-ttu-id="472b9-167">`Survey` 개체를 통해 전달하므로 이 호출은 `SurveyAuthorizationHandler`를 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-167">Because we pass in a `Survey` object, this call will invoke the `SurveyAuthorizationHandler`.</span></span>

<span data-ttu-id="472b9-168">인증 코드에서 사용자의 역할 기반 및 리소스 기반 사용 권한을 모두 집계한 다음 원하는 작업에 대해 집계 집합을 확인하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-168">In your authorization code, a good approach is to aggregate all of the user's role-based and resource-based permissions, then check the aggregate set against the desired operation.</span></span>
<span data-ttu-id="472b9-169">다음은 Surveys 응용 프로그램의 예입니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-169">Here is an example from the Surveys app.</span></span> <span data-ttu-id="472b9-170">이 응용 프로그램에서는 몇 가지 사용 권한 유형을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-170">The application defines several permission types:</span></span>

* <span data-ttu-id="472b9-171">관리자</span><span class="sxs-lookup"><span data-stu-id="472b9-171">Admin</span></span>
* <span data-ttu-id="472b9-172">참가자</span><span class="sxs-lookup"><span data-stu-id="472b9-172">Contributor</span></span>
* <span data-ttu-id="472b9-173">작성자</span><span class="sxs-lookup"><span data-stu-id="472b9-173">Creator</span></span>
* <span data-ttu-id="472b9-174">소유자</span><span class="sxs-lookup"><span data-stu-id="472b9-174">Owner</span></span>
* <span data-ttu-id="472b9-175">판독기</span><span class="sxs-lookup"><span data-stu-id="472b9-175">Reader</span></span>

<span data-ttu-id="472b9-176">또한 설문 조사에서 가능한 작업 집합을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-176">The application also defines a set of possible operations on surveys:</span></span>

* <span data-ttu-id="472b9-177">생성</span><span class="sxs-lookup"><span data-stu-id="472b9-177">Create</span></span>
* <span data-ttu-id="472b9-178">읽기</span><span class="sxs-lookup"><span data-stu-id="472b9-178">Read</span></span>
* <span data-ttu-id="472b9-179">업데이트</span><span class="sxs-lookup"><span data-stu-id="472b9-179">Update</span></span>
* <span data-ttu-id="472b9-180">삭제</span><span class="sxs-lookup"><span data-stu-id="472b9-180">Delete</span></span>
* <span data-ttu-id="472b9-181">게시</span><span class="sxs-lookup"><span data-stu-id="472b9-181">Publish</span></span>
* <span data-ttu-id="472b9-182">게시 취소</span><span class="sxs-lookup"><span data-stu-id="472b9-182">Unpublsh</span></span>

<span data-ttu-id="472b9-183">다음 코드는 특정 사용자 및 설문 조사에 대한 사용 권한 목록을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-183">The following code creates a list of permissions for a particular user and survey.</span></span> <span data-ttu-id="472b9-184">이 코드는 사용자의 응용 프로그램 역할과 설문 조사의 소유자/참가자 필드를 모두 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-184">Notice that this code looks at both the user's app roles, and the owner/contributor fields in the survey.</span></span>

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

<span data-ttu-id="472b9-185">다중 테넌트 응용 프로그램에서는 권한이 다른 테넌트의 데이터로 "누출"되지 않도록 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-185">In a multi-tenant application, you must ensure that permissions don't "leak" to another tenant's data.</span></span> <span data-ttu-id="472b9-186">Surveys 앱에서 참가자 권한은 테넌트 전체에서 허용되므로 다른 테넌트의 사용자를 참가자로 할당할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-186">In the Surveys app, the Contributor permission is allowed across tenants &mdash; you can assign someone from another tenant as a contriubutor.</span></span> <span data-ttu-id="472b9-187">다른 권한 유형은 해당 사용자의 테넌트에 속하는 리소스로 제한됩니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-187">The other permission types are restricted to resources that belong to that user's tenant.</span></span> <span data-ttu-id="472b9-188">이 요구 사항을 적용하기 위해 코드에서 권한을 부여하기 전에 테넌트 ID를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-188">To enforce this requirement, the code checks the tenant ID before granting the permission.</span></span> <span data-ttu-id="472b9-189">`TenantId` 필드는 설문 조사를 만들 때 할당됩니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-189">(The `TenantId` field as assigned when the survey is created.)</span></span>

<span data-ttu-id="472b9-190">다음 단계에서는 사용 권한에 대해 작업(읽기, 업데이트, 삭제 등)을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-190">The next step is to check the operation (read, update, delete, etc) against the permissions.</span></span> <span data-ttu-id="472b9-191">Surveys 응용 프로그램은 함수의 조회 테이블을 사용하여 이 단계를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="472b9-191">The Surveys app implements this step by using a lookup table of functions:</span></span>

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

<span data-ttu-id="472b9-192">[**다음**][web-api]</span><span class="sxs-lookup"><span data-stu-id="472b9-192">[**Next**][web-api]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[응용 프로그램 역할]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[참조 구현]: tailspin.md
[인증 미들웨어 구성]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
