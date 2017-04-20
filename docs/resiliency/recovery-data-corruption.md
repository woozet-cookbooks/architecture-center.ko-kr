---
title: Recover from data corruption or accidental deletion
description: Article on understanding how to recover from data corruption of data or accidental data deletion to and designing resilient, highly available, fault tolerant applications as well as planning for disaster recovery
author: adamglick
ms.service: guidance
ms.topic: article
ms.date: 08/18/2016
ms.author: pnp
---
[!INCLUDE [header](../_includes/header.md)]
# Azure 복원 기술 지침: 데이터 손상 또는 우발적 삭제 시 복구
강력한 비즈니스 연속성 계획에는 데이터가 손상되거나 우발적으로 삭제된 경우에 대비한 계획이 포함됩니다. 아래 정보는 응용 프로그램 오류 또는 운영자 실수로 인하여 데이터가 손상되었거나 우발적으로 삭제된 이후에 복구하는 것에 관한 내용입니다.

## 가상 컴퓨터
응용 프로그램 오류 또는 우발적 삭제로부터 Azure Virtual Machines(때로는 서비스형 인프라 VM이라고도 함)를 보호하려면 [Azure Backup](https://azure.microsoft.com/services/backup/)을 사용하십시오. Azure Backup을 통해 여러 VM 디스크 간에 일관된 백업을 만들 수 있습니다. 또한 백업 모음을 여러 지역에 걸쳐 복제하여 지역 손실 시에 복구를 지원합니다.

## 저장소
Azure Storage가 자동 복제본을 통해서 데이터 복원을 지원하지만, 이것이 우발적 또는 원치 않는 삭제, 업데이트 등으로 인한 데이터 손상으로부터 응용 프로그램 코드(또는 개발자/사용자)를 보호하지는 못합니다. 응용 프로그램 또는 사용자 오류에 직면했을 때 데이터 정확도를 유지하려면, 감사 로그를 사용하여 데이터를 보조 저장소 위치로 복사하는 등 고급 기법이 필요합니다. 개발자들은 Blob 콘텐츠의 읽기 전용 특정 시점 스냅샷을 만들 수 있는 Blob [스냅샷 기능](https://msdn.microsoft.com/library/azure/ee691971.aspx)을 활용할 수 있습니다. 이는 Azure Storage Blob에 대한 데이터 정확도 솔루션의 근거로 사용할 수 있습니다.

### Blob 및 표 저장소 백업
Blob와 표는 지속성이 높지만 항상 데이터의 현재 상태를 나타냅니다. 원치 않는 데이터 수정이나 삭제 시 복구를 위해서는 데이터를 이전 상태로 복원하는 것이 필요할 수 있습니다. 이는 Azure가 특정 시점의 복사본을 저장하여 유지하기 위해 제공하는 기능을 통해서 달성할 수 있습니다.

Azure Blob의 경우 [Blob 스냅샷 기능](https://msdn.microsoft.com/library/ee691971.aspx)을 사용하여 지정 시점 백업을 수행할 수 있습니다. 각각의 스냅샷에 대해, 최근 스냅샷 상태 이후로 Blob 내에서 발생한 차이만을 저장하는 데 필요한 저장소 비용만 부과됩니다. 스냅샷은 기준이 되는 원본 Blob의 존재에 따라 결정되므로, 다른 Blob나 심지어 다른 저장소 계정으로 복사하는 것이 바람직합니다. 그러면 백업 데이터가 우발적 삭제로부터 적절히 보호됩니다. Azure 테이블의 경우, 다른 테이블이나 Azure Blobs에 특정 시점 복사본을 만들 수 있습니다. 표와 Blob를 응용 프로그램 수준에서 백업하는 작업에 관한 자세한 지침과 예는 다음을 참조하십시오.

* [응용 프로그램 오류로부터 표 보호](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/05/03/protecting-your-tables-against-application-errors/)
* [응용 프로그램 오류로부터 Blob 보호](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/04/29/protecting-your-blobs-against-application-errors/)

## 데이터베이스
Azure SQL Database에서 사용할 수 있는 몇 가지 [비즈니스 연속성](/azure/sql-database/sql-database-business-continuity/) (백업, 복원) 옵션이 있습니다. 데이터베이스는 [데이터베이스 복사](/azure/sql-database/sql-database-copy/) 기능을 사용하거나 또는 SQL Server bacpac 파일 [내보내기](/azure/sql-database/sql-database-export/) 및 [가져오기](https://msdn.microsoft.com/library/hh710052.aspx)를 통해 복사할 수 있습니다. 데이터베이스 복사 기능은 트랜잭션 측면에서 일관된 결과를 제공하지만 (가져오기/내보내기 서비스를 통한) bacpac은 그렇지 않습니다. 이 두 가지 옵션은 데이터센터 내에서 큐 기반 서비스로 실행되며, 현재는 완료시간 SLA를 제공하지는 않습니다.

> [!참고]
> 데이터베이스 복사 및 가져오기/내보내기 옵션은 원본 데이터베이스에 상당한 부하를 줍니다. 이들 옵션은 리소스 경합 또는 제한 이벤트를 트리거할 수 있습니다.
> 
> 

### SQL Database 백업
Microsoft Azure SQL Database의 지정 시간 백업은 [Azure SQL Database 복사](/azure/sql-database/sql-database-copy/)를 통해 이루어집니다. 이 명령을 사용하여 동일한 논리 데이터베이스 서버에 또는 다른 서버에 트랜잭션 측면에서 일관된 데이터베이스 복사본을 만들 수 있습니다. 어떤 경우든 데이터베이스 복사본이 완전한 기능을 하며 원본 데이터베이스와는 완전히 독립적으로 유지됩니다. 작성하는 각 복사본은 지정 시간 복구 옵션을 나타냅니다. 새 데이터베이스를 원본 데이터베이스 이름으로 바꿈으로써 데이터베이스 상태를 완전히 복구할 수 있습니다. 또는 Transact-SQL 쿼리를 사용하여 새 데이터베이스로부터 특정 데이터 하위 집합을 복구할 수도 있습니다. SQL Database에 관한 자세한 내용은 [Azure SQL Database를 통한 비즈니스 연속성 개요](/azure/sql-database/sql-database-business-continuity/)를 참조하십시오.

### 가상 컴퓨터의 SQL Server 백업
Azure 서비스형 인프라 가상 컴퓨터(IaaS 또는 IaaS VM이라고도 함)에서 사용되는 SQL Server의 경우 두 가지 백업 방식 즉 일반 백업과 로그 전달 옵션이 있습니다. 일반 백업 옵션을 사용하면 특정 시점으로 복원할 수 있지만 복원 프로세스가 느립니다. 일반 백업을 복원하려면 우선 초기 전체 백업을 시작해야 하고, 그 다음에 확보한 백업을 적용하는 것이 필요합니다. 두 번째 옵션은 로그 백업 복원을 지연시키기 위해 (예를 들어 2시간마다) 로그 전달 세션을 구성하는 것입니다. 이를 통해 주 데이터베이스에 발생한 오류로부터 복원할 구간을 제공합니다.

## 기타 Azure 플랫폼 서비스
일부 Azure 플랫폼 서비스는 정보를 사용자 제어 저장소 계정 또는 Azure SQL Database에 저장합니다. 계정이나 저장소 리소스가 삭제되거나 손상될 경우 서비스에 심각한 오류가 발생할 수 있습니다. 이 경우, 삭제 또는 손상되었을 경우에 리소스를 다시 만들 수 있도록 백업을 유지하는 것이 중요합니다.

Azure 웹 사이트 및 Azure 모바일 서비스의 경우, 관련 데이터베이스를 백업하여 유지해야 합니다. Azure Media Service 및 가상 컴퓨터의 경우, 관련된 Azure Storage 계정과 그 계정의 모든 리소스를 유지해야 합니다. 예를 들어 가상 컴퓨터의 경우, VM 디스크를 Azure Blob 저장소에 백업하여 관리해야 합니다.

## 데이터 손상 또는 우발적 삭제에 대한 체크리스트
## 가상 컴퓨터 체크리스트
1. 이 문서의 가상 컴퓨터 섹션을 검토합니다.
2. Azure Backup을 통해서 (또는 Azure Blob 저장소와 VHD 스냅샷을 사용하여 자체 백업 시스템을 통해서) VM 디스크를 백업하여 유지합니다.

## 저장소 체크리스트
1. 이 문서의 저장소 섹션을 검토합니다.
2. 중요한 저장소 리소스를 정기적으로 백업합니다.
3. Blob에 대해 스냅샷 기능의 사용을 고려합니다.

## 데이터베이스 체크리스트
1. 이 문서의 데이터베이스 섹션을 검토합니다.
2. 데이터베이스 복사 명령을 사용하여 특정 시점의 백업을 작성합니다.

## 가상 컴퓨터의 SQL Server 백업 체크리스트
1. 이 문서에 있는 가상 컴퓨터의 SQL Server 백업 섹션을 검토합니다.
2. 일반 백업 및 복원 기법을 사용합니다.
3. 지연된 로그 전달 세션을 만듭니다.

## 웹 응용 프로그램 체크리스트
1. 관련 데이터베이스를 백업하여 유지합니다 (있을 경우).

## Media Services 체크리스트
1. 관련 저장소 리소스를 백업하여 유지합니다.

## 자세한 정보
Azure의 백업 및 복원에 관한 자세한 내용은 [저장소, 백업 및 복원 시나리오](https://azure.microsoft.com/documentation/scenarios/storage-backup-recovery/)를 참조하십시오.


