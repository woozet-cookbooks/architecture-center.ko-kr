---
title: Azure에서 고가용성 SharePoint Server 2016 팜 실행
description: Azure에 고가용성 SharePoint Server 2016 팜을 설정하는 방법에 대한 검증된 사례입니다.
author: njray
ms.date: 07/14/2018
ms.openlocfilehash: 04c69309e9f96e3bf7cd7faabeedd9b6d9da1ebd
ms.sourcegitcommit: 8b5fc0d0d735793b87677610b747f54301dcb014
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/29/2018
ms.locfileid: "39334133"
---
# <a name="run-a-high-availability-sharepoint-server-2016-farm-in-azure"></a><span data-ttu-id="286e9-103">Azure에서 고가용성 SharePoint Server 2016 팜 실행</span><span class="sxs-lookup"><span data-stu-id="286e9-103">Run a high availability SharePoint Server 2016 farm in Azure</span></span>

<span data-ttu-id="286e9-104">이 참조 아키텍처에서는 MinRole 토폴로지 및 SQL Server Always On 가용성 그룹을 사용하여 Azure에 고가용성 SharePoint Server 2016 팜을 설정하는 방법에 대한 검증된 사례 모음을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-104">This reference architecture shows a set of proven practices for setting up a high availability SharePoint Server 2016 farm on Azure, using MinRole topology and SQL Server Always On availability groups.</span></span> <span data-ttu-id="286e9-105">SharePoint 팜은 인터넷 연결 엔드포인트 또는 실체가 없는 안전한 가상 네트워크에 배포됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-105">The SharePoint farm is deployed in a secured virtual network with no Internet-facing endpoint or presence.</span></span> [<span data-ttu-id="286e9-106">**이 솔루션을 배포합니다**.</span><span class="sxs-lookup"><span data-stu-id="286e9-106">**Deploy this solution**.</span></span>](#deploy-the-solution) 

![](./images/sharepoint-ha.png)

<span data-ttu-id="286e9-107">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="286e9-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="286e9-108">아키텍처</span><span class="sxs-lookup"><span data-stu-id="286e9-108">Architecture</span></span>

<span data-ttu-id="286e9-109">이 아키텍처는 [N 계층 응용 프로그램에 대한 Windows VM 실행][windows-n-tier]에 나와 있는 아키텍처를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-109">This architecture builds on the one shown in [Run Windows VMs for an N-tier application][windows-n-tier].</span></span> <span data-ttu-id="286e9-110">Azure 가상 네트워크(VNet) 내부에 고가용성 SharePoint Server 2016 팜을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-110">It deploys a SharePoint Server 2016 farm with high availability inside an Azure virtual network (VNet).</span></span> <span data-ttu-id="286e9-111">이 아키텍처는 테스트 또는 프로덕션 환경, Office 365를 사용하는 SharePoint 하이브리드 인프라 또는 재해 복구 시나리오의 기준으로 사용하기에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-111">This architecture is suitable for a test or production environment, a SharePoint hybrid infrastructure with Office 365, or as the basis for a disaster recovery scenario.</span></span>

<span data-ttu-id="286e9-112">이 아키텍처는 다음과 같은 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-112">The architecture consists of the following components:</span></span>

- <span data-ttu-id="286e9-113">**리소스 그룹**.</span><span class="sxs-lookup"><span data-stu-id="286e9-113">**Resource groups**.</span></span> <span data-ttu-id="286e9-114">[리소스 그룹][resource-group]은 관련 Azure 리소스를 보유하는 컨테이너입니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-114">A [resource group][resource-group] is a container that holds related Azure resources.</span></span> <span data-ttu-id="286e9-115">한 리소스 그룹은 SharePoint 서버에 사용되고, 다른 리소스 그룹은 가상 네트워크나 부하 분산 장치처럼 VM에 독립적인 인프라 구성 요소에 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-115">One resource group is used for the SharePoint servers, and another resource group is used for infrastructure components that are independent of VMs, such as the virtual network and load balancers.</span></span>

- <span data-ttu-id="286e9-116">**가상 네트워크(VNet)**.</span><span class="sxs-lookup"><span data-stu-id="286e9-116">**Virtual network (VNet)**.</span></span> <span data-ttu-id="286e9-117">VM은 고유한 인트라넷 주소 공간을 사용하는 VNet에 배포됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-117">The VMs are deployed in a VNet with a unique intranet address space.</span></span> <span data-ttu-id="286e9-118">VNet은 서브넷으로 더욱 세분화됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-118">The VNet is further subdivided into subnets.</span></span> 

- <span data-ttu-id="286e9-119">**VM(가상 머신)**.</span><span class="sxs-lookup"><span data-stu-id="286e9-119">**Virtual machines (VMs)**.</span></span> <span data-ttu-id="286e9-120">VM은 VNet에 배포되며, 모든 VM은 개인 고정 IP 주소가 할당됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-120">The VMs are deployed into the VNet, and private static IP addresses are assigned to all of the VMs.</span></span> <span data-ttu-id="286e9-121">SQL Server 및 SharePoint Server 2016을 실행하는 VM에는 재시작 후 IP 주소 캐싱 및 주소 변경 문제를 방지할 수 있도록 고정 IP 주소를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-121">Static IP addresses are recommended for the VMs running SQL Server and SharePoint Server 2016, to avoid issues with IP address caching and changes of addresses after a restart.</span></span>

- <span data-ttu-id="286e9-122">**가용성 집합**.</span><span class="sxs-lookup"><span data-stu-id="286e9-122">**Availability sets**.</span></span> <span data-ttu-id="286e9-123">각 SharePoint 역할의 VM을 별도의 [가용성 집합][availability-set]에 배치하고, 역할마다 VM(가상 머신)을 두 대 이상 프로비전합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-123">Place the VMs for each SharePoint role into separate [availability sets][availability-set], and provision at least two virtual machines (VMs) for each role.</span></span> <span data-ttu-id="286e9-124">이렇게 하면 VM이 더 높은 Service Level Agreement(서비스 수준 약정)를 충족할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-124">This makes the VMs eligible for a higher service level agreement (SLA).</span></span> 

- <span data-ttu-id="286e9-125">**내부 부하 분산 장치**.</span><span class="sxs-lookup"><span data-stu-id="286e9-125">**Internal load balancer**.</span></span> <span data-ttu-id="286e9-126">[부하 분산 장치][load-balancer]는 온-프레미스 네트워크의 SharePoint 요청 트래픽을 SharePoint 팜의 프런트 엔드 웹 서버로 분산합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-126">The [load balancer][load-balancer] distributes SharePoint request traffic from the on-premises network to the front-end web servers of the SharePoint farm.</span></span> 

- <span data-ttu-id="286e9-127">**NSG(네트워크 보안 그룹)**.</span><span class="sxs-lookup"><span data-stu-id="286e9-127">**Network security groups (NSGs)**.</span></span> <span data-ttu-id="286e9-128">가상 머신을 포함하고 있는 서브넷마다 [네트워크 보안 그룹][nsg]이 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-128">For each subnet that contains virtual machines, a [network security group][nsg] is created.</span></span> <span data-ttu-id="286e9-129">서브넷을 격리하려면 NSG를 사용하여 VNet 내부의 네트워크 트래픽을 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-129">Use NSGs to restrict network traffic within the VNet, in order to isolate subnets.</span></span> 

- <span data-ttu-id="286e9-130">**게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="286e9-130">**Gateway**.</span></span> <span data-ttu-id="286e9-131">게이트웨이는 온-프레미스 네트워크와 Azure 가상 네트워크를 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-131">The gateway provides a connection between your on-premises network and the Azure virtual network.</span></span> <span data-ttu-id="286e9-132">연결에 ExpressRoute 또는 사이트 간 VPN을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-132">Your connection can use ExpressRoute or site-to-site VPN.</span></span> <span data-ttu-id="286e9-133">자세한 내용은 [온-프레미스 네트워크를 Azure에 연결][hybrid-ra]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-133">For more information, see [Connect an on-premises network to Azure][hybrid-ra].</span></span>

- <span data-ttu-id="286e9-134">**Windows Server AD(Active Directory) 도메인 컨트롤러**.</span><span class="sxs-lookup"><span data-stu-id="286e9-134">**Windows Server Active Directory (AD) domain controllers**.</span></span> <span data-ttu-id="286e9-135">이 참조 아키텍처는 Windows Server AD 도메인 컨트롤러를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-135">This reference architecture deploys Windows Server AD domain controllers.</span></span> <span data-ttu-id="286e9-136">이러한 도메인 컨트롤러는 Azure VNet에서 실행되며 온-프레미스 Windows Server AD 포리스트와 트러스트 관계에 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-136">These domain controllers run in the Azure VNet and have a trust relationship with the on-premises Windows Server AD forest.</span></span> <span data-ttu-id="286e9-137">게이트웨이 연결의 인증 트래픽을 온-프레미스 네트워크로 전송하는 대신 VNet에서 SharePoint 팜 리소스에 대한 클라이언트 웹 요청이 인증됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-137">Client web requests for SharePoint farm resources are authenticated in the VNet rather than sending that authentication traffic across the gateway connection to the on-premises network.</span></span> <span data-ttu-id="286e9-138">DNS에서 인트라넷 A 또는 CNAME 레코드가 생성되므로 인트라넷 사용자는 내부 부하 분산 장치의 개인 IP 주소에 해당하는 SharePoint 팜 이름을 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-138">In DNS, intranet A or CNAME records are created so that intranet users can resolve the name of the SharePoint farm to the private IP address of the internal load balancer.</span></span>

  <span data-ttu-id="286e9-139">SharePoint Server 2016은 [Azure Active Directory Domain Services](/azure/active-directory-domain-services/)를 사용하도록 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-139">SharePoint Server 2016 also supports using [Azure Active Directory Domain Services](/azure/active-directory-domain-services/).</span></span> <span data-ttu-id="286e9-140">Azure AD Domain Services는 관리 도메인 서비스를 제공하므로 Azure에서 도메인 컨트롤러를 배포하고 관리할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-140">Azure AD Domain Services provides managed domain services, so that you don't need to deploy and manage domain controllers in Azure.</span></span>

- <span data-ttu-id="286e9-141">**SQL Server Always On 가용성 그룹**.</span><span class="sxs-lookup"><span data-stu-id="286e9-141">**SQL Server Always On Availability Group**.</span></span> <span data-ttu-id="286e9-142">SQL Server 데이터 고가용성의 경우 [SQL Server Always On 가용성 그룹][sql-always-on]을 추천합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-142">For high availability of the SQL Server database, we recommend [SQL Server Always On Availability Groups][sql-always-on].</span></span> <span data-ttu-id="286e9-143">두 대의 가상 머신이 SQL Server에 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-143">Two virtual machines are used for SQL Server.</span></span> <span data-ttu-id="286e9-144">하나는 주 데이터베이스 복제본을 포함하고 다른 하나는 보조 복제본을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-144">One contains the primary database replica and the other contains the secondary replica.</span></span> 

- <span data-ttu-id="286e9-145">**주 노드 VM**.</span><span class="sxs-lookup"><span data-stu-id="286e9-145">**Majority node VM**.</span></span> <span data-ttu-id="286e9-146">이 VM은 장애 조치(failover) 클러스터의 쿼럼 설정을 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-146">This VM allows the failover cluster to establish quorum.</span></span> <span data-ttu-id="286e9-147">자세한 내용은 [장애 조치(Failover) 클러스터의 쿼럼 구성 이해][sql-quorum]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-147">For more information, see [Understanding Quorum Configurations in a Failover Cluster][sql-quorum].</span></span>

- <span data-ttu-id="286e9-148">**SharePoint 서버**.</span><span class="sxs-lookup"><span data-stu-id="286e9-148">**SharePoint servers**.</span></span> <span data-ttu-id="286e9-149">SharePoint 서버는 웹 프런트 엔드, 캐싱, 응용 프로그램 및 검색 역할을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-149">The SharePoint servers perform the web front-end, caching, application, and search roles.</span></span> 

- <span data-ttu-id="286e9-150">**Jumpbox**.</span><span class="sxs-lookup"><span data-stu-id="286e9-150">**Jumpbox**.</span></span> <span data-ttu-id="286e9-151">[요새 호스트][bastion-host]라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-151">Also called a [bastion host][bastion-host].</span></span> <span data-ttu-id="286e9-152">관리자가 다른 VM에 연결할 때 사용하는 네트워크의 보안 VM입니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-152">This is a secure VM on the network that administrators use to connect to the other VMs.</span></span> <span data-ttu-id="286e9-153">jumpbox는 안전 목록에 있는 공용 IP 주소의 원격 트래픽만 허용하는 NSG를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-153">The jumpbox has an NSG that allows remote traffic only from public IP addresses on a safe list.</span></span> <span data-ttu-id="286e9-154">NSG는 RDP(원격 데스크톱) 트래픽을 허용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-154">The NSG should permit remote desktop (RDP) traffic.</span></span>

## <a name="recommendations"></a><span data-ttu-id="286e9-155">권장 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-155">Recommendations</span></span>

<span data-ttu-id="286e9-156">개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-156">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="286e9-157">여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-157">Use these recommendations as a starting point.</span></span>

### <a name="resource-group-recommendations"></a><span data-ttu-id="286e9-158">리소스 그룹 권장 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-158">Resource group recommendations</span></span>

<span data-ttu-id="286e9-159">서버 역할에 따라 리소스 그룹을 나누고, 전역 리소스인 인프라 구성 요소에 사용할 별도의 리소스 그룹을 두는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-159">We recommend separating resource groups according to the server role, and having a separate resource group for infrastructure components that are global resources.</span></span> <span data-ttu-id="286e9-160">이 아키텍처에서는 SharePoint 리소스가 한 그룹을 형성하고, SQL Server와 기타 유틸리티 자산이 또 다른 그룹을 형성합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-160">In this architecture, the SharePoint resources form one group, while the SQL Server and other utility assets form another.</span></span>

### <a name="virtual-network-and-subnet-recommendations"></a><span data-ttu-id="286e9-161">가상 네트워크 및 서브넷 권장 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-161">Virtual network and subnet recommendations</span></span>

<span data-ttu-id="286e9-162">SharePoint 역할마다 서브넷을 하나씩 사용하고, 게이트웨이와 jumpbox에도 서브넷을 하나씩 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-162">Use one subnet for each SharePoint role, plus a subnet for the gateway and one for the jumpbox.</span></span> 

<span data-ttu-id="286e9-163">게이트웨이 서브넷의 이름을 *GatewaySubnet*으로 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-163">The gateway subnet must be named *GatewaySubnet*.</span></span> <span data-ttu-id="286e9-164">가상 네트워크 주소 공간의 마지막 부분에서 게이트웨이 서브넷 주소 공간을 할당합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-164">Assign the gateway subnet address space from the last part of the virtual network address space.</span></span> <span data-ttu-id="286e9-165">자세한 내용은 [VPN 게이트웨이를 사용하여 온-프레미스 네트워크를 Azure에 연결][hybrid-vpn-ra]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-165">For more information, see [Connect an on-premises network to Azure using a VPN gateway][hybrid-vpn-ra].</span></span>

### <a name="vm-recommendations"></a><span data-ttu-id="286e9-166">VM 권장 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-166">VM recommendations</span></span>

<span data-ttu-id="286e9-167">이 아키텍처에는 최소 44코어가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-167">This architecture requires a minimum of 44 cores:</span></span>

- <span data-ttu-id="286e9-168">Standard_DS3_v2(각각 4코어)에 SharePoint 서버 8개 = 32코어</span><span class="sxs-lookup"><span data-stu-id="286e9-168">8 SharePoint servers on Standard_DS3_v2 (4 cores each) = 32 cores</span></span>
- <span data-ttu-id="286e9-169">Standard_DS1_v2(각각 1코어)에 Active Directory 도메인 컨트롤러 2개 = 2코어</span><span class="sxs-lookup"><span data-stu-id="286e9-169">2 Active Directory domain controllers on Standard_DS1_v2 (1 core each) = 2 cores</span></span>
- <span data-ttu-id="286e9-170">Standard_DS3_v2에 SQL Server VM 2대 = 8코어</span><span class="sxs-lookup"><span data-stu-id="286e9-170">2 SQL Server VMs on Standard_DS3_v2 = 8 cores</span></span>
- <span data-ttu-id="286e9-171">Standard_DS1_v2에 주 노드 1개 = 1코어</span><span class="sxs-lookup"><span data-stu-id="286e9-171">1 majority node on Standard_DS1_v2 = 1 core</span></span>
- <span data-ttu-id="286e9-172">Standard_DS1_v2에 관리 서버 1개 = 1코어</span><span class="sxs-lookup"><span data-stu-id="286e9-172">1 management server on Standard_DS1_v2 = 1 core</span></span>

<span data-ttu-id="286e9-173">배포에 사용할 Azure 구독의 VM 코어 할당량이 충분해야 하며, 그렇지 않으면 배포가 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-173">Make sure your Azure subscription has enough VM core quota for the deployment, or the deployment will fail.</span></span> <span data-ttu-id="286e9-174">[Azure 구독 및 서비스 제한, 할당량 및 제약 조건][quotas]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-174">See [Azure subscription and service limits, quotas, and constraints][quotas].</span></span> 

<span data-ttu-id="286e9-175">Search Indexer를 제외한 모든 SharePoint 역할에는 [Standard_DS3_v2][vm-sizes-general] VM 크기를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-175">For all SharePoint roles except the Search Indexer, we recommended using the [Standard_DS3_v2][vm-sizes-general] VM size.</span></span> <span data-ttu-id="286e9-176">Search Indexer에는 최소한 [Standard_DS13_v2][vm-sizes-memory] 이상을 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-176">The Search Indexer should be at least the [Standard_DS13_v2][vm-sizes-memory] size.</span></span> <span data-ttu-id="286e9-177">테스트하기 위해 이 참조 아키텍처의 매개 변수 파일은 검색 인덱서 역할에 대해 더 작은 DS3_v2 크기를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-177">For testing, the parameter files for this reference architecture specify the smaller DS3_v2 size for the Search Indexer role.</span></span> <span data-ttu-id="286e9-178">프로덕션 배포의 경우 DS13 이상 크기를 사용하도록 매개 변수 파일을 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-178">For a production deployment, update the parameter files to use the DS13 size or larger.</span></span> <span data-ttu-id="286e9-179">자세한 정보는 [SharePoint Server 2016의 하드웨어 및 소프트웨어 요구 사항][sharepoint-reqs]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-179">For more information, see [Hardware and software requirements for SharePoint Server 2016][sharepoint-reqs].</span></span> 

<span data-ttu-id="286e9-180">SQL Server VM의 경우 최소 4코어 8GB RAM을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-180">For the SQL Server VMs, we recommend a minimum of 4 cores and 8 GB RAM.</span></span> <span data-ttu-id="286e9-181">이 참조 아키텍처의 매개 변수 파일은 DS3_v2 크기를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-181">The parameter files for this reference architecture specify the DS3_v2 size.</span></span> <span data-ttu-id="286e9-182">프로덕션 배포의 경우 더 큰 VM 크기를 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-182">For a production deployment, you might need to specify a larger VM size.</span></span> <span data-ttu-id="286e9-183">자세한 내용은 [저장소 및 SQL Server 용량 계획 및 구성(SharePoint Server)](/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#estimate-memory-requirements)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-183">For more information, see [Storage and SQL Server capacity planning and configuration (SharePoint Server)](/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#estimate-memory-requirements).</span></span> 
 
### <a name="nsg-recommendations"></a><span data-ttu-id="286e9-184">NSG 권장 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-184">NSG recommendations</span></span>

<span data-ttu-id="286e9-185">VM을 포함하는 서브넷마다 NSG를 하나씩 사용하여 서브넷을 격리하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-185">We recommend having one NSG for each subnet that contains VMs, to enable subnet isolation.</span></span> <span data-ttu-id="286e9-186">서브넷 격리를 구성하려면 각 서브넷에 허용 또는 거부되는 인바운드 또는 아웃 바운드 트래픽을 정의하는 NSG 규칙을 추가하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-186">If you want to configure subnet isolation, add NSG rules that define the allowed or denied inbound or outbound traffic for each subnet.</span></span> <span data-ttu-id="286e9-187">자세한 내용은 [네트워크 보안 그룹을 사용하여 네트워크 트래픽 필터링][virtual-networks-nsg]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-187">For more information, see [Filter network traffic with network security groups][virtual-networks-nsg].</span></span> 

<span data-ttu-id="286e9-188">게이트웨이 서브넷에는 NSG를 할당하지 마세요. 할당하면 게이트웨이의 작동이 중지됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-188">Do not assign an NSG to the gateway subnet, or the gateway will stop functioning.</span></span> 

### <a name="storage-recommendations"></a><span data-ttu-id="286e9-189">저장소 권장 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-189">Storage recommendations</span></span>

<span data-ttu-id="286e9-190">팜의 VM 저장소 구성은 온-프레미스 배포에 사용되는 적절한 모범 사례와 일치해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-190">The storage configuration of the VMs in the farm should match the appropriate best practices used for on-premises deployments.</span></span> <span data-ttu-id="286e9-191">SharePoint 서버는 로그에 사용할 별도의 디스크가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-191">SharePoint servers should have a separate disk for logs.</span></span> <span data-ttu-id="286e9-192">검색 인덱스 역할을 호스팅하는 SharePoint 서버는 검색 인덱스를 저장할 추가 디스크 공간이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-192">SharePoint servers hosting search index roles require additional disk space for the search index to be stored.</span></span> <span data-ttu-id="286e9-193">SQL Server의 경우 데이터와 로그를 구분하는 것이 표준 사례입니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-193">For SQL Server, the standard practice is to separate data and logs.</span></span> <span data-ttu-id="286e9-194">데이터베이스 백업 저장소에 디스크를 추가하고 [tempdb][tempdb]에 별도의 디스크를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-194">Add more disks for database backup storage, and use a separate disk for [tempdb][tempdb].</span></span>

<span data-ttu-id="286e9-195">안정성을 최대로 높이려면 [Azure Managed Disks][managed-disks]를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-195">For best reliability, we recommend using [Azure Managed Disks][managed-disks].</span></span> <span data-ttu-id="286e9-196">관리 디스크는 가용성 집합 내의 VM용 디스크를 격리하여 단일 실패 지점을 방지합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-196">Managed disks ensure that the disks for VMs within an availability set are isolated to avoid single points of failure.</span></span> 

> [!NOTE]
> <span data-ttu-id="286e9-197">현재 이 참조 아키텍처에 대한 Resource Manager 템플릿은 관리 디스크를 사용하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-197">Currently the Resource Manager template for this reference architecture does not use managed disks.</span></span> <span data-ttu-id="286e9-198">관리 디스크를 사용하도록 템플릿을 업데이트할 예정입니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-198">We are planning to update the template to use managed disks.</span></span>

<span data-ttu-id="286e9-199">모든 SharePoint 및 SQL Server VM에 프리미엄 관리 디스크를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-199">Use Premium managed disks for all SharePoint and SQL Server VMs.</span></span> <span data-ttu-id="286e9-200">주 노드 서버, 도메인 컨트롤러 및 관리 서버에 표준 관리 디스크를 사용해도 됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-200">You can use Standard managed disks for the majority node server, the domain controllers, and the management server.</span></span> 

### <a name="sharepoint-server-recommendations"></a><span data-ttu-id="286e9-201">SharePoint Server 권장 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-201">SharePoint Server recommendations</span></span>

<span data-ttu-id="286e9-202">SharePoint 팜을 구성하기 전에 서비스마다 Windows Server Active Directory 서비스 계정이 하나씩 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-202">Before configuring the SharePoint farm, make sure you have one Windows Server Active Directory service account per service.</span></span> <span data-ttu-id="286e9-203">이 아키텍처는 역할당 권한을 격리할 다음과 같은 최소 도메인 수준 계정이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-203">For this architecture, you need at a minimum the following domain-level accounts to isolate privilege per role:</span></span>

- <span data-ttu-id="286e9-204">SQL Server 서비스 계정</span><span class="sxs-lookup"><span data-stu-id="286e9-204">SQL Server Service account</span></span>
- <span data-ttu-id="286e9-205">설치 사용자 계정</span><span class="sxs-lookup"><span data-stu-id="286e9-205">Setup User account</span></span>
- <span data-ttu-id="286e9-206">서버 팜 계정</span><span class="sxs-lookup"><span data-stu-id="286e9-206">Server Farm account</span></span>
- <span data-ttu-id="286e9-207">서버 서비스 계정</span><span class="sxs-lookup"><span data-stu-id="286e9-207">Search Service account</span></span>
- <span data-ttu-id="286e9-208">콘텐츠 액세스 계정</span><span class="sxs-lookup"><span data-stu-id="286e9-208">Content Access account</span></span>
- <span data-ttu-id="286e9-209">웹앱 풀 계정</span><span class="sxs-lookup"><span data-stu-id="286e9-209">Web App Pool accounts</span></span>
- <span data-ttu-id="286e9-210">서비스 앱 풀 계정</span><span class="sxs-lookup"><span data-stu-id="286e9-210">Service App Pool accounts</span></span>
- <span data-ttu-id="286e9-211">캐시 슈퍼 사용자 계정</span><span class="sxs-lookup"><span data-stu-id="286e9-211">Cache Super User account</span></span>
- <span data-ttu-id="286e9-212">캐시 슈퍼 읽기 권한자 계정</span><span class="sxs-lookup"><span data-stu-id="286e9-212">Cache Super Reader account</span></span>

<span data-ttu-id="286e9-213">초당 최소 200MB라는 디스크 처리량 지원 요구를 충족하도록 검색 아키텍처를 계획해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-213">To meet the support requirement for disk throughput of 200 MB per second minimum, make sure to plan the Search architecture.</span></span> <span data-ttu-id="286e9-214">[SharePoint Server 2013에서 엔터프라이즈 검색 아키텍처 계획][sharepoint-search]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-214">See [Plan enterprise search architecture in SharePoint Server 2013][sharepoint-search].</span></span> <span data-ttu-id="286e9-215">그리고 [SharePoint Server 2016에서 크롤링하는 방법에 대한 모범 사례][sharepoint-crawling]의 지침을 따르세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-215">Also follow the guidelines in [Best practices for crawling in SharePoint Server 2016][sharepoint-crawling].</span></span>

<span data-ttu-id="286e9-216">또한 검색 구성 요소 데이터를 별도의 고성능 저장소 볼륨 또는 파티션에 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-216">In addition, store the search component data on a separate storage volume or partition with high performance.</span></span> <span data-ttu-id="286e9-217">부하를 줄이고 처리량을 높일 수 있도록 이 아키텍처에 필요한 개체 캐시 사용자 계정을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-217">To reduce load and improve throughput, configure the object cache user accounts, which are required in this architecture.</span></span> <span data-ttu-id="286e9-218">Windows Server 운영 체제 파일, SharePoint Server 2016 프로그램 파일 및 진단 로그를 별도의 중간 성능 저장소 볼륨 또는 파티션 3개에 분할합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-218">Split the Windows Server operating system files, the SharePoint Server 2016 program files, and diagnostics logs across three separate storage volumes or partitions with normal performance.</span></span> 

<span data-ttu-id="286e9-219">이러한 권장 사항에 대한 자세한 내용은 [SharePoint Server 2016의 초기 배포 관리 및 서비스 계정][sharepoint-accounts]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-219">For more information about these recommendations, see [Initial deployment administrative and service accounts in SharePoint Server 2016][sharepoint-accounts].</span></span>


### <a name="hybrid-workloads"></a><span data-ttu-id="286e9-220">하이브리드 워크로드</span><span class="sxs-lookup"><span data-stu-id="286e9-220">Hybrid workloads</span></span>

<span data-ttu-id="286e9-221">이 참조 아키텍처는 [SharePoint 하이브리드 환경] [sharepoint-hybrid]&mdash;으로 사용할 수 있는, 다시 말해서 SharePoint Server 2016을 Office 365 SharePoint Online으로 확장하는 SharePoint Server 2016 팜을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-221">This reference architecture deploys a SharePoint Server 2016 farm that can be used as a [SharePoint hybrid environment][sharepoint-hybrid] &mdash; that is, extending SharePoint Server 2016 to Office 365 SharePoint Online.</span></span> <span data-ttu-id="286e9-222">Office Online Server가 있는 분은 [Azure의 Office Web Apps 및 Office Online Server 지원 여부][office-web-apps]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-222">If you have Office Online Server, see [Office Web Apps and Office Online Server supportability in Azure][office-web-apps].</span></span>

<span data-ttu-id="286e9-223">이 배포의 기본 서비스 응용 프로그램은 하이브리드 워크로드를 지원하도록 설계되었습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-223">The default service applications in this deployment are designed to support hybrid workloads.</span></span> <span data-ttu-id="286e9-224">SharePoint 인프라를 변경하지 않고도 모든 SharePoint Server 2016 및 Office 365 하이브리드 워크로드를 이 팜에 배포할 수 있지만 한 가지 예외가 있습니다. 클라우드 하이브리드 검색 서비스 응용 프로그램은 기존 검색 토폴로지를 호스팅하는 서버에 배포하면 안 됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-224">All SharePoint Server 2016 and Office 365 hybrid workloads can be deployed to this farm without changes to the SharePoint infrastructure, with one exception: The Cloud Hybrid Search Service Application must not be deployed onto servers hosting an existing search topology.</span></span> <span data-ttu-id="286e9-225">따라서 이 하이브리드 시나리오를 지원하려면 하나 이상의 검색-역할 기반 VM을 팜에 추가해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-225">Therefore, one or more search-role-based VMs must be added to the farm to support this hybrid scenario.</span></span>

### <a name="sql-server-always-on-availability-groups"></a><span data-ttu-id="286e9-226">SQL Server Always On 가용성 그룹</span><span class="sxs-lookup"><span data-stu-id="286e9-226">SQL Server Always On Availability Groups</span></span>

<span data-ttu-id="286e9-227">SharePoint Server 2016에서 Azure SQL Database를 사용할 수 없으므로 이 아키텍처는 SQL Server 가상 머신을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-227">This architecture uses SQL Server virtual machines because SharePoint Server 2016 cannot use Azure SQL Database.</span></span> <span data-ttu-id="286e9-228">SQL Server에서 고가용성을 지원하려면 함께 장애 조치(failover)되는 데이터베이스 집합을 지정하여 가용성과 복원력을 높이는 Always On 가용성 그룹을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-228">To support high availability in SQL Server, we recommend using Always On Availability Groups, which specify a set of databases that fail over together, making them highly-available and recoverable.</span></span> <span data-ttu-id="286e9-229">이 참조 아키텍처에서는 배포하는 동안 데이터베이스가 생성되지만, 수동으로 Always On 가용성 그룹을 사용하도록 설정하고 가용성 그룹에 SharePoint 데이터베이스를 추가해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-229">In this reference architecture, the databases are created during deployment, but you must manually enable Always On Availability Groups and add the SharePoint databases to an availability group.</span></span> <span data-ttu-id="286e9-230">자세한 내용은 [가용성 그룹을 만들고 SharePoint 데이터베이스 추가][create-availability-group]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-230">For more information, see [Create the availability group and add the SharePoint databases][create-availability-group].</span></span>

<span data-ttu-id="286e9-231">또한 SQL Server 가상 머신의 내부 부하 분산 장치에 대한 개인 IP 주소인 수신기 IP 주소를 클러스터에 추가하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-231">We also recommend adding a listener IP address to the cluster, which is the private IP address of the internal load balancer for the SQL Server virtual machines.</span></span>

<span data-ttu-id="286e9-232">Azure에서 실행되는 SQL Server에 대한 권장 VM 크기 및 기타 권장 성능은 [Azure Virtual Machines의 SQL Server에 대한 성능 모범 사례][sql-performance]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-232">For recommended VM sizes and other performance recommendations for SQL Server running in Azure, see [Performance best practices for SQL Server in Azure Virtual Machines][sql-performance].</span></span> <span data-ttu-id="286e9-233">또한 [SharePoint Server 2016 팜의 SQL Server에 대한 모범 사례][sql-sharepoint-best-practices]의 권장 사항을 따르세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-233">Also follow the recommendations in [Best practices for SQL Server in a SharePoint Server 2016 farm][sql-sharepoint-best-practices].</span></span>

<span data-ttu-id="286e9-234">주 노드 서버는 복제 파트너와 분리된 별도의 컴퓨터에 상주하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-234">We recommend that the majority node server reside on a separate computer from the replication partners.</span></span> <span data-ttu-id="286e9-235">서버는 보호 우선 모드 세션의 보조 복제 파트너 서버가 자동 장애 조치(failover)의 시작 여부를 인식하도록 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-235">The server enables the secondary replication partner server in a high-safety mode session to recognize whether to initiate an automatic failover.</span></span> <span data-ttu-id="286e9-236">두 파트너와는 달리, 주 노드 서버는 데이터베이스를 제공하지 않는 대신 자동 장애 조치(failover)를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-236">Unlike the two partners, the majority node server doesn't serve the database but rather supports automatic failover.</span></span> 

## <a name="scalability-considerations"></a><span data-ttu-id="286e9-237">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-237">Scalability considerations</span></span>

<span data-ttu-id="286e9-238">기존 서버를 강화하려면 VM 크기를 변경하기만 하면 됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-238">To scale up the existing servers, simply change the VM size.</span></span> 

<span data-ttu-id="286e9-239">SharePoint Server 2016의 [MinRoles][minroles] 기능을 사용하면 서버의 역할을 기반으로 서버를 규모 확장하고 역할에서 서버를 제거할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-239">With the [MinRoles][minroles] capability in SharePoint Server 2016, you can scale out servers based on the server's role and also remove servers from a role.</span></span> <span data-ttu-id="286e9-240">서버를 역할에 추가할 때 원하는 단일 역할 또는 복합 역할을 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-240">When you add servers to a role, you can specify any of the single roles or one of the combined roles.</span></span> <span data-ttu-id="286e9-241">하지만 검색 역할에 서버를 추가하는 경우 PowerShell을 사용하여 검색 토폴로지도 다시 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-241">If you add servers to the Search role, however, you must also reconfigure the search topology using PowerShell.</span></span> <span data-ttu-id="286e9-242">MinRoles을 사용하여 역할을 변환할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-242">You can also convert roles using MinRoles.</span></span> <span data-ttu-id="286e9-243">자세한 내용은 [SharePoint Server 2016에서 MinRole 서버 팜 관리][sharepoint-minrole]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-243">For more information, see [Managing a MinRole Server Farm in SharePoint Server 2016][sharepoint-minrole].</span></span>

<span data-ttu-id="286e9-244">SharePoint Server 2016은 자동 크기 조정에 가상 머신 확장 집합을 사용하는 것을 지원하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-244">Note that SharePoint Server 2016 doesn't support using virtual machine scale sets for auto-scaling.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="286e9-245">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-245">Availability considerations</span></span>

<span data-ttu-id="286e9-246">이 참조 아키텍처는 역할마다 2개 이상의 VM이 가용성 집합에 배포되기 때문에 Azure 지역 내에서 고가용성을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-246">This reference architecture supports high availability within an Azure region, because each role has at least two VMs deployed in an availability set.</span></span>

<span data-ttu-id="286e9-247">지역 장애를 방지하려면 다른 Azure 지역에 별도의 재해 복구 팜을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-247">To protect against a regional failure, create a separate disaster recovery farm in a different Azure region.</span></span> <span data-ttu-id="286e9-248">RTO(복구 시간 목표) 및 RPO(복구 지점 목표)에 따라 설치 요구 사항이 결정됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-248">Your recovery time objectives (RTOs) and recovery point objectives (RPOs) will determine the setup requirements.</span></span> <span data-ttu-id="286e9-249">자세한 내용은 [SharePoint 2016에 대한 재해 복구 전략 선택][sharepoint-dr]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-249">For details, see [Choose a disaster recovery strategy for SharePoint 2016][sharepoint-dr].</span></span> <span data-ttu-id="286e9-250">보조 지역은 주 지역과 *쌍을 이루는 지역*이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-250">The secondary region should be a *paired region* with the primary region.</span></span> <span data-ttu-id="286e9-251">광범위한 중단 발생 시 한 지역의 복구가 모든 쌍의 복구보다 높은 우선 순위를 갖습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-251">In the event of a broad outage, recovery of one region is prioritized out of every pair.</span></span> <span data-ttu-id="286e9-252">자세한 내용은 [BCDR(비즈니스 연속성 및 재해 복구): Azure 쌍을 이루는 지역][paired-regions]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-252">For more information, see [Business continuity and disaster recovery (BCDR): Azure Paired Regions][paired-regions].</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="286e9-253">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-253">Manageability considerations</span></span>

<span data-ttu-id="286e9-254">서버, 서버 팜 및 사이트를 운영하고 유지 관리하려면 SharePoint 운영에 대한 다음 권장 사례를 따르세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-254">To operate and maintain servers, server farms, and sites, follow the recommended practices for SharePoint operations.</span></span> <span data-ttu-id="286e9-255">자세한 내용은 [SharePoint Server 2016 운영][sharepoint-ops]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-255">For more information, see [Operations for SharePoint Server 2016][sharepoint-ops].</span></span>

<span data-ttu-id="286e9-256">SharePoint 환경에서 SQL Server를 관리할 때 고려할 작업은 데이터베이스 응용 프로그램의 일반적인 고려 사항과 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-256">The tasks to consider when managing SQL Server in a SharePoint environment may differ from the ones typically considered for a database application.</span></span> <span data-ttu-id="286e9-257">야간 증분 백업으로 매주 모든 SQL 데이터베이스 전체를 백업하는 것이 모범 사례입니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-257">A best practice is to fully back up all SQL databases weekly with incremental nightly backups.</span></span> <span data-ttu-id="286e9-258">15분마다 백업 트랜잭션이 로깅됩니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-258">Back up transaction logs every 15 minutes.</span></span> <span data-ttu-id="286e9-259">또 다른 방법으로 기본 제공 SharePoint 작업을 비활성화하고 데이터베이스에 SQL Server 유지 관리 작업을 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-259">Another practice is to implement SQL Server maintenance tasks on the databases while disabling the built-in SharePoint ones.</span></span> <span data-ttu-id="286e9-260">자세한 내용은 [저장소 및 SQL Server 용량 계획 및 구성][sql-server-capacity-planning]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-260">For more information, see [Storage and SQL Server capacity planning and configuration][sql-server-capacity-planning].</span></span> 

## <a name="security-considerations"></a><span data-ttu-id="286e9-261">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="286e9-261">Security considerations</span></span>

<span data-ttu-id="286e9-262">SharePoint Server 2016을 실행하는 데 사용되는 도메인 수준 서비스 계정은 도메인 가입 및 인증 프로세스를 처리하기 위한 Windows Server AD 도메인 컨트롤러가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-262">The domain-level service accounts used to run SharePoint Server 2016 require Windows Server AD domain controllers for domain-join and authentication processes.</span></span> <span data-ttu-id="286e9-263">Azure Active Directory Domain Services는 이 용도로 사용할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-263">Azure Active Directory Domain Services can't be used for this purpose.</span></span> <span data-ttu-id="286e9-264">이미 인트라넷에서 가동 중인 Windows Server AD ID 인프라를 확장하기 위해 이 아키텍처는 기존 온-프레미스 Windows Server AD 포리스트의 두 Windows Server AD 복제 도메인 컨트롤러를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-264">To extend the Windows Server AD identity infrastructure already in place in the intranet, this architecture uses two Windows Server AD replica domain controllers of an existing on-premises Windows Server AD forest.</span></span>

<span data-ttu-id="286e9-265">또한 항상 보안 강화를 계획하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-265">In addition, it's always wise to plan for security hardening.</span></span> <span data-ttu-id="286e9-266">기타 권장 사항으로 다음과 같은 것들이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-266">Other recommendations include:</span></span>

- <span data-ttu-id="286e9-267">서브넷 및 역할을 격리하는 규칙을 NSG에 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-267">Add rules to NSGs to isolate subnets and roles.</span></span>
- <span data-ttu-id="286e9-268">VM에 공용 IP 주소를 할당하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-268">Don't assign public IP addresses to VMs.</span></span>
- <span data-ttu-id="286e9-269">침입 감지 및 페이로드 분석의 경우 내부 Azure 부하 분산 장치 대신 프런트 엔드 웹 서버 앞에 네트워크 가상 어플라이언스를 사용하는 방안을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-269">For intrusion detection and analysis of payloads, consider using a network virtual appliance in front of the front-end web servers instead of an internal Azure load balancer.</span></span>
- <span data-ttu-id="286e9-270">필요에 따라 서버 간 일반 텍스트 트래픽을 암호화하는 IPsec 정책을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-270">As an option, use IPsec policies for encryption of cleartext traffic between servers.</span></span> <span data-ttu-id="286e9-271">서브넷도 격리하려는 경우 IPsec 트래픽을 허용하도록 네트워크 보안 그룹 규칙을 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-271">If you are also doing subnet isolation, update your network security group rules to allow IPsec traffic.</span></span>
- <span data-ttu-id="286e9-272">VM을 위한 맬웨어 방지 에이전트를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-272">Install anti-malware agents for the VMs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="286e9-273">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="286e9-273">Deploy the solution</span></span>

<span data-ttu-id="286e9-274">이 참조 아키텍처에 대한 배포는 [GitHub][github]에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-274">A deployment for this reference architecture is available on [GitHub][github].</span></span> <span data-ttu-id="286e9-275">전체 배포를 완료하는 데 몇 시간이 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-275">The entire deployment can take several hours to complete.</span></span>

<span data-ttu-id="286e9-276">배포는 구독에서 다음과 같은 리소스 그룹을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-276">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="286e9-277">ra-onprem-sp2016-rg</span><span class="sxs-lookup"><span data-stu-id="286e9-277">ra-onprem-sp2016-rg</span></span>
- <span data-ttu-id="286e9-278">ra-sp2016-network-rg</span><span class="sxs-lookup"><span data-stu-id="286e9-278">ra-sp2016-network-rg</span></span>

<span data-ttu-id="286e9-279">템플릿 매개 변수 파일은 이러한 이름을 참조하므로 이름을 변경하는 경우 매개 변수 파일을 일치하도록 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-279">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span> 

<span data-ttu-id="286e9-280">매개 변수 파일은 다양한 위치에서 하드 코딩된 암호를 포함하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-280">The parameter files include a hard-coded password in various places.</span></span> <span data-ttu-id="286e9-281">배포하기 전에 이러한 값을 변경하세요.</span><span class="sxs-lookup"><span data-stu-id="286e9-281">Change these values before you deploy.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="286e9-282">필수 조건</span><span class="sxs-lookup"><span data-stu-id="286e9-282">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution"></a><span data-ttu-id="286e9-283">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="286e9-283">Deploy the solution</span></span> 

1. <span data-ttu-id="286e9-284">시뮬레이션된 온-프레미스 네트워크를 배포하려면 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-284">Run the following command to deploy a simulated on-premises network.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p onprem.json --deploy
    ```

2. <span data-ttu-id="286e9-285">Azure VNet 및 VPN Gateway를 배포하려면 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-285">Run the following command to deploy the Azure VNet and the VPN gateway.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p connections.json --deploy
    ```

3. <span data-ttu-id="286e9-286">Jumpbox, AD 도메인 컨트롤러 및 SQL Server VM을 배포하려면 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-286">Run the following command to deploy the jumpbox, AD domain controllers, and SQL Server VMs.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure1.json --deploy
    ```

4. <span data-ttu-id="286e9-287">장애 조치 클러스터 및 가용성 그룹을 만들려면 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-287">Run the following command to create the failover cluster and the availability group.</span></span> 

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure2-cluster.json --deploy

5. Run the following command to deploy the remaining VMs.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure3.json --deploy
    ```

<span data-ttu-id="286e9-288">이 경우에 SQL Server Always On 가용성 그룹의 경우 웹 프런트 엔드에서 부하 분산 장치로 TCP를 연결할 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-288">At this point, verify that you can make a TCP conection from the web front end to the load balancer for the SQL Server Always On availability group.</span></span> <span data-ttu-id="286e9-289">이렇게 하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-289">To do so, perform the following steps:</span></span>

1. <span data-ttu-id="286e9-290">Azure Portal을 사용하여 `ra-sp2016-network-rg` 리소스 그룹에서 `ra-sp-jb-vm1`이라는 VM을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-290">Use the Azure portal to find the VM named `ra-sp-jb-vm1` in the `ra-sp2016-network-rg` resource group.</span></span> <span data-ttu-id="286e9-291">이것이 Jumpbox VM입니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-291">This is the jumpbox VM.</span></span>

2. <span data-ttu-id="286e9-292">`Connect`를 클릭하여 VM에 대한 원격 데스크톱 세션을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-292">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="286e9-293">`azure1.json` 매개 변수 파일에서 지정한 암호를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-293">Use the password that you specified in the `azure1.json` parameter file.</span></span>

3. <span data-ttu-id="286e9-294">원격 데스크톱 세션에서 10.0.5.4로 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-294">From the Remote Desktop session, log into 10.0.5.4.</span></span> <span data-ttu-id="286e9-295">`ra-sp-app-vm1`이라는 VM의 IP 주소입니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-295">This is the IP address of the VM named `ra-sp-app-vm1`.</span></span>

4. <span data-ttu-id="286e9-296">VM에서 PowerShell 콘솔을 열고, `Test-NetConnection` cmdlet을 사용하여 부하 분산 장치에 연결할 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-296">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the load balancer.</span></span>

    ```powershell
    Test-NetConnection 10.0.3.100 -Port 1433
    ```

<span data-ttu-id="286e9-297">출력은 다음과 비슷해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-297">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.3.100
RemoteAddress    : 10.0.3.100
RemotePort       : 1433
InterfaceAlias   : Ethernet 3
SourceAddress    : 10.0.0.132
TcpTestSucceeded : True
```

<span data-ttu-id="286e9-298">실패한 경우 Azure Portal을 사용하여 `ra-sp-sql-vm2`라는 VM을 다시 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-298">If it fails, use the Azure Portal to restart the VM named `ra-sp-sql-vm2`.</span></span> <span data-ttu-id="286e9-299">VM을 다시 시작한 후에 `Test-NetConnection` 명령을 다시 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-299">After the VM restarts, run the `Test-NetConnection` command again.</span></span> <span data-ttu-id="286e9-300">연결이 성공하려면 VM이 다시 시작된 후에 1분 정도 대기해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-300">You may need to wait about a minute after the VM restarts for the connection to succeed.</span></span> 

<span data-ttu-id="286e9-301">이제 다음과 같은 배포를 완료합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-301">Now complete the deployment as follows.</span></span>

1. <span data-ttu-id="286e9-302">SharePoint 팜 주 노드를 배포하려면 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-302">Run the following command to deploy the SharePoint farm primary node.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure4-sharepoint-server.json --deploy
    ```

2. <span data-ttu-id="286e9-303">SharePoint 캐시, 검색 및 웹을 배포하려면 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-303">Run the following command to deploy the SharePoint cache, search, and web.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure5-sharepoint-farm.json --deploy
    ```

3. <span data-ttu-id="286e9-304">NSG 규칙을 만들려면 다음 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-304">Run the following command to create the NSG rules.</span></span>

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure6-security.json --deploy
    ```

### <a name="validate-the-deployment"></a><span data-ttu-id="286e9-305">배포 유효성 검사</span><span class="sxs-lookup"><span data-stu-id="286e9-305">Validate the deployment</span></span>

1. <span data-ttu-id="286e9-306">[Azure Portal][azure-portal]에서 `ra-onprem-sp2016-rg` 리소스 그룹으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-306">In the [Azure portal][azure-portal], navigate to the `ra-onprem-sp2016-rg` resource group.</span></span>

2. <span data-ttu-id="286e9-307">리소스 목록에서 `ra-onpr-u-vm1`이라는 VM 리소스를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-307">In the list of resources, select the VM resource named `ra-onpr-u-vm1`.</span></span> 

3. <span data-ttu-id="286e9-308">[가상 머신에 연결][connect-to-vm]의 설명에 따라 VM을 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-308">Connect to the VM, as described in [Connect to virtual machine][connect-to-vm].</span></span> <span data-ttu-id="286e9-309">사용자 이름은 `\onpremuser`입니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-309">The user name is `\onpremuser`.</span></span>

5.  <span data-ttu-id="286e9-310">VM에 대한 원격 연결이 설정되면 VM에서 브라우저를 열고 `http://portal.contoso.local`로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-310">When the remote connection to the VM is established, open a browser in the VM and navigate to `http://portal.contoso.local`.</span></span>

6.  <span data-ttu-id="286e9-311">**Windows 보안** 상자에서 사용자 이름으로 `contoso.local\testuser`를 사용하여 SharePoint 포털에 로그온합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-311">In the **Windows Security** box, log on to the SharePoint portal using `contoso.local\testuser` for the user name.</span></span>

<span data-ttu-id="286e9-312">이 로그온은 온-프레미스 네트워크에서 사용하는 Fabrikam.com 도메인에서 SharePoint 포털이 사용하는 contoso.local 도메인으로 터널링합니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-312">This logon tunnels from the Fabrikam.com domain used by the on-premises network to the contoso.local domain used by the SharePoint portal.</span></span> <span data-ttu-id="286e9-313">SharePoint 사이트가 열리면 루트 데모 사이트가 보일 것입니다.</span><span class="sxs-lookup"><span data-stu-id="286e9-313">When the SharePoint site opens, you'll see the root demo site.</span></span>

<span data-ttu-id="286e9-314">**_이 참조 아키텍처에 기여하신 분들_** &mdash;  Joe Davies, Bob Fox, Neil Hodgkinson, Paul Stork</span><span class="sxs-lookup"><span data-stu-id="286e9-314">**_Contributors to this reference architecture_** &mdash;  Joe Davies, Bob Fox, Neil Hodgkinson, Paul Stork</span></span>

<!-- links -->

[availability-set]: /azure/virtual-machines/windows/manage-availability
[azure-portal]: https://portal.azure.com
[azure-ps]: /powershell/azure/overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[bastion-host]: https://en.wikipedia.org/wiki/Bastion_host
[create-availability-group]: https://technet.microsoft.com/library/mt793548(v=office.16).aspx
[connect-to-vm]: /azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine
[github]: https://github.com/mspnp/reference-architectures
[hybrid-ra]: ../hybrid-networking/index.md
[hybrid-vpn-ra]: ../hybrid-networking/vpn.md
[load-balancer]: /azure/load-balancer/load-balancer-internal-overview
[managed-disks]: /azure/storage/storage-managed-disks-overview
[minroles]: https://technet.microsoft.com/library/mt346114(v=office.16).aspx
[nsg]: /azure/virtual-network/virtual-networks-nsg
[office-web-apps]: https://support.microsoft.com/help/3199955/office-web-apps-and-office-online-server-supportability-in-azure
[paired-regions]: /azure/best-practices-availability-paired-regions
[readme]: https://github.com/mspnp/reference-architectures/tree/master/sharepoint/sharepoint-2016
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[quotas]: /azure/azure-subscription-service-limits
[sharepoint-accounts]: https://technet.microsoft.com/library/ee662513(v=office.16).aspx
[sharepoint-crawling]: https://technet.microsoft.com/library/dn535606(v=office.16).aspx
[sharepoint-dr]: https://technet.microsoft.com/library/ff628971(v=office.16).aspx
[sharepoint-hybrid]: https://aka.ms/sphybrid
[sharepoint-minrole]: https://technet.microsoft.com/library/mt743705(v=office.16).aspx
[sharepoint-ops]: https://technet.microsoft.com/library/cc262289(v=office.16).aspx
[sharepoint-reqs]: https://technet.microsoft.com/library/cc262485(v=office.16).aspx
[sharepoint-search]: https://technet.microsoft.com/library/dn342836.aspx
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server
[sql-performance]: /virtual-machines/windows/sql/virtual-machines-windows-sql-performance
[sql-server-capacity-planning]: https://technet.microsoft.com/library/cc298801(v=office.16).aspx
[sql-quorum]: https://technet.microsoft.com/library/cc731739(v=ws.11).aspx
[sql-sharepoint-best-practices]: https://technet.microsoft.com/library/hh292622(v=office.16).aspx
[tempdb]: /sql/relational-databases/databases/tempdb-database
[virtual-networks-nsg]: /azure/virtual-network/virtual-networks-nsg
[visio-download]: https://archcenter.blob.core.windows.net/cdn/Sharepoint-2016.vsdx
[vm-sizes-general]: /azure/virtual-machines/windows/sizes-general
[vm-sizes-memory]: /azure/virtual-machines/windows/sizes-memory
[windows-n-tier]: ../virtual-machines-windows/n-tier.md
