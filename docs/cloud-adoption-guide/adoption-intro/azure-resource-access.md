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
# <a name="understanding-resource-access-management-in-azure"></a>Azure에서 리소스 액세스 관리 이해

[리소스 거버넌스란?](governance-explainer.md)에서 거버넌스가 조직의 요구 사항 및 목표 달성을 위해 Azure 리소스의 사용을 관리, 모니터링 및 감사하는 지속적인 프로세스를 가리킨다는 점을 알아보았습니다. 거버넌스 모델을 디자인하는 방법을 알아보기 위해 이동하려면 먼저 Azure에서 리소스 액세스 관리 제어를 이해해야 합니다. 이러한 리소스 액세스 관리 제어의 구성은 거버넌스 모델의 기본을 형성합니다.

Azure에 리소스를 배포하는 방법에 대해 좀 더 자세히 살펴보겠습니다. 

## <a name="what-is-an-azure-resource"></a>Azure 리소스란?

Azure에서 **리소스**라는 용어는 Azure에서 관리하는 엔터티를 가리킵니다. 예를 들어 가상 머신, 가상 네트워크 및 저장소 계정은 모두 Azure 리소스를 가리킵니다.

![](../_images/governance-1-9.png)   
*그림 1. 리소스*

## <a name="what-is-an-azure-resource-group"></a>Azure 리소스 그룹이란?

Azure의 각 리소스는 [리소스 그룹](/azure/azure-resource-manager/resource-group-overview#resource-groups)에 속해야 합니다. 리소스 그룹은 단순히 단일 엔터티로 관리될 수 있도록 여러 리소스를 그룹화하는 논리적 구문입니다. 예를 들어 [n 계층 응용 프로그램](/azure/architecture/guide/architecture-styles/n-tier)에 대한 리소스와 같은 비슷한 수명 주기를 공유하는 리소스를 그룹으로 만들거나 삭제할 수 있습니다. 

![](../_images/governance-1-10.png)   
*그림 2. 리소스 그룹에는 리소스가 포함됩니다.* 

리소스 그룹 및 여기 포함된 리소스는 Azure **구독**과 연결됩니다. 

## <a name="what-is-an-azure-subscription"></a>Azure 구독이란?

Azure 구독은 리소스 그룹 및 해당 리소스를 함께 그룹화하는 논리적 구문에서 리소스 그룹과 비슷합니다. 그러나 Azure 구독은 Azure Resource Manager에서 사용한 컨트롤과도 연결됩니다. 무슨 의미인가요? 이와 Azure 구독 간의 관계에 대해 자세히 알아보기 위해 Azure Resource Manager에 대해 더 자세히 살펴보겠습니다.

![](../_images/governance-1-11.png)   
*그림 3. Azure 구독.*

## <a name="what-is-azure-resource-manager"></a>Azure Resource Manager란?

[Azure의 작동 방식](azure-explainer.md)에서 Azure에는 Azure의 모든 기능을 오케스트레이션하는 여러 서비스를 사용하는 "프런트 엔드"가 포함된다는 사실을 알아보았습니다. [Azure Resource Manager](/azure/azure-resource-manager/)는 이러한 서비스 중 하나이며 이 서비스는 리소스를 관리하기 위해 클라이언트에서 사용하는 RESTful API를 호스트합니다. 

![](../_images/governance-1-12.png)   
*그림 4. Azure Resource Manager.*

다음 그림은 [PowerShell](/powershell/azure/overview), [Azure Portal](https://portal.azure.com) 및 [Azure CLI(명령줄 인터페이스)](/cli/azure)라는 세 가지 클라이언트를 보여줍니다.

![](../_images/governance-1-13.png)   
*그림 5. Azure Resource Manager RESTful API에 대한 Azure 클라이언트 연결.*

이러한 클라이언트가 RESTful API를 사용하여 Azure Resource Manager에 연결하는 반면 Azure Resource Manager에는 리소스를 직접 관리하는 기능이 포함되지 않습니다. 대신 Azure에서 대부분의 리소스 종류에는 고유한 [**리소스 공급자**](/azure/azure-resource-manager/resource-group-overview#terminology)가 포함됩니다. 

![](../_images/governance-1-14.png)   
*그림 6. Azure 리소스 공급자.*

클라이언트가 특정 리소스를 관리하도록 요청하면 Azure Resource Manager는 요청을 완료하기 위해 해당 리소스 종류에 대한 리소스 공급자에 연결합니다. 예를 들어 클라이언트가 가상 머신 리소스를 관리하도록 요청하는 경우 Azure Resource Manager는 **Microsoft.compute** 리소스 공급자에 연결합니다. 

![](../_images/governance-1-15.png)   
*그림 7. Azure Resource Manager는 **Microsoft.compute** 리소스 공급자에 연결하여 클라이언트 요청에 지정된 리소스를 관리합니다.*

해당 Azure Resource Manager는 클라이언트가 가상 머신 리소스를 관리하기 위해 구독 및 리소스 그룹 모두에 대한 식별자를 지정하도록 요구합니다. 

이제 Azure Resource Manager 작동 방식을 이해했으므로 Azure Resource Manager에서 사용하는 컨트롤과 Azure 구독을 연결하는 방법을 다시 살펴보겠습니다. Azure Resource Manager가 리소스 관리 요청을 실행하기 전에 컨트롤의 집합을 확인합니다. 

첫 번째 컨트롤은 유효성이 검사된 사용자가 요청해야 하고 Azure Resource Manager에게는 사용자 ID 기능을 제공하기 위해 [Azure AD(Azure Active Directory)](/azure/active-directory/)를 사용하는 신뢰할 수 있는 관계가 있습니다.

![](../_images/governance-1-16.png)   
*그림 8. Azure Active Directory.*

Azure AD에서 사용자는 **테넌트**로 조각화됩니다. 테넌트는 일반적으로 조직과 연결된 Azure AD의 안전한 전용 인스턴스를 나타내는 논리적 구문입니다. 각 구독은 Azure AD 테넌트와 연결됩니다.

![](../_images/governance-1-17.png)   
*그림 9. 구독과 연결된 Azure AD 테넌트.*

특정 구독에서 리소스를 관리하는 각 클라이언트 요청에는 연결된 Azure AD 테넌트에 사용자가 있어야 합니다. 

다음 컨트롤은 사용자에게 요청할 충분한 권한이 있는지 확인합니다. [RBAC(역할 기반 액세스 제어)](/azure/role-based-access-control/)를 사용하여 사용자에 대한 보안 권한을 구성합니다.

![](../_images/governance-1-18.png)   
*그림 10. 테넌트의 각 사용자에게는 하나 이상의 RBAC 역할이 할당됩니다.*

RBAC 역할은 사용자가 특정 리소스에 대해 수행할 수 있는 일련의 사용 권한을 지정합니다. 역할을 사용자에게 할당하면 해당 사용 권한이 적용됩니다. 예를 들어 [기본 제공 **소유자** 역할](/azure/role-based-access-control/built-in-roles#owner)을 사용하면 사용자가 리소스에서 모든 작업을 수행할 수 있습니다.

다음 컨트롤은 [Azure 리소스 정책](/azure/azure-policy/)에 지정된 설정에서 요청이 허용되는지를 검사합니다. Azure 리소스 정책은 특정 리소스에 허용되는 작업을 지정합니다. 예를 들어 Azure 리소스 정책은 사용자가 특정 형식의 가상 머신만을 배포할 수 있도록 지정할 수 있습니다.

![](../_images/governance-1-19.png)   
*그림 11. Azure 리소스 정책.*

다음 컨트롤은 요청이 [Azure 구독 제한](/azure/azure-subscription-service-limits)을 초과하지 않는지 검사합니다. 예를 들어 각 구독에는 구독당 980개의 리소스 그룹이라는 제한이 있습니다. 제한에 도달하면 추가 리소스 그룹을 배포하는 요청이 수신된 경우 해당 요청이 거부됩니다.

![](../_images/governance-1-20.png)   
*그림 12. Azure 리소스 제한.* 

최종 컨트롤은 요청이 구독과 연결된 재무 약정 내에 있는지를 검사합니다. 예를 들어 요청이 가상 머신을 배포하는 것인 경우 Azure Resource Manager는 구독에 충분한 결제 정보가 있는지 확인합니다.

![](../_images/governance-1-21.png)   
*그림 13. 재무 약정이 구독과 연결됩니다.*

# <a name="summary"></a>요약

이 아티클에서는 Azure Resource Manager를 사용하여 Azure에서 리소스 액세스를 관리하는 방법에 대해 알아보았습니다.

# <a name="next-steps"></a>다음 단계

이제 Azure에서 리소스 액세스를 관리하는 방법을 이해했으므로 이러한 서비스를 사용하여 [거버넌스 모델을 디자인](governance-how-to.md)하는 방법을 알아보겠습니다.