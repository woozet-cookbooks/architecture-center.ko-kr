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
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a><span data-ttu-id="1750a-102">Azure에서 자연어 처리 기술 선택</span><span class="sxs-lookup"><span data-stu-id="1750a-102">Choosing a natural language processing technology in Azure</span></span>

<span data-ttu-id="1750a-103">자유 형식 텍스트 처리는 일반적으로 검색을 지원하기 위해 텍스트 단락을 포함하는 문서에 대해 수행되지만, 감성 분석, 토픽 감지, 언어 감지, 핵심 구 추출 및 문서 분류와 같은 기타 NLP(자연어 처리) 작업을 수행하는 데도 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-103">Free-form text processing is performed against documents containing paragraphs of text, typically for the purpose of supporting search, but is also used to perform other natural language processing (NLP) tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span> <span data-ttu-id="1750a-104">이 문서에서는 NLP 작업을 지원하는 기술 선택 사항을 집중적으로 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-104">This article focuses on the technology choices that act in support of the NLP tasks.</span></span>

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a><span data-ttu-id="1750a-105">NLP 서비스를 선택할 때 사용할 수 있는 옵션은 무엇인가요?</span><span class="sxs-lookup"><span data-stu-id="1750a-105">What are your options when choosing an NLP service?</span></span>

<span data-ttu-id="1750a-106">Azure에서 다음 서비스는 NLP(자연어 처리) 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-106">In Azure, the following services provide natural language processing (NLP) capabilities:</span></span>

- [<span data-ttu-id="1750a-107">Azure HDInsight(Spark 및 Spark MLlib 포함)</span><span class="sxs-lookup"><span data-stu-id="1750a-107">Azure HDInsight with Spark and Spark MLlib</span></span>](/azure/hdinsight/spark/apache-spark-overview)
- [<span data-ttu-id="1750a-108">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="1750a-108">Microsoft Cognitive Services</span></span>](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a><span data-ttu-id="1750a-109">주요 선택 조건</span><span class="sxs-lookup"><span data-stu-id="1750a-109">Key selection criteria</span></span>

<span data-ttu-id="1750a-110">선택 옵션의 범위를 좁히려면 먼저 다음 질문에 답변합니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-110">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="1750a-111">미리 작성된 모델을 사용하려고 하나요?</span><span class="sxs-lookup"><span data-stu-id="1750a-111">Do you want to use prebuilt models?</span></span> <span data-ttu-id="1750a-112">그렇다면 Microsoft Cognitive Services에서 제공하는 API를 사용하는 것을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-112">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

- <span data-ttu-id="1750a-113">대량의 텍스트 데이터에 대해 사용자 지정 모델을 학습해야 하나요?</span><span class="sxs-lookup"><span data-stu-id="1750a-113">Do you need to train custom models against a large corpus of text data?</span></span> <span data-ttu-id="1750a-114">그렇다면 Spark MLlib 및 Spark NLP와 함께 Azure HDInsight를 사용하는 것을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-114">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="1750a-115">토큰화, 형태소 분석, 기본형 찾기 및 TF/IDF(용어 빈도/역 문서 빈도) 같은 기본적인 NLP 기능이 필요한가요?</span><span class="sxs-lookup"><span data-stu-id="1750a-115">Do you need low-level NLP capabilities like tokenization, stemming, lemmatization, and term frequency/inverse document frequency (TF/IDF)?</span></span> <span data-ttu-id="1750a-116">그렇다면 Spark MLlib 및 Spark NLP와 함께 Azure HDInsight를 사용하는 것을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-116">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="1750a-117">엔터티 및 의도 식별, 토픽 감지, 맞춤법 검사 또는 감성 분석 같은 간단한 고급 NLP 기능이 필요한가요?</span><span class="sxs-lookup"><span data-stu-id="1750a-117">Do you need simple, high-level NLP capabilities like entity and intent identification, topic detection, spell check, or sentiment analysis?</span></span> <span data-ttu-id="1750a-118">그렇다면 Microsoft Cognitive Services에서 제공하는 API를 사용하는 것을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-118">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="1750a-119">기능 매트릭스</span><span class="sxs-lookup"><span data-stu-id="1750a-119">Capability matrix</span></span>

<span data-ttu-id="1750a-120">다음 표에서는 주요 기능 차이점을 요약해서 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-120">The following tables summarize the key differences in capabilities.</span></span>  

### <a name="general-capabilities"></a><span data-ttu-id="1750a-121">일반 기능</span><span class="sxs-lookup"><span data-stu-id="1750a-121">General capabilities</span></span>

| | <span data-ttu-id="1750a-122">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="1750a-122">Azure HDInsight</span></span> | <span data-ttu-id="1750a-123">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="1750a-123">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="1750a-124">미리 학습된 모델을 서비스로 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-124">Provides pretrained models as a service</span></span> | <span data-ttu-id="1750a-125">아니요</span><span class="sxs-lookup"><span data-stu-id="1750a-125">No</span></span> | <span data-ttu-id="1750a-126">예</span><span class="sxs-lookup"><span data-stu-id="1750a-126">Yes</span></span> |
| <span data-ttu-id="1750a-127">REST API</span><span class="sxs-lookup"><span data-stu-id="1750a-127">REST API</span></span> | <span data-ttu-id="1750a-128">예</span><span class="sxs-lookup"><span data-stu-id="1750a-128">Yes</span></span> | <span data-ttu-id="1750a-129">예</span><span class="sxs-lookup"><span data-stu-id="1750a-129">Yes</span></span> |
| <span data-ttu-id="1750a-130">프로그래밍 기능</span><span class="sxs-lookup"><span data-stu-id="1750a-130">Programmability</span></span> | <span data-ttu-id="1750a-131">Python, Scala, Java</span><span class="sxs-lookup"><span data-stu-id="1750a-131">Python, Scala, Java</span></span> | <span data-ttu-id="1750a-132">C#, Java, Node.js, Python, PHP, Ruby</span><span class="sxs-lookup"><span data-stu-id="1750a-132">C#, Java, Node.js, Python, PHP, Ruby</span></span> |
| <span data-ttu-id="1750a-133">빅 데이터 집합 및 대형 문서의 처리를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="1750a-133">Support processing of big data sets and large documents</span></span> | <span data-ttu-id="1750a-134">예</span><span class="sxs-lookup"><span data-stu-id="1750a-134">Yes</span></span> | <span data-ttu-id="1750a-135">아니오</span><span class="sxs-lookup"><span data-stu-id="1750a-135">No</span></span> |

### <a name="low-level-natural-language-processing-capabilities"></a><span data-ttu-id="1750a-136">기본적인 자연어 처리 기능</span><span class="sxs-lookup"><span data-stu-id="1750a-136">Low-level natural language processing capabilities</span></span>

| | <span data-ttu-id="1750a-137">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="1750a-137">Azure HDInsight</span></span> | <span data-ttu-id="1750a-138">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="1750a-138">Microsoft Cognitive Services</span></span> |  
| --- | --- | --- | 
| <span data-ttu-id="1750a-139">토크나이저</span><span class="sxs-lookup"><span data-stu-id="1750a-139">Tokenizer</span></span> | <span data-ttu-id="1750a-140">예(Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="1750a-140">Yes (Spark NLP)</span></span> | <span data-ttu-id="1750a-141">예(Linguistic Analysis API)</span><span class="sxs-lookup"><span data-stu-id="1750a-141">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="1750a-142">형태소 분석기</span><span class="sxs-lookup"><span data-stu-id="1750a-142">Stemmer</span></span> | <span data-ttu-id="1750a-143">예(Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="1750a-143">Yes (Spark NLP)</span></span> | <span data-ttu-id="1750a-144">아니오</span><span class="sxs-lookup"><span data-stu-id="1750a-144">No</span></span> |
| <span data-ttu-id="1750a-145">기본형 분석기</span><span class="sxs-lookup"><span data-stu-id="1750a-145">Lemmatizer</span></span> | <span data-ttu-id="1750a-146">예(Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="1750a-146">Yes (Spark NLP)</span></span> | <span data-ttu-id="1750a-147">아니요</span><span class="sxs-lookup"><span data-stu-id="1750a-147">No</span></span> |
| <span data-ttu-id="1750a-148">품사 태그 지정</span><span class="sxs-lookup"><span data-stu-id="1750a-148">Part of speech tagging</span></span> | <span data-ttu-id="1750a-149">예(Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="1750a-149">Yes (Spark NLP)</span></span> | <span data-ttu-id="1750a-150">예(Linguistic Analysis API)</span><span class="sxs-lookup"><span data-stu-id="1750a-150">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="1750a-151">TF/IDF(용어 빈도/역 문서 빈도)</span><span class="sxs-lookup"><span data-stu-id="1750a-151">Term frequency/inverse-document frequency (TF/IDF)</span></span> | <span data-ttu-id="1750a-152">예(Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="1750a-152">Yes (Spark MLlib)</span></span> | <span data-ttu-id="1750a-153">아니오</span><span class="sxs-lookup"><span data-stu-id="1750a-153">No</span></span> |
| <span data-ttu-id="1750a-154">문자열 유사성&mdash;편집 거리 계산</span><span class="sxs-lookup"><span data-stu-id="1750a-154">String similarity&mdash;edit distance calculation</span></span> | <span data-ttu-id="1750a-155">예(Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="1750a-155">Yes (Spark MLlib)</span></span> | <span data-ttu-id="1750a-156">아니오</span><span class="sxs-lookup"><span data-stu-id="1750a-156">No</span></span> |
| <span data-ttu-id="1750a-157">N-gram 계산</span><span class="sxs-lookup"><span data-stu-id="1750a-157">N-gram calculation</span></span> | <span data-ttu-id="1750a-158">예(Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="1750a-158">Yes (Spark MLlib)</span></span> | <span data-ttu-id="1750a-159">아니오</span><span class="sxs-lookup"><span data-stu-id="1750a-159">No</span></span> |
| <span data-ttu-id="1750a-160">중지 단어 제거</span><span class="sxs-lookup"><span data-stu-id="1750a-160">Stop word removal</span></span> | <span data-ttu-id="1750a-161">예(Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="1750a-161">Yes (Spark MLlib)</span></span> | <span data-ttu-id="1750a-162">아니오</span><span class="sxs-lookup"><span data-stu-id="1750a-162">No</span></span> |

### <a name="high-level-natural-language-processing-capabilities"></a><span data-ttu-id="1750a-163">고급 자연어 처리 기능</span><span class="sxs-lookup"><span data-stu-id="1750a-163">High-level natural language processing capabilities</span></span>

| | <span data-ttu-id="1750a-164">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="1750a-164">Azure HDInsight</span></span> | <span data-ttu-id="1750a-165">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="1750a-165">Microsoft Cognitive Services</span></span> |
| --- | --- | --- | 
| <span data-ttu-id="1750a-166">엔터티/의도 식별 및 추출</span><span class="sxs-lookup"><span data-stu-id="1750a-166">Entity/intent identification & extraction</span></span> | <span data-ttu-id="1750a-167">아니요</span><span class="sxs-lookup"><span data-stu-id="1750a-167">No</span></span> | <span data-ttu-id="1750a-168">예(LUIS(언어 인식 인텔리전트 서비스) API)</span><span class="sxs-lookup"><span data-stu-id="1750a-168">Yes (Language Understanding Intelligent Service (LUIS) API)</span></span> |    
| <span data-ttu-id="1750a-169">토픽 검색</span><span class="sxs-lookup"><span data-stu-id="1750a-169">Topic detection</span></span> | <span data-ttu-id="1750a-170">예(Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="1750a-170">Yes (Spark NLP)</span></span> | <span data-ttu-id="1750a-171">예(Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="1750a-171">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="1750a-172">맞춤법 검사</span><span class="sxs-lookup"><span data-stu-id="1750a-172">Spell checking</span></span> | <span data-ttu-id="1750a-173">예(Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="1750a-173">Yes (Spark NLP)</span></span> | <span data-ttu-id="1750a-174">예(Bing Spell Check API)</span><span class="sxs-lookup"><span data-stu-id="1750a-174">Yes (Bing Spell Check API)</span></span> |
| <span data-ttu-id="1750a-175">정서 분석</span><span class="sxs-lookup"><span data-stu-id="1750a-175">Sentiment analysis</span></span> | <span data-ttu-id="1750a-176">예(Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="1750a-176">Yes (Spark NLP)</span></span> | <span data-ttu-id="1750a-177">예(Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="1750a-177">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="1750a-178">언어 검색</span><span class="sxs-lookup"><span data-stu-id="1750a-178">Language detection</span></span> | <span data-ttu-id="1750a-179">아니오</span><span class="sxs-lookup"><span data-stu-id="1750a-179">No</span></span> | <span data-ttu-id="1750a-180">예(Text Analytics API)</span><span class="sxs-lookup"><span data-stu-id="1750a-180">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="1750a-181">영어 이외의 다국어 지원</span><span class="sxs-lookup"><span data-stu-id="1750a-181">Supports multiple languages besides English</span></span> | <span data-ttu-id="1750a-182">아니요</span><span class="sxs-lookup"><span data-stu-id="1750a-182">No</span></span> | <span data-ttu-id="1750a-183">예(API에 따라 다름)</span><span class="sxs-lookup"><span data-stu-id="1750a-183">Yes (varies by API)</span></span> |

## <a name="see-also"></a><span data-ttu-id="1750a-184">참고 항목</span><span class="sxs-lookup"><span data-stu-id="1750a-184">See also</span></span>

[<span data-ttu-id="1750a-185">자연어 처리</span><span class="sxs-lookup"><span data-stu-id="1750a-185">Natural language processing</span></span>](../scenarios/natural-language-processing.md)