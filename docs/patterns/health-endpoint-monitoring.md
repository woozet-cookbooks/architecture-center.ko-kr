---
title: Health Endpoint Monitoring
description: Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.
keywords: design pattern
author: dragon119
ms.service: guidance
ms.topic: article
ms.author: pnp
ms.date: 03/24/2017

pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: [availability, management-monitoring, resiliency]
---

# 상태 끝점 모니터링

[!INCLUDE [header](../_includes/header.md)]

응용 프로그램에 기능 검사를 구현해 외부 도구가 일정한 간격으로 노출된 끝점을 통해 액세스할 수 있도록 지원합니다. 이런 구현은 응용 프로그램과 서비스가 올바르게 수행되고 있다는 것을 확인하는 데 도움을 줄 수 있습니다.

## 배경 및 문제

웹 응용 프로그램과 백 엔드 서비스를 모니터링하여 사용 가능하고 올바르게 수행되고 있다는 것을 확인하는 것은 좋은 방법일 뿐 아니라 비즈니스 요구 사항인 경우가 많습니다. 그러나 클라우드에서 실행 중인 서비스의 모니터링은 온-프레미스 서비스의 모니터링보다 더 어려운데, 호스팅 환경을 완전히 제어할 수 없는 데다가 보통 플랫폼 공급업체와 다른 업체가 제공하는 서로 다른 서비스에 따라 서비스가 달라지기 때문입니다.

네트워크 대기 시간, 기본 계산과 저장소 시스템의 성능 및 가용성, 기본 계산과 저장소 시스템 사이의 네트워크 대역폭과 같이 클라우드 호스팅 응용 프로그램에 영향을 주는 요인이 다양한데, 이런 요인으로 인해 서비스가 완전히 또는 부분적으로 실패할 수 있습니다. 따라서 서비스가 올바르게 수행되고 있다는 것을 일정한 간격으로 확인해 SLA(서비스 수준 계약)에 포함될 수 있는 필요한 수준의 가용성을 보장해야 합니다.

## 해결책

요청을 응용 프로그램의 끝점에 전송해 상태 모니터링을 구현합니다. 응용 프로그램은 필요한 검사를 수행하고 상태의 표시를 반환해야 합니다.

상태 모니터링 검사는 대개 다음의 두 요인을 조합합니다.

- 상태 확인 끝점에 전송된 요청에 대한 응답으로 응용 프로그램 또는 서비스가 수행하는 검사(해당하는 경우)
- 상태 확인 검사를 수행하는 도구 또는 프레임워크의 결과 분석

응답 코드는 응용 프로그램 및 선택적으로 사용하는 구성 요소 또는 서비스의 상태를 표시합니다. 대기 시간 또는 응답 시간 검사는 모니터링 도구 또는 프레임워크가 수행합니다. 다음 그림은 이 패턴의 개요를 보여줍니다.

![Overview of the pattern](./_images/health-endpoint-monitoring-pattern.png)

다음은 응용 프로그램에서 상태 모니터링 코드로 수행할 수 있는 다른 검사입니다.
- 클라우드 저장소 또는 데이터베이스의 가용성과 응답 시간 검사
- 응용 프로그램 내에 있거나 다른 위치에 있지만 응용 프로그램이 사용할 수 있는 다른 리소스 또는 서비스의 검사

서비스와 도구는 요청을 끝점의 구성 가능한 집합에 제출하고 구성 가능한 규칙의 집합을 기준으로 결과를 평가해 웹 응용 프로그램을 모니터링하는 데 사용할 수 있습니다. 시스템에서 일부 기능 테스트를 수행하는 목적만 가진 서비스 끝점을 생성하는 것은 비교적 쉽습니다.

다음은 모니터링 도구로 수행할 수 있는 대표적인 검사입니다.

- 응답 코드의 유효성 검사. 예를 들면 200 (OK)의 HTTP 응답은 응용 프로그램이 오류 없이 응답했다는 것을 의미합니다. 모니터링 시스템은 더 포괄적인 결과를 제공하기 위해 다른 응답 코드도 검사할 수 있습니다.
- 200 (OK) 상태 코드를 반환하더라도 오류를 검색하기 위해 응답의 내용을 검사. 이런 검사는 반환된 웹 페이지 또는 서비스 응답의 섹션에만 영향을 주는 오류를 검색할 수 있습니다. 그 사례로는 페이지의 제목 검사 또는 정확한 페이지가 반환되었다는 것을 의미하는 특정 구문의 찾기를 꼽을 수 있습니다.
- 네트워크 대기 시간과 응용 프로그램이 요청을 실행하는 데 걸린 시간의 조합을 표시하는 응답 시간의 측정. 값이 증가하면 응용 프로그램 또는 네트워크에 문제가 발생했다는 것을 의미할 수 있습니다.
- 전역 캐시의 콘텐츠를 배달하기 위해 응용 프로그램이 사용하는 콘텐츠 배달 네트워크와 같이 응용 프로그램을 벗어나 위치하는 리소스 또는 서비스의 검사
- SSL 인증서의 만료 검사
- DNS 대기 시간과 DNS 오류를 측정하기 위해 응용 프로그램의 URL에 대한 DNS 조회의 응답 시간을 측정
- 정확한 항목인지를 확인하기 위해 DNS 조회가 반환한 URL의 유효성 검사. 이런 유효성 검사는 DNS 서버에서 공격 성공을 통한 악의적인 요청 리디렉션을 방지하는 데 도움을 줄 수 있습니다.

가능한 경우 응답 시간을 측정하고 비교하기 위해 다른 온-프레미스 또는 호스팅된 위치에서 이런 검사를 실행하는 것도 유용합니다. 사용자는 각 위치에서 성능의 정확한 보기를 얻기 위해 고객과 가까운 위치에서 응용 프로그램을 모니터링해야 합니다. 결과는 더 강력한 검사 방식을 제공할 뿐 아니라응용 프로그램의 배포 위치 및 응용 프로그램을 하나 이상의 데이터 센터에 배포할지 여부를 결정하는 데 도움을 줄 수 있습니다.

또한 테스트도 응용 프로그램이 모든 고객을 대상으로 올바르게 작동하고 있는지를 확인하기 위해 고객이 사용하는 모든 서비스 인스턴스에 대해 테스트를 실시해야 합니다. 예를 들어 고객 저장소가 하나 이상의 저장소 계정에 연결되어 있으면 모니터링 프로세스는 모든 계정을 검사해야 합니다.

## 문제 및 고려 사항

이 패턴의 구현 방법을 결정할 때는 다음 사항을 고려해야 합니다.

응답의 유효성을 검사하는 방법. 예를 들어 하나의 200 (OK) 상태 코드만으로 응용 프로그램이 올바르게 작동하고 있다는 것을 충분히 확인할 수 있습니까? 이 방법은 응용 프로그램 가용성에 대한 가장 기본적인 척도를 제공하고 이 패턴의 최소 구현에 해당하지만 작업, 추세 및 응용 프로그램에서 발생할 수 있는 문제에 대한 정보를 거의 제공하지 않습니다.

   >  응용 프로그램이 대상 리소스를 발견하고 처리한 경우에만 200 (OK)을 올바르게 반환하는지 확인합니다. 마스터 페이지를 사용해 대상 웹 페이지를 호스팅하는 경우와 같은 일부 시나리오에서는 서버가 대상 콘텐츠 페이지를 찾지 못했더라도 404 (Not Found) 코드 대신 200 (OK) 상태를 다시 전송하기 때문입니다.

응용 프로그램에 노출시키는 끝점의 개수. 한 가지 접근 방식은 각각의 응용 프로그램이 사용하는 핵심 서비스를 위해 하나 이상의 끝점을 노출시키고 우선 순위가 낮은 서비스를 위해 다른 끝점을 노출시켜 모니터링 결과에 다양한 수준의 중요성을 할당하는 것입니다. 또한 추가 모니터링 세분성을 위해 핵심 서비스마다 끝점을 추가로 노출시키는 것도 고려해야 합니다. 예를 들면 상태 확인 검사는 다른 수준의 작동 시간과 응답 시간을 요구하여 응용 프로그램이 사용하는 데이터베이스, 저장소 및 외부 지오코딩 서비스를 검사할 수 있는데, 지오코딩 서비스 또는 일부 다른 백그라운드 작업을 몇 분간 사용할 수 없는 경우라도 응용 프로그램은 여전히 정상일 수 있습니다.

모니터링에는 일반 액세스에 사용하는 것과 동일한 끝점이 아닌 일반 액세스 끝점에서 상태 확인 검사용으로 설계된 특정 경로(예: /HealthCheck/{GUID}/)를 사용합니다. 이렇게 하면 모니터링 도구를 통해 응용 프로그램에서 새로운 사용자 등록 추가, 로그인 및 테스트 명령 발행과 같은 일부 기능 테스트를 실행할 수 있으며 일반 액세스 끝점의 가용성을 확인할 수도 있습니다.

모니터링 요청에 대한 응답으로 서비스에서 수집하는 정보의 유형 및 이런 정보를 반환하는 방법. 대부분의 기존 도구와 프레임워크는 끝점이 반환하는 HTTP 상태 코드만 확인합니다. 추가 정보를 반환하고 유효성을 검사하려면 사용자 지정 모니터링 유틸리티 또는 서비스를 생성해야 합니다.

수집하는 정보의 양. 검사 중 과도한 처리를 수행하면 응용 프로그램에 오버로드가 발생하고 다른 사용자에게 영향을 미칠 수 있습니다. 이 때 모니터링 시스템의 시간 제한을 초과하게 되면 응용 프로그램안 사용할 수 없음으로 표시됩니다. 대부분의 응용 프로그램에는 오류 처리기, 성능과 자세한 오류 정보를 로그하는 성능 카운터와 같은 계측이 포함되어 있는데, 이런 계측은 상태 확인 검사에서 추가 정보의 반환을 충분히 대신할 수 있습니다.

응용 프로그램을 악의적인 공격에 노출시키고 민감한 정보를 노출할 위험이 있으며 또는 서비스 거부(DoS) 공격을 유도할 수 있는 공용 액세스에서 모니터링 끝점을 보호하도록 보안을 구성하는 방법. 일반적으로 이런 방법은 응용 프로그램을 다시 시작하지 않고 쉽게 업데이트할 수 있도록 응용 프로그램 구성에서 수행되어야 합니다. 다음 기법 중 하나 이상의 사용을 고려해야 합니다.

- 인증을 통한 끝점의 보안. 이 기법은 요청 헤더에 인증 보안 키를 사용하거나 모니터링 서비스 또는 도구가 인증을 지원하는 경우 자격 증명과 요청을 통과시켜 수행할 수 있습니다.

 - 은닉되거나 숨겨진 끝점의 사용. 예를 들면 기본 응용 프로그램 URL이 사용하는 것과 다른 IP 주소에 끝점을 노출시키거나, 끝점을 비표준 HTTP 포트에 구성하거나, 시험 페이지까지 복잡한 경로를 사용합니다. 일반적으로 응용 프로그램 구성에서 끝점 주소와 포트를 추가로 지정하고 IP 주소를 직접 지정하지 않는 경우 이런 끝점의 항목을 DNS 서버에 추가할 수 있습니다.

 - 끝점에서 키 값 또는 작동 모드 값과 같은 매개 변수를 수락하는 메서드의 노출. 이런 매개 변수에 제공된 값에 따라 요청을 수신하면 코드는 특정 테스트 또는 테스트 집합을 수행하거나 매개 변수 값을 인식하지 못하는 경우 404 (Not Found)를 반환할 수 있습니다. 인식된 매개 변수 값은 응용 프로그램 구성에 설정될 수 있습니다.

     >  DoS 공격은 응용 프로그램의 작동에 영향을 미치지 않으면서 기본 기능 테스트를 수행하는 별도의 끝점에 더 적은 영향을 미칠 수 있습니다. 민감한 정보를 노출시킬 수 있는 테스트는 사용하지 않는 것이 좋습니다. 공격자에게 유용할 수 있는 정보를 반환해야 하는 경우, 끝점과 데이터에 권한이 없는 액세스의 차단 방법을 고려해야 하는데, 이 경우 은닉에만 의존하는 것으로는 충분하지 않습니다. 또한 서버에 부하를 추가하더라도 HTTPS 연결을 사용하고 민감한 데이터를 암호화하는 것도 고려해야 합니다.

- 인증을 사용해 보안을 제공하는 끝점에 액세스하는 방법. 모든 도구와 프레임워크를 자격 증명과 상태 확인 요청을 포함하도록 구성할 수는 없습니다. 예를 들어 Microsoft Azure의 기본 제공 상태 확인 기능은 인증 자격 증명을 제공할 수 없습니다. 일부 타사 대안은 [Pingdom](https://www.pingdom.com/), [Panopta](http://www.panopta.com/), [NewRelic](https://newrelic.com/), 및 [Statuscake](https://www.statuscake.com/)입니다.

- 모니터링 에이전트가 올바르게 수행되고 있는지를 확인하는 방법. 한 가지 접근 방식은 응용 프로그램 구성의 값 또는 에이전트를 테스트하는 데 사용할 수 있는 임의 값을 단순 반환하는 끝점을 노출시키는 것입니다.

   >  또한 모니터링 시스템이 잘못된 반대 결과를 발행하지 않도록 자체 테스트 또는 기본 제공 테스트와 같은 검사를 자체적으로 수행하는지도 확인해야 합니다.

## When to use this pattern

This pattern is useful for:
- Monitoring websites and web applications to verify availability.
- Monitoring websites and web applications to check for correct operation.
- Monitoring middle-tier or shared services to detect and isolate a failure that could disrupt other applications.
- Complementing existing instrumentation in the application, such as performance counters and error handlers. Health verification checking doesn't replace the requirement for logging and auditing in the application. Instrumentation can provide valuable information for an existing framework that monitors counters and error logs to detect failures or other issues. However, it can't provide information if the application is unavailable.

## Example

The following code examples, taken from the `HealthCheckController` class (a sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring)), demonstrates exposing an endpoint for performing a range of health checks.

The `CoreServices` method, shown below in C#, performs a series of checks on services used in the application. If all of the tests run without error, the method returns a 200 (OK) status code. If any of the tests raises an exception, the method returns a 500 (Internal Error) status code. The method could optionally return additional information when an error occurs, if the monitoring tool or framework is able to make use of it.

```csharp
public ActionResult CoreServices()
{
  try
  {
    // Run a simple check to ensure the database is available.
    DataStore.Instance.CoreHealthCheck();

    // Run a simple check on our external service.
    MyExternalService.Instance.CoreHealthCheck();
  }
  catch (Exception ex)
  {
    Trace.TraceError("Exception in basic health check: {0}", ex.Message);

    // This can optionally return different status codes based on the exception.
    // Optionally it could return more details about the exception.
    // The additional information could be used by administrators who access the
    // endpoint with a browser, or using a ping utility that can display the
    // additional information.
    return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
  }
  return new HttpStatusCodeResult((int)HttpStatusCode.OK);
}
```
The `ObscurePath` method shows how you can read a path from the application configuration and use it as the endpoint for tests. This example, in C#, also shows how you can accept an ID as a parameter and use it to check for valid requests.

```csharp
public ActionResult ObscurePath(string id)
{
  // The id could be used as a simple way to obscure or hide the endpoint.
  // The id to match could be retrieved from configuration and, if matched,
  // perform a specific set of tests and return the result. If not matched it
  // could return a 404 (Not Found) status.

  // The obscure path can be set through configuration to hide the endpoint.
  var hiddenPathKey = CloudConfigurationManager.GetSetting("Test.ObscurePath");

  // If the value passed does not match that in configuration, return 404 (Not Found).
  if (!string.Equals(id, hiddenPathKey))
  {
    return new HttpStatusCodeResult((int)HttpStatusCode.NotFound);
  }

  // Else continue and run the tests...
  // Return results from the core services test.
  return this.CoreServices();
}
```

The `TestResponseFromConfig` method shows how you can expose an endpoint that performs a check for a specified configuration setting value.

```csharp
public ActionResult TestResponseFromConfig()
{
  // Health check that returns a response code set in configuration for testing.
  var returnStatusCodeSetting = CloudConfigurationManager.GetSetting(
                                                          "Test.ReturnStatusCode");

  int returnStatusCode;

  if (!int.TryParse(returnStatusCodeSetting, out returnStatusCode))
  {
    returnStatusCode = (int)HttpStatusCode.OK;
  }

  return new HttpStatusCodeResult(returnStatusCode);
}
```
## Monitoring endpoints in Azure hosted applications

Some options for monitoring endpoints in Azure applications are:

- Use the built-in monitoring features of Azure.

- Use a third-party service or a framework such as Microsoft System Center Operations Manager.

- Create a custom utility or a service that runs on your own or on a hosted server.

   >  Even though Azure provides a reasonably comprehensive set of monitoring options, you can use additional services and tools to provide extra information. Azure Management Services provides a built-in monitoring mechanism for alert rules. The alerts section of the management services page in the Azure portal allows you to configure up to ten alert rules per subscription for your services. These rules specify a condition and a threshold value for a service such as CPU load, or the number of requests or errors per second, and the service can automatically send email notifications to addresses you define in each rule.

The conditions you can monitor vary depending on the hosting mechanism you choose for your application (such as Web Sites, Cloud Services, Virtual Machines, or Mobile Services), but all of these include the ability to create an alert rule that uses a web endpoint you specify in the settings for your service. This endpoint should respond in a timely way so that the alert system can detect that the application is operating correctly.

>  Read more information about [creating alert notifications][portal-alerts].

If you host your application in Azure Cloud Services web and worker roles or Virtual Machines, you can take advantage of one of the built-in services in Azure called Traffic Manager. Traffic Manager is a routing and load-balancing service that can distribute requests to specific instances of your Cloud Services hosted application based on a range of rules and settings.

In addition to routing requests, Traffic Manager pings a URL, port, and relative path that you specify on a regular basis to determine which instances of the application defined in its rules are active and are responding to requests. If it detects a status code 200 (OK), it marks the application as available. Any other status code causes Traffic Manager to mark the application as offline. You can view the status in the Traffic Manager console, and configure the rule to reroute requests to other instances of the application that are responding.

However, Traffic Manager will only wait ten seconds to receive a response from the monitoring URL. Therefore, you should ensure that your health verification code executes in this time, allowing for network latency for the round trip from Traffic Manager to your application and back again.

>  Read more information about using [Traffic Manager to monitor your applications](https://azure.microsoft.com/documentation/services/traffic-manager/). Traffic Manager is also discussed in [Multiple Datacenter Deployment Guidance](https://msdn.microsoft.com/library/dn589779.aspx).

## Related guidance

The following guidance can be useful when implementing this pattern:
- [Instrumentation and Telemetry Guidance](https://msdn.microsoft.com/library/dn589775.aspx). Checking the health of services and components is typically done by probing, but it's also useful to have information in place to monitor application performance and detect events that occur at runtime. This data can be transmitted back to monitoring tools as additional information for health monitoring. Instrumentation and Telemetry Guidance explores gathering remote diagnostics information that's collected by instrumentation in applications.
- [Receiving alert notifications][portal-alerts].
- This pattern includes a downloadable [sample application](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring).

[portal-alerts]: https://azure.microsoft.com/documentation/articles/insights-receive-alert-notifications/
