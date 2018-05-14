---
title: 고가용성을 위한 다중 지역 N 계층 응용 프로그램
description: 고가용성과 복원력을 위해 Azure의 여러 지역에 VM을 배포하는 방법
author: MikeWasson
ms.date: 05/03/2018
pnp.series.title: Windows VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 48943094e7847e39b9fdc4c3f71e27f2e6e41293
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/06/2018
---
# <a name="multi-region-n-tier-application-for-high-availability"></a>고가용성을 위한 다중 지역 N 계층 응용 프로그램

이 참조 아키텍처는 가용성 및 강력한 재해 복구 인프라를 구축하기 위해, 여러 Azure 지역에서 N 계층 응용 프로그램을 실행하기 위한 일련의 검증된 사례를 보여 줍니다. 

[![0]][0] 

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>아키텍처 

이 아키텍처는 [SQL Server를 통한 N 계층 응용 프로그램](n-tier-sql-server.md)에 나와 있는 아키텍처를 기반으로 합니다. 

* **주 지역 및 보조 지역**. 더 높은 가용성을 달성하기 위해 두 개의 지역을 사용합니다. 하나는 주 지역입니다. 다른 하나는 장애 조치(failover)를 위한 지역입니다.

* **Azure Traffic Manager**. [Traffic Manager][traffic-manager]는 들어오는 요청을 이 지역 중 하나에 라우팅합니다. 정상 작동 중에는 요청을 주 지역으로 라우팅합니다. 이 지역을 사용할 수 없게 되면 Traffic Manager가 보조 지역으로 장애 조치(failover)합니다. 자세한 내용은 [Traffic Manager 구성](#traffic-manager-configuration) 섹션을 참조하세요.

* **리소스 그룹**. 주 지역, 보조 지역 및 Traffic Manager에 대해 별도의 [리소스 그룹][resource groups]을 만듭니다. 따라서 각 지역을 단일 리소스 모음으로 유연하게 관리할 수 있습니다. 예를 들어 다른 지역으로 이동하지 않고 한 지역을 다시 배포할 수 있습니다. [리소스 그룹을 연결][resource-group-links]하므로 응용 프로그램의 모든 리소스를 나열하는 쿼리를 실행할 수 있습니다.

* **VNet**. 각 지역에 대해 별도의 VNet을 만듭니다. 주소 공간이 겹치지 않도록 합니다. 

* **SQL Server Always On 가용성 그룹**. SQL Server를 사용하는 경우 고가용성을 위해 [SQL Always On 가용성 그룹][sql-always-on]을 사용하는 것이 좋습니다. 두 지역 모두에 SQL Server 인스턴스를 포함하는 단일 가용성 그룹을 만듭니다. 

    > [!NOTE]
    > 또한 관계형 데이터베이스를 클라우드 서비스로 제공하는 [Azure SQL Database][azure-sql-db]를 고려합니다. SQL Database를 사용하면 가용성 그룹을 구성하거나 장애 조치(failover)를 관리할 필요가 없습니다.  
    > 

* **VPN Gateway**. 각 VNet에서 [VPN Gateway][vpn-gateway]를 만들고 [VNet 간 연결][vnet-to-vnet]을 구성하여 두 VNet 사이에서 네트워크 트래픽을 사용합니다. 이는 SQL Always On 가용성 그룹에 필요합니다.

## <a name="recommendations"></a>권장 사항

다중 지역 아키텍처는 단일 지역에 배포하는 것보다 더 높은 가용성을 제공할 수 있습니다. 지역 가동 중단이 주 지역에 영향을 주는 경우 [Traffic Manager][traffic-manager]를 사용하여 보조 지역으로 장애 조치(failover)할 수 있습니다. 이 아키텍처는 응용 프로그램의 개별 하위 시스템이 고장난 경우에도 도움이 될 수 있습니다.

지역에서 고가용성을 달성하는 데 몇 가지 일반적인 접근 방식이 있습니다. 

* 활성/수동(상시 대기). 트래픽이 한 지역으로 이동하면 다른 하나가 상시 대기 상태에서 기다립니다. 상시 대기는 항상 보조 지역의 VM이 할당되고 실행 중이라는 의미입니다.
* 활성/수동(수동 대기). 트래픽이 한 지역으로 이동하면 다른 하나가 수동 대기에서 기다립니다. 수동 대기는 장애 조치(failover)에 필요할 때까지 보조 지역의 VM이 할당되지 않는다는 것입니다. 이 방법은 실행하는 데 비용이 덜 들지만, 일반적으로 실패 상태에 있을 때 온라인 상태가 되는 데 더 오래 시간이 걸립니다.
* 활성/활성. 두 지역 모두 활성화되어 있으며, 요청이 두 지역 사이에서 부하 분산됩니다. 한 지역을 사용할 수 없게 되면 회전이 중단됩니다. 

이 참조 아키텍처는 장애 조치(failover)를 위해 Traffic Manager를 사용하여 활성/수동(상시 대기)을 중점적으로 다룹니다. 상시 대기를 위해 VM을 몇 개만 배포한 다음 필요에 따라 규모를 확장할 수 있습니다.

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

### <a name="configure-sql-server-always-on-availability-groups"></a>SQL Server Always On 가용성 그룹 구성

Windows Server 2016 이전 버전에서는 SQL Server Always On 가용성 그룹에 항상 도메인 컨트롤러가 필요하며, 가용성 그룹의 모든 노드가 동일한 AD(Active Directory) 도메인에 있어야 합니다. 

가용성 그룹을 구성하려면:

* 최소한 두 개의 도메인 컨트롤러를 각 지역에 배치합니다.
* 각 도메인 컨트롤러에 고정 IP 주소를 지정합니다.
* VNet 간 연결을 만들어 VNet 간의 통신을 사용하도록 설정합니다.
* 각 VNet에 대해 두 지역 모두에서 도메인 컨트롤러의 IP 주소를 DNS 서버 목록에 추가합니다. 다음 CLI 명령을 사용할 수 있습니다. 자세한 내용은 [VNet(가상 네트워크)에서 사용하는 DNS 서버 관리][vnet-dns]를 참조하세요.

    ```bat
    azure network vnet set --resource-group dc01-rg --name dc01-vnet --dns-servers "10.0.0.4,10.0.0.6,172.16.0.4,172.16.0.6"
    ```

* 두 지역의 SQL Server 인스턴스를 포함하는 WSFC([Windows Server 장애 조치(failover) 클러스터링)][wsfc] 클러스터를 만듭니다. 
* 주 지역과 보조 지역 모두에서 SQL Server 인스턴스를 포함하는 SQL Server Always On 가용성 그룹을 만듭니다. 단계는 [Extending Always On Availability Group to Remote Azure Datacenter (PowerShell)](https://blogs.msdn.microsoft.com/sqlcat/2014/09/22/extending-alwayson-availability-group-to-remote-azure-datacenter-powershell/)(원격 Azure 데이터 센터에 Always On 가용성 그룹 확장(PowerShell))를 참조하세요.

  * 주 지역에 주 복제본을 배치합니다.
  * 주 지역에 하나 이상의 보조 복제본을 배치합니다. 자동 장애 조치(failover)를 사용하는 동기 커밋을 사용하도록 구성합니다.
  * 보조 지역에 보조 복제본을 하나 이상 배치합니다. 성능상의 이유로 *비동기* 커밋을 사용하도록 구성합니다. (그렇지 않은 경우 모든 T-SQL 트랜잭션이 네트워크를 통해 보조 지역으로 왕복하는 동안 기다려야 합니다.)

    > [!NOTE]
    > 비동기 커밋 복제본은 자동 장애 조치(failover)를 지원하지 않습니다.
    >
    >

## <a name="availability-considerations"></a>가용성 고려 사항

복잡한 N 계층 앱에서는 보조 지역에서 전체 응용 프로그램을 복제하지 않아도 될 수 있습니다. 대신 무중단 업무 방식을 지원하는 데 필요한 중요한 하위 시스템을 복제하기면 하면 됩니다.

Traffic Manager는 시스템에서 오류가 발생할 수 있는 지점입니다. Traffic Manager 서비스가 실패하면 클라이언트가 가동 중지 시간 동안 응용 프로그램에 액세스할 수 없습니다. [Traffic Manager SLA][tm-sla]를 검토하고 Traffic Manager만 사용하는 것이 고가용성을 위한 비즈니스 요구 사항을 충족하는지 확인합니다. 그렇지 않은 경우 다른 트래픽 관리 솔루션을 장애 복구(failback)로 추가합니다. Azure Traffic Manager 서비스가 실패하면 다른 트래픽 관리 서비스를 가리키도록 DNS의 CNAME 레코드를 변경합니다. (이 단계는 수동으로 수행해야 하며 DNS 변경 사항이 전파될 때까지 응용 프로그램을 사용할 수 없습니다.)

SQL Server 클러스터의 경우 다음과 같은 두 가지 장애 조치(failover) 시나리오를 고려해야 합니다.

- 주 지역의 모든 SQL Server 데이터베이스 복제본이 실패합니다. 예를 들어 이 장애는 지역 가동 중단 중에 발생할 수 있습니다. 그런 경우 Traffic Manager가 자동으로 프런트 엔드에서 장애 조치(failover)하더라도 가용성 그룹을 수동으로 장애 조치(failover)해야 합니다. [SQL Server 가용성 그룹의 강제 수동 장애 조치(failover) 수행](https://msdn.microsoft.com/library/ff877957.aspx)의 단계를 따릅니다. SQL Server 2016에서 SQL Server Management Studio, Transact-SQL 또는 PowerShell을 사용하여 강제 장애 조치(failover)를 수행하는 방법을 설명합니다.

   > [!WARNING]
   > 강제 장애 조치(failover)를 사용하면 데이터가 손실될 위험이 있습니다. 주 지역이 다시 온라인 상태가 되면 데이터베이스의 스냅숏을 만들고 [tablediff]를 사용하여 차이점을 찾습니다.
   >
   >
- Traffic Manager는 보조 지역으로 장애 조치(failover)하지만 주 SQL Server 데이터베이스 복제본은 계속 사용할 수 있습니다. 예를 들어 SQL Server VM에 영향을 주지 않고 프런트 엔드 계층이 실패할 수 있습니다. 이 경우 인터넷 트래픽은 보조 지역으로 라우팅되며, 해당 지역은 여전히 주 복제본에 연결될 수 있습니다. 그러나 SQL Server 연결이 지역 간에 이루어지므로 대기 시간이 늘어납니다. 이 경우 다음과 같이 수동 장애 조치(failover)를 수행해야 합니다.

   1. 보조 지역의 SQL Server 데이터베이스 복제본을 *동기* 커밋으로 임시 전환합니다. 그러면 장애 조치(failover) 중 데이터 손실이 발생하지 않습니다.
   2. 해당 복제본으로 장애 조치(failover)합니다.
   3. 주 지역으로 다시 장애 복구(failback)하는 경우 비동기 커밋 설정을 복원합니다.

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
[azure-sla]: https://azure.microsoft.com/support/legal/sla/
[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-always-on]: https://msdn.microsoft.com/library/hh510230.aspx
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vnet-dns]: /azure/virtual-network/virtual-networks-manage-dns-in-vnet
[vnet-to-vnet]: /azure/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps
[vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx

[0]: ./images/multi-region-sql-server.png "Azure N 계층 응용 프로그램에 적합한 고가용성 네트워크 아키텍처"
