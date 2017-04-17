---
title: Run load-balanced VMs on Azure for scalability and availability
description: >-
  How to run multiple Linux VMs on Azure for scalability and availability.

author: MikeWasson

ms.service: guidance
ms.topic: article
ms.date: 11/22/2016
ms.author: pnp

pnp.series.title: Linux VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
---
# 더 높은 확장성 및 가용성을 위한 부하 분산 VM 실행

이 참조 아키텍처는 가용성과 확장성을 개선하기 위해 부하 분산 장치 뒤에 여러 Linux VM을 실행하기 위한 일련의 검증된 사례를 보여줍니다. 이 아키텍처는 웹 서버를 포함한 모든 상태 비저장 워크로드에 사용 가능한 N계층 응용 프로그램의 배포를 위한 빌딩 블록입니다. [**이 솔루션 배포하기**.](#deploy-the-solution) 

![[0]][0]

## 아키텍처

이 참조 아키텍처는 [Azure에서 Linux VM 실행][single vm]에 소개된 아키텍처를 기반으로 하고 있습니다. 따라서 해당 권장사항이 이 아키텍처에도 동일하게 적용됩니다.

이 아키텍처에서 워크로드는 여러 VM 인스턴스들로 분산됩니다. 단일 공용 IP 주소가 있고, 인터넷 트래픽은 부하 분산 장치를 사용하여 여러 VM에 분산됩니다. 이 아키텍처는 상태 비저장 웹 응용 프로그램이나 저장소 클러스터와 같은 단일 계층 응용 프로그램에 사용할 수 있습니다. 또한 이 아키텍처는 N계층 응용 프로그램을 위한 빌딩 블록입니다. 

이 아키텍처는 다음과 같은 요소들로 구성되어 있습니다. 

* **가용성 집합**. [•	가용성 집합][availability set]은 여러 개의 VM으로 구성됩니다. 이를 통해 해당 VM은 [Azure VM에 대한 가용성 서비스 수준 계약(SLA)][vm-sla] 요건을 충족할 수 있습니다. SLA를 적용하려면 동일한 가용성 집합 내에 최소 두 개의 VM이 필요합니다. 
* **가상 네트워크(VNet)와 서브넷.** Azure 내 모든 VM은 서브넷으로 나뉘게 되는 VNet에 배포됩니다.
* **Azure 부하 분산 장치**. [부하 분산 장치]는 들어오는 인터넷 요청을 VM 인스턴스들로 분산시킵니다. 부하 분산 장치는 일부 관련 리소스들을 포함합니다.
  * **공용 IP 주소**. 부하 분산 장치가 인터넷 트래픽을 수신하기 위해서는 공용 IP 주소가 필요합니다.
  * **프런트 엔드 구성**. 공용 IP 주소를 부하 분산 장치에 연결합니다.
  * **백 엔드 주소 풀**. 들어오는 트래픽을 수신할 VM을 위한 네트워크 인터페이스(NIC)를 포함하고 있습니다.
* **부하 분산 장치 규칙**. 트워크 트래픽을 백 엔드 주소 풀 내 모든 VM에 분산하기 위해 사용합니다. 
* **NAT(Network Address Translation) 규칙**. 트래픽을 특정 VM에 라우팅하기 위해 사용합니다. 예를 들어, VM에 대해 원격 데스크톱 프로토콜(RDP)을 사용하도록 설정하려면 각각의 VM에 대한 개별 NAT 규칙을 생성해야 합니다. 
* **네트워크 인터페이스(NIC)**. 각각의 VM은 네트워크 연결을 위한 NIC를 갖습니다.
* **저장소**. 저장소 계정은 VM 이미지뿐만 아니라 Azure가 수집한 VM 진단 데이터와 같은 다른 파일 관련 리소스도 포함합니다.

You can download a [Visio file](https://aka.ms/arch-diagrams) of this architecture.

> [!참고]
> Azure는 [Resource Manager][resource-manager-overview]와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 문서에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다. 
>

## 권장사항

이 문서에서 설명하는 아키텍처는 귀하의 요구사항과 정확히 일치하지 않을 수 있습니다. 따라서 이 문서의 권장사항은 하나의 출발점으로 삼으시기 바랍니다.  


### 가용성 집합 권장사항

[Azure VM에 대한 가용성 SLA][vm-sla]를 지원하려면 가용성 집합에 두 개 이상의 VM을 생성해야 합니다. 또한 Azure 부하 분산 장치의 경우, 부하 분산 VM이 동일한 가용성 집합에 속해 있어야 합니다.

각 Azure 구독에는 지역당 최대 VM수를 비롯한 기본 한도가 적용되는데, 지원 신청을 통해 해당 한도를 늘릴 수 있습니다. 자세한 내용은 [Azure 구독 및 서비스 한도, 할당량, 제약 조건][subscription-limits]을 참조하시기 바랍니다.  

### 네트워크 권장사항

VM을 동일한 서브넷 내에 배치합니다. VM을 인터넷에 직접 노출하지 않고 각각의 VM에 사설 IP 주소를 부여합니다. 클라이언트는 부하 분산 장치의 공용 IP 주소를 사용하여 접속합니다. 

### 부하 분산 장치 권장사항

가용성 집합 내 모든 VM을 부하 분산 장치의 백 엔드 주소 풀에 추가합니다. 

네트워크 트래픽을 VM으로 보내기 위한 부하 분산 장치 규칙을 정의합니다. 예를 들어, HTTP 트래픽을 사용하도록 설정하려면 프런트 엔드 구성의 포트 80을 백 엔드 주소 풀의 포트 80으로 매핑하는 규칙을 생성합니다. 클라이언트가 HTTP 요청을 포트 80으로 보내면 부하 분산 장치가 소스 IP 주소를 포함하는 [해시 알고리즘][load balancer hashing]을 사용하여 백 엔드 IP 주소를 선택합니다. 이와 같은 방식으로 클라이언트 요청이 모든 VM으로 분산됩니다. 

NAT 규칙을 사용하여 트래픽을 특정 VM으로 라우팅합니다. 예를 들어, VM에 대해 원격 데스크톱 프로토콜(RDP)을 사용하도록 설정하려면 각각의 VM에 대한 개별 NAT 규칙을 생성해야 합니다. 각각의 규칙은 별개의 포트 번호를 RDP의 기본 포트인 포트 3389로 매핑해야 합니다. 예를 들어, VM1에는 포트 50001을, VM2에는 포트 50002를 사용하는 방식입니다. NAT 규칙을 VM의 네트워크 인터페이스(NIC)로 할당합니다. 

### 저장소 계정 권장사항

저장소 계정에 대한 [IOPS 한도][vm-disk-limits]에 도달하지 않도록 각각의 VM별로 가상 하드 디스크(VHD)를 담기 위한 별도의 Azure 저장소 계정을 생성합니다. 

진단 로그에 대한 단일 저장소 계정을 생성합니다. 이 저장소 계정은 모든 VM이 공유할 수 있습니다. 

## 확장성 고려사항

확장하려면 추가 VM을 프로비전한 후 부하 분산 장치의 백 엔드 주소 풀에 넣습니다.  

> [!팁]
> 새 VM을 가용성 집합에 추가하는 경우 반드시 해당 VM을 위한 네트워크 인터페이스(NIC)를 생성한 뒤 이 NIC를 부하 분산 장치의 백 엔드 주소 풀에 추가해야 합니다. 그렇지 않으면 인터넷 트래픽이 새 VM으로 라우팅되지 않습니다. 
> 
> 

### VM Scale Set

또 다른 확장 방법은 [VM Scale Set][vmss]을 이용하는 것입니다. VM Scale Set을 이용하여 동일한 VM들의 집합을 배포하고 관리할 수 있습니다. VM Scale Set은 성능 메트릭에 기초한 자동 크기 조정을 지원합니다. VM의 부하가 증가하면 추가 VM이 부하 분산 장치에 자동으로 추가됩니다. VM을 신속하게 확장해야 하거나 자동 크기 조정이 필요한 경우 VM Scale Set 사용을 고려해 보시기 바랍니다.  

현재 VM Scale Set은 데이터 디스크를 지원하지 않습니다. 데이터 저장을 위해 Azure File 저장소, OS 드라이브, 임시 드라이브 또는 Azure Storage와 같은 외부 저장소를 활용할 수 있습니다. 

VM Scale Set은 "오버 프로비저닝"을 사용하도록 기본 설정되어 있습니다. 즉, VM Scale Set은 요청된 것보다 더 많은 VM을 프로비전한 후 초과된 VM을 삭제합니다. 이를 통해 VM 프로비전 성공률을 높일 수 있습니다. 오버 프로비저닝을 사용하도록 설정한 경우에는 저장소당 20개 이하의 VM을, 오버 프로비저닝을 해제한 경우에는 40개 이하의 VM을 권장합니다. 

VM Scale Set에 배포된 VM의 구성 방법은 크게 두 가지로 나뉩니다.  

- VM을 프로비전한 후 확장 프로그램을 사용하여 VM을 구성합니다. 이 경우 새 VM 인스턴스의 실행 시간은 확장 프로그램을 사용하지 않은 VM보다 더 길어질 수 있습니다.

- 사용자 지정 이미지를 생성합니다. 이 방법을 활용하면 더 빠르게 배포할 수 있습니다. 그러나 이 방법을 사용하려면 해당 이미지를 최신으로 유지해야 합니다. 사용자 지정 이미지를 기반으로 하는 VM Scale Set은 모든 OS 디스크 가상 하드 디스크(VHD)를 단일 저장소 계정 내에 만들어야 합니다. 

자세한 내용은 [크기 조정을 위한 VM Scale Set 설계][vmss-design]를 참조하시기 바랍니다.

> [!팁]
> 자동 크기 조정 솔루션 사용 시 실운영 수준의 워크로드를 적용하여 사전 테스트를 실시합니다.  
> 
> 

## 가용성 고려사항

가용성 집합을 통해 계획되었거나 계획되지 않은 유지관리 이벤트에 대한 응용 프로그램 복원력을 강화할 수 있습니다. 

* *계획된 유지관리*는 Microsoft가 기본 플랫폼을 업데이트하면서 VM이 재시작되는 경우 실행됩니다. Azure는 가용성 집합의 모든 VM이 동시에 재시작되지 않도록 제어합니다. 즉, 다른 VM이 재시작하는 동안에도 최소 한 개의 VM은 계속 실행됩니다.
* *계획되지 않은 유지관리*는 하드웨어 고장 시 실행됩니다. Azure는 가용성 집합의 VM이 둘 이상의 서버 랙에 걸쳐 프로비전되도록 제어합니다. 이를 통해 하드웨어 고장, 네트워크 정지, 정전 등의 영향을 완화할 수 있습니다.

자세한 내용은 [VM 가용성 관리][availability set]를 참조하시기 바랍니다. 또한 다음 동영상을 통해 가용성 집합에 대한 기본 지식을 얻을 수 있습니다. [VM 크기 조정을 위한 가용성 집합 구성 방법][availability set ch9]. 

> [!경고]
> VM 프로비전 시 반드시 가용성 집합을 설정해야 합니다. 현재로서는 VM을 프로비전한 후 Resource Manager VM을 가용성 집합에 추가할 방법이 없기 때문입니다. 
> 
 

부하 분산 장치는 [상태 프로브]를 사용하여 VM 인스턴스의 가용성을 모니터링합니다. 프로브가 타임아웃 전까지 인스턴스에 도달하지 못하면 부하 분산 장치가 해당 VM에 대한 트래픽 전송을 중단합니다. 그러나 부하 분산 장치가 프로빙을 중단하는 것은 아닙니다. 따라서 VM이 이용가능한 상태로 바뀌면 해당 VM에 대한 트래픽 전송을 재개합니다. 

부하 분산 장치 상태 프로브에 대한 권장사항은 다음과 같습니다. 

* 프로브는 HTTP나 TCP를 테스트할 수 있습니다. VM이 HTTP 서버를 운영하는 경우에는 HTTP 프로브를 생성하고 TCP 서버를 운영하는 경우에는 TCP 프로브를 생성합니다.
* HTTP 프로브의 경우, HTTP 끝점까지의 경로를 지정하면 프로브가 이 경로로부터 HTTP 200 응답을 검사합니다. 이 경로는 루트 경로("/"), 즉 응용 프로그램의 상태를 검사하기 위한 사용자 지정 로직을 수행하는 상태 모니터링 끝점일 수 있습니다. 이 끝점은 익명 HTTP 요청을 허용해야 합니다.
* 이 프로브는 [알려진][health-probe-ip] IP 주소인 168.63.129.16으로부터 전송됩니다. 이때 방화벽 정책이나 네트워크 보안 그룹(NSG) 규칙에서 해당 IP에 대한 트래픽을 차단하지 않도록 주의해야 합니다.
* [상태 프로브 로그][health probe log]를 사용하여 상태 프로브의 상태를 확인합니다. Azure 포털에서 각 부하 분산 장치에 대해 로깅을 사용하도록 설정합니다. 로그는 Azure Blob 저장소에 기록됩니다. 이 로그는 프로브 응답 실패로 인해 네트워크 트래픽을 수신하지 못하는 백 엔드의 VM 수를 표시합니다.

## 관리 효율성 고려사항

여러 VM을 사용하는 경우에는 신뢰성과 반복성을 보장하기 위해 프로세스를 자동화하는 것이 중요합니다. [Azure Automation][azure-automation]을 사용하면 배포, OS 패칭 등의 작업을 자동화할 수 있습니다. [Azure Automation][azure-automation]은 Windows Powersell을 기반으로 한 자동화 서비스로, 그 용도는 다음과 같습니다. TechNet의 [Runbook Gallery]에서 자동화 스크립트 예시를 확인할 수 있습니다.+ 

## 보안 고려사항

가상 네트워크는 Azure의 트래픽 격리 경계입니다. 하나의 VNet에 있는 VM은 다른 VNet의 VM과 직접 통신할 수 없습니다. 동일 VNet 내에 있는 VM은 [네트워크 보안 그룹][nsg] (NSG)을 생성해 트래픽을 제한하지 않는 한 서로 통신할 수 있습니다. 자세한 내용은 [Microsoft 클라우드 서비스 및 네트워크 보안][network-security]을 참조하시기 바랍니다. 

들어오는 인터넷 트래픽의 경우 부하 분산 장치 규칙은 백 엔드에 도달할 수 있는 트래픽을 정의합니다. 그러나 부하 분산 장치 규칙은 IP 안전 목록을 지원하지 않습니다. 따라서 특정한 공용 IP 주소를 안전 목록에 추가하려면 NSG를 서브넷에 추가해야 합니다. 

## 솔루션 배포

이 아키텍처는 [GitHub][github-folder]에 배포할 수 있습니다. 여기에는 VNet, NSG, 부하 분산 장치, 두 개의 VM이 포함됩니다. 이 아키텍처는 Windows VM 또는 Linux VM을 사용하여 배포할 수 있습니다. 이 아키텍처의 배포 단계는 다음과 같습니다. 

1. 아래 버튼을 마우스 오른쪽 단추로 클릭하여 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fvirtual-machines%2Fmulti-vm%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열리면 일부 설정값을 입력해야 합니다. 
   * **리소스 그룹** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-multi-vm-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자는 편집하지 않습니다.
   * **OS 유형** 드롭다운 상자에서 **Windows**나 **Linux** 중 하나를 선택합니다. 
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 버튼을 클릭합니다.
3. 배포가 완료될 때까지 기다립니다.
4. 매개변수 파일에는 하드 코딩된 관리자 사용자 이름 및 암호가 포함되어 있는데, 이 둘 모두를 즉시 변경할 것을 권장합니다. Azure 포털에서 `ra-multi-vm1`으로 명명된 VM을 클릭합니다. 그런 다음 **지원 + 문제 해결** 블레이드에서 **암호 재설정**을 클릭합니다. 모드 드롭다운 상자에서 **암호 재설정**을 선택한 후 새 **사용자 이름** 및 **암호**를 선택합니다. **업데이트** 버튼을 클릭하여 새 사용자 이름 및 암호를 저장합니다. `ra-multi-vm2`로 명명된 VM에 대해서도 동일한 과정을 반복합니다.




<!-- Links -->
[n-tier-linux]: n-tier.md
[single vm]: single-vm.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[availability set]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[availability set ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azure-automation]: https://azure.microsoft.com/documentation/services/automation/
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-automation]: /azure/automation/automation-intro
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health probe log]: /azure/load-balancer/load-balancer-monitor-log
[health probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[load balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load balancer hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[Runbook Gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[0]: ./images/multi-vm-diagram.png "Architecture of a multi-VM solution on Azure comprising an availability set with two VMs and a load balancer"
