---
title: Azure 계산 서비스를 선택하기 위한 조건
description: 여러 축에서 Azure 계산 서비스 비교
author: MikeWasson
layout: LandingPage
ms.date: 06/13/2018
ms.openlocfilehash: 29c21c44bdf3a3bfa29f17015565eecf5f86163b
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206656"
---
# <a name="criteria-for-choosing-an-azure-compute-service"></a>Azure 계산 서비스를 선택하기 위한 조건

*계산*이라는 용어는 응용 프로그램이 실행되는 계산 리소스의 호스팅 모델을 말합니다. 아래 표에서는 다양한 축에 걸쳐 Azure 계산 서비스를 비교합니다. 응용 프로그램에 사용할 계산 옵션을 선택할 때 아래 표를 참조하세요.

## <a name="hosting-model"></a>호스팅 모델

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 응용 프로그램 구성 | 장치 및 시스템 독립성 | 응용 프로그램, 컨테이너 | 서비스, 게스트 실행 파일, 컨테이너 | Functions | 컨테이너 | 컨테이너 | Scheduled jobs  |
| 밀도 | 장치 및 시스템 독립성 | 앱 요금제를 통해 인스턴스당 복수의 앱 지원 | VM당 복수의 서비스 지원 | 전용 인스턴스 없음 <a href="#note1"><sup>1</sup></a> | VM당 복수의 컨테이너 지원 |전용 인스턴스 없음 | VM당 복수의 앱 지원 |
| 최소 노드 개수 | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | 전용 노드 없음 <a href="#note1"><sup>1</sup></a> | 3 | 전용 노드 없음 | 1 <a href="#note4"><sup>4</sup></a> |
| 상태 관리 | 상태 비저장/상태 저장 | 상태 비저장 | 상태 비저장/상태 저장 | 상태 비저장 | 상태 비저장/상태 저장 | 상태 비저장 | 상태 비저장 |
| 웹 호스팅 | 장치 및 시스템 독립성 | 기본 제공 | 장치 및 시스템 독립성 | 해당 없음 | 장치 및 시스템 독립성 | 장치 및 시스템 독립성 | 아니오 |
| OS | Windows, Linux | Windows, Linux  | Windows, Linux | 해당 없음 | Windows(미리 보기), Linux | Windows, Linux | Windows, Linux |
| 전용 VNet에 배포 가능 여부 | 지원됨 | 지원됨 <a href="#note5"><sup>5</sup></a> | 지원됨 | 지원되지 않음 | 지원됨 | 지원되지 않음 | 지원됨 |
| 하이브리드 연결 | 지원됨 | 지원됨 <a href="#note1"><sup>6</sup></a>  | 지원됨 | 지원되지 않음 | 지원됨 | 지원되지 않음 | 지원됨 |

메모

1. <span id="note1">사용 요금제를 사용하는 경우. App Service 요금제를 사용하는 경우, App Service 요금제에 할당된 VM에서 기능 실행. [Azure Functions를 위한 올바른 서비스 요금제 선택][function-plans]을 참조하세요.</span>
2. <span id="note2">둘 이상의 인스턴스가 존재하는 경우 보다 높은 SLA 지원.</span>
3. <span id="note3">프로덕션 환경용.</span>
4. <span id="note4">작업 완료 이후 0으로 축소 가능.</span>
5. <span id="note5">ASE(App Service Environment) 필요.</span>
6. <span id="note7">ASE 또는 BizTalk 하이브리드 연결 필요</span>

## <a name="devops"></a>DevOps

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 로컬 디버깅 | 장치 및 시스템 독립성 | IIS Express, 기타 <a href="#note1b"><sup>1</sup></a> | 로컬 노드 클러스터 | Azure Functions CLI | 로컬 컨테이너 런타임 | 로컬 컨테이너 런타임 | 지원되지 않음 |
| 프로그래밍 모델 | 장치 및 시스템 독립성 | 웹 응용 프로그램, 백그라운드 작업을 위한 WebJob | 게스트 실행 파일, 서비스 모델, 작업자 모델, 컨테이너 | 트리거가 있는 함수 | 장치 및 시스템 독립성 | 장치 및 시스템 독립성 | 명령줄 응용 프로그램 |
| 응용 프로그램 업데이트 | 기본 제공 지원 없음 | 배포 슬롯 | 롤링 업그레이드(서비스당) | 기본 제공 지원 없음 | 오케스트레이터에 따라 달라짐. 대부분 롤링 업데이트를 지원함 | 컨테이너 이미지 업데이트 | 해당 없음 |

메모

1. <span id="note1b">옵션에는 ASP.NET 또는 node.js(iisnode)용 IIS Express, PHP 웹 서버, IntelliJ용 Azure Toolkit, Eclipse용 Azure Toolkit이 포함됩니다. App Service는 배포된 웹앱의 원격 디버깅도 지원합니다.</span>
2. <span id="note2b">[Resource Manager 공급자, 지역, API 버전 및 스키마][resource-manager-supported-services]를 참조하세요.</span> 


## <a name="scalability"></a>확장성

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 자동 확장 | VM 확장 집합 | 기본 제공 서비스 | VM 확장 집합 | 기본 제공 서비스 | 지원되지 않음 | 지원되지 않음 | 해당 없음 |
| 부하 분산 장치 | Azure Load Balancer | 통합형 | Azure Load Balancer | 통합형 | Azure Load Balancer |  기본 제공 지원 없음 | Azure Load Balancer |
| 확장 제한 | 플랫폼 이미지: VMSS당 노드 1000개, 사용자 지정 이미지: VMSS당 노드 100개 | 인스턴스 20개, App Service Environment를 갖는 인스턴스 50개 | VMSS당 노드 100개 | 무제한 <a href="#note1c"><sup>1</sup></a> | 100 <a href="#note2c"><sup>2</sup></a> |기본적으로 구독당 20개의 컨테이너 그룹입니다. 제한 증가가 필요한 경우 고객 서비스에 문의하세요. <a href="#note3c"><sup>3</sup></a> | 기본적으로 코어 20개로 제한. 제한 증가가 필요한 경우 고객 서비스에 문의하세요. |

메모

1. <span id="note1c">사용 요금제를 사용하는 경우. App Service 요금제를 사용하는 경우에는 App Service 확장 제한이 적용됩니다. [Azure Functions를 위한 올바른 서비스 요금제 선택][function-plans]을 참조하세요.</span>
2. <span id="note2c">[Container Service 클러스터의 에이전트 노드 확장][scale-acs]</span>을 참조하세요.
3. <span id="note3c">[Azure Container Instances에 대한 할당량 및 지역 가용성](/azure/container-instances/container-instances-quotas)을 참조하세요.</span>


## <a name="availability"></a>가용성

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [Virtual Machines용 SLA][sla-vm] | [App Service용 SLA][sla-app-service] | [Service Fabric용 SLA][sla-sf] | [Functions용 SLA][sla-functions] | [Azure Container Service용 SLA][sla-acs] | [Container instances용 SLA](https://azure.microsoft.com/support/legal/sla/container-instances/) | [Azure Batch용 SLA][sla-batch] |
| 다중 지역 장애 조치(failover) | 트래픽 관리자 | 트래픽 관리자 | 트래픽 관리자, 다중 지역 클러스터 | 지원되지 않음  | 트래픽 관리자 | 지원되지 않음 | 지원되지 않음 |

## <a name="other"></a>기타

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Container Instances | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | VM에 구성됨 | 지원됨 | 지원됨  | 지원됨 | VM에 구성됨 | 사이드카 컨테이너로 지원됨 | 지원됨 |
| 비용 | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [App Service 가격][cost-app-service] | [Service Fabric 가격][cost-service-fabric] | [Azure Functions 가격][cost-functions] | [Azure Container Service 가격][cost-acs] | [Container Instances 가격 책정](https://azure.microsoft.com/pricing/details/container-instances/) | [Azure Batch 가격][cost-batch]
| 적합한 아키텍처 스타일 | [N 계층][n-tier], [큰 계산][big-compute](HPC) | [Web-Queue-Worker][w-q-w] | [마이크로 서비스][microservices], [이벤트 기반 아키텍처][event-driven] | [마이크로 서비스][microservices], [이벤트 기반 아키텍처][event-driven] | [마이크로 서비스][microservices], [이벤트 기반 아키텍처][event-driven] | [마이크로 서비스][microservices], 작업 자동화, 일괄 처리 작업  | [큰 계산][big-compute](HPC) |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services
[scale-acs]: /azure/container-service/kubernetes/container-service-scale#scaling-considerations

[n-tier]: ../architecture-styles/n-tier.md
[w-q-w]: ../architecture-styles/web-queue-worker.md
[microservices]: ../architecture-styles/microservices.md
[event-driven]: ../architecture-styles/event-driven.md
[big-date]: ../architecture-styles/big-data.md
[big-compute]: ../architecture-styles/big-compute.md

