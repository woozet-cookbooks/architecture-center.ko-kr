---
title: Azure 서비스에 대한 복원력 검사 목록
description: 다양한 Azure 서비스에 대한 복원력 지침을 제공하는 검사 목록입니다.
author: petertaylor9999
ms.date: 03/02/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: 25d961d6bb753b1f515fc073e51bbb912cc59db7
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/08/2018
---
# <a name="resiliency-checklist-for-specific-azure-services"></a>특정 Azure 서비스에 대한 복원력 검사 목록

복원력은 오류를 복구하여 계속 작동하는 시스템 기능이며 [소프트웨어 품질 핵심 요소](../guide/pillars.md) 중 하나입니다. 모든 기술에는 고유한 특정 오류 모드가 있습니다. 이 기능은 응용 프로그램을 디자인하고 구현할 때 고려해야 합니다. 이 검사 목록을 사용하여 특정 Azure 서비스에 대한 복원력 고려 사항을 검토합니다. 또한 [일반 복원력 검사 목록](./resiliency.md)을 검토합니다.

## <a name="app-service"></a>App Service

**표준 또는 프리미엄 계층을 사용합니다.** 이러한 계층은 스테이징 슬롯 및 자동화된 백업을 지원합니다. 자세한 내용은 [Azure App Service 계획 심층 개요](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)를 참조하세요.

**확장 또는 축소를 피합니다.** 그 대신에 일반적인 부하에서 성능 요구 사항에 맞는 계층 및 인스턴스 크기를 선택한 다음 [규모 확장](/azure/app-service-web/web-sites-scale/)하여 트래픽 분량의 변화를 처리합니다. 확장 및 축소는 응용 프로그램 다시 시작을 트리거할 수 있습니다.  

**구성을 앱 설정으로 저장합니다.** 앱 설정을 사용하여 구성 설정을 앱 설정으로 저장합니다. Resource Manager 템플릿에 설정을 정의하거나 PowerShell을 사용하여 해당 설정을 자동화된 배포 / 업데이트 프로세스의 일부로 적용할 수 있도록 합니다(이 방법이 신뢰성이 더 높음). 자세한 내용은 [Azure App Service에서 웹앱 구성](/azure/app-service-web/web-sites-configure/)을 참조하세요.

**프로덕션 및 테스트에 대해 별도의 앱 서비스 계획을 만듭니다.** 테스트를 위한 프로덕션 배포에 슬롯을 사용하지 마십시오.  동일한 App Service 계획 내의 모든 앱은 같은 VM 인스턴스를 공유합니다. 프로덕션과 테스트 배포를 동일한 계획에 배치하면 프로덕션 배포에 부정적 영향을 줄 수 있습니다. 예를 들어 부하 테스트는 실시간 프로덕션 사이트의 성능을 저하시킬 수 있습니다. 테스트 배포를 별도의 계획에 배치하여 프로덕션 버전과 격리합니다.  

**웹앱을 웹 API와 분리합니다.** 솔루션에 웹 프런트 엔드 및 웹 API가 모두 포함된 경우 이들을 별도의 App Service 앱으로 분해하는 것을 고려합니다. 이 설계를 통해 워크로드별로 솔루션을 분해하기가 더 쉬워집니다. 웹앱과 API를 별도의 App Service 계획에서 실행할 수 있으므로 이들을 독립적으로 크기 조정할 수 있습니다. 처음에 이러한 수준의 확장성이 필요하지 않은 경우 앱을 동일한 계획에 배포하고 필요한 경우 나중에 별도의 계획으로 이동할 수 있습니다.

**앱 서비스 백업 기능을 사용하여 Azure SQL Database를 백업하는 것을 피하십시오.** 그 대신에 [SQL Database 자동화된 백업][sql-backup]을 사용합니다. App Service 백업은 데이터베이스를 SQL .bacpac 파일에 내보내는데, 이때 DTU 비용이 발생합니다.  

**스테이징 슬롯에 배포** 스테이징을 위한 배포 슬롯을 만들 수 있습니다. 응용 프로그램 업데이트를 스테이징 슬롯에 배포하고 프로덕션으로 교환하기 전에 배포를 확인합니다. 이렇게 하면 프로덕션에서 잘못된 업데이트의 기회가 감소합니다. 또한 모든 인스턴스가 프로덕션으로 교환되기 전에 확실히 준비되기도 합니다. 많은 응용 프로그램에는 중요한 준비 및 콜드 부팅 시간이 있습니다. 자세한 내용은 [Azure App Service에서 웹앱에 대한 스테이징 환경 설정](/azure/app-service-web/web-sites-staged-publishing/)을 참조하세요.

**마지막으로 성공한(LKG) 배포를 저장하는 배포 슬롯을 만듭니다.** 업데이트를 프로덕션에 배포하는 경우 이전 프로덕션 배포를 LKG 슬롯으로 이동합니다. 이렇게 하면 잘못된 배포를 롤백하기가 더 쉬워집니다. 나중에 문제가 발견되면 LKG 버전으로 신속하게 되돌릴 수 있습니다. 자세한 내용은 [기본적인 웹 응용 프로그램](../reference-architectures/app-service-web-app/basic-web-app.md)을 참조하세요.

응용 프로그램 로깅 및 웹 서버 로깅을 포함하여 **진단 로깅 설정**합니다. 로깅은 모니터링 및 진단을 위해 중요합니다. [Azure App Service에서 웹앱에 대한 진단 로깅 설정](/azure/app-service-web/web-sites-enable-diagnostic-log/) 참조

**Blob 저장소에 기록합니다.** 이렇게 하면 보다 쉽게 데이터를 수집 및 분석할 수 있습니다.

**로그에 대한 별도의 저장소 계정을 만듭니다.** 로그와 응용 프로그램 데이터에 동일한 저장소 계정을 사용하지 마세요. 이렇게 하면 로깅이 응용 프로그램의 성능을 감소시키는 것을 방지하는 데 도움이 됩니다.

**성능을 모니터링합니다.** [New Relic](http://newrelic.com/) 또는 [Application Insights](/azure/application-insights/app-insights-overview/) 같은 성능 모니터링 서비스를 사용하여 응용 프로그램 성능 및 부하를 받을 때의 동작을 모니터링합니다.  성능 모니터링은 응용 프로그램에 대한 실시간 통찰력을 제공합니다. 문제를 진단하고 실패의 근본 원인 분석을 수행할 수 있습니다.

## <a name="application-gateway"></a>Application Gateway

**인스턴스를 두 개 이상 프로비전합니다.** 응용 프로그램 게이트웨이를 두 개 이상의 인스턴스와 함께 배포합니다. 단일 인스턴스는 단일 실패 지점입니다. 중복성 및 확장성을 위해 인스턴스를 두 개 이상 사용합니다. [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/)에 대한 자격을 얻으려면 중, 두 개 이상의 보통 이상의 인스턴스를 두 개 이상 프로비전해야 합니다.

## <a name="cosmos-db"></a>Cosmos DB

**지역에 걸쳐 데이터베이스를 복제합니다.** Cosmos DB를 사용하면 개수에 관계없이 Azure 지역을 원하는 만큼 Cosmos DB 데이터베이스 계정에 연결할 수 있습니다. Cosmos DB 데이터베이스는 하나의 쓰기 지역 및 다중 읽기 지역을 포함할 수 있습니다. 쓰기 지역에 오류가 있으면 다른 복제본에서 읽을 수 있습니다. 클라이언트 SDK는 이 작업을 자동으로 처리합니다. 또한 쓰기 지역을 다른 지역으로 장애 조치할 수도 있습니다. 자세한 내용은 [Azure Cosmos DB로 데이터를 전역적으로 배포하는 방법](/azure/cosmos-db/distribute-data-globally)을 참조하세요.

## <a name="event-hubs"></a>Event Hubs


**검사점 사용**.  이벤트 소비자는 미리 정의된 간격으로 영구 저장소에 현재 위치를 기록해야 합니다. 이렇게 하면 소비자에게 오류가 발생하는 경우(예: 소비자에게 충돌이나 호스트 장애가 발생하는 경우) 새 인스턴스가 마지막으로 기록된 위치에서 스트림 읽기를 다시 시작할 수 있습니다. 자세한 내용은 [이벤트 소비자](/azure/event-hubs/event-hubs-features#event-consumers)를 참조하세요.

**중복 메시지 처리.** 이벤트 소비자가 실패하면 메시지 처리가 기록된 마지막 검사점에서 다시 시작됩니다. 마지막 검사점 이후에 이미 처리된 메시지가 다시 처리됩니다. 따라서 메시지 처리 논리가 idempotent이거나 응용 프로그램이 메시지를 중복할 수 있어야 합니다.

**예외 처리.** 이벤트 소비자는 일반적으로 루프에 있는 메시지를 일괄 처리합니다. 단일 메시지에 예외가 발생하는 경우 전체 메시지의 일괄 처리가 손실되지 않도록 하려면 이 처리 루프 내에서 예외를 처리해야 합니다.

**배달하지 못한 편지 큐 사용.** 메시지를 처리하여 일시적이지 않은 오류가 발생한 경우 상태를 추적할 수 있도록 배달하지 못한 편지 큐에 메시지를 배치합니다. 시나리오에 따라 메시지를 나중에 다시 시도하거나, 보정 트랜잭션을 적용하거나, 다른 작업을 수행할 수 있습니다. Event Hubs에는 기본 제공 배달하지 못한 편지 큐 기능이 없습니다. Azure Queue Storage 또는 Service Bus를 사용하여 배달하지 못한 편지 큐를 구현하거나 Azure Functions 또는 다른 이벤트 메커니즘을 사용할 수 있습니다.  

**보조 Event Hubs 네임스페이스에 장애 조치하여 재해 복구 구현.** 자세한 내용은 [Azure Event Hubs 지역 재해 복구](/azure/event-hubs/event-hubs-geo-dr)를 참조하세요.

## <a name="redis-cache"></a>Redis Cache

**지역에서 복제 구성**. 지역에서 복제는 두 개의 프리미엄 계층 Azure Redis Cache 인스턴스를 연결하는 메커니즘을 제공합니다. 기본 캐시에 작성된 데이터는 읽기 전용 보조 캐시에 복제됩니다. 자세한 내용은 [Azure Redis Cache에 대해 지역에서 복제를 구성하는 방법](/azure/redis-cache/cache-how-to-geo-replication)을 참조하세요.

**데이터 지속성 구성.** Redis 지속성을 사용하면 Redis에 저장된 데이터를 유지할 수 있습니다. 또한 스냅숏을 만들고, 하드웨어 오류 시 로드할 수 있게 데이터를 백업할 수 있습니다. 자세한 내용은 [프리미엄 Azure Redis Cache에 데이터 지속성을 구성하는 방법](/azure/redis-cache/cache-how-to-premium-persistence)을 참조하세요.

Redis Cache를 영구 저장소가 아닌 임시 데이터 캐시로 사용하는 경우 이러한 권장 사항이 적용되지 않을 수 있습니다. 

## <a name="search"></a>검색

**복제본을 두 개 이상 프로비전합니다.** 읽기 고가용성을 위해서는 복제를 두 개 이상, 또는 읽기-쓰기 고가용성을 위해서는 세 개 이상 사용합니다.

**다중 지역 배포의 경우 인덱서를 구성합니다.** 다중 지역 배포를 설정한 경우 옵션에 대해 인덱싱의 지속성을 고려합니다.

  * 데이터 원본이 지리적으로 복제되는 경우 일반적으로 각 Azure Search의 인덱서가 각각 해당 로컬 데이터 원본 복제본을 가리키게 해야 합니다. 그러나 Azure SQL Database에 저장된 대형 데이터 집합의 경우 이 방법이 권장되지 않습니다. 그 이유는 Azure Search가 보조 SQL Database 복제에서 증분 인덱싱을 수행할 수 없고 주 복제본에서만 가능하기 때문입니다. 그 대신에 모든 인덱서가 주 복제본을 가리킵니다. 장애 조치 후 새 주 복제본에서 Azure Search 인덱서를 가리킵니다.  
  * 데이터 원본이 지리적으로 복제되지 않는 경우 여러 지역의 Azure Search 서비스가 지속적으로 그리고 독립적으로 데이터 원본에서 인덱싱하도록 여러 인덱서가 동일한 데이터 원본을 가리키게 합니다. 자세한 내용은 [Azure Search 성능 및 최적화 고려 사항][search-optimization]을 참조하세요.

## <a name="storage"></a>Storage

**응용 프로그램 데이터의 경우 RA-GRS(읽기-쓰기 지역 중복 저장소)를 사용합니다.** RA-GRS 저장소는 데이터를 보조 지역에 복제하고 보조 지역에서 읽기 전용 액세스를 제공합니다. 주 지역에 저장소 중단이 있는 경우 응용 프로그램은 보조 지역에서 데이터를 읽을 수 있습니다. 자세한 내용은 [Azure Storage 복제](/azure/storage/storage-redundancy/)를 참조하세요.

**VM 디스크의 경우 Managed Disks를 사용합니다.** [Managed Disks][managed-disks]는 디스크들이 서로 충분히 격리되어 단일 실패 지점을 방지하므로 가용성 집합의 VM에 대해 더 나은 신뢰성을 제공합니다. 또한 Managed Disks는 저장소 계정에서 만든 VHD의 IOPS 제한이 적용되지 않습니다. 자세한 내용은 [Azure에서 Windows 가상 머신의 가용성 관리][vm-manage-availability]를 참조하세요.

**Queue Storage의 경우 다른 지역에 백업 큐를 만듭니다.** Queue Storage의 경우 항목을 큐에 저장하거나 큐에서 제거할 수 없으므로 읽기 전용 복제본의 사용이 제한됩니다. 그 대신에 다른 지역의 저장소 계정에 백업 큐를 만듭니다. 저장소 중단이 있는 경우 응용 프로그램은 주 지역이 다시 사용할 수 있게 될 때까지 백업 큐를 사용할 수 있습니다. 이런 방식으로 응용 프로그램이 새 요청을 계속 처리할 수 있습니다.  

## <a name="sql-database"></a>SQL Database

**표준 또는 프리미엄 계층을 사용합니다.** 이러한 계층은 더 긴 지정 시간 복원 기간(35일)을 제공합니다. 자세한 내용은 [SQL Database 옵션 및 성능](/azure/sql-database/sql-database-service-tiers/)을 참조하세요.

**SQL Database 감사를 사용합니다.** 감사를 사용하면 악의적 공격 또는 인적 오류를 진단할 수 있습니다. 자세한 내용은 [SQL 데이터베이스 감사 시작](/azure/sql-database/sql-database-auditing-get-started/)을 참조하세요.

**활성 지역 복제 사용** 활성 지역 복제를 사용하여 서로 다른 지역에 읽을 수 있는 보조 복제를 만듭니다.  주 데이터베이스가 실패하거나 단순히 오프라인으로 전환해야 하는 경우 보조 데이터베이스로 장애 조치를 수행합니다.  장애 조치(failover)될 때까지 보조 데이터베이스는 읽기 전용으로 유지됩니다.  자세한 내용은 [SQL Database 활성 지역 복제](/azure/sql-database/sql-database-geo-replication-overview/)를 참조하세요.

**분할을 사용합니다.** 분할을 사용하여 데이터베이스를 가로로 분할하는 것을 고려합니다. 분할은 오류 격리를 제공할 수 있습니다. 자세한 내용은 [Azure SQL Database를 사용하여 분할](/azure/sql-database/sql-database-elastic-scale-introduction/)을 참조하세요.

**지정 시간 복원을 사용하여 인적 오류에서 복구합니다.**  지정 시간 복원은 데이터베이스를 이전 시점으로 되돌려 놓습니다. 자세한 내용은 [자동화된 데이터베이스 백업을 사용하여 Azure SQL 데이터베이스 복구][sql-restore]를 참조하세요.

**지리적 복원을 사용하여 서비스 중단에서 복구합니다.** 지리적 복원은 지역 중복 백업에서 데이터베이스를 복원합니다.  자세한 내용은 [자동화된 데이터베이스 백업을 사용하여 Azure SQL 데이터베이스 복구][sql-restore]를 참조하세요.

## <a name="sql-server-running-in-a-vm"></a>VM에서 실행되는 SQL Server

**데이터베이스를 복제합니다.** SQL Server Always On 가용성 그룹을 사용하여 데이터베이스를 복제합니다. SQL Server 인스턴스 하나가 실패하는 경우 고가용성을 제공 합니다. 자세한 내용은 [N 계층 응용 프로그램에 대해 Windows VM 실행](../reference-architectures/virtual-machines-windows/n-tier.md) 참조

**데이터베이스를 백업합니다**. 이미 [Azure Backup](https://azure.microsoft.com/documentation/services/backup/)을 사용하여 VM을 백업하는 경우 [DPM을 사용하는 SQL Server용 Azure Backup 워크로드](/azure/backup/backup-azure-backup-sql/)의 사용을 고려합니다. 이 방법에서는 조직에 대한 백업 관리자 역할 하나와 VM 및 SQL Server에 대한 통합 복구 절차가 있습니다. 그렇지 않으면 [Microsoft Azure에 대한 SQL Server 관리 백업](https://msdn.microsoft.com/library/dn449496.aspx)을 사용합니다.

## <a name="traffic-manager"></a>Traffic Manager

**수동 장애 복구를 수행합니다.** Traffic Manager 장애 조치 후 자동으로 장애 복구하는 대신 수동 장애 복구를 수행합니다. 장애 복구 전에 모든 응용 프로그램 하위 시스템이 정상 상태인지 확인합니다.  그렇지 않으면 응용 프로그램이 데이터 센터 간에 앞뒤로 뒤집어지는 상황이 발생할 수 있습니다. 자세한 내용은 [고가용성을 위해 여러 지역에서 VM 실행](../reference-architectures/virtual-machines-windows/multi-region-application.md)을 참조하세요.

**상태 프로브 끝점을 만듭니다.** 응용 프로그램의 전반적인 상태에 관하여 보고하는 사용자 지정 끝점을 만듭니다. 이렇게 하면 단지 프런트 엔드뿐만 아니라 중요 경로가 고장인 경우 Traffic Manager가 장애 조치할 수 있습니다. 끝점은 중요 의존성이 비정상 상태이거나 도달할 수 없는 경우 HTTP 오류 코드를 반환해야 합니다. 그러나 중요하지 않은 서비스에 대해서는 오류를 보고하지 마세요. 그렇지 않으면 상태 프로브가 필요하지 않을 때 장애 조치를 트리거하여 가양성이 발생할 수 있습니다. 자세한 내용은 [Traffic Manager 끝점 모니터링 및 장애 조치](/azure/traffic-manager/traffic-manager-monitoring/)를 참조하세요.

## <a name="virtual-machines"></a>Virtual Machines

**단일 VM에서 프로덕션 워크로드를 실행하지 않습니다.** 단일 VM 배포는 계획되거나 계획되지 않은 유지 관리에 탄력적으로 대처할 수 없습니다. 그 대신에 부하 분산 장치를 전면에 배치한 상태에서 여러 VM을 가용성 집합 또는 [VM 확장 집합](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/)에 배치합니다.

**VM을 프로비전할 때 가용성 집합을 지정합니다.** 현재 VM이 프로비전된 후 VM을 가용성 집합에 추가하는 방법은 없습니다. 기존 가용성 집합에 새 VM을 추가하는 경우 VM에 대한 NIC를 만들고 부하 분산 장치의 백 엔드 주소 풀에 NIC를 추가해야 합니다. 그렇지 않으면 부하 분산 장치가 네트워크 트래픽을 해당 VM에 경로 설정하지 않습니다.

**각 응용 프로그램 계층을 별도의 가용성 집합에 배치합니다.** N 계층 응용 프로그램에서 서로 다른 계층의 VM을 동일한 가용성 집합에 배치하지 마세요. 가용성 집합의 VM은 장애 도메인(FD) 및 업데이트 도메인(UD)에 걸쳐 배치됩니다. 그러나 FD와 UD의 중복성 이점을 활용하려면 가용성 집합의 모든 VM이 동일한 클라이언트 요청을 처리할 수 있어야 합니다.

**성능 요구 사항을 기반으로 올바른 VM 크기를 선택합니다.** 기존 워크로드를 Azure로 이동할 때 온-프레미스 서버와 가장 근접하게 일치하는 VM 크기부터 사용하기 시작합니다. 그런 다음 CPU, 메모리 및 디스크 IOPS에 따라 실제 워크로드의 성능을 측정하고 필요에 따라 크기를 조정합니다. 이렇게 하면 해당 응용 프로그램이 클라우드 환경에서 예상한 대로 작동합니다. 또한 여러 NIC가 필요한 경우 각 크기에 대한 NIC 제한을 알아야 합니다.

**VHD에 Managed Disks를 사용합니다.** [Managed Disks][managed-disks]는 디스크들이 서로 충분히 격리되어 단일 실패 지점을 방지하므로 가용성 집합의 VM에 대해 더 나은 신뢰성을 제공합니다. 또한 Managed Disks는 저장소 계정에서 만든 VHD의 IOPS 제한이 적용되지 않습니다. 자세한 내용은 [Azure에서 Windows 가상 머신의 가용성 관리][vm-manage-availability]를 참조하세요.

**응용 프로그램을 OS 디스크가 아닌 데이터 디스크에 설치합니다.** 이렇게 하지 않으면 디스크 크기 제한에 도달할 수 있습니다.

**Azure Backup을 사용하여 VM을 백업합니다.** 백업은 실수로 인한 데이터 손실을 방지합니다. 자세한 내용은 [복구 서비스 자격 증명 모음으로 Azure VM 보호](/azure/backup/backup-azure-vms-first-look-arm/)를 참조하세요.

기본 상태 메트릭, 인프라 로그 및 [부팅 진단][boot-diagnostics]을 포함하여 **진단 로그를 사용합니다**. 부팅 진단은 VM이 부팅할 수 없는 상태로 전환되는 경우 부팅 오류를 진단하는 데 도움이 될 수 있습니다. 자세한 내용은 [Azure 진단 로그][diagnostics-logs]를 참조하세요.

**AzureLogCollector 확장을 사용합니다.** (Windows VM만 해당) 이 확장은 운영자가 원격으로 VM에 로그인하지 않고 Azure 플랫폼 로그를 집계하고 Azure 저장소에 업로드합니다. 자세한 내용은 [AzureLogCollector 확장](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)을 참조하세요.

## <a name="virtual-network"></a>Virtual Network

**공용 IP 주소를 허용하거나 차단하려면 NSG를 서브넷에 추가합니다.** 악의적인 사용자의 액세스를 차단하거나 응용 프로그램에 액세스할 권한을 가진 사용자의 액세스만 허용합니다.  

**사용자 지정 상태 프로브를 만듭니다.** 부하 분산 장치 상태 프로브는 HTTP 또는 TCP를 테스트할 수 있습니다. VM에서 HTTP 서버를 실행하는 경우 HTTP 프로브는 TCP 프로브보다 더 나은 상태 표시기입니다. HTTP 프로브의 경우 모든 중요 의존성을 포함하여 응용 프로그램의 전반적인 상태를 보고하는 사용자 지정 끝점을 사용합니다. 자세한 내용은 [Azure 부하 분산 장치 개요](/azure/load-balancer/load-balancer-overview/)를 참조하세요.

**상태 프로브를 차단하지 않습니다.** 부하 분산 장치 상태 프로브는 알려진 IP 주소 168.63.129.16에서 전송됩니다. 방화벽 정책 또는 NSG(네트워크 보안 그룹)에서 IP에 대한 트래픽을 차단하지 마세요. 상태 프로브를 차단하면 부하 분산 장치가 VM을 윤번에서 제거할 것입니다.

**부하 분산 장치 로깅을 사용하도록 설정합니다.** 로그는 백 엔드의 VM이 실패한 프로브 응답으로 인해 네트워크 트래픽을 수신하지 못하는 횟수를 표시합니다. 자세한 내용은 [Azure Load Balancer에 대한 로그 분석](/azure/load-balancer/load-balancer-monitor-log/)을 참조하세요.

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[search-optimization]: /azure/search/search-performance-optimization/
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
