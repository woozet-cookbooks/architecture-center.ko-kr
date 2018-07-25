---
title: 규제 산업용 Windows 웹 응용 프로그램 보호
description: 확장 집합, Application Gateway 및 부하 분산 장치를 사용하는 Azure의 Windows Server에서 안전한 다중 계층 웹 응용 프로그램을 구축하는 데 입증된 시나리오입니다.
author: iainfoulds
ms.date: 07/11/2018
ms.openlocfilehash: aba714fc1955341645d0faa400768bc09fb8e50b
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060994"
---
# <a name="secure-windows-web-application-for-regulated-industries"></a>규제 산업용 Windows 웹 응용 프로그램 보호

이 샘플 시나리오는 다중 계층 응용 프로그램을 보호해야 하는 규제 산업에 적용할 수 있습니다. 이 시나리오에서는 프런트 엔드 ASP.NET 응용 프로그램이 보호된 백 엔드 Microsoft SQL Server 클러스터에 안전하게 연결됩니다.

예제 응용 프로그램 시나리오에는 수술실 응용 프로그램 실행, 환자 약속 및 기록 보관 또는 처방전 보충 및 주문이 포함됩니다. 일반적으로 조직에서는 이러한 시나리오에 대한 레거시 온-프레미스 응용 프로그램 및 서비스를 유지 관리해야 했습니다. 조직은 Azure에서 이러한 Windows Server 응용 프로그램을 안전하고 확장 가능한 방식으로 배포함으로써 배포를 현대화하여 온-프레미스 운영 비용과 관리 오버헤드를 줄일 수 있습니다.

## <a name="related-use-cases"></a>관련 사용 사례

이 시나리오에 적합한 사용 사례는 다음과 같습니다.

* 보안 클라우드 환경에서 응용 프로그램 배포 현대화
* 레거시 온-프레미스 응용 프로그램 및 서비스 관리 감소
* 새 응용 프로그램 플랫폼을 통해 환자에 대한 의료 서비스 및 환경 개선

## <a name="architecture"></a>아키텍처

![규제 산업용 다중 계층 Windows Server 응용 프로그램과 관련된 Azure 구성 요소 아키텍처에 대한 개요][architecture]

이 시나리오에서는 ASP.NET 및 Microsoft SQL Server를 사용하는 다중 계층 규제 산업 응용 프로그램에 대해 설명합니다. 시나리오를 통한 데이터 흐름은 다음과 같습니다.

1. 사용자는 Azure Application Gateway를 통해 프런트 엔드 ASP.NET 규제 산업 응용 프로그램에 액세스합니다.
2. Application Gateway에서 Azure 가상 머신 확장 집합 내의 VM 인스턴스에 트래픽을 분산시킵니다.
3. ASP.NET 응용 프로그램에서 Azure 부하 분산 장치를 통해 백 엔드 계층의 Microsoft SQL Server 클러스터에 연결합니다. 이러한 백 엔드 SQL Server 인스턴스는 별도의 Azure 가상 네트워크에 있으며 트래픽 흐름을 제한하는 네트워크 보안 그룹 규칙으로 보호됩니다.
4. 부하 분산 장치에서 SQL Server 트래픽을 다른 가상 머신 확장 집합의 VM 인스턴스에 배포합니다.
5. Azure Blob Storage는 백 엔드 계층의 SQL Server 클러스터에 대한 클라우드 감시 역할을 합니다.  VNet 내에서의 연결은 Azure Storage에 대한 VNet 서비스 엔드포인트를 통해 가능합니다.

### <a name="components"></a>구성 요소

* [Azure Application Gateway][appgateway-docs]는 응용 프로그램을 인식하고 특정 라우팅 규칙에 따라 트래픽을 분산할 수 있는 7계층 웹 트래픽 부하 분산 장치입니다. 또한 App Gateway는 향상된 웹 서버 성능을 위해 SSL 오프로드를 처리할 수도 있습니다.
* [Azure Virtual Network][vnet-docs]는 VM과 같은 리소스에서 상호 간 통신, 인터넷 통신 및 온-프레미스 네트워크 통신을 안전하게 수행할 수 있게 합니다. 가상 네트워크는 격리 및 세분화를 제공하고, 트래픽을 필터링 및 라우팅하며, 위치 간 연결을 허용합니다. 이 시나리오에서는 적절한 NSG와 결합된 두 가상 네트워크를 사용하여 [DMZ][dmz](완충 영역) 및 응용 프로그램 구성 요소 격리를 제공합니다. 가상 네트워크 피어링은 두 네트워크를 서로 연결합니다.
* [Azure 가상 머신 확장 집합][scaleset-docs]을 사용하면 부하 분산된 동일한 VM 그룹을 만들고 관리할 수 있습니다. VM 인스턴스의 수는 요구 또는 정의된 일정에 따라 자동으로 늘리거나 줄일 수 있습니다. 이 시나리오에서는 각각 프런트 엔드 ASP.NET 응용 프로그램 인스턴스 및 백 엔드 SQL Server 클러스터 VM 인스턴스에 대한 별도의 두 가상 머신 확장 집합이 사용됩니다. PowerShell DSC(Desired State Configuration) 또는 Azure 사용자 지정 스크립트 확장을 사용하여 VM 인스턴스에 필요한 소프트웨어 및 구성 설정을 프로비전할 수 있습니다.
* [Azure 네트워크 보안 그룹][nsg-docs]에는 원본 또는 대상 IP 주소, 포트 및 프로토콜에 따라 인바운드 또는 아웃바운드 네트워크 트래픽을 허용하거나 거부하는 보안 규칙 목록이 포함되어 있습니다. 이 시나리오의 가상 네트워크는 응용 프로그램 구성 요소 간의 트래픽 흐름을 제한하는 네트워크 보안 그룹 규칙으로 보호됩니다.
* [Azure 부하 분산 장치][loadbalancer-docs]는 규칙 및 상태 프로브에 따라 인바운드 트래픽을 분산시킵니다. 부하 분산 장치는 짧은 대기 시간과 높은 처리량을 제공하고, 모든 TCP 및 UDP 응용 프로그램에 대해 최대 수백만 개의 흐름으로 확장합니다. 이 시나리오에서는 내부 부하 분산 장치를 사용하여 프런트 엔드 응용 프로그램 계층에서 백 엔드 SQL Server 클러스터로 트래픽을 분산시킵니다.
* [Azure Blob Storage][cloudwitness-docs]는 SQL Server 클러스터에 대한 클라우드 감시 위치 역할을 합니다. 이 감시는 쿼럼을 결정하기 위해 추가 투표가 필요한 클러스터 작업 및 의사 결정에 사용됩니다. 클라우드 감시를 사용하면 추가 VM에서 기존 파일 공유 감시 역할을 수행할 필요가 없습니다.

### <a name="alternatives"></a>대안

* 인프라의 구성 요소는 OS에 따라 달라지지 않으므로 Windows는 Unix와 마찬가지로 다양한 다른 OS로 쉽게 대체될 수 있습니다.

* [Linux용 SQL Server][sql-linux]는 백 엔드 데이터 저장소를 대체할 수 있습니다.

* [Cosmos DB][cosmos]는 데이터 저장소의 또 다른 대안입니다.

## <a name="considerations"></a>고려 사항

### <a name="availability"></a>가용성

이 시나리오의 VM 인스턴스는 가용성 영역 전체에 배포됩니다. 각 영역은 독립된 전원, 냉각 및 네트워킹을 갖춘 하나 이상의 데이터 센터로 구성됩니다. 사용 가능한 모든 지역에서 최소 세 개의 영역을 사용할 수 있습니다. 영역을 통한 이러한 VM 인스턴스 배포는 응용 프로그램 계층에 고가용성을 제공합니다. 자세한 내용은 [Azure에서 가용성 영역이란?][azureaz-docs]을 참조하세요.

데이터베이스 계층에서는 Always On 가용성 그룹을 사용하도록 구성할 수 있습니다. 이 SQL Server 구성을 사용하면 클러스터 내에서 하나의 주 데이터베이스가 최대 8개의 보조 데이터베이스로 구성됩니다. 주 데이터베이스에 문제가 발생하면 클러스터에서 보조 데이터베이스 중 하나에 장애 조치하여 응용 프로그램을 계속 사용할 수 있습니다. 자세한 내용은 [SQL Server에 대한 Always On 가용성 그룹 개요][sqlalwayson-docs]를 참조하세요.

다른 가용성 항목은 Azure 아키텍처 센터의 [가용성 검사 목록][availability]을 참조하세요.

### <a name="scalability"></a>확장성

이 시나리오에서는 프론트 엔드 및 백 엔드 구성 요소에 대한 가상 머신 확장 집합을 사용합니다. 확장 집합을 사용하면 프론트 엔드 응용 프로그램 계층을 실행하는 VM 인스턴스의 수를 고객 요구 또는 정의된 일정에 따라 자동으로 조정할 수 있습니다. 자세한 내용은 [가상 머신 확장 집합을 사용한 자동 크기 조정 개요][vmssautoscale-docs]를 참조하세요.

다른 확장성 항목은 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.

### <a name="security"></a>보안

프론트 엔드 응용 프로그램 계층으로의 모든 가상 네트워크 트래픽은 네트워크 보안 그룹으로 보호됩니다. 규칙은 프런트 엔드 응용 프로그램 계층 VM 인스턴스만 백 엔드 데이터베이스 계층에 액세스할 수 있도록 트래픽 흐름을 제한합니다. 데이터베이스 계층에는 아웃바운드 인터넷 트래픽이 허용되지 않습니다. 공격 공간을 줄이기 위해 직접 원격 관리 포트가 열려 있지 않습니다. 자세한 내용은 [Azure 네트워크 보안 그룹][nsg-docs]을 참조하세요.

PCI DSS(지불 카드 산업 데이터 보안 표준) 3.2 규정 준수 인프라 배포에 대한 지침을 보려면 [규정 준수 인프라][pci-dss]를 참조하세요. 보안 시나리오 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.

### <a name="resiliency"></a>복원력

이 시나리오에서는 가용성 영역 및 가상 머신 확장 집합을 사용할 뿐만 아니라 Azure Application Gateway 및 부하 분산 장치도 사용합니다. 이러한 두 네트워킹 구성 요소는 연결된 VM 인스턴스에 트래픽을 분산시키고, 트래픽이 정상 VM에만 분산되도록 하는 상태 프로브를 포함합니다. 두 Application Gateway 인스턴스가 활성-수동 구성으로 구성되고, 영역 중복 부하 분산 장치가 사용됩니다. 이 구성을 사용하면 트래픽을 중단시키고 최종 사용자 액세스에 영향을 미칠 수 있는 문제로부터 네트워킹 리소스와 응용 프로그램을 탄력적으로 복원할 수 있습니다.

복원력 있는 시나리오 설계에 대한 일반적인 지침은 [Azure용 복원 응용 프로그램 디자인][resiliency]을 참조하세요.

## <a name="deploy-the-scenario"></a>시나리오 배포

**필수 조건.**

* 기존 Azure 계정이 있어야 합니다. Azure 구독이 아직 없는 경우 시작하기 전에 [무료 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.
* SQL Server 클러스터를 백 엔드 확장 집합에 배포하려면 Active Directory 디렉터리 서비스 도메인이 필요합니다.

Azure Resource Manager 템플릿을 사용하여 이 시나리오에 대한 핵심 인프라를 배포하려면 다음 단계를 수행합니다.

1. **Azure에 배포** 단추를 선택합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Finfrastructure%2Fregulated-multitier-app%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure Portal에서 템플릿 배포가 열릴 때까지 기다린 후에 다음 단계를 수행합니다.
   * 리소스 그룹 **새로 만들기**를 선택한 다음, 텍스트 상자에서 이름(예: *myWindowsscenario*)을 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * 가상 머신 확장 집합 인스턴스에 대한 사용자 이름과 보안 암호를 제공합니다.
   * 사용 약관을 검토한 다음, **위에 명시된 사용 약관에 동의함**을 선택합니다.
   * **구매** 단추를 선택합니다.

배포가 완료되는 데 15-20분이 걸릴 수 있습니다.

## <a name="pricing"></a>가격

이 시나리오를 실행하는 데 들어가는 비용을 알아보기 위해 모든 서비스가 비용 계산기에서 미리 구성됩니다.  특정 사용 사례에 대한 가격이 변경되는 정도를 확인하려면 필요한 트래픽에 맞게 적절한 변수를 변경합니다.

응용 프로그램을 실행하는 확장 집합 VM 인스턴스의 수를 기준으로 제공한 세 가지 샘플 비용 프로필은 다음과 같습니다.

* [소형][small-pricing]: 2개 프론트 엔드 및 2개 백 엔드 VM 인스턴스와 관련이 있습니다.
* [중형][medium-pricing]: 20개 프론트 엔드 및 5개 백 엔드 VM 인스턴스와 관련이 있습니다.
* [대형][large-pricing]: 100개 프론트 엔드 및 10개 백 엔드 VM 인스턴스와 관련이 있습니다.

## <a name="related-resources"></a>관련 리소스

이 시나리오에서는 Microsoft SQL Server 클러스터를 실행하는 백 엔드 가상 머신 확장 집합을 사용했습니다. 또한 Azure Cosmos DB도 응용 프로그램 데이터에 대한 확장 가능하고 안전한 데이터베이스 계층으로 사용할 수 있습니다. [Azure 가상 네트워크 서비스 엔드포인트][vnetendpoint-docs]를 사용하면 중요한 Azure 서비스 리소스를 가상 네트워크에서만 보호할 수 있습니다. 이 시나리오에서는 VNet 엔드포인트를 통해 프런트 엔드 응용 프로그램 계층과 Cosmos DB 간의 트래픽을 보호할 수 있습니다. Cosmos DB에 대한 자세한 내용은 [Azure Cosmos DB 개요][azurecosmosdb-docs]를 참조하세요.

완전한 [SQL Server를 통한 일반 N 계층 응용 프로그램에 대한 참조 아키텍처][ntiersql-ra]도 참조하세요.

<!-- links -->
[appgateway-docs]: /azure/application-gateway/overview
[architecture]: ./media/regulated-multitier-app/architecture-regulated-multitier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: /architecture/checklist/availability
[azureaz-docs]: /azure/availability-zones/az-overview
[azurecosmosdb-docs]: /azure/cosmos-db/introduction
[cloudwitness-docs]: /windows-server/failover-clustering/deploy-cloud-witness
[loadbalancer-docs]: /azure/load-balancer/load-balancer-overview
[nsg-docs]: /azure/virtual-network/security-overview
[ntiersql-ra]: /azure/architecture/reference-architectures/n-tier/n-tier-sql-server
[resiliency]: /azure/architecture/resiliency/ 
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability 
[scaleset-docs]: /azure/virtual-machine-scale-sets/overview
[sqlalwayson-docs]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[vmssautoscale-docs]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[vnet-docs]: /azure/virtual-network/virtual-networks-overview
[vnetendpoint-docs]: /azure/virtual-network/virtual-network-service-endpoints-overview
[pci-dss]: /azure/security/blueprints/pcidss-iaaswa-overview
[dmz]: /azure/virtual-network/virtual-networks-dmz-nsg
[cosmos]: /azure/cosmos-db/
[sql-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017

[small-pricing]: https://azure.com/e/711bbfcbbc884ef8aa91cdf0f2caff72
[medium-pricing]: https://azure.com/e/b622d82d79b34b8398c4bce35477856f
[large-pricing]: https://azure.com/e/1d99d8b92f90496787abecffa1473a93