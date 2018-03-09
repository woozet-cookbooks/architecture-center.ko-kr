---
title: "Azure에 온-프레미스 네트워크 연결"
description: "온-프레미스 네트워크와 Azure 간에 안전하고 강력한 네트워크 연결을 만들려는 경우에 권장하는 아키텍처입니다."
layout: LandingPage
ms.openlocfilehash: 372efb8ecf69245a5895c51e3da156a348bd665e
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/08/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="connect-an-on-premises-network-to-azure"></a>Azure에 온-프레미스 네트워크 연결

이러한 참조 아키텍처는 온-프레미스 네트워크와 Azure 간에 강력한 네트워크 연결을 만드는 방법에 대한 검증된 사례를 보여 줍니다. [어떤 것을 선택해야 하나요?](./considerations.md)

<section class="series">
    <ul class="panelContent">
    <!-- VPN -->
<li style="display: flex; flex-direction: column;">
    <a href="./vpn.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/vpn.svg" height="140px" />
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
    <!-- ExpressRoute -->
<li style="display: flex; flex-direction: column;">
    <a href="./expressroute.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ExpressRoute</h3>
                        <p>Azure ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure로 확장합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- ExpressRoute with VPN failover -->
<li style="display: flex; flex-direction: column;">
    <a href="./expressroute-vpn-failover.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/expressroute-vpn-failover.svg" height="140px" />
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
    <!-- Hub-spoke topology -->
<li style="display: flex; flex-direction: column;">
    <a href="./hub-spoke.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/hub-spoke.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>허브-스포크 토폴로지</h3>
                        <p>허브는 온-프레미스 네트워크에 대한 연결의 중심입니다. 스포크는 허브와 피어링하는 Vnet이며 워크로드를 격리하는 데 사용할 수 있습니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Hub-spoke topology with shared services -->
<li style="display: flex; flex-direction: column;">
    <a href="./shared-services.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/shared-services.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>공유 서비스를 사용하는 허브-스포크 토폴로지</h3>
                        <p>Active Directory 서비스 및 NVA(네트워크 가상 어플라이언스)를 비롯하여 공유 서비스를 포함하는 허브-스포크 토폴로지를 배포합니다. 공유 서비스는 모든 스포크에서 사용할 수 있습니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
</ul>