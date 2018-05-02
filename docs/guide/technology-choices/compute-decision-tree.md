---
title: Azure 계산 서비스에 대한 의사 결정 트리
description: 계산 서비스를 선택하기 위한 순서도
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: f5703f4906ca2ea6f825b383710eb4bd335f5043
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/23/2018
---
# <a name="decision-tree-for-azure-compute-services"></a>Azure 계산 서비스에 대한 의사 결정 트리

Azure는 응용 프로그램 코드를 호스트하는 다양한 방법을 제공합니다. *계산*이라는 용어는 응용 프로그램이 실행되는 계산 리소스의 호스팅 모델을 말합니다. 다음 순서도는 응용 프로그램에 대한 계산 서비스를 선택하는 데 도움이 됩니다.
 
![](../images/compute-decision-tree.svg)

순서도는 권장 사항에 연결할 주요 의사 결정 기준의 집합을 안내합니다. 모든 응용 프로그램에는 고유한 요구 사항이 있으므로 권장 사항을 시작점으로 처리해야 합니다. 그런 다음, 다음과 같은 측면을 살펴보는 보다 자세한 분석을 수행합니다.
 
- 기능 집합
- [서비스 한도](/azure/azure-subscription-service-limits)
- [비용](https://azure.microsoft.com/pricing/)
- [SLA](https://azure.microsoft.com/support/legal/sla/)
- [국가별 가용성](https://azure.microsoft.com/global-infrastructure/services/)
- 개발자 에코시스템 및 팀 기술
- [계산 비교 표](./compute-comparison.md)

응용 프로그램이 여러 워크로드로 구성된 경우 각 워크로드를 개별적으로 평가합니다. 완벽한 솔루션은 두 개 이상의 계산 서비스를 통합할 수 있습니다.

