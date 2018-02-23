---
title: "설명: Azure Resource Manager란?"
description: "Azure Resource Manager의 내부 작동 설명"
author: petertay
ms.openlocfilehash: 60f09901bdc4b292abd73335b78c7d56a76f27a6
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-azure-resource-manager"></a>설명: Azure Resource Manager란?

[Azure 작동 방식](azure-explainer.md) 설명에서 Azure의 내부 아키텍처에 대해 알아보았습니다. 이 아키텍처에는 내부 Azure 서비스를 관리하는 분산 응용 프로그램을 호스트하는 프런트 엔드가 포함됩니다.

Azure 프런트 엔드에는 Azure Resource Manager라는 서비스가 포함되어 있습니다. Azure Resource Manager는 생성부터 삭제까지, Azure에서 호스트되는 리소스의 수명 주기를 담당합니다. PowerShell, Azure 명령줄 인터페이스, SDK를 비롯한 다양한 방법으로 Azure Resource Manager와 상호 작용할 수 있지만, 이러한 각 도구는 Azure Resource Manager에서 호스트하는 RESTful API 위의 래퍼에 불과합니다.

Azure Resource Manager에서 제공되는 RESTful API는 **리소스 공급자** 집합에 비해 일관된 인터페이스입니다. 리소스 공급자는 단순히 Azure에서 리소스를 만들고, 읽고, 업데이트하고, 삭제하는 Azure 서비스에 불과합니다. 실제로, RESTful API에는 이러한 각 기능에 대한 메서드가 포함되어 있습니다. 

RESTful API에는 사용자의 액세스 토큰, **구독 ID**, 새로운 개념인 **리소스 그룹 ID**가 필요합니다. 리소스 그룹에 대한 설명은 [리소스 그룹 설명](resource-group-explainer.md) 문서에 나와 있습니다. Azure Resource Manager에는 액세스 토큰의 일부로 인코딩되는 **테넌트 ID**도 필요합니다. 

리소스를 생성하기 위한 유효한 API 호출이 수신되면, Azure Resource Manager는 지정된 지역에서 용량을 찾은 후 필요한 모든 파일을 스테이징 위치에 복사합니다. 그런 후에 랙의 패브릭 컨트롤러로 요청이 전송되며, 패브릭 컨트롤러는 리소스를 할당합니다. 패브릭 컨트롤러는 새로 만든 리소스에 대한 **리소스 ID**가 포함된 성공 또는 실패 알림을 사용해서 요청에 응답합니다. 이러한 네 개의 ID는 Azure에 내부적으로 저장되며, 배포된 리소스에 대한 고유 식별자로 사용됩니다.

## <a name="next-steps"></a>다음 단계

* 이제 Azure Resource Manager의 내부 기능을 이해했으므로 첫 번째 리소스 그룹을 만드는 데 도움이 되는 [리소스 그룹](resource-group-explainer.md)에 대해 알아보세요.
