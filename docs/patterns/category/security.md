---
title: Security patterns
description: Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information. Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users. Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.
keywords: design pattern
author: dragon119
ms.author: pnp
ms.date: 03/24/2017
ms.topic: article
ms.service: guidance

pnp.series.title: Cloud Design Patterns
---

# 보안 패턴

[!INCLUDE [header](../../_includes/header.md)]

보안은 설계된 용도를 벗어나는 악의적이거나 우발적인 동작을 방지하고 정보의 공개 또는 손실을 방지하는 시스템의 능력을 의미합니다. 클라우드 응용 프로그램은 신뢰하는 온-프레미스 경계를 벗어나는 인터넷에 노출되고, 대중에 공개되며, 신뢰할 수 없는 사용자가 이용할 수 있습니다. 따라서 악의적인 공격으로부터 응용 프로그램을 보호하고, 승인된 사용자만으로 액세스를 제한하며, 민감한 데이터를 보호하는 방식으로 응용 프로그램을 설계하고 배포해야 합니다.

| 패턴 | 요약 |
| ------- | ------- |
| [페더레이션 ID](../federated-identity.md) | 외부 ID 공급자에게 인증을 위임합니다. |
| [게이트키퍼](../gatekeeper.md) | 클라이언트와 응용 프로그램 또는 서비스 사이에 브로커로 작용하는 전용 호스트 인스턴스를 사용해 응용 프로그램과 서비스를 보호하고, 요청을 확인하고 삭제하며, 요청 및 요청 사이의 데이터를 통과시킵니다. |
| [발렛 키](../valet-key.md) | 특정 리소스 또는 서비스에 대한 제한적인 직접 액세스를 클라이언트에게 제공하는 토큰 또는 키를 사용합니다. |
