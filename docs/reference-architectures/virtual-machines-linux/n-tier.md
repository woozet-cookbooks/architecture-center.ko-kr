---
title: Azure에서 N 계층 응용 프로그램에 대해 Linux VM 실행
description: Microsoft Azure에서 N 계층 아키텍처에 대한 Linux VM 실행 방법
author: MikeWasson
ms.date: 11/22/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 8d3e6e5124a0abb27a3c72e1ecbd52a1a1da2a33
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="run-linux-vms-for-an-n-tier-application"></a>N 계층 응용 프로그램에 대해 Linux VM 실행

이 참조 아키텍처는 N 계층 응용 프로그램에 대해 Linux VM(가상 머신)을 실행하는 데 관해 검증된 일련의 사례를 보여 줍니다 [**이 솔루션을 배포합니다**.](#deploy-the-solution)  

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>건축

N 계층 아키텍처를 구현하는 방법은 여러 가지가 있습니다. 이 다이어그램에서는 일반적인 3계층 웹 응용 프로그램을 보여줍니다. 이 아키텍처는 [부하가 분산된 VM을 실행하여 확장성 및 가용성 확보][multi-vm]를 기반으로 합니다. 웹 및 비즈니스 계층은 부하 분산된 VM을 사용합니다.

* **가용성 집합.** 각 계층에 대해 [가용성 집합][azure-availability-sets]을 만들고 각 계층에서 두 개 이상의 VM을 프로비전합니다.  이렇게 하면 VM이 더 높은 [SLA(서비스 수준 약정)][vm-sla]를 충족할 수 있습니다. 단일 VM을 가용성 집합에서 배포할 수는 있지만, 단일 VM이 모든 OS 및 데이터 디스크에 대한 Azure Premium Storage를 사용하지 않는 한 단일 VM은 SLA를 보장하지 않습니다.  
* **서브넷.** 각 계층에 대해 별도의 서브넷을 만듭니다. [CIDR] 표기법을 사용하여 주소 범위 및 서브넷 마스크를 지정합니다. 
* **부하 분산 장치.** [인터넷 연결 부하 분산 장치][load-balancer-external]를 사용하여 들어오는 인터넷 트래픽을 웹 계층에 분산하고, [내부 부하 분산 장치][load-balancer-internal]를 사용하여 네트워크 트래픽을 웹 계층에서 비즈니스 계층으로 분산합니다.
* **Azure DNS**. [Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다. Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.
* **Jumpbox.** [요새 호스트]라고도 합니다. 관리자가 다른 VM에 연결할 때 사용하는 네트워크의 보안 VM입니다. Jumpbox는 안전 목록에 있는 공용 IP 주소의 원격 트래픽만 허용하는 NSG를 사용합니다. NSG에서 SSH(보안 셸) 트래픽을 허용해야 합니다.
* **모니터링.** [Nagios], [Zabbix] 또는 [Icinga]와 같은 모니터링 소프트웨어는 응답 시간, VM 가동 시간 및 시스템의 전반적인 상태에 대한 정보를 제공합니다. 개별 관리 서브넷에 배치되어 있는 VM에 모니터링 소프트웨어를 설치합니다.
* <strong>NSG.</strong> [NSG(네트워크 보안 그룹)][nsg]을 사용하여 VNet 내 네트워크 트래픽을 제한합니다. 예를 들어 여기에 표시된 3 계층 아키텍처에서 데이터베이스 계층은 비즈니스 계층 및 관리 서브넷뿐 아니라 웹 프런트 엔드의 트래픽을 허용하지 않습니다.
* **Apache Cassandra 데이터베이스**. 복제 및 장애 조치(failover)를 사용하여 데이터 계층에서 높은 가용성을 제공합니다.

## <a name="recommendations"></a>권장 사항

개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다. 여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요. 

### <a name="vnet--subnets"></a>VNet/서브넷

VNet을 만들 때 각 서브넷에 포함된 리소스에 몇 개의 IP 주소가 필요한지 결정합니다. [CIDR] 표기법을 사용하여 필요한 IP 주소를 충족하는 서브넷 마스크와 VNet 주소 범위를 지정합니다. 표준 [사설 IP 주소 블록][private-ip-space](10.0.0.0/8, 172.16.0.0/12 및 192.168.0.0/16)에 해당하는 주소 공간을 사용합니다.

추후 VNet과 온-프레미스 네트워크 사이에 게이트웨이를 설정해야 할 경우에 대비하여 온-프레미스 네트워크와 중복되지 않는 주소 범위를 선택합니다. VNet을 만든 뒤에는 주소 범위를 변경할 수 없습니다.

기능 및 보안 요구 사항을 염두에 두고 서브넷을 구성합니다. 동일한 계층이나 역할에 속한 모든 VM은 동일한 서브넷에 속해야 합니다. 이때 서브넷은 보안 경계가 될 수 있습니다. VNet 및 서브넷 디자인에 대한 자세한 내용은 [Azure Virtual Networks 계획 및 디자인][plan-network]을 참조하세요.

각 서브넷에 대해 CIDR 표기법을 사용하여 서브넷의 주소 공간을 지정합니다. 예를 들어 ‘10.0.0.0/24’는 256개의 IP 주소 범위를 만듭니다. VM은 이 중에서 251개를 사용할 수 있습니다. 나머지 5개는 예약되어 있습니다. 각 서브넷의 주소 공간이 겹치지 않아야 합니다. [Virtual Network FAQ][vnet faq]를 참조하세요.

### <a name="network-security-groups"></a>네트워크 보안 그룹

NSG 규칙을 사용하여 계층 사이의 트래픽을 제한합니다. 예를 들어 위에 표시된 3 계층 아키텍처에서 웹 계층은 데이터베이스 계층과 직접 통신하지 않습니다. 이를 위해서는 데이터베이스 계층에서 웹 계층 서브넷으로부터 수신되는 트래픽을 차단해야 합니다.  

1. NSG를 만든 다음 이를 데이터베이스 계층 서브넷에 연결합니다.
2. VNet으로부터 수신되는 모든 트래픽을 차단하는 규칙을 추가합니다. (규칙에 `VIRTUAL_NETWORK` 태그를 사용합니다.) 
3. 비즈니스 계층 서브넷으로부터 수신되는 모든 트래픽을 허용하는 규칙을 높은 우선 순위로 추가합니다. 이 규칙은 이전 규칙을 재정의하며, 비즈니스 계층이 데이터베이스 계층과 통신할 수 있도록 해 줍니다.
4. 데이터베이스 계층 서브넷 자체로부터 수신되는 트래픽을 허용하는 규칙을 추가합니다. 이 규칙은 데이터베이스 계층에 속한 VM 사이의 통신을 허용합니다. 이것은 데이터베이스 복제와 장애 조치(failover)를 위해 필요합니다.
5. jumpbox 서브넷으로부터 수신되는 SSH 트래픽을 허용하는 규칙을 추가합니다. 관리자는 이 규칙을 사용하여 jumpbox에서 데이터베이스 계층에 연결할 수 있습니다.
   
   > [!NOTE]
   > NSG는 VNet 내부로부터 수신되는 모든 트래픽을 허용하는 [기본 규칙][nsg-rules]을 갖습니다. 이러한 규칙은 삭제할 수 없지만, 우선 순위가 더 높은 규칙을 만들면 재정의할 수 있습니다.
   > 
   > 

### <a name="load-balancers"></a>부하 분산 장치

외부 부하 분산 장치는 인터넷 트래픽을 웹 계층으로 분산합니다. 이 부하 분산 장치에 사용할 공용 IP 주소를 만듭니다. [인터넷 연결 부하 분산 장치 만들기][lb-external-create]를 참조하세요.

내부 부하 분산 장치는 웹 계층에서 비즈니스 계층으로 네트워크 트래픽을 분산합니다. 이 부하 분산 장치에 사설 IP 주소를 부여하려면 프론트 엔드 IP 구성을 만든 다음 이를 비즈니스 계층의 서브넷에 연결합니다. [내부 부하 분산 장치 만들기 시작][lb-internal-create]을 참조하세요.

### <a name="cassandra"></a>Cassandra

프로덕션 사용의 경우 [DataStax Enterprise][datastax]를 권장하나, 이러한 권장 사항은 모든 Cassandra 버전에 적용됩니다. Azure에서 DataStax 실행에 관한 자세한 내용은 [Azure용 DataStax Enterprise 배포 가이드][cassandra-in-azure]를 참조하세요. 

Cassandra 클러스터에 대한 VM을 가용성 집합에 배치하여 Cassandra 복제본이 여러 오류 도메인 및 업그레이드 도메인에 분산되도록 합니다. 장애 도메인 및 업그레이드 도메인에 대한 자세한 내용은 [Virtual Machines의 가용성 관리][azure-availability-sets]를 참조하세요. 

가용성 집합당 오류 도메인(최대) 및 가용성 집합당 업그레이드 도메인 18개를 구성합니다. 그러면 오류 도메인에 균등하게 분산될 수 있는 업그레이드 도메인의 최대 수가 제공됩니다.   

노드를 랙 인식 모드로 구성합니다. 오류 도메인을 `cassandra-rackdc.properties` 파일의 랙에 매핑합니다.

클러스터 앞에 부하 분산 장치가 필요하지 않습니다. 클라이언트는 클러스터의 노드에 직접 연결합니다.

### <a name="jumpbox"></a>Jumpbox

jumpbox는 최소한의 성능 요구 사항만을 가지므로 jumpbox에 대해 Standard A1과 같이 크기가 작은 VM을 선택합니다. 

jumpbox에 대한 [공용 IP 주소]를 만듭니다. jumpbox를 다른 VM과 동일한 VNet 안에 배치하되 별도의 관리 서브넷에 배치합니다.

공용 인터넷으로부터 응용 프로그램 워크로드를 실행하는 VM에 대한 SSH 액세스를 허용하지 않습니다. 대신 이러한 VM에 대한 모든 SSH 액세스는 jumpbox를 통해 이루어져야 합니다. 관리자는 jumpbox에 로그인한 뒤에 jumpbox에서 다른 VM에 로그인하게 됩니다. jumpbox는 인터넷에서 수신되는 SSH 트래픽 중 알려진 안전한 IP 주소만을 허용합니다.

jumpbox를 안전하게 보호하려면 NSG을 만든 다음 이를 jumpbox 서브넷에 적용합니다. 안전한 공용 IP 주소 집합으로부터 수신되는 SSH 연결만 허용하는 NSG 규칙을 추가합니다. NSG는 서브넷 또는 jumpbox NIC에 연결할 수 있습니다. 여기서는 동일한 서브넷에 다른 VM을 추가하더라도 SSH 트래픽이 jumpbox에서만 허용되도록 NIC에 연결하는 것이 좋습니다.

관리 서브넷으로부터 수신되는 SSH 트래픽을 허용하도록 다른 서브넷에 대한 NSG를 구성합니다.

## <a name="availability-considerations"></a>가용성 고려 사항

각 계층 또는 VM 역할을 별도의 가용성 집합에 배치합니다. 

데이터베이스 계층에 여러 개의 VM이 존재한다고 해서 자동으로 고가용성 데이터베이스가 되는 것은 아닙니다. 관계형 데이터베이스의 경우 고가용성을 달성하기 위해서는 일반적으로 복제와 장애 조치(failover)를 사용해야 합니다.  

[VM용 Azure SLA][vm-sla]가 제공하는 가용성보다 높은 가용성이 필요한 경우, 두 지역 간에 응용 프로그램을 복제한 다음 장애 조치(failover)를 위해 Azure Traffic Manager를 사용합니다. 자세한 내용은 [고가용성을 위해 여러 지역에서 Linux VM 실행][multi-dc]을 참조하세요.  

## <a name="security-considerations"></a>보안 고려 사항

NVA(네트워크 가상 어플라이언스)를 추가하여 공용 인터넷과 Azure 가상 네트워크 사이에 DMZ를 만드는 것도 고려합니다. NVA는 방화벽, 패킷 조사, 감사 및 사용자 지정 라우팅과 같은 네트워크 관련 작업을 수행할 수 있는 가상 어플라이언스를 통칭하는 용어입니다. 자세한 내용은 [Azure와 인터넷 사이에 DMZ 구현][dmz]을 참조하세요.

## <a name="scalability-considerations"></a>확장성 고려 사항

부하 분산 장치는 네트워크 트래픽을 웹 계층과 비즈니스 계층으로 분산합니다. 새 VM 인스턴스를 추가하여 수평 확장합니다. 부하의 정도에 따라 웹 계층과 비즈니스 계층을 각각 독립적으로 확장할 수도 있습니다. 클라이언트 선호도로 인해 발생할 수 있는 복잡성을 줄이기 위해서는 웹 계층에 속한 VM이 상태 비저장이어야 합니다. 비즈니스 로직을 호스팅하는 VM도 상태 비저장이어야 합니다.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

[Azure Automation][azure-administration], [Microsoft Operations Management Suite][operations-management-suite], [Chef][chef] 또는 [Puppet][puppet]과 같은 일원화된 관리 도구를 사용하여 전체 시스템 관리를 간소화합니다. 이러한 도구를 사용하면 여러 VM으로부터 캡처된 진단 및 상태 정보를 통합하여 시스템을 종합적으로 파악할 수 있습니다.

## <a name="deploy-the-solution"></a>솔루션 배포

이 참조 아키텍처에 대한 배포는 [GitHub][github-folder]에서 사용할 수 있습니다. 

### <a name="prerequisites"></a>필수 조건

사용자의 구독에 참조 아키텍처를 배포하려면 먼저 다음 단계를 수행해야 합니다.

1. [참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.

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

N 계층 응용 프로그램에 대한 Linux VM 참조 아키텍처를 배포하려면 다음 단계를 따릅니다.

1. 위의 필수 조건 중 1단계에서 복제한 리포지토리가 있는 `virtual-machines\n-tier-linux` 폴더로 이동합니다.

2. 매개 변수 파일은 배포 환경의 각 VM에 대해 관리자 사용자 이름과 암호의 기본값을 지정합니다. 참조 아키텍처를 배포하기 전에 먼저 이 값을 변경해야 합니다. `n-tier-linux.json` 파일을 열고 **adminUsername** 및 **adminPassword** 필드를 새로운 설정으로 변경합니다.   파일을 저장합니다.

3. 아래에 표시된 대로 **azbb** 명령줄 도구를 사용하여 참조 아키텍처를 배포합니다.

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

Azure 구성 요소를 사용하여 이 샘플 참조 아키텍처를 배포하는 방법에 대한 자세한 내용은 [GitHub 리포지토리][git]를 방문하세요.

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[요새 호스트]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
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
[0]: ./images/n-tier-diagram.png "Microsoft Azure를 사용하는 N 계층 아키텍처"

