---
title: Azure에서 리소스 액세스 관리 이해
description: 'Azure에서 리소스 액세스 관리 구문 설명: Azure Resource Manager, 구독, 리소스 그룹 및 리소스'
author: petertay
ms.openlocfilehash: 02e25e943822ba065d360d687d34283d86a805bd
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229582"
---
# <a name="understanding-resource-access-management-in-azure"></a><span data-ttu-id="c21c1-103">Azure에서 리소스 액세스 관리 이해</span><span class="sxs-lookup"><span data-stu-id="c21c1-103">Understanding resource access management in Azure</span></span>

<span data-ttu-id="c21c1-104">[리소스 거버넌스란?](governance-explainer.md)에서 거버넌스가 조직의 요구 사항 및 목표 달성을 위해 Azure 리소스의 사용을 관리, 모니터링 및 감사하는 지속적인 프로세스를 가리킨다는 점을 알아보았습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-104">In [what is resource governance?](governance-explainer.md), you learned that governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of your organization.</span></span> <span data-ttu-id="c21c1-105">거버넌스 모델을 디자인하는 방법을 알아보기 위해 이동하려면 먼저 Azure에서 리소스 액세스 관리 제어를 이해해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-105">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="c21c1-106">이러한 리소스 액세스 관리 제어의 구성은 거버넌스 모델의 기본을 형성합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-106">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="c21c1-107">Azure에 리소스를 배포하는 방법에 대해 좀 더 자세히 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-107">Let's begin by taking a closer look at how are resources are deployed in Azure.</span></span> 

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="c21c1-108">Azure 리소스란?</span><span class="sxs-lookup"><span data-stu-id="c21c1-108">What is an Azure resource?</span></span>

<span data-ttu-id="c21c1-109">Azure에서 **리소스**라는 용어는 Azure에서 관리하는 엔터티를 가리킵니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-109">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="c21c1-110">예를 들어 가상 머신, 가상 네트워크 및 저장소 계정은 모두 Azure 리소스를 가리킵니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-110">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

![](../_images/governance-1-9.png)   
<span data-ttu-id="c21c1-111">*그림 1. 리소스*</span><span class="sxs-lookup"><span data-stu-id="c21c1-111">*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="c21c1-112">Azure 리소스 그룹이란?</span><span class="sxs-lookup"><span data-stu-id="c21c1-112">What is an Azure resource group?</span></span>

<span data-ttu-id="c21c1-113">Azure의 각 리소스는 [리소스 그룹](/azure/azure-resource-manager/resource-group-overview#resource-groups)에 속해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-113">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="c21c1-114">리소스 그룹은 단순히 단일 엔터티로 관리될 수 있도록 여러 리소스를 그룹화하는 논리적 구문입니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-114">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="c21c1-115">예를 들어 [n 계층 응용 프로그램](/azure/architecture/guide/architecture-styles/n-tier)에 대한 리소스와 같은 비슷한 수명 주기를 공유하는 리소스를 그룹으로 만들거나 삭제할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-115">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span> 

![](../_images/governance-1-10.png)   
<span data-ttu-id="c21c1-116">*그림 2. 리소스 그룹에는 리소스가 포함됩니다.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-116">*Figure 2. A resource group contains a resource.*</span></span> 

<span data-ttu-id="c21c1-117">리소스 그룹 및 여기 포함된 리소스는 Azure **구독**과 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-117">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span> 

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="c21c1-118">Azure 구독이란?</span><span class="sxs-lookup"><span data-stu-id="c21c1-118">What is an Azure subscription?</span></span>

<span data-ttu-id="c21c1-119">Azure 구독은 리소스 그룹 및 해당 리소스를 함께 그룹화하는 논리적 구문에서 리소스 그룹과 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-119">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="c21c1-120">그러나 Azure 구독은 Azure Resource Manager에서 사용한 컨트롤과도 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-120">However, an Azure subscription is also associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="c21c1-121">무슨 의미인가요?</span><span class="sxs-lookup"><span data-stu-id="c21c1-121">What does this mean?</span></span> <span data-ttu-id="c21c1-122">이와 Azure 구독 간의 관계에 대해 자세히 알아보기 위해 Azure Resource Manager에 대해 더 자세히 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-122">Let's take a closer look at Azure resource manager to learn about the relationship between it and an Azure subscription.</span></span>

![](../_images/governance-1-11.png)   
<span data-ttu-id="c21c1-123">*그림 3. Azure 구독.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-123">*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="c21c1-124">Azure Resource Manager란?</span><span class="sxs-lookup"><span data-stu-id="c21c1-124">What is Azure resource manager?</span></span>

<span data-ttu-id="c21c1-125">[Azure의 작동 방식](azure-explainer.md)에서 Azure에는 Azure의 모든 기능을 오케스트레이션하는 여러 서비스를 사용하는 "프런트 엔드"가 포함된다는 사실을 알아보았습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-125">In [how does Azure work?](azure-explainer.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="c21c1-126">[Azure Resource Manager](/azure/azure-resource-manager/)는 이러한 서비스 중 하나이며 이 서비스는 리소스를 관리하기 위해 클라이언트에서 사용하는 RESTful API를 호스트합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-126">One of these services is [Azure resource manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span> 

![](../_images/governance-1-12.png)   
<span data-ttu-id="c21c1-127">*그림 4. Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-127">*Figure 4. Azure resource manager.*</span></span>

<span data-ttu-id="c21c1-128">다음 그림은 [PowerShell](/powershell/azure/overview), [Azure Portal](https://portal.azure.com) 및 [Azure CLI(명령줄 인터페이스)](/cli/azure)라는 세 가지 클라이언트를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-128">The following figure shows three clients: [Powershell](/powershell/azure/overview),[the Azure portal](https://portal.azure.com), and the [Azure command line interface (CLI)](/cli/azure):</span></span>

![](../_images/governance-1-13.png)   
<span data-ttu-id="c21c1-129">*그림 5. Azure Resource Manager RESTful API에 대한 Azure 클라이언트 연결.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-129">*Figure 5. Azure clients connect to the Azure resource manager RESTful API.*</span></span>

<span data-ttu-id="c21c1-130">이러한 클라이언트가 RESTful API를 사용하여 Azure Resource Manager에 연결하는 반면 Azure Resource Manager에는 리소스를 직접 관리하는 기능이 포함되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-130">While these clients connect to Azure resource manager using the RESTful API, Azure resource manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="c21c1-131">대신 Azure에서 대부분의 리소스 종류에는 고유한 [**리소스 공급자**](/azure/azure-resource-manager/resource-group-overview#terminology)가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-131">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span> 

![](../_images/governance-1-14.png)   
<span data-ttu-id="c21c1-132">*그림 6. Azure 리소스 공급자.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-132">*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="c21c1-133">클라이언트가 특정 리소스를 관리하도록 요청하면 Azure Resource Manager는 요청을 완료하기 위해 해당 리소스 종류에 대한 리소스 공급자에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-133">When a client makes a request to manage a specific resource, Azure resource manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="c21c1-134">예를 들어 클라이언트가 가상 머신 리소스를 관리하도록 요청하는 경우 Azure Resource Manager는 **Microsoft.compute** 리소스 공급자에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-134">For example, if a client makes a request to manage a virtual machine resource, Azure resource manager connects to the **Microsoft.compute** resource provider.</span></span> 

![](../_images/governance-1-15.png)   
<span data-ttu-id="c21c1-135">*그림 7. Azure Resource Manager는 **Microsoft.compute** 리소스 공급자에 연결하여 클라이언트 요청에 지정된 리소스를 관리합니다.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-135">*Figure 7. Azure resource manager connects to the **Microsoft.compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="c21c1-136">해당 Azure Resource Manager는 클라이언트가 가상 머신 리소스를 관리하기 위해 구독 및 리소스 그룹 모두에 대한 식별자를 지정하도록 요구합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-136">Notice that Azure resource manager required the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span> 

<span data-ttu-id="c21c1-137">이제 Azure Resource Manager 작동 방식을 이해했으므로 Azure Resource Manager에서 사용하는 컨트롤과 Azure 구독을 연결하는 방법을 다시 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-137">Now that you have an understanding of how Azure resource manager works, let's return to our discussion of how an Azure subscription is associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="c21c1-138">Azure Resource Manager가 리소스 관리 요청을 실행하기 전에 컨트롤의 집합을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-138">Before any resource management request can be executed by Azure resource manager, a set of controls are checked.</span></span> 

<span data-ttu-id="c21c1-139">첫 번째 컨트롤은 유효성이 검사된 사용자가 요청해야 하고 Azure Resource Manager에게는 사용자 ID 기능을 제공하기 위해 [Azure AD(Azure Active Directory)](/azure/active-directory/)를 사용하는 신뢰할 수 있는 관계가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-139">The first control is that a request must be made by a validated user, and Azure resource manager has a trusted relationship with [Azure Active Directory (Azure AD)](/azure/active-directory/) to provide user identity functionality.</span></span>

![](../_images/governance-1-16.png)   
<span data-ttu-id="c21c1-140">*그림 8. Azure Active Directory.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-140">*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="c21c1-141">Azure AD에서 사용자는 **테넌트**로 조각화됩니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-141">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="c21c1-142">테넌트는 일반적으로 조직과 연결된 Azure AD의 안전한 전용 인스턴스를 나타내는 논리적 구문입니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-142">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="c21c1-143">각 구독은 Azure AD 테넌트와 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-143">Each subscription is associated with an Azure AD tenant.</span></span>

![](../_images/governance-1-17.png)   
<span data-ttu-id="c21c1-144">*그림 9. 구독과 연결된 Azure AD 테넌트.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-144">*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="c21c1-145">특정 구독에서 리소스를 관리하는 각 클라이언트 요청에는 연결된 Azure AD 테넌트에 사용자가 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-145">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span> 

<span data-ttu-id="c21c1-146">다음 컨트롤은 사용자에게 요청할 충분한 권한이 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-146">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="c21c1-147">[RBAC(역할 기반 액세스 제어)](/azure/role-based-access-control/)를 사용하여 사용자에 대한 보안 권한을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-147">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

![](../_images/governance-1-18.png)   
<span data-ttu-id="c21c1-148">*그림 10. 테넌트의 각 사용자에게는 하나 이상의 RBAC 역할이 할당됩니다.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-148">*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="c21c1-149">RBAC 역할은 사용자가 특정 리소스에 대해 수행할 수 있는 일련의 사용 권한을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-149">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="c21c1-150">역할을 사용자에게 할당하면 해당 사용 권한이 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-150">When the role is assigned to user, those permissions are applied.</span></span> <span data-ttu-id="c21c1-151">예를 들어 [기본 제공 **소유자** 역할](/azure/role-based-access-control/built-in-roles#owner)을 사용하면 사용자가 리소스에서 모든 작업을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-151">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="c21c1-152">다음 컨트롤은 [Azure 리소스 정책](/azure/azure-policy/)에 지정된 설정에서 요청이 허용되는지를 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-152">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/azure-policy/).</span></span> <span data-ttu-id="c21c1-153">Azure 리소스 정책은 특정 리소스에 허용되는 작업을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-153">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="c21c1-154">예를 들어 Azure 리소스 정책은 사용자가 특정 형식의 가상 머신만을 배포할 수 있도록 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-154">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

![](../_images/governance-1-19.png)   
<span data-ttu-id="c21c1-155">*그림 11. Azure 리소스 정책.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-155">*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="c21c1-156">다음 컨트롤은 요청이 [Azure 구독 제한](/azure/azure-subscription-service-limits)을 초과하지 않는지 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-156">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="c21c1-157">예를 들어 각 구독에는 구독당 980개의 리소스 그룹이라는 제한이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-157">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="c21c1-158">제한에 도달하면 추가 리소스 그룹을 배포하는 요청이 수신된 경우 해당 요청이 거부됩니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-158">If a request is received to deploy an additional resource groups once the limit has been reached, it is denied.</span></span>

![](../_images/governance-1-20.png)   
<span data-ttu-id="c21c1-159">*그림 12. Azure 리소스 제한.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-159">*Figure 12. Azure resource limits.*</span></span> 

<span data-ttu-id="c21c1-160">최종 컨트롤은 요청이 구독과 연결된 재무 약정 내에 있는지를 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-160">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="c21c1-161">예를 들어 요청이 가상 머신을 배포하는 것인 경우 Azure Resource Manager는 구독에 충분한 결제 정보가 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-161">For example, if the request is to deploy a virtual machine, Azure resource manager verifies that the subscription has sufficient payment information.</span></span>

![](../_images/governance-1-21.png)   
<span data-ttu-id="c21c1-162">*그림 13. 재무 약정이 구독과 연결됩니다.*</span><span class="sxs-lookup"><span data-stu-id="c21c1-162">*Figure 13. A financial commitment is associated with a subscription.*</span></span>

# <a name="summary"></a><span data-ttu-id="c21c1-163">요약</span><span class="sxs-lookup"><span data-stu-id="c21c1-163">Summary</span></span>

<span data-ttu-id="c21c1-164">이 아티클에서는 Azure Resource Manager를 사용하여 Azure에서 리소스 액세스를 관리하는 방법에 대해 알아보았습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-164">In this article, you learned about how resource access is managed in Azure using Azure resource manager.</span></span>

# <a name="next-steps"></a><span data-ttu-id="c21c1-165">다음 단계</span><span class="sxs-lookup"><span data-stu-id="c21c1-165">Next steps</span></span>

<span data-ttu-id="c21c1-166">이제 Azure에서 리소스 액세스를 관리하는 방법을 이해했으므로 이러한 서비스를 사용하여 [거버넌스 모델을 디자인](governance-how-to.md)하는 방법을 알아보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="c21c1-166">Now that you understand how resource access is managed in Azure, move on to learn how to [design a governance model](governance-how-to.md) using these services.</span></span>