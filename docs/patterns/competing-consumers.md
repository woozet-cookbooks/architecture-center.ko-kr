---
title: Competing Consumers
description: Enable multiple concurrent consumers to process messages received on the same messaging channel.
keywords: design pattern
author: dragon119
ms.service: guidance
ms.topic: article
ms.author: pnp
ms.date: 03/24/2017

pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: [messaging]
---

# 경쟁 소비자

[!INCLUDE [header](../_includes/header.md)]

복수의 동시 소비자가 동일한 메시징 채널에서 수신한 메시지를 처리할 수 있게 해줍니다. 따라서 시스템은 여러 메시지를 동시에 처리해 처리량을 최적화하고, 확장성과 가용성을 향상시키며, 워크로드를 분산시킬 수 있습니다.

## 배경 및 문제

클라우드에서 실행하는 응용 프로그램은 많은 요청을 처리할 것으로 예상됩니다. 일반적인 기법은 응용 프로그램이 각 요청을 동기적으로 처리하기보다 메시징 시스템을 거쳐 비동기적으로 처리하는 다른 서비스(소비자 서비스)에 요청을 전달하는 것입니다. 이런 전략은 요청을 처리하는 동안에 응용 프로그램의 비즈니스 논리가 차단되지 않는다는 장점이 있습니다.

요청 횟수는 많은 이유로 인해 시간에 따라 크게 달라질 수 있습니다. 사용자 활동 또는 여러 테넌트가 전송하는 집계 요청의 갑작스러운 증가는 예기치 않은 워크로드를 초래할 수 있습니다. 사용량이 가장 많은 시간에 시스템은 초당 수백 개의 요청을 처리해야 하는 반면, 다른 시간에 요청 횟수는 매우 적을 수 있습니다. 게다가 이런 요청을 처리하기 위해 수행하는 작업의 특성은 상당히 가변적일 수 있습니다. 소비자 서비스의 단일 인스턴스를 사용하면 단일 인스턴스가 요청으로 인해 서비스 장애를 일으킬 수 있거나 메시징 시스템은 응용 프로그램에서 전송하는 메시지가 밀어닥쳐 오버로드가 발생할 수 있습니다. 이런 변동 워크로드를 처리하기 위해 시스템은 소비자 서비스의 여러 인스턴스를 실행할 수 있습니다. 그러나 이런 소비자는 각 메시지가 단일 소비자에게만 전달되도록 조정되어야 합니다. 워크로드도 인스턴스가 병목 현상을 일으키지 않도록 소비자 사이에서 부하 분산이 필요합니다.

## 해결책

메시지 큐를 사용해 응용 프로그램과 소비자 서비스의 인스턴스 사이에 통신 채널을 구현합니다. 응용 프로그램은 메시지 형태의 요청을 큐에 게시하고, 소비자 서비스 인스턴스는 메시지를 큐에서 수신해 처리합니다.  응용 프로그램은 소비자 서비스 인스턴스의 동일한 풀을 사용해 응용 프로그램의 인스턴스에서 전송한 메시지를 처리할 수 있습니다. 다음 그림은 메시지 큐를 사용해 작업을 서비스의 인스턴스로 분산시키는 과정을 보여줍니다.

![Using a message queue to distribute work to instances of a service](./_images/competing-consumers-diagram.png)

이 해결책은 다음과 같은 장점을 제공합니다.

- 응용 프로그램 인스턴스가 전송하는 요청량의 폭넓은 변화를 처리할 수 있는 부하 평준화 시스템을 제공합니다. 큐는 응용 프로그램 인스턴스와 소비자 서비스 인스턴스 사이의 버퍼로 작용합니다. 따라서 [큐 기반 부하 평준화 패턴](queue-based-load-leveling.md)에 설명한 대로 응용 프로그램과 서비스 인스턴스의 가용성과 응답성에 미치는 영향을 최소화하는 데 도움을 줄 수 있습니다. 일부 장기 실행 처리가 필요한 메시지의 처리는 다른 메시지를 소비자 서비스의 다른 인스턴스가 동시에 처리하는 것을 막지 않습니다.

- 안정성을 향상시킵니다. 생산자가 이 패턴을 사용하는 대신에 소비자와 직접 통신하지만 소비자를 모니터링하지 않는 경우, 메시지가 손실되거나 소비자가 실패하면 처리에 실패할 가능성이 매우 높습니다. 이런 패턴에서 메시지는 특정 서비스 인스턴스에 전송되지 않습니다. 실패한 서비스 인스턴스는 생산자를 차단하지 않을 것이며, 메시지는 작동 중인 서비스 인스턴스가 처리할 수 있습니다.

- 소비자 사이 또는 생산자와 소비자 인스턴스 사이의 복잡한 조정이 필요 없습니다. 메시지 큐는 각 메시지가 한 번 이상 전달되는 것을 보장합니다.

- 확장이 가능합니다. 시스템은 메시지의 양이 변함에 따라 소비자 서비스의 인스턴스 숫자를 동적으로 늘리거나 줄일 수 있습니다.

- 메시지 큐가 트랜잭션 읽기 작업을 제공하는 경우 복원력을 향상시킬 수 있습니다. 소비자 서비스 인스턴스가 트랜잭션 작업의 일부로 메시지를 읽고 처리하며 소비자 서비스 인스턴스가 실패하는 경우, 이 패턴은 소비자 서비스의 다른 인스턴스가 선택해 처리할 수 있도록 메시지를 큐에 반환할 수 있습니다.

## 문제 및 고려 사항

이 패턴의 구현 방법을 결정할 때는 다음 사항을 고려합니다.

- **메시지 정렬**. 소비자 서비스 인스턴스가 메시지를 수신하는 순서는 확정되지 않으며 반드시 메시지가 만들어진 순서를 반영하는 것은 아닙니다. 멱등성은 메시지를 처리하는 순서에 대한 의존성을 제거하는 데 도움을 주기 때문에, 메시지 처리가 멱등성을 제공하도록 시스템을 디자인합니다. 추가 정보는 Jonathan Oliver의 블로그에서 [멱등성 패턴](http://blog.jonathanoliver.com/2010/04/idempotency-patterns/) 을 참조하세요.

    > Microsoft Azure Service Bus Queues는 메시지 세션을 사용해 보장된 메시지의 선입선출 정렬을 구현할 수 있습니다. 추가 정보는 [세션을 사용하는 메시징 패턴](https://msdn.microsoft.com/magazine/jj863132.aspx)을 참조하세요.

- **복원력을 위한 서비스 디자인**. 시스템이 실패한 서비스 인스턴스를 검색하고 다시 시작하도록 디자인된 경우,  한 번 이상 가져와 처리하는 단일 메시지의 영향을 최소화하기 위해 서비스 인스턴스가 수행하는 처리를 멱등 작업으로 구현하는 것이 필요할 수 있습니다.

- **포이즌 메시지 검색**. 잘못된 형식의 메시지 또는 사용할 수 없는 리소스에 액세스를 요구하는 작업은 서비스 인스턴스의 실패를 초래할 수 있습니다. 시스템은 이런 메시지가 큐에 반환되는 것을 방지해야 하며 대신에 필요한 경우 분석할 수 있도록 이런 메시지의 세부 정보를 캡처하고 다른 위치에 저장해야 합니다.

- **결과 처리**. 메시지를 처리하는 서비스 인스턴스는 메시지를 생성하는 응용 프로그램 논리에서 완전히 분리되며 직접 통신할 수 없습니다. 서비스 인스턴스가 응용 프로그램 논리에 다시 전달해야 하는 결과를 생성하는 경우, 이런 정보는 서비스 인스턴스와 응용 프로그램 논리에 액세스할 수 있는 위치에 저장되어야 합니다. 응용 프로그램 논리가 불완전한 데이터를 검색하지 않도록 시스템은 처리가 완료되는 시점을 표시해야 합니다.

     > Azure를 사용하고 있는 경우, 작업자 프로세스는 전용 메시지 회신 큐를 사용해 결과를 응용 프로그램 논리에 다시 전달할 수 있습니다. 응용 프로그램 논리는 이런 결과를 원래 메시지와 상관시킬 수 있어야 합니다. 이런 시나리오는 [비동기 메시징 프라이머](https://msdn.microsoft.com/library/dn589781.aspx) 에 더 자세히 설명되어 있습니다.

- **메시징 시스템의 크기 조정**. 대규모 솔루션에서 단일 메시지 큐는 메시지 개수가 너무 많아 시스템에서 병목 현상을 일으킬 수 있습니다. 이런 상황에서는 메시징 시스템을 분할해 메시지를 특정 생산자에서 특별한 큐로 보내거나 부하 분산을 사용해 여러 메시지 큐에 메시지를 분산시키는 것이 좋습니다. 

- **메시징 시스템의 안정성 보장**. 신뢰할 수 있는 메시징 시스템은 응용 프로그램이 메시지가 손실되지 않도록 메시지를 큐에 넣는 것을 필요로 합니다. 메시지를 큐에 넣으면 모든 메시지를 한 번 이상 전달하는 것이 보장됩니다.

## 이 패턴을 사용하는 경우

이 패턴은 다음 경우에 사용합니다.

- 응용 프로그램의 워크로드가 비동기적으로 실행할 수 있는 작업으로 분할된 경우
- 작업이 독립적이고 동시에 실행할 수 있는 경우
- 작업의 양이 너무 가변적이어서 확장 가능한 솔루션이 필요한 경우
- 솔루션이 높은 확장성을 제공해야 하고 작업의 처리가 실패하는 경우 복원력이 있어야 하는 경우

이 패턴은 다음 경우에 유용하지 않습니다.

- 응용 프로그램 워크로드를 개별 작업으로 분리하기가 쉽지 않거나 작업 사이의 종속성이 너무 큰 경우
- 작업을 동기적으로 수행해야 하고, 응용 프로그램 논리가 계속하기 전에 작업이 완료될 때까지 기다려야 하는 경우
- 작업을 특정 순서로 수행해야 하는 경우

> 일부 메시징 시스템은 생산자가 메시지를 함께 묶고 묶은 메시지를 동일한 소비자가 모두 처리할 수 있는 세션을 지원합니다. 이런 방식은 우선 순위가 지정된 메시지(지원하는 경우)를 이용해 메시지를 단일 생산자에서 단일 소비자로 순서대로 전달하는 메시지 정렬의 형태를 구현하는 데 사용할 수 있습니다.

## 예제

Azure는 경쟁 소비자 패턴을 구현하기 위한 메커니즘으로 작용할 수 있는 저장소 큐와 Service Bus 큐를 제공합니다. 응용 프로그램 논리는 메시지를 큐에 게시할 수 있고, 하나 이상의 역할에서 작업으로 구현된 소비자는 메시지를 이런 큐에서 가져와 처리할 수 있습니다. 복원력의 경우, Service Bus 큐는 소비자가 메시지를 큐에서 가져올 때 `PeekLock` 모드를 사용하게 해줍니다. 이 모드는 실제로 메시지를 제거하지 않고 다른 소비자에게 보이지 않도록 숨깁니다. 원래 소비자는 메시지의 처리가 완료되면 메시지를 삭제할 수 있습니다. 소비자가 실패하면 보기 잠금은 시간을 초과하게 되고 메시지는 다시 보이게 되어 다른 소비자가 메시지를 검색할 수 있습니다. 

> Azure Service Bus 큐의 사용에 대한 자세한 정보는 [Service Bus 큐, 항목 및 구독](https://msdn.microsoft.com/library/windowsazure/hh367516.aspx) 을 참조하세요. 
Azure 저장소 큐의 사용에 대한 정보는 [.NET을 사용해 Azure Queue 저장소 시작](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-queues/) 을 참조하세요.

[GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) 에서 사용할 수 있는 CompetingConsumers 솔루션에 있는 `QueueManager` 클래스의 다음 코드는 웹 또는 작업자 역할에서 `Start` 이벤트 처리기에 있는 `QueueClient` 인스턴스를 사용해 큐를 만드는 방법을 보여줍니다.

```csharp
private string queueName = ...;
private string connectionString = ...;
...

public async Task Start()
{
  // Check if the queue already exists.
  var manager = NamespaceManager.CreateFromConnectionString(this.connectionString);
  if (!manager.QueueExists(this.queueName))
  {
    var queueDescription = new QueueDescription(this.queueName);

    // Set the maximum delivery count for messages in the queue. A message
    // is automatically dead-lettered after this number of deliveries. The
    // default value for dead letter count is 10.
    queueDescription.MaxDeliveryCount = 3;

    await manager.CreateQueueAsync(queueDescription);
  }
  ...

  // Create the queue client. By default the PeekLock method is used.
  this.client = QueueClient.CreateFromConnectionString(
    this.connectionString, this.queueName);
}
```

다음 코드 조각은 응용 프로그램이 메시지의 묶음을 만들고 큐에 전송하는 방법을 보여줍니다.

```csharp
public async Task SendMessagesAsync()
{
  // Simulate sending a batch of messages to the queue.
  var messages = new List<BrokeredMessage>();

  for (int i = 0; i < 10; i++)
  {
    var message = new BrokeredMessage() { MessageId = Guid.NewGuid().ToString() };
    messages.Add(message);
  }
  await this.client.SendBatchAsync(messages);
}
```

다음 코드는 소비자 서비스 인스턴스가 이벤트 구동 접근을 따라 메시지를 큐에서 가져오는 방법을 보여줍니다. `processMessageTask` 매개 변수에서 `ReceiveMessages` 메서드 사이는 메시지가 수신될 때 실행할 코드를 참조하는 대리자입니다. 이 코드는 비동기적으로 실행됩니다.

```csharp
private ManualResetEvent pauseProcessingEvent;
...

public void ReceiveMessages(Func<BrokeredMessage, Task> processMessageTask)
{
  // Set up the options for the message pump.
  var options = new OnMessageOptions();

  // When AutoComplete is disabled it's necessary to manually
  // complete or abandon the messages and handle any errors.
  options.AutoComplete = false;
  options.MaxConcurrentCalls = 10;
  options.ExceptionReceived += this.OptionsOnExceptionReceived;

  // Use of the Service Bus OnMessage message pump.
  // The OnMessage method must be called once, otherwise an exception will occur.
  this.client.OnMessageAsync(
    async (msg) =>
    {
      // Will block the current thread if Stop is called.
      this.pauseProcessingEvent.WaitOne();

      // Execute processing task here.
      await processMessageTask(msg);
    },
    options);
}
...

private void OptionsOnExceptionReceived(object sender,
  ExceptionReceivedEventArgs exceptionReceivedEventArgs)
{
  ...
}
```

Azure에서 제공하는 자동 크기 조정 기능은 큐 길이가 변함에 따라 역할 인스턴스를 시작하고 정지하는 데 사용할 수 있습니다. 추가 정보는 [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx)을 참조하세요. 또한 역할 인스턴스와 작업자 프로세스 사이에 일대일 대응 관계를 유지할 필요가 없습니다(하나의 역할 인스턴스는 여러 작업자 프로세스를 구현할 수 있음). 추가 정보는 [계산 리소스 통합 패턴](compute-resource-consolidation.md) 을 참조하세요.

## 관련 패턴과 지침

다음 패턴과 지침은 이 패턴을 구현할 때 관련될 수 있습니다.

- [비동기 메시징 프라이머](https://msdn.microsoft.com/library/dn589781.aspx). 메시지 큐는 비동기 통신 메커니즘입니다. 소비자 서비스가 회신을 응용 프로그램에 전송해야 하는 경우, 특정 형태의 응답 메시징을 구현하는 것이 필요할 수 있습니다. 비동기 메시징 프라이머는 메시지 큐를 사용해 요청/회신 메시징의 구현 방법에 대한 정보를 제공합니다.

- [자동 크기 조정 지침](https://msdn.microsoft.com/library/dn589774.aspx). 응용 프로그램이 메시지를 게시하는 큐의 길이가 변하기 때문에, 소비자 서비스 인스턴스를 시작하고 정지할 수 있습니다. 자동 크기 조정은 최대 처리 시간 동안 처리량을 유지하는 데 도움을 줄 수 있습니다.

- [계산 리소스 통합 패턴](compute-resource-consolidation.md). 비용과 관리 오버헤드를 줄이기 위해 소비자 서비스의 여러 인스턴스를 하나의 프로세스에 통합할 수 있습니다. 계산 리소스 통합 패턴은 계산 리소스 통합 접근의 장점과 단점을 설명합니다.

- [큐 기반 부하 평준화 패턴](queue-based-load-leveling.md). 메시지 큐를 도입하면 시스템에 복원력을 추가해 서비스 인스턴스가 응용 프로그램 인스턴스에서 전송하는 요청의 폭넓게 변하는 양을 처리할 수 있습니다. 메시지 큐는 버퍼로 작용해 부하 평준화를 제공합니다. 큐 기반 부하 평준화 패턴은 이런 시나리오를 더 자세히 설명합니다.

- 이 패턴은 관련된 [샘플 응용 프로그램](https://github.com/mspnp/cloud-design-patterns/tree/master/competing-consumers) 을 제공합니다.
