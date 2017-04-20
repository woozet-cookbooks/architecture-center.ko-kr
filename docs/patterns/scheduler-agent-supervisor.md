---
title: Scheduler Agent Supervisor
description: Coordinate a set of actions across a distributed set of services and other remote resources.
keywords: design pattern
author: dragon119
ms.service: guidance
ms.topic: article
ms.author: pnp
ms.date: 03/24/2017

pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: [messaging, resiliency]
---

# 스케줄러 에이전트 감독자

[!INCLUDE [header](../_includes/header.md)]

분산된 작업들을 단일 작업으로 조정합니다. 작업이 하나라도 실패할 경우, 실패를 투명하게 처리하거나 아니면 수행된 작업을 원 상태로 돌려놓습니다. 그렇게 함으로써 전체 작업이 전반적으로 성공하거나 실패합니다. 또 일시적 예외로 인해 실패한 작업, 지속적 오류, 프로세스 오류를 복구하고 재시도할 수 있게 하여 분산 시스템에 복원력을 추가할 수 있습니다.

## 컨텍스트와 문제점

응용 프로그램은 많은 단계를 포함하는 태스크를 수행하는데, 일부 업무는 원격 서비스를 호출하거나 원격 리소스를 액세스할 수 있습니다. 개별 단계는 서로서로 의존적일 수 있으나, 태스크를 구현한 응용 프로그램 로직에 의해 조정됩니다. 

가능한 한 응용 프로그램은 태스크가 완료될 때까지 실행되는 것을 확인하고. 원격 서버나 리소스를 액세스할 때 일어날 수 있는 오류를 해결해야 합니다. 오류는 여러 가지 이유로 발생할 수 있습니다. 예를 들면, 네트워크가 다운되거나 통신이 중단될 수 있고, 원격 서비스가 응답하지 않거나 불안정한 상태일 수 있으며, 자원 제약 등의 이유로 원격 리소스가 일시적으로 액세스 불가일 수 있습니다. 많은 경우에 오류는 일시적이기 때문에 [패턴 재시도][retry-pattern]를 사용하여 처리할 수 있습니다. 

응용 프로그램이 쉽게 복구할 수 없는 보다 영구적인 오류를 감지한 경우, 시스템을 일관성 있는 상태로 복원해서 전체 작업의 무결성을 보장할 수 있어야 합니다.

## 솔루션

스케줄러 에이전트 감독자 패턴은 다음과 같은 배우들을 정의합니다. 이 배우들은 전반적인 태스크의 일부로서 수행될 단계들을 조정합니다.

- **스케줄러** 는 실행될 태스크를 구성하는 단계들을 정리하고 그 작업을 조정합니다. 이 단계들은 파이프라인이나 워크플로와 결합될 수 있습니다. 스케줄러는 이 워크플로의 단계들이 올바른 순서로 수행될 수 있게 합니다. 각 단계가 수행될 때, 스케줄러는 "시작하지 않은 단계", "실행 단계" 또는 "완료 단계"와 같은 워크플로 상태를 기록합니다. 상태 정보는 단계 종료에 허용된 상한 시간도 포함해야 하는데, 이를 완료기한(complete-by time)이라고 합니다. 단계가 원격 서비스 또는 리소스에 대한 액세스를 요구할 경우, 스케줄러는 적절한 에이전트를 호출해서 수행될 작업의 세부 내용을 전달합니다.  스케줄러는 일반적으로 비동기 요청/응답 메시징을 사용하여 에이전트와 통신합니다.  스케줄러는 다른 분산 메시징 기술을 대신 사용할 수 있지만, 큐를 사용하여 구현할 수 있습니다.

    > 스케줄러는 [프로세스 관리자 패턴](http://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html)에서 프로세스 관리자와 비슷한 기능을 수행합니다. 실제 워크플로는 일반적으로 스케줄러에 의해 제어되는 워크플로 엔진에 의해 정의되고 구현됩니다. 이 접근 방식은 워크플로의 비즈니스 로직을 스케줄러에서 분리합니다.

- **에이전트** 는 태스크의 어떤 단계에서 찹조하는 원격 서비스 호출이나 원격 리소스 액세스를 캡슐화한 로직을 포함합니다. 각 에이전트는 일반적으로 단일 서비스 또는 리소스 호출을 포함하고, 적절한 오류 처리와 재시도 로직을 구현합니다(시간초과 제약을 조건으로 함, 추후 설명). 여러 단계들 사이에서 스케줄러에 의해 실행되는 워크플로의 단계들이 몇몇 서비스와 리소스를 사용할 경우, 각 단계는 다른 에이전트를 참조할 수 있습니다(이것은 패턴의 세부 구현사항입니다).

- **감독자** 는 스케줄러에 의해 수행되는 태스크에서 단계들의 상태를 모니터합니다. 모니터링은 주기적으로(주기는 시스템 고유의 값입니다) 실행되고 스케줄러에 의해 유지 관리되는 단계의 상태를 검사합니다. 시간초과나 실패를 감지할 경우, 적절한 에이전트를 배치하여 그 단계를 복구하거나 적절한 수정 조치를 실행합니다(이 조치는 단계의 상태 변경을 포함할 수 있음). 복구 또는 수정 조치는 스케줄러와 에이전트에 의해 구현되는 점에 주목하세요. 감독자는 단순히 이 조치를 수행할 것을 요청해야 합니다.

스케줄러, 에이전트, 감독자는 논리적 구성요소이고 그 물리적 구현은 사용되는 기술에 의존합니다. 예를 들면, 몇몇 논리적 에이전트는 단일 웹 서비스의 일부분으로 구현될 수 있습니다.

스케줄러는 상태 저장소라고 하는 지속형 데이터 저장소에서 태스크 진행과 모든 단계의 상태에 관한 정보를 유지 관리합니다. 감독자는 단계가 실패했는지 판단하는 데 도움을 주고자 이 정보를 사용할 수 있습니다. 그림은 스케줄러, 에이전트, 감독자, 상태 저장소 간의 관계를 나타냅니다.

![Figure 1 - The actors in the Scheduler Agent Supervisor pattern](./_images/scheduler-agent-supervisor-pattern.png)


> 이 다이어그램은 패턴을 단순화해서 보여줍니다. 실제 구현에서는 동시에 수행되는 스케줄러의 인스턴스들과 하위 태스크들이 많이 있을 수 있습니다. 마찬가지로 시스템은 각 에이전트의 다중 인스턴스, 심지어 다중 감독자를 실행할 수 있습니다. 이 경우, 감독자들은 실패한 동일한 단계와 태스크를 복구하기 위해 경쟁하지 않도록 서로 주의해서 작업을 조정해야 합니다. [리더 선거 패턴](leader-election.md) 은 이런 문제에 가능한 해결 방법을 제시합니다. 

응용 프로그램이 태스크를 수행할 준비가 되면, 스케줄러에 요청을 제출합니다. 스케줄러는 태스크와 그 단계의 초기 상태 정보를 상태 저장소에 기록한 다음(예: 아직 시작하지 않은 단계), 워크플로에 의해 정의된 작업을 수행하기 시작합니다. 스케줄러는 각 단계를 시작하는 동안, 그 단계의 상태에 관한 정보를 상태 저장소에 업데이트합니다(예: 단계 실행).

단계가 원격 서비스나 리소스를 참조할 경우, 스케줄러는 적절한 에이전트에 메시지를 전송합니다. 메시지에는 작업의 완료기한뿐만 아니라 에이전트가 서비스에 전달하거나 리소스를 액세스할 필요가 있는 정보가 담깁니다. 에이전트가 작업을 완료하면 스케줄러에 응답을 반환합니다. 스케줄러는 상태 정보를 상태 저장소에 업데이트할 수 있습니다(예: 단계 완료). 전체 태스크가 완료될 때까지 이 프로세스가 계속됩니다.

에이전트는 작업 수행에 필요한 재시도 로직을 구현할 수 있습니다. 그러나, 에이전트가 완료기한 안에 작업을 완료하지 않은 경우, 스케줄러는 작업이 실패했다고 간주할 것입니다. 이 경우, 에이전트는 작업을 중단하고 스케줄러에 어떤 것도 반환하려 하거나(오류 메시지라고 해도) 어떤 형태의 복구도 시도하면 안 됩니다.  이런 제한의 이유는, 단계가 시간초과되거나 실패한 후, 에이전트의 또 다른 인스턴스가 실패한 그 단계를 실행하도록 계획되었을 수 있기 때문입니다(이 과정은 추후 설명).

에이전트가 실패한 경우, 스케줄러는 응답을 받지 않을 것입니다. 패턴은 시간초과된 단계와 진짜 실패한 단계를 구분하지 않습니다.

단계가 시간초과되거나 실패한 경우, 상태 저장소는 그 단계가 실행 중임을 나타내는 레코드를 포함하겠지만, 완료기한은 지나가버릴 것입니다.  감독자는 이와 같은 단계를 찾아서 복구하고자 합니다. 한 가지 가능한 전략은 감독자가 완료기한 값을 업데이트해서 단계를 완료하는 데 사용할 수 있는 시간을 연장하는 것입니다. 그런 다음 시간초과된 단계를 확인하는 스케줄러에 메시지를 전송합니다. 스케줄러는 이 단계의 반복을 시도할 수 있습니다. 그러나, 이 설계는 태스크가 멱등원(idempotent) 되는 것을 요구합니다.

감독자는 단계가 계속 실패하거나 시간초과될 경우, 같은 단계를 재시도하지 않게 해야할 수 있습니다. 이를 위해서 감독자는 상태 정보와 함께 각 단계의 재시도 횟수를 상태 저장소에서 유지 관리할 수 있습니다. 이 횟수가 미리 정의된 임계값을 초과할 경우, 감독자는 단계를 재시도해야 한다고 스케줄러에 알리기 전에, 연장된 기간에 오류가 해결되기를 기대하여 이 기간 동안 기다리는 전략을 채택할 수 있습니다. 또는, 감독자가 [보상 트랜잭션 패턴](compensating-transaction.md)을 구현하여 전체 태스크가 처리되지 않게 요청하는 메시지를 스케줄러에 보낼 수 있습니다. 이 접근 방식은, 성공적으로 완료된 단계에 대한 보상 작업 구현에 필요한 정보를 제공하는 스케줄러와 에이전트에 의존할 것입니다.

> 스케줄러와 에이전트를 모니터하고 이들이 실패할 때 재시동하는 것이 감독자의 목적이 아닙니다. 시스템의 이러한 측면은 이런 구성요소들이 실행되는 인프라에 의해 처리되어야 합니다. 마찬가지로, 감독자는 스케줄러에 의해 수행되는 태스크가 실행되는 실제 비즈니스 작업을 몰라야 합니다(이 태스크 실패를 보상하는 방법을 포함하여).   이것이 스케줄러에 의해 구현된 워크플로 로직의 목적입니다. 감독자의 유일한 책임은 단계가 실패했는지 여부를 판단하고, 그 단계를 반복할지 아니면 실패한 단계를 포함한 전체 태스크를 취소할지를 결정하는 것입니다.

실패 후 스케줄러가 재시동되거나 스케줄러에 의해 수행되는 워크플로가 예상치 않게 종료된 경우, 스케줄러는 작업이 실패할 때 처리 중이었던 인플라이트(inflight) 태스크 상태를 파악하고, 그 지점부터 이 태스크를 재개할 준비를 해야 합니다. 이 프로세스의 세부 구현 내용은 대체로 시스템별로 고유합니다. 태스크가 복구될 수 없는 경우, 태스크에 의해 이미 수행된 작업을 취소해야 할 수 있습니다. 이 때 [보상 트랜잭션](compensating-transaction.md) 구현을 요구할 수 있습니다.

이 패턴의 중요한 장점은 예상치 않게 일시적이거나 복구 불가능한 오류가 생겼을 때 시스템이 복원된다는 점입니다. 시스템은 자동 복구되도록 생성될 수 있습니다. 예를 들면, 에이전트나 스케줄러가 실패할 경우, 새 에이전트나 스케줄러가 시작할 수 있고 감독자는 재개될 태스크에 해당되도록 조정할 수 있습니다. 감독자가 실패한 경우, 새로운 인스턴스가 시작되어 실패가 발생한 곳을 이어받을 수 있습니다. 감독자가 주기적으로 수행하도록 계획한 경우, 미리 예정된 간격이 지나면 새 인스턴스가 자동으로 시작될 수 있습니다. 훨씬 큰 규모로 복원하기 위해서 상태 저장소가 복제될 수 있습니다.

## 문제점 및 고려사항

이 패턴을 구현하는 방법을 결정할 때 다음 사항을 고려해야 합니다.

- 이 패턴은 구현하기 어려울 수 있으며, 시스템에 생길 수 있는 오류 모드에 대한 철저한 시험이 필요합니다.

- 스케줄러에 의해 구현된 복구/재시도 로직은 복잡하고 상태 저장소에 저장된 상태 정보에 의존합니다.  또한 보상 트랜잭션을 구현하는 데 요구되는 정보를 지속형 데이터 저장소에 기록해야 할 수 있습니다.

- 감독자가 얼마나 자주 실행될 것인지도 중요합니다. 실패한 단계가 연장된 기간에 응용 프로그램을 차단할 수 없을 만큼 충분히 자주 실행되어야 하지만, 오버헤드가 될 정도로 자주 실행되어서는 안 됩니다.

- 에이전트에 의해 수행되는 단계는 한 번 이상 실행될 수 있습니다. 이 단계를 구현하는 로직은 멱등원이어야 합니다.

## 이 패턴을 사용하는 때

클라우드와 같은 분산 환경에서 실행하는 프로세스가 통신 실패 및/또는 운영상 실패로 인하여 복원되어야 할 때, 이 패턴을 사용합니다.

이 패턴은 원격 서비스를 호출하거나 원격 리소스를 액세스하지 않는 태스크에 적절하지 않을 수 있습니다.

## 예

Microsoft Azure에 전자 상거래 시스템을 구현하는 웹 응용 프로그램이 배포되었습니다. 사용자는 사용 가능한 제품을 찾거나 주문하기 위해서 이 응용 프로그램을 실행할 수 있습니다. 사용자 인터페이스가 웹 역할로 실행되고, 응용 프로그램의 주문 처리 요소가 작업자 역할로 구현됩니다. 주문 처리 로직의 일부는 원격 서비스 액세스를 포함하는데, 시스템의 이런 측면은 일시적이거나 더 오래 지속되는 오류일 가능성이 있습니다. 이런 이유로, 설계자는 시스템의 주문 저리 요소를 구현하기 위해서 스케줄러 에이전트 감독자 패턴을 사용했습니다.

고객이 주문할 때, 응용 프로그램은 주문 내용을 나타내는 메시지를 구성하고 이 메시지를 큐에 게시합니다. 작업자 역할에서 실행되는 별도의 전송 프로세스는 메시지를 검색하고, 세부 주문 내용을 주문 데이터베이스에 삽입하고, 상태 저장소에 주문 프로세스에 대한 레코드를 만듭니다. 주문 데이터베이스와 상태 저장소로의 삽입은 동일 작업의 일환으로 수행된다는 사실에 주의하세요. 제출 프로세스는 두 가지 입력이 동시에 완료되도록 설계됩니다.

전송 프로세스가 주문에 대해서 생성하는 상태 정보에는 다음 항목이 포함됩니다:

- **OrderID**. 주문 데이터베이스에 있는 주문 ID.

- **LockedBy**. 주문을 처리하는 작업자 역할의 인스턴스 ID. 스케줄러를 실행하는 작업자 역할의 현재 인스턴스가 다수일 수 있지만, 각 주문은 단일 인스턴스에 의해서만 처리되어야 합니다. 

- **CompleteBy**. 주문이 처리되어야 하는 기한.

- **ProcessState**. 주문을 처리하는 태스크의 현재 상태 . 가능한 상태:

    - **보류(Pending)**. 주문이 생성되었으나 처리가 아직 시작되지 않았습니다.
    - **처리 중(Processing)**. 주문이 현재 처리되고 있습니다.
    - **처리 완료(Processed)**. 주문이 성공적으로 처리되었습니다.
    - **오류(Error)**. 주문 처리가 실패했습니다.

- **FailureCount**. 주문 처리가 시도된 횟수.

이 상태 정보에서, `OrderID` 필드는 새 주문의 order ID에서 복사됩니다. `LockedBy` 와 `CompleteBy` 필드는 `null`로, `ProcessState` 필드는 `보류`로, `FailureCount` 필드는 0으로 설정됩니다.

> 이 예에서, 주문 처리 로직은 비교적 단순하고 원격 서비스를 호출하는 단일 단계로 구성됩니다. 보다 복잡한 다단계 시나리오에서, 전송 프로세스는 여러 단계를 포함할 것이고, 그에 따라 상태 저장소에 레코드가 여러 개 생성될 것입니다 - 각 레코드는 각 단계의 상태를 설명함.

스케줄러는 또 작업자 역할의 일부로 실행되고 주문을 처리하는 비즈니스 로직을 구현합니다. 새 주문을 폴링하는 스케줄러의 인스턴스는 상태 저장소에서 `LockedBy` 필드가 null이고, `ProcessState` 필드가 보류인 레코드를 검사합니다. 스케줄러가 새 주문을 찾으면, 즉시 `LockedBy` 필드를 자신의 인스턴스 ID로, `CompleteBy` 필드를 적절한 시간으로, `ProcessState` 필드를 처리 중으로 설정합니다. 코드는 스케줄러의 동시성 인스턴스 2개가 동일한 주문을 동시에 처리하는 것을 시도할 수 없게 기본적이고 배타적으로 설계되었습니다.

스케줄러는 상태 저장소에서 `OrderID` 필드 값을 넘겨받으며 주문을 비동기식으로 처리하기 위해서 비즈니스 워크플로를 실행합니다 주문을 처리하는 워크플로는 주문 데이터베이스에서 세부 주문 사항을 검색하고 그 작업을 수행합니다. 워크플로를 처리하는 주문에서 어떤 단계가 원격 서비스를 호출할 필요가 있으면, 에이전트를 사용합니다. 워크플로 단계는 요청/응답 채널로 작용하는 한 쌍의 Azure 서비스 버스 메시지 큐를 사용하여 에이전트와 통신합니다. 그림은 상위 수준의 관점에서 솔루션을 보여줍니다.

![Figure 2 - Using the Scheduler Agent Supervisor pattern to handle orders in a Azure solution](./_images/scheduler-agent-supervisor-solution.png)

The message sent to the Agent from a workflow step describes the order and includes the complete-by time. If the Agent receives a response from the remote service before the complete-by time expires, it posts a reply message on the Service Bus queue on which the workflow is listening. When the workflow step receives the valid reply message, it completes its processing and the Scheduler sets the `ProcessState field of the order state to processed. At this point, the order processing has completed successfully.

If the complete-by time expires before the Agent receives a response from the remote service, the Agent simply halts its processing and terminates handling the order. Similarly, if the workflow handling the order exceeds the complete-by time, it also terminates. In both cases, the state of the order in the state store remains set to processing, but the complete-by time indicates that the time for processing the order has passed and the process is deemed to have failed. Note that if the Agent that's accessing the remote service, or the workflow that's handling the order (or both) terminate unexpectedly, the information in the state store will again remain set to processing and eventually will have an expired complete-by value.

If the Agent detects an unrecoverable, nontransient fault while it's trying to contact the remote service, it can send an error response back to the workflow. The Scheduler can set the status of the order to error and raise an event that alerts an operator. The operator can then try to resolve the reason for the failure manually and resubmit the failed processing step.

The Supervisor periodically examines the state store looking for orders with an expired complete-by value. If the Supervisor finds a record, it increments the `FailureCount` field. If the failure count value is below a specified threshold value, the Supervisor resets the `LockedBy` field to null, updates the `CompleteBy` field with a new expiration time, and sets the `ProcessState` field to pending. An instance of the Scheduler can pick up this order and perform its processing as before. If the failure count value exceeds a specified threshold, the reason for the failure is assumed to be nontransient. The Supervisor sets the status of the order to error and raises an event that alerts an operator.

> In this example, the Supervisor is implemented in a separate worker role. You can use a variety of strategies to arrange for the Supervisor task to be run, including using the Azure Scheduler service (not to be confused with the Scheduler component in this pattern). For more information about the Azure Scheduler service, visit the [Scheduler](https://azure.microsoft.com/services/scheduler/) page.

Although it isn't shown in this example, the Scheduler might need to keep the application that submitted the order informed about the progress and status of the order. The application and the Scheduler are isolated from each other to eliminate any dependencies between them. The application has no knowledge of which instance of the Scheduler is handling the order, and the Scheduler is unaware of which specific application instance posted the order.

To allow the order status to be reported, the application could use its own private response queue. The details of this response queue would be included as part of the request sent to the submission process, which would include this information in the state store. The Scheduler would then post messages to this queue indicating the status of the order (request received, order completed, order failed, and so on). It should include the order ID in these messages so they can be correlated with the original request by the application.

## Related patterns and guidance

The following patterns and guidance might also be relevant when implementing this pattern:
- [Retry pattern][retry-pattern]. An Agent can use this pattern to transparently retry an operation that accesses a remote service or resource that has previously failed. Use when the expectation is that the cause of the failure is transient and can be corrected.
- [Circuit Breaker pattern](circuit-breaker.md). An Agent can use this pattern to handle faults that take a variable amount of time to correct when connecting to a remote service or resource.
- [Compensating Transaction pattern](compensating-transaction.md). If the workflow being performed by a Scheduler can't be completed successfully, it might be necessary to undo any work it's previously performed. The Compensating Transaction pattern describes how this can be achieved for operations that follow the eventual consistency model. These types of operations are commonly implemented by a Scheduler that performs complex business processes and workflows.
- [Asynchronous Messaging Primer](https://msdn.microsoft.com/library/dn589781.aspx). The components in the Scheduler Agent Supervisor pattern typically run decoupled from each other and communicate asynchronously. Describes some of the approaches that can be used to implement asynchronous communication based on message queues.
- [Leader Election pattern](leader-election.md). It might be necessary to coordinate the actions of multiple instances of a Supervisor to prevent them from attempting to recover the same failed process. The Leader Election pattern describes how to do this.
- [Cloud Architecture: The Scheduler-Agent-Supervisor Pattern](https://blogs.msdn.microsoft.com/clemensv/2010/09/27/cloud-architecture-the-scheduler-agent-supervisor-pattern/) on Clemens Vasters' blog
- [Process Manager pattern](http://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html)
- [Reference 6: A Saga on Sagas](https://msdn.microsoft.com/library/jj591569.aspx). An example showing how the CQRS pattern uses a process manager (part of the CQRS Journey guidance).
- [Microsoft Azure Scheduler](https://azure.microsoft.com/services/scheduler/)

[retry-pattern]: ./retry.md
