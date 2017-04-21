---
title: Application roles
description: How to perform authorization using application roles
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 02/16/2016
ms.author: pnp

pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: signup
pnp.series.next: authorize
---
# 응용 프로그램 역할

[![GitHub](../_images/github.png) 샘플 코드][sample application]

응용 프로그램 역할은 사용자들에게 권한을 지정하는 데 사용됩니다. 예를 들면, [Tailspin Surveys][Tailspin] 응용 프로그램은 다음의 역할을 정의합니다:

•	관리자. 해당 테넌트에 속한 설문조사에서 모든 CRUD 연산을 수행할 수 있습니다.

•	작성자. 새 설문조사를 만들 수 있습니다.

•	독자. 해당 테넌트에 속한 설문조사를 읽을 수 있습니다.


[권한 부여](https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/authorize)과정에서 그 역할이 결국 사용 권한로 바뀐 것을 볼 수 있습니다. 그런데, 첫 번째 의문은 어떻게 그 역할을 지정하고 관리하느냐 하는 것입니다. 우리는 3가지 주요 옵션을 확인했습니다:

* [Azure AD 앱 역할](#roles-using-azure-ad-app-roles)

* [Azure AD 보안 그룹](#roles-using-azure-ad-security-groups)

* [응용 애플리케이션 역할 관리자](#roles-using-an-application-role-manager).

## Azure AD 앱 역할을 사용한 역할
이 방식은 우리가 Taispin Surveys 앱에서 사용한 접근 방식입니다.

이 접근 방식에서, SaaS 공급자는 응용 프로그램 매니페스트에 응용 프로그램 역할을 추가하는 방법으로 그 역할을 정의합니다.  고객이 가입하고 나면 고객의 AD 디렉터리에 대한 관리자가 그 역할에 사용자들을 지정합니다. 어떤 사용자가 로그인하면 그 사용자에 지정된 역할이 클레임이 되어 전달됩니다.

> [!참고]
> 고객이 Azure AD Premium을 가진 경우, 관리자는 어떤 역할에 보안 그룹을 지정할 수 있고, 그러면 그룹 구성원들은 그 앱의 역할을 상속받습니다. 그룹 소유자가 AD 관리자일 필요가 없기 때문에 이는 역할을 관리하는데 있어서 편리한 방법입니다.
> 
> 

이 접근 방식의 장점:

•	심플 프로그래밍 모델

•	역할은 응용 프로그램별로 다릅니다. 어느 한 응용 프로그램에 대한 역할 클레임이 다른 응용 프로그램으로 전달되지 않습니다.

•	고객이 AD 테넌트에서 그 응용 프로그램을 제거할 경우, 역할은 사라집니다.

•	그 응용 프로그램은 사용자 프로필을 읽는 것 외에 추가적인 Active Directory 권한을 필요로 하지 않습니다.


결점:

•	Azure AD Premium이 없는 고객은 역할에 보안 그룹을 지정할 수 없습니다. 이러한 고객들의 경우 모든 사용자 지정이 AD 관리자에 의해 이뤄져야 합니다.

•	웹 앱과 분리된 백 엔드 웹 API인 경우, 웹 앱에 대한 역할 지정은 웹 API에 적용되지 않습니다. 더 자세한 내용은 
 [백 엔드 웹 API 보안](https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/web-api)을 참조하세요.

### 구현
**역할 정의.** SaaS 공급자는 [응용 프로그램 매니페스트](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-application-manifest)에서 앱 역할을 명시합니다. 예를 들면, Survey 앱에서 매니페스트 항목은 다음과 같습니다.

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

역할 클레임에서 `value`의 속성이 나타납니다. `id`의 속성은 정해진 역할에 대한 고유 식별자입니다. 항상 `id`에 대한 새로운 GUID 값을 생성합니다.

**사용자 지정**. 신규 고객이 가입하면, 고객의 AD 테넌트에 응용 프로그램이 등록됩니다. 이 때 해당 테넌트의 AD 관리자는 그 역할에 사용자를 지정할 수 있습니다.

> [!참고]
> 앞에서 언급했듯이, Azure AD Premium이 있는 고객들은 역할에 보안 그룹을 지정할 수 있습니다.
> 
> 

Azure 포털에서 캡쳐한 다음 화면은 3 종류의 사용자를 보여줍니다. Alice는 어떤 역할에 직접 지정되었습니다. Bob은 어떤 역할에 지정된 "Survey Admin"이라는 보안 그룹의 구성원으로서 그 역할을 상속받았습니다. Charles는 어떤 역할도 지정되지 않았습니다.

![Assigned users](./images/role-assignments.png)

> [!참고]
> 또는, 응용 프로그램은 Azure AD Graph API를 사용하여 역할을 프로그래밍 방식으로 지정할 수 있습니다. 그러나, 이 방법은 응용 프로그램으로 하여금 고객의 AD 디렉터리에 대한 권한을 확보하도록 합니다. 그러한 권한이 있는 응용 프로그램은 짓궂은 장난을 할 수 있습니다 - 고객은 앱이 자신의 디렉터리를 망치지 않을 것으로 믿고 있습니다. 많은 고객들이 이 수준의 액세스 부여를 꺼릴 수 있습니다.
> 
> 

**역할 클레임 획득**. 사용자가 로그인 하면 응용 프로그램은 사용자가 지정받은 역할을 전달받으며, 이 때 클레임 타입은 `http://schemas.microsoft.com/ws/2008/06/identity/claims/role`입니다. 

사용자는 여러 역할을 가질 수도 있고 역할이 전혀 없을 수도 있습니다. 인증 코드 단계에서 사용자가 역할 클레임을 단 하나만 갖는다고 가정하지 마십시오. 대신 특정 클레임 값이 존재하는지 여부를 확인하는 코드를 적습니다.

```csharp
if (context.User.HasClaim(ClaimTypes.Role, "Admin")) { ... }
```

## Azure AD 보안 그룹을 사용한 역할
이 접근 방식에서, 역할은 AD 보안 그룹으로 표시됩니다. 응용 프로그램은 보안 그룹 멤버십에 근거한 사용자들에게 권한을 지정합니다.

장점:

•	이 접근 방식에서 Azure AD Premium이 없는 고객은 역할 지정을 관리하기 위해 보안 그룹을 이용할 수 있습니다.

단점:

•	복잡성 모든 테넌트가 서로 다른 그룹 클레임을 전송하기 때문에, 각각의 테넌트에 대해서 앱은 어떤 보안 그룹이 어떤 응용 프로그램 역할에 해당하는지를 추적해야 합니다.

•	고객이 자신의 AD 테넌트에서 응용 프로그램을 삭제할 경우, 보안 그룹은 자신의 AD 디렉터리에 남습니다.


### 구현
응용 프로그램 매니페스트에서, "SecurityGroup"에 `groupMembershipClaims` 속성을 설정하세요. 이는 AAD에서 그룹 멤버십 클레임을 획득하는 데 필요합니다.

```
{
   // ...
   "groupMembershipClaims": "SecurityGroup",
}
```

신규 고객이 가입하면, 응용 프로그램은 고객이 응용 프로그램에서 필요로 하는 역할에 대한 보안 그룹을 만들 것을 지시합니다. 그러면 고객은 응용 프로그램에 그룹 개체의 ID를 입력해야 합니다. 응용 프로그램은 테넌트마다 그룹 ID를 응용 프로그램 역할과 연결 짓는 표에 이 값을 저장합니다.

> [!참고]
> 또는, 응용 프로그램은 Azure AD Graph API를 사용하여 그룹을 프로그래밍 방식으로 만들 수 있습니다. 이 방식은 오류가 일어날 가능성이 낮습니다. 그러나, 이 방법은 응용 프로그램으로 하여금 고객의 AD 디렉터리에 대한 "모든 그룹 읽기/쓰기" 권한을 확보하게 합니다. 많은 고객들이 이 수준의 액세스 부여를 꺼릴 수 있습니다.
> 
> 

사용자가 로그인 하면:

1.	응용 프로그램은 클레임으로 사용자 그룹을 수신합니다. 각 클레임의 값은 그룹의 개체 ID입니다.
2.	Azure AD는 토큰에서 전송된 그룹의 수를 제한합니다. 그룹의 수가 이 한도를 넘을 경우, Azure AD는 특별한 "초과" 클레임을 전송합니다. "초과" 클레임이 있을 경우, 응용 프로그램은 Azure AD Graph API에 해당 사용자가 속한 모든 그룹을 수용할지 질의해야 합니다. 자세한 사항은 "그룹 클레임 초과" 절에서 [클라우드 응용 프로그램에서 AD 그룹을 사용한 권한 부여]를 참조하세요. 
3.	응용 프로그램은 자체 데이터베이스에서 개체 ID를 검색하여 사용자에 지정할 해당 응용 프로그램 역할을 찾습니다.
4.	응용 프로그램은 응용 프로그램 역할을 표시하는 사용자 계정에 사용자 지정 클레임 값을 추가합니다. 예: 
 `survey_role` = "SurveyAdmin".

A권한 부여 정책은 그룹 클레임이 아니라 사용자 지정 역할 클레임을 사용해야 합니다.

## 응용 프로그램 역할 관리자를 사용한 역할
이 접근 방식으로는 응용 프로그램 역할이 Azure AD에 전혀 저장되지 않습니다. 대신, 응용 프로그램은 자체 DB에 사용자에 대한 역할 지정을 저장합니다 — 예를 들면, ASP.NET ID에서e **RoleManager** 사용.

장점: 

•	앱은 역할 및 사용자 지정을 모두 제어합니다.

결점:

•	유지 관리가 더 복잡하고 어렵습니다.

•	역할 지정을 관리하는데 AD 보안 그룹을 사용할 수 없습니다.

•	사용자가 추가되거나 제거되기 때문에 테넌트의 AD 데렉터리와의 동기화를 피할 수 있는 응용 프로그램 데이터베이스에 사용자 정보를 저장합니다.  


이 접근 방식에 대한 예가 많이 있습니다. 예를 들면, [인증받은 ASP.NET MVC app과 SQL DB를 생성하고 Azure 앱 서비스에 배포하기](https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database)를 참조하세요.

[**다음**][authorization]

<!-- Links -->
[Tailspin]: tailspin.md

[authorization]: authorize.md
[Securing a backend web API]: web-api.md
[Create an ASP.NET MVC app with auth and SQL DB and deploy to Azure App Service]: /azure/app-service-web/web-sites-dotnet-deploy-aspnet-mvc-app-membership-oauth-sql-database/
[application manifest]: /azure/active-directory/active-directory-application-manifest/
[sample application]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps
