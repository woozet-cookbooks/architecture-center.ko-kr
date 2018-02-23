---
title: "데이터 분석 및 보고 기술 선택"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 830c61bba64a6971c815330887e5cdcc4f2b5f56
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-data-analytics-technology-in-azure"></a>Azure에서 데이터 분석 기술 선택

대부분의 빅 데이터 솔루션의 목표는 분석 및 보고를 통해 데이터에 대한 정보를 제공하는 것입니다. 여기에는 미리 구성된 보고서 및 시각화나 대화형 데이터 탐색이 포함될 수 있습니다. 

## <a name="what-are-your-options-when-choosing-a-data-analytics-technology"></a>데이터 분석 기술을 선택할 때 사용할 수 있는 옵션은 무엇인가요?

Azure에서는 사용자의 요구에 따라 분석, 시각화 및 보고에 대한 여러 옵션을 사용할 수 있습니다.

- [Power BI](/power-bi/)
- [Jupyter 노트북](https://jupyter.readthedocs.io/en/latest/index.html)
- [Zeppelin 노트](https://zeppelin.apache.org/)
- [Microsoft Azure 노트](https://notebooks.azure.com/)

### <a name="power-bi"></a>Power BI

[Power BI](/power-bi/)는 비즈니스 분석 도구 제품군입니다. 이 제품군은 수백 개의 데이터 원본에 연결될 수 있으며 임시 분석에 사용될 수 있습니다. 현재 사용 가능한 데이터 원본에 대해서는 [이 목록](/power-bi/desktop-data-sources)을 참조하세요. [Power BI Embedded](https://azure.microsoft.com/services/power-bi-embedded/)를 사용하여 추가 라이선스 없이도 응용 프로그램 내에 Power BI를 통합할 수 있습니다.

조직에서는 Power BI를 사용하여 보고서를 생성하고 조직에 게시할 수 있습니다. 모든 사용자는 거버넌스 및 [기본 제공된 보안](/power-bi/service-admin-power-bi-security)을 사용하여 개인별 대시보드를 만들 수 있습니다. Power BI는 [Azure AD](/azure/active-directory/)(Azure Active Directory)를 사용하여 Power BI 서비스에 로그인하는 사용자를 인증하고, 사용자가 인증을 요구하는 리소스에 액세스하려고 할 때마다 Power BI 로그인 자격 증명을 사용합니다.

### <a name="jupyter-notebooks"></a>Jupyter 노트북 

[Jupyter 노트](https://jupyter.readthedocs.io/en/latest/index.html)는 데이터 과학자가 Python, Scala 또는 R 코드와 Markdown 텍스트를 포함하는 *노트* 파일을 만들어 코드 및 결과를 공유하고 단일 문서로 문서화함으로써 효과적으로 공동 작업하는 방법을 제공할 수 있도록 하는 브라우저 기반 셸을 제공합니다.

Spark 및 Hadoop과 같은 HDInsight 클러스터의 변형 대부분은 데이터로 상호 작용하고 처리를 위해 작업을 제출하기 위한 [Jupyter 노트로 미리 구성](/azure/hdinsight/spark/apache-spark-jupyter-notebook-kernels)되어 있습니다. 사용 하는 HDInsight 클러스터의 유형에 따라, 코드 해석 및 실행을 위해 하나 이상의 커널이 제공됩니다. 예를 들어, HDInsight의 Spark 클러스터는 Spark 엔진을 사용하여 Python 또는 Scala 코드를 실행하기 위해 선택할 수 있는 Spark 관련 커널을 제공합니다.

Jupyter 노트는 Power BI와 같은 BI/보고 도구로 보다 수준 높은 시각화를 구축하기 전에 데이터를 분석, 시각화 및 처리하기 위한 훌륭한 환경을 제공합니다.

### <a name="zeppelin-notebooks"></a>Zeppelin 노트

[Zeppelin 노트](https://zeppelin.apache.org/)는 기능면에서 Jupyter와 유사한 브라우저 기반 셸을 위한 또 다른 옵션입니다. 일부 HDInsight 클러스터는 [Zeppelin 노트로 미리 구성](/azure/hdinsight/spark/apache-spark-zeppelin-notebook)되어 있습니다. 그러나 현재, [HDInsight 대화형 쿼리](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)(Hive LLAP) 클러스터를 사용하는 경우 대화형 Hive 쿼리를 실행하는 데 사용할 수 있는 유일한 노트는 [Zeppelin](/azure/hdinsight/hdinsight-connect-hive-zeppelin) 뿐입니다. 또한 [도메인에 가입된 HDInsight 클러스터](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)를 사용하는 경우 노트 및 기본 Hive 테이블에 대한 액세스를 제어하기 위해 다른 사용자 로그인을 할당할 수 있는 유일한 유형이 Zeppelin 노트입니다.

### <a name="microsoft-azure-notebooks"></a>Microsoft Azure 노트

[Azure 노트](https://notebooks.azure.com/)는 데이터 과학자가 클라우드 기반 라이브러리에서 Jupyter 노트를 만들고, 실행하고., 공유할 수 있도록 하는 온라인 Jupyter 노트 기반 서비스입니다. Azure 노트는 Python 2, 3 Python, F# 및 R의 실행 환경을 제공하고, ggplot, matplotlib, bokeh 및 seaborn과 같이 데이터를 시각화하기 위한 몇 가지 차트 라이브러리를 제공합니다.

HDInsight 클러스터에서 실행되며 클러스터의 기본 저장소 계정에 연결되는 Jupyter 노트와 달리, Azure 노트는 데이터를 제공하지 않습니다. 온라인 원본에서 데이터 다운로드, Azure Blob 또는 Table Storage와 상호 작용, SQL Database에 연결 또는 Azure Data Factory의 복사 마법사를 사용하여 데이터 로드 등과 같은 다양한 방식으로 [데이터를 로드](https://notebooks.azure.com/Microsoft/libraries/samples/html/Getting%20to%20your%20Data%20in%20Azure%20Notebooks.ipynb)해야 합니다.

주요 이점:

* 무료 서비스&mdash;Azure 구독이 필요하지 않습니다.
* Jupyter 및 지원하는 R 또는 Python 배포를 로컬로 설치할 필요가 없음&mdash;브라우저만 사용하면 됩니다.
* 자체 온라인 라이브러리를 관리하고 모든 장치에서 액세스할 수 있습니다.
* 공동 작업자와 노트를 공유합니다.

고려 사항:

* 오프라인 상태에서는 노트에 액세스할 수 없습니다.
* 무료 노트 서비스의 제한된 처리 기능만으로 크거나 복잡한 모델을 학습하기 어려울 수 있습니다.

## <a name="key-selection-criteria"></a>주요 선택 조건

선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.

- 다양한 데이터 원본에 연결하여 도메인 전체에 분산되어 있는 데이터에 대한 보고서를 만들 수 있는 중앙 위치를 제공해야 하나요? 그렇다면 수백 개의 데이터 원본에 연결할 수 있는 옵션을 선택합니다.

- 외부 웹 사이트 또는 응용 프로그램에 동적 시각화를 포함하려고 하나요? 그렇다면 포함 기능을 제공하는 옵션을 선택합니다.

- 오프라인 상태에서 시각화 및 보고서를 디자인하려고 하나요? 그렇다면 오프라인 기능이 있는 옵션을 선택합니다.

- 크거나 복잡한 AI 모델을 학습하거나 매우 큰 데이터 집합으로 작업하기 위해 높은 처리 능력이 필요한가요? 그렇다면 빅 데이터 클러스터에 연결할 수 있는 옵션을 선택합니다.

## <a name="capability-matrix"></a>기능 매트릭스

다음 표에서 주요 기능 차이점을 요약해서 보여 줍니다. 

### <a name="general-capabilities"></a>일반 기능

| | Power BI | Jupyter 노트북 | Zeppelin 노트 | Microsoft Azure 노트 |
| --- | --- | --- | --- | --- |
| 고급 처리를 위해 빅 데이터 클러스터에 연결 | 예 | 예 | 예 | 아니요 |
| 관리되는 서비스 | 예 | 예 <sup>1</sup> | 예 <sup>1</sup> | 예 |
| 수백 개의 데이터 원본에 연결 | 예 | 아니오 | 아니요 | 아니요 |
| 오프라인 기능 | 예 <sup>2</sup> | 아니요 | 아니요 | 아니요 |
| 포함 기능 | 예 | 아니오 | 아니요 | 아니요 |
| 자동 데이터 새로 고침 | 예 | 아니요 | 아니요 | 아니오 |
| 다양한 오픈 소스 패키지에 액세스 | 아니요 | 예 <sup>3</sup> | 예 <sup>3</sup> | 예 <sup>4</sup> |
| 데이터 변환/정리 옵션 | [파워 쿼리](https://powerbi.microsoft.com/blog/getting-started-with-power-query-part-i/), R | 40개 언어(Python, R, Julia 및 Scala 포함) | 20개 이상의 인터프리터(Python, JDBC 및 R 포함) | Python, F#, R |
| 가격 | 무료 Power BI Desktop(제작)에 대해서는 호스팅 옵션에 대한 [가격 책정](https://powerbi.microsoft.com/pricing/)을 참조하세요. | 무료 | 무료 | 무료 |
| 다중 사용자 공동 작업 | [예](/power-bi/service-how-to-collaborate-distribute-dashboards-reports) | 예(공유를 통해 또는 [JupyterHub](https://github.com/jupyterhub/jupyterhub)와 같은 다중 사용자 서버를 사용하여) | 예 | 예(공유를 통해) |

[1] 관리되는 HDInsight 클러스터의 일부로 사용되는 경우

[2] Power BI Desktop을 사용하여

[2] [Maven 리포지토리](http://search.maven.org/)에서 커뮤니티 제공 패키지를 검색할 수 있습니다.

[3] Python 패키지를 pip 또는 conda를 사용하여 설치할 수 있습니다. R 패키지는 CRAN 또는 GitHub에서 설치할 수 있습니다. F#의 패키지는 [Paket 종속성 관리자](https://fsprojects.github.io/Paket/)를 사용하여 nuget.org를 통해 설치할 수 있습니다.

