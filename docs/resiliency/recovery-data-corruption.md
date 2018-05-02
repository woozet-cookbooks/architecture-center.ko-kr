---
title: 데이터 손상 또는 우발적 삭제로부터 복구
description: 데이터 손상 또는 실수로 인한 데이터 삭제로부터 데이터를 복구하는 방법을 이해하고 재해 복구에 대한 계획 뿐만 아니라 복원력 있고 항상 사용 가능한 내결함성 응용 프로그램을 설계하는 방법에 대한 문서입니다.
author: MikeWasson
ms.date: 01/10/2018
ms.openlocfilehash: b0716de39fe69d607b9a63e51356d28bbcdbfeae
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/16/2018
---
# <a name="recover-from-data-corruption-or-accidental-deletion"></a>데이터 손상 또는 우발적 삭제로부터 복구 

강력한 비즈니스 연속성 계획의 일부로 손상되거나 실수로 삭제된 데이터에 대한 계획을 수립합니다. 다음은 데이터가 응용 프로그램 오류 또는 운영자 오류로 인해 손상되거나 실수로 삭제된 후에 복구하는 방법에 대한 정보입니다.

## <a name="virtual-machines"></a>Virtual Machines

Azure Virtual Machines(VM)를 응용프로그램 오류나 우발적 삭제로부터 보호하려면 [Azure Backup](/azure/backup/)을 사용합니다. Azure Backup을 통해 여러 VM 디스크에 일관성이 있는 백업을 만들 수 있습니다. 또한 지역 손실로부터 복구를 제공하도록 지역에 Backup 자격 증명 모음을 복제할 수 있습니다.

## <a name="storage"></a>Storage

Azure Storage는 자동화된 복제본을 통해 데이터 복구 기능을 제공합니다. 그러나 Azure Storage는 우발적이든 악의적이든 응용프로그램 코드나 사용자가 데이터를 손상하는 것을 방지하지 못합니다. 응용 프로그램이나 사용자 오류가 발생하는 경우 데이터의 정확성을 유지 관리하려면 감사 로그가 포함된 데이터를 보조 저장소 위치에 복사하는 등 고급 기술이 필요합니다. 

- **블록 Blobs**. 각 블록 Blob의 지정 시간 스냅숏을 만듭니다. 자세한 내용은 [Blob의 스냅숏 만들기](/rest/api/storageservices/creating-a-snapshot-of-a-blob)를 참조하세요. 각 스냅숏의 경우 마지막 스냅숏 상태 이후 Blob 내의 차이점을 저장하는 데 필요한 저장소에 대한 비용이 청구됩니다. 스냅숏은 기반하는 원본 Blob의 존재 여부에 종속되므로 다른 Blob 또는 다른 저장소 계정에 대한 복사 작업을 권장합니다. 이렇게 하면 실수로 삭제되지 않도록 백업 데이터를 적절하게 보호합니다. [AzCopy](/azure/storage/common/storage-use-azcopy) 또는 [Azure PowerShell](/azure/storage/common/storage-powershell-guide-full)을 사용하여 다른 저장소 계정에 파일을 복사할 수 있습니다.

- **파일**. [스냅숏 공유](/azure/storage/files/storage-snapshots-files)를 사용하거나 AzCopy나 PowerShell을 사용해 파일을 다른 저장소 계정에 복사합니다.

- **테이블**. AzCopy를 사용하여 다른 지역에 있는 다른 저장소 계정으로 테이블 데이터를 내보냅니다.

## <a name="database"></a>데이터베이스

### <a name="azure-sql-database"></a>Azure SQL Database 

SQL Database는 데이터 손실로부터 비즈니스를 보호하기 위해 매주 전체 데이터베이스 백업과 매시간 차등 데이터베이스 백업, 그리고 5~10분 간격으로 트랜잭션 로그 백업을 모두 자동으로 수행합니다. 좀더 이른 시점으로 데이터베이스를 복원하려면 지정 시간 복원을 사용합니다. 자세한 내용은 다음을 참조하세요.

- [자동화된 데이터베이스 백업을 사용하여 Azure SQL 데이터베이스 복구](/azure/sql-database/sql-database-recovery-using-backups)

- [Azure SQL Database의 비즈니스 연속성 개요](/azure/sql-database/sql-database-business-continuity)

### <a name="sql-server-on-vms"></a>VM의 SQL Server

Vm에서 실행되는 SQL Server의 경우 두 가지 옵션이 있습니다. 전통적인 백업과 로그 전달입니다. 전통적인 백업을 사용하면 지정 시점으로 복원할 수 있지만 복구 프로세스가 느립니다. 전통적인 백업을 복원하려면 초기 전체 백업을 시작한 다음 이후에 수행된 모든 백업을 적용해야 합니다. 두 번째 옵션은 로그 전달 세션을 구성하여 로그 백업의 복원을 지연하는 것입니다(예: 2시간별로). 주 서버에서 일어난 오류로부터 복구하도록 창을 제공합니다.

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Cosmos DB는 자동으로 정기적인 백업을 수행합니다. 백업은 다른 저장소 서비스에 개별적으로 저장되고 이러한 백업은 지역 재해에 대한 복원을 위해 전역적으로 복제됩니다. 데이터베이스 또는 컬렉션을 실수로 삭제한 경우 지원 티켓을 제출하거나 Azure 지원에 문의하여 마지막 자동 백업에서 데이터를 복원할 수 있습니다. 자세한 내용은 [Azure Cosmos DB로 자동 온라인 백업 및 복원](/azure/cosmos-db/online-backup-and-restore)을 참조하세요.

### <a name="azure-database-for-mysql-azure-database-for-postresql"></a>Azure Database for MySQL, Azure Database for PostreSQL

Azure Database for MySQL 또는 Azure Database for PostreSQL를 사용할 경우 데이터베이스 서비스가 자동으로 5분마다 서비스 백업을 수행합니다. 이 자동 백업 기능을 사용하면 서버 및 모든 데이터베이스를 새 서버에 이전 특정 시점으로 복원할 수도 있습니다. 자세한 내용은 다음을 참조하세요.

- [Azure Portal을 사용하여 Azure Database for MySQL에서 서버를 백업 및 복원하는 방법](/azure/mysql/howto-restore-server-portal)

- [Azure Portal을 사용하여 Azure Database for PostgreSQL에서 서버를 백업 및 복원하는 방법](/azure/postgresql/howto-restore-server-portal)

