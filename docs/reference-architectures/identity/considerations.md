---
title: 온-프레미스 Active Directory를 Azure와 통합하기 위한 솔루션을 선택합니다.
description: 온-프레미스 Active Directory를 Azure와 통합하기 위한 참조 아키텍처를 비교합니다.
ms.date: 04/06/2017
ms.openlocfilehash: 413a5463d90547197c4b6834d353b4ecf61483ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a>온-프레미스 Active Directory를 Azure와 통합하기 위한 솔루션 선택

이 문서에서는 온-프레미스 AD(Active Directory) 환경을 Azure 네트워크와 통합하는 옵션을 비교합니다. 각 옵션에 대해 참조 아키텍처와 배포 가능한 솔루션을 제공합니다.

많은 조직에서 [AD DS(Active Directory Domain Services)][active-directory-domain-services] 를 사용하여 보안 경계에 포함된 사용자, 컴퓨터, 응용 프로그램 또는 기타 리소스와 연결된 ID를 인증합니다. 디렉터리 및 ID 서비스는 일반적으로 온-프레미스에서 호스트되지만 응용 프로그램이 일부는 온-프레미스에서, 일부는 Azure에서 호스트되는 경우 Azure의 인증 요청을 온-프레미스로 되돌려 보내는 데 시간이 지연될 수 있습니다. Azure에 디렉터리와 ID 서비스를 구현하면 이러한 대기 시간을 줄일 수 있습니다.

Azure에서는 Azure에 디렉터리와 ID 서비스를 구현하기 위한 두 가지 솔루션을 제공합니다. 

* [Azure AD][azure-active-directory]를 사용하여 클라우드에서 Active Directory 도메인을 만들고 이를 온-프레미스 Active Directory 도메인에 연결합니다. [Azure AD Connect][azure-ad-connect]에서 온-프레미스 디렉터리와 Azure AD를 통합합니다.

* AD DS를 도메인 컨트롤러로 실행하는 Azure에 VM을 배포하여 기존 온-프레미스 Active Directory 인프라를 Azure로 확장합니다. 이 아키텍처는 온-프레미스 네트워크와 Azure VNet(Virtual Network)이 VPN 또는 ExpressRoute 연결을 통해 연결되는 경우에 더 일반적입니다. 이 아키텍처는 다음과 같은 몇 가지 변형이 가능합니다. 

    - Azure에서 도메인을 만들어 온-프레미스 AD 포리스트에 조인합니다.
    - 온-프레미스 포리스트의 도메인이 신뢰하는 별도의 포리스트를 Azure에 만듭니다.
    - AD FS(Active Directory Federation Services) 배포를 Azure에 복제합니다. 

다음 섹션에서는 이러한 각 옵션에 대해 자세히 설명합니다.

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a>온-프레미스 도메인을 Azure AD와 통합

Azure AD(Azure Active Directory)를 사용하여 Azure에서 도메인을 만들어 온-프레미스 AD 도메인에 연결합니다. 

Azure AD 디렉터리는 온-프레미스 디렉터리의 확장이 아닙니다. 오히려 동일한 개체와 ID를 포함하는 복사본입니다. 이러한 온-프레미스 항목에 대한 변경 사항은 Azure AD로 복사되지만 Azure AD의 변경 사항은 온-프레미스 도메인으로 다시 복제되지 않습니다.

또한 온-프레미스 디렉터리를 사용하지 않고 Azure AD를 사용할 수 있습니다. 이러한 경우 Azure AD는 온-프레미스 디렉터리에서 복제된 데이터를 포함하지 않고 모든 ID 정보의 기본 출처 역할을 합니다.


**이점**

* 클라우드에서 AD 인프라를 유지할 필요가 없습니다. Azure AD가 Microsoft에서 완벽하게 관리 및 유지 관리될 수 있습니다.
* Azure AD는 온-프레미스에서 사용할 수 있는 것과 동일한 ID 정보를 제공합니다.
* Azure에서 인증이 발생할 수 있으므로 외부 응용 프로그램과 사용자가 온-프레미스 도메인에 연결할 필요성이 줄어듭니다.

**과제**

* ID 서비스가 사용자 및 그룹으로 제한됩니다. 서비스 및 컴퓨터 계정을 인증할 수 있는 기능이 없습니다.
* Azure AD 디렉터리를 동기화 상태로 유지하려면 온-프레미스 도메인과의 연결을 구성해야 합니다. 
* Azure AD를 통해 인증하려면 응용 프로그램을 다시 작성해야 할 수 있습니다.

**[자세히 알아보기...][aad]**

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a>온-프레미스 포리스트에 조인된 Azure의 AD DS

AD DS(AD Domain Services) 서버를 Azure에 배포합니다. Azure에서 도메인을 만들어 온-프레미스 AD 포리스트에 조인합니다. 

현재 Azure AD에서 구현되지 않은 AD DS 기능을 사용해야 하는 경우 이 옵션을 고려합니다. 

**이점**

* 온-프레미스에서 사용할 수 있는 동일한 ID 정보에 대한 액세스를 제공합니다.
* 온-프레미스 및 Azure에서 사용자, 서비스 및 컴퓨터 계정을 인증할 수 있습니다.
* 별도의 AD 포리스트를 관리할 필요가 없습니다. Azure의 도메인은 온-프레미스 포리스트에 속할 수 있습니다.
* 온-프레미스 그룹 정책 개체에 의해 정의된 그룹 정책을 Azure의 도메인에 적용할 수 있습니다.

**과제**

* 클라우드에서 고유한 AD DS 서버 및 도메인을 배포하고 관리해야 합니다.
* 클라우드의 도메인 서버와 온-프레미스에서 실행되는 서버 간에 동기화 대기 시간이 어느 정도 있을 수 있습니다.

**[자세히 알아보기...][ad-ds]**

## <a name="ad-ds-in-azure-with-a-separate-forest"></a>별도의 포리스트가 있는 Azure의 AD DS

AD DS(AD Domain Services) 서버를 Azure에 배포하지만 온-프레미스 포리스트와 분리된 별도의 Active Directory [포리스트][ad-forest-defn]를 만듭니다. 이 포리스트는 온-프레미스 포리스트의 도메인에서 신뢰할 수 있습니다.

이 아키텍처의 일반적인 용도는 클라우드에 저장된 개체 및 ID에 대한 보안 분리를 유지하는 것과 개별 도메인을 온-프레미스에서 클라우드로 마이그레이션하는 것입니다.

**이점**

* 온-프레미스 ID와 별도의 Azure 전용 ID를 구현할 수 있습니다.
* 온-프레미스 AD 포리스트에서 Azure로 복제할 필요가 없습니다.

**과제**

* 온-프레미스 ID를 Azure 내에서 인증받으려면 온-프레미스 AD 서버에 추가 네트워크 홉이 필요합니다.
* 클라우드에서 고유한 AD DS 서버 및 포리스트를 배포하고 포리스트 간에 적절한 신뢰 관계를 설정해야 합니다.

**[자세히 알아보기...][ad-ds-forest]**

## <a name="extend-ad-fs-to-azure"></a>AD FS를 Azure로 확장

AD FS(Active Directory Federation Services) 배포를 Azure에 복제하여 Azure에서 실행되는 구성 요소에 대해 페더레이션 인증 및 권한 부여를 수행할 수 있습니다. 

이 아키텍처의 일반적인 용도는 다음과 같습니다.

* 파트너 조직에서 사용자 인증 및 권한 부여
* 사용자가 조직 방화벽 외부에서 실행되는 웹 브라우저에서 인증할 수 있도록 허용
* 사용자가 모바일 장치와 같은 승인된 외부 장치에서 연결할 수 있도록 허용 

**이점**

* 클레임 인식 응용 프로그램을 활용할 수 있습니다.
* 인증 시 외부 파트너를 신뢰할 수 있는 기능을 제공합니다.
* 대규모 인증 프로토콜 집합과 호환됩니다.

**과제**

* Azure에서 고유한 AD DS, AD FS 및 AD FS 웹 응용 프로그램 프록시 서버를 배포해야 합니다.
* 이 아키텍처를 구성하는 것은 복잡할 수 있습니다.

**[자세히 알아보기...][adfs]**

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: https://msdn.microsoft.com/library/ms676906.aspx
[adfs]: ./adfs.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
