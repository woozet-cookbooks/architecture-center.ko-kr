---
title: Circuit Breaker
description: Handle faults that might take a variable amount of time to fix when connecting to a remote service or resource.
keywords: design pattern
author: dragon119
ms.service: guidance
ms.topic: article
ms.author: pnp
ms.date: 03/24/2017

pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: [resiliency]
---

# 회로 차단기

원격 서비스 또는 리소스에 연결 시 복구 시간이 가변적인 장애를 처리합니다. 따라서 응용 프로그램의 안정성과 복원력을 향상시킬 수 있습니다.

## 배경 및 문제

분산 환경에서는 느린 네트워크 연결, 시간 초과 또는 오버 커밋되거나 일시적으로 사용할 수 없는 리소스와 같은 일시적인 장애로 인해 원격 리소스와 서비스 호출이 실패할 수 있습니다. 보통 이런 장애는 시간이 지나면 저절로 해결되는데, 강력한 클라우드 응용 프로그램이라면 [다시 시도 패턴][retry-pattern] 과 같은 전략을 사용해 장애를 처리하도록 설계되어야 합니다.

그러나 장애가 예상하지 못한 이벤트로 인해 발생하여 해결 시간이 훨씬 더 소요되는 상황도 있을 수 있는데, 이런 장애는 연결의 부분 상실부터 서비스의 완전한 실패까지 심각도가 다양할 수 있습니다. 이런 상황에서는 응용 프로그램이 성공할 가능성이 없는 작업을 계속 다시 시도하는 것이 의미가 없습니다. 대신 응용 프로그램은 작업 실패를 신속하게 수락하고 이런 장애를 적절하게 처리해야 합니다.

더구나 서비스가 매우 바쁜 경우, 시스템의 한 부분에서 발생한 장애는 단계적 장애로 이어질 수 있습니다. 예를 들면 서비스를 호출하는 작업에 시간 제한을 구현한 뒤 서비스가 제한 시간 이내에 응답에 실패하는 경우 실패 메시지를 표시하도록 구성할 수 있습니다. 그러나 이런 전략으로 인해 동일한 작업에 대한 많은 동시 요청이 제한 시간이 경과할 때까지 차단되는 결과가 초래될 수 있는데, 이렇게 차단된 요청은 메모리, 스레드, 데이터베이스 연결 등과 같은 중요한 시스템 리소스를 차지합니다. 결국 이와 같은 리소스가 차단된 요청으로 인해 소진됨에 따라 시스템 내에서 동일한 리소스를 사용해야 하는 관련 없는 부분의 장애가 초래될 수 있습니다. 이런 상황에서는 작업을 즉시 실패하게 하고 성공할 가능성이 있는 서비스의 호출만 시도하는 것이 바람직할 수 있습니다. 시간 제한을 더 짧게 설정하면 이런 문제를 손 쉽게 해결할 수 있지만, 시간 제한이 너무 짧으면 결과적으로 서비스 요청이 성공하더라도 작업이 실패할 수 있으므로 시간 제한을 너무 짧게 설정해서는 안 됩니다.

## 해결책

회로 차단기 패턴을 사용하면 응용 프로그램이 실패할 가능성이 있는 작업의 실행을 반복적으로 시도하지 못하도록 제어할 수 있습니다. 그러면 장애가 해결될 때까지 기다리거나 장애가 오래 지속되는 유형인지 여부를 결정하는 동안 CPU 사이클을 낭비하지 않고 계속 운영할 수 있습니다. 회로 차단기 패턴을 사용하면 응용 프로그램이 장애가 해결되었는지 감지할 수 있도록 지원할 수 있습니다. 응용 프로그램이 문제가 해결되었다고 판단하면 작업의 호출을 시도할 수 있습니다.

> 회로 차단기 패턴의 목적은 다시 시도 패턴과 다릅니다. 다시 시도 패턴을 사용하면 응용 프로그램이 성공할 것으로 예상되는 작업을 다시 시도하지만 회로 차단기 패턴을 사용하면 응용 프로그램이 실패할 가능성이 있는 작업의 수행을 중단합니다. 응용 프로그램은 다시 시도 패턴을 사용하여 두 패턴을 조합한 뒤 회로 차단기를 통해 작업을 호출할 수 있습니다. 그러나 다시 시도 논리는 회로 차단기가 반환하는 예외에 민감해야 할 뿐 아니라 회로 차단기가 장애가 일시적인 것이 아니라고 표시하는 경우에는 다시 시도를 포기해야 합니다.

회로 차단기는 실패가능성이 있는 작업의 프록시로 작용합니다. 프록시는 최근 발생한 실패 횟수를 모니터링하고 실패 횟수 정보를 사용해 작업을 진행할 것인지, 아니면 단순히 예외를 즉시 반환할 것인지 여부를 결정해야 합니다.

프록시는 전기 회로 차단기의 기능을 모방하는 다음 상태를 포함한 상태 시스템으로 구현할 수 있습니다.

- **폐쇄**: 응용 프로그램의 요청이 작업에 전달됩니다. 프록시는 최근 실패 횟수의 카운트를 유지하고 있다가 작업의 호출이 실패하는 경우 실패 횟수 카운트를 하나씩 늘립니다. 최근 실패 횟수가 지정된 시간 이내에 지정된 임계값을 초과하면 프록시는 **개방** 상태로 전환됩니다. 개방 상태로 전환된 프록시는 시간 초과 타이머를 시작하고, 시간 초과 타이머가 만료되면 프록시는 **반개방** 상태로 전환됩니다.

    > 시간 초과 타이머는 응용 프로그램이 작업의 수행을 다시 시도하기 전에 먼저 시스템이 실패의 원인이 되는 문제를 해결할 시간을 확보하기 위한 것입니다.

- **개방**: 응용 프로그램의 요청이 즉시 실패하고 예외가 응용 프로그램에 반환됩니다.

- **반개방**: 응용 프로그램으로부터 제한된 횟수의 요청이 통과되어 작업을 호출할 수 있게 됩니다. 이와 같은 요청이 성공하면 이전에 실패를 유발한 장애가 해결되었다고 판단하여 회로 차단기가 **폐쇄** 상태로 전환되고 실패 카운터가 초기화되지만 이와 같은 요청이 실패하면 장애가 여전히 남아 있다고 판단하여 회로 차단기가 다시 **개방** 상태로 되돌아가고 시간 초과 타이머를 다시 시작해 시스템이 장애를 복구할 추가 시간을 확보합니다.

    > **반개방** 상태는 복구 중인 서비스가 요청으로 인해 갑자기 서비스 장애를 일으키는 것을 방지하는 데 유용합니다. 복구가 진행되는 동안 서비스는 복구가 완료될 때까지 제한된 양의 요청을 지원할 수 있지만, 복구가 진행되는 동안 엄청난 양의 작업이 요청되면 서비스의 시간 초과 또는 실패가 다시 초래될 수 있습니다.

![Circuit Breaker states](./_images/circuit-breaker-diagram.png)

위의 그림에서 **폐쇄** 상태가 사용하는 실패 카운터는 시간을 기반으로 하고 일정간 간격에 따라 자동으로 초기화됩니다. 이와 같은 정기적 초기화는 실패가 간헐적으로 발생하는 경우 회로 차단기가 **개방** 상태로 전환되지 못하도록 방지하는 데 도움이 됩니다. 지정된 실패 횟수가 지정된 간격 중 발생하는 경우에만 회로 차단기를 **개방** 상태로 전환하는 실패 임계값에 도달합니다. **반개방** 상태가 사용하는 카운터는 작업을 호출하기 위한 시도 중 성공 횟수를 기록합니다. 회로 차단기는 지정된 횟수의 연속 작업 호출이 성공하면 **폐쇄** 상태로 되돌아가지만 호출이 실패하면 회로 차단기는 즉시 **개방** 상태로 전환되고 성공 카운터는 다음에 회로 차단기가 **반개방** 상태로 전환될 때 초기화됩니다.

> 시스템 복구는 실패한 구성 요소를 복원하거나 다시 시작하거나 네트워크 연결을 복구하는 등의 외부적인 방법으로 이루어집니다.

시스템이 장애를 복구하고 성능에 미치는 영향을 최소화하는 동안 안정성을 제공하는 회로 차단기 패턴은 작업의 시간 초과를 기다리기보다 실패할 가능성이 있는 작업의 요청을 신속하게 거부하거나 요청을 반환하지 않음으로써 시스템의 응답 시간을 유지하는 데 도움이 될 수 있습니다. 회로 차단기가 상태를 변경할 때마다 이벤트를 발생시키면 이런 정보를 회로 차단기가 보호하는 시스템 요소의 상태를 모니터링하는데 사용하거나 회로 차단기가 **개방** 상태로 전환될 때 관리자에게 경보를 발령하는 데 사용할 수 있습니다.

회로 차단기 패턴은 사용자 지정이 가능하고 가능한 실패 유형에 따라 조정할 수 있습니다. 예를 들어 증가식 시간 초과 타이머를 회로 차단기에 적용할 수 있습니다. 처음 몇 초간은 회로 차단기를 **개방** 상태로 둔 다음, 실패가 해결되지 않으면 시간 제한을 몇 분으로 늘릴 수 있습니다. 실패를 반환하고 예외를 발생시키는 **개방** 상태보다 작업에 의미 있는 기본값을 반환하는 것이 유용한 경우도 있습니다.

## 문제 및 고려 사항

이 패턴의 구현 방법을 결정할 때는 다음 사항을 고려해야 합니다.

**예외 처리**. 회로 차단기를 통해 작업을 호출하는 응용 프로그램은 작업을 사용할 수 없는 경우 발생된 예외를 처리하도록 설계되어야 합니다. 예외를 처리하는 방식은 응용 프로그램에 따라 다릅니다. 예를 들면 응용 프로그램은 기능을 임시로 저하시키고, 동일한 작업의 수행 또는 동일한 데이터의 획득을 시도하기 위해 대안 작업을 호출하며, 사용자에게 예외를 보고하고 나중에 다시 시도할 것인지 질의할 수 있습니다.

**예외 유형**. 요청은 많은 이유로 인해 실패하는데, 실패의 원인에 따라 실패의 심각도가 달라집니다. 예를 들어 원격 서비스의 충돌이 일어나고 이를 복구하는 데 몇 분이 걸리기 때문에 실패하는 요청이 있는가 하면 일시적으로 과부하가 발생한 서비스로 인한 시간 초과 때문에 실패하는 요청도 있습니다. 회로 차단기는 발생하는 예외의 유형을 검사하고 이런 예외의 특성에 따라 전략을 조정할 수 있습니다. 회로 차단기를 **개방** 상태로 전환하는데 필요한 실패 횟수를 예로 들면, 시간 초과 예외의 실패 횟수가 완전히 사용할 수 없는 서비스로 인한 실패 횟수에 비해 더 많아야 합니다.

**로깅**. 회로 차단기는 실패한 모든 요청(과 가능하면 성공한 요청)을 로그에 기록해 관리자가 작업의 상태를 모니터링할 수 있도록 지원해야 합니다.

**복구성**. 보호 중인 작업의 가능한 복구 패턴과 일치하도록 회로 차단기를 구성해야 합니다. 예를 들어 오랫 동안 **개방** 상태를 유지한 회로 차단기는 실패 이유가 해결되었더라도 예외를 발생시킬 수 있는 반면 **개방** 상태에서 **반개방** 상태로 너무 빠르게 전환된 회로 차단기는 응용 프로그램의 응답 시간을 변경하고 줄일 수 있습니다.

**실패한 작업 테스트**. 회로 차단기는 타이머를 사용해 **개방** 상태에서 **반개방** 상태로 전환할 시점을 결정하는 대신 원격 서비스 또는 리소스에 정기적으로 핑을 전송해 회로 차단기를 다시 사용할 수 있을지 여부를 결정할 수 있습니다. 이런 핑은 이전에 실패한 작업을 호출하기 위한 시도로 이루어질 수 있는데, [상태 끝점 모니터링 패턴](health-endpoint-monitoring.md)에 설명된 대로 특히 서비스의 상태를 테스트하기 위해 원격 서비스가 제공하는 특별한 작업을 사용할 수도 있습니다.

**수동 재정의**. 실패한 작업의 복구 시간이 굉장히 가변적인 시스템에서는 관리자가 회로 차단기를 닫고 실패 카운터를 초기화할 수 있는 수동 초기화 옵션을 제공하는 것이 바람직할 수 있습니다. 이와 비슷하게 관리자는 회로 차단기가 보호하는 작업을 일시적으로 사용할 수 없는 경우 회로 차단기를 강제로 **개방** 상태로 전환하고 시간 초과 타이머를 다시 시작할 수 있습니다.

**동시성**. 많은 수의 응용 프로그램 동시 인스턴스가 하나의 회로 차단기에 액세스할 수 있습니다. 따라서 회로 차단기는 동시 요청을 차단하거나 작업을 호출할 때마다 과도한 오버헤드를 추가하지 않도록 구현되어야 합니다.

**리소스 차별화**. 기본 독립 공급자가 다수 존재할 수 있는 경우, 하나의 리소스 유형을 위해 하나의 회로 차단기를 사용한다면 주의해야 합니다. 예를 들어 복수의 분할된 데이터베이스를 포함하는 데이터 저장소의 경우 어느 분할된 데이터베이스에는 완전히 액세스할 수 있는 반면 어느 분할된 데이터베이스에는 일시적인 문제가 발생할 수 있습니다. 이와 같은 시나리오에서 오류 응답이 병합되면 응용 프로그램은 실패할 가능성이 매우 높은 분할된 데이터베이스에 엑세스를 시도하는가 하면 성공할 가능성이 있는 분할된 데이터베이스에 대한 액세스를 차단하기도 합니다.

**회로 차단 가속화**. 때로 실패 응답에는 즉시 전환되고 최소한의 시간 동안 전환된 상태를 유지하는 회로 차단기에 대한 충분한 정보가 포함되어 있을 수 있습니다. 예를 들어 과부하가 발생한 공유 리소스의 오류 응답은 응용 프로그램의 즉시 다시 시도보다는 몇 분 후 다시 시도를 권장한다는 것을 의미할 수 있습니다.

> [!참고]
> 클라이언트를 제한하는 서비스의 경우 HTTP 429(너무 많은 요청)를 반환할 수 있고, 현재 사용할 수 없는 서비스의 경우 HTTP 503(서비스 사용 불가)을 반환할 수 있습니다. 응답에는 지연의 예상 지속 시간과 같은 추가 정보가 포함되어 있을 수 있습니다.

**실패한 요청 재생**. **개방** 상태의 회로 차단기는 단순히 신속한 실패를 선택하기보다 각 요청의 세부 사항을 저널에 기록하고 원격 리소스 또는 서비스를 사용할 수 있게 될 때 재생할 요청을 정렬할 수도 있습니다.

**외부 서비스에서의 부적절한 제한 시간**. 회로 차단기는 제한 시간이 길게 지정된 외부 서비스에서 실패하는 작업에 대해 응용 프로그램을 완전히 보호할 수 없습니다. 제한 시간이 너무 길면 회로 차단기를 실행하는 스레드가 상당히 긴 시간 동안 차단되고, 그 뒤에야 회로 차단기가 작업 실패를 표시하게 됩니다. 이때 다른 응용 프로그램의 수 많은 인스턴스도 회로 차단기를 통해 서비스의 호출을 시도할 수 있고 스레드가 모두 실패하기 전에 상당한 수의 스레드를 묶을 수 있습니다.

## 패턴 사용 사례

다음의 경우에 이 패턴을 사용합니다.

- 실패 가능성이 매우 높은 경우 응용 프로그램이 원격 서비스를 호출하거나 공유 리소스에 액세스하려는 시도를 방지

다음의 경우에는 이 패턴을 권장하지 않습니다.

- 메모리 내 데이터 구조와 같은 응용 프로그램의 로컬 개인 리소스에 대한 액세스 처리. 이런 환경에서 회로 차단기를 사용하면 시스템에 오버헤드가 추가될 수 있습니다.
- 사용자 응용 프로그램의 비즈니스 논리에서 예외를 처리하기 위한 대안으로 사용

## 예제

웹 응용 프로그램의 여러 페이지는 외부 서비스에서 검색한 데이터로 채워집니다. 시스템이 최소 캐싱을 구현하는 경우, 이런 페이지에 대한 호출이 늘어날수록 서비스까지의 왕복 이동이 늘어나게 됩니다. 웹 응용 프로그램에서 서비스까지의 연결은 제한 시간(보통 60초)으로 구성할 수 있고, 서비스가 제한 시간 이내에 응답하지 않을 경우 각 웹 페이지의 논리는 서비스가 사용할 수 없는 상태이고 예외를 발생시킨다고 판단하게 됩니다.

그러나 서비스가 실패하고 시스템이 매우 바쁜 경우, 사용자는 예외가 발생하기 전에 강제로 60초를 기다려야 합니다. 결국 메모리, 연결, 스레드와 같은 리소스가 소진될 수 있고, 그에 따라 서비스에서 데이터를 검색하는 페이지에 액세스하고 있지 않은 다른 사용자까지 시스템에 연결되지 못할 수 있습니다.

웹 서버를 추가하고 부하 분산을 구현해 시스템을 확장하더라도 리소스가 소진되면 지연될 수 있어 문제를 해결할 수 없습니다. 사용자 요청에 대해 여전히 응답이 없고 모든 웹 서버가 결국 리소스를 전부 소진할 수 있기 때문입니다.

서비스에 연결하고 회로 차단기에서 데이터를 검색하는 논리의 래핑은 이런 문제를 해결하고 서비스 실패를 더 우아하게 처리하는 데 도움이 될 수 있습니다. 사용자 요청이 여전히 실패하는 경우, 사용자 요청은 더 빠르게 실패하고 리소스는 차단되지 않을 것이기 때문입니다.

`CircuitBreaker` 클래스는 회로 차단기에 대한 상태 정보를 다음 코드에서 제시하는 `ICircuitBreakerStateStore` 인터페이스를 구현하는 개체 내에 유지합니다.

```csharp
interface ICircuitBreakerStateStore
{
  CircuitBreakerStateEnum State { get; }

  Exception LastException { get; }

  DateTime LastStateChangedDateUtc { get; }

  void Trip(Exception ex);

  void Reset();

  void HalfOpen();

  bool IsClosed { get; }
}
```

`State` 속성은 회로 차단기의 현재 상태를 표시하며 `CircuitBreakerStateEnum` 열거에 정의된 **개방**, **반개방**, **폐쇄** 중 하나가 됩니다. `IsClosed` 속성은 회로 차단기가 폐쇄된 경우 true이고 개방이나 반개방인 경우 false이어야 합니다. `Trip` 메서드는 회로 차단기의 상태를 개방 상태로 전환하고 예외가 발생한 날짜 및 시간과 함께 상태 변경을 초래한 예외를 기록합니다. `LastException` 및 `LastStateChangedDateUtc` 속성은 이런 정보를 반환합니다. `Reset` 메서드는 회로 차단기를 닫고, `HalfOpen` 메서드는 회로 차단기를 반개방으로 설정합니다.

이 예제의 `InMemoryCircuitBreakerStateStore` 클래스는 `ICircuitBreakerStateStore` 인터페이스의 구현을 포함합니다. `CircuitBreaker` 클래스는 회로 차단기의 상태를 보유하는 이 클래스의 인스턴스를 생성합니다.

`CircuitBreaker` 클래스의 `ExecuteAction` 메서드는 `Action` 대리자로 지정된 작업을 래핑합니다. 회로 차단기가 닫히면 `ExecuteAction` 메서드는 `Action` 대리자를 호출합니다. 작업이 실패하면 예외 처리기는 회로 차단기를 개방으로 설정하는 `TrackException` 를 호출합니다. 다음 코드 예제는 이런 흐름을 강조해 보여줍니다.

```csharp
public class CircuitBreaker
{
  private readonly ICircuitBreakerStateStore stateStore =
    CircuitBreakerStateStoreFactory.GetCircuitBreakerStateStore();

  private readonly object halfOpenSyncObject = new object ();
  ...
  public bool IsClosed { get { return stateStore.IsClosed; } }

  public bool IsOpen { get { return !IsClosed; } }

  public void ExecuteAction(Action action)
  {
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open.
      ... (see code sample below for details)
    }

    // The circuit breaker is Closed, execute the action.
    try
    {
      action();
    }
    catch (Exception ex)
    {
      // If an exception still occurs here, simply
      // retrip the breaker immediately.
      this.TrackException(ex);

      // Throw the exception so that the caller can tell
      // the type of exception that was thrown.
      throw;
    }
  }

  private void TrackException(Exception ex)
  {
    // For simplicity in this example, open the circuit breaker on the first exception.
    // In reality this would be more complex. A certain type of exception, such as one
    // that indicates a service is offline, might trip the circuit breaker immediately.
    // Alternatively it might count exceptions locally or across multiple instances and
    // use this value over time, or the exception/success ratio based on the exception
    // types, to open the circuit breaker.
    this.stateStore.Trip(ex);
  }
}
```

다음 예제는 회로 차단기가 닫히지 않은 경우 실행되는 코드(이전 예제에서 생략)를 보여줍니다. 먼저 회로 차단기가 `CircuitBreaker` 클래스의 로컬 `OpenToHalfOpenWaitTime` 필드에 지정된 시간보다 길게 열려 있는지를 검사합니다. 그러면 `ExecuteAction` 메서드는 회로 차단기를 반개방으로 설정한 다음 `Action` 대리자로 지정된 작업의 수행을 시도합니다.

작업이 성공하면 회로 차단기는 폐쇄 상태로 초기화됩니다. 작업이 실패하면 회로 차단기는 다시 개방 상태로 전환되며 예외가 발생하는 시간이 업데이트되어 회로 차단기는 추가 시간을 기다렸다가 작업의 수행을 다시 시도합니다.

회로 차단기가 `OpenToHalfOpenWaitTime` 값보다 짧은 시간 동안만 열리면 `ExecuteAction` 메서드는 단순히 `CircuitBreakerOpenException` 예외를 발생시키고 회로 차단기를 개방 상태로 전환하게 한 오류를 반환합니다.

거기에 더해 반개방 상태에 있는 회로 차단기가 작업을 동시에 호출하는 시도를 방지하기 위해 잠금을 사용합니다. 작업을 호출하기 위한 동시 시도는 회로 차단기가 열려 있는 것처럼 처리되며 나중에 설명하는 예외와 함께 실패합니다.

```csharp
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open. Check if the Open timeout has expired.
      // If it has, set the state to HalfOpen. Another approach might be to
      // check for the HalfOpen state that had be set by some other operation.
      if (stateStore.LastStateChangedDateUtc + OpenToHalfOpenWaitTime < DateTime.UtcNow)
      {
        // The Open timeout has expired. Allow one operation to execute. Note that, in
        // this example, the circuit breaker is set to HalfOpen after being
        // in the Open state for some period of time. An alternative would be to set
        // this using some other approach such as a timer, test method, manually, and
        // so on, and check the state here to determine how to handle execution
        // of the action.
        // Limit the number of threads to be executed when the breaker is HalfOpen.
        // An alternative would be to use a more complex approach to determine which
        // threads or how many are allowed to execute, or to execute a simple test
        // method instead.
        bool lockTaken = false;
        try
        {
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken)
          if (lockTaken)
          {
            // Set the circuit breaker state to HalfOpen.
            stateStore.HalfOpen();

            // Attempt the operation.
            action();

            // If this action succeeds, reset the state and allow other operations.
            // In reality, instead of immediately returning to the Closed state, a counter
            // here would record the number of successful operations and return the
            // circuit breaker to the Closed state only after a specified number succeed.
            this.stateStore.Reset();
            return;
          }
          catch (Exception ex)
          {
            // If there's still an exception, trip the breaker again immediately.
            this.stateStore.Trip(ex);

            // Throw the exception so that the caller knows which exception occurred.
            throw;
          }
          finally
          {
            if (lockTaken)
            {
              Monitor.Exit(halfOpenSyncObject);
            }
          }
        }
      }
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```

작업을 보호하는 `CircuitBreaker` 개체를 사용하기 위해 응용 프로그램은 `CircuitBreaker` 클래스의 인스턴스를 생성하고 매개 변수로 수행할 작업을 지정해 `ExecuteAction` 메서드를 호출합니다. 응용 프로그램은 회로 차단기가 열려 있어 작업이 실패하는 경우 `CircuitBreakerOpenException` 예외를 발생시키도록 작성되어야 합니다. 다음 코드 예제를 참고하시기 바랍니다.

```csharp
var breaker = new CircuitBreaker();

try
{
  breaker.ExecuteAction(() =>
  {
    // Operation protected by the circuit breaker.
    ...
  });
}
catch (CircuitBreakerOpenException ex)
{
  // Perform some different action when the breaker is open.
  // Last exception details are in the inner exception.
  ...
}
catch (Exception ex)
{
  ...
}
```

## 관련 패턴 및 지침

이 패턴을 구현할 때 유용할 수 있는 패턴은 다음과 같습니다.

- [다시 시도 패턴][retry-pattern]. 이전에 실패한 작업을 투명하게 다시 시도해 서비스 또는 네트워크 리소스에 연결을 시도할 때 응용 프로그램이 예상되는 일시적인 장애를 처리할 수 있는 방법을 설명합니다.

- [상태 끝점 모니터링 패턴](health-endpoint-monitoring.md). 회로 차단기는 서비스가 노출하는 끝점에 요청을 전송해 서비스의 상태를 테스트할 수 있습니다. 서비스는 상태를 표시하는 정보를 반환해야 합니다.


[retry-pattern]: ./retry.md
