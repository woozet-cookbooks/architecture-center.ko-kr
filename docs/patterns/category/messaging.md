---
title: "메시지 패턴"
description: "클라우드 응용 프로그램은 기본적으로 분산되기 때문에 구성 요소와 서비스를 연결하는 메시지 인프라가 필요하며, 가용성을 최대화할 수 있도록 느슨하게 결합된 인프라가 가장 이상적입니다. 널리 사용되는 비동기 메시지는 여러 가지 이점이 있지만 메시지 순서 지정, 포이즌 메시지 관리, 멱등성 같은 문제도 있습니다."
keywords: "디자인 패턴"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 6151f7f76fc7b3a953988122db75bdc25b49811f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="messaging-patterns"></a>메시지 패턴

[!INCLUDE [header](../../_includes/header.md)]

클라우드 응용 프로그램은 기본적으로 분산되기 때문에 구성 요소와 서비스를 연결하는 메시지 인프라가 필요하며, 가용성을 최대화할 수 있도록 느슨하게 결합된 인프라가 가장 이상적입니다. 널리 사용되는 비동기 메시지는 여러 가지 이점이 있지만 메시지 순서 지정, 포이즌 메시지 관리, 멱등성 같은 문제도 있습니다.

| 패턴 | 요약 |
| ------- | ------- |
| [경쟁 소비자](../competing-consumers.md) | 여러 동시 소비자가 동일한 메시징 채널에 수신된 메시지를 처리할 수 있게 해 줍니다. |
| [파이프 및 필터](../pipes-and-filters.md) | 복잡한 처리를 수행하는 작업을 재사용 가능한 일련의 별도 요소로 분류합니다. |
| [우선 순위 큐](../priority-queue.md) | 우선 순위가 높은 요청을 우선 순위가 낮은 요청보다 먼저 받아서 처리하도록 서비스로 전송된 요청의 우선 순위를 지정합니다. |
| [큐 기반 부하 평준화](../queue-based-load-leveling.md) | 작업 그리고 그 작업이 일시적인 높은 부하를 부드럽게 처리하기 위해 호출하는 서비스 사이에서 버퍼 역할을 하는 큐를 사용합니다. |
| [Scheduler 에이전트 감독자](../scheduler-agent-supervisor.md) | 서비스 및 기타 원격 리소스의 분산된 집합에서 일련의 작업을 조정합니다. |