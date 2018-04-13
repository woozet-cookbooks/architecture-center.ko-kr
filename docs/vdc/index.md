---
title: Azure Virtual Datacenter
description: Microsoft Azure Virtual Datacenter의 리소스
keywords: Azure
layout: LandingPage
ms.openlocfilehash: 4aa858a78dab44f34d7155a40554cf0a136aa360
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/03/2018
---
# <a name="azure-virtual-datacenter-and-the-enterprise-control-plane"></a>Azure Virtual Datacenter 및 Enterprise Control Plane

Azure Virtual Datacenter는 기존 보안 및 네트워킹 정책을 적용하는 동시에 Azure 클라우드 플랫폼의 기능을 최대한 활용하는 방법입니다. 엔터프라이즈 워크로드를 클라우드에 배포할 때 IT 조직과 비즈니스 단위는 규제와 개발자 유연성 간에 적절한 균형을 이루어야 합니다. Azure Virtual Datacenter는 규제를 강조하면서 이러한 균형을 이루는 모델을 제공합니다.
 
## <a name="resources"></a>리소스
<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="http://aka.ms/VDC/Concepts"><img src="../_images/virtual-datacenter.svg" alt="Virtual Datacenter eBook" /></a></td>
    <td>
        <h3><a href="http://aka.ms/VDC/Concepts">Azure Virtual Datacenter: 개념</a></h3>
        <p>이 전자책은 기존 보안 및 네트워킹 정책을 적용하는 동시에 Azure 클라우드 플랫폼에 엔터프라이즈 워크로드를 배포하는 방법을 보여줍니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="/azure/networking/networking-virtual-datacenter"><img src="./images/vdc-network.png" alt="Network Perspective" /></a></td>
    <td>
        <h3><a href="https://docs.microsoft.com/en-us/azure/networking/networking-virtual-datacenter">Azure Virtual Datacenter: 네트워크 측면</a></h3>
        <p>이 온라인 문서는 많은 고객들이 클라우드로의 집단 전환을 고려할 때 직면하게 되는 구조적 규모, 성능 및 보안 문제를 해결하는 데 사용할 수 있는 네트워킹 패턴 및 디자인에 대한 개요를 제공합니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="http://aka.ms/VDC/Lift"><img src="./images/vdc-lift-and-shift.png" alt="Lift and Shift Guide" /></a></td>
    <td>
        <h3><a href="http://aka.ms/VDC/Lift">Azure Virtual Datacenter: 리프트 앤 시프트 가이드</a></h3>
        <p>이 백서는 엔터프라이즈 IT 직원 및 의사 결정권자가 클라우드 호스팅 옵션을 최적화하면서 동시에 추가적인 개발 비용을 최소화할 수 있도록 리프트 앤 시프트 방식을 사용하여 Azure로의 응용 프로그램 및 서버 마이그레이션을 식별하고 계획하는 데 사용할 수 있는 프로세스에 대해 논의합니다.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="http://aka.ms/VDC/Deck"><img src="./images/vdc-deck.png" alt="Presentation Deck" /></a></td>
    <td>
        <h3><a href="http://aka.ms/VDC/Deck">Azure Virtual Datacenter: 프레젠테이션 데크</a></h3>
        <p>이 프레젠테이션 데크는 Azure Virtual Datacenter 지침 및 도구를 탐색합니다. VDC 목표, 고객 드라이버, Azure 지역, VDC 자동화의 요소, 산업화되고 신뢰할 수 있는 Azure VDC에 대해 다루며 CIO 지침 관련 작업 계획으로 끝납니다. 지원 및 교육 정보가 제공됩니다.</p>
    </td>
</tr>
</table>

## <a name="what-is-the-azure-virtual-datacenter"></a>Azure Virtual Datacenter란?

클라우드로의 워크로드 배포에 기존 데이터 센터와 동일한 신뢰 수준으로 클라우드에 대한 신뢰를 개발하고 유지 관리해야 하는 필요성이 포함됩니다. Azure Virtual Datacenter 지침의 첫 번째 모델은 잠긴 접근 방식을 통해서 그러한 필요성을 가상 인프라에 연결하기 위해 고안되었습니다. 이 접근 방식은 모든 사용자를 위한 것은 아닙니다. 특별히 Azure 공용 클라우드로 온-프레미스 인프라를 확장하는 엔터프라이즈 IT 그룹을 안내하기 위해 고안되었습니다. 이 접근 방식을 신뢰할 수 있는 데이터 센터 확장 모델이라고 부릅니다. 시간이 지남에 따라 가상 데이터 센터에서 직접 보안 인터넷 액세스를 허용하는 등의 다른 여러 모델이 제공될 예정입니다.

<img src="./images/vdc-components.svg" alt="Virtual Datacenter components" style="max-width:700px;"/>

이러한 네 가지 구성 요소를 통해 Azure Virtual Datacenter는 ID, 암호화, 소프트웨어 정의 네트워킹 및 규정 준수(로그 및 보고 포함)를 수행할 수 있습니다.

Azure Virtual Datacenter 모델에서 격리 정책을 적용하고, 사용자가 아는 물리적 데이터 센터와 더 비슷하게 클라우드를 만들고, 사용자에게 필요한 보안 및 신뢰 수준을 달성할 수 있습니다. 엔터프라이즈 IT 팀이 인정하는 소프트웨어 정의 네트워킹, 암호화, ID 관리 및 Azure 플랫폼의 기본 준수 표준 및 인증의 네 가지 구성 요소를 통해 가능합니다. 이러한 네 가지 구성 요소는 가상 데이터 센터를 기존 인프라 투자의 신뢰할 수 있는 확장으로 만드는 데 핵심입니다.


<a href="http://aka.ms/VDC/eBook">Azure Virtual Datacenter 개념</a> 전자책을 이어서 읽어보세요.
