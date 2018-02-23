---
title: "Azure Data Architecture 가이드"
description: 
author: zoinerTejada
ms:date: 02/12/2018
layout: LandingPage
ms.openlocfilehash: 848601f27faf56ea069852d8983e4d10fbad9d77
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
---
# <a name="azure-data-architecture-guide"></a>Azure Data Architecture 가이드

이 가이드에서는 Microsoft Azure에서 데이터 중심 솔루션을 디자인하는 구조적 접근 방식을 제시합니다. 이 가이드는 고객의 참여를 통해 얻은 사례를 기반으로 합니다.

## <a name="introduction"></a>소개

클라우드는 데이터의 처리 및 저장 방법을 비롯하여 응용 프로그램이 디자인되는 방식을 바꾸고 있습니다. _polyglot 지속성_ 솔루션은 모든 솔루션의 데이터를 처리하는 단일 범용 데이터베이스 대신, 각각이 특정 기능을 제공하도록 최적화된 여러 특수한 데이터 저장소를 사용합니다. 이 솔루션의 데이터에 대한 큐브 뷰는 결과적으로 변경됩니다. 단일 데이터 계층에서 읽고 쓰는 비즈니스 논리의 여러 계층이 더 이상 존재하지 않습니다. 대신, 솔루션을 통해 데이터가 흐르는 방식, 처리되는 위치, 저장되는 위치, 파이프라인의 다음 구성 요소에서 사용되는 방식을 설명하는 *데이터 파이프라인*을 중심으로 솔루션이 디자인됩니다. 

## <a name="how-this-guide-is-structured"></a>이 가이드의 구조

이 가이드는 *관계형* 데이터와 *비관계형* 데이터 간을 구분하는 기본 개념을 중심으로 구성되었습니다. 

![](./images/guide-steps.svg)

일반적으로 관계형 데이터는 전형적인 RDBMS 또는 데이터 웨어하우스에 저장됩니다. 참조 무결성을 유지하기 위해 제약 조건 집합으로 미리 정의된 스키마("쓰기 시 스키마")를 포함합니다. 대부분의 관계형 데이터베이스는 쿼리를 위해 SQL(구조적 쿼리 언어)을 사용합니다. 관계형 데이터베이스를 사용하는 솔루션에는 OLTP(온라인 트랜잭션 처리) 및 OLAP(온라인 분석 처리)가 포함됩니다.

비관계형 데이터는 전형적인 RDBMS 시스템에 있는 [관계형 모델](https://en.wikipedia.org/wiki/Relational_model)을 사용하지 않는 모든 데이터입니다. 여기에는 키-값 데이터, JSON 데이터, 그래프 데이터, 시계열 데이터 및 기타 데이터 형식이 포함될 수 있습니다. 용어 *NoSQL*은 다양한 종류의 비관계형 데이터를 포함하도록 디자인된 데이터베이스를 나타냅니다. 그러나 많은 비관계형 데이터 저장소가 SQL 호환 쿼리를 지원하기 때문에 이 용어가 완전히 정확하다고 할 수는 없습니다. 비관계형 데이터 및 NoSQL 데이터베이스는 종종 *빅 데이터* 솔루션을 논의할 때 거론됩니다. 빅 데이터 아키텍처는 기존의 데이터베이스 시스템에 비해 너무 크거나 복잡한 데이터의 수집, 처리 및 분석을 수행하도록 디자인되었습니다. 

이러한 두 가지 주요 범주와 관련해서 데이터 아키텍처 가이드는 다음 섹션을 포함합니다.

- **개념.** 이러한 형식의 데이터를 사용할 때 이해해야 하는 주요 개념을 소개하는 개요 문서입니다.
- **시나리오.** 관련 Azure 서비스를 논의하는 대표적인 데이터 시나리오 모음과 시나리오에 적합한 아키텍처를 소개합니다.
- **기술 선택.** 오픈 소스 옵션을 포함하여 Azure에서 사용할 수 있는 다양한 데이터 기술을 자세히 비교합니다. 시나리오에 적합한 기술을 선택하는 데 도움을 주기 위해 범주별로 주요 선택 기준과 기능 매트릭스가 제시됩니다.

이 가이드는 데이터 과학 또는 데이터베이스 이론을 학습하기 위한 것이 아닙니다. 이러한 주제에 대한 전체 설명서를 찾아볼 수 있습니다. 이 가이드는 시나리오에 적합한 데이터 아키텍처 또는 데이터 파이프라인을 선택한 다음, 요구에 가장 잘 맞는 Azure 서비스 및 기술을 선택하는 데 도움을 주기 위해 작성되었습니다. 이미 아키텍처를 고려한 경우에는 기술 선택 단계로 바로 건너뛸 수 있습니다.

## <a name="traditional-rdbms"></a>일반적인 RDBMS

### <a name="concepts"></a>개념

- [관계형 데이터](./concepts/relational-data.md) 
- [트랜잭션 데이터](./concepts/transactional-data.md) 
- [의미 체계 모델링](./concepts/semantic-modeling.md) 

### <a name="scenarios"></a>시나리오

- [OLAP(온라인 분석 처리)](./scenarios/online-analytical-processing.md)
- [OLTP(온라인 트랜잭션 처리)](./scenarios/online-transaction-processing.md) 
- [데이터 웨어하우징 및 데이터 마트](./scenarios/data-warehousing.md)
- [ETL](./scenarios/etl.md) 

## <a name="big-data-and-nosql"></a>빅 데이터 및 NoSQL

### <a name="concepts"></a>개념

- [비관계형 데이터 저장소](./concepts/non-relational-data.md)
- [CSV 및 JSON 파일 사용](./concepts/csv-and-json.md)
- [빅 데이터 아키텍처](./concepts/big-data.md)
- [고급 분석](./concepts/advanced-analytics.md) 
- [규모에 맞는 기계 학습](./concepts/machine-learning-at-scale.md)

### <a name="scenarios"></a>시나리오

- [일괄 처리](./scenarios/batch-processing.md)
- [실시간 처리](./scenarios/real-time-processing.md)
- [자유 형식 텍스트 검색](./scenarios/search.md)
- [대화형 데이터 탐색](./scenarios/interactive-data-exploration.md)
- [자연어 처리](./scenarios/natural-language-processing.md)
- [시계열 솔루션](./scenarios/time-series.md)

## <a name="cross-cutting-concerns"></a>복합적인 문제

- [데이터 전송](./scenarios/data-transfer.md) 
- [클라우드로 온-프레미스 데이터 솔루션 확장](./scenarios/hybrid-on-premises-and-cloud.md) 
- [데이터 솔루션 보안](./scenarios/securing-data-solutions.md) 
