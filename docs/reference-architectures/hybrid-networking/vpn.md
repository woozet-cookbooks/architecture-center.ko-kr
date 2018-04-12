---
title: VPN을 사용하여 온-프레미스 네트워크를 Azure에 연결
description: VPN을 사용하여 연결된 온-프레미스 네트워크 및 Azure Virtual Network를 포괄하는 보안 사이트 간 네트워크 아키텍처를 구현하는 방법입니다.
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: dafcee6607d9cc7c56c332f9ed5d9568ff70f0e7
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/30/2018
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a><span data-ttu-id="0c0ae-103">VPN 게이트웨이를 사용하여 온-프레미스 네트워크를 Azure에 연결</span><span class="sxs-lookup"><span data-stu-id="0c0ae-103">Connect an on-premises network to Azure using a VPN gateway</span></span>

<span data-ttu-id="0c0ae-104">이 참조 아키텍처에서는 사이트 간 VPN(가상 사설 네트워크)을 사용하여 온-프레미스 네트워크를 Azure로 확장하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-104">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span> <span data-ttu-id="0c0ae-105">IPSec VPN 터널을 통해 온-프레미스 네트워크와 Azure 가상 네트워크(VNet) 사이에서 트래픽이 흐릅니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-105">Traffic flows between the on-premises network and an Azure Virtual Network (VNet) through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="0c0ae-106">**이 솔루션을 배포합니다**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="0c0ae-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="0c0ae-107">![[0]][0]</span></span>

<span data-ttu-id="0c0ae-108">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="0c0ae-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="0c0ae-109">건축</span><span class="sxs-lookup"><span data-stu-id="0c0ae-109">Architecture</span></span> 

<span data-ttu-id="0c0ae-110">이 아키텍처는 다음 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-110">The architecture consists of the following components.</span></span>

* <span data-ttu-id="0c0ae-111">**온-프레미스 네트워크**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-111">**On-premises network**.</span></span> <span data-ttu-id="0c0ae-112">조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-112">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="0c0ae-113">**VPN 어플라이언스**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-113">**VPN appliance**.</span></span> <span data-ttu-id="0c0ae-114">온-프레미스 네트워크에 외부 연결을 제공하는 장치 또는 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-114">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="0c0ae-115">VPN 어플라이언스는 하드웨어 장치일 수도 있고 Windows Server 2012의 RRAS(라우팅 및 원격 액세스 서비스)와 같은 소프트웨어 솔루션일 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-115">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="0c0ae-116">지원되는 VPN 어플라이언스 목록 및 VPN 어플라이언스가 Azure VPN 게이트웨이에 연결되도록 구성하는 방법은 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치 정보][vpn-appliance] 문서에서 선택한 장치에 대한 지침을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-116">For a list of supported VPN appliances and information on configuring them to connect to an Azure VPN gateway, see the instructions for the selected device in the article [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="0c0ae-117">**가상 네트워크(VNet)**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-117">**Virtual network (VNet)**.</span></span> <span data-ttu-id="0c0ae-118">Azure VPN 게이트웨이의 클라우드 응용 프로그램과 구성 요소는 동일한 [VNet][azure-virtual-network]에 존재합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-118">The cloud application and the components for the Azure VPN gateway reside in the same [VNet][azure-virtual-network].</span></span>

* <span data-ttu-id="0c0ae-119">**Azure VPN 게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-119">**Azure VPN gateway**.</span></span> <span data-ttu-id="0c0ae-120">[VPN 게이트웨이][azure-vpn-gateway] 서비스를 사용하면 VPN 어플라이언스를 통해 VNet을 온-프레미스 네트워크에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-120">The [VPN gateway][azure-vpn-gateway] service enables you to connect the VNet to the on-premises network through a VPN appliance.</span></span> <span data-ttu-id="0c0ae-121">자세한 내용은 [온-프레미스 네트워크를 Microsoft Azure virtual network에 연결][connect-to-an-Azure-vnet]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-121">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span> <span data-ttu-id="0c0ae-122">VPN 게이트웨이에는 다음과 같은 요소가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-122">The VPN gateway includes the following elements:</span></span>
  
  * <span data-ttu-id="0c0ae-123">**가상 네트워크 게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-123">**Virtual network gateway**.</span></span> <span data-ttu-id="0c0ae-124">VNet을 위한 가상 VPN 어플라이언스를 제공하는 리소스입니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-124">A resource that provides a virtual VPN appliance for the VNet.</span></span> <span data-ttu-id="0c0ae-125">온-프레미스 네트워크에서 VNet으로 트래픽을 라우팅하는 역할을 담당합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-125">It is responsible for routing traffic from the on-premises network to the VNet.</span></span>
  * <span data-ttu-id="0c0ae-126">**로컬 네트워크 게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-126">**Local network gateway**.</span></span> <span data-ttu-id="0c0ae-127">온-프레미스 VPN 어플라이언스가 추상화된 것입니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-127">An abstraction of the on-premises VPN appliance.</span></span> <span data-ttu-id="0c0ae-128">클라우드 응용 프로그램에서 온-프레미스 네트워크로 흐르는 네트워크 트래픽은 이 게이트웨이를 통과하도록 라우팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-128">Network traffic from the cloud application to the on-premises network is routed through this gateway.</span></span>
  * <span data-ttu-id="0c0ae-129">**연결**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-129">**Connection**.</span></span> <span data-ttu-id="0c0ae-130">이 연결에는 연결 유형(IPSec) 및 트래픽을 암호화하기 위해 온-프레미스 VPN 어플라이언스와 공유되는 키를 지정하는 속성이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>
  * <span data-ttu-id="0c0ae-131">**게이트웨이 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-131">**Gateway subnet**.</span></span> <span data-ttu-id="0c0ae-132">가상 네트워크 게이트웨이는 자체 서브넷에 존재합니다. 자체 서브넷은 아래의 권장 사항 섹션에서 설명하는 다양한 요구 사항에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-132">The virtual network gateway is held in its own subnet, which is subject to various requirements, described in the Recommendations section below.</span></span>

* <span data-ttu-id="0c0ae-133">**클라우드 응용 프로그램**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-133">**Cloud application**.</span></span> <span data-ttu-id="0c0ae-134">Azure에 호스팅된 응용 프로그램입니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-134">The application hosted in Azure.</span></span> <span data-ttu-id="0c0ae-135">Azure Load Balancer를 통해 여러 서브넷이 연결된 여러 계층이 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-135">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="0c0ae-136">응용 프로그램 인프라에 대한 자세한 내용은 [Windows VM 워크로드 실행][windows-vm-ra] 및 [Linux VM 워크로드 실행][linux-vm-ra]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-136">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="0c0ae-137">**내부 부하 분산 장치**.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-137">**Internal load balancer**.</span></span> <span data-ttu-id="0c0ae-138">VPN 게이트웨이에서 전송되는 네트워크 트래픽은 내부 부하 분산 장치를 통해 클라우드 응용 프로그램으로 라우팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-138">Network traffic from the VPN gateway is routed to the cloud application through an internal load balancer.</span></span> <span data-ttu-id="0c0ae-139">부하 분산 장치는 응용 프로그램의 프론트 엔드 서브넷에 위치합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-139">The load balancer is located in the front-end subnet of the application.</span></span>

## <a name="recommendations"></a><span data-ttu-id="0c0ae-140">권장 사항</span><span class="sxs-lookup"><span data-stu-id="0c0ae-140">Recommendations</span></span>

<span data-ttu-id="0c0ae-141">대부분의 시나리오의 경우 다음 권장 사항을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="0c0ae-142">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gateway-subnet"></a><span data-ttu-id="0c0ae-143">VNet과 게이트웨이 서브넷</span><span class="sxs-lookup"><span data-stu-id="0c0ae-143">VNet and gateway subnet</span></span>

<span data-ttu-id="0c0ae-144">필요한 리소스를 모두 수용할 수 있을 만큼 충분한 주소 공간을 사용하여 Azure VNet을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-144">Create an Azure VNet with an address space large enough for all of your required resources.</span></span> <span data-ttu-id="0c0ae-145">추후 더 많은 VM이 필요할 경우 여분이 충분하도록 VNet 주소 공간을 할당합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-145">Ensure that the VNet address space has sufficient room for growth if additional VMs are likely to be needed in the future.</span></span> <span data-ttu-id="0c0ae-146">VNet 주소 공간은 온-프레미스 네트워크와 겹치지 않아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-146">The address space of the VNet must not overlap with the on-premises network.</span></span> <span data-ttu-id="0c0ae-147">예를 들어 위 다이어그램에서는 VNet에 10.20.0.0/16의 주소 공간을 사용하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-147">For example, the diagram above uses the address space 10.20.0.0/16 for the VNet.</span></span>

<span data-ttu-id="0c0ae-148">이름이 *GatewaySubnet*인 서브넷을 만듭니다. 주소 범위는 /27로 합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-148">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="0c0ae-149">이 서브넷은 가상 네트워크 게이트웨이에서 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-149">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="0c0ae-150">이 서브넷에 주소 32개를 할당하면 추후 게이트웨이 크기 제한에 도달하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-150">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span> <span data-ttu-id="0c0ae-151">주소 공간 가운데에 이 서브넷을 배치하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-151">Also, avoid placing this subnet in the middle of the address space.</span></span> <span data-ttu-id="0c0ae-152">게이트웨이 서브넷을 VNet 주소 공간의 상단 끝에 배치하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-152">A good practice is to set the address space for the gateway subnet at the upper end of the VNet address space.</span></span> <span data-ttu-id="0c0ae-153">다이어그램에서는 10.20.255.224/27을 사용하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-153">The example shown in the diagram uses 10.20.255.224/27.</span></span>  <span data-ttu-id="0c0ae-154">다음은 [CIDR]을 간단하게 계산하는 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-154">Here is a quick procedure to calculate the [CIDR]:</span></span>

1. <span data-ttu-id="0c0ae-155">VNet 주소 공간의 변수 비트를 게이트웨이 서브넷에 의해 사용되는 비트까지 모두 1로 설정하고, 나머지 비트를 0으로 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-155">Set the variable bits in the address space of the VNet to 1, up to the bits being used by the gateway subnet, then set the remaining bits to 0.</span></span>
2. <span data-ttu-id="0c0ae-156">결과 비트를 10진수로 변환한 다음 이것을 게이트웨이 서브넷의 크기로 설정된 접두사 길이를 사용하여 주소 공간으로 표현합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-156">Convert the resulting bits to decimal and express it as an address space with the prefix length set to the size of the gateway subnet.</span></span>

<span data-ttu-id="0c0ae-157">예를 들어 IP 주소 범위가 10.20.0.0/16인 VNet에 1단계를 적용하면 10.20.0b11111111.0b11100000이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-157">For example, for a VNet with an IP address range of 10.20.0.0/16, applying step #1 above becomes 10.20.0b11111111.0b11100000.</span></span>  <span data-ttu-id="0c0ae-158">이것을 10진수로 변환한 다음 주소 공간으로 표현하면 10.20.255.224/27이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-158">Converting that to decimal and expressing it as an address space yields 10.20.255.224/27.</span></span> 

> [!WARNING]
> <span data-ttu-id="0c0ae-159">게이트웨이 서브넷에는 VM을 배포하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-159">Do not deploy any VMs to the gateway subnet.</span></span> <span data-ttu-id="0c0ae-160">또한, 이 서브넷에 NSG을 할당하지 않습니다. NSG를 할당하면 게이트웨이의 작동이 중지됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-160">Also, do not assign an NSG to this subnet, as it will cause the gateway to stop functioning.</span></span>
> 
> 

### <a name="virtual-network-gateway"></a><span data-ttu-id="0c0ae-161">가상 네트워크 게이트웨이</span><span class="sxs-lookup"><span data-stu-id="0c0ae-161">Virtual network gateway</span></span>

<span data-ttu-id="0c0ae-162">가상 네트워크 게이트웨이에 공용 IP 주소를 할당합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-162">Allocate a public IP address for the virtual network gateway.</span></span>

<span data-ttu-id="0c0ae-163">게이트웨이 서브넷에 가상 네트워크 게이트웨이를 만든 다음 이것을 새로 할당된 공용 IP 주소에 할당합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-163">Create the virtual network gateway in the gateway subnet and assign it the newly allocated public IP address.</span></span> <span data-ttu-id="0c0ae-164">사용자 요구 사항에 가장 가까우면서 사용자의 VPN 어플라이언스가 지원하는 게이트웨이 유형을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-164">Use the gateway type that most closely matches your requirements and that is enabled by your VPN appliance:</span></span>

- <span data-ttu-id="0c0ae-165">주소 접두사와 같은 정책 조건을 바탕으로 요청이 라우팅되는 방식을 제어해야 하는 경우 [policy-based gateway][policy-based-routing]를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-165">Create a [policy-based gateway][policy-based-routing] if you need to closely control how requests are routed based on policy criteria such as address prefixes.</span></span> <span data-ttu-id="0c0ae-166">정책 기반 게이트웨이는 정적 라우팅을 사용하며, 사이트 간 연결에서만 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-166">Policy-based gateways use static routing, and only work with site-to-site connections.</span></span>

- <span data-ttu-id="0c0ae-167">RRAS를 사용하여 온-프레미스 네트워크에 연결하거나 다중 사이트 또는 교차 지역 연결을 지원하는 경우, [route-based gateway][route-based-routing]를 만들거나 VNet 간 연결을 구현합니다(복수의 VNet을 이동하는 경로 포함).</span><span class="sxs-lookup"><span data-stu-id="0c0ae-167">Create a [route-based gateway][route-based-routing] if you connect to the on-premises network using RRAS, support multi-site or cross-region connections, or implement VNet-to-VNet connections (including routes that traverse multiple VNets).</span></span> <span data-ttu-id="0c0ae-168">경로 기반 게이트웨이는 네트워크 간에 트래픽을 전달할 때 동적 라우팅을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-168">Route-based gateways use dynamic routing to direct traffic between networks.</span></span> <span data-ttu-id="0c0ae-169">동적 라우팅은 대체 경로를 시도하므로 정적 라우팅보다 네트워크 경로 장애에 대한 내결함성이 높습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-169">They can tolerate failures in the network path better than static routes because they can try alternative routes.</span></span> <span data-ttu-id="0c0ae-170">경로 기반 게이트웨이에서는 네트워크 주소가 변경되어도 경로를 수동으로 업데이트할 필요가 없기 때문에 관리 오버헤드를 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-170">Route-based gateways can also reduce the management overhead because routes might not need to be updated manually when network addresses change.</span></span>

<span data-ttu-id="0c0ae-171">지원되는 VPN 어플라이언스 목록은 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치 정보][vpn-appliances]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-171">For a list of supported VPN appliances, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliances].</span></span>

> [!NOTE]
> <span data-ttu-id="0c0ae-172">게이트웨이를 만든 뒤에는 게이트웨이 유형을 변경하려면 게이트웨이를 삭제하고 다시 만들어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-172">After the gateway has been created, you cannot change between gateway types without deleting and re-creating the gateway.</span></span>
> 
> 

<span data-ttu-id="0c0ae-173">사용자의 처리량 요구 사항에 가장 가까운 Azure VPN 게이트웨이 SKU를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-173">Select the Azure VPN gateway SKU that most closely matches your throughput requirements.</span></span> <span data-ttu-id="0c0ae-174">Azure VPN 게이트웨이는 아래 표에 제시된 것처럼 세 가지 SKU로 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-174">Azure VPN gateway is available in three SKUs shown in the following table.</span></span> 

| <span data-ttu-id="0c0ae-175">SKU</span><span class="sxs-lookup"><span data-stu-id="0c0ae-175">SKU</span></span> | <span data-ttu-id="0c0ae-176">VPN 처리량</span><span class="sxs-lookup"><span data-stu-id="0c0ae-176">VPN Throughput</span></span> | <span data-ttu-id="0c0ae-177">최대 IPSec 터널</span><span class="sxs-lookup"><span data-stu-id="0c0ae-177">Max IPSec Tunnels</span></span> |
| --- | --- | --- |
| <span data-ttu-id="0c0ae-178">Basic</span><span class="sxs-lookup"><span data-stu-id="0c0ae-178">Basic</span></span> |<span data-ttu-id="0c0ae-179">100Mbps</span><span class="sxs-lookup"><span data-stu-id="0c0ae-179">100 Mbps</span></span> |<span data-ttu-id="0c0ae-180">10</span><span class="sxs-lookup"><span data-stu-id="0c0ae-180">10</span></span> |
| <span data-ttu-id="0c0ae-181">Standard</span><span class="sxs-lookup"><span data-stu-id="0c0ae-181">Standard</span></span> |<span data-ttu-id="0c0ae-182">100Mbps</span><span class="sxs-lookup"><span data-stu-id="0c0ae-182">100 Mbps</span></span> |<span data-ttu-id="0c0ae-183">10</span><span class="sxs-lookup"><span data-stu-id="0c0ae-183">10</span></span> |
| <span data-ttu-id="0c0ae-184">고성능</span><span class="sxs-lookup"><span data-stu-id="0c0ae-184">High Performance</span></span> |<span data-ttu-id="0c0ae-185">200Mbps</span><span class="sxs-lookup"><span data-stu-id="0c0ae-185">200 Mbps</span></span> |<span data-ttu-id="0c0ae-186">30</span><span class="sxs-lookup"><span data-stu-id="0c0ae-186">30</span></span> |

> [!NOTE]
> <span data-ttu-id="0c0ae-187">기본 SKU는 Azure ExpressRoute와 호환되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-187">The Basic SKU is not compatible with Azure ExpressRoute.</span></span> <span data-ttu-id="0c0ae-188">게이트웨이를 만든 뒤에 [SKU를 변경][changing-SKUs]할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-188">You can [change the SKU][changing-SKUs] after the gateway has been created.</span></span>
> 
> 

<span data-ttu-id="0c0ae-189">요금은 게이트웨이가 프로비전되고 사용된 시간에 따라 청구됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-189">You are charged based on the amount of time that the gateway is provisioned and available.</span></span> <span data-ttu-id="0c0ae-190">[VPN 게이트웨이 가격][azure-gateway-charges]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-190">See [VPN Gateway Pricing][azure-gateway-charges].</span></span>

<span data-ttu-id="0c0ae-191">요청을 응용 프로그램 VM으로 직접 전달하는 대신 게이트웨이로부터 수신되는 응용 프로그램 트래픽을 내부 부하 분산 장치로 전달하는 게이트웨이 서브넷을 위한 라우팅 규칙을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-191">Create routing rules for the gateway subnet that direct incoming application traffic from the gateway to the internal load balancer, rather than allowing requests to pass directly to the application VMs.</span></span>

### <a name="on-premises-network-connection"></a><span data-ttu-id="0c0ae-192">온-프레미스 네트워크 연결</span><span class="sxs-lookup"><span data-stu-id="0c0ae-192">On-premises network connection</span></span>

<span data-ttu-id="0c0ae-193">로컬 네트워크 게이트웨이를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-193">Create a local network gateway.</span></span> <span data-ttu-id="0c0ae-194">온-프레미스 VPN 어플라이언스의 공용 IP 주소와 온-프레미스 네트워크의 주소 공간을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-194">Specify the public IP address of the on-premises VPN appliance, and the address space of the on-premises network.</span></span> <span data-ttu-id="0c0ae-195">온-프레미스 VPN 어플라이언스는 Azure VPN 게이트웨이에서 로컬 네트워크 게이트웨이에 의해 액세스할 수 있는 공용 IP 주소를 가져야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-195">Note that the on-premises VPN appliance must have a public IP address that can be accessed by the local network gateway in Azure VPN Gateway.</span></span> <span data-ttu-id="0c0ae-196">VPN 장치는 NAT(Network Address Translator) 뒤에 배치될 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-196">The VPN device cannot be located behind a network address translation (NAT) device.</span></span>

<span data-ttu-id="0c0ae-197">가상 네트워크 게이트웨이와 로컬 네트워크 게이트웨이의 사이트 간 연결을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-197">Create a site-to-site connection for the virtual network gateway and the local network gateway.</span></span> <span data-ttu-id="0c0ae-198">사이트 간(IPSec) 연결 유형을 선택하고 공유 키를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-198">Select the site-to-site (IPSec) connection type, and specify the shared key.</span></span> <span data-ttu-id="0c0ae-199">Azure VPN 게이트웨이를 사용하는 사이트 간 암호는 IPSec 프로토콜을 기반으로 하며, 인증을 위해 사전 공유된 키를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-199">Site-to-site encryption with the Azure VPN gateway is based on the IPSec protocol, using preshared keys for authentication.</span></span> <span data-ttu-id="0c0ae-200">사전 공유된 키는 Azure VPN 게이트웨이를 만들 때 사용자가 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-200">You specify the key when you create the Azure VPN gateway.</span></span> <span data-ttu-id="0c0ae-201">온-프레미스에서 실행되는 VPN 어플라이언스도 동일한 키로 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-201">You must configure the VPN appliance running on-premises with the same key.</span></span> <span data-ttu-id="0c0ae-202">현재 다른 인증 메커니즘은 지원되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-202">Other authentication mechanisms are not currently supported.</span></span>

<span data-ttu-id="0c0ae-203">목적지가 Azure VNet에 속한 주소인 요청이 VPN 장치로 전달되도록 온-프레미스 라우팅 인프라가 구성되어 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-203">Ensure that the on-premises routing infrastructure is configured to forward requests intended for addresses in the Azure VNet to the VPN device.</span></span>

<span data-ttu-id="0c0ae-204">온-프레미스 네트워크에서 클라우드 응용 프로그램에 필요한 포트를 모두 엽니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-204">Open any ports required by the cloud application in the on-premises network.</span></span>

<span data-ttu-id="0c0ae-205">연결을 테스트하여 다음을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-205">Test the connection to verify that:</span></span>

* <span data-ttu-id="0c0ae-206">온-프레미스 VPN 어플라이언스가 Azure VPN 게이트웨이를 통해 트래픽을 클라우드 응용 프로그램으로 올바르게 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-206">The on-premises VPN appliance correctly routes traffic to the cloud application through the Azure VPN gateway.</span></span>
* <span data-ttu-id="0c0ae-207">VNet이 트래픽을 온-프레미스 네트워크로 올바르게 라우팅합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-207">The VNet correctly routes traffic back to the on-premises network.</span></span>
* <span data-ttu-id="0c0ae-208">두 방향에서 금지된 트래픽이 올바르게 차단됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-208">Prohibited traffic in both directions is blocked correctly.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="0c0ae-209">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="0c0ae-209">Scalability considerations</span></span>

<span data-ttu-id="0c0ae-210">기본 또는 표준 VPN 게이트웨이 SKU에서 고성능 VPN SKU로 바꾸면 제한된 수직 확장성을 달성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-210">You can achieve limited vertical scalability by moving from the Basic or Standard VPN Gateway SKUs to the High Performance VPN SKU.</span></span>

<span data-ttu-id="0c0ae-211">다량의 VPN 트래픽이 예상되는 VNet이라면 서로 다른 워크로드를 보다 작은 여러 개의 VNet으로 분배하고 각각에 대해 VPN 게이트웨이를 구성하는 방법을 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-211">For VNets that expect a large volume of VPN traffic, consider distributing the different workloads into separate smaller VNets and configuring a VPN gateway for each of them.</span></span>

<span data-ttu-id="0c0ae-212">VNet은 수평 또는 수직으로 분할할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-212">You can partition the VNet either horizontally or vertically.</span></span> <span data-ttu-id="0c0ae-213">수평으로 분할하려면 각 계층에 속한 일부 VM 인스턴스를 새로운 VNet의 서브넷으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-213">To partition horizontally, move some VM instances from each tier into subnets of the new VNet.</span></span> <span data-ttu-id="0c0ae-214">이렇게 하면 각 VNet이 동일한 구조와 기능을 갖게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-214">The result is that each VNet has the same structure and functionality.</span></span> <span data-ttu-id="0c0ae-215">수직으로 분할하려면 기능이 서로 다른 논리 영역(주문 처리, 송장 발행, 고객 계정 관리 등)으로 분할되도록 각 계층을 다시 디자인합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-215">To partition vertically, redesign each tier to divide the functionality into different logical areas (such as handling orders, invoicing, customer account management, and so on).</span></span> <span data-ttu-id="0c0ae-216">이렇게 한 뒤 각 기능 영역을 자체 VNet에 배치할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-216">Each functional area can then be placed in its own VNet.</span></span>

<span data-ttu-id="0c0ae-217">VNet에 온-프레미스 Active Directory 도메인 컨트롤러를 복제하고 VNet에 DNS를 구현하면 온-프레미스에서 클라우드로 흐르는 보안 관련 트래픽과 관리 트래픽 중 일부를 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-217">Replicating an on-premises Active Directory domain controller in the VNet, and implementing DNS in the VNet, can help to reduce some of the security-related and administrative traffic flowing from on-premises to the cloud.</span></span> <span data-ttu-id="0c0ae-218">자세한 내용은 [AD DS(Active Directory Domain Services)를 Azure로 확장][adds-extend-domain]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-218">For more information, see [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="0c0ae-219">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="0c0ae-219">Availability considerations</span></span>

<span data-ttu-id="0c0ae-220">Azure VPN 게이트웨이에서 온-프레미스 네트워크에 계속해서 액세스할 수 있도록 하려면 온-프레미스 VPN 게이트웨이를 위한 장애 조치(failover) 클러스터를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-220">If you need to ensure that the on-premises network remains available to the Azure VPN gateway, implement a failover cluster for the on-premises VPN gateway.</span></span>

<span data-ttu-id="0c0ae-221">조직에 여러 개의 온-프레미스 사이트가 있는 경우 하나 이상의 Azure VNet에 [다중 사이트 연결][vpn-gateway-multi-site]을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-221">If your organization has multiple on-premises sites, create [multi-site connections][vpn-gateway-multi-site] to one or more Azure VNets.</span></span> <span data-ttu-id="0c0ae-222">이렇게 하려면 동적(경로 기반) 라우팅이 필요하므로 온-프레미스 VPN 게이트웨이가 이 기능을 지원하는지 확인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-222">This approach requires dynamic (route-based) routing, so make sure that the on-premises VPN gateway supports this feature.</span></span>

<span data-ttu-id="0c0ae-223">서비스 수준 계약에 대한 자세한 내용은 [VPN 게이트웨이의 SLA][sla-for-vpn-gateway]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-223">For details about service level agreements, see [SLA for VPN Gateway][sla-for-vpn-gateway].</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="0c0ae-224">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="0c0ae-224">Manageability considerations</span></span>

<span data-ttu-id="0c0ae-225">온-프레미스 VPN 어플라이언스에서 진단 정보를 모니터링합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-225">Monitor diagnostic information from on-premises VPN appliances.</span></span> <span data-ttu-id="0c0ae-226">이 프로세스는 VPN 어플라이언스에서 제공하는 기능에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-226">This process depends on the features provided by the VPN appliance.</span></span> <span data-ttu-id="0c0ae-227">예를 들어 Windows Server 2012에서 RRAS(라우팅 및 원격 액세스 서비스)를 사용한다면 [RRAS 로깅][rras-logging]이 모니터링 기능이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-227">For example, if you are using the Routing and Remote Access Service on Windows Server 2012, [RRAS logging][rras-logging].</span></span>

<span data-ttu-id="0c0ae-228">[Azure VPN 게이트웨이 진단][gateway-diagnostic-logs]을 사용하여 연결 문제 정보를 캡처합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-228">Use [Azure VPN gateway diagnostics][gateway-diagnostic-logs] to capture information about connectivity issues.</span></span> <span data-ttu-id="0c0ae-229">이 로그는 연결 요청의 소스와 목적지, 사용된 프로토콜, 연결이 설정된 방식(또는 시도가 실패한 이유)과 같은 정보를 추적하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-229">These logs can be used to track information such as the source and destinations of connection requests, which protocol was used, and how the connection was established (or why the attempt failed).</span></span>

<span data-ttu-id="0c0ae-230">Azure Portal에서 제공되는 감사 로크를 사용하여 Azure VPN 게이트웨이의 운영 로그를 모니터링합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-230">Monitor the operational logs of the Azure VPN gateway using the audit logs available in the Azure portal.</span></span> <span data-ttu-id="0c0ae-231">로컬 네트워크 게이트웨이, Azure 네트워크 게이트웨이 및 연결 각각에 대한 개별적인 로그도 사용 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-231">Separate logs are available for the local network gateway, the Azure network gateway, and the connection.</span></span> <span data-ttu-id="0c0ae-232">이 정보는 게이트웨이에 적용된 변경 사항을 추적하는 데 사용할 수 있고, 잘 작동하는 게이트웨이가 어떤 이유로 작동이 중단되는 경우에도 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-232">This information can be used to track any changes made to the gateway, and can be useful if a previously functioning gateway stops working for some reason.</span></span>

<span data-ttu-id="0c0ae-233">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="0c0ae-233">![[2]][2]</span></span>

<span data-ttu-id="0c0ae-234">연결을 모니터링하고 연결 장애 이벤트를 추적합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-234">Monitor connectivity, and track connectivity failure events.</span></span> <span data-ttu-id="0c0ae-235">[Nagios][nagios]와 같은 모니터링 패키지를 사용하여 이 정보를 캡처 및 보고할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-235">You can use a monitoring package such as [Nagios][nagios] to capture and report this information.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="0c0ae-236">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="0c0ae-236">Security considerations</span></span>

<span data-ttu-id="0c0ae-237">각 VPN 게이트웨이별로 서로 다른 공유 키를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-237">Generate a different shared key for each VPN gateway.</span></span> <span data-ttu-id="0c0ae-238">무작위 공격에 대비할 수 있도록 강력한 공유 키를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-238">Use a strong shared key to help resist brute-force attacks.</span></span>

> [!NOTE]
> <span data-ttu-id="0c0ae-239">현재 Azure Key Vault를 사용하여 Azure VPN 게이트웨이의 키를 사전에 공유할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-239">Currently, you cannot use Azure Key Vault to preshare keys for the Azure VPN gateway.</span></span>
> 
> 

<span data-ttu-id="0c0ae-240">온-프레미스 VPN 어플라이언스가 [Azure VPN 게이트웨이와 호환되는][vpn-appliance-ipsec] 암호화 방법을 사용하는지 확인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-240">Ensure that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance-ipsec].</span></span> <span data-ttu-id="0c0ae-241">정책 기반 라우팅의 경우 Azure VPN 게이트웨이는 AES256, AES128 및 3DES 암호화 알고리즘을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-241">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="0c0ae-242">경로 기반 게이트웨이의 경우 AES256 및 3DES를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-242">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="0c0ae-243">사용 중인 온-프레미스 VPN 어플라이언스가 경계 네트워크와 인터넷 사이에 존재하는 방화벽을 갖는 DMZ(경계 네트워크)상에 있는 경우에는 사이트 간 VPN 연결을 허용하기 위해 [추가적인 방화벽 규칙][additional-firewall-rules]을 구성해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-243">If your on-premises VPN appliance is on a perimeter network (DMZ) that has a firewall between the perimeter network and the Internet, you might have to configure [additional firewall rules][additional-firewall-rules] to allow the site-to-site VPN connection.</span></span>

<span data-ttu-id="0c0ae-244">VNet의 응용 프로그램이 인터넷으로 데이터를 전송하는 경우에는 목적지가 인터넷인 모든 트래픽이 온-프레미스 네트워크를 통과하여 라우팅되도록 [강제 터널링을 구현][forced-tunneling]하는 방법을 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-244">If the application in the VNet sends data to the Internet, consider [implementing forced tunneling][forced-tunneling] to route all Internet-bound traffic through the on-premises network.</span></span> <span data-ttu-id="0c0ae-245">이렇게 하면 온-프레미스 인프라에서 응용 프로그램에 의해 발신 요청을 감사할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-245">This approach enables you to audit outgoing requests made by the application from the on-premises infrastructure.</span></span>

> [!NOTE]
> <span data-ttu-id="0c0ae-246">강제 터널링은 Azure 서비스(저장소 서비스 등)와 Windows 라이선스 관리자로의 연결에 영향을 줄 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-246">Forced tunneling can impact connectivity to Azure services (the Storage Service, for example) and the Windows license manager.</span></span>
> 
> 


## <a name="troubleshooting"></a><span data-ttu-id="0c0ae-247">문제 해결</span><span class="sxs-lookup"><span data-stu-id="0c0ae-247">Troubleshooting</span></span> 

<span data-ttu-id="0c0ae-248">일반적인 VPN 관련 오류를 해결하려면 [일반적인 VPN 관련 오류 해결][troubleshooting-vpn-errors]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-248">For general information on troubleshooting common VPN-related errors, see [Troubleshooting common VPN related errors][troubleshooting-vpn-errors].</span></span>

<span data-ttu-id="0c0ae-249">온-프레미스 VPN 어플라이언스가 올바르게 작동하고 있는지 확인할 때 다음과 같은 권장 사항이 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-249">The following recommendations are useful for determining if your on-premises VPN appliance is functioning correctly.</span></span>

- <span data-ttu-id="0c0ae-250">**VPN 어플라이언스에 의해 생성된 로그 파일에 오류나 장애가 있는지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-250">**Check any log files generated by the VPN appliance for errors or failures.**</span></span>

    <span data-ttu-id="0c0ae-251">이렇게 하면 VPN 어플라이언스가 올바르게 작동하고 있는지 판단하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-251">This will help you determine if the VPN appliance is functioning correctly.</span></span> <span data-ttu-id="0c0ae-252">이 정보가 저장된 위치는 사용 중인 어플라이언스에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-252">The location of this information will vary according to your appliance.</span></span> <span data-ttu-id="0c0ae-253">예를 들어 Windows Server 2012에서 RRAS를 사용하는 경우에는 다음 PowerShell 명령을 사용하여 RRAS 서비스의 오류 이벤트 정보를 표시할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-253">For example, if you are using RRAS on Windows Server 2012, you can use the following PowerShell command to display error event information for the RRAS service:</span></span>

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    <span data-ttu-id="0c0ae-254">각 항목의 *Message* 속성은 오류에 대한 설명을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-254">The *Message* property of each entry provides a description of the error.</span></span> <span data-ttu-id="0c0ae-255">몇 가지 일반적인 예는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-255">Some common examples are:</span></span>

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    <span data-ttu-id="0c0ae-256">다음 PowerShell 명령을 사용하여 RRAS 서비스를 통해 연결하려는 시도에 대한 이벤트 로그 정보를 얻을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-256">You can also obtain event log information about attempts to connect through the RRAS service using the following PowerShell command:</span></span> 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    <span data-ttu-id="0c0ae-257">연결 장애가 발생하면 이 로그에 다음과 비슷한 오류가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-257">In the event of a failure to connect, this log will contain errors that look similar to the following:</span></span>

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- <span data-ttu-id="0c0ae-258">**VPN 게이트웨이의 연결 및 라우팅을 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-258">**Verify connectivity and routing across the VPN gateway.**</span></span>

    <span data-ttu-id="0c0ae-259">VPN 어플라이언스가 Azure VPN 게이트웨이를 통해 트래픽을 올바르게 라우팅하지 않고 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-259">The VPN appliance may not be correctly routing traffic through the Azure VPN Gateway.</span></span> <span data-ttu-id="0c0ae-260">[PsPing][psping]과 같은 도구를 사용하여 VPN 게이트웨이의 연결 및 라우팅을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-260">Use a tool such as [PsPing][psping] to verify connectivity and routing across the VPN gateway.</span></span> <span data-ttu-id="0c0ae-261">예를 들어 온-프레미스 머신에서 VNet에 위치한 웹 서버로의 연결을 테스트하려면 다음 명령을 실행합니다(`<<web-server-address>>`를 웹 서버의 주소로 대체합니다).</span><span class="sxs-lookup"><span data-stu-id="0c0ae-261">For example, to test connectivity from an on-premises machine to a web server located on the VNet, run the following command (replacing `<<web-server-address>>` with the address of the web server):</span></span>

    ```
    PsPing -t <<web-server-address>>:80
    ```

    <span data-ttu-id="0c0ae-262">온-프레미스 머신이 웹 서버로 트래픽을 올바르게 라우팅하고 있다면 다음과 같은 출력이 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-262">If the on-premises machine can route traffic to the web server, you should see output similar to the following:</span></span>

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    <span data-ttu-id="0c0ae-263">온-프레미스 머신이 해당 목적지와 올바르게 통신하지 못하고 있다면 다음과 같은 메시지가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-263">If the on-premises machine cannot communicate with the specified destination, you will see messages like this:</span></span>

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- <span data-ttu-id="0c0ae-264">**온-프레미스 방화벽이 VPN 트래픽이 통과할 수 있도록 허용하고 있으며 올바른 포트가 열려 있는지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-264">**Verify that the on-premises firewall allows VPN traffic to pass and that the correct ports are opened.**</span></span>

- <span data-ttu-id="0c0ae-265">**온-프레미스 VPN 어플라이언스가 [Azure VPN 게이트웨이와 호환되는][vpn-appliance] 암호화 방법을 사용하고 있는지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-265">**Verify that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance].**</span></span> <span data-ttu-id="0c0ae-266">정책 기반 라우팅의 경우 Azure VPN 게이트웨이는 AES256, AES128 및 3DES 암호화 알고리즘을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-266">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="0c0ae-267">경로 기반 게이트웨이의 경우 AES256 및 3DES를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-267">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="0c0ae-268">Azure VPN 게이트웨이에 문제가 있는지 판단할 때 다음과 같은 권장 사항이 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-268">The following recommendations are useful for determining if there is a problem with the Azure VPN gateway:</span></span>

- <span data-ttu-id="0c0ae-269">**[Azure VPN 게이트웨이 진단 로그][gateway-diagnostic-logs]에 잠재적인 문제가 없는지 검사합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-269">**Examine [Azure VPN gateway diagnostic logs][gateway-diagnostic-logs] for potential issues.**</span></span>

- <span data-ttu-id="0c0ae-270">**Azure VPN 게이트웨이와 온-프레미스 VPN 어플라이언스가 동일한 공유 인증 키로 구성되어 있는지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-270">**Verify that the Azure VPN gateway and on-premises VPN appliance are configured with the same shared authentication key.**</span></span>

    <span data-ttu-id="0c0ae-271">다음 Azure CLI 명령을 사용하여 Azure VPN 게이트웨이에 저장된 공유 키를 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-271">You can view the shared key stored by the Azure VPN gateway using the following Azure CLI command:</span></span>

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    <span data-ttu-id="0c0ae-272">사용 중인 온-프레미스 VPN 어플라이언스에 적합한 명령을 사용하여 해당 어플라이언스에 구성된 공유 키를 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-272">Use the command appropriate for your on-premises VPN appliance to show the shared key configured for that appliance.</span></span>

    <span data-ttu-id="0c0ae-273">Azure VPN 게이트웨이가 속한 *GatewaySubnet* 서브넷이 NSG에 연결되어 있지 않은지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-273">Verify that the *GatewaySubnet* subnet holding the Azure VPN gateway is not associated with an NSG.</span></span>

    <span data-ttu-id="0c0ae-274">다음 Azure CLI 명령을 사용하여 서브넷 정보를 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-274">You can view the subnet details using the following Azure CLI command:</span></span>

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    <span data-ttu-id="0c0ae-275">*Network Security Group id*라는 이름을 갖는 데이터 필드가 없는지 확인합니다. 다음은 NSG가 할당된 *GatewaySubnet*의 인스턴스의 결과입니다(*VPN-Gateway-Group*).</span><span class="sxs-lookup"><span data-stu-id="0c0ae-275">Ensure there is no data field named *Network Security Group id*. The following example shows the results for an instance of the *GatewaySubnet* that has an assigned NSG (*VPN-Gateway-Group*).</span></span> <span data-ttu-id="0c0ae-276">이렇게 구성되어 있으면 이 NSG에 대해 규칙이 정의되어 있는 경우 게이트웨이가 올바르게 작동하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-276">This can prevent the gateway from working correctly if there are any rules defined for this NSG.</span></span>

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- <span data-ttu-id="0c0ae-277">**Azure VNet의 가상 머신이 VNet 외부로부터 수신되는 트래픽을 허용되도록 구성되어 있는지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-277">**Verify that the virtual machines in the Azure VNet are configured to permit traffic coming in from outside the VNet.**</span></span>

    <span data-ttu-id="0c0ae-278">이러한 가상 머신이 속한 서브넷에 연결된 NSG 규칙이 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-278">Check any NSG rules associated with subnets containing these virtual machines.</span></span> <span data-ttu-id="0c0ae-279">다음 Azure CLI 명령을 사용하여 모든 NSG 규칙을 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-279">You can view all NSG rules using the following Azure CLI command:</span></span>

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- <span data-ttu-id="0c0ae-280">**Azure VPN 게이트웨이가 연결되어 있는지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-280">**Verify that the Azure VPN gateway is connected.**</span></span>

    <span data-ttu-id="0c0ae-281">다음 Azure PowerShell 명령을 사용하여 Azure VPN 연결의 현재 상태를 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-281">You can use the following Azure PowerShell command to check the current status of the Azure VPN connection.</span></span> <span data-ttu-id="0c0ae-282">`<<connection-name>>` 매개 변수는 가상 네트워크 게이트웨이와 로컬 게이트웨이를 연결하는 Azure VPN 연결의 이름입니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-282">The `<<connection-name>>` parameter is the name of the Azure VPN connection that links the virtual network gateway and the local gateway.</span></span>

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="0c0ae-283">다음 스니펫에서는 게이트웨이가 연결되어 있는 경우(첫 번째 예제)와 연결이 해제된 경우(두 번째 예제)에 생성되는 출력을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-283">The following snippets highlight the output generated if the gateway is connected (the first example), and disconnected (the second example):</span></span>

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

<span data-ttu-id="0c0ae-284">호스트 VM 구성, 네트워크 대역폭 사용 현황 또는 응용 프로그램 성능에 문제가 있는지 판단할 때 다음과 같은 권장 사항이 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-284">The following recommendations are useful for determining if there is an issue with Host VM configuration, network bandwidth utilization, or application performance:</span></span>

- <span data-ttu-id="0c0ae-285">**서브넷에 속한 Azure VM에서 실행 중인 게스트 운영 체제의 방화벽이 온-프레미스 IP 범위로부터 수신되는 허용된 트래픽을 허용하도록 올바르게 구성되어 있는지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-285">**Verify that the firewall in the guest operating system running on the Azure VMs in the subnet is configured correctly to allow permitted traffic from the on-premises IP ranges.**</span></span>

- <span data-ttu-id="0c0ae-286">**트래픽의 양이 Azure VPN 게이트웨이에서 사용할 수 있는 대역폭의 제한에 가깝지 않은지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-286">**Verify that the volume of traffic is not close to the limit of the bandwidth available to the Azure VPN gateway.**</span></span>

    <span data-ttu-id="0c0ae-287">이것을 확인하는 방법은 온-프레미스에서 실행 중인 VPN 어플라이언스에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-287">How to verify this depends on the VPN appliance running on-premises.</span></span> <span data-ttu-id="0c0ae-288">예를 들어 Windows Server 2012에서 RRAS를 사용하고 있는 경우 Performance Monitor를 사용하여 VPN 연결을 통해 송수신되고 있는 데이터의 양을 추적할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-288">For example, if you are using RRAS on Windows Server 2012, you can use Performance Monitor to track the volume of data being received and transmitted over the VPN connection.</span></span> <span data-ttu-id="0c0ae-289">*RAS 총계* 개체를 사용하여 *수신된 바이트/초* 카운터와 *송신된 바이트/초* 카운터를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-289">Using the *RAS Total* object, select the *Bytes Received/Sec* and *Bytes Transmitted/Sec* counters:</span></span>

    <span data-ttu-id="0c0ae-290">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="0c0ae-290">![[3]][3]</span></span>

    <span data-ttu-id="0c0ae-291">결과를 VPN 게이트웨이에서 사용할 수 있는 대역폭(기본 및 표준 SKU: 100Mbps, 고성능 SKU: 200Mbps)과 비교합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-291">You should compare the results with the bandwidth available to the VPN gateway (100 Mbps for the Basic and Standard SKUs, and 200 Mbps for the High Performance SKU):</span></span>

    <span data-ttu-id="0c0ae-292">![[4]][4]</span><span class="sxs-lookup"><span data-stu-id="0c0ae-292">![[4]][4]</span></span>

- <span data-ttu-id="0c0ae-293">**응용 프로그램 부하에 맞는 올바른 VM 개수와 크기를 배포했는지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-293">**Verify that you have deployed the right number and size of VMs for your application load.**</span></span>

    <span data-ttu-id="0c0ae-294">Azure VNet의 가상 머신 중 느리게 실행되고 있는 VM은 없는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-294">Determine if any of the virtual machines in the Azure VNet are running slowly.</span></span> <span data-ttu-id="0c0ae-295">느리게 실행되고 있는 VM이 있다면 과부하되었거나 부하를 처리할 VM이 너무 적거나 부하 분산 장치가 올바르게 구성되어 있지 않은 것일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-295">If so, they may be overloaded, there may be too few to handle the load, or the load-balancers may not be configured correctly.</span></span> <span data-ttu-id="0c0ae-296">원인을 알아내려면 [진단 정보를 캡처 및 분석][azure-vm-diagnostics]합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-296">To determine this, [capture and analyze diagnostic information][azure-vm-diagnostics].</span></span> <span data-ttu-id="0c0ae-297">결과는 Azure Portal을 사용하여 검사할 수 있으며, 그 밖에도 다양한 타사 도구를 사용하여 성능 데이터를 자세히 검사할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-297">You can examine the results using the Azure portal, but many third-party tools are also available that can provide detailed insights into the performance data.</span></span>

- <span data-ttu-id="0c0ae-298">**응용 프로그램이 클라우드 리소스를 효율적으로 사용하고 있는지 확인합니다.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-298">**Verify that the application is making efficient use of cloud resources.**</span></span>

    <span data-ttu-id="0c0ae-299">각 VM에서 실행 중인 응용 프로그램 코드를 이용하여 각 응용 프로그램에서 리소스를 최대한 효율적으로 사용하고 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-299">Instrument application code running on each VM to determine whether applications are making the best use of resources.</span></span> <span data-ttu-id="0c0ae-300">[Application Insights][application-insights]와 같은 도구를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-300">You can use tools such as [Application Insights][application-insights].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="0c0ae-301">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="0c0ae-301">Deploy the solution</span></span>


<span data-ttu-id="0c0ae-302">**필수 조건.**</span><span class="sxs-lookup"><span data-stu-id="0c0ae-302">**Prequisites.**</span></span> <span data-ttu-id="0c0ae-303">적절한 네트워크 어플라이언스를 사용하여 기존 온-프레미스 인프라가 이미 구성된 상태여야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-303">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="0c0ae-304">솔루션을 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-304">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="0c0ae-305">아래 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-305">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="0c0ae-306">Azure Portal에서 링크가 열릴 때까지 기다린 후 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-306">Wait for the link to open in the Azure portal, then follow these steps:</span></span> 
   * <span data-ttu-id="0c0ae-307">**리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-hybrid-vpn-rg`를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-307">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-rg` in the text box.</span></span>
   * <span data-ttu-id="0c0ae-308">**위치** 드롭다운 상자에서 하위 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-308">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="0c0ae-309">**템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-309">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="0c0ae-310">사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-310">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="0c0ae-311">**구매** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-311">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="0c0ae-312">배포가 완료될 때가지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="0c0ae-312">Wait for the deployment to complete.</span></span>



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
<!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "온-프레미스 인프라와 Azure 인프라를 포괄하는 하이브리드 네트워크"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Azure Portal의 감사 로그"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "VPN 네트워크 트래픽 모니터링을 위한 성능 카운터"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "VPN 네트워크 성능 그래프 예"