---
title: Identity Management for Multitenant Applications
description: >-
  Best practices for authentication, authorization, and identity management in
  multitenant apps.
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 06/02/2016

ms.author: pnp
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.next: tailspin
---
# 다중 테넌트 응용 프로그램에서 ID 관리

이 문서는 인증과 ID 관리에 Azure AD를 사용할 때 다중 테넌트에 대한 모범 사례에 대하여 설명합니다.

[![GitHub](../_images/github.png) 샘플 코드][sample application]

다중 테넌트 응용 프로그램을 구성할 때, 이제 모든 사용자가 한 테넌트에 속하기 때문에 첫 번째 직면하는 문제 중 하나가 사용자 ID 관리입니다. 예:

•	사용자가 자기 조직의 자격 증명으로 로그인합니다.

•	사용자는 자기 조직의 데이터를 액세스할 수 있지만, 다른 테넌트에 속한 데이터에는 접근하지 못합니다, 

•	조직은 응용 프로그램에 등록한 다음, 그 구성원들에게 응용 프로그램 역할을 지정합니다.


Azure Active Directory (Azure AD)는 이 모든 시나리오를 지원하는 좋은 기능들을 보유하고 있습니다.

이 문서에 동반하여, 우리는 다중 테넌트 앱에 대해서 완전한 [종단 간 구현][sample application]을 만들어냈습니다. 문서는 응용 프로그램 구성 과정에서 우리가 습득한 내용을 반영합니다. 응용 프로그램을 시작하려면, [Surveys 응용 프로그램 실행하기][running-the-app]를 참조하세요.

## 소개

예를 들면, 독자가 현재 클라우드에 호스트될 기업형 SaaS 응용 프로그램을 만들고 있다고 합시다. 물론, 그 응용 프로그램은 사용자가 있습니다:

![Users](./images/users.png)

그런데 그 사용자들은 조직에 속해 있습니다:

![Organizational users](./images/org-users.png)

예: Tailspin이 SaaS 응용 프로그램 구독을 판매합니다. Contoso와 Fabrikam이 앱에 등록합니다. Alice (`alice@contoso`) 가 로그인하면, 응용 프로그램은 Alice가 Contoso 소속이란 사실을 알아야 합니다.

* Alice는 Contoso 데이터를 액세스할 수 있어야 *합니다*.

* Alice는 Fabrikam 데이터를 액세스하면 *안 됩니다*.

이러한 지침은 로그인과 인증 처리를 위해 [Azure Active Directory][AzureAD] (Azure AD)를 사용하여 다중 테넌트 응용 프로그램에서 사용자 ID를 어떻게 관리해야 하는지 보여줄 것입니다.

## 다중 테넌트 지원이란?
*테넌트*는 사용자 그룹입니다. SaaS 응용 프로그램에서 테넌트는 응용 프로그램의 구독자 또는 고객입니다. 다중 테넌트는 다중 테넌트가 동일한 물리적 인스턴스를 공유하는 아키텍처입니다. 테넌트들이 물리적 리소스를(VM이나 저장소 등) 공유하더라도, 각 테넌트는 앱에 대하여 자기 소유의 논리적 인스턴스를 갖습니다.

일반적으로 응용 프로그램 데이터는 테넌트 안에 있는 사용자들끼리 공유하고, 다른 테넌트와는 공유하지 않습니다.

![Multitenant](./images/multitenant.png)

단일 테넌트 아키텍처와 비교해볼 때 이 아키텍처는 각 테넌트가 전용 물리적 인스턴스를 갖습니다. 단일 테넌트 아키텍처에서는 앱의 새 인스턴스를 스핀업(spinning up)하여 테넌트를 추가합니다.

![Single tenant](./images/single-tenant.png)

### 다중 테넌트 지원과 수평 확장
클라우드 크기를 조정하려면 물리적 인스턴스를 추가하는 것이 일반적입니다. 이를 수평 확정 또는 규모 확장이라고 합니다. 웹 앱을 살펴봅시다. 더 많은 트래픽을 처리하기 위해서, VM 서버를 추가하고 트래픽 처리를 부하 분산 장치에 맡길 수 있습니다. 각 VM은 웹 앱의 별도의 물리적 인스턴스를 실행합니다.

![Load balancing a web site](./images/load-balancing.png)

어떤 요청이든 어떤 인스턴스로도 경로가 설정될 수 있습니다. 동시에, 시스템은 단일의 논리적 인스턴스의 기능을 수행합니다. 사용자들에게 영향을 미치지 않고 VM을 해체하거나 새 VM을 등록할 수 있습니다. 이 아케텍처에서 각 물리적 인스턴스는 다중 테넌트이고, 인스턴스를 추가하여 크기를 확장할 수 있습니다. 한 인스턴스가 중단되더라도 다른 테넌트에 영향을 주면 안 됩니다.

## 다중 테넌트 앱에서의 ID
다중 테넌트 앱에서는 테넌트 맥락에서 사용자들을 고려해야 합니다.

**인증**

•	사용자가 자기 조직의 자격 증명으로 로그인합니다. 사용자는 앱에 새로운 사용자 프로필을 만들 필요가 없습니다.

•	같은 조직에 속한 사용자는 같은 테넌트에 포함됩니다.

•	사용자가 로그인하면 응용 프로그램은 사용자가 어떤 테넌트에 속해있는지 압니다.


**인증**

•	사용자 동작을 인증할 때(즉, 리소스 검토), 앱은 사용자의 테넌트를 고려해야 합니다.

•	사용자는 "관리자" 또는 "표준 사용자"와 같이 응용 프로그램에서 지정된 역할일 수 있습니다.  역할 지정은 SaaS 공급자가 아니라 고객에 의해서 관리되어야 합니다.


**예.**Contoso 직원인 Alice는 자기 브라우저에서 응용 프로그램을 탐색하고 “로그인” 단추를 클릭합니다. 회사의 자격 증명을(사용자 이름과 암호) 입력한 로그인 화면으로 Alice가 리디렉션됩니다. 이제 Alice는 `alice@contoso.com`으로 앱에 로그인되었습니다. 응용 프로그램은 Alice가 이 응용 프로그램의 관리 사용자라는 사실도 압니다. 관리자이기 때문에 Contoso에 속한 모든 리소스를 볼 수 있습니다. 하지만, 자신의 테넌트에서만 관리자이므로 Fabrikam의 리소스는 볼 수 없습니다.

이 지침에서 특히 ID 관리에 대하여 Azure AD를 사용하는 것을 살펴보겠습니다.

•	고객이 자신의 사용자 프로필을 Azure AD에(Office365 및 Dynamics CRM 테넌트 포함) 저장한다고 가정합니다.

•	사내 Active Directory (AD)의 고객들은 사내 AD와 Azure AD를 동기화하기 위해서 [Azure AD Connect][ADConnect]를 사용할 수 있습니다.
 
사내 AD 고객들이 Azure AD Connect를 사용할 수 없는 경우(회사 IT 정책이나 다른 이유로), SaaS 공급자는 Ative Directory Federation Services (AD FS)를 통해서 고객의 AD와 연동할 수 있습니다. 이 옵션은 [고객의 AD FS와 연동하기](https://docs.microsoft.com/en-us/azure/architecture/multitenant-identity/adfs)에서 설명합니다.

이 지침은 데이터 분할, 테넌트당 구성 등 다중 테넌트의 다른 측면을 고려하지 않았습니다.

[**다음**][tailpin]



<!-- Links -->
[ADConnect]: /azure/active-directory/active-directory-aadconnect/
[AzureAD]: https://azure.microsoft.com/documentation/services/active-directory/

[Federating with a customer's AD FS]: adfs.md
[tailpin]: tailspin.md

[running-the-app]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/docs/running-the-app.md
[sample application]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps
