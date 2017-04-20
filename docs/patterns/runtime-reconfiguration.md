---
title: Runtime Reconfiguration
description: Design an application so that it can be reconfigured without requiring redeployment or restarting the application.
keywords: design pattern
author: dragon119
ms.service: guidance
ms.topic: article
ms.author: pnp
ms.date: 03/24/2017

pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: [design-implementation, management-monitoring]
---

# 런타임 재구성

[!INCLUDE [header](../_includes/header.md)]

응용 프로그램은 응용 프로그램을 재배포 또는 재시작할 필요 없이 재구성할 수 있도록 설계합니다.  이렇게 하면 가용성을 유지하고 다운타임을 최소화할 수 있습니다. 

## 컨텍스트와 문제점

상용 및 비즈니스 웹사이트와 같은 응용 프로그램의 기본 목적은 다운타임을 줄이고 고객 및 사용자의 사용 중단을 최소화하는 것입니다.  응용 프로그램을 배포 또는 사용 중에 특정한 동작이나 설정을 변경하도록 재구성해야 하는 경우가 있습니다. 그러므로, 응용 프로그램이 실행되는 동안 이러한 구성 변경을 적용할 수 있고, 프로그램의 구성요소가 그 변경 내용을 최대한 빨리 감지하여 적용하도록 설계한 것은 큰 장점입니다. 

이와 같은 구성 변경이 적용된 예는 로깅 단위를 조정하여 응용 프로그램의 문제점을 디버깅하는 것을 지원하거나, 연결 문자열을 전환하여 다른 데이터 저장소를 사용하는 것, 또는 응용 프로그램의 특정 기능 섹션을 끄거나 켜는 것 등입니다. 

## 솔루션

이러한 패턴 구현의 솔루션은 응용 프로그램 호스팅 환경에서 사용할 수 있는 기능에 좌우됩니다. 일반적으로, 응용 프로그램 코드는 응용 프로그램 구성에 대한 변경 내용을 감지하면  호스팅 인프라에 의해 발생한 하나 이상의 이벤트에 응답합니다. 이는 대개 새로운 구성 파일을 업로드한 결과이거나 API에 액세스함으로써 관리 포털을 통해 구성의 변경 내용에 대한 응답입니다.

구성 변경 이벤트를 처리하는 코드는 변경 내용을 검사하고 이를 응용 프로그램의 구성 요소에 적용할 수 있습니다. 이 구성 요소는 변경 내용을 감지하고 이에 대응해야 합니다. 그러므로 구성 요소에 사용되는 값은 대개 이벤트 처리기의 코드가 새 값으로 설정하거나 실행할 수 있는 쓰기 가능 속성 또는 메서드로 표시됩니다.  이 때부터 구성 요소는 응용 프로그램 동작에 필요한 변경 내용이 발생하도록 새로운 값을 사용합니다. 

구성 요소에서 런타임에 변경 내용을 적용할 수 없는 경우, 응용 프로그램을 다시 시작할 때 이 변경 내용이 적용되도록 응용 프로그램을 다시 시작해야 합니다.  일부 호스팅 환경에서는 이런 종류의 변경 내용을 감지하고, 응용 프로그램을 다시 시작해야 한다는 것을 해당 환경에 표시하는 것이 가능합니다. 또 다른 경우, 설정 변경 내용을 분석하고 필요에 따라 응용 프로그램을 강제로 다시 시작하는 코드를 구현해야 할 수 있습니다.

그림은 이러한 패턴의 개요를 보여 줍니다.

![Figure 1 - A basic overview of this pattern](./_images/runtime-reconfiguration-pattern.png)


대부분의 환경은 구성 변경으로 인해 발생한 이벤트를 표시합니다. 그렇지 않은 경우, 구성에 대한 변경을 정기적으로 확인하여 변경 내용을 적용하는 폴링 메커니즘이 있어야 합니다.  또한, 변경 내용이 런타임에 적용할 수 없는 경우 응용 프로그램을 다시 시작해야 합니다. 예를 들어, 사전에 설정된 간격으로 구성 파일의 날짜와 시간을 비교하고, 새 버전이 발견되면 변경 내용을 적용하는 코드를 실행할 수 있습니다. 또 다른 방식은 응용 프로그램의 관리 UI에 제어를 통합하거나, 응용 프로그램 외부에서 액세스할 수 있는 보안 끝점을 노출하여, 업데이트된 구성을 읽고 적용하는 코드를 실행하도록 하는 것입니다.

또는 응용 프로그램이 해당 환경의 일부 다른 변경 내용에 대응할 수 있습니다. 예를 들어, 특정 런타임 오류의 발생으로 로깅 구성이 변경되어 추가 정보를 자동 수집하거나, 코드가 현재 날짜를 사용해 시즌이나 특별 이벤트를 반영하는 테마를 읽고 적용할 수 있습니다.

## 문제점과 고려 사항

이러한 패턴의 구현 방식을 결정할 때 다음 사항을 고려합니다.

구성 설정은 배포된 응용 프로그램 외부에 저장해야 패키지 전체를 다시 배포하지 않고도 업데이트할 수 있습니다. 일반적으로 설정은 구성 파일 또는 데이터베이스나 온라인 저장소와 같은 외부 리포지토리에 저장됩니다. 런타임 구성 메커니즘에 대한 액세스는 엄격하게 제어해야 하며, 사용할 때도 철저하게 감사해야 합니다.

호스팅 인프라가 구성 변경 이벤트를 자동 감지하지 않고 응용 프로그램 코드에 이 이벤트를 표시하지 않는다면, 변경을 감지하고 적용할 다른 메커니즘을 구현해야 합니다.  이는 폴링 메커니즘을 사용하거나 업데이트 프로세스를 시작하는 대화형 제어 또는 끝점을 표시함으로써 가능합니다.

폴링 메커니즘을 구현해야 하는 경우, 구성 업데이트 확인을 얼마나 자주 수행할 지 고려합니다. 폴링 간격이 길면 일정 시간 동안 변경 내용이 적용되지 않을 수 있습니다. 간격이 짧으면 사용 가능한 게산 및 I/O 리소스를 포함함으로써 운영에 좋지 않은 영향을 줄 수 있습니다. 

응용 프로그램에 하나 이상의 인스턴스가 있는 경우, 변경 감지 방식에 따라 추가 요소를 고려해야 합니다. 호스팅 인프라에 의해 발생한 이벤트를 통해 변경이 자동 감지되는 경우, 모든 응용 프로그램 인스턴스가 같은 시간에 변경을 감지하지 않을 수 있습니다. 즉, 일정 기간 동안 원래의 구성을 사용하는 인스턴스가 일부 있고, 새로운 설정을 사용하는 인스턴스도 있습니다. 폴링 메커니즘을 통해 업데이트를 감지하는 경우, 이는 모든 인스턴스에 변경 내용을 전달해 일관성을 유지해야 합니다. 

일부 구성 변경은 응용 프로그램을 다시 시작해야 하며, 호스팅 서버를 재부팅해야 하는 경우도 있습니다. 이런 종류의 구성 설정을 식별하고 각각에 적절한 작업을 수행해야 합니다. 예를 들어, 응용 프로그램을 다시 시작해야 하는 변경에 대해 이를 자동으로 수행하는 경우가 있고 또는 응용 프로그램이 과도한 로드에 있지 않고 응용 프로그램의 다른 인스턴스가 부하를 처리할 수 있는 경우 관리자가 다시 시작해야 하는 경우도 있습니다. 

업데이트를 모든 인스턴스에 적용하기 전에, 업데이트의 단계별 롤아웃을 계획하고, 업데이트의 성공적 수행 여부와 업데이트된 응용 프로그램 인스턴스가 정확하게 작동하고 있는지 확인합니다. 이를 통해 오류 발생의 경우 전체 응용 프로그램의 중단을 방지할 수 있습니다. 업데이트로 인해 응용 프로그램을 다시 시작하거나 다시 부팅해야 하는 경우, 특히 응용 프로그램의 시작 또는 웜업 시간이 오래 걸리는 경우라면 단계별 롤아웃을 통해 여러 인스턴스가 동시에 오프라인되지 않도록 합니다. 

문제를 야기하거나 응용 프로그램의 실패를 초래하는 구성 변경은 어떻게 롤백할지 고려합니다. 예를 들어, 변경 내용을 감지하는 폴링 간격을 기다리지 않고 즉시 변경을 롤백할 수 있어야 합니다. 

구성 설정의 위치가 어떻게 응용 프로그램 성능에 영향을 줄 수 있는지 고려합니다. 예를 들어, 응용 프로그램을 시작하거나 구성 변경 내용을 적용할 때 외부 저장소를 사용할 수 없는 경우 발생하는 오류를 처리해야 합니다. 기본 구성을 사용하거나, 또는 설정을 서버에 로컬로 캐싱하여 원격 데이터 저장소에 액세스를 시도하는 동안 이 값을 다시 사용하는 방법이 있습니다. 

캐싱을 사용하면 구성 요소가 구성 설정에 계속 액세스해야 하는 경우 지연을 줄일 수 있습니다. 그러나, 구성이 변경되면 응용 프로그램 코드는 캐싱된 설정을 무효화하고 구성 요소는 업데이트된 설정을 사용해야 합니다. 

## 패턴을 언제 사용할 것인가

이 방식은 다음과 같은 경우에 권장합니다:

- 불필요한 모든 다운타임을 방지해야 하는 응용 프로그램, 변경 내용을 응용 프로그램 구성에 계속 적용할 수 있어야 함. 

- 주요 구성이 변경되어 자동으로 발생한 이벤트를 표시한 환경. 일반적으로 새 구성 파일이 감지되거나, 기존 구성 파일이 변경되었을 때입니다. 

- 구성이 자주 변경되고 응용 프로그램을 다시 시작하거나 호스팅 서버를 다시 부팅하지 않고도 변경 내용을 구성 요소에 적용할 수 있는 응용 프로그램. 

런타임 구성 요소가 초기화에서만 구성할 수 있도록 설계된 경우 이 패턴은 적절하지 않을 수 있으며, 그러한 구성 요소를 업데이트하는 작업이 응용 프로그램을 다시 시작하고 짧은 다운타임을 감내하는 것에 비해 타당하지 않을 수 있습니다.

## 예시

Microsoft Azure 클라우드 서비스 역할은 호스팅 환경이 ServiceConfiguration.cscfg 파일의 변경을 감지했을 때 발생하는 두 가지 이벤트를 감지하고 표시합니다. 

- **RoleEnvironment.Changing**. 이 이벤트는 구성 변경이 감지된 후 응용 프로그램에 적용하기 전에 발생합니다. 이벤트를 처리해 변경 내용을 조회하고 런타임 재구성을 취소할 수 있습니다. 변경을 취소하는 경우, 응용 프로그램에서 새 구성을 사용하도록 웹 또는 작업자 역할이 자동으로 재시작됩니다.
- **RoleEnvironment.Changed**. 이 이벤트는 응용 프로그램 구성이 적용된 후 발생합니다. 이벤트를 처리해 적용된 변경 내용을 조회할 수 있습니다.

`RoleEnvironment.Changing` 이벤트의 변경을 취소하는 것은, 응용 프로그램이 실행되는 동안 새 설정을 적용할 수 없으며 새 값을 사용하려면 응용 프로그램을 다시 시작해야 한다는 것을 Azure에 표시하는 것입니다. 실제로는, 런타임 중 응용 프로그램 또는 구성 요소가 변경에 대응할 수 없고 새 값을 사용하려면 다시 시작해야 하는 경우에만 변경을 취소합니다.

> 자세한 내용은 [RoleEnvironment.Changing 이벤트](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.changing.aspx)를 참조하십시오.

`RoleEnvironment.Changing` 및 `RoleEnvironment.Changed` 이벤트를 처리하려면, 일반적으로 이벤트에 사용자 지정 처리기를 추가합니다.  예를 들어, `Global.asax.cs` 클래스의 다음 코드는([GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/runtime-reconfiguration)에 있는 런타임 재구성 솔루션에 포함) 이름이 `RoleEnvironment_Changed` 인 사용자 지정 기능을 어떻게 이벤트 처리기 체인에 추가하는지 보여 줍니다. 이는 예시 Global.asax.cs 파일에서 가져왔습니다.

> 이 패턴의 예시는 RuntimeReconfiguration 솔루션의 RuntimeReconfiguration.Web 프로젝트에 있습니다.

```csharp
protected void Application_Start(object sender, EventArgs e)
{
  ConfigureFromSetting(CustomSettingName);
  RoleEnvironment.Changed += this.RoleEnvironment_Changed;
}
```

In a web or worker role you can use similar code in the `OnStart` event handler of the role to handle the `RoleEnvironment.Changing` event. This is from the WebRole.cs file of the example.

```csharp
public override bool OnStart()
{
  // Add the trace listener. The web role process isn't configured by web.config.
  Trace.Listeners.Add(new DiagnosticMonitorTraceListener());

  RoleEnvironment.Changing +=   this.RoleEnvironment_Changing;
  return base.OnStart();
}
```

웹 역할의 경우 `OnStart` 이벤트 처리기는 웹 응용 프로그램 프로세스와는 다른 프로세스에서 실행됩니다. 그렇기 때문에 일반적으로 웹 응용 프로그램의 런타임 구성과 그 역할의 `RoleEnvironment.Changing` 이벤트를 업데이트할 수 있도록 Global.asax 파일의 `RoleEnvironment.Changed` 이벤트 처리기를 처리하는 것입니다. 작업자 역할의 경우, `RoleEnvironment.Changing` 및 `RoleEnvironment.Changed` 이벤트 모두 `OnStart` 이벤트 처리기로 구독할 수 있습니다. 

> 사용자 지정 구성 설정을 서비스 구성 파일, 사용자 지정 구성 파일, Azure SQL 데이터베이스나 SQL 서버와 같은 가상 컴퓨터 내의 데이터베이스, 또는 Azure Blob나 테이블 저장소에 저장할 수 있습니다.  사용자 지정 구성 설정에 액세스하여 이를 응용 프로그램에 적용할 수 있는 코드를 만들어야 하며, 대개는 응용 프로그램의 구성 요소 속성을 설정하여 수행합니다.

예를 들어 다음 사용자 지정 기능은 Azure 서비스 구성 파일에서 설정 값을 읽은 다음 `SomeRuntimeComponent` 라는 런타임 구성 요소의 현재 인스턴스에 이를 적용합니다. 이는 예시 Global.asax.cs 파일에서 가져왔습니다.

> Windows Identify Framework의 구성 설정 등 일부 구성 설정은 Azure 서비스 구성 파일에 저장할 수 없으며 App.config 또는 Web.config 파일에 저장해야 합니다. 

```csharp
private static void ConfigureFromSetting(string settingName)
{
  var value = RoleEnvironment.GetConfigurationSettingValue(settingName);
  SomeRuntimeComponent.Instance.CurrentValue = value;
}
```

Azure에서 일부 구성 변경 내용은 자동으로 감지되고 적용됩니다.  여기에는, Diagnostics.wadcfg 파일로 된 Azure 진단 시스템의 구성이 포함되며, 이는 수집 정보의 종류 및 로그 파일 보관 방식을 지정합니다.  그러므로, 서비스 구성 파일에 추가할 사용자 지정 설정을 처리하는 코드를 만드는 것이 필요합니다. 코드는 다음 중 하나를 수행합니다.

- 업데이트된 구성의 사용자 지정 설정을 런타임에 응용 프로그램의 해당 구성 요소에 적용하여 새로운 구성을 작동에 반영하도록 합니다.

- 런타임에 새로운 값을 적용할 수 없으며 변경 내용을 적용하려면 응용 프로그램을 다시 시작해야 한다고 Azure에 알리도록 변경을 취소합니다. 

예를 들어, 다음 코드는 `RoleEnvironment.Changing` 이벤트를 사용해 재시작 없이 런타임 중에 적용할 수 있는 설정을 제외한 모든 설정의 업데이트를 취소하는 방법을 나타냅니다. 이 예시를 사용하면 응용 프로그램을 다시 시작하지 않고 런타임 중에 'CustomSetting'의 변경 내용을 적용할 수 있습니다. 이 설정을 사용하는 구성 요소는 새로운 값을 읽고 그에 따라 런타임 중에 동작을 변경할 수 있습니다. 기타 다른 구성 요소 변경 내용으로 인해 웹 또는 작업자 역할이 자동으로 다시 시작됩니다.

```csharp
private void RoleEnvironment_Changing(object sender,
                               RoleEnvironmentChangingEventArgs e)
{
  var changedSettings = e.Changes.OfType<RoleEnvironmentConfigurationSettingChange>()
                                 .Select(c => c.ConfigurationSettingName).ToList();
  Trace.TraceInformation("Changing notification. Settings being changed: "
                         + string.Join(", ", changedSettings));

  if (changedSettings
    .Any(settingName => !string.Equals(settingName, CustomSettingName,
                               StringComparison.Ordinal)))
  {
    Trace.TraceInformation("Cancelling dynamic configuration change (restarting).");

    // Setting this to true will restart the role gracefully. If Cancel isn't
    // set to true, and the change isn't handled by the application, the
    // application won't use the new value until it's restarted (either
    // manually or for some other reason).
    e.Cancel = true;
  }
  Else
  {
    Trace.TraceInformation("Handling configuration change without restarting. ");
  }
}
```

> 이 방식은 응용 프로그램 코드에서 인식하지 못하는(런타임 중에 적용할 수 있는지 확인 불가) 설정 변경으로 인해 재시작되기 때문에 모범 사례가 됩니다. 이런 변경 내용 중 하나라도 취소되면 역할이 다시 시작됩니다. 

`RoleEnvironment.Changing` 이벤트 처리기에서 취소되지 않은 업데이트는 이후에 감지되어 Azure 프레임워크에서 새로운 구성 요소를 승인한 후 응용 프로그램 구성 요소에 적용할 수 있습니다. 예를 들어, 샘플 솔루션의 `Global.asax` 파일에 있는 다음 코드는 `RoleEnvironment.Changed` 이벤트를 처리합니다. 각 구성 설정을 검사하고, 'CustomSetting'을 발견하면 새 설정을 응용 프로그램의 해당 구성 요소에 적용할 기능을 호출합니다.

```csharp
private void RoleEnvironment_Changed(object sender,
                               RoleEnvironmentChangedEventArgs e)
{
  Trace.TraceInformation("Updating instance with new configuration settings.");

  foreach (var settingChange in
           e.Changes.OfType<RoleEnvironmentConfigurationSettingChange>())
  {
    if (string.Equals(settingChange.ConfigurationSettingName,
                      CustomSettingName,
                      StringComparison.Ordinal))
    {
      // Execute a function to update the configuration of the component.
      ConfigureFromSetting(CustomSettingName );
    }
  }
}
```

구성 변경을 취소하지 못했지만 새로운 값을 응용 프로그램 구성 요소에 적용하지 않는 경우, 이후 응용 프로그램을 다시 시작할 때까지 변경 내용이 적용되지 않습니다. 이는 특히 호스팅 역할 인스턴스가 Azure에 의해 정기 유지 관리 작업의 일부로 자동으로 재시작되는 경우(이 시점부터 새 설정값이 적용됩니다), 예상치 못한 동작이 발생할 수 있습니다.

## 관련 패턴 및 지침

- 이 패턴을 잘 보여주는 예시는 [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/runtime-reconfiguration)에 나타나 있습니다.
- 응용 프로그램 배지 패키지에서 중앙 위치로 구성 정보를 이동하면 구성 데이터의 관리와 제어가 간편해 지고, 응용 프로그램 및 응용 프로그램 인스턴스 간에 구성 데이터를 공유할 수 있습니다. 자세한 내용은 [외부 구성 저장소 패턴](external-configuration-store.md)을 참조하십시오.
