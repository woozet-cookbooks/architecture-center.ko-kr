---
title: OLTP(온라인 트랜잭션 처리)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 8650b919fc1a59240343015493a1fe41c8729a72
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848702"
---
# <a name="online-transaction-processing-oltp"></a>OLTP(온라인 트랜잭션 처리)

컴퓨터 시스템을 사용하는 트랜잭션 데이터 관리는 OLTP(온라인 트랜잭션 처리)라고 합니다. OLTP 시스템은 조직의 일상적인 작업에서 발생하는 비즈니스 상호 작용을 기록하고, 이 데이터를 쿼리하여 유추할 수 있도록 지원합니다.

## <a name="transactional-data"></a>트랜잭션 데이터

트랜잭션 데이터는 조직의 활동과 관련된 상호 작용을 추적하는 정보입니다. 이러한 상호 작용은 일반적으로 고객으로부터 수신된 지불, 공급업체로 수행된 지불, 인벤토리를 통해 이동되는 제품, 수행된 주문 또는 배달된 서비스와 같은 비즈니스 트랜잭션입니다. 트랜잭션 자체를 나타내는 트랜잭션 이벤트는 일반적으로 시간 차원, 일부 숫자 값 및 다른 데이터에 대한 참조를 포함합니다. 

트랜잭션은 일반적으로 *원자성*을 가지고 *일관*되어야 합니다. 원자성은 전체 트랜잭션이 항상 하나의 작업 단위로 성공 또는 실패하고, 절반만 완료된 상태를 유지하지 않는다는 것을 의미합니다. 트랜잭션을 완료할 수 없는 경우 데이터베이스 시스템은 해당 트랜잭션의 일부로 이미 수행된 모든 단계를 롤백해야 합니다. 전형적인 RDBMS에서, 이 롤백은 트랜잭션을 완료할 수 없을 때 자동으로 발생합니다. 일관성은 트랜잭션이 항상 데이터를 유효한 상태로 유지함을 의미합니다. (원자성 및 일관성에 대한 매우 비공식적인 설명입니다. 이러한 속성에 대해 [ACID](https://en.wikipedia.org/wiki/ACID)와 같은 좀 더 공식적인 설명이 있습니다.)

트랜잭션 데이터베이스는 모든 데이터가 모든 사용자 및 프로세스에 대해 엔터프라이즈의 컨텍스트 내에서 강력하게 일관된 상태를 유지하도록 하기 위해 비관적 잠금과 같은 다양한 잠금 전략을 사용하여 트랜잭선의 강력한 일관성을 지원할 수 있습니다. 

트랜잭션 데이터를 사용하는 가장 일반적인 배포 아키텍처는 3계층 아키텍처의 데이터 저장소 계층입니다. 3계층 아키텍처는 일반적으로 프레젠테이션 계층, 비즈니스 논리 계층 및 데이터 저장소 계층으로 구성됩니다. 관련된 배포 아키텍처는 여러 중간 계층 처리 비즈니스 논리를 가질 수 있는 [N 계층](/azure/architecture/guide/architecture-styles/n-tier)입니다.

## <a name="typical-traits-of-transactional-data"></a>트랜잭션 데이터의 일반적인 특성

트랜잭션 데이터는 다음과 같은 특성을 가질 수 있습니다.

| 요구 사항 | 설명 |
| --- | --- |
| 정규화 | 고도로 정규화됨 |
| 스키마 | 쓰기 시 스키마, 강력하게 적용|
| 일관성 | 강력한 일관성 ACID 보장 |
| 무결성 | 높은 무결성 |
| 트랜잭션 사용 | 예 |
| 잠금 전략 | 비관적 또는 낙관적|
| 업데이트 가능 | 예 |
| 추가 가능 | 예 |
| 워크로드 | 과도 쓰기, 보통 읽기 |
| 인덱싱 | 기본 및 보조 인덱스 |
| 데이터 크기 | 소규모~중간 규모 |
| 모델 | 관계형 |
| 데이터 모양 | 테이블 형식 |
| 쿼리 유연성 | 매우 유연 |
| 확장 | 작음(MB) ~ 큼(몇 TB) | 

## <a name="when-to-use-this-solution"></a>이 솔루션을 사용해야 하는 경우

비즈니스 트랜잭션을 효율적으로 처리 및 저장하고, 클라이언트 응용 프로그램에 일관된 방식에서 사용할 수 있게 하려는 경우 OLTP를 선택합니다. 또한 분명한 처리 지연이 비즈니스의 일상 작업에 부정적인 영향을 미칠 때 이 아키텍처를 사용합니다.

OLTP 시스템은 트랜잭션을 효율적으로 처리 및 저장할 뿐만 아니라 트랜잭션 데이터를 쿼리하도록 디자인되었습니다. OLTP 시스템에서 개별 트랜잭션을 효율적으로 처리하고 저장한다는 목적은 데이터 정규화, 즉 데이터를 덜 중복되는 좀 더 작은 청크로 분할함으로써 어느 정도 달성됩니다. 이 경우 OLTP 시스템이 많은 수의 트랜잭션을 독립적으로 처리할 수 있도록 하고, 중복 데이터가 있을 때 데이터 무결성을 유지하기 위해 필요한 추가 처리가 해소되므로 효율성이 유지됩니다.

## <a name="challenges"></a>과제
OLTP 시스템을 구현 및 사용할 경우 다음과 같은 몇 가지 해결 과제가 발생할 수 있습니다.

- 잘 계획된 SQL Server 기반 솔루션과 같은 예외도 있지만, OLTP 시스템이 대량의 데이터에 대한 집계를 처리하는 데 항상 적절한 것은 아닙니다. 수백만 개의 개별 트랜잭션에 대한 집계 계산에 의존하는 데이터 분석은 OLTP 시스템의 리소스를 과도하게 사용합니다. 따라서 실행이 느려질 수 있으며, 데이터베이스의 다른 트랜잭션을 차단하게 되어 속도 저하를 야기할 수 있습니다.
- 고도로 정규화된 데이터에 대해 분석 및 보고 작업을 수행할 경우, 조인을 사용해서 대부분의 쿼리를 비정규화해야 하므로 쿼리가 복잡해질 수 있습니다. 또한 OLTP 시스템에서 데이터베이스 개체에 대한 명명 규칙은 간결하고 간단한 편입니다. 명명 규칙이 간결하고 정규화가 높아지면서 비즈니스 사용자가 DBA 또는 데이터 개발자의 도움 없이 OLTP 시스템을 쿼리하는 것이 어려워집니다.
- 트랜잭션 기록을 무기한 저장하고, 하나의 테이블에 너무 많은 데이터를 저장하게 되면 저장된 트랜잭션 수에 따라 쿼리 성능이 저하될 수 있습니다. 일반적인 해결 방법은 OLTP 시스템에서 적절한 기간(예: 현재 회계 연도) 동안 두었다가 기록 데이터를 데이터 마트 또는 [데이터 웨어하우스](./data-warehousing.md) 등의 다른 시스템으로 오프로드하는 것입니다.

## <a name="oltp-in-azure"></a>Azure의 OLTP

[App Service Web Apps](/azure/app-service/app-service-web-overview)에 호스트되는 웹 사이트, App Service에서 실행되는 REST API 등의 응용 프로그램이나 모바일 또는 데스크톱 응용 프로그램은 일반적으로 REST API를 매개자로 사용해서 OLTP 시스템과 통신합니다.

실제로 대부분의 워크로드는 순수한 OLTP가 아닙니다. 분석 구성 요소의 역할을 하기도 합니다. 또한, 운영 체제에 대한 보고서 실행 등, 실시간 보고에 대한 요구도 높아지고 있습니다. 이것을 HTAP(하이브리드 트랜잭션 및 분석 처리)라고도 합니다. 자세한 내용은 [OLAP(온라인 분석 처리)](./online-analytical-processing.md)를 참조하세요.

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

|                              | Azure SQL Database | Azure Virtual Machine의 SQL Server | Azure Database for MySQL | Azure Database for PostgreSQL |
|------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|      관리되는 서비스인지 여부      |        예         |                   아니요                   |           예            |              예              |
|       플랫폼에서 실행       |        해당 없음         |         Windows, Linux, Docker         |           해당 없음            |              해당 없음              |
| 프로그래밍 기능 <sup>1</sup> |   T-SQL, .NET, R   |         T-SQL, .NET, R, Python         |  T-SQL, .NET, R, Python  |              SQL              |

[1] 많은 프로그래밍 언어가 OLTP 데이터 저장소에 연결하고 이 저장소를 사용할 수 있도록 하는 클라이언트 드라이버 지원은 포함되지 않습니다.

### <a name="scalability-capabilities"></a>확장성 기능

| | Azure SQL Database | Azure Virtual Machine의 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- |
| 최대 데이터베이스 인스턴스 크기 | [4TB](/azure/sql-database/sql-database-resource-limits) | 256TB | [1 TB](/azure/mysql/concepts-limits) | [1 TB](/azure/postgresql/concepts-limits) |
| 용량 풀 지원 여부  | 예 | 예 | 아니오 | 아니오 |
| 클러스터 스케일 아웃 지원 여부  | 아니오 | 예 | 아니요 | 아니오 |
| 동적 확장성(강화)  | 예 | 아니오 | 예 | 예 |

### <a name="analytic-workload-capabilities"></a>분석 워크로드 기능

| | Azure SQL Database | Azure Virtual Machine의 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| 임시 테이블 | 예 | 예 | 아니오 | 아니오 |
| 메모리 내(메모리 최적화) 테이블 | 예 | 예 | 아니오 | 아니오 |
| Columnstore 지원 여부 | 예 | 예 | 아니오 | 아니요 |
| 적응 쿼리 처리 | 예 | 예 | 아니오 | 아니요 |

### <a name="availability-capabilities"></a>가용성 기능

| | Azure SQL Database | Azure Virtual Machine의 SQL Server| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| 읽기 가능 보조 복제본 | 예 | 예 | 아니오 | 아니오 | 
| 지리적 복제 | 예 | 예 | 아니오 | 아니오 | 
| 보조 복제본으로 자동 장애 조치(Failover) | 예 | 아니요 | 아니요 | 아니오|
| 지정 시간 복원 | 예 | 예 | 예 | 예 |

### <a name="security-capabilities"></a>보안 기능

|                                                                                                             | Azure SQL Database | Azure Virtual Machine의 SQL Server | Azure Database for MySQL | Azure Database for PostgreSQL |
|-------------------------------------------------------------------------------------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|                                             행 수준 보안                                              |        예         |                  예                   |           예            |              예              |
|                                                데이터 마스킹                                                 |        예         |                  예                   |            아니요            |              아니오               |
|                                         투명한 데이터 암호화                                         |        예         |                  예                   |           예            |              예              |
|                                  특정 IP 주소로 액세스 제한                                   |        예         |                  예                   |           예            |              예              |
|                                  VNET 액세스만 허용하도록 액세스 제한                                  |        예         |                  예                   |            아니오            |              아니오               |
|                                    Azure Active Directory 인증                                    |        예         |                  예                   |            아니오            |              아니요               |
|                                       Active Directory 인증                                       |         아니오         |                  예                   |            아니오            |              아니요               |
|                                         Multi-Factor Authentication                                         |        예         |                  예                   |            아니오            |              아니오               |
| [상시 암호화](/sql/relational-databases/security/encryption/always-encrypted-database-engine) 지원 여부 |        예         |                  예                   |           예            |              아니오               |
|                                                 개인 IP                                                  |         아니오         |                  예                   |           예            |              아니오               |

