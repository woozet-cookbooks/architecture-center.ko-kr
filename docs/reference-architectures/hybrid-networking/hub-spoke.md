---
title: Azure에서 허브-스포크 네트워크 토폴로지 구현
description: Azure에서 허브-스포크 네트워크 토폴로지를 구현하는 방법입니다.
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 925e0f47cf6b9aa1ad48ffae2c9561a2393bf601
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142253"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="0bcc0-103">Azure에서 허브-스포크 네트워크 토폴로지 구현</span><span class="sxs-lookup"><span data-stu-id="0bcc0-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="0bcc0-104">이 참조 아키텍처는 Azure에서 허브-스포크 토폴로지를 구현하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="0bcc0-105">*허브*는 Azure의 가상 네트워크(VNet)로서 사용자의 온-프레미스 네트워크에 대한 연결의 중심으로 기능합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="0bcc0-106">*스포크*는 허브와 피어링하는 VNet으로서 워크로드를 격리하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="0bcc0-107">트래픽은 ExpressRoute 또는 VPN 게이트웨이 연결을 통해 온-프레미스 데이터 센터와 허브 사이를 흐릅니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="0bcc0-108">[**이 솔루션을 배포합니다**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="0bcc0-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="0bcc0-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="0bcc0-109">![[0]][0]</span></span>

<span data-ttu-id="0bcc0-110">*이 아키텍처의 [Visio 파일][visio-download] 다운로드*</span><span class="sxs-lookup"><span data-stu-id="0bcc0-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="0bcc0-111">이 토폴로지의 이점은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="0bcc0-112">**비용 절감** 네트워크 가상 어플라이언스(NVA), DNS 서버와 같이 여러 워크로드에서 공유할 수 있는 서비스를 하나의 위치로 일원화하여 비용을 절감합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="0bcc0-113">**구독 제한 극복**: 여러 구독의 VNet을 중앙 허브로 피어링하여 구독의 제한을 극복합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="0bcc0-114">**문제 구분**: 중앙 IT(SecOps, InfraOps)와 워크로드(DevOps)를 문제를 분리합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="0bcc0-115">이 아키텍처의 일반적인 용도는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="0bcc0-116">개발, 테스트, 생산과 같이 서로 다른 환경에 배포되고 DNS, IDS, NTP 또는 AD DS와 같은 공유 서비스가 필요한 워크로드.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="0bcc0-117">공유 서비스는 허브 VNet에 배치되고 각 환경은 스포크에 배포되어 격리 상태 유지.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="0bcc0-118">서로 연결할 필요가 없으나 공유 서비스에 대한 액세스가 필요한 워크로드.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="0bcc0-119">DMZ로 기능한 허브의 방화벽, 각 스포크에서 워크로드에 대한 별도의 관리 등 보안 측면에 대한 중앙 제어가 필요한 엔터프라이즈.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="0bcc0-120">아키텍처</span><span class="sxs-lookup"><span data-stu-id="0bcc0-120">Architecture</span></span>

<span data-ttu-id="0bcc0-121">이 아키텍처는 다음 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="0bcc0-122">**온-프레미스 네트워크**.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-122">**On-premises network**.</span></span> <span data-ttu-id="0bcc0-123">조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="0bcc0-124">**VPN 장치**.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-124">**VPN device**.</span></span> <span data-ttu-id="0bcc0-125">온-프레미스 네트워크에 외부 연결을 제공하는 장치 또는 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="0bcc0-126">VPN 장치는 하드웨어 장치일 수도 있고 Windows Server 2012의 RRAS(라우팅 및 원격 액세스 서비스)와 같은 소프트웨어 솔루션일 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="0bcc0-127">지원되는 VPN 어플라이언스 목록 및 선택한 VPN 어플라이언스를 Azure에 연결하도록 구성하는 방법에 대한 자세한 내용은 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치 정보][vpn-appliance]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="0bcc0-128">**VPN 가상 네트워크 게이트웨이 또는 ExpressRoute 게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="0bcc0-129">가상 네트워크 게이트웨이를 사용하면 VNet을 온-프레미스 네트워크에 연결하는 데 사용되는 VPN 장치 또는 ExpressRoute 회로에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="0bcc0-130">자세한 내용은 [온-프레미스 네트워크를 Microsoft Azure virtual network에 연결][connect-to-an-Azure-vnet]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="0bcc0-131">이 참조 아키텍처의 배포 스크립트는 연결을 위해 VPN 게이트웨이를 사용하고, 사용자의 온-프레미스 네트워크를 시뮬레이션하기 위해 Azure의 VNet을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="0bcc0-132">**허브 VNet**.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-132">**Hub VNet**.</span></span> <span data-ttu-id="0bcc0-133">허브-스포크 토폴로지에서 허브로 사용되는 Azure VNet입니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="0bcc0-134">허브는 사용자의 온-프레미스 네트워크에 대한 연결의 중앙 위치이자 스포크 VNet에 호스팅된 다양한 워크로드에 의해 사용되는 서비스가 호스팅되는 장소입니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="0bcc0-135">**게이트웨이 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-135">**Gateway subnet**.</span></span> <span data-ttu-id="0bcc0-136">가상 네트워크 게이트웨이는 동일한 서브넷에 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="0bcc0-137">**스포크 VNet**.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-137">**Spoke VNets**.</span></span> <span data-ttu-id="0bcc0-138">허브-스포크 토폴로지에서 스포크로 사용되는 하나 이상의 Azure VNet입니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="0bcc0-139">스포크는 다른 스포크와 별도로 관리되는 자체 VNet에서 워크로드를 격리하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="0bcc0-140">각 워크로드에는 Azure Load Balancer를 통해 여러 서브넷이 연결된 여러 계층이 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="0bcc0-141">응용 프로그램 인프라에 대한 자세한 내용은 [Windows VM 워크로드 실행][windows-vm-ra] 및 [Linux VM 워크로드 실행][linux-vm-ra]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="0bcc0-142">**VNet 피어링**.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-142">**VNet peering**.</span></span> <span data-ttu-id="0bcc0-143">동일한 Azure 지역에 있는 2개의 VNet을 [피어링 연결][vnet-peering]을 사용하여 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="0bcc0-144">피어링 연결은 VNet 사이에 적용되는 비전이적이고 대기 시간이 낮은 연결입니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="0bcc0-145">피어링이 적용되면 VNet은 라우터가 없어도 Azure 백본을 사용하여 트래픽을 교환합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="0bcc0-146">허브-스포크 네트워크 토폴로지에서는 VNet 피어링을 사용하여 허브를 각 스포크에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="0bcc0-147">이 문서에서는 [리소스 관리자](/azure/azure-resource-manager/resource-group-overview) 배포만 다루고 있지만, 동일한 구독에서 클래식 VNet을 리소스 관리자에 연결할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="0bcc0-148">이렇게 하면 스포크가 클래식 배포를 호스팅하면서도 허브에서 공유되는 서비스를 이용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="0bcc0-149">권장 사항</span><span class="sxs-lookup"><span data-stu-id="0bcc0-149">Recommendations</span></span>

<span data-ttu-id="0bcc0-150">대부분의 시나리오의 경우 다음 권장 사항을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="0bcc0-151">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="0bcc0-152">리소스 그룹</span><span class="sxs-lookup"><span data-stu-id="0bcc0-152">Resource groups</span></span>

<span data-ttu-id="0bcc0-153">허브 VNet과 각 스포크 VNet은 동일한 Azure 지역의 동일한 Azure AD(Active Directory) 테넌트에 속하는 한 서로 다른 리소스 그룹이나 서로 다른 구독에 구현될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="0bcc0-154">따라서 허브 VNet에서 관리되는 서비스를 공유하면서도 각 워크로드를 분산 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="0bcc0-155">VNet과 GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="0bcc0-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="0bcc0-156">이름이 *GatewaySubnet*인 서브넷을 만듭니다. 주소 범위는 /27로 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="0bcc0-157">이 서브넷은 가상 네트워크 게이트웨이에서 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="0bcc0-158">이 서브넷에 주소 32개를 할당하면 추후 게이트웨이 크기 제한에 도달하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="0bcc0-159">게이트웨이 설정에 대한 자세한 내용은 연결 유형에 따라 다음과 같은 참조 아키텍처를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="0bcc0-160">[ExpressRoute를 사용하는 하이브리드 네트워크][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="0bcc0-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="0bcc0-161">[VPN 게이트웨이를 사용하는 하이브리드 네트워크][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="0bcc0-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="0bcc0-162">고가용성이 필요한 경우 장애 조치(failover)를 위해 ExpressRoute와 VPN을 모두 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="0bcc0-163">[VPN 장애 조치(failover)를 사용하는 ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결][hybrid-ha]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="0bcc0-164">온-프레미스 네트워크에 연결하지 않아도 되는 경우에는 게이트웨이 없이 허브-스포크 토폴로지를 사용할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="0bcc0-165">VNet 피어링</span><span class="sxs-lookup"><span data-stu-id="0bcc0-165">VNet peering</span></span>

<span data-ttu-id="0bcc0-166">VNet 피어링은 두 VNet 사이에 존재하는 비전이적 관계입니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="0bcc0-167">스포크를 서로 연결해야 한다면 스포크 사이에 별도의 피어링 연결을 추가하는 방법도 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="0bcc0-168">그러나 여러 개의 스포크를 서로 연결해야 하는 경우에는 [VNet 하나당 허용되는 VNet 피어링 개수 제한][vnet-peering-limit]으로 인해 사용 가능한 피어링 연결이 모자라게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="0bcc0-169">이 경우에는 목적지가 스포크인 트래픽이 허브 VNet에서 라우터로 기능하는 NVA로 강제로 전달되도록 UDR(사용자 정의 경로)을 사용하는 방법을 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="0bcc0-170">이렇게 하면 각 스포크가 다른 스포크와 연결할 수 있게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="0bcc0-171">스포크가 허브 VNet 게이트웨이를 사용하여 원격 네트워크와 통신하도록 구성할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="0bcc0-172">게이트웨이 트래픽이 스포크에서 허브로 흐르고 원격 네트워크에 연결되도록 허용하려면:</span><span class="sxs-lookup"><span data-stu-id="0bcc0-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="0bcc0-173">허브의 VNet 피어링 연결이 **게이트웨이 전송을 허용**하도록 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="0bcc0-174">각 스포크의 VNet 피어링 연결이 **원격 게이트웨이를 사용**하도록 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="0bcc0-175">모든 VNet 피어링 연결이 **전달된 트래픽을 허용**하도록 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="0bcc0-176">고려 사항</span><span class="sxs-lookup"><span data-stu-id="0bcc0-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="0bcc0-177">스포크 연결</span><span class="sxs-lookup"><span data-stu-id="0bcc0-177">Spoke connectivity</span></span>

<span data-ttu-id="0bcc0-178">스포크 사이의 연결이 필요한 경우 허브에서의 라우팅을 위해 NVA를 구현하고 트래픽이 허브로 전달되도록 스포크에서 UDR을 사용하는 방법을 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="0bcc0-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="0bcc0-179">![[2]][2]</span></span>

<span data-ttu-id="0bcc0-180">이 시나리오에서는 피어링 연결이 **전달된 트래픽을 허용**하도록 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="0bcc0-181">VNet 피어링 제한 문제 해결</span><span class="sxs-lookup"><span data-stu-id="0bcc0-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="0bcc0-182">반드시 Azure의 [VNet 하나당 허용되는 VNet 피어링 개수 제한][vnet-peering-limit]을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="0bcc0-183">허용되는 한도보다 많은 스포크가 필요한 경우 첫 번째 수준의 스포크가 허브로서 기능하는 허브-스포크-허브-스포크 토폴로지를 만드는 방법을 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="0bcc0-184">다음 다이어그램은 이 토폴로지를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="0bcc0-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="0bcc0-185">![[3]][3]</span></span>

<span data-ttu-id="0bcc0-186">또한, 허브가 다수의 스포크에 맞게 확장될 수 있도록 허브에서 어떤 서비스가 공유되는지도 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="0bcc0-187">예를 들어 허브에서 방화벽 서비스를 제공한다면 복수의 스포크를 추가할 때 방화벽 솔루션의 대역폭 제한을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="0bcc0-188">공유 서비스 중 일부를 두 번째 수준의 허브로 이동하는 것이 좋을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="0bcc0-189">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="0bcc0-189">Deploy the solution</span></span>

<span data-ttu-id="0bcc0-190">이 아키텍처에 대한 배포는 [GitHub][ref-arch-repo]에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="0bcc0-191">이 아키텍처에 대한 배포는 연결을 테스트하기 위해 각 VNet에서 VM을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-191">It uses VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="0bcc0-192">**허브 VNet**의 **shared-services** 서브넷에 실제로 호스팅된 서비스는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

<span data-ttu-id="0bcc0-193">배포는 구독에서 다음과 같은 리소스 그룹을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-193">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="0bcc0-194">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="0bcc0-194">hub-nva-rg</span></span>
- <span data-ttu-id="0bcc0-195">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="0bcc0-195">hub-vnet-rg</span></span>
- <span data-ttu-id="0bcc0-196">onprem-jb-rg</span><span class="sxs-lookup"><span data-stu-id="0bcc0-196">onprem-jb-rg</span></span>
- <span data-ttu-id="0bcc0-197">onprem-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="0bcc0-197">onprem-vnet-rg</span></span>
- <span data-ttu-id="0bcc0-198">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="0bcc0-198">spoke1-vnet-rg</span></span>
- <span data-ttu-id="0bcc0-199">spoke2-vent-rg</span><span class="sxs-lookup"><span data-stu-id="0bcc0-199">spoke2-vent-rg</span></span>

<span data-ttu-id="0bcc0-200">템플릿 매개 변수 파일은 이러한 이름을 참조하므로 이름을 변경하는 경우 매개 변수 파일을 일치하도록 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-200">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="0bcc0-201">필수 조건</span><span class="sxs-lookup"><span data-stu-id="0bcc0-201">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="0bcc0-202">시뮬레이션된 온-프레미스 데이터 센터 배포</span><span class="sxs-lookup"><span data-stu-id="0bcc0-202">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="0bcc0-203">시뮬레이션된 온-프레미스 데이터 센터를 Azure VNet으로서 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-203">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="0bcc0-204">참조 아키텍처 리포지토리의 `hybrid-networking/hub-spoke` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-204">Navigate to the `hybrid-networking/hub-spoke` folder of the reference architectures repository.</span></span>

2. <span data-ttu-id="0bcc0-205">`onprem.json` 파일을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-205">Open the `onprem.json` file.</span></span> <span data-ttu-id="0bcc0-206">`adminUsername` 및 `adminPassword`에 대한 값을 대체합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-206">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. <span data-ttu-id="0bcc0-207">(선택 사항) Linux 배포의 경우 `osType`을 `Linux`로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-207">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

4. <span data-ttu-id="0bcc0-208">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="0bcc0-208">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. <span data-ttu-id="0bcc0-209">배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-209">Wait for the deployment to finish.</span></span> <span data-ttu-id="0bcc0-210">이 배포는 가상 네트워크, 가상 머신 및 VPN 게이트웨이를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-210">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="0bcc0-211">VPN 게이트웨이를 만드는 데 약 40분 정도 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-211">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="0bcc0-212">허브 VNet 배포</span><span class="sxs-lookup"><span data-stu-id="0bcc0-212">Deploy the hub VNet</span></span>

<span data-ttu-id="0bcc0-213">허브 VNet을 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-213">To deploy the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="0bcc0-214">`hub-vnet.json` 파일을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-214">Open the `hub-vnet.json` file.</span></span> <span data-ttu-id="0bcc0-215">`adminUsername` 및 `adminPassword`에 대한 값을 대체합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-215">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="0bcc0-216">(선택 사항) Linux 배포의 경우 `osType`을 `Linux`로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-216">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="0bcc0-217">`sharedKey`의 경우 VPN 연결에 대한 공유 키를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-217">For `sharedKey`, enter a shared key for the VPN connection.</span></span> 

    ```bash
    "sharedKey": "",
    ```

4. <span data-ttu-id="0bcc0-218">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="0bcc0-218">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. <span data-ttu-id="0bcc0-219">배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-219">Wait for the deployment to finish.</span></span> <span data-ttu-id="0bcc0-220">이 배포는 가상 네트워크, 가상 머신, VPN 게이트웨이 및 게이트웨이에 대한 연결을 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-220">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway.</span></span>  <span data-ttu-id="0bcc0-221">VPN 게이트웨이를 만드는 데 약 40분 정도 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-221">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="test-connectivity-with-the-hub"></a><span data-ttu-id="0bcc0-222">허브를 사용하여 연결 테스트</span><span class="sxs-lookup"><span data-stu-id="0bcc0-222">Test connectivity with the hub</span></span>

<span data-ttu-id="0bcc0-223">시뮬레이션된 온-프레미스 환경에서 허브 VNet으로의 연결을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-223">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

<span data-ttu-id="0bcc0-224">**Windows 배포**</span><span class="sxs-lookup"><span data-stu-id="0bcc0-224">**Windows deployment**</span></span>

1. <span data-ttu-id="0bcc0-225">Azure Portal을 사용하여 `onprem-jb-rg` 리소스 그룹에서 `jb-vm1`이라는 VM을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-225">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="0bcc0-226">`Connect`를 클릭하여 VM에 대한 원격 데스크톱 세션을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-226">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="0bcc0-227">`onprem.json` 매개 변수 파일에서 지정한 암호를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-227">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="0bcc0-228">VM에서 PowerShell 콘솔을 열고, `Test-NetConnection` cmdlet을 사용하여 허브 VNet에서 jumpbox VM에 연결할 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-228">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
<span data-ttu-id="0bcc0-229">출력은 다음과 비슷해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-229">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="0bcc0-230">기본적으로 Windows Server VM을 사용하면 Azure에서 ICMP 응답을 허용하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-230">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="0bcc0-231">`ping`을 사용하여 연결을 테스트하려는 경우 각 VM에 대한 Windows 고급 방화벽에서 ICMP 트래픽을 사용하도록 설정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-231">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="0bcc0-232">**Linux 배포**</span><span class="sxs-lookup"><span data-stu-id="0bcc0-232">**Linux deployment**</span></span>

1. <span data-ttu-id="0bcc0-233">Azure Portal을 사용하여 `onprem-jb-rg` 리소스 그룹에서 `jb-vm1`이라는 VM을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-233">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="0bcc0-234">`Connect`를 클릭하고 포털에 표시되는 `ssh` 명령을 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-234">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="0bcc0-235">Linux 프롬프트에서 `ssh`를 실행하여 시뮬레이션된 온-프레미스 환경에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-235">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="0bcc0-236">`onprem.json` 매개 변수 파일에서 지정한 암호를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-236">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="0bcc0-237">`ping` 명령을 사용하여 허브 VNet에서 jumpbox VM에 대한 연결을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-237">Use the `ping` command to test connectivity to the jumpbox VM in the hub VNet:</span></span>

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="0bcc0-238">스포크 VNet 배포</span><span class="sxs-lookup"><span data-stu-id="0bcc0-238">Deploy the spoke VNets</span></span>

<span data-ttu-id="0bcc0-239">스포크 VNet을 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-239">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="0bcc0-240">`spoke1.json` 파일을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-240">Open the `spoke1.json` file.</span></span> <span data-ttu-id="0bcc0-241">`adminUsername` 및 `adminPassword`에 대한 값을 대체합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-241">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="0bcc0-242">(선택 사항) Linux 배포의 경우 `osType`을 `Linux`로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-242">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="0bcc0-243">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="0bcc0-243">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. <span data-ttu-id="0bcc0-244">`spoke2.json` 파일에 대해 1-2단계 반복합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-244">Repeat steps 1-2 for the `spoke2.json` file.</span></span>

5. <span data-ttu-id="0bcc0-245">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="0bcc0-245">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. <span data-ttu-id="0bcc0-246">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="0bcc0-246">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="0bcc0-247">연결 테스트</span><span class="sxs-lookup"><span data-stu-id="0bcc0-247">Test connectivity</span></span>

<span data-ttu-id="0bcc0-248">시뮬레이션된 온-프레미스 환경에서 스포크 VNet으로의 연결을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-248">Test conectivity from the simulated on-premises environment to the spoke VNets.</span></span>

<span data-ttu-id="0bcc0-249">**Windows 배포**</span><span class="sxs-lookup"><span data-stu-id="0bcc0-249">**Windows deployment**</span></span>

1. <span data-ttu-id="0bcc0-250">Azure Portal을 사용하여 `onprem-jb-rg` 리소스 그룹에서 `jb-vm1`이라는 VM을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-250">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="0bcc0-251">`Connect`를 클릭하여 VM에 대한 원격 데스크톱 세션을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-251">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="0bcc0-252">`onprem.json` 매개 변수 파일에서 지정한 암호를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-252">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="0bcc0-253">VM에서 PowerShell 콘솔을 열고, `Test-NetConnection` cmdlet을 사용하여 스포크 VNet에서 jumpbox VM에 연결할 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-253">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VMs in the spoke VNets.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="0bcc0-254">**Linux 배포**</span><span class="sxs-lookup"><span data-stu-id="0bcc0-254">**Linux deployment**</span></span>

<span data-ttu-id="0bcc0-255">Linux VM을 사용하여 시뮬레이션된 온-프레미스 환경에서 스포크 VNet으로의 연결을 테스트하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-255">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="0bcc0-256">Azure Portal을 사용하여 `onprem-jb-rg` 리소스 그룹에서 `jb-vm1`이라는 VM을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-256">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="0bcc0-257">`Connect`를 클릭하고 포털에 표시되는 `ssh` 명령을 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-257">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="0bcc0-258">Linux 프롬프트에서 `ssh`를 실행하여 시뮬레이션된 온-프레미스 환경에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-258">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="0bcc0-259">`onprem.json` 매개 변수 파일에서 지정한 암호를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-259">Use the password that you specified in the `onprem.json` parameter file.</span></span>

5. <span data-ttu-id="0bcc0-260">`ping` 명령을 사용하여 각 스포크에서 jumpbox VM에 대한 연결을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-260">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke:</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="0bcc0-261">스포크 사이의 연결 추가</span><span class="sxs-lookup"><span data-stu-id="0bcc0-261">Add connectivity between spokes</span></span>

<span data-ttu-id="0bcc0-262">이 단계는 옵션입니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-262">This step is optional.</span></span> <span data-ttu-id="0bcc0-263">스포크를 서로 연결할 수 있도록 하려면 NVA(네트워크 가상 어플라이언스)를 허브 VNet의 라우터로 사용하고 다른 스포크에 연결하려고 할 때 스포크에서 라우터로 트래픽을 강제로 적용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-263">If you want to allow spokes to connect to each other, you must use a newtwork virtual appliance (NVA) as a router in the hub VNet, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="0bcc0-264">두 개의 스포크 VNet을 연결할 수 있는 UDR(사용자 정의 경로)과 함께 기본 샘플 NVA를 단일 VM으로 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-264">To deploy a basic sample NVA as a single VM, along with user-defined routes (UDRs) to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="0bcc0-265">`hub-nva.json` 파일을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-265">Open the `hub-nva.json` file.</span></span> <span data-ttu-id="0bcc0-266">`adminUsername` 및 `adminPassword`에 대한 값을 대체합니다.</span><span class="sxs-lookup"><span data-stu-id="0bcc0-266">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="0bcc0-267">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="0bcc0-267">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
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

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Azure의 허브-스포크 토폴로지"
[1]: ./images/hub-spoke-gateway-routing.svg "전이적 라우팅이 사용되는 Azure의 허브-스포크 토폴로지"
[2]: ./images/hub-spoke-no-gateway-routing.svg "NVA로 전이적 라우팅이 사용되는 Azure의 허브-스포크 토폴로지"
[3]: ./images/hub-spokehub-spoke.svg "Azure의 허브-스포크-허브-스포크 토폴로지"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
