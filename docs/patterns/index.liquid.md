---
title: "클라우드 디자인 패턴"
description: "Microsoft Azure에 대한 클라우드 디자인 패턴"
keywords: Azure
ms.openlocfilehash: 264b8296a428f9c1b87314b782efcabc89cf010f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="cloud-design-patterns"></a>클라우드 디자인 패턴

[!INCLUDE [header](../../_includes/header.md)]

이러한 디자인 패턴은 클라우드에서 안정적이고 확장성 있는 안전한 응용 프로그램을 빌드하는 데 유용합니다.

각 패턴은 패턴이 해결하는 문제, 패턴을 적용하기 위한 고려 사항 및 Microsoft Azure 기반의 예제에 대해 설명합니다. 대부분의 패턴은 Azure에서 패턴을 구현하는 방법을 보여주는 코드 샘플 또는 코드 조각을 포함하고 있습니다. 그러나 대부분의 패턴은 Azure에 호스팅되든 다른 클라우드 플랫폼에 호스팅되든, 분산 시스템과 관련되어 있습니다.

## <a name="problem-areas-in-the-cloud"></a>클라우드의 문제 영역

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include 'pattern-category-card' %}
    </li>
{%- endfor %}
</ul>

## <a name="catalog-of-patterns"></a>패턴 카탈로그

| 패턴 | 요약 |
| ------- | ------- |
{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}