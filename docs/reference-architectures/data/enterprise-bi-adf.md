---
title: SQL Data Warehouse 및 Azure Data Factory를 사용하는 자동화된 Enterprise BI
description: Azure Data Factory를 사용하여 Azure에서 ELT 워크플로 자동화
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: ffd75ba8c57a9afbc6abad61f21f738c644c9bc8
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142284"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a>SQL Data Warehouse 및 Azure Data Factory를 사용하는 자동화된 Enterprise BI

이 참조 아키텍처는 [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt)(추출-로드-변환) 파이프라인에서 증분 로드를 수행하는 방법을 보여줍니다. Azure Data Factory를 사용하여 ELT 파이프라인을 자동화합니다. 파이프라인은 증분 방식으로 최신 OLTP 데이터를 온-프레미스 SQL Server 데이터베이스에서 SQL Data Warehouse로 이동합니다. 트랜잭션 데이터는 분석을 위해 테이블 형식 모델로 변환됩니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw-adf.png)

이 아키텍처는 [SQL Data Warehouse를 사용한 Enterprise BI](./enterprise-bi-sqldw.md)에 표시된 아키텍처 위에 빌드하지만 엔터프라이즈 데이터 웨어하우징 시나리오에 대해 중요한 일부 기능을 추가합니다.

-   Data Factory를 사용하여 파이프라인의 자동화.
-   증분 로드.
-   다중 데이터 원본 통합.
-   지리 공간 데이터 및 이미지 같은 이진 데이터 로드.

## <a name="architecture"></a>아키텍처

이 아키텍처는 다음 구성 요소로 구성됩니다.

### <a name="data-sources"></a>데이터 원본

**온-프레미스 SQL Server**. 원본 데이터는 SQL Server 데이터베이스 온-프레미스에 위치합니다. 온-프레미스 환경을 시뮬레이션하기 위해 이 아키텍처에 대한 배포 스크립트는 설치된 SQL Server를 사용하여 Azure에서 가상 머신을 프로비전합니다. [Wide World Importers OLTP 예제 데이터베이스][wwi]는 원본 데이터로 사용됩니다.

**외부 데이터**. 데이터 웨어하우스에 대한 일반적인 시나리오는 여러 데이터 원본을 통합하는 것입니다. 이 참조 아키텍처는 연도별 도시 인구를 포함하는 외부 데이터 집합을 로드하여 OLTP 데이터베이스의 데이터와 통합합니다. "각 지역의 매출 증가가 인구 증가와 일치하거나 초과합니까?" 같은 인사이트에 이 데이터를 사용할 수 있습니다.

### <a name="ingestion-and-data-storage"></a>수집 및 데이터 저장소

**Blob Storage** Blob 저장소는 SQL Data Warehouse로 로딩하기 전에 원본 데이터에 대한 준비 영역으로 사용됩니다.

**Azure SQL Data Warehouse** [SQL Data Warehouse](/azure/sql-data-warehouse/)는 대규모 데이터 분석을 수행하도록 설계되고 배포된 시스템입니다. 고성능 분석을 실행하는 데 적합하도록 하는 MPP(대규모 병렬 처리)를 지원합니다. 

**Azure Data Factory**. [Data Factory][adf]는 데이터 이동 및 데이터 변환을 오케스트레이션하고 자동화하는 관리되는 서비스입니다. 이 아키텍처에서 다양한 단계의 ELT 프로세스를 조정합니다.

### <a name="analysis-and-reporting"></a>분석 및 보고

**Azure Analysis Services**. [Analysis Services](/azure/analysis-services/)는 데이터 모델링 기능을 제공하는 완전히 관리되는 서비스입니다. 의미 체계 모델은 Analysis Services에 로드됩니다.

**Power BI**. Power BI는 비즈니스 정보에 대한 데이터를 분석하는 비즈니스 분석 도구 제품군입니다. 이 아키텍처에서 Analysis Services에 저장된 의미 체계 모델을 쿼리합니다.

### <a name="authentication"></a>인증

**Azure Active Directory**(Azure AD)는 Power BI를 통해 Analysis Services 서버에 연결하는 사용자를 인증합니다.

Data Factory는 서비스 주체 또는 MSI(관리되는 서비스 ID)를 사용하여 SQL Data Warehouse를 인증하려면 Azure AD를 사용할 수 있습니다. 간소화를 위해 예제 배포는 SQL Server 인증을 사용합니다.

## <a name="data-pipeline"></a>데이터 파이프라인

[Azure Data Factory][adf]에서 파이프라인은 이 경우에 작업 &mdash;를 조정하는 데 사용된 활동의 논리적 그룹화로서 데이터를 SQL Data Warehouse로 로드하고 변환합니다. 

이 참조 아키텍처에서는 자식 파이프라인의 시퀀스를 실행하는 마스터 파이프라인을 정의합니다. 각 자식 파이프라인은 하나 이상의 데이터 웨어하우스 테이블에 데이터를 로드합니다.

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a>증분 로드

자동화된 ETL 또는 ELT 프로세스를 실행하는 경우 이전 실행 이후에 변경된 데이터만 로드하는 것이 가장 효율적입니다. 이는 모든 데이터를 로드하는 전체 로드와 달리 *증분 로드*라고 합니다. 증분 로드를 수행하려면 데이터가 변경되었음을 식별하는 방법이 필요합니다. 가장 일반적인 방법은 날짜/시간 열이든 고유 정수 열이든 원본 테이블에서 일부 열의 최신 값을 추적하는 것을 의미하는 *상위 워터 마크* 값을 사용하는 것입니다. 

SQL Server 2016부터 [임시 테이블](/sql/relational-databases/tables/temporal-tables)을 사용할 수 있습니다. 이들은 데이터 변경 내용의 전체 기록을 유지하는 시스템 버전이 지정된 테이블이 있습니다. 데이터베이스 엔진은 별도 기록 테이블의 모든 변경 기록을 자동으로 레코드합니다. FOR SYSTEM_TIME 절을 쿼리에 추가하여 기록 데이터를 쿼리할 수 있습니다. 내부적으로 데이터베이스 엔진은 기록 테이블을 쿼리하지만 응용 프로그램에 대해서는 투명합니다. 

> [!NOTE]
> 이전 버전의 SQL Server의 경우 [변경 데이터 캡처](/sql/relational-databases/track-changes/about-change-data-capture-sql-server)(CDC)를 사용할 수 있습니다. 이 방법은 별도 변경 테이블을 쿼리해야 하고 변경 내용이 타임스탬프보다는 로그 시퀀스 번호로 추적되기 때문에 임시 테이블보다 더 불편합니다. 

임시 테이블은 시간이 지남에 따라 변경될 수 있는 차원 데이터에 유용합니다. 팩트 테이블은 대개 시스템 버전 기록을 유지하는 것이 사리에 맞지 않은 경우에 판매 같은 변경이 불가능한 트랜잭션을 나타냅니다. 대신 트랜잭션에는 대개 워터 마크 값으로 사용될 수 있는 트랜잭션 날짜를 나타내는 열이 있습니다. 예를 들어 Wide World Importers OLTP 데이터베이스에서 Sales.Invoices 및 Sales.InvoiceLines 테이블에는 `sysdatetime()`을 기본값으로 하는 `LastEditedWhen` 필드가 있습니다. 

ELT 파이프라인의 일반적인 흐름은 다음과 같습니다.

1. 원본 데이터베이스의 각 테이블의 경우 마지막 ELT 작업이 실행될 때 마감 시간을 추적하여, 데이터 웨어하우스에 이 정보를 저장합니다. (초기 설치 시 항상 시간은 '1900-1-1'로 설정돼 있습니다.)

2. 데이터 내보내기 단계 중 마감 시간은 원본 데이터베이스의 저장 프로시저 집합에 매개 변수로 전달됩니다. 이러한 저장 프로시저는 마감 시간 이후 변경되거나 생성된 모든 레코드에 대해 쿼리합니다. 판매 팩트 테이블에 대해 `LastEditedWhen` 열을 사용하고, 차원 데이터에 대해 시스템 버전이 있는 임시 테이블을 사용합니다.

3. 데이터 마이그레이션이 완료되면 마감 시간을 저장하는 테이블을 업데이트합니다.

또한 각 ELT 실행에 대해 *계보*를 레코드하는 것이 유용합니다. 지정된 레코드에 대해 계보는 데이터를 생성한 ELT 실행을 사용하여 해당 레코드와 연결합니다. 각 ETL 실행의 경우 새 계보 레코드가 모든 테이블에 대해 만들어져 시작 및 종료 로드 시간을 보여줍니다. 각 레코드에 대한 계보 키는 차원 및 팩트 테이블에 저장됩니다.

![](./images/city-dimension-table.png)

새 일괄 처리 데이터가 웨어하우스에 로드된 후 Analysis Services 테이블 형식 모델을 새로 고칩니다. [REST API를 사용한 비동기 새로 고침](/azure/analysis-services/analysis-services-async-refresh)을 참조합니다.

## <a name="data-cleansing"></a>데이터 정리

데이터 정리는 ELT 프로세스의 일부여야 합니다. 이 참조 아키텍처에서 잘못된 데이터 원본 하나는 아마도 사용할 수 있는 데이터가 없기 때문에 일부 도시에 인구가 없는 도시 인구 테이블입니다. 처리 동안 ELT 파이프라인은 도시 인구 테이블에서 해당 도시를 제거합니다. 외부 테이블보다는 준비 테이블에서 데이터 정리를 수행하세요.

도시 인구 테이블에서 인구가 없는 도시를 제거하는 저장 프로시저는 다음과 같습니다. ([여기](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql)에서 소스 파일을 찾을 수 있습니다.) 

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a>외부 데이터 원본

데이터 웨어하우스는 종종 여러 소스의 데이터를 통합합니다. 이 참조 아키텍처는 인구 통계 데이터를 포함하는 외부 데이터 원본을 로드합니다. 이 데이터 집합은 [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) 샘플의 일부로 Azure Blob Storage에서 사용할 수 있습니다.

Azure Data Factory는 [Blob Storage 커넥터](/azure/data-factory/connector-azure-blob-storage)를 사용하여 Blob Storage에서 직접 복사할 수 있습니다. 그러나 커넥터는 연결 문자열 또는 공유 액세스 서명이 필요하므로 공용 읽기 액세스를 사용하여 BLOB을 복사하는 데 사용할 수 없습니다. 해결 방법으로 PolyBase를 사용하여 Blob 저장소에서 외부 테이블을 만든 다음, 외부 테이블을 SQL Data Warehouse에 복사할 수 있습니다. 

## <a name="handling-large-binary-data"></a>큰 이진 데이터 처리 

원본 데이터베이스에서 도시 테이블에는 [geography](/sql/t-sql/spatial-geography/spatial-types-geography) 공간 데이터 형식을 보유하는 위치 열이 있습니다. SQL Data Warehouse는 기본적으로 **geography** 형식을 지원하지 않으므로 이 필드는 로딩 동안 **varbinary** 형식으로 변환됩니다. ([지원되지 않는 데이터 형식에 대한 해결 방법](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types)을 참조합니다.)

하지만 PolyBase는 `varbinary(8000)`의 최대 열 크기를 지원하여 일부 데이터가 잘릴 수도 있습니다. 이 문제에 대한 해결 방법은 다음과 같이 내보내기 중에 데이터를 청크로 분할한 다음, 청크를 다시 어셈블하는 것입니다.

1. 위치 열에 대해 임시 준비 테이블을 만듭니다.

2. 각 도시의 경우 위치 데이터를 8000바이트 청크로 분할하여 그 결과 각 도시에 대해 1 &ndash; N 행이 됩니다.

3. 청크를 다시 어셈블하려면 T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) 연산자를 사용하여 행을 열로 변환한 다음, 각 도시에 대해 열 값을 연결합니다.

문제는 각 도시가 지리 데이터의 크기에 따라 다른 수의 행으로 분할된다는 것입니다. PIVOT 연산자가 작동하려면 모든 도시에 동일한 수의 행이 있어야 합니다. 이 작업을 수행하려면 피벗 후에 모든 도시가 동일한 수의 열을 가질 수 있도록 T-SQL 쿼리([here][MergeLocation]을 볼 수 있는)가 빈 값이 있는 행을 채우기 위한 몇 가지 트릭을 수행합니다. 결과 쿼리는 한 번에 하나씩 행을 반복하는 것보다 훨씬 더 빠른 것으로 밝혀졌습니다.

동일한 방식이 이미지 데이터에 사용됩니다.

## <a name="slowly-changing-dimensions"></a>느린 변경 차원

차원 데이터는 상대적으로 정적이지만 변경될 수 있습니다. 예를 들어 제품이 다른 제품 범주에 다시 할당될 수 있습니다. 느린 변경 차원을 처리하는 방법은 여러 가지가 있습니다. [유형 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row)라는 일반 기술은 차원이 변경될 때마다 새 레코드를 추가하는 것입니다. 

유형 2 방법을 구현하려면 차원 테이블은 지정된 레코드의 유효 날짜 범위를 지정하는 추가 열이 필요합니다. 또한 원본 데이터베이스의 기본 키가 중복되게 되므로 차원 테이블에 인공적인 기본 키가 있어야 합니다.

다음 이미지에서는 Dimension.City 테이블을 보여줍니다. `WWI City ID` 열은 원본 데이터베이스의 기본 키입니다. `City Key` 열은 ETL 파이프라인 도중 생성된 인공 키입니다. 또한 테이블에는 각 행이 유효한 경우 범위를 정의하는 `Valid From` 및 `Valid To` 열이 있습니다. 현재 값에는 '9999-12-31'과 동일한 `Valid To`가 있습니다.

![](./images/city-dimension-table.png)

이 방식의 장점은 분석에 중요할 수 있는 기록 데이터를 보존한다는 것입니다. 그러나 동일한 엔터티에 대해 여러 행이 있다는 의미이기도 합니다. 예를 들어 `WWI City ID` = 28561과 일치하는 레코드는 다음과 같습니다.

![](./images/city-dimension-table-2.png)

각 판매 팩트의 경우 송장 날짜에 해당하는 도시 차원 테이블의 단일 행과 해당 팩트를 연결하려 합니다. ETL 프로세스의 일부로 다음과 같은 추가 열을 만듭니다. 

다음 T-SQL 쿼리는 도시 차원 테이블에서 올바른 도시 키와 각 송장을 연결하는 임시 테이블을 만듭니다.

```sql
CREATE TABLE CityHolder
WITH (HEAP , DISTRIBUTION = HASH([WWI Invoice ID]))
AS
SELECT DISTINCT s1.[WWI Invoice ID] AS [WWI Invoice ID],
                c.[City Key] AS [City Key]
    FROM [Integration].[Sale_Staging] s1
    CROSS APPLY (
                SELECT TOP 1 [City Key]
                    FROM [Dimension].[City]
                WHERE [WWI City ID] = s1.[WWI City ID]
                    AND s1.[Last Modified When] > [Valid From]
                    AND s1.[Last Modified When] <= [Valid To]
                ORDER BY [Valid From], [City Key] DESC
                ) c

```

이 테이블은 판매 팩트 테이블의 열을 채우는 데 사용됩니다.

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

이 열을 사용하면 Power BI 쿼리가 지정된 판매 송장에 대해 올바른 도시 레코드를 찾을 수 있습니다.

## <a name="security-considerations"></a>보안 고려 사항

추가 보안을 위해 [Virtual Network 서비스 엔드포인트](/azure/virtual-network/virtual-network-service-endpoints-overview)를 사용하여 가상 네트워크에 대해서만 Azure 서비스 리소스를 보호할 수 있습니다. 이렇게 함으로써 해당 리소스에 대한 공용 인터넷 액세스를 완전히 제거하여 가상 네트워크의 트래픽만 허용하게 됩니다.

이 방법을 사용하여 Azure에서 VNet을 만든 다음, Azure 서비스에 대한 개인 서비스 엔드포인트를 만듭니다. 그러면 이러한 서비스는 해당 가상 네트워크의 트래픽으로 제한됩니다. 게이트웨이를 통해 온-프레미스 네트워크에서 이 서비스에 연결할 수도 있습니다.

다음과 같은 제한 사항을 고려해야 합니다.

- 이 참조 아키텍처가 만들어졌을 때 VNet 서비스 엔드포인트는 Azure Analysis Service를 제외한 Azure Storage 및 Azure SQL Data Warehouse에 대해 지원됩니다. [여기](https://azure.microsoft.com/updates/?product=virtual-network)에서 최신 상태를 확인합니다. 

- Azure Storage에 대해 서비스 엔드포인트가 사용하도록 설정된 경우 PolyBase는 Storage의 데이터를 SQL Data Warehouse에 복사할 수 없습니다. 이 문제에 대한 완화 방법이 있습니다. 자세한 내용은 [Azure 저장소에서 VNet 서비스 엔드포인트 사용의 영향](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage)을 참조합니다. 

- 온-프레미스에서 Azure Storage로 데이터를 이동하려면 온-프레미스 또는 ExpressRoute에서 공용 IP 주소를 허용 목록에 추가해야 합니다. 자세한 내용은 [Virtual Network에 대한 Azure 서비스 보호](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks)를 참조합니다.

- SQL Data Warehouse에서 데이터를 읽기 위해 Analysis Services를 사용하려면 SQL Data Warehouse 서비스 엔드포인트를 포함하는 가상 네트워크에 Windows VM을 배포합니다. 이 VM에 [Azure 온-프레미스 데이터 게이트웨이](/azure/analysis-services/analysis-services-gateway)를 설치합니다. 그런 다음, 데이터 게이트웨이에 Azure Analysis 서비스를 연결합니다.

## <a name="deploy-the-solution"></a>솔루션 배포

이 참조 아키텍처에 대한 배포는 [GitHub][ref-arch-repo-folder]에서 사용할 수 있습니다. 다음을 배포합니다.

  * 온-프레미스 데이터베이스 서버를 시뮬레이션하는 Windows VM Power BI Desktop과 함께 SQL Server 2017 및 관련된 도구를 포함합니다.
  * SQL Server 데이터베이스에서 가져온 데이터를 저장할 Blob 저장소를 제공하는 Azure 저장소 계정
  * Azure SQL Data Warehouse 인스턴스
  * Azure Analysis Services 인스턴스
  * ELT 작업에 대한 Azure Data Factory 및 Data Factory 파이프라인

### <a name="prerequisites"></a>필수 조건

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="variables"></a>variables

이후 단계는 일부 사용자 정의 변수를 포함합니다. 이를 자신이 정의하는 값으로 바꿔야 합니다.

- `<data_factory_name>` 데이터 팩터리 이름.
- `<analysis_server_name>` Analysis Services 서버 이름.
- `<active_directory_upn>` Azure Active Directory UPN(사용자 계정 이름). 예: `user@contoso.com`
- `<data_warehouse_server_name>` SQL Data Warehouse 서버 이름.
- `<data_warehouse_password>` SQL Data Warehouse 관리자 암호.
- `<resource_group_name>` 리소스 그룹의 이름.
- `<region>` 리소스가 배포될 Azure 지역.
- `<storage_account_name>` Storage 계정 이름 Storage 계정에 대한 [명명 규칙](../../best-practices/naming-conventions.md#naming-rules-and-restrictions)을 따라야 합니다.
- `<sql-db-password>` SQL Server 로그인 암호.

### <a name="deploy-azure-data-factory"></a>Azure Data Factory 배포

1. [GitHub 리포지터리][ref-arch-repo]의 `data\enterprise_bi_sqldw_advanced\azure\templates` 폴더로 이동합니다.

2. 다음 Azure CLI 명령을 실행하여 리소스 그룹을 만듭니다.  

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

    SQL Data Warehouse, Azure Analysis Services 및 Data Factory v2를 지원하는 지역을 지정합니다. [지역별 Azure 제품](https://azure.microsoft.com/global-infrastructure/services/) 참조

3. 다음 명령을 실행합니다.

    ```
    az group deployment create --resource-group <resource_group_name> \
        --template-file adf-create-deploy.json \
        --parameters factoryName=<data_factory_name> location=<location>
    ```

다음으로, Azure Portal을 사용하여 다음과 같이 Azure Data Factory [통합 런타임](/azure/data-factory/concepts-integration-runtime)에 대한 인증 키를 가져옵니다.

1. [Azure Portal](https://portal.azure.com/)에서 Data Factory 인스턴스로 이동합니다.

2. Data Factory 블레이드에서 **작성자 및 모니터링**을 클릭합니다. 이렇게 하면 다른 브라우저 창에서 Azure Data Factory 포털이 열립니다.

    ![](./images/adf-blade.png)

3. Azure Data Factory 포털에서 연필 아이콘("작성자")을 선택합니다. 

4. **연결**을 클릭한 다음, **Integration Runtime**을 선택합니다.

5. **sourceIntegrationRuntime**에서 연필 아이콘("편집")을 클릭합니다.

    > [!NOTE]
    > 포털의 상태가 "사용할 수 없음"으로 표시됩니다. 이는 온-프레미스 서버를 배포할 때까지 필요합니다.

6. **Key1**을 찾아 인증 키의 값을 복사합니다.

다음 단계에 대한 인증 키가 필요합니다.

### <a name="deploy-the-simulated-on-premises-server"></a>시뮬레이션된 온-프레미스 서버 배포

이 단계에서는 SQL Server 2017 및 관련 도구를 포함하는 시뮬레이션된 온-프레미스 서버로 VM을 배포합니다. 또한 [Wide World Importers OLTP 데이터베이스][wwi]를 SQL Server로 로드합니다.

1. 리포지토리의 `data\enterprise_bi_sqldw_advanced\onprem\templates` 폴더로 이동합니다.

2. `onprem.parameters.json` 파일에서 `adminPassword`을 검색합니다. SQL Server VM에 로그인하기 위한 암호입니다. 다른 암호로 값을 바꿉니다.

3. 동일한 파일에서 `SqlUserCredentials`을 검색합니다. 이 속성은 SQL Server 계정 자격 증명을 지정합니다. 다른 값으로 암호를 바꿉니다.

4. 아래와 같이 동일한 파일에서 Integration Runtime 인증 키를 `IntegrationRuntimeGatewayKey` 매개 변수에 붙여넣습니다.

    ```json
    "protectedSettings": {
        "configurationArguments": {
            "SqlUserCredentials": {
                "userName": ".\\adminUser",
                "password": "<sql-db-password>"
            },
            "IntegrationRuntimeGatewayKey": "<authentication key>"
        }
    ```

5. 다음 명령을 실행합니다.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

이 단계는 완료하는 데 20~30분 정도 걸릴 수 있습니다. 도구를 설치하고 데이터베이스를 복원하려면 [DSC](/powershell/dsc/overview) 스크립트 실행이 포함됩니다. 

### <a name="deploy-azure-resources"></a>Azure 리소스 배포

이 단계에서는 SQL Data Warehouse, Azure Analysis Services 및 Data Factory를 프로비전합니다.

1. [GitHub 리포지터리][ref-arch-repo]의 `data\enterprise_bi_sqldw_advanced\azure\templates` 폴더로 이동합니다.

2. 다음 Azure CLI 명령을 실행합니다. 꺾쇠 괄호 안에 표시된 매개 변수 값을 바꿉니다.

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<data_warehouse_server_name>" \
     "dwAdminLogin"="adminuser" "dwAdminPassword"="<data_warehouse_password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="<user@contoso.com>"
    ```

    - `storageAccountName` 매개 변수는 Storage 계정에 대한 [명명 규칙](../../best-practices/naming-conventions.md#naming-rules-and-restrictions)을 따라야 합니다. 
    - `analysisServerAdmin` 매개 변수의 경우 Azure Active Directory UPN(사용자 계정 이름)을 사용합니다.

3. 저장소 계정에서 액세스 키를 가져오려면 다음 Azure CLI 명령을 실행합니다. 이 키는 다음 단계에서 사용합니다.

    ```bash
    az storage account keys list -n <storage_account_name> -g <resource_group_name> --query [0].value
    ```

4. 다음 Azure CLI 명령을 실행합니다. 꺾쇠 괄호 안에 표시된 매개 변수 값을 바꿉니다. 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
    --template-file adf-pipeline-deploy.json \
    --parameters "factoryName"="<data_factory_name>" \
    "sinkDWConnectionString"="Server=tcp:<data_warehouse_server_name>.database.windows.net,1433;Initial Catalog=wwi;Persist Security Info=False;User ID=adminuser;Password=<data_warehouse_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" \
    "blobConnectionString"="DefaultEndpointsProtocol=https;AccountName=<storage_account_name>;AccountKey=<storage_account_key>;EndpointSuffix=core.windows.net" \
    "sourceDBConnectionString"="Server=sql1;Database=WideWorldImporters;User Id=adminuser;Password=<sql-db-password>;Trusted_Connection=True;"
    ```

    연결 문자열에는 바꿔야 될 하위 문자열이 꺾쇠 괄호로 표시돼 있습니다. `<storage_account_key>`의 경우 이전 단계에서 얻은 키를 사용합니다. `<sql-db-password>`의 경우는 이전에 `onprem.parameters.json` 파일에서 지정한 SQL Server 계정 암호를 사용합니다.

### <a name="run-the-data-warehouse-scripts"></a>데이터 웨어하우스 스크립트 실행

1. [Azure Portal](https://portal.azure.com/)에서 `sql-vm1`이라는 온-프레미스 VM을 찾습니다. VM에 대한 사용자 이름 및 암호는 `onprem.parameters.json` 파일에서 지정됩니다.

2. **연결**을 클릭하고 원격 데스크톱을 사용하여 VM에 연결합니다.

3. 원격 데스크톱 세션에서 명령 프롬프트를 열고 VM에서 다음 폴더로 이동합니다.

    ```
    cd C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw_advanced\azure\sqldw_scripts
    ```

4. 다음 명령 실행:

    ```
    deploy_database.cmd -S <data_warehouse_server_name>.database.windows.net -d wwi -U adminuser -P <data_warehouse_password> -N -I
    ```

    `<data_warehouse_server_name>` 및 `<data_warehouse_password>`의 경우 이전의 데이터 웨어하우스 서버 이름 및 암호를 사용합니다.

이 단계를 확인하려면 SSMS(SQL Server Management Studio)를 사용하여 SQL Data Warehouse 데이터베이스에 연결할 수 있습니다. 데이터베이스 테이블 스키마를 참조해야 합니다.

### <a name="run-the-data-factory-pipeline"></a>Data Factory 파이프라인 실행

1. 동일한 원격 데스크톱 세션에서 PowerShell 창을 엽니다.

2. 다음 PowerShell 명령을 실행합니다. 메시지가 표시되면 **예**를 선택합니다.

    ```powershell
    Install-Module -Name AzureRM -AllowClobber
    ```

3. 다음 PowerShell 명령을 실행합니다. 메시지가 표시되면 Azure 자격 증명을 입력합니다.

    ```powershell
    Connect-AzureRmAccount 
    ```

4. 다음 PowerShell 명령을 실행합니다. 꺽쇠 괄호 안의 값을 바꿉니다.

    ```powershell
    Set-AzureRmContext -SubscriptionId <subscription id>

    Invoke-AzureRmDataFactoryV2Pipeline -DataFactory <data-factory-name> -PipelineName "MasterPipeline" -ResourceGroupName <resource_group_name>

5. In the Azure Portal, navigate to the Data Factory instance that was created earlier.

6. In the Data Factory blade, click **Author & Monitor**. This opens the Azure Data Factory portal in another browser window.

    ![](./images/adf-blade.png)

7. In the Azure Data Factory portal, click the **Monitor** icon. 

8. Verify that the pipeline completes successfully. It can take a few minutes.

    ![](./images/adf-pipeline-progress.png)


## Build the Analysis Services model

In this step, you will create a tabular model that imports data from the data warehouse. Then you will deploy the model to Azure Analysis Services.

**Create a new tabular project**

1. From your Remote Desktop session, launch SQL Server Data Tools 2015.

2. Select **File** > **New** > **Project**.

3. In the **New Project** dialog, under **Templates**, select  **Business Intelligence** > **Analysis Services** > **Analysis Services Tabular Project**. 

4. Name the project and click **OK**.

5. In the **Tabular model designer** dialog, select **Integrated workspace**  and set **Compatibility level** to `SQL Server 2017 / Azure Analysis Services (1400)`. 

6. Click **OK**.


**Import data**

1. In the **Tabular Model Explorer** window, right-click the project and select **Import from Data Source**.

2. Select **Azure SQL Data Warehouse** and click **Connect**.

3. For **Server**, enter the fully qualified name of your Azure SQL Data Warehouse server. You can get this value from the Azure Portal. For **Database**, enter `wwi`. Click **OK**.

4. In the next dialog, choose **Database** authentication and enter your Azure SQL Data Warehouse user name and password, and click **OK**.

5. In the **Navigator** dialog, select the checkboxes for the **Fact.\*** and **Dimension.\*** tables.

    ![](./images/analysis-services-import-2.png)

6. Click **Load**. When processing is complete, click **Close**. You should now see a tabular view of the data.

**Create measures**

1. In the model designer, select the **Fact Sale** table.

2. Click a cell in the the measure grid. By default, the measure grid is displayed below the table. 

    ![](./images/tabular-model-measures.png)

3. In the formula bar, enter the following and press ENTER:

    ```
    Total Sales:=SUM('Fact Sale'[Total Including Tax])
    ```

4. Repeat these steps to create the following measures:

    ```
    Number of Years:=(MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber]))+1
    
    Beginning Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MIN('Fact CityPopulation'[YearNumber])))
    
    Ending Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MAX('Fact CityPopulation'[YearNumber])))
    
    CAGR:=IFERROR((([Ending Population]/[Beginning Population])^(1/[Number of Years]))-1,0)
    ```

    ![](./images/analysis-services-measures.png)

For more information about creating measures in SQL Server Data Tools, see [Measures](/sql/analysis-services/tabular-models/measures-ssas-tabular).

**Create relationships**

1. In the **Tabular Model Explorer** window, right-click the project and select **Model View** > **Diagram View**.

2. Drag the **[Fact Sale].[City Key]** field to the **[Dimension City].[City Key]** field to create a relationship.  

3. Drag the **[Face CityPopulation].[City Key]** field to the **[Dimension City].[City Key]** field.  

    ![](./images/analysis-services-relations-2.png)

**Deploy the model**

1. From the **File** menu, choose **Save All**.

2. In **Solution Explorer**, right-click the project and select **Properties**. 

3. Under **Server**, enter the URL of your Azure Analysis Services instance. You can get this value from the Azure Portal. In the portal, select the Analysis Services resource, click the Overview pane, and look for the **Server Name** property. It will be similar to `asazure://westus.asazure.windows.net/contoso`. Click **OK**.

    ![](./images/analysis-services-properties.png)

4. In **Solution Explorer**, right-click the project and select **Deploy**. Sign into Azure if prompted. When processing is complete, click **Close**.

5. In the Azure portal, view the details for your Azure Analysis Services instance. Verify that your model appears in the list of models.

    ![](./images/analysis-services-models.png)

## Analyze the data in Power BI Desktop

In this step, you will use Power BI to create a report from the data in Analysis Services.

1. From your Remote Desktop session, launch Power BI Desktop.

2. In the Welcome Scren, click **Get Data**.

3. Select **Azure** > **Azure Analysis Services database**. Click **Connect**

    ![](./images/power-bi-get-data.png)

4. Enter the URL of your Analysis Services instance, then click **OK**. Sign into Azure if prompted.

5. In the **Navigator** dialog, expand the tabular project, select the model, and click **OK**.

2. In the **Visualizations** pane, select the **Table** icon. In the Report view, resize the visualization to make it larger.

6. In the **Fields** pane, expand **Dimension City**.

7. From **Dimension City**, drag **City** and **State Province** to the **Values** well.

9. In the **Fields** pane, expand **Fact Sale**.

10. From **Fact Sale**, drag **CAGR**, **Ending Population**,  and **Total Sales** to the **Value** well.

11. Under **Visual Level Filters**, select **Ending Population**. Set the filter to "is greater than 100000" and click **Apply filter**.

12. Under **Visual Level Filters**, select **Total Sales**. Set the filter to "is 0" and click **Apply filter**.

![](./images/power-bi-report-2.png)

The table now shows cities with population greater than 100,000 and zero sales. CAGR  stands for Compounded Annual Growth Rate and measures the rate of population growth per city. You could use this value to find cities with high growth rates, for example. However, note that the values for CAGR in the model aren't accurate, because they are derived from sample data.

To learn more about Power BI Desktop, see [Getting started with Power BI Desktop](/power-bi/desktop-getting-started).


[adf]: //azure/data-factory
[azure-cli-2]: //azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw_advanced
[wwi]: //sql/sample/world-wide-importers/wide-world-importers-oltp-database
