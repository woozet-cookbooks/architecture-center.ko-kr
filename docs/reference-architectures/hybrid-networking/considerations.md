---
title: Choose a solution for connecting an on-premises network to Azure
description: Compares reference architectures for connecting an on-premises network to Azure.
author: telmosampaio
ms.author: pnp
ms.date: 04/06/2017
ms.topic: article
ms.service: guidance
---

# 온-프레미스 네트워크를 Azure에 연결하기 위한 솔루션 선택

이 문서는 온-프레미스 네트워크를 Azure 가상 네트워크(VNet)에 연결하기 위한 솔루션들을 비교하고 그 장점과 고려사항을 소개합니다. 각 옵션마다 각각 한 개의 참조 아키텍처와 배포가능한 솔루션을 제공합니다. 

## 가상 사설 네트워크(VPN) 연결

트래픽은 IPSec VPN 터널을 통해 온-프레미스 네트워크와 Azure 가상 네트워크(VNet) 사이를 흐릅니다.

이 아키텍처는 온-프레미스 하드웨어와 클라우드 사이의 트래픽이 적은 경우와 대기 시간이 약간 증가하는 것을 감수하고 클라우드의 유연성과 처리 능력을 높이려는 경우에 사용되는 하이브리드 애플리케이션에 적합합니다. 

**이점**

- 간단한 구성.

**고려사항**

- 온-프레미스 VPN 장치 필요.
- Microsoft가 각 VPN 게이트웨이에 대해 99.9%의 가용성을 보장하더라도 서비스 수준 계약(SLA)는 VPN 게이트웨이만 다루고 게이트웨이에 대한 네트워크 연결은 다루지 않습니다.
- Azure VPN 게이트웨이에 대한 VPN 연결은 현재 최대 200 Mbps의 대역폭을 지원합니다. 이 처리율이 초과될 것으로 예상된다면 여러 VPN 연결에 걸쳐 Azure 가상 네트워크를 분할할 필요가 있습니다.

**[자세히 보기][vpn]**

## Azure ExpressRoute 연결

ExpressRoute 연결은 외부 연결성 공급자를 통해 사설 전용 연결을 사용합니다. 사설 연결을 통해 온-프레미스 네트워크를 Azure로 확장할 수 있습니다.  

이 아키텍처는 높은 확장성을 요구하는 대규모의 중요한 워크로드를 실행하는 하이브리드 애플리케이션에 적합합니다. 

**이점**

- 연결성 공급자에 따라 최대 10 Gbps까지의 높은 대역폭 가능.
- 수요가 적을 때 비용을 절약할 수 있는 동적 대역폭 스케일링 지원. 그러나 모든 연결성 공급자가 이 옵션을 제공하지는 않습니다.
- 연결성 공급자에 따라 조직이 국가 클라우드에 직접 접속할 수도 있습니다.
- 전체 연결에 대해 99.9% 가용성 SLA

**고려사항**

- 연결 생성에는 외부 연결성 공급자의 협력이 필요합니다. 연결성 공급자는 네트워크 연결의 프로비전을 담당합니다.
- 온-프레미스에 높은 대역폭의 라우터 필요.

**[자세히 보기][expressroute]**

## VPN 장애조치를 지원하는 Expressroute

이 옵션은 앞선 두 가지 방법을 결합한 것으로, 일반적인 상황에서는 ExpressRoute를 사용하다가 ExpressRoute 회선이 끊어진 경우 VPN으로 우회하게 됩니다.

이 아키텍처는 ExpressRoute의 높은 대역폭과 고가용성 네트워크 연결성이 필요한 하이브리드 애플리케이션에 적합합니다.

**이점**

- ExpressRoute 회로 장애 시 고가용성. 그러나 대체 연결(fallback connection)은 더 낮은 대역폭 네트워크에서 이루어집니다.

**고려사항**

- 복잡한 구성. VPN 연결과 ExpressRoute 회로를 모두 설치해야 합니다.
- 중복 하드웨어(VPN 어플라이언스) 및 유료 중복 Azure VPN 게이트웨이 연결 필요.

**[추가 정보][expressroute-vpn-failover]**

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md
