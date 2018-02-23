---
title: "설명: Azure 구독이란?"
description: "Azure 구독, 계정 및 제공 서비스 설명"
author: alexbuckgit
ms.openlocfilehash: 1650d90d6f78b46b7fe4128d2dab6a80bd6cca78
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/23/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a><span data-ttu-id="e471e-103">설명: Azure 구독이란?</span><span class="sxs-lookup"><span data-stu-id="e471e-103">Explainer: What is an Azure subscription?</span></span>

<span data-ttu-id="e471e-104">[Azure Active Directory 테넌트란?](tenant-explainer.md) 설명 문서에서는 조직의 디지털 ID가 Azure Active Directory 테넌트에 저장된다는 사실을 배웠습니다.</span><span class="sxs-lookup"><span data-stu-id="e471e-104">In the [what is an Azure Active Directory tenant?](tenant-explainer.md) explainer article, you learned that digital identity for your organization is stored in an Azure Active Directory tenant.</span></span> <span data-ttu-id="e471e-105">또한 Azure에서 Azure Active Directory를 신뢰하여 리소스를 만들거나, 읽거나, 업데이트하거나, 삭제하기 위한 사용자 요청을 인증한다는 사실도 확인했습니다.</span><span class="sxs-lookup"><span data-stu-id="e471e-105">You also learned that Azure trusts Azure Active Directory to authenticate user requests to create, read, update, or delete a resource.</span></span> 

<span data-ttu-id="e471e-106">기본적으로 인증되지 않은 권한 없는 사용자가 리소스에 액세스하지 못하도록 하기 위해, 리소스에 대해 이러한 작업의 액세스를 제한해야 하는 이유를 기본적으로 이해하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e471e-106">We fundamentally understand why it's necessary to restrict access to these operations on a resource - to prevent unauthenticated and unauthorized users from accessing our resources.</span></span> <span data-ttu-id="e471e-107">그러나 이러한 리소스 작업에는 사용자 또는 사용자 그룹이 만들도록 허용되는 리소스의 수 및 해당 리소스의 실행 비용과 같이 제어하려는 기타 속성이 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="e471e-107">However, these resource operations have other properties that an organization would like to control, such as the number of resources a user or group of users is allowed to create, and, the cost to run those resources.</span></span> 

<span data-ttu-id="e471e-108">Azure는 이러한 제어를 구현하며 이를 **구독**이라고 합니다.</span><span class="sxs-lookup"><span data-stu-id="e471e-108">Azure implements this control, and it is named a **subscription**.</span></span> <span data-ttu-id="e471e-109">구독 그룹은 사용자 및 해당 사용자가 만든 리소스를 함께 그룹화합니다.</span><span class="sxs-lookup"><span data-stu-id="e471e-109">A subscription groups together users and the resources that have been created by those users.</span></span> <span data-ttu-id="e471e-110">이러한 각 리소스는 해당 특정 리소스의 [전체 한도][subscription-service-limits]에 영향을 미칩니다.</span><span class="sxs-lookup"><span data-stu-id="e471e-110">Each of those resources contributes to an [overall limit][subscription-service-limits] on that particular resource.</span></span>

<span data-ttu-id="e471e-111">조직에서는 구독을 사용하여 사용자, 팀, 프로젝트별로 리소스의 비용 및 생성을 관리하거나 기타 다양한 전략을 사용해서 이러한 관리를 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e471e-111">Organizations can use subscriptions to manage costs and creation of resource by users, teams, projects, or using many other strategies.</span></span> <span data-ttu-id="e471e-112">이러한 전략은 중급 및 고급 채택 단계 문서에 설명되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e471e-112">These strategies will be discussed in the intermediate and advanced adoption stage articles.</span></span> 

## <a name="next-steps"></a><span data-ttu-id="e471e-113">다음 단계</span><span class="sxs-lookup"><span data-stu-id="e471e-113">Next steps</span></span>

* <span data-ttu-id="e471e-114">지금까지 Azure 구독에 대해 배웠습니다. 이제 첫 번째 Azure 리소스를 만들기 전에 [구독 만들기](subscription.md)에 대해 자세히 알아보세요.</span><span class="sxs-lookup"><span data-stu-id="e471e-114">Now that you have learned about Azure subscriptions, learn more about [creating a subscription](subscription.md) before you create your first Azure resources..</span></span>

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/get-started/
[azure-offers]: https://azure.microsoft.com/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/azure/active-directory/sign-up-organization
