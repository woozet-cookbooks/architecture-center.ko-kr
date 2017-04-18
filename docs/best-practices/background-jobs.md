---
title: Background jobs guidance
description: Guidance on background tasks that run independently of the user interface.
author: dragon119
ms.service: guidance
ms.topic: article
ms.date: 07/13/2016
ms.author: pnp

pnp.series.title: Best Practices
---
# 백그라운드 작업
[!INCLUDE [header](../_includes/header.md)]

많은 종류의 응용 프로그램에는 사용자 인터페이스(UI)와 무관하게 실행되는 백그라운드 작업이 필요합니다. 그 예로는 일괄 작업, 집약적 처리 작업, 집중 처리 작업, 워크플로와 같은 장기 실행 프로세스가 있습니다. 백그라운드 작업은 사용자 상호 작용 없이 실행될 수 있습니다. 즉, 응용 프로그램은 작업을 시작한 후 계속 사용자의 대화형 요청을 처리할 수 있습니다. 이 작업을 통해 응용 프로그램 UI에서 부하를 최소화하여 가용성을 개선하고 대화형 응답 시간을 개선할 수 있습니다. 

예를 들어, 사용자가 업로드한 이미지의 축소판 그림을 생성하는 데 응용 프로그램이 필요한 경우, 배경 작업으로 이 작업을 수행하고 이 작업이 완료되는 이 축소판 그림을 스토리지에 저장합니다. 이때 사용자는 프로세스가 완료될 때까지 기다리지 않아도 됩니다. 이와 마찬가지로, 주문하는 사용자는 주문을 처리하는 백그라운드 워크플로를 시작하는 동시에 UI는 사용자가 웹 앱을 계속 검색하도록 해줍니다. 백그라운드 작업이 완료되면, 저장된 주문 데이터를 업데이트하고 주문을 확인한 사용자에게 이메일을 보낼 수 있습니다. 

작업을 백그라운드 작업으로 구현할지 여부를 고려할 때 기본 기준은 사용자 상호 작용 없이, 그리고 UI가 작업을 완료할 때까지 기다리지 않고 작업을 실행할 수 있을지 여부입니다. 작업을 완료할 때까지 사용자 또는 UI가 대기하는 작업은 백그라운드 작업으로 적절하지 못할 수도 있습니다. 

## 백그라운드 작업 유형
백그라운드 작업은 일반적으로 다음 작업 유형 중 하나 이상을 포함합니다. 

* 수학 계산이나 구조 모델 분석과 같은 CPU 집약적 작업.
* 일련의 스토리지 트랜잭션 실행이나 파일 인덱싱과 같은 I/O 집약적 작업.
* 매일 밤 데이터 업데이트나 예약된 처리와 같은 일괄 작업.
* 주문 이행이나 서비스와 시스템 프로비저닝과 같은 장기 실행 워크플로.
* 처리를 위해 작업을 보다 안전한 위치로 전달하는 중요한 데이터 처리. 예를 들어, 웹 앱 내에서 중요한 데이터를 처리해야 합니다. 그 대신, [Gatekeeper](http://msdn.microsoft.com/library/dn589793.aspx)와 같은 패턴을 사용하여 보호된 저장소에 액세스할 수 있는 분리된 백그라운드 프로세스에 데이터를 전달할 수 있습니다.

## 트리거
백그라운드 작업은 여러 방식으로 시작할 수 있으며, 다음 카테고리 중 하나에 해당됩니다. 

* [**이벤트 구동 트리거**](#event-driven-triggers). 이 작업은 보통 워크플로에서 사용자가 취하는 조치나 단계 등의 이벤트에 대응하여 시작됩니다.
* [**일정 구동 트리거**](#schedule-driven-triggers). 타이머 기준으로 일정이 되면 작업이 호출됩니다. 이것은 반복 일정 또는 나중을 위해 지정된 일회성 호출일 수 있습니다.

### 이벤트 구동 트리거
이벤트 구동 호출은 트리거를 사용해 백그라운드 작업을 시작합니다. 이벤트 구동 트리거를 사용한 예는 다음과 같습니다. 

* UI 또는 다른 작업은 큐에 메시지를 놓습니다. 이 메시지는 사용자 주문과 같이 발생한 조치에 대한 데이터를 포함합니다. 백그라운드 작업은 이 큐에서 수신 대기하고 새 메시지 도착을 감지합니다. 이 작업은 메시지를 읽고 이 메시지의 데이터를 백그라운드 작업에 대한 입력으로 사용합니다.
* UI 또는 다른 작업은 저장소의 값을 저장하거나 업데이트합니다. 백그라운드 작업은 저장소를 모니터링하고 변경 내용을 감지합니다. 그리고 데이터를 판독하고 이 데이터를 백그라운드 작업에 대한 입력으로 사용합니다.
* UI 또는 다른 작업은 HTTPS URI 또는 웹 서비스로 노출되는 API와 같은 엔드포인트에 요청을 보냅니다. 이 작업에서는 요청의 일부로서 백그라운드 작업을 완료하는 데 필요한 데이터를 전달합니다. 엔드포인트 또는 웹 서비스는 데이터를 입력으로 사용하는 백그라운드 작업을 호출합니다.

이벤트 구동 호출에 적합한 일반 작업의 예로는 이미지 처리, 워크플로, 정보를 원격 서비스에 전송, 이메일 메시지 전송, 다중 테넌트 응용 프로그램에서 신규 사용자 프로비저닝이 있습니다. 

### 일정 구동 트리거
일정 구동 호출은 타이머를 사용해 백그라운드 작업을 시작합니다. 일정 구동 트리거를 사용한 예는 다음과 같습니다. 

* 응용 프로그램 내에서 로컬로 응용 프로그램의 운영 체제의 일부로서 실행되고 있는 타이머가 정기적으로 백그라운드 작업을 호출함.
* 다른 응용 프로그램에서 실행되고 있는 타이머 또는 Azure Scheduler와 같은 타이머 서비스가 API 또는 웹 서비스에 정기적으로 요청을 보냄. API 또는 웹 서비스가 백그라운드 작업을 호출함.
* 별도의 프로세스나 응용 프로그램은 지정된 시간이 지나거나 지정된 시간이 되면 백그라운드 작업을 호출하는 타이머를 시작합니다.

일정 구동 호출에 적합한 작업의 일반적인 예로는 일괄 처리 루틴(최근 동작을 기준으로 사용자 대신 관련 제품 목록 업데이트 등), 루틴 데이터 처리 작업(인덱스 업데이트 또는 누적 결과 생성 등), 일일 보고서의 데이터 분석, 데이터 보존 정리, 데이터 일관성 확인이 있습니다. 

단일 인스턴스로 실행해야 하는 일정 구동 작업을 사용하는 경우, 다음 사항에 주의해야 합니다. 

* 스케줄러(Windows 예약된 작업을 사용한 가상 컴퓨터 등)를 실행하는 계산 인스턴스 크기를 조정하는 경우, 스케줄러에는 여러 인스턴스가 실행됩니다. 이것은 작업 인스턴스를 여러 개 시작할 수 있습니다.
* 스케줄러 이벤트 간 기간보다 오래 작업을 실행하면, 이 스케줄러는 이전 인스턴스를 계속 실행하면서도 다른 작업 인스턴스를 시작할 수 있습니다.

## 결과 반환
백그라운드 작업은 백그라운드 작업을 호출한 프로세스 또는 UI에서 나온 별도의 프로세스 또는 심지어 별도의 위치에서 비동기식으로 실행됩니다. 원칙적으로 백그라운드 작업은 “자체 유도식(fire and forget)” 운영이고 그 실행 진행률은 UI 또는 호출 프로세스에 영향을 주지 않습니다. 즉, 통화 프로세스는 작업이 끝날 때까지 대기하지 않습니다. 따라서 작업이 끝날 때를 자동으로 감지할 수 없습니다. 

백그라운드 작업이 호출 작업과 통신하여 진행률이나 종료를 표시해야 하는 경우에는 이것을 위한 메커니즘을 구현해야 합니다. 다음은 몇 가지 예입니다. 

* 상태 표시기 값을 호출자 작업에 액세스 가능한 저장소에 쓰면, 필요할 때 이 값을 모니터링하거나 확인할 수 있습니다. 백그라운드 작업이 호출자에게 반환해야 하는 다른 데이터를 동일한 저장소에 놓을 수 있습니다.
* UI 또는 호출자가 수신 대기하는 회신 큐를 설정합니다. 백그라운드 작업은 상태와 완료를 나타내는 큐에 메시지를 보낼 수 있습니다. 백그라운드 작업이 호출자에게 반환해야 하는 데이터를 메시지에 배치할 수 있습니다. Azure Service Bus를 사용하는 경우, **ReplyTo** 및 **CorrelationId** 속성을 사용해 이 기능을 구현할 수 있습니다. 자세한 정보는 [서비스 버스에서 조정된 메시징의 상관 관계](http://www.cloudcasts.net/devguide/Default.aspx?id=13029)를 참조하십시오.
* UI 또는 호출자가 상태 정보를 가져오기 위해 액세스할 수 있는 백그라운드 작업에서 API 또는 엔드포인트를 노출시킵니다. 백그라운드 작업이 호출자에게 반환해야 하는 데이터를 메시지에 포함할 수 있습니다.
* API를 통해 백그라운드 작업 호출을 UI 또는 호출자에게 다시 보내 사전 정의된 포인트의 상태나 완료 시 상태를 표시합니다. 이것은 로컬이나 게시하고 구독하는 메커니즘을 통해 발생하는 이벤트를 통해서 실시할 수 있습니다. 백그라운드 작업이 호출자에게 반환해야 하는 데이터는 요청이나 이벤트 페이로드에 포함될 수 있습니다.

## 호스팅 환경
다양한 Azure 플랫폼 서비스를 사용하여 백그라운드 작업을 호스팅할 수 있습니다. 

* [**Azure Web App 및 WebJob**](#azure-web-apps-and-webjobs). WebJob을 사용해 웹 앱의 다양한 스크립트나 실행 프로그램을 기반으로 사용자 지정 작업을 실행할 수 있습니다.
* [**Azure Cloud Services 웹 및 작업자 역할**](#azure-cloud-services-web-and-worker-roles). 백그라운드 작업으로 실행하는 역할 내에서 코드를 쓸 수 있습니다.
* [**Azure Virtual Machines**](#azure-virtual-machines). Windows 서비스를 사용하거나 Windows Task Scheduler를 사용해야 하는 경우, 전용 가상 컴퓨터 내에서 백그라운드 작업을 호스팅하는 것이 일반적입니다.
* [**Azure Batch**](/azure/batch/batch-technical-overview/). 가상 컴퓨터의 관리 컬렉션에서 실행하는 계산 집약적 작업을 예약하고, 사용자의 작업 필요에 맞게 계산 리소스 크기를 자동으로 조정할 수 있는 플랫폼 서비스입니다.

다음 섹션에서는 이 옵션 각각에 대해 보다 자세히 설명하고 적합한 옵션을 선택하는 고려 사항에 대해 다룹니다. 

## Azure Web App 및 WebJob
Azure WebJob을 사용하여 Azure Web App 내에서 사용자 지정 작업을 백그라운드 작업으로 실행할 수 있습니다. WebJob은 연속 프로세스로서 웹 앱의 컨텍스트 내에서 실행됩니다. 또한, WebJob은 저장소 blob 및 메시지 큐의 변경 내용과 같이 Azure Scheduler 또는 외부 요소에서 트리거 이벤트에 대한 반응으로 실행됩니다. 작업을 요청에 따라 시작하고 중지하고, 정상적으로 종료할 수 있습니다. WebJob을 계속 실행하지 못할 경우 WebJob을 자동으로 다시 시작합니다. 다시 시도 및 오류 동작을 구성할 수 있습니다. 

WebJob을 구성할 때: 

* 작업이 이벤트 구동 트리거에 응답하도록 하려면, 해당 작업을 **계속 실행**으로 구성해야 합니다. 스크립트 또는 프로그램은 site/wwwroot/app_data/jobs/continuous라고 명명된 폴더에 저장됩니다.
* 작업이 일정 구동 트리거에 응답하도록 하려면, 해당 작업을 **일정에 따라 실행**으로 구성해야 합니다. 스크립트 또는 프로그램은 site/wwwroot/app_data/jobs/triggered라고 명명된 폴더에 저장됩니다.
* 작업을 구성할 때 **요청 시 실행** 옵션을 선택하면, 시작할 때 동일한 코드를 **일정에 따라 실행** 옵션으로 실행합니다.

Azure WebJob은 웹 앱의 샌드박스 내에서 실행됩니다. 즉, 웹 앱을 통해 연결 문자열과 같은 환경 변수에 액세스하고 정보를 공유할 수 있습니다. 작업은 작업을 실행하고 있는 컴퓨터의 고유 식별자에 액세스할 수 있습니다. **AzureWebJobsStorage**로 명명된 연결 문자열은 응용 프로그램 데이터용 Azure 저장소 큐, blob, 테이블에 액세스하고 메시징과 통신용 서비스 버스에도 액세스할 수 있습니다. **AzureWebJobsDashboard** 로 명명된 연결 문자열은 작업 동작 로그 파일에 액세스할 수 있습니다.

Azure WebJob의 특징은 다음과 같습니다. 

* **보안**: WebJob은 웹 앱의 배포 자격 증명을 통해 보호됩니다.
* **지원 파일 형식**: 명령 스크립트(.cmd), 배치 파일(.bat), PowerShell 스크립트(.ps1), bash shell 스크립트(.sh), PHP 스크립트(.php), Python 스크립트(.py), JavaScript 코드(.js), 실행 프로그램(.exe, .jar 등)을 사용하여 WebJob을 정의할 수 있습니다.
* **배포**: 스크립트 및 실행 파일을 배포하려면 Azure 포털, Visual Studio 또는 [Visual Studio 2013 Update 4](http://www.visualstudio.com/news/vs2013-update4-rc-vs)용 [WebJobsVs](https://marketplace.visualstudio.com/items?itemName=Sayed-Ibrahim-Hashimi.WebJobsVs) 추가 기능(이 옵션을 사용해 만들고 배포할 수 있음), [Azure WebJobs SDK](/azure/app-service-web/websites-dotnet-webjobs-sdk-get-started/) 중 하나를 사용하거나 다음 위치로 직접 복사하면 됩니다.
  * 트리거된 실행: site/wwwroot/app_data/jobs/triggered/{job name}
  * 연속 실행: site/wwwroot/app_data/jobs/continuous/{job name}
* **로깅**: Console.Out은 INFO로 처리됩니다. Console.Error는 ERROR로 처리됩니다. Azure 포털을 사용하여 모니터링 및 진단 정보에 접속할 수 있습니다. 이 사이트에서 직접 로그 파일을 다운로드할 수 있습니다. 로그 파일은 다음 위치에 저장되어 있습니다.
  * 트리거된 실행: Vfs/data/jobs/triggered/jobName
  * 연속 실행: Vfs/data/jobs/continuous/jobName
* **구성**: WebJob을 구성하려면 포털, REST API, PowerShell을 사용하면 됩니다. 구성 파일로 명명된 설정, 작업 스크립트와 동일한 루트 디렉터리의 작업을 사용하여 작업에 대한 구성 정보를 제공할 수 있습니다. 예:
  * { "stopping_wait_time": 60 }
  * { "is_singleton": true }

### 고려 사항
* 기본적으로 WebJob 크기는 웹 앱을 통해 조정됩니다. 그렇지만, **is_singleton** 구성 속성을 **true**로 설정하여 작업을 구성하여 단일 인스턴스에서 실행할 수 있습니다. 단일 인스턴스 WebJob은 재인덱싱, 데이터 분석, 유사한 작업과 같이 크기를 조정하지 않거나 동시 다중 인스턴스로 실행하지 않는 작업에 유용합니다.
* 작업이 웹 앱의 성능에 미치는 영향을 최소화하려면, 새 App Service 플랜에서 빈 Azure 웹 앱 인스턴스를 만들어 오래 실행하거나 리소스를 많이 사용하는 WebJob을 호스팅하는 것이 좋습니다.

### 자세한 정보
* [Azure WebJob 추천 리소스](/azure/app-service-web/websites-webjobs-resources/)에는 유용한 리소스, 다운로드, WebJob용 샘플 등이 나열되어 있습니다.

## Azure Cloud Services 웹 및 작업자 역할
웹 역할이나 별도의 작업자 역할 내에서 백그라운드 작업을 실행할 수 있습니다. 작업자 역할 사용 여부를 결정할 때 확장성과 탄력성 요구사항, 작업 수명, 릴리스 cadence, 보안, 내결함성, 경합, 복잡성, 논리 아키텍처를 고려해야 합니다. 자세한 내용은 [계산 리소스 통합 패턴](http://msdn.microsoft.com/library/dn589778.aspx)을 참조하십시오. 

Cloud Services 역할을 통해 백그라운드 작업을 구현할 수 있는 여러 가지 방법이 있습니다. 

* 역할에서 **RoleEntryPoint** 클래스의 구현을 생성하고 해당 메서드를 사용하여 백그라운드 작업을 실행합니다. 이 작업은 WaIISHost.exe 컨텍스트에서 실행됩니다. 이 작업은 **CloudConfigurationManager** 클래스의 **GetSetting** 메서드를 사용하여 구성 설정을 로드할 수 있습니다. 자세한 내용은 [수명 주기(Cloud Services)](#lifecycle-cloud-services)를 참조하십시오.
* 시작 작업을 사용하여 응용 프로그램이 시작될 때 백그라운드 작업을 실행합니다. 이 작업을 계속 백그라운드에서 강제로 실행하려면, **taskType** 속성을 설정하여 **백그라운드**로 설정합니다(이 작업을 하지 않을 경우 응용 프로그램 시작 프로세스가 중단되고 작업을 마칠 때까지 대기함). 자세한 내용은 [Azure에서 시작 작업 실행](/azure/cloud-services/cloud-services-startup-tasks/)을 참조하십시오.
* WebJobs SDK를 사용하면 시작 작업으로 시작되는 WebJob과 같은 백그라운드 작업을 구현할 수 있습니다. 자세한 내용은 [Azure WebJobs SDK 시작](/azure/app-service-web/websites-dotnet-webjobs-sdk-get-started/)을 참조하십시오.
* 시작 작업을 사용하여 백그라운드 작업 하나 이상을 실행하는 Windows 서비스를 설치할 수 있습니다. **taskType** 속성을 **백그라운드**로 설정하여 백그라운드에서 서비스를 실행해야 합니다. 자세한 내용은 [Azure에서 시작 작업 실행](/azure/cloud-services/cloud-services-startup-tasks/)을 참조하십시오.

### 웹 역할에서 백그라운드 작업 실행
웹 역할에서 백그라운드 작업을 실행하면 추가 역할을 배포하지 않아도 되기 때문에 호스팅 비용이 절감됩니다. 

### 작업자 역할에서 백그라운드 작업 실행
작업자 역할에서 백그라운드 작업을 실행하면 다음과 같은 여러 이점이 있습니다. 

* 각 역할 유형별로 각기 크기 조정을 관리할 수 있습니다. 예를 들어, 현재 부하를 지원하는 데 웹 역할 인스턴스가 더 많이 필요하지만, 백그라운드 작업을 실행하는 작업자 역할 인스턴스는 줄어듭니다. UI 역할에서 백그라운드 작업 계산 인스턴스 크기를 조정하여 호스팅 비용을 절감하는 동시에 적절한 성능을 유지합니다.
* 또한, 웹 역할에서 백그라운드 작업의 처리 오버헤드를 오프로드합니다. UI를 제공하는 웹 역할은 계속 응답할 수 있어 지정된 개수의 사용자 요청을 지원하는 데 필요한 인스턴스 개수를 줄여 줍니다.
* 따라서 관심 사항을 분리할 수 있습니다. 각 역할 유형은 명확하게 정의된 관련 작업 세트를 구현할 수 있습니다. 따라서 각 역할 사이에 코드와 기능의 상호 종속성이 낮기 때문에 코드를 보다 쉽게 설계하고 유지할 수 있습니다.
* 중요한 프로세스와 데이터를 분리하는 데 도움이 될 수 있습니다. 예를 들어, UI를 구현하는 웹 역할은 작업자 역할이 관리하고 제어하는 데이터에 액세스하지 않아도 됩니다. 이것은 보안을 강화하는 데 유용하고, 특히 [Gatekeeper 패턴](http://msdn.microsoft.com/library/dn589793.aspx)과 같은 패턴을 사용할 때 더욱 유용합니다.   

### 고려 사항
Cloud Services 웹과 작업자 역할을 사용하는 경우 백그라운드 작업을 배포하는 방법과 위치를 선택할 때에는 다음 사항을 고려해야 합니다. 

* 기존 웹 역할에서 백그라운드 작업을 호스팅하면 이 작업만을 위한 별도의 작업자 역할을 실행할 때 소요되는 비용을 절약할 수 있습니다. 그렇지만, 처리 및 다른 리소스에 대한 경합이 있으면 응용 프로그램의 성능과 가용성에 영향을 줄 수 있습니다. 별도의 작업자 역할을 사용하면 웹 역할이 장기 실행 또는 리소스를 많이 사용하는 백그라운드 작업으로 인해 영향을 받지 않습니다.
* **RoleEntryPoint** 클래스를 사용하여 백그라운드 작업을 호스팅할 경우, 이 작업을 다른 역할로 쉽게 이동할 수 있습니다. 예를 들어, 웹 역할에서 클래스를 만든 후 작업자 역할에서 이 작업을 실행하기로 결정한 경우 **RoleEntryPoint** 클래스 구현을 작업자 역할로 이동할 수 있습니다.
* 시작 작업은 프로그램이나 스크립트를 실행할 때 사용됩니다. 백그라운드 작업을 실행 파일 프로그램으로 배포하기는 더 어렵고, 종속 어셈블리를 배포해야 하는 경우에는 특히 더 그러합니다. 시작 작업을 사용하면 스크립트를 배포하고 사용하여 백그라운드 작업을 보다 쉽게 배포할 수 있습니다.
* 백그라운드 작업이 실패하도록 만드는 예외는 호스팅되는 방식에 따라 다른 영향을 줍니다.
  * **RoleEntryPoint** 클래스 접근 방식을 사용하는 경우, 작업에 실패하면 역할이 다시 시작되어 작업이 자동으로 다시 시작됩니다. 따라서 응용 프로그램 가용성에 영향을 줄 수 있습니다. 이 상황을 방지하려면, **RoleEntryPoint** 클래스와 모든 백그라운드 작업 내에 확실한 예외 처리를 포함해야 합니다. 코드를 사용해 실패로 끝나는 작업을 다시 시작하고 예외를 발생하여 코드 내의 오류를 정상적으로 복구할 수 없는 경우에만 역할을 다시 시작합니다.
  * 시작 작업을 사용하는 경우, 작업 실행을 관리하고 실패 여부를 확인해야 합니다.
* 시작 작업 관리 및 모니터링은 **RoleEntryPoint** 클래스 접근 방식을 사용하는 것보다 더 어렵습니다. 그렇지만, Azure WebJobs SDK에는 대시보드가 포함되어 있어 시작 작업을 통해 시작하는 WebJob을 보다 쉽게 관리할 수 있습니다.

### 자세한 정보
* [계산 리소스 통합 패턴](http://msdn.microsoft.com/library/dn589778.aspx)
* [Azure WebJobs SDK 시작](/azure/app-service-web/websites-dotnet-webjobs-sdk-get-started/)

## Azure Virtual Machines
백그라운드 작업 구현 방법에 따라 Azure 웹 앱 또는 Cloud Services에 배포되지 않도록 할 수 있지만, 이들 옵션은 편리하지 않을 수도 있습니다. 일반적인 예로는 Windows 서비스, 타사 유틸리티, 실행 프로그램이 있습니다. 다른 예로는 애플리케이션 호스팅과 다른 실행 환경에서 작성된 프로그램이 있습니다. 예를 들어, Windows 또는 .NET 응용 프로그램에서 Unix 또는 Linux 프로그램을 실행하려고 할 수도 있습니다. Azure 가상 컴퓨터에 적합한 다양한 운영 체제 중에서 선택하고, 가상 컴퓨터에서 서비스나 실행 파일을 실행할 수 있습니다. 

가상 컴퓨터를 사용할 때를 선택하려면 [Azure App Services, Cloud Services, Virtual Machines 비교](/azure/app-service-web/choose-web-site-cloud-service-vm/)를 참조하십시오. 가상 컴퓨터 옵션에 대한 자세한 내용은 [Azure용 가상 컴퓨터와 클라우드 서비스 크기](http://msdn.microsoft.com/library/azure/dn197896.aspx)를 참조하십시오. 가상 컴퓨터에서 사용할 수 있는 운영 체제와 기본 제공 이미지에 대한 자세한 내용은 [Azure Virtual Machines 시장](https://azure.microsoft.com/gallery/virtual-machines/)을 참조하십시오. 

별도의 가상 컴퓨터에서 백그라운드 작업을 시작할 수 있도록 다음과 같은 다양한 옵션이 마련되어 있습니다. 

* 작업이 표시되는 엔드포인트에 요청을 보내서 요청 시 응용 프로그램에서 직접 작업을 실행할 수 있습니다. 이 요청은 작업이 필요로 하는 어느 데이터에서든 전달됩니다. 이 엔드포인트는 작업을 호출합니다.
* 선택한 운영 체제에서 지원되는 스케줄러나 타이머를 사용하여 작업을 구성하고 나서 일정에 따라 실행할 수 있습니다. 예를 들어, Windows에서 Windows Task Scheduler를 사용하여 스크립트와 작업을 실행할 수 있습니다. 그렇지 않고, SQL Server가 가상 컴퓨터에 설치된 경우 SQL Server Agent를 사용해 스크립트와 작업을 실행할 수 있습니다.
* Azure Scheduler를 사용하여 작업을 시작하기 위해 작업이 수신 대기하고 있는 큐에 메시지를 추가하거나 작업이 표시되는 API에 요청을 보낼 수 있습니다.

백그라운드 작업 시작 방법에 대한 자세한 정보는 이전 섹션 [트리거](#triggers)를 참조하십시오.    

### 고려 사항
Azure 가상 컴퓨터에 백그라운드 작업을 배포할지 여부를 결정할 때 고려할 사항은 다음과 같습니다. 

* 별도의 Azure 가상 컴퓨터에서 백그라운드 작업을 호스팅하면 시작, 실행, 예약, 리소스 할당을 유연하고 정확하게 제어할 수 있습니다. 그렇지만, 가상 컴퓨터가 백그라운드 작업만을 실행하도록 배포하면 런타임 비용이 증가합니다.
* Azure 포털에서 작업을 모니터링할 시설이 없고 실패한 작업을 자동으로 다시 시작하는 기능이 없습니다. 하지만 [Azure Service Management Cmdlets](http://msdn.microsoft.com/library/azure/dn495240.aspx)를 사용해 가상 컴퓨터의 기본 상태를 모니터링할 수 있습니다. 그렇지만, 계산 노드에서 프로세스와 스레드를 제어할 시설이 없습니다. 일반적으로 가상 컴퓨터를 사용하면 작업 기기 및 가상 컴퓨터의 운영 체제에서 데이터를 수집하는 메커니즘을 구현하는 작업을 추가로 실시해야 합니다. 적합한 한 가지 해결책은 [System Center Management Pack for Azure](http://technet.microsoft.com/library/gg276383.aspx)를 사용하는 것입니다.
* HTTP 엔드포인트를 통해 표시되는 모니터링 프로브 구현을 고려해 볼 수 있습니다. 이 프로브의 코드는 상태 검사를 실시하거나 운영 정보와 통계를 수집하거나 오류 정보를 수집 및 분석하여 관리 응용 프로그램에 반환합니다. 자세한 내용은 [상태 엔드포인트 모니터링 패턴](http://msdn.microsoft.com/library/dn589789.aspx)을 참조하십시오.

### 자세한 정보
* •	Azure의 [Virtual Machines](https://azure.microsoft.com/services/virtual-machines/)
* [Azure Virtual Machines FAQ](/azure/virtual-machines/virtual-machines-linux-classic-faq?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)

## 설계 고려 사항
백그라운드 작업을 설계할 때에는 여러 가지 기본 요소를 고려해야 합니다. 다음 섹션에서는 분할, 충돌 및 조정에 대해 다룹니다. 

## 분할
기존 계산 인스턴스(웹 앱, 웹 역할, 기존 작업자 역할, 가상 컴퓨터 등) 내에서 백그라운드 작업을 포함하기로 결정한 경우에는 이것이 계산 인스턴스와 백그라운드 작업 자체의 품질 속성에 미치는 영향을 고려해야 합니다. 이 요소를 통해 기존 계산 인스턴스로 여러 작업을 같은 작업에 배치할지 또는 이 작업을 별도의 계산 인스턴스로 분리할지 여부를 결정할 수 있습니다. 

* **가용성**: 백그라운드 작업은 응용 프로그램의 다른 부분과 가용성 수준이 동일하지 않아도 되며, 특히 사용자 상호 작용에 직접 관련된 UI와 다른 부분에서는 더욱 그러합니다. 백그라운드 작업은 대기 시간, 연결 재시도 실패 등 운영 큐 상태로 인해 가용성에 영향을 주는 다른 요소 인해 큰 영향을 받지 않습니다. 그렇지만, 큐를 차단하고 전반적으로 응용 프로그램에 영향을 줄 수 있는 요청 백업을 차단할 수 있는 용량이 충분해야 합니다.
* **확장성**: 백그라운드 작업의 확장성은 응용 프로그램의 UI 및 대화형 부분과 다를 수 있습니다. UI 크기 조정은 최고 수요량을 맞추는 동시에 처리되지 않은 백그라운드 작업을 더 적은 계산 인스턴스로 사용량이 적은 시간에 완료할 수 있습니다.
* **탄력성**: 백그라운드 작업만을 호스팅하는 계산 인스턴스 오류가 발생할 때 작업을 다시 사용할 수 있을 때까지 이 작업 요청을 대기 상태에 두거나 지연시켜 응용 프로그램에 전반적으로 치명적인 영향을 주지 않을 수 있습니다. 적절한 간격으로 계산 인스턴스 및 작업을 다시 시작하면, 응용 프로그램의 사용자가 영향을 받지 않을 수 있습니다.
* **보안**: 백그라운드 작업의 보안 요건이나 제한은 응용 프로그램의 UI 또는 다른 부분과 다르기도 합니다. 별도의 계산 인스턴스를 사용하면, 작업에서 다른 보안 환경을 지정할 수 있습니다. 또한, Gatekeeper와 같은 패턴을 사용하여 배경 계산 인스턴스를 UI와 분리하여 보안과 분리 효과를 극대화할 수 있습니다.
* **성능**: 특히 작업 성능 요구사항에 맞도록 백그라운드 작업에 적합한 계산 인스턴스 유형을 선택할 수 있습니다. 즉, 작업에서 UI와 동일한 처리 기능이 필요하지 않을 경우 보다 저렴한 계산 옵션을 사용하고, 추가 용량 및 리소스가 필요할 때 더 큰 인스턴스를 사용하면 됩니다.
* **관리 효율성**: 백그라운드 작업은 개발이나 배포 리듬이 기본 응용 프로그램 코드나 UI와 다를 수 있습니다. 이것을 별도의 계산 인스턴스에 배포하면 업데이트와 버전 관리가 간편해집니다.
* **비용**: 백그라운드 작업을 실행하기 위해 계산 인스턴스를 추가하면 호스팅 비용이 늘어납니다. 용량 추가와 추가 비용 간의 상관 관계를 잘 고려해야 합니다.

자세한 내용은 [리더 선택 패턴](http://msdn.microsoft.com/library/dn568104.aspx) 및 [경쟁 소비자 패턴](http://msdn.microsoft.com/library/dn568101.aspx)을 참조하십시오. 

## 충돌
백그라운드 작업 인스턴스가 여러 개 있으면, 이들은 데이터베이스 및 저장소와 같은 리소스에 액세스하기 위해 경쟁할 수 있습니다. 이 동시 액세스로 인해 리소스 경합이 발생하여 서비스 가용성과 저장소의 데이터 무결성에 충돌이 발생할 수 있습니다. 가장 약한 잠금 방법을 사용하여 리소스 경합을 해결할 수 있습니다. 이 조치를 통해 경합하는 작업 인스턴스가 서비스에 동시 액세스하거나 데이터를 손상시키지 못합니다. 

충돌을 해결하는 또 다른 방법은 백그라운드 작업을 하나의 개체로 정의하여 인스턴스 하나만이 실행되도록 하는 것입니다. 그렇지만, 이 조치는 여러 인스턴스 구성이 제공할 수 있는 안정성 및 성능에 따른 이점을 없애줍니다. 이것은 특히 UI가 백그라운드 작업을 2개 이상 계속 사용할 정도로 충분한 작업을 공급할 수 있는 경우에 그러합니다. 

따라서 백그라운드 작업을 자동으로 다시 시작할 수 있고 최고 수요를 처리할 수 있는 충분한 용량이 마련되어 있어야 합니다. 이를 위해서는 충분한 리소스로 계산 인스턴스를 할당하거나, 수요가 감소할 때 나중에 실행할 수 있도록 요청을 저장하는 큐 메커니즘을 구현하거나, 이 방법을 모두 사용할 수 있습니다. 

## 조정
백그라운드 작업에서는 개별 작업을 실행하여 결과를 도출하거나 모든 요구사항에 부합해야 합니다. 이 시나리오에서는 일반적으로 여러 소비자가 실행할 수 있는 더 작은 단계 또는 하위 작업으로 작업을 나눕니다. 다단계 작업은 개별 단계를 여러 작업에서 다시 사용할 수 있기 때문에 효율과 유연성이 더 우수합니다. 또한 단계 순서를 쉽게 추가, 제거, 수정할 수도 있습니다. 

여러 작업과 단계를 조정하는 작업은 어려울 수 있지만, 솔루션 구현 시 유용하게 사용할 수 있는 일반적인 패턴에는 세 가지가 있습니다. 

* **작업을 재사용 가능한 여러 단계로 분해**. 응용 프로그램은 처리하는 정보의 복잡성이 각기 다른 작업을 여러 개 실행하기도 합니다. 이 응용 프로그램을 구현하는 간단하지만 유연하지 못한 방법은 모놀리식 모듈로 처리하는 것입니다. 하지만, 이 방법은 동일한 처리 부분이 응용 프로그램의 다른 부분에서 필요할 경우 코드의 재고려, 최적화, 재사용 기회를 줄일 수 있습니다. 자세한 내용은 [파이프 및 필터 패턴](http://msdn.microsoft.com/library/dn568100.aspx)을 참조하십시오.
* **작업 단계 실행 관리**. 응용 프로그램은 여러 단계로 이뤄진 작업(그 중 일부는 원격 서비스를 호출하거나 원격 리소스를 액세스할 수 있음)을 실행할 수 있습니다. 이 개별 단계는 서로 무관할 수 있지만, 작업을 구현하는 응용 프로그램 논리를 통해 조정됩니다. 자세한 내용은 [Scheduler Agent Supervisor 패턴](http://msdn.microsoft.com/library/dn589780.aspx)을 참조하십시오.
* **실패한 작업 단계의 복구 관리**. 응용 프로그램에서 하나 이상의 단계에 실패한 경우 여러 단계를 거쳐 실시하는 작업을 취소해야 하는 경우도 있습니다(이 모든 단계는 결국 일관된 운영을 규정함). 자세한 내용은 [트랜잭션 패턴 보상](http://msdn.microsoft.com/library/dn589804.aspx)을 참조하십시오.

## 수명 주기(클라우드 서비스)
 **RoleEntryPoint** 클래스를 사용하여 웹과 작업자 역할을 사용하는 Cloud Services 응용 프로그램을 위한 백그라운드 작업을 구현하기로 결정한 경우, 이 클래스를 올바르게 사용하려면 이 클래스의 수명 주기를 이해해야 합니다. 

웹 및 작업자 역할은 시작하고, 실행하고, 중지할 때 고유한 단계를 거칩니다. **RoleEntryPoint** 클래스는 이 단계가 발생할 때를 나타내는 일련의 이벤트를 표시합니다. 이 이벤트를 사용해 사용자 지정 백그라운드 작업을 초기화하고, 실행하고, 중지합니다. 전체 주기는 다음과 같습니다.

* Azure가 역할 어셈블리를 로드하고 **RoleEntryPoint** 에서 파생된 클래스를 검색합니다.
* 이 클래스를 찾을 경우, **RoleEntryPoint.OnStart()**를 호출합니다. 이 메서드를 재정의하여 백그라운드 작업을 초기화합니다.
* **OnStart** 메서드를 완료한 후 Azure는 응용 프로그램의 Global 파일(있는 경우)에서 **Application_Start()** 를 호출합니다(예: ASP.NET를 실행하고 있는 웹 역할에서 Global.asax).
* Azure는 **OnStart()** 와 함께 실행되는 새로운 전경 스레드에서 **RoleEntryPoint.Run()** 을 호출합니다. 이 메서드를 재정의하여 백그라운드 작업을 시작합니다.
* 이 Run 메서드가 끝나면, Azure는 먼저 응용 프로그램의 Global 파일(있는 경우)에서 **Application_End()** 를 호출하고 나서 **RoleEntryPoint.OnStop()** 을 호출합니다. **OnStop** 메서드를 재정의하여 백그라운드 작업을 중지하고, 리소스를 정리하고, 개체를 삭제하고, 작업이 사용했을 수 있는 연결을 종료합니다.
* Azure 작업자 역할 호스트 프로세스가 중지됩니다. 이 시점에서 역할이 재생되어 다시 시작됩니다.

**RoleEntryPoint** 클래스에 대한 자세한 내용과 예는 [계산 리소스 통합 패턴](http://msdn.microsoft.com/library/dn589778.aspx)을 참조하십시오. 

## 고려 사항
웹이나 작업자 역할에서 백그라운드 작업을 실행하는 방법을 계획할 때에는 다음 사항을 고려해야 합니다. 

* **RoleEntryPoint** 클래스에서 기본 **Run** 메서드 구현에는 역할을 무기한으로 유지해 주는 **Thread.Sleep(Timeout.Infinite)** 호출이 포함됩니다. **Run** 메서드(일반적으로 백그라운드 작업을 실행하는 데 필요함)을 재정의하는 경우, 역할 인스턴스를 재생할 필요가 없으면 코드가 이 메서드에서 끝나지 않도록 해야 합니다.
* **Run** 메서드의 일반적인 구현에는 배경 작업 각각을 시작하기 위한 코드와 모든 백그라운드 작업 상태를 정기적으로 확인하는 루프 구성체가 포함됩니다. 모든 오류를 다시 시작하거나 이 작업이 완료되었음을 나타내는 취소 토큰을 모니터링할 수 있습니다.
* 백그라운드 작업에서 처리되지 않은 예외가 발생하면, 역할에서 다른 백그라운드 작업을 계속 실행하는 동안 이 작업을 재생해야 합니다. 그렇지만, 예외 발생 원인이 작업 이외의 개체 손상인 경우, **RoleEntryPoint** 클래스를 통해 이 예외를 처리하고, 모든 작업을 취소하고 **Run** 메서드를 종료할 수 있어야 합니다. 그리고 나면 Azure는 역할을 다시 시작합니다.
* **OnStop** 메서드를 사용하여 백그라운드 작업을 일시 중지하거나 종료하고 리소스를 정리합니다. 여기에는 장기 실행 작업이나 다단계 작업 중지가 포함될 수 있습니다. 데이터 불일치를 방지하기 위해 이 작업을 실시하는 방법을 반드시 고려해야 합니다. 역할 인스턴스가 사용자가 시작한 종료 이외에 어떤 이유로든 중지되는 경우, 강제로 종료되기 전에 **OnStop** 메서드에서 실행되고 있는 코드를 5분 내에 완료해야 합니다. 코드를 이 시간에 내에 완료하거나, 아니면 실행이 완료되지 않도록 허용할 수 있습니다. 
* **RoleEntryPoint.OnStart** 메서드가 값을 **true**로 반환하면 Azure 부하 분산 장치는 트래픽을 역할 인스턴스로 유도하기 시작합니다. 그러므로, 초기화하지 못한 역할 인스턴스가 어느 트래픽도 수신하지 못하도록 모든 초기화 코드를 **OnStart** 메서드로 배치하는 것이 좋습니다.
* **RoleEntryPoint** 클래스의 메서드 외에 시작 작업을 사용할 수 있습니다. 시작 작업은 역할이 요청을 받기 전에 실행되기 때문에 시작 작업을 사용하여 Azure 부하 분산 장치에서 변경해야 하는 모든 설정을 초기화해야 합니다. 자세한 내용은 [Azure에서 시작 작업 실행](/azure/cloud-services/cloud-services-startup-tasks/)을 참조하십시오.
* 시작 작업에 오류가 있으면, 역할을 강제로 계속 다시 시작할 수 있습니다. 따라서 전환에는 역할에 대한 독점 액세스 권한이 필요하기 때문에 가상 IP(VIP) 주소가 이전에 준비된 버전으로 다시 전환되지 않습니다. 역할을 다시 시작하는 동안 이 작업을 실시할 수 없습니다. 이 문제를 해결하는 방법:
  
  * 사용자 역할에서 **OnStart** 및 **Run** 메서드의 시작에 다음 코드를 추가합니다.
    
    ```C#
    var freeze = CloudConfigurationManager.GetSetting("Freeze");
    if (freeze != null)
    {
     if (Boolean.Parse(freeze))
       {
         Thread.Sleep(System.Threading.Timeout.Infinite);
     }
    }
    ```
    
    * Add the definition of the **Freeze** setting as a Boolean value to the ServiceDefinition.csdef and ServiceConfiguration.*.cscfg files for the role and set it to **false**. If the role goes into a repeated restart mode, you can change the setting to **true** to freeze role execution and allow it to be swapped with a previous version.

## Resiliency considerations
Background tasks must be resilient in order to provide reliable services to the application. When you are planning and designing background tasks, consider the following points:

* Background tasks must be able to gracefully handle role or service restarts without corrupting data or introducing inconsistency into the application. For long-running or multistep tasks, consider using *check pointing* by saving the state of jobs in persistent storage, or as messages in a queue if this is appropriate. For example, you can persist state information in a message in a queue and incrementally update this state information with the task progress so that the task can be processed from the last known good checkpoint--instead of restarting from the beginning. When using Azure Service Bus queues, you can use message sessions to enable the same scenario. Sessions allow you to save and retrieve the application processing state by using the [SetState](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.messagesession.setstate.aspx) and [GetState](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.messagesession.getstate.aspx) methods. For more information about designing reliable multistep processes and workflows, see [Scheduler Agent Supervisor Pattern](http://msdn.microsoft.com/library/dn589780.aspx).
* When you use web or worker roles to host multiple background tasks, design your override of the **Run** method to monitor for failed or stalled tasks, and restart them. Where this is not practical, and you are using a worker role, force the worker role to restart by exiting from the **Run** method.
* When you use queues to communicate with background tasks, the queues can act as a buffer to store requests that are sent to the tasks while the application is under higher than usual load. This allows the tasks to catch up with the UI during less busy periods. It also means that recycling the role will not block the UI. For more information, see [Queue-Based Load Leveling Pattern](http://msdn.microsoft.com/library/dn589783.aspx). If some tasks are more important than others, consider implementing the [Priority Queue Pattern](http://msdn.microsoft.com/library/dn589794.aspx) to ensure that these tasks run before less important ones.
* Background tasks that are initiated by messages or process messages must be designed to handle inconsistencies, such as messages arriving out of order, messages that repeatedly cause an error (often referred to as *poison messages*), and messages that are delivered more than once. Consider the following:
  * Messages that must be processed in a specific order, such as those that change data based on the existing data value (for example, adding a value to an existing value), might not arrive in the original order in which they were sent. Alternatively, they might be handled by different instances of a background task in a different order due to varying loads on each instance. Messages that must be processed in a specific order should include a sequence number, key, or some other indicator that background tasks can use to ensure that they are processed in the correct order. If you are using Azure Service Bus, you can use message sessions to guarantee the order of delivery. However, it is usually more efficient, where possible, to design the process so that the message order is not important.
  * Typically, a background task will peek at messages in the queue, which temporarily hides them from other message consumers. Then it deletes the messages after they have been successfully processed. If a background task fails when processing a message, that message will reappear on the queue after the peek time-out expires. It will be processed by another instance of the task or during the next processing cycle of this instance. If the message consistently causes an error in the consumer, it will block the task, the queue, and eventually the application itself when the queue becomes full. Therefore, it is vital to detect and remove poison messages from the queue. If you are using Azure Service Bus, messages that cause an error can be moved automatically or manually to an associated dead letter queue.
  * Queues are guaranteed at *least once* delivery mechanisms, but they might deliver the same message more than once. In addition, if a background task fails after processing a message but before deleting it from the queue, the message will become available for processing again. Background tasks should be idempotent, which means that processing the same message more than once does not cause an error or inconsistency in the application’s data. Some operations are naturally idempotent, such as setting a stored value to a specific new value. However, operations such as adding a value to an existing stored value without checking that the stored value is still the same as when the message was originally sent will cause inconsistencies. Azure Service Bus queues can be configured to automatically remove duplicated messages.
  * Some messaging systems, such as Azure storage queues and Azure Service Bus queues, support a de-queue count property that indicates the number of times a message has been read from the queue. This can be useful in handling repeated and poison messages. For more information, see [Asynchronous Messaging Primer](http://msdn.microsoft.com/library/dn589781.aspx) and [Idempotency Patterns](http://blog.jonathanoliver.com/2010/04/idempotency-patterns/).

## Scaling and performance considerations
Background tasks must offer sufficient performance to ensure they do not block the application, or cause inconsistencies due to delayed operation when the system is under load. Typically, performance is improved by scaling the compute instances that host the background tasks. When you are planning and designing background tasks, consider the following points around scalability and performance:

* Azure supports autoscaling (both scaling out and scaling back in) based on current demand and load--or on a predefined schedule, for Web Apps, Cloud Services web and worker roles, and Virtual Machines hosted deployments. Use this feature to ensure that the application as a whole has sufficient performance capabilities while minimizing runtime costs.
* Where background tasks have a different performance capability from the other parts of a Cloud Services application (for example, the UI or components such as the data access layer), hosting the background tasks together in a separate worker role allows the UI and background task roles to scale independently to manage the load. If multiple background tasks have significantly different performance capabilities from each other, consider dividing them into separate worker roles and scaling each role type independently. However, note that this might increase runtime costs compared to combining all the tasks into fewer roles.
* Simply scaling the roles might not be sufficient to prevent loss of performance under load. You might also need to scale storage queues and other resources to prevent a single point of the overall processing chain from becoming a bottleneck. Also, consider other limitations, such as the maximum throughput of storage and other services that the application and the background tasks rely on.
* Background tasks must be designed for scaling. For example, they must be able to dynamically detect the number of storage queues in use in order to listen on or send messages to the appropriate queue.
* By default, WebJobs scale with their associated Azure Web Apps instance. However, if you want a WebJob to run as only a single instance, you can create a Settings.job file that contains the JSON data **{ "is_singleton": true }**. This forces Azure to only run one instance of the WebJob, even if there are multiple instances of the associated web app. This can be a useful technique for scheduled jobs that must run as only a single instance.

## Related patterns
* [Asynchronous Messaging Primer](http://msdn.microsoft.com/library/dn589781.aspx)
* [Autoscaling Guidance](http://msdn.microsoft.com/library/dn589774.aspx)
* [Compensating Transaction Pattern](http://msdn.microsoft.com/library/dn589804.aspx)
* [Competing Consumers Pattern](http://msdn.microsoft.com/library/dn568101.aspx)
* [Compute Partitioning Guidance](http://msdn.microsoft.com/library/dn589773.aspx)
* [Compute Resource Consolidation Pattern](http://msdn.microsoft.com/library/dn589778.aspx)
* [Gatekeeper Pattern](http://msdn.microsoft.com/library/dn589793.aspx)
* [Leader Election Pattern](http://msdn.microsoft.com/library/dn568104.aspx)
* [Pipes and Filters Pattern](http://msdn.microsoft.com/library/dn568100.aspx)
* [Priority Queue Pattern](http://msdn.microsoft.com/library/dn589794.aspx)
* [Queue-based Load Leveling Pattern](http://msdn.microsoft.com/library/dn589783.aspx)
* [Scheduler Agent Supervisor Pattern](http://msdn.microsoft.com/library/dn589780.aspx)

## More information
* [Scaling Azure Applications with Worker Roles](http://msdn.microsoft.com/library/hh534484.aspx#sec8)
* [Executing Background Tasks](http://msdn.microsoft.com/library/ff803365.aspx)
* [Azure Role Startup Life Cycle](http://blog.syntaxc4.net/post/2011/04/13/windows-azure-role-startup-life-cycle.aspx) (blog post)
* [Azure Cloud Services Role Lifecycle](http://channel9.msdn.com/Series/Windows-Azure-Cloud-Services-Tutorials/Windows-Azure-Cloud-Services-Role-Lifecycle) (video)
* [Get Started with the Azure WebJobs SDK](/azure/app-service-web/websites-dotnet-webjobs-sdk-get-started/)
* [Azure Queues and Service Bus Queues - Compared and Contrasted](/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted/)
* [How to Enable Diagnostics in a Cloud Service](/azure/cloud-services/cloud-services-dotnet-diagnostics/)

