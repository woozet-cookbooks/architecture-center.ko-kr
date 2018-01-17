---
title: "캐싱 지침"
description: "성능 및 확장성을 향상하기 위한 캐시에 대한 지침입니다."
author: dragon119
ms.date: 05/24/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 7968c1578dfef2c7ad28576b9aafbbe2b6672cd9
ms.sourcegitcommit: 3d6dba524cc7661740bdbaf43870de7728d60a01
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/11/2018
---
# <a name="caching"></a>구성

캐싱은 시스템의 성능 및 확장성을 개선하는 데 목표를 두는 일반적인 기술입니다. 이를 위해 자주 액세스하는 데이터를 응용 프로그램에 가까이 있는 빠른 저장소에 일시적으로 복사합니다. 이 빠른 데이터 저장소가 원래 원본보다 응용 프로그램에 가까이 위치하는 경우 캐싱은 데이터를 보다 신속하게 이용하여 클라이언트 응용 프로그램에 대한 응답 시간을 훨씬 향상할 수 있습니다.

캐싱은 클라이언트 인스턴스가 동일한 데이터를 반복해서 읽을 때 특히 다음 조건이 모두 원래 데이터 저장소에 적용되는 경우에 가장 효과적입니다.

* 상대적으로 정적 상태로 유지됩니다.
* 캐시 속도와 비교하여 느립니다.
* 높은 수준의 경합이 발생하기 쉽습니다.
* 네트워크 대기 시간이 액세스를 느려지게 할 수 있는 경우 멀리 있습니다.

## <a name="caching-in-distributed-applications"></a>분산된 응용 프로그램에서 캐싱
분산된 응용 프로그램은 데이터를 캐시할 때 일반적으로 다음 전략 중 하나 또는 모두를 구현합니다.

* 데이터가 응용 프로그램 또는 서비스의 인스턴스를 실행하는 컴퓨터에서 로컬로 유지되는 전용 캐시를 사용합니다.
* 여러 프로세스 및/또는 컴퓨터가 액세스할 수 있는 일반적인 소스로 제공하는 공유 캐시를 사용합니다.

두 경우 모두 캐싱이 클라이언트 쪽 및/또는 서버 쪽에서 수행될 수 있습니다. 클라이언트 쪽 캐싱은 웹 브라우저 또는 데스크톱 응용 프로그램과 같이 시스템의 사용자 인터페이스를 제공하는 프로세스에서 수행됩니다.
서버 쪽 캐싱은 원격으로 실행되는 비즈니스 서비스를 제공하는 프로세스에서 수행됩니다.

### <a name="private-caching"></a>전용 캐싱
캐시의 가장 기본적인 형식은 메모리 내 저장소입니다. 이는 단일 프로세스의 주소 공간에 보유하고 해당 프로세스에서 실행되는 코드에서 직접 액세스합니다. 이 유형의 캐시는 액세스가 매우 빠릅니다. 캐시의 크기가 일반적으로 프로세스를 호스트하는 컴퓨터에서 사용 가능한 메모리의 볼륨으로 제한되므로 적당한 양의 정적 데이터를 저장하기 위해 매우 효과적인 수단을 제공할 수도 있습니다.

메모리에서 물리적으로 가능한 것 보다 많은 정보를 캐시하는 경우 로컬 파일 시스템에 캐시된 데이터를 작성할 수 있습니다. 이를 통한 액세스는 데이터 메모리에 유지된 데이터보다 느리지만 여전히 네트워크를 통해 데이터를 검색할 때보다 빠르고 안정적입니다.

이 모델을 사용하는 응용 프로그램의 여러 인스턴스가 동시에 실행되는 경우 각 응용 프로그램 인스턴스가 데이터의 자체 복사본을 유지하는 자체 독립적인 캐시 데이터를 보유합니다.

캐시는 과거 어떤 시점에 있던 원래 데이터의 스냅숏으로 생각할 수 있습니다. 이 데이터가 정적이지 않으면 다른 응용 프로그램 인스턴스가 캐시에 서로 다른 버전의 데이터를 유지할 가능성이 있습니다. 따라서 이러한 인스턴스가 수행한 동일한 쿼리는 그림 1에 나와있는 것처럼 서로 다른 결과를 반환할 수 있습니다.

![응용 프로그램의 서로 다른 인스턴스에서 메모리 내 캐시를 사용](./images/caching/Figure1.png)

*그림 1: 응용 프로그램의 서로 다른 인스턴스에서 메모리 내 캐시를 사용*

### <a name="shared-caching"></a>공유 캐싱
메모리 내 캐싱으로 발생하는 각 캐시 데이터가 달라지는 문제는 공유 캐시를 사용하면 완화할 수 있습니다. 공유 캐싱을 통해 다른 응용 프로그램 인스턴스가 캐시된 데이터의 동일한 보기를 볼 수 있습니다. 이를 위해 그림 2와 같이 일반적으로 별도 서비스의 일부로 호스트된 별도 위치에 있는 캐시를 찾습니다.

![공유 캐시 사용](./images/caching/Figure2.png)

*그림 2: 공유 캐시 사용*

공유 캐싱 방법을 사용하는 중요한 이점은 확장성을 제공하는 것입니다. 많은 공유 캐시 서비스는 클러스터의 서버를 사용하여 구현되며 투명한 방식으로 클러스터 전체에서 데이터를 배포하는 소프트웨어를 활용합니다. 응용 프로그램 인스턴스가 캐시 서비스에 요청을 간단히 보냅니다.
기본 인프라가 클러스터에서 캐시된 데이터의 위치를 결정합니다. 더 많은 서버를 추가하여 캐시를 쉽게 확장할 수 있습니다.

공유 캐싱 접근 방법에는 두 가지 주요 약점이 있습니다.

* 캐시는 더 이상 각 응용 프로그램 인스턴스에 로컬로 유지되지 않기 때문에 액세스가 느립니다.
* 별도 캐시 서비스를 구현하는 요구 사항이 솔루션에 복잡성을 추가할 수 있습니다.

## <a name="considerations-for-using-caching"></a>캐싱 사용에 대한 고려 사항
다음 섹션에서는 캐시 설계 및 사용에 대한 고려 사항을 자세히 설명합니다.

### <a name="decide-when-to-cache-data"></a>데이터를 캐시하는 시점을 결정합니다.
캐싱은 성능, 확장성 및 가용성을 크게 향상할 수 있습니다. 보유한 데이터가 많을수록 그리고 이 데이터에 액세스해야 하는 사용자 수가 많을수록 캐싱의 이점은 더 커집니다. 이는 캐싱이 원래 데이터 저장소의 많은 양의 동시 요청 처리와 관련된 대기 시간 및 경합을 줄이기 때문입니다.

예를 들어 데이터베이스는 제한된 수의 동시 연결을 지원할 수 있습니다. 하지만 기본 데이터베이스가 아닌 공유 캐시에서 데이터를 검색하면 사용 가능한 연결 수를 현재 모두 사용한 경우에도 클라이언트 응용 프로그램에서 이 데이터에 액세스할 수 있습니다. 또한 데이터베이스를 사용할 수 없게 된 경우 클라이언트 응용 프로그램은 캐시에 저장된 데이터를 사용하여 계속할 수 있습니다.

자주 읽지만 자주 수정하지 않는 데이터(예: 쓰기 작업보다 읽기 작업의 비율이 더 높은 데이터)를 캐시하는 것이 좋습니다. 그러나 캐시를 중요한 정보의 신뢰할 수 있는 저장소로는 사용하지 않는 것이 좋습니다. 대신 응용 프로그램에서 손실을 수용할 수 없는 모든 변경 내용은 항상 영구 데이터 저장소에 저장되어야 합니다. 이렇게 하면 캐시를 사용할 수 없는 경우 응용 프로그램이 데이터 저장소를 사용하여 작업을 계속할 수 있으며 중요한 정보는 손실되지 않습니다.

### <a name="determine-how-to-cache-data-effectively"></a>데이터를 효과적으로 캐시하는 방법 결정
효과적으로 캐시를 사용하는 열쇠는 캐시에 가장 적합한 데이터를 결정하고 적절한 시점에 캐싱하는 것입니다. 데이터가 처음으로 응용 프로그램에서 검색되었을 때 필요에 따라 캐시에 데이터를 추가할 수 있습니다. 즉, 응용 프로그램이 데이터 저장소에서 한 번만 데이터를 가져와야 하며 이후 액세스는 캐시를 사용하여 충족할 수 있습니다.

또는 일반적으로 응용 프로그램을 시작할 때 캐시가 사전에 부분적으로 또는 완전히 데이터로 채워질 수 있습니다(시딩이라고 알려진 방법). 그러나 이 방법은 응용 프로그램을 실행하기 시작할 때 데이터 저장소에 갑자기 높은 부하를 부과하여 큰 캐시에 대한 시드를 구현하는 데 권장하지 않습니다.

보통, 사용 패턴에 대한 분석을 실행하면 완전히 또는 부분적으로 캐시를 미리 채울지 결정하고 캐시가 될 데이터를 선택하는 데 도움이 됩니다. 예를 들어 응용 프로그램을 일주일에 한 번만 사용하는 고객이 아니라 정기적으로(보통 매일) 사용하는 고객에 대해 정적 사용자 프로필 데이터와 함께 캐시를 시드하는 것이 유용할 것입니다.

캐싱은 일반적으로 변경할 수 없거나 자주 변경되지 않는 데이터와 잘 작동 합니다. 예로 전자 상거래 응용 프로그램에서 제품 및 가격 정보와 같은 참조 정보 또는 생성 비용이 많이 드는 공유 정적 리소스를 포함합니다. 이 데이터의 일부 또는 전부를 응용 프로그램 시작 시 로드하여 리소스에 대한 수요를 최소화하고 성능을 향상할 수 있습니다. 또한 백그라운드 프로세스가 주기적으로 최신 상태를 확인하기 위해 캐시의 참조 데이터를 업데이트하거나 참조 데이터가 변경되는 경우 새로 고치는 것이 적합할 수 있습니다.

이 고려 사항에 일부 예외는 있지만 캐싱이 동적 데이터에서 유용하지 않습니다(자세한 내용은 이 문서의 뒷부분에 나오는 매우 동적인 데이터 캐시 섹션을 참조). 원래 데이터를 정기적으로 변경하면 캐시된 정보가 매우 빠르게 유효하지 않게 되거나 원래 데이터 저장소와 캐시를 동기화하는 오버 헤드에서 캐싱 효율성이 떨어집니다.

캐시는 엔터티에 대한 완전한 데이터를 포함할 필요가 없습니다. 예를 들어 데이터 항목이 이름, 주소 및 계정 잔액을 가진 은행 고객과 같은 다중값 개체를 나타내는 경우 다른 요소(예: 계정 잔액)가 보다 동적인 반면 이러한 요소 중 일부는 정적(예: 이름, 주소)으로 남아 있을 수 있습니다. 이러한 상황에서 필요하다면 데이터의 정적 부분을 캐시하고 남아 있는 정보만 검색(또는 계산)하는 것이 유용할 수 있습니다.

캐시에 대해 미리 채우기, 요청 로딩 또는 둘의 조합 중에 적절한 것을 확인하기 위해 성능 테스트 및 사용 현황 분석을 수행하는 것이 좋습니다. 결정은 변동성 및 데이터의 사용 패턴을 기반으로 해야 합니다. 캐시 사용률 및 성능 분석은 과도한 로드를 발생하며 확장성이 뛰어나야 하는 응용 프로그램에서 특히 중요합니다. 예를 들어 확장성이 뛰어난 시나리오에서 사용량이 많은 시간에 데이터 저장소에서 부하를 줄이도록 캐시를 시드하는 것이 합리적일 수 있습니다.

응용 프로그램이 실행될 때 계산 반복을 피하기 위해 캐싱도 사용할 수 있습니다. 작업이 데이터를 변환하거나 복잡 한 계산을 수행하는 경우 캐시에 작업의 결과를 저장할 수 있습니다. 같은 계산을 이후에 수행해야 하는 경우 응용 프로그램이 캐시에서 결과를 간단히 검색할 수 있습니다.

응용 프로그램이 캐시에 저장된 데이터를 수정할 수 있습니다. 그러나 캐시는 언제든지 사라질 수 있는 임시 데이터 저장소라고 생각하는 것이 좋습니다. 캐시에만 중요한 데이터를 저장하지 말고 원래 데이터 저장소에도 정보를 유지해야 합니다. 즉, 캐시를 사용할 수 없게 되는 경우 데이터 손실 가능성을 최소화합니다.

### <a name="cache-highly-dynamic-data"></a>매우 동적인 데이터 캐싱
영구 데이터 저장소에 빠르게 변경되는 정보를 저장하는 경우 시스템에 오버헤드가 부과될 수 있습니다. 예를 들어 상태 또는 일부 다른 측정을 지속적으로 보고하는 장치를 고려합니다. 응용 프로그램이 캐시된 정보가 거의 항상 오래된 기준으로 이 데이터를 캐시하지 않도록 선택하면 데이터 저장소에서 이 정보를 저장하고 검색하는 경우 동일하게 고려합니다. 이 데이터를 저장하고 가져오는 데 걸리는 시간에 변경되었을 수 있습니다.

이와 같은 상황에서 영구 데이터 저장소 대신 캐시에 동적 정보를 직접 저장하는 것의 이점을 고려하세요. 데이터가 중요하지 않고 감사를 필요로 하지 않으면 필요에 따른 변경이 손실된 경우 중요하지 않습니다.

### <a name="manage-data-expiration-in-a-cache"></a>캐시 데이터의 만료 관리
대부분의 경우 캐시에 저장된 데이터는 원래 데이터 저장소에 저장된 데이터의 복사본입니다. 원래 데이터 저장소의 데이터는 캐시된 이후 변경될 수 있으며 이는 캐시된 데이터의 생산성을 떨어 뜨리게 됩니다. 많은 캐싱 시스템을 사용하면 캐시의 데이터를 만료하고 데이터가 오래된 기간을 줄이도록 구성할 수 있습니다.

캐시된 데이터가 만료되면 캐시에서 제거되고 응용 프로그램은 원래 데이터 저장소에서 데이터를 검색해야 합니다(새로 가져온 정보를 다시 캐시에 저장할 수 있음). 캐시를 구성할 때 기본 만료 정책을 설정할 수 있습니다. 많은 캐시 서비스에서 개별 개체를 프로그래밍 방식으로 캐시에 저장할 때 개별 개체에 대한 만료 기간을 규정할 수도 있습니다.
일부 캐시를 사용하여 만료 기간을 절대값으로 지정하거나 항목이 지정된 시간 내에 액세스되지 않으면 캐시에서 제거되도록 하는 상대값으로 지정할 수 있습니다. 이 설정은 지정된 개체에 대한 모든 캐시 만료 정책보다 우선합니다.

> [!NOTE]
> 캐시와 신중하게 포함된 개체에 대한 만료 기간을 고려해야 합니다. 너무 짧게 만들면 개체는 너무 빨리 만료되고 캐시를 사용하는 이점은 줄어듭니다. 너무 기간을 길게 만들면 데이터가 오래될 위험이 있습니다.
> 
> 

또한 데이터가 오랜 시간 동안 상주하도록 허용되면 캐시를 채우는 것이 가능합니다. 이 경우 캐시에 새 항목을 추가하는 모든 요청이 제거라는 프로세스에서 일부 항목을 강제로 제거할 수 있습니다. 캐시 서비스는 가장 최근 사용(LRU)을 기초로 데이터를 제거하지만 일반적으로 이 정책을 재정의하고 항목을 제거하지 않도록 방지할 수 있습니다. 그러나 이 접근 방식을 사용하면 캐시에서 사용할 수 있는 메모리를 초과할 위험이 있습니다. 응용 프로그램이 캐시에 항목을 추가하려 하면 예외가 발생하여 실패합니다.

일부 캐싱 구현은 추가적인 제거 정책을 제공할 수 있습니다. 여러 가지 유형의 제거 정책이 있습니다. 내용은 다음과 같습니다.

* 가장 최근에 사용한 데이터 제거 정책(데이터가 다시 필요하지 않다고 예상)
* 선입선출 제거 정책(오래된 데이터를 먼저 제거)
* 트리거된 이벤트 기반의 명시적 제거 정책(예: 수정 중인 데이터)

### <a name="invalidate-data-in-a-client-side-cache"></a>클라이언트 쪽 캐시에서 데이터 무효화
클라이언트 쪽 캐시에 저장된 데이터는 일반적으로 클라이언트에 데이터를 제공하는 서비스의 외부로 간주됩니다. 서비스에서 클라이언트가 클라이언트 쪽 캐시에서 정보를 추가하거나 제거하도록 직접 강제할 수 없습니다.

즉, 잘못 구성된 캐시를 사용하는 클라이언트가 오래된 정보를 계속 사용할 수 있습니다. 예를 들어 캐시의 만료 정책이 제대로 구현되지 않은 경우 클라이언트는 원래 데이터 원본의 정보가 변경되었을 때 로컬로 캐시된 오래된 정보를 사용할 수 있습니다.

HTTP 연결을 통해 데이터를 제공하는 웹 응용 프로그램을 작성하는 경우 웹 클라이언트(예: 브라우저 또는 웹 프록시)가 가장 최근의 정보를 가져오도록 암시적으로 강요할 수 있습니다. 리소스가 해당 리소스의 URI에서 변경에 의해 업데이트된 경우 이렇게 수행할 수 있습니다. 웹 클라이언트는 일반적으로 리소스의 URI를 클라이언트 쪽 캐시에서 키로 사용하므로 URI가 변경되면 웹 클라이언트는 모든 이전에 캐시된 버전의 리소스를 무시하고 대신 새 버전을 가져옵니다.

## <a name="managing-concurrency-in-a-cache"></a>캐시에서 동시성 관리
캐시는 종종 응용 프로그램의 여러 인스턴스에서 공유하도록 설계됩니다. 각 응용 프로그램 인스턴스가 캐시에서 데이터를 읽고 수정할 수 있습니다. 따라서 모든 공유 데이터 저장소와 함께 발생하는 동일한 동시성 문제는 캐시에도 적용됩니다. 응용 프로그램이 캐시에 보유하는 데이터를 수정해야 하는 경우 응용 프로그램의 한 인스턴스가 만드는 업데이트가 다른 인스턴스가 만든 변경을 덮어쓰지 않도록 해야 합니다.

데이터의 특성 및 충돌 가능성에 따라 동시성에 대한 두 방법 중 하나를 채택할 수 있습니다.

* **낙관적입니다.** 응용 프로그램을 업데이트하기 바로 전에 캐시에서 데이터가 검색된 이후 변경되지 않았는지를 확인합니다. 데이터가 여전히 동일한 경우 변경이 가능합니다. 그렇지 않으면 응용 프로그램에서 업데이트 여부를 결정해야 합니다. (이를 결정하는 비즈니스 논리는 응용 프로그램에 특정됨) 이 방법은 업데이트가 드물거나 충돌이 발생할 가능성이 없는 상황에 적합합니다.
* **비관적입니다.** 응용 프로그램이 데이터를 검색할 때 다른 인스턴스가 데이터를 변경하지 못하도록 캐시에서 잠급니다. 이렇게 하면 충돌이 발생할 수 없지만 동일한 데이터를 처리해야 하는 다른 인스턴스를 차단할 수도 있습니다. 비관적 동시성은 솔루션의 확장성에 영향을 줄 수 있으며 단기 실행 작업에 대해서만 사용하는 것이 좋습니다. 특히 응용 프로그램이 캐시에 여러 항목을 업데이트하고 이러한 변경 내용이 일관되게 적용되었는지 확인해야 하는 경우 이 방법이 충돌 가능성이 더 높은 상황에 적절할 수 있습니다.

### <a name="implement-high-availability-and-scalability-and-improve-performance"></a>고가용성 및 확장성 구현과 성능 향상
데이터의 기본 리포지토리로 캐시를 사용하지 않도록 합니다. 이는 캐시가 채워지면 원래 데이터 저장소의 역할입니다. 원래 데이터 저장소는 데이터의 지 속성 보장을 담당합니다.

공유 캐시 서비스의 가용성에 대한 높은 종속성을 솔루션에 도입하지 않도록 주의하십시오. 공유 캐시를 제공하는 서비스를 사용할 수 없는 경우 응용 프로그램이 계속 작동될 수 있어야 합니다. 캐시 서비스가 다시 시작되기를 기다리는 동안 응용 프로그램이 중단되거나 실패하지 않아야 합니다.

따라서 응용 프로그램은 캐시 서비스의 가용성을 검색하여 준비하고 캐시에 액세스할 수 없는 경우 원래 데이터 저장소로 대체해야 합니다. [회로 차단기 패턴](http://msdn.microsoft.com/library/dn589784.aspx)은 시나리오를 처리하는데 유용합니다. [캐시 배제 패턴](http://msdn.microsoft.com/library/dn589799.aspx)과 같은 전략을 따라 캐시를 제공하는 서비스를 복구할 수 있으며 원래 데이터 저장소에서 데이터를 읽을 때 서비스를 사용할 수 있게 되면 캐시를 다시 채울 수 있습니다.

그러나 캐시를 일시적으로 사용할 수 없을 때 응용 프로그램이 다시 원래 데이터 저장소로 되돌아가는 경우 시스템의 확장성에 영향을 미칠 수 있습니다.
데이터 저장소가 복구 중인 동안 원래 데이터 저장소에 데이터 요청이 너무 많아 시간이 초과되고 연결되지 않을 수 있습니다.

모든 응용 프로그램 인스턴스가 액세스하는 공유 캐시와 함께 응용 프로그램의 각 인스턴스에 로컬, 개인 캐시를 구현하는 것이 좋습니다. 응용 프로그램이 항목을 검색하는 경우 먼저 로컬 캐시에서 다음 공유 캐시에서 마지막으로 원래 데이터 저장소에서 확인합니다. 공유 캐시 또는 데이터베이스(공유 캐시를 사용할 수 없는 경우)의 데이터를 사용하여 로컬 캐시를 채울 수 있습니다.

이 방법은 로컬 캐시가 공유 캐시와 관련하여 너무 부실해지지 않도록 신중하게 구성해야 합니다. 그러나 공유 캐시에 연결할 수 없는 경우 로컬 캐시가 버퍼 역할을 합니다. 그림 3이 이 구조를 보여줍니다.

![공유 캐시와 로컬, 개인 캐시 사용](./images/caching/Caching3.png)
*Figure 3: 공유 캐시와 로컬, 개인 캐시 사용*

상대적으로 수명이 긴 데이터를 보유하는 대형 캐시를 지원하려면 캐시를 사용할 수 없게 된 경우 일부 캐시 서비스가 자동 장애 조치를 구현하는 고가용성 옵션을 제공합니다. 이 방법은 일반적으로 주 캐시 서버에 저장된 캐시된 데이터를 보조 캐시 서버로 복제하고 주 서버에 오류가 발생하거나 연결이 손실되는 경우 보조 서버로 전환하는 기능을 포함합니다.

여러 대상에 쓰기와 연결된 대기 시간을 줄이려면 주 서버의 캐시에 데이터를 쓸 때 보조 서버에 복제가 비동기적으로 발생할 수 있습니다. 이 방법은 일부 캐시된 정보에 오류가 발생하여 손실될 가능성으로 이어지지만 캐시의 전체 크기에 비해 이 데이터의 비율이 작을 것입니다.

공유 캐시가 크면 노드에 캐시된 데이터를 분할하여 경합 가능성을 줄이고 확장성을 개선하는 데 도움이 될 수 있습니다. 많은 공유 캐시는 노드를 동적으로 추가(및 제거)하고 파티션에 데이터 균형을 다시 조정하는 기능을 지원합니다. 이 방법은 노드 컬렉션이 클라이언트 응용 프로그램에 원활한 단일 캐시로 표시되는 클러스터링을 포함할 수 있습니다. 하지만 내부적으로 데이터는 부하를 균등하게 분산하는 미리 정의된 배포 전략에 따라 노드 간에 분산됩니다. Microsoft 웹 사이트의 [데이터 분할 참고 자료 문서](http://msdn.microsoft.com/library/dn589795.aspx)는 가능한 분할 전략에 대한 자세한 정보를 제공합니다.

또한 클러스터링을 사용하면 캐시의 가용성을 높일 수 있습니다. 노드가 실패해도 캐시의 나머지는 계속 액세스할 수 있습니다.
클러스터링은 복제 및 장애 조치(Failover)와 함께 자주 사용됩니다. 각 노드는 복제할 수 있으며 노드가 실패하면 복제본이 신속하게 온라인 상태로 전환될 수 있습니다.

많은 읽기 및 쓰기 작업은 단일 데이터 값이나 개체를 포함할 가능성이 있습니다. 그러나 필요할 때 많은 양의 데이터를 신속하게 저장하거나 검색해야 할 수 있습니다.
예를 들어 캐시를 시드하면 수백 또는 수천 개의 항목을 캐시에 작성할 수 있습니다. 또한 응용 프로그램은 동일한 요청의 일부로 캐시에서 많은 수의 관련 항목을 검색해야 할 수도 있습니다.

많은 대규모 캐시는 이러한 목적을 위해 배치 작업을 제공합니다. 이를 통해 클라이언트 응용 프로그램에서 대량의 항목을 단일 요청으로 패키지할 수 있고 많은 수의 작은 요청을 수행하는 것과 관련된 오버헤드를 줄일 수 있습니다.

## <a name="caching-and-eventual-consistency"></a>캐싱 및 결과적 일관성
캐시 배제 패턴이 작동하려면 캐시를 채우는 응용 프로그램의 인스턴스가 최근의 일관된 버전의 데이터에 액세스해야 합니다. 결과적인 일관성(예; 복제된 데이터 저장소)을 구현하는 시스템에서는 이 경우가 아닐 수 있습니다.

응용 프로그램의 한 인스턴스는 데이터 항목을 수정하고 해당 항목의 캐시된 버전을 무효화할 수 있습니다. 응용 프로그램의 또 다른 인스턴스는 캐시 누락을 일으키는 캐시에서 이 항목 읽기를 시도하여 데이터 저장소에서 데이터를 읽고 캐시에 추가할 수 있습니다. 그러나 데이터 저장소가 다른 복제본과 완벽하게 동기화되지 않은 경우 응용 프로그램 인스턴스는 이전 값과 함께 캐시를 읽고 채울 수 있습니다.

데이터 일관성을 처리하는 방법에 대한 자세한 내용은 [데이터 일관성 입문서](http://msdn.microsoft.com/library/dn589800.aspx)를 참조하세요.

### <a name="protect-cached-data"></a>캐시된 데이터 보호
사용한 캐시 서비스와 관련 없이 무단 액세스에서 캐시에 보관된 데이터를 보호하는 방법을 고려하세요. 두 개의 주요 관심사가 있습니다.

* 캐시 데이터의 개인 정보 보호
* 캐시와 캐시를 사용하는 응용 프로그램 간에 흐르는 데이터의 개인 정보 보호

캐시의 데이터를 보호하기 위해 캐시 서비스에서 응용 프로그램이 다음을 지정하도록 요구하는 인증 메커니즘을 구현할 수 있습니다.

* 캐시 데이터에 액세스할 수 있는 ID
* 이러한 ID가 수행할 수 있는 작업(읽기 및 쓰기)

읽기 및 쓰기 데이터와 관련된 오버헤드를 줄이려면 ID에 캐시의 읽기 및/또는 쓰기 권한을 부여한 후에 ID가 캐시에서 모든 데이터를 사용할 수 있게 합니다.

캐시된 데이터의 하위 집합에 대한 액세스를 제한해야 하는 경우 다음 중 하나를 수행할 수 있습니다.

* 다양한 캐시 서버를 사용하여 캐시를 파티션으로 분할하고 사용하도록 해야 하는 파티션에 대한 ID에 액세스 권한을 부여합니다.
* 다른 키를 사용하여 각 하위 집합에서 데이터를 암호화하고 각 하위 집합에 액세스해야 하는 ID에만 암호화 키를 제공합니다. 클라이언트 응용 프로그램은 캐시에서 모든 데이터를 검색할 수 있지만 키를 가진 경우 데이터를 해독할 수 있습니다.

또한 데이터가 캐시 안팎으로 이동할 때 해당 데이터를 보호해야 합니다. 이를 위해 클라이언트 응용 프로그램에서 캐시에 연결하는 데 사용하는 네트워크 인프라에서 제공하는 보안 기능을 사용해야 합니다. 클라이언트 응용 프로그램을 호스트하는 같은 조직 내의 온사이트 서버를 사용하여 캐시가 구현되면 추가 단계를 수행하기 위해 네트워크 자체의 격리가 필요하지 않을 수 있습니다. 캐시가 원격에 위치하고 공용 네트워크를 통해 TCP 또는 HTTP 연결(인터넷과 같은)이 필요한 경우 SSL을 구현하는 것이 좋습니다.

## <a name="considerations-for-implementing-caching-with-microsoft-azure"></a>Microsoft Azure로 캐싱을 구현하기 위한 고려 사항

[Azure Redis Cache](/azure/redis-cache/)는 Azure 데이터 센터에서 서비스로 실행되는 오픈 소스 Redis 캐시를 구현합니다. 응용 프로그램이 웹사이트, 클라우드 서비스로 구현되거나 Azure 가상 머신 내에서 구현되거나 무관히 모든 Azure 응용 프로그램에서 액세스할 수 있는 캐싱 서비스를 제공합니다. 적절한 액세스 키가 있는 클라이언트 응용 프로그램이 캐시를 공유할 수 있습니다.

Azure Redis Cache는 가용성, 확장성 및 보안을 제공하는 고성능 캐싱 솔루션입니다. 일반적으로 하나 이상의 전용 컴퓨터에 분산된 서비스로 실행됩니다. 빠른 액세스를 보장하기 위해 메모리에 가능한 한 많은 정보를 저장하려고 합니다. 이 아키텍처는 느린 I/O 작업을 수행할 필요를 줄여 짧은 대기 시간과 높은 처리량을 제공하도록 합니다.

 Azure Redis Cache는 클라이언트 응용 프로그램에서 사용되는 많고 다양한 API와 호환됩니다. 온-프레미스를 실행하는 Azure Redis Cache를 이미 사용하는 기존 응용 프로그램이 있는 경우 Azure Redis Cache는 클라우드에서 캐시에 빠른 마이그레이션 경로를 제공합니다.


### <a name="features-of-redis"></a>Redis의 기능
 Redis는 간단한 캐시 서버 이상입니다. 여러 일반적인 시나리오를 지원하는 광범위한 명령 집합과 함께 분산된 메모리 내 데이터베이스를 제공합니다. 이러한 내용은 이 문서의 뒷부분, Redis 캐싱 사용 섹션에 설명되어 있습니다. 이 섹션은 Redis를 제공하는 주요 기능 중 일부를 요약합니다.

### <a name="redis-as-an-in-memory-database"></a>메모리 내 데이터베이스인 Redis
Redis는 읽기 및 쓰기 작업을 둘 다 지원합니다. Redis에서 쓰기는 로컬 스냅숏 파일 또는 추가 전용 로그 파일 중 하나에 주기적으로 저장되어 시스템 오류로부터 보호될 수 있습니다. 많은 캐시(일시적인 데이터 저장소로 고려해야 함)는 이와 다른 경우입니다.

 모든 쓰기는 비동기적이며 클라이언트가 데이터를 읽고 쓰는 것을 차단하지 않습니다. Redis이 실행되기 시작하는 경우 스냅숏 또는 로그 파일에서 데이터를 읽고 메모리 내 캐시를 생성하는데 사용합니다. 자세한 내용은 Redis 웹 사이트에서 [Redis 지속성](http://redis.io/topics/persistence)을 참조하세요.

> [!NOTE]
> Redis는 치명적인 오류가 발생한 경우 모든 쓰기가 반드시 저장된다고 할 수 없지만 최악의 경우 몇 초 분량의 데이터만 손실될 수 있습니다. 캐시는 신뢰할 수 있는 데이터 소스의 역할을 하며 캐시를 사용하여 중요한 데이터를 적절한 데이터 저장소에 성공적으로 저장하도록 하는 것이 응용 프로그램의 책임입니다. 자세한 내용은 [캐시 배제 패턴](http://msdn.microsoft.com/library/dn589799.aspx)을 참조하세요.
> 
> 

#### <a name="redis-data-types"></a>Redis 데이터 형식
Redis는 해시, 목록 및 집합 같은 단순 형식 또는 복잡한 데이터 구조를 포함할 수 있는 키-값 저장소를 설정합니다. 이러한 데이터 유형에 대한 소규모 작업 집합을 지원합니다. 키는 영구적이거나 라이브가 제한된 시간으로 태그되어 그 시점에 키 및 해당 값이 자동으로 캐시에서 제거됩니다. Redis 키와 값에 대한 자세한 내용은 Redis 웹 사이트에서 [Redis 데이터 형식 및 추상화 소개](http://redis.io/topics/data-types-intro) 페이지를 방문하세요.

#### <a name="redis-replication-and-clustering"></a>Redis 복제 및 클러스터링
Redis는 가용성을 보장하고 처리량을 유지하도록 마스터/하위 복제를 지원합니다. Redis 마스터 노드에 대한 쓰기 작업이 하나 이상의 하위 노드에 복제됩니다. 읽기 작업은 마스터 또는 하위 항목 중 하나에서 제공될 수 있습니다.

네트워크 분할이 발생하는 경우 연결이 다시 설정될 때 부하 직원은 데이터를 제공하고 그런 다음 투명하게 마스터와 재동기화합니다. 자세한 내용은 Redis 웹 사이트에서 [복제](http://redis.io/topics/replication) 페이지를 방문하십시오.

또한 Redis는 클러스터링을 제공하여 데이터를 서버의 분할로 투명하게 분할하고 로드를 분산시킬 수 있습니다. 이 기능은 캐시 크기의 증가에 따라 새 Redis 서버가 추가될 수 있고 데이터가 다시 분할되므로 확장성을 향상합니다.

또한 클러스터의 각 서버는 마스터/하위 복제를 사용하여 복제할 수 있습니다. 이렇게 하면 클러스터의 각 노드에 걸친 가용성을 보장합니다. 클러스터링 및 분할에 대한 자세한 내용은 Redis 웹 사이트에서 [Redis 클러스터 자습서 페이지](http://redis.io/topics/cluster-tutorial)를 방문하세요.

### <a name="redis-memory-use"></a>Redis 메모리 사용
Redis 캐시는 호스트 컴퓨터에서 사용할 수 있는 리소스에 따라 한정된 크기가 있습니다. Redis 서버를 구성할 때 사용할 수 있는 메모리의 최대 크기를 지정할 수 있습니다. 또한 Redis 캐시에 있는 키에 만료 시간이 있도록 구성할 수 있으며 그 이후 캐시에서 자동으로 제거됩니다. 이 기능은 메모리 내 캐시가 오래되거나 유효하지 않은 데이터로 채워지지 않도록 방지할 수 있습니다.

메모리가 차면 Redis는 정책의 수에 따라서 자동으로 키와 값을 제거할 수 있습니다. 기본값은 LRU(오래 전에 사용한 항목)이지만 임의로 키 제거 또는 제거 해제와 같은 기타 정책을 선택할 수 있습니다(이 경우 캐시가 가득 차면 여기에 항목을 추가하는 시도는 실패함). 자세한 정보는 [Redis를 LRU 캐시로 사용](http://redis.io/topics/lru-cache) 페이지를 참조하세요.

### <a name="redis-transactions-and-batches"></a>Redis 트랜잭션 및 배치
Redis는 클라이언트 응용 프로그램을 활성화하여 캐시의 데이터를 원자성 트랜잭션으로 읽고 쓰는 일련의 작업을 제출합니다. 트랜잭션에서 모든 명령은 순차적으로 실행되도록 보장되고 다른 동시 클라이언트가 발급한 어떤 명령도 이들 간에 섞이지 않습니다.

그러나 관계형 데이터베이스가 이것을 수행하는 경우 true 트랜잭션이 아닙니다. 트랜잭션 처리는 두 단계로 구성됩니다. 첫 번째는 명령이 대기할 때이고 두 번째는 명령이 실행될 때입니다. 명령 큐 단계 동안 클라이언트가 트랜잭션을 구성하는 명령을 전송합니다. 이 시점에서 일종의 오류가 발생하는 경우(구문 오류 또는 잘못된 매개 변수 개수 등) Redis는 전체 트랜잭션 처리를 거부하고 삭제합니다.

실행 단계 동안 Redis는 큐에 대기 중인 각 명령을 차례로 수행합니다. 이 단계 동안 명령이 실패한 경우 Redis는 큐에 대기 중인 다음 명령을 사용하여 계속되고 이미 실행된 명령의 효과를 롤백하지 않습니다. 이 간단해진 트랜잭션의 형태가 성능을 유지하고 경합으로 발생하는 성능 문제를 방지합니다.

Redis는 낙관적 잠금이라는 형태를 구현하여 일관성 유지를 지원합니다. Redis를 이용한 트랜잭션 및 잠금에 대한 자세한 정보는 Redis 웹 사이트에서 [트랜잭션 페이지](http://redis.io/topics/transactions)를 참조하십시오.

또한 Redis는 요청의 비 트랜잭션 배치를 지원합니다. Redis 서버에 명령을 보내기 위해 클라이언트가 사용하는 Redis 프로토콜을 사용하면 클라이언트가 동일한 요청의 일부로써 일련의 작업을 보냅니다. 이는 네트워크에서 패킷 조각화를 줄일 수 있습니다. 배치가 처리될 때 각 명령이 수행됩니다. 이러한 명령 중 하나라도 형식이 잘못된 경우 거부되지만(트랜잭션으로 발생하지 않은) 나머지 명령은 수행됩니다. 또한 배치의 명령이 처리되는 순서에 대해서 보장할 수 없습니다.

### <a name="redis-security"></a>Redis 보안
Redis는 빠른 데이터 액세스 제공에 집중되었으며 신뢰할 수 있는 클라이언트만 액세스할 수 있는 신뢰할 수 있는 환경 내에서 실행되도록 설계되었습니다. Redis는 암호 인증을 기반으로 하는 제한된 보안 모델을 지원합니다. (권장하지 않지만 인증을 완전히 제거할 수는 있음)

모든 인증된 클라이언트는 같은 전역 암호를 공유하고 동일한 리소스에 액세스합니다. 더 포괄적인 로그인 보안이 필요한 경우 Redis 서버 앞에 사용자 고유의 보안 계층을 구현해야 하며 모든 클라이언트 요청이 이 추가 계층을 통해 전달되어야 합니다. Redis는 신뢰할 수 없는 또는 인증되지 않은 클라이언트에 직접 노출되지 않아야 합니다.

명령을 사용하지 않도록 하거나 이름을 바꾸고 권한 있는 클라이언트만 새 이름으로 제공하여 액세스를 제한할 수 있습니다.

Redis는 모든 형태의 데이터 암호화를 직접 지원하지 않으므로 모든 인코딩을 클라이언트 응용 프로그램이 수행해야 합니다. 또한 Redis는 어떤 형태의 전송 보안도 제공하지 않습니다. 네트워크를 통해 흐르는 데이터를 보호해야 하는 경우 SSL 프록시를 구현하는 것이 좋습니다.

자세한 내용은 Redis 웹 사이트에서 [Redis 보안](http://redis.io/topics/security) 페이지를 방문하세요.

> [!NOTE]
> Azure Redis Cache는 클라이언트가 연결하는 자체 보안 계층을 제공합니다. 기본 Redis 서버는 공용 네트워크에 노출되지 않습니다.
> 
> 

### <a name="azure-redis-cache"></a>Azure Redis Cache
Azure Redis Cache는 Azure 데이터 센터에서 호스트되는 Redis 서버에 대한 액세스를 제공합니다. 액세스 제어 및 보안을 제공하는 외관의 역할을 합니다. Azure 포털을 사용하여 캐시를 프로비전할 수 있습니다.

포털에서 다양한 미리 정의된 구성을 제공합니다. 이 구성은 SSL 통신(개인) 및 99.9%의 가용성 SLA와 마스터/하위 복제를 지원하는 전용 서비스로 실행하는 53GB 캐시에서 공유 하드웨어에서 실행하는 복제(가용성을 보장하지 않음)를 지원하지 않는 250MB 캐시까지의 범위에 이릅니다.

Azure 포털을 사용하여 캐시의 제거 정책을 구성할 수도 있고 제공된 역할에 사용자를 추가하여 캐시에 액세스를 제어합니다.  멤버가 수행할 수 있는 작업이 정의된 역할에는 소유자, 기여자 및 독자가 포함됩니다. 예를 들어 소유자 역할의 멤버는 캐시(보안 포함) 및 해당 콘텐츠를 완전히 제어하고 기여자 역할의 멤버는 캐시에서 정보를 읽고 쓸 수 있고 읽기 역할의 멤버는 캐시에서 데이터를 검색할 수만 있습니다.

대부분의 관리 작업은 Azure 포털을 통해 수행됩니다. 이러한 이유로 Redis의 표준 버전에서 사용할 수 있는 관리 명령의 대부분은 사용할 수 없습니다. 여기에는 프로그래밍 방식으로 구성을 수정하고 Redis 서버를 종료하며 디스크에 추가적인 종속 장치를 구성하거나 강제로 데이터를 저장하는 기능이 포함됩니다.

Azure 포털은 캐시의 성능을 모니터링할 수 있도록 편리한 그래픽을 포함하여 표시됩니다. 예를 들어 수행되는 연결 수, 수행되는 요청 수, 읽기 및 쓰기 볼륨, 캐시 적중 수 대 캐시 누락 수를 볼 수 있습니다. 이 정보를 사용하여 캐시의 효과를 확인하고 필요하면 다른 구성으로 전환하거나 제거 정책을 변경할 수 있습니다.

또한 하나 이상의 중요한 메트릭이 예상되는 범위를 벗어나면 관리자에게 전자 메일 메시지를 보내는 경고를 만들 수 있습니다. 예를 들어 캐시 누락 수가 지난 시간에서 지정된 값을 초과하는 경우 캐시가 너무 작거나 데이터가 너무 빨리 제거되고 있다는 의미이므로 관리자에게 알리는 것이 좋습니다.

또한 CPU, 메모리 및 캐시용 네트워크 사용량을 모니터링할 수 있습니다.

Azure Redis Cache 만들기 및 구성 방법을 보여주는 자세한 내용 및 예제는 Azure 블로그의 [Azure Redis Cache 살펴보기](https://azure.microsoft.com/blog/2014/06/04/lap-around-azure-redis-cache-preview/) 페이지를 방문하십시오.

## <a name="caching-session-state-and-html-output"></a>캐싱 세션 상태 및 HTML 출력
Azure 웹 역할을 사용하여 실행하는 ASP.NET 웹 응용 프로그램을 구축하면 Azure Redis Cache에서 세션 상태 정보 및 HTML 출력을 저장할 수 있습니다. Azure Redis Cache용 세션 상태 제공자를 사용하면 ASP.NET 웹 응용 프로그램의 다른 인스턴스 간에 세션 정보를 공유할 수 있고 클라이언트-서버 선호도를 사용할 수 없는 경우 웹 팜의 경우에 매우 유용합니다. 메모리 내 캐싱 세션 데이터는 적절하지 않습니다.

Azure Redis Cache와 세션 상태 제공자를 사용하면 다음을 포함하여 다양한 이점이 있습니다.

* ASP.NET 웹 응용 프로그램의 많은 인스턴스와 세션 상태를 공유합니다.
* 향상된 확장성을 제공합니다.
* 여러 판독기와 단일 작성기용 동일한 세션 상태 데이터에 제어된 동시 액세스를 지원합니다.
* 압축을 사용하여 메모리를 절약하고 네트워크 성능을 향상시킵니다.

자세한 내용은 [Azure Redis Cache에 대한 ASP.NET 세션 상태 제공자](/azure/redis-cache/cache-aspnet-session-state-provider/)를 참조하세요.

> [!NOTE]
> Azure 환경 외부에서 실행되는 ASP.NET 응용 프로그램과 함께 Azure Redis Cache에 세션 상태 제공자를 사용하지 마세요. Azure 외부에서 캐시에 액세스하는 대기 시간은 데이터를 캐시하는 성능 혜택을 제거할 수 있습니다.
> 
> 

마찬가지로, Azure Redis Cache용 출력 캐시 공급자를 사용하면 ASP.NET 웹 응용 프로그램이 생성한 HTTP 응답을 저장할 수 있습니다. Azure Redis Cache와 출력 캐시 공급자를 사용하면 복잡한 HTML 출력을 렌더링하는 응용 프로그램의 응답 시간을 향상시킬 수 있습니다. 유사한 응답을 생성하는 응용 프로그램 인스턴스는 이 HTML 출력을 새로 생성하기 보다 캐시에서 공유 출력 조각을 사용할 수 있습니다. 자세한 내용은 [Azure Redis Cache에 대한 ASP.NET 출력 캐시 공급자](/azure/redis-cache/cache-aspnet-output-cache-provider/)를 참조하세요.

## <a name="building-a-custom-redis-cache"></a>사용자 지정 Redis 캐시 빌드
Azure Redis Cache는 기본 Redis 서버에 외관의 역할을 합니다. 이것은 현재 구성의 고정된 집합을 지원하지만 Redis 클러스터링을 위해 제공되지 않습니다. Azure Redis Cache가 다루지 않는 고급 구성이 필요한 경우(예: 53GB보다 더 큰 캐시) Azure 가상 머신을 사용하여 사용자 고유의 Redis 서버를 빌드 및 호스트할 수 있습니다.

복제를 구현하려는 경우 마스터 및 하위 노드 역할을 수행할 여러 VM을 만들어야 할 수 있기 때문에 이는 잠재적으로 복잡한 프로세스입니다. 또한 클러스터를 만들려는 경우 여러 마스터와 하위 서버가 필요합니다. 높은 수준의 가용성과 확장성을 제공하는 최소한의 클러스터된 복제 토폴로지는 세 쌍의 마스터/하위 서버로 구성된 6개 이상의 VM을 구성합니다(클러스터에 3개 이상의 마스터 노드를 포함해야 함).

각 마스터/하위 쌍은 대기 시간을 최소화하기 위해 서로 가까이 배치되어야 합니다. 그러나 이를 사용할 가능성이 큰 응용 프로그램에 가까운 캐시된 데이터를 찾으려는 경우 다른 지역에 있는 다른 Azure 데이터 센터에서 쌍의 각 집합을 실행할 수 있습니다.  빌드 및 Azure VM으로 실행되는 Redis 노드를 빌드 및 구성하는 방법의 예제를 보려면 [Azure의 CentOS Linux VM에서 Redis 실행](http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx)을 참조하세요.

> [!NOTE]
> 이러한 방식으로 사용자 고유의 Redis 캐시를 구현하는 경우 서비스 모니터링, 관리 및 보안에 책임이 있습니다.
> 

## <a name="partitioning-a-redis-cache"></a>Redis 캐시 분할
캐시 분할은 여러 컴퓨터에서 캐시를 나누어는 것이 포함됩니다. 이 구조는 단일 캐시 서버를 사용하며 다음을 포함하여 여러 장점이 있습니다.

* 훨씬 큰 캐시를 만들면 단일 서버에 저장할 수 있습니다.
* 가용성 향상하여 서버에 걸쳐 데이터를 배포합니다. 하나의 서버가 실패하거나 액세스할 수 없게 되면 보유한 데이터는 사용할 수 없지만 나머지 서버에 있는 데이터에는 여전히 액세스할 수 있습니다. 캐시의 경우에는 캐시된 데이터가 데이터베이스에 저장된 데이터의 임시 복사본일 뿐이므로 중요하지 않습니다. 액세스할 수 없게 되는 서버의 캐시된 데이터는 대신 다른 서버에서 캐시될 수 있습니다.
* 성능 및 확장성을 향상하여 서버에 걸쳐 부하를 분산합니다.
* 지리 위치 데이터가 액세스하는 사용자에게 가까우므로 대기 시간이 감소합니다.

캐시에 대해 분할의 가장 일반적인 형태가 분할됩니다. 이 전략에서 각 파티션(또는 분할된 데이터베이스)은 그 자체로 Redis 캐시입니다. 데이터는 분할 논리를 사용하여 특정 파티션으로 전송하여 데이터를 분산하는 데 다양한 방식을 사용할 수 있습니다. [분할 패턴](http://msdn.microsoft.com/library/dn589797.aspx)은 분할을 구현하는 방법에 대한 자세한 정보를 제공합니다.

Redis 캐시에서 분할을 구현하려면 다음 방법 중 하나를 선택할 수 있습니다.

* *서버측 쿼리 라우팅입니다.* 이 기술에서 클라이언트 응용 프로그램은 캐시를 구성하는 Redis 서버 중 하나에 요청을 보냅니다.(아마도 가장 가까운 서버) 각 Redis 서버는 보유한 분할을 설명하는 메타데이터를 저장하며 또한 다른 서버에 위치한 분할에 대한 정보를 포함합니다. Redis 서버는 클라이언트 요청을 검사합니다. 로컬로 해결할 수 있는 경우 요청된 작업을 수행합니다. 그렇지 않으면 적합한 서버로 요청을 전달합니다. 이 모델은 Redis 클러스터링이 구현하고 Redis 웹 사이트의 [Redis 클러스터 자습서](http://redis.io/topics/cluster-tutorial) 페이지에서 자세히 설명합니다. Redis 클러스터링은 클라이언트 응용 프로그램에 대해 투명하며, 클라이언트를 다시 구성하지 않고도 Redis 서버를 클러스터 및 다시 분할된 데이터에 추가할 수 있습니다.
* *클라이언트측 분할입니다.* 이 모델에서 클라이언트 응용 프로그램은 적절한 Redis 서버에 요청을 라우팅하는 논리(아마도 라이브러리 형식으로)를 포함합니다. 이 방법은 Azure Redis Cache와 함께 사용할 수 있습니다. 여러 Azure Redis Cache(각 데이터 파티션에 하나)를 만들고 올바른 캐시에 요청을 라우트하는 클라이언트 쪽 논리를 구현할 수 있습니다. 분할 체계가 변경되면(예를 들어 추가로 Azure Redis Cache를 생성하는 경우) 클라이언트 응용 프로그램을 다시 구성해야 합니다.
* *프록시 지원 분할입니다.* 이 스키마에서 클라이언트 응용 프로그램은 송신 데이터를 분할하는 방법을 이해하는 중간 프록시 서비스에 요청을 보낸 다음 Redis 서버에 적절한 요청을 라우팅합니다. 또한 이 방법은 Azure Redis Cache와 함께 사용할 수 있습니다. 프록시 서비스는 Azure 클라우드 서비스로 구현할 수 있습니다. 이 접근 방법을 사용하려면 서비스를 구현하기 위해 추가적으로 복잡한 수준이 필요하고 요청도 클라이언트 쪽 분할을 사용하는 것보다 오래 걸릴 수 있습니다.

웹사이트는 Redis의 [분할: 여러 Redis 인스턴스 간에 데이터를 분할하는 방법](http://redis.io/topics/partitioning) 페이지는 Redis와 분할을 구현하는 추가 정보를 제공합니다.

### <a name="implement-redis-cache-client-applications"></a>Redis 캐시 클라이언트 응용 프로그램 구현
Redis는 다양한 프로그래밍 언어로 작성된 클라이언트 응용 프로그램을 지원합니다. .NET Framework를 사용하여 새 응용 프로그램을 빌드하면 StackExchange.Redis 클라이언트 라이브러리를 사용하는 것이 좋습니다. 이 라이브러리는 Redis 서버에 연결, 명령 전송 및 응답 수신에 대한 세부 정보를 끌어내는 .NET Framework 개체 모델을 제공합니다. Visual Studio에서 NuGet 패키지로 사용할 수 있습니다. Azure Redis Cache 또는 VM에서 호스트되는 사용자 지정 Redis 캐시에 연결하려면 이 동일한 라이브러리를 사용할 수 있습니다.

Redis 서버에 연결하려면 `ConnectionMultiplexer` 클래스의 정적 `Connect` 메서드를 사용합니다. 이 메서드가 만드는 연결은 클라이언트 응용 프로그램의 전체 수명 동안 사용할 수 있도록 디자인되었으며 같은 연결이 여러 동시 스레드에서 사용될 수 있습니다. 성능을 저하시킬 수 있으므로 Redis 작업을 수행할 때마다 다시 연결하고 끊지 마세요.

Redis 호스트 및 암호의 주소와 같은 연결 매개 변수를 지정할 수 있습니다. Azure Redis Cache를 사용하면 암호는 Azure 관리 포털을 사용하여 Azure Redis Cache용으로 생성된 기본 또는 보조 키입니다.

Redis 서버에 연결한 후 캐시의 역할을 하는 Redis 데이터베이스에 핸들을 가져올 수 있습니다. Redis 연결이 이를 수행하는 `GetDatabase` 메서드를 제공합니다. 그런 다음 `StringGet` 및 `StringSet` 메서드를 사용하여 캐시에서 항목을 검색하고 캐시에 데이터를 저장할 수 있습니다. 이러한 메서드는 키를 매개 변수로 예상하고 일치하는 값을 갖는 캐시에서 항목을 반환하거나(`StringGet`) 이 키로 캐시에 항목을 추가합니다(`StringSet`).

Redis 서버 위치에 따라 요청이 서버에 전송되고 응답이 클라이언트에 반환되는 동안 많은 작업이 약간의 대기 시간을 발생시킬 수 있습니다. StackExchange 라이브러리는 클라이언트 응용 프로그램이 응답을 하도록 노출하는 많은 매서드의 비동기 버전을 제공합니다. 이러한 메서드는 .NET Framework에서 [작업 기반 비동기 패턴](http://msdn.microsoft.com/library/hh873175.aspx)을 지원합니다.

다음 코드 조각에 `RetrieveItem`이라는 메서드가 표시됩니다. Redis 및 StackExchange 라이브러리를 기반으로 한 캐시 배제 패턴의 구현 예를 보여 줍니다. `StringGetAsync` 메서드를 호출하여 메서드가 문자열 키 값을 사용하고 Redis 캐시에서 해당 항목을 검색하려고 시도합니다(`StringGet`의 비동기 버전).

항목이 없는 경우 `GetItemFromDataSourceAsync` 메서드(StackExchange 라이브러리의 일부가 아니고 로컬 메서드임)를 사용하여 기본 데이터 원본에서 가져옵니다. 그런 다음 `StringSetAsync` 메서드를 사용하여 캐시에 추가되어 다음에 더 신속히 검색할 수 있습니다.

```csharp
// Connect to the Azure Redis cache
ConfigurationOptions config = new ConfigurationOptions();
config.EndPoints.Add("<your DNS name>.redis.cache.windows.net");
config.Password = "<Redis cache key from management portal>";
ConnectionMultiplexer redisHostConnection = ConnectionMultiplexer.Connect(config);
IDatabase cache = redisHostConnection.GetDatabase();
...
private async Task<string> RetrieveItem(string itemKey)
{
    // Attempt to retrieve the item from the Redis cache
    string itemValue = await cache.StringGetAsync(itemKey);

    // If the value returned is null, the item was not found in the cache
    // So retrieve the item from the data source and add it to the cache
    if (itemValue == null)
    {
        itemValue = await GetItemFromDataSourceAsync(itemKey);
        await cache.StringSetAsync(itemKey, itemValue);
    }

    // Return the item
    return itemValue;
}
```

`StringGet` 및 `StringSet` 메서드는 문자열 값을 검색하거나 저장하는 데 제한되지 않습니다. 두 메서드에서 바이트 배열로 직렬화된 모든 항목을 사용할 수 있습니다. .NET 개체를 저장해야 하는 경우 바이트 스트림으로 직렬화하고 캐시에 쓰기 위해 `StringSet` 메서드를 사용할 수 있습니다.

마찬가지로, `StringGet` 메서드를 사용하여 캐시에서 개체를 읽을 수 있으며 .NET 개체로 역직렬화할 수 있습니다. 다음 코드는 IDatabase 인터페이스용 확장 메서드 집합(Redis 연결의 `GetDatabase` 메서드는 `IDatabase` 개체를 반환함) 및 `BlogPost` 개체를 캐시로 읽고 쓰기 위해 이러한 메서드를 사용하는 일부 샘플 코드를 보여 줍니다.

```csharp
public static class RedisCacheExtensions
{
    public static async Task<T> GetAsync<T>(this IDatabase cache, string key)
    {
        return Deserialize<T>(await cache.StringGetAsync(key));
    }

    public static async Task<object> GetAsync(this IDatabase cache, string key)
    {
        return Deserialize<object>(await cache.StringGetAsync(key));
    }

    public static async Task SetAsync(this IDatabase cache, string key, object value)
    {
        await cache.StringSetAsync(key, Serialize(value));
    }

    static byte[] Serialize(object o)
    {
        byte[] objectDataAsStream = null;

        if (o != null)
        {
            BinaryFormatter binaryFormatter = new BinaryFormatter();
            using (MemoryStream memoryStream = new MemoryStream())
            {
                binaryFormatter.Serialize(memoryStream, o);
                objectDataAsStream = memoryStream.ToArray();
            }
        }

        return objectDataAsStream;
    }

    static T Deserialize<T>(byte[] stream)
    {
        T result = default(T);

        if (stream != null)
        {
            BinaryFormatter binaryFormatter = new BinaryFormatter();
            using (MemoryStream memoryStream = new MemoryStream(stream))
            {
                result = (T)binaryFormatter.Deserialize(memoryStream);
            }
        }

        return result;
    }
}
```

다음 코드는 캐시 배제 패턴을 따라 직렬화 가능한 `BlogPost` 개체를 읽고 쓰는 데 이러한 확장 메서드를 사용하는 `RetrieveBlogPost`라는 메서드를 보여 줍니다.

```csharp
// The BlogPost type
[Serializable]
public class BlogPost
{
    private HashSet<string> tags;

    public BlogPost(int id, string title, int score, IEnumerable<string> tags)
    {
        this.Id = id;
        this.Title = title;
        this.Score = score;
        this.tags = new HashSet<string>(tags);
    }

    public int Id { get; set; }
    public string Title { get; set; }
    public int Score { get; set; }
    public ICollection<string> Tags => this.tags;
}
...
private async Task<BlogPost> RetrieveBlogPost(string blogPostKey)
{
    BlogPost blogPost = await cache.GetAsync<BlogPost>(blogPostKey);
    if (blogPost == null)
    {
        blogPost = await GetBlogPostFromDataSourceAsync(blogPostKey);
        await cache.SetAsync(blogPostKey, blogPost);
    }

    return blogPost;
}
```

클라이언트 응용 프로그램이 여러 비동기 요청을 보내면 Redis는 명령 파이프라인을 지원합니다. Redis는 엄격한 시퀀스의 명령을 수신 및 응답하기 보다 동일한 연결을 사용하여 요청을 멀티플렉싱할 수 있습니다.

이 방법은 네트워크를 보다 효율적으로 만들어 대기 시간을 줄일 수 있습니다. 다음 코드 조각은 동시에 두 고객의 세부 정보를 검색하는 예를 보여줍니다. 코드는 결과를 수신하려고 대기하기 전에 두 요청을 제출하고 다른 처리(표시되지 않음)를 수행합니다. 캐시 개체의 `Wait` 메서드는 .NET Framework `Task.Wait` 메서드와 비슷합니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
var task1 = cache.StringGetAsync("customer:1");
var task2 = cache.StringGetAsync("customer:2");
...
var customer1 = cache.Wait(task1);
var customer2 = cache.Wait(task2);
```

Azure Redis Cache를 수행할 수 있는 클라이언트 응용 프로그램 작성 방법에 대한 자세한 내용은 [Azure Redis Cache 설명서](https://azure.microsoft.com/documentation/services/cache/)를 참조하세요. [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Basics.md)에서도 자세한 내용을 확인할 수 있습니다.

동일한 웹 사이트의 [파이프라인 및 멀티플렉서](https://stackexchange.github.io/StackExchange.Redis/PipelinesMultiplexers) 페이지에서 Redis와 StackExchange 라이브러리를 통한 파이프라인 및 비동기 작업에 대한 자세한 정보를 제공합니다.  이 문서의 다음 섹션(Redis 캐시 사용)에서는 Redis 캐시에 저장된 데이터에 적용할 수 있는 고급 기술 중 일부를 예제로 제공합니다.

## <a name="using-redis-caching"></a>Redis 캐싱 사용
캐싱 문제에 대한 Redis의 가장 간단한 용도는 키-값 쌍이며 여기서 값은 이진 데이터를 포함할 수 있는 해석되지 않은 임의 길이의 문자열입니다. (기본적으로 문자열로 처리할 수 있는 바이트 배열임) 이 문서의 앞부분에 있는 Redis 캐시 클라이언트 응용 프로그램 구현 섹션에서 이 시나리오를 설명했습니다.

키에는 해석되지 않은 데이터도 포함되므로 모든 이진 정보를 키로 사용할 수 있습니다. 그러나 키가 길수록 차지하는 저장 공간이 더 많아지고 조회 작업을 수행하는 데 더 오래 걸립니다. 유용성 및 용이한 유지 관리를 위해 keyspace를 신중하게 디자인하고 의미는 있지만 자세한 정보를 표시하지 않은 키를 사용합니다.

예를 들어 "customer:100"과 같은 구조화된 키를 사용하여 단순히 "100"이 아닌 ID 100인 고객에 대한 키를 나타냅니다. 이 체계를 사용하면 다른 데이터 형식을 저장하는 값을 쉽게 구분할 수 있습니다. 예를 들어 "orders:100" 키를 사용하여 ID 100으로 순서에 대한 키를 나타낼 수 있습니다.

1차원 이진 문자열 외에도 Redis 키-값 쌍의 값은 목록, 집합(정렬 및 정렬되지 않은) 및 해시를 포함한 더 구조화된 정보를 보유할 수 있습니다. Redis는 이러한 형식을 조작할 수 있는 포괄적인 명령 집합을 제공하고 이 명령 중 다수는 StackExchange와 같은 클라이언트 라이브러리를 통해 .NET Framework 응용 프로그램에서 사용할 수 있습니다. Redis에 웹 사이트의 [Redis 데이터 형식 및 추상화 소개](http://redis.io/topics/data-types-intro) 페이지가 이를 조작하는데 사용할 수 있는 유형 및 명령의 자세한 개요를 제공합니다.

이 섹션은 이러한 데이터 형식 및 명령에 대한 일부 일반적인 사용 사례를 요약합니다.

### <a name="perform-atomic-and-batch-operations"></a>원자성 및 배치 작업 수행
Redis는 문자열 값에 대한 일련의 원자성 get-and-set 작업을 지원합니다. 별도 `GET` 및 `SET` 명령을 사용하는 경우 이러한 작업은 발생할 수 있는 경합 충격을 제거합니다. 사용할 수 있는 작업은 다음과 같습니다.

* 정수 숫자 데이터 값에 원자성 증가 및 감소 작업을 수행하는 `INCR`, `INCRBY`, `DECR` 및 `DECRBY`입니다. StackExchange 라이브러리는 `IDatabase.StringIncrementAsync` 및 `IDatabase.StringDecrementAsync` 메서드의 오버로드된 버전을 제공하여 이러한 작업을 수행하고 캐시에 저장된 결과 값을 반환합니다. 다음 코드 조각에서는 이러한 메서드를 사용하는 방법을 보여줍니다.
  
  ```csharp
  ConnectionMultiplexer redisHostConnection = ...;
  IDatabase cache = redisHostConnection.GetDatabase();
  ...
  await cache.StringSetAsync("data:counter", 99);
  ...
  long oldValue = await cache.StringIncrementAsync("data:counter");
  // Increment by 1 (the default)
  // oldValue should be 100
  
  long newValue = await cache.StringDecrementAsync("data:counter", 50);
  // Decrement by 50
  // newValue should be 50
  ```
* 키와 연결된 값을 검색하고 새 값으로 변경하는 `GETSET`입니다. StackExchange 라이브러리를 사용하면 이 작업을 `IDatabase.StringGetSetAsync` 방법을 통해 사용할 수 있습니다. 아래 코드 조각에 이 메서드의 예가 나와 있습니다. 이 코드는 이전 예제에서 "data:counter" 키와 연결된 현재 값을 반환합니다. 그런 다음 동일한 작업의 일부로 이 키의 값을 0으로 다시 설정합니다.
  
  ```csharp
  ConnectionMultiplexer redisHostConnection = ...;
  IDatabase cache = redisHostConnection.GetDatabase();
  ...
  string oldValue = await cache.StringGetSetAsync("data:counter", 0);
  ```
* 문자열 값의 집합을 단일 작업으로 반환하거나 변경할 수 있는 `MGET` 및 `MSET`입니다. `IDatabase.StringGetAsync` 및 `IDatabase.StringSetAsync` 방법은 다음 예와 같이 이 기능을 지원하기 위해 오버로드됩니다.
  
  ```csharp
  ConnectionMultiplexer redisHostConnection = ...;
  IDatabase cache = redisHostConnection.GetDatabase();
  ...
  // Create a list of key-value pairs
  var keysAndValues =
      new List<KeyValuePair<RedisKey, RedisValue>>()
      {
          new KeyValuePair<RedisKey, RedisValue>("data:key1", "value1"),
          new KeyValuePair<RedisKey, RedisValue>("data:key99", "value2"),
          new KeyValuePair<RedisKey, RedisValue>("data:key322", "value3")
      };
  
  // Store the list of key-value pairs in the cache
  cache.StringSet(keysAndValues.ToArray());
  ...
  // Find all values that match a list of keys
  RedisKey[] keys = { "data:key1", "data:key99", "data:key322"};
  // values should contain { "value1", "value2", "value3" }
  RedisValue[] values = cache.StringGet(keys);

  ```

이 문서의 Redis 트랜잭션 및 배치 섹션에서 설명한 대로 여러 작업을 단일 Redis 트랜잭션으로 결합할 수도 있습니다. StackExchange 라이브러리는 `ITransaction` 인터페이스를 통해 트랜잭션에 대한 지원을 제공합니다.

`IDatabase.CreateTransaction` 메서드를 사용하여 `ITransaction` 개체를 만듭니다. `ITransaction` 개체에서 제공하는 메서드를 사용하여 트랜잭션에 명령을 호출합니다.

모든 메서드가 비동기임을 제외하고 `ITransaction` 인터페이스는 `IDatabase` 인터페이스에서 액세스하는 메서드와 유사한 메서드 집합에 대한 액세스를 제공합니다.  즉, `ITransaction.Execute` 메서드가 호출될 때만 수행됩니다. `ITransaction.Execute` 메서드에 의해 반환되는 값은 트랜잭션이 성공적으로 만들어졌는지(true), 실패했는지(false) 여부를 나타냅니다.

다음 코드 조각은 동일한 트랜잭션의 일부로 두 카운터를 증가 및 감소시키는 예제를 보여줍니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
ITransaction transaction = cache.CreateTransaction();
var tx1 = transaction.StringIncrementAsync("data:counter1");
var tx2 = transaction.StringDecrementAsync("data:counter2");
bool result = transaction.Execute();
Console.WriteLine("Transaction {0}", result ? "succeeded" : "failed");
Console.WriteLine("Result of increment: {0}", tx1.Result);
Console.WriteLine("Result of decrement: {0}", tx2.Result);
```

Redis 트랜잭션은 관계형 데이터베이스의 트랜잭션과 다릅니다. `Execute` 메서드는 실행용 트랜잭션을 구성하는 모든 명령을 큐에 대기시키고 그 중 하나라도 잘못되면 트랜잭션이 중단됩니다. 모든 명령이 성공적으로 큐에 대기한 경우 각 명령은 비동기적으로 실행됩니다.

어떤 명령이 실패해도 나머지는 여전히 계속 처리됩니다. 명령이 성공적으로 완료되었는지 확인하려는 경우 해당 작업의 **Result** 속성을 사용하여 위의 예제와 같이 명령의 결과를 가져와야 합니다. **Result** 속성을 읽으면 작업을 완료할 때까지 호출 스레드를 차단합니다.

자세한 내용은 StackExchange.Redis 웹 사이트에서 [Redis에서 트랜잭션을](https://stackexchange.github.io/StackExchange.Redis/Transactions) 페이지를 참조하십시오.

배치 작업을 수행할 때 StackExchange 라이브러리의 `IBatch` 인터페이스를 사용할 수 있습니다. 이 인터페이스는 모든 메서드가 비동기 작업인 경우를 제외하고 `IDatabase` 인터페이스와 비슷한 메서드 집합에 액세스를 제공합니다.

다음 예제와 같이 `IDatabase.CreateBatch` 메서드를 사용하여 `IBatch` 개체를 만든 다음 `IBatch.Execute` 메서드를 사용하여 배치를 실행합니다. 이 코드는 단순히 문자열 값을 설정하고 이전 예제에서 사용한 동일한 카운터를 증가 및 감소시키며 결과를 표시합니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
IBatch batch = cache.CreateBatch();
batch.StringSetAsync("data:key1", 11);
var t1 = batch.StringIncrementAsync("data:counter1");
var t2 = batch.StringDecrementAsync("data:counter2");
batch.Execute();
Console.WriteLine("{0}", t1.Result);
Console.WriteLine("{0}", t2.Result);
```

트랜잭션과 달리 배치에서 명령의 형식이 잘못되어 실패한 경우 다른 명령은 계속 실행될 수 있다는 점을 이해해야 합니다. `IBatch.Execute` 메서드는 성공 또는 실패의 모든 표시를 반환하지 않습니다.

### <a name="perform-fire-and-forget-cache-operations"></a>실행 후 제거 캐시 작업 수행
Redis는 명령 플래그를 사용하여 실행 후 제거 작업을 지원합니다. 이 경우 클라이언트는 단순히 작업을 시작하지만 결과에 관심이 없고 명령이 완료되기를 기다리지 않습니다. 아래 예제에서는 INCR 명령을 실행 후 제거 작업으로 수행하는 방법을 보여 줍니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
await cache.StringSetAsync("data:key1", 99);
...
cache.StringIncrement("data:key1", flags: CommandFlags.FireAndForget);
```

### <a name="specify-automatically-expiring-keys"></a>자동 만료 키 지정
Redis 캐시에 항목을 저장하는 경우 어떤 항목을 자동으로 캐시에서 제거할지 시간 제한을 지정할 수 있습니다. `TTL` 명령을 사용하여 키가 만료되기 전에 얼마나 더 많은 시간이 필요한지 쿼리할 수도 있습니다. 이 명령은 `IDatabase.KeyTimeToLive` 메서드를 사용하여 StackExchange 응용 프로그램에 사용할 수 있습니다.

다음 코드 조각은 키에 20초로 만료 시간을 설정하고 키의 남은 수명을 쿼리하는 방법을 보여 줍니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
// Add a key with an expiration time of 20 seconds
await cache.StringSetAsync("data:key1", 99, TimeSpan.FromSeconds(20));
...
// Query how much time a key has left to live
// If the key has already expired, the KeyTimeToLive function returns a null
TimeSpan? expiry = cache.KeyTimeToLive("data:key1");
```

또한 EXPIRE 명령을 사용하여 특정 날짜 및 시간에 만료 시간을 설정할 수 있으며 이는 StackExchange 라이브러리의 `KeyExpireAsync` 메서드로 구현 가능합니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
// Add a key with an expiration date of midnight on 1st January 2015
await cache.StringSetAsync("data:key1", 99);
await cache.KeyExpireAsync("data:key1",
    new DateTime(2015, 1, 1, 0, 0, 0, DateTimeKind.Utc));
...
```

> [!TIP] 
> DEL 명령을 사용하여 캐시에서 항목을 수동으로 제거할 수 있으며 이는 `IDatabase.KeyDeleteAsync` 메서드로 StackExchange 라이브러리를 통해 사용할 수 있습니다.

### <a name="use-tags-to-cross-correlate-cached-items"></a>태그를 사용하여 캐시된 항목을 상호 비교
Redis 집합은 단일 키를 공유하는 여러 항목의 컬렉션입니다. SADD 명령을 사용하여 집합을 만들 수 있습니다. SMEMBERS 명령을 사용하여 집합의 항목을 검색할 수 있습니다. StackExchange 라이브러리는 `IDatabase.SetAddAsync` 메서드로 SADD 명령을 구현하고 `IDatabase.SetMembersAsync` 메서드로 SMEMBERS 명령을 구현합니다.

SDIFF(차집합), SINTER(교집합) 및 SUNION(합집합) 명령을 사용하여 새 집합을 만들려면 기존 세트를 결합할 수 있습니다. StackExchange 라이브러리는 `IDatabase.SetCombineAsync` 메서드에서 이러한 작업을 통합합니다. 이 메서드에 대한 첫 번째 매개 변수는 수행할 집합 작업을 지정합니다.

다음 코드 조각은 신속한 저장 및 관련된 항목의 컬렉션을 검색하는데 집합이 유용할 수 있음을 보여줍니다. 이 코드는 이 문서 앞부분의 Redis 캐시 클라이언트 응용 프로그램 구현 섹션에 설명된 `BlogPost` 형식을 사용합니다.

`BlogPost` 개체는 ID, 제목, 순위 점수 및 태그의 컬렉션 등 4개의 필드를 포함합니다. 아래 첫 번째 코드 조각에서 `BlogPost` 개체의 C# 목록을 채우기 위해 사용되는 샘플 데이터를 보여 줍니다.

```csharp
List<string[]> tags = new List<string[]>
{
    new[] { "iot","csharp" },
    new[] { "iot","azure","csharp" },
    new[] { "csharp","git","big data" },
    new[] { "iot","git","database" },
    new[] { "database","git" },
    new[] { "csharp","database" },
    new[] { "iot" },
    new[] { "iot","database","git" },
    new[] { "azure","database","big data","git","csharp" },
    new[] { "azure" }
};

List<BlogPost> posts = new List<BlogPost>();
int blogKey = 1;
int numberOfPosts = 20;
Random random = new Random();
for (int i = 0; i < numberOfPosts; i++)
{
    blogKey++;
    posts.Add(new BlogPost(
        blogKey,                  // Blog post ID
        string.Format(CultureInfo.InvariantCulture, "Blog Post #{0}",
            blogKey),             // Blog post title
        random.Next(100, 10000),  // Ranking score
        tags[i % tags.Count]));   // Tags--assigned from a collection
                                  // in the tags list
}
```

Redis 캐시의 집합으로 각 `BlogPost` 개체에 대한 태그를 저장하고 각 집합을 `BlogPost`의 ID와 연결할 수 있습니다. 이를 통해 응용 프로그램이 특정 블로그 게시물에 속한 모든 태그를 신속하게 찾을 수 있습니다. 반대 방향으로 검색을 수행하고 특정 태그를 공유하는 모든 블로그 게시물을 발견하려면 키에서 태그 ID를 참조하는 블로그 게시물을 보유한 다른 집합을 만들 수 있습니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
// Tags are easily represented as Redis Sets
foreach (BlogPost post in posts)
{
    string redisKey = string.Format(CultureInfo.InvariantCulture,
        "blog:posts:{0}:tags", post.Id);
    // Add tags to the blog post in Redis
    await cache.SetAddAsync(
        redisKey, post.Tags.Select(s => (RedisValue)s).ToArray());

    // Now do the inverse so we can figure how which blog posts have a given tag
    foreach (var tag in post.Tags)
    {
        await cache.SetAddAsync(string.Format(CultureInfo.InvariantCulture,
            "tag:{0}:blog:posts", tag), post.Id);
    }
}
```

이러한 구조를 사용하여 많은 일반적인 쿼리를 매우 효율적으로 수행할 수 있습니다. 예를 들어 다음과 같은 블로그 게시물 1에 대한 태그를 모두 표시할 수 있습니다.

```csharp
// Show the tags for blog post #1
foreach (var value in await cache.SetMembersAsync("blog:posts:1:tags"))
{
    Console.WriteLine(value);
}
```

다음과 같이 교집합 연산을 수행하여 블로그 게시물 1 및 2에 공통되는 모든 태그를 찾을 수 있습니다.

```csharp
// Show the tags in common for blog posts #1 and #2
foreach (var value in await cache.SetCombineAsync(SetOperation.Intersect, new RedisKey[]
    { "blog:posts:1:tags", "blog:posts:2:tags" }))
{
    Console.WriteLine(value);
}
```

특정 태그를 포함하는 모든 블로그 게시물을 찾을 수 있습니다.

```csharp
// Show the ids of the blog posts that have the tag "iot".
foreach (var value in await cache.SetMembersAsync("tag:iot:blog:posts"))
{
    Console.WriteLine(value);
}
```

### <a name="find-recently-accessed-items"></a>최근에 액세스된 항목 찾기
많은 응용 프로그램의 일반적인 작업은 가장 최근에 액세스한 항목을 찾는 것입니다. 예를 들어 사이트를 블로깅하면서 가장 최근에 읽은 블로그 게시물에 대한 정보를 표시하려 할 수 있습니다.

Redis 목록을 사용하여 이 기능을 구현할 수 있습니다. Redis 목록은 동일한 키를 공유하는 여러 항목을 포함합니다. 목록은 양쪽이 큐 역할을 합니다. LPUSH(왼쪽 밀어넣기) 및 RPUSH(오른쪽 밀어넣기) 명령을 사용하여 항목을 목록의 한쪽 끝에 푸시할 수 있습니다. LPOP 및 RPOP 명령을 사용하여 목록의 한쪽 끝에서 항목을 검색할 수 있습니다. 또한 LRANGE 및 RRANGE 명령을 사용하여 요소 집합을 반환할 수 있습니다.

아래 코드 조각은 StackExchange 라이브러리를 사용하여 이러한 작업을 수행하는 방법을 보여줍니다. 이 코드는 이전 예제의 `BlogPost` 형식을 사용합니다. 사용자가 블로그 게시물을 읽을 때 `IDatabase.ListLeftPushAsync` 메서드가 Redis 캐시의 "blog:recent_posts" 키와 연결된 목록에 블로그 게시물의 제목을 푸시합니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
string redisKey = "blog:recent_posts";
BlogPost blogPost = ...; // Reference to the blog post that has just been read
await cache.ListLeftPushAsync(
    redisKey, blogPost.Title); // Push the blog post onto the list
```

더 많은 블로그 게시물을 읽을수록 제목은 같은 목록에 푸시됩니다. 목록은 제목이 추가된 순서로 정렬됩니다. 가장 최근에 읽은 블로그 게시물은 목록의 왼쪽 끝 쪽에 위치합니다. (동일한 블로그 게시물이 두 번 이상 읽혀진 경우 목록에 여러 항목이 있게 됨)

`IDatabase.ListRange` 메서드를 사용하여 가장 최근에 읽은 게시물의 제목을 표시할 수 있습니다. 이 메서드는 목록, 시작점 및 끝점을 포함하는 키를 가져옵니다. 다음 코드는 목록의 가장 왼쪽 끝에서 10개의 블로그 게시물(0에서 9까지의 항목)의 제목을 검색합니다.

```csharp
// Show latest ten posts
foreach (string postTitle in await cache.ListRangeAsync(redisKey, 0, 9))
{
    Console.WriteLine(postTitle);
}
```

`ListRangeAsync` 메서드가 목록에서 항목을 제거하지 않습니다. 이를 위해 `IDatabase.ListLeftPopAsync` 및 `IDatabase.ListRightPopAsync` 메서드를 사용할 수 있습니다.

목록이 무한정 커지지 않도록 하려면 목록을 트리밍하여 항목을 주기적으로 추려내 수 있습니다. 아래 코드 조각은 목록에서 가장 왼쪽의 5개 항목을 제외하고 모두 제거하는 방법을 보여 줍니다.

```csharp
await cache.ListTrimAsync(redisKey, 0, 5);
```

### <a name="implement-a-leader-board"></a>리더 보드 구현
기본적으로 집합에서 항목은 특정 순서로 유지되지 않습니다. ZADD 명령을 사용하여 정렬된 집합을 만들 수 있습니다(StackExchange 라이브러리의 `IDatabase.SortedSetAdd` 메서드). 명령에 매개 변수로 제공된 점수인 숫자 값을 사용하여 항목이 정렬됩니다.

다음 코드 조각은 정렬된 목록에 블로그 게시물의 제목을 추가합니다. 예제에서 각 블로그 게시물에 블로그 게시물의 순위를 포함하는 점수 필드도 있습니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
IDatabase cache = redisHostConnection.GetDatabase();
...
string redisKey = "blog:post_rankings";
BlogPost blogPost = ...; // Reference to a blog post that has just been rated
await cache.SortedSetAddAsync(redisKey, blogPost.Title, blogPost.Score);
```

`IDatabase.SortedSetRangeByRankWithScores` 메서드를 사용하여 블로그 게시물 제목과 점수를 점수 오름차순으로 검색할 수 있습니다.

```csharp
foreach (var post in await cache.SortedSetRangeByRankWithScoresAsync(redisKey))
{
    Console.WriteLine(post);
}
```

> [!NOTE]
> StackExchange 라이브러리가 점수 순서로 데이터를 반환하는 `IDatabase.SortedSetRangeByRankAsync` 메서드도 제공하지만 점수를 반환하지는 않습니다.
> 
> 

`IDatabase.SortedSetRangeByRankWithScoresAsync` 메서드에 추가 매개 변수를 제공하여 점수의 내림차순에서 항목을 검색하고 반환되는 항목 수를 제한할 수도 있습니다. 다음 예제에서 상위 10개의 블로그 게시물의 제목 및 점수를 표시합니다.

```csharp
foreach (var post in await cache.SortedSetRangeByRankWithScoresAsync(
                               redisKey, 0, 9, Order.Descending))
{
    Console.WriteLine(post);
}
```

다음 예제에서는 지정된 점수 범위 내에 있는 대상에게 반환되는 항목을 제한하기 위해 사용할 수 있는 `IDatabase.SortedSetRangeByScoreWithScoresAsync` 메서드가 사용됩니다.

```csharp
// Blog posts with scores between 5000 and 100000
foreach (var post in await cache.SortedSetRangeByScoreWithScoresAsync(
                               redisKey, 5000, 100000))
{
    Console.WriteLine(post);
}
```

### <a name="message-by-using-channels"></a>채널을 사용한 메시지
데이터 캐시의 역할 외에도 Redis 서버는 고성능 게시자/구독자 메커니즘을 통해 메시징을 제공합니다. 클라이언트 응용 프로그램은 채널을 구독할 수 있고 다른 응용 프로그램이나 서비스는 채널에 메시지를 게시할 수 있습니다. 응용 프로그램을 구독하면 이러한 메시지를 받고 처리할 수 있습니다.

Redis는 채널 구독에 사용하도록 클라이언트 응용 프로그램을 위한 SUBSCRIBE 명령을 제공합니다. 이 명령은 응용 프로그램이 메시지를 받는 하나 이상의 채널의 이름을 필요로 합니다. StackExchange 라이브러리에는 .NET Framework 응용 프로그램이 채널을 구독하고 채널에 게시할 수 있는 `ISubscription` 인터페이스가 포함됩니다.

Redis 서버를 연결하는 `GetSubscriber` 메서드를 사용하여 `ISubscription` 개체를 만듭니다. 그런 다음 이 개체의 `SubscribeAsync` 메서드를 사용하여 채널에서 메시지를 수신 대기합니다. 다음 코드 예제에서 "messages:blogPosts"라는 채널을 구독하는 방법을 보여줍니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
...
await subscriber.SubscribeAsync("messages:blogPosts", (channel, message) => Console.WriteLine("Title is: {0}", message));
```

`Subscribe` 메서드의 첫 번째 매개 변수는 채널의 이름입니다. 이 이름은 캐시의 키에서 사용되는 동일한 규칙을 따릅니다. 좋은 성능과 관리 효율을 보장하도록 비교적 짧고 의미 있는 문자열을 사용하는 것이 좋지만 이름에는 모든 이진 데이터가 포함될 수 있습니다.

또한 채널에서 사용되는 네임스페이스는 키에서 사용되는 네임스페이스와는 다릅니다. 즉, 응용 프로그램 코드를 유지하는 데 더 어렵게 만들 수도 있지만 동일한 이름의 채널 및 키를 가질 수 있습니다.

두 번째 매개 변수는 Action 대리자입니다. 이 대리자는 새 메시지가 채널에 나타날 때마다 비동기적으로 실행됩니다. 이 예에서 단순히 콘솔에 메시지를 표시합니다.(메시지는 블로그 게시물의 제목을 포함합니다)

채널에 게시하려면 응용 프로그램이 Redis PUBLISH 명령을 사용할 수 있습니다. StackExchange 라이브러리는 이 작업을 수행할 `IServer.PublishAsync` 메서드를 제공합니다. 다음 코드 조각에서 "messages:blogPosts" 채널에 메시지를 게시하는 방법을 보여줍니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
...
BlogPost blogPost = ...;
subscriber.PublishAsync("messages:blogPosts", blogPost.Title);
```

다음은 게시/구독 메커니즘에 대해 이해해야 할 몇 가지 사항입니다.

* 여러 구독자가 동일한 채널을 구독할 수 있고 해당 채널에 게시된 메시지를 받을 수 있습니다.
* 구독자는 구독한 후 게시된 메시지를 단지 수신합니다. 채널은 버퍼링되지 않으며 메시지가 게시되면 Redis 인프라가 각 구독자에게 메시지를 푸시한 다음 제거합니다.
* 기본적으로 구독자는 보낸 순서대로 메시지를 수신합니다. 메시지 및 많은 구독자와 게시자 다수를 포함한 매우 활발 시스템에서 메시지의 보장된 순차적인 배달은 시스템의 성능을 저하시킬 수 있습니다. 각 메시지가 독립적이며 순서가 중요하지 않은 경우 응답성을 향상시킬 수 있는 Redis 시스템에서 동시 처리를 설정할 수 있습니다. 구독자가 False에 사용하는 연결의 PreserveAsyncOrder를 설정하여 StackExchange 클라이언트에서 이를 달성할 수 있습니다.

```csharp
ConnectionMultiplexer redisHostConnection = ...;
redisHostConnection.PreserveAsyncOrder = false;
ISubscriber subscriber = redisHostConnection.GetSubscriber();
```

### <a name="serialization-considerations"></a>직렬화 시 고려 사항

직렬화 형식을 선택할 때는 성능, 상호 운용성, 버전 관리, 기존 시스템과의 호환성, 데이터 압축 및 메모리 오버 헤드 간의 상쇄를 고려합니다. 성능을 평가하는 경우 벤치마크가 컨텍스트에 따라 크게 달라진다는 점에 유의합니다. 실제 작업을 반영하지 않을 수 있으며, 최신 라이브러리 또는 버전을 고려하지 않을 수 있습니다. 모든 시나리오에 대해 "가장 빠른" 단일 직렬 변환기가 있는 것은 아닙니다. 

다음은 몇 가지 고려해야 할 옵션입니다.

- [프로토콜 버퍼](https://github.com/google/protobuf)(protobuf라고도 함)는 Google에서 구조화된 데이터를 효율적으로 직렬화하기 위해 개발한 직렬화 형식입니다. 여기서는 강력한 형식의 정의 파일을 사용하여 메시지 구조를 정의합니다. 이러한 정의 파일은 메시지의 직렬화 및 역직렬화를 위해 언어 관련 코드로 컴파일됩니다. Protobuf를 기존 RPC 메커니즘 대신 사용할 수도 있고, 이를 통해 RPC 서비스를 생성할 수도 있습니다.

- [Apache Thrift](https://thrift.apache.org/)는 강력한 형식의 정의 파일 및 컴파일 단계와 함께 유사한 방법을 사용하여 직렬화 코드 및 RPC 서비스를 생성합니다.  

- [Apache Avro](https://avro.apache.org/)는 프로토콜 버퍼 및 Thrift에 유사한 기능을 제공하지만 컴파일 단계는 없습니다. 대신, 직렬화된 데이터는 항상 구조를 설명하는 스키마를 포함합니다. 

- [JSON](http://json.org/)은 인간이 이해하기 쉬운 텍스트 필드를 사용하는 개방형 표준입니다. 광범위한 플랫폼 간 지원을 제공합니다. JSON은 메시지 스키마를 사용하지 않습니다. 텍스트 기반 형식이기 때문에, 네트워크상에서는 별로 효율적이지 않습니다. 그러나 경우에 따라, HTTP를 통해 클라이언트로 직접 캐시된 항목을 반환할 수 있습니다. 이 경우 JSON을 저장하면 다른 형식에서 역직렬화했다가 JSON으로 직렬화하는 비용을 절감할 수 있습니다.

- [BSON](http://bsonspec.org/)은 JSON과 유사한 구조를 사용하는 이진 직렬화 형식입니다. BSON은 JSON에 비해 가볍고, 검색이 용이하고, 직렬화 및 역직렬화가 빠르도록 디자인되었습니다. 페이로드는 JSON에 크기와 비슷합니다. 데이터에 따라, BSON 페이로드는 JSON 페이로드보다 작거나 클 수 있습니다. BSON은 JSON에서는 사용할 수 없는 몇 가지 추가 데이터 형식을 갖습니다. 특히 BinData(바이트 배열) 및 Date가 여기에 포함됩니다.

- [MessagePack](http://msgpack.org/)은 네트워크를 통해 전송하기 위해 압축되도록 디자인된 이진 직렬화 형식입니다. 메시지 스키마 또는 메시지 형식 확인은 수행되지 않습니다.

- [Bond](https://microsoft.github.io/bond/)는 스키마화된 데이터로 작업하기 위한 플랫폼 간 프레임워크입니다. 이 프레임워크는 언어 간 직렬화 및 역직렬화를 지원합니다. 여기에 나열된 다른 시스템과의 중요한 차이점은 상속, 형식 별칭 및 제네릭에 대한 지원입니다. 

- [gRPC](http://www.grpc.io/)는 는 Google에서 개발한 오픈 소스 RPC 시스템입니다. 기본적으로 프로토콜 버퍼를 해당 정의 언어 및 기본 메시지 교환 형식으로 사용합니다.

## <a name="related-patterns-and-guidance"></a>관련 패턴 및 지침

또한 응용 프로그램에서 캐싱을 구현하는 경우 다음 패턴은 시나리오와 관련될 수 있습니다.

* [캐시 배제 패턴](http://msdn.microsoft.com/library/dn589799.aspx): 이 패턴은 요청 시 데이터를 데이터 저장소에서 캐시로 로드하는 방법을 설명합니다. 또한 이 패턴은 캐시에 저장된 데이터 및 원래 데이터 저장소의 데이터 간 일관성을 유지할 수 있습니다.
* [분할 패턴](http://msdn.microsoft.com/library/dn589797.aspx)은 많은 양의 데이터를 저장 및 액세스하는 경우 확장성을 향상시키는 수평 분할을 구현할 정보를 제공합니다.

## <a name="more-information"></a>자세한 정보
* Microsoft 웹 사이트의 [MemoryCache 클래스](http://msdn.microsoft.com/library/system.runtime.caching.memorycache.aspx) 페이지.
* Microsoft 웹 사이트의 [Azure Redis Cache 설명서](https://azure.microsoft.com/documentation/services/cache/) 페이지.
* Microsoft 웹 사이트의 [Azure Redis Cache FAQ](/azure/redis-cache/cache-faq) 페이지.
* Microsoft 웹 사이트의 [구성 모델](http://msdn.microsoft.com/library/windowsazure/hh914149.aspx) 페이지.
* Microsoft 웹 사이트의 [작업 기반 비동기 패턴](http://msdn.microsoft.com/library/hh873175.aspx) 페이지.
* StackExchange.Redis GitHub 리포지토리의 [파이프라인 및 멀티플렉서](https://stackexchange.github.io/StackExchange.Redis/PipelinesMultiplexers) 페이지.
* Redis 웹 사이트의 [Redis 지속성](http://redis.io/topics/persistence) 페이지.
* Redis 웹 사이트의 [복제 페이지](http://redis.io/topics/replication) .
* Redis 웹 사이트의 [Redis 클러스터 자습서](http://redis.io/topics/cluster-tutorial) 페이지.
* Redis 웹 사이트의 [분할: 여러 Redis 인스턴스 간에 데이터를 분할하는 방법](http://redis.io/topics/partitioning) 페이지.
* Redis 웹 사이트의 [Redis를 LRU 캐시로 사용](http://redis.io/topics/lru-cache) 페이지.
* Redis 웹 사이트의 [트랜잭션](http://redis.io/topics/transactions) 페이지.
* Redis 웹 사이트의 [Redis 보안](http://redis.io/topics/security) 페이지.
* Azure 블로그의 [Azure Redis Cache 둘러보기](https://azure.microsoft.com/blog/2014/06/04/lap-around-azure-redis-cache-preview/) 페이지.
* Microsoft 웹 사이트에서 [Azure의CentOS Linux VM에서 Redis 실행](http://blogs.msdn.com/b/tconte/archive/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure.aspx) 페이지.
* Microsoft 웹 사이트의 [Azure Redis Cache용 ASP.NET 세션 상태 제공자](/azure/redis-cache/cache-aspnet-session-state-provider) 페이지입니다.
* Microsoft 웹 사이트의 [Azure Redis Cache용 ASP.NET 출력 캐시 제공자](/azure/redis-cache/cache-aspnet-output-cache-provider) 페이지입니다.
* Redis 웹 사이트의 [Redis 데이터 형식 및 추상화 소개](http://redis.io/topics/data-types-intro) 페이지.
* StackExchange.Redis 웹 사이트의 [기본 사용법](https://stackexchange.github.io/StackExchange.Redis/Basics) 페이지.
* StackExchange.Redis 리포지토리의 [Redis에서 트랜잭션](https://stackexchange.github.io/StackExchange.Redis/Transactions) 페이지.
* Microsoft 웹 사이트의 [데이터 분할 가이드](http://msdn.microsoft.com/library/dn589795.aspx) .

