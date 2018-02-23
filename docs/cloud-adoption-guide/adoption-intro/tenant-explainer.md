---
title: "설명: Azure Active Directory 테넌트란?"
description: "Azure에서 IDaaS(Identity as a Service)를 제공하는 Azure Active Directory의 내부 기능 설명"
author: petertay
ms.openlocfilehash: ce5a33b92047e1f360eee8fcbc7a726bcf8cd19f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-active-directory-tenant"></a>설명: Azure Active Directory 테넌트란?

[Azure 작동 방식](azure-explainer.md) 설명 문서에서는 Azure가 사용자를 대신해서 가상화된 하드웨어 및 소프트웨어를 실행하는 서버 및 네트워킹 하드웨어의 컬렉션이라는 사실을 배웠습니다. 또한 일부 서버가 Azure 리소스의 생성, 읽기, 업데이트 및 삭제를 관리하는 분산된 오케스트레이션 응용 프로그램을 실행한다는 사실도 배웠습니다.

그러나 예상할 수 있는 것처럼 Azure가 단지 리소스에 대해 이러한 작업 중 하나를 수행하는 것만은 아닙니다. Azure는 **Azure AD**(Azure Active Directory)라는 신뢰할 수 있는 디지털 ID 서비스를 사용하여 이러한 작업에 대한 액세스를 제한합니다. Azure AD는 사용자 이름, 암호, 프로필 데이터 및 기타 정보를 저장합니다. Azure AD 사용자는 **테넌트**로 조각화됩니다. 테넌트는 일반적으로 조직과 연결된 Azure AD의 안전한 전용 인스턴스를 나타내는 논리적 구문입니다.

테넌트를 만들기 위해서는 Azure에 **권한 있는 계정**이 필요합니다. 이 권한 있는 계정은 Azure 계정 또는 기업계약과 연결됩니다. 이러한 계정은 둘 다 청구 구문으로, Azure AD에 저장되지 않고 매우 안전한 청구 데이터베이스에 저장됩니다. 

테넌트를 만든 후에는 테넌트에 대해 **테넌트 ID**가 생성되고 매우 안전한 내부 Azure AD Database에 저장됩니다. 권한 있는 계정 소유자는 Azure Portal에 로그인한 후 새로 만든 Azure AD 테넌트에 사용자를 추가할 수 있습니다. 

대부분의 기업은 이미 하나 이상의 ID 관리 서비스, 일반적으로 AD DS(Active Directory Domain Services)를 이미 보유하고 있을 것입니다. Azure AD는 AD DS에서 사용자 ID를 동기화 또는 페더레이션할 수 있으므로 기업에서는 두 환경에서 별도로 ID를 관리할 필요가 없습니다. 이 내용은 디지털 ID에 대한 중급 및 고급 채택 단계 문서에 자세히 설명되어 있습니다.

## <a name="next-steps"></a>다음 단계

* Azure AD 테넌트에 대해 배웠으므로 기본 채택 단계의 첫 번째 단계로, [Azure Active Directory 테넌트를 가져오는 방법][how-to-get-aad-tenant]을 알아봅니다. 그런 다음 [Azure AD 테넌트에 대한 디자인 지침](tenant.md)을 검토합니다.

<!-- Links -->
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json