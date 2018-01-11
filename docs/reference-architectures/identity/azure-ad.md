---
title: "Azure Active Directory와 온-프레미스 AD 도메인 통합"
description: "Azure Active Directory를 사용하여 안전한 하이브리드 네트워크 아키텍처를 구현하는 방법입니다."
author: telmosampaio
pnp.series.title: Identity management
ms.date: 11/28/2016
pnp.series.next: adds-extend-domain
pnp.series.prev: ./index
cardTitle: Integrate on-premises AD with Azure AD
ms.openlocfilehash: dd4cf0369974ea68d240ed294b1c50972d361d74
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="integrate-on-premises-active-directory-domains-with-azure-active-directory"></a>Azure Active Directory와 온-프레미스 Active Directory 도메인 통합

Azure AD(Azure Active Directory)는 클라우드 기반의 다중 테넌트 디렉터리 및 ID 서비스입니다. 이 참조 아키텍처에서는 클라우드 기반 ID 인증을 제공하기 위해 온-프레미스 Active Directory 도메인과 Azure AD를 통합하는 모범 사례를 보여 줍니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

[![0]][0] 

*이 아키텍처의 [Visio 파일][visio-download]을 다운로드합니다.*

> [!NOTE]
> 편의상 이 다이어그램에서는 인증 및 ID 페더레이션의 일부로 발생할 수 있는 프로토콜 관련 트래픽이 아니라 Azure AD와 직접적으로 관련된 연결만 보여 줍니다. 예를 들어 웹 응용 프로그램이 Azure AD를 통해 요청을 인증하도록 웹 브라우저를 리디렉션할 수 있습니다. 일단 인증되면 해당 요청은 적절한 ID 정보와 함께 웹 응용 프로그램으로 다시 전달할 수 있습니다.
> 

이 참조 아키텍처의 일반적인 용도는 다음과 같습니다.

* Azure에 배포된 웹 응용 프로그램으로, 조직에 속한 원격 사용자에 대한 액세스를 제공합니다.
* 최종 사용자를 위한 셀프 서비스 기능 구현(예: 암호 재설정 및 그룹 관리 위임). Azure AD Premium 버전이 필요합니다.
* 온-프레미스 네트워크와 응용 프로그램의 Azure VNet이 VPN 터널 또는 ExpressRoute 회로를 통해 연결되지 않은 아키텍처

> [!NOTE]
> Azure AD는 현재 사용자 인증만 지원합니다. SQL Server와 같은 일부 응용 프로그램 및 서비스에는 컴퓨터 인증이 필요할 수 있습니다. 이 경우 이 솔루션은 적절하지 않습니다.
> 

추가 고려 사항은 [온-프레미스 Active Directory를 Azure와 통합하기 위한 솔루션 선택][considerations]을 참조하세요. 

## <a name="architecture"></a>건축

이 아키텍처의 구성 요소는 다음과 같습니다.

* **Azure AD 테넌트**. 조직에서 만든 [Azure AD][azure-active-directory]의 인스턴스입니다. 온-프레미스 Active Directory에서 복사한 개체를 저장하여 클라우드 응용 프로그램에 대한 디렉터리 서비스의 역할을 하며 ID 서비스를 제공합니다.
* **웹 계층 서브넷**. 이 서브넷은 웹 응용 프로그램을 실행하는 VM을 보유합니다. Azure AD는 이 응용 프로그램에 대한 ID broker의 역할을 수행할 수 있습니다.
* **온-프레미스 AD DS 서버**. 온-프레미스 디렉터리 및 ID 서비스입니다. AD DS 디렉터리는 Azure AD와 동기화되어 온-프레미스 사용자를 인증할 수 있습니다.
* **Azure AD Connect 동기화 서버**. [Azure AD Connect][azure-ad-connect] 동기화 서비스를 실행하는 온-프레미스 컴퓨터입니다. 이 서비스는 온-프레미스 Active Directory에 보유된 정보를 Azure AD에 동기화합니다. 예를 들어 온-프레미스 그룹 및 사용자를 프로비전하거나 프로비전 해제하는 경우 이러한 변경 내용이 Azure AD로 전파됩니다. 
  
  > [!NOTE]
  > 보안상의 이유로 Azure AD는 사용자 암호를 해시로 저장합니다. 사용자가 암호 재설정을 요구하는 경우 이 작업은 온-프레미스에서 수행되어야 하며 새 해시는 Azure AD로 보내야 합니다. Azure AD Premium 버전에는 사용자가 자신의 암호를 다시 설정할 수 있도록 이 작업을 자동화할 수 있는 기능이 포함되어 있습니다.
  > 

* **N 계층 응용 프로그램용 VM**. 배포에는 N 계층 응용 프로그램용 인프라가 포함됩니다. 이러한 리소스에 대한 자세한 내용은 [N 계층 아키텍처를 위한 VM 실행][implementing-a-multi-tier-architecture-on-Azure]을 참조하세요.

## <a name="recommendations"></a>권장 사항

대부분의 시나리오의 경우 다음 권장 사항을 적용합니다. 이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다. 

### <a name="azure-ad-connect-sync-service"></a>Azure AD Connect 동기화 서비스

Azure AD Connect 동기화 서비스를 사용하면 클라우드에 저장된 ID 정보가 온-프레미스에서 보유한 ID 정보와 일치하도록 유지할 수 있습니다. Azure AD Connect 소프트웨어를 사용하여 이 서비스를 설치합니다. 

Azure AD Connect 동기화를 구현하기 전에 조직의 동기화 요구 사항을 확인합니다. 예를 들어 동기화 대상, 동기화 원본 도메인 및 동기화 빈도 등이 있습니다. 자세한 내용은 [디렉터리 동기화 요구 사항 결정][aad-sync-requirements]을 참조하세요.

Azure AD Connect 동기화 서비스는 온-프레미스에서 호스팅된 VM이나 컴퓨터에서 실행할 수 있습니다. Active Directory 디렉터리에 있는 정보의 변동에 따라 Azure AD Connect 동기화 서비스의 로드는 Azure AD와의 초기 동기화 후에 높은 상태가 아닐 수 있습니다. VM에서 서비스를 실행하면 필요에 따라 서버의 크기를 쉽게 조정할 수 있습니다. 모니터링 고려 사항 섹션에서 설명한 대로 VM의 활동을 모니터하여 크기 조정이 필요한지 여부를 결정합니다.

포리스트에 여러 온-프레미스 도메인이 있는 경우 전체 포리스트에 대한 정보를 단일 Azure AD 테넌트에 저장하고 동기화하는 것이 좋습니다. 둘 이상의 도메인에서 발생하는 ID에 대한 정보를 필터링하여 각 ID가 중복되지 않고 Azure AD에 한 번만 표시되도록 합니다. 데이터가 동기화되면 중복으로 인해 불일치가 발생할 수 있습니다. 자세한 내용은 아래의 토폴로지 섹션을 참조하세요. 

필요한 데이터만 Azure AD에 저장되도록 필터링을 사용합니다. 예를 들어 조직에서 비활성 계정에 대한 정보를 Azure AD에 저장하지 않으려고 할 수도 있습니다. 필터링은 그룹, 도메인, OU(조직 단위) 또는 특성을 기준으로 수행할 수 있습니다. 필터를 결합하여 더 복잡한 규칙을 생성할 수 있습니다. 예를 들어 선택한 특성의 특정 값을 갖는 도메인에 있는 개체를 동기화할 수 있습니다. 자세한 내용은 [Azure AD Connect 동기화: 필터링 구성][aad-filtering]을 참조하세요.

AD Connect 동기화 서비스에 대한 고가용성을 구현하려면 보조 준비 서버를 실행합니다. 자세한 내용은 토폴로지 권장 사항 섹션을 참조하세요.

### <a name="security-recommendations"></a>보안 권장 사항

**사용자 암호 관리.** Azure AD Premium 버전은 암호 쓰기 저장을 지원하여 온-프레미스 사용자가 Azure Portal에서 셀프 서비스 암호 재설정을 수행할 수 있습니다. 이 기능은 조직의 암호 보안 정책을 검토한 후에만 사용하도록 설정해야 합니다. 예를 들어 자신의 암호를 변경할 수 있는 사용자를 제한할 수 있으며, 암호 관리 환경을 조정할 수 있습니다. 자세한 내용은 [조직의 요구에 맞게 암호 관리 사용자 지정][aad-password-management]을 참조하세요. 

**외부에서 액세스할 수 있는 온-프레미스 응용 프로그램 보호.** Azure AD 응용 프로그램 프록시를 사용하여 Azure AD를 통해 외부 사용자에게 온 프레미스 웹 응용 프로그램에 대한 제어된 액세스를 제공합니다. Azure 디렉터리에 유효한 자격 증명이 있는 사용자만 응용 프로그램을 사용할 수 있는 권한이 있습니다. 자세한 내용은 [Azure Portal에서 응용 프로그램 프록시 사용][aad-application-proxy] 문서를 참조하세요.

**의심스러운 활동 징후에 대한 적극적인 Azure AD 모니터링.**    Azure AD Identity Protection이 포함된 Azure AD Premium P2 버전을 사용하는 것이 좋습니다. ID 보호는 적응형 기계 학습 알고리즘 및 추론을 사용하여 ID가 손상되었음을 나타낼 수 있는 비정상 및 위험 이벤트를 검색합니다. 예를 들어 불규칙한 로그인 활동, 알 수 없는 출처 또는 의심스러운 활동이 있는 IP 주소의 로그인, 감염되었을 수 있는 장치의 로그인과 같이 잠재적으로 비정상적인 활동을 감지할 수 있습니다. ID 보호는 이 데이터를 사용하여 이러한 위험 이벤트를 조사하고 적절한 조치를 수행할 수 있도록 하는 보고서와 경고를 생성합니다. 자세한 내용은 [Azure Active Directory Identity Protection][aad-identity-protection]을 참조하세요.
  
Azure Portal에서 Azure AD의 보고 기능을 사용하여 시스템에서 발생하는 보안 관련 활동을 모니터링할 수 있습니다. 이러한 보고서를 사용하는 방법에 대한 자세한 내용은 [Azure Active Directory 보고 가이드][aad-reporting-guide]를 참조하세요.

### <a name="topology-recommendations"></a>토폴로지 권장 사항

Azure AD Connect를 구성하여 조직의 요구 사항과 가장 잘 맞는 토폴로지를 구현합니다. Azure AD Connect에서 지원하는 토폴로지는 다음과 같습니다.

* **단일 포리스트, 단일 Azure AD Directory**. 이 토폴로지에서 Azure AD Connect는 단일 온-프레미스 포리스트의 하나 이상의 도메인에 있는 개체와 ID 정보를 단일 Azure AD 테넌트와 동기화합니다. 이는 Azure AD Connect의 빠른 설치로 구현된 기본 토폴로지입니다.
  
  > [!NOTE]
  > 아래에서 설명한 대로 서버를 준비 모드에서 실행하지 않는 한, 여러 Azure AD Connect 동기화 서버를 사용하여 동일한 Azure AD 테넌트에 동일한 온-프레미스 포리스트의 다른 도메인을 연결하지 마세요.
  > 
  > 

* **여러 포리스트, 단일 Azure AD 디렉터리**. 이 토폴로지에서 Azure AD Connect는 여러 포리스트의 개체와 ID 정보를 단일 Azure AD 테넌트로 동기화합니다. 조직에 둘 이상의 온-프레미스 포리스트가 있는 경우 이 토폴로지를 사용합니다. 동일한 사용자가 둘 이상의 포리스트에 있더라도 Azure AD 디렉터리에서 고유한 각 사용자가 한 번만 표시되도록 ID 정보를 통합할 수 있습니다. 모든 포리스트에서 동일한 Azure AD Connect 동기화 서버를 사용합니다. Azure AD Connect 동기화 서버는 모든 도메인에 속할 필요는 없지만 모든 포리스트에서 연결할 수 있어야 합니다.
  
  > [!NOTE]
  > 이 토폴로지에서는 별도의 Azure AD Connect 동기화 서버를 사용하여 각 온-프레미스 포리스트를 단일 Azure AD 테넌트에 연결하지 마세요. 사용자가 둘 이상의 포리스트에 있는 경우 이로 인해 Azure AD에서 ID 정보가 중복될 수 있습니다.
  > 
  > 

* **여러 포리스트, 별도의 토폴로지**. 이 토폴로지는 별도의 포리스트에 있는 ID 정보를 단일 Azure AD 테넌트로 병합하여 모든 포리스트를 별도의 엔터티로 처리합니다. 이 토폴로지는 다른 조직의 포리스트를 결합하고 각 사용자에 대한 ID 정보가 하나의 포리스트에서만 유지되는 경우에 유용합니다.
  
  > [!NOTE]
  > 각 포리스트의 GAL(전체 주소 목록)이 동기화되면 한 포리스트의 사용자가 다른 포리스트에서 연락처로 존재할 수 있습니다. 이는 조직에서 Forefront Identity Manager 2010 또는 Microsoft Identity Manager 2016을 사용하여 GALSync를 구현한 경우에 발생할 수 있습니다. 이 시나리오에서는 사용자가 *Mail* 특성으로 식별되도록 지정할 수 있습니다. 또한 *ObjectSID* 및 *msExchMasterAccountSID* 특성을 사용하여 ID를 일치시킬 수도 있습니다. 이는 사용하도록 설정되지 않은 계정이 있는 리소스 포리스트가 하나 이상 있는 경우에 유용합니다.
  > 
  > 

* **준비 서버**. 이 구성에서는 Azure AD Connect 동기화 서버의 두 번째 인스턴스를 첫 번째 인스턴스와 병렬로 실행합니다. 이 구조에서 지원하는 시나리오는 다음과 같습니다.
  
  * 고가용성.
  * Azure AD Connect 동기화 서버의 새 구성 테스트 및 배포
  * 새 서버 도입 및 이전 구성 서비스 해제 
    
    이러한 시나리오에서 두 번째 인스턴스는 *준비 모드*에서 실행됩니다. 서버는 가져온 개체와 동기화 데이터를 해당 데이터베이스에 기록하지만, 데이터를 Azure AD에 전달하지 않습니다. 준비 모드를 해제하면 서버에서 Azure AD에 데이터를 쓰기 시작하고, 적절한 온-프레미스 디렉터리에 암호 쓰기 저장을 수행하기 시작합니다. 자세한 내용은 [Azure AD Connect 동기화: 운영 작업 및 고려 사항][aad-connect-sync-operational-tasks]을 참조하세요.

* **여러 Azure AD 디렉터리**. 조직에 대한 단일 Azure AD 디렉터리를 만드는 것이 좋지만, 별도의 Azure AD 디렉터리 간에 정보를 분할해야 하는 경우가 있을 수 있습니다. 이 경우 온-프레미스 포리스트의 각 개체가 단일 Azure AD 디렉터리에만 표시되도록 하여 동기화 및 암호 쓰기 저장 문제를 방지합니다. 이 시나리오를 구현하려면 각 Azure AD 디렉터리에 대해 개별 Azure AD Connect 동기화 서버를 구성하고, 각 Azure AD Connect 동기화 서버가 상호 배타적인 개체 집합에서 작동하도록 필터링을 사용합니다. 

이 토폴로지에 대한 자세한 내용은 [Azure AD Connect에 대한 토폴로지][aad-topologies]를 참조하세요.

### <a name="user-authentication"></a>사용자 인증

기본적으로 Azure AD Connect 동기화 서버는 온-프레미스 도메인과 Azure AD 간의 암호 동기화를 구성하고, Azure AD 서비스는 사용자가 온-프레미스에서 사용하는 것과 동일한 암호를 제공하여 사용자를 인증한다고 간주합니다. 이 방식은 많은 조직에서 적절하지만 조직의 기존 정책과 인프라를 고려해야 합니다. 예: 

* 조직의 보안 정책에 따라 암호 해시를 클라우드에 동기화하지 못하도록 금지할 수 있습니다.
* 회사 네트워크의 도메인 조인 컴퓨터에서 클라우드 리소스에 액세스할 때 원활한 SSO(Single Sign-On)가 필요할 수 있습니다.
* 조직에 이미 AD FS(Active Directory Federation Service) 또는 타사 페더레이션 공급자가 배포되어 있을 수 있습니다. 클라우드에서 보유한 암호 정보를 사용하는 대신 이 인프라를 사용하여 인증 및 SSO를 구현하도록 Azure AD를 구성할 수 있습니다.

자세한 내용은 [Azure AD Connect 사용자 로그인 옵션][aad-user-sign-in]을 참조하세요.

### <a name="azure-ad-application-proxy"></a>Azure AD 응용 프로그램 프록시 

Azure AD를 사용하여 온-프레미스 응용 프로그램에 대한 액세스를 제공합니다.

Azure AD 응용 프로그램 프록시 구성 요소에서 관리하는 응용 프로그램 프록시 커넥터를 사용하여 온-프레미스 웹 응용 프로그램을 공개합니다. 응용 프로그램 프록시 커넥터는 Azure AD 응용 프로그램 프록시에 대한 아웃바운드 네트워크 연결을 열고, 원격 사용자의 요청은 이 연결을 통해 Azure AD에서 웹앱으로 다시 라우팅됩니다. 이렇게 하면 온-프레미스 방화벽에서 인바운드 포트를 열 필요가 없으며 조직에서 노출하는 공격에 대한 취약성이 줄어듭니다.

자세한 내용은 [Azure AD 응용 프로그램 프록시를 사용하여 응용 프로그램 게시][aad-application-proxy]를 참조하세요.

### <a name="object-synchronization"></a>개체 동기화 

Azure AD Connect의 기본 구성은 [Azure AD Connect 동기화: 기본 구성 이해][aad-connect-sync-default-rules] 문서에서 지정한 규칙에 따라 로컬 Active Directory 디렉터리의 개체를 동기화합니다. 이러한 규칙이 충족되는 개체는 동기화되지만 다른 모든 개체는 무시됩니다. 예제 규칙 일부는 다음과 같습니다.

* 사용자 개체에는 고유한 *sourceAnchor* 특성이 있어야 하고, *accountEnabled* 특성이 채워져야 합니다.
* 사용자 개체에는 *sAMAccountName* 특성이 있어야 하고, *Azure AD_* 또는 *MSOL_* 텍스트로 시작할 수 없습니다.

Azure AD Connect는 User, Contact, Group, ForeignSecurityPrincipal 및 Computer 개체에 몇 가지 규칙을 적용합니다. 기본 규칙 집합을 수정해야 하는 경우 Azure AD Connect와 함께 설치된 동기화 규칙 편집기를 사용합니다. 자세한 내용은 [Azure AD Connect 동기화: 기본 구성 이해][aad-connect-sync-default-rules]를 참조하세요.

또한 사용자 고유의 필터를 정의하여 도메인 또는 OU에서 동기화할 개체를 제한할 수도 있습니다. 또는 [Azure AD Connect 동기화: 필터링 구성][aad-filtering]에서 설명한 대로 더 복잡한 사용자 지정 필터링을 구현할 수 있습니다.

### <a name="monitoring"></a>모니터링 

상태 모니터링은 온-프레미스에 설치된 다음 에이전트에서 수행됩니다.

* Azure AD Connect에서 동기화 작업에 대한 정보를 캡처하는 에이전트를 설치합니다. Azure Portal에서 Azure AD Connect Health 블레이드를 사용하여 상태와 성능을 모니터링합니다. 자세한 내용은 [동기화에 대한 Azure AD Connect Health 사용][aad-health]을 참조하세요.
* Azure에서 AD DS 도메인 및 디렉터리의 상태를 모니터링하려면 AD DS용 Azure AD Connect Health Agent를 온-프레미스 도메인의 컴퓨터에 설치합니다. 상태 모니터링을 위해 Azure Portal에서 Azure Active Directory Connect Health 블레이드를 사용합니다. 자세한 내용은 [AD DS와 함께 Azure AD Connect Health 사용][aad-health-adds]을 참조하세요. 
* AD FS용 Azure AD Connect Health Agent를 설치하여 온-프레미스에서 실행되는 서비스의 상태를 모니터링하고, Azure Portal에서 Azure Active Directory Connect Health 블레이드를 사용하여 AD FS를 모니터링합니다. 자세한 내용은 [AD FS와 함께 Azure AD Connect Health 사용][aad-health-adfs]을 참조하세요.

AD Connect Health Agent 설치 및 해당 요구 사항에 대한 자세한 내용은 [Azure AD Connect Health Agent 설치][aad-agent-installation]를 참조하세요.

## <a name="scalability-considerations"></a>확장성 고려 사항

Azure AD 서비스는 쓰기 작업을 처리하는 단일 주 복제본과 여러 개의 읽기 전용 보조 복제본을 사용하여 복제본에 기반한 확장성을 지원합니다. Azure AD는 보조 복제본에 대해 수행된 쓰기를 주 복제본으로 투명하게 리디렉션하고 최종 일관성을 제공합니다. 주 복제본에 대한 모든 변경 내용은 보조 복제본에 전파됩니다. Azure AD에 대한 작업 대부분이 쓰기가 아닌 읽기이므로 이 아키텍처에서는 크기가 효율적으로 조정됩니다. 자세한 내용은 [Azure AD: 지리적으로 중복된 고가용성 분산 클라우드 디렉터리][aad-scalability]를 참조하세요.

Azure AD Connect 동기화 서버의 경우 로컬 디렉터리에서 동기화할 가능성이 있는 개체의 수를 결정합니다. 개체 수가 100,000개 미만인 경우 Azure AD Connect와 함께 제공되는 기본 SQL Server Express LocalDB 소프트웨어를 사용할 수 있습니다. 더 많은 수의 개체가 있는 경우 SQL Server의 프로덕션 버전을 설치하고 Azure AD Connect의 사용자 지정 설치를 수행하여 기존 SQL Server 인스턴스를 사용하도록 지정해야 합니다.

## <a name="availability-considerations"></a>가용성 고려 사항

Azure AD 서비스는 지리적으로 분산되어 있으며, 자동 장애 조치를 통해 전 세계에 분산된 여러 데이터 센터에서 실행됩니다. 데이터 센터를 사용할 수 없게 되면 Azure AD는 지역적으로 분산된 둘 이상의 데이터 센터에서 디렉터리 데이터를 인스턴스 액세스에 사용할 수 있도록 합니다.

> [!NOTE]
> Azure AD Basic 및 Premium 서비스에 대한 SLA(서비스 수준 계약)는 최소 99.9% 가용성을 보장합니다. Azure AD의 체험 계층에 대한 SLA는 없습니다. 자세한 내용은 [Azure Active Directory에 대한 SLA][sla-aad]를 참조하세요.
> 
> 

토폴로지 권장 사항 섹션에서 설명한 대로 Azure AD Connect 동기화 서버의 두 번째 인스턴스를 준비 모드에서 프로비전하여 가용성을 높이는 것이 좋습니다. 

Azure AD Connect와 함께 제공되는 SQL Server Express LocalDB 인스턴스를 사용하지 않는 경우 SQL 클러스터링을 사용하여 고가용성을 달성하는 것이 좋습니다. 미러링 및 Always On과 같은 솔루션은 Azure AD Connect에서 지원하지 않습니다.

Azure AD Connect 동기화 서버의 고가용성을 달성하고 오류가 발생한 후 복구하는 방법에 대한 추가 고려 사항은 [Azure AD Connect 동기화: 운영 작업 및 고려 사항 - 재해 복구][aad-sync-disaster-recovery]를 참조하세요.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

Azure AD를 관리하는 데는 다음 두 가지 측면이 있습니다.

* 클라우드에서 Azure AD 관리
* Azure AD Connect 동기화 서버 유지 관리

Azure AD에서 클라우드의 도메인과 디렉터리를 관리하기 위해 제공하는 옵션은 다음과 같습니다. 

* **Azure Active Directory PowerShell 모듈**. 사용자 관리, 도메인 관리 및 SSO(Single Sign-On) 구성과 같은 일반적인 Azure AD 관리 작업을 스크립팅해야 하는 경우 이 [모듈][aad-powershell]을 사용합니다.
* **Azure Portal의 Azure AD 관리 블레이드**. 이 블레이드는 디렉터리의 대화형 관리 보기를 제공하며, 대부분의 Azure AD 측면을 제어하고 구성할 수 있습니다. 

Azure AD Connect에서 온-프레미스 컴퓨터에서 Azure AD Connect 동기화 서비스를 유지 관리하기 위해 설치하는 도구는 다음과 같습니다.
  
* **Microsoft Azure Active Directory Connect 콘솔**. 이 도구를 사용하면 Azure AD 동기화 서버의 구성을 수정하고, 동기화 수행 방식을 사용자 지정하며, 준비 모드를 사용하도록 설정하거나 해제하고, 사용자 로그인 모드를 전환할 수 있습니다. 온-프레미스 인프라를 사용하여 Active Directory FS 로그인을 사용하도록 설정할 수 있습니다.
* **Synchronization Service Manager**. 이 도구의 *작업* 탭을 사용하여 동기화 프로세스를 관리하고 프로세스의 모든 부분이 실패했는지 여부를 감지합니다. 이 도구를 사용하여 수동으로 동기화를 트리거할 수 있습니다. *커넥터* 탭을 사용하면 동기화 엔진이 연결된 도메인에 대한 연결을 제어할 수 있습니다.
* **동기화 규칙 편집기**. 이 도구를 사용하여 온-프레미스 디렉터리와 Azure AD 간에 복사할 때 개체가 변환되는 방식을 사용자 지정합니다. 이 도구를 사용하면 동기화에 대한 추가 특성 및 개체를 지정한 다음, 필터를 실행하여 동기화하거나 동기화하지 않아야 하는 개체를 결정할 수 있습니다. 자세한 내용은 [Azure AD Connect 동기화: 기본 구성 이해][aad-connect-sync-default-rules] 문서의 동기화 규칙 편집기 섹션을 참조하세요.

Azure AD Connect 관리에 대한 자세한 내용과 팁은 [Azure AD Connect 동기화: 기본 구성 변경에 대한 모범 사례][aad-sync-best-practices]를 참조하세요.

## <a name="security-considerations"></a>보안 고려 사항

조건부 액세스 제어를 사용하여 예기치 않은 출처의 인증 요청을 거부합니다.

- 사용자가 신뢰할 수 있는 네트워크 대신 인터넷과 같은 신뢰할 수 없는 위치에서 연결하려는 경우 [MFA(Azure Multi-Factor Authentication)][azure-multifactor-authentication]를 트리거합니다.

- 사용자의 장치 플랫폼 유형(iOS, Android, Windows Mobile, Windows)을 사용하여 응용 프로그램 및 기능에 대한 액세스 정책을 결정합니다.

- 사용자 장치의 사용/사용 안 함 상태를 기록하고, 이 정보를 액세스 정책 검사에 통합합니다. 예를 들어 사용자의 전화를 분실하거나 도난당한 경우 액세스 권한을 얻는 데 사용되지 않도록 '사용 안 함'으로 기록되어야 합니다.

- 그룹 멤버 자격에 따라 리소스에 대한 사용자 액세스를 제어합니다. [Azure AD 동적 멤버 자격 규칙][aad-dynamic-membership-rules]을 사용하여 그룹 관리를 간소화합니다. 이러한 작동 방식에 대한 간단한 개요는 [그룹에 대한 동적 멤버 자격 소개][aad-dynamic-memberships]를 참조하세요.

- 조건부 액세스 위험 정책을 Azure AD ID 보호와 함께 사용하여 비정상적인 로그인 활동 또는 다른 이벤트를 기반으로 한 고급 보호 기능을 제공합니다.

자세한 내용은 [Azure Active Directory 조건부 액세스][aad-conditional-access]를 참조하세요.

## <a name="deploy-the-solution"></a>솔루션 배포

이러한 권장 사항 및 고려 사항을 구현하는 참조 아키텍처에 대한 배포는 GitHub에서 사용할 수 있습니다. 이 참조 아키텍처는 테스트 및 실험에 사용할 수 있는 시뮬레이션된 온-프레미스 네트워크를 Azure에 배포합니다. 참조 아키텍처는 아래 지침에 따라 Windows 또는 Linux VM과 함께 배포할 수 있습니다. 

1. 아래 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fidentity%2Fazure-ad%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Azure 포털에서 링크가 열렸으면 일부 설정에 대한 값을 입력해야 합니다. 
   * **리소스 그룹** 이름이 매개 변수 파일에 이미 정의되어 있으므로 **새로 만들기**를 선택하고 텍스트 상자에 `ra-aad-onpremise-rg`를 입력합니다.
   * **위치** 드롭다운 상자에서 하위 지역을 선택합니다.
   * **템플릿 루트 Uri** 또는 **매개 변수 루트 Uri** 텍스트 상자를 편집하지 마세요.
   * **OS 유형** 드롭다운 상자에서 **Windows** 또는 **Linux**를 선택합니다.
   * 사용 약관을 검토한 후 **위에 명시된 사용 약관에 동의함** 확인란을 클릭합니다.
   * **구매** 단추를 클릭합니다.
3. 배포가 완료될 때가지 기다립니다.
4. 매개 변수 파일에는 하드 코딩된 관리자 사용자 이름과 암호가 포함되며, 모든 VM에서 둘 다 즉시 변경하는 것이 좋습니다. Azure Portal에서 각 VM을 클릭한 다음, **지원 + 문제 해결** 블레이드에서 **암호 재설정**을 클릭합니다. **모드** 드롭다운 상자에서 **암호 재설정**을 선택한 다음, 새 **사용자 이름** 및 **암호**를 선택합니다. **업데이트** 단추를 클릭하여 새 사용자 이름 및 암호를 보존합니다.

<!-- links -->

[implementing-a-multi-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[aad-agent-installation]: /azure/active-directory/active-directory-aadconnect-health-agent-install
[aad-application-proxy]: /azure/active-directory/active-directory-application-proxy-enable
[aad-conditional-access]: /azure/active-directory//active-directory-conditional-access
[aad-connect-sync-default-rules]: /azure/active-directory/active-directory-aadconnectsync-understanding-default-configuration
[aad-connect-sync-operational-tasks]: /azure/active-directory/active-directory-aadconnectsync-operations#staging-mode
[aad-dynamic-memberships]: https://youtu.be/Tdiz2JqCl9Q
[aad-dynamic-membership-rules]: /azure/active-directory/active-directory-accessmanagement-groups-with-advanced-rules
[aad-editions]: /azure/active-directory/active-directory-editions
[aad-filtering]: /azure/active-directory/active-directory-aadconnectsync-configure-filtering
[aad-health]: /azure/active-directory/active-directory-aadconnect-health-sync
[aad-health-adds]: /azure/active-directory/active-directory-aadconnect-health-adds
[aad-health-adfs]: /azure/active-directory/active-directory-aadconnect-health-adfs
[aad-identity-protection]: /azure/active-directory/active-directory-identityprotection
[aad-password-management]: /azure/active-directory/active-directory-passwords-customize
[aad-powershell]: https://msdn.microsoft.com/library/azure/mt757189.aspx
[aad-reporting-guide]: /azure/active-directory/active-directory-reporting-guide
[aad-scalability]: https://blogs.technet.microsoft.com/enterprisemobility/2014/09/02/azure-ad-under-the-hood-of-our-geo-redundant-highly-available-distributed-cloud-directory/
[aad-sync-best-practices]: /azure/active-directory/active-directory-aadconnectsync-best-practices-changing-default-configuration
[aad-sync-disaster-recovery]: /azure/active-directory/active-directory-aadconnectsync-operations#disaster-recovery
[aad-sync-requirements]: /azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements
[aad-topologies]: /azure/active-directory/active-directory-aadconnect-topologies
[aad-user-sign-in]: /azure/active-directory/active-directory-aadconnect-user-signin
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
[azure-multifactor-authentication]: /azure/multi-factor-authentication/multi-factor-authentication
[considerations]: ./considerations.md
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[sla-aad]: https://azure.microsoft.com/support/legal/sla/active-directory/v1_0/
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx


[0]: ./images/azure-ad.png "Azure Active Directory를 사용하는 클라우드 ID 아키텍처"