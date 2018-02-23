---
title: "검색을 위해 자유 형식 텍스트 처리"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: e53730bd2e179c82399e0f92b6c5ce7f451a2f46
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/14/2018
---
# <a name="processing-free-form-text-for-search"></a><span data-ttu-id="703a6-102">검색을 위해 자유 형식 텍스트 처리</span><span class="sxs-lookup"><span data-stu-id="703a6-102">Processing free-form text for search</span></span>

<span data-ttu-id="703a6-103">검색을 지원하기 위해 텍스트 단락이 포함된 문서에 대해 자유 형식 텍스트 처리를 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-103">To support search, free-form text processing can be performed against documents containing paragraphs of text.</span></span>

<span data-ttu-id="703a6-104">텍스트 검색은 문서 컬렉션에 대해 미리 계산되는 특수한 인덱스를 구성하여 작동합니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-104">Text search works by constructing a specialized index that is precomputed against a collection of documents.</span></span> <span data-ttu-id="703a6-105">클라이언트 응용 프로그램은 검색 용어를 포함하는 쿼리를 제출합니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-105">A client application submits a query that contains the search terms.</span></span> <span data-ttu-id="703a6-106">이 쿼리는 각 문서가 검색 조건과 얼마나 잘 일치하는지를 기준으로 정렬된 문서 목록으로 구성되는 결과 집합을 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-106">The query returns a result set, consisting of a list of documents sorted by how well each document matches the search criteria.</span></span> <span data-ttu-id="703a6-107">또한 결과 집합에는 해당 문서가 기준과 일치하는 컨텍스트도 포함되어, 응용 프로그램에서 문서의 일치하는 구문이 강조 표시될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-107">The result set may also include the context in which the document matches the criteria, which enables the application to highlight the matching phrase in the document.</span></span> 

![](./images/search-pipeline.png)

<span data-ttu-id="703a6-108">자유 형식 텍스트 처리는 많은 양의 불필요한 텍스트 데이터에서 유용하고 실행 가능한 데이터를 생성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-108">Free-form text processing can produce useful, actionable data from large amounts of noisy text data.</span></span> <span data-ttu-id="703a6-109">결과적으로 구조화되지 않은 문서가 잘 정의되고 쿼리 가능한 구조를 갖게 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-109">The results can give unstructured documents a well-defined and queryable structure.</span></span>


## <a name="challenges"></a><span data-ttu-id="703a6-110">과제</span><span class="sxs-lookup"><span data-stu-id="703a6-110">Challenges</span></span>

- <span data-ttu-id="703a6-111">자유 형식 텍스트 문서 컬렉션의 처리는 일반적으로 시간이 오래 걸릴 뿐만 아니라 계산도 많이 수반됩니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-111">Processing a collection of free-form text documents is typically computationally intensive, as well as time intensive.</span></span>
- <span data-ttu-id="703a6-112">자유 형식 텍스트를 효율적으로 검색하기 위해 검색 인덱스는 비슷한 구성을 갖는 용어를 기준으로 유사 항목 검색을 지원해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-112">In order to search free-form text effectively, the search index should support fuzzy search based on terms that have a similar construction.</span></span> <span data-ttu-id="703a6-113">예를 들어, 검색 인덱스는 표준형 및 언어적 형태소 분석을 사용하여 작성되므로 "run"을 쿼리하면 "ran" 및 "running"을 포함하는 문서가 일치하는 항목으로 검색됩니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-113">For example, search indexes are built with lemmatization and linguistic stemming, so that queries for "run" will match documents that contain "ran" and "running."</span></span>

## <a name="architecture"></a><span data-ttu-id="703a6-114">건축</span><span class="sxs-lookup"><span data-stu-id="703a6-114">Architecture</span></span>

<span data-ttu-id="703a6-115">대부분의 시나리오에서 원본 텍스트 문서는 Azure Storage 또는 Azure Data Lake Store와 같은 개체 저장소에 로드됩니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-115">In most scenarios, the source text documents are loaded into object storage such as Azure Storage or Azure Data Lake Store.</span></span> <span data-ttu-id="703a6-116">예외는 SQL Server 또는 Azure SQL Database 내에서 전체 텍스트 검색을 사용하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-116">An exception is using full text search within SQL Server or Azure SQL Database.</span></span> <span data-ttu-id="703a6-117">이 경우 문서 데이터는 데이터베이스에서 관리되는 테이블에 로드됩니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-117">In this case, the document data is loaded into tables managed by the database.</span></span> <span data-ttu-id="703a6-118">일단 저장되면 문서는 일괄로 처리되어 인덱스를 생성합니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-118">Once stored, the documents are processed in a batch to create the index.</span></span>

## <a name="technology-choices"></a><span data-ttu-id="703a6-119">기술 선택</span><span class="sxs-lookup"><span data-stu-id="703a6-119">Technology choices</span></span>

<span data-ttu-id="703a6-120">검색 인덱스를 만드는 옵션에는 Azure Search, Elasticsearch, HDInsight(Solr 포함)가 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-120">Options for creating a search index include Azure Search, Elasticsearch, and HDInsight with Solr.</span></span> <span data-ttu-id="703a6-121">이러한 각 기술은 문서 컬렉션으로 검색 인덱스를 채울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-121">Each of these technologies can populate a search index from a collection of documents.</span></span> <span data-ttu-id="703a6-122">Azure Search는 일반 텍스트에서 Excel 및 PDF 형식 문서에 대한 인덱스를 자동으로 채울 수 있는 인덱서를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-122">Azure Search provides indexers that can automatically populate the index for documents ranging from plain text to Excel and PDF formats.</span></span> <span data-ttu-id="703a6-123">HDInsight에서 Apache Solr은 일반 텍스트, Word 및 PDF를 비롯한 다양한 형식의 이진 파일을 인덱싱할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-123">On HDInsight, Apache Solr can index binary files of many types, including plain text, Word, and PDF.</span></span> <span data-ttu-id="703a6-124">인덱스가 생성되면 클라이언트는 REST API를 사용하여 검색 인터페이스에 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-124">Once the index is constructed, clients can access the search interface by means of a REST API.</span></span> 

<span data-ttu-id="703a6-125">텍스트 데이터가 SQL Server 또는 Azure SQL Database에 저장되면 데이터베이스로 작성된 전체 텍스트 검색을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-125">If your text data is stored in SQL Server or Azure SQL Database, you can use the full-text search that is built into the database.</span></span> <span data-ttu-id="703a6-126">데이터베이스는 텍스트, 이진 또는 동일한 데이터베이스 내에 저장된 XML 데이터를 사용해서 인덱스를 채웁니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-126">The database populates the index from text, binary, or XML data stored within the same database.</span></span> <span data-ttu-id="703a6-127">클라이언트는 T-SQL 쿼리를 사용하여 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="703a6-127">Clients search by using T-SQL queries.</span></span> 

<span data-ttu-id="703a6-128">자세한 내용은 [지원되는 데이터 원본](../technology-choices/search-options.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="703a6-128">For more information, see [Search data stores](../technology-choices/search-options.md).</span></span>
