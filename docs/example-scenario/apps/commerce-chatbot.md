---
title: 호텔 예약용 대화형 Azure 챗봇
description: Azure Bot Service, Cognitive Services 및 LUIS, Azure SQL Database 및 Application Insights를 사용하여 상거래 응용 프로그램용 대화형 챗봇을 구축하는 입증된 솔루션입니다.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: 85bdc3194961bbbd8d89db34e5c56e4baa8d8599
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891331"
---
# <a name="conversational-azure-chatbot-for-hotel-reservations"></a><span data-ttu-id="b8905-103">호텔 예약용 대화형 Azure 챗봇</span><span class="sxs-lookup"><span data-stu-id="b8905-103">Conversational Azure chatbot for hotel reservations</span></span>

<span data-ttu-id="b8905-104">이 예제 시나리오는 대화형 챗봇을 응용 프로그램에 통합해야 하는 비즈니스에 적용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-104">This example scenario is applicable to businesses that need integrate a conversational chatbot into applications.</span></span> <span data-ttu-id="b8905-105">이 솔루션에서는 고객이 웹 또는 모바일 응용 프로그램을 통해 가용성을 확인하고 숙박을 예약할 수 있는 호텔 체인에 C# 챗봇을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-105">In this solution, a C# chatbot is used for a hotel chain that allows customers to check availability and book accommodation through a web or mobile application.</span></span>

<span data-ttu-id="b8905-106">예제 시나리오에서는 고객이 호텔의 가용성을 확인하고, 객실을 예약하고, 식당 포장 메뉴를 검토하고, 음식 주문을 하거나 사진 인화를 검색하고 주문할 수 있는 방법이 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-106">Example scenarios include providing a way for customers to view hotel availability and book rooms, review a restaurant take-out menu and place a food order, or search for and order prints of photographs.</span></span> <span data-ttu-id="b8905-107">일반적으로 기업은 이러한 고객 요청에 대응하기 위해 고객 서비스 담당자를 고용하고 교육해야 하며, 고객은 담당자로부터 지원이 제공될 때까지 기다려야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-107">Traditionally, businesses would need to hire and train customer service agents to respond to these customer requests, and customers would have to wait until a representative is available to provide assistance.</span></span>

<span data-ttu-id="b8905-108">Bot Service 및 Language Understanding 또는 Speech API 서비스와 같은 Azure 서비스를 사용하면, 회사에서 확장 가능한 자동화 봇을 통해 고객을 지원하고 주문 또는 예약을 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-108">By using Azure services such as the Bot Service and Language Understanding or Speech API services, companies can assist customers and process orders or reservations with automated, scalable bots.</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="b8905-109">잠재적인 사용 사례</span><span class="sxs-lookup"><span data-stu-id="b8905-109">Potential use cases</span></span>

<span data-ttu-id="b8905-110">이 솔루션을 사용하는 데 적합한 사용 사례는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-110">Consider this solution for the following use cases:</span></span>

* <span data-ttu-id="b8905-111">식당 포장 메뉴 보기 및 음식 주문</span><span class="sxs-lookup"><span data-stu-id="b8905-111">View restaurant take-out menu and order food</span></span>
* <span data-ttu-id="b8905-112">호텔 가용성 확인 및 객실 예약</span><span class="sxs-lookup"><span data-stu-id="b8905-112">Check hotel availability and reserve a room</span></span>
* <span data-ttu-id="b8905-113">사용 가능한 사진 검색 및 인화 주문</span><span class="sxs-lookup"><span data-stu-id="b8905-113">Search available photos and order prints</span></span>

## <a name="architecture"></a><span data-ttu-id="b8905-114">아키텍처</span><span class="sxs-lookup"><span data-stu-id="b8905-114">Architecture</span></span>

![대화형 챗봇에 포함된 Azure 구성 요소의 아키텍처에 대한 개요][architecture]

<span data-ttu-id="b8905-116">이 솔루션에서는 호텔의 안내원 역할을 수행하는 대화형 봇을 다루고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-116">This solution covers a conversational bot that functions as a concierge for a hotel.</span></span> <span data-ttu-id="b8905-117">솔루션을 통한 데이터 흐름은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-117">The data flows through the solution as follows:</span></span>

1. <span data-ttu-id="b8905-118">고객이 모바일 앱 또는 웹앱을 통해 챗봇에 액세스합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-118">The customer accesses the chatbot with a mobile or web app.</span></span>
2. <span data-ttu-id="b8905-119">Azure Active Directory B2C(Business 2 Customer)를 사용하면 사용자가 인증됩니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-119">Using Azure Active Directory B2C (Business 2 Customer), the user is authenticated.</span></span>
3. <span data-ttu-id="b8905-120">사용자는 Bot Service와 상호 작용하여 호텔 가용성에 대한 정보를 요청합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-120">Interacting with the Bot Service, the user requests information about hotel availability.</span></span>
4. <span data-ttu-id="b8905-121">Cognitive Services에서 자연어 요청을 처리하여 고객 통신을 파악합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-121">Cognitive Services processes the natural language request to understand the customer communication.</span></span>
5. <span data-ttu-id="b8905-122">사용자가 결과에 만족하면 봇이 SQL Database에서 고객의 예약을 추가하거나 업데이트합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-122">After the user is happy with the results, the bot adds or updates the customer’s reservation in a SQL Database.</span></span>
6. <span data-ttu-id="b8905-123">Application Insights에서 프로세스 전반에 걸쳐 런타임 원격 분석을 수집하여 DevOps 팀에서 봇의 성능과 사용을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-123">Application Insights gathers runtime telemetry throughout the process to help the DevOps team with bot performance and usage.</span></span>

### <a name="components"></a><span data-ttu-id="b8905-124">구성 요소</span><span class="sxs-lookup"><span data-stu-id="b8905-124">Components</span></span>

* <span data-ttu-id="b8905-125">[Azure Active Directory][aad-docs]는 Microsoft의 다중 테넌트 클라우드 기반 디렉터리 및 ID 관리 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-125">[Azure Active Directory][aad-docs] is Microsoft’s multi-tenant cloud-based directory and identity management service.</span></span> <span data-ttu-id="b8905-126">Azure AD는 B2C 커넥터를 지원하여 Google, Facebook 또는 Microsoft 계정과 같은 외부 ID를 사용하여 개인을 식별할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-126">Azure AD supports a B2C connector allowing you to identify individuals using external IDs such as Google, Facebook, or a Microsoft Account.</span></span>
* <span data-ttu-id="b8905-127">[App Service][appservice-docs]를 사용하면 인프라를 관리할 필요 없이 선택한 프로그래밍 언어로 웹 응용 프로그램을 빌드하고 호스팅할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-127">[App Service][appservice-docs] enables you to build and host web applications in the programming language of your choice without managing infrastructure.</span></span>
* <span data-ttu-id="b8905-128">[Bot Service][botservice-docs]는 지능형 봇을 빌드, 테스트, 배포 및 관리할 수 있는 도구를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-128">[Bot Service][botservice-docs] provides tools to build, test, deploy, and manage intelligent bots.</span></span>
* <span data-ttu-id="b8905-129">[Cognitive Services][cognitive-docs]를 사용하면 지능형 알고리즘을 사용하여 자연스러운 의사 소통 방법을 통해 사용자의 요구 사항을 보고, 듣고, 말하고, 이해하고 해석할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-129">[Cognitive Services][cognitive-docs] lets you use intelligent algorithms to see, hear, speak, understand and interpret your user needs through natural methods of communication.</span></span>
* <span data-ttu-id="b8905-130">[SQL Database][sqldatabase-docs]는 SQL Server 엔진 호환성을 제공하는 완전하게 관리되는 관계형 클라우드 데이터베이스 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-130">[SQL Database][sqldatabase-docs] is a fully managed relational cloud database service that provides SQL Server engine compatibility.</span></span>
* <span data-ttu-id="b8905-131">[Application Insights][appinsights-docs]는 챗봇과 같은 응용 프로그램의 성능을 모니터링할 수 있는 확장 가능한 APM(Application Performance Management) 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-131">[Application Insights][appinsights-docs] is an extensible Application Performance Management (APM) service that lets you monitor the performance of applications, such as your chatbot.</span></span>

### <a name="alternatives"></a><span data-ttu-id="b8905-132">대안</span><span class="sxs-lookup"><span data-stu-id="b8905-132">Alternatives</span></span>

* <span data-ttu-id="b8905-133">[Microsoft Speech API][speech-api]를 사용하여 고객이 봇과 인터페이스하는 방법을 변경할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-133">[Microsoft Speech API][speech-api] can be used to change how customers interface with your bot.</span></span>
* <span data-ttu-id="b8905-134">[QnA Maker][qna-maker]를 사용하여 FAQ와 같이 반구조적 콘텐츠에서 봇으로 지식을 빠르게 추가할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-134">[QnA Maker][qna-maker] can be used as to quickly add knowledge to your bot from semi-structured content like an FAQ.</span></span>
* <span data-ttu-id="b8905-135">[Translator Text][translator]는 봇에 다국어 지원을 쉽게 추가하기 위해 고려할 수 있는 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-135">[Translator Text][translator] is a service that you might consider to easily add multi-lingual support to your bot.</span></span>

## <a name="considerations"></a><span data-ttu-id="b8905-136">고려 사항</span><span class="sxs-lookup"><span data-stu-id="b8905-136">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="b8905-137">가용성</span><span class="sxs-lookup"><span data-stu-id="b8905-137">Availability</span></span>

<span data-ttu-id="b8905-138">이 솔루션에서는 Azure SQL Database를 사용하여 고객 예약을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-138">This solution uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="b8905-139">SQL Database에는 영역 중복 데이터베이스, 장애 조치 그룹 및 지역 복제가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-139">SQL Database includes zone redundant databases, failover groups, and geo-replication.</span></span> <span data-ttu-id="b8905-140">자세한 내용은 [Azure SQL Database 가용성 기능][sqlavailability-docs]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-140">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="b8905-141">다른 확장성 항목에 대해서는 Azure 아키텍처 센터의 [가용성 검사 목록][availability]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-141">For other scalability topics, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="b8905-142">확장성</span><span class="sxs-lookup"><span data-stu-id="b8905-142">Scalability</span></span>

<span data-ttu-id="b8905-143">이 솔루션에서는 Azure App Service를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-143">This solution uses Azure App Service.</span></span> <span data-ttu-id="b8905-144">App Service를 사용하면 봇을 실행하는 인스턴스의 수를 자동으로 조정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-144">With App Service, you can automatically scale the number of instances that run your bot.</span></span> <span data-ttu-id="b8905-145">이 기능을 사용하면 웹 응용 프로그램과 챗봇에 대한 고객의 요구 사항을 충족할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-145">This functionality lets you keep up with customer demand for your web application and chatbot.</span></span> <span data-ttu-id="b8905-146">자동 크기 조정에 대한 자세한 내용은 아키텍처 센터의 [자동 크기 조정 모범 사례][autoscaling]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-146">For more information on autoscale, see [Autoscaling best practices][autoscaling] in the architecture center.</span></span>

<span data-ttu-id="b8905-147">다른 확장성 항목에 대해서는 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-147">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="b8905-148">보안</span><span class="sxs-lookup"><span data-stu-id="b8905-148">Security</span></span>

<span data-ttu-id="b8905-149">이 솔루션에서는 Azure Active Directory B2C(Business 2 Consumer)를 사용하여 사용자를 인증합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-149">This solution uses Azure Active Directory B2C (Business 2 Consumer) to authenticate users.</span></span> <span data-ttu-id="b8905-150">AAD B2C를 사용하면 중요한 고객 계정 정보 또는 자격 증명이 챗봇에 저장되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-150">With AAD B2C, your chatbot doesn't store any sensitive customer account information or credentials.</span></span> <span data-ttu-id="b8905-151">자세한 내용은 [Azure Active Directory B2C 개요][aadb2c-docs]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-151">For more information, see [Azure Active Directory B2C overview][aadb2c-docs].</span></span>

<span data-ttu-id="b8905-152">Azure SQL Database에 저장된 미사용 정보는 TDE(투명한 데이터 암호화)를 사용하여 암호화됩니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-152">Information stored in Azure SQL Database is encrypted at rest with transparent data encryption (TDE).</span></span> <span data-ttu-id="b8905-153">또한 SQL Database는 쿼리 및 처리 중에도 데이터를 암호화하는 Always Encrypted를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-153">SQL Database also offers Always Encrypted which encrypts data during querying and processing.</span></span> <span data-ttu-id="b8905-154">SQL Database 보안에 대한 자세한 내용은 [Azure SQL Database 보안 및 준수][sqlsecurity-docs]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-154">For more information on SQL Database security, see [Azure SQL Database security and compliance][sqlsecurity-docs].</span></span>

<span data-ttu-id="b8905-155">보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-155">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="b8905-156">복원력</span><span class="sxs-lookup"><span data-stu-id="b8905-156">Resiliency</span></span>

<span data-ttu-id="b8905-157">이 솔루션에서는 Azure SQL Database를 사용하여 고객 예약을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-157">This solution uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="b8905-158">SQL Database에는 영역 중복 데이터베이스, 장애 조치 그룹, 지역 복제 및 자동 백업이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-158">SQL Database includes zone redundant databases, failover groups, geo-replication, and automatic backups.</span></span> <span data-ttu-id="b8905-159">이러한 기능을 통해 유지 관리 이벤트 또는 중단이 발생하는 경우에도 응용 프로그램을 계속 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-159">These features allow your application to continue running in the event of a maintenance event or outage.</span></span> <span data-ttu-id="b8905-160">자세한 내용은 [Azure SQL Database 가용성 기능][sqlavailability-docs]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-160">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="b8905-161">응용 프로그램 상태를 모니터링하기 위해 이 솔루션에서는 Application Insights를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-161">To monitor the health of your application, this solution uses Application Insights.</span></span> <span data-ttu-id="b8905-162">Application Insights를 사용하면 고객의 경험과 챗봇의 가용성에 영향을 주는 알림을 생성하고 성능 문제에 대응할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-162">With Application Insights, you can generate alerts and respond to performance issues that would impact the customer experience and availability of the chatbot.</span></span> <span data-ttu-id="b8905-163">자세한 내용은 [Application Insights란?][appinsights-docs]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-163">For more information, see [What is Application Insights?][appinsights-docs]</span></span>

<span data-ttu-id="b8905-164">복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 응용 프로그램 디자인][resiliency]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-164">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="b8905-165">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="b8905-165">Deploy the solution</span></span>

<span data-ttu-id="b8905-166">이 솔루션은 가장 중점을 두는 영역을 탐색할 수 있는 다음 세 가지 구성 요소로 구분됩니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-166">This solution is divided into three components for you to explore areas that you are most focused on:</span></span>

* <span data-ttu-id="b8905-167">[인프라 구성 요소](#deploy-infrastructure-components).</span><span class="sxs-lookup"><span data-stu-id="b8905-167">[Infrastructure components](#deploy-infrastructure-components).</span></span> <span data-ttu-id="b8905-168">Azure Resource Manager 템플릿을 사용하여 App Service, Web App, Application Insights, Storage 계정, SQL Server 및 데이터베이스의 핵심 인프라 구성 요소를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-168">Use an Azure Resource Manger template to deploy the core infrastructure components of an App Service, Web App, Application Insights, Storage account, and SQL Server and database.</span></span>
* <span data-ttu-id="b8905-169">[Web App 챗봇](#deploy-web-app-chatbot).</span><span class="sxs-lookup"><span data-stu-id="b8905-169">[Web App Chatbot](#deploy-web-app-chatbot).</span></span> <span data-ttu-id="b8905-170">Azure CLI를 사용하여 Bot Service 및 LUIS(Language Understanding and Intelligent Service) 앱을 통해 봇을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-170">Use the Azure CLI to deploy a bot with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>
* <span data-ttu-id="b8905-171">[샘플 C# 챗봇 응용 프로그램](#deploy-chatbot-c-application-code).</span><span class="sxs-lookup"><span data-stu-id="b8905-171">[Sample C# chatbot application](#deploy-chatbot-c-application-code).</span></span> <span data-ttu-id="b8905-172">Visual Studio를 사용하여 호텔 예약 C# 응용 프로그램 코드 샘플을 검토하고 Azure에서 봇에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-172">Use Visual Studio to review the sample hotel reservation C# application code and deploy to a bot in Azure.</span></span>

<span data-ttu-id="b8905-173">**필수 조건.**</span><span class="sxs-lookup"><span data-stu-id="b8905-173">**Prerequisites.**</span></span> <span data-ttu-id="b8905-174">기존 Azure 계정이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-174">You must have an existing Azure account.</span></span> <span data-ttu-id="b8905-175">Azure 구독이 아직 없는 경우 시작하기 전에 [무료 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-175">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

### <a name="deploy-infrastructure-components"></a><span data-ttu-id="b8905-176">인프라 구성 요소 배포</span><span class="sxs-lookup"><span data-stu-id="b8905-176">Deploy infrastructure components</span></span>

<span data-ttu-id="b8905-177">Azure Resource Manager 템플릿을 사용하여 인프라 구성 요소를 배포하려면 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-177">To deploy the infrastructure components with an Azure Resource Manager template, perform the following steps.</span></span>

1. <span data-ttu-id="b8905-178">**Azure에 배포** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-178">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="b8905-179">Azure Portal에서 템플릿 배포가 열릴 때까지 기다린 후에 다음 단계를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-179">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   * <span data-ttu-id="b8905-180">리소스 그룹 **새로 만들기**를 선택한 다음, 이름(예: *myCommerceChatBotInfrastructure*)을 입력합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-180">Choose to **Create new** resource group, then provide a name such as *myCommerceChatBotInfrastructure* in the text box.</span></span>
   * <span data-ttu-id="b8905-181">**위치** 드롭다운 상자에서 지역을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-181">Select a region from the **Location** drop-down box.</span></span>
   * <span data-ttu-id="b8905-182">SQL Server 관리자 계정에 대한 사용자 이름과 보안 암호를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-182">Provide a username and secure password for the SQL Server administrator account.</span></span>
   * <span data-ttu-id="b8905-183">사용 약관을 검토한 다음, **위에 명시된 사용 약관에 동의함**을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-183">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   * <span data-ttu-id="b8905-184">**구매** 단추를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-184">Select the **Purchase** button.</span></span>

<span data-ttu-id="b8905-185">배포가 완료되는 데 몇 분 정도 걸립니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-185">It takes a few minutes for the deployment to complete.</span></span>

### <a name="deploy-web-app-chatbot"></a><span data-ttu-id="b8905-186">Web App 챗봇 배포</span><span class="sxs-lookup"><span data-stu-id="b8905-186">Deploy Web App chatbot</span></span>

<span data-ttu-id="b8905-187">챗봇을 만들려면 Azure CLI를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-187">To create the chatbot, use the Azure CLI.</span></span> <span data-ttu-id="b8905-188">다음 예제에서는 Bot Service용 CLI 확장을 설치하고, 리소스 그룹을 만든 다음, Application Insights를 사용하는 봇을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-188">The following example installs the CLI extension for Bot Service, creates a resource group, then deploys a bot that uses Application Insights.</span></span> <span data-ttu-id="b8905-189">메시지가 표시되면 Microsoft 계정을 인증하고, 봇에서 Bot Service 및 LUIS(Language Understanding and Intelligent Service) 앱에 자체를 등록하도록 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-189">When prompted, authenticate your Microsoft account and allow the bot to register itself with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>

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

### <a name="deploy-chatbot-c-application-code"></a><span data-ttu-id="b8905-190">챗봇 C# 응용 프로그램 코드 배포</span><span class="sxs-lookup"><span data-stu-id="b8905-190">Deploy chatbot C# application code</span></span>

<span data-ttu-id="b8905-191">샘플 C# 응용 프로그램은 GitHub에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-191">A sample C# application is available on GitHub:</span></span> 

* [<span data-ttu-id="b8905-192">상거래 봇 C# 샘플</span><span class="sxs-lookup"><span data-stu-id="b8905-192">Commerce Bot C# sample</span></span>](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

<span data-ttu-id="b8905-193">샘플 응용 프로그램에는 Azure Active Directory 인증 구성 요소 및 통합된 Cognitive Services의 LUIS(Language Understanding and Intelligent Services) 구성 요소가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-193">The sample application includes the Azure Active Directory authentication components and integration with the Language Understanding and Intelligent Services (LUIS) component of Cognitive Services.</span></span> <span data-ttu-id="b8905-194">응용 프로그램을 사용하려면 Visual Studio에서 솔루션을 빌드하고 배포해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-194">The application requires Visual Studio to build and deploy the solution.</span></span> <span data-ttu-id="b8905-195">AAD B2C 및 LUIS 앱 구성에 대한 추가 정보는 GitHub 리포지토리 설명서에 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-195">Additional information on configuring AAD B2C and the LUIS app can be found in the GitHub repo documentation.</span></span>

## <a name="pricing"></a><span data-ttu-id="b8905-196">가격</span><span class="sxs-lookup"><span data-stu-id="b8905-196">Pricing</span></span>

<span data-ttu-id="b8905-197">이 솔루션을 실행하는 비용을 알아보기 위해 모든 서비스가 비용 계산기에서 미리 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-197">To explore the cost of running this solution, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="b8905-198">특정 사용 사례에 대한 가격이 변경되는 정도를 확인하려면 필요한 트래픽에 맞게 적절한 변수를 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-198">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="b8905-199">챗봇에서 처리하는 데 필요한 메시지 양을 기준으로 다음 세 가지 샘플 비용 프로필을 제공했습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-199">We have provided three sample cost profiles based on the amount of messages you expect your chatbot to process:</span></span>

* <span data-ttu-id="b8905-200">[소량][small-pricing]: 매월 1만 개 미만의 메시지 처리와 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-200">[Small][small-pricing]: this correlates to processing < 10,000 messages per month.</span></span>
* <span data-ttu-id="b8905-201">[중간][medium-pricing]: 매월 50만 개 미만의 메시지 처리와 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-201">[Medium][medium-pricing]: this correlates to processing < 500,000 messages per month.</span></span>
* <span data-ttu-id="b8905-202">[대량][large-pricing]: 매월 1천만 개 미만의 메시지 처리와 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b8905-202">[Large][large-pricing]: this correlates to processing < 10 million messages per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="b8905-203">관련 리소스</span><span class="sxs-lookup"><span data-stu-id="b8905-203">Related Resources</span></span>

<span data-ttu-id="b8905-204">Azure Bot Service를 활용하는 방법에 대한 일단의 단계별 자습서는 설명서의 [자습서 노드][botservice-docs]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b8905-204">For a set of guided tutorials on leveraging the Azure Bot Service, see the [tutorial node][botservice-docs] of the documentation.</span></span>

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