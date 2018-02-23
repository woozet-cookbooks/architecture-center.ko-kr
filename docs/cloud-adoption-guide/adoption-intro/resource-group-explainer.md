---
title: "설명 - Azure 리소스 그룹이란?"
description: "리소스 그룹의 내부 Azure 함수 설명"
author: petertay
ms.openlocfilehash: e7c7334bd88c28f57498486bd2bed3c349565222
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/09/2018
---
# <a name="what-is-an-azure-resource-group"></a>Azure 리소스 그룹이란?

[Azure Resource Manager란?](resource-manager-explainer.md) 설명 문서에서는 리소스를 만들거나 읽거나 업데이트하거나 삭제하기 위한 호출이 수행될 때 Azure Resource Manager에 **리소스 그룹 식별자**가 필요하다는 사실을 배웠습니다. 이 리소스 그룹 ID는 **리소스 그룹**을 참조합니다. 리소스 그룹은 Azure Resource Manager가 리소스를 함께 그룹화하기 위해 적용하는 식별자에 불과합니다. 이 리소스 그룹 ID를 사용하여 Azure Resource Manager는 이 ID를 공유하는 리소스 그룹에 대해 작업을 수행할 수 있습니다.

예를 들어, 사용자는 특정 리소스 ID를 포함하지 않고 리소스 그룹 ID를 지정하여 Azure Resource Manager RESTful API에 대한 **삭제** 호출을 수행할 수 있습니다. Azure Resource Manager는 지정된 리소스 그룹 ID를 사용하여 내부 Azure 데이터베이스에서 모든 리소스를 쿼리하고, RESTful API를 호출하여 각 리소스를 삭제합니다.

리소스 그룹에는 서로 다른 구독의 리소스가 포함될 수 없습니다. 이것은 테넌트 ID와 구독 ID 간에 일대다 관계가 있기 때문입니다. 즉, 여러 구독이 동일한 테넌트를 신뢰하여 인증 및 권한 부여를 제공할 수 있지만 각 구독은 하나의 테넌트만 신뢰할 수 있습니다. 구독 ID와 리소스 그룹 ID 간에도 일대다 관계가 있습니다. 즉, 다중 리소스 그룹이 동일한 구독에 속할 수 있지만 각 리소스 그룹은 하나의 구독에만 속할 수 있습니다. 마지막으로 리소스 그룹 ID와 리소스 ID 간에는 일대다 관계가 있습니다. 즉, 단일 리소스 그룹은 여러 리소스를 포함할 수 있지만 각 리소스는 단일 리소스 그룹에만 속할 수 있습니다.

## <a name="next-steps"></a>다음 단계

* Azure 리소스 그룹에 대해 알아보았으므로 [리소스에 대한 액세스를 제한](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)하는 방법에 대한 기본적인 지식을 얻어보세요. 기본 채택 단계에 포함되지는 않지만 중급 채택 단계에서는 중요합니다. 그런 후 [첫 번째 리소스 그룹을 만들고](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json) [Azure 리소스 그룹에 대한 디자인 지침](resource-group.md)을 검토할 수 있습니다.
