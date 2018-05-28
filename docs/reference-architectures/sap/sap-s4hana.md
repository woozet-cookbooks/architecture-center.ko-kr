---
title: Azure의 Linux Virtual Machines용 SAP S/4HANA
description: Azure의 Linux 환경에서 고가용성을 통해 SAP S/4HANA를 실행하는 검증된 사례입니다.
author: lbrader
ms.date: 05/11/2018
ms.openlocfilehash: d24ef6f9e4eae460d0d0dcfff35568c812d09951
ms.sourcegitcommit: bb348bd3a8a4e27ef61e8eee74b54b07b65dbf98
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/21/2018
---
# <a name="sap-s4hana-for-linux-virtual-machines-on-azure"></a>Azure의 Linux Virtual Machines용 SAP S/4HANA

이 참조 아키텍처는 Azure에서 재해 복구를 지원하는 고가용성 환경에서 S/4HANA를 실행하는 일단의 검증된 사례를 보여 줍니다. 이 아키텍처는 조직의 요구 사항에 맞게 변경할 수 있는 특정 VM(가상 머신) 크기로 배포됩니다. 


![](./images/sap-s4hana.png)

## <a name="architecture"></a>아키텍처
 
> [!NOTE] 
> 이 참조 아키텍처에 따라 SAP 제품을 배포하려면 SAP 제품 및 Microsoft 이외의 기타 기술에 대한 적절한 라이선스가 필요합니다.

이 참조 아키텍처는 엔터프라이즈급 프로덕션 수준 시스템에 대해 설명합니다. 이 구성은 비즈니스 요구 사항에 맞게 단일 가상 머신으로 줄일 수 있습니다. 하지만 필요한 구성 요소는 다음과 같습니다.

**가상 네트워크**. [Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) 서비스는 Azure 리소스를 서로 안전하게 연결합니다. 이 아키텍처에서 가상 네트워크는 [hub-spoke 토폴로지](../hybrid-networking/hub-spoke.md)의 허브에 배포된 게이트웨이를 통해 온-프레미스 환경에 연결합니다. 스포크는 SAP 응용 프로그램 사용되는 가상 네트워크입니다.

**서브넷**. 가상 네트워크는 게이트웨이, 응용 프로그램, 데이터베이스 및 공유 서비스와 같은 각 계층에 대한 별도의 [서브넷](/azure/virtual-network/virtual-network-manage-subnet)으로 세분화됩니다. 

**가상 머신**. 이 아키텍처는 응용 프로그램 계층과 데이터베이스 계층에 대해 다음과 같이 그룹화되어 Linux를 실행하는 가상 머신을 사용합니다.

- **응용 프로그램 계층**. Fiori 프런트 엔드 서버 풀, SAP Web Dispatcher 풀, 응용 프로그램 서버 풀 및 SAP Central Services 클러스터가 포함됩니다. Azure Linux 가상 머신에 있는 Central Services의 고가용성을 위해 고가용성 NFS(네트워크 파일 시스템) 서비스가 필요합니다.
- **NFS 클러스터**. 이 아키텍처는 Linux 클러스터에서 실행되는 [NFS](/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs) 서버를 사용하여 SAP 시스템 간에 공유되는 데이터를 저장합니다. 이 중앙 집중식 클러스터는 여러 SAP 시스템에서 공유할 수 있습니다. NFS 서비스의 고가용성을 위해 선택한 Linux 배포판에 적절한 고가용성 확장이 사용됩니다.
- **SAP HANA**. 데이터베이스 계층은 클러스터에서 둘 이상의 Linux 가상 머신을 사용하여 고가용성을 달성합니다. HSR(HANA 시스템 복제)을 사용하여 주 및 보조 HANA 시스템 간에 콘텐츠를 복제합니다. Linux 클러스터링을 사용하여 시스템 장애를 감지하고 자동 장애 조치를 용이하게 합니다. 저장소 기반 또는 클라우드 기반 펜싱 메커니즘을 사용하여 장애가 발생한 시스템이 클러스터 브레인 분할 상황을 방지하기 위해 격리되거나 종료되도록 할 수 있습니다.
- **Jumpbox**. 요새 호스트라고도 합니다. 이는 관리자가 다른 가상 머신에 연결하는 데 사용하는 네트워크의 보안 가상 머신입니다. Windows 또는 Linux를 실행할 수 있습니다. HANA Cockpit 또는 HANA Studio 관리 도구를 사용할 때 웹을 편리하게 탐색하기 위해 Windows jumpbox를 사용합니다.

**부하 분산 장치**. 기본 제공 SAP 부하 분산 장치와 [Azure Load Balancer](/azure/load-balancer/load-balancer-overview)를 모두 사용하여 HA를 달성합니다. Azure Load Balancer 인스턴스를 사용하여 응용 프로그램 계층 서브넷의 가상 머신에 트래픽을 분산합니다.

**가용성 집합**. 모든 풀 및 클러스터(Web Dispatcher, SAP 응용 프로그램 서버, Central Services, NFS 및 HANA)용 가상 머신은 별도의 [가용성 집합](/azure/virtual-machines/windows/tutorial-availability-sets)으로 그룹화되고, 역할당 둘 이상의 가상 머신이 프로비전됩니다. 이렇게 하면 가상 머신에 더 높은 [SLA(서비스 수준 계약)](https://azure.microsoft.com/support/legal/sla/virtual-machines)를 적용할 수 있습니다. 

**NIC**. [NIC(네트워크 인터페이스 카드)](/azure/virtual-network/virtual-network-network-interface)를 사용하면 가상 네트워크의 모든 가상 머신에서 통신할 수 있습니다.

**네트워크 보안 그룹**. [NSG(네트워크 보안 그룹)](/azure/virtual-network/virtual-networks-nsg)는 가상 네트워크에서 들어오는 트래픽, 나가는 트래픽 및 서브넷 간 트래픽을 제한하기 위해 사용됩니다.

**게이트웨이**. 게이트웨이는 온-프레미스 네트워크를 Azure 가상 네트워크로 확장합니다. [ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/expressroute)는 공용 인터넷을 통해 통신하지 않는 사설 연결을 만드는 데 권장되는 Azure 서비스이지만 [사이트 간 연결](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal)도 사용할 수 있습니다. 

**Azure Storage**. [Azure Storage](/azure/storage/)는 가상 머신의 VHD(가상 하드 디스크)용 영구 저장소를 제공하기 위해 필요합니다. 

## <a name="recommendations"></a>권장 사항

이 아키텍처는 소규모 프로덕션 수준의 엔터프라이즈 배포를 설명합니다. 배포는 비즈니스 요구 사항에 따라 다릅니다. 여기서 추천하는 권장 사항을 단지 시작점으로 활용하세요.

### <a name="virtual-machines"></a>가상 머신

응용 프로그램 서버 풀 및 클러스터에서 요구 사항에 따라 가상 머신 수를 조정합니다. [Azure Virtual Machines 계획 및 구현 가이드](/azure/virtual-machines/workloads/sap/planning-guide)에는 가상 머신에서 SAP NetWeaver를 실행하는 방법에 대한 자세한 정보가 나와 있으며, 이 정보는 SAP S/4HANA에도 적용됩니다.

Azure 가상 머신 유형 및 처리량 메트릭에 대한 SAP 지원(SAPS)을 자세히 알아보려면 [SAP 노트 1928533](https://launchpad.support.sap.com/#/notes/1928533)을 참조하세요. 

### <a name="sap-web-dispatcher-pool"></a>SAP Web Dispatcher 풀

Web Dispatcher 구성 요소가 SAP 응용 프로그램 서버 간의 SAP 트래픽을 위한 부하 분산 장치로 사용됩니다. Web Dispatcher 구성 요소의 고가용성을 달성하기 위해 Azure Load Balancer를 사용하여 분산 장치 백 엔드 풀에서 사용할 수 있는 Web Dispatcher 간에 HTTP(S) 트래픽 분산에 대한 라운드 로빈 구성에서 병렬 Web Dispatcher 설정을 구현합니다. 

### <a name="fiori-front-end-server"></a>Fiori 프런트 엔드 서버

Fiori 프런트 엔드 서버는 [NetWeaver 게이트웨이](https://help.sap.com/doc/saphelp_gateway20sp12/2.0/en-US/76/08828d832e4aa78748e9f82204a864/content.htm?no_cache=true)를 사용합니다. 소규모 배포의 경우 Fiori 서버에 로드할 수 있습니다. 대규모 배포의 경우 별도의 NetWeaver 게이트웨이용 서버를 Fiori 프런트 엔드 서버 풀 앞에 배포할 수 있습니다.

### <a name="application-servers-pool"></a>응용 프로그램 서버 풀

ABAP 응용 프로그램 서버에 대한 로그온 그룹을 관리하기 위해 SMLG 트랜잭션이 사용됩니다. Central Services의 메시지 서버 내에서 부하 분산 기능을 사용하여 SAPGUI 및 RFC 트래픽용 SAP 응용 프로그램 서버 풀 간의 워크로드를 분산합니다. 고가용성 Central Services에 대한 응용 프로그램 서버는 클러스터 가상 네트워크 이름을 통해 연결됩니다. 이렇게 하면 로컬 장애 조치 후에 Central Services 연결에 대한 응용 프로그램 서버 프로필을 변경할 필요가 없습니다. 

### <a name="sap-central-services-cluster"></a>SAP Central Services 클러스터

고가용성이 요구 사항이 아닌 경우 Central Services를 단일 가상 머신에 배포할 수 있습니다. 그러나 단일 가상 머신이 SAP 환경에 대한 단일 SPOF(단일 실패 지점)가 됩니다. 고가용성 Central Services 배포의 경우 고가용성 NFS 클러스터 및 고가용성 Central Services 클러스터가 사용됩니다.

### <a name="nfs-cluster"></a>NFS 클러스터

DRBD(Distributed Replicated Block Device)가 NFS 클러스터의 노드 간 복제에 사용됩니다.

### <a name="availability-sets"></a>가용성 집합

가용성 집합은 서버를 서로 다른 물리적 인프라로 배포하고 그룹을 업데이트하여 서비스 가용성을 향상시킵니다. Azure 인프라 유지 관리로 인한 가동 중지 시간으로부터 보호하고 [SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines)를 충족하기 위해 동일한 역할을 수행하는 가상 머신을 가용성 집합에 배치합니다. 가용성 집합당 둘 이상의 가상 머신을 사용하는 것이 좋습니다.

집합의 모든 가상 머신은 동일한 역할을 수행해야 합니다. 동일한 가용성 집합에 역할이 다른 서버를 혼합하지 마세요. 예를 들어 ASCS 노드를 응용 프로그램 서버와 동일한 가용성 집합에 배치하면 안됩니다.

### <a name="nics"></a>NIC

기존의 온-프레미스 SAP 자산에서는 비즈니스 트래픽과 관리 트래픽을 분리하기 위해 컴퓨터당 여러 개의 NIC(네트워크 인터페이스 카드)를 구현합니다. Azure에서 가상 네트워크는 동일한 네트워크 구조를 통해 모든 트래픽을 보내는 소프트웨어 정의 네트워크입니다. 따라서 여러 개의 NIC를 사용할 필요가 없습니다. 그러나 조직에서 트래픽을 분리해야 하는 경우 VM당 여러 개의 NIC를 배포하고, 각 NIC를 서로 다른 서브넷에 연결한 다음, NSG를 사용하여 서로 다른 액세스 제어 정책을 적용할 수 있습니다.

### <a name="subnets-and-nsgs"></a>서브넷 및 NSG

이 아키텍처는 가상 네트워크 주소 공간을 서브넷으로 세분화합니다. 각 서브넷은 해당 서브넷에 대한 액세스 정책을 정의하는 NSG와 연결할 수 있습니다. 응용 프로그램 서버를 별도의 서브넷에 배치하면 개별 서버가 아니라 서브넷 보안 정책을 관리하여 더 쉽게 보안을 유지할 수 있습니다.

NSG가 서브넷과 연결되면 서브넷 내의 모든 서버에 적용됩니다. NSG를 사용하여 서브넷의 서버를 더 자세히 제어하는 방법에 대한 자세한 내용은 [네트워크 보안 그룹을 통한 네트워크 트래픽 필터링](https://azure.microsoft.com/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure/)을 참조하세요.

[VPN Gateway 계획 및 설계](/azure/vpn-gateway/vpn-gateway-plan-design)도 참조하세요.

### <a name="load-balancers"></a>부하 분산 장치

[SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw73ehp1/7.31.19/en-US/48/8fe37933114e6fe10000000a421937/frameset.htm)는 SAP 응용 프로그램 서버 풀에 대한 HTTP(S) 트래픽(Fiori 스타일 응용 프로그램 포함)의 부하 분산을 처리합니다. 

DIAG 또는 RFC(원격 함수 호출)를 통해 SAP 서버에 연결하는 SAP GUI 클라이언트의 트래픽에 대해 Central Services 메시지 서버는 SAP 응용 프로그램 서버 [로그온 그룹](https://wiki.scn.sap.com/wiki/display/SI/ABAP+Logon+Group+based+Load+Balancing)을 통해 부하를 분산하므로 추가 부하 분산 장치가 필요하지 않습니다. 

### <a name="azure-storage"></a>Azure Storage

데이터베이스 서버 가상 머신에 Azure Premium Storage를 사용하는 것이 좋습니다. Premium Storage는 일관된 읽기/쓰기 대기 시간을 제공합니다. 단일 인스턴스 가상 머신의 운영 체제 디스크 및 데이터 디스크에 Premium Storage를 사용하는 방법에 대한 자세한 내용은 [Virtual Machines에 대한 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/)를 참조하세요. 

모든 프로덕션 SAP 시스템의 경우 [Azure Managed Disks](/azure/storage/storage-managed-disks-overview) 프리미엄을 사용하는 것이 좋습니다. Managed Disks를 사용하여 디스크의 VHD 파일을 관리하므로 안정성이 향상됩니다. 또한 Managed Disks는 가용성 집합 내의 가상 머신용 디스크를 격리하여 단일 실패 지점을 방지합니다.

Central Services 가상 머신을 포함한 SAP 응용 프로그램 서버의 경우 응용 프로그램 실행이 메모리에서 수행되고 디스크가 로깅에만 사용되기 때문에 Azure Standard Storage를 사용하여 비용을 절감할 수 있습니다. 그러나 이 시점에서 Standard Storage는 관리되지 않는 저장소에 대해서만 인증되었습니다. 응용 프로그램 서버는 어떤 데이터도 호스팅하지 않으므로 더 작은 P4 및 P6 Premium Storage 디스크를 사용하여 비용을 최소화할 수 있습니다.

백업 데이터 저장소의 경우 Azure [쿨 액세스 계층 저장소 및/또는 보관 액세스 계층 저장소](/azure/storage/storage-blob-storage-tiers)를 사용하는 것이 좋습니다. 이러한 저장소 계층은 자주 액세스되지 않는 수명이 긴 데이터를 저장하는 비용 효율적인 방법입니다.

## <a name="performance-considerations"></a>성능 고려 사항

SAP 응용 프로그램 서버는 데이터베이스 서버와 지속적으로 통신합니다. HANA 데이터베이스 가상 머신의 경우 [쓰기 가속기](/azure/virtual-machines/linux/how-to-enable-write-accelerator)를 사용하도록 설정하여 로그 쓰기 대기 시간을 향상시키는 것이 좋습니다. 서버 간 통신을 최적화하려면 [가속 네트워크](https://azure.microsoft.com/blog/linux-and-windows-networking-performance-enhancements-accelerated-networking/)를 사용합니다. 이러한 가속기는 특정 VM 시리즈에서만 사용할 수 있습니다.

높은 IOPS 및 대역폭 처리량을 달성하려면 저장소 볼륨 [성능 최적화](/azure/virtual-machines/linux/premium-storage-performance)의 일반 사례를 Azure 저장소 레이아웃에 적용합니다. 예를 들어 여러 디스크를 결합하여 스트라이프 디스크 볼륨을 만들면 IO 성능이 향상됩니다. 자주 변경되지 않는 저장소 콘텐츠에 읽기 캐시를 사용하면 데이터 검색 속도가 향상됩니다. 성능 요구 사항에 대한 자세한 내용은 [SAP Note 1943937 - 하드웨어 구성 검사 도구](https://launchpad.support.sap.com/#/notes/1943937)를 참조하세요. (여기에 액세스하려면 SAP Service Marketplace 계정이 필요합니다.)

## <a name="scalability-considerations"></a>확장성 고려 사항

SAP 응용 프로그램 계층에서 Azure는 강화 및 확장을 위한 다양한 가상 머신 크기를 제공합니다. 전체 목록은 [SAP Note 1928533](https://launchpad.support.sap.com/#/notes/1928533) - Azure의 SAP 응용 프로그램: 지원되는 제품 및 Azure VM 유형을 참조하세요. (여기에 액세스하려면 SAP Service Marketplace 계정이 필요합니다.) 더 많은 가상 머신 유형을 계속 인증함에 따라 동일한 클라우드 배포를 통해 강화 또는 축소할 수 있습니다. 

데이터베이스 계층에서 이 아키텍처는 VM에서 HANA를 실행합니다. 워크로드가 최대 VM 크기를 초과하는 경우 Microsoft는 SAP HANA에 [Azure 대규모 인스턴스](/azure/virtual-machines/workloads/sap/hana-overview-architecture)도 제공합니다. 이러한 실제 서버는 Microsoft Azure 인증 데이터 센터에 함께 배치되며, 이 문서를 작성한 시점에서는 단일 인스턴스에 대해 최대 20TB의 메모리 용량을 제공합니다. 다중 노드 구성도 가능하며 총 메모리 용량은 최대 60TB입니다.

## <a name="availability-considerations"></a>가용성 고려 사항

리소스 중복성은 고가용성 인프라 솔루션의 일반적인 주제입니다. 비교적 엄격하지 않은 SLA가 적용된 엔터프라이즈의 경우 단일 인스턴스 Azure VM에서 가동 시간 SLA를 제공합니다. 자세한 내용은 [Azure 서비스 수준 계약](https://azure.microsoft.com/support/legal/sla/)을 참조하세요.

SAP 응용 프로그램의 이러한 분산 설치에서는 고가용성을 달성하기 위해 기본 설치가 복제됩니다. 아키텍처의 각 계층마다 고가용성 설계가 다릅니다. 

### <a name="application-tier"></a>응용 프로그램 계층

- Web Dispatcher. 고가용성은 중복 Web Dispatcher 인스턴스를 통해 달성됩니다. SAP 설명서의 [SAP Web Dispatcher](https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm)를 참조하세요.
- Fiori 서버. 고가용성은 서버 풀 내의 트래픽의 부하를 분산하여 달성됩니다.
- Central Services. Azure Linux 가상 머신에 있는 Central Services의 고가용성을 위해 선택한 Linux 배포판에 적절한 고가용성 확장이 사용되고 고가용성 NFS 클러스터에서 DRBD 저장소를 호스팅합니다.
- 응용 프로그램 서버. 응용 프로그램 서버 풀 내에서 트래픽을 분산하여 고가용성을 얻습니다.

### <a name="database-tier"></a>데이터베이스 계층

이 참조 아키텍처는 두 개의 Azure 가상 머신으로 구성된 고가용성 SAP HANA 데이터베이스 시스템을 보여 줍니다. 데이터베이스 계층의 기본 시스템 복제 기능은 복제된 노드 간에 수동 또는 자동 장애 조치를 제공합니다.

- 수동 장애 조치의 경우 둘 이상의 HANA 인스턴스를 배포하고 HSR(HANA 시스템 복제)을 사용합니다.
- 자동 장애 조치의 경우 Linux 배포판용 HSR 및 Linux HAE(고가용성 확장)를 모두 사용합니다. Linux HAE는 HANA 리소스에 클러스터 서비스를 제공하여 장애 이벤트를 감지하고 잘못된 서비스를 정상 노드로 장애 조치하도록 오케스트레이션합니다. 

[Microsoft Azure에서 실행되는 SAP 인증 및 구성](/azure/virtual-machines/workloads/sap/sap-certifications)을 참조하세요.

### <a name="disaster-recovery-considerations"></a>재해 복구 고려 사항
각 계층은 서로 다른 전략을 사용하여 재해 복구(DR) 보호를 제공합니다.

- **응용 프로그램 서버 계층**. SAP 응용 프로그램 서버에는 비즈니스 데이터가 포함되어 있지 않습니다. Azure에서 간단한 DR 전략은 보조 지역에 SAP 응용 프로그램 서버를 만든 다음, 종료하는 것입니다. 주 응용 프로그램 서버의 구성이 업데이트되거나 커널이 업데이트되는 즉시 동일한 변경 내용이 보조 지역의 가상 머신에 적용되어야 합니다. 예를 들어 SAP 커널 실행 파일을 DR 가상 머신에 복사합니다. 응용 프로그램 서버를 보조 지역에 자동으로 복제하려면 [Azure Site Recovery](/azure/site-recovery/site-recovery-overview)가 권장되는 솔루션입니다. 이 백서를 작성한 시점에서는 ASR에서 Azure VM의 가속 네트워크 구성 설정 복제를 아직 지원하지 않습니다.

- **Central Services**. 이 SAP 응용 프로그램 스택의 구성 요소도 비즈니스 데이터를 유지하지 않습니다. 보조 지역에 VM을 구축하여 Central Services 역할을 실행할 수 있습니다. 주 Central Services 노드에서 동기화할 수 있는 유일한 콘텐츠는 /sapmnt 공유 콘텐츠입니다. 또한 주 Central Services 서버에서 구성이 업데이트되거나 커널이 업데이트되면 Central Services를 실행하는 보조 지역의 VM에서 반복해야 합니다. 두 서버를 동기화하려면 Azure Site Recovery를 사용하여 클러스터 노드를 복제하거나, 정기적으로 예약된 복사 작업을 사용하여 /sapmnt를 DR 쪽에 복사하면 됩니다. 장애 조치 프로세스를 빌드, 복사 및 테스트하는 방법에 대한 자세한 내용은 [SAP NetWeaver: Hyper-V 및 Microsoft Azure 기반 재해 복구 솔루션 빌드](http://download.microsoft.com/download/9/5/6/956FEDC3-702D-4EFB-A7D3-2DB7505566B6/SAP%20NetWeaver%20-%20Building%20an%20Azure%20based%20Disaster%20Recovery%20Solution%20V1_5%20.docx)를 다운로드하여 4.3, "SAP SPOF 계층(ASCS)"을 참조하세요. 이 백서는 Windows에서 실행되는 NetWeaver에 적용되지만, Linux에 대해 동일한 구성을 만들 수 있습니다. Central Services의 경우 [Azure Site Recovery](/en-us/azure/site-recovery/site-recovery-overview)를 사용하여 클러스터 노드와 저장소를 복제합니다. Linux의 경우 고가용성 확장을 사용하여 3개 노드 지역 클러스터를 만듭니다. 

- **SAP 데이터베이스 계층**. HANA 지원 복제에 HSR을 사용합니다. HSR은 로컬 2개 노드 고가용성 설정 외에도 별도의 Azure 지역에 있는 세 번째 노드가 클러스터에 속하지 않은 외부 엔터티의 역할을 하고, 클러스터된 HSR 쌍의 보조 복제본에 복제 대상으로 등록하는 다중 계층 복제를 지원합니다. 이렇게 하면 복제 데이지 체인이 만들어집니다. DR 노드로의 장애 조치는 수동 프로세스입니다.

Azure Site Recovery를 사용하여 원래 사이트가 완전히 복제된 사이트를 자동으로 구축하려면 사용자 지정 [배포 스크립트](/azure/site-recovery/site-recovery-runbook-automation)를 실행해야 합니다. Site Recovery는 먼저 가용성 집합에 가상 머신을 배포한 다음, 스크립트를 실행하여 부하 분산 장치와 같은 리소스를 추가합니다. 

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

SAP HANA에는 기본 Azure 인프라를 사용하는 백업 기능이 있습니다. Azure 가상 머신에서 실행되는 SAP HANA 데이터베이스를 백업하려면 SAP HANA 스냅숏과 Azure 저장소 스냅숏을 모두 사용하여 백업 파일의 일관성을 유지해야 합니다. 자세한 내용은 [Azure Virtual Machines의 SAP HANA Backup 가이드](/azure/virtual-machines/workloads/sap/sap-hana-backup-guide) 및 [Azure Backup 서비스 FAQ](/azure/backup/backup-azure-backup-faq)를 참조하세요. 단일 HANA 컨테이너 배포만 Azure 저장소 스냅숏을 지원합니다.

### <a name="identity-management"></a>ID 관리

모든 수준에서 중앙 집중식 ID 관리 시스템을 사용하여 리소스에 대한 액세스를 제어합니다.

- [RBAC(역할 기반 액세스 제어)](/azure/active-directory/role-based-access-control-what-is)를 통해 Azure 리소스에 대한 액세스를 제공합니다. 
- LDAP, Azure Active Directory, Kerberos 또는 다른 시스템을 통해 Azure VM에 대한 액세스 권한을 부여합니다. 
- SAP에서 제공하는 서비스를 통해 앱 자체 내의 액세스를 지원하거나 [OAuth 2.0 및 Azure Active Directory](/azure/active-directory/develop/active-directory-protocols-oauth-code)를 사용합니다. 

### <a name="monitoring"></a>모니터링

Azure는 전체 인프라를 [모니터링 및 진단](/azure/architecture/best-practices/monitoring)하는 몇 가지 기능을 제공합니다. 또한 Azure 가상 머신(Linux 또는 Windows)의 향상된 모니터링은 Azure OMS(Operations Management Suite)를 통해 처리됩니다. 

SAP 인프라의 리소스 및 서비스 성능에 대한 SAP 기반 모니터링을 제공하기 위해 [Azure SAP 고급 모니터링 확장](/azure/virtual-machines/workloads/sap/deployment-guide#d98edcd3-f2a1-49f7-b26a-07448ceb60ca)을 사용합니다. 이 확장은 SAP 응용 프로그램에 운영 체제 모니터링 및 DBA Cockpit 함수에 대한 Azure 모니터링 통계를 제공합니다. SAP 고급 모니터링은 Azure에서 SAP를 실행하기 위한 필수 구성 요소입니다. 자세한 내용은 [SAP Note 2191498](https://launchpad.support.sap.com/#/notes/2191498) - "Azure를 사용하는 Linux의 SAP: 고급 모니터링"을 참조하세요.

## <a name="security-considerations"></a>보안 고려 사항

SAP는 자체적인 UME(사용자 관리 엔진)를 사용하여 SAP 응용 프로그램의 역할 기반 액세스 및 권한 부여를 제어합니다. 자세한 내용은 [SAP HANA 보안 - 개요](https://archive.sap.com/documents/docs/DOC-62943)를 참조하세요. (여기에 액세스하려면 SAP Service Marketplace 계정이 필요합니다.)

추가 네트워크 보안을 위해 네트워크 가상 어플라이언스를 사용하여 Web Dispatcher 및 Fiori 프런트 엔드 서버 풀의 서브넷 앞에 방화벽을 만드는 [네트워크 DMZ](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid)를 구현하는 것이 좋습니다.

인프라 보안을 위해 전송 중 데이터와 미사용 데이터가 암호화됩니다. [SAP NetWeaver에 대한 Azure Virtual Machines 계획 및 구현 가이드](/azure/virtual-machines/workloads/sap/planning-guide)의 "보안 고려 사항" 섹션에서 네트워크 보안 처리를 다루고 있으며, 이는 S/4HANA에 적용됩니다. 이 가이드에서는 응용 프로그램 통신을 허용하기 위해 방화벽에서 열어야 하는 네트워크 포트도 지정합니다. 

Linux IaaS 가상 머신 디스크를 암호화하려면 [Azure Disk Encryption](/azure/security/azure-security-disk-encryption)을 사용합니다. Linux의 DM-Crypt 기능을 사용하여 운영 체제 및 데이터 디스크에 대한 볼륨 암호화를 제공합니다. 또한 이 솔루션은 Azure Key Vault와 함께 작동하여 키 자격 증명 모음 구독에서 디스크 암호화 키와 비밀을 제어하고 관리할 수 있습니다. 가상 머신 디스크의 데이터는 미사용 시 Azure 저장소에 암호화됩니다.

SAP HANA 미사용 데이터 암호화의 경우 SAP HANA 네이티브 암호화 기술을 사용하는 것이 좋습니다. 

> [!NOTE]
> HANA 미사용 데이터 암호화는 동일한 서버에서 Azure Disk Encryption을 통해 사용하지 마세요. HANA의 경우 HANA 데이터 암호화만 사용합니다.

## <a name="communities"></a>커뮤니티

커뮤니티는 질문에 대답하고 성공적인 배포를 설정하는 데 도움을 줄 수 있습니다. 다음을 고려해 보세요.

- [Microsoft 플랫폼에서 SAP 응용 프로그램 실행 블로그](https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/)
- [Azure 커뮤니티 지원](https://azure.microsoft.com/support/community/)
- [SAP 커뮤니티](https://www.sap.com/community.html)
- [스택 오버플로](https://stackoverflow.com/tags/sap/)
