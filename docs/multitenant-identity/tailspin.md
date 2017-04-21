---
title: About the Tailspin Surveys application
description: Tailspin Surveys application overview
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 05/23/2016

ms.author: pnp
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: index
pnp.series.next: authenticate
---
# Tailspin 시나리오

[![GitHub](../_images/github.png) 샘플 코드][sample application]

Tailspin은 Surveys라는 SaaS 응용 프로그램을 개발하는 가상 회사입니다. 이 응용 프로그램은 어떤 조직이 온라인 설문조사를 만들고 게시할 수 있게 해줍니다.

•	조직은 응용 프로그램에 등록할 수 있습니다.

•	조직이 등록되고 나면, 사용자는 자기 조직의 자격 증명으로 응용 프로그램에 로그인할 수 있습니다.

•	사용자는 설문조사를 생성, 편집, 검토할 수 있습니다.


> [!참고]
> 응용 프로그램을 시작하려면, [Surveys 응용 프로그램 실행하기]를 참조하세요.
> 
> 

## 사용자는 설문조사를 생성, 편집, 검토할 수 있습니다
인증된 사용자는 사람들이 만들거나 참가자 권한을 갖는 모든 설문조사를 검토할 수 있고, 새 설문조사를 만들 수 있습니다. 사용자는 자기 조직 ID, `bob@contoso.com`으로 로그인되었습니다.

![Surveys app](./images/surveys-screenshot.png)

다음 스크린샷은 설문조사 편집 페이지입니다:

![Edit survey](./images/edit-survey.png)

사용자는 같은 테넌트의 다른 사용자가 만든 설문조사도 볼 수 있습니다.

![Tenant surveys](./images/tenant-surveys.png)

## 설문조자 소유자는 참가자들을 초대할 수 있습니다.
사용자가 설문조사를 만들면, 설문조사에 참가할 다른 사람들을 초대할 수 있습니다. 참가자들은 설문조사를 편집할 수 있지만 삭제하거나 게시할 수 없습니다. 

![Add contributor](./images/add-contributor.png)

사용자는 다른 테넌트에 있는 참가자를 추가할 수 있고, 이를 통해 테넌트 간 리소스 공유가 가능합니다. 이 스크린샷에서, Bob은 (`bob@contoso.com`) Alice (`alice@fabrikam.com`)를 Bob이 만든 설문조사 참가자로 추가합니다.

Alice가 로그인하면, "내가 참여할 설문조사"에서 설문조사를 볼 수 있습니다.

![Survey contributor](./images/contributor.png)

Alice는 Contoso 테넌트의 손님이 아닌, 자신의 테넌트로 로그인했습니다. Alice는 이 설문조사에만 참가자 권한을 갖습니다 - Contoso 테넌트의 다른 설문조사를 볼 수 없습니다.

## 아키텍처
Surveys 응용 프로그램은 웹 프런트 엔드와 웹 API 백 엔드로 구성됩니다. 둘 다 [ASP.NET Core 1.0](https://docs.microsoft.com/en-us/aspnet/core/)을 사용해서 구현되었습니다.

웹 응용 프로그램은 사용자 인증에 Azure Active Directory (Azure AD)를 사용합니다. 웹 응용 프로그램은 Web API에 대한 OAuth 2 액세스 토큰을 가져올 때도 Azure AD를 호출합니다. 액세스 토큰은 Azure Redis Cache에 캐시됩니다. 캐시를 쓰면 다중 인스턴스들은 같은 토큰 캐시를 공유할 수 있습니다(예: 서버 팜에서).

![Architecture](./images/architecture.png)

[**다음**][authentication]

<!-- Links -->

[authentication]: authenticate.md

[Running the Surveys application]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/docs/running-the-app.md
[ASP.NET Core 1.0]: https://docs.asp.net/en/latest/
[sample application]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps
