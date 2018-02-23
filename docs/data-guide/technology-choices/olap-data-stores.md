---
title: "OLAP 데이터 저장소 선택"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f3041b95696c9408a2c9ab747fe1ec3041db0743
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-olap-data-store-in-azure"></a>Azure에서 OLAP 데이터 저장소 선택

OLAP(온라인 분석 처리)는 대규모 비즈니스 데이터베이스를 구성하고 복잡한 분석을 지원하는 기술입니다. 이 항목에서는 Azure의 OLAP 솔루션에 대한 옵션을 비교합니다.

> [!NOTE]
> OLAP 데이터 저장소를 사용해야 하는 경우에 대한 자세한 내용은 [온라인 분석 처리](../scenarios/online-analytical-processing.md)를 참조하세요.

## <a name="what-are-your-options-when-choosing-an-olap-data-store"></a>OLAP 데이터 저장소를 선택할 때의 옵션은 무엇인가요?

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

다음 표에서 주요 기능 차이점을 요약해서 보여 줍니다.

### <a name="general-capabilities"></a>일반 기능

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server(columnstore 인덱스 포함) | Azure SQL Database(columnstore 인덱스 포함) |
| --- | --- | --- | --- | --- |
| 관리되는 서비스인지 여부 | 예 | 아니요 | 아니요 | 예 |
| 다차원 큐브 지원 여부 | 아니요 | 예 | 아니오 | 아니오 |
| 테이블 형식 의미 체계 모델 지원 여부 | 예 | 예 | 아니오 | 아니요 |
| 여러 데이터 원본을 쉽게 통합 | 예 | 예 | 아니요 <sup>1</sup> | 아니요 <sup>1</sup> |
| 실시간 분석 지원 | 아니요 | 아니요 | 예 | 예 |
| 원본에서 데이터를 복사하는 프로세스 필요 | 예 | 예 | 아니요 | 아니요 |
| Azure AD 통합 | 예 | 아니오 | 아니요 <sup>2</sup> | 예 |

[1] SQL Server 및 Azure SQL Database는 여러 외부 데이터 원본에서 쿼리하거나 이러한 원본을 통합하는 데 사용할 수 없지만, [SSIS](/sql/integration-services/sql-server-integration-services) 또는 [Azure Data Factory](/azure/data-factory/)를 사용하여 이 작업을 자동으로 수행하는 파이프라인을 여전히 구축할 수 있습니다. Azure VM에서 호스트되는 SQL Server의 경우 연결된 서버 및 [PolyBase](/sql/relational-databases/polybase/polybase-guide)와 같은 추가 옵션을 사용할 수 있습니다. 자세한 내용은 [파이프라인 오케스트레이션, 제어 흐름 및 데이터 이동](../technology-choices/pipeline-orchestration-data-movement.md)을 참조하세요.

[2] Azure AD 계정을 사용하여 Azure Virtual Machine에서 실행되는 SQL Server에 연결하는 것은 지원되지 않습니다. 대신 도메인 Active Directory 계정을 사용합니다.

### <a name="scalability-capabilities"></a>확장성 기능

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server(columnstore 인덱스 포함) | Azure SQL Database(columnstore 인덱스 포함) |
| --- | --- | --- | --- | --- |
| 고가용성을 위한 중복 지역 서버  | 예 | 아니오 | 예 | 예 |
| 쿼리 확장 지원  | 예 | 아니요 | 예 | 아니오 |
| 동적 확장성(강화)  | 예 | 아니요 | 예 | 아니요 |

