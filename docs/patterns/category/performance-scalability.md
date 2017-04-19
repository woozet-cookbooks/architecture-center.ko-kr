---
title: Performance and Scalability patterns
description: Performance is an indication of the responsiveness of a system to execute any action within a given time interval, while scalability is ability of a system either to handle increases in load without impact on performance or for the available resources to be readily increased. Cloud applications typically encounter variable workloads and peaks in activity. Predicting these, especially in a multi-tenant scenario, is almost impossible. Instead, applications should be able to scale out within limits to meet peaks in demand, and scale in when demand decreases. Scalability concerns not just compute instances, but other elements such as data storage, messaging infrastructure, and more.
keywords: design pattern
author: dragon119
ms.author: pnp
ms.date: 03/24/2017
ms.topic: article
ms.service: guidance

pnp.series.title: Cloud Design Patterns
---

# 성능과 확장성 패턴

[!INCLUDE [header](../../_includes/header.md)]

성능은 지정된 시간 간격 이내에 동작을 실행하는 시스템 응답의 지표인 반면, 확장성은 성능에 영향을 미치지 않고 부하 증가를 처리하거나 사용 가능한 리소스를 쉽게 늘리는 시스템의 능력을 의미합니다. 클라우드 응용 프로그램을 사용하다보면 가변 워크로드 및 피크와 마주치기 마련입니다. 특히 다중 테넌트 시나리오에서는 가변 워크로드와 피크를 거의 예측할 수 없습니다. 대신 응용 프로그램은 한계 이내에서 규모를 확장해 수요의 피크를 충족하고 수요 감소 시 규모를 축소할 수 있어야 합니다. 확장성은 계산 인스턴스뿐 아니라 데이터 저장소, 메시징 인프라 등과 같은 다른 요소와도 관련됩니다.

| 패턴 | 요약 |
| ------- | ------- |
| [캐시 배제](../cache-aside.md) | 주문형 데이터를 데이터 저장소에서 캐시로 로드합니다. |
| [CQRS](../cqrs.md) | 별도의 인터페이스를 사용해 데이터를 업데이트하는 작업에서 데이터를 읽는 작업을 분리합니다. |
| [이벤트 소싱](../event-sourcing.md) | 추가 전용 저장소를 사용해 도메인 내 데이터에 취해진 조치를 설명하는 전체 시리즈의 이벤트를 기록합니다. |
| [인덱스 테이블](../index-table.md) | 쿼리가 자주 참조하는 데이터 저장소의 필드 인덱스를 생성합니다. |
| [구체화된 뷰](../materialized-view.md) | 데이터가 필요한 쿼리 작업에 맞게 포맷되지 않은 경우, 하나 이상의 데이터 저장소 내에 있는 데이터에 대해 미리 채워진 뷰를 생성합니다. |
| [우선 순위 큐](../priority-queue.md) | 우선 순위가 높은 요청을 우선 순위가 낮은 요청보다 빨리 수신하고 처리하도록 서비스에 전송되는 요청의 우선 순위를 지정합니다. |
| [큐 기반 부하 평준화](../queue-based-load-leveling.md) | 작업과 작업이 호출하는 서비스 사이에 버퍼로 작용하는 큐를 사용해 간헐적으로 나타나는 과중한 부하를 평활화합니다.|
| [분할](../sharding.md) | 데이터 저장소를 여러 가로 파티션 또는 분할된 데이터베이스로 나눕니다. |
| [정적 콘텐츠 호스팅](../static-content-hosting.md) | 정적 콘텐츠를 클라이언트에 직접 전달할 수 있는 클라우드 기반 저장소 서비스에 배포합니다. |
| [제한](../throttling.md) | 응용 프로그램의 인스턴스, 개별 테넌트 또는 전체 서비스가 사용하는 리소스의 소비량을 제어합니다. |
