---
title: "외부 구성 저장소"
description: "구성 정보를 응용 프로그램 배포 패키지에서 중앙 위치로 이동합니다."
keywords: "디자인 패턴"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- management-monitoring
ms.openlocfilehash: 733ca979903d1526d3a1a6b281a8903893e19fda
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="external-configuration-store-pattern"></a><span data-ttu-id="76000-104">외부 구성 저장소 패턴</span><span class="sxs-lookup"><span data-stu-id="76000-104">External Configuration Store pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="76000-105">구성 정보를 응용 프로그램 배포 패키지에서 중앙 위치로 이동합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-105">Move configuration information out of the application deployment package to a centralized location.</span></span> <span data-ttu-id="76000-106">이렇게 하면 구성 데이터를 더 쉽게 관리하고 제어하며 구성 데이터를 응용 프로그램과 응용 프로그램 인스턴스에서 공유할 기회를 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-106">This can provide opportunities for easier management and control of configuration data, and for sharing configuration data across applications and application instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="76000-107">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="76000-107">Context and problem</span></span>

<span data-ttu-id="76000-108">대부분의 응용 프로그램 런타임 환경은 응용 프로그램과 함께 배포된 파일에 보관되는 구성 정보를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-108">The majority of application runtime environments include configuration information that's held in files deployed with the application.</span></span> <span data-ttu-id="76000-109">그런데 배포 후 이런 파일을 편집해 응용 프로그램 동작을 변경할 수 없는 경우가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-109">In some cases, it's possible to edit these files to change the application behavior after it's been deployed.</span></span> <span data-ttu-id="76000-110">그런 경우 구성을 변경하려면 응용 프로그램을 재배포해야 하는데, 응용 프로그램의 재배포는 용납할 수 없는 가동 중지 시간 및 다른 관리 오버헤드를 초래하는 경우가 많습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-110">However, changes to the configuration require the application be redeployed, often resulting in unacceptable downtime and other administrative overhead.</span></span>

<span data-ttu-id="76000-111">로컬 구성 파일 역시 구성을 단일 응용 프로그램으로 제한하지만, 때로 여러 응용 프로그램에서 구성 설정을 공유하는 데 로컬 구성 파일이 유용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-111">Local configuration files also limit the configuration to a single application, but sometimes it would be useful to share configuration settings across multiple applications.</span></span> <span data-ttu-id="76000-112">그와 같은 사례로는 데이터베이스 연결 문자열, UI 테마 정보, 큐의 URL 및 응용 프로그램의 관련 집합이 사용하는 저장소를 꼽을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-112">Examples include database connection strings, UI theme information, or the URLs of queues and storage used by a related set of applications.</span></span>

<span data-ttu-id="76000-113">문제는 특히 클라우드 호스팅 시나리오와 관련해 응용 프로그램에서 실행 중인 여러 인스턴스의 로컬 구성 변경 내용을 관리하기가 어렵다는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="76000-113">It's challenging to manage changes to local configurations across multiple running instances of the application, especially in a cloud-hosted scenario.</span></span> <span data-ttu-id="76000-114">그 결과 업데이트를 배포하는 동안 여러 구성 설정을 사용하는 인스턴스가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-114">It can result in instances using different configuration settings while the update is being deployed.</span></span>

<span data-ttu-id="76000-115">또한 응용 프로그램과 구성 요소를 업데이트하려면 구성 스키마를 변경해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-115">In addition, updates to applications and components might require changes to configuration schemas.</span></span> <span data-ttu-id="76000-116">대부분의 구성 시스템은 구성 정보의 다양한 버전을 지원하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-116">Many configuration systems don't support different versions of configuration information.</span></span>

## <a name="solution"></a><span data-ttu-id="76000-117">해결 방법</span><span class="sxs-lookup"><span data-stu-id="76000-117">Solution</span></span>

<span data-ttu-id="76000-118">구성 정보를 외부 저장소에 저장하고 구성 설정을 빠르고 효율적으로 읽고 업데이트하는 데 사용할 수 있는 인터페이스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-118">Store the configuration information in external storage, and provide an interface that can be used to quickly and efficiently read and update configuration settings.</span></span> <span data-ttu-id="76000-119">외부 저장소의 유형은 응용 프로그램의 호스팅과 런타임 환경에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="76000-119">The type of external store depends on the hosting and runtime environment of the application.</span></span> <span data-ttu-id="76000-120">클라우드 호스팅 시나리오에서는 보통 클라우드 기반 저장소 서비스이지만 호스티드 데이터베이스 또는 다른 시스템이 될 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-120">In a cloud-hosted scenario it's typically a cloud-based storage service, but could be a hosted database or other system.</span></span>

<span data-ttu-id="76000-121">구성 정보를 위해 선택하는 백업 저장소는 일관되고 사용하기 쉬운 액세스를 제공하는 인터페이스를 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-121">The backing store you choose for configuration information should have an interface that provides consistent and easy-to-use access.</span></span> <span data-ttu-id="76000-122">인터페이스는 올바르게 형식화된 정보를 구조화된 형식으로 표시해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-122">It should expose the information in a correctly typed and structured format.</span></span> <span data-ttu-id="76000-123">구현도 구성 데이터를 보호하기 위해 사용자 액세스에 권한을 부여해야 하고 구성의 여러 버전(각각의 구성별로 여러 릴리스 버전을 포함하는 개발, 준비, 프로덕션 등)을 저장할 수 있을 정도로 충분히 유연해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-123">The implementation might also need to authorize users’ access in order to protect configuration data, and be flexible enough to allow storage of multiple versions of the configuration (such as development, staging, or production, including multiple release versions of each one).</span></span>

> <span data-ttu-id="76000-124">많은 기본 제공 구성 시스템은 응용 프로그램이 시작될 때 데이터를 읽고 데이터를 메모리에 캐시해 빠른 액세스를 제공하고 응용 프로그램 성능에 미치는 영향을 최소화합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-124">Many built-in configuration systems read the data when the application starts up, and cache the data in memory to provide fast access and minimize the impact on application performance.</span></span> <span data-ttu-id="76000-125">사용하는 백업 저장소의 유형과 백업 저장소의 대기 시간에 따라 외부 구성 저장소 내에 캐싱 메커니즘을 구현하는 것이 유용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-125">Depending on the type of backing store used, and the latency of this store, it might be helpful to implement a caching mechanism within the external configuration store.</span></span> <span data-ttu-id="76000-126">자세한 내용은 [캐싱 지침](https://msdn.microsoft.com/library/dn589802.aspx)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="76000-126">For more information, see the [Caching Guidance](https://msdn.microsoft.com/library/dn589802.aspx).</span></span> <span data-ttu-id="76000-127">다음 그림은 선택적 로컬 캐시가 있는 외부 구성 저장소 패턴의 개요를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="76000-127">The figure illustrates an overview of the External Configuration Store pattern with optional local cache.</span></span>

![선택적 로컬 캐시가 있는 외부 구성 저장소 패턴의 개요](./_images/external-configuration-store-overview.png)


## <a name="issues-and-considerations"></a><span data-ttu-id="76000-129">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="76000-129">Issues and considerations</span></span>

<span data-ttu-id="76000-130">이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="76000-130">Consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="76000-131">수락할 수 있는 성능, 고가용성, 견고성을 제공하고 응용 프로그램 유지와 관리 프로세스의 일부로 백업할 수 있는 백업 저장소를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-131">Choose a backing store that offers acceptable performance, high availability, robustness, and can be backed up as part of the application maintenance and administration process.</span></span> <span data-ttu-id="76000-132">클라우드 호스티드 응용 프로그램에서 클라우드 저장소 메커니즘의 사용은 이런 요구 사항을 충족하는 일반적인 선택입니다.</span><span class="sxs-lookup"><span data-stu-id="76000-132">In a cloud-hosted application, using a cloud storage mechanism is usually a good choice to meet these requirements.</span></span>

<span data-ttu-id="76000-133">백업 저장소의 스키마를 보관할 수 있는 정보 유형에 유연성을 제공할 수 있는 방식으로 설계합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-133">Design the schema of the backing store to allow flexibility in the types of information it can hold.</span></span> <span data-ttu-id="76000-134">형식화된 데이터, 설정 모음, 설정의 여러 버전 및 사용 중인 응용 프로그램에 필요한 다른 모든 기능과 같은 구성 요구 사항을 모두 제공하는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-134">Ensure that it provides for all configuration requirements such as typed data, collections of settings, multiple versions of settings, and any other features that the applications using it require.</span></span> <span data-ttu-id="76000-135">요구 사항이 변경될 때 추가 설정을 지원하도록 스키마는 확장하기 쉬워야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-135">The schema should be easy to extend to support additional settings as requirements change.</span></span>

<span data-ttu-id="76000-136">백업 저장소의 물리적 기능, 이런 기능을 구성 정보가 저장되는 방식과 연관시키는 방법 및 성능에 미치는 영향을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-136">Consider the physical capabilities of the backing store, how it relates to the way configuration information is stored, and the effects on performance.</span></span> <span data-ttu-id="76000-137">예를 들어 구성 정보를 포함하는 XML 문서를 저장하려면 문서를 구문 분석하여 개별 설정을 읽는 구성 인터페이스 또는 응용 프로그램이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-137">For example, storing an XML document containing configuration information will require either the configuration interface or the application to parse the document in order to read individual settings.</span></span> <span data-ttu-id="76000-138">그러나 이런 사례에서 설정의 캐싱은 느려지는 읽기 성능을 보완하는 데 도움을 줄 수 있지만 설정의 업데이트를 더 복잡하게 만든다는 단점이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-138">It'll make updating a setting more complicated, though caching the settings can help to offset slower read performance.</span></span>

<span data-ttu-id="76000-139">구성 인터페이스가 구성 설정의 범위와 상속을 제어하도록 허용하는 방법을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-139">Consider how the configuration interface will permit control of the scope and inheritance of configuration settings.</span></span> <span data-ttu-id="76000-140">예를 들면 조직, 응용 프로그램 및 컴퓨터 수준에서 구성 설정의 범위를 지정하는 요구 사항이 여기에 해당할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-140">For example, it might be a requirement to scope configuration settings at the organization, application, and the machine level.</span></span> <span data-ttu-id="76000-141">한편 다양한 범위의 액세스에 대한 제어의 위임을 지원하고 개별 응용 프로그램에서 설정의 재정의를 방지 또는 허용할 필요가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-141">It might need to support delegation of control over access to different scopes, and to prevent or allow individual applications to override settings.</span></span>

<span data-ttu-id="76000-142">구성 인터페이스가 구성 데이터를 형식화된 값, 모음, 키/값 쌍 또는 속성 모음과 같은 필요한 형식으로 표시할 수 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-142">Ensure that the configuration interface can expose the configuration data in the required formats such as typed values, collections, key/value pairs, or property bags.</span></span>

<span data-ttu-id="76000-143">설정이 오류를 포함하거나 백업 저장소에 설정이 없을 때 구성 저장소 인터페이스의 동작 방법을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-143">Consider how the configuration store interface will behave when settings contain errors, or don't exist in the backing store.</span></span> <span data-ttu-id="76000-144">기본 설정으로 되돌리고 오류를 로그하는 것이 적절할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-144">It might be appropriate to return default settings and log errors.</span></span> <span data-ttu-id="76000-145">또한 구성 설정 키 또는 이름의 대/소문자 구분, 이진 데이터의 저장과 처리 및 null 값 또는 빈 값을 처리하는 방식과 같은 측면도 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-145">Also consider aspects such as the case sensitivity of configuration setting keys or names, the storage and handling of binary data, and the ways that null or empty values are handled.</span></span>

<span data-ttu-id="76000-146">적절한 사용자와 응용 프로그램에만 액세스를 허용하도록 구성 데이터를 보호하는 방법을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-146">Consider how to protect the configuration data to allow access to only the appropriate users and applications.</span></span> <span data-ttu-id="76000-147">이런 방법에는 구성 저장소 인터페이스의 기능이 해당할 수 있지만, 적절한 권한 없이 백업 저장소의 데이터에 직접 액세스할 수 없게 하는 방식도 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-147">This is likely a feature of the configuration store interface, but it's also necessary to ensure that the data in the backing store can't be accessed directly without the appropriate permission.</span></span> <span data-ttu-id="76000-148">구성 데이터를 읽고 쓰는 데 필요한 권한을 엄격하게 분리합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-148">Ensure strict separation between the permissions required to read and to write configuration data.</span></span> <span data-ttu-id="76000-149">또한 구성 설정의 일부 또는 전부에 대한 암호화가 필요한지 여부 및 암호화를 구성 저장소 인터페이스에 구현하는 방법도 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-149">Also consider whether you need to encrypt some or all of the configuration settings, and how this'll be implemented in the configuration store interface.</span></span>

<span data-ttu-id="76000-150">런타임 중 응용 프로그램 동작을 변경하는 중앙 집중식으로 저장하는 구성은 굉장히 중요하며 응용 프로그램 코드의 배포와 동일한 방식을 사용해 배포, 업데이트, 관리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-150">Centrally stored configurations, which change application behavior during runtime, are critically important and should be deployed, updated, and managed using the same mechanisms as deploying application code.</span></span> <span data-ttu-id="76000-151">예를 들어 둘 이상의 응용 프로그램에 영향을 줄 수 있는 변경은 완벽하게 테스트하고 준비한 배포 접근 방식을 사용하여 수행함으로써, 변경이 이런 구성을 사용하는 모든 응용 프로그램에 적절하다는 것을 보장해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-151">For example, changes that can affect more than one application must be carried out using a full test and staged deployment approach to ensure that the change is appropriate for all applications that use this configuration.</span></span> <span data-ttu-id="76000-152">관리자가 하나의 응용 프로그램을 업데이트하기 위해 설정을 편집하는 경우, 동일한 설정을 사용하는 다른 응용 프로그램에 악영향을 미칠 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-152">If an administrator edits a setting to update one application, it could adversely impact other applications that use the same setting.</span></span>

<span data-ttu-id="76000-153">응용 프로그램이 구성 정보를 캐시하는 경우, 구성 정보가 변경되면 응용 프로그램도 변경해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-153">If an application caches configuration information, the application needs to be alerted if the configuration changes.</span></span> <span data-ttu-id="76000-154">캐시된 구성 데이터에 만료 정책을 구현하여 캐시된 정보를 주기적으로 자동 새로 고침하고 모든 변경 내용을 선택하여 반영하도록 조치할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-154">It might be possible to implement an expiration policy over cached configuration data so that this information is automatically refreshed periodically and any changes picked up (and acted on).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="76000-155">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="76000-155">When to use this pattern</span></span>

<span data-ttu-id="76000-156">이 패턴은 다음에 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-156">This pattern is useful for:</span></span>

- <span data-ttu-id="76000-157">구성 설정이 여러 응용 프로그램과 응용 프로그램 인스턴스에 공유되는 경우 또는 여러 응용 프로그램과 응용 프로그램 인스턴스에 표준 구성을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="76000-157">Configuration settings that are shared between multiple applications and application instances, or where a standard configuration must be enforced across multiple applications and application instances.</span></span>

- <span data-ttu-id="76000-158">이미지 저장 또는 복잡한 데이터 유형과 같은 필요한 구성 설정을 모두 지원하지는 않는 표준 구성 시스템</span><span class="sxs-lookup"><span data-stu-id="76000-158">A standard configuration system that doesn't support all of the required configuration settings, such as storing images or complex data types.</span></span>

- <span data-ttu-id="76000-159">응용 프로그램의 일부 설정에 대한 보조 저장소로, 응용 프로그램이 중앙 집중식으로 저장하는 설정의 일부 또는 전부를 재정의할 수 있는 경우</span><span class="sxs-lookup"><span data-stu-id="76000-159">As a complementary store for some of the settings for applications, perhaps allowing applications to override some or all of the centrally-stored settings.</span></span>

- <span data-ttu-id="76000-160">여러 응용 프로그램의 관리를 단순화하는 방식으로 구성 저장소에 대한 액세스의 일부 또는 모든 유형을 로그하여 구성 설정의 사용을 선택적으로 모니터링하는 경우</span><span class="sxs-lookup"><span data-stu-id="76000-160">As a way to simplify administration of multiple applications, and optionally for monitoring use of configuration settings by logging some or all types of access to the configuration store.</span></span>

## <a name="example"></a><span data-ttu-id="76000-161">예</span><span class="sxs-lookup"><span data-stu-id="76000-161">Example</span></span>

<span data-ttu-id="76000-162">Microsoft Azure 호스티드 응용 프로그램에서 구성 정보를 외부적으로 저장하기 위한 대표적인 선택은 Azure Storage를 사용하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="76000-162">In a Microsoft Azure hosted application, a typical choice for storing configuration information externally is to use Azure Storage.</span></span> <span data-ttu-id="76000-163">Azure Storage는 복원력이 있고, 고성능을 제공하며, 자동 장애 조치(Failover)로 3번 복제되어 고가용성을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-163">This is resilient, offers high performance, and is replicated three times with automatic failover to offer high availability.</span></span> <span data-ttu-id="76000-164">Azure Table Storage는 값에 유연한 스키마를 사용할 수 있는 키/값 저장소를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-164">Azure Table storage provides a key/value store with the ability to use a flexible schema for the values.</span></span> <span data-ttu-id="76000-165">Azure Blob Storage는 데이터의 유형을 개별적으로 명명된 blob에 보관할 수 있는 계층적 컨테이너 기반 저장소를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-165">Azure Blob storage provides a hierarchical, container-based store that can hold any type of data in individually named blobs.</span></span>

<span data-ttu-id="76000-166">다음 예제는 구성 저장소를 구성 정보를 저장하고 표시하는 Blob 저장소로 구현하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="76000-166">The following example shows how a configuration store can be implemented over Blob storage to store and expose configuration information.</span></span> <span data-ttu-id="76000-167">`BlobSettingsStore` 클래스는 구성 정보를 보관하는 Blob 저장소를 추상화하고, 다음 코드에 제시되는 `ISettingsStore` 인터페이스를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-167">The `BlobSettingsStore` class abstracts Blob storage for holding configuration information, and implements the `ISettingsStore` interface shown in the following code.</span></span>

> <span data-ttu-id="76000-168">이 코드는 _ExternalConfigurationStore_ 솔루션의 _ExternalConfigurationStore.Cloud_ 프로젝트에 포함되어 있고 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store)에서 다운로드할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-168">This code is provided in the _ExternalConfigurationStore.Cloud_ project in the _ExternalConfigurationStore_ solution, available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

<span data-ttu-id="76000-169">이 인터페이스는 구성 저장소에 보관된 구성 설정을 검색하고 업데이트하는 메서드를 정의하며 어떤 구성 설정이 최근에 수정되었는지를 검색하는 데 사용할 수 있는 버전 번호를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-169">This interface defines methods for retrieving and updating configuration settings held in the configuration store, and includes a version number that can be used to detect whether any configuration settings have been modified recently.</span></span> <span data-ttu-id="76000-170">`BlobSettingsStore` 클래스는 blob의 `ETag` 속성을 사용해 버전 관리를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-170">The `BlobSettingsStore` class uses the `ETag` property of the blob to implement versioning.</span></span> <span data-ttu-id="76000-171">`ETag` 속성은 blob에 쓸 때마다 자동으로 업데이트됩니다.</span><span class="sxs-lookup"><span data-stu-id="76000-171">The `ETag` property is updated automatically each time the blob is written.</span></span>

> <span data-ttu-id="76000-172">보통 이런 간단한 솔루션은 모든 구성 설정을 형식화된 값이 아닌 문자열 값으로 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-172">By design, this simple solution exposes all configuration settings as string values rather than typed values.</span></span>

<span data-ttu-id="76000-173">`ExternalConfigurationManager` 클래스는 `BlobSettingsStore` 개체를 감싸는 래퍼를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-173">The `ExternalConfigurationManager` class provides a wrapper around a `BlobSettingsStore` object.</span></span> <span data-ttu-id="76000-174">응용 프로그램은 이 클래스를 사용해 구성 정보를 저장하고 검색할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-174">An application can use this class to store and retrieve configuration information.</span></span> <span data-ttu-id="76000-175">이 클래스는 Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) 라이브러리를 사용하여 `IObservable` 인터페이스의 구현을 통해 구성에 이루어진 모든 변경 내용을 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-175">This class uses the Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) library to expose any changes made to the configuration through an implementation of the `IObservable` interface.</span></span> <span data-ttu-id="76000-176">`SetAppSetting` 메서드를 호출해 설정을 수정하면 `Changed` 이벤트가 발생하고 이 이벤트의 모든 구독자에게 알려집니다.</span><span class="sxs-lookup"><span data-stu-id="76000-176">If a setting is modified by calling the `SetAppSetting` method, the `Changed` event is raised and all subscribers to this event will be notified.</span></span>

<span data-ttu-id="76000-177">모든 설정은 빠른 액세스를 위해 `ExternalConfigurationManager` 클래스 내의 `Dictionary` 개체에 캐시되기도 합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-177">Note that all settings are also cached in a `Dictionary` object inside the `ExternalConfigurationManager` class for fast access.</span></span> <span data-ttu-id="76000-178">구성 설정을 검색하는 데 사용되는 `GetSetting` 메서드는 캐시에서 데이터를 읽습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-178">The `GetSetting` method used to retrieve a configuration setting reads the data from the cache.</span></span> <span data-ttu-id="76000-179">설정이 캐시에 없으면 그 대신 `BlobSettingsStore` 개체에서 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="76000-179">If the setting isn't found in the cache, it's retrieved from the `BlobSettingsStore` object instead.</span></span>

<span data-ttu-id="76000-180">`GetSettings` 메서드는 `CheckForConfigurationChanges` 메서드를 호출해 blob 저장소의 구성 정보가 변경되었는지 여부를 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-180">The `GetSettings` method invokes the `CheckForConfigurationChanges` method to detect whether the configuration information in blob storage has changed.</span></span> <span data-ttu-id="76000-181">이런 검색은 버전 번호를 검사하고 검사한 버전 번호와 `ExternalConfigurationManager` 개체에 보관된 현재 버전 번호의 비교를 통해 이루어집니다.</span><span class="sxs-lookup"><span data-stu-id="76000-181">It does this by examining the version number and comparing it with the current version number held by the `ExternalConfigurationManager` object.</span></span> <span data-ttu-id="76000-182">하나 이상의 변경 내용이 발생하면 `Changed` 이벤트가 발생하고 `Dictionary` 개체에 캐시된 구성 설정이 새로 고쳐집니다.</span><span class="sxs-lookup"><span data-stu-id="76000-182">If one or more changes have occurred, the `Changed` event is raised and the configuration settings cached in the `Dictionary` object are refreshed.</span></span> <span data-ttu-id="76000-183">이런 과정은 [캐시 배제 패턴](cache-aside.md)의 적용에 해당합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-183">This is an application of the [Cache-Aside pattern](cache-aside.md).</span></span>

<span data-ttu-id="76000-184">다음 코드 샘플은 `Changed` 이벤트, `GetSettings` 메서드 및 `CheckForConfigurationChanges` 메서드의 구현 방법을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="76000-184">The following code sample shows how the `Changed` event, the `GetSettings` method, and the `CheckForConfigurationChanges` method are implemented:</span></span>

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache. 
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> <span data-ttu-id="76000-185">`ExternalConfigurationManager` 클래스도 `Environment`로 명명된 속성을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-185">The `ExternalConfigurationManager` class also provides a property named `Environment`.</span></span> <span data-ttu-id="76000-186">이 속성은 준비 및 프로덕션과 같은 다양한 환경에서 실행하는 응용 프로그램을 위한 다양한 구성을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="76000-186">This property supports varying configurations for an application running in different environments, such as staging and production.</span></span>

<span data-ttu-id="76000-187">`ExternalConfigurationManager` 개체도 변경 내용을 확인하기 위해 `BlobSettingsStore` 개체를 주기적으로 쿼리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76000-187">An `ExternalConfigurationManager` object can also query the `BlobSettingsStore` object periodically for any changes.</span></span> <span data-ttu-id="76000-188">다음 코드에서 앞서 설명한 것처럼 `StartMonitor` 메서드는 주기적으로 `CheckForConfigurationChanges`를 호출하여 변경 내용을 검색하고 `Changed` 이벤트를 발생시킵니다.</span><span class="sxs-lookup"><span data-stu-id="76000-188">In the following code, the `StartMonitor` method calls `CheckForConfigurationChanges` at an interval to detect any changes and raise the `Changed` event, as described earlier.</span></span>

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

<span data-ttu-id="76000-189">`ExternalConfigurationManager` 클래스는 `ExternalConfiguration` 클래스를 통해 단일 인스턴스로 인스턴스화됩니다.</span><span class="sxs-lookup"><span data-stu-id="76000-189">The `ExternalConfigurationManager` class is instantiated as a singleton instance by the `ExternalConfiguration` class shown below.</span></span>

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

<span data-ttu-id="76000-190">다음 코드는 _ExternalConfigurationStore.Cloud_ 프로젝트의 `WorkerRole` 클래스에서 가져온 것입니다.</span><span class="sxs-lookup"><span data-stu-id="76000-190">The following code is taken from the `WorkerRole` class in the _ExternalConfigurationStore.Cloud_ project.</span></span> <span data-ttu-id="76000-191">여기서는 응용 프로그램이 `ExternalConfiguration` 클래스를 사용해 설정을 읽는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="76000-191">It shows how the application uses the `ExternalConfiguration` class to read a setting.</span></span>

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

<span data-ttu-id="76000-192">역시 `WorkerRole` 클래스에서 가져온 다음 코드도 응용 프로그램이 구성 이벤트를 구독하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="76000-192">The following code, also from the `WorkerRole` class, shows how the application subscribes to configuration events.</span></span>

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="76000-193">관련 패턴 및 지침</span><span class="sxs-lookup"><span data-stu-id="76000-193">Related patterns and guidance</span></span>

- <span data-ttu-id="76000-194">이 패턴을 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store)에서 사용할 수 있음을 보여주는 샘플.</span><span class="sxs-lookup"><span data-stu-id="76000-194">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).</span></span>
