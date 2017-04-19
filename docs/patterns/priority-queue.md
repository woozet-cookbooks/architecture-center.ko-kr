---
title: Priority Queue
description: Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.
keywords: design pattern
author: dragon119
ms.service: guidance
ms.topic: article
ms.author: pnp
ms.date: 03/24/2017

pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: [messaging, performance-scalability]
---

# 우선 순위 큐

[!INCLUDE [header](../_includes/header.md)]

서비스로 전송하는 요청의 우선 순위를 정해, 우선도가 높은 요청은 우선도가 낮은 요청보다 더 빠르게 수신, 처리하도록 합니다. 이 패턴은 개별 클라이언트에 다양한 서비스 수준을 보장하는 응용 프로그램에 유용합니다. 

## 컨텍스트와 문제점

응용 프로그램은 예를 들어 백그라운드 처리를 수행하거나 기타 응용 프로그램 또는 서비스와 통합하기 위해 특정 작업을 다른 서비스에 위임할 수 있습니다. 클라우드에서, 메시지 큐는 일반적으로 작업을 백그라운드 처리에 위임할 때 사용합니다. 대부분의 경우, 서비스에서 요청을 수신하는 순서는 중요하지 않습니다. 그러나, 특정한 요청은 우선도를 지정해야 하는 경우도 있습니다. 이러한 요청은 응용 프로그램에서 이전에 전송한 우선도 낮은 요청보다 더 빨리 처리해야 합니다. 

## 솔루션

큐는 대개 FIFO(first-in, first-out) 구성이 되며, 소비자는 대개 큐에 게시된 것과 같은 순서대로 메시지를 받게 됩니다. 그러나, 일부 메시지 큐는 우선 메시징 기능을 지원합니다. 메시지를 게시하는 응용 프로그램은 우선 순위를 할당할 수 있으며, 큐의 메시지는 자동으로 기록되어 우선 순위가 높은 메시지가 낮은 메시지보다 먼저 수신되도록 합니다. 그림은 우선 메시징을 사용한 큐를 나타냅니다.

![Figure 1 - Using a queuing mechanism that supports message prioritization](./_images/priority-queue-pattern.png)

> 대부분의 메시지 큐 구현은 여러 소비자를 지원하며([경쟁 소비자 패턴](https://msdn.microsoft.com/library/dn568101.aspx)에 따라), 소비자 프로세스의 수는 요구에 따라 조정할 수 있습니다.

우선 기반 메시지 큐를 지원하지 않는 시스템에서 대안 솔루션은 우선도 별로 다른 큐를 유지하는 것입니다. 응용 프로그램은 적합한 큐에 메시지를 게시해야 합니다. 각 큐에 별도의 소비자 풀을 둘 수 있습니다. 우선 순위가 높은 큐는 우선 순위가 낮은 큐보다 더 큰 소비자 풀을 더 빠른 하드웨어에서 실행할 수 있습니다. 다음 그림은 각 우선 순위마다 별도의 메시지 큐를 사용하는 그림입니다.

![Figure 2 - Using separate message queues for each priority](./_images/priority-queue-separate.png)


이 전략의 차이점은 우선 순위가 높은 큐의 메시지를 먼저 확인한 다음 우선 순위가 낮은 큐에서 메시지를 가져오기 시작하는 단일 소비자 풀이 있다는 것입니다.  단일 소비자 프로세스 풀을 사용하는(우선 순위가 다양한 메시지를 지원하는 단일 큐 또는 각각 단일한 우선 순위의 메시지를 처리하는 여러 큐를 사용) 솔루션과 각 큐에 대한 별도의 풀이 있는 큐를 여러 개 사용하는 솔루션 간에는 의미 차이가 있습니다.

단일 풀 방식에서, 우선 순위가 높은 메시지는 항상 우선 순위가 낮은 메시지보다 먼저 수신 및 처리됩니다. 이론상으로 우선 순위가 낮은 메시지는 계속 대체되다가 결국 처리되지 않을 수 있습니다. 다중 풀 방식에서, 우선 순위가 낮은 메시지는 항상 처리되며 단지 우선 순위가 높은 메시지만큼 빠르게 처리되는 것은 아닙니다(풀의 상대적 크기 및 사용 가능한 리소스에 따라 좌우됨).

우선 큐 메커니즘은 다음과 같은 장점을 제공합니다.

- 응용 프로그램이 특정 사용자 그룹에 다양한 수준의 서비스를 제공하는 등 가용성 또는 성능 우선 순위 지정이 필요한 비즈니스 요구사항을 충족시킬 수 있습니다. 

- 운영 비용을 최소화할 수 있습니다. 단일 큐 방식에서, 필요한 경우 소비자 수를 축소할 수 있습니다. 우선 순위가 높은 메시지는 여기서도 먼저 처리되며(속도는 더 느릴 수 있음) 우선 순위가 낮은 메시지는 더 오래 지연될 수 있습니다. 각 큐에 대해 별도의 소비자 풀이 있는 다중 메시지 큐 방식을 구현한 경우, 우선 순위가 낮은 큐에 대해 소비자 풀을 축소하거나, 우선 순위가 아주 낮은 큐에서 메시지를 기다리는 소비자를 모두 중지시킴으로써 그러한 일부 큐의 처리를 중단할 수도 있습니다. 

- 다중 메시지 큐 방식은 처리 요구사항에 따라 메시지를 분할하여 응용 프로그램 성능과 확장성을 최대화할 수 있습니다. 예를 들어, 중요한 작업은 즉시 처리하는 수신기가 처리하고 덜 중요한 백그라운드 작업은 덜 바쁜 시간에 실행하도록 정해진 수신기에서 처리하도록 우선 순위를 정할 수 있습니다. 

## 문제점과 고려 사항

이러한 패턴의 구현 방식을 결정할 때 다음 사항을 고려합니다.

솔루션의 컨텍스트에서 우선 순위를 정합니다. 예를 들어, 높은 우선 순위란 메시지를 10초 이내에 처리해야 한다는 뜻이 될 수 있습니다.  우선 순위가 높은 항목을 처리하는 데 필요한 요구 사항과 이 기준을 충족하기 위해 할당되어야 할 기타 리소스를 식별합니다. 

우선 순위가 높은 모든 항목을 반드시 우선 순위가 낮은 항목보다 먼저 처리해야 하는지 결정합니다.  메시지를 단일 소비자 풀에서 처리하고 있는 경우, 우선 순위가 높은 메시지가 발생할 때 우선 순위 낮은 메시지를 처리 중인 작업을 선취하여 중단할 수 있는 메커니즘을 제공해야 합니다. 

다중 큐 방식에서, 각 큐 별 소비자 풀이 아니라 모든 큐에서 대기하는 단일 소비자 프로세스 풀을 사용하는 경우, 소비자는 우선 순위가 낮은 큐의 메시지보다 우선 순위가 높은 큐의 메시지를 항상 먼저 처리하는 알고리즘을 적용해야 합니다. 

우선 순위가 높은 큐 및 낮은 큐의 처리 속도를 모니터링하여 이 큐의 메시지가 예상 속도로 처리되는지 확인합니다. 

우선 순위 낮은 메시지가 처리되도록 보증해야 하는 경우, 다중 소비자 풀로 다중 메시지 큐 방식을 구현해야 합니다.  또는, 메시지 우선 순위 지정을 지원하는 큐에서, 큐에 포함된 메시지의 우선 순위를 시간이 지나면서 동적으로 높이는 것도 가능합니다. 그러나, 이 방식은 이러한 기능을 제공하는 메시지 큐에 따라 달라집니다. 

각 메시지 우선 순위마다 별도 큐를 사용하는 방식은 소량의 우선 순위가 잘 정의되어 있는 시스템에 가장 적합합니다. 

메시지 우선 순위는 시스템에서 논리적으로 결정할 수 있습니다. 예를 들어 명시적으로 우선 순위가 높거나 낮은 메시지보다, '유료 소비자' 또는 '무료 소비자'로 지정할 수 있습니다.  비즈니스 모델에 따라, 시스템은 무료 소비자보다 유료 소비자의 처리 메시지에 더 많은 리소스를 할당할 수 있습니다.

큐의 메시지를 확인하면 이와 관련된 재정적 및 처리 비용이 발생할 수 있습니다(일부 상업 메시징 시스템은 메시지를 게시하거나 검색할 때, 메시지에 대해 큐를 쿼리할 때마다 소정의 수수료를 부과합니다). 여러 개의 큐를 확인하면 이 비용이 증가합니다. 

풀이 사용하는 큐의 길이에 따라 소비자 풀의 크기를 동적으로 조정할 수 있습니다. 자세한 내용은 [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx)을 참조하십시오.

## 패턴을 언제 사용할 것인가

이 패턴은 다음과 같은 시나리오에 적합합니다:

- 시스템은 우선 순위가 다양한 작업을 여러 개 처리해야 합니다.

- 다양한 사용자 또는 테넌트에 대해 서로 다른 우선 순위로 처리합니다. 

## 예시

Microsoft Azure는 정렬을 통해 메시지 자동 우선 순위 지정을 기본적으로 지원하는 큐 메커니즘을 제공하지 않습니다.  그러나 Azure 서비스 버스 주제 및 구독을 제공하여 메시지 필터링을 제공하는 큐 메커니즘을 지원하며, 이와 함께 대부분의 우선 순위 큐 구현에서 사용을 최적화해 주는 광범위하고 유연한 기능을 제공합니다. 

Azure 솔루션은 응용 프로그램이 큐와 같은 방식으로 메시지를 게시할 수 있는 서비스 버스 주제를 구현할 수 있습니다.  메시지는 응용 프로그램에 정의된 사용자 지정 속성 형식의 메타데이터를 포함할 수 있습니다. 서비스 버스 구독은 주제와 연결할 수 있으며, 이러한 구독은 속성을 기준으로 메시지를 필터링할 수 있습니다.  응용 프로그램이 메시지를 주제로 전송하면, 메시지는 소비자가 읽을 수 있는 적절한 구독으로 전달됩니다.  소비자 프로세스는 동일한 의미 체계를 메시지 큐로 사용해(구독은 논리적 큐입니다) 구독의 메시지를 검색할 수 있습니다. 다음 그림은 Azure 서비스 버스 주제 및 구독으로 우선 순위 큐를 구현하는 내용을 보여 줍니다.

![Figure 3 - Implementing a priority queue with Azure Service Bus topics and subscriptions](./_images/priority-queue-service-bus.png)


위 그림에서 응용 프로그램은 여러 개의 메시지를 만들고 각 메시지에 `Priority` 라는 사용자 지정 속성과 `High` 또는 `Low` 값을 함께 할당합니다. 응용 프로그램은 이 메시지를 주제에 게시합니다. 주제에는 `Priority` 속성을 검사하면서 메시지를 필터링하는 두 개의 연결된 구독이 있습니다.  한 구독은 `Priority` 속성이 `High`로 설정된 메시지를 수락하고 다른 구독은 `Priority` 속성이 `Low`로 설정된 메시지를 수락합니다. 소비자 풀은 각 구독의 메시지를 읽습니다.  우선 순위가 높은 구독은 풀이 더 크고, 이 소비자들은 우선 순위가 낮은 풀의 소비자보다 가용 리소스가 더 많고 더 강력한 컴퓨터에서 실행할 수 있습니다. 

이 예시에서 우선 순위가 높은 메시지와 낮은 메시지의 지정에 대해서는 특이할 사항이 없습니다. 단지 각 메시지의 속성으로 명시되는 라벨이며, 특정 구독으로 메시지를 보낼 때 사용됩니다. 추가적인 우선 순위가 필요한 경우, 이 우선 순위를 처리할 추가 구독 및 소비자 프로세스 풀을 만들기가 상대적으로 간편합니다. 

[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/priority-queue)에 있는 PriorityQueue 솔루션에 이 방식의 구현이 포함되어 있습니다. 이 솔루션은 `PriorityQueue.High` 및 `PriorityQueue.Low`라는 두 개의 작업자 역할 프로젝트를 포함하고 있습니다. 이 작업자 역할은 `OnStart` 메서드의 지정 구독과 연결하는 기능이 포함된 `PriorityWorkerRole` 클래스에서 상속됩니다.

`PriorityQueue.High` 및 `PriorityQueue.Low` 작업자 역할은 해당 구성 설정에서 정의한 다양한 구독과 연결됩니다. 관리자는 각 역할의 실행 횟수를 다양하게 설정할 수 있습니다. 일반적으로 `PriorityQueue.High` 작업자 역할의 인스턴스가 `PriorityQueue.Low` 작업자 역할보다 더 많습니다. 

`PriorityWorkerRole` 클래스의`Run` 메서드는  큐에 수신된 각 메시지에 대해 가상 `ProcessMessage` 메서드(`PriorityWorkerRole` 클래스에도 정의되어 있음)가 실행되도록 준비합니다. 다음 코드는 `Run` 및 `ProcessMessage` 메서드를 보여 줍니다. PriorityQueue.Shared 프로젝트에 정의된 `QueueManager` 클래스는 Azure 서비스 버스 큐를 사용하기 위한 도우미 메서드를 제공합니다. 

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
`PriorityQueue.High` 및 `PriorityQueue.Low` 작업자 역할은 모두 `ProcessMessage` 메서드의 기본 기능을 오버라이드합니다. 아래 코드는 `PriorityQueue.High` 작업자 역할에 대한 `ProcessMessage` 메서드를 보여 줍니다. 

```csharp
protected override async Task ProcessMessage(BrokeredMessage message)
{
  // Simulate message processing for High priority messages.
  await base.ProcessMessage(message);
  Trace.TraceInformation("High priority message processed by " +
    RoleEnvironment.CurrentRoleInstance.Id + " MessageId: " + message.MessageId);
}
```

응용 프로그램이 `PriorityQueue.High` 및 `PriorityQueue.Low` 작업자 역할에서 사용하는 구독과 연결된 주제에 메시지를 게시하는 경우, 아래 코드 예시에 나타난 것처럼 `Priority` 사용자 지정 속성을 사용하여 우선 순위를 지정합니다. 이 코드(PriorityQueue.Sender 프로젝트의 `WorkerRole` 클래스에 구현됨)는 `QueueManager` 클래스의 `SendBatchAsync` 도우미 메서드를 사용해 메시지를 주제에 일괄적으로 게시합니다. 

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

## 관련 패턴 및 지침

이 패턴을 구현할 때 아래 패턴과 지침도 관련될 수 있습니다.

- 이 패턴을 잘 보여주는 예시는 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/priority-queue)에 나타나 있습니다.

- [비동기 메시징 프라이머](https://msdn.microsoft.com/library/dn589781.aspx). 요청을 처리하는 소비자 서비스는 요청을 게시한 응용 프로그램 인스턴스에 회신을 보내야 할 수 있습니다. 요청/응답 메시징 구현에 사용할 수 있는 전략에 대한 정보를 제공합니다.

- [경쟁 소비자 패턴](competing-consumers.md). 큐의 처리량을 높이기 위해, 여러 명의 소비자가 같은 큐에 대기하고 작업을 동시에 처리하는 것이 가능합니다.  이 소비자들은 메시지를 위해 경쟁하지만, 한 명만 각 메시지를 처리할 수 있습니다.  이 방식의 장점과 단점에 대해 자세히 설명합니다.

- [제한 패턴](throttling.md). 큐를 사용해 제한을 구현할 수 있습니다. 우선 메시지를 사용하여 중요한 응용 프로그램 또는 고급 소비자가 실행하는 응용 프로그램의 요청에 덜 중요한 응용 프로그램의 요청보다 높은 우선 순위를 매깁니다.

- [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx). 큐의 길이에 따라 큐를 처리하는 소비자 프로세스 풀의 크기르 조정할 수 있습니다. 이 전략을 통해 특히 우선 순위가 높은 메시지를 처리하는 풀에서 성능을 향상시킬 수 있습니다.

- 아비세크 랄의 블로그에 게시된 [서비스 버스를 사용한 엔터프라이즈 통합 패턴](http://abhishekrlal.com/2013/01/11/enterprise-integration-patterns-with-service-bus-part-2/).

