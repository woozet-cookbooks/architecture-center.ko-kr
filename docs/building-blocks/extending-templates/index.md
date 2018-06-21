---
title: Azure Resource Manager 템플릿 기능 확장
description: Azure Resource Manager 템플릿 기능을 확장하는 방법에 대한 팁 및 트릭 설명
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 33ae6850ffa5b28108f30475804be5347859f0c3
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/17/2018
ms.locfileid: "29963196"
---
# <a name="extend-azure-resource-manager-template-functionality"></a><span data-ttu-id="3a747-103">Azure Resource Manager 템플릿 기능 확장</span><span class="sxs-lookup"><span data-stu-id="3a747-103">Extend Azure Resource Manager template functionality</span></span>

<span data-ttu-id="3a747-104">2016년에 Microsoft 패턴 및 사례 팀은 리소스 배포의 단순화를 목표로 Azure Resource Manager [템플릿 구성 요소](https://github.com/mspnp/template-building-blocks/wiki) 집합을 만들었습니다.</span><span class="sxs-lookup"><span data-stu-id="3a747-104">In 2016, the Microsoft patterns & practices team created a set of Azure Resource Manager [template building blocks](https://github.com/mspnp/template-building-blocks/wiki) with the goal of simplifying resource deployment.</span></span> <span data-ttu-id="3a747-105">각 구성 요소는 별도의 매개 변수 파일에 의해 지정한 리소스 집합을 배포하는 미리 작성된 템플릿 집합을 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="3a747-105">Each building block contains a set of pre-built templates that deploy sets of resources specified by separate parameter files.</span></span>

<span data-ttu-id="3a747-106">구성 요소 템플릿을 서로 결합하여 더 크고 복잡한 배포를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="3a747-106">The building block templates are designed to be combined together to create larger and more complex deployments.</span></span> <span data-ttu-id="3a747-107">예를 들어 Azure의 가상 머신을 배포하려면 가상 네트워크, 저장소 계정 및 다른 리소스가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="3a747-107">For example, deploying a virtual machine in Azure requires a virtual network, storage accounts, and other resources.</span></span> <span data-ttu-id="3a747-108">[가상 네트워크 구성 요소 템플릿](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1))은 가상 네트워크와 서브넷을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3a747-108">The [virtual network building block template](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1)) deploys a virtual network and subnets.</span></span> <span data-ttu-id="3a747-109">[가상 머신 구성 요소 템플릿](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1))은 저장소 계정, 네트워크 인터페이스 및 실제 VM을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3a747-109">The [virtual machine building block template](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1)) deploys storage accounts, network interfaces, and the actual VMs.</span></span> <span data-ttu-id="3a747-110">그런 다음 스크립트 또는 템플릿을 만들어 두 구성 요소 템플릿을 모두 해당 매개 변수 파일과 함께 호출하여 한 작업으로 전체 아키텍처를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="3a747-110">You can then create a script or template to call both building block templates with their corresponding parameter files to deploy a complete architecture with one operation.</span></span>

<span data-ttu-id="3a747-111">구성 요소 템플릿을 배포하는 동안 p&p는 여러 개념을 설계하여 Azure Resource Manager 템플릿 기능을 확장했습니다.</span><span class="sxs-lookup"><span data-stu-id="3a747-111">While developing the building block templates, p&p designed several concepts to extend Azure Resource Manager template functionality.</span></span> <span data-ttu-id="3a747-112">이 시리즈에서는 이러한 여러 개념을 사용자만의 템플릿에 사용할 수 있도록 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="3a747-112">In this series, we will describe several of these concepts so you can use them in your own templates.</span></span>

> [!NOTE]
> <span data-ttu-id="3a747-113">이러한 문서에서는 사용자가 Azure Resource Manager 템플릿을 좀 더 깊이 이해하고 있다고 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="3a747-113">These articles assume you have an advanced understanding of Azure Resource Manager templates.</span></span>