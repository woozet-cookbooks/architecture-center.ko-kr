---
title: "데이터 웨어하우스 선택"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 9cb3d4d0196b02da76d85c7f7f0e4a2a69d531e9
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-data-warehouse-in-azure"></a>Azure에서 데이터 웨어하우스 선택

데이터 웨어하우스는 하나 이상의 개별 원본에서 가져온 통합된 데이터의 조직적인 중앙 관계형 리포지토리입니다. 이 항목에서는 Azure의 데이터 웨어하우스에 대한 옵션을 비교합니다.

> [!NOTE]
> 데이터 웨어하우스를 사용해야 하는 경우에 대한 자세한 내용은 [데이터 웨어하우징 및 데이터 마트](../scenarios/data-warehousing.md)를 참조하세요.

## <a name="what-are-your-options-when-choosing-a-data-warehouse"></a>데이터 웨어하우스를 선택할 때에 옵션은 무엇입니까?

Azure에서는 사용자의 요구에 따라 다음과 같은 몇 가지 데이터 웨어하우스 구현 옵션을 사용할 수 있습니다. 다음 목록은 SMP([Symmetric Multiprocessing](https://en.wikipedia.org/wiki/Symmetric_multiprocessing)) 및 MPP([Massively Parallel Processing](https://en.wikipedia.org/wiki/Massively_parallel))의 두 범주로 구분됩니다. 

SMP:

- [Azure SQL Database](/azure/sql-database/)
- [가상 컴퓨터의 SQL Server](/sql/sql-server/sql-server-technical-documentation)

MPP:

- [Azure Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [HDInsight의 Apache Hive](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [HDInsight의 대화형 쿼리(Hive LLAP)](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

일반적으로 MPP는 종종 빅 데이터에 사용되지만, SMP 기반 웨어하우스는 소형에서 중형 데이터 집합(최대 4-100TB)에 가장 적합합니다. 소형/중형과 빅 데이터 간의 경계는 조직의 정의 및 지원 인프라와 어느 정도 관련이 있습니다. ([OLTP 데이터 저장소 선택](oltp-data-stores.md#scalability-capabilities)을 참조하세요.) 

데이터 크기보다는, 워크로드 패턴 유형이 더 큰 고려 요인일 수 있습니다. 예를 들어, 복잡한 쿼리는 SMP 솔루션에 비해 너무 느리므로 대신 MPP 솔루션이 필요할 수 있습니다. MPP 기반 시스템은 노드에서 작업이 배포되고 통합되는 방식 때문에, 데이터 크기가 작은 경우 성능이 저하될 수 있습니다. 데이터 크기가 이미 1TB를 초과하며, 지속적으로 증가할 수 있는 경우 MPP 솔루션을 선택하는 것이 좋습니다. 그러나 데이터 크기가 이 범위보다 작지만 워크로드가 SMP 솔루션의 가용 리소스를 초과할 경우에는 MPP가 가장 적합할 수 있습니다.

데이터 웨어하우스에서 액세스하거나 저장하는 데이터는 [Azure Data Lake Store](/azure/data-lake-store/)와 같은 데이터 레이크를 포함하는 다양한 데이터 원본에서 가져올 수 있습니다. Azure Data Lake를 사용할 수 있는 MPP 서비스의 서로 다른 장점을 비교하는 비디오 세션을 보려면 [Azure Data Lake and Azure Data Warehouse: Applying Modern Practices to Your App](https://azure.microsoft.com/resources/videos/build-2016-azure-data-lake-and-azure-data-warehouse-applying-modern-practices-to-your-app/)(Azure Data Lake 및 Azure Data Warehouse: 앱에 최신 사례 적용)을 참조하세요.

SMP 시스템은 모든 리소스(CPU/메모리/디스크)를 공유하는 관계형 데이터베이스 관리 시스템의 단일 인스턴스로 나타낼 수 있습니다. SMP 시스템은 확장할 수 있습니다. VM에서 실행 중인 SQL Server의 경우 VM 크기를 확장할 수 있습니다. Azure SQL Database의 경우 다른 서비스 계층을 선택하여 확장할 수 있습니다. 

MPP 시스템은 계산 노드(자체 CUP, 메모리 및 I/O 하위 시스템 포함)를 더 추가하여 스케일 아웃할 수 있습니다. 서버를 강화하는 데는 물리적 제한이 있습니다. 이 경우 워크로드에 따라 스케일 아웃하는 것이 좀 더 바람직합니다. 그러나 MPP 솔루션에는 데이터의 쿼리, 모델링, 분할 방식의 차이와 병렬 처리와 관련된 기타 요인 때문에 다른 기술 집합이 필요합니다. 

사용할 SMP 솔루션을 결정할 때는 [Azure SQL 데이터베이스 및 Azure VM의 SQL Server에서 자세히 보기](/azure/sql-database/sql-database-paas-vs-sql-server-iaas#a-closer-look-at-azure-sql-database-and-sql-server-on-azure-vms)를 참조하세요. 

워크로드가 계산 및 메모리 집약적인 소형 및 중형 데이터 집합에도 Azure SQL Data Warehouse를 사용할 수 있습니다. SQL Data Warehouse 패턴 및 일반적인 시나리오에 대해 읽어보세요.

- [SQL Data Warehouse 패턴 및 안티패턴](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/)
- [SQL Data Warehouse 로딩 패턴 및 전략](https://blogs.msdn.microsoft.com/sqlcat/2017/05/17/azure-sql-data-warehouse-loading-patterns-and-strategies/)
- [Azure SQL Data Warehouse로 데이터 마이그레이션](https://blogs.msdn.microsoft.com/sqlcat/2016/08/18/migrating-data-to-azure-sql-data-warehouse-in-practice/)(영문)
- [Azure SQL Data Warehouse를 사용하는 일반적인 ISV 응용 프로그램 패턴](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/common-isv-application-patterns-using-azure-sql-data-warehouse/)

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 사용자 고유의 서버를 관리하지 않고 관리되는 서비스를 원하시나요?

- 매우 큰 데이터 집합 또는 매우 복잡하고 오래 실행되는 쿼리로 작업하고 있나요? 그렇다면 MPP 옵션을 고려합니다. 

- 큰 데이터 집합의 경우 데이터 원본이 구조적인가요 아니면 반구조적인가요? 구조화되지 않은 데이터는 HDInsight의 Spark, Azure Databricks, HDInsight의 Hive LLAP 또는 Azure Data Lake Analytics와 같은 빅 데이터 환경에서 처리해야 할 수 있습니다. 이러한 모든 환경은 ELT(추출, 로드, 변환) 및 ETL(추출, 변환, 로드) 엔진으로 사용할 수 있습니다. 처리된 데이터를 구조화된 데이터로 출력하여 SQL Data Warehouse 또는 다른 옵션 중 하나로 보다 쉽게 로드할 수 있습니다. 구조화된 데이터의 경우, SQL Data Warehouse는 아주 높은 성능을 요구하는 계산 집약적 워크로드를 위해 “계산에 최적화”라고 지칭하는 성능 계층을 갖습니다.

- 현재, 운영 데이터에서 기록 데이터를 분리하려고 하나요? 그렇다면 [오케스트레이션](pipeline-orchestration-data-movement.md)이 필요한 옵션 중 하나를 선택합니다. 이것은 과도한 읽기 액세스에 최적화된 독립 실행형 웨어하우스로, 별도의 기록 데이터 저장소로 가장 적합합니다.

- OLTP 데이터 저장소 이외의 여러 원본에 있는 데이터를 통합해야 하나요? 그렇다면 여러 데이터 원본을 쉽게 통합하는 옵션을 고려합니다. 

- 다중 테넌트 요구 사항이 있나요? 그렇다면 이 요구 사항에는 SQL Data Warehouse가 바람직하지 않습니다. 자세한 내용은 [SQL Data Warehouse 패턴 및 안티패턴](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/)을 참조하세요.

- 관계형 데이터 저장소를 선호하나요? 그렇다면, 관계형 데이터 저장소가 있는 옵션으로 범위를 좁히되, 필요한 경우 PolyBase와 같은 도구를 사용하여 비관계형 데이터 저장소를 쿼리할 수 있다는 점에 유의하세요. 그러나 PolyBase를 사용하기로 결정한 경우 워크로드의 구조화되지 않은 데이터 집합에 대해 성능 테스트를 실행합니다.

- 실시간 보고 요구 사항이 있나요? 고용량의 단일 삽입에 대해 신속한 쿼리 응답 시간이 필요한 경우 실시간 보고를 지원할 수 있는 옵션으로 범위를 좁힙니다.

- 많은 수의 동시 사용자 및 연결을 지원해야 하나요? 동시 사용자/연결 수를 지원하는 기능은 여러 가지 요인에 따라 달라집니다. 

    - Azure SQL Database에 대해서는 서비스 계층에 따라 [문서화된 리소스 제한](/azure/sql-database/sql-database-resource-limits)을 참조하세요. 
    
    - SQL Server에서는 최대 32,767개의 사용자 연결을 허용합니다. VM에서 실행할 경우 성능은 VM 크기 및 기타 요인에 따라 달라집니다. 
    
    - SQL Data Warehouse는 동시 쿼리 및 동시 연결에 대한 제한이 있습니다. 자세한 내용은 [SQL Data Warehouse의 동시성 및 워크로드 관리](/azure/sql-data-warehouse/sql-data-warehouse-develop-concurrency)를 참조하세요. [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)와 같은 보완 서비스를 사용하여 SQL Data Warehouse의 제한을 극복하는 것이 바람직합니다.

- 어떤 종류의 워크로드가 발생하나요? 일반적으로 MPP 기반 웨어하우스 솔루션은 분석, 일괄 처리 워크로드에 가장 적합합니다. 워크로드가 기본적으로 많은 작은 읽기/쓰기 작업 또는 여러 행 단위 작업을 포함하는 트랜잭션 작업인 경우 SMP 옵션 중 하나를 사용하는 것이 좋습니다. 이 지침의 한 가지 예외는 HDInsight 클러스터에서 Spark Streaming 같은 스트림 처리를 사용하고 데이터를 Hive 테이블 내에 저장하는 경우입니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.

### <a name="general-capabilities"></a>일반 기능

| | Azure SQL Database | SQL Server(VM) | SQL Data Warehouse | HDInsight의 Apache Hive | HDInsight의 Hive LLAP |
| --- | --- | --- | --- | --- | --- | -- |
| 관리되는 서비스인지 여부 | 예 | 아니요 | 예 | 예 <sup>1</sup> | 예 <sup>1</sup> |
| 데이터 오케스트레이션 필요(데이터 사본/기록 데이터 보유) | 아니요 | 아니요 | 예 | 예 | 예 |
| 여러 데이터 원본을 쉽게 통합 | 아니오 | 아니요 | 예 | 예 | 예 |
| 계산 일시 중지 지원 여부 | 아니오 | 아니요 | 예 | 아니요 <sup>2</sup> | 아니요 <sup>2</sup> |
| 관계형 데이터 저장소 | 예 | 예 |  예 | 아니요 | 아니오 |
| 실시간 보고 | 예 | 예 | 아니요 | 아니요 | 예 |
| 유연한 백업/복원 지점 | 예 | 예 | 아니요 <sup>3</sup> | 예 <sup>4</sup> | 예 <sup>4</sup> |
| SMP/MPP | SMP | SMP | MPP | MPP | MPP |

[1] 수동 구성 및 크기 조정

[2] 필요할 때 HDInsight 클러스터를 삭제한 다음, 다시 만들 수 있습니다. 클러스터를 삭제할 떼 데이터가 유지되도록 외부 데이터 저장소를 클러스터에 연결합니다. Azure Data Factory로 워크로드를 처리하는 요청 시 HDInsight 클러스터를 만들어 클러스터의 수명 주기를 자동화한 다음, 처리가 완료된 후 삭제할 수 있습니다.

[3] SQL Data Warehouse를 사용할 경우 지난 7일 이내의 사용 가능한 복원 지점으로 데이터베이스를 복원할 수 있습니다. 스냅숏은 4~8시간마다 시작되며 7일 동안 사용할 수 있습니다. 7일보다 오래된 스냅숏은 만료되고 해당 복원 지점을 더 이상 사용할 수 없게 됩니다.

[4] 필요할 때 백업 및 복원할 수 있는 [외부 Hive metastore](/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters#use-hiveoozie-metastore)를 사용하는 것이 좋습니다. Blob Storage 또는 Data Lake Store에 적용되는 표준 백업 및 복원 옵션을 데이터에 사용할 수 있으며, 더 큰 유연성이나 사용 편의성을 원할 경우 [Imanis Data](https://azure.microsoft.com/blog/imanis-data-cloud-migration-backup-for-your-big-data-applications-on-azure-hdinsight/)와 같은 타사 HDInsight 백업 및 복원 솔루션을 사용할 수 있습니다.

### <a name="scalability-capabilities"></a>확장성 기능

| | Azure SQL Database | SQL Server(VM) |  SQL Data Warehouse | HDInsight의 Apache Hive | HDInsight의 Hive LLAP |
| --- | --- | --- | --- | --- | --- | -- |
| 고가용성을 위한 중복 지역 서버  | 예 | 예 | 예 | 아니오 | 아니요 |
| 쿼리 스케일 아웃 지원 여부(분산 쿼리)  | 아니요 | 아니오 | 예 | 예 | 예 |
| 동적 확장성(강화)  | 예 | 아니오 | 예 <sup>1</sup> | 아니오 | 아니요 |
| 데이터의 메모리 내 캐싱 지원 여부 | 예 |  예 | 아니요 | 예 | 예 |

[1] SQL Data Warehouse에서는 DWU(데이터 웨어하우스 단위) 수를 조정하여 강화 및 축소할 수 있습니다. [Azure SQL Data Warehouse의 계산 능력 관리](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview)를 참조하세요.

### <a name="security-capabilities"></a>보안 기능

| | Azure SQL Database | 가상 컴퓨터의 SQL Server | SQL Data Warehouse | HDInsight의 Apache Hive | HDInsight의 Hive LLAP |
| --- | --- | --- | --- | --- | --- | -- |
| 인증  | SQL/Azure AD(Azure Active Directory) | SQL / Azure AD / Active Directory | SQL / Azure AD | 로컬/Azure AD <sup>1</sup> | 로컬/Azure AD <sup>1</sup> |
| 권한 부여  | 예 | 예 | 예 | 예 | 예 <sup>1</sup> | 예 <sup>1</sup> |
| 감사  | 예 | 예 | 예 | 예 | 예 <sup>1</sup> | 예 <sup>1</sup> |
| 휴지 상태의 암호화 | 예 <sup>2</sup> | 예 <sup>2</sup> | 예 <sup>2</sup> | 예 <sup>2</sup> | 예 <sup>1</sup> | 예 <sup>1</sup> |
| 행 수준 보안 | 예 | 예 | 예 | 아니오 | 예 <sup>1</sup> | 예 <sup>1</sup> |
| 방화벽 지원 여부 | 예 | 예 | 예 | 예 | 예 <sup>3</sup> | 예 <sup>3</sup> |
| 동적 데이터 마스킹 | 예 | 예 | 예 | 아니오 | 예 <sup>1</sup> | 예 <sup>1</sup> |

[1] [도메인 가입 HDInsight 클러스터](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)를 사용해야 합니다.

[2] 미사용 데이터의 암호화 및 암호 해독을 위해 TDE(투명한 데이터 암호화)를 사용해야 합니다.

[3] [Azure Virtual Network 내에서 사용](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)할 때 지원됩니다.

데이터 웨어하우스 보안에 대해 읽어보세요.

* [SQL Database 보안 설정](/azure/sql-database/sql-database-security-overview#connection-security)
* [SQL Data Warehouse에서 데이터베이스 보호](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security)
* [Azure Virtual Network를 사용하여 Azure HDInsight 확장](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)
* [도메인 조인 HDInsight 클러스터의 엔터프라이즈 수준 Hadoop 보안](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)

