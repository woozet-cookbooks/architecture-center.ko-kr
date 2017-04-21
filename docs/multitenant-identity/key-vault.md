---
title: Use Key Vault to protect application secrets
description: How a use the Key Vault service to store application secrets
author: MikeWasson
ms.service: guidance
ms.topic: article
ms.date: 02/16/2016
ms.author: pnp

pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
---
# 응용 프로그램 암호 보호를 위한 Azure 주요 자격 증명 모음 사용

[![GitHub](../_images/github.png) 샘플 코드][sample application]

일반적으로 중요하고 보호해야 할 응용 프로그램 설정이 있습니다. 예를 들면:

* 데이터베이스 연결 문자열
* 암호
* 암호화 키

보안의 모범 사례로서, 이 암호들을 소스 제어에 저장하면 안 됩니다. 소스 코드 저장소가 개인용이어도 정보가 유출되는 것은 너무 쉽습니다. 이는 단순히 일반 대중으로부터 비밀을 유지하는 문제가 아닙니다. 큰 프로젝트에서 어떤 개발자와 운영자가 제작 암호를 액세스하는 것을 제한하고 싶을 수 있습니다. (시험이나 개발 환경에 대한 설정은 다릅니다.)

보다 안전한 옵션은 이 암호들을 [Azure 주요 자격 증명 모음][KeyVault]에 저장하는 것입니다. 주요 자격 증명 모음은 암호화 키 및 기타 암호를 관리하기 위한 클라우드 호스트된(cloud-hosted) 서비스입니다. 이 문서는 앱의 구성 설정을 저장하는 데 필요한 주요 자격 증명 모음을 사용하는 방법을 설명합니다.

[Tailspin Surveys][Surveys] 응용 프로그램에서 다음 설정들은 암호입니다:

* 데이터베이스 연결 문자열.
* Redis 연결 문자열.
* 웹 응용 프로그램에 대한 클라이언트 암호.

주요 자격 증명 모음에 구성 암호를 저장하기 위해서, Surveys 응용 프로그램은 ASP.NET Core 1.0 [구성 시스템][configuration]을 끌어들인 사용자 지정 구성 공급자를 구현합니다. 사용자 지정 공급자는 startup의 주요 자격 증명 모음에서 구성 설정을 읽어 들입니다.

Surveys 응용 프로그램은 다음 장소에서 구성 설정을 로드합니다:

* appsettings.json 파일
* [사용자 암호 저장소][user-secrets] (개발 환경만; 시험용)
* 호스팅 환경 (Azure 웹 앱에서 앱 설정)
* 주요 자격 증명 모음

이 모두는 앞의 것을 재정의하므로, 주요 자격 증명 모음에 저장된 모든 설정에 우선순위가 있습니다:

> [!참고]
> 기본값으로, 주요 자격 증명 모음의 구성 공급자는 사용하지 않도록 설정됩니다. 응용 프로그램을 로컬로 실행할 경우 불필요하기 때문입니다. 생산품 배포에서는 사용할 수 있습니다.
> 
> 주요 자격 증명 모음은 .NET Core에서 지원되지 않는데, [Microsoft.Azure.KeyVault][Microsoft.Azure.KeyVault] 패키지를 필요로 하기 때문입니다.
> 
> 

startup에서, 응용 프로그램은 등록된 모든 구성 공급자로부터 설정을 읽어 들여, 강력한 형식의 옵션 개체의 정보 표시에 사용합니다. (자세한 정보는 [옵션 및 구성 개체 사용하기][options]를 참조하세요.)

## 구현
[KeyVaultConfigurationProvider][KeyVaultConfigurationProvider] 클래스는 ASP.NET Core 1.0 [구성 시스템][configuration]과 자동 연결된 구성 공급자입니다.

`KeyVaultConfigurationProvider`를 사용하려면, startup 클래스에서 `AddKeyVaultSecrets` 확장 메서드를 호출합니다:

```csharp
    var builder = new ConfigurationBuilder()
        .SetBasePath(appEnv.ApplicationBasePath)
        .AddJsonFile("appsettings.json");

    if (env.IsDevelopment())
    {
        builder.AddUserSecrets();
    }
    builder.AddEnvironmentVariables();
    var config = builder.Build();

    // Add key vault configuration:
    builder.AddKeyVaultSecrets(config["AzureAd:ClientId"],
        config["KeyVault:Name"],
        config["AzureAd:Asymmetric:CertificateThumbprint"],
        Convert.ToBoolean(config["AzureAd:Asymmetric:ValidationRequired"]),
        loggerFactory);
```

`KeyVaultConfigurationProvider` 가 구성 설정 일부를 요구하는데, 이 설정은 다른 구성 원본 중 하나에 저장되어야 한다는 점에 주의하세요.

응용 프로그램이 시작되면, `KeyVaultConfigurationProvider` 는 주요 자격 증명 모음에 모든 암호를 열거합니다. 각 암호에서 'ConfigKey'라는 태그를 찾습니다. 태그 값은 구성 설정의 이름입니다.

> [!참고]
> [태그][key-tags]는 키와 같이 저장되는 선택적 메타데이터입니다. 키 이름은 콜론 (:) 문자를 포함할 수 없기 때문에 여기서 태그가 사용되었습니다.
> 
> 

```csharp
var kvClient = new KeyVaultClient(GetTokenAsync);
var secretsResponseList = await kvClient.GetSecretsAsync(_vault, MaxSecrets, token);
foreach (var secretItem in secretsResponseList.Value)
{
    //The actual config key is stored in a tag with the Key "ConfigKey"
    // because ':' is not supported in a shared secret name by Key Vault.
    if (secretItem.Tags != null && secretItem.Tags.ContainsKey(ConfigKey))
    {
        var secret = await kvClient.GetSecretAsync(secretItem.Id, token);
        Data.Add(secret.Tags[ConfigKey], secret.Value);
    }
}
```

> [!참고]
> [KeyVaultConfigurationProvider.cs]를 참조하세요.
> 
> 

## Surveys 앱에서 주요 자격 증명 모음 설정하기
필수 구성 요소:

* [Azure 리소스 관리자 Cmdlets][azure-rm-cmdlets]를 설치하세요.
* [Surveys 응용 프로그램 실행하기][readme]에 설명된 것과 같이 Surveys 응용 프로그램을 구성합니다.

상위 수준 단계:

1. 테넌트에서 관리 사용자를 설정합니다.
2. 클라이언트 인증서를 설정합니다.
3. 주요 자격 증명 모음을 만듭니다.
4. 주요 자격 증명 모음에 구성 설정을 추가합니다.
5. 주요 자격 증명 모음을 사용하는 코드의 주석 처리를 제거합니다.
6. 응용 프로그램의 사용자 암호를 업데이트합니다.

### 관리 사용자를 설정합니다.
> [!참고]
> 주요 자격 증명 모음을 만들려면 Azure 구독을 관리할 수 있는 계정을 사용해야 합니다. 또한 주요 자격 증명 모음의 읽기 권한을 부여받은 응용 프로그램은 그 계정과 같은 테넌트에 등록되어야 합니다.
> 
> 

이 단계에서 Surveys 앱이 등록된 테넌트의 사용자로 로그인하면서 주요 자격 증명 모음을 만들 수 있는지 확인합니다.

첫째, Azure 구독과 관련된 디렉터리를 변경합니다.

1. [Azure 관리 포털][azure-management-portal]에 로그인합니다.
2. **설정**을 클릭합니다.
   
    ![Settings](./images/settings.png)
3. Azure 구독을 선택합니다.
4. 포털 단추에서 **디렉터리 편집** 을 클릭합니다.
   
    ![Settings](./images/edit-directory.png)
5. "관련 디렉터리 변경"에서, Surveys 응용 프로그램이 등록된 Azure AD 테넌트를 선택합니다.
   
    ![Settings](./images/edit-directory2.png)
6. 화살표 단추를 클릭하고 대화 상자를 종료합니다.

Surverys 응용 프로그램이 등록된 Azure AD 테넌트에서 관리 사용자를 만듭니다.

1. [Azure 관리 포털][azure-management-portal]에 로그인합니다.
2. 응용 프로그램이 등록된 Azure AD 테넌트를 선택합니다.
3. **사용자** > **사용자 추가**를 클릭합니다.
4. **사용자 추가** 대화 상자에서 사용자를 전역 관리자 역할에 지정합니다.

Azure 구독에 대한 공동 관리자로 관리 사용자를 추가합니다.

1. [Azure 관리 포털][azure-management-portal]에 로그인합니다.
2. **설정** 을 클릭하고 Azure 구독을 선택합니다.
3. **관리자**를 클릭합니다.
4. 포털 하단의 **추가** 를 클릭합니다.
5. 앞서 만든 관리 사용자의 이메일을 입력합니다.
6. 구독 확인란에 확인 표시를 합니다.
7. 확인 표시 단추를 클릭하고 대화상자를 종료합니다.

![Add a co-administrator](./images/co-admin.png)

### 클라이언트 인증서 설정
1. 다음과 같이 PowerShell 스크립트 [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault]을 실행합니다:
   
    ```
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    `Subject` 매개변수에, "surveysapp"과 같이, 아무 이름이나 입력합니다. 스크립트는 자체 서명된 인증서를 생성해서 "현재 사용자/개인" 인증서 저장소에 저장합니다.
2. 스크립트의 출력은 JSON 조각입니다. 이 출력을 다음과 같이 웹 앱의 응용 프로그램 매니페스트에 추가합니다.
   
   a. [Azure 관리 포털][azure-management-portal]에 로그인하여 Azure AD 디렉터리를 탐색합니다.
   b. **Applications**를 클릭합니다.
   c. Surveys 응용 프로그램을 선택합니다.
   d. **Manage Manifest** 를 클릭하고 **Download Manifest**를 선택합니다.
   e. 텍스트 편집기에서 매니페스트 JSON 파일을 엽니다. 스크립트의 출력을 `keyCredentials` 속성에 붙여넣습니다. 결과는 다음과 같습니다.
      
      ```
        "keyCredentials": [
            {
              "type": "AsymmetricX509Cert",
              "usage": "Verify",
              "keyId": "29d4f7db-0539-455e-b708-....",
              "customKeyIdentifier": "ZEPpP/+KJe2fVDBNaPNOTDoJMac=",
              "value": "MIIDAjCCAeqgAwIBAgIQFxeRiU59eL.....
            }
          ],
      ```          
   f. 변경 내용을 JSON 파일에 저장합니다.
   g. 포털로 돌아갑니다. **매니페스트 관리** > **매니페스트 업로드** 를 클릭하고 JSON 파일을 업로드합니다.
3. 웹 API의 응용 프로그램 매니페스트에 동일한 JSON 조각을 추가합니다 (Surveys.WebAPI).
4. 인증서 지문을 받으려면 다음 명령어를 실행합니다.
   
    ```
    certutil -store -user my [subject]
    ```
   여기서 `[subject]`는 PowerShell 스크립트에서 Subject에 지정한 값입니다. 지문은 "Cert Hash(sha1)"에 있습니다. 16진수 사이에 있는 여백을 제거합니다.

나중에 지문을 사용할 것입니다.

### 주요 자격 증명 모음 생성
1. 다음과 같이 PowerShell 스크립트 [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault]을 실행합니다:
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```
   
    인증서를 요구하는 메시지가 표시되면 앞서 만든 Azure AD 사용자로 로그인합니다. 스크립트는 새 리소스 그룹과 그 리소스 그룹에 새 주요 자격 증명 모음을 만듭니다.
   
    참고: -Location 매개변수에서 유효한 지역 목록을 얻기 위해 다음의 PowerShell 명령어를 사용할 수 있습니다:
   
    ```
    Get-AzureRmResourceProvider -ProviderNamespace "microsoft.keyvault" | Where-Object { $_.ResourceTypes.ResourceTypeName -eq "vaults" } | Select-Object -ExpandProperty Locations
    ```
2. 매개변수를 다음과 같이 설정하고 SetupKeyVault.ps를 다시 실행합니다:
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<web app client ID>>", "<<web API client ID>>")
    ```
   
    여기서
   
   * key vault name = 전 단계에서 주요 자격 증명 모음에 부여한 이름.
   * web app client ID = Surveys 웹 응용 프로그램의 클라이언트 ID.
   * web api client ID = Surveys.WebAPI 응용 프로그램의 클라이언트 ID.
     
     예:
     
     ```
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
     ```
     
     > [!참고]
     > [Azure 관리 포털][azure-management-portal]에서 클라이언트 ID를 가져올 수 있습니다. Azure AD 테넌트를 선택하고 응용 프로그램을 선택한 다음 **구성**을 클릭합니다.
     > 
     > 
     
     이 스크립트는 웹 앱과 웹 API가 주요 자격 증명 모음에서 암호를 검색하는 권한을 부여합니다. 자세한 정보는 [주요 자격 증명 모음 시작](/azure/key-vault/key-vault-get-started/)을 참조하세요.

### 주요 자격 증명 모음에 구성 설정 추가
1. 아래와 같이 SetupKeyVault.ps를 실행합니다::
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName RedisCache -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true" -ConfigName "Redis:Configuration"
    ```
   여기서
   
   * key vault name = 전 단계에서 주요 자격 증명 모음에 부여한 이름.
   * Redis DNS name = Redis 캐시 인스턴스의 DNS 이름.
   * Redis access key = Redis 캐시 인스턴스의 액세스 키.
     
     이 명령은 주요 자격 증명 모음에 암호를 추가합니다. 암호는 이름/값의 쌍과 태그를 더한 것입니다:
   * •	키 이름은 응용 프로그램에 사용되지 않지만 주요 자격 증명 모음에서 고유한 이름이어야 합니다.
   * •	키 값은 구성 옵션의 값입니다. 이 경우에 Redis 연결 문자열입니다.
   * "ConfigKey" 태그는 구성 키의 이름을 갖고 있습니다.
2. 이 때, 주요 자격 증명 모음에 암호를 잘 저장했는지 시험해보는 것이 좋겠습니다. 다음의 PowerShell 명령을 실행합니다:
   
    ```
    Get-AzureKeyVaultSecret <<key vault name>> RedisCache | Select-Object *
    ```
    출력은 암호값과 메타데이터를 더한 것을 보여주어야 합니다:
   
    ![PowerShell output](./images/get-secret.png)
3. 데이터베이스 연결 문자을을 추가하려면 SetupKeyVault.ps를 다시 실행합니다:
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName ConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```
   
    여기서 `<<DB connection string>>`은 데이터베이스 연결 문자열 값입니다.
   
    로컬 데이터베이스로 시험하려면, Tailspin.Surveys.Web/appsettings.json 파일에서 연결 문자열을 복사합니다. 그렇게 하고 나서, 이중 역슬래시('\\\\')를 반드시 단일 역슬래시로 변경하도록 합니다. JSON 파일에서 이중 역슬래시는 이스케이프 문자입니다.
   
    예:
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName ConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true" -ConfigName "Data:SurveysConnectionString"
    ```

### 주요 자격 증명 모음을 사용하는 코드의 주석 처리 제거
1. Tailspin.Surveys 솔루션을 엽니다.
2. [Tailspin.Surveys.Web/Startup.cs][web-startup] 에서 다음 코드 블록을 찾아 주석 처리를 제거합니다.
   
    ```csharp
    //#if DNX451
    //            _configuration = builder.Build();
    //            builder.AddKeyVaultSecrets(_configuration["AzureAd:ClientId"],
    //                _configuration["KeyVault:Name"],
    //                _configuration["AzureAd:Asymmetric:CertificateThumbprint"],
    //                Convert.ToBoolean(_configuration["AzureAd:Asymmetric:ValidationRequired"]),
    //                loggerFactory);
    //#endif
    ```
3. [Tailspin.Surveys.WebAPI/Startup.cs][web-api-startup] 에서 다음 코드 블록을 찾아 주석 처리를 제거합니다.
   
    ```csharp
    //#if DNX451
    //            var config = builder.Build();
    //            builder.AddKeyVaultSecrets(config["AzureAd:ClientId"],
    //                config["KeyVault:Name"],
    //                config["AzureAd:Asymmetric:CertificateThumbprint"],
    //                Convert.ToBoolean(config["AzureAd:Asymmetric:ValidationRequired"]),
    //                loggerFactory);
    //#endif
    ```
4. [Tailspin.Surveys.Web/Startup.cs][web-startup]에서 `ICredentialService`를 등록하는 코드를 찾습니다. `CertificateCredentialService`를 사용한 줄에서 주석 처리를 제거하고, `ClientCredentialService`를 사용한 줄을 주석으로 처리합니다 :
   
    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```
   
    이러한 변경은 웹 앱이 [클라이언트 어설션][client-assertion]을 사용하여 Oauth 액세스 토큰을 가져올 수 있게 합니다. 클라이언트 어설션이 있는 경우, OAuth 클라이언트 암호가 필요 없습니다. 또는 주요 자격 증명 모음에 클라이언트 암호를 저장할 수 있습니다. 그러나, 주요 자격 증명 모음과 클라이언트 어설션 둘 다 클라이언트 인증서를 사용하므로, 주요 자격 증명 모음을 사용할 경우 클라이언트 어설션도 사용하는 것이 바람직합니다.

### 사용자 암호 업데이트
Solution Explorer에서, Tailspin.Surveys.Web 프로젝트에서 마우스 오른쪽 단추를 클릭하고 **사용자 암호 관리**를 선택합니다. secrets.json 파일에서, 기존의 JSON을 삭제하고 다음 코드에 붙여넣기 합니다:

    ```
    {
      "AzureAd": {
        "ClientId": "[Surveys web app client ID]",
        "PostLogoutRedirectUri": "https://localhost:44300/",
        "WebApiResourceId": "[App ID URI of your Surveys.WebAPI application]",
        "Asymmetric": {
          "CertificateThumbprint": "[certificate thumbprint. Example: 105b2ff3bc842c53582661716db1b7cdc6b43ec9]",
          "StoreName": "My",
          "StoreLocation": "CurrentUser",
          "ValidationRequired": "false"
        }
      },
      "KeyVault": {
        "Name": "[key vault name]"
      }
    }
    ```

[꺾쇠괄호] 안에 있는 항목을 정확한 값으로 대체합니다.

* `AzureAd:ClientId`: Surveys 앱의 클라이언트 ID.
* `AzureAd:WebApiResourceId`: Azure AD에서 Surveys.WebAPI 응용 프로그램을 만들 때 지정한 앱 ID.
* `Asymmetric:CertificateThumbprint`: 클라이언트 인증서를 만들 때, 앞에서 가져온 인증서 지문.
* `KeyVault:Name`: 주요 자격 증명 모음의 이름.

> [!참고]
> 앞에서 만든 인증서가 최상위 인증기관(CA)이 서명한 것이 아니므로 `Asymmetric:ValidationRequired`는 false입니다. 생산품에서, 최상위 CA 기관이 서명한 인증서를 사용하고 `ValidationRequired`를 true로 설정합니다.
> 
> 

업데이트된 secrets.json 파일을 저장합니다.

Solution Explorer에서, Tailspin.Surveys.WebApi 프로젝트에서 마우스 오른쪽 단추를 클릭하고 **사용자 암호 관리**를 선택합니다. 기존의 JSON을 삭제하고 다음 코드에 붙여넣기 합니다:

```
{
  "AzureAd": {
    "ClientId": "[Surveys.WebAPI client ID]",
    "WebApiResourceId": "https://tailspin5.onmicrosoft.com/surveys.webapi",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

[꺾쇠괄호] 안에 있는 대체하고 secrets.json 파일을 저장합니다.

> [!참고]
> web API의 경우, 반드시 Surveys 응용 프로그램이 아닌 Surveys.WebAPI 응용 프로그램의 클라이언트 ID를 사용하도록 합니다.
> 
> 

[**다음**][adfs]

<!-- Links -->
[adfs]: ./adfs.md
[authorize-app]: /azure/key-vault/key-vault-get-started//#authorize
[azure-management-portal]: https://manage.windowsazure.com/
[azure-rm-cmdlets]: https://msdn.microsoft.com/library/mt125356.aspx
[client-assertion]: client-assertion.md
[configuration]: https://docs.asp.net/en/latest/fundamentals/configuration.html
[KeyVault]: https://azure.microsoft.com/services/key-vault/
[KeyVaultConfigurationProvider]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Configuration.KeyVault/KeyVaultConfigurationProvider.cs
[key-tags]: https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_Keytags
[Microsoft.Azure.KeyVault]: https://www.nuget.org/packages/Microsoft.Azure.KeyVault/
[options]: https://docs.asp.net/en/latest/fundamentals/configuration.html#using-options-and-configuration-objects
[readme]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/docs/running-the-app.md
[Setup-KeyVault]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[user-secrets]: http://go.microsoft.com/fwlink/?LinkID=532709
[web-startup]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Web/Startup.cs
[web-api-startup]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.WebAPI/Startup.cs

[KeyVaultConfigurationProvider.cs]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps/blob/master/src/Tailspin.Surveys.Configuration.KeyVault/KeyVaultConfigurationProvider.cs
[sample application]: https://github.com/Azure-Samples/guidance-identity-management-for-multitenant-apps
