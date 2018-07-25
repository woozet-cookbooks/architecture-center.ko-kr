---
title: Azure에서 실시간 부정 행위 감지
description: Azure Event Hubs 및 Stream Analytics를 사용하여 부정 행위를 실시간으로 감지하는 데 입증된 시나리오입니다.
author: alexbuckgit
ms.date: 07/05/2018
ms.openlocfilehash: e22322133adf40d033ac5af98069cb00765d14ca
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060815"
---
# <a name="real-time-fraud-detection-on-azure"></a>Azure에서 실시간 부정 행위 감지

이 예제 시나리오는 데이터를 실시간으로 분석하여 부정 행위 거래 또는 기타 비정상적인 활동을 감지해야 하는 조직과 관련이 있습니다.

잠재적인 응용 프로그램에는 신용 카드 부정 행위 또는 휴대폰 전화를 식별하는 기능이 포함됩니다. 기존 온라인 분석 시스템에서 데이터를 변환하고 분석하여 비정상적인 활동을 식별하는 데 몇 시간이 걸릴 수 있습니다.

Event Hubs 및 Stream Analytics와 같이 완전하게 관리되는 Azure 서비스를 사용하면, 개별 서버를 관리할 필요가 없으며, 비용을 줄이고, 클라우드 규모의 데이터 수집 및 실시간 분석에 대한 Microsoft의 전문 지식을 활용할 수 있습니다. 이 시나리오에서는 특히 부정 행위 감지에 대해 다루고 있습니다. 데이터 분석에 대한 다른 요구 사항이 있으면 사용 가능한 [Azure Analytics 서비스][product-category] 목록을 검토해야 합니다.

이 샘플은 광범위한 데이터 처리 아키텍처 및 전략의 한 부분을 나타냅니다. 전체 아키텍처에서 이 측면에 대한 다른 옵션은 이 문서의 뒷부분에서 설명합니다.

## <a name="related-use-cases"></a>관련 사용 사례

이 시나리오에 적합한 사용 사례는 다음과 같습니다.

* 통신 시나리오에서 휴대폰의 부정 전화 행위 감지
* 은행 기관에서 신용 카드의 부정 거래 행위 식별
* 소매 또는 전자 상거래 시나리오에서 부정 구매 행위 식별

## <a name="architecture"></a>아키텍처

![실시간 부정 행위 감지 시나리오의 Azure 구성 요소 아키텍처에 대한 개요][architecture-diagram]

이 시나리오에서는 실시간 분석 파이프라인의 백 엔드 구성 요소에 대해 설명합니다. 시나리오를 통한 데이터 흐름은 다음과 같습니다.

1. 휴대폰 호출 메타데이터는 원본 시스템에서 Azure Event Hubs 인스턴스로 보내집니다. 
2. 이벤트 허브 원본을 통해 데이터를 받는 Stream Analytics 작업이 시작됩니다.
3. Stream Analytics 작업은 미리 정의된 쿼리를 실행하여 입력 스트림을 변환하고 부정 행위 트랜잭션 알고리즘에 따라 분석합니다. 이 쿼리는 연속 창을 사용하여 스트림을 개별 시간 단위로 구분합니다.
4. Stream Analytics 작업은 감지된 부정 행위 호출을 나타내는 변환된 스트림을 Azure Blob 저장소의 출력 싱크에 기록합니다.

### <a name="components"></a>구성 요소

* [Azure Event Hubs][docs-event-hubs]는 초당 수백만 개의 이벤트를 받고 처리할 수 있는 실시간 스트리밍 플랫폼 및 이벤트 수집 서비스입니다. Event Hubs는 분산된 소프트웨어와 장치에서 생성된 이벤트, 데이터 또는 원격 분석을 처리하고 저장할 수 있습니다. 이 시나리오에서 Event Hubs에서 모든 전화 통화 메타데이터를 수신하여 부정 행위에 대해 분석합니다.
* [Azure Stream Analytics][docs-stream-analytics]는 장치 및 다른 데이터 원본에서 스트림하는 대량의 데이터를 분석할 수 있는 이벤트 처리 엔진입니다. 또한 데이터 스트림에서 정보를 추출하여 패턴과 관계를 식별할 수 있습니다. 이러한 패턴은 다른 다운스트림 작업을 트리거할 수 있습니다. 이 시나리오에서는 Stream Analytics에서 Event Hubs로부터의 입력 스트림을 변환하여 부정 행위 호출을 식별합니다.
* [Blob 저장소][docs-blob-storage]는 Stream Analytics 작업의 결과를 저장하기 위해 이 시나리오에서 사용됩니다.

## <a name="considerations"></a>고려 사항

### <a name="alternatives"></a>대안

실시간 메시지 수집, 데이터 저장, 스트림 처리, 분석 데이터 저장, 분석 및 보고에 많은 기술을 선택하여 사용할 수 있습니다. 이러한 옵션, 기능 및 주요 선택 기준에 대한 개요는 Azure 데이터 아키텍처 가이드의 [빅 데이터 아키텍처: 실시간 처리](/azure/architecture/data-guide/technology-choices/real-time-ingestion)를 참조하세요.

또한 부정 행위 감지에 대해 더 복잡한 알고리즘은 Azure의 다양한 기계 학습 서비스에서 생성할 수 있습니다. 이러한 옵션에 대한 개요는 [Azure 데이터 아키텍처 가이드](../../data-guide/index.md)의 [기계 학습에 대한 기술 선택](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning)을 참조하세요.

### <a name="availability"></a>가용성

Azure Monitor는 다양한 Azure 서비스를 모니터링하기 위한 통합된 사용자 인터페이스를 제공합니다. 자세한 내용은 [Microsoft Azure에서 모니터링](/azure/monitoring-and-diagnostics/monitoring-overview)을 참조하세요. Event Hubs 및 Stream Analytics는 모두 Azure Monitor와 통합됩니다. 

다른 가용성 고려 사항에 대해서는 Azure 아키텍처 센터의 [가용성 검사 목록][availability]을 참조하세요.

### <a name="scalability"></a>확장성

이 시나리오의 구성 요소는 하이퍼스케일 수집 및 대규모 병렬 실시간 분석을 위해 설계되었습니다. Azure Event Hubs는 확장성이 뛰어나고, 대기 시간이 짧고 초당 수백만 개의 이벤트를 받고 처리할 수 있습니다.  Event Hubs는 사용량 요구 사항에 맞게 처리량 단위 수를 [자동으로 확장](/azure/event-hubs/event-hubs-auto-inflate)할 수 있습니다. Azure Stream Analytics는 많은 원본에서 대량의 스트리밍 데이터를 분석할 수 있습니다. 스트리밍 작업을 실행하기 위해 할당된 [스트리밍 단위](/azure/stream-analytics/stream-analytics-streaming-unit-consumption)의 수를 늘려 Stream Analytics를 확장할 수 있습니다.

확장 가능한 시나리오 설계에 대한 일반적인 지침은 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.

### <a name="security"></a>보안

Azure Event Hubs는 SAS(공유 액세스 서명) 토큰과 이벤트 게시자의 조합에 기반한 [인증 및 보안 모델][docs-event-hubs-security-model]을 통해 데이터를 보호합니다. 이벤트 게시자는 이벤트 허브에 대한 가상 끝점을 정의합니다. 게시자는 이벤트 허브에 메시지를 보내는 데만 사용할 수 있습니다. 게시자에서 메시지를 받을 수 없습니다.

보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.

### <a name="resiliency"></a>복원력

복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 응용 프로그램 디자인][resiliency]을 참조하세요.

## <a name="deploy-the-scenario"></a>시나리오 배포

이 시나리오를 배포하려면 이 [단계별 자습서][tutorial]에 따라 시나리오의 각 구성 요소를 수동으로 배포하는 방법을 시연할 수 있습니다. 또한 이 자습서에서는 .NET 클라이언트 응용 프로그램을 제공하여 샘플 전화 호출 메타데이터를 생성하고 해당 데이터를 이벤트 허브 인스턴스로 보냅니다.

## <a name="pricing"></a>가격

이 시나리오를 실행하는 데 들어가는 비용을 알아보기 위해 모든 서비스가 비용 계산기에서 미리 구성됩니다. 특정 사용 사례에 대한 가격이 변경되는 정도를 확인하려면 필요한 데이터 양에 맞게 적절한 변수를 변경합니다.

가져오는 데 필요한 트래픽 양을 기준으로 다음 세 가지 샘플 비용 프로필을 제공했습니다.

* [소량][small-pricing]: 매월 1개 표준 스트리밍 단위를 통해 백만 개의 이벤트를 처리합니다.
* [중간][medium-pricing]: 매월 5개 표준 스트리밍 단위를 통해 1억 개의 이벤트를 처리합니다.
* [대량][large-pricing]: 매월 20개 표준 스트리밍 단위를 통해 9억 9천 9백만 개의 이벤트를 처리합니다.

## <a name="related-resources"></a>관련 리소스

더 복잡한 부정 행위 감지 시나리오는 기계 학습 모델을 통해 이점을 누릴 수 있습니다. Machine Learning Server를 사용하여 구축된 시나리오는 [Machine Learning Server를 사용하여 부정 행위 감지][r-server-fraud-detection]를 참조하세요. Machine Learning Server를 사용하는 다른 솔루션 템플릿은 [데이터 과학 시나리오 및 솔루션 템플릿][docs-r-server-sample-solutions]을 참조하세요. Azure Data Lake Analytics를 사용하는 예제 솔루션은 [부정 행위 감지에 Azure Data Lake 및 R 사용][technet-fraud-detection]을 참조하세요.  

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[tutorial]: /azure/stream-analytics/stream-analytics-real-time-fraud-detection
[small-pricing]: https://azure.com/e/74149ec312c049ccba79bfb3cfa67606
[medium-pricing]: https://azure.com/e/4fc94f7376de484d8ae67a6958cae60a
[large-pricing]: https://azure.com/e/7da8804396f9428a984578700003ba42
[architecture-diagram]: ./images/architecture-diagram-fraud-detection.png
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-event-hubs-security-model]: /azure/event-hubs/event-hubs-authentication-and-security-model-overview
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-blob-storage]: /azure/storage/blobs/storage-blobs-introduction
[docs-r-server-sample-solutions]: /machine-learning-server/r/sample-solutions
[r-server-fraud-detection]: https://microsoft.github.io/r-server-fraud-detection/
[technet-fraud-detection]: https://blogs.technet.microsoft.com/machinelearning/2017/06/28/using-azure-data-lake-and-r-for-fraud-detection/
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/

