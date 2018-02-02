---
title: "실패 모드 분석"
description: "Azure에 기반한 클라우드 솔루션에 대한 장애 모드 분석을 수행하기 위한 지침입니다."
author: MikeWasson
ms.date: 03/24/2017
ms.custom: resiliency
pnp.series.title: Design for Resiliency
ms.openlocfilehash: aca2088cb007728c5717a968969000c0a19bcd07
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/30/2018
---
# <a name="failure-mode-analysis"></a>실패 모드 분석
[!INCLUDE [header](../_includes/header.md)]

FMA(장애 모드 분석)는 가능한 장애 지점을 식별하여 시스템에 복원력을 구축하는 프로세스입니다. FMA는 아키텍처 및 디자인 단계의 일부여야 하므로 처음부터 장애 복구를 시스템에 구축할 수 있습니다.

FMA를 수행하는 일반적인 프로세스는 다음과 같습니다.

1. 시스템의 모든 구성 요소를 식별합니다. ID 공급자, 타사 서비스 등과 같은 외부 종속성을 포함합니다.   
2. 각 구성 요소마다 발생할 수 있는 잠재적 장애를 식별합니다. 단일 구성 요소에 둘 이상의 장애 모드가 있을 수 있습니다. 예를 들어 영향력 및 가능한 완화 방식이 서로 다르기 때문에 읽기 오류와 쓰기 오류를 별도로 고려해야 합니다.
3. 전체적인 위험도에 따라 각 장애 모드를 평가합니다. 다음 항목을 고려합니다.  

   * 장애의 가능성은 무엇인가요? 비교적 일반적인가요? 매우 드물게 발생하나요? 우선 순위를 지정하기 위한 것이므로 정확한 숫자는 필요하지 않습니다.
   * 가용성, 데이터 손실, 금전적 비용 및 비즈니스 중단과 관련하여 응용 프로그램에 미치는 영향은 무엇인가요?
4. 각 장애 모드에 대해 응용 프로그램에서 응답하고 복구하는 방법을 결정합니다. 비용 및 응용 프로그램 복잡성의 장단점을 고려합니다.   

FMA 프로세스에 대한 시작점으로, 이 문서에는 잠재적인 장애 모드의 카탈로그 및 해당 완화 방법이 포함되어 있습니다. 카탈로그는 기술 또는 Azure 서비스와 응용 프로그램 수준 디자인을 위한 일반 범주로 구성됩니다. 카탈로그는 완벽하지 않지만 Azure 핵심 서비스의 많은 부분을 다루고 있습니다.

## <a name="app-service"></a>App Service
### <a name="app-service-app-shuts-down"></a>App Service 앱이 종료됩니다.
**검색**. 가능한 원인:

* 예상된 종료

  * 운영자가 응용 프로그램(예: Azure Portal 사용)을 종료합니다.
  * 앱이 유휴 상태이므로 언로드되었습니다. (`Always On` 설정이 사용 안 함인 경우에만)
* 예기치 않은 종료

  * 앱이 충돌합니다.
  * App Service VM 인스턴스를 사용할 수 없게 됩니다.

Application_End 로깅은 응용 프로그램 도메인 종료(소프트 프로세스 크래시)를 catch하며, 응용 프로그램 도메인 종료를 catch할 수 있는 유일한 방법입니다.

**복구**

* 종료가 예상되면 응용 프로그램의 종료 이벤트를 사용하여 정상적으로 종료합니다. 예를 들어 ASP.NET에서는 `Application_End` 메서드를 사용합니다.
* 유휴 상태에 있는 응용 프로그램이 언로드되었으면 다음 요청 시 자동으로 다시 시작됩니다. 그러나 "콜드 부팅" 비용이 발생합니다.
* 유휴 상태에 있는 응용 프로그램이 언로드되지 않도록 방지하려면 웹앱에서 `Always On` 설정을 활성화합니다. [Azure App Service에서 웹앱 구성][app-service-configure]을 참조하세요.
* 운영자가 앱을 종료하지 못하도록 하려면 `ReadOnly` 수준의 리소스 잠금을 설정합니다. [Azure Resource Manager를 사용하여 리소스 잠그기][rm-locks]를 참조하세요.
* 앱의 작동이 중단되거나 App Service VM을 사용할 수 없게 되면 App Service에서 앱을 자동으로 다시 시작합니다.

**진단**. 응용 프로그램 및 웹 서버 로그. [Azure App Service에서 웹앱에 대한 진단 로깅 설정][app-service-logging]을 참조하세요.

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a>특정 사용자가 반복적으로 잘못 요청하거나 시스템을 오버로드합니다.
**검색**. 사용자를 인증하고 응용 프로그램 로그에 사용자 ID를 포함합니다.

**복구**

* [Azure API Management][api-management]를 사용하여 사용자의 요청을 제한합니다. [Azure API Management로 고급 요청 제한][api-management-throttling]을 참조하세요.
* 사용자를 차단합니다.

**진단**. 모든 인증 요청을 기록합니다.

### <a name="a-bad-update-was-deployed"></a>잘못된 업데이트가 배포되었습니다.
**검색**. Azure Portal을 통해 응용 프로그램 상태를 모니터링하거나([Azure 웹앱 성능 모니터링][app-insights-web-apps] 참조) [상태 엔드포인트 모니터링 패턴][health-endpoint-monitoring-pattern]을 구현합니다.

**복구**. 여러 [배포 슬롯][app-service-slots]을 사용하고 마지막으로 성공한 배포로 롤백합니다. 자세한 내용은 [기본 웹 응용 프로그램][ra-web-apps-basic]을 참조하세요.

## <a name="azure-active-directory"></a>Azure Active Directory
### <a name="openid-connect-oidc-authentication-fails"></a>OIDC(OpenID Connect) 인증이 실패합니다.
**검색**. 가능한 장애 모드는 다음과 같습니다.

1. Azure AD를 사용할 수 없거나 네트워크 문제로 인해 연결할 수 없습니다. 인증 엔드포인트로의 리디렉션이 실패하고 OIDC 미들웨어에서 예외를 throw합니다.
2. Azure AD 테넌트가 없습니다. 인증 엔드포인트로 리디렉션하면 HTTP 오류 코드가 반환되고 OIDC 미들웨어에서 예외를 throw합니다.
3. 사용자를 인증할 수 없습니다. 검색 전략이 필요하지 않으며, Azure AD에서 로그인 실패를 처리합니다.

**복구**

1. 미들웨어에서 처리되지 않은 예외를 catch합니다.
2. `AuthenticationFailed` 이벤트를 처리합니다.
3. 사용자를 오류 페이지로 리디렉션합니다.
4. 사용자가 다시 시도합니다.

## <a name="azure-search"></a>Azure Search
### <a name="writing-data-to-azure-search-fails"></a>Azure Search에 데이터를 쓰는 작업이 실패합니다.
**검색**. `Microsoft.Rest.Azure.CloudException` 오류를 catch합니다.

**복구**

일시적 장애 후에 [Search .NET SDK][search-sdk]에서 자동으로 다시 시도합니다. 클라이언트 SDK에서 throw된 모든 예외는 일시적이지 않은 오류로 처리되어야 합니다.

기본 다시 시도 정책은 지수 백오프를 사용합니다. 다른 다시 시도 정책을 사용하려면 `SearchIndexClient` 또는 `SearchServiceClient` 클래스에서 `SetRetryPolicy`를 호출합니다. 자세한 내용은 [자동 다시 시도][auto-rest-client-retry]를 참조하세요.

**진단**. [Search 트래픽 분석][search-analytics]을 사용합니다.

### <a name="reading-data-from-azure-search-fails"></a>Azure Search에서 데이터를 읽는 작업이 실패합니다.
**검색**. `Microsoft.Rest.Azure.CloudException` 오류를 catch합니다.

**복구**

일시적 장애 후에 [Search .NET SDK][search-sdk]에서 자동으로 다시 시도합니다. 클라이언트 SDK에서 throw된 모든 예외는 일시적이지 않은 오류로 처리되어야 합니다.

기본 다시 시도 정책은 지수 백오프를 사용합니다. 다른 다시 시도 정책을 사용하려면 `SearchIndexClient` 또는 `SearchServiceClient` 클래스에서 `SetRetryPolicy`를 호출합니다. 자세한 내용은 [자동 다시 시도][auto-rest-client-retry]를 참조하세요.

**진단**. [Search 트래픽 분석][search-analytics]을 사용합니다.

## <a name="cassandra"></a>Cassandra
### <a name="reading-or-writing-to-a-node-fails"></a>노드에서 읽거나 쓰는 작업이 실패합니다.
**검색**. 예외를 catch합니다. .NET 클라이언트의 경우 일반적으로 `System.Web.HttpException`입니다. 다른 클라이언트에는 다른 예외 형식이 있을 수 있습니다.  자세한 내용은 [올바른 Cassandra 오류 처리](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right)를 참조하세요.

**복구**

* 각 [Cassandra 클라이언트](https://wiki.apache.org/cassandra/ClientOptions)에는 자체의 다시 시도 정책 및 기능이 있습니다. 자세한 내용은 [올바른 Cassandra 오류 처리][cassandra-error-handling]를 참조하세요.
* 데이터 노드가 장애 도메인에 분산되어 있는 랙 인식 배포를 사용합니다.
* 로컬 쿼럼 일관성을 사용하여 여러 지역에 배포합니다. 일시적이지 않은 장애가 발생하면 다른 지역으로 장애 조치합니다.

**진단**. 응용 프로그램 로그

## <a name="cloud-service"></a>클라우드 서비스
### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a>웹 또는 작업자 역할이 예기치 않게 종료됩니다.
**검색**. [RoleEnvironment.Stopping][RoleEnvironment.Stopping] 이벤트가 발생합니다.

**복구**. [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] 메서드를 재정의하여 정상적으로 정리합니다. 자세한 내용은 [Azure OnStop 이벤트를 처리하는 올바른 방법][onstop-events](블로그)을 참조하세요.

## <a name="cosmos-db"></a>Cosmos DB 
### <a name="reading-data-fails"></a>데이터 읽기가 실패합니다.
**검색**. `System.Net.Http.HttpRequestException` 또는 `Microsoft.Azure.Documents.DocumentClientException`을 catch합니다.

**복구**

* SDK에서 실패한 시도를 자동으로 다시 시도합니다. 다시 시도 횟수 및 최대 대기 시간을 설정하려면 `ConnectionPolicy.RetryOptions`를 구성합니다. 클라이언트에서 발생시키는 예외는 재시도 정책 시도 횟수를 초과하거나 일시적인 오류가 아닙니다.
* Cosmos DB에서 클라이언트를 제한하는 경우 HTTP 429 오류를 반환합니다. `DocumentClientException`에서 상태 코드를 확인합니다. 429 오류가 지속적으로 발생하면 컬렉션의 처리량 값을 늘리는 것이 좋습니다.
    * MongoDB API를 사용하는 경우 제한할 때 서비스에서 16500 오류 코드를 반환합니다.
* 둘 이상의 지역에서 Cosmos DB 데이터베이스를 복제합니다. 모든 복제본은 읽을 수 있습니다. 클라이언트 SDK를 사용하여 `PreferredLocations` 매개 변수를 지정합니다. 이는 순서가 지정된 Azure 지역 목록입니다. 모든 읽기는 목록에서 사용 가능한 첫 번째 지역으로 보내집니다. 요청이 실패하면 클라이언트에서 목록의 다른 지역을 순서대로 시도합니다. 자세한 내용은 [SQL API를 사용하여 Azure Cosmos DB 전역 배포를 설정하는 방법][cosmosdb-multi-region]을 참조하세요.

**진단**. 클라이언트 쪽에서 모든 오류를 기록합니다.

### <a name="writing-data-fails"></a>데이터 쓰기가 실패합니다.
**검색**. `System.Net.Http.HttpRequestException` 또는 `Microsoft.Azure.Documents.DocumentClientException`을 catch합니다.

**복구**

* SDK에서 실패한 시도를 자동으로 다시 시도합니다. 다시 시도 횟수 및 최대 대기 시간을 설정하려면 `ConnectionPolicy.RetryOptions`를 구성합니다. 클라이언트에서 발생시키는 예외는 재시도 정책 시도 횟수를 초과하거나 일시적인 오류가 아닙니다.
* Cosmos DB에서 클라이언트를 제한하는 경우 HTTP 429 오류를 반환합니다. `DocumentClientException`에서 상태 코드를 확인합니다. 429 오류가 지속적으로 발생하면 컬렉션의 처리량 값을 늘리는 것이 좋습니다.
* 둘 이상의 지역에서 Cosmos DB 데이터베이스를 복제합니다. 주 지역이 실패하면 다른 지역이 쓰기 지역으로 승격됩니다. 또한 장애 조치를 수동으로 트리거할 수도 있습니다. SDK에서 자동 검색 및 라우팅을 수행하므로 장애 조치 후에 응용 프로그램 코드가 계속 작동합니다. 장애 조치 기간(일반적으로 몇 분) 동안 SDK에서 새 쓰기 지역을 찾으므로 쓰기 작업의 대기 시간이 길어집니다.
  자세한 내용은 [SQL API를 사용하여 Azure Cosmos DB 전역 배포를 설정하는 방법][cosmosdb-multi-region]을 참조하세요.
* 대체(fallback) 방식으로, 백업 큐에 문서를 유지하고 나중에 큐를 처리합니다.

**진단**. 클라이언트 쪽에서 모든 오류를 기록합니다.

## <a name="elasticsearch"></a>Elasticsearch
### <a name="reading-data-from-elasticsearch-fails"></a>Elasticsearch에서 데이터를 읽는 작업이 실패합니다.
**검색**. 사용 중인 특정 [Elasticsearch 클라이언트][elasticsearch-client]에 대한 적절한 예외를 catch합니다.

**복구**

* 다시 시도 메커니즘을 사용합니다. 각 클라이언트에는 자체의 다시 시도 정책이 있습니다.
* 여러 Elasticsearch 노드를 배포하고 고가용성을 위해 복제를 사용합니다.

자세한 내용은 [Azure에서 Elasticsearch 실행][elasticsearch-azure]을 참조하세요.

**진단**. Elasticsearch에 대한 모니터링 도구를 사용하거나 클라이언트 쪽에서 페이로드를 사용하여 모든 오류를 기록할 수 있습니다. [Azure에서 Elasticsearch 실행][elasticsearch-azure]의 '모니터링' 섹션을 참조하세요.

### <a name="writing-data-to-elasticsearch-fails"></a>Elasticsearch에 데이터를 쓰는 작업이 실패합니다.
**검색**. 사용 중인 특정 [Elasticsearch 클라이언트][elasticsearch-client]에 대한 적절한 예외를 catch합니다.  

**복구**

* 다시 시도 메커니즘을 사용합니다. 각 클라이언트에는 자체의 다시 시도 정책이 있습니다.
* 응용 프로그램에서 축소된 일관성 수준을 허용할 수 있으면 `quorum`의 `write_consistency` 설정으로 쓰기 작업을 수행하는 것이 좋습니다.

자세한 내용은 [Azure에서 Elasticsearch 실행][elasticsearch-azure]을 참조하세요.

**진단**. Elasticsearch에 대한 모니터링 도구를 사용하거나 클라이언트 쪽에서 페이로드를 사용하여 모든 오류를 기록할 수 있습니다. [Azure에서 Elasticsearch 실행][elasticsearch-azure]의 '모니터링' 섹션을 참조하세요.

## <a name="queue-storage"></a>큐 저장소
### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a>Azure Queue 저장소에 메시지를 쓰는 작업이 일관되게 실패합니다.
**검색**. *N*회의 다시 시도를 수행한 후에도 쓰기 작업이 계속 실패합니다.

**복구**

* 로컬 캐시에 데이터를 저장하고, 나중에 서비스를 사용할 수 있게 되면 저장소에 쓰기를 전달합니다.
* 보조 큐를 만들고, 기본 큐를 사용할 수 없는 경우 이 보조 큐에 씁니다.

**진단**. [저장소 메트릭][storage-metrics]을 사용합니다.

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>응용 프로그램에서 큐의 특정 메시지를 처리할 수 없습니다.
**검색**. 응용 프로그램 특정입니다. 예를 들어 메시지에 유효하지 않은 데이터가 있거나 비즈니스 논리가 어떤 이유로 실패합니다.

**복구**

메시지를 별도의 큐로 이동합니다. 별도의 프로세스를 실행하여 해당 큐의 메시지를 검사합니다.

이 목적을 위해 [배달 못 한 편지 큐][sb-dead-letter-queue] 기능을 제공하는 Azure Service Bus 메시지 큐를 사용하는 것이 좋습니다.

> [!NOTE]
> WebJobs를 통해 저장소 큐를 사용하는 경우 WebJobs SDK에서 기본적으로 포이즌 메시지 처리를 제공합니다. [WebJobs SDK를 통해 Azure 큐 저장소를 사용하는 방법][sb-poison-message]을 참조하세요.

**진단**. 응용 프로그램 로깅을 사용합니다.

## <a name="redis-cache"></a>Redis Cache
### <a name="reading-from-the-cache-fails"></a>캐시에서 읽는 작업이 실패합니다.
**검색**. `StackExchange.Redis.RedisConnectionException`을 catch합니다.

**복구**

1. 일시적 장애 시 다시 시도합니다. Azure Redis Cache는 기본 제공 다시 시도를 지원합니다. [Redis 캐시 다시 시도 지침][redis-retry]을 참조하세요.
2. 일시적이지 않은 장애를 캐시 누락으로 처리하고 원래 데이터 원본으로 대체합니다.

**진단**. [Redis Cache 진단][redis-monitor]을 사용합니다.

### <a name="writing-to-the-cache-fails"></a>캐시에 쓰는 작업이 실패합니다.
**검색**. `StackExchange.Redis.RedisConnectionException`을 catch합니다.

**복구**

1. 일시적 장애 시 다시 시도합니다. Azure Redis Cache는 기본 제공 다시 시도를 지원합니다. [Redis 캐시 다시 시도 지침][redis-retry]을 참조하세요.
2. 오류가 일시적이지 않으면 이를 무시하고 나중에 다른 트랜잭션에서 캐시에 쓰도록 합니다.

**진단**. [Redis Cache 진단][redis-monitor]을 사용합니다.

## <a name="sql-database"></a>SQL Database
### <a name="cannot-connect-to-the-database-in-the-primary-region"></a>주 지역의 데이터베이스에 연결할 수 없습니다.
**검색**. 연결이 실패합니다.

**복구**

필수 구성 요소: 활성 지역 복제를 위한 데이터베이스를 구성해야 합니다. [SQL Database 활성 지역 복제][sql-db-replication]를 참조하세요.

* 쿼리의 경우 보조 복제본에서 읽습니다.
* 삽입 및 업데이트의 경우 수동으로 보조 복제본에 장애 조치합니다. [Azure SQL Database에 대해 계획되거나 계획되지 않은 장애 조치 시작][sql-db-failover]을 참조하세요.

복제본은 다른 연결 문자열을 사용하므로 응용 프로그램에서 연결 문자열을 업데이트해야 합니다.

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a>클라이언트에 연결 풀의 연결이 부족합니다.
**검색**. `System.InvalidOperationException` 오류를 catch합니다.

**복구**

* 작업을 다시 시도하세요.
* 완화 계획으로, 각 사용 사례에 대한 연결 풀을 격리하여 하나의 사용 사례에서 모든 연결을 점유할 수 없도록 합니다.
* 최대 연결 풀 수를 늘립니다.

**진단**. 응용 프로그램 로그.

### <a name="database-connection-limit-is-reached"></a>데이터베이스 연결 제한에 도달했습니다.
**검색**. Azure SQL Database는 동시 작업자, 로그인 및 세션의 수를 제한합니다. 이 제한은 서비스 계층에 따라 다릅니다. 자세한 내용은 [Azure SQL Database 리소스 제한][sql-db-limits]을 참조하세요.

이러한 오류를 감지하려면 `System.Data.SqlClient.SqlException`을 catch하고 SQL 오류 코드에서 `SqlException.Number` 값을 확인합니다. 관련 오류 코드 목록은 [SQL Database 클라이언트 응용 프로그램의 SQL 오류 코드: 데이터베이스 연결 오류 및 기타 문제][sql-db-errors]를 참조하세요.

**복구**. 이러한 오류는 일시적인 것으로 간주되므로 다시 시도하면 문제가 해결될 수 있습니다. 이러한 오류가 지속적으로 발생하면 데이터베이스 크기를 조정하는 것이 좋습니다.

**진단**. [sys.event_log][sys.event_log] 쿼리에서 성공적인 데이터베이스 연결, 연결 실패 및 교착 상태를 반환합니다.

* 실패한 연결에 대한 [경고 규칙][azure-alerts]을 만듭니다.
* [SQL Database 감사][sql-db-audit]를 사용하도록 설정하고 실패한 로그인을 확인합니다.

## <a name="service-bus-messaging"></a>Service Bus 메시징
### <a name="reading-a-message-from-a-service-bus-queue-fails"></a>Service Bus 큐에서 메시지를 읽는 작업이 실패합니다.
**검색**. 클라이언트 SDK에서 예외를 catch합니다. Service Bus 예외에 대한 기본 클래스는 [MessagingException][sb-messagingexception-class]입니다. 일시적인 오류인 경우 `IsTransient` 속성이 true입니다.

자세한 내용은 [Service Bus 메시징 예외][sb-messaging-exceptions]를 참조하세요.

**복구**

1. 일시적 장애 시 다시 시도합니다. [Service Bus 다시 시도 지침][sb-retry]을 참조하세요.
2. 받는 사람에게 배달할 수 없는 메시지는 *배달 못 한 편지 큐*에 넣습니다. 이 큐를 사용하여 받지 못한 메시지를 확인합니다. 배달 못 한 편지 큐에 대한 자동 정리는 지원되지 않습니다. 메시지는 명시적으로 검색할 때까지 이 큐에 남아 있습니다. [Service Bus 배달 못 한 편지 큐의 개요][sb-dead-letter-queue]를 참조하세요.

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a>Service Bus 큐에 메시지를 쓰는 작업이 실패합니다.
**검색**. 클라이언트 SDK에서 예외를 catch합니다. Service Bus 예외에 대한 기본 클래스는 [MessagingException][sb-messagingexception-class]입니다. 일시적인 오류인 경우 `IsTransient` 속성이 true입니다.

자세한 내용은 [Service Bus 메시징 예외][sb-messaging-exceptions]를 참조하세요.

**복구**

1. 일시적인 오류 후에 Service Bus 클라이언트에서 자동으로 다시 시도합니다. 기본적으로 지수는 백오프를 사용합니다. 최대 다시 시도 횟수 또는 최대 제한 시간 후에 클라이언트에서 예외를 throw합니다. 자세한 내용은 [Service Bus 다시 시도 지침][sb-retry]을 참조하세요.
2. 큐 할당량이 초과되면 클라이언트에서 [QuotaExceededException][QuotaExceededException]을 throw합니다. 예외 메시지에서 자세한 정보를 제공합니다. 다시 시도하기 전에 큐에서 일부 메시지를 배출하고, 회로 차단기 패턴을 사용하여 할당량이 초과되는 동안 계속 다시 시도하지 않도록 방지하는 것이 좋습니다. 또한 [BrokeredMessage.TimeToLive] 속성이 너무 높게 설정되지 않도록 합니다.
3. 지역 내에서 [분할된 큐 또는 토픽][sb-partition]을 사용하여 복원력을 향상시킬 수 있습니다. 분할되지 않은 큐나 항목은 하나의 메시징 저장소에 할당됩니다. 이 메시지 저장소를 사용할 수 없게 되면 해당 큐 또는 항목의 모든 작업이 실패하게 됩니다. 분할된 큐 또는 토픽은 여러 메시지 저장소로 분할됩니다.
4. 추가 복원력을 위해 서로 다른 지역에 두 개의 Service Bus 네임스페이스를 만들고 메시지를 복제합니다. 활성 복제 또는 수동 복제를 사용할 수 있습니다.

   * 활성 복제: 클라이언트에서 모든 메시지를 두 큐로 보냅니다. 수신기는 두 큐 모두에서 수신 대기합니다. 고유 식별자를 사용하여 메시지에 태그를 지정하면 클라이언트에서 중복 메시지를 버릴 수 있습니다.
   * 수동 복제: 클라이언트에서 하나의 큐에 메시지를 보냅니다. 오류가 있으면 클라이언트에서 다른 큐로 대체합니다. 수신기는 두 큐 모두에서 수신 대기합니다. 이 방식은 전송되는 중복 메시지의 수를 줄입니다. 그러나 여전히 수신기에서 중복 메시지를 처리해야 합니다.

     자세한 내용은 [GeoReplication 샘플][sb-georeplication-sample] 및 [Service Bus 가동 중단 및 재해로부터 응용 프로그램을 보호하기 위한 모범 사례](/azure/service-bus-messaging/service-bus-outages-disasters/)를 참조하세요.

### <a name="duplicate-message"></a>메시지가 중복되었습니다.
**검색**. 메시지의 `MessageId` 및 `DeliveryCount` 속성을 검사합니다.

**복구**

* 가능하면 메시지 처리 작업이 idempotent(멱등원)가 되도록 설계합니다. 그렇지 않으면 이미 처리된 메시지의 메시지 ID를 저장하고, 메시지를 처리하기 전에 이 ID를 확인합니다.
* `RequiresDuplicateDetection`이 true로 설정된 큐를 만들어 중복 검색을 사용하도록 설정합니다. 이 설정을 사용하면 Service Bus에서 이전 메시지와 동일한 `MessageId`로 보낸 모든 메시지를 자동으로 삭제합니다.  다음 사항에 유의하세요.

  * 이 설정은 중복 메시지를 큐에 넣지 않도록 방지합니다. 수신기에서 동일한 메시지를 두 번 이상 처리하지 않도록 방지합니다.
  * 중복 검색에는 기간이 있습니다. 중복 메시지가 이 기간을 초과하여 전송되면 검색되지 않습니다.

**진단**. 중복 메시지를 기록합니다.

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>응용 프로그램에서 큐의 특정 메시지를 처리할 수 없습니다.
**검색**. 응용 프로그램 특정입니다. 예를 들어 메시지에 유효하지 않은 데이터가 있거나 비즈니스 논리가 어떤 이유로 실패합니다.

**복구**

고려해야 할 두 가지 장애 모드가 있습니다.

* 수신기에서 장애를 검색합니다. 이 경우 메시지가 배달 못 한 편지 큐로 이동됩니다. 나중에 배달 못 한 편지 큐의 메시지를 검사하는 별도의 프로세스를 실행합니다.
* 예를 들어 처리되지 않은 예외로 인해 수신기가 메시지를 처리하는 중에 실패합니다. 이 경우를 처리하려면 `PeekLock` 모드를 사용합니다. 이 모드에서는 잠금이 만료되는 경우 다른 수신기에서 메시지를 사용할 수 있게 됩니다. 메시지가 최대 배달 횟수 또는 TTL(Time to Live)을 초과하면 메시지는 자동으로 배달 못 한 편지 큐로 이동됩니다.

자세한 내용은 [Service Bus 배달 못 한 편지 큐의 개요][sb-dead-letter-queue]를 참조하세요.

**진단**. 응용 프로그램에서 메시지를 배달 못 한 편지 큐로 이동할 때마다 응용 프로그램 로그에 이벤트를 기록합니다.

## <a name="service-fabric"></a>Service Fabric
### <a name="a-request-to-a-service-fails"></a>서비스에 대한 요청이 실패합니다.
**검색**. 서비스에서 오류를 반환합니다.

**복구**

* 프록시(`ServiceProxy` 또는 `ActorProxy`)를 다시 찾은 다음 서비스/행위자 메서드를 다시 호출합니다.
* **상태 저장 서비스**. 트랜잭션의 신뢰할 수 있는 컬렉션에 대한 작업을 래핑합니다. 오류가 있으면 트랜잭션이 롤백됩니다. 큐에서 끌어온 요청은 다시 처리됩니다.
* **상태 비저장 서비스**. 서비스에서 외부 저장소에 데이터를 유지하는 경우 모든 작업이 idempotent가 되어야 합니다.

**진단**. 응용 프로그램 로그

### <a name="service-fabric-node-is-shut-down"></a>Service Fabric 노드가 종료됩니다.
**검색**. 취소 토큰이 서비스의 `RunAsync` 메서드로 전달됩니다. Service Fabric에서 노드를 종료하기 전에 작업을 취소합니다.

**복구**. 취소 토큰을 사용하여 종료를 검색합니다. Service Fabric에서 취소를 요청하면 가능한 빨리 모든 작업을 끝내고  `RunAsync`을 종료합니다.

**진단**. 응용 프로그램 로그

## <a name="storage"></a>Storage
### <a name="writing-data-to-azure-storage-fails"></a>Azure Storage에 데이터를 쓰는 작업이 실패합니다.
**검색**. 쓰기 작업 중에 클라이언트에서 오류를 받습니다.

**복구**

1. 작업을 다시 시도하여 일시적인 장애로부터 복구합니다. 클라이언트 SDK의 [다시 시도 정책][Storage.RetryPolicies]에서 이 작업을 자동으로 처리합니다.
2. 저장소의 과부하를 방지하도록 회로 차단기 패턴을 구현합니다.
3. N회의 다시 시도가 실패하면 정상적인 대체(fallback)를 수행합니다. 예: 

   * 로컬 캐시에 데이터를 저장하고, 나중에 서비스를 사용할 수 있게 되면 저장소에 쓰기를 전달합니다.
   * 쓰기 작업이 트랜잭션 범위에 있는 경우 트랜잭션을 보정합니다.

**진단**. [저장소 메트릭][storage-metrics]을 사용합니다.

### <a name="reading-data-from-azure-storage-fails"></a>Azure Storage에서 데이터를 읽는 작업이 실패합니다.
**검색**. 읽기 작업 중에 클라이언트에서 오류를 받습니다.

**복구**

1. 작업을 다시 시도하여 일시적인 장애로부터 복구합니다. 클라이언트 SDK의 [다시 시도 정책][Storage.RetryPolicies]에서 이 작업을 자동으로 처리합니다.
2. RA-GRS 저장소의 경우 기본 엔드포인트에서 읽기 작업이 실패하면 보조 엔드포인트에서 읽기 작업을 시도합니다. 클라이언트 SDK에서 이 작업을 자동으로 처리할 수 있습니다. [Azure Storage 복제][storage-replication]를 참조하세요.
3. *N*회의 다시 시도가 실패하면 대체(fallback)를 수행하여 성능을 정상적으로 낮춥니다. 예를 들어 제품 이미지를 저장소에서 검색할 수 없는 경우 일반적인 자리 표시자 이미지가 표시됩니다.

**진단**. [저장소 메트릭][storage-metrics]을 사용합니다.

## <a name="virtual-machine"></a>Virtual Machine
### <a name="connection-to-a-backend-vm-fails"></a>백 엔드 VM에 대한 연결이 실패합니다.
**검색**. 네트워크 연결 오류입니다.

**복구**

* 부하 분산 장치 뒤에 있는 가용성 집합에 둘 이상의 백 엔드 VM을 배포합니다.
* 일시적인 연결 오류이면 TCP에서 메시지 전송을 성공적으로 다시 시도하는 경우가 있습니다.
* 응용 프로그램에서 다시 시도 정책을 구현합니다.
* 영구 오류 또는 일시적이지 않은 오류의 경우 [회로 차단기][circuit-breaker] 패턴을 구현합니다.
* 호출하는 VM에서 네트워크 송신 제한을 초과하면 아웃바운드 큐가 채워집니다. 아웃바운드 큐가 일관되게 가득 차면 크기를 확장하는 것이 좋습니다.

**진단**. 서비스 경계에서 이벤트를 로깅합니다.

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a>VM 인스턴스가 사용할 수 없거나 비정상적인 상태가 됩니다.
**검색**. VM 인스턴스가 정상인지 여부를 나타내는 Load Balancer [상태 프로브][lb-probe]를 구성합니다. 프로브를 통해 중요한 기능이 올바르게 응답하는지 여부를 확인해야 합니다.

**복구**. 각 응용 프로그램 계층에 대해 여러 VM 인스턴스를 동일한 가용성 집합에 배치하고, VM 앞에 Load Balancer를 배치합니다. 상태 프로브가 실패하면 Load Balancer에서 비정상 인스턴스에 대한 새 연결 전송을 중지합니다.

**진단**. Load Balancer [로그 분석][lb-monitor]을 사용합니다.

* 모든 상태 모니터링 엔드포인트를 모니터링하도록 모니터링 시스템을 구성합니다.

### <a name="operator-accidentally-shuts-down-a-vm"></a>운영자가 실수로 VM을 종료합니다.
**검색**. 해당 없음

**복구**. `ReadOnly` 수준의 리소스 잠금을 설정합니다. [Azure Resource Manager를 사용하여 리소스 잠그기][rm-locks]를 참조하세요.

**진단**. [Azure 활동 로그][azure-activity-logs]를 사용합니다.

## <a name="webjobs"></a>웹 작업
### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a>SCM 호스트가 유휴 상태일 때 연속 작업의 실행이 중지됩니다.
**검색**. 취소 토큰을 WebJob 함수에 전달합니다. 자세한 내용은 [정상 종료][web-jobs-shutdown]를 참조하세요.

**복구**. 웹앱에서 `Always On` 설정을 사용하도록 설정합니다. 자세한 내용은 [WebJobs로 백그라운드 작업 실행][web-jobs]을 참조하세요.

## <a name="application-design"></a>응용 프로그램 설계
### <a name="application-cant-handle-a-spike-in-incoming-requests"></a>응용 프로그램에서 들어오는 요청의 스파이크를 처리할 수 없습니다.
**검색**. 응용 프로그램에 따라 다릅니다. 일반적인 증상은 다음과 같습니다.

* 웹 사이트에서 HTTP 5xx 오류 코드를 반환하기 시작합니다.
* 데이터베이스 또는 저장소와 같은 종속 서비스에서 요청을 제한하기 시작합니다. 서비스에 따라 HTTP 429(너무 많은 요청)와 같은 HTTP 오류를 찾습니다.
* HTTP 큐 길이가 늘어납니다.

**복구**

* 규모를 확장하여 증가된 부하를 처리합니다.
* 연속 장애로 인해 전체 응용 프로그램이 중단되지 않도록 장애를 완화합니다. 완화 전략은 다음과 같습니다.

  * [패턴 제한][throttling-pattern]을 구현하여 백 엔드 시스템의 과부하를 방지합니다.
  * [큐 기반 부하 평준화][queue-based-load-leveling]를 사용하여 요청을 버퍼링하고 적절한 속도로 처리합니다.
  * 특정 클라이언트의 우선 순위를 지정합니다. 예를 들어 응용 프로그램에 체험 계층과 유료 계층이 있는 경우 체험 계층의 고객은 제한하고 유료 계층의 고객은 제한하지 않습니다. [우선 순위 큐 패턴][priority-queue-pattern]을 참조하세요.

**진단**. [App Service 진단 로깅][app-service-logging]을 사용합니다. [Azure Log Analytics][azure-log-analytics], [Application Insights][app-insights] 또는 [New Relic][new-relic]과 같은 서비스를 사용하면 진단 로그를 이해하는 데 도움이 됩니다.

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a>워크플로 또는 분산 트랜잭션의 작업 중 하나가 실패합니다.
**검색**. *N*회의 다시 시도를 수행한 후에도 계속 실패합니다.

**복구**

* 완화 계획으로, [스케줄러 에이전트 감독자][scheduler-agent-supervisor] 패턴을 구현하여 전체 워크플로를 관리합니다.
* 시간 제한 시 다시 시도를 수행하지 않습니다. 이 오류에 대한 성공률은 낮습니다.
* 나중에 다시 시도하기 위해 큐에 작업을 넣습니다.

**진단**. 보정 작업을 포함하여 모든 작업(성공 및 실패)을 기록합니다. 상관 관계 ID를 사용하여 동일한 트랜잭션 내의 모든 작업을 추적할 수 있습니다.

### <a name="a-call-to-a-remote-service-fails"></a>원격 서비스에 대한 호출이 실패합니다.
**검색**. HTTP 오류 코드입니다.

**복구**

1. 일시적 장애 시 다시 시도합니다.
2. *N*회의 시도 후에 호출이 실패하면 대체(fallback) 작업을 수행합니다. (응용 프로그램 특정입니다.)
3. 연속 장애를 방지하도록 [회로 차단기 패턴][circuit-breaker]을 구현합니다.

**진단**. 모든 원격 호출 실패를 기록합니다.

## <a name="next-steps"></a>다음 단계
FMA 프로세스에 대한 자세한 내용은 [클라우드 서비스 디자인에 의한 복원력][resilience-by-design-pdf](PDF 다운로드)을 참조하세요.

<!-- links -->

[api-management]: https://azure.microsoft.com/documentation/services/api-management/
[api-management-throttling]: /azure/api-management/api-management-sample-flexible-throttling/
[app-insights]: /azure/application-insights/app-insights-overview/
[app-insights-web-apps]: /azure/application-insights/app-insights-azure-web-apps/
[app-service-configure]: /azure/app-service-web/web-sites-configure/
[app-service-logging]: /azure/app-service-web/web-sites-enable-diagnostic-log/
[app-service-slots]: /azure/app-service-web/web-sites-staged-publishing/
[auto-rest-client-retry]: https://github.com/Azure/autorest/tree/master/docs
[azure-activity-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs/
[azure-alerts]: /azure/monitoring-and-diagnostics/insights-alerts-portal/
[azure-log-analytics]: /azure/log-analytics/log-analytics-overview/
[BrokeredMessage.TimeToLive]: https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx
[cassandra-error-handling]: http://www.datastax.com/dev/blog/cassandra-error-handling-done-right
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[cosmosdb-multi-region]: /azure/cosmos-db/tutorial-global-distribution-sql-api
[elasticsearch-azure]: ../elasticsearch/index.md
[elasticsearch-client]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[onstop-events]: https://azure.microsoft.com/blog/the-right-way-to-handle-azure-onstop-events/
[lb-monitor]: /azure/load-balancer/load-balancer-monitor-log/
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview/#learn-about-the-types-of-probes
[new-relic]: https://newrelic.com/
[priority-queue-pattern]: https://msdn.microsoft.com/library/dn589794.aspx
[queue-based-load-leveling]: https://msdn.microsoft.com/library/dn589783.aspx
[QuotaExceededException]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx
[ra-web-apps-basic]: ../reference-architectures/app-service-web-app/basic-web-app.md
[redis-monitor]: /azure/redis-cache/cache-how-to-monitor/
[redis-retry]: ../best-practices/retry-service-specific.md#azure-redis-cache-retry-guidelines
[resilience-by-design-pdf]: http://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
[RoleEntryPoint.OnStop]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[RoleEnvironment.Stopping]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx
[rm-locks]: /azure/azure-resource-manager/resource-group-lock-resources/
[sb-dead-letter-queue]: /azure/service-bus-messaging/service-bus-dead-letter-queues/
[sb-georeplication-sample]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/GeoReplication
[sb-messagingexception-class]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingexception.aspx
[sb-messaging-exceptions]: /azure/service-bus-messaging/service-bus-messaging-exceptions/
[sb-outages]: /azure/service-bus-messaging/service-bus-outages-disasters/#protecting-queues-and-topics-against-datacenter-outages-or-disasters
[sb-partition]: /azure/service-bus-messaging/service-bus-partitioning/
[sb-poison-message]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#poison
[sb-retry]: ../best-practices/retry-service-specific.md#service-bus-retry-guidelines
[search-sdk]: https://msdn.microsoft.com/library/dn951165.aspx
[scheduler-agent-supervisor]: https://msdn.microsoft.com/library/dn589780.aspx
[search-analytics]: /azure/search/search-traffic-analytics/
[sql-db-audit]: /azure/sql-database/sql-database-auditing-get-started/
[sql-db-errors]: /azure/sql-database/sql-database-develop-error-messages/#resource-governance-errors
[sql-db-failover]: /azure/sql-database/sql-database-geo-replication-failover-portal/
[sql-db-limits]: /azure/sql-database/sql-database-resource-limits/
[sql-db-replication]: /azure/sql-database/sql-database-geo-replication-overview/
[storage-metrics]: https://msdn.microsoft.com/library/dn782843.aspx
[storage-replication]: /azure/storage/storage-redundancy/
[Storage.RetryPolicies]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.aspx
[sys.event_log]: https://msdn.microsoft.com/library/dn270018.aspx
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[web-jobs]: /azure/app-service-web/web-sites-create-web-jobs/
[web-jobs-shutdown]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful
