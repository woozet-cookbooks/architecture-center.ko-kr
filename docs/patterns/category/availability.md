---
title: Availability patterns
description: Availability defines the proportion of time that the system is functional and working. It will be affected by system errors, infrastructure problems, malicious attacks, and system load. It is usually measured as a percentage of uptime. Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.
keywords: design pattern
author: dragon119
ms.author: pnp
ms.date: 03/24/2017
ms.topic: article
ms.service: guidance

pnp.series.title: Cloud Design Patterns
---

# 가용성 패턴

[!INCLUDE [header](../../_includes/header.md)]

가용성은 시스템이 기능하고 작동하는 시간의 비율을 정의합니다. 가용성은 시스템 오류, 인프라 문제, 악의적인 공격, 시스템 부하에 영향을 받을 수 있습니다.  일반적으로 가용성은 작동 시간의 백분율로 측정합니다. 보통 클라우드 응용 프로그램은 사용자에게 서비스 수준 계약(SLA)을 제공합니다. 즉, 응용 프로그램은 가용성을 최대화하는 방식으로 설계되고 구현되어야 합니다.

| 패턴 | 요약 |
| ------- | ------- |
| [상태 끝점 모니터링](../health-endpoint-monitoring.md) | 응용 프로그램에 기능 검사를 구현해 외부 도구가 일정한 간격으로 노출된 끝점을 통해 액세스할 수 있도록 지원합니다. |
| [큐 기반 부하 평준화](../queue-based-load-leveling.md) | 작업과 작업이 호출하는 서비스 사이에 버퍼로 작용하는 큐를 사용해 간헐적으로 나타나는 과중한 부하를 평활화합니다. |
| [제한](../throttling.md) | 응용 프로그램의 인스턴스, 개별 테넌트 또는 전체 서비스가 사용하는 리소스의 소비량을 제어합니다. |
