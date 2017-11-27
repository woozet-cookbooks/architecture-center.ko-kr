---
title: "Azure에 온-프레미스 네트워크 연결"
description: "온-프레미스 네트워크와 Azure 간에 안전하고 강력한 네트워크 연결을 만들려는 경우에 권장하는 아키텍처입니다."
layout: LandingPage
ms.openlocfilehash: 0707d17295e338af0176bd0cea806615ef05f9ad
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure"></a>Azure에 온-프레미스 네트워크 연결

이러한 참조 아키텍처는 온-프레미스 네트워크와 Azure 간에 강력한 네트워크 연결을 만드는 방법에 대한 검증된 사례를 보여 줍니다. [어떤 것을 선택해야 하나요?](./considerations.md)

<ul class="panelContent">
    <li>
        <a href="./vpn.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/vpn.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>VPN</h3>
                            <p>사이트 간 VPN(가상 사설망)을 사용하여 온-프레미스 네트워크를 Azure로 확장합니다.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./expressroute.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/expressroute.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>ExpressRoute</h3>
                            <p>Azure ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure로 확장</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./expressroute-vpn-failover.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/expressroute-vpn-failover.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>VPN 장애 조치(failover)를 사용하는 ExpressRoute</h3>
                            <p>Azure ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure로 확장하고, VPN을 장애 조치(failover) 연결로 사용합니다.</p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
    <li>
        <a href="./hub-spoke.md">
            <div class="cardSize">
                <div class="cardPadding">
                    <div class="card">
                        <div class="cardImageOuter">
                            <div class="cardImage">
                            <img src="./images/hub-spoke.svg">
                            </div>
                        </div>
                        <div class="cardText">
                            <h3>허브-스포크 토폴로지</h3>
                            <p>허브는 온-프레미스 네트워크에 대한 연결의 중심입니다. 스포크는 허브와 피어링하는 Vnet이며 워크로드를 격리하는 데 사용할 수 있습니다. </p>
                        </div>
                    </div>
                </div>
            </div>
        </a>
    </li>
</ul>

