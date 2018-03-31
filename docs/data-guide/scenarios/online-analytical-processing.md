---
title: "OLAP(온라인 분석 처리)"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f5ceea9c9dd03812e92fff811e54316edc22b59c
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
---
# <a name="online-analytical-processing-olap"></a>OLAP(온라인 분석 처리)

OLAP(온라인 분석 처리)는 대규모 비즈니스 데이터베이스를 구성하고 복잡한 분석을 지원하는 기술입니다. 트랜잭션 시스템에 부정적인 영향을 주지 않고 복잡한 분석 쿼리를 수행하는 데 사용할 수 있습니다.

기업에서 모든 트랜잭션 및 레코드를 저장하는 데 사용하는 데이터베이스를 [OLTP(온라인 트랜잭션 처리)](online-transaction-processing.md) 데이터베이스라고 합니다. 일반적으로 이러한 데이터베이스의 레코드는 한 번에 하나씩 입력됩니다. 종종 이러한 데이터베이스는 조직에 귀중한 정보를 풍부하게 포함합니다. 그러나 OLTP에 사용되는 데이터베이스는 분석용으로 디자인되지 않았습니다. 따라서 이러한 데이터베이스에서 답변을 검색할 때는 시간과 노력이 많이 듭니다. OLAP 시스템은 고효율적 방식으로 데이터에서 이러한 비즈니스 인텔리전스 정보를 추출하는 데 도움이 되도록 디자인되었습니다. OLAP 데이터베이스가 과도한 읽기, 낮은 쓰기 워크로드에 최적화되어 있기 때문입니다.

![Azure의 OLAP](./images/olap-data-pipeline.png) 

## <a name="when-to-use-this-solution"></a>이 솔루션을 사용해야 하는 경우

다음과 같은 시나리오에서 OLAP를 고려하세요.

- OLTP 시스템에 부정적인 영향을 주지 않고, 복잡한 분석 및 임시 쿼리를 빠르게 실행해야 합니다. 
- 비즈니스 사용자에게 데이터에서 보고서를 생성하는 간단한 방법을 제공하려고 합니다.
- 사용자가 신속하고 일관된 결과를 얻을 수 있도록 많은 집계를 제공하려고 합니다. 

OLAP은 방대한 양의 데이터에 대해 집계 계산을 적용하는 데 특히 유용합니다. OLAP 시스템은 분석 및 비즈니스 인텔리전스 등, 과도한 읽기 시나리오에 대해 최적화되어 있습니다. OLAP는 다차원 데이터를 2차원으로 볼 수 있는 조각(예: 피벗 테이블)으로 분할하거나 데이터를 특정 값을 기준으로 필터링할 수 있도록 합니다. 이 프로세스는 데이터의 "조각화 및 분석"이라고도 하며, 데이터가 여러 데이터 원본 간에 분할되는지 여부에 관계없이 수행될 수 있습니다. 이 프로세스는 사용자들이 전형적인 데이터 분석의 세부 정보를 알지 못하더라도 추세를 알아내고, 패턴을 찾고, 데이터를 탐색하도록 합니다.

[의미 체계 모델](../concepts/semantic-modeling.md)은 비즈니스 사용자가 관계 복잡성을 추상화하고 데이터를 보다 쉽고 빠르게 분석하도록 지원할 수 있습니다.

## <a name="challenges"></a>과제

OLAP 시스템은 다양한 혜택을 제공하지만 다음과 같은 문제도 발생합니다.

- OLTP 시스템의 데이터는 다양한 원본에서 진행되는 트랜잭션을 통해 지속적으로 업데이트되지만, OLAP 데이터 저장소는 일반적으로 비즈니스 요구에 따라 훨씬 더 느린 간격으로 새로 고쳐집니다. 즉, OLAP 시스템은 변경에 대한 즉각적인 대응보다는, 전략적 비즈니스 의사 결정에 더 적합합니다. 또한 OLAP 데이터 저장소를 최신 상태로 유지하려면 일정 수준의 데이터 정리 및 오케스트레이션을 계획해야 합니다.
- OLTP 시스템의 전형적인 정규화된 관계형 테이블과 달리, OLAP 데이터 모델은 다차원이기 쉽습니다. 따라서 각 특성이 하나의 열에 매핑되는 엔터티-관계 또는 개체 지향 모델에 직접 매핑하기는 어렵거나 불가능합니다. 대신, OLAP 시스템은 일반적으로 기존의 정규화 대신 별모양 또는 눈송이 스키마를 사용합니다.

## <a name="olap-in-azure"></a>Azure의 OLAP

Azure에서 Azure SQL Database와 같은 OLTP 시스템에 포함된 데이터는 [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)와 같은 OLAP 시스템에 복사됩니다. [Power BI](https://powerbi.microsoft.com), Excel 및 타사 옵션과 같은 데이터 탐색 및 시각화 도구는 Analysis Services 서버에 연결되며, 모델링된 데이터에 대해 시각적으로 풍부한 대화형 정보를 제공합니다. OLTP 간의 데이터 흐름은 일반적으로 [Azure Data Factory](/azure/data-factory/concepts-integration-runtime)를 사용하여 실행될 수 있는 SQL Server Integration Services를 사용하여 오케스트레이션됩니다.

## <a name="technology-choices"></a>기술 선택

- [OLAP(온라인 분석 처리) 데이터 저장소](../technology-choices/olap-data-stores.md)

