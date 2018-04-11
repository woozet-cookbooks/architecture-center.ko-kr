---
title: 조건부로 Azure Resource Manager 템플릿의 리소스 배포
description: 조건부로 매개 변수의 값에 종속되는 리소스를 배포하도록 Azure Resource Manager 템플릿의 기능을 확장하는 방법을 설명합니다.
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: e911e7dc41b4f71ebfaf13a00f8cdbb5b4e2578b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="54c35-103">조건부로 Azure Resource Manager 템플릿의 리소스 배포</span><span class="sxs-lookup"><span data-stu-id="54c35-103">Conditionally deploy a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="54c35-104">매개 변수 값이 있는지 여부와 같은 조건에 따라 리소스를 배포하도록 템플릿을 디자인해야 하는 시나리오가 있을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-104">There are some scenarios in which you need to design your template to deploy a resource based on a condition, such as whether or not a parameter value is present.</span></span> <span data-ttu-id="54c35-105">예를 들어 템플릿은 가상 네트워크를 배포하고, 피어링을 위한 다른 가상 네트워크를 지정하기 위한 매개 변수를 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-105">For example, your template may deploy a virtual network and include parameters to specify other virtual networks for peering.</span></span> <span data-ttu-id="54c35-106">피어링에 대해 매개 변수 값을 지정하지 않은 경우 Resource Manager가 피어링 리소스를 배포하는 것을 원하지 않을 것입니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-106">If you've not specified any parameter values for peering, you don't want Resource Manager to deploy the peering resource.</span></span>

<span data-ttu-id="54c35-107">이렇게 하려면 리소스에 [`condition` 요소][azure-resource-manager-condition]를 사용하여 매개 변수 배열의 길이를 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-107">To accomplish this, use the [`condition` element][azure-resource-manager-condition] in the resource to test the length of your parameter array.</span></span> <span data-ttu-id="54c35-108">길이가 0인 경우 `false`를 반환하여 배포를 방지하고, 0보다 큰 모든 값의 경우에는 `true`를 반환하여 배포를 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-108">If the length is zero, return `false` to prevent deployment, but for all values greater than zero return `true` to allow deployment.</span></span>

## <a name="example-template"></a><span data-ttu-id="54c35-109">예제 템플릿</span><span class="sxs-lookup"><span data-stu-id="54c35-109">Example template</span></span>

<span data-ttu-id="54c35-110">이 내용을 설명하는 예제 템플릿을 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-110">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="54c35-111">템플릿은 [`condition` 요소][azure-resource-manager-condition]를 사용하여 `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` 리소스의 배포를 제어합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-111">Our template uses the [`condition` element][azure-resource-manager-condition] to control deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource.</span></span> <span data-ttu-id="54c35-112">이 리소스는 동일한 지역에 있는 두 Azure Virtual Network 간에 피어링을 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-112">This resource creates a peering between two Azure Virtual Networks in the same region.</span></span>

<span data-ttu-id="54c35-113">템플릿의 각 섹션에 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-113">Let's take a look at each section of the template.</span></span>

<span data-ttu-id="54c35-114">`parameters` 요소는 `virtualNetworkPeerings`라는 단일 매개 변수를 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-114">The `parameters` element defines a single parameter named `virtualNetworkPeerings`:</span></span> 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualNetworkPeerings": {
      "type": "array",
      "defaultValue": []
    }
  },
```
<span data-ttu-id="54c35-115">`virtualNetworkPeerings` 매개 변수는 `array`이며 다음 스키마를 갖습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-115">Our `virtualNetworkPeerings` parameter is an `array` and has the following schema:</span></span>

```json
"virtualNetworkPeerings": [
    {
        "remoteVirtualNetwork": {
            "name": "my-other-virtual-network"
        },
        "allowForwardedTraffic": true,
        "allowGatewayTransit": true,
        "useRemoteGateways": false
    }
]
```

<span data-ttu-id="54c35-116">매개 변수의 속성은 [가상 네트워크 피어링과 관련된 설정][vnet-peering-resource-schema]을 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-116">The properties in our parameter specify the [settings related to peering virtual networks][vnet-peering-resource-schema].</span></span> <span data-ttu-id="54c35-117">`resources` 섹션에서 `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` 리소스를 지정할 때 이러한 속성에 대한 값을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-117">We'll provide the values for these properties when we specify the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the `resources` section:</span></span>

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "virtualNetworks"
      ],
      "copy": {
          "name": "iterator",
          "count": "[length(variables('peerings'))]",
          "mode": "serial"
      },
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
          },
          "variables": {
          },
          "resources": [
            {
              "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
              "apiVersion": "2016-06-01",
              "location": "[resourceGroup().location]",
              "name": "[variables('peerings')[copyIndex()].name]",
              "properties": "[variables('peerings')[copyIndex()].properties]"
            }
          ],
          "outputs": {
          }
        }
      }
    }
]
```
<span data-ttu-id="54c35-118">템플릿의 이 부분에서 진행되는 몇 가지 작업이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-118">There are a couple of things going on in this part of our template.</span></span> <span data-ttu-id="54c35-119">먼저, 배포될 실제 리소스는 `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`를 실제로 배포하는 자체 템플릿을 포함하는 `Microsoft.Resources/deployments` 형식의 인라인 템플릿입니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-119">First, the actual resource being deployed is an inline template of type `Microsoft.Resources/deployments` that includes its own template that actually deploys the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span></span>

<span data-ttu-id="54c35-120">인라인 템플릿의 `name`은 `copyIndex()`의 현재 반복을 접두사 `vnp-`와 연결해서 고유하게 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-120">Our `name` for the inline template is made unique by concatenating the current iteration of the `copyIndex()` with the prefix `vnp-`.</span></span> 

<span data-ttu-id="54c35-121">`condition` 요소는 `greater()` 함수가 `true`로 평가될 때 리소스가 처리되도록 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-121">The `condition` element specifies that our resource should be processed when the `greater()` function evaluates to `true`.</span></span> <span data-ttu-id="54c35-122">여기서는 `virtualNetworkPeerings` 매개 변수 배열이 0보다 `greater()` 상태인지를 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-122">Here, we're testing if the `virtualNetworkPeerings` parameter array is `greater()` than zero.</span></span> <span data-ttu-id="54c35-123">0보다 크면 `true`로 평가되고 `condition`은 충족됩니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-123">If it is, it evaluates to `true` and the `condition` is satisfied.</span></span> <span data-ttu-id="54c35-124">0보다 작으면 `false`입니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-124">Otherwise, it's `false`.</span></span>

<span data-ttu-id="54c35-125">다음으로, `copy` 루프를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-125">Next, we specify our `copy` loop.</span></span> <span data-ttu-id="54c35-126">루프가 순서대로 수행되며, 각 리소스는 마지막 리소스가 배포될 때까지 대기함을 의미하는 `serial` 루트입니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-126">It's a `serial` loop that means the loop is done in sequence, with each resource waiting until the last resource has been deployed.</span></span> <span data-ttu-id="54c35-127">`count` 속성은 루프 반복 횟수를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-127">The `count` property specifies the number of times the loop iterates.</span></span> <span data-ttu-id="54c35-128">여기에서, 일반적으로는 `virtualNetworkPeerings`의 길이로 설정합니다. 배포하려는 리소스를 지정하는 매개 변수 개체를 포함하기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-128">Here, normally we'd set it to the length of the `virtualNetworkPeerings` array because it contains the parameter objects specifying the resource we want to deploy.</span></span> <span data-ttu-id="54c35-129">그러나 이렇게 할 경우, 존재하지 않는 속성에 액세스하려고 한다는 사실을 Resource Manager가 알게 되므로 배열이 비어 있으면 유효성 검사가 실패합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-129">However, if we do that, validation will fail if the array is empty because Resource Manager notices that we are attempting to access properties that do not exist.</span></span> <span data-ttu-id="54c35-130">그러나 이 문제는 해결할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-130">We can work around this, however.</span></span> <span data-ttu-id="54c35-131">필요한 변수를 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-131">Let's take a look at the variables we'll need:</span></span>

```json
  "variables": {
    "workaround": {
       "true": "[parameters('virtualNetworkPeerings')]",
       "false": [{
           "name": "workaround",
           "properties": {}
       }]
     },
     "peerings": "[variables('workaround')[string(greater(length(parameters('virtualNetworkPeerings')), 0))]]"
  },
```

<span data-ttu-id="54c35-132">`workaround` 변수에는 `true` 및 `false`라는 2개의 속성이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-132">Our `workaround` variable includes two properties, one named `true` and one named `false`.</span></span> <span data-ttu-id="54c35-133">`true` 속성은 `virtualNetworkPeerings` 매개 변수 배열의 값으로 평가됩니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-133">The `true` property evaluates to the value of the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="54c35-134">`false` 속성은 유효성을 충족하는 `virtualNetworkPeerings` 매개 변수와 마찬가지로, `false`가 실제로 배열이라는 사실을 알기 위해 Resource Manager가 예상하는 명명된 속성을 포함하는 빈 개체로 평가됩니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-134">The `false` property evaluates to an empty object including the named properties that Resource Manager expects to see&mdash;note that `false` is actually an array, just as our `virtualNetworkPeerings` parameter is, which will satisfy validation.</span></span> 

<span data-ttu-id="54c35-135">`peerings` 변수는 `workaround` 변수를 한 번 더 사용하여, `virtualNetworkPeerings` 매개 변수 배열의 길이가 0보다 큰지 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-135">Our `peerings` variable uses our `workaround` variable by once again testing if the length of the `virtualNetworkPeerings` parameter array is greater than zero.</span></span> <span data-ttu-id="54c35-136">0보다 크면 `string`은 `true`이고, `workaround` 변수는 `virtualNetworkPeerings` 매개 변수 배열로 평가됩니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-136">If it is, the `string` evaluates to `true` and the `workaround` variable evalutes to the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="54c35-137">그렇지 않은 경우 `false`로 평가되고, `workaround` 변수는 배열의 첫 번째 요소에 있는 빈 개체로 평가됩니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-137">Otherwise, it evaluates to `false` and the `workaround` variable evaluates to our empty object in the first element of the array.</span></span>

<span data-ttu-id="54c35-138">유효성 검사 문제가 해결되었으므로, 중첩된 템플릿에서 `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` 리소스의 배포를 간단히 지정하고, `virtualNetworkPeerings` 매개 변수 배열의 `name` 및 `properties`를 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-138">Now that we've worked around the validation issue, we can simply specify the deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the nested template, passing the `name` and `properties` from our `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="54c35-139">이 내용은 리소스의 `properties` 요소에 중첩된 `template` 요소에서 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-139">You can see this in the `template` element nested in the `properties` element of our resource.</span></span>

## <a name="next-steps"></a><span data-ttu-id="54c35-140">다음 단계</span><span class="sxs-lookup"><span data-stu-id="54c35-140">Next steps</span></span>

* <span data-ttu-id="54c35-141">이 기법은 [템플릿 구성 요소 프로젝트](https://github.com/mspnp/template-building-blocks) 및 [Azure 참조 아키텍처](/azure/architecture/reference-architectures/)에서도 구현됩니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-141">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="54c35-142">이러한 참조 아키텍처를 사용하여 고유한 아키텍처를 만들거나 참조 아키텍처 중 하나를 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="54c35-142">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings