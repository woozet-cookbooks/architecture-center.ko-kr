---
title: OLAP(온라인 분석 처리)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 92b71934f2081e95c3c9b0d4dc9edeb3885b12e8
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846810"
---
# <a name="online-analytical-processing-olap"></a>OLAP(온라인 분석 처리)

OLAP(온라인 분석 처리)는 대규모 비즈니스 데이터베이스를 구성하고 복잡한 분석을 지원하는 기술입니다. 트랜잭션 시스템에 부정적인 영향을 주지 않고 복잡한 분석 쿼리를 수행하는 데 사용할 수 있습니다.

기업에서 모든 트랜잭션 및 레코드를 저장하는 데 사용하는 데이터베이스를 [OLTP(온라인 트랜잭션 처리)](online-transaction-processing.md) 데이터베이스라고 합니다. 일반적으로 이러한 데이터베이스의 레코드는 한 번에 하나씩 입력됩니다. 종종 이러한 데이터베이스는 조직에 귀중한 정보를 풍부하게 포함합니다. 그러나 OLTP에 사용되는 데이터베이스는 분석용으로 디자인되지 않았습니다. 따라서 이러한 데이터베이스에서 답변을 검색할 때는 시간과 노력이 많이 듭니다. OLAP 시스템은 고효율적 방식으로 데이터에서 이러한 비즈니스 인텔리전스 정보를 추출하는 데 도움이 되도록 디자인되었습니다. OLAP 데이터베이스가 과도한 읽기, 낮은 쓰기 워크로드에 최적화되어 있기 때문입니다.

![Azure의 OLAP](../images/olap-data-pipeline.png) 

## <a name="semantic-modeling"></a>의미 체계 모델링

의미 체계 데이터 모델은 포함된 데이터 요소의 의미를 설명하는 개념적 모델입니다. 조직에서는 종종 사물에 대해 자체 용어를 사용하며 동의어를 사용하기도 하고, 같은 용어에 대해 다른 의미를 사용할 때도 있습니다. 예를 들어, 재고 데이터베이스는 자산 ID와 일련 번호를 사용해서 장비를 추적할 수 있지만 판매 데이터베이스는 일련 번호를 자산 ID로 나타낼 수 있습니다. 관계를 설명하는 모델 없이 이러한 값을 연관짓는 간단한 방식은 없습니다. 

의미 체계 모델링은 데이터베이스 스키마에 대해 추상화 수준을 제공하므로, 사용자는 기본 데이터 구조를 알 필요가 없습니다. 따라서 최종 사용자는 기본 스키마에 대해 집계 및 조인을 수행하지 않고도 데이터를 보다 쉽게 쿼리할 수 있습니다. 또한 일반적으로 열 이름이 좀 더 친숙한 이름으로 바뀌므로 데이터의 컨텍스트 및 의미가 좀 더 명확해집니다.

의미 체계 모델링은 좀 더 쓰기 집약적인 트랜잭션 데이터 처리(OLTP)와 달리, 분석 및 비즈니스 인텔리전스(OLAP)와 같은 읽기 집약적인 시나리오에 더 많이 사용됩니다. 이러한 사실은 다음과 같은 일반적인 의미 체계 계층의 특정 때문입니다.

- 보고 도구에 제대로 표시되도록 집계 동작이 설정됩니다.
- 비즈니스 논리 및 계산이 정의됩니다.
- 시간 기반 계산이 포함됩니다.
- 종종 여러 원본의 데이터가 통합됩니다. 

일반적으로 의미 체계 계층은 이러한 이유로 데이터 웨어하우스 위에 배치됩니다.

![데이터 웨어하우스와 보고 도구 간의 의미 체계 계층 예제 다이어그램](../images/semantic-modeling.png)

의미 체계 모델에는 다음과 같은 2가지 기본 유형이 있습니다.

* **테이블 형식**. 관계형 모델링 구문(모델, 테이블, 열)을 사용합니다. 내부적으로 메타데이터는 OLAP 모델링 구문(큐브, 차원, 측정값)에서 상속됩니다. 코드 및 스크립트에는 OLAP 메타데이터를 사용합니다.
* **다차원**. 기존 OLAP 모델링 구문(큐브, 차원, 측정값)을 사용합니다.

관련 Azure 서비스:
- [Azure Analysis Services](https://azure.microsoft.com/services/analysis-services/)

## <a name="example-use-case"></a>사용 사례

조직의 데이터가 대형 데이터베이스에 저장되어 있습니다. 비즈니스 사용자 및 고객이 자체 보고서를 만들고 분석을 수행하는 데 이 데이터를 사용할 수 있게 하려고 합니다. 한 가지 방법은 해당 사용자에게 데이터베이스에 대한 직접 액세스 권한을 제공하는 것입니다. 그러나 이러한 방식은 보안 관리 및 액세스 제어를 비롯한 몇 가지 단점이 있습니다. 또한 테이블 및 열 이름을 포함하는 데이터베이스의 디자인을 사용자가 이해하기 어려울 수 있습니다. 사용자는 쿼리할 테이블, 해당 테이블이 조인되는 방법, 올바른 결과를 얻기 위해 적용해야 하는 기타 비즈니스 논리를 알고 있어야 합니다. 또한 시작하기 위해 SQL과 같은 쿼리 언어를 알고 있어야 합니다. 일반적으로 이러한 경우 여러 사용자가 다른 결과로 동일한 메트릭을 보고하게 됩니다.

또 다른 옵션은 사용자에게 필요한 모든 정보를 의미 체계 모델에 캡슐화하는 것입니다. 의미 체계 모델은 사용자가 선택한 보고 도구를 사용해서 보다 쉽게 쿼리할 수 있습니다. 의미 체계 모델이 제공하는 데이터는 데이터 웨어하우스에서 끌어오므로, 모든 사용자가 신뢰할 수 있는 한 가지 버전을 보게 됩니다. 또한 의미 체계 모델은 친숙한 테이블 및 열 이름, 테이블 간 관계, 설명, 계산 및 행 수준 보안을 제공합니다.

## <a name="typical-traits-of-semantic-modeling"></a>의미 체계 모델링의 일반적인 특성

의미 체계 모델링 및 분석 처리는 다음과 같은 특성을 가질 수 있습니다.

| 요구 사항 | 설명 |
| --- | --- |
| 스키마 | 쓰기 시 스키마, 강력하게 적용|
| 트랜잭션 사용 | 아니오 |
| 잠금 전략 | 없음 |
| 업데이트 가능 | 아니요(일반적으로 큐브를 다시 계산해야 함) |
| 추가 가능 | 아니요(일반적으로 큐브를 다시 계산해야 함) |
| 워크로드 | 과도한 읽기, 읽기 전용 |
| 인덱싱 | 다차원 인덱싱 |
| 데이터 크기 | 소규모~중간 규모 |
| 모델 | 다차원 |
| 데이터 모양:| 큐브 또는 별/눈송이 스키마 |
| 쿼리 유연성 | 매우 유연 |
| 크기: | 큼(10s-100s GB) |

## <a name="when-to-use-this-solution"></a>이 솔루션을 사용해야 하는 경우

다음과 같은 시나리오에서 OLAP를 고려하세요.

- OLTP 시스템에 부정적인 영향을 주지 않고, 복잡한 분석 및 임시 쿼리를 빠르게 실행해야 합니다. 
- 비즈니스 사용자에게 데이터에서 보고서를 생성하는 간단한 방법을 제공하려고 합니다.
- 사용자가 신속하고 일관된 결과를 얻을 수 있도록 많은 집계를 제공하려고 합니다. 

OLAP은 방대한 양의 데이터에 대해 집계 계산을 적용하는 데 특히 유용합니다. OLAP 시스템은 분석 및 비즈니스 인텔리전스 등, 과도한 읽기 시나리오에 대해 최적화되어 있습니다. OLAP는 다차원 데이터를 2차원으로 볼 수 있는 조각(예: 피벗 테이블)으로 분할하거나 데이터를 특정 값을 기준으로 필터링할 수 있도록 합니다. 이 프로세스는 데이터의 "조각화 및 분석"이라고도 하며, 데이터가 여러 데이터 원본 간에 분할되는지 여부에 관계없이 수행될 수 있습니다. 이 프로세스는 사용자들이 전형적인 데이터 분석의 세부 정보를 알지 못하더라도 추세를 알아내고, 패턴을 찾고, 데이터를 탐색하도록 합니다.

의미 체계 모델은 비즈니스 사용자가 관계 복잡성을 추상화하고 데이터를 보다 쉽고 빠르게 분석하도록 지원할 수 있습니다.

## <a name="challenges"></a>과제

OLAP 시스템은 다양한 혜택을 제공하지만 다음과 같은 문제도 발생합니다.

- OLTP 시스템의 데이터는 다양한 원본에서 진행되는 트랜잭션을 통해 지속적으로 업데이트되지만, OLAP 데이터 저장소는 일반적으로 비즈니스 요구에 따라 훨씬 더 느린 간격으로 새로 고쳐집니다. 즉, OLAP 시스템은 변경에 대한 즉각적인 대응보다는, 전략적 비즈니스 의사 결정에 더 적합합니다. 또한 OLAP 데이터 저장소를 최신 상태로 유지하려면 일정 수준의 데이터 정리 및 오케스트레이션을 계획해야 합니다.
- OLTP 시스템의 전형적인 정규화된 관계형 테이블과 달리, OLAP 데이터 모델은 다차원이기 쉽습니다. 따라서 각 특성이 하나의 열에 매핑되는 엔터티-관계 또는 개체 지향 모델에 직접 매핑하기는 어렵거나 불가능합니다. 대신, OLAP 시스템은 일반적으로 기존의 정규화 대신 별모양 또는 눈송이 스키마를 사용합니다.

## <a name="olap-in-azure"></a>Azure의 OLAP

Azure에서 Azure SQL Database와 같은 OLTP 시스템에 포함된 데이터는 [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)와 같은 OLAP 시스템에 복사됩니다. [Power BI](https://powerbi.microsoft.com), Excel 및 타사 옵션과 같은 데이터 탐색 및 시각화 도구는 Analysis Services 서버에 연결되며, 모델링된 데이터에 대해 시각적으로 풍부한 대화형 정보를 제공합니다. OLTP 간의 데이터 흐름은 일반적으로 [Azure Data Factory](/azure/data-factory/concepts-integration-runtime)를 사용하여 실행될 수 있는 SQL Server Integration Services를 사용하여 오케스트레이션됩니다.

Azure에서 다음의 모든 데이터 저장소는 OLAP의 요구 사항을 충족합니다.

- [SQL Server(columnstore 인덱스 포함)](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)
- [Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [SSAS(SQL Server Analysis Services)](/sql/analysis-services/analysis-services)

SSAS(SQL Server Analysis Services)는 비즈니스 인텔리전스 응용 프로그램에 대한 OLAP 및 데이터 마이닝 기능을 제공합니다. 로컬 서버에서 SSAS를 설치할 수도 있고, Azure의 가상 머신 내에 호스트할 수도 있습니다. Azure Analysis Services는 SSAS와 같은 주요 기능을 제공하는 완전히 관리되는 서비스입니다. Azure Analysis Services는 조직의 클라우드 및 온-프레미스에 있는 [다양한 데이터 원본](/azure/analysis-services/analysis-services-datasource)에 대한 연결을 지원합니다.

클러스터형 columnstore 인덱스는 Azure SQL Database 뿐만 아니라 SQL Server 2014 이상에서 사용할 수 있으며, OLAP 작업에 이상적입니다. 그러나 SQL Server 2016(Azure SQL Database 포함)부터 업데이트 가능 비클러스터형 columnstore 인덱스를 사용하여 HTAP(하이브리드 트랜잭션/분석 처리)를 활용할 수 있습니다. HTAP를 사용하면 동일한 플랫폼에서 OLTP 및 OLAP 처리를 수행할 수 있으므로, 데이터의 여러 복사본을 저장할 필요가 없으며, 고유한 OLTP 및 OLAP 시스템이 없어도 됩니다. 자세한 내용은 [실시간 운영 분석을 위한 columnstore 시작](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)을 참조하세요.

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 사용자 고유의 서버를 관리하지 않고 관리되는 서비스를 원하시나요?

- Azure AD(Azure Active Directory)를 사용하는 보안 인증이 필요한가요?

- 실시간 분석을 수행하려고 하나요? 그렇다면 실시간 분석을 지원하는 옵션으로 범위를 줄입니다. 

    이 컨텍스트의 *실시간 분석*은 운영 및 분석 워크로드를 둘 다 실행하는 ERP(전사적 자원 관리) 응용 프로그램 같은 단일 데이터 원본에 적용됩니다. 여러 원본의 데이터를 통합해야 하거나 큐브와 같은 사전 집계된 데이터를 사용하여 분석 성능을 높여야 할 경우 여전히 별도의 데이터 웨어하우스가 필요할 수 있습니다.

- 예를 들어, 비즈니스 사용자가 보다 편리하게 분석을 수행할 수 있도록 하는 의미 체계 모델을 제공하기 위해 미리 집계된 데이터를 사용해야 하나요? 그렇다면 다차원 큐브 또는 테이블 형식의 의미 체계 모델을 지원하는 옵션을 선택합니다. 

    집계를 제공하면 데이터 집계를 일관되게 계산하는 데 도움이 될 수 있습니다. 또한 미리 집계된 데이터는 여러 행의 여러 열을 처리할 때 성능을 크게 향상시킬 수 있습니다. 데이터는 다차원 큐브 또는 테이블 형식 의미 체계 모델에 사전 집계될 수 있습니다.

- OLTP 데이터 저장소 이외의 여러 원본에 있는 데이터를 통합해야 하나요? 그렇다면 여러 데이터 원본을 쉽게 통합하는 옵션을 고려합니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.

### <a name="general-capabilities"></a>일반 기능

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server(columnstore 인덱스 포함) | Azure SQL Database(columnstore 인덱스 포함) |
| --- | --- | --- | --- | --- |
| 관리되는 서비스인지 여부 | 예 | 아니오 | 아니요 | 예 |
| 다차원 큐브 지원 여부 | 아니오 | 예 | 아니요 | 아니오 |
| 테이블 형식 의미 체계 모델 지원 여부 | 예 | 예 | 아니오 | 아니오 |
| 여러 데이터 원본을 쉽게 통합 | 예 | 예 | 아니요 <sup>1</sup> | 아니요 <sup>1</sup> |
| 실시간 분석 지원 | 아니오 | 아니요 | 예 | 예 |
| 원본에서 데이터를 복사하는 프로세스 필요 | 예 | 예 | 아니오 | 아니요 |
| Azure AD 통합 | 예 | 아니오 | 아니요 <sup>2</sup> | 예 |

[1] SQL Server 및 Azure SQL Database는 여러 외부 데이터 원본에서 쿼리하거나 이러한 원본을 통합하는 데 사용할 수 없지만, [SSIS](/sql/integration-services/sql-server-integration-services) 또는 [Azure Data Factory](/azure/data-factory/)를 사용하여 이 작업을 자동으로 수행하는 파이프라인을 여전히 구축할 수 있습니다. Azure VM에서 호스트되는 SQL Server의 경우 연결된 서버 및 [PolyBase](/sql/relational-databases/polybase/polybase-guide)와 같은 추가 옵션을 사용할 수 있습니다. 자세한 내용은 [파이프라인 오케스트레이션, 제어 흐름 및 데이터 이동](../technology-choices/pipeline-orchestration-data-movement.md)을 참조하세요.

[2] Azure AD 계정을 사용하여 Azure Virtual Machine에서 실행되는 SQL Server에 연결하는 것은 지원되지 않습니다. 대신 도메인 Active Directory 계정을 사용합니다.

### <a name="scalability-capabilities"></a>확장성 기능

|                                                  | Azure Analysis Services | SQL Server Analysis Services | SQL Server(columnstore 인덱스 포함) | Azure SQL Database(columnstore 인덱스 포함) |
|--------------------------------------------------|-------------------------|------------------------------|-------------------------------------|---------------------------------------------|
| 고가용성을 위한 중복 지역 서버 |           예           |              아니오              |                 예                 |                     예                     |
|             쿼리 확장 지원 여부             |           예           |              아니요              |                 예                 |                     아니오                      |
|          동적 확장성(강화)          |           예           |              아니요              |                 예                 |                     아니요                      |

