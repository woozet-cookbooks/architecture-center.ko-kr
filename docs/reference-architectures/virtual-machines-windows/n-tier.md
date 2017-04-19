---
title: Running Windows VMs for an N-tier architecture
description: >-
  How to implement a multi-tier architecture on Azure, paying particular
  attention to availability, security, scalability, and manageability security.

author: MikeWasson

ms.service: guidance
ms.topic: article
ms.date: 11/22/2016
ms.author: pnp

pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
---

# RN계층 응용 프로그램을 위한 Windows VM 실행

이 참조 아키텍처는 N계층 응용 프로그램을 위한 Windows 가상 컴퓨터(VM)를 실행하기 위한 일련의 검증된 사례들을 보여줍니다. [**이 솔루션 배포하기**.](#deploy-the-solution) 

![[0]][0]

## 아키텍처

N계층 아키텍처를 구현하는 방법은 다양합니다. 아래 다이어그램은 전형적인 3계층 웹 응용 프로그램을 보여줍니다. 이 아키텍처는 [더 높은 확장성 및 가용성을 위한 부하 분산 VM 실행][multi-vm]을 기반으로 합니다. 웹 및 비즈니스 계층은 부하 분산 VM을 사용합니다.

* **가용성 집합.** 계층별로 [가용성 집합][azure-availability-sets]을 생성하고 계층당 최소 2개의 VM을 프로비전합니다. 이를 통해 해당 VM들은 더 높은 [서비스 수준 계약(SLA)][vm-sla] 요건을 충족할 수 있습니다.
* **서브넷.** 계층별로 별개의 서브넷을 생성합니다. [CIDR] 표기법을 사용하여 주소 범위와 서브넷 마스크를 지정합니다. 
* **부하 분산 장치.** [인터넷 연결 부하 분산 장치][load-balancer-external]를 통해 들어오는 인터넷 트래픽을 웹 계층에 분산시키고 [내부 부하 분산 장치][load-balancer-internal]를 통해 웹 계층의 네트워크 트래픽을 비즈니스 계층에 분산시킵니다.
* **점프박스.** [배스천 호스트(bastion host)](https://en.wikipedia.org/wiki/Bastion_host)라고도 불리는 점프박스는 관리자가 다른 VM에 연결하기 위해 사용하는 네트워크 상의 보안 VM입니다. 점프박스는 안전 목록에 포함되어 있는 공용 IP 주소로부터의 원격 트래픽만을 허용하는 네트워크 보안 그룹(NSG)을 보유하는데, 네트워크 보안 그룹은 원격 데스크톱(RDP) 트래픽을 허용해야 합니다.
* **모니터링.** [Nagios], [Zabbix] 또는 [Icinga]와 같은 모니터링 소프트웨어를 사용하여 응답 시간, VM 가동 시간, 전반적인 시스템 상태에 대한 정보를 파악할 수 있습니다.  모니터링 소프트웨어는 별도의 관리 서브넷에 위치한 VM 상에 설치합니다.
* **네트워크 보안 그룹(NSG).** [네트워크 보안 그룹][nsg] (NSG)을 사용하여 VNet 내 네트워크 트래픽을 제한합니다. 예를 들어, 위의 그림과 같은 3계층 아키텍처에서는 DB 계층이 웹 프런트엔드로부터 오는 트래픽은 수용하지 않고 오직 비즈니스 계층과 관리 서브넷으로부터 오는 트래픽만을 수용합니다.
* **SQL 서버 Always On 가용성 그룹.** 복제 및 장애조치를 사용하도록 설정하여 데이터 계층에서 고가용성을 제공합니다.
* **Active Directory 도메인 서비스(AD DS) 서버**. Windows Server 2016에 앞서 SQL 서버 Always On 가용성 그룹을 도메인에 추가해야 합니다. 이는 가용성 그룹이 Windows Server Failover Cluster(WSFC) 기술을 기반으로 하기 때문입니다. Windows Server 2016은 Active Directory 없이 Failover Cluster를 생성할 수 있는 기능을 지원하며 이 경우 이 아키텍처에서는 AD DS 서버가 필요하지 않습니다.  자세한 내용은 [Windows Server 2016의 Failover Clustering의 새로운 특징][wsfc-whats-new]을 참조하시기 바랍니다.

You can download a [Visio file](https://aka.ms/arch-diagrams) of this architecture.

> [!참고]
> Azure는 [Resource Manager][resource-manager-overview]와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 문서에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다. 
> 
 

## 권장사항

이 문서에서 설명하는 아키텍처는 귀하의 요구사항과 정확히 일치하지 않을 수 있습니다. 따라서 이 문서의 권장사항은 하나의 출발점으로 삼으시기 바랍니다. 

### VNet / 서브넷

VNet을 생성하는 경우 각 서브넷의 리소스에 필요한 IP 주소의 개수를 확인해야 합니다. [CIDR] 표기법을 사용하여 서브넷 마스크와 해당 IP 주소에 적합한 크기의 VNet 주소 범위를 지정해야 하는데, 표준 [사설 IP 주소 블록][private-ip-space](10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)에 해당하는 주소 공간을 사용합니다.

추후 VNet과 온-프레미스 네트워크 사이에 게이트웨이를 설치해야 할 경우에 대비하여 온-프레미스 네트워크와 겹치지 않는 주소 범위를 선택합니다. VNet을 생성한 후에는 주소 범위를 변경할 수 없습니다.

기능 및 보안 요구사항을 고려하여 서브넷을 설계합니다. 동일한 계층이나 역할 내의 모든 VM은 보안 경계가 될 수 있는 동일한 서브넷에 들어가야 합니다. VNet과 서브넷의 설계에 관한 자세한 내용은 [Azure 가상 네트워크의 계획 및 설계][plan-network]를 참조하시기 바랍니다. 

CIDR 표기법을 사용하여 각 서브넷에 대한 주소 공간을 지정합니다. 예를 들어, '10.0.0.0/24'은 256개의 IP 주소 범위를 생성하는데, VM은 이중 251개를 사용할 수 있고 나머지 5개는 예약됩니다. 주소 범위가 서브넷끼리 겹치지 않도록 주의합니다. [가상 네트워크 FAQ][vnet faq]를 참조하시기 바랍니다.

### 네트워크 보안 그룹(NSG)

네트워크 보안 그룹(NSG) 규칙을 사용하여 계층 간 트래픽을 제한할 수 있습니다. 예를 들어, 위에 제시된 3계층 아키텍처에서 웹 계층은 DB 계층과 직접 통신하지 않습니다. 이를 강제로 실행하려면 DB 계층이 웹 계층 서브넷으로부터 오는 트래픽을 차단해야 합니다. 

1. 네트워크 보안 그룹을 생성해 DB 계층 서브넷에 연결합니다.
2. VNet으로부터 오는 모든 트래픽을 거부하는 규칙을 추가합니다.  (`VIRTUAL_NETWORK` 태그 사용) 
3. 비즈니스 계층 서브넷으로부터 오는 트래픽을 허용하는 더 높은 우선순위를 가진 규칙을 추가합니다. 기존 규칙에 우선하는 이 규칙 덕분에 비즈니스 계층은 DB 계층과 통신할 수 있습니다.
4. DB 계층 서브넷 내부로부터 들어오는 트래픽을 허용하는 규칙을 추가합니다. 이 규칙은 DB 복제 및 장애조치에 필요한 DB 계층 내 VM 간 통신을 허용합니다.
5. 점프박스 서브넷으로부터 오는 RDP 트래픽을 허용하는 규칙을 추가합니다. 이 규칙을 통해 관리자는 점프박스로부터 DB 계층에 접속할 수 있습니다.
   
   > [!참고]
   > 네트워크 보안 그룹에는 VNet 내부로부터 오는 모든 트래픽을 허용하는 규칙들이 기본 설정되어 있습니다. 이 규칙들을 삭제할 수는 없지만 더 높은 우선순위의 규칙을 만들어 무시할 수 있습니다.
   > 
   > 

### 부하 분산 장치

인터넷 트래픽을 웹 계층에 분산하는 외부 부하 분산 장치를 위한 공용 IP 주소를 생성합니다. [인터넷 연결 부하 분산 장치 생성][lb-external-create]을 참조하시기 바랍니다.

웹 계층에서 오는 네트워크 트래픽을 비즈니스 계층으로 분산하는 내부 부하 분산 장치에 사설 IP 주소를 제공하려면 프런트엔드 IP 구성을 생성해 비즈니스 계층의 서브넷에 연결합니다. [내부 부하 분산 장치 생성][lb-internal-create]을 참조하시기 바랍니다.

### SQL 서버 Always On 가용성 그룹

SQL 서버의 고가용성을 위해 [Always On 가용성 그룹][sql-alwayson]을 사용할 것을 권장합니다.  Windows Server 2016에 앞서 Always On 가용성 그룹에는 도메인 컨트롤러가 필요하고 가용성 그룹 내 모든 노드는 동일한 AD 도메인 내에 있어야 합니다.

다른 계층은 [가용성 그룹 리스너][sql-alwayson-listeners]를 통해 DB에 접속합니다. SQL 클라이언트가 SQL 서버의 물리적 인스턴스의 이름을 모르더라도 리스너를 통해 연결할 수 있습니다. DB에 접속하는 VM은 도메인에 추가되어야 합니다. 클라이언트(여기서는 다른 계층)는 DNS를 사용하여 리스너의 가상 네트워크 이름을 IP 주소로 변환합니다. 

SQL 서버 Always On 가용성 그룹은 다음과 같은 방식으로 구성할 수 있습니다.

1. Windows Server Failover Clustering(WSFC) 클러스터, SQL 서버 Always On 가용성 그룹, 주 복제본을 만듭니다. 자세한 내용은 [Always On 가용성 그룹 시작하기][sql-alwayson-getting-started]를 참조하시기 바랍니다.   
2. 정적 사설 IP 주소를 가진 내부 부하 분산 장치를 만듭니다.
3. 가용성 그룹 리스너를 만들고 리스너의 DNS 이름을 내부 부하 분산 장치의 IP 주소에 매핑합니다. 
4. SQL 서버 리스닝 포트(기본 설정은 TCP 포트 1433)에 대한 부하 분산 장치 규칙을 생성합니다. 부하 분산 장치 규칙은 Direct Server Return이라고도 부르는 *유동 IP*를 활성화해야 하는데, 이를 통해 VM이 클라이언트에 직접 응답할 수 있고 주 복제본에 직접 연결할 수 있기 때문입니다.
  
  > [!참고]
  > 유동 IP를 사용하도록 설정하는 경우, 프런트엔드 포트 번호는 부하 분산 장치 규칙의 백엔드 포트 번호와 동일해야 합니다.
  > 
  > 

SQL 클라이언트가 연결을 시도하면 부하 분산 장치는 연결 요청을 주 복제본으로 라우팅합니다. 다른 복제본에 대한 장애조치가 있는 경우, 부하 분산 장치는 다음 요청을 새로운 주 복제본으로 자동 라우팅합니다. [SQL 서버 Always On 가용성 그룹에 대한 ILB 리스너 구성][sql-alwayson-ilb]을 참조하시기 바랍니다.

장애조치 중에는 기존 클라이언트 연결이 종료됩니다. 장애조치가 완료된 이후 새로 생성된 연결은 장애조치 시 사용된 새로운 주 복제본으로 라우팅됩니다. 

응용 프로그램이 쓰기보다 읽기를 훨씬 더 많이 실행하는 경우에는 일부 읽기 전용 쿼리를 부 복제본으로 오프로드할 수 있습니다. [리스너를 사용하여 읽기 전용 부 복제본에 연결(읽기 전용 라우팅)][sql-alwayson-read-only-routing]을 참조하시기 바랍니다.

가용성 그룹의 [수동 장애조치 강제 실행][sql-alwayson-force-failover]을 통해 배포된 리소스를 테스트합니다.

### 점프박스

점프박스가 최소한의 성능 요구사항을 만족시킬 수 있도록 Standard A1과 같은 작은 VM 크기를 선택합니다. 

점프박스를 위한 [공용 IP 주소]를 생성합니다. 점프박스는 다른 VM과 동일한 VNet에 배치하되 별도의 관리 서브넷에 배치합니다. 

공개 인터넷에서 응용 프로그램 워크로드를 실행하는 VM으로의 원격 데스크톱 액세스는 허용하지 않습니다. 대신 이 VM들로의 모든 원격 데스크톱 액세스는 점프박스를 통해 이루어져야 합니다. 관리자는 우선 점프박스로 로그인한 후 점프박스로부터 다른 VM으로 로그인합니다. 점프박스를 통해 알려진 안전한 IP 주소에 한해 인터넷에서 오는 원격 데스크톱 트래픽을 허용할 수 있습니다. 

점프 박스를 안전하게 보호하려면 NSG를 생성하여 점프박스 서브넷에 적용합니다. 안전한 공용 IP 주소 집합으로부터만 원격 데스크톱 연결을 허용하는 NSG 규칙을 추가합니다. NSG는 서브넷 또는 점프박스 네트워크 인터페이스(NIC)에 연결할 수 있습니다. 그러나 동일한 서브넷에 다른 VM을 추가하더라도 원격 데스크톱 트래픽이 점프박스에만 허용될 수 있도록 NSG를 네트워크 인터페이스에 연결할 것을 권장합니다.

다른 서브넷에 대한 네트워크 보안 그룹이 관리 서브넷으로부터 오는 원격 데스크톱 트래픽을 허용하도록 구성합니다. 

## 가용성 고려사항

DB 계층에서는 VM 수가 많다고 무조건 고가용성 DB가 되지 않습니다. 관계형 DB의 경우 고가용성을 얻기 위해서는 일반적으로 복제와 장애조치를 사용해야 합니다. SQL 서버의 경우에는 [Always On 가용성 그룹][sql-alwayson]의 사용을 권장합니다. 

[Azure SLA for VMs][vm-sla]이 제공하는 것보다 더 높은 가용성이 필요한 경우에는 응용 프로그램을 두 지역에 복제하고 장애조치를 위해 Azure Traffic Manager를 사용합니다. 자세한 내용은 [고가용성을 위해 다지역에서 Windows VM 실행][multi-dc]을 참조하시기 바랍니다.   

## 보안 고려사항

민감한 데이터를 암호화하고 [Azure Key Vault][azure-key-vault]를 사용하여 DB 암호화 키를 관리합니다. Key Vault는 하드웨어 보안 모듈(HSM)에 암호화 키를 저장할 수 있습니다. 자세한 내용은 [Azure VM SQL 서버를 위한 Azure Key Vault 통합 구성][sql-keyvault]을 참조하시기 바랍니다. DB 연결 문자열과 같은 응용 프로그램 비밀 정보는 Key Vault에 저장할 것을 권장합니다. 

네트워크 가상 어플라이언스(NVA)를 추가하여 인터넷과 Azure 가상 네트워크 사이에 DMZ를 생성하는 것을 고려해야 합니다. 네트워크 가상 어플라이언스는 방화벽, 패킷 검사, 감사, 사용자 지정 라우팅과 같은 네트워크 관련 작업을 수행할 수 있는 가상 어플라이언스를 가리키는 용어입니다. 자세한 내용은 [Azure와 인터넷 사이에 DMZ 구현][dmz]을 참조하시기 바랍니다.

## 확장성 고려사항

부하 분산 장치는 네트워크 트래픽을 웹 계층과 비즈니스 계층으로 분산시킵니다. 새 VM 인스턴스를 추가하여 계층을 확장할 수 있는데, 부하에 따라 웹 계층과 비즈니스 계층을 독립적으로 확장할 수도 있습니다. 클라이언트 선호도 유지 필요성에 따른 문제 발생의 가능성을 낮추려면 웹 계층의 VM은 상태 비저장 상태여야 합니다. 비즈니스 로직을 호스팅하는 VM 또한 상태 비저장 상태여야 합니다. 

## 관리 효율성 고려사항

[Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef], [Puppet][puppet]과 같은 중앙 집중식 관리 도구를 사용하여 전체 시스템 관리를 간소화합니다. 이 도구들은 여러 VM에서 수집한 진단 및 상태 정보를 통합하여 시스템의 전반적인 상태 정보를 제공합니다. 

## 솔루션 배포

이 아키텍처는 [GitHub][github-folder]를 통해 배포할 수 있습니다. 이 아키텍처는 세 단계를 거쳐 배포되는데, 이 아키텍처를 배포하려면 아래의 단계들을 수행하십시오.  

1. 아래 버튼을 마우스 오른쪽 단추로 클릭한 후 "새 탭에서 열기" 또는 "새 창에서 열기"를 선택하여 배포 1단계를 시작합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fvirtual-machines%2Fn-tier-windows%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열리면 아래의 설정값을 입력합니다. 
   * **리소스 그룹** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-ntier-sql-network-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자는 편집하지 않습니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 버튼을 클릭합니다.
3. Azure 포털 알림에서 1단계 배포 완료 메시지를 확인합니다.
4. 아래 버튼을 마우스 오른쪽 단추로 클릭한 후 "새 탭에서 열기" 또는 "새 창에서 열기"를 선택하여 2단계 배포를 시작합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fvirtual-machines%2Fn-tier-windows%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Azure 포털에서 링크가 열리면 아래의 설정값을 입력합니다. 
   * **리소스 그룹** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-ntier-sql-workload-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자는 편집하지 않습니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   
   * **구입** 버튼을 클릭합니다.
6. Azure 포털 알림에서 2단계 배포 완료 메시지를 확인합니다.
7. 아래 버튼을 마우스 오른쪽 단추로 클릭한 후 "새 탭에서 열기" 또는 "새 창에서 열기"를 선택하여 3단계 배포를 시작합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fvirtual-machines%2Fn-tier-windows%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. Azure 포털에서 링크가 열리면 아래의 설정값을 입력합니다. 
   * **리소스 그룹** 이름이 매개변수 파일에 이미 정의되어 있으므로 **기존 사용**을 선택한 다음 텍스트 상자에 `ra-ntier-sql-network-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자는 편집하지 않습니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 버튼을 클릭합니다.
9. Azure 포털 알림에서 3단계 배포 완료 메시지를 확인합니다.
10. 매개변수 파일에는 하드 코딩된 관리자 사용자 이름 및 암호가 포함되어 있는데, 이 둘 모두를 모든 VM에 대해 즉시 변경하는 것이 좋습니다. Azure 포털에서 각 VM을 클릭한 후 **지원 + 문제해결** 블레이드에서 **암호 재설정**을 클릭합니다. **모드** 드롭다운 상자에서 **암호 재설정**을 선택한 후 새 **사용자 이름** 및 **암호**를 선택합니다. **업데이트** 버튼을 클릭하여 새 사용자 이름 및 암호를 저장합니다. 


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md

[naming conventions]: /azure/guidance/guidance-naming-conventions
[arm-templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
[azure-administration]: /azure/automation/automation-intro
[azure-audit-logs]: /azure/resource-group-audit
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[azure-load-balancer]: /azure/load-balancer/load-balancer-overview
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[public IP address]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-planned-maintenance]: /azure/virtual-machines/virtual-machines-windows-planned-maintenance
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[VM-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx
[0]: ./images/n-tier-diagram.png "N-tier architecture using Microsoft Azure"
