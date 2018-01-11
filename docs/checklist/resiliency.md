---
title: "복원력 검사 목록"
description: "설계하는 동안 복원력 문제에 대한 지침을 제공하는 검사 목록입니다."
author: petertaylor9999
ms.date: 03/24/2017
ms.custom: resiliency, checklist
ms.openlocfilehash: d0ae48be603391a7be043130d6d77d3c82abad18
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="resiliency-checklist"></a>복원력 검사 목록
[!INCLUDE [header](../_includes/header.md)]

응용 프로그램을 복원력에 적합하게 설계하려면 발생할 수 있는 다양한 실패 모드를 계획하고 완화해야 합니다. 복원력을 개선하려면 이 검사 목록의 항목을 응용 프로그램 설계에 비추어 검토해 보세요.

## <a name="requirements"></a>요구 사항
* **고객의 가용성 요구 사항을 정의합니다.** 고객은 응용 프로그램의 구성 요소에 대한 가용성 요구 사항을 가지고 있으며 이는 응용 프로그램의 설계에 영향을 줍니다. 응용 프로그램 각 부분의 가용성 목표에 대해 고객의 동의를 받으십시오. 그렇지 않으면 귀하의 설계가 고객의 기대를 충족하지 못할 수 있습니다. 자세한 내용은 [복원력 요구 사항 정의](../resiliency/index.md#defining-your-resiliency-requirements)를 참조하세요.

## <a name="application-design"></a>응용 프로그램 설계
* **응용 프로그램에 대한 FMA(실패 모드 분석)를 수행합니다.** FMA는 설계 단계의 조기에 응용 프로그램에 복원력을 구축하기 위한 프로세스입니다. 자세한 내용은 [실패 모드 분석][fma]을 참조하세요. FMA의 목표는 다음과 같습니다.  

  * 응용 프로그램에서 발생할 수 있는 고장의 유형을 식별합니다.
  * 각 고장 유형의 잠재적 효과 및 응용 프로그램에 주는 영향을 파악합니다.
  * 복구 전략을 식별합니다.
  
* **서비스의 여러 인스턴스를 배포합니다.** 응용 프로그램이 서비스의 단일 인스턴스에 종속된 경우 단일 실패 지점이 생깁니다. 여러 인스턴스를 프로비전하면 복원력 및 확장성이 모두 개선됩니다. [Azure App Service](/azure/app-service/app-service-value-prop-what-is/)의 경우 여러 인스턴스를 제공하는 [App Service 계획](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)을 선택합니다. Azure Cloud Services의 경우 각각의 역할을 [여러 인스턴스](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management)를 사용하도록 구성합니다. [Azure Virtual Machines(VM)](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)의 경우, VM 아키텍처가 둘 이상의 VM을 포함하는지 그리고 각각의 VM이 [가용성 집합][availability-sets]에 포함되는지 확인합니다.   
* **자동 크기 조정을 사용하여 부하의 증가에 대응합니다.** 응용 프로그램이 부하가 증가할 때 자동으로 규모 확장되도록 구성되지 않은 경우 사용자 요청으로 포화되면 응용 프로그램의 서비스가 실패할 가능성이 있습니다. 자세한 내용은 다음을 참조하세요.

  * 일반 사항: [확장성 검사 목록](./scalability.md)
  * Azure App Service: [인스턴스 개수를 수동 또는 자동으로 크기 조정][app-service-autoscale]
  * Cloud Services: [클라우드 서비스 크기를 자동으로 조정하는 방법][cloud-service-autoscale]
  * Virtual Machines: [자동 크기 조정 및 Virtual Machine Scale Sets][vmss-autoscale]
* **부하 분산을 사용하여 요청을 분산합니다.** 부하 분산은 비정상 인스턴스를 윤번에서 제거하여 응용 프로그램의 요청을 정상 서비스 인스턴스에 분산합니다. 서비스에 Azure App Service 또는 Azure Cloud Services를 사용하는 경우 이미 부하 분산이 적용됩니다. 그러나 응용 프로그램이 Azure VM을 사용하는 경우 부하 분산 장치를 프로비전해야 합니다. 자세한 내용은 [Azure Load Balancer](/azure/load-balancer/load-balancer-overview/) 개요를 참조하세요.
* **여러 인스턴스를 사용하도록 Azure Application Gateway를 구성 합니다.** 응용 프로그램의 요구 사항에 따라 [Azure Application Gateway](/azure/application-gateway/application-gateway-introduction/)가 요청을 응용 프로그램의 서비스에 배포하기에 더 적합할 수 있습니다. 그러나 응용 프로그램 게이트웨이 서비스의 단일 인스턴스는 SLA에 의해 보증되지 않으므로 응용 프로그램 게이트웨이 인스턴스가 실패하는 경우 응용 프로그램이 실패할 수 있습니다. [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/)의 조항에 따라 서비스의 가용성을 보증하려면 보통 이상의 응용 프로그램 게이트웨이 인스턴스를 두 개 이상 프로비전합니다.
* **각 응용 프로그램 계층에 대해 가용성 집합을 사용합니다.** 인스턴스를 [가용성 집합][availability-sets]에 배치하면 더 높은 [SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/)를 제공합니다. 
* **여러 지역에 걸쳐 응용 프로그램을 배포하는 것을 고려합니다.** 응용 프로그램이 단일 지역에 배포되면 전체 지역이 사용할 수 없게 되는 드문 경우에 응용 프로그램도 사용할 수 없습니다. 이러한 상황이 응용 프로그램의 SLA의 조항에 따라 허용되지 않을 수 있습니다. 그러한 경우 응용 프로그램 및 해당 서비스를 여러 지역에 걸쳐 배포하는 것을 고려합니다. 다중 지역 배포는 능동-능동 패턴(여러 활성 인스턴스에 걸쳐 요청을 배포) 또는 능동-수동 패턴(기본 인스턴스가 실패할 경우 “관심 있음” 인스턴스를 예비로 보관)을 사용할 수 있습니다. 응용 프로그램 서비스의 여러 인스턴스를 지역 쌍에 걸쳐 배포하는 것이 좋습니다. 자세한 내용은 [BCDR(비즈니스 연속성 및 재해 복구): Azure 쌍을 이루는 지역](/azure/best-practices-availability-paired-regions)을 참조하세요.
* **Azure Traffic Manager를 사용하여 응용 프로그램의 트래픽을 다른 지역으로 경로 설정합니다.**  [Azure Traffic Manager][traffic-manager]는 DNS 수준에서 부하 분산을 수행하며 트래픽을 사용자가 지정하는 [트래픽 라우팅][traffic-manager-routing] 방법 및 응용 프로그램 끝점의 상태를 기반으로 서로 다른 지역에 경로 설정합니다. Traffic Manager가 없으면 배포가 단일 지역으로 제한되어 크기가 제한되고 일부 사용자에 대한 대기 시간이 증가하며, 지역 전체 서비스가 중단된 경우 응용 프로그램 가동 중지 시간이 야기됩니다.
* **부하 분산 장치 및 트래픽 관리자에 대한 상태 프로브를 구성하고 테스트합니다** 상태 논리가 시스템의 중요 부분을 확인하고 상태 프로브에 적절히 반응하도록 합니다.
  * [Azure Traffic Manager][traffic-manager] 및 [Azure Load Balancer][load-balancer]에 대한 상태 프로브는 특정 기능을 수행합니다. Traffic Manager의 경우 상태 프로브는 다른 지역으로 장애 조치할지 여부를 결정합니다. 부하 분산 장치의 경우 VM을 윤번에서 제거할지 여부를 결정합니다.      
  * Traffic Manager 프로브의 경우, 상태 끝점이 같은 지역 내에 배포된 중요 종속성을 확인해야 하며, 해당 끝점이 실패하면 다른 지역으로 장애 조치를 트리거해야 합니다.  
  * 부하 분산 장치의 경우 상태 끝점은 VM의 상태를 보고해야 합니다. 다른 계층 또는 외부 서비스를 포함하지 마세요. 그렇지 않으면 VM 외부에서 발생하는 실패 때문에 부하 분산 장치가 VM을 윤번에서 제거하게 됩니다.
  * 응용 프로그램의 상태 모니터링 구현에 대한 지침은 [상태 끝점 모니터링 패턴](https://msdn.microsoft.com/library/dn589789.aspx)을 참조하세요.
* **타사 서비스를 모니터링합니다.** 응용 프로그램이 타사 서비스에 종속된 경우 이러한 타사 서비스가 실패할 수 있는 경우와 방법 및 해당 실패가 응용 프로그램에 주는 영향을 파악합니다. 타사 서비스는 모니터링 및 진단을 포함하지 않은 경우가 있으므로 이러한 서비스의 호출을 기록하고 고유 식별자를 사용하여 응용 프로그램의 상태 및 진단 로깅과 상호 연결하는 것이 중요합니다. 입증된 모니터링 및 진단 사례에 대한 자세한 내용은 [모니터링 및 진단 지침][monitoring-and-diagnostics-guidance]을 참조하세요.
* **사용하는 타사 서비스가 SLA를 제공하는지 확인합니다.** 응용 프로그램이 타사 서비스에 의존하지만 해당 타사가 SLA 형태로 가용성에 대한 보증을 제공하지 않는 경우 응용 프로그램의 가용성도 보증할 수 없습니다. SLA는 응용 프로그램의 가장 가용성이 낮은 구성 요소만큼만 품질이 보장될 뿐입니다.
* **적절한 경우 원격 작업에 대한 복원력 패턴을 구현합니다.** 응용 프로그램이 원격 서비스 간의 통신에 의존하는 경우 [재시도 패턴][retry-pattern] 및 [회로 차단기 패턴][circuit-breaker] 등 일시적 고장을 처리하기 위한 설계 패턴을 따릅니다. 자세한 내용은 [복원력 전략](../resiliency/index.md#resiliency-strategies)을 참조하세요.
* **가능하면 언제나 비동기 작업을 구현합니다.** 동기 작업은 프로세스가 완료될 때까지 호출자가 대기하는 동안 리소스를 독점하고 다른 작업을 차단할 수 있습니다. 가능하면 언제나 응용 프로그램의 각 부분을 비동기 작업이 가능하도록 설계합니다. C#에서 비동기 프로그래밍을 구현하는 자세한 방법은 [async 및 await를 사용한 비동기 프로그래밍][asynchronous-c-sharp]을 참조하세요.

## <a name="data-management"></a>데이터 관리
* **응용 프로그램의 데이터 원본에 대한 복제 방법을 이해합니다.** 응용 프로그램 데이터가 서로 다른 데이터 원본에 저장되고 가용성 요구 사항도 서로 다릅니다. [Azure Storage 복제](/azure/storage/storage-redundancy/) 및 [SQL Database 활성 지역 복제](/azure/sql-database/sql-database-geo-replication-overview/)를 포함하여 Azure에서 각 유형의 데이터 저장장치에 대한 복제 방법을 평가하여 응용 프로그램의 데이터 요구 사항이 충족되는지 확인합니다.
* **프로덕션 및 백업 데이터 둘 다에 대해 액세스 권한을 가진 단일 사용자 계정이 없는지 확인합니다.** 단일 사용자 계정 하나가 프로덕션 및 백업 원본 둘 다에 대한 쓰기 권한을 가진 경우 데이터 백업이 손상됩니다. 악의적인 사용자는 모든 데이터를 고의로 삭제할 수 있는 반면, 정규 사용자는 해당 데이터를 실수로 삭제할 수 있습니다. 쓰기 액세스를 요구하는 사용자만이 쓰기 액세스 권한을 갖고 이 권한이 프로덕션 또는 백업에만 적용되고 둘 다에 함께 적용되지는 않도록 하려면 응용 프로그램을 각 사용자 계정의 사용 권한을 제한하도록 설계합니다.
* **데이터 원본 장애 조치 및 장애 복구 프로세스를 문서화하고 테스트합니다.** 데이터 원본이 치명적으로 실패한 경우 운영자가 문서화된 지침 집합에 따라 새 데이터 원본으로 장애 조치해야 합니다. 문서화된 단계에 오류가 있는 경우 운영자는 해당 단계를 성공적으로 따르고 리소스를 장애 조치할 수 없습니다. 정기적으로 지침 단계를 테스트하면 해당 지침을 따르는 운영자가 데이터 원본을 성공적으로 장애 조치 및 장애 복구할 수 있습니다.
* **데이터 백업 유효성을 검사합니다.** 정기적으로 스크립트를 실행하여 데이터 무결성, 스키마 및 쿼리의 유효성을 검사함으로써 백업 데이터가 예상한 것인지 확인합니다. 해당 데이터가 데이터 원본을 복원하는 데 유용하지 않으면 백업을 포함한 지점이 없는 것입니다. 백업 서비스를 수리할 수 있도록 모순점을 기록하고 보고합니다.
* **지리적으로 중복된 저장소 계정 유형의 사용을 고려합니다.** Azure 저장소 계정에 저장된 데이터는 항상 로컬로 복제됩니다. 그러나 저장소 계정이 프로비전될 때 선택할 수 있는 여러 복제 전략이 있습니다. 전체 지역이 사용할 수 없게 되는 드문 경우 응용 프로그램 데이터를 보호하려면 [RA-GRS(Azure 읽기 액세스 지역 중복 저장소)](/azure/storage/storage-redundancy/#read-access-geo-redundant-storage)를 선택합니다.

  > [!NOTE]
  > VM의 경우 VM 디스크(VHD 파일)를 복원하기 위해 RA-GRS 복제에 의존하지 마세요. 그 대신 [Azure Backup][azure-backup]을 사용하세요.   
  >
  >

## <a name="security"></a>보안
* **DDoS(배포된 서비스 거부) 공격에 대한 응용 프로그램 수준의 보호를 구현합니다.** Azure 서비스는 네트워크 계층에서 DDoS 공격으로부터 보호됩니다. 그러나 실제 사용자 요청을 악의적인 사용자 요청과 구별하기 어려우므로 Azure는 응용 프로그램 계층 공격으로부터 보호할 수 없습니다. 응용 프로그램 계층 DDoS 공격으로부터 보호하는 방법은 [Microsoft Azure 네트워크 보안](http://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf)(PDF 다운로드)의 “DDoS로부터 보호” 섹션을 참조하세요.
* **응용 프로그램의 리소스에 액세스하기 위한 최소 권한의 원칙을 구현합니다.** 응용 프로그램의 리소스에 액세스하기 위한 기본값은 가능하면 제한적이어야 합니다. 승인을 받은 경우 더 높은 권한을 부여합니다. 응용 프로그램의 리소스에 대해 과도하게 허용된 액세스 권한을 부여하면 누군가가 고의로 또는 실수로 리소스를 삭제하게 될 수 있습니다. Azure는 사용자 권한을 관리하기 위한 [역할 기반 액세스 제어](/azure/active-directory/role-based-access-built-in-roles/)를 제공하지만, SQL Server 같은 자체의 권한 시스템을 가진 다른 리소스에 대해서는 최소 권한 허용을 확인하는 것이 중요합니다.

## <a name="testing"></a>테스트
* **응용 프로그램에 대한 장애 조치 및 장애 복구 테스트를 수행합니다.** 장애 조치 및 장애 복구를 완전하게 테스트하지 않은 경우 응용 프로그램의 의존 서비스가 재해 복구 중에 동기화된 방법으로 백업된다고 확신할 수 없습니다. 응용 프로그램의 종속 서비스가 장애 조치 및 장애 복구를 올바른 순서로 수행하는지 확인합니다.
* **응용 프로그램에 대한 오류-삽입 테스트를 수행합니다.** 응용 프로그램은 인증서 만료, VM의 시스템 리소스 소모, 저장소 고장 등 많은 다른 이유로 실패할 수 있습니다. 실제 고장을 시뮬레이션 또는 트리거하여 프로덕션에 가능하면 근접한 환경에서 응용 프로그램을 테스트합니다. 예를 들어 인증서를 삭제하거나 인위적으로 시스템 리소스를 사용하거나 저장소 원본을 삭제합니다. 단독 및 조합 등 모든 유형의 결함에서 복구하는 응용 프로그램의 기능을 확인합니다. 오류가 시스템을 통해 전파되거나 종속 연결되지 않는지 확인합니다.
* **가상 및 실제 사용자 데이터를 모두 사용하여 프로덕션에서 테스트를 실행합니다.** 테스트 및 프로덕션은 동일한 경우가 드물기 때문에 파란색/녹색 또는 카나리아 배포를 사용하고 프로덕션에서 응용 프로그램을 테스트하는 것이 중요합니다. 이렇게 하면 실제 부하의 프로덕션에서 응용 프로그램을 테스트하고 완전히 배포되었을 때 예상한 대로 기능하는지 확인할 수 있습니다.

## <a name="deployment"></a>배포
* **응용 프로그램에 대한 릴리스 프로세스를 문서화합니다.** 자세한 릴리스 프로세스 문서가 없으면 운영자가 잘못된 업데이트를 배포하거나 응용 프로그램에 대한 설정을 부적절하게 구성할 수 있습니다. 릴리스 프로세스를 명확히 정의 및 문서화하고 전체 작업 팀에 사용할 수 있도록 했는지 확인합니다. 
* **응용 프로그램의 배포 프로세스를 자동화합니다.** 운영 담당자가 응용 프로그램을 수동으로 배포해야 하는 경우 인적 오류 때문에 배포가 실패할 수 있습니다. 
* **응용 프로그램 가용성을 최대화하도록 릴리스 프로세스를 설계합니다.** 릴리스 프로세스가 배포 중에 서비스를 오프라인으로 전환할 것을 요구하는 경우 다시 온라인이 될 때까지 응용 프로그램을 사용할 수 없습니다. [파란색/녹색](http://martinfowler.com/bliki/BlueGreenDeployment.html) 또는 [카나리아 릴리스](http://martinfowler.com/bliki/CanaryRelease.html) 배포 기술을 사용하여 응용 프로그램을 프로덕션에 배포합니다. 두 기술 모두 고장 시 릴리스 코드 사용자가 프로덕션 코드로 리디렉션될 수 있도록 프로덕션 코드와 함께 릴리스 코드 배포를 포함합니다.
* **로그인하고 응용 프로그램의 배포를 감사합니다.** 파란색/녹색 또는 카나리아 릴리스 같은 단계별 배포 기술을 사용하는 경우 프로덕션에서 실행 중인 응용 프로그램의 버전이 둘 이상이 됩니다. 문제가 발생한 경우 응용 프로그램의 버전 중 문제를 야기하는 버전을 결정해야 합니다. 최대한 많은 버전 관련 정보를 캡처하도록 강력한 로깅 전략을 구현합니다.
* **배포에 대한 롤백 계획을 수립합니다.** 응용 프로그램 배포가 실패하여 응용 프로그램이 사용할 수 없게 될 가능성이 있습니다. 마지막으로 성공한 버전으로 돌아가서 가동 중지 시간을 최소화하도록 롤백 프로세스를 설계합니다. 

## <a name="operations"></a>작업
* **응용 프로그램을 모니터링 및 경고하기 위한 모범 사례를 구현합니다.** 적절한 모니터링, 진단 및 경고가 없으면 응용 프로그램의 실패를 탐지하고 운영자에게 수정하도록 경고할 방법이 없습니다. 자세한 내용은 [모니터링 및 진단 지침][monitoring-and-diagnostics-guidance]을 참조하세요.
* **원격 호출 통계를 측정하고 이 정보를 응용 프로그램 팀에 사용할 수 있도록 합니다.**  원격 호출 통계를 실시간으로 추적하여 보고하고 이 정보를 검토하는 쉬운 방법을 제공하지 않으면 운영 팀이 응용 프로그램의 순간적인 상태를 확인할 수 없습니다. 그리고 평균 원격 호출 시간만 측정하면 서비스의 문제를 드러낼 만큼 충분한 정보를 확보하지 못합니다. 대기 시간, 처리량 및 99 및 95 백분위 수의 오류와 같은 원격 호출 메트릭을 요약합니다. 각 백분위 수 내에서 발생하는 오류를 발견하는 메트릭에 대해 통계 분석을 수행합니다.
* **일시적 예외 수를 추적하고 적절한 시간 범위에 대해 재시도합니다.** 시간 범위에 대한 일시적 예외 및 재시도 횟수를 추적 및 모니터링하지 않으면 응용 프로그램의 재시도 논리에 의해 문제 또는 실패가 표시되지 않을 가능성이 있습니다. 즉, 모니터링 및 로깅이 작업의 성공 또는 실패를 표시하면 작업이 예외 때문에 여러 번 재시도되었다는 사실이 표시되지 않을 것입니다. 시간이 지남에 따라 예외가 증가하는 추세는 서비스에 문제가 있어 서비스가 실패할 수 있다는 것을 나타냅니다. 자세한 내용은 [서비스 관련 재시도 지침][retry-service-guidance]을 참조하세요.
* **운영자에게 알려 주는 조기 경고 시스템을 구현합니다.** 일시적 예외 및 원격 호출 대기 시간 등 응용 프로그램 상태의 주요 성능 지표를 식별하고 각각의 지표에 대해 적절한 임계값을 설정합니다. 임계값에 도달할 때 작업에 경고를 보냅니다. 이러한 임계값을 문제가 심각해져서 복구 응답이 필요하게 되기 전에 식별하는 수준으로 설정합니다.
* **팀의 두 명 이상이 응용 프로그램 모니터링 및 수동 복구 단계의 수행에 대해 교육을 받도록 합니다.** 팀에 응용 프로그램을 모니터링하고 복구 단계를 시작할 수 있는 운영자가 한 명뿐이면 바로 그 운영자가 단일 실패 지점이 됩니다. 검색 및 복구에 대해 여러 개인을 교육하고 어느 때라도 최소한 한 명이 확보되도록 합니다.
* **[Azure 구독 제한](/azure/azure-subscription-service-limits/)을 기준으로 응용 프로그램이 그보다 많이 실행되지 않도록 합니다.** Azure 구독은 리소스 그룹 수, 코어 수 및 저장소 계정 수 등 특정 리소스 유형에 대한 제한을 둡니다.  응용 프로그램 요구 사항이 Azure 구독 제한을 초과하는 경우 또 다른 Azure 구독을 만들고 거기에 충분한 리소스를 프로비전합니다.
* **[서비스 단위 제한](/azure/azure-subscription-service-limits/)을 기준으로 응용 프로그램이 그보다 많이 실행되지 않도록 합니다.** 개별 Azure 서비스는 사용 제한을 둡니다. &mdash; 예들 들어 저장소, 처리량, 연결 수, 초당 요청 수 및 기타 메트릭에 대한 제한을 둡니다. 응용 프로그램은 이러한 제한을 초과하여 리소스 사용을 시도하면 실패합니다. 그러면 서비스가 제한되며 영향을 받는 사용자에 대해 가동 중지 시간이 발생할 가능성이 있습니다. 특정 서비스 및 응용 프로그램 요구 사항에 따라 많은 경우 확장(예: 다른 가격 계층 선택) 또는 규모 확장(새 인스턴스 추가)에 의해 이러한 제한을 피할 수 있습니다.  
* **응용 프로그램의 저장 요구 사항을 Azure 저장 확장성 및 성능 목표 범위 이내가 되도록 설계합니다.** Azure 저장소는 미리 정의된 확장성 및 성능 목표 내에서 작동하도록 설계되었으므로 응용 프로그램이 저장소를 해당 목표 범위 이내에서 이용하게 됩니다. 이러한 목표를 초과하면 응용 프로그램 저장소 제한이 발생합니다. 이 문제를 해결하려면 추가 저장소 계정을 프로비전합니다. 저장소 계정 제한보다 많이 실행한 경우 추가 Azure 구독을 프로비전한 다음 거기서 추가 저장소 계정을 프로비전합니다. 자세한 내용은 [Azure Storage 확장성 및 성능 목표](/azure/storage/storage-scalability-targets/)를 참조하세요.
* **응용 프로그램에 대한 올바른 VM 크기를 선택합니다.** 프로덕션에 있는 VM의 실제 CPU, 메모리, 디스크 및 I/O를 측정하고 선택한 VM 크기가 충분한지 확인합니다. 그렇지 않은 경우 VM이 제한에 근접하면 응용 프로그램 용량 문제가 발생할 수 있습니다. VM 크기는 [Azure의 가상 머신에 대한 크기](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)에서 자세히 설명합니다.
* **응용 프로그램의 워크로드가 안정적인지 또는 시간이 지남에 따라 변동하는지 결정합니다.** 워크로드가 시간이 지남에 따라 변동하는 경우 Azure VM 확장 집합을 사용하여 VM 인스턴스 수를 자동으로 크기 조정합니다. 그렇지 않으면 VM 수를 수동으로 늘리거나 줄여야 합니다. 자세한 내용은 [가상 머신 확장 집합 개요](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/)를 참조하세요.
* **Azure SQL Database에 적합한 서비스 계층을 선택합니다.** 응용 프로그램이 Azure SQL Database를 사용하는 경우 적절한 서비스 계층을 선택했는지 확인합니다. 응용 프로그램의 DTU(데이터베이스 트랜잭션 단위) 요구 사항을 처리할 수 없는 계층을 선택하면 데이터 사용이 제한을 받습니다. 올바른 서비스 계획 선택에 대한 자세한 내용은 [SQL Database 옵션 및 성능: 각 서비스 계층에서 사용할 수 있는 것 이해](/azure/sql-database/sql-database-service-tiers/)를 참조하세요.
* **Azure 지원 담당자와 상호 작용하는 프로세스를 만듭니다.** 지원 담당자와 연락할 필요가 발생하기 전에 [Azure 지원 담당자](https://azure.microsoft.com/support/plans/)와 연락하기 위한 프로세스를 설정하지 않으면 지원 프로세스를 처음 탐색하는 동안 가동 중지 시간이 길어집니다. 처음부터 응용 프로그램 복원력의 일부로 지원 담당자와 연락 및 문제 에스컬레이션에 대한 프로세스를 포함합니다.
* **응용 프로그램이 구독당 최대 저장소 계정 수보다 많이 사용하지 않도록 합니다.** Azure는 구독당 저장소 계정 수를 최대 200까지 허용합니다. 응용 프로그램에 현재 구독에서 사용할 수 있는 것보다 많은 저장소 계정이 필요한 경우 새 구독을 만들고 거기서 추가 저장소 계정을 만듭니다. 자세한 내용은 [Azure 구독 및 서비스 제한, 할당량 및 제약 조건](/azure/azure-subscription-service-limits/#storage-limits)을 참조하세요.
* **응용 프로그램이 가상 머신 디스크에 대한 확장성 목표를 초과하지 않도록 합니다.** Azure IaaS VM은 VM 크기 및 저장소 계정의 유형을 포함하여 다양한 요인에 따라 데이터 디스크 수의 연결을 지원합니다. 응용 프로그램이 가상 머신 디스크에 대한 확장성 목표를 초과하는 경우 추가 저장소 계정을 프로비전하고 거기서 가상 머신 디스크를 만듭니다. 자세한 내용은 [Azure Storage 확장성 및 성능 목표](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks) 참조

## <a name="telemetry"></a>원격 분석
* **응용 프로그램이 프로덕션 환경에서 실행하는 동안 원격 분석 데이터를 기록합니다.** 응용 프로그램이 프로덕션 환경에서 실행하는 동안 강력한 원격 분석 정보를 캡처합니다. 그렇지 않으면 응용 프로그램이 사용자에게 서비스하는 동안 문제의 원인을 진단하기에 충분한 정보를 확보하지 못합니다. 자세한 내용은 [모니터링 및 진단][monitoring-and-diagnostics-guidance]을 참조하세요.
* **비동기 패턴을 사용하여 로깅을 구현합니다.** 로깅 작업들이 동기화되면 응용 프로그램 코드를 차단할 수 있습니다. 로깅 작업이 비동기 작업으로 구현되도록 합니다.
* **서비스 경계에 걸쳐 로그 데이터를 상호 연결합니다.** 일반적인 n계층 응용 프로그램에서 사용자 요청은 여러 서비스 경계를 횡단할 수 있습니다. 예를 들어 사용자 요청은 일반적으로 웹 계층에서 시작하여 비즈니스 계층에 전달되며 마지막으로 데이터 계층에 유지됩니다. 더 복잡한 시나리오에서 사용자 요청은 여러 다른 서비스 및 데이터 저장소에 배포될 수 있습니다. 응용 프로그램 전체에 걸쳐 요청을 추적할 수 있도록 로깅 시스템이 서비스 경계에 걸쳐 호출을 상호 연결하도록 합니다.

## <a name="azure-resources"></a>Azure 리소스
* **Azure Resource Manager 템플릿을 사용하여 리소스를 프로비전합니다.** Resource Manager 템플릿을 사용하면 PowerShell 또는 Azure CLI를 통해 배포를 자동화하여 배포 프로세스를 더 신뢰성 있게 만들기 쉬워집니다. 자세한 내용은 [Azure Resource Manager 개요][resource-manager]를 참조하세요.
* **리소스에 의미 있는 이름을 지정합니다.** 리소스에 의미 있는 이름을 지정하면 쉽게 특정 리소스를 찾고 역할을 이해할 수 있습니다. 자세한 내용은 [Azure 리소스의 명명 규칙](../best-practices/naming-conventions.md)을 참조하세요.
* **RBAC(역할 기반 액세스 제어)를 사용합니다.** RBAC를 사용하여 배포하는 Azure 리소스에 대한 액세스를 제어합니다. RBAC를 사용하면 DevOps 팀의 팀원에게 권한 부여 역할을 할당하여 배포된 리소스에 대한 우발적 삭제 또는 변경을 방지할 수 있습니다. 자세한 내용은 [Azure Portal에서 액세스 관리 시작](/azure/active-directory/role-based-access-control-what-is/)을 참조하세요.
* **VM 같은 중요 리소스에 대해 리소스 잠금을 사용합니다.** 리소스 잠금은 운영자가 리소스를 실수로 삭제하는 것을 예방합니다. 자세한 내용은 [Azure Resource Manager를 사용하여 리소스 잠그기](/azure/azure-resource-manager/resource-group-lock-resources/) 참조
* **지역 쌍을 선택합니다.** 두 지역을 배포하는 경우 같은 지역 쌍에서 지역을 선택합니다. 광범위한 중단 발생 시 한 지역의 복구가 모든 쌍의 복구보다 높은 우선 순위를 갖습니다. 지역 정보 저장소 같은 일부 서비스는 쌍을 이루는 지역에 대해 자동 복제를 제공합니다. 자세한 내용은 [BCDR(비즈니스 연속성 및 재해 복구): Azure 쌍을 이루는 지역](/azure/best-practices-availability-paired-regions) 참조
* **기능 및 수명 주기별로 리소스 그룹을 구성합니다.**  일반적으로 리소스 그룹은 동일한 수명 주기를 공유하는 리소스를 포함해야 합니다. 이렇게 하면 배포 관리, 테스트 배포 삭제 및 액세스 권한 할당을 보다 쉽게 관리할 수 있어 프로덕션 배포가 실수로 삭제되거나 수정될 기회가 감소합니다. 프로덕션, 개발 및 테스트 환경에 대해 별도의 리소스 그룹을 만듭니다. 다중 지역 배포에서는 각 지역에 대한 리소스를 별도의 리소스 그룹에 배치합니다. 이렇게 하면 다른 지역에 영향을 주지 않고 한 지역을 재배포하기 쉽습니다.

## <a name="azure-services"></a>Azure 서비스
다음 검사 목록 항목은 Azure의 특정 서비스에 적용됩니다.

### <a name="app-service"></a>App Service
* **표준 또는 프리미엄 계층을 사용합니다.** 이러한 계층은 스테이징 슬롯 및 자동화된 백업을 지원합니다. 자세한 내용은 [Azure App Service 계획 심층 개요](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)를 참조하세요.
* **확장 또는 축소를 피합니다.** 그 대신에 일반적인 부하에서 성능 요구 사항에 맞는 계층 및 인스턴스 크기를 선택한 다음 [규모 확장](/azure/app-service-web/web-sites-scale/)하여 트래픽 분량의 변화를 처리합니다. 확장 및 축소는 응용 프로그램 다시 시작을 트리거할 수 있습니다.  
* **구성을 앱 설정으로 저장합니다.** 앱 설정을 사용하여 구성 설정을 앱 설정으로 저장합니다. Resource Manager 템플릿에 설정을 정의하거나 PowerShell을 사용하여 해당 설정을 자동화된 배포 / 업데이트 프로세스의 일부로 적용할 수 있도록 합니다(이 방법이 신뢰성이 더 높음). 자세한 내용은 [Azure App Service에서 웹앱 구성](/azure/app-service-web/web-sites-configure/)을 참조하세요.
* **프로덕션 및 테스트에 대해 별도의 앱 서비스 계획을 만듭니다.** 테스트를 위한 프로덕션 배포에 슬롯을 사용하지 마십시오.  동일한 App Service 계획 내의 모든 앱은 같은 VM 인스턴스를 공유합니다. 프로덕션과 테스트 배포를 동일한 계획에 배치하면 프로덕션 배포에 부정적 영향을 줄 수 있습니다. 예를 들어 부하 테스트는 실시간 프로덕션 사이트의 성능을 저하시킬 수 있습니다. 테스트 배포를 별도의 계획에 배치하여 프로덕션 버전과 격리합니다.  
* **웹앱을 웹 API와 분리합니다.** 솔루션에 웹 프런트 엔드 및 웹 API가 모두 포함된 경우 이들을 별도의 App Service 앱으로 분해하는 것을 고려합니다. 이 설계를 통해 워크로드별로 솔루션을 분해하기가 더 쉬워집니다. 웹앱과 API를 별도의 App Service 계획에서 실행할 수 있으므로 이들을 독립적으로 크기 조정할 수 있습니다. 처음에 이러한 수준의 확장성이 필요하지 않은 경우 앱을 동일한 계획에 배포하고 필요한 경우 나중에 별도의 계획으로 이동할 수 있습니다.
* **앱 서비스 백업 기능을 사용하여 Azure SQL Database를 백업하는 것을 피하십시오.** 그 대신에 [SQL Database 자동화된 백업][sql-backup]을 사용합니다. App Service 백업은 데이터베이스를 SQL .bacpac 파일에 내보내는데, 이때 DTU 비용이 발생합니다.  
* **스테이징 슬롯에 배포** 스테이징을 위한 배포 슬롯을 만들 수 있습니다. 응용 프로그램 업데이트를 스테이징 슬롯에 배포하고 프로덕션으로 교환하기 전에 배포를 확인합니다. 이렇게 하면 프로덕션에서 잘못된 업데이트의 기회가 감소합니다. 또한 모든 인스턴스가 프로덕션으로 교환되기 전에 확실히 준비되기도 합니다. 많은 응용 프로그램에는 중요한 준비 및 콜드 부팅 시간이 있습니다. 자세한 내용은 [Azure App Service에서 웹앱에 대한 스테이징 환경 설정](/azure/app-service-web/web-sites-staged-publishing/)을 참조하세요.
* **마지막으로 성공한(LKG) 배포를 저장하는 배포 슬롯을 만듭니다.** 업데이트를 프로덕션에 배포하는 경우 이전 프로덕션 배포를 LKG 슬롯으로 이동합니다. 이렇게 하면 잘못된 배포를 롤백하기가 더 쉬워집니다. 나중에 문제가 발견되면 LKG 버전으로 신속하게 되돌릴 수 있습니다. 자세한 내용은 [기본적인 웹 응용 프로그램](../reference-architectures/app-service-web-app/basic-web-app.md)을 참조하세요.
* 응용 프로그램 로깅 및 웹 서버 로깅을 포함하여 **진단 로깅 설정**합니다. 로깅은 모니터링 및 진단을 위해 중요합니다. [Azure App Service에서 웹앱에 대한 진단 로깅 설정](/azure/app-service-web/web-sites-enable-diagnostic-log/) 참조
* **Blob 저장소에 기록합니다.** 이렇게 하면 보다 쉽게 데이터를 수집 및 분석할 수 있습니다.
* **로그에 대한 별도의 저장소 계정을 만듭니다.** 로그와 응용 프로그램 데이터에 동일한 저장소 계정을 사용하지 마세요. 이렇게 하면 로깅이 응용 프로그램의 성능을 감소시키는 것을 방지하는 데 도움이 됩니다.
* **성능을 모니터링합니다.** [New Relic](http://newrelic.com/) 또는 [Application Insights](/azure/application-insights/app-insights-overview/) 같은 성능 모니터링 서비스를 사용하여 응용 프로그램 성능 및 부하를 받을 때의 동작을 모니터링합니다.  성능 모니터링은 응용 프로그램에 대한 실시간 통찰력을 제공합니다. 문제를 진단하고 실패의 근본 원인 분석을 수행할 수 있습니다.

### <a name="application-gateway"></a>응용 프로그램 게이트웨이
* **인스턴스를 두 개 이상 프로비전합니다.** 응용 프로그램 게이트웨이를 두 개 이상의 인스턴스와 함께 배포합니다. 단일 인스턴스는 단일 실패 지점입니다. 중복성 및 확장성을 위해 인스턴스를 두 개 이상 사용합니다. [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/)에 대한 자격을 얻으려면 중, 두 개 이상의 보통 이상의 인스턴스를 두 개 이상 프로비전해야 합니다.

### <a name="azure-search"></a>Azure Search
* **복제본을 두 개 이상 프로비전합니다.** 읽기 고가용성을 위해서는 복제를 두 개 이상, 또는 읽기-쓰기 고가용성을 위해서는 세 개 이상 사용합니다.
* **다중 지역 배포의 경우 인덱서를 구성합니다.** 다중 지역 배포를 설정한 경우 옵션에 대해 인덱싱의 지속성을 고려합니다.

  * 데이터 원본이 지리적으로 복제되는 경우 일반적으로 각 Azure Search의 인덱서가 각각 해당 로컬 데이터 원본 복제본을 가리키게 해야 합니다. 그러나 Azure SQL Database에 저장된 대형 데이터 집합의 경우 이 방법이 권장되지 않습니다. 그 이유는 Azure Search가 보조 SQL Database 복제에서 증분 인덱싱을 수행할 수 없고 주 복제본에서만 가능하기 때문입니다. 그 대신에 모든 인덱서가 주 복제본을 가리킵니다. 장애 조치 후 새 주 복제본에서 Azure Search 인덱서를 가리킵니다.  
  * 데이터 원본이 지리적으로 복제되지 않는 경우 여러 지역의 Azure Search 서비스가 지속적으로 그리고 독립적으로 데이터 원본에서 인덱싱하도록 여러 인덱서가 동일한 데이터 원본을 가리키게 합니다. 자세한 내용은 [Azure Search 성능 및 최적화 고려 사항][search-optimization]을 참조하세요.

### <a name="azure-storage"></a>Azure Storage
* **응용 프로그램 데이터의 경우 RA-GRS(읽기-쓰기 지역 중복 저장소)를 사용합니다.** RA-GRS 저장소는 데이터를 보조 지역에 복제하고 보조 지역에서 읽기 전용 액세스를 제공합니다. 주 지역에 저장소 중단이 있는 경우 응용 프로그램은 보조 지역에서 데이터를 읽을 수 있습니다. 자세한 내용은 [Azure Storage 복제](/azure/storage/storage-redundancy/)를 참조하세요.
* **VM 디스크의 경우 Managed Disks를 사용합니다.** [Managed Disks][managed-disks]는 디스크들이 서로 충분히 격리되어 단일 실패 지점을 방지하므로 가용성 집합의 VM에 대해 더 나은 신뢰성을 제공합니다. 또한 Managed Disks는 저장소 계정에서 만든 VHD의 IOPS 제한이 적용되지 않습니다. 자세한 내용은 [Azure에서 Windows 가상 머신의 가용성 관리][vm-manage-availability]를 참조하세요.
* **Queue Storage의 경우 다른 지역에 백업 큐를 만듭니다.** Queue Storage의 경우 항목을 큐에 저장하거나 큐에서 제거할 수 없으므로 읽기 전용 복제본의 사용이 제한됩니다. 그 대신에 다른 지역의 저장소 계정에 백업 큐를 만듭니다. 저장소 중단이 있는 경우 응용 프로그램은 주 지역이 다시 사용할 수 있게 될 때까지 백업 큐를 사용할 수 있습니다. 이런 방식으로 응용 프로그램이 새 요청을 계속 처리할 수 있습니다.  

### <a name="cosmos-db"></a>Cosmos DB
* **지역에 걸쳐 데이터베이스를 복제합니다.** Cosmos DB를 사용하면 개수에 관계없이 Azure 지역을 원하는 만큼 Cosmos DB 데이터베이스 계정에 연결할 수 있습니다. Cosmos DB 데이터베이스는 하나의 쓰기 지역 및 다중 읽기 지역을 포함할 수 있습니다. 쓰기 지역에 오류가 있으면 다른 복제본에서 읽을 수 있습니다. 클라이언트 SDK는 이 작업을 자동으로 처리합니다. 또한 쓰기 지역을 다른 지역으로 장애 조치할 수도 있습니다. 자세한 내용은 [Azure Cosmos DB로 데이터를 전역적으로 배포하는 방법](/azure/documentdb/documentdb-distribute-data-globally)을 참조하세요.

### <a name="sql-database"></a>SQL Database
* **표준 또는 프리미엄 계층을 사용합니다.** 이러한 계층은 더 긴 지정 시간 복원 기간(35일)을 제공합니다. 자세한 내용은 [SQL Database 옵션 및 성능](/azure/sql-database/sql-database-service-tiers/)을 참조하세요.
* **SQL Database 감사를 사용합니다.** 감사를 사용하면 악의적 공격 또는 인적 오류를 진단할 수 있습니다. 자세한 내용은 [SQL 데이터베이스 감사 시작](/azure/sql-database/sql-database-auditing-get-started/)을 참조하세요.
* **활성 지역 복제 사용** 활성 지역 복제를 사용하여 서로 다른 지역에 읽을 수 있는 보조 복제를 만듭니다.  주 데이터베이스가 실패하거나 단순히 오프라인으로 전환해야 하는 경우 보조 데이터베이스로 장애 조치를 수행합니다.  장애 조치(failover)될 때까지 보조 데이터베이스는 읽기 전용으로 유지됩니다.  자세한 내용은 [SQL Database 활성 지역 복제](/azure/sql-database/sql-database-geo-replication-overview/)를 참조하세요.
* **분할을 사용합니다.** 분할을 사용하여 데이터베이스를 가로로 분할하는 것을 고려합니다. 분할은 오류 격리를 제공할 수 있습니다. 자세한 내용은 [Azure SQL Database를 사용하여 분할](/azure/sql-database/sql-database-elastic-scale-introduction/)을 참조하세요.
* **지정 시간 복원을 사용하여 인적 오류에서 복구합니다.**  지정 시간 복원은 데이터베이스를 이전 시점으로 되돌려 놓습니다. 자세한 내용은 [자동화된 데이터베이스 백업을 사용하여 Azure SQL 데이터베이스 복구][sql-restore]를 참조하세요.
* **지리적 복원을 사용하여 서비스 중단에서 복구합니다.** 지리적 복원은 지역 중복 백업에서 데이터베이스를 복원합니다.  자세한 내용은 [자동화된 데이터베이스 백업을 사용하여 Azure SQL 데이터베이스 복구][sql-restore]를 참조하세요.

### <a name="sql-server-running-in-a-vm"></a>SQL Server(VM에서 실행)
* **데이터베이스를 복제합니다.** SQL Server Always On 가용성 그룹을 사용하여 데이터베이스를 복제합니다. SQL Server 인스턴스 하나가 실패하는 경우 고가용성을 제공 합니다. 자세한 내용은 [N 계층 응용 프로그램에 대해 Windows VM 실행](../reference-architectures/virtual-machines-windows/n-tier.md) 참조
* **데이터베이스를 백업합니다**. 이미 [Azure Backup](https://azure.microsoft.com/documentation/services/backup/)을 사용하여 VM을 백업하는 경우 [DPM을 사용하는 SQL Server용 Azure Backup 워크로드](/azure/backup/backup-azure-backup-sql/)의 사용을 고려합니다. 이 방법에서는 조직에 대한 백업 관리자 역할 하나와 VM 및 SQL Server에 대한 통합 복구 절차가 있습니다. 그렇지 않으면 [Microsoft Azure에 대한 SQL Server 관리 백업](https://msdn.microsoft.com/library/dn449496.aspx)을 사용합니다.

### <a name="traffic-manager"></a>Traffic Manager
* **수동 장애 복구를 수행합니다.** Traffic Manager 장애 조치 후 자동으로 장애 복구하는 대신 수동 장애 복구를 수행합니다. 장애 복구 전에 모든 응용 프로그램 하위 시스템이 정상 상태인지 확인합니다.  그렇지 않으면 응용 프로그램이 데이터 센터 간에 앞뒤로 뒤집어지는 상황이 발생할 수 있습니다. 자세한 내용은 [고가용성을 위해 여러 지역에서 VM 실행](../reference-architectures/virtual-machines-windows/multi-region-application.md)을 참조하세요.
* **상태 프로브 끝점을 만듭니다.** 응용 프로그램의 전반적인 상태에 관하여 보고하는 사용자 지정 끝점을 만듭니다. 이렇게 하면 단지 프런트 엔드뿐만 아니라 중요 경로가 고장인 경우 Traffic Manager가 장애 조치할 수 있습니다. 끝점은 중요 의존성이 비정상 상태이거나 도달할 수 없는 경우 HTTP 오류 코드를 반환해야 합니다. 그러나 중요하지 않은 서비스에 대해서는 오류를 보고하지 마세요. 그렇지 않으면 상태 프로브가 필요하지 않을 때 장애 조치를 트리거하여 가양성이 발생할 수 있습니다. 자세한 내용은 [Traffic Manager 끝점 모니터링 및 장애 조치](/azure/traffic-manager/traffic-manager-monitoring/)를 참조하세요.

### <a name="virtual-machines"></a>Virtual Machines
* **단일 VM에서 프로덕션 워크로드를 실행하지 않습니다.** 단일 VM 배포는 계획되거나 계획되지 않은 유지 관리에 탄력적으로 대처할 수 없습니다. 그 대신에 부하 분산 장치를 전면에 배치한 상태에서 여러 VM을 가용성 집합 또는 [VM 확장 집합](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/)에 배치합니다.
* **VM을 프로비전할 때 가용성 집합을 지정합니다.** 현재 VM이 프로비전된 후 VM을 가용성 집합에 추가하는 방법은 없습니다. 기존 가용성 집합에 새 VM을 추가하는 경우 VM에 대한 NIC를 만들고 부하 분산 장치의 백 엔드 주소 풀에 NIC를 추가해야 합니다. 그렇지 않으면 부하 분산 장치가 네트워크 트래픽을 해당 VM에 경로 설정하지 않습니다.
* **각 응용 프로그램 계층을 별도의 가용성 집합에 배치합니다.** N 계층 응용 프로그램에서 서로 다른 계층의 VM을 동일한 가용성 집합에 배치하지 마세요. 가용성 집합의 VM은 장애 도메인(FD) 및 업데이트 도메인(UD)에 걸쳐 배치됩니다. 그러나 FD와 UD의 중복성 이점을 활용하려면 가용성 집합의 모든 VM이 동일한 클라이언트 요청을 처리할 수 있어야 합니다.
* **성능 요구 사항을 기반으로 올바른 VM 크기를 선택합니다.** 기존 워크로드를 Azure로 이동할 때 온-프레미스 서버와 가장 근접하게 일치하는 VM 크기부터 사용하기 시작합니다. 그런 다음 CPU, 메모리 및 디스크 IOPS에 따라 실제 워크로드의 성능을 측정하고 필요에 따라 크기를 조정합니다. 이렇게 하면 해당 응용 프로그램이 클라우드 환경에서 예상한 대로 작동합니다. 또한 여러 NIC가 필요한 경우 각 크기에 대한 NIC 제한을 알아야 합니다.
* **VHD에 Managed Disks를 사용합니다.** [Managed Disks][managed-disks]는 디스크들이 서로 충분히 격리되어 단일 실패 지점을 방지하므로 가용성 집합의 VM에 대해 더 나은 신뢰성을 제공합니다. 또한 Managed Disks는 저장소 계정에서 만든 VHD의 IOPS 제한이 적용되지 않습니다. 자세한 내용은 [Azure에서 Windows 가상 머신의 가용성 관리][vm-manage-availability]를 참조하세요.
* **응용 프로그램을 OS 디스크가 아닌 데이터 디스크에 설치합니다.** 이렇게 하지 않으면 디스크 크기 제한에 도달할 수 있습니다.
* **Azure Backup을 사용하여 VM을 백업합니다.** 백업은 실수로 인한 데이터 손실을 방지합니다. 자세한 내용은 [복구 서비스 자격 증명 모음으로 Azure VM 보호](/azure/backup/backup-azure-vms-first-look-arm/)를 참조하세요.
* 기본 상태 메트릭, 인프라 로그 및 [부팅 진단][boot-diagnostics]을 포함하여 **진단 로그를 사용합니다**. 부팅 진단은 VM이 부팅할 수 없는 상태로 전환되는 경우 부팅 오류를 진단하는 데 도움이 될 수 있습니다. 자세한 내용은 [Azure 진단 로그][diagnostics-logs]를 참조하세요.
* **AzureLogCollector 확장을 사용합니다.** (Windows VM만 해당) 이 확장은 운영자가 원격으로 VM에 로그인하지 않고 Azure 플랫폼 로그를 집계하고 Azure 저장소에 업로드합니다. 자세한 내용은 [AzureLogCollector 확장](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)을 참조하세요.

### <a name="virtual-network"></a>Virtual Network
* **공용 IP 주소를 허용하거나 차단하려면 NSG를 서브넷에 추가합니다.** 악의적인 사용자의 액세스를 차단하거나 응용 프로그램에 액세스할 권한을 가진 사용자의 액세스만 허용합니다.  
* **사용자 지정 상태 프로브를 만듭니다.** 부하 분산 장치 상태 프로브는 HTTP 또는 TCP를 테스트할 수 있습니다. VM에서 HTTP 서버를 실행하는 경우 HTTP 프로브는 TCP 프로브보다 더 나은 상태 표시기입니다. HTTP 프로브의 경우 모든 중요 의존성을 포함하여 응용 프로그램의 전반적인 상태를 보고하는 사용자 지정 끝점을 사용합니다. 자세한 내용은 [Azure 부하 분산 장치 개요](/azure/load-balancer/load-balancer-overview/)를 참조하세요.
* **상태 프로브를 차단하지 않습니다.** 부하 분산 장치 상태 프로브는 알려진 IP 주소 168.63.129.16에서 전송됩니다. 방화벽 정책 또는 NSG(네트워크 보안 그룹)에서 IP에 대한 트래픽을 차단하지 마세요. 상태 프로브를 차단하면 부하 분산 장치가 VM을 윤번에서 제거할 것입니다.
* **부하 분산 장치 로깅을 사용하도록 설정합니다.** 로그는 백 엔드의 VM이 실패한 프로브 응답으로 인해 네트워크 트래픽을 수신하지 못하는 횟수를 표시합니다. 자세한 내용은 [Azure Load Balancer에 대한 로그 분석](/azure/load-balancer/load-balancer-monitor-log/)을 참조하세요.

<!-- links -->
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale/
[asynchronous-c-sharp]: /dotnet/articles/csharp/async
[availability-sets]:/azure/virtual-machines/virtual-machines-windows-manage-availability/
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[circuit-breaker]: ../patterns/circuit-breaker.md
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[fma]: ../resiliency/failure-mode-analysis.md
[resilient-deployment]: ../resiliency/index.md#resilient-deployment
[load-balancer]: /azure/load-balancer/load-balancer-overview/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring-and-diagnostics-guidance]: ../best-practices/monitoring.md
[resource-manager]: /azure/azure-resource-manager/resource-group-overview/
[retry-pattern]: ../patterns/retry.md
[retry-service-guidance]: ../best-practices/retry-service-specific.md
[search-optimization]: /azure/search/search-performance-optimization/
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[traffic-manager]: /azure/traffic-manager/traffic-manager-overview/
[traffic-manager-routing]: /azure/traffic-manager/traffic-manager-routing-methods/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
[vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview/
