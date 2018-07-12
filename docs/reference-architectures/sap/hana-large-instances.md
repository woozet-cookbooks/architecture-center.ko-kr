---
title: Azure의 SAP HANA(대규모 인스턴스) 실행
description: Azure 대규모 인스턴스의 고가용성 환경에서 SAP HANA를 실행하는 검증된 사례입니다.
author: lbrader
ms.date: 05/16/2018
ms.openlocfilehash: 746161ac51335af5c48a559830d6e0345dcfb7b1
ms.sourcegitcommit: 86d86d71e392550fd65c4f76320d7ecf0b72e1f6
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/06/2018
ms.locfileid: "37864524"
---
# <a name="run-sap-hana-on-azure-large-instances"></a>Azure의 SAP HANA(대규모 인스턴스) 실행

이 참조 아키텍처는 고가용성 및 DR(재해 복구)을 통해 Azure의 SAP HANA(대규모 인스턴스)를 실행하는 일단의 검증된 사례를 보여 줍니다. HANA 대규모 인스턴스라고 하는 이 제품은 Azure 지역의 실제 서버에 배포됩니다. 

![0][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

> [!NOTE]
> 이 참조 아키텍처를 배포하려면 적절한 SAP 제품 라이선스 및 기타 Microsoft 이외의 기술이 필요합니다.

## <a name="architecture"></a>아키텍처

이 아키텍처를 구성하는 인프라 구성 요소는 다음과 같습니다.

- **가상 네트워크**. [Azure 가상 네트워크][vnet] 서비스는 Azure 리소스를 서로 안전하게 연결하고, 각 계층마다 별도의 [서브넷][subnet]으로 세분화됩니다. SAP 응용 프로그램 계층은 대규모 인스턴스에 있는 HANA 데이터베이스 계층에 연결하기 위해 Azure VM(가상 머신)에 배포됩니다.

- **가상 머신**. 가상 머신은 SAP 응용 프로그램 계층과 공유 서비스 계층에서 사용됩니다. 후자에는 관리자가 HANA 대규모 인스턴스를 설정하고 다른 가상 머신에 대한 액세스를 제공하는 데 사용되는 jumpbox가 포함됩니다. 

- **HANA 대규모 인스턴스**. SAP HANA TDI(Tailored Datacenter Integration) 표준을 충족하도록 인증된 [실제 서버][physical]에서 SAP HANA를 실행합니다. 이 아키텍처는 각각 기본 및 보조 계산 단위인 두 개의 HANA 대규모 인스턴스를 사용합니다. 데이터 계층의 고가용성은 HSR(HANA 시스템 복제)을 통해 제공됩니다.

- **고가용성 쌍**. HANA 대규모 인스턴스 블레이드 그룹을 함께 관리하여 응용 프로그램 중복성과 안정성을 제공합니다. 

- **MSEE(Microsoft Enterprise Edge)**. MSEE는 ExpressRoute 회선을 통한 연결 공급자 또는 네트워크 에지의 연결 지점입니다. 

- **NIC(네트워크 인터페이스 카드)**. HANA 대규모 인스턴스 서버에서 통신을 위해 기본적으로 4개의 가상 NIC를 제공합니다. 이 아키텍처에는 클라이언트 통신, HSR에 필요한 노드 간 연결, HANA 대규모 인스턴스 저장소 및 고가용성 클러스터링에 사용되는 iSCSI를 위해 각각 하나씩 4개의 NIC가 필요합니다.
    
- **NFS(네트워크 파일 시스템) 저장소**. [NFS][nfs] 서버는 HANA 대규모 인스턴스에 대한 보안 데이터 지속성을 제공하는 네트워크 파일 공유를 지원합니다.

- **ExpressRoute.** [ExpressRoute][expressroute]는 공용 인터넷을 통해 통신하지 않는 Azure 가상 네트워크와 온-프레미스 네트워크 간의 사설 연결을 만드는 데 권장되는 Azure 네트워킹 서비스입니다. Azure VM은 또 다른 ExpressRoute 연결을 사용하여 HANA 대규모 인스턴스에 연결합니다. Azure 가상 네트워크와 HANA 대규모 인스턴스 간의 ExpressRoute 연결은 Microsoft 제품의 일부로 설정됩니다.

- **게이트웨이**. ExpressRoute 게이트웨이를 사용하여 SAP 응용 프로그램 계층에 사용되는 Azure 가상 네트워크를 HANA 대규모 인스턴스 네트워크에 연결합니다. [고성능 또는 초고성능][sku] SKU를 사용합니다.

- **DR(재해 복구)**. 요청 시 저장소 복제가 지원되며, 주 사이트에서 다른 지역에 있는 [DR 사이트][DR-site]로의 복제로 구성됩니다.  
 
## <a name="recommendations"></a>권장 사항
요구 사항이 다를 수 있으므로 이러한 권장 사항을 시작 지점으로 사용합니다.

### <a name="hana-large-instances-compute"></a>HANA 대규모 인스턴스 계산
[대규모 인스턴스][physical]는 Intel EX E7 CPU 아키텍처에 따라 대규모 인스턴스 스탬프, 즉 서버 또는 블레이드의 특정 집합으로 구성된 실제 서버입니다. 계산 단위는 하나의 서버 또는 블레이드와 같고, 스탬프는 여러 개의 서버 또는 블레이드로 구성됩니다. 서버는 대규모 인스턴스 스탬프 내에서 공유되지 않으며, 한 고객의 SAP HANA 배포를 실행하는 데 전적으로 사용됩니다.

HANA 대규모 인스턴스에 다양한 SKU를 사용할 수 있으며, S/4HANA 또는 다른 SAP HANA 워크로드에 최대 20TB의 단일 인스턴스(60TB 확장 가능) 메모리를 지원합니다. 서버에서 제공하는 [두 가지 등급][classes]은 다음과 같습니다.

- 유형 I 등급: S72, S72m, S144, S144m, S192 및 S192m

- 유형 II 등급: S384, S384m, S384xm, S576m, S768m 및 S960m

예를 들어 S72 SKU에서는 768GB RAM, 3TB 저장소 및 2개 Intel Xeon 프로세서(E7-8890 v3, 36개 코어 포함)를 제공합니다. 아키텍처 및 디자인 세션에서 결정한 크기 조정 요구 사항을 충족하는 SKU를 선택합니다. 항상 크기 조정이 올바른 SKU에 적용되는지 확인합니다. 기능 및 배포 요구 사항은 [유형][type]에 따라 다르며, 가용성은 [지역][region]에 따라 다릅니다. 하나의 SKU에서 더 큰 SKU로 확장할 수 있습니다.

Microsoft는 대규모 인스턴스 설치를 설정하는 데 지원할 수 있지만, 운영 체제의 구성 설정은 사용자가 확인해야 합니다. 정확한 Linux 릴리스는 최신 SAP Note를 검토해야 합니다.

### <a name="storage"></a>Storage
저장소 레이아웃은 SAP HANA에 대한 TDI 권장 사항에 따라 구현됩니다. HANA 대규모 인스턴스는 표준 TDI 사양에 적합한 특정 저장소 구성을 제공합니다. 그러나 추가 저장소는 1TB 단위로 구매할 수 있습니다. 

NFS는 빠른 복구를 포함하여 중요 업무용 환경의 요구 사항을 지원하기 위해 사용되며, 직접 연결된 저장소가 아닙니다. HANA 대규모 인스턴스용 NFS 저장소 서버는 계산, 네트워크 및 저장소 격리를 사용하여 테넌트를 분리 및 보호하는 다중 테넌트 환경에서 호스팅됩니다.

주 사이트에서 고가용성을 지원하려면 다른 저장소 레이아웃을 사용합니다. 예를 들어 다중 호스트 확장에서는 저장소가 공유됩니다. 또 다른 고가용성 옵션은 HSR과 같은 응용 프로그램 기반 복제입니다. 그러나 DR에서는 스냅숏 기반 저장소 복제가 사용됩니다.

### <a name="networking"></a>네트워킹
이 아키텍처는 가상 및 실제 네트워크를 모두 사용합니다. 가상 네트워크는 Azure IaaS의 일부이며, [ExpressRoute][expressroute]] 회로를 통해 분리된 개별 HANA 대규모 인스턴스 실제 네트워크에 연결됩니다. 프레미스 간 게이트웨이는 Azure 가상 네트워크의 워크로드를 온-프레미스 사이트에 연결합니다.

HANA 대규모 인스턴스 네트워크는 보안을 위해 서로 격리되어 있습니다. 다른 지역에 있는 인스턴스는 전용 저장소 복제를 제외하고는 서로 통신하지 않습니다. 그러나 HSR을 사용하려면 지역 간 통신이 필요합니다. [IP 라우팅 테이블][ip] 또는 프록시는 지역 간 HSR을 수행할 있게 하는 데 사용할 수 있습니다.

한 지역의 HANA 대규모 인스턴스에 연결된 모든 Azure 가상 네트워크는 ExpressRoute를 통해 보조 지역의 HANA 대규모 인스턴스에 [상호 연결][cross-connected]될 수 있습니다.

HANA 대규모 인스턴스용 ExpressRoute는 기본적으로 프로비전 중에 포함됩니다. 설치 시 필요한 CIDR 주소 범위 및 도메인 라우팅을 포함한 특정 네트워크 레이아웃이 필요합니다. 자세한 내용은 [Azure의 SAP HANA(대규모 인스턴스) 인프라 및 연결][HLI-infrastructure]을 참조하세요.

## <a name="scalability-considerations"></a>확장성 고려 사항
확장하거나 축소하려면 HANA 대규모 인스턴스에서 사용할 수 있는 다양한 크기의 서버 중에서 선택할 수 있습니다. 이러한 서버는 [유형 I 및 유형 II][classes]로 분류되며 서로 다른 워크로드에 맞게 조정됩니다. 향후 3년 동안 워크로드에 따라 증가할 수 있는 크기를 선택합니다. 1년 약정도 가능합니다.

다중 호스트 확장 배포는 일반적으로 BW/4HANA 배포를 위한 일종의 데이터베이스 분할 전략으로 사용됩니다. 설치하기 전에 HANA 테이블 배치를 계획하여 확장합니다. 인프라 관점에서 여러 호스트가 공유 저장소 볼륨에 연결되어 HANA 시스템의 계산 작업자 노드 중 하나에 장애가 발생하면 대기 호스트에서 신속하게 인수할 수 있습니다.

단일 블레이드의 HANA S/4HANA 및 SAP Business Suite는 단일 HANA 대규모 인스턴스로 최대 20TB까지 확장할 수 있습니다.

개발 가능한 시나리오의 경우 [SAP Quick Sizer][quick-sizer]를 사용하여 HANA에 기반한 SAP 소프트웨어를 구현하는 데 필요한 메모리 요구 사항을 계산할 수 있습니다. 데이터 볼륨이 증가함에 따라 HANA에 대한 메모리 요구 사항도 증가합니다. 시스템의 현재 메모리 사용량을 향후 사용량을 예측하기 위한 기준으로 사용한 다음, 수요량을 HANA 대규모 인스턴스 크기 중 하나에 매핑합니다.

SAP 배포가 이미 있는 경우 SAP는 기존 시스템에서 사용하는 데이터를 확인하고 HANA 인스턴스의 메모리 요구 사항을 계산하는 데 사용할 수 있는 보고서를 제공합니다. 예를 들어 다음 SAP Note를 참조하세요. 

- SAP Note [1793345][sap-1793345] - SAP Suite on HANA 크기 조정
- SAP Note [1872170][sap-1872170] - Suite on HANA 및 S/4 HANA 크기 조정 보고서
- SAP Note [2121330][sap-2121330] - FAQ: SAP BW on HANA 크기 조정 보고서
- SAP Note [1736976][sap-1736976] - BW on HANA 크기 조정 보고서
- SAP Note [2296290][sap-2296290] - 새 BW on HANA 크기 조정 보고서

## <a name="availability-considerations"></a>가용성 고려 사항

리소스 중복성은 고가용성 인프라 솔루션의 일반적인 주제입니다. 비교적 엄격하지 않은 SLA가 적용된 엔터프라이즈의 경우 단일 인스턴스 Azure VM에서 가동 시간 SLA를 제공합니다. 자세한 내용은 [Azure 서비스 수준 계약](https://azure.microsoft.com/support/legal/sla/)을 참조하세요.

SAP, 시스템 통합업체 또는 Microsoft와 협력하여 [고가용성 및 재해 복구][hli-hadr] 전략을 적절하게 설계하고 구현합니다. 이 아키텍처는 Azure의 HANA(대규모 인스턴스)에 대한 Azure [SLA(서비스 수준 계약)][sla]를 따릅니다. 가용성 요구 사항을 평가하려면 단일 실패 지점, 서비스에 원하는 가동 시간 수준 및 다음과 같은 공통 메트릭을 고려합니다.

- RTO(복구 시간 목표)는 HANA 대규모 인스턴스 서버를 사용할 수 없는 기간을 의미합니다.

- RPO(복구 지점 목표)는 장애로 인해 고객 데이터가 손실될 수 있는 최대 허용 기간을 의미합니다.

고가용성을 위해 HA 쌍에 둘 이상의 인스턴스를 배포하고, 동기 모드에서 HSR을 사용하여 데이터 손실 및 가동 중지 시간을 최소화합니다. HSR은 로컬 2개 노드 고가용성 설정 외에도 별도의 Azure 지역에 있는 세 번째 노드를 클러스터된 HSR 쌍의 보조 복제본에 복제 대상으로 등록하는 다중 계층 복제를 지원합니다. 이렇게 하면 복제 데이지 체인이 만들어집니다. DR 노드로의 장애 조치는 수동 프로세스입니다.

자동 장애 조치를 사용하도록 HANA 대규모 인스턴스 HSR을 설정하는 경우 기존 서버에 대해 [STONITH 장치][stonith]를 설정하도록 Microsoft 서비스 관리 팀에 요청할 수 있습니다. 

## <a name="disaster-recovery-considerations"></a>재해 복구 고려 사항
이 아키텍처는 다른 Azure 지역에 있는 HANA 대규모 인스턴스 간의 [재해 복구][hli-dr]를 지원합니다. HANA 대규모 인스턴스를 사용하여 DR을 지원하는 두 가지 방법은 다음과 같습니다.

- 저장소 복제. 주 저장소 콘텐츠는 지정된 DR HANA 대규모 인스턴스 서버에서 사용할 수 있는 원격 DR 저장소 시스템으로 지속적으로 복제됩니다. 저장소 복제에서 HANA 데이터베이스는 메모리로 로드되지 않습니다. 이 DR 옵션은 관리 관점에서 더 간단합니다. 이 전략이 적합한지 확인하려면 가용성 SLA에 대한 데이터베이스 로드 시간을 고려합니다. 저장소 복제를 사용하면 특정 시점 복구를 수행할 수도 있습니다. 다목적(비용 최적화) DR을 설정하는 경우 DR 위치에서 동일한 크기의 추가 저장소를 구매해야 합니다. Microsoft는 HANA 대규모 인스턴스 제품의 일부로 HANA 장애 조치에 대한 자체 서비스 [저장소 스냅숏 및 장애 조치 스크립트][scripts]를 제공합니다.

- DR 지역에 세 번째 복제본이 있는 다중 계층 HSR(HANA 데이터베이스가 메모리에 로드됨). 이 옵션은 빠른 복구 시간을 지원하지만 특정 시점 복구는 지원하지 않습니다. HSR에는 보조 시스템이 필요합니다. DR 사이트에 대한 HANA 시스템 복제는 nginx 또는 IP 테이블과 같은 프록시를 통해 처리됩니다. 

> [!NOTE]
> 이 참조 아키텍처는 단일 인스턴스 환경에서 실행하여 비용에 맞게 최적화할 수 있습니다. 이 [비용 최적화 시나리오](https://blogs.sap.com/2016/07/19/new-whitepaper-for-high-availability-for-sap-hana-cost-optimized-scenario/)는 비프로덕션 HANA 워크로드에 적합합니다. 

## <a name="backup-considerations"></a>백업 고려 사항
비즈니스 요구 사항에 따라 [백업 및 복구][hli-backup]에 사용할 수 있는 몇 가지 옵션 중에서 선택합니다.

| 백업 옵션                   | 장점                                                                                                   | 단점                                                       |
|---------------------------------|--------------------------------------------------------------------------------------------------------|------------------------------------------------------------|
| HANA 백업        | 기본적인 SAP 옵션입니다. 일관성 확인이 기본적으로 제공됩니다.                                                             | 백업 및 복구 시간이 오래 걸립니다. 저장소 공간을 소모합니다. |
| HANA 스냅숏      | 기본적인 SAP 옵션입니다. 백업 및 복원 시간이 빠릅니다.                                                               |                                       |
| 스냅숏 저장소   | HANA 대규모 인스턴스에 포함됩니다. DR을 HANA 대규모 인스턴스에 맞게 최적화합니다. 부팅 볼륨 백업을 지원합니다. | 볼륨당 스냅숏 수가 최대 254개입니다.                          |
| 로그 백업         | 특정 시점 복구에 필요합니다.                                                                   |                                                            |
| 기타 백업 도구 | 중복 백업 위치를 사용합니다.                                                                             | 추가 라이선스 비용이 발생합니다.                                |

또한 SapHanaTutorial.com은 유용한 문서인 [HANA 백업 옵션 간 비교][sap-hana-tutorial]를 제공합니다.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항
SAP HANA Studio, SAP HANA Cockpit, SAP Solution Manager 및 기타 네이티브 Linux 도구를 사용하여 HANA 대규모 인스턴스 리소스(예: CPU, 메모리, 네트워크 대역폭 및 저장소 공간)를 모니터링합니다. HANA 대규모 인스턴스에는 모니터링 도구가 기본 제공되지 않습니다. Microsoft는 조직의 요구 사항에 따라 [문제를 해결하고 모니터링][hli-troubleshoot]하는 데 유용한 리소스를 제공하며, Microsoft 기술 지원 팀에서 기술 문제를 해결하는 데 지원할 수 있습니다. 

컴퓨팅 기능이 더 많이 필요하면 더 큰 SKU를 획득해야 합니다. 

## <a name="security-considerations"></a>보안 고려 사항
- 기본적으로 HANA 대규모 인스턴스는 미사용 데이터에 대해 TDE(투명한 데이터 암호화) 기반의 저장소 암호화를 사용합니다.

- HANA 대규모 인스턴스와 가상 머신 간에 전송되는 데이터는 암호화되지 않습니다. 데이터 전송을 암호화하려면 응용 프로그램 특정 암호화를 사용하도록 설정합니다. SAP Note [2159014][sap-2159014] - FAQ: SAP HANA 보안을 참조하세요.

- 격리는 다중 테넌트 HANA 대규모 인스턴스 환경에서 테넌트 간 보안을 제공합니다. 테넌트는 자체 VLAN을 사용하여 격리됩니다.

- [Azure 네트워크 보안 모범 사례][network-best-practices]에서 유용한 지침을 제공합니다.

- 모든 배포와 마찬가지로 [운영 체제 강화][os-hardening]가 권장됩니다.

- 물리적 보안을 위해 권한이 있는 직원만 Azure 데이터 센터에 액세스할 수 있습니다. 고객은 실제 서버에 액세스할 수 없습니다.

자세한 내용은 [SAP HANA 보안 - 개요][sap-security]를 참조하세요. (여기에 액세스하려면 SAP Service Marketplace 계정이 필요합니다.)

## <a name="communities"></a>커뮤니티
커뮤니티는 질문에 대답하고 성공적인 배포를 설정하는 데 도움을 줄 수 있습니다. 다음을 고려해 보세요.

* [Microsoft 플랫폼에서 SAP 응용 프로그램 실행 블로그][running-sap-blog]
* [Azure 커뮤니티 지원][azure-forum]
* [SAP 커뮤니티][sap-community]
* [Stack Overflow SAP][stack-overflow]

[azure-forum]: https://azure.microsoft.com/support/forums/
[azure-large-instances]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[classes]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[cross-connected]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#network-considerations-for-disaster-recovery-with-hana-large-instances
[dr-site]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery
[expressroute]: /azure/architecture/reference-architectures/hybrid-networking/expressroute
[filter-network]: https://azure.microsoft.com/blog/multiple-vm-nics-and-network-virtual-appliances-in-azure/
[hli-dr]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#network-considerations-for-disaster-recovery-with-hana-large-instances
[hli-backup]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#backup-and-restore
[hli-hadr]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[hli-infrastructure]: /azure/virtual-machines/workloads/sap/hana-overview-infrastructure-connectivity
[hli-overview]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[hli-troubleshoot]: /azure/virtual-machines/workloads/sap/troubleshooting-monitoring
[ip]: https://blogs.msdn.microsoft.com/saponsqlserver/2018/02/10/setting-up-hana-system-replication-on-azure-hana-large-instances/
[network-best-practices]: /azure/security/azure-security-network-security-best-practices
[nfs]: /azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs
[os-hardening]: /azure/security/azure-security-iaas
[physical]: /azure/virtual-machines/workloads/sap/hana-overview-architecture
[planning]: /azure/vpn-gateway/vpn-gateway-plan-design
[protecting-sap]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/05/06/protecting-sap-systems-running-on-vmware-with-azure-site-recovery/
[ref-arch]: /azure/architecture/reference-architectures/
[running-SAP]: https://blogs.msdn.microsoft.com/saponsqlserver/2016/06/07/sap-on-sql-general-update-for-customers-partners-june-2016/
[region]: https://azure.microsoft.com/global-infrastructure/services/
[running-sap-blog]: https://blogs.msdn.microsoft.com/saponsqlserver/2017/05/04/sap-on-azure-general-update-for-customers-partners-april-2017/
[quick-sizer]: http://service.sap.com/quicksizing
[sap-1793345]: https://launchpad.support.sap.com/#/notes/1793345
[sap-1872170]: https://launchpad.support.sap.com/#/notes/1872170
[sap-2121330]: https://launchpad.support.sap.com/#/notes/2121330
[sap-2159014]: https://launchpad.support.sap.com/#/notes/2159014
[sap-1736976]: https://launchpad.support.sap.com/#/notes/1736976
[sap-2296290]: https://launchpad.support.sap.com/#/notes/2296290
[sap-community]: https://www.sap.com/community.html
[sap-hana-tutorial]: http://saphanatutorial.com/comparison-between-hana-backup-options/
[sap-security]: https://archive.sap.com/documents/docs/DOC-62943
[scripts]: /azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery
[sku]: /azure/expressroute/expressroute-about-virtual-network-gateways
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[stack-overflow]: http://stackoverflow.com/tags/sap/info
[stonith]: /azure/virtual-machines/workloads/sap/ha-setup-with-stonith
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[swd]: https://help.sap.com/doc/saphelp_nw70ehp2/7.02.16/en-us/48/8fe37933114e6fe10000000a421937/frameset.htm
[type]: /azure/virtual-machines/workloads/sap/hana-installation
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/sap-hana-large-instances.png "Azure 대규모 인스턴스를 사용하는 SAP HANA 아키텍처"

[visio-download]: https://archcenter.blob.core.windows.net/cdn/sap-reference-architectures.vsdx