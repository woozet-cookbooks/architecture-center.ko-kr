---
title: "설명: Azure 구독이란?"
description: "Azure 구독, 계정 및 제공 서비스 설명"
author: abuck
ms.openlocfilehash: 7b88e9489b40b100eecb76602b45901566b3f37f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-subscription"></a>설명: Azure 구독이란?

[Azure Active Directory 테넌트란?](tenant-explainer.md) 설명 문서에서는 조직의 디지털 ID가 Azure Active Directory 테넌트에 저장된다는 사실을 배웠습니다. 또한 Azure에서 Azure Active Directory를 신뢰하여 리소스를 만들거나, 읽거나, 업데이트하거나, 삭제하기 위한 사용자 요청을 인증한다는 사실도 확인했습니다. 

기본적으로 인증되지 않은 권한 없는 사용자가 리소스에 액세스하지 못하도록 하기 위해, 리소스에 대해 이러한 작업의 액세스를 제한해야 하는 이유를 기본적으로 이해하고 있습니다. 그러나 이러한 리소스 작업에는 사용자 또는 사용자 그룹이 만들도록 허용되는 리소스의 수 및 해당 리소스의 실행 비용과 같이 제어하려는 기타 속성이 적용됩니다. 

Azure는 이러한 제어를 구현하며 이를 **구독**이라고 합니다. 구독 그룹은 사용자 및 해당 사용자가 만든 리소스를 함께 그룹화합니다. 이러한 각 리소스는 해당 특정 리소스의 [전체 한도][subscription-service-limits]에 영향을 미칩니다.

조직에서는 구독을 사용하여 사용자, 팀, 프로젝트별로 리소스의 비용 및 생성을 관리하거나 기타 다양한 전략을 사용해서 이러한 관리를 수행할 수 있습니다. 이러한 전략은 중급 및 고급 채택 단계 문서에 설명되어 있습니다. 

## <a name="next-steps"></a>다음 단계

* 지금까지 Azure 구독에 대해 배웠습니다. 이제 첫 번째 Azure 리소스를 만들기 전에 [구독 만들기](subscription.md)에 대해 자세히 알아보세요.

<!-- Links -->
[azure-get-started]: https://azure.microsoft.com/en-us/get-started/
[azure-offers]: https://azure.microsoft.com/en-us/support/legal/offer-details/
[azure-free-trial]: https://azure.microsoft.com/en-us/offers/ms-azr-0044p/
[azure-change-subscription-offer]: /azure/billing/billing-how-to-switch-azure-offer
[microsoft-account]: https://account.microsoft.com/account
[subscription-service-limits]: /azure/azure-subscription-service-limits
[docs-organizational-account]: https://docs.microsoft.com/en-us/azure/active-directory/sign-up-organization
