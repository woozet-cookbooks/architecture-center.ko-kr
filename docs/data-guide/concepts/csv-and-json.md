---
title: CSV 및 JSON 파일 처리
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 6b888ff230afefbd74249aa913e5bab66d47d7e2
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/31/2018
---
# <a name="working-with-csv-and-json-files-for-data-solutions"></a>데이터 솔루션용 CSV 및 JSON 파일 작업

CSV 및 JSON은 구조화되었거나 반구조화된 데이터의 수집, 교환 및 저장에 사용되는 가장 일반적인 형식일 것입니다. 

## <a name="about-csv-format"></a>CSV 형식 정보

CSV(쉼표로 구분된 값) 파일은 시스템 간에 일반 텍스트의 테이블 형식 데이터를 교환하는 데 일반적으로 사용됩니다. 일반적으로 데이터에 대한 열 이름을 제공하는 머리글 행이 포함되지만 반구조적인 것으로 간주됩니다. 이것은 기본적으로는 CSV를 계층형 또는 관계형 데이터로 나타낼 수 없기 때문입니다. 데이터 관계는 일반적으로 외래 키가 여러 파일의 열에 저장되되지만 해당 파일 간 관계는 형식 자체로 표시되지 않는 여러 CSV 파일로 처리됩니다. CSV 형식의 파일은 쉼표를 제외한 다른 구분 기호(예: 탭 또는 공백)를 사용할 수 있습니다.

이러한 제한에도 불구하고, CSV 파일은 광범위한 비즈니스, 소비자 및 과학 응용 프로그램에서 지원되므로 데이터 교환 방식으로 널리 사용됩니다. 예를 들어, 데이터베이스 및 스프레드시트 프로그램은 CSV 파일을 가져오고 내보낼 수 있습니다. 마찬가지로, 대부분 일괄 처리 및 스트림 데이터 처리 엔진(예: Spark 및 Hadoop)은 CSV 형식 파일을 고유하게 직렬화 및 역직렬화하도록 지원하며, 읽을 때 스키마를 적용하는 방법을 제공합니다. 이를 통해 데이터에 대해 쿼리를 수행하고 더 빠른 처리를 위해 보다 효율적인 데이터 형식으로 정보를 저장하는 옵션이 제공되므로 데이터를 보다 쉽게 사용할 수 있게 됩니다.

## <a name="about-json-format"></a>JSON 형식 정보

JSON(JavaScript Object Notation) 데이터는 반구조화된 형식의 키-값 쌍으로 표현됩니다. JSON은 종종 XML과 비교됩니다. 둘 다 하위 데이터가 상위 데이터와 인라인으로 표시되는 계층 구조 형식으로 데이터를 저장할 수 있기 때문입니다. 둘 다 자체 설명적이며 사람이 읽을 수 있지만, JSON 문서가 훨씬 더 작으므로 특히 REST 기반 웹 서비스가 출현하면서 온라인 데이터 교환에서 더 많이 사용되게 되었습니다. 

JSON 형식 파일은 CSV에 비해 다음과 같은 이점을 제공합니다.

* JSON은 계층 구조를 유지하므로, 관련 데이터를 단일 문서에 두고 복잡한 관계를 나타내기가 더 쉽습니다.
* 대부분의 프로그래밍 언어는 JSON을 개체로 역직렬화하는 기능을 기본적으로 지원하거나, 간단한 JSON 직렬화 라이브러리를 제공합니다.
* JSON은 개체 목록을 지원하여, 복잡한 방식으로 목록을 관계형 데이터 모델로 변환할 필요가 없습니다.
* JSON은 MongoDB, Couchbase 및 Azure Cosmos DB와 같은 NoSQL 데이터베이스에 자주 사용되는 파일 형식입니다.

네트워크를 통해 들어오는 많은 데이터가 이미 JSON 형식이므로, 대부분의 웹 기반 프로그래밍 언어는 원래부터 JSON 사용을 지원하거나, 외부 라이브러리를 사용해서 JSON 데이터를 직렬화 및 역직렬화하도록 지원합니다. JSON에 대한 이러한 범용 지원은 데이터 구조 표현, 핫 데이터에 대한 교환 형식 및 콜드 데이터에 대한 데이터 저장소를 통해 데이터를 논리적 형식으로 사용할 수 있도록 합니다.

많은 일괄 처리 및 스트림 데이터 처리 엔진은 기본적으로 JSON 직렬화 및 역직렬화를 지원합니다. JSON 문서에 포함된 데이터는 궁극적으로 Parquet 또는 Avro와 같은 보다 강력한 성능 최적화 형식으로 저장될 수 있지만, 데이터를 필요에 따라 다시 처리하는 데 중요한 신뢰할 수 있는 원본의 원시 데이터로 사용됩니다.

## <a name="when-to-use-csv-or-json-formats"></a>CSV 또는 JSON 형식을 사용하는 경우

CSV는 데이터의 내보내기 및 가져오기 또는 분석 및 기계 학습을 위한 처리에 좀 더 일반적으로 사용됩니다. JSON 형식 파일은 같은 이점을 가지지만, 핫 데이터 교환 솔루션에서 좀 더 일반적입니다. JSON 문서는 온라인 트랜잭션을 수행하는 웹 및 모바일 장치에서, 단방향 또는 양방향 통신을 위해 IoT(사물 인터넷) 장치에서 또는 SaaS 및 PaaS 서비스 또는 서버 없는 아키텍처와 통신하는 클라이언트 응용 프로그램에서 주로 전송합니다. 

CSV 및 JSON 파일 형식 둘 다, 서로 다른 시스템이나 장치 간에 데이터를 쉽게 교환할 수 있도록 합니다. 이러한 반구조화된 형식은 거의 모든 데이터 형식을 유연하게 전송하도록 하며, 이러한 형식이 범용으로 지원되므로 사용도 간편합니다. 둘 다 좀 더 효율적인 쿼리를 위해 처리된 데이터가 이진 형식으로 저장될 경우, 신뢰할 수 있는 원시 원본으로 사용될 수 있습니다. 

## <a name="working-with-csv-and-json-data-in-azure"></a>Azure에서 CSV 및 JSON 데이터 사용

Azure에서는 사용자의 요구에 따라, CSV 및 JSON 파일을 사용하기 위한 몇 가지 솔루션을 제공합니다. 이러한 파일이 기본적으로 저장되는 위치는 Azure Storage 또는 Azure Data Lake Store입니다. 이러한 파일 및 다른 텍스트 기반 파일을 사용하는 대부분의 Azure 서비스는 개체 저장소 서비스와 통합됩니다. 그러나 경우에 따라, 데이터를 Azure SQL 또는 기타 데이터 저장소로 직접 가져오도록 선택할 수 있습니다. SQL Server에서는 본래부터 JSON 문서를 저장하고 사용할 수 있도록 지원하므로 [해당 파일 형식을 쉽게 가져오고 처리](/sql/relational-databases/json/import-json-documents-into-sql-server)할 수 있습니다. SQL 대량 가져오기와 같은 유틸리티를 사용하여 [CSV 파일을 쉽게 가져올](/sql/relational-databases/json/import-json-documents-into-sql-server) 수 있습니다.

시나리오에 따라, 데이터의 [일괄 처리](../scenarios/batch-processing.md) 또는 [실시간 처리](../scenarios/real-time-processing.md)를 수행할 수 있습니다.

## <a name="challenges"></a>과제

이러한 형식을 사용할 때 다음과 같은 몇 가지 문제를 고려해야 합니다.

* 데이터 모델에 대한 제약이 없으면, CSV 및 JSON 파일의 데이터는 손상되기 쉽습니다("가비지 인, 가비지 아웃"). 예를 들어, 이러한 파일에는 날짜/시간 개체 개념이 없으므로, 파일 형식 때문에 날짜 필드에 "ABC123"를 삽입하지 못하는 경우는 없습니다.

* 콜드 저장소 솔루션이 크기가 조정되지 않으므로 빅 데이터로 작업하는 경우 CSV 및 JSON 파일을 사용합니다. 대부분의 경우 병렬 처리를 위해 파티션으로 분할할 수 없으며, 이진 형식처럼 압축할 수 없습니다. 이로 인해 이 데이터는 주로 Parquet 및 ORC(최적화된 행 칼럼 형식)과 같은 읽기 최적화 형식으로 처리 및 저장되며, 포함된 데이터에 대한 인덱스 및 인라인 통계도 제공됩니다.

* 반구조화된 데이터를 보다 쉽게 쿼리하고 분석할 수 있도록 스키마를 적용해야 할 수 있습니다. 일반적으로 이를 위해 작업 환경(예: 데이터베이스)의 데이터 저장소 요구에 부합되는 다른 형식으로 데이터를 저장해야 합니다.
