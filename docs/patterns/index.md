---
title: Cloud Design Patterns
description: Cloud Design Patterns for Microsoft Azure
keywords: Azure
ms.service: guidance
ms.author: pnp
---
# 클라우드 설계 패턴

클라우드 설계 패턴은 클라우드에서 신뢰할 수 있고, 확장이 가능하며, 안전한 응용 프로그램을 작성하는 데 유용합니다.

각각의 패턴은 패턴이 초점을 맞추는 문제, 패턴 적용을 위한 고려 사항 및 Microsoft Azure를 기반으로 하는 예제를 설명합니다. 대부분의 패턴에는 Azure에서 패턴의 구현 방법을 보여주는 코드 샘플이나 코드 조각이 포함되어 있습는데, 대부분의 패턴은 Azure 또는 다른 클라우드 플랫폼에서 호스팅하는 분산 시스템과 관련이 있습니다.

## 클라우드 개발의 당면 과제

<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/availability.md"><img src="_images/category/availability.svg" alt="Availability" /></a></td>
    <td>
        <h3><a href="./category/availability.md">가용성</a></h3>
        <p>가용성은 시스템이 기능하고 작동하는 시간의 비율로, 일반적으로 작동 시간의 백분율로 측정합니다. 가용성은 시스템 오류, 인프라 문제, 악의적인 공격, 시스템 부하에 영향을 받을 수 있습니다. 보통 클라우드 응용 프로그램은 사용자에게 서비스 수준 계약(SLA)을 제공합니다. 즉, 응용 프로그램은 가용성을 최대화하는 방식으로 설계되어야 합니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/data-management.md"><img src="_images/category/data-management.svg" alt="Data Management" /></a></td>
    <td>
        <h3><a href="./category/data-management.md">데이터 관리</a></h3>
        <p>데이터 관리는 클라우드 응용 프로그램의 주요 요소로, 대부분의 품질 특성에 영향을 미칩니다. 보통 데이터는 성능, 확장성 또는 가용성과 같은 이유로 인해 여러 위치와 다수의 서버에 호스팅되는데, 이로 인해 다양한 과제에 직면할 수 있습니다. 예를 들면 데이터 일관성을 유지하기 위해 보통 여러 위치에 보관되어 있는 데이터의 동기화가 필요하게 됩니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/design-implementation.md"><img src="_images/category/design-implementation.svg" alt="Design and Implementation" /></a></td>
    <td>
        <h3><a href="./category/design-implementation.md">설계와 구현</a></h3>
        <p>좋은 설계는 구성 요소 설계와 배포의 일관성과 일치성, 관리와 개발을 간소화하는 유지 관리성, 구성 요소와 하위 시스템을 다른 응용 프로그램과 다른 시나리오에 사용할 수 있는 재사용성과 같은 요인을 망라합니다. 설계와 구현 단계에서 이루어진 결정은 클라우드 호스팅 응용 프로그램과 서비스의 품질 및 총 소유 비용에 큰 영향을 미칩니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/messaging.md"><img src="_images/category/messaging.svg" alt="Messaging" /></a></td>
    <td>
        <h3><a href="./category/messaging.md">메시징</a></h3>
        <p>클라우드 응용 프로그램의 분산 특성으로 인해 구성 요소와 서비스를 느슨하게 결합하는 방식으로 연결해 확장성을 최대화하는 메시징 인프라가 필요합니다. 널리 사용되는 비동기식 메시징은 많은 이점을 제공하지만 메시지의 순서 지정, 포이즌 메시지 관리, 멱등성 등과 같은 과제도 안겨줍니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/management-monitoring.md"><img src="_images/category/management-monitoring.svg" alt="Management and Monitoring" /></a></td>
    <td>
        <h3><a href="./category/management-monitoring.md">관리와 모니터링</a></h3>
        <p>클라우드 응용 프로그램은 인프라 또는 일부 경우 운영 체제의 제어가 필요 없는 원격 데이터 센터에서 실행합니다. 따라서 관리와 모니터링이 온-프레미스 배포보다 어려울 수 있습니다. 응용 프로그램은 런타임 정보를 표시해 관리자와 운영자가 시스템을 관리하고 모니터링하는 데 사용할 수 있도록 지원할 뿐 아니라 응용 프로그램의 중단 또는 재배포 없이 변화하는 기업 요구 사항과 사용자 지정을 지원해야 합니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/performance-scalability.md"><img src="_images/category/performance-scalability.svg" alt="Performance and Scalability" /></a></td>
    <td>
        <h3><a href="./category/performance-scalability.md">성능과 확장성</a></h3>
        <p>성능은 지정된 시간 간격 이내에 동작을 실행하는 시스템 응답의 지표인 반면, 확장성은 성능에 영향을 미치지 않고 부하 증가를 처리하거나 사용 가능한 리소스를 쉽게 늘리는 시스템의 능력입니다. 클라우드 응용 프로그램을 사용하다보면 가변 워크로드 및 피크와 마주치기 마련입니다. 특히 다중 테넌트 시나리오에서는 가변 워크로드와 피크를 거의 예측할 수 없습니다. 대신 응용 프로그램은 한계 이내에서 규모를 확장해 수요의 피크를 충족하고 수요 감소 시 규모를 축소할 수 있어야 합니다. 확장성은 계산 인스턴스뿐 아니라 데이터 저장소, 메시징 인프라 등과 같은 다른 요소와도 관련됩니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/resiliency.md"><img src="_images/category/resiliency.svg" alt="Resiliency" /></a></td>
    <td>
        <h3><a href="./category/resiliency.md">복원력</a></h3>
        <p>복원력은 장애를 점진적으로 처리하고 복구하는 시스템의 능력입니다. 대부분의 응용 프로그램이 다중 테넌트가 되고, 공유 플랫폼 서비스를 사용하며, 리소스와 대역폭을 두고 경쟁하고, 인터넷상에서 통신하며, 범용 하드웨어에서 실행되는 클라우드 호스팅의 특성으로 인해 일시적인 장애와 더 영구적인 장애가 모두 발생할 가능성이 높습니다. 따라서 복원력을 유지하기 위해 장애 감지 및 신속하고 효율적인 복구가 필요합니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/security.md"><img src="_images/category/security.svg" alt="Security" /></a></td>
    <td>
        <h3><a href="./category/security.md">보안</a></h3>
        <p>보안은 설계된 용도를 벗어나는 악의적이거나 우발적인 동작을 방지하고 정보의 공개 또는 손실을 방지하는 시스템의 능력을 의미합입니다. 클라우드 응용 프로그램은 신뢰하는 온-프레미스 경계를 벗어나는 인터넷에 노출되고, 대중에 공개되며, 신뢰할 수 없는 사용자가 이용할 수 있습니다. 따라서 악의적인 공격으로부터  응용 프로그램을 보호하고, 승인된 사용자만으로 액세스를 제한하며, 민감한 데이터를 보호하는 방식으로 응용 프로그램을 설계하고 배포해야 합니다.</p>
    </td>
</tr>
</table>

## 패턴의 카탈로그

| 패턴 | 요약 |
| ------- | ------- |
| [캐시 배제](./cache-aside.md) | 주문형 데이터를 데이터 저장소에서 캐시로 로드합니다. |
| [회로 차단기](./circuit-breaker.md) | 원격 서비스 또는 리소스에 연결 시 해결 시간이 가변적인 장애를 처리합니다. |
| [CQRS](./cqrs.md) | 별도의 인터페이스를 사용해 데이터를 업데이트하는 작업에서 데이터를 읽는 작업을 분리합니다. |
| [보상 트랜잭션](./compensating-transaction.md) | (결과적으로 일관된 작업을 정의하는) 여러 단계를 거쳐 수행된 작업의 실행을 취소합니다. |
| [경쟁 소비자](./competing-consumers.md) | 다수의 동시 소비자가 동일한 메시징 채널에서 수신한 메시지를 처리할 수 있도록 지원합니다. |
| [계산 리소스 통합](./compute-resource-consolidation.md) | 다중 작업 또는 연산을 하나의 계산 단위로 통합합니다. |
| [이벤트 소싱](./event-sourcing.md) | 추가 전용 저장소를 사용해 도메인 내 데이터에 취해진 조치를 설명하는 전체 시리즈의 이벤트를 기록합니다. |
| [외부 구성 저장소](./external-configuration-store.md) | 구성 정보를 응용 프로그램 배포 패키지에서 중앙 집중식 위치로 옮깁니다. |
| [페더레이션 ID](./federated-identity.md) | 외부 ID 공급자에게 인증을 위임합니다. |
| [게이트키퍼](./gatekeeper.md) | 클라이언트와 응용 프로그램 또는 서비스 사이에 브로커로 작용하는 전용 호스트 인스턴스를 사용해 응용 프로그램과 서비스를 보호하고, 요청을 확인하고 삭제하며, 요청 및 요청 사이의 데이터를 통과시킵니다. |
| [상태 끝점 모니터링](./health-endpoint-monitoring.md) | 응용 프로그램에 기능 검사를 구현해 외부 도구가 일정한 간격으로 노출된 끝점을 통해 액세스할 수 있도록 지원합니다. |
| [인덱스 테이블](./index-table.md) | 쿼리가 자주 참조하는 데이터 저장소의 필드 인덱스를 생성합니다. |
| [리더 선정](./leader-election.md) | 하나의 인스턴스를 다른 인스턴스의 관리를 책임지는 리더로 선정해 배포 응용 프로그램 내에 있는 공동 작업 인스턴스 모음이 수행하는 작업을 조정합니다. |
| [구체화된 뷰](./materialized-view.md) | 데이터가 필요한 쿼리 작업에 맞게 포맷되지 않은 경우, 하나 이상의 데이터 저장소 내에 있는 데이터에 대해 미리 채워진 뷰를 생성합니다. |
| [파이프 및 필터](./pipes-and-filters.md) | 복잡한 처리를 수행하는 작업을 재사용할 수 있는 여러 개별 요소로 분할합니다. |
| [우선 순위 큐](./priority-queue.md) | 우선 순위가 높은 요청을 우선 순위가 낮은 요청보다 빨리 수신하고 처리하도록 서비스에 전송되는 요청의 우선 순위를 지정합니다. |
| [큐 기반 부하 평준화](./queue-based-load-leveling.md) | 작업과 작업이 호출하는 서비스 사이에 버퍼로 작용하는 큐를 사용해 간헐적으로 나타나는 과중한 부하를 평활화합니다. |
| [다시 시도](./retry.md) | 이전에 실패한 작업을 투명하게 다시 시도해 서비스 또는 네트워크 리소스에 연결을 시도할 때 응용 프로그램이 예상되는 일시적인 장애를 처리할 수 있도록 지원합니다. |
| [런타임 재구성](./runtime-reconfiguration.md) | 응용 프로그램을 재배포하거나 재시작하지 않고 재구성할 수 있도록 설계합니다. |
| [스케줄러 에이전트 감독자](./scheduler-agent-supervisor.md) | 서비스와 다른 원격 리소스의 분산 집합에서 실행하는 동작의 집합을 조정합니다. |
| [분할](./sharding.md) | 데이터 저장소를 여러 가로 파티션 또는 분할된 데이터베이스로 나눕니다. |
| [정적 콘텐츠 호스팅](./static-content-hosting.md) | 정적 콘텐츠를 클라이언트에 직접 전달할 수 있는 클라우드 기반 저장소 서비스에 배포합니다. |
| [제한](./throttling.md) | 응용 프로그램의 인스턴스, 개별 테넌트 또는 전체 서비스가 사용하는 리소스의 소비량을 제어합니다. |
| [발렛 키](./valet-key.md) | 특정 리소스 또는 서비스에 대한 제한적인 직접 액세스를 클라이언트에게 제공하는 토큰 또는 키를 사용합니다. |
