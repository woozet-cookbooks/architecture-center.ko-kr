---
title: Connect an on-premises network to Azure using ExpressRoute
description: >-
  How to implement a secure site-to-site network architecture that spans an
  Azure virtual network and an on-premises network connected using Azure
  ExpressRoute.
author: telmosampaio
ms.service: guidance
ms.topic: article
ms.date: 11/28/2016
ms.author: pnp

pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
---
# ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결

이 참조 아키텍처는 [Azure ExpressRoute][expressroute-introduction]를 사용하여 온-프레미스 네트워크를 Azure 가상 네트워크에 연결하는 방법을 보여줍니다. ExpressRoute 연결은 외부 연결성 공급자를 통해 사설 전용 연결을 사용합니다. 사설 연결을 통해 온-프레미스 네트워크를 Azure로 확장할 수 있습니다. [**이 솔루션 배포하기**.](#deploy-the-solution)

![[0]][0]

## 아키텍처

이 아키텍처는 다음과 같은 요소들로 구성되어 있습니다.

* **온-프레미스 기업 네트워크**. 조직 내에서 실행되는 사설 로컬 영역 네트워크.

* **ExpressRoute 회로**. 에지 라우트(edge router)를 통해 온-프레미스 네트워크를 Azure에 연결하는 연결성 공급자가 제공하는 레이어2 또는 레이어3 회로입니다. 이 회로는 연결성 공급자가 관리하는 하드웨어 인프라를 사용합니다.

* **로컬 에지 라우터**. 온-프레미스 네트워크를 제공자가 관리하는 회로에 연결시켜주는 라우터. 연결 프로비전 방식에 따라서는 라우터가 사용하는 공인 IP 주소를 제공해야 할 수도 있습니다.
* **Microsoft 에지 라우터**. 액티브-액티브의 고가용성 구성을 가진 두 개의 라우터. 이 라우터를 통해 연결성 공급자는 회로를 데이터센터에 직접 연결할 수 있습니다. 연결 프로비전 방식에 따라서는 라우터가 사용하는 공인 IP 주소를 제공해야 할 수도 있습니다.

* **Azure 가상 네트워크(VNet)**. 각 VNet은 단일 Azure 지역에 위치하며 여러 애플리케이션 계층을 호스팅할 수 있습니다. 애플리케이션 계층은 각 VNet의 서브넷을 사용하여 나눌 수 있습니다.

* **Azure 공개 서비스**. 하이브리드 애플리케이션 내에서 사용할 수 있는 Azure 서비스입니다. 이 서비스들은 인터넷에서도 사용할 수 있지만 ExpressRoute 회로를 사용하여 접속하면 트래픽이 인터넷을 거치지 않으므로 대기 시간을 줄이고 보다 예측 가능한 성능을 얻을 수 있습니다. 연결은 [공개 피어링][expressroute-peering] 및 조직이 소유하고 있거나 연결성 공급자가 제공하는 주소를 사용하여 수행됩니다.

* **Office 365 서비스**. Microsoft의 공개적으로 이용 가능한 Office 365 애플리케이션 및 서비스입니다. 연결은 [Microsoft 피어링][expressroute-peering] 및 조직이 소유하고 있거나 연결성 공급자가 제공하는 주소를 사용하여 수행됩니다.

* **연결성 공급자**. 레이어2 또는 레이어3 연결을 사용하여 귀하의 데이터 센터와 Azure 데이터센터 간 연결을 제공하는 회사입니다.

아키텍처 다이어그램은 Visio를 통해 [다운로드](https://aka.ms/arch-diagrams) 하실 수 있습니다.

> [!참고]
> Azure는 [리소스 매니저](/azure/azure-resource-manager/resource-group-overview)와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 문서에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다.
> 
> 

## 추천

다음 권장사항은 대부분의 시나리오에 적용됩니다. 다른 구체적인 요구사항이 없다면 가급적 권장사항을 따르시기 바랍니다.

### 연결성 공급자

귀하의 지역의 적합한 ExpressRoute 연결성 공급자를 선택하세요. 다음 Azure PowerShell 명령어를 사용하여 귀하의 지역에 있는 연결성 공급자 목록을 확인할 수 있습니다.

```powershell
Get-AzureRmExpressRouteServiceProvider
```

ExpressRoute 연결성 공급자는 다음과 같은 방법을 통해 귀하의 데이터센터를 Microsoft에 연결합니다.

* **클라우드 익스체인지 코로케이션(co-location)**. 클라우드 익스체인지를 사용하는 시설에 코로케이션 중이라면 코로케이션 공급자의 이더넷 익스체인지를 통해 Azure에 대한 가상 교차 연결을 주문할 수 있습니다. 코로케이션 공급자는 코로케이션 시설 내 귀하의 인프라와 Azure 간 레이어2 교차 연결 또는 매니지드 레이어3 교차 연결을 제공할 수 있습니다.
* **P2P 이더넷 연결**. P2P 이더넷 링크를 통해 온-프레미스 데이터센터/오피스를 Azure에 연결할 수 있습니다. P2P 이더넷 공급자는 귀하의 사이트와 Azure 간 레이어2 연결 또는 매니지드 레이어3 연결을 제공할 수 있습니다.
* **애니-투-애니(IPVPN) 네트워크**. 광역 네트워크(WAN)를 Azure와 통합할 수 있습니다. 인터넷 프로토콜 가상 사설 네트워크(IPVPN) 공급자(주로 멀티프로토콜 레이블 스위칭 VPN)는 지사와 데이터센터 간 애니-투-애니 연결을 제공합니다. Azure는 WAN에 상호 연결되어 여타의 지사처럼 보이게 할 수 있습니다. WAN 공급자는 일반적으로 매니지드 레이어3 연결을 제공합니다.

연결성 공급자에 대한 자세한 내용은 [ExpressRoute 소개][expressroute-introduction]를 참조하세요.

### ExpressRoute 회로

귀하의 조직이 Azure 연결을 위한 [ExpressRoute 사전 요구사항][expressroute-prereqs]을 충족하는지 확인하세요.

아직 충족되지 않았다면 GatewaySubnet이라는 이름의 서브넷을 Azure VNet에 추가하고 Azure VPN 게이트웨이 서비스를 이용하여 ExpressRoute 가상 네트워크 게이트웨이를 만듭니다. 이 프로세스에 대한 자세한 내용은 [회로 프로비전 및 회로 상태에 대한 ExpressRoute 워크플로우][ExpressRoute-provisioning]를 참조하세요.

다음과 같이 ExpressRoute 회로를 생성합니다.

1. 다음과 같은 PowerShell 명령어를 실행합니다.
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. 새 회로에 대한 서비스키를 서비스 공급자에게 전달합니다.

3. 공급자가 회로를 프로비전할 때까지 기다립니다. 회로의 프로비전 상태를 확인하려면 다음과 같은 PowerShell 명령어를 실행합니다.
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    회로가 준비되면 출력의 서비스 공급자 섹션의 프로비전 상태 필드가 NotProvisioned에서 Provisioned로 변경됩니다.

> [!참고]
> 레이어3 연결 사용자의 경우, 공급자는 사용자를 위한 라우팅을 구성하고 관리해야합니다. 공급자가 적합한 루트를 구현하기 위해 필요한 정보를 공급자에게 제공합니다.
> 
> 

4. 레이어2 연결 사용자의 경우,

    a. 구현하려는 각 피어링 유형에 대한 유효한 공인 IP 주소로 구성된 두 개의 /30 서브넷을 예약합니다. 이 /30 서브넷들은 회로에 사용되는 라우터에 대한 IP 주소를 공급하기 위해 사용됩니다. 사설, 공개 및 Microsoft 피어링을 구현하는 경우에는 유효한 공인 IP 주소를 가진 여섯 개의 /30 서브넷이 필요합니다.      

    b. ExpressRoute 회로에 대한 라우팅을 구성합니다. 구성하려는 각 피어링 유형(사설, 공개, Microsoft)에 대하여 다음과 같은 PowerShell 명령어를 실행합니다. 자세한 내용은 [ExpressRoute 회로를 위한 라우팅 생성 및 변경][configure-expressroute-routing]을 참조하세요.
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    c.	공개 피어링 및 Microsoft 피어링에 대한 네트워크 주소 변환(NAT)에 사용할 또 다른 유효한 IP 주소 풀을 예약합니다. 각 피어링에 대해 별도의 풀을 가지는 것을 권장합니다. 해당하는 풀을 연결성 공급자에게 알려 이 범위에 대한 경계 게이트웨이 프로토콜(BGP) 광고를 구성할 수 있도록 합니다.

5. 다음 PowerShell 명령어를 실행하여 사설 VNet을 ExpressRoute 회로에 연결합니다. 자세한 내용은 [가상 네트워크를 ExpressRoute 회로에 연결][link-vnet-to-expressroute]을 참조하세요.

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

서로 다른 지역에 위치한 여러 VNet을 동일한 ExpressRoute 회로에 연결할 수 있습니다. 단, 모든 VNet과 ExpressRoute 회로는 하나의 지정학적 지역에 있어야 합니다.

### 문제 해결 

온-프레미스나 사설 VNet의 설정 변경 없이 이전에 작동하던 ExpressRoute 회로가 연결이 중단된다면 연결성 공급자에게 연락하여 이 문제를 해결해야 합니다. 다음 PowerShell 명령어를 사용하여 ExpressRoute 회로가 프로비전되었는지 확인할 수 있습니다.

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

이 명령어의 출력은 아래와 같이 `ProvisioningState`, `CircuitProvisioningState`, `ServiceProviderProvisioningState`를 포함한 회로의 여러 속성들을 보여줍니다.

```
ProvisioningState                : Succeeded
Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : NotProvisioned
```

새 회로의 생성을 시도한 후 `ProvisioningState`가 `Succeeded`로 되어있지 않다면, 아래 명령어를 사용하여 회로를 삭제하고 다시 시도하세요.

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

공급자가 이미 회로를 프로비전했는데도 불구하고 `ProvisioningState`가 `Failed`로 되어있거나 `CircuitProvisioningState`가 `Enabled`가 아니라면, 공급자에게 연락하여 도움을 받으세요.

## 확장성 고려사항

ExpressRoute 회로는 네트워크 간 높은 대역폭 경로를 제공합니다. 일반적으로 대역폭이 높을 수록 비용이 증가합니다. 

ExpressRoute는 고객에게 계량 요금제와 무제한 데이터 요금제의 두 가지 [요금제][expressroute-pricing]를 제공합니다. 요금은 회로 대역폭에 따라 달라집니다. 사용 가능한 대역폭은 공급자에 따라 다를 수 있습니다. `Get-AzureRmExpressRouteServiceProvider`cmdlet을 사용하면 귀하의 지역의 공급자와 제공되는 대역폭을 확인할 수 있습니다.
 
단일 ExpressRoute 회로는 일정한 수의 피어링과 VNet 링크를 지원합니다. 자세한 내용은 [ExpressRoute 한도](/azure/azure-subscription-service-limits)를 참조하세요.

추가 요금을 내면 ExpressRoute Premium 애드온을 통해 추가 기능을 사용할 수 있습니다.

* 공개 및 사설 피어링 루트 한도 증가. 
* ExpressRoute 회로당 VNet 링크 수 증가. 
* 글로벌 서비스 연결성.

자세한 내용은 [ExpressRoute 가격 책정][expressroute-pricing]을 참조하세요. 

ExpressRoute 회로는 추가 비용 없이 구입한 대역폭 한도의 최대 두 배까지 일시적인 네트워크 버스트를 허용하도록 설계됩니다. 이러한 설계는 중복 링크를 통해 구현됩니다.  그러나 모든 연결성 공급자가 이 기능을 제공하지는 않습니다. 이 기능을 선택하기 전에 귀하의 연결성 공급자가 이 기능을 제공하는지 확인하세요.

일부 공급자가 대역폭 변경을 허용하더라도 처음부터 니즈보다 여유있는 대역폭을 선택하여 수요 증가에 대비하는 것이 좋습니다. 향후 대역폭을 늘려야하는 경우에는 두 가지 옵션 중 하나를 선택해야 합니다.

- 대역폭 증가. 가능한 한 이 옵션은 피하는 것이 좋으며, 모든 공급자가 동적인 대역폭 증가를 제공하는 것도 아닙니다.  그럼에도 불구하고 대역폭 증가가 필요하다면 공급자가 PowerShell 명령어를 통해 ExpressRoute 대역폭 속성 변경을 지원하는지 확인해 보세요. 만약 공급자가 이를 지원한다면, 아래의 명령어를 실행합니다.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    연결 중단 없이 대역폭을 늘릴 수 있습니다. 대역폭을 낮추는 경우에는 회로를 삭제한 후 새 구성으로 재생성해야 하므로 연결이 중단됩니다.

- 만약 요금제를 변경하거나 Premium 요금제로 업그레이드하려면 다음 명령을 실행합니다. Sku.Tier 속성은 Standard 또는 Premium, Sku.Name 속성은 MeteredData 또는 UnlimitedData 중 하나입니다.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!중요]
    > SKu.Name 속성이 Sku.Tier 및 Sku.Family와 일치해야 합니다. 패밀리와 계층을 변경하고 이름은 변경하지 않으면 연결을 사용할 수 없게 됩니다.  
    >
    >
    
    연결 중단 없이 SKU를 업그레이드할 수는 있지만 무제한 요금제에서 계량 요금제로 변경하는 것은 불가능합니다. SKU를 다운그레이드하는 경우에는 대역폭 소비가 Standard SKU의 기본 한도 내에 있어야 합니다. 

## 가용성 고려사항

ExpressRoute는 고가용성을 구현하기 위한 상시 대기 라우팅 프로토콜(HSRP)이나 가상 라우터 중복 프로토콜(VRRP)과 같은 라우터 중복 프로토콜을 지원하지 않습니다. 대신 ExpressRoute는 피어링당 BGP 세션 중복 페어를 사용합니다. 네트워크에 고가용성 연결을 구축하기 위해 Azure는 액티브-액티브로 구성된 두 개의 라우터(Microsoft 에지의 일부)에 대한 두 개의 중복 포트를 제공합니다. 

BGP 세션은 60초의 유휴 타임아웃을 사용하도록 기본 설정되어 있습니다. 세션 타임아웃이 세 번(총 180초) 발생하면 라우터는 Unavailable로 표시되고 모든 트래픽이 다른 라우터로 리디렉팅됩니다. 180초 타임 아웃은 중요한 애플리케이션에 적용하기에는 너무 길 수 있습니다. 이 경우에는 온-프레미스 라우터의 BGP 타임아웃 설정값을 줄일 수 있습니다.

공급자 유형, ExpressRoute 회로 수 및 구성하려는 가상 네트워크 게이트웨이 연결에 따라 다양한 방법으로 Azure 연결의 고가용성을 구성할 수 있습니다. 가용성 관련 옵션을 간략히 살펴보면 다음과 같습니다.

* 레이어2 연결 사용자의 경우에는 온-프레미스 네트워크에 액티브-액티브 구성의 중복 라우터를 배포합니다. 주 회로를 하나의 라우터에, 부 회로를 다른 라우터에 연결합니다. 이렇게 하면 연결 양 끝에서 고가용성 연결을 얻을 수 있습니다. ExpressRoute 서비스 수준 계약(SLA)을 요구하는 경우 이러한 구성은 반드시 필요합니다. 자세한 내용은 [Azure ExpressRoute를 위한 서비스 수준 계약(SLA)][sla-for-expressroute]을 참조하세요.

    다음 다이어그램은 주 회로와 부 회로에 연결된 중복 온-프레미스 라우터의 구성을 보여줍니다. 각 회로는 공개 피어링과 사설 피어링 주소 공간(앞서 설명했듯이 각각 한 쌍의 /30 주소 공간 지정)에 대한 트래픽을 처리합니다.

    ![[1]][1]

* 레이어3 연결 사용자인 경우에는 가용성을 처리하는 중복 BGP 세션이 제공되는지 확인합니다.

* VNet을 서로 다른 서비스 공급자가 공급하는 여러 ExpressRoute 회로에 연결합니다. 이 전략을 통해 추가적인 고가용성과 재해 복구력을 얻을 수 있습니다.

* 사이트 간 VPN을 ExpressRoute를 위한 장애조치 경로로 설정합니다. 이 옵션에 대한 자세한 내용은 [VPN 장애조치를 지원하는 ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결][highly-available-network-architecture]을 참조하세요.
 T이 옵션은 사설 피어링에만 적용할 수 있습니다. Azure와 Office 365 서비스의 경우 유일한 장애조치 경로는 인터넷입니다.

## 관리 효율성 고려사항

[Azure Connectivity Toolkit (AzureCT)][azurect]을 사용하여 온-프레미스 데이터센터와 Azure 간 연결성을 모니터링할 수 있습니다. 

## 보안 고려사항

Azure 연결에 대한 보안 옵션을 보안 관심사항과 규제 준수 니즈에 따라 다양한 방법으로 설정할 수 있습니다. 

ExpressRoute는 레이어3에서 실행됩니다. 애플리케이션 계층에서의 위협은 합법적인 리소스에 대한 트래픽을 제한하는 네트워크 보안 어플라이언스를 사용하여 방지할 수 있습니다. 또한 공개 피어링을 사용하는 ExpressRoute 연결은 온-프레미스로부터만 시작할 수 있습니다. 이를 통해 악성 서비스가 인터넷으로부터 온-프레미스 데이터에 접근하여 피해를 주는 것을 방지할 수 있습니다.

보안을 극대화하려면 온-프레미스 네트워크와 공급자 에지 라우터 사이에 네트워크 보안 어플라이언스를 추가합니다. 이를 통해 VNet으로부터 불법 트래픽이 유입되는 것을 제한할 수 있습니다.

![[2]][2]

감사 또는 규제 준수를 위해서는 VNet에서 실행되는 구성 요소에서 인터넷으로의 직접 액세스를 금지하고 [강제 터널링][forced-tuneling]을 구현하는 것이 필요할 수 있습니다. 이 경우 인터넷 트래픽은 감사를 실시할 수 있는 온-프레미스 실행 프록시를 통해 다시 리디렉팅되어야 합니다. 이 프록시는 불법 트래픽이 나가는 것을 차단하고 잠재적 악성 인바운드 트래픽을 필터링하도록 설정될 수 있습니다.

![[3]][3]

보안을 극대화하려면 VM에 대해 공인 IP 주소를 사용하도록 설정하는 대신 NSG를 사용하여 VM에 공개적으로 접속할 수 없도록 해야 합니다. VM은 내부 IP 주소를 통해서만 이용하도록 해야 합니다. 이 주소들은 ExpressRoute 네트워크를 통해 접속할 수 있고 온-프레미스 DevOps 담당자가 설정 또는 유지관리를 수행할 수 있습니다.

VM에 대한 관리 끝점을 외부 네트워크에 노출해야만 하는 경우에는 NSG 또는 액세스 제어 목록을 사용하여 이 포트들의 가시성을 IP 주소 또는 네트워크 화이트리스트에만 허용하세요.

> [!참고]
> Azure 포털을 통해 배포된 Azure VM은 기본적으로 로그인 접속을 제공하는 공인 IP 주소를 포함하도록 설정되어 있습니다. 
> 
> 


## 솔루션 배포

**사전 준비 사항** 적합한 네트워크 어플라이언스를 통해 이미 구성된 기존의 온-프레미스 인프라가 존재해야 합니다.

다음 절차를 통해 이 솔루션을 배포할 수 있습니다.

1. 아래 단추를 우클릭하여 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열리면 다음 절차를 따릅니다.
   
   * **리소스 그룹** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-hybrid-er-rg`를 입력하세요.   
   * **위치** 드롭다운 상자에서 지역을 선택하세요.
   
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용약관을 검토한 후 **위에 명시된 사용약관에 동의함** 확인란을 클릭합니다.
   
   * **구입** 단추를 클릭합니다.
3. 명령이 완료될 때까지 기다립니다.
4. 아래 단추를 우클릭하여 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Azure 포털에서 링크가 열리면 다음 절차를 따릅니다.
   
   * **리소스 그룹** 섹션에서 **기존 사용**을 선택한 다음 텍스트 상자에 `ra-hybrid-er-rg`를 입력하세요.
   
   * **위치** 드롭다운 상자에서 지역을 선택하세요.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용약관을 검토한 후 **위에 명시된 사용약관에 동의함** 확인란을 클릭합니다.
   
   * **구입** 단추를 클릭합니다.
6. 명령이 완료될 때까지 기다립니다.


<!-- links -->
[forced-tuneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md
[naming-conventions]: /azure/guidance/guidance-naming-conventions

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[azure-powershell]: /azure/powershell-azure-resource-manager
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute/v1_0/
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[sample-script]: #sample-solution-script
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[arm-templates]: /azure/resource-group-authoring-templates
[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/Deploy-ReferenceArchitecture.ps1
[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/deploy-reference-architecture.sh
[vnet-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/virtualNetwork.parameters.json
[virtualnetworkgateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/virtualNetworkGateway.parameters.json
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ../_images/guidance-hybrid-network-expressroute/figure1.png "Hybrid network architecture using Azure ExpressRoute"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "Using redundant routers with ExpressRoute primary and secondary circuits"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "Adding security devices to the on-premises network"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "Using forced tunneling to audit Internet-bound traffic"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "Locating the ServiceKey of an ExpressRoute circuit"  
