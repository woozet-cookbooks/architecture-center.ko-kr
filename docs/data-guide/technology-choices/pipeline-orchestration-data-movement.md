---
title: 데이터 파이프라인 오케스트레이션 기술 선택
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 17aeb871bc815793295ed610795e5e83de72c637
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288805"
---
# <a name="choosing-a-data-pipeline-orchestration-technology-in-azure"></a>Azure의 데이터 파이프라인 오케스트레이션 기술 선택

대부분의 빅 데이터 솔루션은 워크플로에 캡슐화된 반복되는 데이터 처리 작업으로 구성됩니다. 파이프라인 오케스트레이터는 이러한 워크플로를 자동화하는 데 도움이 되는 도구입니다. 오케스트레이터는 작업을 예약하고, 워크플로를 실행하고, 태스크 간 종속 관계를 조정할 수 있습니다.

## <a name="what-are-your-options-for-data-pipeline-orchestration"></a>데이터 파이프라인 오케스트레이션에 사용할 수 있는 옵션은 무엇인가요?

Azure에서 다음 서비스 및 도구는 파이프라인 오케스트레이션, 제어 흐름 및 데이터 이동에 대한 핵심 요구 사항을 충족합니다.

- [Azure 데이터 팩터리](/azure/data-factory/)
- [HDInsight의 Oozie](/azure/hdinsight/hdinsight-use-oozie-linux-mac)
- [SSIS(SQL Server Integration Services)](/sql/integration-services/sql-server-integration-services)

이러한 서비스 및 도구는 서로 독립적으로 사용되거나, 함께 사용되어 하이브리드 솔루션을 생성할 수 있습니다. 예를 들어, Azure Data Factory V2의 IR(Integration Runtime)은 기본적으로 관리되는 Azure 계산 환경에서 SSIS 패키지를 실행할 수 있습니다. 이러한 서비스의 경우 일부 기능은 중복되지만 몇 가지 핵심적인 차이점이 있습니다.

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 데이터를 이동하고 변환하기 위한 빅 데이터 기능이 필요한가요? 일반적으로 수기가바이트에서 수테라바이트 단위의 데이터를 의미합니다. 그렇다면 빅 데이터에 가장 적합한 옵션으로 범위를 좁혀보세요.

- 대규모로 작동될 수 있는 관리되는 서비스가 필요한가요? 그렇다면 로컬 처리 능력에 따라 제한되지 않는 클라우드 기반 서비스 중 하나를 선택합니다.

- 일부 데이터 원본 위치은 온-프레미스에 있나요? 그렇다면 클라우드 및 온-프레미스 데이터 원본 또는 대상에 작동할 수 있는 옵션을 찾아보세요.

- 원본 데이터가 HDFS 파일 시스템의 Blob 저장소에 저장되어 있나요? 그렇다면 Hive 하이브 쿼리를 지원하는 옵션을 선택합니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.

### <a name="general-capabilities"></a>일반 기능

| | Azure 데이터 팩터리 | SQL Server 통합 서비스(SSIS) | HDInsight의 Oozie
| --- | --- | --- | --- |
| 관리 | 예 | 아니요 | 예 |
| 클라우드 기반 | 예 | 아니요(로컬) | 예 |
| 필수 요소 | Azure 구독 | SQL Server  | Azure 구독, HDInsight 클러스터 |
| 관리 도구 | Azure Portal, PowerShell, CLI, .NET SDK | SSMS, PowerShell | Bash 셸, Oozie REST API, Oozie Web UI |
| 가격 | 사용당 지급 | 라이선스/기능 요금 | HDInsight 클러스터 실행에 대한 추가 비용 없음 |

### <a name="pipeline-capabilities"></a>파이프라인 기능

| | Azure 데이터 팩터리 | SQL Server 통합 서비스(SSIS) | HDInsight의 Oozie
| --- | --- | --- | --- |
| 데이터 복사 | 예 | 예 | 예 |
| 사용자 지정 변환 | 예 | 예 | 예(MapReduce, Pig 및 Hive 작업) |
| Azure Machine Learning 점수 매기기 | 예 | 예(스크립팅 사용) | 아니오 |
| 요청 시 HDInsight | 예 | 아니오 | 아니오 |
| Azure Batch | 예 | 아니요 | 아니오 |
| Pig, Hive, MapReduce | 예 | 아니요 | 예 |
| Spark | 예 | 아니요 | 아니요 |
| SSIS 패키지 실행 | 예 | 예 | 아니요 |
| 흐름 제어 | 예 | 예 | 예 |
| 온-프레미스 데이터 액세스 | 예 | 예 | 아니요 |

### <a name="scalability-capabilities"></a>확장성 기능

| | Azure 데이터 팩터리 | SQL Server 통합 서비스(SSIS) | HDInsight의 Oozie
| --- | --- | --- | --- |
| 강화 | 예 | 아니오 | 아니요 |
| 확장 | 예 | 아니오 | 예(클러스터에 작업자 노드 추가) |
| 빅 데이터에 최적화 | 예 | 아니요 | 예 |

