---
title: Design and Implementation patterns
description: Good design encompasses factors such as consistency and coherence in component design and deployment, maintainability to simplify administration and development, and reusability to allow components and subsystems to be used in other applications and in other scenarios. Decisions made during the design and implementation phase have a huge impact on the quality and the total cost of ownership of cloud hosted applications and services.
keywords: design pattern
author: dragon119
ms.author: pnp
ms.date: 03/24/2017
ms.topic: article
ms.service: guidance

pnp.series.title: Cloud Design Patterns
---

# 설계와 구현 패턴

[!INCLUDE [header](../../_includes/header.md)]

좋은 설계는 구성 요소 설계와 배포의 일관성과 일치성, 관리와 개발을 간소화하는 유지 관리성, 구성 요소와 하위 시스템을 다른 응용 프로그램과 다른 시나리오에 사용할 수 있는 재사용성과 같은 요인을 망라합니다. 설계와 구현 단계에서 이루어진 결정은 클라우드 호스팅 응용 프로그램과 서비스의 품질 및 총 소유 비용에 큰 영향을 미칩니다.

| 패턴 | 요약 |
| ------- | ------- |
| [CQRS](../cqrs.md) | 별도의 인터페이스를 사용해 데이터를 업데이트하는 작업에서 데이터를 읽는 작업을 분리합니다. |
| [계산 리소스 통합](../compute-resource-consolidation.md) | 다중 작업 또는 연산을 하나의 계산 단위로 통합합니다. |
| [외부 구성 저장소](../external-configuration-store.md) | 구성 정보를 응용 프로그램 배포 패키지에서 중앙 집중식 위치로 옮깁니다. |
| [리더 선정](../leader-election.md) | 하나의 인스턴스를 다른 인스턴스의 관리를 책임지는 리더로 선정해 배포 응용 프로그램 내에 있는 공동 작업 인스턴스 모음이 수행하는 작업을 조정합니다. |
| [파이프 및 필터](../pipes-and-filters.md) | 복잡한 처리를 수행하는 작업을 재사용할 수 있는 여러 개별 요소로 분할합니다. |
| [런타임 재구성](../runtime-reconfiguration.md) | 응용 프로그램을 재배포하거나 재시작하지 않고 재구성할 수 있도록 설계합니다. |
| [정적 콘텐츠 호스팅](../static-content-hosting.md) | 정적 콘텐츠를 클라이언트에 직접 전달할 수 있는 클라우드 기반 저장소 서비스에 배포합니다. |
