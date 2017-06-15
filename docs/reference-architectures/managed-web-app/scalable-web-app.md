---
title: Scalable web application
description: Improving scalability in a web application running in Microsoft Azure.


author: MikeWasson



pnp.series.title: Azure App Service
pnp.series.prev: basic-web-app
pnp.series.next: multi-region-web-app

ms.service: guidance

ms.topic: article


ms.date: 11/23/2016
ms.author: pnp
cardTitle: Improve scalability
---
# 웹 응용 프로그램에서 확장성 개선
[!INCLUDE [header](../../_includes/header.md)]

이 참조 아키텍처는 Microsoft Azure에서 실행되는 웹 응용 프로그램의 확장성과 성능을 개선하기 위한 일련의 검증된 사례를 보여줍니다. 

![[0]][0]

## 아키텍처

이 참조 아키텍처는 [기본 웹 응용 프로그램][basic-web-app]에 소개된 아키텍처를 기반으로 하고 있습니다. 이 아키텍처에는 다음 구성 요소가 포함됩니다.

* **리소스 그룹**. [리소스 그룹][resource-group]은 Azure 리소스를 위한 논리적 컨테이너입니다.
* **[웹 앱][app-service-web-app]** 및 **[API 앱][app-service-api-app]**. 일반적으로 최신 응용 프로그램에는 웹사이트와 하나 이상의 RESTful Web API가 모두 포함됩니다. 웹 API는 AJAX를 통해 네이티브 클라이언트 응용 프로그램 또는 서버 쪽 응용 프로그램을 사용하여 브라우저 클라이언트에 의해 사용될 수 있습니다. 웹 API 설계 시 고려사항에 관해서는 [API 설계 가이드][api-guidance]를 참조하시기 바랍니다.    
* **WebJob**. [Azure WebJobs][webjobs]를 사용하여 배경에서 장시간 작업을 실행합니다. WebJobs는 일정에 따라, 지속적으로, 또는 큐에 메시지를 넣는 등의 트리거에 의해 실행할 수 있습니다. WebJob은 App Service 앱의 컨텍스트에서 배경 프로세스로 실행됩니다.
* **큐**. 이 아키텍처에서 응용 프로그램은 [Azure Queue 저장소][queue-storage] 큐에 메시지를 넣는 방식으로 배경 작업을 큐에 추가합니다. 이 메시지는 WebJob의 기능을 작동시킵니다. 또 다른 방식은 Service Bus 큐를 이용하는 것입니다. 두 방식을 비교하려면 [Azure Queues 및 Service Bus 큐 - 비교 및 대조][queues-compared]를 참조하시기 바랍니다.
* **캐시**. [Azure Redis Cache][azure-redis]에 준정적(semi-static) 데이터를 저장합니다.  
* **CDN**. 컨텐츠의 낮은 대기 시간과 신속한 전달을 위해 [Azure Content Delivery Network][azure-cdn] (CDN)를 사용하여 공개적으로 이용할 수 있는 콘텐츠를 캐시에 저장합니다.
* **데이터 저장**. 관계형 데이터를 위한 [Azure SQL Database][sql-db]를 사용합니다. 비관계형 데이터의 경우 Azure Table 저장소나 [DocumentDB][documentdb]와 같은 NoSQL 저장소를 고려해 보시기 바랍니다.
* **Azure Search**. [Azure Search][azure-search]를 사용하여 검색 제안, 퍼지 검색, 언어별 검색과 같은 검색 기능을 추가할 수 있습니다. Azure Search는 특히 주 데이터 저장소가 엄격한 일관성을 요구하는 경우 다른 데이터 저장소와 함께 사용됩니다. 이 방식에서는 신뢰할 수 있는 데이터를 다른 데이터 저장소에 저장하고 검색 인덱스를 Azure Search에 저장합니다. Azure Search를 사용하여 여러 데이터 저장소의 단일 검색 인덱스를 통합할 수도 있습니다.   
* **이메일/SMS**. 이메일이나 SMS 메시지 전송 기능을 응용 프로그램에 직접 구축하는 대신 SendGrid나 Twilio와 같은 타사 서비스를 이용할 수 있습니다.

## 권장사항

이 문서에서 설명하는 아키텍처는 귀하의 요구사항과 정확히 일치하지 않을 수 있습니다. 따라서 이 문서의 권장사항은 하나의 출발점으로 삼으시기 바랍니다.

### App Service 앱
웹 응용 프로그램과 웹 API는 별개의 App Service 앱으로 만들 것을 권장합니다. 이러한 설계를 통해 앱을 별개의 App Service 요금제로 실행함으로써 단독으로도 확장이 가능해집니다. 처음부터 높은 확장성이 필요하지 않다면 우선 앱을 동일한 요금제로 배포한 다음 필요할 때 별개의 요금제로 변경할 수 있습니다. 

> [!참고]
> Basic, Standard 및 Premium 요금제의 경우 앱별 요금이 아닌 해당 요금제의 VM 인스턴스를 기준으로 요금이 청구됩니다. [App Service 가격 책정][app-service-pricing]을 참조하세요.
> 
> 

App Service Mobile Apps의 *Easy Tables*나 *Easy APIs* 기능을 사용하려면 이러한 용도를 위한 별개의 App Service 앱을 생성하시기 바랍니다. 이 기능들의 사용 가능 여부는 특정 응용 프로그램 프레임워크에 의해 결정됩니다.

### WebJobs
리소스가 많이 요구되는 WebJobs는 별개의 App Service 요금제의 비어 있는 App Service 앱에 배포하는 것을 고려해 보시기 바랍니다. 이를 통해 WebJob 전용 인스턴스를 얻을 수 있습니다. [배경 작업 가이드][webjobs-guidance]를 참조하시기 바랍니다.  

### 캐시
[Azure Redis Cache][azure-redis]를 사용하여 일부 데이터를 캐시에 저장하여 성능과 확장성을 개선할 수 있습니다. 다음과 같은 대상에 대해 Redis Cache를 사용하는 것을 고려해 보세요.

* 준정적(semi-static) 트랜잭션 데이터.
* 세션 상태.
* HTML 출력. Redis Cache는 복잡한 HTML 출력을 렌더링하는 응용 프로그램에 유용합니다.

캐싱 전략 설계에 관한 자세한 내용은 [캐싱 가이드][caching-guidance]를 참조하세요.

### CDN
[Azure CDN][azure-cdn]을 사용하여 정적 컨텐츠를 캐시에 저장합니다. CDN의 주요 이점은 콘텐츠가 사용자와 지리적으로 가까운 곳에 위치한 에지 서버에서 캐시에 저장되므로 사용자 대기 시간을 줄일 수 있다는 점입니다. CDN은 응용 프로그램이 트래픽을 처리하지 않기 때문에 응용 프로그램의 부하를 감소시켜주기도 합니다. 

앱이 대부분 정적 페이지로 구성되어 있다면 [전체 앱 캐싱을 위한 CDN][cdn-app-service]의 사용을 고려해 보시기 바랍니다. 사용하지 않을 경우에는 이미지, CSS, HTML 파일을 [Azure 스토리지에 저장하고 CDN을 사용하여 해당 파일들을 캐시에 저장합니다][cdn-storage-account].

> [!참고]
> Azure CDN은 인증이 필요한 컨텐츠에는 사용할 수 없습니다.
> 
> 

자세한 내용은 [컨텐츠 제공 네트워크(CDN) 가이드][cdn-guidance]를 참조하시기 바랍니다.

### 저장소
최신 응용 프로그램들은 대용량 데이터를 처리하는 경우가 많습니다. 클라우드 확장을 위해서는 올바른 저장소 유형을 선택하는 것이 중요합니다. 기본적인 권장사항은 다음과 같습니다.

| 저장할 대상 | 예제 | 권장 저장소 |
| --- | --- | --- |
| 파일 |이미지, 문서, PDF |Azure Blob 저장소 |
| 키/값 결합 |사용자 ID로 검색되는 사용자 프로필 데이터 |Azure Table 저장소 |
| 추가적인 처리를 발동시키기 위한 단문 메시지 |명령 요청 |Azure Queue 저장소, Service Bus 큐 또는 Service Bus 토픽 |
| 기본적인 조회를 요구하는 유연한 스키마를 가진 비관계형 데이터 |제품 카탈로그 |Azure DocumentDB, MongoDB, Apache CouchDB와 같은 문서 DB |
| 더 높은 수준의 쿼리 지원, 엄격한 스키마, 높은 일관성을 요구하는 관계형 데이터 |제품 인벤토리 |Azure SQL Database |

## 확장성 고려사항

Azure App Service의 주요 이점은 응용 프로그램을 부하에 따라 확장할 수 있다는 점입니다. 응용 프로그램 확장 계획 수립 시 고려해야할 사항은 다음과 같습니다.

### App Service 앱
솔루션에 여러 App Service 앱이 포함되는 경우, 각각 별개의 App Service 요금제로 배포하는 것을 고려해 보시기 바랍니다. 이러한 방식을 택할 경우 각각의 앱이 별개의 인스턴스에서 실행되므로 독립적으로 확장을 수행할 수 있습니다. 

마찬가지로 WebJob을 요금제에 넣어 배경 작업이 HTTP 요청을 처리하는 것과 동일한 인스턴스에서 실행되지 않도록 할 수도 있습니다.   

### SQL 데이터베이스
*분할*을 통해 SQL DB의 확장성을 높일 수 있습니다. *분할*이란 DB의 수평 파티셔닝을 의미합니다. 분할을 통해 [탄력적 DB 도구][sql-elastic]를 사용하여 DB를 수평적으로 확장할 수 있습니다. 분할의 잠재적 이점은 다음과 같습니다.

- 향상된 트랜잭션 처리율.
- 쿼리는 데이터 하위집합에서 더 빠르게 실행됩니다. 

### Azure Search
Azure Search는 주 데이터 저장소로부터 복잡한 데이터 검색을 수행하는 오버헤드를 제거하고 부하를 처리하도록 확장될 수 있습니다. [Azure Search에서 쿼리 및 인덱싱 워크로드를 위한 리소스 수준 확장][azure-search-scaling]을 참조하시기 바랍니다.

## 보안 고려사항
이 섹션에서는 이 문서에 설명된 Azure 서비스에 대한 보안 고려사항을 다룹니다. 이 섹션에 포함되지 않은 추가적인 고려사항은 [Azure App 서비스의 앱 보안][app-service-security]을 참조하시기 바랍니다.

### Cross-Origin 리소스 공유(CORS)
웹사이트와 웹 API를 별개의 앱으로 생성한 경우 CORS를 사용하도록 설정하지 않으면 웹사이트는 API에 클라이언트 쪽 AJAX 요청을 할 수 없습니다. 

> [!참고]
> 브라우저 보안은 웹 페이지가 다른 도메인에 AJAX 요청을 하는 것을 방지합니다. 이러한 제한은 Same-Origin 정책이라고 하며 악성 사이트가 다른 사이트로부터 민감한 데이터를 읽지 못하도록 합니다. CORS는 서버가 Same-Origin 정책을 완화하여 일부 Cross-Origin 요청을 허용하고 다른 요청은 거절하도록 지원하는 W3C 표준입니다.  
> 
> 

App Services는 CORS를 기본으로 지원하므로 응용 프로그램 코드를 쓸 필요가 없습니다. [CORS를 사용하여 JavaScript로부터 API 앱 사용][cors]을 참조하시기 바랍니다. 웹사이트를 해당 API에 대해 허용된 오리진 목록에 추가합니다. 

### SQL Database 암호화
[투명한 데이터 암호화(TDE)][sql-encryption]를 사용하여 DB의 데이터를 안전하게 암호화합니다. 이 기능은 응용 프로그램 변경 없이 (백업 및 트랜잭션 로그 파일을 포함한) 전체 DB를 실시간으로 암호화/해독합니다.   암호화는 대기 시간을 증가시키므로 보호해야 할 데이터를 자체 DB로 분리하고 해당 DB에 대해서만 암호화를 사용하도록 설정하는 것이 좋습니다.
  

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
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
[web-app-multi-region]: ./multi-region-web-app.md
[webjobs-guidance]: ../../best-practices/background-jobs.md
[webjobs]: /azure/app-service/app-service-webjobs-readme
[0]: ../_images/blueprints/paas-web-scalability.png "Web application in Azure with improved scalability"
