---
title: 클라우드 디자인 패턴
description: Microsoft Azure에 대한 클라우드 디자인 패턴
keywords: Azure
ms.openlocfilehash: 4747c896fc6fc5866be782d76c5290d6b49ad451
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="ef5ea-104">클라우드 디자인 패턴</span><span class="sxs-lookup"><span data-stu-id="ef5ea-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="ef5ea-105">이러한 디자인 패턴은 클라우드에서 안정적이고 확장성 있는 안전한 응용 프로그램을 빌드하는 데 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="ef5ea-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="ef5ea-106">각 패턴은 패턴이 해결하는 문제, 패턴을 적용하기 위한 고려 사항 및 Microsoft Azure 기반의 예제에 대해 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="ef5ea-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="ef5ea-107">대부분의 패턴은 Azure에서 패턴을 구현하는 방법을 보여주는 코드 샘플 또는 코드 조각을 포함하고 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ef5ea-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="ef5ea-108">그러나 대부분의 패턴은 Azure에 호스팅되든 다른 클라우드 플랫폼에 호스팅되든, 분산 시스템과 관련되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ef5ea-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="ef5ea-109">클라우드의 문제 영역</span><span class="sxs-lookup"><span data-stu-id="ef5ea-109">Problem areas in the cloud</span></span>

<ul id="categories" class="panel">
<span data-ttu-id="ef5ea-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="ef5ea-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="ef5ea-111">{% include 'pattern-category-card' %}</span><span class="sxs-lookup"><span data-stu-id="ef5ea-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="ef5ea-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="ef5ea-112">{%- endfor %}</span></span>
</ul>

## <a name="catalog-of-patterns"></a><span data-ttu-id="ef5ea-113">패턴 카탈로그</span><span class="sxs-lookup"><span data-stu-id="ef5ea-113">Catalog of patterns</span></span>

| <span data-ttu-id="ef5ea-114">패턴</span><span class="sxs-lookup"><span data-stu-id="ef5ea-114">Pattern</span></span> | <span data-ttu-id="ef5ea-115">요약</span><span class="sxs-lookup"><span data-stu-id="ef5ea-115">Summary</span></span> |
|---------|---------|
|         |         |

<span data-ttu-id="ef5ea-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="ef5ea-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>
