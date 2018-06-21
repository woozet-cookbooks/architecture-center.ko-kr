---
title: 고가용성 하이브리드 네트워크 아키텍처 구현
description: VPN Gateway 장애 조치(failover)를 사용하는 ExpressRoute를 사용하여 연결된 Azure 가상 네트워크 및 온-프레미스 네트워크를 포함하는 보안 사이트 간 네트워크 아키텍처를 구축하는 방법
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 81298215c814cee805eff57fdc28f7c127148b5f
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/30/2018
ms.locfileid: "30270441"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a>VPN 장애 조치(failover)를 사용하는 ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결

이 참조 아키텍처는 사이트 간 VPN(가상 사설망)을 장애 조치(failover) 연결로 사용하여 ExpressRoute를 통해 온-프레미스 네트워크를 Azure VNet(Virtual Network)에 연결하는 방법을 보여 줍니다. ExpressRoute 연결을 통해 온-프레미스 네트워크와 Azure VNet 사이에서 트래픽이 흐릅니다. ExpressRoute 회로의 연결이 끊어진 경우 트래픽은 IPSec VPN 터널을 통해 라우팅됩니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

ExpressRoute 회로를 사용할 수 없는 경우 VPN 경로가 개인 피어링 연결만 처리합니다. 공용 피어링 및 Microsoft 피어링 연결은 인터넷을 통과합니다. 

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>건축 

이 아키텍처는 다음 구성 요소로 구성됩니다.

* **온-프레미스 네트워크**. 조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.

* **VPN 어플라이언스**. 온-프레미스 네트워크에 외부 연결을 제공하는 장치 또는 서비스입니다. VPN 어플라이언스는 하드웨어 장치일 수도 있고 Windows Server 2012의 RRAS(라우팅 및 원격 액세스 서비스)와 같은 소프트웨어 솔루션일 수도 있습니다. 지원되는 VPN 어플라이언스 목록 및 선택한 VPN 어플라이언스를 Azure에 연결하도록 구성하는 방법에 대한 자세한 내용은 [사이트 간 VPN Gateway 연결에 대한 VPN 장치 정보][vpn-appliance]를 참조하세요.

* **ExpressRoute 회로**. 에지 라우터를 통해 Azure에서 온-프레미스 네트워크를 연결하는 연결 공급자가 제공하는 레이어 2 또는 레이어 3 회로입니다. 이 회로는 연결 공급자가 관리하는 하드웨어 인프라를 사용합니다.

* **ExpressRoute 가상 네트워크 게이트웨이**. ExpressRoute 가상 네트워크 게이트웨이를 사용하면 VNet을 온-프레미스 네트워크에 연결하는 데 사용되는 ExpressRoute 회로에 연결할 수 있습니다.

* **VPN 가상 네트워크 게이트웨이**. VPN 가상 네트워크 게이트웨이를 사용하면 VNet을 온-프레미스 네트워크의 VPN 어플라이언스에 연결할 수 있습니다. VPN 가상 네트워크 게이트웨이는 VPN 어플라이언스를 통해서만 온-프레미스 네트워크의 요청을 수락하도록 구성되어 있습니다. 자세한 내용은 [온-프레미스 네트워크를 Microsoft Azure Virtual Network에 연결][connect-to-an-Azure-vnet]을 참조하세요.

* **VPN 연결**. 이 연결에는 연결 형식(IPSec) 및 트래픽을 암호화하기 위해 온-프레미스 VPN 어플라이언스와 공유되는 키를 지정하는 속성이 있습니다.

* **Azure VNet(Virtual Network)**. 각 VNet은 단일 Azure 지역에 상주하며 여러 응용 프로그램 계층을 호스트할 수 있습니다. 응용 프로그램 계층은 각 VNet의 서브넷을 사용하여 분할할 수 있습니다.

* **게이트웨이 서브넷**. 가상 네트워크 게이트웨이는 동일한 서브넷에 있습니다.

* **클라우드 응용 프로그램**. Azure에 호스팅된 응용 프로그램입니다. Azure Load Balancer를 통해 여러 서브넷이 연결된 여러 계층이 포함될 수 있습니다. 응용 프로그램 인프라에 대한 자세한 내용은 [Windows VM 워크로드 실행][windows-vm-ra] 및 [Linux VM 워크로드 실행][linux-vm-ra]을 참조하세요.

## <a name="recommendations"></a>권장 사항

대부분의 시나리오의 경우 다음 권장 사항을 적용합니다. 이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.

### <a name="vnet-and-gatewaysubnet"></a>VNet과 GatewaySubnet

동일한 VNet에서 ExpressRoute 가상 네트워크 게이트웨이와 VPN 가상 네트워크 게이트웨이를 만듭니다. 즉, 이름이 *GatewaySubnet*인 동일한 서브넷을 공유해야 한다는 의미입니다.

VNet에 이름이*GatewaySubnet*인 서브넷이 이미 포함되어 있는 경우 /27 이상의 주소 공간이 있어야 합니다. 기존 서브넷이 너무 작은 경우 다음 PowerShell 명령을 사용하여 서브넷을 제거합니다. 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

VNet에 이름이 **GatewaySubnet**인 서브넷이 없는 경우 다음 Powershell 명령을 사용하여 새 서브넷을 만듭니다.

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a>VPN 및 ExpressRoute 게이트웨이

조직이 Azure에 연결하기 위한 [ExpressRoute 필수 조건 요구 사항][expressroute-prereq]을 충족하는지 확인합니다.

Azure VNet에 VPN 가상 네트워크 게이트웨이가 이미 포함되어 있는 경우 다음과 같은 Powershell 명령을 사용하여 제거합니다.

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

[Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][implementing-expressroute]의 지침을 따라 ExpressRoute 연결을 설정합니다.

[Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][implementing-vpn]의 지침을 따라 VPN 가상 네트워크 게이트웨이 연결을 설정합니다.

가상 네트워크 게이트웨이 연결을 설정한 후 다음과 같이 환경을 테스트합니다.

1. 온-프레미스 네트워크에서 Azure VNet으로 연결할 수 있는지 확인합니다.
2. 테스트를 위해 ExpressRoute 연결을 중지하려면 공급자에게 문의합니다.
3. VPN 가상 네트워크 게이트웨이 연결을 사용하여 온-프레미스 네트워크에서 Azure VNet으로 계속 연결할 수 있는지 확인합니다.
4. ExpressRoute 연결을 재설정하려면 공급자에게 문의합니다.

## <a name="considerations"></a>고려 사항

ExpressRoute 고려 사항은 [Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-expressroute] 지침을 참조하세요.

사이트 간 VPN 고려 사항은 [Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-vpn] 지침을 참조하세요.

일반적인 Azure 보안 고려 사항은 [Microsoft Cloud Services 및 네트워크 보안][best-practices-security]을 참조하세요.

## <a name="deploy-the-solution"></a>솔루션 배포

**필수 조건.** 적절한 네트워크 어플라이언스를 사용하여 기존 온-프레미스 인프라가 이미 구성된 상태여야 합니다.

솔루션을 배포하려면 다음 단계를 수행합니다.

1. 아래 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure Portal에서 링크가 열릴 때까지 기다린 후 다음 단계를 수행합니다.   
   * **리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-hybrid-vpn-er-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 하위 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구매** 단추를 클릭합니다.
3. 배포가 완료될 때가지 기다립니다.
4. 아래 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Azure Portal에 링크가 열릴 때까지 기다린 후 다음 단계를 수행합니다.
   * **리소스 그룹** 섹션에서 **기존 항목 사용**을 선택하고 텍스트 상자에 `ra-hybrid-vpn-er-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 하위 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구매** 단추를 클릭합니다.

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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "ExpressRoute 및 VPN Gateway를 사용하는 고가용성 하이브리드 네트워크 아키텍처의 아키텍처"
