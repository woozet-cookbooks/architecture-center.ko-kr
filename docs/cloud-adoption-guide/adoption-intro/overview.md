---
title: "Azure 채택: 기본"
description: "엔터프라이즈가 Azure를 채택하기 위해 필요한 기본적인 지식 수준 설명"
author: petertay
ms.openlocfilehash: e9421b610e4eb07a3ed37bca56e513b0689484ef
ms.sourcegitcommit: 9ba82cf84cee06ccba398ec04c51dab0e1ca8974
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/13/2018
---
# <a name="adopting-azure-foundational"></a>Azure 채택: 기본

Azure를 채택하는 것은 엔터프라이즈의 조직을 완성하는 첫 번째 단계입니다. 이 단계가 끝나면 조직의 사용자들은 간단한 워크로드를 Azure에 배포할 수 있습니다.

아래 목록에는 기본 채택 단계를 완료하기 위한 태스크가 포함되어 있습니다. 이 목록은 점진적이므로 각 태스크는 순서대로 완료해야 합니다. 이전에 해당 태스크를 완료했으면 목록의 다음 태스크를 계속 진행합니다. 

1. Azure 내부 기능 이해
    - **설명:** [Azure의 작동 방식](azure-explainer.md)
2. Azure의 엔터프라이즈 디지털 ID 이해
    - **설명:** [Azure Active Directory 테넌트란?](tenant-explainer.md)
    - **방법:** [Azure Active Directory 테넌트 얻기](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **지침:** [Azure AD 테넌트 디자인](tenant.md)
    - **방법:** [Azure Active Directory에 새 사용자 추가](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)    
3. Azure의 구독 이해
    - **설명:** [Azure 구독이란?](subscription-explainer.md)
    - **지침:** [Azure 구독 디자인](subscription.md)
4. Azure의 리소스 관리 이해 
    - **설명:** [Azure Resource Manager란?](resource-manager-explainer.md)
    - **설명:** [Azure 리소스 그룹이란?](resource-group-explainer.md)
    - **설명:** [Azure의 리소스 액세스 이해](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **방법:** [Azure Portal을 사용하여 Azure 리소스 그룹 만들기](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **지침:** [Azure 리소스 그룹 디자인 지침](resource-group.md)
    - **지침:** [Azure 리소스에 대한 명명 규칙](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)
5. 기본 Azure 아키텍처 배포
    - [Azure 계산 옵션 개요](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)에서 다양한 유형의 Azure 계산 옵션[예: IaaS(Infrastructure-as-a-Service) 및 PaaS(Platform-as-a-Service)]에 대해 알아보세요.
    - 다양한 유형의 Azure 계산 옵션을 이해했으므로 이제 Azure에서 첫 번째 리소스로 PaaS 웹 응용 프로그램 또는 IaaS 가상 머신을 선택합니다.
    - PaaS: Platform as a Service 소개
        - **방법:** [Azure에 기본 웹 응용 프로그램 배포](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **지침:** [기본 웹 응용 프로그램](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json)을 Azure에 배포하기 위한 입증된 방법
    - IaaS: 가상 네트워킹 소개
        - **설명:** [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **방법:** [포털을 사용하여 Azure에 Virtual Network 배포](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - IasS: 단일 VM(가상 머신) 워크로드(예: Windows 및 Linux) 배포
        - **방법:** [포털에서 Azure에 Windows VM 배포](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **지침:** [Azure에서 Windows VM을 실행하기 위한 입증된 방법](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **방법:** [포털에서 Azure에 Linux VM 배포](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **지침:** [Azure에서 Linux VM을 실행하기 위한 입증된 방법](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
