---
title: Connect an on-premises network to Azure using VPN
description: >-
  How to implement a secure site-to-site network architecture that spans an
  Azure virtual network and an on-premises network connected using a VPN.

author: RohitSharma-pnp

pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute

ms.service: guidance

ms.topic: article

ms.date: 11/28/2016
ms.author: pnp
pnp.series.prev: ./index
cardTitle: VPN
---
# VPN 게이트웨이를 사용하여 온-프레미스 네트워크를 Azure에 연결

이 참조 아키텍처는 사이트 간 가상 사설 네트워크(VPN)를 통해 온-프레미스 네트워크를 Azure로 확장하는 방법을 보여줍니다. 트래픽은 IPSec VPN 터널을 통해 온-프레미스 네트워크와 Azure 가상 네트워크(VNet) 사이를 흐릅니다. [**이 솔루션 배포하기**.](#deploy-the-solution)

![[0]][0]

## 아키텍처

이 아키텍처는 다음과 같은 요소들로 구성되어 있습니다.

* **온-프레미스 네트워크**. 조직 내에서 실행되는 사설 로컬 영역 네트워크.

* **VPN 어플라이언스**. 외부에서 온-프레미스 네트워크로 연결할 수 있도록 지원하는 장치 또는 서비스. VPN 어플라이언스는 하드웨어 장치일 수도 있고, Windows Server 2012의 라우팅 및 원격 액세스 서비스(RRAS)와 같은 소프트웨어일 수도 있습니다. 지원되는 VPN 어플라이언스 목록과 Azure VPN 게이트웨이로의 연결 구성에 관한 정보는 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치]의 선택된 장치에 대한 지침을 참조하시기 바랍니다.

* **가상 네트워크(VNet)**. 클라우드 응용 프로그램과 Azure VPN 게이트웨이를 위한 구성요소는 동일한 [VNet][azure-virtual-network]에 존재합니다.

* **Azure VPN 게이트웨이**. [VPN gateway][azure-vpn-gateway] 서비스를 활용하면 VPN 어플라이언스를 통해 VNet을 온-프레미스 네트워크에 연결할 수 있습니다. 자세한 내용은 [온-프레미스 네트워크를 Microsoft Azure 가상 네트워크에 연결][connect-to-an-Azure-vnet]을 참조하시기 바랍니다. VPN 게이트웨이에는 다음과 같은 요소들이 포함되어 있습니다.
  
  * **가상 네트워크 게이트웨이**. VNet을 위한 가상 VPN 어플라이언스를 제공하는 리소스. 이 리소스는 온-프레미스 네트워크로부터 VNet으로 트래픽을 라우팅하는 역할을 합니다.
  * **로컬 네트워크 게이트웨이**. 온-프레미스 VPN 어플라이언스의 추상화. 클라우드 응용 프로그램에서 온-프레미스 네트워크로의 네트워크 트래픽은 이 게이트웨이를 통해 라우팅됩니다.
  * **연결**. 연결은 연결 유형(IPSec)과 트래픽을 암호화기 위해 온-프레미스 VPN 어플라이언스와 공유하는 키를 지정하는 속성을 갖습니다.
  * **게이트웨이 서브넷**. 가상 네트워크 게이트웨이는 자체 서브넷에 배치됩니다. 이 서브넷에는 아래 권장사항 섹션에 설명된 여러 요구사항이 적용됩니다.

* **클라우드 응용 프로그램**. The application hosted in Azure. It might include multiple tiers, with multiple subnets connected through Azure load balancers. For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].

* **내부 부하 분산 장치**. VPN 게이트웨이로부터의 네트워크 트래픽은 내부 부하 분산 장치를 통해 클라우드 응용 프로그램으로 라우팅됩니다. 이 부하 분산 장치는 응용 프로그램의 프런트엔드 서브넷에 위치합니다.

You can download a [Visio file](https://aka.ms/arch-diagrams) of this architecture.

> [!참고]
> Azure는 [Azure Resource Manager][resource-manager-overview]와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 참조 아키텍처에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다.
> 

## 권장사항

다음 권장사항은 대부분의 시나리오에 적용됩니다. 다른 구체적인 요구사항이 없다면 가급적 권장사항을 따르시기 바랍니다.

### VVNet 및 게이트웨이 서브넷

필요한 모든 리소스를 위한 충분한 주소 공간을 가진 Azure Vnet을 생성합니다. 향후 VM을 추가할 가능성이 높다면 충분한 주소 공간을 정의해야 합니다. VNet의 주소 공간은 온-프레미스 네트워크와 겹치지 않아야 합니다. 예를 들어, 위의 다이어그램에서는 VNet에 대해 주소 공간 10.20.0.0/16을 사용합니다.

*GatewaySubnet*이라는 서브넷을 만들고 주소 범위는 /27로 설정합니다. 이 서브넷은 가상 네트워크 게이트웨이에 의해 요구되는 것으로, 이 서브넷에 32개의 주소를 할당하면 향후 게이트웨이 크기 한계에 도달하는 것을 방지하는 데 도움이 됩니다. 또한 이 서브넷을 주소 공간의 중간에 두지 말아야 합니다. 게이트웨이 서브넷에 대한 주소 공간은 VNet 주소 공간의 상단에 두는 것이 좋습니다. 위의 다이어그램에 제시된 예에서는 10.20.255.224/27을 사용합니다.  [CIDR]을 계산하는 간단한 절차는 다음과 같습니다.

1. VNet 주소 공간의 가변 비트를 게이트웨이 서브넷이 사용하는 비트까지는 1로 설정하고, 나머지 비트는 0으로 설정합니다. 
2. 결과로 산출된 비트를 10진법으로 변환하고 게이트웨이 서브넷 크기로 설정된 접두사 길이를 가진 주소 공간으로 표현합니다.

예를 들어, IP 주소 범위가 10.20.0.0/16인 Vnet에 위의 1단계를 적용하면 10.20.0b11111111.0b11100000이 됩니다. 이를 10진법으로 변환하여 주소 공간으로 표현하면 10.20.255.224/27이 됩니다. 

> [!경고]
> 게이트웨이 서브넷에는 VM을 배포해서는 안 됩니다. 또한 이 서브넷에는 네트워크 보안 그룹을 할당해서는 안 됩니다. 게이트웨이 작동이 중단될 수 있습니다.
> 
> 

### 가상 네트워크 게이트웨이

가상 네트워크 게이트웨이에 대한 공용 IP 주소를 할당합니다. 

게이트웨이 서브넷에 가상 네트워크 게이트웨이를 생성하고 새로 할당된 공용 IP 주소를 할당합니다. 요구사항에 가장 잘 부합하고 VPN 어플라이언스에 의해 활성화되는 게이트웨이 유형을 사용합니다. 

- 요청이 주소 접두사 등의 정책 조건을 바탕으로 라우팅되는 방식을 면밀히 제어할 필요가 있다면 [정책 기반 게이트웨이][policy-based-routing]를 생성합니다. 정책 기반 게이트웨이는 정적 라우팅을 사용하고 사이트 간 연결에만 사용할 수 있습니다.

- 라우팅 및 원격 액세스 서비스(RRAS)를 사용하는 온-프레미스 네트워크에 연결하거나 멀티 사이트나 지역 간 연결을 지원하거나 (여러 VNet을 트래버스하는 라우트를 포함한) VNet 간 연결을 구현하는 경우에는 [루트 기반 게이트웨이][route-based-routing]를 생성합니다. 루트 기반 게이트웨이는 동적 라우팅을 사용하여 네트워크 간 트래픽을 전달합니다. 대안 루트를 시도할 수 있어 정적 루트에 비해 네트워크 경로 장애를 더 잘 견딜 수 있는 루트 기반 게이트웨이는 네트워크 주소 변경 시 루트를 수동으로 업데이트할 필요가 없어 관리 오버헤드를 줄일 수도 있습니다.

지원되는 VPN 어플라이언스 목록은 [사이트 간 VPN 게이트웨이 연결을 위한 VPN 장치][vpn-appliances]를 참조하시기 바랍니다.

> [!참고]
> 게이트웨이를 생성한 후에는 삭제하고 다시 생성하지 않는 한 게이트웨이 유형을 변경할 수 없습니다.
> 
> 

따라서 처리율 요구사항에 가장 잘 부합하는 Azure 게이트웨이 SKU를 선택해야 합니다. 다음 표에는 Azure VPN 게이트웨이의 세 가지 SKU가 정리되어 있습니다. 

| SKU | VPN 처리율 | 최대 IPSec 터널 수 |
| --- | --- | --- |
| 기본(Basic) |100 Mbps |10 |
| 표준(Standard) |100 Mbps |10 |
| 고성능(High Performance) |200 Mbps |30 |

> [!참고]
> 기본(Basic) SKU는 Azure ExpressRoute와 호환되지 않습니다. 게이트웨이를 생성한 후 [SKU를 변경할 수 있습니다][changing-SKUs].
> 
> 

게이트웨이가 프로비전되고 사용 가능해질 때까지 소요되는 시간에 따라 요금이 청구됩니다. [VPN 게이트웨이 가격 책정][azure-gateway-charges]을 참조하시기 바랍니다.

요청을 곧바로 응용 프로그램 VM으로 전달하는 대신 게이트웨이로부터 들어오는 응용 프로그램 트래픽을 내부 부하 분산 장치로 보내는 게이트웨이 서브넷 라우팅 규칙을 만듭니다.

### 온-프레미스 네트워크 연결

로컬 네트워크 게이트웨이를 생성합니다. 온-프레미스 VPN 어플라이언스의 공용 IP 주소와 온-프레미스 네트워크의 주소 공간을 지정합니다. 온-프레미스 VPN 어플라이언스에는 Azure VPN 게이트웨이 내 로컬 네트워크 게이트웨이가 액세스할 수 있는 공용 IP 주소가 반드시 있어야 합니다. VPN 장치는 네트워크 주소 변환(NAT) 장치 뒤에 위치할 수 없습니다.

가상 네트워크 게이트웨이와 로컬 네트워크 게이트웨이를 위한 사이트 간 연결을 생성합니다. 사이트 간(IPSec) 연결 유형을 선택하고 공유 키를 지정합니다. Azure VPN 게이트웨이를 이용한 사이트 간 암호화는 IPSec 프로토콜을 기반으로 사전 공유 인증키를 사용하여 수행됩니다. Azure VPN 게이트웨이를 생성할 때 키를 지정하는데, 반드시 동일한 키로 온-프레미스에서 실행되는 VPN 어플라이언스를 구성해야 합니다. 다른 인증 방법은 현재로서는 지원되지 않습니다.

온-프레미스 라우팅 인프라는 Azure Vnet에 있는 주소로 향하는 요청을 VPN 장치로 전달하도록 구성되어야 합니다.

온-프레미스 네트워크에서 클라우드 응용 프로그램이 요구하는 모든 포트를 엽니다.

연결을 테스트하여 다음 사항을 확인합니다.

* 온-프레미스 VPN 어플라이언스는 Azure VPN 게이트웨이를 통해 클라우드 응용 프로그램으로 트래픽을 정확히 라우팅합니다.
* VNet은 온-프레미스 네트워크로 트래픽을 정확히 라우팅합니다.
* 금지된 트래픽은 양방향 모두에 대해 정확히 차단됩니다.

## 확장성 고려사항

Basic 또는 Standard VPN 게이트웨이 SKU에서 High Performance VPN SKU로 이동하여 제한적인 수직 확장을 할 수 있습니다.

VNet이 대량의 VPN 트래픽이 예상되는 경우에는 서로 다른 워크로드를 별도의 더 작은 VNet들로 분산시키고 각각에 대해 VPN 게이트웨이를 구성하는 방법을 고려해 보세요. 

VNet은 수평 또는 수직으로 분할할 수 있습니다. 수평으로 분할하려며 일부 VM 인스턴스를 각 계층에서 새로운 VNet의 서브넷으로 이동시킵니다. 그렇게 하면 각 VNet이 동일한 구조와 기능을 갖게 됩니다. 수직으로 분할하려면 각 계층의 기능을 다양한 논리적 영역(주문 처리, 청구서, 고객 계정 관리 등)으로 분할하도록 재설계합니다. 각 기능 영역은 자체 VNet에 배치됩니다. 

Vnet에 온-프레미스 Active Directory 도메인 컨트롤러를 복제하고 VNet에 DNS를 구현하면 온-프레미스에서 클라우드로 흐르는 보안 관련 및 관리 트래픽 중 일부를 줄일 수 있습니다. 자세한 내용은 [Active Directory Domain Services (AD DS)를 Azure로 확장][adds-extend-domain]을 참조하시기 바랍니다.

## 가용성 고려사항

Azure VPN 게이트웨이가 계속해서 온-프레미스 네트워크를 이용할 수 있게 하려면 온-프레미스 VPN 게이트웨이를 위한 장애조치 클러스터를 구현해야 합니다.

귀하의 조직이 다수의 온-프레미스 사이트를 운영한다면 하나 이상의 Azure VNet에 대한 [멀티사이트 연결][vpn-gateway-multi-site]을 생성해야 합니다. 이 접근 방식은 동적(라우트-기반) 라우팅이 필요하므로 온-프레미스 VPN 게이트웨이가 이 기능을 지원하는지 확인해야 합니다.

서비스 수준 계약(SLA)에 대한 자세한 내용은 [VPN 게이트웨이의 서비스 수준 계약(SLA)][sla-for-vpn-gateway]을 참조하시기 바랍니다.. 

## 관리 효율성 고려사항

온-프레미스 VPN 어플라이언스로부터 진단 정보를 모니터링합니다. 이 프로세스는 VPN 어플라이언스가 제공하는 기능에 따라 달라집니다. 예를 들어, Windows Server 2012에서 라우팅 및 원격 액세스 서비스(RRAS)를 사용한다면, [RRAS 로깅][rras-logging]이 이에 해당합니다.

[Azure VPN 게이트웨이 진단][gateway-diagnostic-logs]을 사용하여 연결 문제에 대한 정보를 수집합니다. 이 로그를 통해 연결 요청의 소스 및 목적지, 사용된 프로토콜, 연결 방법(또는 연결 실패 이유)와 같은 정보를 추적할 수 있습니다.

Azure 포털이 제공하는 감사 로그를 이용하여 Azure VPN 게이트웨이의 운영 로그를 모니터링합니다. 로컬 네트워크 게이트웨이, Azure 네트워크 게이트웨이 및 연결에 대해 각각 별도의 로그를 이용할 수 있습니다. 이 정보는 게이트웨이에 발생한 변경사항을 추적하는 데 사용되며, 작동 중이던 게이트웨이가 어떤 이유로 인해 정지되는 경우에 유용하게 사용될 수 있습니다.

![[2]][2]

연결을 모니터링하고 연결 실패 이벤트를 추적합니다. [Nagios][nagios]와 같은 모니터링 패키지를 이용하여 이 정보를 수집하고 보고할 수 있습니다..

## 보안 고려사항

각 VPN 게이트웨이에 대해 다른 공유 키를 생성합니다. 강력한 공유키를 사용하여 무차별 암호 대입 공격에 효과적으로 저항합니다.

> [!참고]
> 현재로서는 Azure Key Vault를 이용하여 Azure VPN 게이트웨이에 대한 키의 사전 공유는 지원되지 않습니다.
> 
> 

온-프레미스 VPN 어플라이언스는 [Azure VPN 게이트웨이와 호환되는][vpn-appliance-ipsec] 암호화 방법을 사용해야 합니다. 정책 기반 라우팅에 대해 Azure VPN 게이트웨이는 AES256, AES128 및 3DES 암호화 알고리즘을 지원합니다. 루트 기반 게이트웨이는 AES256과 3DES를 지원합니다.

온-프레미스 VPN 어플라이언스가 경계 네트워크와 인터넷 사이에 방화벽을 가진 경계 네트워크 상에 있다면 사이트 간 VPN 연결을 허용하는 [추가 방화벽 규칙][additional-firewall-rules]을 구성해야 할 수도 있습니다.

VNet의 응용 프로그램이 인터넷으로 데이터를 전송한다면 [강제 터널링을 구현해][forced-tunneling] 인터넷으로 가는 모든 트래픽을 온-프레미스 네트워크를 통해 라우팅하는 것을 고려해 보아야 합니다. 이러한 접근 방식을 통해 온-프레미스 인프라로부터 나가는 응용 프로그램의 요청에 대한 감사를 수행할 수 있습니다.

> [!참고]
> 강제 터널링은 (저장소 서비스 등) Azure 서비스로의 연결과 Windows 라이선스 관리자에 영향을 줄 수 있습니다.
> 
> 


## 문제 해결

일반적인 VPN 관련 오류 해결에 관한 정보는 [일반적인 VPN 관련 오류의 해결][troubleshooting-vpn-errors]을 참조하시기 바랍니다.

다음 권장사항은 온-프레미스 VPN 어플라이언스의 정상 작동 여부를 확인하는 데 유용합니다.

- **VPN 어플라이언스가 생성한 로그 파일 중 오류나 장애 정보가 있는지 확인합니다.**

    이를 통해 VPN 어플라이언스가 제대로 작동하는지 확인할 수 있습니다. 이 정보의 위치는 어플라이언스에 따라 다릅니다. 예를 들어, Windows Server 2012에서 RRAS를 사용한다면 다음과 같은 PowerShell 명령어를 사용하여 RRAS 서비스에 대한 오류 이벤트 정보를 표시할 수 있습니다.

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    각 항목의 *메시지* 속성은 해당 오류에 대한 설명을 제공합니다. 일반적인 예는 아래와 같습니다.

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

    다음 PowerShell 명령어를 사용하여 RRAS 서비스를 통한 연결 시도에 관한 이벤트 로그 정보도 얻을 수 있습니다. 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    연결 실패 시, 이 로그는 다음과 유사한 오류를 포함합니다.

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

- **VPN 게이트웨이 연결 및 라우팅을 확인합니다.**

    VPN 어플라이언스가 Azure VPN 게이트웨이를 통해 트래픽을 제대로 라우팅하지 못할 수도 있습니다. [PsPing][psping]과 같은 도구를 사용하여 VPN 게이트웨이의 연결과 라우팅을 확인할 수 있습니다. 예를 들어, 온-프레미스 컴퓨터로부터 VNet에 위치한 웹 서버로의 연결을 테스트하려면 다음과 같은 명령어 (`<<web-server-address>>` 부분을 해당 웹 서버 주소로 교체)를 실행합니다.

    ```
    PsPing -t <<web-server-address>>:80
    ```

    온-프레미스 컴퓨터가 트래픽을 웹 서버로 라우팅할 수 있다면, 다음과 유사한 출력 결과가 나와야 합니다.

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

    온-프레미스 컴퓨터가 지정된 목적지와 통신할 수 없는 경우에는 다음과 같은 메시지가 표시됩니다.

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

- **온-프레미스 방화벽이 VPN 트래픽을 통과시키는지 그리고 정확한 포트가 열리는지 확인합니다.**

- **온-프레미스 VPN 어플라이언스가 [Azure VPN 게이트웨이와 호환되는][vpn-appliance] 암호화 방법을 사용하는지 확인합니다.** 정책 기반 라우팅에 대해 Azure VPN 게이트웨이는 AES256, AES128 및 3DES 암호화 알고리즘을 지원합니다. 루트 기반 게이트웨이는 AES256과 3DES를 지원합니다.

다음 권장사항은 Azure VPN 게이트웨이에 문제가 없는지 확인하는 데 유용합니다.

- **잠재적 문제를 확인하기 위해 [Azure VPN 게이트웨이 진단 로그][gateway-diagnostic-logs]를 검사합니다.**

- **Azure VPN 게이트웨이와 온-프레미스 VPN 어플라이언스가 동일한 공유 인증키로 구성되었는지 확인합니다.**

    아래의 Azure CLI 명령어를 사용하여 Azure VPN 게이트웨이에 저장된 공유키를 확인할 수 있습니다.

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    온-프레미스 VPN 어플라이언스에 적합한 명령어를 사용하여 해당 어플라이언스를 위해 구성된 공유키를 표시합니다.

    Azure VPN 게이트웨이가 속한 *GatewaySubnet* 서브넷이 네트워크 보안 그룹과 연결되지는 않았는지 확인합니다.

    다음 Azure CLI 명령어를 사용하여 서브넷 세부정보를 확인할 수 있습니다.

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    *Network Security Group id*라는 이름의 데이터 필드가 있으면 안 됩니다. 아래의 예는 할당된 NSG (*VPN-Gateway-Group*)를 가진 *GatewaySubnet*의 인스턴스에 대한 결과를 보여줍니다. 이 경우 이 NSG에 대해 정의된 규칙이 있다면 게이트웨이가 제대로 작동하지 않을 수 있습니다.

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

- **Azure VNet의 가상 컴퓨터가 VNet 외부로부터 들어오는 트래픽을 허용하도록 구성되었는지 확인합니다.**

    이러한 가상 컴퓨터를 포함하는 서브넷과 관련한 NSG 규칙이 있는지 확인합니다. 다음 Azure CLI 명령어를 사용하여 모든 NSG 규칙을 확인할 수 있습니다.

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- **Azure VPN 게이트웨이가 연결되었는지 확인합니다.**

    다음 Azure PowerShell 명령어를 사용하여 Azure VPN 연결 상태를 확인할 수 있습니다. `<<connection-name>>` 매개변수는 가상 네트워크 게이트웨이와 로컬 게이트웨이를 연결하는 Azure VPN 연결의 이름입니다.

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    다음 코드 조각은 게이트웨이 연결 시(첫 번째 예)와 연결 종료 시(두 번째 예) 생성된 출력을 보여줍니다.

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
다음 권장사항은 Host VM 구성, 네트워크 대역폭 이용률 또는 응용 프로그램 성능 관련 문제가 있는지 확인하는 데 유용합니다.

- **서브넷 내 Azure VM에서 실행되는 게스트 운영 체제의 방화벽이 온-프레미스 IP 범위로부터의 허용된 트래픽을 허용하도록 정확히 구성되었는지 확인합니다.**

- **트래픽량이 Azure VPN 게이트웨이가 이용할 수 있는 대역폭의 한도에 가깝지 않은지 확인합니다.**

    이를 확인하는 방법은 온-프레미스에서 실행되는 VPN 어플라이언스에 따라 달라집니다. 예를 들어, Windows Server 2012에서 라우팅 및 원격 액세스 서비스(RRAS)를 사용하는 경우 Performance Monitor를 통해 VPN 연결을 통해 송수신되는 데이터량을 추적할 수 있습니다. *RAS Total* 개체를 사용하여*Bytes Received/Sec* 및 *Bytes Transmitted/Sec* 카운터를 선택합니다.

    ![[3]][3]

    산출된 결과를 VPN 게이트웨이가 사용할 수 있는 대역폭(기본(Basic)/표준(Standard) SKU는 100 Mbps, 고성능(High Performance) SKU는 200 Mpbs)과 비교합니다.

    ![[4]][4]

- **응용 프로그램 부하에 적합한 수량과 크기의 VM을 배포했는지 확인합니다.**

    Azure VNet에 느리게 실행되는 가상 컴퓨터가 있는지 확인합니다. 만약 있다면 과부하 상태이거나 가상 컴퓨터 수가 부하를 처리하기에 너무 적거나 부하 분산 장치가 제대로 구성되지 않았을 수 있습니다. 이를 확인하려면 [진단 정보를 수집하고 분석합니다][azure-vm-diagnostics]. 결과는 Azure 포털을 통해 확인할 수 있고, 성능 데이터에 대한 세부 정보를 제공하는 다양한 타사 도구를 이용할 수도 있습니다.

- **응용 프로그램이 클라우드 리소스를 효율적으로 이용하는지 확인합니다.**

    각각의 VM에서 실행되는 응용 프로그램 코드에 대한 계측을 수행하여 응용 프로그램이 리소스를 효율적으로 활용하고 있는지 확인합니다. [Application Insights][application-insights]와 같은 도구를 사용할 수 있습니다.

## 솔루션 배포


**사전 준비 사항** 적합한 네트워크 어플라이언스를 통해 이미 구성된 기존의 온-프레미스 인프라가 존재해야 합니다.

다음 절차를 통해 이 솔루션을 배포할 수 있습니다.

1. 아래 버튼을 마우스 오른쪽 단추로 클릭하여 "새 탭에서 링크 열기" 또는 "새 창에서 링크 열기"를 선택하십시오.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열리면 다음 절차를 따릅니다.  
   * **리소스 그룹 ** 이름이 매개변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택한 다음 텍스트 상자에 `ra-hybrid-vpn-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개변수 루트 Uri** 텍스트 상자는 편집하지 않습니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구입** 단추를 클릭합니다.
3. 배포가 완료될 때까지 기다립니다.



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
[create-azure-vnet]: /azure/virtual-network/virtual-networks-create-vnet-classic-cli
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[installing-ad]: /azure/active-directory/active-directory-install-replica-active-directory-domain-controller
[deploying-ad]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[creating-dns]: https://blogs.msdn.microsoft.com/mcsuksoldev/2014/03/04/creating-a-dns-server-in-azure-iaas/
[configuring-dns]: /azure/virtual-network/virtual-networks-manage-dns-in-vnet
[stormshield]: https://azure.microsoft.com/marketplace/partners/stormshield/stormshield-network-security-for-cloud/
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
<!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx
[vnet-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetwork.parameters.json
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ../_images/blueprints/hybrid-network-vpn.png "Structure of a hybrid network spanning the on-premises and cloud infrastructures"
[1]: ../_images/guidance-hybrid-network-vpn/partitioned-vpn.png "Partitioning a VNet to improve scalability"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Audit logs in the Azure portal"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Performance counters for monitoring VPN network traffic"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Example VPN network performance graph"
