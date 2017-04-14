---
title: Implementing a highly available hybrid network architecture
description: >-
  How to implement a secure site-to-site network architecture that spans an
  Azure virtual network and an on-premises network connected using ExpressRoute
  with VPN gateway failover.
author: telmosampaio
ms.service: guidance
ms.topic: article
ms.date: 11/28/2016
ms.author: pnp

pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
---
# VPN 장애조치를 지원하는 ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결

이 참조 아키텍처는 사이트 간 가상 사설 네트워크(VPN)를 장애조치 연결로 사용하는 ExpressRoute를 통해 온-프레미스 네트워크를 Azure에 연결하는 방법을 보여줍니다. 트래픽은 ExpressRoute 연결을 통해 온-프레미스 네트워크와 Azure 가상 네트워크(VNet) 사이를 흐릅니다. ExpressRoute 회로에서 연결이 중단되면, 트래픽은 IPSec VPN 터널을 통해 라우팅됩니다. [**이 솔루션 배포하기**.](#deploy-the-solution)

ExpressRoute 회로를 사용할 수 없는 경우 VPN 루트는 사설 피어링 연결만 처리하게 됩니다. 공개 피어링 및 Microsoft 피어링 연결은 인터넷을 통해 이루어집니다. 

![[0]][0]

## 아키텍처 

이 아키텍처는 다음과 같은 요소들로 구성되어 있습니다.

* **온-프레미스 네트워크**. 조직 내에서 실행되는 사설 로컬 영역 네트워크.

* **VPN 어플라이언스**. 온-프레미스 네트워크로의 외부 연결을 제공하는 장치 또는 서비스. VPN 어플라이언스는 하드웨어 장치일 수도 있고, Windows Server 2012의 라우팅 및 원격 액세스 서비스(RRAS)와 같은 소프트웨어일 수도 있습니다. 지원되는 VPN 어플라이언스 목록 및 Azure 연결을 위해 선택된 VPN 어플라이언스 설정에 관한 정보는 사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치를 참조하세요.

* **ExpressRoute 회로**. 에지 라우트(edge router)를 통해 온-프레미스 네트워크를 Azure에 연결하는 연결성 공급자가 제공하는 레이어2 또는 레이어3 회로입니다. 이 회로는 연결성 공급자가 관리하는 하드웨어 인프라를 사용합니다.

* **ExpressRoute 가상 네트워크 게이트웨이**. ExpressRoute 가상 네트워크 게이트웨이를 통해 온-프레미스 네트워크 연결을 위해 사용되는 ExpressRoute 회로에 VNet을 연결할 수 있습니다.

* **VPN 가상 네트워크 게이트웨이**. VPN 가상 네트워크 게이트웨이를 통해 VNet을 온-프레미스 네트워크의 VPN 어플라이언스에 연결할 수 있습니다. VPN 가상 네트워크 게이트웨이는 VPN 어플라이언스를 통해서만 온-프레미스 네트워크로부터 오는 요청을 수용하도록 설정됩니다. 자세한 내용은 온-프레미스 네트워크를 Microsoft Azure 가상 네트워크에 연결을 참조하세요.

* **VPN 연결**. 연결은 연결 유형(IPSec)과 트래픽을 암호화기 위해 온-프레미스 VPN 어플라이언스와 공유하는 키를 지정하는 속성을 갖습니다.

* **Azure 가상 네트워크(VNet)**. 각 VNet은 단일 Azure 지역에 위치하며 여러 애플리케이션 계층을 호스팅할 수 있습니다. 애플리케이션 계층은 각 VNet의 서브넷을 사용하여 나눌 수 있습니다.

* **게이트웨이 서브넷**. 가상 네트워크 게이트웨이들은 동일한 서브넷에 배치됩니다.

* **클라우드 애플리케이션**. Azure에 호스팅된 애플리케이션. 여러 계층이 포함될 수 있고, 여러 서브넷이 Azure 부하 분산 장치를 통해 연결됩니다. For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].

You can download a [Visio file](https://aka.ms/arch-diagrams) of this architecture.

> [!참고]
> Azure는 [Resource Manager](/azure/azure-resource-manager/resource-group-overview) 와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 참조 아키텍처에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다.
> 
> 

## 추천

다음 권장사항은 대부분의 시나리오에 적용됩니다. 다른 구체적인 요구사항이 없다면 가급적 권장사항을 따르시기 바랍니다.

### VNet 및 GatewaySubnet

ExpressRoute 가상 네트워크 게이트웨이와 VPN 가상 네트워크 게이트웨이를 동일한 VNet에 생성합니다. 즉, 두 게이트웨이는 **GatewaySubnet**이라는 이름의 동일한 서브넷을 공유해야 합니다.

VNet이 이미 GatewaySubnet이라는 이름의 서브넷을 포함하고 있는 경우에는 해당 서브넷이 /27 이상의 주소 공간을 가져야 합니다. 기존 서브넷이 너무 작은 경우에는 다음 PowerShell 명령어를 사용하여 해당 서브넷을 삭제합니다. 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

VNet이 **GatewaySubnet**이라는 이름의 서브넷을 포함하지 않으면, 다음 PowerShell 명령어를 사용하여 새 서브넷을 생성합니다.

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### VPN 및 ExpressRoute 게이트웨이

귀하의 조직이 Azure 연결을 위한 [ExpressRoute 사전 요구사항][expressroute-prereq]을 충족하는지 확인하세요.

Azure VNet에 VPN 가상 네트워크 게이트웨이가 이미 있다면, 다음 PowerShell 명령어를 사용하여 삭제하세요.

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

[Azure ExpressRoute로 하이브리드 네트워크 아키텍처 구현][implementing-expressroute]에 있는 지침에 따라 ExpressRoute 연결을 구축합니다.

[Azure 및 온-프레미스 VPN으로 하이브리드 네트워크 아키텍처 구현][implementing-vpn] 에 있는 지침에 따라 VPN 가상 네트워크 게이트웨이 연결을 구축합니다.

가상 네트워크 게이트웨이 연결을 구축했으면, 다음과 같이 환경을 테스트합니다.

1. 온-프레미스 네트워크에서 Azure VNet으로 연결이 되는지 확인합니다.
2. 공급자에게 연락하여 테스트를 위해 ExpressRoute 연결을 중단합니다.
3. 여전히 VPN 가상 네트워크 게이트웨이 연결을 사용하여 온-프레미스 네트워크로부터 Azure VNet으로 연결이 가능한지 확인합니다.
4. 공급자에게 연락하여 ExpressRoute 연결을 재구축합니다.

## 고려사항

ExpressRoute연결에 대한 내용은 [Azure ExpressRoute로 하이브리드 네트워크 아키텍처 구현][guidance-expressroute] 가이드를 참조하세요.

사이트 간 VPN에 관한 고려사항은 [Azure 및 온-프레미스 VPN으로 하이브리드 네트워크 아키텍처 구현][guidance-vpn] 가이드를 참조하세요.

Azure 보안 고려사항은 [Microsoft 클라우드 서비스 및 네트워크 보안][best-practices-security]을 참조하세요.

## 솔루션 배포

**사전 준비 사항.** 적합한 네트워크 어플라이언스를 통해 이미 구성된 기존의 온-프레미스 인프라가 존재해야 합니다.

다음 절차를 통해 이 솔루션을 배포할 수 있습니다.

1. 아래 단추를 우클릭하여 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열리면 다음 절차를 따릅니다.   
   * **리소스 그룹** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-hybrid-vpn-er-rg`를 입력하세요.
   * **위치** 드롭다운 상자에서 지역을 선택하세요.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용약관을 검토한 후 **위에 명시된 사용약관에 동의함** 확인란을 클릭합니다.
   * **구입** 단추를 클릭합니다.
3. 명령이 완료될 때까지 기다립니다.
4. 아래 단추를 우클릭하여 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Azure 포털에서 링크가 열리면 엔터를 누른 후 아래의 절차를 따릅니다.
   * **리소스 그룹** 섹션에서 **기존 사용** 을 선택한 다음 텍스트 상자에 `ra-hybrid-vpn-er-rg`를 입력하세요.
   * **위치** 드롭다운 상자에서 지역을 선택하세요.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용약관을 검토한 후 **위에 명시된 사용약관에 동의함** 확인란을 클릭합니다.
   * **구입** 단추를 클릭합니다.

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md


[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute-vpn-failover/Deploy-ReferenceArchitecture.ps1
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx
[0]: ../_images/blueprints/hybrid-network-expressroute-vpn-failover.png "Architecture of a highly available hybrid network architecture using ExpressRoute and VPN gateway"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
