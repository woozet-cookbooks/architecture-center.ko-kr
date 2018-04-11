---
title: '지침: Azure AD 테넌트 디자인'
description: 기본 클라우드 채택 전략의 일환으로 고려되는 Azure 테넌트 디자인 지침
author: telmosampaio
ms.openlocfilehash: 9ac52e9fd44bd8b9c777625002d5960f4f269be2
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/27/2018
---
# <a name="guidance-azure-ad-tenant-design"></a>지침: Azure AD 테넌트 디자인

Azure AD 테넌트는 하나 이상의 [Azure 구독](subscription-explainer.md)에 사용되는 디지털 ID 서비스 및 네임스페이스를 제공합니다. 기본 채택 개요를 따르고 있다면 [Azure AD 테넌트를 가져오는 방법][how-to-get-aad-tenant]을 이미 배웠을 것입니다. 

## <a name="design-considerations"></a>디자인 고려 사항

- 기본 채택 단계에서는 단일 Azure AD 테넌트로 시작할 수 있습니다. 조직에 기존 Office 365 구독 또는 Azure 구독이 있는 경우 사용할 수 있는 Azure AD 테넌트가 이미 있는 것입니다. 이러한 테넌트 중 하나 또는 둘 다 없는 경우 [Azure AD 테넌트를 가져오는 방법][how-to-get-aad-tenant]에 대해 알아볼 수 있습니다. 
- 중급 및 고급 채택 단계에서 온-프레미스 디렉터리를 Azure AD와 동기화하거나 페더레이션하는 방법을 알아봅니다. 이렇게 하면 Azure AD에서 온-프레미스 디지털 ID를 사용할 수 있습니다. 그러나 기본 단계에서 단일 Azure AD 테넌트에 ID가 있는 새 사용자만 추가하게 됩니다. 해당 ID는 귀하가 직접 관리해야 합니다. 예를 들어, 새로운 Azure AD 사용자를 온보딩하고, 더 이상 Azure 리소스 및 사용자 권한에 대한 기타 변경 내용에 액세스하지 않으려는 Azure AD 사용자는 오프보딩해야 합니다.

## <a name="next-steps"></a>다음 단계

* 이제 Azure AD 테넌트가 있으므로 [사용자를 추가하는 방법][azure-ad-add-user]을 알아봅니다. Azure AD 테넌트에 하나 이상의 새 사용자를 추가한 후에는 [Azure 구독](subscription-explainer.md)에 대해 알아봅니다.

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
