---
title: "ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결"
description: "Azure ExpressRoute를 사용하여 연결된 온-프레미스 네트워크 및 Azure Virtual Network를 포괄하는 보안 사이트 간 네트워크 아키텍처를 구현하는 방법입니다."
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute-vpn-failover
pnp.series.prev: vpn
cardTitle: ExpressRoute
ms.openlocfilehash: 671be5118faaefab5ba5348de81642d8a8124b59
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute"></a><span data-ttu-id="ecc31-103">ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결</span><span class="sxs-lookup"><span data-stu-id="ecc31-103">Connect an on-premises network to Azure using ExpressRoute</span></span>

<span data-ttu-id="ecc31-104">이 참조 아키텍처는 [Azure ExpressRoute][expressroute-introduction]를 사용하여 온-프레미스 네트워크를 Azure의 가상 네트워크에 연결하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-104">This reference architecture shows how to connect an on-premises network to virtual networks on Azure, using [Azure ExpressRoute][expressroute-introduction].</span></span> <span data-ttu-id="ecc31-105">ExpressRoute 연결은 타사 연결 공급자를 통해 개인 전용 연결을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-105">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="ecc31-106">개인 연결은 온-프레미스 네트워크를 Azure로 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-106">The private connection extends your on-premises network into Azure.</span></span> [<span data-ttu-id="ecc31-107">**이 솔루션을 배포합니다**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="ecc31-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="ecc31-108">![[0]][0]</span></span>

<span data-ttu-id="ecc31-109">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="ecc31-109">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="ecc31-110">건축</span><span class="sxs-lookup"><span data-stu-id="ecc31-110">Architecture</span></span>

<span data-ttu-id="ecc31-111">이 아키텍처는 다음 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-111">The architecture consists of the following components.</span></span>

* <span data-ttu-id="ecc31-112">**온-프레미스 회사 네트워크**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-112">**On-premises corporate network**.</span></span> <span data-ttu-id="ecc31-113">조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-113">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="ecc31-114">**ExpressRoute 회로**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-114">**ExpressRoute circuit**.</span></span> <span data-ttu-id="ecc31-115">에지 라우터를 통해 Azure에서 온-프레미스 네트워크를 연결하는 연결 공급자가 제공하는 레이어 2 또는 레이어 3 회로입니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-115">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="ecc31-116">이 회로는 연결 공급자가 관리하는 하드웨어 인프라를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-116">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="ecc31-117">**로컬 에지 라우터**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-117">**Local edge routers**.</span></span> <span data-ttu-id="ecc31-118">온-프레미스 네트워크를 공급자가 관리하는 회로에 연결하는 라우터입니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-118">Routers that connect the on-premises network to the circuit managed by the provider.</span></span> <span data-ttu-id="ecc31-119">연결이 프로비전된 방법에 따라 라우터에서 사용하는 공용 IP 주소를 제공해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-119">Depending on how your connection is provisioned, you may need to provide the public IP addresses used by the routers.</span></span>
* <span data-ttu-id="ecc31-120">**Microsoft Edge 라우터**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-120">**Microsoft edge routers**.</span></span> <span data-ttu-id="ecc31-121">활성-활성 고가용성 구성의 두 라우터입니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-121">Two routers in an active-active highly available configuration.</span></span> <span data-ttu-id="ecc31-122">이 라우터를 사용하면 연결 공급자가 해당 회로를 데이터 센터에 직접 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-122">These routers enable a connectivity provider to connect their circuits directly to their datacenter.</span></span> <span data-ttu-id="ecc31-123">연결이 프로비전된 방법에 따라 라우터에서 사용하는 공용 IP 주소를 제공해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-123">Depending on how your connection is provisioned, you may need to provide the public IP addresses used by the routers.</span></span>

* <span data-ttu-id="ecc31-124">**Azure Virtual Network(VNet)**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-124">**Azure virtual networks (VNets)**.</span></span> <span data-ttu-id="ecc31-125">각 VNet은 단일 Azure 지역에 상주하며 여러 응용 프로그램 계층을 호스트할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-125">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="ecc31-126">각 VNet의 서브넷을 사용하여 응용 프로그램 계층을 분할할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-126">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="ecc31-127">**Azure 공용 서비스**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-127">**Azure public services**.</span></span> <span data-ttu-id="ecc31-128">하이브리드 응용 프로그램 내에서 사용할 수 있는 Azure 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-128">Azure services that can be used within a hybrid application.</span></span> <span data-ttu-id="ecc31-129">이 서비스는 인터넷에서도 사용할 수 있지만, ExpressRoute 회로를 통해 액세스하는 경우 트래픽이 인터넷을 통과하지 않으므로 대기 시간이 줄고 성능이 더 예측 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-129">These services are also available over the Internet, but accessing them using an ExpressRoute circuit provides low latency and more predictable performance, because traffic does not go through the Internet.</span></span> <span data-ttu-id="ecc31-130">[공용 피어링][expressroute-peering]을 사용하여 연결이 수행되고, 조직이 주소를 소유하거나 연결 공급자가 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-130">Connections are performed using [public peering][expressroute-peering], with addresses that are either owned by your organization or supplied by your connectivity provider.</span></span>

* <span data-ttu-id="ecc31-131">**Office 365 서비스**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-131">**Office 365 services**.</span></span> <span data-ttu-id="ecc31-132">Microsoft에서 제공하는, 공개적으로 사용할 수 있는 Office 365 응용 프로그램 및 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-132">The publicly available Office 365 applications and services provided by Microsoft.</span></span> <span data-ttu-id="ecc31-133">[Microsoft 피어링][expressroute-peering]을 사용하여 연결이 수행되고, 조직이 주소를 소유하거나 연결 공급자가 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-133">Connections are performed using [Microsoft peering][expressroute-peering], with addresses that are either owned by your organization or supplied by your connectivity provider.</span></span> <span data-ttu-id="ecc31-134">Microsoft 피어링을 통해 Microsoft CRM Online에 직접 연결할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-134">You can also connect directly to Microsoft CRM Online through Microsoft peering.</span></span>

* <span data-ttu-id="ecc31-135">**연결 공급자**(표시되지 않음).</span><span class="sxs-lookup"><span data-stu-id="ecc31-135">**Connectivity providers** (not shown).</span></span> <span data-ttu-id="ecc31-136">레이어 2 또는 레이어 3 연결을 사용하여 데이터 센터와 Azure 데이터 센터 간의 연결을 제공하는 회사입니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-136">Companies that provide a connection either using layer 2 or layer 3 connectivity between your datacenter and an Azure datacenter.</span></span>

## <a name="recommendations"></a><span data-ttu-id="ecc31-137">권장 사항</span><span class="sxs-lookup"><span data-stu-id="ecc31-137">Recommendations</span></span>

<span data-ttu-id="ecc31-138">대부분의 시나리오의 경우 다음 권장 사항을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-138">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="ecc31-139">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-139">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="connectivity-providers"></a><span data-ttu-id="ecc31-140">연결 공급자</span><span class="sxs-lookup"><span data-stu-id="ecc31-140">Connectivity providers</span></span>

<span data-ttu-id="ecc31-141">해당 위치에 적합한 ExpressRoute 연결 공급자를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-141">Select a suitable ExpressRoute connectivity provider for your location.</span></span> <span data-ttu-id="ecc31-142">해당 위치에서 사용할 수 있는 연결 공급자 목록을 가져오려면 다음 Azure PowerShell 명령을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-142">To get a list of connectivity providers available at your location, use the following Azure PowerShell command:</span></span>

```powershell
Get-AzureRmExpressRouteServiceProvider
```

<span data-ttu-id="ecc31-143">ExpressRoute 연결 공급자는 다음과 같은 방법으로 데이터 센터를 Microsoft에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-143">ExpressRoute connectivity providers connect your datacenter to Microsoft in the following ways:</span></span>

* <span data-ttu-id="ecc31-144">**클라우드 Exchange에 함께 배치**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-144">**Co-located at a cloud exchange**.</span></span> <span data-ttu-id="ecc31-145">클라우드 Exchange와 같은 시설에 있는 경우 공동 배치 공급자의 이더넷 Exchange를 통해 Azure에 대한 가상 교차 연결을 주문할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-145">If you're co-located in a facility with a cloud exchange, you can order virtual cross-connections to Azure through the co-location provider’s Ethernet exchange.</span></span> <span data-ttu-id="ecc31-146">공동 배치 공급자는 공동 배치 시설의 인프라와 Azure 간에 레이어 2 교차 연결 또는 관리되는 레이어 3 교차 연결을 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-146">Co-location providers can offer either layer 2 cross-connections, or managed layer 3 cross-connections between your infrastructure in the co-location facility and Azure.</span></span>
* <span data-ttu-id="ecc31-147">**지점 간 이더넷 연결**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-147">**Point-to-point Ethernet connections**.</span></span> <span data-ttu-id="ecc31-148">지점 간 이더넷 연결을 통해 온-프레미스 데이터 센터/사무소를 Azure에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-148">You can connect your on-premises datacenters/offices to Azure through point-to-point Ethernet links.</span></span> <span data-ttu-id="ecc31-149">지점 간 이더넷 공급자는 사이트와 Azure 간에 레이어 2 연결 또는 관리되는 레이어 3 연결을 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-149">Point-to-point Ethernet providers can offer layer 2 connections, or managed layer 3 connections between your site and Azure.</span></span>
* <span data-ttu-id="ecc31-150">**임의(IPVPN)의 네트워크**.</span><span class="sxs-lookup"><span data-stu-id="ecc31-150">**Any-to-any (IPVPN) networks**.</span></span> <span data-ttu-id="ecc31-151">WAN(광역 네트워크)과 Azure를 통합할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-151">You can integrate your wide area network (WAN) with Azure.</span></span> <span data-ttu-id="ecc31-152">IPVPN(인터넷 프로토콜 가상 사설망) 공급자(일반적으로 MPLS(Multi-Protocol Label Switching) VPN)는 지점과 데이터 센터 간의 임의 연결을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-152">Internet protocol virtual private network (IPVPN) providers (typically a multiprotocol label switching VPN) offer any-to-any connectivity between your branch offices and datacenters.</span></span> <span data-ttu-id="ecc31-153">Azure를 WAN에 상호 연결하여 다른 지점처럼 보이도록 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-153">Azure can be interconnected to your WAN to make it look just like any other branch office.</span></span> <span data-ttu-id="ecc31-154">WAN 공급자는 일반적으로 관리되는 레이어 3 연결을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-154">WAN providers typically offer managed layer 3 connectivity.</span></span>

<span data-ttu-id="ecc31-155">연결 공급자에 대한 자세한 내용은 [ExpressRoute 소개][expressroute-introduction]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-155">For more information about connectivity providers, see the [ExpressRoute introduction][expressroute-introduction].</span></span>

### <a name="expressroute-circuit"></a><span data-ttu-id="ecc31-156">ExpressRoute 회로</span><span class="sxs-lookup"><span data-stu-id="ecc31-156">ExpressRoute circuit</span></span>

<span data-ttu-id="ecc31-157">조직이 Azure에 연결하기 위한 [ExpressRoute 필수 조건 요구 사항][expressroute-prereqs]을 충족하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-157">Ensure that your organization has met the [ExpressRoute prerequisite requirements][expressroute-prereqs] for connecting to Azure.</span></span>

<span data-ttu-id="ecc31-158">아직 수행하지 않은 경우 `GatewaySubnet` 서브넷을 Azure VNet에 추가하고 Azure VPN Gateway 서비스를 사용하여 ExpressRoute 가상 네트워크 게이트웨이를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-158">If you haven't already done so, add a subnet named `GatewaySubnet` to your Azure VNet and create an ExpressRoute virtual network gateway using the Azure VPN gateway service.</span></span> <span data-ttu-id="ecc31-159">이 프로세스에 대한 자세한 내용은 [회로 프로비전 및 회로 상태에 대한 ExpressRoute 워크플로][ExpressRoute-provisioning]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-159">For more information about this process, see [ExpressRoute workflows for circuit provisioning and circuit states][ExpressRoute-provisioning].</span></span>

<span data-ttu-id="ecc31-160">다음과 같이 ExpressRoute 회로를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-160">Create an ExpressRoute circuit as follows:</span></span>

1. <span data-ttu-id="ecc31-161">다음 PowerShell 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-161">Run the following PowerShell command:</span></span>
   
    ```powershell
    New-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>> -Location <<location>> -SkuTier <<sku-tier>> -SkuFamily <<sku-family>> -ServiceProviderName <<service-provider-name>> -PeeringLocation <<peering-location>> -BandwidthInMbps <<bandwidth-in-mbps>>
    ```
2. <span data-ttu-id="ecc31-162">새 회로의 `ServiceKey`을 서비스 공급자에게 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-162">Send the `ServiceKey` for the new circuit to the service provider.</span></span>

3. <span data-ttu-id="ecc31-163">공급자가 회로를 프로비전할 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-163">Wait for the provider to provision the circuit.</span></span> <span data-ttu-id="ecc31-164">회로의 프로비저닝 상태를 확인하려면 다음 PowerShell 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-164">To verify the provisioning state of a circuit, run the following PowerShell command:</span></span>
   
    ```powershell
    Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="ecc31-165">회로 준비가 완료되면 출력의 `Service Provider` 섹션에 있는 `Provisioning state` 필드가 `NotProvisioned`에서 `Provisioned`로 변경됩니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-165">The `Provisioning state` field in the `Service Provider` section of the output will change from `NotProvisioned` to `Provisioned` when the circuit is ready.</span></span>

    > [!NOTE]
    > <span data-ttu-id="ecc31-166">레이어 3 연결을 사용하는 경우 공급자가 라우팅을 구성하고 관리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-166">If you're using a layer 3 connection, the provider should configure and manage routing for you.</span></span> <span data-ttu-id="ecc31-167">공급자가 적절한 경로를 구현하는 데 필요한 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-167">You provide the information necessary to enable the provider to implement the appropriate routes.</span></span>
    > 
    > 

4. <span data-ttu-id="ecc31-168">레이어 2 연결을 사용하는 경우:</span><span class="sxs-lookup"><span data-stu-id="ecc31-168">If you're using a layer 2 connection:</span></span>

    1. <span data-ttu-id="ecc31-169">구현할 각 피어링 유형에 적합한 공용 IP 주소로 구성된 /30 서브넷 두 개를 예약합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-169">Reserve two /30 subnets composed of valid public IP addresses for each type of peering you want to implement.</span></span> <span data-ttu-id="ecc31-170">이러한 /30 서브넷은 회로에 사용되는 라우터의 IP 주소를 제공하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-170">These /30 subnets will be used to provide IP addresses for the routers used for the circuit.</span></span> <span data-ttu-id="ecc31-171">개인, 공용 및 Microsoft 피어링을 구현하는 경우 유효한 공용 IP 주소가 포함된 /30 서브넷 6개가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-171">If you are implementing private, public, and Microsoft peering, you'll need 6 /30 subnets with valid public IP addresses.</span></span>     

    2. <span data-ttu-id="ecc31-172">ExpressRoute 회로의 라우팅 구성</span><span class="sxs-lookup"><span data-stu-id="ecc31-172">Configure routing for the ExpressRoute circuit.</span></span> <span data-ttu-id="ecc31-173">구성할 각 피어링 유형(개인, 공용 및 Microsoft)에 대해 다음 PowerShell 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-173">Run the following PowerShell commands for each type of peering you want to configure (private, public, and Microsoft).</span></span> <span data-ttu-id="ecc31-174">자세한 내용은 [ExpressRoute 회로의 라우팅 만들기 및 수정][configure-expressroute-routing]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-174">For more information, see [Create and modify routing for an ExpressRoute circuit][configure-expressroute-routing].</span></span>
   
        ```powershell
        Set-AzureRmExpressRouteCircuitPeeringConfig -Name <<peering-name>> -Circuit <<circuit-name>> -PeeringType <<peering-type>> -PeerASN <<peer-asn>> -PrimaryPeerAddressPrefix <<primary-peer-address-prefix>> -SecondaryPeerAddressPrefix <<secondary-peer-address-prefix>> -VlanId <<vlan-id>>

        Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit <<circuit-name>>
        ```

    3. <span data-ttu-id="ecc31-175">공용 및 Microsoft 피어링의 NAT(Network Address Translation)에 사용할 다른 유효한 공용 IP 주소 풀을 예약합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-175">Reserve another pool of valid public IP addresses to use for network address translation (NAT) for public and Microsoft peering.</span></span> <span data-ttu-id="ecc31-176">각 피어링에 다른 풀을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-176">It is recommended to have a different pool for each peering.</span></span> <span data-ttu-id="ecc31-177">연결 공급자에 풀을 지정하여 해당 범위에 대한 BGP(Border Gateway Protocol) 알림을 구성할 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-177">Specify the pool to your connectivity provider, so they can configure border gateway protocol (BGP) advertisements for those ranges.</span></span>

5. <span data-ttu-id="ecc31-178">다음 PowerShell 명령을 실행하여 개인 VNet을 ExpressRoute 회로에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-178">Run the following PowerShell commands to link your private VNet(s) to the ExpressRoute circuit.</span></span> <span data-ttu-id="ecc31-179">자세한 내용은 [가상 네트워크를 ExpressRoute 회로에 연결][link-vnet-to-expressroute]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-179">For more information,see [Link a virtual network to an ExpressRoute circuit][link-vnet-to-expressroute].</span></span>

    ```powershell
    $circuit = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $gw = Get-AzureRmVirtualNetworkGateway -Name <<gateway-name>> -ResourceGroupName <<resource-group>>
    New-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> -ResourceGroupName <<resource-group>> -Location <<location> -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute
    ```

<span data-ttu-id="ecc31-180">모든 VNet 및 ExpressRoute 회로가 동일한 지정학적 지역에 있는 한, 서로 다른 지역에 있는 여러 VNet을 동일한 ExpressRoute 회로에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-180">You can connect multiple VNets located in different regions to the same ExpressRoute circuit, as long as all VNets and the ExpressRoute circuit are located within the same geopolitical region.</span></span>

### <a name="troubleshooting"></a><span data-ttu-id="ecc31-181">문제 해결</span><span class="sxs-lookup"><span data-stu-id="ecc31-181">Troubleshooting</span></span> 

<span data-ttu-id="ecc31-182">온-프레미스의 구성 변경 없이 또는 개인 VNet 내에서 이전에 작동하던 ExpressRoute 회로가 현재 연결되지 않는 경우 연결 공급자에 문의하여 함께 문제를 해결해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-182">If a previously functioning ExpressRoute circuit now fails to connect, in the absence of any configuration changes on-premises or within your private VNet, you may need to contact the connectivity provider and work with them to correct the issue.</span></span> <span data-ttu-id="ecc31-183">다음 PowerShell 명령을 사용하여 ExpressRoute 회로가 프로비전되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-183">Use the following Powershell commands to verify that the ExpressRoute circuit has been provisioned:</span></span>

```powershell
Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

<span data-ttu-id="ecc31-184">이 명령의 출력에는 아래 그림과 같이 `ProvisioningState`, `CircuitProvisioningState` 및 `ServiceProviderProvisioningState`를 포함하여 회로에 대한 여러 속성이 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-184">The output of this command shows several properties for your circuit, including `ProvisioningState`, `CircuitProvisioningState`, and `ServiceProviderProvisioningState` as shown below.</span></span>

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

<span data-ttu-id="ecc31-185">새 회로를 만들려고 시도한 후 `ProvisioningState`가 `Succeeded`로 설정되지 않은 경우 아래 명령을 사용하여 회로를 제거하고 다시 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-185">If the `ProvisioningState` is not set to `Succeeded` after you tried to create a new circuit, remove the circuit by using the command below and try to create it again.</span></span>

```powershell
Remove-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
```

<span data-ttu-id="ecc31-186">공급자가 이미 회로를 프로비전했으며 `ProvisioningState`가 `Failed`로 설정되었거나 `CircuitProvisioningState`가 `Enabled`가 아닌 경우 공급자에게 추가 지원을 요청하세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-186">If your provider had already provisioned the circuit, and the `ProvisioningState` is set to `Failed`, or the `CircuitProvisioningState` is not `Enabled`, contact your provider for further assistance.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="ecc31-187">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ecc31-187">Scalability considerations</span></span>

<span data-ttu-id="ecc31-188">ExpressRoute 회로는 네트워크 간에 높은 대역폭 경로를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-188">ExpressRoute circuits provide a high bandwidth path between networks.</span></span> <span data-ttu-id="ecc31-189">일반적으로 대역폭이 높을수록 비용이 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-189">Generally, the higher the bandwidth the greater the cost.</span></span> 

<span data-ttu-id="ecc31-190">ExpressRoute는 종량제 요금제와 무제한 데이터 요금제라는 두 가지 [요금제][expressroute-pricing]를 고객에게 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-190">ExpressRoute offers two [pricing plans][expressroute-pricing] to customers, a metered plan and an unlimited data plan.</span></span> <span data-ttu-id="ecc31-191">요금은 회로 대역폭에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-191">Charges vary according to circuit bandwidth.</span></span> <span data-ttu-id="ecc31-192">사용 가능한 대역폭은 공급자에 따라 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-192">Available bandwidth will likely vary from provider to provider.</span></span> <span data-ttu-id="ecc31-193">`Get-AzureRmExpressRouteServiceProvider` cmdlet을 사용하여 해당 지역에서 사용할 수 있는 공급자 및 공급자가 제공하는 대역폭을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-193">Use the `Get-AzureRmExpressRouteServiceProvider` cmdlet to see the providers available in your region and the bandwidths that they offer.</span></span>
 
<span data-ttu-id="ecc31-194">단일 ExpressRoute 회로에서 특정 개수의 피어링 및 VNet 연결을 지원할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-194">A single ExpressRoute circuit can support a certain number of peerings and VNet links.</span></span> <span data-ttu-id="ecc31-195">자세한 내용은 [ExpressRoute 제한](/azure/azure-subscription-service-limits)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-195">See [ExpressRoute limits](/azure/azure-subscription-service-limits) for more information.</span></span>

<span data-ttu-id="ecc31-196">추가 요금을 지불할 경우 ExpressRoute Premium 추가 기능이 다음과 같은 몇 가지 추가 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-196">For an extra charge, the ExpressRoute Premium add-on provides some additional capability:</span></span>

* <span data-ttu-id="ecc31-197">공용 및 개인 피어링에 대한 경로 제한 증가</span><span class="sxs-lookup"><span data-stu-id="ecc31-197">Increased route limits for public and private peering.</span></span> 
* <span data-ttu-id="ecc31-198">ExpressRoute 회로당 VNet 연결 수 증가</span><span class="sxs-lookup"><span data-stu-id="ecc31-198">Increased number of VNet links per ExpressRoute circuit.</span></span> 
* <span data-ttu-id="ecc31-199">서비스에 대한 전역 연결입니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-199">Global connectivity for services.</span></span>

<span data-ttu-id="ecc31-200">자세한 내용은 [ExpressRoute 가격][expressroute-pricing]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-200">See [ExpressRoute pricing][expressroute-pricing] for details.</span></span> 

<span data-ttu-id="ecc31-201">ExpressRoute 회로는 추가 비용 없이 확보한 대역폭 제한의 최대 2배까지 일시적 네트워크 버스트를 허용하도록 디자인되었습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-201">ExpressRoute circuits are designed to allow temporary network bursts up to two times the bandwidth limit that you procured for no additional cost.</span></span> <span data-ttu-id="ecc31-202">이 기능은 중복 연결을 사용하여 지원됩니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-202">This is achieved by using redundant links.</span></span> <span data-ttu-id="ecc31-203">그러나 모든 연결 공급자가 이 기능을 지원하는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-203">However, not all connectivity providers support this feature.</span></span> <span data-ttu-id="ecc31-204">이 기능을 사용하기 전에 연결 공급자가 기능을 지원하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-204">Verify that your connectivity provider enables this feature before depending on it.</span></span>

<span data-ttu-id="ecc31-205">일부 공급자는 대역폭 변경을 허용하지만, 요구를 충족하고 확장을 위한 여유 공간을 제공하는 초기 대역폭을 선택해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-205">Although some providers allow you to change your bandwidth, make sure you pick an initial bandwidth that surpasses your needs and provides room for growth.</span></span> <span data-ttu-id="ecc31-206">나중에 대역폭을 늘려야 하는 경우 다음 두 가지 옵션 중에서 선택해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-206">If you need to increase bandwidth in the future, you are left with two options:</span></span>

- <span data-ttu-id="ecc31-207">대역폭 증가.</span><span class="sxs-lookup"><span data-stu-id="ecc31-207">Increase the bandwidth.</span></span> <span data-ttu-id="ecc31-208">이 옵션은 가능한 한 피해야 하며, 모든 공급자가 동적 대역폭 증가를 허용하는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-208">You should avoid this option as much as possible, and not all providers allow you to increase bandwidth dynamically.</span></span> <span data-ttu-id="ecc31-209">그러나 대역폭 증가가 필요한 경우 공급자에게 문의하여 Powershell 명령을 통한 ExpressRoute 대역폭 속성 변경을 지원하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-209">But if a bandwidth increase is needed, check with your provider to verify they support changing ExpressRoute bandwidth properties via Powershell commands.</span></span> <span data-ttu-id="ecc31-210">지원하는 경우 아래 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-210">If they do, run the commands below.</span></span>

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>
    $ckt.ServiceProviderProperties.BandwidthInMbps = <<bandwidth-in-mbps>>
    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    <span data-ttu-id="ecc31-211">연결 손실 없이 대역폭을 늘릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-211">You can increase the bandwidth without loss of connectivity.</span></span> <span data-ttu-id="ecc31-212">대역폭을 다운그레이드하는 경우 회로를 삭제하고 새 구성으로 다시 만들어야 하기 때문에 연결이 중단됩니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-212">Downgrading the bandwidth will result in disruption in connectivity, because you must delete the circuit and recreate it with the new configuration.</span></span>

- <span data-ttu-id="ecc31-213">요금제 변경 및/또는 Premium으로 업그레이드.</span><span class="sxs-lookup"><span data-stu-id="ecc31-213">Change your pricing plan and/or upgrade to Premium.</span></span> <span data-ttu-id="ecc31-214">이렇게 하려면 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-214">To do so, run the following commands.</span></span> <span data-ttu-id="ecc31-215">`Sku.Tier` 속성은 `Standard` 또는 `Premium`, `Sku.Name` 속성은 `MeteredData` 또는 `UnlimitedData`일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-215">The `Sku.Tier` property can be `Standard` or `Premium`; the `Sku.Name` property can be `MeteredData` or `UnlimitedData`.</span></span>

    ```powershell
    $ckt = Get-AzureRmExpressRouteCircuit -Name <<circuit-name>> -ResourceGroupName <<resource-group>>

    $ckt.Sku.Tier = "Premium"
    $ckt.Sku.Family = "MeteredData"
    $ckt.Sku.Name = "Premium_MeteredData"

    Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

    > [!IMPORTANT]
    > <span data-ttu-id="ecc31-216">`Sku.Name` 속성이 `Sku.Tier` 및 `Sku.Family`와 일치하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-216">Make sure the `Sku.Name` property matches the `Sku.Tier` and `Sku.Family`.</span></span> <span data-ttu-id="ecc31-217">패밀리와 계층을 변경하고 이름을 변경하지 않으면 연결이 비활성화됩니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-217">If you change the family and tier, but not the name, your connection will be disabled.</span></span>
    > 
    > 

    <span data-ttu-id="ecc31-218">중단 없이 SKU를 업그레이드할 수 있지만 무제한 요금제에서 종량제 요금제로 전환할 수는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-218">You can upgrade the SKU without disruption, but you cannot switch from the unlimited pricing plan to metered.</span></span> <span data-ttu-id="ecc31-219">SKU를 다운그레이드하는 경우 대역폭 소비를 표준 SKU의 기본 제한 이내로 유지해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-219">When downgrading the SKU, your bandwidth consumption must remain within the default limit of the standard SKU.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="ecc31-220">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ecc31-220">Availability considerations</span></span>

<span data-ttu-id="ecc31-221">ExpressRoute는 고가용성 구현을 위해 HSRP(Hot Standby Routing Protocol), VRRP(Virtual Router Redundancy Protocol) 등의 라우터 중복 프로토콜을 지원하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-221">ExpressRoute does not support router redundancy protocols such as hot standby routing protocol (HSRP) and virtual router redundancy protocol (VRRP) to implement high availability.</span></span> <span data-ttu-id="ecc31-222">대신, 피어링당 BGP 세션의 중복 쌍을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-222">Instead, it uses a redundant pair of BGP sessions per peering.</span></span> <span data-ttu-id="ecc31-223">네트워크에 대한 고가용성 연결을 지원하기 위해 Azure는 활성-활성 구성의 두 라우터(Microsoft Edge의 일부)에 두 개의 중복 포트를 프로비전합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-223">To facilitate highly-available connections to your network, Azure provisions you with two redundant ports on two routers (part of the Microsoft edge) in an active-active configuration.</span></span>

<span data-ttu-id="ecc31-224">기본적으로 BGP 세션은 유휴 시간 제한 값으로 60초를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-224">By default, BGP sessions use an idle timeout value of 60 seconds.</span></span> <span data-ttu-id="ecc31-225">세션이 3번(총 180초) 시간 초과되면 라우터가 사용 불가능으로 표시되고 모든 트래픽이 나머지 라우터로 리디렉션됩니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-225">If a session times out three times (180 seconds total), the router is marked as unavailable, and all traffic is redirected to the remaining router.</span></span> <span data-ttu-id="ecc31-226">이 180초 시간 제한이 중요 응용 프로그램에는 너무 길 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-226">This 180-second timeout might be too long for critical applications.</span></span> <span data-ttu-id="ecc31-227">너무 긴 경우 온-프레미스 라우터의 BGP 시간 제한 설정을 더 작은 값으로 변경할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-227">If so, you can change your BGP time-out settings on the on-premises router to a smaller value.</span></span>

<span data-ttu-id="ecc31-228">사용하는 공급자 유형과 구성할 ExpressRoute 회로 및 가상 네트워크 게이트웨이 연결 수에 따라 다양한 방법으로 Azure 연결에 대해 고가용성을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-228">You can configure high availability for your Azure connection in different ways, depending on the type of provider you use, and the number of ExpressRoute circuits and virtual network gateway connections you're willing to configure.</span></span> <span data-ttu-id="ecc31-229">다음은 가용성 옵션을 요약한 것입니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-229">The following summarizes your availability options:</span></span>

* <span data-ttu-id="ecc31-230">레이어 2 연결을 사용하는 경우 온-프레미스 네트워크에 중복 라우터를 활성-활성 구성으로 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-230">If you're using a layer 2 connection, deploy redundant routers in your on-premises network in an active-active configuration.</span></span> <span data-ttu-id="ecc31-231">기본 회로를 하나의 라우터에 연결하고 보조 회로를 다른 라우터에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-231">Connect the primary circuit to one router, and the secondary circuit to the other.</span></span> <span data-ttu-id="ecc31-232">이렇게 하면 연결의 양쪽 끝에서 고가용성 연결이 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-232">This will give you a highly available connection at both ends of the connection.</span></span> <span data-ttu-id="ecc31-233">ExpressRoute SLA(서비스 수준 계약)가 필요한 경우 이 옵션이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-233">This is necessary if you require the ExpressRoute service level agreement (SLA).</span></span> <span data-ttu-id="ecc31-234">자세한 내용은 [Azure ExpressRoute의 SLA][sla-for-expressroute]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-234">See [SLA for Azure ExpressRoute][sla-for-expressroute] for details.</span></span>

    <span data-ttu-id="ecc31-235">다음 다이어그램은 중복 온-프레미스 라우터가 기본 및 보조 회로에 연결된 구성을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-235">The following diagram shows a configuration with redundant on-premises routers connected to the primary and secondary circuits.</span></span> <span data-ttu-id="ecc31-236">각 회로는 공용 피어링 및 개인 피어링의 트래픽을 처리합니다(이전 섹션에서 설명한 대로 각 피어링에 한 쌍의 /30 주소 공간이 지정됨).</span><span class="sxs-lookup"><span data-stu-id="ecc31-236">Each circuit handles the traffic for a public peering and a private peering (each peering is designated a pair of /30 address spaces, as described in the previous section).</span></span>

    <span data-ttu-id="ecc31-237">![[1]][1]</span><span class="sxs-lookup"><span data-stu-id="ecc31-237">![[1]][1]</span></span>

* <span data-ttu-id="ecc31-238">레이어 3 연결을 사용하는 경우 연결이 가용성을 처리하는 중복 BGP 세션을 제공하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-238">If you're using a layer 3 connection, verify that it provides redundant BGP sessions that handle availability for you.</span></span>

* <span data-ttu-id="ecc31-239">다른 서비스 공급자가 제공하는 여러 ExpressRoute 회로에 VNet을 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-239">Connect the VNet to multiple ExpressRoute circuits, supplied by different service providers.</span></span> <span data-ttu-id="ecc31-240">이 전략은 추가 고가용성 및 재해 복구 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-240">This strategy provides additional high-availability and disaster recovery capabilities.</span></span>

* <span data-ttu-id="ecc31-241">사이트 간 VPN을 ExpressRoute에 대한 장애 조치(failover) 경로로 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-241">Configure a site-to-site VPN as a failover path for ExpressRoute.</span></span> <span data-ttu-id="ecc31-242">이 옵션에 대한 자세한 내용은 [VPN 장애 조치(failover)와 함께 ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결][highly-available-network-architecture]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-242">For more about this option, see [Connect an on-premises network to Azure using ExpressRoute with VPN failover][highly-available-network-architecture].</span></span>
 <span data-ttu-id="ecc31-243">이 옵션은 개인 피어링에만 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-243">This option only applies to private peering.</span></span> <span data-ttu-id="ecc31-244">Azure 및 Office 365 서비스의 경우 인터넷이 유일한 장애 조치(failover) 경로입니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-244">For Azure and Office 365 services, the Internet is the only failover path.</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="ecc31-245">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ecc31-245">Manageability considerations</span></span>

<span data-ttu-id="ecc31-246">[AzureCT(Azure Connectivity Toolkit)][azurect]를 사용하여 온-프레미스 데이터 센터와 Azure 간의 연결을 모니터할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-246">You can use the [Azure Connectivity Toolkit (AzureCT)][azurect] to monitor connectivity between your on-premises datacenter and Azure.</span></span> 

## <a name="security-considerations"></a><span data-ttu-id="ecc31-247">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ecc31-247">Security considerations</span></span>

<span data-ttu-id="ecc31-248">보안 문제와 준수 요구 사항에 따라 다양한 방법으로 Azure 연결에 대한 보안 옵션을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-248">You can configure security options for your Azure connection in different ways, depending on your security concerns and compliance needs.</span></span> 

<span data-ttu-id="ecc31-249">ExpressRoute는 레이어 3에서 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-249">ExpressRoute operates in layer 3.</span></span> <span data-ttu-id="ecc31-250">응용 프로그램 레이어의 위협은 합법적인 리소스에 대한 트래픽을 제한하는 네트워크 보안 어플라이언스를 사용하여 방지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-250">Threats in the application layer can be prevented by using a network security appliance that restricts traffic to legitimate resources.</span></span> <span data-ttu-id="ecc31-251">또한 공용 피어링을 사용하는 ExpressRoute 연결은 온-프레미스에서만 시작할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-251">Additionally, ExpressRoute connections using public peering can only be initiated from on-premises.</span></span> <span data-ttu-id="ecc31-252">이렇게 하면 악성 서비스가 인터넷에서 온-프레미스 데이터에 액세스하여 손상하는 것을 방지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-252">This prevents a rogue service from accessing and compromising on-premises data from the Internet.</span></span>

<span data-ttu-id="ecc31-253">보안을 최대화하려면 온-프레미스 네트워크와 공급자 에지 라우터 사이에 네트워크 보안 어플라이언스를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-253">To maximize security, add network security appliances between the on-premises network and the provider edge routers.</span></span> <span data-ttu-id="ecc31-254">이렇게 하면 VNet에서 권한 없는 트래픽이 유입되는 것을 제한할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-254">This will help to restrict the inflow of unauthorized traffic from the VNet:</span></span>

<span data-ttu-id="ecc31-255">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="ecc31-255">![[2]][2]</span></span>

<span data-ttu-id="ecc31-256">감사 또는 준수를 위해 VNet에서 실행 중인 구성 요소에서 인터넷에 직접 액세스하지 못하도록 차단하고 [강제 터널링][forced-tuneling]을 구현해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-256">For auditing or compliance purposes, it may be necessary to prohibit direct access from components running in the VNet to the Internet and implement [forced tunneling][forced-tuneling].</span></span> <span data-ttu-id="ecc31-257">이 경우 온-프레미스에서 실행 중인 프록시를 통해 인터넷 트래픽을 감사할 수 있는 위치로 다시 리디렉션해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-257">In this situation, Internet traffic should be redirected back through a proxy running  on-premises where it can be audited.</span></span> <span data-ttu-id="ecc31-258">권한 없는 트래픽의 유출을 차단하고 잠재적인 악성 인바운드 트래픽을 필터링하도록 프록시를 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-258">The proxy can be configured to block unauthorized traffic flowing out, and filter potentially malicious inbound traffic.</span></span>

<span data-ttu-id="ecc31-259">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="ecc31-259">![[3]][3]</span></span>

<span data-ttu-id="ecc31-260">보안을 최대화하려면 VM에 대해 공용 IP 주소를 사용하지 않도록 설정하고 NSG를 사용하여 이러한 VM에 공개적으로 액세스할 수 없도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-260">To maximize security, do not enable a public IP address for your VMs, and use NSGs to ensure that these VMs aren't publicly accessible.</span></span> <span data-ttu-id="ecc31-261">내부 IP 주소를 통해서만 VM을 사용할 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-261">VMs should only be available using the internal IP address.</span></span> <span data-ttu-id="ecc31-262">ExpressRoute 네트워크를 통해 이 주소에 액세스하여 온-프레미스 DevOps 직원이 구성 또는 유지 관리를 수행할 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-262">These addresses can be made accessible through the ExpressRoute network, enabling on-premises DevOps staff to perform configuration or maintenance.</span></span>

<span data-ttu-id="ecc31-263">VM의 관리 끝점을 외부 네트워크에 노출해야 하는 경우 NSG 또는 액세스 제어 목록을 사용하여 이러한 포트의 가시성을 IP 주소 또는 네트워크의 허용 목록으로 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-263">If you must expose management endpoints for VMs to an external network, use NSGs or access control lists to restrict the visibility of these ports to a whitelist of IP addresses or networks.</span></span>

> [!NOTE]
> <span data-ttu-id="ecc31-264">기본적으로 Azure Portal을 통해 배포된 Azure VM에는 로그인 액세스를 제공하는 공용 IP 주소가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-264">By default, Azure VMs deployed through the Azure portal include a public IP address that provides login access.</span></span>  
> 
> 


## <a name="deploy-the-solution"></a><span data-ttu-id="ecc31-265">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="ecc31-265">Deploy the solution</span></span>

<span data-ttu-id="ecc31-266">**필수 조건.**</span><span class="sxs-lookup"><span data-stu-id="ecc31-266">**Prequisites.**</span></span> <span data-ttu-id="ecc31-267">적절한 네트워크 어플라이언스를 사용하여 기존 온-프레미스 인프라가 이미 구성된 상태여야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-267">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="ecc31-268">솔루션을 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-268">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="ecc31-269">아래 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-269">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="ecc31-270">Azure Portal에서 링크가 열릴 때까지 기다린 후 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-270">Wait for the link to open in the Azure portal, then follow these steps:</span></span>
   * <span data-ttu-id="ecc31-271">**리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-hybrid-er-rg`를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-271">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-er-rg` in the text box.</span></span>
   * <span data-ttu-id="ecc31-272">**위치** 드롭다운 상자에서 하위 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-272">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="ecc31-273">**템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-273">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="ecc31-274">사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-274">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="ecc31-275">**구매** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-275">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="ecc31-276">배포가 완료될 때가지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-276">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="ecc31-277">아래 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-277">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="ecc31-278">Azure Portal에서 링크가 열릴 때까지 기다린 후 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-278">Wait for the link to open in the Azure portal, then follow these steps:</span></span>
   * <span data-ttu-id="ecc31-279">**리소스 그룹** 섹션에서 **기존 항목 사용**을 선택하고 텍스트 상자에 `ra-hybrid-er-rg`를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-279">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-er-rg` in the text box.</span></span>
   * <span data-ttu-id="ecc31-280">**위치** 드롭다운 상자에서 하위 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-280">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="ecc31-281">**템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="ecc31-281">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="ecc31-282">사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-282">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="ecc31-283">**구매** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-283">Click the **Purchase** button.</span></span>
6. <span data-ttu-id="ecc31-284">배포가 완료될 때가지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="ecc31-284">Wait for the deployment to complete.</span></span>


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
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[er-circuit-parameters]: https://github.com/mspnp/reference-architectures/tree/master/hybrid-networking/expressroute/parameters/expressRouteCircuit.parameters.json
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[0]: ./images/expressroute.png "Azure ExpressRoute를 사용하는 하이브리드 네트워크 아키텍처"
[1]: ../_images/guidance-hybrid-network-expressroute/figure2.png "ExpressRoute 기본 및 보조 회로와 함께 중복 라우터 사용"
[2]: ../_images/guidance-hybrid-network-expressroute/figure3.png "온-프레미스 네트워크에 보안 장치 추가"
[3]: ../_images/guidance-hybrid-network-expressroute/figure4.png "강제 터널링을 사용하여 인터넷 바인딩된 트래픽 감사"
[4]: ../_images/guidance-hybrid-network-expressroute/figure5.png "ExpressRoute 회로의 ServiceKey 찾기"  