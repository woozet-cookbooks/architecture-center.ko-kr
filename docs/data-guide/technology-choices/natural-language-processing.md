---
title: 자연어 처리 기술 선택
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: dacf7bf9cf3e9efed212f34da93c1470954965cf
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
ms.locfileid: "29288855"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a>Azure에서 자연어 처리 기술 선택

자유 형식 텍스트 처리는 일반적으로 검색을 지원하기 위해 텍스트 단락을 포함하는 문서에 대해 수행되지만, 감성 분석, 토픽 감지, 언어 감지, 핵심 구 추출 및 문서 분류와 같은 기타 NLP(자연어 처리) 작업을 수행하는 데도 사용됩니다. 이 문서에서는 NLP 작업을 지원하는 기술 선택 사항을 집중적으로 설명합니다.

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a>NLP 서비스를 선택할 때 사용할 수 있는 옵션은 무엇인가요?

Azure에서 다음 서비스는 NLP(자연어 처리) 기능을 제공합니다.

- [Azure HDInsight(Spark 및 Spark MLlib 포함)](/azure/hdinsight/spark/apache-spark-overview)
- [Microsoft Cognitive Services](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 미리 작성된 모델을 사용하려고 하나요? 그렇다면 Microsoft Cognitive Services에서 제공하는 API를 사용하는 것을 고려합니다.

- 대량의 텍스트 데이터에 대해 사용자 지정 모델을 학습해야 하나요? 그렇다면 Spark MLlib 및 Spark NLP와 함께 Azure HDInsight를 사용하는 것을 고려합니다.

- 토큰화, 형태소 분석, 기본형 찾기 및 TF/IDF(용어 빈도/역 문서 빈도) 같은 기본적인 NLP 기능이 필요한가요? 그렇다면 Spark MLlib 및 Spark NLP와 함께 Azure HDInsight를 사용하는 것을 고려합니다.

- 엔터티 및 의도 식별, 토픽 감지, 맞춤법 검사 또는 감성 분석 같은 간단한 고급 NLP 기능이 필요한가요? 그렇다면 Microsoft Cognitive Services에서 제공하는 API를 사용하는 것을 고려합니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.  

### <a name="general-capabilities"></a>일반 기능

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- |
| 미리 학습된 모델을 서비스로 제공합니다. | 아니요 | 예 |
| REST API | 예 | 예 |
| 프로그래밍 기능 | Python, Scala, Java | C#, Java, Node.js, Python, PHP, Ruby |
| 빅 데이터 집합 및 대형 문서의 처리를 지원합니다. | 예 | 아니오 |

### <a name="low-level-natural-language-processing-capabilities"></a>기본적인 자연어 처리 기능

| | Azure HDInsight | Microsoft Cognitive Services |  
| --- | --- | --- | 
| 토크나이저 | 예(Spark NLP) | 예(Linguistic Analysis API) |
| 형태소 분석기 | 예(Spark NLP) | 아니오 |
| 기본형 분석기 | 예(Spark NLP) | 아니요 |
| 품사 태그 지정 | 예(Spark NLP) | 예(Linguistic Analysis API) |
| TF/IDF(용어 빈도/역 문서 빈도) | 예(Spark MLlib) | 아니오 |
| 문자열 유사성&mdash;편집 거리 계산 | 예(Spark MLlib) | 아니오 |
| N-gram 계산 | 예(Spark MLlib) | 아니오 |
| 중지 단어 제거 | 예(Spark MLlib) | 아니오 |

### <a name="high-level-natural-language-processing-capabilities"></a>고급 자연어 처리 기능

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- | 
| 엔터티/의도 식별 및 추출 | 아니요 | 예(LUIS(언어 인식 인텔리전트 서비스) API) |    
| 토픽 검색 | 예(Spark NLP) | 예(Text Analytics API) |
| 맞춤법 검사 | 예(Spark NLP) | 예(Bing Spell Check API) |
| 정서 분석 | 예(Spark NLP) | 예(Text Analytics API) |
| 언어 검색 | 아니오 | 예(Text Analytics API) |
| 영어 이외의 다국어 지원 | 아니요 | 예(API에 따라 다름) |

## <a name="see-also"></a>참고 항목

[자연어 처리](../scenarios/natural-language-processing.md)