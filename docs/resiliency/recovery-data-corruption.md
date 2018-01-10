---
title: "데이터 손상 또는 우발적 삭제로부터 복구"
description: "데이터 손상 또는 실수로 인한 데이터 삭제로부터 데이터를 복구하는 방법을 이해하고 재해 복구에 대한 계획 뿐만 아니라 복원력 있고 항상 사용 가능한 내결함성 응용 프로그램을 설계하는 방법에 대한 문서입니다."
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: b75c774f85c42f64472167897f08a7302ab50a3f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-data-corruption-or-accidental-deletion"></a>Azure 복원력 기술 지침: 데이터 손상 또는 삭제 실수로부터 복구
강력한 비즈니스 연속성 계획의 일부로 손상되거나 실수로 삭제된 데이터에 대한 계획을 수립합니다. 다음은 데이터가 응용 프로그램 오류 또는 운영자 오류로 인해 손상되거나 실수로 삭제된 후에 복구하는 방법에 대한 정보입니다.

## <a name="virtual-machines"></a>Virtual Machines
Azure Virtual Machines(IaaS(infrastructure-as-a-service) VM이라고도 함)를 응용 프로그램 오류 또는 삭제 실수로부터 보호하려면 [Azure Backup](https://azure.microsoft.com/services/backup/)을 사용합니다. Azure Backup을 통해 여러 VM 디스크에 일관성이 있는 백업을 만들 수 있습니다. 또한 지역 손실로부터 복구를 제공하도록 지역에 Backup 자격 증명 모음을 복제할 수 있습니다.

## <a name="storage"></a>Storage
Azure Storage가 자동화된 복제본을 통해 데이터 복원력을 제공하는 반면 이렇게 하더라도 실수로 의도치 않은 삭제, 업데이트 등을 통해 응용 프로그램 코드(또는 개발자/사용자)의 데이터를 손상시키지 않도록 방지하지 않습니다. 응용 프로그램이나 사용자 오류가 발생하는 경우 데이터의 정확성을 유지 관리하려면 감사 로그가 포함된 데이터를 보조 저장소 위치에 복사하는 등 고급 기술이 필요합니다. 개발자는 [스냅숏 기능](https://msdn.microsoft.com/library/azure/ee691971.aspx) Blob을 활용할 수 있습니다. 이를 통해 Blob 내용의 읽기 전용 특정 시점 스냅숏을 만들 수 있습니다. Azure Storage Blob에 대한 데이터 정확성 솔루션의 기초로 사용할 수 있습니다.

### <a name="blob-and-table-storage-backup"></a>Blob 및 Table Storage 백업
Blob 및 테이블이 지속되는 동안 데이터의 현재 상태를 나타냅니다. 데이터를 원치 않는 수정 또는 삭제에서 복구하려면 이전 상태로 복원해야 합니다. 특정 시점 복사본을 저장하고 유지하기 위해 Azure에서 제공하는 기능을 활용하여 구현할 수 있습니다.

Azure Blob의 경우 [Blob 스냅숏 기능](https://msdn.microsoft.com/library/ee691971.aspx)을 사용하여 특정 시점 백업을 수행할 수 있습니다. 각 스냅숏의 경우 마지막 스냅숏 상태 이후 Blob 내의 차이점을 저장하는 데 필요한 저장소에 대한 비용이 청구됩니다. 스냅숏은 기반하는 원본 Blob의 존재 여부에 종속되므로 다른 Blob 또는 다른 저장소 계정에 대한 복사 작업을 권장합니다. 이렇게 하면 실수로 삭제되지 않도록 백업 데이터를 적절하게 보호합니다. Azure 테이블의 경우 다른 테이블 또는 Azure Blob에 특정 시간 복사본을 만들 수 있습니다. 테이블 및 Blob의 응용 프로그램 수준 백업을 수행하는 자세한 지침과 예제를 다음에서 찾을 수 있습니다.

* [응용 프로그램 오류로부터 테이블 보호](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/05/03/protecting-your-tables-against-application-errors/)
* [응용 프로그램 오류로부터 Blob 보호](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/04/29/protecting-your-blobs-against-application-errors/)

## <a name="database"></a>데이터베이스
Azure SQL Database에 대해 사용할 수 있는 몇 가지 [비즈니스 연속성](/azure/sql-database/sql-database-business-continuity/) (백업, 복원) 옵션이 있습니다. [데이터베이스 복사](/azure/sql-database/sql-database-copy/) 기능을 사용하거나 SQL Server bacpac 파일을 [내보내고](/azure/sql-database/sql-database-export/) [가져와서](https://msdn.microsoft.com/library/hh710052.aspx) 데이터베이스를 복사할 수 있습니다. 데이터베이스 복사가 트랜잭션에 일관성이 있는 결과를 제공하는 반면 bacpac(가져오기/내보내기 서비스를 통해)은 그렇지 않습니다. 다음 두 옵션은 데이터 센터 내에서 큐 기반 서비스로 실행되며 현재는 완료 시간 SLA를 제공하지 않습니다.

> [!NOTE]
> 데이터베이스 복사 및 가져오기/내보내기 옵션은 원본 데이터베이스에 상당한 수준의 부하를 둘 수 있습니다. 그리고 리소스 경합 또는 제한 이벤트를 트리거할 수 있습니다.
> 
> 

### <a name="sql-database-backup"></a>SQL Database Backup
Microsoft Azure SQL Database에 대한 특정 시점 백업은 [Azure SQL Database를 복사](/azure/sql-database/sql-database-copy/)하여 수행됩니다. 이 명령을 사용하여 동일한 논리 데이터베이스 서버 또는 다른 서버에 트랜잭션 측면에서 일관된 데이터베이스의 복사본을 만들 수 있습니다. 두 경우 모두 데이터베이스 복사는 완벽하게 작동하고 원본 데이터베이스에서 완전히 독립적입니다. 만든 각 복사본은 특정 시점 복구 옵션을 나타냅니다. 새 데이터베이스를 원본 데이터베이스 이름으로 바꾸어 데이터베이스 상태를 완전히 복구할 수 있습니다. 또는 Transact-SQL 쿼리를 사용하여 새 데이터베이스에서 데이터의 특정 하위 집합을 복구할 수 있습니다. SQL Database에 대한 자세한 내용은 [Azure SQL Database 비즈니스 연속성 개요](/azure/sql-database/sql-database-business-continuity/)를 참조하세요.

### <a name="sql-server-on-virtual-machines-backup"></a>Virtual Machines 백업의 SQL Server
Azure IaaS(infrastructure as a service) 가상 머신 또는 IaaS VM과 함께 사용되는 SQL 서버의 경우 전통적인 백업과 로그 전달이라는 두 가지 옵션이 있습니다. 전통적인 백업을 사용하면 특정 시점으로 복원할 수 있지만 복구 프로세스가 느립니다. 전통적인 백업을 복원하려면 초기 전체 백업을 시작한 다음 이후에 수행된 모든 백업을 적용해야 합니다. 두 번째 옵션은 로그 전달 세션을 구성하여 로그 백업의 복원을 지연하는 것입니다(예: 2시간별로). 주 서버에서 일어난 오류로부터 복구하도록 창을 제공합니다.

## <a name="other-azure-platform-services"></a>다른 Azure 플랫폼 서비스
일부 Azure 플랫폼 서비스는 사용자 제어 저장소 계정 또는 Azure SQL Database에 정보를 저장합니다. 계정이나 저장소 리소스가 삭제되거나 손상된 경우 서비스에 심각한 오류가 발생할 수 있습니다. 이러한 경우에 이러한 리소스가 삭제되거나 손상되면 다시 만드는 데 사용할 수 있는 백업을 유지 관리하는 것이 중요합니다.

Azure 웹 사이트 및 Azure Mobile Services의 경우 연결된 데이터베이스를 백업하고 유지 관리해야 합니다. Azure 미디어 서비스 및 Virtual Machines의 경우 연결된 Azure Storage 계정 및 해당 계정의 모든 리소스를 유지 관리해야 합니다. 예를 들어 Virtual Machines의 경우 Azure Blob 저장소에서 VM 디스크를 백업하고 관리해야 합니다.

## <a name="checklists-for-data-corruption-or-accidental-deletion"></a>데이터 손상 또는 삭제 실수에 대한 검사 목록
## <a name="virtual-machines-checklist"></a>Virtual Machines 검사 목록
1. 이 문서의 Virtual Machines 섹션을 검토합니다.
2. Azure Backup(또는 Azure Blob 저장소 및 VHD 스냅숏을 사용한 고유한 백 시스템)을 사용하여 VM 디스크를 백업 및 유지 관리합니다.

## <a name="storage-checklist"></a>저장소 검사 목록
1. 이 문서의 저장소 섹션을 검토합니다.
2. 정기적으로 중요한 저장소 리소스를 백업합니다.
3. Blob에 스냅숏 기능을 사용하도록 고려합니다.

## <a name="database-checklist"></a>데이터베이스 검사 목록
1. 이 문서의 데이터베이스 섹션을 검토합니다.
2. 데이터베이스 복사 명령을 사용하여 특정 시점 백업을 만듭니다.

## <a name="sql-server-on-virtual-machines-backup-checklist"></a>Virtual Machines 백업 검사 목록의 SQL Server
1. 이 문서의 Virtual Machines 백업에서 SQL Server 섹션을 검토합니다.
2. 전통적인 백업 및 복원 기술을 사용합니다.
3. 지연된 로그 전달 세션을 만듭니다.

## <a name="web-apps-checklist"></a>Web Apps 검사 목록
1. 있는 경우 연결된 데이터베이스를 백업 및 유지 관리합니다.

## <a name="media-services-checklist"></a>Media Services 검사 목록
1. 연결된 저장소 리소스를 백업 및 유지 관리합니다.

## <a name="more-information"></a>자세한 정보
Azure의 백업 및 복원 기능에 대한 자세한 내용은 [저장소, 백업 및 복구 시나리오](https://azure.microsoft.com/documentation/scenarios/storage-backup-recovery/)를 참조하세요.


