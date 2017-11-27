---
title: "Azure 응용 프로그램 디자인 원칙"
description: "Azure 응용 프로그램 디자인 원칙"
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 57b04839e14804ad97fc9c86e1f9c4fe6e0da472
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="design-principles-for-azure-applications"></a>Azure 응용 프로그램 디자인 원칙

응용 프로그램의 확장성, 복원력 및 관리성을 높이려면 다음과 같은 디자인 원칙을 따르십시오. 

**[자체 복구를 위한 디자인](self-healing.md)**. 분산 시스템에서는 오류가 발생합니다. 오류가 발생하면 자체 복구되도록 응용 프로그램을 디자인하십시오.

**[모두 중복으로 구성](redundancy.md)**. 단일 실패 지점을 피하도록 응용 프로그램에 중복성을 구축합니다.
 
**[조정 최소화](minimize-coordination.md)**. 확장성을 위해 응용 프로그램 서비스 간의 조정을 최소화합니다.
 
**[규모 확장을 위한 디자인](scale-out.md)**. 수요에 따라 새 인스턴스를 추가하거나 제거하여 규모 확장이 가능하도록 응용 프로그램을 디자인합니다.

**[한도에 맞춘 분할](partition.md)**. 분할을 사용하여 데이터베이스, 네트워크 및 계산 한도를 해결합니다.

**[운영을 위한 디자인](design-for-operations.md)**. 운영 팀에 필요한 도구가 포함되도록 응용 프로그램을 디자인합니다.

**[관리되는 서비스 사용](managed-services.md)**. 가능하면 IaaS(Infrastructure as a Service)보다 PaaS(Platform as a Service)를 사용합니다.

**[작업에 가장 적합한 데이터 저장소 사용](use-the-best-data-store.md)**. 데이터에 가장 적합한 저장소 기술과 사용 방법을 선택합니다. 
 
**[진화를 위한 디자인](design-for-evolution.md)**. 모든 성공적인 응용 프로그램은 시간에 따라 변화합니다. 혁신적인 디자인은 지속적인 혁신의 핵심입니다.

**[비즈니스 요구 사항에 맞게 구축](build-for-business.md)**. 모든 디자인 결정은 비즈니스 요구 사항에 맞춰 정당화되어야 합니다.

