---
title: ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결
description: Azure ExpressRoute를 사용하여 연결된 온-프레미스 네트워크 및 Azure Virtual Network를 포괄하는 보안 사이트 간 네트워크 아키텍처를 구현하는 방법입니다.
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: ada07f399925da6da28b24260f5c73f1e106fd7d
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/30/2018
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a>ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결

이 참조 아키텍처는 [Azure ExpressRoute][expressroute-introduction]를 사용하여 온-프레미스 네트워크를 Azure의 가상 네트워크에 연결하는 방법을 보여 줍니다. ExpressRoute 연결은 타사 연결 공급자를 통해 개인 전용 연결을 사용합니다. 개인 연결은 온-프레미스 네트워크를 Azure로 확장합니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>건축

이 아키텍처는 다음 구성 요소로 구성됩니다.

* **온-프레미스 회사 네트워크**. 조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.

* **ExpressRoute 회로**. 에지 라우터를 통해 Azure에서 온-프레미스 네트워크를 연결하는 연결 공급자가 제공하는 레이어 2 또는 레이어 3 회로입니다. 이 회로는 연결 공급자가 관리하는 하드웨어 인프라를 사용합니다.

* **로컬 에지 라우터**. 온-프레미스 네트워크를 공급자가 관리하는 회로에 연결하는 라우터입니다. 연결이 프로비전된 방법에 따라 라우터에서 사용하는 공용 IP 주소를 제공해야 할 수도 있습니다.
* **Microsoft Edge 라우터**. 활성-활성 고가용성 구성의 두 라우터입니다. 이 라우터를 사용하면 연결 공급자가 해당 회로를 데이터 센터에 직접 연결할 수 있습니다. 연결이 프로비전된 방법에 따라 라우터에서 사용하는 공용 IP 주소를 제공해야 할 수도 있습니다.

* **Azure Virtual Network(VNet)**. 각 VNet은 단일 Azure 지역에 상주하며 여러 응용 프로그램 계층을 호스트할 수 있습니다. 각 VNet의 서브넷을 사용하여 응용 프로그램 계층을 분할할 수 있습니다.

* **Azure 공용 서비스**. 하이브리드 응용 프로그램 내에서 사용할 수 있는 Azure 서비스입니다. 이 서비스는 인터넷에서도 사용할 수 있지만, ExpressRoute 회로를 통해 액세스하는 경우 트래픽이 인터넷을 통과하지 않으므로 대기 시간이 줄고 성능이 더 예측 가능합니다. [공용 피어링][expressroute-peering]을 사용하여 연결이 수행되고, 조직이 주소를 소유하거나 연결 공급자가 제공합니다.

* **Office 365 서비스**. Microsoft에서 제공하는, 공개적으로 사용할 수 있는 Office 365 응용 프로그램 및 서비스입니다. [Microsoft 피어링][expressroute-peering]을 사용하여 연결이 수행되고, 조직이 주소를 소유하거나 연결 공급자가 제공합니다. Microsoft 피어링을 통해 Microsoft CRM Online에 직접 연결할 수도 있습니다.

* **연결 공급자**(표시되지 않음). 레이어 2 또는 레이어 3 연결을 사용하여 데이터 센터와 Azure 데이터 센터 간의 연결을 제공하는 회사입니다.

## <a name="recommendations"></a>권장 사항

대부분의 시나리오의 경우 다음 권장 사항을 적용합니다. 이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.

### <a name="connectivity-providers"></a>연결 공급자

해당 위치에 적합한 ExpressRoute 연결 공급자를 선택합니다. 해당 위치에서 사용할 수 있는 연결 공급자 목록을 가져오려면 다음 Azure PowerShell 명령을 사용합니다.

```powershell
Get-AzureRmExpressRouteServiceProvider
```

ExpressRoute 연결 공급자는 다음과 같은 방법으로 데이터 센터를 Microsoft에 연결합니다.

* **클라우드 Exchange에 함께 배치**. 클라우드 Exchange와 같은 시설에 있는 경우 공동 배치 공급자의 이더넷 Exchange를 통해 Azure에 대한 가상 교차 연결을 주문할 수 있습니다. 공동 배치 공급자는 공동 배치 시설의 인프라와 Azure 간에 레이어 2 교차 연결 또는 관리되는 레이어 3 교차 연결을 제공할 수 있습니다.
* **지점 간 이더넷 연결**. 지점 간 이더넷 연결을 통해 온-프레미스 데이터 센터/사무소를 Azure에 연결할 수 있습니다. 지점 간 이더넷 공급자는 사이트와 Azure 간에 레이어 2 연결 또는 관리되는 레이어 3 연결을 제공할 수 있습니다.
* **임의(IPVPN)의 네트워크**. WAN(광역 네트워크)과 Azure를 통합할 수 있습니다. IPVPN(인터넷 프로토콜 가상 사설망) 공급자(일반적으로 MPLS(Multi-Protocol Label Switching) VPN)는 지점과 데이터 센터 간의 임의 연결을 제공합니다. Azure를 WAN에 상호 연결하여 다른 지점처럼 보이도록 할 수 있습니다. WAN 공급자는 일반적으로 관리되는 레이어 3 연결을 제공합니다.

연결 공급자에 대한 자세한 내용은 [ExpressRoute 소개][expressroute-introduction]를 참조하세요.

### <a name="expressroute-circuit"></a>ExpressRoute 회로

조직이 Azure에 연결하기 위한 [ExpressRoute 필수 조건 요구 사항][expressroute-prereqs]을 충족하는지 확인합니다.

아직 수행하지 않은 경우 `GatewaySubnet` 서브넷을 Azure VNet에 추가하고 Azure VPN Gateway 서비스를 사용하여 ExpressRoute 가상 네트워크 게이트웨이를 만듭니다. 이 프로세스에 대한 자세한 내용은 [회로 프로비전 및 회로 상태에 대한 ExpressRoute 워크플로][ExpressRoute-provisioning]를 참조하세요.

다음과 같이 ExpressRoute 회로를 만듭니다.

1. 다음 PowerShell 명령을 실행합니다.
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. 새 회로의 `ServiceKey`을 서비스 공급자에게 보냅니다.

3. 공급자가 회로를 프로비전할 때까지 기다립니다. 회로의 프로비저닝 상태를 확인하려면 다음 PowerShell 명령을 실행합니다.
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    회로 준비가 완료되면 출력의 `Service Provider` 섹션에 있는 `Provisioning state` 필드가 `NotProvisioned`에서 `Provisioned`로 변경됩니다.

    > [!NOTE]
    > 레이어 3 연결을 사용하는 경우 공급자가 라우팅을 구성하고 관리해야 합니다. 공급자가 적절한 경로를 구현하는 데 필요한 정보를 제공합니다.
    > 
    > 

4. 레이어 2 연결을 사용하는 경우:

    1. 구현할 각 피어링 유형에 적합한 공용 IP 주소로 구성된 /30 서브넷 두 개를 예약합니다. 이러한 /30 서브넷은 회로에 사용되는 라우터의 IP 주소를 제공하는 데 사용됩니다. 개인, 공용 및 Microsoft 피어링을 구현하는 경우 유효한 공용 IP 주소가 포함된 /30 서브넷 6개가 필요합니다.     

    2. ExpressRoute 회로의 라우팅 구성 구성할 각 피어링 유형(개인, 공용 및 Microsoft)에 대해 다음 PowerShell 명령을 실행합니다. 자세한 내용은 [ExpressRoute 회로의 라우팅 만들기 및 수정][configure-expressroute-routing]을 참조하세요.
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. 공용 및 Microsoft 피어링의 NAT(Network Address Translation)에 사용할 다른 유효한 공용 IP 주소 풀을 예약합니다. 각 피어링에 다른 풀을 사용하는 것이 좋습니다. 연결 공급자에 풀을 지정하여 해당 범위에 대한 BGP(Border Gateway Protocol) 알림을 구성할 수 있도록 합니다.

5. 다음 PowerShell 명령을 실행하여 개인 VNet을 ExpressRoute 회로에 연결합니다. 자세한 내용은 [가상 네트워크를 ExpressRoute 회로에 연결][link-vnet-to-expressroute]을 참조하세요.

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

모든 VNet 및 ExpressRoute 회로가 동일한 지정학적 지역에 있는 한, 서로 다른 지역에 있는 여러 VNet을 동일한 ExpressRoute 회로에 연결할 수 있습니다.

### <a name="troubleshooting"></a>문제 해결 

온-프레미스의 구성 변경 없이 또는 개인 VNet 내에서 이전에 작동하던 ExpressRoute 회로가 현재 연결되지 않는 경우 연결 공급자에 문의하여 함께 문제를 해결해야 할 수 있습니다. 다음 PowerShell 명령을 사용하여 ExpressRoute 회로가 프로비전되었는지 확인합니다.

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

이 명령의 출력에는 아래 그림과 같이 `ProvisioningState`, `CircuitProvisioningState` 및 `ServiceProviderProvisioningState`를 포함하여 회로에 대한 여러 속성이 표시됩니다.

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

새 회로를 만들려고 시도한 후 `ProvisioningState`가 `Succeeded`로 설정되지 않은 경우 아래 명령을 사용하여 회로를 제거하고 다시 만듭니다.

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

공급자가 이미 회로를 프로비전했으며 `ProvisioningState`가 `Failed`로 설정되었거나 `CircuitProvisioningState`가 `Enabled`가 아닌 경우 공급자에게 추가 지원을 요청하세요.

## <a name="scalability-considerations"></a>확장성 고려 사항

ExpressRoute 회로는 네트워크 간에 높은 대역폭 경로를 제공합니다. 일반적으로 대역폭이 높을수록 비용이 증가합니다. 

ExpressRoute는 종량제 요금제와 무제한 데이터 요금제라는 두 가지 [요금제][expressroute-pricing]를 고객에게 제공합니다. 요금은 회로 대역폭에 따라 달라집니다. 사용 가능한 대역폭은 공급자에 따라 다를 수 있습니다. `Get-AzureRmExpressRouteServiceProvider` cmdlet을 사용하여 해당 지역에서 사용할 수 있는 공급자 및 공급자가 제공하는 대역폭을 확인합니다.
 
단일 ExpressRoute 회로에서 특정 개수의 피어링 및 VNet 연결을 지원할 수 있습니다. 자세한 내용은 [ExpressRoute 제한](/azure/azure-subscription-service-limits)을 참조하세요.

추가 요금을 지불할 경우 ExpressRoute Premium 추가 기능이 다음과 같은 몇 가지 추가 기능을 제공합니다.

* 공용 및 개인 피어링에 대한 경로 제한 증가 
* ExpressRoute 회로당 VNet 연결 수 증가 
* 서비스에 대한 전역 연결입니다.

자세한 내용은 [ExpressRoute 가격][expressroute-pricing]을 참조하세요. 

ExpressRoute 회로는 추가 비용 없이 확보한 대역폭 제한의 최대 2배까지 일시적 네트워크 버스트를 허용하도록 디자인되었습니다. 이 기능은 중복 연결을 사용하여 지원됩니다. 그러나 모든 연결 공급자가 이 기능을 지원하는 것은 아닙니다. 이 기능을 사용하기 전에 연결 공급자가 기능을 지원하는지 확인합니다.

일부 공급자는 대역폭 변경을 허용하지만, 요구를 충족하고 확장을 위한 여유 공간을 제공하는 초기 대역폭을 선택해야 합니다. 나중에 대역폭을 늘려야 하는 경우 다음 두 가지 옵션 중에서 선택해야 합니다.

- 대역폭 증가. 이 옵션은 가능한 한 피해야 하며, 모든 공급자가 동적 대역폭 증가를 허용하는 것은 아닙니다. 그러나 대역폭 증가가 필요한 경우 공급자에게 문의하여 Powershell 명령을 통한 ExpressRoute 대역폭 속성 변경을 지원하는지 확인합니다. 지원하는 경우 아래 명령을 실행합니다.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    연결 손실 없이 대역폭을 늘릴 수 있습니다. 대역폭을 다운그레이드하는 경우 회로를 삭제하고 새 구성으로 다시 만들어야 하기 때문에 연결이 중단됩니다.

- 요금제 변경 및/또는 Premium으로 업그레이드. 이렇게 하려면 다음 명령을 실행합니다. `Sku.Tier` 속성은 `Standard` 또는 `Premium`, `Sku.Name` 속성은 `MeteredData` 또는 `UnlimitedData`일 수 있습니다.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > `Sku.Name` 속성이 `Sku.Tier` 및 `Sku.Family`와 일치하는지 확인합니다. 패밀리와 계층을 변경하고 이름을 변경하지 않으면 연결이 비활성화됩니다.
    > 
    > 

    중단 없이 SKU를 업그레이드할 수 있지만 무제한 요금제에서 종량제 요금제로 전환할 수는 없습니다. SKU를 다운그레이드하는 경우 대역폭 소비를 표준 SKU의 기본 제한 이내로 유지해야 합니다.

## <a name="availability-considerations"></a>가용성 고려 사항

ExpressRoute는 고가용성 구현을 위해 HSRP(Hot Standby Routing Protocol), VRRP(Virtual Router Redundancy Protocol) 등의 라우터 중복 프로토콜을 지원하지 않습니다. 대신, 피어링당 BGP 세션의 중복 쌍을 사용합니다. 네트워크에 대한 고가용성 연결을 지원하기 위해 Azure는 활성-활성 구성의 두 라우터(Microsoft Edge의 일부)에 두 개의 중복 포트를 프로비전합니다.

기본적으로 BGP 세션은 유휴 시간 제한 값으로 60초를 사용합니다. 세션이 3번(총 180초) 시간 초과되면 라우터가 사용 불가능으로 표시되고 모든 트래픽이 나머지 라우터로 리디렉션됩니다. 이 180초 시간 제한이 중요 응용 프로그램에는 너무 길 수 있습니다. 너무 긴 경우 온-프레미스 라우터의 BGP 시간 제한 설정을 더 작은 값으로 변경할 수 있습니다.

사용하는 공급자 유형과 구성할 ExpressRoute 회로 및 가상 네트워크 게이트웨이 연결 수에 따라 다양한 방법으로 Azure 연결에 대해 고가용성을 구성할 수 있습니다. 다음은 가용성 옵션을 요약한 것입니다.

* 레이어 2 연결을 사용하는 경우 온-프레미스 네트워크에 중복 라우터를 활성-활성 구성으로 배포합니다. 기본 회로를 하나의 라우터에 연결하고 보조 회로를 다른 라우터에 연결합니다. 이렇게 하면 연결의 양쪽 끝에서 고가용성 연결이 제공됩니다. ExpressRoute SLA(서비스 수준 계약)가 필요한 경우 이 옵션이 필요합니다. 자세한 내용은 [Azure ExpressRoute의 SLA][sla-for-expressroute]를 참조하세요.

    다음 다이어그램은 중복 온-프레미스 라우터가 기본 및 보조 회로에 연결된 구성을 보여 줍니다. 각 회로는 공용 피어링 및 개인 피어링의 트래픽을 처리합니다(이전 섹션에서 설명한 대로 각 피어링에 한 쌍의 /30 주소 공간이 지정됨).

    ![[1]][1]

* 레이어 3 연결을 사용하는 경우 연결이 가용성을 처리하는 중복 BGP 세션을 제공하는지 확인합니다.

* 다른 서비스 공급자가 제공하는 여러 ExpressRoute 회로에 VNet을 연결합니다. 이 전략은 추가 고가용성 및 재해 복구 기능을 제공합니다.

* 사이트 간 VPN을 ExpressRoute에 대한 장애 조치(failover) 경로로 구성합니다. 이 옵션에 대한 자세한 내용은 [VPN 장애 조치(failover)와 함께 ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결][highly-available-network-architecture]을 참조하세요.
 이 옵션은 개인 피어링에만 적용됩니다. Azure 및 Office 365 서비스의 경우 인터넷이 유일한 장애 조치(failover) 경로입니다. 

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

[AzureCT(Azure Connectivity Toolkit)][azurect]를 사용하여 온-프레미스 데이터 센터와 Azure 간의 연결을 모니터할 수 있습니다. 

## <a name="security-considerations"></a>보안 고려 사항

보안 문제와 준수 요구 사항에 따라 다양한 방법으로 Azure 연결에 대한 보안 옵션을 구성할 수 있습니다. 

ExpressRoute는 레이어 3에서 작동합니다. 응용 프로그램 레이어의 위협은 합법적인 리소스에 대한 트래픽을 제한하는 네트워크 보안 어플라이언스를 사용하여 방지할 수 있습니다. 또한 공용 피어링을 사용하는 ExpressRoute 연결은 온-프레미스에서만 시작할 수 있습니다. 이렇게 하면 악성 서비스가 인터넷에서 온-프레미스 데이터에 액세스하여 손상하는 것을 방지할 수 있습니다.

보안을 최대화하려면 온-프레미스 네트워크와 공급자 에지 라우터 사이에 네트워크 보안 어플라이언스를 추가합니다. 이렇게 하면 VNet에서 권한 없는 트래픽이 유입되는 것을 제한할 수 있습니다.

![[2]][2]

감사 또는 준수를 위해 VNet에서 실행 중인 구성 요소에서 인터넷에 직접 액세스하지 못하도록 차단하고 [강제 터널링][forced-tuneling]을 구현해야 할 수도 있습니다. 이 경우 온-프레미스에서 실행 중인 프록시를 통해 인터넷 트래픽을 감사할 수 있는 위치로 다시 리디렉션해야 합니다. 권한 없는 트래픽의 유출을 차단하고 잠재적인 악성 인바운드 트래픽을 필터링하도록 프록시를 구성할 수 있습니다.

![[3]][3]

보안을 최대화하려면 VM에 대해 공용 IP 주소를 사용하지 않도록 설정하고 NSG를 사용하여 이러한 VM에 공개적으로 액세스할 수 없도록 합니다. 내부 IP 주소를 통해서만 VM을 사용할 수 있어야 합니다. ExpressRoute 네트워크를 통해 이 주소에 액세스하여 온-프레미스 DevOps 직원이 구성 또는 유지 관리를 수행할 수 있도록 합니다.

VM의 관리 끝점을 외부 네트워크에 노출해야 하는 경우 NSG 또는 액세스 제어 목록을 사용하여 이러한 포트의 가시성을 IP 주소 또는 네트워크의 허용 목록으로 제한합니다.

> [!NOTE]
> 기본적으로 Azure Portal을 통해 배포된 Azure VM에는 로그인 액세스를 제공하는 공용 IP 주소가 포함되어 있습니다.  
> 
> 


## <a name="deploy-the-solution"></a>솔루션 배포

**필수 조건.** 적절한 네트워크 어플라이언스를 사용하여 기존 온-프레미스 인프라가 이미 구성된 상태여야 합니다.

솔루션을 배포하려면 다음 단계를 수행합니다.

1. 아래 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure Portal에서 링크가 열릴 때까지 기다린 후 다음 단계를 수행합니다.
   * **리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-hybrid-er-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 하위 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구매** 단추를 클릭합니다.
3. 배포가 완료될 때가지 기다립니다.
4. 아래 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Azure Portal에서 링크가 열릴 때까지 기다린 후 다음 단계를 수행합니다.
   * **리소스 그룹** 섹션에서 **기존 항목 사용**을 선택하고 텍스트 상자에 `ra-hybrid-er-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 하위 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구매** 단추를 클릭합니다.
6. 배포가 완료될 때가지 기다립니다.


<!-- links -->
[forced-tuneling]: ../dmz/secure-vnet-hybrid.md
[highly-available-network-architecture]: ./expressroute-vpn-failover.md

[expressroute-technical-overview]: /azure/expressroute/expressroute-introduction
[expressroute-prereqs]: /azure/expressroute/expressroute-prerequisites
[configure-expressroute-routing]: /azure/expressroute/expressroute-howto-routing-arm
[sla-for-expressroute]: https://azure.microsoft.com/support/legal/sla/expressroute/v1_0/
[link-vnet-to-expressroute]: /azure/expressroute/expressroute-howto-linkvnet-arm
[ExpressRoute-provisioning]: /azure/expressroute/expressroute-workflows
[expressroute-introduction]: /azure/expressroute/expressroute-introduction
[expressroute-peering]: /azure/expressroute/expressroute-circuit-peerings
[expressroute-pricing]: https://azure.microsoft.com/pricing/details/expressroute/
[expressroute-limits]: /azure/azure-subscription-service-limits#networking-limits
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "Azure ExpressRoute를 사용하는 하이브리드 네트워크 아키텍처"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "ExpressRoute 기본 및 보조 회로와 함께 중복 라우터 사용"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "온-프레미스 네트워크에 보안 장치 추가"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "강제 터널링을 사용하여 인터넷 바인딩된 트래픽 감사"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "ExpressRoute 회로의 ServiceKey 찾기"  