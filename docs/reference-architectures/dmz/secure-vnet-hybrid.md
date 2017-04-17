---
title: Implementing a secure hybrid network architecture in Azure
description: How to implement a secure hybrid network architecture in Azure.
author: telmosampaio
ms.service: guidance
ms.topic: article
ms.date: 11/23/2016
ms.author: pnp

pnp.series.title: Network DMZ
pnp.series.prev: ./index
pnp.series.next: secure-vnet-dmz
cardTitle: DMZ between Azure and on-premises
---
# Azure와 온-프레미스 데이터센터 사이
[!INCLUDE [header](../../_includes/header.md)]

이 문서는 온-프레미스 네트워크를 Azure로 확장하는 보안된 하이브리드 네트워크를 구현하기 위한 모범 사례를 소개합니다. 이 참조 아키텍처는 온-프레미스 네트워크와 Azure 가상 네트워크 간 DMZ(또는 경계 네트워크)를 구현합니다. DMZ는 방화벽이나 패킷 검사와 같은 보안 기능을 구현하는 고가용성 네트워크 가상 어플라이언스(NVA)를 포함합니다. VNet에서 나가는 모든 트래픽은 감사를 위해 온-프레미스 네트워크를 통해 인터넷으로 강제 터널링됩니다.

이 아키텍처는 [VPN 게이트웨이][ra-vpn]나 [ExpressRoute][ra-expressroute] 연결을 통해 온-프레미스 데이터센터에 연결할 필요가 있습니다.

> [!참고]
> Azure는 [Resource Manager](/azure/azure-resource-manager/resource-group-overview)와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 참조 아키텍처에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다.
> 
> 

일반적으로 이 아키텍처는 다음과 같은 용도로 사용됩니다.

* 워크로드의 일부는 온-프레미스에서, 일부는 Azure에서 실행되는 하이브리드 응용 프로그램.
* 온-프레미스 데이터센터로부터 Azure VNet으로 들어오는 트래픽에 대한 세분화된 제어가 요구되는 인프라.
* 나가는 트래픽에 대한 감사가 필요한 응용 프로그램. 나가는 트래픽에 대한 감사는 많은 상업용 시스템의 규제 요건인 경우가 많습니다. 이는 개인 정보가 외부로 공개되는 것을 방지하는데 기여합니다.

## 아키텍처 다이어그램
다음 다이어그램은 이 아키텍처의 중요한 요소들을 보여줍니다.

> [Microsoft 다운로드 센터][visio-download]에서 이 아키텍처 다이어그램을 포함한 Visio 문서를 다운로드할 수 있습니다. 이 다이어그램은 "DMZ - Private" 페이지에 있습니다.
> 
> 

[![0]][0] 

* **온-프레미스 네트워크**. 조직에 구현된 사설 로컬 영역 네트워크.
* **Azure 가상 네트워크(VNet)**. VNet은 Azure에서 실행되는 응용 프로그램과 기타 리소스를 호스팅합니다.
* **게이트웨이**. 게이트웨이는 온-프레미스 네트워크와 VNet의 라우터 간 연결성을 제공합니다.
* **네트워크 가상 어플라이언스(NVA)**. 네트워크 가상 어플라이언스(NVA)는 방화벽으로서 액세스의 허용 또는 거부, (네트워크 압축을 포함한) 광역 네트워크(WAN) 작업의 최적화, 사용자 지정 라우팅 또는 기타 네트워크 기능과 같은 작업을 수행하는 가상 컴퓨터를 말합니다.
* **웹 계층, 비즈니스 계층 및 데이터 계층 서브넷**. 클라우드에서 실행되는 3계층 응용 프로그램을 구현하는 VM 및 서비스를 호스팅하는 서브넷. [Azure에서 N계층 아키텍처를 위한 Windows VM 실행][ra-n-tier]을 참조하시기 바랍니다.
* **사용자 지정 루트(UDR)**. [사용자 지정 루트][udr-overview]는 Azure VNet 내 IP 트래픽 흐름을 정의합니다.

    > [!참고]
    > VPN 연결 요구사항에 따라 UDR 대신 경계 게이트웨이 프로토콜(BGP)을 구성하여 온-프레미스 네트워크를 통해 트래픽을 되돌리는 포워딩 규칙을 구현할 수 있습니다.
    > 
    > 

* **관리 서브넷.** 이 서브넷은 VNet에서 실행되는 구성요소를 위한 관리 및 모니터링 기능을 구현하는 VM을 포함합니다.

## 권장사항

다음 권장사항은 대부분의 시나리오에 적용됩니다. 다른 구체적인 요구사항이 없다면 가급적 권장사항을 따르시기 바랍니다.

### 액세스 제어 권장사항

[역할 기반 액세스 제어][rbac] (RBAC)를 사용하여 응용 프로그램의 리소스를 관리합니다. 다음과 같은 [사용자 지정 역할][rbac-custom-roles]의 생성을 고려해 보시기 바랍니다.

- 응용 프로그램을 위한 인프라 관리, 응용 프로그램 구성요소 배포, VM 모니터링 및 재시작 권한이 있는 DevOps 역할.  

- 네트워크 리소스를 관리하고 모니터링하는 중앙집중식 IT 관리자 역할.

- NVA와 같은 네트워크 리소스 보안 관리를 위한 보안 IT 관리자 역할. 

DevOps 및 IT 관리자 역할에는 NVA 리소스 액세스 권한이 없어야 합니다. 이 권한은 보안 IT 관리자 역할로 한정되어야 합니다. 

### 리소스 그룹 권장사항
VM, VNet, 부하 분산 장치와 같은 Azure 리소스를 리소스 그룹으로 묶어서 보다 쉽게 관리할 수 있습니다. 각각의 리소스 그룹에 RBAC 역할을 할당하여 액세스를 제한합니다.

다음과 같은 리소스 그룹 생성을 권장합니다.

* (VM을 제외한) 서브넷, 네트워크 보안 그룹, 온-프레미스 네트워크 연결을 위한 게이트웨이 리소스를 포함하는 리소스 그룹.  이 리소스 그룹에 중앙집중식 IT 관리자 역할을 할당합니다.
* (부하 분산 장치를 포함한) NVA용 VM, 점프박스 및 기타 관리 VM, 모든 트래픽이 NVA를 통하도록 강제하는 게이트웨이 서브넷을 위한 UDR을 포함하는 리소스 그룹. 이 리소스 그룹에 보안 IT 관리자 역할을 할당합니다.
* 부하 분산 장와 VM을 포함하는 응용 프로그램 계층별 리소스 그룹. 이 리소스 그룹에는 계층별 서브넷이 포함되지 않아야 합니다. 이 리소스 그룹에 DevOps 역할을 할당합니다.

### 가상 네트워크 게이트웨이 권장사항

온-프레미스 트래픽은 가상 네트워크 게이트웨이를 통해 VNet으로 전달됩니다. [Azure VPN 게이트웨이][guidance-vpn-gateway] 또는 [Azure ExpressRoute 게이트웨이][guidance-expressroute]의 사용을 권장합니다.

### 네트워크 가상 어플라이언스(NVA) 권장사항

NVA는 네트워크 트래픽을 관리하고 모니터링하기 위한 다양한 서비스를 제공합니다. Azure Marketplace는 다음을 포함한 여러 타사 NVA 제품을 제공합니다.

* [Barracuda Web Application Firewall][barracuda-waf] 및 [Barracuda NextGen Firewall][barracuda-nf]
* [Cohesive Networks VNS3 Firewall/Router/VPN][vns3]
* [Fortinet FortiGate-VM][fortinet]
* [SecureSphere Web Application Firewall][securesphere]
* [DenyAll Web Application Firewall][denyall]
* [Check Point vSEC][checkpoint]
* [Kemp LoadMaster Load Balancer ADC Content Switch][kemp-loadmaster]

이들 중 귀하의 요구사항을 만족하는 제품이 없다면 VM을 사용하여 사용자 지정 NVA를 생성할 수도 있습니다. 사용자 지정 NVA 생성 예로서, 이 참조 아키텍처를 위한 솔루션 배포는 다음과 같은 기능을 구현합니다.

* NVA 네트워크 인터페이스(NIC)의 [IP 포워딩][ip-forwarding]을 사용하여 트래픽을 라우팅합니다.
* 적절한 경우에만 트래픽이 NVA를 통과하도록 허용됩니다. 이 참조 아키텍처의 각각의 NVA VM은 단순한 Linux 라우터입니다. 들어오는 트래픽은 네트워크 인터페이스 *eth0*에 도착하고 나가는 트래픽은 네트워크 인터페이스 *eth1*을 통해 보내진 사용자 지정 스크립트에 정의된 규칙에 부응합니다.
* 이 NVA들은 관리 서브넷을 통해서만 구성할 수 있습니다. 
* 관리 서브넷으로 라우팅된 트래픽은 NVA를 통과하지 않습니다. 그렇지 않으면, NVA에 장애가 발생할 경우 이를 해결하기 위한 관리 서브넷으로의 루트가 없게 됩니다. 
* 이 NVA를 위한 VM은 부하 분산 장치 뒤의 [가용성 집합][availability-set]에 배치됩니다. 게이트웨이 서브넷의 UDR은 NVA 요청을 부하 분산 장치로 보냅니다.

NVA 수준에서 응용 프로그램 연결을 종료하고 백엔드 계층 선호도를 유지하기 위해 레이어7 NVA를 포함시킵니다. 이를 통해 백엔드 계층으로부터의 응답 트래픽이 NVA를 통해 돌아오는 대칭 연결이 보장됩니다. 

또 다른 선택 가능한 방법은 각각 특화된 보안 작업을 수행하는 여러 NVA를 직렬로 연결하는 것입니다. 이를 통해 각 보안 기능이 NVA 단위로 관리될 수 있습니다. 예를 들어, 방화벽을 구현하는 NVA를 계정 서비스를 실행하는 NVA와 함께 직렬로 배치할 수 있습니다. 네트워크 홉을 추가하면 관리 용이성이 개선되지만 대신 대기 시간이 늘어날 수 있으므로 응용 프로그램의 성능에 영향을 미치지 않도록 주의하시기 바랍니다. 


### 네트워크 보안 그룹(NSG) 권장사항

VPN 게이트웨이는 온-프레미스 네트워크로의 연결을 위해 공용 IP 주소를 노출시킵니다. 따라서 온-프레미스 네트워크가 아닌 다른 곳에서 오는 모든 트래픽을 차단하는 규칙을 가진 수신 NVA 서브넷을 위한 네트워크 보안 그룹(NSG)을 만드는 것을 권장합니다. 

또한 부적절하게 구성되었거나 폐기된 NVA를 우회하는 수신 트래픽으로부터 시스템을 보호하기 위해 서브넷별로 네트워크 보안 그룹을 사용하는 것도 권장합니다. 예를 들어, 이 참조 아키텍처의 웹 계층 서브넷은 온-프레미스 네트워크(192.168.0.0/16) 또는 VNet으로부터 수신하지 않은 모든 요청을 무시하는 규칙과 포트 80을 통하지 않은 모든 요청을 무시하는 규칙을 가진 NSG를 구현합니다.

### 인터넷 액세스 권장사항

사이트 간 VPN 터널을 사용하여 모든 송신 트래픽을 온-프레미스 네트워크로 [강제 터널링][azure-forced-tunneling]하고 네트워크 주소 변환(NAT)을 통해 인터넷으로 라우팅합니다. 이를 통해 데이터 계층에 저장된 기밀 정보 유출을 방지하고 모든 나가는 트래픽에 대한 검사 및 감사를 수행할 수 있습니다. 

> [!참고]
> VM 진단 로깅, VM 확장 프로그램 다운로드 등과 같은 공용 IP 주소에 의존하는 Azure PaaS 서비스를 사용하려면 응용 프로그램 계층으로부터 인터넷 트래픽을 완전히 차단해서는 안 됩니다. Azure 진단을 이용하려면 구성요소들이 Azure 저장소 계정에 대해 쓰기와 읽기를 수행할 수 있어야 합니다. 
> 
> 

나가는 인터넷 트래픽이 정확하게 강제 터널링되는지 확인합니다. 온-프레미스 서버에서 [라우팅 및 원격 액세스 서비스][routing-and-remote-access-service]를 제공하는 VPN 연결을 사용한다면 [WireShark][wireshark]나 [Microsoft Message Analyzer](https://www.microsoft.com/download/details.aspx?id=44226)와 같은 도구를 사용하시기 바랍니다.

### 관리 서브넷 권장사항

관리 서브넷은 관리 및 모니터링 기능을 수행하는 점프박스를 포함합니다. 점프박스에 대한 모든 보안 관리 작업의 실행을 제한합니다. 
 
점프박스를 위한 공용 IP 주소를 만들지 않습니다. 대신, 들어오는 게이트웨이를 통해 점프박스에 액세스하는 단일 루트를 만듭니다. 관리 서브넷이 허용된 루트로부터 오는 요청에만 응답하도록 하는 NSG 규칙을 생성합니다. 

## 확장성 고려사항

이 참조 아키텍처는 부하 분산 장치를 사용하여 온-프레미스 네트워크 트래픽을 트래픽을 라우팅하는 NVA 장치 풀로 전달합니다. 이 NVA들은 하나의 [가용성 집합][availability-set]에 배치됩니다. 이러한 설계를 통해 일정 시간 동안의 NVA 처리율을 모니터링하고 부하 증가 시 NVA 장치를 추가할 수 있습니다.

표준 SKU VPN 게이트웨이는 최대 100Mbps의 연속 처리율을 제공합니다. 고성능 SKU는 최대 200 Mbps의 연속 처리율을 제공합니다.  더 높은 대역폭이 필요한 경우 ExpressRoute 게이트웨이로 업그레이드할 것을 고려할 필요가 있습니다. ExpressRoute는 최대 10 Gpbs의 대역폭을 제공하는 동시에 VPN 연결보다 대기 시간이 더 짧습니다.

Azure 게이트웨이의 확장성에 대한 자세한 내용은 [Azure와 온-프레미스 VPN으로 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-scalability] 및 [Azure ExpressRoute로 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-scalability]을 참조하시기 바랍니다.

## 가용성 고려사항

앞서 언급했듯이 이 참조 아키텍처는 부하 분산 장치 뒤에 여러 NVA 장치의 풀을 사용합니다. 부하 분산 장치는 상태 프로브를 사용하여 각각의 NVA를 모니터링하고 응답하지 않는 NVA는 풀에서 삭제합니다. 

Azure ExpressRoute를 사용하여 VNet과 온-프레미스 네트워크 간 연결성을 제공하는 경우, ExpressRoute 연결이 사용 불가 상태일 때는 [VPN 게이트웨이가 장애조치 기능을 제공하도록 구성해 보시기 바랍니다][ra-vpn-failover].

VPN 및 ExpressRoute 연결을 위한 가용성 유지에 관한 자세한 내용은 [Azure와 온-프레미스 VPN으로 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-availability] 및 [Azure ExpressRoute로 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-availability]을 참조하시기 바랍니다. 

## 관리 효율성 고려사항

모든 응용 프로그램 및 리소스에 대한 모니터링은 관리 서브넷의 점프박스를 통해 수행되어야 합니다. 응용 프로그램 요구사항에 따라 관리 서브넷에 추가적인 모니터링 리소스가 필요할 수도 있습니다. 이 경우 이 리소스들은 점프박스를 통해 액세스되어야 합니다. 

온-프레미스 네트워크에서 Azure로의 게이트웨이 연결이 중단되는 경우에도 공용 IP 주소를 배포하고 점프박스에 추가한 후 인터넷으로부터 원격 접속을 통해 점프박스에 액세스할 수 있습니다. 

이 참조 아키텍처의 각 계층의 서브넷은 NSG 규칙을 통해 보호됩니다. Windows VM에서 원격 데스크톱 프로토콜(RDP) 액세스를 위해 포트 3389를 열거나 Linux VM에서 SSH 액세스를 위해 포트 22를 여는 규칙을 생성해야 할 수 있습니다.  다른 관리 및 모니터링 도구의 경우 추가 포트를 여는 규칙이 필요할 수 있습니다. 

Azure ExpressRoute를 사용하여 온-프레미스 네트워크와 Azure 간 연결성을 제공하는 경우, [Azure Connectivity Toolkit (AzureCT)][azurect]을 사용하여 연결 문제를 모니터링하고 해결하시기 바랍니다.

VPN 및 ExpressRoute 연결의 모니터링 및 관리에 대한 추가 정보는 [Azure와 온-프레미스 VPN으로 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-manageability] 및 [Azure ExpressRoute로 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-manageability]을 참조하시기 바랍니다.

## 보안 고려사항

이 참조 아키텍처는 여러 수준의 보안을 구현합니다.

### 모든 온-프레미스 사용자 요청을 NVA를 통해 라우팅
게이트웨이 서브넷의 UDR은 온-프레미스가 아닌 곳으로부터 수신한 모든 사용자 요청을 차단합니다. UDR은 허용된 요청을 사설 DMZ 서브넷의 NVA로 보내고, 이 요청들은 NVA 규칙을 만족하는 경우에만 해당 응용 프로그램으로 전달됩니다. UDR에 접근하는 다른 루트를 추가할 수도 있습니다. 단, 이 루트들이 부주의로 인해 NVA를 우회하거나 관리 서브넷으로 가야할 관리 트래픽을 차단하지 않도록 주의합니다. 

NVA 앞의 부하 분산 장치 역시 부하 분상 규칙에서 열려있지 않은 포트의 트래픽을 무시하는 방식으로 보안 장치로서의 역할을 수행합니다. 이 참조 아키텍처의 부하 분산 장치는 포트 80의 HTTP 요청과 포트 443의 HTTPS 요청만을 수용합니다. 부하 분산 장치에 추가하는 모든 규칙을 문서화하고 보안 문제가 발생하지 않도록 트래픽을 모니터링합니다.

### 네트워크 보안 그룹(NSG)을 사용하여 응용 프로그램 계층 간 트래픽을 차단/허용
NSG를 사용하여 계층 간 트래픽을 제한할 수 있습니다. 비즈니스 계층은 웹 계층이 아닌 곳에서 오는 모든 트래픽을 차단하고, 데이터 계층은 비즈니스 계층이 아닌 곳에서 오는 모든 트래픽을 차단합니다. 이 계층들에 대해 보다 넓은 범위의 액세스를 허용하기 위해 NSG 규칙을 확대할 필요가 있다면, 이러한 요구사항과 보안 위험 간의 득실을 따져보시기 바랍니다. 새로운 수신 경로가 추가되면 사고 또는 고의에 의한 데이터 유출이나 응용 프로그램 손상의 가능성도 커집니다. 

### DevOps 액세스
[RBAC][rbac]를 사용하여 DevOps가 각 계층에 대해 수행하는 작업을 제한할 수 있습니다. 권한을 부여할 때는 [최소 권한의 원칙][security-principle-of-least-privilege]을 적용하시기 바랍니다. 모든 관리 작업의 로그를 작성하고 정기적인 감사를 수행하여 모든 구성 변경이 계획되도록 합니다. 

## 솔루션 배포

이러한 권장사항을 구현하는 참조 아키텍처 배포는 [GitHub][github-folder]를 통해 수행할 수 있습니다. 이 참조 아키텍처는 아래 지침에 따라 배포할 수 있습니다.

1. 아래 버튼을 마우스 오른쪽 단추로 클릭한 후 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-hybrid%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열리면 일부 설정값을 입력합니다. 
   * **리소스 그룹 ** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-private-dmz-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 확인란을 클릭합니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** checkbox.
   * **구입** 버튼을 클릭합니다.
3. 배포가 완료될 때까지 기다립니다.
4. 매개변수 파일에는 모든 VM에 대한 하드 코딩된 관리자 사용자 이름 및 암호가 포함되어 있는데, 이 둘 모두를 즉시 변경할 것을 권장합니다. Azure 포털에서 배포의 각 VM을 선택한 후 **지원 + 문제해결** 블레이드에서 **암호 재설정**을 클릭합니다. **모드** 드롭다운 상자에서 **암호 재설정**을 선택한 후 새 **사용자 이름** 및 **암호**를 선택합니다. **업데이트** 버튼을 클릭하여 저장합니다.

## 다음 단계

* [Azure와 인터넷 간 DMZ](secure-vnet-dmz.md)를 구현하는 방법에 대해 알아보시기 바랍니다.
* [고가용성 하이브리드 네트워크 아키텍처][ra-vpn-failover]를 구현하는 방법에 대해 알아보시기 바랍니다.
* Azure의 네트워크 보안 관리에 관한 자세한 내용은 [Microsoft 클라우드 서비스 및 네트워크 보안][cloud-services-network-security]을 참조하시기 바랍니다.
* Azure에서의 리소스 보호에 대한 자세한 내용은 [Microsoft Azure 보안 시작하기][getting-started-with-azure-security]를 참조하시기 바랍니다. 
* Azure 게이트웨이 연결 보안 문제 해결에 관한 자세한 내용은 [Azure와 온-프레미스 VPN으로 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-security] 및 [Azure ExpressRoute로 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-security]을 참조하시기 바랍니다.
> 

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[azure-forced-tunneling]: https://azure.microsoft.com/en-gb/documentation/articles/vpn-gateway-forced-tunneling-rm/
[barracuda-nf]: https://azure.microsoft.com/marketplace/partners/barracudanetworks/barracuda-ng-firewall/
[barracuda-waf]: https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/
[checkpoint]: https://azure.microsoft.com/marketplace/partners/checkpoint/check-point-r77-10/
[cloud-services-network-security]: https://azure.microsoft.com/documentation/articles/best-practices-network-security/
[denyall]: https://azure.microsoft.com/marketplace/partners/denyall/denyall-web-application-firewall/
[fortinet]: https://azure.microsoft.com/marketplace/partners/fortinet/fortinet-fortigate-singlevmfortigate-singlevm/
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
[kemp-loadmaster]: https://azure.microsoft.com/marketplace/partners/kemptech/vlm-azure/
[ra-expressroute]: ../hybrid-networking/expressroute.md
[ra-n-tier]: ../virtual-machines-windows/n-tier.md
[ra-vpn]: ../hybrid-networking/vpn.md
[ra-vpn-failover]: ../hybrid-networking/expressroute-vpn-failover.md
[rbac]: /azure/active-directory/role-based-access-control-configure
[rbac-custom-roles]: /azure/active-directory/role-based-access-control-custom-roles
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[routing-and-remote-access-service]: https://technet.microsoft.com/library/dd469790(v=ws.11).aspx
[securesphere]: https://azuremarketplace.microsoft.com/en-us/marketplace/apps/imperva.securesphere-waf
[security-principle-of-least-privilege]: https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx
[vns3]: https://azure.microsoft.com/marketplace/partners/cohesive/cohesiveft-vns3-for-azure/
[wireshark]: https://www.wireshark.org/
[0]: ../_images/blueprints/hybrid-network-secure-vnet.png "Secure hybrid network architecture"
