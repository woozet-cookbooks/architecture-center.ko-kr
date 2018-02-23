---
title: "스트림 처리 기술 선택"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: e06f46e2951159219bd8cc430102e2ec0c5d6d4d
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-stream-processing-technology-in-azure"></a>Azure에서 스트림 처리 기술 선택

이 문서에서는 Azure의 실시간 스트림 처리에 대한 기술 선택 사항을 비교합니다.

실시간 스트림 처리는 큐 또는 파일 기반 저장소의 메시지를 사용하고, 메시지를 처리하고, 결과를 다른 메시지 큐, 파일 저장소 또는 데이터베이스에 전달합니다. 처리에는 메시지의 쿼리, 필터링 및 집계가 포함될 수 있습니다. 스트림 처리 엔진은 데이터의 무한 스트림을 사용하고 최소 대기 시간으로 결과를 생성할 수 있어야 합니다. 자세한 내용은 [실시간 처리](../scenarios/real-time-processing.md)를 참조하세요.

## <a name="what-are-your-options-when-choosing-a-technology-for-real-time-processing"></a>실시간 처리를 위한 기술을 선택할 때 사용할 수 있는 옵션은 무엇인가요?
Azure에서 다음의 모든 데이터 저장소는 핵심 요구 사항을 충족하여 실시간 처리를 지원합니다.
- [Azure Stream Analytics](/azure/stream-analytics/)
- [HDInsight(Spark Streaming 포함)](/azure/hdinsight/spark/apache-spark-streaming-overview)
- [HDInsight(Storm 포함)](/azure/hdinsight/storm/apache-storm-overview)
- [Azure 기능](/azure/azure-functions/functions-overview)
- [Azure App Service WebJobs](/azure/app-service/web-sites-create-web-jobs)

## <a name="key-selection-criteria"></a>주요 선택 조건

실시간 처리 시나리오에 대해 먼저 다음 질문에 응답하여 사용자 요구에 적합한 서비스를 선택합니다.

- 스트림 처리 논리 제작에 대해 선언적 방식과 명령적 방법 중에서 어떤 방식을 선호하나요?

- 임시 처리 또는 기간 지정 기능이 기본적으로 지원될 필요가 있나요?

- Avro, JSON 또는 CSV 형식 외의 형식으로 데이터가 들어오나요? 그렇다면 사용자 지정 코드를 사용하는 형식을 지원하는 옵션을 고려합니다.

- 1GB/s 이상으로 처리 크기를 조정해야 하나요? 그렇다면 클러스터 크기에 따라 확장되는 옵션을 고려합니다. 

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서 주요 기능 차이점을 요약해서 보여 줍니다. 

### <a name="general-capabilities"></a>일반 기능
| | Azure Stream Analytics | HDInsight(Spark Streaming 포함) | HDInsight(Storm 포함) | Azure 기능 | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | 
| 프로그래밍 기능 | Stream Analytics 쿼리 언어, JavaScript | Scala, Python, Java | Java, C# | C#, F#, Node.js | C#, Node.js, PHP, Java, Python |
| 프로그래밍 패러다임 | 선언적 | 선언적 및 명령적 방식 혼합 | 명령적 | 명령적 | 명령적 |    
| 가격 책정 모델 | 스트리밍 단위 기준 | 클러스터 시간 기준 | 클러스터 시간 기준 | 함수 실행 및 리소스 사용량 기준 | App Service 계획 시간 기준 |  

### <a name="integration-capabilities"></a>통합 기능
| | Azure Stream Analytics | HDInsight(Spark Streaming 포함) | HDInsight(Storm 포함) | Azure 기능 | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | 
| 입력 | [Stream Analytics 입력](/azure/stream-analytics/stream-analytics-define-inputs)  | Event Hubs, IoT Hub, Kafka, HDFS  | Event Hubs, IoT Hub, Storage Blob, Azure Data Lake Store  | [지원되는 바인딩](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus, Storage Queues, Storage Blob, Event Hubs, WebHooks, Cosmos DB, Files |
| Sinks |  [Stream Analytics 출력](/azure/stream-analytics/stream-analytics-define-outputs) | HDFS | Event Hubs, Service Bus, Kafka | [지원되는 바인딩](/azure/azure-functions/functions-triggers-bindings#supported-bindings) | Service Bus, Storage Queues, Storage Blob, Event Hubs, WebHooks, Cosmos DB, Files | 

### <a name="processing-capabilities"></a>처리 기능
| | Azure Stream Analytics | HDInsight(Spark Streaming 포함) | HDInsight(Storm 포함) | Azure 기능 | Azure App Service WebJobs |
| --- | --- | --- | --- | --- | --- | 
| 기본 제공 임시/창 지원 | 예 | 예 | 예 | 아니오 | 아니요 |
| 입력 데이터 형식 | Avro, JSON 또는 CSV, UTF-8로 인코딩 | 사용자 지정 코드를 사용하는 모든 형식 | 사용자 지정 코드를 사용하는 모든 형식 | 사용자 지정 코드를 사용하는 모든 형식 | 사용자 지정 코드를 사용하는 모든 형식 |
| 확장성 | [쿼리 파티션](/azure/stream-analytics/stream-analytics-parallelization) | 클러스터 크기에 따라 제한 | 클러스터 크기에 따라 제한 | 최대 200개의 함수 앱 인스턴스를 병렬로 처리 | App Service 계획 용량에 따라 제한 | 
| 지연 도착 및 순서가 벗어난 이벤트 처리 지원 | 예 | 예 | 예 | 아니요 | 아니요 |

참고 항목:

- [실시간 메시지 수집 기술 선택](./real-time-ingestion.md)
- [Apache Storm 및 Azure Stream Analytics 비교](/azure/stream-analytics/stream-analytics-comparison-storm)
- [실시간 처리](../scenarios/real-time-processing.md)
