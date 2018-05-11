---
title: N 계층 응용 프로그램 참조 아키텍처
description: Azure에 엔터프라이즈 규모 응용 프로그램을 호스트하는 VM을 배포하기 위한 몇 가지 일반적인 아키텍처를 설명합니다.
layout: LandingPage
ms.openlocfilehash: 288acc36e7c310e70240caa3ed9f2095bbb8bc58
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/10/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="n-tier-application-reference-architectures"></a>N 계층 응용 프로그램 참조 아키텍처

<section class="series">
    <ul class="panelContent">

<!-- N-tier Windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier-sql-server.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SQL Server를 통한 N 계층 응용 프로그램</h3>
                        <p>Windows에서 SQL Server를 사용하여 N 계층 응용 프로그램에 대해 구성된 가상 네트워크와 VM을 배포합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- Multi-region Windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./multi-region-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/multi-region-application.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SQL Server를 통한 다중 지역 N 계층 응용 프로그램</h3>
                        <p>SQL Server Always On 가용성 그룹을 사용하여 고가용성을 위해 N 계층 응용 프로그램을 두 지역에 배포합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- N-tier Linux -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier-cassandra.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/n-tier-cassandra.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Cassandra를 통한 N 계층 응용 프로그램</h3>
                        <p>Apache Cassandra를 사용하여 N 계층 응용 프로그램에 대해 구성된 Linux VM과 가상 네트워크를 배포합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <a href="./windows-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/Windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Windows VM</h3>
                        <p>Azure에서 Windows VM을 실행하기 위한 기준 권장 사항입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<li style="display: flex; flex-direction: column;">
    <a href="./linux-vm.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/LinuxPenguin.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Linux VM</h3>
                        <p>Azure에서 Linux VM을 실행하기 위한 기준 권장 사항입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

</ul>