---
title: "AWS 전문가를 위한 Azure"
description: "Microsoft Azure 계정, 플랫폼 및 서비스의 기본 사항에 대해 알아봅니다. 또한 AWS 및 Azure 플랫폼 간의 주요 유사점과 차이점에 대해 알아봅니다. Azure에서 AWS 환경을 활용합니다."
keywords: "AWS 전문가, Azure 비교, AWS 비교, azure와 aws의 차이점, azure 및 aws"
author: lbrader
ms.date: 03/24/2017
pnp.series.title: Azure for AWS Professionals
ms.openlocfilehash: e5f7cb5062b0b4a8526f3b29a9fa4ddaff399fc0
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/30/2018
---
# <a name="azure-for-aws-professionals"></a><span data-ttu-id="14b96-106">AWS 전문가를 위한 Azure</span><span class="sxs-lookup"><span data-stu-id="14b96-106">Azure for AWS Professionals</span></span>

<span data-ttu-id="14b96-107">이 문서는 AWS(Amazon Web Services) 전문가가 Microsoft Azure 계정, 플랫폼 및 서비스의 기본 사항을 이해하는 데 도움을 줍니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-107">This article helps Amazon Web Services (AWS) experts understand the basics of Microsoft Azure accounts, platform, and services.</span></span> <span data-ttu-id="14b96-108">또한 AWS 및 Azure 플랫폼 간의 주요 유사점과 차이점에 대해 다룹니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-108">It also covers key similarities and differences between the AWS and Azure platforms.</span></span>

<span data-ttu-id="14b96-109">다음 내용을 배웁니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-109">You'll learn:</span></span>

* <span data-ttu-id="14b96-110">Azure에서 계정 및 리소스가 구성되는 방식.</span><span class="sxs-lookup"><span data-stu-id="14b96-110">How accounts and resources are organized in Azure.</span></span>
* <span data-ttu-id="14b96-111">Azure에서 사용 가능한 솔루션이 구성되는 방식.</span><span class="sxs-lookup"><span data-stu-id="14b96-111">How available solutions are structured in Azure.</span></span>
* <span data-ttu-id="14b96-112">주요 Azure 서비스가 AWS 서비스와 다른 점.</span><span class="sxs-lookup"><span data-stu-id="14b96-112">How the major Azure services differ from AWS services.</span></span>

<span data-ttu-id="14b96-113">Azure와 AWS는 시간에 관계없이 각 기능을 독립적으로 개발했기 때문에 각 기능의 구현 및 디자인이 큰 차이를 보입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-113">Azure and AWS built their capabilities independently over time so that each has important implementation and design differences.</span></span>

## <a name="overview"></a><span data-ttu-id="14b96-114">개요</span><span class="sxs-lookup"><span data-stu-id="14b96-114">Overview</span></span>

<span data-ttu-id="14b96-115">AWS와 마찬가지로, Microsoft Azure는 핵심 계산, 저장소, 데이터베이스 및 네트워킹 서비스 집합을 중심으로 빌드됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-115">Like AWS, Microsoft Azure is built around a core set of compute, storage, database, and networking services.</span></span> <span data-ttu-id="14b96-116">두 플랫폼이 제공하는 제품과 서비스의 기본적인 사항은 동일한 경우가 많습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-116">In many cases, both platforms offer a basic equivalence between the products and services they offer.</span></span> <span data-ttu-id="14b96-117">AWS와 Azure 모두 Windows 또는 Linux 호스트를 기반으로 고가용성 솔루션을 빌드할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-117">Both AWS and Azure allow you to build highly available solutions based on Windows or Linux hosts.</span></span> <span data-ttu-id="14b96-118">따라서 Linux 및 OSS 기술을 사용하여 개발하는 데 익숙한 분들은 어느 플랫폼으로도 작업을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-118">So, if you're used to development using Linux and OSS technology, both platforms can do the job.</span></span>

<span data-ttu-id="14b96-119">두 플랫폼의 기능은 유사하지만, 기능을 제공하는 리소스는 종종 다르게 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-119">While the capabilities of both platforms are similar, the resources that provide those capabilities are often organized differently.</span></span> <span data-ttu-id="14b96-120">솔루션을 빌드하는 데 필요한 서비스 간에 항상 정확한 일대일 관계가 성립하는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-120">Exact one-to-one relationships between the services required to build a solution are not always clear.</span></span> <span data-ttu-id="14b96-121">특정 서비스가 한 플랫폼에만 제공되고 다른 플랫폼에는 제공되지 않는 경우가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-121">There are also cases where a particular service might be offered on one platform, but not the other.</span></span> <span data-ttu-id="14b96-122">[Azure 및 AWS 서비스 비교 차트](services.md)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="14b96-122">See [charts of comparable Azure and AWS services](services.md).</span></span>

## <a name="accounts-and-subscriptions"></a><span data-ttu-id="14b96-123">계정 및 구독</span><span class="sxs-lookup"><span data-stu-id="14b96-123">Accounts and subscriptions</span></span>

<span data-ttu-id="14b96-124">조직의 규모와 요구 사항에 따라 여러 가격 책정 옵션을 사용하여 Azure 서비스를 구입할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-124">Azure services can be purchased using several pricing options, depending on your organization's size and needs.</span></span> <span data-ttu-id="14b96-125">자세한 내용은 [가격 책정 개요](https://azure.microsoft.com/pricing/) 페이지를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="14b96-125">See the [pricing overview](https://azure.microsoft.com/pricing/) page for details.</span></span>

<span data-ttu-id="14b96-126">[Azure 구독](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/)은 대금 청구 및 권한 관리 책임이 있는 소유자가 할당된 리소스 그룹입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-126">[Azure subscriptions](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/) are a grouping of resources with an assigned owner responsible for billing and permissions management.</span></span> <span data-ttu-id="14b96-127">AWS 계정 하에서 생성되는 모든 리소스가 해당 계정에 연결되는 AWS와는 달리, 구독이 소유자 계정에 독립적으로 존재하며 필요에 따라 새로운 소유자에게 할당할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-127">Unlike AWS, where any resources created under the AWS account are tied to that account, subscriptions exist independently of their owner accounts, and can be reassigned to new owners as needed.</span></span>

<span data-ttu-id="14b96-128">![AWS 계정과 Azure 구독의 구조 및 소유권 비교](./images/azure-aws-account-compare.png "AWS 계정과 Azure 구독의 구조 및 소유권 비교")</span><span class="sxs-lookup"><span data-stu-id="14b96-128">![Comparison of structure and ownership of AWS accounts and Azure subscriptions](./images/azure-aws-account-compare.png "Comparison of structure and ownership of AWS accounts and Azure subscriptions")</span></span>
<br/><span data-ttu-id="14b96-129">*AWS 계정과 Azure 구독의 구조 및 소유권 비교*</span><span class="sxs-lookup"><span data-stu-id="14b96-129">*Comparison of structure and ownership of AWS accounts and Azure subscriptions*</span></span>
<br/><br/>

<span data-ttu-id="14b96-130">구독에는 세 가지 유형의 관리자 계정이 할당됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-130">Subscriptions are assigned three types of administrator accounts:</span></span>

-   <span data-ttu-id="14b96-131">**계정 관리자** - 구독에 사용된 리소스 요금이 청구되는 구독 소유자 및 계정입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-131">**Account Administrator** - The subscription owner and the account billed for the resources used in the subscription.</span></span> <span data-ttu-id="14b96-132">구독 소유권을 양도하여 계정 관리자를 변경하는 것만 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-132">The account administrator can only be changed by transferring ownership of the subscription.</span></span>

-   <span data-ttu-id="14b96-133">**서비스 관리자** - 이 계정은 구독에서 리소스를 만들고 관리할 수 있는 권한을 갖고 있지만 대금 청구를 처리하지는 않습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-133">**Service Administrator** - This account has rights to create and manage resources in the subscription, but is not responsible for billing.</span></span> <span data-ttu-id="14b96-134">기본적으로 계정 관리자 및 서비스 관리자는 동일한 계정에 할당됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-134">By default, the account administrator and service administrator are assigned to the same account.</span></span> <span data-ttu-id="14b96-135">계정 관리자는 구독의 기술 및 운영적 측면을 관리할 별도의 사용자를 서비스 관리자 계정에 할당할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-135">The account administrator can assign a separate user to the service administrator account for managing the technical and operational aspects of a subscription.</span></span> <span data-ttu-id="14b96-136">구독당 서비스 관리자는 한 명만 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-136">There is only one service administrator per subscription.</span></span>

-   <span data-ttu-id="14b96-137">**공동 관리자** - 한 구독에 공동 관리자 계정을 여러 개 할당할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-137">**Co-administrator** - There can be multiple co-administrator accounts assigned to a subscription.</span></span> <span data-ttu-id="14b96-138">공동 관리자는 서비스 관리자를 변경할 수 없지만 구독 리소스와 사용자를 완전하게 제어할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-138">Co-administrators cannot change the service administrator, but otherwise have full control over subscription resources and users.</span></span>

<span data-ttu-id="14b96-139">AWS에서 IAM 사용자 및 그룹에 권한을 부여하는 방법과 비슷하게, 구독 수준 아래에서 사용자 역할 및 개별 권한을 특정 리소스에 할당할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-139">Below the subscription level user roles and individual permissions can also be assigned to specific resources, similarly to how permissions are granted to IAM users and groups in AWS.</span></span> <span data-ttu-id="14b96-140">Azure에서는 모든 사용자 계정이 Microsoft 계정 또는 조직 계정(Azure Active Directory를 통해 관리되는 계정)과 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-140">In Azure all user accounts are associated with either a Microsoft Account or Organizational Account (an account managed through an Azure Active Directory).</span></span>

<span data-ttu-id="14b96-141">AWS 계정과 마찬가지로, 구독의 기본 서비스 할당량 및 제한이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-141">Like AWS accounts, subscriptions have default service quotas and limits.</span></span> <span data-ttu-id="14b96-142">전체 제한 목록은 [Azure 구독 및 서비스 제한, 할당량 및 제약 조건](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="14b96-142">For a full list of these limits, see [Azure subscription and service limits, quotas, and constraints](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/).</span></span>
<span data-ttu-id="14b96-143">[관리 포털에서 지원 요청을 제출](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/)하여 이러한 제한을 최대값까지 높일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-143">These limits can be increased up to the maximum by [filing a support request in the management portal](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/).</span></span>

### <a name="see-also"></a><span data-ttu-id="14b96-144">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-144">See also</span></span>

-   [<span data-ttu-id="14b96-145">Azure 관리자 역할을 추가 또는 변경하는 방법</span><span class="sxs-lookup"><span data-stu-id="14b96-145">How to add or change Azure administrator roles</span></span>](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [<span data-ttu-id="14b96-146">Azure 청구 송장 및 일간 사용 현황 데이터를 다운로드하는 방법</span><span class="sxs-lookup"><span data-stu-id="14b96-146">How to download your Azure billing invoice and daily usage data</span></span>](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## <a name="resource-management"></a><span data-ttu-id="14b96-147">리소스 관리</span><span class="sxs-lookup"><span data-stu-id="14b96-147">Resource management</span></span>

<span data-ttu-id="14b96-148">Azure에서 말하는 "리소스"라는 용어는 AWS와 똑같은 의미로 사용됩니다. 즉, 모든 계산 인스턴스, 저장소 개체, 네트워킹 장치 또는 플랫폼 내에서 만들거나 구성할 수 있는 기타 엔터티를 의미합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-148">The term "resource" in Azure is used in the same way as in AWS, meaning any compute instance, storage object, networking device, or other entity you can create or configure within the platform.</span></span>

<span data-ttu-id="14b96-149">Azure 리소스는 [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) 또는 기존 Azure [클래식 배포 모델](/azure/azure-resource-manager/resource-manager-deployment-model)을 사용하여 배포 및 관리됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-149">Azure resources are deployed and managed using one of two models: [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview), or the older Azure [classic deployment model](/azure/azure-resource-manager/resource-manager-deployment-model).</span></span>
<span data-ttu-id="14b96-150">모든 새 리소스는 Resource Manager 모델을 사용하여 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-150">Any new resources are created using the Resource Manager model.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="14b96-151">리소스 그룹</span><span class="sxs-lookup"><span data-stu-id="14b96-151">Resource groups</span></span>

<span data-ttu-id="14b96-152">Azure와 AWS 둘 다 VM, 저장소, 가상 네트워킹 장치 등의 리소스를 구성하는 "리소스 그룹"이라는 엔터티를 갖고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-152">Both Azure and AWS have entities called "resource groups" that organize resources such as VMs, storage, and virtual networking devices.</span></span> <span data-ttu-id="14b96-153">그러나 [Azure 리소스 그룹](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)을 AWS 리소스 그룹과 직접 비교하는 것은 무리입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-153">However, [Azure resource groups](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/) are not directly comparable to AWS resource groups.</span></span>

<span data-ttu-id="14b96-154">AWS는 한 리소스를 여러 리소스 그룹에 태깅할 수 있지만 Azure 리소스는 항상 한 리소스 그룹에만 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-154">While AWS allows a resource to be tagged into multiple resource groups, an Azure resource is always associated with one resource group.</span></span> <span data-ttu-id="14b96-155">한 리소스 그룹에서 생성된 리소스를 다른 그룹으로 이동할 수 있지만, 한 번에 한 리소스 그룹에만 소속될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-155">A resource created in one resource group can be moved to another group, but can only be in one resource group at a time.</span></span> <span data-ttu-id="14b96-156">리소스 그룹은 Azure Resource Manager가 사용하는 기본적인 그룹화 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-156">Resource groups are the fundamental grouping used by Azure Resource Manager.</span></span>

<span data-ttu-id="14b96-157">[태그](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/)를 사용하여 리소스를 구성할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-157">Resources can also be organized using [tags](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/).</span></span>
<span data-ttu-id="14b96-158">태그는 리소스 그룹 멤버 자격에 관계없이 구독의 리소스를 그룹화할 수 있는 키-값 쌍입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-158">Tags are key-value pairs that allow you to group resources across your subscription irrespective of resource group membership.</span></span>

### <a name="management-interfaces"></a><span data-ttu-id="14b96-159">관리 인터페이스</span><span class="sxs-lookup"><span data-stu-id="14b96-159">Management interfaces</span></span>

<span data-ttu-id="14b96-160">Azure는 리소스를 관리하는 여러 방법을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-160">Azure offers several ways to manage your resources:</span></span>

-   <span data-ttu-id="14b96-161">[웹 인터페이스](https://azure.microsoft.com/documentation/articles/resource-group-portal/).</span><span class="sxs-lookup"><span data-stu-id="14b96-161">[Web interface](https://azure.microsoft.com/documentation/articles/resource-group-portal/).</span></span>
    <span data-ttu-id="14b96-162">AWS 대시보드와 마찬가지로, Azure Portal은 Azure 리소스에 대한 완전한 웹 기반 관리 인터페이스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-162">Like the AWS Dashboard, the Azure portal provides a full web-based management interface for Azure resources.</span></span>

-   <span data-ttu-id="14b96-163">[REST API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/).</span><span class="sxs-lookup"><span data-stu-id="14b96-163">[REST API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/).</span></span>
    <span data-ttu-id="14b96-164">Azure Resource Manager REST API는 Azure Portal에서 사용 가능한 대부분의 기능에 대해 프로그래밍 방식의 액세스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-164">The Azure Resource Manager REST API provides programmatic access to most of the features available in the Azure portal.</span></span>

-   <span data-ttu-id="14b96-165">[명령줄](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/).</span><span class="sxs-lookup"><span data-stu-id="14b96-165">[Command Line](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/).</span></span>
    <span data-ttu-id="14b96-166">Azure CLI 2.0 도구는 Azure 리소스를 만들고 관리할 수 있는 명령줄 인터페이스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-166">The Azure CLI 2.0 tool provides a command-line interface capable of creating and managing Azure resources.</span></span> <span data-ttu-id="14b96-167">Azure CLI는 [Windows, Linux 및 Mac OS](https://aka.ms/azurecli2)에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-167">Azure CLI is available for [Windows, Linux, and Mac OS](https://aka.ms/azurecli2).</span></span>

-   <span data-ttu-id="14b96-168">[PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/).</span><span class="sxs-lookup"><span data-stu-id="14b96-168">[PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/).</span></span>
    <span data-ttu-id="14b96-169">PowerShell용 Azure 모듈을 사용하면 스크립트를 사용하여 자동화 관리 작업을 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-169">The Azure modules for PowerShell allow you to execute automated management tasks using a script.</span></span> <span data-ttu-id="14b96-170">PowerShell은 [Windows, Linux 및 Mac OS](https://github.com/PowerShell/PowerShell)에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-170">PowerShell is available for [Windows, Linux, and Mac OS](https://github.com/PowerShell/PowerShell).</span></span>

-   <span data-ttu-id="14b96-171">[템플릿](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/).</span><span class="sxs-lookup"><span data-stu-id="14b96-171">[Templates](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/).</span></span>
    <span data-ttu-id="14b96-172">Azure Resource Manager 템플릿은 AWS CloudFormation 서비스와 비슷한 JSON 템플릿 기반 리소스 관리 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-172">Azure Resource Manager templates provide similar JSON template-based resource management capabilities to the AWS CloudFormation service.</span></span>

<span data-ttu-id="14b96-173">이러한 각 인터페이스에서 리소스 그룹은 Azure 리소스를 만들고 배포하고 수정하는 데 있어서 핵심적인 역할을 합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-173">In each of these interfaces, the resource group is central to how Azure resources get created, deployed, or modified.</span></span> <span data-ttu-id="14b96-174">CloudFormation 배포 시 "스택"이 AWS 리소스 그룹화에서 수행하는 역할과 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-174">This is similar to the role a "stack" plays in grouping AWS resources during CloudFormation deployments.</span></span>

<span data-ttu-id="14b96-175">이러한 인터페이스의 구문 및 구조는 AWS와 다르지만, 비슷한 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-175">The syntax and structure of these interfaces are different from their AWS equivalents, but they provide comparable capabilities.</span></span> <span data-ttu-id="14b96-176">또한 AWS에 사용되는 [Hashicorp's Terraform](https://www.terraform.io/docs/providers/azurerm/) 및 [Netflix Spinnaker](http://www.spinnaker.io/) 같은 여러 타사 관리 도구가 Azure에서도 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-176">In addition, many third party management tools used on AWS, like [Hashicorp's Terraform](https://www.terraform.io/docs/providers/azurerm/) and [Netflix Spinnaker](http://www.spinnaker.io/), are also available on Azure.</span></span>

### <a name="see-also"></a><span data-ttu-id="14b96-177">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-177">See also</span></span>

-   [<span data-ttu-id="14b96-178">Azure 리소스 그룹 지침</span><span class="sxs-lookup"><span data-stu-id="14b96-178">Azure resource group guidelines</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## <a name="regions-and-zones-high-availability"></a><span data-ttu-id="14b96-179">지역 및 영역(고가용성)</span><span class="sxs-lookup"><span data-stu-id="14b96-179">Regions and zones (high availability)</span></span>

<span data-ttu-id="14b96-180">오류는 해당 영향의 범위가 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-180">Failures can vary in the scope of their impact.</span></span> <span data-ttu-id="14b96-181">실패한 디스크와 같은 일부 하드웨어 오류는 단일 호스트 컴퓨터에 영향을 줄 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-181">Some hardware failures, such as a failed disk, may affect a single host machine.</span></span> <span data-ttu-id="14b96-182">실패한 네트워크 스위치는 전체 서버 랙에 영향을 줄 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-182">A failed network switch could affect a whole server rack.</span></span> <span data-ttu-id="14b96-183">데이터 센터의 전원 손실 등 전체 데이터 센터를 방해하는 오류는 일반적이지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-183">Less common are failures that disrupt a whole data center, such as loss of power in a data center.</span></span> <span data-ttu-id="14b96-184">가끔 전체 지역을 사용할 수 없게 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-184">Rarely, an entire region could become unavailable.</span></span>

<span data-ttu-id="14b96-185">중복성을 통해 응용 프로그램을 복원력 있게 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-185">One of the main ways to make an application resilient is through redundancy.</span></span> <span data-ttu-id="14b96-186">그러나 응용 프로그램을 디자인할 때 이러한 중복성에 대해 계획해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-186">But you need to plan for this redundancy when you design the application.</span></span> <span data-ttu-id="14b96-187">또한 필요한 중복성 수준은 비즈니스 요구 사항에 따라 달라집니다. &mdash; 지역 가동 중단으로부터 보호하기 위해 일부 응용 프로그램에 지역 간 중복성이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-187">Also, the level of redundancy that you need depends on your business requirements &mdash; not every application needs redundancy across regions to guard against a regional outage.</span></span> <span data-ttu-id="14b96-188">일반적으로 중복성과 안정성 및 비용과 복잡성 간에 균형을 조절해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-188">In general, there is a tradeoff between greater redundancy and reliability versus higher cost and complexity.</span></span>  

<span data-ttu-id="14b96-189">AWS에서는 한 영역이 두 개 이상의 가용성 영역으로 나뉩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-189">In AWS, a region is divided into two or more Availability Zones.</span></span> <span data-ttu-id="14b96-190">가용성 영역은 지리적 지역에 물리적으로 격리된 데이터 센터와 일치합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-190">An Availability Zone corresponds with a physically isolated datacenter in the geographic region.</span></span> <span data-ttu-id="14b96-191">Azure에는 **가용성 집합**, **가용성 영역** 및 **쌍을 이루는 지역** 등 각 응용 프로그램 오류 수준에서 중복되는 응용 프로그램을 만드는 여러 기능이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-191">Azure has a number of features to make an application redundant at every level of failure, including **availability sets**, **availability zones**, and **paired regions**.</span></span> 

![](../resiliency/images/redundancy.svg)

<span data-ttu-id="14b96-192">다음 표에서는 각 옵션을 요약합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-192">The following table summarizes each option.</span></span>

| &nbsp; | <span data-ttu-id="14b96-193">가용성 집합</span><span class="sxs-lookup"><span data-stu-id="14b96-193">Availability Set</span></span> | <span data-ttu-id="14b96-194">가용성 영역</span><span class="sxs-lookup"><span data-stu-id="14b96-194">Availability Zone</span></span> | <span data-ttu-id="14b96-195">쌍을 이루는 지역</span><span class="sxs-lookup"><span data-stu-id="14b96-195">Paired region</span></span> |
|--------|------------------|-------------------|---------------|
| <span data-ttu-id="14b96-196">오류의 범위</span><span class="sxs-lookup"><span data-stu-id="14b96-196">Scope of failure</span></span> | <span data-ttu-id="14b96-197">랙</span><span class="sxs-lookup"><span data-stu-id="14b96-197">Rack</span></span> | <span data-ttu-id="14b96-198">데이터 센터</span><span class="sxs-lookup"><span data-stu-id="14b96-198">Datacenter</span></span> | <span data-ttu-id="14b96-199">지역</span><span class="sxs-lookup"><span data-stu-id="14b96-199">Region</span></span> |
| <span data-ttu-id="14b96-200">요청 라우팅</span><span class="sxs-lookup"><span data-stu-id="14b96-200">Request routing</span></span> | <span data-ttu-id="14b96-201">Load Balancer</span><span class="sxs-lookup"><span data-stu-id="14b96-201">Load Balancer</span></span> | <span data-ttu-id="14b96-202">영역 간 부하 분산 장치</span><span class="sxs-lookup"><span data-stu-id="14b96-202">Cross-zone Load Balancer</span></span> | <span data-ttu-id="14b96-203">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="14b96-203">Traffic Manager</span></span> |
| <span data-ttu-id="14b96-204">네트워크 대기 시간</span><span class="sxs-lookup"><span data-stu-id="14b96-204">Network latency</span></span> | <span data-ttu-id="14b96-205">매우 낮음</span><span class="sxs-lookup"><span data-stu-id="14b96-205">Very low</span></span> | <span data-ttu-id="14b96-206">낮음</span><span class="sxs-lookup"><span data-stu-id="14b96-206">Low</span></span> | <span data-ttu-id="14b96-207">중간부터 높음</span><span class="sxs-lookup"><span data-stu-id="14b96-207">Mid to high</span></span> |
| <span data-ttu-id="14b96-208">가상 네트워킹</span><span class="sxs-lookup"><span data-stu-id="14b96-208">Virtual networking</span></span>  | <span data-ttu-id="14b96-209">VNet</span><span class="sxs-lookup"><span data-stu-id="14b96-209">VNet</span></span> | <span data-ttu-id="14b96-210">VNet</span><span class="sxs-lookup"><span data-stu-id="14b96-210">VNet</span></span> | <span data-ttu-id="14b96-211">지역 간 VNet 피어링(미리 보기)</span><span class="sxs-lookup"><span data-stu-id="14b96-211">Cross-region VNet peering (preview)</span></span> |

### <a name="availability-sets"></a><span data-ttu-id="14b96-212">가용성 집합</span><span class="sxs-lookup"><span data-stu-id="14b96-212">Availability sets</span></span> 

<span data-ttu-id="14b96-213">디스크 또는 네트워크 전환이 실패한 경우 하드웨어 오류로부터 보호하려면 가용성 집합에 둘 이상의 VM을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-213">To protect against localized hardware failures, such as a disk or network switch failing, deploy two or more VMs in an availability set.</span></span> <span data-ttu-id="14b96-214">가용성 집합은 공통 전원 소스 및 네트워크 스위치를 공유하는 두 개 이상의 *장애 도메인*으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-214">An availability set consists of two or more *fault domains* that share a common power source and network switch.</span></span> <span data-ttu-id="14b96-215">가용성 집합의 VM은 장애 도메인에 분산되어 있으므로 하드웨어 오류가 하나의 장애 도메인에 영향을 주는 경우 네트워크 트래픽은 다른 오류 도메인에서 VM을 라우팅할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-215">VMs in an availability set are distributed across the fault domains, so if a hardware failure affects one fault domain, network traffic can still be routed the VMs in the other fault domains.</span></span> <span data-ttu-id="14b96-216">가용성 집합에 대한 자세한 내용은 [Azure에서 Windows 가상 머신의 가용성 관리](/azure/virtual-machines/windows/manage-availability)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="14b96-216">For more information about Availability Sets, see [Manage the availability of Windows virtual machines in Azure](/azure/virtual-machines/windows/manage-availability).</span></span>

<span data-ttu-id="14b96-217">가용성 집합에 추가되는 VM 인스턴스에는 [업데이트 도메인](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/)이 할당됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-217">When VM instances are added to availability sets, they are also assigned an [update domain](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/).</span></span> <span data-ttu-id="14b96-218">업데이트 도메인은 동시에 계획된 유지 관리 이벤트를 수행하도록 설정된 VM 그룹입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-218">An update domain is a group of VMs that are set for planned maintenance events at the same time.</span></span> <span data-ttu-id="14b96-219">VM을 여러 업데이트 도메인에 분산하면 계획된 업데이트 및 패치 이벤트가 지정된 시간에 이러한 VM의 하위 집합에만 영향을 줍니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-219">Distributing VMs across multiple update domains ensures that planned update and patching events affect only a subset of these VMs at any given time.</span></span>

<span data-ttu-id="14b96-220">각 역할의 한 인스턴스가 정상적으로 작동하려면 응용 프로그램의 인스턴스 역할을 통해 가용성 집합을 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-220">Availability sets should be organized by the instance's role in your application to ensure one instance in each role is operational.</span></span> <span data-ttu-id="14b96-221">예를 들어 3계층 웹 응용 프로그램에서는 프런트 엔드, 응용 프로그램 및 데이터 계층에 대한 별도의 가용성 집합을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-221">For example, in a three-tier web application, create separate availability sets for the front-end, application, and data tiers.</span></span>

<span data-ttu-id="14b96-222">![각 응용 프로그램 역할에 대한 Azure 가용성 집합](./images/three-tier-example.png "각 응용 프로그램 역할에 대한 Azure 가용성 집합")</span><span class="sxs-lookup"><span data-stu-id="14b96-222">![Azure availability sets for each application role](./images/three-tier-example.png "Availability sets for each application role")</span></span>

### <a name="availability-zones-preview"></a><span data-ttu-id="14b96-223">가용성 영역(미리 보기)</span><span class="sxs-lookup"><span data-stu-id="14b96-223">Availability zones (preview)</span></span>

<span data-ttu-id="14b96-224">[가용성 영역](/azure/availability-zones/az-overview)은 Azure 지역 내에서 물리적으로 별도의 영역입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-224">An [Availability Zone](/azure/availability-zones/az-overview) is a physically separate zone within an Azure region.</span></span> <span data-ttu-id="14b96-225">각 가용성 영역에는 고유한 소스, 네트워크 및 냉각 장치가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-225">Each Availability Zone has a distinct power source, network, and cooling.</span></span> <span data-ttu-id="14b96-226">가용성 영역 간에 VM을 배포하면 데이터 센터 전체의 오류로부터 응용 프로그램을 보호할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-226">Deploying VMs across availability zones helps to protect an application against datacenter-wide failures.</span></span> 

### <a name="paired-regions"></a><span data-ttu-id="14b96-227">쌍을 이루는 지역</span><span class="sxs-lookup"><span data-stu-id="14b96-227">Paired regions</span></span>

<span data-ttu-id="14b96-228">지역 가동 중단으로부터 응용 프로그램을 보호하려면 인터넷 트래픽을 서로 다른 지역에 배포하는 [Azure Traffic Manager][traffic-manager]를 사용하여 응용 프로그램을 여러 지역에 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-228">To protect an application against a regional outage, you can deploy the application across multiple regions, using [Azure Traffic Manager][traffic-manager] to distribute internet traffic to the different regions.</span></span> <span data-ttu-id="14b96-229">각 Azure 지역은 다른 지역과 쌍을 이룹니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-229">Each Azure region is paired with another region.</span></span> <span data-ttu-id="14b96-230">이러한 지역은 함께 [지역 쌍][paired-regions]을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-230">Together, these form a [regional pair][paired-regions].</span></span> <span data-ttu-id="14b96-231">브라질 남부를 제외하고 지역 쌍은 세금 및 법률 집행 관할 구역의 데이터 상주 요구 사항을 충족하기 위해 동일한 지리적 위치 내에 위치합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-231">With the exception of Brazil South, regional pairs are located within the same geography in order to meet data residency requirements for tax and law enforcement jurisdiction purposes.</span></span>

<span data-ttu-id="14b96-232">데이터 센터가 물리적으로 떨어져 있지만 비교적 가까운 영역에 있는 가용성 영역과는 달리, 쌍을 이루는 지역은 일반적으로 480킬로미터 이상 떨어져 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-232">Unlike Availability Zones, which are physically separate datacenters but may be in relatively nearby geographic areas, paired regions are usually separated by at least 300 miles.</span></span> <span data-ttu-id="14b96-233">이는 대규모 재해가 발생하더라도 쌍을 이루는 지역의 한 쪽 지역만 영향을 받게 하려는 의도입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-233">This is intended to ensure larger scale disasters only impact one of the regions in the pair.</span></span> <span data-ttu-id="14b96-234">인접한 쌍은 데이터베이스 및 저장소 서비스 데이터를 동기화할 때 설정할 수 있으며, 쌍을 이루는 지역 중 한 번에 한 영역에서만 플랫폼 업데이트가 수행되도록 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-234">Neighboring pairs can be set to sync database and storage service data, and are configured so that platform updates are rolled out to only one region in the pair at a time.</span></span>

<span data-ttu-id="14b96-235">Azure [지역 중복 저장소](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage)는 적절한 쌍을 이루는 지역에 자동으로 백업됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-235">Azure [geo-redundant storage](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage) is automatically backed up to the appropriate paired region.</span></span> <span data-ttu-id="14b96-236">그 외 리소스의 경우 쌍을 이루는 지역을 사용하여 완전한 이중화 솔루션을 만든다는 것은 두 영역 모두에 솔루션 전체 복사본을 만든다는 의미입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-236">For all other resources, creating a fully redundant solution using paired regions means creating a full copy of your solution in both regions.</span></span>


### <a name="see-also"></a><span data-ttu-id="14b96-237">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-237">See also</span></span>

-   [<span data-ttu-id="14b96-238">Azure에서 가상 머신의 영역 및 가용성</span><span class="sxs-lookup"><span data-stu-id="14b96-238">Regions and availability for virtual machines in Azure</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [<span data-ttu-id="14b96-239">Azure 응용 프로그램의 고가용성</span><span class="sxs-lookup"><span data-stu-id="14b96-239">High availability for Azure applications</span></span>](../resiliency/high-availability-azure-applications.md)

-   [<span data-ttu-id="14b96-240">Azure 응용 프로그램에 대한 재해 복구</span><span class="sxs-lookup"><span data-stu-id="14b96-240">Disaster recovery for Azure applications</span></span>](../resiliency/disaster-recovery-azure-applications.md)

-   [<span data-ttu-id="14b96-241">Azure에서 Linux 가상 머신에 대한 계획된 유지 관리</span><span class="sxs-lookup"><span data-stu-id="14b96-241">Planned maintenance for Linux virtual machines in Azure</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## <a name="services"></a><span data-ttu-id="14b96-242">Services</span><span class="sxs-lookup"><span data-stu-id="14b96-242">Services</span></span>

<span data-ttu-id="14b96-243">모든 서비스가 플랫폼 간에 매핑되는 방식에 대한 전체 목록은 [AWS와 Azure의 전체 서비스 비교표](https://aka.ms/azure4aws-services)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="14b96-243">Consult the [complete AWS and Azure service comparison matrix](https://aka.ms/azure4aws-services) for a full listing of how all services map between platforms.</span></span>

<span data-ttu-id="14b96-244">지역에 따라 일부 Azure 제품 및 서비스가 제공되지 않을 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-244">Not all Azure products and services are available in all regions.</span></span> <span data-ttu-id="14b96-245">자세한 내용은 [지역별 제품](https://azure.microsoft.com/regions/services/) 페이지를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="14b96-245">Consult the [Products by Region](https://azure.microsoft.com/regions/services/) page for details.</span></span> <span data-ttu-id="14b96-246">[서비스 수준 계약](https://azure.microsoft.com/support/legal/sla/) 페이지에서 각 Azure 제품 및 서비스의 작동 시간 보장 및 가동 중지 시간 크레딧 정책을 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-246">You can find the uptime guarantees and downtime credit policies for each Azure product or service on the [Service Level Agreements](https://azure.microsoft.com/support/legal/sla/) page.</span></span>

<span data-ttu-id="14b96-247">다음 단원에서는 AWS 및 Azure 플랫폼에서 자주 사용되는 기능과 서비스가 서로 어떻게 다른지 간략하게 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-247">The following sections provide a brief explanation of how commonly used features and services differ between the AWS and Azure platforms.</span></span>

### <a name="compute-services"></a><span data-ttu-id="14b96-248">Compute 서비스</span><span class="sxs-lookup"><span data-stu-id="14b96-248">Compute services</span></span>

#### <a name="ec2-instances-and-azure-virtual-machines"></a><span data-ttu-id="14b96-249">EC2 인스턴스 및 Azure 가상 머신</span><span class="sxs-lookup"><span data-stu-id="14b96-249">EC2 Instances and Azure virtual machines</span></span>

<span data-ttu-id="14b96-250">AWS 인스턴스 형식과 Azure 가상 머신 크기는 비슷한 방식으로 분류되지만 RAM, CPU 및 저장소 기능에서 차이가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-250">Although AWS instance types and Azure virtual machine sizes breakdown in a similar way, there are differences in the RAM, CPU, and storage capabilities.</span></span>

-   [<span data-ttu-id="14b96-251">Amazon EC2 인스턴스 형식</span><span class="sxs-lookup"><span data-stu-id="14b96-251">Amazon EC2 Instance Types</span></span>](https://aws.amazon.com/ec2/instance-types/)

-   [<span data-ttu-id="14b96-252">Azure에서 가상 머신 크기(Windows)</span><span class="sxs-lookup"><span data-stu-id="14b96-252">Sizes for virtual machines in Azure (Windows)</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [<span data-ttu-id="14b96-253">Azure에서 가상 머신 크기(Linux)</span><span class="sxs-lookup"><span data-stu-id="14b96-253">Sizes for virtual machines in Azure (Linux)</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

<span data-ttu-id="14b96-254">초 단위로 요금이 청구되는 AWS와는 달리, Azure 주문형 VM은 분 단위로 요금이 청구됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-254">Unlike AWS' per second billing, Azure on-demand VMs are billed by the minute.</span></span>

<span data-ttu-id="14b96-255">Azure에는 EC2 스폿 인스턴스 또는 전용 호스트에 해당하는 항목이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-255">Azure has no equivalent to EC2 Spot Instances or Dedicated Hosts.</span></span>

#### <a name="ebs-and-azure-storage-for-vm-disks"></a><span data-ttu-id="14b96-256">VM 디스크용 EBS 및 Azure Storage</span><span class="sxs-lookup"><span data-stu-id="14b96-256">EBS and Azure Storage for VM disks</span></span>

<span data-ttu-id="14b96-257">BLOB 저장소에 상주하는 [데이터 디스크](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/)가 내구성이 우수한 Azure VM용 데이터 저장소를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-257">Durable data storage for Azure VMs is provided by [data disks](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/) residing in blob storage.</span></span> <span data-ttu-id="14b96-258">EC2 인스턴스가 EBS(Elastic Block Store)에 디스크 볼륨을 저장하는 방식과 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-258">This is similar to how EC2 instances store disk volumes on Elastic Block Store (EBS).</span></span> <span data-ttu-id="14b96-259">[Azure 임시 저장소](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/) 역시 EC2 인스턴스 저장소(사용 후 삭제 저장소라고도 함)와 동일하게 대기 시간이 짧은 임시 읽기-쓰기 저장소를 VM에 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-259">[Azure temporary storage](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/) also provides VMs the same low-latency temporary read-write storage as EC2 Instance Storage (also called ephemeral storage).</span></span>

<span data-ttu-id="14b96-260">[Azure Premium Storage](https://docs.microsoft.com/azure/storage/storage-premium-storage)를 사용하여 고성능 디스크 IO를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-260">Higher performance disk IO is supported using [Azure premium storage](https://docs.microsoft.com/azure/storage/storage-premium-storage).</span></span>
<span data-ttu-id="14b96-261">이것은 AWS에서 제공하는 프로비전된 IOPS 스토리지 옵션과 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-261">This is similar to the Provisioned IOPS storage options provided by AWS.</span></span>

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a><span data-ttu-id="14b96-262">Lambda, Azure Functions, Azure Web-Jobs 및 Azure Logic Apps</span><span class="sxs-lookup"><span data-stu-id="14b96-262">Lambda, Azure Functions, Azure Web-Jobs, and Azure Logic Apps</span></span>

<span data-ttu-id="14b96-263">[Azure Functions](https://azure.microsoft.com/services/functions/)는 서버 없는 주문형 코드를 제공한다는 측면에서 AWS Lambda와 가장 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-263">[Azure Functions](https://azure.microsoft.com/services/functions/) is the primary equivalent of AWS Lambda in providing serverless, on-demand code.</span></span>
<span data-ttu-id="14b96-264">그러나 Lambda 기능은 다른 Azure 서비스와도 겹칩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-264">However, Lambda functionality also overlaps with other Azure services:</span></span>

-   <span data-ttu-id="14b96-265">[WebJobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - 예약된 또는 지속적으로 실행되는 백그라운드 작업을 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-265">[WebJobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - allow you to create scheduled or continuously running background tasks.</span></span>

-   <span data-ttu-id="14b96-266">[Logic Apps](https://azure.microsoft.com/services/logic-apps/) - 통신, 통합 및 비즈니스 규칙 관리 서비스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-266">[Logic Apps](https://azure.microsoft.com/services/logic-apps/) - provides communications, integration, and business rule management services.</span></span>

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a><span data-ttu-id="14b96-267">자동 크기 조정, Azure VM 크기 조정 및 Azure App Service</span><span class="sxs-lookup"><span data-stu-id="14b96-267">Autoscaling, Azure VM scaling, and Azure App Service Autoscale</span></span>

<span data-ttu-id="14b96-268">Azure의 자동 크기 조정은 다음 두 서비스에서 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-268">Autoscaling in Azure is handled by two services:</span></span>

-   <span data-ttu-id="14b96-269">[VM 확장 집합](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) - 동일한 VM 집합을 배포하고 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-269">[VM scale sets](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) - allow you to deploy and manage an identical set of VMs.</span></span> <span data-ttu-id="14b96-270">인스턴스 수는 성능 요구 사항에 따라 자동으로 조정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-270">The number of instances can autoscale based on performance needs.</span></span>

-   <span data-ttu-id="14b96-271">[App Service 자동 크기 조정](https://azure.microsoft.com/documentation/articles/web-sites-scale/) - Azure App Service 솔루션의 크기를 자동으로 조정하는 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-271">[App Service Autoscale](https://azure.microsoft.com/documentation/articles/web-sites-scale/) - provides the capability to autoscale Azure App Service solutions.</span></span>


#### <a name="container-service"></a><span data-ttu-id="14b96-272">컨테이너 서비스</span><span class="sxs-lookup"><span data-stu-id="14b96-272">Container Service</span></span>
<span data-ttu-id="14b96-273">[Azure Container Service](https://docs.microsoft.com/azure/container-service/container-service-intro)는 Docker Swarm, Kubernetes 또는 DC/OS를 통해 관리되는 Docker 컨테이너를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-273">The [Azure Container Service](https://docs.microsoft.com/azure/container-service/container-service-intro) supports Docker containers managed through Docker Swarm, Kubernetes, or DC/OS.</span></span>

#### <a name="other-compute-services"></a><span data-ttu-id="14b96-274">기타 계산 서비스</span><span class="sxs-lookup"><span data-stu-id="14b96-274">Other compute services</span></span> 


<span data-ttu-id="14b96-275">Azure는 AWS와 약간 차이가 있는 여러 계산 서비스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-275">Azure offers several compute services that do not have direct equivalents in AWS:</span></span>

-   <span data-ttu-id="14b96-276">[Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) - 확장 가능한 가상 머신 컬렉션에서 계산 집약적인 작업을 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-276">[Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) - allows you to manage compute-intensive work across a scalable collection of virtual machines.</span></span>

-   <span data-ttu-id="14b96-277">[Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) - 확장 가능한 [마이크로 서비스](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/) 솔루션을 개발하고 호스팅할 수 있는 플랫폼입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-277">[Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) - platform for developing and hosting scalable [microservice](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/) solutions.</span></span>

#### <a name="see-also"></a><span data-ttu-id="14b96-278">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-278">See also</span></span>

-   [<span data-ttu-id="14b96-279">포털을 사용하여 Azure에서 Linux VM 만들기</span><span class="sxs-lookup"><span data-stu-id="14b96-279">Create a Linux VM on Azure using the Portal</span></span>](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [<span data-ttu-id="14b96-280">Azure 참조 아키텍처: Azure에서 Linux VM 실행</span><span class="sxs-lookup"><span data-stu-id="14b96-280">Azure Reference Architecture: Running a Linux VM on Azure</span></span>](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [<span data-ttu-id="14b96-281">Azure App Service에서 Node.js 웹앱 시작</span><span class="sxs-lookup"><span data-stu-id="14b96-281">Get started with Node.js web apps in Azure App Service</span></span>](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [<span data-ttu-id="14b96-282">Azure 참조 아키텍처: 기본 웹 응용 프로그램</span><span class="sxs-lookup"><span data-stu-id="14b96-282">Azure Reference Architecture: Basic web application</span></span>](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [<span data-ttu-id="14b96-283">첫 번째 Azure Function 만들기</span><span class="sxs-lookup"><span data-stu-id="14b96-283">Create your first Azure Function</span></span>](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### <a name="storage"></a><span data-ttu-id="14b96-284">Storage</span><span class="sxs-lookup"><span data-stu-id="14b96-284">Storage</span></span>

#### <a name="s3ebsefs-and-azure-storage"></a><span data-ttu-id="14b96-285">S3/EBS/EFS 및 Azure Storage</span><span class="sxs-lookup"><span data-stu-id="14b96-285">S3/EBS/EFS and Azure Storage</span></span>

<span data-ttu-id="14b96-286">AWS 플랫폼에서 클라우드 저장소는 주로 세 가지 서비스로 분류됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-286">In the AWS platform, cloud storage is primarily broken down into three services:</span></span>

-   <span data-ttu-id="14b96-287">**S3(Simple Storage Service)** - 기본 개체 저장소입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-287">**Simple Storage Service (S3)** - basic object storage.</span></span> <span data-ttu-id="14b96-288">인터넷 액세스가 가능한 API를 통해 데이터를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-288">Makes data available through an Internet accessible API.</span></span>

-   <span data-ttu-id="14b96-289">**EBS(Elastic Block Storage)** - 단일 VM 액세스를 위한 블록 수준 저장소입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-289">**Elastic Block Storage (EBS)** - block level storage, intended for access by a single VM.</span></span>

-   <span data-ttu-id="14b96-290">**EFS(Elastic File System)** - EC2 인스턴스 수천 개의 공유 저장소로 사용되는 파일 저장소입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-290">**Elastic File System (EFS)** - file storage meant for use as shared storage for up to thousands of EC2 instances.</span></span>

<span data-ttu-id="14b96-291">Azure Storage에서는 구독에 바인딩된 [저장소 계정](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/)을 사용하여 다음과 같은 저장소 서비스를 만들고 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-291">In Azure Storage, subscription-bound [storage accounts](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) allow you to create and manage the following storage services:</span></span>

-   <span data-ttu-id="14b96-292">[Blob Storage](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - 문서, 미디어 파일 또는 응용 프로그램 설치 프로그램과 같은 모든 종류의 텍스트 또는 이진 데이터를 저장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-292">[Blob storage](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - stores any type of text or binary data, such as a document, media file, or application installer.</span></span> <span data-ttu-id="14b96-293">개인 액세스에 대해 Blob Storage를 설정하거나 인터넷에 공개적으로 콘텐츠를 공유할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-293">You can set Blob storage for private access or share contents publicly to the Internet.</span></span> <span data-ttu-id="14b96-294">Blob Storage는 AWS S3 및 EBS와 동일한 용도로 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-294">Blob storage serves the same purpose as both AWS S3 and EBS.</span></span>
-   <span data-ttu-id="14b96-295">[Table Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - 구조화된 데이터 집합을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-295">[Table storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - stores structured datasets.</span></span> <span data-ttu-id="14b96-296">Table Storage는 신속한 개발과 대량 데이터에 대한 빠른 액세스를 가능하게 하는 NoSQL 키-특성 데이터 저장소입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-296">Table storage is a NoSQL key-attribute data store that allows for rapid development and fast access to large quantities of data.</span></span> <span data-ttu-id="14b96-297">AWS의 SimpleDB 및 DynamoDB 서비스와 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-297">Similar to AWS' SimpleDB and DynamoDB services.</span></span>

-   <span data-ttu-id="14b96-298">[Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - 워크플로 처리 및 클라우드 서비스 구성 요소 사이의 통신을 위한 메시지를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-298">[Queue storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - provides messaging for workflow processing and for communication between components of cloud services.</span></span>

-   <span data-ttu-id="14b96-299">[File Storage](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - 표준 SMB(서버 메시지 블록) 프로토콜을 사용하는 기존 응용 프로그램을 위한 공유 저장소를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-299">[File storage](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - offers shared storage for legacy applications using the standard server message block (SMB) protocol.</span></span> <span data-ttu-id="14b96-300">File Storage는 AWS 플랫폼의 EFS와 비슷한 방식으로 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-300">File storage is used in a similar manner to EFS in the AWS platform.</span></span>
 
#### <a name="glacier-and-azure-storage"></a><span data-ttu-id="14b96-301">Glacier 및 Azure Storage</span><span class="sxs-lookup"><span data-stu-id="14b96-301">Glacier and Azure Storage</span></span> 

<span data-ttu-id="14b96-302">[Azure Archive Blob Storage](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier)는 AWS Glacier 저장소 서비스에 해당합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-302">[Azure Archive Blob Storage](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier) is comparable to AWS Glacier storage service.</span></span> <span data-ttu-id="14b96-303">180일 이상 저장되면서 거의 액세스하지 않으며 몇 시간 동안의 검색 대기 시간도 용인할 수 있는 데이터를 대상으로 합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-303">It is intended for rarely accessed data that is stored for at least 180 days and can tolerate several hours of retrieval latency.</span></span> 

<span data-ttu-id="14b96-304">자주 액세스하지 않지만 액세스할 때 즉시 사용 가능해야 하는 데이터의 경우 [Azure Cool Blob Storage 계층](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier)에서 표준 Blob 저장소보다 저렴한 저장소를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-304">For data that is infrequently accessed but must be available immediately when accessed, [Azure Cool Blob Storage tier](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier) provides cheaper storage than standard blob storage.</span></span> <span data-ttu-id="14b96-305">이 저장소 계층은 AWS S3 - 자주 액세스하지 않는 저장소 서비스에 해당합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-305">This storage tier is comparable to AWS S3 - Infrequent Access storage service.</span></span>

#### <a name="see-also"></a><span data-ttu-id="14b96-306">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-306">See also</span></span>

-   [<span data-ttu-id="14b96-307">Microsoft Azure Storage 성능 및 확장성 검사 목록</span><span class="sxs-lookup"><span data-stu-id="14b96-307">Microsoft Azure Storage Performance and Scalability Checklist</span></span>](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [<span data-ttu-id="14b96-308">Azure Storage 보안 가이드</span><span class="sxs-lookup"><span data-stu-id="14b96-308">Azure Storage security guide</span></span>](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [<span data-ttu-id="14b96-309">패턴 및 연습: CDN(Content Delivery Network) 지침</span><span class="sxs-lookup"><span data-stu-id="14b96-309">Patterns & Practices: Content Delivery Network (CDN) guidance</span></span>](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### <a name="networking"></a><span data-ttu-id="14b96-310">네트워킹</span><span class="sxs-lookup"><span data-stu-id="14b96-310">Networking</span></span>

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a><span data-ttu-id="14b96-311">Elastic Load Balancing, Azure Load Balancer 및 Azure Application Gateway</span><span class="sxs-lookup"><span data-stu-id="14b96-311">Elastic Load Balancing, Azure Load Balancer, and Azure Application Gateway</span></span>

<span data-ttu-id="14b96-312">두 Elastic Load Balancing 서비스에 해당하는 Azure 서비스는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-312">The Azure equivalents of the two Elastic Load Balancing services are:</span></span>

-   <span data-ttu-id="14b96-313">[Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - AWS Classic Load Balancer와 동일한 기능을 제공하며, 네트워크 수준에서 여러 VM의 트래픽을 분산할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-313">[Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - provides the same capabilities as the AWS Classic Load Balancer, allowing you to distribute traffic for multiple VMs at the network level.</span></span> <span data-ttu-id="14b96-314">장애 조치(failover) 기능도 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-314">It also provides failover capability.</span></span>

-   <span data-ttu-id="14b96-315">[Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - AWS Application Load Balancer와 비슷한 응용 프로그램 수준 규칙 기반 라우팅을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-315">[Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - offers application-level rule-based routing comparable to the AWS Application Load Balancer.</span></span>

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a><span data-ttu-id="14b96-316">Route 53, Azure DNS 및 Azure Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="14b96-316">Route 53, Azure DNS, and Azure Traffic Manager</span></span>

<span data-ttu-id="14b96-317">AWS의 Route 53은 DNS 이름 관리 및 DNS 수준 트래픽 라우팅과 장애 조치(failover) 서비스를 모두 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-317">In AWS, Route 53 provides both DNS name management and DNS-level traffic routing and failover services.</span></span> <span data-ttu-id="14b96-318">Azure에서 이러한 작업이 다음 두 서비스를 통해 처리됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-318">In Azure this is handled through two services:</span></span>

-   <span data-ttu-id="14b96-319">[Azure DNS](https://azure.microsoft.com/documentation/services/dns/)는 도메인 및 DNS 관리를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-319">[Azure DNS](https://azure.microsoft.com/documentation/services/dns/) provides domain and DNS management.</span></span>

-   <span data-ttu-id="14b96-320">[Traffic Manager][traffic-manager]는 DNS 수준 트래픽 라우팅, 부하 분산 및 장애 조치(failover) 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-320">[Traffic Manager][traffic-manager] provides DNS level traffic routing, load balancing, and failover capabilities.</span></span>

#### <a name="direct-connect-and-azure-expressroute"></a><span data-ttu-id="14b96-321">Direct Connect 및 Azure ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="14b96-321">Direct Connect and Azure ExpressRoute</span></span>

<span data-ttu-id="14b96-322">Azure는 [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) 서비스를 통해 AWS와 비슷한 사이트 간 전용 연결을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-322">Azure provides similar site-to-site dedicated connections through its [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) service.</span></span> <span data-ttu-id="14b96-323">ExpressRoute를 사용하면 전용 개인 네트워크 연결을 사용하여 로컬 네트워크를 Azure 리소스에 직접 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-323">ExpressRoute allows you to connect your local network directly to Azure resources using a dedicated private network connection.</span></span> <span data-ttu-id="14b96-324">또한 Azure는 기존의 [사이트 간 VPN 연결](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)을 좀 더 저렴한 가격에 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-324">Azure also offers more conventional [site-to-site VPN connections](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/) at a lower cost.</span></span>

#### <a name="see-also"></a><span data-ttu-id="14b96-325">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-325">See also</span></span>

-   [<span data-ttu-id="14b96-326">Azure Portal을 사용하여 가상 네트워크 만들기</span><span class="sxs-lookup"><span data-stu-id="14b96-326">Create a virtual network using the Azure portal</span></span>](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [<span data-ttu-id="14b96-327">Azure Virtual Network 계획 및 디자인</span><span class="sxs-lookup"><span data-stu-id="14b96-327">Plan and design Azure Virtual Networks</span></span>](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [<span data-ttu-id="14b96-328">Azure 네트워크 보안 모범 사례</span><span class="sxs-lookup"><span data-stu-id="14b96-328">Azure Network Security Best Practices</span></span>](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a><span data-ttu-id="14b96-329">데이터베이스 서비스</span><span class="sxs-lookup"><span data-stu-id="14b96-329">Database services</span></span>

#### <a name="rds-and-azure-relational-database-services"></a><span data-ttu-id="14b96-330">RDS 및 Azure 관계형 데이터베이스 서비스</span><span class="sxs-lookup"><span data-stu-id="14b96-330">RDS and Azure relational database services</span></span>

<span data-ttu-id="14b96-331">Azure의 AWS RDS(관계형 데이터베이스 서비스)에 해당하는 몇 가지 다른 관계형 데이터베이스 서비스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-331">Azure provides several different relational database services that are the equivalent of AWS' Relational Database Service (RDS).</span></span>

-   [<span data-ttu-id="14b96-332">SQL Database</span><span class="sxs-lookup"><span data-stu-id="14b96-332">SQL Database</span></span>](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
-   [<span data-ttu-id="14b96-333">Azure Database for MySQL</span><span class="sxs-lookup"><span data-stu-id="14b96-333">Azure Database for MySQL</span></span>](https://docs.microsoft.com/azure/mysql/overview)
-   [<span data-ttu-id="14b96-334">Azure Database for PostgreSQL</span><span class="sxs-lookup"><span data-stu-id="14b96-334">Azure Database for PostgreSQL</span></span>](https://docs.microsoft.com/azure/postgresql/overview)

<span data-ttu-id="14b96-335">[SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/), [Oracle](https://azure.microsoft.com/campaigns/oracle/) 및 [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) 같은 다른 데이터베이스 엔진은 Azure VM 인스턴스를 사용하여 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-335">Other database engines such as [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/), [Oracle](https://azure.microsoft.com/campaigns/oracle/), and [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/) can be deployed using Azure VM Instances.</span></span>

<span data-ttu-id="14b96-336">AWS RDS의 비용은 CPU, RAM, 저장소, 네트워크 대역폭 등 인스턴스에서 사용하는 하드웨어 리소스의 양에 따라 결정됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-336">Costs for AWS RDS are determined by the amount of hardware resources that your instance uses, like CPU, RAM, storage, and network bandwidth.</span></span> <span data-ttu-id="14b96-337">Azure 데이터베이스 서비스의 비용은 데이터베이스 크기, 동시 연결 수 및 처리량 수준에 따라 결정됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-337">In the Azure database services, cost depends on your database size, concurrent connections, and throughput levels.</span></span>

#### <a name="see-also"></a><span data-ttu-id="14b96-338">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-338">See also</span></span>

-   [<span data-ttu-id="14b96-339">Azure SQL Database 자습서</span><span class="sxs-lookup"><span data-stu-id="14b96-339">Azure SQL Database Tutorials</span></span>](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [<span data-ttu-id="14b96-340">Azure Portal로 Azure SQL Database에 대한 지역에서 복제 구성</span><span class="sxs-lookup"><span data-stu-id="14b96-340">Configure geo-replication for Azure SQL Database with the Azure portal</span></span>](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [<span data-ttu-id="14b96-341">Cosmos DB 소개: NoSQL JSON 데이터베이스</span><span class="sxs-lookup"><span data-stu-id="14b96-341">Introduction to Cosmos DB: A NoSQL JSON Database</span></span>](/azure/cosmos-db/sql-api-introduction)

-   [<span data-ttu-id="14b96-342">Node.js에서 Azure Table Storage를 사용하는 방법</span><span class="sxs-lookup"><span data-stu-id="14b96-342">How to use Azure Table storage from Node.js</span></span>](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a><span data-ttu-id="14b96-343">보안 및 ID</span><span class="sxs-lookup"><span data-stu-id="14b96-343">Security and identity</span></span>

#### <a name="directory-service-and-azure-active-directory"></a><span data-ttu-id="14b96-344">디렉터리 서비스 및 Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="14b96-344">Directory service and Azure Active Directory</span></span>

<span data-ttu-id="14b96-345">Azure는 디렉터리 서비스를 다음과 같은 제품으로 나눠 놓았습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-345">Azure splits up directory services into the following offerings:</span></span>

-   <span data-ttu-id="14b96-346">[Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - 클라우드 기반 디렉터리 및 ID 관리 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-346">[Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - cloud based directory and identity management service.</span></span>

-   <span data-ttu-id="14b96-347">[Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - 파트너가 관리하는 ID로 회사 응용 프로그램에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-347">[Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - enables access to your corporate applications from partner-managed identities.</span></span>

-   <span data-ttu-id="14b96-348">[Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - 소비자용 응용 프로그램의 Single Sign-On 및 사용자 관리를 지원하는 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-348">[Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - service offering support for single sign-on and user management for consumer facing applications.</span></span>

-   <span data-ttu-id="14b96-349">[Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - 호스팅된 도메인 컨트롤러 서비스로, Active Directory 호환 도메인 가입 및 사용자 관리 기능을 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-349">[Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - hosted domain controller service, allowing Active Directory compatible domain join and user management functionality.</span></span>

#### <a name="web-application-firewall"></a><span data-ttu-id="14b96-350">웹 응용 프로그램 방화벽</span><span class="sxs-lookup"><span data-stu-id="14b96-350">Web application firewall</span></span>

<span data-ttu-id="14b96-351">[Application Gateway 웹 응용 프로그램 방화벽](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) 외에도 [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/) 같은 타사 공급업체의 [웹 응용 프로그램 방화벽을 사용](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-351">In addition to the [Application Gateway Web Application Firewall](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/), you can also [use web application firewalls](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) from third-party vendors like [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/).</span></span>

#### <a name="see-also"></a><span data-ttu-id="14b96-352">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-352">See also</span></span>

-   [<span data-ttu-id="14b96-353">Microsoft Azure 보안 시작</span><span class="sxs-lookup"><span data-stu-id="14b96-353">Getting started with Microsoft Azure security</span></span>](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [<span data-ttu-id="14b96-354">Azure Identity Management 및 액세스 제어 보안 모범 사례</span><span class="sxs-lookup"><span data-stu-id="14b96-354">Azure Identity Management and access control security best practices</span></span>](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### <a name="application-and-messaging-services"></a><span data-ttu-id="14b96-355">응용 프로그램 및 메시지 서비스</span><span class="sxs-lookup"><span data-stu-id="14b96-355">Application and messaging services</span></span>

#### <a name="simple-email-service"></a><span data-ttu-id="14b96-356">Simple Email Service</span><span class="sxs-lookup"><span data-stu-id="14b96-356">Simple Email Service</span></span>

<span data-ttu-id="14b96-357">AWS는 알림, 트랜잭션 또는 마케팅 전자 메일을 보낼 수 있는 SES(Simple Email Service)를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-357">AWS provides the Simple Email Service (SES) for sending notification, transactional, or marketing emails.</span></span> <span data-ttu-id="14b96-358">Azure에서는 [Sendgrid](https://sendgrid.com/partners/azure/) 같은 타사 솔루션이 전자 메일 서비스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-358">In Azure, third-party solutions like [Sendgrid](https://sendgrid.com/partners/azure/) provide email services.</span></span>

#### <a name="simple-queueing-service"></a><span data-ttu-id="14b96-359">Simple Queueing Service</span><span class="sxs-lookup"><span data-stu-id="14b96-359">Simple Queueing Service</span></span>

<span data-ttu-id="14b96-360">AWS SQS(AWS Simple Queueing Service)는 AWS 플랫폼 내부의 응용 프로그램, 서비스 및 장치를 연결하는 메시지 시스템을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-360">AWS Simple Queueing Service (SQS) provides a messaging system for connecting applications, services, and devices within the AWS platform.</span></span> <span data-ttu-id="14b96-361">Azure에도 비슷한 기능을 제공하는 두 가지 서비스가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-361">Azure has two services that provide similar functionality:</span></span>

-   <span data-ttu-id="14b96-362">[큐 저장소](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - Azure 플랫폼 내부의 응용 프로그램 구성 요소 간 통신을 허용하는 클라우드 메시지 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-362">[Queue storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - a cloud messaging service that allows communication between application components within the Azure platform.</span></span>

-   <span data-ttu-id="14b96-363">[Service Bus](https://azure.microsoft.com/en-us/services/service-bus/) - 응용 프로그램, 서비스 및 장치를 연결하는 보다 강력한 메시지 시스템입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-363">[Service Bus](https://azure.microsoft.com/en-us/services/service-bus/) - a more robust messaging system for connecting applications, services, and devices.</span></span> <span data-ttu-id="14b96-364">관련 [Service Bus Relay](https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it)를 사용하면 Service Bus가 원격으로 호스팅되는 응용 프로그램 및 서비스에도 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-364">Using the related [Service Bus relay](https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it), Service Bus can also connect to remotely hosted applications and services.</span></span>

#### <a name="device-farm"></a><span data-ttu-id="14b96-365">Device Farm</span><span class="sxs-lookup"><span data-stu-id="14b96-365">Device Farm</span></span>

<span data-ttu-id="14b96-366">AWS Device Farm은 장치 간 테스트 서비스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-366">The AWS Device Farm provides cross-device testing services.</span></span> <span data-ttu-id="14b96-367">Azure에서는 [Xamarin Test Cloud](https://www.xamarin.com/test-cloud)가 모바일 장치에 이와 비슷한 장치 간 프런트 엔드 테스트를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-367">In Azure, [Xamarin Test Cloud](https://www.xamarin.com/test-cloud) provides similar cross-device front-end testing for mobile devices.</span></span>

<span data-ttu-id="14b96-368">프런트 엔드 테스트 외에도 [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/)가 Linux 및 Windows 환경을 위한 백 엔드 테스트 리소스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-368">In addition to front-end testing, the [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/) provides back end testing resources for Linux and Windows environments.</span></span>

#### <a name="see-also"></a><span data-ttu-id="14b96-369">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-369">See also</span></span>

-   [<span data-ttu-id="14b96-370">Node.js에서 큐 저장소를 사용하는 방법</span><span class="sxs-lookup"><span data-stu-id="14b96-370">How to use Queue storage from Node.js</span></span>](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [<span data-ttu-id="14b96-371">Service Bus 큐를 사용하는 방법</span><span class="sxs-lookup"><span data-stu-id="14b96-371">How to use Service Bus queues</span></span>](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/)

### <a name="analytics-and-big-data"></a><span data-ttu-id="14b96-372">분석 및 빅 데이터</span><span class="sxs-lookup"><span data-stu-id="14b96-372">Analytics and big data</span></span>

<span data-ttu-id="14b96-373">[Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/)는 대량의 데이터를 캡처, 구성, 분석 및 시각화할 수 있도록 디자인된 Azure의 제품 및 서비스 패키지입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-373">[The Cortana Intelligence Suite](https://azure.microsoft.com/suites/cortana-intelligence-suite/) is Azure's package of products and services designed to capture, organize, analyze, and visualize large amounts of data.</span></span> <span data-ttu-id="14b96-374">Cortana 제품군은 다음 서비스로 구성되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-374">The Cortana suite consists of the following services:</span></span>

-   <span data-ttu-id="14b96-375">[HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - Hadoop, Spark, Storm 또는 HBase를 포함하고 있는 관리형 Apache 배포입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-375">[HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - managed Apache distribution that includes Hadoop, Spark, Storm, or HBase.</span></span>

-   <span data-ttu-id="14b96-376">[Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) - 데이터 오케스트레이션 및 데이터 파이프라인 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-376">[Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) - provides data orchestration and data pipeline functionality.</span></span>

-   <span data-ttu-id="14b96-377">[SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - 대규모 관계형 데이터 저장소입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-377">[SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - large-scale relational data storage.</span></span>

-   <span data-ttu-id="14b96-378">[Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - 빅 데이터 분석 워크로드에 최적화된 대규모 저장소입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-378">[Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) - large-scale storage optimized for big data analytics workloads.</span></span>

-   <span data-ttu-id="14b96-379">[Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/) - 데이터에 대한 예측 분석을 빌드하고 적용하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-379">[Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/) - used to build and apply predictive analytics on data.</span></span>

-   <span data-ttu-id="14b96-380">[Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/) - 실시간으로 데이터를 분석합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-380">[Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/) - real-time data analysis.</span></span>

-   <span data-ttu-id="14b96-381">[Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - Data Lake Store에 사용하도록 최적화된 대규모 분석 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-381">[Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - large-scale analytics service optimized to work with Data Lake Store</span></span>

-   <span data-ttu-id="14b96-382">[PowerBI](https://powerbi.microsoft.com/) -전원 데이터 시각화를 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-382">[PowerBI](https://powerbi.microsoft.com/) - used to power data visualization.</span></span>

#### <a name="see-also"></a><span data-ttu-id="14b96-383">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-383">See also</span></span>

-   [<span data-ttu-id="14b96-384">Cortana Intelligence 갤러리</span><span class="sxs-lookup"><span data-stu-id="14b96-384">Cortana Intelligence Gallery</span></span>](https://gallery.cortanaintelligence.com/)

-   [<span data-ttu-id="14b96-385">Microsoft 빅 데이터 솔루션의 이해</span><span class="sxs-lookup"><span data-stu-id="14b96-385">Understanding Microsoft big data solutions</span></span>](https://msdn.microsoft.com/library/dn749804.aspx)

-   [<span data-ttu-id="14b96-386">Azure Data Lake 및 Azure HDInsight 블로그</span><span class="sxs-lookup"><span data-stu-id="14b96-386">Azure Data Lake & Azure HDInsight Blog</span></span>](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a><span data-ttu-id="14b96-387">사물 인터넷</span><span class="sxs-lookup"><span data-stu-id="14b96-387">Internet of Things</span></span>

#### <a name="see-also"></a><span data-ttu-id="14b96-388">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-388">See also</span></span>

-   [<span data-ttu-id="14b96-389">Azure IoT Hub 시작</span><span class="sxs-lookup"><span data-stu-id="14b96-389">Get started with Azure IoT Hub</span></span>](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [<span data-ttu-id="14b96-390">IoT Hub 및 Event Hubs의 비교</span><span class="sxs-lookup"><span data-stu-id="14b96-390">Comparison of IoT Hub and Event Hubs</span></span>](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a><span data-ttu-id="14b96-391">모바일 서비스</span><span class="sxs-lookup"><span data-stu-id="14b96-391">Mobile services</span></span>

#### <a name="notifications"></a><span data-ttu-id="14b96-392">공지</span><span class="sxs-lookup"><span data-stu-id="14b96-392">Notifications</span></span>

<span data-ttu-id="14b96-393">Notification Hubs는 SMS 또는 전자 메일 메시지 보내기를 지원하지 않으므로 이러한 기능을 제공하는 타사 서비스가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="14b96-393">Notification Hubs do not support sending SMS or email messages, so third-party services are needed for those delivery types.</span></span>

#### <a name="see-also"></a><span data-ttu-id="14b96-394">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-394">See also</span></span>

-   [<span data-ttu-id="14b96-395">Android 앱 만들기</span><span class="sxs-lookup"><span data-stu-id="14b96-395">Create an Android app</span></span>](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [<span data-ttu-id="14b96-396">Azure Mobile Apps의 인증 및 권한 부여</span><span class="sxs-lookup"><span data-stu-id="14b96-396">Authentication and Authorization in Azure Mobile Apps</span></span>](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [<span data-ttu-id="14b96-397">Azure Notification Hubs를 사용하여 푸시 알림 보내기</span><span class="sxs-lookup"><span data-stu-id="14b96-397">Sending push notifications with Azure Notification Hubs</span></span>](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a><span data-ttu-id="14b96-398">관리 및 모니터링</span><span class="sxs-lookup"><span data-stu-id="14b96-398">Management and monitoring</span></span>

#### <a name="see-also"></a><span data-ttu-id="14b96-399">참고 항목</span><span class="sxs-lookup"><span data-stu-id="14b96-399">See also</span></span>
-   [<span data-ttu-id="14b96-400">모니터링 및 진단 지침</span><span class="sxs-lookup"><span data-stu-id="14b96-400">Monitoring and diagnostics guidance</span></span>](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [<span data-ttu-id="14b96-401">Azure Resource Manager 템플릿 생성 모범 사례</span><span class="sxs-lookup"><span data-stu-id="14b96-401">Best practices for creating Azure Resource Manager templates</span></span>](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [<span data-ttu-id="14b96-402">Azure Resource Manager 빠른 시작 템플릿</span><span class="sxs-lookup"><span data-stu-id="14b96-402">Azure Resource Manager Quickstart templates</span></span>](https://azure.microsoft.com/documentation/templates/)


## <a name="next-steps"></a><span data-ttu-id="14b96-403">다음 단계</span><span class="sxs-lookup"><span data-stu-id="14b96-403">Next steps</span></span>

-   [<span data-ttu-id="14b96-404">AWS와 Azure의 전체 서비스 비교표</span><span class="sxs-lookup"><span data-stu-id="14b96-404">Complete AWS and Azure service comparison matrix</span></span>](https://aka.ms/azure4aws-services)

-   [<span data-ttu-id="14b96-405">대화형 Azure 플랫폼 큰 그림</span><span class="sxs-lookup"><span data-stu-id="14b96-405">Interactive Azure Platform Big Picture</span></span>](http://azureplatform.azurewebsites.net/)

-   [<span data-ttu-id="14b96-406">Azure 시작</span><span class="sxs-lookup"><span data-stu-id="14b96-406">Get started with Azure</span></span>](https://azure.microsoft.com/get-started/)

-   [<span data-ttu-id="14b96-407">Azure 솔루션 아키텍처</span><span class="sxs-lookup"><span data-stu-id="14b96-407">Azure solution architectures</span></span>](https://azure.microsoft.com/solutions/architecture/)

-   [<span data-ttu-id="14b96-408">Azure 참조 아키텍처</span><span class="sxs-lookup"><span data-stu-id="14b96-408">Azure Reference Architectures</span></span>](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

-   [<span data-ttu-id="14b96-409">패턴 및 연습: Azure 지침</span><span class="sxs-lookup"><span data-stu-id="14b96-409">Patterns & Practices: Azure Guidance</span></span>](https://azure.microsoft.com/documentation/articles/guidance/)

-   [<span data-ttu-id="14b96-410">무료 온라인 강좌: AWS 전문가를 위한 Microsoft Azure</span><span class="sxs-lookup"><span data-stu-id="14b96-410">Free Online Course: Microsoft Azure for AWS Experts</span></span>](http://aka.ms/azureforaws)


<!-- links -->

[paired-regions]: https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/
[traffic-manager]: /azure/traffic-manager/