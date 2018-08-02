---
title: 서비스 관련 재시도 지침
description: 재시도 메커니즘 설정에 대한 서비스 관련 지침입니다.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 72dfb59c3357c5f14806a33ef5f6cdd3e7937915
ms.sourcegitcommit: 8b5fc0d0d735793b87677610b747f54301dcb014
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/29/2018
ms.locfileid: "39334167"
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="865dc-103">특정 서비스에 대한 다시 시도 지침</span><span class="sxs-lookup"><span data-stu-id="865dc-103">Retry guidance for specific services</span></span>

<span data-ttu-id="865dc-104">대부분의 Azure 서비스 및 클라이언트 SDK는 재시도 메커니즘을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="865dc-105">그러나 서비스마다 특성 및 요구 사항이 다르기 때문에 이러한 메커니즘을 서로 다르므로 각 재시도 메커니즘은 특정 서비스에 맞게 조정됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="865dc-106">이 가이드에서는 대부분의 Azure 서비스에 대한 재시도 메커니즘 기능을 요약하고 해당 서비스에 대한 재시도 메커니즘을 사용, 적용 또는 확장하는 데 도움이 되는 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="865dc-107">일시적인 오류 처리, 서비스와 리소스에 대해 연결 및 작업 재시도에 대한 일반 지침은 [재시도 지침](./transient-faults.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="865dc-108">다음 표에는 이 지침에서 설명하는 Azure 서비스에 대한 재시도 기능이 요약되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="865dc-109">**서비스**</span><span class="sxs-lookup"><span data-stu-id="865dc-109">**Service**</span></span> | <span data-ttu-id="865dc-110">**재시도 기능**</span><span class="sxs-lookup"><span data-stu-id="865dc-110">**Retry capabilities**</span></span> | <span data-ttu-id="865dc-111">**정책 구성**</span><span class="sxs-lookup"><span data-stu-id="865dc-111">**Policy configuration**</span></span> | <span data-ttu-id="865dc-112">**범위**</span><span class="sxs-lookup"><span data-stu-id="865dc-112">**Scope**</span></span> | <span data-ttu-id="865dc-113">**원격 분석 기능**</span><span class="sxs-lookup"><span data-stu-id="865dc-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="865dc-114">**[Azure Active Directory](#azure-active-directory)**</span><span class="sxs-lookup"><span data-stu-id="865dc-114">**[Azure Active Directory](#azure-active-directory)**</span></span> |<span data-ttu-id="865dc-115">ADAL 라이브러리에서 기본</span><span class="sxs-lookup"><span data-stu-id="865dc-115">Native in ADAL library</span></span> |<span data-ttu-id="865dc-116">ADAL 라이브러리에 포함</span><span class="sxs-lookup"><span data-stu-id="865dc-116">Embeded into ADAL library</span></span> |<span data-ttu-id="865dc-117">내부</span><span class="sxs-lookup"><span data-stu-id="865dc-117">Internal</span></span> |<span data-ttu-id="865dc-118">없음</span><span class="sxs-lookup"><span data-stu-id="865dc-118">None</span></span> |
| <span data-ttu-id="865dc-119">**[Cosmos DB](#cosmos-db)**</span><span class="sxs-lookup"><span data-stu-id="865dc-119">**[Cosmos DB](#cosmos-db)**</span></span> |<span data-ttu-id="865dc-120">서비스의 네이티브</span><span class="sxs-lookup"><span data-stu-id="865dc-120">Native in service</span></span> |<span data-ttu-id="865dc-121">구성할 수 없음</span><span class="sxs-lookup"><span data-stu-id="865dc-121">Non-configurable</span></span> |<span data-ttu-id="865dc-122">전역</span><span class="sxs-lookup"><span data-stu-id="865dc-122">Global</span></span> |<span data-ttu-id="865dc-123">TraceSource</span><span class="sxs-lookup"><span data-stu-id="865dc-123">TraceSource</span></span> |
| <span data-ttu-id="865dc-124">**[Event Hubs](#event-hubs)**</span><span class="sxs-lookup"><span data-stu-id="865dc-124">**[Event Hubs](#event-hubs)**</span></span> |<span data-ttu-id="865dc-125">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="865dc-125">Native in client</span></span> |<span data-ttu-id="865dc-126">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="865dc-126">Programmatic</span></span> |<span data-ttu-id="865dc-127">클라이언트</span><span class="sxs-lookup"><span data-stu-id="865dc-127">Client</span></span> |<span data-ttu-id="865dc-128">없음</span><span class="sxs-lookup"><span data-stu-id="865dc-128">None</span></span> |
| <span data-ttu-id="865dc-129">**[Redis Cache](#azure-redis-cache)**</span><span class="sxs-lookup"><span data-stu-id="865dc-129">**[Redis Cache](#azure-redis-cache)**</span></span> |<span data-ttu-id="865dc-130">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="865dc-130">Native in client</span></span> |<span data-ttu-id="865dc-131">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="865dc-131">Programmatic</span></span> |<span data-ttu-id="865dc-132">클라이언트</span><span class="sxs-lookup"><span data-stu-id="865dc-132">Client</span></span> |<span data-ttu-id="865dc-133">TextWriter</span><span class="sxs-lookup"><span data-stu-id="865dc-133">TextWriter</span></span> |
| <span data-ttu-id="865dc-134">**[Search](#azure-search)**</span><span class="sxs-lookup"><span data-stu-id="865dc-134">**[Search](#azure-search)**</span></span> |<span data-ttu-id="865dc-135">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="865dc-135">Native in client</span></span> |<span data-ttu-id="865dc-136">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="865dc-136">Programmatic</span></span> |<span data-ttu-id="865dc-137">클라이언트</span><span class="sxs-lookup"><span data-stu-id="865dc-137">Client</span></span> |<span data-ttu-id="865dc-138">ETW 또는 사용자 지정</span><span class="sxs-lookup"><span data-stu-id="865dc-138">ETW or Custom</span></span> |
| <span data-ttu-id="865dc-139">**[Service Bus](#service-bus)**</span><span class="sxs-lookup"><span data-stu-id="865dc-139">**[Service Bus](#service-bus)**</span></span> |<span data-ttu-id="865dc-140">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="865dc-140">Native in client</span></span> |<span data-ttu-id="865dc-141">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="865dc-141">Programmatic</span></span> |<span data-ttu-id="865dc-142">네임스페이스 관리자, 메시징 팩터리 및 클라이언트</span><span class="sxs-lookup"><span data-stu-id="865dc-142">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="865dc-143">ETW</span><span class="sxs-lookup"><span data-stu-id="865dc-143">ETW</span></span> |
| <span data-ttu-id="865dc-144">**[Service Fabric](#service-fabric)**</span><span class="sxs-lookup"><span data-stu-id="865dc-144">**[Service Fabric](#service-fabric)**</span></span> |<span data-ttu-id="865dc-145">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="865dc-145">Native in client</span></span> |<span data-ttu-id="865dc-146">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="865dc-146">Programmatic</span></span> |<span data-ttu-id="865dc-147">클라이언트</span><span class="sxs-lookup"><span data-stu-id="865dc-147">Client</span></span> |<span data-ttu-id="865dc-148">없음</span><span class="sxs-lookup"><span data-stu-id="865dc-148">None</span></span> | 
| <span data-ttu-id="865dc-149">**[ADO.NET을 사용하는 SQL Database](#sql-database-using-adonet)**</span><span class="sxs-lookup"><span data-stu-id="865dc-149">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span></span> |[<span data-ttu-id="865dc-150">Polly</span><span class="sxs-lookup"><span data-stu-id="865dc-150">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="865dc-151">선언적 방식 및 프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="865dc-151">Declarative and programmatic</span></span> |<span data-ttu-id="865dc-152">코드의 단일 문 또는 블록</span><span class="sxs-lookup"><span data-stu-id="865dc-152">Single statements or blocks of code</span></span> |<span data-ttu-id="865dc-153">사용자 지정</span><span class="sxs-lookup"><span data-stu-id="865dc-153">Custom</span></span> |
| <span data-ttu-id="865dc-154">**[Entity Framework를 사용하는 SQL Database](#sql-database-using-entity-framework-6)**</span><span class="sxs-lookup"><span data-stu-id="865dc-154">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span></span> |<span data-ttu-id="865dc-155">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="865dc-155">Native in client</span></span> |<span data-ttu-id="865dc-156">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="865dc-156">Programmatic</span></span> |<span data-ttu-id="865dc-157">AppDomain에 따라 전역</span><span class="sxs-lookup"><span data-stu-id="865dc-157">Global per AppDomain</span></span> |<span data-ttu-id="865dc-158">없음</span><span class="sxs-lookup"><span data-stu-id="865dc-158">None</span></span> |
| <span data-ttu-id="865dc-159">**[Entity Framework Core를 사용하는 SQL Database](#sql-database-using-entity-framework-core)**</span><span class="sxs-lookup"><span data-stu-id="865dc-159">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span></span> |<span data-ttu-id="865dc-160">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="865dc-160">Native in client</span></span> |<span data-ttu-id="865dc-161">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="865dc-161">Programmatic</span></span> |<span data-ttu-id="865dc-162">AppDomain에 따라 전역</span><span class="sxs-lookup"><span data-stu-id="865dc-162">Global per AppDomain</span></span> |<span data-ttu-id="865dc-163">없음</span><span class="sxs-lookup"><span data-stu-id="865dc-163">None</span></span> |
| <span data-ttu-id="865dc-164">**[Storage](#azure-storage)**</span><span class="sxs-lookup"><span data-stu-id="865dc-164">**[Storage](#azure-storage)**</span></span> |<span data-ttu-id="865dc-165">클라이언트의 네이티브</span><span class="sxs-lookup"><span data-stu-id="865dc-165">Native in client</span></span> |<span data-ttu-id="865dc-166">프로그래밍 방식</span><span class="sxs-lookup"><span data-stu-id="865dc-166">Programmatic</span></span> |<span data-ttu-id="865dc-167">클라이언트 및 개별 작업</span><span class="sxs-lookup"><span data-stu-id="865dc-167">Client and individual operations</span></span> |<span data-ttu-id="865dc-168">TraceSource</span><span class="sxs-lookup"><span data-stu-id="865dc-168">TraceSource</span></span> |

> [!NOTE]
> <span data-ttu-id="865dc-169">대부분의 Azure 기본 제공 재시도 메커니즘에서는 현재 다른 유형의 오류 또는 예외에 대해 서로 다른 재시도 정책을 적용할 방법이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception.</span></span> <span data-ttu-id="865dc-170">최적의 평균 성능 및 가용성을 제공하는 정책을 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-170">You should configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="865dc-171">정책을 미세 조정하는 한 가지 방법은 로그 파일을 분석하여 발생하는 일시적인 오류의 유형을 확인하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> 

## <a name="azure-active-directory"></a><span data-ttu-id="865dc-172">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="865dc-172">Azure Active Directory</span></span>
<span data-ttu-id="865dc-173">Azure AD(Azure Active Directory)는 핵심 디렉터리 서비스, 고급 ID 관리, 보안 및 응용 프로그램 액세스 관리를 결합하는 포괄적인 ID 및 액세스 관리 클라우드 솔루션입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-173">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="865dc-174">또한 Azure AD는 개발자에게 ID 관리 플랫폼을 제공하여 중앙 집중식 정책 및 규칙에 따라 해당 응용 프로그램에 대한 액세스 제어 권한을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-174">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

> [!NOTE]
> <span data-ttu-id="865dc-175">관리 서비스 ID 엔드포인트에 대한 다시 시도 지침은 [토큰 획득을 위해 Azure VM MSI(관리 서비스 ID)를 사용하는 방법](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling)을 참조합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-175">For retry guidance on Managed Service Identity endpoints, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="865dc-176">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="865dc-176">Retry mechanism</span></span>
<span data-ttu-id="865dc-177">ADAL(Active Directory 인증 라이브러리)의 Azure Active Directory에 대한 기본 제공 재시도 메커니즘은 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-177">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="865dc-178">예기치 않은 잠김을 방지하기 위해, 타사 라이브러리와 응용 프로그램 코드가 실패한 연결을 재시도하지 **않고** ADAL이 재시도를 처리하게 하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-178">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="865dc-179">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="865dc-179">Retry usage guidance</span></span>
<span data-ttu-id="865dc-180">Azure Active Directory를 사용하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-180">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="865dc-181">가능하다면 재시도에 ADAL 라이브러리와 기본 제공 지원을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-181">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="865dc-182">Azure Active Directory용 REST API를 사용하는 경우, 결과 코드가 429(너무 많은 요청)이거나 5xx 범위의 오류인 경우 작업을 재시도합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-182">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="865dc-183">다른 오류의 경우에는 재시도하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-183">Do not retry for any other errors.</span></span>
* <span data-ttu-id="865dc-184">Azure Active Directory의 일괄 처리 시나리오에는 지수 백오프 정책을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-184">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="865dc-185">재시도 작업에 대해 다음 설정을 사용하여 시작하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-185">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="865dc-186">이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-186">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="865dc-187">**컨텍스트**</span><span class="sxs-lookup"><span data-stu-id="865dc-187">**Context**</span></span> | <span data-ttu-id="865dc-188">**샘플 대상 E2E<br />최대 대기 시간**</span><span class="sxs-lookup"><span data-stu-id="865dc-188">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="865dc-189">**재시도 전략**</span><span class="sxs-lookup"><span data-stu-id="865dc-189">**Retry strategy**</span></span> | <span data-ttu-id="865dc-190">**설정**</span><span class="sxs-lookup"><span data-stu-id="865dc-190">**Settings**</span></span> | <span data-ttu-id="865dc-191">**값**</span><span class="sxs-lookup"><span data-stu-id="865dc-191">**Values**</span></span> | <span data-ttu-id="865dc-192">**작동 방법**</span><span class="sxs-lookup"><span data-stu-id="865dc-192">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="865dc-193">대화형, UI</span><span class="sxs-lookup"><span data-stu-id="865dc-193">Interactive, UI,</span></span><br /><span data-ttu-id="865dc-194">또는 포그라운드</span><span class="sxs-lookup"><span data-stu-id="865dc-194">or foreground</span></span> |<span data-ttu-id="865dc-195">2초</span><span class="sxs-lookup"><span data-stu-id="865dc-195">2 sec</span></span> |<span data-ttu-id="865dc-196">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="865dc-196">FixedInterval</span></span> |<span data-ttu-id="865dc-197">재시도 횟수</span><span class="sxs-lookup"><span data-stu-id="865dc-197">Retry count</span></span><br /><span data-ttu-id="865dc-198">재시도 간격</span><span class="sxs-lookup"><span data-stu-id="865dc-198">Retry interval</span></span><br /><span data-ttu-id="865dc-199">첫 번째 빠른 재시도</span><span class="sxs-lookup"><span data-stu-id="865dc-199">First fast retry</span></span> |<span data-ttu-id="865dc-200">3</span><span class="sxs-lookup"><span data-stu-id="865dc-200">3</span></span><br /><span data-ttu-id="865dc-201">500ms</span><span class="sxs-lookup"><span data-stu-id="865dc-201">500 ms</span></span><br /><span data-ttu-id="865dc-202">true</span><span class="sxs-lookup"><span data-stu-id="865dc-202">true</span></span> |<span data-ttu-id="865dc-203">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-203">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="865dc-204">시도 2 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-204">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="865dc-205">시도 3 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-205">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="865dc-206">백그라운드 또는</span><span class="sxs-lookup"><span data-stu-id="865dc-206">Background or</span></span><br /><span data-ttu-id="865dc-207">일괄 처리</span><span class="sxs-lookup"><span data-stu-id="865dc-207">batch</span></span> |<span data-ttu-id="865dc-208">60초</span><span class="sxs-lookup"><span data-stu-id="865dc-208">60 sec</span></span> |<span data-ttu-id="865dc-209">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-209">ExponentialBackoff</span></span> |<span data-ttu-id="865dc-210">재시도 횟수</span><span class="sxs-lookup"><span data-stu-id="865dc-210">Retry count</span></span><br /><span data-ttu-id="865dc-211">최소 백오프</span><span class="sxs-lookup"><span data-stu-id="865dc-211">Min back-off</span></span><br /><span data-ttu-id="865dc-212">최대 백오프</span><span class="sxs-lookup"><span data-stu-id="865dc-212">Max back-off</span></span><br /><span data-ttu-id="865dc-213">델타 백오프</span><span class="sxs-lookup"><span data-stu-id="865dc-213">Delta back-off</span></span><br /><span data-ttu-id="865dc-214">첫 번째 빠른 재시도</span><span class="sxs-lookup"><span data-stu-id="865dc-214">First fast retry</span></span> |<span data-ttu-id="865dc-215">5</span><span class="sxs-lookup"><span data-stu-id="865dc-215">5</span></span><br /><span data-ttu-id="865dc-216">0초</span><span class="sxs-lookup"><span data-stu-id="865dc-216">0 sec</span></span><br /><span data-ttu-id="865dc-217">60초</span><span class="sxs-lookup"><span data-stu-id="865dc-217">60 sec</span></span><br /><span data-ttu-id="865dc-218">2초</span><span class="sxs-lookup"><span data-stu-id="865dc-218">2 sec</span></span><br /><span data-ttu-id="865dc-219">false</span><span class="sxs-lookup"><span data-stu-id="865dc-219">false</span></span> |<span data-ttu-id="865dc-220">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-220">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="865dc-221">시도 2 - ~2초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-221">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="865dc-222">시도 3 - ~6초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-222">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="865dc-223">시도 4 - ~14초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-223">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="865dc-224">시도 5 - ~30초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-224">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="865dc-225">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="865dc-225">More information</span></span>
* <span data-ttu-id="865dc-226">[Azure Active Directory 인증 라이브러리][adal]</span><span class="sxs-lookup"><span data-stu-id="865dc-226">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="865dc-227">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="865dc-227">Cosmos DB</span></span>

<span data-ttu-id="865dc-228">Cosmos DB는 스키마 없는 JSON 데이터를 지원하는 완전 관리 다중 model 데이터베이스입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-228">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="865dc-229">DocumentDB는 구성 가능하고 안정적인 성능, 네이티브 JavaScript 트랜잭션 처리를 제공하며 탄력적인 확장성으로 클라우드에 대해 구축됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-229">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="865dc-230">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="865dc-230">Retry mechanism</span></span>
<span data-ttu-id="865dc-231">`DocumentClient` 클래스는 실패 횟수를 자동으로 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-231">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="865dc-232">재시도 횟수와 최대 대기 시간을 설정하려면 [ConnectionPolicy.RetryOptions]을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-232">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="865dc-233">클라이언트에서 발생시키는 예외는 재시도 정책 시도 횟수를 초과하거나 일시적인 오류가 아닙니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-233">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="865dc-234">Cosmos DB에서 클라이언트를 제한하는 경우 HTTP 429 오류를 반환합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-234">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="865dc-235">`DocumentClientException`에서 상태 코드를 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-235">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="865dc-236">정책 구성</span><span class="sxs-lookup"><span data-stu-id="865dc-236">Policy configuration</span></span>
<span data-ttu-id="865dc-237">다음 표에서는 `RetryOptions` 클래스의 기본 설정을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-237">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="865dc-238">설정</span><span class="sxs-lookup"><span data-stu-id="865dc-238">Setting</span></span> | <span data-ttu-id="865dc-239">기본값</span><span class="sxs-lookup"><span data-stu-id="865dc-239">Default value</span></span> | <span data-ttu-id="865dc-240">설명</span><span class="sxs-lookup"><span data-stu-id="865dc-240">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="865dc-241">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="865dc-241">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="865dc-242">9</span><span class="sxs-lookup"><span data-stu-id="865dc-242">9</span></span> |<span data-ttu-id="865dc-243">Cosmos DB가 클라이언트에서 속도 제한을 적용하기 때문에 요청에 실패하면 다시 시도하는 최대 수입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-243">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="865dc-244">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="865dc-244">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="865dc-245">30</span><span class="sxs-lookup"><span data-stu-id="865dc-245">30</span></span> |<span data-ttu-id="865dc-246">다시 시도하는 최대 시간(초)입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-246">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="865dc-247">예</span><span class="sxs-lookup"><span data-stu-id="865dc-247">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="865dc-248">원격 분석</span><span class="sxs-lookup"><span data-stu-id="865dc-248">Telemetry</span></span>
<span data-ttu-id="865dc-249">재시도 횟수는 .NET **TraceSource**를 통해 구조화되지 않은 추적 메시지로 기록됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-249">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="865dc-250">이벤트를 캡처하여 적합한 대상 로그에 기록하려면 **TraceListener**를 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-250">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="865dc-251">예를 들어 App.config 파일에 다음을 추가하는 경우 추적은 실행 파일과 동일한 위치에 있는 텍스트 파일에 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-251">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```xml
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

## <a name="event-hubs"></a><span data-ttu-id="865dc-252">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="865dc-252">Event Hubs</span></span>

<span data-ttu-id="865dc-253">Azure Event Hubs는 수백만 개의 이벤트를 수집, 변환 및 저장하는 하이퍼스케일(hyper-scale) 원격 분석 수집 서비스입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-253">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="865dc-254">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="865dc-254">Retry mechanism</span></span>
<span data-ttu-id="865dc-255">Azure Event Hubs 클라이언트 라이브러리의 재시도 동작은 `EventHubClient` 클래스의 `RetryPolicy` 속성에서 제어됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-255">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="865dc-256">기본 정책에서는 Azure Event Hub가 일시적 `EventHubsException` 또는 `OperationCanceledException`을 반환할 때 지수 백오프를 통해 재시도합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-256">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="865dc-257">예</span><span class="sxs-lookup"><span data-stu-id="865dc-257">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="865dc-258">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="865dc-258">More information</span></span>
[<span data-ttu-id="865dc-259">Azure Event Hubs용 .NET 표준 클라이언트 라이브러리</span><span class="sxs-lookup"><span data-stu-id="865dc-259"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="azure-redis-cache"></a><span data-ttu-id="865dc-260">Azure Redis 캐시(영문)</span><span class="sxs-lookup"><span data-stu-id="865dc-260">Azure Redis Cache</span></span>
<span data-ttu-id="865dc-261">Azure Redis Cache는 많이 사용되는 오픈 소스 Redis 캐시에 기반한 캐시 서비스로 데이터 액세스 속도가 빠르고 대기 시간이 짧습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-261">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="865dc-262">이 캐시는 Microsoft에서 관리되어 안전하며 Azure의 모든 응용 프로그램에서 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-262">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="865dc-263">이 섹션의 지침은 StackExchange.Redis 클라이언트를 사용하여 캐시에 액세스하는 지침을 기반으로 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-263">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="865dc-264">기타 적합한 클라이언트 목록은 [Redis 웹 사이트](http://redis.io/clients)에서 확인할 수 있으며 클라이언트마다 재시도 메커니즘이 다를 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-264">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="865dc-265">StackExchange.Redis 클라이언트는 단일 연결을 통해 멀티플렉싱을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-265">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="865dc-266">권장되는 사용법은 응용 프로그램 시작 시 클라이언트의 인스턴스를 만들고 캐시에 대한 모든 작업에 이 인스턴스를 사용하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-266">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="865dc-267">따라서 캐시에 한 번만 연결되므로 이 섹션의 모든 지침은 캐시에 액세스하는 각 작업이 아니라 이 초기 연결에 대한 재시도 정책과 관련이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-267">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="865dc-268">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="865dc-268">Retry mechanism</span></span>
<span data-ttu-id="865dc-269">StackExchange.Redis 클라이언트는 다음과 같이 옵션 집합을 통해 구성되는 연결 관리자 클래스를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-269">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="865dc-270">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="865dc-270">**ConnectRetry**.</span></span> <span data-ttu-id="865dc-271">실패한 캐시 연결을 재시도한 횟수입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-271">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="865dc-272">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="865dc-272">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="865dc-273">사용하는 재시도 전략입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-273">The retry strategy to use.</span></span>
- <span data-ttu-id="865dc-274">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="865dc-274">**ConnectTimeout**.</span></span> <span data-ttu-id="865dc-275">최대 대기 시간(밀리초)입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-275">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="865dc-276">정책 구성</span><span class="sxs-lookup"><span data-stu-id="865dc-276">Policy configuration</span></span>
<span data-ttu-id="865dc-277">재시도 정책은 캐시에 연결하기 전에 클라이언트에 대한 옵션을 설정하여 프로그래밍 방식으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-277">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="865dc-278">이 작업은 **ConfigurationOptions** 클래스의 인스턴스를 만들고 해당 속성을 채운 후 **Connect** 메서드에 전달하여 수행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-278">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="865dc-279">기본 제공 클래스는 임의 재시도 간격을 통해 선형(일정) 지연 및 지수 백오프를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-279">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="865dc-280">**IReconnectRetryPolicy** 인터페이스를 구현하여 사용자 지정 재시도 정책을 만들 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-280">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="865dc-281">다음 예제에서는 지수 백오프를 사용하여 재시도 전략을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-281">The following example configures a retry strategy using exponential backoff.</span></span>

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="865dc-282">또는 옵션을 문자열로 지정하고 이를 **Connect** 메서드에 전달할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-282">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="865dc-283">**ReconnectRetryPolicy** 속성은 이 방식으로 설정할 수 없고 코드를 통해서만 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-283">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="865dc-284">캐시에 연결할 때 직접 옵션을 지정할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-284">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="865dc-285">자세한 내용은 StackExchange.Redis 설명서의 [Stack Exchange Redis 구성](https://stackexchange.github.io/StackExchange.Redis/Configuration)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-285">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="865dc-286">다음 표에서는 기본 제공 재시도 정책의 기본 설정을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-286">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="865dc-287">**컨텍스트**</span><span class="sxs-lookup"><span data-stu-id="865dc-287">**Context**</span></span> | <span data-ttu-id="865dc-288">**설정**</span><span class="sxs-lookup"><span data-stu-id="865dc-288">**Setting**</span></span> | <span data-ttu-id="865dc-289">**기본값**</span><span class="sxs-lookup"><span data-stu-id="865dc-289">**Default value**</span></span><br /><span data-ttu-id="865dc-290">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="865dc-290">(v 1.2.2)</span></span> | <span data-ttu-id="865dc-291">**의미**</span><span class="sxs-lookup"><span data-stu-id="865dc-291">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="865dc-292">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="865dc-292">ConfigurationOptions</span></span> |<span data-ttu-id="865dc-293">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="865dc-293">ConnectRetry</span></span><br /><br /><span data-ttu-id="865dc-294">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="865dc-294">ConnectTimeout</span></span><br /><br /><span data-ttu-id="865dc-295">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="865dc-295">SyncTimeout</span></span><br /><br /><span data-ttu-id="865dc-296">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="865dc-296">ReconnectRetryPolicy</span></span> |<span data-ttu-id="865dc-297">3</span><span class="sxs-lookup"><span data-stu-id="865dc-297">3</span></span><br /><br /><span data-ttu-id="865dc-298">최대 5000ms와 SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="865dc-298">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="865dc-299">1000</span><span class="sxs-lookup"><span data-stu-id="865dc-299">1000</span></span><br /><br /><span data-ttu-id="865dc-300">LinearRetry 5000ms</span><span class="sxs-lookup"><span data-stu-id="865dc-300">LinearRetry 5000 ms</span></span> |<span data-ttu-id="865dc-301">초기 연결 작업 중 연결 시도 반복 횟수입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-301">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="865dc-302">연결 작업에 대한 제한 시간(ms)입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-302">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="865dc-303">재시도 사이에 지연은 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-303">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="865dc-304">동기 작업을 허용하는 시간(ms)입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-304">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="865dc-305">5000 밀리초마다 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-305">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="865dc-306">동기 작업의 경우 `SyncTimeout`을 종단 간 대기 시간에 추가할 수 있으나 이 값을 너무 낮게 설정하면 과도한 시간 제한이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-306">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="865dc-307">[Azure Redis Cache 문제를 해결하는 방법][redis-cache-troubleshoot]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-307">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="865dc-308">일반적으로 동기 작업보다는 비동기 작업을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-308">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="865dc-309">자세한 내용은 [파이프라인 및 멀티플렉서](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md)(영문)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-309">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="865dc-310">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="865dc-310">Retry usage guidance</span></span>
<span data-ttu-id="865dc-311">Azure Redis Cache를 사용하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-311">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="865dc-312">StackExchange Redis 클라이언트는 고유한 재시도를 관리하지만 응용 프로그램이 처음 시작될 때 캐시에 대한 연결을 설정할 때만 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-312">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="865dc-313">연결 제한 시간, 재시도 횟수 및 재시도 사이의 시간을 구성하여 이 연결을 설정할 수 있지만 재시도 정책은 캐시에 대한 작업에 적용되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-313">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="865dc-314">많은 수의 재시도 횟수를 사용하는 대신 원래 데이터 소스에 액세스하여 폴백하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-314">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="865dc-315">원격 분석</span><span class="sxs-lookup"><span data-stu-id="865dc-315">Telemetry</span></span>
<span data-ttu-id="865dc-316">**TextWriter**를 사용하여 다른 작업이 아닌 연결에 대한 정보를 수집할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-316">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="865dc-317">생성되는 출력의 예는 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-317">An example of the output this generates is shown below.</span></span>

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a><span data-ttu-id="865dc-318">예</span><span class="sxs-lookup"><span data-stu-id="865dc-318">Examples</span></span>
<span data-ttu-id="865dc-319">다음 코드 예제에서는 StackExchange.Redis 클라이언트를 초기화할 때 재시도 사이에 일정(선형) 지연을 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-319">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="865dc-320">이 예제에서는 **ConfigurationOptions** 인스턴스를 사용하여 구성을 설정하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-320">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="865dc-321">다음 예제에서는 옵션을 문자열로 지정하여 구성을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-321">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="865dc-322">연결 제한 시간은 재시도 간의 지연이 아니라 캐시에 대한 연결을 대기하는 최대 시간입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-322">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="865dc-323">**ReconnectRetryPolicy** 속성은 코드로만 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-323">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="865dc-324">더 많은 예제는 프로젝트 웹 사이트의 [구성](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration)(영문)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-324">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="865dc-325">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="865dc-325">More information</span></span>
* [<span data-ttu-id="865dc-326">Redis 웹 사이트</span><span class="sxs-lookup"><span data-stu-id="865dc-326">Redis website</span></span>](http://redis.io/)

## <a name="azure-search"></a><span data-ttu-id="865dc-327">Azure Search</span><span class="sxs-lookup"><span data-stu-id="865dc-327">Azure Search</span></span>
<span data-ttu-id="865dc-328">Azure Search를 사용하면 강력하고 정교한 검색 기능을 웹 사이트 또는 응용 프로그램에 추가하고, 검색 결과를 쉽고 빠르게 조정하며, 풍부하고 미세 조정된 순위 모델을 생성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-328">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="865dc-329">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="865dc-329">Retry mechanism</span></span>
<span data-ttu-id="865dc-330">Azure Search SDK의 재시도 동작은 [SearchServiceClient] 및 [SearchIndexClient] 클래스의 `SetRetryPolicy` 메서드에 의해 제어됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-330">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="865dc-331">Azure Search가 5xx 또는 408(요청 시간 초과) 응답을 반환하는 경우 기본 정책은 지수 백오프를 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-331">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="865dc-332">원격 분석</span><span class="sxs-lookup"><span data-stu-id="865dc-332">Telemetry</span></span>
<span data-ttu-id="865dc-333">ETW를 사용하거나 사용자 지정 추적 공급자를 등록하는 추적입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-333">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="865dc-334">자세한 내용은 [AutoRest 설명서][autorest]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-334">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="service-bus"></a><span data-ttu-id="865dc-335">Service Bus</span><span class="sxs-lookup"><span data-stu-id="865dc-335">Service Bus</span></span>
<span data-ttu-id="865dc-336">Service Bus는 클라우드에서 호스트되는지 온-프레미스에서 호스트되는지에 관계없이 향상된 확장성 및 복원력으로 느슨하게 결합된 메시지 교환을 응용 프로그램의 구성 요소에 제공하는 클라우드 메시징 플랫폼입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-336">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="865dc-337">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="865dc-337">Retry mechanism</span></span>
<span data-ttu-id="865dc-338">Service Bus는 [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) 기본 클래스의 구현을 사용하여 재시도를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-338">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="865dc-339">모든 Service Bus 클라이언트는 **RetryPolicy** 기본 클래스의 구현 중 하나로 설정할 수 있는 **RetryPolicy** 속성을 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-339">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="865dc-340">기본 제공 구현은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-340">The built-in implementations are:</span></span>

* <span data-ttu-id="865dc-341">[RetryExponential 클래스](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="865dc-341">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="865dc-342">이 클래스는 백오프 간격, 재시도 횟수 및 작업이 완료되는 총 시간을 제한하는 데 사용되는 **TerminationTimeBuffer** 속성을 제어하는 속성을 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-342">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="865dc-343">[NoRetry 클래스](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="865dc-343">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="865dc-344">이 클래스는 재시도가 일괄 처리 또는 다단계 작업의 일부로 다른 프로세스에서 관리되는 경우처럼 Service Bus API 수준의 재시도가 필요하지 않은 경우에 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-344">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="865dc-345">Service Bus 작업은 [Service Bus 메시징 예외](/azure/service-bus-messaging/service-bus-messaging-exceptions)에 나열된 일련의 예외를 반환할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-345">Service Bus actions can return a range of exceptions, as listed in [Service Bus messaging exceptions](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span></span> <span data-ttu-id="865dc-346">이 목록에서는 작업 재시도가 적절한지 여부를 나타내는 예외에 대한 정보를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-346">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="865dc-347">예를 들어 **ServerBusyException** 은 클라이언트가 일정 기간 동안 대기한 후 작업을 재시도해야 함을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-347">For example, a **ServerBusyException** indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="865dc-348">또한 **ServerBusyException** 이 발생하면 Service Bus가 다른 모드로 전환되어 추가 10초의 지연이 계산된 재시도 지연에 추가될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-348">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="865dc-349">이 모드는 잠시 후 다시 설정됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-349">This mode is reset after a short period.</span></span>

<span data-ttu-id="865dc-350">Service Bus에서 반환된 예외는 클라이언트가 작업을 재시도해야 하는지 여부를 나타내는 **IsTransient** 속성을 표시합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-350">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="865dc-351">기본 제공 **RetryExponential** 정책은 모든 Service Bus 예외에 대한 기본 클래스인 **MessagingException** 클래스의 **IsTransient** 속성을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-351">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="865dc-352">**RetryPolicy** 기본 클래스의 사용자 지정 구현을 만드는 경우 예외 유형과 **IsTransient** 속성의 조합을 사용하여 재시도 작업을 보다 세부적으로 제어할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-352">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="865dc-353">예를 들어 **QuotaExceededException**을 검색하고 조치를 취하여 메시지 보내기를 재시도하기 전에 큐를 비울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-353">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="865dc-354">정책 구성</span><span class="sxs-lookup"><span data-stu-id="865dc-354">Policy configuration</span></span>
<span data-ttu-id="865dc-355">재시도 정책은 프로그래밍 방식으로 설정되며 **NamespaceManager** 및 **MessagingFactory**에 대한 기본 정책으로 설정하거나 각 메시징 클라이언트에 대해 개별적으로 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-355">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="865dc-356">메시징 세션에 대해 기본 재시도 정책을 설정하려면 **NamespaceManager**의 **RetryPolicy**를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-356">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

<span data-ttu-id="865dc-357">메시징 팩터리에서 만든 모든 클라이언트에 대해 기본 재시도 정책을 설정하려면 **MessagingFactory**의 **RetryPolicy**를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-357">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

<span data-ttu-id="865dc-358">메시징 클라이언트에 대한 재시도 정책을 설정하거나 기본 정책을 재정의하려면 필요한 정책 클래스의 인스턴스를 사용하여 **RetryPolicy** 속성을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-358">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="865dc-359">재시도 정책은 개별 작업 수준에서 설정할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-359">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="865dc-360">메시징 클라이언트에 대한 모든 작업에 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-360">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="865dc-361">다음 표에서는 기본 제공 재시도 정책의 기본 설정을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-361">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="865dc-362">설정</span><span class="sxs-lookup"><span data-stu-id="865dc-362">Setting</span></span> | <span data-ttu-id="865dc-363">기본값</span><span class="sxs-lookup"><span data-stu-id="865dc-363">Default value</span></span> | <span data-ttu-id="865dc-364">의미</span><span class="sxs-lookup"><span data-stu-id="865dc-364">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="865dc-365">정책</span><span class="sxs-lookup"><span data-stu-id="865dc-365">Policy</span></span> | <span data-ttu-id="865dc-366">지수</span><span class="sxs-lookup"><span data-stu-id="865dc-366">Exponential</span></span> | <span data-ttu-id="865dc-367">지수적 백오프.</span><span class="sxs-lookup"><span data-stu-id="865dc-367">Exponential back-off.</span></span> |
| <span data-ttu-id="865dc-368">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-368">MinimalBackoff</span></span> | <span data-ttu-id="865dc-369">0</span><span class="sxs-lookup"><span data-stu-id="865dc-369">0</span></span> | <span data-ttu-id="865dc-370">최소 백오프 간격입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-370">Minimum back-off interval.</span></span> <span data-ttu-id="865dc-371">deltaBackoff에서 계산된 재시도 간격에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-371">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="865dc-372">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-372">MaximumBackoff</span></span> | <span data-ttu-id="865dc-373">30초</span><span class="sxs-lookup"><span data-stu-id="865dc-373">30 seconds</span></span> | <span data-ttu-id="865dc-374">최대 백오프 간격입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-374">Maximum back-off interval.</span></span> <span data-ttu-id="865dc-375">MaximumBackoff는 계산된 재시도 간격이 MaxBackoff보다 큰 경우 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-375">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="865dc-376">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-376">DeltaBackoff</span></span> | <span data-ttu-id="865dc-377">3초</span><span class="sxs-lookup"><span data-stu-id="865dc-377">3 seconds</span></span> | <span data-ttu-id="865dc-378">재시도 사이의 백오프 간격입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-378">Back-off interval between retries.</span></span> <span data-ttu-id="865dc-379">후속 재시도에 이 시간 범위의 배수가 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-379">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="865dc-380">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="865dc-380">TimeBuffer</span></span> | <span data-ttu-id="865dc-381">5초</span><span class="sxs-lookup"><span data-stu-id="865dc-381">5 seconds</span></span> | <span data-ttu-id="865dc-382">재시도와 연결된 종료 시간 버퍼입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-382">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="865dc-383">남은 시간이 TimeBuffer보다 적으면 재시도가 중지됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-383">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="865dc-384">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="865dc-384">MaxRetryCount</span></span> | <span data-ttu-id="865dc-385">10</span><span class="sxs-lookup"><span data-stu-id="865dc-385">10</span></span> | <span data-ttu-id="865dc-386">최대 재시도 수</span><span class="sxs-lookup"><span data-stu-id="865dc-386">The maximum number of retries.</span></span> |
| <span data-ttu-id="865dc-387">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="865dc-387">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="865dc-388">10초</span><span class="sxs-lookup"><span data-stu-id="865dc-388">10 seconds</span></span> | <span data-ttu-id="865dc-389">마지막으로 발생한 예외가 **ServerBusyException**인 경우 이 값이 계산된 재시도 간격에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-389">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="865dc-390">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-390">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="865dc-391">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="865dc-391">Retry usage guidance</span></span>
<span data-ttu-id="865dc-392">Service Bus를 사용하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-392">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="865dc-393">기본 제공 **RetryExponential** 구현을 사용하는 경우 정책이 서버 사용 중 예외에 반응하고 적절한 재시도 모드로 자동으로 전환되므로 폴백 작업을 구현하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-393">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="865dc-394">Service Bus는 기본 네임스페이스의 큐가 실패할 경우 별도의 네임스페이스에 있는 백업 큐로 자동 장애 조치(failover)를 구현하는 쌍을 이루는 네임스페이스라는 기능을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-394">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="865dc-395">기본 큐가 복구되면 보조 큐의 메시지를 다시 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-395">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="865dc-396">이 기능은 일시적인 오류를 해결하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-396">This feature helps to address transient failures.</span></span> <span data-ttu-id="865dc-397">자세한 내용은 [비동기 메시징 패턴 및 고가용성](http://msdn.microsoft.com/library/azure/dn292562.aspx)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-397">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="865dc-398">재시도 작업에 대해 다음 설정을 사용하여 시작하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-398">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="865dc-399">이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-399">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="865dc-400">Context</span><span class="sxs-lookup"><span data-stu-id="865dc-400">Context</span></span> | <span data-ttu-id="865dc-401">예제 최대 대기 시간</span><span class="sxs-lookup"><span data-stu-id="865dc-401">Example maximum latency</span></span> | <span data-ttu-id="865dc-402">다시 시도 정책</span><span class="sxs-lookup"><span data-stu-id="865dc-402">Retry policy</span></span> | <span data-ttu-id="865dc-403">설정</span><span class="sxs-lookup"><span data-stu-id="865dc-403">Settings</span></span> | <span data-ttu-id="865dc-404">작동 방법</span><span class="sxs-lookup"><span data-stu-id="865dc-404">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="865dc-405">대화형, UI 또는 포그라운드</span><span class="sxs-lookup"><span data-stu-id="865dc-405">Interactive, UI, or foreground</span></span> | <span data-ttu-id="865dc-406">2초\*</span><span class="sxs-lookup"><span data-stu-id="865dc-406">2 seconds\*</span></span>  | <span data-ttu-id="865dc-407">지수</span><span class="sxs-lookup"><span data-stu-id="865dc-407">Exponential</span></span> | <span data-ttu-id="865dc-408">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="865dc-408">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="865dc-409">MaximumBackoff = 30 sec.</span><span class="sxs-lookup"><span data-stu-id="865dc-409">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="865dc-410">DeltaBackoff = 300 msec.</span><span class="sxs-lookup"><span data-stu-id="865dc-410">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="865dc-411">TimeBuffer = 300 msec.</span><span class="sxs-lookup"><span data-stu-id="865dc-411">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="865dc-412">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="865dc-412">MaxRetryCount = 2</span></span> | <span data-ttu-id="865dc-413">Attempt 1: Delay 0 sec.</span><span class="sxs-lookup"><span data-stu-id="865dc-413">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="865dc-414">Attempt 2: Delay ~300 msec.</span><span class="sxs-lookup"><span data-stu-id="865dc-414">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="865dc-415">Attempt 3: Delay ~900 msec.</span><span class="sxs-lookup"><span data-stu-id="865dc-415">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="865dc-416">백그라운드 또는 일괄 처리</span><span class="sxs-lookup"><span data-stu-id="865dc-416">Background or batch</span></span> | <span data-ttu-id="865dc-417">30초</span><span class="sxs-lookup"><span data-stu-id="865dc-417">30 seconds</span></span> | <span data-ttu-id="865dc-418">지수</span><span class="sxs-lookup"><span data-stu-id="865dc-418">Exponential</span></span> | <span data-ttu-id="865dc-419">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="865dc-419">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="865dc-420">MaximumBackoff = 30 sec.</span><span class="sxs-lookup"><span data-stu-id="865dc-420">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="865dc-421">DeltaBackoff = 1.75 sec.</span><span class="sxs-lookup"><span data-stu-id="865dc-421">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="865dc-422">TimeBuffer = 5 sec.</span><span class="sxs-lookup"><span data-stu-id="865dc-422">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="865dc-423">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="865dc-423">MaxRetryCount = 3</span></span> | <span data-ttu-id="865dc-424">Attempt 1: Delay ~1 sec.</span><span class="sxs-lookup"><span data-stu-id="865dc-424">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="865dc-425">Attempt 2: Delay ~3 sec.</span><span class="sxs-lookup"><span data-stu-id="865dc-425">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="865dc-426">Attempt 3: Delay ~6 msec.</span><span class="sxs-lookup"><span data-stu-id="865dc-426">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="865dc-427">Attempt 4: Delay ~13 msec.</span><span class="sxs-lookup"><span data-stu-id="865dc-427">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="865dc-428">\* 서버 사용 중 응답을 받은 경우 추가된 추가 지연을 포함하지 않음</span><span class="sxs-lookup"><span data-stu-id="865dc-428">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="865dc-429">원격 분석</span><span class="sxs-lookup"><span data-stu-id="865dc-429">Telemetry</span></span>
<span data-ttu-id="865dc-430">Service Bus는 **EventSource**를 사용하여 재시도를 ETW 이벤트로 기록합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-430">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="865dc-431">이벤트를 캡처하여 성능 뷰어에서 보거나 적합한 대상 로그에 기록하려면 **EventListener**를 이벤트 소스에 연결해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-431">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="865dc-432">재시도 이벤트는 다음과 같은 형식입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-432">The retry events are of the following form:</span></span>

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a><span data-ttu-id="865dc-433">예</span><span class="sxs-lookup"><span data-stu-id="865dc-433">Examples</span></span>
<span data-ttu-id="865dc-434">다음 코드 예제에서는 다음에 대해 재시도 정책을 설정하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-434">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="865dc-435">네임스페이스 관리자.</span><span class="sxs-lookup"><span data-stu-id="865dc-435">A namespace manager.</span></span> <span data-ttu-id="865dc-436">정책은 해당 관리자의 모든 작업에 적용되며 개별 작업에 대해 재정의할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-436">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="865dc-437">메시징 팩터리.</span><span class="sxs-lookup"><span data-stu-id="865dc-437">A messaging factory.</span></span> <span data-ttu-id="865dc-438">정책은 해당 팩터리에서 만든 모든 클라이언트에 적용되며 개별 클라이언트를 만드는 경우 재정의할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-438">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="865dc-439">개별 메시징 클라이언트.</span><span class="sxs-lookup"><span data-stu-id="865dc-439">An individual messaging client.</span></span> <span data-ttu-id="865dc-440">클라이언트를 만든 후 해당 클라이언트에 대한 재시도 정책을 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-440">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="865dc-441">정책은 해당 클라이언트의 모든 작업에 적용됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-441">The policy applies to all operations on that client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }

            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }

            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="865dc-442">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="865dc-442">More information</span></span>
* [<span data-ttu-id="865dc-443">비동기 메시징 패턴 및 고가용성</span><span class="sxs-lookup"><span data-stu-id="865dc-443">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="service-fabric"></a><span data-ttu-id="865dc-444">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="865dc-444">Service Fabric</span></span>

<span data-ttu-id="865dc-445">Service Fabric 클러스터에서 신뢰할 수 있는 서비스를 배포하면 이 문서에서 논의된 대부분의 가능한 일시적 오류를 방지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-445">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="865dc-446">그러나 일부 일시적 오류는 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-446">Some transient faults are still possible, however.</span></span> <span data-ttu-id="865dc-447">예를 들어 명명 서비스가 변경을 전달하는 도중에 요청을 받으면 예외가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-447">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="865dc-448">같은 요청을 100밀리초 후에 받으면 성공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-448">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="865dc-449">Service Fabric은 내부적으로 이런 종류의 일시적 오류를 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-449">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="865dc-450">서비스를 설정할 때 `OperationRetrySettings` 클래스를 사용하여 일부 설정을 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-450">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="865dc-451">다음은 예를 보여 주는 코드입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-451">The following code shows an example.</span></span> <span data-ttu-id="865dc-452">이것은 대부분의 경우 불필요하며 기본 설정으로 충분합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-452">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

```csharp
FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
{
    OperationTimeout = TimeSpan.FromSeconds(30)
};

var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
    new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
    new ServicePartitionKey(0));
```

### <a name="more-information"></a><span data-ttu-id="865dc-453">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="865dc-453">More information</span></span>

* [<span data-ttu-id="865dc-454">원격 예외 처리</span><span class="sxs-lookup"><span data-stu-id="865dc-454">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a><span data-ttu-id="865dc-455">ADO.NET을 사용하는 SQL Database</span><span class="sxs-lookup"><span data-stu-id="865dc-455">SQL Database using ADO.NET</span></span>
<span data-ttu-id="865dc-456">SQL Database는 다양한 크기와 표준(공유) 및 프리미엄(비공유) 서비스로 사용할 수 있는 호스트된 SQL Database입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-456">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="865dc-457">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="865dc-457">Retry mechanism</span></span>
<span data-ttu-id="865dc-458">SQL Database는 ADO.NET을 사용하여 액세스하는 경우 재시도에 대한 기본 제공 지원을 제공하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-458">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="865dc-459">그러나 요청에서 반환된 코드를 사용하여 요청이 실패한 이유를 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-459">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="865dc-460">SQL Database 제한에 대한 자세한 내용은 [Azure SQL Database 리소스 제한](/azure/sql-database/sql-database-resource-limits)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-460">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="865dc-461">관련 오류 코드 목록은 [SQL Database 클라이언트 응용 프로그램에 대한 SQL 오류 코드](/azure/sql-database/sql-database-develop-error-messages)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-461">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="865dc-462">SQL Database에 대한 재시도 구현에 Polly 라이브러리를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-462">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="865dc-463">[Polly를 통한 일시적인 오류 처리](#transient-fault-handling-with-polly)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-463">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="865dc-464">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="865dc-464">Retry usage guidance</span></span>
<span data-ttu-id="865dc-465">ADO.NET을 사용하는 SQL Database에 액세스하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-465">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="865dc-466">적절한 서비스 옵션(공유 또는 프리미엄)을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-466">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="865dc-467">공유 인스턴스는 공유 서버의 다른 테넌트에서 사용되기 때문에 일반적인 연결 지연 및 제한보다 더 길게 영향을 받을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-467">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="865dc-468">예측 가능한 성능 및 대기 시간이 짧은 안정적인 작업이 필요한 경우 프리미엄 옵션을 선택하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-468">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="865dc-469">적절한 수준 또는 범위에서 재시도를 수행하여 데이터 불일치를 발생시키는 비멱등 작업을 방지해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-469">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="865dc-470">이상적으로는 불일치를 발생시키지 않고 반복할 수 있도록 모든 작업이 멱등이어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-470">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="865dc-471">그렇지 않으면 한 작업이 실패할 경우 모든 관련 변경 내용이 실행 취소되도록 하는 수준 또는 범위(예: 트랜잭션 범위 내)에서 재시도를 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-471">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="865dc-472">자세한 내용은 [클라우드 서비스의 기본 데이터 액세스 계층 – 일시적인 오류 처리](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee)(영문)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-472">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="865dc-473">고정 간격 전략은 매우 짧은 간격에 몇 번의 재시도만 수행되는 대화형 시나리오를 제외하고 Azure SQL Database에서 사용하지 않는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-473">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="865dc-474">대신 대부분의 시나리오에 지수 백오프 전략을 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-474">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="865dc-475">연결을 정의할 때 연결 및 명령 제한 시간에 적합한 값을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-475">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="865dc-476">제한 시간이 너무 짧으면 데이터베이스가 사용 중일 때 중간에 연결 오류가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-476">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="865dc-477">제한 시간이 너무 길면 실패한 연결을 검색할 때까지 너무 오래 대기하여 재시도 논리가 올바르게 작동하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-477">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="865dc-478">제한 시간 값은 종단 간 대기 시간의 구성 요소이지므로 모든 재시도에 대한 재시도 정책에 지정된 재시도 지연에 효과적으로 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-478">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="865dc-479">지수 백오프 재시도 논리를 사용하는 경우에도 특정 횟수의 재시도 후에는 연결을 닫고 새 연결에서 작업을 재시도합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-479">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="865dc-480">동일한 연결에서 동일한 작업을 여러 번 재시도하면 연결 문제에 영향을 주는 요인이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-480">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="865dc-481">이 기술에 대한 예제는 [클라우드 서비스의 기본 데이터 액세스 계층 – 일시적인 오류 처리](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)(영문)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-481">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="865dc-482">연결 풀링이 사용 중(기본값)인 경우 연결을 닫았다가 다시 연 후에도 풀에서 동일한 연결을 선택할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-482">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="865dc-483">이런 경우 해결하는 방법은 **SqlConnection** 클래스의 **ClearPool** 메서드를 호출하여 연결을 재사용할 수 없음으로 표시하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-483">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="865dc-484">그러나 이 방법은 여러 번의 연결 시도가 실패한 후에만 수행하고 오류가 발생한 연결과 관련된 SQL 제한 시간(오류 코드 -2)과 같은 특정 클래스의 일시적인 오류가 발생하는 경우에만 수행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-484">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="865dc-485">데이터 액세스 코드가 **TransactionScope** 인스턴스로 시작된 트랜잭션을 사용하는 경우 재시도 논리에서 연결을 다시 열고 새 트랜잭션 범위를 시작해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-485">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="865dc-486">이러한 이유로 재시도 가능한 코드 블록은 트랜잭션의 전체 범위를 포함해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-486">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="865dc-487">재시도 작업에 대해 다음 설정을 사용하여 시작하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-487">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="865dc-488">이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-488">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="865dc-489">**컨텍스트**</span><span class="sxs-lookup"><span data-stu-id="865dc-489">**Context**</span></span> | <span data-ttu-id="865dc-490">**샘플 대상 E2E<br />최대 대기 시간**</span><span class="sxs-lookup"><span data-stu-id="865dc-490">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="865dc-491">**재시도 전략**</span><span class="sxs-lookup"><span data-stu-id="865dc-491">**Retry strategy**</span></span> | <span data-ttu-id="865dc-492">**설정**</span><span class="sxs-lookup"><span data-stu-id="865dc-492">**Settings**</span></span> | <span data-ttu-id="865dc-493">**값**</span><span class="sxs-lookup"><span data-stu-id="865dc-493">**Values**</span></span> | <span data-ttu-id="865dc-494">**작동 방법**</span><span class="sxs-lookup"><span data-stu-id="865dc-494">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="865dc-495">대화형, UI</span><span class="sxs-lookup"><span data-stu-id="865dc-495">Interactive, UI,</span></span><br /><span data-ttu-id="865dc-496">또는 포그라운드</span><span class="sxs-lookup"><span data-stu-id="865dc-496">or foreground</span></span> |<span data-ttu-id="865dc-497">2초</span><span class="sxs-lookup"><span data-stu-id="865dc-497">2 sec</span></span> |<span data-ttu-id="865dc-498">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="865dc-498">FixedInterval</span></span> |<span data-ttu-id="865dc-499">재시도 횟수</span><span class="sxs-lookup"><span data-stu-id="865dc-499">Retry count</span></span><br /><span data-ttu-id="865dc-500">재시도 간격</span><span class="sxs-lookup"><span data-stu-id="865dc-500">Retry interval</span></span><br /><span data-ttu-id="865dc-501">첫 번째 빠른 재시도</span><span class="sxs-lookup"><span data-stu-id="865dc-501">First fast retry</span></span> |<span data-ttu-id="865dc-502">3</span><span class="sxs-lookup"><span data-stu-id="865dc-502">3</span></span><br /><span data-ttu-id="865dc-503">500ms</span><span class="sxs-lookup"><span data-stu-id="865dc-503">500 ms</span></span><br /><span data-ttu-id="865dc-504">true</span><span class="sxs-lookup"><span data-stu-id="865dc-504">true</span></span> |<span data-ttu-id="865dc-505">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-505">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="865dc-506">시도 2 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-506">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="865dc-507">시도 3 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-507">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="865dc-508">백그라운드</span><span class="sxs-lookup"><span data-stu-id="865dc-508">Background</span></span><br /><span data-ttu-id="865dc-509">또는 일괄 처리</span><span class="sxs-lookup"><span data-stu-id="865dc-509">or batch</span></span> |<span data-ttu-id="865dc-510">30초</span><span class="sxs-lookup"><span data-stu-id="865dc-510">30 sec</span></span> |<span data-ttu-id="865dc-511">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-511">ExponentialBackoff</span></span> |<span data-ttu-id="865dc-512">재시도 횟수</span><span class="sxs-lookup"><span data-stu-id="865dc-512">Retry count</span></span><br /><span data-ttu-id="865dc-513">최소 백오프</span><span class="sxs-lookup"><span data-stu-id="865dc-513">Min back-off</span></span><br /><span data-ttu-id="865dc-514">최대 백오프</span><span class="sxs-lookup"><span data-stu-id="865dc-514">Max back-off</span></span><br /><span data-ttu-id="865dc-515">델타 백오프</span><span class="sxs-lookup"><span data-stu-id="865dc-515">Delta back-off</span></span><br /><span data-ttu-id="865dc-516">첫 번째 빠른 재시도</span><span class="sxs-lookup"><span data-stu-id="865dc-516">First fast retry</span></span> |<span data-ttu-id="865dc-517">5</span><span class="sxs-lookup"><span data-stu-id="865dc-517">5</span></span><br /><span data-ttu-id="865dc-518">0초</span><span class="sxs-lookup"><span data-stu-id="865dc-518">0 sec</span></span><br /><span data-ttu-id="865dc-519">60초</span><span class="sxs-lookup"><span data-stu-id="865dc-519">60 sec</span></span><br /><span data-ttu-id="865dc-520">2초</span><span class="sxs-lookup"><span data-stu-id="865dc-520">2 sec</span></span><br /><span data-ttu-id="865dc-521">false</span><span class="sxs-lookup"><span data-stu-id="865dc-521">false</span></span> |<span data-ttu-id="865dc-522">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-522">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="865dc-523">시도 2 - ~2초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-523">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="865dc-524">시도 3 - ~6초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-524">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="865dc-525">시도 4 - ~14초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-525">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="865dc-526">시도 5 - ~30초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-526">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="865dc-527">종단 간 대기 시간 대상은 서비스에 대한 연결의 기본 제한 시간을 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-527">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="865dc-528">연결 제한 시간을 더 길게 지정하면 종단 간 대기 시간이 모든 재시도에 대해 이 추가 시간만큼 확장됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-528">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="865dc-529">예</span><span class="sxs-lookup"><span data-stu-id="865dc-529">Examples</span></span>
<span data-ttu-id="865dc-530">이 섹션에서는 `Policy` 클래스에 구성된 재시도 정책 집합을 사용하는 Azure SQL Database에 Polly를 사용하여 액세스하는 방법을 보여 줍니다. </span><span class="sxs-lookup"><span data-stu-id="865dc-530">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="865dc-531">다음 코드는 지수 백오프를 통해 `ExecuteAsync`를 호출하는 `SqlCommand` 클래스의 확장 메서드를 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-531">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

<span data-ttu-id="865dc-532">이 비동기 확장 메서드는 다음과 같이 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-532">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="865dc-533">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="865dc-533">More information</span></span>
* [<span data-ttu-id="865dc-534">클라우드 서비스의 기본 데이터 액세스 계층 – 일시적인 오류 처리(영문)</span><span class="sxs-lookup"><span data-stu-id="865dc-534">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="865dc-535">SQL Database를 최대한 활용하기 위한 일반적인 지침은 [Azure SQL Database 성능 및 탄력성 가이드](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-535">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>

## <a name="sql-database-using-entity-framework-6"></a><span data-ttu-id="865dc-536">Entity Framework 6을 사용하는 SQL Database</span><span class="sxs-lookup"><span data-stu-id="865dc-536">SQL Database using Entity Framework 6</span></span>
<span data-ttu-id="865dc-537">SQL Database는 다양한 크기와 표준(공유) 및 프리미엄(비공유) 서비스로 사용할 수 있는 호스트된 SQL Database입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-537">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="865dc-538">Entity Framework는 .NET 개발자가 도메인별 개체를 사용하여 관계형 데이터로 작업할 수 있는 개체 관계형 매퍼입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-538">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="865dc-539">여기서는 개발자가 일반적으로 작성해야 하는 대부분의 데이터 액세스 코드가 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-539">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="865dc-540">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="865dc-540">Retry mechanism</span></span>
<span data-ttu-id="865dc-541">재시도는 [연결 복원/재시도 논리](http://msdn.microsoft.com/data/dn456835.aspx)(영문)라는 메커니즘을 통해 Entity Framework 6.0 이상을 사용하는 SQL Database에 액세스할 때 지원됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-541">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="865dc-542">재시도 메커니즘의 주요 기능은 다음과 같습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-542">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="865dc-543">기본 추상화는 **IDbExecutionStrategy** 인터페이스입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-543">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="865dc-544">이 인터페이스는 다음을 수행합니다.:</span><span class="sxs-lookup"><span data-stu-id="865dc-544">This interface:</span></span>
  * <span data-ttu-id="865dc-545">동기 및 비동기 **Execute**\* 메서드를 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-545">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="865dc-546">직접 사용하거나 데이터베이스 컨텍스트에서 기본 전략으로 구성하거나, 공급자 이름에 매핑하거나, 공급자 이름 및 서버 이름에 매핑할 수 있는 클래스를 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-546">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="865dc-547">컨텍스트에서 구성할 경우 재시도는 지정된 컨텍스트 작업에 대해 여러 개가 있을 수 있는 개별 데이터베이스 작업 수준에서 수행됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-547">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="865dc-548">실패한 연결을 재시도할 시기 및 방법을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-548">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="865dc-549">여기에는 다음과 같은 **IDbExecutionStrategy** 인터페이스의 여러 가지 기본 제공 구현이 포함됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-549">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="865dc-550">기본값 - 재시도하지 않음.</span><span class="sxs-lookup"><span data-stu-id="865dc-550">Default - no retrying.</span></span>
  * <span data-ttu-id="865dc-551">SQL Database(자동)에 대한 기본값 - 재시도하지 않지만 예외를 검사하고 SQL Database 전략을 사용하는 제안으로 래핑합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-551">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="865dc-552">SQL Database에 대한 기본값 - 지수(기본 클래스에서 상속됨) 및 SQL Database 검색 논리.</span><span class="sxs-lookup"><span data-stu-id="865dc-552">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="865dc-553">불규칙을 포함하는 지수 백오프 전략을 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-553">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="865dc-554">기본 제공 재시도 클래스는 상태 저장 클래스이며 스레드로부터 안전하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-554">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="865dc-555">그러나 현재 작업이 완료된 후 다시 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-555">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="865dc-556">지정된 재시도 횟수를 초과하는 경우 결과는 새 예외에 래핑됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-556">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="865dc-557">현재 예외를 버블 업하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-557">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="865dc-558">정책 구성</span><span class="sxs-lookup"><span data-stu-id="865dc-558">Policy configuration</span></span>
<span data-ttu-id="865dc-559">재시도는 Entity Framework 6.0 이상을 사용하는 SQL Database에 액세스할 때 지원됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-559">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="865dc-560">재시도 정책은 프로그래밍 방식으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-560">Retry policies are configured programmatically.</span></span> <span data-ttu-id="865dc-561">구성은 작업 단위로 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-561">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="865dc-562">컨텍스트에서 기본값으로 전략을 구성하는 경우 필요에 따라 새 전략을 만드는 함수를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-562">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="865dc-563">다음 코드에서는 **DbConfiguration** 기본 클래스를 확장하는 재시도 구성 클래스를 만드는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-563">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

<span data-ttu-id="865dc-564">그런 다음 응용 프로그램이 시작될 때 **DbConfiguration** 인스턴스의 **SetConfiguration** 메서드를 사용하여 모든 작업에 대해 기본 재시도 전략으로 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-564">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="865dc-565">기본적으로 EF는 구성 클래스를 자동으로 검색하고 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-565">By default, EF will automatically discover and use the configuration class.</span></span>

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

<span data-ttu-id="865dc-566">**DbConfigurationType** 특성으로 컨텍스트 클래스에 주석을 추가하여 컨텍스트에 대한 재시도 구성 클래스를 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-566">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="865dc-567">그러나 구성 클래스가 하나만 있는 경우 EF는 컨텍스트에 주석을 추가할 필요 없이 해당 클래스를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-567">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

<span data-ttu-id="865dc-568">특정 작업에 다른 재시도 전략을 사용하거나 특정 작업에 대해 재시도를 사용하지 않도록 설정해야 하는 경우 **CallContext**에서 플래그를 설정하여 전략을 일시 중단하거나 교환할 수 있도록 하는 구성 클래스를 만들 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-568">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="865dc-569">구성 클래스는 이 플래그를 사용하여 전략을 전환하거나, 제공된 전략을 사용하지 않도록 설정하고 기본 전략을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-569">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="865dc-570">자세한 내용은 재시도 실행 전략의 제한 사항(EF6 이상) 페이지에서 [실행 전략 일시 중단](http://msdn.microsoft.com/dn307226#transactions_workarounds)(영문)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-570">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="865dc-571">개별 작업에 특정 재시도 전략을 사용하기 위한 다른 기술은 필요한 전략 클래스의 인스턴스를 만들고 매개 변수를 통해 원하는 설정을 제공하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-571">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="865dc-572">그런 다음 **ExecuteAsync** 메서드를 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-572">You then invoke its **ExecuteAsync** method.</span></span>

```csharp
var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
var blogs = await executionStrategy.ExecuteAsync(
    async () =>
    {
        using (var db = new BloggingContext("Blogs"))
        {
            // Acquire some values asynchronously and return them
        }
    },
    new CancellationToken()
);
```

<span data-ttu-id="865dc-573">**DbConfiguration** 클래스를 사용하는 가장 간단한 방법은 **DbContext** 클래스와 동일한 어셈블리에 배치하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-573">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="865dc-574">그러나 다양한 대화형 및 백그라운드 재시도 전략과 같이 서로 다른 시나리오에서 동일한 컨텍스트 필요한 경우에는 적합하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-574">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="865dc-575">서로 다른 컨텍스트가 별도의 AppDomain에서 실행되는 경우 구성 파일에서 구성 클래스를 지정하는 데 기본 제공 지원을 사용하거나 코드를 사용하여 명시적으로 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-575">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="865dc-576">다른 컨텍스트가 동일한 AppDomain에서 실행되어야 하는 경우 사용자 지정 솔루션이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-576">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="865dc-577">자세한 내용은 [코드 기반 구성(EF6 이상)](http://msdn.microsoft.com/data/jj680699.aspx)(영문)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-577">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="865dc-578">다음 표에서는 EF6을 사용하는 경우 기본 제공 재시도 정책의 기본 설정을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-578">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="865dc-579">설정</span><span class="sxs-lookup"><span data-stu-id="865dc-579">Setting</span></span> | <span data-ttu-id="865dc-580">기본값</span><span class="sxs-lookup"><span data-stu-id="865dc-580">Default value</span></span> | <span data-ttu-id="865dc-581">의미</span><span class="sxs-lookup"><span data-stu-id="865dc-581">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="865dc-582">정책</span><span class="sxs-lookup"><span data-stu-id="865dc-582">Policy</span></span> | <span data-ttu-id="865dc-583">지수</span><span class="sxs-lookup"><span data-stu-id="865dc-583">Exponential</span></span> | <span data-ttu-id="865dc-584">지수적 백오프.</span><span class="sxs-lookup"><span data-stu-id="865dc-584">Exponential back-off.</span></span> |
| <span data-ttu-id="865dc-585">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="865dc-585">MaxRetryCount</span></span> | <span data-ttu-id="865dc-586">5</span><span class="sxs-lookup"><span data-stu-id="865dc-586">5</span></span> | <span data-ttu-id="865dc-587">최대 재시도 수</span><span class="sxs-lookup"><span data-stu-id="865dc-587">The maximum number of retries.</span></span> |
| <span data-ttu-id="865dc-588">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="865dc-588">MaxDelay</span></span> | <span data-ttu-id="865dc-589">30초</span><span class="sxs-lookup"><span data-stu-id="865dc-589">30 seconds</span></span> | <span data-ttu-id="865dc-590">재시도 사이의 최대 지연.</span><span class="sxs-lookup"><span data-stu-id="865dc-590">The maximum delay between retries.</span></span> <span data-ttu-id="865dc-591">이 값은 일련의 지연이 계산되는 방식에는 영향이 없으며</span><span class="sxs-lookup"><span data-stu-id="865dc-591">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="865dc-592">상한 값만 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-592">It only defines an upper bound.</span></span> |
| <span data-ttu-id="865dc-593">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="865dc-593">DefaultCoefficient</span></span> | <span data-ttu-id="865dc-594">1초</span><span class="sxs-lookup"><span data-stu-id="865dc-594">1 second</span></span> | <span data-ttu-id="865dc-595">지수 백오프 계산을 위한 계수.</span><span class="sxs-lookup"><span data-stu-id="865dc-595">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="865dc-596">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-596">This value cannot be changed.</span></span> |
| <span data-ttu-id="865dc-597">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="865dc-597">DefaultRandomFactor</span></span> | <span data-ttu-id="865dc-598">1.1</span><span class="sxs-lookup"><span data-stu-id="865dc-598">1.1</span></span> | <span data-ttu-id="865dc-599">각 항목에 대해 임의 지연을 추가하는 데 사용되는 승수입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-599">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="865dc-600">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-600">This value cannot be changed.</span></span> |
| <span data-ttu-id="865dc-601">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="865dc-601">DefaultExponentialBase</span></span> | <span data-ttu-id="865dc-602">2</span><span class="sxs-lookup"><span data-stu-id="865dc-602">2</span></span> | <span data-ttu-id="865dc-603">다음 지연을 계산하는 데 사용되는 승수입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-603">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="865dc-604">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-604">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="865dc-605">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="865dc-605">Retry usage guidance</span></span>
<span data-ttu-id="865dc-606">EF6을 사용하는 SQL Database에 액세스하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-606">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="865dc-607">적절한 서비스 옵션(공유 또는 프리미엄)을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-607">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="865dc-608">공유 인스턴스는 공유 서버의 다른 테넌트에서 사용되기 때문에 일반적인 연결 지연 및 제한보다 더 길게 영향을 받을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-608">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="865dc-609">예측 가능한 성능 및 대기 시간이 짧은 안정적인 작업이 필요한 경우 프리미엄 옵션을 선택하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-609">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="865dc-610">Azure SQL Database에는 고정 간격 전략을 사용하지 않는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-610">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="865dc-611">대신 서비스가 오버로드될 수 있고 더 긴 지연으로 인해 복구에 더 많은 시간이 필요할 수 있으므로 지수 백오프 전략을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-611">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="865dc-612">연결을 정의할 때 연결 및 명령 제한 시간에 적합한 값을 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-612">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="865dc-613">제한 시간은 비즈니스 논리 디자인 및 테스트를 기반으로 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-613">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="865dc-614">시간에 따라 데이터의 양이나 비즈니스 프로세스가 변경되므로 이 값을 수정해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-614">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="865dc-615">제한 시간이 너무 짧으면 데이터베이스가 사용 중일 때 중간에 연결 오류가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-615">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="865dc-616">제한 시간이 너무 길면 실패한 연결을 검색할 때까지 너무 오래 대기하여 재시도 논리가 올바르게 작동하지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-616">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="865dc-617">제한 시간 값은 종단 간 대기 시간의 구성 요소이지만 컨텍스트를 저장할 때 실행될 명령 수를 쉽게 확인할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-617">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="865dc-618">**DbContext** 인스턴스의 **CommandTimeout** 속성을 설정하여 기본 제한 시간을 변경할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-618">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="865dc-619">Entity Framework는 구성 파일에 정의된 재시도 구성을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-619">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="865dc-620">그러나 Azure에서 최대한의 유연성을 제공하려면 응용 프로그램 내에서 프로그래밍 방식으로 구성을 만드는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-620">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="865dc-621">재시도 정책에 대한 특정 매개 변수(예: 재시도 횟수 및 다시 시도 간격)는 서비스 구성 파일에 저장하여 런타임에 적절한 정책을 만드는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-621">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="865dc-622">이렇게 하면 응용 프로그램을 다시 시작하지 않고도 설정을 변경할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-622">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="865dc-623">재시도 작업에 대해 다음 설정을 사용하여 시작하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-623">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="865dc-624">재시도 횟수 사이에는 지연을 지정할 수 없습니다. 이 값은 고정이며 지수 시퀀스로 생성됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-624">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="865dc-625">다음과 같이 사용자 지정 재시도 전략을 만들지 않는 경우 최대값만 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-625">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="865dc-626">이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-626">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="865dc-627">**컨텍스트**</span><span class="sxs-lookup"><span data-stu-id="865dc-627">**Context**</span></span> | <span data-ttu-id="865dc-628">**샘플 대상 E2E<br />최대 대기 시간**</span><span class="sxs-lookup"><span data-stu-id="865dc-628">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="865dc-629">**다시 시도 정책**</span><span class="sxs-lookup"><span data-stu-id="865dc-629">**Retry policy**</span></span> | <span data-ttu-id="865dc-630">**설정**</span><span class="sxs-lookup"><span data-stu-id="865dc-630">**Settings**</span></span> | <span data-ttu-id="865dc-631">**값**</span><span class="sxs-lookup"><span data-stu-id="865dc-631">**Values**</span></span> | <span data-ttu-id="865dc-632">**작동 방법**</span><span class="sxs-lookup"><span data-stu-id="865dc-632">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="865dc-633">대화형, UI</span><span class="sxs-lookup"><span data-stu-id="865dc-633">Interactive, UI,</span></span><br /><span data-ttu-id="865dc-634">또는 포그라운드</span><span class="sxs-lookup"><span data-stu-id="865dc-634">or foreground</span></span> |<span data-ttu-id="865dc-635">2초</span><span class="sxs-lookup"><span data-stu-id="865dc-635">2 seconds</span></span> |<span data-ttu-id="865dc-636">지수</span><span class="sxs-lookup"><span data-stu-id="865dc-636">Exponential</span></span> |<span data-ttu-id="865dc-637">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="865dc-637">MaxRetryCount</span></span><br /><span data-ttu-id="865dc-638">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="865dc-638">MaxDelay</span></span> |<span data-ttu-id="865dc-639">3</span><span class="sxs-lookup"><span data-stu-id="865dc-639">3</span></span><br /><span data-ttu-id="865dc-640">750ms</span><span class="sxs-lookup"><span data-stu-id="865dc-640">750 ms</span></span> |<span data-ttu-id="865dc-641">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-641">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="865dc-642">시도 2 - 750ms 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-642">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="865dc-643">시도 3 – 750ms 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-643">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="865dc-644">백그라운드</span><span class="sxs-lookup"><span data-stu-id="865dc-644">Background</span></span><br /> <span data-ttu-id="865dc-645">또는 일괄 처리</span><span class="sxs-lookup"><span data-stu-id="865dc-645">or batch</span></span> |<span data-ttu-id="865dc-646">30초</span><span class="sxs-lookup"><span data-stu-id="865dc-646">30 seconds</span></span> |<span data-ttu-id="865dc-647">지수</span><span class="sxs-lookup"><span data-stu-id="865dc-647">Exponential</span></span> |<span data-ttu-id="865dc-648">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="865dc-648">MaxRetryCount</span></span><br /><span data-ttu-id="865dc-649">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="865dc-649">MaxDelay</span></span> |<span data-ttu-id="865dc-650">5</span><span class="sxs-lookup"><span data-stu-id="865dc-650">5</span></span><br /><span data-ttu-id="865dc-651">12초</span><span class="sxs-lookup"><span data-stu-id="865dc-651">12 seconds</span></span> |<span data-ttu-id="865dc-652">시도 1 - 0초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-652">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="865dc-653">시도 2 - ~1초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-653">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="865dc-654">시도 3 - ~3초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-654">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="865dc-655">시도 4 - ~7초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-655">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="865dc-656">시도 5 - ~12초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-656">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="865dc-657">종단 간 대기 시간 대상은 서비스에 대한 연결의 기본 제한 시간을 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-657">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="865dc-658">연결 제한 시간을 더 길게 지정하면 종단 간 대기 시간이 모든 재시도에 대해 이 추가 시간만큼 확장됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-658">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="865dc-659">예</span><span class="sxs-lookup"><span data-stu-id="865dc-659">Examples</span></span>
<span data-ttu-id="865dc-660">다음 코드 예제에서는 Entity Framework를 사용하는 간단한 데이터 액세스 솔루션을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-660">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="865dc-661">이 예제에서는 **DbConfiguration**을 확장하는 **BlogConfiguration**이라는 클래스의 인스턴스를 정의하여 특정 재시도 전략을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-661">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

<span data-ttu-id="865dc-662">Entity Framework 재시도 메커니즘 사용에 대한 더 많은 예제는 [연결 복구/재시도 논리](http://msdn.microsoft.com/data/dn456835.aspx)(영문)에서 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-662">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="865dc-663">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="865dc-663">More information</span></span>
* [<span data-ttu-id="865dc-664">Azure SQL Database 성능 및 탄력성 가이드(영문)</span><span class="sxs-lookup"><span data-stu-id="865dc-664">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a><span data-ttu-id="865dc-665">Entity Framework Core를 사용하는 SQL Database</span><span class="sxs-lookup"><span data-stu-id="865dc-665">SQL Database using Entity Framework Core</span></span>
<span data-ttu-id="865dc-666">[Entity Framework Core](/ef/core/)는 .NET Core 개발자가 도메인별 개체를 사용하여 데이터로 작업할 수 있는 개체 관계형 매퍼입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-666">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="865dc-667">여기서는 개발자가 일반적으로 작성해야 하는 대부분의 데이터 액세스 코드가 필요하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-667">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="865dc-668">이 Entity Framework 버전은 처음부터 새로 작성되었으며 EF6.x의 모든 기능을 자동으로 상속하지는 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-668">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="865dc-669">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="865dc-669">Retry mechanism</span></span>
<span data-ttu-id="865dc-670">재시도 지원은 [연결 복원력](/ef/core/miscellaneous/connection-resiliency)이라는 메커니즘을 통해 Entity Framework Core를 사용하는 SQL Database에 액세스할 때 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-670">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="865dc-671">연결 복원력은 EF Core 1.1.0에서 도입되었습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-671">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="865dc-672">기본 추상화는 `IExecutionStrategy` 인터페이스입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-672">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="865dc-673">SQL Azure를 포함한 SQL Server 실행 전략은 항상 재시도할 수 있는 예외 유형을 인지하며 최대 재시도, 재시도 간 지연 등에 대한 인식 가능한 기본값을 갖습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-673">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="865dc-674">예</span><span class="sxs-lookup"><span data-stu-id="865dc-674">Examples</span></span>

<span data-ttu-id="865dc-675">다음 코드는 데이터베이스와의 세션을 나타내는 DbContext 개체를 구성할 때 자동 재시도를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-675">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="865dc-676">다음 코드는 실행 전략을 사용하여 자동 재시도가 있는 트랜잭션 실행 방법을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-676">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="865dc-677">트랜잭션은 대리자에서 정의됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-677">The transaction is defined in a delegate.</span></span> <span data-ttu-id="865dc-678">일시적인 오류가 발생하면 실행 적략이 대리자를 다시 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-678">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a><span data-ttu-id="865dc-679">Azure Storage</span><span class="sxs-lookup"><span data-stu-id="865dc-679">Azure Storage</span></span>
<span data-ttu-id="865dc-680">Azure Storage 서비스에는 테이블 및 Blob Storage, 파일 및 저장소 큐가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-680">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="865dc-681">재시도 메커니즘</span><span class="sxs-lookup"><span data-stu-id="865dc-681">Retry mechanism</span></span>
<span data-ttu-id="865dc-682">재시도는 개별 REST 작업 수준에서 수행되며 클라이언트 API 구현의 중요한 부분입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-682">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="865dc-683">클라이언트 저장소 SDK는 [IExtendedRetryPolicy 인터페이스](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx)를 구현하는 클래스를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-683">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="865dc-684">인터페이스 구현은 여러 가지가 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-684">There are different implementations of the interface.</span></span> <span data-ttu-id="865dc-685">Storage 클라이언트는 테이블, blob 및 큐에 액세스하기 위해 특별히 설계된 정책에서 선택할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-685">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="865dc-686">각 구현에서는 기본적으로 다시 시도 간격 및 기타 세부 정보를 정의하는 다양한 재시도 전략을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-686">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="865dc-687">기본 제공 클래스는 불규칙 다시 시도 간격으로 선형(일정한 지연) 및 지수를 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-687">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="865dc-688">또한 다른 프로세스가 더 높은 수준에서 재시도를 처리하는 경우 사용할 재시도 정책이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-688">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="865dc-689">그러나 기본 제공 클래스에서 제공하지 않는 특정 요구 사항이 있는 경우 고유한 재시도 클래스를 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-689">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="865dc-690">RA-GRS(읽기 액세스 지역 중복 저장소)를 사용하고 요청의 결과가 재시도 가능한 오류인 경우 기본과 보조 저장소 서비스 위치 간에 대체 재시도가 전환합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-690">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="865dc-691">자세한 내용은 [Azure Storage 중복 옵션](http://msdn.microsoft.com/library/azure/dn727290.aspx) 을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-691">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="865dc-692">정책 구성</span><span class="sxs-lookup"><span data-stu-id="865dc-692">Policy configuration</span></span>
<span data-ttu-id="865dc-693">재시도 정책은 프로그래밍 방식으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-693">Retry policies are configured programmatically.</span></span> <span data-ttu-id="865dc-694">일반적인 프로시저는 **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** 또는 **QueueRequestOptions** 인스턴스를 만들고 채우는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-694">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="865dc-695">그런 다음 클라이언트에서 요청 옵션 인스턴스를 설정할 수 있으며 클라이언트와 관련된 모든 작업에서 지정된 요청 옵션을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-695">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="865dc-696">요청 옵션 클래스의 채워진 인스턴스를 매개 변수로 작업 메서드에 전달하여 클라이언트 요청 옵션을 재정의할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-696">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="865dc-697">**OperationContext** 인스턴스를 사용하여 재시도가 수행될 때와 작업이 완료되었을 때 실행할 코드를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-697">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="865dc-698">이 코드는 로그 및 원격 분석에서 사용하기 위해 작업에 대한 정보를 수집할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-698">This code can collect information about the operation for use in logs and telemetry.</span></span>

```csharp
// Set up notifications for an operation
var context = new OperationContext();
context.ClientRequestID = "some request id";
context.Retrying += (sender, args) =>
{
    /* Collect retry information */
};
context.RequestCompleted += (sender, args) =>
{
    /* Collect operation completion information */
};
var stats = await client.GetServiceStatsAsync(null, context);
```

<span data-ttu-id="865dc-699">확장된 재시도 정책은 오류가 재시도에 적합한지 여부를 나타낼 뿐만 아니라 재시도 횟수, 마지막 요청의 결과, 다음 재시도가 기본 위치에서 발생되는지 보조 위치에서 발생되는지를 나타내는 **RetryContext** 개체를 반환합니다(자세한 내용은 아래 표 참조).</span><span class="sxs-lookup"><span data-stu-id="865dc-699">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="865dc-700">**RetryContext** 개체의 속성은 재시도를 시도할 경우 및 시기를 결정하는 데 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-700">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="865dc-701">자세한 내용은 [IExtendedRetryPolicy.Evaluate 메서드](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-701">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="865dc-702">다음 표는 기본 제공 재시도 정책의 기본 설정을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-702">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="865dc-703">**요청 옵션**</span><span class="sxs-lookup"><span data-stu-id="865dc-703">**Request options**</span></span>

| <span data-ttu-id="865dc-704">**설정**</span><span class="sxs-lookup"><span data-stu-id="865dc-704">**Setting**</span></span> | <span data-ttu-id="865dc-705">**기본값**</span><span class="sxs-lookup"><span data-stu-id="865dc-705">**Default value**</span></span> | <span data-ttu-id="865dc-706">**의미**</span><span class="sxs-lookup"><span data-stu-id="865dc-706">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="865dc-707">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="865dc-707">MaximumExecutionTime</span></span> | <span data-ttu-id="865dc-708">없음</span><span class="sxs-lookup"><span data-stu-id="865dc-708">None</span></span> | <span data-ttu-id="865dc-709">모든 잠재적인 재시도 횟수를 포함하여 요청의 최대 실행 시간입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-709">Maximum execution time for the request, including all potential retry attempts.</span></span> <span data-ttu-id="865dc-710">지정하지 않으면 요청에 허용되는 시간이 제한되지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-710">If it is not specified, then the amount of time that a request is permitted to take is unlimited.</span></span> <span data-ttu-id="865dc-711">즉, 요청이 중단될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-711">In other words, the request might hang.</span></span> |
| <span data-ttu-id="865dc-712">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="865dc-712">ServerTimeout</span></span> | <span data-ttu-id="865dc-713">없음</span><span class="sxs-lookup"><span data-stu-id="865dc-713">None</span></span> | <span data-ttu-id="865dc-714">요청에 대한 서버 제한 시간 간격(값은 초로 반올림됨)입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-714">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="865dc-715">지정하지 않으면 서버에 대한 모든 요청에 기본값이 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-715">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="865dc-716">일반적으로 최상의 옵션은 서버 기본값이 사용되도록 이 설정을 생략하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-716">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="865dc-717">LocationMode</span><span class="sxs-lookup"><span data-stu-id="865dc-717">LocationMode</span></span> | <span data-ttu-id="865dc-718">없음</span><span class="sxs-lookup"><span data-stu-id="865dc-718">None</span></span> | <span data-ttu-id="865dc-719">RA-GRS(읽기 액세스 지역 중복 저장소) 복제 옵션을 사용하여 저장소 계정을 만드는 경우 위치 모드를 사용하여 요청을 수신할 위치를 나타낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-719">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="865dc-720">예를 들어 **PrimaryThenSecondary**를 지정하면 요청은 항상 기본 위치로 먼저 전송됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-720">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="865dc-721">요청이 실패하면 보조 위치로 전송됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-721">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="865dc-722">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="865dc-722">RetryPolicy</span></span> | <span data-ttu-id="865dc-723">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="865dc-723">ExponentialPolicy</span></span> | <span data-ttu-id="865dc-724">각 옵션에 대한 자세한 내용은 아래를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-724">See below for details of each option.</span></span> |

<span data-ttu-id="865dc-725">**지수 정책**</span><span class="sxs-lookup"><span data-stu-id="865dc-725">**Exponential policy**</span></span> 

| <span data-ttu-id="865dc-726">**설정**</span><span class="sxs-lookup"><span data-stu-id="865dc-726">**Setting**</span></span> | <span data-ttu-id="865dc-727">**기본값**</span><span class="sxs-lookup"><span data-stu-id="865dc-727">**Default value**</span></span> | <span data-ttu-id="865dc-728">**의미**</span><span class="sxs-lookup"><span data-stu-id="865dc-728">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="865dc-729">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="865dc-729">maxAttempt</span></span> | <span data-ttu-id="865dc-730">3</span><span class="sxs-lookup"><span data-stu-id="865dc-730">3</span></span> | <span data-ttu-id="865dc-731">재시도 횟수입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-731">Number of retry attempts.</span></span> |
| <span data-ttu-id="865dc-732">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-732">deltaBackoff</span></span> | <span data-ttu-id="865dc-733">4초</span><span class="sxs-lookup"><span data-stu-id="865dc-733">4 seconds</span></span> | <span data-ttu-id="865dc-734">재시도 사이의 백오프 간격입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-734">Back-off interval between retries.</span></span> <span data-ttu-id="865dc-735">임의 요소를 포함하여 이 timespan의 배수가 이후 재시도 횟수에 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-735">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="865dc-736">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-736">MinBackoff</span></span> | <span data-ttu-id="865dc-737">3초</span><span class="sxs-lookup"><span data-stu-id="865dc-737">3 seconds</span></span> | <span data-ttu-id="865dc-738">deltaBackoff에서 계산된 모든 다시 시도 간격에 추가됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-738">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="865dc-739">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-739">This value cannot be changed.</span></span>
| <span data-ttu-id="865dc-740">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-740">MaxBackoff</span></span> | <span data-ttu-id="865dc-741">120초</span><span class="sxs-lookup"><span data-stu-id="865dc-741">120 seconds</span></span> | <span data-ttu-id="865dc-742">계산된 다시 시도 간격이 MaxBackoff보다 큰 경우 MaxBackoff가 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-742">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="865dc-743">이 값은 변경할 수 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-743">This value cannot be changed.</span></span> |

<span data-ttu-id="865dc-744">**선형 정책**</span><span class="sxs-lookup"><span data-stu-id="865dc-744">**Linear policy**</span></span>

| <span data-ttu-id="865dc-745">**설정**</span><span class="sxs-lookup"><span data-stu-id="865dc-745">**Setting**</span></span> | <span data-ttu-id="865dc-746">**기본값**</span><span class="sxs-lookup"><span data-stu-id="865dc-746">**Default value**</span></span> | <span data-ttu-id="865dc-747">**의미**</span><span class="sxs-lookup"><span data-stu-id="865dc-747">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="865dc-748">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="865dc-748">maxAttempt</span></span> | <span data-ttu-id="865dc-749">3</span><span class="sxs-lookup"><span data-stu-id="865dc-749">3</span></span> | <span data-ttu-id="865dc-750">재시도 횟수입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-750">Number of retry attempts.</span></span> |
| <span data-ttu-id="865dc-751">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-751">deltaBackoff</span></span> | <span data-ttu-id="865dc-752">30초</span><span class="sxs-lookup"><span data-stu-id="865dc-752">30 seconds</span></span> | <span data-ttu-id="865dc-753">재시도 사이의 백오프 간격입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-753">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="865dc-754">재시도 사용 지침</span><span class="sxs-lookup"><span data-stu-id="865dc-754">Retry usage guidance</span></span>
<span data-ttu-id="865dc-755">저장소 클라이언트 API를 사용하여 Azure 저장소 서비스에 액세스하는 경우 다음 지침을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-755">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="865dc-756">요구 사항에 적합한 Microsoft.WindowsAzure.Storage.RetryPolicies 네임스페이스에서 기본 제공 재시도 정책을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-756">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="865dc-757">대부분의 경우 이러한 정책이면 충분합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-757">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="865dc-758">일괄 작업, 백그라운드 작업 또는 비대화형 시나리오에서 **ExponentialRetry** 정책을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-758">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="865dc-759">이러한 시나리오에서는 일반적으로 서비스가 복구되는 데 더 많은 시간을 허용하여 결과적으로 작업이 성공할 가능성이 증가합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-759">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="865dc-760">**RequestOptions** 매개 변수의 **MaximumExecutionTime** 속성을 지정하여 총 실행 시간을 제한하고 제한 시간 값을 선택할 때 작업의 유형 및 크기를 고려하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-760">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="865dc-761">사용자 지정 재시도를 구현해야 하는 경우 저장소 클라이언트 클래스 주위에 래퍼를 만들지 마세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-761">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="865dc-762">대신 **IExtendedRetryPolicy** 인터페이스를 통해 기존 정책을 확장하는 기능을 사용하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-762">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="865dc-763">RA-GRS(읽기 액세스 지역 중복 저장소)를 사용하는 경우 기본 액세스에 실패하면 **LocationMode**를 사용하여 재시도에서 저장소의 보조 읽기 전용 복사본에 액세스하도록 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-763">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="865dc-764">그러나 이 옵션을 사용할 때 기본 저장소에서 복제가 아직 완료되지 않은 경우 응용 프로그램이 오래된 데이터에서 성공적으로 작동하는지 확인해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-764">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="865dc-765">재시도 작업에 대해 다음 설정을 사용하여 시작하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-765">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="865dc-766">이러한 설정은 범용이므로 작업을 모니터링하고 고유한 시나리오에 맞게 값을 미세 조정해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-766">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="865dc-767">**컨텍스트**</span><span class="sxs-lookup"><span data-stu-id="865dc-767">**Context**</span></span> | <span data-ttu-id="865dc-768">**샘플 대상 E2E<br />최대 대기 시간**</span><span class="sxs-lookup"><span data-stu-id="865dc-768">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="865dc-769">**다시 시도 정책**</span><span class="sxs-lookup"><span data-stu-id="865dc-769">**Retry policy**</span></span> | <span data-ttu-id="865dc-770">**설정**</span><span class="sxs-lookup"><span data-stu-id="865dc-770">**Settings**</span></span> | <span data-ttu-id="865dc-771">**값**</span><span class="sxs-lookup"><span data-stu-id="865dc-771">**Values**</span></span> | <span data-ttu-id="865dc-772">**작동 방법**</span><span class="sxs-lookup"><span data-stu-id="865dc-772">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="865dc-773">대화형, UI</span><span class="sxs-lookup"><span data-stu-id="865dc-773">Interactive, UI,</span></span><br /><span data-ttu-id="865dc-774">또는 포그라운드</span><span class="sxs-lookup"><span data-stu-id="865dc-774">or foreground</span></span> |<span data-ttu-id="865dc-775">2초</span><span class="sxs-lookup"><span data-stu-id="865dc-775">2 seconds</span></span> |<span data-ttu-id="865dc-776">선형</span><span class="sxs-lookup"><span data-stu-id="865dc-776">Linear</span></span> |<span data-ttu-id="865dc-777">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="865dc-777">maxAttempt</span></span><br /><span data-ttu-id="865dc-778">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-778">deltaBackoff</span></span> |<span data-ttu-id="865dc-779">3</span><span class="sxs-lookup"><span data-stu-id="865dc-779">3</span></span><br /><span data-ttu-id="865dc-780">500ms</span><span class="sxs-lookup"><span data-stu-id="865dc-780">500 ms</span></span> |<span data-ttu-id="865dc-781">시도 1 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-781">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="865dc-782">시도 2 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-782">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="865dc-783">시도 3 - 500ms 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-783">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="865dc-784">백그라운드</span><span class="sxs-lookup"><span data-stu-id="865dc-784">Background</span></span><br /><span data-ttu-id="865dc-785">또는 일괄 처리</span><span class="sxs-lookup"><span data-stu-id="865dc-785">or batch</span></span> |<span data-ttu-id="865dc-786">30초</span><span class="sxs-lookup"><span data-stu-id="865dc-786">30 seconds</span></span> |<span data-ttu-id="865dc-787">지수</span><span class="sxs-lookup"><span data-stu-id="865dc-787">Exponential</span></span> |<span data-ttu-id="865dc-788">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="865dc-788">maxAttempt</span></span><br /><span data-ttu-id="865dc-789">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="865dc-789">deltaBackoff</span></span> |<span data-ttu-id="865dc-790">5</span><span class="sxs-lookup"><span data-stu-id="865dc-790">5</span></span><br /><span data-ttu-id="865dc-791">4초</span><span class="sxs-lookup"><span data-stu-id="865dc-791">4 seconds</span></span> |<span data-ttu-id="865dc-792">시도 1 - ~3초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-792">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="865dc-793">시도 2 - ~7초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-793">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="865dc-794">시도 3 - ~15초 지연</span><span class="sxs-lookup"><span data-stu-id="865dc-794">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="865dc-795">원격 분석</span><span class="sxs-lookup"><span data-stu-id="865dc-795">Telemetry</span></span>
<span data-ttu-id="865dc-796">재시도 횟수는 **TraceSource**에 기록됩니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-796">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="865dc-797">이벤트를 캡처하여 적합한 대상 로그에 기록하려면 **TraceListener**를 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-797">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="865dc-798">**TextWriterTraceListener** 또는 **XmlWriterTraceListener**를 사용하여 데이터를 로그 파일에 기록하거나, **EventLogTraceListener**를 사용하여 Windows 이벤트 로그에 기록하거나, **EventProviderTraceListener**를 사용하여 추적 데이터를 ETW 하위 시스템에 기록할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-798">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="865dc-799">버퍼의 자동 플러시 및 기록될 이벤트의 자세한 정도(예: 오류, 경고, 정보 및 세부 정보 표시)를 구성할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-799">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="865dc-800">자세한 내용은 [.NET Storage 클라이언트 라이브러리를 사용한 클라이언트 쪽 로깅](http://msdn.microsoft.com/library/azure/dn782839.aspx)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-800">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="865dc-801">작업은 **OperationContext** 인스턴스를 수신하여 사용자 지정 원격 분석 논리를 추가하는 데 사용할 수 있는 **Retrying** 이벤트를 표시할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-801">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="865dc-802">자세한 내용은 [OperationContext.Retrying 이벤트](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-802">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="865dc-803">예</span><span class="sxs-lookup"><span data-stu-id="865dc-803">Examples</span></span>
<span data-ttu-id="865dc-804">다음 코드 예제에서는 서로 다른 재시도 설정을 사용하여 대화형 요청과 백그라운드 요청에 대해 각각 하나씩 두 개의 **TableRequestOptions** 인스턴스를 만드는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-804">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="865dc-805">이 예제에서는 클라이언트에서 이러한 두 재시도 정책을 설정하여 모든 요청에 대해 적용하고 특정 요청에서 대화형 전략을 설정하여 클라이언트에 적용된 기본 설정을 재정의합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-805">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="865dc-806">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="865dc-806">More information</span></span>
* [<span data-ttu-id="865dc-807">Azure Storage 클라이언트 라이브러리 재시도 정책 권장 사항(영문)</span><span class="sxs-lookup"><span data-stu-id="865dc-807">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="865dc-808">Storage 클라이언트 라이브러리 2.0 – 재시도 정책 구현(영문)</span><span class="sxs-lookup"><span data-stu-id="865dc-808">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="865dc-809">일반 REST 및 다시 시도 지침</span><span class="sxs-lookup"><span data-stu-id="865dc-809">General REST and retry guidelines</span></span>
<span data-ttu-id="865dc-810">Azure 또는 타사 서비스에 액세스하는 경우 다음 사항을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-810">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="865dc-811">재사용 가능한 코드로 재시도를 관리하는 체계적인 방법을 사용하여 모든 클라이언트 및 모든 솔루션에서 일관된 방법론을 적용할 수 있도록 합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-811">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="865dc-812">대상 서비스 또는 클라이언트에 기본 제공 재시도 메커니즘이 없는 경우 [Polly][polly]와 같은 재시도 프레임워크를 사용하여 재시도를 관리하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-812">Consider using a retry framework such as [Polly][polly] to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="865dc-813">그러면 일관된 재시도 동작을 구현하는 데 도움이 되며 대상 서비스에 적합한 기본 재시도 전략을 제공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-813">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="865dc-814">그러나 비표준 동작을 사용하는 서비스, 일시적인 오류를 나타내는 데 예외를 사용하지 않는 서비스 또는 **Retry-Response** 회신을 사용하여 재시도 동작을 관리하려는 경우 사용자 지정 재시도 코드를 만들어야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-814">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="865dc-815">일시적인 검색 논리는 REST를 호출하는 데 사용하는 실제 클라이언트 API에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-815">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="865dc-816">최신 **HttpClient** 클래스와 같은 일부 클라이언트는 성공이 아닌 HTTP 상태 코드를 사용하여 완료된 요청에 대한 예외를 throw하지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-816">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> 
* <span data-ttu-id="865dc-817">서비스에서 반환된 HTTP 상태 코드는 오류가 일시적인지 여부를 나타내는 데 도움이 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-817">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="865dc-818">클라이언트 또는 재시도 프레임워크에서 생성된 예외를 검사하여 상태 코드에 액세스하거나 해당되는 예외 유형을 확인해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-818">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="865dc-819">다음 HTTP 코드는 일반적으로 재시도가 적합함을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-819">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="865dc-820">408 요청 시간 초과</span><span class="sxs-lookup"><span data-stu-id="865dc-820">408 Request Timeout</span></span>
  * <span data-ttu-id="865dc-821">429 요청이 너무 많음</span><span class="sxs-lookup"><span data-stu-id="865dc-821">429 Too Many Requests</span></span>
  * <span data-ttu-id="865dc-822">500 내부 서버 오류</span><span class="sxs-lookup"><span data-stu-id="865dc-822">500 Internal Server Error</span></span>
  * <span data-ttu-id="865dc-823">502 잘못된 게이트웨이</span><span class="sxs-lookup"><span data-stu-id="865dc-823">502 Bad Gateway</span></span>
  * <span data-ttu-id="865dc-824">503 서비스를 사용할 수 없음</span><span class="sxs-lookup"><span data-stu-id="865dc-824">503 Service Unavailable</span></span>
  * <span data-ttu-id="865dc-825">504 게이트웨이 시간 초과</span><span class="sxs-lookup"><span data-stu-id="865dc-825">504 Gateway Timeout</span></span>
* <span data-ttu-id="865dc-826">재시도 논리가 예외를 기반으로 하는 경우 다음은 일반적으로 연결을 설정할 수 없는 일시적인 오류를 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-826">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="865dc-827">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="865dc-827">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="865dc-828">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="865dc-828">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="865dc-829">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="865dc-829">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="865dc-830">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="865dc-830">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="865dc-831">서비스를 사용할 수 없음 상태의 경우 서비스는 **Retry-After** 응답 헤더 또는 다른 사용자 지정 헤더에서 재시도하기 전에 적절한 지연 시간을 나타낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-831">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="865dc-832">또한 서비스는 추가 정보를 사용자 지정 헤더로 전송하거나 응답의 내용에 포함할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-832">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> 
* <span data-ttu-id="865dc-833">408 요청 시간 초과를 제외한 클라이언트 오류(4xx 범위의 오류)를 나타내는 상태 코드에 대해서는 재시도하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="865dc-833">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="865dc-834">서로 다른 네트워크 상태 및 다양한 시스템 부하와 같은 다양한 조건에서 재시도 전략 및 메커니즘을 철저히 테스트합니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-834">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="865dc-835">재시도 전략</span><span class="sxs-lookup"><span data-stu-id="865dc-835">Retry strategies</span></span>
<span data-ttu-id="865dc-836">다음은 일반적인 재시도 전략 간격의 유형입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-836">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="865dc-837">**지수**.</span><span class="sxs-lookup"><span data-stu-id="865dc-837">**Exponential**.</span></span> <span data-ttu-id="865dc-838">재시도 사이의 간격을 결정하는 무작위 추출 지수 백오프 방법을 사용하여 지정된 횟수의 재시도를 수행하는 재시도 정책입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-838">A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="865dc-839">예: </span><span class="sxs-lookup"><span data-stu-id="865dc-839">For example:</span></span>

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* <span data-ttu-id="865dc-840">**증분**.</span><span class="sxs-lookup"><span data-stu-id="865dc-840">**Incremental**.</span></span> <span data-ttu-id="865dc-841">재시도 횟수가 지정되고 재시도 간의 시간 간격이 증분되는 재시도 전략입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-841">A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="865dc-842">예: </span><span class="sxs-lookup"><span data-stu-id="865dc-842">For example:</span></span>

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* <span data-ttu-id="865dc-843">**LinearRetry**.</span><span class="sxs-lookup"><span data-stu-id="865dc-843">**LinearRetry**.</span></span> <span data-ttu-id="865dc-844">재시도 간에 지정된 고정 시간 간격을 사용하여 지정된 횟수의 재시도를 수행하는 재시도 정책입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-844">A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="865dc-845">예: </span><span class="sxs-lookup"><span data-stu-id="865dc-845">For example:</span></span>

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="865dc-846">Polly를 통한 일시적인 오류 처리</span><span class="sxs-lookup"><span data-stu-id="865dc-846">Transient fault handling with Polly</span></span>
<span data-ttu-id="865dc-847">[Polly][polly]는 재시도 및 [회로 차단기][circuit-breaker] 전략을 프로그래밍 방식으로 처리하는 라이브러리입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-847">[Polly][polly] is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="865dc-848">Polly 프로젝트는 [.NET Foundation][dotnet-foundation]의 멤버입니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-848">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="865dc-849">Polly는 클라이언트가 기본적으로 재시도를 지원하지 않는 서비스의 유효한 대안이 되며 정확한 구현이 까다로운 사용자 지정 재시도 코드를 작성할 필요가 없습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-849">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="865dc-850">Polly는 발생한 오류를 추적하는 방법도 제공하므로 재시도를 기록할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="865dc-850">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>


<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx


### <a name="more-information"></a><span data-ttu-id="865dc-854">자세한 정보</span><span class="sxs-lookup"><span data-stu-id="865dc-854">More information</span></span>
* [<span data-ttu-id="865dc-855">연결 복원력</span><span class="sxs-lookup"><span data-stu-id="865dc-855">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="865dc-856">데이터 요소 - EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="865dc-856">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)


