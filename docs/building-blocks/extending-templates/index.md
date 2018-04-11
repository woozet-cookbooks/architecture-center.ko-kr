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
---
# <a name="extend-azure-resource-manager-template-functionality"></a>Azure Resource Manager 템플릿 기능 확장

2016년에 Microsoft 패턴 및 사례 팀은 리소스 배포의 단순화를 목표로 Azure Resource Manager [템플릿 구성 요소](https://github.com/mspnp/template-building-blocks/wiki) 집합을 만들었습니다. 각 구성 요소는 별도의 매개 변수 파일에 의해 지정한 리소스 집합을 배포하는 미리 작성된 템플릿 집합을 포함합니다.

구성 요소 템플릿을 서로 결합하여 더 크고 복잡한 배포를 만듭니다. 예를 들어 Azure의 가상 머신을 배포하려면 가상 네트워크, 저장소 계정 및 다른 리소스가 필요합니다. [가상 네트워크 구성 요소 템플릿](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1))은 가상 네트워크와 서브넷을 배포합니다. [가상 머신 구성 요소 템플릿](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1))은 저장소 계정, 네트워크 인터페이스 및 실제 VM을 배포합니다. 그런 다음 스크립트 또는 템플릿을 만들어 두 구성 요소 템플릿을 모두 해당 매개 변수 파일과 함께 호출하여 한 작업으로 전체 아키텍처를 배포합니다.

구성 요소 템플릿을 배포하는 동안 p&p는 여러 개념을 설계하여 Azure Resource Manager 템플릿 기능을 확장했습니다. 이 시리즈에서는 이러한 여러 개념을 사용자만의 템플릿에 사용할 수 있도록 설명합니다.

> [!NOTE]
> 이러한 문서에서는 사용자가 Azure Resource Manager 템플릿을 좀 더 깊이 이해하고 있다고 가정합니다.