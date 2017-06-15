---
title: Static Content Hosting
description: Deploy static content to a cloud-based storage service that can deliver them directly to the client.
keywords: design pattern
author: dragon119
ms.service: guidance
ms.topic: article
ms.author: pnp
ms.date: 03/24/2017

pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: [data-management, design-implementation, performance-scalability]
---

# 정적 콘텐츠 호스팅

[!INCLUDE [header](../_includes/header.md)]

클라이언트에 직접 전달할 수 있는 클라우드 기반 저장소 서비스에 정적 콘텐츠를 배포합니다. 이렇게 하면 잠재적으로 값 비싼 계산 인스턴스의 필요성을 줄일 수 있습니다.

## 컨텍스트와 문제점

일반적으로 웹 응용 프로그램은 정적 콘텐츠의 몇 가지 요소를 포함합니다. 이 정적 콘텐츠는 HTML 페이지와 기타 클라이언트에 이용 가능한 이미지, 문서 등의 자원을 HTML 페이지의 일부로(인라인 이미지, 스타일시트, 클라이언트 쪽 JavaScript 파일 등), 또는 별도의 다운로드로(PDF 문서 등) 포함할 수 있습니다.

웹 서버가 효율적인 동적 페이지 코드 실행과 출력 캐싱을 통해서 요청을 최적화하도록 잘 맞춰졌지만, 여전히 정적 콘텐츠를 다운로드하는 요청을 처리해야 합니다. 이는 종종 유효하게 이용될 수 있는 처리 주기를 사용합니다.

## 솔루션

대부분의 클라우드 호스팅 환경에서는, 저장소 서비스에서 응용 프로그램의 리소스와 정적 페이지를 찾아 계산 인스턴스의 필요성을 최소화하는 것이(예: 크기가 작은 인스턴스 또는 적은 수의 인스턴스 사용) 가능합니다. 일반적으로 클라우드 호스트된 저장소에 대한 비용은 계산 인스턴스보다 훨씬 적습니다.

저장소 서비스에서 응용 프로그램의 일부를 호스팅할 때, 주요 고려사항은 응용 프로그램 배포와, 익명 사용자가 이용할 수 없는 리소스의 보안과 관련이 있습니다.

## 문제점 및 고려사항

이 패턴을 구현하는 방법을 결정할 때 다음 사항을 고려해야 합니다:

- 호스트된 저장소 서비스는 사용자가 정적 리소스를 다운로드하기 위해 액세스할 수 있는 HTTP 끝점을 표시해야 합니다. 일부 저장소 서비스 또한 HTTPS를 지원하기 때문에, SSL을 요구하는 저장소 서비스에 리소스를 호스트하는 것이 가능합니다.

- 최대 성능과 가용성을 위해서, 전 세계 다중 데이터센터에 저장소 컨테이너의 콘텐츠를 캐시하는 데 콘텐츠 전송 네트워크(CDN)의 사용을 고려합니다. 그러나, CDN 사용 비용을 지불해야 할 수 있습니다.

- 일반적으로 저장소 계정은 데이터센터에 영향을 줄 수 있는 이벤트에 대한 복원력을 제공하기 위해서 기본값으로 지역이 복제됩니다. 이는 IP 주소는 변경될 수 있지만 URL은 계속 동일하게 유지된다는 뜻입니다.

- 일부 콘텐체가 저장소 계정에 있고 다른 콘텐츠가 호스트된 계산 인스턴스에 있으면, 응용 프로그램을 배포하고 업데이트하는 것이 더 어려워집니다. 좀 더 쉽게 관리하려면 응용 프로그램 및 콘텐츠에 대한 별도의 배포와 버전을 수행해야할 수 있습니다 - 특히 정적 콘텐츠가 스크립트 파일이나 UI 구성요소를 포함할 때. 그러나, 정적 리소스는 업데이트되어야할 경우에만 응용 프로그램 패키지를 재배포할 필요 없이 저장소 계정에 업로드될 수 있습니다.

- 저장소 서비스는 사용자 지정 도메인 이름 사용을 지원하지 않을 수 있습니다. 이 경우, 리소스가 링크를 포함한 동적 생성된 콘텐츠와 다른 도메인에 있을 것이기 때문에 리소스의 전체 URL을 링크에 지정할 필요가 있습니다.

- 저장소 컨테이너는 공용 읽기 권한으로 구성되어야 하는데, 사용자가 콘텐츠를 업로드할 수 없게 공용 쓰기 권한으로 구성되지 않도록 하는 것이 중요합니다. 익명으로 사용해서는 안 되는 리소스에 대한 액세스를 제어할 발렛 키 또는 토큰의 사용을 고려합니다 — 자세한 정보는 [발렛 키 패턴](valet-key.md) 을 참조합니다.

## 이 패턴을 사용하는 경우

이 방식은 다음과 같은 경우에 유용합니다:

- 정적 리소스를 포함한 웹사이트와 응용 프로그램의 호스팅 비용 최소화.

- 정적 콘텐츠와 리소스만으로 구성된 웹사이트의 호스팅 비용 최소화. 호스팅 공급자의 저장 시스템 역량에 따라, 저장소 계정에서 완전히 정적인 웹사이트를 호스트하는 것이 가능할 수 있습니다. 

- 다른 호스팅 환경 또는 사내 서버에서 실행되는 응용 프로그램의 정적 리소스와 콘텐츠 표시. 

- 전 세계 다중 데이터센터에 저장소 계정의 콘텐츠를 캐시하는 콘텐츠 전송 네트워크를 사용하여 한 곳 이상의 지역에서 콘텐츠를 찾는 경우.

- 비용 및 대역 사용 모니터링. 일부 또는 모든 정적 콘텐츠에 대한 별도의 저장소 계정 사용으로 그 비용이 호스팅 및 런타임 비용과 쉽게 구분될 수 있습니다.

이 패턴은 다음 상황에서 유용하지 않을 수 있습니다:

- 응용 프로그램은 일부 처리를 클라이언트로 전달하기 전에 정적 콘텐츠에서 실행할 필요가 있는 경우. 예를 들면, 문서에 타임스탬프 추가가 필요할 수 있습니다.

- 정적 콘텐츠의 볼륨이 매우 작은 경우. 별도의 저장소에서 이 콘텐츠를 검색하는 데 대한 오버헤드가 계산 리소스에서 그것을 구분하는 비용 혜택보다 클 수 있습니다.

## 예

Azure Blob 저장소에 있는 정적 콘텐츠는 웹 브라우저에 의해 직접 액세스될 수 있습니다. Azure는 클라이언트에 공개적으로 노출될 수 있는 저장소에 대해서 HTTP 기반 인터페이스를 제공합니다. 예를 들면, Azure Blob 저장소 컨테이너의 콘텐츠는 아래와 같은 형태의 URL로 표시됩니다.

`http://[ storage-account-name ].blob.core.windows.net/[ container-name ]/[ file-name ]`


콘텐츠를 업로드할 때, 파일과 문서를 보관할 블롭 컨테이너를 하나 이상 만들 필요가 있습니다. 새 컨테이너에 대한 기본 사용 권한은 Private이고, 고객이 콘텐츠을 액세스할 수 있게 하려면 이를 Public으로 변경해야 하는 점에 주의하세요. 익명의 액세스에서 콘텐츠를 보호할 필요가 있을 경우, [발렛 키 패턴](valet-key.md) 을 구현할 수 있어야 하고, 이에 사용자는 리소스를 다운로드하기 위해 유효한 토큰을 제시해야 합니다.

> [Blob 서비스 개념](https://msdn.microsoft.com/library/azure/dd179376.aspx) 은 블롭 저장소에 관한 정보와, 그것을 액세스해서 사용할 수 있는 방법을 갖고 있습니다.

각 페이지의 링크는 저장소의 URL을 지정하고, 클라이언트는 이를 저장소 서비스에서 직접 액세스할 것입니다. 그림은 저장소 서비스에서 응용 프로그램의 정적인 부분을 직접 전달하는 것을 설명합니다.

![Figure 1 - Delivering static parts of an application directly from a storage service](./_images/static-content-hosting-pattern.png)


클라이언트에 전달된 페이지의 링크는 블롭 컨테이너와 리소스의 URL 전체를 지정해야 합니다. 예를 들면, 공용 컨테이너에 있는 이미지에 대한 링크를 포함한 페이지는 다음의 HTML을 포함할 수 있습니다.

```html
<img src="http://mystorageaccount.blob.core.windows.net/myresources/image1.png"
     alt="My image" />
```

> 리소스가 Azure 공유 액세스 서명과 같은 발렛 키로 보호된 경우, 이 서명은 링크의 URL에 포함되어야 합니다.

정적 리소스의 외부 저장소 사용을 보여주는 StaticContentHosting이라는 솔루션은 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting)에서 이용 가능합니다. StaticContentHosting.Cloud 프로젝트는 정적 콘텐츠를 보관한 저장소 계정과 컨테이너를 지정하는 구성 파일을 포함합니다.

```xml
<Setting name="StaticContent.StorageConnectionString"
         value="UseDevelopmentStorage=true" />
<Setting name="StaticContent.Container" value="static-content" />
```

StaticContentHosting.Web 프로젝트의 Settings.cs 파일에 있는 `Settings` 클래스는 이 값들을 추출하고 클라우드 저장소 계정 컨테이너 URL을 포함한 문자열 값을 빌드할 메서드를 포함합니다. 

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

전달된 URL이 ASP.NET 루트 경로 문자(~)로 시작할 경우, StaticContentUrlHtmlHelper.cs 파일에 있는 `StaticContentUrlHtmlHelper` 클래스는 클라우드 저장소 계정에 대한 경로를 포함한 URL을 생성하는 `StaticContentUrl` 이라는 메서드를 표시합니다.

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

Views\Home 폴더의 Index.cshtml 파일은 `src` 특성의 URL을 만들기 위해 `StaticContentUrl` 메서드를 사용하는 이미지 요소를 포함합니다.

```html
<img src="@Html.StaticContentUrl("~/media/orderedList1.png")" alt="Test Image" />
```

## 관련 패턴 및 지침

- 이 패턴을 보여주는 샘플은 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/static-content-hosting)에서 이용 가능합니다.
- [발렛 키 패턴](valet-key.md). 대상 리소스를 익명 사용자가 이용하면 안 될 경우, 정적 콘텐츠를 보관하는 저장소에 대한 보안을 구현하는 것이 필요합니다.  클라우드 호스트된 저장소 서비스와 같은 특정 리소스나 서비스에 대한 제한적 직접 액세스를 클라이언트에 제공하는 토큰 또는 키를 사용하는 방법에 대해 설명합니다.
- Infosys 블로그의 [An efficient way of deploying a static web site on Azure](http://www.infosysblogs.com/microsoft/2010/06/an_efficient_way_of_deploying.html).
- [Blob 서비스 개념](https://msdn.microsoft.com/library/azure/dd179376.aspx)
