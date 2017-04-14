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

* **•	연결성 공급자** (미표시). 레이어2 또는 레이어3 연결을 사용하여 귀하의 데이터 센터와 Azure 데이터센터 간 연결을 제공하는 회사입니다.

You can download a [Visio file](https://aka.ms/arch-diagrams) of this architecture.

> [!참고]
> Azure는 [Resource Manager](/azure/azure-resource-manager/resource-group-overview)와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 문서에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다.
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

Ensure that your organization has met the [ExpressRoute prerequisite requirements][expressroute-prereqs] for connecting to Azure.

If you haven't already done so, add a subnet named `GatewaySubnet` to your Azure VNet and create an ExpressRoute virtual network gateway using the Azure VPN gateway service. For more information about this process, see [ExpressRoute workflows for circuit provisioning and circuit states][ExpressRoute-provisioning].

Create an ExpressRoute circuit as follows:

1. Run the following PowerShell command:
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. Send the `ServiceKey` for the new circuit to the service provider.

3. Wait for the provider to provision the circuit. To verify the provisioning state of a circuit, run the following PowerShell command:
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    The `Provisioning state` field in the `Service Provider` section of the output will change from `NotProvisioned` to `Provisioned` when the circuit is ready.

    > [!NOTE]
    > If you're using a layer 3 connection, the provider should configure and manage routing for you. You provide the information necessary to enable the provider to implement the appropriate routes.
    > 
    > 

4. If you're using a layer 2 connection:

    1. Reserve two /30 subnets composed of valid public IP addresses for each type of peering you want to implement. These /30 subnets will be used to provide IP addresses for the routers used for the circuit. If you are implementing private, public, and Microsoft peering, you'll need 6 /30 subnets with valid public IP addresses.     

    2. Configure routing for the ExpressRoute circuit. Run the following PowerShell commands for each type of peering you want to configure (private, public, and Microsoft). For more information, see [Create and modify routing for an ExpressRoute circuit][configure-expressroute-routing].
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. Reserve another pool of valid public IP addresses to use for network address translation (NAT) for public and Microsoft peering. It is recommended to have a different pool for each peering. Specify the pool to your connectivity provider, so they can configure border gateway protocol (BGP) advertisements for those ranges.

5. Run the following PowerShell commands to link your private VNet(s) to the ExpressRoute circuit. For more information,see [Link a virtual network to an ExpressRoute circuit][link-vnet-to-expressroute].

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

You can connect multiple VNets located in different regions to the same ExpressRoute circuit, as long as all VNets and the ExpressRoute circuit are located within the same geopolitical region.

### Troubleshooting 

If a previously functioning ExpressRoute circuit now fails to connect, in the absence of any configuration changes on-premises or within your private VNet, you may need to contact the connectivity provider and work with them to correct the issue. Use the following Powershell commands to verify that the ExpressRoute circuit has been provisioned:

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

The output of this command shows several properties for your circuit, including `ProvisioningState`, `CircuitProvisioningState`, and `ServiceProviderProvisioningState` as shown below.

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

If the `ProvisioningState` is not set to `Succeeded` after you tried to create a new circuit, remove the circuit by using the command below and try to create it again.

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

If your provider had already provisioned the circuit, and the `ProvisioningState` is set to `Failed`, or the `CircuitProvisioningState` is not `Enabled`, contact your provider for further assistance.

## Scalability considerations

ExpressRoute circuits provide a high bandwidth path between networks. Generally, the higher the bandwidth the greater the cost. 

ExpressRoute offers two [pricing plans][expressroute-pricing] to customers, a metered plan and an unlimited data plan. Charges vary according to circuit bandwidth. Available bandwidth will likely vary from provider to provider. Use the `Get-AzureRmExpressRouteServiceProvider` cmdlet to see the providers available in your region and the bandwidths that they offer.
 
A single ExpressRoute circuit can support a certain number of peerings and VNet links. See [ExpressRoute limits](/azure/azure-subscription-service-limits) for more information.

For an extra charge, the ExpressRoute Premium add-on provides some additional capability:

* Increased route limits for public and private peering. 
* Increased number of VNet links per ExpressRoute circuit. 
* Global connectivity for services.

See [ExpressRoute pricing][expressroute-pricing] for details. 

ExpressRoute circuits are designed to allow temporary network bursts up to two times the bandwidth limit that you procured for no additional cost. This is achieved by using redundant links. However, not all connectivity providers support this feature. Verify that your connectivity provider enables this feature before depending on it.

Although some providers allow you to change your bandwidth, make sure you pick an initial bandwidth that surpasses your needs and provides room for growth. If you need to increase bandwidth in the future, you are left with two options:

- Increase the bandwidth. You should avoid this option as much as possible, and not all providers allow you to increase bandwidth dynamically. But if a bandwidth increase is needed, check with your provider to verify they support changing ExpressRoute bandwidth properties via Powershell commands. If they do, run the commands below.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    You can increase the bandwidth without loss of connectivity. Downgrading the bandwidth will result in disruption in connectivity, because you must delete the circuit and recreate it with the new configuration.

- Change your pricing plan and/or upgrade to Premium. To do so, run the following commands. The `Sku.Tier` property can be `Standard` or `Premium`; the `Sku.Name` property can be `MeteredData` or `UnlimitedData`.

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > Make sure the `Sku.Name` property matches the `Sku.Tier` and `Sku.Family`. If you change the family and tier, but not the name, your connection will be disabled.
    > 
    > 

    You can upgrade the SKU without disruption, but you cannot switch from the unlimited pricing plan to metered. When downgrading the SKU, your bandwidth consumption must remain within the default limit of the standard SKU.

## Availability considerations

ExpressRoute does not support router redundancy protocols such as hot standby routing protocol (HSRP) and virtual router redundancy protocol (VRRP) to implement high availability. Instead, it uses a redundant pair of BGP sessions per peering. To facilitate highly-available connections to your network, Azure provisions you with two redundant ports on two routers (part of the Microsoft edge) in an active-active configuration.

By default, BGP sessions use an idle timeout value of 60 seconds. If a session times out three times (180 seconds total), the router is marked as unavailable, and all traffic is redirected to the remaining router. This 180-second timeout might be too long for critical applications. If so, you can change your BGP time-out settings on the on-premises router to a smaller value.

You can configure high availability for your Azure connection in different ways, depending on the type of provider you use, and the number of ExpressRoute circuits and virtual network gateway connections you're willing to configure. The following summarizes your availability options:

* If you're using a layer 2 connection, deploy redundant routers in your on-premises network in an active-active configuration. Connect the primary circuit to one router, and the secondary circuit to the other. This will give you a highly available connection at both ends of the connection. This is necessary if you require the ExpressRoute service level agreement (SLA). See [SLA for Azure ExpressRoute][sla-for-expressroute] for details.

    The following diagram shows a configuration with redundant on-premises routers connected to the primary and secondary circuits. Each circuit handles the traffic for a public peering and a private peering (each peering is designated a pair of /30 address spaces, as described in the previous section).

    ![[1]][1]

* If you're using a layer 3 connection, verify that it provides redundant BGP sessions that handle availability for you.

* Connect the VNet to multiple ExpressRoute circuits, supplied by different service providers. This strategy provides additional high-availability and disaster recovery capabilities.

* Configure a site-to-site VPN as a failover path for ExpressRoute. For more about this option, see [Connect an on-premises network to Azure using ExpressRoute with VPN failover][highly-available-network-architecture].
 This option only applies to private peering. For Azure and Office 365 services, the Internet is the only failover path. 

## Manageability considerations

You can use the [Azure Connectivity Toolkit (AzureCT)][azurect] to monitor connectivity between your on-premises datacenter and Azure. 

## Security considerations

You can configure security options for your Azure connection in different ways, depending on your security concerns and compliance needs. 

ExpressRoute operates in layer 3. Threats in the application layer can be prevented by using a network security appliance that restricts traffic to legitimate resources. Additionally, ExpressRoute connections using public peering can only be initiated from on-premises. This prevents a rogue service from accessing and compromising on-premises data from the Internet.

To maximize security, add network security appliances between the on-premises network and the provider edge routers. This will help to restrict the inflow of unauthorized traffic from the VNet:

![[2]][2]

For auditing or compliance purposes, it may be necessary to prohibit direct access from components running in the VNet to the Internet and implement [forced tunneling][forced-tuneling]. In this situation, Internet traffic should be redirected back through a proxy running  on-premises where it can be audited. The proxy can be configured to block unauthorized traffic flowing out, and filter potentially malicious inbound traffic.

![[3]][3]

To maximize security, do not enable a public IP address for your VMs, and use NSGs to ensure that these VMs aren't publicly accessible. VMs should only be available using the internal IP address. These addresses can be made accessible through the ExpressRoute network, enabling on-premises DevOps staff to perform configuration or maintenance.

If you must expose management endpoints for VMs to an external network, use NSGs or access control lists to restrict the visibility of these ports to a whitelist of IP addresses or networks.

> [!NOTE]
> By default, Azure VMs deployed through the Azure portal include a public IP address that provides login access.  
> 
> 


## Deploy the solution

**Prequisites.** You must have an existing on-premises infrastructure already configured with a suitable network appliance.

To deploy the solution, perform the following steps.

1. Click the button below:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Wait for the link to open in the Azure portal, then follow these steps:
   * The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-er-rg` in the text box.
   * Select the region from the **Location** drop down box.
   * Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.
   * Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.
   * Click the **Purchase** button.
3. Wait for the deployment to complete.
4. Click the button below:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Wait for the link to open in the Azure portal, then follow these steps:
   * Select **Use existing** in the **Resource group** section and enter `ra-hybrid-er-rg` in the text box.
   * Select the region from the **Location** drop down box.
   * Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.
   * Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.
   * Click the **Purchase** button.
6. Wait for the deployment to complete.


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
