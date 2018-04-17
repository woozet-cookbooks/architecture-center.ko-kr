---
title: N 계층 아키텍처에 대한 Windows VM 실행
description: 가용성, 보안, 확장성 및 관리 보안에 주의를 기울이며 Azure에서 다중 계층 아키텍처를 구현하는 방법입니다.
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 5ed94eb9ab8203d35d9597336e367d54e03944d7
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="run-windows-vms-for-an-n-tier-application"></a><span data-ttu-id="f0387-103">N 계층 응용 프로그램에 대한 Windows VM 실행</span><span class="sxs-lookup"><span data-stu-id="f0387-103">Run Windows VMs for an N-tier application</span></span>

<span data-ttu-id="f0387-104">이 참조 아키텍처는 N 계층 응용 프로그램에 대해 Windows VM(가상 머신)을 실행하는 데 관해 검증된 일련의 사례를 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-104">This reference architecture shows a set of proven practices for running Windows virtual machines (VMs) for an N-tier application.</span></span> [<span data-ttu-id="f0387-105">**이 솔루션을 배포합니다**.</span><span class="sxs-lookup"><span data-stu-id="f0387-105">**Deploy this solution**.</span></span>](#deploy-the-solution) 

<span data-ttu-id="f0387-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="f0387-106">![[0]][0]</span></span>

<span data-ttu-id="f0387-107">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="f0387-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="f0387-108">건축</span><span class="sxs-lookup"><span data-stu-id="f0387-108">Architecture</span></span> 

<span data-ttu-id="f0387-109">N 계층 아키텍처를 구현하는 방법은 여러 가지가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-109">There are many ways to implement an N-tier architecture.</span></span> <span data-ttu-id="f0387-110">이 다이어그램에서는 일반적인 3계층 웹 응용 프로그램을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-110">The diagram shows a typical 3-tier web application.</span></span> <span data-ttu-id="f0387-111">이 아키텍처는 [부하가 분산된 VM을 실행하여 확장성 및 가용성 확보][multi-vm]를 기반으로 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-111">This architecture builds on [Run load-balanced VMs for scalability and availability][multi-vm].</span></span> <span data-ttu-id="f0387-112">웹 및 비즈니스 계층은 부하 분산된 VM을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-112">The web and business tiers use load-balanced VMs.</span></span>

* <span data-ttu-id="f0387-113">**가용성 집합.**</span><span class="sxs-lookup"><span data-stu-id="f0387-113">**Availability sets.**</span></span> <span data-ttu-id="f0387-114">각 계층에 대해 [가용성 집합][azure-availability-sets]을 만들고 각 계층에서 두 개 이상의 VM을 프로비전합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-114">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span> <span data-ttu-id="f0387-115">이렇게 하면 VM이 더 높은 [SLA(서비스 수준 약정)][vm-sla]를 충족할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-115">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span> <span data-ttu-id="f0387-116">단일 VM을 가용성 집합에서 배포할 수는 있지만, 단일 VM이 모든 OS 및 데이터 디스크에 대한 Azure Premium Storage를 사용하지 않는 한 단일 VM은 SLA를 보장하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-116">You can deploy a single VM in an availability set, but the single VM will not qualify for an SLA guarantee unless the single VM is using Azure Premium Storage for all OS and data disks.</span></span>  
* <span data-ttu-id="f0387-117">**서브넷.**</span><span class="sxs-lookup"><span data-stu-id="f0387-117">**Subnets.**</span></span> <span data-ttu-id="f0387-118">각 계층에 대해 별도의 서브넷을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-118">Create a separate subnet for each tier.</span></span> <span data-ttu-id="f0387-119">[CIDR] 표기법을 사용하여 주소 범위 및 서브넷 마스크를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-119">Specify the address range and subnet mask using [CIDR] notation.</span></span> 
* <span data-ttu-id="f0387-120">**부하 분산 장치.**</span><span class="sxs-lookup"><span data-stu-id="f0387-120">**Load balancers.**</span></span> <span data-ttu-id="f0387-121">[인터넷 연결 부하 분산 장치][load-balancer-external]를 사용하여 들어오는 인터넷 트래픽을 웹 계층에 분산하고, [내부 부하 분산 장치][load-balancer-internal]를 사용하여 네트워크 트래픽을 웹 계층에서 비즈니스 계층으로 분산합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-121">Use an [Internet-facing load balancer][load-balancer-external] to distribute incoming Internet traffic to the web tier, and an [internal load balancer][load-balancer-internal] to distribute network traffic from the web tier to the business tier.</span></span>
* <span data-ttu-id="f0387-122">**Jumpbox.**</span><span class="sxs-lookup"><span data-stu-id="f0387-122">**Jumpbox.**</span></span> <span data-ttu-id="f0387-123">[요새 호스트]라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-123">Also called a [bastion host].</span></span> <span data-ttu-id="f0387-124">관리자가 다른 VM에 연결할 때 사용하는 네트워크의 보안 VM입니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-124">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="f0387-125">Jumpbox는 안전 목록에 있는 공용 IP 주소의 원격 트래픽만 허용하는 NSG를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-125">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="f0387-126">NSG는 RDP(원격 데스크톱) 트래픽을 허용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-126">The NSG should permit remote desktop (RDP) traffic.</span></span>
* <span data-ttu-id="f0387-127">**모니터링.**</span><span class="sxs-lookup"><span data-stu-id="f0387-127">**Monitoring.**</span></span> <span data-ttu-id="f0387-128">[Nagios], [Zabbix] 또는 [Icinga]와 같은 모니터링 소프트웨어는 응답 시간, VM 가동 시간 및 시스템의 전반적인 상태에 대한 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-128">Monitoring software such as [Nagios], [Zabbix], or [Icinga] can give you insight into response time, VM uptime, and the overall health of your system.</span></span> <span data-ttu-id="f0387-129">개별 관리 서브넷에 배치되어 있는 VM에 모니터링 소프트웨어를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-129">Install the monitoring software on a VM that's placed in a separate management subnet.</span></span>
* <span data-ttu-id="f0387-130"><strong>NSG.</strong></span><span class="sxs-lookup"><span data-stu-id="f0387-130"><strong>NSGs.</strong></span></span> <span data-ttu-id="f0387-131">[NSG(네트워크 보안 그룹)][nsg]을 사용하여 VNet 내 네트워크 트래픽을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-131">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="f0387-132">예를 들어 여기에 표시된 3계층 아키텍처에서 데이터베이스 계층은 비즈니스 계층 및 관리 서브넷뿐 아니라 웹 프론트 엔드의 트래픽을 허용하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-132">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>
* <span data-ttu-id="f0387-133">**SQL Server Always On 가용성 그룹.**</span><span class="sxs-lookup"><span data-stu-id="f0387-133">**SQL Server Always On Availability Group.**</span></span> <span data-ttu-id="f0387-134">복제 및 장애 조치(failover)를 사용하여 데이터 계층에서 높은 가용성을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-134">Provides high availability at the data tier, by enabling replication and failover.</span></span>
* <span data-ttu-id="f0387-135">**AD DS(Active Directory Domain Services) 서버**.</span><span class="sxs-lookup"><span data-stu-id="f0387-135">**Active Directory Domain Services (AD DS) Servers**.</span></span> <span data-ttu-id="f0387-136">Windows Server 2016 전 버전에서는 SQL Server Always On 가용성 그룹이 도메인에 연결되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-136">Prior to Windows Server 2016, SQL Server Always On Availability Groups must be joined to a domain.</span></span> <span data-ttu-id="f0387-137">가용성 그룹이 WSFC(Windows Server 장애 조치 클러스터) 기술을 사용하기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-137">This is because Availability Groups depend on Windows Server Failover Cluster (WSFC) technology.</span></span> <span data-ttu-id="f0387-138">Windows Server 2016부터는 Active Directory 없이도 장애 조치(failover) 클러스터를 만들 수 있는 기능이 추가되었기 때문에 이 아키텍처에 AD DS 서버가 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-138">Windows Server 2016 introduces the ability to create a Failover Cluster without Active Directory, in which case the AD DS servers are not required for this architecture.</span></span> <span data-ttu-id="f0387-139">자세한 내용은 [Windows Server 2016 장애 조치(failover) 클러스터링의 새로운 기능][wsfc-whats-new]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-139">For more information, see [What's new in Failover Clustering in Windows Server 2016][wsfc-whats-new].</span></span>
* <span data-ttu-id="f0387-140">**Azure DNS**.</span><span class="sxs-lookup"><span data-stu-id="f0387-140">**Azure DNS**.</span></span> <span data-ttu-id="f0387-141">[Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-141">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="f0387-142">Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-142">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

## <a name="recommendations"></a><span data-ttu-id="f0387-143">권장 사항</span><span class="sxs-lookup"><span data-stu-id="f0387-143">Recommendations</span></span>

<span data-ttu-id="f0387-144">개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-144">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="f0387-145">여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-145">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="f0387-146">VNet/서브넷</span><span class="sxs-lookup"><span data-stu-id="f0387-146">VNet / Subnets</span></span>

<span data-ttu-id="f0387-147">VNet을 만들 때는 각 서브넷에 포함된 리소스에 몇 개의 IP 주소가 필요한지 결정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-147">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="f0387-148">[CIDR] 표기법을 사용하여 필요한 IP 주소를 충족하는 서브넷 마스크와 VNet 주소 범위를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-148">Specify a subnet mask and a VNet address range large enough for the required IP addresses, using [CIDR] notation.</span></span> <span data-ttu-id="f0387-149">표준 [사설 IP 주소 블록][private-ip-space](10.0.0.0/8, 172.16.0.0/12 및 192.168.0.0/16)에 해당하는 주소 공간을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-149">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="f0387-150">추후 VNet과 온-프레미스 네트워크 사이에 게이트웨이를 설정해야 할 경우에 대비하여 온-프레미스 네트워크와 중복되지 않는 주소 범위를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-150">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premise network later.</span></span> <span data-ttu-id="f0387-151">VNet을 만든 뒤에는 주소 범위를 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-151">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="f0387-152">기능 및 보안 요구 사항을 염두에 두고 서브넷을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-152">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="f0387-153">동일한 계층이나 역할에 속한 모든 VM은 동일한 서브넷에 속해야 합니다. 이때 서브넷은 보안 경계가 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-153">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="f0387-154">VNet 및 서브넷 디자인에 대한 자세한 내용은 [Azure Virtual Networks 계획 및 디자인][plan-network]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-154">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

<span data-ttu-id="f0387-155">각 서브넷에 대해 CIDR 표기법을 사용하여 서브넷의 주소 공간을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-155">For each subnet, specify the address space for the subnet in CIDR notation.</span></span> <span data-ttu-id="f0387-156">예를 들어 ‘10.0.0.0/24’는 256개의 IP 주소 범위를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-156">For example, '10.0.0.0/24' creates a range of 256 IP addresses.</span></span> <span data-ttu-id="f0387-157">VM은 이 중에서 251개를 사용할 수 있습니다. 나머지 5개는 예약되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-157">VMs can use 251 of these; five are reserved.</span></span> <span data-ttu-id="f0387-158">각 서브넷의 주소 공간이 겹치지 않아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-158">Make sure the address ranges don't overlap across subnets.</span></span> <span data-ttu-id="f0387-159">[Virtual Network FAQ][vnet faq]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-159">See the [Virtual Network FAQ][vnet faq].</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="f0387-160">네트워크 보안 그룹</span><span class="sxs-lookup"><span data-stu-id="f0387-160">Network security groups</span></span>

<span data-ttu-id="f0387-161">NSG 규칙을 사용하여 계층 사이의 트래픽을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-161">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="f0387-162">예를 들어 위에 표시된 3 계층 아키텍처에서 웹 계층은 데이터베이스 계층과 직접 통신하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-162">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="f0387-163">이를 위해서는 데이터베이스 계층에서 웹 계층 서브넷으로부터 수신되는 트래픽을 차단해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-163">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="f0387-164">NSG를 만든 다음 이를 데이터베이스 계층 서브넷에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-164">Create an NSG and associate it to the database tier subnet.</span></span>
2. <span data-ttu-id="f0387-165">VNet으로부터 수신되는 모든 트래픽을 차단하는 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-165">Add a rule that denies all inbound traffic from the VNet.</span></span> <span data-ttu-id="f0387-166">(규칙에 `VIRTUAL_NETWORK` 태그를 사용합니다.)</span><span class="sxs-lookup"><span data-stu-id="f0387-166">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
3. <span data-ttu-id="f0387-167">비즈니스 계층 서브넷으로부터 수신되는 모든 트래픽을 허용하는 규칙을 높은 우선 순위로 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-167">Add a rule with a higher priority that allows inbound traffic from the business tier subnet.</span></span> <span data-ttu-id="f0387-168">이 규칙은 이전 규칙을 재정의하며, 비즈니스 계층이 데이터베이스 계층과 통신할 수 있도록 해 줍니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-168">This rule overrides the previous rule, and allows the business tier to talk to the database tier.</span></span>
4. <span data-ttu-id="f0387-169">데이터베이스 계층 서브넷 자체로부터 수신되는 트래픽을 허용하는 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-169">Add a rule that allows inbound traffic from within the database tier subnet itself.</span></span> <span data-ttu-id="f0387-170">이 규칙은 데이터베이스 계층에 속한 VM 사이의 통신을 허용합니다. 이것은 데이터베이스 복제와 장애 조치(failover)를 위해 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-170">This rule allows communication between VMs in the database tier, which is needed for database replication and failover.</span></span>
5. <span data-ttu-id="f0387-171">jumpbox 서브넷으로부터 수신되는 RDP 트래픽을 허용하는 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-171">Add a rule that allows RDP traffic from the jumpbox subnet.</span></span> <span data-ttu-id="f0387-172">이 규칙은 관리자가 jumpbox에서 데이터베이스 계층에 연결할 수 있도록 해 줍니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-172">This rule lets administrators connect to the database tier from the jumpbox.</span></span>
   
   > [!NOTE]
   > <span data-ttu-id="f0387-173">NSG는 VNet 내부로부터 수신되는 모든 트래픽을 허용하는 기본 규칙을 갖습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-173">An NSG has default rules that allow any inbound traffic from within the VNet.</span></span> <span data-ttu-id="f0387-174">이러한 규칙은 삭제할 수 없지만, 우선 순위가 더 높은 규칙을 만들면 재정의할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-174">These rules can't be deleted, but you can override them by creating higher priority rules.</span></span>
   > 
   > 

### <a name="load-balancers"></a><span data-ttu-id="f0387-175">부하 분산 장치</span><span class="sxs-lookup"><span data-stu-id="f0387-175">Load balancers</span></span>

<span data-ttu-id="f0387-176">외부 부하 분산 장치는 인터넷 트래픽을 웹 계층으로 분산합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-176">The external load balancer distributes Internet traffic to the web tier.</span></span> <span data-ttu-id="f0387-177">이 부하 분산 장치에 사용할 공용 IP 주소를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-177">Create a public IP address for this load balancer.</span></span> <span data-ttu-id="f0387-178">[인터넷 연결 부하 분산 장치 만들기][lb-external-create]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-178">See [Creating an Internet-facing load balancer][lb-external-create].</span></span>

<span data-ttu-id="f0387-179">내부 부하 분산 장치는 웹 계층에서 비즈니스 계층으로 네트워크 트래픽을 분산합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-179">The internal load balancer distributes network traffic from the web tier to the business tier.</span></span> <span data-ttu-id="f0387-180">이 부하 분산 장치에 사설 IP 주소를 부여하려면 프론트 엔드 IP 구성을 만든 다음 이를 비즈니스 계층의 서브넷에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-180">To give this load balancer a private IP address, create a frontend IP configuration and associate it with the subnet for the business tier.</span></span> <span data-ttu-id="f0387-181">[내부 부하 분산 장치 만들기 시작][lb-internal-create]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-181">See [Get started creating an Internal load balancer][lb-internal-create].</span></span>

### <a name="sql-server-always-on-availability-groups"></a><span data-ttu-id="f0387-182">SQL Server Always On 가용성 그룹</span><span class="sxs-lookup"><span data-stu-id="f0387-182">SQL Server Always On Availability Groups</span></span>

<span data-ttu-id="f0387-183">SQL Server의 고가용성을 위해 [Always On 가용성 그룹][sql-alwayson]을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-183">We recommend [Always On Availability Groups][sql-alwayson] for SQL Server high availability.</span></span> <span data-ttu-id="f0387-184">Windows Server 2016 전 버전에서는 Always On 가용성 그룹에 도메인 컨트롤러가 필요하며, 가용성 그룹의 모든 노드가 동일한 AD 도메인에 속해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-184">Prior to Windows Server 2016, Always On Availability Groups require a domain controller, and all nodes in the availability group must be in the same AD domain.</span></span>

<span data-ttu-id="f0387-185">다른 계층은 [가용성 그룹 수신기][sql-alwayson-listeners]를 통해 데이터베이스에 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-185">Other tiers connect to the database through an [availability group listener][sql-alwayson-listeners].</span></span> <span data-ttu-id="f0387-186">수신기는 SQL 클라이언트가 SQL Server의 물리적 인스턴스의 이름을 알지 못해도 연결할 수 있도록 해 줍니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-186">The listener enables a SQL client to connect without knowing the name of the physical instance of SQL Server.</span></span> <span data-ttu-id="f0387-187">데이터베이스에 액세스하는 VM은 도메인에 연결되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-187">VMs that access the database must be joined to the domain.</span></span> <span data-ttu-id="f0387-188">클라이언트(여기서는 다른 계층)는 DNS를 사용하여 수신기의 가상 네트워크 이름을 IP 주소로 해석합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-188">The client (in this case, another tier) uses DNS to resolve the listener's virtual network name into IP addresses.</span></span>

<span data-ttu-id="f0387-189">다음과 같이 SQL Server Always On 가용성 그룹을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-189">Configure the SQL Server Always On Availability Group as follows:</span></span>

1. <span data-ttu-id="f0387-190">WSFC(Windows Server 장애 조치 클러스터링) 클러스터, SQL Server Always On 가용성 그룹과 주 복제본을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-190">Create a Windows Server Failover Clustering (WSFC) cluster, a SQL Server Always On Availability Group, and a primary replica.</span></span> <span data-ttu-id="f0387-191">자세한 내용은 [Always On 가용성 그룹 시작][sql-alwayson-getting-started]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-191">For more information, see [Getting Started with Always On Availability Groups][sql-alwayson-getting-started].</span></span> 
2. <span data-ttu-id="f0387-192">고정 사설 IP 주소를 사용하여 내부 부하 분산 장치를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-192">Create an internal load balancer with a static private IP address.</span></span>
3. <span data-ttu-id="f0387-193">가용성 그룹 수신기를 만든 다음 수신기의 DNS 이름을 내부 부하 분산 장치의 IP 주소로 매핑합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-193">Create an availability group listener, and map the listener's DNS name to the IP address of an internal load balancer.</span></span> 
4. <span data-ttu-id="f0387-194">SQL Server 수신 포트(기본값: TCP 포트 1433)에 대한 부하 분산 장치 규칙을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-194">Create a load balancer rule for the SQL Server listening port (TCP port 1433 by default).</span></span> <span data-ttu-id="f0387-195">부하 분산 장치 규칙은 Direct Server Return이라고도 불리는 *부동 IP*를 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-195">The load balancer rule must enable *floating IP*, also called Direct Server Return.</span></span> <span data-ttu-id="f0387-196">이로 인해 VM은 클라이언트에 직접 응답하여 주 복제본에 대한 직접 연결을 지원하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-196">This causes the VM to reply directly to the client, which enables a direct connection to the primary replica.</span></span>
  
   > [!NOTE]
   > <span data-ttu-id="f0387-197">부동 IP가 지원된 경우에는 부하 분산 장치 규칙의 프론트 엔드 포트 번호가 백엔드 포트 번호와 같아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-197">When floating IP is enabled, the front-end port number must be the same as the back-end port number in the load balancer rule.</span></span>
   > 
   > 

<span data-ttu-id="f0387-198">SQL 클라이언트가 연결을 시도하면 부하 분산 장치가 연결 요청을 주 복제본으로 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-198">When a SQL client tries to connect, the load balancer routes the connection request to the primary replica.</span></span> <span data-ttu-id="f0387-199">다른 복제본으로의 장애 조치(failover)가 이루어지면 부하 분산 장치가 이후의 요청을 새로운 주 복제본으로 자동 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-199">If there is a failover to another replica, the load balancer automatically routes subsequent requests to a new primary replica.</span></span> <span data-ttu-id="f0387-200">자세한 내용은 [SQL Server Always On 가용성 그룹에 대한 ILB 수신기 구성][sql-alwayson-ilb]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-200">For more information, see [Configure an ILB listener for SQL Server Always On Availability Groups][sql-alwayson-ilb].</span></span>

<span data-ttu-id="f0387-201">장애 조치(failover)가 진행되는 동안에는 기존 클라이언트 연결이 닫힙니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-201">During a failover, existing client connections are closed.</span></span> <span data-ttu-id="f0387-202">장애 조치(failover)가 완료되면 새로운 연결이 새로운 주 복제본으로 라우팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-202">After the failover completes, new connections will be routed to the new primary replica.</span></span>

<span data-ttu-id="f0387-203">응용 프로그램에서 쓰기보다 읽기가 훨씬 많이 발생한다면 읽기 전용 쿼리 중 일부를 보조 복제본으로 부하 분산할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-203">If your application makes significantly more reads than writes, you can offload some of the read-only queries to a secondary replica.</span></span> <span data-ttu-id="f0387-204">[수신기를 사용하여 읽기 전용 보조 복제본에 연결(읽기 전용 라우팅)][sql-alwayson-read-only-routing]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-204">See [Using a Listener to Connect to a Read-Only Secondary Replica (Read-Only Routing)][sql-alwayson-read-only-routing].</span></span>

<span data-ttu-id="f0387-205">가용성 그룹의 [수동 장애 조치(failover)를 강제로 수행][sql-alwayson-force-failover]하여 배포 환경을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-205">Test your deployment by [forcing a manual failover][sql-alwayson-force-failover] of the availability group.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="f0387-206">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="f0387-206">Jumpbox</span></span>

<span data-ttu-id="f0387-207">jumpbox는 최소한의 성능 요구 사항만을 가지므로 jumpbox에 대해 Standard A1과 같이 크기가 작은 VM을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-207">The jumpbox will have minimal performance requirements, so select a small VM size for the jumpbox such as Standard A1.</span></span> 

<span data-ttu-id="f0387-208">jumpbox에 대한 [공용 IP 주소]를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-208">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="f0387-209">jumpbox를 다른 VM과 동일한 VNet 안의 별도의 관리 서브넷에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-209">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="f0387-210">공용 인터넷으로부터 응용 프로그램 워크로드를 실행하는 VM에 대한 RDP 액세스를 허용하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-210">Do not allow RDP access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="f0387-211">대신 이러한 VM에 대한 모든 RDP 액세스는 jumpbox를 통해 이루어져야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-211">Instead, all RDP access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="f0387-212">관리자는 jumpbox에 로그인한 뒤에 jumpbox에서 다른 VM에 로그인하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-212">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="f0387-213">jumpbox는 인터넷에서 수신되는 RDP 트래픽 중 알려진 안전한 IP 주소만을 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-213">The jumpbox allows RDP traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="f0387-214">jumpbox를 안전하게 보호하기 위해 NSG을 만든 다음 이를 jumpbox 서브넷에 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-214">To secure the jumpbox, create an NSG and apply it to the jumpbox subnet.</span></span> <span data-ttu-id="f0387-215">안전한 공용 IP 주소 집합으로부터 수신되는 RDP 연결만 허용하는 NSG 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-215">Add an NSG rule that allows RDP connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="f0387-216">NSG는 서브넷 또는 jumpbox NIC에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-216">The NSG can be attached either to the subnet or to the jumpbox NIC.</span></span> <span data-ttu-id="f0387-217">여기서는 동일한 서브넷에 다른 VM을 추가하더라도 RDP 트래픽이 jumpbox에서만 허용되도록 NIC에 연결하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-217">In this case, we recommend attaching it to the NIC, so RDP traffic is permitted only to the jumpbox, even if you add other VMs to the same subnet.</span></span>

<span data-ttu-id="f0387-218">관리 서브넷으로부터 수신되는 RDP 트래픽을 허용하도록 다른 서브넷에 대한 NSG를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-218">Configure the NSGs for the other subnets to allow RDP traffic from the management subnet.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="f0387-219">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f0387-219">Availability considerations</span></span>

<span data-ttu-id="f0387-220">데이터베이스 계층에 여러 개의 VM이 존재한다고 해서 자동으로 고가용성 데이터베이스가 되는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-220">At the database tier, having multiple VMs does not automatically translate into a highly available database.</span></span> <span data-ttu-id="f0387-221">관계형 데이터베이스의 고가용성을 달성하기 위해서는 일반적으로 복제와 장애 조치(failover)를 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-221">For a relational database, you will typically need to use replication and failover to achieve high availability.</span></span> <span data-ttu-id="f0387-222">SQL Server의 고가용성을 달성하기 위해서는 [Always On 가용성 그룹][sql-alwayson]을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-222">For SQL Server, we recommend using [Always On Availability Groups][sql-alwayson].</span></span> 

<span data-ttu-id="f0387-223">[VM용 Azure SLA][vm-sla]가 제공하는 가용성보다 높은 가용성이 필요한 경우, 두 지역 간에 응용 프로그램을 복제한 다음 장애 조치(failover)를 위해 Azure Traffic Manager를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-223">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, replicate the application across two regions and use Azure Traffic Manager for failover.</span></span> <span data-ttu-id="f0387-224">자세한 내용은 [고가용성을 위해 여러 지역에서 Windows VM 실행][multi-dc]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-224">For more information, see [Run Windows VMs in multiple regions for high availability][multi-dc].</span></span>   

## <a name="security-considerations"></a><span data-ttu-id="f0387-225">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f0387-225">Security considerations</span></span>

<span data-ttu-id="f0387-226">중요한 미사용 데이터를 암호화하고 [Azure Key Vault][azure-key-vault]를 사용하여 데이터베이스 암호화 키를 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-226">Encrypt sensitive data at rest and use [Azure Key Vault][azure-key-vault] to manage the database encryption keys.</span></span> <span data-ttu-id="f0387-227">Key Vault는 암호화 키를 HSM(하드웨어 보안 모듈)에 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-227">Key Vault can store encryption keys in hardware security modules (HSMs).</span></span> <span data-ttu-id="f0387-228">자세한 내용은 [Azure VM에서 SQL Server에 대한 Azure Key Vault 통합 구성][sql-keyvault]을 참조하세요. 데이터베이스 연결 문자열과 같은 응용 프로그램 비밀 데이터를 Key Vault에 저장하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-228">For more information, see [Configure Azure Key Vault Integration for SQL Server on Azure VMs][sql-keyvault] It's also recommended to store application secrets, such as database connection strings, in Key Vault.</span></span>

<span data-ttu-id="f0387-229">NVA(네트워크 가상 어플라이언스)를 추가하여 인터넷과 Azure 가상 네트워크 사이에 DMZ를 만드는 것도 좋은 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-229">Consider adding a network virtual appliance (NVA) to create a DMZ between the Internet and the Azure virtual network.</span></span> <span data-ttu-id="f0387-230">NVA는 방화벽, 패킷 조사, 감사, 사용자 지정 라우팅과 같은 네트워크 관련 작업을 수행하는 가상 어플라이언스를 통칭하는 용어입니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-230">NVA is a generic term for a virtual appliance that can perform network-related tasks, such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="f0387-231">자세한 내용은 [Azure와 인터넷 사이에 DMZ 구현][dmz]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-231">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="f0387-232">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f0387-232">Scalability considerations</span></span>

<span data-ttu-id="f0387-233">부하 분산 장치는 네트워크 트래픽을 웹 계층과 비즈니스 계층으로 분산합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-233">The load balancers distribute network traffic to the web and business tiers.</span></span> <span data-ttu-id="f0387-234">새 VM 인스턴스를 추가하여 수평 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-234">Scale horizontally by adding new VM instances.</span></span> <span data-ttu-id="f0387-235">부하의 정도에 따라 웹 계층과 비즈니스 계층을 각각 독립적으로 확장할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-235">Note that you can scale the web and business tiers independently, based on load.</span></span> <span data-ttu-id="f0387-236">클라이언트 선호도로 인해 발생할 수 있는 복잡성을 줄이기 위해서는 웹 계층에 속한 VM이 상태 비저장이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-236">To reduce possible complications caused by the need to maintain client affinity, the VMs in the web tier should be stateless.</span></span> <span data-ttu-id="f0387-237">비즈니스 로직을 호스팅하는 VM도 상태 비저장이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-237">The VMs hosting the business logic should also be stateless.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="f0387-238">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f0387-238">Manageability considerations</span></span>

<span data-ttu-id="f0387-239">[Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef] 또는 [Puppet][puppet]과 같은 일원화된 관리 도구를 사용하여 전체 시스템 관리를 간소화합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-239">Simplify management of the entire system by using centralized administration tools such as [Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef], or [Puppet][puppet].</span></span> <span data-ttu-id="f0387-240">이러한 도구를 사용하면 여러 VM으로부터 캡처된 진단 및 상태 정보를 통합하여 시스템을 종합적으로 파악할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-240">These tools can consolidate diagnostic and health information captured from multiple VMs to provide an overall view of the system.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="f0387-241">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="f0387-241">Deploy the solution</span></span>

<span data-ttu-id="f0387-242">이 참조 아키텍처에 대한 배포는 [GitHub][github-folder]에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-242">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="f0387-243">필수 조건</span><span class="sxs-lookup"><span data-stu-id="f0387-243">Prerequisites</span></span>

<span data-ttu-id="f0387-244">사용자의 구독에 참조 아키텍처를 배포하려면 먼저 다음 단계를 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-244">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="f0387-245">[참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-245">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="f0387-246">Azure CLI 2.0이 컴퓨터에 설치되어 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-246">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="f0387-247">CLI를 설치하려면 [Install Azure CLI 2.0][azure-cli-2](Azure CLI 2.0 설치)에 제시된 지침을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-247">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="f0387-248">[Azure 빌딩 블록][azbb] npm 패키지를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-248">Install the [Azure building blocks][azbb] npm package.</span></span>

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. <span data-ttu-id="f0387-249">명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 다음 명령 중 하나를 사용하여 Azure 계정에 로그인한 다음 프롬프트에 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-249">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="f0387-250">azbb를 사용하여 솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="f0387-250">Deploy the solution using azbb</span></span>

<span data-ttu-id="f0387-251">N 계층 응용 프로그램에 대한 Windows VM 참조 아키텍처를 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-251">To deploy the Windows VMs for an N-tier application reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="f0387-252">위의 필수 조건 단계 중 1단계에서 복제한 리포지토리가 있는 `virtual-machines\n-tier-windows` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-252">Navigate to the `virtual-machines\n-tier-windows` folder for the repository you cloned in step 1 of the pre-requisites above.</span></span>

2. <span data-ttu-id="f0387-253">매개 변수 파일은 배포 환경의 각 VM에 대해 관리자 사용자 이름과 암호의 기본값을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-253">The parameter file specifies a default adminstrator user name and password for each VM in the deployment.</span></span> <span data-ttu-id="f0387-254">참조 아키텍처를 배포하기 전에 먼저 이 값을 변경해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-254">You must change these before you deploy the reference architecture.</span></span> <span data-ttu-id="f0387-255">`n-tier-windows.json` 파일을 열고 **adminUsername** 필드와 **adminPassword** 필드를 새로운 설정으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-255">Open the `n-tier-windows.json` file and replace each **adminUsername** and **adminPassword** field with your new settings.</span></span>
  
   > [!NOTE]
   > <span data-ttu-id="f0387-256">이 배포가 진행될 때 **VirtualMachineExtension** 개체와 일부 **VirtualMachine** 개체의 **extensions** 설정에서 여러 개의 스크립트가 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-256">There are multiple scripts that run during this deployment both in the  **VirtualMachineExtension** objects and in the **extensions** settings for some of the **VirtualMachine** objects.</span></span> <span data-ttu-id="f0387-257">이러한 스크립트 중 일부에서 방금 변경한 관리자 사용자 이름과 암호가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-257">Some of these scripts require the administrator user name and password that you have just changed.</span></span> <span data-ttu-id="f0387-258">해당 스크립트를 검토하여 올바른 자격 증명을 지정했는지 확인하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-258">It's recommended that you review these scripts to ensure that you specified the correct credentials.</span></span> <span data-ttu-id="f0387-259">올바른 자격 증명을 지정하지 않으면 배포가 실패하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-259">The deployment may fail if you have not specified the correct credentials.</span></span>
   > 
   > 

<span data-ttu-id="f0387-260">파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-260">Save the file.</span></span>

3. <span data-ttu-id="f0387-261">아래에 표시된 대로 **azbb** 명령줄 도구를 사용하여 참조 아키텍처를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="f0387-261">Deploy the reference architecture using the **azbb** command line tool as shown below.</span></span>

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
   ```

<span data-ttu-id="f0387-262">Azure 구성 요소를 사용하여 이 샘플 참조 아키텍처를 배포하는 방법에 대한 자세한 내용은 [GitHub 리포지토리][git]를 방문하세요.</span><span class="sxs-lookup"><span data-stu-id="f0387-262">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[요새 호스트]: https://en.wikipedia.org/wiki/Bastion_host
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[공용 IP 주소]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[public IP address]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "Microsoft Azure를 사용하는 N 계층 아키텍처"