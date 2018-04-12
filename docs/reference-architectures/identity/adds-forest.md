---
title: Azure에서 ADDS 리소스 포리스트 만들기
description: >-
  Azure에서 신뢰할 수 있는 Active Directory 도메인을 만드는 방법입니다.

  지침, vpn-게이트웨이, expressroute, 부하 분산 장치, 가상 네트워크, active-directory
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: e32a6420821e70c84e77d2c39614f0c45efbb7e2
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a><span data-ttu-id="c7311-104">Azure에서 AAD DS(Active Directory Domain Services) 리소스 포리스트 만들기</span><span class="sxs-lookup"><span data-stu-id="c7311-104">Create an Active Directory Domain Services (AD DS) resource forest in Azure</span></span>

<span data-ttu-id="c7311-105">이 참조 아키텍처는 온-프레미스 AD 포레스트에 있는 도메인의 신뢰를 받는 Azure의 별도 Active Directory 도메인을 생성하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-105">This reference architecture shows how to create a separate Active Directory domain in Azure that is trusted by domains in your on-premises AD forest.</span></span> [<span data-ttu-id="c7311-106">**이 솔루션을 배포합니다**.</span><span class="sxs-lookup"><span data-stu-id="c7311-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="c7311-107">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="c7311-107">[![0]][0]</span></span> 

<span data-ttu-id="c7311-108">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="c7311-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="c7311-109">AD DS(Active Directory Domain Services)는 ID 정보를 계층 구조에 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-109">Active Directory Domain Services (AD DS) stores identity information in a hierarchical structure.</span></span> <span data-ttu-id="c7311-110">계층 구조의 최상위 노드를 포레스트라고 합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-110">The top node in the hierarchical structure is known as a forest.</span></span> <span data-ttu-id="c7311-111">포레스트는 도메인을, 도메인은 다른 유형의 개체를 담고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-111">A forest contains domains, and domains contain other types of objects.</span></span> <span data-ttu-id="c7311-112">이 참조 아키텍처는 온-프레미스 도메인과의 단방향의 보내는 트러스트 관계를 구축하여 Azure에 AD DS 포레스트를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-112">This reference architecture creates an AD DS forest in Azure with a one-way outgoing trust relationship with an on-premises domain.</span></span> <span data-ttu-id="c7311-113">Azure에 있는 포레스트는 온-프레미스에 없는 도메인을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-113">The forest in Azure contains a domain that does not exist on-premises.</span></span> <span data-ttu-id="c7311-114">트러스트 관계를 바탕으로 온-프레미스 도메인에서 수행한 로그인을 통해 별개의 Azure 도메인에 있는 리소스에 접속할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-114">Because of the trust relationship, logons made against on-premises domains can be trusted for access to resources in the separate Azure domain.</span></span> 

<span data-ttu-id="c7311-115">이 아키텍처의 일반적인 용도는 클라우드에 저장된 개체 및 ID에 대한 보안 분리를 유지하는 것과 개별 도메인을 온-프레미스에서 클라우드로 마이그레이션하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-115">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span> 

<span data-ttu-id="c7311-116">추가 고려 사항은 [온-프레미스 Active Directory를 Azure와 통합하기 위한 솔루션 선택][considerations]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-116">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span> 

## <a name="architecture"></a><span data-ttu-id="c7311-117">건축</span><span class="sxs-lookup"><span data-stu-id="c7311-117">Architecture</span></span>

<span data-ttu-id="c7311-118">이 아키텍처의 구성 요소는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-118">The architecture has the following components.</span></span>

* <span data-ttu-id="c7311-119">**온-프레미스 네트워크**.</span><span class="sxs-lookup"><span data-stu-id="c7311-119">**On-premises network**.</span></span> <span data-ttu-id="c7311-120">온-프레미스 네트워크는 자체 Active Directory 포레스트와 도메인을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-120">The on-premises network contains its own Active Directory forest and domains.</span></span>
* <span data-ttu-id="c7311-121">**Active Directory 서버**.</span><span class="sxs-lookup"><span data-stu-id="c7311-121">**Active Directory servers**.</span></span> <span data-ttu-id="c7311-122">클라우드에서 VM으로 실행되는 도메인 서비스를 구현하는 도메인 컨트롤러입니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-122">These are domain controllers implementing domain services running as VMs in the cloud.</span></span> <span data-ttu-id="c7311-123">이러한 서버는 온-프레미스에 위치한 도메인들과 분리된 하나 이상의 도메인을 포함하는 포레스트를 호스트합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-123">These servers host a forest containing one or more domains, separate from those located on-premises.</span></span>
* <span data-ttu-id="c7311-124">**단방향 트러스트 관계**.</span><span class="sxs-lookup"><span data-stu-id="c7311-124">**One-way trust relationship**.</span></span> <span data-ttu-id="c7311-125">이 다이어그램의 예제는 Azure 도메인으로부터 온-프레미스 도메인으로의 단방향 트러스트 관계를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-125">The example in the diagram shows a one-way trust from the domain in Azure to the on-premises domain.</span></span> <span data-ttu-id="c7311-126">이 관계를 통해 온-프레미스 사용자는 Azure의 도메인에 있는 리소스에 접근할 수 있지만 Azure 사용자가 온-프레미스의 리소스에 접근할 수는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-126">This relationship enables on-premises users to access resources in the domain in Azure, but not the other way around.</span></span> <span data-ttu-id="c7311-127">클라우드 사용자가 온-프레미스 리소스 액세스를 요구하는 경우 양방향 트러스트 관계를 생성할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-127">It is possible to create a two-way trust if cloud users also require access to on-premises resources.</span></span>
* <span data-ttu-id="c7311-128">**Active Directory 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="c7311-128">**Active Directory subnet**.</span></span> <span data-ttu-id="c7311-129">AD DS 서버는 별도의 서브넷에 호스팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-129">The AD DS servers are hosted in a separate subnet.</span></span> <span data-ttu-id="c7311-130">NSG(네트워크 보안 그룹) 규칙은 AD DS 서버를 보호하고 예상치 못한 소스로부터의 트래픽에 대한 방화벽을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-130">Network security group (NSG) rules protect the AD DS servers and provide a firewall against traffic from unexpected sources.</span></span>
* <span data-ttu-id="c7311-131">**Azure 게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="c7311-131">**Azure gateway**.</span></span> <span data-ttu-id="c7311-132">Azure 게이트웨이는 온-프레미스 네트워크와 Azure VNet 간 연결을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-132">The Azure gateway provides a connection between the on-premises network and the Azure VNet.</span></span> <span data-ttu-id="c7311-133">[VPN 연결][azure-vpn-gateway] 또는 [Azure ExpressRoute][azure-expressroute]가 제공될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-133">This can be a [VPN connection][azure-vpn-gateway] or [Azure ExpressRoute][azure-expressroute].</span></span> <span data-ttu-id="c7311-134">자세한 내용은 [Azure에 안전한 하이브리드 보안 네트워크 아키텍처 구현][implementing-a-secure-hybrid-network-architecture]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-134">For more information, see [Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture].</span></span>

## <a name="recommendations"></a><span data-ttu-id="c7311-135">권장 사항</span><span class="sxs-lookup"><span data-stu-id="c7311-135">Recommendations</span></span>

<span data-ttu-id="c7311-136">Azure에서 Active Directory를 구현하는 방법에 대한 특정 권장 구성은 다음 문서를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-136">For specific recommendations on implementing Active Directory in Azure, see the following articles:</span></span>

- <span data-ttu-id="c7311-137">[AD DS(Active Directory Domain Services)를 Azure로 확장][adds-extend-domain]</span><span class="sxs-lookup"><span data-stu-id="c7311-137">[Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span> 
- <span data-ttu-id="c7311-138">[Azure Virtual Machines에 Windows Server Active Directory를 배포하기 위한 지침][ad-azure-guidelines]</span><span class="sxs-lookup"><span data-stu-id="c7311-138">[Guidelines for Deploying Windows Server Active Directory on Azure Virtual Machines][ad-azure-guidelines].</span></span>

### <a name="trust"></a><span data-ttu-id="c7311-139">신뢰</span><span class="sxs-lookup"><span data-stu-id="c7311-139">Trust</span></span>

<span data-ttu-id="c7311-140">온-프레미스 도메인은 클라우드에 있는 도메인과는 다른 포레스트 내에 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-140">The on-premises domains are contained within a different forest from the domains in the cloud.</span></span> <span data-ttu-id="c7311-141">클라우드에서 온-프레미스 사용자 인증을 하려면 Azure 도메인이 온-프레미스 포레스트의 로그인 도메인을 신뢰해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-141">To enable authentication of on-premises users in the cloud, the domains in Azure must trust the logon domain in the on-premises forest.</span></span> <span data-ttu-id="c7311-142">마찬가지로, 클라우드가 외부 사용자를 위한 로그인 도메인을 제공한다면 온-프레미스 포레스트가 클라우드 도메인을 신뢰해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-142">Similarly, if the cloud provides a logon domain for external users, it may be necessary for the on-premises forest to trust the cloud domain.</span></span>

<span data-ttu-id="c7311-143">포레스트 수준에서는 [포레스트 트러스트 생성][creating-forest-trusts]을 통해, 도메인 수준에서는 [외부 트러스트 생성][creating-external-trusts]을 통해 트러스트를 구축할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-143">You can establish trusts at the forest level by [creating forest trusts][creating-forest-trusts], or at the domain level by [creating external trusts][creating-external-trusts].</span></span> <span data-ttu-id="c7311-144">포레스트 수준 트러스트는 두 개의 포레스트 내에 있는 모든 도메인들 사이에 관계를 형성합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-144">A forest level trust creates a relationship between all domains in two forests.</span></span> <span data-ttu-id="c7311-145">외부 도메인 수준 트러스트는 두 개의 지정된 도메인 간 관계만을 형성합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-145">An external domain level trust only creates a relationship between two specified domains.</span></span> <span data-ttu-id="c7311-146">서로 다른 포레스트의 도메인 간에는 외부 도메인 수준 트러스트만 형성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-146">You should only create external domain level trusts between domains in different forests.</span></span>

<span data-ttu-id="c7311-147">트러스트는 단방향 또는 양방향으로 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-147">Trusts can be unidirectional (one-way) or bidirectional (two-way):</span></span>

* <span data-ttu-id="c7311-148">단방향 트러스트는 하나의 도메인 또는 포레스트(*수신* 도메인 또는 포리스트)의 사용자가 다른(*송신*) 도메인 또는 포레스트에 있는 리소스에 액세스할 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-148">A one-way trust enables users in one domain or forest (known as the *incoming* domain or forest) to access the resources held in another (the *outgoing* domain or forest).</span></span>
* <span data-ttu-id="c7311-149">양방향 트러스트는 어느 한 쪽의 도메인 또는 포레스트에 있는 사용자가 다른 쪽 도메인 또는 포레스트에 있는 리소스에 액세스할 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-149">A two-way trust enables users in either domain or forest to access resources held in the other.</span></span>

<span data-ttu-id="c7311-150">다음 표에는 간단한 시나리오에 대한 트러스트 구성이 요약되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-150">The following table summarizes trust configurations for some simple scenarios:</span></span>

| <span data-ttu-id="c7311-151">시나리오</span><span class="sxs-lookup"><span data-stu-id="c7311-151">Scenario</span></span> | <span data-ttu-id="c7311-152">온-프레미스 트러스트</span><span class="sxs-lookup"><span data-stu-id="c7311-152">On-premises trust</span></span> | <span data-ttu-id="c7311-153">클라우드 트러스트</span><span class="sxs-lookup"><span data-stu-id="c7311-153">Cloud trust</span></span> |
| --- | --- | --- |
| <span data-ttu-id="c7311-154">온-프레미스 사용자는 클라우드에 있는 리소스에 대한 액세스를 요구할 수 있지만 그 반대 방향은 불가능합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-154">On-premises users require access to resources in the cloud, but not vice versa</span></span> |<span data-ttu-id="c7311-155">단방향, 수신</span><span class="sxs-lookup"><span data-stu-id="c7311-155">One-way, incoming</span></span> |<span data-ttu-id="c7311-156">단방향, 송신</span><span class="sxs-lookup"><span data-stu-id="c7311-156">One-way, outgoing</span></span> |
| <span data-ttu-id="c7311-157">클라우드 사용자는 온-프레미스에 위치한 리소스에 대한 액세스를 요구할 수 있지만 그 반대 방향은 불가능합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-157">Users in the cloud require access to resources located on-premises, but not vice versa</span></span> |<span data-ttu-id="c7311-158">단방향, 송신</span><span class="sxs-lookup"><span data-stu-id="c7311-158">One-way, outgoing</span></span> |<span data-ttu-id="c7311-159">단방향, 수신</span><span class="sxs-lookup"><span data-stu-id="c7311-159">One-way, incoming</span></span> |
| <span data-ttu-id="c7311-160">클라우드와 온-프레미스 사용자 모두 클라우드와 온-프레미스에 있는 리소스에 대한 액세스를 요구할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-160">Users in the cloud and on-premises both requires access to resources held in the cloud and on-premises</span></span> |<span data-ttu-id="c7311-161">양방향, 수신 및 송신</span><span class="sxs-lookup"><span data-stu-id="c7311-161">Two-way, incoming and outgoing</span></span> |<span data-ttu-id="c7311-162">양방향, 수신 및 송신</span><span class="sxs-lookup"><span data-stu-id="c7311-162">Two-way, incoming and outgoing</span></span> |

## <a name="scalability-considerations"></a><span data-ttu-id="c7311-163">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="c7311-163">Scalability considerations</span></span>

<span data-ttu-id="c7311-164">Active Directory는 동일한 도메인의 일부인 도메인 컨트롤러를 위해 자동으로 확장 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-164">Active Directory is automatically scalable for domain controllers that are part of the same domain.</span></span> <span data-ttu-id="c7311-165">요청은 하나의 도메인 내에 있는 모든 컨트롤러로 분산됩니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-165">Requests are distributed across all controllers within a domain.</span></span> <span data-ttu-id="c7311-166">도메인 컨트롤러를 추가하면 자동으로 해당 도메인과 동기화가 진행됩니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-166">You can add another domain controller, and it synchronizes automatically with the domain.</span></span> <span data-ttu-id="c7311-167">해당 도메인 내에서 트래픽을 컨트롤러로 전송하기 위해 별도의 부하 분산 장치를 구성하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-167">Do not configure a separate load balancer to direct traffic to controllers within the domain.</span></span> <span data-ttu-id="c7311-168">모든 도메인 컨트롤러는 도메인 데이터베이스를 처리하기에 충부한 메모리와 저장소 리소스를 가져야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-168">Ensure that all domain controllers have sufficient memory and storage resources to handle the domain database.</span></span> <span data-ttu-id="c7311-169">모든 도메인 컨트롤러 VM은 동일한 크기여야 합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-169">Make all domain controller VMs the same size.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="c7311-170">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="c7311-170">Availability considerations</span></span>

<span data-ttu-id="c7311-171">각 도메인에 대해 최소 두 개의 도메인 컨트롤러를 프로비전합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-171">Provision at least two domain controllers for each domain.</span></span> <span data-ttu-id="c7311-172">이를 통해 서버 간 자동 복제가 가능해집니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-172">This enables automatic replication between servers.</span></span> <span data-ttu-id="c7311-173">각 도메인을 처리하는 Active Directory 서버의 역할을 하는 VM에 대한 가용성 집합을 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-173">Create an availability set for the VMs acting as Active Directory servers handling each domain.</span></span> <span data-ttu-id="c7311-174">이 가용성 집합 내에 최소 두 개의 서버를 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-174">Put at least two servers in this availability set.</span></span>

<span data-ttu-id="c7311-175">또한 유연한 단일 마스터 작업(FSMO) 역할을 수행하는 서버에 대한 연결 실패를 대비하여 각 도메인에 있는 하나 이상의 서버를 [대기 작업 마스터][standby-operations-masters]로 지정하는 것을 고려해 보세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-175">Also, consider designating one or more servers in each domain as [standby operations masters][standby-operations-masters] in case connectivity to a server acting as a flexible single master operation (FSMO) role fails.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="c7311-176">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="c7311-176">Manageability considerations</span></span>

<span data-ttu-id="c7311-177">관리 및 모니터링 고려사항에 대한 자세한 내용은 [Active Directory를 Azure로 확장][adds-extend-domain]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-177">For information about management and monitoring considerations, see [Extending Active Directory to Azure][adds-extend-domain].</span></span> 
 
<span data-ttu-id="c7311-178">자세한 내용은 [Active Directory 모니터링][monitoring_ad]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-178">For additional information, see [Monitoring Active Directory][monitoring_ad].</span></span> <span data-ttu-id="c7311-179">관리 서브넷의 모니터링 서버에 [Microsoft Systems Center][microsoft_systems_center]와 같은 도구를 설치하며 이러한 작업 수행이 수월해질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-179">You can install tools such as [Microsoft Systems Center][microsoft_systems_center] on a monitoring server in the management subnet to help perform these tasks.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="c7311-180">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="c7311-180">Security considerations</span></span>

<span data-ttu-id="c7311-181">포레스트 수준 트러스트는 전이적입니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-181">Forest level trusts are transitive.</span></span> <span data-ttu-id="c7311-182">온-프레미스 포레스트와 클라우드 포레스트 간 포레스트 수준 트러스트를 구축하는 경우 이 트러스트는 어느 하나의 포레스트에 생성된 다른 새로운 도메인로 확장됩니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-182">If you establish a forest level trust between an on-premises forest and a forest in the cloud, this trust is extended to other new domains created in either forest.</span></span> <span data-ttu-id="c7311-183">보안을 목적으로 도메인을 사용하여 분리를 제공하는 경우에는 도메인 수준에서만 트러스트를 생성하는 것을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-183">If you use domains to provide separation for security purposes, consider creating trusts at the domain level only.</span></span> <span data-ttu-id="c7311-184">도메인 수준 트러스트는 비전이적입니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-184">Domain level trusts are non-transitive.</span></span>

<span data-ttu-id="c7311-185">Active Directory 관련 보안 고려사항을 확인하려면 [Active Directory를 Azure로 확장][adds-extend-domain]의 보안 고려항 섹션을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-185">For Active Directory-specific security considerations, see the security considerations section in [Extending Active Directory to Azure][adds-extend-domain].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="c7311-186">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="c7311-186">Deploy the solution</span></span>

<span data-ttu-id="c7311-187">[GitHub][github]에서 이 참조 아키텍처를 배포할 수 있는 솔루션을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-187">A solution is available on [GitHub][github] to deploy this reference architecture.</span></span> <span data-ttu-id="c7311-188">솔루션을 배포하는 PowerShell 스크립트를 실행하려면 최신 버전의 Azure CLI가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-188">You will need the latest version of the Azure CLI to run the Powershell script that deploys the solution.</span></span> <span data-ttu-id="c7311-189">이 참조 아키텍처를 배포하려면 아래의 단계를 수행하세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-189">To deploy the reference architecture, follow these steps:</span></span>

1. <span data-ttu-id="c7311-190">[GitHub][github]의 해당 솔루션 폴더를 로컬 컴퓨터로 다운로드하거나 복제합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-190">Download or clone the solution folder from [GitHub][github] to your local machine.</span></span>

2. <span data-ttu-id="c7311-191">Azure CLI를 열고 로컬 솔루션 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-191">Open the Azure CLI and navigate to the local solution folder.</span></span>

3. <span data-ttu-id="c7311-192">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="c7311-192">Run the following command:</span></span>
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    <span data-ttu-id="c7311-193">`<subscription id>` 를 Azure 구독 ID로 바꿉니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-193">Replace `<subscription id>` with your Azure subscription ID.</span></span>
   
    <span data-ttu-id="c7311-194">`<location>`에서 Azure 지역(예: `eastus` 또는 `westus`)을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-194">For `<location>`, specify an Azure region, such as `eastus` or `westus`.</span></span>
   
    <span data-ttu-id="c7311-195">`<mode>` 매개 변수는 배포의 세분성을 제어합니다. 매개 변수는 다음 값 중 하나일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-195">The `<mode>` parameter controls the granularity of the deployment, and can be one of the following values:</span></span>
   
   * <span data-ttu-id="c7311-196">`Onpremise`: 시뮬레이션된 온-프레미스 환경을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-196">`Onpremise`: deploys the simulated on-premises environment.</span></span>
   * <span data-ttu-id="c7311-197">`Infrastructure`: Azure에 VNet 인프라 및 jumpbox를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-197">`Infrastructure`: deploys the VNet infrastructure and jump box in Azure.</span></span>
   * <span data-ttu-id="c7311-198">`CreateVpn`: Azure 가상 네트워크 게이트웨이를 배포하고 시뮬레이트된 온-프레미스 네트워크에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-198">`CreateVpn`: deploys the Azure virtual network gateway and connects it to the simulated on-premises network.</span></span>
   * <span data-ttu-id="c7311-199">`AzureADDS`: Active Directory DS 서버 역할을 하는 VM을 배포하고, 이러한 VM에 Active Directory를 배포하고, Azure에서 도메인을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-199">`AzureADDS`: deploys the VMs acting as Active Directory DS servers, deploys Active Directory to these VMs, and deploys the domain in Azure.</span></span>
   * <span data-ttu-id="c7311-200">`WebTier`: 웹 계층 VM 및 부하 분산 장치를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-200">`WebTier`: deploys the web tier VMs and load balancer.</span></span>
   * <span data-ttu-id="c7311-201">`Prepare`: 모든 이전 배포를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-201">`Prepare`: deploys all of the preceding deployments.</span></span> <span data-ttu-id="c7311-202">**기존의 온-프레미스 네트워크가 없지만 테스트나 평가를 위해 위의 완전한 참조 아키텍처를 배포하려는 경우를 위한 추천 옵션입니다.**</span><span class="sxs-lookup"><span data-stu-id="c7311-202">**This is the recommended option if If you do not have an existing on-premises network but you want to deploy the complete reference architecture described above for testing or evaluation.**</span></span> 
   * <span data-ttu-id="c7311-203">`Workload`: 비즈니스 및 데이터 계층 VM과 부하 분산 장치를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-203">`Workload`: deploys the business and data tier VMs and load balancers.</span></span> <span data-ttu-id="c7311-204">이 VM은 `Prepare` 배포에는 포함되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-204">Note that these VMs are not included in the `Prepare` deployment.</span></span>

4. <span data-ttu-id="c7311-205">배포가 완료될 때가지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-205">Wait for the deployment to complete.</span></span> <span data-ttu-id="c7311-206">`Prepare` 배포를 배포하는 데는 몇 시간 정도 걸립니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-206">If you are deploying the `Prepare` deployment, it will take several hours.</span></span>
     
5. <span data-ttu-id="c7311-207">5.시뮬레이션된 온-프레미스 구성을 사용하는 경우에는 수신 트러스트 관계를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-207">If you are using the simulated on-premises configuration, configure the incoming trust relationship:</span></span>
   
   1. <span data-ttu-id="c7311-208">점프 박스(<em>ra-adtrust-security-rg</em> 리소스 그룹의 <em>ra-adtrust-mgmt-vm1</em>)에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-208">Connect to the jump box (<em>ra-adtrust-mgmt-vm1</em> in the <em>ra-adtrust-security-rg</em> resource group).</span></span> <span data-ttu-id="c7311-209"><em>AweS0me@PW</em> 암호를 사용해 <em>testuser</em>로 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-209">Log in as <em>testuser</em> with password <em>AweS0me@PW</em>.</span></span>
   2. <span data-ttu-id="c7311-210">점프 박스에서 <em>contoso.com</em> 도메인(온-프레미스 도메인) 내 첫 번째 VM에서 RDP 세션을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-210">On the jump box open an RDP session on the first VM in the <em>contoso.com</em> domain (the on-premises domain).</span></span> <span data-ttu-id="c7311-211">이 VM의 IP 주소는 192.168.0.4입니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-211">This VM has the IP address 192.168.0.4.</span></span> <span data-ttu-id="c7311-212">사용자 이름은 <em>contoso\testuser</em>이고 암호는 <em>AweS0me@PW</em>입니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-212">The username is <em>contoso\testuser</em> with password <em>AweS0me@PW</em>.</span></span>
   3. <span data-ttu-id="c7311-213">[incoming-trust.ps1][incoming-trust] 스크립트를 다운로드 및 실행하여 *treyresearch.com* 도메인으로부터 수신 트러스트를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-213">Download the [incoming-trust.ps1][incoming-trust] script and run it to create the incoming trust from the *treyresearch.com* domain.</span></span>

6. <span data-ttu-id="c7311-214">자신의 온-프레미스 인프라를 사용한다면</span><span class="sxs-lookup"><span data-stu-id="c7311-214">If you are using your own on-premises infrastructure:</span></span>
   
   1. <span data-ttu-id="c7311-215">[incoming-trust.ps1][incoming-trust] 스크립트를 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-215">Download the [incoming-trust.ps1][incoming-trust] script.</span></span>
   2. <span data-ttu-id="c7311-216">스크립트를 수정하고 변수 `$TrustedDomainName`의 값을 귀하의 도메인 이름으로 대체합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-216">Edit the script and replace the value of the `$TrustedDomainName` variable with the name of your own domain.</span></span>
   3. <span data-ttu-id="c7311-217">스크립트를 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-217">Run the script.</span></span>

7. <span data-ttu-id="c7311-218">점프 박스로부터 <em>treyresearch.com</em> 도메인(클라우드 도메인)의 첫 번째 VM에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-218">From the jump-box, connect to the first VM in the <em>treyresearch.com</em> domain (the domain in the cloud).</span></span> <span data-ttu-id="c7311-219">이 VM의 IP 주소는 10.0.4.4입니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-219">This VM has the IP address 10.0.4.4.</span></span> <span data-ttu-id="c7311-220">사용자 이름은 <em>treyresearch\testuser</em>이고 암호는 <em>AweS0me@PW</em>입니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-220">The username is <em>treyresearch\testuser</em> with password <em>AweS0me@PW</em>.</span></span>

8. <span data-ttu-id="c7311-221">[incoming-trust.ps1][outgoing-trust] 스크립트를 다운로드 및 실행하여 *treyresearch.com* 도메인으로부터 송신 트러스트를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-221">Download the [outgoing-trust.ps1][outgoing-trust] script and run it to create the incoming trust from the *treyresearch.com* domain.</span></span> <span data-ttu-id="c7311-222">자신의 온-프레미스 컴퓨터를 사용한다면, 우선 이 스크립트를 수정합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-222">If you are using your own on-premises machines, then edit the script first.</span></span> <span data-ttu-id="c7311-223">변수 `$TrustedDomainName`을 귀하의 온-프레미스 도메인 이름으로 설정하고, 변수 `$TrustedDomainDnsIpAddresses`에 이 도메인에 대한 Active Directory DS 서버의 IP 주소를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-223">Set the `$TrustedDomainName` variable to the name of your on-premises domain, and specify the IP addresses of the Active Directory DS servers for this domain in the `$TrustedDomainDnsIpAddresses` variable.</span></span>

9. <span data-ttu-id="c7311-224">4.이전 단계가 완료될 때까지 몇 분간 기다린 다음 온-프레미스 VM에 접속하여 [트러스트 확인][verify-a-trust] 문서에 설명된 절차를 수행하여 *contoso.com*과 *treyresearch.com* 도메인 간 트러스트 관계가 정확히 설정되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="c7311-224">Wait a few minutes for the previous steps to complete, then connect to an on-premises VM and perform the steps outlined in the article [Verify a Trust][verify-a-trust] to determine whether the trust relationship between the *contoso.com* and *treyresearch.com* domains is correctly configured.</span></span>

## <a name="next-steps"></a><span data-ttu-id="c7311-225">다음 단계</span><span class="sxs-lookup"><span data-stu-id="c7311-225">Next steps</span></span>

* <span data-ttu-id="c7311-226">[온-프레미스 AD DS 도메인을 Azure로 확장][adds-extend-domain]하기 위한 모범 사례를 살펴보세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-226">Learn the best practices for [extending your on-premises AD DS domain to Azure][adds-extend-domain]</span></span>
* <span data-ttu-id="c7311-227">Azure에서 [AD FS 인프라를 생성][adfs]하기 위한 모범 사례를 살펴보세요.</span><span class="sxs-lookup"><span data-stu-id="c7311-227">Learn the best practices for [creating an AD FS infrastructure][adfs] in Azure.</span></span>

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "별도 Active Directory 도메인으로 하이브리드 네트워크 아키텍처 보안 유지"