---
title: Azure에서 보안 하이브리드 네트워크 아키텍처 구현
description: Azure에서 보안 하이브리드 네트워크 아키텍처를 구현하는 방법입니다.
author: telmosampaio
ms.date: 07/01/2018
pnp.series.title: Network DMZ
pnp.series.prev: ./index
pnp.series.next: secure-vnet-dmz
cardTitle: DMZ between Azure and on-premises
ms.openlocfilehash: 45583473ef297b2c7a5b0c4baff52485286dd051
ms.sourcegitcommit: 9b459f75254d97617e16eddd0d411d1f80b7fe90
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/03/2018
ms.locfileid: "37403167"
---
# <a name="dmz-between-azure-and-your-on-premises-datacenter"></a><span data-ttu-id="f2490-103">Azure와 온-프레미스 데이터 센터 간의 DMZ</span><span class="sxs-lookup"><span data-stu-id="f2490-103">DMZ between Azure and your on-premises datacenter</span></span>

<span data-ttu-id="f2490-104">이 참조 아키텍처는 온-프레미스 네트워크를 Azure로 확장하는 보안 하이브리드 네트워크를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure.</span></span> <span data-ttu-id="f2490-105">아키텍처는 DMZ를 구현하며 또한 온-프레미스 네트워크와 Azure VNet(Virtual Network) 간의 *경계 네트워크*라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-105">The architecture implements a DMZ, also called a *perimeter network*, between the on-premises network and an Azure virtual network (VNet).</span></span> <span data-ttu-id="f2490-106">DMZ에는 방화벽 및 패킷 검사와 같은 보안 기능을 구현하는 NVA(네트워크 가상 어플라이언스)가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-106">The DMZ includes network virtual appliances (NVAs) that implement security functionality such as firewalls and packet inspection.</span></span> <span data-ttu-id="f2490-107">VNet에서 나가는 모든 트래픽이 감사될 수 있도록 온-프레미스 네트워크를 통해 인터넷에 강제 터널링됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-107">All outgoing traffic from the VNet is force-tunneled to the Internet through the on-premises network, so that it can be audited.</span></span> [<span data-ttu-id="f2490-108">**이 솔루션을 배포합니다**.</span><span class="sxs-lookup"><span data-stu-id="f2490-108">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="f2490-109">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="f2490-109">[![0]][0]</span></span> 

<span data-ttu-id="f2490-110">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="f2490-110">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="f2490-111">이 아키텍처는 [VPN Gateway][ra-vpn] 또는 [ExpressRoute][ra-expressroute] 연결을 사용하여 온-프레미스 데이터 센터에 연결해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-111">This architecture requires a connection to your on-premises datacenter, using either a [VPN gateway][ra-vpn] or an [ExpressRoute][ra-expressroute] connection.</span></span> <span data-ttu-id="f2490-112">일반적으로 이 아키텍처는 다음과 같은 용도로 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-112">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="f2490-113">워크로드의 일부는 온-프레미스에서, 일부는 Azure에서 실행되는 하이브리드 응용 프로그램</span><span class="sxs-lookup"><span data-stu-id="f2490-113">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
* <span data-ttu-id="f2490-114">온-프레미스 데이터 센터에서 Azure VNet을 입력하는 트래픽을 통해 세부적으로 제어해야 하는 인프라</span><span class="sxs-lookup"><span data-stu-id="f2490-114">Infrastructure that requires granular control over traffic entering an Azure VNet from an on-premises datacenter.</span></span>
* <span data-ttu-id="f2490-115">나가는 트래픽을 감사해야 하는 응용 프로그램</span><span class="sxs-lookup"><span data-stu-id="f2490-115">Applications that must audit outgoing traffic.</span></span> <span data-ttu-id="f2490-116">이 항목은 여러 상용 시스템의 규제 요구 사항이며 개인 정보를 공개하지 않도록 방지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-116">This is often a regulatory requirement of many commercial systems and can help to prevent public disclosure of private information.</span></span>

## <a name="architecture"></a><span data-ttu-id="f2490-117">아키텍처</span><span class="sxs-lookup"><span data-stu-id="f2490-117">Architecture</span></span>

<span data-ttu-id="f2490-118">이 아키텍처는 다음 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-118">The architecture consists of the following components.</span></span>

* <span data-ttu-id="f2490-119">**온-프레미스 네트워크**.</span><span class="sxs-lookup"><span data-stu-id="f2490-119">**On-premises network**.</span></span> <span data-ttu-id="f2490-120">조직에서 구현되는 개인 로컬 영역 네트워크입니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-120">A  private local-area network implemented in an organization.</span></span>
* <span data-ttu-id="f2490-121">**Azure VNet(Virtual Network)**</span><span class="sxs-lookup"><span data-stu-id="f2490-121">**Azure virtual network (VNet)**.</span></span> <span data-ttu-id="f2490-122">VNet은 응용 프로그램 및 Azure에서 실행되는 다른 리소스를 호스트합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-122">The VNet hosts the application and other resources running in Azure.</span></span>
* <span data-ttu-id="f2490-123">**게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="f2490-123">**Gateway**.</span></span> <span data-ttu-id="f2490-124">게이트웨이는 온-프레미스 네트워크와 VNet의 라우터 간에 연결을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-124">The gateway provides connectivity between the routers in the on-premises network and the VNet.</span></span>
* <span data-ttu-id="f2490-125">**NVA(네트워크 가상 어플라이언스)**</span><span class="sxs-lookup"><span data-stu-id="f2490-125">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="f2490-126">NVA는 방화벽, 최적화 WAN(광역 네트워크) 작업(네트워크 압축 포함), 사용자 지정 라우팅 또는 기타 네트워크 기능으로 액세스를 허용하거나 거부하는 등의 작업을 수행하는 VM에 대해 설명하는 일반 용어입니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-126">NVA is a generic term that describes a VM performing tasks such as allowing or denying access as a firewall, optimizing wide area network (WAN) operations (including network compression), custom routing, or other network functionality.</span></span>
* <span data-ttu-id="f2490-127">**웹 계층, 비즈니스 계층 및 데이터 계층 서브넷**</span><span class="sxs-lookup"><span data-stu-id="f2490-127">**Web tier, business tier, and data tier subnets**.</span></span> <span data-ttu-id="f2490-128">클라우드에서 실행되는 3계층 응용 프로그램 예제를 구현하는 VM 및 서비스를 호스트하는 서브넷입니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-128">Subnets hosting the VMs and services that implement an example 3-tier application running in the cloud.</span></span> <span data-ttu-id="f2490-129">자세한 내용은 [Azure에서 N 계층 아키텍처에 대한 Windows VM 실행][ra-n-tier]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f2490-129">See [Running Windows VMs for an N-tier architecture on Azure][ra-n-tier] for more information.</span></span>
* <span data-ttu-id="f2490-130">**UDR(사용자 정의 경로)**</span><span class="sxs-lookup"><span data-stu-id="f2490-130">**User defined routes (UDR)**.</span></span> <span data-ttu-id="f2490-131">[사용자 정의 경로][udr-overview]는 Azure VNet 내에서 IP 트래픽 흐름을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-131">[User defined routes][udr-overview] define the flow of IP traffic within Azure VNets.</span></span>

    > [!NOTE]
    > <span data-ttu-id="f2490-132">VPN 연결의 요구 사항에 따라 온-프레미스 네트워크를 통해 트래픽을 다시 전달하는 전달 규칙을 구현하기 위해 UDR을 사용하는 대신 BGP(경계 게이트웨이 프로토콜) 경로를 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-132">Depending on the requirements of your VPN connection, you can configure Border Gateway Protocol (BGP) routes instead of using UDRs to implement the forwarding rules that direct traffic back through the on-premises network.</span></span>
    > 
    > 

* <span data-ttu-id="f2490-133">**관리 서브넷**</span><span class="sxs-lookup"><span data-stu-id="f2490-133">**Management subnet.**</span></span> <span data-ttu-id="f2490-134">이 서브넷에는 VNet에서 실행되는 구성 요소에 대한 관리 및 모니터링 기능을 구현하는 VM이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-134">This subnet contains VMs that implement management and monitoring capabilities for the components running in the VNet.</span></span>

## <a name="recommendations"></a><span data-ttu-id="f2490-135">권장 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-135">Recommendations</span></span>

<span data-ttu-id="f2490-136">대부분의 시나리오의 경우 다음 권장 사항을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-136">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="f2490-137">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-137">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="access-control-recommendations"></a><span data-ttu-id="f2490-138">액세스 제어 권장 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-138">Access control recommendations</span></span>

<span data-ttu-id="f2490-139">RBAC([역할 기반 액세스 제어][rbac])를 사용하여 응용 프로그램에서 리소스를 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-139">Use [Role-Based Access Control ][rbac] (RBAC) to manage the resources in your application.</span></span> <span data-ttu-id="f2490-140">다음 [사용자 정의 역할][rbac-custom-roles]을 만드는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-140">Consider creating the following [custom roles][rbac-custom-roles]:</span></span>

- <span data-ttu-id="f2490-141">응용 프로그램에서 인프라를 관리하는 사용 권한이 있는 DevOps 역할은 응용 프로그램 구성 요소를 배포하고 VM을 모니터링하고 다시 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-141">A DevOps role with permissions to administer the infrastructure for the application, deploy the application components, and monitor and restart VMs.</span></span>  

- <span data-ttu-id="f2490-142">네트워크 리소스를 관리하고 모니터링하는 중앙 집중식 IT 관리자 역할</span><span class="sxs-lookup"><span data-stu-id="f2490-142">A centralized IT administrator role to manage and monitor network resources.</span></span>

- <span data-ttu-id="f2490-143">NVA와 같은 보안 네트워크 리소스를 관리하는 보안 IT 관리자 역할</span><span class="sxs-lookup"><span data-stu-id="f2490-143">A security IT administrator role to manage secure network resources such as the NVAs.</span></span> 

<span data-ttu-id="f2490-144">DevOps 및 IT 관리자 역할에는 NVA 리소스에 대한 액세스 권한이 없어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-144">The DevOps and IT administrator roles should not have access to the NVA resources.</span></span> <span data-ttu-id="f2490-145">이 권한은 보안 IT 관리자 역할로 제한되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-145">This should be restricted to the security IT administrator role.</span></span>

### <a name="resource-group-recommendations"></a><span data-ttu-id="f2490-146">리소스 그룹 권장 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-146">Resource group recommendations</span></span>

<span data-ttu-id="f2490-147">VM, VNet 및 부하 분산 장치와 같은 Azure 리소스는 리소스 그룹으로 함께 그룹화하여 쉽게 관리될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-147">Azure resources such as VMs, VNets, and load balancers can be easily managed by grouping them together into resource groups.</span></span> <span data-ttu-id="f2490-148">RBAC 역할을 각 리소스 그룹을 할당하여 액세스를 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-148">Assign RBAC roles to each resource group to restrict access.</span></span>

<span data-ttu-id="f2490-149">다음과 같은 리소스 그룹을 만드는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-149">We recommend creating the following resource groups:</span></span>

* <span data-ttu-id="f2490-150">온-프레미스 네트워크에 연결하는 VNet(VM 제외), NSG 및 게이트웨이 리소스를 포함하는 리소스 그룹</span><span class="sxs-lookup"><span data-stu-id="f2490-150">A resource group containing the VNet (excluding the VMs), NSGs, and the gateway resources for connecting to the on-premises network.</span></span> <span data-ttu-id="f2490-151">이 리소스 그룹에 중앙 집중식 IT 관리자 역할을 할당합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-151">Assign the centralized IT administrator role to this resource group.</span></span>
* <span data-ttu-id="f2490-152">부하 분산 장치를 비롯한 NVA에 VM을 포함하는 리소스 그룹, jumpbox 및 기타 관리 VM 및 NVA를 통해 모든 트래픽에 강제로 적용하는 게이트웨이 서브넷의 UDR</span><span class="sxs-lookup"><span data-stu-id="f2490-152">A resource group containing the VMs for the NVAs (including the load balancer), the jumpbox and other management VMs, and the UDR for the gateway subnet that forces all traffic through the NVAs.</span></span> <span data-ttu-id="f2490-153">이 리소스 그룹에 보안 IT 관리자 역할을 할당합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-153">Assign the security IT administrator role to this resource group.</span></span>
* <span data-ttu-id="f2490-154">부하 분산 장치 및 VM을 포함하는 각 응용 프로그램 계층의 별도 리소스 그룹</span><span class="sxs-lookup"><span data-stu-id="f2490-154">Separate resource groups for each application tier that contain the load balancer and VMs.</span></span> <span data-ttu-id="f2490-155">이 리소스 그룹은 각 계층에 서브넷을 포함하지 않아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-155">Note that this resource group shouldn't include the subnets for each tier.</span></span> <span data-ttu-id="f2490-156">이 리소스 그룹에 DevOps 역할을 할당합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-156">Assign the DevOps role to this resource group.</span></span>

### <a name="virtual-network-gateway-recommendations"></a><span data-ttu-id="f2490-157">가상 네트워크 게이트웨이 권장 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-157">Virtual network gateway recommendations</span></span>

<span data-ttu-id="f2490-158">온-프레미스 트래픽은 가상 네트워크 게이트웨이를 통해 VNet에 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-158">On-premises traffic passes to the VNet through a virtual network gateway.</span></span> <span data-ttu-id="f2490-159">[Azure VPN Gateway][guidance-vpn-gateway] 또는 [Azure ExpressRoute 게이트웨이][guidance-expressroute]를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-159">We recommend an [Azure VPN gateway][guidance-vpn-gateway] or an [Azure ExpressRoute gateway][guidance-expressroute].</span></span>

### <a name="nva-recommendations"></a><span data-ttu-id="f2490-160">NVA 권장 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-160">NVA recommendations</span></span>

<span data-ttu-id="f2490-161">NVA는 네트워크 트래픽을 관리하고 모니터링하기 위한 다양한 서비스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-161">NVAs provide different services for managing and monitoring network traffic.</span></span> <span data-ttu-id="f2490-162">
  [Azure Marketplace][azure-marketplace-nva]는 사용할 수 있는 여러 타사 공급 업체 NVA를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-162">The [Azure Marketplace][azure-marketplace-nva] offers several third-party vendor NVAs that you can use.</span></span> <span data-ttu-id="f2490-163">이러한 타사 NVA 중 요구 사항을 충족하는 것이 없는 경우 VM을 사용하여 사용자 지정 NVA를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-163">If none of these third-party NVAs meet your requirements, you can create a custom NVA using VMs.</span></span> 

<span data-ttu-id="f2490-164">예를 들어 이 참조 아키텍처에 솔루션을 배포하면 VM에서 다음과 같은 기능을 사용하여 NVA를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-164">For example, the solution deployment for this reference architecture implements an NVA with the following functionality on a VM:</span></span>

* <span data-ttu-id="f2490-165">NVA NIC(네트워크 인터페이스)에서 [IP 전달][ip-forwarding]을 사용하여 트래픽을 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-165">Traffic is routed using [IP forwarding][ip-forwarding] on the NVA network interfaces (NICs).</span></span>
* <span data-ttu-id="f2490-166">이런 방법이 적절한 경우에만 NVA를 통해 트래픽을 전달할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-166">Traffic is permitted to pass through the NVA only if it is appropriate to do so.</span></span> <span data-ttu-id="f2490-167">참조 아키텍처의 각 NVA VM은 간단한 Linux 라우터입니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-167">Each NVA VM in the reference architecture is a simple Linux router.</span></span> <span data-ttu-id="f2490-168">인바운드 트래픽이 네트워크 인터페이스 *eth0*에 도착하고, 아웃바운드 트래픽이 네트워크 인터페이스*eth1*을 통해 발송된 사용자 지정 스크립트에서 정의한 규칙과 일치합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-168">Inbound traffic arrives on network interface *eth0*, and outbound traffic matches rules defined by custom scripts dispatched through network interface *eth1*.</span></span>
* <span data-ttu-id="f2490-169">NVA는 관리 서브넷에서만 구성될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-169">The NVAs can only be configured from the management subnet.</span></span> 
* <span data-ttu-id="f2490-170">관리 서브넷에 라우팅되는 트래픽은 NVA를 통해 전달되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-170">Traffic routed to the management subnet does not pass through the NVAs.</span></span> <span data-ttu-id="f2490-171">그렇지 않고 NVA에 실패하는 경우 이를 수정할 관리 서브넷에 대한 경로가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-171">Otherwise, if the NVAs fail, there would be no route to the management subnet to fix them.</span></span>  
* <span data-ttu-id="f2490-172">NVA의 VM은 부하 분산 장치 뒤의 [가용성 집합][availability-set]에 배치됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-172">The VMs for the NVA are placed in an [availability set][availability-set] behind a load balancer.</span></span> <span data-ttu-id="f2490-173">게이트웨이 서브넷의 UDR은 부하 분산 장치에 NVA 요청을 전달합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-173">The UDR in the gateway subnet directs NVA requests to the load balancer.</span></span>

<span data-ttu-id="f2490-174">NVA 수준에서 응용 프로그램 연결을 종료하고 백 엔드 계층 선호도를 유지하기 위해 계층 7 NVA를 포함시킵니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-174">Include a layer-7 NVA to terminate application connections at the NVA level and maintain affinity with the backend tiers.</span></span> <span data-ttu-id="f2490-175">이를 통해 백 엔드 계층으로부터의 응답 트래픽이 NVA를 통해 돌아오는 대칭 연결이 보장됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-175">This guarantees symmetric connectivity, in which response traffic from the backend tiers returns through the NVA.</span></span>

<span data-ttu-id="f2490-176">고려해야 할 또 다른 옵션은 특수화된 보안 작업을 수행하는 각 NVA를 사용하여 여러 NVA를 함께 연결하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-176">Another option to consider is connecting multiple NVAs in series, with each NVA performing a specialized security task.</span></span> <span data-ttu-id="f2490-177">이렇게 하면 각 보안 함수를 NVA 단위 기준으로 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-177">This allows each security function to be managed on a per-NVA basis.</span></span> <span data-ttu-id="f2490-178">예를 들어 방화벽을 구현하는 NVA는 ID 서비스를 실행하는 NVA와 함께 배치될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-178">For example, an NVA implementing a firewall could be placed in series with an NVA running identity services.</span></span> <span data-ttu-id="f2490-179">관리 편의상 단점은 대기 시간이 늘어날 수 있는 네트워크 홉이 추가된다는 것입니다. 따라서 응용 프로그램의 성능에 영향을 주지 않도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-179">The tradeoff for ease of management is the addition of extra network hops that may increase latency, so ensure that this doesn't affect your application's performance.</span></span>


### <a name="nsg-recommendations"></a><span data-ttu-id="f2490-180">NSG 권장 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-180">NSG recommendations</span></span>

<span data-ttu-id="f2490-181">VPN Gateway는 온-프레미스 네트워크에 연결하기 위한 공용 IP 주소를 노출합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-181">The VPN gateway exposes a public IP address for the connection to the on-premises network.</span></span> <span data-ttu-id="f2490-182">온-프레미스 네트워크에서 발생하지 않는 모든 트래픽을 차단하는 규칙을 사용하여 인바운드 NVA 서브넷에 NSG(네트워크 보안 그룹)를 만드는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-182">We recommend creating a network security group (NSG) for the inbound NVA subnet, with rules to block all traffic not originating from the on-premises network.</span></span>

<span data-ttu-id="f2490-183">각 서브넷의 NSG의 경우 잘못 구성되었거나 사용할 수 없는 NVA를 무시하는 인바운드 트래픽에 대해 두 번째 수준의 보호를 제공하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-183">We also recommend NSGs for each subnet to provide a second level of protection against inbound traffic bypassing an incorrectly configured or disabled NVA.</span></span> <span data-ttu-id="f2490-184">예를 들어 참조 아키텍처의 웹 계층 서브넷은 온-프레미스 네트워크(192.168.0.0/16) 또는 VNet에서 수신되지 않은 모든 요청을 무시하는 규칙 및 포트 80에서 만들지 않은 모든 요청을 무시하는 다른 규칙을 사용하여 NSG를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-184">For example, the web tier subnet in the reference architecture implements an NSG with a rule to ignore all requests other than those received from the on-premises network (192.168.0.0/16) or the VNet, and another rule that ignores all requests not made on port 80.</span></span>

### <a name="internet-access-recommendations"></a><span data-ttu-id="f2490-185">인터넷 액세스 권장 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-185">Internet access recommendations</span></span>

<span data-ttu-id="f2490-186">사이트 간 VPN 터널을 사용하는 온-프레미스 네트워크를 통한 모든 아웃바운드 인터넷 트래픽 및 NAT(네트워크 주소 변환)를 사용하는 인터넷에 대한 경로를 [강제 터널링][azure-forced-tunneling]합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-186">[Force-tunnel][azure-forced-tunneling] all outbound Internet traffic through your on-premises network using the site-to-site VPN tunnel, and route to the Internet using network address translation (NAT).</span></span> <span data-ttu-id="f2490-187">그러면 데이터 계층에 저장된 모든 기밀 정보를 실수로 누출하지 않도록 방지하고 모든 나가는 트래픽을 검사하고 감사할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-187">This prevents accidental leakage of any confidential information stored in your data tier and allows inspection and auditing of all outgoing traffic.</span></span>

> [!NOTE]
> <span data-ttu-id="f2490-188">응용 프로그램 계층의 인터넷 트래픽을 완전히 차단하지 마십시오. 차단하는 경우 이러한 계층이 VM 진단 로깅, VM 확장 및 기타 기능 다운로드 등 공용 IP 주소를 사용하는 Azure PaaS 서비스를 사용하지 않도록 방해하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-188">Don't completely block Internet traffic from the application tiers, as this will prevent these tiers from using Azure PaaS services that rely on public IP addresses, such as VM diagnostics logging, downloading of VM extensions, and other functionality.</span></span> <span data-ttu-id="f2490-189">또한 Azure 진단에서는 구성 요소가 Azure Storage 계정에 읽고 쓸 수 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-189">Azure diagnostics also requires that components can read and write to an Azure Storage account.</span></span>
> 
> 

<span data-ttu-id="f2490-190">아웃바운드 인터넷 트래픽이 올바르게 강제 터널링되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-190">Verify that outbound internet traffic is force-tunneled correctly.</span></span> <span data-ttu-id="f2490-191">온-프레미스 서버에서 [라우팅 및 원격 액세스 서비스][routing-and-remote-access-service]를 사용하여 VPN 연결을 사용하는 경우 [WireShark][wireshark] 또는 [Microsoft 메시지 분석기](https://www.microsoft.com/download/details.aspx?id=44226)와 같은 도구를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-191">If you're using a VPN connection with the [routing and remote access service][routing-and-remote-access-service] on an on-premises server, use a tool such as [WireShark][wireshark] or [Microsoft Message Analyzer](https://www.microsoft.com/download/details.aspx?id=44226).</span></span>

### <a name="management-subnet-recommendations"></a><span data-ttu-id="f2490-192">관리 서브넷 권장 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-192">Management subnet recommendations</span></span>

<span data-ttu-id="f2490-193">관리 서브넷에는 관리 및 모니터링 기능을 수행하는 jumpbox가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-193">The management subnet contains a jumpbox that performs management and monitoring functionality.</span></span> <span data-ttu-id="f2490-194">jumpbox에 대한 모든 보안 관리 작업의 실행을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-194">Restrict execution of all secure management tasks to the jumpbox.</span></span>
 
<span data-ttu-id="f2490-195">jumpbox에 대한 공용 IP 주소를 만들지 마십시오.</span><span class="sxs-lookup"><span data-stu-id="f2490-195">Do not create a public IP address for the jumpbox.</span></span> <span data-ttu-id="f2490-196">대신 들어오는 게이트웨이를 통해 jumpbox에 액세스할 수 있는 하나의 경로를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-196">Instead, create one route to access the jumpbox through the incoming gateway.</span></span> <span data-ttu-id="f2490-197">관리 서브넷이 허용되는 경로의 요청에만 응답하므로 NSG 규칙을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-197">Create NSG rules so the management subnet only responds to requests from the allowed route.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="f2490-198">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-198">Scalability considerations</span></span>

<span data-ttu-id="f2490-199">참조 아키텍처는 부하 분산 장치를 사용하여 온-프레미스 네트워크 트래픽을 NVA 장치의 풀로 전달합니다. 여기서 트래픽을 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-199">The reference architecture uses a load balancer to direct on-premises network traffic to a pool of NVA devices, which route the traffic.</span></span> <span data-ttu-id="f2490-200">NVA는 [가용성 집합][availability-set]에 배치됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-200">The NVAs are placed in an [availability set][availability-set].</span></span> <span data-ttu-id="f2490-201">이 디자인을 사용하면 시간에 따라 NVA의 처리량을 모니터링하고 부하의 증가에 대한 대응으로 NVA 장치를 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-201">This design allows you to monitor the throughput of the NVAs over time and add NVA devices in response to increases in load.</span></span>

<span data-ttu-id="f2490-202">표준 SKU VPN Gateway는 최대 100Mbps 지속 처리량을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-202">The standard SKU VPN gateway supports sustained throughput of up to 100 Mbps.</span></span> <span data-ttu-id="f2490-203">고성능 SKU는 최대 200Mbps를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-203">The High Performance SKU provides up to 200 Mbps.</span></span> <span data-ttu-id="f2490-204">높은 대역폭의 경우 ExpressRoute 게이트웨이로 업그레이드하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-204">For higher bandwidths, consider upgrading to an ExpressRoute gateway.</span></span> <span data-ttu-id="f2490-205">ExpressRoute는 VPN 연결보다 대기 시간이 낮은 최대 10Gbps 대역폭을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-205">ExpressRoute provides up to 10 Gbps bandwidth with lower latency than a VPN connection.</span></span>

<span data-ttu-id="f2490-206">Azure 게이트웨이의 확장성에 대한 자세한 내용은 [Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-scalability] 및 [Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-scalability]의 확장성 고려 사항 섹션을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f2490-206">For more information about the scalability of Azure gateways, see the scalability consideration section in [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-scalability] and [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-scalability].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="f2490-207">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-207">Availability considerations</span></span>

<span data-ttu-id="f2490-208">언급했듯이 참조 아키텍처는 부하 분산 장치 뒤의 NVA 장치 풀을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-208">As mentioned, the reference architecture uses a pool of NVA devices behind a load balancer.</span></span> <span data-ttu-id="f2490-209">부하 분산 장치는 상태 프로브를 사용하여 각 NVA를 모니터링하고 응답하지 않는 NVA를 풀에서 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-209">The load balancer uses a health probe to monitor each NVA and will remove any unresponsive NVAs from the pool.</span></span>

<span data-ttu-id="f2490-210">VNet과 온-프레미스 네트워크 간의 연결을 제공하는 데 Azure ExpressRoute를 사용하는 경우 ExpressRoute 연결을 사용할 수 없게 되면 [장애 조치를 제공하는 VPN Gateway를 구성합니다][ra-vpn-failover].</span><span class="sxs-lookup"><span data-stu-id="f2490-210">If you're using Azure ExpressRoute to provide connectivity between the VNet and on-premises network, [configure a VPN gateway to provide failover][ra-vpn-failover] if the ExpressRoute connection becomes unavailable.</span></span>

<span data-ttu-id="f2490-211">VPN 및 ExpressRoute 연결에서 가용성을 유지 관리하는 방법에 대한 자세한 내용은 [Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-availability] 및 [Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-availability]의 가용성 고려 사항을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f2490-211">For specific information on maintaining availability for VPN and ExpressRoute connections, see the availability considerations in [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-availability] and [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-availability].</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="f2490-212">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-212">Manageability considerations</span></span>

<span data-ttu-id="f2490-213">모든 응용 프로그램 및 리소스 모니터링은 관리 서브넷에 있는 jumpbox에서 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-213">All application and resource monitoring should be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="f2490-214">응용 프로그램 요구 사항에 따라 관리 서브넷에 추가 모니터링 리소스가 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-214">Depending on your application requirements, you may need additional monitoring resources in the management subnet.</span></span> <span data-ttu-id="f2490-215">그러면 이러한 리소스는 jumpbox를 통해 액세스되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-215">If so, these resources should be accessed through the jumpbox.</span></span>

<span data-ttu-id="f2490-216">온-프레미스 네트워크에서 Azure로의 게이트웨이 연결이 중단되는 경우에도 공용 IP 주소를 배포하고 점프박스에 추가한 후 인터넷으로부터 로그인하여 점프박스에 원격으로 액세스할 수 있습니다</span><span class="sxs-lookup"><span data-stu-id="f2490-216">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and remoting in from the internet.</span></span>

<span data-ttu-id="f2490-217">참조 아키텍처에서 각 계층의 서브넷은 NSG 규칙에 의해 보호됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-217">Each tier's subnet in the reference architecture is protected by NSG rules.</span></span> <span data-ttu-id="f2490-218">Windows VM에서 RDP(원격 데스크톱 프로토콜) 액세스의 경우 포트 3389 또는 Linux VM에서 SSH(Secure Shell) 액세스의 경우 포트 22를 여는 규칙을 만들어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-218">You may need to create a rule to open port 3389 for remote desktop protocol (RDP) access on Windows VMs or port 22 for secure shell (SSH) access on Linux VMs.</span></span> <span data-ttu-id="f2490-219">다른 관리 및 모니터링 도구에는 추가 포트를 여는 규칙이 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-219">Other management and monitoring tools may require rules to open additional ports.</span></span>

<span data-ttu-id="f2490-220">ExpressRoute를 사용하여 온-프레미스 데이터 센터와 Azure 간의 연결을 제공하는 경우 [AzureCT(Azure 연결 도구 키트)][azurect]를 사용하여 연결 문제를 모니터링하고 해결합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-220">If you're using ExpressRoute to provide the connectivity between your on-premises datacenter and Azure, use the [Azure Connectivity Toolkit (AzureCT)][azurect] to monitor and troubleshoot connection issues.</span></span>

<span data-ttu-id="f2490-221">VPN 및 ExpressRoute 연결을 모니터링하고 관리하는 방법에 대해 특별히 다루는 추가 정보는 [Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-manageability] 및 [Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-manageability] 문서에서 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-221">You can find additional information specifically aimed at monitoring and managing VPN and ExpressRoute connections in the articles [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-manageability] and [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-manageability].</span></span>

## <a name="security-considerations"></a><span data-ttu-id="f2490-222">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f2490-222">Security considerations</span></span>

<span data-ttu-id="f2490-223">이 참조 아키텍처는 여러 수준의 보안을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-223">This reference architecture implements multiple levels of security.</span></span>

### <a name="routing-all-on-premises-user-requests-through-the-nva"></a><span data-ttu-id="f2490-224">NVA를 통해 모든 온-프레미스 사용자 요청 라우팅</span><span class="sxs-lookup"><span data-stu-id="f2490-224">Routing all on-premises user requests through the NVA</span></span>
<span data-ttu-id="f2490-225">게이트웨이 서브넷의 UDR은 온-프레미스에서 수신되지 않은 모든 사용자 요청을 차단합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-225">The UDR in the gateway subnet blocks all user requests other than those received from on-premises.</span></span> <span data-ttu-id="f2490-226">UDR은 개인 DMZ 서브넷의 NVA에 허용된 요청을 전달하고 이러한 요청이 NVA 규칙에 의해 허용된 경우 응용 프로그램에 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-226">The UDR passes allowed requests to the NVAs in the private DMZ subnet, and these requests are passed on to the application if they are allowed by the NVA rules.</span></span> <span data-ttu-id="f2490-227">UDR에 다른 경로를 추가할 수 있지만 NVA를 실수로 우회하거나 관리 서브넷에 전달하려는 관리 트래픽을 차단하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-227">You can add other routes to the UDR, but make sure they don't inadvertently bypass the NVAs or block administrative traffic intended for the management subnet.</span></span>

<span data-ttu-id="f2490-228">NVA 앞의 부하 분산 장치는 부하 분산 규칙에서 열려 있지 않은 포트에 있는 트래픽을 무시하여 보안 장치의 역할을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-228">The load balancer in front of the NVAs also acts as a security device by ignoring traffic on ports that are not open in the load balancing rules.</span></span> <span data-ttu-id="f2490-229">참조 아키텍처의 부하 분산 장치는 포트 80의 HTTP 요청 및 포트 443의 HTTPS 요청만을 수신합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-229">The load balancers in the reference architecture only listen for HTTP requests on port 80 and HTTPS requests on port 443.</span></span> <span data-ttu-id="f2490-230">부하 분산 장치에 추가한 모든 추가 규칙을 문서화하고 보안 문제가 없는지 확인하기 위해 트래픽을 모니터링합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-230">Document any additional rules that you add to the load balancers, and monitor traffic to ensure there are no security issues.</span></span>

### <a name="using-nsgs-to-blockpass-traffic-between-application-tiers"></a><span data-ttu-id="f2490-231">NSG를 사용하여 응용 프로그램 계층 간 트래픽 차단/통과</span><span class="sxs-lookup"><span data-stu-id="f2490-231">Using NSGs to block/pass traffic between application tiers</span></span>
<span data-ttu-id="f2490-232">NSG를 사용하여 계층 간 트래픽을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-232">Traffic between tiers is restricted by using NSGs.</span></span> <span data-ttu-id="f2490-233">비즈니스 계층은 웹 계층에서 발생하지 않는 모든 트래픽을 차단하고 데이터 계층은 비즈니스 계층에서 발생하지 않는 모든 트래픽을 차단합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-233">The business tier blocks all traffic that doesn't originate in the web tier, and the data tier blocks all traffic that doesn't originate in the business tier.</span></span> <span data-ttu-id="f2490-234">이러한 계층에 대한 광범위한 액세스를 허용하기 위해 NSG 규칙을 확장해야 하는 경우 보안 위험에 비하여 이러한 요구 사항을 평가합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-234">If you have a requirement to expand the NSG rules to allow broader access to these tiers, weigh these requirements against the security risks.</span></span> <span data-ttu-id="f2490-235">새로운 인바운드 경로는 각각 우발적 또는 고의적 데이터 유출 또는 응용 프로그램 손상이 발생할 가능성을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-235">Each new inbound pathway represents an opportunity for accidental or purposeful data leakage or application damage.</span></span>

### <a name="devops-access"></a><span data-ttu-id="f2490-236">DevOps 액세스</span><span class="sxs-lookup"><span data-stu-id="f2490-236">DevOps access</span></span>
<span data-ttu-id="f2490-237">[RBAC][rbac]를 사용하여 DevOps가 각 계층에서 수행할 수 있는 작업을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-237">Use [RBAC][rbac] to restrict the operations that DevOps can perform on each tier.</span></span> <span data-ttu-id="f2490-238">사용 권한을 부여할 때 [최소 권한의 원칙][security-principle-of-least-privilege]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-238">When granting permissions, use the [principle of least privilege][security-principle-of-least-privilege].</span></span> <span data-ttu-id="f2490-239">모든 관리 작업을 기록하고 정기 감사를 수행하여 구성 변경을 계획했는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-239">Log all administrative operations and perform regular audits to ensure any configuration changes were planned.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="f2490-240">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="f2490-240">Deploy the solution</span></span>

<span data-ttu-id="f2490-241">이러한 권장 사항을 구현하는 참조 아키텍처 배포는 [GitHub][github-folder]를 통해 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-241">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="f2490-242">필수 조건</span><span class="sxs-lookup"><span data-stu-id="f2490-242">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a><span data-ttu-id="f2490-243">리소스 배포</span><span class="sxs-lookup"><span data-stu-id="f2490-243">Deploy resources</span></span>

1. <span data-ttu-id="f2490-244">참조 아키텍처 GitHub 리포지토리의 `/dmz/secure-vnet-hybrid` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-244">Navigate to the `/dmz/secure-vnet-hybrid` folder of the reference architectures GitHub repository.</span></span>

2. <span data-ttu-id="f2490-245">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="f2490-245">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. <span data-ttu-id="f2490-246">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="f2490-246">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a><span data-ttu-id="f2490-247">온-프레미스와 Azure 게이트웨이 연결</span><span class="sxs-lookup"><span data-stu-id="f2490-247">Connect the on-premises and Azure gateways</span></span>

<span data-ttu-id="f2490-248">이 단계에서는 두 개의 로컬 네트워크 게이트웨이를 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-248">In this step, you will connect the two local network gateways.</span></span>

1. <span data-ttu-id="f2490-249">Azure Portal에서 만든 리소스 그룹으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-249">In the Azure Portal, navigate to the resource group that you created.</span></span> 

2. <span data-ttu-id="f2490-250">`ra-vpn-vgw-pip`라는 리소스를 찾고 **개요** 블레이드에 표시된 IP 주소를 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-250">Find the resource named `ra-vpn-vgw-pip` and copy the IP address shown in the **Overview** blade.</span></span>

3. <span data-ttu-id="f2490-251">`onprem-vpn-lgw`라는 리소스를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-251">Find the resource named `onprem-vpn-lgw`.</span></span>

4. <span data-ttu-id="f2490-252">**구성** 블레이드를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-252">Click the **Configuration** blade.</span></span> <span data-ttu-id="f2490-253">**IP 주소** 아래에서 2단계의 IP 주소에 붙여넣습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-253">Under **IP address**, paste in the IP address from step 2.</span></span>

    ![](./images/local-net-gw.png)

5. <span data-ttu-id="f2490-254">**저장**을 클릭하고 작업이 완료되기를 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-254">Click **Save** and wait for the operation to complete.</span></span> <span data-ttu-id="f2490-255">5분 정도 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-255">It can take about 5 minutes.</span></span>

6. <span data-ttu-id="f2490-256">`onprem-vpn-gateway1-pip`라는 리소스를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-256">Find the resource named `onprem-vpn-gateway1-pip`.</span></span> <span data-ttu-id="f2490-257">**개요** 블레이드에 표시된 IP 주소를 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-257">Copy the IP address shown in the **Overview** blade.</span></span>

7. <span data-ttu-id="f2490-258">`ra-vpn-lgw`라는 리소스를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-258">Find the resource named `ra-vpn-lgw`.</span></span> 

8. <span data-ttu-id="f2490-259">**구성** 블레이드를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-259">Click the **Configuration** blade.</span></span> <span data-ttu-id="f2490-260">**IP 주소** 아래에서 6단계의 IP 주소에 붙여넣습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-260">Under **IP address**, paste in the IP address from step 6.</span></span>

9. <span data-ttu-id="f2490-261">**저장**을 클릭하고 작업이 완료되기를 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-261">Click **Save** and wait for the operation to complete.</span></span>

10. <span data-ttu-id="f2490-262">연결을 확인하려면 각 게이트웨이에 대한 **연결** 블레이드로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-262">To verify the connection, go to the **Connections** blade for each gateway.</span></span> <span data-ttu-id="f2490-263">상태가 **연결돼** 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-263">The status should be **Connected**.</span></span>

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a><span data-ttu-id="f2490-264">네트워크 트래픽이 웹 계층에 도달하는지 확인</span><span class="sxs-lookup"><span data-stu-id="f2490-264">Verify that network traffic reaches the web tier</span></span>

1. <span data-ttu-id="f2490-265">Azure Portal에서 만든 리소스 그룹으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-265">In the Azure Portal, navigate to the resource group that you created.</span></span> 

2. <span data-ttu-id="f2490-266">개인 DMZ 앞의 부하 분산 장치인 `int-dmz-lb`라는 리소스를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-266">Find the resource named `int-dmz-lb`, which is the load balancer in front of the private DMZ.</span></span> <span data-ttu-id="f2490-267">**개요** 블레이드에서 개인 IP 주소를 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-267">Copy the private IP address from the **Overview** blade.</span></span>

3. <span data-ttu-id="f2490-268">`jb-vm1`이라는 VM을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-268">Find the VM named `jb-vm1`.</span></span> <span data-ttu-id="f2490-269">**연결**을 클릭하고 원격 데스크톱을 사용하여 VM에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-269">Click **Connect** and use Remote Desktop to connect to the VM.</span></span> <span data-ttu-id="f2490-270">사용자 이름 및 암호는 onprem.json 파일에서 지정됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-270">The user name and password are specified in the onprem.json file.</span></span>

4. <span data-ttu-id="f2490-271">원격 데스크톱 세션에서 웹 브라우저를 열고 2단계의 IP 주소로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-271">From the Remote Desktop Session, open a web browser and navigate to the IP address from step 2.</span></span> <span data-ttu-id="f2490-272">기본 Apache2 서버 홈 페이지가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-272">You should see the default Apache2 server home page.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f2490-273">다음 단계</span><span class="sxs-lookup"><span data-stu-id="f2490-273">Next steps</span></span>

* <span data-ttu-id="f2490-274">[Azure와 인터넷 간의 DMZ](secure-vnet-dmz.md)를 구현하는 방법에 대해 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-274">Learn how to implement a [DMZ between Azure and the Internet](secure-vnet-dmz.md).</span></span>
* <span data-ttu-id="f2490-275">[고가용성 하이브리드 네트워크 아키텍처][ra-vpn-failover]를 구현하는 방법에 대해 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="f2490-275">Learn how to implement a [highly available hybrid network architecture][ra-vpn-failover].</span></span>
* <span data-ttu-id="f2490-276">Azure에서 네트워크 보안을 관리하는 방법에 대한 자세한 내용은 [Microsoft Cloud Services 및 네트워크 보안][cloud-services-network-security]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f2490-276">For more information about managing network security with Azure, see [Microsoft cloud services and network security][cloud-services-network-security].</span></span>
* <span data-ttu-id="f2490-277">Azure에서 리소스를 보호하는 방법에 대한 자세한 내용은 [Microsoft Azure 보안 시작][getting-started-with-azure-security]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f2490-277">For detailed information about protecting resources in Azure, see [Getting started with Microsoft Azure security][getting-started-with-azure-security].</span></span> 
* <span data-ttu-id="f2490-278">Azure 게이트웨이 연결에서 보안 문제를 해결하는 방법에 대한 자세한 내용은 [Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-vpn-gateway-security] 및 [Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-expressroute-security]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f2490-278">For additional details on addressing security concerns across an Azure gateway connection, see [Implementing a hybrid network architecture with Azure and on-premises VPN][guidance-vpn-gateway-security] and [Implementing a hybrid network architecture with Azure ExpressRoute][guidance-expressroute-security].</span></span>
  > 

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azurect]: https://github.com/Azure/NetworkMonitoring/tree/master/AzureCT
[azure-forced-tunneling]: https://azure.microsoft.com/en-gb/documentation/articles/vpn-gateway-forced-tunneling-rm/
[azure-marketplace-nva]: https://azuremarketplace.microsoft.com/marketplace/apps/category/networking
[cloud-services-network-security]: https://azure.microsoft.com/documentation/articles/best-practices-network-security/
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
[ra-expressroute]: ../hybrid-networking/expressroute.md
[ra-n-tier]: ../virtual-machines-windows/n-tier.md
[ra-vpn]: ../hybrid-networking/vpn.md
[ra-vpn-failover]: ../hybrid-networking/expressroute-vpn-failover.md
[rbac]: /azure/active-directory/role-based-access-control-configure
[rbac-custom-roles]: /azure/active-directory/role-based-access-control-custom-roles
[routing-and-remote-access-service]: https://technet.microsoft.com/library/dd469790(v=ws.11).aspx
[security-principle-of-least-privilege]: https://msdn.microsoft.com/library/hdb58b2f(v=vs.110).aspx#Anchor_1
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview
[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
[wireshark]: https://www.wireshark.org/
[0]: ./images/dmz-private.png "하이브리드 네트워크 아키텍처 보안"
