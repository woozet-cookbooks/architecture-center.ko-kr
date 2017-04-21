---
title: Federate with a customer's AD FS
description: How to federate with a customer's AD FS in a multitenant application
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 06/02/2016
ms.author: pnp

pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: token-cache
pnp.series.next: client-assertion
---
# 고객 AD FS와의 연동

이 문서는 다중 테넌트 SaaS 응용 프로그램이 액티브 디렉터리 연동 서비스(Active Directory Federation Service(AD FS))를 통해서 어떻게 인증을 지원하여 고객의 AD FS와 연동할 수 있는지에 대해 설명합니다.

## 개요
Azure Active Directory(Azure AD)는 Office365 및 Dynamics CRM Online 고객뿐만 아니라 Azure AD 고객들이 로그인하기 쉽습니다. 하지만 회사 인트라넷에서 사내 Active Directory를 사용하는 고객은 어떨까요?

한 가지 옵션은 고객들이 [Azure AD Connect]를 사용해서 사내 AD와 Azure AD를 동기화하는 것입니다. 그러나, 어떤 고객은 회사의 IT 정책이나 다른 이유 때문에 이 방법을 사용하지 못할 수 있습니다. 그런 경우, 또 다른 옵션은 Active Directory Federation Services (AD FS)를 통해서 연동하는 것입니다.

이 시나리오를 사용하기 위해서:

* 고객은 인터넷에 연결된 AD FS 팜을 갖고 있어야 합니다.
* SaaS 공급자는 자체 AD FS 팜을 배포합니다.
* 고객과 SaaS 공급자는 [연동 신뢰(federation trust)]를 설정합니다. 이 작업은 수동식 프로세스입니다.

신뢰 관계에는 3가지 주요 역할이 있습니다:

* 고객의 AD FS는 [계정 파트너]로서 고객 AD의 사용자를 인증하고 사용자 클레임으로 보안 토큰을 만듭니다.
* SaaS 공급자의 AD FS는 [리소스 파트너]이며, 계정 파트너를 신뢰하고 사용자 클레임을 수신합니다.
* SaaS 공급자의 AD FS에서 응용 프로그램은 신뢰 당사자(RP) 자격으로 구성됩니다. 
  
  ![Federation trust](./images/federation-trust.png)

> [!참고]
> 우리는 이 문서에서 응용 프로그램이 인증 프로토콜로 OpenId connect를 사용한다고 가정합니다.  또 다른 옵션은 WS-Federation을 사용하는 것입니다. 
> 
> For OpenID Connect, the SaaS provider must use AD FS 4.0 running in Windows Server 2016, which is currently in Technical Preview. AD FS 3.0 does not support OpenID Connect.
> 
> ASP.NET Core 1.0은 WS-Federation에 대해서 기본으로 제공되는 지원을 포함하지 않습니다.
> 
> 

ASP.NET 4와 WS-Federation을 사용한 예를 보려면, [active-directory-dotnet-webapp-wsfederation 샘플][active-directory-dotnet-webapp-wsfederation]을 참조하세요.

## 인증 흐름
1. 사용자가 "로그인"을 클릭하면 응용 프로그램은 SaaS 공급자의 AD FS의 OpenId Connect 끝점으로 리디렉션합니다. 
2. 사용자는 자기 조직에서 쓰는 사용자 이름을 입력합니다("`alice@corp.contoso.com`"). AD FS는 홈 영역 검색을 이용하여, 사용자가 자격 증명을 입력한 고객의 AD FS로 리디렉션합니다.
3. 고객의 AD FS는 WF-Federation (또는 SAML)을 사용하여 사용자 클레임을 SaaS 공급자의 AD FS로 전송합니다.
4. OpenID Connect를 사용한, AD FS에서 앱으로의 클레임 흐름. 여기서는 WS-Federation에서 전달된 프로토콜의 전환이 필요합니다.

## 제한
쓰기 수행 시, 응용 프로그램은 다음 표에 열거된 것과 같이, OpenID인 id_token 안에 제한된 클레임 세트를 담아 수신합니다. AD FS 4.0는 아직 미리 보기 상태이므로, 이 세트는 변경될 수 있습니다. 현재 추가 클레임을 정의하는 것은 불가능합니다:

| 클레임 | 설명 |
| --- | --- |
| aud |대상. 클레임이 발급된 대상인 응용 프로그램. |
| authenticationinstant |[인증 인스턴트]. 인증이 발생한 시간. |
| c_hash |코드 해시 값. 이 값은 토큰 콘텐츠의 해시입니다. |
| exp |[만료 시간]. 시간이 지나 토큰이 더 이상 수락되지 않는 시간. |
| iat |발급된 때. 토큰이 발급된 시간. |
| iss |발급자. 이 클레임 값은 항상 리소스 파트너의 AD FS입니다. |
| name |사용자 이름. 예: `john@corp.fabrikam.com`. |
| nameidentifier |[이름 식별자]. 토큰이 발급된 주체의 이름에 대한 식별자. |
| nonce |임시 세션. 재생 공격을 예방하기 위해서 AD FS에 의해 생성된 고유 값. |
| upn |사용자 계정 이름(UPN). 예: john@corp.fabrikam.com |
| pwd_exp |암호 만료 기간. 사용자 암호 또는 PIN 같은 유사한 인증 암호가 만료될 때까지의 기간(초 단위). |

> [!참고]
> "iss" 클레임은 파트너의 AD FS를 포함합니다(보통 이 클레임은 SaaS 공급자를 발급자로 식별할 것입니다). 이 클레임은 고객의 AD FS를 식별하지 않습니다. UPN의 일부로서 고객의 도메인을 발견할 수 있습니다.
> 
> 

문서의 남은 부분에서 RP(앱)과 계정 파트너(고객) 간의 신뢰 관계를 어떻게 설정하는지에 대해 설명합니다.

## AD FS 배포
SaaS 공급자는 사내에 또는 AzureVM에 AD FS를 배포할 수 있습니다. 보안과 가용성을 위해서 다음 지침이 중요합니다:

* AD FS 서비스를 최대로 이용할 수 있으려면 최소 AD FS 서버 2대와 AD FS 프록시 서버 2대를 배포합니다.
* 도메인 컨트롤러와 AD FS 서버는 절대 인터넷에 직접 노출되면 안 되고, 직접 액세스할 수 있는 가상 네트워크에 있어야 합니다. 
* 웹 응용 프로그램 프록시는(앞서 AD FS 프록시로 언급) 인터넷에 AD FS 서버를 게시하는 데 사용되어야 합니다.

Azure에서 유사한 토콜로지를 설정하기 위해서는 가상 네트워크, NSG, azure VM, 가용성 세트의 사용이 필요합니다. 더 자세한 정보는, [가상 컴퓨터에 Windows Server Active Directory 배포를 위한 지침][active-directory-on-azure]을 참조하세요.

## AD FS와 연동하여 OpenID Connect 인증 구성
SaaS 공급자는 응용 프로그램과 AD FS 사이에 OpenID Connect를 사용해야 합니다. 그렇게 하려면, AD FS에 응용 프로그램 그룹을 추가합니다. "AD FS에서 OpenId Connect 표지에 대한 웹 앱 설정하기"의 [블로그 포스트]에서 자세한 지침을 볼 수 있습니다. 

다음, OpenID Connect 미들웨어 구성. 메타데이터 끝점은 `https://domain/adfs/.well-known/openid-configuration`이고, 도메인은 SaaS 공급자의 AD FS 도메인입니다.

일반적으로 이 끝점과 다른 OpenID Connect 끝점을 조합할 수 있습니다(AAD 등). 사용자를 정확한 인증 끝점으로 보내기 위해서는, 서로 다른 두 개의 로드인 단추가 필요하거나 아니면 두 끝점을 구별할 다른 방법이 필요합니다.

## AD FS 리소스 파트너 구성
SaaS 공급자는 ADFS를 통해 연결하고자 하는 고객을 위해서 다음을 수행해야 합니다:

1. 클레임 고객 공급자 트러스트를 추가합니다.
2. 클레임 규칙을 추가합니다.
3. 홈 영역 검색을 사용합니다.

더 자세한 단계는 다음과 같습니다. 

### 클레임 고객 공급자 트러스트 추가
1. Server Manager에서, **도구**를 클릭하고 **AD FS 관리**를 선택합니다.
2. 콘솔 트리에서, **AD FS**의 **클레임 공급자 트러스트**에서 마우스 오른쪽 단추를 클릭합니다. **클레임 공급자 트러스트 추가**를 선택합니다.
3. **Start**를 클릭하여 마법사를 시작합니다.
4. "온라인 또는 로컬 네트워크에 게시된 클레임 공급자에 관한 데이터 가져오기" 옵션을 선택합니다. 고객의 연동 메타데이터 끝점의 URI를 입력합니다. (예: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`.) 고객으로부터 이 값을 받아야 합니다. 
5. 기본 옵션을 사용해서 마법사를 완료합니다.

### 클레임 규칙 편집
1. 새로 추가한 클레임 공급자 트러스트에서 마우스 오른쪽을 클릭하고 **클레임 규칙 편집**을 선택합니다.
2. **규칙 추가**를 클릭합니다.
3. "들어오는 클레임 통과 또는 필터"를 선택하고 **다음**을 클릭합니다.
   ![Add Transform Claim Rule Wizard](./images/edit-claims-rule.png)
4. 규칙 이름을 입력합니다.
5. "들어오는 클레임 유형"에서 **UPN**을 선택합니다.
6. "모든 값 통과"를 선택합니다. 
   ![Add Transform Claim Rule Wizard](./images/edit-claims-rule2.png)
7. **마침**을 클릭합니다.
8. 단계 2 - 7을 반복하고, 들어오는 클레임 유형을 **앵커 클레임 유형** 으로 지정합니다.
9. **OK**를 클릭하고 마법사를 종료합니다.

### 홈 영역 검색 사용
다음의 PowerShell 스크립트를 실행합니다:

```
Set-ADFSClaimsProviderTrust -TargetName "name" -OrganizationalAccountSuffix @("suffix")
```

여기서 "name"은 클레임 공급자 트러스트의 식별 이름이고, "suffix"는 고객 AD의 UPN 접미사입니다(예: "corp.fabrikam.com").

이러한 구성에서, 최종 사용자가 자기 조직에서 쓰는 계정을 입력하면, AD FS는 그에 대응하는 클레임 공급자를 자동으로 선택합니다. "특정 이메일 접미사 사용을 위한 ID 공급자 구성" 절에서 [AD FS 로그인 페이지 사용자 지정하기]를 참조하세요.

## AD FS 계정 파트너 구성
고객은 다음을 수행해야 합니다:

1. 신뢰 당사자(RP) 트러스트를 추가합니다.
2. 클레임 규칙을 추가합니다.

### RP 트러스트 추가
1. Server Manager에서, **도구**를 클릭하고 **AD FS 관리**를 선택합니다.
2. 콘솔 트리에서, **AD FS**의 **신뢰 당사자 트러스트**에서 마우스 오른쪽 단추를 클릭합니다. **신뢰 당사자 트러스트 추가**를 선택합니다.
3. **클레임 인식**을 선택하고 **시작**을 클릭합니다.
4. **데이터 원본 선택** 페이지에서, "온라인 또는 로컬 네트워크에 게시된 클레임 공급자에 관한 데이터 가져오기" 옵션을 선택합니다. SaaS 공급자의 연동 메타데이터 끝점의 URI를 입력합니다. 
   ![Add Relying Party Trust Wizard](./images/add-rp-trust.png)
5. **표시 이름 지정** 페이지에서 임의의 이름을 입력합니다.
6. **액세스 제어 정책 선택** 페이지에서 정책을 선택합니다. 조직 내 모든 사람들을 허용하거나 특정한 보안 그룹을 선택할 수 있습니다.
   ![Add Relying Party Trust Wizard](./images/add-rp-trust2.png)
7. **정책** 상자에 필요한 임의의 매개변수를 선택합니다.
8. **다음**을 클릭하고 마법사를 종료합니다.

### 클레임 규칙 추가
1. 새로 추가한 신뢰 당사자 트러스트에서 마우스 오른쪽을 클릭하고 **클레임 발급 정책 편집**을 선택합니다.
2. **규칙 추가**를 클릭합니다.
3. "클레임으로 LDAP 특성 전달"을 선택하고 **다음**을 클릭합니다.
4. "UPN"과 같은 규칙 이름을 입력합니다.
5. **특성 저장**에서 **Active Directory**를 선택합니다.
   ![Add Transform Claim Rule Wizard](./images/add-claims-rules.png)
6. **LDAP 특성 대응** 절에서:
   * **LDAP 특성**에서 **사용자-계정-이름**을 선택합니다.
   * **나가는 클레임 유형**에서 **UPN**을 선택합니다.
     ![Add Transform Claim Rule Wizard](./images/add-claims-rules2.png)
7. **마침**을 클릭합니다.
8. 다시 **규칙 추가** 를 클릭합니다.
9. "사용자 지정 규칙을 사용하여 클레임 전달"을 선택하고 **다음**을 클릭합니다.
10. "앵커 클레임 유형"과 같은 규칙 이름을 입력합니다.
11. 사용자 지정 규칙**에서 다음을 입력합니다:
    
    ```
    EXISTS([Type == "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype"])=>
    issue (Type = "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype",
          Value = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn");
    ```
    
    이 규칙은 `anchorclaimtype`이라는 클레임 유형을 발급합니다. 클레임은 신뢰 당사자에게 변경 불가능한 사용자 ID로 UPN을 사용하라고 알려줍니다.
12. **마침**을 클릭합니다.
13. **확인** 을 클릭하고 마법사를 종료합니다.


<!-- Links -->
[Azure AD Connect]: /azure/active-directory/active-directory-aadconnect/
[federation trust]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[account partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[resource partner]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[Authentication instant]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Expiration time]: http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Name identifier]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[active-directory-on-azure]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[blog post]: http://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[Customizing the AD FS Sign-in Pages]: https://technet.microsoft.com/library/dn280950.aspx
[sample application]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps
[client assertion]: client-assertion.md
[active-directory-dotnet-webapp-wsfederation]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation
