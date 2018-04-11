---
title: Azure Resource Manager 템플릿에서 속성 변환기 및 수집기 구현
description: Azure Resource Manager 템플릿에서 속성 변환기 및 수집기를 구현하는 방법을 설명합니다.
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 893779e652b845b3d936d11936dc767ef632fa43
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a>Azure Resource Manager 템플릿에서 속성 변환기 및 수집기 구현

[Azure Resource Manager 템플릿에서 개체를 매개 변수로 사용][objects-as-parameters]에서는 리소스 속성 값을 개체에 저장하고, 배포 중에 리소스에 적용하는 방법을 배웠습니다. 이 방법은 매개 변수를 관리하는 매우 유용한 방법이지만, 여전히 템플릿에서 개체의 속성을 사용할 때마다 리소스 속성에 매핑해야 합니다.

이 문제를 해결하기 위해 개체 배열을 반복하고 리소스에 필요한 JSON 스키마로 변환하는 속성 변형 및 수집기 템플릿을 구현할 수 있습니다.

> [!IMPORTANT]
> 이 방법을 사용하려면 Resource Manager 템플릿 및 함수를 깊이 있게 이해해야 합니다.

[NSG(네트워크 보안 그룹)][nsg]를 배포하는 예제로 속성 수집기 및 변환기를 구현하는 방법을 살펴보겠습니다. 아래 다이어그램은 템플릿과 해당 템플릿 내의 리소스 간 관계를 보여 줍니다.

![속성 수집기 및 변환기 아키텍처](../_images/collector-transformer.png)

**호출 템플릿**에는 다음 두 리소스가 포함됩니다.
* **수집기 템플릿**을 호출하는 템플릿 링크
* 배포할 NSG 리소스

**수집기 템플릿**에는 다음 두 리소스가 포함됩니다.
* **앵커** 리소스
* 복사 루프에서 변환 템플릿을 호출하는 템플릿 링크

**변환 템플릿**에는 단일 리소스가 포함되어 있습니다. 이 템플릿은 `source` JSON을 **주 템플릿**의 NSG 리소스에 필요한 JSON 스키마로 변환하는 변수를 포함하는 빈 템플릿입니다.

## <a name="parameter-object"></a>매개 변수 개체

개체의 `securityRules` 매개 변수 개체를 [매개 변수로 사용][objects-as-parameters]할 것입니다. **변환 템플릿**은 `securityRules` 배열의 각 개체를 **호출 템플릿**의 NSG에 필요한 JSON 스키마로 변환합니다.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{ 
      "networkSecurityGroupsSettings": {
      "value": {
          "securityRules": [
            {
              "name": "RDPAllow",
              "description": "allow RDP connections",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.0.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            },
            {
              "name": "HTTPAllow",
              "description": "allow HTTP connections",
              "direction": "Inbound",
              "priority": 200,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.1.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      }
    }
  }
```

먼저 **변환 템플릿**을 살펴보겠습니다.

## <a name="transform-template"></a>변환 템플릿

**변환 템플릿**에는 **수집기 템플릿**에서 전달되는 다음 두 매개 변수가 포함되어 있습니다. 
* `source`는 속성 배열에서 속성 값 개체 중 하나를 수신하는 개체입니다. 이 예제에서 `"securityRules"` 배열의 각 개체는 한 번에 하나의 전달됩니다.
* `state`는 모든 이전 변환의 연결된 결과를 수신하는 배열입니다. 변환된 JSON의 컬렉션입니다.

매개 변수는 다음과 같습니다.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

템플릿은 `instance`라는 변수도 정의합니다. 이 변수는 `source` 개체를 필요한 JSON 스키마로 실제로 변환합니다.

```json
  "variables": {
    "instance": [
      {
        "name": "[parameters('source').name]",
        "properties":{
            "description": "[parameters('source').description]",
            "protocol": "[parameters('source').protocol]",
            "sourcePortRange": "[parameters('source').sourcePortRange]",
            "destinationPortRange": "[parameters('source').destinationPortRange]",
            "sourceAddressPrefix": "[parameters('source').sourceAddressPrefix]",
            "destinationAddressPrefix": "[parameters('source').destinationAddressPrefix]",
            "access": "[parameters('source').access]",
            "priority": "[parameters('source').priority]",
            "direction": "[parameters('source').direction]"            
        }
      }
    ]

  },
```

마지막으로 템플릿의 `output`은 `state` 매개 변수의 수집된 변환을 `instance` 변수가 수행하는 현재 변환에 연결합니다.

```json
  "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

다음으로, **수집기 템플릿**이 매개 변수 값을 전달하는 방법을 살펴보겠습니다.

## <a name="collector-template"></a>수집기 템플릿

**수집기 템플릿**에는 다음 3가지 매개 변수가 포함되어 있습니다.
* `source`는 완전한 매개 변수 개체 배열입니다. 이 개체는 **호출 템플릿**에 의해 전달됩니다. 이 개체는 **변환 템플릿**에서 `source` 매개 변수와 같은 이름을 가지지만, 이미 알려져 있는 한 가지 주요 차이점이 있습니다. 완전한 배열이지만, 이 배열의 요소를 한 번에 하나씩만 **변환 템플릿**에 전달한다는 것입니다.
* `transformTemplateUri`는 **변환 템플릿**의 URI입니다. 여기서는 템플릿 재사용을 위해 매개 변수로 정의합니다.
* `state`는 **변환 템플릿**에 전달하는 배열로, 처음에는 빈 상태입니다. 복사 루프가 완료되면 변환된 매개 변수 개체 컬렉션이 저장됩니다.

매개 변수는 다음과 같습니다.

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

다음으로, `count`라는 변수를 정의합니다. 해당 값은 `source` 매개 변수 개체 배열의 길이를 갖습니다.

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

예상할 수 있는 것처럼, 이 변수는 복사 루프의 반복 횟수에 사용합니다.

이제 리소스를 살펴보겠습니다. 다음 두 리소스를 정의합니다.
* `loop-0`는 복사 루프의 0부터 시작하는 리소스입니다.
* `loop-`는 `copyIndex(1)` 함수의 결과와 연결되어, `1`부터 시작하는 리소스의 고유한 반복 기반 이름을 생성합니다.

리소스는 다음과 같습니다.

```json
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "loop-0",
      "properties": {
        "mode": "Incremental",
        "parameters": { },
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": { },
          "variables": { },
          "resources": [ ],
          "outputs": {
            "collection": {
              "type": "array",
              "value": "[parameters('state')]"
            }
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "[concat('loop-', copyindex(1))]",
      "copy": {
        "name": "iterator",
        "count": "[variables('count')]",
        "mode": "serial"
      },
      "dependsOn": [
        "loop-0"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": { "uri": "[parameters('transformTemplateUri')]" },
        "parameters": {
          "source": { "value": "[parameters('source')[copyindex()]]" },
          "state": { "value": "[reference(concat('loop-', copyindex())).outputs.collection.value]" }
        }
      }
    }
  ],
```

중첩 템플릿에서 **변환 템플릿**에 전달하는 매개 변수를 좀 더 자세히 살펴보겠습니다. 앞서 설명한 대로 `source` 매개 변수는 `source` 매개 변수 개체 배열에 현재 개체를 전달합니다. `state` 매개 변수는 컬렉션이 발생하는 위치입니다. 이 매개 변수는 `reference()` 함수가 매개 변수 없이 `copyIndex()` 함수를 사용하여 이전 연결된 템플릿 개체의 `name`을 참조한다는 사실을 토대로, 복사 루프의 이전 반복에 대한 출력을 가져온 후 현재 반복으로 전달합니다.

마지막으로 템플릿의 `output`은 **변환 템플릿** 마지막 반복의 `output`을 반환합니다.

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
**변환 템플릿** 마지막 반복의 `output`을 **호출 템플릿**에 반환하는 것은 간단해 보이지 않을 수 있습니다. 이러한 항목은 `source` 매개 변수에 저장한 것으로 보이기 때문입니다. 그렇지만 변환된 속성 개체의 완전한 배열을 포함하는 것이 바로 **변환 템플릿** 마지막 반복이며, 반환하려는 항목은 바로 이것입니다.

마지막으로 **호출 템플릿**에서 **수집기 템플릿**을 호출하는 방법을 살펴보겠습니다.

## <a name="calling-template"></a>호출 템플릿

**호출 템플릿**은 `networkSecurityGroupsSettings`라는 단일 매개 변수를 정의합니다.

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

다음으로, 이 템플릿은 `collectorTemplateUri`라는 단일 변수를 정의합니다.

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

예상한 것처럼 이것은 연결된 템플릿 리소스에서 사용되는 **수집기 템플릿**의 URI입니다.

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('linkedTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

다음 두 매개 변수를 **수집기 템플릿**에 전달합니다.
* `source`는 이 속성 개체 배열입니다. 이 예제에서는 `networkSecurityGroupsSettings` 매개 변수입니다.
* `transformTemplateUri`는 **수집기 템플릿**의 URI로 방금 정의한 변수입니다.

마지막으로, `Microsoft.Network/networkSecurityGroups` 리소스는 `collector` 연결된 템플릿 리소스의 `output`을 해당 `securityRules` 속성에 직접 할당합니다.

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('firstResource').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('firstResource').outputs.result.value]"
      }

  }
```

## <a name="next-steps"></a>다음 단계

* 이 기법은 [템플릿 구성 요소 프로젝트](https://github.com/mspnp/template-building-blocks) 및 [Azure 참조 아키텍처](/azure/architecture/reference-architectures/)에서도 구현됩니다. 이러한 참조 아키텍처를 사용하여 고유한 아키텍처를 만들거나 참조 아키텍처 중 하나를 배포할 수 있습니다.

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
