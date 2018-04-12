---
title: 검색 데이터 저장소 선택
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: ead07e307e96696faa5ddf48505eee378027523c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-search-data-store-in-azure"></a>Azure에서 검색 데이터 저장소 선택

이 문서에서는 Azure에서 검색 데이터 저장소에 대한 기술 선택 옵션을 비교해서 설명합니다. 검색 데이터 저장소는 자유 형식 텍스트에 대한 검색을 수행하기 위한 특수한 인덱스를 만들고 저장하는 데 사용됩니다. 인덱싱되는 텍스트는 Blob 저장소와 같은 별도 데이터 저장소에 있을 수 있습니다. 응용 프로그램은 검색 데이터 저장소로 쿼리를 제출하며, 일치하는 문서 목록이 검색됩니다. 이 시나리오에 대한 자세한 내용은 [검색을 위해 자유 형식 텍스트 처리](../scenarios/search.md)를 참조하세요. 

## <a name="what-are-your-options-when-choosing-a-search-data-store"></a>검색 데이터 저장소를 선택할 때의 옵션은 무엇인가요?
Azure에서 다음의 모든 데이터 저장소는 검색 인덱스를 제공하여 자유 형식 텍스트 데이터에 대한 핵심 검색 요구 사항을 충족합니다.
- [Azure Search](/azure/search/search-what-is-azure-search)
- [Elasticsearch](https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch?tab=Overview)
- [Solr을 포함하는 HDInsight](/azure/hdinsight/hdinsight-hadoop-solr-install-linux)
- [전체 텍스트 검색을 사용하는 Azure SQL Database](/sql/relational-databases/search/full-text-search)


## <a name="key-selection-criteria"></a>주요 선택 조건

검색 시나리오에 대해 먼저 다음 질문에 응답하여 사용자 요구에 적합한 검색 데이터 저장소를 선택합니다.

- 사용자 고유의 서버를 관리하지 않고 관리되는 서비스를 원하시나요?

- 디자인 타임에 인덱스 스키마를 지정할 수 있나요? 그렇지 않으면 업데이트 가능 스키마를 지원하는 옵션을 선택합니다.

- 전체 텍스트 검색용 인덱스만 필요한가요 아니면 숫자 데이터의 빠른 집계 및 기타 분석도 필요한가요? 전체 텍스트 검색 이상의 기능이 필요한 경우 추가 분석을 지원하는 옵션을 고려하세요.

- 인덱싱된 데이터에 대한 로그 수집, 집계 및 시각화가 지원되는 로그 분석을 위한 검색 인덱스가 필요한가요? 그렇다면 로그 분석 스택의 일부인 Elasticsearch를 고려하세요.

- PDF, Word, PowerPoint 및 Excel 등의 일반적인 문서 형식으로 데이터를 인덱싱해야 하나요? 그렇다면 문서 인덱서를 제공하는 옵션을 선택합니다.

- 데이터베이스에 특정 보안 요구가 있나요? 그렇다면 아래에 나열된 보안 기능을 고려합니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.

### <a name="general-capabilities"></a>일반 기능

| | Azure Search | Elasticsearch | Solr을 포함하는 HDInsight | SQL Database | 
| --- | --- | --- | --- | --- | 
| 관리되는 서비스인지 여부 | 예 | 아니오 | 예 | 예 |  
| REST API | 예 | 예 | 예 | 아니요 |
| 프로그래밍 기능 | .NET | 자바 | 자바 | T-SQL | 
| 일반적인 파일 형식(PDF, DOCX, TXT 등)에 대한 문서 인덱서 | 예 | 아니오 | 예 | 아니오 |

### <a name="manageability-capabilities"></a>관리 효율성

| | Azure Search | Elasticsearch | Solr을 포함하는 HDInsight | SQL Database | 
| --- | --- | --- | --- | --- |
| 업데이트 가능 스키마 | 아니요 | 예 | 예 | 예 |
| 확장 지원  | 예 | 예 | 예 | 아니오 |

### <a name="analytic-workload-capabilities"></a>분석 워크로드 기능

| | Azure Search | Elasticsearch | Solr을 포함하는 HDInsight | SQL Databash | 
| --- | --- | --- | --- | --- | 
| 전체 텍스트 검색을 능가하는 분석 기능 지원 | 아니요 | 예 | 예 | 예 |
| 로그 분석 스택에 포함되는지 여부 | 아니오 | 예(ELK) |  아니오 | 아니오 |
| 의미 체계 검색 지원 | 예(유사 문서만 찾기) | 예 | 예 | 예 | 

### <a name="security-capabilities"></a>보안 기능

| | Azure Search | Elasticsearch | Solr을 포함하는 HDInsight | SQL Databash | 
| --- | --- | --- | --- | --- | 
| 행 수준 보안 | 부분적(그룹 ID별로 필터링하기 위한 응용 프로그램 쿼리 필요) | 부분적(그룹 ID별로 필터링하기 위한 응용 프로그램 쿼리 필요) | 예 | 예 | 
| 투명한 데이터 암호화 | 아니오 | 아니요 | 아니요 | 예 |  
| 특정 IP 주소로 액세스 제한 | 아니오 | 예 | 예 | 예 |   
| 가상 네트워크 액세스만 허용하도록 액세스 제한 | 아니요 | 예 | 예 | 예 |  
| Active Directory 인증(통합 인증) | 아니오 | 아니요 | 아니요 | 예 | 

## <a name="see-also"></a>참고 항목

[검색을 위해 자유 형식 텍스트 처리](../scenarios/search.md)