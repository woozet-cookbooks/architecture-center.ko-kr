---
title: 데이터 솔루션 보안
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 57897c31a8abdcd801874bf92d60360f7a80d1fa
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288925"
---
# <a name="securing-data-solutions"></a>데이터 솔루션 보안

많은 경우, 특히 온-프레미스 데이터 저장소에서만 작업하던 방식에서 전환하여 클라우드에서 데이터에 액세스할 수 있도록 하면, 해당 데이터에 대한 액세스가 증가하는 문제와 새로운 보안 유지 방법을 고민하게 됩니다.

## <a name="challenges"></a>과제

* 다양한 로그에 저장되어 있는 보안 이벤트를 중앙에서 모니터링하고 분석합니다.
* 여러 응용 프로그램 및 서비스에서 암호화 및 권한 부여 관리를 구현합니다.
* 중앙 집중식 ID 관리가 온-프레미스 또는 클라우드의 모든 솔루션 구성 요소에 작동되도록 합니다.

## <a name="data-protection"></a>데이터 보호

정보를 보호하는 첫 번째 단계는 보호할 대상을 식별하는 것입니다. 가장 중요한 데이터 자산이 어디에 위치하든, 식별하고, 보호하고, 모니터링할 수 있는 명확하고 간단하며 잘 소통되는 지침을 개발합니다. 조직의 업무 또는 수익성에 과도한 영향을 미치는 자산에 대해 가장 강력한 보호를 설정합니다. 이러한 자산을 고부가가치 자산 또는 HVA라고 합니다. HVA 수명 주기 및 보안 종속성을 엄격히 분석하고, 적절한 보안 제어 및 상태를 설정합니다. 마찬가지로 중요한 자산을 식별 및 분류하고, 보안 제어를 자동으로 적용할 기술 및 프로세스를 정의합니다.

보호해야 하는 데이터가 식별되면 *미사용* 데이터 및 *전송 중* 데이터를 보호하는 방법을 고려합니다.

* **미사용 데이터**: 물리적 미디어(자기 디스크 또는 광 디스크), 온-프레미스 또는 클라우드에 정적으로 존재하는 데이터입니다.
* **전송 중 데이터**: 네트워크를 통해, 서비스 버스를 통해(온-프레미스에서 클라우드로 또는 그 반대로) 또는 입/출력 프로세스 동안 구성 요소, 위치 또는 프로그램 간에 전송되는 데이터입니다.

미사용 데이터 또는 전송 중 데이터를 보호하는 방법에 대한 자세한 내용은 [Azure 데이터 보안 및 암호화 모범 사례](/azure/security/azure-security-data-encryption-best-practices)를 참조하세요.

## <a name="access-control"></a>Access Control

클라우드의 데이터 보호 방식은 기본적으로 ID 관리 및 액세스 제어를 통합하는 것입니다. 클라우드 서비스 유형이 다양하고, [하이브리드 클라우드](../scenarios/hybrid-on-premises-and-cloud.md)의 사용도 증가하는 가운데, ID 및 액세스 제어와 관련해서 다음과 같은 몇 가지 핵심 방식을 따라야 합니다.

* ID 관리를 중앙에서 집중적으로 수행합니다.
* SSO(Single Sign-On)를 사용합니다.
* 암호 관리를 배포합니다.
* 사용자에 대해 MFA(Multi-Factor Authentication)를 적용합니다.
* RBAC(역할 기반 액세스 제어)를 사용합니다.
* 사용자 위치, 장치 유형, 패치 수준 등과 관련된 추가 속성을 포함하는 사용자 ID의 기본 개념을 향상시키는 조건부 액세스 정책을 구성해야 합니다.
* Resource Manager를 사용하여 리소스가 만들어지는 위치를 제어합니다.
* 의심스러운 활동을 적극적으로 모니터링

자세한 내용은 [Azure Identity Management 및 액세스 제어 보안 모범 사례](/azure/security/azure-security-identity-management-best-practices)를 참조하세요.

## <a name="auditing"></a>감사

앞서 언급한 ID 및 액세스 모니터링 외에도, 클라우드에서 사용하는 서비스 및 응용 프로그램은 모니터링할 수 있는 보안 관련 이벤트를 생성하게 됩니다. 이러한 이벤트를 모니터링할 때의 가장 큰 해결 과제는 잠재적인 문제를 방지하거나 지난 문제를 해결하기 위해 방대한 로그를 처리하는 것입니다. 클라우드 기반 응용 프로그램은 많은 이동 부분을 포함하게 되며, 이러한 부분은 대개 일정 수준의 로깅 및 원격 분석을 생성합니다. 중앙 집중식 모니터링 및 분석을 사용하면 대량의 정보를 관리하고 이해하는 데 도움이 됩니다.

자세한 내용은 [Azure 로깅 및 감사](/azure/security/azure-log-audit)를 참조하세요.



## <a name="securing-data-solutions-in-azure"></a>Azure에서 데이터 솔루션 보안

### <a name="encryption"></a>암호화

**가상 머신**. [Azure Disk Encryption](/azure/security/azure-security-disk-encryption)을 사용하여 Windows 또는 Linux VM에서 연결된 디스크를 암호화합니다. 이 솔루션은 [Azure Key Vault](/azure/key-vault/)와 통합되어 디스크 암호화 키 및 비밀을 제어 및 관리할 수 있도록 합니다. 

**Azure Storage**. [Azure Storage Service Encryption](/azure/storage/common/storage-service-encryption)를 사용하여 Azure Storage에서 미사용 데이터를 자동으로 암호화할 수 있습니다. 암호화, 암호 해독 및 키 관리는 사용자에게 완전히 투명하게 처리됩니다. Azure Key Vault와 클라이언트 쪽 암호화를 사용하여 전송 중인 데이터도 보호할 수 있습니다. 자세한 내용은 [Microsoft Azure Storage용 클라이언트 쪽 암호화 및 Azure Key Vault](/azure/storage/common/storage-client-side-encryption)를 참조하세요.

**SQL Database** 및 **Azure SQL Data Warehouse**. TDE([투명한 데이터 암호화](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql))를 사용하여 응용 프로그램을 변경하지 않고도 데이터베이스, 연결된 백업 및 트랜잭션 로그 파일의 실시간 암호화 및 암호 해독을 수행할 수 있습니다. 또한 SQL Database는 [상시 암호화](/azure/sql-database/sql-database-always-encrypted-azure-key-vault)를 사용하여 클라이언트와 서버 간을 이동하는 동안, 그리고 데이터를 사용 중일 때 서버에서 중요한 미사용 데이터를 보호할 수 있습니다. Azure Key Vault를 사용하여 상시 암호화의 암호화 키를 저장할 수 있습니다. 

### <a name="rights-management"></a>권한 관리

[Azure Rights Management](/information-protection/understand-explore/what-is-azure-rms)는 암호화, ID 및 권한 부여 정책을 사용하여 파일 및 전자 메일을 보호하는 클라우드 기반 서비스입니다. 이 기능은 휴대폰, 태블릿 및 PC를 비롯한 여러 장치에서 작동합니다. 정보가 조직의 경계를 벗어나더라도 데이터가 계속 보호되므로 조직 내부 및 외부에서 정보를 보호할 수 있습니다.

### <a name="access-control"></a>Access Control

RBAC([역할 기반 액세스 제어](/azure/active-directory/role-based-access-control-what-is))를 사용하여 사용자 역할을 기준으로 Azure 리소스에 대한 액세스를 제어할 수 있습니다. Active Directory 온-프레미스를 사용하는 경우 [Azure AD와 동기화](/azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements)하여 사용자에게 온-프레미스 ID를 기준으로 하는 클라우드 ID를 제공할 수 있습니다.

[Azure Active Directory의 조건부 액세스](/azure/active-directory/active-directory-conditional-access-azure-portal)를 사용하여 특정 조건에 따라 사용자 환경의 응용 프로그램에 대한 액세스를 제어할 수 있습니다. 예를 들어, 정책 설명이 _계약자가 신뢰할 수 없는 네트워크에서 클라우드 앱에 액세스하려고 시도하는 경우 액세스를 차단합니다._ 형식일 수 있습니다. 

[Azure AD Privileged Identity Management](/azure/active-directory/active-directory-privileged-identity-management-configure)는 사용자와 사용자가 해당 관리 권한으로 수행하는 태스크 종류를 관리, 제어 및 모니터링하는 데 도움을 줄 수 있습니다. 이것은 조직에서 Azure AD, Azure, Office 365 또는 SaaS 앱에 대해 권한 있는 작업을 수행하고 해당 작업을 모니터링할 수 있는 사용자를 제한하는 중요한 단계입니다.

### <a name="network"></a>네트워크

전송 중인 데이터를 보호하려면 다른 위치 간에 데이터를 교환할 때 항상 SSL/TLS를 사용합니다. VPN(가상 사설망) 또는 [ExpressRoute](/azure/expressroute/)를 사용하여 온-프레미스와 클라우드 인프라 간에 전체 통신 채널을 격리해야 하는 경우도 있습니다. 자세한 내용은 [클라우드로 온-프레미스 데이터 솔루션 확장](../scenarios/hybrid-on-premises-and-cloud.md)을 참조하세요.

NSG([네트워크 보안 그룹](/azure/virtual-network/virtual-networks-nsg))를 사용하여 잠재적인 공격 벡터의 수를 줄일 수 있습니다. 네트워크 보안 그룹에는 원본 또는 대상 IP 주소, 포트 및 프로토콜에 따라 인바운드 또는 아웃바운드 네트워크 트래픽을 허용하거나 거부하는 보안 규칙 목록이 포함되어 있습니다. 

사용자의 가상 네트워크에서 들어오는 트래픽만 이러한 리소스에 액세스할 수 있도록 [Virtual Network 서비스 끝점](/azure/virtual-network/virtual-network-service-endpoints-overview)을 사용하여 Azure SQL 또는 Azure Storage 리소스의 보안을 유지합니다.

Azure VNet(Virtual Network) 내의 VM은 [가상 네트워크 피어링](/azure/virtual-network/virtual-network-peering-overview)을 사용하여 다른 VNet과 안전하게 통신할 수 있습니다. 피어링된 가상 네트워크 간의 네트워크 트래픽이 개인 전용입니다. 가상 네트워크 간의 트래픽이 Microsoft 백본 네트워크에서 유지됩니다.

자세한 내용은 [Azure 네트워크 보안](/azure/security/azure-network-security)을 참조하세요.

### <a name="monitoring"></a>모니터링

[Azure Security Center](/azure/security-center/security-center-intro)는 Azure 리소스, 네트워크 및 연결된 파트너 솔루션(예: 방화벽 솔루션)의 로그 데이터를 자동으로 수집, 분석 및 통합하여 실제 위협을 검색하고 가양성을 줄입니다. 

[Log Analytics](/azure/log-analytics/log-analytics-overview)는 중앙에서 로그에 액세스하도록 하고, 해당 데이터를 분석하고 사용자 지정 경고를 만들 수 있도록 합니다.

[Azure SQL Database 위협 감지](/azure/sql-database/sql-database-threat-detection)는 데이터베이스를 액세스하거나 악용하려는 비정상적이고 잠재적으로 해로운 시도를 나타내는 비정상적인 활동을 감지합니다. 보안 책임자 또는 지정된 다른 관리자는 의심스러운 데이터베이스 활동이 발생하는 즉시 알림을 받을 수 있습니다. 각 알림에서는 의심스러운 활동에 대한 세부 정보를 제공하고 해당 위협을 자세히 조사하고 완화하는 방법을 권장합니다.


