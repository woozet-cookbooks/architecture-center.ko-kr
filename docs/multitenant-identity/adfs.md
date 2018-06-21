---
title: 고객의 AD FS로 페더레이션
description: 다중 테넌트 응용 프로그램에서 고객의 AD FS로 페더레이션하는 방법
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: token-cache
pnp.series.next: client-assertion
ms.openlocfilehash: 08bf567085a940287de310f61b9f447d0ce5d5ec
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/23/2018
ms.locfileid: "29477447"
---
# <a name="federate-with-a-customers-ad-fs"></a><span data-ttu-id="b71c0-103">고객의 AD FS로 페더레이션</span><span class="sxs-lookup"><span data-stu-id="b71c0-103">Federate with a customer's AD FS</span></span>

<span data-ttu-id="b71c0-104">이 문서에서는 다중 테넌트 SaaS 응용 프로그램이 고객의 AD FS와 페더레이션하기 위해 AD FS(Active Directory Federation Services)를 통해 인증을 지원할 수 있는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-104">This article describes how a multi-tenant SaaS application can support authentication via Active Directory Federation Services (AD FS), in order to federate with a customer's AD FS.</span></span>

## <a name="overview"></a><span data-ttu-id="b71c0-105">개요</span><span class="sxs-lookup"><span data-stu-id="b71c0-105">Overview</span></span>
<span data-ttu-id="b71c0-106">Azure AD(Azure Active Directory)를 통해 Office365 및 Dynamics CRM Online 고객을 포함하여 Azure AD 테넌트에서 사용자를 쉽게 로그인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-106">Azure Active Directory (Azure AD) makes it easy to sign in users from Azure AD tenants, including Office365 and Dynamics CRM Online customers.</span></span> <span data-ttu-id="b71c0-107">그러나 회사 인트라넷에서 온-프레미스 Active Directory를 사용하는 고객은 어떨까요?</span><span class="sxs-lookup"><span data-stu-id="b71c0-107">But what about customers who use on-premise Active Directory on a corporate intranet?</span></span>

<span data-ttu-id="b71c0-108">이러한 고객을 위한 한 가지 옵션은 [Azure AD Connect]를 사용하여 고객의 온-프레미스 AD와 Azure AD를 동기화하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-108">One option is for these customers to sync their on-premise AD with Azure AD, using [Azure AD Connect].</span></span> <span data-ttu-id="b71c0-109">그러나 일부 고객은 회사 IT 정책이나 기타 원인으로 이러한 방법을 사용할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-109">However, some customers may be unable to use this approach, due to corporate IT policy or other reasons.</span></span> <span data-ttu-id="b71c0-110">이 경우 다른 옵션은 AD FS(Active Directory Federation Services)를 통해 페더레이션하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-110">In that case, another option is to federate through Active Directory Federation Services (AD FS).</span></span>

<span data-ttu-id="b71c0-111">이 시나리오를 사용하려면</span><span class="sxs-lookup"><span data-stu-id="b71c0-111">To enable this scenario:</span></span>

* <span data-ttu-id="b71c0-112">고객에게 인터넷 연결 AD FS 팜이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-112">The customer must have an Internet-facing AD FS farm.</span></span>
* <span data-ttu-id="b71c0-113">SaaS 공급자는 자신의 AD FS 팜을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-113">The SaaS provider deploys their own AD FS farm.</span></span>
* <span data-ttu-id="b71c0-114">고객 및 SaaS 공급자는 [페더레이션 트러스트]를 설정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-114">The customer and the SaaS provider must set up [federation trust].</span></span> <span data-ttu-id="b71c0-115">수동 프로세스입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-115">This is a manual process.</span></span>

<span data-ttu-id="b71c0-116">트러스트 관계에는 세 가지 주요 역할이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-116">There are three main roles in the trust relation:</span></span>

* <span data-ttu-id="b71c0-117">고객의 AD FS는 [계정 파트너]이며, 고객의 AD에서 사용자 인증 및 사용자 클레임으로 보안 토큰 생성을 담당합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-117">The customer's AD FS is the [account partner], responsible for authenticating users from the customer's AD, and creating security tokens with user claims.</span></span>
* <span data-ttu-id="b71c0-118">SaaS 공급자의 AD FS는 [리소스 파트너]이며, 계정 파트너를 트러스트하고 사용자 클레임을 받습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-118">The SaaS provider's AD FS is the [resource partner], which trusts the account partner and receives the user claims.</span></span>
* <span data-ttu-id="b71c0-119">응용 프로그램은 SaaS 공급자의 AD FS에서 RP(신뢰 당사자)로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-119">The application is configured as a relying party (RP) in the SaaS provider's AD FS.</span></span>
  
  ![페더레이션 트러스트](./images/federation-trust.png)

> [!NOTE]
> <span data-ttu-id="b71c0-121">이 문서에서는 응용 프로그램이 인증 프로토콜로 OpenID Connect를 사용한다고 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-121">In this article, we assume the application uses OpenID connect as the authentication protocol.</span></span> <span data-ttu-id="b71c0-122">다른 옵션은 WS-Federation을 사용하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-122">Another option is to use WS-Federation.</span></span>
> 
> <span data-ttu-id="b71c0-123">OpenID Connect의 경우 SaaS 공급자는 Windows Server 2016에서 실행되는 AD FS 2016을 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-123">For OpenID Connect, the SaaS provider must use AD FS 2016, running in Windows Server 2016.</span></span> <span data-ttu-id="b71c0-124">AD FS 3.0은 OpenID Connect를 지원하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-124">AD FS 3.0 does not support OpenID Connect.</span></span>
> 
> <span data-ttu-id="b71c0-125">ASP.NET Core는 기본적으로 WS-Federation을 지원하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-125">ASP.NET Core does not include out-of-the-box support for WS-Federation.</span></span>
> 
> 

<span data-ttu-id="b71c0-126">ASP.NET 4에서 WS-Federation을 사용하는 예제는 [active-directory-dotnet-webapp-wsfederation 샘플][active-directory-dotnet-webapp-wsfederation]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b71c0-126">For an example of using WS-Federation with ASP.NET 4, see the [active-directory-dotnet-webapp-wsfederation sample][active-directory-dotnet-webapp-wsfederation].</span></span>

## <a name="authentication-flow"></a><span data-ttu-id="b71c0-127">인증 흐름</span><span class="sxs-lookup"><span data-stu-id="b71c0-127">Authentication flow</span></span>
1. <span data-ttu-id="b71c0-128">사용자가 "로그인"을 클릭하면 응용 프로그램은 SaaS 공급자의 AD FS에서 OpenID Connect 끝점으로 리디렉션됩니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-128">When the user clicks "sign in", the application redirects to an OpenID Connect endpoint on the SaaS provider's AD FS.</span></span>
2. <span data-ttu-id="b71c0-129">사용자가 자신의 조직 사용자 이름("`alice@corp.contoso.com`")을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-129">The user enters his or her organizational user name ("`alice@corp.contoso.com`").</span></span> <span data-ttu-id="b71c0-130">AD FS는 고객의 AD FS로 리디렉션하기 위해 홈 영역 검색을 사용하며 여기서 사용자는 자신의 자격 증명을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-130">AD FS uses home realm discovery to redirect to the customer's AD FS, where the user enters their credentials.</span></span>
3. <span data-ttu-id="b71c0-131">고객의 AD FS는 WF-Federation(또는 SAML)을 사용하여 사용자 클레임을 SaaS 공급자의 AD FS로 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-131">The customer's AD FS sends user claims to the SaaS provider's AD FS, using WF-Federation (or SAML).</span></span>
4. <span data-ttu-id="b71c0-132">클레임은 OpenID Connect를 사용하여 AD FS에서 앱으로 흐릅니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-132">Claims flow from AD FS to the app, using OpenID Connect.</span></span> <span data-ttu-id="b71c0-133">여기에 WS-Federation에서 프로토콜 전환이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-133">This requires a protocol transition from WS-Federation.</span></span>

## <a name="limitations"></a><span data-ttu-id="b71c0-134">제한 사항</span><span class="sxs-lookup"><span data-stu-id="b71c0-134">Limitations</span></span>
<span data-ttu-id="b71c0-135">기본적으로 신뢰 당사자 응용 프로그램은 다음 표와 같이 id_token에서 사용할 수 있는 고정 클레임 집합만 받습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-135">By default, the relying party application receives only a fixed set of claims available in the id_token, shown in the following table.</span></span> <span data-ttu-id="b71c0-136">AD FS 2016에서는 OpenID Connect 시나리오의 id_token을 사용자 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-136">With AD FS 2016, you can customize the id_token in OpenID Connect scenarios.</span></span> <span data-ttu-id="b71c0-137">자세한 내용은 [AD FS의 사용자 지정 ID 토큰](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b71c0-137">For more information, see [Custom ID Tokens in AD FS](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016).</span></span>

| <span data-ttu-id="b71c0-138">클레임</span><span class="sxs-lookup"><span data-stu-id="b71c0-138">Claim</span></span> | <span data-ttu-id="b71c0-139">설명</span><span class="sxs-lookup"><span data-stu-id="b71c0-139">Description</span></span> |
| --- | --- |
| <span data-ttu-id="b71c0-140">aud</span><span class="sxs-lookup"><span data-stu-id="b71c0-140">aud</span></span> |<span data-ttu-id="b71c0-141">대상.</span><span class="sxs-lookup"><span data-stu-id="b71c0-141">Audience.</span></span> <span data-ttu-id="b71c0-142">클레임이 발급된 응용 프로그램입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-142">The application for which the claims were issued.</span></span> |
| <span data-ttu-id="b71c0-143">authenticationinstant</span><span class="sxs-lookup"><span data-stu-id="b71c0-143">authenticationinstant</span></span> |<span data-ttu-id="b71c0-144">[인증 인스턴트].</span><span class="sxs-lookup"><span data-stu-id="b71c0-144">[Authentication instant].</span></span> <span data-ttu-id="b71c0-145">인증이 발생한 시간입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-145">The time at which authentication occurred.</span></span> |
| <span data-ttu-id="b71c0-146">c_hash</span><span class="sxs-lookup"><span data-stu-id="b71c0-146">c_hash</span></span> |<span data-ttu-id="b71c0-147">코드 해시 값.</span><span class="sxs-lookup"><span data-stu-id="b71c0-147">Code hash value.</span></span> <span data-ttu-id="b71c0-148">토큰 콘텐츠의 해시입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-148">This is a hash of the token contents.</span></span> |
| <span data-ttu-id="b71c0-149">exp</span><span class="sxs-lookup"><span data-stu-id="b71c0-149">exp</span></span> |<span data-ttu-id="b71c0-150">[만료 시간].</span><span class="sxs-lookup"><span data-stu-id="b71c0-150">[Expiration time].</span></span> <span data-ttu-id="b71c0-151">이 시간 후에는 토큰이 더 이상 허용되지 않는 시간입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-151">The time after which the token will no longer be accepted.</span></span> |
| <span data-ttu-id="b71c0-152">iat</span><span class="sxs-lookup"><span data-stu-id="b71c0-152">iat</span></span> |<span data-ttu-id="b71c0-153">발급 시간.</span><span class="sxs-lookup"><span data-stu-id="b71c0-153">Issued at.</span></span> <span data-ttu-id="b71c0-154">토큰이 발급된 시간입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-154">The time when the token was issued.</span></span> |
| <span data-ttu-id="b71c0-155">iss</span><span class="sxs-lookup"><span data-stu-id="b71c0-155">iss</span></span> |<span data-ttu-id="b71c0-156">발급자.</span><span class="sxs-lookup"><span data-stu-id="b71c0-156">Issuer.</span></span> <span data-ttu-id="b71c0-157">이 클레임 값은 항상 리소스 파트너의 AD FS입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-157">The value of this claim is always the resource partner's AD FS.</span></span> |
| <span data-ttu-id="b71c0-158">이름</span><span class="sxs-lookup"><span data-stu-id="b71c0-158">name</span></span> |<span data-ttu-id="b71c0-159">사용자 이름.</span><span class="sxs-lookup"><span data-stu-id="b71c0-159">User name.</span></span> <span data-ttu-id="b71c0-160">예제: `john@corp.fabrikam.com`</span><span class="sxs-lookup"><span data-stu-id="b71c0-160">Example: `john@corp.fabrikam.com`</span></span> |
| <span data-ttu-id="b71c0-161">nameidentifier</span><span class="sxs-lookup"><span data-stu-id="b71c0-161">nameidentifier</span></span> |<span data-ttu-id="b71c0-162">[이름 식별자].</span><span class="sxs-lookup"><span data-stu-id="b71c0-162">[Name identifier].</span></span> <span data-ttu-id="b71c0-163">토큰이 발급된 엔터티 이름의 식별자입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-163">The identifier for the name of the entity for which the token was issued.</span></span> |
| <span data-ttu-id="b71c0-164">nonce</span><span class="sxs-lookup"><span data-stu-id="b71c0-164">nonce</span></span> |<span data-ttu-id="b71c0-165">세션 nonce.</span><span class="sxs-lookup"><span data-stu-id="b71c0-165">Session nonce.</span></span> <span data-ttu-id="b71c0-166">AD FS에서 재생 공격을 방지하기 위해 생성하는 고유 값입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-166">A unique value generated by AD FS to help prevent replay attacks.</span></span> |
| <span data-ttu-id="b71c0-167">upn</span><span class="sxs-lookup"><span data-stu-id="b71c0-167">upn</span></span> |<span data-ttu-id="b71c0-168">사용자 계정 이름(UPN).</span><span class="sxs-lookup"><span data-stu-id="b71c0-168">User principal name (UPN).</span></span> <span data-ttu-id="b71c0-169">예제: `john@corp.fabrikam.com`</span><span class="sxs-lookup"><span data-stu-id="b71c0-169">Example: `john@corp.fabrikam.com`</span></span> |
| <span data-ttu-id="b71c0-170">pwd_exp</span><span class="sxs-lookup"><span data-stu-id="b71c0-170">pwd_exp</span></span> |<span data-ttu-id="b71c0-171">암호 만료 기간.</span><span class="sxs-lookup"><span data-stu-id="b71c0-171">Password expiration period.</span></span> <span data-ttu-id="b71c0-172">사용자 암호 또는 유사한 인증 비밀(예: PIN)이 만료되기까지의</span><span class="sxs-lookup"><span data-stu-id="b71c0-172">The number of seconds until the user's password or a similar authentication secret, such as a PIN.</span></span> <span data-ttu-id="b71c0-173">시간(초)입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-173">expires.</span></span> |

> [!NOTE]
> <span data-ttu-id="b71c0-174">"iss" 클레임에는 파트너의 AD FS가 포함됩니다. 일반적으로 이 클레임은 SaaS 공급자를 발급자로 식별하며</span><span class="sxs-lookup"><span data-stu-id="b71c0-174">The "iss" claim contains the AD FS of the partner (typically, this claim will identify the SaaS provider as the issuer).</span></span> <span data-ttu-id="b71c0-175">고객의 AD FS를 식별하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-175">It does not identify the customer's AD FS.</span></span> <span data-ttu-id="b71c0-176">고객 도메인을 UPN의 일부로 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-176">You can find the customer's domain as part of the UPN.</span></span>
> 
> 

<span data-ttu-id="b71c0-177">이 문서의 나머지 부분에서는 RP(앱) 및 계정 파트너(고객) 간에 트러스트 관계를 설정하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-177">The rest of this article describes how to set up the trust relationship between the RP (the app) and the account partner (the customer).</span></span>

## <a name="ad-fs-deployment"></a><span data-ttu-id="b71c0-178">AD FS 배포</span><span class="sxs-lookup"><span data-stu-id="b71c0-178">AD FS deployment</span></span>
<span data-ttu-id="b71c0-179">SaaS 공급자는 온-프레미스 또는 Azure VM에 AD FS를 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-179">The SaaS provider can deploy AD FS either on-premise or on Azure VMs.</span></span> <span data-ttu-id="b71c0-180">보안 및 가용성을 위해 다음 지침이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-180">For security and availability, the following guidelines are important:</span></span>

* <span data-ttu-id="b71c0-181">두 개 이상의 AD FS 서버와 두 개의 AD FS 프록시 서버를 배포하여 AD FS 서비스의 최적의 가용성을 달성합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-181">Deploy at least two AD FS servers and two AD FS proxy servers to achieve the best availability of the AD FS service.</span></span>
* <span data-ttu-id="b71c0-182">도메인 컨트롤러와 AD FS 서버는 인터넷에 직접 노출하면 안 되며 여기에 직접 액세스할 수 있는 가상 네트워크에 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-182">Domain controllers and AD FS servers should never be exposed directly to the Internet and should be in a virtual network with direct access to them.</span></span>
* <span data-ttu-id="b71c0-183">AD FS 서버를 인터넷에 게시하는 데는 웹 응용 프로그램 프록시(이전에 AD FS 프록시)를 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-183">Web application proxies (previously AD FS proxies) must be used to publish AD FS servers to the Internet.</span></span>

<span data-ttu-id="b71c0-184">Azure에서 유사한 토폴로지를 설정하기 위해서는 가상 네트워크, NSG, azure VM 및 가용성 집합을 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-184">To set up a similar topology in Azure requires the use of Virtual networks, NSG’s, azure VM’s and availability sets.</span></span> <span data-ttu-id="b71c0-185">자세한 내용은 [Azure Virtual Machines에 Windows Server Active Directory를 배포하기 위한 지침][active-directory-on-azure]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b71c0-185">For more details, see [Guidelines for Deploying Windows Server Active Directory on Azure Virtual Machines][active-directory-on-azure].</span></span>

## <a name="configure-openid-connect-authentication-with-ad-fs"></a><span data-ttu-id="b71c0-186">AD FS를 사용하여 OpenID Connect 인증 구성</span><span class="sxs-lookup"><span data-stu-id="b71c0-186">Configure OpenID Connect authentication with AD FS</span></span>
<span data-ttu-id="b71c0-187">SaaS 공급자는 응용 프로그램 및 AD FS 간의 OpenID Connect를 사용하도록 설정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-187">The SaaS provider must enable OpenID Connect between the application and AD FS.</span></span> <span data-ttu-id="b71c0-188">이렇게 하려면 AD FS에서 응용 프로그램 그룹을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-188">To do so, add an application group in AD FS.</span></span>  <span data-ttu-id="b71c0-189">이 [블로그 게시물]의 "OpenId Connect 로그인 AD FS를 위한 웹앱 설정" 아래에서 상세 지침을 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-189">You can find detailed instructions in this [blog post], under " Setting up a Web App for OpenId Connect sign in AD FS."</span></span> 

<span data-ttu-id="b71c0-190">그런 다음 OpenID Connect 미들웨어를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-190">Next, configure the OpenID Connect middleware.</span></span> <span data-ttu-id="b71c0-191">메타데이터 끝점은 `https://domain/adfs/.well-known/openid-configuration`이며, 여기서 도메인은 SaaS 공급자의 AD FS 도메인입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-191">The metadata endpoint is `https://domain/adfs/.well-known/openid-configuration`, where domain is the SaaS provider's AD FS domain.</span></span>

<span data-ttu-id="b71c0-192">일반적으로 이것을 다른 OpenID Connect 끝점(예: AAD)과 결합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-192">Typically you might combine this with other OpenID Connect endpoints (such as AAD).</span></span> <span data-ttu-id="b71c0-193">사용자가 올바른 인증 끝점으로 보내지도록 두 개의 서로 다른 로그인 단추 또는 이를 구분하기 위한 기타 방법이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-193">You'll need two different sign-in buttons or some other way to distinguish them, so that the user is sent to the correct authentication endpoint.</span></span>

## <a name="configure-the-ad-fs-resource-partner"></a><span data-ttu-id="b71c0-194">AD FS 리소스 파트너 구성</span><span class="sxs-lookup"><span data-stu-id="b71c0-194">Configure the AD FS Resource Partner</span></span>
<span data-ttu-id="b71c0-195">SaaS 공급자는 ADFS를 통한 연결을 원하는 각 고객을 위해 다음을 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-195">The SaaS provider must do the following for each customer that wants to connect via ADFS:</span></span>

1. <span data-ttu-id="b71c0-196">클레임 공급자 트러스트를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-196">Add a claims provider trust.</span></span>
2. <span data-ttu-id="b71c0-197">클레임 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-197">Add claims rules.</span></span>
3. <span data-ttu-id="b71c0-198">홈 영역 검색을 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-198">Enable home-realm discovery.</span></span>

<span data-ttu-id="b71c0-199">다음은 자세한 단계입니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-199">Here are the steps in more detail.</span></span>

### <a name="add-the-claims-provider-trust"></a><span data-ttu-id="b71c0-200">클레임 공급자 트러스트 추가</span><span class="sxs-lookup"><span data-stu-id="b71c0-200">Add the claims provider trust</span></span>
1. <span data-ttu-id="b71c0-201">서버 관리자에서 **도구**를 클릭하고 **AD FS 관리**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-201">In Server Manager, click **Tools**, and then select **AD FS Management**.</span></span>
2. <span data-ttu-id="b71c0-202">콘솔 트리의 **AD FS** 아래에서 **클레임 공급자 트러스트**를 마우스 오른쪽 단추로 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-202">In the console tree, under **AD FS**, right click **Claims Provider Trusts**.</span></span> <span data-ttu-id="b71c0-203">**클레임 공급자 트러스트 추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-203">Select **Add Claims Provider Trust**.</span></span>
3. <span data-ttu-id="b71c0-204">**시작** 을 클릭하여 마법사를 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-204">Click **Start** to start the wizard.</span></span>
4. <span data-ttu-id="b71c0-205">"온라인으로 게시된 또는 로컬 네트워크에서 클레임 공급자에 대한 데이터 가져오기" 옵션을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-205">Select the option "Import data about the claims provider published online or on a local network".</span></span> <span data-ttu-id="b71c0-206">고객의 페더레이션 메타데이터 끝점 URI를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-206">Enter the URI of the customer's federation metadata endpoint.</span></span> <span data-ttu-id="b71c0-207">(예: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`) 이 정보는 고객으로부터 받아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-207">(Example: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`.) You will need to get this from the customer.</span></span>
5. <span data-ttu-id="b71c0-208">기본 옵션을 사용하여 마법사를 완료합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-208">Complete the wizard using the default options.</span></span>

### <a name="edit-claims-rules"></a><span data-ttu-id="b71c0-209">클레임 규칙 편집</span><span class="sxs-lookup"><span data-stu-id="b71c0-209">Edit claims rules</span></span>
1. <span data-ttu-id="b71c0-210">새로 추가된 클레임 공급자 트러스트를 마우스 오른쪽 단추로 클릭하고 **클레임 규칙 편집**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-210">Right-click the newly added claims provider trust, and select **Edit Claims Rules**.</span></span>
2. <span data-ttu-id="b71c0-211">**규칙 추가**를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-211">Click **Add Rule**.</span></span>
3. <span data-ttu-id="b71c0-212">"들어오는 클레임 통과 또는 필터링"을 선택하고 **다음**을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-212">Select "Pass Through or Filter an Incoming Claim" and click **Next**.</span></span>
   <span data-ttu-id="b71c0-213">![변환 클레임 규칙 추가 마법사](./images/edit-claims-rule.png)</span><span class="sxs-lookup"><span data-stu-id="b71c0-213">![Add Transform Claim Rule Wizard](./images/edit-claims-rule.png)</span></span>
4. <span data-ttu-id="b71c0-214">규칙의 이름을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-214">Enter a name for the rule.</span></span>
5. <span data-ttu-id="b71c0-215">"들어오는 클레임 유형"에서 **UPN**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-215">Under "Incoming claim type", select **UPN**.</span></span>
6. <span data-ttu-id="b71c0-216">"모든 클레임 값 통과"를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-216">Select "Pass through all claim values".</span></span>
   <span data-ttu-id="b71c0-217">![변환 클레임 규칙 추가 마법사](./images/edit-claims-rule2.png)</span><span class="sxs-lookup"><span data-stu-id="b71c0-217">![Add Transform Claim Rule Wizard](./images/edit-claims-rule2.png)</span></span>
7. <span data-ttu-id="b71c0-218">**Finish**를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-218">Click **Finish**.</span></span>
8. <span data-ttu-id="b71c0-219">2 - 7단계를 반복하고 들어오는 클레임 유형에 대해 **앵커 클레임 유형** 을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-219">Repeat steps 2 - 7, and specify **Anchor Claim Type** for the incoming claim type.</span></span>
9. <span data-ttu-id="b71c0-220">**확인** 을 클릭하여 마법사를 완료합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-220">Click **OK** to complete the wizard.</span></span>

### <a name="enable-home-realm-discovery"></a><span data-ttu-id="b71c0-221">홈 영역 검색을 사용하도록 설정</span><span class="sxs-lookup"><span data-stu-id="b71c0-221">Enable home-realm discovery</span></span>
<span data-ttu-id="b71c0-222">다음 PowerShell 스크립트를 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-222">Run the following PowerShell script:</span></span>

```
Set-ADFSClaimsProviderTrust -TargetName "name" -OrganizationalAccountSuffix @("suffix")
```

<span data-ttu-id="b71c0-223">여기서 "name"은 클레임 공급자 트러스트의 친숙한 이름이고 "suffix"는 고객의 AD에 대한 UPN 접미사입니다(예: "corp.fabrikam.com").</span><span class="sxs-lookup"><span data-stu-id="b71c0-223">where "name" is the friendly name of the claims provider trust, and "suffix" is the UPN suffix for the customer's AD (example, "corp.fabrikam.com").</span></span>

<span data-ttu-id="b71c0-224">이 구성을 사용하여 최종 사용자는 자신의 조직 계정을 입력할 수 있으며 AD FS가 해당 클레임 공급자를 자동으로 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-224">With this configuration, end users can type in their organizational account, and AD FS automatically selects the corresponding claims provider.</span></span> <span data-ttu-id="b71c0-225">"특정 전자 메일 접미사를 사용하도록 ID 공급자 구성" 섹션 아래 [AD FS 로그인 페이지 사용자 지정]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b71c0-225">See [Customizing the AD FS Sign-in Pages], under the section "Configure Identity Provider to use certain email suffixes".</span></span>

## <a name="configure-the-ad-fs-account-partner"></a><span data-ttu-id="b71c0-226">AD FS 계정 파트너 구성</span><span class="sxs-lookup"><span data-stu-id="b71c0-226">Configure the AD FS Account Partner</span></span>
<span data-ttu-id="b71c0-227">고객은 다음을 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-227">The customer must do the following:</span></span>

1. <span data-ttu-id="b71c0-228">신뢰 당사자(RP) 트러스트를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-228">Add a relying party (RP) trust.</span></span>
2. <span data-ttu-id="b71c0-229">클레임 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-229">Adds claims rules.</span></span>

### <a name="add-the-rp-trust"></a><span data-ttu-id="b71c0-230">RP 트러스트 추가</span><span class="sxs-lookup"><span data-stu-id="b71c0-230">Add the RP trust</span></span>
1. <span data-ttu-id="b71c0-231">서버 관리자에서 **도구**를 클릭하고 **AD FS 관리**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-231">In Server Manager, click **Tools**, and then select **AD FS Management**.</span></span>
2. <span data-ttu-id="b71c0-232">콘솔 트리의 **AD FS** 아래에서 **신뢰 당사자 트러스트**를 마우스 오른쪽 단추로 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-232">In the console tree, under **AD FS**, right click **Relying Party Trusts**.</span></span> <span data-ttu-id="b71c0-233">**신뢰 당사자 트러스트 추가**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-233">Select **Add Relying Party Trust**.</span></span>
3. <span data-ttu-id="b71c0-234">**클레임 인식**을 선택하고 **시작**을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-234">Select **Claims Aware** and click **Start**.</span></span>
4. <span data-ttu-id="b71c0-235">**데이터 원본 선택** 페이지에서 "온라인으로 게시된 또는 로컬 네트워크에서 클레임 공급자에 대한 데이터 가져오기" 옵션을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-235">On the **Select Data Source** page, select the option "Import data about the claims provider published online or on a local network".</span></span> <span data-ttu-id="b71c0-236">SaaS 공급자의 페더레이션 메타데이터 끝점의 URI를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-236">Enter the URI of the SaaS provider's federation metadata endpoint.</span></span>
   <span data-ttu-id="b71c0-237">![신뢰 당사자 트러스트 추가 마법사](./images/add-rp-trust.png)</span><span class="sxs-lookup"><span data-stu-id="b71c0-237">![Add Relying Party Trust Wizard](./images/add-rp-trust.png)</span></span>
5. <span data-ttu-id="b71c0-238">**표시 이름 지정** 페이지에서 이름을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-238">On the **Specify Display Name** page, enter any name.</span></span>
6. <span data-ttu-id="b71c0-239">**Access Control 정책 선택** 페이지에서 정책을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-239">On the **Choose Access Control Policy** page, choose a policy.</span></span> <span data-ttu-id="b71c0-240">조직에 있는 모든 사람을 허용하거나 특정 보안 그룹을 선택할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-240">You could permit everyone in the organization, or choose a specific security group.</span></span>
   <span data-ttu-id="b71c0-241">![신뢰 당사자 트러스트 추가 마법사](./images/add-rp-trust2.png)</span><span class="sxs-lookup"><span data-stu-id="b71c0-241">![Add Relying Party Trust Wizard](./images/add-rp-trust2.png)</span></span>
7. <span data-ttu-id="b71c0-242">**정책** 상자에 필요한 매개 변수를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-242">Enter any parameters required in the **Policy** box.</span></span>
8. <span data-ttu-id="b71c0-243">**다음** 을 클릭하여 마법사를 완료합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-243">Click **Next** to complete the wizard.</span></span>

### <a name="add-claims-rules"></a><span data-ttu-id="b71c0-244">클레임 규칙 추가</span><span class="sxs-lookup"><span data-stu-id="b71c0-244">Add claims rules</span></span>
1. <span data-ttu-id="b71c0-245">새로 추가된 신뢰 당사자 트러스트를 마우스 오른쪽 단추로 클릭하고 **클레임 발급 정책 편집**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-245">Right-click the newly added relying party trust, and select **Edit Claim Issuance Policy**.</span></span>
2. <span data-ttu-id="b71c0-246">**규칙 추가**를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-246">Click **Add Rule**.</span></span>
3. <span data-ttu-id="b71c0-247">"LDAP 특성을 클레임으로 전송"을 선택하고 **다음**을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-247">Select "Send LDAP Attributes as Claims" and click **Next**.</span></span>
4. <span data-ttu-id="b71c0-248">규칙의 이름을 입력합니다(예: "UPN").</span><span class="sxs-lookup"><span data-stu-id="b71c0-248">Enter a name for the rule, such as "UPN".</span></span>
5. <span data-ttu-id="b71c0-249">**특성 저장소** 아래에서 **Active Directory**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-249">Under **Attribute store**, select **Active Directory**.</span></span>
   <span data-ttu-id="b71c0-250">![변환 클레임 규칙 추가 마법사](./images/add-claims-rules.png)</span><span class="sxs-lookup"><span data-stu-id="b71c0-250">![Add Transform Claim Rule Wizard](./images/add-claims-rules.png)</span></span>
6. <span data-ttu-id="b71c0-251">**LDAP 특성의 매핑** 섹션에서:</span><span class="sxs-lookup"><span data-stu-id="b71c0-251">In the **Mapping of LDAP attributes** section:</span></span>
   * <span data-ttu-id="b71c0-252">**LDAP 특성** 아래에서 **User-Principal-Name**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-252">Under **LDAP Attribute**, select **User-Principal-Name**.</span></span>
   * <span data-ttu-id="b71c0-253">**나가는 클레임 유형** 아래에서 **UPN**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-253">Under **Outgoing Claim Type**, select **UPN**.</span></span>
     <span data-ttu-id="b71c0-254">![변환 클레임 규칙 추가 마법사](./images/add-claims-rules2.png)</span><span class="sxs-lookup"><span data-stu-id="b71c0-254">![Add Transform Claim Rule Wizard](./images/add-claims-rules2.png)</span></span>
7. <span data-ttu-id="b71c0-255">**Finish**를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-255">Click **Finish**.</span></span>
8. <span data-ttu-id="b71c0-256">**규칙 추가** 를 다시 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-256">Click **Add Rule** again.</span></span>
9. <span data-ttu-id="b71c0-257">"사용자 지정 규칙을 사용하여 클레임 보내기"를 선택하고 **다음**을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-257">Select "Send Claims Using a Custom Rule" and click **Next**.</span></span>
10. <span data-ttu-id="b71c0-258">규칙의 이름을 입력합니다(예: "앵커 클레임 유형").</span><span class="sxs-lookup"><span data-stu-id="b71c0-258">Enter a name for the rule, such as "Anchor Claim Type".</span></span>
11. <span data-ttu-id="b71c0-259">**사용자 지정 규칙**아래에서 다음을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-259">Under **Custom rule**, enter the following:</span></span>
    
    ```
    EXISTS([Type == "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype"])=>
    issue (Type = "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype",
          Value = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn");
    ```
    
    <span data-ttu-id="b71c0-260">이 규칙은 `anchorclaimtype` 유형의 클레임을 발급합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-260">This rule issues a claim of type `anchorclaimtype`.</span></span> <span data-ttu-id="b71c0-261">이 클레임은 신뢰 당사자에게 UPN을 변경할 수 없는 사용자 ID로 사용하도록 알립니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-261">The claim tells the relying party to use UPN as the user's immutable ID.</span></span>
12. <span data-ttu-id="b71c0-262">**Finish**를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-262">Click **Finish**.</span></span>
13. <span data-ttu-id="b71c0-263">**확인** 을 클릭하여 마법사를 완료합니다.</span><span class="sxs-lookup"><span data-stu-id="b71c0-263">Click **OK** to complete the wizard.</span></span>


<!-- Links -->
[Azure AD Connect]: /azure/active-directory/active-directory-aadconnect/
[페더레이션 트러스트]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[federation trust]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[계정 파트너]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[account partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[리소스 파트너]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[resource partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[인증 인스턴트]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Authentication instant]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[만료 시간]: http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Expiration time]: http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[이름 식별자]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[Name identifier]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[active-directory-on-azure]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[블로그 게시물]: http://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[blog post]: http://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[AD FS 로그인 페이지 사용자 지정]: https://technet.microsoft.com/library/dn280950.aspx
[Customizing the AD FS Sign-in Pages]: https://technet.microsoft.com/library/dn280950.aspx
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[client assertion]: client-assertion.md
[active-directory-dotnet-webapp-wsfederation]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation
