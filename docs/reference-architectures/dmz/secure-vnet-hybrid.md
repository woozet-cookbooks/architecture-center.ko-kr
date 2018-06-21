---
title: Azure에서 보안 하이브리드 네트워크 아키텍처 구현
description: Azure에서 보안 하이브리드 네트워크 아키텍처를 구현하는 방법입니다.
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.prev: ./index
pnp.series.next: secure-vnet-dmz
cardTitle: DMZ between Azure and on-premises
ms.openlocfilehash: 81dea2e4439d5a01ebb88ab86dc0a59609bb7bc3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30849657"
---
# <a name="dmz-between-azure-and-your-on-premises-datacenter"></a>Azure와 온-프레미스 데이터 센터 간의 DMZ

이 참조 아키텍처는 온-프레미스 네트워크를 Azure로 확장하는 보안 하이브리드 네트워크를 보여줍니다. 아키텍처는 DMZ를 구현하며 또한 온-프레미스 네트워크와 Azure VNet(Virtual Network) 간의 *경계 네트워크*라고도 합니다. DMZ에는 방화벽 및 패킷 검사와 같은 보안 기능을 구현하는 NVA(네트워크 가상 어플라이언스)가 포함됩니다. VNet에서 나가는 모든 트래픽이 감사될 수 있도록 온-프레미스 네트워크를 통해 인터넷에 강제 터널링됩니다.

[![0]][0] 

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

이 아키텍처는 [VPN Gateway][ra-vpn] 또는 [ExpressRoute][ra-expressroute] 연결을 사용하여 온-프레미스 데이터 센터에 연결해야 합니다. 일반적으로 이 아키텍처는 다음과 같은 용도로 사용됩니다.

* 워크로드의 일부는 온-프레미스에서, 일부는 Azure에서 실행되는 하이브리드 응용 프로그램
* 온-프레미스 데이터 센터에서 Azure VNet을 입력하는 트래픽을 통해 세부적으로 제어해야 하는 인프라
* 나가는 트래픽을 감사해야 하는 응용 프로그램 이 항목은 여러 상용 시스템의 규제 요구 사항이며 개인 정보를 공개하지 않도록 방지할 수 있습니다.

## <a name="architecture"></a>건축

이 아키텍처는 다음 구성 요소로 구성됩니다.

* **온-프레미스 네트워크**. 조직에서 구현되는 개인 로컬 영역 네트워크입니다.
* **Azure VNet(Virtual Network)** VNet은 응용 프로그램 및 Azure에서 실행되는 다른 리소스를 호스트합니다.
* **게이트웨이**. 게이트웨이는 온-프레미스 네트워크와 VNet의 라우터 간에 연결을 제공합니다.
* **NVA(네트워크 가상 어플라이언스)** NVA는 방화벽, 최적화 WAN(광역 네트워크) 작업(네트워크 압축 포함), 사용자 지정 라우팅 또는 기타 네트워크 기능으로 액세스를 허용하거나 거부하는 등의 작업을 수행하는 VM에 대해 설명하는 일반 용어입니다.
* **웹 계층, 비즈니스 계층 및 데이터 계층 서브넷** 클라우드에서 실행되는 3계층 응용 프로그램 예제를 구현하는 VM 및 서비스를 호스트하는 서브넷입니다. 자세한 내용은 [Azure에서 N 계층 아키텍처에 대한 Windows VM 실행][ra-n-tier]을 참조하세요.
* **UDR(사용자 정의 경로)** [사용자 정의 경로][udr-overview]는 Azure VNet 내에서 IP 트래픽 흐름을 정의합니다.

    > [!NOTE]
    > VPN 연결의 요구 사항에 따라 온-프레미스 네트워크를 통해 트래픽을 다시 전달하는 전달 규칙을 구현하기 위해 UDR을 사용하는 대신 BGP(경계 게이트웨이 프로토콜) 경로를 구성할 수 있습니다.
    > 
    > 

* **관리 서브넷** 이 서브넷에는 VNet에서 실행되는 구성 요소에 대한 관리 및 모니터링 기능을 구현하는 VM이 포함됩니다.

## <a name="recommendations"></a>권장 사항

대부분의 시나리오의 경우 다음 권장 사항을 적용합니다. 이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다. 

### <a name="access-control-recommendations"></a>액세스 제어 권장 사항

RBAC([역할 기반 액세스 제어][rbac])를 사용하여 응용 프로그램에서 리소스를 관리합니다. 다음 [사용자 정의 역할][rbac-custom-roles]을 만드는 것이 좋습니다.

- 응용 프로그램에서 인프라를 관리하는 사용 권한이 있는 DevOps 역할은 응용 프로그램 구성 요소를 배포하고 VM을 모니터링하고 다시 시작합니다.  

- 네트워크 리소스를 관리하고 모니터링하는 중앙 집중식 IT 관리자 역할

- NVA와 같은 보안 네트워크 리소스를 관리하는 보안 IT 관리자 역할 

DevOps 및 IT 관리자 역할에는 NVA 리소스에 대한 액세스 권한이 없어야 합니다. 이 권한은 보안 IT 관리자 역할로 제한되어야 합니다.

### <a name="resource-group-recommendations"></a>리소스 그룹 권장 사항

VM, VNet 및 부하 분산 장치와 같은 Azure 리소스는 리소스 그룹으로 함께 그룹화하여 쉽게 관리될 수 있습니다. RBAC 역할을 각 리소스 그룹을 할당하여 액세스를 제한합니다.

다음과 같은 리소스 그룹을 만드는 것이 좋습니다.

* 온-프레미스 네트워크에 연결하는 VNet(VM 제외), NSG 및 게이트웨이 리소스를 포함하는 리소스 그룹 이 리소스 그룹에 중앙 집중식 IT 관리자 역할을 할당합니다.
* 부하 분산 장치를 비롯한 NVA에 VM을 포함하는 리소스 그룹, jumpbox 및 기타 관리 VM 및 NVA를 통해 모든 트래픽에 강제로 적용하는 게이트웨이 서브넷의 UDR 이 리소스 그룹에 보안 IT 관리자 역할을 할당합니다.
* 부하 분산 장치 및 VM을 포함하는 각 응용 프로그램 계층의 별도 리소스 그룹 이 리소스 그룹은 각 계층에 서브넷을 포함하지 않아야 합니다. 이 리소스 그룹에 DevOps 역할을 할당합니다.

### <a name="virtual-network-gateway-recommendations"></a>가상 네트워크 게이트웨이 권장 사항

온-프레미스 트래픽은 가상 네트워크 게이트웨이를 통해 VNet에 전달됩니다. [Azure VPN Gateway][guidance-vpn-gateway] 또는 [Azure ExpressRoute 게이트웨이][guidance-expressroute]를 사용하는 것이 좋습니다.

### <a name="nva-recommendations"></a>NVA 권장 사항

NVA는 네트워크 트래픽을 관리하고 모니터링하기 위한 다양한 서비스를 제공합니다. [Azure Marketplace][azure-marketplace-nva]는 사용할 수 있는 여러 타사 공급 업체 NVA를 제공합니다. 이러한 타사 NVA 중 요구 사항을 충족하는 것이 없는 경우 VM을 사용하여 사용자 지정 NVA를 만들 수 있습니다. 

예를 들어 이 참조 아키텍처에 솔루션을 배포하면 VM에서 다음과 같은 기능을 사용하여 NVA를 구현합니다.

* NVA NIC(네트워크 인터페이스)에서 [IP 전달][ip-forwarding]을 사용하여 트래픽을 라우팅합니다.
* 이런 방법이 적절한 경우에만 NVA를 통해 트래픽을 전달할 수 있습니다. 참조 아키텍처의 각 NVA VM은 간단한 Linux 라우터입니다. 인바운드 트래픽이 네트워크 인터페이스 *eth0*에 도착하고, 아웃바운드 트래픽이 네트워크 인터페이스*eth1*을 통해 발송된 사용자 지정 스크립트에서 정의한 규칙과 일치합니다.
* NVA는 관리 서브넷에서만 구성될 수 있습니다. 
* 관리 서브넷에 라우팅되는 트래픽은 NVA를 통해 전달되지 않습니다. 그렇지 않고 NVA에 실패하는 경우 이를 수정할 관리 서브넷에 대한 경로가 없습니다.  
* NVA의 VM은 부하 분산 장치 뒤의 [가용성 집합][availability-set]에 배치됩니다. 게이트웨이 서브넷의 UDR은 부하 분산 장치에 NVA 요청을 전달합니다.

NVA 수준에서 응용 프로그램 연결을 종료하고 백 엔드 계층 선호도를 유지하기 위해 계층 7 NVA를 포함시킵니다. 이를 통해 백 엔드 계층으로부터의 응답 트래픽이 NVA를 통해 돌아오는 대칭 연결이 보장됩니다.

고려해야 할 또 다른 옵션은 특수화된 보안 작업을 수행하는 각 NVA를 사용하여 여러 NVA를 함께 연결하는 것입니다. 이렇게 하면 각 보안 함수를 NVA 단위 기준으로 관리할 수 있습니다. 예를 들어 방화벽을 구현하는 NVA는 ID 서비스를 실행하는 NVA와 함께 배치될 수 있습니다. 관리 편의상 단점은 대기 시간이 늘어날 수 있는 네트워크 홉이 추가된다는 것입니다. 따라서 응용 프로그램의 성능에 영향을 주지 않도록 합니다.


### <a name="nsg-recommendations"></a>NSG 권장 사항

VPN Gateway는 온-프레미스 네트워크에 연결하기 위한 공용 IP 주소를 노출합니다. 온-프레미스 네트워크에서 발생하지 않는 모든 트래픽을 차단하는 규칙을 사용하여 인바운드 NVA 서브넷에 NSG(네트워크 보안 그룹)를 만드는 것이 좋습니다.

각 서브넷의 NSG의 경우 잘못 구성되었거나 사용할 수 없는 NVA를 무시하는 인바운드 트래픽에 대해 두 번째 수준의 보호를 제공하는 것이 좋습니다. 예를 들어 참조 아키텍처의 웹 계층 서브넷은 온-프레미스 네트워크(192.168.0.0/16) 또는 VNet에서 수신되지 않은 모든 요청을 무시하는 규칙 및 포트 80에서 만들지 않은 모든 요청을 무시하는 다른 규칙을 사용하여 NSG를 구현합니다.

### <a name="internet-access-recommendations"></a>인터넷 액세스 권장 사항

사이트 간 VPN 터널을 사용하는 온-프레미스 네트워크를 통한 모든 아웃바운드 인터넷 트래픽 및 NAT(네트워크 주소 변환)를 사용하는 인터넷에 대한 경로를 [강제 터널링][azure-forced-tunneling]합니다. 그러면 데이터 계층에 저장된 모든 기밀 정보를 실수로 누출하지 않도록 방지하고 모든 나가는 트래픽을 검사하고 감사할 수 있습니다.

> [!NOTE]
> 응용 프로그램 계층의 인터넷 트래픽을 완전히 차단하지 마십시오. 차단하는 경우 이러한 계층이 VM 진단 로깅, VM 확장 및 기타 기능 다운로드 등 공용 IP 주소를 사용하는 Azure PaaS 서비스를 사용하지 않도록 방해하게 됩니다. 또한 Azure 진단에서는 구성 요소가 Azure Storage 계정에 읽고 쓸 수 있어야 합니다.
> 
> 

아웃바운드 인터넷 트래픽이 올바르게 강제 터널링되었는지 확인합니다. 온-프레미스 서버에서 [라우팅 및 원격 액세스 서비스][routing-and-remote-access-service]를 사용하여 VPN 연결을 사용하는 경우 [WireShark][wireshark] 또는 [Microsoft 메시지 분석기](https://www.microsoft.com/download/details.aspx?id=44226)와 같은 도구를 사용합니다.

### <a name="management-subnet-recommendations"></a>관리 서브넷 권장 사항

관리 서브넷에는 관리 및 모니터링 기능을 수행하는 jumpbox가 포함되어 있습니다. jumpbox에 대한 모든 보안 관리 작업의 실행을 제한합니다.
 
jumpbox에 대한 공용 IP 주소를 만들지 마십시오. 대신 들어오는 게이트웨이를 통해 jumpbox에 액세스할 수 있는 하나의 경로를 만듭니다. 관리 서브넷이 허용되는 경로의 요청에만 응답하므로 NSG 규칙을 만듭니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

참조 아키텍처는 부하 분산 장치를 사용하여 온-프레미스 네트워크 트래픽을 NVA 장치의 풀로 전달합니다. 여기서 트래픽을 라우팅합니다. NVA는 [가용성 집합][availability-set]에 배치됩니다. 이 디자인을 사용하면 시간에 따라 NVA의 처리량을 모니터링하고 부하의 증가에 대한 대응으로 NVA 장치를 추가할 수 있습니다.

표준 SKU VPN Gateway는 최대 100Mbps 지속 처리량을 지원합니다. 고성능 SKU는 최대 200Mbps를 제공합니다. 높은 대역폭의 경우 ExpressRoute 게이트웨이로 업그레이드하는 것이 좋습니다. ExpressRoute는 VPN 연결보다 대기 시간이 낮은 최대 10Gbps 대역폭을 제공합니다.

Azure 게이트웨이의 확장성에 대한 자세한 내용은 [Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-scalability] 및 [Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-scalability]의 확장성 고려 사항 섹션을 참조하세요.

## <a name="availability-considerations"></a>가용성 고려 사항

언급했듯이 참조 아키텍처는 부하 분산 장치 뒤의 NVA 장치 풀을 사용합니다. 부하 분산 장치는 상태 프로브를 사용하여 각 NVA를 모니터링하고 응답하지 않는 NVA를 풀에서 제거합니다.

VNet과 온-프레미스 네트워크 간의 연결을 제공하는 데 Azure ExpressRoute를 사용하는 경우 ExpressRoute 연결을 사용할 수 없게 되면 [장애 조치를 제공하는 VPN Gateway를 구성합니다][ra-vpn-failover].

VPN 및 ExpressRoute 연결에서 가용성을 유지 관리하는 방법에 대한 자세한 내용은 [Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-availability] 및 [Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-availability]의 가용성 고려 사항을 참조하세요. 

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

모든 응용 프로그램 및 리소스 모니터링은 관리 서브넷에 있는 jumpbox에서 수행해야 합니다. 응용 프로그램 요구 사항에 따라 관리 서브넷에 추가 모니터링 리소스가 필요할 수 있습니다. 그러면 이러한 리소스는 jumpbox를 통해 액세스되어야 합니다.

온-프레미스 네트워크에서 Azure로의 게이트웨이 연결이 중단되는 경우에도 공용 IP 주소를 배포하고 점프박스에 추가한 후 인터넷으로부터 로그인하여 점프박스에 원격으로 액세스할 수 있습니다

참조 아키텍처에서 각 계층의 서브넷은 NSG 규칙에 의해 보호됩니다. Windows VM에서 RDP(원격 데스크톱 프로토콜) 액세스의 경우 포트 3389 또는 Linux VM에서 SSH(Secure Shell) 액세스의 경우 포트 22를 여는 규칙을 만들어야 합니다. 다른 관리 및 모니터링 도구에는 추가 포트를 여는 규칙이 필요할 수 있습니다.

ExpressRoute를 사용하여 온-프레미스 데이터 센터와 Azure 간의 연결을 제공하는 경우 [AzureCT(Azure 연결 도구 키트)][azurect]를 사용하여 연결 문제를 모니터링하고 해결합니다.

VPN 및 ExpressRoute 연결을 모니터링하고 관리하는 방법에 대해 특별히 다루는 추가 정보는 [Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-manageability] 및 [Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-manageability] 문서에서 찾을 수 있습니다.

## <a name="security-considerations"></a>보안 고려 사항

이 참조 아키텍처는 여러 수준의 보안을 구현합니다.

### <a name="routing-all-on-premises-user-requests-through-the-nva"></a>NVA를 통해 모든 온-프레미스 사용자 요청 라우팅
게이트웨이 서브넷의 UDR은 온-프레미스에서 수신되지 않은 모든 사용자 요청을 차단합니다. UDR은 개인 DMZ 서브넷의 NVA에 허용된 요청을 전달하고 이러한 요청이 NVA 규칙에 의해 허용된 경우 응용 프로그램에 전달됩니다. UDR에 다른 경로를 추가할 수 있지만 NVA를 실수로 우회하거나 관리 서브넷에 전달하려는 관리 트래픽을 차단하지 않습니다.

NVA 앞의 부하 분산 장치는 부하 분산 규칙에서 열려 있지 않은 포트에 있는 트래픽을 무시하여 보안 장치의 역할을 수행합니다. 참조 아키텍처의 부하 분산 장치는 포트 80의 HTTP 요청 및 포트 443의 HTTPS 요청만을 수신합니다. 부하 분산 장치에 추가한 모든 추가 규칙을 문서화하고 보안 문제가 없는지 확인하기 위해 트래픽을 모니터링합니다.

### <a name="using-nsgs-to-blockpass-traffic-between-application-tiers"></a>NSG를 사용하여 응용 프로그램 계층 간 트래픽 차단/통과
NSG를 사용하여 계층 간 트래픽을 제한합니다. 비즈니스 계층은 웹 계층에서 발생하지 않는 모든 트래픽을 차단하고 데이터 계층은 비즈니스 계층에서 발생하지 않는 모든 트래픽을 차단합니다. 이러한 계층에 대한 광범위한 액세스를 허용하기 위해 NSG 규칙을 확장해야 하는 경우 보안 위험에 비하여 이러한 요구 사항을 평가합니다. 새로운 인바운드 경로는 각각 우발적 또는 고의적 데이터 유출 또는 응용 프로그램 손상이 발생할 가능성을 나타냅니다.

### <a name="devops-access"></a>DevOps 액세스
[RBAC][rbac]를 사용하여 DevOps가 각 계층에서 수행할 수 있는 작업을 제한합니다. 사용 권한을 부여할 때 [최소 권한의 원칙][security-principle-of-least-privilege]을 사용합니다. 모든 관리 작업을 기록하고 정기 감사를 수행하여 구성 변경을 계획했는지 확인합니다.

## <a name="solution-deployment"></a>솔루션 배포

이러한 권장 사항을 구현하는 참조 아키텍처 배포는 [GitHub][github-folder]를 통해 수행할 수 있습니다. 이 참조 아키텍처는 다음 지침에 따라 배포할 수 있습니다.

1. 아래 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-hybrid%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열렸으면 일부 설정에 대한 값을 입력해야 합니다.   
   * **리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-private-dmz-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 하위 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구매** 단추를 클릭합니다.
3. 배포가 완료될 때가지 기다립니다.
4. 매개 변수 파일에는 모든 VM에 대한 하드 코딩된 관리자 사용자 이름 및 암호가 포함되어 있는데, 이 둘 모두를 즉시 변경할 것을 권장합니다. 배포의 각 VM을 Azure Portal에서 선택한 후 **지원 + 문제 해결** 블레이드에서 **암호 재설정**을 클릭합니다. **모드** 드롭다운 상자에서 **암호 재설정**을 선택한 후 새 **사용자 이름** 및 **암호**를 선택합니다. **업데이트** 단추를 클릭하여 저장합니다.

## <a name="next-steps"></a>다음 단계

* [Azure와 인터넷 간의 DMZ](secure-vnet-dmz.md)를 구현하는 방법에 대해 알아봅니다.
* [고가용성 하이브리드 네트워크 아키텍처][ra-vpn-failover]를 구현하는 방법에 대해 알아봅니다.
* Azure에서 네트워크 보안을 관리하는 방법에 대한 자세한 내용은 [Microsoft Cloud Services 및 네트워크 보안][cloud-services-network-security]을 참조하세요.
* Azure에서 리소스를 보호하는 방법에 대한 자세한 내용은 [Microsoft Azure 보안 시작][getting-started-with-azure-security]을 참조하세요. 
* Azure 게이트웨이 연결에서 보안 문제를 해결하는 방법에 대한 자세한 내용은 [Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-security] 및 [Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-security]을 참조하세요.
  > 

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[azure-forced-tunneling]: https://azure.microsoft.com/en-gb/documentation/articles/vpn-gateway-forced-tunneling-rm/
[azure-marketplace-nva]: https://azuremarketplace.microsoft.com/marketplace/apps/category/networking
[cloud-services-network-security]: https://azure.microsoft.com/documentation/articles/best-practices-network-security/
[getting-started-with-azure-security]: /azure/security/azure-security-getting-started
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-hybrid
[guidance-expressroute]: ../hybrid-networking/expressroute.md
[guidance-expressroute-availability]: ../hybrid-networking/expressroute.md#availability-considerations
[guidance-expressroute-manageability]: ../hybrid-networking/expressroute.md#manageability-considerations
[guidance-expressroute-security]: ../hybrid-networking/expressroute.md#security-considerations
[guidance-expressroute-scalability]: ../hybrid-networking/expressroute.md#scalability-considerations
[guidance-vpn-gateway]: ../hybrid-networking/vpn.md
[guidance-vpn-gateway-availability]: ../hybrid-networking/vpn.md#availability-considerations
[guidance-vpn-gateway-manageability]: ../hybrid-networking/vpn.md#manageability-considerations
[guidance-vpn-gateway-scalability]: ../hybrid-networking/vpn.md#scalability-considerations
[guidance-vpn-gateway-security]: ../hybrid-networking/vpn.md#security-considerations
[ip-forwarding]: /azure/virtual-network/virtual-networks-udr-overview#ip-forwarding
[ra-expressroute]: ../hybrid-networking/expressroute.md
[ra-n-tier]: ../virtual-machines-windows/n-tier.md
[ra-vpn]: ../hybrid-networking/vpn.md
[ra-vpn-failover]: ../hybrid-networking/expressroute-vpn-failover.md
[rbac]: /azure/active-directory/role-based-access-control-configure
[rbac-custom-roles]: /azure/active-directory/role-based-access-control-custom-roles
[routing-and-remote-access-service]: https://technet.microsoft.com/library/dd469790(v=ws.11).aspx
[security-principle-of-least-privilege]: https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview
[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
[wireshark]: https://www.wireshark.org/
[0]: ./images/dmz-private.png "하이브리드 네트워크 아키텍처 보안"
