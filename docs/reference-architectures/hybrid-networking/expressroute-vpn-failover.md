---
title: "고가용성 하이브리드 네트워크 아키텍처 구현"
description: "VPN Gateway 장애 조치(failover)를 사용하는 ExpressRoute를 사용하여 연결된 Azure 가상 네트워크 및 온-프레미스 네트워크를 포함하는 보안 사이트 간 네트워크 아키텍처를 구축하는 방법"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 4c101f17e5e91085b61178f9efb2bc5acb61189c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a><span data-ttu-id="75233-103">VPN 장애 조치(failover)를 사용하는 ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결</span><span class="sxs-lookup"><span data-stu-id="75233-103">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span>

<span data-ttu-id="75233-104">이 참조 아키텍처는 사이트 간 VPN(가상 사설망)을 장애 조치(failover) 연결로 사용하여 ExpressRoute를 통해 온-프레미스 네트워크를 Azure VNet(Virtual Network)에 연결하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="75233-104">This reference architecture shows how to connect an on-premises network to an Azure virtual network (VNet) using ExpressRoute, with a site-to-site virtual private network (VPN) as a failover connection.</span></span> <span data-ttu-id="75233-105">ExpressRoute 연결을 통해 온-프레미스 네트워크와 Azure VNet 사이에서 트래픽이 흐릅니다.</span><span class="sxs-lookup"><span data-stu-id="75233-105">Traffic flows between the on-premises network and the Azure VNet through an ExpressRoute connection.</span></span> <span data-ttu-id="75233-106">ExpressRoute 회로의 연결이 끊어진 경우 트래픽은 IPSec VPN 터널을 통해 라우팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="75233-106">If there is a loss of connectivity in the ExpressRoute circuit, traffic is routed through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="75233-107">**이 솔루션을 배포합니다**.</span><span class="sxs-lookup"><span data-stu-id="75233-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="75233-108">ExpressRoute 회로를 사용할 수 없는 경우 VPN 경로가 개인 피어링 연결만 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-108">Note that if the ExpressRoute circuit is unavailable, the VPN route will only handle private peering connections.</span></span> <span data-ttu-id="75233-109">공용 피어링 및 Microsoft 피어링 연결은 인터넷을 통과합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-109">Public peering and Microsoft peering connections will pass over the Internet.</span></span> 

<span data-ttu-id="75233-110">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="75233-110">![[0]][0]</span></span>

<span data-ttu-id="75233-111">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="75233-111">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="75233-112">건축</span><span class="sxs-lookup"><span data-stu-id="75233-112">Architecture</span></span> 

<span data-ttu-id="75233-113">이 아키텍처는 다음 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="75233-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="75233-114">**온-프레미스 네트워크**.</span><span class="sxs-lookup"><span data-stu-id="75233-114">**On-premises network**.</span></span> <span data-ttu-id="75233-115">조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.</span><span class="sxs-lookup"><span data-stu-id="75233-115">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="75233-116">**VPN 어플라이언스**.</span><span class="sxs-lookup"><span data-stu-id="75233-116">**VPN appliance**.</span></span> <span data-ttu-id="75233-117">온-프레미스 네트워크에 외부 연결을 제공하는 장치 또는 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="75233-117">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="75233-118">VPN 어플라이언스는 하드웨어 장치일 수도 있고 Windows Server 2012의 RRAS(라우팅 및 원격 액세스 서비스)와 같은 소프트웨어 솔루션일 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="75233-118">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="75233-119">지원되는 VPN 어플라이언스 목록 및 선택한 VPN 어플라이언스를 Azure에 연결하도록 구성하는 방법에 대한 자세한 내용은 [사이트 간 VPN Gateway 연결에 대한 VPN 장치 정보][vpn-appliance]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="75233-119">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="75233-120">**ExpressRoute 회로**.</span><span class="sxs-lookup"><span data-stu-id="75233-120">**ExpressRoute circuit**.</span></span> <span data-ttu-id="75233-121">에지 라우터를 통해 Azure에서 온-프레미스 네트워크를 연결하는 연결 공급자가 공급하는 계층 2 또는 계층 3 회로입니다.</span><span class="sxs-lookup"><span data-stu-id="75233-121">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="75233-122">이 회로는 연결 공급자가 관리하는 하드웨어 인프라를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-122">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="75233-123">**ExpressRoute 가상 네트워크 게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="75233-123">**ExpressRoute virtual network gateway**.</span></span> <span data-ttu-id="75233-124">ExpressRoute 가상 네트워크 게이트웨이를 사용하면 VNet을 온-프레미스 네트워크에 연결하는 데 사용되는 ExpressRoute 회로에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="75233-124">The ExpressRoute virtual network gateway enables the VNet to connect to the ExpressRoute circuit used for connectivity with your on-premises network.</span></span>

* <span data-ttu-id="75233-125">**VPN 가상 네트워크 게이트웨이**.</span><span class="sxs-lookup"><span data-stu-id="75233-125">**VPN virtual network gateway**.</span></span> <span data-ttu-id="75233-126">VPN 가상 네트워크 게이트웨이를 사용하면 VNet을 온-프레미스 네트워크의 VPN 어플라이언스에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="75233-126">The VPN virtual network gateway enables the VNet to connect to the VPN appliance in the on-premises network.</span></span> <span data-ttu-id="75233-127">VPN 가상 네트워크 게이트웨이는 VPN 어플라이언스를 통해서만 온-프레미스 네트워크의 요청을 수락하도록 구성되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="75233-127">The VPN virtual network gateway is configured to accept requests from the on-premises network only through the VPN appliance.</span></span> <span data-ttu-id="75233-128">자세한 내용은 [온-프레미스 네트워크를 Microsoft Azure Virtual Network에 연결][connect-to-an-Azure-vnet]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="75233-128">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

* <span data-ttu-id="75233-129">**VPN 연결**.</span><span class="sxs-lookup"><span data-stu-id="75233-129">**VPN connection**.</span></span> <span data-ttu-id="75233-130">이 연결에는 연결 형식(IPSec) 및 트래픽을 암호화하기 위해 온-프레미스 VPN 어플라이언스와 공유되는 키를 지정하는 속성이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="75233-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>

* <span data-ttu-id="75233-131">**Azure VNet(Virtual Network)**.</span><span class="sxs-lookup"><span data-stu-id="75233-131">**Azure Virtual Network (VNet)**.</span></span> <span data-ttu-id="75233-132">각 VNet은 단일 Azure 지역에 상주하며 여러 응용 프로그램 계층을 호스트할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="75233-132">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="75233-133">응용 프로그램 계층은 각 VNet의 서브넷을 사용하여 분할할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="75233-133">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="75233-134">**게이트웨이 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="75233-134">**Gateway subnet**.</span></span> <span data-ttu-id="75233-135">가상 네트워크 게이트웨이는 동일한 서브넷에 있습니다.</span><span class="sxs-lookup"><span data-stu-id="75233-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="75233-136">**클라우드 응용 프로그램**.</span><span class="sxs-lookup"><span data-stu-id="75233-136">**Cloud application**.</span></span> <span data-ttu-id="75233-137">Azure 에서 호스팅되는 응용 프로그램입니다.</span><span class="sxs-lookup"><span data-stu-id="75233-137">The application hosted in Azure.</span></span> <span data-ttu-id="75233-138">Azure 부하 분산 장치를 통해 여러 서브넷이 연결된 여러 계층이 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="75233-138">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="75233-139">응용 프로그램 인프라에 대한 자세한 내용은 [Windows VM 워크로드 실행][windows-vm-ra] 및 [Linux VM 워크로드 실행][linux-vm-ra]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="75233-139">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

## <a name="recommendations"></a><span data-ttu-id="75233-140">권장 사항</span><span class="sxs-lookup"><span data-stu-id="75233-140">Recommendations</span></span>

<span data-ttu-id="75233-141">대부분의 시나리오의 경우 다음 권장 사항을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="75233-142">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="75233-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="75233-143">VNet과 GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="75233-143">VNet and GatewaySubnet</span></span>

<span data-ttu-id="75233-144">동일한 VNet에서 ExpressRoute 가상 네트워크 게이트웨이와 VPN 가상 네트워크 게이트웨이를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="75233-144">Create the ExpressRoute virtual network gateway and the VPN virtual network gateway in the same VNet.</span></span> <span data-ttu-id="75233-145">즉, 이름이 *GatewaySubnet*인 동일한 서브넷을 공유해야 한다는 의미입니다.</span><span class="sxs-lookup"><span data-stu-id="75233-145">This means that they should share the same subnet named *GatewaySubnet*.</span></span>

<span data-ttu-id="75233-146">VNet에 이름이*GatewaySubnet*인 서브넷이 이미 포함되어 있는 경우 /27 이상의 주소 공간이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-146">If the VNet already includes a subnet named *GatewaySubnet*, ensure that it has a /27 or larger address space.</span></span> <span data-ttu-id="75233-147">기존 서브넷이 너무 작은 경우 다음 PowerShell 명령을 사용하여 서브넷을 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-147">If the existing subnet is too small, use the following PowerShell command to remove the subnet:</span></span> 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

<span data-ttu-id="75233-148">VNet에 이름이 **GatewaySubnet**인 서브넷이 없는 경우 다음 Powershell 명령을 사용하여 새 서브넷을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="75233-148">If the VNet does not contain a subnet named **GatewaySubnet**, create a new one using the following Powershell command:</span></span>

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a><span data-ttu-id="75233-149">VPN 및 ExpressRoute 게이트웨이</span><span class="sxs-lookup"><span data-stu-id="75233-149">VPN and ExpressRoute gateways</span></span>

<span data-ttu-id="75233-150">조직이 Azure에 연결하기 위한 [ExpressRoute 필수 조건 요구 사항][expressroute-prereq]을 충족하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-150">Verify that your organization meets the [ExpressRoute prerequisite requirements][expressroute-prereq] for connecting to Azure.</span></span>

<span data-ttu-id="75233-151">Azure VNet에 VPN 가상 네트워크 게이트웨이가 이미 포함되어 있는 경우 다음과 같은 Powershell 명령을 사용하여 제거합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-151">If you already have a VPN virtual network gateway in your Azure VNet, use the following  Powershell command to remove it:</span></span>

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

<span data-ttu-id="75233-152">[Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][implementing-expressroute]의 지침을 따라 ExpressRoute 연결을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-152">Follow the instructions in [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] to establish your ExpressRoute connection.</span></span>

<span data-ttu-id="75233-153">[Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][implementing-vpn]의 지침을 따라 VPN 가상 네트워크 게이트웨이 연결을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-153">Follow the instructions in [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] to establish your VPN virtual network gateway connection.</span></span>

<span data-ttu-id="75233-154">가상 네트워크 게이트웨이 연결을 설정한 후 다음과 같이 환경을 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-154">After you have established the virtual network gateway connections, test the environment as follows:</span></span>

1. <span data-ttu-id="75233-155">온-프레미스 네트워크에서 Azure VNet으로 연결할 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-155">Make sure you can connect from your on-premises network to your Azure VNet.</span></span>
2. <span data-ttu-id="75233-156">테스트를 위해 ExpressRoute 연결을 중지하려면 공급자에게 문의합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-156">Contact your provider to stop ExpressRoute connectivity for testing.</span></span>
3. <span data-ttu-id="75233-157">VPN 가상 네트워크 게이트웨이 연결을 사용하여 온-프레미스 네트워크에서 Azure VNet으로 계속 연결할 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-157">Verify that you can still connect from your on-premises network to your Azure VNet using the VPN virtual network gateway connection.</span></span>
4. <span data-ttu-id="75233-158">ExpressRoute 연결을 재설정하려면 공급자에게 문의합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-158">Contact your provider to reestablish ExpressRoute connectivity.</span></span>

## <a name="considerations"></a><span data-ttu-id="75233-159">고려 사항</span><span class="sxs-lookup"><span data-stu-id="75233-159">Considerations</span></span>

<span data-ttu-id="75233-160">ExpressRoute 고려 사항은 [Azure ExpressRoute를 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-expressroute] 지침을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="75233-160">For ExpressRoute considerations, see the [Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute] guidance.</span></span>

<span data-ttu-id="75233-161">사이트 간 VPN 고려 사항은 [Azure 및 온-프레미스 VPN을 사용하여 하이브리드 네트워크 아키텍처 구현][guidance-vpn] 지침을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="75233-161">For site-to-site VPN considerations, see the [Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn] guidance.</span></span>

<span data-ttu-id="75233-162">일반적인 Azure 보안 고려 사항은 [Microsoft Cloud Services 및 네트워크 보안][best-practices-security]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="75233-162">For general Azure security considerations, see [Microsoft cloud services and network security][best-practices-security].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="75233-163">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="75233-163">Deploy the solution</span></span>

<span data-ttu-id="75233-164">**필수 조건.**</span><span class="sxs-lookup"><span data-stu-id="75233-164">**Prequisites.**</span></span> <span data-ttu-id="75233-165">적절한 네트워크 어플라이언스를 사용하여 이미 구성된 기존 온-프레미스 인프라가 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-165">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="75233-166">솔루션을 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-166">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="75233-167">아래 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-167">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="75233-168">Azure Portal에 링크가 열릴 때까지 기다린 후 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-168">Wait for the link to open in the Azure portal, then follow these steps:</span></span>   
   * <span data-ttu-id="75233-169">**리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-hybrid-vpn-er-rg`를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-169">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="75233-170">**위치** 드롭다운 상자에서 하위 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-170">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="75233-171">**템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="75233-171">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="75233-172">사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-172">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="75233-173">**구매** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-173">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="75233-174">배포가 완료될 때가지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="75233-174">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="75233-175">아래 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-175">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="75233-176">Azure Portal에 링크가 열릴 때까지 기다린 후 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-176">Wait for the link to open in the Azure portal, then enter then follow these steps:</span></span>
   * <span data-ttu-id="75233-177">**리소스 그룹** 섹션에서 **기존 항목 사용**을 선택하고 텍스트 상자에 `ra-hybrid-vpn-er-rg`를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-177">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="75233-178">**위치** 드롭다운 상자에서 하위 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-178">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="75233-179">**템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="75233-179">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="75233-180">사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-180">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="75233-181">**구매** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="75233-181">Click the **Purchase** button.</span></span>

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md


[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "ExpressRoute 및 VPN Gateway를 사용하는 고가용성 하이브리드 네트워크 아키텍처의 아키텍처"
