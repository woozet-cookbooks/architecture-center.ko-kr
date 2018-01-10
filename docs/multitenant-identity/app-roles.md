---
title: "응용 프로그램 역할"
description: "응용 프로그램 역할을 사용하여 권한 부여를 수행하는 방법"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: signup
pnp.series.next: authorize
ms.openlocfilehash: a39c64f003c26f860086701dd988a8bb21fab5bf
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="application-roles"></a>응용 프로그램 역할

[![GitHub](../_images/github.png) 샘플 코드][sample application]

응용 프로그램 역할은 사용자에게 사용 권한을 할당하는 데 사용됩니다. 예를 들어 [Tailspin Surveys][Tailspin] 응용 프로그램은 다음 역할을 정의합니다.

* 관리자. 해당 테넌트에 속하는 모든 설문 조사에 대한 모든 CRUD 작업을 수행할 수 있습니다.
* 작성자. 새 설문 조사를 만들 수 있습니다.
* 읽기 권한자. 해당 테넌트에 속하는 모든 설문 조사를 읽을 수 있습니다.

이 역할은 결국 [권한 부여]중에 사용 권한으로 변환되는 것을 확인할 수 있습니다. 그러나 첫 번째 질문은 역할을 할당 및 관리하는 방법입니다. 세 가지 기본 옵션을 확인했습니다.

* [Azure AD 앱 역할](#roles-using-azure-ad-app-roles)
* [Azure AD 보안 그룹](#roles-using-azure-ad-security-groups)
* [응용 프로그램 역할 관리자](#roles-using-an-application-role-manager)

## <a name="roles-using-azure-ad-app-roles"></a>Azure AD 앱 역할을 사용하는 역할
이 방식은 Tailspin Surveys 앱에서 사용하는 방식입니다.

이 방법에서 SaaS 공급자는 응용 프로그램 역할을 응용 프로그램 매니페스트에 추가하여 응용 프로그램 역할을 정의합니다. 고객이 등록하면 고객의 AD 디렉터리 관리자가 사용자를 해당 역할에 할당합니다. 사용자가 로그인하면 사용자에게 할당된 역할이 클레임으로 전송됩니다.

> [!NOTE]
> 고객에게 Azure AD Premium이 있는 경우 관리자는 보안 그룹을 역할에 할당할 수 있으며 해당 그룹의 구성원은 앱 역할을 상속합니다. 그룹 소유자가 AD 관리자일 필요가 없으므로 역할을 관리할 수 있는 편리한 방법입니다.
> 
> 

이 방법의 장점

* 단순한 프로그래밍 모델입니다.
* 역할이 응용 프로그램에 한정됩니다. 한 응용 프로그램에 대한 역할 클레임은 다른 응용 프로그램에 전송되지 않습니다.
* 고객이 해당 AD 테넌트에서 응용 프로그램을 제거하면 역할이 사라집니다.
* 응용 프로그램에는 사용자의 프로필을 읽는 것 외에 어떠한 추가 Active Directory 권한도 필요하지 않습니다.

단점

* Azure AD Premium이 없는 고객은 보안 그룹을 역할에 할당할 수 없습니다. 이러한 고객의 경우 모든 사용자 할당은 AD 관리자에 의해 수행되어야 합니다.
* 웹앱과 별도로 백 엔드 Web API가 있는 경우 웹앱에 대한 역할 할당은 Web API에 적용되지 않습니다. 이에 대한 자세한 내용은 [백 엔드 Web API 보안]을 참조하세요.

### <a name="implementation"></a>구현
**역할 정의.** SaaS 공급자는 [응용 프로그램 매니페스트]에서 앱 역할을 선언합니다. 예를 들어 다음은 Surveys 앱을 위한 매니페스트 항목입니다.

```
"appRoles": [
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Creators can create Surveys",
    "displayName": "SurveyCreator",
    "id": "1b4f816e-5eaf-48b9-8613-7923830595ad",
    "isEnabled": true,
    "value": "SurveyCreator"
  },
  {
    "allowedMemberTypes": [
      "User"
    ],
    "description": "Administrators can manage the Surveys in their tenant",
    "displayName": "SurveyAdmin",
    "id": "c20e145e-5459-4a6c-a074-b942bbd4cfe1",
    "isEnabled": true,
    "value": "SurveyAdmin"
  }
],
```

`value` 속성은 역할 클레임에 표시됩니다. `id` 속성은 정의된 역할에 대한 고유한 식별자입니다. 항상 `id`에 대한 새 GUID 값을 생성합니다.

**사용자 할당**. 새 고객이 등록하면 응용 프로그램이 고객의 AD 테넌트에 등록됩니다. 이 시점에서 해당 테넌트의 AD 관리자가 역할에 사용자를 할당할 수 있습니다.

> [!NOTE]
> 앞에서 설명한 대로 Azure AD Premium이 있는 고객은 보안 그룹을 역할에 할당할 수도 있습니다.
> 
> 

Azure Portal의 다음 스크린샷은 설문 조사 응용 프로그램의 사용자와 그룹을 보여 줍니다. Admin 및 Creator는 각각 SurveyAdmin 및 SurveyCreator 역할에 할당된 그룹입니다. Alice는 SurveyAdmin 역할에 직접 할당된 사용자입니다. Bob과 Charles는 역할에 직접 할당되지 않은 사용자입니다.

![사용자 및 그룹](./images/running-the-app/users-and-groups.png)

다음 스크린샷과 같이 Charles는 Admin 그룹에 속해 있으므로 SurveyAdmin 역할을 상속합니다. Bob의 경우 아직 역할이 할당되지 않았습니다.

![Admin 그룹 구성원](./images/running-the-app/admin-members.png)


> [!NOTE]
> 또는 응용 프로그램이 Azure AD Graph API를 사용하여 프로그래밍 방식으로 역할을 할당할 수 있습니다. 그러나 이 경우 응용 프로그램이 고객의 AD 디렉터리에 대한 쓰기 권한을 확보해야 합니다. 이러한 권한을 가진 응용 프로그램은 많은 오류를 만들 수 있으며, 앱이 해당 디렉터리를 손상하지 않을 것이라고 고객이 신뢰해야 합니다. 많은 고객이 이 수준의 액세스를 부여하는 것을 꺼려할 수 있습니다.
> 

**역할 클레임 가져오기**. 사용자가 로그인하는 경우 응용 프로그램은 유형이 `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`인 클레임에서 사용자의 할당된 역할을 수신합니다.  

사용자는 여러 역할을 보유하거나 역할이 없을 수 있습니다. 인증 코드에서 사용자가 정확한 한 개의 역할 클레임만 포함한다고 가정하지 마세요. 대신, 특정 클레임 값이 있는지 여부를 확인하는 코드를 작성합니다.

```csharp
if (context.User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
```

## <a name="roles-using-azure-ad-security-groups"></a>Azure AD 보안 그룹을 사용하는 역할
이 접근 방법에서는 역할이 AD 보안 그룹으로 표시됩니다. 응용 프로그램은 보안 그룹 구성원 자격에 따라 사용자에게 권한을 할당합니다.

장점

* Azure AD Premium이 없는 고객의 경우 이 접근 방법을 통해 고객은 보안 그룹을 사용하여 역할 할당을 관리할 수 있습니다.

단점

* 복잡성. 모든 테넌트가 서로 다른 그룹 클레임을 전송하므로 앱이 각 테넌트에 대해 어떤 보안 그룹이 어떤 응용 프로그램 역할에 해당하는지를 추적해야 합니다.
* 고객이 AD 테넌트에서 응용 프로그램을 제거하면 보안 그룹은 AD 디렉터리에 남아 있습니다.

### <a name="implementation"></a>구현
응용 프로그램 매니페스트에서 `groupMembershipClaims` 속성을 "SecurityGroup"으로 설정합니다. 이를 위해서는 AAD에서 그룹 구성원 자격 클레임을 가져와야 합니다.

```
{
   // ...
   "groupMembershipClaims": "SecurityGroup",
}
```

새 고객이 등록하면 응용 프로그램은 고객에게 응용 프로그램에 필요한 역할에 대한 보안 그룹을 만들라고 지시합니다. 그러면 고객은 응용 프로그램에 그룹 개체 ID를 입력해야 합니다. 응용 프로그램은 이 내용을 테넌트별로 그룹 ID를 응용 프로그램 역할에 매핑하는 테이블에 저장합니다.

> [!NOTE]
> 또는 응용 프로그램이 Azure AD Graph API를 사용하여 그룹을 프로그래밍 방식으로 만들 수 있습니다.  이렇게 하면 오류 가능성이 적습니다. 그러나 이 경우 응용 프로그램이 고객의 AD 디렉터리에 대한 "모든 그룹의 읽기 및 쓰기" 권한을 확보해야 합니다. 많은 고객이 이 수준의 액세스를 부여하는 것을 꺼려할 수 있습니다.
> 
> 

사용자가 로그인하는 경우

1. 응용 프로그램은 사용자의 그룹을 클레임으로 수신합니다. 각 클레임의 값은 그룹의 개체 ID입니다.
2. Azure AD는 토큰으로 전송한 그룹 수를 제한합니다. 그룹 수가 이 한도를 초과하면 Azure AD는 특별한 "초과" 클레임을 보냅니다. 해당 클레임이 있는 경우 응용 프로그램은 사용자가 속한 모든 그룹을 얻기 위해 Azure AD Graph API를 쿼리해야 합니다. 자세한 내용은 "그룹 클레임 초과" 섹션 아래, [AD 그룹을 사용하여 클라우드 응용 프로그램에서 권한 부여]를 참조하세요.
3. 응용 프로그램은 사용자에게 할당할 해당 응용 프로그램 역할을 찾기 위해 자신의 데이터베이스에서 개체 ID를 조회합니다.
4. 응용 프로그램은 사용자 지정 클레임 값을 응용 프로그램 역할을 표현하는 사용자 계정에 추가합니다. 예를 들어, `survey_role` = "SurveyAdmin"입니다.

권한 부여 정책은 그룹 클레임이 아닌, 사용자 지정 역할 클레임을 사용해야 합니다.

## <a name="roles-using-an-application-role-manager"></a>응용 프로그램 역할 관리자를 사용하는 역할
이 접근 방법에서는 응용 프로그램 역할이 Azure AD에 저장되지 않습니다. 대신, 응용 프로그램이 ASP.NET ID의 **RoleManager** 클래스 등을 사용하여 각 사용자에 대한 역할 할당을 해당 DB에 저장합니다.

장점

* 앱은 역할 및 사용자 할당에 대한 모든 권한을 보유합니다.

단점

* 좀더 복잡하고 유지 관리하기 어렵습니다.
* 역할 할당을 관리하는 데 AD 보안 그룹을 사용할 수 없습니다.
* 사용자를 추가하거나 제거하면 테넌트의 AD 디렉터리와 동기화되지 않을 수 있는 응용 프로그램 데이터베이스에 사용자 정보를 저장합니다.   


[**다음**][권한 부여]

<!-- Links -->
[Tailspin]: tailspin.md

[권한 부여]: authorize.md
[백 엔드 Web API 보안]: web-api.md
[응용 프로그램 매니페스트]: /azure/active-directory/active-directory-application-manifest/
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
