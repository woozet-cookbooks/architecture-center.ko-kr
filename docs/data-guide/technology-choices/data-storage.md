---
title: 데이터 저장소 기술 선택
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b14611a2dc34bcb145cf420441795d4124e7baeb
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-big-data-storage-technology-in-azure"></a>Azure의 빅 데이터 저장소 기술 선택

이 항목에서는 [분석 데이터 저장소](./analytical-data-stores.md) 또는 [실시간 스트리밍 수집](./real-time-ingestion.md)과는 반대로, 빅 데이터 솔루션&mdash;에 대한 데이터 저장소 옵션, 특히 대량 데이터 수집 및 일괄 처리를 위한 데이터 저장소에 대해 비교합니다.

## <a name="what-are-your-options-when-choosing-data-storage-in-azure"></a>Azure에서 데이터 저장소를 선택할 때의 옵션은 무엇인가요?

Azure에서는 사용자의 요구에 따라 다음과 같은 몇 가지 데이터 수집 옵션을 사용할 수 있습니다.

**File Storage**

- [Azure Storage Blob](/azure/storage/blobs/storage-blobs-introduction)
- [Azure Data Lake Storage](/azure/data-lake-store/)

**NoSQL 데이터베이스**

- [Azure Cosmos DB](/azure/cosmos-db/)
- [HDInsight의 HBase](http://hbase.apache.org/)

## <a name="azure-storage-blobs"></a>Azure Storage Blob

Azure Storage는 가용성, 보안, 내구성, 확장성 및 중복성이 높은 관리되는 저장소 서비스입니다. Microsoft는 유지 관리를 담당하고 사용자에 대한 중요한 문제를 처리합니다. Azure Storage는 함께 사용할 수 많은 서비스 및 도구 때문에, Azure에서 제공하는 가장 보편적인 저장소 솔루션입니다.

다양한 Azure Storage 서비스를 사용하여 데이터를 저장할 수 있습니다. 다양한 데이터 원본의 Blob을 저장하는 가장 유연한 옵션은 [Blob 저장소](/azure/storage/blobs/storage-blobs-introduction)입니다. Blob은 기본적으로 파일입니다. 사진, 문서, HTML 파일, VHD(가상 하드 디스크), 로그와 같은 빅 데이터, 데이터베이스 백업 등 거의 모든 항목을 저장합니다. Blob은 폴더와 유사한 컨테이너에 저장됩니다. 컨테이너는 Blob 집합의 그룹화를 제공합니다. 한 저장소 계정에 포함될 수 있는 컨테이너 수에 제한이 없으며, 컨테이너에 저장될 수 있는 Blob 수에도 제한이 없습니다.

Azure Storage는 유연성, 고가용성 및 저렴한 비용으로 인해 빅 데이터 및 분석 솔루션에 적합합니다. 다양한 사용 사례에 맞게 핫 저장소 계층, 쿨 저장소 계층 및 보관 저장소 계층을 제공합니다. 자세한 내용은 [Azure Blob Storage: 핫, 쿨 및 보관 저장소 계층](/azure/storage/blobs/storage-blob-storage-tiers)을 참조하세요.

Azure Blob 저장소는 Hadoop(HDInsight를 통해 사용 가능)에서 액세스할 수 있습니다. HDInsight는 Azure Storage의 Blob 컨테이너를 클러스터의 기본 파일 시스템으로 사용합니다. WASB 드라이버에서 제공하는 HDFS(Hadoop Distributed File System) 인터페이스를 통해 HDInsight의 전체 구성 요소 집합을 Blob로 저장된 구조적 또는 비구조적 데이터에 대해 직접 작동할 수 있습니다. Azure Blob 저장소는 PolyBase 기능을 사용하여 Azure SQL Data Warehouse를 통해 액세스할 수도 있습니다.

Azure Storage의 선택 가능성을 높이는 기타 기능에는 다음이 포함됩니다.

- [여러 동시성 전략](/azure/storage/common/storage-concurrency?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)합니다.
- [재해 복구 및 고가용성 옵션](/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [휴지 상태의 암호화](/azure/storage/common/storage-service-encryption?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [RBAC(역할 기반 액세스 제어)](/azure/storage/common/storage-security-guide?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#management-plane-security): Azure Active Directory 사용자 및 그룹을 사용하여 액세스 제어

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

[Azure Data Lake Store](/azure/data-lake-store/)는 빅 데이터 분석 작업을 위한 엔터프라이즈 수준 하이퍼 스케일 리포지토리입니다. Data Lake를 사용하면 작동 및 예비 분석에 대해 하나의 [보안](/azure/data-lake-store/data-lake-store-overview#DataLakeStoreSecurity) 위치에 모든 크기, 형식 및 수집 속도의 데이터를 캡처할 수 있습니다.

Data Lake에 저장될 수 있는 계정 크기, 파일 크기 또는 데이터 양에 어떠한 제한도 적용하지 않습니다. 데이터는 여러 복사본을 만들어 영구적으로 저장되며 데이터가 Data Lake에 저장될 수 있는 기간에 제한이 없습니다. Data Lake는 예기치 않은 오류로부터 보호하기 위해 파일의 여러 복사본을 만드는 것 외에도, 여러 개별 저장소 서버에 파일 부분을 분산합니다. 데이터 분석을 수행하기 위해 병렬로 파일을 읽을 때 읽기 처리량이 향상됩니다.

Data Lake Store는 WebHDFS 호환 REST API를 사용하여 Hadoop(HDInsight를 통해 사용 가능)에서 액세스할 수 있습니다. 개별 파일 크기 또는 조합된 파일 크기가 Azure Storage에서 지원하는 크기를 초과하는 경우 Azure Storage 대신 Data Lake Store를 사용할 수 있습니다. 그러나 HDInsight 클러스터의 기본 저장소로 Data Lake Store를 사용할 때는 [Spark](/azure/data-lake-store/data-lake-store-performance-tuning-spark), [ Hive](/azure/data-lake-store/data-lake-store-performance-tuning-hive), [MapReduce](/azure/data-lake-store/data-lake-store-performance-tuning-mapreduce) 및 [Storm](/azure/data-lake-store/data-lake-store-performance-tuning-storm)에 대한 특정 지침과 함께 [성능 튜닝 지침](/azure/data-lake-store/data-lake-store-performance-tuning-guidance#optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight)도 따라야 합니다. 또한 Data Lake Store는 Azure Storage의 경우만큼 많은 지역에서 사용할 수는 없으며, HDInsight 클러스터와 동일한 지역에 있어야 하므로 해당 [국가별 가용성](https://azure.microsoft.com/regions/#services)을 확인해야 합니다.

Azure Data Lake Analytics와 함께 사용되는 Data Lake Store는 저장된 데이터에 대한 분석을 사용하도록 특별히 설계되었으며 데이터 분석 시나리오에 대한 성능을 위해 조정됩니다. Data Lake Store는 PolyBase 기능을 사용하여 Azure SQL Data Warehouse를 통해 액세스할 수도 있습니다.

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

[Azure Cosmos DB](/azure/cosmos-db/)는 전 세계에 배포된 Microsoft의 멀티모델 데이터베이스입니다. Cosmos DB는 전 세계 어디서나 99 백분위수의 한 자리 밀리초 대기 시간을 보장하고, 제대로 정의된 여러 일관성 모델을 제공하여 성능을 미세 조정하고, 멀티 호밍 기능으로 고가용성을 보장합니다.

Azure Cosmos DB는 스키마에 구애받지 않습니다. 또한 사용자가 스키마 및 인덱스 관리를 처리하지 않아도 되도록 모든 데이터를 자동으로 인덱싱합니다. 또한 기본적으로 문서, 키-값, 그래프 및 열 패밀리 데이터 모델을 지원하는 다중 모델입니다. 

Azure DB Cosmos 기능은 다음과 같습니다.

- [지역에서 복제](/azure/cosmos-db/distribute-data-globally)
- 전 세계에 [처리량 및 저장소의 탄력적인 크기 조정](/azure/cosmos-db/partition-data)
- [잘 정의된 5개의 일관성 수준](/azure/cosmos-db/consistency-levels)

## <a name="hbase-on-hdinsight"></a>HDInsight의 HBase

[Apache HBase](http://hbase.apache.org/)는 Hadoop을 기반으로 하고 Google BigTable 이후에 모델링된 오픈 소스 NoSQL 데이터베이스입니다. HBase는 열 패밀리로 구성된 스키마 없는 데이터베이스에서 구조화되지 않은/반구조화된 대량 데이터에 대해 임의 액세스 및 강력한 일관성을 제공합니다.

데이터는 테이블의 행에 저장되고 행 내의 데이터는 열 제품군으로 그룹화됩니다. HBase는 사용 전에 열과 열에 저장되는 데이터 형식을 정의할 필요가 없다는 점에서 스키마 없는 데이터베이스입니다. 오픈 소스 코드는 수천 대의 노드에 있는 페타바이트 크기의 데이터를 처리할 수 있을 정도로 선형으로 확장됩니다. Hadoop 에코시스템의 분산 응용 프로그램이 제공하는 데이터 중복, 일괄 처리 및 기타 기능을 사용할 수 있습니다.

[HDInsight 구현](/azure/hdinsight/hbase/apache-hbase-overview)은 HBase의 규모 확장 아키텍처를 활용하여 테이블 자동 분할, 읽기 및 쓰기에 대한 강력한 일관성 및 자동 장애 조치(Failover)를 제공합니다. 읽기를 위한 메모리 내 캐싱과 쓰기를 위한 높은 처리량 스트리밍을 통해 성능이 향상됩니다. 대부분의 경우 다른 HDInsight 클러스터 및 응용 프로그램이 테이블에 직접 액세스할 수 있도록 [가상 네트워크 내에 HBase 클러스터를 만들 수 있습니다](/azure/hdinsight/hbase/apache-hbase-provision-vnet).

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 모든 종류의 텍스트 또는 이진 데이터에 대한 고속의 관리되는 클라우드 기반 저장소가 필요한가요? 그렇다면 파일 저장소 옵션 중 하나를 선택합니다.

- 병렬 분석 워크로드 및 높은 처리량/IOPS에 대해 최적화된 파일 저장소가 필요한가요? 그렇다면 분석 워크로드 성능에 맞춰 조정되는 옵션을 선택합니다.

- 스키마 없는 데이터베이스에 구조화되지 않았거나 반구조화된 데이터를 저장해야 하나요? 그렇다면 비관계형 옵션 중 하나를 선택합니다. 인덱싱 및 데이터베이스 모델에 대한 옵션을 비교합니다. 저장해야 하는 데이터의 형식에 따라, 주 데이터베이스 모델이 가장 큰 요인이 될 수 있습니다.

- 사용자의 지역에서 이 서비스를 사용할 수 있나요? 각 Azure 서비스에 대한 지역별 가용성을 확인합니다. [지역별 사용 가능 제품](https://azure.microsoft.com/regions/services/)을 참조하세요.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.

### <a name="file-storage-capabilities"></a>파일 저장소 기능

|  | Azure Data Lake Store | Azure Blob 저장소 컨테이너 |
| --- | --- | --- |
| 목적 | 빅 데이터 분석 워크로드에 대해 최적화된 저장소 |다양한 저장소 시나리오에 대한 범용 개체 저장소 |
| 사용 사례 | 일괄 처리, 스트리밍 분석 및 로그 파일, IoT 데이터, 클릭 스트림, 대형 데이터 집합 등과 같은 기계 학습 데이터 | 응용 프로그램 백 엔드, 백업 데이터, 스트리밍용 미디어 저장소 및 범용 데이터 등과 같은 모든 종류의 텍스트 또는 이진 데이터 |
| 구조 | 계층적 파일 시스템 | 단일 구조 네임스페이스를 가진 개체 저장소 |
| 인증 | [Azure Active Directory ID](/azure/active-directory/active-directory-authentication-scenarios) | 공유 비밀 기반 [계정 액세스 키](/azure/storage/common/storage-create-storage-account#manage-your-storage-account), [공유 액세스 서명 키](/azure/storage/common/storage-dotnet-shared-access-signature-part-1) 및 [RBAC(역할 기반 액세스 제어)](/azure/security/security-storage-overview) |
| 인증 프로토콜 | OAuth 2.0. 호출은 Azure Active Directory가 발급한 유효한 JWT(JSON Web Token)를 포함해야 합니다. | HMAC(해시 기반 메시지 인증 코드). 호출은 HTTP 요청 일부를 통해 Base64 인코딩된 SHA-256 해시를 포함해야 합니다. |
| 권한 부여 | POSIX ACL(액세스 제어 목록). Azure Active Directory ID에 따른 ACL은 파일 및 폴더 수준에서 설정할 수 있습니다. | 계정 수준 인증의 경우 [계정 액세스 키](/azure/storage/common/storage-create-storage-account#manage-your-storage-account)를 사용합니다. 계정, 컨테이너 또는 Blob 권한 부여의 경우 [공유 액세스 서명 키](/azure/storage/common/storage-dotnet-shared-access-signature-part-1)를 사용합니다. |
| 감사 | 사용 가능.  |사용 가능 |
| 휴지 상태의 암호화 | 투명한, 서버 쪽 | 투명한, 서버 쪽, 클라이언트 쪽 암호화 |
| 개발자 SDK | .NET, Java, Python, Node.js | .NET, Java, Python, Node.js, c + +, Ruby |
| 분석 워크로드 성능 | 병렬 분석 워크로드, 높은 처리량 및 IOPS에 대해 최적화된 성능 | 분석 워크로드에 대해 최적화되지 않음 |
| 크기 한도 | 계정 크기, 파일 크기 또는 파일 수에 한도가 없음 | 문서화된 특정 한도 [여기](/azure/azure-subscription-service-limits#storage-limits) |
| 지리적 중복 | 로컬 중복(Azure 하위 지역에 있는 데이터의 여러 복사본) | 로컬 중복(LRS), 전역 중복(GRS), 읽기 액세스 전역 중복(RA-GRS). 자세한 내용은 [여기](/azure/storage/common/storage-redundancy) 참조 |

### <a name="nosql-database-capabilities"></a>NoSQL 데이터베이스 기능

|                                    |                                           Azure Cosmos DB                                           |                                                             HDInsight의 HBase                                                             |
|------------------------------------|-----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|       주 데이터베이스 모델       |                      문서 저장소, 그래프, 키-값 저장소, 넓은 열 저장소                      |                                                             넓은 열 저장소                                                              |
|         보조 인덱스          |                                                 예                                                 |                                                                     아니요                                                                     |
|        SQL 언어 지원        |                                                 예                                                 |                                     예([Phoenix](http://phoenix.apache.org/) JDBC 드라이버 사용)                                      |
|            일관성             |                   강력, 제한된 부실, 세션, 일관적인 접두사, 최종                   |                                                                   강력                                                                   |
| 네이티브 Azure Functions 통합 |                        [예](/azure/cosmos-db/serverless-computing-database)                        |                                                                     아니오                                                                     |
|   자동 글로벌 배포    |                          [예](/azure/cosmos-db/distribute-data-globally)                           | 아니요 [HBase 클러스터 복제](/azure/hdinsight/hbase/apache-hbase-replication)를 최종 일관성을 갖는 지역 간에 구성할 수 있습니다. |
|           가격 책정 모델            | 탄력적으로 확장 가능한 RU(요청 단위)에 필요에 따라 초당 요금 부과, 탄력적으로 확장 가능한 저장소 |                              HDInsight 클러스터에 대해 분단위 가격 책정(수평 노드 확장), 저장소                               |

