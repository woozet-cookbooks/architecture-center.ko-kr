---
title: Cognitive Services 기술 선택
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 055769188fbd6742b94094ee18766293812849fa
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-microsoft-cognitive-services-technology"></a>Microsoft Cognitive Services 기술 선택

Microsoft Cognitive Services는 AI(인공 지능) 응용 프로그램 및 데이터 흐름에 사용할 수 있는 클라우드 기반 API입니다. 이 기능은 응용 프로그램에서 바로 사용할 수 있는 미리 학습된 모델을 제공하므로, 사용자 입장에서 데이터 및 모델 학습이 없어도 됩니다. Cognitive Services는 Microsoft의 AI 및 연구 팀에서 개발했으며 최신 심층 학습 알고리즘을 활용합니다. 이 서비스는 HTTP REST 인터페이스를 통해 사용됩니다. 또한 많은 일반적인 응용 프로그램 개발 프레임워크에 SDK를 사용할 수 있습니다.

Cognitive Services에는 다음이 포함됩니다.

* 텍스트 분석
* Computer Vision
* 동영상 분석
* 음성 인식 및 생성
* 자연어 인식
* 지능형 검색

주요 이점:

* 최신 AI 서비스에 대한 개발 노력을 최소화할 수 있습니다.
* HTTP REST 인터페이스를 통해 앱에 쉽게 통합됩니다.
* Azure Data Lake Analytics에서 Cognitive Services를 사용할 수 있도록 기본적으로 지원됩니다.

고려 사항:

* 웹을 통해서만 사용할 수 있습니다. 일반적으로 인터넷 연결이 필요합니다. 예외는 장치 및 IoT Edge에서 예측을 위해 학습된 모델을 내보낼 수 있는 Custom Vision Service입니다.
* 풍부한 사용자 지정이 지원되지만 사용 가능한 서비스가 모든 예측 분석 요구 사항에 부합되지 않을 수도 있습니다.

## <a name="what-are-your-options-when-choosing-amongst-the-cognitive-services"></a>Cognitive Services 중에서 선택할 때 어떤 옵션이 있나요?
Azure에는 수십 가지의 Cognitive Services가 제공됩니다. 이러한 서비스 목록은 다음과 같이 지원되는 기능 영역별로 범주화된 디렉터리로 제공됩니다.
- [Vision](https://azure.microsoft.com/services/cognitive-services/directory/vision/)
- [음성](https://azure.microsoft.com/services/cognitive-services/directory/speech/)
- [Knowledge](https://azure.microsoft.com/services/cognitive-services/directory/know/)
- [이를 통해 검색](https://azure.microsoft.com/services/cognitive-services/directory/search/)
- [언어](https://azure.microsoft.com/services/cognitive-services/directory/lang/)

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 어떤 유형의 데이터를 다루나요? 사용하는 입력 데이터 유형에 따라 옵션 수를 줄입니다. 예를 들어, 텍스트를 입력하는 경우 입력 형식이 텍스트인 서비스 중에서 선택합니다. 

- 데이터가 모델을 학습하도록 할 것인가요? 그렇다면 정확성 및 성능 향상을 위해, 제공하는 데이터로 기본 모델을 학습할 수 있는 사용자 지정 서비스를 고려합니다. 

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서 주요 기능 차이점을 요약해서 보여 줍니다. 

### <a name="uses-prebuilt-models"></a>미리 작성된 모델 사용

|                                                   |             입력 형식              |                                                                                주요 이점                                                                                |
|---------------------------------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                Text Analytics API                 |                텍스트                 |                                                       의미 및 주제를 평가하여 사용자가 무엇을 원하는지 파악합니다.                                                        |
|                엔터티 연결 API                 |                텍스트                 |                                               명명된 엔터티 인식 및 명확성을 통해 앱 데이터 링크 기능을 강화합니다.                                               |
| LUIS(언어 인식 인텔리전트 서비스) |                텍스트                 |                                                          앱이 사용자의 명령을 인식하도록 학습합니다.                                                          |
|                 QnA Maker Service                 |                텍스트                 |                                             FAQ 형식 정보를 탐색하기 쉬운 대화형 답변으로 추출합니다.                                              |
|              Linguistic Analysis API              |                텍스트                 |                                                            복잡한 언어 개념을 간소화하고 텍스트를 구문 분석합니다.                                                             |
|           지식 탐색 서비스           |                텍스트                 |                                          자연어 입력을 통해 구조적 데이터에 대한 대화형 검색 환경을 구현합니다.                                          |
|              Web Language Model API               |                텍스트                 |                                                         웹 규모 데이터에서 학습한 예측 언어 모델을 활용합니다.                                                         |
|              Academic Knowledge API               |                텍스트                 |                                        Bing을 통해 채워진 Microsoft Academic Graph의 다양한 교육 콘텐츠를 활용합니다.                                         |
|               Bing Autosuggest API                |                텍스트                 |                                                        앱에 검색에 대한 지능형 자동 제안 옵션을 제공합니다.                                                        |
|               Bing Spell Check API                |                텍스트                 |                                                             앱에서 맞춤법 오류를 감지 및 수정합니다.                                                             |
|                Translator Text API                |                텍스트                 |                                                                           기계 번역입니다.                                                                            |
|                Recommendations API                |                텍스트                 |                                                             고객이 원하는 품목을 예측 및 추천합니다.                                                              |
|              Bing Entity Search API               |       텍스트(웹 검색 쿼리)       |                                                           웹에서 엔터티 정보를 식별하고 보강합니다.                                                           |
|               Bing Image Search API               |       텍스트(웹 검색 쿼리)       |                                                                            이미지를 검색합니다.                                                                             |
|               Bing News Search API                |       텍스트(웹 검색 쿼리)       |                                                                             뉴스를 검색합니다.                                                                              |
|               Bing Video Search API               |       텍스트(웹 검색 쿼리)       |                                                                            비디오를 검색합니다.                                                                             |
|                Bing Web Search API                |       텍스트(웹 검색 쿼리)       |                                                        수십억 개의 웹 문서에서 향상된 검색 세부 정보를 얻습니다.                                                        |
|                  Bing Speech API                  |           텍스트 또는 음성            |                                                                  음성을 텍스트로 변환하고 다시 음성으로 변환합니다.                                                                   |
|              화자 인식 API              |               음성                |                                                       음성을 사용하여 개별 화자를 식별하고 인증합니다.                                                        |
|               Translator Speech API               |               음성                |                                                                   실시간 음성 번역을 수행합니다.                                                                   |
|                Computer Vision API                |    이미지(또는 비디오의 프레임)    | 이미지에서 실행 가능한 정보를 추출하고, 사진 설명을 자동으로 만들고, 태그를 파생하고, 연예인을 인식하고, 텍스트를 추출하고, 정확한 축소판 그림을 만듭니다. |
|                 Content Moderator                 |        텍스트, 이미지 또는 비디오        |                                                               자동화된 이미지, 텍스트 및 비디오 조정                                                                |
|                    Emotion API                    | 이미지(인간 피사체가 있는 사진) |                                                              인간 피사체의 다양한 감정을 식별합니다.                                                               |
|                     Face API                      | 이미지(인간 피사체가 있는 사진) |                                                       사진에서 얼굴을 감지, 식별, 분석, 구성하고 태그를 지정합니다.                                                       |
|                   비디오 인덱서                   |                비디오                |                        감정과 같은 정보를 비디오로 나타내고, 음성을 기록하고, 음성을 번역하고, 안면 및 감정을 인식하고, 키워드를 추출합니다.                         |

### <a name="trained-with-custom-data-you-provide"></a>제공한 사용자 지정 데이터로 학습

| | 입력 형식 | 주요 이점 |
| --- | --- | --- |
| 사용자 지정 시각 서비스 | 이미지(또는 비디오의 프레임) | 나만의 Computer Vision 모델을 사용자 지정합니다. |
| Custom Speech Service | 음성 | 말하기 스타일, 배경 소음 및 어휘와 같은 음성 인식 장벽을 극복합니다. | 
| 사용자 지정 의사 결정 서비스 | 웹 콘텐츠(예: RSS 피드) | Machine Learning을 사용하여 홈페이지에 적합한 콘텐츠를 자동으로 선택합니다. |
| Bing Custom Search API | 텍스트(웹 검색 쿼리) | 커머셜급 검색 도구입니다. |

