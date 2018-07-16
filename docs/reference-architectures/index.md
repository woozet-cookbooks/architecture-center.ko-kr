---
title: Azure 참조 아키텍처
description: Azure의 일반 워크로드에 대한 참조 아키텍처, 청사진 및 규범적 구현 지침입니다.
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 374ca51d70e4999fbb1bacf47547040db6f0071f
ms.sourcegitcommit: 776b8c1efc662d42273a33de3b82ec69e3cd80c5
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/12/2018
ms.locfileid: "38987627"
---
# <a name="azure-reference-architectures"></a>Azure 참조 아키텍처

Azure 참조 아키텍처는 시나리오별로 정리되며, 관련 아키텍처가 함께 그룹화됩니다. 각 아키텍처는 확장성, 가용성, 관리성 및 보안에 대한 고려 사항과 함께 권장 방법을 포함하고 있습니다. 또한 대부분은 배포 가능한 솔루션을 포함하고 있습니다.

[빅 데이터](#big-data-solutions) | [웹 응용 프로그램](#web-applications) | [N 계층 응용 프로그램](#n-tier-applications) | [가상 네트워크](#virtual-networks) | [Active Directory](#extending-on-premises-active-directory-to-azure) | [VM 워크로드](#vm-workloads)로 이동합니다.

## <a name="big-data-solutions"></a>빅 데이터 솔루션

<ul  class="panelContent cardsF">
<!-- SQL Data Warehouse -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/enterprise-bi-sqldw.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./data/images/data-guide.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SQL Data Warehouse를 사용하는 Enterprise BI</h3>
                        <p>온-프레미스 데이터베이스에서 SQL Data Warehouse로 데이터를 이동하는 ELT(추출-부하-변형) 파이프라인입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./data/enterprise-bi-adf.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./data/images/data-guide.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure Data Factory를 사용하는 자동화된 Enterprise BI</h3>
                        <p>ELT 파이프라인을 자동화하여 온-프레미스 데이터베이스에서 증분 로드를 수행합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="web-applications"></a>웹 응용 프로그램

<ul  class="panelContent cardsF">
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/basic-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>기본 웹앱 응용 프로그램</h3>
                        <p>Azure App Service 및 Azure SQL Database를 사용하는 웹 응용 프로그램입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/scalable-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>확장성이 뛰어난 웹 응용 프로그램</h3>
                        <p>웹 응용 프로그램의 확장성을 향상시키기 위한 검증된 사례입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/scalable-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>고가용성 웹 응용 프로그램</h3>
                        <p>여러 지역에서 App Service 웹앱을 실행하여 고가용성을 달성합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="n-tier-applications"></a>N 계층 응용 프로그램

<ul  class="panelContent cardsF">
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/n-tier-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./n-tier/images/Windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SQL Server를 통한 N 계층 응용 프로그램</h3>
                        <p>Windows에서 SQL Server를 사용하여 N 계층 응용 프로그램에 대해 구성된 가상 머신입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- Multi-region Windows -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/multi-region-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./n-tier/images/Windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>다중 지역 N 계층 응용 프로그램</h3>
                        <p>고가용성을 위해 SQL Server AlwaysOn 가용성 그룹을 사용하여 두 지역에 배포된 N 계층 응용 프로그램입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- N-tier Linux -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/n-tier-cassandra.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./n-tier/images/LinuxPenguin.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Cassandra를 통한 N 계층 응용 프로그램</h3>
                        <p>Linux에서 Apache Cassandra를 사용하여 N 계층 응용 프로그램에 대해 구성된 가상 머신입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="virtual-networks"></a>가상 네트워크

<ul  class="panelContent cardsF">
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/vpn.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>VPN(가상 사설망)을 사용하는 하이브리드 네트워크</h3>
                        <p>Azure 가상 네트워크에 온-프레미스 네트워크 연결</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- ExpressRoute -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/expressroute.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>ExpressRoute를 사용하는 하이브리드 네트워크</h3>
                        <p>개인 전용 연결을 사용하여 온-프레미스 네트워크를 Azure로 확장합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- ExpressRoute + VPN -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/expressroute-vpn-failover.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>VPN 장애 조치(failover) 및 ExpressRoute를 사용하는 하이브리드 네트워크</h3>
                        <p>고가용성을 위한 장애 조치 연결로 VPN을 사용하는 ExpressRoute를 사용합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hub spoke -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/hub-spoke.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/gateway.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>허프 스포크 네트워크 토폴로지</h3>
                        <p>워크로드를 격리하는 동안 온-프레미스 네트워크에 대한 연결의 중심을 만듭니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Shared services -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/hub-spoke.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/gateway.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>공유 서비스를 사용하는 허브-스포크 토폴로지</h3>
                        <p>Active Directory와 같은 공유 서비스를 포함하여 허브-스포크 토폴로지를 확장합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hybrid DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/secure-vnet-hybrid.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure와 온-프레미스 간의 DMZ</h3>
                        <p>네트워크 가상 어플라이언스를 사용하여 보안 하이브리드 네트워크를 만듭니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Internet DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/secure-vnet-dmz.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure와 인터넷 간의 DMZ</h3>
                        <p>네트워크 가상 어플라이언스를 사용하여 인터넷 트래픽을 허용하는 보안 네트워크를 만듭니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="extending-on-premises-active-directory-to-azure"></a>Azure로 온-프레미스 Active Directory 확장

<ul class="panelContent cardsF">
<!-- Azure AD -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/azure-ad.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/azure-active-directory.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure Active Directory와 통합</h3>
                        <p>Azure Active Directory와 온-프레미스 AD 도메인을 통합합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- AD DS -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adds-extend-domain.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>온-프레미스 Active Directory 도메인을 Azure로 확장</h3>
                        <p>Azure에서 AD DS(Active Directory Domain Services)를 배포하여 온-프레미스 도메인을 확장합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- AD DS Forest -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adds-forest.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure에 AD DS 포리스트 만들기</h3>
                        <p>온-프레미스 AD 포리스트에서 신뢰하는 별도의 AD 도메인을 Azure에 만듭니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- AD FS -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adfs.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./_images/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure로 AD FS(Active Directory Federation Services) 확장</h3>
                        <p>Azure에서 실행 중인 구성 요소의 페더레이션된 인증 및 권한 부여에 AD FS를 사용합니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="vm-workloads"></a>VM 워크로드

<ul  class="panelContent cardsF">
<!-- Jenkins -->
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
                        <p>Azure의 확장성 있는 엔터프라이즈급 Jenkins 서버입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- SharePoint -->
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
                        <p>Azure에서 SQL Server Always On 가용성 그룹을 사용하는 고가용성 SharePoint Server 2016 팜입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- SAP -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/sap-netweaver.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sap/images/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP NetWeaver</h3>
                        <p>재해 복구를 지원하는 고가용성 환경에 설치된 Windows의 SAP NetWeaver입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./sap/sap-s4hana.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sap/images/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP S/4HANA</h3>
                        <p>재해 복구를 지원하는 고가용성 환경에 설치된 Linux의 SAP S/4HANA입니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./sap/hana-large-instances.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sap/images/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Azure의 SAP HANA(대규모 인스턴스)</h3>
                        <p>HANA 대규모 인스턴스는 Azure 지역의 실제 서버에 배포됩니다.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

