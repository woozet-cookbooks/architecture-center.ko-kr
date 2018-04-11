---
title: Machine Learning 기술 선택
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 995349c795066ec3067b20ad2615e40b0fb152db
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a>Azure에서 Machine Learning 기술 선택

데이터 과학 및 Machine Learning은 일반적으로 데이터 과학자가 수행하는 작업입니다. 여기에는 데이터 과학자가 수행해야 하는 대화형 데이터 탐색 및 모델링 작업 유형에 맞게 특수하게 고안된 많은 전문가 도구가 필요합니다.

Machine Learning 솔루션은 반복적으로 구축되며, 다음과 같은 두 가지 고유한 단계로 구성됩니다.
* 데이터 준비 및 모델링
* 예측 서비스의 배포 및 소비

## <a name="tools-and-services-for-data-preparation-and-modeling"></a>데이터 준비 및 모델링을 위한 도구 및 서비스
데이터 과학자는 일반적으로 Python 또는 R로 작성한 사용자 지정 코드를 사용하여 데이터 작업을 수행하려고 합니다. 이 코드는 일반적으로 대화형으로 실행되며, 데이터 과학자는 이를 사용해서 데이터를 쿼리 및 탐색하고, 시각화 및 통계를 생성하여 관계 파악에 도움을 줍니다. 데이터 과학자가 사용할 수 있는 R 및 Python용 대화형 환경은 다양합니다. 데이터 과학자가 R 또는 Python 코드와 Markdown 텍스트를 포함하는 *노트* 파일을 만들 수 있도록 하는 브라우저 기반 셸을 제공하는 **Jupyter 노트**가 특히 선호됩니다. 이 기능은 코드 및 결과를 공유하고 단일 문서로 정리하여 공동 작업을 수행할 수 있는 효과적인 방법입니다.

일반적으로 사용되는 다른 도구는 다음과 같습니다.
* **Spyder**: Anaconda Python 배포판과 함께 제공되는 Python용 IDE(대화형 개발 환경)입니다.
* **R Studio**: R 프로그래밍 언어용 IDE입니다.
* **Visual Studio Code**: Machine Learning 및 AI 개발을 위해 일반적으로 사용되는 프레임워크일 뿐만 아니라 Python을 지원하는 경량, 플랫폼 간 코딩 환경입니다.

이러한 도구 외에도, 데이터 과학자는 코드 및 모델 관리를 간소화하기 위해 Azure 서비스를 활용할 수 있습니다.

### <a name="azure-notebooks"></a>Azure Notebooks
Azure 노트는 데이터 과학자가 클라우드 기반 라이브러리에서 Jupyter 노트를 만들고, 실행하고., 공유할 수 있도록 하는 온라인 Jupyter 노트 서비스입니다.

주요 이점:

* 무료 서비스&mdash;Azure 구독이 필요하지 않습니다.
* Jupyter 및 지원하는 R 또는 Python 배포를 로컬로 설치할 필요가 없음&mdash;브라우저만 사용하면 됩니다.
* 자체 온라인 라이브러리를 관리하고 모든 장치에서 액세스할 수 있습니다.
* 공동 작업자와 노트를 공유합니다.

고려 사항:

* 오프라인 상태에서는 노트에 액세스할 수 없습니다.
* 무료 노트 서비스의 제한된 처리 기능만으로 크거나 복잡한 모델을 학습하기 어려울 수 있습니다.

### <a name="data-science-virtual-machine"></a>데이터 과학 Virtual Machine
데이터 과학 Virtual Machine은 R, Python, Jupyter 노트, Visual Studio Code 및 Machine Learning 모델용 라이브러리(예: Microsoft Cognitive 도구 키트)를 포함하여 데이터 과학자가 일반적으로 사용하는 도구 및 프레임워크를 포함하는 Azure Virtual Machine 이미지입니다. 이러한 도구는 설치하는 데 복잡하고 시간이 많이 걸리며, 종종 버전 관리 문제를 야기하는 많은 상호 종속성을 포함할 수 있습니다. 미리 설치된 이미지를 사용하면 데이터 과학자가 환경 문제를 해결하는 데 소요되는 시간을 줄일 수 있으며, 과학자가 수행해야 하는 데이터 탐색 및 모델링 태스크에 좀 더 집중하도록 할 수 있습니다.

주요 이점:
* 데이터 과학 도구 및 프레임워크를 설치 및 관리하고 문제를 해결하는 데 드는 시간을 단축할 수 있습니다.
* 가장 일반적으로 사용되는 도구 및 프레임워크의 최신 버전이 포함되어 있습니다.
* 가상 머신 옵션에는 집약적 데이터 모델링을 위한 GPU 기능이 있는 확장성 높은 이미지가 포함되어 있습니다.

고려 사항:
* 오프라인 상태에서는 가상 머신에 액세스할 수 없습니다.
* 가상 머신을 실행하면 Azure 요금이 발생하므로, 필요할 때만 실행하도록 주의해야 합니다.

### <a name="azure-machine-learning"></a>Azure Machine Learning

Azure Machine Learning은 Machine Learning 실험 및 모델을 관리하기 위한 클라우드 기반 서비스입니다. 여기에는 데이터 준비 및 모델링 학습 스크립트를 추적하는 실험 서비스가 포함되며, 반복 간에 모델 성능을 비교할 수 있도록 모든 실행 기록이 유지 관리됩니다. Azure Machine Learning Workbench라는 플랫폼 간 클라이언트 도구는 데이터 과학자가 Jupyter 노트 또는 Visual Studio Code 같은 선택한 도구에서 스크립트를 만들 수 있도록 하면서, 스크립트 관리 및 기록을 위한 중앙 인터페이스를 제공합니다.

Azure Machine Learning Workbench에서 대화형 데이터 준비 도구를 사용하여 일반적인 데이터 변환 작업을 단순화하고, 확장 가능한 Docker 컨테이너 또는 Spark에서 모델 학습 스크립트를 로컬로 실행하도록 스크립트 실행 환경을 구성할 수 있습니다.

모델을 배포할 준비가 되면, Workbench 환경을 사용하여 모델을 패키지하고 Docker 컨테이너, Azure HDinsight의 Spark, Microsoft Machine Learning Server 또는 SQL Server에 웹 서비스로 배포합니다. 그런 다음 Azure Machine Learning 모델 관리 서비스를 사용하여 클라우드, 에지 장치 또는 엔터프라이즈 전체에서 모델 배포를 추적하고 관리할 수 있습니다.

주요 이점:

* 스크립트 및 실행 기록이 중앙에서 관리되므로 모델 버전을 쉽게 비교할 수 있습니다.
* 시각적 편집기를 통해 대화형 데이터를 변환합니다.
* 클라우드 또는 에지 장치로 모델을 쉽게 배포 및 관리합니다.

고려 사항:
* 모델 관리 모델과 워크벤치 도구 환경을 어느 정도 숙지합니다.

### <a name="azure-batch-ai"></a>Azure Batch AI

Azure Batch AI를 사용하면 Machine Learning 실험을 동시에 실행하고, GPU를 사용하여 가상 머신 클러스터 간에 대규모 모델 학습을 수행할 수 있습니다. 일괄 처리 AI 교육을 사용하면 Cognitive 도구 키트, Caffe, Chainer 및 TensorFlow와 같은 프레임워크를 사용하여 클러스터링된 GPU에서 심층 교육 작업을 스케일 아웃할 수 있습니다. 

Azure Machine Learning 모델 관리를 일괄 처리 AI 교육에서 모델을 생성하여 배포, 관리 및 모니터링하는 데 사용할 수 있습니다. 

### <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

Azure Machine Learning Studio는 데이터 실험을 만들고, Machine Learning 모델을 학습하고, Azure에 웹 서비스로 게시하기 위한 클라우드 기반, 시각적 개발 환경입니다. 시각적 끌어서 놓기 인터페이스를 사용하여 데이터 과학자 및 파워 유저는 Machine Learning 솔루션을 빠르게 만들 수 있으며, 사용자 지정 R 및 Python 논리, Machine Learning 모델링 태스크에 대해 설정된 다양한 통계 알고리즘 및 Jupyter 노트에 대한 기본 제공 지원도 활용할 수 있습니다.

주요 이점:

* 대화형 시각적 인터페이스를 사용하면 최소한의 코드로 Machine Learning 모델링을 수행할 수 있습니다.
* 데이터 탐색을 위한 기본 제공 Jupyter 노트
* 학습된 모델을 Azure 웹 서비스로서 직접 배포

고려 사항:

* 제한된 확장성. 학습 데이터 집합의 최대 크기는 10GB입니다.
* 온라인 전용 오프라인 개발 환경은 없습니다.

## <a name="tools-and-services-for-deploying-machine-learning-models"></a>Machine Learning 모델을 배포하기 위한 도구 및 서비스

데이터 과학자가 기계 학습 모델을 만들고 나면 사용자는 배포한 후 응용 프로그램 또는 다른 데이터 흐름에 사용해야 합니다. Machine Learning 모델에 대한 잠재적인 배포 대상은 많이 있습니다.

### <a name="spark-on-azure-hdinsight"></a>Azure HDInsight의 Spark

Apache Spark에는 Machine Learning 모델을 위한 Spark MLlib, 프레임워크 및 라이브러리가 포함되어 있습니다. MMLSpark(Spark용 Microsoft Machine Learning 라이브러리)에서도 Spark의 예측 모델에 대해 심화 학습 알고리즘 지원을 제공합니다.

주요 이점:

* Spark는 대용량 Machine Learning 프로세스에 대해 높은 확장성을 제공하는 분산 플랫폼입니다.
* Azure Machine Learning Workbench에서 HDinsight의 Spark로 직접 모델을 배포하고, Azure Machine Learning 모델 관리 서비스를 사용하여 관리할 수 있습니다.

고려 사항:

* Spark는 실행하는 전체 시간에 요금이 부과되는 HDinsight 클러스터에서 실행됩니다. Machine Learning 서비스를 가끔씩만 사용하는 경우에는 이것이 불필요한 비용일 수 있습니다.

### <a name="web-service-in-a-container"></a>컨테이너의 웹 서비스

Docker 컨테이너에서 Python 웹 서비스로 Machine Learning 모델을 배포할 수 있습니다. Azure 또는 에지 장치에 모델을 배포할 수 있습니다. 이 경우 모델이 작동하는 데이터와 함께 로컬로 데이터를 사용할 수 있습니다.

주요 이점:

* 컨테이너는 서비스를 패키지 및 배포하는 간단하면서 비용 효율적인 방법입니다.
* 에지 장치에 배포하는 기능을 사용하면 예측 논리를 데이터에 좀 더 가까운 위치로 이동할 수 있습니다.
* Azure Machine Learning Workbench에서 직접 컨테이너에 배포할 수 있습니다.

고려 사항:

* 이 배포 모델은 Docker 컨테이너를 기준으로 하므로, 이러한 방식으로 웹 서비스를 배포하기 전에 이 기술을 숙지하는 것이 좋습니다.

### <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning 서버

Machine Learning Server(이전의 Microsoft R Server)는 Machine Learning 시나리오에 맞게 특수하게 디자인된 R 및 Python 코드를 위한 확장 가능한 플랫폼입니다.

주요 이점:

* 높은 확장성.
* Azure Machine Learning Workbench에서 직접 배포

고려 사항:

* 엔터프라이즈에서 Machine Learning Server를 배포하고 관리해야 합니다.

### <a name="microsoft-sql-server"></a>Microsoft SQL Server에 대한 연결 문자열

Microsoft SQL Server는 기본적으로 R 및 Python을 지원하므로, 이러한 언어로 작성된 Machine Learning 모델을 데이터베이스에 Transact-SQL 함수로 캡슐화할 수 있습니다.

주요 이점:

* 예측 논리를 데이터베이스 함수에 캡슐화하여 데이터 계층 논리에 쉽게 포함할 수 있습니다.

고려 사항:

* SQL Server Database를 응용 프로그램에 대한 데이터 계층으로 간주합니다.

### <a name="azure-machine-learning-web-service"></a>Azure Machine Learning 웹 서비스

Azure Machine Learning Studio를 사용하여 Machine Learning 모델을 만들 경우 해당 모델을 웹 서비스로서 배포할 수 있습니다. 그런 후 HTTP에서 통신 가능한 모든 클라이언트 응용 프로그램에서 REST 인터페이스를 통해 이 모델을 사용할 수 있습니다.

주요 이점:

* 쉬운 개발 및 배포
* 기본 모니터링 메트릭을 사용하는 웹 서비스 관리 포털
* Azure Data Lake Analytics, Azure Data Factory 및 Azure Stream Analytics에서 Azure Machine Learning 웹 서비스를 호출하기 위한 지원이 기본적으로 제공

고려 사항:

* Azure Machine Learning Studio를 사용하여 작성된 모델에만 사용할 수 있습니다.
* 웹 기반 액세스 전용의 학습된 모델은 온-프레미스 또는 오프라인에서 실행할 수 없습니다.

