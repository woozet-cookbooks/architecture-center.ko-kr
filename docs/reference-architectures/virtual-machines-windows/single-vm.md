---
title: Run a Windows VM on Azure
description: >-
  How to run a VM on Azure, paying attention to scalability, resiliency,
  manageability, and security.

author: MikeWasson

ms.service: guidance
ms.topic: article
ms.date: 11/22/2016
ms.author: pnp

pnp.series.title: Windows VM workloads
pnp.series.next: multi-vm
pnp.series.prev: ./index
---

# Azure에서 Windows VM 실행

이 참조 아키텍처는 Azure에서 Windows 가상 컴퓨터(VM)를 실행하기 위한 일련의 검증된 사례들을 보여줍니다. 여기에는 네트워킹 및 저장소 구성요소와 더불어 VM의 프로비전에 관한 권장사항이 포함됩니다. 이 아키텍처는 단일 인스턴스 실행에 사용될 수 있고 N계층 응용 프로그램과 같은 보다 복잡한 아키텍처의 기초가 됩니다. [**이 솔루션 배포하기**.](#deploy-the-solution)

![[0]][0]

## 아키텍처

Azure에서 VM을 프로비전하는 데는 VM 외에도 많은 유동적 요소들이 관여하는데, 고려해야 할 요소로는 컴퓨팅, 네트워킹, 저장소 요소 등을 꼽을 수 있습니다.

* **리소스 그룹.** [*리소스 그룹*][resource-manager-overview]은 관련 리소스들을 보관하는 컨테이너입니다. 따라서 이 VM을 위한 리소스들을 보관할 수 있는 리소스 그룹을 생성해야 합니다.
* **VM**. 출판된 이미지 목록 또는 사용자가 Azure Blob 저장소에 업로드한 가상 하드 디스크(VHD)를 통해 VM 프로비전을 수행할 수 있습니다.
* **OS 디스크.** OS 디스크는 [Azure Storage][azure-storage]에 저장된 가상 하드 디스크(VHD)로. 호스트 컴퓨터의 작동이 중단되더라도 계속 작동합니다.
* **임시 디스크.** VM은 임시 디스크(Windows의 `D:` 드라이브)에 생성됩니다. 이 디스크는 호스트 컴퓨터의 물리적 드라이브에만 저장되고 Azure Storage에는 저장되지 *않으며* 재부팅이나 기타 VM 수명 주기 이벤트 시 삭제될 수 있습니다. 따라서 이 디스크는 페이지나 스왑 파일 등 임시 데이터용으로만 사용할 것을 권장합니다.
* **데이터 디스크.** [데이터 디스크][data-disk]는 응용 프로그램 데이터용으로 사용되는 영구적 가상 하드 디스크(VHD)로, OS 디스크처럼 Azure Storage에 저장됩니다.
* **가상 네트워크(VNet)와 서브넷.** Azure 내 모든 VM은 서브넷으로 나뉘게 되는 VNet에 배포됩니다.
* **공용 IP 주소.** VM과 통신(예: 원격 데스크톱(RDP)을 통한 통신 등)하기 위해 공용 IP 주소가 필요합니다.
* **네트워크 인터페이스(NIC)**. 네트워크 인터페이스(NIC)는 VM이 가상 네트워크와 통신할 수 있도록 지원합니다.
* **네트워크 보안 그룹(NSG)**. [네트워크 보안 그룹(NSG)][nsg]은 서브넷에 네트워크 트래픽을 허용하거나 거부하는 데 사용됩니다. 사용자는 NSG를 개별 네트워크 인터페이스(NIC) 또는 서브넷과 연관지을 수 있습니다. NSG를 서브넷과 연관짓는 경우, 해당 서브넷의 모든 VM에 NSG 규칙이 적용됩니다.
* **진단.** 진단 로깅은 VM의 관리 및 문제해결에 매우 중요한 역할을 합니다.

You can download a [Visio file](https://aka.ms/arch-diagrams) of this architecture.

> [!참고]
> Azure는  [Azure Resource Manager][resource-manager-overview]와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 문서에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다. 
> 

## 권장사항

이 아키텍처는 Azure에서 Windows VM을 실행하기 위한 기초적인 권장사항을 보여줍니다. 그러나 단일 실패 지점을 생성하므로 중요한 업무용 작업에는 단일 VM을 사용하지 않을 것을 권장합니다. 가용성을 높이려면 [가용성 집합][availability-set]에 여러 VM을 배포해야 합니다. 자세한 내용은 [Azure에서 여러 VM 실행][multi-vm]을 참조하시기 바랍니다. 

### VM 권장사항

Azure는 매우 다양한 크기의 VM을 제공하지만 [Premium Storage][premium-storage]를 지원하는 크기의 DS 시리즈 및 GS 시리즈를 권장합니다. 고성능 컴퓨팅과 같은 특수한 워크로드에 사용하는 경우가 아니라면 이 크기의 컴퓨터 중 하나를 선택하세요.  자세한 내용은 [VM 크기][virtual-machine-sizes]를 참조하시기 바랍니다. 

기존 워크로드를 Azure로 옮기려면 기존 온-프레미스 서버와 가장 유사한 크기의 VM으로 시작한 다름 CPU, 메모리, 초당 디스크 I/O 횟수(IOPS)의 관점에서 실제 워크로드 성능을 측정하여 필요한 경우 크기를 조절하면 됩니다. VM에 여러 네트워크 인터페이스(NIC)가 필요한 경우, 네트워크 인터페이스(NIC)의 최대수는 [VM 크기][vm-size-tables]와 함수 관계에 있다는 점에 주의하시기 바랍니다.   

VM과 기타 리소스를 프로비전하는 경우 지역을 지정해야 합니다. 보통은 내부 사용자나 고객과 가장 가까운 지역을 선택하지만 모든 지역에서 모든 VM 크기를 지원하는 것은 아닙니다. 자세한 내용은 [지역별 서비스][services-by-region]를 참조하시기 바랍니다. 특정 지역에서 활용할 수 있는 VM 크기 목록을 확인하려면 다음 Azure 명령줄 인터페이스(CLI) 명령어를 사용하시기 바랍니다. 

```
azure vm sizes --location <location>
```

출판된 VM 이미지 선택에 관한 정보는 [Powershell 또는 CLI를 사용하여 Azure에서 Windows VM 이미지 검색 및 선택][select-vm-image]을 참조하시기 바랍니다.

### 디스크 및 저장소 권장사항

최상의 디스크 I/O 성능을 위해 데이터를 SSD에 저장하는 [Premium Storage][premium-storage]를 권장합니다. 비용은 프로비전된 디스크의 크기에 따라 산정됩니다. 초당 디스크 I/O 횟수(IOPS)와 처리량은 디스크 크기에 따라 달라지므로 디스크를 프로비전할 때는 용량, IOPS, 처리량의 세 가지 요소를 모두 고려해야 합니다. 

저장소 계정에 대한 IOPS 한도에 도달하지 않도록 각각의 VM별로 가상 하드 디스크(VHD)를 담기 위한 별도의 Azure 저장소 계정을 생성합니다.  

하나 이상의 데이터 디스크를 추가합니다. 새로 생성하는 가상 하드 디스크는 포맷되지 않은 상태입니다. VM에 로그인해 디스크를 포맷합니다. 데이터 디스크 수가 많은 경우에는 저장소 계정의 I/O 총한도에 주의해야 합니다. 자세한 내용은 [VM 디스크 한도][vm-disk-limits]를 참조하시기 바랍니다.

응용 프로그램은 되도록 OS 디스크가 아닌 데이터 디스크에 설치합니다. 그러나 일부 레거시 응용 프로그램은 C: 드라이브에 요소들을 설치해야 할 수도 있습니다. 이 경우, PowerShell을 이용하여 [OS 디스크의 크기 조정][resize-os-disk]을 실시할 수 있습니다.

최상의 성능을 위해 진단 로그를 저장할 별도의 저장소 계정을 생성해야 하는데, 표준 로컬 중복 저장소(LRS) 계정이면 진단 로그 저장용으로 충분합니다.

### 네트워크 권장사항

공용 IP 주소는 동적 또는 정적인데, 기본 설정은 동적 IP 주소입니다. 

* DNS에 A 기록을 생성하거나 안전 목록에 IP 주소를 추가하는 등 변경되지 않는 고정 IP 주소가 필요한 경우에는 [정적 IP 주소][static-ip]를 예약합니다.
* IP 주소에 대한 정규화된 도메인 이름(FQDN)을 생성할 수도 있습니다. 그런 다음 해당 FQDN을 가리키는 DNS에 [CNAME 기록][cname-record]을 등록할 수 있습니다. 자세한 내용은 [Azure 포털에서 정규화된 도메인 이름(FQDN) 생성][fqdn]을 참조하시기 바랍니다.

모든 네트워크 보안 그룹(NSG)은 모든 인바운드 인터넷 트래픽을 차단하는 규칙을 포함한 일련의 [기본 규칙][nsg-default-rules]을 포함합니다. 기본 규칙은 삭제가 불가능하지만 그 밖의 규칙들은 덮어쓰기가 가능합니다. 인터넷 트래픽을 허용하려면 특정 포트(예: HTTP를 위한 포트 80)에 대한 인바운드 트래픽을 허용하는 규칙을 생성합니다.   

RDP를 사용하려면 TCP 포트 3389에 대한 인바운드 트래픽을 허용하는 NSG 규칙을 추가합니다.

## 확장성 고려사항

[VM 크기 조정][vm-resize]을 통해 VM의 크기를 확장 또는 축소할 수 있습니다. 크기를 확장하려면 부하 분산 장치 뒤에 두 개 이상의 VM을 가용성 집합에 추가합니다. 자세한 내용은 [더 높은 확장성과 가용성을 위해 Azure에서 둘 이상의 VM 실행][multi-vm]을 참조하시기 바랍니다.

## 가용성 고려사항

가용성을 높이려면 가용성 집합에 여러 VM을 배포해야 합니다. 이를 통해 더 높은 수준의 [서비스 수준 계약][vm-sla] (SLA)을 달성할 수 있습니다.

VM은 [계획된 유지관리][planned-maintenance] 또는 [계획되지 않은 유지관리][manage-vm-availability]의 영향을 받을 수 있습니다. [VM 재부팅 로그][reboot-logs]를 통해 VM 재부팅이 계획된 유지관리에 의해 실시되었는지 여부를 확인할 수 있습니다. 

가상 하드 디스크(VHD)는 [Azure 저장소][azure-storage]에 저장되고, Azure 저장소를 복제하여 내구성과 가용성을 보장합니다.  

정상 작업 중 실수로 인한 데이터 손실(예: 사용자 오류로 인한 데이터 손실)을 막기 위해 [Blob 스냅샷][blob-snapshot] 또는 다른 도구를 사용하여 지정 시간 백업도 구현해야 합니다. 

## 관리 효율성 고려사항

**리소스 그룹.** 동일한 수명 주기를 공유하는 긴밀하게 결합된 리소스를 동일한 [리소스 그룹][resource-manager-overview]에 추가합니다. 리소스 그룹을 사용하여 리소스를 그룹 단위로 배포 및 모니터링하고, 리소스 그룹별로 비용을 청구할 수 있습니다. 또한 리소스들을 하나의 집합으로 삭제할 수 있어 테스트 배포 시 매우 유용합니다. 리소스에 의미 있는 이름을 지정하면 특정 리소스를 보다 쉽게 찾고 그 역할을 이해할 수 있습니다. [권장 Azure 리소스명][naming conventions]을 참조하시기 바랍니다.

**VM 진단.** 기본 상태 메트릭, 진단 인프라 로그, [부팅 진단][boot-diagnostics]을 포함한 모니터링 및 진단 기능을 사용하도록 설정할 수 있습니다. VM 부팅이 되지 않는 경우 부팅 진단을 통해 부팅 실패를 진단할 수 있습니다. 자세한 내용은 [모니터링 및 진단 사용][enable-monitoring]을 참조하시기 바랍니다. [Azure 로그 수집][log-collector] 확장 프로그램을 사용하면 Azure 플랫폼 로그를 수집하여 Azure 저장소에 업로드할 수 있습니다.    

다음과 같은 CLI 명령어를 통해 진단을 사용 가능하도록 설정할 수 있습니다:

```
azure vm enable-diag <resource-group> <vm-name>
```

**VM 정지.** Azure는 "정지(stopped)" 상태와 "해제(deallocated)" 상태를 구분합니다. VM이 "정지" 상태일 때는 요금이 부과되지만 "해제" 상태에서는 부과되지 않습니다. 

다음과 같은 CLI 명령어를 사용하여 VM의 할당을 해제하십시오:

```
azure vm deallocate <resource-group> <vm-name>
```

Azure 포털에 있는 **정지(Stop)** 단추를 누르면 해당 VM의 할당이 해제됩니다. 그러나 로그인된 상태에서 OS를 통해 정지시키는 경우 VM은 정지되지만 해제되는 것은 *아니므로* 계속해서 요금이 부과됩니다.

**VM 삭제.** VM을 삭제하더라도 가상 하드 디스크(VHD)는 삭제되지 않습니다. 따라서 데이터 손실 걱정 없이 안전하게 VM을 삭제할 수 있습니다. 그러나 저장소 사용에 대한 요금은 계속해서 부과됩니다. VHD를 삭제하려면 [Blob 저장소][blob-storage]로부터 해당 파일을 삭제합니다. 

실수로 삭제하지 않도록 [리소스 잠금][resource-lock]을 사용하여 전체 리소스 그룹을 잠그거나 또는 해당 VM과 같은 개별 리소스를 잠급니다.

## 보안 고려사항

[Azure Security Center][security-center]를 사용하여 Azure 리소스의 보안 상태를 한 눈에 확인할 수 있습니다. Security Center는 잠재적 보안 문제를 모니터링하고 배포된 리소스의 종합적인 보안 상태를 보여줍니다. Security Center의 환경설정은 Azure 구독별로 이루어집니다. [Security Center 이용]을 참조하여 보안 데이터 수집 기능을 활성화할 수 있습니다. 데이터 수집 기능이 활성화되면 Security Center가 해당 구독 하에 생성된 모든 VM을 자동으로 스캔합니다.

**패치 관리.** 패치 관리 기능을 켜면 Security Center가 필요한 보안 업데이트와 기타 주요 업데이트가 있는지 확인합니다. VM의 [그룹 정책 설정][group-policy]을 활용하여 자동 시스템 업데이트 기능을 활성화할 수 있습니다.

**멀웨어 방지 프로그램.** 멀웨어 방지 기능을 활성화하면 Security Center에서 멀웨어 방지 프로그램이 설치되어 있는지 확인합니다. 또한 Security Center를 통해 Azure 포털에서 멀웨어 방지 프로그램을 설치할 수도 있습니다. 

**실행.** [역할 기반 액세스 제어][rbac](RBAC)를 사용하여 배포된 Azure 리소스에 대한 액세스를 관리하십시오. RBAC를 이용하여 DevOps팀 구성원에게 역할을 부여할 수 있습니다. 예를 들어, Reader 역할로는 Azure 리소스를 볼 수만 있고 생성, 관리, 삭제할 수는 없습니다. 일부 역할은 특정 Azure 리소스 유형에만 한정됩니다. 예를 들어, Virtual Machine Contributor 역할에는 VM을 재시작 또는 해제하고 관리자 암호를 재설정하며 새 VM을 생성하는 등의 권한이 부여됩니다. 이 아키텍처에 유용한 그 밖의 [기본 RBAC 역할][rbac-roles]로는 [DevTest Labs User][rbac-devtest]와 [Network Contributor][rbac-network] 등이 있습니다. 한 사용자에게 여러 가지 역할을 부여할 수 있고, 권한을 보다 세분화해 사용자 지정할 수도 있습니다. 

> [!참고]
> RBAC는 VM에 로그인한 사용자가 수행할 수 있는 동작을 제한하지 않습니다. 이러한 권한들은 게스트 OS 상의 계정 유형에 따라 결정됩니다.  
> 
> 

로컬 관리자 암호를 재설정하려면 `vm reset-access` Azure CLI 명령어를 실행합니다.

```
azure vm reset-access -u <user> -p <new-password> <resource-group> <vm-name>
```

[감사 로그][audit-logs]를 통해 프로비전 동작 및 기타 VM 이벤트를 확인할 수 있습니다.

**데이터 암호화.** OS와 데이터 디스크를 암호화해야 하는 경우에는 [Azure Disk Encryption][disk-encryption] 사용을 고려해 보시기 바랍니다. 

## 솔루션 배포

이 아키텍처는 [GitHub][github-folder]에 배포할 수 있습니다. 여기에는 VNet, NSG, 단일 VM이 포함됩니다. 이 아키텍처의 배포 단계는 다음과 같습니다.

1. 아래 버튼를 마우스 오른쪽 단추로 클릭하여 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fvirtual-machines%2Fsingle-vm%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열리면 일부 설정값을 입력해야 합니다. 
   * **리소스 그룹 ** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-single-vm-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자는 편집하지 않습니다.
   * **Os 유형** 드롭다운 상자에서 **Windows**를 선택합니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 버튼을 클릭합니다.
3. 배포가 완료될 때까지 기다립니다.
4. 매개변수 파일에는 하드 코딩된 관리자 사용자 이름 및 암호가 포함되어 있는데, 이 둘 모두를 즉시 변경할 것을 권장합니다. Azure 포털에서 `ra-single-vm0 `으로 명명된 VM을 클릭합니다. 그런 다음 **지원 + 문제 해결** 블레이드에서 **암호 재설정**을 클릭합니다. **모드** 드롭다운 상자에서 **암호 재설정**을 선택한 후 새 **사용자 이름** 및 **암호**를 선택합니다. **업데이트** 버튼을 클릭하여 새 사용자 이름 및 암호를 보존합니다.

이 아키텍처를 배포하는 다른 방법에 대해서는 [guidance-single-vm][github-folder]] Github 폴더의 추가 정보 파일을 참조하시기 바랍니다. 

요구사항을 충족하기 위해 배포를 변경해야 하는 경우 [readme][github-folder] 페이지에 있는 지침을 따릅니다.  


<!-- links -->

[audit-logs]: https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/
[availability-set]: /azure/virtual-machines/virtual-machines-windows-create-availability-set
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-storage]: /azure/storage/storage-introduction
[blob-snapshot]: /azure/storage/storage-blob-snapshots
[blob-storage]: /azure/storage/storage-introduction
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[cname-record]: https://en.wikipedia.org/wiki/CNAME_record
[data-disk]: /azure/virtual-machines/virtual-machines-windows-about-disks-vhds
[disk-encryption]: /azure/security/azure-security-disk-encryption
[enable-monitoring]: /azure/monitoring-and-diagnostics/insights-how-to-use-diagnostics
[fqdn]: /azure/virtual-machines/virtual-machines-windows-portal-create-fqdn
[github-folder]: http://github.com/mspnp/reference-architectures/tree/master/virtual-machines/single-vm
[group-policy]: https://technet.microsoft.com/en-us/library/dn595129.aspx
[log-collector]: https://azure.microsoft.com/blog/simplifying-virtual-machine-troubleshooting-using-azure-log-collector/
[manage-vm-availability]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[multi-vm]: multi-vm.md
[naming conventions]: ../../best-practices/naming-conventions.md
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-default-rules]: /azure/virtual-network/virtual-networks-nsg#default-rules
[planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[premium-storage]: /azure/storage/storage-premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rbac-roles]: /azure/active-directory/role-based-access-built-in-roles
[rbac-devtest]: /azure/active-directory/role-based-access-built-in-roles#devtest-labs-user
[rbac-network]: /azure/active-directory/role-based-access-built-in-roles#network-contributor
[reboot-logs]: https://azure.microsoft.com/blog/viewing-vm-reboot-logs/
[resize-os-disk]: /azure/virtual-machines/virtual-machines-windows-expand-os-disk
[Resize-VHD]: https://technet.microsoft.com/en-us/library/hh848535.aspx
[Resize virtual machines]: https://azure.microsoft.com/blog/resize-virtual-machines/
[resource-lock]: /azure/resource-group-lock-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[security-center]: https://azure.microsoft.com/services/security-center/
[select-vm-image]: /azure/virtual-machines/virtual-machines-windows-cli-ps-findimage
[services-by-region]: https://azure.microsoft.com/regions/#services
[static-ip]: /azure/virtual-network/virtual-networks-reserved-public-ip
[storage-account-limits]: /azure/azure-subscription-service-limits#storage-limits
[storage-price]: https://azure.microsoft.com/pricing/details/storage/
[Use Security Center]: /azure/security-center/security-center-get-started#use-security-center
[virtual-machine-sizes]: /azure/virtual-machines/virtual-machines-windows-sizes
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-resize]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vm-size-tables]: /azure/virtual-machines/virtual-machines-windows-sizes#size-tables
[0]: ./images/single-vm-diagram.png "Single Windows VM architecture in Azure"
[readme]: https://github.com/mspnp/reference-architectures/blob/master/virtual-machines/single-vm/README.md
