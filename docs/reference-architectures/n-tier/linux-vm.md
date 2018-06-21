---
title: Azure에서 Linux VM 실행
description: 확장성, 복원력, 관리 효율성 및 보안에 주의하면서 Azure에서 Linux VM을 실행하는 방법입니다.
author: telmosampaio
ms.date: 04/03/2018
ms.openlocfilehash: f29b7225c2e0edbb1569c9e3a55d112d12041af8
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/06/2018
ms.locfileid: "33673585"
---
# <a name="run-a-linux-vm-on-azure"></a><span data-ttu-id="f8684-103">Azure에서 Linux VM 실행</span><span class="sxs-lookup"><span data-stu-id="f8684-103">Run a Linux VM on Azure</span></span>

<span data-ttu-id="f8684-104">이 문서에서는 Azure에서 Linux VM(가상 머신)을 실행하기 위해 검증된 일단의 사례에 대해 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-104">This article describes a set of proven practices for running a Linux virtual machine (VM) on Azure.</span></span> <span data-ttu-id="f8684-105">여기에는 VM 프로비전과 함께 네트워킹 및 저장소 구성 요소에 관한 권장 사항이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-105">It includes recommendations for provisioning the VM along with networking and storage components.</span></span> [<span data-ttu-id="f8684-106">**이 솔루션을 배포합니다.**</span><span class="sxs-lookup"><span data-stu-id="f8684-106">**Deploy this solution.**</span></span>](#deploy-the-solution)

<span data-ttu-id="f8684-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="f8684-107">![[0]][0]</span></span>

## <a name="components"></a><span data-ttu-id="f8684-108">구성 요소</span><span class="sxs-lookup"><span data-stu-id="f8684-108">Components</span></span>

<span data-ttu-id="f8684-109">Azure VM을 프로비전하려면 VM 외에도 네트워킹 및 저장소 리소스와 같은 추가 구성 요소가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-109">Provisioning an Azure VM requires some additional components besides the VM itself, including networking and storage resources.</span></span>

* <span data-ttu-id="f8684-110">**리소스 그룹.**</span><span class="sxs-lookup"><span data-stu-id="f8684-110">**Resource group.**</span></span> <span data-ttu-id="f8684-111">[리소스 그룹][resource-manager-overview]은 관련 Azure 리소스를 보유하는 논리 컨테이너입니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-111">A [resource group][resource-manager-overview] is a logical container that holds related Azure resources.</span></span> <span data-ttu-id="f8684-112">일반적으로 리소스의 수명 및 관리하는 주체에 따라 리소스를 그룹화합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-112">In general, group resources based on their lifetime and who will manage them.</span></span> 

* <span data-ttu-id="f8684-113">**VM**.</span><span class="sxs-lookup"><span data-stu-id="f8684-113">**VM**.</span></span> <span data-ttu-id="f8684-114">게시된 이미지 목록 또는 Azure Blob Storage에 업로드된 사용자 지정 관리되는 이미지나 VHD(가상 하드 디스크) 파일에서 VM을 프로비전할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-114">You can provision a VM from a list of published images, or from a custom managed image or virtual hard disk (VHD) file uploaded to Azure Blob storage.</span></span> <span data-ttu-id="f8684-115">Azure에서는 CentOS, Debian, Red Hat Enterprise, Ubuntu 및 FreeBSD를 포함하여 인기 있는 다양한 Linux 배포판을 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-115">Azure supports running various popular Linux distributions, including CentOS, Debian, Red Hat Enterprise, Ubuntu, and FreeBSD.</span></span> <span data-ttu-id="f8684-116">자세한 내용은 [Azure 및 Linux][azure-linux]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-116">For more information, see [Azure and Linux][azure-linux].</span></span>

* <span data-ttu-id="f8684-117">**관리되는 디스크**.</span><span class="sxs-lookup"><span data-stu-id="f8684-117">**Managed Disks**.</span></span> <span data-ttu-id="f8684-118">[Azure Managed Disks][managed-disks]는 저장소를 처리하여 디스크 관리를 단순화합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-118">[Azure Managed Disks][managed-disks] simplify disk management by handling the storage for you.</span></span> <span data-ttu-id="f8684-119">OS 디스크는 [Azure Storage][azure-storage]에 저장된 VHD이므로 호스트 컴퓨터가 중단되어도 계속 유지됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-119">The OS disk is a VHD stored in [Azure Storage][azure-storage], so it persists even when the host machine is down.</span></span> <span data-ttu-id="f8684-120">Linux VM의 경우 OS 디스크는 `/dev/sda1`입니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-120">For Linux VMs, the OS disk is `/dev/sda1`.</span></span> <span data-ttu-id="f8684-121">또한 응용 프로그램 데이터에 사용되는 영구 VHD인 [데이터 디스크][data-disk]를 하나 이상 만드는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-121">We also recommend creating one or more [data disks][data-disk], which are persistent VHDs used for application data.</span></span> 

* <span data-ttu-id="f8684-122">**임시 디스크.**</span><span class="sxs-lookup"><span data-stu-id="f8684-122">**Temporary disk.**</span></span> <span data-ttu-id="f8684-123">VM은 임시 디스크를 사용하여 만들어집니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-123">The VM is created with a temporary disk.</span></span> <span data-ttu-id="f8684-124">이 디스크는 호스트 컴퓨터의 실제 드라이브에 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-124">This disk is stored on a physical drive on the host machine.</span></span> <span data-ttu-id="f8684-125">Azure Storage에는 저장되지 *않으며* 다시 부팅되는 동안에 그리고 다른 VM의 수명 주기 이벤트 동안에 삭제될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-125">It is *not* saved in Azure Storage and may be deleted during reboots and other VM lifecycle events.</span></span> <span data-ttu-id="f8684-126">페이지 또는 스왑 파일과 같은 임시 데이터에 대해서만 이 디스크를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-126">Use this disk only for temporary data, such as page or swap files.</span></span> <span data-ttu-id="f8684-127">Linux VM의 경우 임시 디스크는 `/dev/sdb1`이며 `/mnt/resource` 또는 `/mnt`에 탑재됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-127">For Linux VMs, the temporary disk is `/dev/sdb1` and is mounted at `/mnt/resource` or `/mnt`.</span></span>

* <span data-ttu-id="f8684-128">**VNet(가상 네트워크).**</span><span class="sxs-lookup"><span data-stu-id="f8684-128">**Virtual network (VNet).**</span></span> <span data-ttu-id="f8684-129">모든 Azure VM은 VNet에 배포되어 여러 서브넷으로 분할될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-129">Every Azure VM is deployed into a VNet that can be segmented into multiple subnets.</span></span>

* <span data-ttu-id="f8684-130">**NIC(네트워크 인터페이스)**.</span><span class="sxs-lookup"><span data-stu-id="f8684-130">**Network interface (NIC)**.</span></span> <span data-ttu-id="f8684-131">NIC를 통해 VM을 가상 네트워크와 통신하도록 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-131">The NIC enables the VM to communicate with the virtual network.</span></span>

* <span data-ttu-id="f8684-132">**공용 IP 주소.**</span><span class="sxs-lookup"><span data-stu-id="f8684-132">**Public IP address.**</span></span> <span data-ttu-id="f8684-133">공용 IP 주소는 예를 들어 SSH를 통해 VM과 통신하는 데 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-133">A public IP address is needed to communicate with the VM &mdash; for example, via SSH.</span></span>

* <span data-ttu-id="f8684-134">**Azure DNS**.</span><span class="sxs-lookup"><span data-stu-id="f8684-134">**Azure DNS**.</span></span> <span data-ttu-id="f8684-135">[Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-135">[Azure DNS][azure-dns] is a hosting service for DNS domains, providing name resolution using Microsoft Azure infrastructure.</span></span> <span data-ttu-id="f8684-136">Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-136">By hosting your domains in Azure, you can manage your DNS records using the same credentials, APIs, tools, and billing as your other Azure services.</span></span>

* <span data-ttu-id="f8684-137">**NSG(네트워크 보안 그룹)**.</span><span class="sxs-lookup"><span data-stu-id="f8684-137">**Network security group (NSG)**.</span></span> <span data-ttu-id="f8684-138">[네트워크 보안 그룹][nsg]은 VM에 대한 네트워크 트래픽을 허용하거나 거부하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-138">[Network security groups][nsg] are used to allow or deny network traffic to VMs.</span></span> <span data-ttu-id="f8684-139">NSG는 서브넷 또는 개별 VM 인스턴스로 연결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-139">NSGs can be associated either with subnets or with individual VM instances.</span></span>

* <span data-ttu-id="f8684-140">**진단**</span><span class="sxs-lookup"><span data-stu-id="f8684-140">**Diagnostics.**</span></span> <span data-ttu-id="f8684-141">VM을 관리 및 문제 해결하는 데 진단 로깅이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-141">Diagnostic logging is crucial for managing and troubleshooting the VM.</span></span>

## <a name="vm-recommendations"></a><span data-ttu-id="f8684-142">VM 권장 사항</span><span class="sxs-lookup"><span data-stu-id="f8684-142">VM recommendations</span></span>

<span data-ttu-id="f8684-143">Azure는 다양한 가상 머신 크기를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-143">Azure offers many different virtual machine sizes.</span></span> <span data-ttu-id="f8684-144">자세한 내용은 [Azure의 가상 머신 크기][virtual-machine-sizes]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-144">For more information, see [Sizes for virtual machines in Azure][virtual-machine-sizes].</span></span> <span data-ttu-id="f8684-145">기존 워크로드를 Azure로 이동하는 경우 온-프레미스 서버와 가장 근접하게 일치하는 VM 크기부터 사용하기 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-145">If you are moving an existing workload to Azure, start with the VM size that's the closest match to your on-premises servers.</span></span> <span data-ttu-id="f8684-146">그런 다음 CPU, 메모리, 디스크 IOPS(초당 입력/출력 작업 수)에 따라 실제 워크로드의 성능을 측정하고 필요에 따라 크기를 조정합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-146">Then measure the performance of your actual workload with respect to CPU, memory, and disk input/output operations per second (IOPS), and adjust the size as needed.</span></span> <span data-ttu-id="f8684-147">VM에 여러 개의 NIC가 필요한 경우 각 [VM 크기][vm-size-tables]에 대한 최대 NIC 수가 정의되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-147">If you require multiple NICs for your VM, be aware that a maximum number of NICs is defined for each [VM size][vm-size-tables].</span></span>

<span data-ttu-id="f8684-148">일반적으로 내부 사용자 또는 고객에게 가장 가까운 Azure 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-148">Generally, choose an Azure region that is closest to your internal users or customers.</span></span> <span data-ttu-id="f8684-149">그러나 일부 지역에서는 일부 VM 크기를 사용하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-149">However, not all VM sizes are available in all regions.</span></span> <span data-ttu-id="f8684-150">자세한 내용은 [지역별 서비스][services-by-region]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-150">For more information, see [Services by region][services-by-region].</span></span> <span data-ttu-id="f8684-151">지정된 지역에서 사용할 수 있는 VM 크기 목록을 보려면 다음 Azure CLI(명령줄 인터페이스) 명령을 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-151">For a list of the VM sizes available in a specific region, run the following command from the Azure command-line interface (CLI):</span></span>

```
az vm list-sizes --location <location>
```

<span data-ttu-id="f8684-152">게시된 VM 이미지를 선택하는 방법에 대한 자세한 내용은 [Linux VM 이미지 찾기][select-vm-image]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-152">For information about choosing a published VM image, see [Find Linux VM images][select-vm-image].</span></span>

<span data-ttu-id="f8684-153">기본 상태 메트릭, 진단 인프라 로그 및 [부팅 진단][boot-diagnostics]을 모니터링 및 진단을 사용하도록 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-153">Enable monitoring and diagnostics, including basic health metrics, diagnostics infrastructure logs, and [boot diagnostics][boot-diagnostics].</span></span> <span data-ttu-id="f8684-154">부팅 진단은 VM이 부팅할 수 없는 상태로 전환되는 경우 부팅 오류를 진단하는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-154">Boot diagnostics can help you diagnose boot failure if your VM gets into a non-bootable state.</span></span> <span data-ttu-id="f8684-155">자세한 내용은 [모니터링 및 진단 사용][enable-monitoring]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-155">For more information, see [Enable monitoring and diagnostics][enable-monitoring].</span></span>  

## <a name="disk-and-storage-recommendations"></a><span data-ttu-id="f8684-156">디스크 및 저장소 권장 사항</span><span class="sxs-lookup"><span data-stu-id="f8684-156">Disk and storage recommendations</span></span>

<span data-ttu-id="f8684-157">최상의 디스크 I/O 성능을 위해서는 SSD(반도체 드라이브)에 데이터를 저장하는 [Premium Storage][premium-storage]가 권장됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-157">For best disk I/O performance, we recommend [Premium Storage][premium-storage], which stores data on solid-state drives (SSDs).</span></span> <span data-ttu-id="f8684-158">비용은 프로비전된 디스크 용량을 기준으로 산정됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-158">Cost is based on the capacity of the provisioned disk.</span></span> <span data-ttu-id="f8684-159">IOPS 및 처리량(즉, 데이터 전송 속도)도 디스크 크기에 따라 달라지므로 디스크를 프로비전할 때 세 가지 요소(용량, IOPS, 처리량)를 모두 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-159">IOPS and throughput (that is, data transfer rate) also depend on disk size, so when you provision a disk, consider all three factors (capacity, IOPS, and throughput).</span></span> 

<span data-ttu-id="f8684-160">또한 [관리되는 디스크][managed-disks]를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-160">We also recommend using [Managed Disks][managed-disks].</span></span> <span data-ttu-id="f8684-161">관리 디스크에는 저장소 계정이 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-161">Managed disks do not require a storage account.</span></span> <span data-ttu-id="f8684-162">디스크의 크기와 유형을 지정하기만 하면 고가용성 리소스로 배포됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-162">You simply specify the size and type of disk and it is deployed as a highly available resource.</span></span>

<span data-ttu-id="f8684-163">하나 이상의 데이터 디스크를 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-163">Add one or more data disks.</span></span> <span data-ttu-id="f8684-164">VHD를 만들 때 형식은 지정되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-164">When you create a VHD, it is unformatted.</span></span> <span data-ttu-id="f8684-165">디스크를 포맷하려면 VM에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-165">Log into the VM to format the disk.</span></span> <span data-ttu-id="f8684-166">Linux 셸에서 데이터 디스크는 `/dev/sdc`, `/dev/sdd` 등으로 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-166">In the Linux shell, data disks are displayed as `/dev/sdc`, `/dev/sdd`, and so on.</span></span> <span data-ttu-id="f8684-167">`lsblk` 를 실행하여 디스크를 포함하는 블록 장치를 나열할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-167">You can run `lsblk` to list the block devices, including the disks.</span></span> <span data-ttu-id="f8684-168">데이터 디스크를 사용하려면 파티션 및 파일 시스템을 만들고 디스크를 탑재합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-168">To use a data disk, create a partition and file system, and mount the disk.</span></span> <span data-ttu-id="f8684-169">예: </span><span class="sxs-lookup"><span data-stu-id="f8684-169">For example:</span></span>

```bat
# Create a partition.
sudo fdisk /dev/sdc     # Enter 'n' to partition, 'w' to write the change.

# Create a file system.
sudo mkfs -t ext3 /dev/sdc1

# Mount the drive.
sudo mkdir /data1
sudo mount /dev/sdc1 /data1
```

<span data-ttu-id="f8684-170">데이터 디스크를 추가하면 디스크에 LUN(논리 단위 번호) ID가 할당됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-170">When you add a data disk, a logical unit number (LUN) ID is assigned to the disk.</span></span> <span data-ttu-id="f8684-171">예를 들어, 디스크를 교체하고 동일한 LUN ID를 유지하거나 특정 LUN ID를 검색하는 응용 프로그램이 있는 경우 필요에 따라 LUN ID &mdash;을(를) 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-171">Optionally, you can specify the LUN ID &mdash; for example, if you're replacing a disk and want to retain the same LUN ID, or you have an application that looks for a specific LUN ID.</span></span> <span data-ttu-id="f8684-172">그렇지만 LUN ID는 디스크마다 고유해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-172">However, remember that LUN IDs must be unique for each disk.</span></span>

<span data-ttu-id="f8684-173">프리미엄 저장소 계정에서 VM의 디스크가 SSD이기 때문에 I/O 스케줄러를 변경하여 SSD의 성능을 최적화하려고 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-173">You may want to change the I/O scheduler to optimize for performance on SSDs because the disks for VMs with premium storage accounts are SSDs.</span></span> <span data-ttu-id="f8684-174">SSD에 NOOP 스케줄러를 사용하는 것이 일반적으로 권장되지만 [iostat] 와 같은 도구를 사용하여 워크로드에 대한 디스크 I/O 성능을 모니터링할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-174">A common recommendation is to use the NOOP scheduler for SSDs, but you should use a tool such as [iostat] to monitor disk I/O performance for your workload.</span></span>

<span data-ttu-id="f8684-175">진단 로그를 저장할 저장소 계정을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-175">Create a storage account to hold diagnostic logs.</span></span> <span data-ttu-id="f8684-176">표준 LRS(로컬 중복 저장소) 계정은 진단 로그에 충분합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-176">A standard locally redundant storage (LRS) account is sufficient for diagnostic logs.</span></span>

> [!NOTE]
> <span data-ttu-id="f8684-177">관리되는 디스크를 사용하지 않는 경우 저장소 계정에 대한 [(IOPS) 제한][vm-disk-limits]에 도달하지 않도록 하기 위해 VHD(가상 하드 디스크)를 보유하는 각 VM에 대한 별도 Azure Storage 계정을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-177">If you aren't using Managed Disks, create separate Azure storage accounts for each VM to hold the virtual hard disks (VHDs), in order to avoid hitting the [(IOPS) limits][vm-disk-limits] for storage accounts.</span></span> <span data-ttu-id="f8684-178">저장소 계정의 총 I/O 제한을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-178">Be aware of the total I/O limits of the storage account.</span></span> <span data-ttu-id="f8684-179">자세한 내용은 [가상 머신 디스크 제한][vm-disk-limits]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-179">For more information, see [virtual machine disk limits][vm-disk-limits].</span></span>

## <a name="network-recommendations"></a><span data-ttu-id="f8684-180">네트워크 권장 사항</span><span class="sxs-lookup"><span data-stu-id="f8684-180">Network recommendations</span></span>

<span data-ttu-id="f8684-181">이 공용 IP 주소는 동적 또는 정적일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-181">The public IP address can be dynamic or static.</span></span> <span data-ttu-id="f8684-182">기본값은 동적입니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-182">The default is dynamic.</span></span>

* <span data-ttu-id="f8684-183">변경되지 않는 고정 IP 주소가 필요한 경우 [정적 IP 주소][static-ip]를 예약합니다(예를 들어 DNS에 A 레코드를 만들어야 하는 경우 또는 안전한 목록에 추가될 IP 주소가 필요한 경우).</span><span class="sxs-lookup"><span data-stu-id="f8684-183">Reserve a [static IP address][static-ip] if you need a fixed IP address that won't change &mdash; for example, if you need to create an A record in DNS, or need the IP address to be added to a safe list.</span></span>
* <span data-ttu-id="f8684-184">IP 주소의 FQDN(정규화된 도메인 이름)을 만들 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-184">You can also create a fully qualified domain name (FQDN) for the IP address.</span></span> <span data-ttu-id="f8684-185">그런 후 FQDN을 가리키는 DNS에 [CNAME 레코드][cname-record]를 등록할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-185">You can then register a [CNAME record][cname-record] in DNS that points to the FQDN.</span></span> <span data-ttu-id="f8684-186">자세한 내용은 [Azure Portal에서 정규화된 도메인 이름 만들기][fqdn]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-186">For more information, see [Create a fully qualified domain name in the Azure portal][fqdn].</span></span> <span data-ttu-id="f8684-187">[Azure DNSS][azure-dns] 또는 다른 DNS 서비스를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-187">You can use [Azure DNS][azure-dns] or another DNS service.</span></span>

<span data-ttu-id="f8684-188">모든 NSG에는 모든 인바운드 인터넷 트래픽을 차단하는 규칙을 포함하여 [기본 규칙][nsg-default-rules] 집합이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-188">All NSGs contain a set of [default rules][nsg-default-rules], including a rule that blocks all inbound Internet traffic.</span></span> <span data-ttu-id="f8684-189">기본 규칙은 삭제할 수 없으나 다른 규칙으로 재정의할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-189">The default rules cannot be deleted, but other rules can override them.</span></span> <span data-ttu-id="f8684-190">인터넷 트래픽을 사용하도록 설정하려면 특정 포트(예: HTTP용 포트 80)로의 인바운드 트래픽을 허용하는 규칙을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-190">To enable Internet traffic, create rules that allow inbound traffic to specific ports &mdash; for example, port 80 for HTTP.</span></span>

<span data-ttu-id="f8684-191">SSH를 사용하도록 설정하려면 TCP 포트 22에 인바운드 트래픽을 허용하는 NSG 규칙을 추가합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-191">To enable SSH, add an NSG rule that allows inbound traffic to TCP port 22.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="f8684-192">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f8684-192">Scalability considerations</span></span>

<span data-ttu-id="f8684-193">[VM 크기를 변경][vm-resize]하여 VM을 규모 확장 또는 축소할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-193">You can scale a VM up or down by [changing the VM size][vm-resize].</span></span> <span data-ttu-id="f8684-194">규모를 확장하려면 부하 분산 장치 뒤에 둘 이상의 VM을 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-194">To scale out horizontally, put two or more VMs behind a load balancer.</span></span> <span data-ttu-id="f8684-195">자세한 내용은 [N 계층 참조 아키텍처](./n-tier-cassandra.md)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-195">For more information, see the [N-tier reference architecture](./n-tier-cassandra.md).</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="f8684-196">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f8684-196">Availability considerations</span></span>

<span data-ttu-id="f8684-197">더 높은 가용성을 위해 가용성 집합에 여러 VM을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-197">For higher availability, deploy multiple VMs in an availability set.</span></span> <span data-ttu-id="f8684-198">그러면 [SLA(서비스 수준 계약)][vm-sla]도 높아집니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-198">This also provides a higher [service level agreement (SLA)][vm-sla].</span></span>

<span data-ttu-id="f8684-199">사용자의 VM은 [계획된 유지 관리][planned-maintenance] 또는 [계획되지 않은 유지 관리][manage-vm-availability]의 영향을 받을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-199">Your VM may be affected by [planned maintenance][planned-maintenance] or [unplanned maintenance][manage-vm-availability].</span></span> <span data-ttu-id="f8684-200">[VM 다시 부팅 로그][reboot-logs]를 사용하여 VM 재부팅이 계획된 유지 관리로 발생했는지 여부를 결정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-200">You can use [VM reboot logs][reboot-logs] to determine whether a VM reboot was caused by planned maintenance.</span></span>

<span data-ttu-id="f8684-201">정상 작업 중 실수로 인한 데이터 손실(예: 사용자 오류 때문에 발생)을 막기 위해 [Blob 스냅숏][blob-snapshot] 또는 다른 도구를 사용하여 지정 시간 백업도 구현해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-201">To protect against accidental data loss during normal operations (for example, because of user error), you should also implement point-in-time backups, using [blob snapshots][blob-snapshot] or another tool.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="f8684-202">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f8684-202">Manageability considerations</span></span>

<span data-ttu-id="f8684-203">**리소스 그룹**</span><span class="sxs-lookup"><span data-stu-id="f8684-203">**Resource groups.**</span></span> <span data-ttu-id="f8684-204">동일한 수명 주기를 공유하는 긴밀하게 연결된 리소스를 동일한 [리소스 그룹][resource-manager-overview]에 배치합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-204">Put closely associated resources that share the same lifecycle into the same [resource group][resource-manager-overview].</span></span> <span data-ttu-id="f8684-205">리소스 그룹을 사용하여 리소스를 그룹 단위로 배포 및 모니터링하고, 리소스 그룹별로 청구 비용을 추적할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-205">Resource groups allow you to deploy and monitor resources as a group and track billing costs by resource group.</span></span> <span data-ttu-id="f8684-206">리소스를 하나의 집합으로 삭제할 수도 있습니다. 이러한 기능은 테스트 배포에서 매우 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-206">You can also delete resources as a set, which is very useful for test deployments.</span></span> <span data-ttu-id="f8684-207">의미 있는 리소스 이름을 할당하면 간단하게 특정 리소스를 찾고 해당 역할을 이해할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-207">Assign meaningful resource names to simplify locating a specific resource and understanding its role.</span></span> <span data-ttu-id="f8684-208">자세한 내용은 [Azure 리소스에 대해 권장되는 명명 규칙][naming-conventions]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-208">For more information, see [Recommended naming conventions for Azure resources][naming-conventions].</span></span>

<span data-ttu-id="f8684-209">**SSH**.</span><span class="sxs-lookup"><span data-stu-id="f8684-209">**SSH**.</span></span> <span data-ttu-id="f8684-210">Linux VM을 만들기 전에 2048비트 RSA 공개-개인 키 쌍을 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-210">Before you create a Linux VM, generate a 2048-bit RSA public-private key pair.</span></span> <span data-ttu-id="f8684-211">VM을 만들 때 공개 키 파일을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-211">Use the public key file when you create the VM.</span></span> <span data-ttu-id="f8684-212">자세한 내용은 [Azure에서 Linux 및 Mac과 함께 SSH를 사용하는 방법][ssh-linux]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-212">For more information, see [How to Use SSH with Linux and Mac on Azure][ssh-linux].</span></span>

<span data-ttu-id="f8684-213">**VM 중지.**</span><span class="sxs-lookup"><span data-stu-id="f8684-213">**Stopping a VM.**</span></span> <span data-ttu-id="f8684-214">Azure에서는 "중지됨"과 "할당 취소됨" 상태를 구분합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-214">Azure makes a distinction between "stopped" and "deallocated" states.</span></span> <span data-ttu-id="f8684-215">VM 상태가 중지되면 요금이 청구되지만 VM 할당이 취소되면 청구되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-215">You are charged when the VM status is stopped, but not when the VM is deallocated.</span></span> <span data-ttu-id="f8684-216">Azure Portal에서 **중지** 버튼은 VM 할당을 취소합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-216">In the Azure portal, the **Stop** button deallocates the VM.</span></span> <span data-ttu-id="f8684-217">로그인한 상태에서 OS를 통해 종료하면 VM은 중지되지만 할당 취소되지 **않으므로** 비용이 계속 청구됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-217">If you shut down through the OS while logged in, the VM is stopped but **not** deallocated, so you will still be charged.</span></span>

<span data-ttu-id="f8684-218">**VM 삭제.**</span><span class="sxs-lookup"><span data-stu-id="f8684-218">**Deleting a VM.**</span></span> <span data-ttu-id="f8684-219">VM을 삭제하는 경우 VHD는 삭제되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-219">If you delete a VM, the VHDs are not deleted.</span></span> <span data-ttu-id="f8684-220">즉, 데이터 손실 없이 안전하게 VM을 삭제할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-220">That means you can safely delete the VM without losing data.</span></span> <span data-ttu-id="f8684-221">그러나 저장소에 대한 비용은 계속 청구됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-221">However, you will still be charged for storage.</span></span> <span data-ttu-id="f8684-222">VHD를 삭제하려면 [Blob 저장소][blob-storage]에서 파일을 삭제합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-222">To delete the VHD, delete the file from [Blob storage][blob-storage].</span></span> <span data-ttu-id="f8684-223">실수로 삭제하지 않도록 하려면 [리소스 잠금][resource-lock]을 사용하여 전체 리소스 그룹을 잠그거나 VM과 같은 개별 리소스를 잠급니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-223">To prevent accidental deletion, use a [resource lock][resource-lock] to lock the entire resource group or lock individual resources, such as a VM.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="f8684-224">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="f8684-224">Security considerations</span></span>

<span data-ttu-id="f8684-225">[Azure Security Center][security-center]를 사용하여 Azure 리소스의 보안 상태를 중앙에서 살펴볼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-225">Use [Azure Security Center][security-center] to get a central view of the security state of your Azure resources.</span></span> <span data-ttu-id="f8684-226">Security Center는 잠재적인 보안 문제를 모니터링하고 배포의 보안 상태에 대한 종합적인 그림을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-226">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="f8684-227">보안 센터는 각 Azure 구독을 기준으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-227">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="f8684-228">[Azure Security Center 빠른 시작 가이드][security-center-get-started]에 설명된 것처럼 보안 데이터 수집을 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-228">Enable security data collection as described in the [Azure Security Center quick start guide][security-center-get-started].</span></span> <span data-ttu-id="f8684-229">데이터 수집이 사용되도록 설정되면 보안 센터는 해당 구독에서 만든 모든 VM을 자동으로 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-229">When data collection is enabled, Security Center automatically scans any VMs created under that subscription.</span></span>

<span data-ttu-id="f8684-230">**패치 관리.**</span><span class="sxs-lookup"><span data-stu-id="f8684-230">**Patch management.**</span></span> <span data-ttu-id="f8684-231">이 기능이 설정된 경우 Security Center는 보안 및 중요 업데이트 누락 여부를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-231">If enabled, Security Center checks whether any security and critical updates are missing.</span></span> 

<span data-ttu-id="f8684-232">**맬웨어 방지.**</span><span class="sxs-lookup"><span data-stu-id="f8684-232">**Antimalware.**</span></span> <span data-ttu-id="f8684-233">이 기능이 설정되면 보안 센터는 맬웨어 방지 소프트웨어 설치 여부를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-233">If enabled, Security Center checks whether antimalware software is installed.</span></span> <span data-ttu-id="f8684-234">또한 보안 센터를 사용하여 Azure 포털 내에서 맬웨어 방지 소프트웨어를 설치할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-234">You can also use Security Center to install antimalware software from inside the Azure portal.</span></span>

<span data-ttu-id="f8684-235">**작업.**</span><span class="sxs-lookup"><span data-stu-id="f8684-235">**Operations.**</span></span> <span data-ttu-id="f8684-236">[RBAC(역할 기반 액세스 제어)][rbac]를 사용하여 배포하는 Azure 리소스에 대한 액세스를 제어합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-236">Use [role-based access control (RBAC)][rbac] to control access to the Azure resources that you deploy.</span></span> <span data-ttu-id="f8684-237">RBAC를 통해 DevOps 팀의 구성원에게 권한 역할을 할당할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-237">RBAC lets you assign authorization roles to members of your DevOps team.</span></span> <span data-ttu-id="f8684-238">예를 들어 읽기 권한자 역할은 Azure 리소스를 볼 수 있지만 만들거나 관리하거나 삭제할 수는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-238">For example, the Reader role can view Azure resources but not create, manage, or delete them.</span></span> <span data-ttu-id="f8684-239">일부 역할은 특정 Azure 리소스 유형에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-239">Some roles are specific to particular Azure resource types.</span></span> <span data-ttu-id="f8684-240">예를 들어 Virtual Machine Contributor 역할은 VM을 다시 시작하거나 할당을 취소하고, 관리자 암호를 재설정하고, 새 VM을 만드는 등의 작업을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-240">For example, the Virtual Machine Contributor role can restart or deallocate a VM, reset the administrator password, create a new VM, and so on.</span></span> <span data-ttu-id="f8684-241">이 아키텍처에 유용할 수 있는 기타 [기본 제공 RBAC 역할][rbac-roles]에는 [DevTest Labs 사용자][rbac-devtest] 및 [네트워크 참가자][rbac-network]가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-241">Other [built-in RBAC roles][rbac-roles] that may be useful for this architecture include [DevTest Labs User][rbac-devtest] and [Network Contributor][rbac-network].</span></span> <span data-ttu-id="f8684-242">한 명의 사용자가 여러 역할에 할당될 수 있으며 좀 더 세분화된 권한의 사용자 지정 역할을 만들 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-242">A user can be assigned to multiple roles, and you can create custom roles for even more fine-grained permissions.</span></span>

> [!NOTE]
> <span data-ttu-id="f8684-243">RBAC는 VM에 로그온한 사용자가 수행할 수 있는 작업을 제한하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-243">RBAC does not limit the actions that a user logged into a VM can perform.</span></span> <span data-ttu-id="f8684-244">이러한 사용 권한은 게스트 OS의 계정 유형에 따라 결정됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-244">Those permissions are determined by the account type on the guest OS.</span></span>   

<span data-ttu-id="f8684-245">[감사 로그][audit-logs]를 사용하여 프로비전 동작 및 기타 VM 이벤트를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-245">Use [audit logs][audit-logs] to see provisioning actions and other VM events.</span></span>

<span data-ttu-id="f8684-246">**데이터 암호화.**</span><span class="sxs-lookup"><span data-stu-id="f8684-246">**Data encryption.**</span></span> <span data-ttu-id="f8684-247">OS 및 데이터 디스크를 암호화해야 하는 경우 [Azure Disk Encryption][disk-encryption]을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-247">Consider [Azure Disk Encryption][disk-encryption] if you need to encrypt the OS and data disks.</span></span> 

## <a name="deploy-the-solution"></a><span data-ttu-id="f8684-248">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="f8684-248">Deploy the solution</span></span>

<span data-ttu-id="f8684-249">배포는 [GitHub][github-folder]에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-249">A deployment is available on [GitHub][github-folder].</span></span> <span data-ttu-id="f8684-250">다음을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-250">It deploys the following:</span></span>

  * <span data-ttu-id="f8684-251">VM을 호스트하는 데 사용되는 **웹**이라는 이름의 단일 서브넷을 가진 가상 네트워크</span><span class="sxs-lookup"><span data-stu-id="f8684-251">A virtual network with a single subnet named **web** used to host the VM.</span></span>
  * <span data-ttu-id="f8684-252">SSH 및 HTTP 트래픽이 VM으로 이동할 수 있도록 허용하는 두 개의 들어오는 규칙이 있는 NSG</span><span class="sxs-lookup"><span data-stu-id="f8684-252">An NSG with two incoming rules to allow SSH and HTTP traffic to the VM.</span></span>
  * <span data-ttu-id="f8684-253">최신 버전의 Ubuntu16.04.3 LTS를 실행하는 VM</span><span class="sxs-lookup"><span data-stu-id="f8684-253">A VM running the latest version of Ubuntu 16.04.3 LTS.</span></span>
  * <span data-ttu-id="f8684-254">두 개의 데이터 디스크를 포맷하고 Apache HTTP Server를 Ubuntu VM에 배포하는 샘플 사용자 지정 스크립트 확장</span><span class="sxs-lookup"><span data-stu-id="f8684-254">A sample custom script extension that formats the two data disks and deploys Apache HTTP Server to the Ubuntu VM.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f8684-255">필수 조건</span><span class="sxs-lookup"><span data-stu-id="f8684-255">Prerequisites</span></span>

1. <span data-ttu-id="f8684-256">[참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-256">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="f8684-257">Azure CLI 2.0이 컴퓨터에 설치되어 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-257">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="f8684-258">CLI 설치 지침은 [Install Azure CLI 2.0][azure-cli-2](Azure CLI 2.0 설치)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-258">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="f8684-259">[Azure 빌딩 블록][azbb] npm 패키지를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-259">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="f8684-260">명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 다음 명령을 입력하여 Azure 계정에 로그인합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-260">From a command prompt, bash prompt, or PowerShell prompt, enter the following command to log into your Azure account.</span></span>

  ```bash
  az login
  ```

5. <span data-ttu-id="f8684-261">SSH 키 쌍을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-261">Create an SSH key pair.</span></span> <span data-ttu-id="f8684-262">자세한 내용은 [Azure에서 Linux VM용 SSH 공개 및 개인 키 쌍을 만들고 사용하는 방법](/azure/virtual-machines/linux/mac-create-ssh-keys)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="f8684-262">For more information, see [How to create and use an SSH public and private key pair for Linux VMs in Azure](/azure/virtual-machines/linux/mac-create-ssh-keys).</span></span>

### <a name="deploy-the-solution-using-azbb"></a><span data-ttu-id="f8684-263">azbb를 사용하여 솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="f8684-263">Deploy the solution using azbb</span></span>

1. <span data-ttu-id="f8684-264">위의 필수 조건 단계에서 다운로드한 리포지토리의 `virtual-machines/single-vm/parameters/linux` 폴더로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-264">Navigate to the `virtual-machines/single-vm/parameters/linux` folder for the repository you downloaded in the prerequisites step above.</span></span>

2. <span data-ttu-id="f8684-265">`single-vm-v2.json` 파일을 열고 큰따옴표 사이에 사용자 이름과 SSH 공개 키를 입력한 다음, 파일을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-265">Open the `single-vm-v2.json` file and enter a username and your SSH public key between the quotes, then save the file.</span></span>

  ```bash
  "adminUsername": "<your username>",
  "sshPublicKey": "ssh-rsa AAAAB3NzaC1...",
  ```

3. <span data-ttu-id="f8684-266">아래 표시된 대로 `azbb`를 실행하여 샘플 VM을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-266">Run `azbb` to deploy the sample VM as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p single-vm-v2.json --deploy
  ```

<span data-ttu-id="f8684-267">배포를 확인하려면 다음 Azure CLI 명령을 실행하여 VM의 공용 IP 주소를 찾습니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-267">To verify the deployment, run the following Azure CLI command to find the public IP address of the VM:</span></span>

```bash
az vm show -n ra-single-linux-vm1 -g <resource-group-name> -d -o table
```

<span data-ttu-id="f8684-268">웹 브라우저에서 이 주소로 이동하면 기본 Apache2 홈 페이지가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="f8684-268">If you navigate to this address in a web browser, you should see the default Apache2 homepage.</span></span>

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
[managed-disks]: /azure/storage/storage-managed-disks-overview
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-size-tables]: /azure/virtual-machines/virtual-machines-linux-sizes
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[0]: ./images/single-vm-diagram.png "Azure의 단일 Linux VM"
