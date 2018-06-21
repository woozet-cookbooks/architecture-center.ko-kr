---
title: SQL Data Warehouse를 사용하는 Enterprise BI
description: Azure를 사용하여 온-프레미스에 저장된 관계형 데이터에서 비즈니스 정보 얻기
author: alexbuckgit
ms.date: 04/13/2018
ms.openlocfilehash: b5e5aa32fc9cc8c7b8b5a42c9a4fc3e0216b2f72
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/16/2018
ms.locfileid: "31012839"
---
# <a name="enterprise-bi-with-sql-data-warehouse"></a>SQL Data Warehouse를 사용하는 Enterprise BI
 
이 참조 아키텍처는 온-프레미스 SQL Server 데이터베이스에서 SQL Data Warehouse로 데이터를 이동하고 분석을 위해 데이터를 변경하는 [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt)(추출 부하 변형) 파이프라인을 구현합니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

![](./images/enterprise-bi-sqldw.png)

**시나리오**: 조직에는 SQL Server 데이터베이스 온-프레미스에 저장된 대규모 OLTP 데이터 집합이 있습니다. 조직은 Power BI를 사용하여 분석을 수행하기 위해 SQL Data Warehouse를 사용하려고 합니다. 

이 참조 아키텍처는 일회성 또는 주문형 작업을 위해 설계되었습니다. 지속적으로(매시간 또는 매일) 데이터를 이동해야 하는 경우 Azure Data Factory를 사용하여 자동화된 워크플로를 정의하는 것이 좋습니다.

## <a name="architecture"></a>아키텍처

이 아키텍처는 다음 구성 요소로 구성됩니다.

**SQL Server**. 원본 데이터는 SQL Server 데이터베이스 온-프레미스에 위치합니다. 온-프레미스 환경을 시뮬레이션하기 위해 이 아키텍처에 대한 배포 스크립트는 설치된 SQL Server를 사용하여 Azure에서 가상 머신을 프로비전합니다. 

**Blob Storage** Blob 저장소는 SQL Data Warehouse로 로딩하기 전에 데이터를 복사하는 준비 영역으로 사용됩니다.

**Azure SQL Data Warehouse** [SQL Data Warehouse](/azure/sql-data-warehouse/)는 대규모 데이터 분석을 수행하도록 설계되고 배포된 시스템입니다. 고성능 분석을 실행하는 데 적합하도록 하는 MPP(대규모 병렬 처리)를 지원합니다. 

**Azure Analysis Services**. [Analysis Services](/azure/analysis-services/)는 데이터 모델링 기능을 제공하는 완전히 관리되는 서비스입니다. Analysis Services를 사용하여 사용자가 쿼리할 수 있는 의미 체계 모델을 만듭니다. Analysis Services는 BI 대시보드 시나리오에서 특히 유용합니다. 이 아키텍처에서 Analysis Services는 의미 체계 모델을 처리하도록 데이터 웨어하우스에서 데이터를 읽고, 대시보드 쿼리를 효율적으로 처리합니다. 또한 신속한 쿼리 처리에 대한 복제본을 확장하여 탄력적 동시성을 지원합니다.

현재 Azure Analysis Services는 테이블 형식 모델을 지원하지만 다차원 모델을 지원하지는 않습니다. 테이블 형식 모델은 관계형 모델링 구문(테이블 및 열)을 사용하는 반면 다차원 모델은 OLAP 모델링 구문(큐브, 차원 및 측정값)을 사용합니다. 다차원 모델이 필요한 경우 SSAS(SQL Server Analysis Services)를 사용합니다. 자세한 내용은 [테이블 형식 및 다차원 솔루션 비교](/sql/analysis-services/comparing-tabular-and-multidimensional-solutions-ssas)를 참조하세요.

**Power BI**. Power BI는 비즈니스 정보에 대한 데이터를 분석하는 비즈니스 분석 도구 제품군입니다. 이 아키텍처에서 Analysis Services에 저장된 의미 체계 모델을 쿼리합니다.

**Azure Active Directory**(Azure AD)는 Power BI를 통해 Analysis Services 서버에 연결하는 사용자를 인증합니다.

## <a name="data-pipeline"></a>데이터 파이프라인
 
이 참조 아키텍처는 데이터 원본으로 [WorldWideImporters](/sql/sample/world-wide-importers/wide-world-importers-oltp-database) 샘플 데이터베이스를 사용합니다. 데이터 파이프라인에는 다음 단계가 있습니다.

1. SQL Server에서 플랫 파일로 데이터를 내보냅니다(bcp 유틸리티).
2. Azure Blob Storage로 플랫 파일을 복사합니다(AzCopy).
3. SQL Data Warehouse로 데이터를 로드합니다(PolyBase).
4. 데이터를 별모양 스키마로 변환합니다(T-SQL).
5. 의미 체계 모델을 Analysis Services로 로드합니다(SQL Server Data Tools).

![](./images/enterprise-bi-sqldw-pipeline.png)
 
> [!NOTE]
> 1&ndash;3단계의 경우 Redgate Data Platform Studio를 사용하는 것이 좋습니다. Data Platform Studio는 가장 적합한 호환성 수정 및 최적화를 적용하므로 SQL Data Warehouse를 시작하는 가장 빠른 방법입니다. 자세한 내용은 [Redgate Data Platform Studio를 사용하여 데이터 로드](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate)를 참조하세요. 

다음 섹션에서는 이러한 단계를 자세히 설명합니다.

### <a name="export-data-from-sql-server"></a>SQL Server에서 데이터 내보내기

[bcp](/sql/tools/bcp-utility)(대량 복사 프로그램) 유틸리티는 SQL 테이블에서 플랫 텍스트 파일을 만드는 신속한 방법입니다. 이 단계에서는 내보내려는 열을 선택하지만 데이터를 변환하지 않습니다. 모든 데이터 변환은 SQL Data Warehouse에서 수행되어야 합니다.

**권장 사항**

가능하면 프로덕션 환경에서 리소스 경합을 최소화하기 위해 사용량이 적은 시간에 데이터 추출을 예약합니다. 

데이터베이스 서버에서 bcp를 실행하지 않습니다. 대신 다른 컴퓨터에서 실행합니다. 로컬 드라이브에 파일을 씁니다. 동시 쓰기를 처리하기에 충분한 I/O 리소스가 있는지 확인합니다. 최상의 성능을 위해서 전용 고속 저장소 드라이브에 파일을 내보냅니다.

Gzip 압축된 형식으로 내보낸 데이터를 저장하여 네트워크 전송 속도를 높일 수 있습니다. 그러나 압축된 파일을 웨어하우스로 로드하는 것은 압축되지 않은 파일을 로드하는 것보다 느리므로 더 빠른 네트워크 전송과 더 빠른 로딩 간에 균형 유지가 있습니다. Gzip 압축을 사용하려는 경우 단일 Gzip 파일을 만들지 마십시오. 대신 여러 개의 압축된 파일로 데이터를 분할합니다.

### <a name="copy-flat-files-into-blob-storage"></a>플랫 파일을 Blob 저장소에 복사

[AzCopy](/azure/storage/common/storage-use-azcopy) 유틸리티는 Azure Blob 저장소로 데이터의 고성능 복사를 위해 설계되었습니다.

**권장 사항**

원본 데이터의 위치 근처 지역에서 저장소 계정을 만듭니다. 동일한 지역에 저장소 계정 및 SQL Data Warehouse 인스턴스를 배포합니다. 

CPU 및 I/O 사용은 프로덕션 작업에 영향을 줄 수 있으므로 프로덕션 작업을 실행하는 동일한 컴퓨터에서 AzCopy를 실행하지 마십시오. 

업로드 속도를 확인하려면 먼저 업로드를 테스트합니다. AzCopy에서 /NC 옵션을 사용하여 동시 복사 작업의 수를 지정할 수 있습니다. 기본 값으로 시작한 다음, 이 설정으로 실험하여 성능을 조정합니다. 저대역폭 환경에서는 많은 수의 동시 작업으로 네트워크 연결에 과부하가 걸려 작업이 성공적으로 실행되지 못할 수 있습니다.  

AzCopy는 공용 인터넷을 통해 저장소로 데이터를 이동합니다. 충분히 빠르지 않은 경우 [ExpressRoute](/azure/expressroute/) 회로 설정을 고려합니다. ExpressRoute는 Azure에 대한 전용 비공개 연결을 통해 데이터를 라우팅하는 서비스입니다. 네트워크 연결 속도가 너무 느린 경우 다른 옵션은 디스크의 데이터를 Azure 데이터 센터로 물리적으로 배달하는 것입니다. 자세한 내용은 [Azure 간 데이터 전송](/azure/architecture/data-guide/scenarios/data-transfer)을 참조하세요.

복사 작업 중 AzCopy는 임시 저널 파일을 만듭니다. 이를 통해 중단되는 경우(예: 네트워크 오류로 인해) AzCopy에서 작업을 다시 시작할 수 있습니다. 저널 파일을 저장할 디스크 공간이 충분한지 확인합니다. /Z 옵션을 사용하여 저널 파일이 기록되는 위치를 지정할 수 있습니다.

### <a name="load-data-into-sql-data-warehouse"></a>SQL Data Warehouse로 데이터 로드

[PolyBase](/sql/relational-databases/polybase/polybase-guide)를 사용하여 Blob 저장소에서 데이터 웨어하우스로 파일을 로드합니다. PolyBase는 SQL Data Warehouse의 MPP(대규모 병렬 처리) 아키텍처를 활용하도록 디자인되었으며 가장 빠르게 SQL Data Warehouse로 데이터를 로드할 수 있게 합니다. 

데이터 로드는 두 단계 프로세스로 이루어집니다.

1. 데이터에 대한 외부 테이블 집합을 만듭니다. 외부 테이블은 웨어하우스의 외부에 저장된 데이터를 가리키는 테이블 정의이며 &mdash; 이 경우 Blob 저장소의 플랫 파일입니다. 이 단계는 데이터를 웨어하우스로 이동하지 않습니다.
2. 준비 테이블을 만들고, 준비 테이블로 데이터를 로드합니다. 이 단계는 데이터를 웨어하우스로 복사합니다.

**권장 사항**

많은 양의 데이터(1TB 이상)가 있고 병렬 처리를 활용하는 분석 워크 로드를 실행하는 경우 SQL Data Warehouse를 고려합니다. SQL Data Warehouse는 OLTP 워크로드 또는 소량의 데이터 집합(< 250GB)에 잘 맞지 않습니다. 250GB보다 작은 데이터 집합의 경우 Azure SQL Database 또는 SQL Server를 고려합니다. 자세한 내용은 [데이터 웨어하우징](../../data-guide/relational-data/data-warehousing.md)을 참조하세요.

힙 테이블로 인덱싱되지 않은 준비 테이블을 만듭니다. 프로덕션 테이블을 만드는 쿼리는 전체 테이블 검색이 발생하므로 준비 테이블을 인덱싱할 이유가 없습니다.

PolyBase는 웨어하우스에서 자동으로 병렬 처리를 활용합니다. 로드 성능은 DWU를 늘리면 확장합니다. 최상의 성능을 위해 단일 로드 작업을 사용합니다. 입력 데이터를 청크로 분리하고 여러 동시 로드를 실행하는 성능 이점이 없습니다.

PolyBase는 Gzip 압축된 파일을 읽을 수 있습니다. 그러나 파일 압축 풀기는 단일 스레드 작업이므로 압축된 파일당 단일 판독기만 사용됩니다. 따라서 대량의 단일 압축된 파일을 로드하지 않도록 합니다. 대신 병렬 처리를 활용하기 위해 여러 개의 압축된 파일로 데이터를 분할합니다. 

다음과 같은 제한 사항을 고려해야 합니다.

- PolyBase는 `varchar(8000)`, `nvarchar(4000)` 또는 `varbinary(8000)`의 최대 열 크기를 지원합니다. 이러한 한도를 초과하는 데이터가 있는 경우 한 가지 옵션은 내보낼 때 데이터를 청크로 분리한 다음, 가져온 후 청크를 다시 어셈블하는 것입니다. 

- PolyBase는 \n 또는 새 줄의 고정된 행 종결자를 사용합니다. 새 줄 문자가 원본 데이터에 표시되는 경우 문제가 발생할 수 있습니다.

- 원본 데이터 스키마는 SQL Data Warehouse에서 지원되지 않는 데이터 형식을 포함할 수 있습니다.

이러한 제한 사항을 해결하기 위해 필요한 전환을 수행하는 저장 프로시저를 만들 수 있습니다. bcp를 실행할 때 이 저장 프로시저를 참조합니다. 또는 [Redgate Data Platform Studio](/azure/sql-data-warehouse/sql-data-warehouse-load-with-redgate)에서 SQL Data Warehouse에서 지원되지 않는 데이터 형식을 자동으로 변환합니다.

자세한 내용은 다음 문서를 참조하세요.

- [Azure SQL Data Warehouse에 데이터를 로드하는 모범 사례](/azure/sql-data-warehouse/guidance-for-loading-data)
- [SQL Data Warehouse로 스키마 마이그레이션](/azure/sql-data-warehouse/sql-data-warehouse-migrate-schema)
- [SQL Data Warehouse의 테이블에 대한 데이터 형식을 정의하기 위한 지침](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types)

### <a name="transform-the-data"></a>데이터 변환

데이터를 변환하고 프로덕션 테이블로 이동합니다. 이 단계에서 데이터는 의미 체계 모델링에 적합한 차원 테이블 및 팩트 테이블을 사용하여 별모양 스키마로 변환됩니다.

전반적으로 최적의 쿼리 성능을 제공하는 클러스터형 columnstore 인덱스가 포함된 프로덕션 테이블을 만듭니다. Columnstore 인덱스는 많은 레코드를 검색하는 쿼리에 최적화됩니다. Columnstore 인덱스는 싱글톤 조회(즉, 단일 행 조회)도 수행하지 않습니다. 자주 싱글톤 조회를 수행해야 하는 경우 테이블에 비클러스터형 인덱스를 추가할 수 있습니다. 싱글톤 조회는 비클러스터형 인덱스를 사용하여 훨씬 더 빠르게 실행할 수 있습니다. 그러나 싱글톤 조회는 일반적으로 OLTP 작업보다 데이터 웨어하우스 시나리오에서 덜 일반적입니다. 자세한 내용은 [SQL Data Warehouse의 테이블 인덱싱](/azure/sql-data-warehouse/sql-data-warehouse-tables-index)을 참조하세요.

> [!NOTE]
> 클러스터형 columnstore 테이블은 `varchar(max)`, `nvarchar(max)` 또는 `varbinary(max)` 데이터 형식을 지원하지 않습니다. 이 경우 힙 또는 클러스터형 인덱스를 고려합니다. 별도 테이블에 해당 열을 넣을 수 있습니다.

샘플 데이터베이스는 매우 크지 않으므로 파티션이 없는 복제된 테이블을 만들었습니다. 프로덕션 작업의 경우 분산 테이블을 사용하면 쿼리 성능을 개선할 수 있습니다. [Azure SQL Data Warehouse의 분산 테이블 디자인 지침](/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute)을 참조하세요. 예제 스크립트는 정적 [리소스 클래스](/azure/sql-data-warehouse/resource-classes-for-workload-management)를 사용하여 쿼리를 실행합니다.

### <a name="load-the-semantic-model"></a>의미 체계 모델 로드

Azure Analysis Services에서 테이블 형식 모델로 데이터를 로드합니다. 이 단계에서는 SSDT(SQL Server Data Tools)를 사용하여 의미 체계 데이터 모델을 만듭니다. 또한 Power BI Desktop 파일에서 가져와서 모델을 만들 수도 있습니다. SQL Data Warehouse는 외래 키를 지원하지 않으므로 테이블에서 조인할 수 있도록 관계를 의미 체계 모델에 추가해야 합니다.

### <a name="use-power-bi-to-visualize-the-data"></a>Power BI를 사용하여 데이터 시각화

Power BI는 Azure Analysis Services에 연결하기 위한 두 가지 옵션을 지원합니다.

- 가져오기 데이터를 Power BI 모델로 가져옵니다.
- 라이브 연결 데이터를 Analysis Services에서 직접 가져옵니다.

Power BI 모델로 데이터를 복사할 필요가 없기 때문에 라이브 연결을 권장합니다. 또한 DirectQuery를 사용하면 결과는 항상 최신 원본 데이터와 일치하게 됩니다. 자세한 내용은 [Power BI로 연결](/azure/analysis-services/analysis-services-connect-pbi)을 참조하세요.

**권장 사항**

데이터 웨어하우스에 대해 직접 BI 대시보드 쿼리를 실행하지 마십시오. BI 대시보드는 웨어하우스에 대한 직접 쿼리가 충족할 수 없는 매우 낮은 응답 시간이 필요합니다. 또한 대시보드를 새로 고치면 성능에 영향을 줄 수 있는 동시 쿼리 수에 불리하게 간주됩니다. 

Azure Analysis Services는 BI 대시보드의 쿼리 요구 사항을 처리하도록 설계되었으므로 Power BI에서 Analysis Services를 쿼리하는 것이 좋습니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

### <a name="sql-data-warehouse"></a>SQL Data Warehouse

SQL Data Warehouse를 사용하여 주문형 계산 리소스를 확장할 수 있습니다. 쿼리 엔진은 계산 노드 수에 따라 병렬 처리에 대한 쿼리를 최적화하고, 필요에 따라 노드 간에 데이터를 이동합니다. 자세한 내용은 [Azure SQL Data Warehouse에서 계산 관리](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview)를 참조하세요.

### <a name="analysis-services"></a>Analysis Services

프로덕션 작업의 경우 Azure Analysis Services에 대한 표준 계층은 분할 및 DirectQuery를 지원하므로 이를 권장합니다. 계층 내에서 인스턴스 크기는 메모리 및 처리 용량을 결정합니다. 처리 능력은 QPU(쿼리 처리 단위)로 측정됩니다. QPU 사용량을 모니터링하여 적절한 크기를 선택합니다. 자세한 내용은 [서버 메트릭 모니터링](/azure/analysis-services/analysis-services-monitor)을 참조하세요.

높은 부하 상태에서 쿼리 성능은 쿼리 동시성으로 인해 저하될 수 있습니다. 더 많은 쿼리를 동시에 수행할 수 있도록 쿼리를 처리하는 복제본의 풀을 만들어 Analysis Services를 확장할 수 있습니다. 데이터 모델 처리 작업은 항상 주 서버에서 발생합니다. 기본적으로 주 서버도 쿼리를 처리합니다. 필요에 따라 쿼리 풀이 모든 쿼리를 처리하도록 단독으로 처리를 실행하도록 주 서버를 지정할 수 있습니다. 높은 처리 요구 사항이 있는 경우 쿼리 풀에서 처리를 구분해야 합니다. 높은 쿼리 부하와 상대적으로 약한 처리가 있는 경우 쿼리 풀에 주 서버를 포함할 수 있습니다. 자세한 내용은 [Azure Analysis Services 확장](/azure/analysis-services/analysis-services-scale-out)을 참조하세요. 

불필요한 처리 시간을 줄이기 위해 테이블 형식 모델을 논리적 부분으로 분할하는 데 파티션을 사용하는 것이 좋습니다. 각 파티션은 개별적으로 처리될 수 있습니다. 자세한 내용은 [파티션](/sql/analysis-services/tabular-models/partitions-ssas-tabular)을 참조하세요.

## <a name="security-considerations"></a>보안 고려사항

### <a name="ip-whitelisting-of-analysis-services-clients"></a>Analysis Services 클라이언트의 IP 허용 목록

클라이언트 IP 주소를 허용 목록에 추가하는 데 Analysis Services 방화벽 기능을 사용하는 것이 좋습니다. 활성화된 경우 방화벽은 방화벽 규칙에 지정된 것 이외의 모든 클라이언트 연결을 차단합니다. 기본 규칙은 Power BI 서비스를 허용 목록에 추가하지만 필요한 경우 이 규칙을 비활성화할 수 있습니다. 자세한 내용은 [새 방화벽 기능을 사용하여 Azure Analysis Services 보안 강화](https://azure.microsoft.com/blog/hardening-azure-analysis-services-with-the-new-firewall-capability/)를 참조하세요.

### <a name="authorization"></a>권한 부여

Azure Analysis Services는 Azure AD(Azure Active Directory)를 사용하여 Analysis Services 서버에 연결하는 사용자를 인증합니다. 역할을 만든 다음, 해당 역할에 Azure AD 사용자 또는 그룹을 할당하여 특정 사용자가 볼 수 있는 데이터를 제한할 수 있습니다. 각 역할의 경우 다음을 수행할 수 있습니다. 

- 테이블 또는 개별 열을 보호합니다. 
- 필터 식에 따라 개별 행을 보호합니다. 

자세한 내용은 [데이터베이스 역할 및 사용자 관리](/azure/analysis-services/analysis-services-database-users)를 참조하세요.

## <a name="deploy-the-solution"></a>솔루션 배포

이 참조 아키텍처에 대한 배포는 [GitHub][ref-arch-repo-folder]에서 사용할 수 있습니다. 다음을 배포합니다.

  * 온-프레미스 데이터베이스 서버를 시뮬레이션하는 Windows VM Power BI Desktop과 함께 SQL Server 2017 및 관련된 도구를 포함합니다.
  * SQL Server 데이터베이스에서 가져온 데이터를 저장할 Blob 저장소를 제공하는 Azure 저장소 계정
  * Azure SQL Data Warehouse 인스턴스
  * Azure Analysis Services 인스턴스

### <a name="prerequisites"></a>필수 조건

1. [Azure 참조 아키텍처][ref-arch-repo] GitHub 리포지토리의 zip 파일을 복제, 포크 또는 다운로드합니다.

2. [Azure 빌딩 블록][azbb-wiki](azbb)을 설치합니다.

3. 명령 프롬프트, bash 프롬프트 또는 PowerShell 프롬프트에서 아래 명령을 사용하여 Azure 계정에 로그인하고, 지침을 따릅니다.

  ```bash
  az login  
  ```

### <a name="deploy-the-simulated-on-premises-server"></a>시뮬레이션된 온-프레미스 서버 배포

먼저 SQL Server 2017 및 관련된 도구를 포함하는 시뮬레이션된 온-프레미스 서버로 VM을 배포합니다. 이 단계는 또한 샘플 [Wide World Importers OLTP 데이터베이스](/sql/sample/world-wide-importers/wide-world-importers-oltp-database)를 SQL Server로 로드합니다.

1. 위의 필수 조건에서 다운로드한 리포지토리의 `data\enterprise-bi-sqldw\onprem\templates` 폴더로 이동합니다.

2. `onprem.parameters.json` 파일에서 `adminUsername` 및 `adminPassword`에 대한 값을 대체합니다. 또한 `SqlUserCredentials` 섹션의 값을 사용자 이름 및 암호와 일치하도록 변경합니다. userName 속성의 `.\\` 접두사를 적어 둡니다.
    
    ```bash
    "SqlUserCredentials": {
      "userName": ".\\username",
      "password": "password"
    }
    ```

3. 아래와 같이 `azbb`를 실행하여 온-프레미스 서버를 배포합니다.

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <location> -p onprem.parameters.json --deploy
    ```

4. 배포는 완료하는 데 20~30분이 걸릴 수 있습니다. 이는 도구를 설치하고 데이터베이스를 복원하는 [DSC](/powershell/dsc/overview) 스크립트 실행을 포함합니다. 리소스 그룹에서 리소스를 검토하여 Azure Portal에서 배포를 확인합니다. `sql-vm1` 가상 머신 및 연결된 리소스가 표시됩니다.

### <a name="deploy-the-azure-resources"></a>Azure 리소스 배포

이 단계는 Storage 계정과 함께 Azure SQL Data Warehouse 및 Azure Analysis Services를 프로비전합니다. 원하는 경우에 이전 단계와 동시에 이 단계를 실행할 수 있습니다.

1. 위의 필수 조건에서 다운로드한 리포지토리의 `data\enterprise-bi-sqldw\azure\templates` 폴더로 이동합니다.

2. 다음 Azure CLI 명령을 실행하여 지정된 대괄호로 묶은 매개 변수를 대체하는 리소스 그룹을 만듭니다. 이전 단계에서 온-프레미스 서버에 대해 사용한 것과 다른 리소스 그룹에 배포할 수 있습니다. 

    ```bash
    az group create --name <resource_group_name> --location <location>  
    ```

3. 다음 Azure CLI 명령을 실행하여 지정된 대괄호로 묶은 매개 변수를 대체하는 Azure 리소스를 배포합니다. `storageAccountName` 매개 변수는 Storage 계정에 대한 [명명 규칙](../../best-practices/naming-conventions.md#naming-rules-and-restrictions)을 따라야 합니다. `analysisServerAdmin` 매개 변수의 경우 Azure Active Directory UPN(사용자 계정 이름)을 사용합니다.

    ```bash
    az group deployment create --resource-group <resource_group_name> --template-file azure-resources-deploy.json --parameters "dwServerName"="<server_name>" "dwAdminLogin"="<admin_username>" "dwAdminPassword"="<password>" "storageAccountName"="<storage_account_name>" "analysisServerName"="<analysis_server_name>" "analysisServerAdmin"="user@contoso.com"
    ```

4. 리소스 그룹에서 리소스를 검토하여 Azure Portal에서 배포를 확인합니다. 저장소 계정, Azure SQL Data Warehouse 인스턴스 및 Analysis Services 인스턴스가 표시됩니다.

5. Azure Portal을 사용하여 저장소 계정에 대한 액세스 키를 가져옵니다. 저장소 계정을 선택하여 엽니다. **설정** 아래에서 **액세스 키**를 선택합니다. 기본 키 값을 복사합니다. 다음 단계에서 사용하게 됩니다.

### <a name="export-the-source-data-to-azure-blob-storage"></a>Azure Blob 저장소로 원본 데이터 내보내기 

이 단계에서는 bcp를 사용하여 VM의 플랫 파일에 SQL 데이터베이스를 내보낸 다음, AzCopy를 사용하여 Azure Blob Storage로 해당 파일을 복사하는 PowerShell 스크립트를 실행합니다.

1. 원격 데스크톱을 사용하여 시뮬레이션된 온-프레미스 VM에 연결합니다.

2. VM에 로그인하는 동안 PowerShell 창에서 다음 명령을 실행합니다.  

    ```powershell
    cd 'C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\onprem'

    .\Load_SourceData_To_Blob.ps1 -File .\sql_scripts\db_objects.txt -Destination 'https://<storage_account_name>.blob.core.windows.net/wwi' -StorageAccountKey '<storage_account_key>'
    ```

    `Destination` 매개 변수의 경우 이전에 만든 Storage 계정의 이름으로 `<storage_account_name>`을 대체합니다. `StorageAccountKey` 매개 변수의 경우 해당 Storage 계정에 대한 액세스 키를 사용합니다.

3. Azure Portal에서 저장소 계정으로 이동하고, Blob 서비스를 선택하고, `wwi` 컨테이너를 열어 원본 데이터가 Blob 저장소로 복사되었는지 확인합니다. 앞에 `WorldWideImporters_Application_*`이 추가된 테이블 목록이 표시됩니다.

### <a name="execute-the-data-warehouse-scripts"></a>데이터 웨어하우스 스크립트 실행

1. 원격 데스크톱 세션에서 SSMS(SQL Server Management Studio)를 시작합니다. 

2. SQL Data Warehouse에 연결

    - 서버 유형: 데이터베이스 엔진
    
    - 서버 이름: `<dwServerName>.database.windows.net`, 여기서 `<dwServerName>`은 Azure 리소스를 배포했을 때 지정한 이름입니다. Azure Portal에서 이 이름을 가져올 수 있습니다.
    
    - 인증: SQL Server 인증 `dwAdminLogin` 및 `dwAdminPassword` 매개 변수에서 Azure 리소스를 배포했을 때 지정한 자격 증명을 사용합니다.

2. VM에서 `C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw\azure\sqldw_scripts` 폴더로 이동합니다. `STEP_1`에서 `STEP_7`로 번호순으로 이 폴더에서 스크립트를 실행합니다.

3. SSMS에서 `master` 데이터베이스를 선택하고 `STEP_1` 스크립트를 엽니다. 다음 줄에서 암호의 값을 변경한 다음, 스크립트를 실행합니다.

    ```sql
    CREATE LOGIN LoaderRC20 WITH PASSWORD = '<change this value>';
    ```

4. SSMS에서 `wwi` 데이터베이스를 선택합니다. `STEP_2` 스크립트를 열고 스크립트를 실행합니다. 오류가 발생하는 경우 `master`가 아닌 `wwi` 데이터베이스에 대해 스크립트를 실행하고 있는지 확인합니다.

5. `STEP_1` 스크립트에 표시된 `LoaderRC20` 사용자 및 암호를 사용하여 SQL Data Warehouse에 대한 새 연결을 엽니다.

6. 이 연결을 사용하여 `STEP_3` 스크립트를 엽니다. 스크립트에서 다음 값을 설정합니다.

    - SECRET: 저장소 계정에 대한 액세스 키를 사용합니다.
    - LOCATION: `wasbs://wwi@<storage_account_name>.blob.core.windows.net`과 같이 저장소 계정의 이름을 사용합니다.

7. 동일한 연결을 사용하여 `STEP_4`에서 `STEP_7`로 순차적으로 스크립트를 실행합니다. 각 스크립트에서 다음을 실행하기 전에 제대로 완료하는지 확인합니다.

SMSS에서 `wwi` 데이터베이스에 `prd.*` 테이블의 집합이 표시됩니다. 데이터가 생성되었는지 확인하려면 다음 쿼리를 실행합니다. 

```sql
SELECT TOP 10 * FROM prd.CityDimensions
```

### <a name="build-the-azure-analysis-services-model"></a>Azure Analysis Services 모델 빌드

이 단계에서는 데이터 웨어하우스에서 데이터를 가져오는 테이블 형식 모델을 만듭니다. 그런 다음, Azure Analysis Services에 모델을 배포합니다.

1. 원격 데스크톱 세션에서 SQL Server Data Tools 2015를 시작합니다.

2. **파일** > **새로 만들기** > **프로젝트**를 선택합니다.

3. **새 프로젝트** 대화 상자의 **템플릿** 아래에서 **비즈니스 인텔리전스** > **Analysis Services** > **Analysis Services 테이블 형식 프로젝트**를 선택합니다. 

4. 프로젝트 이름을 지정하고 **확인**을 클릭합니다.

5. **테이블 형식 모델 디자이너** 대화 상자에서 **통합된 작업 영역**을 선택하고 **호환성 수준**을 `SQL Server 2017 / Azure Analysis Services (1400)`로 설정합니다. **확인**을 클릭합니다.

6. **테이블 형식 모델 탐색기** 창에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 **데이터 원본에서 가져오기**를 선택합니다.

7. **Azure SQL Data Warehouse**를 선택하고 **연결**을 클릭합니다.

8. **서버**에 Azure SQL Data Warehouse 서버의 정규화된 이름을 입력합니다. **데이터베이스**에 `wwi`를 입력합니다. **확인**을 클릭합니다.

9. 다음 대화 상자에서 **데이터베이스** 인증을 선택하고 Azure SQL Data Warehouse 사용자 이름 및 암호를 입력하고, **확인**을 클릭합니다.

10. **탐색기** 대화 상자에서 **prd.CityDimensions**, **prd.DateDimensions** 및 **prd.SalesFact**에 대한 확인란을 선택합니다. 

    ![](./images/analysis-services-import.png)

11. **로드**를 클릭합니다. 처리가 완료되면 **닫기**를 클릭합니다. 이제 데이터의 테이블 형식 보기가 표시됩니다.

12. **테이블 형식 모델 탐색기** 창에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 **모델 보기** > **다이어그램 보기**를 선택합니다.

13. **[prd.SalesFact].[WWI City ID]** 필드를 **[prd.CityDimensions].[WWI City ID]** 필드로 끌어서 관계를 만듭니다.  

14. **[prd.SalesFact].[Invoice Date Key]** 필드를 **[prd.DateDimensions].[Date]** 필드로 끕니다.  
    ![](./images/analysis-services-relations.png)

15. **파일** 메뉴에서 **모두 저장**을 선택합니다.  

16. **솔루션 탐색기**에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 **속성**을 선택합니다. 

17. **서버** 아래에서 Azure Analysis Services 인스턴스의 URL을 입력합니다. Azure Portal에서 이 값을 가져올 수 있습니다. 포털에서 Analysis Services 리소스를 선택하고, 개요 창을 클릭하고, **서버 이름** 속성을 찾습니다. `asazure://westus.asazure.windows.net/contoso`와 유사합니다. **확인**을 클릭합니다.

    ![](./images/analysis-services-properties.png)

18. **솔루션 탐색기**에서 프로젝트를 마우스 오른쪽 단추로 클릭하고 **배포**를 선택합니다. 메시지가 표시되면 Azure에 로그인 합니다. 처리가 완료되면 **닫기**를 클릭합니다.

19. Azure Portal에서 Azure Analysis Services 인스턴스에 대한 정보를 봅니다. 모델이 모델 목록에 표시되는지 확인합니다.

    ![](./images/analysis-services-models.png)

### <a name="analyze-the-data-in-power-bi-desktop"></a>Power BI Desktop에서 데이터 분석

이 단계에서는 Power BI를 사용하여 Analysis Services의 데이터에서 보고서를 만듭니다.

1. 원격 데스크톱 세션에서 Power BI Desktop을 시작합니다.

2. 시작 화면에서 **데이터 가져오기**를 클릭합니다.

3. **Azure** > **Azure Analysis Services 데이터베이스**를 선택합니다. **연결**

    ![](./images/power-bi-get-data.png)

4. Analysis Services 인스턴스의 URL을 입력한 다음, **확인**을 클릭합니다. 메시지가 표시되면 Azure에 로그인 합니다.

5. **탐색기** 대화 상자에서 배포한 테이블 형식 프로젝트를 확장하고, 만든 모델을 선택하고 **확인**을 클릭합니다.

2. **시각화** 창에서 **누적 가로 막대형 차트** 아이콘을 선택합니다. 보고서 보기에서 시각화의 크기를 더 크게 조정합니다.

6. **필드** 창에서 **prd.CityDimensions**를 확장합니다.

7. **prd.CityDimensions** > **WWI City ID**를 **축**으로 끕니다.

8. **prd.CityDimensions** > **City**를 **범례**로 끕니다.

9. 필드 창에서 **prd.SalesFact**를 확장합니다.

10. **prd.SalesFact** > **Total Excluding Tax**를 **값**으로 끕니다.

    ![](./images/power-bi-visualization.png)

11. **시각적 수준 필터**아래에서 **WWI City ID**를 선택합니다.

12. **필터 형식**을 `Top N`으로 설정하고, **표시 항목**을 `Top 10`으로 설정합니다.

13. **prd.SalesFact** > **Total Excluding Tax**를 **값별**로 끕니다.

    ![](./images/power-bi-visualization2.png)

14. **필터 적용**을 클릭합니다. 시각화는 도시별로 상위 10개의 총 판매액을 보여줍니다.

    ![](./images/power-bi-report.png)

Power BI Desktop에 대해 자세히 알아보려면 [Power BI Desktop 시작](/power-bi/desktop-getting-started)을 참조하세요.

## <a name="next-steps"></a>다음 단계

- 이 참조 아키텍처에 대한 자세한 내용은 [GitHub 리포지토리][ref-arch-repo-folder]를 참조하세요.
- [Azure 빌딩 블록][azbb-repo]에 대해 알아봅니다.

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw

