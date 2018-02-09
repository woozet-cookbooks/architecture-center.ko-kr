---
title: "Azure Service Fabric으로 Azure Cloud Services 응용 프로그램 마이그레이션"
description: "Azure Cloud Services에서 Azure Service Fabric으로 응용 프로그램을 마이그레이션하는 방법입니다."
author: MikeWasson
ms.date: 04/27/2017
ms.openlocfilehash: 73e34c53ffd2f2eeb466d12a5f6c65dcfdaae389
ms.sourcegitcommit: 2c9a8edf3e44360d7c02e626ea8ac3b03fdfadba
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/03/2018
---
# <a name="migrate-an-azure-cloud-services-application-to-azure-service-fabric"></a>Azure Service Fabric으로 Azure Cloud Services 응용 프로그램 마이그레이션 

[![GitHub](../_images/github.png) 샘플 코드][sample-code]

이 문서는 Azure Cloud Services에서 Azure Service Fabric으로 응용 프로그램을 마이그레이션하는 방법을 설명합니다. 아키텍처 관련 결정 사항 및 권장 사례에 중점을 둡니다. 

이 프로젝트의 경우 설문 조사라고 하는 Cloud Services 응용 프로그램으로 시작했으며, Service Fabric으로 포팅했습니다. 목적은 응용 프로그램을 최대한 적게 변경하며 마이그레이션하는 것입니다. 이후 문서에서는 마이크로 서비스 아키텍처를 채택하여 응용 프로그램을 Service Fabric에 최적화합니다.

이는 이 문서를 읽기 전에 Service Fabric 및 마이크로 서비스 아키텍처의 기본 사항을 전반적으로 이해하는 데 유용할 것입니다. 다음 문서를 참조하세요.

- [Azure Service Fabric의 개요][sf-overview]
- [응용 프로그램 구축에 마이크로 서비스 접근 방식이 필요한 이유][sf-why-microservices]


## <a name="about-the-surveys-application"></a>설문 조사 응용 프로그램에 대한 정보

2012년에 패턴 및 연습 그룹은 [클라우드에 대한 다중 테넌트 응용 프로그램 개발][tailspin-book]이란 책을 위해 설문 조사라는 응용 프로그램을 만들었습니다. 책은 설문 조사 응용 프로그램을 디자인하고 구현하는 Tailspin이라는 가상의 회사를 설명합니다.

설문 조사는 고객이 설문 조사를 만들 수 있도록 허용하는 다중 테넌트 응용 프로그램입니다. 고객이 응용 프로그램에 등록하면 고객의 조직 내 팀원들이 설문 조사를 만들고, 게시하고, 분석에 대한 결과를 수집할 수 있습니다. 응용 프로그램에는 사람들이 설문 조사를 수행할 수 있는 공용 웹 사이트가 포함됩니다. [여기][tailspin-scenario]에서 Tailspin 시나리오에 대해 자세히 읽어보세요.

이제 Tailspin은 Azure에서 실행되는 Service Fabric를 사용하여 설문 조사 응용 프로그램을 마이크로 서비스 아키텍처로 이동하고자 합니다. 응용 프로그램은 이미 Cloud Services 응용 프로그램으로 배포했으므로 Tailspin은 다중 단계 접근 방식을 채택합니다.

1.  응용 프로그램에 대한 변경을 최소화하면서 클라우드 서비스를 Service Fabric에 포팅합니다.
2.  마이크로 서비스 아키텍처로 이동하여 응용 프로그램을 Service Fabric에 최적화합니다.

이 문서에서는 첫 번째 단계를 설명합니다. 이후 문서에서는 두 번째 단계를 설명합니다. 실제 프로젝트에서는 두 단계가 겹칠 가능성이 높습니다. Service Fabric에 포팅하는 동안 마이크로 서비스로 응용 프로그램을 재설계할 수도 있습니다. 나중에 성긴(coarse-grained) 서비스를 더 작은 서비스로 나누면서 아키텍처를 추가로 구체화할 수도 있습니다.  

응용 프로그램 코드는 [GitHub][sample-code]에서 사용할 수 있습니다. 이 리포지토리에는 Cloud Services 응용 프로그램 및 Service Fabric 버전이 모두 포함되어 있습니다. 

> 클라우드 서비스는 *다중 테넌트 응용 프로그램 개발* 책에 나오는 원래 응용 프로그램의 업데이트된 버전입니다.

## <a name="why-microservices"></a>마이크로 서비스를 사용하는 이유

마이크로 서비스에 대한 심도있는 논의는 이 문서의 범위를 벗어납니다. 하지만 마이크로 서비스 아키텍처로 이동함으로써 Tailspin에서 얻고자 하는 몇 가지 이점은 다음과 같습니다.

- **응용 프로그램 업그레이드**. 서비스를 독립적으로 배포할 수 있으므로 응용 프로그램을 업그레이드하는 증분 방식을 사용할 수 있습니다.
- **복원력 및 결함 분리**. 서비스가 실패하는 경우 다른 서비스가 계속 실행합니다.
- **확장성**. 서비스를 독립적으로 확장할 수 있습니다.
- **유연성**. 서비스는 기술 스택이 아닌 비즈니스 시나리오를 위해 설계되어 새 기술, 프레임워크 또는 데이터 저장소로 서비스를 더욱 쉽게 마이그레이션할 수 있습니다.
- **기민한 개발**. 개별 서비스에는 모놀리식 응용 프로그램보다 코드가 더 적으므로 코드 베이스를 더 쉽게 이해, 추론 및 테스트할 수 있습니다.
- **집중화된 소규모 팀**. 응용 프로그램이 많은 소형 서비스로 나뉘기 때문에 집중화된 소규모 팀이 각 서비스를 빌드할 수 있습니다.

## <a name="why-service-fabric"></a>왜 Service Fabric인가?
      
Service Fabric에는 다음과 같이 분산된 시스템에 필요한 기능 대부분이 빌드되어 있어 마이크로 서비스 아키텍처에 적합합니다.

- **클러스터 관리**. Service Fabric은 노드 장애 조치(failover), 상태 모니터링 및 기타 클러스터 관리 기능을 자동으로 처리합니다.
- **수평적 크기 조정**. Service Fabric 클러스터에 노드를 추가하면 서비스가 새 노드에 분산됨에 따라 응용 프로그램은 자동으로 크기를 조정합니다.
- **서비스 검색**. Service Fabric은 명명된 서비스에서 엔드포인트를 해결할 수 있는 검색 서비스를 제공합니다.
- **상태 비저장 및 상태 저장 서비스**. 상태 저장 서비스는 캐시 또는 큐를 대신 사용할 수 있으며 분할될 수 있는 [신뢰할 수 있는 컬렉션][sf-reliable-collections]을 사용합니다.
- **응용 프로그램 수명 주기 관리**. 독립적으로 응용 프로그램의 가동 중지 시간 없이 서비스를 업그레이드할 수 있습니다.
- **서비스 오케스트레이션** - 컴퓨터의 클러스터 전체에 해당합니다.
- **더 높은 밀도** - 리소스 사용을 최적화합니다. 단일 노드는 여러 서비스를 호스팅할 수 있습니다.

Service Fabric은 분산된 클라우드 응용 프로그램을 빌드하기 위한 검증된 플랫폼을 만드는 Azure SQL Database, Cosmos DB, Azure Event Hubs 등을 포함한 다양한 Microsoft 서비스에서 사용됩니다. 

## <a name="comparing-cloud-services-with-service-fabric"></a>Cloud Services와 Service Fabric 비교

다음 표에는 Cloud Services와 Service Fabric 응용 프로그램 간의 중요한 차이점이 몇 가지 요약되어 있습니다. 자세한 내용은 [응용 프로그램을 마이그레이션하기 전에 Cloud Services와 Service Fabric 간의 차이점 알아보기][sf-compare-cloud-services]를 참조하세요.

|        | Cloud Services | Service Fabric |
|--------|---------------|----------------|
| 응용 프로그램 구성 | 역할| Services |
| 밀도 |VM당 하나의 역할 인스턴스 | 단일 노드의 여러 서비스 |
| 최소 노드 수 | 역할당 2개 | 프로덕션 배포의 경우 클러스터당 5개 |
| 상태 관리 | 상태 비저장 | 상태 비저장 또는 상태 저장* |
| Hosting | Azure | 클라우드 또는 온-프레미스 |
| 웹 호스팅 | IIS** | 자체 호스팅 |
| 배포 모델 | [클래식 배포 모델][azure-deployment-models] | [리소스 관리자][azure-deployment-models]  |
| 패키징 | 클라우드 서비스 패키지 파일(.cspkg) | 응용 프로그램 및 서비스 패키지 |
| 응용 프로그램 업데이트 | VIP 스왑 또는 롤링 업데이트 | 롤링 업데이트 |
| 자동 확장 | [기본 제공 서비스][cloud-service-autoscale] | 자동 규모 확장에 대한 VM Scale Sets |
| 디버그 | 로컬 에뮬레이터 | 로컬 클러스터 |


\* 상태 저장 서비스는 [신뢰할 수 있는 컬렉션][sf-reliable-collections]을 사용하여 복제본 간에 상태를 저장하므로 모든 읽기는 클러스터의 노드에 대해 로컬입니다. 쓰기는 안정성을 위해 노드에 걸쳐 복제됩니다. 상태 비저장 서비스는 데이터베이스 또는 다른 외부 저장소를 사용하는 외부 상태를 가질 수 있습니다.

** 작업자 역할은 OWIN을 사용하여 ASP.NET Web API를 자체 호스트팅할 수 있습니다.

## <a name="the-surveys-application-on-cloud-services"></a>Cloud Services에서의 설문 조사 응용 프로그램

다음 다이어그램은 Cloud Services에서 실행되는 설문 조사 응용 프로그램의 아키텍처를 보여 줍니다. 

![](./images/tailspin01.png)

응용 프로그램은 두 개의 웹 역할과 하나의 작업자 역할로 구성됩니다.

- **Tailspin.Web** 웹 역할은 Tailspin 고객이 설문 조사를 만들고 관리하는 데 사용하는 ASP.NET 웹 사이트를 호스트합니다. 또한 고객은 이 웹 사이트를 사용하여 응용 프로그램에 등록하고 해당 구독을 관리합니다. 마지막으로, Tailspin 관리자는 이를 통해 테넌트 목록을 확인하고 테넌트 데이터를 관리할 수 있습니다. 

- **Tailspin.Web.Survey.Public** 웹 역할은 ASP.NET 웹 사이트를 호스트하는데, 여기에서 Tailspin 고객이 게시하는 설문 조사를 사람들이 수행할 수 있습니다. 

- **Tailspin.Workers.Survey** 작업자 역할은 백그라운드 처리를 수행합니다. 웹 역할은 큐에 작업 항목을 배치하며 작업자 역할은 항목을 처리합니다. Azure SQL Database에 대한 설문 조사 답변을 내보내고, 설문 조사 답변에 대한 통계를 계산하는 두 개의 백그라운드 작업이 정의됩니다.

Cloud Services 외에도 설문 조사 응용 프로그램은 다음과 같은 다른 Azure 서비스를 사용합니다.

- **Azure Storage** 설문 조사, 설문 조사 답변 및 테넌트 정보를 저장합니다.

- **Azure Redis Cache** 더 빠른 읽기 권한을 위해 Azure Storage에 저장된 일부 데이터를 캐시합니다. 

- **Azure Active Directory**(Azure AD) 고객과 Tailspin 관리자를 인증합니다.

- **Azure SQL Database** 분석에 대한 설문 조사 답변을 저장합니다. 

## <a name="moving-to-service-fabric"></a>Service Fabric으로 마이그레이션

언급했듯이 이 단계의 목표는 필수 변경 내용을 최소한으로 하면서 Service Fabric으로 마이그레이션하는 것입니다. 이를 위해 원래 응용 프로그램에서 각 클라우드 서비스 역할에 해당하는 상태 비저장 서비스를 만들었습니다.

![](./images/tailspin02.png)

의도적으로, 이 아키텍처는 원래 응용 프로그램과 매우 비슷합니다. 단, 다이어그램에는 몇 가지 중요한 차이점이 숨겨져 있습니다. 이 문서의 나머지 부분에서는 이러한 차이점에 대해 살펴보겠습니다. 


## <a name="converting-the-cloud-service-roles-to-services"></a>클라우드 서비스 역할을 서비스로 변환

설명한 것처럼 각 클라우드 서비스 역할을 Service Fabric 서비스로 마이그레이션했습니다. 클라우드 서비스 역할은 상태 비저장이므로, 이 단계에서는 Service Fabric에 상태 비저장 서비스를 만들어야 합니다. 

마이그레이션의 경우 [웹 및 작업자 역할을 Service Fabric 상태 비저장 서비스로 변환하기 위한 가이드][sf-migration]에 간략히 나온 단계를 수행하였습니다. 

### <a name="creating-the-web-front-end-services"></a>웹 프런트 엔드 서비스 만들기

Service Fabric에서 서비스는 Service Fabric 런타임에서 만든 프로세스 내에서 실행됩니다. 웹 프런트 엔드의 경우에는 서비스가 IIS 내에서 실행되지 않음 의미합니다. 대신, 서비스는 웹 서버를 호스팅해야 합니다. 이 방법은 프로세스 내에서 실행되는 코드가 웹 서버 호스트처럼 작동하므로 *자체 호스팅*이라고 부릅니다. 

자체 호스트에 대한 요구 사항은 그러한 프레임워크에 IIS가 필요하며 자체 호스팅을 지원하지 않으므로 Service Fabric 서비스가 ASP.NET MVC 또는 ASP.NET Web Forms를 사용할 수 없음을 의미합니다. 자체 호스팅에 대한 옵션은 다음과 같습니다.

- [ASP.NET Core][aspnet-core], [Kestrel][kestrel] 웹 서버를 사용하여 자체 호스팅됨. 
- [ASP.NET Web API][aspnet-webapi], [OWIN][owin]을 사용하여 자체 호스팅됨.
- [Nancy](http://nancyfx.org/)와 같은 타사 프레임워크.

ASP.NET MVC를 사용하는 원래 설문 조사 응용 프로그램. ASP.NET MVC는 Service Fabric에서 자체 호스팅될 수 없으므로 다음과 같은 마이그레이션 옵션을 고려했습니다.

- 웹 역할을 자체적으로 호스팅될 수 있는 ASP.NET Core로 포팅합니다.
- 웹 사이트를 ASP.NET Web API를 사용하여 구현된 웹 API를 호출하는 SPA(단일 페이지 응용 프로그램)로 변환합니다. 이를 위해서는 웹 프런트 엔드를 완전히 다시 디자인해야 합니다.
- 기존 ASP.NET MVC 코드를 유지하고 Windows Server 컨테이너의 IIS를 Service Fabric에 배포합니다. 이 방법은 코드가 거의 변경되지 않습니다. 그러나 Service Fabric의 [컨테이너 지원][sf-containers]은 현재 미리 보기 상태입니다.

이러한 고려 사항에 따라, 첫 번째 옵션을 선택하여 ASP.NET Core로 포팅했습니다. 이를 위해 [ASP.NET MVC에서 ASP.NET Core MVC로 마이그레이션][aspnet-migration]에 설명된 단계를 수행하였습니다. 

> [!NOTE]
> Kestrel과 함께 ASP.NET Core를 사용하는 경우 보안상의 이유로 인터넷을 통해 트래픽을 처리하도록 Kestrel 앞에 역방향 프록시를 배치해야 합니다. 자세한 내용은 [ASP.NET Core에서 Kestrel 웹 서버 구현][kestrel]을 참조하세요. [응용 프로그램 배포](#deploying-the-application) 섹션에 권장되는 Azure 배포가 설명되어 있습니다.

### <a name="http-listeners"></a>HTTP 수신기

Cloud Services에서 웹 또는 작업자 역할은 [서비스 정의 파일][cloud-service-endpoints]에서 HTTP 엔드포인트를 선언함으로써 노출합니다. 웹 역할에는 하나 이상의 엔드포인트가 있어야 합니다.

```xml
<!-- Cloud service endpoint -->
<Endpoints>
    <InputEndpoint name="HttpIn" protocol="http" port="80" />
</Endpoints>
```

마찬가지로, Service Fabric 엔드포인트는 서비스 매니페스트에서 선언됩니다. 

```xml
<!-- Service Fabric endpoint -->
<Endpoints>
    <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8002" />
</Endpoints>
```

하지만 클라우드 서비스 역할과 달리 Service Fabric 서비스는 동일한 노드 내에 공동 배치할 수 있습니다. 따라서 모든 서비스는 서로 다른 포트에서 수신해야 합니다. 이 문서의 뒷부분에서는 포트 80 또는 포트 443에 대한 클라이언트 요청이 어떻게 서비스에 대한 올바른 포트로 라우팅되는지에 대해 논의합니다.

서비스는 각 엔드포인트에 대한 수신기를 명시적으로 만들어야 합니다. Service Fabric은 통신 스택에 구애받지 않기 때문입니다. 자세한 내용은 [ASP.NET Core를 사용하여 응용 프로그램에 대한 웹 서비스 프런트 엔드 구축][sf-aspnet-core]을 참조하세요.

## <a name="packaging-and-configuration"></a>패키징 및 구성

 클라우드 서비스에는 다음과 같은 구성 및 패키지 파일이 들어 있습니다.

| 파일 | 설명 |
|------|-------------|
| 서비스 정의(.csdef) | Azure에서 클라우드 서비스를 구성하는 데 사용하는 설정입니다. 역할, 엔드포인트, 시작 작업 및 구성 설정의 이름을 정의합니다. |
| 서비스 구성(.cscfg) | 역할 인스턴스 수, 엔드포인트 포트 번호 및 구성 설정의 값을 포함하는 배포별 설정입니다. 
| 서비스 패키지(.cspkg) | 응용 프로그램 코드와 구성 및 서비스 정의 파일이 포함됩니다.  |

전체 응용 프로그램에 대해 하나의 .csdef 파일이 있습니다. 로컬 테스트 또는 프로덕션과 같은 다양한 환경에서 여러 개의 .cscfg 파일을 가질 수 있습니다. 서비스가 실행 중인 경우 .csdef가 아닌 .cscfg를 업데이트할 수 있습니다. 자세한 내용은 [클라우드 서비스 모델 정의 및 패키지 방법][cloud-service-config]을 참조하세요.

Service Fabric은 서비스 *정의*와 서비스 *설정* 간 비슷한 구분을 갖지만 구조는 더 세분화됩니다. Service Fabric의 구성 모델을 이해하려면 Service Fabric 응용 프로그램을 어떻게 패키징하는지 이해하는 것이 도움이 됩니다. 구조는 다음과 같습니다.

```
Application package
  - Service packages
    - Code package
    - Configuration package
    - Data package (optional)
```

응용 프로그램 패키지란 배포하는 것입니다. 여기에는 하나 이상의 서비스 패키지가 포함됩니다. 서비스 패키지에는 코드, 구성 및 데이터 패키지가 포함됩니다. 코드 패키지에는 서비스의 이진 파일이 포함되고 구성 패키지에는 구성 설정이 포함됩니다. 이 모델을 사용하면 전체 응용 프로그램을 다시 배포하지 않고도 개별 서비스를 업그레이드할 수 있습니다. 또한 코드를 다시 배포하거나 서비스를 다시 시작하지 않고도 구성 설정만 업데이트할 수 있습니다.

Service Fabric 응용 프로그램에는 다음과 같은 구성 파일이 들어 있습니다.

| 파일 | 위치 | 설명 |
|------|----------|-------------|
| ApplicationManifest.xml | 응용 프로그램 패키지 | 응용 프로그램을 구성하는 서비스를 정의합니다. |
| ServiceManifest.xml | 서비스 패키지| 하나 이상의 서비스를 설명합니다. |
| Settings.xml | 구성 패키지 | 서비스 패키지에서 정의되어 있는 서비스에 대한 구성 설정을 포함합니다. |

자세한 내용은 [Service Fabric에서 응용 프로그램 모델링][sf-application-model]을 참조하세요.

여러 환경에 대한 다양한 구성 설정을 지원하려면 [여러 환경에 대한 응용 프로그램 매개 변수 관리][sf-multiple-environments]에 설명된 다음 접근 방식을 사용합니다.

1. 서비스에 대한 Setting.xml 파일에 있는 설정을 정의합니다.
2. 응용 프로그램 매니페스트에서 설정에 대한 재정의를 정의합니다.
3. 환경 관련 설정을 응용 프로그램 매개 변수 파일에 배치합니다.


## <a name="deploying-the-application"></a>응용 프로그램 배포

반면 Azure Cloud Services는 관리되는 서비스이며, Service Fabric은 런타임입니다. Azure 또는 온-프레미스를 포함한 다양한 환경에서 Service Fabric 클러스터를 만들 수 있습니다. 이 문서에서는 Azure에 배포하는 것에 초점을 맞춥니다. 

다음 다이어그램에는 권장되는 배포가 나와 있습니다.

![](./images/tailspin-cluster.png)

Service Fabric 클러스터가 [VM 확장 집합][vm-scale-sets]에 배포되었습니다. 크기 집합은 동일한 VM 집합을 배포하고 관리하는 데 사용할 수 있는 Azure Compute 리소스입니다. 

앞서 언급했듯이 보안상의 이유로 Kestrel 웹 서버에는 역방향 프록시가 필요합니다. 이 다이어그램에는 다양한 계층 7 부하 분산 기능을 제공하는 Azure 서비스인 [Azure Application Gateway][application-gateway]가 나와 있습니다. 이 게이트웨이는 역방향 프록시 서비스 역할을 하며 클라이언트 연결을 종료하고 백 엔드 끝점으로 요청을 전달합니다. nginx와 같은 다양한 역방향 프록시 솔루션을 사용할 수도 있습니다.  

### <a name="layer-7-routing"></a>계층 7 라우팅

[원래 설문 조사 응용 프로그램](https://msdn.microsoft.com/en-us/library/hh534477.aspx#sec21)에서 한 웹 역할은 포트 80에서, 다른 웹 역할은 포트 443에서 수신 대기합니다. 

| 공용 사이트 | 설문 조사 관리 사이트 |
|-------------|------------------------|
| `http://tailspin.cloudapp.net` | `https://tailspin.cloudapp.net` |

다른 옵션은 계층 7 라우팅을 사용하는 것입니다. 이 방법에서는 다양한 URL 경로가 백 엔드에서 다른 포트 번호에 라우팅됩니다. 예를 들어 공용 사이트는 `/public/`으로 시작하는 URL 경로를 사용할 수도 있습니다. 

계층 7 라우팅에 대한 옵션에는 다음이 포함됩니다.

- Application Gateway를 사용합니다. 

- nginx와 같은 NVA(네트워크 가상 어플라이언스)를 사용합니다.

- 사용자 지정 게이트웨이를 상태 비저장 서비스로 작성합니다.

공용 HTTP 엔드포인트를 사용하는 서비스가 둘 이상 있지만, 단일 도메인 이름을 사용하여 하나의 사이트로 표시하려는 경우 이 방법을 고려합니다.

> 외부 클라이언트에서 Service Fabric [역방향 프록시][sf-reverse-proxy]를 통해 요청을 보내도록 허용하는 방법은 권장하지 *않습니다*. 가능한 경우, 역방향 프록시는 서비스 간 통신에 사용되도록 고안되었습니다. 이를 외부 클라이언트로 열면 HTTP 엔드포인트가 있는 클러스터에서 실행되는 *모든* 서비스가 노출됩니다.

### <a name="node-types-and-placement-constraints"></a>노드 형식 및 배치 제약 조건

위에 표시된 배포에서 모든 서비스는 모든 노드에서 실행됩니다. 그러나 서비스를 그룹화하여 특정 서비스가 클러스터 내 특정 노드에 대해서만 실행되도록 할 수도 있습니다. 이 방법을 사용하는 이유는 다음과 같습니다.

- 다른 VM 유형에서 일부 서비스를 실행합니다. 예를 들어 일부 서비스는 계산 집약적이거나 GPU가 필요할 수 있습니다. Service Fabric 클러스터에서 VM 형식 혼합을 사용할 수 있습니다.
- 보안상의 이유로 백 엔드 서비스에서 프런트 엔드 서비스를 격리합니다. 모든 프런트 엔드 서비스는 하나의 노드 집합에서 실행되고 백 엔드 서비스는 동일한 클러스터의 다른 노드에서 실행됩니다.
- 다른 크기 조정 요구 사항입니다. 일부 서비스는 다른 서비스에 비해 더 많은 노드에서 실행해야 할 수도 합니다. 예를 들어 프런트 엔드 노드 및 백 엔드 노드를 정의하는 경우 각 집합은 독립적으로 확장할 수 있습니다.

다음 다이어그램에서는 프런트 엔드 및 백 엔드 서비스를 구분하는 클러스터를 보여 줍니다.

![](././images/node-placement.png)

이 방법을 구현하려면 다음을 수행합니다.

1.  클러스터를 만들 때 두 개 이상의 노드 형식을 정의합니다. 
2.  각 서비스에 대해 [배치 제약 조건][sf-placement-constraints]을 사용하여 서비스를 노드 형식에 할당합니다.

Azure에 배포할 때 각 노드 형식이 별도 VM 확장 집합에 배포됩니다. Service Fabric 클러스터는 모든 노드 형식에 걸쳐 있습니다. 자세한 내용은 [Service Fabric 노드 형식과 Virtual Machine Scale Sets 간의 관계][sf-node-types]를 참조하세요.

> 클러스터에 여러 개 노드 형식이 있는 경우 하나의 노드 형식이 *기본* 노드 형식으로 지정됩니다. 클러스터 관리 서비스와 같은 Service Fabric 런타임 서비스는 기본 노드 형식에서 실행됩니다. 프로덕션 환경에서 기본 노드 형식에 대한 최소 5개 노드를 프로비전합니다. 다른 노드 형식에는 2개 이상의 노드가 있어야 합니다.

## <a name="configuring-and-managing-the-cluster"></a>클러스터 구성 및 관리

권한이 없는 사용자가 연결되지 않도록 클러스터를 보호해야 합니다. Azure AD를 사용하여 클라이언트 및 노드 간 보안을 위한 X.509 인증서를 인증하는 것이 좋습니다. 자세한 내용은 [Service Fabric 클러스터 보안 시나리오][sf-security]를 참조하세요.

공용 HTTPS 엔드포인트를 구성하려면 [서비스 매니페스트에서 리소스 지정][sf-manifest-resources]을 참조하세요.

클러스터에 VM을 추가하여 응용 프로그램을 확장할 수 있습니다. VM 확장 집합은 성능 카운터에 따라 자동 크기 조정 규칙을 사용하는 자동 크기 조정을 지원합니다. 자세한 내용은 [자동 크기 조정 규칙을 사용하여 Service Fabric 클러스터 크기 조정][sf-auto-scale]을 참조하세요.

클러스터를 실행하는 동안 중앙 위치에서 모든 노드의 로그를 수집해야 합니다. 자세한 내용은 [Azure 진단을 사용하여 로그 수집][sf-logs]을 참조하세요.   


## <a name="conclusion"></a>결론

설문 조사 응용 프로그램을 Service Fabric으로 포팅하는 것은 매우 간단합니다. 요약하자면 다음을 수행했습니다.

- 역할을 상태 비저장 서비스로 변환했습니다.
- 웹 프런트 엔드를 ASP.NET Core로 변환했습니다.
- Service Fabric 모델에 대해 패키징 및 구성 파일을 변경했습니다.

또한 배포가 Cloud Services에서 VM 확장 집합에서 실행되는 Service Fabric 클러스터로 변경되었습니다.

## <a name="next-steps"></a>다음 단계

설문 조사 응용 프로그램이 성공적으로 이식되었으므로 Tailspin은 독립적인 서비스 배포 및 버전 관리와 같은 Service Fabric 기능을 활용하려고 합니다. Tailspin에서 이러한 서비스를 더 세부적인 아키텍처로 분해하여 [Azure Cloud Services에서 마이그레이션된 Azure Service Fabric 응용 프로그램을 리팩터링][refactor-surveys]에서 이러한 Service Fabric 기능을 활용하는 방법을 알아봅니다.

<!-- links -->

[application-gateway]: /azure/application-gateway/
[aspnet-core]: /aspnet/core/
[aspnet-webapi]: https://www.asp.net/web-api
[aspnet-migration]: /aspnet/core/migration/mvc
[aspnet-hosting]: /aspnet/core/fundamentals/hosting
[aspnet-webapi]: https://www.asp.net/web-api
[azure-deployment-models]: /azure/azure-resource-manager/resource-manager-deployment-model
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale-portal
[cloud-service-config]: /azure/cloud-services/cloud-services-model-and-package
[cloud-service-endpoints]: /azure/cloud-services/cloud-services-enable-communication-role-instances#worker-roles-vs-web-roles
[kestrel]: https://docs.microsoft.com/aspnet/core/fundamentals/servers/kestrel
[lb-probes]: /azure/load-balancer/load-balancer-custom-probe-overview
[owin]: https://www.asp.net/aspnet/overview/owin-and-katana
[refactor-surveys]: refactor-migrated-app.md
[sample-code]: https://github.com/mspnp/cloud-services-to-service-fabric
[sf-application-model]: /azure/service-fabric/service-fabric-application-model
[sf-aspnet-core]: /azure/service-fabric/service-fabric-add-a-web-frontend
[sf-auto-scale]: /azure/service-fabric/service-fabric-cluster-scale-up-down
[sf-compare-cloud-services]: /azure/service-fabric/service-fabric-cloud-services-migration-differences
[sf-connect-and-communicate]: /azure/service-fabric/service-fabric-connect-and-communicate-with-services
[sf-containers]: /azure/service-fabric/service-fabric-containers-overview
[sf-logs]: /azure/service-fabric/service-fabric-diagnostics-how-to-setup-wad
[sf-manifest-resources]: /azure/service-fabric/service-fabric-service-manifest-resources
[sf-migration]: /azure/service-fabric/service-fabric-cloud-services-migration-worker-role-stateless-service
[sf-multiple-environments]: /azure/service-fabric/service-fabric-manage-multiple-environment-app-configuration
[sf-node-types]: /azure/service-fabric/service-fabric-cluster-nodetypes
[sf-overview]: /azure/service-fabric/service-fabric-overview
[sf-placement-constraints]: /azure/service-fabric/service-fabric-cluster-resource-manager-cluster-description
[sf-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[sf-reliable-services]: /azure/service-fabric/service-fabric-reliable-services-introduction
[sf-reverse-proxy]: /azure/service-fabric/service-fabric-reverseproxy
[sf-security]: /azure/service-fabric/service-fabric-cluster-security
[sf-why-microservices]: /azure/service-fabric/service-fabric-overview-microservices
[tailspin-book]: https://msdn.microsoft.com/en-us/library/ff966499.aspx
[tailspin-scenario]: https://msdn.microsoft.com/en-us/library/hh534482.aspx
[unity]: https://msdn.microsoft.com/en-us/library/ff647202.aspx
[vm-scale-sets]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
