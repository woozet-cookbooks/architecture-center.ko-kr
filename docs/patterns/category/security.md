---
title: 보안 패턴
description: 보안은 설계된 용도를 벗어나는 악의적 작업 또는 실수로 인한 작업을 방지하고, 정보의 공개 또는 손실을 방지하는 시스템 기능입니다. 클라우드 응용 프로그램은 신뢰할 수 있는 온-프레미스 경계 외부에서 인터넷에 노출되고, 종종 일반에 공개되고, 신뢰할 수 없는 사용자가 사용할 수도 있습니다. 응용 프로그램은 악의적인 공격으로부터 보호하고 승인된 사용자만 액세스하도록 제한하고 중요한 데이터를 보호하는 방식으로 설계 및 배포해야 합니다.
keywords: 디자인 패턴
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 8437b8dfef751226580437a1b5678ca0e0e71f18
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847001"
---
# <a name="security-patterns"></a>보안 패턴

[!INCLUDE [header](../../_includes/header.md)]

보안은 설계된 용도를 벗어나는 악의적 작업 또는 실수로 인한 작업을 방지하고, 정보의 공개 또는 손실을 방지하는 시스템 기능입니다. 클라우드 응용 프로그램은 신뢰할 수 있는 온-프레미스 경계 외부에서 인터넷에 노출되고, 종종 일반에 공개되고, 신뢰할 수 없는 사용자가 사용할 수도 있습니다. 응용 프로그램은 악의적인 공격으로부터 보호하고 승인된 사용자만 액세스하도록 제한하고 중요한 데이터를 보호하는 방식으로 설계 및 배포해야 합니다.


|                    패턴                     |                                                                                                         요약                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [페더레이션 ID](../federated-identity.md) |                                                                                외부 ID 공급자에게 인증을 위임합니다.                                                                                |
|         [게이트 키퍼](../gatekeeper.md)         | 클라이언트와 응용 프로그램 또는 서비스 간 브로커 역할을 하며, 요청을 검사 및 정리하고, 요청 및 데이터를 전달하는 전용 호스트 인스턴스를 사용하여 응용 프로그램 및 서비스를 보호합니다. |
|          [발레 키](../valet-key.md)          |                                                        클라이언트에 특정 리소스 또는 서비스에 대한 제한된 직접 액세스를 제공하는 토큰 또는 키를 사용합니다.                                                        |

