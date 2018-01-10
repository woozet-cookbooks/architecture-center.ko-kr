---
title: "Azure와 인터넷 간의 DMZ 구현"
description: "Azure에서 인터넷 액세스로 보안 하이브리드 네트워크 아키텍처를 구현하는 방법입니다."
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: 372d5bb0fc0e3c272843e062210dec5c15b2b78a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="dmz-between-azure-and-the-internet"></a>Azure와 인터넷 간의 DMZ

이 참조 아키텍처는 온-프레미스 네트워크를 Azure로 확장하고 인터넷 트래픽을 수락하는 보안 하이브리드 네트워크를 보여 줍니다. 

[![0]][0] 

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

이 참조 아키텍처는 [Azure와 온-프레미스 데이터 센터 간 DMZ 구현][implementing-a-secure-hybrid-network-architecture]에 설명된 아키텍처를 확장합니다. 이 참조 아키텍처는 온-프레미스 네트워크로부터 오는 트래픽을 처리하는 사설 DMZ 외에 인터넷 트래픽을 처리하는 공용 DMZ를 추가합니다. 

일반적으로 이 아키텍처는 다음과 같은 용도로 사용됩니다.

* 워크로드의 일부는 온-프레미스에서, 일부는 Azure에서 실행되는 하이브리드 응용 프로그램
* 온-프레미스 및 인터넷으로부터 오는 트래픽을 라우팅하는 Azure 인프라

## <a name="architecture"></a>건축

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

## <a name="solution-deployment"></a>솔루션 배포

이러한 권장 사항을 구현하는 참조 아키텍처 배포는 [GitHub][github-folder]를 통해 수행할 수 있습니다. 이 참조 아키텍처는 다음 지침에 따라 Windows 또는 Linux VM을 사용하여 배포할 수 있습니다.

1. 아래 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열렸으면 일부 설정에 대한 값을 입력해야 합니다.
   * **리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-public-dmz-network-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 하위 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 드롭다운 상자에서 **OS 유형**을 선택으로 **Windows** 또는 **Linux**를 선택합니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 단추를 클릭합니다.
3. 배포가 완료될 때가지 기다립니다.
4. 아래 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Azure 포털에서 링크가 열렸으면 일부 설정에 대한 값을 입력해야 합니다.
   * **리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-public-dmz-wl-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 하위 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 단추를 클릭합니다.
6. 배포가 완료될 때가지 기다립니다.
7. 아래 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. Azure 포털에서 링크가 열렸으면 일부 설정에 대한 값을 입력해야 합니다.
   * **리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **기존 사용**을 선택하고 텍스트 상자에 `ra-public-dmz-network-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 하위 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 단추를 클릭합니다.
9. 배포가 완료될 때가지 기다립니다.
10. 매개 변수 파일에는 모든 VM에 대한 하드 코딩된 관리자 사용자 이름 및 암호가 포함되어 있는데, 이 둘 모두를 즉시 변경할 것을 권장합니다. 배포의 각 VM을 Azure Portal에서 선택한 후 **지원 + 문제 해결** 블레이드에서 **암호 재설정**을 클릭합니다. **모드** 드롭다운 상자에서 **암호 재설정**을 선택한 후 새 **사용자 이름** 및 **암호**를 선택합니다. **업데이트** 단추를 클릭하여 저장합니다.


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.azureedge.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "하이브리드 네트워크 아키텍처 보안"