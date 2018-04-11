---
title: 우선 순위 큐
description: 우선 순위가 높은 요청을 우선 순위가 낮은 요청보다 먼저 받아서 처리하도록 서비스로 전송된 요청의 우선 순위를 지정합니다.
keywords: 디자인 패턴
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- messaging
- performance-scalability
ms.openlocfilehash: ecfbb38304bb95587e9ca15523ad9594898d9b32
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="priority-queue-pattern"></a>우선 순위 큐 패턴

[!INCLUDE [header](../_includes/header.md)]

우선 순위가 높은 요청을 우선 순위가 낮은 요청보다 먼저 받아서 처리하도록 서비스로 전송된 요청의 우선 순위를 지정합니다. 이 패턴은 개별 클라이언트에 서로 다른 서비스 수준 보장을 제공하는 응용 프로그램에서 유용합니다.

## <a name="context-and-problem"></a>컨텍스트 및 문제

응용 프로그램은 특정 작업(예: 백그라운드 처리 수행 또는 다른 응용 프로그램이나 서비스와의 통합)을 다른 서비스에 위임할 수 있습니다. 클라우드에서 메시지 큐는 일반적으로 작업을 백그라운드 처리에 위임하는 데 사용됩니다. 대부분의 경우 서비스에서 수신되는 주문 요청은 중요하지 않습니다. 그러나 경우에 따라 특정 요청에 대해 우선 순위를 지정해야 합니다. 이러한 요청은 이전에 응용 프로그램에서 보낸 우선 순위가 낮은 요청보다 먼저 처리해야 합니다.

## <a name="solution"></a>해결 방법

큐는 일반적으로 FIFO(선입 선출) 구조이며, 소비자는 대개 큐에 게시된 것과 동일한 순서로 메시지를 받습니다. 그러나 일부 메시지 큐에서는 우선 순위 메시지를 지원합니다. 메시지를 게시하는 응용 프로그램은 우선 순위를 지정할 수 있으며, 이 경우 큐의 메시지가 자동으로 다시 정렬되어 우선 순위가 높은 메시지가 우선 순위가 낮은 메시지보다 먼저 수신됩니다. 그림에서는 우선 순위 메시지가 있는 큐를 보여 줍니다.

![그림 1 - 메시지 우선 순위 지정을 지원하는 큐 메커니즘 사용](./_images/priority-queue-pattern.png)

> 대부분의 메시지 큐 구현은 [경쟁 소비자 패턴](https://msdn.microsoft.com/library/dn568101.aspx)에 따라 여러 소비자를 지원하며, 수요에 따라 소비자 프로세스의 수를 늘리거나 줄일 수 있습니다.

우선 순위 기반 메시지 큐를 지원하지 않는 시스템에서 적용할 수 있는 다른 솔루션은 각 우선 순위마다 별도의 큐를 유지하는 것입니다. 응용 프로그램은 해당 큐에 메시지를 게시해야 합니다. 각 큐에는 별도의 소비자 풀이 있을 수 있습니다. 우선 순위가 높은 큐는 우선 순위가 낮은 큐보다 더 빠른 하드웨어에서 실행되는 더 큰 소비자 풀을 가질 수 있습니다. 다음 그림에서는 각 우선 순위마다 별도의 메시지 큐를 사용하는 방법을 보여 줍니다.

![그림 2 - 각 우선 순위마다 별도의 메시지 큐 사용](./_images/priority-queue-separate.png)


이 전략의 변형은 우선 순위가 높은 큐의 메시지를 먼저 확인한 다음, 우선 순위가 낮은 큐에서 메시지를 가져오기 시작하는 단일 소비자 풀을 갖는 것입니다. 단일 소비자 프로세스 풀을 사용하는 솔루션(우선 순위가 다른 메시지를 지원하는 단일 큐 또는 각각 단일 우선 순위의 메시지를 처리하는 큐를 여러 개 사용하는 솔루션) 및 각각 별도의 풀이 있는 큐를 여러 개 사용하는 솔루션 간에는 몇 가지 의미상의 차이가 있습니다.

단일 풀 접근 방식에서 우선 순위가 높은 메시지는 항상 우선 순위가 낮은 메시지보다 먼저 수신되고 처리됩니다. 이론적으로, 우선 순위가 매우 낮은 메시지는 지속적으로 대체될 수 있으며, 이로 인해 처리되지 않을 수도 있습니다. 다중 풀 접근 방식에서 우선 순위가 낮은 메시지는 풀의 상대적 크기와 사용 가능한 리소스에 따라 항상 우선 순위가 높은 메시지만큼 빠르게 처리되지 않습니다.

우선 순위 큐 메커니즘을 사용하면 다음과 같은 이점이 있습니다.

- 이를 통해 특정 고객 그룹에 서로 다른 수준의 서비스를 제공하는 것과 같이 가용성 또는 성능의 우선 순위를 요구한 비즈니스 요구 사항이 응용 프로그램에서 충족될 수 있습니다.

- 운영 비용을 최소화할 수 있습니다. 단일 큐 접근 방식에서는 필요에 따라 소비자의 수를 조정할 수 있습니다. 우선 순위가 높은 메시지가 먼저 처리되며(다소 느릴 수도 있음), 우선 순위가 낮은 메시지는 더 오래 지연될 수 있습니다. 각 큐에 대해 별도의 소비자 풀이 있는 여러 메시지 큐 방식을 구현한 경우, 우선 순위가 낮은 큐의 소비자 풀을 줄이거나 이러한 큐의 메시지를 수신 대기하는 모든 소비자를 중지하여 우선 순위가 매우 낮은 큐의 처리를 일시 중단 할 수도 있습니다.

- 여러 메시지 큐 방식을 사용하면 처리 요구 사항에 따라 메시지를 분할하여 응용 프로그램 성능 및 확장성을 최대화할 수 있습니다. 예를 들어, 덜 중요한 백그라운드 작업은 한가한 시간에 실행하도록 예약된 수신기에서 처리하는 대신, 중요한 작업은 즉시 실행되는 수신기에서 처리할 수 있도록 더 높은 우선 순위를 지정할 수 있습니다.

## <a name="issues-and-considerations"></a>문제 및 고려 사항

이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.

솔루션의 컨텍스트에서 우선 순위를 정의합니다. 예를 들어 높은 우선 순위는 메시지를 10초 이내에 처리해야 함을 의미할 수 있습니다. 우선 순위가 높은 항목 및 이러한 기준을 맞추기 위해 할당되어야 하는 다른 리소스를 처리하기 위한 요구 사항을 식별합니다.

우선 순위가 높은 모든 항목을 우선 순위가 낮은 항목보다 먼저 처리해야 하는지 결정합니다. 메시지가 단일 소비자 풀에서 처리되는 경우, 우선 순위가 높은 메시지를 사용할 수 있게 되면 우선 순위가 낮은 메시지를 처리하는 작업을 선점하여 일시 중단할 수 있는 메커니즘을 제공해야 합니다.

여러 큐 방식에서 각 큐의 전용 소비자 풀이 아닌 모든 큐에서 수신 대기하는 단일 소비자 프로세스 풀을 사용하는 경우, 소비자는 항상 우선 순위가 높은 큐의 메시지를 우선 순위가 낮은 큐의 메시지보다 먼저 처리하는 알고리즘을 적용해야 합니다.

우선 순위가 높고 낮은 큐의 처리 속도를 모니터링하여 이러한 큐의 메시지가 예상 속도로 처리되도록 합니다.

우선 순위가 낮은 메시지가 처리되도록 하려면 여러 개의 소비자 풀을 사용하는 여러 메시지 큐 방식을 구현해야 합니다. 또는 메시지 우선 순위 지정을 지원하는 큐에서 대기 중인 메시지의 우선 순위를 동적으로 높일 수 있습니다. 그러나 이 방식은 이 기능을 제공하는 메시지 큐에 따라 달라집니다.

각 메시지 우선 순위에 대해 별도의 큐를 사용하면 잘 정의된 우선 순위의 수가 적은 시스템에서 가장 잘 작동합니다.

메시지 우선 순위는 시스템에서 논리적으로 결정될 수 있습니다. 예를 들어 명시적으로 우선 순위가 높고 낮은 메시지를 사용하는 것이 아니라 "유료 고객" 또는 "무료 고객"으로 지정할 수도 있습니다. 비즈니스 모델에 따라 무료 고객보다 유료 고객의 메시지를 처리하는 데 더 많은 리소스를 할당할 수 있습니다.

큐에서 메시지를 확인하는 것과 관련된 금융 및 처리 비용이 있을 수 있습니다(일부 상용 메시지 시스템은 메시지를 게시하거나 검색할 때마다 그리고 메시지에 대해 큐를 쿼리할 때마다 약간의 요금을 부과합니다). 여러 큐를 확인하는 경우 이 비용이 늘어납니다.

풀에서 처리하는 큐의 길이에 따라 소비자 풀의 크기를 동적으로 조정할 수 있습니다. 자세한 내용은 [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx)을 참조하세요.

## <a name="when-to-use-this-pattern"></a>이 패턴을 사용해야 하는 경우

이 패턴은 다음과 같은 시나리오에서 유용합니다.

- 시스템에서 우선 순위가 다른 여러 작업을 처리해야 합니다.

- 다른 사용자 또는 테넌트에는 서로 다른 우선 순위로 서비스가 제공되어야 합니다.

## <a name="example"></a>예

Microsoft Azure는 기본적으로 정렬을 통해 자동 메시지 우선 순위 지정을 지원하는 큐 메커니즘을 제공하지 않습니다. 그러나 메시지 필터링을 제공하는 큐 메커니즘을 지원하는 Azure Service Bus 토픽 및 구독과 함께 대부분의 우선 순위 큐 구현에 완벽하게 사용할 수 있는 유연성 있는 다양한 기능을 제공합니다.

Azure 솔루션은 응용 프로그램에서 큐와 동일한 방식으로 메시지를 게시할 수 있는 Service Bus 토픽을 구현할 수 있습니다. 메시지에는 응용 프로그램 정의 사용자 지정 속성 형식의 메타데이터가 포함될 수 있습니다. Service Bus 구독은 토픽과 연결될 수 있으며, 이러한 구독은 해당 속성에 따라 메시지를 필터링할 수 있습니다. 응용 프로그램에서 토픽으로 메시지를 보내는 경우 해당 메시지는 소비자가 읽을 수 있는 적절한 구독으로 보내집니다. 소비자 프로세스는 메시지 큐와 동일한 의미 체계를 사용하여 구독에서 메시지를 검색할 수 있습니다(구독은 논리적 큐임). 다음 그림에서는 Azure Service Bus 토픽 및 구독으로 우선 순위 큐를 구현하고 있습니다.

![그림 3 - Azure Service Bus 항목 및 구독으로 우선 순위 큐 구현](./_images/priority-queue-service-bus.png)


위의 그림에서 응용 프로그램은 여러 메시지를 만들고 각 메시지에 있는 `Priority`라는 사용자 지정 속성에 `High` 또는 `Low` 값을 할당합니다. 응용 프로그램은 이러한 메시지를 토픽에 게시합니다. 토픽에는 `Priority` 속성을 검사하여 메시지를 필터링하는 데 관련된 두 개의 구독이 있습니다. 한 구독은 `Priority` 속성이 `High`로 설정된 메시지를 수락하고, 다른 한 구독은 `Priority` 속성이 `Low`로 설정된 메시지를 수락합니다. 소비자 풀은 각 구독에서 메시지를 읽습니다. 우선 순위가 높은 구독에는 더 큰 풀이 있으며, 이러한 소비자는 우선 순위가 낮은 풀의 소비자보다 리소스를 더 많이 사용할 수 있는 더 강력한 컴퓨터에서 실행될 수 있습니다.

이 예에서 높음 및 낮음 우선 순위의 메시지 지정에는 특별한 의미가 없습니다. 다만 각 메시지의 속성으로 지정된 레이블일 뿐이며, 메시지를 특정 구독으로 보내는 데 사용됩니다. 추가 우선 순위가 필요한 경우 추가 구독 및 소비자 프로세스 풀을 비교적 쉽게 만들어 이러한 우선 순위를 처리할 수 있습니다.

[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/priority-queue)에서 사용할 수 있는 PriorityQueue 솔루션에는 이러한 방식의 구현이 포함되어 있습니다. 이 솔루션에는 `PriorityQueue.High` 및 `PriorityQueue.Low`라는 두 개의 작업자 역할 프로젝트가 있습니다. 이러한 작업자 역할은 `OnStart` 메서드에 지정된 구독에 연결하는 기능이 포함된 `PriorityWorkerRole` 클래스에서 상속됩니다.

`PriorityQueue.High` 및 `PriorityQueue.Low` 작업자 역할은 해당 구성 설정으로 정의된 다른 구독에 연결됩니다. 관리자는 실행할 각 역할의 수를 다르게 구성할 수 있습니다. 일반적으로 `PriorityQueue.High` 작업자 역할의 인스턴스가 `PriorityQueue.Low` 작업자 역할보다 더 많습니다.

`PriorityWorkerRole` 클래스의 `Run` 메서드는 큐에서 받은 각 메시지에 대해 실행될 가상 `ProcessMessage` 메서드(`PriorityWorkerRole` 클래스에서도 정의됨)를 정렬합니다. 다음 코드에서는 `Run` 및 `ProcessMessage` 메서드를 보여 줍니다. PriorityQueue.Shared 프로젝트에 정의된 `QueueManager` 클래스는 Azure Service Bus 큐를 사용하기 위한 도우미 메서드를 제공합니다.

```csharp
public class PriorityWorkerRole : RoleEntryPoint
{
  private QueueManager queueManager;
  ...

  public override void Run()
  {
    // Start listening for messages on the subscription.
    var subscriptionName = CloudConfigurationManager.GetSetting("SubscriptionName");
    this.queueManager.ReceiveMessages(subscriptionName, this.ProcessMessage);
    ...;
  }
  ...

  protected virtual async Task ProcessMessage(BrokeredMessage message)
  {
    // Simulating processing.
    await Task.Delay(TimeSpan.FromSeconds(2));
  }
}
```
`PriorityQueue.High` 및 `PriorityQueue.Low` 작업자 역할은 모두`ProcessMessage` 메서드의 기본 기능을 재정의합니다. 아래 코드에서는 `PriorityQueue.High` 작업자 역할에 대한 `ProcessMessage` 메서드를 보여 줍니다.

```csharp
protected override async Task ProcessMessage(BrokeredMessage message)
{
  // Simulate message processing for High priority messages.
  await base.ProcessMessage(message);
  Trace.TraceInformation("High priority message processed by " +
    RoleEnvironment.CurrentRoleInstance.Id + " MessageId: " + message.MessageId);
}
```

응용 프로그램이 `PriorityQueue.High` 및 `PriorityQueue.Low` 작업자 역할에서 사용하는 구독과 연결된 토픽에 메시지를 게시하는 경우, 다음 코드 예제와 같이 `Priority` 사용자 지정 속성을 사용하여 우선 순위를 지정합니다. 이 코드(PriorityQueue.Sender 프로젝트의 `WorkerRole` 클래스에서 구현됨)는 `QueueManager` 클래스의 `SendBatchAsync` 도우미 메서드를 사용하여 메시지를 토픽에 일괄적으로 게시합니다.

```csharp
// Send a low priority batch.
var lowMessages = new List<BrokeredMessage>();

for (int i = 0; i < 10; i++)
{
  var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
  message.Properties["Priority"] = Priority.Low;
  lowMessages.Add(message);
}

this.queueManager.SendBatchAsync(lowMessages).Wait();
...

// Send a high priority batch.
var highMessages = new List<BrokeredMessage>();

for (int i = 0; i < 10; i++)
{
  var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
  message.Properties["Priority"] = Priority.High;
  highMessages.Add(message);
}

this.queueManager.SendBatchAsync(highMessages).Wait();
```

## <a name="related-patterns-and-guidance"></a>관련 패턴 및 지침

이 패턴을 구현할 때 다음 패턴 및 지침도 관련이 있을 수 있습니다.

- 이 패턴을 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/priority-queue)에서 사용할 수 있음을 보여주는 샘플.

- [비동기 메시징 입문](https://msdn.microsoft.com/library/dn589781.aspx). 요청을 처리하는 소비자 서비스에서 해당 요청을 게시한 응용 프로그램의 인스턴스에 응답을 보내야 할 수도 있습니다. 요청/응답 메시지를 구현하는 데 사용할 수 있는 전략에 대한 정보를 제공하세요.

- [경쟁 소비자 패턴](competing-consumers.md). 큐의 처리량을 늘리려면 동일한 큐에서 수신 대기하는 여러 소비자를 갖추고 작업을 병렬로 처리할 수 있습니다. 이러한 소비자는 메시지를 얻기 위해 경쟁하지만, 한 소비자만 각 메시지를 처리할 수 있어야 합니다. 이 방식을 구현할 경우의 장단점에 대한 자세한 정보를 제공하세요.

- [제한 패턴](throttling.md). 큐를 사용하여 제한을 구현할 수 있습니다. 우선 순위 메시지를 사용하여 덜 중요한 응용 프로그램의 요청보다 높은 우선 순위를 중요한 응용 프로그램 또는 고부가 가치 고객이 실행하는 응용 프로그램의 요청에 지정할 수 있습니다.

- [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx). 큐의 길이에 따라 큐를 처리하는 소비자 프로세스 풀의 크기를 조정할 수 있습니다. 이 전략은 특히 우선 순위가 높은 메시지를 처리하는 풀의 성능을 향상시키는 데 도움이 됩니다.

- [Service Bus가 있는 엔터프라이즈 통합 패턴](http://abhishekrlal.com/2013/01/11/enterprise-integration-patterns-with-service-bus-part-2/)(Abhishek Lal의 블로그)

