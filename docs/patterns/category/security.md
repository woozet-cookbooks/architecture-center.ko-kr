---
title: "보안 패턴"
description: "보안은 설계된 용도를 벗어나는 악의적 작업 또는 실수로 인한 작업을 방지하고, 정보의 공개 또는 손실을 방지하는 시스템 기능입니다. 클라우드 응용 프로그램은 신뢰할 수 있는 온-프레미스 경계 외부에서 인터넷에 노출되고, 종종 일반에 공개되고, 신뢰할 수 없는 사용자가 사용할 수도 있습니다. 응용 프로그램은 악의적인 공격으로부터 보호하고 승인된 사용자만 액세스하도록 제한하고 중요한 데이터를 보호하는 방식으로 설계 및 배포해야 합니다."
keywords: "디자인 패턴"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 266b5c4283d82a107783fc7a746f065be9027b51
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="security-patterns"></a><span data-ttu-id="7125f-106">보안 패턴</span><span class="sxs-lookup"><span data-stu-id="7125f-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="7125f-107">보안은 설계된 용도를 벗어나는 악의적 작업 또는 실수로 인한 작업을 방지하고, 정보의 공개 또는 손실을 방지하는 시스템 기능입니다.</span><span class="sxs-lookup"><span data-stu-id="7125f-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="7125f-108">클라우드 응용 프로그램은 신뢰할 수 있는 온-프레미스 경계 외부에서 인터넷에 노출되고, 종종 일반에 공개되고, 신뢰할 수 없는 사용자가 사용할 수도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="7125f-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="7125f-109">응용 프로그램은 악의적인 공격으로부터 보호하고 승인된 사용자만 액세스하도록 제한하고 중요한 데이터를 보호하는 방식으로 설계 및 배포해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="7125f-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>

| <span data-ttu-id="7125f-110">패턴</span><span class="sxs-lookup"><span data-stu-id="7125f-110">Pattern</span></span> | <span data-ttu-id="7125f-111">요약</span><span class="sxs-lookup"><span data-stu-id="7125f-111">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="7125f-112">페더레이션 ID</span><span class="sxs-lookup"><span data-stu-id="7125f-112">Federated Identity</span></span>](../federated-identity.md) | <span data-ttu-id="7125f-113">외부 ID 공급자에게 인증을 위임합니다.</span><span class="sxs-lookup"><span data-stu-id="7125f-113">Delegate authentication to an external identity provider.</span></span> |
| [<span data-ttu-id="7125f-114">게이트 키퍼</span><span class="sxs-lookup"><span data-stu-id="7125f-114">Gatekeeper</span></span>](../gatekeeper.md) | <span data-ttu-id="7125f-115">클라이언트와 응용 프로그램 또는 서비스 간 브로커 역할을 하며, 요청을 검사 및 정리하고, 요청 및 데이터를 전달하는 전용 호스트 인스턴스를 사용하여 응용 프로그램 및 서비스를 보호합니다.</span><span class="sxs-lookup"><span data-stu-id="7125f-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
| [<span data-ttu-id="7125f-116">발레 키</span><span class="sxs-lookup"><span data-stu-id="7125f-116">Valet Key</span></span>](../valet-key.md) | <span data-ttu-id="7125f-117">클라이언트에 특정 리소스 또는 서비스에 대한 제한된 직접 액세스를 제공하는 토큰 또는 키를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="7125f-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span> |