---
title: "마이크로 서비스에서 로깅 및 모니터링"
description: "마이크로 서비스에서 로깅 및 모니터링"
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 1da67047daa9ae87cda5dd7dd581d6081183c428
ms.sourcegitcommit: 786bafefc731245414c3c1510fc21027afe303dc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/12/2017
---
# <a name="designing-microservices-logging-and-monitoring"></a>마이크로 서비스 디자인: 로깅 및 모니터링

모든 복잡한 응용 프로그램의 특정 시점에서 일부는 잘못될 수 있습니다. 마이크로 서비스 응용 프로그램에서는 수십 또는 수백 개의 서비스에서 발생하는 것을 추적해야 합니다. 로깅 및 모니터링은 시스템에 대한 전체적인 뷰를 제공하는 데 매우 중요합니다. 

![](./images/monitoring.png)

마이크로 서비스 아키텍처에서는 오류 또는 성능 병목 상태의 정확한 원인을 찾는 데 특히 어려울 수 있습니다. 단일 사용자 작업에는 여러 서비스가 포함될 수 있습니다. 서비스는 클러스터 내의 네트워크 I/O 제한에 도달할 수 있습니다. 서비스의 호출 체인은 시스템에서 역압을 생성하여 긴 대기 시간 또는 연속적인 오류를 발생시킬 수 있습니다. 또한 일반적으로 특정 컨테이너에서 실행할 노드를 알지 못합니다. 동일한 노드에 배치된 컨테이너는 제한된 CPU 또는 메모리에 대해 경쟁할 수 있습니다. 

상황을 이해하기 위해 응용 프로그램은 원격 분석 이벤트를 내보내야 합니다. 이를 메트릭 및 텍스트 기반 로그로 분류할 수 있습니다. 

*메트릭*은 분석될 수 있는 숫자 값입니다. 실시간으로(또는 실시간에 가까운) 시스템을 관찰하거나 시간에 따른 성능 추세를 분석하기 위해 사용할 수 있습니다. 메트릭은 다음과 같습니다.

- CPU, 메모리, 네트워크, 디스크 및 파일 시스템 사용량을 포함하는 노드 수준의 시스템 메트릭 시스템 메트릭은 클러스터의 각 노드에 대한 리소스 할당을 이해하고 이상값의 문제를 해결할 수 있도록 합니다.
 
- Kubernetes 메트릭 서비스는 컨테이너에서 실행되므로 VM 수준에서 뿐만 아니라 컨테이너 수준에서 메트릭을 수집해야 합니다. Kubernetes에서 cAdvisor(컨테이너 관리자)는 각 컨테이너에서 사용되는 CPU, 메모리, 파일 시스템 및 네트워크 리소스에 대한 통계를 수집하는 에이전트입니다. Kubelet 디먼은 cAdvisor에서 리소스 통계를 수집하고 REST API를 통해 노출합니다.
   
- 응용 프로그램 메트릭 서비스의 동작을 이해하는 데 관련된 모든 메트릭을 포함합니다. 큐에 대기 중인 인바운드 HTTP 요청 수, 요청 대기 시간, 메시지 큐 길이 또는 초당 처리된 트랜잭션의 수를 예로 들 수 있습니다.

- 종속 서비스 메트릭 클러스터 내 서비스는 관리 PaaS 서비스와 같은 클러스터 외부에 있는 외부 서비스를 호출할 수 있습니다. [Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview)를 사용하여 Azure 서비스를 모니터링할 수 있습니다. 타사 서비스는 모든 메트릭을 제공하거나 제공하지 않을 수 있습니다. 제공하지 않는 경우 사용자 고유의 응용 프로그램 메트릭을 사용하여 대기 시간 및 오류 비율에 대한 통계를 추적해야 합니다.

*로그*는 응용 프로그램이 실행되는 동안 발생하는 이벤트의 기록입니다. 응용 프로그램 로그(추적문) 또는 웹 서버 로그 등이 포함됩니다. 로그는 포렌식스 및 근본 원인 분석에 주로 유용합니다. 

## <a name="considerations"></a>고려 사항

[모니터링 및 진단](../best-practices/monitoring.md) 문서는 응용 프로그램 모니터링에 대한 일반적인 모범 사례를 설명합니다. 마이크로 서비스 아키텍처의 컨텍스트에서 고려할 몇 가지 특정 사항은 다음과 같습니다.

**구성 및 관리** 로깅 및 모니터링에 관리 서비스를 사용하거나 클러스터 내 컨테이너로 구성 요소 로깅 및 모니터링을 배포하겠습니까? 이러한 옵션에 대한 자세한 내용은 아래 [기술 옵션](#technology-options) 섹션을 참조하세요.

**수집 속도** 시스템이 원격 분석 이벤트를 수집할 수 있는 처리량이란? 해당 속도를 초과하는 경우 어떻게 되나요? 예를 들어 시스템은 클라이언트를 제한할 수 있습니다. 이 경우 원격 분석 데이터가 손실되거나 데이터를 저해상도 처리할 수 있습니다. 경우에 따라 수집하는 데이터의 양을 줄여 이 문제를 완화할 수 있습니다.

  - 평균 및 표준 편차 등의 통계를 계산하여 메트릭을 집계하고 모니터링 시스템에 해당 통계 데이터를 보냅니다.  

  - 데이터를 저해상도 처리합니다. &mdash; 즉, 이벤트의 일부분만 처리합니다.

  - 모니터링 서비스에 대한 네트워크 호출 수를 줄이기 위해 데이터를 일괄 처리합니다.

**비용** 원격 분석 데이터를 수집하고 저장하는 비용은 특히 대용량에서 높을 수 있습니다. 경우에 따라 응용 프로그램 실행 비용을 초과할 수도 있습니다. 이 경우 위에서 설명한 것처럼 데이터를 집계, 저해상도 처리 또는 일괄 처리하여 원격 분석의 양을 줄여야 합니다. 
        
**데이터 충실도** 메트릭은 얼마나 정확하나요? 평균은 특히 규모에서 이상값을 숨길 수 있습니다. 또한 샘플링 비율이 너무 낮으면 데이터의 변동을 제거할 수 있습니다. 실제로 요청 작업의 상당 부분에 더 많은 시간이 소요되는 경우 모든 요청에 동일한 종단 간 대기 시간이 있는 것처럼 나타날 수 있습니다. 

**대기 시간** 실시간 모니터링 및 경고를 사용하려면 원격 분석 데이터를 신속하게 사용할 수 있어야 합니다. 모니터링 대시보드에 표시되는 데이터는 얼마나 "실시간"인가요? 몇 초 늦나요? 1분 이상인가요?

**저장소** 로그의 경우 클러스터의 임시 저장소에 로그 이벤트를 기록하고 더 영구적인 저장소로 로그 파일을 제공하도록 에이전트를 구성하는 것이 가장 효율적일 수 있습니다.  데이터는 회고 분석에 사용할 수 있도록 결국 장기 저장소로 옮겨야 합니다. 해당 데이터를 저장하는 비용은 중요한 고려 사항이므로 마이크로 서비스 아키텍처에서 많은 양의 원격 분석 데이터를 생성할 수 있습니다. 또한 데이터를 쿼리하는 방법을 고려합니다. 

**대시보드 및 시각화** 클러스터 및 외부 서비스 내에서 모든 서비스에 대한 시스템의 전체적 뷰를 가져오나요? 둘 이상의 위치에 원격 분석 데이터 및 로그를 작성하는 경우 대시보드는 모두를 표시하고 상관 관계를 보여 줄 수 있나요? 모니터링 대시보드는 적어도 다음 정보를 표시해야 합니다.

- 용량 및 증가에 대한 전체 리소스 할당 여기에 컨테이너 수, 파일 시스템 메트릭, 네트워크 및 코어 할당이 포함됩니다.
- 서비스 수준에서 상호 관련된 컨테이너 메트릭
- 컨테이너와 상호 관련된 시스템 메트릭
- 서비스 오류 및 이상값
    

## <a name="distributed-tracing"></a>분산된 추적

언급한 바와 같이 마이크로 서비스에서 한 가지 문제는 서비스 전반에 걸쳐 이벤트 흐름을 파악하는 것입니다. 단일 작업 또는 트랜잭션은 여러 서비스에 대한 호출을 포함할 수 있습니다. 단계의 전체 시퀀스를 다시 구성하기 위해 각 서비스는 해당 작업에 대해 고유 식별자의 역할을 하는 *상관 관계 ID*를 전파해야 합니다. 상관 관계 ID는 서비스 전반에 걸쳐 [분산된 추적](http://microservices.io/patterns/observability/distributed-tracing.html)을 사용합니다.

클라이언트 요청을 수신하는 첫 번째 서비스는 상관 관계 ID를 생성해야 합니다. 서비스에서 다른 서비스에 대한 HTTP 호출을 수행하는 경우 요청 헤더에 상관 관계 ID를 배치합니다. 마찬가지로 서비스가 비동기 메시지를 보내는 경우 메시지에 상관 관계 ID를 배치합니다. 전체 시스템을 통해 흐르도록 다운스트림 서비스는 상관 관계 ID를 계속해서 전파합니다. 또한 응용 프로그램 메트릭 또는 로그 이벤트를 작성하는 모든 코드는 상관 관계 ID를 포함해야 합니다.

서비스 호출이 상호 관련된 경우 완전한 트랜잭션에 대한 종단 간 대기 시간, 초당 성공한 트랜잭션 수 및 실패한 트랜잭션의 백분율과 같은 운영 메트릭을 계산할 수 있습니다. 응용 프로그램 로그에 상관 관계 ID를 포함하여 근본 원인 분석을 수행할 수 있습니다. 작업이 실패하면 동일한 작업의 일부분이었던 모든 서비스 호출에 대한 로그문을 찾을 수 있습니다. 

분산된 추적을 구현하는 경우 몇 가지 고려 사항은 다음과 같습니다.

- 현재 상관 관계 ID에 대한 표준 HTTP 헤더가 없습니다. 팀에서 사용자 지정 헤더 값을 표준화해야 합니다. 프레임워크 로깅/모니터링 또는 서비스 메시의 선택으로 선택을 결정할 수 있습니다.

- 비동기 메시지의 경우 메시징 인프라가 메시지에 메타데이터 추가를 지원하는 경우 메타데이터로 상관 관계 ID를 포함해야 합니다. 그렇지 않은 경우 메시지 스키마의 일부분으로 포함합니다.

- 단일 불투명 식별자 대신 호출자-호출 수신자 관계와 같은 다양한 정보를 포함하는 *상관 관계 컨텍스트*를 보낼 수 있습니다. 

- Azure Application Insights SDK는 자동으로 상관 관계 컨텍스트를 HTTP 헤더로 삽입하고 Application Insights 로그에 상관 관계 ID를 포함합니다. Application Insights에 기본 제공된 상관 관계 기능을 사용하려는 경우 일부 서비스는 사용되는 라이브러리에 따라 상관 관계 헤더를 여전히 명시적으로 전파해야 합니다. 자세한 내용은 [Application Insights에서 원격 분석 상관 관계](/azure/application-insights/application-insights-correlation)를 참조하세요.
   
- 서비스 메시로 Istio 또는 linkerd를 사용하는 경우 이러한 기술은 HTTP 호출이 서비스 메시 프록시를 통해 라우팅될 때 자동으로 상관 관계 헤더를 생성합니다. 서비스는 관련 헤더를 전달해야 합니다. 

    - Istio: [분산된 요청 추적](https://istio-releases.github.io/v0.1/docs/tasks/zipkin-tracing.html)
    
    - linkerd: [컨텍스트 헤더](https://linkerd.io/config/1.3.0/linkerd/index.html#http-headers)
    
- 로그 집계 방법을 고려합니다. 로그에서 상관 관계 ID를 포함하는 방법을 팀 간에 표준화할 수도 있습니다. JSON과 같은 구조화되거나 반구조화된 형식을 사용하고 상관 관계 ID를 보유하도록 공통 필드를 정의합니다.

## <a name="technology-options"></a>기술 옵션

**Application Insights**는 원격 분석 데이터를 수집 및 저장하고 데이터 분석 및 검색을 위한 도구를 제공하는 Azure의 관리 서비스입니다. Application Insights를 사용하려면 응용 프로그램에서 계측 패키지를 설치합니다. 이 패키지는 앱을 모니터링하고 원격 분석 데이터를 Application Insights 서비스로 보냅니다. 호스트 환경에서 원격 분석 데이터를 가져올 수도 있습니다. Application Insights는 기본 제공 상관 관계 및 종속성 추적을 제공합니다. 시스템 메트릭, 응용 프로그램 메트릭 및 Azure 서비스 메트릭을 모두 한 곳에서 추적할 수 있습니다.

Application Insights는 데이터 속도가 최대 한도를 초과하는 경우를 제한합니다. 자세한 내용은 [Application Insights 제한](/azure/azure-subscription-service-limits#application-insights-limits)을 참조하세요. 단일 작업에서 여러 원격 분석 이벤트를 생성할 수 있으므로 응용 프로그램에 큰 용량의 트래픽이 있는 경우 제한될 가능성이 높습니다. 이 문제를 완화하기 위해 원격 분석 트래픽을 줄이도록 샘플링을 수행할 수 있습니다. 이 경우 메트릭은 덜 정확하게 됩니다. 자세한 내용은 [Application Insights의 샘플링](/azure/application-insights/app-insights-sampling)을 참조하세요. 메트릭을 사전 집계하여 &mdash; 즉, 평균 및 표준 편차 등의 통계 값을 계산하고 원시 원격 분석 대신 해당 값을 전송하여 데이터 볼륨을 줄일 수도 있습니다. 다음 블로그 게시물 :[규모에서 Azure 모니터링 및 Analytics](https://blogs.msdn.microsoft.com/azurecat/2017/05/11/azure-monitoring-and-analytics-at-scale/)는 규모에서 Application Insights를 사용하는 방법을 설명합니다.

또한 데이터 양에 따라 요금이 청구되므로 Application Insights에 대한 가격 책정 모델을 이해해야 합니다. 자세한 내용은 [Application Insights에서 가격 책정 및 데이터 볼륨 관리](/azure/application-insights/app-insights-pricing)를 참조하세요. 응용 프로그램에서 많은 양의 원격 분석을 생성하고 데이터의 샘플링 또는 집계를 수행하지 않으려는 경우 Application Insights는 적절한 선택이 아닐 수 있습니다. 

Application Insights가 요구 사항을 충족하지 않는 경우 인기 있는 오픈 소스 기술을 사용하는 몇 가지 제안 방식은 다음과 같습니다.

시스템 및 컨테이너 메트릭의 경우 클러스터에서 실행되는 **Prometheus** 또는 **InfluxDB**와 같은 시계열 데이터베이스에 메트릭을 내보내는 것이 좋습니다.

- InfluxDB는 푸시 기반 시스템입니다. 에이전트는 메트릭을 푸시해야 합니다. kubelet에서 클러스터 전체 메트릭을 수집하고, 데이터를 집계하고, InfluxDB 또는 다른 시계열 저장소 솔루션에 푸시하는 서비스인 [Heapster][heapster]를 사용할 수 있습니다. Azure Container Service는 Heapster를 클러스터 설치의 일부로 배포합니다. 또 다른 옵션은 메트릭을 수집 및 보고하기 위한 에이전트인 [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/)입니다. 

- Prometheus는 풀 기반 시스템입니다. 구성된 위치에서 메트릭을 주기적으로 스크랩합니다. Prometheus는 cAdvisor 또는 kube-상태-메트릭에 의해 생성된 메트릭을 스크랩할 수 있습니다. [kube-상태-메트릭][kube-state-metrics]은 Kubernetes API 서버에서 메트릭을 수집하고 Prometheus(또는 Prometheus 클라이언트 엔드포인트와 호환되는 스크레이퍼)에 사용할 수 있도록 하는 서비스입니다. Heapster는 메트릭을 집계하는 반면 Kubernetes는 메트릭을 생성하고 싱크에 전달하고, kube-상태-메트릭은 자체 메트릭을 생성하고 스크랩에 대한 엔드포인트를 통해 사용할 수 있도록 합니다. 시스템 메트릭의 경우 시스템 메트릭에 대한 Prometheus 내보내기인 [노드 내보내기](https://github.com/prometheus/node_exporter)를 사용합니다. Prometheus는 부동 소수점 데이터를 지원하지만 문자열 데이터를 지원하지 않으므로 로그가 아닌 시스템 메트릭에 적합합니다.

- **Kibana** 또는 **Grafana**와 같은 대시보드 도구를 사용하여 데이터를 시각화 및 모니터링합니다. 대시보드 서비스는 클러스터의 컨테이너 내부에서 실행될 수도 있습니다.

응용 프로그램 로그의 경우 **Fluentd** 및 **Elasticsearch**를 사용하는 것이 좋습니다. Fluentd는 오픈 소스 데이터 수집기이며 Elasticsearch는 검색 엔진 역할에 최적화된 문서 데이터베이스입니다. 이 방법을 사용하여 각 서비스는 로그를 `stdout` 및 `stderr`에 보내고 Kubernetes는 로컬 파일 시스템에 이러한 스트림을 작성합니다. Fluentd는 로그를 수집하고, 경우에 따라 Kubernetes의 추가 메타데이터로 강화하고, 로그를 Elasticsearch에 보냅니다. Kibana, Grafana 또는 유사한 도구를 사용하여 Elasticsearch에 대한 대시보드를 만듭니다. Fluentd는 하나의 Fluentd Pod가 각 노드에 할당되도록 클러스터에서 DaemonSet으로 실행됩니다. kubelet 로그 및 컨테이너 로그를 수집하도록 Fluentd를 구성할 수 있습니다. 높은 볼륨에서 로컬 파일 시스템에 로그를 작성하면 특히 여러 서비스가 동일한 노드에서 실행되는 경우 성능 병목 상태가 될 수 있습니다. 프로덕션 환경에서 디스크 대기 시간 및 파일 시스템 사용률을 모니터링합니다.

로그에 대한 Elasticsearch로 Fluentd를 사용하는 장점 중 하나는 서비스에 추가 라이브러리 종속성이 필요하지 않다는 것입니다. 각 서비스는 `stdout` 및 `stderr`에 바로 작성하고 Fluentd는 Elasticsearch로 로그 내보내기를 처리합니다. 또한 서비스를 작성하는 팀은 로깅 인프라를 구성하는 방법을 이해할 필요가 없습니다. 한 가지 문제는 트래픽을 처리할 수 있도록 확장하도록 프로덕션 배포에 대한 Elasticsearch 클러스터를 구성하는 것입니다. 

다른 옵션은 OMS(Operations Management Suite) Log Analytics에 로그를 보내는 것입니다. [Log Analytics][log-analytics] 서비스는 중앙 리포지토리로 로그 데이터를 수집하고, 응용 프로그램에서 사용하는 다른 Azure 서비스의 데이터를 통합할 수도 있습니다. 자세한 내용은 [Microsoft OMS(Operations Management Suite)를 사용하여 Azure Container Service 클러스터 모니터링][k8s-to-oms]을 참조하세요.

## <a name="example-logging-with-correlation-ids"></a>예제: 상관 관계 ID를 사용하여 로깅

이 챕터에서 설명한 사항 중 일부를 보여 주기 위해 패키지 서비스에서 로깅을 구현하는 방법의 확장된 예제는 다음과 같습니다. 패키지 서비스는 TypeScript에서 작성되었으며 Node.js용 [Koa](http://koajs.com/) 웹 프레임워크를 사용합니다. 여러 개의 Node.js 로깅 라이브러리에서 선택할 수 있습니다. 테스트했을 때 성능 요구 사항을 충족했던 일반적인 로깅 라이브러리인 [Winston](https://github.com/winstonjs/winston)을 선택했습니다.

구현 세부 정보를 캡슐화하기 위해 추상 `ILogger` 인터페이스를 정의했습니다.

```ts
export interface ILogger {
    log(level: string, msg: string, meta?: any)
    debug(msg: string, meta?: any)
    info(msg: string, meta?: any)
    warn(msg: string, meta?: any)
    error(msg: string, meta?: any)
}
```

다음은 Winston 라이브러리를 래핑하는 `ILogger` 구현입니다. 생성자 매개 변수로 상관 관계 ID를 사용하고 모든 로그 메시지에 ID를 삽입합니다. 

```ts
class WinstonLogger implements ILogger {
    constructor(private correlationId: string) {}
    log(level: string, msg: string, payload?: any) {
        var meta : any = {};
        if (payload) { meta.payload = payload };
        if (this.correlationId) { meta.correlationId = this.correlationId }
        winston.log(level, msg, meta)
    }
  
    info(msg: string, payload?: any) {
        this.log('info', msg, payload);
    }
    debug(msg: string, payload?: any) {
        this.log('debug', msg, payload);
    }
    warn(msg: string, payload?: any) {
        this.log('warn', msg, payload);
    }
    error(msg: string, payload?: any) {
        this.log('error', msg, payload);
    }
}
```

패키지 서비스는 HTTP 요청에서 상관 관계 ID를 추출해야 합니다. 예를 들어 linkerd를 사용하는 경우 상관 관계 ID는 `l5d-ctx-trace` 헤더에 있습니다. Koa에서 HTTP 요청은 파이프라인 처리 요청을 통해 전달되는 컨텍스트 개체에 저장됩니다. 컨텍스트에서 상관 관계 ID를 가져오고 로거를 초기화하도록 미들웨어 함수를 정의할 수 있습니다. (Koa의 미들웨어 함수는 단순히 각 요청에 대해 실행되는 함수입니다.)

```ts
export type CorrelationIdFn = (ctx: Context) => string;

export function logger(level: string, getCorrelationId: CorrelationIdFn) {
    winston.configure({ 
        level: level,
        transports: [new (winston.transports.Console)()]
        });
    return async function(ctx: any, next: any) {
        ctx.state.logger = new WinstonLogger(getCorrelationId(ctx));
        await next();
    }
}
```

이 미들웨어는 상관 관계 ID를 가져오도록 호출자 정의 함수, `getCorrelationId`를 호출합니다. 그런 다음 로거의 인스턴스를 만들고 `ctx.state` 내에 스태시합니다. 이는 Koa에서 파이프라인을 통해 정보를 전달하는 데 사용되는 키-값 사전입니다. 

로거 미들웨어는 시작할 때 파이프라인에 추가됩니다.

```ts
app.use(logger(Settings.logLevel(), function (ctx) {
    return ctx.headers[Settings.correlationHeader()];  
}));
```

모든 항목이 구성되면 코드에 로깅문을 추가하기 쉽습니다. 예를 들어 다음은 패키지를 조회하는 메서드입니다. `ILogger.info` 메서드에 대한 두 호출을 실행합니다.

```ts
async getById(ctx: any, next: any) {
  var logger : ILogger = ctx.state.logger;
  var packageId = ctx.params.packageId;
  logger.info('Entering getById, packageId = %s', packageId);

  await next();

  let pkg = await this.repository.findPackage(ctx.params.packageId)

  if (pkg == null) {
    logger.info(`getById: %s not found`, packageId);
    ctx.response.status= 404;
    return;
  }

  ctx.response.status = 200;
  ctx.response.body = this.mapPackageDbToApi(pkg);
}
```

미들웨어 함수에 의해 자동으로 수행되기 때문에 로깅문에서 상관 관계 ID를 포함할 필요가 없습니다. 이는 로깅 코드를 간결하게 하고 개발자가 상관 관계 ID를 포함하는 것을 잊어버릴 가능성을 줄입니다. 또한 모든 로깅문은 추상 `ILogger` 인터페이스를 사용하므로 나중에 로거 구현을 쉽게 교체할 수 있습니다.

> [!div class="nextstepaction"]
> [지속적인 통합 및 업데이트](./ci-cd.md)

<!-- links -->

[app-insights]: /azure/application-insights/app-insights-overview
[heapster]: https://github.com/kubernetes/heapster
[kube-state-metrics]: https://github.com/kubernetes/kube-state-metrics
[k8s-to-oms]: /azure/container-service/kubernetes/container-service-kubernetes-oms
[log-analytics]: /azure/log-analytics/