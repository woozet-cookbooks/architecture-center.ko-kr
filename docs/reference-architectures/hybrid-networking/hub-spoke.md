---
title: "Azure에서 허브-스포크 네트워크 토폴로지 구현"
description: "Azure에서 허브-스포크 네트워크 토폴로지를 구현하는 방법입니다."
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
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

## <a name="architecture"></a>건축

이 아키텍처는 다음 구성 요소로 구성됩니다.

* **온-프레미스 네트워크**. 조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.

* **VPN 장치**. 온-프레미스 네트워크에 외부 연결을 제공하는 장치 또는 서비스입니다. VPN 장치는 하드웨어 장치일 수도 있고 Windows Server 2012의 RRAS(라우팅 및 원격 액세스 서비스)와 같은 소프트웨어 솔루션일 수도 있습니다. 지원되는 VPN 어플라이언스 목록 및 선택한 VPN 어플라이언스를 Azure에 연결하도록 구성하는 방법에 대한 자세한 내용은 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치 정보][vpn-appliance]를 참조하세요.

* **VPN 가상 네트워크 게이트웨이 또는 ExpressRoute 게이트웨이**. 가상 네트워크 게이트웨이를 사용하면 VNet을 온-프레미스 네트워크에 연결하는 데 사용되는 VPN 장치 또는 ExpressRoute 회로에 연결할 수 있습니다. 자세한 내용은 [온-프레미스 네트워크를 Microsoft Azure virtual network에 연결][connect-to-an-Azure-vnet]을 참조하세요.

> [!NOTE]
> 이 참조 아키텍처의 배포 스크립트는 연결을 위해 VPN 게이트웨이를 사용하고, 사용자의 온-프레미스 네트워크를 시뮬레이션하기 위해 Azure의 VNet을 사용합니다.

* **허브 VNet**. 허브-스포크 토폴로지에서 허브로 사용되는 Azure VNet입니다. 허브는 사용자의 온-프레미스 네트워크에 대한 연결의 중앙 위치이자 스포크 VNet에 호스팅된 다양한 워크로드에 의해 사용되는 서비스가 호스팅되는 장소입니다.

* **게이트웨이 서브넷**. 가상 네트워크 게이트웨이는 동일한 서브넷에 있습니다.

* **공유 서비스 서브넷**. DNS, AD DS를 비롯한 모든 스포크에서 공유될 수 있는 서비스를 호스팅하는 데 사용되는 허브 VNet의 서브넷입니다.

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

이 아키텍처에 대한 배포는 [GitHub][ref-arch-repo]에서 사용할 수 있습니다. 이 아키텍처에 대한 배포는 연결을 테스트하기 위해 각 VNet에서 Ubuntu VM을 사용합니다. **허브 VNet**의 **shared-services** 서브넷에 실제로 호스팅된 서비스는 없습니다.

### <a name="prerequisites"></a>필수 조건

사용자의 구독에 참조 아키텍처를 배포하려면 먼저 다음 단계를 수행해야 합니다.

1. [AzureCAT 참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.

2. Azure CLI를 사용하려면 컴퓨터에 Azure CLI 2.0이 설치되어 있어야 합니다. CLI를 설치하려면 [Install Azure CLI 2.0][azure-cli-2](Azure CLI 2.0 설치)에 제시된 지침을 참조하세요.

3. PowerShell을 사용하려면 컴퓨터에 Azure용 최신 PowerShell 모듈이 설치되어 있어야 합니다. 최신 Azure PowerShell 모듈을 설치하려면 [Azure용 PowerShell 설치][azure-powershell]에 제시된 지침을 참조하세요.

4. 명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 다음 명령 중 하나를 사용하여 Azure 계정에 로그인한 다음 프롬프트에 따릅니다.

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>시뮬레이션된 온-프레미스 데이터 센터 배포

시뮬레이션된 온-프레미스 데이터 센터를 Azure VNet으로서 배포하려면 다음 단계를 수행합니다.

1. 위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\onprem` 폴더로 이동합니다.

2. `onprem.vm.parameters.json` 파일을 열고 아래에 나와 있는 대로 11열과 12열에 큰따옴표 사이에 사용자 이름과 암호를 입력한 다음 파일을 저장합니다.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. 아래의 bash 또는 PowerShell 명령을 실행하여 시뮬레이션된 온-프레미스 환경을 Azure의 VNet으로서 배포합니다. 값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > `ra-onprem-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.

4. 배포가 완료될 때까지 기다립니다. 이 배포는 가상 네트워크, Ubuntu를 실행하는 가상 머신 및 VPN 게이트웨이를 생성합니다. VPN 게이트웨이의 생성이 완료되기까지 40분 이상이 걸릴 수 있습니다.

### <a name="azure-hub-vnet"></a>Azure 허브 VNet

허브 VNet을 배포하고 위에서 만든 시뮬레이션된 온-프레미스 VNet에 연결하려면 다음 단계를 수행합니다.

1. 위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\hub` 폴더로 이동합니다.

2. `hub.vm.parameters.json` 파일을 열고 아래에 나와 있는 대로 11열과 12열에 큰따옴표 사이에 사용자 이름과 암호를 입력한 다음 파일을 저장합니다.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. `hub.gateway.parameters.json` 파일을 열고 아래에 나와 있는 대로 23열에 큰따옴표 사이에 공유 키를 입력한 다음 파일을 저장합니다. 이 값은 나중에 배포에서 필요하므로 따로 기록해 둡니다.

  ```bash
  "sharedKey": "",
  ```

4. 아래의 bash 또는 PowerShell 명령을 실행하여 시뮬레이션된 온-프레미스 환경을 Azure의 VNet으로서 배포합니다. 값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > `ra-hub-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.

5. 배포가 완료될 때까지 기다립니다. 이 배포는 가상 네트워크, Ubuntu를 실행하는 가상 머신, VPN 게이트웨이 및 이전 섹션에서 생성한 게이트웨이에 대한 연결을 생성합니다. VPN 게이트웨이의 생성이 완료되기까지 40분 이상이 걸릴 수 있습니다.

### <a name="connection-from-on-premises-to-the-hub"></a>온-프레미스에서 허브로의 연결

시뮬레이션된 온-프레미스 데이터 센터를 허브 VNet에 연결하려면 다음 단계를 수행합니다.

1. 위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\onprem` 폴더로 이동합니다.

2. `onprem.connection.parameters.json` 파일을 열고 아래에 나와 있는 대로 9열에 큰따옴표 사이에 공유 키를 입력한 다음 파일을 저장합니다. 이 공유 키 값은 앞에서 배포한 온-프레미스 게이트웨이에서 사용하는 값과 동일해야 합니다.

  ```bash
  "sharedKey": "",
  ```

3. 아래의 bash 또는 PowerShell 명령을 실행하여 시뮬레이션된 온-프레미스 환경을 Azure의 VNet으로서 배포합니다. 값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > `ra-onprem-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.

4. 배포가 완료될 때까지 기다립니다. 이 배포는 온-프레미스 데이터 센터를 시뮬레이션하는 데 사용되는 VNet과 허브 VNet 사이의 연결을 만듭니다.

### <a name="azure-spoke-vnets"></a>Azure 스포크 VNet

스포크 VNet을 배포하고 위에서 만든 허브 VNet에 연결하려면 다음 단계를 수행합니다.

1. 위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\spokes` 폴더로 이동합니다.

2. `spoke1.web.parameters.json` 파일을 열고 아래에 나와 있는 대로 53열과 54열에 큰따옴표 사이에 사용자 이름과 암호를 입력한 다음 파일을 저장합니다.

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. 파일 `spoke2.web.parameters.json`에 대해 직전 단계를 반복합니다.

4. 아래의 bash 또는 PowerShell 명령을 실행하여 첫 번째 스포크를 배포하고 허브에 연결합니다. 값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > `ra-spoke1-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.

5. 배포가 완료될 때까지 기다립니다. 이 배포는 가상 네트워크, Ubuntu와 Apache를 실행하는 3개의 가상 머신 및 이전 섹션에서 생성한 허브 VNet에 대한 VNet 피어링 연결을 생성합니다. 이 배포가 완료되기까지 20분이 넘게 걸릴 수 있습니다.

6. 아래의 bash 또는 PowerShell 명령을 실행하여 첫 번째 스포크를 배포하고 허브에 연결합니다. 값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > `ra-spoke2-rg`이 아닌 다른 리소스 그룹 이름을 사용하려면 해당 이름을 사용하는 모든 매개 변수 파일을 검색하여 각 파일에서 사용자의 리소스 그룹 이름을 사용하도록 편집해야 합니다.

5. 배포가 완료될 때까지 기다립니다. 이 배포는 가상 네트워크, Ubuntu와 Apache를 실행하는 3개의 가상 머신 및 이전 섹션에서 생성한 허브 VNet에 대한 VNet 피어링 연결을 생성합니다. 이 배포가 완료되기까지 20분이 넘게 걸릴 수 있습니다.

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a>스포크 VNet에 대한 Azure 허브 VNet 피어링

허브 VNet에 대한 VNet 피어링 연결을 배포하려면 다음 단계를 수행합니다.

1. 위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\hub` 폴더로 이동합니다.

2. 아래의 bash 또는 PowerShell 명령을 실행하여 첫 번째 스포크로의 피어링 연결을 배포합니다. 값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. 아래의 bash 또는 PowerShell 명령을 실행하여 두 번째 스포크로의 피어링 연결을 배포합니다. 값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a>연결 테스트

온-프레미스 데이터 센터 배포에 연결된 허브-스포크 토폴로지가 올바르게 작동하는지 확인하려면 다음 단계를 수행합니다.

1. [Azure Portal][포털]에서 사용자의 구독에 연결한 다음 `ra-onprem-rg` 리소스 그룹에 있는 `ra-onprem-vm1` 가상 머신으로 이동합니다.

2. `Overview` 블레이드에서 VM의 `Public IP address`를 확인합니다.

3. SSH 클라이언트에서 배포 중에 지정한 사용자 이름과 암호를 사용하여 위에서 확인한 IP 주소에 연결합니다.

4. 접속한 VM의 명령 프롬프트에서 아래의 명령을 실행하여 온-프레미스 VNet과 Spoke1 VNet 사이의 연결을 테스트합니다.

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a>스포크 사이의 연결 추가

각 스포크 사이의 연결을 허용하려면 목적지가 다른 스포크인 트래픽을 허브 VNet의 게이트웨이로 전달하는 UDR을 각 스포크에 배포해야 합니다. 아래의 단계를 수행하여 현재 하나의 스포크에서 다른 스포크로 연결할 수 없음을 확인한 다음 UDR을 배포하고 연결을 다시 테스트합니다.

1. 더 이상 jumpbox VM에 연결되어 있지 않다면 위의 1~4단계를 반복합니다.

2. 스포크 1에 있는 웹 서버 중 하나에 연결합니다.

  ```bash
  ssh 10.1.1.37
  ```

3. 스포크 1과 스포크 2 사이의 연결을 테스트합니다. 연결되지 않는 것이 정상입니다.

  ```bash
  ping 10.1.2.37
  ```

4. 컴퓨터의 명령 프롬프트로 이동합니다.

5. 위의 필수 조건 단계에서 다운로드한 리포지토리가 있는 `hybrid-networking\hub-spoke\spokes` 폴더로 이동합니다.

6. 아래의 bash 또는 PowerShell 명령을 실행하여 첫 번째 스포크에 UDR을 배포합니다. 값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. 아래의 bash 또는 PowerShell 명령을 실행하여 두 번째 스포크에 UDR을 배포합니다. 값을 사용자의 구독, 리소스 그룹 이름 및 Azure 지역으로 변경합니다.

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. 다시 ssh 터미널로 이동합니다.

9. 스포크 1과 스포크 2 사이의 연결을 테스트합니다. 이번에는 연결에 성공해야 합니다.

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
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

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Azure의 허브-스포크 토폴로지"
[1]: ./images/hub-spoke-gateway-routing.svg "전이적 라우팅이 사용되는 Azure의 허브-스포크 토폴로지"
[2]: ./images/hub-spoke-no-gateway-routing.svg "NVA로 전이적 라우팅이 사용되는 Azure의 허브-스포크 토폴로지"
[3]: ./images/hub-spokehub-spoke.svg "Azure의 허브-스포크-허브-스포크 토폴로지"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
