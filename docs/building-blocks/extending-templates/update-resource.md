---
title: Azure Resource Manager 템플릿의 리소스 업데이트
description: Azure Resource Manager 템플릿의 기능을 확장하여 리소스를 업데이트하는 방법을 설명합니다.
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: fc2565819c66ee7695224ef5793e7276e6e552e0
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="f002a-103">Azure Resource Manager 템플릿의 리소스 업데이트</span><span class="sxs-lookup"><span data-stu-id="f002a-103">Update a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="f002a-104">배포하는 동안 리소스를 업데이트해야 하는 몇 가지 시나리오가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-104">There are some scenarios in which you need to update a resource during a deployment.</span></span> <span data-ttu-id="f002a-105">다른 종속 리소스를 만들 때까지 리소스에 대한 모든 속성을 지정할 수는 없을 때 이 시나리오가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-105">You might encounter this scenario when you cannot specify all the properties for a resource until other, dependent resources are created.</span></span> <span data-ttu-id="f002a-106">예를 들어 부하 분산 장치에 대한 백 엔드 풀을 만드는 경우 VM(가상 머신)의 NIC(네트워크 인터페이스)를 업데이트하여 백 엔드 풀에 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-106">For example, if you create a backend pool for a load balancer, you might update the network interfaces (NICs) on your virtual machines (VMs) to include them in the backend pool.</span></span> <span data-ttu-id="f002a-107">또한 Resource Manager는 배포 동안 리소스를 업데이트하도록 지원하지만 오류를 피하고 배포가 업데이트로 처리되도록 템플릿을 적절히 디자인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-107">And while Resource Manager supports updating resources during deployment, you must design your template correctly to avoid errors and to ensure the deployment is handled as an update.</span></span>

<span data-ttu-id="f002a-108">첫째, 리소스를 만들려면 템플릿에서 리소스를 한 번 참조한 후, 나중에 업데이트할 때 같은 이름으로 리소스를 참조해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-108">First, you must reference the resource once in the template to create it and then reference the resource by the same name to update it later.</span></span> <span data-ttu-id="f002a-109">그러나 두 리소스가 템플릿에서 같은 이름을 갖는 경우 Resource Manager는 예외를 throw합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-109">However, if two resources have the same name in a template, Resource Manager throws an exception.</span></span> <span data-ttu-id="f002a-110">이 오류를 방지하려면 `Microsoft.Resources/deployments` 리소스 형식을 사용하여 하위 템플릿으로 연결 또는 포함된 두 번째 템플릿에 업데이트된 리소스를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-110">To avoid this error, specify the updated resource in a second template that's either linked or included as a subtemplate using the `Microsoft.Resources/deployments` resource type.</span></span>

<span data-ttu-id="f002a-111">둘째, 중첩된 템플릿에서 변경할 기존 속성의 이름 또는 추가할 속성의 새 이름을 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-111">Second, you must either specify the name of the existing property to change or a new name for a property to add in the nested template.</span></span> <span data-ttu-id="f002a-112">또한 원래 속성 및 원래 값도 함께 지정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-112">You must also specify the original properties and their original values.</span></span> <span data-ttu-id="f002a-113">원래 속성 및 값을 제공하지 못하면 Resource Manager는 사용자가 새 리소스를 만들려고 한다고 가정하고 원래 리소스를 삭제합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-113">If you fail to provide the original properties and values, Resource Manager assumes you want to create a new resource and deletes the original resource.</span></span>

## <a name="example-template"></a><span data-ttu-id="f002a-114">예제 템플릿</span><span class="sxs-lookup"><span data-stu-id="f002a-114">Example template</span></span>

<span data-ttu-id="f002a-115">이 내용을 설명하는 예제 템플릿을 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-115">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="f002a-116">템플릿은 `firstSubnet`이라는 서브넷이 있는 `firstVNet`이라는 VNet(가상 네트워크)을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-116">Our template deploys a virtual network  named `firstVNet` that has one subnet named `firstSubnet`.</span></span> <span data-ttu-id="f002a-117">그런 후 `nic1`라는 가상 NIC(네트워크 인터페이스)를 배포하고 해당 서브넷에 연결합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-117">It then deploys a virtual network interface (NIC) named `nic1` and associates it with our subnet.</span></span> <span data-ttu-id="f002a-118">그런 다음 `secondSubnet`이라는 두 번째 서브넷을 추가하여 `updateVNet`라는 배포 리소스에 `firstVNet` 리소스를 업데이트하는 중첩된 템플릿이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-118">Then, a deployment resource named `updateVNet` includes a nested template that updates our `firstVNet` resource by adding a second subnet named `secondSubnet`.</span></span> 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
      {
      "apiVersion": "2016-03-30",
      "name": "firstVNet",
      "location":"[resourceGroup().location]",
      "type": "Microsoft.Network/virtualNetworks",
      "properties": {
          "addressSpace":{"addressPrefixes": [
              "10.0.0.0/22"
          ]},
          "subnets":[              
              {
                  "name":"firstSubnet",
                  "properties":{
                    "addressPrefix":"10.0.0.0/24"
                  }
              }
            ]
      }
    },
    {
        "apiVersion": "2015-06-15",
        "type":"Microsoft.Network/networkInterfaces",
        "name":"nic1",
        "location":"[resourceGroup().location]",
        "dependsOn": [
            "firstVNet"
        ],
        "properties": {
            "ipConfigurations":[
                {
                    "name":"ipconfig1",
                    "properties": {
                        "privateIPAllocationMethod":"Dynamic",
                        "subnet": {
                            "id": "[concat(resourceId('Microsoft.Network/virtualNetworks','firstVNet'),'/subnets/firstSubnet')]"
                        }
                    }
                }
            ]
        }
    },
    {
      "apiVersion": "2015-01-01",
      "type": "Microsoft.Resources/deployments",
      "name": "updateVNet",
      "dependsOn": [
          "nic1"
      ],
      "properties": {
        "mode": "Incremental",
        "parameters": {},
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
              {
                  "apiVersion": "2016-03-30",
                  "name": "firstVNet",
                  "location":"[resourceGroup().location]",
                  "type": "Microsoft.Network/virtualNetworks",
                  "properties": {
                      "addressSpace": "[reference('firstVNet').addressSpace]",
                      "subnets":[
                          {
                              "name":"[reference('firstVNet').subnets[0].name]",
                              "properties":{
                                  "addressPrefix":"[reference('firstVNet').subnets[0].properties.addressPrefix]"
                                  }
                          },
                          {
                              "name":"secondSubnet",
                              "properties":{
                                  "addressPrefix":"10.0.1.0/24"
                                  }
                          }
                     ]
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

<span data-ttu-id="f002a-119">먼저 `firstVNet` 리소스의 리소스 개체를 살펴보겠습니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-119">Let's take a look at the resource object for our `firstVNet` resource first.</span></span> <span data-ttu-id="f002a-120">중첩된 템플릿에서 `firstVNet`에 대한 설정을 지정합니다. Resource Manager는 같은 템플릿 내에서 같은 배포 이름을 허용하지 않으며 중첩된 템플릿은 다른 템플릿으로 간주되기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-120">Notice that we respecify the settings for our `firstVNet` in a nested template&mdash;this is because Resource Manager doesn't allow the same deployment name within the same template and nested templates are considered to be a different template.</span></span> <span data-ttu-id="f002a-121">`firstSubnet` 리소스에 대한 값을 지정하여 Resource Manager에 기존 리소스를 삭제한 후 다시 배포하는 대신, 업데이트하도록 지시합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-121">By respecifying our values for our `firstSubnet` resource, we are telling Resource Manager to update the existing resource instead of deleting it and redeploying it.</span></span> <span data-ttu-id="f002a-122">마지막으로, `secondSubnet`에 대한 새 설정이 이 업데이트 동안 선택됩니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-122">Finally, our new settings for `secondSubnet` are picked up during this update.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="f002a-123">템플릿 시도</span><span class="sxs-lookup"><span data-stu-id="f002a-123">Try the template</span></span>

<span data-ttu-id="f002a-124">이 템플릿으로 실험하려면 다음 단계를 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-124">If you would like to experiment with this template, follow these steps:</span></span>

1.  <span data-ttu-id="f002a-125">Azure Portal로 이동하여 **+** 아이콘을 선택하고 **템플릿 배포** 리소스 종류를 검색하고 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-125">Go to the Azure portal, select the **+** icon, and search for the **template deployment** resource type, and select it.</span></span>
2.  <span data-ttu-id="f002a-126">**템플릿 배포** 페이지로 이동한 후 **만들기** 단추를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-126">Navigate to the **template deployment** page, select the **create** button.</span></span> <span data-ttu-id="f002a-127">그러면 **사용자 지정 배포** 블레이드가 열립니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-127">This button opens the **custom deployment** blade.</span></span>
3.  <span data-ttu-id="f002a-128">**편집** 아이콘을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-128">Select the **edit** icon.</span></span>
4.  <span data-ttu-id="f002a-129">비어 있는 템플릿을 삭제합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-129">Delete the empty template.</span></span>
5.  <span data-ttu-id="f002a-130">샘플 템플릿을 복사하여 오른쪽 창에 붙여 넣습니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-130">Copy and paste the sample template into the right pane.</span></span>
6.  <span data-ttu-id="f002a-131">**저장** 단추를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-131">Select the **save** button.</span></span>
7.  <span data-ttu-id="f002a-132">**사용자 지정 배포**창으로 돌아오지만 이번에는 일부 드롭다운 상자가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-132">You return to the **custom deployment** pane, but this time there are some drop-down list boxes.</span></span> <span data-ttu-id="f002a-133">구독을 선택하고, 새로 만들거나 기존 리소스 그룹을 사용하고, 위치를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-133">Select your subscription, either create new or use existing resource group, and select a location.</span></span> <span data-ttu-id="f002a-134">사용 약관을 검토하고 **동의함** 단추를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-134">Review the terms and conditions, then select the **I agree** button.</span></span>
8.  <span data-ttu-id="f002a-135">**구입** 단추를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-135">Select the **purchase** button.</span></span>

<span data-ttu-id="f002a-136">배포가 완료된 후 포털에서 지정한 리소스 그룹을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-136">Once deployment has finished, open the resource group you specified in the portal.</span></span> <span data-ttu-id="f002a-137">`firstVNet`라는 가상 네트워크와 `nic1`라는 NIC가 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-137">You see a virtual network named `firstVNet` and a NIC named `nic1`.</span></span> <span data-ttu-id="f002a-138">`firstVNet`을 클릭한 후 `subnets`를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-138">Click `firstVNet`, then click `subnets`.</span></span> <span data-ttu-id="f002a-139">처음에 만들어진 `firstSubnet`이 표시된 후 `updateVNet` 리소스에 추가된 `secondSubnet`이 표시됩니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-139">You see the `firstSubnet` that was originally created, and you see the `secondSubnet` that was added in the `updateVNet` resource.</span></span> 

![원래 서브넷 및 업데이트된 서브넷](../_images/firstVNet-subnets.png)

<span data-ttu-id="f002a-141">그런 후 리소스 그룹으로 다시 이동하고 `nic1`을 클릭한 후 `IP configurations`를 클릭합니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-141">Then, go back to the resource group and click `nic1` then click `IP configurations`.</span></span> <span data-ttu-id="f002a-142">`IP configurations` 섹션에서 `subnet`은 `firstSubnet (10.0.0.0/24)`으로 설정되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-142">In the `IP configurations` section, the `subnet` is set to `firstSubnet (10.0.0.0/24)`.</span></span> 

![nic1 IP configurations 설정](../_images/nic1-ipconfigurations.png)

<span data-ttu-id="f002a-144">원래 `firstVNet`이 다시 생성되지 않고 업데이트되었습니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-144">The original `firstVNet` has been updated instead of recreated.</span></span> <span data-ttu-id="f002a-145">`firstVNet`이 다시 생성되었으면 `nic1`이 `firstVNet`에 연결되지 않았을 것입니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-145">If `firstVNet` had been recreated, `nic1` would not be associated with `firstVNet`.</span></span>

## <a name="next-steps"></a><span data-ttu-id="f002a-146">다음 단계</span><span class="sxs-lookup"><span data-stu-id="f002a-146">Next steps</span></span>

* <span data-ttu-id="f002a-147">이 기법은 [템플릿 구성 요소 프로젝트](https://github.com/mspnp/template-building-blocks) 및 [Azure 참조 아키텍처](/azure/architecture/reference-architectures/)에서도 구현됩니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-147">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="f002a-148">이러한 참조 아키텍처를 사용하여 고유한 아키텍처를 만들거나 참조 아키텍처 중 하나를 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="f002a-148">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>
