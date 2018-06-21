---
title: VPN을 사용하여 온-프레미스 네트워크를 Azure에 연결
description: VPN을 사용하여 연결된 온-프레미스 네트워크 및 Azure Virtual Network를 포괄하는 보안 사이트 간 네트워크 아키텍처를 구현하는 방법입니다.
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: dafcee6607d9cc7c56c332f9ed5d9568ff70f0e7
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/30/2018
ms.locfileid: "30270696"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a>VPN 게이트웨이를 사용하여 온-프레미스 네트워크를 Azure에 연결

이 참조 아키텍처에서는 사이트 간 VPN(가상 사설 네트워크)을 사용하여 온-프레미스 네트워크를 Azure로 확장하는 방법을 설명합니다. IPSec VPN 터널을 통해 온-프레미스 네트워크와 Azure 가상 네트워크(VNet) 사이에서 트래픽이 흐릅니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>건축 

이 아키텍처는 다음 구성 요소로 구성됩니다.

* **온-프레미스 네트워크**. 조직 내에서 실행되는 개인 로컬 영역 네트워크입니다.

* **VPN 어플라이언스**. 온-프레미스 네트워크에 외부 연결을 제공하는 장치 또는 서비스입니다. VPN 어플라이언스는 하드웨어 장치일 수도 있고 Windows Server 2012의 RRAS(라우팅 및 원격 액세스 서비스)와 같은 소프트웨어 솔루션일 수도 있습니다. 지원되는 VPN 어플라이언스 목록 및 VPN 어플라이언스가 Azure VPN 게이트웨이에 연결되도록 구성하는 방법은 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치 정보][vpn-appliance] 문서에서 선택한 장치에 대한 지침을 참조하세요.

* **가상 네트워크(VNet)**. Azure VPN 게이트웨이의 클라우드 응용 프로그램과 구성 요소는 동일한 [VNet][azure-virtual-network]에 존재합니다.

* **Azure VPN 게이트웨이**. [VPN 게이트웨이][azure-vpn-gateway] 서비스를 사용하면 VPN 어플라이언스를 통해 VNet을 온-프레미스 네트워크에 연결할 수 있습니다. 자세한 내용은 [온-프레미스 네트워크를 Microsoft Azure virtual network에 연결][connect-to-an-Azure-vnet]을 참조하세요. VPN 게이트웨이에는 다음과 같은 요소가 포함되어 있습니다.
  
  * **가상 네트워크 게이트웨이**. VNet을 위한 가상 VPN 어플라이언스를 제공하는 리소스입니다. 온-프레미스 네트워크에서 VNet으로 트래픽을 라우팅하는 역할을 담당합니다.
  * **로컬 네트워크 게이트웨이**. 온-프레미스 VPN 어플라이언스가 추상화된 것입니다. 클라우드 응용 프로그램에서 온-프레미스 네트워크로 흐르는 네트워크 트래픽은 이 게이트웨이를 통과하도록 라우팅됩니다.
  * **연결**. 이 연결에는 연결 유형(IPSec) 및 트래픽을 암호화하기 위해 온-프레미스 VPN 어플라이언스와 공유되는 키를 지정하는 속성이 있습니다.
  * **게이트웨이 서브넷**. 가상 네트워크 게이트웨이는 자체 서브넷에 존재합니다. 자체 서브넷은 아래의 권장 사항 섹션에서 설명하는 다양한 요구 사항에 따라 달라집니다.

* **클라우드 응용 프로그램**. Azure에 호스팅된 응용 프로그램입니다. Azure Load Balancer를 통해 여러 서브넷이 연결된 여러 계층이 포함될 수 있습니다. 응용 프로그램 인프라에 대한 자세한 내용은 [Windows VM 워크로드 실행][windows-vm-ra] 및 [Linux VM 워크로드 실행][linux-vm-ra]을 참조하세요.

* **내부 부하 분산 장치**. VPN 게이트웨이에서 전송되는 네트워크 트래픽은 내부 부하 분산 장치를 통해 클라우드 응용 프로그램으로 라우팅됩니다. 부하 분산 장치는 응용 프로그램의 프론트 엔드 서브넷에 위치합니다.

## <a name="recommendations"></a>권장 사항

대부분의 시나리오의 경우 다음 권장 사항을 적용합니다. 이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.

### <a name="vnet-and-gateway-subnet"></a>VNet과 게이트웨이 서브넷

필요한 리소스를 모두 수용할 수 있을 만큼 충분한 주소 공간을 사용하여 Azure VNet을 만듭니다. 추후 더 많은 VM이 필요할 경우 여분이 충분하도록 VNet 주소 공간을 할당합니다. VNet 주소 공간은 온-프레미스 네트워크와 겹치지 않아야 합니다. 예를 들어 위 다이어그램에서는 VNet에 10.20.0.0/16의 주소 공간을 사용하고 있습니다.

이름이 *GatewaySubnet*인 서브넷을 만듭니다. 주소 범위는 /27로 합니다. 이 서브넷은 가상 네트워크 게이트웨이에서 사용합니다. 이 서브넷에 주소 32개를 할당하면 추후 게이트웨이 크기 제한에 도달하지 않을 수 있습니다. 주소 공간 가운데에 이 서브넷을 배치하지 않습니다. 게이트웨이 서브넷을 VNet 주소 공간의 상단 끝에 배치하는 것이 좋습니다. 다이어그램에서는 10.20.255.224/27을 사용하고 있습니다.  다음은 [CIDR]을 간단하게 계산하는 방법입니다.

1. VNet 주소 공간의 변수 비트를 게이트웨이 서브넷에 의해 사용되는 비트까지 모두 1로 설정하고, 나머지 비트를 0으로 설정합니다.
2. 결과 비트를 10진수로 변환한 다음 이것을 게이트웨이 서브넷의 크기로 설정된 접두사 길이를 사용하여 주소 공간으로 표현합니다.

예를 들어 IP 주소 범위가 10.20.0.0/16인 VNet에 1단계를 적용하면 10.20.0b11111111.0b11100000이 됩니다.  이것을 10진수로 변환한 다음 주소 공간으로 표현하면 10.20.255.224/27이 됩니다. 

> [!WARNING]
> 게이트웨이 서브넷에는 VM을 배포하지 않습니다. 또한, 이 서브넷에 NSG을 할당하지 않습니다. NSG를 할당하면 게이트웨이의 작동이 중지됩니다.
> 
> 

### <a name="virtual-network-gateway"></a>가상 네트워크 게이트웨이

가상 네트워크 게이트웨이에 공용 IP 주소를 할당합니다.

게이트웨이 서브넷에 가상 네트워크 게이트웨이를 만든 다음 이것을 새로 할당된 공용 IP 주소에 할당합니다. 사용자 요구 사항에 가장 가까우면서 사용자의 VPN 어플라이언스가 지원하는 게이트웨이 유형을 사용합니다.

- 주소 접두사와 같은 정책 조건을 바탕으로 요청이 라우팅되는 방식을 제어해야 하는 경우 [policy-based gateway][policy-based-routing]를 만듭니다. 정책 기반 게이트웨이는 정적 라우팅을 사용하며, 사이트 간 연결에서만 작동합니다.

- RRAS를 사용하여 온-프레미스 네트워크에 연결하거나 다중 사이트 또는 교차 지역 연결을 지원하는 경우, [route-based gateway][route-based-routing]를 만들거나 VNet 간 연결을 구현합니다(복수의 VNet을 이동하는 경로 포함). 경로 기반 게이트웨이는 네트워크 간에 트래픽을 전달할 때 동적 라우팅을 사용합니다. 동적 라우팅은 대체 경로를 시도하므로 정적 라우팅보다 네트워크 경로 장애에 대한 내결함성이 높습니다. 경로 기반 게이트웨이에서는 네트워크 주소가 변경되어도 경로를 수동으로 업데이트할 필요가 없기 때문에 관리 오버헤드를 줄일 수 있습니다.

지원되는 VPN 어플라이언스 목록은 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치 정보][vpn-appliances]를 참조하세요.

> [!NOTE]
> 게이트웨이를 만든 뒤에는 게이트웨이 유형을 변경하려면 게이트웨이를 삭제하고 다시 만들어야 합니다.
> 
> 

사용자의 처리량 요구 사항에 가장 가까운 Azure VPN 게이트웨이 SKU를 선택합니다. Azure VPN 게이트웨이는 아래 표에 제시된 것처럼 세 가지 SKU로 제공됩니다. 

| SKU | VPN 처리량 | 최대 IPSec 터널 |
| --- | --- | --- |
| Basic |100Mbps |10 |
| Standard |100Mbps |10 |
| 고성능 |200Mbps |30 |

> [!NOTE]
> 기본 SKU는 Azure ExpressRoute와 호환되지 않습니다. 게이트웨이를 만든 뒤에 [SKU를 변경][changing-SKUs]할 수 있습니다.
> 
> 

요금은 게이트웨이가 프로비전되고 사용된 시간에 따라 청구됩니다. [VPN 게이트웨이 가격][azure-gateway-charges]을 참조하세요.

요청을 응용 프로그램 VM으로 직접 전달하는 대신 게이트웨이로부터 수신되는 응용 프로그램 트래픽을 내부 부하 분산 장치로 전달하는 게이트웨이 서브넷을 위한 라우팅 규칙을 만듭니다.

### <a name="on-premises-network-connection"></a>온-프레미스 네트워크 연결

로컬 네트워크 게이트웨이를 만듭니다. 온-프레미스 VPN 어플라이언스의 공용 IP 주소와 온-프레미스 네트워크의 주소 공간을 지정합니다. 온-프레미스 VPN 어플라이언스는 Azure VPN 게이트웨이에서 로컬 네트워크 게이트웨이에 의해 액세스할 수 있는 공용 IP 주소를 가져야 합니다. VPN 장치는 NAT(Network Address Translator) 뒤에 배치될 수 없습니다.

가상 네트워크 게이트웨이와 로컬 네트워크 게이트웨이의 사이트 간 연결을 만듭니다. 사이트 간(IPSec) 연결 유형을 선택하고 공유 키를 지정합니다. Azure VPN 게이트웨이를 사용하는 사이트 간 암호는 IPSec 프로토콜을 기반으로 하며, 인증을 위해 사전 공유된 키를 사용합니다. 사전 공유된 키는 Azure VPN 게이트웨이를 만들 때 사용자가 지정합니다. 온-프레미스에서 실행되는 VPN 어플라이언스도 동일한 키로 구성해야 합니다. 현재 다른 인증 메커니즘은 지원되지 않습니다.

목적지가 Azure VNet에 속한 주소인 요청이 VPN 장치로 전달되도록 온-프레미스 라우팅 인프라가 구성되어 있어야 합니다.

온-프레미스 네트워크에서 클라우드 응용 프로그램에 필요한 포트를 모두 엽니다.

연결을 테스트하여 다음을 확인합니다.

* 온-프레미스 VPN 어플라이언스가 Azure VPN 게이트웨이를 통해 트래픽을 클라우드 응용 프로그램으로 올바르게 라우팅합니다.
* VNet이 트래픽을 온-프레미스 네트워크로 올바르게 라우팅합니다.
* 두 방향에서 금지된 트래픽이 올바르게 차단됩니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

기본 또는 표준 VPN 게이트웨이 SKU에서 고성능 VPN SKU로 바꾸면 제한된 수직 확장성을 달성할 수 있습니다.

다량의 VPN 트래픽이 예상되는 VNet이라면 서로 다른 워크로드를 보다 작은 여러 개의 VNet으로 분배하고 각각에 대해 VPN 게이트웨이를 구성하는 방법을 고려할 수 있습니다.

VNet은 수평 또는 수직으로 분할할 수 있습니다. 수평으로 분할하려면 각 계층에 속한 일부 VM 인스턴스를 새로운 VNet의 서브넷으로 이동합니다. 이렇게 하면 각 VNet이 동일한 구조와 기능을 갖게 됩니다. 수직으로 분할하려면 기능이 서로 다른 논리 영역(주문 처리, 송장 발행, 고객 계정 관리 등)으로 분할되도록 각 계층을 다시 디자인합니다. 이렇게 한 뒤 각 기능 영역을 자체 VNet에 배치할 수 있습니다.

VNet에 온-프레미스 Active Directory 도메인 컨트롤러를 복제하고 VNet에 DNS를 구현하면 온-프레미스에서 클라우드로 흐르는 보안 관련 트래픽과 관리 트래픽 중 일부를 줄일 수 있습니다. 자세한 내용은 [AD DS(Active Directory Domain Services)를 Azure로 확장][adds-extend-domain]을 참조하세요.

## <a name="availability-considerations"></a>가용성 고려 사항

Azure VPN 게이트웨이에서 온-프레미스 네트워크에 계속해서 액세스할 수 있도록 하려면 온-프레미스 VPN 게이트웨이를 위한 장애 조치(failover) 클러스터를 구현합니다.

조직에 여러 개의 온-프레미스 사이트가 있는 경우 하나 이상의 Azure VNet에 [다중 사이트 연결][vpn-gateway-multi-site]을 만듭니다. 이렇게 하려면 동적(경로 기반) 라우팅이 필요하므로 온-프레미스 VPN 게이트웨이가 이 기능을 지원하는지 확인해야 합니다.

서비스 수준 계약에 대한 자세한 내용은 [VPN 게이트웨이의 SLA][sla-for-vpn-gateway]를 참조하세요. 

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

온-프레미스 VPN 어플라이언스에서 진단 정보를 모니터링합니다. 이 프로세스는 VPN 어플라이언스에서 제공하는 기능에 따라 달라집니다. 예를 들어 Windows Server 2012에서 RRAS(라우팅 및 원격 액세스 서비스)를 사용한다면 [RRAS 로깅][rras-logging]이 모니터링 기능이 됩니다.

[Azure VPN 게이트웨이 진단][gateway-diagnostic-logs]을 사용하여 연결 문제 정보를 캡처합니다. 이 로그는 연결 요청의 소스와 목적지, 사용된 프로토콜, 연결이 설정된 방식(또는 시도가 실패한 이유)과 같은 정보를 추적하는 데 사용할 수 있습니다.

Azure Portal에서 제공되는 감사 로크를 사용하여 Azure VPN 게이트웨이의 운영 로그를 모니터링합니다. 로컬 네트워크 게이트웨이, Azure 네트워크 게이트웨이 및 연결 각각에 대한 개별적인 로그도 사용 가능합니다. 이 정보는 게이트웨이에 적용된 변경 사항을 추적하는 데 사용할 수 있고, 잘 작동하는 게이트웨이가 어떤 이유로 작동이 중단되는 경우에도 유용합니다.

![[2]][2]

연결을 모니터링하고 연결 장애 이벤트를 추적합니다. [Nagios][nagios]와 같은 모니터링 패키지를 사용하여 이 정보를 캡처 및 보고할 수 있습니다.

## <a name="security-considerations"></a>보안 고려 사항

각 VPN 게이트웨이별로 서로 다른 공유 키를 생성합니다. 무작위 공격에 대비할 수 있도록 강력한 공유 키를 사용합니다.

> [!NOTE]
> 현재 Azure Key Vault를 사용하여 Azure VPN 게이트웨이의 키를 사전에 공유할 수 없습니다.
> 
> 

온-프레미스 VPN 어플라이언스가 [Azure VPN 게이트웨이와 호환되는][vpn-appliance-ipsec] 암호화 방법을 사용하는지 확인해야 합니다. 정책 기반 라우팅의 경우 Azure VPN 게이트웨이는 AES256, AES128 및 3DES 암호화 알고리즘을 지원합니다. 경로 기반 게이트웨이의 경우 AES256 및 3DES를 지원합니다.

사용 중인 온-프레미스 VPN 어플라이언스가 경계 네트워크와 인터넷 사이에 존재하는 방화벽을 갖는 DMZ(경계 네트워크)상에 있는 경우에는 사이트 간 VPN 연결을 허용하기 위해 [추가적인 방화벽 규칙][additional-firewall-rules]을 구성해야 할 수 있습니다.

VNet의 응용 프로그램이 인터넷으로 데이터를 전송하는 경우에는 목적지가 인터넷인 모든 트래픽이 온-프레미스 네트워크를 통과하여 라우팅되도록 [강제 터널링을 구현][forced-tunneling]하는 방법을 고려할 수 있습니다. 이렇게 하면 온-프레미스 인프라에서 응용 프로그램에 의해 발신 요청을 감사할 수 있습니다.

> [!NOTE]
> 강제 터널링은 Azure 서비스(저장소 서비스 등)와 Windows 라이선스 관리자로의 연결에 영향을 줄 수 있습니다.
> 
> 


## <a name="troubleshooting"></a>문제 해결 

일반적인 VPN 관련 오류를 해결하려면 [일반적인 VPN 관련 오류 해결][troubleshooting-vpn-errors]을 참조하세요.

온-프레미스 VPN 어플라이언스가 올바르게 작동하고 있는지 확인할 때 다음과 같은 권장 사항이 유용합니다.

- **VPN 어플라이언스에 의해 생성된 로그 파일에 오류나 장애가 있는지 확인합니다.**

    이렇게 하면 VPN 어플라이언스가 올바르게 작동하고 있는지 판단하는 데 도움이 됩니다. 이 정보가 저장된 위치는 사용 중인 어플라이언스에 따라 다릅니다. 예를 들어 Windows Server 2012에서 RRAS를 사용하는 경우에는 다음 PowerShell 명령을 사용하여 RRAS 서비스의 오류 이벤트 정보를 표시할 수 있습니다.

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    각 항목의 *Message* 속성은 오류에 대한 설명을 제공합니다. 몇 가지 일반적인 예는 다음과 같습니다.

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    다음 PowerShell 명령을 사용하여 RRAS 서비스를 통해 연결하려는 시도에 대한 이벤트 로그 정보를 얻을 수 있습니다. 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    연결 장애가 발생하면 이 로그에 다음과 비슷한 오류가 포함됩니다.

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- **VPN 게이트웨이의 연결 및 라우팅을 확인합니다.**

    VPN 어플라이언스가 Azure VPN 게이트웨이를 통해 트래픽을 올바르게 라우팅하지 않고 있을 수 있습니다. [PsPing][psping]과 같은 도구를 사용하여 VPN 게이트웨이의 연결 및 라우팅을 확인합니다. 예를 들어 온-프레미스 머신에서 VNet에 위치한 웹 서버로의 연결을 테스트하려면 다음 명령을 실행합니다(`<<web-server-address>>`를 웹 서버의 주소로 대체합니다).

    ```
    PsPing -t <<web-server-address>>:80
    ```

    온-프레미스 머신이 웹 서버로 트래픽을 올바르게 라우팅하고 있다면 다음과 같은 출력이 표시됩니다.

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    온-프레미스 머신이 해당 목적지와 올바르게 통신하지 못하고 있다면 다음과 같은 메시지가 표시됩니다.

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- **온-프레미스 방화벽이 VPN 트래픽이 통과할 수 있도록 허용하고 있으며 올바른 포트가 열려 있는지 확인합니다.**

- **온-프레미스 VPN 어플라이언스가 [Azure VPN 게이트웨이와 호환되는][vpn-appliance] 암호화 방법을 사용하고 있는지 확인합니다.** 정책 기반 라우팅의 경우 Azure VPN 게이트웨이는 AES256, AES128 및 3DES 암호화 알고리즘을 지원합니다. 경로 기반 게이트웨이의 경우 AES256 및 3DES를 지원합니다.

Azure VPN 게이트웨이에 문제가 있는지 판단할 때 다음과 같은 권장 사항이 유용합니다.

- **[Azure VPN 게이트웨이 진단 로그][gateway-diagnostic-logs]에 잠재적인 문제가 없는지 검사합니다.**

- **Azure VPN 게이트웨이와 온-프레미스 VPN 어플라이언스가 동일한 공유 인증 키로 구성되어 있는지 확인합니다.**

    다음 Azure CLI 명령을 사용하여 Azure VPN 게이트웨이에 저장된 공유 키를 확인할 수 있습니다.

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    사용 중인 온-프레미스 VPN 어플라이언스에 적합한 명령을 사용하여 해당 어플라이언스에 구성된 공유 키를 표시합니다.

    Azure VPN 게이트웨이가 속한 *GatewaySubnet* 서브넷이 NSG에 연결되어 있지 않은지 확인합니다.

    다음 Azure CLI 명령을 사용하여 서브넷 정보를 확인할 수 있습니다.

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    *Network Security Group id*라는 이름을 갖는 데이터 필드가 없는지 확인합니다. 다음은 NSG가 할당된 *GatewaySubnet*의 인스턴스의 결과입니다(*VPN-Gateway-Group*). 이렇게 구성되어 있으면 이 NSG에 대해 규칙이 정의되어 있는 경우 게이트웨이가 올바르게 작동하지 않을 수 있습니다.

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- **Azure VNet의 가상 머신이 VNet 외부로부터 수신되는 트래픽을 허용되도록 구성되어 있는지 확인합니다.**

    이러한 가상 머신이 속한 서브넷에 연결된 NSG 규칙이 있는지 확인합니다. 다음 Azure CLI 명령을 사용하여 모든 NSG 규칙을 확인할 수 있습니다.

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- **Azure VPN 게이트웨이가 연결되어 있는지 확인합니다.**

    다음 Azure PowerShell 명령을 사용하여 Azure VPN 연결의 현재 상태를 확인할 수 있습니다. `<<connection-name>>` 매개 변수는 가상 네트워크 게이트웨이와 로컬 게이트웨이를 연결하는 Azure VPN 연결의 이름입니다.

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    다음 스니펫에서는 게이트웨이가 연결되어 있는 경우(첫 번째 예제)와 연결이 해제된 경우(두 번째 예제)에 생성되는 출력을 보여줍니다.

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

호스트 VM 구성, 네트워크 대역폭 사용 현황 또는 응용 프로그램 성능에 문제가 있는지 판단할 때 다음과 같은 권장 사항이 유용합니다.

- **서브넷에 속한 Azure VM에서 실행 중인 게스트 운영 체제의 방화벽이 온-프레미스 IP 범위로부터 수신되는 허용된 트래픽을 허용하도록 올바르게 구성되어 있는지 확인합니다.**

- **트래픽의 양이 Azure VPN 게이트웨이에서 사용할 수 있는 대역폭의 제한에 가깝지 않은지 확인합니다.**

    이것을 확인하는 방법은 온-프레미스에서 실행 중인 VPN 어플라이언스에 따라 달라집니다. 예를 들어 Windows Server 2012에서 RRAS를 사용하고 있는 경우 Performance Monitor를 사용하여 VPN 연결을 통해 송수신되고 있는 데이터의 양을 추적할 수 있습니다. *RAS 총계* 개체를 사용하여 *수신된 바이트/초* 카운터와 *송신된 바이트/초* 카운터를 선택합니다.

    ![[3]][3]

    결과를 VPN 게이트웨이에서 사용할 수 있는 대역폭(기본 및 표준 SKU: 100Mbps, 고성능 SKU: 200Mbps)과 비교합니다.

    ![[4]][4]

- **응용 프로그램 부하에 맞는 올바른 VM 개수와 크기를 배포했는지 확인합니다.**

    Azure VNet의 가상 머신 중 느리게 실행되고 있는 VM은 없는지 확인합니다. 느리게 실행되고 있는 VM이 있다면 과부하되었거나 부하를 처리할 VM이 너무 적거나 부하 분산 장치가 올바르게 구성되어 있지 않은 것일 수 있습니다. 원인을 알아내려면 [진단 정보를 캡처 및 분석][azure-vm-diagnostics]합니다. 결과는 Azure Portal을 사용하여 검사할 수 있으며, 그 밖에도 다양한 타사 도구를 사용하여 성능 데이터를 자세히 검사할 수 있습니다.

- **응용 프로그램이 클라우드 리소스를 효율적으로 사용하고 있는지 확인합니다.**

    각 VM에서 실행 중인 응용 프로그램 코드를 이용하여 각 응용 프로그램에서 리소스를 최대한 효율적으로 사용하고 있는지 확인합니다. [Application Insights][application-insights]와 같은 도구를 사용할 수 있습니다.

## <a name="deploy-the-solution"></a>솔루션 배포


**필수 조건.** 적절한 네트워크 어플라이언스를 사용하여 기존 온-프레미스 인프라가 이미 구성된 상태여야 합니다.

솔루션을 배포하려면 다음 단계를 수행합니다.

1. 아래 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure Portal에서 링크가 열릴 때까지 기다린 후 다음 단계를 수행합니다. 
   * **리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-hybrid-vpn-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 하위 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구매** 단추를 클릭합니다.
3. 배포가 완료될 때가지 기다립니다.



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
<!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "온-프레미스 인프라와 Azure 인프라를 포괄하는 하이브리드 네트워크"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Azure Portal의 감사 로그"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "VPN 네트워크 트래픽 모니터링을 위한 성능 카운터"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "VPN 네트워크 성능 그래프 예"