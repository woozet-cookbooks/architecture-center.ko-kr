---
title: "분석 데이터 저장소 선택"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b2e5e63982d4b89b95cd28e596d3b882a4a2263e
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-analytical-data-store-in-azure"></a>Azure에서 분석 데이터 저장소 선택

[빅 데이터](../concepts/big-data.md) 아키텍처에서는 처리된 데이터를 분석 도구로 쿼리할 수 있는 구조화된 형식으로 제공하는 분석 데이터 저장소가 필요한 경우가 많습니다. 실행 부하 과다 경로 및 실행 부하 미달 경로 데이터의 쿼리를 모두 지원하는 분석 데이터 저장소를 톨틀어 서비스 계층 또는 데이터 서비스 저장소라고 합니다.

서비스 계층은 실행 부하 과다 경로 및 실행 부하 미달 경로의 처리된 데이터를 다룹니다. [람다 아키텍처](../concepts/big-data.md#lambda-architecture)에서 서비스 계층은 증분 방식으로 처리된 데이터를 저장하는 _빠른 서비스_ 계층과 일괄 처리된 출력을 포함하는 _일괄 처리 서비스_ 계층으로 세분화됩니다. 서비스 계층에서는 짧은 대기 시간의 임의 읽기를 반드시 지원해야 합니다. 또한 빠른 계층용 데이터 저장소는 임의 쓰기도 지원해야 합니다. 이 저장소로 데이터를 일괄 로드하면 원치 않는 지연이 발생하기 때문입니다. 그렇지만 일괄 처리 계층용 데이터 저장소는 임의 쓰기를 지원하지 않아도 되며, 대신 일괄 쓰기를 지원해야 합니다.

모든 데이터 저장소 태스크에 가장 적합한 단일 데이터 관리 옵션은 없습니다. 태스크마다 최적화된 데이터 관리 솔루션이 다릅니다. 대부분의 실제 클라우드 앱 및 빅 데이터 프로세스는 다양한 데이터 저장소 요구를 가지며, 데이터 저장소 솔루션을 조합해서 사용하는 경우가 많습니다.

## <a name="what-are-your-options-when-choosing-an-analytical-data-store"></a>분석 데이터 저장소를 선택할 때의 옵션은 무엇인가요?

Azure에서는 사용자의 요구에 따라 다음과 같은 몇 가지 데이터 서비스 저장소 옵션을 사용할 수 있습니다.

- [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Azure SQL Database](/azure/sql-database/)
- [Azure VM의 SQL Server](/sql/sql-server/sql-server-technical-documentation)
- [HDInsight의 HBase/Phoenix](/azure/hdinsight/hbase/apache-hbase-overview)
- [HDInsight의 Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [Azure Cosmos DB](/azure/cosmos-db/)

이러한 옵션에서는 다양한 유형의 태스크에 최적화된 다양한 데이터베이스 모델을 제공합니다.

- [키/값](https://msdn.microsoft.com/library/dn313285.aspx#sec7) 데이터베이스는 각 키 값에 대해 직렬화된 단일 개체를 보유합니다. 이 데이터베이스는 지정된 키 값에 대해 하나의 항목을 가져오려고 하며 항목의 다른 속성을 기준으로 쿼리할 필요가 없는 경우에 대용량의 데이터를 저장하는 데 좋습니다.
- [문서](https://msdn.microsoft.com/library/dn313285.aspx#sec8) 데이터베이스는 값이 *document*인 키/값 데이터베이스입니다. 이 컨텍스트에서 "document"는 명명된 필드 및 값의 컬렉션입니다. 이 데이터베이스는 일반적으로 XML, YAML, JSON 또는 BSON과 같은 형식으로 데이터를 저장하지만 일반 텍스트를 사용할 수도 있습니다. 문서 데이터베이스는 키가 아닌 필드에서 쿼리를 수행하고, 보조 인덱스를 정의하여 쿼리를 보다 효율적으로 수행할 수 있습니다. 따라서 문서 데이터베이스는 문서 키의 값보다 좀 더 복잡한 조건에 따라 데이터를 검색해야 하는 응용 프로그램에 더 적합합니다. 예를 들어, 제품 ID, 고객 ID 또는 고객 이름과 같은 필드에서 쿼리를 수행할 수 있습니다.
- [열 패밀리](https://msdn.microsoft.com/library/dn313285.aspx#sec9) 데이터베이스는 데이터 저장소를 열 패밀리라고 하는 관련된 열 컬렉션으로 구성하는 키/값 데이터 저장소입니다. 예를 들어, 인구 조사 데이터베이스에는 개인의 이름(이름, 중간 이름, 성)에 대한 열 그룹 1개, 개인의 주소에 대한 그룹 1개, 개인 프로필 정보(생년월일, 성별)에 대한 그룹 1개가 포함될 수 있습니다. 이 데이터베이스는 한 사용자의 모든 데이터를 동일한 키와 연결해서 유지하면서 각 열 패밀리를 별도 파티션에 저장할 수 있습니다. 응용 프로그램은 엔터티의 모든 데이터를 읽지 않더라도 단일 열 패밀리를 읽을 수 있습니다.
- [그래프](https://msdn.microsoft.com/library/dn313285.aspx#sec10) 데이터베이스는 정보를 개체 및 관계의 컬렉션으로 저장합니다. 그래프 데이터베이스는 개체 네트워크 및 개체 간 관계를 트래버스 통과하는 쿼리를 효율적으로 수행할 수 있습니다. 예를 들어, 개체가 인사 데이터베이스의 직원일 수 있으며, “find all employees who directly or indirectly work for Scott”와 같은 쿼리를 용이하게 진행할 수 있습니다.

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 데이터에 대해 실행 부하 과다 경로로 사용될 수 있는 저장소를 제공해야 하나요? 그렇다면 빠른 서비스 계층에 최적화된 옵션으로 선택 범위를 좁혀보세요.

- 쿼리가 여러 프로세스 또는 노드 사이에서 자동으로 분산되는 MPP(Massively Parallel Processing) 지원이 필요한가요? 그렇다면 쿼리 확장을 지원하는 옵션을 선택합니다.

- 관계형 데이터 저장소를 사용하고 싶나요? 그렇다면 관계형 데이터베이스 모델을 포함하는 옵션으로 선택 범위를 좁혀보세요. 그러나 일부 비관계형 저장소는 쿼리에 대해 SQL 구문을 지원하며, PolyBase와 같은 도구를 사용하여 비관계형 데이터 저장소를 쿼리할 수 있습니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.

### <a name="general-capabilities"></a>일반 기능

| | SQL Database | SQL Data Warehouse | HDInsight의 HBase/Phoenix | HDInsight의 Hive LLAP | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| 관리되는 서비스인지 여부 | 예 | 예 | 예 <sup>1</sup> | 예 <sup>1</sup> | 예 | 예 |
| 주 데이터베이스 모델 | 관계형(columnstore 인덱스를 사용할 경우 칼럼 형식) | 칼럼 형식 저장소가 있는 관계형 테이블 | 넓은 열 저장소 | Hive/메모리 내 | 테이블 형식/MOLAP 의미 체계 모델 | 문서 저장소, 그래프, 키-값 저장소, 넓은 열 저장소 |
| SQL 언어 지원 | 예 | 예 | 예([Phoenix](http://phoenix.apache.org/) JDBC 드라이버 사용) | 예 | 아니요 | 예 |
| 빠른 서비스 계층에 최적화됨 | 예 <sup>2</sup> | 아니요 | 예 | 예 | 아니요 | 예 |

[1] 수동 구성 및 크기 조정 사용

[2] 메모리 최적화 테이블 및 해시 또는 비클러스터형 인덱스 사용
 
### <a name="scalability-capabilities"></a>확장성 기능

| | SQL Database | SQL Data Warehouse | HDInsight의 HBase/Phoenix | HDInsight의 Hive LLAP | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| 고가용성을 위한 중복 지역 서버  | 예 | 예 | 예 | 아니요 | 아니요 | 예 | 예 |
| 쿼리 확장 지원 여부  | 아니요 | 예 | 예 | 예 | 예 | 예 |
| 동적 확장성(강화)  | 예 | 예 | 아니요 | 아니오 | 예 | 예 |
| 데이터의 메모리 내 캐싱 지원 여부 | 예 | 예 | 아니요 | 예 | 예 | 아니요 |

### <a name="security-capabilities"></a>보안 기능

| | SQL Database | SQL Data Warehouse | HDInsight의 HBase/Phoenix | HDInsight의 Hive LLAP | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| 인증  | SQL/Azure AD(Azure Active Directory) | SQL/Azure AD | 로컬/Azure AD <sup>1</sup> | 로컬/Azure AD <sup>1</sup> | Azure AD | 액세스 제어(IAM)을 통한 데이터베이스 사용자/Azure AD |
| 휴지 상태의 암호화 | 예 <sup>2</sup> | 예 <sup>2</sup> | 예 <sup>1</sup> | 예 <sup>1</sup> | 예 | 예 |
| 행 수준 보안 | 예 | 아니요 | 예 <sup>1</sup> | 예 <sup>1</sup> | 예(모델에 개체 수준 보안 사용) | 아니요 |
| 방화벽 지원 여부 | 예 | 예 | 예 <sup>3</sup> | 예 <sup>3</sup> | 예 | 예 |
| 동적 데이터 마스킹 | 예 | 아니요 | 예 <sup>1</sup> | 예 * | 아니오 | 아니요 |

[1] [도메인 가입 HDInsight 클러스터](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)를 사용해야 합니다.

[2] 미사용 데이터의 암호화 및 암호 해독을 위해 TDE(투명한 데이터 암호화)를 사용해야 합니다.

[3] Azure Virtual Network 내에서 사용할 때. [Azure Virtual Network를 사용하여 Azure HDInsight 확장](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)을 참조하세요.
