---
title: Azure에서 ADDS 리소스 포리스트 만들기
description: >-
  Azure에서 신뢰할 수 있는 Active Directory 도메인을 만드는 방법입니다.

  지침, vpn-게이트웨이, expressroute, 부하 분산 장치, 가상 네트워크, active-directory
author: telmosampaio
ms.date: 05/02/2018
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: 64253d900dbce9966aa76d99d758bfb581b9df5f
ms.sourcegitcommit: 2154e93a0a075e1f7425a6eb11fc3f03c1300c23
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352613"
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a>Azure에서 AAD DS(Active Directory Domain Services) 리소스 포리스트 만들기

이 참조 아키텍처는 온-프레미스 AD 포레스트에 있는 도메인의 신뢰를 받는 Azure의 별도 Active Directory 도메인을 생성하는 방법을 설명합니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

[![0]][0] 

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

AD DS(Active Directory Domain Services)는 ID 정보를 계층 구조에 저장합니다. 계층 구조의 최상위 노드를 포레스트라고 합니다. 포레스트는 도메인을, 도메인은 다른 유형의 개체를 담고 있습니다. 이 참조 아키텍처는 온-프레미스 도메인과의 단방향의 보내는 트러스트 관계를 구축하여 Azure에 AD DS 포레스트를 만듭니다. Azure에 있는 포레스트는 온-프레미스에 없는 도메인을 포함합니다. 트러스트 관계를 바탕으로 온-프레미스 도메인에서 수행한 로그인을 통해 별개의 Azure 도메인에 있는 리소스에 접속할 수 있습니다. 

이 아키텍처의 일반적인 용도는 클라우드에 저장된 개체 및 ID에 대한 보안 분리를 유지하는 것과 개별 도메인을 온-프레미스에서 클라우드로 마이그레이션하는 것입니다. 

추가 고려 사항은 [온-프레미스 Active Directory를 Azure와 통합하기 위한 솔루션 선택][considerations]을 참조하세요. 

## <a name="architecture"></a>아키텍처

이 아키텍처의 구성 요소는 다음과 같습니다.

* **온-프레미스 네트워크**. 온-프레미스 네트워크는 자체 Active Directory 포레스트와 도메인을 포함합니다.
* **Active Directory 서버**. 클라우드에서 VM으로 실행되는 도메인 서비스를 구현하는 도메인 컨트롤러입니다. 이러한 서버는 온-프레미스에 위치한 도메인들과 분리된 하나 이상의 도메인을 포함하는 포레스트를 호스트합니다.
* **단방향 트러스트 관계**. 이 다이어그램의 예제는 Azure 도메인으로부터 온-프레미스 도메인으로의 단방향 트러스트 관계를 보여 줍니다. 이 관계를 통해 온-프레미스 사용자는 Azure의 도메인에 있는 리소스에 접근할 수 있지만 Azure 사용자가 온-프레미스의 리소스에 접근할 수는 없습니다. 클라우드 사용자가 온-프레미스 리소스 액세스를 요구하는 경우 양방향 트러스트 관계를 생성할 수도 있습니다.
* **Active Directory 서브넷**. AD DS 서버는 별도의 서브넷에 호스팅됩니다. NSG(네트워크 보안 그룹) 규칙은 AD DS 서버를 보호하고 예상치 못한 소스로부터의 트래픽에 대한 방화벽을 제공합니다.
* **Azure 게이트웨이**. Azure 게이트웨이는 온-프레미스 네트워크와 Azure VNet 간 연결을 제공합니다. [VPN 연결][azure-vpn-gateway] 또는 [Azure ExpressRoute][azure-expressroute]가 제공될 수 있습니다. 자세한 내용은 [Azure에 안전한 하이브리드 보안 네트워크 아키텍처 구현][implementing-a-secure-hybrid-network-architecture]을 참조하세요.

## <a name="recommendations"></a>권장 사항

Azure에서 Active Directory를 구현하는 방법에 대한 특정 권장 구성은 다음 문서를 참조하세요.

- [AD DS(Active Directory Domain Services)를 Azure로 확장][adds-extend-domain] 
- [Azure Virtual Machines에 Windows Server Active Directory를 배포하기 위한 지침][ad-azure-guidelines]

### <a name="trust"></a>신뢰

온-프레미스 도메인은 클라우드에 있는 도메인과는 다른 포레스트 내에 있습니다. 클라우드에서 온-프레미스 사용자 인증을 하려면 Azure 도메인이 온-프레미스 포레스트의 로그인 도메인을 신뢰해야 합니다. 마찬가지로, 클라우드가 외부 사용자를 위한 로그인 도메인을 제공한다면 온-프레미스 포레스트가 클라우드 도메인을 신뢰해야 합니다.

포레스트 수준에서는 [포레스트 트러스트 생성][creating-forest-trusts]을 통해, 도메인 수준에서는 [외부 트러스트 생성][creating-external-trusts]을 통해 트러스트를 구축할 수 있습니다. 포레스트 수준 트러스트는 두 개의 포레스트 내에 있는 모든 도메인들 사이에 관계를 형성합니다. 외부 도메인 수준 트러스트는 두 개의 지정된 도메인 간 관계만을 형성합니다. 서로 다른 포레스트의 도메인 간에는 외부 도메인 수준 트러스트만 형성해야 합니다.

트러스트는 단방향 또는 양방향으로 생성됩니다.

* 단방향 트러스트는 하나의 도메인 또는 포레스트(*수신* 도메인 또는 포리스트)의 사용자가 다른(*송신*) 도메인 또는 포레스트에 있는 리소스에 액세스할 수 있도록 합니다.
* 양방향 트러스트는 어느 한 쪽의 도메인 또는 포레스트에 있는 사용자가 다른 쪽 도메인 또는 포레스트에 있는 리소스에 액세스할 수 있도록 합니다.

다음 표에는 간단한 시나리오에 대한 트러스트 구성이 요약되어 있습니다.

| 시나리오 | 온-프레미스 트러스트 | 클라우드 트러스트 |
| --- | --- | --- |
| 온-프레미스 사용자는 클라우드에 있는 리소스에 대한 액세스를 요구할 수 있지만 그 반대 방향은 불가능합니다. |단방향, 수신 |단방향, 송신 |
| 클라우드 사용자는 온-프레미스에 위치한 리소스에 대한 액세스를 요구할 수 있지만 그 반대 방향은 불가능합니다. |단방향, 송신 |단방향, 수신 |
| 클라우드와 온-프레미스 사용자 모두 클라우드와 온-프레미스에 있는 리소스에 대한 액세스를 요구할 수 있습니다. |양방향, 수신 및 송신 |양방향, 수신 및 송신 |

## <a name="scalability-considerations"></a>확장성 고려 사항

Active Directory는 동일한 도메인의 일부인 도메인 컨트롤러를 위해 자동으로 확장 가능합니다. 요청은 하나의 도메인 내에 있는 모든 컨트롤러로 분산됩니다. 도메인 컨트롤러를 추가하면 자동으로 해당 도메인과 동기화가 진행됩니다. 해당 도메인 내에서 트래픽을 컨트롤러로 전송하기 위해 별도의 부하 분산 장치를 구성하지 마세요. 모든 도메인 컨트롤러는 도메인 데이터베이스를 처리하기에 충부한 메모리와 저장소 리소스를 가져야 합니다. 모든 도메인 컨트롤러 VM은 동일한 크기여야 합니다.

## <a name="availability-considerations"></a>가용성 고려 사항

각 도메인에 대해 최소 두 개의 도메인 컨트롤러를 프로비전합니다. 이를 통해 서버 간 자동 복제가 가능해집니다. 각 도메인을 처리하는 Active Directory 서버의 역할을 하는 VM에 대한 가용성 집합을 생성합니다. 이 가용성 집합 내에 최소 두 개의 서버를 배치합니다.

또한 유연한 단일 마스터 작업(FSMO) 역할을 수행하는 서버에 대한 연결 실패를 대비하여 각 도메인에 있는 하나 이상의 서버를 [대기 작업 마스터][standby-operations-masters]로 지정하는 것을 고려해 보세요.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

관리 및 모니터링 고려사항에 대한 자세한 내용은 [Active Directory를 Azure로 확장][adds-extend-domain]을 참조하세요. 
 
자세한 내용은 [Active Directory 모니터링][monitoring_ad]을 참조하세요. 관리 서브넷의 모니터링 서버에 [Microsoft Systems Center][microsoft_systems_center]와 같은 도구를 설치하며 이러한 작업 수행이 수월해질 수 있습니다.

## <a name="security-considerations"></a>보안 고려 사항

포레스트 수준 트러스트는 전이적입니다. 온-프레미스 포레스트와 클라우드 포레스트 간 포레스트 수준 트러스트를 구축하는 경우 이 트러스트는 어느 하나의 포레스트에 생성된 다른 새로운 도메인로 확장됩니다. 보안을 목적으로 도메인을 사용하여 분리를 제공하는 경우에는 도메인 수준에서만 트러스트를 생성하는 것을 고려하세요. 도메인 수준 트러스트는 비전이적입니다.

Active Directory 관련 보안 고려사항을 확인하려면 [Active Directory를 Azure로 확장][adds-extend-domain]의 보안 고려항 섹션을 참조하세요.

## <a name="deploy-the-solution"></a>솔루션 배포

이 아키텍처에 대한 배포는 [GitHub][github]에서 사용할 수 있습니다. 전체 배포는 최대 2시간이 걸릴 수 있으며 VPN 게이트웨이 만들기 및 AD DS를 구성하는 스크립트 실행을 포함합니다.

### <a name="prerequisites"></a>필수 조건

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a>시뮬레이션된 온-프레미스 데이터 센터 배포

1. GitHub 리포지토리의 `identity/adds-forest` 폴더로 이동합니다.

2. `onprem.json` 파일을 엽니다. `adminPassword` 및 `Password` 인스턴스를 검색하고 암호 값을 추가합니다.

3. 다음 명령을 실행하고 배포가 끝나기를 기다립니다.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a>Azure VNet에 배포

1. `azure.json` 파일을 엽니다. `adminPassword` 및 `Password` 인스턴스를 검색하고 암호 값을 추가합니다.

2. 동일한 파일에서 `sharedKey` 인스턴스를 검색하고 VPN 연결에 대한 공유 키를 입력합니다. 

    ```bash
    "sharedKey": "",
    ```

3. 다음 명령을 실행하고 배포가 끝나기를 기다립니다.

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onoprem.json --deploy
    ```

   온-프레미스 VNet과 동일한 리소스 그룹에 배포합니다.


### <a name="test-the-ad-trust-relation"></a>AD 신뢰 관계 테스트

1. Azure Portal을 사용하여 만든 리소스 그룹으로 이동합니다.

2. Azure Portal을 사용하여 `ra-adt-mgmt-vm1`이라는 VM을 찾습니다.

2. `Connect`를 클릭하여 VM에 대한 원격 데스크톱 세션을 엽니다. 사용자 이름은 `contoso\testuser`이고, 암호는 `onprem.json` 매개 변수 파일에서 지정한 것입니다.

3. 원격 데스크톱 세션 내에서 `ra-adtrust-onpremise-ad-vm1`이라는 VM의 IP 주소인 192.168.0.4에 대한 다른 원격 데스크톱 세션을 엽니다. 사용자 이름은 `contoso\testuser`이고, 암호는 `azure.json` 매개 변수 파일에서 지정한 것입니다.

4. `ra-adtrust-onpremise-ad-vm1`에 대한 원격 데스크톱 세션 내에서 **서버 관리자**로 이동하고, **도구** > **Active Directory 도메인 및 신뢰**를 클릭합니다. 

5. 왼쪽 창에서 contoso.com을 마우스 오른쪽 단추로 클릭하고, **속성**을 선택합니다.

6. **신뢰** 탭을 클릭합니다. 들어오는 신뢰로 나열된 treyresearch.net이 표시됩니다.

![](./images/ad-forest-trust.png)


## <a name="next-steps"></a>다음 단계

* [온-프레미스 AD DS 도메인을 Azure로 확장][adds-extend-domain]하기 위한 모범 사례를 살펴보세요.
* Azure에서 [AD FS 인프라를 생성][adfs]하기 위한 모범 사례를 살펴보세요.

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://microsoft.com/cloud-platform/system-center
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "별도 Active Directory 도메인으로 하이브리드 네트워크 아키텍처 보안 유지"
