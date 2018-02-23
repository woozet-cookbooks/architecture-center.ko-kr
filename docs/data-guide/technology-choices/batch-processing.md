---
title: "일괄 처리 기술 선택"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bfb850ee8e9d8fd41927b4ca3b612e15b5ae6b11
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Azure에서 일괄 처리 기술 선택

빅 데이터 솔루션은 종종 장기 실행 일괄 처리 작업을 사용하여 데이터를 필터링 및 집계하고, 분석을 위해 준비합니다. 일반적으로 이러한 작업이 수행되는 동안 확장 가능한 저장소(예: HDFS, Azure Data Lake Store 및 Azure Storage)에서 원본 데이터를 읽고, 처리하고, 확장 가능한 저장소의 새 파일에 출력을 씁니다. 

대량의 데이터를 처리하기 위해 이러한 일괄 처리 엔진은 기본적으로 계산 능력을 스케일 아웃할 수 있어야 합니다. 그러나 실시간 처리와 달리, 일괄 처리는 분 단위에서 시간 단위로 측정되는 대기 시간(데이터 수집 시간과 결과 계산 사이의 간격)을 발생하게 됩니다.

## <a name="what-are-your-options-when-choosing-a-batch-processing-technology"></a>일괄 처리 기술을 선택할 때 사용할 수 있는 옵션은 무엇인가요?

Azure에서 다음의 모든 데이터 저장소는 일괄 처리의 핵심 요구 사항을 충족합니다.

- [Azure 데이터 레이크 분석](/azure/data-lake-analytics/)
- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [HDInsight(Spark 포함)](/azure/hdinsight/spark/apache-spark-overview)
- [HDInsight(Hive 포함)](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [HDInsight(Hive LLAP 포함)](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 사용자 고유의 서버를 관리하지 않고 관리되는 서비스를 원하시나요?

- 선언적 또는 명령적 방식 중에서 어떤 방식으로 일괄 처리 논리를 작성하려고 하나요?

- 일괄 처리를 많이 수행할 예정인가요? 그렇다면 클러스터를 일시 중지하거나, 일괄 처리 작업을 기준으로 가격 책정 모델을 선택할 수 있는 옵션을 고려합니다.

- 예를 들어 참조 데이터를 조회하기 위해 일괄 처리 방식으로 관계형 데이터 저장소를 쿼리해야 하나요? 그렇다면 외부 관계형 저장소의 쿼리를 허용하는 옵션을 고려합니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다. 

### <a name="general-capabilities"></a>일반 기능

| | Azure 데이터 레이크 분석 | Azure SQL Data Warehouse | HDInsight(Spark 포함) | HDInsight(Spark 포함) | HDInsight(Hive LLAP 포함) |
| --- | --- | --- | --- | --- | --- |
| 관리되는 서비스인지 여부 | 예 | 예 | 예 <sup>1</sup> | 예 <sup>1</sup> | 예 <sup>1</sup> |
| 계산 일시 중지 지원 여부 | 아니요 | 예 | 아니요 | 아니요 | 아니요 |
| 관계형 데이터 저장소 | 예 | 예 | 아니요 | 아니오 | 아니오 |
| 프로그래밍 기능 | U-SQL | T-SQL | Python, Scala, Java, R | HiveQL | HiveQL |
| 프로그래밍 패러다임 | 선언적 및 명령적 방식 혼합  | 선언적 | 선언적 및 명령적 방식 혼합 | 선언적 | 선언적 | 
| 가격 책정 모델 | 일괄 처리 작업 기준 | 클러스터 시간 기준 | 클러스터 시간 기준 | 클러스터 시간 기준 | 클러스터 시간 기준 |  

[1] 수동 구성 및 크기 조정 사용
 
### <a name="integration-capabilities"></a>통합 기능
| | Azure 데이터 레이크 분석 | SQL Data Warehouse | HDInsight(Spark 포함) | HDInsight(Spark 포함) | HDInsight(Hive LLAP 포함) |
| --- | --- | --- | --- | --- | --- |
| Azure Data Lake Store에서 액세스 | 예 | 예 | 예 | 예 | 예 |
| Azure Storage에서 쿼리 | 예 | 예 | 예 | 예 | 예 |
| 외부 관계형 저장소에서 쿼리 | 예 | 아니요 | 예 | 아니요 | 아니요 |

### <a name="scalability-capabilities"></a>확장성 기능
| | Azure 데이터 레이크 분석 | SQL Data Warehouse | HDInsight(Spark 포함) | HDInsight(Spark 포함) | HDInsight(Hive LLAP 포함) |
| --- | --- | --- | --- | --- | --- |
| 스케일 아웃 단위  | 작업 기준 | 클러스터 기준 | 클러스터 기준 | 클러스터 기준 | 클러스터 기준 |
| 빠른 스케일 아웃(1분 이내) | 예 | 예 | 아니요 | 아니요 | 아니오 |
| 데이터의 메모리 내 캐싱 | 아니오 | 예 | 예 | 아니오 | 예 | 

### <a name="security-capabilities"></a>보안 기능
| | Azure 데이터 레이크 분석 | SQL Data Warehouse | HDInsight(Spark 포함) | HDInsight의 Apache Hive | HDInsight의 Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| 인증  | Azure AD(Azure Active Directory) | SQL/Azure AD | 아니요 | 로컬/Azure AD <sup>1</sup> | 로컬/Azure AD <sup>1</sup> |
| 권한 부여  | 예 | 예| 아니오 | 예 <sup>1</sup> | 예 <sup>1</sup> |
| 감사  | 예 | 예 | 아니오 | 예 <sup>1</sup> | 예 <sup>1</sup> |
| 휴지 상태의 암호화 | 예| 예 <sup>2</sup> | 예 | 예 | 예 |
| 행 수준 보안 | 아니요 | 예 | 아니요 | 예 <sup>1</sup> | 예 <sup>1</sup> |
| 방화벽 지원 여부 | 예 | 예 | 예 | 예 <sup>3</sup> | 예 <sup>3</sup> |
| 동적 데이터 마스킹 | 아니요 | 아니요 | 아니요 | 예 <sup>1</sup> | 예 <sup>1</sup> |

[1] [도메인 가입 HDInsight 클러스터](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)를 사용해야 합니다.

[2] 미사용 데이터의 암호화 및 암호 해독을 위해 TDE(투명한 데이터 암호화)를 사용해야 합니다.

[3] [Azure Virtual Network 내에서 사용](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)할 때 지원됩니다.
