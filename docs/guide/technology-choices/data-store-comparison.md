---
title: 데이터 저장소를 선택하는 기준
description: Azure 계산 옵션 개요
author: MikeWasson
ms.openlocfilehash: 70f746f80c29623004620d83eb38747777df7f84
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252876"
---
# <a name="criteria-for-choosing-a-data-store"></a><span data-ttu-id="76e71-103">데이터 저장소를 선택하는 기준</span><span class="sxs-lookup"><span data-stu-id="76e71-103">Criteria for choosing a data store</span></span>

<span data-ttu-id="76e71-104">Azure는 다양한 유형의 데이터 저장소 솔루션을 지원합니다. 이러한 데이터 저장소 솔루션은 서로 다른 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-104">Azure supports many types of data storage solutions, each providing different features and capabilities.</span></span> <span data-ttu-id="76e71-105">이 문서에서는 데이터 저장소를 평가할 때 사용해야 하는 비교 기준에 대해 설명합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-105">This article describes the comparison criteria you should use when evaluating a data store.</span></span> <span data-ttu-id="76e71-106">이 문서의 목표는 사용자의 솔루션 요구 사항을 충족하는 데이터 저장소 유형을 판단할 수 있도록 지원하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-106">The goal is to help you determine which data storage types can meet your solution's requirements.</span></span>

## <a name="general-considerations"></a><span data-ttu-id="76e71-107">일반적인 고려 사항</span><span class="sxs-lookup"><span data-stu-id="76e71-107">General Considerations</span></span>

<span data-ttu-id="76e71-108">비교를 시작할 때는 데이터 요구에 대해 다음과 같은 정보를 가능한 한 많이 수집합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-108">To start your comparison, gather as much of the following information as you can about your data needs.</span></span> <span data-ttu-id="76e71-109">이러한 정보는 어느 데이터 저장소 유형이 사용자의 데이터 요구를 충족하는지 판단하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-109">This information will help you to determine which data storage types will meet your needs.</span></span>

### <a name="functional-requirements"></a><span data-ttu-id="76e71-110">기능적인 요구 사항</span><span class="sxs-lookup"><span data-stu-id="76e71-110">Functional requirements</span></span>

- <span data-ttu-id="76e71-111">**데이터 형식**.</span><span class="sxs-lookup"><span data-stu-id="76e71-111">**Data format**.</span></span> <span data-ttu-id="76e71-112">저장하려는 데이터의 유형은 무엇인가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-112">What type of data are you intending to store?</span></span> <span data-ttu-id="76e71-113">일반적인 유형에는 트랜잭션 데이터, JSON 개체, 원격 분석, 검색 인덱스, 플랫 파일 등이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-113">Common types include transactional data, JSON objects, telemetry, search indexes, or flat files.</span></span>
- <span data-ttu-id="76e71-114">**데이터 크기**.</span><span class="sxs-lookup"><span data-stu-id="76e71-114">**Data size**.</span></span> <span data-ttu-id="76e71-115">저장하려는 엔터티의 크기는 얼마나 되나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-115">How large are the entities you need to store?</span></span> <span data-ttu-id="76e71-116">이 엔터티를 하나의 단일 문서로 관리해야 하나요? 아니면 여러 개의 문서, 테이블, 컬렉션 등으로 분할할 수 있나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-116">Will these entities need to be maintained as a single document, or can they be split across multiple documents, tables, collections, and so forth?</span></span>
- <span data-ttu-id="76e71-117">**규모 및 구조**.</span><span class="sxs-lookup"><span data-stu-id="76e71-117">**Scale and structure**.</span></span> <span data-ttu-id="76e71-118">필요한 전체적인 저장소 용량은 얼마나 되나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-118">What is the overall amount of storage capacity you need?</span></span> <span data-ttu-id="76e71-119">데이터를 분할할 예정인가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-119">Do you anticipate partitioning your data?</span></span> 
- <span data-ttu-id="76e71-120">**데이터 관계**.</span><span class="sxs-lookup"><span data-stu-id="76e71-120">**Data relationships**.</span></span> <span data-ttu-id="76e71-121">데이터가 일대일 또는 다대다 관계를 지원해야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-121">Will your data need to support one-to-many or many-to-many relationships?</span></span> <span data-ttu-id="76e71-122">관계가 데이터의 중요한 부분인가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-122">Are relationships themselves an important part of the data?</span></span> <span data-ttu-id="76e71-123">동일한 데이터 집합의 데이터를 또는 외부 데이터 집합의 데이터를 조인하거나 그 밖의 방식으로 결합해야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-123">Will you need to join or otherwise combine data from within the same dataset, or from external datasets?</span></span> 
- <span data-ttu-id="76e71-124">**일관성 모델**.</span><span class="sxs-lookup"><span data-stu-id="76e71-124">**Consistency model**.</span></span> <span data-ttu-id="76e71-125">하나의 노드에 업데이트를 적용했을 때 추가로 변경을 수행하기 전에 해당 업데이트가 다른 노드에 반영되는 것이 얼마나 중요하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-125">How important is it for updates made in one node to appear in other nodes, before further changes can be made?</span></span> <span data-ttu-id="76e71-126">최종 일관성으로 만족할 수 있나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-126">Can you accept eventual consistency?</span></span> <span data-ttu-id="76e71-127">트랜잭션에서 ACID가 보장되어야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-127">Do you need ACID guarantees for transactions?</span></span>
- <span data-ttu-id="76e71-128">**스키마 유연성**.</span><span class="sxs-lookup"><span data-stu-id="76e71-128">**Schema flexibility**.</span></span> <span data-ttu-id="76e71-129">데이터에 어떤 종류의 스키마를 적용할 예정인가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-129">What kind of schemas will you apply to your data?</span></span> <span data-ttu-id="76e71-130">고정된 스키마, 쓰기 시 스키마 방식, 읽기 시 스키마 방식 중 어느 것을 사용할 예정인가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-130">Will you use a fixed schema, a schema-on-write approach, or a schema-on-read approach?</span></span>
- <span data-ttu-id="76e71-131">**동시성**.</span><span class="sxs-lookup"><span data-stu-id="76e71-131">**Concurrency**.</span></span> <span data-ttu-id="76e71-132">데이터 업데이트 및 동기화 시 어떤 종류의 동시성 메커니즘이 필요한가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-132">What kind of concurrency mechanism do you want to use when updating and synchronizing data?</span></span> <span data-ttu-id="76e71-133">응용 프로그램에서 잠재적 충돌 가능성이 있는 업데이트를 다수 수행할 예정인가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-133">Will the application perform many updates that could potentially conflict.</span></span> <span data-ttu-id="76e71-134">그렇다면 기록 잠금 및 비관적 동시성 제어가 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-134">If so, you may require record locking and pessimistic concurrency control.</span></span> <span data-ttu-id="76e71-135">비관적 동시성 제어 대신 낙관적 동시성 제어를 지원할 수 있나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-135">Alternatively, can you support optimistic concurrency controls?</span></span> <span data-ttu-id="76e71-136">그렇다면 단순한 타임스탬프 기반 동시성 제어로도 충분한가요? 아니면 추가로 여러 버전의 동시성 제어가 필요한가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-136">If so, is simple timestamp-based concurrency control enough, or do you need the added functionality of multi-version concurrency control?</span></span>
- <span data-ttu-id="76e71-137">**데이터 이동**.</span><span class="sxs-lookup"><span data-stu-id="76e71-137">**Data movement**.</span></span> <span data-ttu-id="76e71-138">솔루션에서 데이터를 다른 저장소나 데이터 웨어하우스로 이동하기 위해 ETL 작업을 수행해야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-138">Will your solution need to perform ETL tasks to move data to other stores or data warehouses?</span></span>
- <span data-ttu-id="76e71-139">**데이터 수명 주기**.</span><span class="sxs-lookup"><span data-stu-id="76e71-139">**Data lifecycle**.</span></span> <span data-ttu-id="76e71-140">데이터 쓰기가 한 번 수행되면 읽기가 다수 수행되나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-140">Is the data write-once, read-many?</span></span> <span data-ttu-id="76e71-141">데이터를 쿨 또는 콜드 저장소로 이동할 수 있나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-141">Can it be moved into cool or cold storage?</span></span>
- <span data-ttu-id="76e71-142">**기타 지원되는 기능**.</span><span class="sxs-lookup"><span data-stu-id="76e71-142">**Other supported features**.</span></span> <span data-ttu-id="76e71-143">그 밖에 스키마 유효성 검사, 집계, 인덱스 작업, 전체 텍스트 검색, MapReduce, 기타 쿼리 기능과 같은 특정 기능이 필요하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-143">Do you need any other specific features, such as schema validation, aggregation, indexing, full-text search, MapReduce, or other query capabilities?</span></span>

### <a name="non-functional-requirements"></a><span data-ttu-id="76e71-144">비기능적 요구 사항</span><span class="sxs-lookup"><span data-stu-id="76e71-144">Non-functional requirements</span></span>

- <span data-ttu-id="76e71-145">**성능 및 확장성**.</span><span class="sxs-lookup"><span data-stu-id="76e71-145">**Performance and scalability**.</span></span> <span data-ttu-id="76e71-146">데이터 성능 요구 사항은 어떻게 되나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-146">What are your data performance requirements?</span></span> <span data-ttu-id="76e71-147">데이터 수집률과 데이터 처리율에 대한 구체적인 요구 사항이 있나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-147">Do you have specific requirements for data ingestion rates and data processing rates?</span></span> <span data-ttu-id="76e71-148">수집된 데이터의 허용되는 쿼리 및 집계 응답 시간은 어떻게 되나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-148">What are the acceptable response times for querying and aggregation of data once ingested?</span></span> <span data-ttu-id="76e71-149">데이터 저장소가 얼마만큼 확장되어야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-149">How large will you need the data store to scale up?</span></span> <span data-ttu-id="76e71-150">워크로드가 읽기 집약적인가요 아니면 쓰기 집약적인가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-150">Is your workload more read-heavy or write-heavy?</span></span>
- <span data-ttu-id="76e71-151">**안전성**.</span><span class="sxs-lookup"><span data-stu-id="76e71-151">**Reliability**.</span></span> <span data-ttu-id="76e71-152">지원해야 하는 전반적인 SLA는 어떻게 되나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-152">What overall SLA do you need to support?</span></span> <span data-ttu-id="76e71-153">데이터 사용자에게 제공해야 하는 내결함성 수준은 어떻게 되나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-153">What level of fault-tolerance do you need to provide for data consumers?</span></span> <span data-ttu-id="76e71-154">어떤 백업 및 복원 기능이 필요하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-154">What kind of backup and restore capabilities do you need?</span></span> 
- <span data-ttu-id="76e71-155">**복제**.</span><span class="sxs-lookup"><span data-stu-id="76e71-155">**Replication**.</span></span> <span data-ttu-id="76e71-156">데이터를 여러 개의 복제본 또는 지역으로 분산해야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-156">Will your data need to be distributed among multiple replicas or regions?</span></span> <span data-ttu-id="76e71-157">어떤 종류의 데이터 복제 기능이 필요하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-157">What kind of data replication capabilities do you require?</span></span> 
- <span data-ttu-id="76e71-158">**제한**.</span><span class="sxs-lookup"><span data-stu-id="76e71-158">**Limits**.</span></span> <span data-ttu-id="76e71-159">규모, 연결 개수 및 처리량에 대한 요구 사항을 지원하는 데이터 저장소의 제한은 어떻게 되나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-159">Will the limits of a particular data store support your requirements for scale, number of connections, and throughput?</span></span> 

### <a name="management-and-cost"></a><span data-ttu-id="76e71-160">관리 및 비용</span><span class="sxs-lookup"><span data-stu-id="76e71-160">Management and cost</span></span>

- <span data-ttu-id="76e71-161">**관리되는 서비스**.</span><span class="sxs-lookup"><span data-stu-id="76e71-161">**Managed service**.</span></span> <span data-ttu-id="76e71-162">IaaS로 호스팅되는 데이터 저장소에서만 제공하는 구체적인 기능이 필요한 경우가 아니라면 가능한 한 관리되는 데이터 서비스를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-162">When possible, use a managed data service, unless you require specific capabilities that can only be found in an IaaS-hosted data store.</span></span>
- <span data-ttu-id="76e71-163">**지역 가용성**.</span><span class="sxs-lookup"><span data-stu-id="76e71-163">**Region availability**.</span></span> <span data-ttu-id="76e71-164">관리되는 서비스의 경우, 모든 Azure 지역에서 서비스를 사용할 수 있나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-164">For managed services, is the service available in all Azure regions?</span></span> <span data-ttu-id="76e71-165">솔루션이 특정 Azure 지역에서 호스팅되어야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-165">Does your solution need to be hosted in certain Azure regions?</span></span>
- <span data-ttu-id="76e71-166">**이식성**.</span><span class="sxs-lookup"><span data-stu-id="76e71-166">**Portability**.</span></span> <span data-ttu-id="76e71-167">온-프레미스, 외부 데이터 센터 또는 기타 클라우드 호스팅 환경으로 데이터를 마이그레이션해야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-167">Will your data need to migrated to on-premises, external datacenters, or other cloud hosting environments?</span></span>
- <span data-ttu-id="76e71-168">**라이선스**.</span><span class="sxs-lookup"><span data-stu-id="76e71-168">**Licensing**.</span></span> <span data-ttu-id="76e71-169">독점적 라이선스 유형과 OSS 라이선스 유형 중 어느 것을 선호하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-169">Do you have a preference of a proprietary versus OSS license type?</span></span> <span data-ttu-id="76e71-170">사용 가능한 라이선스 유형에 대한 그 밖의 외부 제한 사항이 존재하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-170">Are there any other external restrictions on what type of license you can use?</span></span>
- <span data-ttu-id="76e71-171">**전체 비용**.</span><span class="sxs-lookup"><span data-stu-id="76e71-171">**Overall cost**.</span></span> <span data-ttu-id="76e71-172">솔루션에서 서비스를 사용할 때 발생하는 전체 비용은 어떻게 되나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-172">What is the overall cost of using the service within your solution?</span></span> <span data-ttu-id="76e71-173">가동 시간 및 처리량 요구 사항을 지원하기 위해 실행해야 하는 인스턴스는 몇 개인가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-173">How many instances will need to run, to support your uptime and throughput requirements?</span></span> <span data-ttu-id="76e71-174">이 계산에서 운영 비용을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-174">Consider operations costs in this calculation.</span></span> <span data-ttu-id="76e71-175">운영 비용 절감은 관리되는 서비스가 선호되는 이유 중 하나입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-175">One reason to prefer managed services is the reduced operational cost.</span></span>
- <span data-ttu-id="76e71-176">**비용 효율성**.</span><span class="sxs-lookup"><span data-stu-id="76e71-176">**Cost effectiveness**.</span></span> <span data-ttu-id="76e71-177">데이터를 더욱 비용 효율적으로 저장하기 위해 데이터를 분할할 수 있나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-177">Can you partition your data, to store it more cost effectively?</span></span> <span data-ttu-id="76e71-178">예를 들어 값비싼 관계형 데이터베이스에서 개체 저장소로 대용량 개체를 이동할 수 있나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-178">For example, can you move large objects out of an expensive relational database into an object store?</span></span>

### <a name="security"></a><span data-ttu-id="76e71-179">보안</span><span class="sxs-lookup"><span data-stu-id="76e71-179">Security</span></span>

- <span data-ttu-id="76e71-180">**보안**.</span><span class="sxs-lookup"><span data-stu-id="76e71-180">**Security**.</span></span> <span data-ttu-id="76e71-181">어떤 유형의 암호화가 필요하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-181">What type of encryption do you require?</span></span> <span data-ttu-id="76e71-182">저장 시 암호화가 필요하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-182">Do you need encryption at rest?</span></span> <span data-ttu-id="76e71-183">데이터를 연결하는 데 어떤 인증 메커니즘을 사용할 예정인가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-183">What authentication mechanism do you want to use to connect to your data?</span></span>
- <span data-ttu-id="76e71-184">**감사**.</span><span class="sxs-lookup"><span data-stu-id="76e71-184">**Auditing**.</span></span> <span data-ttu-id="76e71-185">어떤 종류의 감사 로그를 생성해야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-185">What kind of audit log do you need to generate?</span></span>
- <span data-ttu-id="76e71-186">**네트워킹 요구 사항**.</span><span class="sxs-lookup"><span data-stu-id="76e71-186">**Networking requirements**.</span></span> <span data-ttu-id="76e71-187">다른 네트워크 리소스로부터 데이터를 제한 또는 액세스를 관리해야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-187">Do you need to restrict or otherwise manage access to your data from other network resources?</span></span> <span data-ttu-id="76e71-188">Azure 환경 내부에서만 데이터를 액세스할 수 있어야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-188">Does data need to be accessible only from inside the Azure environment?</span></span> <span data-ttu-id="76e71-189">특정 IP 주소 또는 서브넷에서 데이터를 액세스할 수 있어야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-189">Does the data need to be accessible from specific IP addresses or subnets?</span></span> <span data-ttu-id="76e71-190">온-프레미스 또는 기타 외부 데이터 센터에 호스팅된 응용 프로그램 또는 서비스에서 데이터를 액세스할 수 있어야 하나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-190">Does it need to be accessible from applications or services hosted on-premises or in other external datacenters?</span></span>

### <a name="devops"></a><span data-ttu-id="76e71-191">DevOps</span><span class="sxs-lookup"><span data-stu-id="76e71-191">DevOps</span></span>

- <span data-ttu-id="76e71-192">**기술 수준**.</span><span class="sxs-lookup"><span data-stu-id="76e71-192">**Skill set**.</span></span> <span data-ttu-id="76e71-193">팀원들에게 특히 익숙한 프로그래밍 언어, 운영 체제 또는 기타 기술이 있나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-193">Are there particular programming languages, operating systems, or other technology that your team is particularly adept at using?</span></span> <span data-ttu-id="76e71-194">팀원들이 사용에 어려움을 느낄 프로그래밍 언어, 운영 체제 또는 기타 기술이 있나요?</span><span class="sxs-lookup"><span data-stu-id="76e71-194">Are there others that would be difficult for your team to work with?</span></span>
- <span data-ttu-id="76e71-195">**클라이언트** 사용하는 개발 언어에 대한 클라이언트 지원이 우수한가요?</span><span class="sxs-lookup"><span data-stu-id="76e71-195">**Clients** Is there good client support for your development languages?</span></span>

<span data-ttu-id="76e71-196">다음 섹션에서는 워크로드 프로필, 데이터 유형 및 사용 사례라는 측면에서 다양한 데이터 저장소 모델을 비교합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-196">The following sections compare various data store models in terms of workload profile, data types, and example use cases.</span></span>

## <a name="relational-database-management-systems-rdbms"></a><span data-ttu-id="76e71-197">관계형 데이터베이스 관리 시스템(RDBMS)</span><span class="sxs-lookup"><span data-stu-id="76e71-197">Relational database management systems (RDBMS)</span></span>

<table>
<tr><td><span data-ttu-id="76e71-198"><strong>워크로드</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-198"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-199">새 레코드의 생성과 기존 데이터의 업데이트가 정기적으로 이루어집니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-199">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="76e71-200">하나의 트랜잭션에서 여러 개의 작업이 완료되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-200">Multiple operations have to be completed in a single transaction.</span></span></li>
            <li><span data-ttu-id="76e71-201">교차 집계를 수행하기 위해 집계 함수가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-201">Requires aggregation functions to perform cross-tabulation.</span></span></li>
            <li><span data-ttu-id="76e71-202">보고 도구와의 강력한 통합이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-202">Strong integration with reporting tools is required.</span></span></li>
            <li><span data-ttu-id="76e71-203">관계는 데이터베이스 제약 조건을 사용하여 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-203">Relationships are enforced using database constraints.</span></span></li>
            <li><span data-ttu-id="76e71-204">쿼리 성능을 최적화하는 데 인덱스가 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-204">Indexes are used to optimize query performance.</span></span></li>
            <li><span data-ttu-id="76e71-205">특정 데이터 하위 집합에 대한 액세스를 허용합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-205">Allows access to specific subsets of data.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-206"><strong>데이터 형식</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-206"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-207">데이터가 고도로 정규화되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-207">Data is highly normalized.</span></span></li>
            <li><span data-ttu-id="76e71-208">데이터 스키마가 필요하며, 적용되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-208">Database schemas are required and enforced.</span></span></li>
            <li><span data-ttu-id="76e71-209">데이터베이스에 저장된 데이터 엔터티 사이에 다대다 관계가 존재합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-209">Many-to-many relationships between data entities in the database.</span></span></li>
            <li><span data-ttu-id="76e71-210">스키마에서 제약 조건이 정의되고, 데이터베이스에 저장된 모든 데이터에 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-210">Constraints are defined in the schema and imposed on any data in the database.</span></span></li>
            <li><span data-ttu-id="76e71-211">데이터에 높은 무결성이 요구됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-211">Data requires high integrity.</span></span> <span data-ttu-id="76e71-212">인덱스 및 관계가 정확하게 유지되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-212">Indexes and relationships need to be maintained accurately.</span></span></li>
            <li><span data-ttu-id="76e71-213">데이터에 강력한 일관성이 요구됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-213">Data requires strong consistency.</span></span> <span data-ttu-id="76e71-214">모든 사용자와 프로세스에 대해 모든 데이터가 100% 일관성을 갖도록 트랜잭션이 이루어집니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-214">Transactions operate in a way that ensures all data are 100% consistent for all users and processes.</span></span></li>
            <li><span data-ttu-id="76e71-215">개별 데이터 엔터티의 크기가 작거나 중간 규모입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-215">Size of individual data entries is intended to be small to medium-sized.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-216"><strong>예</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-216"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-217">사업부(인적 자원 관리, 고객 관계 관리, 엔터프라이즈 리소스 계획)</span><span class="sxs-lookup"><span data-stu-id="76e71-217">Line of business  (human capital management, customer relationship management, enterprise resource planning)</span></span></li>
            <li><span data-ttu-id="76e71-218">재고 관리</span><span class="sxs-lookup"><span data-stu-id="76e71-218">Inventory management</span></span></li>
            <li><span data-ttu-id="76e71-219">보고 데이터베이스</span><span class="sxs-lookup"><span data-stu-id="76e71-219">Reporting database</span></span></li>
            <li><span data-ttu-id="76e71-220">회계</span><span class="sxs-lookup"><span data-stu-id="76e71-220">Accounting</span></span></li>
            <li><span data-ttu-id="76e71-221">자산 관리</span><span class="sxs-lookup"><span data-stu-id="76e71-221">Asset management</span></span></li>
            <li><span data-ttu-id="76e71-222">펀드 관리</span><span class="sxs-lookup"><span data-stu-id="76e71-222">Fund management</span></span></li>
            <li><span data-ttu-id="76e71-223">주문 관리</span><span class="sxs-lookup"><span data-stu-id="76e71-223">Order management</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a><span data-ttu-id="76e71-224">문서 데이터베이스</span><span class="sxs-lookup"><span data-stu-id="76e71-224">Document databases</span></span>

<table>
<tr><td><span data-ttu-id="76e71-225"><strong>워크로드</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-225"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-226">범용입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-226">General purpose.</span></span></li>
            <li><span data-ttu-id="76e71-227">삽입 및 업데이트 작업이 빈번하게 이루어집니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-227">Insert and update operations are common.</span></span> <span data-ttu-id="76e71-228">새 레코드의 생성과 기존 데이터의 업데이트가 정기적으로 이루어집니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-228">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="76e71-229">개체 관계형 불일치가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-229">No object-relational impedance mismatch.</span></span> <span data-ttu-id="76e71-230">문서가 응용 프로그램 코드에서 사용되는 개체 구조와 더욱 잘 매칭됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-230">Documents can better match the object structures used in application code.</span></span></li>
            <li><span data-ttu-id="76e71-231">낙관적 동시성이 더욱 일반적으로 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-231">Optimistic concurrency is more commonly used.</span></span></li>
            <li><span data-ttu-id="76e71-232">데이터를 사용하는 응용 프로그램에서 데이터를 수정 및 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-232">Data must be modified and processed by consuming application.</span></span></li>
            <li><span data-ttu-id="76e71-233">데이터의 여러 필드에서 인덱스가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-233">Data requires index on multiple fields.</span></span></li>
            <li><span data-ttu-id="76e71-234">개별 문서가 단일 블록으로 검색되고 쓰기됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-234">Individual documents are retrieved and written as a single block.</span></span></li>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-235"><strong>데이터 형식</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-235"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-236">데이터를 정규화되지 않은 방식으로 관리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-236">Data can be managed in de-normalized way.</span></span></li>
            <li><span data-ttu-id="76e71-237">개별 문서 데이터의 크기가 상대적으로 작습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-237">Size of individual document data is relatively small.</span></span></li>
            <li><span data-ttu-id="76e71-238">각 문서 유형이 자체 스키마를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-238">Each document type can use its own schema.</span></span></li>
            <li><span data-ttu-id="76e71-239">문서에 선택적 필드가 포함될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-239">Documents can include optional fields.</span></span></li>
            <li><span data-ttu-id="76e71-240">문서 데이터가 반정형입니다. 즉, 각 필드의 데이터 형식이 엄격하게 정의되어 있지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-240">Document data is semi-structured, meaning that data types of each field are not strictly defined.</span></span></li>
            <li><span data-ttu-id="76e71-241">데이터 집계가 지원됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-241">Data aggregation is supported.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-242"><strong>예</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-242"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-243">상품 카탈로그</span><span class="sxs-lookup"><span data-stu-id="76e71-243">Product catalog</span></span></li>
            <li><span data-ttu-id="76e71-244">사용자 계정</span><span class="sxs-lookup"><span data-stu-id="76e71-244">User accounts</span></span></li>
            <li><span data-ttu-id="76e71-245">제품 구성 정보</span><span class="sxs-lookup"><span data-stu-id="76e71-245">Bill of materials</span></span></li>
            <li><span data-ttu-id="76e71-246">개인 설정</span><span class="sxs-lookup"><span data-stu-id="76e71-246">Personalization</span></span></li>
            <li><span data-ttu-id="76e71-247">콘텐츠 관리</span><span class="sxs-lookup"><span data-stu-id="76e71-247">Content management</span></span></li>
            <li><span data-ttu-id="76e71-248">작업 데이터</span><span class="sxs-lookup"><span data-stu-id="76e71-248">Operations data</span></span></li>
            <li><span data-ttu-id="76e71-249">재고 관리</span><span class="sxs-lookup"><span data-stu-id="76e71-249">Inventory management</span></span></li>
            <li><span data-ttu-id="76e71-250">트랜잭션 기록 데이터</span><span class="sxs-lookup"><span data-stu-id="76e71-250">Transaction history data</span></span></li>
            <li><span data-ttu-id="76e71-251">다른 NoSQL 저장소에 대한 구체화된 뷰.</span><span class="sxs-lookup"><span data-stu-id="76e71-251">Materialized view of other NoSQL stores.</span></span> <span data-ttu-id="76e71-252">파일/BLOB 인덱스 작업 대체.</span><span class="sxs-lookup"><span data-stu-id="76e71-252">Replaces file/BLOB indexing.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a><span data-ttu-id="76e71-253">키/값 저장소</span><span class="sxs-lookup"><span data-stu-id="76e71-253">Key/value stores</span></span>

<table>
<tr><td><span data-ttu-id="76e71-254"><strong>워크로드</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-254"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-255">데이터가 마치 사전처럼 단일 ID 키를 사용하여 식별 및 액세스됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-255">Data is identified and accessed using a single ID key, like a dictionary.</span></span></li>
            <li><span data-ttu-id="76e71-256">대규모로 확장 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-256">Massively scalable.</span></span></li>
            <li><span data-ttu-id="76e71-257">조인, 잠금 또는 공용 구조체가 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-257">No joins, lock, or unions are required.</span></span></li>
            <li><span data-ttu-id="76e71-258">집계 메커니즘이 사용되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-258">No aggregation mechanisms are used.</span></span></li>
            <li><span data-ttu-id="76e71-259">일반적으로 보조 인덱스가 사용되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-259">Secondary indexes are generally not used.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-260"><strong>데이터 형식</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-260"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-261">데이터의 크기가 큰 편입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-261">Data size tends to be large.</span></span></li>
            <li><span data-ttu-id="76e71-262">각 키는 관리되지 않는 데이터 BLOB인 하나의 값에 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-262">Each key is associated with a single value, which is an unmanaged data BLOB.</span></span></li>
            <li><span data-ttu-id="76e71-263">스키마 적용이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-263">There is no schema enforcement.</span></span></li>
            <li><span data-ttu-id="76e71-264">엔터티 사이에 관계가 존재하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-264">No relationships between entities.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-265"><strong>예</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-265"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-266">데이터 캐싱</span><span class="sxs-lookup"><span data-stu-id="76e71-266">Data caching</span></span></li>
            <li><span data-ttu-id="76e71-267">세션 관리</span><span class="sxs-lookup"><span data-stu-id="76e71-267">Session management</span></span></li>
            <li><span data-ttu-id="76e71-268">사용자 기본 설정 및 프로필 관리</span><span class="sxs-lookup"><span data-stu-id="76e71-268">User preference and profile management</span></span></li>
            <li><span data-ttu-id="76e71-269">상품 권장 사항 및 광고 게재</span><span class="sxs-lookup"><span data-stu-id="76e71-269">Product recommendation and ad serving</span></span></li>
            <li><span data-ttu-id="76e71-270">사전</span><span class="sxs-lookup"><span data-stu-id="76e71-270">Dictionaries</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a><span data-ttu-id="76e71-271">그래프 데이터베이스</span><span class="sxs-lookup"><span data-stu-id="76e71-271">Graph databases</span></span>

<table>
<tr><td><span data-ttu-id="76e71-272"><strong>워크로드</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-272"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-273">데이터 항목 사이의 관계가 여러 개의 홉이 존재하는 등 매우 복잡합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-273">The relationships between data items are very complex, involving many hops between related data items.</span></span></li>
            <li><span data-ttu-id="76e71-274">데이터 항목 사이의 관계가 동적이고 시간에 따라 변화합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-274">The relationship between data items are dynamic and change over time.</span></span></li>
            <li><span data-ttu-id="76e71-275">개체 사이의 관계가 최고의 리소스입니다. 즉, 탐색을 위해 외부 키와 조인이 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-275">Relationships between objects are first-class citizens, without requiring foreign-keys and joins to traverse.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-276"><strong>데이터 형식</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-276"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-277">데이터가 노드와 관계로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-277">Data is comprised of nodes and relationships.</span></span></li>
            <li><span data-ttu-id="76e71-278">노드는 테이블 행이나 JSON 문서와 비슷합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-278">Nodes are similar to table rows or JSON documents.</span></span></li>
            <li><span data-ttu-id="76e71-279">관계는 노드만큼 중요하며, 쿼리 언어에서 직접적으로 노출됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-279">Relationships are just as important as nodes, and are exposed directly in the query language.</span></span></li>
            <li><span data-ttu-id="76e71-280">여러 개의 전화번호를 갖는 사람과 같은 복합 개체는 크기가 작은 별도의 노드로 분할되어 탐색 가능한 관계와 결합됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-280">Composite objects, such as a person with multiple phone numbers, tend to be broken into separate, smaller nodes, combined with traversable relationships</span></span> </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-281"><strong>예</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-281"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-282">조직도</span><span class="sxs-lookup"><span data-stu-id="76e71-282">Organization charts</span></span></li>
            <li><span data-ttu-id="76e71-283">소셜 그래프</span><span class="sxs-lookup"><span data-stu-id="76e71-283">Social graphs</span></span></li>
            <li><span data-ttu-id="76e71-284">부정 행위 감지</span><span class="sxs-lookup"><span data-stu-id="76e71-284">Fraud detection</span></span></li>
            <li><span data-ttu-id="76e71-285">분석</span><span class="sxs-lookup"><span data-stu-id="76e71-285">Analytics</span></span></li>
            <li><span data-ttu-id="76e71-286">추천 엔진</span><span class="sxs-lookup"><span data-stu-id="76e71-286">Recommendation engines</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a><span data-ttu-id="76e71-287">열 패밀리 데이터베이스</span><span class="sxs-lookup"><span data-stu-id="76e71-287">Column-family databases</span></span>

<table>
<tr><td><span data-ttu-id="76e71-288"><strong>워크로드</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-288"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-289">대부분의 열 패밀리 데이터베이스는 쓰기 작업을 매우 빠르게 수행합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-289">Most column-family databases perform write operations extremely quickly.</span></span></li>
            <li><span data-ttu-id="76e71-290">업데이트 및 삭제 작업은 드물게 이루어집니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-290">Update and delete operations are rare.</span></span></li>
            <li><span data-ttu-id="76e71-291">높은 처리량과 낮은 대기 시간을 갖는 액세스를 제공하도록 설계되었습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-291">Designed to provide high throughput and low-latency access.</span></span></li>
            <li><span data-ttu-id="76e71-292">대규모 레코드에 속한 특정 필드 집합에 대한 간편한 쿼리 액세스를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-292">Supports easy query access to a particular set of fields within a much larger record.</span></span></li>
            <li><span data-ttu-id="76e71-293">대규모로 확장 가능합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-293">Massively scalable.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-294"><strong>데이터 형식</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-294"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-295">데이터가 하나의 키 열과 하나 이상의 열 패밀리로 이루어진 테이블에 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-295">Data is stored in tables consisting of a key column and one or more column families.</span></span></li>
            <li><span data-ttu-id="76e71-296">구체적인 열은 개별 행에 따라 달라질 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-296">Specific columns can vary by individual rows.</span></span></li>
            <li><span data-ttu-id="76e71-297">개별 셀에는 get 및 put 명령을 사용하여 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-297">Individual cells are accessed via get and put commands</span></span></li>
            <li><span data-ttu-id="76e71-298">scan 명령을 사용하면 여러 개의 행이 반환됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-298">Multiple rows are returned using a scan command.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-299"><strong>예</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-299"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-300">권장 사항</span><span class="sxs-lookup"><span data-stu-id="76e71-300">Recommendations</span></span></li>
            <li><span data-ttu-id="76e71-301">개인 설정</span><span class="sxs-lookup"><span data-stu-id="76e71-301">Personalization</span></span></li>
            <li><span data-ttu-id="76e71-302">센서 데이터</span><span class="sxs-lookup"><span data-stu-id="76e71-302">Sensor data</span></span></li>
            <li><span data-ttu-id="76e71-303">원격 분석</span><span class="sxs-lookup"><span data-stu-id="76e71-303">Telemetry</span></span></li>
            <li><span data-ttu-id="76e71-304">메시징</span><span class="sxs-lookup"><span data-stu-id="76e71-304">Messaging</span></span></li>
            <li><span data-ttu-id="76e71-305">소셜 미디어 분석</span><span class="sxs-lookup"><span data-stu-id="76e71-305">Social media analytics</span></span></li>
            <li><span data-ttu-id="76e71-306">웹 분석</span><span class="sxs-lookup"><span data-stu-id="76e71-306">Web analytics</span></span></li>
            <li><span data-ttu-id="76e71-307">활동 모니터링</span><span class="sxs-lookup"><span data-stu-id="76e71-307">Activity monitoring</span></span></li>
            <li><span data-ttu-id="76e71-308">날씨 및 기타 시계열 데이터</span><span class="sxs-lookup"><span data-stu-id="76e71-308">Weather and other time-series data</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a><span data-ttu-id="76e71-309">검색 엔진 데이터베이스</span><span class="sxs-lookup"><span data-stu-id="76e71-309">Search engine databases</span></span>

<table>
<tr><td><span data-ttu-id="76e71-310"><strong>워크로드</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-310"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-311">여러 개의 원본과 서비스로부터 수집된 데이터에 대한 인덱스 작업이 수행됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-311">Indexing data from multiple sources and services.</span></span></li>
            <li><span data-ttu-id="76e71-312">쿼리가 임시이며 복잡한 경우가 많습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-312">Queries are ad-hoc and can be complex.</span></span></li>
            <li><span data-ttu-id="76e71-313">집계가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-313">Requires aggregation.</span></span></li>
            <li><span data-ttu-id="76e71-314">전체 텍스트 검색이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-314">Full text search is required.</span></span></li>
            <li><span data-ttu-id="76e71-315">임시 셀프 서비스 쿼리가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-315">Ad hoc self-service query is required.</span></span></li>
            <li><span data-ttu-id="76e71-316">모든 필드에 인덱스가 있는 데이터 분석이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-316">Data analysis with index on all fields is required.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-317"><strong>데이터 형식</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-317"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-318">반정형 또는 비정형</span><span class="sxs-lookup"><span data-stu-id="76e71-318">Semi-structured or unstructured</span></span></li>
            <li><span data-ttu-id="76e71-319">텍스트</span><span class="sxs-lookup"><span data-stu-id="76e71-319">Text</span></span></li>
            <li><span data-ttu-id="76e71-320">정형 데이터에 대한 참조가 있는 텍스트</span><span class="sxs-lookup"><span data-stu-id="76e71-320">Text with reference to structured data</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-321"><strong>예</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-321"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-322">상품 카탈로그</span><span class="sxs-lookup"><span data-stu-id="76e71-322">Product catalogs</span></span></li>
            <li><span data-ttu-id="76e71-323">사이트 검색</span><span class="sxs-lookup"><span data-stu-id="76e71-323">Site search</span></span></li>
            <li><span data-ttu-id="76e71-324">로깅</span><span class="sxs-lookup"><span data-stu-id="76e71-324">Logging</span></span></li>
            <li><span data-ttu-id="76e71-325">분석</span><span class="sxs-lookup"><span data-stu-id="76e71-325">Analytics</span></span></li>
            <li><span data-ttu-id="76e71-326">쇼핑 사이트</span><span class="sxs-lookup"><span data-stu-id="76e71-326">Shopping sites</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a><span data-ttu-id="76e71-327">데이터 웨어하우스</span><span class="sxs-lookup"><span data-stu-id="76e71-327">Data warehouse</span></span>

<table>
<tr><td><span data-ttu-id="76e71-328"><strong>워크로드</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-328"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-329">데이터 분석</span><span class="sxs-lookup"><span data-stu-id="76e71-329">Data analytics</span></span></li>
            <li><span data-ttu-id="76e71-330">엔터프라이즈 BI</span><span class="sxs-lookup"><span data-stu-id="76e71-330">Enterprise BI</span></span>   </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-331"><strong>데이터 형식</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-331"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-332">여러 원본으로부터 수집된 기록 데이터입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-332">Historical data from multiple sources.</span></span></li>
            <li><span data-ttu-id="76e71-333">일반적으로 팩트와 차원 테이블로 구성된 &quot;스타&quot; 또는 &quot;눈송이&quot; 스키마에서 비정규화되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-333">Usually denormalized in a &quot;star&quot; or &quot;snowflake&quot; schema, consisting of fact and dimension tables.</span></span></li>
            <li><span data-ttu-id="76e71-334">일반적으로 스케줄링에 따라 새로운 데이터가 로드됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-334">Usually loaded with new data on a scheduled basis.</span></span></li>
            <li><span data-ttu-id="76e71-335">일반적으로 차원 테이블에는 <em>느린 변경 차원</em>이라 불리는 엔터티의 복수의 기록 버전이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-335">Dimension tables often include multiple historic versions of an entity, referred to as a <em>slowly changing dimension</em>.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-336"><strong>예</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-336"><strong>Examples</strong></span></span></td>
    <td><span data-ttu-id="76e71-337">분석 모델, 보고서 및 대시보드에 데이터를 제공하는 엔터프라이즈 데이터 웨어하우스.</span><span class="sxs-lookup"><span data-stu-id="76e71-337">An enterprise data warehouse that provides data for analytical models, reports, and dashboards.</span></span>
    </td>
</tr>
</table>


## <a name="time-series-databases"></a><span data-ttu-id="76e71-338">시계열 데이터베이스</span><span class="sxs-lookup"><span data-stu-id="76e71-338">Time series databases</span></span>

<table>
<tr><td><span data-ttu-id="76e71-339"><strong>워크로드</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-339"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-340">작업의 대다수(95~99%)가 쓰기 작업입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-340">An overwhelming proportion of operations (95-99%) are writes.</span></span></li>
            <li><span data-ttu-id="76e71-341">레코드는 일반적으로 시간순으로 순차적으로 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-341">Records are generally appended sequentially in time order.</span></span></li>
            <li><span data-ttu-id="76e71-342">업데이트는 드물게 이루어집니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-342">Updates are rare.</span></span></li>
            <li><span data-ttu-id="76e71-343">삭제는 연속 블록 또는 레코드에 대해 대량으로 이루어집니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-343">Deletes occur in bulk, and are made to contiguous blocks or records.</span></span></li>
            <li><span data-ttu-id="76e71-344">읽기 요청이 가용 메모리보다 클 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-344">Read requests can be larger than available memory.</span></span></li>
            <li><span data-ttu-id="76e71-345">흔히 복수의 읽기가 동시에 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-345">It&#39;s common for multiple reads to occur simultaneously.</span></span></li>
            <li><span data-ttu-id="76e71-346">오름차순 또는 내림차순 시간순으로 순차적으로 데이터 읽기가 수행됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-346">Data is read sequentially in either ascending or descending time order.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-347"><strong>데이터 형식</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-347"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-348">기본 키 및 정렬 메커니즘으로 사용되는 타임스탬프입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-348">A time stamp that is used as the primary key and sorting mechanism.</span></span></li>
            <li><span data-ttu-id="76e71-349">항목 또는 항목이 무엇을 나타내는지에 대한 설명으로부터의 측정입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-349">Measurements from the entry or descriptions of what the entry represents.</span></span></li>
            <li><span data-ttu-id="76e71-350">유형, 원본 및 항목에 대한 그 밖의 정보 등 추가 정보를 정의하는 태그입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-350">Tags that define additional information about the type, origin, and other information about the entry.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-351"><strong>예</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-351"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-352">모니터링 및 이벤트 원격 분석.</span><span class="sxs-lookup"><span data-stu-id="76e71-352">Monitoring and event telemetry.</span></span></li>
            <li><span data-ttu-id="76e71-353">센서 또는 기타 IoT 데이터.</span><span class="sxs-lookup"><span data-stu-id="76e71-353">Sensor or other IoT data.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a><span data-ttu-id="76e71-354">개체 저장소</span><span class="sxs-lookup"><span data-stu-id="76e71-354">Object storage</span></span>

<table>
<tr><td><span data-ttu-id="76e71-355"><strong>워크로드</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-355"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-356">키로 식별됩니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-356">Identified by key.</span></span></li>
            <li><span data-ttu-id="76e71-357">개체에 공개 또는 비공개로 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-357">Objects may be publicly or privately accessible.</span></span></li>
            <li><span data-ttu-id="76e71-358">콘텐츠는 일반적으로 스프레드시트, 이미지 또는 비디오 파일과 같은 자산입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-358">Content is typically an asset such as a spreadsheet, image, or video file.</span></span></li>
            <li><span data-ttu-id="76e71-359">콘텐츠는 내구성이 높은(영구적) 상태이고 모든 응용 프로그램 계층 또는 가상 머신의 외부에 존재해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-359">Content must be durable (persistent), and external to any application tier or virtual machine.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-360"><strong>데이터 형식</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-360"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-361">데이터 크기가 큽니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-361">Data size is large.</span></span></li>
            <li><span data-ttu-id="76e71-362">Blob 데이터입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-362">Blob data.</span></span></li>
            <li><span data-ttu-id="76e71-363">값은 불투명합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-363">Value is opaque.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-364"><strong>예</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-364"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-365">이미지, 비디오, 사무 문서, PDF</span><span class="sxs-lookup"><span data-stu-id="76e71-365">Images, videos, office documents, PDFs</span></span></li>
            <li><span data-ttu-id="76e71-366">CSS, 스크립트, CSV</span><span class="sxs-lookup"><span data-stu-id="76e71-366">CSS, Scripts, CSV</span></span></li>
            <li><span data-ttu-id="76e71-367">정적 HTML, JSON</span><span class="sxs-lookup"><span data-stu-id="76e71-367">Static HTML, JSON</span></span></li>
            <li><span data-ttu-id="76e71-368">로그 및 감사 파일</span><span class="sxs-lookup"><span data-stu-id="76e71-368">Log and audit files</span></span></li>
            <li><span data-ttu-id="76e71-369">데이터베이스 백업</span><span class="sxs-lookup"><span data-stu-id="76e71-369">Database backups</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a><span data-ttu-id="76e71-370">공유 파일</span><span class="sxs-lookup"><span data-stu-id="76e71-370">Shared files</span></span>

<table>
<tr><td><span data-ttu-id="76e71-371"><strong>워크로드</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-371"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-372">파일 시스템과 상호 작용하는 기존 앱에서의 마이그레이션입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-372">Migration from existing apps that interact with the file system.</span></span></li>
            <li><span data-ttu-id="76e71-373">SMB 인터페이스가 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-373">Requires SMB interface.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-374"><strong>데이터 형식</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-374"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-375">계층 구조 폴더 집합에 포함된 파일입니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-375">Files in a hierarchical set of folders.</span></span></li>
            <li><span data-ttu-id="76e71-376">표준 I/O 라이브러리를 사용하여 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="76e71-376">Accessible with standard I/O libraries.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="76e71-377"><strong>예</strong></span><span class="sxs-lookup"><span data-stu-id="76e71-377"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="76e71-378">레거시 파일</span><span class="sxs-lookup"><span data-stu-id="76e71-378">Legacy files</span></span></li>
            <li><span data-ttu-id="76e71-379">여러 VM 또는 앱 인스턴스로부터 액세스할 수 있는 공유 콘텐츠</span><span class="sxs-lookup"><span data-stu-id="76e71-379">Shared content accessible among a number of VMs or app instances</span></span></li>
        </ul>
    </td>
</tr>
</table>
