---
title: Azure에서 호텔 예약을 위한 대화형 챗봇
description: Azure Bot Service, Cognitive Services 및 LUIS, Azure SQL Database 및 Application Insights를 사용하여 상거래 응용 프로그램용 대화형 챗봇을 구축하는 데 입증된 시나리오입니다.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: b664faf20d806824c2581346aaa592b0d74207da
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060866"
---
# <a name="conversational-chatbot-for-hotel-reservations-on-azure"></a>Azure에서 호텔 예약을 위한 대화형 챗봇

이 예제 시나리오는 대화형 챗봇을 응용 프로그램에 통합해야 하는 비즈니스에 적용할 수 있습니다. 이 시나리오에서는 고객이 웹 또는 모바일 응용 프로그램을 통해 가용성을 확인하고 숙박을 예약할 수 있는 호텔 체인에 C# 챗봇을 사용합니다.

예제 시나리오에서는 고객이 호텔의 가용성을 확인하고, 객실을 예약하고, 식당 포장 메뉴를 검토하고, 음식 주문을 하거나 사진 인화를 검색하고 주문할 수 있는 방법이 제공됩니다. 일반적으로 기업은 이러한 고객 요청에 대응하기 위해 고객 서비스 담당자를 고용하고 교육해야 하며, 고객은 담당자로부터 지원이 제공될 때까지 기다려야 합니다.

Bot Service 및 Language Understanding 또는 Speech API 서비스와 같은 Azure 서비스를 사용하면, 회사에서 확장 가능한 자동화 봇을 통해 고객을 지원하고 주문 또는 예약을 처리할 수 있습니다.

## <a name="related-use-cases"></a>관련 사용 사례

이 시나리오에 적합한 사용 사례는 다음과 같습니다.

* 식당 포장 메뉴 보기 및 음식 주문
* 호텔 가용성 확인 및 객실 예약
* 사용 가능한 사진 검색 및 인화 주문

## <a name="architecture"></a>아키텍처

![대화형 챗봇에 포함된 Azure 구성 요소의 아키텍처에 대한 개요][architecture]

이 시나리오에서는 호텔의 안내원 역할을 수행하는 대화형 봇에 대해 설명합니다. 시나리오를 통한 데이터 흐름은 다음과 같습니다.

1. 고객이 모바일 앱 또는 웹앱을 통해 챗봇에 액세스합니다.
2. Azure Active Directory B2C(Business 2 Customer)를 사용하면 사용자가 인증됩니다.
3. 사용자는 Bot Service와 상호 작용하여 호텔 가용성에 대한 정보를 요청합니다.
4. Cognitive Services에서 자연어 요청을 처리하여 고객 통신을 파악합니다.
5. 사용자가 결과에 만족하면 봇이 SQL Database에서 고객의 예약을 추가하거나 업데이트합니다.
6. Application Insights에서 프로세스 전반에 걸쳐 런타임 원격 분석을 수집하여 DevOps 팀에서 봇의 성능과 사용을 지원합니다.

### <a name="components"></a>구성 요소

* [Azure Active Directory][aad-docs]는 Microsoft의 다중 테넌트 클라우드 기반 디렉터리 및 ID 관리 서비스입니다. Azure AD는 B2C 커넥터를 지원하여 Google, Facebook 또는 Microsoft 계정과 같은 외부 ID를 사용하여 개인을 식별할 수 있습니다.
* [App Service][appservice-docs]를 사용하면 인프라를 관리할 필요 없이 선택한 프로그래밍 언어로 웹 응용 프로그램을 빌드하고 호스팅할 수 있습니다.
* [Bot Service][botservice-docs]는 지능형 봇을 빌드, 테스트, 배포 및 관리할 수 있는 도구를 제공합니다.
* [Cognitive Services][cognitive-docs]를 사용하면 지능형 알고리즘을 사용하여 자연스러운 의사 소통 방법을 통해 사용자의 요구 사항을 보고, 듣고, 말하고, 이해하고 해석할 수 있습니다.
* [SQL Database][sqldatabase-docs]는 SQL Server 엔진 호환성을 제공하는 완전하게 관리되는 관계형 클라우드 데이터베이스 서비스입니다.
* [Application Insights][appinsights-docs]는 챗봇과 같은 응용 프로그램의 성능을 모니터링할 수 있는 확장 가능한 APM(Application Performance Management) 서비스입니다.

### <a name="alternatives"></a>대안

* [Microsoft Speech API][speech-api]를 사용하여 고객이 봇과 인터페이스하는 방법을 변경할 수 있습니다.
* [QnA Maker][qna-maker]를 사용하여 FAQ와 같이 반구조적 콘텐츠에서 봇으로 지식을 빠르게 추가할 수 있습니다.
* [Translator Text][translator]는 봇에 다국어 지원을 쉽게 추가하기 위해 고려할 수 있는 서비스입니다.

## <a name="considerations"></a>고려 사항

### <a name="availability"></a>가용성

이 시나리오에서는 Azure SQL Database를 사용하여 고객 예약을 저장합니다. SQL Database에는 영역 중복 데이터베이스, 장애 조치 그룹 및 지역 복제가 포함됩니다. 자세한 내용은 [Azure SQL Database 가용성 기능][sqlavailability-docs]을 참조하세요.

다른 가용성 항목에 대해서는 Azure 아키텍처 센터의 [가용성 검사 목록][availability]을 참조하세요.

### <a name="scalability"></a>확장성

이 시나리오에서는 Azure App Service를 사용합니다. App Service를 사용하면 봇을 실행하는 인스턴스의 수를 자동으로 조정할 수 있습니다. 이 기능을 사용하면 웹 응용 프로그램과 챗봇에 대한 고객의 요구 사항을 충족할 수 있습니다. 자동 크기 조정에 대한 자세한 내용은 아키텍처 센터의 [자동 크기 조정 모범 사례][autoscaling]를 참조하세요.

다른 확장성 항목에 대해서는 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.

### <a name="security"></a>보안

이 시나리오에서는 Azure Active Directory B2C(Business 2 Consumer)를 사용하여 사용자를 인증합니다. AAD B2C를 사용하면 중요한 고객 계정 정보 또는 자격 증명이 챗봇에 저장되지 않습니다. 자세한 내용은 [Azure Active Directory B2C 개요][aadb2c-docs]를 참조하세요.

Azure SQL Database에 저장된 미사용 정보는 TDE(투명한 데이터 암호화)를 사용하여 암호화됩니다. 또한 SQL Database는 쿼리 및 처리 중에도 데이터를 암호화하는 Always Encrypted를 제공합니다. SQL Database 보안에 대한 자세한 내용은 [Azure SQL Database 보안 및 준수][sqlsecurity-docs]를 참조하세요.

보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.

### <a name="resiliency"></a>복원력

이 시나리오에서는 Azure SQL Database를 사용하여 고객 예약을 저장합니다. SQL Database에는 영역 중복 데이터베이스, 장애 조치 그룹, 지역 복제 및 자동 백업이 포함됩니다. 이러한 기능을 통해 유지 관리 이벤트 또는 중단이 발생하는 경우에도 응용 프로그램을 계속 실행할 수 있습니다. 자세한 내용은 [Azure SQL Database 가용성 기능][sqlavailability-docs]을 참조하세요.

이 시나리오에서는 응용 프로그램 상태를 모니터링하기 위해 Application Insights를 사용합니다. Application Insights를 사용하면 고객의 경험과 챗봇의 가용성에 영향을 주는 알림을 생성하고 성능 문제에 대응할 수 있습니다. 자세한 내용은 [Application Insights란?][appinsights-docs]을 참조하세요.

복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 응용 프로그램 디자인][resiliency]을 참조하세요.

## <a name="deploy-the-scenario"></a>시나리오 배포

이 시나리오는 가장 중점을 두는 영역을 탐색할 수 있는 다음 세 가지 구성 요소로 구분됩니다.

* [인프라 구성 요소](#deploy-infrastructure-components). Azure Resource Manager 템플릿을 사용하여 App Service, Web App, Application Insights, Storage 계정, SQL Server 및 데이터베이스의 핵심 인프라 구성 요소를 배포합니다.
* [Web App 챗봇](#deploy-web-app-chatbot). Azure CLI를 사용하여 Bot Service 및 LUIS(Language Understanding and Intelligent Service) 앱을 통해 봇을 배포합니다.
* [샘플 C# 챗봇 응용 프로그램](#deploy-chatbot-c-application-code). Visual Studio를 사용하여 호텔 예약 C# 응용 프로그램 코드 샘플을 검토하고 Azure에서 봇에 배포합니다.

**필수 조건.** 기존 Azure 계정이 있어야 합니다. Azure 구독이 아직 없는 경우 시작하기 전에 [무료 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.

### <a name="deploy-infrastructure-components"></a>인프라 구성 요소 배포

Azure Resource Manager 템플릿을 사용하여 인프라 구성 요소를 배포하려면 다음 단계를 수행합니다.

1. **Azure에 배포** 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Azure Portal에서 템플릿 배포가 열릴 때까지 기다린 후에 다음 단계를 수행합니다.
   * 리소스 그룹 **새로 만들기**를 선택한 다음, 이름(예: *myCommerceChatBotInfrastructure*)을 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * SQL Server 관리자 계정에 대한 사용자 이름과 보안 암호를 제공합니다.
   * 사용 약관을 검토한 다음, **위에 명시된 사용 약관에 동의함**을 선택합니다.
   * **구매** 단추를 선택합니다.

배포가 완료되는 데 몇 분 정도 걸립니다.

### <a name="deploy-web-app-chatbot"></a>Web App 챗봇 배포

챗봇을 만들려면 Azure CLI를 사용합니다. 다음 예제에서는 Bot Service용 CLI 확장을 설치하고, 리소스 그룹을 만든 다음, Application Insights를 사용하는 봇을 배포합니다. 메시지가 표시되면 Microsoft 계정을 인증하고, 봇에서 Bot Service 및 LUIS(Language Understanding and Intelligent Service) 앱에 자체를 등록하도록 허용합니다.

```azurecli-interactive
# Install the Azure CLI extension for the Bot Service
az extension add --name botservice --yes

# Create a resource group
az group create --name myCommerceChatbot --location eastus

# Create a Web App Chatbot that uses Application Insights
az bot create \
    --resource-group myCommerceChatbot \
    --name commerceChatbot \
    --location eastus \
    --kind webapp \
    --sku S1 \
    --insights eastus
```

### <a name="deploy-chatbot-c-application-code"></a>챗봇 C# 응용 프로그램 코드 배포

샘플 C# 응용 프로그램은 GitHub에서 사용할 수 있습니다. 

* [상거래 봇 C# 샘플](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

샘플 응용 프로그램에는 Azure Active Directory 인증 구성 요소 및 통합된 Cognitive Services의 LUIS(Language Understanding and Intelligent Services) 구성 요소가 포함되어 있습니다. 응용 프로그램을 사용하려면 Visual Studio에서 시나리오를 빌드하고 배포해야 합니다. AAD B2C 및 LUIS 앱 구성에 대한 추가 정보는 GitHub 리포지토리 설명서에 있습니다.

## <a name="pricing"></a>가격

이 시나리오를 실행하는 데 들어가는 비용을 알아보기 위해 모든 서비스가 비용 계산기에서 미리 구성됩니다. 특정 사용 사례에 대한 가격이 변경되는 정도를 확인하려면 필요한 트래픽에 맞게 적절한 변수를 변경합니다.

챗봇에서 처리하는 데 필요한 메시지 양을 기준으로 다음 세 가지 샘플 비용 프로필을 제공했습니다.

* [소량][small-pricing]: 매월 1만 개 미만의 메시지 처리와 관련이 있습니다.
* [중간][medium-pricing]: 매월 50만 개 미만의 메시지 처리와 관련이 있습니다.
* [대량][large-pricing]: 매월 1천만 개 미만의 메시지 처리와 관련이 있습니다.

## <a name="related-resources"></a>관련 리소스

Azure Bot Service를 활용하는 방법에 대한 일단의 단계별 자습서는 설명서의 [자습서 노드][botservice-docs]를 참조하세요.

<!-- links -->
[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/commerce-chatbot/architecture-commerce-chatbot.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[botservice-docs]: /azure/bot-service/
[cognitive-docs]: /azure/cognitive-services/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sqlavailability-docs]: /azure/sql-database/sql-database-technical-overview#availability-capabilities
[sqldatabase-docs]: /azure/sql-database/
[sqlsecurity-docs]: /azure/sql-database/sql-database-technical-overview#advanced-security-and-compliance
[qna-maker]: /azure/cognitive-services/QnAMaker/Overview/overview
[speech-api]: /azure/cognitive-services/speech/home
[translator]: /azure/cognitive-services/translator/translator-info-overview

[small-pricing]: https://azure.com/e/dce05b6184904c50b38e1a8654f726b6
[medium-pricing]: https://azure.com/e/304d17106afc480dbc414f9726078a03
[large-pricing]: https://azure.com/e/8319dd5e5e3d4f118f9029e32a80e887