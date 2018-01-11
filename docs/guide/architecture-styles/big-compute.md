---
title: "큰 계산 아키텍처 스타일"
description: "Azure에서 큰 계산 아키텍처의 이점, 과제 및 모범 사례를 설명합니다."
author: MikeWasson
ms.openlocfilehash: b16be4133143d7d73062eeb280b44779c390f387
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="big-compute-architecture-style"></a><span data-ttu-id="9ccc5-103">큰 계산 아키텍처 스타일</span><span class="sxs-lookup"><span data-stu-id="9ccc5-103">Big compute architecture style</span></span>

<span data-ttu-id="9ccc5-104">*큰 계산*이라는 용어는 수백 또는 수천 개의 번호 매기기 등 코어 수를 많이 요구하는 대규모 워크로드를 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-104">The term *big compute* describes large-scale workloads that require a large number of cores, often numbering in the hundreds or thousands.</span></span> <span data-ttu-id="9ccc5-105">시나리오에는 이미지 렌더링, 유체 역학, 재무 위험 모델링, 석유 탐색, 약 디자인 및 스트레스 분석 엔지니어링이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-105">Scenarios include image rendering, fluid dynamics, financial risk modeling, oil exploration, drug design, and engineering stress analysis, among others.</span></span>

![](./images/big-compute-logical.png)

<span data-ttu-id="9ccc5-106">큰 계산 응용 프로그램의 몇 가지 일반적인 특성은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-106">Here are some typical characteristics of big compute applications:</span></span>

- <span data-ttu-id="9ccc5-107">작업은 동시에 여러 코어에서 실행될 수 있는 개별 작업으로 분할될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-107">The work can be split into discrete tasks, which can be run across many cores simultaneously.</span></span>
- <span data-ttu-id="9ccc5-108">각 작업은 한정됩니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-108">Each task is finite.</span></span> <span data-ttu-id="9ccc5-109">일부 입력을 사용하고, 일부 처리를 수행하고, 출력을 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-109">It takes some input, does some processing, and produces output.</span></span> <span data-ttu-id="9ccc5-110">전체 응용 프로그램은 제한된 양의 시간(몇 분, 몇 일) 동안 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-110">The entire application runs for a finite amount of time (minutes to days).</span></span> <span data-ttu-id="9ccc5-111">일반적인 패턴에서는 갑자기 많은 수의 코어를 프로비전하고 응용 프로그램이 완료되면 0으로 스핀 다운합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-111">A common pattern is to provision a large number of cores in a burst, and then spin down to zero once the application completes.</span></span> 
- <span data-ttu-id="9ccc5-112">응용 프로그램을 일주일 내내 사용하지 않아도 됩니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-112">The application does not need to stay up 24/7.</span></span> <span data-ttu-id="9ccc5-113">그러나 시스템은 노드 오류 또는 응용 프로그램 충돌을 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-113">However, the system must handle node failures or application crashes.</span></span>
- <span data-ttu-id="9ccc5-114">일부 응용 프로그램의 경우 작업이 독립적이며 병렬로 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-114">For some applications, tasks are independent and can run in parallel.</span></span> <span data-ttu-id="9ccc5-115">다른 경우에 작업은 밀접하게 연결됩니다. 즉, 중간 결과와 상호 작용하거나 교환해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-115">In other cases, tasks are tightly coupled, meaning they must interact or exchange intermediate results.</span></span> <span data-ttu-id="9ccc5-116">이 경우에 InfiniBand 및 RDMA(원격 직접 메모리 액세스)와 같은 고속 네트워킹 기술을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-116">In that case, consider using high-speed networking technologies such as InfiniBand and remote direct memory access (RDMA).</span></span> 
- <span data-ttu-id="9ccc5-117">워크로드에 따라 계산 집약적인 VM 크기(H16r, H16mr 및 A9)를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-117">Depending on your workload, you might use compute-intensive VM sizes (H16r, H16mr, and A9).</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="9ccc5-118">이 아키텍처를 사용하는 경우</span><span class="sxs-lookup"><span data-stu-id="9ccc5-118">When to use this architecture</span></span>

- <span data-ttu-id="9ccc5-119">시뮬레이션 및 숫자 고속 처리와 같은 계산 집약적인 작업</span><span class="sxs-lookup"><span data-stu-id="9ccc5-119">Computationally intensive operations such as simulation and number crunching.</span></span>
- <span data-ttu-id="9ccc5-120">계산 집약적이고 여러 컴퓨터(10~1000대)에서 CPU에 분할되어야 하는 시뮬레이션</span><span class="sxs-lookup"><span data-stu-id="9ccc5-120">Simulations that are computationally intensive and must be split across CPUs in multiple computers (10-1000s).</span></span>
- <span data-ttu-id="9ccc5-121">한 대의 컴퓨터에 메모리가 너무 많이 필요하고 여러 컴퓨터 간에 분할되야 하는 시뮬레이션</span><span class="sxs-lookup"><span data-stu-id="9ccc5-121">Simulations that require too much memory for one computer, and must be split across multiple computers.</span></span>
- <span data-ttu-id="9ccc5-122">단일 컴퓨터에서 완료되는 시간이 너무 오래 걸리는 장기 실행 계산</span><span class="sxs-lookup"><span data-stu-id="9ccc5-122">Long-running computations that would take too long to complete on a single computer.</span></span>
- <span data-ttu-id="9ccc5-123">Monte Carlo 시뮬레이션과 같이 100번 또는 1000번 실행해야 하는 더 작은 계산</span><span class="sxs-lookup"><span data-stu-id="9ccc5-123">Smaller computations that must be run 100s or 1000s of times, such as Monte Carlo simulations.</span></span>

## <a name="benefits"></a><span data-ttu-id="9ccc5-124">이점</span><span class="sxs-lookup"><span data-stu-id="9ccc5-124">Benefits</span></span>

- <span data-ttu-id="9ccc5-125">"[병렬][embarrassingly-parallel]" 처리가 적합한 고성능</span><span class="sxs-lookup"><span data-stu-id="9ccc5-125">High performance with "[embarrassingly parallel][embarrassingly-parallel]" processing.</span></span>
- <span data-ttu-id="9ccc5-126">큰 문제를 더 빠르게 해결하도록 수백 또는 수천 개의 컴퓨터 코어를 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-126">Can harness hundreds or thousands of computer cores to solve large problems faster.</span></span>
- <span data-ttu-id="9ccc5-127">빠른 전용 InfiniBand 네트워크를 사용하여 특수한 고성능 하드웨어에 액세스합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-127">Access to specialized high-performance hardware, with dedicated high-speed InfiniBand networks.</span></span>
- <span data-ttu-id="9ccc5-128">작업을 수행하고 종료하는 데 필요한 만큼 VM을 프로비전할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-128">You can provision VMs as needed to do work, and then tear them down.</span></span> 

## <a name="challenges"></a><span data-ttu-id="9ccc5-129">과제</span><span class="sxs-lookup"><span data-stu-id="9ccc5-129">Challenges</span></span>

- <span data-ttu-id="9ccc5-130">VM 인프라 관리</span><span class="sxs-lookup"><span data-stu-id="9ccc5-130">Managing the VM infrastructure.</span></span>
- <span data-ttu-id="9ccc5-131">숫자 고속 처리의 볼륨 관리</span><span class="sxs-lookup"><span data-stu-id="9ccc5-131">Managing the volume of number crunching.</span></span> 
- <span data-ttu-id="9ccc5-132">적절하게 수천 개의 코어 프로비전</span><span class="sxs-lookup"><span data-stu-id="9ccc5-132">Provisioning thousands of cores in a timely manner.</span></span>
- <span data-ttu-id="9ccc5-133">밀접하게 연결된 작업의 경우 더 많은 코어를 추가하면 반환을 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-133">For tightly coupled tasks, adding more cores can have diminishing returns.</span></span> <span data-ttu-id="9ccc5-134">최적의 코어 수를 찾기 위해 실험이 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-134">You may need to experiment to find the optimum number of cores.</span></span>

## <a name="big-compute-using-azure-batch"></a><span data-ttu-id="9ccc5-135">Azure Batch를 사용하는 큰 계산</span><span class="sxs-lookup"><span data-stu-id="9ccc5-135">Big compute using Azure Batch</span></span>

<span data-ttu-id="9ccc5-136">[Azure Batch][batch]는 대규모 HPC(고성능 컴퓨팅) 응용 프로그램을 실행하기 위한 관리 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-136">[Azure Batch][batch] is a managed service for running large-scale high-performance computing (HPC) applications.</span></span>

<span data-ttu-id="9ccc5-137">Azure Batch를 사용하여 VM 풀을 구성하고, 응용 프로그램 및 데이터 파일을 업로드합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-137">Using Azure Batch, you configure a VM pool, and upload the applications and data files.</span></span> <span data-ttu-id="9ccc5-138">Batch 서비스는 VM을 프로비전하고, VM에 작업을 할당하고, 작업을 실행하고, 진행 상황을 모니터링합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-138">Then the Batch service provisions the VMs, assign tasks to the VMs, runs the tasks, and monitors the progress.</span></span> <span data-ttu-id="9ccc5-139">Batch는 워크로드에 따라 VM을 자동으로 스케일 아웃할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-139">Batch can automatically scale out the VMs in response to the workload.</span></span> <span data-ttu-id="9ccc5-140">또한 Batch는 작업 예약 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-140">Batch also provides job scheduling.</span></span>

![](./images/big-compute-batch.png) 

## <a name="big-compute-running-on-virtual-machines"></a><span data-ttu-id="9ccc5-141">Virtual Machines에서 실행되는 큰 계산</span><span class="sxs-lookup"><span data-stu-id="9ccc5-141">Big compute running on Virtual Machines</span></span>

<span data-ttu-id="9ccc5-142">[Microsoft HPC Pack][hpc-pack]을 사용하여 VM의 클러스터를 관리하고 HPC 작업을 예약하고 모니터링할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-142">You can use [Microsoft HPC Pack][hpc-pack] to administer a cluster of VMs, and schedule and monitor HPC jobs.</span></span> <span data-ttu-id="9ccc5-143">이 접근 방식으로 VM 및 네트워크 인프라를 프로비전하고 관리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-143">With this approach, you must provision and manage the VMs and network infrastructure.</span></span> <span data-ttu-id="9ccc5-144">기존 HPC 워크로드가 있고 그 중 일부 또는 모두를 Azure로 이동하려면 이 방법을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-144">Consider this approach if you have existing HPC workloads and want to move some or all it to Azure.</span></span> <span data-ttu-id="9ccc5-145">전체 HPC 클러스터를 Azure로 이동하거나 HPC 클러스터 온-프레미스를 유지하면서 버스트 용량에 Azure를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-145">You can move the entire HPC cluster to Azure, or keep your HPC cluster on-premises but use Azure for burst capacity.</span></span> <span data-ttu-id="9ccc5-146">자세한 내용은 [대규모 컴퓨팅 워크로드를 위한 Batch 및 HPC 솔루션][batch-hpc-solutions]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-146">For more information, see [Batch and HPC solutions for large-scale computing workloads][batch-hpc-solutions].</span></span>

### <a name="hpc-pack-deployed-to-azure"></a><span data-ttu-id="9ccc5-147">Azure에 배포된 HPC Pack</span><span class="sxs-lookup"><span data-stu-id="9ccc5-147">HPC Pack deployed to Azure</span></span>

<span data-ttu-id="9ccc5-148">이 시나리오에서 HPC 클러스터는 Azure 내에서 완전히 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-148">In this scenario, the HPC cluster is created entirely within Azure.</span></span>

![](./images/big-compute-iaas.png) 
 
<span data-ttu-id="9ccc5-149">헤드 노드는 클러스터에 대한 관리 및 작업 예약 서비스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-149">The head node provides management and job scheduling services to the cluster.</span></span> <span data-ttu-id="9ccc5-150">밀접하게 연결된 작업의 경우 VM 간에 매우 높은 대역폭으로 대기 시간이 짧은 통신을 제공하는 RDMA 네트워크를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-150">For tightly coupled tasks, use an RDMA network that provides very high bandwidth, low latency communication between VMs.</span></span> <span data-ttu-id="9ccc5-151">자세한 내용은 [Azure에서 HPC Pack 2016 클러스터 배포][deploy-hpc-azure]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-151">For more information see [Deploy an HPC Pack 2016 cluster in Azure][deploy-hpc-azure].</span></span>

### <a name="burst-an-hpc-cluster-to-azure"></a><span data-ttu-id="9ccc5-152">HPC 클러스터를 Azure로 버스트</span><span class="sxs-lookup"><span data-stu-id="9ccc5-152">Burst an HPC cluster to Azure</span></span>

<span data-ttu-id="9ccc5-153">이 시나리오에서 조직은 HPC Pack 온-프레미스를 실행하고 버스트 용량에 대해 Azure VM을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-153">In this scenario, an organization is running HPC Pack on-premises, and uses Azure VMs for burst capacity.</span></span> <span data-ttu-id="9ccc5-154">클러스터 헤드 노드는 온-프레미스입니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-154">The cluster head node is on-premises.</span></span> <span data-ttu-id="9ccc5-155">ExpressRoute 또는 VPN Gateway는 온-프레미스 네트워크를 Azure VNet에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="9ccc5-155">ExpressRoute or VPN Gateway connects the on-premises network to the Azure VNet.</span></span>

![](./images/big-compute-hybrid.png) 


[batch]: /azure/batch/
[batch-hpc-solutions]: /azure/batch/batch-hpc-solutions
[deploy-hpc-azure]: /azure/virtual-machines/windows/hpcpack-2016-cluster
[embarrassingly-parallel]: https://en.wikipedia.org/wiki/Embarrassingly_parallel
[hpc-pack]: https://technet.microsoft.com/library/cc514029

 
