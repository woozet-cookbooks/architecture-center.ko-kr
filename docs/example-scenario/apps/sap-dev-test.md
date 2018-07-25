---
title: 개발/테스트 작업에 대한 SAP
description: 개발/테스트 환경에 대한 SAP 시나리오입니다.
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: 675a5cb4b1ee4001ca50d24c145ce1a177f90da4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060964"
---
# <a name="sap-for-devtest-workloads"></a><span data-ttu-id="be249-103">개발/테스트 작업에 대한 SAP</span><span class="sxs-lookup"><span data-stu-id="be249-103">SAP for dev/test workloads</span></span>

<span data-ttu-id="be249-104">이 예제에서는 Azure의 Windows 또는 Linux 환경에서 SAP NetWeaver의 개발/테스트 구현을 실행하는 방법에 대한 지침을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="be249-104">This example provides guidance for how to run a dev/test implementation of SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="be249-105">사용되는 데이터베이스는 지원되는 모든 DBMS(SAP HANA가 아님)를 의미하는 SAP 용어인 AnyDB입니다.</span><span class="sxs-lookup"><span data-stu-id="be249-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="be249-106">이 아키텍처는 비프로덕션 환경에 대해 설계되었기 때문에 단일 VM(가상 머신)으로 배포되며 크기는 조직의 요구에 맞게 변경할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="be249-107">프로덕션 사용 사례의 경우 아래에서 사용할 수 있는 SAP 참조 아키텍처를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="be249-107">For production use cases review the SAP reference architectures available below:</span></span>

* <span data-ttu-id="be249-108">[AnyDB용 SAP NetWeaver][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="be249-108">[SAP netweaver for AnyDB][sap-netweaver]</span></span>
* <span data-ttu-id="be249-109">[SAP S/4Hana][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="be249-109">[SAP S/4Hana][sap-hana]</span></span>
* <span data-ttu-id="be249-110">[Azure의 SAP(대규모 인스턴스)][sap-large]</span><span class="sxs-lookup"><span data-stu-id="be249-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="be249-111">관련 사용 사례</span><span class="sxs-lookup"><span data-stu-id="be249-111">Related use cases</span></span>

<span data-ttu-id="be249-112">이 시나리오에 적합한 사용 사례는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-112">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="be249-113">중요하지 않은 비생산적 SAP 워크로드(샌드박스, 개발, 테스트, 품질 보증)</span><span class="sxs-lookup"><span data-stu-id="be249-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
* <span data-ttu-id="be249-114">중요하지 않은 단일 SAP 비즈니스 워크로드</span><span class="sxs-lookup"><span data-stu-id="be249-114">Non-critical SAP business one workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="be249-115">아키텍처</span><span class="sxs-lookup"><span data-stu-id="be249-115">Architecture</span></span>

![다이어그램](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

<span data-ttu-id="be249-117">이 시나리오에서는 단일 가상 머신에 단일 SAP 시스템 데이터베이스와 SAP 응용 프로그램 서버를 프로비전하는 방법에 대해 설명하며, 시나리오를 통한 데이터 흐름은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-117">This scenario covers the provision of a single SAP system database and SAP application Server on a single virtual machine, the data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="be249-118">프레젠테이션 계층의 고객은 온-프레미스에서 자신의 SAP GUI 또는 다른 사용자 인터페이스(Internet Explorer, Excel 또는 다른 웹 응용 프로그램)를 사용하여 Azure 기반 SAP 시스템에 액세스합니다.</span><span class="sxs-lookup"><span data-stu-id="be249-118">Customers from the Presentation Tier use their SAP gui, or other user interfaces (Internet Explorer, Excel or other web application) on premise to access the Azure based SAP system.</span></span>
2. <span data-ttu-id="be249-119">연결은 설정된 ExpressRoute를 사용하여 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="be249-119">Connectivity is provided through the use of the established Express Route.</span></span> <span data-ttu-id="be249-120">ExpressRoute는 Azure의 ExpressRoute 게이트웨이에서 종료됩니다.</span><span class="sxs-lookup"><span data-stu-id="be249-120">The Express Route is terminated in Azure at the Express Route Gateway.</span></span> <span data-ttu-id="be249-121">네트워크 트래픽은 ExpressRoute 게이트웨이를 통해 게이트웨이 서브넷으로 라우팅되고, 게이트웨이 서브넷에서 응용 프로그램 계층 스포크 서브넷([허브-스포크][hub-spoke] 패턴 참조)으로 라우팅되며, 네트워크 보안 게이트웨이를 통해 SAP 응용 프로그램 가상 머신으로 라우팅됩니다.</span><span class="sxs-lookup"><span data-stu-id="be249-121">Network traffic routes through the Express Route gateway to the Gateway Subnet and from the gateway subnet to the Application Tier Spoke subnet (see the [hub-spoke][hub-spoke] pattern) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="be249-122">ID 관리 서버는 인증 서비스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="be249-122">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="be249-123">점프 박스는 로컬 관리 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="be249-123">The Jump Box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="be249-124">구성 요소</span><span class="sxs-lookup"><span data-stu-id="be249-124">Components</span></span>

* <span data-ttu-id="be249-125">[리소스 그룹](/azure/azure-resource-manager/resource-group-overview#resource-groups)은 Azure 리소스에 대한 논리 컨테이너입니다.</span><span class="sxs-lookup"><span data-stu-id="be249-125">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) is a logical container for Azure resources.</span></span>
* <span data-ttu-id="be249-126">[가상 네트워크](/azure/virtual-network/virtual-networks-overview)는 Azure 내에서 네트워크 통신의 기초입니다.</span><span class="sxs-lookup"><span data-stu-id="be249-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) is the basis of network communications within Azure</span></span>
* <span data-ttu-id="be249-127">[가상 머신](/azure/virtual-machines/windows/overview) Azure Virtual Machines는 Windows 또는 Linux 서버를 사용하여 안전하고 가상화된 주문형 대규모 인프라를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="be249-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server</span></span>
* <span data-ttu-id="be249-128">[ExpressRoute](/azure/expressroute/expressroute-introduction)를 사용하면 연결 공급자가 지원하는 개인 연결을 통해 온-프레미스 네트워크를 Microsoft 클라우드로 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-128">[Express Route](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
* <span data-ttu-id="be249-129">[네트워크 보안 그룹](/azure/virtual-network/security-overview)을 사용하면 네트워크 트래픽을 가상 네트워크의 리소스로 제한할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="be249-130">네트워크 보안 그룹에는 원본 또는 대상 IP 주소, 포트 및 프로토콜에 따라 인바운드 또는 아웃바운드 네트워크 트래픽을 허용하거나 거부하는 보안 규칙 목록이 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span> 

## <a name="considerations"></a><span data-ttu-id="be249-131">고려 사항</span><span class="sxs-lookup"><span data-stu-id="be249-131">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="be249-132">가용성</span><span class="sxs-lookup"><span data-stu-id="be249-132">Availability</span></span>

 <span data-ttu-id="be249-133">Microsoft는 단일 VM 인스턴스에 대한 SLA(서비스 수준 계약)를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="be249-133">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="be249-134">Virtual Machines와 관련된 Microsoft Azure 서비스 수준 계약에 대한 자세한 내용은 [Virtual Machines에 대한 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="be249-134">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="be249-135">확장성</span><span class="sxs-lookup"><span data-stu-id="be249-135">Scalability</span></span>

<span data-ttu-id="be249-136">확장 가능한 솔루션 설계에 대한 일반적인 지침은 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="be249-136">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="be249-137">보안</span><span class="sxs-lookup"><span data-stu-id="be249-137">Security</span></span>

<span data-ttu-id="be249-138">보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="be249-138">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="be249-139">복원력</span><span class="sxs-lookup"><span data-stu-id="be249-139">Resiliency</span></span>

<span data-ttu-id="be249-140">복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 응용 프로그램 디자인][resiliency]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="be249-140">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="be249-141">가격</span><span class="sxs-lookup"><span data-stu-id="be249-141">Pricing</span></span>

<span data-ttu-id="be249-142">이 시나리오를 실행하는 데 드는 비용을 알아보려면 비용 계산기에서 모든 서비스를 미리 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="be249-142">Explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span>  <span data-ttu-id="be249-143">특정 사용 사례에 대한 가격 변동을 확인하려면 필요한 트래픽에 맞게 변수를 적절하게 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="be249-143">To see how the pricing would change for your particular use case change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="be249-144">가져오는 데 필요한 트래픽 양을 기준으로 제공한 네 가지 샘플 비용 프로필은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-144">We have provided four sample cost profiles based on amount of traffic you expect to get:</span></span>

|<span data-ttu-id="be249-145">크기</span><span class="sxs-lookup"><span data-stu-id="be249-145">Size</span></span>|<span data-ttu-id="be249-146">SAP</span><span class="sxs-lookup"><span data-stu-id="be249-146">SAPs</span></span>|<span data-ttu-id="be249-147">VM 유형</span><span class="sxs-lookup"><span data-stu-id="be249-147">VM Type</span></span>|<span data-ttu-id="be249-148">Storage</span><span class="sxs-lookup"><span data-stu-id="be249-148">Storage</span></span>|<span data-ttu-id="be249-149">Azure 요금 계산기</span><span class="sxs-lookup"><span data-stu-id="be249-149">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="be249-150">작음</span><span class="sxs-lookup"><span data-stu-id="be249-150">Small</span></span>|<span data-ttu-id="be249-151">8000</span><span class="sxs-lookup"><span data-stu-id="be249-151">8000</span></span>|<span data-ttu-id="be249-152">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="be249-152">D8s_v3</span></span>|<span data-ttu-id="be249-153">2xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="be249-153">2xP20, 1xP10</span></span>|[<span data-ttu-id="be249-154">소형</span><span class="sxs-lookup"><span data-stu-id="be249-154">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="be249-155">중간</span><span class="sxs-lookup"><span data-stu-id="be249-155">Medium</span></span>|<span data-ttu-id="be249-156">16000</span><span class="sxs-lookup"><span data-stu-id="be249-156">16000</span></span>|<span data-ttu-id="be249-157">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="be249-157">D16s_v3</span></span>|<span data-ttu-id="be249-158">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="be249-158">3xP20, 1xP10</span></span>|[<span data-ttu-id="be249-159">중형</span><span class="sxs-lookup"><span data-stu-id="be249-159">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="be249-160">큰</span><span class="sxs-lookup"><span data-stu-id="be249-160">Large</span></span>|<span data-ttu-id="be249-161">32000</span><span class="sxs-lookup"><span data-stu-id="be249-161">32000</span></span>|<span data-ttu-id="be249-162">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="be249-162">E32s_v3</span></span>|<span data-ttu-id="be249-163">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="be249-163">3xP20, 1xP10</span></span>|[<span data-ttu-id="be249-164">대형</span><span class="sxs-lookup"><span data-stu-id="be249-164">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="be249-165">초대형</span><span class="sxs-lookup"><span data-stu-id="be249-165">Extra Large</span></span>|<span data-ttu-id="be249-166">64000</span><span class="sxs-lookup"><span data-stu-id="be249-166">64000</span></span>|<span data-ttu-id="be249-167">M64s</span><span class="sxs-lookup"><span data-stu-id="be249-167">M64s</span></span>|<span data-ttu-id="be249-168">4xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="be249-168">4xP20, 1xP10</span></span>|[<span data-ttu-id="be249-169">초대형</span><span class="sxs-lookup"><span data-stu-id="be249-169">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

<span data-ttu-id="be249-170">참고: 가격 책정은 지표이며, VM 및 저장소 비용을 나타냅니다(네트워킹, 백업 저장소 및 데이터 수신/송신 비용은 제외).</span><span class="sxs-lookup"><span data-stu-id="be249-170">Note: pricing is a guide and only indicates the VMs and storage costs (excludes, networking, backup storage and data ingress/egress charges).</span></span>

* <span data-ttu-id="be249-171">[소형](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): 소형 시스템은 8개 vCPU, 32GB RAM 및 200GB 임시 저장소가 있는 D8s_v3 VM 유형으로 구성되며, 2개 512GB 및 1개 128GB 프리미엄 저장소 디스크도 추가로 있습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-171">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32GB RAM and 200GB temp storage, additionally two 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="be249-172">[중형](https://azure.com/e/465bd07047d148baab032b2f461550cd): 중형 시스템은 16개 vCPU, 64GB RAM 및 400GB 임시 저장소가 있는 D16s_v3 VM 유형으로 구성되며, 3개 512GB 및 1개 128GB 프리미엄 저장소 디스크도 추가로 있습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-172">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64GB RAM and 400GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="be249-173">[대형](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): 대형 시스템은 32개 vCPU, 256GB RAM 및 512GB 임시 저장소가 있는 E32s_v3 VM 유형으로 구성되며, 3개 512GB 및 1개 128GB 프리미엄 저장소 디스크도 추가로 있습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-173">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256GB RAM and 512GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="be249-174">[초대형](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): 초대형 시스템은 64개 vCPU, 1,024GB RAM 및 2,000GB 임시 저장소가 있는 M64s VM 유형으로 구성되며, 4개 512GB 및 1개 128GB 프리미엄 저장소 디스크도 추가로 있습니다.</span><span class="sxs-lookup"><span data-stu-id="be249-174">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024GB RAM and 2000GB temp storage, additionally four 512GB and one 128GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="be249-175">배포</span><span class="sxs-lookup"><span data-stu-id="be249-175">Deployment</span></span>

<span data-ttu-id="be249-176">위 시나리오와 비슷한 기본 인프라를 배포하려면 배포 단추를 사용해 주세요.</span><span class="sxs-lookup"><span data-stu-id="be249-176">To deploy the underlying infrastructure similar to the scenario above, please use the deploy button</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="be249-177">\* SAP가 설치되지 않으므로 인프라를 수동으로 구축한 후에 이 작업을 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="be249-177">\* SAP will not be installed, you'll need to do this after the infrastructure is built manually.</span></span>

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke