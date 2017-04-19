---
title: Management and Monitoring patterns
description: Cloud applications run in in a remote datacenter where you do not have full control of the infrastructure or, in some cases, the operating system. This can make management and monitoring more difficult than an on-premises deployment. Applications must expose runtime information that administrators and operators can use to manage and monitor the system, as well as supporting changing business requirements and customization without requiring the application to be stopped or redeployed.
keywords: design pattern
author: dragon119
ms.author: pnp
ms.date: 03/24/2017
ms.topic: article
ms.service: guidance

pnp.series.title: Cloud Design Patterns
---

# M관리와 모니터링 패턴

[!INCLUDE [header](../../_includes/header.md)]

클라우드 응용 프로그램은 인프라 또는 일부 경우 운영 체제의 제어가 필요 없는 원격 데이터 센터에서 실행합니다. 따라서 관리와 모니터링이 온-프레미스 배포보다 어려울 수 있습니다. 응용 프로그램은 런타임 정보를 표시해 관리자와 운영자가 시스템을 관리하고 모니터링하는 데 사용할 수 있도록 지원할 뿐 아니라 응용 프로그램의 중단 또는 재배포 없이 변화하는 기업 요구 사항과 사용자 지정을 지원해야 합니다.

| 패턴 | 요약 |
| ------- | ------- |
| [외부 구성 저장소](../external-configuration-store.md) | 구성 정보를 응용 프로그램 배포 패키지에서 중앙 집중식 위치로 옮깁니다. |
| [상태 끝점 모니터링](../health-endpoint-monitoring.md) | 응용 프로그램에 기능 검사를 구현해 외부 도구가 일정한 간격으로 노출된 끝점을 통해 액세스할 수 있도록 지원합니다. |
| [런타임 재구성](../runtime-reconfiguration.md) | 응용 프로그램을 재배포하거나 재시작하지 않고 재구성할 수 있도록 설계합니다. |
