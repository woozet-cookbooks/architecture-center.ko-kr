---
title: 클라우드 디자인 패턴
description: Microsoft Azure에 대한 클라우드 디자인 패턴
keywords: Azure
ms.openlocfilehash: 0b564931fe027e42b3a6db1a5d6a207e6441e536
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847255"
---
# <a name="cloud-design-patterns"></a>클라우드 디자인 패턴

이러한 디자인 패턴은 클라우드에서 안정적이고 확장성 있는 안전한 응용 프로그램을 빌드하는 데 유용합니다.

각 패턴은 패턴이 해결하는 문제, 패턴을 적용하기 위한 고려 사항 및 Microsoft Azure 기반의 예제에 대해 설명합니다. 대부분의 패턴은 Azure에서 패턴을 구현하는 방법을 보여주는 코드 샘플 또는 코드 조각을 포함하고 있습니다. 그러나 대부분의 패턴은 Azure에 호스팅되든 다른 클라우드 플랫폼에 호스팅되든, 분산 시스템과 관련되어 있습니다.

## <a name="challenges-in-cloud-development"></a>클라우드 개발의 과제

<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/availability.md"><img src="_images/category/availability.svg" alt="Availability" /></a></td>
    <td>
        <h3><a href="./category/availability.md">가용성</a></h3>
        <p>가용성이란 시스템이 작동하는 시간의 비율을 말하며, 일반적으로 작동 시간의 백분율로 측정합니다. 시스템 오류, 인프라 문제, 악의적인 공격, 시스템 부하 등의 영향을 받을 수 있습니다.  클라우드 응용 프로그램은 일반적으로 사용자에게 Service Level Agreement(서비스 수준 약정)를 제공하므로 가용성이 최대화되도록 응용 프로그램을 디자인해야 합니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/data-management.md"><img src="_images/category/data-management.svg" alt="Data Management" /></a></td>
    <td>
        <h3><a href="./category/data-management.md">데이터 관리</a></h3>
        <p>데이터 관리는 클라우드 응용 프로그램의 핵심 요소이며 대부분의 품질 특성에 영향을 줍니다. 일반적으로 데이터는 성능, 확장성, 가용성 등의 이유로 여러 위치의 여러 서버에 호스팅되며, 이로 인해 다양한 문제가 발생할 수 있습니다. 예를 들어 데이터 일관성을 유지해야 하며, 일반적으로 여러 위치 간에 데이터를 동기화해야 합니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/design-implementation.md"><img src="_images/category/design-implementation.svg" alt="Design and Implementation" /></a></td>
    <td>
        <h3><a href="./category/design-implementation.md">디자인 및 구현</a></h3>
        <p>좋은 디자인이 되려면 구성 요소 디자인 및 배포의 일관성, 관리 및 배포 방법이 간단한 유지 관리 용이성, 구성 요소 및 하위 시스템을 다른 응용 프로그램 및 다른 시나리오에 사용할 수 있는 재사용 가능성 등의 요소를 고려해야 합니다. 디자인 및 구현 단계에서 결정된 사항은 클라우드에 호스팅되는 응용 프로그램과 서비스의 품질 및 총 소유 비용에 엄청난 영향을 미칩니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/messaging.md"><img src="_images/category/messaging.svg" alt="Messaging" /></a></td>
    <td>
        <h3><a href="./category/messaging.md">메시징</a></h3>
        <p>클라우드 응용 프로그램은 기본적으로 분산되기 때문에 구성 요소와 서비스를 연결하는 메시지 인프라가 필요하며, 가용성을 최대화할 수 있도록 느슨하게 결합된 인프라가 가장 이상적입니다. 널리 사용되는 비동기 메시지는 여러 가지 이점이 있지만 메시지 순서 지정, 포이즌 메시지 관리, 멱등성 같은 문제도 있습니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/management-monitoring.md"><img src="_images/category/management-monitoring.svg" alt="Management and Monitoring" /></a></td>
    <td>
        <h3><a href="./category/management-monitoring.md">관리 및 모니터링</a></h3>
        <p>클라우드 응용 프로그램은 개발자가 인프라 전체에 대한 제어 권한을 갖고 있지 않은 원격 데이터 센터에서 실행되거나, 또는 경우에 따라 운영 체제에서 실행됩니다. 그렇기 때문에 관리 및 모니터링이 온-프레미스 배포보다 더 어려울 수도 있습니다. 응용 프로그램은 관리자 및 운영자가 시스템 관리 및 모니터링에 사용할 수 있는 런타임 정보를 노출해야 할 뿐 아니라, 응용 프로그램을 중지하거나 다시 배포하지 않고 변화하는 비즈니스 요구 사항과 사용자 지정을 지원해야 합니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/performance-scalability.md"><img src="_images/category/performance-scalability.svg" alt="Performance and Scalability" /></a></td>
    <td>
        <h3><a href="./category/performance-scalability.md">성능 및 확장성</a></h3>
        <p>성능은 지정된 시간 간격 내에서 어떤 작업을 실행하는 시스템의 응답성을 나타내는 척도이고, 가용성은 성능에 영향을 주지 않고 부하 증가를 처리하거나 가용 리소스를 즉시 늘릴 수 있는 시스템 기능을 의미합니다. 클라우드 응용 프로그램은 일반적으로 작업에서 수시로 변하는 워크로드 및 최대 부하와 맞닥뜨리게 됩니다. 이러한 변수를 예측하기란, 특히 다중 테넌트 시나리오에서 예측하기란 거의 불가능합니다. 대신, 수요가 증가하면 응용 프로그램이 한도 내에서 규모 확장하고, 수요가 감소하면 규모 감축할 수 있어야 합니다. 확장성은 계산 인스턴스뿐 아니라 데이터 저장소, 메시지 인프라 등의 다른 요소에도 영향을 줍니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/resiliency.md"><img src="_images/category/resiliency.svg" alt="Resiliency" /></a></td>
    <td>
        <h3><a href="./category/resiliency.md">복원력</a></h3>
        <p>복원력은 중단 없이 오류를 처리하고 정상적으로 복구하는 시스템 기능입니다. 응용 프로그램이 다중 테넌트인 경우가 많고, 공유 플랫폼 서비스를 사용하고, 리소스 및 대역폭을 두고 경합하고, 인터넷을 통해 통신하고, 주로 하드웨어에서 실행되는 클라우드 호스팅의 특성 상, 일시적 오류와 영구적 오류가 모두 발생할 가능성이 좀 더 높습니다. 복원력을 유지하려면 오류를 감지하여 신속하고 효율적으로 복구하는 기능이 필요합니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="./category/security.md"><img src="_images/category/security.svg" alt="Security" /></a></td>
    <td>
        <h3><a href="./category/security.md">보안</a></h3>
        <p>보안은 설계된 용도를 벗어나는 악의적 작업 또는 실수로 인한 작업을 방지하고, 정보의 공개 또는 손실을 방지하는 시스템 기능입니다. 클라우드 응용 프로그램은 신뢰할 수 있는 온-프레미스 경계 외부에서 인터넷에 노출되고, 종종 일반에 공개되고, 신뢰할 수 없는 사용자가 사용할 수도 있습니다. 응용 프로그램은 악의적인 공격으로부터 보호하고 승인된 사용자만 액세스하도록 제한하고 중요한 데이터를 보호하는 방식으로 설계 및 배포해야 합니다.</p>
    </td>
</tr>
</table>

## <a name="catalog-of-patterns"></a>패턴 카탈로그

|                                패턴                                |                                                                                                         요약                                                                                                         |
|-----------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                     [특사](./ambassador.md)                     |                                                            소비자 서비스 또는 응용 프로그램을 대신하여 네트워크 요청을 전송하는 도우미 서비스를 만듭니다.                                                            |
|          [손상 방지 레이어](./anti-corruption-layer.md)          |                                                                  현대식 응용 프로그램과 레거시 시스템 사이에 외관 또는 어댑터 레이어를 구현합니다.                                                                  |
|         [프런트 엔드에 대한 백 엔드](./backends-for-frontends.md)         |                                                            특정 프런트 엔드 응용 프로그램 또는 인터페이스에서 사용할 별도의 백 엔드 서비스를 만듭니다.                                                             |
|                       [격벽](./bulkhead.md)                       |                                                        하나가 고장 나더라도 나머지는 정상적으로 작동하도록 응용 프로그램의 요소를 여러 풀에 격리합니다.                                                        |
|                    [Cache-Aside](./cache-aside.md)                    |                                                                                   필요할 때 데이터를 데이터 저장소에서 캐시로 로드                                                                                    |
|                [회로 차단기](./circuit-breaker.md)                |                                                     원격 서비스 또는 리소스에 연결할 때 해결하는 데 걸리는 시간이 유동적인 오류를 처리합니다.                                                     |
|                           [CQRS](./cqrs.md)                           |                                                           별도의 인터페이스를 사용하여 데이터를 업데이트하는 작업과 데이터를 읽는 작업을 분리합니다.                                                            |
|       [보정 트랜잭션](./compensating-transaction.md)       |                                                         여러 단계로 나뉘어 있지만 결국에는 일관적인 작업을 정의하는 일련의 단계에서 수행한 작업을 실행 취소합니다.                                                         |
|            [경쟁 소비자](./competing-consumers.md)            |                                                            여러 동시 소비자가 동일한 메시징 채널에 수신된 메시지를 처리할 수 있게 해 줍니다.                                                             |
| [계산 리소스 통합](./compute-resource-consolidation.md) |                                                                        여러 작업을 단일 계산 단위로 통합합니다.                                                                        |
|                 [이벤트 소싱](./event-sourcing.md)                 |                                                      추가 전용 저장소를 사용하여 도메인의 데이터에 대해 수행된 작업을 설명하는 일련의 이벤트 전체를 기록합니다.                                                      |
|   [외부 구성 저장소](./external-configuration-store.md)   |                                                           구성 정보를 응용 프로그램 배포 패키지에서 중앙 위치로 이동합니다.                                                           |
|             [페더레이션 ID](./federated-identity.md)             |                                                                                외부 ID 공급자에게 인증을 위임합니다.                                                                                |
|                     [게이트 키퍼](./gatekeeper.md)                     | 클라이언트와 응용 프로그램 또는 서비스 간 브로커 역할을 하며, 요청을 검사 및 정리하고, 요청 및 데이터를 전달하는 전용 호스트 인스턴스를 사용하여 응용 프로그램 및 서비스를 보호합니다. |
|            [게이트웨이 집계](./gateway-aggregation.md)            |                                                                     게이트웨이를 사용하여 여러 개별 요청을 단일 요청으로 집계합니다.                                                                      |
|             [게이트웨이 오프로딩](./gateway-offloading.md)             |                                                                         공유 또는 특수 서비스 기능을 게이트웨이 프록시에 오프로드합니다.                                                                         |
|                [게이트웨이 라우팅](./gateway-routing.md)                |                                                                              단일 엔드포인트를 사용하여 요청을 여러 서비스에 라우팅합니다.                                                                               |
|     [상태 엔드포인트 모니터링](./health-endpoint-monitoring.md)     |                                              외부 도구가 노출된 엔드포인트를 통해 주기적으로 액세스할 수 있는 기능 검사를 응용 프로그램 내부에 구현합니다.                                               |
|                    [인덱스 테이블](./index-table.md)                    |                                                                쿼리에서 자주 참조하는 데이터 저장소의 필드에 대한 인덱스를 만듭니다.                                                                 |
|                [리더 선택](./leader-election.md)                |   인스턴스 중 하나를 다른 인스턴스를 관리하는 리더로 선택하여 분산된 응용 프로그램의 공동 작업 인스턴스 컬렉션이 수행하는 작업을 조정합니다.    |
|              [구체화된 뷰](./materialized-view.md)              |                                        데이터가 필요한 쿼리 작업에 대해 이상적으로 포맷되지 않은 경우 하나 이상의 데이터 저장소에 있는 데이터에 대한 미리 채워진 뷰를 생성합니다.                                        |
|              [파이프 및 필터](./pipes-and-filters.md)              |                                                        복잡한 처리를 수행하는 작업을 재사용 가능한 일련의 별도 요소로 분류합니다.                                                        |
|                 [우선 순위 큐](./priority-queue.md)                 |                                 우선 순위가 높은 요청을 우선 순위가 낮은 요청보다 먼저 받아서 처리하도록 서비스로 전송된 요청의 우선 순위를 지정합니다.                                  |
|      [큐 기반 부하 평준화](./queue-based-load-leveling.md)      |                                               작업 그리고 그 작업이 일시적인 높은 부하를 부드럽게 처리하기 위해 호출하는 서비스 사이에서 버퍼 역할을 하는 큐를 사용합니다.                                               |
|                          [다시 시도](./retry.md)                          |               이전에 실패한 작업을 투명하게 다시 시도하여 서비스 또는 네트워크 리소스에 연결하려 할 때 응용 프로그램을 사용하여 예상된 일시적 오류를 처리합니다.                |
|     [Scheduler 에이전트 감독자](./scheduler-agent-supervisor.md)     |                                                              서비스 및 기타 원격 리소스의 분산된 집합에서 일련의 작업을 조정합니다.                                                               |
|                       [분할](./sharding.md)                       |                                                                           데이터 저장소를 수평 파티션 또는 분할 집합으로 나눕니다.                                                                            |
|                        [사이드카](./sidecar.md)                        |                                                    격리 및 캡슐화를 제공하는 별도의 프로세스 또는 컨테이너에 응용 프로그램 구성 요소를 배포합니다.                                                     |
|         [정적 콘텐츠 호스팅](./static-content-hosting.md)         |                                                          정적 콘텐츠를 클라이언트에 직접 제공할 수 있는 클라우드 기반 저장소 서비스에 배포합니다.                                                           |
|                      [스트랭글러](./strangler.md)                      |                                            특정 기능을 새로운 응용 프로그램 및 서비스로 점진적으로 교체하여 레거시 시스템을 단계적으로 마이그레이션합니다.                                            |
|                     [제한](./throttling.md)                     |                                                 응용 프로그램 인스턴스, 개별 테넌트 또는 서비스 전체의 리소스 사용량을 제어합니다.                                                 |
|                      [발레 키](./valet-key.md)                      |                                                        클라이언트에 특정 리소스 또는 서비스에 대한 제한된 직접 액세스를 제공하는 토큰 또는 키를 사용합니다.                                                        |

