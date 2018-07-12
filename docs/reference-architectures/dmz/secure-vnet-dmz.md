---
title: Azure와 인터넷 간의 DMZ 구현
description: Azure에서 인터넷 액세스로 보안 하이브리드 네트워크 아키텍처를 구현하는 방법입니다.
author: telmosampaio
ms.date: 07/02/2018
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: 7a062d2394ae8b3bd1b17c19cbdf512327f9a766
ms.sourcegitcommit: 9b459f75254d97617e16eddd0d411d1f80b7fe90
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/03/2018
ms.locfileid: "37403150"
---
# <a name="dmz-between-azure-and-the-internet"></a>Azure와 인터넷 간의 DMZ

이 참조 아키텍처는 온-프레미스 네트워크를 Azure로 확장하고 인터넷 트래픽을 수락하는 보안 하이브리드 네트워크를 보여 줍니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

[![0]][0] 

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

이 참조 아키텍처는 [Azure와 온-프레미스 데이터 센터 간 DMZ 구현][implementing-a-secure-hybrid-network-architecture]에 설명된 아키텍처를 확장합니다. 이 참조 아키텍처는 온-프레미스 네트워크로부터 오는 트래픽을 처리하는 사설 DMZ 외에 인터넷 트래픽을 처리하는 공용 DMZ를 추가합니다. 

일반적으로 이 아키텍처는 다음과 같은 용도로 사용됩니다.

* 워크로드의 일부는 온-프레미스에서, 일부는 Azure에서 실행되는 하이브리드 응용 프로그램
* 온-프레미스 및 인터넷으로부터 오는 트래픽을 라우팅하는 Azure 인프라

## <a name="architecture"></a>아키텍처

이 아키텍처는 다음 구성 요소로 구성됩니다.

* **공용 IP 주소(PIP)**. 공용 끝점의 IP 주소. 인터넷에 연결된 외부 사용자는 이 주소를 통해 시스템에 액세스할 수 있습니다.
* **NVA(네트워크 가상 어플라이언스)**. 이 아키텍처는 인터넷에서 오는 트래픽을 위한 별도의 NVA 풀을 포함합니다.
* **Azure Load Balancer**. 인터넷에서 오는 모든 요청은 부하 분산 장치를 통과하고 공용 DMZ 내 NVA로 분산됩니다.
* **공용 DMZ 인바운드 서브넷**. 이 서브넷은 Azure Load Balancer로부터 오는 요청을 수용합니다. 들어오는 요청은 공용 DMZ 내 NVA 중 하나로 전달됩니다.
* **공용 DMZ 아웃바운드 서브넷**. NVA에 의해 승인된 요청은 이 서브넷을 거쳐 웹 계층을 위한 내부 부하 분산 장치로 전달됩니다.

## <a name="recommendations"></a>권장 사항

대부분의 시나리오의 경우 다음 권장 사항을 적용합니다. 이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다. 

### <a name="nva-recommendations"></a>NVA 권장 사항

인터넷에서 발생하는 트래픽에 대해 하나의 NVA 집합을 사용하고, 온-프레미스에서 발생하는 트래픽에 대해 다른 NVA 집합을 사용합니다. 둘 다에 대해 하나의 NVA 집합만 사용하면 두 네트워크 트래픽 집합 사이에 보안 경계가 없어지므로 보안 위험이 초래됩니다. 별도 NVA를 사용하면 보안 규칙을 검사하는 복잡성이 줄어들고 들어오는 네트워크 요청에 해당하는 규칙이 명확해집니다. 하나의 NVA 집합은 인터넷 트래픽에만 적용되는 규칙을 수행하고 다른 하나의 NVA 집합은 온-프레미스 트래픽에만 적용되는 규칙을 수행합니다.

NVA 수준에서 응용 프로그램 연결을 종료하고 백 엔드 계층 호환성을 유지하기 위해 레이어7 NVA를 포함시킵니다. 이를 통해 백 엔드 계층으로부터의 응답 트래픽이 NVA를 통해 돌아오는 대칭 연결이 보장됩니다.  

### <a name="public-load-balancer-recommendations"></a>공용 부하 분산 장치 권장 사항

높은 확장성과 가용성을 위해 공용 DMZ NVA를 하나의 [가용성 집합][availability-set] 내에 배포하고 [인터넷 연결 부하 분산 장치][load-balancer]를 사용하여 인터넷 요청을 가용성 집합 내 NVA에 분산시킵니다.  

부하 분산 장치가 인터넷 트래픽에 필요한 포트로만 요청을 받도록 구성합니다. 예를 들어, 인바운드 HTTP 요청은 포트 80으로 제한하고 아웃바운드 HTTPS 요청은 포트 443으로 제한합니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

처음에는 아키텍처가 공용 DMZ 내 단일 NVA를 요구하지만, 처음부터 공용 DMZ 앞에 부하 분산 장치를 배치할 것을 권장합니다. 이를 통해 향후 필요한 경우 여러 NVA로 보다 수월하게 확장할 수 있습니다.

## <a name="availability-considerations"></a>가용성 고려 사항

인터넷 연결 부하 분산 장치는 공용 DMZ 수신 서브넷 내의 각 NVA가 [상태 프로브][lb-probe]를 구현하도록 요구합니다. 이 끝점에서 응답하지 않는 상태 프로브는 사용 불가능한 것으로 간주되어 부하 분산 장치가 요청을 동일 가용성 집합 내 다른 NVA로 전달하게 됩니다. 모든 NVA가 응답하지 않는 경우 응용 프로그램이 중지되므로 정상적인 NVA 인스턴스 수가 정의된 한계 아래로 내려가면 DevOps에 알리도록 모니터링 기능을 구성하는 것이 중요합니다.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

공용 DMZ 내 NVA에 대한 모든 모니터링 및 관리는 관리 서브넷의 점프박스를 통해 수행되어야 합니다. [Azure와 온-프레미스 데이터 센터 간 DMZ 구현][implementing-a-secure-hybrid-network-architecture]에서 언급되었듯이, 온-프레미스 네트워크로부터 게이트웨이를 거쳐 점프박스로 이어지는 단일의 네트워크 루트를 정의하여 액세스를 제한합니다.

온-프레미스 네트워크에서 Azure로의 게이트웨이 연결이 중단되는 경우에도 공용 IP 주소를 배포하고 점프박스에 추가한 후 인터넷으로부터 로그인하여 점프박스에 액세스할 수 있습니다

## <a name="security-considerations"></a>보안 고려 사항

이 참조 아키텍처는 여러 수준의 보안을 구현합니다.

* 인터넷 연결 부하 분산 장치는 해당 응용 프로그램에 필요한 포트를 통해서만 수신 공용 DMZ 서브넷 내 NVA로 요청을 전달합니다.
* 인바운드 및 아웃바운드 공용 DMZ 서브넷을 위한 NSG(네트워크 보안 그룹) 규칙을 통해 NSG 규칙을 충족하지 않는 요청을 차단함으로써 NVA를 위험으로부터 보호할 수 있습니다.
* NVA를 위한 NAT 라우팅 구성에 따라 포트 80과 포트 443으로 들어오는 요청을 웹 계층의 부하 분산 장치로 전달하고 그 외 다른 모든 포트로 오는 요청은 무시합니다.

모든 포트로 오는 모든 요청에 대한 로그를 기록해야 합니다. 예상되는 매개변수 범위를 벗어나는 요청은 침입 시도를 의미할 수 있으므로 이에 관심을 기울여 정기적인 로그 감사를 실시합니다.


## <a name="deploy-the-solution"></a>솔루션 배포

이러한 권장 사항을 구현하는 참조 아키텍처 배포는 [GitHub][github-folder]를 통해 수행할 수 있습니다. 

### <a name="prerequisites"></a>필수 조건

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a>리소스 배포

1. 참조 아키텍처 GitHub 리포지토리의 `/dmz/secure-vnet-hybrid` 폴더로 이동합니다.

2. 다음 명령 실행:

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. 다음 명령 실행:

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a>온-프레미스와 Azure 게이트웨이 연결

이 단계에서는 두 개의 로컬 네트워크 게이트웨이를 연결합니다.

1. Azure Portal에서 만든 리소스 그룹으로 이동합니다. 

2. `ra-vpn-vgw-pip`라는 리소스를 찾고 **개요** 블레이드에 표시된 IP 주소를 복사합니다.

3. `onprem-vpn-lgw`라는 리소스를 찾습니다.

4. **구성** 블레이드를 클릭합니다. **IP 주소** 아래에서 2단계의 IP 주소에 붙여넣습니다.

    ![](./images/local-net-gw.png)

5. **저장**을 클릭하고 작업이 완료되기를 기다립니다. 5분 정도 걸릴 수 있습니다.

6. `onprem-vpn-gateway1-pip`라는 리소스를 찾습니다. **개요** 블레이드에 표시된 IP 주소를 복사합니다.

7. `ra-vpn-lgw`라는 리소스를 찾습니다. 

8. **구성** 블레이드를 클릭합니다. **IP 주소** 아래에서 6단계의 IP 주소에 붙여넣습니다.

9. **저장**을 클릭하고 작업이 완료되기를 기다립니다.

10. 연결을 확인하려면 각 게이트웨이에 대한 **연결** 블레이드로 이동합니다. 상태가 **연결돼** 있어야 합니다.

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a>네트워크 트래픽이 웹 계층에 도달하는지 확인

1. Azure Portal에서 만든 리소스 그룹으로 이동합니다. 

2. 공용 DMZ 앞의 부하 분산 장치인 `pub-dmz-lb`라는 리소스를 찾습니다. 

3. **개요** 블레이드에서 공용 IP 주소를 복사하고 웹 브라우저에서 이 주소를 엽니다. 기본 Apache2 서버 홈 페이지가 표시됩니다.

4. 개인 DMZ 앞의 부하 분산 장치인 `int-dmz-lb`라는 리소스를 찾습니다. **개요** 블레이드에서 개인 IP 주소를 복사합니다.

5. `jb-vm1`이라는 VM을 찾습니다. **연결**을 클릭하고 원격 데스크톱을 사용하여 VM에 연결합니다. 사용자 이름 및 암호는 onprem.json 파일에서 지정됩니다.

6. 원격 데스크톱 세션에서 웹 브라우저를 열고 4단계의 IP 주소로 이동합니다. 기본 Apache2 서버 홈 페이지가 표시됩니다.

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "하이브리드 네트워크 아키텍처 보안"
