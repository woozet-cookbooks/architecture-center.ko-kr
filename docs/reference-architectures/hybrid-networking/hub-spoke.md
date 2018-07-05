---
title: Azure에서 허브-스포크 네트워크 토폴로지 구현
description: Azure에서 허브-스포크 네트워크 토폴로지를 구현하는 방법입니다.
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 925e0f47cf6b9aa1ad48ffae2c9561a2393bf601
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142253"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a>Azure에서 허브-스포크 네트워크 토폴로지 구현

이 참조 아키텍처는 Azure에서 허브-스포크 토폴로지를 구현하는 방법을 설명합니다. *허브*는 Azure의 가상 네트워크(VNet)로서 사용자의 온-프레미스 네트워크에 대한 연결의 중심으로 기능합니다. *스포크*는 허브와 피어링하는 VNet으로서 워크로드를 격리하는 데 사용할 수 있습니다. 트래픽은 ExpressRoute 또는 VPN 게이트웨이 연결을 통해 온-프레미스 데이터 센터와 허브 사이를 흐릅니다.  [**이 솔루션을 배포합니다**](#deploy-the-solution).

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download] 다운로드*


이 토폴로지의 이점은 다음과 같습니다.

* **비용 절감** 네트워크 가상 어플라이언스(NVA), DNS 서버와 같이 여러 워크로드에서 공유할 수 있는 서비스를 하나의 위치로 일원화하여 비용을 절감합니다.
* **구독 제한 극복**: 여러 구독의 VNet을 중앙 허브로 피어링하여 구독의 제한을 극복합니다.
* **문제 구분**: 중앙 IT(SecOps, InfraOps)와 워크로드(DevOps)를 문제를 분리합니다.

이 아키텍처의 일반적인 용도는 다음과 같습니다.

* 개발, 테스트, 생산과 같이 서로 다른 환경에 배포되고 DNS, IDS, NTP 또는 AD DS와 같은 공유 서비스가 필요한 워크로드. 공유 서비스는 허브 VNet에 배치되고 각 환경은 스포크에 배포되어 격리 상태 유지.
* 서로 연결할 필요가 없으나 공유 서비스에 대한 액세스가 필요한 워크로드.
* DMZ로 기능한 허브의 방화벽, 각 스포크에서 워크로드에 대한 별도의 관리 등 보안 측면에 대한 중앙 제어가 필요한 엔터프라이즈.

## <a name="architecture"></a>아키텍처

이 아키텍처는 다음 구성 요소로 구성됩니다.

* **온-프레미스 네트워크**. 조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.

* **VPN 장치**. 온-프레미스 네트워크에 외부 연결을 제공하는 장치 또는 서비스입니다. VPN 장치는 하드웨어 장치일 수도 있고 Windows Server 2012의 RRAS(라우팅 및 원격 액세스 서비스)와 같은 소프트웨어 솔루션일 수도 있습니다. 지원되는 VPN 어플라이언스 목록 및 선택한 VPN 어플라이언스를 Azure에 연결하도록 구성하는 방법에 대한 자세한 내용은 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치 정보][vpn-appliance]를 참조하세요.

* **VPN 가상 네트워크 게이트웨이 또는 ExpressRoute 게이트웨이**. 가상 네트워크 게이트웨이를 사용하면 VNet을 온-프레미스 네트워크에 연결하는 데 사용되는 VPN 장치 또는 ExpressRoute 회로에 연결할 수 있습니다. 자세한 내용은 [온-프레미스 네트워크를 Microsoft Azure virtual network에 연결][connect-to-an-Azure-vnet]을 참조하세요.

> [!NOTE]
> 이 참조 아키텍처의 배포 스크립트는 연결을 위해 VPN 게이트웨이를 사용하고, 사용자의 온-프레미스 네트워크를 시뮬레이션하기 위해 Azure의 VNet을 사용합니다.

* **허브 VNet**. 허브-스포크 토폴로지에서 허브로 사용되는 Azure VNet입니다. 허브는 사용자의 온-프레미스 네트워크에 대한 연결의 중앙 위치이자 스포크 VNet에 호스팅된 다양한 워크로드에 의해 사용되는 서비스가 호스팅되는 장소입니다.

* **게이트웨이 서브넷**. 가상 네트워크 게이트웨이는 동일한 서브넷에 있습니다.

* **스포크 VNet**. 허브-스포크 토폴로지에서 스포크로 사용되는 하나 이상의 Azure VNet입니다. 스포크는 다른 스포크와 별도로 관리되는 자체 VNet에서 워크로드를 격리하는 데 사용할 수 있습니다. 각 워크로드에는 Azure Load Balancer를 통해 여러 서브넷이 연결된 여러 계층이 포함될 수 있습니다. 응용 프로그램 인프라에 대한 자세한 내용은 [Windows VM 워크로드 실행][windows-vm-ra] 및 [Linux VM 워크로드 실행][linux-vm-ra]을 참조하세요.

* **VNet 피어링**. 동일한 Azure 지역에 있는 2개의 VNet을 [피어링 연결][vnet-peering]을 사용하여 연결할 수 있습니다. 피어링 연결은 VNet 사이에 적용되는 비전이적이고 대기 시간이 낮은 연결입니다. 피어링이 적용되면 VNet은 라우터가 없어도 Azure 백본을 사용하여 트래픽을 교환합니다. 허브-스포크 네트워크 토폴로지에서는 VNet 피어링을 사용하여 허브를 각 스포크에 연결합니다.

> [!NOTE]
> 이 문서에서는 [리소스 관리자](/azure/azure-resource-manager/resource-group-overview) 배포만 다루고 있지만, 동일한 구독에서 클래식 VNet을 리소스 관리자에 연결할 수도 있습니다. 이렇게 하면 스포크가 클래식 배포를 호스팅하면서도 허브에서 공유되는 서비스를 이용할 수 있습니다.

## <a name="recommendations"></a>권장 사항

대부분의 시나리오의 경우 다음 권장 사항을 적용합니다. 이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.

### <a name="resource-groups"></a>리소스 그룹

허브 VNet과 각 스포크 VNet은 동일한 Azure 지역의 동일한 Azure AD(Active Directory) 테넌트에 속하는 한 서로 다른 리소스 그룹이나 서로 다른 구독에 구현될 수 있습니다. 따라서 허브 VNet에서 관리되는 서비스를 공유하면서도 각 워크로드를 분산 관리할 수 있습니다.

### <a name="vnet-and-gatewaysubnet"></a>VNet과 GatewaySubnet

이름이 *GatewaySubnet*인 서브넷을 만듭니다. 주소 범위는 /27로 합니다. 이 서브넷은 가상 네트워크 게이트웨이에서 사용합니다. 이 서브넷에 주소 32개를 할당하면 추후 게이트웨이 크기 제한에 도달하지 않을 수 있습니다.

게이트웨이 설정에 대한 자세한 내용은 연결 유형에 따라 다음과 같은 참조 아키텍처를 참조하세요.

- [ExpressRoute를 사용하는 하이브리드 네트워크][guidance-expressroute]
- [VPN 게이트웨이를 사용하는 하이브리드 네트워크][guidance-vpn]

고가용성이 필요한 경우 장애 조치(failover)를 위해 ExpressRoute와 VPN을 모두 사용할 수 있습니다. [VPN 장애 조치(failover)를 사용하는 ExpressRoute를 사용하여 온-프레미스 네트워크를 Azure에 연결][hybrid-ha]을 참조하세요.

온-프레미스 네트워크에 연결하지 않아도 되는 경우에는 게이트웨이 없이 허브-스포크 토폴로지를 사용할 수도 있습니다. 

### <a name="vnet-peering"></a>VNet 피어링

VNet 피어링은 두 VNet 사이에 존재하는 비전이적 관계입니다. 스포크를 서로 연결해야 한다면 스포크 사이에 별도의 피어링 연결을 추가하는 방법도 고려할 수 있습니다.

그러나 여러 개의 스포크를 서로 연결해야 하는 경우에는 [VNet 하나당 허용되는 VNet 피어링 개수 제한][vnet-peering-limit]으로 인해 사용 가능한 피어링 연결이 모자라게 됩니다. 이 경우에는 목적지가 스포크인 트래픽이 허브 VNet에서 라우터로 기능하는 NVA로 강제로 전달되도록 UDR(사용자 정의 경로)을 사용하는 방법을 고려할 수 있습니다. 이렇게 하면 각 스포크가 다른 스포크와 연결할 수 있게 됩니다.

스포크가 허브 VNet 게이트웨이를 사용하여 원격 네트워크와 통신하도록 구성할 수도 있습니다. 게이트웨이 트래픽이 스포크에서 허브로 흐르고 원격 네트워크에 연결되도록 허용하려면:

  - 허브의 VNet 피어링 연결이 **게이트웨이 전송을 허용**하도록 구성해야 합니다.
  - 각 스포크의 VNet 피어링 연결이 **원격 게이트웨이를 사용**하도록 구성해야 합니다.
  - 모든 VNet 피어링 연결이 **전달된 트래픽을 허용**하도록 구성해야 합니다.

## <a name="considerations"></a>고려 사항

### <a name="spoke-connectivity"></a>스포크 연결

스포크 사이의 연결이 필요한 경우 허브에서의 라우팅을 위해 NVA를 구현하고 트래픽이 허브로 전달되도록 스포크에서 UDR을 사용하는 방법을 고려할 수 있습니다.

![[2]][2]

이 시나리오에서는 피어링 연결이 **전달된 트래픽을 허용**하도록 구성해야 합니다.

### <a name="overcoming-vnet-peering-limits"></a>VNet 피어링 제한 문제 해결

반드시 Azure의 [VNet 하나당 허용되는 VNet 피어링 개수 제한][vnet-peering-limit]을 고려해야 합니다. 허용되는 한도보다 많은 스포크가 필요한 경우 첫 번째 수준의 스포크가 허브로서 기능하는 허브-스포크-허브-스포크 토폴로지를 만드는 방법을 고려할 수 있습니다. 다음 다이어그램은 이 토폴로지를 보여줍니다.

![[3]][3]

또한, 허브가 다수의 스포크에 맞게 확장될 수 있도록 허브에서 어떤 서비스가 공유되는지도 고려해야 합니다. 예를 들어 허브에서 방화벽 서비스를 제공한다면 복수의 스포크를 추가할 때 방화벽 솔루션의 대역폭 제한을 고려해야 합니다. 공유 서비스 중 일부를 두 번째 수준의 허브로 이동하는 것이 좋을 수 있습니다.

## <a name="deploy-the-solution"></a>솔루션 배포

이 아키텍처에 대한 배포는 [GitHub][ref-arch-repo]에서 사용할 수 있습니다. 이 아키텍처에 대한 배포는 연결을 테스트하기 위해 각 VNet에서 VM을 사용합니다. **허브 VNet**의 **shared-services** 서브넷에 실제로 호스팅된 서비스는 없습니다.

배포는 구독에서 다음과 같은 리소스 그룹을 만듭니다.

- hub-nva-rg
- hub-vnet-rg
- onprem-jb-rg
- onprem-vnet-rg
- spoke1-vnet-rg
- spoke2-vent-rg

템플릿 매개 변수 파일은 이러한 이름을 참조하므로 이름을 변경하는 경우 매개 변수 파일을 일치하도록 업데이트합니다.

### <a name="prerequisites"></a>필수 조건

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a>시뮬레이션된 온-프레미스 데이터 센터 배포

시뮬레이션된 온-프레미스 데이터 센터를 Azure VNet으로서 배포하려면 다음 단계를 수행합니다.

1. 참조 아키텍처 리포지토리의 `hybrid-networking/hub-spoke` 폴더로 이동합니다.

2. `onprem.json` 파일을 엽니다. `adminUsername` 및 `adminPassword`에 대한 값을 대체합니다.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. (선택 사항) Linux 배포의 경우 `osType`을 `Linux`로 설정합니다.

4. 다음 명령 실행:

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. 배포가 완료될 때까지 기다립니다. 이 배포는 가상 네트워크, 가상 머신 및 VPN 게이트웨이를 생성합니다. VPN 게이트웨이를 만드는 데 약 40분 정도 걸릴 수 있습니다.

### <a name="deploy-the-hub-vnet"></a>허브 VNet 배포

허브 VNet을 배포하려면 다음 단계를 수행합니다.

1. `hub-vnet.json` 파일을 엽니다. `adminUsername` 및 `adminPassword`에 대한 값을 대체합니다.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (선택 사항) Linux 배포의 경우 `osType`을 `Linux`로 설정합니다.

3. `sharedKey`의 경우 VPN 연결에 대한 공유 키를 입력합니다. 

    ```bash
    "sharedKey": "",
    ```

4. 다음 명령 실행:

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. 배포가 완료될 때까지 기다립니다. 이 배포는 가상 네트워크, 가상 머신, VPN 게이트웨이 및 게이트웨이에 대한 연결을 생성합니다.  VPN 게이트웨이를 만드는 데 약 40분 정도 걸릴 수 있습니다.

### <a name="test-connectivity-with-the-hub"></a>허브를 사용하여 연결 테스트

시뮬레이션된 온-프레미스 환경에서 허브 VNet으로의 연결을 테스트합니다.

**Windows 배포**

1. Azure Portal을 사용하여 `onprem-jb-rg` 리소스 그룹에서 `jb-vm1`이라는 VM을 찾습니다.

2. `Connect`를 클릭하여 VM에 대한 원격 데스크톱 세션을 엽니다. `onprem.json` 매개 변수 파일에서 지정한 암호를 사용합니다.

3. VM에서 PowerShell 콘솔을 열고, `Test-NetConnection` cmdlet을 사용하여 허브 VNet에서 jumpbox VM에 연결할 수 있는지 확인합니다.

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
출력은 다음과 비슷해야 합니다.

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> 기본적으로 Windows Server VM을 사용하면 Azure에서 ICMP 응답을 허용하지 않습니다. `ping`을 사용하여 연결을 테스트하려는 경우 각 VM에 대한 Windows 고급 방화벽에서 ICMP 트래픽을 사용하도록 설정해야 합니다.

**Linux 배포**

1. Azure Portal을 사용하여 `onprem-jb-rg` 리소스 그룹에서 `jb-vm1`이라는 VM을 찾습니다.

2. `Connect`를 클릭하고 포털에 표시되는 `ssh` 명령을 복사합니다. 

3. Linux 프롬프트에서 `ssh`를 실행하여 시뮬레이션된 온-프레미스 환경에 연결합니다. `onprem.json` 매개 변수 파일에서 지정한 암호를 사용합니다.

4. `ping` 명령을 사용하여 허브 VNet에서 jumpbox VM에 대한 연결을 테스트합니다.

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a>스포크 VNet 배포

스포크 VNet을 배포하려면 다음 단계를 수행합니다.

1. `spoke1.json` 파일을 엽니다. `adminUsername` 및 `adminPassword`에 대한 값을 대체합니다.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. (선택 사항) Linux 배포의 경우 `osType`을 `Linux`로 설정합니다.

3. 다음 명령 실행:

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. `spoke2.json` 파일에 대해 1-2단계 반복합니다.

5. 다음 명령 실행:

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. 다음 명령 실행:

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a>연결 테스트

시뮬레이션된 온-프레미스 환경에서 스포크 VNet으로의 연결을 테스트합니다.

**Windows 배포**

1. Azure Portal을 사용하여 `onprem-jb-rg` 리소스 그룹에서 `jb-vm1`이라는 VM을 찾습니다.

2. `Connect`를 클릭하여 VM에 대한 원격 데스크톱 세션을 엽니다. `onprem.json` 매개 변수 파일에서 지정한 암호를 사용합니다.

3. VM에서 PowerShell 콘솔을 열고, `Test-NetConnection` cmdlet을 사용하여 스포크 VNet에서 jumpbox VM에 연결할 수 있는지 확인합니다.

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

**Linux 배포**

Linux VM을 사용하여 시뮬레이션된 온-프레미스 환경에서 스포크 VNet으로의 연결을 테스트하려면 다음 단계를 수행합니다.

1. Azure Portal을 사용하여 `onprem-jb-rg` 리소스 그룹에서 `jb-vm1`이라는 VM을 찾습니다.

2. `Connect`를 클릭하고 포털에 표시되는 `ssh` 명령을 복사합니다. 

3. Linux 프롬프트에서 `ssh`를 실행하여 시뮬레이션된 온-프레미스 환경에 연결합니다. `onprem.json` 매개 변수 파일에서 지정한 암호를 사용합니다.

5. `ping` 명령을 사용하여 각 스포크에서 jumpbox VM에 대한 연결을 테스트합니다.

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a>스포크 사이의 연결 추가

이 단계는 옵션입니다. 스포크를 서로 연결할 수 있도록 하려면 NVA(네트워크 가상 어플라이언스)를 허브 VNet의 라우터로 사용하고 다른 스포크에 연결하려고 할 때 스포크에서 라우터로 트래픽을 강제로 적용해야 합니다. 두 개의 스포크 VNet을 연결할 수 있는 UDR(사용자 정의 경로)과 함께 기본 샘플 NVA를 단일 VM으로 배포하려면 다음 단계를 수행합니다.

1. `hub-nva.json` 파일을 엽니다. `adminUsername` 및 `adminPassword`에 대한 값을 대체합니다.

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. 다음 명령 실행:

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Azure의 허브-스포크 토폴로지"
[1]: ./images/hub-spoke-gateway-routing.svg "전이적 라우팅이 사용되는 Azure의 허브-스포크 토폴로지"
[2]: ./images/hub-spoke-no-gateway-routing.svg "NVA로 전이적 라우팅이 사용되는 Azure의 허브-스포크 토폴로지"
[3]: ./images/hub-spokehub-spoke.svg "Azure의 허브-스포크-허브-스포크 토폴로지"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
