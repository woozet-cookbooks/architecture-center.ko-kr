---
title: 개발/테스트 작업에 대한 SAP
description: 개발/테스트 환경에 대한 SAP 시나리오입니다.
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: 675a5cb4b1ee4001ca50d24c145ce1a177f90da4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060964"
---
# <a name="sap-for-devtest-workloads"></a>개발/테스트 작업에 대한 SAP

이 예제에서는 Azure의 Windows 또는 Linux 환경에서 SAP NetWeaver의 개발/테스트 구현을 실행하는 방법에 대한 지침을 제공합니다. 사용되는 데이터베이스는 지원되는 모든 DBMS(SAP HANA가 아님)를 의미하는 SAP 용어인 AnyDB입니다. 이 아키텍처는 비프로덕션 환경에 대해 설계되었기 때문에 단일 VM(가상 머신)으로 배포되며 크기는 조직의 요구에 맞게 변경할 수 있습니다.

프로덕션 사용 사례의 경우 아래에서 사용할 수 있는 SAP 참조 아키텍처를 검토합니다.

* [AnyDB용 SAP NetWeaver][sap-netweaver]
* [SAP S/4Hana][sap-hana]
* [Azure의 SAP(대규모 인스턴스)][sap-large]

## <a name="related-use-cases"></a>관련 사용 사례

이 시나리오에 적합한 사용 사례는 다음과 같습니다.

* 중요하지 않은 비생산적 SAP 워크로드(샌드박스, 개발, 테스트, 품질 보증)
* 중요하지 않은 단일 SAP 비즈니스 워크로드

## <a name="architecture"></a>아키텍처

![다이어그램](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

이 시나리오에서는 단일 가상 머신에 단일 SAP 시스템 데이터베이스와 SAP 응용 프로그램 서버를 프로비전하는 방법에 대해 설명하며, 시나리오를 통한 데이터 흐름은 다음과 같습니다.

1. 프레젠테이션 계층의 고객은 온-프레미스에서 자신의 SAP GUI 또는 다른 사용자 인터페이스(Internet Explorer, Excel 또는 다른 웹 응용 프로그램)를 사용하여 Azure 기반 SAP 시스템에 액세스합니다.
2. 연결은 설정된 ExpressRoute를 사용하여 제공됩니다. ExpressRoute는 Azure의 ExpressRoute 게이트웨이에서 종료됩니다. 네트워크 트래픽은 ExpressRoute 게이트웨이를 통해 게이트웨이 서브넷으로 라우팅되고, 게이트웨이 서브넷에서 응용 프로그램 계층 스포크 서브넷([허브-스포크][hub-spoke] 패턴 참조)으로 라우팅되며, 네트워크 보안 게이트웨이를 통해 SAP 응용 프로그램 가상 머신으로 라우팅됩니다.
3. ID 관리 서버는 인증 서비스를 제공합니다.
4. 점프 박스는 로컬 관리 기능을 제공합니다.

### <a name="components"></a>구성 요소

* [리소스 그룹](/azure/azure-resource-manager/resource-group-overview#resource-groups)은 Azure 리소스에 대한 논리 컨테이너입니다.
* [가상 네트워크](/azure/virtual-network/virtual-networks-overview)는 Azure 내에서 네트워크 통신의 기초입니다.
* [가상 머신](/azure/virtual-machines/windows/overview) Azure Virtual Machines는 Windows 또는 Linux 서버를 사용하여 안전하고 가상화된 주문형 대규모 인프라를 제공합니다.
* [ExpressRoute](/azure/expressroute/expressroute-introduction)를 사용하면 연결 공급자가 지원하는 개인 연결을 통해 온-프레미스 네트워크를 Microsoft 클라우드로 확장할 수 있습니다.
* [네트워크 보안 그룹](/azure/virtual-network/security-overview)을 사용하면 네트워크 트래픽을 가상 네트워크의 리소스로 제한할 수 있습니다. 네트워크 보안 그룹에는 원본 또는 대상 IP 주소, 포트 및 프로토콜에 따라 인바운드 또는 아웃바운드 네트워크 트래픽을 허용하거나 거부하는 보안 규칙 목록이 포함되어 있습니다. 

## <a name="considerations"></a>고려 사항

### <a name="availability"></a>가용성

 Microsoft는 단일 VM 인스턴스에 대한 SLA(서비스 수준 계약)를 제공합니다. Virtual Machines와 관련된 Microsoft Azure 서비스 수준 계약에 대한 자세한 내용은 [Virtual Machines에 대한 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines)를 참조하세요.

### <a name="scalability"></a>확장성

확장 가능한 솔루션 설계에 대한 일반적인 지침은 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.

### <a name="security"></a>보안

보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.

### <a name="resiliency"></a>복원력

복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 응용 프로그램 디자인][resiliency]을 참조하세요.

## <a name="pricing"></a>가격

이 시나리오를 실행하는 데 드는 비용을 알아보려면 비용 계산기에서 모든 서비스를 미리 구성합니다.  특정 사용 사례에 대한 가격 변동을 확인하려면 필요한 트래픽에 맞게 변수를 적절하게 변경합니다.

가져오는 데 필요한 트래픽 양을 기준으로 제공한 네 가지 샘플 비용 프로필은 다음과 같습니다.

|크기|SAP|VM 유형|Storage|Azure 요금 계산기|
|----|----|-------|-------|---------------|
|작음|8000|D8s_v3|2xP20, 1xP10|[소형](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|중간|16000|D16s_v3|3xP20, 1xP10|[중형](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
큰|32000|E32s_v3|3xP20, 1xP10|[대형](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
초대형|64000|M64s|4xP20, 1xP10|[초대형](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

참고: 가격 책정은 지표이며, VM 및 저장소 비용을 나타냅니다(네트워킹, 백업 저장소 및 데이터 수신/송신 비용은 제외).

* [소형](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): 소형 시스템은 8개 vCPU, 32GB RAM 및 200GB 임시 저장소가 있는 D8s_v3 VM 유형으로 구성되며, 2개 512GB 및 1개 128GB 프리미엄 저장소 디스크도 추가로 있습니다.
* [중형](https://azure.com/e/465bd07047d148baab032b2f461550cd): 중형 시스템은 16개 vCPU, 64GB RAM 및 400GB 임시 저장소가 있는 D16s_v3 VM 유형으로 구성되며, 3개 512GB 및 1개 128GB 프리미엄 저장소 디스크도 추가로 있습니다.
* [대형](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): 대형 시스템은 32개 vCPU, 256GB RAM 및 512GB 임시 저장소가 있는 E32s_v3 VM 유형으로 구성되며, 3개 512GB 및 1개 128GB 프리미엄 저장소 디스크도 추가로 있습니다.
* [초대형](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): 초대형 시스템은 64개 vCPU, 1,024GB RAM 및 2,000GB 임시 저장소가 있는 M64s VM 유형으로 구성되며, 4개 512GB 및 1개 128GB 프리미엄 저장소 디스크도 추가로 있습니다.

## <a name="deployment"></a>배포

위 시나리오와 비슷한 기본 인프라를 배포하려면 배포 단추를 사용해 주세요.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

\* SAP가 설치되지 않으므로 인프라를 수동으로 구축한 후에 이 작업을 수행해야 합니다.

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke