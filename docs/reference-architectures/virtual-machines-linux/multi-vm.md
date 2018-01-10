---
title: "확장성 및 가용성을 위해 Azure에서 부하 분산된 VM 실행"
description: "확장성 및 가용성을 위해 Azure에서 복수의 Linux VM을 실행하는 방법."
author: telmosampaio
ms.date: 11/16/2017
pnp.series.title: Linux VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: b1b3c94524d50d05c90b46d26cab54fea8c8061a
ms.sourcegitcommit: 115db7ee008a0b1f2b0be50a26471050742ddb04
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/17/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>확장성 및 가용성을 위해 부하 분산된 VM 실행

이 참조 아키텍처에서는 확장성과 가용성을 개선하기 위해 부하 분산 장치 뒤에서 확장 집합에 포함된 복수의 Linux VM(가상 머신)을 실행하는 검증된 모범 사례를 보여줍니다. 이 아키텍처는 웹 서버와 같은 상태 비저장 워크로드에 사용할 수 있므며, N 계층 응용 프로그램 배포를 위한 기초가 됩니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>건축

이 아키텍처는 [단일 VM 참조 아키텍처][single-vm]를 기반으로 합니다. 이 아키텍처에는 단일 VM 참조 아키텍처의 권장 사항이 적용됩니다.

이 아키텍처에서 워크로드는 여러 VM 인스턴스에 배포됩니다. 공용 IP 주소는 한 개가 사용되며, 부하 분산 장치에 의해 인터넷 트래픽이 여러 VM으로 분산됩니다. 이 아키텍처는 상태 비저장 웹 응용 프로그램과 같은 단일 계층 응용 프로그램에 사용할 수 있습니다.

이 아키텍처의 구성 요소는 다음과 같습니다.

* **리소스 그룹.** [리소스 그룹][resource-manager-overview]은 리소스를 수명, 소유자를 비롯한 기준으로 관리할 수 있도록 리소스를 그룹화하는 데 사용됩니다.
* **가상 네트워크(VNet) 및 서브넷.** 모든 Azure VM은 VNet에 배포되어 여러 서브넷으로 분할될 수 있습니다.
* **Azure Load Balancer**. [부하 분산 장치][load-balancer]는 수신되는 인터넷 요청을 여러 VM 인스턴스로 분산합니다. 
* **공용 IP 주소**. 부하 분산 장치가 인터넷 트래픽을 수신하려면 공용 IP 주소가 필요합니다.
* **VM 확장 집합**. [VM 확장 집합][vm-scaleset]은 워크로드를 호스팅하는 데 사용되는 동일한 VM들의 집합입니다. 확장 집합을 사용하면 여러 VM을 수동 확장 또는 축소하거나 사전 정의된 규칙을 바탕으로 자동 확장 또는 축소할 수 있습니다.
* **가용성 집합**. [가용성 집합][availability-set]은 여러 VM을 포함합니다. 포함된 VM은 더 높은 [SLA(서비스 수준 계약)][vm-sla]를 충족할 수 있게 됩니다. 더 높은 SLA를 적용하기 위해서는 가용성 집합에 둘 이상의 VM이 포함되어야 합니다. 가용성 집합은 확장 집합에 내포되어 있습니다. VM을 확장 집합 외부에 만든 경우에는 가용성 집합을 독립적으로 만들어야 합니다.
* **관리 디스크**. Azure 관리 디스크는 VM 디스크의 VHD(가상 하드 디스크) 파일을 관리합니다. 
* **저장소**. 여러 VM의 진단 로그를 저장하기 위한 Azure Storage 계정을 만듭니다.

## <a name="recommendations"></a>권장 사항

사용자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다. 여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요. 

### <a name="availability-and-scalability-recommendations"></a>가용성 및 확장성 권장 사항

가용성 및 확장성을 높이기 위한 한 가지 방법은 [가상 머신 확장 집합][vmss]을 사용하는 것입니다. VM 확장 집합을 사용하면 동일한 VM으로 구성된 집합을 배포 및 관리할 수 있습니다. 확장 집합은 성능 메트릭을 바탕으로 자동 확장을 지원합니다. VM들의 부하가 늘어나면 부하 분산 장치에 VM이 자동으로 추가됩니다. VM을 신속하게 확장해야 하거나 자동 확장이 필요한 경우 확장 집합을 사용합니다.

기본적으로 확장 집합은 "오버프로비전"을 사용합니다. 즉, 사용자에게 필요한 VM보다 더 많은 VM을 처음부터 프로비전한 다음 여분의 VM을 삭제합니다. 이로 인해 VM 프로비전 시의 전반적인 성공률이 높아집니다. [관리 디스크](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks)를 사용하지 않는 경우에는 오버프로비전이 적용된 저장소 계정 하나당 최대 20개의 VM을, 오버프로비전이 적용되지 않은 저장소 계정 하나당 최대 40개의 VM을 사용하는 것이 좋습니다.

확장 집합에 배포된 VM을 구성하는 방법에는 두 가지가 있습니다.

- VM이 프로비전된 뒤에 확장명을 사용하여 VM을 구성합니다. 이렇게 하면 확장명이 없는 VM보다 새 VM 인스턴스를 시작하는 데 시간이 더 오래 걸릴 수 있습니다.

- 사용자 지정 디스크 이미지를 사용하여 [관리 디스크](/azure/storage/storage-managed-disks-overview)를 배포합니다. 이 옵션을 사용하면 배포 시간이 단축될 수 있습니다. 단, 이 방법의 경우 사용자가 이미지를 최신 상태로 유지해야 합니다.

추가 고려 사항은 [확장 집합의 설계 고려 사항][vmss-design]을 참조하세요.

> [!TIP]
> 자동 확장 솔루션을 사용할 때는 사전에 미리 프로덕션 수준 워크로드로 테스트해야 합니다.

확장 집합을 사용하지 않는 경우에는 최소한 가용성 집합이라도 사용하는 것이 좋습니다. [Azure VM의 가용성 SLA][vm-sla]를 지원하기 위해 가용성 집합에 2개 이상의 VM을 만듭니다. Azure Load Balancer를 사용하려면 부하 분산된 VM이 동일한 가용성 집합에 속해 있어야 합니다.

각 Azure 구독에는 지역당 최대 VM 개수를 비롯해 기본적인 제한이 적용됩니다. 지원 요청을 제출하여 제한을 늘릴 수 있습니다. 자세한 내용은 [Azure 구독 및 서비스 제한, 할당량 및 제약 조건][subscription-limits]을 참조하세요.

### <a name="network-recommendations"></a>네트워크 권장 사항

여러 VM을 동일한 서브넷에 배포합니다. VM을 인터넷에 직접 노출시키는 대신 각 VM에 사설 IP 주소를 부여합니다. 클라이언트는 부하 분산 장치의 공용 IP 주소를 사용하여 연결합니다.

부하 분산 장치 뒤에 있는 VM에 로그인해야 하는 경우에는 로그인에 사용할 수 있는 공용 IP 주소를 갖는 jumpbox(요새 호스트)로서 VM 하나를 추가하는 방법을 고려할 수 있습니다. 이렇게 하면 jumpbox에서 부하 분산 장치 뒤에 있는 VM에 로그인할 수 있습니다. 또는 부하 분산 장치의 인바운드 NAT(Network Address Translation) 규칙을 구성할 수도 있습니다. N 계층 워크로드나 여러 개의 워크로드를 처리해야 하는 경우에는 jumpbox를 구성하는 것이 더 좋습니다.

### <a name="load-balancer-recommendations"></a>부하 분산 장치 권장 사항

가용성 집합에 속한 모든 VM을 부하 분산 장치의 백엔드 주소 풀에 추가합니다.

네트워크 트래픽이 VM으로 전달되도록 부하 분산 장치 규칙을 정의합니다. 예를 들어 HTTP 트래픽을 허용하려면 프론트 엔드 구성의 포트 80을 백엔드 주소 풀의 포트 80으로 매핑하는 규칙을 만듭니다. 클라이언트가 포트 80으로 HTTP 요청을 전송하면 부하 분산 장치가 소스 IP 주소를 포함하는 [해싱 알고리즘][load-balancer-hashing]을 사용하여 백엔드 IP 주소를 선택합니다. 클라이언트 요청은 이런 식으로 모든 VM에 걸쳐 분산됩니다.

트래픽을 특정 VM으로 라우팅하려면 NAT 규칙을 사용합니다. 예를 들어 VM에서 RDP를 사용할 수 있도록 하려면 각 VM별로 NAT 규칙을 만듭니다. 각 규칙은 서로 다른 포트 번호를 RDP의 기본 포트인 포트 3389로 매핑해야 합니다. 예를 들어 "VM1"에는 포트 50001을, "VM2"에는 포트 50002를 사용합니다. VM상의 NIC에 NAT 규칙을 할당합니다.

### <a name="storage-account-recommendations"></a>저장소 계정 권장 사항

[프리미엄 저장소][premium]가 있는 [관리 디스크](/azure/storage/storage-managed-disks-overview)를 사용하는 것이 좋습니다. 관리 디스크에는 저장소 계정이 필요하지 않습니다. 디스크의 크기와 유형을 지정하기만 하면 고가용성 리소스로 배포됩니다.

관리되지 않는 디스크를 사용 중인 경우에는 저장소 계정의 [IOPS 제한][vm-disk-limits]에 도달하지 않도록 각 VM에 대해 VHD(가상 하드 디스크)를 갖는 별도의 Azure 저장소 계정을 만듭니다.

진단 로그를 위한 하나의 저장소 계정을 만듭니다. 이 저장소 계정은 모든 VM이 공유할 수 있습니다. 진단 로그를 위한 저장소 계정은 표준 디스크를 사용하는 관리되지 않는 저장소 계정이어도 됩니다.

## <a name="availability-considerations"></a>가용성 고려 사항

가용성 집합을 사용하면 계획된 유지 관리 이벤트와 계획되지 않은 유지 관리 이벤트에 응용 프로그램이 복원력 있게 대처하게 됩니다.

* *계획된 유지 관리*는 Microsoft가 기반 플랫폼을 업데이트하는 경우 일어납니다. 계획된 유지 관리 시에는 VM이 다시 시작되기도 합니다. Azure는 가용성 집합에 속한 VM들이 모두 동시에 다시 시작되지 않도록 관리합니다. 다른 VM들이 다시 시작되는 동안에 적어도 하나는 가동 상태로 유지됩니다.
* *계획되지 않은 유지 관리*는 하드웨어에 장애가 발생하는 경우 일어납니다. Azure는 가용성 집합에 속한 VM들이 둘 이상의 서버 랙에 걸쳐 프로비전되도록 관리합니다. 이로 인해 하드웨어 장애, 네트워크 중단, 전원 중단과 같은 문제의 영향이 줄어듭니다.

자세한 내용은 [가상 머신의 가용성 관리][availability-set]를 참조하세요. [VM 확장을 위해 가용성 집합을 구성하는 방법][availability-set-ch9] 비디오에서는 가용성 집합에 대한 개요를 확인할 수 있습니다.

> [!WARNING]
> VM을 프로비전할 때는 반드시 가용성 집합을 구성해야 합니다. 현재 VM이 프로비전된 뒤에 리소스 관리자 VM을 가용성 집합에 추가하는 방법은 없습니다.

부하 분산 장치는 [상태 프로브][health-probes]를 사용하여 VM 인스턴스의 가용성을 모니터링합니다. 프로브가 특정 시간 안에 인스턴스에 도달하지 못한 경우, 부하 분산 장치가 해당 VM으로 전달되는 트래픽을 차단합니다. 이때도 부하 분산 장치가 프로브를 계속 진행하며, VM을 다시 사용할 수 있게 되면 해당 VM으로 전달되는 트래픽을 재개합니다.

다음은 부하 분산 장치 상태 프로브에 대한 몇 가지 권장 사항입니다.

* 프로브는 HTTP와 TCP를 모두 테스트할 수 있습니다. VM이 HTTP 서버를 실행 중인 경우에는 HTTP 프로브를 만들고, HTTP 서버를 실행하지 않는 경우에는 TCP 프로브를 만듭니다.
* HTTP 프로브를 만들 때는 HTTP 끝점으로 향하는 경로를 지정합니다. 프로브는 이 경로에서 HTTP 200 응답을 확인합니다. 경로는 루트 경로("/")일 수도 있고, 응용 프로그램의 상태를 확인하는 사용자 지정 로직을 구현하는 상태 모니터링 끝점일 수도 있습니다. 끝점은 익명의 HTTP 요청을 허용해야 합니다.
* 프로브는 [알려진 IP 주소][health-probe-ip]인 168.63.129.16에서 전송됩니다. 방화벽 정책이나 NSG(네트워크 보안 그룹) 규칙에서 이 IP 주소로의 트래픽 송수신을 차단하지 않도록 합니다.
* [상태 프로브 로그][health-probe-log]를 사용하여 상태 프로브의 상태를 확인합니다. Azure Portal에서 각 부하 분산 장치에 대해 로깅을 활성화합니다. 로그는 Azure Blob Storage에 쓰기됩니다. 이 로그에서는 백엔드의 VM 중 실패한 프로브 응답으로 인해 네트워크 트래픽을 수신하지 못하는 VM의 개수를 확인할 수 있습니다.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

VM이 여러 개인 경우에는 프로세스를 안정적이고 반복적으로 수행하기 위해 자동화하는 것이 중요합니다. [Azure Automation][azure-automation]을 사용하여 배포, OS 패치 및 기타 작업을 자동화할 수 있습니다.

## <a name="security-considerations"></a>보안 고려 사항

가상 네트워크는 Azure의 트래픽 격리 경계입니다. 하나의 VNet에 속한 VM은 다른 VNet에 속한 VM과 직접 통신할 수 없습니다. 하나의 VNet에 속한 여러 VM은 사용자가 트래픽을 제한하기 위해 NSG([네트워크 보안 그룹][nsg])를 만들지 않은 이상 서로 통신할 수 있습니다. 자세한 내용은 [Microsoft 클라우드 서비스 및 네트워크 보안][network-security]을 참조하세요.

수신되는 인터넷 트래픽의 경우 부하 분산 장치의 규칙이 어느 트래픽이 백엔드에 도달할 수 있는지 정의합니다. 단, 부하 분산 장치의 규칙은 IP 안전 목록을 지원하지 않으므로 안전 목록에 특정 공용 IP 주소를 추가하려면 서브넷에 NSG를 추가해야 합니다.

## <a name="deploy-the-solution"></a>솔루션 배포

이 아키텍처에 대한 배포는 [GitHub][github-folder]에서 사용할 수 있습니다. 다음을 배포합니다.

  * **web**이라는 이름을 가지며 VM을 포함하는 하나의 서브넷이 있는 가상 네트워크.
  * 최신 버전의 Ubuntu 16.04.3 LTS를 실행하는 VM을 포함하는 VM 확장 집합. 자동 확장이 활성화되어 있음.
  * VM 확장 집합 앞에 위치한 부하 분산 장치.
  * VM 확장 집합에서 HTTP 트래픽을 수신할 수 있도록 허용하는 수신 규칙을 갖는 NSG.

### <a name="prerequisites"></a>필수 조건

사용자의 구독에 참조 아키텍처를 배포하려면 먼저 다음 단계를 수행해야 합니다.

1. [AzureCAT 참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.

2. Azure CLI 2.0이 컴퓨터에 설치되어 있는지 확인합니다. CLI 설치 지침은 [Install Azure CLI 2.0][azure-cli-2](Azure CLI 2.0 설치)을 참조하세요.

3. [Azure 빌딩 블록][azbb] npm 패키지를 설치합니다.

4. 명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 다음 명령 중 하나를 사용하여 Azure 계정에 로그인한 다음 프롬프트에 따릅니다.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>azbb를 사용하여 솔루션 배포

샘플 단일 VM 워크로드를 배포하려면 다음 단계를 따릅니다.

1. 위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `virtual-machines\multi-vm\parameters\linux` 폴더로 이동합니다.

2. `multi-vm-v2.json` 파일을 열고 아래 나와 있는 대로 큰따옴표 사이에 사용자 이름과 SSH 키를 입력한 다음 파일을 저장합니다.

  ```bash
  "adminUsername": "",
  "sshPublicKey": "",
  ```

3. 아래 표시된 대로 `azbb`를 실행하여 VM을 배포합니다.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

이 샘플 참조 아키텍처를 배포하는 방법에 대한 자세한 내용은 [GitHub 리포지토리][git]를 참조하세요.

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability-set-ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-automation]: /azure/automation/automation-intro
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[naming-conventions]: ../../best-practices/naming-conventions.md
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[premium]: /azure/storage/common/storage-premium-storage
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[runbook-gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[single-vm]: single-vm.md
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[0]: ./images/multi-vm-diagram.png "2개의 VM과 1개의 부하 분산 장치를 갖는 가용성 집합으로 구성된 Azure의 다중 VM 솔루션의 아키텍처"
