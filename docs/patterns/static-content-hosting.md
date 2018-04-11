---
title: 정적 콘텐츠 호스팅
description: 정적 콘텐츠를 클라이언트에 직접 제공할 수 있는 클라우드 기반 저장소 서비스에 배포합니다.
keywords: 디자인 패턴
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: deb15001bea2598d56a2793be78bbc3e7473bdf3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="static-content-hosting-pattern"></a><span data-ttu-id="01442-104">정적 콘텐츠 호스팅 패턴</span><span class="sxs-lookup"><span data-stu-id="01442-104">Static Content Hosting pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="01442-105">정적 콘텐츠를 클라이언트에 직접 제공할 수 있는 클라우드 기반 저장소 서비스에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-105">Deploy static content to a cloud-based storage service that can deliver them directly to the client.</span></span> <span data-ttu-id="01442-106">이렇게 하면 잠재적으로 비용이 많이 들 수 있는 계산 인스턴스에 대한 필요성이 줄어듭니다.</span><span class="sxs-lookup"><span data-stu-id="01442-106">This can reduce the need for potentially expensive compute instances.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="01442-107">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="01442-107">Context and problem</span></span>

<span data-ttu-id="01442-108">웹 응용 프로그램에는 일반적으로 일부 정적 콘텐츠 요소가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-108">Web applications typically include some elements of static content.</span></span> <span data-ttu-id="01442-109">이 정적 콘텐츠에는 HTML 페이지, 그리고 HTML 페이지의 일부(예: 인라인 이미지, 스타일시트 및 클라이언트 쪽 JavaScript 파일) 혹은 별도의 다운로드(예: PDF 문서)로 클라이언트에서 사용할 수 있는 이미지 및 문서와 같은 기타 리소스가 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-109">This static content might include HTML pages and other resources such as images and documents that are available to the client, either as part of an HTML page (such as inline images, style sheets, and client-side JavaScript files) or as separate downloads (such as PDF documents).</span></span>

<span data-ttu-id="01442-110">웹 서버는 효율적인 동적 페이지 코드 실행 및 출력 캐싱을 통해 요청을 최적화하도록 잘 조정되어 있지만, 여전히 정적 콘텐츠의 다운로드 요청을 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-110">Although web servers are well tuned to optimize requests through efficient dynamic page code execution and output caching, they still have to handle requests to download static content.</span></span> <span data-ttu-id="01442-111">이를 위해 소비되는 처리 주기는 실상 다른 목적을 위해 보다 효율적으로 사용될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-111">This consumes processing cycles that could often be put to better use.</span></span>

## <a name="solution"></a><span data-ttu-id="01442-112">해결 방법</span><span class="sxs-lookup"><span data-stu-id="01442-112">Solution</span></span>

<span data-ttu-id="01442-113">대부분의 클라우드 호스팅 환경에서는 응용 프로그램의 리소스 및 정적 페이지 일부를 저장소 서비스에 배치하여 계산 인스턴스에 대한 필요성을 최소화할 수 있습니다(예를 들어 더 작은 인스턴스 또는 더 적은 수의 인스턴스 사용).</span><span class="sxs-lookup"><span data-stu-id="01442-113">In most cloud hosting environments it's possible to minimize the need for compute instances (for example, use a smaller instance or fewer instances), by locating some of an application’s resources and static pages in a storage service.</span></span> <span data-ttu-id="01442-114">클라우드에 호스트된 저장소의 비용은 일반적으로 계산 인스턴스의 경우보다 훨씬 저렴합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-114">The cost for cloud-hosted storage is typically much less than for compute instances.</span></span>

<span data-ttu-id="01442-115">응용 프로그램의 일부를 저장소 서비스에 호스트할 때 주요 고려 사항은 응용 프로그램의 배포와, 그리고 익명 사용자의 사용을 의도하지 않는 리소스의 보안과 관련되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-115">When hosting some parts of an application in a storage service, the main considerations are related to deployment of the application and to securing resources that aren't intended to be available to anonymous users.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="01442-116">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="01442-116">Issues and considerations</span></span>

<span data-ttu-id="01442-117">이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="01442-117">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="01442-118">호스트된 저장소 서비스는 사용자가 정적 리소스를 다운로드하기 위해 액세스할 수 있는 HTTP 끝점을 노출해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-118">The hosted storage service must expose an HTTP endpoint that users can access to download the static resources.</span></span> <span data-ttu-id="01442-119">일부 저장소 서비스는 HTTPS도 지원하므로 SSL이 필요한 리소스를 저장소 서비스에 호스트할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-119">Some storage services also support HTTPS, so it's possible to host resources in storage services that require SSL.</span></span>

- <span data-ttu-id="01442-120">최대 성능 및 가용성을 위해, CDN(콘텐츠 전송 네트워크)을 사용하여 전 세계에 있는 여러 데이터 센터에 저장소 컨테이너의 콘텐츠를 캐시하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-120">For maximum performance and availability, consider using a content delivery network (CDN) to cache the contents of the storage container in multiple datacenters around the world.</span></span> <span data-ttu-id="01442-121">그렇지만 CDN 사용 비용을 지불해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-121">However, you'll likely have to pay for using the CDN.</span></span>

- <span data-ttu-id="01442-122">저장소 계정은 데이터 센터에 영향을 줄 수 있는 이벤트에 대한 복원력을 제공하기 위해 기본적으로 지리적으로 복제되는 경우가 많습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-122">Storage accounts are often geo-replicated by default to provide resiliency against events that might affect a datacenter.</span></span> <span data-ttu-id="01442-123">즉, IP 주소는 변경될 수 있지만 URL은 동일하게 유지됩니다.</span><span class="sxs-lookup"><span data-stu-id="01442-123">This means that the IP address might change, but the URL will remain the same.</span></span>

- <span data-ttu-id="01442-124">저장소 계정에 있는 콘텐츠도 있고, 호스트된 계산 인스턴스에 있는 콘텐츠도 있는 경우, 응용 프로그램을 배포하고 업데이트하는 것이 더 어려워집니다.</span><span class="sxs-lookup"><span data-stu-id="01442-124">When some content is located in a storage account and other content is in a hosted compute instance it becomes more challenging to deploy an application and to update it.</span></span> <span data-ttu-id="01442-125">보다 쉬운 관리를 위해 배포를 개별로 수행하고 응용 프로그램 및 콘텐츠의 버전을 관리해야 할 수 있습니다(특히 정적 콘텐츠에 스크립트 파일 또는 UI 구성 요소가 포함되어 있는 경우).</span><span class="sxs-lookup"><span data-stu-id="01442-125">You might have to perform separate deployments, and version the application and content to manage it more easily&mdash;especially when the static content includes script files or UI components.</span></span> <span data-ttu-id="01442-126">그러나 정적 리소스만 업데이트해야 할 경우에는, 응용 프로그램 패키지를 다시 배포하지 않고도 해당 리소스를 저장소 계정에 업로드하기만 하면 됩니다.</span><span class="sxs-lookup"><span data-stu-id="01442-126">However, if only static resources have to be updated, they can simply be uploaded to the storage account without needing to redeploy the application package.</span></span>

- <span data-ttu-id="01442-127">저장소 서비스는 사용자 지정 도메인 이름의 사용을 지원하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-127">Storage services might not support the use of custom domain names.</span></span> <span data-ttu-id="01442-128">이 경우, 리소스가 링크를 포함하는 동적으로 생성된 콘텐츠와는 다른 도메인에 포함되므로, 리소스의 전체 URL을 링크에 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-128">In this case it's necessary to specify the full URL of the resources in links because they'll be in a different domain from the dynamically-generated content containing the links.</span></span>

- <span data-ttu-id="01442-129">저장소 컨테이너는 공용 읽기 액세스용으로 구성해야 하며, 공용 쓰기 액세스용으로 구성하지 않아야 합니다. 사용자가 콘텐츠를 업로드하지 못하게 해야 하기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="01442-129">The storage containers must be configured for public read access, but it's vital to ensure that they aren't configured for public write access to prevent users being able to upload content.</span></span> <span data-ttu-id="01442-130">발렛(Valet) 키 또는 토큰을 사용해서 익명으로 사용할 수 없도록 할 리소스에 대한 액세스를 제어하는 것이 좋습니다(자세한 내용은 [발렛(Valet) 키 패턴](valet-key.md) 참조).</span><span class="sxs-lookup"><span data-stu-id="01442-130">Consider using a valet key or token to control access to resources that shouldn't be available anonymously&mdash;see the [Valet Key pattern](valet-key.md) for more information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="01442-131">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="01442-131">When to use this pattern</span></span>

<span data-ttu-id="01442-132">이 패턴은 다음의 경우에 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-132">This pattern is useful for:</span></span>

- <span data-ttu-id="01442-133">정적 리소스를 일부 포함한 웹 사이트 및 응용 프로그램에 대한 호스팅 비용을 최소화할 경우</span><span class="sxs-lookup"><span data-stu-id="01442-133">Minimizing the hosting cost for websites and applications that contain some static resources.</span></span>

- <span data-ttu-id="01442-134">정적 콘텐츠 및 리소스만으로 구성된 웹 사이트에 대한 호스팅 비용을 최소화할 경우.</span><span class="sxs-lookup"><span data-stu-id="01442-134">Minimizing the hosting cost for websites that consist of only static content and resources.</span></span> <span data-ttu-id="01442-135">호스팅 공급자의 저장소 시스템 기능에 따라, 완전 정적인 웹 사이트를 저장소 계정에 전적으로 호스트할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-135">Depending on the capabilities of the hosting provider’s storage system, it might be possible to entirely host a fully static website in a storage account.</span></span>

- <span data-ttu-id="01442-136">다른 호스팅 환경 또는 온-프레미스 서버에서 실행되는 응용 프로그램에 대한 정적 리소스 및 콘텐츠를 노출할 경우</span><span class="sxs-lookup"><span data-stu-id="01442-136">Exposing static resources and content for applications running in other hosting environments or on-premises servers.</span></span>

- <span data-ttu-id="01442-137">전 세계 여러 데이터 센터에 저장소 계정의 콘텐츠를 캐시하는 콘텐츠 전송 네트워크를 사용하여 둘 이상의 지리적 영역에 콘텐츠를 배치할 경우</span><span class="sxs-lookup"><span data-stu-id="01442-137">Locating content in more than one geographical area using a content delivery network that caches the contents of the storage account in multiple datacenters around the world.</span></span>

- <span data-ttu-id="01442-138">비용 및 대역폭 사용량을 모니터링할 경우.</span><span class="sxs-lookup"><span data-stu-id="01442-138">Monitoring costs and bandwidth usage.</span></span> <span data-ttu-id="01442-139">정적 콘텐츠의 일부나 전체에 대해 별도 저장소 계정을 사용하면 호스팅 및 런타임 비용에서 콘텐츠 비용을 보다 쉽게 분리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-139">Using a separate storage account for some or all of the static content allows the costs to be more easily separated from hosting and runtime costs.</span></span>

<span data-ttu-id="01442-140">이 패턴은 다음과 같은 상황에서는 유용하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-140">This pattern might not be useful in the following situations:</span></span>

- <span data-ttu-id="01442-141">응용 프로그램은 정적 콘텐츠를 클라이언트에 배달하기 전에 몇 가지 처리를 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-141">The application needs to perform some processing on the static content before delivering it to the client.</span></span> <span data-ttu-id="01442-142">예를 들어, 문서에 타임스탬프를 추가해야 할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-142">For example, it might be necessary to add a timestamp to a document.</span></span>

- <span data-ttu-id="01442-143">정적 콘텐츠의 양이 매우 작습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-143">The volume of static content is very small.</span></span> <span data-ttu-id="01442-144">이 콘텐츠를 별도 저장소에서 가져올 때 발생하는 오버헤드가 계산 리소스에서 이 콘텐츠를 별도로 관리할 때 파생되는 비용 혜택보다 클 우려가 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-144">The overhead of retrieving this content from separate storage can outweigh the cost benefit of separating it out from the compute resource.</span></span>

## <a name="example"></a><span data-ttu-id="01442-145">예</span><span class="sxs-lookup"><span data-stu-id="01442-145">Example</span></span>

<span data-ttu-id="01442-146">Azure Blob 저장소에 있는 정적 콘텐츠는 웹 브라우저에서 직접 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-146">Static content located in Azure Blob storage can be accessed directly by a web browser.</span></span> <span data-ttu-id="01442-147">Azure는 클라이언트에 공개적으로 노출될 수 있는 저장소에 대해 HTTP 기반 인터페이스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-147">Azure provides an HTTP-based interface over storage that can be publicly exposed to clients.</span></span> <span data-ttu-id="01442-148">예를 들어, Azure Blob 저장소 컨테이너의 콘텐츠는 다음 형식의 URL을 사용하여 노출됩니다.</span><span class="sxs-lookup"><span data-stu-id="01442-148">For example, content in an Azure Blob storage container is exposed using a URL with the following form:</span></span>

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


<span data-ttu-id="01442-149">콘텐츠를 업로드하는 경우, 파일 및 문서를 보유할 하나 이상의 Blob 컨테이너를 만들어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-149">When uploading the content it's necessary to create one or more blob containers to hold the files and documents.</span></span> <span data-ttu-id="01442-150">새 컨테이너에 대한 기본 사용 권한은 개인(Private)이며, 클라이언트가 콘텐츠에 액세스하도록 하려면 이 권한을 공용(Public)으로 변경해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-150">Note that the default permission for a new container is Private, and you must change this to Public to allow clients to access the contents.</span></span> <span data-ttu-id="01442-151">콘텐츠를 익명 액세스로부터 보호해야 하는 경우 사용자가 리소스 다운로드를 위해 유효한 토큰을 제공해야 하도록 [발렛(Valet) 키 패턴](valet-key.md)을 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-151">If it's necessary to protect the content from anonymous access, you can implement the [Valet Key pattern](valet-key.md) so users must present a valid token to download the resources.</span></span>

> <span data-ttu-id="01442-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)(Blob 서비스 개념)에서 Blob 저장소 및 이 저장소를 액세스하고 사용하는 방법을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="01442-152">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx) has information about blob storage, and the ways that you can access and use it.</span></span>

<span data-ttu-id="01442-153">각 페이지의 링크는 리소스의 URL을 지정하며, 클라이언트는 저장소 서비스에서 리소스에 직접 액세스합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-153">The links in each page will specify the URL of the resource and the client will access it directly from the storage service.</span></span> <span data-ttu-id="01442-154">다음 그림에서는 저장소 서비스에서 직접 응용 프로그램의 정적 부분을 전달하는 모습을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="01442-154">The figure illustrates delivering static parts of an application directly from a storage service.</span></span>

![그림 1 - 저장소 서비스에서 직접 응용 프로그램의 정적 부분 전달](./_images/static-content-hosting-pattern.png)


<span data-ttu-id="01442-156">클라이언트에 전달된 페이지의 링크는 Blob 컨테이너 및 리소스의 전체 URL을 반드시 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-156">The links in the pages delivered to the client must specify the full URL of the blob container and resource.</span></span> <span data-ttu-id="01442-157">예를 들어, 공용 컨테이너에 있는 이미지에 대한 링크를 포함하는 페이지에는 다음 HTML이 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-157">For example, a page that contains a link to an image in a public container might contain the following HTML.</span></span>

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> <span data-ttu-id="01442-158">리소스가 발렛 키(예: Azure 공유 액세스 서명)를 사용하여 보호되는 경우, 이 서명이 링크의 URL에 포함되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-158">If the resources are protected by using a valet key, such as an Azure shared access signature, this signature must be included in the URLs in the links.</span></span>

<span data-ttu-id="01442-159">정적 리소스에 대해 외부 저장소를 사용하는 모습을 보여 주는 StaticContentHosting이라는 솔루션은 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting)에서 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-159">A solution named StaticContentHosting that demonstrates using external storage for static resources is available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span> <span data-ttu-id="01442-160">StaticContentHosting.Cloud 프로젝트에는 저장소 계정을 지정하는 구성 파일과, 정적 콘텐츠를 보유하는 컨테이너가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-160">The StaticContentHosting.Cloud project contains configuration files that specify the storage account and container that holds the static content.</span></span>

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

<span data-ttu-id="01442-161">StaticContentHosting.Web 프로젝트의 Settings.cs 파일에 있는 `Settings` 클래스는 이러한 값을 추출하고 클라우드 저장소 계정 컨테이너 URL을 포함하는 문자열 값을 작성하기 위한 메서드를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-161">The `Settings` class in the file Settings.cs of the StaticContentHosting.Web project contains methods to extract these values and build a string value containing the cloud storage account container URL.</span></span>

```csharp
public class Settings
{
  public static string StaticContentStorageConnectionString {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue(
                              "StaticContent.StorageConnectionString");
    }
  }

  public static string StaticContentContainer
  {
    get
    {
      return RoleEnvironment.GetConfigurationSettingValue("StaticContent.Container");
    }
  }

  public static string StaticContentBaseUrl
  {
    get
    {
      var account = CloudStorageAccount.Parse(StaticContentStorageConnectionString);

      return string.Format("{0}/{1}", account.BlobEndpoint.ToString().TrimEnd('/'),
                                      StaticContentContainer.TrimStart('/'));
    }
  }
}
```

<span data-ttu-id="01442-162">StaticContentUrlHtmlHelper.cs 파일의 `StaticContentUrlHtmlHelper` 클래스는 전달된 URL이 ASP.NET 루트 경로 문자(~)로 시작하는 경우 클라우드 저장소 계정에 대한 경로를 포함한 URL을 생성하는 `StaticContentUrl`이라는 메서드를 노출합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-162">The `StaticContentUrlHtmlHelper` class in the file StaticContentUrlHtmlHelper.cs exposes a method named `StaticContentUrl` that generates a URL containing the path to the cloud storage account if the URL passed to it starts with the ASP.NET root path character (~).</span></span>

```csharp
public static class StaticContentUrlHtmlHelper
{
  public static string StaticContentUrl(this HtmlHelper helper, string contentPath)
  {
    if (contentPath.StartsWith("~"))
    {
      contentPath = contentPath.Substring(1);
    }

    contentPath = string.Format("{0}/{1}", Settings.StaticContentBaseUrl.TrimEnd('/'),
                                contentPath.TrimStart('/'));

    var url = new UrlHelper(helper.ViewContext.RequestContext);

    return url.Content(contentPath);
  }
}
```

<span data-ttu-id="01442-163">Views\Home 폴더의 Index.cshtml에는 `StaticContentUrl` 메서드를 사용하여 해당 `src` 특성에 대한 URL을 만드는 이미지 요소가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-163">The file Index.cshtml in the Views\Home folder contains an image element that uses the `StaticContentUrl` method to create the URL for its `src` attribute.</span></span>

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="01442-164">관련 패턴 및 지침</span><span class="sxs-lookup"><span data-stu-id="01442-164">Related patterns and guidance</span></span>

- <span data-ttu-id="01442-165">이 패턴의 사용을 보여주는 예제는 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting)에서 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="01442-165">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting).</span></span>
- <span data-ttu-id="01442-166">[발레 키 패턴](valet-key.md).</span><span class="sxs-lookup"><span data-stu-id="01442-166">[Valet Key pattern](valet-key.md).</span></span> <span data-ttu-id="01442-167">대상 리소스를 익명 사용자가 사용할 수 없게 하려는 경우 정적 콘텐츠를 보유하는 저장소에 보안 조치를 구현해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-167">If the target resources aren't supposed to be available to anonymous users it's necessary to implement security over the store that holds the static content.</span></span> <span data-ttu-id="01442-168">클라이언트에 특정 리소스 또는 서비스(예: 클라우드 호스티드 저장소 서비스)에 대한 제한된 직접 액세스를 제공하는 토큰 또는 키를 사용하는 방법을 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="01442-168">Describes how to use a token or key that provides clients with restricted direct access to a specific resource or service such as a cloud-hosted storage service.</span></span>
- <span data-ttu-id="01442-169">Infosys 블로그의 [An efficient way of deploying a static web site on Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html)(Azure에서 정적 웹 사이트를 배포하는 효율적인 방법)</span><span class="sxs-lookup"><span data-stu-id="01442-169">[An efficient way of deploying a static web site on Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html) on the Infosys blog.</span></span>
- <span data-ttu-id="01442-170">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)(Blob 서비스 개념)</span><span class="sxs-lookup"><span data-stu-id="01442-170">[Blob Service Concepts](https://msdn.microsoft.com/library/azure/dd179376.aspx)</span></span>
