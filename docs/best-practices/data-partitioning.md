---
title: Data partitioning guidance
description: Guidance for how to separate partitions to be managed and accessed separately.
author: dragon119
ms.service: guidance
ms.topic: article
ms.date: 07/13/2016
ms.author: pnp

pnp.series.title: Best Practices
---
# 데이터 분할
[!INCLUDE [header](../_includes/header.md)]

많은 대규모 솔루션에서 데이터는 별도로 관리 및 액세스할 수 있는 별도의 파티션으로 나뉩니다. 분할 전략을 주의 깊게 선택하여 장점을 극대화하고 부작용을 최소화해야 합니다. 분할은 확장성을 개선하고, 경합을 줄이며, 성능을 최적화합니다. 분할의 다른 이점으로는 사용 패턴에 따라 데이터를 나누는 메커니즘을 지원할 수 있다는 점입니다. 예를 들어, 저렴한 데이터 저장소에 오래된 비활성(콜드) 데이터를 보관할 수 있습니다. 

## 왜 파티션 데이터인가?
대다수 클라우드 응용 프로그램과 서비스는 데이터 저장과 검색 기능을 갖추고 있습니다. 응용 프로그램이 사용하는 데이터 저장소 설계는 시스템의 성능, 처리량 및 확장성과 밀접한 관련이 있습니다. 대규모 시스템에서 일반적으로 적용되는 한 가지 기법은 데이터를 별도의 파티션으로 나누는 것입니다. 

> 이 지침에서 사용되는 *분할*이라는 용어는 데이터를 별도의 데이터 저장소로 물리적으로 나누는 프로세스를 말합니다. 여기서 분할은 SQL Server 테이블의 분할과는 다른 개념입니다.
>
>

데이터를 분할하면 다양한 이점이 있습니다. 예를 들면, 다음과 같은 이점을 위해 데이터 분할을 적용할 수 있습니다. 

* **확장성 향상**. 단일 데이터베이스 시스템을 강화하다 보면 결국 물리적 하드웨어의 한계에 도달하게 됩니다. 각기 별도의 서버에서 호스팅되는 파티션 여러 개에서 데이터를 나눌 경우, 시스템은 거의 무제한적으로 규모가 확장됩니다.
* **성능 개선**. 각 파티션에서 데이터 액세스 작업은 적은 양의 데이터에서 진행됩니다. 데이터가 적절한 방식으로 분할되기만 하면, 분할은 시스템의 효율성을 높여 줄 수 있습니다. 2개 이상의 파티션에 영향을 주는 작업은 동시에 실행될 수 있습니다. 각 파티션은 해당 파티션을 사용하는 응용 프로그램 근처에 위치하여 네트워크 대기 시간을 최소화할 수 있습니다.
* **가용성 개선**. 여러 서버에서 데이터를 분리하면 단일 장애 지점을 차단해 줍니다. 서버에 장애가 발생하거나 예정된 유지 관리 작업이 실행중인 경우, 해당 파티션의 데이터만 사용할 수 없는 상태가 됩니다. 다른 파티션에서는 작업을 계속할 수 있습니다. 파티션 수가 늘어나면 사용할 수 없는 데이터의 비율이 줄어 단일 서버 장애와 관련된 영향이 감소합니다. 각 파티션을 복제하면 작업에 영향을 주는 단일 파티션 장애 발생 가능성이 더 줄어들 수 있습니다. 또한 가용성 요구사항이 낮은 저가치 데이터에서 계속 많이 사용해야 하는 중요 데이터를 분리할 수도 있습니다.
* **보안 강화**. 데이터와 분할 방식 특성에 따라 중요한 데이터와 중요하지 않은 데이터를 다른 파티션으로 분리한 다음, 다른 서버나 데이터 저장소로 분리할 수도 있습니다. 그리고 나면 특히 보안이 중요한 데이터에 맞게 최적화될 수 있습니다.
* **운영 유연성 지원**. 분할은 그 자체로 작업을 미세 조정할 수 있는 다양한 가능성을 열어 주므로, 관리 효율을 극대화하고 비용을 최소화할 수 있습니다. 예를 들어, 관리, 모니터링, 백업, 복원 등 각 파티션의 데이터 중요도에 따라 관리 작업별 전략을 정의할 수 있습니다.
* **데이터 저장소와 사용 패턴 일치**. 분할을 통해 데이터 저장소가 지원하는 기본 제공 기능과 비용에 따라 각 파티션을 다양한 유형의 데이터 저장소에서 배포할 수 있습니다. 예를 들어, 대형 이진 데이터를 blob 데이터 저장소에 저장하면서도, 더 많은 구조적 데이터를 문서 데이터베이스에 보관할 수 있습니다. 자세한 내용은 패턴 및 사례 가이드의 [Polyglot 솔루션 빌드]와 Microsoft 웹사이트의 [확장성이 우수한 솔루션의 데이터 액세스:. SQL, NoSQL 및 Polyglot 지속성]을 참조하십시오.

일부 시스템은 이점 이외에 비용을 감안하여 분할을 구현하지 않습니다. 이에 대한 일반적인 이유는 다음과 같습니다. 

•	많은 데이터 저장소 시스템이 여러 파티션에서 조인을 지원하지 않아 분할된 시스템에서 참조 무결성을 유지하기 어려울 수 있습니다. 흔히 응용 프로그램 코드(분할 계층)에서 조인과 무결성 검사를 구현해야 하기 때문에 추가 입출력이 필요하고 응용 프로그램 복잡성이 발생할 수 있습니다.

•	파티션 유지는 언제나 사소한 작업이 아닙니다. 데이터가 변동이 심한 시스템에서는 정기적으로 파티션 균형을 다시 맞춰 경합과 핫스폿을 줄여야 합니다.

•	일반적인 도구는 자연히 분할된 데이터에서 작동하지 않습니다.


## 파티션 설계
데이터 분할 방식으로는 수평, 수직 및 기능이 있습니다. 선택하는 전략은 데이터 분할 이유와 데이터를 사용하는 응용 프로그램 및 서비스 요구사항에 따라 달라집니다. 

> [!참고]
> 이 지침에 설명된 분할 체계 방식은 기본 데이터 저장소 기술과 무관합니다. 또한 그 근거와 NoSQL 데이터베이스를 포함한 많은 유형의 데이터 저장소에 적용될 수 있습니다. 
>
>

### 분할 전략
일반적인 데이터 분할 전략 세 가지는 다음과 같습니다. 

* **수평 분할**(흔히 샤딩*[sharding]*이라고 함). 이 전략에서는 각 파티션이 그 자체로 데이터 저장소이지만, 모든 파티션의 체계가 동일합니다. 각 파티션은 *샤드(shard)*로 알려져 있고 전자 상거래 응용 프로그램에서 특정 고객 집합의 모든 주문과 같은 특정 데이터 하위 집합을 보관합니다.
* **수직 분할**. 이 전략에서 각 파티션은 데이터 저장소의 여러 항목 필드의 하위 집합을 저장합니다. 이 필드는 사용 패턴에 따라 나뉩니다. 예를 들어, 자주 액세스하는 필드를 하나의 수직 파티션에 배치하고 자주 액세스하지 않는 필드는 다른 파티션에 배치할 수 있습니다.
* **기능 분할**. 이 전략에서는 시스템에서 바인딩된 각 컨텍스트가 데이터를 사용하는 방법에 따라 데이터를 집계합니다. 예를 들어, 청구 및 제품 재고 관리를 위해 별도의 비즈니스 기능을 구현한 전자 상거래 시스템은 송장 데이터와 제품 재고 데이터를 각기 다른 파티션에 저장할 수 있습니다.

여기에 나와 있는 이 세 가지 전략을 조합하여 사용할 수 있다는 점을 잘 알고 있어야 합니다. 이들 전략은 서로 배타적이지 않으므로, 분할 체계를 설계할 때 모든 전략을 고려하는 것이 좋습니다. 예를 들어, 데이터를 샤드로 나눈 후 수직 분할을 사용해 각 샤드에서 데이터를 추가로 세분화할 수 있습니다. 마찬가지로, 기능 파티션의 데이터를 샤드로 분할할 수 있습니다(수직으로도 분할 가능). 

그렇지만, 각 전략의 요구사양이 다양하여 많은 충돌 문제가 발생할 수 있습니다. 전체 데이터 처리 성능 목표에 부합하는 분할 체계를 설계할 때에는 이 모든 사항을 평가하고 균형을 맞춰야 합니다. 다음 섹션에서는 각 전략에 대해 자세히 알아봅니다. 

### 수평 분할(샤딩)
그림 1은 수평 분할 또는 샤딩에 대한 개요를 나타냅니다. 이 예에서 제품 재고 데이터는 제품 키를 기준으로 샤드(shard)로 나뉩니다. 각 샤드에는 알파벳 순으로 정렬된 연속 범위의 샤드 키((A-G 및 H-Z) 데이터가 저장되어 있습니다. 

![Horizontally partitioning (sharding) data based on a partition key](./images/data-partitioning/DataPartitioning01.png)

*그림 1. 파티션 키 기준의 수평 분할(샤딩) 데이터*

샤딩(Sharding)을 통해 더 많은 컴퓨터에서 부하를 분산시켜 경합을 줄이고 성능을 개선할 수 있습니다. 시스템을 확장하려면 추가 서버에서 실행되는 샤드를 추가합니다. 

이 분할 전략을 구현할 때 가장 중요한 요소는 샤딩 키를 선택하는 것입니다. 시스템을 작동한 후에는 키를 변경하기 어려울 수 있습니다. 키는 데이터가 분할되어 워크로드가 샤드에서 가능한 한 균등하도록 보장해야 합니다. 

다른 샤드에는 유사한 분량의 데이터가 포함되어 있어서는 안 됩니다. 그 보다 더 중요한 고려 사항은 요청 개수의 균형을 유지하는 것입니다. 일부 샤드는 매우 클 수 있지만, 각 항목은 적은 수의 액세스 작업의 제목입니다. 물론, 작은 샤드가 있을 수 있고, 각 항목은 훨씬 더 자주 액세스됩니다. 단일 샤드는 샤드를 호스팅하는 데 사용되고 있는 데이터 저장소의 규모 제한(용량 및 리소스 처리 측면)을 초과해서는 안 됩니다. 

샤딩 체계를 사용할 경우, 성능과 가용성에 영향을 줄 수 있는 핫스폿(또는 핫 파티션)을 만들지 말아야 합니다. 예를 들어, 고객 이름의 첫 번째 문자가 아닌 고객 식별자의 해시를 사용할 경우, 일반적인 머리글자와 일반적이지 않은 머리글자로 인한 배포 불균형을 방지해 줍니다. 이것은 여러 파티션에서 더 고르게 데이터를 배포하도록 해주는 일반적인 기법입니다. 

샤딩 키를 선택할 때에는 대규모 샤드를 작은 조각으로 분할하거나, 작은 샤드를 큰 파티션으로 합치거나, 파티션 집합에 저장된 데이터를 설명하는 체계를 변경하는 데 따른 향후 요구사항을 최소화해야 합니다. 이 작업은 매우 시간 소모적일 수 있고, 이 작업을 실행하는 동안 하나 이상의 샤드를 오프라인으로 전환해야 할 수도 있습니다. 

샤드를 복제하는 경우, 온라인 상태를 유지해야 하는 복제본도 있고 분할, 병합 또는 재구성해야 하는 복제본도 있습니다. 그렇지만, 이 시스템은 재구성을 실행하는 동안 이 샤드에서 실행할 수 있는 작업을 제한해야 합니다. 예를 들어, 복제본의 데이터는 읽기 전용으로 표시되어 샤드가 재구성되는 동안 발생할 수 있는 불일치 범위를 제한할 수 있습니다. 

>이 많은 고려 사항들을 비롯해, 수평 분할을 구현하는 데이터 저장소를 설계할 때의 모범 사례 기법에 대한 자세한 정보와 지침은 [샤딩 패턴]을 참조하십시오. 
>
>

### 수직 분할
수직 분할의 가장 일반적인 용도는 가장 자주 액세스하는 항목 가져오기와 관련된 I/O 및 성능 비용을 절감하는 데 있습니다. 그림 2에는 수직 분할의 예가 나와 있습니다. 이 예에서 각 데이터 항목별 속성은 파티션별로 다릅니다. 제품 이름, 설명, 가격 정보 등 자주 액세스되는 데이터를 저장한 파티션도 있고, 재고 수량과 마지막 주문 날짜를 저장한 파티션도 있습니다. 

![Vertically partitioning data by its pattern of use](./images/data-partitioning/DataPartitioning02.png)

*그림 2. 사용 패턴별 수직 분할 데이터*

이 예에서 제품 세부 정보를 고객에게 표시할 때 응용 프로그램은 제품 이름, 설명, 가격에 대해 정기적으로 쿼리합니다. 제조업체에서 제품을 마지막으로 주문했을 당시 재고 수준과 날짜는 보통 함께 사용되기 때문에 별도의 파티션에 보관됩니다. 

이 분할 체계는 비교적 유동성이 낮은 데이터(제품 이름, 설명, 가격)가 보다 동적인 데이터(재고 수준과 마지막 주문 날짜)와 분리되는 추가적인 이점이 있습니다. 응용 프로그램은 자주 액세스되는 유동성이 낮은 데이터를 메모리에 캐시하는 것이 유용할 수 있습니다. 

이 분할 전략의 또 다른 시나리오로는 중요한 데이터의 보안을 극대화하는 것입니다. 예를 들어, 신용카드 번호와 해당 카드의 보안 확인 번호를 분리된 파티션에 저장함으로써 보안을 극대화할 수 있습니다. 

또한 수직 분할은 데이터에 필요한 동시 액세스 분량을 줄여줄 수도 있습니다. 

> 수직 분할은 데이터 저장소 내에서 엔터티 수준에서 작동하면서 엔터티를 일부 정규화하여 *넓은* 항목에서 *좁은* 항목 집합으로 분해합니다. 이 분할 방식은 HBase 및 Cassandra와 같은 열 기반 데이터 저장소에 적합합니다. 열 컬렉션의 데이터가 변경될 가능성이 적으면, SQL Server에서 열 저장소를 사용하는 것을 고려해 볼 수도 있습니다. 
>
>

### 기능 분할
시스템이 응용 프로그램에서 고유한 각 비즈니스 영역이나 서비스별로 바인딩된 컨텍스트를 식별할 수 있는 경우, 기능 분할 기법은 분리 및 데이터 액세스 성능을 높여 줍니다. 기능 분할의 또 다른 일반적인 용도는 보고 목적으로 사용되는 읽기 전용 데이터에서 읽기-쓰기 데이터를 분리하는 것입니다. 그림 3은 재고 데이터가 고객 데이터와 분리되는 기능 분할에 대한 개요를 나타냅니다. 

![Functionally partitioning data by bounded context or subdomain](./images/data-partitioning/DataPartitioning03.png)

*그림 3. 바운딩된 컨텍스트 또는 하위 도메인별 기능 분할 데이터*

이 분할 전략은 시스템의 여러 부분에서 데이터 액세스 경합을 줄이는 데 도움이 될 수 있습니다. 

## 확장성을 고려한 파티션 설계
확장성을 극대화하는 방향으로 데이터를 배포하기 위해서는 각 파티션별로 크기와 워크로드를 고려하고 균형을 맞추는 것이 필수적입니다. 그렇지만, 단일 파티션 저장소의 크기 조정 한도를 초과하지 않으면서 데이터를 분할해야 합니다. 

확장성을 고려하면서 파티션을 설계할 때에는 다음 단계를 따릅니다. 

1.	응용 프로그램을 분석하여 각 쿼리에서 반환한 결과 집합 크기, 액세스 빈도, 내재된 대기 시간, 서버 측 계산 처리 요구사항과 같은 데이터 액세스 패턴을 파악합니다. 많은 경우 일부 주요 엔터티가 처리 리소스의 대부분을 요구하게 됩니다.
2.	이 분석을 사용하여 데이터의 크기 및 워크로드와 같은 현재 및 향후 확장성 목표를 결정합니다. 그런 다음, 여러 파티션에서 데이터를 배포하여 확장성 목표를 충족합니다. 균등한 배포를 위해서는 반드시 수평 분할 전략에서 적절한 샤드 키를 선택해야 합니다. 자세한 내용은 [샤딩 패턴]을 참조하십시오.
3.	각 파티션에서 사용할 수 있는 리소스는 데이터 크기와 처리량 측면에서 확장성 요구사항을 처리하기에 충분해야 합니다. 예를 들어, 파티션을 호스팅하고 있는 노드로 인해 저장소 공간의 용량, 처리 능력, 지원되는 네트워크 대역폭에 엄격한 제한이 걸릴 수도 있습니다. 데이터 저장소와 처리 요구사항이 이 제한을 초과할 경우, 분할 전략을 정비하거나 데이터를 추가로 분할할 수도 있습니다. 예를 들어, 한 가지 확장성 접근 방식은 로깅 데이터를 핵심 응용 프로그램 기능과 분리하는 것입니다. 이 작업을 하려면 별도의 데이터 저장소를 사용하여 전체 데이터 저장소 요구사항이 노드의 확장 제한을 초과하지 않도록 해야 합니다. 데이터 저장소의 전체 개수가 노드 제한을 초과할 경우, 별도의 저장소 노드를 사용해야 할 수도 있습니다.
4.	데이터가 예상대로 배포되고 파티션이 자체 부하를 처리할 수 있는지 확인하기 위해 용도에 따라 시스템을 모니터링합니다. 사용 현황은 분석에 따라 예상되는 사용 현황과 일치하지 않을 수도 있습니다. 이 경우에 파티션 균형을 재조정할 수도 있습니다. 이 작업에 실패하면, 시스템의 일부 부분을 재설계하여 필요한 균형을 확보할 수도 있습니다.

일부 클라우드 환경은 인프라 경계 관점에서 리소스를 할당합니다. 선택한 경계 한도는 데이터 저장소, 처리 능력 및 대역폭 측면에서 데이터 볼륨을 예상대로 증가시킬 수 있는 여지가 있어야 합니다. 

예를 들어 Azure 테이블 저장소를 사용하는 경우, 사용 중인 샤드는 요청을 처리하기 위해 단일 파티션에서 사용할 수 있는 것보다 더 많은 리소스가 필요할 수 있습니다. (특정 기간 동안 단일 파티션에서 처리 가능한 요청 볼륨에는 제한이 있습니다. 자세한 내용은 Microsoft 웹사이트에서 [Azure 저장소 확장성 및 성능 목표] 페이지를 참조하십시오.

 이 경우에 샤드를 다시 분할하여 부하를 분산시켜야 할 수도 있습니다. 이 테이블의 전체 크기나 처리량이 저장소 계정의 용량을 초과하는 경우, 추가 저장소 계정을 만들고 이 계정에서 테이블을 분산시켜야 합니다. 저장소 계정 개수가 1회 구독 시 사용 가능한 계정 개수를 초과하는 경우, 여러 번 구독해야 합니다. 

## 쿼리 성능을 고려한 파티션 설계
쿼리 성능은 흔히 소량 데이터 집합을 사용하고 병렬 쿼리를 실행하여 강화할 수 있습니다. 각 파티션은 전체 데이터 집합 중 적은 비율을 포함해야 합니다. 볼륨을 줄이면 쿼리 성능이 개선됩니다. 그렇지만, 분할은 데이터베이스를 설계하고 구성할 수 있는 대체 방안이 아닙니다. 예를 들어, 관계형 데이터베이스를 사용하고 있는 경우 필요한 인덱스를 지정해야 합니다. 

쿼리 성능을 고려하면서 파티션을 설계할 때에는 다음 단계를 따릅니다. 

1.	다음과 같이 응용 프로그램 요구사항과 성능을 점검합니다.
•	비즈니스 요구사항을 사용하여 항상 빠르게 실행해야 하는 필수 쿼리를 결정합니다.

•	시스템을 모니터링하여 천천히 실행되는 모든 쿼리를 식별합니다.

•	가장 자주 실행되는 쿼리를 설정합니다. 각 쿼리의 단일 인스턴스는 비용을 최소화하면서도 리소스의 누적 사용량은 클 수 있습니다. 이들 쿼리에 의해 검색된 별도의 파티션이나 캐시 안에 따로 분리시키는 것이 유용합니다.

2.	다음과 같이 성능을 저하시키는 데이터를 분할합니다.
•	쿼리 응답 시간이 목표 내에 들어오도록 각 파티션 크기를 제한합니다.

•	수평 분할을 구현하는 경우 응용 프로그램이 파티션을 쉽게 찾을 수 있도록 샤드 키를 설계합니다. 그러면 쿼리가 모든 파티션을 스캔할 필요가 없습니다.

•	파티션의 위치를 고려합니다. 가능한 한 응용 프로그램과 이 응용 프로그램에  액세스하는 사용자와 지리적으로 가까운 곳에 있는 파티션에 데이터를 보관해야 합니다.

3.	엔터티에 처리량 및 쿼리 성능 요구사항이 있으면, 해당 엔터티 기반의 기능 분할을 사용합니다. 하지만 그래도 이 요구사항에 계속 부합하지 않으면, 수평 분할도 적용합니다. 대다수의 경우 단일 분할 전략으로 충분하지만, 경우에 따라서는 두 가지 전략을 조합하는 것이 더 효율적입니다.
4.	성능을 개선하려면 여러 파티션에서 동시에 실행되는 비동기 쿼리를 사용하는 것이 좋습니다.

## 가용성을 고려한 파티션 설계
분할 데이터를 통해 응용 프로그램의 가용성을 개선하려면 전체 데이터 집합에 단일 장애 지점이 포함되지 않고 데이터 집합의 개별 하위 집합이 독립적으로 관리될 수 있도록 보장해야 합니다. 필수 데이터를 포함하는 파티션 복제도 가용성을 개선할 수 있습니다. 

파티션을 설계하고 구현할 때에는 가용성에 영향을 주는 다음 요소를 고려해야 합니다. 

* **비즈니스 작업에서 데이터의 중요도**. 일부 데이터에는 송장 세부 정보 및 은행 거래 내역과 같은 중요한 비즈니스 정보가 포함되기도 하고, 로그 파일, 성능 추적 등 중요도가 낮은 운영 데이터가 포함되기도 합니다. 각 데이터 형식을 확인한 후 다음 사항을 고려해야 합니다.
o	적절한 백업 계획이 지원되는 고가용성 파티션에 중요 데이터 저장.

o	각 데이터 집합의 서로 다른  중요도에 맞게 별도의 관리 및 모니터링 메커니즘이나 절차 수립. 적절한 빈도로 함께 백업할 수 있도록 동일한 파티션에 중요도가 동일한 데이터 배치. 예를 들어, 은행 거래용 데이터를 저장한 파티션은 로깅이나 추적 정보를 저장한 파티션보다 더 자주 백업해야 합니다.

* **개별 파티션을 관리하는 방법**. 독립적인 관리와 유지 관리를 지원하는 파티션을 설계하면 여러 장점이 있습니다. 예:
o	파티션에 오류가 발생하면, 다른 파티션의 데이터에 액세스하는 응용 프로그램의 인스턴스에 영향을 주지 않고 독립적으로 복구할 수 있습니다.

o	지리적 영역별로 데이터를 분할하면 예정된 유지 관리 작업이 각 위치별로 사용량이 적은 시간에 진행됩니다. 이 기간 중에 예정된 유지 관리를 완료하지 못할 정도로 파티션이 너무 커서는 안 됩니다.

* **여러 파티션에서 중요 데이터의 복제 여부**. 이 전략은 일관성 문제를 발생시킬 수 있지만 가용성과 성능을 높여 줍니다. 파티션의 데이터에 적용된 변경 내용이 모든 복제본과 동기화되기까지는 다소 시간이 걸립니다. 이 기간 중에 다른 파티션은 다른 데이터 값을 포함합니다.

## 분할이 설계와 개발에 미치는 영향 파악
분할을 사용하면 시스템 설계와 개발에 따른 복잡성이 커집니다. 시스템이 처음에 단일 파티션만을 포함하더라도 시스템 설계의 기본 부분으로서 분할을 고려해야 합니다. 분할을 나중에 추가하게 된 경우, 시스템에 성능 및 확장성 문제가 발생하면 이미 실행간 시스템을 유지해야 하기 때문에 복잡성이 커집니다.

시스템이 이 환경에서 분할을 포함하도록 업데이트하는 경우, 데이터 액세스 논리를 수정해야 합니다. 또한 대량의 기존 데이터를 마이그레이션하여 여러 파티션에서 배포할 수도 있고, 이와 동시에 사용자는 흔히 시스템을 계속 사용할 수 있을 것으로 예상합니다. 

경우에 따라서는 초기 데이터 집합이 작으며 단일 서버로 쉽게 처리할 수 있기 때문에 분할이 중요하게 고려되지 않습니다. 이것은 초기의 크기 이상으로 확장할 것으로 예상되지 않는 시스템의 경우에 적용되지만, 많은 상업용 시스템은 사용자 수 증가에 따라 확장해야 합니다. 이러한 확장에는 보통 데이터 볼륨 증가가 동반됩니다. 

또한 분할이 항상 대형 데이터 저장소의 기능이 아님을 이해해야 합니다. 예를 들어, 작은 데이터 저장소를 수백 개의 동시 클라이언트가 과도하게 액세스할 수도 있습니다. 이 상황에서 데이터를 분할하면 경합을 줄이고 처리량을 개선할 수 있습니다.

데이터 분할 체계를 설계할 때에는 다음 사항에 유의해야 합니다. 

* **가능한 경우, 각 파티션에 가장 많이 사용되는 데이터베이스 작업용 데이터를 보관하여 교차 파티션 액세스 작업을 최소화합니다**. 여러 파티션에서 쿼리를 진행하면 단일 파티션의 쿼리보다 시간이 더 소요될 수 있지만, 하나의 쿼리 집합에서 파티션을 최적화하면 다른 쿼리 집합에 부정적인 영향을 줄 수 있습니다. 어쩔 수 없이 여러 파티션에서 쿼리해야 하는 경우 응용 프로그램 내에서 병렬 쿼리를 실행하고 결과를 집계하여 쿼리 시간을 최소화합니다. 이 접근 방식은 하나의 쿼리에서 결과를 가져와 다음 쿼리에서 사용하는 경우 등 경우에 따라 불가능할 수 있습니다.
* **쿼리가 우편 번호 테이블이나 제품 목록과 같은 비교적 정적 참조 데이터를 사용하는 경우, 모든 파디션에서 이 데이터를 복제하여 다른 파티션에서 별도의 조회 작업 요구사항을 줄여야 합니다**. 또한, 이 접근 방식은 참조 데이터가 전체 시스템에서 많은 트래픽을 발생시킬 수 있는 "핫" 데이터 집합이 될 가능성을 낮출 수 있습니다.  그렇지만, 발생 가능한 모든 변경 내용을 이 참조 데이터와 동기화할 때 관련 비용이 추가됩니다.
* **가능한 경우, 수직 및 기능 파티션에서 참조 무결성의 요구사항을 최소화합니다**. 이 체계에서 응용 프로그램은 데이터가 업데이트 및 소비되면 전체 파티션에서 직접 참조 무결성을 유지해야 합니다. 여러 파티션에서 데이터를 조인해야 하는 쿼리는 동일한 파티션 내에서만 데이터를 조인하는 쿼리보다 더 느리게 실행됩니다. 왜냐하면, 일반적으로 응용 프로그램은 키와 외래 키 순서로 연속 쿼리를 실행하기 때문입니다. 그 대신, 관련 데이터의 복제나 비정규화를 고려해야 합니다. 교차 파티션 조인이 필요한 쿼리 시간을 최소화하려면, 파티션에서 쿼리를 실행하고 응용 프로그램 내에서 데이터를 조인해야 합니다.
* **분할 체계가 파티션 전체의 데이터 일관성에 미칠 수 있는 영향을 고려합니다.** 철저한 일관성이 실제로 꼭 필요한지 여부를 평가합니다. 그 대신, 클라우드에서 일반적인 접근 방식은 최종 일관성을 구현하는 것입니다. 각 파티션에서 데이터는 개별적으로 업데이트되고, 응용 프로그램 논리는 업데이트를 모두 완료하도록 보장합니다. 또한, 결국 일관된 작업을 실행하는 동안 데이터를 쿼리하면서 발생할 수 있는 불일치를 처리합니다. 최종 일관성에 대한 자세한 내용은 [데이터 일관성 프라이머]를 참조하십시오.
* **쿼리가 올바른 파티션을 찾는 방법을 고려합니다**. 쿼리가 모든 파티션을 스캔하여 필요한 데이터를 찾아야 하면, 여러 병렬 쿼리를 실행하고 있더라도 성능에 상당한 영향을 주게 됩니다. 수직 및 기능 분할 전략에서 사용되는 쿼리는 당연히 파티션을 지정할 수 있습니다. 그렇지만, 수평 분할(샤딩)은 모든 샤드의 체계가 동일하기 때문에 항목을 찾기 어렵게 만들기도 합니다. 샤딩에서 일반적인 해결책은 데이터의 특정 항목에서 샤드 위치를 찾는 데 사용할 수 있는 맵을 유지하는 것입니다. 이 맵은 응용 프로그램의 샤딩 논리에서 구현되거나 데이터 저장소가 투명한 샤딩을 지원하는 경우 이 데이터 저장소를 통해 유지됩니다.
* **수평 분할 전략을 사용할 때에는 정기적으로 샤드 균형을 다시 맞춰야 합니다**. 이 작업을 통해 크기 및 워크로드별로 데이터를 균일하게 배포하여 핫스폿을 최소화하고, 쿼리 성능을 극대화하고, 물리적 저장소 제한을 잘 처리할 수 있습니다. 그렇지만, 이 작업은 흔히 사용자 지정 도구나 프로세스를 사용해야 하기 때문에 복잡합니다.
* **각 파티션을 복제하는 경우, 오류 발생을 차단해 주는 추가 보호 장치가 마련됩니다**. 단일 복제본에 오류가 발생하면, 쿼리가 실행 중인 복사본으로 전달될 수 있습니다.
* **분할 전략의 물리적 제한에 도달하면, 확장성을 다양한 수준으로 넓혀야 할 수도 있습니다**. 예를 들어, 분할이 데이터베이스 수준인 경우 여러 데이터베이스에서 파티션을 찾거나 복제해야 합니다. 분할이 이미 데이터베이스 수준이고 물리적 제한에 대한 문제가 생기면 여러 호스팅 계정에서 파티션을 찾거나 복제해야 합니다.
* **여러 파티션에서 데이터에 액세스하는 트랜잭션을 차단합니다**. 일부 데이터 저장소는 단일 파티션에서 데이터를 찾을 때에만 데이터를 수정하는 작업에서 트랜잭션 일관성과 무결성을 구현합니다. 여러 파티션에서 트랜잭션 지원이 필요한 경우, 대다수 분할 시스템은 기본 지원을 제공하지 않기 때문에 응용 프로그램 논리의 일부로서 이것을 구현해야 합니다.

모든 데이터 저장소는 운영 관리 및 모니터링 활동을 필요로 합니다. 이 작업은 데이터 로드에서부터 데이터 백업 및 복원, 데이터 재구성, 시스템의 올바르고 효율적인 실행 보장에 이르기까지 포괄적입니다. 

운영 관리에 영향을 주는 다음 요소를 고려해야 합니다. 

* **데이터를 분할할 때 적절한 관리와 운영 작업을 구현하는 방법**. 이 작업에는 백업과 복원, 데이터 보관, 시스템 모니터링 등 여타 관리 작업이 포함됩니다. 예를 들어, 백업과 복원 작업 중에 논리 일관성을 유지하는 일이 문제가 될 수 있습니다.
* **데이터를 여러 파티션에 로드하고 다른 소스에서 오는 새 데이터를 추가하는 방법**. 일부 도구와 유틸리티는 데이터를 올바른 파티션에 로드하는 작업 등 샤딩된 데이터 작업을 지원하지 않을 수도 있습니다. 즉, 새 도구와 유틸리티를 만들거나 가져와야 할 수도 있습니다.
* **정기적으로 데이터를 보관하고 삭제하는 방법**. 과도한 파티션 증가를 방지하려면, 정기적으로(보통월 단위로) 데이터를 보관하고 삭제해야 합니다. 각기 다른 보관 체계에 맞게 데이터를 변환해야 할 수도 있습니다.
* **데이터 무결성 문제를 찾는 방법**. 한 파티션의 데이터가 다른 파티션의 누락된 정보를 참조하는 경우 등 데이터 무결성 문제를 찾는 정기적인 프로세스를 실행하는 것을 고려해 보십시오. 이 프로세스에서 이 문제를 자동으로 수정하거나 작업자에게 경고를 보내 문제를 직접 수정하도록 할 수 있습니다. 예를 들어, 전자 상거래 응용 프로그램에서 주문 정보는 하나의 파티션에 저장될 수 있지만 각 주문을 구성하는 품목은 다른 파티션에 저장될 수 있습니다. 주문 프로세스는 데이터를 다른 파티션에 추가해야 합니다. 이 프로세스가 실패하면, 해당 주문이 없는 줄 항목에 저장될 수 있습니다.

다른 데이터 저장소 기술은 일반적으로 분할을 지원하는 자체 기능을 내장하고 있습니다. 다음 섹션에서는 Azure 응용 프로그램에서 많이 사용되는 데이터 저장소의 실행 옵션에 대해 간략하게 알아봅니다. 또한, 이 기능을 최대한 활용할 수 있는 응용 프로그램 설계 시 고려 사항에 대해서도 설명합니다. 

## Azure SQL Database에 적합한 분할 전략
Azure SQL Database는 클라우드에서 실행되는 관계형 DaaS(database-as-a-service) 데이터베이스로서 Microsoft SQL Server를 기반으로 합니다. 관계형 데이터베이스는 정보를 테이블로 나누고, 각 테이블에 엔터티에 대한 정보를 일련의 행으로 보관합니다. 각 행에는 개별 엔터티 필드에 대한 데이터가 있는 열이 포함되어 있습니다. Microsoft 웹사이트의 [Azure SQL Database란?] 페이지에는 SQL 데이터베이스 만들기와 사용에 대한 세부 설명서가 나와 있습니다. 

## Elastic Database를 사용한 수평 분할
단일 SQL 데이터베이스는 저장할 수 있는 데이터의 용량에 제한이 있습니다. 처리량은 구조적인 요소와 지원되는 많은 수의 동시 연결로 인해 제약을 받습니다. SQL Database의 Elastic Database 기능은 SQL 데이터베이스의 수평 크기 조정을 지원합니다. Elastic Database를 사용하면, 데이터를 샤드로 분할하여 여러 SQL 데이터베이스에 분산시킬 수 있습니다. 또한 증가와 감소를 처리해야 하는 데이터 볼륨으로서 샤드를 추가 또는 제거할 수도 있습니다. Elastic Database를 사용하면 데이터베이스에서 부하를 분산시켜 경합을 줄이는 데도 도움이 됩니다. 

> [!참고]
Elastic Database는 Azure SQL Database의 Federations 기능을 대체할 수 있는 기능입니다. 기존 SQL Database Federation 설치는 Federations 마이그레이션 유틸리티를 사용하여 Elastic Database로 마이그레이션할 수 있습니다. 그렇지 않고, 시나리오가 Elastic Database가 제공하는 기능에 잘 부합하지 않는 경우 자체 샤딩 메커니즘을 구현하는 방법도 있습니다. 
>
>

각 샤드는 SQL 데이터베이스로서 구현됩니다. 샤드는 2개 이상의 데이터 집합 (*샤들렛[shardlet]*이라고 함)을 가질 수 있습니다. 각 데이터베이스는 각각 갖고 있는 샤들렛에 대해 설명된 메타데이터를 유지합니다. 샤들렛은 단일 데이터 항목이거나 동일한 샤들렛 키를 공유하는 항목 그룹입니다. 예를 들어, 다중 테넌트 응용 프로그램의 데이터를 샤딩하는 경우, 샤들렛 키는 테넌트 ID일 수 있고, 지정된 테넌트의 모든 데이터는 동일한 샤들렛의 일부로 저장될 수 있습니다. 다른 테넌트의 데이터는 각기 다른 샤들렛에 저장됩니다. 

데이터 집합을 샤들렛 키와 연결하는 것은 프로그래머의 책임입니다. 별도의 SQL 데이터베이스는 전역 샤드 맵 관리자 역할을 합니다. 이 데이터베이스에는 시스템에 있는 모든 샤드와 샤들렛 목록이 들어 있습니다. 데이터에 액세스하는 클라이언트 응용 프로그램은 먼저 전역 샤드 맵 관리자 데이터베이스에 연결되어 샤드 맵(샤드와 샤들렛 열거) 복사본을 가져온 후 로컬에서 캐시합니다. 

그런 다음 이 응용 프로그램은 이 정보를 사용해 데이터 요청을 적절한 샤드에 전달합니다. 이 기능은 NuGet 패키지로 출시된 Azure SQL Database Elastic Database Client Library에 포함된 일련의 API 뒤에 숨겨져 있습니다. Microsoft 웹사이트의 [Elastic Database 기능 개요] 페이지에는 Elastic Database에 대한 보다 포괄적인 소개가 나와 있습니다.

> [!NOTE]
> You can replicate the global shard map manager database to reduce latency and improve availability. If you implement the database by using one of the Premium pricing tiers, you can configure active geo-replication to continuously copy data to databases in different regions. Create a copy of the database in each region in which users are based. Then configure your application to connect to this copy to obtain the shard map.
>
> An alternative approach is to use Azure SQL Data Sync or an Azure Data Factory pipeline to replicate the shard map manager database across regions. This form of replication runs periodically and is more suitable if the shard map changes infrequently. Additionally, the shard map manager database does not have to be created by using a Premium pricing tier.
>
>

Elastic Database provides two schemes for mapping data to shardlets and storing them in shards:

* A **list shard map** describes an association between a single key and a shardlet. For example, in a multitenant system, the data for each tenant can be associated with a unique key and stored in its own shardlet. To guarantee privacy and isolation (that is, to prevent one tenant from exhausting the data storage resources available to others), each shardlet can be held within its own shard.

![Using a list shard map to store tenant data in separate shards](./images/data-partitioning/PointShardlet.png)

*Figure 4. Using a list shard map to store tenant data in separate shards*

* A **range shard map** describes an association between a set of contiguous key values and a shardlet. In the multitenant example described previously, as an alternative to implementing dedicated shardlets, you can group the data for a set of tenants (each with their own key) within the same shardlet. This scheme is less expensive than the first (because tenants share data storage resources), but it also creates a risk of reduced data privacy and isolation.

![Using a range shard map to store data for a range of tenants in a shard](./images/data-partitioning/RangeShardlet.png)

*Figure 5. Using a range shard map to store data for a range of tenants in a shard*

Note that a single shard can contain the data for several shardlets. For example, you can use list shardlets to store data for different non-contiguous tenants in the same shard. You can also mix range shardlets and list shardlets in the same shard, although they will be addressed through different maps in the global shard map manager database. (The global shard map manager database can contain multiple shard maps.) Figure 6 depicts this approach.

![Implementing multiple shard maps](./images/data-partitioning/MultipleShardMaps.png)

*Figure 6. Implementing multiple shard maps*

The partitioning scheme that you implement can have a significant bearing on the performance of your system. It can also affect the rate at which shards have to be added or removed, or the rate at which data must be repartitioned across shards. Consider the following points when you use Elastic Database to partition data:

* Group data that is used together in the same shard, and avoid operations that need to access data that's held in multiple shards. Keep in mind that with Elastic Database, a shard is a SQL database in its own right, and Azure SQL Database does not support cross-database joins (which have to be performed on the client side). Remember also that in Azure SQL Database, referential integrity constraints, triggers, and stored procedures in one database cannot reference objects in another. Therefore, don't design a system that has dependencies between shards. A SQL database can, however, contain tables that hold copies of reference data frequently used by queries and other operations. These tables do not have to belong to any specific shardlet. Replicating this data across shards can help remove the need to join data that spans databases. Ideally, such data should be static or slow-moving to minimize the replication effort and reduce the chances of it becoming stale.

  > [!NOTE]
  > Although SQL Database does not support cross-database joins, you can perform cross-shard queries with the Elastic Database API. These queries can transparently iterate through the data held in all the shardlets that are referenced by a shard map. The Elastic Database API breaks cross-shard queries down into a series of individual queries (one for each database) and then merges the results. For more information, see the page [Multi-shard querying] on the Microsoft website.
  >
  >
* The data stored in shardlets that belong to the same shard map should have the same schema. For example, don't create a list shard map that points to some shardlets containing tenant data and other shardlets containing product information. This rule is not enforced by Elastic Database, but data management and querying becomes very complex if each shardlet has a different schema. In the example just cited, a good is solution is to create two list shard maps: one that references tenant data and another that points to product information. Remember that the data belonging to different shardlets can be stored in the same shard.

  > [!NOTE]
  > The cross-shard query functionality of the Elastic Database API depends on each shardlet in the shard map containing the same schema.
  >
  >
* Transactional operations are only supported for data that's held within the same shard, and not across shards. Transactions can span shardlets as long as they are part of the same shard. Therefore, if your business logic needs to perform transactions, either store the affected data in the same shard or implement eventual consistency. For more information, see the [Data consistency primer].
* Place shards close to the users that access the data in those shards (in other words, geo-locate the shards). This strategy helps reduce latency.
* Avoid having a mixture of highly active (hotspots) and relatively inactive shards. Try to spread the load evenly across shards. This might require hashing the shardlet keys.
* If you are geo-locating shards, make sure that the hashed keys map to shardlets held in shards stored close to the users that access that data.
* Currently, only a limited set of SQL data types are supported as shardlet keys; *int, bigint, varbinary,* and *uniqueidentifier*. The SQL *int* and *bigint* types correspond to the *int* and *long* data types in C#, and have the same ranges. The SQL *varbinary* type can be handled by using a *Byte* array in C#, and the SQL *uniqueidentier* type corresponds to the *Guid* class in the .NET Framework.

As the name implies, Elastic Database makes it possible for a system to add and remove shards as the volume of data shrinks and grows. The APIs in the Azure SQL Database Elastic Database client library enable an application to create and delete shards dynamically (and transparently update the shard map manager). However, removing a shard is a destructive operation that also requires deleting all the data in that shard.

If an application needs to split a shard into two separate shards or combine shards, Elastic Database provides a separate split-merge service. This service runs in a cloud-hosted service (which must be created by the developer) and migrates data safely between shards. For more information, see the topic [Scaling using the Elastic Database split-merge tool] on the Microsoft website.

## Partitioning strategies for Azure Storage
Azure storage provides three abstractions for managing data:

* Table storage, which implements scalable structure storage. A table contains a collection of entities, each of which can include a set of properties and values.
* Blob storage, which supplies storage for large objects and files.
* Storage queues, which support reliable asynchronous messaging between applications.

Table storage and blob storage are essentially key-value stores that are optimized to hold structured and unstructured data respectively. Storage queues provide a mechanism for building loosely coupled, scalable applications. Table storage, blob storage, and storage queues are created within the context of an Azure storage account. Storage accounts support three forms of redundancy:

* **Locally redundant storage**, which maintains three copies of data within a single datacenter. This form of redundancy protects against hardware failure but not against a disaster that encompasses the entire datacenter.
* **Zone-redundant storage**, which maintains three copies of data spread across different datacenters within the same region (or across two geographically close regions). This form of redundancy can protect against disasters that occur within a single datacenter, but cannot protect against large-scale network disconnects that affect an entire region. Note that zone-redundant storage is currently only currently available for block blobs.
* **Geo-redundant storage**, which maintains six copies of data: three copies in one region (your local region), and another three copies in a remote region. This form of redundancy provides the highest level of disaster protection.

Microsoft has published scalability targets for Azure Storage. For more information, see the page [Azure Storage scalability and performance targets] on the Microsoft website. Currently, the total storage account capacity cannot exceed 500 TB. (This includes the size of data that's held in table storage and blob storage, as well as outstanding messages that are held in storage queue).

The maximum request rate (assuming a 1-KB entity, blob, or message size) is 20 KBps. If your system is likely to exceed these limits, consider partitioning the load across multiple storage accounts. A single Azure subscription can create up to 100 storage accounts. However, note that these limits might change over time.

## Partitioning Azure table storage
Azure table storage is a key-value store that's designed around partitioning. All entities are stored in a partition, and partitions are managed internally by Azure table storage. Each entity that's stored in a table must provide a two-part key that includes:

* **The partition key**. This is a string value that determines in which partition Azure table storage will place the entity. All entities with the same partition key will be stored in the same partition.
* **The row key**. This is another string value that identifies the entity within the partition. All entities within a partition are sorted lexically, in ascending order, by this key. The partition key/row key combination must be unique for each entity and cannot exceed 1 KB in length.

The remainder of the data for an entity consists of application-defined fields. No particular schemas are enforced, and each row can contain a different set of application-defined fields. The only limitation is that the maximum size of an entity (including the partition and row keys) is currently 1 MB. The maximum size of a table is 200 TB, although these figures might change in the future. (Check the page [Azure Storage scalability and performance targets] on the Microsoft website for the most recent information about these limits.)

If you are attempting to store entities that exceed this capacity, then consider splitting them into multiple tables. Use vertical partitioning to divide the fields into the groups that are most likely to be accessed together.

Figure 7 shows the logical structure of an example storage account (Contoso Data) for a fictitious e-commerce application. The storage account contains three tables: Customer Info, Product Info, and Order Info. Each table has multiple partitions.

In the Customer Info table, the data is partitioned according to the city in which the customer is located, and the row key contains the customer ID. In the Product Info table, the products are partitioned by product category, and the row key contains the product number. In the Order Info table, the orders are partitioned by the date on which they were placed, and the row key specifies the time the order was received. Note that all data is ordered by the row key in each partition.

![The tables and partitions in an example storage account](./images/data-partitioning/TableStorage.png)

*Figure 7. The tables and partitions in an example storage account*

> [!NOTE]
> Azure table storage also adds a timestamp field to each entity. The timestamp field is maintained by table storage and is updated each time the entity is modified and written back to a partition. The table storage service uses this field to implement optimistic concurrency. (Each time an application writes an entity back to table storage, the table storage service compares the value of the timestamp in the entity that's being written with the value that's held in table storage. If the values are different, it means that another application must have modified the entity since it was last retrieved, and the write operation fails. Don't modify this field in your own code, and don't specify a value for this field when you create a new entity.
>
>

Azure table storage uses the partition key to determine how to store the data. If an entity is added to a table with a previously unused partition key, Azure table storage creates a new partition for this entity. Other entities with the same partition key will be stored in the same partition.

This mechanism effectively implements an automatic scale-out strategy. Each partition is stored on a single server in an Azure datacenter to help ensure that queries that retrieve data from a single partition run quickly. However, different partitions can be distributed across multiple servers. Additionally, a single server can host multiple partitions if these partitions are limited in size.

Consider the following points when you design your entities for Azure table storage:

* The selection of partition key and row key values should be driven by the way in which the data is accessed. Choose a partition key/row key combination that supports the majority of your queries. The most efficient queries retrieve data by specifying the partition key and the row key. Queries that specify a partition key and a range of row keys can be completed by scanning a single partition. This is relatively fast because the data is held in row key order. If queries don't specify which partition to scan, the partition key might require Azure table storage to scan every partition for your data.

  > [!TIP]
  > If an entity has one natural key, then use it as the partition key and specify an empty string as the row key. If an entity has a composite key comprising two properties, select the slowest changing property as the partition key and the other as the row key. If an entity has more than two key properties, use a concatenation of properties to provide the partition and row keys.
  >
  >
* If you regularly perform queries that look up data by using fields other than the partition and row keys, consider implementing the [index table pattern].
* If you generate partition keys by using a monotonic increasing or decreasing sequence (such as "0001", "0002", "0003", and so on) and each partition only contains a limited amount of data, then Azure table storage can physically group these partitions together on the same server. This mechanism assumes that the application is most likely to perform queries across a contiguous range of partitions (range queries) and is optimized for this case. However, this approach can lead to hotspots focused on a single server because all insertions of new entities are likely to be concentrated at one end or the other of the contiguous ranges. It can also reduce scalability. To spread the load more evenly across servers, consider hashing the partition key to make the sequence more random.
* Azure table storage supports transactional operations for entities that belong to the same partition. This means that an application can perform multiple insert, update, delete, replace, or merge operations as an atomic unit (as long as the transaction doesn't include more than 100 entities and the payload of the request doesn't exceed 4 MB). Operations that span multiple partitions are not transactional, and might require you to implement eventual consistency as described by the [Data consistency primer]. For more information about table storage and transactions, go to the page [Performing entity group transactions] on the Microsoft website.
* Give careful attention to the granularity of the partition key because of the following reasons:
  * Using the same partition key for every entity causes the table storage service to create a single large partition that's held on one server. This prevents it from scaling out and instead focuses the load on a single server. As a result, this approach is only suitable for systems that manage a small number of entities. However, this approach does ensure that all entities can participate in entity group transactions.
  * Using a unique partition key for every entity causes the table storage service to create a separate partition for each entity, possibly resulting in a large number of small partitions (depending on the size of the entities). This approach is more scalable than using a single partition key, but entity group transactions are not possible. Also, queries that fetch more than one entity might involve reading from more than one server. However, if the application performs range queries, then using a monotonic sequence to generate the partition keys might help to optimize these queries.
  * Sharing the partition key across a subset of entities makes it possible for you to group related entities in the same partition. Operations that involve related entities can be performed by using entity group transactions, and queries that fetch a set of related entities can be satisfied by accessing a single server.

For additional information about partitioning data in Azure table storage, see the article [Azure storage table design guide] on the Microsoft website.

## Partitioning Azure blob storage
Azure blob storage makes it possible to hold large binary objects--currently up to 200 GB in size for block blobs or 1 TB for page blobs. (For the most recent information, go to the page [Azure Storage scalability and performance targets] on the Microsoft website.) Use block blobs in scenarios such as streaming where you need to upload or download large volumes of data quickly. Use page blobs for applications that require random rather than serial access to parts of the data.

Each blob (either block or page) is held in a container in an Azure storage account. You can use containers to group related blobs that have the same security requirements, although this grouping is logical rather than physical. Inside a container, each blob has a unique name.

Blob storage is automatically partitioned based on the blob name. Each blob is held in its own partition. Blobs in the same container do not share a partition. This architecture helps Azure blob storage to balance the load across servers transparently because different blobs in the same container can be distributed across different servers.

The actions of writing a single block (block blob) or page (page blob) are atomic, but operations that span blocks, pages, or blobs are not. If you need to ensure consistency when performing write operations across blocks, pages, and blobs, take out a write lock by using a blob lease.

Azure blob storage supports transfer rates of up to 60 MB per second or 500 requests per second for each blob. If you anticipate surpassing these limits, and the blob data is relatively static, then consider replicating blobs by using the Azure Content Delivery Network. For more information, see the page [Using Delivery Content Network for Azure] on the Microsoft website. For additional guidance and considerations, see  [Using Content Delivery Network for Azure].

## Partitioning Azure storage queues
Azure storage queues enable you to implement asynchronous messaging between processes. An Azure storage account can contain any number of queues, and each queue can contain any number of messages. The only limitation is the space that's available in the storage account. The maximum size of an individual message is 64 KB. If you require messages bigger than this, then consider using Azure Service Bus queues instead.

Each storage queue has a unique name within the storage account that contains it. Azure partitions queues based on the name. All messages for the same queue are stored in the same partition, which is controlled by a single server. Different queues can be managed by different servers to help balance the load. The allocation of queues to servers is transparent to applications and users.

 In a large-scale application, don't use the same storage queue for all instances of the application because this approach might cause the server that's hosting the queue to become a hotspot. Instead, use different queues for different functional areas of the application. Azure storage queues do not support transactions, so directing messages to different queues should have little impact on messaging consistency.

An Azure storage queue can handle up to 2,000 messages per second.  If you need to process messages at a greater rate than this, consider creating multiple queues. For example, in a global application, create separate storage queues in separate storage accounts to handle application instances that are running in each region.

## Partitioning strategies for Azure Service Bus
Azure Service Bus uses a message broker to handle messages that are sent to a Service Bus queue or topic. By default, all messages that are sent to a queue or topic are handled by the same message broker process. This architecture can place a limitation on the overall throughput of the message queue. However, you can also partition a queue or topic when it is created. You do this by setting the *EnablePartitioning* property of the queue or topic description to *true*.

A partitioned queue or topic is divided into multiple fragments, each of which is backed by a separate message store and message broker. Service Bus takes responsibility for creating and managing these fragments. When an application posts a message to a partitioned queue or topic, Service Bus assigns the message to a fragment for that queue or topic. When an application receives a message from a queue or subscription, Service Bus checks each fragment for the next available message and then passes it to the application for processing.

This structure helps distribute the load across message brokers and message stores, increasing scalability and improving availability. If the message broker or message store for one fragment is temporarily unavailable, Service Bus can retrieve messages from one of the remaining available fragments.

Service Bus assigns a message to a fragment as follows:

* If the message belongs to a session, all messages with the same value for the * SessionId*  property are sent to the same fragment.
* If the message does not belong to a session, but the sender has specified a value for the *PartitionKey* property, then all messages with the same *PartitionKey* value are sent to the same fragment.

  > [!NOTE]
  > If the *SessionId* and *PartitionKey* properties are both specified, then they must be set to the same value or the message will be rejected.
  >
  >
* If the *SessionId* and *PartitionKey* properties for a message are not specified, but duplicate detection is enabled, the *MessageId* property will be used. All messages with the same *MessageId* will be directed to the same fragment.
* If messages do not include a *SessionId, PartitionKey,* or *MessageId* property, then Service Bus assigns messages to fragments sequentially. If a fragment is unavailable, Service Bus will move on to the next. This means that a temporary fault in the messaging infrastructure does not cause the message-send operation to fail.

Consider the following points when deciding if or how to partition a Service Bus message queue or topic:

* Service Bus queues and topics are created within the scope of a Service Bus namespace. Service Bus currently allows up to 100 partitioned queues or topics per namespace.
* Each Service Bus namespace imposes quotas on the available resources, such as the number of subscriptions per topic, the number of concurrent send and receive requests per second, and the maximum number of concurrent connections that can be established. These quotas are documented on the Microsoft website on the page [Service Bus quotas]. If you expect to exceed these values, then create additional namespaces with their own queues and topics, and spread the work across these namespaces. For example, in a global application, create separate namespaces in each region and configure application instances to use the queues and topics in the nearest namespace.
* Messages that are sent as part of a transaction must specify a partition key. This can be a *SessionId*, *PartitionKey*, or *MessageId* property. All messages that are sent as part of the same transaction must specify the same partition key because they must be handled by the same message broker process. You cannot send messages to different queues or topics within the same transaction.
* Partitioned queues and topics can't be configured to be automatically deleted when they become idle.
* Partitioned queues and topics can't currently be used with the Advanced Message Queuing Protocol (AMQP) if you are building cross-platform or hybrid solutions.

## Partitioning strategies for Azure DocumentDB databases
Azure DocumentDB is a NoSQL database that can store documents. A document in a DocumentDB database is a JSON-serialized representation of an object or other piece of data. No fixed schemas are enforced except that every document must contain a unique ID.

Documents are organized into collections. You can group related documents together in a collection. For example, in a system that maintains blog postings, you can store the contents of each blog post as a document in a collection. You can also create collections for each subject type. Alternatively, in a multitenant application, such as a system where different authors control and manage their own blog posts, you can partition blogs by author and create separate collections for each author. The storage space that's allocated to collections is elastic and can shrink or grow as needed.

Document collections provide a natural mechanism for partitioning data within a single database. Internally, a DocumentDB database can span several servers and might attempt to spread the load by distributing collections across servers. The simplest way to implement sharding is to create a collection for each shard.

> [!NOTE]
> Each DocumentDB database has a *performance level* that determines the amount of resources it gets. A performance level is associated with a *request unit* (RU) rate limit. The RU rate limit specifies the volume of resources that's reserved and available for exclusive use by that collection. The cost of a collection depends on the performance level that's selected for that collection. The higher the performance level (and RU rate limit) the higher the charge. You can adjust the performance level of a collection by using the Azure portal. For more information, see the page [Performance levels in DocumentDB] on the Microsoft website.
>
>

All databases are created in the context of a DocumentDB account. A single DocumentDB account can contain several databases, and it specifies in which region the databases are created. Each DocumentDB account also enforces its own access control. You can use DocumentDB accounts to geo-locate shards (collections within databases) close to the users who need to access them, and enforce restrictions so that only those users can connect to them.

Each DocumentDB account has a quota that limits the number of databases and collections that it can contain and the amount of document storage that's available. These limits are subject to change, but are described on the page [DocumentDB limits and quotas] on the Microsoft website. It is theoretically possible that if you implement a system where all shards belong to the same database, you might reach the storage capacity limit of the account.

In this case, you might need to create additional DocumentDB accounts and databases, and distribute the shards across these databases. However, even if you are unlikely to reach the storage capacity of a database, it's a good practice to use multiple databases. That's because each database has its own set of users and permissions, and you can use this mechanism to isolate access to collections on a per-database basis.

Figure 8 illustrates the high-level structure of the DocumentDB architecture.

![The structure of DocumentDB](./images/data-partitioning/DocumentDBStructure.png)

*Figure 8.  The structure of the DocumentDB architecture*

It is the task of the client application to direct requests to the appropriate shard, usually by implementing its own mapping mechanism based on some attributes of the data that define the shard key. Figure 9 shows two DocumentDB databases, each containing two collections that are acting as shards. The data is sharded by a tenant ID and contains the data for a specific tenant. The databases are created in separate DocumentDB accounts. These accounts are located in the same region as the tenants for which they contain data. The routing logic in the client application uses the tenant ID as the shard key.

![Implementing sharding using Azure DocumentDB](./images/data-partitioning/DocumentDBPartitions.png)

*Figure 9. Implementing sharding using an Azure DocumentDB database*

Consider the following points when deciding how to partition data with a DocumentDB database:

* **The resources available to a DocumentDB database are subject to the quota limitations of the DocumentDB account**. Each database can hold a number of collections (again, there is a limit), and each collection is associated with a performance level that governs the RU rate limit (reserved throughput) for that collection. For more information, go to the page [DocumentDB limits and quotas] on the Microsoft website.
* **Each document must have an attribute that can be used to uniquely identify that document within the collection in which it is held**. This attribute is different from the shard key, which defines which collection holds the document. A collection can contain a large number of documents. In theory, it's limited only by the maximum length of the document ID. The document ID can be up to 255 characters.
* **All operations against a document are performed within the context of a transaction. Transactions in DocumentDB databases are scoped to the collection in which the document is contained.** If an operation fails, the work that it has performed is rolled back. While a document is subject to an operation, any changes that are made are subject to snapshot-level isolation. This mechanism guarantees that if, for example, a request to create a new document fails, another user who's querying the database simultaneously will not see a partial document that is then removed.
* **DocumentDB database queries are also scoped to the collection level**. A single query can retrieve data from only one collection. If you need to retrieve data from multiple collections, you must query each collection individually and merge the results in your application code.
* **DocumentDB databases supports programmable items that can all be stored in a collection alongside documents**. These include stored procedures, user-defined functions, and triggers (written in JavaScript). These items can access any document within the same collection. Furthermore, these items run either inside the scope of the ambient transaction (in the case of a trigger that fires as the result of a create, delete, or replace operation performed against a document), or by starting a new transaction (in the case of a stored procedure that is run as the result of an explicit client request). If the code in a programmable item throws an exception, the transaction is rolled back. You can use stored procedures and triggers to maintain integrity and consistency between documents, but these documents must all be part of the same collection.
* **The collections that you intend to hold in the databases in a DocumentDB account should be unlikely to exceed the throughput limits defined by the performance levels of the collections**. These limits are described on the page [Manage DocumentDB capacity needs] on the Microsoft website. If you anticipate reaching these limits, consider splitting collections across databases in different DocumentDB accounts to reduce the load per collection.

## Partitioning strategies for Azure Search
The ability to search for data is often the primary method of navigation and exploration that's provided by many web applications. It helps users find resources quickly (for example, products in an e-commerce application) based on combinations of search criteria. The Azure Search service provides full-text search capabilities over web content, and includes features such as type-ahead, suggested queries based on near matches, and faceted navigation. A full description of these capabilities is available on the page [What is Azure Search?] on the Microsoft website.

Azure Search stores searchable content as JSON documents in a database. You define indexes that specify the searchable fields in these documents and provide these definitions to Azure Search. When a user submits a search request, Azure Search uses the appropriate indexes to find matching items.

To reduce contention, the storage that's used by Azure Search can be divided into 1, 2, 3, 4, 6, or 12 partitions, and each partition can be replicated up to 6 times. The product of the number of partitions multiplied by the number of replicas is called the *search unit* (SU). A single instance of Azure Search can contain a maximum of 36 SUs (a database with 12 partitions only supports a maximum of 3 replicas).

You are billed for each SU that is allocated to your service. As the volume of searchable content increases or the rate of search requests grows, you can add SUs to an existing instance of Azure Search to handle the extra load. Azure Search itself distributes the documents evenly across the partitions. No manual partitioning strategies are currently supported.

Each partition can contain a maximum of 15 million documents or occupy 300 GB of storage space (whichever is smaller). You can create up to 50 indexes. The performance of the service varies and depends on the complexity of the documents, the available indexes, and the effects of network latency. On average, a single replica (1 SU) should be able to handle 15 queries per second (QPS), although we recommend performing benchmarking with your own data to obtain a more precise measure of throughput. For more information, see the page [Service limits in Azure Search] on the Microsoft website.

> [!NOTE]
> You can store a limited set of data types in searchable documents, including strings, Booleans, numeric data, datetime data, and some geographical data. For more details, see the page [Supported data types (Azure Search)] on the Microsoft website.
>
>

You have limited control over how Azure Search partitions data for each instance of the service. However, in a global environment you might be able to improve performance and reduce latency and contention further by partitioning the service itself using either of the following strategies:

* Create an instance of Azure Search in each geographic region, and ensure that client applications are directed towards the nearest available instance. This strategy requires that any updates to searchable content are replicated in a timely manner across all instances of the service.
* Create two tiers of Azure Search:

  * A local service in each region that contains the data that's most frequently accessed by users in that region. Users can direct requests here for fast but limited results.
  * A global service that encompasses all the data. Users can direct requests here for slower but more complete results.

This approach is most suitable when there is a significant regional variation in the data that's being searched.

## Partitioning strategies for Azure Redis Cache
Azure Redis Cache provides a shared caching service in the cloud that's based on the Redis key-value data store. As its name implies, Azure Redis Cache is intended as a caching solution. Use it only for holding transient data and not as a permanent data store. Applications that utilize Azure Redis Cache should be able to continue functioning if the cache is unavailable. Azure Redis Cache supports primary/secondary replication to provide high availability, but currently limits the maximum cache size to 53 GB. If you need more space than this, you must create additional caches. For more information, go to the page [Azure Redis Cache] on the Microsoft website.

Partitioning a Redis data store involves splitting the data across instances of the Redis service. Each instance constitutes a single partition. Azure Redis Cache abstracts the Redis services behind a façade and does not expose them directly. The simplest way to implement partitioning is to create multiple Azure Redis Cache instances and spread the data across them.

You can associate each data item with an identifier (a partition key) that specifies which cache stores the data item. The client application logic can then use this identifier to route requests to the appropriate partition. This scheme is very simple, but if the partitioning scheme changes (for example, if additional Azure Redis Cache instances are created), client applications might need to be reconfigured.

Native Redis (not Azure Redis Cache) supports server-side partitioning based on Redis clustering. In this approach, you can divide the data evenly across servers by using a hashing mechanism. Each Redis server stores metadata that describes the range of hash keys that the partition holds, and also contains information about which hash keys are located in the partitions on other servers.

Client applications simply send requests to any of the participating Redis servers (probably the closest one). The Redis server examines the client request. If it can be resolved locally, it performs the requested operation. Otherwise it forwards the request on to the appropriate server.

This model is implemented by using Redis clustering, and is described in more detail on the [Redis cluster tutorial] page on the Redis website. Redis clustering is transparent to client applications. Additional Redis servers can be added to the cluster (and the data can be re-partitioned) without requiring that you reconfigure the clients.

> [!IMPORTANT]
> Azure Redis Cache does not currently support Redis clustering. If you want to implement this approach with Azure, then you must implement your own Redis servers by installing Redis on a set of Azure virtual machines and configuring them manually. The page [Running Redis on a CentOS Linux VM in Azure] on the Microsoft website walks through an example that shows you how to build and configure a Redis node running as an Azure VM.
>
>

The page [Partitioning: how to split data among multiple Redis instances] on the Redis website provides more information about implementing partitioning with Redis. The remainder of this section assumes that you are implementing client-side or proxy-assisted partitioning.

Consider the following points when deciding how to partition data with Azure Redis Cache:

* Azure Redis Cache is not intended to act as a permanent data store, so whatever partitioning scheme you implement, your application code must be able to retrieve data from a location that's not the cache.
* Data that is frequently accessed together should be kept in the same partition. Redis is a powerful key-value store that provides several highly optimized mechanisms for structuring data. These mechanisms can be one of the following:

  * Simple strings (binary data up to 512 MB in length)
  * Aggregate types such as lists (which can act as queues and stacks)
  * Sets (ordered and unordered)
  * Hashes (which can group related fields together, such as the items that represent the fields in an object)
* The aggregate types enable you to associate many related values with the same key. A Redis key identifies a list, set, or hash rather than the data items that it contains. These types are all available with Azure Redis Cache and are described by the [Data types] page on the Redis website. For example, in part of an e-commerce system that tracks the orders that are placed by customers, the details of each customer can be stored in a Redis hash that is keyed by using the customer ID. Each hash can hold a collection of order IDs for the customer. A separate Redis set can hold the orders, again structured as hashes, and keyed by using the order ID. Figure 10 shows this structure. Note that Redis does not implement any form of referential integrity, so it is the developer's responsibility to maintain the relationships between customers and orders.

![Suggested structure in Redis storage for recording customer orders and their details](./images/data-partitioning/RedisCustomersandOrders.png)

*Figure 10. Suggested structure in Redis storage for recording customer orders and their details*

> [!NOTE]
> In Redis, all keys are binary data values (like Redis strings) and can contain up to 512 MB of data. In theory, a key can contain almost any information. However, we recommend adopting a consistent naming convention for keys that is descriptive of the type of data and that identifies the entity, but is not excessively long. A common approach is to use keys of the form "entity_type:ID". For example, you can use "customer:99" to indicate the key for a customer with the ID 99.
>
>

* You can implement vertical partitioning by storing related information in different aggregations in the same database. For example, in an e-commerce application, you can store commonly accessed information about products in one Redis hash and less frequently used detailed information in another.
  Both hashes can use the same product ID as part of the key. For example, you can use "product: *nn*" (where *nn* is the product ID) for the product information and "product_details: *nn*" for the detailed data. This strategy can help reduce the volume of data that most queries are likely to retrieve.
* You can repartition a Redis data store, but keep in mind that it's a complex and time-consuming task. Redis clustering can repartition data automatically, but this capability is not available with Azure Redis Cache. Therefore, when you design your partitioning scheme, try to leave sufficient free space in each partition to allow for expected data growth over time. However, remember that Azure Redis Cache is intended to cache data temporarily, and that data held in the cache can have a limited lifetime specified as a time-to-live (TTL) value. For relatively volatile data, the TTL can be short, but for static data the TTL can be a lot longer. Avoid storing large amounts of long-lived data in the cache if the volume of this data is likely to fill the cache. You can specify an eviction policy that causes Azure Redis Cache to remove data if space is at a premium.

  > [!NOTE]
  > When you use Azure Redis cache, you specify the maximum size of the cache (from 250 MB to 53 GB) by selecting the appropriate pricing tier. However, after an Azure Redis Cache has been created, you cannot increase (or decrease) its size.
  >
  >
* Redis batches and transactions cannot span multiple connections, so all data that is affected by a batch or transaction should be held in the same database (shard).

  > [!NOTE]
  > A sequence of operations in a Redis transaction is not necessarily atomic. The commands that compose a transaction are verified and queued before they run. If an error occurs during this phase, the entire queue is discarded. However, after the transaction has been successfully submitted, the queued commands run in sequence. If any command fails, only that command stops running. All previous and subsequent commands in the queue are performed. For more information, go to the [Transactions] page on the Redis website.
  >
  >
* Redis supports a limited number of atomic operations. The only operations of this type that support multiple keys and values are MGET and MSET operations. MGET operations return a collection of values for a specified list of keys, and MSET operations store a collection of values for a specified list of keys. If you need to use these operations, the key-value pairs that are referenced by the MSET and MGET commands must be stored within the same database.

## Rebalancing partitions
As a system matures and you understand the usage patterns better, you might have to adjust the partitioning scheme. For example, individual partitions might start attracting a disproportionate volume of traffic and become hot, leading to excessive contention. Additionally, you might have underestimated the volume of data in some partitions, causing you to approach the limits of the storage capacity in these partitions. Whatever the cause, it is sometimes necessary to rebalance partitions to spread the load more evenly.

In some cases, data storage systems that don't publicly expose how data is allocated to servers can automatically rebalance partitions within the limits of the resources available. In other situations, rebalancing is an administrative task that consists of two stages:

1. Determining the new partitioning strategy to ascertain:
   * Which partitions might need to be split (or possibly combined).
   * How to allocate data to these new partitions by designing new partition keys.
2. Migrating the affected data from the old partitioning scheme to the new set of partitions.

> [!NOTE]
> The mapping of DocumentDB database collections to servers is transparent, but you can still reach the storage capacity and throughput limits of a DocumentDB account. If this happens, you might need to redesign your partitioning scheme and migrate the data.
>
>

Depending on the data storage technology and the design of your data storage system, you might be able to migrate data between partitions while they are in use (online migration). If this isn't possible, you might need to make the affected partitions temporarily unavailable while the data is relocated (offline migration).

## Offline migration
Offline migration is arguably the simplest approach because it reduces the chances of contention occurring. Don't make any changes to the data while it is being moved and restructured.

Conceptually, this process includes the following steps:

1. Mark the shard offline.
2. Split-merge and move the data to the new shards.
3. Verify the data.
4. Bring the new shards online.
5. Remove the old shard.

To retain some availability, you can mark the original shard as read-only in step 1 rather than making it unavailable. This allows applications to read the data while it is being moved but not to change it.

## Online migration
Online migration is more complex to perform but less disruptive to users because data remains available during the entire procedure. The process is similar to that used by offline migration, except that the original shard is not marked offline (step 1). Depending on the granularity of the migration process (for example, whether it's done item by item or shard by shard), the data access code in the client applications might have to handle reading and writing data that's held in two locations (the original shard and the new shard).

For an example of a solution that supports online migration, see the article [Scaling using the Elastic Database split-merge tool] on the Microsoft website.

## Related patterns and guidance
When considering strategies for implementing data consistency, the following patterns might also be relevant to your scenario:

* The [Data consistency primer] page on the Microsoft website describes strategies for maintaining consistency in a distributed environment such as the cloud.
* The [Data partitioning guidance] page on the Microsoft website provides a general overview of how to design partitions to meet various criteria in a distributed solution.
* The [sharding pattern] as described on the Microsoft website summarizes some common strategies for sharding data.
* The [index table pattern] as described on the Microsoft website illustrates how to create secondary indexes over data. An application can quickly retrieve data with this approach, by using queries that do not reference the primary key of a collection.
* The [materialized view pattern] as described on the Microsoft website describes how to generate pre-populated views that summarize data to support fast query operations. This approach can be useful in a partitioned data store if the partitions that contain the data being summarized are distributed across multiple sites.
* The [Using Azure Content Delivery Network] article on the Microsoft website provides additional guidance on configuring and using Content Delivery Network with Azure.

## More information
* The page [What is Azure SQL Database?] on the Microsoft website provides detailed documentation that describes how to create and use SQL databases.
* The page [Elastic Database features overview] on the Microsoft website provides a comprehensive introduction to Elastic Database.
* The page [Scaling using the Elastic Database split-merge tool] on the Microsoft website contains information about using the split-merge service to manage Elastic Database shards.
* The page [Azure storage scalability and performance targets](https://msdn.microsoft.com/library/azure/dn249410.aspx) on the Microsoft website documents the current sizing and throughput limits of Azure Storage.
* The page [Performing entity group transactions] on the Microsoft website provides detailed information about implementing transactional operations over entities that are stored in Azure table storage.
* The article [Azure Storage table design guide] on the Microsoft website contains detailed information about partitioning data in Azure table storage.
* The page [Using Azure Content Delivery Network] on the Microsoft website describes how to replicate data that's held in Azure blob storage by using the Azure Content Delivery Network.
* The page [Manage DocumentDB capacity needs] on the Microsoft website contains information about how Azure DocumentDB databases allocate resources.
* The page [What is Azure Search?] on the Microsoft website provides a full description of the capabilities that are available in Azure Search.
* The page [Service limits in Azure Search] on the Microsoft website contains information about the capacity of each instance of Azure Search.
* The page [Supported data types (Azure Search)] on the Microsoft website summarizes the data types that you can use in searchable documents and indexes.
* The page [Azure Redis Cache] on the Microsoft website provides an introduction to Azure Redis Cache.
* The [Partitioning: how to split data among multiple Redis instances] page on the Redis website provides information about how to implement partitioning with Redis.
* The page [Running Redis on a CentOS Linux VM in Azure] on the Microsoft website walks through an example that shows you how to build and configure a Redis node running as an Azure VM.
* The [Data types] page on the Redis website describes the data types that are available with Redis and Azure Redis Cache.

[Azure Redis Cache]: http://azure.microsoft.com/services/cache/
[Azure Storage Scalability and Performance Targets]: /azure/storage/storage-scalability-targets
[Azure Storage Table Design Guide]: /azure/storage/storage-table-design-guide
[Building a Polyglot Solution]: https://msdn.microsoft.com/library/dn313279.aspx
[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence]: https://msdn.microsoft.com/library/dn271399.aspx
[Data consistency primer]: http://aka.ms/Data-Consistency-Primer
[Data Partitioning Guidance]: https://msdn.microsoft.com/library/dn589795.aspx
[Data Types]: http://redis.io/topics/data-types
[DocumentDB limits and quotas]: /azure/documentdb/documentdb-limits
[Elastic Database features overview]: /azure/sql-database/sql-database-elastic-scale-introduction
[Federations Migration Utility]: https://code.msdn.microsoft.com/vstudio/Federations-Migration-ce61e9c1
[Index Table Pattern]: http://aka.ms/Index-Table-Pattern
[Manage DocumentDB capacity needs]: /azure/documentdb/documentdb-manage
[Materialized View Pattern]: http://aka.ms/Materialized-View-Pattern
[Multi-shard querying]: /azure/sql-database/sql-database-elastic-scale-multishard-querying
[Partitioning: how to split data among multiple Redis instances]: http://redis.io/topics/partitioning
[Performance levels in DocumentDB]: /azure/documentdb/documentdb-performance-levels
[Performing Entity Group Transactions]: https://msdn.microsoft.com/library/azure/dd894038.aspx
[Redis cluster tutorial]: http://redis.io/topics/cluster-tutorial
[Running Redis on a CentOS Linux VM in Azure]: http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx
[Scaling using the Elastic Database split-merge tool]: /azure/sql-database/sql-database-elastic-scale-overview-split-and-merge
[Using Azure Content Delivery Network]: /azure/cdn/cdn-create-new-endpoint
[Service Bus quotas]: /azure/service-bus-messaging/service-bus-quotas
[Service limits in Azure Search]:  /azure/search/search-limits-quotas-capacity
[Sharding pattern]: http://aka.ms/Sharding-Pattern
[Supported Data Types (Azure Search)]:  https://msdn.microsoft.com/library/azure/dn798938.aspx
[Transactions]: http://redis.io/topics/transactions
[What is Azure Search?]: /azure/search/search-what-is-azure-search
[What is Azure SQL Database?]: /azure/sql-database/sql-database-technical-overview
