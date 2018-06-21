---
title: 온-프레미스 네트워크를 Azure에 연결하기 위한 솔루션 선택
description: 온-프레미스 네트워크를 Azure에 연결하기 위한 참조 아키텍처를 비교합니다.
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
ms.locfileid: "26582728"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>온-프레미스 네트워크를 Azure에 연결하기 위한 솔루션 선택

이 문서에서는 온-프레미스 네트워크를 Azure VNet(Virtual Network)에 연결하는 옵션을 비교합니다. 각 옵션에 대해 참조 아키텍처와 배포 가능한 솔루션을 제공합니다.

## <a name="vpn-connection"></a>VPN 연결

IPSec VPN 터널을 통해 Azure VNet과 온-프레미스 네트워크를 연결하기 위해 VPN(가상 사설망)을 사용합니다.

이 아키텍처는 온-프레미스 하드웨어와 클라우드 간의 트래픽이 가벼울 가능성이 높은 하이브리드 응용 프로그램에 적합하거나, 클라우드의 유연성 및 처리 능력을 위해 대기 시간을 약간 연장하고자 하는 경우에 적합합니다.

**이점**

- 구성이 간단합니다.

**과제**

- 온-프레미스 VPN 장치가 필요합니다.
- Microsoft에서 각 VPN Gateway에 99.9%의 가용성을 보장하지만, 이 SLA는 게이트웨이에 대한 네트워크 연결이 아닌 VPN Gateway만 다룹니다.
- 현재 Azure VPN Gateway를 통한 VPN 연결은 최대 200Mbps의 대역폭을 지원합니다. 이러한 처리량을 초과할 것으로 예상되는 경우 여러 VPN 연결에 걸쳐 Azure 가상 네트워크를 분할해야 할 수 있습니다.

**[자세히 알아보기...][vpn]**

## <a name="azure-expressroute-connection"></a>Azure ExpressRoute 연결

ExpressRoute 연결은 타사 연결 공급자를 통해 개인 전용 연결을 사용합니다. 개인 연결에서 온-프레미스 네트워크를 Azure로 확장할 수 있습니다. 

이 아키텍처는 높은 수준의 확장성이 필요한 대규모 중요 업무 워크로드를 실행하는 하이브리드 응용 프로그램에 적합합니다. 

**이점**

- 연결 공급자에 따라 훨씬 더 높은 대역폭(최대 10Gbps)을 사용할 수 있습니다.
- 동적 대역폭 확장을 지원하여 수요가 더 낮은 기간 동안 비용을 줄일 수 있습니다. 그러나 모든 연결 공급자에 이 옵션이 있는 것은 아닙니다.
- 연결 공급자에 따라 조직에서 국가 클라우드에 직접 액세스할 수 있게 할 수 있습니다.
- 전체 연결에서 99.9%의 가용성 SLA를 실현할 수 있습니다.

**과제**

- 설정이 복잡할 수 있습니다. ExpressRoute 연결을 만들려면 타사 연결 공급자를 함께 사용해야 합니다. 해당 공급자가 네트워크 연결 프로비저닝을 담당합니다.
- 온-프레미스에 높은 대역폭의 라우터가 필요합니다.

**[자세히 알아보기...][expressroute]**

## <a name="expressroute-with-vpn-failover"></a>VPN 장애 조치(failover)를 사용하는 ExpressRoute

이 옵션은 앞에 나온 두 가지 옵션을 결합합니다. 정상적인 조건에서는 ExpressRoute를 사용하고 ExpressRoute 회로에서 연결이 끊기면 VPN 연결로 장애 조치(failover)합니다.

이 아키텍처는 더 높은 ExpressRoute 대역폭도 필요하고 고가용성 네트워크 연결도 필요한 하이브리드 응용 프로그램에 적합합니다. 

**이점**

- ExpressRoute 회로가 고장나는 경우 대체 연결이 더 낮은 대역폭 네트워크에 있더라도 고가용성입니다.

**과제**

- 구성이 복잡합니다. VPN 연결과 ExpressRoute 회로를 모두 설정해야 합니다.
- 중복 하드웨어(VPN 어플라이언스)와 중복 Azure VPN Gateway 연결이 필요하며 이는 유료입니다.

**[자세히 알아보기...][expressroute-vpn-failover]**

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md