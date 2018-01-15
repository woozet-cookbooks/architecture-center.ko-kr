---
title: "고가용성을 위해 여러 Azure 지역에서 Linux VM 실행"
description: "고가용성과 복원력을 위해 Azure의 여러 지역에 VM을 배포하는 방법"
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 7d720a004d21edbffc0ddeba54e291aa817550e0
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/08/2018
---
# <a name="run-linux-vms-in-multiple-regions-for-high-availability"></a>고가용성을 위해 여러 지역에서 Linux VM 실행

이 참조 아키텍처는 가용성 및 강력한 재해 복구 인프라를 구축하기 위해, 여러 Azure 지역에서 N 계층 응용 프로그램을 실행하기 위한 일련의 검증된 사례를 보여 줍니다. 

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>건축 

이 아키텍처는 [N 계층 응용 프로그램에 대한 Linux VM 실행](n-tier.md)에 나와 있는 아키텍처를 사용합니다. 

* **주 지역 및 보조 지역**. 더 높은 가용성을 달성하기 위해 두 개의 지역을 사용합니다. 하나는 주 지역이고 다른 하나는 장애 조치(failover)를 위한 지역입니다.
* **Azure DNS**. [Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다. Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.
* **Azure Traffic Manager**. [Traffic Manager][traffic-manager]는 들어오는 요청을 이 지역 중 하나에 라우팅합니다. 정상 작동 중에는 요청을 주 지역으로 라우팅합니다. 이 지역을 사용할 수 없게 되면 Traffic Manager가 보조 지역으로 장애 조치(failover)합니다. 자세한 내용은 [Traffic Manager 구성](#traffic-manager-configuration) 섹션을 참조하세요.
* **리소스 그룹**. 주 지역, 보조 지역 및 Traffic Manager에 대해 별도의 [리소스 그룹][resource groups]을 만듭니다. 따라서 각 지역을 단일 리소스 모음으로 유연하게 관리할 수 있습니다. 예를 들어 다른 지역으로 이동하지 않고 한 지역을 다시 배포할 수 있습니다. [리소스 그룹을 연결][resource-group-links]하므로 응용 프로그램의 모든 리소스를 나열하는 쿼리를 실행할 수 있습니다.
* **VNet**. 각 지역에 대해 별도의 VNet을 만듭니다. 주소 공간이 겹치지 않도록 합니다.
* **Apache Cassandra**. 고가용성을 위해 Azure 지역 전반의 데이터 센터에 Cassandra를 배포합니다. 각 지역 내의 노드는 지역 내의 복원력을 위해 장애 및 업그레이드 도메인을 사용하여 랙 인식 모드로 구성됩니다.

## <a name="recommendations"></a>권장 사항

다중 지역 아키텍처는 단일 지역에 배포하는 것보다 더 높은 가용성을 제공할 수 있습니다. 지역 가동 중단이 주 지역에 영향을 주는 경우 [Traffic Manager][traffic-manager]를 사용하여 보조 지역으로 장애 조치(failover)할 수 있습니다. 이 아키텍처는 응용 프로그램의 개별 하위 시스템이 고장난 경우에도 도움이 될 수 있습니다.

지역에서 고가용성을 달성하는 데 몇 가지 일반적인 접근 방식이 있습니다.   

* 활성/수동(상시 대기). 트래픽이 한 지역으로 이동하면 다른 하나가 상시 대기 상태에서 기다립니다. 상시 대기는 항상 보조 지역의 VM이 할당되고 실행 중이라는 의미입니다.
* 활성/수동(수동 대기). 트래픽이 한 지역으로 이동하면 다른 하나가 수동 대기에서 기다립니다. 수동 대기는 장애 조치(failover)에 필요할 때까지 보조 지역의 VM이 할당되지 않는다는 것입니다. 이 방법은 실행하는 데 비용이 덜 들지만, 일반적으로 실패 상태에 있을 때 온라인 상태가 되는 데 더 오래 시간이 걸립니다.
* 활성/활성. 두 지역 모두 활성화되어 있으며, 요청이 두 지역 사이에서 부하 분산됩니다. 한 지역을 사용할 수 없게 되면 회전이 중단됩니다. 

이 아키텍처는 장애 조치(failover)를 위해 Traffic Manager를 사용하여 활성/수동(상시 대기)을 중점적으로 다룹니다. 상시 대기를 위해 VM을 몇 개만 배포한 다음 필요에 따라 규모를 확장할 수 있습니다.


### <a name="regional-pairing"></a>지역을 쌍으로 연결

Azure 지역은 동일한 지역 내에서 다른 지역과 쌍을 이룹니다. 일반적으로 같은 지역 쌍에서 지역을 선택합니다. 예를 들어 미국 동부 2와 미국 중부입니다. 이에 따른 장점은 다음과 같습니다.

* 광범위한 가동 중단이 발생한 경우 모든 쌍 중에서 하나 이상의 지역에 대한 복구에 우선 순위가 지정됩니다.
* 계획된 Azure 시스템 업데이트는 순차적으로 쌍을 이루는 지역으로 출시되어 가동 중지 시간을 최소화할 수 있습니다.
* 쌍은 데이터 상주 요구 사항을 충족하기 위해 동일한 지역 내에 상주합니다.

그러나 두 지역 모두 응용 프로그램에 필요한 모든 Azure 서비스를 지원하는지 확인합니다([지역별 서비스][services-by-region] 참조). 지역 쌍에 대한 자세한 내용은 [BCDR(무중단 업무 방식 및 재해 복구): Azure 쌍을 이루는 지역][regional-pairs]을 참조하세요.

### <a name="traffic-manager-configuration"></a>Traffic Manager 구성

Traffic Manager를 구성할 때 다음 사항을 고려합니다.

* **라우팅**. Traffic Manager는 여러 [라우팅 알고리즘][tm-routing]을 지원합니다. 이 문서에 설명된 시나리오는 *우선 순위* 라우팅(이전에는 *장애 조치(failover)* 라우팅이라고 함)을 사용합니다. 이 설정을 사용하면 Traffic Manager가 주 지역에 연결할 수 없는 경우가 아닌 한 모든 요청을 주 지역으로 보냅니다. 이때 자동으로 보조 지역으로 장애 조치(failover)됩니다. [장애 조치(failover) 라우팅 방법 구성][tm-configure-failover]을 참조하세요.
* **상태 프로브**. Traffic Manager는 HTTP(또는 HTTPS) [프로브][tm-monitoring]를 사용하여 각 지역의 가용성을 모니터링합니다. 프로브는 지정된 URL 경로에 대한 HTTP 200 응답을 확인합니다. 응용 프로그램의 전반적인 상태를 보고하고 이 끝점을 상태 프로브에 사용하는 끝점을 생성하는 것이 좋습니다. 그렇지 않으면 응용 프로그램의 중요한 부분이 실제로 실패할 때 프로브에서 정상 끝점을 보고할 수 있습니다. 자세한 내용은 [상태 끝점 모니터링 패턴][health-endpoint-monitoring-pattern]을 참조하세요.

Traffic Manager가 장애 조치(failover)할 때 클라이언트가 응용 프로그램에 연결할 수 없는 기간이 있습니다. 그 기간은 다음과 같은 요인에 의해 영향을 받습니다.

* 상태 프로브가 주 지역에 연결할 수 없는지 검색해야 합니다.
* DNS 서버가 DNS TTL(time-to-live)에 따라 IP 주소에 대해 캐시된 DNS 레코드를 업데이트해야 합니다. 기본 TTL은 300초(5분)이지만 Traffic Manager 프로필을 만들 때 이 값을 구성할 수 있습니다.

자세한 내용은 [Traffic Manager 모니터링 정보][tm-monitoring]를 참조하세요.

Traffic Manager가 장애 조치(failover)를 수행하는 경우 자동 장애 복구(failback)를 구현하기 보다는 수동 장애 복구(failback)를 수행하는 것이 좋습니다. 그렇지 않으면 응용 프로그램이 지역 간에 앞뒤로 대칭 이동하는 상황이 발생할 수 있습니다. 장애 복구(failback) 전에 모든 응용 프로그램 하위 시스템이 정상 상태인지 확인합니다.

Traffic Manager는 기본적으로 자동으로 장애를 복구(failback)합니다. 이를 방지하려면 장애 조치(failover) 이벤트 후 수동으로 주 지역의 우선 순위를 낮춥니다. 예를 들어 주 지역의 우선 순위가 1이고 보조 지역의 우선 순위를 2로 가정합니다. 장애 조치(failover) 후 자동 장애 복구(failback)를 방지하기 위해 주 지역의 우선 순위를 3으로 설정합니다. 다시 전환할 준비가 되면 우선 순위를 1로 업데이트합니다.

다음 [Azure CLI][install-azure-cli] 명령은 우선 순위를 업데이트합니다.

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

또 다른 방법은 장애 복구(failback)할 준비가 될 때까지 끝점을 일시적으로 사용하지 않도록 설정하는 것입니다.

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```    

장애 조치(failover)의 원인에 따라 한 지역 내에서 리소스를 다시 배포해야 할 수 있습니다. 장애 복구(failback) 전에 운영 준비 상태 테스트를 수행합니다. 이 테스트는 다음과 같은 사항을 확인해야 합니다.

* VM이 올바르게 구성되어 있습니다. 필요한 모든 소프트웨어가 설치되어 있고, IIS가 실행 중인지 확인합니다.
* 응용 프로그램 하위 시스템이 정상입니다.
* 기능을 테스트합니다. 예를 들어 웹 계층에서 데이터베이스 계층에 연결 가능한지 테스트합니다.

### <a name="cassandra-deployment-across-multiple-regions"></a>여러 지역에 Cassandra 배포

Cassandra 데이터 센터는 복제와 워크로드 분리를 위해 클러스터 내에서 함께 구성된 관련 데이터 노드의 그룹입니다.

프로덕션 용도의 경우 [DataStax Enterprise][datastax]를 사용하는 것이 좋습니다. Azure에서 DataStax 실행에 대한 자세한 내용은 [DataStax Enterprise Deployment Guide for Azure][cassandra-in-azure](Asure용 DataStax 엔터프라이즈 배포 가이드)를 참조하세요. 다음 일반 권장 사항은 Cassandra 버전에 적용됩니다. 

* 공용 IP 주소를 각 노드에 할당합니다. 이를 통해 클러스터는 Azure 백본 인프라를 사용하여 여러 지역에서 통신할 수 있으므로 낮은 비용으로 높은 처리량을 제공할 수 있습니다.
* 적절한 방화벽 및 NSG(네트워크 보안 그룹) 구성을 사용하여 노드를 보호하므로 클라이언트 및 다른 클러스터 노드를 포함한 알려진 호스트로의 트래픽만 허용할 수 있습니다. Cassandra는 통신을 위해 OpCoenter, Spark 등의 다른 포트를 사용합니다. Cassandra에서 포트 사용은 [방화벽 포트 액세스 구성][cassandra-ports]을 참조하세요.
* 모든 [클라이언트-노드][ssl-client-node] 및 [노드-노드][ssl-node-node] 통신을 위해 SSL 암호화를 사용합니다.
* 지역 내에서 [Cassandra 권장 사항](n-tier.md#cassandra) 지침을 따릅니다.

## <a name="availability-considerations"></a>가용성 고려 사항

복잡한 N 계층 앱에서는 보조 지역에서 전체 응용 프로그램을 복제하지 않아도 될 수 있습니다. 대신 무중단 업무 방식을 지원하는 데 필요한 중요한 하위 시스템을 복제하기면 하면 됩니다.

Traffic Manager는 시스템에서 오류가 발생할 수 있는 지점입니다. Traffic Manager 서비스가 실패하면 클라이언트가 가동 중지 시간 동안 응용 프로그램에 액세스할 수 없습니다. [Traffic Manager SLA][tm-sla]를 검토하고 Traffic Manager만 사용하는 것이 고가용성을 위한 비즈니스 요구 사항을 충족하는지 확인합니다. 그렇지 않은 경우 다른 트래픽 관리 솔루션을 장애 복구(failback)로 추가합니다. Azure Traffic Manager 서비스가 실패하면 다른 트래픽 관리 서비스를 가리키도록 DNS의 CNAME 레코드를 변경합니다. (이 단계는 수동으로 수행해야 하며 DNS 변경 사항이 전파될 때까지 응용 프로그램을 사용할 수 없습니다.)

Cassandra 클러스터의 경우 고려할 장애 조치(failover) 시나리오는 응용 프로그램이 사용하는 일관성 수준과 사용한 복제 수에 따라 달라집니다. Cassandra의 일관성 수준 및 사용에 대해서는 [Configuring data consistency][cassandra-consistency](데이터 일관성 구성) 및 [Cassandra: How many nodes are talked to with Quorum?][cassandra-consistency-usage](Cassandra: Quorum과 연결되는 노드 수)을 참조하세요. Cassandra의 데이터 가용성은 응용 프로그램 및 복제 방법에서 사용되는 일관성 수준에 의해 결정됩니다. Cassandra의 복제에 대해서는 [Data Replication in NoSQL Databases Explained][cassandra-replication](NoSQL 데이터베이스의 데이터 복제 설명)를 참조하세요.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

배포를 업데이트할 때는 한 번에 하나의 지역만 업데이트하여 잘못된 구성이나 응용 프로그램의 오류로 인한 글로벌 오류 발생 가능성을 줄입니다.

장애에 대한 시스템의 복원력을 테스트합니다. 다음은 테스트에 자주 사용되는 몇 가지 일반적인 오류 시나리오입니다.

* VM 인스턴스가 중단됩니다.
* CPU 및 메모리 같은 리소스 사용을 가중시킵니다.
* 네트워크 연결을 끊거나 지연시킵니다.
* 프로세스가 충돌합니다.
* 인증서가 만료됩니다.
* 하드웨어 오류를 시뮬레이트합니다.
* 도메인 컨트롤러에서 DNS 서비스를 중단합니다.

복구 시간을 측정하고 비즈니스 요구 사항이 충족되었는지 확인합니다. 오류 모드를 조합하여 테스트합니다.


<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md
[azure-dns]: /azure/dns/dns-overview
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2
[cassandra-ports]: https://docs.datastax.com/en/datastax_enterprise/5.0/datastax_enterprise/sec/configFirewallPorts.html
[datastax]: https://www.datastax.com/products/datastax-enterprise
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssl-client-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLClientToNode_t.html
[ssl-node-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLNodeToNode_t.html
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx
[0]: ./images/multi-region-application-diagram.png "Azure N 계층 응용 프로그램에 대해 고가용성 네트워크 아키텍처"
