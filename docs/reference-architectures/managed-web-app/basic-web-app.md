---
title: Basic web application
description: >-
  Recommended architecture for a basic web application running in Microsoft
  Azure.


author: MikeWasson




ms.service: guidance

ms.topic: article


ms.date: 11/23/2016
ms.author: pnp
cardTitle: Basic web application
---
# 기본 웹 응용 프로그램
[!INCLUDE [header](../../_includes/header.md)]

이 참조 아키텍처는 [Azure App Service][app-service] 와 [Azure SQL Database][sql-db]를 사용하는 웹 응용 프로그램에 대한 일련의 검증된 사례를 보여줍니다. [**이 솔루션 배포하기.**](#deploy-the-solution)


![[0]][0]

## 아키텍처

> [!참고]
> 이 아키텍처는 응용 프로그램 개발에 중점을 두지 않으며 특정 응용 프로그램 프레임워크를 가정하지 않습니다. 이 아키텍처의 목표는 얼마나 다양한 Azure 서비스가 서로 조화를 이룰 수 있는지를 이해하는 것입니다. 
>
>

이 아키텍처는 다음과 같은 요소들로 구성되어 있습니다.

* **리소스 그룹**. [리소스 그룹](/azure/azure-resource-manager/resource-group-overview)은 Azure 리소스를 위한 논리적 컨테이너입니다.
* **App Service 앱 **. [Azure App Service][app-service]는 클라우드 응용 프로그램을 생성하고 배포하기 위한 완전 관리 플랫폼입니다.     
* **App Service 요금제**. [App Service 요금제][app-service-plans]는 앱을 호스팅하는 관리 가상 컴퓨터(VM)를 제공합니다. 하나의 요금제에 연결된 모든 앱은 동일한 VM 인스턴스에서 실행됩니다.

* **배포 슬롯**. [배포 슬롯][deployment-slots]을 통해 배포를 스테이징한 후 프로덕션 배포로 변경할 수 있습니다.  이런 방법을 통해 프로덕션 용도로 직접 배포하는 것을 피할 수 있습니다. 구체적인 권장사항은 [관리 효율성](#manageability-considerations) 섹션을 참조하시기 바랍니다.

* **IP 주소**. App Service 앱은 공용 IP 주소와 도메인 이름을 갖습니다. 도메인 이름은 `contoso.azurewebsites.net`과 같이 `azurewebsites.net`의 하위 도메인입니다. `contoso.com`과 같이 사용자 지정 도메인 이름을 사용하려면 사용자 지정 도메인 이름을 해당 IP 주소에 매핑하는 도메인 이름 서비스(DNS) 기록을 생성합니다. 자세한 내용은 [Azure App Service에서 사용자 지정 도메인 이름 구성][custom-domain-name]을 참조하시기 바랍니다.
* **Azure SQL Database**. [SQL Database][sql-db]는 클라우드의 관계형 DaaS(database-as-a-service)입니다.
* **논리적 서버**. Azure SQL Database에서는 논리적 서버가 DB를 호스팅합니다. 논리적 서버 하나에 여러 DB를 생성할 수 있습니다.
* **Azure 저장소**. 진단 로그를 저장하기 위한 blob 컨테이너를 사용하는 Azure 저장소 계정을 만듭니다.
* **Azure Active Directory** (Azure AD). Azure AD 또는 다른 계정 제공자를 이용하여 인증을 수행합니다.

## 권장사항

이 문서에서 설명하는 아키텍처는 귀하의 요구사항과 정확히 일치하지 않을 수 있습니다. 따라서 이 문서의 권장사항은 하나의 출발점으로 삼으시기 바랍니다.

### App Service 요금제
확장, 자동 크기 조정, 보안 소켓 레이어(SSL)를 이용하려면 Standard 또는 Premium 요금제를 사용하시기 바랍니다. 각각의 요금제는 코어 수와 메모리에 따라 몇 가지 다른 *인스턴스 크기*를 지원합니다. 요금제를 선택한 후에도 요금제나 인스턴스 크기를 변경할 수 있습니다. App Service 요금제에 대한 자세한 내용은 [App Service 가격 책정][app-service-plans-tiers]을 참조하시기 바랍니다.

요금은 앱이 중단되더라도 App Service 요금제에 포함된 인스턴스에 대해 청구됩니다. (테스트 배포 등) 사용하지 않는 요금제는 반드시 삭제하시기 바랍니다.

### SQL 데이터베이스
[V12 version][sql-db-v12]의 SQL Database를 사용합니다. SQL Database는 Basic, Standard, Premium [요금제][sql-db-service-tiers]를 제공하고 각 요금제는 다시 [Database Transaction Units (DTUs)][sql-dtu]에서 측정된 여러 성능 수준으로 나뉩니다. 용량 계획을 수립한 후 요구사항에 적합한 요금제와 성능 수준을 선택하시기 바랍니다.

### 지역
App Service 요금제와 SQL Database를 동일 지역에 프로비전하여 네트워크 대기 시간을 최소화합니다. 일반적으로 사용자와 가장 가까운 지역을 선택합니다.

또한 리소스 그룹은 배포 메타데이터가 저장되는 위치인 지역을 포함합니다. 리소스 그룹과 그 리소스를 동일 지역에 배치합니다. 이를 통해 배포 시 향상된 가용성을 얻을 수 있습니다.

## 확장성 고려사항

Azure App Service의 주요 이점은 응용 프로그램을 부하에 따라 확장할 수 있다는 점입니다. 응용 프로그램 확장 계획 수립 시 고려해야할 사항은 다음과 같습니다.

### App Service 앱 확장

App Service 앱을 확장하는 방법은 두 가지입니다. 

* 하나는 인스턴스 크기를 변경하는 방법인 *강화*입니다. 인스턴스 크기는 각각의 VM 인스턴스의 메모리, 코어 수 및 저장 용량을 결정합니다. 인스턴스 크기나 요금제를 변경하여 수동으로 확장할 수 있습니다.  

* 다른 하나는 증가된 부하를 처리하기 위해 인스턴스를 추가하는 방법인 *규모 확장*입니다. 각 요금제는 서로 다른 최대 인스턴스 수를 제공합니다.

  인스턴스 수를 변경하여 수동으로 확장하거나 [자동 크기 조정][web-app-autoscale]을 사용하여 Azure가 일정이나 성능 메트릭에 따라 자동으로 인스턴스를 추가하거나 삭제하도록 할 수 있습니다. 각각의 확장 작업은 일반적으로 수 초 내에 신속히 진행됩니다. 

  자동 크기 조정을 사용하려면 최소/최대 인스턴스 수를 정의하는 자동 크기 조정 프로필을 생성합니다. 프로필은 일정 관리가 가능합니다. 예를 들어, 주중과 주말을 분리하여 프로필을 생성할 수 있습니다. 또한 인스턴스의 추가/삭제 시점 규칙을 프로필에 포함시킬 수도 있습니다. (예: CPU 사용률이 5분 동안 70% 이상인 경우 두 개의 인스턴스를 추가.) 
  
웹 앱 확장에 대한 권장사항:

* 확장 및 축소는 응용 프로그램 재시작을 유발할 수 있으므로 가급적 사용을 자제하세요. 대신, 일반적인 부하 수준에서 성능 요구사항을 만족하는 요금제와 크기를 선택한 후 트래픽량에 변화가 있는 경우 인스턴스를 확장합니다.     
* 자동 크기 조정 사용. 응용 프로그램의 워크로드가 예측 가능하고 규칙적이라면 프로필을 생성하여 인스턴스 수를 예약할 수 있습니다. 워크로드의 예측이 어려운 경우에는 부하 변경 시 반응하는 규칙 기반 자동 크기 조정을 사용합니다. 또는 두 가지 방법을 결합할 수도 있습니다.
* 일반적으로 CPU 사용률은 자동 크기 조정 규칙을 위한 효과적인 메트릭입니다.  그러나 응용 프로그램에 대한 로드 테스트를 실시하고 잠재적 병목 현상을 확인하여 자동 크기 조정 규칙이 데이터에 기반하여 구성되도록 해야 합니다.   
* 자동 크기 조정 규칙에는 하나의 확장 동작이 완료된 후 새 확장 동작이 시작되기 전까지의 대기 간격인 휴지 시간이 포함됩니다. 휴지 시간을 통해 다음 확장 전까지 시스템을 안정화시킬 수 있습니다. 인스턴스 추가 시에는 휴지 시간을 짧게, 인스턴스 삭제 시에는 휴지 시간을 길게 설정합니다. 예를 들어, 인스턴스 추가 시 휴지 시간을 5분으로 설정한다면 인스턴스 삭제 시에는 60분으로 설정합니다. 추가 트래픽을 처리하기 위해 부하가 증가한 상황에서는 새 인스턴스를 신속히 추가한 다음 서서히 축소하는 것이 좋습니다.

### SQL Database 확장
SQL Database에 대해 상위 서비스 또는 성능 수준이 필요한 경우에는 응용 프로그램 중단 시간 없이 개별 DB를 확장할 수 있습니다. 자세한 내용은 [SQL 데이터베이스의 서비스 및 성능 수준 변경][sql-db-scale]을 참조하시기 바랍니다.

## 가용성 고려사항
쓰기를 할 때 Basic, Standard, Premium 요금제의 App Service 서비스 수준 계약(SLA)은 99.5%, SQL Database 서비스 수준 계약은 99.99%입니다. 

> [!참고]
> App Service SLA는 단일 및 여러 인스턴스에 모두 적용됩니다.   
>
>

### 백업
데이터 손실 발생 시 SQL Database는 특정 시점(point-in-time) 복원 및 지역 복원(geo-restore)을 제공합니다. 이 기능은 모든 요금제에서 제공되며, 자동으로 적용됩니다. 백업은 일정을 예약하거나 관리할 필요가 없습니다. 

- 특정 시점 복원을 사용하여 DB를 이전 시점으로 되돌림으로써 [사용자의 실수를 복구할 수 있습니다][sql-human-error]. 
- 지역 복원을 사용하면 지역 중복 백업으로부터 DB를 복원하여 [중단된 서비스를 복구할 수 있습니다][sql-outage-recovery]. 

자세한 내용은 [SQL Database의 클라우드 비즈니스 연속성 및 DB 재해 복구][sql-backup]를 참조하시기 바랍니다.

App Service는 응용 프로그램 파일을 위해 [백업 및 복원][web-app-backup] 기능을 제공합니다. 그러나 백업된 파일에는 일반 텍스트로 쓰여진 앱 설정이 포함되며 여기에는 연결 문자열과 같은 비밀 정보가 포함될 수 있으므로 주의해야 합니다. SQL DB를 백업 시 App Service 백업 기능을 사용하면 DB를 SQL .bacpac 파일로 내보내면서 [DTUs][sql-dtu]를 소비하므로 사용을 피합니다. 대신, 위에 설명한 SQL Database 특정 시점 복원 기능을 사용합니다.

## 관리 효율성 고려사항
프로덕션, 개발 및 테스트 환경을 위한 각각의 리소스 그룹을 생성합니다. 이를 통해 보다 쉽게 배포를 관리하고 테스트 배포를 삭제하고 접근 권한을 할당할 수 있습니다.

리소스 그룹에 리소스를 할당할 때는 다음과 같은 사항을 고려하시기 바랍니다.

* 수명 주기. 일반적으로 동일한 수명 주기를 갖는 리소스를 동일한 리소스 그룹에 할당합니다.
* 접근. [역할 기반 접근 제어][rbac] (RBAC)를 사용하여 하나의 그룹 내에 있는 리소스에 접근 정책을 적용할 수 있습니다.
* 요금 청구. 리소스 그룹에 대한 비용을 확인할 수 있습니다.  

자세한 내용은 [Azure Resource Manager 개요](/azure/azure-resource-manager/resource-group-overview)를 참조하시기 바랍니다.

### 배포
배포는 두 단계로 진행됩니다.

1. Azure 리소스 프로비전. [Azure Resoure Manager 템플릿][arm-template]을 이용하여 이 절차를 진행할 것을 권장합니다. 템플릿을 이용하면 PowerShell이나 Azure 명령줄 인터페이스(CLI)를 통해 보다 손쉽게 배포를 자동화할 수 있습니다.
2. 응용 프로그램 배포(코드, 바이너리, 컨텐츠 파일). 로컬 Git 저장소로부터의 배포, Visual Studio 이용 또는 클라우드 기반 소스 제어로부터의 계속 배포를 포함한 몇 가지 배포 방법을 선택할 수 있습니다. [Azure App Service에 앱 배포][deploy]를 참조하세요.  

App Service 앱은 항상 프로덕션 사이트를 말하는 `production`으로 명명된 하나의 개발 슬롯을 갖습니다. 업데이트 배포를 위한 스테이징 슬롯을 생성할 것을 권장합니다. 스테이징 슬롯을 사용하면 다음과 같은 이점을 얻을 수 있습니다.

* 배포를 프로덕션으로 전환하기 전에 배포의 성공 여부를 확인할 수 있습니다.
* 스테이징 슬롯으로 배포함으로써 프로덕션으로 전환하기 전에 모든 인스턴스가 준비되었는지 확인할 수 있습니다. 많은 응용 프로그램이 상당한 준비 및 콜드 부팅 시간을 갖습니다.

마지막으로 성공한(last-known-good) 배포를 담기 위한 세 번째 슬롯을 생성할 것을 권장합니다. 스테이징과 프로덕션의 전환 후 (현재 스테이징 상태인) 기존의 프로덕션 배포를 마지막으로 성공한 배포 슬롯으로 이동시킵니다. 이런 방법을 통해 추후 문제가 발견될 경우 신속하게 마지막으로 성공한 버전으로 되돌아갈 수 있습니다. 

![[1]][1]

이전 버전으로 되돌아갈 경우, 모든 DB 스키마 변경 사항이 이전 버전과 호환이 되는지 확인해야 합니다. 

동일한 App Service 요금제 내의 모든 앱은 동일한 VM 인스턴스를 공유하므로 프로덕션 배포 슬롯을 테스트용으로 사용해서는 안 됩니다. 예를 들어, 부하 테스트는 프로덕션 사이트의 성능을 저하시킬 수 있습니다. 따라서 프로덕션과 테스트는 별개의 App Service 요금제로 사용하시기 바랍니다. 테스트 배포를 별개의 요금제로 사용하면 프로덕션 버전으로부터 격리할 수 있습니다. 

### 구성
구성 설정을 [앱 설정][app-settings]으로 저장합니다. Resource Manager에 또는 PowerShell을 사용하여 앱 설정을 정의합니다. 런타임 시 앱 설정은 해당 응용 프로그램에 의해 환경 변수로 이용됩니다.

절대로 암호, 액세스 키 또는 연결 문자열을 소스 제어에 추가하지 마세요. 대신 이 값들을 앱 설정으로 저장하는 배포 스크립트에 매개변수로 전달합니다. 

배포 슬롯 전환 시 앱 설정도 전환되도록 기본 설정되어 있습니다. 프로덕션과 스테이징에 대해 서로 다른 설정이 필요한 경우에는 한 슬롯에 고정된 앱 설정을 만들어 전환을 방지할 수 있습니다. 

### 진단 및 모니터링
응용 프로그램 로깅, 웹 서버 로깅 등 [진단 로깅][diagnostic-logs]을 사용하도록 설정합니다. 로깅을 구성하여 Blob 스토리지를 사용합니다. 최상의 성능을 위해 진단 로그를 저장할 별도의 저장소 계정을 생성합니다. 로그와 응용 프로그램 데이터에 대해 동일한 저장소 계정을 사용해서는 안 됩니다. 로깅에 대한 자세한 내용은 [모니터링 및 진단 가이드][monitoring-guidance]를 참조하시기 바랍니다.

[New Relic][new-relic] 또는 [Application Insights][app-insights]를 사용하여 부하 상태에서의 응용 프로그램 성능과 거동을 모니터링할 수 있습니다. Application Insights의 경우 [데이터율 제한][app-insights-data-rate]에 주의하시기 바랍니다.

[Visual Studio Team Services][vsts]와 같은 도구를 사용하여 부하 테스트를 수행합니다. 클라우드 응용 프로그램에서의 성능 분석 개관은 [성능 분석의 기초][perf-analysis]를 참조하시기 바랍니다.

응용 프로그램 문제 해결 팁은 다음과 같습니다.

* Azure 포털의 [문제해결 블레이드][troubleshoot-blade]를 통해 일반적인 문제들에 대한 해결책을 찾을 수 있습니다.
* [로그 스트리밍][web-app-log-stream]을 사용하도록 설정하면 실시간에 가까운 로깅 정보를 확인할 수 있습니다.
* [Kudu 대시보드][kudu]는 응용 프로그램의 모니터링 및 디버깅을 위한 몇 가지 도구를 제공합니다. 자세한 내용은 [알면 유용한 Azure Websites 온라인 도구][kudu] (블로그)를 참조하시기 바랍니다. Kudu 대시보드는 Azure 포털에서 이용할 수 있습니다. 앱에 대한 블레이드를 열고 **도구**를 클릭한 다음**Kudu**를 클릭합니다.
* Visual Studio 사용자라면 [Visual Studio를 사용하는 Azure App Service 웹 앱의 문제 해결][troubleshoot-web-app]에서 디버깅 및 문제 해결 팁을 얻으시기 바랍니다.

## 보안 고려사항
이 섹션에서는 이 문서에 설명된 Azure 서비스에 대한 보안 고려사항을 다룹니다. 이 섹션에 포함되지 않은 추가적인 고려사항은 [Azure App 서비스의 앱 보안][app-service-security]을 참조하시기 바랍니다.

### SQL Database 감사
감사 기능을 통해 규제를 준수하고 비즈니스 문제나 보안 침해 가능성을 의미할 수 있는 불일치성이나 불규칙성을 발견할 수 있습니다. [SQL DB 감사 시작하기][sql-audit]를 참조하시기 바랍니다.

### 배포 슬롯
각각의 배포 슬롯은 공용 IP 주소를 갖습니다. [Azure Active Directory 로그인][aad-auth]을 통해 프로덕션용이 아닌 슬롯을 보호하여 귀하의 개발팀 및 DevOps팀만이 이러한 끝점에 접근할 수 있도록 합니다.

### 로깅
로그에는 절대로 사용자 암호나 기타 명의 도용에 이용될 수 있는 정보가 포함되지 않아야 합니다. 데이터를 저장하기 전에 이러한 정보를 삭제합니다.  

### SSL
App Service 앱에는 추가 비용 없이 `azurewebsites.net`의 하위 도메인의 SSL 끝점이 포함됩니다. SSL 끝점에는 `*.azurewebsites.net` 도메인에 대한 와일드카드 인증서가 포함됩니다.  사용자 지정 도메인 이름을 사용하려면 해당 사용자 지정 도메인에 대한 인증서를 제공해야 합니다. 가장 간단한 방법은 Azure 포털에서 직접 인증서를 구매하는 것입니다. 다른 인증서 기관의 인증서를 불러오는 방법도 있습니다. 자세한 내용은 [Azure App Service용 SSL 인증서 구입 및 구성][ssl-cert]을 참조하시기 바랍니다.

보안 모범 사례는 앱이 HTTP 요청을 리디렉션하여 HTTPS를 강제 실행하는 것입니다. 이를 앱 내에서 실행하거나 [Azure App Service 앱을 위한 HTTPS 사용 설정][ssl-redirect]에 설명된 URL 다시 쓰기 규칙을 사용할 수 있습니다.

### 인증
Azure AD, Facebook, Google, Twitter와 같은 신원 제공자(IDP)를 통한 인증을 권장합니다. 인증 흐름은 OAuth 2나 OpenID Connect (OIDC)를 사용합니다. Azure AD는 사용자 및 그룹을 관리하고, 응용 프로그램 역할을 생성하고, 온-프레미스 ID를 통합하고, Office 365나 Skype for Business와 같은 백엔드 서비스를 사용하는 기능을 제공합니다.

응용 프로그램이 사용자 로그인과 자격 증명을 직접 관리하도록 할 경우 잠재적인 공격 대상이 될 수 있으므로 피해야 합니다. 최소한의 조건으로, 이메일 확인, 암호 복구, 다중 요소 인증을 갖추고, 암호 강도를 확인하며, 암호 해시를 안전하게 저장할 필요가 있습니다. 규모가 큰 신원 제공자는 이러한 모든 서비스를 제공하며, 자사의 보안 사례를 지속적으로 모니터링하고 개선합니다.

[App Service 인증][app-service-auth]을 이용하여 OAuth/OIDC 인증 흐름을 수행하는 것을 고려해 보시기 바랍니다. App Service 인증의 장점은 다음과 같습니다.

* 구성이 간편합니다.
* 단순한 인증 시나리오에 대해서는 코드를 요구하지 않습니다.
* 사용자를 대신해 리소스를 사용하기 위한 OAuth 접근 토큰을 이용한 위임 인증을 지원합니다.
* 내장 토큰 캐시를 제공합니다.

App Service 인증의 한계는 다음과 같습니다. 

* 제한된 사용자 지정 옵션.
* 위임 인증은 로그인 세션당 하나의 백엔드 리소스로 제한됩니다.
* 여러 IDP를 이용할 경우, 홈 영역 검색에 대해 내장된 메커니즘이 없습니다.
* 다중 테넌트 시나리오의 경우 응용 프로그램은 토큰 발행자를 확인하기 위한 로직을 수행해야 합니다.

## 솔루션 배포
이 아키텍처를 위한 Resource Manager 템플릿 샘플은 [GitHub에서 얻을 수 있습니다][paas-basic-arm-template].

PowerShell을 사용하여 템플릿을 배포하려면 다음과 같은 명령어를 실행합니다.

```
New-AzureRmResourceGroup -Name <resource-group-name> -Location "West US"

$parameters = @{"appName"="<app-name>";"environment"="dev";"locationShort"="uw";"databaseName"="app-db";"administratorLogin"="<admin>";"administratorLoginPassword"="<password>"}

New-AzureRmResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile .\PaaS-Basic.json -TemplateParameterObject  $parameters
```

자세한 내용은 [Azure Resource Manager 템플릿을 이용한 리소스 배포][deploy-arm-template]를 참조하시기 바랍니다.

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
[sql-db-scale]: /azure/sql-database/sql-database-scale-up-powershell
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
[vsts]: https://www.visualstudio.com/features/vso-cloud-load-testing-vs.aspx
[web-app-autoscale]: /azure/app-service-web/web-sites-scale
[web-app-backup]: /azure/app-service-web/web-sites-backup
[web-app-log-stream]: /azure/app-service-web/web-sites-enable-diagnostic-log#streamlogs
[0]: ../_images/blueprints/paas-basic-web-app.png "Architecture of a basic Azure web application"
[1]: ../_images/blueprints/paas-basic-web-app-staging-slots.png "Swapping slots for production and staging deployments"
