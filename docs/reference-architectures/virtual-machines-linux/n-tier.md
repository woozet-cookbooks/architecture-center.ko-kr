---
title: "Azure에서 N 계층 응용 프로그램에 대해 Linux VM 실행"
description: "Microsoft Azure에서 N 계층 아키텍처에 대한 Linux VM 실행 방법"
author: MikeWasson
ms.date: 11/22/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: e875a58aa83339560fd1de5b03a960f071883927
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/08/2018
---
# <a name="run-linux-vms-for-an-n-tier-application"></a><span data-ttu-id="ee34d-103">N 계층 응용 프로그램에 대해 Linux VM 실행</span><span class="sxs-lookup"><span data-stu-id="ee34d-103">Run Linux VMs for an N-tier application</span></span>

<span data-ttu-id="ee34d-104">이 참조 아키텍처는 N 계층 응용 프로그램에 대해 Linux VM(가상 머신)을 실행하는 데 관해 검증된 일련의 사례를 보여 줍니다</span><span class="sxs-lookup"><span data-stu-id="ee34d-104">This reference architecture shows a set of proven practices for running Linux virtual machines (VMs) for an N-tier application.</span></span> [<span data-ttu-id="ee34d-105">**이 솔루션을 배포합니다**.</span><span class="sxs-lookup"><span data-stu-id="ee34d-105">**Deploy this solution**.</span></span>](#deploy-the-solution)  

<span data-ttu-id="ee34d-106">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="ee34d-106">![[0]][0]</span></span>

<span data-ttu-id="ee34d-107">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="ee34d-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="ee34d-108">건축</span><span class="sxs-lookup"><span data-stu-id="ee34d-108">Architecture</span></span>

<span data-ttu-id="ee34d-109">N 계층 아키텍처를 구현하는 방법은 여러 가지가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-109">There are many ways to implement an N-tier architecture.</span></span> <span data-ttu-id="ee34d-110">이 다이어그램에서는 일반적인 3계층 웹 응용 프로그램을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-110">The diagram shows a typical 3-tier web application.</span></span> <span data-ttu-id="ee34d-111">이 아키텍처는 [부하가 분산된 VM을 실행하여 확장성 및 가용성 확보][multi-vm]를 기반으로 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-111">This architecture builds on [Run load-balanced VMs for scalability and availability][multi-vm].</span></span> <span data-ttu-id="ee34d-112">웹 및 비즈니스 계층은 부하 분산된 VM을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-112">The web and business tiers use load-balanced VMs.</span></span>

* <span data-ttu-id="ee34d-113">**가용성 집합.**</span><span class="sxs-lookup"><span data-stu-id="ee34d-113">**Availability sets.**</span></span> <span data-ttu-id="ee34d-114">각 계층에 대해 [가용성 집합][azure-availability-sets]을 만들고 각 계층에서 두 개 이상의 VM을 프로비전합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-114">Create an [availability set][azure-availability-sets] for each tier, and provision at least two VMs in each tier.</span></span>  <span data-ttu-id="ee34d-115">이렇게 하면 VM이 더 높은 [SLA(서비스 수준 약정)][vm-sla]를 충족할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-115">This makes the VMs eligible for a higher [service level agreement (SLA)][vm-sla] for VMs.</span></span> <span data-ttu-id="ee34d-116">단일 VM을 가용성 집합에서 배포할 수는 있지만, 단일 VM이 모든 OS 및 데이터 디스크에 대한 Azure Premium Storage를 사용하지 않는 한 단일 VM은 SLA를 보장하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-116">You can deploy a single VM in an availability set, but the single VM will not qualify for an SLA guarantee unless the single VM is using Azure Premium Storage for all OS and data disks.</span></span>  
* <span data-ttu-id="ee34d-117">**서브넷.**</span><span class="sxs-lookup"><span data-stu-id="ee34d-117">**Subnets.**</span></span> <span data-ttu-id="ee34d-118">각 계층에 대해 별도의 서브넷을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-118">Create a separate subnet for each tier.</span></span> <span data-ttu-id="ee34d-119">[CIDR] 표기법을 사용하여 주소 범위 및 서브넷 마스크를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-119">Specify the address range and subnet mask using [CIDR] notation.</span></span> 
* <span data-ttu-id="ee34d-120">**부하 분산 장치.**</span><span class="sxs-lookup"><span data-stu-id="ee34d-120">**Load balancers.**</span></span> <span data-ttu-id="ee34d-121">[인터넷 연결 부하 분산 장치][load-balancer-external]를 사용하여 들어오는 인터넷 트래픽을 웹 계층에 분산하고, [내부 부하 분산 장치][load-balancer-internal]를 사용하여 네트워크 트래픽을 웹 계층에서 비즈니스 계층으로 분산합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-121">Use an [Internet-facing load balancer][load-balancer-external] to distribute incoming Internet traffic to the web tier, and an [internal load balancer][load-balancer-internal] to distribute network traffic from the web tier to the business tier.</span></span>
* <span data-ttu-id="ee34d-122">**Azure DNS**.</span><span class="sxs-lookup"><span data-stu-id="ee34d-122">**Azure DNS**.</span></span> <span data-ttu-id="ee34d-123">[Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-123">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="ee34d-124">Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-124">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>
* <span data-ttu-id="ee34d-125">**Jumpbox.**</span><span class="sxs-lookup"><span data-stu-id="ee34d-125">**Jumpbox.**</span></span> <span data-ttu-id="ee34d-126">[요새 호스트]라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-126">Also called a [bastion host].</span></span> <span data-ttu-id="ee34d-127">관리자가 다른 VM에 연결할 때 사용하는 네트워크의 보안 VM입니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-127">A secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="ee34d-128">Jumpbox는 안전 목록에 있는 공용 IP 주소의 원격 트래픽만 허용하는 NSG를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-128">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="ee34d-129">NSG에서 SSH(보안 셸) 트래픽을 허용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-129">The NSG should permit secure shell (SSH) traffic.</span></span>
* <span data-ttu-id="ee34d-130">**모니터링.**</span><span class="sxs-lookup"><span data-stu-id="ee34d-130">**Monitoring.**</span></span> <span data-ttu-id="ee34d-131">[Nagios], [Zabbix] 또는 [Icinga]와 같은 모니터링 소프트웨어는 응답 시간, VM 가동 시간 및 시스템의 전반적인 상태에 대한 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-131">Monitoring software such as [Nagios], [Zabbix], or [Icinga] can give you insight into response time, VM uptime, and the overall health of your system.</span></span> <span data-ttu-id="ee34d-132">개별 관리 서브넷에 배치되어 있는 VM에 모니터링 소프트웨어를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-132">Install the monitoring software on a VM that's placed in a separate management subnet.</span></span>
* <span data-ttu-id="ee34d-133">**NSG.**</span><span class="sxs-lookup"><span data-stu-id="ee34d-133">**NSGs.**</span></span> <span data-ttu-id="ee34d-134">[NSG(네트워크 보안 그룹)][nsg]을 사용하여 VNet 내 네트워크 트래픽을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-134">Use [network security groups][nsg] (NSGs) to restrict network traffic within the VNet.</span></span> <span data-ttu-id="ee34d-135">예를 들어 여기에 표시된 3 계층 아키텍처에서 데이터베이스 계층은 비즈니스 계층 및 관리 서브넷뿐 아니라 웹 프런트 엔드의 트래픽을 허용하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-135">For example, in the 3-tier architecture shown here, the database tier does not accept traffic from the web front end, only from the business tier and the management subnet.</span></span>
* <span data-ttu-id="ee34d-136">**Apache Cassandra 데이터베이스**.</span><span class="sxs-lookup"><span data-stu-id="ee34d-136">**Apache Cassandra database**.</span></span> <span data-ttu-id="ee34d-137">복제 및 장애 조치(failover)를 사용하여 데이터 계층에서 높은 가용성을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-137">Provides high availability at the data tier, by enabling replication and failover.</span></span>

## <a name="recommendations"></a><span data-ttu-id="ee34d-138">권장 사항</span><span class="sxs-lookup"><span data-stu-id="ee34d-138">Recommendations</span></span>

<span data-ttu-id="ee34d-139">개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-139">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="ee34d-140">여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-140">Use these recommendations as a starting point.</span></span> 

### <a name="vnet--subnets"></a><span data-ttu-id="ee34d-141">VNet/서브넷</span><span class="sxs-lookup"><span data-stu-id="ee34d-141">VNet / Subnets</span></span>

<span data-ttu-id="ee34d-142">VNet을 만들 때 각 서브넷에 포함된 리소스에 몇 개의 IP 주소가 필요한지 결정합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-142">When you create the VNet, determine how many IP addresses your resources in each subnet require.</span></span> <span data-ttu-id="ee34d-143">[CIDR] 표기법을 사용하여 필요한 IP 주소를 충족하는 서브넷 마스크와 VNet 주소 범위를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-143">Specify a subnet mask and a VNet address range large enough for the required IP addresses using [CIDR] notation.</span></span> <span data-ttu-id="ee34d-144">표준 [사설 IP 주소 블록][private-ip-space](10.0.0.0/8, 172.16.0.0/12 및 192.168.0.0/16)에 해당하는 주소 공간을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-144">Use an address space that falls within the standard [private IP address blocks][private-ip-space], which are 10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16.</span></span>

<span data-ttu-id="ee34d-145">추후 VNet과 온-프레미스 네트워크 사이에 게이트웨이를 설정해야 할 경우에 대비하여 온-프레미스 네트워크와 중복되지 않는 주소 범위를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-145">Choose an address range that does not overlap with your on-premises network, in case you need to set up a gateway between the VNet and your on-premises network later.</span></span> <span data-ttu-id="ee34d-146">VNet을 만든 뒤에는 주소 범위를 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-146">Once you create the VNet, you can't change the address range.</span></span>

<span data-ttu-id="ee34d-147">기능 및 보안 요구 사항을 염두에 두고 서브넷을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-147">Design subnets with functionality and security requirements in mind.</span></span> <span data-ttu-id="ee34d-148">동일한 계층이나 역할에 속한 모든 VM은 동일한 서브넷에 속해야 합니다. 이때 서브넷은 보안 경계가 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-148">All VMs within the same tier or role should go into the same subnet, which can be a security boundary.</span></span> <span data-ttu-id="ee34d-149">VNet 및 서브넷 디자인에 대한 자세한 내용은 [Azure Virtual Networks 계획 및 디자인][plan-network]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-149">For more information about designing VNets and subnets, see [Plan and design Azure Virtual Networks][plan-network].</span></span>

<span data-ttu-id="ee34d-150">각 서브넷에 대해 CIDR 표기법을 사용하여 서브넷의 주소 공간을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-150">For each subnet, specify the address space for the subnet in CIDR notation.</span></span> <span data-ttu-id="ee34d-151">예를 들어 ‘10.0.0.0/24’는 256개의 IP 주소 범위를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-151">For example, '10.0.0.0/24' creates a range of 256 IP addresses.</span></span> <span data-ttu-id="ee34d-152">VM은 이 중에서 251개를 사용할 수 있습니다. 나머지 5개는 예약되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-152">VMs can use 251 of these; five are reserved.</span></span> <span data-ttu-id="ee34d-153">각 서브넷의 주소 공간이 겹치지 않아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-153">Make sure the address ranges don't overlap across subnets.</span></span> <span data-ttu-id="ee34d-154">[Virtual Network FAQ][vnet faq]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-154">See the [Virtual Network FAQ][vnet faq].</span></span>

### <a name="network-security-groups"></a><span data-ttu-id="ee34d-155">네트워크 보안 그룹</span><span class="sxs-lookup"><span data-stu-id="ee34d-155">Network security groups</span></span>

<span data-ttu-id="ee34d-156">NSG 규칙을 사용하여 계층 사이의 트래픽을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-156">Use NSG rules to restrict traffic between tiers.</span></span> <span data-ttu-id="ee34d-157">예를 들어 위에 표시된 3 계층 아키텍처에서 웹 계층은 데이터베이스 계층과 직접 통신하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-157">For example, in the 3-tier architecture shown above, the web tier does not communicate directly with the database tier.</span></span> <span data-ttu-id="ee34d-158">이를 위해서는 데이터베이스 계층에서 웹 계층 서브넷으로부터 수신되는 트래픽을 차단해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-158">To enforce this, the database tier should block incoming traffic from the web tier subnet.</span></span>  

1. <span data-ttu-id="ee34d-159">NSG를 만든 다음 이를 데이터베이스 계층 서브넷에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-159">Create an NSG and associate it to the database tier subnet.</span></span>
2. <span data-ttu-id="ee34d-160">VNet으로부터 수신되는 모든 트래픽을 차단하는 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-160">Add a rule that denies all inbound traffic from the VNet.</span></span> <span data-ttu-id="ee34d-161">(규칙에 `VIRTUAL_NETWORK` 태그를 사용합니다.)</span><span class="sxs-lookup"><span data-stu-id="ee34d-161">(Use the `VIRTUAL_NETWORK` tag in the rule.)</span></span> 
3. <span data-ttu-id="ee34d-162">비즈니스 계층 서브넷으로부터 수신되는 모든 트래픽을 허용하는 규칙을 높은 우선 순위로 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-162">Add a rule with a higher priority that allows inbound traffic from the business tier subnet.</span></span> <span data-ttu-id="ee34d-163">이 규칙은 이전 규칙을 재정의하며, 비즈니스 계층이 데이터베이스 계층과 통신할 수 있도록 해 줍니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-163">This rule overrides the previous rule, and allows the business tier to talk to the database tier.</span></span>
4. <span data-ttu-id="ee34d-164">데이터베이스 계층 서브넷 자체로부터 수신되는 트래픽을 허용하는 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-164">Add a rule that allows inbound traffic from within the database tier subnet itself.</span></span> <span data-ttu-id="ee34d-165">이 규칙은 데이터베이스 계층에 속한 VM 사이의 통신을 허용합니다. 이것은 데이터베이스 복제와 장애 조치(failover)를 위해 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-165">This rule allows communication between VMs in the database tier, which is needed for database replication and failover.</span></span>
5. <span data-ttu-id="ee34d-166">jumpbox 서브넷으로부터 수신되는 SSH 트래픽을 허용하는 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-166">Add a rule that allows SSH traffic from the jumpbox subnet.</span></span> <span data-ttu-id="ee34d-167">관리자는 이 규칙을 사용하여 jumpbox에서 데이터베이스 계층에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-167">This rule lets administrators connect to the database tier from the jumpbox.</span></span>
   
   > [!NOTE]
   > <span data-ttu-id="ee34d-168">NSG는 VNet 내부로부터 수신되는 모든 트래픽을 허용하는 [기본 규칙][nsg-rules]을 갖습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-168">An NSG has [default rules][nsg-rules] that allow any inbound traffic from within the VNet.</span></span> <span data-ttu-id="ee34d-169">이러한 규칙은 삭제할 수 없지만, 우선 순위가 더 높은 규칙을 만들면 재정의할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-169">These rules can't be deleted, but you can override them by creating higher-priority rules.</span></span>
   > 
   > 

### <a name="load-balancers"></a><span data-ttu-id="ee34d-170">부하 분산 장치</span><span class="sxs-lookup"><span data-stu-id="ee34d-170">Load balancers</span></span>

<span data-ttu-id="ee34d-171">외부 부하 분산 장치는 인터넷 트래픽을 웹 계층으로 분산합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-171">The external load balancer distributes Internet traffic to the web tier.</span></span> <span data-ttu-id="ee34d-172">이 부하 분산 장치에 사용할 공용 IP 주소를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-172">Create a public IP address for this load balancer.</span></span> <span data-ttu-id="ee34d-173">[인터넷 연결 부하 분산 장치 만들기][lb-external-create]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-173">See [Creating an Internet-facing load balancer][lb-external-create].</span></span>

<span data-ttu-id="ee34d-174">내부 부하 분산 장치는 웹 계층에서 비즈니스 계층으로 네트워크 트래픽을 분산합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-174">The internal load balancer distributes network traffic from the web tier to the business tier.</span></span> <span data-ttu-id="ee34d-175">이 부하 분산 장치에 사설 IP 주소를 부여하려면 프론트 엔드 IP 구성을 만든 다음 이를 비즈니스 계층의 서브넷에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-175">To give this load balancer a private IP address, create a frontend IP configuration and associate it with the subnet for the business tier.</span></span> <span data-ttu-id="ee34d-176">[내부 부하 분산 장치 만들기 시작][lb-internal-create]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-176">See [Get started creating an Internal load balancer][lb-internal-create].</span></span>

### <a name="cassandra"></a><span data-ttu-id="ee34d-177">Cassandra</span><span class="sxs-lookup"><span data-stu-id="ee34d-177">Cassandra</span></span>

<span data-ttu-id="ee34d-178">프로덕션 사용의 경우 [DataStax Enterprise][datastax]를 권장하나, 이러한 권장 사항은 모든 Cassandra 버전에 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-178">We recommend [DataStax Enterprise][datastax] for production use, but these recommendations apply to any Cassandra edition.</span></span> <span data-ttu-id="ee34d-179">Azure에서 DataStax 실행에 관한 자세한 내용은 [Azure용 DataStax Enterprise 배포 가이드][cassandra-in-azure]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-179">For more information on running DataStax in Azure, see [DataStax Enterprise Deployment Guide for Azure][cassandra-in-azure].</span></span> 

<span data-ttu-id="ee34d-180">Cassandra 클러스터에 대한 VM을 가용성 집합에 배치하여 Cassandra 복제본이 여러 오류 도메인 및 업그레이드 도메인에 분산되도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-180">Put the VMs for a Cassandra cluster in an availability set to ensure that the Cassandra replicas are distributed across multiple fault domains and upgrade domains.</span></span> <span data-ttu-id="ee34d-181">장애 도메인 및 업그레이드 도메인에 대한 자세한 내용은 [Virtual Machines의 가용성 관리][azure-availability-sets]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-181">For more information about fault domains and upgrade domains, see [Manage the availability of virtual machines][azure-availability-sets].</span></span> 

<span data-ttu-id="ee34d-182">가용성 집합당 오류 도메인(최대) 및 가용성 집합당 업그레이드 도메인 18개를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-182">Configure three fault domains (the maximum) per availability set and 18 upgrade domains per availability set.</span></span> <span data-ttu-id="ee34d-183">그러면 오류 도메인에 균등하게 분산될 수 있는 업그레이드 도메인의 최대 수가 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-183">This provides the maximum number of upgrade domains that can still be distributed evenly across the fault domains.</span></span>   

<span data-ttu-id="ee34d-184">노드를 랙 인식 모드로 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-184">Configure nodes in rack-aware mode.</span></span> <span data-ttu-id="ee34d-185">오류 도메인을 `cassandra-rackdc.properties` 파일의 랙에 매핑합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-185">Map fault domains to racks in the `cassandra-rackdc.properties` file.</span></span>

<span data-ttu-id="ee34d-186">클러스터 앞에 부하 분산 장치가 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-186">You don't need a load balancer in front of the cluster.</span></span> <span data-ttu-id="ee34d-187">클라이언트는 클러스터의 노드에 직접 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-187">The client connects directly to a node in the cluster.</span></span>

### <a name="jumpbox"></a><span data-ttu-id="ee34d-188">Jumpbox</span><span class="sxs-lookup"><span data-stu-id="ee34d-188">Jumpbox</span></span>

<span data-ttu-id="ee34d-189">jumpbox는 최소한의 성능 요구 사항만을 가지므로 jumpbox에 대해 Standard A1과 같이 크기가 작은 VM을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-189">The jumpbox will have minimal performance requirements, so select a small VM size for the jumpbox such as Standard A1.</span></span> 

<span data-ttu-id="ee34d-190">jumpbox에 대한 [공용 IP 주소]를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-190">Create a [public IP address] for the jumpbox.</span></span> <span data-ttu-id="ee34d-191">jumpbox를 다른 VM과 동일한 VNet 안에 배치하되 별도의 관리 서브넷에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-191">Place the jumpbox in the same VNet as the other VMs, but in a separate management subnet.</span></span>

<span data-ttu-id="ee34d-192">공용 인터넷으로부터 응용 프로그램 워크로드를 실행하는 VM에 대한 SSH 액세스를 허용하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-192">Do not allow SSH access from the public Internet to the VMs that run the application workload.</span></span> <span data-ttu-id="ee34d-193">대신 이러한 VM에 대한 모든 SSH 액세스는 jumpbox를 통해 이루어져야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-193">Instead, all SSH access to these VMs must come through the jumpbox.</span></span> <span data-ttu-id="ee34d-194">관리자는 jumpbox에 로그인한 뒤에 jumpbox에서 다른 VM에 로그인하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-194">An administrator logs into the jumpbox, and then logs into the other VM from the jumpbox.</span></span> <span data-ttu-id="ee34d-195">jumpbox는 인터넷에서 수신되는 SSH 트래픽 중 알려진 안전한 IP 주소만을 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-195">The jumpbox allows SSH traffic from the Internet, but only from known, safe IP addresses.</span></span>

<span data-ttu-id="ee34d-196">jumpbox를 안전하게 보호하려면 NSG을 만든 다음 이를 jumpbox 서브넷에 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-196">To secure the jumpbox, create an NSG and apply it to the jumpbox subnet.</span></span> <span data-ttu-id="ee34d-197">안전한 공용 IP 주소 집합으로부터 수신되는 SSH 연결만 허용하는 NSG 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-197">Add an NSG rule that allows SSH connections only from a safe set of public IP addresses.</span></span> <span data-ttu-id="ee34d-198">NSG는 서브넷 또는 jumpbox NIC에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-198">The NSG can be attached either to the subnet or to the jumpbox NIC.</span></span> <span data-ttu-id="ee34d-199">여기서는 동일한 서브넷에 다른 VM을 추가하더라도 SSH 트래픽이 jumpbox에서만 허용되도록 NIC에 연결하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-199">In this case, we recommend attaching it to the NIC, so SSH traffic is permitted only to the jumpbox, even if you add other VMs to the same subnet.</span></span>

<span data-ttu-id="ee34d-200">관리 서브넷으로부터 수신되는 SSH 트래픽을 허용하도록 다른 서브넷에 대한 NSG를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-200">Configure the NSGs for the other subnets to allow SSH traffic from the management subnet.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="ee34d-201">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ee34d-201">Availability considerations</span></span>

<span data-ttu-id="ee34d-202">각 계층 또는 VM 역할을 별도의 가용성 집합에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-202">Put each tier or VM role into a separate availability set.</span></span> 

<span data-ttu-id="ee34d-203">데이터베이스 계층에 여러 개의 VM이 존재한다고 해서 자동으로 고가용성 데이터베이스가 되는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-203">At the database tier, having multiple VMs does not automatically translate into a highly available database.</span></span> <span data-ttu-id="ee34d-204">관계형 데이터베이스의 경우 고가용성을 달성하기 위해서는 일반적으로 복제와 장애 조치(failover)를 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-204">For a relational database, you will typically need to use replication and failover to achieve high availability.</span></span>  

<span data-ttu-id="ee34d-205">[VM용 Azure SLA][vm-sla]가 제공하는 가용성보다 높은 가용성이 필요한 경우, 두 지역 간에 응용 프로그램을 복제한 다음 장애 조치(failover)를 위해 Azure Traffic Manager를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-205">If you need higher availability than the [Azure SLA for VMs][vm-sla] provides, replicate the application across two regions and use Azure Traffic Manager for failover.</span></span> <span data-ttu-id="ee34d-206">자세한 내용은 [고가용성을 위해 여러 지역에서 Linux VM 실행][multi-dc]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-206">For more information, see [Run Linux VMs in multiple regions for high availability][multi-dc].</span></span>  

## <a name="security-considerations"></a><span data-ttu-id="ee34d-207">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ee34d-207">Security considerations</span></span>

<span data-ttu-id="ee34d-208">NVA(네트워크 가상 어플라이언스)를 추가하여 공용 인터넷과 Azure 가상 네트워크 사이에 DMZ를 만드는 것도 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-208">Consider adding a network virtual appliance (NVA) to create a DMZ between the public Internet and the Azure virtual network.</span></span> <span data-ttu-id="ee34d-209">NVA는 방화벽, 패킷 조사, 감사 및 사용자 지정 라우팅과 같은 네트워크 관련 작업을 수행할 수 있는 가상 어플라이언스를 통칭하는 용어입니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-209">NVA is a generic term for a virtual appliance that can perform network-related tasks such as firewall, packet inspection, auditing, and custom routing.</span></span> <span data-ttu-id="ee34d-210">자세한 내용은 [Azure와 인터넷 사이에 DMZ 구현][dmz]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-210">For more information, see [Implementing a DMZ between Azure and the Internet][dmz].</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="ee34d-211">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ee34d-211">Scalability considerations</span></span>

<span data-ttu-id="ee34d-212">부하 분산 장치는 네트워크 트래픽을 웹 계층과 비즈니스 계층으로 분산합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-212">The load balancers distribute network traffic to the web and business tiers.</span></span> <span data-ttu-id="ee34d-213">새 VM 인스턴스를 추가하여 수평 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-213">Scale horizontally by adding new VM instances.</span></span> <span data-ttu-id="ee34d-214">부하의 정도에 따라 웹 계층과 비즈니스 계층을 각각 독립적으로 확장할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-214">Note that you can scale the web and business tiers independently, based on load.</span></span> <span data-ttu-id="ee34d-215">클라이언트 선호도로 인해 발생할 수 있는 복잡성을 줄이기 위해서는 웹 계층에 속한 VM이 상태 비저장이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-215">To reduce possible complications caused by the need to maintain client affinity, the VMs in the web tier should be stateless.</span></span> <span data-ttu-id="ee34d-216">비즈니스 로직을 호스팅하는 VM도 상태 비저장이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-216">The VMs hosting the business logic should also be stateless.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="ee34d-217">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ee34d-217">Manageability considerations</span></span>

<span data-ttu-id="ee34d-218">[Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef] 또는 [Puppet][puppet]과 같은 일원화된 관리 도구를 사용하여 전체 시스템 관리를 간소화합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-218">Simplify management of the entire system by using centralized administration tools such as [Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef], or [Puppet][puppet].</span></span> <span data-ttu-id="ee34d-219">이러한 도구를 사용하면 여러 VM으로부터 캡처된 진단 및 상태 정보를 통합하여 시스템을 종합적으로 파악할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-219">These tools can consolidate diagnostic and health information captured from multiple VMs to provide an overall view of the system.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="ee34d-220">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="ee34d-220">Deploy the solution</span></span>

<span data-ttu-id="ee34d-221">이 참조 아키텍처에 대한 배포는 [GitHub][github-folder]에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-221">A deployment for this reference architecture is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="ee34d-222">필수 구성 요소</span><span class="sxs-lookup"><span data-stu-id="ee34d-222">Prerequisites</span></span>

<span data-ttu-id="ee34d-223">사용자의 구독에 참조 아키텍처를 배포하려면 먼저 다음 단계를 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-223">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="ee34d-224">[AzureCAT 참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-224">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="ee34d-225">Azure CLI 2.0이 컴퓨터에 설치되어 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-225">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="ee34d-226">CLI를 설치하려면 [Install Azure CLI 2.0][azure-cli-2](Azure CLI 2.0 설치)에 제시된 지침을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-226">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="ee34d-227">[Azure 빌딩 블록][azbb] npm 패키지를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-227">Install the [Azure building blocks][azbb] npm package.</span></span>

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. <span data-ttu-id="ee34d-228">명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 다음 명령 중 하나를 사용하여 Azure 계정에 로그인한 다음 프롬프트에 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-228">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="ee34d-229">azbb를 사용하여 솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="ee34d-229">Deploy the solution using azbb</span></span>

<span data-ttu-id="ee34d-230">N 계층 응용 프로그램에 대한 Linux VM 참조 아키텍처를 배포하려면 다음 단계를 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-230">To deploy the Linux VMs for an N-tier application reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="ee34d-231">위의 필수 조건 중 1단계에서 복제한 리포지토리가 있는 `virtual-machines\n-tier-linux` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-231">Navigate to the `virtual-machines\n-tier-linux` folder for the repository you cloned in step 1 of the pre-requisites above.</span></span>

2. <span data-ttu-id="ee34d-232">매개 변수 파일은 배포 환경의 각 VM에 대해 관리자 사용자 이름과 암호의 기본값을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-232">The parameter file specifies a default adminstrator user name and password for each VM in the deployment.</span></span> <span data-ttu-id="ee34d-233">참조 아키텍처를 배포하기 전에 먼저 이 값을 변경해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-233">You must change these before you deploy the reference architecture.</span></span> <span data-ttu-id="ee34d-234">`n-tier-linux.json` 파일을 열고 **adminUsername** 및 **adminPassword** 필드를 새로운 설정으로 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-234">Open the `n-tier-linux.json` file and replace each **adminUsername** and **adminPassword** field with your new settings.</span></span>   <span data-ttu-id="ee34d-235">파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-235">Save the file.</span></span>

3. <span data-ttu-id="ee34d-236">아래에 표시된 대로 **azbb** 명령줄 도구를 사용하여 참조 아키텍처를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="ee34d-236">Deploy the reference architecture using the **azbb** command line tool as shown below.</span></span>

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
  ```

<span data-ttu-id="ee34d-237">Azure 구성 요소를 사용하여 이 샘플 참조 아키텍처를 배포하는 방법에 대한 자세한 내용은 [GitHub 리포지토리][git]를 방문하세요.</span><span class="sxs-lookup"><span data-stu-id="ee34d-237">For more information on deploying this sample reference architecture using Azure Building Blocks, visit the [GitHub repository][git].</span></span>

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[요새 호스트]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[공용 IP 주소]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-diagram.png "Microsoft Azure를 사용하는 N 계층 아키텍처"

