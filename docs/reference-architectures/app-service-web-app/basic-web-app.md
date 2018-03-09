---
title: "기본 웹앱 응용 프로그램"
description: "Microsoft Azure에서 실행되는 기본 웹 응용 프로그램에 권장하는 아키텍처입니다."
author: MikeWasson
ms.date: 12/12/2017
cardTitle: Basic web application
ms.openlocfilehash: 38b0739cc61d679742b610b99e92aaad8d3b394d
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/08/2018
---
# <a name="basic-web-application"></a>기본 웹앱 응용 프로그램
[!INCLUDE [header](../../_includes/header.md)]

이 참조 아키텍처는 [Azure App Service][app-service] 및 [Azure SQL Database][sql-db]를 사용하는 웹 응용 프로그램에 관한 일련의 검증된 사례를 보여 줍니다. [**이 솔루션을 배포합니다.**](#deploy-the-solution)

![[0]][0]

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

## <a name="architecture"></a>건축 

> [!NOTE]
> 이 아키텍처는 응용 프로그램 개발에 초점을 두지 않으며 특정 응용 프로그램 프레임워크를 가정하지 않습니다. 다양한 Azure 서비스가 어떻게 연결되는지 이해하는 것이 목표입니다.
>
>

이 아키텍처의 구성 요소는 다음과 같습니다.

* **리소스 그룹**. [리소스 그룹](/azure/azure-resource-manager/resource-group-overview)은 Azure 리소스에 대한 논리적 컨테이너입니다.

* **App Service 앱**. [Azure App Service][app-service]는 클라우드 응용 프로그램을 만들고 배포하기 위한 완전히 관리되는 플랫폼입니다.     

* **App Service 계획**. [App Service 계획][app-service-plans]은 앱을 호스트하는 관리되는 VM(가상 머신)을 제공합니다. 계획과 연결된 모든 앱은 같은 VM 인스턴스에서 실행됩니다.

* **배포 슬롯**.  [배포 슬롯][deployment-slots]을 사용하면 배포를 스테이징한 다음 프로덕션 배포로 교환할 수 있습니다. 따라서 프로덕션으로 바로 배포하지 않아도 됩니다. 관련 권장 사항은 [관리 효율성](#manageability-considerations) 섹션을 참조하세요.

* **IP 주소**. App Service 앱에는 공용 IP 주소와 도메인 이름이 있습니다. 도메인 이름은 `contoso.azurewebsites.net`와 같이 `azurewebsites.net`의 하위 도메인입니다.  

* **Azure DNS**. [Azure DNS][azure-dns]는 Microsoft Azure 인프라를 사용하여 이름 확인을 제공하는 DNS 도메인에 대한 호스팅 서비스입니다. Azure에 도메인을 호스트하면 다른 Azure 서비스와 동일한 자격 증명, API, 도구 및 대금 청구를 사용하여 DNS 레코드를 관리할 수 있습니다. 사용자 지정 도메인 이름(예: `contoso.com`)을 사용하려면 사용자 지정 도메인 이름을 IP 주소에 매핑하는 DNS 레코드를 작성합니다. 자세한 내용은 [Azure App Service에서 사용자 지정 도메인 이름 구성][custom-domain-name]을 참조하세요.  

* **Azure SQL Database**. [SQL Database][sql-db]는 클라우드에서 실행되는 관계형 DaaS(Database-as-a-Service)입니다. SQL Database는 해당 코드 베이스를 Microsoft SQL Server 데이터베이스 엔진과 공유합니다. 응용 프로그램 요구 사항에 따라 [Azure Database for MySQL](/azure/mysql) 또는 [Azure Database for PostgreSQL](/azure/postgresql)을 사용할 수도 있습니다. 이 기능은 각각 오픈 소스 MySQL Server 및 Postgres 데이터베이스 엔진에 기반하여 완전히 관리되는 데이터베이스 서비스입니다.

* **논리 서버**. Azure SQL Database에서 논리 서버는 데이터베이스를 호스트합니다. 논리 서버당 데이터베이스를 여럿 만들 수 있습니다.

* **Azure Storage**. 메타데이터를 저장할 Blob 컨테이너가 있는 Azure Storage 계정을 만듭니다.

* **Azure AD(Azure Active Directory)**. 인증을 위해 Azure AD 또는 다른 ID 공급자를 사용합니다.

## <a name="recommendations"></a>권장 사항

개발자의 요구 사항이 여기에 설명된 아키텍처와 다를 수 있습니다. 이 섹션의 권장 사항을 시작점으로 사용합니다.

### <a name="app-service-plan"></a>App Service 계획
규모 확장, 자동 크기 조정 및 SSL(Secure Sockets Layer)을 지원하는 표준 또는 프리미엄 계층을 사용합니다. 각 계층은 코어 및 메모리 수가 다른 여러 *인스턴스 크기*를 지원합니다. 계획을 만든 후 계층이나 인스턴스 크기를 변경할 수 있습니다. App Service 계획에 대한 자세한 내용은 [App Service 가격][app-service-plans-tiers]을 참조하세요.

앱이 중지된 경우에도 App Service 계획의 인스턴스에 대해 요금이 청구됩니다. 사용하지 않는 계획(예: 테스트 배포)은 삭제하세요.

### <a name="sql-database"></a>SQL Database
SQL Database의 [V12 버전][sql-db-v12]을 사용합니다. SQL Database는 기본, 표준 및 프리미엄 [서비스 계층][sql-db-service-tiers]을 지원하며, 각 계층 내에서 [DTU(데이터베이스 트랜잭션 단위)][sql-dtu]로 측정되는 성능 수준이 다양합니다. 용량 계획을 수행하고 요구 사항에 맞은 계층과 성능 수준을 선택합니다.

### <a name="region"></a>지역
네트워크 대기 시간을 최소화하려면 App Service 계획과 SQL Database를 같은 지역에 프로비전합니다. 일반적으로 사용자에게 가장 가까운 지역을 선택합니다.

리소스 그룹에는 배포 메타데이터가 저장되는 위치를 지정하는 지역도 있습니다. 리소스 그룹과 해당 리소스를 같은 지역에 배치합니다. 이렇게 하면 배포 중 가용성을 향상할 수 있습니다. 

## <a name="scalability-considerations"></a>확장성 고려 사항

Azure App Service의 주요 이점은 부하에 따라 응용 프로그램을 확장할 수 있다는 점입니다. 다음은 응용 프로그램 확장을 계획할 때 염두할 몇 가지 고려 사항입니다.

### <a name="scaling-the-app-service-app"></a>App Service 앱 크기 조정

App Service 앱은 확장하는 방법에는 다음 두 가지가 있습니다.

* *강화* 즉, 인스턴스 크기를 변경합니다. 인스턴스 크기에 따라 각 VM 인스턴스의 메모리, 코어 수 및 저장소가 결정됩니다. 인스턴스 크기나 계획 계층을 변경하여 수동으로 확장할 수 있습니다.  

* *규모 확장* 즉, 증가된 부하를 처리할 인스턴스를 추가합니다. 각 가격 책정 계층에는 최대 인스턴스 수가 있습니다. 

  인스턴스 수를 변경하여 수동으로 확장하거나 [자동 크기 조정][web-app-autoscale]을 사용하여 Azure에서 일정 및/또는 성능 메트릭에 따라 인스턴스를 자동으로 추가하거나 제거하도록 할 수 있습니다. 각 확장 작업은 빠르게, 대개 몇 초 내에 실행됩니다. 

  자동 크기 조정을 사용하려면 최소 및 최대 인스턴스 수를 정의하는 자동 크기 조정 *프로필*을 만듭니다. 프로필을 예약할 수 있습니다. 예를 들어 평일과 주말의 프로필을 별개로 만들 수 있습니다. 프로필에 인스턴스를 추가하거나 제거할 시기에 대한 규칙을 포함할 수도 있습니다. (예: CPU 사용량이 5분 동안 70% 이상인 경우 두 개의 인스턴스를 추가).
  
웹앱 확장에 대한 권장 사항:

* 규모를 확장하거나 축소하면 응용 프로그램이 다시 시작될 수 있으므로 가능한 한 피합니다. 그 대신에 일반적인 부하에서 성능 요구 사항에 맞는 계층 및 크기를 선택한 다음 규모 확장하여 트래픽 볼륨의 변화를 처리합니다.    
* 자동 크기 조정을 사용합니다. 응용 프로그램의 워크로드가 예측 가능하고 정기적이면 인스턴스 수를 미리 예약하는 프로필을 만듭니다. 워크로드를 예측할 수 없는 경우에는 규칙 기반 자동 크기 조정을 사용하여 발생하는 부하 변경에 대처합니다. 이 두 가지 방법을 결합할 수 있습니다.
* CPU 사용량은 일반적으로 자동 크기 조정 규칙에 적합한 메트릭입니다. 하지만 응용 프로그램 부하 테스트를 수행하고, 잠재적인 병목 상태를 식별하고 이 데이터를 기준으로 자동 크기 조정 규칙을 만들어야 합니다.  
* 자동 크기 조정 규칙에는 확장 작업 완료 후 새 확장 작업을 시작하기 전에 대기하는 간격을 의미하는 *휴지* 기간이 포함됩니다. 휴지 기간을 이용하여 시스템을 다시 확장하기 전에 안정화합니다. 인스턴스 추가의 경우 휴지 기간을 짧게 설정하고 인스턴스 제거의 경우 휴지 기간을 길게 설정합니다. 예를 들어 인스턴스를 추가하려면 5분을 설정하지만 인스턴스를 제거하려면 60분을 설정합니다. 부하가 높은 경우 새 인스턴스를 빠르게 추가하여 추가 트래픽을 처리한 다음 점차 다시 축소하는 것이 좋습니다.

### <a name="scaling-sql-database"></a>SQL Database 크기 조정
SQL Database에 대해 더 높은 서비스 계층이나 성능 수준이 필요한 경우 응용 프로그램 가동을 중지하지 않고 개별 데이터베이스를 확장할 수 있습니다. 자세한 내용은 [SQL Database 옵션 및 성능: 각 서비스 계층에서 사용할 수 있는 옵션 이해][sql-db-scale]를 참조하세요.

## <a name="availability-considerations"></a>가용성 고려 사항
이 문서 작성 시점을 기준으로 App Service의 SLA(서비스 수준 계약)는 99.95%이고 SQL Database의 SLA는 기본, 표준 및 프리미엄 계층에 대해 99.99%입니다. 

> [!NOTE]
> App Service SLA는 단일 인스턴스와 여러 인스턴스 모두에 적용됩니다.  
>
>

### <a name="backups"></a>Backup
데이터 손실이 발생하면 SQL Database에서는 지정 시간 복원과 지역 복원을 제공합니다. 이러한 기능은 모든 계층에서 제공되며 자동으로 사용하도록 설정됩니다. 백업을 예약하거나 관리할 필요가 없습니다. 

- 지정 시간 복원을 사용하면 데이터베이스를 이전 시간으로 되돌려 [사용자 오류를 복구][sql-human-error]할 수 있습니다. 
- 지역 복원을 사용하면 지역 중복 백업에서 데이터베이스를 복원하여 [서비스 중단에 따른 손상을 복구][sql-outage-recovery]할 수 있습니다. 

자세한 내용은 [SQL Database의 클라우드 무중단 업무 방식 및 데이터베이스 재해 복구][sql-backup]를 참조하세요.

App Service에서는 응용 프로그램 파일에 대한 [백업 및 복원][web-app-backup] 기능을 제공합니다. 그러나 백업된 파일에는 앱 설정이 일반 텍스트로 포함되며 연결 문자열과 같은 비밀이 포함될 수 있다는 점에 유의합니다. SQL Database를 백업하기 위해 App Service 백업 기능을 사용하지 마세요. 이 경우 데이터베이스를 SQL .bacpac 파일로 내보내 [DTU][sql-dtu]를 소모하기 때문입니다. 대신 위에서 설명한 SQL Database 지정 시간 복원을 사용합니다.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항
프로덕션, 개발 및 테스트 환경에 대해 별도의 리소스 그룹을 만듭니다. 이렇게 하면 배포 관리, 테스트 배포 삭제, 액세스 권한 할당 등이 더 간단해집니다.

리소스를 리소스 그룹에 할당할 때는 다음 사항을 고려하세요.

* 수명 주기. 일반적으로 수명 주기가 동일한 리소스를 동일한 리소스 그룹에 배치합니다.
* 액세스. [RBAC(역할 기반 액세스 제어)][rbac]를 사용하여 그룹의 리소스에 액세스 정책을 적용할 수 있습니다.
* 청구. 리소스 그룹에 대한 롤업 비용을 확인할 수 있습니다.  

자세한 내용은 [Azure Resource Manager 개요](/azure/azure-resource-manager/resource-group-overview)를 참조하세요.

### <a name="deployment"></a>배포
배포는 다음 두 단계로 구성됩니다.

1. Azure 리소스 프로비전. 이 단계에는 [Azure 리소스 관리자 템플릿][arm-template]을 사용하는 것이 좋습니다. 템플릿을 사용하면 PowerShell이나 Azure CLI(명령줄 인터페이스)를 통해 배포를 쉽게 자동화할 수 있습니다.
2. 응용 프로그램(코드, 이진 파일 및 콘텐츠 파일) 배포. 로컬 Git 리포지토리에서 배포하거나, Visual Studio를 사용하여 배포하거나, 클라우드 기반 소스 제어를 통해 연속 배포하는 등 여러 옵션이 있습니다. [Azure App Service에 앱 배포][deploy]를 참조하세요.  

App Service App에는 라이브 프로덕션 사이트를 나타내는 `production`이라는 배포 슬롯 하나가 항상 있습니다. 업데이트를 배포하기 위한 스테이징 슬롯을 만드는 것이 좋습니다. 스테이징 슬롯을 사용하면 다음과 같은 이점이 있습니다.

* 프로덕션으로 교환하기 전에 배포가 성공했는지 확인할 수 있습니다.
* 스테이징 슬롯에 배포하면 모든 인스턴스가 프로덕션으로 교환되기 전에 확실히 준비됩니다. 많은 응용 프로그램에는 중요한 준비 및 콜드 부팅 시간이 있습니다.

또한 마지막으로 성공한 올바른 배포를 보유할 세 번째 슬롯을 만드는 것이 좋습니다. 스테이징과 프로덕션을 교환한 후 이전 프로덕션 배포(이제는 스테이징에 있음)를 마지막으로 성공한 올바른 슬롯으로 이동합니다. 이렇게 하면 나중에 문제를 발견하는 경우 마지막으로 성공한 올바른 버전으로 신속하게 되돌릴 수 있습니다.

![[1]][1]

이전 버전으로 되돌리는 경우에는 데이터베이스 스키마 변경 내용이 이전 버전과 호환되는지 확인합니다.

동일한 App Service 계획 내의 모든 앱이 동일한 VM 인스턴스를 공유하므로 프로덕션 배포의 슬롯을 테스트에 사용하지 마세요. 예를 들어 부하 테스트는 라이브 프로덕션 사이트의 성능을 떨어뜨릴 수 있습니다. 대신 프로덕션과 테스트에 대한 App Service 계획을 별도로 만듭니다. 테스트 배포를 별도의 계획에 배치하여 프로덕션 버전과 격리합니다.

### <a name="configuration"></a>구성
구성 설정을 [앱 설정][app-settings]으로 저장합니다. 앱 설정을 리소스 관리자 템플릿에서 정의하거나 PowerShell을 사용하여 정의합니다. 런타임 시 앱 설정을 응용 프로그램에 환경 변수로 사용할 수 있습니다.

암호, 액세스 키 또는 연결 문자열을 소스 제어로 체크 인하지 마세요. 대신 이러한 값을 배포 스크립트에 매개 변수로 전달하여 앱 설정으로 저장합니다.

배포 슬롯을 교환하면 앱 설정이 기본적으로 교환됩니다. 프로덕션 및 스테이징에 서로 다른 설정이 필요하면 슬롯에 고정되어 교환되지 않는 앱 설정을 만들 수 있습니다.

### <a name="diagnostics-and-monitoring"></a>진단 및 모니터링
응용 프로그램 로깅 및 웹 서버 로깅을 포함하여 [진단 로깅][diagnostic-logs]을 사용하도록 설정합니다. Blob Storage를 사용하도록 로깅을 구성합니다. 성능을 위해 진단 로그를 저장할 별도의 저장소 계정을 만듭니다. 로그와 응용 프로그램 데이터에 동일한 저장소 계정을 사용하지 마세요. 로깅에 대한 자세한 내용은 [모니터링 및 진단 지침][monitoring-guidance]을 참조하세요.

[New Relic][ new-relic] 또는 [Application Insights][app-insights] 같은 서비스를 사용하여 응용 프로그램 성능 및 부하를 받을 때의 동작을 모니터링합니다. Application Insights에 대한 [데이터 속도 제한][app-insights-data-rate]을 알아둡니다.

[Visual Studio Team Services][vsts]와 같은 도구를 사용하여 부하 테스트를 수행합니다. 클라우드 응용 프로그램의 성능 분석에 대한 개요는 [Performance Analysis Primer][perf-analysis]를 참조하세요.

응용 프로그램 문제 해결 팁:

* Azure Portal에 [문제 해결 블레이드][troubleshoot-blade]를 사용하여 일반적인 문제에 대한 해결 방법을 찾습니다.
* [로그 스트리밍][web-app-log-stream]을 사용하도록 설정하여 로그 정보를 거의 실시간으로 확인할 수 있습니다.
* [Kudu 대시보드][ kudu]에는 응용 프로그램을 모니터링하고 디버깅하기 위한 여러 도구가 있습니다. 자세한 내용은 [Azure Websites online tools you should know about][kudu](알아 두면 도움이 되는 Azure 웹 사이트 온라인 도구)(블로그 게시물)를 참조하세요. Azure Portal에서 Kudu 대시보드에 연결할 수 있습니다. 앱의 블레이드를 열고 **도구**, **Kudu**를 차례로 클릭합니다.
* Visual Studio를 사용하는 경우 [Visual Studio를 사용하여 Azure App Service에서 웹앱 문제 해결][troubleshoot-web-app]에서 디버깅 및 문제 해결 팁을 참조하세요.

## <a name="security-considerations"></a>보안 고려 사항
이 섹션에는 이 문서에 설명된 Azure 서비스와 관련된 보안 고려 사항이 나와 있습니다. 보안 모범 사례가 완전히 다 나와 있는 것은 아닙니다. 몇 가지 추가 보안 고려 사항은 [Azure App Service에서 앱 보안][app-service-security]을 참조하세요.

### <a name="sql-database-auditing"></a>SQL Database 감사
감사는 규정 준수를 유지 관리하고, 비즈니스 문제나 의심스러운 보안 위반을 나타낼 수 있는 불일치 및 이상 활동을 파악하는 데 도움이 될 수 있습니다. [SQL Database 감사 시작][sql-audit]을 참조하세요.

### <a name="deployment-slots"></a>배포 슬롯
각 배포 슬롯에는 공용 IP 주소가 있습니다. 개발 및 DevOps 팀의 구성원만 해당 끝점에 연결할 수 있도록 [Azure Active Directory 로그인][ aad-auth]을 사용하여 프로덕션이 아닌 슬롯을 보호합니다.

### <a name="logging"></a>로깅
사용자 암호 또는 ID 사기에 사용될 수 있는 기타 정보는 절대로 기록해서는 안 됩니다. 데이터를 저장하기 전에 데이터에서 이러한 세부 정보를 지웁니다.   

### <a name="ssl"></a>SSL
App Service 앱에서는 추가 비용 없이 `azurewebsites.net`의 하위 도메인의 SSL 끝점을 제공합니다. SSL 끝점에는 `*.azurewebsites.net` 도메인에 대한 와일드카드 인증서가 포함되어 있습니다. 사용자 지정 도메인 이름을 사용하는 경우 사용자 지정 도메인과 일치하는 인증서를 제공해야 합니다. 가장 간단한 방법은 Azure Portal에서 직접 인증서를 구입하는 것입니다. 다른 인증 기관에서 인증서를 가져올 수도 있습니다. 자세한 내용은 [Azure App Service에 대한 SSL 인증서 구입 및 구성][ssl-cert]을 참조하세요.

보안을 극대화하려면 앱에서 HTTP 요청을 리디렉션하여 HTTPS를 적용해야 합니다. 응용 프로그램 내부에서 이를 구현하거나 로[Azure App Service에서 앱에 HTTPS 사용][ssl-redirect]에 설명된 대로 URL 다시 쓰기 규칙을 사용할 수 있습니다.

### <a name="authentication"></a>인증
Azure AD, Facebook, Google 또는 Twitter 같은 IDP(ID 공급자)를 통해 인증하는 것이 좋습니다. 인증 흐름에 OAuth 2 또는 OIDC(OpenID Connect)를 사용합니다. Azure AD는 사용자 및 그룹을 관리하고, 응용 프로그램 역할을 만들고, 온-프레미스 ID를 통합하고, Office 365, 비즈니스용 Skype 같은 백 엔드 서비스를 사용할 수 있는 기능을 제공합니다.

응용 프로그램에서 사용자 로그인 및 자격 증명을 직접 관리하도록 하지 마세요. 이 경우 잠재적인 공격 노출 영역이 생길 수 있습니다.  최소한 메일 확인, 암호 복구 및 다단계 인증을 사용하도록 해야 하고 암호 강도를 확인하고, 암호 해시를 안전하게 저장해야 합니다. 대형 ID 공급자는 이러한 기능을 모두 자동으로 처리하며 보안 방법을 지속적으로 모니터링하고 개선합니다.

[App Service 인증][app-service-auth]을 사용하여 OAuth/OIDC 인증 흐름을 구현하는 것이 좋습니다. App Service 인증의 이점은 다음과 같습니다.

* 구성하기 쉽습니다.
* 단순 인증 시나리오의 경우 코드가 필요하지 않습니다.
* OAuth 액세스 토큰을 사용한 위임된 권한 부여를 통해 사용자 대신 리소스를 사용하도록 지원합니다.
* 토큰 캐시를 기본 제공합니다.

App Service 인증의 몇 가지 제한 사항은 다음과 같습니다.  

* 사용자 지정 옵션이 제한되어 있습니다.
* 위임된 권한 부여는 로그인 세션당 하나의 백 엔드 리소스로 제한됩니다.
* 둘 이상의 IDP를 사용하는 경우 홈 영역 검색을 위한 기본 메커니즘이 없습니다.
* 다중 테넌트 시나리오의 경우 응용 프로그램에서 토큰 발급자의 유효성을 검사하는 논리를 구현해야 합니다.

## <a name="deploy-the-solution"></a>솔루션 배포
이 아키텍처에 대한 예제 리소스 관리자 템플릿은 [GitHub에서 제공][paas-basic-arm-template]됩니다.

PowerShell을 사용하여 템플릿을 배포하려면 다음 명령을 실행합니다.

```
New-AzureRmResourceGroup -Name <resource-group-name> -Location "West US"

$parameters = @{"appName"="<app-name>";"environment"="dev";"locationShort"="uw";"databaseName"="app-db";"administratorLogin"="<admin>";"administratorLoginPassword"="<password>"}

New-AzureRmResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile .\PaaS-Basic.json -TemplateParameterObject  $parameters
```

자세한 내용은 [Azure 리소스 관리자 템플릿을 사용하여 리소스 배포][deploy-arm-template]를 참조하세요.

<!-- links -->

[aad-auth]: /azure/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication
[app-insights]: /azure/application-insights/app-insights-overview
[app-insights-data-rate]: /azure/application-insights/app-insights-pricing
[app-service]: https://azure.microsoft.com/documentation/services/app-service/
[app-service-auth]: /azure/app-service-api/app-service-api-authentication
[app-service-plans]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[app-service-plans-tiers]: https://azure.microsoft.com/pricing/details/app-service/
[app-service-security]: /azure/app-service-web/web-sites-security
[app-settings]: /azure/app-service-web/web-sites-configure
[arm-template]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[azure-dns]: /azure/dns/dns-overview
[custom-domain-name]: /azure/app-service-web/web-sites-custom-domain-name
[deploy]: /azure/app-service-web/web-sites-deploy
[deploy-arm-template]: /azure/resource-group-template-deploy
[deployment-slots]: /azure/app-service-web/web-sites-staged-publishing
[diagnostic-logs]: /azure/app-service-web/web-sites-enable-diagnostic-log
[kudu]: https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/
[monitoring-guidance]: ../../best-practices/monitoring.md
[new-relic]: http://newrelic.com/
[paas-basic-arm-template]: https://github.com/mspnp/reference-architectures/tree/master/managed-web-app/basic-web-app/Paas-Basic/Templates
[perf-analysis]: https://github.com/mspnp/performance-optimization/blob/master/Performance-Analysis-Primer.md
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[sla]: https://azure.microsoft.com/support/legal/sla/
[sql-audit]: /azure/sql-database/sql-database-auditing-get-started
[sql-backup]: /azure/sql-database/sql-database-business-continuity
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-db-overview]: /azure/sql-database/sql-database-technical-overview
[sql-db-scale]: /azure/sql-database/sql-database-service-tiers#scaling-up-or-scaling-down-a-single-database
[sql-db-service-tiers]: /azure/sql-database/sql-database-service-tiers
[sql-db-v12]: /azure/sql-database/sql-database-features
[sql-dtu]: /azure/sql-database/sql-database-service-tiers
[sql-human-error]: /azure/sql-database/sql-database-business-continuity#recover-a-database-after-a-user-or-application-error
[sql-outage-recovery]: /azure/sql-database/sql-database-business-continuity#recover-a-database-to-another-region-from-an-azure-regional-data-center-outage
[ssl-redirect]: /azure/app-service-web/web-sites-configure-ssl-certificate#bkmk_enforce
[sql-resource-limits]: /azure/sql-database/sql-database-resource-limits
[ssl-cert]: /azure/app-service-web/web-sites-purchase-ssl-web-site
[troubleshoot-blade]: https://azure.microsoft.com/updates/self-service-troubleshooting-for-app-service-web-apps-customers/
[troubleshoot-web-app]: /azure/app-service-web/web-sites-dotnet-troubleshoot-visual-studio
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx
[vsts]: https://www.visualstudio.com/features/vso-cloud-load-testing-vs.aspx
[web-app-autoscale]: /azure/app-service-web/web-sites-scale
[web-app-backup]: /azure/app-service-web/web-sites-backup
[web-app-log-stream]: /azure/app-service-web/web-sites-enable-diagnostic-log#streamlogs
[0]: ./images/basic-web-app.png "기본 Azure 웹 응용 프로그램의 아키텍처"
[1]: ./images/paas-basic-web-app-staging-slots.png "프로덕션 및 스테이징 배포에 대한 슬롯 교환"
