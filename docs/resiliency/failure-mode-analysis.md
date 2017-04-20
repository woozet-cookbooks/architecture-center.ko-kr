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
**검색**. Catch the appropriate exception for the particular [Elasticsearch client][elasticsearch-client] being used.

**Recovery**

* Use a retry mechanism. Each client has its own retry policies.
* Deploy multiple Elasticsearch nodes and use replication for high availability.

For more information, see [Running Elasticsearch on Azure][elasticsearch-azure].

**Diagnostics**. You can use monitoring tools for Elasticsearch, or log all errors on the client side with the payload. See the 'Monitoring' section in [Running Elasticsearch on Azure][elasticsearch-azure].

### Writing data to Elasticsearch fails.
**검색**. Catch the appropriate exception for the particular [Elasticsearch client][elasticsearch-client] being used.  

**Recovery**

* Use a retry mechanism. Each client has its own retry policies.
* If the application can tolerate a reduced consistency level, consider writing with `write_consistency` setting of `quorum`.

For more information, see [Running Elasticsearch on Azure][elasticsearch-azure].

**Diagnostics**. You can use monitoring tools for Elasticsearch, or log all errors on the client side with the payload. See the 'Monitoring' section in [Running Elasticsearch on Azure][elasticsearch-azure].

## Queue storage
### Writing a message to Azure Queue storage fails consistently.
**검색**. After *N* retry attempts, the write operation still fails.

**Recovery**

* Store the data in a local cache, and forward the writes to storage later, when the service becomes available.
* Create a secondary queue, and write to that queue if the primary queue is unavailable.

**Diagnostics**. Use [storage metrics][storage-metrics].

### The application cannot process a particular message from the queue.
**검색**. Application specific. For example, the message contains invalid data, or the business logic fails for some reason.

**Recovery**

Move the message to a separate queue. Run a separate process to examine the messages in that queue.

Consider using Azure Service Bus Messaging queues, which provides a [dead-letter queue][sb-dead-letter-queue] functionality for this purpose.

> [!NOTE]
> If you are using Storage queues with WebJobs, the WebJobs SDK provides built-in poison message handling. See [How to use Azure queue storage with the WebJobs SDK][sb-poison-message].

**Diagnostics**. Use application logging.

## Redis Cache
### Reading from the cache fails.
**검색**. Catch `StackExchange.Redis.RedisConnectionException`.

**Recovery**

1. Retry on transient failures. Azure Redis cache supports built-in retry through See [Redis Cache retry guidelines][redis-retry].
2. Treat non-transient failures as a cache miss, and fall back to the original data source.

**Diagnostics**. Use [Redis Cache diagnostics][redis-monitor].

### Writing to the cache fails.
**검색**. Catch `StackExchange.Redis.RedisConnectionException`.

**Recovery**

1. Retry on transient failures. Azure Redis cache supports built-in retry through See [Redis Cache retry guidelines][redis-retry].
2. If the error is non-transient, ignore it and let other transactions write to the cache later.

**Diagnostics**. Use [Redis Cache diagnostics][redis-monitor].

## SQL Database
### Cannot connect to the database in the primary region.
**검색**. Connection fails.

**Recovery**

Prerequisite: The database must be configured for active geo-replication. See [SQL Database Active Geo-Replication][sql-db-replication].

* For queries, read from a secondary replica.
* For inserts and updates, manually fail over to a secondary replica. See [Initiate a planned or unplanned failover for Azure SQL Database][sql-db-failover].

The replica uses a different connection string, so you will need to update the connection string in your application.

### Client runs out of connections in the connection pool.
**검색**. Catch `System.InvalidOperationException` errors.

**Recovery**

* Retry the operation.
* As a mitigation plan, isolate the connection pools for each use case, so that one use case can't dominate all the connections.
* Increase the maximum connection pools.

**Diagnostics**. Application logs.

### Database connection limit is reached.
**검색**. Azure SQL Database limits the number of concurrent workers, logins, and sessions. The limits depend on the service tier. For more information, see [Azure SQL Database resource limits][sql-db-limits].

To detect these errors, catch `System.Data.SqlClient.SqlException` and check the value of `SqlException.Number` for the SQL error code. For a list of relevant error codes, see [SQL error codes for SQL Database client applications: Database connection error and other issues][sql-db-errors].

**Recovery**. These errors are considered transient, so retrying may resolve the issue. If you consistently hit these errors, consider scaling the database.

**Diagnostics**. - The [sys.event_log][sys.event_log] query returns successful database connections, connection failures, and deadlocks.

* Create an [alert rule][azure-alerts] for failed connections.
* Enable [SQL Database auditing][sql-db-audit] and check for failed logins.

## Service Bus Messaging
### Reading a message from a Service Bus queue fails.
**검색**. Catch exceptions from the client SDK. The base class for Service Bus exceptions is [MessagingException][sb-messagingexception-class]. If the error is transient, the `IsTransient` property is true.

For more information, see [Service Bus messaging exceptions][sb-messaging-exceptions].

**Recovery**

1. Retry on transient failures. See [Service Bus retry guidelines][sb-retry].
2. Messages that cannot be delivered to any receiver are placed in a *dead-letter queue*. Use this queue to see which messages could not be received. There is no automatic cleanup of the dead-letter queue. Messages remain there until you explicitly retrieve them. See [Overview of Service Bus dead-letter queues][sb-dead-letter-queue].

### Writing a message to a Service Bus queue fails.
**검색**. Catch exceptions from the client SDK. The base class for Service Bus exceptions is [MessagingException][sb-messagingexception-class]. If the error is transient, the `IsTransient` property is true.

For more information, see [Service Bus messaging exceptions][sb-messaging-exceptions].

**Recovery**

1. The Service Bus client automatically retries after transient errors. By default, it uses exponential back-off. After the maximum retry count or maximum timeout period, the client throws an exception. For more information, see [Service Bus retry guidelines][sb-retry].
2. If the queue quota is exceeded, the client throws [QuotaExceededException][QuotaExceededException]. The exception message gives more details. Drain some messages from the queue before retrying, and consider using the Circuit Breaker pattern to avoid continued retries while the quota is exceeded. Also, make sure the [BrokeredMessage.TimeToLive] property is not set too high.
3. Within a region, resiliency can be improved by using [partitioned queues or topics][sb-partition]. A non-partitioned queue or topic is assigned to one messaging store. If this messaging store is unavailable, all operations on that queue or topic will fail. A partitioned queue or topic is partitioned across multiple messaging stores.
4. For additional resiliency, create two Service Bus namespaces in different regions, and replicate the messages. You can use either active replication or passive replication.

   * Active replication: The client sends every message to both queues. The receiver listens on both queues. Tag messages with a unique identifier, so the client can discard duplicate messages.
   * Passive replication: The client sends the message to one queue. If there is an error, the client falls back to the other queue. The receiver listens on both queues. This approach reduces the number of duplicate messages that are sent. However, the receiver must still handle duplicate messages.

     For more information, see [GeoReplication sample][sb-georeplication-sample] and [Best practices for insulating applications against Service Bus outages and disasters](/azure/service-bus-messaging/service-bus-outages-disasters/).

### Duplicate message.
**검색**. Examine the `MessageId` and `DeliveryCount` properties of the message.

**Recovery**

* If possible, design your message processing operations to be idempotent. Otherwise, store message IDs of messages that are already processed, and check the ID before processing a message.
* Enable duplicate detection, by creating the queue with `RequiresDuplicateDetection` set to true. With this setting, Service Bus automatically deletes any message that is sent with the same `MessageId` as a previous message.  Note the following:

  * This setting prevents duplicate messages from being put into the queue. It doesn't prevent a receiver from processing the same message more than once.
  * Duplicate detection has a time window. If a duplicate is sent beyond this window, it won't be detected.

**Diagnostics**. Log duplicated messages.

### The application cannot process a particular message from the queue.
**검색**. Application specific. For example, the message contains invalid data, or the business logic fails for some reason.

**Recovery**

There are two failure modes to consider.

* The receiver detects the failure. In this case, move the message to the dead-letter queue. Later, run a separate process to examine the messages in the dead-letter queue.
* The receiver fails in the middle of processing the message &mdash; for example, due to an unhandled exception. To handle this case, use `PeekLock` mode. In this mode, if the lock expires, the message becomes available to other receivers. If the message exceeds the maximum delivery count or the time-to-live, the message is automatically moved to the dead-letter queue.

For more information, see [Overview of Service Bus dead-letter queues][sb-dead-letter-queue].

**Diagnostics**. Whenever the application moves a message to the dead-letter queue, write an event to the application logs.

## Service Fabric
### A request to a service fails.
**검색**. The service returns an error.

**Recovery**

* Locate a proxy again (`ServiceProxy` or `ActorProxy`) and call the service/actor method again.
* **Stateful service**. Wrap operations on reliable collections in a transaction. If there is an error, the transaction will be rolled back. The request, if pulled from a queue, will be processed again.
* **Stateless service**. If the service persists data to an external store, all operations need to be idempotent.

**Diagnostics**. Application log

### Service Fabric node is shut down.
**검색**. A cancellation token is passed to the service's `RunAsync` method. Service Fabric cancels the task before shutting down the node.

**Recovery**. Use the cancellation token to detect shutdown. When Service Fabric requests cancellation, finish any work and exit `RunAsync` as quickly as possible.

**Diagnostics**. Application logs

## Storage
### Writing data to Azure Storage fails
**검색**. The client receives errors when writing.

**Recovery**

1. Retry the operation, to recover from transient failures. The [retry policy][Storage.RetryPolicies] in the client SDK handles this automatically.
2. Implement the Circuit Breaker pattern to avoid overwhelming storage.
3. If N retry attempts fail, perform a graceful fallback. For example:

   * Store the data in a local cache, and forward the writes to storage later, when the service becomes available.
   * If the write action was in a transactional scope, compensate the transaction.

**Diagnostics**. Use [storage metrics][storage-metrics].

### Reading data from Azure Storage fails.
**검색**. The client receives errors when reading.

**Recovery**

1. Retry the operation, to recover from transient failures. The [retry policy][Storage.RetryPolicies] in the client SDK handles this automatically.
2. For RA-GRS storage, if reading from the primary endpoint fails, try reading from the secondary endpoint. The client SDK can handle this automatically. See [Azure Storage replication][storage-replication].
3. If *N* retry attempts fail, take a fallback action to degrade gracefully. For example, if a product image can't be retrieved from storage, show a generic placeholder image.

**Diagnostics**. Use [storage metrics][storage-metrics].

## Virtual Machine
### Connection to a backend VM fails.
**검색**. Network connection errors.

**Recovery**

* Deploy at least two backend VMs in an availability set, behind a load balancer.
* If the connection error is transient, sometimes TCP will successfully retry sending the message.
* Implement a retry policy in the application.
* For persistent or non-transient errors, implement the [Circuit Breaker][circuit-breaker] pattern.
* If the calling VM exceeds its network egress limit, the outbound queue will fill up. If the outbound queue is consistently full, consider scaling out.

**Diagnostics**. Log events at service boundaries.

### VM instance becomes unavailable or unhealthy.
**검색**. Configure a Load Balancer [health probe][lb-probe] that signals whether the VM instance is healthy. The probe should check whether critical functions are responding correctly.

**Recovery**. For each application tier, put multiple VM instances into the same availability set, and place a load balancer in front of the VMs. If the health probe fails, the Load Balancer stops sending new connections to the unhealthy instance.

**Diagnostics**. - Use Load Balancer [log analytics][lb-monitor].

* Configure your monitoring system to monitor all of the health monitoring endpoints.

### Operator accidentally shuts down a VM.
**검색**. N/A

**Recovery**. Set a resource lock with `ReadOnly` level. See [Lock resources with Azure Resource Manager][rm-locks].

**Diagnostics**. Use [Azure Activity Logs][azure-activity-logs].

## WebJobs
### Continuous job stops running when the SCM host is idle.
**검색**. Pass a cancellation token to the WebJob function. For more information, see [Graceful shutdown][web-jobs-shutdown].

**Recovery**. Enable the `Always On` setting in the web app. For more information, see [Run Background tasks with WebJobs][web-jobs].

## Application design
### Application can't handle a spike in incoming requests.
**검색**. Depends on the application. Typical symptoms:

* The website starts returning HTTP 5xx error codes.
* Dependent services, such as database or storage, start to throttle requests. Look for HTTP errors such as HTTP 429 (Too Many Requests), depending on the service.
* HTTP queue length grows.

**Recovery**

* Scale out to handle increased load.
* Mitigate failures to avoid having cascading failures disrupt the entire application. Mitigation strategies include:

  * Implement the [Throttling Pattern][throttling-pattern] to avoid overwhelming backend systems.
  * Use [queue-based load leveling][queue-based-load-leveling] to buffer requests and process them at an appropriate pace.
  * Prioritize certain clients. For example, if the application has free and paid tiers, throttle customers on the free tier, but not paid customers. See [Priority queue pattern][priority-queue-pattern].

**Diagnostics**. Use [App Service diagnostic logging][app-service-logging]. Use a service such as [Azure Log Analytics][azure-log-analytics], [Application Insights][app-insights], or [New Relic][new-relic] to help understand the diagnostic logs.

### One of the operations in a workflow or distributed transaction fails.
**검색**. After *N* retry attempts, it still fails.

**Recovery**

* As a mitigation plan, implement the [Scheduler Agent Supervisor][scheduler-agent-supervisor] pattern to manage the entire workflow.
* Don't retry on timeouts. There is a low success rate for this error.
* Queue work, in order to retry later.

**Diagnostics**. Log all operations (successful and failed), including compensating actions. Use correlation IDs, so that you can track all operations within the same transaction.

### A call to a remote service fails.
**검색**. HTTP error code.

**Recovery**

1. Retry on transient failures.
2. If the call fails after *N* attempts, take a fallback action. (Application specific.)
3. Implement the [Circuit Breaker pattern][circuit-breaker] to avoid cascading failures.

**Diagnostics**. Log all remote call failures.

## Next steps
For more information about the FMA process, see [Resilience by design for cloud services][resilience-by-design-pdf] (PDF download).

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
