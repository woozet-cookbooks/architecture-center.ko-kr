---
title: "N 계층 아키텍처에 대한 Windows VM 실행"
description: "가용성, 보안, 확장성 및 관리 보안에 주의를 기울이며 Azure에서 다중 계층 아키텍처를 구현하는 방법입니다."
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: e25d10d661ac4759f209bd27384303dee2ee454e
ms.sourcegitcommit: 583e54a1047daa708a9b812caafb646af4d7607b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/28/2017
---
# <a name="run-windows-vms-for-an-n-tier-application"></a>N 계층 응용 프로그램에 대한 Windows VM 실행

이 참조 아키텍처는 N 계층 응용 프로그램에 대해 Windows VM(가상 머신)을 실행하는 데 관해 검증된 일련의 사례를 보여줍니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution) 

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>건축 

n 계층 아키텍처를 구현하는 방법은 여러 가지가 있습니다. 이 다이어그램에서는 일반적인 3계층 웹 응용 프로그램을 보여줍니다. 이 아키텍처는 [부하가 분산된 VM을 실행하여 확장성 및 가용성 확보][multi-vm]를 기반으로 합니다. 웹 및 비즈니스 계층은 부하 분산된 VM을 사용합니다.

* **가용성 집합.** 각 계층에 대해 [가용성 집합][azure-availability-sets]을 만들고 각 계층에서 두 개 이상의 VM을 프로비전합니다. 이렇게 하면 VM이 더 높은 [SLA(서비스 수준 계약)][vm-sla]를 충족할 수 있습니다. 단일 VM을 가용성 집합에서 배포할 수는 있지만, 단일 VM이 모든 OS 및 데이터 디스크에 대한 Azure Premium Storage를 사용하지 않는 한 단일 VM은 SLA를 보장하지 않습니다.  
* **서브넷.** 각 계층에 대해 별도의 서브넷을 만듭니다. [CIDR] 표기법을 사용하여 주소 범위 및 서브넷 마스크를 지정합니다. 
* **부하 분산 장치.** [인터넷 연결 부하 분산 장치][load-balancer-external]를 사용하여 들어오는 인터넷 트래픽을 웹 계층에 배포하고, [내부 부하 분산 장치][load-balancer-internal]를 사용하여 네트워크 트래픽을 웹 계층에서 비즈니스 계층으로 분산합니다.
* **Jumpbox.** [요새 호스트]라고도 합니다. 관리자가 다른 VM에 연결할 때 사용하는 네트워크의 보안 VM입니다. Jumpbox는 안전 목록에 있는 공용 IP 주소의 원격 트래픽만 허용하는 NSG를 사용합니다. NSG는 RDP(원격 데스크톱) 트래픽을 허용해야 합니다.
* **모니터링.** [Nagios], [Zabbix] 또는 [Icinga]와 같은 모니터링 소프트웨어는 응답 시간, VM 가동 시간 및 시스템의 전반적인 상태에 대한 정보를 제공합니다. 개별 관리 서브넷에 배치되어 있는 VM에 모니터링 소프트웨어를 설치합니다.
* **NSG.** [NSG(네트워크 보안 그룹)][nsg]을 사용하여 VNet 내 네트워크 트래픽을 제한합니다. 예를 들어 여기에 표시된 3계층 아키텍처에서 데이터베이스 계층은 비즈니스 계층 및 관리 서브넷뿐 아니라 웹 프론트 엔드의 트래픽을 허용하지 않습니다.
* **SQL Server Always On 가용성 그룹.** 복제 및 장애 조치(failover)를 사용하여 데이터 계층에서 높은 가용성을 제공합니다.
* **AD DS(Active Directory Domain Services) 서버**. Windows Server 2016 전 버전에서는 SQL Server Always On 가용성 그룹이 도메인에 연결되어야 합니다. 가용성 그룹이 WSFC(Windows Server 장애 조치 클러스터) 기술을 사용하기 때문입니다. Windows Server 2016부터는 Active Directory 없이도 장애 조치(failover) 클러스터를 만들 수 있는 기능이 추가되었기 때문에 이 아키텍처에 AD DS 서버가 필요하지 않습니다. 자세한 내용은 [Windows Server 2016 장애 조치(failover) 클러스터링의 새로운 기능][wsfc-whats-new]을 참조하세요.

## <a name="recommendations"></a>권장 사항

개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다. 여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요. 

### <a name="vnet--subnets"></a>VNet/서브넷

VNet을 만들 때는 각 서브넷에 포함된 리소스에 몇 개의 IP 주소가 필요한지 결정해야 합니다. [CIDR] 표기법을 사용하여 필요한 IP 주소를 충족하는 서브넷 마스크와 VNet 주소 범위를 지정합니다. 표준 [사설 IP 주소 블록][private-ip-space](10.0.0.0/8, 172.16.0.0/12 및 192.168.0.0/16)에 해당하는 주소 공간을 사용합니다.

추후 VNet과 온-프레미스 네트워크 사이에 게이트웨이를 설정해야 할 경우에 대비하여 온-프레미스 네트워크와 중복되지 않는 주소 범위를 선택합니다. VNet을 만든 뒤에는 주소 범위를 변경할 수 없습니다.

기능 및 보안 요구 사항을 염두에 두고 서브넷을 구성합니다. 동일한 계층이나 역할에 속한 모든 VM은 동일한 서브넷에 속해야 합니다. 이때 서브넷은 보안 경계가 될 수 있습니다. VNet 및 서브넷 디자인에 대한 자세한 내용은 [Azure 가상 네트워크 계획 및 디자인][plan-network]을 참조하세요.

각 서브넷에 대해 CIDR 표기법을 사용하여 서브넷의 주소 공간을 지정합니다. 예를 들어 '10.0.0.0/24'는 256개의 IP 주소 범위를 만듭니다. VM은 이 중에서 251개를 사용할 수 있습니다. 나머지 5개는 예약되어 있습니다. 각 서브넷의 주소 공간이 겹치지 않아야 합니다. [가상 네트워크 FAQ][vnet faq]를 참조하세요.

### <a name="network-security-groups"></a>네트워크 보안 그룹

NSG 규칙을 사용하여 계층 사이의 트래픽을 제한합니다. 예를 들어 위에 표시된 3계층 아키텍처에서 웹 계층은 데이터베이스 계층과 직접 통신하지 않습니다. 이를 위해서는 데이터베이스 계층에서 웹 계층 서브넷으로부터 수신되는 트래픽을 차단해야 합니다.  

1. NSG를 만든 다음 이를 데이터베이스 계층 서브넷에 연결합니다.
2. VNet으로부터 수신되는 모든 트래픽을 차단하는 규칙을 추가합니다. (규칙에 `VIRTUAL_NETWORK` 태그를 사용합니다.) 
3. 비즈니스 계층 서브넷으로부터 수신되는 모든 트래픽을 허용하는 규칙을 높은 우선 순위로 추가합니다. 이 규칙은 이전 규칙을 재정의하며, 비즈니스 계층이 데이터베이스 계층과 통신할 수 있도록 해 줍니다.
4. 데이터베이스 계층 서브넷 자체로부터 수신되는 트래픽을 허용하는 규칙을 추가합니다. 이 규칙은 데이터베이스 계층에 속한 VM 사이의 통신을 허용합니다. 이것은 데이터베이스 복제와 장애 조치(failover)를 위해 필요합니다.
5. jumpbox 서브넷으로부터 수신되는 RDP 트래픽을 허용하는 규칙을 추가합니다. 이 규칙은 관리자가 jumpbox에서 데이터베이스 계층에 연결할 수 있도록 해 줍니다.
   
   > [!NOTE]
   > NSG는 VNet 내부로부터 수신되는 모든 트래픽을 허용하는 기본 규칙을 갖습니다. 이러한 규칙은 삭제할 수 없지만, 우선 순위가 더 높은 규칙을 만들면 재정의할 수 있습니다.
   > 
   > 

### <a name="load-balancers"></a>부하 분산 장치

외부 부하 분산 장치는 인터넷 트래픽을 웹 계층으로 분산합니다. 부하 분산 장치에 사용할 공용 IP 주소를 만듭니다. [인터넷 연결 부하 분산 장치 만들기][lb-external-create]를 참조하세요.

내부 부하 분산 장치는 웹 계층에서 비즈니스 계층으로 네트워크 트래픽을 분산합니다. 내부 부하 분산 장치에 사설 IP 주소를 부여하기 위해 프론트 엔드 IP 구성을 만든 다음 이를 비즈니스 계층의 서브넷에 연결합니다. [내부 부하 분산 장치 만들기 시작][lb-internal-create]을 참조하세요.

### <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On 가용성 그룹

SQL Server의 고가용성을 위해 [Always On 가용성 그룹][sql-alwayson]을 사용하는 것이 좋습니다. Windows Server 2016 전 버전에서는 Always On 가용성 그룹에 도메인 컨트롤러가 필요하며, 가용성 그룹의 모든 노드가 동일한 AD 도메인에 속해야 합니다.

다른 계층은 [가용성 그룹 수신기][sql-alwayson-listeners]를 통해 데이터베이스에 연결됩니다. 수신기는 SQL 클라이언트가 SQL Server의 물리적 인스턴스의 이름을 알지 못해도 연결할 수 있도록 해 줍니다. 데이터베이스에 액세스하는 VM은 도메인에 연결되어야 합니다. 클라이언트(여기서는 다른 계층)는 DNS를 사용하여 수신기의 가상 네트워크 이름을 IP 주소로 해석합니다.

다음과 같이 SQL Server Always On 가용성 그룹을 구성합니다.

1. WSFC(Windows Server 장애 조치 클러스터링) 클러스터, SQL Server Always On 가용성 그룹과 주 복제본을 만듭니다. 자세한 내용은 [Always On 가용성 그룹 시작][sql-alwayson-getting-started]을 참조하세요. 
2. 고정 사설 IP 주소를 사용하여 내부 부하 분산 장치를 만듭니다.
3. 가용성 그룹 수신기를 만든 다음 수신기의 DNS 이름을 내부 부하 분산 장치의 IP 주소로 매핑합니다. 
4. SQL Server 수신 포트(기본값: TCP 포트 1433)에 대한 부하 분산 장치 규칙을 만듭니다. 부하 분산 장치 규칙은 Direct Server Return이라고도 불리는 *부동 IP*를 지원해야 합니다. 이로 인해 VM은 클라이언트에 직접 응답하여 주 복제본에 대한 직접 연결을 지원하게 됩니다.
  
  > [!NOTE]
  > 부동 IP가 지원된 경우에는 부하 분산 장치 규칙의 프론트 엔드 포트 번호가 백엔드 포트 번호와 같아야 합니다.
  > 
  > 

SQL 클라이언트가 연결을 시도하면 부하 분산 장치가 연결 요청을 주 복제본으로 라우팅합니다. 다른 복제본으로의 장애 조치(failover)가 이루어지면 부하 분산 장치가 이후의 요청을 새로운 주 복제본으로 자동 라우팅합니다. 자세한 내용은 [SQL Server Always On 가용성 그룹에 대한 ILB 수신기 구성][sql-alwayson-ilb]을 참조하세요.

장애 조치(failover)가 진행되는 동안에는 기존 클라이언트 연결이 닫힙니다. 장애 조치(failover)가 완료되면 새로운 연결이 새로운 주 복제본으로 라우팅됩니다.

응용 프로그램에서 쓰기보다 읽기가 훨씬 많이 발생한다면 읽기 전용 쿼리 중 일부를 보조 복제본으로 부하 분산할 수 있습니다. [수신기를 사용하여 읽기 전용 보조 복제본에 연결(읽기 전용 라우팅)][sql-alwayson-read-only-routing]을 참조하세요.

가용성 그룹의 [수동 장애 조치(failover)를 강제로 수행][sql-alwayson-force-failover]하여 배포 환경을 테스트합니다.

### <a name="jumpbox"></a>Jumpbox

jumpbox는 최소한의 성능 요구 사항만을 가지므로 jumpbox에 대해 Standard A1과 같이 크기가 작은 VM을 선택합니다. 

jumpbox에 대한 [공용 IP 주소]를 만듭니다. jumpbox를 다른 VM과 동일한 VNet 안의 별도의 관리 서브넷에 배치합니다.

공용 인터넷으로부터 응용 프로그램 워크로드를 실행하는 VM에 대한 RDP 액세스를 허용하지 않습니다. 대신 이러한 VM에 대한 모든 RDP 액세스는 jumpbox를 통해 이루어져야 합니다. 관리자는 jumpbox에 로그인한 뒤에 jumpbox에서 다른 VM에 로그인하게 됩니다. jumpbox는 인터넷에서 수신되는 RDP 트래픽 중 알려진 안전한 IP 주소만을 허용합니다.

jumpbox를 안전하게 보호하기 위해 NSG을 만든 다음 이를 jumpbox 서브넷에 적용합니다. 안전한 공용 IP 주소 집합으로부터 수신되는 RDP 연결만 허용하는 NSG 규칙을 추가합니다. NSG는 서브넷 또는 jumpbox NIC에 연결할 수 있습니다. 여기서는 동일한 서브넷에 다른 VM을 추가하더라도 RDP 트래픽이 jumpbox에서만 허용되도록 NIC에 연결하는 것이 좋습니다.

관리 서브넷으로부터 수신되는 RDP 트래픽을 허용하도록 다른 서브넷에 대한 NSG를 구성합니다.

## <a name="availability-considerations"></a>가용성 고려 사항

데이터베이스 계층에 여러 개의 VM이 존재한다고 해서 곧바로 고가용성 데이터베이스가 되는 것은 아닙니다. 관계형 데이터베이스의 고가용성을 달성하기 위해서는 일반적으로 복제와 장애 조치(failover)를 사용해야 합니다. SQL Server의 고가용성을 달성하기 위해서는 [Always On 가용성 그룹][sql-alwayson]을 사용하는 것이 좋습니다. 

[VM용 Azure SLA][vm-sla]가 제공하는 가용성보다 높은 가용성이 필요한 경우, 두 지역 간에 응용 프로그램을 복제한 다음 장애 조치(failover)를 위해 Azure Traffic Manager를 사용합니다. 자세한 내용은 [고가용성을 위해 여러 지역에서 Windows VM 실행][multi-dc]을 참조하세요.   

## <a name="security-considerations"></a>보안 고려 사항

중요한 미사용 데이터를 암호화하고 [Azure Key Vault][azure-key-vault]를 사용하여 데이터베이스 암호화 키를 관리합니다. Key Vault는 암호화 키를 HSM(하드웨어 보안 모듈)에 저장합니다. 자세한 내용은 [Azure VM에서 SQL Server에 대한 Azure Key Vault 통합 구성][sql-keyvault]을 참조하세요. 데이터베이스 연결 문자열과 같은 응용 프로그램 비밀 데이터를 Key Vault에 저장하는 것이 좋습니다.

NVA(네트워크 가상 어플라이언스)를 추가하여 인터넷과 Azure 가상 네트워크 사이에 DMZ를 만드는 것도 좋은 방법입니다. NVA는 방화벽, 패킷 조사, 감사, 사용자 지정 라우팅과 같은 네트워크 관련 작업을 수행하는 가상 어플라이언스를 통칭하는 용어입니다. 자세한 내용은 [Azure와 인터넷 사이에 DMZ 구현][dmz]을 참조하세요.

## <a name="scalability-considerations"></a>확장성 고려 사항

부하 분산 장치는 네트워크 트래픽을 웹 계층과 비즈니스 계층으로 분산합니다. 새 VM 인스턴스를 추가하여 수평 확장합니다. 부하의 정도에 따라 웹 계층과 비즈니스 계층을 각각 독립적으로 확장할 수도 있습니다. 클라이언트 선호도로 인해 발생할 수 있는 복잡성을 줄이기 위해서는 웹 계층에 속한 VM이 상태 비저장이어야 합니다. 비즈니스 로직을 호스팅하는 VM도 상태 비저장이어야 합니다.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

[Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef] 또는 [Puppet][puppet]과 같은 일원화된 관리 도구를 사용하여 시스템 전체의 관리를 간소화합니다. 이러한 도구를 사용하면 여러 VM으로부터 캡처된 진단 및 상태 정보를 통합하여 시스템을 종합적으로 파악할 수 있습니다.

## <a name="deploy-the-solution"></a>솔루션 배포

이 참조 아키텍처에 대한 배포는 [GitHub][github-folder]에서 사용할 수 있습니다. 

### <a name="prerequisites"></a>필수 조건

사용자의 구독에 참조 아키텍처를 배포하려면 먼저 다음 단계를 수행해야 합니다.

1. [AzureCAT 참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.

2. Azure CLI 2.0이 컴퓨터에 설치되어 있는지 확인합니다. CLI를 설치하려면 [Install Azure CLI 2.0][azure-cli-2](Azure CLI 2.0 설치)에 제시된 지침을 참조하세요.

3. [Azure 빌딩 블록][azbb] npm 패키지를 설치합니다.

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. 명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 다음 명령 중 하나를 사용하여 Azure 계정에 로그인한 다음 프롬프트에 따릅니다.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>azbb를 사용하여 솔루션 배포

N 계층 응용 프로그램에 대한 Windows VM 참조 아키텍처를 배포하려면 다음 단계를 수행합니다.

1. 위의 필수 조건 단계 중 1단계에서 복제한 리포지토리가 있는 `virtual-machines\n-tier-windows` 폴더로 이동합니다.

2. 매개 변수 파일은 배포 환경의 각 VM에 대해 관리자 사용자 이름과 암호의 기본값을 지정합니다. 참조 아키텍처를 배포하기 전에 먼저 이 값을 변경해야 합니다. `n-tier-windows.json` 파일을 열고 **adminUsername** 필드와 **adminPassword** 필드를 새로운 설정으로 변경합니다.
  
  > [!NOTE]
  > 이 배포가 진행될 때 **VirtualMachineExtension** 개체와 일부 **VirtualMachine** 개체의 **extensions** 설정에서 여러 개의 스크립트가 실행됩니다. 이러한 스크립트 중 일부에서 방금 변경한 관리자 사용자 이름과 암호가 필요합니다. 해당 스크립트를 검토하여 올바른 자격 증명을 지정했는지 확인하는 것이 좋습니다. 올바른 자격 증명을 지정하지 않으면 배포가 실패하게 됩니다.
  > 
  > 

파일을 저장합니다.

3. 아래에 표시된 대로 **azbb** 명령줄 도구를 사용하여 참조 아키텍처를 배포합니다.

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
  ```

Azure 빌딩 블록을 사용하여 이 샘플 참조 아키텍처를 배포하는 자세한 방법을 보려면 [GitHub 리포지토리][git]를 방문하세요.


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[요새 호스트]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[공용 IP 주소]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "Microsoft Azure를 사용하는 N 계층 아키텍처"