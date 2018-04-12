---
title: Azure 지역의 손실로부터 복구
description: 재해 복구에 대한 계획 뿐만 아니라 복원력 있고 항상 사용 가능한 내결함성 응용 프로그램을 이해하고 설계하는 방법에 대한 문서입니다.
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: f551e8af8aece8aa30abfba2438c41c3944209bd
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-a-region-wide-service-disruption"></a>Azure 복원력 기술 지침: 지역 전체의 서비스 중단으로부터 복구
Azure는 물리적 및 논리적으로 지역이라는 단위로 구분됩니다. 지역은 가까운 위치에 있는 하나 이상의 데이터로 구성됩니다. 

드물지만 예를 들어 네트워크 오류로 인해 전체 지역의 시설에 액세스할 수 없게 될 수 있습니다. 또는 자연 재해로 인해 시설이 완전히 손실될 수 있습니다. 이 섹션에서는 지역에 걸쳐 배포되는 응용 프로그램을 만드는 Azure의 기능을 설명합니다. 이러한 배포는 한 지역의 오류가 다른 지역에 영향을 줄 수 있는 가능성을 최소화하는 데 도움을 줍니다.

## <a name="cloud-services"></a>클라우드 서비스
### <a name="resource-management"></a>리소스 관리
각 대상 지역에 별도 클라우드 서비스를 만들고 각 클라우드 서비스에 배포 패키지를 게시하여 지역에 걸쳐 계산 인스턴스를 배포할 수 있습니다. 그러나 다른 지역의 클라우드 서비스에 트래픽을 분산시키면 응용 프로그램 개발자 또는 트래픽 관리 서비스에서 구현해야 합니다.

재해 복구의 사전에 배포할 예비 역할 인스턴스의 수를 결정하는 것은 용량 계획의 중요한 측면입니다. 대규모 보조 배포를 통해 필요할 때 용량을 사용할 수 있는지 확인할 수 있지만 사실상 비용은 두 배가 됩니다. 일반적인 패턴은 중요 서비스를 실행할 만큼의 크기인 소규모 보조 배포입니다. 소규모 보조 배포는 예약 용량에 대해 보조 환경의 구성을 테스트하는 좋은 방법입니다.

> [!NOTE]
> 구독 할당량은 용량을 보장하지 않습니다. 할당량은 단순한 신용 한도입니다. 역할 수에 필요한 용량을 보장하려면 서비스 모델에서 정의되어야 하고 역할을 배포해야 합니다.
> 
> 

### <a name="load-balancing"></a>부하 분산
지역에 걸쳐 트래픽의 부하를 분산하려면 트래픽 관리 솔루션이 필요합니다. Azure는 [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/)를 제공합니다. 유사한 트래픽 관리 기능을 제공하는 타사 서비스의 장점도 활용할 수 있습니다.

### <a name="strategies"></a>전략
다양한 대체 전략은 지역에 걸쳐 분산된 계산을 구현하는 데 사용할 수 있습니다. 특정 비즈니스 요구 사항 및 응용 프로그램의 상황에 맞게 조정해야 합니다. 높은 수준에서 방법을 다음과 같은 범주로 나눌 수 있습니다.

* **재해 시 재배포**: 이 방식에서 응용 프로그램은 재해 시에 처음부터 재배포됩니다. 복구 시간이 보장되지 않아도 되는 중요하지 않은 응용 프로그램에 적합합니다.
* **웜 스페어(활성/수동)**: 보조 호스티드 서비스는 다른 지역에 만들어지고 역할은 최소 용량을 보장하도록 배포되지만 프로덕션 트래픽을 받지는 않습니다. 이 방법은 지역에 걸쳐 트래픽을 분산하기 위해 설계되지 않은 응용 프로그램에 유용합니다.
* **핫 스페어(활성/활성)**: 응용 프로그램이 여러 지역에서 프로덕션 부하를 수신하도록 설계되었습니다. 각 지역의 클라우드 서비스가 재해 복구 목적에 필요한 것보다 많은 용량에 구성될 수 있습니다. 또한 클라우드 서비스는 재해와 장애 조치 시 필요한 만큼 규모를 확장할 수 있습니다. 이 방법은 응용 프로그램 설계에 많은 투자가 필요하지만 상당한 이점이 있습니다. 즉, 짧은 복구 시간을 보장하고 모든 복구 위치를 연속으로 테스트하며 용량을 효율적으로 사용할 수 있습니다.

분산된 설계에 대한 자세한 내용은 이 문서에서 다루지 않습니다. 자세한 내용은 [Azure 응용 프로그램에 대한 재해 복구 및 고가용성](https://aka.ms/drtechguide)을 참조하세요.

## <a name="virtual-machines"></a>Virtual Machines
IaaS(infrastructure as a service) VM(가상 머신)의 복구는 여러 가지 면에서 PaaS(platform as a service) 계산 복구와 비슷합니다. 그러나 IaaS VM은 VM 및 VM 디스크 모두로 구성된다는 중요한 차이점이 있습니다.

* **Azure Backup을 사용하여 지역 백업에 일관된 응용 프로그램을 만듭니다**.
  [Azure Backup](https://azure.microsoft.com/services/backup/) 을 사용하면 고객이 여러 VM 디스크에 일관된 응용 프로그램을 만들고 지역에 백업을 복제하도록 지원할 수 있습니다. 생성 시 백업 자격 증명 모음을 지역에 복제하도록 선택하여 수행할 수 있습니다. 백업 자격 증명 모음의 복제를 생성 시 구성해야 합니다. 나중에 설정할 수 없습니다. 지역이 손실된 경우 Microsoft는 고객이 사용할 수 있는 백업을 만듭니다. 고객은 해당 구성된 복원 지점 중 하나를 복원할 수 있습니다.
* **운영 체제 디스크에서 데이터 디스크를 분리합니다**. IaaS VM의 경우 VM을 다시 만들지 않고 운영 체제 디스크를 변경할 수 없다는 점을 고려합니다. 복구 전략을 재해 후 재배포하는 경우 문제가 되지 않습니다. 그러나 예약 용량에 대해 웜 스페어 방법을 사용하는 경우 문제가 될 수 있습니다. 이를 제대로 구현하려면 올바른 운영 체제 디스크를 주 및 보조 위치에 배치하고 별도 드라이브에 응용 프로그램 데이터를 저장해야 합니다. 가능하면 두 위치에서 제공할 수 있는 표준 운영 체제 구성을 사용합니다. 장애 조치 후에 보조 DC의 기존 IaaS VM에 데이터 드라이브를 연결해야 합니다. AzCopy를 사용하여 원격 사이트에 데이터 디스크의 스냅숏을 복사합니다.
* **여러 VM 디스크의 지역 장애 조치 후에 발생할 수 있는 일관성 문제에 유의하세요**. VM 디스크는 Azure Storage Blob으로 구현되고 지역에서 복제 특성이 동일합니다. [Azure Backup](https://azure.microsoft.com/services/backup/) 을 사용하지 않는 한 지역에서 복제가 비동기적이며 독립적으로 복제되기 때문에 디스크에 걸쳐 일관성을 보장할 수 없습니다 . 개별 VM 디스크는 지역 장애 조치 후에 일관된 충돌 상태이지만 디스크에 걸쳐 일관되지 않습니다. 경우에 따라 문제가 발생할 수 있습니다(예: 디스크 스트라이프의 경우).

## <a name="storage"></a>Storage
### <a name="recovery-by-using-geo-redundant-storage-of-blob-table-queue-and-vm-disk-storage"></a>Blob, 테이블, 큐 및 VM 디스크 저장소의 지역 중복 저장소를 사용하여 복구
Azure에서 Blob, 테이블, 큐 및 VM 디스크는 기본적으로 모든 지역에서 복제됩니다. 지역 중복 저장소(GRS)라고 합니다. GRS는 특정 지역 내에서 수백 킬로미터 떨어져 있는 쌍을 이루는 데이터 센터에 저장소 데이터를 복제합니다. GRS는 주요 데이터 센터에 재해가 발생한 경우 추가 내구성을 제공하도록 설계되었습니다. Microsoft는 장애 조치가 발생하는 시점을 제어하고 장애 조치는 원래 주 위치가 적절한 시간에 복구할 수 없다고 여겨지는 경우 주요 재해로 제한됩니다. 일부 시나리오에서 몇 일이 걸릴 수 있습니다. 동기화 간격이 Service Level Agreement(서비스 수준 약정)에 아직 포함되지 않지만 데이터는 일반적으로 몇 분 안에 복제됩니다.

지역 장애 조치의 경우 계정이 액세스하는 방법은 변경되지 않습니다(URL 및 계정 키 변경되지 않음). 그러나 저장소 계정이 장애 조치 후에 다른 지역에 위치할 수 있습니다 해당 저장소 계정을 사용하여 지역 선호도를 필요로 하는 응용 프로그램에 영향을 줄 수 있습니다. 동일한 데이터 센터에 있는 저장소 계정이 필요하지 않은 서비스 및 응용 프로그램의 경우에도 데이터 센터 간 지연과 대역폭 요금은 트래픽을 장애 조치 지역에 일시적으로 이동하는 특별한 이유가 될 수 있습니다. 전체 재해 복구 전략에 요인이 될 수 있습니다.

GRS에서 제공하는 자동 장애 조치 외에도 Azure는 보조 저장소 위치에서 데이터의 복사본에 대한 읽기 액세스를 제공하는 서비스를 도입했습니다. 이를 RA-GRS(Read-Access Geo Redundant Storage)라고 합니다.

GRS 및 RA-GRS 저장소에 대한 자세한 내용은 [Azure Storage 복제](/azure/storage/storage-redundancy/)를 참조하세요.

### <a name="geo-replication-region-mappings"></a>지역에서 복제 지역 매핑:
저장소가 있는 지역 선호도를 필요로 하는 데이터의 다른 인스턴스를 배포하는 위치를 알기 위해 데이터를 지역으로 복제하는 위치를 알아야 합니다. 자세한 내용은 [Azure 쌍을 이루는 지역](/azure/best-practices-availability-paired-regions)을 참조하세요.

### <a name="geo-replication-pricing"></a>지역에서 복제 가격 책정:
지역에서 복제는 Azure Storage에 대한 현재 가격 책정에 포함됩니다. 이를 GRS(지역 중복 저장소)라고 합니다. 데이터 지역에서 복제하려고 하지 않는 경우 사용자 계정에 대한 지역에서 복제를 해제할 수 있습니다. 이를 로컬 중복 저장소라고 하고 GRS에 비해 할인된 가격으로 청구됩니다.

### <a name="determining-if-a-geo-failover-has-occurred"></a>지역 장애 조치가 발생했는지 여부를 결정합니다.
지역 장애 조치가 발생하는 경우 [Azure 서비스 상태 대시보드](https://azure.microsoft.com/status/)에 게시됩니다. 하지만 응용 프로그램은 해당 저장소 계정에 대한 지리적 지역을 모니터링하여 이를 감지하는 자동화된 방법을 구현할 수 있습니다. 해당 저장소로 이동하는 지역에서 계산 리소스의 정품 인증과 같은 다른 복구 작업을 트리거하는 데 사용될 수 있습니다. [Storage 계정 속성 가져오기](https://msdn.microsoft.com/library/ee460802.aspx)를 사용하여 Service Management API에서 이를 쿼리할 수 있습니다. 관련 속성은 다음과 같습니다.

    <GeoPrimaryRegion>primary-region</GeoPrimaryRegion>
    <StatusOfPrimary>[Available|Unavailable]</StatusOfPrimary>
    <LastGeoFailoverTime>DateTime</LastGeoFailoverTime>
    <GeoSecondaryRegion>secondary-region</GeoSecondaryRegion>
    <StatusOfSecondary>[Available|Unavailable]</StatusOfSecondary>

### <a name="vm-disks-and-geo-failover"></a>VM 디스크 및 지역 장애 조치
VM 디스크에 대한 섹션에서 설명한 대로 장애 조치 후에 VM 디스크에 걸쳐 데이터 일관성이 보장되지 않습니다. 백업의 정확성을 유지하려면 Data Protection Manager와 같은 백업 제품은 응용 프로그램 데이터를 백업하고 복원하는 데 사용되어야 합니다.

## <a name="database"></a>데이터베이스
### <a name="sql-database"></a>SQL Database
Azure SQL Database는 지역 복원 및 활성 지역 복제와 같은 두 가지 유형의 복구를 제공합니다.

#### <a name="geo-restore"></a>지역 복원
[지리적 복원](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) 은 기본, 표준 및 프리미엄 데이터베이스에서도 사용할 수 있습니다. 이 기능은 데이터베이스가 호스팅되는 지역에 사고가 발생하여 데이터베이스를 사용할 수 없게 되었을 때를 위한 기본 복구 옵션을 제공합니다. 특정 시점 복원과 유사하게, 지역 복원은 지역 중복 Azure 저장소의 데이터베이스 백업을 사용합니다. 지역에서 복제된 백업 복사본에서 복원하기 때문에 주 지역의 저장소 중단에 영향을 받지 않습니다. 자세한 내용은 [Azure SQL Database 복원 또는 보조 데이터베이스에 대한 장애 조치](/azure/sql-database/sql-database-disaster-recovery/)를 참조하세요.

#### <a name="active-geo-replication"></a>활성 지역 복제
[활성 지역 복제](/azure/sql-database/sql-database-geo-replication-overview/) 는 모든 데이터베이스 계층에서 사용할 수 있습니다. 이 기능은 지역 복원이 제공하는 것보다 더 까다로운 복구 요구 사항이 있는 응용 프로그램을 위해 설계되었습니다. 활성 지역 복제를 사용하여 서로 다른 지역의 서버에 최대 4개의 읽기 가능한 보조 복제본을 만들 수 있습니다. 보조 복제본으로 장애 조치를 시작할 수 있습니다. 또한 활성 지역 복제는 응용 프로그램 업그레이드 또는 재배치 시나리오뿐 아니라 읽기 전용 작업에 대한 부하 분산을 지원하는 데에도 사용할 수 있습니다. 자세한 내용은 [지역에서 복제 구성](/azure/sql-database/sql-database-geo-replication-portal/) 및 [보조 데이터베이스로 장애 조치(failover)](/azure/sql-database/sql-database-geo-replication-failover-portal/)를 참조하세요. 가동 중단 없는 응용 프로그램 및 응용 프로그램 업그레이드를 설계하고 구현하는 방법에 대한 자세한 내용은 [SQL Database에서 활성 지역 복제를 사용하여 클라우드 재해 복구를 위한 응용 프로그램 설계](/azure/sql-database/sql-database-designing-cloud-solutions-for-disaster-recovery/) 및 [SQL Database에서 활성 지역 복제를 사용하여 클라우드 재해 복구를 위한 응용 프로그램 설계](/azure/sql-database/sql-database-manage-application-rolling-upgrade/)를 참조하세요.

### <a name="sql-server-on-virtual-machines"></a>Virtual Machines의 SQL Server
Azure Virtual Machines에서 실행 중에 SQL Server 2012(또는 이상)에 대한 복구 및 고가용성에 다양한 옵션을 사용할 수 있습니다. 자세한 내용은 [Azure Virtual Machines의 SQL Server에 대한 고가용성 및 재해 복구](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/)를 참조하세요.

## <a name="other-azure-platform-services"></a>다른 Azure 플랫폼 서비스
여러 Azure 지역에서 클라우드 서비스를 실행하려고 시도할 경우 종속성 각각에 대한 영향을 고려해야 합니다. 다음 섹션에서 서비스 관련 지침에서는 대체 Azure 데이터 센터에서 동일한 Azure 서비스를 사용해야 한다고 가정합니다. 여기에서는 구성 및 데이터 복제 작업을 모두 포함합니다.

> [!NOTE]
> 경우에 따라 이러한 단계는 전체 데이터 센터 이벤트가 아닌 서비스 관련 중단을 완화시킬 수 있습니다. 응용 프로그램 관점에서 서비스 관련 중단은 제한과 마찬가지일 수 있으며 대체 Azure 지역에 서비스를 일시적으로 마이그레이션해야 할 수 있습니다.
> 
> 

### <a name="service-bus"></a>Service Bus
Azure Service Bus는 Azure 지역에 확장되지 않은 고유한 네임스페이스를 사용합니다. 따라서 첫 번째 요구 사항은 대체 지역에서 필요한 서비스 버스 네임스페이스를 설치하는 것입니다. 그러나 대기 중인 메시지의 지속성에 대한 고려 사항도 있습니다. Azure 지역에 메시지를 복제하기 위한 몇 가지 전략이 있습니다. 이러한 복제 전략 및 다른 재해 복구 전략에 대한 세부 정보는 [Service Bus 중단 및 재해로부터 응용 프로그램을 격리하는 모범 사례](/azure/service-bus-messaging/service-bus-outages-disasters/)를 참조하세요. 다른 가용성 고려 사항은 [Service Bus(가용성)](recovery-local-failures.md#other-azure-platform-services)을 참조하세요.

### <a name="app-service"></a>App Service
Web Apps 또는 Mobile Apps와 같은 Azure App Service 응용 프로그램을 보조 Azure 지역에 마이그레이션하려면 게시에 사용할 수 있는 웹 사이트의 백업이 있어야 합니다. 가동 중단이 전체 Azure 데이터 센터를 포함하지 않은 경우 FTP를 사용하여 사이트 콘텐츠의 최근 백업을 다운로드할 수 있습니다. 그런 다음 이전에 예약 용량에 대해 이렇게 수행하지 않았으면 대체 지역에서 새 앱을 만듭니다. 새 지역에 사이트를 게시하고 필요한 구성을 변경합니다. 이러한 변경 내용은 데이터베이스 연결 문자열이나 기타 지역 관련 설정을 포함할 수 있습니다. 필요한 경우 사이트의 SSL 인증서를 추가하고 DNS CNAME 레코드를 변경하여 재배포된 Azure 웹앱 URL에 사용자 지정 도메인 이름을 가리키도록 합니다.

### <a name="hdinsight"></a>HDInsight
HDInsight와 연결된 데이터는 기본적으로 Azure Blob Storage에 저장됩니다. HDInsight은 MapReduce 작업을 처리하는 Hadoop 클러스터가 분석 중인 데이터를 포함하는 저장소 계정과 동일한 지역에 배치되도록 해야 합니다. Azure Storage에 사용할 수 있는 지역에서 복제 기능이 제공되면 어떤 이유로든 주 지역을 사용할 수 없는 경우 데이터를 복제하는 보조 지역의 데이터에 액세스할 수 있습니다. 데이터가 복제되고 처리를 계속하는 지역에 새로운 Hadoop 클러스터를 만들 수 있습니다. 다른 가용성 고려 사항은 [HDInsight(가용성)](recovery-local-failures.md#other-azure-platform-services)을 참조하세요.

### <a name="sql-reporting"></a>SQL Reporting
이 경우에 Azure 지역의 손실로부터 복구하려면 다른 Azure 지역에서 여러 SQL Reporting 인스턴스가 필요합니다. 이러한 SQL Reporting 인스턴스는 동일한 데이터에 액세스해야 하고 해당 데이터는 재해가 발생 시 고유한 복구 계획이 있어야 합니다. 또한 각 보고서에 대한 RDL 파일의 외부 백업 복사본을 유지할 수 있습니다.

### <a name="media-services"></a>Media Services
Azure Media Services에는 인코딩 및 스트리밍을 위한 다른 복구 방법이 있습니다. 일반적으로 지역 가동 중단 중에 스트리밍은 더 중요합니다. 이를 대비하려면 두 개의 다른 Azure 지역에 Media Services 계정이 있어야 합니다. 인코딩된 콘텐츠는 두 지역에 있어야 합니다. 오류 발생 중에 대체 지역에 스트리밍 트래픽을 리디렉션할 수 있습니다. Encoding은 모든 Azure 지역에서 수행될 수 있습니다. 예를 들어 라이브 이벤트 처리 중 인코딩이 시간에 민감한 경우 오류 발생 중에 대체 데이터 센터에 작업을 제출하도록 준비해야 합니다.

### <a name="virtual-network"></a>가상 네트워크
구성 파일은 대체 Azure 지역에서 가상 네트워크를 설정하는 가장 빠른 방법을 제공합니다. 주 Azure 지역에서 가상 네트워크를 구성한 후에 네트워크 구성 파일에 현재 네트워크에 대한 [가상 네트워크 설정을 내보냅니다](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) . 주 지역에서 가동 중단 시 저장된 구성 파일에서 [가상 네트워크를 복원합니다](/azure/virtual-network/virtual-networks-create-vnet-classic-portal/) . 다른 클라우드 서비스, 가상 머신 또는 프레미스 간 설정을 구성하여 새 가상 네트워크로 작동하도록 합니다.

## <a name="checklists-for-disaster-recovery"></a>재해 복구를 위한 검사 목록
## <a name="cloud-services-checklist"></a>Cloud Services 검사 목록
1. 이 문서의 Cloud Services 섹션을 검토합니다.
2. 지역 간 재해 복구 전략을 만듭니다.
3. 대체 지역에서 용량 예약의 장단점을 이해합니다.
4. Azure Traffic Manager와 같은 트래픽 라우팅 도구를 사용합니다.

## <a name="virtual-machines-checklist"></a>Virtual Machines 검사 목록
1. 이 문서의 Virtual Machines 섹션을 검토합니다.
2. [Azure Backup](https://azure.microsoft.com/services/backup/)을 사용하여 지역에 걸쳐 응용 프로그램에 일관된 백업을 만듭니다.

## <a name="storage-checklist"></a>저장소 검사 목록
1. 이 문서의 저장소 섹션을 검토합니다.
2. 저장소 리소스의 지역에서 복제를 해제하지 않습니다.
3. 장애 조치 시 지역에서 복제에 대한 대체 지역을 이해합니다.
4. 사용자 제어 장애 조치 전략에 대한 사용자 지정 백업 전략을 만듭니다.

## <a name="sql-database-checklist"></a>SQL Database 검사 목록
1. 이 문서의 SQL Database 섹션을 검토합니다.
2. [지역 복원](/azure/sql-database/sql-database-recovery-using-backups/#geo-restore) 또는 [지역에서 복제](/azure/sql-database/sql-database-geo-replication-overview/)를 적절하게 사용합니다.

## <a name="sql-server-on-virtual-machines-checklist"></a>Virtual Machines 검사 목록의 SQL Server
1. 이 문서의 Virtual Machines에서 SQL Server 섹션을 검토합니다.
2. 지역 간 AlwaysOn 가용성 그룹 또는 데이터베이스 미러링을 사용합니다.
3. 또는 백업 사용 및 Blob 저장소에 복원합니다.

## <a name="service-bus-checklist"></a>Service Bus 검사 목록
1. 이 문서의 Service Bus 섹션을 검토합니다.
2. 대체 지역에서 Service Bus 네임스페이스를 구성합니다.
3. 지역에 걸쳐 메시지에 대한 사용자 지정 복제 전략을 고려합니다.

## <a name="app-service-checklist"></a>App Service 검사 목록
1. 이 문서의 App Service 섹션을 검토합니다.
2. 주 지역 외부에서 사이트 백업을 유지 관리합니다.
3. 가동 중단이 부분적인 경우 FTP로 현재 사이트 검색을 시도합니다.
4. 대체 지역에서 기존 또는 새 사이트에 사이트를 배포할 계획입니다.
5. 응용 프로그램 및 DNS CNAME 레코드 모두에 대한 구성을 변경할 계획입니다.

## <a name="hdinsight-checklist"></a>HDInsight 검사 목록
1. 이 문서의 HDInsight 섹션을 검토합니다.
2. 복제된 데이터를 사용하여 지역에서 새로운 Hadoop 클러스터를 만듭니다.

## <a name="sql-reporting-checklist"></a>SQL Reporting 검사 목록
1. 이 문서의 SQL Reporting 섹션을 검토합니다.
2. 다른 지역에 대체 SQL Reporting 인스턴스를 유지 관리합니다.
3. 별도 계획을 유지 관리하여 해당 지역에 대상을 복제합니다.

## <a name="media-services-checklist"></a>Media Services 검사 목록
1. 이 문서의 Media Services 섹션을 검토합니다.
2. 대체 지역에서 Media Services 계정을 만듭니다.
3. 두 지역에서 동일한 콘텐츠를 인코딩하여 스트리밍 장애 조치를 지원합니다.
4. 서비스 중단 시 대체 지역에 인코딩 작업을 제출합니다.

## <a name="virtual-network-checklist"></a>Virtual Network 검사 목록
1. 이 문서의 Virtual Network 섹션을 검토합니다.
2. 가상 네트워크 설정을 사용하여 다른 지역에 다시 만듭니다.

