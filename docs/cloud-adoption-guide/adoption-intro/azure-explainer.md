---
title: '설명: Azure의 작동 방식'
description: Azure의 내부 작동 설명
author: petertay
ms.openlocfilehash: 1cebcc001b8d2ae93d8b0271c48d54617281c7c2
ms.sourcegitcommit: b3d74d8a89b2224fc796ce0e89cea447af43a0d4
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/11/2018
ms.locfileid: "35290512"
---
# <a name="explainer-how-does-azure-work"></a>설명: Azure의 작동 방식

Azure는 Microsoft의 공용 클라우드 플랫폼입니다. Azure는 PaaS(Platform as a Service), IaaS(Infrastructure as a Service), DBaaS(Database as a Service) 등을 비롯한 다양한 서비스 컬렉션을 제공합니다. 하지만 Azure란 정확히 무엇이며 어떤 방식으로 작동할까요?

다른 클라우드 플랫폼과 마찬가지로, Azure는 **가상화**라는 기술에 의존합니다. 대부분의 컴퓨터 하드웨어는 간단히 말해서 실리콘에 영구적으로 또는 반영구적으로 인코딩되는 명령 집합이므로, 소프트웨어에 에뮬레이트할 수 있습니다. 소프트웨어 명령을 하드웨어의 명령에 매핑하는 에뮬레이션 계층을 사용하여, 가상화된 하드웨어는 마치 실제 하드웨어인 것처럼 소프트웨어에서 실행될 수 있습니다.

기본적으로, 클라우드는 고객을 대신해서 가상화된 하드웨어를 실행하는 하나 이상의 데이터 센터의 물리적 서버 집합입니다. 그렇다면 클라우드는 어떻게 수백만 명의 고객을 위해 수백만 개의 가상화된 하드웨어 인스턴스를 동시에 만들고 시작하고 중지하고 삭제할 수 있을까요?

이를 이해하기 위해 데이터 센터의 하드웨어 아키텍처를 살펴보겠습니다.  각 데이터 센터 내의 서버 랙에는 서버 컬렉션이 있습니다. 각 서버 랙에는 네트워크 연결 및 전력을 공급하는 PDU(전원 분배 장치)를 제공하는 네트워크 스위치와 함께 많은 서버 **블레이드**가 포함되어 있습니다. 경우에 따라 랙은 **클러스터**라는 좀 더 큰 단위로 함께 그룹화됩니다. 

각 랙 또는 클러스터 내에서, 대부분의 서버는 사용자 대신 이러한 가상화된 하드웨어 인스턴스를 실행하도록 지정됩니다. 그러나 많은 서버는 패브릭 컨트롤러라고 하는 클라우드 관리 소프트웨어를 실행합니다. **패브릭 컨트롤러**는 많은 책임을 담당하는 분산된 응용 프로그램입니다. 서비스를 할당하고, 서버와 해당 서버에서 실행되는 서비스의 상태를 모니터링하고, 실패할 경우 서버를 복구합니다.

패브릭 컨트롤러의 각 인스턴스는 클라우드 오케스트레이션 소프트웨어를 실행하는 **프런트 엔드**라고도 하는 또 다른 서버 집합에 연결됩니다. 프런트 엔드는 웹 서비스, RESTful API 및 클라우드가 수행하는 모든 기능에 사용되는 내부 Azure Database를 호스트합니다. 

예를 들어, 프런트 엔드는 [가상 네트워크][vnet], [가상 머신][vms]과 같은 Azure 리소스와 [Cosmos DB][cosmosdb]와 같은 서비스에 대한 고객의 할당 요청을 처리하는 서비스를 호스트합니다. 첫째, 프런트 엔드는 사용자의 유효성을 검사하고 사용자가 요청된 리소스를 할당할 수 있도록 허가되는지 확인합니다. 사용자에게 권한이 있는 경우, 프런트 엔드는 데이터베이스를 확인하여 충분한 용량이 있는 서버 랙을 찾은 다음, 랙의 패브릭 컨트롤러에 리소스를 할당하도록 지시합니다.

따라서 아주 간단하지만, Azure는 서버 및 네트워킹 하드웨어의 방대한 컬렉션이며, 해당 서버에서 가상화된 하드웨어 및 소프트웨어의 구성과 작동을 오케스트레이션하는 복잡한 분산 응용 프로그램을 포함합니다. 또한 이러한 오케스트레이션을 통해 Azure가 강력해질 수 있습니다. 즉, Azure가 백그라운드에서 하드웨어를 유지 관리하고 업그레이드하므로 사용자가 이러한 작업을 수행할 필요가 없습니다. 

## <a name="next-steps"></a>다음 단계

* Azure의 내부 기능에 대해 이해했으므로 [리소스 액세스 거버넌스](governance-explainer.md)에 대해 알아봅니다. 그런 다음, Azure를 채택하는 첫 번째 단계로 이동하여 [Azure에서 디지털 ID를 이해](tenant-explainer.md)합니다. 해당 단계를 완료되면 [Azure AD에서 첫 번째 사용자 만들기][docs-add-users-to-aad]가 준비된 것입니다.

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview
