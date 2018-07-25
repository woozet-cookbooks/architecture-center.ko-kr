---
title: Azure의 확장성 있는 주문 처리
description: Azure Cosmos DB를 사용하여 확장성이 높은 주문 처리 파이프라인을 구축하는 예제 시나리오입니다.
author: alexbuckgit
ms.date: 07/10/2018
ms.openlocfilehash: 541b5e9f523c64bc55526e4e2dffc57a5212e67f
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060984"
---
# <a name="scalable-order-processing-on-azure"></a>Azure의 확장성 있는 주문 처리

이 예제 시나리오는 온라인 주문 처리를 위해 확장성이 뛰어나고 복원력 있는 아키텍처가 필요한 조직과 관련이 있습니다. 잠재적인 응용 프로그램으로 전자 상거래 및 소매 POS(Point of Sale), 주문 이행 및 재고 예약 및 추적이 있습니다. 

이 시나리오에서는 마이크로 서비스를 통해 구현되는 함수 프로그래밍 모델을 사용하여 이벤트 소싱 방식을 수행합니다. 각 마이크로 서비스는 스트림 프로세서로 처리되고, 모든 비즈니스 논리는 마이크로 서비스를 통해 구현됩니다. 이 방법을 통해 고가용성 및 복원력, 지역 복제 및 빠른 성능을 구현할 수 있습니다.

Cosmos DB 및 HDInsight와 같은 관리되는 Azure 서비스를 사용하면 전역으로 분산된 클라우드 규모 데이터 저장 및 검색에 대한 Microsoft의 전문 지식을 활용하여 비용을 줄일 수 있습니다. 이 시나리오에서는 특히 전자 상거래 또는 소매 시나리오를 다루고 있습니다. 데이터 서비스에 대한 다른 요구 사항이 있는 경우 사용 가능한 [Azure에서 완전히 관리되는 지능형 데이터베이스 서비스][product-category] 목록을 검토해야 합니다.

## <a name="related-use-cases"></a>관련 사용 사례

이 시나리오에 적합한 사용 사례는 다음과 같습니다.

* 전자 상거래 또는 소매 POS 백 엔드 시스템
* 재고 관리 시스템
* 주문 이행 시스템
* 주문 처리 파이프라인과 관련된 다른 통합 시나리오

## <a name="architecture"></a>아키텍처

![확장 가능한 주문 처리 파이프라인에 대한 예제 아키텍처][architecture-diagram]

이 아키텍처에서는 주문 처리 파이프라인의 주요 구성 요소에 대해 자세히 설명합니다. 시나리오를 통한 데이터 흐름은 다음과 같습니다.

1. 이벤트 메시지는 고객 지향 응용 프로그램(HTTP를 통해 동기적으로)과 다양한 백 엔드 시스템(Apache Kafka를 통해 비동기적으로)을 통해 시스템으로 들어갑니다. 이러한 메시지는 명령 처리 파이프라인으로 전달됩니다.
2. 각 이벤트 메시지는 명령 프로세서 마이크로 서비스에서 정의된 명령 집합 중 하나로 수집되고 매핑됩니다. 명령 프로세서는 이벤트 스트림 스냅숏 데이터베이스에서 명령을 실행하는 것과 관련된 현재 상태를 검색합니다. 그런 다음, 명령이 실행되고 명령의 출력이 새 이벤트로 내보내집니다.
3. 명령 출력으로 내보낸 각 이벤트는 Cosmos DB를 사용하여 이벤트 스트림 데이터베이스에 커밋됩니다.
4. 이벤트 스트림 데이터베이스에 커밋된 각 데이터베이스를 삽입하거나 업데이트할 때 Cosmos DB 변경 피드를 통해 이벤트가 발생합니다. 다운스트림 시스템에서 해당 시스템과 관련된 모든 이벤트 항목을 구독할 수 있습니다.
5. Cosmos DB 변경 피드의 모든 이벤트는 발생한 이벤트로 인한 모든 상태 변경을 계산하는 스냅숏 이벤트 스트림 마이크로 서비스로 보내집니다. 그런 다음, 새 상태가 Cosmos DB에 저장된 이벤트 스트림 스냅숏 데이터베이스에 커밋됩니다.  스냅숏 데이터베이스는 모든 데이터 요소의 현재 상태에 대해 전역으로 분산되고 대기 시간이 짧은 데이터 원본을 제공합니다. 이벤트 스트림 데이터베이스는 아키텍처를 통과한 모든 이벤트 메시지의 전체 기록을 제공하므로 강력한 테스트, 문제 해결 및 재해 복구 시나리오가 가능합니다.  

### <a name="components"></a>구성 요소

* [Cosmos DB][docs-cosmos-db]는 전역으로 분산된 Microsoft의 다중 모델 데이터베이스로, 솔루션을 통해 여러 지리적 지역에 걸쳐 있는 처리량과 저장소의 크기를 탄력적이고 독립적으로 조정할 수 있습니다. 포괄적인 SLA(서비스 수준 계약)를 통해 처리량, 대기 시간, 가용성 및 일관성을 보장합니다. 이 시나리오에서는 이벤트 스트림 저장소 및 스냅숏 저장소에 Cosmos DB를 사용하고, Cosmos DB의 변경 피드 기능을 활용하여 데이터 일관성 및 오류 복구를 제공합니다. 
* [HDInsight의 Apache Kafka][docs-kafka]는 실시간 스트리밍 데이터 파이프라인 및 응용 프로그램을 구축하기 위한 오픈 소스 분산 스트리밍 플랫폼인 Apache Kafka의 관리 서비스 구현입니다. 또한 Kafka는 명명된 데이터 스트림을 게시하고 구독하기 위해 메시지 큐와 비슷한 메시지 브로커 기능을 제공합니다. 이 시나리오에서는 Kafka를 사용하여 주문 처리 파이프라인에서 들어오는 이벤트와 다운스트림 이벤트를 처리합니다. 

## <a name="considerations"></a>고려 사항

다양한 기술 옵션을 통해 실시간 메시지 수집, 데이터 저장, 스트림 처리, 분석 데이터 저장, 분석 및 보고를 수행할 수 있습니다. 이러한 옵션, 기능 및 주요 선택 조건에 대한 개요는 [Azure 데이터 아키텍처 가이드](/azure/architecture/data-guide/)의 [빅 데이터 아키텍처: 실시간 처리](/azure/architecture/data-guide/technology-choices/real-time-ingestion)를 참조하세요.

마이크로 서비스는 복원력이 있고, 확장성이 뛰어나며, 독립적으로 배포할 수 있고, 신속하게 진화할 수 있는 클라우드 응용 프로그램을 구축하는 데 널리 사용되는 아키텍처 스타일이 되었습니다. 마이크로 서비스에는 응용 프로그램을 설계하고 구축하는 데 있어 다른 방법이 필요합니다. Docker, Kubernetes, Azure Service Fabric 및 Nomad와 같은 도구를 사용하면 마이크로 서비스 기반 아키텍처를 개발할 수 있습니다. 마이크로 서비스 기반 아키텍처의 구축 및 실행에 대한 지침은 Azure 아키텍처 센터의 [Azure에서 마이크로 서비스 설계](/azure/architecture/microservices/)를 참조하세요.

### <a name="availability"></a>가용성

이 시나리오의 이벤트 소싱 방식을 통해 시스템 구성 요소를 느슨하게 결합하고 서로 독립적으로 배포할 수 있습니다. Cosmos DB는 [고가용성][docs-cosmos-db-regional-failover]을 제공하고 조직에서 일관성, 가용성 및 성능과 관련된 절충 작업을 [해당 보장][docs-cosmos-db-guarantees]으로 모두 관리하는 데 도움을 줍니다. HDInsight의 Apache Kafka도 [고가용성][docs-kafka-high-availability]을 위해 설계되었습니다.

Azure Monitor는 다양한 Azure 서비스를 모니터링하기 위한 통합된 사용자 인터페이스를 제공합니다. 자세한 내용은 [Microsoft Azure에서 모니터링](/azure/monitoring-and-diagnostics/monitoring-overview)을 참조하세요. Event Hubs 및 Stream Analytics는 모두 Azure Monitor와 통합됩니다. 

다른 가용성 고려 사항은 [가용성 검사 목록][availability]을 참조하세요.

### <a name="scalability"></a>확장성

HDInsight의 Kafka는 Kafka 클러스터에 대한 [저장소 구성 및 확장성](/azure/hdinsight/kafka/apache-kafka-scalability)을 허용합니다. Cosmos DB는 빠르고 예측 가능한 성능을 제공하고, 응용 프로그램이 확장함에 따라 크기를 [원활하게 조정합니다](/azure/cosmos-db/partition-data).
이 시나리오의 마이크로 서비스 기반 아키텍처를 소싱하는 이벤트에서도 시스템을 더 쉽게 크기 조정하고 해당 기능을 확장할 수 있습니다.

다른 확장성 고려 사항은 Azure 아키텍처 센터에서 사용할 수 있는 [확장성 검사 목록][scalability]을 참조하세요.

### <a name="security"></a>보안

[Cosmos DB 보안 모델](/azure/cosmos-db/secure-access-to-data)은 사용자를 인증하고 해당 데이터 및 리소스에 대한 액세스를 제공합니다. 자세한 내용은 [Cosmos DB 데이터베이스 보안](/en-us/azure/cosmos-db/database-security)을 참조하세요.

보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.

### <a name="resiliency"></a>복원력

이 예제 시나리오에서 이벤트 소싱 아키텍처 및 관련 기술을 사용하면 오류 발생 시 복원력이 매우 높은 시나리오로 만들 수 있습니다. 복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 응용 프로그램 디자인][resiliency]을 참조하세요.

## <a name="pricing"></a>가격

이 시나리오를 실행하는 데 드는 비용을 알아보려면 비용 계산기에서 모든 서비스를 미리 구성합니다.  특정 시나리오에 대한 가격 변동을 확인하려면 필요한 데이터 양에 맞게 변수를 적절하게 변경합니다. 이 시나리오에 대한 예제 가격 책정에는 Cosmos DB 변경 피드에서 발생한 이벤트를 처리하는 Cosmos DB 및 Kafka 클러스터만 포함됩니다. 원본 시스템 및 기타 다운스트림 시스템의 이벤트 프로세서 및 마이크로 서비스는 포함되지 않으며, 비용은 이러한 서비스를 구현하기 위해 선택한 기술뿐만 아니라 해당 서비스의 양과 규모에 따라 크게 달라집니다.

Azure Cosmos DB의 통화는 RU(요청 단위)입니다. 요청 단위를 사용하면 읽기/쓰기 용량을 예약하거나 CPU, 메모리 및 IOPS를 프로비전할 필요가 없습니다. Azure Cosmos DB는 단순한 읽기 및 쓰기부터 복잡한 그래프 쿼리에 이르기까지 다양한 작업에서 많은 API를 지원합니다. 모든 요청 값이 같지 않으므로 요청을 처리하는 데 필요한 계산의 양에 기반하여 정규화된 양의 요청 단위가 할당됩니다. 솔루션에 필요한 요청 단위의 수는 데이터 요소 크기 및 초당 데이터베이스 읽기/쓰기 작업 수에 따라 다릅니다. 자세한 내용은 [Azure Cosmos DB의 요청 단위](/azure/cosmos-db/request-units)를 참조하세요. 이러한 예상 가격은 두 Azure 지역에서 실행되는 Cosmos DB를 기반으로 합니다.

필요한 활동량을 기준으로 제공한 세 가지 샘플 비용 프로필은 다음과 같습니다.

* [소형][small-pricing]: Cosmos DB의 1TB 데이터 저장소 및 소형(D3 v2) Kafka 클러스터로 예약된 5개 RU와 관련이 있습니다.
* [중형][medium-pricing]: Cosmos DB의 10TB 데이터 저장소 및 중형(D4 v2) Kafka 클러스터로 예약된 50개 RU와 관련이 있습니다.
* [대형][large-pricing]: Cosmos DB의 30TB 데이터 저장소 및 대형(D5 v2) Kafka 클러스터로 예약된 500개 RU와 관련이 있습니다.

## <a name="related-resources"></a>관련 리소스

이 예제 시나리오는 종단 간 주문 처리 파이프라인에 대해 [Jet.com](https://jet.com)에서 구축된 이 아키텍처의 더 광범위한 버전을 기반으로 합니다. 자세한 내용은 [jet.com 기술 고객 프로필][source-document] 및 [Build 2018에 있는 jet.com의 프레젠테이션][source-presentation]을 참조하세요. 

기타 관련 리소스는 다음과 같습니다.
* _[데이터 집약적인 응용 프로그램 설계](https://dataintensive.net/)_ - Martin Kleppmann 작성, O'Reilly Media, 2017
* _[기능적으로 만든 도메인 모델링: 도메인 기반 디자인 및 F#으로 소프트웨어 복잡성 해결](https://pragprog.com/book/swdddf/domain-modeling-made-functional)_ - Scott Wlaschin 작성, Pragmatic Programmers LLC, 2018
* 다른 [Cosmos DB 사용 사례][docs-cosmos-db-use-cases]
* [Azure 데이터 아키텍처 가이드](/azure/architecture/data-guide/)의 [실시간 처리 아키텍처](/azure/architecture/data-guide/big-data/real-time-processing)

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/databases/
[source-document]: https://customers.microsoft.com/en-us/story/jet-com-powers-innovative-e-commerce-engine-on-azure-in-less-than-12-months
[source-presentation]: https://channel9.msdn.com/events/Build/2018/BRK3602
[small-pricing]: https://azure.com/e/3d43949ffbb945a88cc0a126dc3a0e6e
[medium-pricing]: https://azure.com/e/1f1e7bf2a6ad4f7799581211f4369b9b
[large-pricing]: https://azure.com/e/75207172ece94cf6b5fb354a2252b333
[architecture-diagram]: ./images/architecture-diagram-cosmos-db.png
[docs-cosmos-db]: /azure/cosmos-db
[docs-cosmos-db-change-feed]: /azure/cosmos-db/change-feed
[docs-cosmos-db-regional-failover]: /azure/cosmos-db/regional-failover
[docs-cosmos-db-guarantees]: /azure/cosmos-db/distribute-data-globally#AvailabilityGuarantees
[docs-cosmos-db-use-cases]: /azure/cosmos-db/use-cases
[docs-kafka]: /azure/hdinsight/kafka/apache-kafka-introduction
[docs-kafka-high-availability]: /azure/hdinsight/kafka/apache-kafka-high-availability
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-blob-storage]: /azure/storage/blobs/storage-blobs-introduction
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: /azure/architecture/patterns/category/resiliency/
[security]: /azure/security/
