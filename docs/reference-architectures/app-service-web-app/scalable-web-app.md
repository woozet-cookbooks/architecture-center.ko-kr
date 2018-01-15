---
title: "확장 가능한 웹 응용 프로그램"
description: "Microsoft Azure에서 실행되는 웹 응용 프로그램의 확장성을 향상합니다."
author: MikeWasson
pnp.series.title: Azure App Service
pnp.series.prev: basic-web-app
pnp.series.next: multi-region-web-app
ms.date: 11/23/2016
cardTitle: Improve scalability
ms.openlocfilehash: 1fdaf6e3695cb814fa4c275a4a273f9fa9a7b71b
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/08/2018
---
# <a name="improve-scalability-in-a-web-application"></a>웹 응용 프로그램의 확장성 향상

이 참조 아키텍처는 Azure App Service 웹 응용 프로그램의 확장성과 성능 향상을 위한 검증된 사례를 보여 줍니다.

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>건축  

이 아키텍처는 [기본 웹 응용 프로그램][basic-web-app]에 표시된 아키텍처를 기반으로 합니다. 다음 구성 요소가 포함되어 있습니다.

* **리소스 그룹**. [리소스 그룹][resource-group]은 Azure 리소스에 대한 논리적 컨테이너입니다.
* **[웹앱][app-service-web-app]** 및 **[API 앱][app-service-api-app]**. 일반적인 최신 응용 프로그램에는 웹 사이트와 하나 이상의 RESTful 웹 API가 모두 포함되어 있을 수 있습니다. AJAX를 통한 브라우저 클라이언트, 기본 클라이언트 응용 프로그램 또는 서버 쪽 응용 프로그램에서 웹 API를 사용할 수 있습니다. 웹 API 디자인에 대한 고려 사항은 [API 디자인 지침][api-guidance]을 참조하세요.    
* **WebJob**. 백그라운드에서 장기 실행 작업을 실행하려면 [Azure WebJobs][webjobs]를 사용합니다. WebJob은 예약에 따라, 연속적으로 또는 큐에 메시지를 넣는 등 트리거에 대한 응답으로 실행될 수 있습니다. WebJob은 App Service 앱의 컨텍스트에서 백그라운드 프로세스로 실행됩니다.
* **큐**. 여기에 표시된 아키텍처에서는 응용 프로그램이 [Azure Queue Storage][queue-storage] 큐에 메시지를 넣어 백그라운드 작업을 큐에 넣습니다. 메시지가 WebJob의 함수를 트리거합니다. 또는 Service Bus 큐를 사용할 수 있습니다. 비교하려면 [Azure 큐 및 Service Bus 큐 - 비교 및 대조][queues-compared]를 참조하세요.
* **캐시**. [Azure Redis Cache][azure-redis]의 반정적 데이터를 저장합니다.  
* **CDN**. [Azure CDN(Content Delivery Network)][azure-cdn]을 사용하여 지연 시간을 단축하고 더 신속한 콘텐츠 배달을 위해 공개적으로 사용 가능한 콘텐츠를 캐시합니다.
* **데이터 저장소**. 관계형 데이터의 경우 [Azure SQL Database][sql-db]를 사용합니다. 비관계형 데이터의 경우 [Cosmos DB][documentdb] 같은 NoSQL 저장소를 고려합니다.
* **Azure Search**. [Azure Search][azure-search]를 사용하여 검색 제안, 유사 항목 검색 및 언어별 검색과 같은 검색 기능을 추가합니다. Azure Search는 일반적으로 다른 데이터 저장소와 함께 사용되는데, 특히 기본 데이터 저장소에 엄격한 일관성이 필요한 경우 그렇습니다. 이러한 접근 방식에서는 신뢰할 수 있는 데이터를 다른 데이터 저장소에 저장하고 검색 인덱스를 Azure Search에 저장합니다. 또한 Azure Search는 여러 데이터 저장소의 단일 검색 인덱스를 통합하는 데 사용할 수 있습니다.  
* **메일/SMS**. SendGrid 또는 Twilio와 같은 타사 서비스를 사용하여 응용 프로그램에 직접 이 기능을 빌드하는 대신 메일이나 SMS 메시지를 전송합니다.
* **Azure DNS**. [Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다. Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다.

## <a name="recommendations"></a>권장 사항

개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다. 이 섹션의 권장 사항을 시작점으로 사용합니다.

### <a name="app-service-apps"></a>App Service 앱
웹 응용 프로그램과 웹 API를 별도의 App Service 앱으로 만드는 것이 좋습니다. 이렇게 디자인하면 이들을 별도의 App Service 계획에서 실행할 수 있으므로 독립적으로 확장할 수 있습니다. 처음에 이러한 수준의 확장성이 필요하지 않은 경우 앱을 동일한 계획에 배포하고 필요한 경우 나중에 별도의 계획으로 이동할 수 있습니다.

> [!NOTE]
> Basic, Standard 및 Premium 계획은 앱 단위가 아니라 계획의 VM 인스턴스에 대해 청구됩니다. [App Service 가격 책정][app-service-pricing]을 참조하세요.
> 
> 

App Service Mobile Apps의 *간편한 테이블* 또는 *간편한 API* 기능을 사용하려는 경우 이를 위해 별도의 App Service 앱을 만듭니다.  이러한 기능은 특정 응용 프로그램 프레임워크에 의존하여 사용할 수 있습니다.

### <a name="webjobs"></a>웹 작업
별도의 App Service 계획 내에 있는 빈 App Service 앱에 리소스를 많이 사용하는 WebJob을 배포하는 것을 고려합니다. 이렇게 하면 WebJob에 전용 인스턴스가 제공됩니다. [백그라운드 작업 지침][webjobs-guidance]을 참조하세요.  

### <a name="cache"></a>캐시
[Azure Redis Cache][azure-redis]를 사용하여 일부 데이터를 캐시하면 성능과 확장성을 향상할 수 있습니다. 다음에 대해 Redis Cache 사용을 고려합니다.

* 반정적 트랜잭션 데이터
* 세션 상태
* HTML 출력 이는 복잡한 HTML 출력을 렌더링하는 응용 프로그램에 유용할 수 있습니다.

캐싱 전략 디자인에 대한 자세한 지침은 [캐싱 지침][caching-guidance]을 참조하세요.

### <a name="cdn"></a>CDN
[Azure CDN][azure-cdn]을 사용하여 정적 콘텐츠를 캐시합니다. CDN의 주요 이점은 사용자의 대기 시간이 줄어드는 것인데 그 이유는 콘텐츠가 사용자와 지리적으로 가까운 에지 서버에 캐시되기 때문입니다. 또한 응용 프로그램에 의해 트래픽이 처리되지 않기 때문에 CDN은 응용 프로그램의 부하를 줄일 수 있습니다.

앱이 대부분 정적 페이지로 구성된 경우 [전체 앱을 캐시하는 데 CDN][cdn-app-service] 사용을 고려합니다. 그렇지 않은 경우에는 이미지, CSS, HTML 파일 등의 정적 콘텐츠를 [Azure Storage에 넣고 CDN을 사용하여 이러한 파일을 캐시합니다][cdn-storage-account].

> [!NOTE]
> Azure CDN은 인증이 필요한 콘텐츠를 제공할 수 없습니다.
> 
> 

자세한 지침은 [CDN(콘텐츠 배달 네트워크) 지침][cdn-guidance]을 참조하세요.

### <a name="storage"></a>Storage
최신 응용 프로그램은 종종 많은 양의 데이터를 처리합니다. 클라우드에 대한 크기를 조정하려면 올바른 저장소 유형을 선택해야 합니다. 다음은 몇 가지 기본 권장 사항입니다. 

| 저장 형식 | 예 | 권장 저장소 |
| --- | --- | --- |
| 파일 |이미지, 문서, PDF |Azure Blob Storage |
| 키/값 쌍 |사용자 ID로 조회된 사용자 프로필 데이터 |Azure 테이블 저장소 |
| 추가 처리를 트리거할 짧은 메시지 |주문 요청 |Azure Queue Storage, Service Bus 큐 또는 Service Bus 항목 |
| 기본 쿼리를 요구하는 유연한 스키마를 사용하는 비관계형 데이터 |제품 카탈로그 |Azure Cosmos DB, MongoDB 또는 Apache CouchDB와 같은 문서 데이터베이스 |
| 보다 풍부한 쿼리 지원, 엄격한 스키마 및/또는 강력한 일관성이 필요한 관계형 데이터 |제품 인벤토리 |Azure SQL Database |

## <a name="scalability-considerations"></a>확장성 고려 사항

Azure App Service의 주요 이점은 부하에 따라 응용 프로그램을 확장할 수 있다는 점입니다. 다음은 응용 프로그램 확장을 계획할 때 염두할 몇 가지 고려 사항입니다.

### <a name="app-service-app"></a>App Service 앱
솔루션에 여러 App Service 앱이 포함되어 있는 경우 App Service 계획이 분리되도록 배포하는 것을 고려합니다. 이러한 방식을 사용하면 개별 인스턴스에서 실행되므로 개별적으로 확장할 수 있습니다. 

마찬가지로, 백그라운드 작업이 HTTP 요청을 처리하는 동일한 인스턴스에서 실행되지 않도록 WebJob을 고유한 계획에 넣는 것을 고려합니다.  

### <a name="sql-database"></a>SQL Database
데이터베이스를 *분할*하여 SQL데이터베이스의 확장성을 높입니다. 분할은 데이터베이스를 가로로 분할합니다. 분할을 사용하면 [Elastic Database 도구][sql-elastic]로 데이터베이스를 가로로 확장할 수 있습니다. 분할의 잠재적 이점은 다음과 같습니다.

- 트랜잭션 처리량이 늘어납니다.
- 쿼리가 데이터의 하위 집합에서 더 빨리 실행될 수 있습니다.

### <a name="azure-search"></a>Azure Search
Azure Search는 주요 데이터 저장소에서 복잡한 데이터 검색을 수행하는 데 따른 오버헤드를 제거하므로 부하를 처리하도록 확장할 수 있습니다. [Azure Search의 쿼리 및 인덱싱 작업을 위한 리소스 수준 확장][azure-search-scaling]을 참조하세요.

## <a name="security-considerations"></a>보안 고려 사항
이 섹션에는 이 문서에 설명된 Azure 서비스와 관련된 보안 고려 사항이 나와 있습니다. 보안 모범 사례가 완전히 다 나와 있는 것은 아닙니다. 몇 가지 추가 보안 고려 사항은 [Azure App Service에서 앱 보안][app-service-security]을 참조하세요.

### <a name="cross-origin-resource-sharing-cors"></a>CORS(크로스-원본 자원 공유)
웹 사이트와 웹 API를 별도의 앱으로 만드는 경우 CORS를 사용하도록 설정하지 않으면 클라이언트 쪽 AJAX에서 API를 호출할 수 없습니다.

> [!NOTE]
> 브라우저 보안은 웹 페이지에서 다른 도메인으로 AJAX 요청을 수행하지 못하도록 방지합니다. 이렇게 제한하는 것을 동일 원본 정책이라고 하며, 악성 사이트에서 다른 사이트의 중요한 데이터를 읽을 수 없도록 합니다. CORS는 서버에서 동일 원본 정책을 완화하고 크로스-원본 요청을 일부는 허용하고 일부는 거부할 수 있는 W3C표준입니다.
> 
> 

App Services에서는 응용 프로그램 코드를 작성할 필요 없이 기본적으로 CORS를 지원합니다. [CORS를 사용하여 JavaScript에서 API 앱 사용][cors]을 참조하세요. API에 대해 허용되는 원본 목록에 웹 사이트를 추가합니다.

### <a name="sql-database-encryption"></a>SQL Database 암호화
데이터베이스에서 미사용 데이터를 암호화해야 하는 경우 [투명한 데이터 암호화][sql-encryption]를 사용합니다. 이 기능은 전체 데이터베이스(백업 및 트랜잭션 로그 파일 포함)의 실시간 암호화 및 암호 해독을 수행하며 응용 프로그램을 변경할 필요가 없습니다. 암호화를 수행하면 대기 시간이 늘어나므로, 고유한 데이터베이스에 보호해야 하는 데이터를 분리하고 해당 데이터베이스에 대해서만 암호화를 사용하도록 설정하는 것이 좋습니다.  
  

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-dns]: /azure/dns/dns-overview
[azure-redis]: https://azure.microsoft.com/services/cache/
[azure-search]: https://azure.microsoft.com/documentation/services/search/
[azure-search-scaling]: /azure/search/search-capacity-planning
[background-jobs]: ../../best-practices/background-jobs.md
[basic-web-app]: basic-web-app.md
[basic-web-app-scalability]: basic-web-app.md#scalability-considerations
[caching-guidance]: ../../best-practices/caching.md
[cdn-app-service]: /azure/app-service-web/cdn-websites-with-cdn
[cdn-storage-account]: /azure/cdn/cdn-create-a-storage-account-with-cdn
[cdn-guidance]: ../../best-practices/cdn.md
[cors]: /azure/app-service-api/app-service-api-cors-consume-javascript
[documentdb]: https://azure.microsoft.com/documentation/services/documentdb/
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
[webjobs-guidance]: ../../best-practices/background-jobs.md
[webjobs]: /azure/app-service/app-service-webjobs-readme
[0]: ./images/scalable-web-app.png "확장성이 향상된 Azure의 웹 응용 프로그램"
