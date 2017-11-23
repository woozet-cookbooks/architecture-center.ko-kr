---
title: Resiliency patterns
description: Resiliency is the ability of a system to gracefully handle and recover from failures. The nature of cloud hosting, where applications are often multi-tenant, use shared platform services, compete for resources and bandwidth, communicate over the Internet, and run on commodity hardware means there is an increased likelihood that both transient and more permanent faults will arise. Detecting failures, and recovering quickly and efficiently, is necessary to maintain resiliency.
keywords: design pattern
author: dragon119
ms.author: pnp
ms.date: 03/24/2017
ms.topic: article
ms.service: guidance

pnp.series.title: Cloud Design Patterns
---

# 복원력 패턴

[!INCLUDE [header](../../_includes/header.md)]

복원력은 장애를 점진적으로 처리하고 복구하는 시스템의 능력입니다. 대부분의 응용 프로그램이 다중 테넌트가 되고, 공유 플랫폼 서비스를 사용하며, 리소스와 대역폭을 두고 경쟁하고, 인터넷상에서 통신하며, 범용 하드웨어에서 실행되는 클라우드 호스팅의 특성으로 인해 일시적인 장애와 더 영구적인 장애가 모두 발생할 가능성이 높습니다. 따라서 복원력을 유지하기 위해 장애 감지 및 신속하고 효율적인 복구가 필요합니다.

| 패턴 | 요약 |
| ------- | ------- |
| [회로 차단기](../circuit-breaker.md) | 원격 서비스 또는 리소스에 연결 시 해결 시간이 가변적인 장애를 처리합니다. |
| [보상 트랜잭션](../compensating-transaction.md) | (결과적으로는 일관된 작업을 정의하는) 여러 단계를 거쳐 수행된 작업의 실행을 취소합니다. |
| [상태 끝점 모니터링](../health-endpoint-monitoring.md) | 응용 프로그램에 기능 검사를 구현해 외부 도구가 일정한 간격으로 노출된 끝점을 통해 액세스할 수 있도록 지원합니다. |
| [리더 선정](../leader-election.md) | 하나의 인스턴스를 다른 인스턴스의 관리를 책임지는 리더로 선정해 배포 응용 프로그램 내에 있는 공동 작업 인스턴스 모음이 수행하는 동작을 조정합니다. |
| [큐 기반 부하 평준화](../queue-based-load-leveling.md) | 작업과 작업이 호출하는 서비스 사이에 버퍼로 작용하는 큐를 사용해 간헐적으로 나타나는 과중한 부하를 평활화합니다. |
| [다시 시도](../retry.md) | 이전에 실패한 작업을 투명하게 다시 시도해 서비스 또는 네트워크 리소스에 연결을 시도할 때 응용 프로그램이 예상되는 일시적인 장애를 처리할 수 있도록 지원합니다. |
| [스케줄러 에이전트 감독자](../scheduler-agent-supervisor.md) | 서비스와 다른 원격 리소스의 분산 집합에서 실행하는 동작의 집합을 조정합니다. |
