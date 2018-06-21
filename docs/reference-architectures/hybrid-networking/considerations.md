---
title: 온-프레미스 네트워크를 Azure에 연결하기 위한 솔루션 선택
description: 온-프레미스 네트워크를 Azure에 연결하기 위한 참조 아키텍처를 비교합니다.
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
ms.locfileid: "26582728"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="34d6e-103">온-프레미스 네트워크를 Azure에 연결하기 위한 솔루션 선택</span><span class="sxs-lookup"><span data-stu-id="34d6e-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="34d6e-104">이 문서에서는 온-프레미스 네트워크를 Azure VNet(Virtual Network)에 연결하는 옵션을 비교합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="34d6e-105">각 옵션에 대해 참조 아키텍처와 배포 가능한 솔루션을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-105">We provide a reference architecture and a deployable solution for each option.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="34d6e-106">VPN 연결</span><span class="sxs-lookup"><span data-stu-id="34d6e-106">VPN connection</span></span>

<span data-ttu-id="34d6e-107">IPSec VPN 터널을 통해 Azure VNet과 온-프레미스 네트워크를 연결하기 위해 VPN(가상 사설망)을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-107">Use a virtual private network (VPN) to connect your on-premises network with an Azure VNet through an IPSec VPN tunnel.</span></span>

<span data-ttu-id="34d6e-108">이 아키텍처는 온-프레미스 하드웨어와 클라우드 간의 트래픽이 가벼울 가능성이 높은 하이브리드 응용 프로그램에 적합하거나, 클라우드의 유연성 및 처리 능력을 위해 대기 시간을 약간 연장하고자 하는 경우에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-108">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="34d6e-109">**이점**</span><span class="sxs-lookup"><span data-stu-id="34d6e-109">**Benefits**</span></span>

- <span data-ttu-id="34d6e-110">구성이 간단합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-110">Simple to configure.</span></span>

<span data-ttu-id="34d6e-111">**과제**</span><span class="sxs-lookup"><span data-stu-id="34d6e-111">**Challenges**</span></span>

- <span data-ttu-id="34d6e-112">온-프레미스 VPN 장치가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-112">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="34d6e-113">Microsoft에서 각 VPN Gateway에 99.9%의 가용성을 보장하지만, 이 SLA는 게이트웨이에 대한 네트워크 연결이 아닌 VPN Gateway만 다룹니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-113">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this SLA only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="34d6e-114">현재 Azure VPN Gateway를 통한 VPN 연결은 최대 200Mbps의 대역폭을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-114">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="34d6e-115">이러한 처리량을 초과할 것으로 예상되는 경우 여러 VPN 연결에 걸쳐 Azure 가상 네트워크를 분할해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-115">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="34d6e-116">**[자세히 알아보기...][vpn]**</span><span class="sxs-lookup"><span data-stu-id="34d6e-116">**[Read more...][vpn]**</span></span>

## <a name="azure-expressroute-connection"></a><span data-ttu-id="34d6e-117">Azure ExpressRoute 연결</span><span class="sxs-lookup"><span data-stu-id="34d6e-117">Azure ExpressRoute connection</span></span>

<span data-ttu-id="34d6e-118">ExpressRoute 연결은 타사 연결 공급자를 통해 개인 전용 연결을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-118">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="34d6e-119">개인 연결에서 온-프레미스 네트워크를 Azure로 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-119">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="34d6e-120">이 아키텍처는 높은 수준의 확장성이 필요한 대규모 중요 업무 워크로드를 실행하는 하이브리드 응용 프로그램에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-120">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="34d6e-121">**이점**</span><span class="sxs-lookup"><span data-stu-id="34d6e-121">**Benefits**</span></span>

- <span data-ttu-id="34d6e-122">연결 공급자에 따라 훨씬 더 높은 대역폭(최대 10Gbps)을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-122">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="34d6e-123">동적 대역폭 확장을 지원하여 수요가 더 낮은 기간 동안 비용을 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-123">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="34d6e-124">그러나 모든 연결 공급자에 이 옵션이 있는 것은 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-124">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="34d6e-125">연결 공급자에 따라 조직에서 국가 클라우드에 직접 액세스할 수 있게 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-125">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="34d6e-126">전체 연결에서 99.9%의 가용성 SLA를 실현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-126">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="34d6e-127">**과제**</span><span class="sxs-lookup"><span data-stu-id="34d6e-127">**Challenges**</span></span>

- <span data-ttu-id="34d6e-128">설정이 복잡할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-128">Can be complex to set up.</span></span> <span data-ttu-id="34d6e-129">ExpressRoute 연결을 만들려면 타사 연결 공급자를 함께 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-129">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="34d6e-130">해당 공급자가 네트워크 연결 프로비저닝을 담당합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-130">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="34d6e-131">온-프레미스에 높은 대역폭의 라우터가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-131">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="34d6e-132">**[자세히 알아보기...][expressroute]**</span><span class="sxs-lookup"><span data-stu-id="34d6e-132">**[Read more...][expressroute]**</span></span>

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="34d6e-133">VPN 장애 조치(failover)를 사용하는 ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="34d6e-133">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="34d6e-134">이 옵션은 앞에 나온 두 가지 옵션을 결합합니다. 정상적인 조건에서는 ExpressRoute를 사용하고 ExpressRoute 회로에서 연결이 끊기면 VPN 연결로 장애 조치(failover)합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-134">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="34d6e-135">이 아키텍처는 더 높은 ExpressRoute 대역폭도 필요하고 고가용성 네트워크 연결도 필요한 하이브리드 응용 프로그램에 적합합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-135">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="34d6e-136">**이점**</span><span class="sxs-lookup"><span data-stu-id="34d6e-136">**Benefits**</span></span>

- <span data-ttu-id="34d6e-137">ExpressRoute 회로가 고장나는 경우 대체 연결이 더 낮은 대역폭 네트워크에 있더라도 고가용성입니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-137">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="34d6e-138">**과제**</span><span class="sxs-lookup"><span data-stu-id="34d6e-138">**Challenges**</span></span>

- <span data-ttu-id="34d6e-139">구성이 복잡합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-139">Complex to configure.</span></span> <span data-ttu-id="34d6e-140">VPN 연결과 ExpressRoute 회로를 모두 설정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-140">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="34d6e-141">중복 하드웨어(VPN 어플라이언스)와 중복 Azure VPN Gateway 연결이 필요하며 이는 유료입니다.</span><span class="sxs-lookup"><span data-stu-id="34d6e-141">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="34d6e-142">**[자세히 알아보기...][expressroute-vpn-failover]**</span><span class="sxs-lookup"><span data-stu-id="34d6e-142">**[Read more...][expressroute-vpn-failover]**</span></span>

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md