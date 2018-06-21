---
title: Tailspin 설문 조사 응용 프로그램 정보
description: Tailspin 설문 조사 응용 프로그램 개요
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: index
pnp.series.next: authenticate
ms.openlocfilehash: 028f7940d2e3cd7e8e629554f8af290ec5fdd184
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
ms.locfileid: "26582685"
---
# <a name="the-tailspin-scenario"></a><span data-ttu-id="b52d3-103">Tailspin 시나리오</span><span class="sxs-lookup"><span data-stu-id="b52d3-103">The Tailspin scenario</span></span>

<span data-ttu-id="b52d3-104">[![GitHub](../_images/github.png) 샘플 코드][sample application]</span><span class="sxs-lookup"><span data-stu-id="b52d3-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="b52d3-105">Tailspin은 설문 조사라는 SaaS 응용 프로그램을 개발하는 가상 회사입니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-105">Tailspin is a fictitious company that is developing a SaaS application named Surveys.</span></span> <span data-ttu-id="b52d3-106">이 응용 프로그램을 사용하면 조직에서 온라인 설문 조사를 만들어 게시할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-106">This application enables organizations to create and publish online surveys.</span></span>

* <span data-ttu-id="b52d3-107">조직은 응용 프로그램을 등록할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-107">An organization can sign up for the application.</span></span>
* <span data-ttu-id="b52d3-108">조직이 등록한 후 사용자는 조직 자격 증명으로 응용 프로그램에 로그인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-108">After the organization is signed up, users can sign into the application with their organizational credentials.</span></span>
* <span data-ttu-id="b52d3-109">사용자는 설문 조사를 만들고, 편집하고, 게시할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-109">Users can create, edit, and publish surveys.</span></span>

> [!NOTE]
> <span data-ttu-id="b52d3-110">응용 프로그램을 시작하려면 [설문 조사 응용 프로그램 실행]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b52d3-110">To get started with the application, see [Run the Surveys application].</span></span>
> 
> 

## <a name="users-can-create-edit-and-view-surveys"></a><span data-ttu-id="b52d3-111">사용자는 설문 조사를 만들고, 편집하고, 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-111">Users can create, edit, and view surveys</span></span>
<span data-ttu-id="b52d3-112">인증된 사용자는 자신이 만들었거나 참가자 권한이 있는 모든 설문 조사를 볼 수 있고, 새 설문 조사를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-112">An authenticated user can view all the surveys that he or she has created or has contributor rights to, and create new surveys.</span></span> <span data-ttu-id="b52d3-113">사용자가 조직 ID, `bob@contoso.com`을 사용하여 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-113">Notice that the user is signed in with his organizational identity, `bob@contoso.com`.</span></span>

![설문 조사 앱](./images/surveys-screenshot.png)

<span data-ttu-id="b52d3-115">이 스크린샷은 설문 조사 편집 페이지를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-115">This screenshot shows the Edit Survey page:</span></span>

![설문 조사 편집](./images/edit-survey.png)

<span data-ttu-id="b52d3-117">또한 사용자는 동일한 테넌트 내에서 다른 사용자가 만든 설문 조사를 볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-117">Users can also view any surveys created by other users within the same tenant.</span></span>

![테넌트 설문 조사](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a><span data-ttu-id="b52d3-119">설문 조사 소유자는 참가자를 초대할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-119">Survey owners can invite contributors</span></span>
<span data-ttu-id="b52d3-120">사용자는 설문 조사를 만들 때 다른 사람을 설문 조사의 참가자로 초대할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-120">When a user creates a survey, he or she can invite other people to be contributors on the survey.</span></span> <span data-ttu-id="b52d3-121">참가자는 설문 조사를 편집할 수 있으나 삭제하거나 게시할 수는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-121">Contributors can edit the survey, but cannot delete or publish it.</span></span>  

![참여자 추가](./images/add-contributor.png)

<span data-ttu-id="b52d3-123">사용자는 다른 테넌트에서 참가자를 추가하여 테넌트 간에 리소스를 공유할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-123">A user can add contributors from other tenants, which enables cross-tenant sharing of resources.</span></span> <span data-ttu-id="b52d3-124">이 스크린샷에서 Bob(`bob@contoso.com`)은 자신이 만든 설문 조사에 참가자로 Alice(`alice@fabrikam.com`)를 추가하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-124">In this screenshot, Bob (`bob@contoso.com`) is adding Alice (`alice@fabrikam.com`) as a contributor to a survey that Bob created.</span></span>

<span data-ttu-id="b52d3-125">Alice가 로그인할 때 "Surveys I can contribute to" 아래에 나열된 설문 조사를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-125">When Alice logs in, she sees the survey listed under "Surveys I can contribute to".</span></span>

![설문 조사 참가자](./images/contributor.png)

<span data-ttu-id="b52d3-127">Alice는 Contoso 테넌트의 게스트가 아니라 자신의 테넌트로 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-127">Note that Alice signs into her own tenant, not as a guest of the Contoso tenant.</span></span> <span data-ttu-id="b52d3-128">Alice는 해당 설문 조사에 대해서만 참가자 권한을 가지며 Contoso 테넌트의 다른 설문 조사는 볼 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-128">Alice has contributor permissions only for that survey &mdash; she cannot view other surveys from the Contoso tenant.</span></span>

## <a name="architecture"></a><span data-ttu-id="b52d3-129">건축</span><span class="sxs-lookup"><span data-stu-id="b52d3-129">Architecture</span></span>
<span data-ttu-id="b52d3-130">설문 조사 응용 프로그램은 웹 프런트 엔드 및 Web API 백 엔드로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-130">The Surveys application consists of a web front end and a web API backend.</span></span> <span data-ttu-id="b52d3-131">둘 다 [ASP.NET Core]를 사용하여 구현됩니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-131">Both are implemented using [ASP.NET Core].</span></span>

<span data-ttu-id="b52d3-132">웹 응용 프로그램은 Azure AD(Azure Active Directory)를 사용하여 사용자를 인증합니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-132">The web application uses Azure Active Directory (Azure AD) to authenticate users.</span></span> <span data-ttu-id="b52d3-133">또한 웹 응용 프로그램은 Azure AD를 호출하여 Web API에 대한 OAuth 2 액세스 토큰을 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-133">The web application also calls Azure AD to get OAuth 2 access tokens for the Web API.</span></span> <span data-ttu-id="b52d3-134">액세스 토큰은 Azure Redis Cache에 캐시됩니다.</span><span class="sxs-lookup"><span data-stu-id="b52d3-134">Access tokens are cached in Azure Redis Cache.</span></span> <span data-ttu-id="b52d3-135">캐시는 여러 인스턴스가 동일한 토큰 캐시를 공유할 수 있도록 해줍니다(예: 서버 팜에서).</span><span class="sxs-lookup"><span data-stu-id="b52d3-135">The cache enables multiple instances to share the same token cache (e.g., in a server farm).</span></span>

![건축](./images/architecture.png)

<span data-ttu-id="b52d3-137">[**다음**][authentication]</span><span class="sxs-lookup"><span data-stu-id="b52d3-137">[**Next**][authentication]</span></span>

<!-- Links -->

[authentication]: authenticate.md

[설문 조사 응용 프로그램 실행]: ./run-the-app.md
[Run the Surveys application]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
