---
title: Failure mode analysis
description: Guidelines for performing failure mode analysis for cloud solutions based on Azure.
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 03/24/2017
ms.author: pnp
ms.custom: resiliency

pnp.series.title: Design for Resiliency
---
# 장애 모드 분석
[!INCLUDE [header](../_includes/header.md)]

장애 모드 분석(Failure Mode Analysis, FMA)은 시스템의 예상 장애 지점을 식별하여 시스템에 복원력을 구축하는 프로세스입니다. FMA는 아키텍처 및 디자인 단계의 일부가 되어야 하며, 처음부터 장애 복원 기능을 시스템에 구축할 수 있도록 해야 합니다.

일반적인 FMA 수행 절차는 다음과 같습니다.

1. 시스템의 모든 구성 요소를 식별합니다. ID 제공업체, 타사 서비스 등 외부 종속성을 포함시킵니다.  
2. 각 구성 요소에 대해 발생 가능한 잠재적 장애를 식별합니다. 단일 구성 요소에 하나 이상의 고장 모드가 있을 수 있습니다. 예를 들어 읽기 오류와 쓰기 오류를 별도로 고려해야 하는데, 그 이유는 영향과 가능한 완화 조치가 다르기 때문입니다.
3. 전체적 위험도에 따라 각 장애 모드를 평가합니다. 아래 요소들을 고려합니다.

   * 장애 가능성이 어느 정도입니까? 비교적 일반적인 것입니까? 매우 드문 것입니까? 정확한 수치가 필요하지는 않으며, 그 목적은 우선순위를 정하는 데 도움을 주는 것입니다.
   * 가용성, 데이터 손실, 금전적 비용, 비즈니스 중단 측면에서 응응 프로그램에 대한 영향이 무엇입니까?
4. 각 장애 모드에 대해 응용 프로그램의 대응 및 복구 방법을 결정합니다. 비용 및 응용 프로그램의 복잡성 측면에서 장단점을 고려합니다.   

FMA 프로세스의 출발점으로서 본 문서에는 잠재적 장애 모드와 해당 완화책 카탈로그가 포함되어 있습니다. 카탈로그는 기술 또는 Azure 서비스별로 구성되며 응용 프로그램 수준의 설계에 대한 일반 범주도 포함되어 있습니다. 카탈로그에 모든 내용이 포함된 것은 아니며, 많은 핵심적 Azure 서비스를 다룹니다.

## 앱 서비스
### 앱 서비스 응용 프로그램이 종료됨.
**검색**. 예상 원인:

* 예상된 종료

  * 운영자가 예를 들어 Azure 포털을 통해서 응용 프로그램을 종료합니다.
  * 응용 프로그램이 유휴 상태여서 언로드되었습니다. (`Always On` 설정이 비활성화된 경우에만 해당됩니다.)
* 예상치 못한 종료

  * 응용 프로그램의 작동이 중단됩니다.
  * 앱 서비스 VM 인스턴스가 사용할 수 없는 상태가 됩니다.

Application_End 로깅이 앱 도메인 종료(소프트 프로세스 크래시)를 포착하며, 이것이 응용 프로그램 도메인 종료를 포착하는 유일한 방법입니다.

**복구**

* 종료가 예상되었으면, 응용 프로그램의 종료 이벤트를 사용하여 안정적으로 종료하십시오. 예를 들어 ASP.NET에서는 `Application_End` 메서드를 사용합니다.
* 유휴 상태에서 응용 프로그램이 언로드되었을 경우 다음 요청 시 자동으로 다시 시작됩니다. 하지만 "콜드 부팅" 비용이 발생합니다.
* 유휴 상태에서 응용 프로그램이 언로드되는 것을 방지하려면 웹 앱에서 `Always On` 설정을 활성화하십시오.  [Azure App Service에서 웹 앱 구성][app-service-configure]을 참조하십시오.
* 운영자가 앱을 종료하는 것을 방지하기 위해 리소스 잠금을 `ReadOnly` 수준으로 설정하십시오. [Azure Resource Manager로 리소스 잠금][rm-locks]을 참조하십시오.
* 앱 작동이 중단하거나 앱 서비스 VM이 사용할 수 없는 상태가 되면 App Service가 해당 앱을 자동으로 다시 시작합니다.

**진단**. 응용 프로그램 로그 및 웹 서버 로그. [Azure App Service에서 웹 앱의 진단 로깅 활성화][app-service-logging]를 참조하십시오.

### 특정 사용자가 반복하여 불량 요청을 하거나 시스템 과부하를 야기함.
**검색**. 사용자를 인증하고 사용자 ID를 응용 프로그램 로그에 포함시킵니다.

**복구**

* [Azure API Management][api-management]를 사용하여 해당 사용자의 요청을 제한합니다. [Azure API Management로 고급 요청 제한][api-management-throttling]
* 사용자를 차단합니다.

**진단**. 모든 인증 요청을 기록합니다.

### 불량 업데이트가 배포되었음.
**검색**. Azure 포털을 통해 응용 프로그램 상태를 모니터링하거나 (참조: [Azure 웹 앱 성능 모니터링][app-insights-web-apps]) 또는 [상태 끝점 모니터링 패턴][health-endpoint-monitoring-pattern]을 구현하십시오.

**복구**. 여러 [배포 슬롯][app-service-slots]을 사용하고, 마지막으로 성공한 배포로 롤백합니다. 자세한 내용은 [기본 웹 응용 프로그램][ra-web-apps-basic]을 참조하십시오.

## Azure Active Directory
### OpenID Connect (OIDC) 인증 실패.
**검색**. 예상 장애 모드의 예를 들면 다음과 같습니다.

1. Azure AD를 사용할 수 없거나, 또는 네트워크 문제로 인하여 도달할 수 없습니다. 인증 끝점으로 리디렉션이 실패하고, OIDC 미들웨어가 예외를 나타냅니다. 
2. Azure AD 테넌트가 없습니다. 인증 끝점으로 리디렉션하면 HTTP 오류 코드가 반환되고, OIDC 미들웨어가 예외를 나타냅니다.
3. 사용자가 인증할 수 없습니다. 검색 전략이 필요하지 않으며, Azure AD가 로그인 오류를 처리합니다.

**복구**

1. 미들웨어에서 미처리 예외를 포착합니다.
2. `AuthenticationFailed` 이벤트를 처리합니다.
3. 사용자를 오류 페이지로 리디렉션합니다.
4. 사용자가 다시 시도합니다.

## Azure Search
### Azure Search에 대한 데이터 쓰기가 실패함.
**검색**. `Microsoft.Rest.Azure.CloudException` 오류를 포착합니다.

**복구**

일시적 장애 후에 [Search .NET SDK][search-sdk]가 자동으로 재시도를 수행합니다. 클라이언트 SDK가 나타내는 모든 예외는 일시적이 아닌 오류로 취급해야 합니다.

기본 재시도 정책은 지수 백오프를 사용합니다. 다른 재시도 정책을 사용하려면 `SetRetryPolicy`를 `SearchIndexClient` 또는 `SearchServiceClient` 클래스에서 호출하십시오. 자세한 내용은 [자동 재시도][auto-rest-client-retry]를 참조하십시오.

**진단**. [검색 트래픽 분석][search-analytics]을 참조하십시오.

### Azure Search에서 데이터 읽기가 실패함.
**검색**. `Microsoft.Rest.Azure.CloudException` 오류를 포착합니다.

**복구**

일시적 장애 후에 [Search .NET SDK][search-sdk]가 자동으로 재시도를 수행합니다. 클라이언트 SDK가 나타내는 모든 예외는 일시적이 아닌 오류로 취급해야 합니다.

기본 재시도 정책은 지수 백오프를 사용합니다. 다른 재시도 정책을 사용하려면 `SetRetryPolicy`를 `SearchIndexClient` 또는 `SearchServiceClient` 클래스에서 호출하십시오. 자세한 내용은 [자동 재시도][auto-rest-client-retry]를 참조하십시오.

**진단**. [검색 트래픽 분석][search-analytics]을 참조하십시오.

## Cassandra
### 노드에 읽기 쓰기 실패.
**검색**. 예외를 포착합니다. .NET 클라이언트의 경우 일반적 예외는 `System.Web.HttpException`입니다. 다른 클라이언트에는 다른 예외 유형이 있을 수 있습니다. 자세한 내용은 [Cassandra의 올바른 오류 처리](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right)를 참조하십시오.

**복구**

* 각 [Cassandra 클라이언트](https://wiki.apache.org/cassandra/ClientOptions)는 자체 재시도 정책과 기능을 가지고 있습니다. 자세한 내용은 [Cassandra의 올바른 오류 처리][cassandra-error-handling]를 참조하십시오.
* 데이터 노드가 장애 도메인들에 걸쳐 분산된 랙 인식형 배포를 사용합니다.
* 로컬 쿼림 일관성이 있는 여러 지역에 배포합니다. 일시적이 아닌 장애가 발생할 경우 다른 지역으로 장애 조치를 합니다.

**진단**. 응용 프로그램 로그

## 클라우드 서비스
### 웹 역할 또는 작업자 역할이 예상치 못하게 종료됨.
**검색**. [RoleEnvironment.Stopping][RoleEnvironment.Stopping] 이벤트가 발생합니다.

**복구**. [RoleEntryPoint.OnStop][RoleEntryPoint.OnStop] 메서드를 무시하고 안정적으로 정리합니다. 자세한 내용은 [Azure OnStop 이벤트를 처리하는 올바른 방법][onstop-events] (블로그)을 참조하십시오.

## DocumentDB
### DocumentDB에서 데이터 읽기가 실패함.
**검색**. `System.Net.Http.HttpRequestException` 또는 `Microsoft.Azure.Documents.DocumentClientException`을 포착합니다.

**복구**

* SDK가 실패한 시도를 자동으로 재시도합니다. 재시도 횟수와 최대 대기 시간을 설정하려면 `ConnectionPolicy.RetryOptions`를 구성합니다. 클라이언트가 제기하는 예외는 재시도 정책을 벗어나거나 일시적 오류가 아닙니다.
* DocumentDB가 클라이언트를 제한할 경우 HTTP 429 오류를 반환합니다. `DocumentClientException`에서 상태 코드를 확인합니다. 오류 429가 지속적으로 발생하면 DocumentDB 컬렉션의 처리량 값을 높이는 것을 고려하십시오.
* DocumentDB 데이터베이스를 두 개 이상의 지역에 복제합니다. 모든 복제본은 읽을 수 있습니다. 클라이언트 SDK를 사용하여 `PreferredLocations` 매개 변수를 지정합니다. 이는 순서가 정해진 Azure 지역 목록입니다. 모든 읽은 내용이 목록의 첫 번째 사용 가능한 지역으로 전송됩니다. 요청이 실패할 경우 클라이언트가 목록의 다른 지역들을 순차적으로 시도합니다. 자세한 내용은 [다중 지역 DocumentDB 계정으로 개발][docdb-multi-region]을 참조하십시오.

**진단**. 클라이언트 측에 모든 오류를 기록합니다.

### DocumentDB에 데이터 쓰기가 실패함.
**검색**. `System.Net.Http.HttpRequestException` 또는 `Microsoft.Azure.Documents.DocumentClientException`을 포착합니다.

**복구**

* SDK가 실패한 시도를 자동으로 재시도합니다. 재시도 횟수와 최대 대기 시간을 설정하려면 `ConnectionPolicy.RetryOptions`를 구성합니다. 클라이언트가 제기하는 예외는 재시도 정책을 벗어나거나 일시적 오류가 아닙니다.
* DocumentDB가 클라이언트를 제한할 경우 HTTP 429 오류를 반환합니다. `DocumentClientException`에서 상태 코드를 확인합니다. 오류 429가 지속적으로 발생하면 DocumentDB 컬렉션의 처리량 값을 높이는 것을 고려하십시오.
* DocumentDB 데이터베이스를 두 개 이상의 지역에 복제합니다. 기본 지역에 장애가 발생하면 다른 지역이 쓸 수 있는 상태가 됩니다. 또한 수동으로 장애 조치를 트리거할 수도 있습니다. SDK가 자동 복구 및 라우팅을 수행하므로, 장애 조치 후에 응용 프로그램 코드가 계속 작동합니다. 장애 조치 기간(일반적으로 수분) 동안 에는 SDK가 새로운 쓰기 지역을 찾으므로 쓰기 작업의 대기 시간이 높습니다. 
  자세한 내용은 [다중 지역 DocumentDB 계정으로 개발][docdb-multi-region]을 참조하십시오.
* 대체 수단으로서 문서를 백업 큐에 유지하고 큐를 나중에 처리합니다.

**진단**. 클라이언트 측에 모든 오류를 기록합니다.

## Elasticsearch
### Elasticsearch에서 데이터 읽기가 실패함.
**검색**. 사용 중인 특정 [Elasticsearch 클라이언트][elasticsearch-client]의 해당 예외를 포착합니다.

**복구**

* 재시도 메커니즘을 사용합니다. 각 클라이언트는 자체 재시도 정책을 가지고 있습니다.
* 여러 Elasticsearch 노드를 배포하고 고가용성을 위해 복제를 사용합니다.

자세한 내용은 [Azure에서 Elasticsearch 실행하기][elasticsearch-azure]를 참조하십시오.

**진단**. Elasticsearch에 대한 모니터링 도구를 사용하거나, 또는 모든 오류를 페이로드가 있는 클라이언트 측에 로깅합니다. [Azure에서 Elasticsearch 실행하기][elasticsearch-azure]에서 '모니터링' 섹션을 참조하십시오.

### Elasticsearch에 데이터 쓰기가 실패함.
**검색**. 사용 중인 특정 [Elasticsearch 클라이언트][elasticsearch-client]의 해당 예외를 포착합니다.  

**복구**

* 재시도 메커니즘을 사용합니다. 각 클라이언트는 자체 재시도 정책을 가지고 있습니다.
* 응용 프로그램이 축소된 일관성 수준을 허용할 수 없다면, `quorum`의 `write_consistency` 설정으로 쓰는 것을 고려하십시오.

자세한 내용은 [Azure에서 Elasticsearch 실행하기][elasticsearch-azure]를 참조하십시오.

**진단**. Elasticsearch에 대한 모니터링 도구를 사용하거나, 또는 모든 오류를 페이로드가 있는 클라이언트 측에 로깅합니다. [Azure에서 Elasticsearch 실행하기][elasticsearch-azure]에서 '모니터링' 섹션을 참조하십시오.

## 큐 저장소
### Azure 큐 저장소에 메시지는 쓰는 작업이 계속 실패함.
**검색**. *N* 회 재시도를 수행한 후에도 쓰기 작업이 여전히 실패합니다.

**복구**

* 데이터를 로컬 캐시에 저장하고, 서비스가 사용 가능한 상태가 되면 나중에 쓰기 내용을 저장소로 전달합니다.
* 보조 큐를 만들고, 기본 큐를 사용할 수 없게 되면 보조 큐에 쓰기를 수행합니다.

**진단**. [저장소 메트릭][storage-metrics]을 사용하십시오.

### 응용 프로그램이 큐의 특정 메시지를 처리할 수 없습니다.
**검색**. 응용 프로그램별로 다릅니다. 예를 들어 메시지에 유효하지 않은 데이터가 포함되어 있거나, 어떠한 이유로 비즈니스 논리가 실패합니다.

**복구**

메시지를 별도 큐로 이동합니다. 별도 프로세스를 실행하여 큐에 있는 메시지를 검사합니다.

이 목적으로 [배달 못 한 편지 큐][sb-dead-letter-queue] 기능을 제공하는 Azure Service Bus 메시징 큐의 사용을 고려하십시오.

> [!참고]
> 웹 작업에 저장소 큐를 사용하고 있을 경우, 웹 작업 SDK가 포이즌 메시지 큐를 기본으로 제공합니다. [웹 작업 SDK로 Azure 큐 저장소를 사용하는 방법][sb-poison-message]을 참조하십시오.

**진단**. 응용 프로그램 로깅을 사용합니다.

## Redis 캐시
### 캐시에서 읽기 실패.
**검색**. `StackExchange.Redis.RedisConnectionException`을 포착합니다.

**복구**

1. 일시적 장애 시 재시도를 수행합니다. Azure Redis 캐시는 기본으로 재시도 기능을 지원합니다. [Redis 캐시 재시도 지침][redis-retry]을 참조하십시오.
2. 일시적이 아닌 장애는 캐시 미스(Cache Miss)로 취급하고, 원래 데이터 소스로 대체됩니다.

**진단**. [Redis 캐시 진단][redis-monitor]을 사용합니다.

### 캐시에 쓰기 실패함.
**검색**. `StackExchange.Redis.RedisConnectionException`을 포착합니다.

**복구**

1. 일시적 장애 시 재시도를 수행합니다. Azure Redis 캐시는 기본으로 재시도 기능을 지원합니다. [Redis 캐시 재시도 지침][redis-retry]을 참조하십시오.
2. 오류가 일시적인 것이 아니면 무시하고 다른 트랜잭션으로 하여금 나중에 캐시에 기록하도록 합니다.

**진단**. [Redis 캐시 진단][redis-monitor]을 사용합니다.

## SQL 데이터베이스
### 기본 지역의 데이터베이스에 연결할 수 없음.
**검색**. 연결이 실패합니다.

**복구**

선행 요구 사항: 데이터베이스에 활성 지리적 복제가 구성되어 있어야 합니다. [SQL 데이터베이스 활성 지리적 복제][sql-db-replication]를 참조하십시오.

* 쿼리의 경우 보조 복제본에서 읽습니다.
* 삽입 및 업데이트의 경우 수동으로 보조 복제본으로 장애 조치를 취합니다. [Azure SQL Database에 대해 계획된 또는 계획되지 않은 장애 조치 시작][sql-db-failover]을 참조하십시오.

복제본은 다른 연결 문자열을 사용하므로 응용 프로그램의 연결 문자열을 업데이트해야 합니다.

### 연결 풀에서 클라이언트의 연결이 소진됨.
**검색**. `System.InvalidOperationException` 오류를 포착합니다.

**복구**

* 작업을 다시 시도합니다.
* 완화 계획으로서 각 사용 건에 대해 연결 풀을 격리함으로써 하나의 사용 건이 전체 연결을 지배할 수 없도록 합니다.
* 최대 연결 풀의 수를 늘립니다.

**진단**. 응용 프로그램 로그.

### 데이터베이스 연결 제한에 도달함.
**검색**. Azure SQL Database는 동시 작업자, 로그인 및 세션의 수를 제한합니다. 이 제한은 서비스 계층에 따라 다릅니다. 자세한 내용은 [Azure SQL Database 리소스 제한][sql-db-limits]을 참조하십시오.

이들 오류를 검색하려면 `System.Data.SqlClient.SqlException` 을 포착하고 SQL 오류 코드에서 `SqlException.Number`의 값을 확인합니다. 해당 오류 코드 목록은 [SQL 데이터베이스 클라이언트 응용 프로그램의 SQL 오류 코드: 데이터베이스 연결 오류 및 기타 문제][sql-db-errors]를 참조하십시오.

**복구**. 이들 오류는 일시적인 것으로 간주되므로 재시도를 하면 문제가 해결될 수 있습니다. 이들 오류가 지속적으로 발생하면 데이터베이스 확장을 고려하십시오.

**진단**. - [sys.event_log][sys.event_log] 쿼리가 성공적 데이터베이스 연결, 연결 실패 및 교착 상태 정보를 반환합니다.

* 실패한 연결에 대해 [알림 규칙][azure-alerts]을 만듭니다.
* [SQL 데이터베이스 감사][sql-db-audit]를 활성화하고 실패한 로그인을 확인합니다.

## Service Bus 메시징
### Service Bus 큐의 메시지 읽기 실패.
**검색**. 클라이언트 SDK의 예외를 포착합니다. Service Bus 예외의 기본 클래스는 [MessagingException][sb-messagingexception-class]입니다. 오류가 일시적이면 `IsTransient` 속성이 true입니다.

자세한 내용은 [Service Bus 메시징 예외][sb-messaging-exceptions]를 참조하십시오.

**복구**

1. 일시적 장애 시 재시도를 수행합니다. [Service Bus 재시도 지침][sb-retry]을 참조하십시오.
2. 수신기에 전달할 수 없는 메시지는 *배달 못 한 편지 큐* 에 넣습니다. 이 큐를 사용하여 어떤 메시지를 수신할 수 없었는지 확인합니다. 배달 못 한 편지 큐의 자동 정리 기능은 없습니다. 해당 메시지들이 확실하게 수신될 때까지 거기에 계속 유지됩니다. [Service Bus 배달 못 한 편지 큐][sb-dead-letter-queue]를 참조하십시오.

### Service Bus 큐에 메시지 쓰기 실패.
**검색**. 클라이언트 SDK의 예외를 포착합니다. Service Bus 예외의 기본 클래스는 [MessagingException][sb-messagingexception-class]입니다. 오류가 일시적이면 `IsTransient` 속성이 true입니다.

자세한 내용은 [Service Bus 메시징 예외][sb-messaging-exceptions]를 참조하십시오.

**복구**

1. 일시적 오류 후에 Service Bus 클라이언트가 자동으로 재시도를 수행합니다. 기본적으로 지수 백오프를 사용합니다. 최다 재시도 횟수 또는 최대 시간 초과 기간이 경과하고 나면 클라이언트가 예외를 발생시킵니다. 자세한 내용은 [Service Bus 재시도 지침][sb-retry]을 참조하십시오.
2. 큐 할당량이 초과하면 클라이언트가 [QuotaExceededException][QuotaExceededException]을 발생시킵니다. 예외 메시지가 보다 자세한 정보를 제공합니다. 재시도 전에 큐에서 일부 메시지를 배출하고, 할당량이 초과할 때 지속적 재시도를 방지하기 위해 회로 차단기를 사용하는 것을 고려하십시오. 또한 [BrokeredMessage.TimeToLive] 속성을 너무 높게 설정하지 않도록 하십시오.
3. [분할된 큐 또는 주제][sb-partition]를 사용하면 한 지역 내에서 복원력이 향상될 수 있습니다. 분할되지 않은 큐나 주제는 하나의 메시징 스토어에 할당됩니다. 이 메시징 스토어를 사용할 수 없으며, 해당 큐나 주제의 모든 작업이 실패합니다. 분할된 큐나 주제가 여러 메시징 스토어에 걸쳐 분할됩니다.
4. 추가적인 복원력을 확보하려면 다른 지역에 두 개의 Service Bus 네임스페이스를 만들어 메시지를 복제합니다. 액티브 복제 또는 패시브 복제를 사용할 수 있습니다.

   * 액티브 복제: 클라이언트가 모든 메시지를 양쪽 큐에 전송합니다. 수신기는 양쪽 대기열에서 모두 수신합니다. 클라이언트가 중복 메시지를 버릴 수 있도록 고유한 식별자로 메시지에 태그를 지정합니다.
   * 패시브 복제: 클라이언트가 메시지를 하나의 큐에 전송합니다. 오류가 발생하면 클라이언트가 다른 큐로 전환합니다. 수신기가 양쪽 대기열에서 모두 수신합니다. 이 접근법은 전송되는 중복 메시지의 수를 줄여줍니다. 하지만 여전히 수신기가 중복 메시지를 처리해야 합니다.

     자세한 내용은 [GeoReplication 샘플][sb-georeplication-sample] 및 [Service Bus 중단 및 재해로부터 응용 프로그램을 분리하는 모범 사례](/azure/service-bus-messaging/service-bus-outages-disasters/)를 참조하십시오.

### 중복 메시지.
**검색**. 메시지의 `MessageId` 및 `DeliveryCount` 속성을 검사합니다.

**복구**

* 가능하면 메시지 처리 작업을 idempotent가 되도록 설계합니다. 그렇지 않으면 이미 처리된 메시지의 메시지 ID를 저장하고 메시지를 처리하기 전에 ID를 확인합니다.
* `RequiresDuplicateDetection`을 true로 설정하여 큐를 생성함으로써 중복 검색을 활성화합니다. 이 설정을 통해서 Service Bus는 이전 메시지와 동일한 `MessageId`로 전송되는 모든 메시지를 자동으로 삭제합니다. 아래 내용에 유의하십시오.

  * 이 설정은 중복 메시지를 큐에 넣는 것을 방지합니다. 수신기가 동일한 메시지를 두 번 이상 처리하는 것을 방지하지는 않습니다.
  * 중복 검색에는 시간대가 있습니다. 이 시간대를 벗어나서 중복 메시지가 전송되면 검색되지 않습니다.

**진단**. 중복 메시지를 로깅합니다.

### 응용 프로그램이 큐의 특정 메시지를 처리할 수 없습니다.
**검색**. 응용 프로그램별로 다릅니다. 예를 들어 메시지에 유효하지 않은 데이터가 포함되어 있거나, 어떠한 이유로 비즈니스 논리가 실패합니다.

**복구**

고려해야 할 장애 모드가 두 가지 있습니다.

* 수신기가 장애를 감지합니다. 이 경우, 메시지를 배달 못 한 편지 큐로 이동시킵니다. 나중에 별도 프로세스를 실행하여 배달 못 한 편지 큐에 있는 메시지를 검사합니다.
* 예를 들어 미처리 예외로 인하여 메시지 처리 중에 수신기에 장애가 발생합니다. 이 경우를 처리하려면 `PeekLock` 모드를 사용합니다. 이 모드에서는 잠금이 만료될 경우, 다른 수신기에서 메시지를 사용할 수 있습니다. 메시지가 최대 배달 횟수 또는 TTL(Time to Live)을 초과할 경우, 메시지가 배달 못 한 편지 큐로 자동으로 이동됩니다.

자세한 내용은 [Service Bus 배달 못 한 편지 큐 개요][sb-dead-letter-queue]를 참조하십시오.

**진단**. 응용 프로그램이 메시지를 배달 못 한 편지 큐로 이동할 때마다 이벤트를 응용 프로그램 로그에 기록합니다.

## 서비스 패브릭
### 서비스 요청 실패.
**검색**. 서비스가 오류를 반환합니다.

**복구**

* 프록시를 다시 찾아 (`ServiceProxy` 또는 `ActorProxy`), 서비스/행위자 메서드를 다시 호출합니다.
* **상태 저장 서비스**. 작업들을 트랜잭션의 신뢰성 있는 컬렉션에 래핑합니다. 오류가 있으면 트랜잭션이 롤백됩니다. 요청을 큐에서 가져오면 다시 처리됩니다.
* **상태 비저장 서비스**. 서비스가 데이터를 외부 스토어에 유지할 경우, 모든 작업이 idempotent여야 합니다.

**진단**. 응용 프로그램 로그

### 서비스 패브릭 노드가 종료됨.
**검색**. 취소 토큰이 서비스의 `RunAsync` 메서드로 전달됩니다. 서비스 패브릭이 노드를 종료하기 전에 작업을 취소합니다.

**복구**. 종료를 감지하기 위해 취소 토큰을 사용합니다. 서비스 패브릭이 취소를 요구하면 최대한 신속히 모든 작업을 마치고 `RunAsync`를 종료합니다.

**진단**. 응용 프로그램 로그

## 저장소
### Azure 저장소에 데이터 쓰기가 실패함.
**검색**. 쓰기 작업 중에 클라이언트가 오류를 수신합니다.

**복구**

1. 작업을 재시도하여 일시적 장애로부터 복구합니다. 클라이언트 SDK의 [재시도 정책][Storage.RetryPolicies]이 이를 자동으로 처리합니다.
2. 저장소의 과부하를 방지하도록 회로 차단기 패턴을 구현합니다.
3. N 재시도 횟수가 실패할 경우 안정적인 대체 수단을 수행합니다. 예를 들면 다음과 같습니다.

   * 데이터를 로컬 캐시에 저장하고, 서비스가 사용 가능한 상태가 되면 나중에 쓰기 내용을 저장소로 전달합니다.
   * 쓰기 작업이 트랜잭션 범위 내에 있을 경우, 트랜잭션을 보상합니다.

**진단**. [저장소 메트릭][storage-metrics]을 사용하십시오.

### Azure 저장소에서 데이터 읽기가 실패함.
**검색**. 읽기 작업 중에 클라이언트가 오류를 수신합니다.

**복구**

1. 작업을 재시도하여 일시적 장애로부터 복구합니다. 클라이언트 SDK의 [재시도 정책][Storage.RetryPolicies]이 이를 자동으로 처리합니다.
2. RA-GRS 저장소의 경우, 기본 끝점의 읽기가 실패한 경우 보조 끝점에서 읽기를 시도합니다. 클라이언트 SDK가 이를 자동으로 처리할 수 있습니다. [Azure Storage 복제][storage-replication]를 참조하십시오.
3. *N* 재시도 횟수가 실패하면 대체 조치를 취하여 안정적인 기능 저하를 유도합니다. 예를 들어 저장소에서 제품 이미지를 가져올 수 없을 경우 일반 자리 표시자 이미지를 보여줍니다.

**진단**. [저장소 메트릭][storage-metrics]을 사용하십시오.

## 가상 컴퓨터
### 백엔드 VM 연결 실패.
**검색**. 네트워크 연결 오류.

**복구**

* 부하 분산 장치 뒤에, 적어도 두 개의 백엔드 VM을 가용성 집합에 배포합니다.
* 연결 오류가 일시적이면, TCP가 메시지 전송 재시도를 성공적으로 수행하기도 합니다.
* 응용 프로그램에 재시도 정책을 구현합니다.
* 지속적 또는 일시적이 아닌 오류의 경우 [회로 차단기][circuit-breaker] 패턴을 구현합니다.
* 호출 VM이 해당 네트워크 송신 제한을 초과할 경우 아웃바운드 큐가 채워집니다. 아웃바운드 큐가 지속적으로 가득 차면 확장을 고려하십시오.

**진단**. 서비스 경계에서 이벤트를 로그하십시오

### VM 인스턴스가 사용 불가 또는 비정상 상태가 됨.
**검색**. VM 인스턴스가 정상인이 여부를 표시하는 부하 분산 장치 [상태 검색][lb-probe]을 구성합니다. 이 검색을 통해서 주요 기능이 올바로 응답하는지 여부를 확인해야 합니다.

**복구**. 각 응용 프로그램 계층에 대해 여러 VM 인스턴스를 동일한 가용성 집합에 넣고, 부하 분산 장치를 VM 앞에 배치합니다. 상태 검색이 실패할 경우, 부하 분산 장치가 새로운 연결을 비정상 인스턴스에 전송하는 것을 중지합니다.

**진단**. - 부하 분산 장치 [로그 분석][lb-monitor]을 사용합니다.

* 모든 상태 모니터링 끝점들을 모니터링하도록 모니터링 시스템을 구성합니다.

### 운영자가 실수로 VM을 종료함.
**검색**. 해당 없음

**복구**. 리소스 잠금을 `ReadOnly` 수준으로 설정합니다. [Azure Resource Manager로 리소스 잠금][rm-locks]을 참조하십시오.

**진단**. [Azure 활동 로그][azure-activity-logs]를 사용합니다.

## 웹 작업
### SCM 호스트가 유휴 상태일 때 연속 작업의 실행이 중지됨.
**검색**. 취소 토큰을 웹 작업 기능에 전달합니다. 자세한 내용은 [정상 종료][web-jobs-shutdown]를 참조하십시오.

**복구**. 웹 앱에서 `Always On` 설정을 활성화합니다. 자세한 내용은 [웹 작업으로 백그라운드 작업 실행][web-jobs]을 참조하십시오.

## 응용 프로그램 설계
### 응용 프로그램이 들어오는 요청의 급증을 처리할 수 없음.
**검색**. 응용 프로그램에 따라 다릅니다. 일반적인 증상:

* 웹사이트가 HTTP 5xx 오류 코드를 반환하기 시작합니다.
* 데이터베이스, 저장소 등 종속 서비스가 요청을 제한하기 시작합니다. 서비스에 따라 HTTP 429 (너무 많은 요청) 등 HTTP 오류를 찾습니다.
* HTTP 큐 길이가 증가합니다.

**복구**

* 부하 증가를 처리하려면 확장합니다.
* 연속 장애가 전체 응용 프로그램을 중단시키는 것을 방지하기 위해 장애를 완화합니다. 완화 전략의 예를 들면 다음과 같습니다.

  * [제한 패턴][throttling-pattern]을 구현하여 백엔드 시스템의 과부하를 방지합니다.
  * [큐 기반 부하 평준화][queue-based-load-leveling]를 사용하여 요청을 버퍼링하여 적절한 속도로 처리합니다.
  * 특정 클라이언트들의 우선 순위를 정합니다. 예를 들어 응용 프로그램에 무료 및 유료 계층이 있을 경우, 무료 계층의 고객들은 제한하지만 유료 고객들은 제한하지 않습니다. [우선순위 큐 패턴][priority-queue-pattern]을 참조하십시오.

**진단**. [앱 서비스 진단 로깅][app-service-logging]을 사용하십시오. 진단 로그를 이해하는 데 도움이 되도록 [Azure Log Analytics][azure-log-analytics], [Application Insights][app-insights] 또는 [New Relic][new-relic] 등의 서비스를 사용하십시오.

### 워크플로 또는 분산 트랜잭션의 작업 중 하나가 실패함.
**검색**. *N* 회 재시도를 수행한 후에도 여전히 실패합니다.

**복구**

* 완화 계획으로서 [Scheduler Agent Supervisor][scheduler-agent-supervisor] 패턴을 사용하여 전체 워크플로를 관리합니다.
* 시간 초과 시 재시도를 수행하지 마십시오. 이 오류는 성공률이 낮습니다.
* 나중에 재시도하기 위해 작업을 큐에 넣습니다.

**진단**. 보상 작업을 포함하여 모든 작업(성공 작업 및 실패 작업)을 기록합니다. 동일 트랜잭션 내에서 모든 작업을 추적할 수 있도록 상관 관계 ID를 사용합니다.

### 원격 서비스 호출 실패.
**검색**. HTTP 오류 코드.

**복구**

1. 일시적 장애 시 재시도를 수행합니다.
2. *N* 회 시도 후에 호출이 실패할 경우 대체 조치를 취합니다. (응용 프로그램별로 다름).
3. 연속 장애를 방지하기 위해 [회로 차단기 패턴][circuit-breaker]을 구현합니다.

**진단**. 모든 원격 호출 장애를 기록합니다.

## 다음 단계
FMA 프로세스에 관한 추가 정보는 [클라우드 서비스 디자인에 의한 복원력][resilience-by-design-pdf] (PDF 다운로드)을 참조하십시오.

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
[docdb-multi-region]: /azure/documentdb/documentdb-developing-with-multiple-regions/
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
[ra-web-apps-basic]: ../reference-architectures/managed-web-app/basic-web-app.md
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
