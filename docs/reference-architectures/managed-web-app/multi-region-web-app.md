---
title: Multi-region web application
description: >-
  Recommended architecture for web application with high availability, running
  in Microsoft Azure.


author: MikeWasson





ms.service: guidance

ms.topic: article


ms.date: 11/23/2016
ms.author: pnp
cardTitle: Run in multiple regions
---
# 다지역에서 웹 응용 프로그램 실행
[!INCLUDE [header](../../_includes/header.md)]

이 참조 아키텍처는 고가용성을 얻기 위해 여러 지역에서 웹 응용 프로그램을 실행하는 방법에 대해 보여줍니다.  

![Reference architecture: Web application with high availability](./images/multi-region-web-app-diagram.png) 

## 아키텍처

이 참조 아키텍처는 [웹 응용 프로그램의 확장성 개선][guidance-web-apps-scalability]에 소개된 아키텍처를 기반으로 하고 있습니다. 주요 차이점은 다음과 같습니다.

* **주 지역 및 부 지역**. 이 아키텍처는 고가용성을 얻기 위해 두 지역을 사용합니다. 응용 프로그램은 각 지역에 배포됩니다. 정상적으로 실행되는 경우에는 네트워크 트래픽이 주 지역으로 라우팅되는데, 주 지역이 사용 불가능한 경우 트래픽은 부 지역으로 라우팅됩니다.  
* **Azure Traffic Manager**. [Traffic Manager][traffic-manager]는 들어오는 요청을 주 지역으로 라우팅하는데, 주 지역에서 실행되는 응용 프로그램이 사용 불가능한 경우에 부 지역으로 장애조치합니다.
* SQL Database와 DocumentDB의 **지역 복제(geo-replication)**. 

다지역 아키텍처는 단일 지역 배포보다 더 높은 가용성을 제공합니다. 주 지역이 지역 정전으로 영향을 받는 경우 [Traffic Manager][traffic-manager]를 사용하여 부 지역으로 장애조치할 수 있습니다. 이 아키텍처는 응용 프로그램의 개별 하위시스템 장애 발생 시에도 유용합니다.

여러 지역에서 고가용성을 얻을 수 있는 몇 가지 일반적인 접근 방식이 있습니다. 

* 액티브/패시브 상시 대기. 트래픽이 한 지역으로 가는 동안 다른 지역은 상시 대기 모드가 됩니다. 상시 대기 모드란 부 지역의 VM이 할당되어 항상 실행되는 상태를 말합니다.
* 액티브/패시브 수동 대기. 트래픽이 한 지역으로 가는 동안 다른 지역은 수동 대기 모드가 됩니다. 수동 대기 모드란 장애조치를 위해 필요해지기 전까지 부 지역의 VM이 할당되지 않는 상태를 말합니다. 이 접근 방식은 비용을 절감할 수 있지만 장애 복구까지 더 오랜 시간이 소요됩니다.
* 액티브/액티브. 두 지역 모두 액티브 상태로서 요청 부하는 두 지역에 분산됩니다. 한 지역이 중단되면 로테이션에서 제외됩니다. 

이 참조 아키텍처는 장애조치를 위한 Traffic Manager를 사용하여 상시 대기 모드에서의 액티브/패시브에 초점을 맞추고 있습니다. 


## 권장사항

이 문서에서 설명하는 아키텍처는 귀하의 요구사항과 정확히 일치하지 않을 수 있습니다. 따라서 이 문서의 권장사항은 하나의 출발점으로 삼으시기 바랍니다.

### 지역 연결
각 Azure 지역은 동일한 상위 지역 내에 위치한 다른 지역과 연결됩니다. 일반적으로 동일 지역 연결로부터 지역을 선택합니다. (예: East US 2 및 Central US) 이렇게 함으로써 다음과 같은 이점을 얻을 수 있습니다.

* 넓은 지역에서 정전이 발생하는 경우 모든 연결 중 최소 한 지역에 대해서 우선적으로 복구를 수행합니다.
* 계획된 Azure 시스템 업데이트를 연결된 지역들에 순차적으로 발행하여 중단 시간을 최소화합니다.
* 대부분의 경우, 한 쌍의 연결 지역은 동일 상위 지역 내에 위치해 데이터 상주 요구사항을 충족합니다.

그러나 두 지역 모두 응용 프로그램에 필요한 모든 Azure 서비스를 지원해야 합니다. 자세한 내용은 [지역별 서비스][services-by-region]를 참조하시기 바랍니다. 지역 연결에 대한 자세한 내용은 [비즈니스 연속성 및 재해 복구(BCDR): Azure 연결 지역][regional-pairs]을 참조하시기 바랍니다.

### 리소스 그룹
주 지역, 부 지역, Traffic Manager를 별개의 [리소스 그룹][resource groups]에 배치하는 것을 고려해 보시기 바랍니다. 이를 통해 각 지역에 배포된 리소스를 단일 모음으로 관리할 수 있습니다.

### Traffic Manager 구성

**라우팅**. Traffic Manager는 몇 가지 [라우팅 알고리즘][tm-routing]을 지원합니다. 이 문서에서 설명하는 시나리오의 경우에는 (과거 *장애조치* 라우팅으로 불린) *우선순위* 라우팅을 사용합니다. 우선순위 라우팅 설정 시 Traffic Manager는 주 지역의 끝점이 통신 불가능한 경우가 아니라며 모든 요청을 주 지역으로 전송합니다. 만약 주 지역이 통신 불가능한 상태라면 Traffic Manager가 부 지역으로 자동 장애조치를 수행합니다. [장애조치 라우팅 메서드 구성][tm-configure-failover]을 참조하시기 바랍니다.

**상태 프로브**. Traffic Manager는 HTTP(또는 HTTPS) 프로브를 사용하여 각 끝점의 가용성을 모니터링합니다. 이 프로브를 통해 Traffic Manager는 부 지역으로의 장애 조치에 대한 통과/실패 테스트를 수행할 수 있습니다. 테스트는 특정 URL 경로에 요청을 전송하는 방식으로 이루어집니다. 타임아웃 전에 비 200 응답을 수신하는 경우 해당 프로브는 실패합니다. 네 번의 요청 실패 후 Traffic Manager는 해당 끝점을 Degraded로 표시하고 다른 끝점으로 장애 조치합니다. 자세한 내용은 [Traffic Manager 끝점 모니터링 및 장애조치][tm-monitoring]를 참조하시기 바랍니다.

모범 사례 중 하나는 응용 프로그램의 전반적인 상태를 보고하는 상태 프로브 끝점을 생성하여 사용하는 것입니다. 이 끝점은 App Service 앱, 저장소 쿼리, SQL Database와 같은 중요한 의존 관계를 확인할 수 있어야 합니다. 그렇지 않으면 이 프로브는 응용 프로그램의 중요한 부분이 실제로는 고장난 경우에도 끝점의 상태를 양호하다고 보고할 수 있습니다. 

반면 이 상태 프로브를 더 낮은 우선순위 서비스를 확인하는 데 사용해서는 안 됩니다. 예를 들어, 이메일 서비스에 장애가 발생하는 경우 응용 프로그램을 부 제공자로 변경하거나 단순히 이메일을 나중에 보낼 수도 있습니다. 이러한 문제는 응용 프로그램의 장애조치를 실행할 정도로 우선순위가 높지 않습니다. 자세한 내용은 [상태 끝점 모니터링 패턴][health-endpoint-monitoring-pattern]을 참조하시기 바랍니다. 
 
### SQL 데이터베이스
[액티브 지역 복제(Geo-repilication)][sql-replication]를 사용하여 다른 지역에 읽기 가능한 부 복제본을 생성합니다. 최대 네 개의 읽기 가능한 부 복제본을 만들 수 있습니다. 주 DB에 장애가 발생하거나 주 DB를 종료해야 하는 경우에는 부 DB로 장애조치합니다. 액티브 지역 복제는 모든 탄력적 DB 풀의 모든 DB에 대해 구성될 수 있습니다. 

### DocumentDB
DocumentDB는 여러 지역에 걸친 지역 복제를 지원합니다. 한 지역은 쓰기 가능으로 지정되고 다른 지역들은 읽기 전용 복제본으로 지정됩니다. 

지역 정전 발생 시, 다른 지역을 쓰기 지역으로 선택하는 방식으로 장애 조치를 할 수 있습니다. DocumentDB 클라이언트 SDK는 자동으로 쓰기 요청을 현재 쓰기 지역으로 전송하므로 장애 조치 후 클라이언트 구성을 업데이트할 필요가 없습니다. 자세한 내용은 [DocumentDB를 이용한 글로벌 데이터 배포][docdb-geo]를 참조하시기 바랍니다.

> [!참고]
> 모든 복제본은 동일한 리소스 그룹에 속합니다.
>
>

### 저장소
Azure 저장소의 경우, [읽기-접속 지역 중복 저장소][ra-grs] (RA-GRS)를 사용합니다. RA-GRS 저장소 사용 시 데이터는 부 지역으로 복제됩니다. 개별 끝점을 통해 부 지역의 데이터에 읽기 전용으로 접속할 수 있습니다. 지역 정전이나 재해 발생 시, Azure Storage팀이 부 지역으로 지역 장애조치를 수행할 지 여부를 결정합니다. 이러한 장애 조치 시 고객이 직접 수행해야할 조치는 없습니다.

큐(Queue) 저장소의 경우 부 지역에 백업 큐를 생성합니다. 장애 조치 중 앱은 주 지역이 복구될 때까지 백업 큐를 사용할 수 있습니다. 이런 방식으로 앱은 계속해서 새 요청을 처리할 수 있습니다.

## 가용성 고려사항


### Traffic Manager

Traffic Manager는 주 지역이 사용 불가능한 경우 자동으로 장애 조치를 수행합니다. Traffic Manager가 장애 조치를 수행하는 동안 클라이언트가 해당 응용 프로그램에 접속할 수 없는 시간이 발생하는데, 이 시간은 다음과 같은 요소에 따라 달라집니다.

* 상태 프로브는 주 데이터 센터가 통신할 수 없는 상태인 경우 이를 감지해야 합니다.
* DNS 서버는 해당 IP 주소에 대한 캐시된 DNS 기록을 업데이트해야 하는데, 이는 DNS TTL(time-to-live)의 영향을 받습니다. TTL은 300초(5분)로 기본 설정되어 있지만 Traffic Manager 프로필을 생성할 때 이 값을 구성할 수 있습니다.

자세한 내용은 [Traffic Manager 모니터링][tm-monitoring]을 참조하시기 바랍니다.

이러한 시스템에서는 Traffic Manager가 장애 지점이 됩니다. 서비스 장애 시, 클라이언트는 중단 시간 동안 응용 프로그램에 액세스할 수 없습니다. [Traffic Manager SLA][tm-sla]를 검토하여 Traffic Manager를 단독으로 사용하는 것이 고가용성을 위한 비즈니스 요구사항을 충족하는지 확인하시기 바랍니다. 만일 요구사항이 충족되지 않는다면, 또 다른 트래픽 관리 솔루션을 장애 복구용으로 추가할 것을 고려해 보아야 합니다. Azure Traffic Manager 서비스 장애 시, DNS의 CNAME 기록을 변경하여 다른 트래픽 관리 서비스를 지정하도록 설정합니다. 이 단계는 수동으로 수행해야 하는데, DNS 변경이 전파될 때까지는 응용 프로그램을 이용할 수 없습니다.

### SQL 데이터베이스
SQL Database의 복구 지점 목표(RPO)와 예상 복구 시간(ERT)은 [Azure SQL Database를 통한 비즈니스 연속성 개요][sql-rpo]에 정리되어 있습니다. 

### 저장소
RA-GRS 저장소는 내구성이 우수한 저장소를 제공하지만 정전 시 어떤 일이 발생할 수 있는지를 이해하는 것이 중요합니다. 

* 저장소 정전 발생 시 데이터에 대한 쓰기 접속이 불가능한 시간이 발생합니다. 정전 중에도 부 끝점으로부터 읽기는 계속해서 가능합니다.
* 지역 정전 또는 재해가 주 위치에 영향을 주어 데이터 복구가 불가능한 경우 Azure Storage팀은 부 지역으로 지역 장애 조치를 수행할 것인지 여부를 결정할 수 있습니다.
* 부 지역으로의 데이터 복제는 비동기로 수행됩니다. 따라서 지역 장애 조치 수행 시 주 지역으로부터 데이터 복구가 불가능한 경우에는 일부 데이터 손실이 발생할 수 있습니다.
* 네트워크 정전과 같은 일시적 장애의 경우에는 저장소 장애 복구가 수행되지 않습니다. 응용 프로그램이 일시적 장애에 대한 복원성을 갖도록 설계해야 합니다. 다음과 같이 완화할 수 있습니다.
  
  * 부 지역으로부터 읽기를 수행합니다.
  * 일시적으로 새 쓰기 작업(예를 들어 큐 메시지)을 위한 다른 저장소 계정으로 변경합니다.
  * 부 지역의 데이터를 다른 저장소 계정으로 복사합니다.
  * 시스템이 장애 복구될 때까지 제한된 기능을 제공합니다. 

자세한 내용은 [Azure 저장소 정전 시 조치][storage-outage]를 참조하시기 바랍니다.

## 관리 효율성 고려사항

### Traffic Manager

Traffic Manager가 장애조치를 실행하는 경우 자동 장애 복구가 아닌 수동 장애 복구 실행을 권장합니다. 그렇지 않으면 해당 응용 프로그램이 여러 다른 지역을 왔다 갔다 하는 상황이 발생할 수 있습니다. 장애 복구를 실시하기 전 모든 응용 프로그램 하위 시스템이 정상적인 상태인지 확인합니다.

Traffic Manager는 자동 장애 복구로 기본 설정되어 있습니다. 이를 방지하려면 장애 복구 이벤트 발생 후 주 지역의 우선순위를 수동으로 낮춰야 합니다. 예를 들어, 주 지역이 우선순위 1이고 부 지역이 우선순위 2인 경우, 장애 조치 후, 주 지역을 우선순위 3으로 설정하여 자동 장애 복구를 방지할 수 있습니다. 원상 복구할 준비가 완료되면 그 때 우선순위를 다시 1로 업데이트하면 됩니다.

우선순위는 다음과 같은 명령어를 사용하여 업데이트할 수 있습니다.

**PowerShell**

```bat
$endpoint = Get-AzureRmTrafficManagerEndpoint -Name <endpoint> -ProfileName <profile> -ResourceGroupName <resource-group> -Type AzureEndpoints
$endpoint.Priority = 3
Set-AzureRmTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint
```

자세한 내용은 [Azure Traffic Manager Cmdlets][tm-ps]를 참조하시기 바랍니다. 

**Azure 명령줄 인터페이스(CLI)**

```bat
azure network traffic-manager endpoint set --name <endpoint> --profile-name <profile> --resource-group <resource-group> --type AzureEndpoints --priority 3
```    

### SQL 데이터베이스
주 DB에 장애가 발생할 경우, 수동으로 부 DB로 장애 조치를 수행합니다. 자세한 내용은 [Azure SQL DB 복구 또는 부 DB로 장애조치][sql-failover]를 참조하시기 바랍니다. 부 DB는 장애 조치 전까지 읽기 전용으로 유지됩니다.


<!-- links -->

[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[docdb-geo]: /azure/documentdb/documentdb-distribute-data-globally
[guidance-web-apps-scalability]: ./scalable-web-app.md
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[ra-grs]: /azure/storage/storage-redundancy#read-access-geo-redundant-storage
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-failover]: /azure/sql-database/sql-database-disaster-recovery
[sql-replication]: /azure/sql-database/sql-database-geo-replication-overview
[sql-rpo]: /azure/sql-database/sql-database-business-continuity#sql-database-features-that-you-can-use-to-provide-business-continuity
[storage-outage]: /azure/storage/storage-disaster-recovery-guidance
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-ps]: https://msdn.microsoft.com/library/mt125941.aspx
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
