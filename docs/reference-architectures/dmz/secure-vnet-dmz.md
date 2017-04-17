---
title: Implementing a DMZ between Azure and the Internet
description: >-
  How to implement a secure hybrid network architecture with Internet access in
  Azure.
author: telmosampaio
ms.service: guidance
ms.topic: article
ms.date: 11/23/2016
ms.author: pnp

pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
---
# Azure와 인터넷 사이
[!INCLUDE [header](../../_includes/header.md)]

이 문서는 온-프레미스 네트워크를 Azure로 확장하고 인터넷 트래픽도 수용하는 보안된 하이브리드 네트워크를 구현하기 위한 모범 사례를 소개합니다. 

> [!참고]
> Azure는 [Resource Manager](/azure/azure-resource-manager/resource-group-overview) 와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 참조 아키텍처에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다.
> 
> 

이 참조 아키텍처는 [Azure와 온-프레미스 데이터센터 간 DMZ를 구현][implementing-a-secure-hybrid-network-architecture]에 설명된 아키텍처를 확장한 것입니다. 이 참조 아키텍처는 온-프레미스 네트워크로부터 오는 트래픽을 처리하는 사설 DMZ 외에 인터넷 트래픽을 처리하는 공용 DMZ를 추가합니다. 

일반적으로 이 아키텍처는 다음과 같은 용도로 사용됩니다.

* 워크로드의 일부는 온-프레미스에서, 일부는 Azure에서 실행되는 하이브리드 응용 프로그램.
* 온-프레미스 및 인터넷으로부터 오는 트래픽을 라우팅하는 Azure 인프라.

## 아키텍처 다이어그램

다음 다이어그램은 이 아키텍처의 중요한 요소들을 보여줍니다.

> [Microsoft 다운로드 센터][visio-download]에서 이 아키텍처 다이어그램을 포함한 Visio 문서를 다운로드할 수 있습니다. 이 다이어그램은 "DMZ - Public" 페이지에 있습니다.
> 
> 

[![0]][0] 

Azure로 인터넷 트래픽을 허용하기 위해 이 아키텍처는 다음과 같은 구성요소를 포함합니다. 

* **공용 IP 주소(PIP)**. 공용 끝점의 IP 주소. 인터넷에 연결된 외부 사용자는 이 주소를 통해 시스템에 액세스할 수 있습니다.
* **네트워크 가상 어플라이언스(NVA)**. 이 아키텍처는 인터넷에서 오는 트래픽을 위한 별도의 NVA 풀을 포함합니다.
* **Azure 부하 분산 장치**. 인터넷에서 오는 모든 요청은 부하 분산 장치를 통과하고 공용 DMZ 내 NVA로 분산됩니다.
* **공용 DMZ 인바우드 서브넷**. 이 서브넷은 Azure 부하 분산 장치로부터 오는 요청을 수용합니다. 들어오는 요청은 공용 DMZ 내 NVA 중 하나로 전달됩니다.
* **공용 DMZ 송신 서브넷**. NVA에 의해 승인된 요청은 이 서브넷을 거쳐 웹 계층을 위한 내부 부하 분산 장치로 전달됩니다.

## 권장사항

다음 권장사항은 대부분의 시나리오에 적용됩니다. 다른 구체적인 요구사항이 없다면 가급적 권장사항을 따르시기 바랍니다. 

### 네트워크 가상 어플라이언스(NVA) 권장사항

인터넷에서 오는 트래픽과 온-프레미스에서 오는 트래픽에 대해 각각 하나의 NVA 집합을 사용합니다. 두 트래픽에 대해 단일 NVA 집합을 사용하면 두 개의 네트워크 트래픽 집합 간 보안 경계가 없으므로 보안 위험이 발생합니다. 별개의 NVA를 사용하면 보안 규칙 검사의 복잡성이 감소하고 각각의 들어오는 네트워크 요청에 어떤 규칙이 적용되는지를 보다 명확히 할 수 있습니다.  하나의 NVA 집합은 인터넷 트래픽에만 적용되는 규칙을 수행하고 다른 하나의 NVA 집합은 온-프레미스 트래픽에만 적용되는 규칙을 수행합니다. 

NVA 수준에서 응용 프로그램 연결을 종료하고 백엔드 계층 호환성을 유지하기 위해 레이어7 NVA를 포함시킵니다. 이를 통해 백엔드 계층으로부터의 응답 트래픽이 NVA를 통해 돌아오는 대칭 연결이 보장됩니다.

### 공개 부하 분산 장치 권장사항

높은 확장성과 가용성을 위해 공용 DMZ NVA를 하나의 [가용성 집합][availability-set] 내에 배포하고 [인터넷 연결 부하 분산 장치][load-balancer]를 사용하여 인터넷 요청을 가용성 집합 내 NVA에 분산시킵니다.   

부하 분산 장치가 인터넷 트래픽에 필요한 포트로만 요청을 받도록 구성합니다. 예를 들어, 들어오는 HTTP 요청은 포트 80으로 제한하고 들어오는 HTTPS 요청은 포트 443으로 제한합니다.

## 확장성 고려사항

아키텍처가 공용 DMZ 내 단일 NVA를 요구하는 경우에도 처음부터 공용 DMZ 앞에 부하 분산 장치를 배치할 것을 권장합니다. 이를 통해 향후 필요한 경우 여러 NVA로 보다 수월하게 확장할 수 있습니다. 

## 가용성 고려사항

인터넷 연결 부하 분산 장치는 공용 DMZ 수신 서브넷 내의 각 NVA가 [상태 프로브][lb-probe]를 수행하도록 요구합니다. 이 끝점에서 응답하지 않는 상태 프로브는 사용 불가능한 것으로 간주되어 부하 분산 장치가 요청을 동일 가용성 집합 내 다른 NVA로 전달하게 됩니다. 모든 NVA가 응답하지 않는 경우 응용 프로그램이 중지되므로 정상적인 NVA 인스턴스 수가 정의된 한계 아래로 내려가면 모니터링 기능이 DevOps에 알리도록 구성하는 것이 중요합니다. 

## 관리 효율성 고려사항

공용 DMZ 내 NVA에 대한 모든 모니터링 및 관리는 관리 서브넷의 점프박스를 통해 수행되어야 합니다. [Azure와 온-프레미스 데이터센터 간 DMZ를 구현][implementing-a-secure-hybrid-network-architecture]에서 언급되었듯이, 온-프레미스 네트워크로부터 게이트웨이를 거쳐 점프박스로 이어지는 단일의 네트워크 루트를 정의하여 액세스를 제한합니다.

온-프레미스 네트워크에서 Azure로의 게이트웨이 연결이 중단되는 경우에도 공용 IP 주소를 배포하고 점프박스에 추가한 후 인터넷으로부터 로그인하여 점프박스에 액세스할 수 있습니다.

## 보안 고려사항

이 참조 아키텍처는 여러 수준의 보안을 구현합니다.

* 인터넷 연결 부하 분산 장치는 해당 응용 프로그램에 필요한 포트를 통해서만 수신 공용 DMZ 서브넷 내 NVA로 요청을 전달합니다.
* 송수신 공용 DMZ 서브넷을 위한 네트워크 보안 그룹(NSG) 규칙을 통해 NSG 규칙을 충족하지 않는 요청을 차단함으로써 NVA를 위험으로부터 보호할 수 있습니다.
* NVA를 위한 NAT 라우팅 구성에 따라 포트 80과 포트 443으로 들어오는 요청을 웹 계층의 부하 분산 장치로 전달하고 그 외 다른 모든 포트로 오는 요청은 무시합니다.

모든 포트로 오는 모든 요청에 대한 로그를 기록해야 합니다. 예상되는 매개변수 범위를 벗어나는 요청은 침입 시도를 의미할 수 있으므로 이에 관심을 기울여 정기적인 로그 감사를 실시합니다.

## 솔루션 배포

이러한 권장사항을 구현하는 참조 아키텍처 배포는 [GitHub][github-folder]를 통해 수행할 수 있습니다. 이 참조 아키텍처는 다음 지침에 따라 Windows 또는 Linux VM을 사용하여 배포할 수 있습니다. 

1. 아래 버튼을 마우스 오른쪽 단추로 클릭하여 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열리면 일부 설정값을 입력합니다.

   * **리소스 그룹** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-public-dmz-network-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자는 편집하지 않습니다.
   
   * **OS 유형** 드롭다운 상자에서 **Windows** 또는 **Linux**를 선택합니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 버튼을 클릭합니다.
3. 배포가 완료될 때까지 기다립니다.
4. 아래 버튼을 마우스 오른쪽 단추로 클릭하여 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Azure 포털에서 링크가 열리면 일부 설정값을 입력합니다.

   * **리소스 그룹** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-public-dmz-wl-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자는 편집하지 않습니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 버튼을 클릭합니다.
6. 배포가 완료될 때까지 기다립니다.
7. 아래 버튼을 마우스 오른쪽 단추로 클릭하여 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. Azure 포털에서 링크가 열리면 일부 설정값을 입력합니다.
   * **리소스 그룹** 이름이 매개변수 파일에 이미 정의되어 있으므로 **기존 사용**을 선택한 다음 텍스트 상자에 `ra-public-dmz-network-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자는 편집하지 않습니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 버튼을 클릭합니다.
9. 배포가 완료될 때까지 기다립니다.
10. 매개변수 파일에는 모든 VM에 대한 하드 코딩된 관리자 사용자 이름 및 암호가 포함되어 있는데, 이 둘 모두를 즉시 변경할 것을 권장합니다. Azure 포털에서 배포의 각 VM을 선택한 후 **지원 + 문제해결** 블레이드에서 **암호 재설정**을 클릭합니다. **모드** 드롭다운 상자에서 **암호 재설정**을 선택한 후 새 **사용자 이름** 및 **암호**를 선택합니다. **업데이트** 버튼을 클릭하여 저장합니다.

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-multi-tier-architecture-on-Azure]: ./guidance-compute-3-tier-vm.md
[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx

[0]: ../_images/blueprints/hybrid-network-secure-vnet-dmz.png "Secure hybrid network architecture"
