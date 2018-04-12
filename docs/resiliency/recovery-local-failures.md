---
title: '기술 지침: Azure에서 로컬 오류 복구'
description: Azure 내에서 로컬 오류에 중점을 둔 재해 복구에 대한 계획 뿐만 아니라 복원력 있고 항상 사용 가능한 내결함성 응용 프로그램을 이해하고 설계하는 방법에 대한 문서입니다.
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 5fc929bd1affe3dd6616f908bae0e7d2fefb89d5
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-local-failures-in-azure"></a>Azure 복원력 기술 지침 - Azure의 로컬 오류로부터 복구

응용 프로그램에 대한 다음과 같은 두 가지 주요 위협 요인이 있습니다.

* 드라이브 및 서버와 같은 장치의 오류
* 최대 부하 조건에서 계산 등 중요한 리소스 소모

Azure는 리소스 관리, 유연성, 부하 분산 및 분할을 조합하여 이러한 상황에서 가용성을 높입니다. 이러한 기능 중 일부는 모든 Azure 서비스에서 자동으로 수행됩니다. 그러나 경우에 따라 응용 프로그램 개발자는 몇 가지 추가 작업을 수행해야만 혜택을 얻을 수 있습니다.

## <a name="cloud-services"></a>Cloud Services
Azure Cloud Services는 하나 이상의 웹 또는 작업자 역할의 컬렉션으로 구성됩니다. 하나 이상인 역할의 인스턴스를 동시에 실행할 수 있습니다. 구성은 인스턴스 수를 결정합니다. 역할 인스턴스는 패브릭 컨트롤러라는 구성 요소를 통해 모니터링되고 관리됩니다. 패브릭 컨트롤러는 자동으로 소프트웨어와 하드웨어 오류를 감지하고 대응합니다.

모든 역할 인스턴스는 고유한 VM(가상 컴퓨터)에서 실행되고 게스트 에이전트를 통해 패브릭 컨트롤러와 통신합니다. 게스트 에이전트는 VM 사용, 상태, 로그, 리소스 사용, 예외 및 오류 상태를 포함하는 리소스 및 노드 메트릭을 수집합니다. 패브릭 컨트롤러는 구성 가능한 간격의 게스트 에이전트를 쿼리하고 게스트 에이전트가 응답하지 못하는 경우 VM을 다시 시작합니다. 하드웨어 오류가 발생한 경우 연결된 패브릭 컨트롤러는 영향을 받은 모든 역할 인스턴스를 새 하드웨어 노드로 이동하고 거기서 트래픽을 라우팅하도록 네트워크를 다시 구성합니다.

이러한 기능을 제대로 활용하려면 개발자는 모든 서비스 역할이 역할 인스턴스에 상태를 저장하지 않도록 해야 합니다. 대신, 모든 영구 데이터는 Azure Storage 또는 Azure SQL Database와 같은 지속성 저장소에서 액세스되어야 합니다. 이렇게 하면 역할이 요청을 처리할 수 있습니다. 즉, 역할 인스턴스가 서비스의 임시 상태나 영구 상태에서 불일치를 만들지 않고 언제든지 중단될 수 있습니다.

역할 외부에 상태를 저장하라는 요구 사항은 여러 가지를 의미합니다. 예를 들어 가능한 경우 단일 엔터티 그룹 트랜잭션에서 Azure Storage 테이블에 대한 모든 관련된 변경 사항을 변경해야 합니다. 물론 단일 트랜잭션에서 모든 사항을 변경할 수 있는 것은 아닙니다. 서비스의 영구 상태에 대한 두 개 이상의 업데이트에 걸쳐 있는 장기 실행 작업을 중단시킬 경우 역할 인스턴스 오류가 문제를 발생시키지 않도록 특별히 주의해야 합니다. 다른 역할이 이러한 작업을 다시 시도하려면 작업이 부분적으로 완료된 경우 이를 처리해야 합니다.

예를 들어 여러 저장소에 데이터를 분할하는 서비스를 사용하도록 고려합니다. 작업자 역할이 분할된 데이터베이스를 재배치하는 동안 중단되더라도 분할된 데이터베이스의 재배치는 종료되지 않습니다. 또는 재배치가 다른 작업자 역할에 의해 처음부터 반복될 수 있습니다. 이는 잠재적으로 분리된 데이터 또는 데이터 손상을 초래합니다. 문제를 방지하려면 장기 실행 작업은 다음 중 하나 또는 모두여야 합니다.

* *Idempotent*: 부작용 없이 반복 가능합니다. Idempotent이려면 장기 실행 작업은 실행하는 동안 중단되는 경우더라도 실행되는 횟수에 관계 없이 동일한 효과를 가져야 합니다.
* *증분 방식으로 다시 시작 가능*: 가장 최근의 오류 지점에서 계속할 수 있습니다. 증분 방식으로 다시 시작 가능하려면 장기 실행 작업은 보다 작은 원자성 작업의 시퀀스로 구성되어야 합니다. 또한 각 후속 호출이 이전 버전에서 중지된 위치를 선택하도록 해당 진행 상태를 지속성 저장소에 기록해야 합니다.

마지막으로 모든 장기 실행 작업은 성공할 때까지 반복적으로 호출되어야 합니다. 예를 들어 프로비전 작업은 Azure 큐에 배치되고 성공한 경우에 작업자 역할에 의해 큐에서 제거될 수 있습니다. 가비지 수집은 중단된 작업이 만든 데이터를 정리해야 합니다.

### <a name="elasticity"></a>탄력성
각 역할에 실행되는 인스턴스의 초기 수는 각 역할의 구성에서 결정됩니다. 관리자는 역할 각각을 처음에 구성하여 예상된 부하에 따라 두 개 이상의 인스턴스를 사용하여 실행해야 합니다. 그러나 역할 인스턴스를 사용량 패턴 변경으로 쉽게 확장하거나 축소할 수 있습니다. Azure 포털에서 수동으로 수행할 수 있고 Windows PowerShell, Service Management API 또는 타사 도구를 사용하여 프로세스를 자동화할 수 있습니다. 자세한 내용은 [응용 프로그램 크기를 자동 조정하는 방법](/azure/cloud-services/cloud-services-how-to-scale/)을 참조하세요.

### <a name="partitioning"></a>분할
Azure 패브릭 컨트롤러는 두 가지 유형의 파티션을 사용합니다.

* 그룹에서 서비스의 역할 인스턴스를 업그레이드하는 데 *업데이트 도메인* 을 사용합니다. Azure는 여러 개의 업데이트 도메인에 서비스 인스턴스를 배포합니다. 전체 업데이트의 경우 패브릭 컨트롤러는 하나의 업데이트 도메인에 있는 모든 인스턴스를 가져와서 업데이트한 후에 다음 업데이트 도메인으로 이동하기 전에 다시 시작합니다. 이 방법은 업데이트 과정에서 전체 서비스를 사용할 수 있도록 합니다.
* *장애 도메인* 은 하드웨어 또는 네트워크 오류의 잠재적인 지점을 정의합니다. 둘 이상의 인스턴스가 있는 역할의 경우 패브릭 컨트롤러는 분리된 하드웨어 오류가 서비스를 중단하지 않도록 방지하기 위해 인스턴스가 여러 장애 도메인에 걸쳐 분산되도록 합니다. 장애 도메인은 서버와 클러스터 오류에 대한 노출을 모두 제어합니다.

둘 이상의 웹 역할 인스턴스를 여러 장애 도메인 및 업그레이드 도메인에 배포하는 경우 Microsoft는 [Azure SLA(서비스 수준 계약)](https://azure.microsoft.com/support/legal/sla/) 는 적어도 99.95% 이상 외부와 연결되어 있습니다. 업데이트 도메인과 달리 장애 도메인의 수를 제어할 방법이 없습니다. Azure는 자동으로 장애 도메인을 할당하고 이들 간에 역할 인스턴스를 배포합니다. 적어도 두 개의 인스턴스로 역할이 SLA를 충족하도록 하려면 적어도 모든 역할에서 처음 두 개의 인스턴스를 다른 장애 도메인 및 업그레이드 도메인에 배치합니다. 이 내용은 다음 다이어그램에서 나타납니다.

![장애 도메인 격리의 단순화된 보기](./images/technical-guidance-recovery-local-failures/partitioning-1.png)

### <a name="load-balancing"></a>부하 분산
웹 역할에 대한 인바운드 트래픽은 모두 상태 비저장 부하 분산 장치를 통해 전달되며 역할 인스턴스에 클라이언트 요청을 분산합니다. 개별 역할 인스턴스에는 공용 IP 주소가 없으며 인터넷에서 직접 주소를 지정할 수 없습니다. 웹 역할은 상태 비저장이므로 클라이언트 요청을 어떠한 역할 인스턴스로도 라우팅할 수 있습니다. [StatusCheck](https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleenvironment.statuscheck.aspx) 이벤트는 15초마다 발생합니다. 역할이 트래픽을 받을 준비가 되거나 사용 중이어서 부하 분산 장치 순환에서 제거되어야 함을 나타내는 데 사용할 수 있습니다.

## <a name="virtual-machines"></a>Virtual Machines
Azure Virtual Machines는 고가용성과 관련된 몇 가지 측면에서 PaaS(platform as a service) 계산 역할과 다릅니다. 일부 인스턴스에서 고가용성을 보장하기 위해 추가적인 작업을 수행해야 합니다.

### <a name="disk-durability"></a>디스크 내구성
PaaS 역할 인스턴스와 달리 가상 머신이 재배치되는 경우 가상 머신 드라이브에 저장된 데이터는 영구적입니다. Azure 가상 머신은 Azure Storage에서 Blob으로 존재하는 VM 디스크를 사용합니다. 또한 Azure Storage의 가용성 특성 때문에 가상 머신의 드라이브에 저장된 데이터를 항상 사용 가능합니다.

D 드라이브(Windows VM)는 이 규칙의 예외입니다. D 드라이브는 VM을 호스팅하는 랙 서버에 실제 물리적 저장소이며 VM을 재활용하는 경우 해당 데이터가 손실됩니다. D 드라이브는 임시 저장소로만 사용됩니다. Linux에서 Azure는 "일반적으로"(항상 그렇지는 않음) 로컬 임시 디스크를 /dev/sdb 블록 장치로 노출합니다. Azure Linux 에이전트에 의해 /mnt/resource 또는 /mnt 탑재 지점으로 탑재되는 경우가 많습니다(/etc/waagent.conf를 통해 구성 가능).

### <a name="partitioning"></a>분할
Azure는 PaaS 응용 프로그램(웹 역할 및 작업자 역할)의 계층을 기본적으로 이해하므로 장애 도메인 및 업데이트 도메인에 걸쳐 제대로 배포될 수 있습니다. 반면에 IaaS(infrastructure as a service) 응용 프로그램의 계층은 가용성 집합을 통해 수동으로 정의되어야 합니다. 가용성 집합은 IaaS에서 SLA에 필요합니다.

![Azure 가상 머신에 대한 가용성 집합](./images/technical-guidance-recovery-local-failures/partitioning-2.png)

위의 다이어그램에서 웹앱 계층으로 작동하는 IIS(인터넷 정보 서비스) 계층 및 데이터 계층으로 작동하는 SQL 계층은 서로 다른 가용성 집합에 할당됩니다. 이렇게 하면 각 계층의 모든 인스턴스가 장애 도메인에 가상 머신을 분산하여 하드웨어 중복성을 가질 수 있고 업데이트하는 동안 전체 계층의 작동이 중단되지 않습니다.

### <a name="load-balancing"></a>부하 분산
VM에 트래픽을 분산해야 하는 경우 응용 프로그램에서 VM을 그룹화하고 특정 TCP 또는 UDP 끝점에서 부하를 분산해야 합니다. 자세한 내용은 [가상 머신 부하 분산](/azure/virtual-machines/virtual-machines-linux-load-balance/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)(영문)을 참조하십시오. VM이 다른 원본(예: 큐 메커니즘)에서 입력을 수신하는 경우 부하 분산 장치가 필요하지 않습니다. 부하 분산 장치는 기본적인 상태 검사를 사용하여 트래픽이 노드로 전송되어야 하는지를 결정합니다. 또한 고유한 프로브를 만들어 VM 트래픽을 수신해야 하는지를 결정하는 응용 프로그램 관련 상태 메트릭을 구현할 수 있습니다.

## <a name="storage"></a>Storage
Azure Storage는 Azure에 대한 지속 가능한 기본 데이터 서비스입니다. Blob, 테이블, 큐 및 VM 디스크 저장소를 제공합니다. 복제 및 리소스 관리의 조합을 사용하여 단일 데이터 센터 내에서 고가용성을 제공합니다. Azure Storage 가용성 SLA는 최소한 99.9%의 시간을 보장합니다.

* 데이터를 추가, 업데이트, 읽기 및 삭제할 올바른 형식이 지정된 요청이 성공적이고 올바르게 처리됩니다.
* Storage 계정은 인터넷 게이트웨이에 연결됩니다.

### <a name="replication"></a>복제
Azure Storage는 지역 내에서 완전히 독립된 물리적 저장소 하위 시스템에서 여러 드라이브에 있는 모든 데이터의 여러 복사본을 유지 관리하여 데이터 내구성을 촉진합니다. 데이터를 동기적으로 복제하고 쓰기를 승인하기 전에 모든 복사본을 커밋합니다. Azure Storage는 강력한 일관성을 유지하여 읽기가 가장 최근의 쓰기를 반영하도록 합니다. 또한 데이터의 복사본을 지속적으로 스캔하여 저장된 데이터의 무결성에 위협이 되지만 흔히 간과하는 비트 손상을 탐색하고 복구합니다.

서비스는 Azure Storage를 사용하여 복제를 활용합니다. 서비스 개발자는 로컬 오류로부터 복구하기 위해 추가 작업을 수행할 필요가 없습니다.

### <a name="resource-management"></a>리소스 관리
2014년 5월 이후에 만들어진 Storage 계정은 최대 500TB까지 가능합니다(이전 최대 크기는 200TB). 추가 공간이 필요한 경우 응용 프로그램이 여러 저장소 계정을 사용하도록 설계해야 합니다.

### <a name="virtual-machine-disks"></a>가상 머신 디스크
가상 머신의 VM 디스크는 Azure Storage에서 페이지 Blob으로 저장되고 Blob Storage와 동일한 내구성 및 확장성 속성을 제공합니다. 이 설계를 통해 VM을 실행하는 서버가 실패하고 다른 서버에서 VM이 다시 시작해야 하더라도 가상 머신의 디스크에 있는 데이터를 영구적으로 만듭니다.

## <a name="database"></a>데이터베이스
### <a name="sql-database"></a>SQL Database
Azure SQL Database는 DaaS(database as a service)를 제공합니다. 응용 프로그램을을 신속하게 프로비전하고 데이터를 삽입하며 관계형 데이터베이스를 쿼리할 수 있습니다. 친숙한 여러 SQL Server 기능을 제공하는 동시에 하드웨어, 구성, 패치 및 복구의 부담을 줄였습니다.

> [!NOTE]
> Azure SQL Database는 SQL Server와 일대일 기능 패리티를 제공하지 않습니다. 클라우드 응용 프로그램에 고유하게 맞춘 일련의 요구 사항을 만족하도록 계획되었습니다(유지 관리 비용을 절감하는 탄력적인 확장, DaaS(Database-as-a-Service)). 자세한 내용은 [클라우드 SQL Server 옵션 선택: Azure SQL(PaaS) 데이터베이스 또는 Azure VM(IaaS)의 SQL Server](/azure/sql-database/sql-database-paas-vs-sql-server-iaas/)를 참조하세요.
> 
> 

#### <a name="replication"></a>복제
Azure SQL Database는 노드 수준 오류에 대한 기본 제공 탄력성을 제공합니다. 데이터베이스에 대한 모든 쓰기는 쿼럼 커밋 기술을 통해 두 개 이상의 배경 노드에 자동으로 복제됩니다. (트랜잭션이 성공하고 반환되려먼 먼저 기본 데이터베이스와 하나 이상의 보조 복제본에서 트랜잭션 로그에 작업이 기록되었음을 확인해야 합니다.) 노드 오류가 발생할 경우 데이터베이스는 자동으로 보조 복제본 중 하나에 장애 조치합니다. 이로 인해 클라이언트 응용 프로그램에 대한 임시 연결 중단이 발생합니다. 이러한 이유로 Azure SQL Database 클라이언트는 모두 어떤 형태의 임시 연결 오류 처리를 구현해야 합니다. 자세한 내용은 [서비스 관련 재시도 지침](/azure/best-practices-retry-service-specific/)을 참조하세요.

#### <a name="resource-management"></a>리소스 관리
각 데이터베이스를 만들 경우 최대 크기 제한으로 구성됩니다. 현재 사용할 수 있는 최대 크기는 1TB(크기 제한은 서비스 계층에 따라 다름)입니다. [서비스 계층 및 Azure SQL Databases의 성능 수준](/azure/sql-database/sql-database-resource-limits/#service-tiers-and-performance-levels)을 참조하세요. 데이터베이스가 최대 크기 제한에 도달하면 추가 INSERT 또는 UPDATE 명령을 거부합니다. (데이터 쿼리 및 삭제는 여전히 가능합니다.)

데이터베이스 내에서 Azure SQL Database는 패브릭을 사용하여 리소스를 관리합니다. 그러나 패브릭 컨트롤러 대신 링 토폴로지를 사용하여 오류를 감지합니다. 클러스터에 있는 모든 복제본에는 두 개의 인접 항목이 있으며 중단되었을 경우 감지할 책임이 있습니다. 복제본이 중단되면 인접 항목은 재구성 에이전트를 트리거하여 다른 컴퓨터에서 다시 만듭니다. 논리 서버가 컴퓨터에서 너무 많은 리소스를 사용하거나 컴퓨터의 물리적 제한을 초과하지 않도록 엔진 제한을 제공합니다.

### <a name="elasticity"></a>탄력성
응용 프로그램이 제한인 1TB 데이터베이스보다 더 많은 데이터베이스가 필요한 경우 규모 확장 방식을 구현해야 합니다. 여러 SQL Database 간에 데이터를 수동으로 분할하여(분할된 데이터라고도 함) Azure SQL Database의 규모를 확장합니다. 이 규모 확장 방법은 눈금으로 가까운 선형 비용 증가를 달성할 기회를 제공합니다. 데이터베이스는 최대 크기가 아닌 하루에 사용되는 실제 평균 크기에 따라 요금이 청구되기 때문에 필요에 따라 비용이 증가하면서 요청되는 탄력적인 성장이나 용량도 증가할 수 있습니다.

## <a name="sql-server-on-virtual-machines"></a>Virtual Machines의 SQL Server
SQL Server(버전 2014 이상)를 Azure Virtual Machines에 설치하여 SQL Server의 기존 가용성 기능을 이용할 수 있습니다. 이러한 기능에는 AlwaysOn 가용성 그룹 및 데이터베이스 미러링이 포함됩니다. Azure VM, 저장소 및 네트워킹은 온-프레미스, 가상화되지 않은 IT 인프라에서는 작동 특성이 달라집니다. Azure에서 고가용성/재해 복구(HA/DR) SQL Server 솔루션을 성공적으로 구현하기 위해서는 이러한 차이점을 이해하고 그에 맞게 솔루션을 설계해야 합니다.

### <a name="high-availability-nodes-in-an-availability-set"></a>가용성 집합의 고가용성 노드
Azure의 고가용성 솔루션을 구현하는 경우 Azure의 가용성 집합을 사용하여 고가용성 노드를 별도의 장애 도메인과 업그레이드 도메인에 배치할 수 있습니다. 분명히 하자면 가용성 집합은 Azure 개념입니다. AlwaysOn 가용성 그룹, 미러링 또는 다른 기능을 사용하는 것과 상관 없이 데이터베이스가 실제로 항상 사용 가능한지 확인하기 위해 따라야 하는 모범 사례입니다. 이 모범 사례를 따르지 않으면 시스템을 항상 사용할 수 있다고 잘못 생각할 수 있습니다. 하지만 Azure 지역에서 동일한 장애 도메인에 배치되었기 때문에 실제로 노드가 모두 동시에 실패할 수 있습니다.

이 권장 사항은 로그 전달처럼 적용될 수 없습니다. 따라서 재해 복구 기능으로 서버가 별도 Azure 지역에서 실행되도록 해야 합니다. 기본적으로 이러한 지역은 별도의 장애 도메인입니다.

클래식 포털을 통해 배포되는 Azure Cloud Services가 동일한 가용성 집합에 포함되려면 동일한 클라우드 서비스에 배포해야 합니다. Azure Resource Manager(현재 포털)을 통해 배포된 VM에는 이러한 제한이 적용되지 않습니다. Azure 클라우드 서비스의 클래식 포털 배포 VM의 경우 같은 클라우드 서비스에 있는 노드만 같은 가용성 집합에 참여할 수 있습니다. 또한 Cloud Services VM은 서비스 복구 후에 해당 IP를 유지하는지 확인하기 위해 동일한 가상 네트워크에 위치해야 합니다. 그러면 DNS 업데이트 중단을 피할 수 있습니다.

### <a name="azure-only-high-availability-solutions"></a>Azure 전용: 고가용성 솔루션
AlwaysOn 가용성 그룹 또는 데이터베이스 미러링을 사용하여 Azure에서 SQL Server 데이터베이스를 위한 고가용성 솔루션을 구현할 수 있습니다.

다음 다이어그램에서는 Azure Virtual Machines에서 실행되는 AlwaysOn 가용성 그룹의 아키텍처를 보여 줍니다. 이 다이어그램은 [Azure Virtual Machines에서 SQL Server에 대한 고가용성 및 재해 복구](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/)라는 제목의 심층 문서에서 가져왔습니다.

![Microsoft Azure의 AlwaysOn 가용성 그룹](./images/technical-guidance-recovery-local-failures/high_availability_solutions-1.png)

Azure 포털에서 AlwaysOn 템플릿을 사용하여 Azure VM에 AlwaysOn 가용성 그룹 배포 종단 간을 자동으로 프로비전할 수 있습니다. 자세한 내용은 [Microsoft Azure Portal 갤러리의 SQL Server AlwaysOn 제품](https://blogs.technet.microsoft.com/dataplatforminsider/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery/)을 참조하세요.

다음 다이어그램은 Azure Virtual Machines에서 데이터베이스 미러링의 사용법을 보여 줍니다. [Azure Virtual Machines에서 SQL Server에 대한 고가용성 및 재해 복구](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/)심층 항목에서 가져왔습니다.

![Microsoft Azure의 데이터베이스 미러링](./images/technical-guidance-recovery-local-failures/high_availability_solutions-2.png)

> [!NOTE]
> 두 아키텍처는 모두 도메인 컨트롤러가 필요합니다. 그러나 데이터베이스 미러링으로 서버 인증서를 사용하여 도메인 컨트롤러에 대한 필요성을 제거할 수 있습니다.
> 
> 

## <a name="other-azure-platform-services"></a>다른 Azure 플랫폼 서비스
Azure를 기반으로 하는 응용 프로그램은 플랫폼 기능을 활용하여 로컬 오류로부터 복구합니다. 경우에 따라 특정 시나리오에 대한 가용성을 높이기 위해 특정 작업을 수행할 수 있습니다.

### <a name="service-bus"></a>Service Bus
Azure Service Bus의 일시적 중단을 줄이려면 내구성이 있는 클라이언트 쪽 큐를 만드는 것이 좋습니다. 일시적으로 대체 로컬 저장소 메커니즘을 사용하여 Service Bus 큐에 추가할 수 없는 메시지를 저장합니다. 응용 프로그램은 서비스가 복원된 후에 일시적으로 저장된 메시지를 처리하는 방법을 결정할 수 있습니다. 자세한 내용은 [Service Bus 조정된 메시징을 사용한 성능 향상의 모범 사례](/azure/service-bus-messaging/service-bus-performance-improvements/) 및 [Service Bus(재해 복구)](recovery-loss-azure-region.md#other-azure-platform-services)를 참조하세요.

### <a name="hdinsight"></a>HDInsight
Azure HDInsight와 연결된 데이터는 기본적으로 Azure Blob 저장소에 저장됩니다. Azure Storage는 Blob Storage에 고가용성 및 내구성 속성을 지정합니다. Hadoop MapReduce 작업과 연결된 다중 노드는 HDInsight에서 필요할 경우 프로비전되는 일시적인 HDFS(Hadoop 분산 파일 시스템)에서 처리됩니다. 또한 MapReduce 작업의 결과가 기본적으로 Azure Blob 저장소에 저장되므로 처리된 데이터는 Hadoop 클러스터의 프로비전을 해제한 후에 내구성 있고 항상 사용 가능한 상태로 유지합니다. 자세한 내용은 [HDInsight(재해 복구)](recovery-loss-azure-region.md#other-azure-platform-services)를 참조하세요.

## <a name="checklists-for-local-failures"></a>로컬 오류의 검사 목록
### <a name="cloud-services"></a>Cloud Services
1. 이 문서의 Cloud Services 섹션을 검토합니다.
2. 각 역할에 두 개 이상의 인스턴스를 구성합니다.
3. 역할 인스턴스가 아니라 내구성 저장소에서 상태를 유지합니다.
4. StatusCheck 이벤트를 올바르게 처리합니다.
5. 가능한 경우 트랜잭션에서 관련된 변경 내용을 래핑합니다.
6. 작업자 역할 작업이 idempotent 또는 다시 시작 가능한지 확인합니다.
7. 작업 호출이 성공할 때까지 계속합니다.
8. 자동 크기 조정 전략을 고려합니다.

### <a name="virtual-machines"></a>Virtual Machines
1. 이 문서의 Virtual Machines 섹션을 검토합니다.
2. 영구 저장소에 D 드라이브를 사용하지 않습니다.
3. 가용성 집합에 서비스 계층의 컴퓨터를 그룹화합니다.
4. 부하 분산 및 선택적 프로브를 구성합니다.

### <a name="storage"></a>Storage
1. 이 문서의 저장소 섹션을 검토합니다.
2. 데이터 또는 대역폭이 할당량을 초과하는 경우 여러 저장소 계정을 사용합니다.

### <a name="sql-database"></a>SQL Database
1. 이 문서의 SQL Database 섹션을 검토합니다.
2. 재시도 정책을 구현하여 임시 오류를 처리합니다.
3. 분할을 규모 확장 전략으로 사용합니다.

### <a name="sql-server-on-virtual-machines"></a>Virtual Machines의 SQL Server
1. 이 문서의 Virtual Machines에서 SQL Server 섹션을 검토합니다.
2. Virtual Machines에 대한 이전의 권장 사항을 수행합니다.
3. AlwaysOn과 같은 SQL Server 고가용성 기능을 사용합니다.

### <a name="service-bus"></a>Service Bus
1. 이 문서의 Service Bus 섹션을 검토합니다.
2. 백업으로 내구성이 있는 클라이언트 쪽 큐를 만드는 것이 좋습니다.

### <a name="hdinsight"></a>HDInsight
1. 이 문서의 HDInsight 섹션을 검토합니다.
2. 로컬 오류에 필요한 추가 가용성 단계가 없습니다.

