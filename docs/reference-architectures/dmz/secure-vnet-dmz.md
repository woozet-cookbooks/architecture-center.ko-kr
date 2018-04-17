---
title: Azure와 인터넷 간의 DMZ 구현
description: Azure에서 인터넷 액세스로 보안 하이브리드 네트워크 아키텍처를 구현하는 방법입니다.
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: c88545b1fcae49b413e7e2b6ac5bd92d3fd3456d
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/30/2018
---
# <a name="dmz-between-azure-and-the-internet"></a><span data-ttu-id="0a9ea-103">Azure와 인터넷 간의 DMZ</span><span class="sxs-lookup"><span data-stu-id="0a9ea-103">DMZ between Azure and the Internet</span></span>

<span data-ttu-id="0a9ea-104">이 참조 아키텍처는 온-프레미스 네트워크를 Azure로 확장하고 인터넷 트래픽을 수락하는 보안 하이브리드 네트워크를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure and also accepts Internet traffic.</span></span> 

<span data-ttu-id="0a9ea-105">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="0a9ea-105">[![0]][0]</span></span> 

<span data-ttu-id="0a9ea-106">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="0a9ea-106">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="0a9ea-107">이 참조 아키텍처는 [Azure와 온-프레미스 데이터 센터 간 DMZ 구현][implementing-a-secure-hybrid-network-architecture]에 설명된 아키텍처를 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-107">This reference architecture extends the architecture described in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture].</span></span> <span data-ttu-id="0a9ea-108">이 참조 아키텍처는 온-프레미스 네트워크로부터 오는 트래픽을 처리하는 사설 DMZ 외에 인터넷 트래픽을 처리하는 공용 DMZ를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-108">It adds a public DMZ that handles Internet traffic, in addition to the private DMZ that handles traffic from the on-premises network</span></span> 

<span data-ttu-id="0a9ea-109">일반적으로 이 아키텍처는 다음과 같은 용도로 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-109">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="0a9ea-110">워크로드의 일부는 온-프레미스에서, 일부는 Azure에서 실행되는 하이브리드 응용 프로그램</span><span class="sxs-lookup"><span data-stu-id="0a9ea-110">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
* <span data-ttu-id="0a9ea-111">온-프레미스 및 인터넷으로부터 오는 트래픽을 라우팅하는 Azure 인프라</span><span class="sxs-lookup"><span data-stu-id="0a9ea-111">Azure infrastructure that routes incoming traffic from on-premises and the Internet.</span></span>

## <a name="architecture"></a><span data-ttu-id="0a9ea-112">건축</span><span class="sxs-lookup"><span data-stu-id="0a9ea-112">Architecture</span></span>

<span data-ttu-id="0a9ea-113">이 아키텍처는 다음 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="0a9ea-114">**공용 IP 주소(PIP)**.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-114">**Public IP address (PIP)**.</span></span> <span data-ttu-id="0a9ea-115">공용 끝점의 IP 주소.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-115">The IP address of the public endpoint.</span></span> <span data-ttu-id="0a9ea-116">인터넷에 연결된 외부 사용자는 이 주소를 통해 시스템에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-116">External users connected to the Internet can access the system through this address.</span></span>
* <span data-ttu-id="0a9ea-117">**NVA(네트워크 가상 어플라이언스)**.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-117">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="0a9ea-118">이 아키텍처는 인터넷에서 오는 트래픽을 위한 별도의 NVA 풀을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-118">This architecture includes a separate pool of NVAs for traffic originating on the Internet.</span></span>
* <span data-ttu-id="0a9ea-119">**Azure Load Balancer**.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-119">**Azure load balancer**.</span></span> <span data-ttu-id="0a9ea-120">인터넷에서 오는 모든 요청은 부하 분산 장치를 통과하고 공용 DMZ 내 NVA로 분산됩니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-120">All incoming requests from the Internet pass through the load balancer and are distributed to the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="0a9ea-121">**공용 DMZ 인바운드 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-121">**Public DMZ inbound subnet**.</span></span> <span data-ttu-id="0a9ea-122">이 서브넷은 Azure Load Balancer로부터 오는 요청을 수용합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-122">This subnet accepts requests from the Azure load balancer.</span></span> <span data-ttu-id="0a9ea-123">들어오는 요청은 공용 DMZ 내 NVA 중 하나로 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-123">Incoming requests are passed to one of the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="0a9ea-124">**공용 DMZ 아웃바운드 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-124">**Public DMZ outbound subnet**.</span></span> <span data-ttu-id="0a9ea-125">NVA에 의해 승인된 요청은 이 서브넷을 거쳐 웹 계층을 위한 내부 부하 분산 장치로 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-125">Requests that are approved by the NVA pass through this subnet to the internal load balancer for the web tier.</span></span>

## <a name="recommendations"></a><span data-ttu-id="0a9ea-126">권장 사항</span><span class="sxs-lookup"><span data-stu-id="0a9ea-126">Recommendations</span></span>

<span data-ttu-id="0a9ea-127">대부분의 시나리오의 경우 다음 권장 사항을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-127">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="0a9ea-128">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-128">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="nva-recommendations"></a><span data-ttu-id="0a9ea-129">NVA 권장 사항</span><span class="sxs-lookup"><span data-stu-id="0a9ea-129">NVA recommendations</span></span>

<span data-ttu-id="0a9ea-130">인터넷에서 발생하는 트래픽에 대해 하나의 NVA 집합을 사용하고, 온-프레미스에서 발생하는 트래픽에 대해 다른 NVA 집합을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-130">Use one set of NVAs for traffic originating on the Internet, and another for traffic originating on-premises.</span></span> <span data-ttu-id="0a9ea-131">둘 다에 대해 하나의 NVA 집합만 사용하면 두 네트워크 트래픽 집합 사이에 보안 경계가 없어지므로 보안 위험이 초래됩니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-131">Using only one set of NVAs for both is a security risk, because it provides no security perimeter between the two sets of network traffic.</span></span> <span data-ttu-id="0a9ea-132">별도 NVA를 사용하면 보안 규칙을 검사하는 복잡성이 줄어들고 들어오는 네트워크 요청에 해당하는 규칙이 명확해집니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-132">Using separate NVAs reduces the complexity of checking security rules, and makes it clear which rules correspond to each incoming network request.</span></span> <span data-ttu-id="0a9ea-133">하나의 NVA 집합은 인터넷 트래픽에만 적용되는 규칙을 수행하고 다른 하나의 NVA 집합은 온-프레미스 트래픽에만 적용되는 규칙을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-133">One set of NVAs implements rules for Internet traffic only, while another set of NVAs implement rules for on-premises traffic only.</span></span>

<span data-ttu-id="0a9ea-134">NVA 수준에서 응용 프로그램 연결을 종료하고 백 엔드 계층 호환성을 유지하기 위해 레이어7 NVA를 포함시킵니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-134">Include a layer-7 NVA to terminate application connections at the NVA level and maintain compatibility with the backend tiers.</span></span> <span data-ttu-id="0a9ea-135">이를 통해 백 엔드 계층으로부터의 응답 트래픽이 NVA를 통해 돌아오는 대칭 연결이 보장됩니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-135">This guarantees symmetric connectivity where response traffic from the backend tiers returns through the NVA.</span></span>  

### <a name="public-load-balancer-recommendations"></a><span data-ttu-id="0a9ea-136">공용 부하 분산 장치 권장 사항</span><span class="sxs-lookup"><span data-stu-id="0a9ea-136">Public load balancer recommendations</span></span>

<span data-ttu-id="0a9ea-137">높은 확장성과 가용성을 위해 공용 DMZ NVA를 하나의 [가용성 집합][availability-set] 내에 배포하고 [인터넷 연결 부하 분산 장치][load-balancer]를 사용하여 인터넷 요청을 가용성 집합 내 NVA에 분산시킵니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-137">For scalability and availability, deploy the public DMZ NVAs in an [availability set][availability-set] and use an [Internet facing load balancer][load-balancer] to distribute Internet requests across the NVAs in the availability set.</span></span>  

<span data-ttu-id="0a9ea-138">부하 분산 장치가 인터넷 트래픽에 필요한 포트로만 요청을 받도록 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-138">Configure the load balancer to accept requests only on the ports necessary for Internet traffic.</span></span> <span data-ttu-id="0a9ea-139">예를 들어, 인바운드 HTTP 요청은 포트 80으로 제한하고 아웃바운드 HTTPS 요청은 포트 443으로 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-139">For example, restrict inbound HTTP requests to port 80 and inbound HTTPS requests to port 443.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="0a9ea-140">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="0a9ea-140">Scalability considerations</span></span>

<span data-ttu-id="0a9ea-141">처음에는 아키텍처가 공용 DMZ 내 단일 NVA를 요구하지만, 처음부터 공용 DMZ 앞에 부하 분산 장치를 배치할 것을 권장합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-141">Even if your architecture initially requires a single NVA in the public DMZ, we recommend putting a load balancer in front of the public DMZ from the beginning.</span></span> <span data-ttu-id="0a9ea-142">이를 통해 향후 필요한 경우 여러 NVA로 보다 수월하게 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-142">That will make it easier to scale to multiple NVAs in the future, if needed.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="0a9ea-143">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="0a9ea-143">Availability considerations</span></span>

<span data-ttu-id="0a9ea-144">인터넷 연결 부하 분산 장치는 공용 DMZ 수신 서브넷 내의 각 NVA가 [상태 프로브][lb-probe]를 구현하도록 요구합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-144">The Internet facing load balancer requires each NVA in the public DMZ inbound subnet to implement a [health probe][lb-probe].</span></span> <span data-ttu-id="0a9ea-145">이 끝점에서 응답하지 않는 상태 프로브는 사용 불가능한 것으로 간주되어 부하 분산 장치가 요청을 동일 가용성 집합 내 다른 NVA로 전달하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-145">A health probe that fails to respond on this endpoint is considered to be unavailable, and the load balancer will direct requests to other NVAs in the same availability set.</span></span> <span data-ttu-id="0a9ea-146">모든 NVA가 응답하지 않는 경우 응용 프로그램이 중지되므로 정상적인 NVA 인스턴스 수가 정의된 한계 아래로 내려가면 DevOps에 알리도록 모니터링 기능을 구성하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-146">Note that if all NVAs fail to respond, your application will fail, so it's important to have monitoring configured to alert DevOps when the number of healthy NVA instances falls below a defined threshold.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="0a9ea-147">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="0a9ea-147">Manageability considerations</span></span>

<span data-ttu-id="0a9ea-148">공용 DMZ 내 NVA에 대한 모든 모니터링 및 관리는 관리 서브넷의 점프박스를 통해 수행되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-148">All monitoring and management for the NVAs in the public DMZ should be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="0a9ea-149">[Azure와 온-프레미스 데이터 센터 간 DMZ 구현][implementing-a-secure-hybrid-network-architecture]에서 언급되었듯이, 온-프레미스 네트워크로부터 게이트웨이를 거쳐 점프박스로 이어지는 단일의 네트워크 루트를 정의하여 액세스를 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-149">As discussed in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture], define a single network route from the on-premises network through the gateway to the jumpbox, in order to restrict access.</span></span>

<span data-ttu-id="0a9ea-150">온-프레미스 네트워크에서 Azure로의 게이트웨이 연결이 중단되는 경우에도 공용 IP 주소를 배포하고 점프박스에 추가한 후 인터넷으로부터 로그인하여 점프박스에 액세스할 수 있습니다</span><span class="sxs-lookup"><span data-stu-id="0a9ea-150">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and logging in from the Internet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="0a9ea-151">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="0a9ea-151">Security considerations</span></span>

<span data-ttu-id="0a9ea-152">이 참조 아키텍처는 여러 수준의 보안을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-152">This reference architecture implements multiple levels of security:</span></span>

* <span data-ttu-id="0a9ea-153">인터넷 연결 부하 분산 장치는 해당 응용 프로그램에 필요한 포트를 통해서만 수신 공용 DMZ 서브넷 내 NVA로 요청을 전달합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-153">The Internet facing load balancer directs requests to the NVAs in the inbound public DMZ subnet, and only on the ports necessary for the application.</span></span>
* <span data-ttu-id="0a9ea-154">인바운드 및 아웃바운드 공용 DMZ 서브넷을 위한 NSG(네트워크 보안 그룹) 규칙을 통해 NSG 규칙을 충족하지 않는 요청을 차단함으로써 NVA를 위험으로부터 보호할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-154">The NSG rules for the inbound and outbound public DMZ subnets prevent the NVAs from being compromised, by blocking requests that fall outside of the NSG rules.</span></span>
* <span data-ttu-id="0a9ea-155">NVA를 위한 NAT 라우팅 구성에 따라 포트 80과 포트 443으로 들어오는 요청을 웹 계층의 부하 분산 장치로 전달하고 그 외 다른 모든 포트로 오는 요청은 무시합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-155">The NAT routing configuration for the NVAs directs incoming requests on port 80 and port 443 to the web tier load balancer, but ignores requests on all other ports.</span></span>

<span data-ttu-id="0a9ea-156">모든 포트로 오는 모든 요청에 대한 로그를 기록해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-156">You should log all incoming requests on all ports.</span></span> <span data-ttu-id="0a9ea-157">예상되는 매개변수 범위를 벗어나는 요청은 침입 시도를 의미할 수 있으므로 이에 관심을 기울여 정기적인 로그 감사를 실시합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-157">Regularly audit the logs, paying attention to requests that fall outside of expected parameters, as these may indicate intrusion attempts.</span></span>

## <a name="solution-deployment"></a><span data-ttu-id="0a9ea-158">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="0a9ea-158">Solution deployment</span></span>

<span data-ttu-id="0a9ea-159">이러한 권장 사항을 구현하는 참조 아키텍처 배포는 [GitHub][github-folder]를 통해 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-159">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span> <span data-ttu-id="0a9ea-160">이 참조 아키텍처는 다음 지침에 따라 Windows 또는 Linux VM을 사용하여 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-160">The reference architecture can be deployed either with Windows or Linux VMs by following the directions below:</span></span>

1. <span data-ttu-id="0a9ea-161">아래 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-161">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="0a9ea-162">Azure 포털에서 링크가 열렸으면 일부 설정에 대한 값을 입력해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-162">Once the link has opened in the Azure portal, you must enter values for some of the settings:</span></span>
   * <span data-ttu-id="0a9ea-163">**리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-public-dmz-network-rg`를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-163">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-public-dmz-network-rg` in the text box.</span></span>
   * <span data-ttu-id="0a9ea-164">**위치** 드롭다운 상자에서 하위 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-164">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="0a9ea-165">**템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-165">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="0a9ea-166">드롭다운 상자에서 **OS 유형**을 선택으로 **Windows** 또는 **Linux**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-166">Select the **Os Type** from the drop down box, **windows** or **linux**.</span></span>
   * <span data-ttu-id="0a9ea-167">사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-167">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="0a9ea-168">**구매** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-168">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="0a9ea-169">배포가 완료될 때가지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-169">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="0a9ea-170">아래 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-170">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="0a9ea-171">Azure 포털에서 링크가 열렸으면 일부 설정에 대한 값을 입력해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-171">Once the link has opened in the Azure portal, you must enter values for some of the settings:</span></span>
   * <span data-ttu-id="0a9ea-172">**리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-public-dmz-wl-rg`를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-172">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-public-dmz-wl-rg` in the text box.</span></span>
   * <span data-ttu-id="0a9ea-173">**위치** 드롭다운 상자에서 하위 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-173">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="0a9ea-174">**템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-174">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="0a9ea-175">사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-175">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="0a9ea-176">**구매** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-176">Click the **Purchase** button.</span></span>
6. <span data-ttu-id="0a9ea-177">배포가 완료될 때가지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-177">Wait for the deployment to complete.</span></span>
7. <span data-ttu-id="0a9ea-178">아래 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-178">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. <span data-ttu-id="0a9ea-179">Azure 포털에서 링크가 열렸으면 일부 설정에 대한 값을 입력해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-179">Once the link has opened in the Azure portal, you must enter values for some of the settings:</span></span>
   * <span data-ttu-id="0a9ea-180">**리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **기존 사용**을 선택하고 텍스트 상자에 `ra-public-dmz-network-rg`를 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-180">The **Resource group** name is already defined in the parameter file, so select **Use Existing** and enter `ra-public-dmz-network-rg` in the text box.</span></span>
   * <span data-ttu-id="0a9ea-181">**위치** 드롭다운 상자에서 하위 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-181">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="0a9ea-182">**템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-182">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="0a9ea-183">사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-183">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="0a9ea-184">**구매** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-184">Click the **Purchase** button.</span></span>
9. <span data-ttu-id="0a9ea-185">배포가 완료될 때가지 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-185">Wait for the deployment to complete.</span></span>
10. <span data-ttu-id="0a9ea-186">매개 변수 파일에는 모든 VM에 대한 하드 코딩된 관리자 사용자 이름 및 암호가 포함되어 있는데, 이 둘 모두를 즉시 변경할 것을 권장합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-186">The parameter files include hard-coded administrator user name and password for all VMs, and it is strongly recommended that you immediately change both.</span></span> <span data-ttu-id="0a9ea-187">배포의 각 VM을 Azure Portal에서 선택한 후 **지원 + 문제 해결** 블레이드에서 **암호 재설정**을 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-187">For each VM in the deployment, select it in the Azure portal and then click  **Reset password** in the **Support + troubleshooting** blade.</span></span> <span data-ttu-id="0a9ea-188">**모드** 드롭다운 상자에서 **암호 재설정**을 선택한 후 새 **사용자 이름** 및 **암호**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-188">Select **Reset password** in the **Mode** drop down box, then select a new **User name** and **Password**.</span></span> <span data-ttu-id="0a9ea-189">**업데이트** 단추를 클릭하여 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="0a9ea-189">Click the **Update** button to save.</span></span>


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "하이브리드 네트워크 아키텍처 보안"
