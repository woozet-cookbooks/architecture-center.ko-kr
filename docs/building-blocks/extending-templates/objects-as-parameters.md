---
title: Azure Resource Manager 템플릿에서 개체를 매개 변수로 사용
description: Azure Resource Manager 템플릿의 기능을 확장하여 개체를 매개 변수로 사용하는 방법을 설명합니다.
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 76f8b9d459f4ab3147b52762b7c26552ec92c7a3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a>Azure Resource Manager 템플릿에서 개체를 매개 변수로 사용

[Azure Resource Manager 템플릿을 작성][azure-resource-manager-create-template]할 때 템플릿에서 직접 리소스 속성 값을 지정하거나, 배포 중에 매개 변수를 정의하고 값을 제공할 수 있습니다. 소규모 배포의 경우에는 각 속성 값에 대해 매개 변수를 사용하는 것이 좋지만, 매개 변수 수는 배포당 255개로 제한됩니다. 보다 크고 복잡한 배포가 되면 매개 변수가 부족해질 수 있습니다.

이 문제를 해결하는 한 가지 방법은 개체를 값이 아닌 매개 변수로 사용하는 것입니다. 이를 수행하려면 템플릿에서 매개 변수를 정의하고 배포 중에 단일 값 대신 JSON 개체를 지정하세요. 그런 다음 템플릿에서 [`parameter()` 함수][azure-resource-manager-functions] 및 점 연산자를 사용하여 매개 변수의 하위 속성을 참조합니다.

가상 네트워크 리소스를 배포하는 예를 살펴보겠습니다. 먼저 템플릿에서 `VNetSettings` 매개 변수를 지정하고 `type`을 `object`로 설정해봅니다.

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
그런 다음 `VNetSettings` 개체의 값을 제공합니다.

> [!NOTE]
> 배포 동안 매개 변수 값을 제공하는 방법을 알아보려면 [Azure Resource Manager 템플릿의 구조 및 구문 이해][azure-resource-manager-authoring-templates]의 **매개 변수** 섹션을 참조하세요. 

```json
"parameters":{
    "VNetSettings":{
        "value":{
            "name":"VNet1",
            "addressPrefixes": [
                {
                    "name": "firstPrefix",
                    "addressPrefix": "10.0.0.0/22"
                }
            ],
            "subnets":[
                {
                    "name": "firstSubnet",
                    "addressPrefix": "10.0.0.0/24"
                },
                {
                    "name":"secondSubnet",
                    "addressPrefix":"10.0.1.0/24"
                }
            ]
        }
    }
}
```

아시다시피 단일 매개 변수는 실제로 3개의 하위 속성, `name`, `addressPrefixes` 및 `subnets`를 지정합니다. 이러한 각 하위 속성은 값 또는 다른 하위 속성을 지정합니다. 그 결과, 단일 매개 변수는 가상 네트워크를 배포하는 데 필요한 모든 값을 지정합니다.

이제 템플릿의 나머지 부분에서 `VNetSettings` 개체가 사용되는 방식을 알아봅니다.

```json
...
"resources": [
    {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('VNetSettings').name]",
        "location":"[resourceGroup().location]",
        "properties": {
          "addressSpace":{
              "addressPrefixes": [
                "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
              ]
          },
          "subnets":[
              {
                  "name":"[parameters('VNetSettings').subnets[0].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
                  }
              },
              {
                  "name":"[parameters('VNetSettings').subnets[1].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
                  }
              }
          ]
        }
    }
  ]
```
`VNetSettings` 개체 값은 `parameters()` 함수를 `[]` 배열 인덱서 및 점 연산자와 함께 사용하여 가상 네트워크 리소스에 필요한 속성에 적용됩니다. 이 방법은 매개 변수 개체의 값을 리소스에 정적으로 적용하려는 경우에만 작동합니다. 그러나 배포하는 동안 속성 값의 배열을 동적으로 할당하려는 경우에는 [복사 루프][azure-resource-manager-create-multiple-instances]를 사용할 수 있습니다. 복사 루프를 사용 하려면 리소스 속성 값의 JSON 배열을 제공하면, 복사 루프가 해당 값을 리소스 속성에 동적으로 적용합니다. 

한 가지 문제점은 동적 방법을 사용하는지를 인식하는 것입니다. 이 문제를 이해하기 위해 속성 값의 일반 배열을 살펴보겠습니다. 이 예제에서 속성 값은 변수에 저장됩니다. 여기에는 이름이 각각 `firstProperty` 및 `secondProperty`인 두 배열이 있습니다. 

```json
"variables": {
    "firstProperty": [
        {
            "name": "A",
            "type": "typeA"
        },
        {
            "name": "B",
            "type": "typeB"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ],
    "secondProperty": [
        "one","two", "three"
    ]
}
```

이제 복사 루프를 사용하여 변수에서 속성에 액세스하는 방법을 살펴보겠습니다.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    ...
    "copy": {
        "name": "copyLoop1",
        "count": "[length(variables('firstProperty'))]"
    },
    ...
    "properties": {
        "name": { "value": "[variables('firstProperty')[copyIndex()].name]" },
        "type": { "value": "[variables('firstProperty')[copyIndex()].type]" },
        "number": { "value": "[variables('secondProperty')[copyIndex()]]" }
    }
}
```

`copyIndex()` 함수가 복사 루프의 현재 반복을 반환하면, 해당 결과를 동시에 두 배열 각각에 대한 인덱스로 사용합니다.

이러한 방식은 두 배열의 길이가 같을 때 잘 작동합니다. 실수를 했거나 두 배열의 길이가 아르면 문제가 발생합니다. 이 경우에는 배포 동안 템플릿의 유효성 검사가 실패합니다. 모든 속성을 단일 개체에 포함하면 값이 누락되었는지를 훨씬 쉽게 파악할 수 있으므로 이러한 방식으로 이 문제를 방지할 수 있습니다. 예를 들어, `propertyObject` 배열의 각 요소가 `firstProperty` 및 `secondProperty` 배열의 합집합인 다른 매개 변수 개체를 살펴보겠습니다.

```json
"variables": {
    "propertyObject": [
        {
            "name": "A",
            "type": "typeA",
            "number": "one"
        },
        {
            "name": "B",
            "type": "typeB",
            "number": "two"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ]
}
```

배열의 세 번째 요소를 확인할 수 있나요? `number` 속성이 없으나, 이러한 방식으로 매개 변수 값을 작성할 때는 누락되었는지를 훨씬 더 쉽게 알 수 있습니다.

## <a name="using-a-property-object-in-a-copy-loop"></a>복사 루프에 속성 개체 사용

이 방법은 [직렬 복사 루프][azure-resource-manager-create-multiple]와 함께 사용할 경우(특히 자식 리소스를 배포하는 경우) 훨씬 더 유용해집니다. 

이를 이해하기 위해 2개의 보안 규칙과 함께 [NSG(네트워크 보안 그룹)][nsg]를 배포하는 템플릿을 살펴보겠습니다. 

먼저, 매개 변수를 살펴보겠습니다. 템플릿을 확인하면, `securityRules`라는 배열을 포함하는 `networkSecurityGroupsSettings` 매개 변수를 정의했다는 것을 알 수 있습니다. 이 배열에는 보안 규칙에 대한 다양한 설정을 지정하는 두 가지 JSON 개체가 포함되어 있습니다.

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

이제 템플릿을 살펴보겠습니다. `NSG1`라는 첫 번째 리소스는 NSG를 배포합니다. `loop-0`이라는 두 번째 리소스는 2개의 함수를 수행합니다. 첫째, NSG에 `dependsOn`하므로 `NSG1`이 완료될 때까지 배포가 시작되지 않으며 순차 루프의 첫 번째 반복입니다. 세 번째 리소스는 마지막 예제와 같이 매개 변수 값으로 개체를 사용해서 보안 규칙을 배포하는 중첩된 템플릿입니다.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "networkSecurityGroupsSettings": {"type":"object"}
  },
  "variables": {},
  "resources": [
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "NSG1",
      "location":"[resourceGroup().location]",
      "properties": {
          "securityRules":[]
      }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "loop-0",
        "dependsOn": [
            "NSG1"
        ],
        "properties": {
            "mode":"Incremental",
            "parameters":{},
            "template": {
                "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            }
        }       
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "[concat('loop-', copyIndex(1))]",
        "dependsOn": [
          "[concat('loop-', copyIndex())]"
        ],
        "copy": {
          "name": "iterator",
          "count": "[length(parameters('networkSecurityGroupsSettings').securityRules)]"
        },
        "properties": {
          "mode": "Incremental",
          "template": {
            "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
           "parameters": {},
            "variables": {},
            "resources": [
                {
                    "name": "[concat('NSG1/' , parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].name)]",
                    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                    "apiVersion": "2016-09-01",
                    "location":"[resourceGroup().location]",
                    "properties":{
                        "description": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].description]",
                        "priority":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].priority]",
                        "protocol":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].protocol]",
                        "sourcePortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourcePortRange]",
                        "destinationPortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationPortRange]",
                        "sourceAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourceAddressPrefix]",
                        "destinationAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationAddressPrefix]",
                        "access":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].access]",
                        "direction":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].direction]"
                        }
                  }
            ],
            "outputs": {}
          }
        }
    }
  ],          
  "outputs": {}
}
```

`securityRules` 자식 리소스에서 속성 값을 지정하는 방법을 좀 더 자세히 살펴보겠습니다. 모든 속성이 `parameter()` 기능을 사용해서 참조됩니다. 그러면 점 연산자를 사용하여 반복의 현재 값으로 인덱싱된 `securityRules` 배열을 참조합니다. 마지막으로 다른 점 연산자를 사용하여 개체의 이름을 참조합니다. 

## <a name="try-the-template"></a>템플릿 시도

이 템플릿으로 실험하려면 다음 단계를 따릅니다. 

1.  Azure Portal로 이동하여 **+** 아이콘을 선택하고 **템플릿 배포** 리소스 종류를 검색하고 선택합니다.
2.  **템플릿 배포** 페이지로 이동한 후 **만들기** 단추를 선택합니다. 그러면 **사용자 지정 배포** 블레이드가 열립니다.
3.  **템플릿 편집** 단추를 선택합니다.
4.  비어 있는 템플릿을 삭제합니다. 
5.  샘플 템플릿을 복사하여 오른쪽 창에 붙여 넣습니다.
6.  **저장** 단추를 선택합니다.
7.  **사용자 지정 배포** 창으로 돌아가면 **매개 변수 편집** 단추를 선택합니다.
8.  **매개 변수 편집** 블레이드에서 기존 템플릿을 삭제합니다.
9.  위의 샘플 매개 변수 템플릿을 복사한 후 붙여 넣습니다.
10. **저장** 단추를 선택합니다. 그러면 **사용자 지정 배포** 블레이드로 돌아갑니다.
11. **사용자 지정 배포** 블레이드에서 구독을 선택하고, 새로 만들거나 기존 리소스 그룹을 사용하고, 위치를 선택합니다. 사용 약관을 검토하고 **동의함** 확인란을 선택합니다.
12. **구입** 단추를 선택합니다.

## <a name="next-steps"></a>다음 단계

* 이러한 기술을 확장하여 [속성 개체 변환기 및 수집기](./collector.md)를 구현합니다. 변환기 및 수집기 기술은 보다 일반적이며 템플릿에서 연결할 수 있습니다.
* 이 기법은 [템플릿 구성 요소 프로젝트](https://github.com/mspnp/template-building-blocks) 및 [Azure 참조 아키텍처](/azure/architecture/reference-architectures/)에서도 구현됩니다. 템플릿을 검토하여 이 기술을 구현하는 방법을 확인할 수 있습니다.

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg