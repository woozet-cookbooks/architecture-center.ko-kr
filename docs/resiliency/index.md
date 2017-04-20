---
title: Design resilient applications
description: How to build resilient applications in Azure, for high availability and disaster recovery.
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 03/24/2017
ms.author: pnp
ms.custom: resiliency

pnp.series.title: Design for Resiliency
---
# 복원 가능한 Azure용 응용 프로그램 설계

분산 시스템에서도 장애가 발생합니다. 하드웨어 고장도 발생할 수 있습니다. 네트워크에 일시적 장애가 발생할 수 있습니다. 드물게 전체 서비스나 지역이 중단될 수 있지만, 그에 대한 계획도 세워야 합니다.

클라우드에 신뢰성 있는 응용 프로그램을 구축하는 것은 엔터프라이즈 환경에 신뢰성 있는 응용 프로그램을 구축하는 것과는 다릅니다. 과거에는 수직 확장을 위해 고급 사양의 하드웨어를 구매하였을 수 있지만, 클라우드 환경에서는 수직 확장 대신에 수평 확장을 해야 합니다. 상용 하드웨어를 사용함으로써 클라우드 환경 비용이 낮게 유지됩니다. 장애 방지 및 "평균 무고장 시간" 최적화에 초점을 맞추는 대신에, 이 새로운 환경에서는 "평균 복원 시간"에 초점을 맞춥니다. 목표는 장애의 영향을 최소화하는 것입니다.

이 문서에서는 Microsoft Azure에서 복원 가능한 응용 프로그램을 구축하는 방법의 개요를 설명합니다. 우선 *복원력* 이라는 용어와 관련 개념의 정의부터 시작합니다. 그 다음에 설계, 구현, 배포, 운영에 이르기까지 응용 프로그램의 수명 동안 구조화된 접근법을 사용하여 복원력을 달성하는 프로세스를 설명합니다.

## 복원력이란 무엇일까요?
**복원력** 은 장애로부터 복구하여 계속 기능을 발휘하는 것을 말합니다. 이는 장애를 *피하는 것* 이 아니라 가동 중시 또는 데이터 손실을 방지하는 방식으로 장애에 *대응* 하는 것을 말합니다. 복원의 목표는 장애 이후에 응용 프로그램을 완전히 작동하는 상태로 되돌리는 것입니다.

복원의 중요한 두 가지 측면은 고가용성과 장애 복구입니다.

* **고가용성** (HA) 은 응용 프로그램이 유의미한 가동 중지 없이 정상 상태로 계속 작동하게 하는 능력입니다. "정상 상태"란 응용 프로그램이 응답하고 사용자가 응용 프로그램에 연결하여 상호작용할 수 있는 상태를 의미합니다.  
* **재해 복구** (DR) 는 드물지만 중대한 사고 즉 전체 지역에 영향을 주는 서비스 중단처럼 일시적이지 않고 광범위한 규모의 장애로부터 복구하는 능력을 말합니다. 재해 복구에는 데이터 백업과 보관이 포함되며, 백업에서 데이터베이스를 복원하는 등 수동 개입도 포함될 수 있습니다.

DR에 대비하여 HA를 생각하는 한 가지 방법은, DR은 장애의 영향이 설계된 HA의 처리 능력을 초과했을 때 시작됩니다. 예를 들어, 부하 분산 장치 뒤에 몇 개의 VM을 두면 하나의 VM에 장애가 발생해도 가용성을 제공하지만, 그 VM 모두에 동시에 장애가 발생하면 그렇게 할 수 없습니다.

복원 가능한 응용 프로그램을 설계할 때에는 가용성 요구사항을 이해해야 합니다. 어느 정도의 가동 중지 시간이 허용 가능합니까? 이는 부분적으로 비용의 함수입니다. 잠재적 가동 중지로 얼마의 비즈니스 비용이 발생할까요? 응용 프로그램의 고가용성 확보를 위해 얼마를 투자해야 할까요? 또한 응용 프로그램의 가용성에 대한 정의도 해야 합니다. 예를 들어, 고객이 주문서를 제출할 수 있지만 시스템이 정상 기간 내에 처리할 수 없다면 응용 프로그램이 "다운"되었다고 간주합니까?

또 하나의 일반적인 용어가 **비즈니스 연속성**(BC)인데, 이는 재해 기간 또는 그 후에 필수적 비즈니스 기능을 수행하는 능력을 말합니다. BC는 물리적 시설, 인력, 커뮤니케이션, 운송, IT 등 전체 비즈니스 운영에 적용됩니다. 이 문서에서는 클라우드 응용 프로그램에만 초점을 맞추지만, 복원 계획은 전반적 BC 요구 사항 맥락에서 수행해야 합니다. 

## 복원력 달성을 위한 프로세스
복원력은 애드온이 아닙니다. 이는 반드시 시스템 내에 설계하고 운영 시 실행해야 하는 것입니다. 따라야 할 일반적인 모델은 다음과 같습니다.

1. **정의:** 비즈니스 요구를 기준으로 가용성 요구 사항을 정의합니다.
2. **설계:** 응용 프로그램의 복원력을 설계합니다. 검증된 관행을 따르는 아키텍처로 시작한 후, 그 아키텍처에서 예상되는 장애 지점을 식별합니다.
3. **구현:** 장애를 감지하고 복원하는 전략을 구현합니다.
4. **테스트:** 장애를 시뮬레이션하고 강제 장애 조치를 트리거하여 구현 내용을 테스트합니다.
5. **배포:** 신뢰성 있고 반복할 수 있는 프로세스를 사용하여 응용 프로그램을 생산 환경에 배포합니다.
6. **모니터링:** 응용 프로그램을 모니터링하여 장애를 감지합니다. 시스템을 모니터링함으로써 응용 프로그램의 상태를 측정하고 필요 시 사고에 대응할 수 있습니다.
7. **대응:** 수동 개입이 필요한 사고가 있을 경우에 대응합니다.

이 문서의 나머지 부분에서는 이들 각 단계에 관해서 자세히 설명합니다.

## 복원 요구 사항의 정의
복원 계획은 비즈니스 요구 사항에서 시작됩니다. 그러한 조건에서 복원력에 관하여 생각하는 몇 가지 접근법이 있습니다.

### 작업을 기준으로 구성 해체
다수의 클라우드 솔루션은 여러 응용 프로그램 작업으로 구성됩니다. 이러한 맥락에서 "작업"(workload)이란 용어는 비즈니스 논리와 데이터 저장 요구 사항 측면에서 다른 작업과 논리적으로 분리될 수 있는 개별 기능 또는 컴퓨팅 작업을 의미합니다. 예를 들어, 전자 상거래 앱에는 아래의 작업이 포함될 수 있습니다.

* 제품 카탈로그 찾아보기 및 검색.
* 주문서 작성 및 추적.
* 권장 사항 보기.

이들 작업의 가용성, 확장성, 데이터 일관성, 재해 복구 등에 대한 요구 사항은 다를 수 있습니다. 다시 말하지만 그것은 비즈니스 결정 사항입니다.

또한 이용 패턴도 생각하십시오. 시스템이 반드시 가용성을 유지해야 할 중요한 특정 기간이 있습니까? 예를 들어 신고 기간 직전에 세무 신고 서비스가 다운되어서는 안 되며, 큰 스포츠 이벤트 기간 동안에는 비디오 스트리밍 서비스가 반드시 작동되어야 합니다. 중요한 기간에는, 한 지역에 장애가 발생해도 응용 프로그램이 장애 조치를 취할 수 있도록 여러 지역에 걸쳐 중복 배포를 보유할 수 있습니다. 하지만 다중 지역 배포는 비용이 더 많이 소요되므로 덜 중요한 기간에는 응용 프로그램을 단일 지역에서 실행할 수 있습니다.

### RTO와 RPO
고려해야 할 중요한 두 가지 메트릭은 복구 시간 목표와 복구 지점 목표입니다.

* **복구 시간 목표** (RTO)는 사고 후에 응용 프로그램의 사용 불가 상태를 허용하는 최대 수용 가능한 시간입니다. RTO가 90분이면 재해 시작 시점부터 90분 이내에 응용 프로그램을 작동 상태로 복원할 수 있어야 합니다. RTO 시간이 매우 낮으면 지역의 서비스 중단을 방지하기 위해 보조 배포를 계속 대기 상태로 운영하는 것이 필요할 수 있습니다.
* **복구 지점 목표** (RPO) 는 재해 시 수용 가능한 최대 데이터 손실 기간을 말합니다. 예를 들어 데이터를 단일 데이터베이스에 저장하는데 다른 데이터베이스에 복제하지 않고 매시간 백업을 수행한다면, 최대 1시간 분량의 데이터가 손실될 수 있습니다.

RTO와 RPO는 비즈니스 요구 사항입니다. 또 하나의 일반적 메트릭으로 **평균 복구 시간** (MTTR)이 있는데 이는 장애 후에 응용 프로그램을 복구하는 데 소요되는 평균 시간을 말합니다. MTTR는 시스템에 관한 경험적 사실입니다. MTTR가 RTO를 초과하면 정의된 RTO 내에 시스템을 복원할 수 없기 때문에 시스템 장애가 수용 불가능한 비즈니스 중단을 야기합니다. 

### SLAs
Azure에서 [서비스 수준 계약][sla] (SLA)은 가동 시간과 연결성에 관한 Microsoft의 약속을 설명합니다. 특정 서비스의 SLA가 99.9%이면, 서비스가 시간 중 99.9% 동안 이용할 수 있다고 기대할 수 있습니다.

> [!참고]
> T또한 Azure SLA에는 SLA가 충족되지 않았을 경우에 서비스 크레딧을 받는 조항, 그리고 각 서비스에 대한 "가용성"의 구체적 정의도 포함되어 있습니다. SLA의 그러한 측면이 적용 정책의 역할을 수행합니다.
> 
> 

여러분은 솔루션 내의 각 작업에 대해 자체적인 목표 SLA를 정의해야 합니다. SLA가 있으면 아키텍처에 관한 추론이 가능하고 아키텍처가 비즈니스 요구 사항을 충족하는지 여부를 판단할 수 있습니다. 예를 들어 작업이 99.99%의 가동 시간을 요구하지만 99.99% SLA의 서비스에 의존한다면, 그 서비스는 시스템에서 단일 실패 지점이 될 수 없습니다. 한 가지 해결책은 서비스 장애가 발생할 경우에 대비하여 대체 경로를 확보하거나 그 서비스의 장애로부터 복구하는 다른 조치를 취하는 것입니다.

아래 표는 다양한 SLA 수준별로 예상되는 누적 가동 중지 시간을 표시하고 있습니다.

| SLA | 주간 가동 중지 시간 | 월간 가동 중지 시간 | 연간 가동 중지 시간 |
| --- | --- | --- | --- |
| 99% |1.68 시간 |7.2 시간 |3.65 일 |
| 99.9% |10.1 분 |43.2 분 |8.76 시간 |
| 99.95% |5 분 |21.6 분 |4.38 시간 |
| 99.99% |1.01 분 |4.32 분 |52.56 분 |
| 99.999% |6 초 |25.9 초 |5.26 분 |

물론 모든 조건이 동일하다면 고가용성이 더 좋습니다. 하지만 더 많은 9를 확보하려면 그 수준의 가용성을 달성하기 위한 비용과 복잡성도 증가합니다. 99.99%의 가동 시간은 월간 총 가동 중지 시간이 약 5분임을 의미합니다. 다섯 개의 9를 달성하는 데 소요되는 추가적 복잡성과 비용을 투입할 가치가 있습니까? 그 답변은 비즈니스 요구 사항에 달려있습니다.

SLA를 정의할 때 고려해야 할 기타 몇 가지 고려사항이 있습니다.

* To achieve four 9's (99.99%), you probably can't rely on manual intervention to recover from failures. The application must be self-diagnosing and self-healing. 
* Beyond four 9's, it is challenging to detect outages quickly enough to meet the SLA.
* Think about the time window that your SLA is measured against. The smaller the window, the tighter the tolerances. It probably doesn't make sense to define your SLA in terms of hourly or daily uptime. 

### Composite SLAs
Consider an App Service web app that writes to Azure SQL Database. At the time of this writing, these Azure services have the following SLAs:

* App Service Web Apps = 99.95%
* SQL Database = 99.99%

![Composite SLA](./images/sla1.png)

What is the maximum downtime you would expect for this application? If either service fails, the whole application fails. In general, the probability of each service failing is independent, so the composite SLA for this application is 99.95% x 99.99% = 99.94%. That's lower than the individual SLAs, which isn't surprising, because an application that relies on multiple services has more potential failure points. 

On the other hand, you can improve the composite SLA by creating independent fallback paths. For example, if SQL Database is unavailable, put transactions into a queue, to be processed later.

![Composite SLA](./images/sla2.png)

With this design, the application is still available even if it can't connect to the database. However, it fails if the database and the queue both fail at the same time. The expected percentage of time for a simultaneous failure is 0.0001 × 0.001, so the composite SLA for this combined path is  

* Database OR queue = 1.0 &minus; (0.0001 &times; 0.001) = 99.99999%

The total composite SLA is:

* Web app AND (database OR queue) = 99.95% &times; 99.99999% = ~99.95%

But there are tradeoffs to this approach. The application logic is more complex, you are paying for the queue, and there may be data consistency issues to consider.

**SLA for multi-region deployments**. Another HA technique is to deploy the application in more than one region, and use Azure Traffic Manager to fail over if the application fails in one region. For a two-region deployment, the composite SLA is calculated as follows. 

Let *N* be the composite SLA for the application deployed in one region. The expected chance that the application will fail in both regions at the same time is (1 &minus; N) &times; (1 &minus; N). Therefore,

* Combined SLA for both regions = 1 &minus; (1 &minus; N)(1 &minus; N) = N + (1 &minus; N)N

Finally, you must factor in the [SLA for Traffic Manager][tm-sla]. As of when this article was written, the SLA for Traffic Manager SLA is 99.99%.

* Composite SLA = 99.99% &times; (combined SLA for both regions)

A further detail is that failing over is not instantaneous, which can result in some downtime during a failover. See [Traffic Manager endpoint monitoring and failover][tm-failover].

The calculated SLA number is a useful baseline, but it doesn't tell the whole story about availability. Often, an application can degrade gracefully when a non-critical path fails. Consider an application that shows a catalog of books. If the application can't retrieve the thumbnail image for the cover, it might show a placeholder image. In that case, failing to get the image does not reduce the application's uptime, although it affects the user experience.  

## Designing for resiliency
During the design phase, you should perform a failure mode analysis (FMA). The goal of an FMA is to identify possible points of failure, and define how the application will respond to those failures.

* How will the application detect this type of failure?
* How will the application respond to this type of failure?
* How will you log and monitor this type of failure? 

For more information about the FMA process, with specific recommendations for Azure, see [Azure resiliency guidance: Failure mode analysis][fma].

### Example of identifying failure modes and detection strategy
**Failure point:** Call to an external web service / API.

| Failure mode | Detection strategy |
| --- | --- |
| Service is unavailable |HTTP 5xx |
| Throttling |HTTP 429 (Too Many Requests) |
| Authentication |HTTP 401 (Unauthorized) |
| Slow response |Request times out |

## Resiliency strategies
This section provides a survey of some common resiliency strategies. Most of these are not limited to a particular technology. The descriptions in this section are meant to summarize the general idea behind each technique, with links to further reading.

### Retry transient failures
Transient failures can be caused by momentary loss of network connectivity, a dropped database connection, or a timeout when a service is busy. Often, a transient failure can be resolved simply by retrying the request. For many Azure services, the client SDK implements automatic retries, in a way that is transparent to the caller; see [Retry service specific guidance][retry-service-specific guidance].

Each retry attempt adds to the total latency. Also, too many failed requests can cause a bottleneck, as pending requests accumulate in the queue. These blocked requests might hold critical system resources such as memory, threads, database connections, and so on, which can cause cascading failures. To avoid this, increase the delay between each retry attempt, and limit the total number of failed requests.

![Composite SLA](./images/retry.png)

For more information, see [Retry Pattern][retry-pattern].

### Load balance across instances
For scalability, a cloud application should be able to scale out by adding more instances. This approach also improves resiliency, because unhealthy instances can be taken out of rotation.  

For example:

* Put two or more VMs behind a load balancer. The load balancer distributes traffic to all the VMs. See [Running multiple VMs on Azure for scalability and availability][ra-multi-vm].
* Scale out an Azure App Service app to multiple instances. App Service automatically load balances across instances. See [Basic web application][ra-basic-web].
* Use [Azure Traffic Manager][tm] to distribute traffic across a set of endpoints.

### Replicate data
Replicating data is a general strategy for handling non-transient failures in a data store. Many storage technologies provide built-in replication, including Azure SQL Database, DocumentDB, and Apache Cassandra.  

It's important consider both the read and write paths. Depending on the storage technology, you might have multiple writable replicas, or a single writable replica and multiple read-only replicas. 

For highest availability, replicas can be placed in multiple regions. However, this increases the latency to replicate the data. Typically, replicating across regions is done asynchronously, which implies an eventual consistency model and potential data loss if a replica fails. 

### Degrade gracefully
If a service fails and there is no failover path, the application may be able to degrade gracefully, in a way that still provides an acceptable user experience. For example:

* Put a work item on a queue, to be executed later. 
* Return an estimated value 
* Use locally cached data. 
* Show the user an error message. (This option is better than having the application stop responding to requests.)

### Throttle high-volume users
Sometimes a small number of users create excessive load. That can have an impact on other users, reducing the overall availability of your application.

When a single client makes an excessive number of requests, the application might throttle the client for a certain period of time. During the throttling period, the application refuses some or all of the requests from that client (depending on the exact throttling strategy). The threshold for throttling might depend on the customer's service tier. 

Throttling does not imply the client was necessarily acting maliciously. It just means the client exceeded their service quota.  In some cases, a consumer might consistently exceed their quota or otherwise behave badly. In that case, you might go further and block the user. Typically, this is done by blocking an API key or an IP address range.

For more information, see [Throttling Pattern][throttling-pattern].

### Use a circuit breaker
The Circuit Breaker pattern can prevent an application from repeatedly trying an operation that is likely to fail. The analogy is to a physical circuit breaker, a switch that interrupts the flow of current when a circuit is overloaded.

The circuit breaker wraps calls to a service. It has three states:

* **Closed**. This is the normal state. The circuit breaker sends requests to the service, and a counter tracks the number of recent failures. If the failure count exceeds a threshold within a given time period, the circuit breaker switches to the Open state. 
* **Open**. In this state, the circuit breaker immediately fails all requests, without calling the service. The application should use a mitigation path, such as reading data from a replica or simply returning an error to the user. When the circuit breaker switches to Open, it starts a timer. When the timer expires, the circuit breaker switches to the Half-open state.
* **Half-open**. In this state, the circuit breaker lets a limited number of requests go through to the service. If they succeed, the service is assumed to be recovered, and the circuit breaker switches back to the Closed state. Otherwise, it reverts to the Open state. The Half-Open state prevents a recovering service from suddenly being inundated with requests.

For more information, see [Circuit Breaker Pattern][circuit-breaker-pattern].

### Use load leveling to smooth out spikes in traffic
Applications may experience sudden spikes in traffic, which can overwhelm services on the backend. If a backend service cannot respond to requests quickly enough, it may cause requests to queue (back up), or cause the service to throttle the application.

To avoid this, you can use a queue as a buffer. When there is a new work item, instead of calling the backend service immediately, the application queues a work item to run asynchronously. The queue acts as a buffer that smooths out peaks in the load. 

For more information, see [Queue-Based Load Leveling Pattern][load-leveling-pattern].

### Isolate critical resources
Failures in one subsystem can sometimes cascade, causing failures in other parts of the application. This can happen if a failure causes some resources, such as threads or sockets, not to get freed in a timely manner, leading to resource exhaustion. 

To avoid this, you can partition a system into isolated groups, so that a failure in one partition does not bring down the entire system. This technique is sometimes called the Bulkhead pattern.

Examples:

* Partition a database -- for example, by tenant -- and assign a separate pool of web server instances for each partition.  
* Use separate thread pools to isolate calls to different services. This helps to prevent cascading failures if one of the services fails. For an example, see the Netflix [Hystrix library][hystrix].
* Use [containers][containers] to limit the resources available to a particular subsystem. 

![Composite SLA](./images/bulkhead.png)

### Apply compensating transactions
A compensating transaction is a transaction that undoes the effects of another completed transaction.

In a distributed system, it can be very difficult to achieve strong transactional consistency. Compensating transactions are a way to achieve consistency by using a series of smaller, individual transactions that can be undone at each step.

For example, to book a trip, a customer might reserve a car, a hotel room, and a flight. If any of these steps fails, the entire operation fails. Instead of trying to use a single distributed transaction for the entire operation, you can define a compensating transaction for each step. For example, to undo a car reservation, you cancel the reservation. In order to complete the whole operation, a coordinator executes each step. If any step fails, the coordinator applies compensating transactions to undo any steps that were completed. 

For more information, see [Compensating Transaction Pattern][compensating-transaction-pattern]. 

## Testing for resiliency
Generally, you can't test resiliency in the same way that you test application functionality (by running unit tests and so on). Instead, you must test how the end-to-end workload performs under failure conditions, which by definition don't happen all of the time.

Testing is part of an iterative process. Test the application, measure the outcome, analyze and fix any failures that result, and repeat the process.

**Fault injection testing**. Test the resiliency of the system to failures, either by triggering actual failures or by simulating them. Here are some common failure scenarios to test:

* Shut down VM instances.
* Crash processes.
* Expire certificates.
* Change access keys.
* Shut down the DNS service on domain controllers.
* Limit available system resources, such as RAM or number of threads.
* Unmount disks.
* Redeploy a VM.

Measure the recovery times and verify they meet your business requirements. Test combinations of failure modes, as well. Make sure that failures don't cascade, and are handled in an isolated way.

This is another reason why it's important to analyze possible failure points during the design phase. The results of that analysis should be inputs into your test plan.

**Load testing**. Load test the application using a tool such as [Visual Studio Team Services][vsts] or [Apache JMeter][jmeter] Load testing is crucial for identifying failures that only happen under load, such as the backend database being overwhelmed or service throttling. Test for peak load, using production data, or synthetic data that is as close to production data as possible. The goal is to see how the application behaves under real-world conditions.   

## Resilient deployment
Once an application is deployed to production, updates are a possible source of errors. In the worst case, a bad update can cause downtime. To avoid this, the deployment process must be predictable and repeatable. Deployment includes provisioning Azure resources, deploying application code, and applying configuration settings. An update may involve all three, or a subset. 

The crucial point is that manual deployments are prone to error. Therefore, it's recommended to have an automated, idempotent process that you can run on demand, and re-run if something fails. 

* Use Resource Manager templates to automate provisioning of Azure resources.
* Use [Azure Automation Desired State Configuration][dsc] (DSC) to configure VMs.
* Use an automated deployment process for application code.

Two concepts related to resilient deployment are *infrastructure as code* and *immutable infrastructure*.

* **Infrastructure as code** is the practice of using code to provision and configure infrastructure. Infrastructure as code may use a declarative approach or an imperative approach (or a combination of both). Resource Manager templates are an example of a declarative approach. PowerShell scripts are an example of an imperative approach.
* **Immutable infrastructure** is the principle that you shouldn’t modify infrastructure after it’s deployed to production. Otherwise, you can get into a state where ad hoc changes have been applied, so it’s hard to know exactly what changed, and hard to reason about the system. 

Another question is how to roll out an application update. We recommend techniques such as blue-green deployment or canary releases, which push updates in highly controlled way to minimize possible impacts from a bad deployment.

* [Blue-green deployment][blue-green] is a technique where you deploy an update into a separate production environment from the live application. After you validate the deployment, switch the traffic routing to the updated version. For example, Azure App Service Web Apps enables this with [staging slots][staging-slots]. 
* [Canary releases][canary-release] are similar to blue-green deployment. Instead of switching all traffic to the updated version, you roll out the update to a small percentage of users, by routing a portion of the traffic to the new deployment. If there is a problem, back off and revert to the old deployment. Otherwise, route more traffic to the new version, until it gets 100% of traffic.

Whatever approach you take, make sure that you can roll back to the last-known good-deployment, in case the new version is not functioning. Also, if errors occur, it must be possible to tell from the application logs which version caused the error. 

## Monitoring and diagnostics
Monitoring and diagnostics are crucial for resiliency. If something fails, you need to know that it failed, and you need insights into the cause of the failure. 

Monitoring a large-scale distributed system poses a significant challenge. Think about an application that runs on a few dozen VMs -- it's not practical to log into each VM, one at a time, and look through log files, trying to troubleshoot a problem. Moreover, the number of VM instances is probably not static. VMs get added and removed as the application scales in and out, and occasionally an instance may fail and need to be reprovisioned. In addition, a typical cloud application might use multiple data stores (Azure storage, SQL Database, DocumentDB, Redis cache), and a single user action may span multiple subsystems. 

You can think of the monitoring and diagnostics process as a pipeline with several distinct stages:

![Composite SLA](./images/monitoring.png)

* **Instrumentation**. The raw data for monitoring and diagnostics comes from a variety of sources, including application logs, web server logs, OS performance counters, database logs, and diagnostics built into the Azure platform. Most Azure services have a diagnostics feature that you can use to figure out the cause of problems.
* **Collection and storage**. The raw instrumentation data can be held in a variety of locations and with varying formats (application trace logs, performace counters, IIS logs). These disparate sources are collected, consolidated, and put into reliable storage.
* **Analysis and diagnosis**. After the data is consolidated, it can be analyzed, in order to troubleshoot issues and provide an overall view of the health of the application.
* **Visualization and alerts**. In this stage, telemetry data is presented in such a way that an operator can quickly spot trends or problems. Example include dashboards or email alerts.  

Monitoring is different than failure detection. For example, your application might detect a transient error and retry, resulting in no downtime. But it should also log the retry operation, so that you can monitor the error rate, in order to get an overall picture of the application health. 

Application logs are an important source of diagnostics data. Here are some best practices for application logging:

* Log in production. Otherwise, you lose insight at the very times when you need it the most.
* Log events at service boundaries. Include a correlation ID that flows across service boundaries. If transaction X flows through multiple services and one of them fails, the correlation ID will help you pinpoint why the transaction failed.
* Use semantic logging, also called structured logging. Unstructured logs make it hard to automate the consumption and analysis of the log data, which is needed at cloud scale.
* Use asynchronous logging. Otherwise, the logging system itself can cause the application to fail, by causing requests to back up, as they block waiting to write a logging event.
* Application logging is not the same as auditing. Auditing may be done for compliance or regulatory reasons. As such, audit records must be complete, and it's not acceptible to drop any while processing transactions. If an application requires auditing, this should be kept separate from diagnostics logging. 

For more information about monitoring and diagnostics, see [Monitoring and diagnostics guidance][monitoring-guidance].

## Manual failure responses
Previous sections have focused on automated recovery strategies, which are critical for high availability. However, sometimes manual intervention is needed.

* **Alerts**. Monitor your application for warning signs that may require pro-active intervention. For example, if you see that SQL Database or DocumentDB consistently throttles your application, you might need to increase your database capacity or optimize your queries. In this example, even though the application might handle the throttling errors transparently, your telemetry should still raise an alert, so that you can follow up.  
* **Manual failover**. Some systems cannot fail over automatically, and require a manual failover. 
* **Operational readiness testing**. If your application fails over to a secondary region, you should perform an operational readiness test before you fail back to the primary region. The test should verify that the primary region is healthy and ready to receive traffic again.
* **Data consistency check**. If a failure happens in a data store, there may be data inconsistencies when the store becomes available again, especially if the data was replicated. 
* **Restoring from backup**. For example, if SQL Database experiences a regional outage, you can geo-restore the database from the latest backup.

Document and test your disaster recovery plan. Include written procedures for any manual steps, such as manual failover, restoring data from backups, and so forth. 

## Summary
This article looked at resiliency from a holistic perspective, emphasizing some of the unique challenges of the cloud. These include the distributed nature of cloud computing, the use of commodity hardware, and the presence of transience network faults.

Here are the major points to take away from this article:

* Resiliency leads to higher availability, and lower mean time to recover from failures. 
* Achieving resiliency in the cloud requires a different set of techniques from traditional on-premises solutions. 
* Resiliency does not happen by accident. It must be designed and built in from the start.
* Resiliency touches every part of the application lifecycle, from planning and coding to operations.
* Test and monitor!


<!-- links -->

[blue-green]: http://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]: http://martinfowler.com/bliki/CanaryRelease.html
[circuit-breaker-pattern]: https://msdn.microsoft.com/library/dn589784.aspx
[compensating-transaction-pattern]: https://msdn.microsoft.com/library/dn589804.aspx
[containers]: https://en.wikipedia.org/wiki/Operating-system-level_virtualization
[dsc]: https://azure.microsoft.com/documentation/articles/automation-dsc-overview/
[fma]: failure-mode-analysis.md
[hystrix]: http://techblog.netflix.com/2012/11/hystrix.html
[jmeter]: http://jmeter.apache.org/
[load-leveling-pattern]: https://msdn.microsoft.com/library/dn589783.aspx
[monitoring-guidance]: ../best-practices/monitoring.md
[ra-basic-web]: https://azure.microsoft.com/documentation/articles/web-apps-basic/
[ra-multi-vm]: https://azure.microsoft.com/documentation/articles/compute-multi-vm/
[checklist]: ../checklist/resiliency.md
[retry-pattern]: https://msdn.microsoft.com/library/dn589788.aspx
[retry-service-specific guidance]: ../best-practices/retry-service-specific.md
[sla]: https://azure.microsoft.com/support/legal/sla/
[staging-slots]: https://azure.microsoft.com/documentation/articles/web-apps-basic/
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[tm-failover]: https://azure.microsoft.com/documentation/articles/traffic-manager-monitoring/
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[vsts]: https://www.visualstudio.com/features/vso-cloud-load-testing-vs.aspx
