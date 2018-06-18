---
title: 데이터 전송 기술 선택
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 53dcf8a69ad8ae100dbdbb230a9280efd419342a
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252756"
---
# <a name="transferring-data-to-and-from-azure"></a>Azure에서의 데이터 전송

사용자의 요구에 따라 Azure에서 몇 가지 데이터 전송 옵션을 사용할 수 있습니다.

## <a name="physical-transfer"></a>실제 전송

다음과 같은 경우에는 실제 하드웨어를 사용하여 Azure로 데이터를 전송하는 것이 좋습니다.

- 네트워크 속도가 느리거나 불안정한 경우
- 추가 네트워크 대역폭을 사용하는 데 드는 비용을 감수하기 어려운 경우
- 보안 또는 조직 정책이 중요한 데이터를 처리할 때 아웃바운드 연결을 허용하지 않는 경우 

데이터를 전송하는 데 소요되는 시간이 1차적으로 고려되는 경우, 테스트를 통해 네트워크 전송 방식이 물리적 전송보다 실제로 느린지 확인할 수 있습니다.

Azure에 물리적 방식으로 데이터를 전송하는 두 가지 옵션은 다음과 같습니다.
- **Azure Import/Export**. [Azure Import/Export 서비스](/azure/storage/common/storage-import-export-service)를 사용하면 내부 SATA HDD 또는 SDD를 Azure 데이터 센터에 배송하여 대량의 데이터를 Azure Blob 저장소 또는 Azure Files에 안전하게 전송할 수 있습니다. 이 서비스를 사용하여 데이터를 Azure Storage에서 하드 디스크 드라이브로 전송하고, 온-프레미스에 로드할 수 있게 사용자에게 배송할 수도 있습니다.

- **Azure Data Box**. [Azure Data Box](https://azure.microsoft.com/services/storage/databox/)는 Azure Import/Export 서비스처럼 작동하는 Microsoft에서 제공한 어플라이언스입니다. Microsoft는 독자적이며 안전한 변조 방지 전송 어플라이언스를 사용자에게 배송한 후, 종단 간 내부 프로세스를 처리합니다. 사용자는 이러한 프로세스를 포털을 통해 추적할 수 있습니다. Azure Data Box 서비스의 한 가지 이점은 사용 편의성입니다. 여러 하드 드라이브를 구입하고, 준비하고, 각 드라이브로 파일을 전송할 필요가 없습니다. Azure Data Box는 다양한 업계의 주도적인 Azure 파트너가 지원하므로, 제품에서 클라우드로의 오프라인 전송을 보다 쉽게 원활히 진행할 수 있습니다. 

## <a name="command-line-tools-and-apis"></a>명령줄 도구 및 API

스크립트 및 프로그래밍 방식으로 데이터를 전송하려는 경우 이러한 옵션을 고려합니다.

- **Azure CLI**. [Azure CLI](/azure/hdinsight/hdinsight-upload-data#commandline)는 Azure 서비스를 관리하고 Azure Storage로 데이터를 업로드할 수 있도록 하는 플랫폼 간 도구입니다. 

- **AzCopy**. [Windows](/azure/storage/common/storage-use-azcopy?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 또는 [Linux](/azure/storage/common/storage-use-azcopy-linux?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) 명령줄에서 AzCopy를 사용하여 최적의 성능으로 Azure Blob, File 및 Table Storage 간에 쉽게 데이터를 복사할 수 있습니다. AzCopy는 동시성 및 병렬 처리 기능과 중단된 복사 작업을 다시 시작하는 기능을 지원합니다. 또한 대부분의 다른 옵션보다 빠릅니다. 프로그래밍 방식 액세스에서 [Microsoft Azure Storage 데이터 이동 라이브러리](/azure/storage/common/storage-use-data-movement-library)는 AzCopy를 구동하는 핵심 프레임워크입니다. .NET Core 라이브러리로도 제공됩니다. 

- **PowerShell**. [`Start-AzureStorageBlobCopy` PowerShell cmdlet](/powershell/module/azure.storage/start-azurestorageblobcopy?view=azurermps-5.0.0)은 PowerShell을 수행하는 데 사용되는 Windows 관리자를 위한 옵션입니다.  

- **AdlCopy**. [AdlCopy](/azure/data-lake-store/data-lake-store-copy-data-azure-storage-blob)를 사용하면 Azure Storage Blob에서 Data Lake Store로 데이터를 복사할 수 있습니다. 또한 이 기능은 두 Azure Data Lake Store 계정 간에 데이터를 복사하는 데도 사용될 수 있습니다. 그러나 Data Lake Store에서 Storage Blob으로 데이터를 복사하는 데는 사용할 수 없습니다.

- **Distcp**. Data Lake Store에 액세스할 수 있는 HDInsight 클러스터를 갖고 있다면 [Distcp](/azure/data-lake-store/data-lake-store-copy-data-wasb-distcp) 같은 Hadoop 에코시스템 도구를 사용하여 HDInsight 클러스터 저장소(WASB)와 주고 받는 데이터를 Data Lake Store 계정에 복사할 수 있습니다.

- **Sqoop**. [Sqoop](/azure/hdinsight/hadoop/hdinsight-use-sqoop)는 Apache 프로젝트이며, Hadoop 에코시스템의 일부입니다. 모든 HDInsight 클러스터에 미리 설치됩니다. HDInsight 클러스터와 관계형 데이터베이스(예: SQL, Oracle, MySQL 등) 간의 데이터 전송을 허용합니다. Sqoop는 가져오기 및 내보내기를 포함하는 관련 도구의 컬렉션입니다. Sqoop는 Azure Storage Blob 또는 Data Lake Store 연결 저장소를 사용하는 HDInsight 클러스터에 작동합니다.

- **PolyBase**. [PolyBase](/sql/relational-databases/polybase/get-started-with-polybase)는 T-SQL 언어를 통해 데이터베이스 외부의 데이터에 액세스하는 기술입니다. SQL Server 2016에서는 이 기술을 통해 Hadoop에서 외부 데이터에 대해 쿼리를 실행하거나 Azure Blob 저장소에서 데이터를 가져오거나 내보낼 수 있습니다. Azure SQL Data Warehouse에서는 Azure Blob 저장소 및 Azure Data Lake Store에서 데이터를 가져오거나 내보낼 수 있습니다. 현재, PolyBase는 SQL Data Warehouse로 데이터를 가져오는 가장 빠른 방법입니다.

- **Hadoop 명령줄**. HDInsight 클러스터 헤드 노드에 데이터가 있는 경우 `hadoop -copyFromLocal` 명령을 사용하여 해당 데이터를 Azure Storage Blob 또는 Azure Data Lake Store 같은 클러스터 연결 저장소로 복사할 수 있습니다. Hadoop 명령을 사용하려면 먼저 헤드 노드에 연결해야 합니다. 연결되면 저장소에 파일을 업로드할 수 있습니다.

## <a name="graphical-interface"></a>그래픽 인터페이스

소수의 파일 또는 데이터 개체만 전송하려고 하며 프로세스를 자동화할 필요는 없다면 다음 옵션을 고려합니다.

- **Azure Storage 탐색기**. [Azure Storage 탐색기](https://azure.microsoft.com/features/storage-explorer/)는 Azure Storage 계정의 내용을 관리할 수 있는 플랫폼 간 도구입니다. BLOB, 파일, 큐, 테이블 및 Azure Cosmos DB 엔터티를 업로드, 다운로드 및 관리할 수 있습니다. Blob 저장소에서 사용하여 Blob 및 폴더를 관리하고, 로컬 파일 시스템과 Blob 저장소 간에 또는 저장소 계정 간에 Blob을 업로드 및 다운로드할 수 있습니다.

- **Azure Portal**. Blob 저장소와 Data Lake Store 둘 다 파일을 탐색한 호 새 파일을 한 번에 하나씩 업로드하기 위한 웹 기반 인터페이스를 제공합니다. 파일을 빠르게 탐색하거나 많은 새 파일을 간편하게 업로드하기 위해 도구를 설치하거나 명령을 실행하는 방식을 원치 않을 경우에 유용한 옵션입니다.

## <a name="data-pipeline"></a>데이터 파이프라인

**Azure Data Factory**. [Azure Data Factory](/azure/data-factory/)는 다양한 Azure 서비스, 온-프레미스 또는 두 환경의 조합 간에 정기적으로 파일을 전송하는 데 가장 적합한 관리되는 서비스입니다. Azure Data Factory를 사용하여 서로 다른 데이터 저장소의 데이터를 수집하는 데이터 기반 워크플로(파이프라인이라고 함)를 만들고 예약할 수 있습니다. Azure HDInsight Hadoop, Spark, Azure Data Lake Analytics 및 Azure Machine Learning과 같은 계산 서비스를 사용하여 데이터를 처리하고 변환할 수 있습니다. 데이터 이동 및 데이터 변환을 [조정](../technology-choices/pipeline-orchestration-data-movement.md)하고 자동화하기 위한 데이터 기반 워크플로를 만듭니다.

## <a name="key-selection-criteria"></a>주요 선택 조건

데이터 전송 시나리오에 대해, 다음 질문에 답변하여 요구에 적합한 시스템을 선택합니다.

- 많은 양의 데이터를 전송해야 하는데, 인터넷을 통해 전송하면 너무 오래 걸리거나, 신뢰할 수 없거나, 비용이 너무 많이 걸리나요? 그렇다면 실제 전송을 고려합니다.

- 다시 사용할 수 있도록 데이터 전송 태스크를 스크립트로 작성하고 싶나요? 그렇다면 명령줄 옵션 또는 Azure Data Factory 중 하나를 선택합니다.

- 네트워크 연결을 통해 대량의 데이터를 전송해야 하나요? 그렇다면 빅 데이터에 최적화된 옵션을 선택합니다.

- 관계형 데이터베이스 간에 데이터를 전송해야 하나요? 그렇다면 하나 이상의 관계형 데이터베이스를 지원하는 옵션을 선택합니다. 이러한 옵션 중 일부에는 Hadoop 클러스터도 필요합니다.

- 자동화된 데이터 파이프라인 또는 워크플로 오케스트레이션이 필요한가요? 그렇다면 Azure Data Factory를 고려합니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.

### <a name="physical-transfer"></a>실제 전송

| | Azure Import/Export 서비스 | Azure Data Box |
| --- | --- | --- |
| 폼 팩터 | 내부 SATA HDD 또는 SDD | 안전한, 변조 방지, 단일 하드웨어 어플라이언스 |
| Microsoft에서 배송 내부 프로세스 관리 | 아니오 | 예 |
| 파트너 제품과의 통합 | 아니오 | 예 |
| 사용자 지정 어플라이언스 | 아니오 | 예 |

### <a name="command-line-tools"></a>명령줄 도구.

**Hadoop/HDInsight**

| | Distcp | Sqoop | Hadoop CLI |
| --- | --- | --- | --- |
| 빅 데이터에 최적화 | 예 | 예 |  예 |
| 관계형 데이터베이스로 복사 |  아니오 | 예 | 아니오 |
| 관계형 데이터베이스에서 복사 |  아니오 | 예 | 아니오 |
| Blob 저장소로 복사 |  예 | 예 | 예 |
| Blob 저장소에서 복사 | 예 |  예 | 아니오 |
| Data Lake Store로 복사 | 예 | 예 | 예 |
| Data Lake Store에서 복사 | 예 | 예 | 아니오 |

**기타**

| | Azure CLI | AzCopy | PowerShell | AdlCopy | PolyBase |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 호환 플랫폼 | Linux, OS X, Windows | Linux, Windows | Windows | Linux, OS X, Windows | SQL Server, Azure SQL Data Warehouse | 
| 빅 데이터에 최적화 | 아니오 | 아니요 | 아니오 | 예 <sup>1</sup> | 예 <sup>2</sup> |
| 관계형 데이터베이스로 복사 | 아니오 | 아니요 | 아니요 | 아니요 | 예 | 
| 관계형 데이터베이스에서 복사 | 아니오 | 아니요 | 아니요 | 아니요 | 예 | 
| Blob 저장소로 복사 | 예 | 예 | 예 | 아니오 | 예 | 
| Blob 저장소에서 복사 | 예 | 예 | 예 | 예 | 예 |
| Data Lake Store로 복사 | 아니오 | 아니요 | 예 | 예 |  예 | 
| Data Lake Store에서 복사 | 아니오 | 아니요 | 예 | 예 | 예 | 


[1] AdlCopy는 Data Lake Analytics 계정과 함께 사용할 경우 빅 데이터를 전송하는 데 최적화되어 있습니다.

[2] PolyBase는 Hadoop에 계산을 푸시하고 [PolyBase 스케일 아웃 그룹](/sql/relational-databases/polybase/polybase-scale-out-groups)을 사용하여 SQL Server 인스턴스와 Hadoop 노드 간에 병렬 데이터 전송을 허용함으로써 [성능을 늘릴 수 있습니다](/sql/relational-databases/polybase/polybase-guide#performance).

### <a name="graphical-interface-and-azure-data-factory"></a>그래픽 인터페이스 및 Azure Data Factory

| | Azure Storage 탐색기 | Azure Portal * | Azure 데이터 팩터리 |
| --- | --- | --- | --- |
| 빅 데이터에 최적화 | 아니오 | 아니요 | 예 | 
| 관계형 데이터베이스로 복사 | 아니오 | 아니요 | 예 |
| 관계형 데이터베이스로 복사 | 아니오 | 아니요 | 예 |
| Blob 저장소로 복사 | 예 | 아니오 | 예 |
| Blob 저장소에서 복사 | 예 | 아니오 | 예 |
| Data Lake Store로 복사 | 아니오 | 아니요 | 예 |
| Data Lake Store에서 복사 | 아니오 | 아니요 | 예 |
| Blob 저장소로 업로드 | 예 | 예 | 예 |
| Data Lake Store로 업로드 | 예 | 예 | 예 |
| 데이터 전송 조정 | 아니오 | 아니요 | 예 |
| 사용자 지정 데이터 변환 | 아니오 | 아니요 | 예 |
| 가격 책정 모델 | 무료 | 무료 | 사용당 지급 |

\* 이 경우의 Azure Portal은 Blob 저장소 및 Data Lake Store에 대해 웹 기반 탐색 도구를 사용하는 것을 의미합니다.

