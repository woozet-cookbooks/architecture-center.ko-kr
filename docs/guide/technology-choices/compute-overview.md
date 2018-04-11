---
title: Azure 계산 옵션 개요
description: Azure 계산 옵션 개요
author: MikeWasson
ms.openlocfilehash: a23dd49f24bc52db6f357540e3ebccb19e0497ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="overview-of-azure-compute-options"></a>Azure 계산 옵션 개요

*계산*이라는 용어는 응용 프로그램이 실행되는 계산 리소스의 호스팅 모델을 말합니다. 

스펙트럼의 한쪽 끝은 IaaS(**Intrastructure-as a Service**)입니다. IaaS를 사용하여 연결된 네트워크 및 저장소 구성 요소와 함께 필요한 VM을 프로비전합니다. 그런 다음 해당 VM에 배치하려는 모든 소프트웨어 및 응용 프로그램을 배포합니다. 이 모델은 Microsoft에서 인프라를 관리한다는 점을 제외하고는 기존 온-프레미스 환경과 가장 가깝습니다. 여전히 개별 VM을 관리하게 됩니다.  

PaaS(**Platform-as-a-Service**)는 VM 또는 네트워킹 리소스를 관리할 필요 없이 응용 프로그램을 배포할 수 있는 관리되는 호스팅 환경을 제공합니다. 예를 들어, 개별 VM을 만드는 대신, 인스턴스 수를 지정하기만 하면 서비스가 필요한 리소스를 프로비전, 구성 및 관리합니다. Azure App Service는 PaaS 서비스의 예제입니다.

IaaS에서 순수한 PaaS로의 스펙트럼이 구현됩니다. 예를 들어 Azure VM은 VM Scale Sets를 사용하여 자동으로 크기가 조정될 수 있습니다. 이러한 자동 크기 조정 기능은 엄밀히 PaaS는 아니며, PaaS 서비스에서 확인될 수 있는 관리 기의 유형입니다.

FaaS(**Functions-as-a-Service**)는 호스팅 환경을 걱정할 필요가 없는 좀 더 발전한 형태입니다. 계산 인스턴스를 만들고 해당 인스턴스에 코드를 배포하는 대신, 사용자가 코드를 배포하기만 하면 서비스가 코드를 자동으로 실행합니다. 계산 리소스를 관리할 필요가 없습니다. 이러한 서비스는 서버 없는 아키텍처를 사용하며, 트래픽 처리에 필요한 수준까지 원활하게 규모가 확장 및 축소됩니다. Azure Functions는 FaaS 서비스입니다.

IaaS가 가장 높은 제어 능력, 유연성 및 이동성을 제공합니다. FaaS는 단순성, 탄력적인 크기 조정 기능을 제공하며, 코드를 실행할 때만 비용이 청구되므로 비용 절감 효과도 제공합니다. PaaS는 두 서비스의 중간에 해당합니다. 일반적으로 서비스가 더 많은 유연성을 제공할수록, 리소스를 구성하고 관리하는 사용자의 책임은 커집니다. IaaS 솔루션을 사용하면 사용자가 만든 VM 및 네트워크 구성 요소를 프로비전, 구성 및 관리해야 하지만, FaaS 서비스는 응용 프로그램 실행의 거의 모든 측면을 자동으로 관리합니다.

다음은 Azure에서 현재 사용 가능한 주 계산 옵션입니다.

- [Virtual Machines](/azure/virtual-machines/)는 IaaS 서비스로, VNet(가상 네트워크) 내에서 VM을 배포 및 관리할 수 있도록 합니다.
- [App Service](/azure/app-service/app-service-value-prop-what-is)는 웹앱, 모바일 앱 백 엔드, RESTful API 또는 자동화된 비즈니스 프로세스를 호스트하기 위한 관리되는 서비스입니다.
- [Service Fabric](/azure/service-fabric/service-fabric-overview)은 Azure 또는 온-프레미스를 포함하는 다양한 환경에서 실행될 수 있는 분산 시스템 플랫폼입니다. Service Fabric은 컴퓨터 클러스터 간의 마이크로 서비스 오케스트레이터입니다. 
- [Azure Container Service](/azure/container-service/container-service-intro)는 컨테이너화된 응용 프로그램을 실행하는 미리 구성된 VM의 클러스터를 만들고 구성하고 관리할 수 있도록 합니다.
- [Azure Functions](/azure/azure-functions/functions-overview)는 관리되는 FaaS 서비스입니다.
- [Azure Batch](/azure/batch/batch-technical-overview)는 대규모 병렬 HPC(고성능 컴퓨팅) 응용 프로그램을 실행하기 위한 관리 서비스입니다.
- [Cloud Services](/azure/cloud-services/cloud-services-choose-me)는 클라우드 응용 프로그램을 실행하기 위한 관리되는 서비스입니다. PaaS 호스팅 모델을 사용합니다. 

계산 옵션을 선택할 때 고려해야 할 몇 가지 요인은 다음과 같습니다.

- 호스팅 모델. 서비스 호스트 방법 이 호스팅 환경으로 인해 발생하는 요구 사항 및 제한 사항 
- DevOps 응용 프로그램 업그레이드에 대해 기본 제공되는 지원이 있나요? 배포 모델이란?
- 확장성 서비스는 인스턴스의 추가 또는 제거를 어떻게 처리하나요? 부하 및 기타 메트릭을 기준으로 크기를 자동으로 조정할 수 있나요? 
- 가용성 서비스 SLA란? 
- 비용 서비스 자체의 비용 외에도, 해당 서비스에 구축된 솔루션을 관리하기 위한 운영 비용을 고려합니다. 예를 들어, IaaS 솔루션의 운영 비용은 더 높을 수 있습니다.
- 각 서비스의 전반적인 제한 사항은 무엇인가요? 
- 이 서비스에 어떤 유형의 응용 프로그램 아키텍처가 적절한가요? 

Azure의 계산 옵션에 대한 보다 자세한 비교 데이터를 보려면 [Azure 계산 옵션 선택 기준](./compute-comparison.md)을 참조하세요.