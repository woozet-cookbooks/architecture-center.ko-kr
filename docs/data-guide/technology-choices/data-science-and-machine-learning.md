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
# <a name="choosing-a-machine-learning-technology-in-azure"></a><span data-ttu-id="0f9ea-102">Azure에서 Machine Learning 기술 선택</span><span class="sxs-lookup"><span data-stu-id="0f9ea-102">Choosing a machine learning technology in Azure</span></span>

<span data-ttu-id="0f9ea-103">데이터 과학 및 Machine Learning은 일반적으로 데이터 과학자가 수행하는 작업입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-103">Data science and machine learning is a workload that is usually undertaken by data scientists.</span></span> <span data-ttu-id="0f9ea-104">여기에는 데이터 과학자가 수행해야 하는 대화형 데이터 탐색 및 모델링 작업 유형에 맞게 특수하게 고안된 많은 전문가 도구가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-104">It requires specialist tools, many of which are designed specifically for the type of interactive data exploration and modeling tasks that a data scientist must perform.</span></span>

<span data-ttu-id="0f9ea-105">Machine Learning 솔루션은 반복적으로 구축되며, 다음과 같은 두 가지 고유한 단계로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-105">Machine learning solutions are built iteratively, and have two distinct phases:</span></span>
* <span data-ttu-id="0f9ea-106">데이터 준비 및 모델링</span><span class="sxs-lookup"><span data-stu-id="0f9ea-106">Data preparation and modeling.</span></span>
* <span data-ttu-id="0f9ea-107">예측 서비스의 배포 및 소비</span><span class="sxs-lookup"><span data-stu-id="0f9ea-107">Deployment and consumption of predictive services.</span></span>

## <a name="tools-and-services-for-data-preparation-and-modeling"></a><span data-ttu-id="0f9ea-108">데이터 준비 및 모델링을 위한 도구 및 서비스</span><span class="sxs-lookup"><span data-stu-id="0f9ea-108">Tools and services for data preparation and modeling</span></span>
<span data-ttu-id="0f9ea-109">데이터 과학자는 일반적으로 Python 또는 R로 작성한 사용자 지정 코드를 사용하여 데이터 작업을 수행하려고 합니다. 이 코드는 일반적으로 대화형으로 실행되며, 데이터 과학자는 이를 사용해서 데이터를 쿼리 및 탐색하고, 시각화 및 통계를 생성하여 관계 파악에 도움을 줍니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-109">Data scientists typically prefer to work with data using custom code written in Python or R. This code is generally run interactively, with the data scientists using it to query and explore the data, generating visualizations and statistics to help determine the relationships with it.</span></span> <span data-ttu-id="0f9ea-110">데이터 과학자가 사용할 수 있는 R 및 Python용 대화형 환경은 다양합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-110">There are many interactive environments for R and Python that data scientists can use.</span></span> <span data-ttu-id="0f9ea-111">데이터 과학자가 R 또는 Python 코드와 Markdown 텍스트를 포함하는 *노트* 파일을 만들 수 있도록 하는 브라우저 기반 셸을 제공하는 **Jupyter 노트**가 특히 선호됩니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-111">A particular favorite is **Jupyter Notebooks** that provides a browser-based shell that enables data scientists to create *notebook* files that contain R or Python code and markdown text.</span></span> <span data-ttu-id="0f9ea-112">이 기능은 코드 및 결과를 공유하고 단일 문서로 정리하여 공동 작업을 수행할 수 있는 효과적인 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-112">This is an effective way to collaborate by sharing and documenting code and results in a single document.</span></span>

<span data-ttu-id="0f9ea-113">일반적으로 사용되는 다른 도구는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-113">Other commonly used tools include:</span></span>
* <span data-ttu-id="0f9ea-114">**Spyder**: Anaconda Python 배포판과 함께 제공되는 Python용 IDE(대화형 개발 환경)입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-114">**Spyder**: The interactive development environment (IDE) for Python provided with the Anaconda Python distribution.</span></span>
* <span data-ttu-id="0f9ea-115">**R Studio**: R 프로그래밍 언어용 IDE입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-115">**R Studio**: An IDE for the R programming language.</span></span>
* <span data-ttu-id="0f9ea-116">**Visual Studio Code**: Machine Learning 및 AI 개발을 위해 일반적으로 사용되는 프레임워크일 뿐만 아니라 Python을 지원하는 경량, 플랫폼 간 코딩 환경입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-116">**Visual Studio Code**: A lightweight, cross-platform coding environment that supports Python as well as commonly used frameworks for machine learning and AI development.</span></span>

<span data-ttu-id="0f9ea-117">이러한 도구 외에도, 데이터 과학자는 코드 및 모델 관리를 간소화하기 위해 Azure 서비스를 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-117">In addition to these tools, data scientists can leverage Azure services to simplify code and model management.</span></span>

### <a name="azure-notebooks"></a><span data-ttu-id="0f9ea-118">Azure Notebooks</span><span class="sxs-lookup"><span data-stu-id="0f9ea-118">Azure Notebooks</span></span>
<span data-ttu-id="0f9ea-119">Azure 노트는 데이터 과학자가 클라우드 기반 라이브러리에서 Jupyter 노트를 만들고, 실행하고., 공유할 수 있도록 하는 온라인 Jupyter 노트 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-119">Azure Notebooks is an online Jupyter Notebooks service that enables data scientists to create, run, and share Jupyter Notebooks in cloud-based libraries.</span></span>

<span data-ttu-id="0f9ea-120">주요 이점:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-120">Key benefits:</span></span>

* <span data-ttu-id="0f9ea-121">무료 서비스&mdash;Azure 구독이 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-121">Free service&mdash;no Azure subscription required.</span></span>
* <span data-ttu-id="0f9ea-122">Jupyter 및 지원하는 R 또는 Python 배포를 로컬로 설치할 필요가 없음&mdash;브라우저만 사용하면 됩니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-122">No need to install Jupyter and the supporting R or Python distributions locally&mdash;just use a browser.</span></span>
* <span data-ttu-id="0f9ea-123">자체 온라인 라이브러리를 관리하고 모든 장치에서 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-123">Manage your own online libraries and access them from any device.</span></span>
* <span data-ttu-id="0f9ea-124">공동 작업자와 노트를 공유합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-124">Share your notebooks with collaborators.</span></span>

<span data-ttu-id="0f9ea-125">고려 사항:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-125">Considerations:</span></span>

* <span data-ttu-id="0f9ea-126">오프라인 상태에서는 노트에 액세스할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-126">You will be unable to access your notebooks when offline.</span></span>
* <span data-ttu-id="0f9ea-127">무료 노트 서비스의 제한된 처리 기능만으로 크거나 복잡한 모델을 학습하기 어려울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-127">Limited processing capabilities of the free notebook service may not be enough to train large or complex models.</span></span>

### <a name="data-science-virtual-machine"></a><span data-ttu-id="0f9ea-128">데이터 과학 Virtual Machine</span><span class="sxs-lookup"><span data-stu-id="0f9ea-128">Data science virtual machine</span></span>
<span data-ttu-id="0f9ea-129">데이터 과학 Virtual Machine은 R, Python, Jupyter 노트, Visual Studio Code 및 Machine Learning 모델용 라이브러리(예: Microsoft Cognitive 도구 키트)를 포함하여 데이터 과학자가 일반적으로 사용하는 도구 및 프레임워크를 포함하는 Azure Virtual Machine 이미지입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-129">The data science virtual machine is an Azure virtual machine image that includes the tools and frameworks commonly used by data scientists, including R, Python, Jupyter Notebooks, Visual Studio Code, and libraries for machine learning modeling such as the Microsoft Cognitive Toolkit.</span></span> <span data-ttu-id="0f9ea-130">이러한 도구는 설치하는 데 복잡하고 시간이 많이 걸리며, 종종 버전 관리 문제를 야기하는 많은 상호 종속성을 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-130">These tools can be complex and time consuming to install, and contain many interdependencies that often lead to version management issues.</span></span> <span data-ttu-id="0f9ea-131">미리 설치된 이미지를 사용하면 데이터 과학자가 환경 문제를 해결하는 데 소요되는 시간을 줄일 수 있으며, 과학자가 수행해야 하는 데이터 탐색 및 모델링 태스크에 좀 더 집중하도록 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-131">Having a preinstalled image can reduce the time data scientists spend troubleshooting environment issues, allowing them to focus on the data exploration and modeling tasks they need to perform.</span></span>

<span data-ttu-id="0f9ea-132">주요 이점:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-132">Key benefits:</span></span>
* <span data-ttu-id="0f9ea-133">데이터 과학 도구 및 프레임워크를 설치 및 관리하고 문제를 해결하는 데 드는 시간을 단축할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-133">Reduced time to install, manage, and troubleshoot data science tools and frameworks.</span></span>
* <span data-ttu-id="0f9ea-134">가장 일반적으로 사용되는 도구 및 프레임워크의 최신 버전이 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-134">The latest versions of all commonly used tools and frameworks are included.</span></span>
* <span data-ttu-id="0f9ea-135">가상 머신 옵션에는 집약적 데이터 모델링을 위한 GPU 기능이 있는 확장성 높은 이미지가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-135">Virtual machine options include highly scalable images with GPU capabilities for intensive data modeling.</span></span>

<span data-ttu-id="0f9ea-136">고려 사항:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-136">Considerations:</span></span>
* <span data-ttu-id="0f9ea-137">오프라인 상태에서는 가상 머신에 액세스할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-137">The virtual machine cannot be accessed when offline.</span></span>
* <span data-ttu-id="0f9ea-138">가상 머신을 실행하면 Azure 요금이 발생하므로, 필요할 때만 실행하도록 주의해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-138">Running a virtual machine incurs Azure charges, so you must be careful to have it running only when required.</span></span>

### <a name="azure-machine-learning"></a><span data-ttu-id="0f9ea-139">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="0f9ea-139">Azure Machine Learning</span></span>

<span data-ttu-id="0f9ea-140">Azure Machine Learning은 Machine Learning 실험 및 모델을 관리하기 위한 클라우드 기반 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-140">Azure Machine Learning is a cloud-based service for managing machine learning experiments and models.</span></span> <span data-ttu-id="0f9ea-141">여기에는 데이터 준비 및 모델링 학습 스크립트를 추적하는 실험 서비스가 포함되며, 반복 간에 모델 성능을 비교할 수 있도록 모든 실행 기록이 유지 관리됩니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-141">It includes an experimentation service that tracks data preparation and modeling training scripts, maintaining a history of all executions so you can compare model performance across iterations.</span></span> <span data-ttu-id="0f9ea-142">Azure Machine Learning Workbench라는 플랫폼 간 클라이언트 도구는 데이터 과학자가 Jupyter 노트 또는 Visual Studio Code 같은 선택한 도구에서 스크립트를 만들 수 있도록 하면서, 스크립트 관리 및 기록을 위한 중앙 인터페이스를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-142">A cross-platform client tool named Azure Machine Learning Workbench provides a central interface for script management and history, while still enabling data scientists to create scripts in their tool of choice, such as Jupyter Notebooks or Visual Studio Code.</span></span>

<span data-ttu-id="0f9ea-143">Azure Machine Learning Workbench에서 대화형 데이터 준비 도구를 사용하여 일반적인 데이터 변환 작업을 단순화하고, 확장 가능한 Docker 컨테이너 또는 Spark에서 모델 학습 스크립트를 로컬로 실행하도록 스크립트 실행 환경을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-143">From Azure Machine Learning Workbench, you can use the interactive data preparation tools to simplify common data transformation tasks, and you can configure the script execution environment to run model training scripts locally, in a scalable Docker container, or in Spark.</span></span>

<span data-ttu-id="0f9ea-144">모델을 배포할 준비가 되면, Workbench 환경을 사용하여 모델을 패키지하고 Docker 컨테이너, Azure HDinsight의 Spark, Microsoft Machine Learning Server 또는 SQL Server에 웹 서비스로 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-144">When you are ready to deploy your model, use the Workbench environment to package the model and deploy it as a web service to a Docker container, Spark on Azure HDinsight, Microsoft Machine Learning Server, or SQL Server.</span></span> <span data-ttu-id="0f9ea-145">그런 다음 Azure Machine Learning 모델 관리 서비스를 사용하여 클라우드, 에지 장치 또는 엔터프라이즈 전체에서 모델 배포를 추적하고 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-145">The Azure Machine Learning Model Management service then enables you to track and manage model deployments in the cloud, on edge devices, or across the enterprise.</span></span>

<span data-ttu-id="0f9ea-146">주요 이점:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-146">Key benefits:</span></span>

* <span data-ttu-id="0f9ea-147">스크립트 및 실행 기록이 중앙에서 관리되므로 모델 버전을 쉽게 비교할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-147">Central management of scripts and run history, making it easy to compare model versions.</span></span>
* <span data-ttu-id="0f9ea-148">시각적 편집기를 통해 대화형 데이터를 변환합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-148">Interactive data transformation through a visual editor.</span></span>
* <span data-ttu-id="0f9ea-149">클라우드 또는 에지 장치로 모델을 쉽게 배포 및 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-149">Easy deployment and management of models to the cloud or edge devices.</span></span>

<span data-ttu-id="0f9ea-150">고려 사항:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-150">Considerations:</span></span>
* <span data-ttu-id="0f9ea-151">모델 관리 모델과 워크벤치 도구 환경을 어느 정도 숙지합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-151">Requires some familiarity with the model management model and Workbench tool environment.</span></span>

### <a name="azure-batch-ai"></a><span data-ttu-id="0f9ea-152">Azure Batch AI</span><span class="sxs-lookup"><span data-stu-id="0f9ea-152">Azure Batch AI</span></span>

<span data-ttu-id="0f9ea-153">Azure Batch AI를 사용하면 Machine Learning 실험을 동시에 실행하고, GPU를 사용하여 가상 머신 클러스터 간에 대규모 모델 학습을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-153">Azure Batch AI enables you to run your machine learning experiments in parallel, and perform model training at scale across a cluster of virtual machines with GPUs.</span></span> <span data-ttu-id="0f9ea-154">일괄 처리 AI 교육을 사용하면 Cognitive 도구 키트, Caffe, Chainer 및 TensorFlow와 같은 프레임워크를 사용하여 클러스터링된 GPU에서 심층 교육 작업을 스케일 아웃할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-154">Batch AI training enables you to scale out deep learning jobs across clustered GPUs, using frameworks such as Cognitive Toolkit, Caffe, Chainer, and TensorFlow.</span></span> 

<span data-ttu-id="0f9ea-155">Azure Machine Learning 모델 관리를 일괄 처리 AI 교육에서 모델을 생성하여 배포, 관리 및 모니터링하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-155">Azure Machine Learning Model Management can be used to take models from Batch AI training to deploy, manage, and monitor them.</span></span> 

### <a name="azure-machine-learning-studio"></a><span data-ttu-id="0f9ea-156">Azure Machine Learning Studio</span><span class="sxs-lookup"><span data-stu-id="0f9ea-156">Azure Machine Learning Studio</span></span>

<span data-ttu-id="0f9ea-157">Azure Machine Learning Studio는 데이터 실험을 만들고, Machine Learning 모델을 학습하고, Azure에 웹 서비스로 게시하기 위한 클라우드 기반, 시각적 개발 환경입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-157">Azure Machine Learning Studio is a cloud-based, visual development environment for creating data experiments, training machine learning models, and publishing them as web services in Azure.</span></span> <span data-ttu-id="0f9ea-158">시각적 끌어서 놓기 인터페이스를 사용하여 데이터 과학자 및 파워 유저는 Machine Learning 솔루션을 빠르게 만들 수 있으며, 사용자 지정 R 및 Python 논리, Machine Learning 모델링 태스크에 대해 설정된 다양한 통계 알고리즘 및 Jupyter 노트에 대한 기본 제공 지원도 활용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-158">Its visual drag-and-drop interface lets data scientists and power users create machine learning solutions quickly, while supporting custom R and Python logic, a wide range of established statistical algorithms and techniques for machine learning modeling tasks, and built-in support for Jupyter Notebooks.</span></span>

<span data-ttu-id="0f9ea-159">주요 이점:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-159">Key benefits:</span></span>

* <span data-ttu-id="0f9ea-160">대화형 시각적 인터페이스를 사용하면 최소한의 코드로 Machine Learning 모델링을 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-160">Interactive visual interface enables machine learning modeling with minimal code.</span></span>
* <span data-ttu-id="0f9ea-161">데이터 탐색을 위한 기본 제공 Jupyter 노트</span><span class="sxs-lookup"><span data-stu-id="0f9ea-161">Built-in Jupyter Notebooks for data exploration.</span></span>
* <span data-ttu-id="0f9ea-162">학습된 모델을 Azure 웹 서비스로서 직접 배포</span><span class="sxs-lookup"><span data-stu-id="0f9ea-162">Direct deployment of trained models as Azure web services.</span></span>

<span data-ttu-id="0f9ea-163">고려 사항:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-163">Considerations:</span></span>

* <span data-ttu-id="0f9ea-164">제한된 확장성.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-164">Limited scalability.</span></span> <span data-ttu-id="0f9ea-165">학습 데이터 집합의 최대 크기는 10GB입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-165">The maximum size of a training dataset is 10 GB.</span></span>
* <span data-ttu-id="0f9ea-166">온라인 전용</span><span class="sxs-lookup"><span data-stu-id="0f9ea-166">Online only.</span></span> <span data-ttu-id="0f9ea-167">오프라인 개발 환경은 없습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-167">No offline development environment.</span></span>

## <a name="tools-and-services-for-deploying-machine-learning-models"></a><span data-ttu-id="0f9ea-168">Machine Learning 모델을 배포하기 위한 도구 및 서비스</span><span class="sxs-lookup"><span data-stu-id="0f9ea-168">Tools and services for deploying machine learning models</span></span>

<span data-ttu-id="0f9ea-169">데이터 과학자가 기계 학습 모델을 만들고 나면 사용자는 배포한 후 응용 프로그램 또는 다른 데이터 흐름에 사용해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-169">After a data scientist has created a machine learning model, you will typically need to deploy it and consume it from applications or in other data flows.</span></span> <span data-ttu-id="0f9ea-170">Machine Learning 모델에 대한 잠재적인 배포 대상은 많이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-170">There are a number of potential deployment targets for machine learning models.</span></span>

### <a name="spark-on-azure-hdinsight"></a><span data-ttu-id="0f9ea-171">Azure HDInsight의 Spark</span><span class="sxs-lookup"><span data-stu-id="0f9ea-171">Spark on Azure HDInsight</span></span>

<span data-ttu-id="0f9ea-172">Apache Spark에는 Machine Learning 모델을 위한 Spark MLlib, 프레임워크 및 라이브러리가 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-172">Apache Spark includes Spark MLlib, a framework and library for machine learning models.</span></span> <span data-ttu-id="0f9ea-173">MMLSpark(Spark용 Microsoft Machine Learning 라이브러리)에서도 Spark의 예측 모델에 대해 심화 학습 알고리즘 지원을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-173">The Microsoft Machine Learning library for Spark (MMLSpark) also provides deep learning algorithm support for predictive models in Spark.</span></span>

<span data-ttu-id="0f9ea-174">주요 이점:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-174">Key benefits:</span></span>

* <span data-ttu-id="0f9ea-175">Spark는 대용량 Machine Learning 프로세스에 대해 높은 확장성을 제공하는 분산 플랫폼입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-175">Spark is a distributed platform that offers high scalability for high-volume machine learning processes.</span></span>
* <span data-ttu-id="0f9ea-176">Azure Machine Learning Workbench에서 HDinsight의 Spark로 직접 모델을 배포하고, Azure Machine Learning 모델 관리 서비스를 사용하여 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-176">You can deploy models directly to Spark in HDinsight from Azure Machine Learning Workbench, and manage them using the Azure Machine Learning Model Management service.</span></span>

<span data-ttu-id="0f9ea-177">고려 사항:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-177">Considerations:</span></span>

* <span data-ttu-id="0f9ea-178">Spark는 실행하는 전체 시간에 요금이 부과되는 HDinsight 클러스터에서 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-178">Spark runs in an HDinsght cluster that incurs charges the whole time it is running.</span></span> <span data-ttu-id="0f9ea-179">Machine Learning 서비스를 가끔씩만 사용하는 경우에는 이것이 불필요한 비용일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-179">If the machine learning service will only be used occasionally, this may result in unnecessary costs.</span></span>

### <a name="web-service-in-a-container"></a><span data-ttu-id="0f9ea-180">컨테이너의 웹 서비스</span><span class="sxs-lookup"><span data-stu-id="0f9ea-180">Web service in a container</span></span>

<span data-ttu-id="0f9ea-181">Docker 컨테이너에서 Python 웹 서비스로 Machine Learning 모델을 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-181">You can deploy a machine learning model as a Python web service in a Docker container.</span></span> <span data-ttu-id="0f9ea-182">Azure 또는 에지 장치에 모델을 배포할 수 있습니다. 이 경우 모델이 작동하는 데이터와 함께 로컬로 데이터를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-182">You can deploy the model to Azure or to an edge device, where it can be used locally with the data on which it operates.</span></span>

<span data-ttu-id="0f9ea-183">주요 이점:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-183">Key Benefits:</span></span>

* <span data-ttu-id="0f9ea-184">컨테이너는 서비스를 패키지 및 배포하는 간단하면서 비용 효율적인 방법입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-184">Containers are a lightweight and generally cost effective way to package and deploy services.</span></span>
* <span data-ttu-id="0f9ea-185">에지 장치에 배포하는 기능을 사용하면 예측 논리를 데이터에 좀 더 가까운 위치로 이동할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-185">The ability to deploy to an edge device enables you to move your predictive logic closer to the data.</span></span>
* <span data-ttu-id="0f9ea-186">Azure Machine Learning Workbench에서 직접 컨테이너에 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-186">You can deploy to a container directly from Azure Machine Learning Workbench.</span></span>

<span data-ttu-id="0f9ea-187">고려 사항:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-187">Considerations:</span></span>

* <span data-ttu-id="0f9ea-188">이 배포 모델은 Docker 컨테이너를 기준으로 하므로, 이러한 방식으로 웹 서비스를 배포하기 전에 이 기술을 숙지하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-188">This deployment model is based on Docker containers, so you should be familiar with this technology before deploying a web service this way.</span></span>

### <a name="microsoft-machine-learning-server"></a><span data-ttu-id="0f9ea-189">Microsoft Machine Learning 서버</span><span class="sxs-lookup"><span data-stu-id="0f9ea-189">Microsoft Machine Learning Server</span></span>

<span data-ttu-id="0f9ea-190">Machine Learning Server(이전의 Microsoft R Server)는 Machine Learning 시나리오에 맞게 특수하게 디자인된 R 및 Python 코드를 위한 확장 가능한 플랫폼입니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-190">Machine Learning Server (formerly Microsoft R Server) is a scalable platform for R and Python code, specifically designed for machine learning scenarios.</span></span>

<span data-ttu-id="0f9ea-191">주요 이점:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-191">Key benefits:</span></span>

* <span data-ttu-id="0f9ea-192">높은 확장성.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-192">High scalability.</span></span>
* <span data-ttu-id="0f9ea-193">Azure Machine Learning Workbench에서 직접 배포</span><span class="sxs-lookup"><span data-stu-id="0f9ea-193">Direct deployment from Azure Machine Learning Workbench.</span></span>

<span data-ttu-id="0f9ea-194">고려 사항:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-194">Considerations:</span></span>

* <span data-ttu-id="0f9ea-195">엔터프라이즈에서 Machine Learning Server를 배포하고 관리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-195">You need to deploy and manage Machine Learning Server in your enterprise.</span></span>

### <a name="microsoft-sql-server"></a><span data-ttu-id="0f9ea-196">Microsoft SQL Server에 대한 연결 문자열</span><span class="sxs-lookup"><span data-stu-id="0f9ea-196">Microsoft SQL Server</span></span>

<span data-ttu-id="0f9ea-197">Microsoft SQL Server는 기본적으로 R 및 Python을 지원하므로, 이러한 언어로 작성된 Machine Learning 모델을 데이터베이스에 Transact-SQL 함수로 캡슐화할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-197">Microsoft SQL Server supports R and Python natively, enabling you to encapsulate machine learning models built in these languages as Transact-SQL functions in a database.</span></span>

<span data-ttu-id="0f9ea-198">주요 이점:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-198">Key benefits:</span></span>

* <span data-ttu-id="0f9ea-199">예측 논리를 데이터베이스 함수에 캡슐화하여 데이터 계층 논리에 쉽게 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-199">Encapsulate predictive logic in a database function, making it easy to include in data-tier logic.</span></span>

<span data-ttu-id="0f9ea-200">고려 사항:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-200">Considerations:</span></span>

* <span data-ttu-id="0f9ea-201">SQL Server Database를 응용 프로그램에 대한 데이터 계층으로 간주합니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-201">Assumes a SQL Server database as the data tier for your application.</span></span>

### <a name="azure-machine-learning-web-service"></a><span data-ttu-id="0f9ea-202">Azure Machine Learning 웹 서비스</span><span class="sxs-lookup"><span data-stu-id="0f9ea-202">Azure Machine Learning web service</span></span>

<span data-ttu-id="0f9ea-203">Azure Machine Learning Studio를 사용하여 Machine Learning 모델을 만들 경우 해당 모델을 웹 서비스로서 배포할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-203">When you create a machine learning model using Azure Machine Learning Studio, you can deploy it as a web service.</span></span> <span data-ttu-id="0f9ea-204">그런 후 HTTP에서 통신 가능한 모든 클라이언트 응용 프로그램에서 REST 인터페이스를 통해 이 모델을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-204">This can then be consumed through a REST interface from any client applications capable of communicating by HTTP.</span></span>

<span data-ttu-id="0f9ea-205">주요 이점:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-205">Key benefits:</span></span>

* <span data-ttu-id="0f9ea-206">쉬운 개발 및 배포</span><span class="sxs-lookup"><span data-stu-id="0f9ea-206">Ease of development and deployment.</span></span>
* <span data-ttu-id="0f9ea-207">기본 모니터링 메트릭을 사용하는 웹 서비스 관리 포털</span><span class="sxs-lookup"><span data-stu-id="0f9ea-207">Web service management portal with basic monitoring metrics.</span></span>
* <span data-ttu-id="0f9ea-208">Azure Data Lake Analytics, Azure Data Factory 및 Azure Stream Analytics에서 Azure Machine Learning 웹 서비스를 호출하기 위한 지원이 기본적으로 제공</span><span class="sxs-lookup"><span data-stu-id="0f9ea-208">Built-in support for calling Azure Machine Learning web services from Azure Data Lake Analytics, Azure Data Factory, and Azure Stream Analytics.</span></span>

<span data-ttu-id="0f9ea-209">고려 사항:</span><span class="sxs-lookup"><span data-stu-id="0f9ea-209">Considerations:</span></span>

* <span data-ttu-id="0f9ea-210">Azure Machine Learning Studio를 사용하여 작성된 모델에만 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-210">Only available for models built using Azure Machine Learning Studio.</span></span>
* <span data-ttu-id="0f9ea-211">웹 기반 액세스 전용의 학습된 모델은 온-프레미스 또는 오프라인에서 실행할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="0f9ea-211">Web-based access only, trained models cannot run on-premises or offline.</span></span>

