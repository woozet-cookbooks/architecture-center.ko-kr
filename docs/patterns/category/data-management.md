---
title: Data Management patterns
description: Data management is the key element of cloud applications, and influences most of the quality attributes. Data is typically hosted in different locations and across multiple servers for reasons such as performance, scalability or availability, and this can present a range of challenges. For example, data consistency must be maintained, and data will typically need to be synchronized across different locations.
keywords: design pattern
author: dragon119
ms.author: pnp
ms.date: 03/24/2017
ms.topic: article
ms.service: guidance

pnp.series.title: Cloud Design Patterns
---

# 데이터 관리 패턴

[!INCLUDE [header](../../_includes/header.md)]

데이터 관리는 클라우드 응용 프로그램의 주요 요소로, 대부분의 품질 특성에 영향을 미칩니다. 보통 데이터는 성능, 확장성 또는 가용성과 같은 이유로 인해 여러 위치와 다수의 서버에 호스팅되는데, 이로 인해 다양한 과제에 직면할 수 있습니다. 예를 들면 데이터 일관성을 유지하기 위해 보통 여러 위치에 보관되어 있는 데이터의 동기화가 필요하게 됩니다.

| 패턴 | 요약 |
| ------- | ------- |
| [캐시 배제](../cache-aside.md) | 주문형 데이터를 데이터 저장소에서 캐시로 로드합니다. |
| [CQRS](../cqrs.md) | 별도의 인터페이스를 사용해 데이터를 업데이트하는 작업에서 데이터를 읽는 작업을 분리합니다. |
| [이벤트 소싱](../event-sourcing.md) | 추가 전용 저장소를 사용해 도메인 내 데이터에 취해진 조치를 설명하는 전체 시리즈의 이벤트를 기록합니다. |
| [인덱스 테이블](../index-table.md) | 쿼리가 자주 참조하는 데이터 저장소의 필드 인덱스를 생성합니다. |
| [구체화된 뷰](../materialized-view.md) | 데이터가 필요한 쿼리 작업에 맞게 포맷되지 않은 경우, 하나 이상의 데이터 저장소 내에 있는 데이터에 대해 미리 채워진 뷰를 생성합니다. |
| [분할](../sharding.md) | 데이터 저장소를 여러 가로 파티션 또는 분할된 데이터베이스로 나눕니다. |
| [정적 콘텐츠 호스팅](../static-content-hosting.md) | 정적 콘텐츠를 클라이언트에 직접 전달할 수 있는 클라우드 기반 저장소 서비스에 배포합니다. |
| [발렛 키](../valet-key.md) | 특정 리소스 또는 서비스에 대한 제한적인 직접 액세스를 클라이언트에게 제공하는 토큰 또는 키를 사용합니다. |
