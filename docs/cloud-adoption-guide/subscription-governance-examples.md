---
title: 구독 관리에 대한 시나리오 및 예제 | Microsoft Docs
description: 일반적인 시나리오에서 Azure 구독 관리를 구현하는 방법에 대한 예제를 제공합니다.
services: azure-resource-manager
documentationcenter: na
author: rdendtler
manager: timlt
editor: tysonn
ms.assetid: e8fbeeb8-d7a1-48af-804f-6fe1a6024bcb
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/03/2017
ms.author: rodend;karlku;tomfitz
ms.openlocfilehash: 4a4b25e6a6e54fd18c8ae21a88cab39143c3bf48
ms.sourcegitcommit: 4ec010846b9b5545c843a32e08293f906e512302
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/17/2018
ms.locfileid: "34299911"
---
# <a name="examples-of-implementing-azure-enterprise-scaffold"></a>Azure 엔터프라이즈 스캐폴드 구현 예제
이 문서에서는 기업에서 [Azure 엔터프라이즈 스캐폴드](subscription-governance.md)에 대한 권장 사항을 구현하는 방법에 대한 예를 제공합니다. Contoso라는 가상의 회사를 사용하여 일반적인 시나리오의 모범 사례를 보여 줍니다.

## <a name="background"></a>백그라운드
Contoso는 고객을 위해 공급망 솔루션을 제공하는 세계적인 회사입니다. 그들은 Service 모델인 소프트웨어에서 온 - 프레미스에 배포된 패키지 모델에 이르기까지 모든 것을 제공합니다.  인도, 미국 및 캐나다에 중요한 개발 센터를 갖추고 전 세계적으로 소프트웨어를 개발합니다.

회사의 ISV 부분은 중요한 비즈니스의 제품을 관리하는 여러 독립적인 사업부로 나뉩니다. 각 사업부에는 고유한 개발자, 제품 관리자 및 설계자가 있습니다.

엔터프라이즈 기술 서비스(ETS) 사업부에서는 중앙 집중화된 IT 기능을 제공하고 사업부에서 응용 프로그램을 호스팅하는 여러 데이터 센터를 관리합니다. ETS 조직은 데이터 센터를 관리하는 것과 더불어, 중앙 집중화된 공동 작업(예: 전자 메일 및 웹 사이트) 및 네트워크/전화 통신 서비스를 제공하고 관리합니다. 또한 운영 담당자가 없는 소규모 사업부를 위한 고객 관련 워크로드도 관리합니다.

이 문서에서는 다음과 같은 가상 사용자가 사용됩니다.

* Dave는 ETS Azure 관리자입니다.
* Alice는 공급망 사업부의 개발 이사입니다.

Contoso는 LOB(기간 업무) 앱과 고객에게 표시되는 앱을 작성해야 합니다. Azure에서 앱을 실행하기로 결정했습니다. Dave는 [규범적 구독 거버넌스](subscription-governance.md) 아티클을 읽었고 권장 사항을 구현할 준비가 되었습니다.

## <a name="scenario-1-line-of-business-application"></a>시나리오 1: 기간 업무 응용 프로그램
Contoso는 전 세계 개발자가 사용할 소스 코드 관리 시스템(BitBucket)을 구축 중입니다.  응용 프로그램에서는 호스팅에 IaaS(Infrastructure as a Service)를 사용하는데, 이는 웹 서버와 데이터베이스 서버로 구성됩니다. 개발자는 자신의 개발 환경에서 서버에 액세스하지만 Azure에서 서버에 액세스할 필요는 없습니다. Contoso ETS는 응용 프로그램 소유자와 팀에서 응용 프로그램을 관리할 수 있도록 하려고 합니다. 응용 프로그램은 Contoso의 회사 네트워크에 있는 상태에서만 사용 가능합니다. Dave는 이 응용 프로그램에 대한 구독을 설정해야 합니다. 구독은 향후 다른 개발자 관련 소프트웨어도 호스트합니다.  

### <a name="naming-standards--resource-groups"></a>이름 지정 표준 및 리소스 그룹
Dave는 모든 사업부 간에 일반적인 개발자 도구를 지원하는 구독을 만듭니다. Dave는 구독과 리소스 그룹(응용 프로그램 및 네트워크용)에 대해 의미 있는 이름을 만들어야 합니다. 다음 구독 및 리소스 그룹을 만듭니다.

| 항목 | Name | 설명 |
| --- | --- | --- |
| 구독 |Contoso ETS DeveloperTools Production |일반적인 개발자 도구 지원 |
| 리소스 그룹 |bitbucket-prod-rg |응용 프로그램 웹 서버 및 데이터베이스 서버 포함 |
| 리소스 그룹 |corenetworks-prod-rg |가상 네트워크 및 사이트 간 게이트웨이 연결 포함 |

### <a name="role-based-access-control"></a>역할 기반 액세스 제어
Dave는 구독을 만든 후 적절한 팀 및 응용 프로그램 소유자가 자신의 리소스에 액세스할 수 있는지 확인하려고 합니다. Dave는 팀마다 요구 사항이 서로 다르다는 것을 알게 됩니다. 그는 Contoso의 온-프레미스 AD(Active Directory)에서 Azure Active Directory로 동기화된 그룹을 활용하고 팀에 적합한 액세스 수준을 제공합니다.

Dave는 구독에 대해 다음 역할을 할당합니다.

| 역할 | 할당 대상 | 설명 |
| --- | --- | --- |
| [소유자](/azure/role-based-access-control/built-in-roles#owner) |Contoso AD에서 관리되는 ID |이 ID는 Contoso의 ID 관리 도구를 통해 JIT(Just-in-Time) 액세스로 제어되며, 구독 소유자 액세스를 완전히 감사할 수 있도록 합니다. |
| [보안 판독기](/azure/role-based-access-control/built-in-roles#security-reader) |보안 및 위험 관리 부서 |이 역할을 통해 사용자는 Azure Security Center와 리소스 상태를 살펴볼 수 있습니다. |
| [네트워크 기여자](/azure/role-based-access-control/built-in-roles#network-contributor) |네트워크 팀 |이 역할을 통해 Contoso 네트워크 팀에서 사이트 간 VPN 및 Virtual Network를 관리할 수 있습니다. |
| *사용자 지정 역할* |응용 프로그램 소유자 |Dave는 리소스 그룹 내에서 리소스를 수정하는 기능이 부여된 역할을 만듭니다. 자세한 내용은 [Azure RBAC에서 사용자 지정 역할](/azure/role-based-access-control/custom-roles)을 참조하세요. |

### <a name="policies"></a>정책
Dave는 구독에서 리소스 관리를 위해 다음을 요구 사항으로 포함합니다.

* 개발 도구는 전 세계 개발자는 지원하므로 사용자가 어느 지역에서든 리소스를 만들 수 있도록 하고 싶습니다. 하지만 리소스가 생성되는 위치를 알아야 합니다.
* 그는 비용에 대해 고민합니다. 따라서 응용 프로그램 소유자가 불필요하게 비용이 드는 가상 머신을 만들지 않도록 하려고 합니다.  
* 이 응용 프로그램은 많은 사업부의 개발자에게 제공되므로 각 리소스에 사업부 및 응용 프로그램 소유자로 태그를 지정하고자 합니다. ETS는 다음 태그를 사용하여 적절한 팀에 요금을 청구할 수 있습니다.

그는 다음과 같은 [Azure 정책](/azure/azure-policy/azure-policy-introduction)을 만듭니다.

| 필드 | 결과 | 설명 |
| --- | --- | --- |
| 위치 |감사 |모든 지역에서 리소스 생성 감사 |
| 형식 |deny |G-시리즈 가상 머신 생성 거부 |
| tags |deny |응용 프로그램 소유자 태그 필요 |
| tags |deny |비용 센터 태그 필요 |
| tags |추가 |태그 이름 **BusinessUnit** 및 태그 값 **ETS**를 모든 리소스에 추가 |

### <a name="resource-tags"></a>리소스 태그
Dave는 청구서에 BitBucket 구현에 대한 비용 센터를 식별할 수 있는 특정 정보가 있어야 한다는 것을 알게 됩니다. 또한 ETS에서 소유한 모든 리소스를 알고자 합니다.

다음 [태그](/azure/azure-resource-manager/resource-group-using-tags)를 리소스 그룹 및 리소스에 추가합니다.

| 태그 이름 | 태그 값 |
| --- | --- |
| ApplicationOwner |이 응용 프로그램을 관리하는 사용자의 이름입니다. |
| CostCenter |Azure 사용에 대해 지불하는 그룹의 비용 센터입니다. |
| BusinessUnit |**ETS**(구독과 연결된 사업부) |

### <a name="core-network"></a>핵심 네트워크
Contoso ETS 정보 보안 및 위험 관리 팀은 응용 프로그램을 Azure로 이동하기 위해 Dave가 제안한 계획을 검토합니다. 그들은 응용 프로그램이 인터넷에 노출되지 않기를 원합니다.  Dave는 향후 Azure로 이동할 개발자 앱도 보유하고 있습니다. 이러한 앱에는 공용 인터페이스가 필요합니다.  이러한 요구 사항을 충족하기 위해 내부 및 외부 가상 네트워크와 액세스를 제한할 네트워크 보안 그룹을 제공합니다.

그는 다음 리소스를 만듭니다.

| 리소스 종류 | Name | 설명 |
| --- | --- | --- |
| Virtual Network |internal-vnet |BitBucket 응용 프로그램에 사용되며 ExpressRoute를 통해 Contoso의 회사 네트워크에 연결됩니다.  서브넷(`bitbucket`)에서 응용 프로그램에 특정 IP 주소 공간을 제공합니다. |
| Virtual Network |external-vnet |공용 끝점이 필요한 미래의 응용 프로그램에 사용할 수 있습니다. |
| 네트워크 보안 그룹 |bitbucket-nsg |응용 프로그램이 있는 서브넷(`bitbucket`)에 대한 443 포트에서만 연결을 허용하여 이 워크로드의 공격에 대한 취약성이 최소화되도록 합니다. |

### <a name="resource-locks"></a>리소스 잠금
Dave는 난해한 스크립트나 실수로 인한 삭제로부터 Contoso 회사 네트워크에서 내부 가상 네트워크로의 연결을 보호해야 한다는 것을 알게 됩니다.

그는 다음 [리소스 잠금](/azure/azure-resource-manager/resource-group-lock-resources)을 만듭니다.

| 잠금 유형 | 리소스 | 설명 |
| --- | --- | --- |
| **CanNotDelete** |internal-vnet |사용자가 가상 네트워크 또는 서브넷을 삭제할 수 없도록 하지만 새 서브넷은 추가할 수 있도록 합니다. |

### <a name="azure-automation"></a>Azure Automation
Dave는 이 응용 프로그램을 자동화할 일이 없습니다. Azure Automation 계정을 만들기는 했지만 아예 사용하지 않습니다.

### <a name="azure-security-center"></a>Azure Security Center
Contoso IT 서비스 관리를 위해서는 위협을 신속하게 파악하고 처리해야 합니다. 또한 어떤 문제가 존재할 수 있는지도 파악하려고 합니다.  

이러한 요구 사항을 충족하기 위해 Dave는 [Azure Security Center](/azure/security-center/security-center-intro)를 사용하도록 설정하고 Security Reader 역할에 대한 액세스를 제공합니다.

## <a name="scenario-2-customer-facing-app"></a>시나리오 2: 고객에게 표시되는 앱
공급망 사업부의 비즈니스 리더십은 로열티 카드를 사용하여 Contoso 고객 참여를 확대하는 다양한 기회를 파악했습니다. Alice 팀은 이 응용 프로그램을 만들고 Azure가 비즈니스 요구를 충족하는 기능을 향상시키는지 확인해야 합니다. ETS의 Dave와 협력하여 이 응용 프로그램의 개발 및 작동을 위한 두 가지 구독을 구성합니다.

### <a name="azure-subscriptions"></a>Azure 구독
Dave는 Azure Enterprise Portal에 로그인하여 공급망 부서가 이미 있는지 확인합니다.  하지만 이 프로젝트는 Azure에서 공급망 팀을 위한 첫 번째 개발 프로젝트이므로 Dave는 Alice 개발 팀을 위해 새 계정이 필요하다는 것을 알게 됩니다.  Alice 팀을 위한 "R&D" 계정을 만들고 Alice에게 액세스 권한을 할당해 줍니다. Alice는 Azure Portal을 통해 로그인하고 두 개의 구독을 만듭니다. 하나는 개발 서버를 보유하고 하나는 프로덕션 서버를 보유합니다.  다음 구독을 만들 때 이전에 설정한 이름 지정 표준을 따릅니다.

| 구독 용도 | Name |
| --- | --- |
| 개발 |Contoso SupplyChain ResearchDevelopment LoyaltyCard Development |
| 프로덕션 |Contoso SupplyChain Operations LoyaltyCard Production |

### <a name="policies"></a>정책
Dave와 Alice는 응용 프로그램에 대해 논의하고 이 응용 프로그램이 북아메리카 지역의 고객에게만 서비스되는지 확인합니다.  Alice와 팀은 응용 프로그램을 만들기 위해 Azure의 응용 프로그램 서비스 환경과 Azure SQL을 사용하기로 합니다. 개발 중에 가상 머신을 만들어야 할 수 있습니다.  Alice는 개발자에게 ETS 지원 없이 문제를 탐색 및 확인하는 데 필요한 리소스가 있는지 확인합니다.

**개발 구독**을 위해 다음 정책을 만듭니다.

| 필드 | 결과 | 설명 |
| --- | --- | --- |
| 위치 |감사 |모든 지역에서 리소스 생성 감사 |

개발 중에 사용자가 만들 수 있는 sku 유형을 제한하지 않으며 리소스 그룹 또는 리소스에 태그가 필요하지 않습니다.

**프로덕션 구독**을 위해 다음 정책을 만듭니다.

| 필드 | 결과 | 설명 |
| --- | --- | --- |
| location |deny |미국 데이터 센터 외부에서 리소스를 만드는 것을 모두 거부합니다. |
| tags |deny |응용 프로그램 소유자 태그 필요 |
| tags |deny |부서 태그가 필요합니다. |
| tags |추가 |프로덕션 환경을 나타내는 각 리소스 그룹에 태그를 추가합니다. |

개발 중에 사용자가 만들 수 있는 sku 유형을 제한하지 않으며 리소스 그룹 또는 리소스에 태그가 필요하지 않습니다.

### <a name="resource-tags"></a>리소스 태그
Dave는 청구 및 소유권에 대한 정확한 비즈니스 그룹을 식별할 수 있는 특정 정보가 있어야 한다는 것을 알게 됩니다. 리소스 그룹 및 리소스에 대한 리소스 태그를 정의합니다.

| 태그 이름 | 태그 값 |
| --- | --- |
| ApplicationOwner |이 응용 프로그램을 관리하는 사용자의 이름입니다. |
| department |Azure 사용에 대해 지불하는 그룹의 비용 센터입니다. |
| EnvironmentType |**Production**(구독 이름에 **Production**이 포함되어 있지만, 이 태그를 포함하면 포털 또는 청구서에서 리소스를 확인할 때 쉽게 식별할 수 있습니다.) |

### <a name="core-networks"></a>코어 네트워크
Contoso ETS 정보 보안 및 위험 관리 팀은 응용 프로그램을 Azure로 이동하기 위해 Dave가 제안한 계획을 검토합니다. 로열티 카드 응용 프로그램이 DMZ 네트워크에서 제대로 격리 및 보호되는지 확인하려고 합니다.  이 요구 사항을 충족시키기 위해 Dave와 Alice는 외부 가상 네트워크 및 네트워크 보안 그룹을 만들어 로열티 카드 응용 프로그램을 Contoso 회사 네트워크와 격리합니다.  

**개발 구독**을 위해 다음을 만듭니다.

| 리소스 종류 | Name | 설명 |
| --- | --- | --- |
| Virtual Network |internal-vnet |Contoso 로열티 카드 개발 환경을 제공하고 ExpressRoute를 통해 Contoso의 회사 네트워크에 연결됩니다. |

**프로덕션 구독**을 위해 다음을 만듭니다.

| 리소스 종류 | Name | 설명 |
| --- | --- | --- |
| Virtual Network |external-vnet |로열티 카드 응용 프로그램을 호스트하고 Contoso의 ExpressRoute에 직접 연결되지 않습니다. 코드는 소스 코드 시스템을 통해 PaaS 서비스에 직접 푸시됩니다. |
| 네트워크 보안 그룹 |loyaltycard-nsg |TCP 443에서 인바운드 통신만 허용하여 이 워크로드의 공격 취약성이 최소화되도록 합니다.  또한 Contoso는 웹 응용 프로그램 방화벽을 사용하여 추가 보호를 조사하고 있습니다. |

### <a name="resource-locks"></a>리소스 잠금
Dave와 Alice는 서로 상의한 후 환경에서 일부 주요 리소스에 리소스 잠금을 추가하여 잘못된 코드 푸시 중에 실수로 삭제되는 일이 없도록 합니다.

다음 잠금을 만듭니다.

| 잠금 유형 | 리소스 | 설명 |
| --- | --- | --- |
| **CanNotDelete** |external-vnet |사용자가 가상 네트워크 또는 서브넷을 삭제하지 않도록 합니다. 잠금은 새 서브넷 추가를 차단하지 않습니다. |

### <a name="azure-automation"></a>Azure Automation
Alice와 개발 팀은 이 응용 프로그램에 대한 환경을 관리할 수 있는 광범위한 runbook을 포함합니다. runbook은 응용 프로그램 및 기타 DevOps 작업에 대한 노드의 추가/삭제를 허용합니다.

이러한 runbook을 사용하려면 [Automation](/azure/automation/automation-intro)을 사용하도록 설정합니다.

### <a name="azure-security-center"></a>Azure Security Center
Contoso IT 서비스 관리를 위해서는 위협을 신속하게 파악하고 처리해야 합니다. 또한 어떤 문제가 존재할 수 있는지도 파악하려고 합니다.  

이러한 요구 사항을 충족하기 위해 Dave는 Azure Security Center를 사용하도록 설정합니다. 그는 Azure Security Center에서 리소스를 모니터링하고 있고 DevOps 및 보안 팀에 대한 액세스를 제공하는지 확인합니다.

## <a name="next-steps"></a>다음 단계
* Resource Manager 템플릿 작성에 대해 알아보려면 [Azure Resource Manager 템플릿 생성 모범 사례](/azure/azure-resource-manager/resource-manager-template-best-practices)를 참조하세요.
