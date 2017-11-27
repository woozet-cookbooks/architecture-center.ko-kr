---
title: "Azure에 SAP NetWeaver 및 SAP HANA 배포"
description: "Azure의 고가용성 환경에서 SAP HANA를 실행하는 방법에 대한 검증된 사례입니다."
author: njray
ms.date: 06/29/2017
ms.openlocfilehash: 27a97103c0c6f305cb8e830d670c8d0ba7e22aa5
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="deploy-sap-netweaver-and-sap-hana-on-azure"></a>Azure에 SAP NetWeaver 및 SAP HANA 배포

이 참조 아키텍처는 Azure의 고가용성 환경에서 SAP HANA를 실행하는 방법에 대한 검증된 사례 집합을 보여줍니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

![0][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

> [!NOTE]
> 이 참조 아키텍처를 배포하려면 적절한 SAP 제품 라이선스 및 기타 Microsoft 이외의 기술이 필요합니다. Microsoft와 SAP 간의 파트너 관계에 대한 정보는 [Azure 기반의 SAP HANA][sap-hana-on-azure]를 참조하세요.

## <a name="architecture"></a>아키텍처

이 아키텍처는 다음 구성 요소로 구성됩니다.

- **가상 네트워크(VNet)**. VNet은 Azure에서 논리적으로 격리된 네트워크의 표현입니다. 이 참조 아키텍처의 모든 VM은 동일한 VNet에 배포됩니다. VNet은 서브넷으로 더욱 세분화됩니다. 응용 프로그램(SAP NetWeaver), 데이터베이스(SAP HANA), 관리(jumpbox) 및 Active Directory를 포함하여 계층마다 별도의 서브넷을 만듭니다.

- **VM(가상 머신)**. 이 아키텍처의 VM은 여러 고유 계층으로 그룹화됩니다.

    - **SAP NetWeaver**. SAP ASCS, SAP Web Dispatcher 및 SAP 응용 프로그램 서버를 포함하고 있습니다. 
    
    - **SAP HANA**. 이 참조 아키텍처는 데이터베이스 계층에 SAP HANA를 사용하며, 단일 [GS5][vm-sizes-mem] 인스턴스에서 실행됩니다. SAP HANA는 GS5의 프로덕션 OLAP 워크로드 또는 [Azure Large Instances 기반의 SAP HANA][azure-large-instances]에 대해 인증을 받았습니다. 이 참조 아키텍처는 G 시리즈 및 M 시리즈의 Azure 가상 머신용입니다. Azure Large Instances 기반의 SAP HANA에 대한 자세한 내용은 [Azure의 SAP HANA(대규모 인스턴스) 개요 및 아키텍처][azure-large-instances]를 참조하세요.
   
    - **Jumpbox**. 요새 호스트라고도 합니다. 관리자가 다른 VM에 연결할 때 사용하는 네트워크의 보안 VM입니다. 
     
    - **Windows Server AD(Active Directory) 도메인 컨트롤러.** 도메인 컨트롤러는 Windows Server 장애 조치(failover) 클러스터를 구성하는 데 사용됩니다.
 
- **가용성 집합**. SAP Web Dispatcher, SAP 응용 프로그램 서버 및 SAP ACSC 역할을 위한 VM을 별도의 가용성 집합에 배치하고, 역할마다 VM을 적어도 두 대 이상 프로비전합니다. 이렇게 하면 VM이 더 높은 Service Level Agreement(서비스 수준 약정)를 충족할 수 있습니다.
    
- **NIC.** SAP NetWeaver 및 SAP HANA를 실행하는 VM은 두 개의 네트워크 인터페이스(NIC)가 필요합니다. 각 NIC는 서로 다른 서브넷에 할당되어 다른 종류의 트래픽을 분리합니다. 자세한 내용은 아래의 [권장 사항을](#recommendations) 참조하세요.

- **Windows Server 장애 조치(failover) 클러스터**. SAP ACSC를 실행하는 VM은 고가용성을 위해 장애 조치(failover) 클러스터로 구성됩니다. 장애 조치(failover) 클러스터를 지원하기 위해 SIOS DataKeeper Cluster Edition은 클러스터 노드가 소유한 독립 디스크를 복제하여 CSV(클러스터 공유 볼륨) 기능을 수행합니다. 자세한 내용은 [Microsoft 플랫폼에서 SAP 응용 프로그램 실행][running-sap]을 참조하세요.
    
- **부하 분산 장치.** 두 가지 [Azure Load Balancer][azure-lb] 인스턴스가 사용됩니다. 다이어그램의 왼쪽에 표시된 첫 번째는 SAP Web Dispatcher VM에 트래픽을 분산합니다. 이 구성은 [SAP Web Dispatcher의 고가용성][sap-dispatcher-ha]에 설명된 병렬 웹 디스패처 옵션을 구현합니다. 오른쪽에 표시된 두 번째 부하 분산 장치는 들어오는 연결을 활성/정상 노드로 보내서 Windows Server 장애 조치(failover) 클러스터에서 장애 조치(failover)를 지원합니다.

- **VPN Gateway.** VPN Gateway는 온-프레미스 네트워크를 Azure VNet으로 확장합니다. 공용 인터넷을 통과하지 않는 전용 사설 연결을 사용하는 ExpressRoute를 사용할 수도 있습니다. 예제 솔루션에서는 게이트웨이를 배포하지 않습니다. 자세한 내용은 [온-프레미스 네트워크를 Azure에 연결][hybrid-networking]을 참조하세요.

## <a name="recommendations"></a>권장 사항

개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다. 여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요.

### <a name="load-balancers"></a>부하 분산 장치

[SAP Web Dispatcher][sap-dispatcher]는 HTTP(S) 트래픽을 이중 스택 서버(ABAP 및 Java)로 분산합니다. SAP는 수년째 단일 스택 응용 프로그램 서버를 고집하고 있으며, 최근에는 이중 스택 배포 모델에서 실행되는 응용 프로그램이 거의 없습니다. 아키텍처 다이어그램에 표시된 Azure 부하 분산 장치는 SAP Web Dispatcher를 위한 고가용성 클러스터를 구현합니다.

트래픽을 응용 프로그램 서버로 부하 분산하는 작업은 SAP 내에서 처리됩니다. DIAG 및 RFC(원격 함수 호출)를 통해 SAP 서버에 연결하는 SAPGUI 클라이언트에서 오는 트래픽의 경우 SCS 메시지 서버가 SAP 앱 서버 [로그온 그룹][logon-groups]을 만들어서 부하를 분산합니다. 

SMLG는 SAP Central Services의 로그온 부하 분산 기능을 관리하는 데 사용되는 SAP ABAP 트랜잭션입니다. 로그온 그룹의 백 엔드 풀에는 두 대 이상의 ABAP 응용 프로그램 서버가 있습니다. ASCS 클러스터 서비스에 액세스하는 클라이언트는 프런트 엔드 IP 주소를 통해 Azure 부하 분산 장치에 연결합니다. ASCS 클러스터 가상 네트워크 이름에는 IP 주소도 포함됩니다. 필요에 따라 이 주소를 Azure 부하 분산 장치의 추가 IP 주소와 연결하여 클러스터를 원격으로 관리할 수 있습니다.  

### <a name="nics"></a>NIC

SAP 환경 관리 기능을 사용하려면 여러 NIC의 서버 트래픽을 분리해야 합니다. 예를 들어 비즈니스 데이터를 관리 트래픽 및 백업 트래픽과 분리해야 합니다. 여러 NIC를 여러 서브넷에 할당하면 데이터를 분리할 수 있습니다. 자세한 내용은 [SAP NetWeaver 및 SAP HANA를 위한 고가용성 구축][sap-ha](PDF)의 "네트워크" 섹션을 참조하세요.

관리 NIC는 관리 서브넷에 할당하고 데이터 통신 NIC는 별도의 서브넷에 할당합니다. 구성에 대한 자세한 내용은 [여러 NIC가 있는 Windows 가상 머신 만들기 및 관리][multiple-vm-nics]를 참조하세요.

### <a name="azure-storage"></a>Azure Storage

읽기/쓰기 대기 시간을 일관적으로 유지할 수 있도록 모든 데이터베이스 서버 VM에 Azure Premium Storage를 사용하는 것이 좋습니다. (A)SCS 가상 머신을 포함한 SAP 응용 프로그램 서버의 경우 응용 프로그램 실행이 메모리에서 발생하고 디스크를 로깅에만 사용하기 때문에 Azure Standard Storage를 사용해도 됩니다.

안정성을 최대로 높이려면 [Azure Managed Disks][managed-disks]를 사용하는 것이 좋습니다. 관리 디스크는 가용성 집합 내의 VM용 디스크를 격리하여 단일 실패 지점을 방지합니다.

> [!NOTE]
> 현재 이 참조 아키텍처에 대한 Resource Manager 템플릿은 관리 디스크를 사용하지 않습니다. 관리 디스크를 사용하도록 템플릿을 업데이트할 예정입니다.

높은 IOPS 및 대역폭 처리량을 얻으려면 저장소 볼륨 성능 최적화의 일반 사례를 Azure 저장소 레이아웃에 적용합니다. 예를 들어 여러 디스크를 스트라이프하여 하나의 더 큰 디스크 볼륨을 만들면 IO 성능이 향상됩니다. 자주 변경되지 않는 저장소 콘텐츠에 읽기 캐시를 사용하면 데이터 검색 속도가 향상됩니다. 성능 요구 사항에 대한 자세한 내용은 [SAP Note 1943937 - 하드웨어 구성 검사 도구][sap-1943937]를 참조하세요.

백업 데이터 저장소의 경우 Azure Blob Storage의 [쿨 저장소 계층][cool-blob-storage]을 사용하는 것이 좋습니다. 쿨 저장소 계층은 액세스 빈도가 낮고 수명이 긴 데이터를 비용 효율적으로 저장하는 방법입니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

SAP 응용 프로그램 레이어에서 Azure는 확장 가능한 다양한 가상 머신 크기를 제공합니다. 전체 목록은 [SAP Note 1928533 - Azure의 SAP 응용 프로그램: 지원 제품 및 Azure VM 유형][sap-1928533]을 참조하세요. 가용성 집합에 더 많은 VM을 추가하여 규모 확장합니다.

OLTP 및 OLAP SAP 응용 프로그램을 모두 사용하는 Azure 가상 머신의 SAP HANA를 위한 SAP 인증 가상 머신의 크기는 단일 VM 인스턴스를 사용하는 GS5입니다. 더 큰 워크로드를 원하는 고객을 위해 Microsoft에서는 Microsoft Azure 인증 데이터 센터에 공동 배치된 물리적 서버의 SAP HANA에 [Azure Large Instances][azure-large-instances]를 제공합니다. 이 서비스는 현재 단일 인스턴스에 최대 4TB의 메모리 용량을 제공합니다. 다중 노드 구성도 가능하며 총 메모리 용량은 최대 32TB입니다.

## <a name="availability-considerations"></a>가용성 고려 사항

중앙의 데이터베이스에 SAP 응용 프로그램이 분산 설치되는 이 시스템에서는 고가용성을 얻기 위해 기본 설치가 복제됩니다. 아키텍처의 레이어마다 고가용성 디자인이 달라집니다.

- **Web Dispatcher.** SAP 응용 프로그램 트래픽을 처리하는 SAP Web Dispatcher 인스턴스를 이중화하여 고가용성을 얻습니다. SAP 설명서의 [SAP Web Dispatcher][swd]를 참조하세요.

- **ASCS.** Azure Windows 가상 머신에서 ASCS의 가용성을 높이기 위해 SIOS DataKeeper와 함께 Windows Sever 장애 조치(failover) 클러스터를 사용하여 클러스터 공유 볼륨을 구현합니다. 구현에 대한 자세한 내용은 [Azure에서 SAP ASCS 클러스터링][clustering]을 참조하세요.

- **응용 프로그램 서버.** 응용 프로그램 서버 풀 내에서 트래픽을 분산하여 고가용성을 얻습니다.

- **데이터베이스 계층.** 이 참조 아키텍처는 단일 SAP HANA 데이터베이스 인스턴스를 배포합니다. 고가용성을 얻으려면 둘 이상의 인스턴스를 배포하고 HSR(HANA System Replication)을 사용하여 수동 장애 조치(failover)를 구현합니다. 자동 장애 조치(failover)를 사용하려면 특정 Linux 배포를 위한 HA 확장이 필요합니다.

### <a name="disaster-recovery-considerations"></a>재해 복구 고려 사항

각 계층은 서로 다른 전략을 사용하여 재해 복구(DR) 보호를 제공합니다.

- **응용 프로그램 서버.** SAP 응용 프로그램 서버는 비즈니스 데이터를 포함하지 않습니다. Azure에서는 다른 지역에 SAP 응용 프로그램 서버를 만드는 단순한 DR 전략을 사용합니다. 주 응용 프로그램 서버의 구성이 업데이트되거나 커널이 업데이트되는 즉시 동일한 변경 내용이 DR 지역의 VM으로 복사되어야 합니다. 예를 들어 커널 실행 파일이 DR VM에 복사됩니다.

- **SAP Central Services.** 이 SAP 응용 프로그램 스택의 구성 요소 역시 비즈니스 데이터를 유지하지 않습니다. DR 지역에 SCS 역할을 실행할 VM을 빌드할 수 있습니다. 주 SCS 노드에서 동기화할 유일한 콘텐츠는 **/sapmnt** 공유 콘텐츠입니다. 또한 주 SCS 서버에서 구성 변경 또는 커널 업데이트가 발생하면 DR SCS에서도 반복되어야 합니다. 두 서버를 동기화하려면 간단하게 주기적 예약 복사 작업을 사용하여 **/sapmnt**를 DR 쪽에 복사하면 됩니다. 장애 조치(failover) 프로세스를 빌드, 복사 및 테스트하는 방법에 대한 자세한 내용은 [Hyper-V 및 Microsoft Azure 기반 재해 복구 솔루션 빌드][sap-netweaver-dr]를 다운로드하여 "4.3. SAP SPOF 레이어(ASCS)"를 참조하세요.

- **데이터베이스 계층.** HSR 또는 Storage Replication처럼 HANA에서 지원하는 복제 솔루션을 사용합니다. 

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

SAP HANA는 기본 Azure 인프라를 사용하는 백업 기능을 제공합니다. Azure 가상 머신에서 실행되는 SAP HANA 데이터베이스를 백업하려면 SAP HANA 스냅숏과 Azure 저장소 스냅숏을 모두 사용하여 백업 파일의 일관성을 유지해야 합니다. 자세한 내용은 [Azure Virtual Machines의 SAP HANA Backup 가이드][hana-backup] 및 [Azure Backup 서비스 FAQ][backup-faq]를 참조하세요.

Azure는 전체 인프라를 [모니터링 및 진단][monitoring]하는 여러 기능을 제공합니다. 또한 Azure 가상 머신(Linux 또는 Windows)의 향상된 모니터링은 Azure OMS(Operations Management Suite)를 통해 처리됩니다.

SAP 인프라의 리소스 및 서비스 성능을 모니터링할 수 있는 SAP 기반 모니터링 기능을 제공하려면 Azure SAP 고급 모니터링 확장을 사용합니다. 이 확장은 SAP 응용 프로그램에 운영 체제 모니터링 및 DBA Cockpit 함수에 대한 Azure 모니터링 통계를 제공합니다. 

## <a name="security-considerations"></a>보안 고려 사항

SAP는 자체적인 UME(사용자 관리 엔진)를 사용하여 SAP 응용 프로그램의 역할 기반 액세스 및 권한 부여를 제어합니다. 자세한 내용은 [SAP HANA 보안 - 개요][sap-security]를 참조하세요. (액세스하려면 SAP Service Marketplace 계정이 필요합니다.)

인프라 보안을 위해 전송 중 데이터와 대기 중인 데이터가 보호됩니다. [Azure VMs(Virtual Machines)의 SAP NetWeaver - 계획 및 구현 가이드][netweaver-on-azure]의 “보안 고려 사항” 섹션에서 네트워크 보안을 해결하는 방법이 시작됩니다. 이 가이드에서는 응용 프로그램 통신을 허용하기 위해 방화벽에서 열어야 하는 네트워크 포트도 지정합니다. 

Windows 및 Linux IaaS 가상 머신 디스크를 암호화하려면 [Azure Disk Encryption][disk-encryption]을 사용합니다. Azure Disk Encryption은 Windows의 BitLocker 기능과 Linux의 DM-Crypt 기능을 사용하여 운영 체제 및 데이터 디스크를 위한 볼륨 암호화를 제공합니다. 이 솔루션은 Key Vault 구독에서 디스크 암호화 키 및 비밀을 제어하고 관리할 수 있는 Azure Key Vault와도 호환됩니다. 가상 머신 디스크의 데이터는 미사용 시 Azure 저장소에 암호화됩니다.

SAP HANA 미사용 데이터 암호화의 경우 SAP HANA 네이티브 암호화 기술을 사용하는 것이 좋습니다.

> [!NOTE]
> 동일한 서버에 있는 Azure 디스크 암호화에는 HANA 미사용 데이터 암호화를 사용하지 마세요.

[NSG(네트워크 보안 그룹)][nsg]를 사용하여 VNet의 다양한 서브넷 간 트래픽을 제한하는 방법을 고려해 보세요.

## <a name="deploy-the-solution"></a>솔루션 배포 

이 참조 아키텍처의 배포 스크립트는 [GitHub][github]에서 얻을 수 있습니다.


### <a name="prerequisites"></a>필수 조건

- SAP 소프트웨어 다운로드 센터에 액세스하여 설치를 완료해야 합니다.
 
- [Azure PowerShell][azure-ps] 최신 버전을 설치합니다. 

- 이 배포에는 코어 51개가 필요합니다. 배포하기 전에 구독의 VM 코어 할당량이 충분한지 확인하세요. 충분하지 않으면 Azure Portal을 사용하여 할당량을 늘려 달라는 지원 요청을 제출하세요.
 
- 이 배포에는 GS 시리즈 VM을 사용합니다. [여기][region-availability]서 해당 지역의 GS 시리즈 가용성을 확인하세요.

- 이 배포의 비용을 예상하는 방법은 [Azure 가격 계산기][azure-pricing]를 참조하세요. 
 
이 참조 아키텍처는 다음 VM을 배포합니다.

| 리소스 이름 | VM 크기 | 목적  |
|---------------|---------|----------|
| `ra-sapApps-scs-vm1` ... `ra-sapApps-scs-vmN` | DS11v2 | SAP Central Services |
| `ra-sapApps-vm1` ... `ra-sapApps-vmN` | DS11v2 | SAP NetWeaver 응용 프로그램 |
| `ra-sap-wdp-vm1` ... `ra-sap-wdp-vmN` | DS11v2 | SAP Web Dispatcher |
| `ra-sap-data-vm1` | GS5 | SAP HANA 데이터베이스 인스턴스 |
| `ra-sap-jumpbox-vm1` | DS1V2 | Jumpbox |

단일 SAP HANA 인스턴스가 배포됩니다. 응용 프로그램 VM의 경우 배포할 인스턴스 수가 템플릿 매개 변수에 지정됩니다.

### <a name="deploy-sap-infrastructure"></a>SAP 인프라 배포

이 아키텍처를 단계적으로 또는 한 번에 배포할 수 있습니다. 처음에는 각 배포 단계의 역할을 살펴볼 수 있도록 증분 방식 배포를 사용하는 것이 좋습니다. 다음 *mode* 매개 변수 중 하나를 사용하여 증분 지정

| Mode           | 기능                                                                                                            |
|----------------|-----------------------------------------------------|
| infrastructure | Azure에 네트워크 인프라를 배포합니다.        |
| workload       | 네트워크에 SAP 서버를 배포합니다.             |
| 모두            | 모든 이전 배포를 배포합니다.              |

솔루션을 배포하려면 다음 단계를 수행합니다.

1. [GitHub 리포지토리][github]를 로컬 컴퓨터에 다운로드 또는 복제합니다.

2. PowerShell 창을 열고 `/sap/sap-hana/` 폴더로 이동합니다.

3. 다음 PowerShell cmdlet을 실행합니다. `<subscription id>`에 Azure 구독 ID를 입력합니다. `<location>`에서 Azure 지역(예: `eastus` 또는 `westus`)을 지정합니다. `<mode>`에서 위에 나열된 모드 중 하나를 지정합니다.

    ```powershell
     .\Deploy-ReferenceArchitecture -SubscriptionId <subscription id> -Location <location> -ResourceGroupName <resource group> <mode>
    ```

4.  로그온하라는 메시지가 표시되면 Azure 계정에 로그온합니다. 

선택한 모드에 따라 배포 스크립트가 완료되는 데 여러 시간이 걸릴 수 있습니다.

> [!WARNING]
> 매개 변수 파일은 다양한 위치에 하드 코드된 암호(`AweS0me@PW`)를 포함하고 있습니다. 배포하기 전에 이러한 값을 변경하세요.
 
### <a name="configure-sap-applications-and-database"></a>SAP 응용 프로그램 및 데이터베이스 구성

SAP 인프라를 배포한 후에는 다음과 같이 가상 머신에 SAP 응용 프로그램 및 HANA 데이터베이스를 설치하고 구성합니다.

> [!NOTE]
> SAP 설치 지침을 보려면 SAP 지원 포털 사용자 이름 및 암호를 사용하여 [SAP 설치 가이드][sap-guide]를 다운로드해야 합니다.

1. jumpbox(`ra-sap-jumpbox-vm1`)에 로그인합니다. jumpbox를 사용하여 다른 VM에 로그인할 것입니다. 

2.  `ra-sap-wdp-vm1`부터 `ra-sap-wdp-vmN`까지 각 VM에 로그인하고, [Web Dispatcher 설치][sap-dispatcher-install] wiki에 설명된 단계에 따라 SAP Web Dispatcher 인스턴스를 설치 및 구성합니다.

3.  `ra-sap-data-vm1` VM에 로그인합니다. [SAP HANA 서버 설치 및 업데이트 가이드][hana-guide]를 사용하여 SAP Hana Database 인스턴스를 설치 및 구성합니다.

4. `ra-sapApps-scs-vm1`부터 `ra-sapApps-scs-vmN`까지 각 VM에 로그인하고, [SAP 설치 가이드][sap-guide]를 사용하여 SCS(SAP Central Services)를 설치 및 구성합니다.

5.  `ra-sapApps-vm1`부터 `ra-sapApps-vmN`까지 각 VM에 로그인하고, [SAP 설치 가이드][sap-guide]를 사용하여 SAP NetWeaver 응용 프로그램을 설치 및 구성합니다.

**_이 참조 아키텍처에 기여하신 분들_** &mdash; Rick Rainey, Ross Sponholtz, Ben Trinh

[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[azure-lb]: /azure/load-balancer/load-balancer-overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[azure-ps]: /powershell/azure/overview
[backup-faq]: /azure/backup/backup-azure-backup-faq
[clustering]: https://blogs.msdn.microsoft.com/saponsqlserver/2015/05/20/clustering-sap-ascs-instance-using-windows-server-failover-cluster-on-microsoft-azure-with-sios-datakeeper-and-azure-internal-load-balancer/
[cool-blob-storage]: /azure/storage/storage-blob-storage-tiers
[disk-encryption]: /azure/security/azure-security-disk-encryption
[github]: https://github.com/mspnp/reference-architectures/tree/master/sap/sap-hana
[hana-backup]: /azure/virtual-machines/workloads/sap/sap-hana-backup-guide
[hana-guide]: https://help.sap.com/viewer/2c1988d620e04368aa4103bf26f17727/2.0.01/en-US/7eb0167eb35e4e2885415205b8383584.html
[hybrid-networking]: ../hybrid-networking/index.md
[logon-groups]: https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring]: /azure/architecture/best-practices/monitoring
[multiple-vm-nics]: /azure/virtual-machines/windows/multiple-nics
[netweaver-on-azure]: /azure/virtual-machines/workloads/sap/planning-guide
[nsg]: /azure/virtual-network/virtual-networks-nsg
[region-availability]: https://azure.microsoft.com/regions/services/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[sap-1943937]: https://launchpad.support.sap.com/#/notes/1943937
[sap-1928533]: https://launchpad.support.sap.com/#/notes/1928533
[sap-dispatcher]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm
[sap-dispatcher-ha]: https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/9a9a6b48c673e8e10000000a42189b/frameset.htm
[sap-dispatcher-install]: https://wiki.scn.sap.com/wiki/display/SI/Web+Dispatcher+Installation
[sap-guide]: https://service.sap.com/instguides
[sap-ha]: https://support.sap.com/content/dam/SAAP/SAP_Activate/AGS_70.pdf
[sap-hana-on-azure]: https://azure.microsoft.com/services/virtual-machines/sap-hana/
[sap-netweaver-dr]: http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[visio-download]: https://archcenter.azureedge.net/cdn/SAP-HANA-architecture.vsdx
[vm-sizes-mem]: /azure/virtual-machines/windows/sizes-memory
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[0]: ./images/sap-hana.png "Microsoft Azure를 사용하는 SAP HANA 아키텍처"
