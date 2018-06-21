---
title: Azure 참조 아키텍처
description: Azure의 일반 워크로드에 대한 참조 아키텍처, 청사진 및 규범적 구현 지침입니다.
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 6c9be20e2b831f2e6c1ffd33aa89a56375a0511c
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/21/2018
ms.locfileid: "34422901"
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="azure-reference-architectures"></a>Azure 참조 아키텍처

Azure 참조 아키텍처는 시나리오별로 정리되며, 관련 아키텍처가 함께 그룹화됩니다. 각 아키텍처는 확장성, 가용성, 관리성 및 보안에 대한 고려 사항과 함께 권장 방법을 포함하고 있습니다. 또한 대부분은 배포 가능한 솔루션을 포함하고 있습니다.

<section class="series">
    <ul class="panelContent">

<!-- N-tier -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./n-tier/images/n-tier-sql-server.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>N 계층 응용 프로그램</h3>
                        <p>Azure에 Windows 또는 Linux용 N 계층 응용 프로그램을 배포합니다.</p>
                        <p>SQL Server 및 Apache Cassandra에 대한 구성이 표시됩니다. 고가용성을 위해 두 지역에 활성-수동 구성을 배포합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- Hybrid network -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./hybrid-networking/images/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>하이브리드 네트워크</h3>
                        <p>이 시리즈에서는 온-프레미스 네트워크와 Azure 사이에 네트워크 연결을 만드는 옵션을 보여 줍니다.</p>
                        <p>구성에는 개인 전용 연결을 위한 사이트 간 VPN 또는 Azure ExpressRoute가 포함됩니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Network DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./dmz/images/secure-vnet-dmz.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>네트워크 DMZ</h3>
                        <p>이 시리즈에서는 Azure 가상 네트워크와 온-프레미스 네트워크 또는 인터넷 사이에 경계를 보호하는 네트워크 DMZ를 만드는 방법을 보여 줍니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Identity management -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./identity/images/adds-extend-domain.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ID 관리</h3>
                        <p>이 시리즈에서는 온-프레미스 AD(Active Directory) 환경을 Azure 네트워크와 통합하는 옵션을 보여줍니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- App Service web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./app-service-web-app/images/scalable-web-app.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>App Service 웹 응용 프로그램</h3>
                        <p>이 시리즈에서는 Azure App Service를 사용하는 웹 응용 프로그램 모범 사례를 보여 줍니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
    <!-- Jenkins build server -->
<li style="display: flex; flex-direction: column;">
    <a href="./jenkins/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./jenkins/images/logo.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Jenkins 빌드 서버</h3>
                        <p>Azure에서 확장성 있는 엔터프라이즈급 Jenkins 서버를 배포 및 작동합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- SharePoint Server 2016 farm -->
<li style="display: flex; flex-direction: column;">
    <a href="./sharepoint/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sharepoint/images/sharepoint.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SharePoint Server 2016 팜</h3>
                        <p>Azure에서 SQL Server Always On 가용성 그룹을 사용하여 고가용성 SharePoint Server 2016 팜을 배포 및 실행합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- SAP NetWeaver and SAP HANA -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sap/images/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure의 SAP 실행</h3>
                        <p>Azure의 고가용성 환경에 SAP를 배포하고 실행합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>