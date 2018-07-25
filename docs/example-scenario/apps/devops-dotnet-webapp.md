---
title: VSTS를 사용하는 CI/CD 파이프라인
description: .NET 앱을 Azure Web Apps에 빌드하고 릴리스하는 예제입니다.
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: ae4ac5fc02cc841fc39b3cbef46124fe9da75e9b
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39061014"
---
# <a name="cicd-pipeline-with-vsts"></a><span data-ttu-id="09be0-103">VSTS를 사용하는 CI/CD 파이프라인</span><span class="sxs-lookup"><span data-stu-id="09be0-103">CI/CD pipeline with VSTS</span></span>

<span data-ttu-id="09be0-104">DevOps는 개발, 품질 보증 및 IT 운영의 통합입니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-104">DevOps is the integration of development, quality assurance and IT operations.</span></span> <span data-ttu-id="09be0-105">DevOps에는 소프트웨어를 제공하기 위한 통합 문화권 및 강력한 프로세스 집합이 모두 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-105">DevOps requires both unified culture and a strong set of processes for delivering software.</span></span>

<span data-ttu-id="09be0-106">이 예제 시나리오에서는 개발 팀이 Visual Studio Team Services를 사용하여 .NET 2계층 웹 응용 프로그램을 Azure App Service에 배포하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-106">This example scenario demonstrates how development teams can use Visual Studio Team Services to deploy a .NET two-tier web application to Azure App Service.</span></span> <span data-ttu-id="09be0-107">웹 응용 프로그램은 Azure PaaS(Platform as a Service) 서비스에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-107">The Web Application dependends on downstream Azure Platform as a Service (PaaS) services.</span></span> <span data-ttu-id="09be0-108">또한 이 문서에서는 Azure PaaS를 사용하여 이러한 시나리오를 설계할 때 고려해야 할 몇 가지 사항에 대해서도 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-108">This document also points out some considerations that you should make when designing such a scenario using Azure Platform as a Service (PaaS).</span></span>

<span data-ttu-id="09be0-109">CI(지속적인 통합) 및 CD(지속적인 배포)를 사용하는 최신 응용 프로그램 개발 방식을 채택하면 강력한 빌드, 테스트, 배포 및 모니터링 서비스를 통해 사용자에게 가치 전달을 가속화할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-109">Adopting a modern approach to application development using Continuous Integration (CI) and Continuous Deployment (CD), helps you to accelerate the delivery of value to your users through a robust build, test, deployment and monitoring service.</span></span> <span data-ttu-id="09be0-110">App Service와 같은 Azure 서비스 외에도 Visual Studio Team Services와 같은 플랫폼을 사용함으로써 조직에서 인프라 관리를 사용하도록 설정하는 대신 자신의 시나리오 개발에 집중할 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-110">By using a platform such as Visual Studio Team Services in addition to Azure services such as App Service, organizations can ensure they remain focused on the development of their scenario, rather than the management of the infrastructure to enable it.</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="09be0-111">관련 사용 사례</span><span class="sxs-lookup"><span data-stu-id="09be0-111">Related use cases</span></span>

<span data-ttu-id="09be0-112">DevOps에 적합한 사용 사례는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-112">Consider DevOps for the following use cases:</span></span>

* <span data-ttu-id="09be0-113">응용 프로그램 개발 및 개발 수명 주기 가속화</span><span class="sxs-lookup"><span data-stu-id="09be0-113">Speeding up application development and development life cycles</span></span>
* <span data-ttu-id="09be0-114">자동화된 빌드 및 릴리스 프로세스에 대한 품질 및 일관성 구축</span><span class="sxs-lookup"><span data-stu-id="09be0-114">Building quality and consistency into an automated build and release process</span></span>

## <a name="architecture"></a><span data-ttu-id="09be0-115">아키텍처</span><span class="sxs-lookup"><span data-stu-id="09be0-115">Architecture</span></span>

![Visual Studio Team Services 및 Azure App Service를 사용하는 DevOps 시나리오와 관련된 Azure 구성 요소 아키텍처에 대한 개요][architecture]

<span data-ttu-id="09be0-117">이 시나리오에서는 VSTS(Visual Studio Team Services)를 사용하는 .NET 웹 응용 프로그램의 DevOps 파이프라인에 대해 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-117">This scenario covers a DevOps pipeline for a .NET web application using Visual Studio Team Services (VSTS).</span></span> <span data-ttu-id="09be0-118">시나리오를 통한 데이터 흐름은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-118">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="09be0-119">응용 프로그램 소스 코드를 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-119">Change application source code.</span></span>
2. <span data-ttu-id="09be0-120">응용 프로그램 코드 및 Web Apps web.config 파일을 커밋합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-120">Commit application code and Web Apps web.config file.</span></span>
3. <span data-ttu-id="09be0-121">지속적인 통합에서 응용 프로그램 빌드 및 단위 테스트를 트리거합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-121">Continuous integration triggers application build and unit tests.</span></span>
4. <span data-ttu-id="09be0-122">지속적인 배포 트리거에서 *매개 변수화된 환경 특정 구성 값*을 사용하여 응용 프로그램 아티팩트의 배포를 오케스트레이션합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-122">Continuous deployment trigger orchestrates deployment of application artifacts *with environment-specific parameterized configuration values*.</span></span>
5. <span data-ttu-id="09be0-123">Azure App Service에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-123">Deployment to Azure App Service.</span></span>
6. <span data-ttu-id="09be0-124">Azure Application Insights에서 상태, 성능 및 사용량 데이터를 수집합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-124">Azure Application Insights collects and analyzes health, performance, and usage data.</span></span>
7. <span data-ttu-id="09be0-125">상태, 성능 및 사용량 정보를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-125">Review health, performance, and usage information.</span></span>

### <a name="components"></a><span data-ttu-id="09be0-126">구성 요소</span><span class="sxs-lookup"><span data-stu-id="09be0-126">Components</span></span>

* <span data-ttu-id="09be0-127">[리소스 그룹][resource-groups]은 Azure 리소스에 대한 논리 컨테이너이며, 관리 평면에 대한 액세스 제어 경계도 제공합니다. 리소스 그룹을 "배포 단위"로 간주합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-127">[Resource Groups][resource-groups] are a logical container for Azure resources and also provide an access control boundary for the management plane - think of a Resource Group as representing a "unit of deployment".</span></span>
* <span data-ttu-id="09be0-128">[VSTS(Visual Studio Team Services)][vsts]는 계획 및 프로젝트 관리에서 코드 관리, 빌드 및 릴리스에 이르기까지 개발 수명 주기를 종단 간으로 관리할 수 있게 하는 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-128">[Visual Studio Team Services (VSTS)][vsts] is a service that enables you to manage your development life cycle end-to-end; from planning and project management, to code management, through to build and release.</span></span>
* <span data-ttu-id="09be0-129">[Azure Web Apps][web-apps]는 웹 응용 프로그램, REST API 및 모바일 백 엔드를 호스팅하는 PaaS(Platform as a Service) 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-129">[Azure Web Apps][web-apps] is a Platform as a Service (PaaS) service for hosting web applications, REST APIs, and mobile back ends.</span></span> <span data-ttu-id="09be0-130">이 문서에서는 .NET에 집중하고 있지만 몇 가지 추가 개발 플랫폼 옵션이 지원됩니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-130">While this article focuses on .NET, there are several additional development platform options supported.</span></span>
* <span data-ttu-id="09be0-131">[Application Insights][application-insights]는 웹 개발자를 위해 여러 플랫폼에서 확장 가능한 자체 APM(Application Performance Management) 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-131">[Application Insights][application-insights] is a first-party, extensible Application Performance Management (APM) service for web developers on multiple platforms.</span></span>

### <a name="alternative-devops-tooling-options"></a><span data-ttu-id="09be0-132">대체 DevOps 도구 옵션</span><span class="sxs-lookup"><span data-stu-id="09be0-132">Alternative DevOps tooling options</span></span>

<span data-ttu-id="09be0-133">이 문서에서는 Visual Studio Team Services에 집중하고 있지만, [Team Foundation Server][team-foundation-server]를 온-프레미스 대체 도구로 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-133">Whilst this article focuses on Visual Studio Team Services, [Team Foundation Server][team-foundation-server] could be used as on premises substitute.</span></span> <span data-ttu-id="09be0-134">또는 [Jenkins][jenkins-on-azure]를 활용하는 오픈 소스 개발 파이프라인에 함께 사용되는 기술 모음을 찾을 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-134">Alternatively, you may also find a collection of technologies being used together for an Open Source development pipeline leveraging [Jenkins][jenkins-on-azure].</span></span>

<span data-ttu-id="09be0-135">인프라를 코드로 변환(Infrastructure as Code) 관점에서 [ARM(Azure Resource Manager) 템플릿][arm-templates]은 Azure DevOps 프로젝트의 일부로 포함되지만, 여기서는 [Terraform][terraform] 또는 [Chef][chef]를 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-135">From an Infrastructure as Code perspective, [Azure Resource Manager (ARM) Templates][arm-templates] are included as part of the Azure DevOps project, but you could consider [Terraform][terraform] or [Chef][chef] if you have investments here.</span></span> <span data-ttu-id="09be0-136">IaaS(Infrastructure as a Service) 기반 배포를 선호하고 구성 관리가 필요한 경우 [Azure Desired State Configuration][desired-state-configuration], [Ansible][ansible] 또는 [Chef][chef] 중 하나를 고려할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-136">If you prefer an Infrastructure as a Service (IaaS) based deployment and require configuration management, then you could consider either [Azure Desired State Configuration][desired-state-configuration], [Ansible][ansible] or [Chef][chef].</span></span>

### <a name="alternatives-to-web-app-hosting"></a><span data-ttu-id="09be0-137">Web App 호스팅에 대한 대안</span><span class="sxs-lookup"><span data-stu-id="09be0-137">Alternatives to Web App Hosting</span></span>

<span data-ttu-id="09be0-138">Azure Web Apps에서 호스팅하는 데 사용할 수 있는 대체 도구는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-138">Alternatives to hosting in Azure Web Apps:</span></span>

* <span data-ttu-id="09be0-139">[VM][compare-vm-hosting] - 높은 수준의 제어가 필요하거나 Web Apps에서 가능하지 않은 OS 구성 요소/서비스를 사용하는 워크로드의 경우(예: Windows GAC 또는 COM)</span><span class="sxs-lookup"><span data-stu-id="09be0-139">[VM][compare-vm-hosting] - For workloads that require a high degree of control, or depend on OS components / services that are not possible with Web Apps (e.g. the Windows GAC, or COM)</span></span>
* <span data-ttu-id="09be0-140">[컨테이너 호스팅][azure-containers] - OS 종속성과 호스팅 이식성 또는 호스팅 밀도가 필요한 경우</span><span class="sxs-lookup"><span data-stu-id="09be0-140">[Container Hosting][azure-containers] - Where there are OS dependencies and hosting portability, or hosting density, are also requirements.</span></span>
* <span data-ttu-id="09be0-141">[Service Fabric][service-fabric] - 워크로드 아키텍처가 높은 수준의 제어를 통해 클러스터 전체에 배포되고 실행되는 이점이 있는 분산된 구성 요소에 집중되는 경우에 좋은 옵션입니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-141">[Service Fabric][service-fabric] - A good option if the workload architecture is focused around distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="09be0-142">또한 컨테이너를 호스팅하는 데도 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-142">Service Fabric can also be used to host containers.</span></span>
* <span data-ttu-id="09be0-143">[서버리스 Azure 함수][azure-functions] - 워크로드 아키텍처가 세분화된 분산된 구성 요소에 집중되고, 최소한의 종속성이 필요하며, 개별 구성 요소가 요청 시에만 실행되고(불연속) 구성 요소의 오케스트레이션이 필요하지 않은 경우에 좋은 옵션입니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-143">[Serverless Azure functions][azure-functions] - A good option if the workload architecture is centered around fine grained distributed components, requiring minimal dependencies, where individual components are only required to run on demand (not continuously) and orchestration of components is not required.</span></span>

### <a name="devops"></a><span data-ttu-id="09be0-144">DevOps</span><span class="sxs-lookup"><span data-stu-id="09be0-144">DevOps</span></span>

<span data-ttu-id="09be0-145">**[CI(지속적인 통합)][continuous-integration]** 는 둘 이상의 개별 개발자 또는 팀에서 공유 코드 베이스에 대해 작고 빈번한 변경을 지속적으로 커밋하는 안정적인 빌드를 시연하는 것을 목표로 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-145">**[Continuous Integration (CI)][continuous-integration]** should aim to demonstrate a stable build, with more than one individual developer or team continuously committing small, frequent changes to the shared codebase.</span></span>
<span data-ttu-id="09be0-146">지속적인 통합 파이프라인의 일환으로 다음을 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-146">As part of your Continuous Integration pipeline you should;</span></span>

* <span data-ttu-id="09be0-147">작은 크기의 코드를 자주 체크 인합니다(성공적으로 병합하기가 더 어려울 수 있으므로 더 크거나 복잡한 변경을 일괄 처리하지 않도록 방지합니다).</span><span class="sxs-lookup"><span data-stu-id="09be0-147">Check in small amounts of code frequently (avoid batching up larger or more complex changes as these can be harder to merge successfully)</span></span>
* <span data-ttu-id="09be0-148">충분한 코드 범위(불편한 경로 포함)를 사용하여 응용 프로그램의 구성 요소에 대한 단위 테스트를 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-148">Unit Test the components of your application with sufficient code coverage (including the unhappy paths)</span></span>
* <span data-ttu-id="09be0-149">빌드가 공유 마스터(또는 트렁크) 분기에 대해 실행되는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-149">Ensuring the build is run against the shared, master (or trunk) branch.</span></span> <span data-ttu-id="09be0-150">이 분기는 안정적이어야 하며 "배포 준비 완료" 상태로 유지되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-150">This branch should be stable and maintained as "deployment ready".</span></span> <span data-ttu-id="09be0-151">불완전하거나 작업 진행 중인 변경은 나중에 충돌하지 않도록 방지하기 위해 빈번한 '정방향 통합' 병합을 통해 별도의 분기에서 격리되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-151">Incomplete or work-in-progress changes should be isolated in a separate branch with frequent 'forward integration' merges to avoid conflicts later.</span></span>

<span data-ttu-id="09be0-152">**[CD(지속적인 업데이트)][continuous-delivery]** 는 안정적인 빌드뿐만 아니라 안정적인 배포를 시연하는 것을 목표로 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-152">**[Continuous Delivery (CD)][continuous-delivery]** should aim to demonstrate not only a stable build but a stable deployment.</span></span> <span data-ttu-id="09be0-153">이에 따라 CD를 인식하기가 좀 더 어렵고, 환경 특정 구성이 필요하며, 이러한 값을 올바르게 설정하는 메커니즘이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-153">This makes realising CD a little more difficult, environment specific configuration is required and a mechanism for setting those values correctly.</span></span>

<span data-ttu-id="09be0-154">또한 다양한 구성 요소가 종단 간에 올바르게 구성되고 작동하도록 보장하기 위해 통합 테스트의 충분한 적용 범위가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-154">In addition, sufficient coverage of Integration Testing is required to ensure that the various components are configured and working correctly end-to-end.</span></span>

<span data-ttu-id="09be0-155">환경 특정 데이터를 설정 및 재설정하고 데이터베이스 스키마 버전을 관리해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-155">This may also require setting up and resetting environment specific data and managing database schema versions.</span></span>

<span data-ttu-id="09be0-156">또한 지속적인 업데이트는 부하 테스트 및 사용자 승인 테스트 환경으로 확장될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-156">Continuous Delivery may also extend to Load Testing and User Acceptance Testing Environments.</span></span>

<span data-ttu-id="09be0-157">지속적인 업데이트는 이상적으로 모든 환경에서 지속적인 모니터링을 통해 이점을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-157">Continuous Delivery benefits from continuous Monitoring, ideally across all environments.</span></span>
<span data-ttu-id="09be0-158">만들기 및 구성 또는 호스팅 인프라를 스크립팅하여 환경 전반에 걸쳐 배포 및 통합 테스트의 일관성과 안정성을 더 쉽게 만들 수 있습니다(클라우드 기반 워크로드에서 상당히 쉬운 작업, Azure 인프라를 코드로 변환(Infrastructure as Code) 참조). 이를 ["infrastructure-as-code"][infra-as-code]라고도 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-158">The consistency and reliability of deployments and integration testing across environments is made easier by scripting the creation and configuration or the hosting infrastructure (something which is considerably easier for Cloud based workloads, see Azure infrastructure as code) - this is also known as ["infrastructure-as-code"][infra-as-code].</span></span>

* <span data-ttu-id="09be0-159">지속적인 업데이트는 프로젝트 수명 주기에서 가능한 한 빨리 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-159">Start Continuous Delivery as early as possible in the project life-cycle.</span></span> <span data-ttu-id="09be0-160">늦게 시작할수록 더 어려워집니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-160">The later you leave it, the more difficult it will be.</span></span>
* <span data-ttu-id="09be0-161">통합 및 단위 테스트에는 프로젝트 기능과 동일한 우선 순위를 부여해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-161">Integration & unit tests should be given the same priority as the project features</span></span>
* <span data-ttu-id="09be0-162">환경에 독립적인 배포 패키지를 사용하고, 릴리스 프로세스를 통해 환경 특정 구성을 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-162">Use environment agnostic deployment packages and manage environment specific configuration through the release process.</span></span>
* <span data-ttu-id="09be0-163">릴리스 프로세스 중에 릴리스 관리 도구 내에서 또는 HSM(하드웨어 보안 모듈)이나 [Key Vault][azure-key-vault]로 호출하여 중요한 구성을 보호합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-163">Protect sensitive configuration within the release management tooling or by calling out to a Hardware-security-module (HSM), or [Key Vault][azure-key-vault], during the release process.</span></span> <span data-ttu-id="09be0-164">소스 제어 내에는 중요한 구성을 저장하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-164">Do not store sensitive configuration within source control.</span></span>

<span data-ttu-id="09be0-165">**지속적인 학습** - CD 환경에 대한 가장 효과적인 모니터링은 Microsoft의 [Application Insights][application-insights]와 같은 APM(Application-Performance-Monitoring) 도구에서 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-165">**Continuous Learning** - The most effective monitoring of a CD environment is provided by Application-Performance-Monitoring tools (APM for short), for example Microsoft's [Application Insights][application-insights].</span></span> <span data-ttu-id="09be0-166">응용 프로그램 워크로드에 대한 충분한 모니터링은 버그, 로드 시 성능을 파악하는 데 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-166">Sufficient depth of monitoring for an application workload is critical to understand bugs, performance under load.</span></span> <span data-ttu-id="09be0-167">[App Insight를 VSTS에 통합하여 CD 파이프라인을 지속적으로 모니터링할 수 있습니다][app-insights-cd-monitoring].</span><span class="sxs-lookup"><span data-stu-id="09be0-167">[App Insights can be integrated into VSTS to enable continuous monitoring of the CD pipeline][app-insights-cd-monitoring].</span></span> <span data-ttu-id="09be0-168">이를 통해 사용자 개입 없이 자동으로 다음 단계로 진행하거나 경고가 검색되면 롤백할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-168">This could be used to enable automatic progression to the next stage, without human intervention, or rollback if an alert is detected.</span></span>

## <a name="considerations"></a><span data-ttu-id="09be0-169">고려 사항</span><span class="sxs-lookup"><span data-stu-id="09be0-169">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="09be0-170">가용성</span><span class="sxs-lookup"><span data-stu-id="09be0-170">Availability</span></span>

<span data-ttu-id="09be0-171">클라우드 응용 프로그램을 구축하는 경우 [일반적인 가용성 디자인 패턴][design-patterns-availability]을 활용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-171">Consider leveraging the [typical design patterns for availability][design-patterns-availability] when building your cloud application.</span></span>

<span data-ttu-id="09be0-172">적절한 [App Service 웹 응용 프로그램 참조 아키텍처][app-service-reference-architecture]의 가용성 고려 사항을 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-172">Review the availability considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture]</span></span>

<span data-ttu-id="09be0-173">다른 가용성 항목은 Azure 아키텍처 센터의 [가용성 검사 목록][availability]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="09be0-173">For other availability topics, see the [availability checklist][availability] in the Azure Architecure Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="09be0-174">확장성</span><span class="sxs-lookup"><span data-stu-id="09be0-174">Scalability</span></span>

<span data-ttu-id="09be0-175">클라우드 응용 프로그램을 구축하는 경우 [일반적인 확장성 디자인 패턴][design-patterns-scalability]에 대해 알고 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-175">When building a cloud application be aware of the [typical design patterns for scalability][design-patterns-scalability].</span></span>

<span data-ttu-id="09be0-176">적절한 [App Service 웹 응용 프로그램 참조 아키텍처][app-service-reference-architecture]의 확장성 고려 사항을 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-176">Review the scalability considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture]</span></span>

<span data-ttu-id="09be0-177">다른 확장성 항목은 Azure 아키텍처 센터의 [확장성 검사 목록][scalability]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="09be0-177">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecure Center.</span></span>

### <a name="security"></a><span data-ttu-id="09be0-178">보안</span><span class="sxs-lookup"><span data-stu-id="09be0-178">Security</span></span>

<span data-ttu-id="09be0-179">적절한 경우 [일반적인 보안 디자인 패턴][design-patterns-security]을 활용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-179">Consider leveraging the [typical design patterns for security][design-patterns-security] where appropriate.</span></span>

<span data-ttu-id="09be0-180">적절한 [App Service 웹 응용 프로그램 참조 아키텍처][app-service-reference-architecture]의 보안 고려 사항을 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-180">Review the security considerations in the appropriate [App Service web application reference architecture][app-service-reference-architecture].</span></span>

<span data-ttu-id="09be0-181">보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="09be0-181">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="09be0-182">복원력</span><span class="sxs-lookup"><span data-stu-id="09be0-182">Resiliency</span></span>

<span data-ttu-id="09be0-183">[일반적인 복원력 디자인 패턴][design-patterns-resiliency]을 검토하고, 적절할 경우 이를 구현하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-183">Review the [typical design patterns for resiliency][design-patterns-resiliency] and consider implementing these where appropriate.</span></span>

<span data-ttu-id="09be0-184">아키텍처 서비스 센터에서 [App Service에 대한 다양한 복원력 권장 사례][resiliency-app-service]를 찾을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-184">You can find a number of [resiliency recommended practices for App Service][resiliency-app-service] on the architecture center.</span></span>

<span data-ttu-id="09be0-185">복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 응용 프로그램 디자인][resiliency]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="09be0-185">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="09be0-186">시나리오 배포</span><span class="sxs-lookup"><span data-stu-id="09be0-186">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="09be0-187">필수 조건</span><span class="sxs-lookup"><span data-stu-id="09be0-187">Prerequisites</span></span>

* <span data-ttu-id="09be0-188">기존 Azure 계정이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-188">You must have an existing Azure account.</span></span> <span data-ttu-id="09be0-189">Azure 구독이 없는 경우 시작하기 전에 [체험 계정][azure-free-account]을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-189">If you don't have an Azure subscription, create a [free account][azure-free-account] before you begin.</span></span>
* <span data-ttu-id="09be0-190">기존 VSTS(Visual Studio Team Services) 계정이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-190">You must have an existing Visual Studio Team Services (VSTS) account.</span></span> <span data-ttu-id="09be0-191">자세한 내용은 [VSTS(Visual Studio Team Services) 계정 만들기][vsts-account-create]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="09be0-191">Find out more details about [creating a Visual Studio Team Services (VSTS) account][vsts-account-create].</span></span>

### <a name="walk-through"></a><span data-ttu-id="09be0-192">방법 설명</span><span class="sxs-lookup"><span data-stu-id="09be0-192">Walk through</span></span>

<span data-ttu-id="09be0-193">이 시나리오에서는 Azure DevOps 프로젝트를 사용하여 CI/CD 파이프라인을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-193">In this scenario you'll use the Azure DevOps Project to create your CI/CD pipeline.</span></span>

<span data-ttu-id="09be0-194">DevOps 프로젝트는 App Service 계획, App Service 및 App Insights 리소스를 배포하고 Visual Studio Team Services 프로젝트를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-194">The DevOps project will deploy an App Service Plan, App Service and an App Insights resource for you, as well as configure the Visual Studio Team Services Project for you.</span></span>

<span data-ttu-id="09be0-195">DevOps 프로젝트를 만들고 빌드가 완료되면 관련 코드 변경, 작업 항목 및 테스트 결과를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-195">Once you've  the DevOps project and the build is completed, review the associated code changes, work items and test results.</span></span> <span data-ttu-id="09be0-196">실행할 모든 테스트가 코드에 포함되어 있지 않으므로 테스트 결과가 표시되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-196">You will notice no test results are displayed, as the code does not contain any tests to run.</span></span>

<span data-ttu-id="09be0-197">릴리스 정의를 검토합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-197">Review the Release definitions.</span></span> <span data-ttu-id="09be0-198">릴리스 파이프라인이 설정되어 응용 프로그램이 개발 환경으로 릴리스되었습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-198">Notice that a release pipeline has been setup, releasing our application into Dev.</span></span> <span data-ttu-id="09be0-199">**삭제** 빌드 아티팩트에서 **지속적인 배포 트리거**가 설정되어 있고, 개발 환경으로 자동 릴리스되는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-199">Observe that there is a **continuous deployment trigger** set from the **Drop** build artifact, with automatic releases into the Dev environments.</span></span> <span data-ttu-id="09be0-200">지속적인 배포 프로세스의 일부로 릴리스가 여러 환경에 걸쳐 있음을 알 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-200">As part of a Continuous Deployment process, you may see releases span across multiple environments.</span></span> <span data-ttu-id="09be0-201">릴리스는 인프라를 모두 확장하고(인프라를 코드로 변환(Infrastructure as Code)과 같은 기술을 사용), 필요한 모든 응용 프로그램 패키지와 구성 후 작업도 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-201">A release can span both infrastructure (using techniques such as Infrastructure as Code), and also deploy the application packages required as well as any post-configuration tasks.</span></span>

<span data-ttu-id="09be0-202">**추가 고려 사항**</span><span class="sxs-lookup"><span data-stu-id="09be0-202">**Additional Considerations.**</span></span>

* <span data-ttu-id="09be0-203">VSTS 마켓플레이스에서 사용할 수 있는 [토큰화 작업][vsts-tokenization] 중 하나를 활용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-203">Consider leveraging one of the [tokenization tasks][vsts-tokenization] that are available in the VSTS marketplace.</span></span>
* <span data-ttu-id="09be0-204">[배포: Azure Key Vault][download-keyvault-secrets] VSTS 작업을 사용하여 Azure Key Vault에서 비밀을 다운로드하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-204">Consider using the [Deploy: Azure Key Vault][download-keyvault-secrets] VSTS task to download secrets from an Azure KeyVault into your release.</span></span> <span data-ttu-id="09be0-205">그런 다음, 릴리스 정의의 일부로서 해당 비밀을 변수로 사용할 수 있으며 소스 제어에 저장하지 않아야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-205">You can then use those secrets as variables as part of your release definition, and should not be storing them in source control.</span></span>
* <span data-ttu-id="09be0-206">릴리스 환경에서 [릴리스 변수][vsts-release-variables]를 사용하여 환경 구성을 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-206">Consider using [release variables][vsts-release-variables] in your release definitions to drive configuration changes of your environments.</span></span> <span data-ttu-id="09be0-207">릴리스 변수의 범위는 전체 릴리스 또는 지정된 환경으로 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-207">Release variables can be scoped to an entire release or a given environment.</span></span> <span data-ttu-id="09be0-208">비밀 정보에 변수를 사용하는 경우 자물쇠 아이콘을 선택해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-208">If using variables for secret information, ensure that you select the padlock icon.</span></span>
* <span data-ttu-id="09be0-209">릴리스 파이프라인에서 [배포 게이트][vsts-deployment-gates]를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-209">Consider using [deployment gates][vsts-deployment-gates] in your release pipeline.</span></span> <span data-ttu-id="09be0-210">이렇게 하면 외부 시스템(예: 인시던트 관리 또는 추가 맞춤형 시스템)과 공동으로 모니터링 데이터를 활용하여 릴리스를 승격해야 하는지 여부를 결정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-210">This allows you to leverage monitoring data in association with external systems (e.g. incident management or additional bespoke systems) to determine whether a release should be promoted.</span></span>
* <span data-ttu-id="09be0-211">릴리스 파이프라인에 수동으로 개입해야 하는 경우 [승인][vsts-approvals] 기능을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-211">Where manual intervention in a release pipeline is required, consider using the [approvals][vsts-approvals] functionality.</span></span>
* <span data-ttu-id="09be0-212">릴리스 파이프라인에서 가능한 한 빨리 [Application Insights][application-insights] 및 추가 모니터링 도구를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-212">Consider using [Application Insights][application-insights] and additional monitoring tooling as early as possible in your release pipeline.</span></span> <span data-ttu-id="09be0-213">대부분의 조직에서는 프로세스 초기에 잠재적인 버그를 식별하고 프로덕션 환경에서 사용자에게 미치는 영향을 방지할 수 있지만, 프로덕션 환경에서만 모니터링을 시작합니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-213">Most organizations only begin monitoring in their production environment, though you could identify potential bugs earlier in the process and prevent impact to your users in production.</span></span>

## <a name="pricing"></a><span data-ttu-id="09be0-214">가격</span><span class="sxs-lookup"><span data-stu-id="09be0-214">Pricing</span></span>

<span data-ttu-id="09be0-215">Visual Studio Team Services 비용은 필요한 동시 빌드/릴리스 수 및 테스트 사용자 수와 같은 요소 외에도 액세스해야 하는 조직의 사용자 수에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-215">Your Visual Studio Team Services costing will depend upon the number of users in your organization that require access, in addition to factors such as the number of concurrent build/releases required, and number of test users.</span></span> <span data-ttu-id="09be0-216">자세한 내용은 [VSTS 가격 페이지][vsts-pricing-page]에 자세히 나와 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-216">These are detailed further on the [VSTS pricing page][vsts-pricing-page].</span></span>

* <span data-ttu-id="09be0-217">[VSTS(Visual Studio Team Services)][vsts-pricing-calculator]는 개발 수명 주기를 관리하고 사용자별 및 월별 기준으로 지불하는 데 사용할 수 있는 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-217">[Visual Studio Team Services (VSTS)][vsts-pricing-calculator] is a service that enables you to manage your development life cycle and is paid for on a per user, per month basis.</span></span> <span data-ttu-id="09be0-218">추가 테스트 사용자 또는 사용자 기본 라이선스 외에도 필요한 동시 파이프라인 수에 따라 추가 요금이 부과될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="09be0-218">There may be additional charges dependent upon concurrent pipelines needed, in addition to any additional test users, or user basic licenses.</span></span>

## <a name="related-resources"></a><span data-ttu-id="09be0-219">관련 리소스</span><span class="sxs-lookup"><span data-stu-id="09be0-219">Related Resources</span></span>

* <span data-ttu-id="09be0-220">[DevOps란?][devops-whatis]</span><span class="sxs-lookup"><span data-stu-id="09be0-220">[What is DevOps?][devops-whatis]</span></span>
* <span data-ttu-id="09be0-221">[Microsoft의 DevOps - Visual Studio Team Services를 사용하는 방법][devops-microsoft]</span><span class="sxs-lookup"><span data-stu-id="09be0-221">[DevOps at Microsoft - How we work with Visual Studio Team Services][devops-microsoft]</span></span>
* <span data-ttu-id="09be0-222">[단계별 자습서: Visual Studio Team Services를 사용하는 DevOps][devops-with-vsts]</span><span class="sxs-lookup"><span data-stu-id="09be0-222">[Step-by-step Tutorials: DevOps with Visual Studio Team Services][devops-with-vsts]</span></span>
* <span data-ttu-id="09be0-223">[Azure DevOps 프로젝트를 사용하여 .NET용 CI/CD 파이프라인 만들기][devops-project-create]</span><span class="sxs-lookup"><span data-stu-id="09be0-223">[Create a CI/CD pipeline for .NET with the Azure DevOps project][devops-project-create]</span></span>

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: /azure/architecture/reference-architectures/app-service-web-app/
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/devops-dotnet-webapp/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[application-insights]: https://azure.microsoft.com/en-gb/services/application-insights/
[cloud-based-load-testing]: https://visualstudio.microsoft.com/team-services/cloud-load-testing/
[cloud-based-load-testing-on-premises]: /vsts/test/load-test/clt-with-private-machines?view=vsts
[jenkins-on-azure]: /azure/jenkins/
[devops-whatis]: /azure/devops/what-is-devops
[download-keyvault-secrets]: /vsts/pipelines/tasks/deploy/azure-key-vault?view=vsts
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[terraform]: /azure/terraform/
[vsts-account-create]: /vsts/organizations/accounts/create-account-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/en-us/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]:/azure/service-fabric/
[azure-functions]:/azure/azure-functions/
[azure-containers]:https://azure.microsoft.com/en-us/overview/containers/
[compare-vm-hosting]:/azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]:/azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]:/azure/best-practices-availability-paired-regions
[devops-project-create]: /vsts/pipelines/apps/cd/azure/azure-devops-project-aspnetcore?view=vsts
[security]: /azure/security/