---
title: 다중 테넌트 응용 프로그램에서 클레임 기반 ID 작업
description: 발급자 유효성 검사 및 권한 부여에 클레임을 사용하는 방법
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authenticate
pnp.series.next: signup
ms.openlocfilehash: 61788d9759715b21ef1bdda59c5b54d923fd8f62
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="work-with-claims-based-identities"></a><span data-ttu-id="c1d85-103">클레임 기반 ID 사용</span><span class="sxs-lookup"><span data-stu-id="c1d85-103">Work with claims-based identities</span></span>

<span data-ttu-id="c1d85-104">[![GitHub](../_images/github.png) 샘플 코드][sample application]</span><span class="sxs-lookup"><span data-stu-id="c1d85-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

## <a name="claims-in-azure-ad"></a><span data-ttu-id="c1d85-105">Azure AD의 클레임</span><span class="sxs-lookup"><span data-stu-id="c1d85-105">Claims in Azure AD</span></span>
<span data-ttu-id="c1d85-106">사용자가 로그인하면 Azure AD는 사용자에 대한 클레임 집합을 포함하는 ID 토큰을 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-106">When a user signs in, Azure AD sends an ID token that contains a set of claims about the user.</span></span> <span data-ttu-id="c1d85-107">클레임은 단순한 정보 조각이며, 키/값 쌍으로 표현됩니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-107">A claim is simply a piece of information, expressed as a key/value pair.</span></span> <span data-ttu-id="c1d85-108">예를 들어 `email`=`bob@contoso.com`입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-108">For example, `email`=`bob@contoso.com`.</span></span>  <span data-ttu-id="c1d85-109">클레임에는 사용자를 인증하고 클레임을 만드는 엔터티인 발급자(이 경우는 Azure AD)가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-109">Claims have an issuer &mdash; in this case, Azure AD &mdash; which is the entity that authenticates the user and creates the claims.</span></span> <span data-ttu-id="c1d85-110">발급자를 신뢰하므로 클레임도 신뢰합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-110">You trust the claims because you trust the issuer.</span></span> <span data-ttu-id="c1d85-111">(반대로 발급자를 신뢰하지 않으면 클레임도 신뢰하지 않습니다.)</span><span class="sxs-lookup"><span data-stu-id="c1d85-111">(Conversely, if you don't trust the issuer, don't trust the claims!)</span></span>

<span data-ttu-id="c1d85-112">개요:</span><span class="sxs-lookup"><span data-stu-id="c1d85-112">At a high level:</span></span>

1. <span data-ttu-id="c1d85-113">사용자를 인증합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-113">The user authenticates.</span></span>
2. <span data-ttu-id="c1d85-114">IDP는 클레임 집합을 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-114">The IDP sends a set of claims.</span></span>
3. <span data-ttu-id="c1d85-115">앱은 클레임을 정규화 또는 보강(선택 사항)합니다</span><span class="sxs-lookup"><span data-stu-id="c1d85-115">The app normalizes or augments the claims (optional).</span></span>
4. <span data-ttu-id="c1d85-116">앱은 권한 부여를 결정하기 위해 클레임을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-116">The app uses the claims to make authorization decisions.</span></span>

<span data-ttu-id="c1d85-117">OpenID Connect에서 사용자가 받은 클레임 집합은 인증 요청의 [범위 매개 변수]에 의해 제어됩니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-117">In OpenID Connect, the set of claims that you get is controlled by the [scope parameter] of the authentication request.</span></span> <span data-ttu-id="c1d85-118">그러나, Azure AD는 OpenID Connect를 통해서 제한된 클레임 집합을 발급합니다. [지원되는 토큰 및 클레임 유형]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c1d85-118">However, Azure AD issues a limited set of claims through OpenID Connect; see [Supported Token and Claim Types].</span></span> <span data-ttu-id="c1d85-119">사용자에 관한 자세한 정보를 원하는 경우, Azure AD Graph API를 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-119">If you want more information about the user, you'll need to use the Azure AD Graph API.</span></span>

<span data-ttu-id="c1d85-120">다음은 앱에서 일반적으로 고려하는 AAD의 일부 클레임입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-120">Here are some of the claims from AAD that an app might typically care about:</span></span>

| <span data-ttu-id="c1d85-121">ID 토큰에서 클레임 유형</span><span class="sxs-lookup"><span data-stu-id="c1d85-121">Claim type in ID token</span></span> | <span data-ttu-id="c1d85-122">설명</span><span class="sxs-lookup"><span data-stu-id="c1d85-122">Description</span></span> |
| --- | --- |
| <span data-ttu-id="c1d85-123">aud</span><span class="sxs-lookup"><span data-stu-id="c1d85-123">aud</span></span> |<span data-ttu-id="c1d85-124">토큰을 발급한 사람입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-124">Who the token was issued for.</span></span> <span data-ttu-id="c1d85-125">응용 프로그램의 클라이언트 ID가 됩니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-125">This will be the application's client ID.</span></span> <span data-ttu-id="c1d85-126">일반적으로 미들웨어가 자동으로 유효성 검사를 수행하기 때문에 이 클레임에 대해 걱정할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-126">Generally, you shouldn't need to worry about this claim, because the middleware automatically validates it.</span></span> <span data-ttu-id="c1d85-127">예제: `"91464657-d17a-4327-91f3-2ed99386406f"`</span><span class="sxs-lookup"><span data-stu-id="c1d85-127">Example:  `"91464657-d17a-4327-91f3-2ed99386406f"`</span></span> |
| <span data-ttu-id="c1d85-128">groups</span><span class="sxs-lookup"><span data-stu-id="c1d85-128">groups</span></span> |<span data-ttu-id="c1d85-129">사용자가 구성원인 AAD 그룹 목록입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-129">A list of AAD groups of which the user is a member.</span></span> <span data-ttu-id="c1d85-130">예제: `["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]`</span><span class="sxs-lookup"><span data-stu-id="c1d85-130">Example: `["93e8f556-8661-4955-87b6-890bc043c30f", "fc781505-18ef-4a31-a7d5-7d931d7b857e"]`</span></span> |
| <span data-ttu-id="c1d85-131">iss</span><span class="sxs-lookup"><span data-stu-id="c1d85-131">iss</span></span> |<span data-ttu-id="c1d85-132">OIDC 토큰의 [발급자] 입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-132">The [issuer] of the OIDC token.</span></span> <span data-ttu-id="c1d85-133">예제: `https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/`</span><span class="sxs-lookup"><span data-stu-id="c1d85-133">Example: `https://sts.windows.net/b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4/`</span></span> |
| <span data-ttu-id="c1d85-134">이름</span><span class="sxs-lookup"><span data-stu-id="c1d85-134">name</span></span> |<span data-ttu-id="c1d85-135">사용자의 표시 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-135">The user's display name.</span></span> <span data-ttu-id="c1d85-136">예제: `"Alice A."`</span><span class="sxs-lookup"><span data-stu-id="c1d85-136">Example: `"Alice A."`</span></span> |
| <span data-ttu-id="c1d85-137">oid</span><span class="sxs-lookup"><span data-stu-id="c1d85-137">oid</span></span> |<span data-ttu-id="c1d85-138">AAD에서 사용자의 개체 ID.</span><span class="sxs-lookup"><span data-stu-id="c1d85-138">The object identifier for the user in AAD.</span></span> <span data-ttu-id="c1d85-139">이 값은 변경이 불가능하고 다시 사용할 수 없는 사용자 ID입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-139">This value is the immutable and non-reusable identifier of the user.</span></span> <span data-ttu-id="c1d85-140">이 값은 전자 메일이 아닌, 사용자의 고유 ID로 사용합니다. 전자 메일 주소는 변경 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-140">Use this value, not email, as a unique identifier for users; email addresses can change.</span></span> <span data-ttu-id="c1d85-141">앱에서 Azure AD Graph API를 사용할 경우, 개체 ID는 프로필 정보를 쿼리하는 데 사용되는 값입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-141">If you use the Azure AD Graph API in your app, object ID is that value used to query profile information.</span></span> <span data-ttu-id="c1d85-142">예제: `"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"`</span><span class="sxs-lookup"><span data-stu-id="c1d85-142">Example: `"59f9d2dc-995a-4ddf-915e-b3bb314a7fa4"`</span></span> |
| <span data-ttu-id="c1d85-143">roles</span><span class="sxs-lookup"><span data-stu-id="c1d85-143">roles</span></span> |<span data-ttu-id="c1d85-144">사용자에 대한 앱 역할 목록입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-144">A list of app roles for the user.</span></span>    <span data-ttu-id="c1d85-145">예제: `["SurveyCreator"]`</span><span class="sxs-lookup"><span data-stu-id="c1d85-145">Example: `["SurveyCreator"]`</span></span> |
| <span data-ttu-id="c1d85-146">tid</span><span class="sxs-lookup"><span data-stu-id="c1d85-146">tid</span></span> |<span data-ttu-id="c1d85-147">테넌트 ID.</span><span class="sxs-lookup"><span data-stu-id="c1d85-147">Tenant ID.</span></span> <span data-ttu-id="c1d85-148">이 값은 Azure AD에서 테넌트에 대한 고유한 식별자입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-148">This value is a unique identifier for the tenant in Azure AD.</span></span> <span data-ttu-id="c1d85-149">예제: `"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"`</span><span class="sxs-lookup"><span data-stu-id="c1d85-149">Example: `"b9bd2162-77ac-4fb2-8254-5c36e9c0a9c4"`</span></span> |
| <span data-ttu-id="c1d85-150">unique_name</span><span class="sxs-lookup"><span data-stu-id="c1d85-150">unique_name</span></span> |<span data-ttu-id="c1d85-151">사람이 읽을 수 있는 사용자의 표시 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-151">A human readable display name of the user.</span></span> <span data-ttu-id="c1d85-152">예제: `"alice@contoso.com"`</span><span class="sxs-lookup"><span data-stu-id="c1d85-152">Example: `"alice@contoso.com"`</span></span> |
| <span data-ttu-id="c1d85-153">upn</span><span class="sxs-lookup"><span data-stu-id="c1d85-153">upn</span></span> |<span data-ttu-id="c1d85-154">사용자 계정 이름.</span><span class="sxs-lookup"><span data-stu-id="c1d85-154">User principal name.</span></span> <span data-ttu-id="c1d85-155">예제: `"alice@contoso.com"`</span><span class="sxs-lookup"><span data-stu-id="c1d85-155">Example: `"alice@contoso.com"`</span></span> |

<span data-ttu-id="c1d85-156">이 표는 클레임 유형을 ID 토큰에 표시되는 대로 나열합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-156">This table lists the claim types as they appear in the ID token.</span></span> <span data-ttu-id="c1d85-157">ASP.NET Core에서 OpenID Connect 미들웨어는 사용자 계정에 대한 클레임 컬렉션을 채울 때 일부 클레임 유형을 변환합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-157">In ASP.NET Core, the OpenID Connect middleware converts some of the claim types when it populates the Claims collection for the user principal:</span></span>

* <span data-ttu-id="c1d85-158">oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`</span><span class="sxs-lookup"><span data-stu-id="c1d85-158">oid > `http://schemas.microsoft.com/identity/claims/objectidentifier`</span></span>
* <span data-ttu-id="c1d85-159">tid > `http://schemas.microsoft.com/identity/claims/tenantid`</span><span class="sxs-lookup"><span data-stu-id="c1d85-159">tid > `http://schemas.microsoft.com/identity/claims/tenantid`</span></span>
* <span data-ttu-id="c1d85-160">unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`</span><span class="sxs-lookup"><span data-stu-id="c1d85-160">unique_name > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`</span></span>
* <span data-ttu-id="c1d85-161">upn > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`</span><span class="sxs-lookup"><span data-stu-id="c1d85-161">upn > `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`</span></span>

## <a name="claims-transformations"></a><span data-ttu-id="c1d85-162">클레임 변환</span><span class="sxs-lookup"><span data-stu-id="c1d85-162">Claims transformations</span></span>
<span data-ttu-id="c1d85-163">인증 흐름 중에 IDP에서 가져온 클레임을 수정하려고 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-163">During the authentication flow, you might want to modify the claims that you get from the IDP.</span></span> <span data-ttu-id="c1d85-164">ASP.NET Core에서는 OpenID Connect 미들웨어의 **AuthenticationValidated** 이벤트 내부에서 클레임 변환을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-164">In ASP.NET Core, you can perform claims transformation inside of the **AuthenticationValidated** event from the OpenID Connect middleware.</span></span> <span data-ttu-id="c1d85-165">( [인증 이벤트]참조)</span><span class="sxs-lookup"><span data-stu-id="c1d85-165">(See [Authentication events].)</span></span>

<span data-ttu-id="c1d85-166">**AuthenticationValidated** 중에 추가한 모든 클레임은 세션 인증 쿠키에 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-166">Any claims that you add during **AuthenticationValidated** are stored in the session authentication cookie.</span></span> <span data-ttu-id="c1d85-167">Azure AD로 다시 푸시되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-167">They don't get pushed back to Azure AD.</span></span>

<span data-ttu-id="c1d85-168">클레임 변환에 대한 몇 가지 예는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-168">Here are some examples of claims transformation:</span></span>

* <span data-ttu-id="c1d85-169">**클레인 정규화** 또는 클레임을 사용자 사이에서 일관되게 만들기.</span><span class="sxs-lookup"><span data-stu-id="c1d85-169">**Claims normalization**, or making claims consistent across users.</span></span> <span data-ttu-id="c1d85-170">이 방법은 특히 유사한 정보에 대해서 다양한 클레임 유형을 사용할 수 있는 다중 IDP에서 클레임을 받는 경우에 적절합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-170">This is particularly relevant if you are getting claims from multiple IDPs, which might use different claim types for similar information.</span></span>
  <span data-ttu-id="c1d85-171">예를 들어, Azure AD는 사용자 전자 메일을 포함한 "upn" 클레임을 전송합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-171">For example, Azure AD sends a "upn" claim that contains the user's email.</span></span> <span data-ttu-id="c1d85-172">다른 IDP는 "email" 클레임을 전송할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-172">Other IDPs might send an "email" claim.</span></span> <span data-ttu-id="c1d85-173">다음 코드는 "upn" 클레임을 "email" 클레임으로 변환합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-173">The following code converts the "upn" claim into an "email" claim:</span></span>
  
  ```csharp
  var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
  if (!string.IsNullOrWhiteSpace(email))
  {
      identity.AddClaim(new Claim(ClaimTypes.Email, email));
  }
  ```
* <span data-ttu-id="c1d85-174">존재하지 않는 클레임의 **기본 클레임 값**을 추가합니다(예를 들어 기본 역할에 사용자 할당).</span><span class="sxs-lookup"><span data-stu-id="c1d85-174">Add **default claim values** for claims that aren't present &mdash; for example, assigning a user to a default role.</span></span> <span data-ttu-id="c1d85-175">경우에 따라, 이를 통해 인증 논리가 단순화될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-175">In some cases this can simplify authorization logic.</span></span>
* <span data-ttu-id="c1d85-176">**사용자 지정 클레임 유형** 을 사용자에 대한 응용 프로그램 관련 정보와 함께 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-176">Add **custom claim types** with application-specific information about the user.</span></span> <span data-ttu-id="c1d85-177">예를 들어 데이터베이스에 사용자에 대한 일부 정보를 저장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-177">For example, you might store some information about the user in a database.</span></span> <span data-ttu-id="c1d85-178">이 정보와 함께 사용자 지정 클레임을 인증 티켓에 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-178">You could add a custom claim with this information to the authentication ticket.</span></span> <span data-ttu-id="c1d85-179">클레임이 쿠키에 저장되므로 로그인 세션당 1번 데이터베이스에서 가져와야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-179">The claim is stored in a cookie, so you only need to get it from the database once per login session.</span></span> <span data-ttu-id="c1d85-180">반면, 쿠키를 지나치게 많이 만들고 싶지 않으므로 쿠키 크기와 데이터베이스 조회 간에 균형을 맞추어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-180">On the other hand, you also want to avoid creating excessively large cookies, so you need to consider the trade-off between cookie size versus database lookups.</span></span>   

<span data-ttu-id="c1d85-181">인증 흐름이 완료된 후, 클레임은 `HttpContext.User`에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-181">After the authentication flow is complete, the claims are available in `HttpContext.User`.</span></span> <span data-ttu-id="c1d85-182">이 때, 클레임을 읽기 전용 컬렉션으로 처리해야 합니다(예: 인증 결정을 내리는 데 사용).</span><span class="sxs-lookup"><span data-stu-id="c1d85-182">At that point, you should treat them as a read-only collection &mdash; e.g., use them to make authorization decisions.</span></span>

## <a name="issuer-validation"></a><span data-ttu-id="c1d85-183">발급자 유효성 검사</span><span class="sxs-lookup"><span data-stu-id="c1d85-183">Issuer validation</span></span>
<span data-ttu-id="c1d85-184">OpenID Connect에서 발급자 클레임("iss")은 ID 토큰을 발급한 IDP를 식별합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-184">In OpenID Connect, the issuer claim ("iss") identifies the IDP that issued the ID token.</span></span> <span data-ttu-id="c1d85-185">OIDC 인증 흐름의 일부는 발급자 클레임이 실제 발급자와 일치하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-185">Part of the OIDC authentication flow is to verify that the issuer claim matches the actual issuer.</span></span> <span data-ttu-id="c1d85-186">OIDC 미들웨어가 이를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-186">The OIDC middleware handles this for you.</span></span>

<span data-ttu-id="c1d85-187">Azure AD에서 발급자 값은 AD 테넌트당 고유합니다(`https://sts.windows.net/<tenantID>`).</span><span class="sxs-lookup"><span data-stu-id="c1d85-187">In Azure AD, the issuer value is unique per AD tenant (`https://sts.windows.net/<tenantID>`).</span></span> <span data-ttu-id="c1d85-188">따라서 응용 프로그램은 발급자가 앱에 로그인하도록 허용된 테넌트를 나타내는지 확인하기 위해 추가 검사를 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-188">Therefore, an application should do an additional check, to make sure the issuer represents a tenant that is allowed to sign in to the app.</span></span>

<span data-ttu-id="c1d85-189">단일 테넌트 응용 프로그램의 경우 발급자가 사용자 고유의 테넌트인지 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-189">For a single-tenant application, you can just check that the issuer is your own tenant.</span></span> <span data-ttu-id="c1d85-190">실제로는 OIDC 미들웨어가 기본적으로 이 작업을 자동으로 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-190">In fact, the OIDC middleware does this automatically by default.</span></span> <span data-ttu-id="c1d85-191">다중 테넌트 앱에서는 서로 다른 테넌트에 해당하는 여러 발급자를 허용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-191">In a multi-tenant app, you need to allow for multiple issuers, corresponding to the different tenants.</span></span> <span data-ttu-id="c1d85-192">다음은 사용되는 일반적인 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-192">Here is a general approach to use:</span></span>

* <span data-ttu-id="c1d85-193">OIDC 미들웨어 옵션에서 **ValidateIssuer** 를 false로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-193">In the OIDC middleware options, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="c1d85-194">자동 확인이 해제됩니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-194">This turns off the automatic check.</span></span>
* <span data-ttu-id="c1d85-195">테넌트를 등록할 때 테넌트와 발급자를 사용자 DB에 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-195">When a tenant signs up, store the tenant and the issuer in your user DB.</span></span>
* <span data-ttu-id="c1d85-196">사용자가 로그인할 때마다 데이터베이스에서 발급자를 조회합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-196">Whenever a user signs in, look up the issuer in the database.</span></span> <span data-ttu-id="c1d85-197">발급자를 찾을 수 없으면 해당 테넌트가 등록되지 않은 것입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-197">If the issuer isn't found, it means that tenant hasn't signed up.</span></span> <span data-ttu-id="c1d85-198">등록 페이지로 리디렉션할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-198">You can redirect them to a sign up page.</span></span>
* <span data-ttu-id="c1d85-199">특정 테넌트를 블랙리스트에 올릴 수도 있습니다(예를 들어, 구독 요금을 지불하지 않은 고객).</span><span class="sxs-lookup"><span data-stu-id="c1d85-199">You could also blacklist certain tenants; for example, for customers that didn't pay their subscription.</span></span>

<span data-ttu-id="c1d85-200">자세한 내용은 [다중 테넌트 응용 프로그램에서 등록 및 테넌트 온보딩][signup]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c1d85-200">For a more detailed discussion, see [Sign-up and tenant onboarding in a multitenant application][signup].</span></span>

## <a name="using-claims-for-authorization"></a><span data-ttu-id="c1d85-201">권한 부여에 클레임 사용</span><span class="sxs-lookup"><span data-stu-id="c1d85-201">Using claims for authorization</span></span>
<span data-ttu-id="c1d85-202">클레임이 있음으로 해서 사용자 ID는 더 이상 단일체가 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-202">With claims, a user's identity is no longer a monolithic entity.</span></span> <span data-ttu-id="c1d85-203">예를 들어, 사용자는 전자 메일 주소, 전화 번호, 생일, 성별 등을 가집니다. 아마 사용자 IDP는 이 모든 정보를 저장할 것입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-203">For example, a user might have an email address, phone number, birthday, gender, etc. Maybe the user's IDP stores all of this information.</span></span> <span data-ttu-id="c1d85-204">그런데 사용자를 인증할 때, 대개 이 정보의 일부를 클레임으로 받습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-204">But when you authenticate the user, you'll typically get a subset of these as claims.</span></span> <span data-ttu-id="c1d85-205">이 모델에서 사용자 ID는 단순히 클레임의 묶음입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-205">In this model, the user's identity is simply a bundle of claims.</span></span> <span data-ttu-id="c1d85-206">사용자에 대한 인증을 결정할 때, 특정한 클레임 집합을 찾을 것입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-206">When you make authorization decisions about a user, you will look for particular sets of claims.</span></span> <span data-ttu-id="c1d85-207">다시 말해서, "사용자 X가 동작 Y를 수행할 수 있나요"라는 질문은 결국 "사용자 X가 클레임 Z를 갖고 있나요"입니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-207">In other words, the question "Can user X perform action Y" ultimately becomes "Does user X have claim Z".</span></span>

<span data-ttu-id="c1d85-208">클레임을 확인하는 몇 가지 기본 패턴은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-208">Here are some basic patterns for checking claims.</span></span>

* <span data-ttu-id="c1d85-209">사용자에게 특정 값이 있는 특정 클레임이 있는지 확인하려면</span><span class="sxs-lookup"><span data-stu-id="c1d85-209">To check that the user has a particular claim with a particular value:</span></span>
  
   ```csharp
   if (User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
   ```
   <span data-ttu-id="c1d85-210">이 코드는 사용자에게 "Admin" 값이 있는 역할 클레임이 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-210">This code checks whether the user has a Role claim with the value "Admin".</span></span> <span data-ttu-id="c1d85-211">사용자에게 역할 클레임이 없거나 여러 역할 클레임이 있는 경우 적절하게 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-211">It correctly handles the case where the user has no Role claim or multiple Role claims.</span></span>
  
   <span data-ttu-id="c1d85-212">**ClaimTypes** 클래스는 일반적으로 사용되는 클레임 유형에 대한 상수를 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-212">The **ClaimTypes** class defines constants for commonly-used claim types.</span></span> <span data-ttu-id="c1d85-213">그러나 클레임 유형에 대해 임의의 문자열 값을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c1d85-213">However, you can use any string value for the claim type.</span></span>
* <span data-ttu-id="c1d85-214">클레임 유형에 대해 단일 값을 가져오려면 최대 1개 값이 있을 것으로 예상되는 경우:</span><span class="sxs-lookup"><span data-stu-id="c1d85-214">To get a single value for a claim type, when you expect there to be at most one value:</span></span>
  
  ```csharp
  string email = User.FindFirst(ClaimTypes.Email)?.Value;
  ```
* <span data-ttu-id="c1d85-215">클레임 유형에 대한 모든 값을 가져오려면:</span><span class="sxs-lookup"><span data-stu-id="c1d85-215">To get all the values for a claim type:</span></span>
  
  ```csharp
  IEnumerable<Claim> groups = User.FindAll("groups");
  ```

<span data-ttu-id="c1d85-216">자세한 내용은 [다중 테넌트 응용 프로그램에서 역할 기반 및 리소스 기반 권한 부여][authorization]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c1d85-216">For more information, see [Role-based and resource-based authorization in multitenant applications][authorization].</span></span>

<span data-ttu-id="c1d85-217">[**다음**][signup]</span><span class="sxs-lookup"><span data-stu-id="c1d85-217">[**Next**][signup]</span></span>


<!-- Links -->

[범위 매개 변수]: http://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[scope parameter]: http://nat.sakimura.org/2012/01/26/scopes-and-claims-in-openid-connect/
[지원되는 토큰 및 클레임 유형]: /azure/active-directory/active-directory-token-and-claims/
[Supported Token and Claim Types]: /azure/active-directory/active-directory-token-and-claims/
[발급자]: http://openid.net/specs/openid-connect-core-1_0.html#IDToken
[issuer]: http://openid.net/specs/openid-connect-core-1_0.html#IDToken
[인증 이벤트]: authenticate.md#authentication-events
[Authentication events]: authenticate.md#authentication-events
[signup]: signup.md
[Claims-Based Authorization]: /aspnet/core/security/authorization/claims
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[authorization]: authorize.md
