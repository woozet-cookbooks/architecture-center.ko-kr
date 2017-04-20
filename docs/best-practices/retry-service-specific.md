---
title: Retry service specific guidance
description: Service specific guidance for setting the retry mechanism.
author: dragon119
ms.service: guidance
ms.topic: article
ms.date: 07/13/2016
ms.author: pnp

pnp.series.title: Best Practices
---
# 특정 서비스를 위한 다시 시도 지침
[!INCLUDE [header](../_includes/header.md)]

대부분의 Azure 서비스 및 클라이언트 SDK는 다시 시도 메커니즘을 제공합니다. 그러나 서비스마다 특성 및 요구 사항이 다르기 때문에 다시 시도 메커니즘도 서로 다르며, 따라서 각 다시 시도 메커니즘은 특정 서비스에 맞게 조정됩니다. 이 가이드는 대부분의 Azure 서비스에 대한 다시 시도 메커니즘 기능을 요약하며 해당 서비스에서 다시 시도 메커니즘을 사용, 적용  또는 확장하는 데 도움이 되는 정보를 제공합니다. 

일시적 오류 처리 및 서비스와 리소스에 대한 연결과 작업 다시 시도에 대한 일반 지침은 [다시 시도 지침](./transient-faults.md)을 참조하세요.

아래 표에는 본 가이드에 설명되어 있는 Azure 서비스에 대한 다시 시도 기능이 요약되어 있습니다. 

| **서비스** | **다시 시도 기능** | **정책 구성** | **범위** | **원격 분석 기능** |
| --- | --- | --- | --- | --- |
| **[Azure 저장소](#azure-storage-retry-guidelines)** |클라이언트에서 기본 지원 |프로그래밍 방식 |클라이언트 및 개별 작업 |TraceSource |
| **[Entity Framework를 사용하는 SQL 데이터베이스](#sql-database-using-entity-framework-6-retry-guidelines)** |클라이언트에서 기본 지원 |프로그래밍 방식 |AppDomain에 따라 전역 |없음 |
| **[ADO.NET을 사용하는 SQL 데이터베이스](#azure-storage-retry-guidelines)** |Topaz* |선언적 및 프로그래밍 방식 |코드의 단일 문 또는 블록 |사용자 지정 |
| **[서비스 버스](#service-bus-retry-guidelines)** |클라이언트에서 기본 지원 |프로그래밍 방식 |네임스페이스 관리자, 메시징 팩터리 및 클라이언트 |ETW |
| **[Azure Redis Cache](#azure-redis-cache-retry-guidelines)** |클라이언트에서 기본 지원 |프로그래밍 방식 |클라이언트 |TextWriter |
| **[DocumentDB](#documentdb-retry-guidelines)** |서비스에서 기본 지원 |구성할 수 없음 |전역 |TraceSource |
| **[Azure Search](#azure-storage-retry-guidelines)** |클라이언트에서 기본 지원 |프로그래밍 방식 |클라이언트 |ETW 또는 사용자 지정 |
| **[Active Directory](#azure-active-directory-retry-guidelines)** |Topaz*(사용자 지정 검색 전략 포함) |선언적 및 프로그래밍 방식 |코드의 블록 |사용자 지정 |

*Topaz는 [Enterprise Library 6.0][entlib]에 포함된 일시적 오류 처리 응용 프로그램 블록에 대한 식별 이름입니다. 이 가이드에 설명되어 있듯이 대부분 유형의 서비스에는 Topaz와 사용자 지정 검색 전략을 사용할 수 있습니다. Topaz에 대한 기본 전략은 본 가이드 끝의 [일시적 오류 처리 응용 프로그램 블록(Topaz) 전략](#transient-fault-handling-application-block-topaz-strategies) 섹션에 나와 있습니다. 블록은 현재 오픈 소스 프레임워크이며 Microsoft가 직접 지원하지 않습니다. 

> [!참고]
> 대부분의 Azure 기본 제공 다시 시도 메커니즘에서는 다시 시도 정책에 포함된 기능을 넘어서 오류 또는 예외 유형에 따라 서로 다른 다시 시도 정책을 적용할 방법이 없습니다. 따라서 작성 시 사용할 수 있는 최상의 지침은 최적의 평균 성능과 가용성을 제공하는 정책을 구성하는 것입니다. 정책을 미세 조정하는 한 가지 방법은 로그 파일을 분석하여 발생하는 일시적 오류의 유형을 확인하는 것입니다. 예를 들어 대부분의 오류가 네트워크 연결 문제와 관련된 경우, 첫 번째 다시 시도를 위해 오랜 시간 기다리기 보다 즉시 다시 시도를 수행할 수 있습니다. 
>
>

## Azure 저장소 다시 시도 지침
Azure 저장소 서비스에는 테이블과 blob 저장소, 파일 및 저장소 큐가 포함되어 있습니다. 

### 다시 시도 메커니즘
다시 시도는 개별 REST 작업 수준에서 이루어지며 클라이언트 API 구현의 중요한 부분입니다. 클라이언트 저장소 SDK는 [IExtendedRetryPolicy 인터페이스](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx)를 구현하는 클래스를 사용합니다.

인터페이스 구현은 여러 가지가 있습니다. 저장소 클라이언트는 테이블, blob 및 큐에 액세스하기 위해 특별히 설계된 정책에서 선택할 수 있습니다. 각 구현에서는 기본적으로 다시 시도 간격 및 기타 세부 정보를 정의하는 서로 다른 다시 시도 전략을 사용합니다. 

기본 제공 클래스는 불규칙 다시 시도 간격으로 선형(일정한 지연) 및 지수를 지원합니다. 또한 다른 프로세스가 더 높은 수준에서 다시 시도를 처리하는 경우 사용할 다시 시도 정책이 없습니다. 그러나 기본 제공 클래스에서 제공하지 않는 특정 요구 사항이 있는 경우, 고유한 다시 시도 클래스를 구현할 수 있습니다. 

RA-GRS(읽기 액세스 지역 중복 저장소)를 사용하고 요청의 결과가 다시 시도 가능한 오류인 경우, 기본과 보조 저장소 서비스 위치 간에 대체 다시 시도가 전환됩니다. 자세한 내용은 [Azure 저장소 중복 옵션](http://msdn.microsoft.com/library/azure/dn727290.aspx)을 참조하세요.

### 정책 구성
다시 시도 정책은 프로그래밍 방식으로 구성됩니다. 일반적인 절차는 **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, 또는 **QueueRequestOptions** 인스턴스를 만들고 채우는 것입니다.

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. Maximum value up to 10 seconds.
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

그 다음 요청 옵션 인스턴스를 클라이언트에서 설정할 수 있으며, 클라이언트와 관련된 모든 작업은 지정된 요청 옵션을 사용하게 됩니다. 

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

요청 옵션 클래스에 채워진 인스턴스를 매개변수로 작업 메서드에 전달하여 클라이언트 요청 옵션을 재정의할 수 있습니다.

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

**OperationContext** 인스턴스를 사용하여 다시 시도가 발생할 때와 작업이 완료되었을 때 실행할 코드를 지정합니다. 이 코드는 로그 및 원격 분석에 사용할 작업에 대한 정보를 수집할 수 있습니다.

    // Set up notifications for an operation
    var context = new OperationContext();
    context.ClientRequestID = "some request id";
    context.Retrying += (sender, args) =>
    {
      /* Collect retry information */
    };
    context.RequestCompleted += (sender, args) =>
    {
      /* Collect operation completion information */
    };
    var stats = await client.GetServiceStatsAsync(null, context);

확장된 다시 시도 정책은 오류가 다시 시도에 적합한지 여부를 나타낼 뿐만 아니라 다시 시도 횟수, 마지막 요청의 결과, 다음 다시 시도가 기본 또는 보조 위치에서  발생할지를 나타내는 **RetryContext** 개체를 반환합니다(자세한 내용은 아래 표 참조). 개체의 속성은 다시 시도를 시도할 경우 및 시기를 결정하는 데 사용할 수 있습니다. 자세한 내용은 [IExtendedRetryPolicy.Evaluate 메서드](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx)를 참조하세요.

다음 표는 기본 제공 다시 시도 정책의 기본 설정을 보여줍니다. 

| **컨텍스트** | **설정** | **기본값** | **의미** |
| --- | --- | --- | --- |
| Table / Blob / File<br />QueueRequestOptions |MaximumExecutionTime<br /><br />ServerTimeout<br /><br /><br /><br /><br />LocationMode<br /><br /><br /><br /><br /><br /><br />RetryPolicy |120초<br /><br />없음<br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br />ExponentialPolicy |모든 잠재적 다시 시도 횟수를 포함한 요청의 최대 실행 시간입니다.<br />요청에 대한 서버 제한 시간 간격입니다(값은 초로 반올림). 지정되지 않은 경우, 서버에 대한 모든 요청에 기본값이 사용됩니다. 일반적으로 서버 기본값이 사용되도록 이 설정을 생략하는 것이 가장 좋습니다.<br />저장소 계정을 RA-GRS(읽기 액세스 지역 중복 저장소) 복제 옵션을 사용해 만드는 경우, 위치 모드를 사용하여 어떤 위치에서 요청을 수신해야 하는지 표시할 수 있습니다. 예를 들어 **PrimaryThenSecondary** 를 지정하면 요청은 항상 기본 위치로 먼저 전송됩니다. 요청이 실패하면 보조 위치로 전송됩니다.<br />각 옵션에 대한 자세한 내용은 아래를 참조하세요. |
| 지수 정책 |maxAttempt<br />deltaBackoff<br /><br /><br />MinBackoff<br /><br />MaxBackoff |3<br />4초<br /><br /><br />3초<br /><br />30초 |다시 시도 횟수입니다.<br />다시 시도 사이의 백오프 간격입니다. 임의 요소를 포함해 이 timespan의 배수가 후속 다시 시도 횟수에 사용됩니다.<br />deltaBackoff에서 계산된 모든 다시 시도 간격에 추가됩니다. 이 값은 변경할 수 없습니다.<br />MaxBackoff보다 큰 경우에는 MaxBackoff가 사용됩니다. 이 값은 변경할 수 없습니다. |
| 선형 정책 |maxAttempt<br />deltaBackoff |3<br />30초 |다시 시도 횟수입니다.<br />다시 시도 사이의 백오프 간격입니다. |

### 다시 시도 사용 지침
저장소 클라이언트 API를 사용하여 Azure 저장소 서비스에 액세스할 때는 다음 지침을 고려합니다.

•	요구 사항에 적합한 Microsoft.WindowsAzure.Storage.RetryPolicies 네임스페이스에서 기본 제공 다시 시도 정책을 사용합니다. 대부분의 경우에는 이러한 정책들로 충분합니다.

•	일괄 작업, 백그라운드 작업 또는 비대화형 시나리오에서는 **ExponentialRetry** 정책을 사용합니다. 이러한 시나리오에서는 일반적으로 서비스가 복구되는 데 더 많은 시간을 허용하여 결과적으로 작업이 성공할 가능성이 높아집니다.

• **RequestOptions** 매개 변수의 **MaximumExecutionTime** 속성을 지정해서 총 실행 시간을 제한하고 제한 시간 값을 선택할 때 작업의 유형과 크기를 고려하는 것이 좋습니다.

•	사용자 지정 다시 시도를 구현해야 할 경우에는 저장소 클라이언트 클래스를 감싸는 래퍼를 만들지 않습니다. 대신에 **IExtendedRetryPolicy** 인터페이스를 통해 기존 정책을 확장하는 기능을 사용합니다.

•	RA-GRS(읽기 액세스 지역 중복 저장소)를 사용하는 경우, 기본 액세스에 실패하면 **LocationMode** 를 사용해 다시 시도에서 저장소의 보조 읽기 전용 복사본에 액세스하도록 지정할 수 있습니다. 그러나 이 옵션을 사용할 때는 기본 저장소에서의 복제가 아직 완료되지 않은 경우에 응용 프로그램이 오래된 데이터에서 성공적으로 작동할 수 있는지 확인해야 합니다.

다시 시도 작업의 경우 다음 설정을 사용해 시작하는 것이 좋습니다. 이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.  

| **컨텍스트** | **샘플 대상 E2E<br />최대 대기 시간** | **다시 시도 정책** | **설정** | **값** | **작동 방법** |
| --- | --- | --- | --- | --- | --- |
| 대화형, UI<br />또는 포그라운드 |2초 |선형 |maxAttempt<br />deltaBackoff |3<br />500 ms |시도 1 - 지연 시간 500 ms<br />시도 2 - 지연 시간 500 ms<br />시도 3 - 지연 시간 500 ms |
| 백그라운드<br />또는 일괄 처리 |30초 |지수 |maxAttempt<br />deltaBackoff |5<br />4초 |시도 1 - 3초 지연<br />시도 2 - 7초 지연<br />시도 3 - 15초 지연 |

### 원격 분석
다시 시도 횟수는 **TraceSource**에 기록됩니다. 이벤트를 캡처하여 적합한 대상 로그에 기록하려면 **TraceListener** 를 구성해야 합니다.  **TextWriterTraceListener** 또는 **XmlWriterTraceListener** 를 사용해 데이터를 로그 파일에 기록하고, **EventLogTraceListener** 를 사용해 Windows 이벤트 로그에 기록하며, **EventProviderTraceListener** 를 사용해 추적 데이터를 ETW 하위 시스템에 기록할 수 있습니다. 또한 버퍼의 자동 플러시 및 로그할 이벤트의 자세한 정보(예; 오류, 경고, 정보 및 자세한 정보 표시)를 구성할 수도 있습니다. 자세한 내용은 [.NET 저장소 클라이언트 라이브러리를 사용한 클라이언트측 로깅](http://msdn.microsoft.com/library/azure/dn782839.aspx)을 참조하세요.

작업은 **OperationContext** 인스턴스를 수신해 사용자 지정 원격 분석 논리를 추가하는 데 사용할 수 있는 **Retrying** 이벤트를 표시할 수 있습니다.  자세한 내용은 [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx)를 참조하세요.

### 예제
다음 코드 예제는 서로 다른 다시 시도 설정을 사용하여 대화형 요청과 백그라운드 요청에 대해 각각 하나씩 두 개의 **TableRequestOptions** 인스턴스를 만드는 방법을 보여줍니다. 이 예제에서는 클라이언트에서 이러한 두 다시 시도 정책을 설정하여 모든 요청에 적용하고 특정 요청에서 대화형 전략을 설정하여 클라이언트에 적용된 기본 설정을 재정의합니다. 

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. Maximum value up to 10 seconds.
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### 자세한 정보
* [Azure 저장소 클라이언트 라이브러리다시 시도 정책 권장 사항](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [저장소 클라이언트 라이브러리 2.0 – 다시 시도 정책 구현](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## Entity Framework 6을 사용하는 SQL 데이터베이스 다시 시도 지침
SQL 데이터베이스는 다양한 크기와 표준(공유) 서비스 및 프리미엄(비공유) 서비스로 사용할 수 있는 호스팅되는 SQL 데이터베이스입니다. Entity Framework는 .NET 개발자가 도메인별 개체를 사용해 관계형 데이터로 작업할 수 있는 개체 관계형 매퍼입니다. 여기서는 개발자가 일반적으로 작성해야 하는 대부분의 데이터 액세스 코드가 필요하지 않습니다. 

### 다시 시도 메커니즘
다시 시도는 [연결 복원 / 다시 시도 논리](http://msdn.microsoft.com/data/dn456835.aspx)라는 메커니즘을 통해 Entity Framework 6.0 이상을 사용하는 SQL 데이터베이스에 액세스할 때 지원됩니다. 전체 사양은 Codeplex의 [.NET Entity Framework wiki](https://entityframework.codeplex.com/wikipage?title=Connection%20Resiliency%20Spec)에서 다운로드할 수 있습니다. 다시 시도 메커니즘의 주요 기능은 다음과 같습니다. 

•	기본 추상화는 **IDbExecutionStrategy** 인터페이스입니다. 이 인터페이스는 다음을 수행합니다.

  o	동기 및 비동기 **Execute*** 메서드를 정의합니다.
  
  o	직접 사용하거나 데이터베이스 컨텍스트에서 기본 전략으로 구성하거나, 공급자 이름에 매핑하거나, 공급자 이름 및 서버 이름에 매핑할 수 있는 클래스를 정의합니다. 컨텍스트에서 구성할 경우 다시 시도는 지정된 컨텍스트 작업에 대해 여러 개가 있을 수 있는 개별 데이터베이스 작업 수준에서 수행됩니다.
  
  o	실패한 연결을 다시 시도할 시기와 방법을 정의합니다.
  
 •	여기에는 다음과 같은 **IDbExecutionStrategy** 인터페이스의 여러 가지 기본 제공 구현이 포함됩니다.
 
  o	기본값 - 다시 시도를 수행하지 않습니다.
  
  o	SQL 데이터베이스(자동)의 기본값 - 다시 시도하지 않지만 예외를 검사하고 SQL 데이터베이스 전략을 사용하는 제안으로 래핑합니다.
  
  o	SQL 데이터베이스의 기본값 - 지수(기본 클래스에서 상속) 및 SQL 데이터베이스 검색 논리
  
•	불규칙을 포함한 지수 백오프 전략을 구현합니다.

•	기본 제공 다시 시도 클래스는 상태 저장 클래스이며 스레드로부터 안전하지 않습니다. 그러나 현재 작업이 완료된 후에는 다시 사용할 수 있습니다.

•	지정된 다시 시도 횟수가 초과되면 결과는 새 예외에 래핑되며, 현재 예외를 버블 업하지 않습니다.


### 정책 구성
다시 시도는 Entity Framework 6.0 이상을 사용하는 SQL 데이터베이스에 액세스할 때 지원됩니다. 다시 시도 정책은 프로그래밍 방식으로 구성됩니다. 구성은 작업 단위로 변경할 수 없습니다. 

컨텍스트에서 기본값으로 전략을 구성하는 경우 필요에 따라 새 전략을 만드는 함수를 지정합니다. 다음 코드는 **DbConfiguration** 기본 클래스를 확장하는 다시 시도 구성 클래스를 만드는 방법을 보여줍니다. 

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

그 다음 응용 프로그램이 시작될 때 **DbConfiguration** 인스턴스의 **SetConfiguration** 메서드를 사용하여 모든 작업에 대해 기본 다시 시도 전략으로 지정할 수 있습니다. 기본적으로 EF는 구성 클래스를 자동으로 검색하고 사용합니다. 

    DbConfiguration.SetConfiguration(new BloggingContextConfiguration());

컨텍스트 클래스에 **DbConfigurationType** 특성으로 주석을 추가하여 컨텍스트에 대한 다시 시도 구성 클래스를 지정할 수 있습니다. 그러나 구성 클래스가 하나만 있는 경우, EF는 컨텍스트에 주석을 추가할 필요 없이 해당 클래스를 사용합니다. 

    [DbConfigurationType(typeof(BloggingContextConfiguration))]
    public class BloggingContext : DbContext
    { ...

특정 작업에 다른 다시 시도 전략을 사용해야 하거나 특정 작업에 대해 다시 시도를 비활성화해야 하는 경우, **CallContext**에서 플래그를 설정하여 전략을 일시 중단하거나 교환할 수 있는 구성 클래스를 만들 수 있습니다. 구성 클래스는 이 플래그를 이용하여 전략을 전환학거나 제공된 전략을 사용하지 않도록 설정하고 기본 전략을 사용할 수 있습니다. 자세한 내용은 다시 시도 실행 전략의 제힌 시힝(EF6 이상) 페이지에서 [실행 전략 일시 중단](http://msdn.microsoft.com/dn307226#transactions_workarounds)을 참조하세요.

개별 작업에 특정 다시 시도 전략을 사용하기 위한 다른 기법은 필요한 전략 클래스의 인스턴스를 만들고 매개 변수를 통해 원하는 설정을 제공하는 것입니다. 그런 다음 **ExecuteAsync** 메서드를 호출합니다.

    var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
    var blogs = await executionStrategy.ExecuteAsync(
        async () =>
        {
            using (var db = new BloggingContext("Blogs"))
            {
                // Acquire some values asynchronously and return them
            }
        },
        new CancellationToken()
    );

**DbConfiguration** 클래스를 사용하는 가장 간단한 방법은 **DbContext** 클래스와 동일한 어셈블리에 배치하는 것입니다. 그러나 다양한 대화형 및 백그라운드 다시 시도 전략과 같이 서로 다른 시나리오에서 동일한 컨텍스트가 필요한 경우에는 적합하지 않습니다. 다른 컨텍스트가 별도의 AppDomains에서 실행되는 경우, 기본 제공 지원을 사용하여 구성 파일에서 구성 클래스를 지정하거나 코드를 사용해 명시적으로 설정할 수 있습니다. 동일한 AppDomain에서 서로 다른 컨텍스트를 실행해야 하는 경우에는 사용자 지정 솔루션이 필요합니다.

자세한 내용은 [코드 기반 구성(EF6 이상)](http://msdn.microsoft.com/data/jj680699.aspx)을 참조하세요. 

다음 표는 EF6를 사용하는 경우 기본 제공 다시 시도 정책의 기본 설정을 보여줍니다. 

![Retry guidance table](./images/retry-service-specific/RetryServiceSpecificGuidanceTable4.png)

### 다시 시도 사용 지침
EF6을 사용하는 SQL 데이터베이스에 액세스하는 경우 다음 지침을 고려합니다. 

•	적절한 서비스 옵션(공유 또는 프리미엄)을 선택합니다. 공유 인스턴스는 공유 서버의 다른 테넌트도 사용하기 때문에 일반적인 연결 지연 및 제한보다 더 길게 영향을 받을 수 있습니다. 예측 가능한 성능 및 대기 시간이 짧은 안정적인 작업이 필요한 경우에는 프리미엄 옵션의 선택을 고려합니다.

•	고정 간격 전략은 Azure SQL 데이터베이스에 권장되지 않습니다. 대신에 서비스가 오버로드될 수 있고 지연 시간이 길어지면 복구에 더 많은 시간이 걸릴 수 있으므로 지수 백오프 전략을 사용하는 것이 좋습니다.

•	연결을 정의할 때 연결 및 명령 제한 시간에 적합한 값을 선택합니다. 제한 시간은 비즈니스 논리 디자인 및 테스트를 기반으로 합니다. 시간이 지나면서 데이터 양이나 비즈니스 프로세스가 변경됨에 따라 이 값을 수정해야 할 수 있습니다. 제한 시간이 너무 짧으면 데이터베이스가 사용 중일 때 조기에 연결 오류가 발생할 수 있습니다. 반대로 제한 시간이 너무 길면 실패한 연결을 감지하기까지 너무 오래 기다려서 다시 시도 논리가 올바르게 작동하지 못할 수 있습니다. 제한 시간 값은 종단 간 대기 시간의 구성 요소이지만, 컨텍스트를 저장할 때 실행될 명령 수는 쉽게 확인할 수 없습니다. **CommandTimeout** 인스턴스의 **DbContext** 속성을 설정하면 기본 제한 시간을 변경할 수 있습니다.

•	Entity Framework는 구성 파일에 정의된 다시 시도 구성을 지원합니다. 그러나 Azure에서 유연성을 최대화하려면 응용 프로그램 내에서 프로그래밍 방식으로 구성을 만드는 것을 고려해야 합니다. 다시 시도 횟수 및 다시 시도 간격과 같이 다시 시도 정책을 위한 특정 매개 변수는 서비스 구성 파일에 저장하여 런타임에 적절한 정책을 만드는 데 사용할 수 있습니다. 이렇게 하면 다시 시작해야 하는 응용 프로그램 내에서 설정을 변경할 수 있습니다.


다시 시도 작업의 경우 다음 설정을 사용해 시작하는 것이 좋습니다. 다시 시도 룃수 사이에는 지연을 지정할 수 없습니다(지연은 고정되며 지수 시퀀스로 생성). 다음과 같이 사용자 지정 다시 시도 전략을 만들지 않는 경우 최대값만 지정할 수 있습니다. 이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다. 

| **컨텍스트** | **샘플 대상 E2E<br />최대 대기 시간** | **다시 시도 정책** | **설정** | **값** | **작동 방법** |
| --- | --- | --- | --- | --- | --- |
| 대화형, UI<br />또는 포그라운드 |2초 |지수 |MaxRetryCount<br />MaxDelay |3<br />750 ms |시도 1 - 0초 지연<br />시도 2 - 750 ms 지연<br />시도 3 - 750 ms 지연 |
| 백그라운드<br />또는 일괄 처리 |30초 |지수 |MaxRetryCount<br />MaxDelay |5<br />12초 |시도 1 - 0초 지연<br />시도 2 - 1초 지연<br />시도 3 - 3초 지연<br />시도 4 - 7초 지연<br />시도 5 - 12초 지연 |

> [!참고]
> 종단 간 대기 시간 대상은 서비스에 대한 연결의 기본 제한 시간을 가정합니다. 연결 제한 시간을 더 길게 지정하면 종단 간 대기 시간이 다시 시도마다 이 추가 시간만큼 연장됩니다. 
>
>

### 예제
다음 코드 예제는 Entity Framework를 사용하는 간단한 데이터 액세스 솔루션을 정의합니다. 이 코드는 **DbConfiguration**을 확장하는 **BlogConfiguration**이라는 클래스 인스턴스를 정의하여 특정한 다시 시도 전략을 설정합니다. 

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

Entity Framework 다시 시도 메커니즘 사용에 대한 더 많은 예제는 [연결 복원 / 다시 시도 논리](http://msdn.microsoft.com/data/dn456835.aspx)에서 확인할 수 있습니다.

### 자세한 정보
* [Azure SQL 데이터베이스 성능 및 탄력성 가이드](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## ADO.NET을 사용하는 SQL 데이터베이스 다시 시도 지침
SQL 데이터베이스는 다양한 크기와 표준(공유) 서비스 및 프리미엄(비공유) 서비스로 사용할 수 있는 호스팅되는 SQL 데이터베이스입니다. 

### 다시 시도 메커니즘
SQL 데이터베이스는 ADO.NET을 사용해 액세스를 하는 경우 다시 시도에 대한기본 제공 지원을 제공하지 않습니다. 그러나 요청에서 반환된 코드는 요청이 실패한 이유를 확인하는 데 사용할 수 있습니다. [Azure SQL 데이터베이스 제한](http://msdn.microsoft.com/library/dn338079.aspx) 페이지에는 제한으로 연결을 방지하는 방법, 특정 상황에 대한 반환 코드, 이러한 코드 및 다시 시도 작업을 처리하는 방법이 설명되어 있습니다. 

Nuget 패키지 EnterpriseLibrary.TransientFaultHandling.Data(클래스**SqlAzureTransientErrorDetectionStrategy**)에서 일시적 오류 처리 응용 프로그램 블록(Topaz)을 사용하여 SQL 데이터베이스를 위한 다시 시도  메커니즘을 구현할 수 있습니다. 

또한 이 블록은 기존의 ADO.NET 1.0 API(**DbConnection 대신에 IDbConnection**)를 구현하고 다시 시도 및 연결 관리를 내부적으로 수행하는**ReliableSqlConnection**클래스를 제공합니다. 이 클래스는 편리하기는 하지만 다시 시도로 작업을 호출하기 위해 다양한 메서드 집합을 사용해야 하므로 간단하고 직접적인 대체가 아닙니다. 또한 이 클래스는 Azure 서비스를 구현하고 사용할 때 권장되는 비동기 실행을 지원하지 않습니다. 게다가 이 클래스는 ADO.NET 1.0을 사용하기 때문에 ADO.NET의 최근 개선된 기능 및 업데이트의 이점을 사용할 수 없습니다. 

### 정책 구성(ADO.NET을 사용하는 SQL 데이터베이스)
일시적 오류 처리 응용 프로그램 블록은 파일 기반 구성과 프로그래밍 방식 구성을 모두 지원합니다. 일반적으로 유연성을 최대화하기 위해서는 프로그래밍 방식 구성을 사용해야 합니다(자세한 내용은 다음 섹션의 참고를 참조). 응용 프로그램이 시작될 때 한 번 실행되는 다음 코드는 **RetryManager** 를 만들고 Azure SQL 데이터베이스에서 사용하는 데 적합한 4개의 다시 시도 전략 목록으로 채웁니다. 또한 **RetryManager**를 위한 기본 전략을 설정합니다. 기본 전략은 연결 또는 명령을 만들 때 대체 전략을 지정하지 않은 경우에 연결 및 명령에 사용됩니다. 

```csharp
RetryManager.SetDefault(new RetryManager(
    new List<RetryStrategy> { new ExponentialBackoff(name: "default", retryCount: 3,
                                                    minBackoff:     TimeSpan.FromMilliseconds(100),
                                                    maxBackoff:     TimeSpan.FromSeconds(30),
                                                    deltaBackoff:     TimeSpan.FromSeconds(1),
                                                    firstFastRetry: true),
                            new ExponentialBackoff(name: "default sql connection", retryCount: 3,
                                                    minBackoff:     TimeSpan.FromMilliseconds(100),
                                                    maxBackoff:     TimeSpan.FromSeconds(30),
                                                    deltaBackoff:     TimeSpan.FromSeconds(1),
                                                    firstFastRetry: true),
                            new ExponentialBackoff(name: "default sql command", retryCount: 3,
                                                    minBackoff:     TimeSpan.FromMilliseconds(100),
                                                    maxBackoff:     TimeSpan.FromSeconds(30),
                                                    deltaBackoff:     TimeSpan.FromSeconds(1),
                                                    firstFastRetry: true),
                            new ExponentialBackoff(name: "alt sql", retryCount: 5,
                                                    minBackoff:     TimeSpan.FromMilliseconds(100),
                                                    maxBackoff:     TimeSpan.FromSeconds(30),
                                                    deltaBackoff:     TimeSpan.FromSeconds(1),
                                                    firstFastRetry: true), },
    "default",
    new Dictionary<string, string> {
        {
        RetryManagerSqlExtensions.DefaultStrategyConnectionTechnologyName, "default sql connection"
        },
        {
        RetryManagerSqlExtensions.DefaultStrategyCommandTechnologyName, "default sql command"}
        }));
```

Azure SQL 데이터베이스에 액세스할 때 구성한 다시 시도 정책을 사용하는 방법에 대한 자세한 내용은 아래 [예제](#examples) 섹션을 참조하세요. 

일시적 오류 처리 응용 프로그램 블록에 대한 기본 전략은 본 지침의 끝에 있는 [일시적 오류 처리 응용 프로그램 블록(Topaz) 전략](#transient-fault-handling-application-block-topaz-strategies) 섹션에 나와 있습니다. 

### 다시 시도 사용 지침
ADO.NET을 사용해 SQL 데이터베이스에 액세스하는 경우 다음 지침을 고려합니다. 

•	적절한 서비스 옵션(공유 또는 프리미엄)을 선택합니다. 공유 인스턴스는 공유 서버의 다른 테넌트도 사용하기 때문에 일반적인 연결 지연 및 제한보다 더 길게 영향을 받을 수 있습니다. 예측 가능한 성능 및 대기 시간이 짧은 안정적인 작업이 추가로 필요한 경우에는 프리미엄 옵션의 선택을 고려합니다.

•	적절한 수준 또는 범위에서 다시 시도를 수행하여 데이터 불일치를 발생시키는 비멱등 작업을 방지해야 합니다. 이상적으로는 불일치를 발생시키지 않고 반복할 수 있도록 모든 작업이 멱등이어야 합니다. 그렇지 않으면 예를 들어 트랜잭션 범위 내에서 한 작업이 실패할 경우 관련된 모든 변경 사항을 실행 취소할 수 있는 수준 또는 범위에서 다시 시도를 수행해야 합니다. 자세한 내용은 [클라우드 서비스의 기본 데이터 액세스 계층 – 일시적 오류 처리](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee)를 참조하세요.

•	고정 간격 전략은 매우 짧은 간격으로 몇 번의 다시 시도만 수행되는 대화형 시나리오를 제외하고 Azure SQL 데이터베이스에서 권장되지 않습니다. 대신에 대부분의 시나리오에 지수 백오프 전략을 사용하는 것을 고려합니다.

•	연결을 정의할 때 연결 및 명령 제한 시간에 적합한 값을 선택합니다. 제한 시간이 너무 짧으면 데이터베이스가 사용 중일 때 조기에 연결 오류가 발생할 수 있습니다. 반대로 제한 시간이 너무 길면 실패한 연결을 감지하기까지 너무 오래 기다려서 다시 시도 논리가 올바르게 작동하지 못할 수 있습니다. 제한 시간 값은 종단 간 대기 시간의 구성 요소이지므로 모든 다시 시도에 대한 다시 시도 정책에 지정된 다시 시도 지연에 효과적으로 추가됩니다.

•	지수 백오프 다시 시도 로직을 사용하는 경우에도 특정 횟수의 다시 시도 후에는 연결을 닫고 새로운 연결에서 작업을 다시 시도합니다. 동일한 연결에서 동일한 작업을 여러 번 다시 시도하면 연결 문제를 야기하는 요인이 될 수 있습니다. 이 기술에 대한 예는 [클라우드 서비스의 기본 데이터 액세스 계층 – 일시적 오류 처리](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)를 참조하세요.

•	연결 풀링을 사용 중인 경우(기본값) 연결을 닫았다가 다시 연 이후에도 풀에서 동일한 연결을 선택할 수 있습니다. 이 경우 문제를 해결하는 방법은 **SqlConnection** 클래스의 **ClearPool** 메서드를 호출하여 연결을 재사용할 수 없음으로 표시하는 것입니다. 그러나 이 방법은 여러 번의 연결 시도가 실패한 이후에만 수행하고 오류가 발생한 연결과 관련된 SQL 제한 시간(오류 코드 -2)과 같은 특정 클래스의 일시적 오류가 발생하는 경우에만 수행해야 합니다.

•	데이터 액세스 코드가 **TransactionScope** 인스턴스로 시작된 트랜잭션을 사용하는 경우, 다시 시도 논리에서 연결을 다시 열고 새로운 트랜잭션 범위를 시작해야 합니다. 이러한 이유로 다시 시도가 가능한 코드 블록은 트랜잭션의 전체 범위를 포함해야 합니다.

•	일시적 오류 처리 응용 프로그램 블록은 구성 파일에 정의된 다시 시도 구성 전체를 지원합니다. 그러나 Azure에서 유연성을 최대화하려면 응용 프로그램 내에서 프로그래밍 방식으로 구성을 만드는 것을 고려해야 합니다. 다시 시도 횟수 및 다시 시도 간격과 같이 다시 시도 정책을 위한 특정 매개 변수는 서비스 구성 파일에 저장하여 런타임에 적절한 정책을 만드는 데 사용할 수 있습니다. 이렇게 하면 다시 시작해야 하는 응용 프로그램 내에서 설정을 변경할 수 있습니다.

다시 시도 작업의 경우 다음 설정을 사용해 시작하는 것이 좋습니다. 이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다. 

| **컨텍스트** | **샘플 대상 E2E<br />최대 대기 시간** | **다시 시도 전략** | **설정** | **값** | **작동 방법** |
| --- | --- | --- | --- | --- | --- |
| 대화형, UI<br />또는 포그라운드 |2초 |FixedInterval |다시 시도 횟수<br />다시 시도 간격 <br />첫 번째 빠른 다시 시도 |3<br />500 ms<br />true |시도 1 - 0초 지연<br />시도 2 - 500 ms 지연<br />시도 3 - 500 ms 지연 |
| 백그라운드<br />또는 일괄 처리 |30초 |ExponentialBackoff |다시 시도 횟수<br />최소 백오프<br />최대 백오프<br />델타 백오프<br />첫 번째 빠른 다시 시도 |5<br />0초<br />60초<br />2초<br />false |시도 1 - 0초 지연<br />시도 2 - 2초 지연<br />시도 3 - 6초 지연<br />시도 4 - 14초 지연<br />시도 5 - 30초 지연 |

> [!참고]
> 종단 간 대기 시간 대상은 서비스에 대한 연결의 기본 제한 시간을 가정합니다. 연결 제한 시간을 더 길게 지정하면 종단 간 대기 시간이 다시 시도마다 이 추가 시간만큼 연장됩니다. 
>
>

### 예제
이 섹션에서는 일시적인 오류 처리 응용 프로그램 블록을 사용하여 **RetryManager** (이전의 [정책 구성](#policy-configuration)참조)에서 구성한 다시 시도 정책 집합으로 Azure SQL 데이터베이스에 액세스하는 방법을 설명합니다. 블록을 사용하는 가장 간단한 방법은 **ReliableSqlConnection** 클래스를 사용하거나 연결에서 **OpenWithRetry** 같은 확장 메서드를 호출(자세한 내용은 [일시적 오류 처리 응용 프로그램 블록](http://msdn.microsoft.com/library/hh680934.aspx) 참조)하는 것입니다. 

그러나 현재 버전의 일시적 오류 처리 응용 프로그램 블록에서는 이러한 접근이 SQL 데이터베이스에 대한 비동기 작업을 기본적으로 지원하지 않습니다. 비동기 기법만 사용하여 SQL 데이터베이스와 같은 Azure 서비스에 액세스하는 것이 좋으므로  SQL 데이터베이스에서 일시적 오류 처리 응용 프로그램 블록을 사용하려면 다음과 같은 기법을 고려해야 합니다. 

버전 5의 C# 언어에서 간단한 비동기 지원을 사용하여 블록이 제공하는 메서드의 비동기 버전을 만들 수 있습니다. 예를 들어 다음 코드는 **ExecuteReaderWithRetry** 확장 메서드의 비동기 버전을 만드는 방법을 보여줍니다. 원래 코드에 대한 변경 사항 및 추가 사항은 강조 표시됩니다. Topaz에 대한 소스 코드는 Codeplex의 [일시적 오류 처리 응용 프로그램 블록("Topaz")](http://topaz.codeplex.com/SourceControl/latest).

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command, RetryPolicy cmdRetryPolicy,
RetryPolicy conRetryPolicy)
{
    GuardConnectionIsNotNull(command);

    // Check if retry policy was specified, if not, use the default retry policy.
    return await (cmdRetryPolicy ?? RetryPolicy.NoRetry).ExecuteAsync(async () =>
    {
        var hasOpenConnection = await EnsureValidConnectionAsync(command, conRetryPolicy).ConfigureAwait(false);

        try
        {
            return await command.ExecuteReaderAsync().ConfigureAwait(false);
        }
        catch (Exception)
        {
            if (hasOpenConnection && command.Connection != null && command.Connection.State == ConnectionState.Open)
            {
                command.Connection.Close();
            }

            throw;
        }
    }).ConfigureAwait(false);
}
```

이 새로운 비동기 확장 메서드는 블록에 포함된 동기 버전과 동일한 방식으로 사용할 수 있습니다. 

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

var retryPolicy =
    RetryManager.Instance.GetRetryPolicy<SqlDatabaseTransientErrorDetectionStrategy>("alt sql");
using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync(retryPolicy))
{
    // Do something with the values
}
```

그러나 이러한 접근은 개별 작업이나 명령만 처리하며 적절하게 정의된 트랜잭션 경계일 수 있는 문의 블록은 처리하지 않습니다. 또한 이후 시도에서 선택되지 않도록 연결 풀에서 오류가 발생한 연결을 제거하는 상황은 해결하지 못합니다. 이러한 문제를 해결하는 동기 예제는 [클라우드 서비스의 기본 데이터 액세스 계층 – 일시적 오류 처리](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Timeouts_amp_Connection_Management)에서 확인하실 수 있습니다. 이 방식은 데이터베이스 명령의 임의 시퀀스를 다시 시도할 뿐만 아니라 연결 풀을 지워 잘못된 연결을 제거하고 전체 프로세스를 계측합니다. 이 예제에에 제시된 코드는 동기이지만 비교적 쉽게 비동기 코드로 변환할 수 있습니다.

### 자세한 정보
일시적 오류 처리 응용 프로그램 블록의 사용에 관한 자세한 내용은 다음을 참조하세요. 

* [SQL Azure에서 일시적 오류 처리 응용 프로그램 블록 사용](http://msdn.microsoft.com/library/hh680899.aspx)

* [인내, 모든 승리의 비결: 일시적 오류 처리 응용 프로그램 블록의 사용](http://msdn.microsoft.com/library/dn440719.aspx)

* [클라우드 서비스의 기본 데이터 액세스 계층 – 일시적 오류 처리](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

SQL 데이터베이스를 최대한 활용하기 위한 일반적인 지침은 [Azure SQL 데이터베이스 성능 및 탄력성 가이드](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)를 참조하세요.


## 서비스 버스 다시 시도 지침
서비스 버스는 클라우드에서 또는 온 프레미스에서 호스팅되는지에 관계 없이 응용 프로그램 구성 요소에 향상된 확장성과 복원력으로 소결합된 메시지 교환을 제공하는 클라우드 메시징 플랫폼입니다. 

### 다시 시도 메커니즘
서비스 버스는 [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) 기본 클래스의 구현을 사용해 다시 시도를 구현합니다. 모든 서비스 버스 클라이언트는 **RetryPolicy** 기본 클래스의 구현 중 하나로 설정할 수 있는 **RetryPolicy** 속성을 표시합니다. 기본 제공 구현은 다음과 같습니다. 

* [RetryExponential 클래스](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx). 이 클래스는 백오프 간격, 다시 시도 횟수, 작업이 완료되는 총 시간을 제한하는 데 사용되는 **TerminationTimeBuffer** 속성을 제어하는 속성을 표시합니다.

* [NoRetry 클래스](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx). 이 클래스는 일괄 처리 또는 다단계 작업의 일환으로 다른 프로세스가 다시 시도를 관리하는 경우처럼 서비스 버스 API 수준에서 다시 시도가 필요하지 않을 때 사용됩니다.

서비스 버스 작업은 [부록: 메시징 예외](http://msdn.microsoft.com/library/hh418082.aspx)에 나열된 다양한 예외를 반환할 수 있습니다. 목록에는 어떤 경우에 작업 다시 시도가 적절한지를 표시하는 정보가 나와 있습니다. 예를 들어 [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx)은 클라이언트가 일정 시간 동안 대기한 후 작업을 다시 시도해야 한다는 것을 나타냅니다. **ServerBusyException**이 발생하면 서비스 버스가 다른 모드로 전환되기 때문에 계산된 다시 시도 지연 시간에 10초가 더 추가될 수 있습니다. 이 모드는 잠시 후 다시 설정됩니다. 

서비스 버스에서 반환된 예외는 클라이언트가 작업을 다시 시도해야 할지 여부를 나타내는 **IsTransient** 속성을 표시합니다. 기본 제공  **RetryExponential** 정책은 모든 서비스 버스 예외에 대한 기본 클래스인  **MessagingException** 클래스의 **IsTransient** 속성을 사용합니다.  **RetryPolicy** 기본 클래스의 사용자 지정 구현을 만드는 경우, 예외 유형과 **IsTransient** 속성의 조합을 사용하여 다시 시도 작업을 더 세부적으로 제어할 수 있습니다. 예를 들어 **QuotaExceededException** 을 검색하고 메시지 전송을 다시 시도하기 전에 큐를 비우는 조치를 취할 수 있습니다. 

### 정책 구성
다시 시도 정책은 프로그래밍 방식으로 설정되며 **NamespaceManager** 및 **MessagingFactory**에 대한 기본 정책으로 설정하거나 각 메시징 클라이언트에 대해 개별적으로 설정할 수 있습니다. 메시징 세션에서 기본 다시 시도 정책을 설정하려면 **NamespaceManager**의 **NamespaceManager**를 설정합니다. 

    namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                 maxBackoff: TimeSpan.FromSeconds(30),
                                                                 maxRetryCount: 3);

이 코드는 명확성을 위해 명명된 매개 변수를 사용합니다. 또는 매개 변수가 선택 사항이 아니기 때문에 이름을 생략할 수 있습니다. 

    namespaceManager.Settings.RetryPolicy = new RetryExponential(TimeSpan.FromSeconds(0.1),
                     TimeSpan.FromSeconds(30), TimeSpan.FromSeconds(2), TimeSpan.FromSeconds(5), 3);

메시징 팩터리에서 만든 모든 클라이언트에 대한 기본 다시 시도 정책을 설정하려면 **MessagingFactory**의 **RetryPolicy**를 설정합니다. 

    messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                        maxBackoff: TimeSpan.FromSeconds(30),
                                                        maxRetryCount: 3);

메시징 클라이언트에 대한 다시 시도 정책을 설정하거나 기본 정책을 재정의하려면 필요한 정책 클래스의 인스턴스를 사용하여 **RetryPolicy** 속성을 설정합니다. 

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

다시 시도 정책은 개별 작업 수준에서 설정할 수 없습니다. 그리고 메시징 클라이언트의 모든 작업에 적용됩니다. 다음 표는 기본 제공 다시 시도 정책의 기본 설정을 보여줍니다. 

![Retry guidance table](./images/retry-service-specific/RetryServiceSpecificGuidanceTable7.png)

### 다시 시도 사용 지침
서비스 버스를 사용할 때는 다음 지침을 고려합니다. 

* 기본 제공 **RetryExponential** 구현을 사용할 때는 정책이 서버 사용 중 예외에 반응하고 적절한 다시 시도 모드로 자동 전환되기 때문에 폴백 작업을 구현하지 않습니다.

* 서비스 버스는 기본 네임스페이스의 큐가 실패할 경우 별도의 네임스페이스에 있는 백업 큐로 자동 장애 조치(failover)를 구현하는 쌍을 이루는 네임스페이스라는 기능을 지원합니다. 기본 큐가 복구되면 보조 큐의 메시지를 기본 큐로 다시 전송할 수 있습니다. 이 기능은 일시적 오류를 해결하는 데 도움이 됩니다. 자세한 내용은 [비동기 메시징 패턴 및 고가용성](http://msdn.microsoft.com/library/azure/dn292562.aspx)을 참조하세요.

다시 시도 작업의 경우 다음 설정을 사용해 시작하는 것이 좋습니다. 이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다. 

![Retry guidance table](./images/retry-service-specific/RetryServiceSpecificGuidanceTable8.png)

### 원격 분석
서비스 버스는 **EventSource**를 사용해 다시 시도를 ETW 이벤트로 로그합니다. 이벤트를 캡처하여 성능 뷰어에서 보거나 적합한 대상 로그에 기록하려면 **EventListener**를 이벤트 소스에 연결해야 합니다. 이런 연결을 위해 [의미 체계 로깅 응용 프로그램 블록](http://msdn.microsoft.com/library/dn775006.aspx)을 사용할 수 있습니다. 다시 시도 이벤트는 다음과 같은 형식입니다.

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### 예제
다음 코드 예제는 다음에 대해 다시 시도 정책을 설정하는 방법을 보여줍니다. 

•	네임스페이스 관리자. 정책은 해당 관리자의 모든 작업에 적용되며 개별 작업에 대해 재정의할 수 없습니다.

•	메시징 팩터리. 정책은 해당 팩터리에서 만든 모든 클라이언트에 적용되며 개별 클라이언트를 만들 때 재정의할 수 없습니다.

•	개별 메시징 클라이언트. 클라이언트를 만든 후 해당 클라이언트에 대한 다시 시도 정책을 설정할 수 있습니다. 정책은 해당 클라이언트의 모든 작업에 적용됩니다.


```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }


            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }


            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### 자세한 정보
* [비동기 메시징 패턴 및 고가용성](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## Azure Redis Cache 다시 시도 지침
Azure Redis Cache는 널리 사용되는 오픈 소스 Redis Cache 기반의 캐시 서비스로 데이터 액세스 속도가 빠르고 대기 시간이 짧습니다. Azure Redis Cache는 Microsoft에 의해 관리되어 안전하며 Azure의 모든 응용 프로그램에서 액세스가 가능합니다.

이 섹션의 지침은 StackExchange.Redis 클라이언트를 사용하여 캐시에 액세스하는 경우를 기준으로 합니다. 다른 적합한 클라이언트의 목록은 [Redis 웹 사이트](http://redis.io/clients)에서 확인할 수 있으며, 클라이언트마다 다시 시도 메커니즘이 다를 수 있습니다. 

StackExchange.Redis 클라이언트는 단일 연결을 통해 멀티플렉싱을 사용합니다. 권장되는 사용법은 응용 프로그램 시작 시 클라이언트의 인스턴스를 만들고 캐시에 대한 모든 작업에 이 인스턴스를 사용하는 것입니다. 이러한 이유로 인해 캐시에 대한 연결은 한 번만 이루어지므로 이 섹션의 모든 지침은 캐시에 액세스하는 각 작업이 아니라 이러한 초기 연결에 대한 다시 시도 정책과 관련이 있습니다. 

### 다시 시도 메커니즘
StackExchange.Redis 클라이언트는 옵션 집합을 통해 구성되는 연결 관리자 클래스를 사용합니다. 이러한 옵션에는 캐시에 대한 연결 실패가 다시 시도되는 횟수를 지정하는 **ConnectRetry** 속성이 포함됩니다. 그러나 다시 시도 정책은 초기 연결 작업에만 사용되며 다시 시도 사이에서 대기하지 않습니다. 

### 정책 구성
다시 시도 정책은 캐시에 연결하기 전에 클라이언트에 대한 옵션을 설정하여 프로그래밍 방식으로 구성됩니다. 구성은 **ConfigurationOptions** 클래스의 인스턴스를 만들고 해당 속성을 채운 후 **Connect** 메서드에 전달하여 수행할 수 있습니다. 

```csharp
var options = new ConfigurationOptions { EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ConnectTimeout = 2000 };
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

**ConnectTimeout** 속성은 다시 시도 간의 지연이 아니라 최대 대기 시간을 밀리초 단위로 지정합니다. 

또는 옵션을 문자열로 지정하고 이를 **Connect** 메서드에 전달할 수 있습니다

```csharp
    var options = "localhost,connectRetry=3,connectTimeout=2000";
    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

또한 캐시에 연결할 때 옵션을 직접 지정할 수도 있습니다. 

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

다음 표는 기본 제공 다시 시도 정책의 기본 설정을 보여줍니다. 

| **컨텍스트** | **설정g** | **기본값**<br />(v 1.0.331) | **의미** |
| --- | --- | --- | --- |
| ConfigurationOptions |ConnectRetry<br /><br />ConnectTimeout<br /><br />SyncTimeout |3<br /><br />최대 5000 ms + SyncTimeout<br />1000 |초기 연결 작업 중 연결 시도를 반복하는 횟수입니다.<br />연결 작업에 대한 제한 시간(ms)입니다. 다시 시도 사이에 지연은 없습니다.<br />동기 작업을 허용하는 시간(ms)입니다. |

> [!참고]
> SyncTimeout은 작업의 종단 간 대기 시간에 영향을 줍니다. 그러나 일반적으로 동기 작업을 사용하는 것을 권장하지 않습니다. 자세한 내용은 [파이프라인 및 멀티플렉서](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md)를 참조하세요.
>
>

### 다시 시도 사용 지침
Azure Redis Cache를 사용할 때는 다음 지침을 고려합니다. 

•	StackExchange Redis 클라이언트는 고유한 다시 시도를 관리하는 가운데 응용 프로그램이 처음 시작되어 캐시에 대한 연결을 설정할 때만 관리합니다. 이 연결을 설정하기 위해 연결 제한 시간과 다시 시도 횟수를 구성할 수 있지만, 다시 시도 정책은 캐시에 대한 작업에 적용되지 않습니다.

•	다시 시도 메커니즘에는 다시 시도 사이의 지연이 없습니다. 지정된 연결 제한 시간이 만료된 후 지정된 횟수만큼 실패한 연결을 다시 시도할 뿐입니다.

•	다시 시도 횟수를 많이 사용하는 대신 원래 데이터 소스에 액세스하여 폴백하는 것을 고려합니다.


### 원격 분석
**TextWriter**를 사용해 다른 작업이 아닌 연결에 대한 정보를 수집할 수 있습니다. 

```csharp
var writer = new StringWriter();
...
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

생성되는 출력의 예는 다음과 같습니다. 

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### 예제
다음 코드는 StackExchange.Redis 클라이언트를 초기화하여 응용 프로그램 시작 시 Azure Redis Cache에 액세스할 때 연결 제한 시간 설정과 다시 시도 횟수를 구성하는 방법을 보여줍니다. 연결 제한 시간은 캐시에 연결될 때까지 기다려야 하는 시간이며 다시 시도 사이의 지연 시간이 아닙니다. 


다음 예는 **ConfigurationOptions**의 인스턴스를 사용해 구성을 설정하는 방법을 보여줍니다. 

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ConnectTimeout = 2000  // The maximum waiting time (ms), not the delay for retries.
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

다음 예는 옵션을 문자열로 지정하여 구성을 설정하는 방법을 보여줍니다. 

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

더 많은 예는 프로젝트 웹 사이트의 [구성](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration)을 참조하세요.

### 자세한 정보
* [Redis 웹 사이트](http://redis.io/)

## DocumentDB 다시 시도 지침
DocumentDB는 스키마 없는 JSON 데이터 모델에 대해 풍부한 쿼리 및 인덱싱 기능으로 완전히 관리되는 문서 DaaS(Database-as-a-Service)입니다. DocumentDB는 구성 가능하고 안정적인 성능과 네이티브 JavaScript 트랜잭션 처리를 제공하며 탄력적인 확장성으로 클라우드에 맞게 구축됩니다. 

### 다시 시도 메커니즘
`DocumentClient` 클래스는 실패한 작업을 자동으로 다시 시도합니다. 다시 시도 횟수와 최대 대기 시간을 설정하려면 [ConnectionPolicy.RetryOptions]을 구성합니다. 클라이언트에서 발생하는 예외는 다시 시도 정책의 범위를 벗어나거나 일시적 오류가 아닙니다. 

DocumentDB가 클라이언트를 제한하는 경우 HTTP 429 오류가 반환됩니다. `DocumentClientException`의 상태 코드를 확인합니다. 

### 정책 구성
다음 표에는 `RetryOptions` 클래스의 기본 설정을 보여줍니다. 

| 설정 | 기본값 | 설명 |
| --- | --- | --- |
| MaxRetryAttemptsOnThrottledRequests |9 |DocumentDB가 클라이언트에 요청률 제한을 적용하기 때문에 요청이 실패하는 경우의 최대 다시 시도 횟수입니다. |
| MaxRetryWaitTimeInSeconds |30 |초 단위의 최대 다시 시도 시간입니다. |

### 예제
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### 원격 분석
다시 시도 횟수는 .NET **TraceSource**를 통해 구조화되지 않은 추적 메시지로 로그됩니다. 이벤트를 캡처하여 적합한 대상 로그에 기록하려면 **TraceListener**를 구성해야 합니다. 

예를 들어 App.config 파일에 다음을 추가하는 경우, 추적 메시지는 실행 파일과 동일한 위치에 텍스트 파일 형식으로  생성됩니다. 

```
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="DocumentDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```


## Azure Search 다시 시도 지침
Azure Search를 사용하면 강력하고 정교한 검색 기능을 웹 사이트 또는 응용 프로그램에 추가하고, 검색 결과를 신속하고 쉽게 조정하며, 풍부하고 미세 조정된 순위 모델을 생성할 수 있습니다. 

### 다시 시도 메커니즘
Azure Search SDK의 다시 시도 동작은 [SearchServiceClient](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.search.searchserviceclient?redirectedfrom=MSDN&view=azuresearch-3.0.2#microsoft_azure_search_searchserviceclient) 및 [SearchIndexClient](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.search.searchindexclient?redirectedfrom=MSDN&view=azuresearch-3.0.2#microsoft_azure_search_searchindexclient) 클래스의 `SetRetryPolicy` 메서드로 제어합니다. 기본 정책은 Azure Search가 5xx 또는 408 (Request Timeout) 응답을 반환할 때 지수 백오프를 사용해 다시 시도하는 것입니다. 

### 원격 분석 
ETW를 사용하거나 사용자 지정 추적 공급자를 등록하여 추적합니다. 자세한 내용은 [AutoRest documentation][autorest]를 참조하세요.

## Azure Redis Directory 다시 시도 지침
Azure Active Directory(AD)는 핵심 디렉터리 서비스, 고급 ID 관리, 보안 및 응용 프로그램 액세스 관리를 결합하는 포괄적인 ID 및 액세스 관리 클라우드 솔루션입니다. 또한 Azure AD는 중앙 집중식 정책 및 규칙에 따라 응용 프로그램에 대한 액세스를 제어할 수 있도록 개발자에게 ID 관리 플랫폼을 제공합니다.

### 다시 시도 메커니즘
ADAL(Active Directory 인증 라이브러리)에는 Azure Active Directory를 위한 기본 제공 다시 시도 메커니즘이 없습니다. 일시적 오류 처리 응용 프로그램 블록을 사용해 Active Directory가 반환한 예외에 대한 사용자 지정 검색 메커니즘을 포함하는 다시 시도 전략을 구현할 수 있습니다. 

### 정책 구성(Azure Active Directory)
Azure Active Directory에서 일시적 오류 처리 응용 프로그램 블록을 사용할 때는 사용할 검색 전략을 정의하는 클래스를 기반으로 **RetryPolicy** 인스턴스를 만듭니다. 

```csharp
var policy = new RetryPolicy<AdalDetectionStrategy>(new ExponentialBackoff(retryCount: 5,
                                                                     minBackoff: TimeSpan.FromSeconds(0),
                                                                     maxBackoff: TimeSpan.FromSeconds(60),
                                                                     deltaBackoff: TimeSpan.FromSeconds(2)));
```

그런 다음 다시 시도 정책의 **ExecuteAction** 또는 **ExecuteAsync** 메서드를 호출하여 실행하려는 작업에 전달합니다. 

```csharp
var result = await policy.ExecuteAsync(() => authContext.AcquireTokenAsync(resourceId, clientId, uc));
```

검색 전략 클래스는 오류 발생 시 예외를 수신하고 일시적 오류일 가능성이 높은지 아니면 더 영구적인 오류인지를 감지해야 합니다. 일반적으로 예외 유형 및 상태 코드를 검사하여 이 작업을 수행합니다. 예를 들어 서비스를 사용할 수 없음(Service Unavailable) 응답은 다시 시도가 필요하다는 것을 나타냅니다. 일시적 오류 처리 응용 프로그램 블록에는 ADAL 클라이언트에서 사용하는 데 적합한 검색 전략 클래스가 포함되어 있지 않으며, 사용자 지정 검색 전략의 예는 아래의 [예제](#examples) 섹션에 제공합니다. 사용자 지정 검색 전략의 사용은 블록과 함께 제공되는 전략과 다르지 않습니다. 

일시적 오류 처리 응용 프로그램 블록에 대한 기본 전략은 본 지침의 끝에 있는 [일시적 오류 처리 응용 프로그램 블록(Topaz) 전략](#transient-fault-handling-application-block-topaz-strategies) 섹션에 나와 있습니다. 

### 다시 시도 사용 지침
Azure Active Directory를 사용할 때는 다음 지침을 고려합니다. 

•	Azure Active Directory에 REST API를 사용하고 있는 경우, 결과가 5xx 범위의 오류(500 내부 서버 오류, 502 잘못된 게이트웨이, 503 서비스를 사용할 수 없음, 504 게이트웨이 시간 초과)인 경우에만 작업을 다시 시도해야 합니다. 다른 어떤 오류에 대해서도 다시 시도를 해서는 안 됩니다.

•	ADAL(Active Directory 인증 라이브러리)을 사용하고 있는 경우 HTTP 코드에 쉽게 액세스할 수 없습니다. 따라서 ADAL 관련 예외의 속성을 확인하는 논리가 포함된 사용자 지정 검색 전략을 만들어야 합니다. 아래 [예제](#examples) 섹션을 참조하세요.

•	Azure Active Directory의 일괄 처리 시나리오에는 지수 백오프 정책을 사용하는 것이 좋습니다.

다시 시도 작업의 경우 다음 설정을 사용해 시작하는 것이 좋습니다. 이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다. 

| **컨텍스트** | **샘플 대상 E2E<br />최대 대기 시간** | **다시 시도 전략** | **설정** | **값** | **작동 방법** |
| --- | --- | --- | --- | --- | --- |
| 대화형, UI<br />또는 포그라운드 |2초 |FixedInterval |다시 시도 횟수<br />다시 시도 간격<br />첫 번째 빠른 다시 시도 |3<br />500 ms<br />true |시도 1 - 0초 지연<br />시도 2 - 500 ms 지연<br />시도 3 - 500 ms 지연 |
| 백그라운드 또는<br />일괄 처리 |60초 |ExponentialBackoff |다시 시도 횟수<br />최소 백오프<br />최대 백오프<br />델타 백오프<br />첫 번째 빠른 다시 시도 |5<br />0초<br />60초<br />2초<br />false |시도 1 - 0초 지연<br />시도 2 - 2초 지연<br />시도 3 - 6초 지연<br />시도 4 - 14초 지연<br />시도 5 - 30초 지연 |

### 예제
다음 코드 예제는 일시적 오류 처리 응용 프로그램 블록(Topaz)을 사용해 ADAL 클라이언트에서 사용하는 데 적합한 사용자 지정 일시적 오류 검색 전략을 정의하는 방법을 보여줍니다. 이 코드는 아래 코드 목록에 정의된 대로 **AdalDetectionStrategy** 형식의 사용자 지정 검색 전략에 따라 새로운**RetryPolicy** 인스턴스를 만듭니다. Topaz에 대한 사용자 지정 검색 전략은 **ITransientErrorDetectionStrategy** 인터페이스를 구현하고 다시 시도를 수행해야 하는 경우 true를 반환하거나 오류가 일시적이지 않은 것으로 나타나 다시 시도를 수행하지 않아야 하는 경우 **false**를 반환합니다. 

    using System;
    using System.Linq;
    using System.Net;
    using System.Threading.Tasks;
    using Microsoft.Practices.TransientFaultHandling;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;

    namespace RetryCodeSamples
    {
        class ActiveDirectoryCodeSamples
        {
            public async static Task Samples()
            {
                var authority = "[some authority]";
                var resourceId = “[some resource id]”;
                var clientId = “[some client id]”;

                var authContext = new AuthenticationContext(authority);

                var uc = new UserCredential(“[user]", "[password]");

                // Use Topaz with a custom detection strategy to manage retries.
                var policy =
                    new RetryPolicy<AdalDetectionStrategy>(
                        new ExponentialBackoff(
                            retryCount: 5,
                            minBackoff: TimeSpan.FromSeconds(0),
                            maxBackoff: TimeSpan.FromSeconds(60),
                            deltaBackoff: TimeSpan.FromSeconds(2)));

                var result = await policy.ExecuteAsync(() => authContext.AcquireTokenAsync(resourceId, clientId, uc));

                // Get the access token
                var accessToken = result.AccessToken;

                // Use the result, probably to authorize an API call.
            }
        }

        // TODO: This is sample code that needs validation from the WAAD team!
        // based on existing detection strategies
        public class AdalDetectionStrategy : ITransientErrorDetectionStrategy
        {
            private static readonly WebExceptionStatus[] webExceptionStatus =
                new[]
                {
                    WebExceptionStatus.ConnectionClosed,
                    WebExceptionStatus.Timeout,
                    WebExceptionStatus.RequestCanceled
                };

            private static readonly HttpStatusCode[] httpStatusCodes =
                new[]
                {
                    HttpStatusCode.InternalServerError,
                    HttpStatusCode.GatewayTimeout,
                    HttpStatusCode.ServiceUnavailable,
                    HttpStatusCode.RequestTimeout
                };

            public bool IsTransient(Exception ex)
            {
                var adalException = ex as AdalException;
                if (adalException == null)
                {
                    return false;
                }

                if (adalException.ErrorCode == AdalError.ServiceUnavailable)
                {
                    return true;
                }

                var innerWebException = adalException.InnerException as WebException;
                if (innerWebException != null)
                {
                    if (webExceptionStatus.Contains(innerWebException.Status))
                    {
                        return true;
                    }

                    if (innerWebException.Status == WebExceptionStatus.ProtocolError)
                    {
                        var response = innerWebException.Response as HttpWebResponse;
                        return response != null && httpStatusCodes.Contains(response.StatusCode);
                    }
                }

                return false;
            }
        }
    }

Active Directory Graph API 작업 다시 시도 및 반환된 오류 코드에 대한 자세한 내용은 다음을 참조하세요. 

* [코드 샘플: 다시 시도 논리](http://msdn.microsoft.com/library/azure/dn448547.aspx)
* [Azure AD Graph 오류 코드](http://msdn.microsoft.com/library/azure/hh974480.aspx)

### 자세한 정보
* [사용자 지정 검색 전략 구현](http://msdn.microsoft.com/library/hh680940.aspx) (Topaz)
* [사용자 지정 다시 시도 전략 구현](http://msdn.microsoft.com/library/hh680943.aspx) (Topaz)
* [토큰 발급 및 다시 시도 지침](http://msdn.microsoft.com/library/azure/dn168916.aspx)

## 일반 REST 및 다시 시도 지침
Azure 또는 타사 서비스에 액세스할 때는 다음을 고려합니다. 

•	재사용 가능한 코드로 다시 시도를 관리하는 체계적인 접근을 사용해 모든 클라이언트 및 모든 솔루션에 일관된 방법론을 적용할 수 있도록 합니다.

•	대상 서비스 또는 클라이언트에 기본 제공 다시 시도 메커니즘이 없는 경우, 일시적 오류 처리 응용 프로그램 블록과 같은 다시 시도 프레임워크를 사용해 다시 시도를 관리하는 것이 좋습니다. 그러면 일관된 다시 시도 동작을 구현하는 데 도움이 되며  대상 서비스에 적합한 기본 다시 시도 전략을 제공할 수 있습니다. 그러나 비표준 동작을 사용하는 서비스, 일시적 오류를 표시하는 데 예외를 사용하지 않는 서비스 또는 **Retry-Response** 회신을 사용해 다시 시도 작업을 관리하려는 경우 사용자 지정 다시 시도 코드를 만들어야 할 수 있습니다.

•	일시적 검색 논리는 REST 호출에 사용하는 실제 클라이언트 API에 따라 달라집니다. 최신 **HttpClient** 클래스와 같은 일부 클라이언트는 성공이 아닌 HTTP 상태 코드를 사용해 완료된 요청에 대한 예외를 발생시키지 않습니다. 따라서 성능은 향상되지만 일시적 오류 처리 응용 프로그램 블록을 사용할 수 없게 됩니다. 이 경우 성공이 아닌 HTTP 상태 코드에 대한 예외를 생성하는 코드를 사용해 REST API에 대한 호출을 래핑한 다음 블록에서 처리할 수 있습니다. 또는  다른 메커니즘을 사용해 다시 시도를 실행할 수 있습니다.

•	서비스에서 반환된 HTTP 상태 코드는 오류가 일시적인지 여부를 나타내는 데 도움이 될 수 있습니다. 상태 코드에 액세스하거나 해당하는 예외 유형을 확인하기 위해 클라이언트 또는 다시 시도 프레임워크에서 생성된 예외를 검사해야 할 수 있습니다. 다음 HTTP 코드는 일반적으로 다시 시도가 적합함을 나타냅니다.

o	408 요청 시간 초과

o	500 내부 서버 오류

o	502 잘못된 게이트웨이

o	503 서비스를 사용할 수 없음

o	504 게이트웨이 시간 초과

•	다시 시도 논리가 예외를 기반으로 하는 경우, 다음은 일반적으로 연결을 설정할 수 없는 일시적 오류를 나타냅니다.

o	WebExceptionStatus.ConnectionClosed

o	WebExceptionStatus.ConnectFailure

o	WebExceptionStatus.Timeout

o	WebExceptionStatus.RequestCanceled

•	서비스를 사용할 수 없음 상태의 경우, 서비스는 **Retry-After** 응답 헤더 또는 다른 사용자 지정 헤더(예: DocumentDB 서비스)에서 다시 시도를 수행하기 전에 해당되는 지연 시간을 나타낼 수 있습니다. 또한 서비스는 추가 정보를 사용자 지정 헤더로 전송하거나 응답 내용에 포함시킬 수 있습니다. 일시적 오류 처리 응용 프로그램 블록은 표준 또는 사용자 지정 "retry-after" 헤더를 사용할 수 없습니다.

•	408 요청 시간 초과를 제외한 클라이언트 오류(4xx 범위의 오류)를 나타내는 상태 코드에 대해서는 다시 시도를 수행하지 않습니다. 

•	서로 다른 네트워크 상태 및 다양한 시스템 부하와 같은 다양한 조건에서 다시 시도 전략 및 메커니즘을 철저하게 테스트합니다.

### 다시 시도 전략
다음은 다시 시도 전략 간격의 일반적인 유형입니다. 

* **지수**: 지정된 횟수만큼 다시 시도를 수행하고 임의 지정된 지수 백오프 방법을 사용하여 다시 시도 사이의 간격을 결정하는 다시 시도 정책입니다. 예:

        var random = new Random();

        var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                    random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                    (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
        var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                       this.maxBackoff.TotalMilliseconds);
        retryInterval = TimeSpan.FromMilliseconds(interval);
        
* **증분**: 다시 시도 횟수가 지정되고 다시 시도 사이의 간격이 증분되는 다시 시도 전략입니다. 예:

        retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                       (this.increment.TotalMilliseconds * currentRetryCount));
* **LinearRetry**: 다시 시도 사이의 지정된 고정 시간 간격을 사용해 지정된 횟수만큼 다시 시도를 수행하는 다시 시도 정책입니다. 예:

        retryInterval = this.deltaBackoff;

### 자세한 정보
* [회로 차단기 전략](http://msdn.microsoft.com/library/dn589784.aspx)

## 일시적 오류 처리 응용 프로그램 블록(Topax) 전략
일시적 오류 처리 응용 프로그램 블록에는 다음과 같은 기본 전략이 있습니다. 

| **전략** | **설정** | **기본값** | **의미** |
| --- | --- | --- | --- |
| **지수** |retryCount<br />minBackoff<br /><br />maxBackoff<br /><br />deltaBackoff<br /><br />fastFirstRetry |10<br />1초<br /><br />30초<br /><br />10초<br /><br />true |다시 시도 횟수입니다.<br />최소 백오프 시간입니다. 이 값 또는 계산된 백오프보다 높은 값은 다시 시도 지연 시간으로 사용됩니다.<br />최소 백오프 시간입니다. 이 값 또는 계산된 백오프보다 낮은 값은 다시 시도 지연 시간으로 사용됩니다.<br />다시 시도 사이의 지수 지연 시간에 대한 임의 델타 값을 계산하는 데 사용되는 값입니다.<br />첫 번째 다시 시도가 즉시 수행되는지 여부입니다. |
| **증분** |retryCount<br />initialInterval<br />increment<br /><br />fastFirstRetry<br /> |10<br />1초<br />1초<br /><br />true |다시 시도 횟수입니다.<br />첫 번째 다시 시도에 적용될 초기 간격입니다.<br />다시 시도 사이의 점진적 지연 시간을 계산하는 데 사용될 증분 시간 값입니다.<br />첫 번째 다시 시도가 즉시 수행되는지 여부입니다. |
| **선형(고정 간격)** |retryCount<br />retryInterval<br />fastFirstRetry<br /> |10<br />1초<br />true |다시 시도 횟수입니다.<br />다시 시도 사이의 지연 시간입니다.<br />첫 번째 다시 시도가 즉시 수행되는지 여부입니다. |

일시적 오류 처리 응용 프로그램 블록을 사용하는 예제는 이 지침의 앞 부분에서 ADO.NET을 사용하는 Azure SQL 데이터베이스 및 Azure Active Directory에 대한 예제 섹션을 참조하세요.
<!-- links -->

[autorest]: https://github.com/Azure/autorest/tree/master/docs
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[entlib]: http://msdn.microsoft.com/library/dn440719.aspx
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
