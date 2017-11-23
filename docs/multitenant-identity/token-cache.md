---
title: Cache acess tokens in a multitenant application
description: Caching access tokens used for invoking a backend Web API
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 02/16/2016
ms.author: pnp

pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
---
# 캐시 액세스 토큰

[![GitHub](../_images/github.png) 샘플 코드][sample application]

OAuth 액세스 토큰을 받는 것은 상대적으로 비싼데, 토큰 끝점으로 HTTP 요청을 요구하기 때문입니다. 그러므로 가능하면 토큰을 캐시하는 것이 좋습니다. [Azure AD 인증 라이브러리][ADAL] (ADAL)는 새로 고침 토큰을 포함하여 Azure AD에서 받은 토큰을 자동으로 캐시합니다.

ADAL은 기본 토큰 캐시 구현을 제공합니다. 그러나, 이 토큰 캐시는 네이티브 클라이언트 앱을 위한 것으로, 웹 앱에는 적절하지 *않습니다*:

•	고정 인스턴스이며, 스레드로부터 안전하지 않습니다.

•	모든 사용자들로부터 받은 토큰이 같은 디렉터리로 가기 때문에, 다수의 사용자들로 확장될 수 없습니다.

•	팜의 웹 서버들에서 공유할 수 없습니다.


대신, ADAL `TokenCache` 클래스에서 파생되었지만 서버 환경에 적절한 사용자 지정 토큰 캐시를 구현해야 합니다. 이 토큰 캐시는 다양한 사용자들의 토큰들에서 안정적 수준의 격리를 제공합니다.

`TokenCache` 클래스는 발급자, 리소스, 클라이언트 ID, 사용자에 의해 인덱싱된 토큰 사전을 저장합니다. 사용자 지정 토큰 캐시는 이 사전을 Redis 캐시와 같은 백업 저장소에 써야 합니다.

Tailspin Surveys 응용 프로그램에서, `DistributedTokenCache`클래스는 토큰 캐시를 구현합니다. 이 구현에서는 ASP.NET Core 1.0의 [IDistributedCache][distributed-cache] 개념을 사용합니다. 이와 같이, 어떠한 `IDistributedCache` i구현도 백업 저장소로 사용될 수 있습니다.

•	기본값으로, Surveys 앱은 Redis 캐시를 사용합니다.

•	단일 인스턴스 웹 서버인 경우, ASP.NET Core 1.0 [메모리 내 캐시][in-memory-cache]를 사용할 수 있습니다.  (개발 중 앱을 로컬로 실행할 때 이 방법도 좋은 선택입니다.)

> [!참고]
> 현재 .NET Core에서는 Redis 캐시가 지원되지 않습니다.
> 
> 

`DistributedTokenCache` 는 백업 저장소에 키/값의 쌍으로 캐시 데이터를 저장합니다. 키는 사용자 ID 더하기 클라이언트 ID이므로, 백업 저장소는 사용자/클라이언트의 고유한 조합에 대해서 별도의 캐시 데이터를 저장합니다.

![Token cache](./images/token-cache.png)

백업 저장소는 사용자에 의해 분할됩니다. HTTP 요청이 오면, 그 사용자에 대한 토큰이 백업 저장소에서 읽히고 `TokenCache` 사전으로 로드됩니다. Redis가 백업 저장소로 사용될 경우, 서버 팜에 있는 모든 서버 인스턴스는 같은 캐시를 읽기/쓰기 하고, 이런 접근 방식은 다수의 사용자들로 확장될 수 있습니다.

## 캐시된 토큰 암호화
토큰은 사용자 리소스에 대한 액세스를 허용하기 때문에 중요한 데이터입니다. (또한, 사용자 암호와 달리, 토큰 해시를 저장할 수 없습니다.) 그러므로, 토큰이 손상되지 않게 보호하는 것이 중요합니다. Redis에 백업된 캐시는 암호에 의해 보호되지만, 누군가 암호를 확보할 경우 캐시된 액세스 토큰을 모두 가져갈 수 있습니다. 이런 이유로, `DistributedTokenCache` 는 백업 저장소에 쓴 것을 모두 암호화합니다. 암호화는 ASP.NET Core 1.0 [데이터 보호][data-protection] API를 사용하여 이루어집니다.

> [!참고]
> 사용자가 Azure 웹 사이트에 배포할 경우, 암호화 키가 네트워크 저장소에 백업되고 모든 컴퓨터에서 동기화됩니다([키 관리][key-management])를 참조하세요). 기본값으로, 키는 Azure 웹 사이트에서 실행될 때 암호화되지 않지만, [X.509 인증서를 사용한 암호화][x509-cert-encryption]를 할 수 있습니다.
> 
> 

## DistributedTokenCache 구현
[DistributedTokenCache][DistributedTokenCache] 클래스는 ADAL  [TokenCache][tokencache-class] 클래스에서 파생됩니다.

생성자에서, `DistributedTokenCache` 클래스는 현 사용자용 키를 만들고 백업 저장소의 캐시를 로드합니다:

```csharp
public DistributedTokenCache(
    ClaimsPrincipal claimsPrincipal,
    IDistributedCache distributedCache,
    ILoggerFactory loggerFactory,
    IDataProtectionProvider dataProtectionProvider)
    : base()
{
    _claimsPrincipal = claimsPrincipal;
    _cacheKey = BuildCacheKey(_claimsPrincipal);
    _distributedCache = distributedCache;
    _logger = loggerFactory.CreateLogger<DistributedTokenCache>();
    _protector = dataProtectionProvider.CreateProtector(typeof(DistributedTokenCache).FullName);
    AfterAccess = AfterAccessNotification;
    LoadFromCache();
}
```

키는 사용자 ID와 클라이언트 ID를 연결하여 생성됩니다. 두 ID는 사용자의 `ClaimsPrincipal`에서 찾은 클레임에서 가져온 것입니다.

```csharp
private static string BuildCacheKey(ClaimsPrincipal claimsPrincipal)
{
    string clientId = claimsPrincipal.FindFirstValue("aud", true);
    return string.Format(
        "UserId:{0}::ClientId:{1}",
        claimsPrincipal.GetObjectIdentifierValue(),
        clientId);
}
```

캐시 데이터를 로드하기 위해서는, 백업 저장소에서 직렬화된 블롭을 읽은 다음 `TokenCache.Deserialize`를 호출하여 블롭을 캐시 데이터로 변환합니다.

```csharp
private void LoadFromCache()
{
    byte[] cacheData = _distributedCache.Get(_cacheKey);
    if (cacheData != null)
    {
        this.Deserialize(_protector.Unprotect(cacheData));
    }
}
```

ADAL는 캐시를 액세스할 때마다 `AfterAccess` 이벤트를 발생시킵니다. 캐시 데이터가 변경된 경우, `HasStateChanged` 속성이 true가 됩니다. 그런 경우, 변경이 반영되도록 백업 저장소를 업데이트한 다음 `HasStateChanged`를 false로 설정합니다.

```csharp
public void AfterAccessNotification(TokenCacheNotificationArgs args)
{
    if (this.HasStateChanged)
    {
        try
        {
            if (this.Count > 0)
            {
                _distributedCache.Set(_cacheKey, _protector.Protect(this.Serialize()));
            }
            else
            {
                // There are no tokens for this user/client, so remove the item from the cache.
                _distributedCache.Remove(_cacheKey);
            }
            this.HasStateChanged = false;
        }
        catch (Exception exp)
        {
            _logger.WriteToCacheFailed(exp);
            throw;
        }
    }
}
```

TokenCache는 두 개의 다른 이벤트를 전송합니다:

* `BeforeWrite`. ADAL이 캐시에 쓰기 직전에 호출됩니다. 동시성 전략 구현에 사용할 수 있습니다.

* `BeforeAccess`. ADAL이 캐시에서 읽기 직전에 호출됩니다. 여기서 캐시를 다시 로드하여 최종 버전을 얻을 수 있습니다.

우리는 이 두 이벤트를 처리하지 않기로 결정했습니다.

•	동시성을 위해서 마지막 쓰기를 합니다. 좋아요. 이제 토큰이 사용자 + 클라이언트별로 따로 저장되기 때문에, 사용자 한 명이 두 개의 로그인 세션을 동시에 연결한 경우에만 충돌이 일어납니다.

•	읽기의 경우, 요청이 있을 때마다 캐시를 로드합니다. 요청은 일시적입니다. 그 때 캐시가 수정된 경우, 다음 요청은 새로운 값을 선택할 것입니다. 


[**다음**][client-assertion]

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[client-assertion]: ./client-assertion.md
[data-protection]: https://docs.asp.net/en/latest/security/data-protection/index.html
[distributed-cache]: https://docs.microsoft.com/aspnet/core/performance/caching/distributed
[DistributedTokenCache]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.TokenStorage/DistributedTokenCache.cs
[key-management]: https://docs.asp.net/en/latest/security/data-protection/configuration/default-settings.html
[in-memory-cache]: https://docs.microsoft.com/aspnet/core/performance/caching/memory
[tokencache-class]: https://msdn.microsoft.com/library/azure/microsoft.identitymodel.clients.activedirectory.tokencache.aspx
[x509-cert-encryption]: https://docs.asp.net/en/latest/security/data-protection/implementation/key-encryption-at-rest.html#x-509-certificate

[sample application]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps
