---
title: Apache Cassandra를 통한 N 계층 응용 프로그램
description: Microsoft Azure에서 N 계층 아키텍처에 대한 Linux VM 실행 방법
author: MikeWasson
ms.date: 05/03/2018
ms.openlocfilehash: 7ee14088a2fae3cfc5c1119daf717236c75ecc6a
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142236"
---
# <a name="n-tier-application-with-apache-cassandra"></a>Apache Cassandra를 통한 N 계층 응용 프로그램

이 참조 아키텍처에서는 데이터 계층에 대해 Linux에서 Apache Cassandra를 사용하여 N 계층 응용 프로그램을 위해 구성되는 VM 및 가상 네트워크를 배포하는 방법을 보여 줍니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution) 

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>아키텍처 

이 아키텍처의 구성 요소는 다음과 같습니다.

* **리소스 그룹.** [리소스 그룹][resource-manager-overview]은 리소스를 수명, 소유자를 비롯한 기준으로 관리할 수 있도록 리소스를 그룹화하는 데 사용됩니다.

* **VNet(가상 네트워크) 및 서브넷.** 모든 Azure VM은 VNet에 배포되어 여러 서브넷으로 분할될 수 있습니다. 각 계층에 대해 별도의 서브넷을 만듭니다. 

* **NSG.** [NSG(네트워크 보안 그룹)][nsg]을 사용하여 VNet 내 네트워크 트래픽을 제한합니다. 예를 들어 여기에 표시된 3계층 아키텍처에서 데이터베이스 계층은 비즈니스 계층 및 관리 서브넷뿐 아니라 웹 프론트 엔드의 트래픽을 허용하지 않습니다.

* **가상 머신**. VM 구성 권장 사항은 [Azure에서 Windows VM 실행](./windows-vm.md) 및 [Azure에서 Linux VM 실행](./linux-vm.md)을 참조하세요.

* **가용성 집합.** 각 계층에 대해 [가용성 집합][azure-availability-sets]을 만들고 각 계층에서 두 개 이상의 VM을 프로비전합니다. 이렇게 하면 VM이 더 높은 [SLA(서비스 수준 약정)][vm-sla]를 충족할 수 있습니다. 

* **VM 확장 집합**(표시되지 않음). 가용성 집합을 사용하는 대신 [VM 확장 집합][vmss]을 사용할 수 있습니다. 확장 집합을 사용하면 미리 정의된 규칙에 따라 계층에서 VM을 수동 또는 자동으로 쉽게 확장할 수 있습니다.

* **Azure 부하 분산 장치.** [부하 분산 장치][load-balancer]는 들어오는 인터넷 요청을 VM 인스턴스로 분산합니다. [공용 부하 분산 장치][load-balancer-external]를 사용하여 들어오는 인터넷 트래픽을 웹 계층에 분산하고, [내부 부하 분산 장치][load-balancer-internal]를 사용하여 네트워크 트래픽을 웹 계층에서 비즈니스 계층으로 분산합니다.

* **공용 IP 주소**. 공용 IP 주소는 공용 부하 분산 장치에서 인터넷 트래픽을 받는 데 필요합니다.

* **Jumpbox.** [요새 호스트]라고도 합니다. 관리자가 다른 VM에 연결할 때 사용하는 네트워크의 보안 VM입니다. Jumpbox는 안전 목록에 있는 공용 IP 주소의 원격 트래픽만 허용하는 NSG를 사용합니다. NSG에서 ssh 트래픽을 허용해야 합니다.

* **Apache Cassandra 데이터베이스**. 복제 및 장애 조치(failover)를 사용하여 데이터 계층에서 높은 가용성을 제공합니다.

* **Azure DNS**. [Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다. Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.

## <a name="recommendations"></a>권장 사항

개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다. 여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요. 

### <a name="vnet--subnets"></a>VNet/서브넷

VNet을 만들 때는 각 서브넷에 포함된 리소스에 몇 개의 IP 주소가 필요한지 결정해야 합니다. [CIDR] 표기법을 사용하여 필요한 IP 주소를 충족하는 서브넷 마스크와 VNet 주소 범위를 지정합니다. 표준 [사설 IP 주소 블록][private-ip-space](10.0.0.0/8, 172.16.0.0/12 및 192.168.0.0/16)에 해당하는 주소 공간을 사용합니다.

추후 VNet과 온-프레미스 네트워크 사이에 게이트웨이를 설정해야 할 경우에 대비하여 온-프레미스 네트워크와 중복되지 않는 주소 범위를 선택합니다. VNet을 만든 뒤에는 주소 범위를 변경할 수 없습니다.

기능 및 보안 요구 사항을 염두에 두고 서브넷을 구성합니다. 동일한 계층이나 역할에 속한 모든 VM은 동일한 서브넷에 속해야 합니다. 이때 서브넷은 보안 경계가 될 수 있습니다. VNet 및 서브넷 디자인에 대한 자세한 내용은 [Azure Virtual Networks 계획 및 디자인][plan-network]을 참조하세요.

### <a name="load-balancers"></a>부하 분산 장치

VM을 인터넷에 직접 노출시키는 대신 각 VM에 사설 IP 주소를 부여합니다. 클라이언트에서 공용 부하 분산 장치의 IP 주소를 사용하여 연결합니다.

네트워크 트래픽이 VM으로 전달되도록 부하 분산 장치 규칙을 정의합니다. 예를 들어 HTTP 트래픽을 허용하려면 프론트 엔드 구성의 포트 80을 백엔드 주소 풀의 포트 80으로 매핑하는 규칙을 만듭니다. 클라이언트가 포트 80으로 HTTP 요청을 전송하면 부하 분산 장치가 소스 IP 주소를 포함하는 [해싱 알고리즘][load-balancer-hashing]을 사용하여 백엔드 IP 주소를 선택합니다. 클라이언트 요청은 이런 식으로 모든 VM에 걸쳐 분산됩니다.

### <a name="network-security-groups"></a>네트워크 보안 그룹

NSG 규칙을 사용하여 계층 사이의 트래픽을 제한합니다. 예를 들어 위에 표시된 3 계층 아키텍처에서 웹 계층은 데이터베이스 계층과 직접 통신하지 않습니다. 이를 위해서는 데이터베이스 계층에서 웹 계층 서브넷으로부터 수신되는 트래픽을 차단해야 합니다.  

1. VNet의 모든 인바운드 트래픽을 거부합니다. (규칙에 `VIRTUAL_NETWORK` 태그를 사용합니다.) 
2. 비즈니스 계층 서브넷의 인바운드 트래픽을 허용합니다.  
3. 데이터베이스 계층 서브넷 자체의 인바운드 트래픽을 허용합니다. 이 규칙은 데이터베이스 복제와 장애 조치에 필요한 데이터베이스 VM 간 통신을 허용합니다.
4. jumpbox 서브넷에서 ssh 트래픽(22 포트)을 허용합니다. 관리자는 이 규칙을 사용하여 jumpbox에서 데이터베이스 계층에 연결할 수 있습니다.

첫 번째 규칙보다 우선 순위가 높은 2&ndash;4 규칙을 만들어 재정의합니다.

### <a name="cassandra"></a>Cassandra

프로덕션 사용의 경우 [DataStax Enterprise][datastax]를 권장하나, 이러한 권장 사항은 모든 Cassandra 버전에 적용됩니다. Azure에서 DataStax 실행에 관한 자세한 내용은 [Azure용 DataStax Enterprise 배포 가이드][cassandra-in-azure]를 참조하세요. 

Cassandra 클러스터에 대한 VM을 가용성 집합에 배치하여 Cassandra 복제본이 여러 오류 도메인 및 업그레이드 도메인에 분산되도록 합니다. 장애 도메인 및 업그레이드 도메인에 대한 자세한 내용은 [Virtual Machines의 가용성 관리][azure-availability-sets]를 참조하세요. 

가용성 집합당 오류 도메인(최대) 및 가용성 집합당 업그레이드 도메인 18개를 구성합니다. 그러면 오류 도메인에 균등하게 분산될 수 있는 업그레이드 도메인의 최대 수가 제공됩니다.   

노드를 랙 인식 모드로 구성합니다. 오류 도메인을 `cassandra-rackdc.properties` 파일의 랙에 매핑합니다.

클러스터 앞에 부하 분산 장치가 필요하지 않습니다. 클라이언트는 클러스터의 노드에 직접 연결합니다.

고가용성을 위해 하나 이상의 Azure 지역에 Cassandra를 배포합니다. 각 지역 내의 노드는 지역 내의 복원력을 위해 장애 및 업그레이드 도메인을 사용하여 랙 인식 모드로 구성됩니다.


### <a name="jumpbox"></a>Jumpbox

공용 인터넷에서 응용 프로그램 워크로드를 실행하는 VM으로의 ssh 액세스를 허용하지 않습니다. 대신, 이러한 VM에 대한 모든 ssh 액세스는 jumpbox를 통해 이루어져야 합니다. 관리자는 jumpbox에 로그인한 뒤에 jumpbox에서 다른 VM에 로그인하게 됩니다. jumpbox는 인터넷으로부터의 ssh 트래픽을 허용하지만, 알려진 안전한 IP 주소만 허용됩니다.

jumpbox에는 최소 성능 요구 사항이 있으므로 작은 VM 크기를 선택합니다. jumpbox에 대한 [공용 IP 주소]를 만듭니다. jumpbox를 다른 VM과 동일한 VNet 안에 배치하되 별도의 관리 서브넷에 배치합니다.

jumpbox를 보호하려면 안전한 공용 IP 주소 집합의 ssh 연결만 허용하는 NSG 규칙을 추가합니다. 관리 서브넷에서 ssh 트래픽을 허용하도록 다른 서브넷에 대한 NSG를 구성합니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

[VM 확장 집합][vmss]을 사용하면 동일한 VM 집합을 배포 및 관리할 수 있습니다. 확장 집합은 성능 메트릭을 바탕으로 자동 확장을 지원합니다. VM들의 부하가 늘어나면 부하 분산 장치에 VM이 자동으로 추가됩니다. VM을 신속하게 확장해야 하거나 자동 확장이 필요한 경우 확장 집합을 사용합니다.

확장 집합에 배포된 VM을 구성하는 방법에는 두 가지가 있습니다.

- VM이 프로비전된 뒤에 확장명을 사용하여 VM을 구성합니다. 이렇게 하면 확장명이 없는 VM보다 새 VM 인스턴스를 시작하는 데 시간이 더 오래 걸릴 수 있습니다.

- 사용자 지정 디스크 이미지를 사용하여 [관리 디스크](/azure/storage/storage-managed-disks-overview)를 배포합니다. 이 옵션을 사용하면 배포 시간이 단축될 수 있습니다. 단, 이 방법의 경우 사용자가 이미지를 최신 상태로 유지해야 합니다.

추가 고려 사항은 [확장 집합의 설계 고려 사항][vmss-design]을 참조하세요.

> [!TIP]
> 자동 확장 솔루션을 사용할 때는 사전에 미리 프로덕션 수준 워크로드로 테스트해야 합니다.

각 Azure 구독에는 지역당 최대 VM 개수를 비롯해 기본적인 제한이 적용됩니다. 지원 요청을 제출하여 제한을 늘릴 수 있습니다. 자세한 내용은 [Azure 구독 및 서비스 제한, 할당량 및 제약 조건][subscription-limits]을 참조하세요.

## <a name="availability-considerations"></a>가용성 고려 사항

VM 확장 집합을 사용하지 않는 경우 동일한 계층의 VM을 가용성 집합에 배치합니다. [Azure VM의 가용성 SLA][vm-sla]를 지원하기 위해 가용성 집합에 2개 이상의 VM을 만듭니다. 자세한 내용은 [가상 머신의 가용성 관리][availability-set]를 참조하세요. 

부하 분산 장치는 [상태 프로브][health-probes]를 사용하여 VM 인스턴스의 가용성을 모니터링합니다. 프로브가 특정 시간 안에 인스턴스에 도달하지 못한 경우, 부하 분산 장치가 해당 VM으로 전달되는 트래픽을 차단합니다. 이때도 부하 분산 장치가 프로브를 계속 진행하며, VM을 다시 사용할 수 있게 되면 해당 VM으로 전달되는 트래픽을 재개합니다.

다음은 부하 분산 장치 상태 프로브에 대한 몇 가지 권장 사항입니다.

* 프로브는 HTTP와 TCP를 모두 테스트할 수 있습니다. VM이 HTTP 서버를 실행 중인 경우에는 HTTP 프로브를 만들고, HTTP 서버를 실행하지 않는 경우에는 TCP 프로브를 만듭니다.
* HTTP 프로브를 만들 때는 HTTP 끝점으로 향하는 경로를 지정합니다. 프로브는 이 경로에서 HTTP 200 응답을 확인합니다. 경로는 루트 경로("/")일 수도 있고, 응용 프로그램의 상태를 확인하는 사용자 지정 로직을 구현하는 상태 모니터링 끝점일 수도 있습니다. 끝점은 익명의 HTTP 요청을 허용해야 합니다.
* 프로브는 [알려진 IP 주소][health-probe-ip]인 168.63.129.16에서 전송됩니다. 방화벽 정책이나 NSG(네트워크 보안 그룹) 규칙에서 이 IP 주소로의 트래픽 송수신을 차단하지 않도록 합니다.
* [상태 프로브 로그][health-probe-log]를 사용하여 상태 프로브의 상태를 확인합니다. Azure Portal에서 각 부하 분산 장치에 대해 로깅을 활성화합니다. 로그는 Azure Blob Storage에 쓰기됩니다. 이 로그에서는 백엔드의 VM 중 실패한 프로브 응답으로 인해 네트워크 트래픽을 수신하지 못하는 VM의 개수를 확인할 수 있습니다.

Cassandra 클러스터의 경우 고려할 장애 조치(failover) 시나리오는 응용 프로그램이 사용하는 일관성 수준과 사용한 복제 수에 따라 달라집니다. Cassandra의 일관성 수준 및 사용에 대해서는 [Configuring data consistency][cassandra-consistency](데이터 일관성 구성) 및 [Cassandra: How many nodes are talked to with Quorum?][cassandra-consistency-usage](Cassandra: Quorum과 연결되는 노드 수)을 참조하세요. Cassandra의 데이터 가용성은 응용 프로그램 및 복제 방법에서 사용되는 일관성 수준에 의해 결정됩니다. Cassandra의 복제에 대해서는 [Data Replication in NoSQL Databases Explained][cassandra-replication](NoSQL 데이터베이스의 데이터 복제 설명)를 참조하세요.

## <a name="security-considerations"></a>보안 고려 사항

가상 네트워크는 Azure의 트래픽 격리 경계입니다. 하나의 VNet에 속한 VM은 다른 VNet에 속한 VM과 직접 통신할 수 없습니다. 하나의 VNet에 속한 여러 VM은 사용자가 트래픽을 제한하기 위해 NSG([네트워크 보안 그룹][nsg])를 만들지 않은 이상 서로 통신할 수 있습니다. 자세한 내용은 [Microsoft 클라우드 서비스 및 네트워크 보안][network-security]을 참조하세요.

수신되는 인터넷 트래픽의 경우 부하 분산 장치의 규칙이 어느 트래픽이 백엔드에 도달할 수 있는지 정의합니다. 단, 부하 분산 장치의 규칙은 IP 안전 목록을 지원하지 않으므로 안전 목록에 특정 공용 IP 주소를 추가하려면 서브넷에 NSG를 추가해야 합니다.

NVA(네트워크 가상 어플라이언스)를 추가하여 인터넷과 Azure 가상 네트워크 사이에 DMZ를 만드는 것도 좋은 방법입니다. NVA는 방화벽, 패킷 조사, 감사, 사용자 지정 라우팅과 같은 네트워크 관련 작업을 수행하는 가상 어플라이언스를 통칭하는 용어입니다. 자세한 내용은 [Azure와 인터넷 사이에 DMZ 구현][dmz]을 참조하세요.

중요한 미사용 데이터를 암호화하고 [Azure Key Vault][azure-key-vault]를 사용하여 데이터베이스 암호화 키를 관리합니다. Key Vault는 암호화 키를 HSM(하드웨어 보안 모듈)에 저장합니다. 또한 Key Vault에 데이터베이스 연결 문자열과 같은 응용 프로그램 비밀을 저장하는 것이 좋습니다.

## <a name="deploy-the-solution"></a>솔루션 배포

이 참조 아키텍처에 대한 배포는 [GitHub][github-folder]에서 사용할 수 있습니다. 

### <a name="prerequisites"></a>필수 조건

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution-using-azbb"></a>azbb를 사용하여 솔루션 배포

N 계층 응용 프로그램에 대한 Linux VM 참조 아키텍처를 배포하려면 다음 단계를 따릅니다.

1. 위의 필수 조건 중 1단계에서 복제한 리포지토리가 있는 `virtual-machines\n-tier-linux` 폴더로 이동합니다.

2. 매개 변수 파일은 배포 환경의 각 VM에 대해 관리자 사용자 이름과 암호의 기본값을 지정합니다. 참조 아키텍처를 배포하기 전에 먼저 이 값을 변경해야 합니다. `n-tier-linux.json` 파일을 열고 **adminUsername** 및 **adminPassword** 필드를 새로운 설정으로 변경합니다.   파일을 저장합니다.

3. 아래에 표시된 대로 **azbb** 명령줄 도구를 사용하여 참조 아키텍처를 배포합니다.

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

Azure 구성 요소를 사용하여 이 샘플 참조 아키텍처를 배포하는 방법에 대한 자세한 내용은 [GitHub 리포지토리][git]를 방문하세요.

<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault

[요새 호스트]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2

[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
[git]: https://github.com/mspnp/template-building-blocks
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
[공용 IP 주소]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-cassandra.png "Microsoft Azure를 사용하는 N 계층 아키텍처"

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security