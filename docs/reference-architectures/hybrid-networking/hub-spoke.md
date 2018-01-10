---
title: "Azure에서 허브-스포크 네트워크 토폴로지 구현"
description: "Azure에서 허브-스포크 네트워크 토폴로지를 구현하는 방법입니다."
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="6b513-103">Azure에서 허브-스포크 네트워크 토폴로지 구현</span><span class="sxs-lookup"><span data-stu-id="6b513-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="6b513-104">이 참조 아키텍처는 Azure에서 허브-스포크 토폴로지를 구현하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="6b513-105">*허브*는 Azure의 가상 네트워크(VNet)로서 사용자의 온-프레미스 네트워크에 대한 연결의 중심으로 기능합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="6b513-106">*스포크*는 허브와 피어링하는 VNet으로서 워크로드를 격리하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="6b513-107">트래픽은 ExpressRoute 또는 VPN 게이트웨이 연결을 통해 온-프레미스 데이터 센터와 허브 사이를 흐릅니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="6b513-108">[**이 솔루션을 배포합니다**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="6b513-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="6b513-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="6b513-109">![[0]][0]</span></span>

<span data-ttu-id="6b513-110">*이 아키텍처의 [Visio 파일][visio-download] 다운로드*</span><span class="sxs-lookup"><span data-stu-id="6b513-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="6b513-111">이 토폴로지의 이점은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="6b513-112">**비용 절감** 네트워크 가상 어플라이언스(NVA), DNS 서버와 같이 여러 워크로드에서 공유할 수 있는 서비스를 하나의 위치로 일원화하여 비용을 절감합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="6b513-113">**구독 제한 극복**: 여러 구독의 VNet을 중앙 허브로 피어링하여 구독의 제한을 극복합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="6b513-114">**문제 구분**: 중앙 IT(SecOps, InfraOps)와 워크로드(DevOps)를 문제를 분리합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="6b513-115">이 아키텍처의 일반적인 용도는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="6b513-116">개발, 테스트, 생산과 같이 서로 다른 환경에 배포되고 DNS, IDS, NTP 또는 AD DS와 같은 공유 서비스가 필요한 워크로드.</span><span class="sxs-lookup"><span data-stu-id="6b513-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="6b513-117">공유 서비스는 허브 VNet에 배치되고 각 환경은 스포크에 배포되어 격리 상태 유지.</span><span class="sxs-lookup"><span data-stu-id="6b513-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="6b513-118">서로 연결할 필요가 없으나 공유 서비스에 대한 액세스가 필요한 워크로드.</span><span class="sxs-lookup"><span data-stu-id="6b513-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="6b513-119">DMZ로 기능한 허브의 방화벽, 각 스포크에서 워크로드에 대한 별도의 관리 등 보안 측면에 대한 중앙 제어가 필요한 엔터프라이즈.</span><span class="sxs-lookup"><span data-stu-id="6b513-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="6b513-120">건축</span><span class="sxs-lookup"><span data-stu-id="6b513-120">Architecture</span></span>

<span data-ttu-id="6b513-121">이 아키텍처는 다음 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="6b513-122">**온-프레미스 네트워크**.</span><span class="sxs-lookup"><span data-stu-id="6b513-122">**On-premises network**.</span></span> <span data-ttu-id="6b513-123">조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="6b513-124">**VPN 장치**.</span><span class="sxs-lookup"><span data-stu-id="6b513-124">**VPN device**.</span></span> <span data-ttu-id="6b513-125">온-프레미스 네트워크에 외부 연결을 제공하는 장치 또는 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="6b513-126">VPN 장치는 하드웨어 장치일 수도 있고 Windows Server 2012의 RRAS(라우팅 및 원격 액세스 서비스)와 같은 소프트웨어 솔루션일 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="6b513-127">지원되는 VPN 어플라이언스 목록 및 선택한 VPN 어플라이언스를 Azure에 연결하도록 구성하는 방법에 대한 자세한 내용은 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치 정보][vpn-appliance]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6b513-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="6b513-128">**VPN 가상 네트워크 게이트웨이 또는 ExpressRoute 게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="6b513-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="6b513-129">가상 네트워크 게이트웨이를 사용하면 VNet을 온-프레미스 네트워크에 연결하는 데 사용되는 VPN 장치 또는 ExpressRoute 회로에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="6b513-130">자세한 내용은 [온-프레미스 네트워크를 Microsoft Azure virtual network에 연결][connect-to-an-Azure-vnet]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6b513-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="6b513-131">이 참조 아키텍처의 배포 스크립트는 연결을 위해 VPN 게이트웨이를 사용하고, 사용자의 온-프레미스 네트워크를 시뮬레이션하기 위해 Azure의 VNet을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="6b513-132">**허브 VNet**.</span><span class="sxs-lookup"><span data-stu-id="6b513-132">**Hub VNet**.</span></span> <span data-ttu-id="6b513-133">허브-스포크 토폴로지에서 허브로 사용되는 Azure VNet입니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="6b513-134">허브는 사용자의 온-프레미스 네트워크에 대한 연결의 중앙 위치이자 스포크 VNet에 호스팅된 다양한 워크로드에 의해 사용되는 서비스가 호스팅되는 장소입니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="6b513-135">**게이트웨이 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="6b513-135">**Gateway subnet**.</span></span> <span data-ttu-id="6b513-136">가상 네트워크 게이트웨이는 동일한 서브넷에 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="6b513-137">**공유 서비스 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="6b513-137">**Shared services subnet**.</span></span> <span data-ttu-id="6b513-138">DNS, AD DS를 비롯한 모든 스포크에서 공유될 수 있는 서비스를 호스팅하는 데 사용되는 허브 VNet의 서브넷입니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="6b513-139">**스포크 VNet**.</span><span class="sxs-lookup"><span data-stu-id="6b513-139">**Spoke VNets**.</span></span> <span data-ttu-id="6b513-140">허브-스포크 토폴로지에서 스포크로 사용되는 하나 이상의 Azure VNet입니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-140">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="6b513-141">스포크는 다른 스포크와 별도로 관리되는 자체 VNet에서 워크로드를 격리하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-141">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="6b513-142">각 워크로드에는 Azure Load Balancer를 통해 여러 서브넷이 연결된 여러 계층이 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-142">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="6b513-143">응용 프로그램 인프라에 대한 자세한 내용은 [Windows VM 워크로드 실행][windows-vm-ra] 및 [Linux VM 워크로드 실행][linux-vm-ra]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6b513-143">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="6b513-144">**VNet 피어링**.</span><span class="sxs-lookup"><span data-stu-id="6b513-144">**VNet peering**.</span></span> <span data-ttu-id="6b513-145">동일한 Azure 지역에 있는 2개의 VNet을 [피어링 연결][vnet-peering]을 사용하여 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-145">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="6b513-146">피어링 연결은 VNet 사이에 적용되는 비전이적이고 대기 시간이 낮은 연결입니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-146">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="6b513-147">피어링이 적용되면 VNet은 라우터가 없어도 Azure 백본을 사용하여 트래픽을 교환합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-147">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="6b513-148">허브-스포크 네트워크 토폴로지에서는 VNet 피어링을 사용하여 허브를 각 스포크에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-148">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="6b513-149">이 문서에서는 [리소스 관리자](/azure/azure-resource-manager/resource-group-overview) 배포만 다루고 있지만, 동일한 구독에서 클래식 VNet을 리소스 관리자에 연결할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="6b513-150">이렇게 하면 스포크가 클래식 배포를 호스팅하면서도 허브에서 공유되는 서비스를 이용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>


## <a name="recommendations"></a><span data-ttu-id="6b513-151">권장 사항</span><span class="sxs-lookup"><span data-stu-id="6b513-151">Recommendations</span></span>

<span data-ttu-id="6b513-152">대부분의 시나리오의 경우 다음 권장 사항을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="6b513-153">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="6b513-154">리소스 그룹</span><span class="sxs-lookup"><span data-stu-id="6b513-154">Resource groups</span></span>

<span data-ttu-id="6b513-155">허브 VNet과 각 스포크 VNet은 동일한 Azure 지역의 동일한 Azure AD(Active Directory) 테넌트에 속하는 한 서로 다른 리소스 그룹이나 서로 다른 구독에 구현될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="6b513-156">따라서 허브 VNet에서 관리되는 서비스를 공유하면서도 각 워크로드를 분산 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-156">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="6b513-157">VNet과 GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="6b513-157">VNet and GatewaySubnet</span></span>

<span data-ttu-id="6b513-158">이름이 *GatewaySubnet*인 서브넷을 만듭니다. 주소 범위는 /27로 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-158">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="6b513-159">이 서브넷은 가상 네트워크 게이트웨이에서 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-159">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="6b513-160">이 서브넷에 주소 32개를 할당하면 추후 게이트웨이 크기 제한에 도달하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-160">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="6b513-161">게이트웨이 설정에 대한 자세한 내용은 연결 유형에 따라 다음과 같은 참조 아키텍처를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6b513-161">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="6b513-162">[ExpressRoute를 사용하는 하이브리드 네트워크][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="6b513-162">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="6b513-163">[VPN 게이트웨이를 사용하는 하이브리드 네트워크][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="6b513-163">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="6b513-164">고가용성이 필요한 경우 장애 조치(failover)를 위해 ExpressRoute와 VPN을 모두 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-164">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="6b513-165">[VPN 장애 조치(failover)를 사용하는 ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결][hybrid-ha]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6b513-165">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="6b513-166">온-프레미스 네트워크에 연결하지 않아도 되는 경우에는 게이트웨이 없이 허브-스포크 토폴로지를 사용할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-166">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="6b513-167">VNet 피어링</span><span class="sxs-lookup"><span data-stu-id="6b513-167">VNet peering</span></span>

<span data-ttu-id="6b513-168">VNet 피어링은 두 VNet 사이에 존재하는 비전이적 관계입니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-168">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="6b513-169">스포크를 서로 연결해야 한다면 스포크 사이에 별도의 피어링 연결을 추가하는 방법도 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-169">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="6b513-170">그러나 여러 개의 스포크를 서로 연결해야 하는 경우에는 [VNet 하나당 허용되는 VNet 피어링 개수 제한][vnet-peering-limit]으로 인해 사용 가능한 피어링 연결이 모자라게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-170">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="6b513-171">이 경우에는 목적지가 스포크인 트래픽이 허브 VNet에서 라우터로 기능하는 NVA로 강제로 전달되도록 UDR(사용자 정의 경로)을 사용하는 방법을 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-171">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="6b513-172">이렇게 하면 각 스포크가 다른 스포크와 연결할 수 있게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-172">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="6b513-173">스포크가 허브 VNet 게이트웨이를 사용하여 원격 네트워크와 통신하도록 구성할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-173">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="6b513-174">게이트웨이 트래픽이 스포크에서 허브로 흐르고 원격 네트워크에 연결되도록 허용하려면:</span><span class="sxs-lookup"><span data-stu-id="6b513-174">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="6b513-175">허브의 VNet 피어링 연결이 **게이트웨이 전송을 허용**하도록 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-175">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="6b513-176">각 스포크의 VNet 피어링 연결이 **원격 게이트웨이를 사용**하도록 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-176">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="6b513-177">모든 VNet 피어링 연결이 **전달된 트래픽을 허용**하도록 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-177">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="6b513-178">고려 사항</span><span class="sxs-lookup"><span data-stu-id="6b513-178">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="6b513-179">스포크 연결</span><span class="sxs-lookup"><span data-stu-id="6b513-179">Spoke connectivity</span></span>

<span data-ttu-id="6b513-180">스포크 사이의 연결이 필요한 경우 허브에서의 라우팅을 위해 NVA를 구현하고 트래픽이 허브로 전달되도록 스포크에서 UDR을 사용하는 방법을 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-180">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="6b513-181">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="6b513-181">![[2]][2]</span></span>

<span data-ttu-id="6b513-182">이 시나리오에서는 피어링 연결이 **전달된 트래픽을 허용**하도록 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-182">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="6b513-183">VNet 피어링 제한 문제 해결</span><span class="sxs-lookup"><span data-stu-id="6b513-183">Overcoming VNet peering limits</span></span>

<span data-ttu-id="6b513-184">반드시 Azure의 [VNet 하나당 허용되는 VNet 피어링 개수 제한][vnet-peering-limit]을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-184">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="6b513-185">허용되는 한도보다 많은 스포크가 필요한 경우 첫 번째 수준의 스포크가 허브로서 기능하는 허브-스포크-허브-스포크 토폴로지를 만드는 방법을 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-185">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="6b513-186">다음 다이어그램은 이 토폴로지를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-186">The following diagram shows this approach.</span></span>

<span data-ttu-id="6b513-187">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="6b513-187">![[3]][3]</span></span>

<span data-ttu-id="6b513-188">또한, 허브가 다수의 스포크에 맞게 확장될 수 있도록 허브에서 어떤 서비스가 공유되는지도 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-188">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="6b513-189">예를 들어 허브에서 방화벽 서비스를 제공한다면 복수의 스포크를 추가할 때 방화벽 솔루션의 대역폭 제한을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-189">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="6b513-190">공유 서비스 중 일부를 두 번째 수준의 허브로 이동하는 것이 좋을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-190">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="6b513-191">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="6b513-191">Deploy the solution</span></span>

<span data-ttu-id="6b513-192">이 아키텍처에 대한 배포는 [GitHub][ref-arch-repo]에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-192">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="6b513-193">이 아키텍처에 대한 배포는 연결을 테스트하기 위해 각 VNet에서 Ubuntu VM을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-193">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="6b513-194">**허브 VNet**의 **shared-services** 서브넷에 실제로 호스팅된 서비스는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-194">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="6b513-195">필수 조건</span><span class="sxs-lookup"><span data-stu-id="6b513-195">Prerequisites</span></span>

<span data-ttu-id="6b513-196">사용자의 구독에 참조 아키텍처를 배포하려면 먼저 다음 단계를 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-196">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="6b513-197">[AzureCAT 참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-197">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="6b513-198">Azure CLI를 사용하려면 컴퓨터에 Azure CLI 2.0이 설치되어 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-198">If you prefer to use the Azure CLI, make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="6b513-199">CLI를 설치하려면 [Install Azure CLI 2.0][azure-cli-2](Azure CLI 2.0 설치)에 제시된 지침을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6b513-199">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="6b513-200">PowerShell을 사용하려면 컴퓨터에 Azure용 최신 PowerShell 모듈이 설치되어 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-200">If you prefer to use PowerShell, make sure you have the latest PowerShell module for Azure installed on you computer.</span></span> <span data-ttu-id="6b513-201">최신 Azure PowerShell 모듈을 설치하려면 [Azure용 PowerShell 설치][azure-powershell]에 제시된 지침을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="6b513-201">To install the latest Azure PowerShell module, follow the instructions in [Install PowerShell for Azure][azure-powershell].</span></span>

4. <span data-ttu-id="6b513-202">명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 다음 명령 중 하나를 사용하여 Azure 계정에 로그인한 다음 프롬프트에 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-202">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="6b513-203">시뮬레이션된 온-프레미스 데이터 센터 배포</span><span class="sxs-lookup"><span data-stu-id="6b513-203">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="6b513-204">시뮬레이션된 온-프레미스 데이터 센터를 Azure VNet으로서 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-204">To deploy the simulated on-premises datacenter as an Azure VNet, perform the following steps.</span></span>

1. <span data-ttu-id="6b513-205">위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\onprem` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-205">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="6b513-206">`onprem.vm.parameters.json` 파일을 열고 아래에 나와 있는 대로 11열과 12열에 큰따옴표 사이에 사용자 이름과 암호를 입력한 다음 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-206">Open the `onprem.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="6b513-207">아래의 bash 또는 PowerShell 명령을 실행하여 시뮬레이션된 온-프레미스 환경을 Azure의 VNet으로서 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-207">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="6b513-208">값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-208">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="6b513-209">`ra-onprem-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-209">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="6b513-210">배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-210">Wait for the deployment to finish.</span></span> <span data-ttu-id="6b513-211">이 배포는 가상 네트워크, Ubuntu를 실행하는 가상 머신 및 VPN 게이트웨이를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-211">This deployment creates a virtual network, a virtual machine running Ubuntu, and a VPN gateway.</span></span> <span data-ttu-id="6b513-212">VPN 게이트웨이의 생성이 완료되기까지 40분 이상이 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-212">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="6b513-213">Azure 허브 VNet</span><span class="sxs-lookup"><span data-stu-id="6b513-213">Azure hub VNet</span></span>

<span data-ttu-id="6b513-214">허브 VNet을 배포하고 위에서 만든 시뮬레이션된 온-프레미스 VNet에 연결하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-214">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="6b513-215">위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\hub` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-215">Navigate to the `hybrid-networking\hub-spoke\hub` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="6b513-216">`hub.vm.parameters.json` 파일을 열고 아래에 나와 있는 대로 11열과 12열에 큰따옴표 사이에 사용자 이름과 암호를 입력한 다음 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-216">Open the `hub.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="6b513-217">`hub.gateway.parameters.json` 파일을 열고 아래에 나와 있는 대로 23열에 큰따옴표 사이에 공유 키를 입력한 다음 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-217">Open the `hub.gateway.parameters.json` file and enter a shared key between the quotes in line 23, as shown below, then save the file.</span></span> <span data-ttu-id="6b513-218">이 값은 나중에 배포에서 필요하므로 따로 기록해 둡니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-218">Keep a note of this value, you will need to use it later in the deployment.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="6b513-219">아래의 bash 또는 PowerShell 명령을 실행하여 시뮬레이션된 온-프레미스 환경을 Azure의 VNet으로서 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-219">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="6b513-220">값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-220">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="6b513-221">`ra-hub-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-221">If you decide to use a different resource group name (other than `ra-hub-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="6b513-222">배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-222">Wait for the deployment to finish.</span></span> <span data-ttu-id="6b513-223">이 배포는 가상 네트워크, Ubuntu를 실행하는 가상 머신, VPN 게이트웨이 및 이전 섹션에서 생성한 게이트웨이에 대한 연결을 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-223">This deployment creates a virtual network, a virtual machine running Ubuntu, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="6b513-224">VPN 게이트웨이의 생성이 완료되기까지 40분 이상이 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-224">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="connection-from-on-premises-to-the-hub"></a><span data-ttu-id="6b513-225">온-프레미스에서 허브로의 연결</span><span class="sxs-lookup"><span data-stu-id="6b513-225">Connection from on-premises to the hub</span></span>

<span data-ttu-id="6b513-226">시뮬레이션된 온-프레미스 데이터 센터를 허브 VNet에 연결하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-226">To connect from the simulated on-premises datacenter to the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="6b513-227">위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\onprem` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-227">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="6b513-228">`onprem.connection.parameters.json` 파일을 열고 아래에 나와 있는 대로 9열에 큰따옴표 사이에 공유 키를 입력한 다음 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-228">Open the `onprem.connection.parameters.json` file and enter a shared key between the quotes in line 9, as shown below, then save the file.</span></span> <span data-ttu-id="6b513-229">이 공유 키 값은 앞에서 배포한 온-프레미스 게이트웨이에서 사용하는 값과 동일해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-229">This shared key value must be the same used in the on-premises gateway you deployed previously.</span></span>

  ```bash
  "sharedKey": "",
  ```

3. <span data-ttu-id="6b513-230">아래의 bash 또는 PowerShell 명령을 실행하여 시뮬레이션된 온-프레미스 환경을 Azure의 VNet으로서 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-230">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="6b513-231">값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-231">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="6b513-232">`ra-onprem-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-232">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="6b513-233">배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-233">Wait for the deployment to finish.</span></span> <span data-ttu-id="6b513-234">이 배포는 온-프레미스 데이터 센터를 시뮬레이션하는 데 사용되는 VNet과 허브 VNet 사이의 연결을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-234">This deployment creates a connection between the VNet used to simulate an on-premises datacenter, and the hub VNet.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="6b513-235">Azure 스포크 VNet</span><span class="sxs-lookup"><span data-stu-id="6b513-235">Azure spoke VNets</span></span>

<span data-ttu-id="6b513-236">스포크 VNet을 배포하고 위에서 만든 허브 VNet에 연결하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-236">To deploy the spoke VNets, and connect to the hub VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="6b513-237">위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\spokes` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-237">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="6b513-238">`spoke1.web.parameters.json` 파일을 열고 아래에 나와 있는 대로 53열과 54열에 큰따옴표 사이에 사용자 이름과 암호를 입력한 다음 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-238">Open the `spoke1.web.parameters.json` file and enter a username and password between the quotes in line 53 and 54, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="6b513-239">파일 `spoke2.web.parameters.json`에 대해 직전 단계를 반복합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-239">Repeat the previous step for file `spoke2.web.parameters.json`.</span></span>

4. <span data-ttu-id="6b513-240">아래의 bash 또는 PowerShell 명령을 실행하여 첫 번째 스포크를 배포하고 허브에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-240">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="6b513-241">값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-241">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > <span data-ttu-id="6b513-242">`ra-spoke1-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-242">If you decide to use a different resource group name (other than `ra-spoke1-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="6b513-243">배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-243">Wait for the deployment to finish.</span></span> <span data-ttu-id="6b513-244">이 배포는 가상 네트워크, Ubuntu와 Apache를 실행하는 3개의 가상 머신 및 이전 섹션에서 생성한 허브 VNet에 대한 VNet 피어링 연결을 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-244">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="6b513-245">이 배포가 완료되기까지 20분이 넘게 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-245">This deployment may take over 20 minutes.</span></span>

6. <span data-ttu-id="6b513-246">아래의 bash 또는 PowerShell 명령을 실행하여 첫 번째 스포크를 배포하고 허브에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-246">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="6b513-247">값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-247">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > <span data-ttu-id="6b513-248">`ra-spoke2-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-248">If you decide to use a different resource group name (other than `ra-spoke2-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="6b513-249">배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-249">Wait for the deployment to finish.</span></span> <span data-ttu-id="6b513-250">이 배포는 가상 네트워크, Ubuntu와 Apache를 실행하는 3개의 가상 머신 및 이전 섹션에서 생성한 허브 VNet에 대한 VNet 피어링 연결을 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-250">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="6b513-251">이 배포가 완료되기까지 20분이 넘게 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-251">This deployment may take over 20 minutes.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="6b513-252">스포크 VNet에 대한 Azure 허브 VNet 피어링</span><span class="sxs-lookup"><span data-stu-id="6b513-252">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="6b513-253">허브 VNet에 대한 VNet 피어링 연결을 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-253">To deploy the VNet peering connections for the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="6b513-254">위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\hub` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-254">Switch to the `hybrid-networking\hub-spoke\hub` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="6b513-255">아래의 bash 또는 PowerShell 명령을 실행하여 첫 번째 스포크로의 피어링 연결을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-255">Run the bash or PowerShell command below to deploy the peering connection to the first spoke.</span></span> <span data-ttu-id="6b513-256">값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-256">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. <span data-ttu-id="6b513-257">아래의 bash 또는 PowerShell 명령을 실행하여 두 번째 스포크로의 피어링 연결을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-257">Run the bash or PowerShell command below to deploy the peering connection to the second spoke.</span></span> <span data-ttu-id="6b513-258">값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-258">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a><span data-ttu-id="6b513-259">연결 테스트</span><span class="sxs-lookup"><span data-stu-id="6b513-259">Test connectivity</span></span>

<span data-ttu-id="6b513-260">온-프레미스 데이터 센터 배포에 연결된 허브-스포크 토폴로지가 올바르게 작동하는지 확인하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-260">To verify that the hub-spoke topology connected to an on-premises datacenter deployment worked, follow these steps.</span></span>

1. <span data-ttu-id="6b513-261">[Azure Portal][포털]에서 사용자의 구독에 연결한 다음 `ra-onprem-rg` 리소스 그룹에 있는 `ra-onprem-vm1` 가상 머신으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-261">From the [Azure portal][portal], connect to your subscription, and navigate to the `ra-onprem-vm1` virtual machine in the `ra-onprem-rg` resource group.</span></span>

2. <span data-ttu-id="6b513-262">`Overview` 블레이드에서 VM의 `Public IP address`를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-262">In the `Overview` blade, note the `Public IP address` for the VM.</span></span>

3. <span data-ttu-id="6b513-263">SSH 클라이언트에서 배포 중에 지정한 사용자 이름과 암호를 사용하여 위에서 확인한 IP 주소에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-263">Use an SSH client to connect to the IP address you noted above using the user name and password you specified during deployment.</span></span>

4. <span data-ttu-id="6b513-264">접속한 VM의 명령 프롬프트에서 아래의 명령을 실행하여 온-프레미스 VNet과 Spoke1 VNet 사이의 연결을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-264">From the command prompt on the VM you connected to, run the command below to test connectivity from the on-premises VNet to the Spoke1 VNet.</span></span>

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="6b513-265">스포크 사이의 연결 추가</span><span class="sxs-lookup"><span data-stu-id="6b513-265">Add connectivity between spokes</span></span>

<span data-ttu-id="6b513-266">각 스포크 사이의 연결을 허용하려면 목적지가 다른 스포크인 트래픽을 허브 VNet의 게이트웨이로 전달하는 UDR을 각 스포크에 배포해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-266">If you want to allow spokes to connect to each other, you must deploy UDRs to each spoke that forward traffic destined to other spokes to the gateway in the hub VNet.</span></span> <span data-ttu-id="6b513-267">아래의 단계를 수행하여 현재 하나의 스포크에서 다른 스포크로 연결할 수 없음을 확인한 다음 UDR을 배포하고 연결을 다시 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-267">Perform the following steps to verify that currently you are not able to connect from a spoke to another, then deploy the UDRs and test connectivity again.</span></span>

1. <span data-ttu-id="6b513-268">더 이상 jumpbox VM에 연결되어 있지 않다면 위의 1~4단계를 반복합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-268">Repeat steps 1 to 4 above, if you are not connected to the jumpbox VM any longer.</span></span>

2. <span data-ttu-id="6b513-269">스포크 1에 있는 웹 서버 중 하나에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-269">Connect to one of the web servers in spoke 1.</span></span>

  ```bash
  ssh 10.1.1.37
  ```

3. <span data-ttu-id="6b513-270">스포크 1과 스포크 2 사이의 연결을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-270">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="6b513-271">연결되지 않는 것이 정상입니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-271">It should fail.</span></span>

  ```bash
  ping 10.1.2.37
  ```

4. <span data-ttu-id="6b513-272">컴퓨터의 명령 프롬프트로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-272">Switch back to your computer's command prompt.</span></span>

5. <span data-ttu-id="6b513-273">위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\spokes` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-273">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you downloaded in the pre-requisites step above.</span></span>

6. <span data-ttu-id="6b513-274">아래의 bash 또는 PowerShell 명령을 실행하여 첫 번째 스포크에 UDR을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-274">Run the bash or PowerShell command below to deploy an UDR to the first spoke.</span></span> <span data-ttu-id="6b513-275">값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-275">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. <span data-ttu-id="6b513-276">아래의 bash 또는 PowerShell 명령을 실행하여 두 번째 스포크에 UDR을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-276">Run the bash or PowerShell command below to deploy an UDR to the second spoke.</span></span> <span data-ttu-id="6b513-277">값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-277">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. <span data-ttu-id="6b513-278">다시 ssh 터미널로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-278">Switch back to the ssh terminal.</span></span>

9. <span data-ttu-id="6b513-279">스포크 1과 스포크 2 사이의 연결을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-279">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="6b513-280">이번에는 연결에 성공해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6b513-280">It should succeed.</span></span>

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Azure의 허브-스포크 토폴로지"
[1]: ./images/hub-spoke-gateway-routing.svg "전이적 라우팅이 사용되는 Azure의 허브-스포크 토폴로지"
[2]: ./images/hub-spoke-no-gateway-routing.svg "NVA로 전이적 라우팅이 사용되는 Azure의 허브-스포크 토폴로지"
[3]: ./images/hub-spokehub-spoke.svg "Azure의 허브-스포크-허브-스포크 토폴로지"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
