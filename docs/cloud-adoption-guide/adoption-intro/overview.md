---
title: 'Azure 채택: 기본'
description: 엔터프라이즈가 Azure를 채택하기 위해 필요한 기본 지식 수준 설명
author: petertay
ms.openlocfilehash: b5a0a4a2c4ed1d06c97774b0eca643a89a5a2110
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229170"
---
# <a name="adopting-microsoft-azure-foundational"></a>Microsoft Azure 채택: 기초

클라우드 기술이 새로 추가된 조직의 경우 해당 채택 과정을 시작할 시점을 결정하기 어려울 수 있습니다. 기초 채택 단계는 시작점을 제공하기 위한 것입니다. 조직 내의 개인이 이 단계를 통해 작업했다면 간단한 워크로드에 대한 계산 리소스를 Azure에 배포하는 데 필요한 모든 정보 및 기술이 있는 것입니다. 

> [!NOTE]
> 이 가이드는 응용 프로그램 개발을 다루지 않습니다. Azure에서 응용 프로그램을 개발하는 방법에 대한 자세한 내용은 [Azure 응용 프로그램 아키텍처 가이드](/azure/architecture/guide/)를 참조하세요.

가이드의 이 단계에 대한 대상은 다음과 같은 조직 내의 가상 사용자입니다.

- *재무:* Azure에 대한 재무 약정의 소유자로서 청구 및 차지백을 포함한 리소스 소비 비용을 추적하기 위한 절차 및 정책 개발을 담당합니다.
- *중앙 IT:* 리소스 관리 및 액세스, 워크로드 상태 및 모니터링을 포함한 조직의 클라우드 리소스 관리를 담당합니다.
- *워크로드 소유자:* 개발자, 테스터 및 빌드 엔지니어를 비롯하여 워크로드를 Azure에 배포하는 데 포함된 모든 개발 역할입니다.

## <a name="section-1-azure-basics"></a>섹션 1: Azure 기본 사항

이 소개 섹션은 *재무* 및 *중앙 IT* 가상 사용자를 위한 것입니다. 이 섹션은 기본적으로 [클라우드 거버넌스의 개념](governance-explainer.md)에 대해 알아보는 준비 과정에서 [Azure 작동 방식](azure-explainer.md)을 이해하는 데 중점을 둡니다. 조직의 *워크로드 소유자*가 리소스 액세스를 관리하는 방법을 이해할 수 있도록 이 콘텐츠를 검토하는 데 유용할 수도 있습니다.

## <a name="section-2-governance-design-guide"></a>섹션 2: 거버넌스 디자인 가이드

이제 Azure 작동 방식 및 클라우드 거버넌스의 기본 사항을 이해했으므로 Azure를 채택하는 첫 번째 단계인 Azure에서 [리소스 액세스를 관리하는](azure-resource-access.md) 방법에 대해 알아봅니다. 이 아티클에서는 해당 요청의 유효성을 검사하는 데 사용되는 리소스 액세스 요청 및 컨트롤을 만드는 Azure 서비스를 설명합니다.

다음 단계에서는 단일 팀에 대해 [거버넌스 모델을 디자인하는](governance-how-to.md) 방법을 알아봅니다. 이 아티클에서는 이전에 알아본 리소스 액세스 관리 서비스 및 컨트롤을 구성하는 방법을 설명합니다.

## <a name="section-3-implementing-a-basic-resource-access-management-model"></a>섹션 3: 기본 리소스 액세스 관리 모델 구현

채택 과정의 마지막 단계는 이전에 설계된 거버넌스 모델을 구현하는 방법을 알아보는 것입니다. 

시작하기 위해 조직에는 Azure 계정이 필요합니다. 조직에 Azure가 포함되지 않은 기존 [Microsoft 기업계약](https://www.microsoft.com/licensing/licensing-programs/enterprise.aspx)이 있는 경우 선불 현금 약정 금액을 만들어서 Azure를 추가할 수 있습니다. 자세한 내용은 [엔터프라이즈용 Azure 라이선스](https://azure.microsoft.com/pricing/enterprise-agreement/)를 참조하세요. 

Azure 계정의 만들어지면 조직의 사용자를 Azure **계정 소유자**로 지정합니다. 기본적으로 그런 다음, Azure AD(Azure Active Directory) 테넌트가 만들어집니다. Azure **계정 소유자**는 **워크로드 소유자**인 조직의 사용자에 대한 [사용자 계정을 만들](/azure/active-directory/add-users-azure-active-directory)어야 합니다. 

다음으로, Azure **계정 소유자**는 [구독을 만들고](https://docs.microsoft.com/partner-center/create-a-new-subscription) 여기에 [Azure AD 테넌트를 연결](/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory)해야 합니다.

이제 구독을 만들고 Azure AD 테넌트와 연결했으므로 마지막으로, 기본 제공 **소유자** 역할](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-for-a-subscription-in-azure-portal)을 포함하는 구독에 **워크로드 소유자**를 [추가할 수 있습니다.

## <a name="section-4-deploy-a-basic-workload-architecture-to-azure"></a>섹션 4: Azure에 기본 워크로드 아키텍처 배포

이 섹션에 대한 대상 그룹은 *워크로드 소유자* 가상 사용자입니다. *워크로드 소유자*는 해당 워크로드에 대한 계산 및 네트워킹 요구 사항을 정의하고, 해당 요구 사항을 충족하는 올바른 리소스를 선택하고, 이를 Azure에 배포합니다. 

기초 채택 단계의 경우 워크로드 소유자는 기본 웹 응용 프로그램 또는 VNet(가상 네트워크) 및 VM(가상 머신)을 선택할 수 있습니다. Azure에서 이러한 다양한 형식의 계산 옵션에 대한 자세한 정보는 [Azure 계산 옵션 개요](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)를 검토하세요.

선택한 계산 옵션에 관계 없이 이러한 각 배포에는 **리소스 그룹**이 필요합니다. **워크로드 소유자**는 [리소스 그룹을 만들](/azure/azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy)어야 합니다. **워크로드 소유자**는 배포의 일부로 리소스 그룹의 이름을 지정합니다. 이 이름은 다음 섹션에 사용됩니다.

### <a name="basic-web-application-paas"></a>기본 웹 응용 프로그램(PaaS)

기본 웹 응용 프로그램의 경우 [웹앱 설명서](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json)에서 5분 빠른 시작 중 하나를 선택하여 단계를 수행합니다. 

> [!NOTE]
> 기본적으로 일부 빠른 시작은 리소스 그룹을 배포합니다. 이 경우에 **워크로드 소유자**는 리소스 그룹을 명시적으로 만들 필요가 없습니다. 그렇지 않으면 위에서 만든 리소스 그룹에 웹 응용 프로그램을 배포합니다.

간단한 워크로드를 배포하면 [기본 웹 응용 프로그램](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json)을 Azure에 배포하는 입증된 방법에 대해 자세히 알아볼 수 있습니다.

### <a name="single-windows-or-linux-vm-iaas"></a>단일 Windows 또는 Linux VM(IaaS)

가상 머신에서 실행되는 간단한 워크로드의 경우 첫 번째 단계는 가상 네트워크를 배포하는 것입니다. 가상 머신, 부하 분산 장치 및 게이트웨이와 같은 Azure의 모든 IaaS 리소스에는 가상 네트워크가 필요합니다. [Azure 가상 네트워크](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)에 대해 알아본 다음, [포털을 사용하여 Azure에 Virtual Network를 배포](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)하는 단계를 수행합니다. Azure Portal에서 가상 네트워크에 대한 설정을 지정할 때 위에서 만든 리소스 그룹의 이름을 지정합니다.

다음 단계에서는 단일 Windows 또는 Linux VM을 배포할지를 결정합니다. Windows VM의 경우 [포털에서 Azure에 Windows VM을 배포](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)하는 단계를 수행합니다. 다시 Azure Portal에서 가상 머신에 대한 설정을 지정할 때 위에서 만든 리소스 그룹의 이름을 지정합니다.

단계를 수행하고 VM을 배포하면 [Azure에서 Windows VM을 실행하는 입증된 방법](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)에 대해 알아볼 수 있습니다. Linux VM의 경우 [포털에서 Azure에 Linux VM을 배포](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)하는 단계를 수행합니다. [Azure에서 Linux VM을 실행하기 위한 입증된 방법](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)에 대해 자세히 알아볼 수도 있습니다.

## <a name="next-steps"></a>다음 단계

클라우드 준비의 다음 단계는 [**중간 단계**](../intermediate-stage/overview.md)입니다. 중간 단계에서 여러 팀에 대해 여러 워크로드 및 거버넌스 모델을 실행하는 온-프레미스 네트워크를 확장하는 방법을 알아봅니다.