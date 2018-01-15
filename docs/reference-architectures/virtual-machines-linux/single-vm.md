---
title: "Azure에서 Linux VM 실행"
description: "확장성, 복원력, 관리 효율성 및 보안에 주의하면서 Azure에서 Linux VM을 실행하는 방법입니다."
author: telmosampaio
ms.date: 12/12/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
ms.openlocfilehash: 7caef46e53b42011b5a12ef53384c0352b9b9a72
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/08/2018
---
# <a name="run-a-linux-vm-on-azure"></a><span data-ttu-id="e707c-103">Azure에서 Linux VM 실행</span><span class="sxs-lookup"><span data-stu-id="e707c-103">Run a Linux VM on Azure</span></span>

<span data-ttu-id="e707c-104">이 참조 아키텍처는 Azure에서 Linux VM(가상 머신)을 실행하는 데 대해 검증된 일련의 사례를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-104">This reference architecture shows a set of proven practices for running a Linux virtual machine (VM) on Azure.</span></span> <span data-ttu-id="e707c-105">여기에는 VM 프로비전과 함께 네트워킹 및 저장소 구성 요소에 관한 권장 사항이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-105">It includes recommendations for provisioning the VM along with networking and storage components.</span></span> <span data-ttu-id="e707c-106">이 아키텍처는 단일 VM 인스턴스를 실행하는 데 사용할 수 있으며 N 계층 응용 프로그램과 같은 복잡한 아키텍처의 기반이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-106">This architecture can be used to run a single VM instance, and is the basis for more complex architectures such as N-tier applications.</span></span> [<span data-ttu-id="e707c-107">**이 솔루션을 배포합니다.**</span><span class="sxs-lookup"><span data-stu-id="e707c-107">**Deploy this solution.**</span></span>](#deploy-the-solution)

<span data-ttu-id="e707c-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="e707c-108">![[0]][0]</span></span>

<span data-ttu-id="e707c-109">*이 아키텍처 다이어그램이 포함된 [Visio 파일][visio-download]을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="e707c-109">*Download a [Visio file][visio-download] that contains this architecture diagram.*</span></span>

## <a name="architecture"></a><span data-ttu-id="e707c-110">건축</span><span class="sxs-lookup"><span data-stu-id="e707c-110">Architecture</span></span>

<span data-ttu-id="e707c-111">Azure VM을 프로비전하려면 계산, 네트워킹 및 저장소 리소스와 같은 추가 구성 요소가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-111">Provisioning an Azure VM requires additional components, such as compute, networking, and storage resources.</span></span>

* <span data-ttu-id="e707c-112">**리소스 그룹.**</span><span class="sxs-lookup"><span data-stu-id="e707c-112">**Resource group.**</span></span> <span data-ttu-id="e707c-113">[리소스 그룹][resource-manager-overview]은 관련된 리소스를 보유하는 컨테이너입니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-113">A [resource group][resource-manager-overview] is a container that holds related resources.</span></span> <span data-ttu-id="e707c-114">일반적으로 해당 수명과 리소스 관리자를 기준으로 솔루션의 리소스를 그룹화해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-114">In general, you should group resources in a solution based on their lifetime and who will manage the resources.</span></span> <span data-ttu-id="e707c-115">단일 VM 워크로드의 경우 모든 리소스에 대해 단일 리소스 그룹을 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-115">For a single VM workload, you may want to create a single resource group for all resources.</span></span>
* <span data-ttu-id="e707c-116">**VM**.</span><span class="sxs-lookup"><span data-stu-id="e707c-116">**VM**.</span></span> <span data-ttu-id="e707c-117">게시된 이미지 목록 또는 Azure Blob Storage에 업로드된 사용자 지정 관리되는 이미지나 VHD(가상 하드 디스크) 파일에서 VM을 프로비전할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-117">You can provision a VM from a list of published images, or from a custom managed image or virtual hard disk (VHD) file uploaded to Azure Blob storage.</span></span> <span data-ttu-id="e707c-118">Azure에서는 CentOS, Debian, Red Hat Enterprise, Ubuntu 및 FreeBSD를 포함하여 인기 있는 다양한 Linux 배포판을 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-118">Azure supports running various popular Linux distributions, including CentOS, Debian, Red Hat Enterprise, Ubuntu, and FreeBSD.</span></span> <span data-ttu-id="e707c-119">자세한 내용은 [Azure 및 Linux][azure-linux]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-119">For more information, see [Azure and Linux][azure-linux].</span></span>
* <span data-ttu-id="e707c-120">**OS 디스크.**</span><span class="sxs-lookup"><span data-stu-id="e707c-120">**OS disk.**</span></span> <span data-ttu-id="e707c-121">OS 디스크는 [Azure Storage][azure-storage]에 저장된 VHD이므로 호스트 컴퓨터가 중단되어도 계속 유지됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-121">The OS disk is a VHD stored in [Azure Storage][azure-storage], so it persists even when the host machine is down.</span></span> <span data-ttu-id="e707c-122">Linux VM의 경우 OS 디스크는 `/dev/sda1`입니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-122">For Linux VMs, the OS disk is `/dev/sda1`.</span></span>
* <span data-ttu-id="e707c-123">**임시 디스크.**</span><span class="sxs-lookup"><span data-stu-id="e707c-123">**Temporary disk.**</span></span> <span data-ttu-id="e707c-124">VM은 임시 디스크를 사용하여 만들어집니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-124">The VM is created with a temporary disk.</span></span> <span data-ttu-id="e707c-125">이 디스크는 호스트 컴퓨터의 실제 드라이브에 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-125">This disk is stored on a physical drive on the host machine.</span></span> <span data-ttu-id="e707c-126">Azure Storage에는 저장되지 **않으며** 다시 부팅되는 동안에 그리고 다른 VM의 수명 주기 이벤트 동안에 삭제될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-126">It is **not** saved in Azure Storage and may be deleted during reboots and other VM lifecycle events.</span></span> <span data-ttu-id="e707c-127">페이지 또는 스왑 파일과 같은 임시 데이터에 대해서만 이 디스크를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-127">Use this disk only for temporary data, such as page or swap files.</span></span> <span data-ttu-id="e707c-128">Linux VM의 경우 임시 디스크는 `/dev/sdb1`이며 `/mnt/resource` 또는 `/mnt`에 탑재됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-128">For Linux VMs, the temporary disk is `/dev/sdb1` and is mounted at `/mnt/resource` or `/mnt`.</span></span>
* <span data-ttu-id="e707c-129">**데이터 디스크.**</span><span class="sxs-lookup"><span data-stu-id="e707c-129">**Data disks.**</span></span> <span data-ttu-id="e707c-130">[데이터 디스크][data-disk]는 응용 프로그램 데이터에 사용되는 영구 VHD입니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-130">A [data disk][data-disk] is a persistent VHD used for application data.</span></span> <span data-ttu-id="e707c-131">데이터 디스크는 OS 디스크와 같은 Azure Storage에 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-131">Data disks are stored in Azure Storage, like the OS disk.</span></span>
* <span data-ttu-id="e707c-132">**가상 네트워크(VNet) 및 서브넷.**</span><span class="sxs-lookup"><span data-stu-id="e707c-132">**Virtual network (VNet) and subnet.**</span></span> <span data-ttu-id="e707c-133">모든 Azure VM은 VNet에 배포되어 여러 서브넷으로 분할될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-133">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span>
* <span data-ttu-id="e707c-134">**공용 IP 주소.**</span><span class="sxs-lookup"><span data-stu-id="e707c-134">**Public IP address.**</span></span> <span data-ttu-id="e707c-135">공용 IP 주소는 예를 들어 SSH를 통해 VM과 통신하는 데 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-135">A public IP address is needed to communicate with the VM &mdash; for example, via SSH.</span></span>
* <span data-ttu-id="e707c-136">**Azure DNS**.</span><span class="sxs-lookup"><span data-stu-id="e707c-136">**Azure DNS**.</span></span> <span data-ttu-id="e707c-137">[Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-137">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="e707c-138">Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-138">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>
* <span data-ttu-id="e707c-139">**NIC(네트워크 인터페이스)**.</span><span class="sxs-lookup"><span data-stu-id="e707c-139">**Network interface (NIC)**.</span></span> <span data-ttu-id="e707c-140">할당된 NIC를 통해 VM을 가상 네트워크와 통신하도록 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-140">An assigned NIC enables the VM to communicate with the virtual network.</span></span>
* <span data-ttu-id="e707c-141">**NSG(네트워크 보안 그룹)**.</span><span class="sxs-lookup"><span data-stu-id="e707c-141">**Network security group (NSG)**.</span></span> <span data-ttu-id="e707c-142">[네트워크 보안 그룹][nsg]은 네트워크 리소스에 대한 네트워크 트래픽을 허용하거나 거부하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-142">[Network security groups][nsg] are used to allow or deny network traffic to a network resource.</span></span> <span data-ttu-id="e707c-143">NSG를 개별 NIC 또는 서브넷에 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-143">You can associate an NSG with an individual NIC or with a subnet.</span></span> <span data-ttu-id="e707c-144">서브넷에 연결하는 경우 NSG 규칙이 해당 서브넷의 모든 VM에 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-144">If you associate it with a subnet, the NSG rules apply to all VMs in that subnet.</span></span>
* <span data-ttu-id="e707c-145">**진단**</span><span class="sxs-lookup"><span data-stu-id="e707c-145">**Diagnostics.**</span></span> <span data-ttu-id="e707c-146">VM을 관리 및 문제 해결하는 데 진단 로깅이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-146">Diagnostic logging is crucial for managing and troubleshooting the VM.</span></span>

## <a name="recommendations"></a><span data-ttu-id="e707c-147">권장 사항</span><span class="sxs-lookup"><span data-stu-id="e707c-147">Recommendations</span></span>

<span data-ttu-id="e707c-148">이 아키텍처는 Azure에서 Linux VM을 실행하기 위한 기본 권장 사항을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-148">This architecture shows the baseline recommendations for running a Linux VM in Azure.</span></span> <span data-ttu-id="e707c-149">그러나 단일 실패 지점을 생성하므로 중요한 작업에 단일 VM을 사용하지 않는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-149">However, we don't recommend using a single VM for mission critical workloads because it creates a single point of failure.</span></span> <span data-ttu-id="e707c-150">더 높은 가용성을 위해 [가용성 집합][availability-set]에 여러 VM을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-150">For higher availability, deploy multiple VMs in an [availability set][availability-set].</span></span> <span data-ttu-id="e707c-151">자세한 내용은 [Azure에서 여러 VM 실행][multi-vm]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-151">For more information, see [Running multiple VMs on Azure][multi-vm].</span></span> 

### <a name="vm-recommendations"></a><span data-ttu-id="e707c-152">VM 권장 사항</span><span class="sxs-lookup"><span data-stu-id="e707c-152">VM recommendations</span></span>

<span data-ttu-id="e707c-153">Azure는 다양한 가상 머신 크기를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-153">Azure offers many different virtual machine sizes.</span></span> <span data-ttu-id="e707c-154">[Premium Storage][premium-storage]는 높은 성능과 짧은 대기 시간으로 인해 권장되며 [특정 VM 크기에서 지원][premium-storage-supported]됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-154">[Premium Storage][premium-storage] is recommended due to its high performance and low latency, and is [supported by specific VM sizes][premium-storage-supported].</span></span> <span data-ttu-id="e707c-155">고성능 컴퓨팅과 같은 특수한 워크로드가 발생하지 않는다면 이러한 크기 중 하나를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-155">Select one of these sizes unless you have a specialized workload such as high-performance computing.</span></span> <span data-ttu-id="e707c-156">자세한 내용은 [가상 머신 크기][virtual-machine-sizes]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-156">For more information, see [virtual machine sizes][virtual-machine-sizes].</span></span>

<span data-ttu-id="e707c-157">기존 워크로드를 Azure로 이동하는 경우 온-프레미스 서버와 가장 근접하게 일치하는 VM 크기부터 사용하기 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-157">If you are moving an existing workload to Azure, start with the VM size that's the closest match to your on-premises servers.</span></span> <span data-ttu-id="e707c-158">그런 다음 CPU, 메모리, 디스크 IOPS(초당 입력/출력 작업 수)에 따라 실제 워크로드의 성능을 측정하고 필요에 따라 크기를 조정합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-158">Then measure the performance of your actual workload with respect to CPU, memory, and disk input/output operations per second (IOPS), and adjust the size as needed.</span></span> <span data-ttu-id="e707c-159">VM에 여러 개의 NIC가 필요한 경우 각 [VM 크기][vm-size-tables]에 대한 최대 NIC 수가 정의되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-159">If you require multiple NICs for your VM, be aware that a maximum number of NICs is defined for each [VM size][vm-size-tables].</span></span>

<span data-ttu-id="e707c-160">Azure 리소스를 프로비전할 때는 지역을 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-160">When you provision Azure resources, you must specify a region.</span></span> <span data-ttu-id="e707c-161">일반적으로 내부 사용자 또는 고객에게 가장 가까운 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-161">Generally, choose a region closest to your internal users or customers.</span></span> <span data-ttu-id="e707c-162">그러나 일부 지역에서는 일부 VM 크기를 사용하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-162">However, not all VM sizes are available in all regions.</span></span> <span data-ttu-id="e707c-163">자세한 내용은 [지역별 서비스][services-by-region]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-163">For more information, see [Services by region][services-by-region].</span></span> <span data-ttu-id="e707c-164">지정된 지역에서 사용할 수 있는 VM 크기 목록을 보려면 다음 Azure CLI(명령줄 인터페이스) 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-164">For a list of the VM sizes available in a specific region, run the following command from the Azure command-line interface (CLI):</span></span>

```
az vm list-sizes --location <location>
```

<span data-ttu-id="e707c-165">게시된 VM 이미지를 선택하는 방법에 대한 자세한 내용은 [Linux VM 이미지 찾기][select-vm-image]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-165">For information about choosing a published VM image, see [Find Linux VM images][select-vm-image].</span></span>

<span data-ttu-id="e707c-166">기본 상태 메트릭, 진단 인프라 로그 및 [부팅 진단][boot-diagnostics]을 모니터링 및 진단을 사용하도록 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-166">Enable monitoring and diagnostics, including basic health metrics, diagnostics infrastructure logs, and [boot diagnostics][boot-diagnostics].</span></span> <span data-ttu-id="e707c-167">부팅 진단은 VM이 부팅할 수 없는 상태로 전환되는 경우 부팅 오류를 진단하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-167">Boot diagnostics can help you diagnose boot failure if your VM gets into a non-bootable state.</span></span> <span data-ttu-id="e707c-168">자세한 내용은 [모니터링 및 진단 사용][enable-monitoring]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-168">For more information, see [Enable monitoring and diagnostics][enable-monitoring].</span></span>  

### <a name="disk-and-storage-recommendations"></a><span data-ttu-id="e707c-169">디스크 및 저장소 권장 사항</span><span class="sxs-lookup"><span data-stu-id="e707c-169">Disk and storage recommendations</span></span>

<span data-ttu-id="e707c-170">최상의 디스크 I/O 성능을 위해서는 SSD(반도체 드라이브)에 데이터를 저장하는 [Premium Storage][premium-storage]가 권장됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-170">For best disk I/O performance, we recommend [Premium Storage][premium-storage], which stores data on solid-state drives (SSDs).</span></span> <span data-ttu-id="e707c-171">비용은 프로비전된 디스크 용량을 기준으로 산정됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-171">Cost is based on the capacity of the provisioned disk.</span></span> <span data-ttu-id="e707c-172">IOPS 및 처리량(즉, 데이터 전송 속도)도 디스크 크기에 따라 달라지므로 디스크를 프로비전할 때 세 가지 요소(용량, IOPS, 처리량)를 모두 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-172">IOPS and throughput (that is, data transfer rate) also depend on disk size, so when you provision a disk, consider all three factors (capacity, IOPS, and throughput).</span></span> 

<span data-ttu-id="e707c-173">또한 [관리 디스크](/azure/storage/storage-managed-disks-overview)를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-173">We also recommend the use of [managed disks](/azure/storage/storage-managed-disks-overview).</span></span> <span data-ttu-id="e707c-174">관리 디스크에는 저장소 계정이 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-174">Managed disks do not require a storage account.</span></span> <span data-ttu-id="e707c-175">디스크의 크기와 유형을 지정하기만 하면 고가용성 리소스로 배포됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-175">You simply specify the size and type of disk and it is deployed as a highly available resource.</span></span>

<span data-ttu-id="e707c-176">관리되지 않는 디스크를 사용 중인 경우 저장소 계정에 대한 [IOPS 제한][vm-disk-limits]에 도달하지 않도록 하기 위해 VHD(가상 하드 디스크)를 보유하는 각 VM에 대한 별도 Azure Storage 계정을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-176">If you are using unmanaged disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs), in order to avoid hitting the [(IOPS) limits][vm-disk-limits] for storage accounts.</span></span>

<span data-ttu-id="e707c-177">하나 이상의 데이터 디스크를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-177">Add one or more data disks.</span></span> <span data-ttu-id="e707c-178">VHD를 만들 때 형식은 지정되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-178">When you create a VHD, it is unformatted.</span></span> <span data-ttu-id="e707c-179">디스크를 포맷하려면 VM에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-179">Log into the VM to format the disk.</span></span> <span data-ttu-id="e707c-180">관리 디스크를 사용하고 있지 않고 데이터 디스크가 많은 경우 저장소 계정의 총 I/O 제한을 알아두어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-180">If you are not using managed disks and have a large number of data disks, be aware of the total I/O limits of the storage account.</span></span> <span data-ttu-id="e707c-181">자세한 내용은 [가상 머신 디스크 제한][vm-disk-limits]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-181">For more information, see [virtual machine disk limits][vm-disk-limits].</span></span>

<span data-ttu-id="e707c-182">Linux 셸에서 데이터 디스크는 `/dev/sdc`, `/dev/sdd` 등으로 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-182">In the Linux shell, data disks are displayed as `/dev/sdc`, `/dev/sdd`, and so on.</span></span> <span data-ttu-id="e707c-183">`lsblk` 를 실행하여 디스크를 포함하는 블록 장치를 나열할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-183">You can run `lsblk` to list the block devices, including the disks.</span></span> <span data-ttu-id="e707c-184">데이터 디스크를 사용하려면 파티션 및 파일 시스템을 만들고 디스크를 탑재합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-184">To use a data disk, create a partition and file system, and mount the disk.</span></span> <span data-ttu-id="e707c-185">예: </span><span class="sxs-lookup"><span data-stu-id="e707c-185">For example:</span></span>

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

<span data-ttu-id="e707c-186">데이터 디스크를 추가하면 디스크에 LUN(논리 단위 번호) ID가 할당됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-186">When you add a data disk, a logical unit number (LUN) ID is assigned to the disk.</span></span> <span data-ttu-id="e707c-187">예를 들어, 디스크를 교체하고 동일한 LUN ID를 유지하거나 특정 LUN ID를 검색하는 응용 프로그램이 있는 경우 필요에 따라 LUN ID &mdash;을(를) 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-187">Optionally, you can specify the LUN ID &mdash; for example, if you're replacing a disk and want to retain the same LUN ID, or you have an application that looks for a specific LUN ID.</span></span> <span data-ttu-id="e707c-188">그렇지만 LUN ID는 디스크마다 고유해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-188">However, remember that LUN IDs must be unique for each disk.</span></span>

<span data-ttu-id="e707c-189">프리미엄 저장소 계정에서 VM의 디스크가 SSD이기 때문에 I/O 스케줄러를 변경하여 SSD의 성능을 최적화하려고 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-189">You may want to change the I/O scheduler to optimize for performance on SSDs because the disks for VMs with premium storage accounts are SSDs.</span></span> <span data-ttu-id="e707c-190">SSD에 NOOP 스케줄러를 사용하는 것이 일반적으로 권장되지만 [iostat] 와 같은 도구를 사용하여 워크로드에 대한 디스크 I/O 성능을 모니터링할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-190">A common recommendation is to use the NOOP scheduler for SSDs, but you should use a tool such as [iostat] to monitor disk I/O performance for your workload.</span></span>

<span data-ttu-id="e707c-191">성능을 최대화하기 위해 진단 로그를 저장할 별도의 저장소 계정을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-191">To maximize performance, create a separate storage account to hold diagnostic logs.</span></span> <span data-ttu-id="e707c-192">표준 LRS(로컬 중복 저장소) 계정은 진단 로그에 충분합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-192">A standard locally redundant storage (LRS) account is sufficient for diagnostic logs.</span></span>

### <a name="network-recommendations"></a><span data-ttu-id="e707c-193">네트워크 권장 사항</span><span class="sxs-lookup"><span data-stu-id="e707c-193">Network recommendations</span></span>

<span data-ttu-id="e707c-194">이 공용 IP 주소는 동적 또는 정적일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-194">The public IP address can be dynamic or static.</span></span> <span data-ttu-id="e707c-195">기본값은 동적입니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-195">The default is dynamic.</span></span>

* <span data-ttu-id="e707c-196">변경되지 않는 고정 IP 주소가 필요한 경우 [정적 IP 주소][static-ip]를 예약합니다(예를 들어 DNS에 A 레코드를 만들어야 하는 경우 또는 안전한 목록에 추가될 IP 주소가 필요한 경우).</span><span class="sxs-lookup"><span data-stu-id="e707c-196">Reserve a [static IP address][static-ip] if you need a fixed IP address that won't change &mdash; for example, if you need to create an A record in DNS, or need the IP address to be added to a safe list.</span></span>
* <span data-ttu-id="e707c-197">IP 주소의 FQDN(정규화된 도메인 이름)을 만들 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-197">You can also create a fully qualified domain name (FQDN) for the IP address.</span></span> <span data-ttu-id="e707c-198">그런 후 FQDN을 가리키는 DNS에 [CNAME 레코드][cname-record]를 등록할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-198">You can then register a [CNAME record][cname-record] in DNS that points to the FQDN.</span></span> <span data-ttu-id="e707c-199">자세한 내용은 [Azure Portal에서 정규화된 도메인 이름 만들기][fqdn]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-199">For more information, see [Create a fully qualified domain name in the Azure portal][fqdn].</span></span> <span data-ttu-id="e707c-200">[Azure DNSS][azure-dns] 또는 다른 DNS 서비스를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-200">You can use [Azure DNS][azure-dns] or another DNS service.</span></span>

<span data-ttu-id="e707c-201">모든 NSG에는 모든 인바운드 인터넷 트래픽을 차단하는 규칙을 포함하여 [기본 규칙][nsg-default-rules] 집합이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-201">All NSGs contain a set of [default rules][nsg-default-rules], including a rule that blocks all inbound Internet traffic.</span></span> <span data-ttu-id="e707c-202">기본 규칙은 삭제할 수 없으나 다른 규칙으로 재정의할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-202">The default rules cannot be deleted, but other rules can override them.</span></span> <span data-ttu-id="e707c-203">인터넷 트래픽을 사용하도록 설정하려면 특정 포트(예: HTTP용 포트 80)로의 인바운드 트래픽을 허용하는 규칙을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-203">To enable Internet traffic, create rules that allow inbound traffic to specific ports &mdash; for example, port 80 for HTTP.</span></span>

<span data-ttu-id="e707c-204">SSH를 사용하도록 설정하려면 TCP 포트 22에 인바운드 트래픽을 허용하는 NSG 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-204">To enable SSH, add an NSG rule that allows inbound traffic to TCP port 22.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="e707c-205">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="e707c-205">Scalability considerations</span></span>

<span data-ttu-id="e707c-206">[VM 크기를 변경][vm-resize]하여 VM을 규모 확장 또는 축소할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-206">You can scale a VM up or down by [changing the VM size][vm-resize].</span></span> <span data-ttu-id="e707c-207">규모를 확장하려면 부하 분산 장치 뒤에 둘 이상의 VM을 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-207">To scale out horizontally, put two or more VMs behind a load balancer.</span></span> <span data-ttu-id="e707c-208">자세한 내용은 [Azure에서 확장성 및 가용성을 위해 여러 VM 실행][multi-vm]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-208">For more information, see [Running multiple VMs on Azure for scalability and availability][multi-vm].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="e707c-209">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="e707c-209">Availability considerations</span></span>

<span data-ttu-id="e707c-210">더 높은 가용성을 위해 가용성 집합에 여러 VM을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-210">For higher availability, deploy multiple VMs in an availability set.</span></span> <span data-ttu-id="e707c-211">그러면 [SLA(서비스 수준 계약)][vm-sla]도 높아집니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-211">This also provides a higher [service level agreement (SLA)][vm-sla].</span></span>

<span data-ttu-id="e707c-212">사용자의 VM은 [계획된 유지 관리][planned-maintenance] 또는 [계획되지 않은 유지 관리][manage-vm-availability]의 영향을 받을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-212">Your VM may be affected by [planned maintenance][planned-maintenance] or [unplanned maintenance][manage-vm-availability].</span></span> <span data-ttu-id="e707c-213">[VM 다시 부팅 로그][reboot-logs]를 사용하여 VM 재부팅이 계획된 유지 관리로 발생했는지 여부를 결정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-213">You can use [VM reboot logs][reboot-logs] to determine whether a VM reboot was caused by planned maintenance.</span></span>

<span data-ttu-id="e707c-214">VHD는 [Azure Storage][azure-storage]에 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-214">VHDs are stored in [Azure storage][azure-storage].</span></span> <span data-ttu-id="e707c-215">Azure Storage는 내구성 및 가용성을 위해 복제됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-215">Azure storage is replicated for durability and availability.</span></span>

<span data-ttu-id="e707c-216">정상 작업 중 실수로 인한 데이터 손실(예: 사용자 오류 때문에 발생)을 막기 위해 [Blob 스냅숏][blob-snapshot] 또는 다른 도구를 사용하여 지정 시간 백업도 구현해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-216">To protect against accidental data loss during normal operations (for example, because of user error), you should also implement point-in-time backups, using [blob snapshots][blob-snapshot] or another tool.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="e707c-217">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="e707c-217">Manageability considerations</span></span>

<span data-ttu-id="e707c-218">**리소스 그룹**</span><span class="sxs-lookup"><span data-stu-id="e707c-218">**Resource groups.**</span></span> <span data-ttu-id="e707c-219">동일한 수명 주기를 공유하는 긴밀하게 연결된 리소스를 동일한 [리소스 그룹][resource-manager-overview]에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-219">Put closely associated resources that share the same lifecycle into the same [resource group][resource-manager-overview].</span></span> <span data-ttu-id="e707c-220">리소스 그룹을 사용하여 리소스를 그룹 단위로 배포 및 모니터링하고, 리소스 그룹별로 청구 비용을 추적할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-220">Resource groups allow you to deploy and monitor resources as a group and track billing costs by resource group.</span></span> <span data-ttu-id="e707c-221">리소스를 하나의 집합으로 삭제할 수도 있습니다. 이러한 기능은 테스트 배포에서 매우 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-221">You can also delete resources as a set, which is very useful for test deployments.</span></span> <span data-ttu-id="e707c-222">의미 있는 리소스 이름을 할당하면 간단하게 특정 리소스를 찾고 해당 역할을 이해할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-222">Assign meaningful resource names to simplify locating a specific resource and understanding its role.</span></span> <span data-ttu-id="e707c-223">자세한 내용은 [Azure 리소스에 대해 권장되는 명명 규칙][naming-conventions]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-223">For more information, see [Recommended naming conventions for Azure resources][naming-conventions].</span></span>

<span data-ttu-id="e707c-224">**SSH**.</span><span class="sxs-lookup"><span data-stu-id="e707c-224">**SSH**.</span></span> <span data-ttu-id="e707c-225">Linux VM을 만들기 전에 2048비트 RSA 공개-개인 키 쌍을 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-225">Before you create a Linux VM, generate a 2048-bit RSA public-private key pair.</span></span> <span data-ttu-id="e707c-226">VM을 만들 때 공개 키 파일을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-226">Use the public key file when you create the VM.</span></span> <span data-ttu-id="e707c-227">자세한 내용은 [Azure에서 Linux 및 Mac과 함께 SSH를 사용하는 방법][ssh-linux]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-227">For more information, see [How to Use SSH with Linux and Mac on Azure][ssh-linux].</span></span>

<span data-ttu-id="e707c-228">**VM 중지.**</span><span class="sxs-lookup"><span data-stu-id="e707c-228">**Stopping a VM.**</span></span> <span data-ttu-id="e707c-229">Azure에서는 "중지됨"과 "할당 취소됨" 상태를 구분합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-229">Azure makes a distinction between "stopped" and "deallocated" states.</span></span> <span data-ttu-id="e707c-230">VM 상태가 중지되면 요금이 청구되지만 VM 할당이 취소되면 청구되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-230">You are charged when the VM status is stopped, but not when the VM is deallocated.</span></span>

<span data-ttu-id="e707c-231">Azure Portal에서 **중지** 버튼은 VM 할당을 취소합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-231">In the Azure portal, the **Stop** button deallocates the VM.</span></span> <span data-ttu-id="e707c-232">로그인한 상태에서 OS를 통해 종료하면 VM은 중지되지만 할당 취소되지 **않으므로** 비용이 계속 청구됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-232">If you shut down through the OS while logged in, the VM is stopped but **not** deallocated, so you will still be charged.</span></span>

<span data-ttu-id="e707c-233">**VM 삭제.**</span><span class="sxs-lookup"><span data-stu-id="e707c-233">**Deleting a VM.**</span></span> <span data-ttu-id="e707c-234">VM을 삭제하는 경우 VHD는 삭제되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-234">If you delete a VM, the VHDs are not deleted.</span></span> <span data-ttu-id="e707c-235">즉, 데이터 손실 없이 안전하게 VM을 삭제할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-235">That means you can safely delete the VM without losing data.</span></span> <span data-ttu-id="e707c-236">그러나 저장소에 대한 비용은 계속 청구됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-236">However, you will still be charged for storage.</span></span> <span data-ttu-id="e707c-237">VHD를 삭제하려면 [Blob 저장소][blob-storage]에서 파일을 삭제합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-237">To delete the VHD, delete the file from [Blob storage][blob-storage].</span></span>

<span data-ttu-id="e707c-238">실수로 삭제하지 않도록 하려면 [리소스 잠금][resource-lock]을 사용하여 전체 리소스 그룹을 잠그거나 VM과 같은 개별 리소스를 잠급니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-238">To prevent accidental deletion, use a [resource lock][resource-lock] to lock the entire resource group or lock individual resources, such as a VM.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="e707c-239">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="e707c-239">Security considerations</span></span>

<span data-ttu-id="e707c-240">[Azure Security Center][security-center]를 사용하여 Azure 리소스의 보안 상태를 중앙에서 살펴볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-240">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="e707c-241">Security Center는 잠재적인 보안 문제를 모니터링하고 배포의 보안 상태에 대한 종합적인 그림을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-241">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="e707c-242">보안 센터는 각 Azure 구독을 기준으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-242">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="e707c-243">[Azure Security Center 빠른 시작 가이드][security-center-get-started]에 설명된 것처럼 보안 데이터 수집을 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-243">Enable security data collection as described in the [Azure Security Center quick start guide][security-center-get-started].</span></span> <span data-ttu-id="e707c-244">데이터 수집이 사용되도록 설정되면 보안 센터는 해당 구독에서 만든 모든 VM을 자동으로 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-244">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="e707c-245">**패치 관리.**</span><span class="sxs-lookup"><span data-stu-id="e707c-245">**Patch management.**</span></span> <span data-ttu-id="e707c-246">이 기능이 설정된 경우 Security Center는 보안 및 중요 업데이트 누락 여부를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-246">If enabled, Security Center checks whether any security and critical updates are missing.</span></span> 

<span data-ttu-id="e707c-247">**맬웨어 방지.**</span><span class="sxs-lookup"><span data-stu-id="e707c-247">**Antimalware.**</span></span> <span data-ttu-id="e707c-248">이 기능이 설정되면 보안 센터는 맬웨어 방지 소프트웨어 설치 여부를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-248">If enabled, Security Center checks whether antimalware software is installed.</span></span> <span data-ttu-id="e707c-249">또한 보안 센터를 사용하여 Azure 포털 내에서 맬웨어 방지 소프트웨어를 설치할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-249">You can also use Security Center to install antimalware software from inside the Azure portal.</span></span>

<span data-ttu-id="e707c-250">**작업.**</span><span class="sxs-lookup"><span data-stu-id="e707c-250">**Operations.**</span></span> <span data-ttu-id="e707c-251">[RBAC(역할 기반 액세스 제어)][rbac]를 사용하여 배포하는 Azure 리소스에 대한 액세스를 제어합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-251">Use [role-based access control (RBAC)][rbac] to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="e707c-252">RBAC를 통해 DevOps 팀의 구성원에게 권한 역할을 할당할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-252">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="e707c-253">예를 들어 읽기 권한자 역할은 Azure 리소스를 볼 수 있지만 만들거나 관리하거나 삭제할 수는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-253">For example, the Reader role can view Azure resources but not create, manage, or delete them.</span></span> <span data-ttu-id="e707c-254">일부 역할은 특정 Azure 리소스 유형에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-254">Some roles are specific to particular Azure resource types.</span></span> <span data-ttu-id="e707c-255">예를 들어 Virtual Machine Contributor 역할은 VM을 다시 시작하거나 할당을 취소하고, 관리자 암호를 재설정하고, 새 VM을 만드는 등의 작업을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-255">For example, the Virtual Machine Contributor role can restart or deallocate a VM, reset the administrator password, create a new VM, and so on.</span></span> <span data-ttu-id="e707c-256">이 아키텍처에 유용할 수 있는 기타 [기본 제공 RBAC 역할][rbac-roles]에는 [DevTest Labs 사용자][rbac-devtest] 및 [네트워크 참가자][rbac-network]가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-256">Other [built-in RBAC roles][rbac-roles] that may be useful for this architecture include [DevTest Labs User][rbac-devtest] and [Network Contributor][rbac-network].</span></span> <span data-ttu-id="e707c-257">한 명의 사용자가 여러 역할에 할당될 수 있으며 좀 더 세분화된 권한의 사용자 지정 역할을 만들 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-257">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained permissions.</span></span>

> [!NOTE]
> <span data-ttu-id="e707c-258">RBAC는 VM에 로그온한 사용자가 수행할 수 있는 작업을 제한하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-258">RBAC does not limit the actions that a user logged into a VM can perform.</span></span> <span data-ttu-id="e707c-259">이러한 사용 권한은 게스트 OS의 계정 유형에 따라 결정됩니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-259">Those permissions are determined by the account type on the guest OS.</span></span>   

<span data-ttu-id="e707c-260">[감사 로그][audit-logs]를 사용하여 프로비전 동작 및 기타 VM 이벤트를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-260">Use [audit logs][audit-logs] to see provisioning actions and other VM events.</span></span>

<span data-ttu-id="e707c-261">**데이터 암호화.**</span><span class="sxs-lookup"><span data-stu-id="e707c-261">**Data encryption.**</span></span> <span data-ttu-id="e707c-262">OS 및 데이터 디스크를 암호화해야 하는 경우 [Azure Disk Encryption][disk-encryption]을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-262">Consider [Azure Disk Encryption][disk-encryption] if you need to encrypt the OS and data disks.</span></span> 

## <a name="deploy-the-solution"></a><span data-ttu-id="e707c-263">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="e707c-263">Deploy the solution</span></span>

<span data-ttu-id="e707c-264">이 아키텍처에 대한 배포는 [GitHub][github-folder]에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-264">A deployment for this architecture is available on [GitHub][github-folder].</span></span> <span data-ttu-id="e707c-265">다음을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-265">It deploys the following:</span></span>

  * <span data-ttu-id="e707c-266">VM을 호스트하는 데 사용되는 **웹**이라는 이름의 단일 서브넷을 가진 가상 네트워크</span><span class="sxs-lookup"><span data-stu-id="e707c-266">A virtual network with a single subnet named **web** used to host the VM.</span></span>
  * <span data-ttu-id="e707c-267">SSH 및 HTTP 트래픽이 VM으로 이동할 수 있도록 허용하는 두 개의 들어오는 규칙이 있는 NSG</span><span class="sxs-lookup"><span data-stu-id="e707c-267">An NSG with two incoming rules to allow SSH and HTTP traffic to the VM.</span></span>
  * <span data-ttu-id="e707c-268">최신 버전의 Ubuntu16.04.3 LTS를 실행하는 VM</span><span class="sxs-lookup"><span data-stu-id="e707c-268">A VM running the latest version of Ubuntu 16.04.3 LTS.</span></span>
  * <span data-ttu-id="e707c-269">두 개의 데이터 디스크를 포맷하고 Apache HTTP Server를 Ubuntu VM에 배포하는 샘플 사용자 지정 스크립트 확장</span><span class="sxs-lookup"><span data-stu-id="e707c-269">A sample custom script extension that formats the two data disks and deploys Apache HTTP Server to the Ubuntu VM.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="e707c-270">필수 구성 요소</span><span class="sxs-lookup"><span data-stu-id="e707c-270">Prerequisites</span></span>

<span data-ttu-id="e707c-271">사용자의 구독에 참조 아키텍처를 배포하려면 먼저 다음 단계를 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-271">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="e707c-272">[AzureCAT 참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-272">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="e707c-273">Azure CLI 2.0이 컴퓨터에 설치되어 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-273">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="e707c-274">CLI 설치 지침은 [Install Azure CLI 2.0][azure-cli-2](Azure CLI 2.0 설치)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-274">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="e707c-275">[Azure 빌딩 블록][azbb] npm 패키지를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-275">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="e707c-276">명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 다음 명령 중 하나를 사용하여 Azure 계정에 로그인한 다음 프롬프트에 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-276">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="e707c-277">azbb를 사용하여 솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="e707c-277">Deploy the solution using azbb</span></span>

<span data-ttu-id="e707c-278">샘플 단일 VM 워크로드를 배포하려면 다음 단계를 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-278">To deploy the sample single VM workload, follow these steps:</span></span>

1. <span data-ttu-id="e707c-279">위의 필수 조건 단계에서 다운로드한 리포지토리의 `virtual-machines\single-vm\parameters\linux` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-279">Navigate to the `virtual-machines\single-vm\parameters\linux` folder for the repository you downloaded in the prerequisites step above.</span></span>

2. <span data-ttu-id="e707c-280">`single-vm-v2.json` 파일을 열고 아래 나와 있는 대로 큰따옴표 사이에 사용자 이름과 SSH 공개 키를 입력한 다음 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-280">Open the `single-vm-v2.json` file and enter a username and SSH public key between the quotes, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "",
  "sshPublicKey": "",
  ```

3. <span data-ttu-id="e707c-281">아래 표시된 대로 `azbb`를 실행하여 샘플 VM을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-281">Run `azbb` to deploy the sample VM as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

<span data-ttu-id="e707c-282">이 샘플 참조 아키텍처를 배포하는 방법에 대한 자세한 내용은 [GitHub 리포지토리][git]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="e707c-282">For more information on deploying this sample reference architecture, visit our [GitHub repository][git].</span></span>

## <a name="next-steps"></a><span data-ttu-id="e707c-283">다음 단계</span><span class="sxs-lookup"><span data-stu-id="e707c-283">Next steps</span></span>

- <span data-ttu-id="e707c-284">[Azure 빌딩 블록][azbbv2]에 대해 알아봅니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-284">Learn about our [Azure building Blocks][azbbv2].</span></span>
- <span data-ttu-id="e707c-285">Azure에서 [여러 VM][multi-vm]을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="e707c-285">Deploy [multiple VMs][multi-vm] in Azure.</span></span>

<!-- links -->
[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azbbv2]: https://github.com/mspnp/template-building-blocks
[azure-cli-2]: /cli/azure/install-azure-cli?view=azure-cli-latest
[azure-linux]: /azure/virtual-machines/virtual-machines-linux-azure-overview
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-linux-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[azure-dns]: /azure/dns/dns-overview
[fqdn]: /azure/virtual-machines/virtual-machines-linux-portal-create-fqdn
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[iostat]: https://en.wikipedia.org/wiki/Iostat
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[multi-vm]: multi-vm.md
[naming-conventions]: /azure/architecture/best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-linux-planned-maintenance
[premium-storage]: /azure/virtual-machines/linux/premium-storage
[premium-storage-supported]: /azure/virtual-machines/linux/premium-storage#supported-vms
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: /azure/security-center/security-center-intro
[security-center-get-started]: /azure/security-center/security-center-get-started
[select-vm-image]: /azure/virtual-machines/virtual-machines-linux-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssh-linux]: /azure/virtual-machines/virtual-machines-linux-mac-create-ssh-keys
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-linux-sizes
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Azure의 단일 Linux VM 아키텍처"
