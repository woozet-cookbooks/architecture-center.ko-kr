---
title: Run Linux VMs for an N-tier application on Azure
description: How to run Linux VMs for an N-tier architecture in Microsoft Azure.

author: MikeWasson

ms.service: guidance
ms.topic: article
ms.date: 11/22/2016
ms.author: pnp

pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
---

# N계층 애플리케이션을 위한 Linux VM 실행

이 참조 아키텍처는 N계층 애플리케이션을 위한 Linux 가상 컴퓨터(VM)를 실행하기 위한 일련의 검증된 사례들을 보여줍니다. [**이 솔루션 배포하기**.](#deploy-the-solution)  

![[0]][0]

## 아키텍처

N계층 아키텍처를 구현하는 방법은 다양합니다. 아래 다이어그램은 전형적인 3계층 웹 애플리케이션을 보여줍니다. 이 아키텍처는 [더 높은 확장성 및 가용성을 위한 부하 분산 VM 실행][multi-vm]을 기반으로 합니다. 웹 및 비즈니스 계층은 부하 분산 VM을 사용합니다.

* **가용성 집합** 계층별로 [가용성 집합][azure-availability-sets]을 만들고 계층당 최소 2개의 VM을 프로비전합니다. 이를 통해 해당 VM들은 더 높은 [서비스 수준 계약(SLA)][vm-sla] 요건을 충족합니다.
* **서브넷** 계층별로 별개의 서브넷을 생성합니다. [CIDR] 표기법을 사용하여 주소 범위와 서브넷 마스크를 지정합니다.  
* **부하 분산 장치** [인터넷 연결 부하 분산 장치][load-balancer-external]를 통해 들어오는 인터넷 트래픽을 웹 계층에 분산시키고 [내부 부하 분산 장치][load-balancer-internal]를 통해 웹 계층의 네트워크 트래픽을 비즈니스 계층에 분산시킵니다.
* **점프박스** [배스천 호스트]로도 알려진 점프박스는 관리자가 다른 VM에 연결하기 위해 사용하는 네트워크 상의 보안 VM입니다. 점프박스는 안전 목록의 공인 IP 주소로부터의 원격 트래픽만을 허용하는 네트워크 보안 그룹(NSG)을 갖습니다. 네트워크 보안 그룹은 보안 셸(SSH) 트래픽을 허용해야 합니다.
* **모니터링.** [Nagios], [Zabbix] 또는 [Icinga]와 같은 모니터링 소프트웨어를 사용하여 응답 시간, VM 가동 시간, 전반적인 시스템 상태에 대한 정보를 파악할 수 있습니다. 모니터링 소프트웨어는 별도의 관리 서브넷에 위치한 VM 상에 설치합니다.
* **네트워크 보안 그룹(NSG).** [네트워크 보안 그룹][nsg] (NSG)을 사용하여 VNet 내 네트워크 트래픽을 제한합니다. 예를 들어, 위의 그림과 같은 3계층 아키텍처에서는 DB 계층이 웹 프런트엔드로부터의 트래픽을 수용하지 않고 오직 비즈니스 계층과 관리 서브넷으로부터의 트래픽만을 수용합니다.
* **Apache Cassandra database**. DB 복제 및 장애조치(failover)를 사용하도록 설정하여 데이터 계층에서 고가용성을 제공합니다.

You can download a [Visio file](https://aka.ms/arch-diagrams) of this architecture.

> [!참고]
> Azure는 [Resource Manager][resource-manager-overview]와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 문서에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다. 
> 
> 

## 추천

이 문서에서 설명하는 아키텍처는 귀하의 요구사항과 정확히 일치하지 않을 수도 있습니다. 따라서 이 문서의 권장사항을 출발점으로 삼으시기 바랍니다. 

### VNet / 서브넷

VNet을 만드는 경우 각 서브넷의 리소스에 필요한 IP 주소의 개수를 확인해야 합니다. [CIDR] 표기법을 사용하여 서브넷 마스크와 해당 IP 주소에 적합한 크기의 VNet 주소 범위를 지정합니다. 표준 [사설 IP 주소 블록][private-ip-space](10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)에 해당하는 주소 공간을 사용합니다.

추후 VNet과 온-프레미스 네트워크 사이에 게이트웨어를 설치해야하는 경우에 대비하여 온-프레미스 네트워크와 겹치지 않는 주소 범위를 선택합니다. VNet을 생성한 후에는 주소 범위를 변경할 수 없습니다.

기능 및 보안 요구사항을 고려하여 서브넷을 설계합니다. 동일한 계층이나 역할 내의 모든 VM은 보안 경계가 될 수 있는 동일한 서브넷에 들어가야 합니다. VNet과 서브넷의 설계에 관한 자세한 내용은 [Azure 가상 네트워크의 계획 및 설계][plan-network]를 참조하세요.

CIDR 표기법을 사용하여 각 서브넷에 대한 주소 공간을 지정합니다. 예를 들어, '10.0.0.0/24'은 256 IP 주소의 범위를 생성합니다. VM은 이중 251개를 사용할 수 있고 나머지 5개는 예약됩니다. 주소 범위가 서브넷끼리 겹치지 않도록 주의합니다. [가상 네트워크 FAQ][vnet faq]를 참조하세요.

### 네트워크 보안 그룹

네트워크 보안 그룹(NSG) 규칙을 사용하여 계층 간 트래픽을 제한할 수 있습니다. 예를 들어, 위의 3계층 아키텍처에서는 웹 계층은 DB 계층과 직접 통신하지 않습니다. 이를 강제로 실행하려면 DB 계층이 웹 계층 서브넷으로부터 오는 트래픽을 차단해야 합니다.  

1. 네트워크 보안 그룹을 만들어 DB 계층 서브넷에 연결시킵니다.
2. VNet으로부터 오는 모든 트래픽을 거부하는 규칙을 추가합니다. (`VIRTUAL_NETWORK` 태그를 사용합니다.) 
3. 비즈니스 계층 서브넷으로부터 오는 트래픽을 허용하는 더 높은 우선순위를 가진 규칙을 추가합니다. 이 규칙은 기존의 규칙에 우선하여 비즈니스 계층이 DB 계층과 통신하도록 허용합니다.
4. DB 계층 서브넷 내부로부터 들어오는 트래픽을 허용하는 규칙을 추가합니다. 이 규칙은 DB 복제 및 장애조치(failover)에 필요한 DB 계층 내 VM 간 통신을 허용합니다.
5. 점프박스 서브넷으로부터 오는 SSH 트래픽을 허용하는 규칙을 추가합니다. 이 규칙을 통해 관리자는 점프박스로부터 DB 계층에 접속할 수 있습니다.
   
   > [!참고]
   > 네트워크 보안 그룹은 VNet 내부로부터 오는 모든 트래픽을 허용하는 [규칙들이 기본으로 설정][nsg-rules]되어 있습니다. 이 규칙들은 삭제할 수 없지만 더 높은 우선순위의 규칙을 만들어 무시할 수 있습니다.
   > 
   > 

### 부하 분산 장치

외부 부하 분산 장치는 인터넷 트래픽을 웹 계층에 분산시킵니다. 이 부하 분산 장치를 위한 공인 IP 주소를 만듭니다. [인터넷 연결 부하 분산 장치 만들기][lb-external-create]를 참조하세요.

내부 부하 분산 장치는 웹 계층에서 오는 네트워크 트래픽을 비즈니스 계층으로 분산시킵니다. 이 부하 분산 장치에 사설 IP 주소를 제공하려면 프런트엔드 IP 구성을 만들어 비즈니스 계층의 서브넷에 연결합니다. [내부 부하 분산 장치 만들기][lb-internal-create]를 참조하세요.

### Cassandra

실운영 용도로 [DataStax Enterprise][datastax]를 권장하며, 이러한 추천은 모든 cassandra 에디션에 적용됩니다. Azure에서 DataStax 실행에 관한 자세한 내용은 [Azure용 DataStax Enterprise 배포 가이드][cassandra-in-azure]를 참조하세요. 

Cassandra 클러스터를 위한 VM을 가용성 집합에 배치하여 Cassandra 복제본이 여러 장애 도메인과 업그레이드 도메인에 분산되도록 합니다. 장애 도메인과 업그레이드 도메인에 대한 자세한 내용은 [가상 컴퓨터 가용성 관리][azure-availability-sets]를 참조하세요.

가용성 집합당 (최대) 세 개의 장애 도메인과 18개의 업그레이드 도메인을 구성합니다. 이를 통해 장애 도메인에 고르게 분산될 수 있는 최대 수의 업그레이드 도메인을 얻을 수 있습니다.  

Rack-aware 모드로 노드를 구성합니다. `cassandra-rackdc.properties` 파일에서 장애 도메인을 랙에 매핑합니다. 

부하 분산 장치를 클러스터 앞에 둘 필요는 없습니다. 클라이언트는 클러스터 내 노드에 직접 연결됩니다. 

### 점프박스

점프박스가 최소한의 성능 요구사항을 만족시킬 수 있도록 Standard A1과 같은 작은 VM 크기를 선택합니다. 

점프박스를 위한 [공인 IP 주소]를 생성합니다. 점프박스는 다른 VM과 동일한 VNet에 배치하되 별도의 관리 서브넷에 배치합니다. 

공개 인터넷에서 애플리케이션 워크로드를 실행하는 VM으로의 SSH 접속은 허용하지 않습니다. 대신 이 VM들로의 모든 SSH 접속은 점프박스를 통해 이루어져야 합니다. 관리자는 우선 점프박스로 로그인한 후 점프박스로부터 다른 VM으로 로그인합니다. 점프박스를 통해 알려진 안전한 IP 주소에 한하여 인터넷에서 오는 SSH 트래픽을 허용할 수 있습니다. 

점프 박스를 안전하게 보호하려면 NSG를 생성하여 점프박스 서브넷에 적용합니다. 안전한 공인 IP 주소 집합으로부터만 SSH 연결을 허용하는 NSG 규칙을 추가합니다. NSG는 서브넷 또는 점프박스 네트워크 인터페이스(NIC)에 연결할 수 있습니다. 그러나 동일한 서브넷에 다른 VM을 추가하더라도 SSH 트래픽이 점프박스에만 허용될 수 있도록 NSG를 네트워크 인터페이스에 연결하는 것을 권장합니다.

다른 서브넷에 대한 네트워크 보안 그룹이 관리 서브넷으로부터 오는 SSH 트래픽을 허용하도록 설정합니다. 

## 가용성 고려사항

각 계층 또는 VM 역할을 별도의 가용성 집합에 배치합니다. 

DB 계층에서는 VM 수가 많다고 무조건 고가용성 DB가 되는 것은 아닙니다. 관계형 DB의 경우 일반적으로 고가용성을 얻기 위해서는 복제와 장애조치(failover)를 사용해야 합니다. 

[Azure SLA for VMs][vm-sla]이 제공하는 것보다 더 높은 가용성이 필요한 경우에는 애플리케이션을 두 지역에 복제하고 장애조치를 위해 Azure Traffic Manager를 사용합니다. 자세한 내용은 [고가용성을 위해 다지역에서 Linux VM 실행][multi-dc]을 참조하세요.  

## 보안 고려사항

공개 인터넷과 Azure 가상 네트워크 사이에 DMZ를 만들기 위한 네트워크 가상 어플라이언스(NVA)를 추가하는 것을 고려해 보세요. 네트워크 가상 어플라이언스는 방화벽, 패킷 검사, 감사, 사용자 지정 라우팅과 같은 네트워크 관련 작업을 수행할 수 있는 가상 어플라이언스를 가리키는 용어입니다. 자세한 내용은 [Azure와 인터넷 사이에 DMZ 구현][dmz]을 참조하세요.

## 확장성 고려사항

부하 분산 장치는 네트워크 트래픽을 웹 계층과 비즈니스 계층으로 분산시킵니다. 새 VM 인스턴스를 추가하여 계층을 확장하세요. 부하에 따라 웹 계층과 비즈니스 계층을 독립적으로 확장할 수도 있습니다. 클라이언트 선호도 유지의 필요성에 따른 문제 발생의 가능성을 낮추려면 웹 계층의 VM은 상태 비저장 상태여야 합니다. 비즈니스 로직을 호스팅하는 VM 또한 상태 비저장 상태여야 합니다. 

## 관리 효율성 고려사항

[Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef], [Puppet][puppet]과 같은 중앙 집중식 관리 도구를 사용하여 전체 시스템 관리를 간소화합니다. 이 도구들은 여러 VM에서 수집한 진단 및 상태 정보를 통합하여 시스템의 전반적인 상태 정보를 제공합니다. 

## 솔루션 배포

이 아키텍처는 [GitHub][github-folder]를 통해 배포할 수 있습니다. 이 아키텍처는 세 단계를 거쳐 배포됩니다. 이 아키텍처를 배포하려면 아래의 단계들을 수행하십시오.  

1. 아래 단추를 우클릭하여 "새 탭에서 열기" 또는 "새 창에서 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fvirtual-machines%2Fn-tier-linux%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열리면 아래의 설정값을 입력합니다. 
   * **리소스 그룹** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-ntier-cassandra-rg`를 입력하세요.
   * **위치** 드롭다운 상자에서 지역을 선택하세요.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용약관을 검토한 후 **위에 명시된 사용약관에 동의함** 확인란을 클릭합니다.
   * **구입** 단추를 클릭합니다.
3. Azure 포털 알림에서 배포 완료 메시지를 확인합니다.
4. 매개변수 파일에는 하드 코딩된 관리자 사용자 이름 및 암호가 포함되며 모든 VM에 대해 둘 다 즉시 변경하는 것이 좋습니다. Azure 포털에서 각 VM을 클릭한 후 **지원 + 문제해결** 블레이드에서 **암호 재설정**을 클릭합니다. 모드 드롭다운 상자에서 **암호 재설정**을 선택한 후 새 **사용자 이름** 및 **암호**를 선택합니다. **업데이트** 단추를 클릭하여 새 사용자 이름 및 암호를 보존합니다.

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions

[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[bastion host]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-consistency-usage]: http://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[public IP address]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-diagram.png "N-tier architecture using Microsoft Azure"

