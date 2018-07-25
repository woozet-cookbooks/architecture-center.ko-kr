---
title: Azure에서 보험 청구에 대한 이미지 분류
description: Azure 응용 프로그램에 이미지 처리를 구축하는 데 입증된 시나리오입니다.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 361a88234fd9ed918ab7664893f86666b4328b8c
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060832"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a>Azure에서 보험 청구에 대한 이미지 분류

이 예제 시나리오는 이미지를 처리해야 하는 비즈니스에 적용할 수 있습니다.

잠재적인 응용 프로그램에는 패션 웹 사이트에 대한 이미지 분류, 보험 청구에 대한 텍스트 및 이미지 분석 또는 게임 스크린샷의 원격 분석 데이터 인식이 포함됩니다. 일반적으로 회사에서는 기계 학습 모델에 대한 전문 지식을 개발하고, 모델을 학습하며, 마지막으로 사용자 지정 프로세스를 통해 이미지를 실행하여 이미지에서 데이터를 가져와야 합니다.

Computer Vision API 및 Azure Functions와 같은 Azure 서비스를 사용하면, 개별 서버를 관리할 필요가 없으며, 비용을 줄이고, Microsoft에서 Cognitive Services를 사용하여 이미지를 처리할 때 이미 개발한 전문 지식을 활용할 수 있습니다. 이 시나리오에서는 특히 이미지 처리 시나리오에 대해 다루고 있습니다. 다양한 AI 요구 사항이 있으면 [Cognitive Services][cognitive-docs]의 전체 제품군을 사용하는 것이 좋습니다.

## <a name="related-use-cases"></a>관련 사용 사례

이 시나리오에 적합한 사용 사례는 다음과 같습니다.

* 패션 웹 사이트에 대한 이미지 분류
* 게임의 스크린샷의 원격 분석 데이터 분류

## <a name="architecture"></a>아키텍처

![지능형 앱 아키텍처 - Computer Vision][architecture-computer-vision]

이 시나리오에서는 웹 또는 모바일 응용 프로그램의 백 엔드 구성 요소에 대해 설명합니다. 시나리오를 통한 데이터 흐름은 다음과 같습니다.

1. Azure Functions는 API 계층으로 작동합니다. 이러한 API를 통해 응용 프로그램은 Cosmos DB에서 이미지를 업로드하고 데이터를 검색할 수 있습니다.

2. 이미지가 API 호출을 통해 업로드되면 Blob 저장소에 저장됩니다.

3. Blob 저장소에 새 파일이 추가되면 Azure Function에 EventGrid 알림을 보냅니다.

4. Azure Functions에서 분석을 위해 새로 업로드된 파일에 대한 링크를 Computer Vision API로 보냅니다.

5. 데이터가 Computer Vision API로부터 반환되면 Azure Functions에서 Cosmos DB에 항목을 만들어 이미지 메타데이터와 함께 분석 결과를 유지합니다.

### <a name="components"></a>구성 요소

* [Computer Vision API][computer-vision-docs]는 Cognitive Services 제품군의 일부이며 각 이미지에 대한 정보를 검색하는 데 사용됩니다.

* [Azure Functions][functions-docs]는 업로드된 이미지에 대한 이벤트 처리뿐만 아니라 웹 응용 프로그램에 대한 백 엔드 API도 제공합니다.

* [Event Grid][eventgrid-docs]는 Blob 저장소에 새 이미지가 업로드되면 이벤트를 트리거합니다. 그런 다음, 이미지는 Azure Functions에서 처리됩니다.

* [Blob Storage][storage-docs]는 웹 응용 프로그램에 업로드되는 모든 이미지 파일과 웹 응용 프로그램에서 사용하는 모든 정적 파일을 저장합니다.

* [Cosmos DB][cosmos-docs]는 Computer Vision API에서 처리한 결과를 포함하여 업로드된 각 이미지에 대한 메타데이터를 저장합니다.

## <a name="alternatives"></a>대안

* [Custom Vision Service][custom-vision-docs]. Computer Vision API는 일단의 [분류 기반 범주][cv-categories]를 반환합니다. Computer Vision API에서 반환하지 않는 정보를 처리해야 하는 경우 사용자 지정 이미지 분류기를 만들 수 있는 Custom Vision Service를 사용하는 것이 좋습니다.

* [Azure Search][azure-search-docs]. 특정 조건을 충족하는 이미지를 찾기 위해 메타데이터를 쿼리하는 사용 사례가 있는 경우 Azure Search를 사용하는 것이 좋습니다. 현재 미리 보기에서 [인지 검색][cognitive-search]은 이 워크플로를 원활하게 통합합니다.

## <a name="considerations"></a>고려 사항

### <a name="scalability"></a>확장성

대부분의 경우 이 시나리오의 모든 구성 요소는 자동으로 크기 조정되는 관리 서비스입니다. 몇 가지 주목할 만한 예외: Azure Functions에는 최대 200개의 인스턴스 제한이 있습니다. 이 제한을 초과하여 확장해야 하는 경우 여러 지역 또는 앱 계획을 사용하는 것이 좋습니다.

Cosmos DB는 프로비전된 RU(요청 단위)를 기준으로 자동으로 크기 조정되지 않습니다.  요구 사항 추정에 대한 지침은 설명서의 [요청 단위][request-units]를 참조하세요. Cosmos DB의 크기 조정을 최대한 활용하려면 [파티션 키][partition-key]도 확인해야 합니다.

NoSQL 데이터베이스는 가용성, 확장성 및 파티션에 대한 일관성을 자주 교환합니다(CAP 정리의 의미에서).  그러나 이 시나리오에서 사용되는 키-값 데이터 모델의 경우 대부분의 작업이 원자성 정의이므로 트랜잭션 일관성은 거의 필요하지 않습니다. [적절한 데이터 저장소 선택](../../guide/technology-choices/data-store-overview.md)에 대한 추가 지침은 아키텍처 센터에서 사용할 수 있습니다.

확장 가능한 솔루션 설계에 대한 일반적인 지침은 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.

### <a name="security"></a>보안

[MSI][msi](관리 서비스 ID)는 계정 내부의 다른 리소스에 대한 액세스를 제공한 다음, Azure Functions에 할당되는 데 사용됩니다. 이러한 ID에 필요한 리소스에만 액세스할 수 있도록 허용하여 추가적으로 사용자의 함수에(그리고 잠재적으로 고객에게 ) 노출되지 않도록 합니다.  

보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.

### <a name="resiliency"></a>복원력

이 시나리오의 모든 구성 요소가 관리되므로 모두 지역 수준에서 자동으로 복원됩니다.

복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 응용 프로그램 디자인][resiliency]을 참조하세요.

## <a name="pricing"></a>가격

이 시나리오를 실행하는 데 들어가는 비용을 알아보기 위해 모든 서비스가 비용 계산기에서 미리 구성됩니다. 특정 사용 사례에 대한 가격이 변경되는 정도를 확인하려면 필요한 트래픽에 맞게 적절한 변수를 변경합니다.

트래픽 양을 기준으로 다음 세 가지 샘플 비용 프로필을 제공했습니다(모든 이미지가 100kb 크기라고 가정).

* [소량][pricing]: 매월 5,000개의 이미지 처리와 관련이 있습니다.
* [중간][medium-pricing]: 매월 50만 개의 이미지 처리와 관련이 있습니다.
* [대규모][large-pricing]: 매월 5천만 개의 이미지 처리와 관련이 있습니다.

## <a name="related-resources"></a>관련 리소스

이 시나리오의 단계별 학습 경로는 [Azure에서 서버를 사용하지 않는 웹앱 빌드][serverless]를 참조하세요.  

이 솔루션을 프로덕션 환경에 배치하기 전에 Azure Functions [모범 사례][functions-best-practices]를 검토하세요.

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data