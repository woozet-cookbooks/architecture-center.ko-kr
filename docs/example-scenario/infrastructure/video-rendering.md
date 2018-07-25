---
title: Azure의 3D 비디오 렌더링
description: Azure Batch 서비스를 사용하여 Azure에서 원시 HPC 워크로드를 실행합니다.
author: adamboeglin
ms.date: 07/13/2018
ms.openlocfilehash: b3af0641642d7ec4b022e8c96f51693eeb0adee4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39061004"
---
# <a name="3d-video-rendering-on-azure"></a><span data-ttu-id="4c50a-103">Azure의 3D 비디오 렌더링</span><span class="sxs-lookup"><span data-stu-id="4c50a-103">3D video rendering on Azure</span></span>

<span data-ttu-id="4c50a-104">3D 렌더링은 공동으로 완료하는 데 상당한 양의 CPU 시간이 필요한 시간 소모적인 프로세스입니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-104">3D rendering is a time consuming process that requires a significant amount of CPU time co complete.</span></span>  <span data-ttu-id="4c50a-105">단일 머신에서 정적 자산으로부터 비디오 파일을 생성하는 프로세스에는 생성하는 비디오의 길이와 복잡성에 따라 몇 시간 또는 며칠이 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-105">On a single machine, the process of generating a video file from static assets can take hours or even days depending on the length and complexity of the video you are producing.</span></span>  <span data-ttu-id="4c50a-106">대부분의 회사에서 이러한 작업을 수행하기 위해 비용이 많이 드는 고사양 데스크톱 컴퓨터를 구입하거나 작업을 제출할 수 있는 대형 렌더 팜에 투자합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-106">Many companies will purchase either expensive high end desktop computers to perform these tasks, or invest in large render farms that they can submit jobs to.</span></span>  <span data-ttu-id="4c50a-107">그러나 Azure Batch를 활용하면 필요할 때 이 기능을 사용할 수 있고, 그렇지 않은 경우 어떠한 자본 투자 없이도 기능 자체를 종료할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-107">However, by taking advantage of Azure Batch, that power is available to you when you need it and shuts itself down when you don't, all without any capital investment.</span></span>

<span data-ttu-id="4c50a-108">Batch는 Windows Server 또는 Linux 계산 노드 중 어느 것을 선택하든지 간에 일관된 관리 환경과 작업 예약을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-108">Batch gives you a consistent management experience and job scheduling, whether you select Windows Server or Linux compute nodes.</span></span> <span data-ttu-id="4c50a-109">Batch를 사용하면 AutoDesk Maya 및 Blender를 포함하여 기존 Windows 또는 Linux 응용 프로그램을 통해 Azure에서 대규모 렌더 작업을 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-109">With Batch, you can use your existing Windows or Linux applications, including AutoDesk Maya and Blender, to run large-scale render jobs in Azure.</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="4c50a-110">관련 사용 사례</span><span class="sxs-lookup"><span data-stu-id="4c50a-110">Related use cases</span></span>

<span data-ttu-id="4c50a-111">이 시나리오는 다음과 비슷한 사용 사례에 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-111">Consider this scenario for these similar use cases:</span></span>

* <span data-ttu-id="4c50a-112">3D 모델링</span><span class="sxs-lookup"><span data-stu-id="4c50a-112">3D Modeling</span></span>
* <span data-ttu-id="4c50a-113">VFX(Visual FX) 렌더링</span><span class="sxs-lookup"><span data-stu-id="4c50a-113">Visual FX (VFX) Rendering</span></span>
* <span data-ttu-id="4c50a-114">비디오 코드 변환</span><span class="sxs-lookup"><span data-stu-id="4c50a-114">Video transcoding</span></span>
* <span data-ttu-id="4c50a-115">이미지 처리, 색 보정 및 크기 조정</span><span class="sxs-lookup"><span data-stu-id="4c50a-115">Image processing, color correction, & resizing</span></span>

## <a name="architecture"></a><span data-ttu-id="4c50a-116">아키텍처</span><span class="sxs-lookup"><span data-stu-id="4c50a-116">Architecture</span></span>

![Azure Batch를 사용하는 클라우드 원시 HPC 솔루션과 관련된 구성 요소 아키텍처에 대한 개요][architecture]

<span data-ttu-id="4c50a-118">이 샘플 시나리오에서는 Azure Batch를 사용하는 경우의 워크플로에 대해 설명하며, 데이터 흐름은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-118">This sample scenario covers the workflow when using Azure Batch, the data flows as follows:</span></span>

1. <span data-ttu-id="4c50a-119">입력 파일과 응용 프로그램을 업로드하여 이러한 파일을 Azure Storage 계정으로 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-119">Upload input files and the applications to process those files to your Azure Storage account</span></span>
2. <span data-ttu-id="4c50a-120">배치 계정에 계산 노드의 Batch 풀을 만들고, 풀에 워크로드를 실행하는 작업을 만들고, 작업에 태스크를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-120">Create a Batch pool of compute nodes in your Batch account, a job to run the workload on the pool, and tasks in the job.</span></span>
3. <span data-ttu-id="4c50a-121">입력 파일과 응용 프로그램을 Batch에 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-121">Download input files and the applications to Batch</span></span>
4. <span data-ttu-id="4c50a-122">태스크 실행 모니터링</span><span class="sxs-lookup"><span data-stu-id="4c50a-122">Monitor task execution</span></span>
5. <span data-ttu-id="4c50a-123">태스크 출력을 업로드합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-123">Upload task output</span></span>
6. <span data-ttu-id="4c50a-124">출력 파일을 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-124">Download output files</span></span>

<span data-ttu-id="4c50a-125">이 프로세스를 간소화하기 위해 [Maya 및 3ds Max용 Batch 플러그 인][batch-plugins]을 사용할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-125">To simplify this process, you could also use the [Batch Plugins for Maya & 3ds Max][batch-plugins]</span></span>

### <a name="components"></a><span data-ttu-id="4c50a-126">구성 요소</span><span class="sxs-lookup"><span data-stu-id="4c50a-126">Components</span></span>

<span data-ttu-id="4c50a-127">Azure Batch는 다음 Azure 기술을 기반으로 합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-127">Azure Batch builds upon the following Azure technologies:</span></span>

* <span data-ttu-id="4c50a-128">[리소스 그룹][resource-groups]은 Azure 리소스에 대한 논리 컨테이너입니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-128">[Resource Groups][resource-groups] is a logical container for Azure resources.</span></span>
* <span data-ttu-id="4c50a-129">[가상 네트워크][vnet]는 헤드 노드 및 계산 리소스 모두에 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-129">[Virtual Networks][vnet] are used to for both the Head Node and Compute resources</span></span>
* <span data-ttu-id="4c50a-130">[저장소][storage] 계정은 동기화 및 데이터 보존에 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-130">[Storage][storage] accounts are used for the synchronisation and data retention</span></span>
* <span data-ttu-id="4c50a-131">[Virtual Machine Scale Sets][vmss]는 CycleCloud에서 계산 리소스에 대해 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-131">[Virtual Machine Scale Sets][vmss] are utilised by CycleCloud for compute resources</span></span>

## <a name="considerations"></a><span data-ttu-id="4c50a-132">고려 사항</span><span class="sxs-lookup"><span data-stu-id="4c50a-132">Considerations</span></span>

### <a name="machine-sizes-available-for-azure-batch"></a><span data-ttu-id="4c50a-133">Azure Batch에 사용 가능한 머신 크기</span><span class="sxs-lookup"><span data-stu-id="4c50a-133">Machine Sizes available for Azure Batch</span></span>
<span data-ttu-id="4c50a-134">대부분의 렌더링 고객은 CPU 능력이 높은 리소스를 선택하지만, VM 확장 집합을 사용하는 다른 워크로드는 VM을 다르게 선택할 수 있으며 여러 요인에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-134">While most rendering customers whill choose resources with high CPU power, other workloads using VM Scale Sets may choose VM's differently and will depend on a number of factors:</span></span>
  - <span data-ttu-id="4c50a-135">응용 프로그램이 메모리에 바인딩되어 실행되고 있나요?</span><span class="sxs-lookup"><span data-stu-id="4c50a-135">Is the Application being run memory bound?</span></span>
  - <span data-ttu-id="4c50a-136">응용 프로그램에서 GPU를 사용해야 하나요?</span><span class="sxs-lookup"><span data-stu-id="4c50a-136">Does the Application need to use GPU's?</span></span> 
  - <span data-ttu-id="4c50a-137">당황스러울 정도의 병렬 작업 유형이거나 긴밀하게 결합된 작업에 대한 Infiniband 연결이 필요한가요?</span><span class="sxs-lookup"><span data-stu-id="4c50a-137">Are the job types embarassingly parallel or require Infiniband connectivity for tightly coupled jobs?</span></span>
  - <span data-ttu-id="4c50a-138">계산 노드에서 저장소에 대한 빠른 I/O가 필요한가요?</span><span class="sxs-lookup"><span data-stu-id="4c50a-138">Require fast I/O to Storage on the Compute Nodes</span></span>

<span data-ttu-id="4c50a-139">Azure에는 위의 응용 프로그램 요구 사항을 모두 처리할 수 있는 광범위한 VM 크기가 있으며, 일부는 HPC에만 관련되지만 가장 작은 크기조차도 효과적인 그리드 구현을 제공하는 데 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-139">Azure has a wide range of VM sizes that can address each and every one of the above application requirements, some are specific to HPC, but even the smallest sizes can be utilised to provide an effective grid implementation:</span></span>

  - <span data-ttu-id="4c50a-140">[HPC VM 크기][compute-hpc] 렌더링의 CPU 바인딩 특성으로 인해 Microsoft는 일반적으로 Azure H 시리즈 VM을 제안합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-140">[HPC VM sizes][compute-hpc] Due to the CPU bound nature of rendering, Microsoft typically suggests the Azure H-Series VM's.</span></span>  <span data-ttu-id="4c50a-141">이러한 VM은 고급 계산 요구 사항에 맞게 특별히 만들어졌으며, 8개 및 16개 코어 vCPU 크기를 사용할 수 있고, DDR4 메모리, SSD 임시 저장소 및 Haswell E5 Intel 기술을 갖추고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-141">These are built specifically available for high end computational needs, have 8 and 16 core vCPU sizes available, and feature DDR4 memory, SSD temporary storage and Haswell E5 Intel technology.</span></span>
  - <span data-ttu-id="4c50a-142">[GPU VM 크기][compute-gpu] GPU 최적화된 VM 크기는 단일 또는 여러 NVIDIA GPU에서 사용할 수 있는 특수한 가상 머신입니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-142">[GPU VM sizes][compute-gpu] GPU optimized VM sizes are specialized virtual machines available with single or multiple NVIDIA GPUs.</span></span> <span data-ttu-id="4c50a-143">이러한 크기는 계산 집약적이며 그래픽 집약적인 시각화 워크로드용으로 설계되었습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-143">These sizes are designed for compute-intensive, graphics-intensive, and visualization workloads.</span></span>
    - <span data-ttu-id="4c50a-144">NC, NCv2, NCv3 및 ND 크기는 CUDA 기반 및 OpenCL 기반 응용 프로그램 및 시뮬레이션, AI, 딥 러닝을 포함하여 계산 집약적이고 네트워크 집약적인 응용 프로그램과 알고리즘에 최적화되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-144">NC, NCv2, NCv3, and ND sizes are optimized for compute-intensive and network-intensive applications and algorithms, including CUDA and OpenCL-based applications and simulations, AI, and Deep Learning.</span></span> <span data-ttu-id="4c50a-145">NV 크기는 OpenGL 및 DirectX와 같은 프레임워크를 활용하는 원격 시각화, 스트리밍, 게임, 인코딩 및 VDI 시나리오에 맞게 최적화되고 설계되었습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-145">NV sizes are optimized and designed for remote visualization, streaming, gaming, encoding, and VDI scenarios utilizing frameworks such as OpenGL and DirectX.</span></span>
  - <span data-ttu-id="4c50a-146">[메모리 최적화된 VM 크기][compute-memory] 더 많은 메모리가 필요한 경우 메모리 최적화된 VM 크기는 더 높은 메모리 대 CPU 비율을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-146">[Memory optmised VM sizes][compute-memory] When more memory is required, the memory optimized VM sizes offer a higher memory-to-CPU ratio.</span></span>
  - <span data-ttu-id="4c50a-147">[범용 VM 크기][compute-general] 범용 VM 크기도 사용할 수 있으며, 균형 잡힌 CPU 대 메모리 비율을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-147">[General purposes VM sizes][compute-general] General purpose VM sizes are also available and provide balanced CPU-to-memory ratio.</span></span>

### <a name="alternatives"></a><span data-ttu-id="4c50a-148">대안</span><span class="sxs-lookup"><span data-stu-id="4c50a-148">Alternatives</span></span>

<span data-ttu-id="4c50a-149">Azure에서 렌더링 환경을 더 자세히 제어해야 하거나 하이브리드 구현이 필요한 경우, CycleCloud 컴퓨팅은 클라우드에서 IaaS 그리드를 오케스트레이션하는 데 도움을 줄 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-149">If you require more control over your rendering environment in Azure or need a hybrid implementation, then CycleCloud computing can help orchestrate an IaaS grid in the cloud.</span></span> <span data-ttu-id="4c50a-150">Azure Batch와 동일한 기본 Azure 기술을 사용하여 IaaS 그리드를 효율적으로 작성하고 유지 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-150">Using the same underlying Azure technologies as Azure Batch, it makes building and maintaining an IaaS grid an efficient process.</span></span> <span data-ttu-id="4c50a-151">자세한 내용을 찾고 디자인 원칙을 알아보려면 다음 링크를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-151">To find out more and learn about the design principles, please use the following link:</span></span>

<span data-ttu-id="4c50a-152">Azure에서 사용할 수 있는 모든 HPC 솔루션에 대한 전체 개요는 [Azure VM을 사용하여 HPC, 일괄 처리 및 큰 계산 솔루션][hpc-alt-solutions] 문서를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4c50a-152">For a complete overview of all the HPC solutions that are available to you in Azure, please see the article [HPC, Batch, and Big Compute solutions using Azure VMs][hpc-alt-solutions]</span></span>

### <a name="availability"></a><span data-ttu-id="4c50a-153">가용성</span><span class="sxs-lookup"><span data-stu-id="4c50a-153">Availability</span></span>

<span data-ttu-id="4c50a-154">Azure Batch 구성 요소에 대한 모니터링은 다양한 서비스, 도구 및 API를 통해 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-154">Monitoring of the Azure Batch components is available through a range of services, tools and APIs.</span></span> <span data-ttu-id="4c50a-155">이에 대해서는 [Batch 솔루션 모니터링][batch-monitor] 문서에서 자세히 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-155">This is discussed further in the [Monitor Batch solutions][batch-monitor] article.</span></span>

### <a name="scalability"></a><span data-ttu-id="4c50a-156">확장성</span><span class="sxs-lookup"><span data-stu-id="4c50a-156">Scalability</span></span>

<span data-ttu-id="4c50a-157">Azure Batch 계정 내의 풀은 수동 개입을 통해 크기 조정하거나 Azure Batch 메트릭에 기반한 수식을 사용하여 자동으로 크기 조정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-157">Pools within an Azure Batch account can either scale through manual intervention or, by using an formula based on Azure Batch metrics, be scaled automatically.</span></span> <span data-ttu-id="4c50a-158">이에 대한 자세한 내용은 [Batch 풀에서 노드의 크기를 조정하는 자동 크기 조정 수식 만들기][batch-scaling] 문서를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4c50a-158">For more information on this, please see the article [Create an automatic scaling formula for scaling nodes in a Batch pool][batch-scaling].</span></span>

### <a name="security"></a><span data-ttu-id="4c50a-159">보안</span><span class="sxs-lookup"><span data-stu-id="4c50a-159">Security</span></span>

<span data-ttu-id="4c50a-160">보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4c50a-160">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="4c50a-161">복원력</span><span class="sxs-lookup"><span data-stu-id="4c50a-161">Resiliency</span></span>

<span data-ttu-id="4c50a-162">Azure Batch에는 현재 장애 조치 기능이 없지만, 계획되지 않은 중단이 발생하는 경우 가용성을 보장하기 위해 다음 단계를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-162">While there is currently no failover capability in Azure Batch, we recommend using the following steps to ensure availbility in case of an unplanned outage:</span></span>

* <span data-ttu-id="4c50a-163">대체 저장소 계정을 사용하여 Azure Batch 계정을 대체 Azure 위치에 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-163">Create a Azure Batch account in an alternate Azure location with an alternate Storage Account</span></span>
* <span data-ttu-id="4c50a-164">동일한 이름을 사용하여 0개 노드가 할당된 동일한 노드 풀을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-164">Create the same node pools with the same name, with 0 nodes allocated</span></span>
* <span data-ttu-id="4c50a-165">응용 프로그램이 만들어지고, 대체 저장소 계정으로 업데이트되었는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-165">Ensure Applications are created and updated to the alternate Storage Account</span></span>
* <span data-ttu-id="4c50a-166">입력 파일을 업로드하고, 대체 Azure Batch 계정에 작업을 제출합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-166">Upload input files and submit jobs to the alternate Azure Batch account</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="4c50a-167">시나리오 배포</span><span class="sxs-lookup"><span data-stu-id="4c50a-167">Deploy this scenario</span></span>

### <a name="creating-an-azure-batch-account-and-pools-manually"></a><span data-ttu-id="4c50a-168">수동으로 Azure Batch 계정 및 풀 만들기</span><span class="sxs-lookup"><span data-stu-id="4c50a-168">Creating an Azure Batch account and pools manually</span></span>

<span data-ttu-id="4c50a-169">이 샘플 시나리오는 Azure Batch 작동 방법을 알아보는 데 도움이 되며, Azure Batch Labs를 자신의 고객을 위해 개발할 수 있는 예제 SaaS 솔루션으로 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-169">This sample scenario will provide help in learning how Azure Batch works while showcasing Azure Batch Labs as an example SaaS solution that can be developed for your own customers:</span></span>

<span data-ttu-id="4c50a-170">[Azure Batch Masterclass][batch-labs-masterclass]</span><span class="sxs-lookup"><span data-stu-id="4c50a-170">[Azure Batch Masterclass][batch-labs-masterclass]</span></span>

### <a name="deploying-the-sample-scenario-using-an-azure-resource-manager-arm-template"></a><span data-ttu-id="4c50a-171">ARM(Azure Resource Manager) 템플릿을 사용하여 샘플 시나리오 배포</span><span class="sxs-lookup"><span data-stu-id="4c50a-171">Deploying the sample scenario using an Azure Resource Manager (ARM) template</span></span>

<span data-ttu-id="4c50a-172">템플릿에서 다음과 같이 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-172">The template will deploy:</span></span>
  - <span data-ttu-id="4c50a-173">새 Azure Batch 계정</span><span class="sxs-lookup"><span data-stu-id="4c50a-173">A new Azure Batch account</span></span>
  - <span data-ttu-id="4c50a-174">저장소 계정</span><span class="sxs-lookup"><span data-stu-id="4c50a-174">A storage account</span></span>
  - <span data-ttu-id="4c50a-175">배치 계정과 연결된 노드 풀</span><span class="sxs-lookup"><span data-stu-id="4c50a-175">A node pool associated with the batch account</span></span>
  - <span data-ttu-id="4c50a-176">Canonical Ubuntu 이미지를 사용하여 노드 풀에서 A2 v2 VM을 사용하도록 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-176">The node pool will be configured to use A2 v2 VMs with Canonical Ubuntu images</span></span>
  - <span data-ttu-id="4c50a-177">처음에는 노드 풀에 0개의 VM이 포함되며, VM을 추가하려면 수동으로 크기 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-177">The node pool will contain 0 VMs initially and will require scaling manually to add VMs</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fhpc%2Fbatchcreatewithpools.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="4c50a-178">[ARM 템플릿에 대한 자세한 정보][azure-arm-templates]</span><span class="sxs-lookup"><span data-stu-id="4c50a-178">[Learn more about ARM templates][azure-arm-templates]</span></span>

## <a name="pricing"></a><span data-ttu-id="4c50a-179">가격</span><span class="sxs-lookup"><span data-stu-id="4c50a-179">Pricing</span></span>

<span data-ttu-id="4c50a-180">Azure Batch를 사용하는 데 드는 비용은 풀에 사용되는 VM 크기와 이러한 VM이 할당되고 실행되는 기간에 따라 다르며, Azure Batch 계정을 만드는 데 관련된 비용은 없습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-180">The cost of using Azure Batch will depend on the VM sizes that are used for the pools and how long these are allocated and running, there is no cost associated with an Azure Batch account creation.</span></span> <span data-ttu-id="4c50a-181">추가 비용이 적용되므로 저장소 및 데이터 송신도 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-181">Storage and data egress should also be taken into account as these will apply additional costs.</span></span>

<span data-ttu-id="4c50a-182">서로 다른 수의 서버를 사용하여 8시간 내에 완료되는 작업에 청구될 수 있는 비용의 예는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-182">The following are examples of costs that could be incurred for a job that completes in 8 hours using a different number of servers:</span></span>


- <span data-ttu-id="4c50a-183">100개 고성능 CPU VM: [예상 비용][hpc-est-high]</span><span class="sxs-lookup"><span data-stu-id="4c50a-183">100 High Performance CPU VMs: [Cost Estimate][hpc-est-high]</span></span>

  <span data-ttu-id="4c50a-184">100 x H16m(16개 코어, 225GB RAM, 512GB Premium Storage), 2TB Blob Storage, 1TB 송신</span><span class="sxs-lookup"><span data-stu-id="4c50a-184">100 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

- <span data-ttu-id="4c50a-185">50개 고성능 CPU VM: [예상 비용][hpc-est-med]</span><span class="sxs-lookup"><span data-stu-id="4c50a-185">50 High Performance CPU VMs: [Cost Estimate][hpc-est-med]</span></span>

  <span data-ttu-id="4c50a-186">50 x H16m(16개 코어, 225GB RAM, 512GB Premium Storage), 2TB Blob Storage, 1TB 송신</span><span class="sxs-lookup"><span data-stu-id="4c50a-186">50 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

- <span data-ttu-id="4c50a-187">10개 고성능 CPU VM: [예상 비용][hpc-est-low]</span><span class="sxs-lookup"><span data-stu-id="4c50a-187">10 High Performance CPU VMs: [Cost Estimate][hpc-est-low]</span></span>
  
  <span data-ttu-id="4c50a-188">10 x H16m(16개 코어, 225GB RAM, 512GB Premium Storage), 2TB Blob Storage, 1TB 송신</span><span class="sxs-lookup"><span data-stu-id="4c50a-188">10 x H16m (16 cores, 225GB RAM, Premium Storage 512GB), 2 TB Blob Storage, 1 TB egress</span></span>

### <a name="low-priority-vm-pricing"></a><span data-ttu-id="4c50a-189">낮은 우선 순위 VM 가격</span><span class="sxs-lookup"><span data-stu-id="4c50a-189">Low Priority VM Pricing</span></span>

<span data-ttu-id="4c50a-190">Azure Batch는 노드 풀에서 낮은 우선 순위 VM\*도 사용할 수 있도록 지원하므로 잠재적으로 상당한 비용을 절감할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="4c50a-190">Azure Batch also supports the use of Low Priority VMs\* in the node pools, which can potentially provide a substantial cost saving.</span></span> <span data-ttu-id="4c50a-191">표준 VM과 낮은 우선 순위 VM 간의 가격 비교 및 우선 순위가 낮은 VM에 대한 자세한 내용은 [Batch 가격][batch-pricing]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="4c50a-191">For a price comparison between standard VMs and Low Priority VMs, and to find out more about Low Priority VMs, please see [Batch Pricing][batch-pricing].</span></span>

<span data-ttu-id="4c50a-192">\* 특정 응용 프로그램과 워크로드만 낮은 우선 순위 VM에서 실행하는 데 적합하다는 점에 유의하세요.</span><span class="sxs-lookup"><span data-stu-id="4c50a-192">\* Please note that only certain applications and workloads will be suitable to run on Low Priority VMs.</span></span>

## <a name="related-resources"></a><span data-ttu-id="4c50a-193">관련 리소스</span><span class="sxs-lookup"><span data-stu-id="4c50a-193">Related Resources</span></span>

<span data-ttu-id="4c50a-194">[Azure Batch 개요][batch-overview]</span><span class="sxs-lookup"><span data-stu-id="4c50a-194">[Azure Batch Overview][batch-overview]</span></span>

<span data-ttu-id="4c50a-195">[Azure Batch 설명서][batch-doc]</span><span class="sxs-lookup"><span data-stu-id="4c50a-195">[Azure Batch Documentation][batch-doc]</span></span>

<span data-ttu-id="4c50a-196">[Azure Batch에서 컨테이너 사용][batch-containers]</span><span class="sxs-lookup"><span data-stu-id="4c50a-196">[Using containers on Azure Batch][batch-containers]</span></span>

<!-- links -->
[architecture]: ./media/native-hpc-ref-arch.png
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmss]: /azure/virtual-machine-scale-sets/overview
[vnet]: /azure/virtual-network/virtual-networks-overview
[storage]: https://azure.microsoft.com/services/storage/
[batch]: https://azure.microsoft.com/services/batch/
[batch-arch]: https://azure.microsoft.com/solutions/architecture/big-compute-with-azure-batch/
[compute-hpc]: /azure/virtual-machines/windows/sizes-hpc
[compute-gpu]: /azure/virtual-machines/windows/sizes-gpu
[compute-compute]: /azure/virtual-machines/windows/sizes-compute
[compute-memory]: /azure/virtual-machines/windows/sizes-memory
[compute-general]: /azure/virtual-machines/windows/sizes-general
[compute-storage]: /azure/virtual-machines/windows/sizes-storage
[compute-acu]: /azure/virtual-machines/windows/acu
[compute=benchmark]: /azure/virtual-machines/windows/compute-benchmark-scores
[hpc-est-high]: https://azure.com/e/9ac25baf44ef49c3a6b156935ee9544c
[hpc-est-med]: https://azure.com/e/0286f1d6f6784310af4dcda5aec8c893
[hpc-est-low]: https://azure.com/e/e39afab4e71949f9bbabed99b428ba4a
[batch-labs-masterclass]: https://github.com/azurebigcompute/BigComputeLabs/tree/master/Azure%20Batch%20Masterclass%20Labs
[batch-scaling]: /azure/batch/batch-automatic-scaling
[hpc-alt-solutions]: /azure/virtual-machines/linux/high-performance-computing?toc=%2fazure%2fbatch%2ftoc.json
[batch-monitor]: /azure/batch/monitoring-overview
[batch-pricing]: https://azure.microsoft.com/en-gb/pricing/details/batch/
[batch-doc]: /azure/batch/
[batch-overview]: https://azure.microsoft.com/services/batch/
[batch-containers]: https://github.com/Azure/batch-shipyard
[azure-arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[batch-plugins]: /azure/batch/batch-rendering-service#options-for-submitting-a-render-job