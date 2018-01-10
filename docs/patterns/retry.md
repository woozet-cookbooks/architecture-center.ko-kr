---
title: Retry
description: "이전에 실패한 작업을 투명하게 다시 시도하여 서비스 또는 네트워크 리소스에 연결하려 할 때 응용 프로그램을 사용하여 예상된 일시적 오류를 처리합니다."
keywords: "디자인 패턴"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: resiliency
ms.openlocfilehash: 6c02b384e71c068ecbc78f3170d28cea406538e2
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="retry-pattern"></a><span data-ttu-id="b92a5-104">다시 시도 패턴</span><span class="sxs-lookup"><span data-stu-id="b92a5-104">Retry pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="b92a5-105">실패한 작업을 다시 시도하여 서비스 또는 네트워크 리소스에 연결하려 할 때 응용 프로그램을 사용하여 일시적 오류를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-105">Enable an application to handle transient failures when it tries to connect to a service or network resource, by transparently retrying a failed operation.</span></span> <span data-ttu-id="b92a5-106">이렇게 하면 응용 프로그램의 안정성을 개선할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-106">This can improve the stability of the application.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="b92a5-107">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="b92a5-107">Context and problem</span></span>

<span data-ttu-id="b92a5-108">클라우드에서 실행되는 요소와 통신하는 응용 프로그램은 이 환경에서 발생할 수 있는 일시적인 오류에 민감해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-108">An application that communicates with elements running in the cloud has to be sensitive to the transient faults that can occur in this environment.</span></span> <span data-ttu-id="b92a5-109">오류로는 구성 요소 및 서비스의 순간적인 네트워크 연결 끊김, 서비스의 일시적인 사용 중단, 서비스가 사용 중일 때 발생하는 시간 제한 등이 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-109">Faults include the momentary loss of network connectivity to components and services, the temporary unavailability of a service, or timeouts that occur when a service is busy.</span></span>

<span data-ttu-id="b92a5-110">이러한 오류는 일반적으로 자체적으로 수정되고, 잠시 후 오류를 발생시킨 동작을 반복하면 성공할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-110">These faults are typically self-correcting, and if the action that triggered a fault is repeated after a suitable delay it's likely to be successful.</span></span> <span data-ttu-id="b92a5-111">예를 들어, 많은 수의 동시 요청을 처리하는 데이터베이스 서비스는 워크로드가 줄어들 때까지 추가 요청을 일시적으로 거부하는 제한 전략을 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-111">For example, a database service that's processing a large number of concurrent requests can implement a throttling strategy that temporarily rejects any further requests until its workload has eased.</span></span> <span data-ttu-id="b92a5-112">데이터베이스에 액세스하려는 응용 프로그램이 연결하지 못할 수 있지만 잠시 후 다시 시도하면 성공할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-112">An application trying to access the database might fail to connect, but if it tries again after a delay it might succeed.</span></span>

## <a name="solution"></a><span data-ttu-id="b92a5-113">해결 방법</span><span class="sxs-lookup"><span data-stu-id="b92a5-113">Solution</span></span>

<span data-ttu-id="b92a5-114">클라우드에서 일시적인 오류는 흔한 것이 아니며 응용 프로그램은 이들 오류를 깔끔하고 투명하게 처리하도록 설계해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-114">In the cloud, transient faults aren't uncommon and an application should be designed to handle them elegantly and transparently.</span></span> <span data-ttu-id="b92a5-115">이렇게 하면 응용 프로그램이 수행하는 비즈니스 작업에서 발생할 수 있는 오류에 미치는 영향을 최소화합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-115">This minimizes the effects faults can have on the business tasks the application is performing.</span></span>

<span data-ttu-id="b92a5-116">응용 프로그램이 원격 서비스에 요청을 보내려 할 때 오류를 감지한 경우 다음 전략을 통해 오류를 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-116">If an application detects a failure when it tries to send a request to a remote service, it can handle the failure using the following strategies:</span></span>

- <span data-ttu-id="b92a5-117">**취소**.</span><span class="sxs-lookup"><span data-stu-id="b92a5-117">**Cancel**.</span></span> <span data-ttu-id="b92a5-118">오류가 일시적이지 않거나 반복해도 성공 가능성이 없는 것으로 나타나는 오류의 경우, 응용 프로그램은 작업을 취소하고 예외를 보고해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-118">If the fault indicates that the failure isn't transient or is unlikely to be successful if repeated, the application should cancel the operation and report an exception.</span></span> <span data-ttu-id="b92a5-119">예를 들어, 잘못된 자격 증명을 제공하여 발생한 인증 실패는 여러 번 시동해도 성공할 가능성이 없습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-119">For example, an authentication failure caused by providing invalid credentials is not likely to succeed no matter how many times it's attempted.</span></span>

- <span data-ttu-id="b92a5-120">**다시 시도**.</span><span class="sxs-lookup"><span data-stu-id="b92a5-120">**Retry**.</span></span> <span data-ttu-id="b92a5-121">보고된 특정 오류가 비정상적이거나 드문 것인 경우 이는 전송하는 동안 손상되는 네트워크 패킷과 같은 특이한 상황으로 인해 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-121">If the specific fault reported is unusual or rare, it might have been caused by unusual circumstances such as a network packet becoming corrupted while it was being transmitted.</span></span> <span data-ttu-id="b92a5-122">이 경우 동일한 오류가 반복될 가능성이 없고 요청이 성공할 가능성이 높기 때문에 응용 프로그램은 즉시 실패한 요청을 다시 시도합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-122">In this case, the application could retry the failing request again immediately because the same failure is unlikely to be repeated and the request will probably be successful.</span></span>

- <span data-ttu-id="b92a5-123">**잠시 후 다시 시도.**</span><span class="sxs-lookup"><span data-stu-id="b92a5-123">**Retry after delay.**</span></span> <span data-ttu-id="b92a5-124">오류가 보다 일반적인 연결 또는 잦은 오류 중 하나에 의해 발생하는 경우 네트워크 또는 서비스는 연결 문제를 수정하거나 작업의 백로그를 해소하는 동안 잠깐의 시간이 필요할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-124">If the fault is caused by one of the more commonplace connectivity or busy failures, the network or service might need a short period while the connectivity issues are corrected or the backlog of work is cleared.</span></span> <span data-ttu-id="b92a5-125">응용 프로그램은 요청을 다시 시도하기 전에 잠시 기다려야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-125">The application should wait for a suitable time before retrying the request.</span></span>

<span data-ttu-id="b92a5-126">보다 일반적인 일시적 오류의 경우, 다시 시도의 시간 간격을 선택하여 응용 프로그램의 여러 인스턴스 요청을 최대한 균등하게 분배해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-126">For the more common transient failures, the period between retries should be chosen to spread requests from multiple instances of the application as evenly as possible.</span></span> <span data-ttu-id="b92a5-127">이렇게 하면 사용 중인 서비스의 과부하가 지속되는 가능성을 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-127">This reduces the chance of a busy service continuing to be overloaded.</span></span> <span data-ttu-id="b92a5-128">응용 프로그램의 여러 인스턴스 다시 시도 요청으로 서비스 과부하가 걸리는 경우 서비스를 복구하는 데 더 오랜 시간이 걸립니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-128">If many instances of an application are continually overwhelming a service with retry requests, it'll take the service longer to recover.</span></span>

<span data-ttu-id="b92a5-129">요청이 여전히 실패하면 응용 프로그램은 기다렸다가 다시 시도할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-129">If the request still fails, the application can wait and make another attempt.</span></span> <span data-ttu-id="b92a5-130">필요한 경우 최대 요청 시도 횟수에 도달할 때까지 다시 시도 간 지연 시간을 늘려 이 프로세스를 반복할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-130">If necessary, this process can be repeated with increasing delays between retry attempts, until some maximum number of requests have been attempted.</span></span> <span data-ttu-id="b92a5-131">지연 시간은 오류 유형과 이 지연 시간 동안 수정할 수 있을 가능성에 따라 서서히 또는 대폭 늘릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-131">The delay can be increased incrementally or exponentially, depending on the type of failure and the probability that it'll be corrected during this time.</span></span>

<span data-ttu-id="b92a5-132">다음 다이어그램은 이 패턴을 사용하여 호스티드 서비스에서 작업을 호출하는 방법을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-132">The following diagram illustrates invoking an operation in a hosted service using this pattern.</span></span> <span data-ttu-id="b92a5-133">미리 정의된 횟수의 시도 후 요청이 실패하는 경우 응용 프로그램은 오류를 예외로 처리하고 적절히 처리해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-133">If the request is unsuccessful after a predefined number of attempts, the application should treat the fault as an exception and handle it accordingly.</span></span>

![그림 1- 다시 시도 패턴을 사용하여 호스티드 서비스에서 작업 호출](./_images/retry-pattern.png)

<span data-ttu-id="b92a5-135">응용 프로그램은 나열된 전략 중 하나와 일치하는 다시 시도 정책을 구현하는 코드에서 원격 서비스에 액세스하려는 모든 시도를 래핑해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-135">The application should wrap all attempts to access a remote service in code that implements a retry policy matching one of the strategies listed above.</span></span> <span data-ttu-id="b92a5-136">여러 서비스에 전송된 요청은 서로 다른 정책이 적용될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-136">Requests sent to different services can be subject to different policies.</span></span> <span data-ttu-id="b92a5-137">일부 공급업체는 응용 프로그램이 최대 다시 시도 횟수, 다시 시도 시간 간격 및 기타 매개 변수를 지정할 수 있는 다시 시도 정책을 구현하는 라이브러리를 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-137">Some vendors provide libraries that implement retry policies, where the application can specify the maximum number of retries, the time between retry attempts, and other parameters.</span></span>

<span data-ttu-id="b92a5-138">응용 프로그램은 오류 및 실패 작업의 세부 정보를 기록해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-138">An application should log the details of faults and failing operations.</span></span> <span data-ttu-id="b92a5-139">이 정보는 작업자에게 유용 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-139">This information is useful to operators.</span></span> <span data-ttu-id="b92a5-140">서비스를 자주 사용할 수 없거나 사용 중인 경우는 서비스가 종종 해당 리소스를 모두 사용했기 때문에 발생합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-140">If a service is frequently unavailable or busy, it's often because the service has exhausted its resources.</span></span> <span data-ttu-id="b92a5-141">서비스를 확장하여 이러한 오류 발생 빈도를 줄일 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-141">You can reduce the frequency of these faults by scaling out the service.</span></span> <span data-ttu-id="b92a5-142">예를 들어 데이터베이스 서비스가 지속적으로 과부하가 걸리는 경우 데이터베이스를 분할하고 여러 서버에 부하를 분산하는 것이 유용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-142">For example, if a database service is continually overloaded, it might be beneficial to partition the database and spread the load across multiple servers.</span></span>

> <span data-ttu-id="b92a5-143">[Microsoft Entity Framework](https://docs.microsoft.com/ef/)는 데이터베이스 작업 다시 시도를 위한 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-143">[Microsoft Entity Framework](https://docs.microsoft.com/ef/) provides facilities for retrying database operations.</span></span> <span data-ttu-id="b92a5-144">또한 대부분의 Azure 서비스 및 클라이언트 SDK는 다시 시도 메커니즘을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-144">Also, most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="b92a5-145">자세한 내용은 [특정 서비스에 대한 다시 시도 지침](https://docs.microsoft.com/en-us/azure/architecture/best-practices/retry-service-specific)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b92a5-145">For more information, see [Retry guidance for specific services](https://docs.microsoft.com/en-us/azure/architecture/best-practices/retry-service-specific).</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="b92a5-146">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="b92a5-146">Issues and considerations</span></span>

<span data-ttu-id="b92a5-147">이 패턴을 구현할 방법을 결정할 때 다음 사항을 고려해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-147">You should consider the following points when deciding how to implement this pattern.</span></span>

<span data-ttu-id="b92a5-148">다시 시도 정책은 응용 프로그램의 비즈니스 요구 사항 및 장애의 특성에 맞게 튜닝해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-148">The retry policy should be tuned to match the business requirements of the application and the nature of the failure.</span></span> <span data-ttu-id="b92a5-149">중요하지 않은 일부 작업의 경우 여러 번 다시 시도하는 것보다 빠르게 실패로 처리하여 응용 프로그램의 처리량에 영향을 주지 않도록 하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-149">For some noncritical operations, it's better to fail fast rather than retry several times and impact the throughput of the application.</span></span> <span data-ttu-id="b92a5-150">예를 들어 원격 서비스에 액세스하는 대화형 웹 응용 프로그램의 경우, 다시 시도 간 지연 시간을 짧게 하여 몇 번의 다시 시도 후 실패로 처리하고 사용자에게 적절한 메시지(예: "나중에 다시 시도하십시오.")를 표시하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-150">For example, in an interactive web application accessing a remote service, it's better to fail after a smaller number of retries with only a short delay between retry attempts, and display a suitable message to the user (for example, “please try again later”).</span></span> <span data-ttu-id="b92a5-151">배치 응용 프로그램의 경우 시도 간 지연 시간을 대폭 늘려 다시 시도 횟수를 증가시키는 것이 더 적절할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-151">For a batch application, it might be more appropriate to increase the number of retry attempts with an exponentially increasing delay between attempts.</span></span>

<span data-ttu-id="b92a5-152">시도간 지연 시간을 최소화한 적극적인 다시 시도 정책과 많은 다시 시도 횟수는 최대 용량에 근접하거나 최대 용량으로 실행 중인 서비스를 저하시킬 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-152">An aggressive retry policy with minimal delay between attempts, and a large number of retries, could further degrade a busy service that's running close to or at capacity.</span></span> <span data-ttu-id="b92a5-153">계속해서 실패한 작업을 수행하려고 시도하려는 경우 이 다시 시도 정책은 응용 프로그램의 응답성에 영향을 줄 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-153">This retry policy could also affect the responsiveness of the application if it's continually trying to perform a failing operation.</span></span>

<span data-ttu-id="b92a5-154">많은 여러 번 다시 시도한 후에도 요청이 계속 실패하는 경우, 응용 프로그램이 이후 요청이 동일한 리소스로 가지 않도록 하고 즉시 실패를 간단히 보고하도록 하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-154">If a request still fails after a significant number of retries, it's better for the application to prevent further requests going to the same resource and simply report a failure immediately.</span></span> <span data-ttu-id="b92a5-155">기간이 만료되면 응용 프로그램은 한 번 이상의 요청을 임시로 허용하여 요청이 성공했는지 확인할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-155">When the period expires, the application can tentatively allow one or more requests through to see whether they're successful.</span></span> <span data-ttu-id="b92a5-156">이 전략의 자세한 내용은 [회로 차단기 패턴](circuit-breaker.md)를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b92a5-156">For more details of this strategy, see the [Circuit Breaker pattern](circuit-breaker.md).</span></span>

<span data-ttu-id="b92a5-157">작업이 idempotent인지 여부를 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-157">Consider whether the operation is idempotent.</span></span> <span data-ttu-id="b92a5-158">idempotent인 경우 다시 시도해도 안전합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-158">If so, it's inherently safe to retry.</span></span> <span data-ttu-id="b92a5-159">그렇지 않은 경우 다시 시도하면 작업이 여러 번 실행되고 의도하지 않은 부작용이 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-159">Otherwise, retries could cause the operation to be executed more than once, with unintended side effects.</span></span> <span data-ttu-id="b92a5-160">예를 들어 서비스가 요청을 수신하고 요청을 성공적으로 처리하지만, 응답을 보내지 못할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-160">For example, a service might receive the request, process the request successfully, but fail to send a response.</span></span> <span data-ttu-id="b92a5-161">이 경우 재시도 논리는 첫 번째 요청이 수신되지 않은 것으로 가정하고 요청을 다시 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-161">At that point, the retry logic might re-send the request, assuming that the first request wasn't received.</span></span>

<span data-ttu-id="b92a5-162">서비스에 대한 요청은 오류의 성격에 따라 다른 예외에서 발생하는 다양한 원인으로 실패할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-162">A request to a service can fail for a variety of reasons raising different exceptions depending on the nature of the failure.</span></span> <span data-ttu-id="b92a5-163">일부 예외는 신속하게 해결할 수 있는 오류라고 나타내는 반면 다른 예외는 오류가 더 오래 지속될 것이라고 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-163">Some exceptions indicate a failure that can be resolved quickly, while others indicate that the failure is longer lasting.</span></span> <span data-ttu-id="b92a5-164">예외 형식에 따라 다시 시도 간의 시간을 조정하는 다시 시도 정책에 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-164">It's useful for the retry policy to adjust the time between retry attempts based on the type of the exception.</span></span>

<span data-ttu-id="b92a5-165">트랜잭션의 일부인 작업을 다시 시도하는 것이 전체 트랜잭션 일관성에 어떻게 영향을 주는지 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-165">Consider how retrying an operation that's part of a transaction will affect the overall transaction consistency.</span></span> <span data-ttu-id="b92a5-166">성공 가능성을 최대화하고 모든 트랜잭션 단계를 취소할 필요를 줄이기 위해 트랜잭션 작업을 다시 시도 정책을 세부 조정합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-166">Fine tune the retry policy for transactional operations to maximize the chance of success and reduce the need to undo all the transaction steps.</span></span>

<span data-ttu-id="b92a5-167">다양한 오류 상태에 대해 다시 시도 코드를 모두 테스트되는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-167">Ensure that all retry code is fully tested against a variety of failure conditions.</span></span> <span data-ttu-id="b92a5-168">응용 프로그램의 성능이 나 안정성에 심각하게 영향을 주지 않는지, 서비스와 리소스에 과도한 부하를 주지 않는지, 경합 상태 또는 병목 현상이 발생하지 않는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-168">Check that it doesn't severely impact the performance or reliability of the application, cause excessive load on services and resources, or generate race conditions or bottlenecks.</span></span>

<span data-ttu-id="b92a5-169">실패한 작업의 전체 컨텍스트를 이해하는 위치에서만 재시도 논리를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-169">Implement retry logic only where the full context of a failing operation is understood.</span></span> <span data-ttu-id="b92a5-170">예를 들어 다시 시도 정책이 다시 시도 정책을 포함하는 또 다른 작업을 호출하는 경우, 이 다시 시도 추가 계층으로 처리하는 데 더욱 오래 걸릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-170">For example, if a task that contains a retry policy invokes another task that also contains a retry policy, this extra layer of retries can add long delays to the processing.</span></span> <span data-ttu-id="b92a5-171">이 경우 빠르게 실패로 처리하고 호출한 작업에 실패한 이유를 보고하도록 하위 수준 작업을 구성하는 것이 더 나을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-171">It might be better to configure the lower-level task to fail fast and report the reason for the failure back to the task that invoked it.</span></span> <span data-ttu-id="b92a5-172">그런 다음 이 상위 수준 작업은 자체 정책에 따라 실패를 처리할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-172">This higher-level task can then handle the failure based on its own policy.</span></span>

<span data-ttu-id="b92a5-173">응용 프로그램, 서비스 또는 리소스와 관련된 기본 문제를 식별할 수 있도록 다시 시도를 발생시키는 모든 연결 실패를 기록하는 것이 중요합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-173">It's important to log all connectivity failures that cause a retry so that underlying problems with the application, services, or resources can be identified.</span></span>

<span data-ttu-id="b92a5-174">서비스 또는 리소스에 발생할 수 있는 오류를 조사하여 오류가 오래 지속되거나 터미널이 될 가능성이 있는지 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-174">Investigate the faults that are most likely to occur for a service or a resource to discover if they're likely to be long lasting or terminal.</span></span> <span data-ttu-id="b92a5-175">오류인 경우 해당 오류를 예외로 처리하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-175">If they are, it's better to handle the fault as an exception.</span></span> <span data-ttu-id="b92a5-176">응용 프로그램은 예외를 보고하거나 기록한 다음, 대체 서비스(있는 경우)를 호출하거나 기능을 저하시켜 계속할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-176">The application can report or log the exception, and then try to continue either by invoking an alternative service (if one is available), or by offering degraded functionality.</span></span> <span data-ttu-id="b92a5-177">오래 지속되는 오류를 검색하고 처리하는 방법에 대한 자세한 내용은 [회로 차단기 패턴](circuit-breaker.md)을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="b92a5-177">For more information on how to detect and handle long-lasting faults, see the [Circuit Breaker pattern](circuit-breaker.md).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="b92a5-178">이 패턴을 사용해야 하는 경우</span><span class="sxs-lookup"><span data-stu-id="b92a5-178">When to use this pattern</span></span>

<span data-ttu-id="b92a5-179">원격 서비스와 상호 작용하거나 원격 리소스에 액세스할 때 응용 프로그램에서 일시적인 오류를 발생할 수 있는 경우 이 패턴을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-179">Use this pattern when an application could experience transient faults as it interacts with a remote service or accesses a remote resource.</span></span> <span data-ttu-id="b92a5-180">이러한 오류는 단기간만 존재할 것이며, 이전에 실패한 요청을 반복하면 이후 시도에서 성공할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-180">These faults are expected to be short lived, and repeating a request that has previously failed could succeed on a subsequent attempt.</span></span>

<span data-ttu-id="b92a5-181">이 패턴은 다음과 같은 경우 유용하지 않을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-181">This pattern might not be useful:</span></span>

- <span data-ttu-id="b92a5-182">응용 프로그램의 응답성에 영향을 줄 수 때문에 오류가 오래 지속될 것 같은 경우.</span><span class="sxs-lookup"><span data-stu-id="b92a5-182">When a fault is likely to be long lasting, because this can affect the responsiveness of an application.</span></span> <span data-ttu-id="b92a5-183">응용 프로그램은 실패할 가능성이 있는 요청을 반복하여 시간 및 리소스를 낭비할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-183">The application might be wasting time and resources trying to repeat a request that's likely to fail.</span></span>
- <span data-ttu-id="b92a5-184">응용 프로그램의 비즈니스 논리의 오류로 인해 발생한 내부 예외 등 일시적인 오류로 인해 발생하지 않은 실패를 처리하는 경우.</span><span class="sxs-lookup"><span data-stu-id="b92a5-184">For handling failures that aren't due to transient faults, such as internal exceptions caused by errors in the business logic of an application.</span></span>
- <span data-ttu-id="b92a5-185">시스템에서 확장성 문제를 해결하는 대체 패턴으로 사용.</span><span class="sxs-lookup"><span data-stu-id="b92a5-185">As an alternative to addressing scalability issues in a system.</span></span> <span data-ttu-id="b92a5-186">응용 프로그램에서 자주 사용 중 오류가 발생하는 경우 이는 액세스하는 서비스 또는 리소스를 확장해야 함을 의미하기도 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-186">If an application experiences frequent busy faults, it's often a sign that the service or resource being accessed should be scaled up.</span></span>

## <a name="example"></a><span data-ttu-id="b92a5-187">예</span><span class="sxs-lookup"><span data-stu-id="b92a5-187">Example</span></span>

<span data-ttu-id="b92a5-188">이 예제에서는 C#가 다시 시도 패턴의 구현을 보여줍니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-188">This example in C# illustrates an implementation of the Retry pattern.</span></span> <span data-ttu-id="b92a5-189">아래에 표시된 `OperationWithBasicRetryAsync` 메서드는 `TransientOperationAsync` 메서드를 통해 외주 서비스를 비동기적으로 호출합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-189">The `OperationWithBasicRetryAsync` method, shown below, invokes an external service asynchronously through the `TransientOperationAsync` method.</span></span> <span data-ttu-id="b92a5-190">`TransientOperationAsync` 메서드에 대한 세부 정보는 서비스마다 고유하며, 샘플 코드에서 생략됩니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-190">The details of the `TransientOperationAsync` method will be specific to the service and are omitted from the sample code.</span></span>

```csharp
private int retryCount = 3;
private readonly TimeSpan delay = TimeSpan.FromSeconds(5);

public async Task OperationWithBasicRetryAsync()
{
  int currentRetry = 0;

  for (;;)
  {
    try
    {
      // Call external service.
      await TransientOperationAsync();

      // Return or break.
      break;
    }
    catch (Exception ex)
    {
      Trace.TraceError("Operation Exception");

      currentRetry++;

      // Check if the exception thrown was a transient exception
      // based on the logic in the error detection strategy.
      // Determine whether to retry the operation, as well as how
      // long to wait, based on the retry strategy.
      if (currentRetry > this.retryCount || !IsTransient(ex))
      {
        // If this isn't a transient error or we shouldn't retry, 
        // rethrow the exception.
        throw;
      }
    }

    // Wait to retry the operation.
    // Consider calculating an exponential delay here and
    // using a strategy best suited for the operation and fault.
    await Task.Delay(delay);
  }
}

// Async method that wraps a call to a remote service (details not shown).
private async Task TransientOperationAsync()
{
  ...
}
```

<span data-ttu-id="b92a5-191">이 메서드를 호출하는 문은 For 루프에 래핑된 Try/Catch 블록에 포함되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-191">The statement that invokes this method is contained in a try/catch block wrapped in a for loop.</span></span> <span data-ttu-id="b92a5-192">`TransientOperationAsync` 메서드에 대한 호출이 예외를 발생시키지 않고 성공하면 For 루프가 종료됩니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-192">The for loop exits if the call to the `TransientOperationAsync` method succeeds without throwing an exception.</span></span> <span data-ttu-id="b92a5-193">`TransientOperationAsync` 메서드가 실패하는 경우 Catch 블록에서 실패 원인을 검사합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-193">If the `TransientOperationAsync` method fails, the catch block examines the reason for the failure.</span></span> <span data-ttu-id="b92a5-194">일시적인 오류라고 판단되면 코드는 작업을 다시 시도하기 전에 잠시 대기합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-194">If it's believed to be a transient error the code waits for a short delay before retrying the operation.</span></span>

<span data-ttu-id="b92a5-195">For 루프도 작업을 시도한 횟수를 추적하며 코드가 세 번 실패한 경우 보다 오래 지속되는 예외로 간주합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-195">The for loop also tracks the number of times that the operation has been attempted, and if the code fails three times the exception is assumed to be more long lasting.</span></span> <span data-ttu-id="b92a5-196">예외가 일시적이지 않거나 오래 지속되는 경우 Catch 처리기는 예외를 발생시킵니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-196">If the exception isn't transient or it's long lasting, the catch handler throws an exception.</span></span> <span data-ttu-id="b92a5-197">이 예외는 For 루프를 종료하고 `OperationWithBasicRetryAsync` 메서드를 호출하는 코드에 의해 처리되어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-197">This exception exits the for loop and should be caught by the code that invokes the `OperationWithBasicRetryAsync` method.</span></span>

<span data-ttu-id="b92a5-198">아래 표시된 `IsTransient` 메서드는 코드가 실행되고 있는 환경과 관련된 특정 예외 집합을 확인합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-198">The `IsTransient` method, shown below, checks for a specific set of exceptions that are relevant to the environment the code is run in.</span></span> <span data-ttu-id="b92a5-199">일시적인 예외의 정의는 액세스중인 리소스와 작업이 수행되고 있는 환경에 따라 다릅니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-199">The definition of a transient exception will vary according to the resources being accessed and the environment the operation is being performed in.</span></span>

```csharp
private bool IsTransient(Exception ex)
{
  // Determine if the exception is transient.
  // In some cases this is as simple as checking the exception type, in other
  // cases it might be necessary to inspect other properties of the exception.
  if (ex is OperationTransientException)
    return true;

  var webException = ex as WebException;
  if (webException != null)
  {
    // If the web exception contains one of the following status values
    // it might be transient.
    return new[] {WebExceptionStatus.ConnectionClosed,
                  WebExceptionStatus.Timeout,
                  WebExceptionStatus.RequestCanceled }.
            Contains(webException.Status);
  }

  // Additional exception checking logic goes here.
  return false;
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="b92a5-200">관련 패턴 및 지침</span><span class="sxs-lookup"><span data-stu-id="b92a5-200">Related patterns and guidance</span></span>

- <span data-ttu-id="b92a5-201">[회로 차단기 패턴](circuit-breaker.md).</span><span class="sxs-lookup"><span data-stu-id="b92a5-201">[Circuit Breaker pattern](circuit-breaker.md).</span></span> <span data-ttu-id="b92a5-202">다시 시도 패턴은 일시적인 오류 처리 하는 데 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-202">The Retry pattern is useful for handling transient faults.</span></span> <span data-ttu-id="b92a5-203">오류가 오래 지속될 것 같은 경우 회로 차단기 패턴을 구현하는 것이 적절할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-203">If a failure is expected to be more long lasting, it might be more appropriate to implement the Circuit Breaker pattern.</span></span> <span data-ttu-id="b92a5-204">또한 다시 시도 패턴은 오류를 처리하기 위해 포괄적인 접근 방법을 제공하는 회로 차단기와 함께 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="b92a5-204">The Retry pattern can also be used in conjunction with a circuit breaker to provide a comprehensive approach to handling faults.</span></span>
- [<span data-ttu-id="b92a5-205">특정 서비스에 대한 다시 시도 지침</span><span class="sxs-lookup"><span data-stu-id="b92a5-205">Retry guidance for specific services</span></span>](https://docs.microsoft.com/en-us/azure/architecture/best-practices/retry-service-specific)
- [<span data-ttu-id="b92a5-206">연결 복원력</span><span class="sxs-lookup"><span data-stu-id="b92a5-206">Connection Resiliency</span></span>](https://docs.microsoft.com/en-us/ef/core/miscellaneous/connection-resiliency)
