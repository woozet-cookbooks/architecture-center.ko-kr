---
title: Azure와 인터넷 간의 DMZ 구현
description: Azure에서 인터넷 액세스로 보안 하이브리드 네트워크 아키텍처를 구현하는 방법입니다.
author: telmosampaio
ms.date: 07/02/2018
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: 7a062d2394ae8b3bd1b17c19cbdf512327f9a766
ms.sourcegitcommit: 9b459f75254d97617e16eddd0d411d1f80b7fe90
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/03/2018
ms.locfileid: "37403150"
---
# <a name="dmz-between-azure-and-the-internet"></a><span data-ttu-id="d8439-103">Azure와 인터넷 간의 DMZ</span><span class="sxs-lookup"><span data-stu-id="d8439-103">DMZ between Azure and the Internet</span></span>

<span data-ttu-id="d8439-104">이 참조 아키텍처는 온-프레미스 네트워크를 Azure로 확장하고 인터넷 트래픽을 수락하는 보안 하이브리드 네트워크를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure and also accepts Internet traffic.</span></span> [<span data-ttu-id="d8439-105">**이 솔루션을 배포합니다**.</span><span class="sxs-lookup"><span data-stu-id="d8439-105">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="d8439-106">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="d8439-106">[![0]][0]</span></span> 

<span data-ttu-id="d8439-107">*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="d8439-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="d8439-108">이 참조 아키텍처는 [Azure와 온-프레미스 데이터 센터 간 DMZ 구현][implementing-a-secure-hybrid-network-architecture]에 설명된 아키텍처를 확장합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-108">This reference architecture extends the architecture described in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture].</span></span> <span data-ttu-id="d8439-109">이 참조 아키텍처는 온-프레미스 네트워크로부터 오는 트래픽을 처리하는 사설 DMZ 외에 인터넷 트래픽을 처리하는 공용 DMZ를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-109">It adds a public DMZ that handles Internet traffic, in addition to the private DMZ that handles traffic from the on-premises network</span></span> 

<span data-ttu-id="d8439-110">일반적으로 이 아키텍처는 다음과 같은 용도로 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-110">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="d8439-111">워크로드의 일부는 온-프레미스에서, 일부는 Azure에서 실행되는 하이브리드 응용 프로그램</span><span class="sxs-lookup"><span data-stu-id="d8439-111">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
* <span data-ttu-id="d8439-112">온-프레미스 및 인터넷으로부터 오는 트래픽을 라우팅하는 Azure 인프라</span><span class="sxs-lookup"><span data-stu-id="d8439-112">Azure infrastructure that routes incoming traffic from on-premises and the Internet.</span></span>

## <a name="architecture"></a><span data-ttu-id="d8439-113">아키텍처</span><span class="sxs-lookup"><span data-stu-id="d8439-113">Architecture</span></span>

<span data-ttu-id="d8439-114">이 아키텍처는 다음 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-114">The architecture consists of the following components.</span></span>

* <span data-ttu-id="d8439-115">**공용 IP 주소(PIP)**.</span><span class="sxs-lookup"><span data-stu-id="d8439-115">**Public IP address (PIP)**.</span></span> <span data-ttu-id="d8439-116">공용 끝점의 IP 주소.</span><span class="sxs-lookup"><span data-stu-id="d8439-116">The IP address of the public endpoint.</span></span> <span data-ttu-id="d8439-117">인터넷에 연결된 외부 사용자는 이 주소를 통해 시스템에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-117">External users connected to the Internet can access the system through this address.</span></span>
* <span data-ttu-id="d8439-118">**NVA(네트워크 가상 어플라이언스)**.</span><span class="sxs-lookup"><span data-stu-id="d8439-118">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="d8439-119">이 아키텍처는 인터넷에서 오는 트래픽을 위한 별도의 NVA 풀을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-119">This architecture includes a separate pool of NVAs for traffic originating on the Internet.</span></span>
* <span data-ttu-id="d8439-120">**Azure Load Balancer**.</span><span class="sxs-lookup"><span data-stu-id="d8439-120">**Azure load balancer**.</span></span> <span data-ttu-id="d8439-121">인터넷에서 오는 모든 요청은 부하 분산 장치를 통과하고 공용 DMZ 내 NVA로 분산됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-121">All incoming requests from the Internet pass through the load balancer and are distributed to the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="d8439-122">**공용 DMZ 인바운드 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="d8439-122">**Public DMZ inbound subnet**.</span></span> <span data-ttu-id="d8439-123">이 서브넷은 Azure Load Balancer로부터 오는 요청을 수용합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-123">This subnet accepts requests from the Azure load balancer.</span></span> <span data-ttu-id="d8439-124">들어오는 요청은 공용 DMZ 내 NVA 중 하나로 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-124">Incoming requests are passed to one of the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="d8439-125">**공용 DMZ 아웃바운드 서브넷**.</span><span class="sxs-lookup"><span data-stu-id="d8439-125">**Public DMZ outbound subnet**.</span></span> <span data-ttu-id="d8439-126">NVA에 의해 승인된 요청은 이 서브넷을 거쳐 웹 계층을 위한 내부 부하 분산 장치로 전달됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-126">Requests that are approved by the NVA pass through this subnet to the internal load balancer for the web tier.</span></span>

## <a name="recommendations"></a><span data-ttu-id="d8439-127">권장 사항</span><span class="sxs-lookup"><span data-stu-id="d8439-127">Recommendations</span></span>

<span data-ttu-id="d8439-128">대부분의 시나리오의 경우 다음 권장 사항을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-128">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="d8439-129">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-129">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="nva-recommendations"></a><span data-ttu-id="d8439-130">NVA 권장 사항</span><span class="sxs-lookup"><span data-stu-id="d8439-130">NVA recommendations</span></span>

<span data-ttu-id="d8439-131">인터넷에서 발생하는 트래픽에 대해 하나의 NVA 집합을 사용하고, 온-프레미스에서 발생하는 트래픽에 대해 다른 NVA 집합을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-131">Use one set of NVAs for traffic originating on the Internet, and another for traffic originating on-premises.</span></span> <span data-ttu-id="d8439-132">둘 다에 대해 하나의 NVA 집합만 사용하면 두 네트워크 트래픽 집합 사이에 보안 경계가 없어지므로 보안 위험이 초래됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-132">Using only one set of NVAs for both is a security risk, because it provides no security perimeter between the two sets of network traffic.</span></span> <span data-ttu-id="d8439-133">별도 NVA를 사용하면 보안 규칙을 검사하는 복잡성이 줄어들고 들어오는 네트워크 요청에 해당하는 규칙이 명확해집니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-133">Using separate NVAs reduces the complexity of checking security rules, and makes it clear which rules correspond to each incoming network request.</span></span> <span data-ttu-id="d8439-134">하나의 NVA 집합은 인터넷 트래픽에만 적용되는 규칙을 수행하고 다른 하나의 NVA 집합은 온-프레미스 트래픽에만 적용되는 규칙을 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-134">One set of NVAs implements rules for Internet traffic only, while another set of NVAs implement rules for on-premises traffic only.</span></span>

<span data-ttu-id="d8439-135">NVA 수준에서 응용 프로그램 연결을 종료하고 백 엔드 계층 호환성을 유지하기 위해 레이어7 NVA를 포함시킵니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-135">Include a layer-7 NVA to terminate application connections at the NVA level and maintain compatibility with the backend tiers.</span></span> <span data-ttu-id="d8439-136">이를 통해 백 엔드 계층으로부터의 응답 트래픽이 NVA를 통해 돌아오는 대칭 연결이 보장됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-136">This guarantees symmetric connectivity where response traffic from the backend tiers returns through the NVA.</span></span>  

### <a name="public-load-balancer-recommendations"></a><span data-ttu-id="d8439-137">공용 부하 분산 장치 권장 사항</span><span class="sxs-lookup"><span data-stu-id="d8439-137">Public load balancer recommendations</span></span>

<span data-ttu-id="d8439-138">높은 확장성과 가용성을 위해 공용 DMZ NVA를 하나의 [가용성 집합][availability-set] 내에 배포하고 [인터넷 연결 부하 분산 장치][load-balancer]를 사용하여 인터넷 요청을 가용성 집합 내 NVA에 분산시킵니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-138">For scalability and availability, deploy the public DMZ NVAs in an [availability set][availability-set] and use an [Internet facing load balancer][load-balancer] to distribute Internet requests across the NVAs in the availability set.</span></span>  

<span data-ttu-id="d8439-139">부하 분산 장치가 인터넷 트래픽에 필요한 포트로만 요청을 받도록 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-139">Configure the load balancer to accept requests only on the ports necessary for Internet traffic.</span></span> <span data-ttu-id="d8439-140">예를 들어, 인바운드 HTTP 요청은 포트 80으로 제한하고 아웃바운드 HTTPS 요청은 포트 443으로 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-140">For example, restrict inbound HTTP requests to port 80 and inbound HTTPS requests to port 443.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="d8439-141">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="d8439-141">Scalability considerations</span></span>

<span data-ttu-id="d8439-142">처음에는 아키텍처가 공용 DMZ 내 단일 NVA를 요구하지만, 처음부터 공용 DMZ 앞에 부하 분산 장치를 배치할 것을 권장합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-142">Even if your architecture initially requires a single NVA in the public DMZ, we recommend putting a load balancer in front of the public DMZ from the beginning.</span></span> <span data-ttu-id="d8439-143">이를 통해 향후 필요한 경우 여러 NVA로 보다 수월하게 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-143">That will make it easier to scale to multiple NVAs in the future, if needed.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="d8439-144">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="d8439-144">Availability considerations</span></span>

<span data-ttu-id="d8439-145">인터넷 연결 부하 분산 장치는 공용 DMZ 수신 서브넷 내의 각 NVA가 [상태 프로브][lb-probe]를 구현하도록 요구합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-145">The Internet facing load balancer requires each NVA in the public DMZ inbound subnet to implement a [health probe][lb-probe].</span></span> <span data-ttu-id="d8439-146">이 끝점에서 응답하지 않는 상태 프로브는 사용 불가능한 것으로 간주되어 부하 분산 장치가 요청을 동일 가용성 집합 내 다른 NVA로 전달하게 됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-146">A health probe that fails to respond on this endpoint is considered to be unavailable, and the load balancer will direct requests to other NVAs in the same availability set.</span></span> <span data-ttu-id="d8439-147">모든 NVA가 응답하지 않는 경우 응용 프로그램이 중지되므로 정상적인 NVA 인스턴스 수가 정의된 한계 아래로 내려가면 DevOps에 알리도록 모니터링 기능을 구성하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-147">Note that if all NVAs fail to respond, your application will fail, so it's important to have monitoring configured to alert DevOps when the number of healthy NVA instances falls below a defined threshold.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="d8439-148">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="d8439-148">Manageability considerations</span></span>

<span data-ttu-id="d8439-149">공용 DMZ 내 NVA에 대한 모든 모니터링 및 관리는 관리 서브넷의 점프박스를 통해 수행되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-149">All monitoring and management for the NVAs in the public DMZ should be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="d8439-150">[Azure와 온-프레미스 데이터 센터 간 DMZ 구현][implementing-a-secure-hybrid-network-architecture]에서 언급되었듯이, 온-프레미스 네트워크로부터 게이트웨이를 거쳐 점프박스로 이어지는 단일의 네트워크 루트를 정의하여 액세스를 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-150">As discussed in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture], define a single network route from the on-premises network through the gateway to the jumpbox, in order to restrict access.</span></span>

<span data-ttu-id="d8439-151">온-프레미스 네트워크에서 Azure로의 게이트웨이 연결이 중단되는 경우에도 공용 IP 주소를 배포하고 점프박스에 추가한 후 인터넷으로부터 로그인하여 점프박스에 액세스할 수 있습니다</span><span class="sxs-lookup"><span data-stu-id="d8439-151">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and logging in from the Internet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="d8439-152">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="d8439-152">Security considerations</span></span>

<span data-ttu-id="d8439-153">이 참조 아키텍처는 여러 수준의 보안을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-153">This reference architecture implements multiple levels of security:</span></span>

* <span data-ttu-id="d8439-154">인터넷 연결 부하 분산 장치는 해당 응용 프로그램에 필요한 포트를 통해서만 수신 공용 DMZ 서브넷 내 NVA로 요청을 전달합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-154">The Internet facing load balancer directs requests to the NVAs in the inbound public DMZ subnet, and only on the ports necessary for the application.</span></span>
* <span data-ttu-id="d8439-155">인바운드 및 아웃바운드 공용 DMZ 서브넷을 위한 NSG(네트워크 보안 그룹) 규칙을 통해 NSG 규칙을 충족하지 않는 요청을 차단함으로써 NVA를 위험으로부터 보호할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-155">The NSG rules for the inbound and outbound public DMZ subnets prevent the NVAs from being compromised, by blocking requests that fall outside of the NSG rules.</span></span>
* <span data-ttu-id="d8439-156">NVA를 위한 NAT 라우팅 구성에 따라 포트 80과 포트 443으로 들어오는 요청을 웹 계층의 부하 분산 장치로 전달하고 그 외 다른 모든 포트로 오는 요청은 무시합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-156">The NAT routing configuration for the NVAs directs incoming requests on port 80 and port 443 to the web tier load balancer, but ignores requests on all other ports.</span></span>

<span data-ttu-id="d8439-157">모든 포트로 오는 모든 요청에 대한 로그를 기록해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-157">You should log all incoming requests on all ports.</span></span> <span data-ttu-id="d8439-158">예상되는 매개변수 범위를 벗어나는 요청은 침입 시도를 의미할 수 있으므로 이에 관심을 기울여 정기적인 로그 감사를 실시합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-158">Regularly audit the logs, paying attention to requests that fall outside of expected parameters, as these may indicate intrusion attempts.</span></span>


## <a name="deploy-the-solution"></a><span data-ttu-id="d8439-159">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="d8439-159">Deploy the solution</span></span>

<span data-ttu-id="d8439-160">이러한 권장 사항을 구현하는 참조 아키텍처 배포는 [GitHub][github-folder]를 통해 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-160">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="d8439-161">필수 조건</span><span class="sxs-lookup"><span data-stu-id="d8439-161">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a><span data-ttu-id="d8439-162">리소스 배포</span><span class="sxs-lookup"><span data-stu-id="d8439-162">Deploy resources</span></span>

1. <span data-ttu-id="d8439-163">참조 아키텍처 GitHub 리포지토리의 `/dmz/secure-vnet-hybrid` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-163">Navigate to the `/dmz/secure-vnet-hybrid` folder of the reference architectures GitHub repository.</span></span>

2. <span data-ttu-id="d8439-164">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="d8439-164">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. <span data-ttu-id="d8439-165">다음 명령 실행:</span><span class="sxs-lookup"><span data-stu-id="d8439-165">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a><span data-ttu-id="d8439-166">온-프레미스와 Azure 게이트웨이 연결</span><span class="sxs-lookup"><span data-stu-id="d8439-166">Connect the on-premises and Azure gateways</span></span>

<span data-ttu-id="d8439-167">이 단계에서는 두 개의 로컬 네트워크 게이트웨이를 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-167">In this step, you will connect the two local network gateways.</span></span>

1. <span data-ttu-id="d8439-168">Azure Portal에서 만든 리소스 그룹으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-168">In the Azure Portal, navigate to the resource group that you created.</span></span> 

2. <span data-ttu-id="d8439-169">`ra-vpn-vgw-pip`라는 리소스를 찾고 **개요** 블레이드에 표시된 IP 주소를 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-169">Find the resource named `ra-vpn-vgw-pip` and copy the IP address shown in the **Overview** blade.</span></span>

3. <span data-ttu-id="d8439-170">`onprem-vpn-lgw`라는 리소스를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-170">Find the resource named `onprem-vpn-lgw`.</span></span>

4. <span data-ttu-id="d8439-171">**구성** 블레이드를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-171">Click the **Configuration** blade.</span></span> <span data-ttu-id="d8439-172">**IP 주소** 아래에서 2단계의 IP 주소에 붙여넣습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-172">Under **IP address**, paste in the IP address from step 2.</span></span>

    ![](./images/local-net-gw.png)

5. <span data-ttu-id="d8439-173">**저장**을 클릭하고 작업이 완료되기를 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-173">Click **Save** and wait for the operation to complete.</span></span> <span data-ttu-id="d8439-174">5분 정도 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-174">It can take about 5 minutes.</span></span>

6. <span data-ttu-id="d8439-175">`onprem-vpn-gateway1-pip`라는 리소스를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-175">Find the resource named `onprem-vpn-gateway1-pip`.</span></span> <span data-ttu-id="d8439-176">**개요** 블레이드에 표시된 IP 주소를 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-176">Copy the IP address shown in the **Overview** blade.</span></span>

7. <span data-ttu-id="d8439-177">`ra-vpn-lgw`라는 리소스를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-177">Find the resource named `ra-vpn-lgw`.</span></span> 

8. <span data-ttu-id="d8439-178">**구성** 블레이드를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-178">Click the **Configuration** blade.</span></span> <span data-ttu-id="d8439-179">**IP 주소** 아래에서 6단계의 IP 주소에 붙여넣습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-179">Under **IP address**, paste in the IP address from step 6.</span></span>

9. <span data-ttu-id="d8439-180">**저장**을 클릭하고 작업이 완료되기를 기다립니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-180">Click **Save** and wait for the operation to complete.</span></span>

10. <span data-ttu-id="d8439-181">연결을 확인하려면 각 게이트웨이에 대한 **연결** 블레이드로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-181">To verify the connection, go to the **Connections** blade for each gateway.</span></span> <span data-ttu-id="d8439-182">상태가 **연결돼** 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-182">The status should be **Connected**.</span></span>

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a><span data-ttu-id="d8439-183">네트워크 트래픽이 웹 계층에 도달하는지 확인</span><span class="sxs-lookup"><span data-stu-id="d8439-183">Verify that network traffic reaches the web tier</span></span>

1. <span data-ttu-id="d8439-184">Azure Portal에서 만든 리소스 그룹으로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-184">In the Azure Portal, navigate to the resource group that you created.</span></span> 

2. <span data-ttu-id="d8439-185">공용 DMZ 앞의 부하 분산 장치인 `pub-dmz-lb`라는 리소스를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-185">Find the resource named `pub-dmz-lb`, which is the load balancer in front of the public DMZ.</span></span> 

3. <span data-ttu-id="d8439-186">**개요** 블레이드에서 공용 IP 주소를 복사하고 웹 브라우저에서 이 주소를 엽니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-186">Copy the public IP addess from the **Overview** blade and open this address in a web browser.</span></span> <span data-ttu-id="d8439-187">기본 Apache2 서버 홈 페이지가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-187">You should see the default Apache2 server home page.</span></span>

4. <span data-ttu-id="d8439-188">개인 DMZ 앞의 부하 분산 장치인 `int-dmz-lb`라는 리소스를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-188">Find the resource named `int-dmz-lb`, which is the load balancer in front of the private DMZ.</span></span> <span data-ttu-id="d8439-189">**개요** 블레이드에서 개인 IP 주소를 복사합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-189">Copy the private IP address from the **Overview** blade.</span></span>

5. <span data-ttu-id="d8439-190">`jb-vm1`이라는 VM을 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-190">Find the VM named `jb-vm1`.</span></span> <span data-ttu-id="d8439-191">**연결**을 클릭하고 원격 데스크톱을 사용하여 VM에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-191">Click **Connect** and use Remote Desktop to connect to the VM.</span></span> <span data-ttu-id="d8439-192">사용자 이름 및 암호는 onprem.json 파일에서 지정됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-192">The user name and password are specified in the onprem.json file.</span></span>

6. <span data-ttu-id="d8439-193">원격 데스크톱 세션에서 웹 브라우저를 열고 4단계의 IP 주소로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-193">From the Remote Desktop Session, open a web browser and navigate to the IP address from step 4.</span></span> <span data-ttu-id="d8439-194">기본 Apache2 서버 홈 페이지가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="d8439-194">You should see the default Apache2 server home page.</span></span>

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "하이브리드 네트워크 아키텍처 보안"
