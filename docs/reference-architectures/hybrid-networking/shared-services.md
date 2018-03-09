---
title: "Azure에서 공유 서비스를 사용하여 허브-스포크 네트워크 토폴로지 구현"
description: "Azure에서 공유 서비스를 사용하여 허브-스포크 네트워크 토폴로지를 구현하는 방법입니다."
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: c0fb1d1ddd7c70ed914d58e7c73b10475b91aedf
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="11dff-103">Azure에서 공유 서비스를 사용하여 허브-스포크 네트워크 토폴로지 구현</span><span class="sxs-lookup"><span data-stu-id="11dff-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="11dff-104">이 참조 아키텍처는 모든 스포크에서 사용할 수 있는 허브의 공유 서비스를 포함하는 [hub-spoke][guidance-hub-spoke] 아키텍처에 기반합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="11dff-105">데이터 센터를 클라우드로 마이그레이션하고 [가상 데이터 센터]를 빌드하는 첫 번째 단계로 공유해야 하는 첫 번째 서비스는 ID와 보안입니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="11dff-106">이 참조 아키텍처에서는 온-프레미스 데이터 센터에서 Azure로 Active Directory 서비스를 확장하는 방법 및 허브-스포크 토폴로지에서 방화벽으로 사용할 수 있는 NVA(네트워크 가상 어플라이언스)를 추가하는 방법을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-106">This reference archiecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="11dff-107">[**이 솔루션을 배포합니다**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="11dff-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="11dff-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="11dff-108">![[0]][0]</span></span>

<span data-ttu-id="11dff-109">*이 아키텍처의 [Visio 파일][visio-download] 다운로드*</span><span class="sxs-lookup"><span data-stu-id="11dff-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="11dff-110">이 토폴로지의 이점은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-110">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="11dff-111">**비용 절감** 네트워크 가상 어플라이언스(NVA), DNS 서버와 같이 여러 워크로드에서 공유할 수 있는 서비스를 하나의 위치로 일원화하여 비용을 절감합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="11dff-112">**구독 제한 극복**: 여러 구독의 VNet을 중앙 허브로 피어링하여 구독의 제한을 극복합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="11dff-113">**문제 구분**: 중앙 IT(SecOps, InfraOps)와 워크로드(DevOps)를 문제를 분리합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="11dff-114">이 아키텍처의 일반적인 용도는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="11dff-115">개발, 테스트, 생산과 같이 서로 다른 환경에 배포되고 DNS, IDS, NTP 또는 AD DS와 같은 공유 서비스가 필요한 워크로드.</span><span class="sxs-lookup"><span data-stu-id="11dff-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="11dff-116">공유 서비스는 허브 VNet에 배치되고 각 환경은 스포크에 배포되어 격리 상태 유지.</span><span class="sxs-lookup"><span data-stu-id="11dff-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="11dff-117">서로 연결할 필요가 없으나 공유 서비스에 대한 액세스가 필요한 워크로드.</span><span class="sxs-lookup"><span data-stu-id="11dff-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="11dff-118">DMZ로 기능한 허브의 방화벽, 각 스포크에서 워크로드에 대한 별도의 관리 등 보안 측면에 대한 중앙 제어가 필요한 엔터프라이즈.</span><span class="sxs-lookup"><span data-stu-id="11dff-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="11dff-119">건축</span><span class="sxs-lookup"><span data-stu-id="11dff-119">Architecture</span></span>

<span data-ttu-id="11dff-120">이 아키텍처는 다음 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="11dff-121">**온-프레미스 네트워크**.</span><span class="sxs-lookup"><span data-stu-id="11dff-121">**On-premises network**.</span></span> <span data-ttu-id="11dff-122">조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="11dff-123">**VPN 장치**.</span><span class="sxs-lookup"><span data-stu-id="11dff-123">**VPN device**.</span></span> <span data-ttu-id="11dff-124">온-프레미스 네트워크에 외부 연결을 제공하는 장치 또는 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="11dff-125">VPN 장치는 하드웨어 장치일 수도 있고 Windows Server 2012의 RRAS(라우팅 및 원격 액세스 서비스)와 같은 소프트웨어 솔루션일 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="11dff-126">지원되는 VPN 어플라이언스 목록 및 선택한 VPN 어플라이언스를 Azure에 연결하도록 구성하는 방법에 대한 자세한 내용은 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치 정보][vpn-appliance]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="11dff-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="11dff-127">**VPN 가상 네트워크 게이트웨이 또는 ExpressRoute 게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="11dff-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="11dff-128">가상 네트워크 게이트웨이를 사용하면 VNet을 온-프레미스 네트워크에 연결하는 데 사용되는 VPN 장치 또는 ExpressRoute 회로에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="11dff-129">자세한 내용은 [온-프레미스 네트워크를 Microsoft Azure virtual network에 연결][connect-to-an-Azure-vnet]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="11dff-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="11dff-130">이 참조 아키텍처의 배포 스크립트는 연결을 위해 VPN 게이트웨이를 사용하고, 사용자의 온-프레미스 네트워크를 시뮬레이션하기 위해 Azure의 VNet을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="11dff-131">**허브 VNet**.</span><span class="sxs-lookup"><span data-stu-id="11dff-131">**Hub VNet**.</span></span> <span data-ttu-id="11dff-132">허브-스포크 토폴로지에서 허브로 사용되는 Azure VNet입니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="11dff-133">허브는 사용자의 온-프레미스 네트워크에 대한 연결의 중앙 위치이자 스포크 VNet에 호스팅된 다양한 워크로드에 의해 사용되는 서비스가 호스팅되는 장소입니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="11dff-134">**게이트웨이 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="11dff-134">**Gateway subnet**.</span></span> <span data-ttu-id="11dff-135">가상 네트워크 게이트웨이는 동일한 서브넷에 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="11dff-136">**공유 서비스 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="11dff-136">**Shared services subnet**.</span></span> <span data-ttu-id="11dff-137">DNS, AD DS를 비롯한 모든 스포크에서 공유될 수 있는 서비스를 호스팅하는 데 사용되는 허브 VNet의 서브넷입니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="11dff-138">**DMZ 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="11dff-138">**DMZ subnet**.</span></span> <span data-ttu-id="11dff-139">방화벽과 같은 보안 어플라이언스로 사용할 수 있는 NVA를 호스트하는 데 사용되는 허브 VNet의 서브넷입니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="11dff-140">**스포크 VNet**.</span><span class="sxs-lookup"><span data-stu-id="11dff-140">**Spoke VNets**.</span></span> <span data-ttu-id="11dff-141">허브-스포크 토폴로지에서 스포크로 사용되는 하나 이상의 Azure VNet입니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="11dff-142">스포크는 다른 스포크와 별도로 관리되는 자체 VNet에서 워크로드를 격리하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="11dff-143">각 워크로드에는 Azure Load Balancer를 통해 여러 서브넷이 연결된 여러 계층이 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="11dff-144">응용 프로그램 인프라에 대한 자세한 내용은 [Windows VM 워크로드 실행][windows-vm-ra] 및 [Linux VM 워크로드 실행][linux-vm-ra]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="11dff-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="11dff-145">**VNet 피어링**.</span><span class="sxs-lookup"><span data-stu-id="11dff-145">**VNet peering**.</span></span> <span data-ttu-id="11dff-146">동일한 Azure 지역에 있는 2개의 VNet을 [피어링 연결][vnet-peering]을 사용하여 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="11dff-147">피어링 연결은 VNet 사이에 적용되는 비전이적이고 대기 시간이 낮은 연결입니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="11dff-148">피어링이 적용되면 VNet은 라우터가 없어도 Azure 백본을 사용하여 트래픽을 교환합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="11dff-149">허브-스포크 네트워크 토폴로지에서는 VNet 피어링을 사용하여 허브를 각 스포크에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="11dff-150">이 문서에서는 [리소스 관리자](/azure/azure-resource-manager/resource-group-overview) 배포만 다루고 있지만, 동일한 구독에서 클래식 VNet을 리소스 관리자에 연결할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="11dff-151">이렇게 하면 스포크가 클래식 배포를 호스팅하면서도 허브에서 공유되는 서비스를 이용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="11dff-152">권장 사항</span><span class="sxs-lookup"><span data-stu-id="11dff-152">Recommendations</span></span>

<span data-ttu-id="11dff-153">[hub-spoke][guidance-hub-spoke] 참조 아키텍처에 대한 모든 권장 사항은 공유 서비스 참조 아키텍처에도 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="11dff-154">또한 공유 서비스에서 대부분의 시나리오에 다음 권장 사항이 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-154">ALso, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="11dff-155">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="11dff-156">ID</span><span class="sxs-lookup"><span data-stu-id="11dff-156">Identity</span></span>

<span data-ttu-id="11dff-157">대부분의 엔터프라이즈 조직은 온-프레미스 데이터 센터에 ADDS(Active Directory 디렉터리 서비스) 환경을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="11dff-158">ADDS를 사용하는 온-프레미스 네트워크에서 Azure로 이동된 자산 관리를 용이하게 하려면 Azure에서 ADDS 도메인 컨트롤러를 호스트하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="11dff-159">Azure 및 온-프레미스 환경에서 개별적으로 제어하도록 그룹 정책 개체를 사용하는 경우 각 Azure 지역에 다른 AD 사이트를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="11dff-160">종속 워크로드가 액세스할 수 있는 중앙 VNet(허브)에 도메인 컨트롤러를 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="11dff-161">보안</span><span class="sxs-lookup"><span data-stu-id="11dff-161">Security</span></span>

<span data-ttu-id="11dff-162">온-프레미스 환경에서 Azure로 워크로드를 이동할 때 이러한 일부 워크로드는 VM에서 호스트되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="11dff-163">규정 준수상 해당 워크로드를 트래버스하는 트래픽에 대한 제한 사항을 적용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="11dff-164">Azure에서 NVA(네트워크 가상 어플라이언스)를 사용하여 다른 형식의 보안 및 성능 서비스를 호스트할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="11dff-165">지정된 집합의 어플라이언스 온-프레미스에 익숙하다면 해당되는 경우 Azure에서 동일한 가상 어플라이언스를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="11dff-166">이 참조 아키텍처의 배포 스크립트는 네트워크 가상 어플라이언스를 모방하기 위해 사용하는 IP를 전달하여 Ubuntu VM을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enbaled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="11dff-167">고려 사항</span><span class="sxs-lookup"><span data-stu-id="11dff-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="11dff-168">VNet 피어링 제한 문제 해결</span><span class="sxs-lookup"><span data-stu-id="11dff-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="11dff-169">반드시 Azure의 [VNet 하나당 허용되는 VNet 피어링 개수 제한][vnet-peering-limit]을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="11dff-170">허용되는 한도보다 많은 스포크가 필요한 경우 첫 번째 수준의 스포크가 허브로서 기능하는 허브-스포크-허브-스포크 토폴로지를 만드는 방법을 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="11dff-171">다음 다이어그램은 이 토폴로지를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="11dff-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="11dff-172">![[3]][3]</span></span>

<span data-ttu-id="11dff-173">또한, 허브가 다수의 스포크에 맞게 확장될 수 있도록 허브에서 어떤 서비스가 공유되는지도 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="11dff-174">예를 들어 허브에서 방화벽 서비스를 제공한다면 복수의 스포크를 추가할 때 방화벽 솔루션의 대역폭 제한을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="11dff-175">공유 서비스 중 일부를 두 번째 수준의 허브로 이동하는 것이 좋을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="11dff-176">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="11dff-176">Deploy the solution</span></span>

<span data-ttu-id="11dff-177">이 아키텍처에 대한 배포는 [GitHub][ref-arch-repo]에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="11dff-178">이 아키텍처에 대한 배포는 연결을 테스트하기 위해 각 VNet에서 Ubuntu VM을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-178">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="11dff-179">**허브 VNet**의 **shared-services** 서브넷에 실제로 호스팅된 서비스는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-179">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="11dff-180">필수 조건</span><span class="sxs-lookup"><span data-stu-id="11dff-180">Prerequisites</span></span>

<span data-ttu-id="11dff-181">사용자의 구독에 참조 아키텍처를 배포하려면 먼저 다음 단계를 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-181">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="11dff-182">[AzureCAT 참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-182">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="11dff-183">Azure CLI 2.0이 컴퓨터에 설치되어 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-183">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="11dff-184">CLI 설치 지침은 [Install Azure CLI 2.0][azure-cli-2](Azure CLI 2.0 설치)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="11dff-184">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="11dff-185">[Azure 빌딩 블록][azbb] npm 패키지를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-185">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="11dff-186">명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 아래 명령을 사용하여 Azure 계정에 로그인하고, 프롬프트에 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-186">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="11dff-187">azbb를 사용하여 시뮬레이션된 온-프레미스 데이터 센터 배포</span><span class="sxs-lookup"><span data-stu-id="11dff-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="11dff-188">시뮬레이션된 온-프레미스 데이터 센터를 Azure VNet으로서 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-188">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="11dff-189">위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\shared-services-stack\` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-189">Navigate to the `hybrid-networking\shared-services-stack\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="11dff-190">`onprem.json` 파일을 열고 아래에 나와 있는 대로 45열과 46열의 큰따옴표 사이에 사용자 이름과 암호를 입력한 다음, 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-190">Open the `onprem.json` file and enter a username and password between the quotes in line 45 and 46, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="11dff-191">`azbb`를 실행하여 아래와 같이 시뮬레이션된 온-프레미스 환경을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-191">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="11dff-192">`onprem-vnet-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-192">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="11dff-193">배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-193">Wait for the deployment to finish.</span></span> <span data-ttu-id="11dff-194">이 배포는 가상 네트워크, Windows를 실행하는 가상 머신 및 VPN 게이트웨이를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-194">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="11dff-195">VPN 게이트웨이의 생성이 완료되기까지 40분 이상이 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-195">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="11dff-196">Azure 허브 VNet</span><span class="sxs-lookup"><span data-stu-id="11dff-196">Azure hub VNet</span></span>

<span data-ttu-id="11dff-197">허브 VNet을 배포하고 위에서 만든 시뮬레이션된 온-프레미스 VNet에 연결하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-197">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="11dff-198">`hub-vnet.json` 파일을 열고 아래에 나와 있는 대로 50열과 51열의 큰따옴표 사이에 사용자 이름과 암호를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-198">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 50 and 51, as shown below.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="11dff-199">`osType`의 52열에서 `Windows` 또는 `Linux`를 입력하여 jumpbox의 운영 체제로 Windows Server 2016 데이터 센터 또는 Ubuntu 16.04를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-199">On line 52, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="11dff-200">아래에 나와 있는 대로 83열의 큰따옴표 사이에 공유 키를 입력한 다음, 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-200">Enter a shared key between the quotes in line 83, as shown below, then save the file.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="11dff-201">`azbb`를 실행하여 아래와 같이 시뮬레이션된 온-프레미스 환경을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-201">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="11dff-202">`hub-vnet-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-202">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="11dff-203">배포가 완료될 때까지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-203">Wait for the deployment to finish.</span></span> <span data-ttu-id="11dff-204">이 배포는 가상 네트워크, 가상 머신, VPN 게이트웨이 및 이전 섹션에서 생성한 게이트웨이에 대한 연결을 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-204">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="11dff-205">VPN 게이트웨이의 생성이 완료되기까지 40분 이상이 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-205">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="adds-in-azure"></a><span data-ttu-id="11dff-206">Azure의 ADDS</span><span class="sxs-lookup"><span data-stu-id="11dff-206">ADDS in Azure</span></span>

<span data-ttu-id="11dff-207">Azure에서 ADDS 도메인 컨트롤러를 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-207">To deploy the ADDS domain controllers in Azure, perform the following steps.</span></span>

1. <span data-ttu-id="11dff-208">`hub-adds.json` 파일을 열고 아래에 나와 있는 대로 14열과 15열의 큰따옴표 사이에 사용자 이름과 암호를 입력한 다음, 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-208">Open the `hub-adds.json` file and enter a username and password between the quotes in lines 14 and 15, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="11dff-209">아래와 같이 `azbb`를 실행하여 ADDS 도메인 컨트롤러를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-209">Run `azbb` to deploy the ADDS domain controllers as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="11dff-210">`hub-adds-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-210">If you decide to use a different resource group name (other than `hub-adds-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

  > [!NOTE]
  > <span data-ttu-id="11dff-211">두 개의 VM을 시뮬레이션된 온-프레미스 데이터 센터의 호스트 도메인에 조인한 다음, AD DS를 설치해야 하므로 배포에서 이 파트는 몇 분 정도 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-211">This part of the deployment may take several minutes, since it requires joining the two VMs to the domain hosted int he simulated on-premises datacenter, then installing AD DS on them.</span></span>

### <a name="nva"></a><span data-ttu-id="11dff-212">NVA</span><span class="sxs-lookup"><span data-stu-id="11dff-212">NVA</span></span>

<span data-ttu-id="11dff-213">`dmz` 서브넷에서 NVA를 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-213">To deploy an NVA in the `dmz` subnet, perform the following steps:</span></span>

1. <span data-ttu-id="11dff-214">`hub-nva.json` 파일을 열고 아래에 나와 있는 대로 13열과 14열의 큰따옴표 사이에 사용자 이름과 암호를 입력한 다음, 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-214">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. <span data-ttu-id="11dff-215">`azbb`를 실행하여 NVA VM 및 사용자 정의 경로를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-215">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="11dff-216">`hub-nva-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-216">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="11dff-217">Azure 스포크 VNet</span><span class="sxs-lookup"><span data-stu-id="11dff-217">Azure spoke VNets</span></span>

<span data-ttu-id="11dff-218">스포크 VNet을 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-218">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="11dff-219">`spoke1.json` 파일을 열고 아래에 나와 있는 대로 52열과 53열의 큰따옴표 사이에 사용자 이름과 암호를 입력한 다음, 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-219">Open the `spoke1.json` file and enter a username and password between the quotes in lines 52 and 53, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="11dff-220">`osType`의 54열에서 `Windows` 또는 `Linux`를 입력하여 jumpbox의 운영 체제로 Windows Server 2016 데이터 센터 또는 Ubuntu 16.04를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-220">On line 54, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="11dff-221">`azbb`를 실행하여 아래와 같이 첫 번째 스포크 VNet 환경을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-221">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="11dff-222">`spoke1-vnet-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-222">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

3. <span data-ttu-id="11dff-223">`spoke2.json` 파일에서 위의 1단계를 반복합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-223">Repeat step 1 above for file `spoke2.json`.</span></span>

4. <span data-ttu-id="11dff-224">`azbb`를 실행하여 아래와 같이 두 번째 스포크 VNet 환경을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-224">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="11dff-225">`spoke2-vnet-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-225">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="11dff-226">스포크 VNet에 대한 Azure 허브 VNet 피어링</span><span class="sxs-lookup"><span data-stu-id="11dff-226">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="11dff-227">허브 VNet에서 스포크 VNet으로의 피어링을 연결하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-227">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="11dff-228">`hub-vnet-peering.json` 파일을 열고, 29열에서 리소스 그룹 이름 및 각 가상 네트워크 피어링의 가상 네트워크 이름이 올바른지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-228">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="11dff-229">`azbb`를 실행하여 아래와 같이 첫 번째 스포크 VNet 환경을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-229">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > <span data-ttu-id="11dff-230">`hub-vnet-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="11dff-230">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[가상 데이터 센터]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "Azure의 공유 서비스 토폴로지"
[3]: ./images/hub-spokehub-spoke.svg "Azure의 허브-스포크-허브-스포크 토폴로지"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
