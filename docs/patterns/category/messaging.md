---
title: Messaging patterns
description: The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability. Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.
keywords: design pattern
author: dragon119
ms.author: pnp
ms.date: 03/24/2017
ms.topic: article
ms.service: guidance

pnp.series.title: Cloud Design Patterns
---

# 메시징 패턴

[!INCLUDE [header](../../_includes/header.md)]

클라우드 응용 프로그램의 분산 특성으로 인해 구성 요소와 서비스를 느슨하게 결합하는 방식으로 연결해 확장성을 최대화하는 메시징 인프라가 필요합니다. 널리 사용되는 비동기식 메시징은 많은 이점을 제공하지만 메시지의 순서 지정, 포이즌 메시지 관리, 멱등성 등과 같은 과제도 안겨줍니다.

| 패턴 | 요약 |
| ------- | ------- |
| [경쟁 소비자](../competing-consumers.md) | 다수의 동시 소비자가 동일한 메시징 채널에서 수신한 메시지를 처리할 수 있도록 지원합니다. |
| [파이프 및 필터](../pipes-and-filters.md) | 복잡한 처리를 수행하는 작업을 재사용할 수 있는 여러 개별 요소로 분할합니다. |
| [우선 순위 큐](../priority-queue.md) | 우선 순위가 높은 요청을 우선 순위가 낮은 요청보다 빨리 수신하고 처리하도록 서비스에 전송되는 요청의 우선 순위를 지정합니다. |
| [큐 기반 부하 평준화](../queue-based-load-leveling.md) | 작업과 작업이 호출하는 서비스 사이에 버퍼로 작용하는 큐를 사용해 간헐적으로 나타나는 과중한 부하를 평활화합니다. |
| [스케줄러 에이전트 감독자](../scheduler-agent-supervisor.md) | 서비스와 다른 원격 리소스의 분산 집합에서 실행하는 동작의 집합을 조정합니다. |
