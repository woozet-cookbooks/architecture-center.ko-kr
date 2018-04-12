---
title: OLTP 데이터 저장소 선택
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 1c27d7d5f3b78f40822de6b77664dbf49b1367f6
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/31/2018
---
# <a name="choosing-an-oltp-data-store-in-azure"></a>Azure에서 OLTP 데이터 저장소 선택

OLTP(온라인 트랜잭션 처리)는 트랜잭션 데이터 및 트랜잭션 처리의 관리 기능입니다. 이 항목에서는 Azure의 OLTP 솔루션에 대한 옵션을 비교합니다.

> [!NOTE]
> OLTP 데이터 저장소를 사용해야 하는 경우에 대한 자세한 내용은 [온라인 트랜잭션 처리](../scenarios/online-analytical-processing.md)를 참조하세요.

## <a name="what-are-your-options-when-choosing-an-oltp-data-store"></a>OLTP 데이터 저장소를 선택할 때의 옵션은 무엇인가요?

Azure에서 다음의 모든 데이터 저장소는 OLTP 및 트랜잭션 데이터 관리의 핵심 요구 사항을 충족합니다.

- [Azure SQL Database](/azure/sql-database/)
- [Azure Virtual Machine의 SQL Server](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database for PostgreSQL](/azure/postgresql/)

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 사용자 고유의 서버를 관리하지 않고 관리되는 서비스를 원하시나요?

- 사용하는 솔루션이 Microsoft SQL Server, MySQL 또는 PostgreSQL 호환성에 대해 특정 종속성을 갖나요? 응용 프로그램은 데이터 저장소와 통신하기 위해, 선택 가능한 데이터 저장소를 지원되는 드라이버를 기준으로 또는 사용되는 데이터베이스에 대해 가정되는 조건에 따라 제한할 수 있습니다.

- 쓰기 처리량 요구 수준이 특히 높은 편인가요? 그렇다면 메모리 내 테이블을 제공하는 옵션을 선택합니다. 

- 사용하는 솔루션이 다중 테넌트 솔루션인가요? 그렇다면 데이터베이스마다 고정 리소스가 있는 것이 아니라, 탄력적인 리소스 풀에서 여러 데이터베이스 인스턴스를 가져오는 방식의 용량 풀을 지원하는 옵션을 고려합니다. 이렇게 하면 모든 데이터베이스 인스턴스에서 용량을 보다 잘 분산하고, 좀 더 비용 효율적인 솔루션을 만들 수 있습니다.

- 데이터를 여러 지역에서 짧은 대기 시간으로 읽을 수 있어야 하나요? 그렇다면 읽을 수 있는 보조 복제본을 지원하는 옵션을 선택합니다.

- 여러 지리적 지역에서 데이터베이스의 높은 가용성을 유지해야 하나요? 그렇다면 지리적 복제를 지원하는 옵션을 선택합니다. 또한 주 복제본에서 보조 복제본으로의 자동 장애 조치(Failover)를 지원하는 옵션을 고려합니다.

- 데이터베이스에 특정 보안 요구가 있나요? 그렇다면 행 수준 보안, 데이터 마스킹 및 투명한 데이터 암호화와 같은 기능을 제공하는 옵션을 검토합니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.

### <a name="general-capabilities"></a>일반 기능 
| | Azure SQL Database | Azure Virtual Machine의 SQL Server | Azure Database for MySQL | Azure Database for PostgreSQL |
| --- | --- | --- | --- | --- | --- |
| 관리되는 서비스인지 여부 | 예 | 아니오 | 예 | 예 |
| 플랫폼에서 실행 | 해당 없음 | Windows, Linux, Docker | 해당 없음 | 해당 없음 |
| 프로그래밍 기능 <sup>1</sup> | T-SQL, .NET, R | T-SQL, .NET, R, Python | T-SQL, .NET, R, Python | SQL | SQL |

[1] 많은 프로그래밍 언어가 OLTP 데이터 저장소에 연결하고 이 저장소를 사용할 수 있도록 하는 클라이언트 드라이버 지원은 포함되지 않습니다.

### <a name="scalability-capabilities"></a>확장성 기능
| | Azure SQL Database | Azure Virtual Machine의 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- |
| 최대 데이터베이스 인스턴스 크기 | [4TB](/azure/sql-database/sql-database-resource-limits) | 256TB | [1 TB](/azure/mysql/concepts-limits) | [1 TB](/azure/postgresql/concepts-limits) |
| 용량 풀 지원 여부  | 예 | 예 | 아니요 | 아니오 |
| 클러스터 스케일 아웃 지원 여부  | 아니요 | 예 | 아니오 | 아니오 |
| 동적 확장성(강화)  | 예 | 아니오 | 예 | 예 |

### <a name="analytic-workload-capabilities"></a>분석 워크로드 기능
| | Azure SQL Database | Azure Virtual Machine의 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| 임시 테이블 | 예 | 예 | 아니오 | 아니요 |
| 메모리 내(메모리 최적화) 테이블 | 예 | 예 | 아니오 | 아니오 |
| Columnstore 지원 여부 | 예 | 예 | 아니요 | 아니요 |
| 적응 쿼리 처리 | 예 | 예 | 아니요 | 아니요 |

### <a name="availability-capabilities"></a>가용성 기능
| | Azure SQL Database | Azure Virtual Machine의 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| 읽기 가능 보조 복제본 | 예 | 예 | 아니오 | 아니요 | 
| 지리적 복제 | 예 | 예 | 아니요 | 아니오 | 
| 보조 복제본으로 자동 장애 조치(Failover) | 예 | 아니요 | 아니요 | 아니오|
| 지정 시간 복원 | 예 | 예 | 예 | 예 |

### <a name="security-capabilities"></a>보안 기능
| | Azure SQL Database | Azure Virtual Machine의 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| 행 수준 보안 | 예 | 예 | 예 | 예 |
| 데이터 마스킹 | 예 | 예 | 아니오 | 아니오 |
| 투명한 데이터 암호화 | 예 | 예 | 예 | 예 |
| 특정 IP 주소로 액세스 제한 | 예 | 예 | 예 | 예 |
| VNET 액세스만 허용하도록 액세스 제한 | 예 | 예 | 아니오 | 아니요 |
| Azure Active Directory 인증 | 예 | 예 | 아니오 | 아니오 |
| Active Directory 인증 | 아니오 | 예 | 아니오 | 아니오 |
| Multi-Factor Authentication | 예 | 예 | 아니오 | 아니요 |
| [상시 암호화](/sql/relational-databases/security/encryption/always-encrypted-database-engine) 지원 여부 | 예 | 예 | 예 | 아니요 | 아니요 |
| 개인 IP | 아니오 | 예 | 예 | 아니오 | 아니오 |

