---
title: 실시간 메시지 수집 기술 선택
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 2e6578b779950b5ef11bda7b8ba1fb2e45e09f4e
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/23/2018
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a>Azure에서 실시간 메시지 수집 기술 선택

실시간 처리에서는 데이터 스트림이 실시간으로 캡처되고 최소 대기 시간으로 처리됩니다. 많은 실시간 처리 솔루션에는 메시지에 대한 버퍼로 작동하고 스케일 아웃 처리, 안정적인 전달 및 기타 메시지 큐 의미 체계를 지원하는 메시지 수집 저장소가 필요합니다. 

## <a name="what-are-your-options-for-real-time-message-ingestion"></a>실시간 메시지 수집을 위해 사용할 수 있는 옵션은 무엇인가요?

- [Azure Event Hubs](/azure/event-hubs/)
- [Azure IoT Hub](/azure/iot-hub/)
- [HDInsight의 Kafka](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a>Azure Event Hubs

[Azure Event Hubs](/azure/event-hubs/)는 초당 수백만 개의 이벤트를 수신하고 처리할 수 있는 확장성이 뛰어난 데이터 스트리밍 플랫폼 및 이벤트 수집 서비스입니다. Event Hubs는 분산된 소프트웨어와 장치에서 생성된 이벤트, 데이터 또는 원격 분석을 처리하고 저장할 수 있습니다. Event Hub로 전송된 데이터는 실시간 분석 공급자 또는 일괄 처리/저장소 어댑터를 사용하여 변환하고 저장할 수 있습니다. Event Hubs는 낮은 대기 시간으로 대규모의 게시-구독 기능을 제공하므로 빅 데이터 시나리오에 적합합니다.

## <a name="azure-iot-hub"></a>Azure IoT Hub

[Azure IoT Hub](/azure/iot-hub/)는 수백만의 IoT 장치와 클라우드 기반 백 엔드 간에서 안정적이고 안전한 양방향 통신이 가능하도록 관리되는 서비스입니다.

IoT Hub의 기능은 다음과 같습니다.

* 다수의 장치-클라우드 및 클라우드-장치 통신 옵션. 이러한 옵션에는 단방향 메시징, 파일 전송 및 요청-회신 메서드가 포함됩니다.
* 다른 Azure 서비스로의 메시지 라우팅
* 장치 메타데이터와 동기화된 상태 정보에 대한 쿼리 가능한 저장소
* 장치 단위 보안 키 또는 X.509 인증서를 사용하는 보안 통신 및 액세스 제어
* 장치 연결 및 장치 ID 관리 이벤트에 대한 모니터링

메시지 수집 측면에서 IoT Hub는 Event Hubs와 비슷합니다. 그러나 메시지 수집 뿐만 아니라 IoT 장치 연결을 관리하도록 특수하게 디자인되었습니다. 자세한 내용은 [Azure IoT Hub 및 Azure Event Hubs의 비교](/azure/iot-hub/iot-hub-compare-event-hubs)를 참조하세요. 

## <a name="kafka-on-hdinsight"></a>HDInsight의 Kafka

[Apache Kafka](https://kafka.apache.org/)는 실시간 스트리밍 데이터 파이프라인 및 스트리밍 응용 프로그램을 만드는 데 사용할 수 있는 오픈 소스 분산형 스트리밍 플랫폼입니다. 또한 Kafka는 명명된 데이터 스트림을 게시하고 구독할 수 있는 메시지 대기열과 비슷한 메시지 브로커 기능을 제공합니다. 수평으로 확장이 가능하고 내결함성이며 매우 빠릅니다. [HDInsight의 Kafka](/azure/hdinsight/kafka/apache-kafka-get-started)는 Azure에서 관리되고 확장성이 뛰어난 고가용성 서비스로서 Kafka를 제공합니다. 

Kafka의 몇 가지 일반적인 사용 사례는 다음과 같습니다.

* **메시징** Kafka는 게시-구독 메시지 패턴을 지원하므로 종종 메시지 브로커로 사용됩니다.
* **활동 추적**. Kafka는 순서대로 레코드 로그를 기록하므로 활동(예: 웹 사이트의 사용자 작업)을 추적하고 다시 만드는 데 사용할 수 있습니다.
* **집계**. 스트림 처리를 사용하여 결합할 서로 다른 스트림의 정보를 한데 모으고 중앙에서 이 정보를 운영 데이터로 집중적으로 처리할 수 있습니다.
* **변환**. 스트림 처리를 사용하여 여러 입력 토픽의 데이터를 하나 이상의 출력 토픽으로 결합하고 보강할 수 있습니다.

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- IoT 장치와 Azure 간에 양방향 통신이 필요한가요? 그렇다면 IoT Hub를 선택합니다.

- 개별 장치에 대한 액세스를 관리하고, 특정 장치에 대한 액세스 권한을 취소할 수 있어야 하나요? 그렇다면 IoT Hub를 선택합니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다. 

| | IoT 허브 | Event Hubs | HDInsight의 Kafka |
| --- | --- | --- | --- |
| 클라우드-장치 통신 | 예 | 아니요 | 아니요 |
| 장치에서 시작한 파일 업로드 | 예 | 아니오 | 아니오 |
| 장치 상태 정보 | [장치 쌍](/azure/iot-hub/iot-hub-devguide-device-twins) | 아니요 | 아니오 |
| 프로토콜 지원 | MQTT, AMQP, HTTPS <sup>1</sup> | AMQP, HTTPS | [Kafka Protocol](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| 보안 | 장치 단위 ID. 취소 가능한 액세스 제어 | 공유 액세스 정책. 게시자 정책을 통해 취소 제한 | SASL을 사용한 인증, 플러그 가능 인증, 외부 인증 서비스와의 통합 지원 |

[1] [Azure IoT 프로토콜 게이트웨이](/azure/iot-hub/iot-hub-protocol-gateway)를 사용자 지정 게이트웨이로 사용하여 IoT Hub에 대한 프로토콜 적응을 사용할 수도 있습니다.

자세한 내용은 [Azure IoT Hub 및 Azure Event Hubs의 비교](/azure/iot-hub/iot-hub-compare-event-hubs)를 참조하세요.
