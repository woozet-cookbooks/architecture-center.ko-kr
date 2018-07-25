---
title: Azure의 전자 상거래 프런트 엔드
description: Azure에서 전자 상거래 사이트를 호스팅하는 데 입증된 시나리오입니다.
author: masonch
ms.date: 7/13/18
ms.openlocfilehash: 568821e97c6b90a36429dfa8ec0ef9ed38c7963c
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060974"
---
# <a name="e-commerce-front-end-on-azure"></a>Azure의 전자 상거래 프런트 엔드

이 예제 시나리오에서는 Azure PaaS(Platform-as-a-Service) 도구를 사용하여 전자 상거래 프런트 엔드를 구현하는 과정을 안내합니다. 대부분의 전자 상거래 웹 사이트는 시간 경과에 따른 계절성 및 트래픽 가변성에 직면하고 있습니다. 제품이나 서비스에 대한 수요가 예측 가능 여부에 관계없이 급격히 증가하는 경우 PaaS 도구를 사용하면 더 많은 고객과 더 많은 거래를 자동으로 처리할 수 있습니다. 또한 이 시나리오에서는 사용하는 용량에 대해서만 지불함으로써 경제성을 활용합니다.

이 문서에서는 온라인 콘서트 발권 플랫폼인 *Relecloud Concerts*의 전자 상거래 응용 프로그램 샘플을 배포하는 데 함께 사용되는 다양한 Azure PaaS 구성 요소 및 고려 사항에 대해 알아볼 수 있습니다.

## <a name="potential-use-cases"></a>잠재적인 사용 사례

이 시나리오에 적합한 사용 사례는 다음과 같습니다.

* 서로 다른 시간에 갑자기 집중되는 사용자를 처리할 수 있는 탄력적인 크기 조정이 필요한 응용 프로그램 구축
* 전 세계의 여러 Azure 지역에서 고가용성으로 작동하도록 설계된 프로그램 구축

## <a name="architecture"></a>아키텍처

![전자 상거래 응용 프로그램에 대한 샘플 시나리오 아키텍처][architecture-diagram]

이 시나리오에서는 전자 상거래 사이트에서 티켓을 구매하는 방법에 대해 설명하며, 시나리오를 통한 데이터 흐름은 다음과 같습니다.

1. Azure Traffic Manager에서 사용자의 요청을 Azure App Service에서 호스팅되는 전자 상거래 사이트로 라우팅합니다.
2. Azure CDN에서 사용자에게 정적 이미지와 콘텐츠를 제공합니다.
3. 사용자가 Azure Active Directory B2C 테넌트를 통해 응용 프로그램에 로그인합니다.
4. 사용자가 Azure Search를 사용하여 콘서트를 검색합니다.
5. 웹 사이트에서 Azure SQL Database로부터 콘서트 세부 정보를 가져옵니다. 
6. 웹 사이트에서 Blob Storage에 있는 구매한 티켓 이미지를 참조합니다.
7. 데이터베이스 쿼리 결과는 더 나은 성능을 위해 Azure Redis Cache에 캐시됩니다.
8. 사용자가 큐에 있는 티켓 주문 및 콘서트 리뷰를 제출합니다.
9. Azure Functions에서 주문 결제 및 콘서트 리뷰를 처리합니다.
10. Cognitive Services에서 콘서트 리뷰에 대한 분석을 제공하여 감정(긍정 또는 부정)을 결정합니다.
11. Application Insights에서 웹 응용 프로그램의 상태를 모니터링하기 위한 성능 메트릭을 제공합니다.

### <a name="components"></a>구성 요소

* [Azure CDN][docs-cdn]은 사용자와 가까운 위치에서 캐시된 정적 콘텐츠를 제공하여 대기 시간을 줄입니다.
* [Azure Traffic Manager][docs-traffic-manager]는 다른 Azure 지역의 서비스 엔드포인트에 대한 사용자 트래픽 분산을 제어합니다.
* [App Services - Web Apps][docs-webapps]는 인프라를 관리할 필요 없이 자동 크기 조정 및 고가용성을 허용하는 웹 응용 프로그램을 호스팅합니다.
* [Azure Active Directory - B2C][docs-b2c]는 고객이 응용 프로그램에서 자신의 프로필을 등록, 로그인 및 관리하는 방법을 사용자 지정하고 제어할 수 있는 ID 관리 서비스입니다.
* [Storage Queues][docs-storage-queues]는 응용 프로그램에서 액세스할 수 있는 많은 수의 큐 메시지를 저장합니다.
* [Functions][docs-functions]는 응용 프로그램에서 인프라를 관리할 필요 없이 주문형으로 실행할 수 있도록 하는 서버리스 계산 옵션입니다.
* [Cognitive Services - 감정 분석][docs-sentiment-analysis]은 기계 학습 API를 사용하고, 개발자가 응용 프로그램에 지능형 기능(예: 감정/비디오 감지, 얼굴/음성/시각 인식, 음성/언어 이해)을 쉽게 추가할 수 있게 합니다.
* [Azure Search][docs-search]는 웹, 모바일 및 엔터프라이즈 응용 프로그램에서 이질적인 비공개 콘텐츠에 대한 풍부한 검색 환경을 제공하는 검색 기반 클라우드 솔루션입니다.
* [저장소 Blob][docs-storage-blobs]은 텍스트 또는 이진 데이터와 같은 많은 양의 구조화되지 않은 데이터를 저장하도록 최적화됩니다.
* [Redis Cache][docs-redis-cache]는 응용 프로그램 가까이에 있는 고속 저장소에 자주 액세스하는 데이터를 일시적으로 복사하여 백 엔드 데이터 저장소를 많이 사용하는 시스템의 성능과 확장성을 향상시킵니다.
* [SQL Database][docs-sql-database]는 관계형 데이터, JSON, 공간 및 XML과 같은 구조를 지원하는 Microsoft Azure의 범용 관계형 데이터베이스 관리 서비스입니다.
* [Application Insights][docs-application-insights]는 사용자가 앱에서 수행하는 작업을 파악하는 데 도움이 되는 기본 제공 분석 도구를 통해 성능 이상을 자동으로 감지하여 성능 및 유용성을 지속적으로 향상시킬 수 있도록 설계되었습니다.

### <a name="alternatives"></a>대안

대규모 전자 상거래에 집중하는 고객 지향 응용 프로그램을 구축하는 데 사용할 수 있는 많은 기술이 있습니다. 여기에는 응용 프로그램의 프론트 엔드와 데이터 계층이 모두 포함됩니다.

웹 계층 및 기능에 대한 다른 옵션은 다음과 같습니다.

* [Service Fabric][docs-service-fabric] - 높은 수준의 제어를 통해 클러스터 전체에 배포되고 실행되는 이점이 있는 분산 구성 요소를 구축하는 데 중점을 둔 플랫폼입니다. 또한 컨테이너를 호스팅하는 데도 사용할 수 있습니다.
* [Azure Kubernetes Service][docs-kubernetes-service] - 마이크로 서비스 아키텍처의 한 구현으로 사용할 수 있는 컨테이너 기반 솔루션을 구축하고 배포하는 플랫폼입니다. 필요에 따라 응용 프로그램의 여러 구성 요소를 독립적으로 민첩하게 크기 조정할 수 있습니다.
* [Azure Container Instances][docs-container-instances] - 짧은 수명 주기의 컨테이너를 빠르게 배포하고 실행할 수 있습니다. 여기에 있는 컨테이너는 일반적으로 메시지 처리 또는 계산 수행과 같은 빠른 처리 작업을 실행하기 위해 배포된 다음, 완료되는 즉시 프로비전 해제됩니다.
* [Service Bus][service-bus]는 저장소 큐를 대신하여 사용할 수 있습니다.

데이터 계층에 대한 다른 옵션은 다음과 같습니다.

* [Cosmos DB][docs-cosmosdb] - 전역으로 분산된 Microsoft의 다중 모델 데이터베이스입니다. Mongo DB, Cassandra, Graph 데이터 또는 간단한 테이블 저장소와 같은 다른 데이터 모델을 실행하는 플랫폼을 제공합니다.

## <a name="considerations"></a>고려 사항

### <a name="availability"></a>가용성

* 클라우드 응용 프로그램을 구축하는 경우 [일반적인 가용성 디자인 패턴][design-patterns-availability]을 활용하는 것이 좋습니다.
* 적절한 [App Service 웹 응용 프로그램 참조 아키텍처][app-service-reference-architecture]의 가용성 고려 사항을 검토합니다.
* 가용성에 대한 추가 고려 사항은 아키텍처 센터의 [가용성 검사 목록][availability]을 참조하세요.

### <a name="scalability"></a>확장성

* 클라우드 응용 프로그램을 구축하는 경우 [일반적인 확장성 디자인 패턴][design-patterns-scalability]에 대해 알고 있어야 합니다.
* 적절한 [App Service 웹 응용 프로그램 참조 아키텍처][app-service-reference-architecture]의 확장성 고려 사항을 검토합니다.
* 다른 확장성 항목은 아키텍처 센터에서 사용할 수 있는 [확장성 검사 목록][scalability]을 참조하세요.

### <a name="security"></a>보안

* 적절한 경우 [일반적인 보안 디자인 패턴][design-patterns-security]을 활용하는 것이 좋습니다.
* 적절한 [App Service 웹 응용 프로그램 참조 아키텍처][app-service-reference-architecture]의 보안 고려 사항을 검토합니다.
* 개발자가 [보안 개발 수명 주기][secure-development] 프로세스에 따라 보안 소프트웨어를 구축하고 개발 비용을 줄이면서 보안 준수 요구 사항을 처리할 수 있도록 하는 것이 좋습니다.
* [Azure PCI DSS 규정 준수][pci-dss-blueprint]에 대한 청사진 아키텍처를 검토합니다.

### <a name="resiliency"></a>복원력

* 응용 프로그램의 일부를 사용할 수 없는 경우 [회로 차단기 패턴][circuit-breaker]을 활용하여 정상적인 오류 처리를 제공하는 것이 좋습니다.
* [일반적인 복원력 디자인 패턴][design-patterns-resiliency]을 검토하고, 적절할 경우 이를 구현하는 것이 좋습니다.
* 아키텍처 서비스 센터에서 [App Service에 대한 다양한 복원력 권장 사례][resiliency-app-service]를 찾을 수 있습니다.
* 데이터 계층에는 활성 [지역 복제][sql-geo-replication], 이미지 및 큐에는 [지역 중복][storage-geo-redudancy] 저장소를 사용하는 것이 좋습니다.
* [복원력][resiliency]에 대한 심층적인 논의는 아키텍처 센터의 관련 문서를 참조하세요.

## <a name="deploy-the-scenario"></a>시나리오 배포

이 시나리오를 배포하려면 이 [단계별 자습서][end-to-end-walkthrough]에 따라 각 구성 요소를 수동으로 배포하는 방법을 시연할 수 있습니다. 이 자습서에서는 간단한 티켓 구매 응용 프로그램을 실행하는 .NET 샘플 응용 프로그램도 제공합니다. 또한 Azure 리소스 대부분의 배포를 자동화하는 ARM 템플릿도 있습니다.

## <a name="pricing"></a>가격

이 시나리오를 실행하는 데 드는 비용을 알아보려면 비용 계산기에서 모든 서비스를 미리 구성합니다. 특정 사용 사례에 대한 가격 변동을 확인하려면 필요한 트래픽에 맞게 변수를 적절하게 변경합니다.

가져오는 데 필요한 트래픽 양을 기준으로 다음 세 가지 샘플 비용 프로필을 제공했습니다.

* [소형][small-pricing]: 최소 프로덕션 수준 인스턴스를 구축하는 데 필요한 구성 요소를 나타냅니다. 여기서는 매월 수천 명에 불과한 적은 수의 사용자를 가정합니다. 앱에서 자동 크기 조정을 사용하는 데 충분한 표준 웹앱의 단일 인스턴스를 사용합니다. 다른 각 구성 요소는 최소 비용을 허용하지만 SLA를 지원하고 프로덕션 수준 워크로드를 처리할 수 있을 만큼 충분한 용량을 보장하는 기본 계층으로 조정됩니다.
* [중형.][medium-pricing]: 적당한 크기의 배포를 암시하는 구성 요소를 나타냅니다. 여기서는 한 달 동안 시스템을 사용하는 시용자가 약 10만 명이라고 예상합니다. 보통의 표준 계층이 있는 단일 앱 서비스 인스턴스에서 필요한 트래픽이 처리됩니다. 또한 인지 및 검색 서비스의 중간 계층이 계산기에 추가됩니다.
* [대형][large-pricing]: 매월 테라바이트 단위의 데이터를 이동하는 수백만 명의 사용자가 주문하는 수준의 대규모 응용 프로그램을 나타냅니다. 이 고성능 사용 수준에서는 여러 지역에 배포되어 트래픽 관리자에서 제어되는 프리미엄 계층 웹앱이 필요합니다. 데이터는 저장소, 데이터베이스 및 CDN으로 구성되며, 이러한 구성 요소는 테라바이트 단위의 데이터로 구성됩니다.

## <a name="related-resources"></a>관련 리소스

* [다중 지역 웹 응용 프로그램에 대한 참조 아키텍처][multi-region-web-app]
* [eShopOnContainers 참조 예제][microservices-ecommerce]

<!-- links -->
[small-pricing]: https://azure.com/e/90fbb6a661a04888a57322985f9b34ac
[medium-pricing]: https://azure.com/e/38d5d387e3234537b6859660db1c9973
[large-pricing]: https://azure.com/e/f07f99b6c3134803a14c9b43fcba3e2f
[app-service-reference-architecture]: /azure/architecture/reference-architectures/app-service-web-app/
[architecture-diagram]: ./media/architecture-diagram-ecommerce-solution.png
[availability]: /azure/architecture/checklist/availability
[circuit-breaker]: /azure/architecture/patterns/circuit-breaker
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[docs-application-insights]: /azure/application-insights/app-insights-overview
[docs-b2c]: /azure/active-directory-b2c/active-directory-b2c-overview
[docs-cdn]: /azure/cdn/cdn-overview
[docs-container-instances]: /azure/container-instances/
[docs-kubernetes-service]: /azure/aks/
[docs-cosmosdb]: /azure/cosmos-db/
[docs-functions]: /azure/azure-functions/functions-overview
[docs-redis-cache]: /azure/redis-cache/cache-overview
[docs-search]: /azure/search/search-what-is-azure-search
[docs-service-fabric]: /azure/service-fabric/
[docs-sentiment-analysis]: /azure/cognitive-services/welcome
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-storage-blobs]: /azure/storage/blobs/storage-blobs-introduction
[docs-storage-queues]: /azure/storage/queues/storage-queues-introduction
[docs-traffic-manager]: /azure/traffic-manager/traffic-manager-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[microservices-ecommerce]: https://github.com/dotnet-architecture/eShopOnContainers
[multi-region-web-app]: /azure/architecture/reference-architectures/app-service-web-app/multi-region
[pci-dss-blueprint]: /azure/security/blueprints/payment-processing-blueprint
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[secure-development]: https://www.microsoft.com/en-us/SDL/process/design.aspx
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
[storage-geo-redudancy]: /azure/storage/common/storage-redundancy-grs
