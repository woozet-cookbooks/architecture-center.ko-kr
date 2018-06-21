---
title: 다중 테넌트 응용 프로그램에 대한 ID 관리
description: 다중 테넌트 앱에서 인증, 권한 부여 및 ID 관리에 대한 모범 사례입니다.
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.next: tailspin
ms.openlocfilehash: c363ac01e798b522fa95f39586e28fe3af5fae4a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
ms.locfileid: "24851563"
---
# <a name="manage-identity-in-multitenant-applications"></a><span data-ttu-id="fbeb8-103">다중 테넌트 응용 프로그램의 ID 관리</span><span class="sxs-lookup"><span data-stu-id="fbeb8-103">Manage Identity in Multitenant Applications</span></span>

<span data-ttu-id="fbeb8-104">이 문서 시리즈에서는 인증 및 ID 관리에 Azure AD를 사용할 때 다중 테넌트 지원에 대한 모범 사례를 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-104">This series of articles describes best practices for multitenancy, when using Azure AD for authentication and identity management.</span></span>

<span data-ttu-id="fbeb8-105">[![GitHub](../_images/github.png) 샘플 코드][sample application]</span><span class="sxs-lookup"><span data-stu-id="fbeb8-105">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="fbeb8-106">다중 테넌트 응용 프로그램을 빌드할 때 첫 번째 과제 중 하나는 이제 모든 사용자가 테넌트에 속하므로 사용자 ID를 관리하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-106">When you're building a multitenant application, one of the first challenges is managing user identities, because now every user belongs to a tenant.</span></span> <span data-ttu-id="fbeb8-107">예:</span><span class="sxs-lookup"><span data-stu-id="fbeb8-107">For example:</span></span>

* <span data-ttu-id="fbeb8-108">사용자가 조직 자격 증명으로 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-108">Users sign in with their organizational credentials.</span></span>
* <span data-ttu-id="fbeb8-109">사용자는 조직의 데이터에는 액세스할 수 있지만 다른 테넌트에 속한 데이터에는 액세스하지 못합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-109">Users should have access to their organization's data, but not data that belongs to other tenants.</span></span>
* <span data-ttu-id="fbeb8-110">조직은 응용 프로그램을 등록한 다음 조직의 멤버에게 응용 프로그램 역할을 할당할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-110">An organization can sign up for the application, and then assign application roles to its members.</span></span>

<span data-ttu-id="fbeb8-111">Azure AD(Azure Active Directory)에는 이러한 모든 시나리오를 지원하는 일부 유용한 기능이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-111">Azure Active Directory (Azure AD) has some great features that support all of these scenarios.</span></span>

<span data-ttu-id="fbeb8-112">이 문서 시리즈를 수행하기 위해 완전한 다중 테넌트 응용 프로그램의 [종단 간 구현][sample application]을 만들었습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-112">To accompany this series of articles, we created a complete [end-to-end implementation][sample application] of a multitenant application.</span></span> <span data-ttu-id="fbeb8-113">이 문서에는 응용 프로그램 빌드 프로세스에서 습득한 내용이 반영되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-113">The articles reflect what we learned in the process of building the application.</span></span> <span data-ttu-id="fbeb8-114">응용 프로그램을 시작하려면 [설문 조사 응용 프로그램 실행][running-the-app]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-114">To get started with the application, see [Run the Surveys application][running-the-app].</span></span>

## <a name="introduction"></a><span data-ttu-id="fbeb8-115">소개</span><span class="sxs-lookup"><span data-stu-id="fbeb8-115">Introduction</span></span>

<span data-ttu-id="fbeb8-116">클라우드에서 호스트할 엔터프라이즈 SaaS 응용 프로그램을 작성하고 있다고 가정해 봅니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-116">Let's say you're writing an enterprise SaaS application to be hosted in the cloud.</span></span> <span data-ttu-id="fbeb8-117">물론 응용 프로그램에는 다음 사용자가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-117">Of course, the application will have users:</span></span>

![사용자](./images/users.png)

<span data-ttu-id="fbeb8-119">그러나 이러한 사용자는 다음 조직에 속해 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-119">But those users belong to organizations:</span></span>

![조직 사용자](./images/org-users.png)

<span data-ttu-id="fbeb8-121">예: Tailspin은 해당 SaaS 응용 프로그램에 대한 구독을 판매합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-121">Example: Tailspin sells subscriptions to its SaaS application.</span></span> <span data-ttu-id="fbeb8-122">Contoso와 Fabrikam에서 이 응용 프로그램에 등록합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-122">Contoso and Fabrikam sign up for the app.</span></span> <span data-ttu-id="fbeb8-123">Alice(`alice@contoso`)가 로그인한 경우 응용 프로그램은 Alice가 Contoso에 속해 있음을 알아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-123">When Alice (`alice@contoso`) signs in, the application should know that Alice is part of Contoso.</span></span>

* <span data-ttu-id="fbeb8-124">Alice는 Contoso 데이터에 대한 액세스 권한이 *있어야* 합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-124">Alice *should* have access to Contoso data.</span></span>
* <span data-ttu-id="fbeb8-125">Alice는 Fabrikam 데이터에 대한 액세스 권한이 *없어야* 합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-125">Alice *should not* have access to Fabrikam data.</span></span>

<span data-ttu-id="fbeb8-126">이 지침에서는 [Azure AD(Azure Active Directory)][AzureAD]를 사용하여 로그인 및 인증을 처리하는 방식으로 다중 테넌트 응용 프로그램에서 사용자 ID를 관리하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-126">This guidance will show you how to manage user identities in a multitenant application, using [Azure Active Directory][AzureAD] (Azure AD) to handle sign-in and authentication.</span></span>

## <a name="what-is-multitenancy"></a><span data-ttu-id="fbeb8-127">다중 테넌트 지원이란?</span><span class="sxs-lookup"><span data-stu-id="fbeb8-127">What is multitenancy?</span></span>
<span data-ttu-id="fbeb8-128">*테넌트*는 사용자 그룹입니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-128">A *tenant* is a group of users.</span></span> <span data-ttu-id="fbeb8-129">SaaS 응용 프로그램에서 테넌트는 응용 프로그램의 고객 또는 구독자입니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-129">In a SaaS application, the tenant is a subscriber or customer of the application.</span></span> <span data-ttu-id="fbeb8-130">*다중 테넌트*는 여러 테넌트가 동일한 실제 앱 인스턴스를 공유하는 아키텍처입니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-130">*Multitenancy* is an architecture where multiple tenants share the same physical instance of the app.</span></span> <span data-ttu-id="fbeb8-131">각 테넌트는 실제 리소스(예: VM 또는 저장소)를 공유하지만 응용 프로그램의 고유한 논리 인스턴스를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-131">Although tenants share physical resources (such as VMs or storage), each tenant gets its own logical instance of the app.</span></span>

<span data-ttu-id="fbeb8-132">일반적으로 응용 프로그램 데이터는 다른 테넌트와 공유되는 것이 아니라 테넌트 내의 사용자 간에 공유됩니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-132">Typically, application data is shared among the users within a tenant, but not with other tenants.</span></span>

![다중 테넌트](./images/multitenant.png)

<span data-ttu-id="fbeb8-134">각 테넌트에 전용 실제 인스턴스가 있는 단일 테넌트 아키텍처와 이 아키텍처를 비교해 보세요.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-134">Compare this architecture with a single-tenant architecture, where each tenant has a dedicated physical instance.</span></span> <span data-ttu-id="fbeb8-135">단일 테넌트 아키텍처에서는 응용 프로그램의 새 인스턴스를 스핀업하여 테넌트를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-135">In a single-tenant architecture, you add tenants by spinning up new instances of the app.</span></span>

![단일 테넌트](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a><span data-ttu-id="fbeb8-137">다중 테넌트 지원과 수평 확장</span><span class="sxs-lookup"><span data-stu-id="fbeb8-137">Multitenancy and horizontal scaling</span></span>
<span data-ttu-id="fbeb8-138">클라우드에서 크기를 조정하려면 일반적으로 더 많은 실제 인스턴스를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-138">To achieve scale in the cloud, it’s common to add more physical instances.</span></span> <span data-ttu-id="fbeb8-139">이것을 *수평 확장* 또는 *규모 확장*이라고 합니다. 웹앱을 예로 들어 보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-139">This is known as *horizontal scaling* or *scaling out*. Consider a web app.</span></span> <span data-ttu-id="fbeb8-140">더 많은 트래픽을 처리하기 위해 서버 VM을 추가하여 부하 분산 장치 뒤에 둘 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-140">To handle more traffic, you can add more server VMs and put them behind a load balancer.</span></span> <span data-ttu-id="fbeb8-141">각 VM은 웹앱의 별도 실제 인스턴스를 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-141">Each VM runs a separate physical instance of the web app.</span></span>

![웹 사이트 부하 분산](./images/load-balancing.png)

<span data-ttu-id="fbeb8-143">모든 요청을 모든 인스턴스로 라우팅할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-143">Any request can be routed to any instance.</span></span> <span data-ttu-id="fbeb8-144">시스템은 모두 함께 단일 논리 인스턴스로 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-144">Together, the system functions as a single logical instance.</span></span> <span data-ttu-id="fbeb8-145">사용자에게 영향을 주지 않고 VM을 삭제하거나 새 VM을 스핀업할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-145">You can tear down a VM or spin up a new VM, without affecting users.</span></span> <span data-ttu-id="fbeb8-146">이 아키텍처에서 각 실제 인스턴스는 다중 테넌트이므로 인스턴스를 추가하여 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-146">In this architecture, each physical instance is multi-tenant, and you scale by adding more instances.</span></span> <span data-ttu-id="fbeb8-147">하나의 인스턴스가 중지된 경우 테넌트는 영향을 받지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-147">If one instance goes down, it should not affect any tenant.</span></span>

## <a name="identity-in-a-multitenant-app"></a><span data-ttu-id="fbeb8-148">다중 테넌트 응용 프로그램 식별</span><span class="sxs-lookup"><span data-stu-id="fbeb8-148">Identity in a multitenant app</span></span>
<span data-ttu-id="fbeb8-149">다중 테넌트 응용 프로그램에서는 테넌트의 컨텍스트에서 사용자를 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-149">In a multitenant app, you must consider users in the context of tenants.</span></span>

<span data-ttu-id="fbeb8-150">**인증**</span><span class="sxs-lookup"><span data-stu-id="fbeb8-150">**Authentication**</span></span>

* <span data-ttu-id="fbeb8-151">사용자는 조직 자격 증명으로 응용 프로그램에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-151">Users sign into the app with their organization credentials.</span></span> <span data-ttu-id="fbeb8-152">응용 프로그램의 새 사용자 프로필을 만들 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-152">They don't have to create new user profiles for the app.</span></span>
* <span data-ttu-id="fbeb8-153">동일한 조직 내의 사용자는 동일한 테넌트에 속합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-153">Users within the same organization are part of the same tenant.</span></span>
* <span data-ttu-id="fbeb8-154">사용자가 로그인하면 응용 프로그램에서 사용자가 속한 테넌트를 알고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-154">When a user signs in, the application knows which tenant the user belongs to.</span></span>

<span data-ttu-id="fbeb8-155">**권한 부여**</span><span class="sxs-lookup"><span data-stu-id="fbeb8-155">**Authorization**</span></span>

* <span data-ttu-id="fbeb8-156">사용자 작업(예: 리소스 보기)에 대한 권한을 부여할 때 응용 프로그램에서 사용자의 테넌트를 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-156">When authorizing a user's actions (say, viewing a resource), the app must take into account the user's tenant.</span></span>
* <span data-ttu-id="fbeb8-157">응용 프로그램 내에서 사용자에게 "관리자" 또는 "표준 사용자"와 역할이 할당될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-157">Users might be assigned roles within the application, such as "Admin" or "Standard User".</span></span> <span data-ttu-id="fbeb8-158">역할 할당은 SaaS 공급자가 아니라 고객에 의해 관리됩니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-158">Role assignments should be managed by the customer, not by the SaaS provider.</span></span>

<span data-ttu-id="fbeb8-159">**예제.**</span><span class="sxs-lookup"><span data-stu-id="fbeb8-159">**Example.**</span></span> <span data-ttu-id="fbeb8-160">Contoso의 직원인 Alice는 브라우저에서 응용 프로그램을 탐색하여 “Log in” 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-160">Alice, an employee at Contoso, navigates to the application in her browser and clicks the “Log in” button.</span></span> <span data-ttu-id="fbeb8-161">그러자 회사 자격 증명(사용자 이름 및 암호)을 입력하는 로그인 화면으로 리디렉션됩니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-161">She is redirected to a login screen where she enters her corporate credentials (username and password).</span></span> <span data-ttu-id="fbeb8-162">이때 Alice는 `alice@contoso.com`으로 응용 프로그램에 로그인됩니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-162">At this point, she is logged into the app as `alice@contoso.com`.</span></span> <span data-ttu-id="fbeb8-163">응용 프로그램에서는 Alice가 이 응용 프로그램의 관리자임을 알고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-163">The application also knows that Alice is an admin user for this application.</span></span> <span data-ttu-id="fbeb8-164">Alice는 관리자이므로 Contoso에 속한 모든 리소스 목록을 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-164">Because she is an admin, she can see a list of all the resources that belong to Contoso.</span></span> <span data-ttu-id="fbeb8-165">그러나 자신의 테넌트 내에서만 관리자이기 때문에 Fabrikam의 리소스는 볼 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-165">However, she cannot view Fabrikam's resources, because she is an admin only within her tenant.</span></span>

<span data-ttu-id="fbeb8-166">이 지침에서는 특별히 ID 관리에 Azure AD를 사용하는 방법을 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-166">In this guidance, we'll look specifically at using Azure AD for identity management.</span></span>

* <span data-ttu-id="fbeb8-167">고객이 자신의 사용자 프로필을 Azure AD(Office365 및 Dynamics CRM 테넌트 포함)에 저장한 것으로 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-167">We assume the customer stores their user profiles in Azure AD (including Office365 and Dynamics CRM tenants)</span></span>
* <span data-ttu-id="fbeb8-168">온-프레미스 AD(Active Directory)를 사용하는 고객은 [Azure AD Connect][ADConnect]를 사용하여 온-프레미스 AD를 Azure AD와 동기화할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-168">Customers with on-premise Active Directory (AD) can use [Azure AD Connect][ADConnect] to sync their on-premise AD with Azure AD.</span></span>

<span data-ttu-id="fbeb8-169">온-프레미스 AD를 사용하는 고객이 회사 IT 정책 또는 다른 이유로 Azure AD Connect를 사용할 수 없는 경우 SaaS 공급자는 AD FS(Active Directory Federation Services)를 통해 고객의 AD와 페더레이션할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-169">If a customer with on-premise AD cannot use Azure AD Connect (due to corporate IT policy or other reasons), the SaaS provider can federate with the customer's AD through Active Directory Federation Services (AD FS).</span></span> <span data-ttu-id="fbeb8-170">이 옵션은 [고객의 AD FS와 페더레이션]에 설명되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-170">This option is described in [Federating with a customer's AD FS].</span></span>

<span data-ttu-id="fbeb8-171">이 지침에서는 데이터 분할, 테넌트별 구성 등 다중 테넌트 지원의 다른 측면을 고려하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="fbeb8-171">This guidance does not consider other aspects of multitenancy such as data partitioning, per-tenant configuration, and so forth.</span></span>

<span data-ttu-id="fbeb8-172">[**다음**][tailpin]</span><span class="sxs-lookup"><span data-stu-id="fbeb8-172">[**Next**][tailpin]</span></span>



<!-- Links -->
[ADConnect]: /azure/active-directory/active-directory-aadconnect
[AzureAD]: /azure/active-directory

[고객의 AD FS와 페더레이션]: adfs.md
[Federating with a customer's AD FS]: adfs.md
[tailpin]: tailspin.md

[running-the-app]: ./run-the-app.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
