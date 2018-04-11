---
title: Azure 계산 옵션을 선택하기 위한 조건
description: 여러 축에서 Azure 계산 서비스를 비교합니다.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 36b57d1fb674b5a1452a0e8208de836963b2b01b
ms.sourcegitcommit: c53adf50d3a787956fc4ebc951b163a10eeb5d20
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/23/2017
---
# <a name="criteria-for-choosing-an-azure-compute-option"></a>Azure 계산 옵션을 선택하기 위한 조건

*계산*이라는 용어는 응용 프로그램이 실행되는 계산 리소스의 호스팅 모델을 말합니다. 아래 표에서는 다양한 축에 걸쳐 Azure 계산 서비스를 비교합니다. 응용 프로그램에 사용할 계산 옵션을 선택할 때 아래 표를 참조하세요.

## <a name="hosting-model"></a>호스팅 모델

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 응용 프로그램 구성 | 장치 및 시스템 독립성 | 응용 프로그램 | 서비스, 게스트 실행 파일, 컨테이너 | Functions | 컨테이너 | 역할 | Scheduled jobs  |
| 밀도 | 장치 및 시스템 독립성 | 앱 요금제를 통해 인스턴스당 복수의 앱 지원 | VM당 복수의 서비스 지원 | 전용 인스턴스 없음 <a href="#note1"><sup>1</sup></a> | VM당 복수의 컨테이너 지원 | VM당 하나의 역할 인스턴스 지원 | VM당 복수의 앱 지원 |
| 최소 노드 개수 | 1 <a href="#note2"><sup>2</sup></a>  | 1 | 5 <a href="#note3"><sup>3</sup></a> | 전용 노드 없음 <a href="#note1"><sup>1</sup></a> | 3 | 2 | 1 <a href="#note4"><sup>4</sup></a> |
| 상태 관리 | 상태 비저장/상태 저장 | 상태 비저장 | 상태 비저장/상태 저장 | 상태 비저장 | 상태 비저장/상태 저장 | 상태 비저장 | 상태 비저장 |
| 웹 호스팅 | 장치 및 시스템 독립성 | 기본 제공 | 장치 및 시스템 독립성 | 해당 없음 | 장치 및 시스템 독립성 | 기본 제공(IIS) | 아니오 |
| OS | Windows, Linux | Windows, Linux  | Windows, Linux | 해당 없음 | Windows(미리 보기), Linux | Windows | Windows, Linux |
| 전용 VNet에 배포 가능 여부 | 지원됨 | 지원됨 <a href="#note5"><sup>5</sup></a> | 지원됨 | 지원되지 않음 | 지원됨 | 지원됨 <a href="#note6"><sup>6</sup></a> | 지원됨 |
| 하이브리드 연결 | 지원됨 | 지원됨 <a href="#note1"><sup>7</sup></a>  | 지원됨 | 지원되지 않음 | 지원됨 | 지원됨 <a href="#note8"><sup>8</sup></a> | 지원됨 |

메모

1. <span id="note1">사용 요금제를 사용하는 경우. App Service 요금제를 사용하는 경우, App Service 요금제에 할당된 VM에서 기능 실행. [Azure Functions를 위한 올바른 서비스 요금제 선택][function-plans]을 참조하세요.</a>
2. <span id="note2">둘 이상의 인스턴스가 존재하는 경우 보다 높은 SLA 지원.</a>
3. <span id="note3">프로덕션 환경용.</a>
4. <span id="note4">작업 완료 이후 0으로 축소 가능.</a>
5. <span id="note5">ASE(App Service Environment) 필요.</a>
6. <span id="note6">클래식 VNet 전용.</a>
7. <span id="note7">ASE 또는 BizTalk 하이브리드 연결 필요</a>
8. <span id="note8">클래식 VNet 또는 VNet 피어링을 통한 리소스 관리자 VNet</a>

## <a name="devops"></a>DevOps

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 로컬 디버깅 | 장치 및 시스템 독립성 | IIS Express, 기타 <a href="#note1b"><sup>1</sup></a> | 로컬 노드 클러스터 | Azure Functions CLI | 로컬 컨테이너 런타임 | 로컬 에뮬레이터 | 지원되지 않음 |
| 프로그래밍 모델 | 장치 및 시스템 독립성 | 웹 응용 프로그램, 백그라운드 작업을 위한 WebJob | 게스트 실행 파일, 서비스 모델, 작업자 모델, 컨테이너 | 트리거가 있는 함수 | 장치 및 시스템 독립성 | 웹 역할, 작업자 역할 | 명령줄 응용 프로그램 |
| 리소스 관리자 | 지원됨 | 지원됨 | 지원됨 | 지원됨 | 지원됨 | 제한적 <a href="#note2b"><sup>2</sup></a> | 지원됨 |  
| 응용 프로그램 업데이트 | 기본 제공 지원 없음 | 배포 슬롯 | 롤링 업그레이드(서비스당) | 기본 제공 지원 없음 | 오케스트레이터에 따라 달라짐. 대부분 롤링 업데이트를 지원함 | VIP 스왑 또는 롤링 업데이트 | 해당 없음 |

메모

1. <span id="note1b">옵션에는 ASP.NET 또는 node.js(iisnode)용 IIS Express, PHP 웹 서버, IntelliJ용 Azure Toolkit, Eclipse용 Azure Toolkit이 포함됩니다. App Service는 배포된 웹앱의 원격 디버깅도 지원합니다.</a>
2. <span id="note2b">[리소스 관리자 공급자, 지역, API 버전 및 스키마][resource-manager-supported-services]를 참조하세요. 


## <a name="scalability"></a>확장성

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 자동 확장 | VM 확장 집합 | 기본 제공 서비스 | VM 확장 집합 | 기본 제공 서비스 | 지원되지 않음 | 기본 제공 서비스 | 해당 없음 |
| 부하 분산 장치 | Azure Load Balancer | 통합형 | Azure Load Balancer | 통합형 | Azure Load Balancer | 통합형 | Azure Load Balancer |
| 확장 제한 | 플랫폼 이미지: VMSS당 노드 1000개, 사용자 지정 이미지: VMSS당 노드 100개 | 인스턴스 20개, App Service Environment를 갖는 인스턴스 50개 | VMSS당 노드 100개 | 무제한 <a href="#note1c"><sup>1</sup></a> | 100 | 정의된 제한 없음, 최대 200개 권장 | 기본적으로 코어 20개로 제한. 제한 증가가 필요한 경우 고객 서비스에 문의하세요. |

메모

1. <span id="note1c">사용 요금제를 사용하는 경우. App Service 요금제를 사용하는 경우에는 App Service 확장 제한이 적용됩니다. [Azure Functions를 위한 올바른 서비스 요금제 선택][function-plans]을 참조하세요.</a>

## <a name="availability"></a>Availability

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SLA | [Virtual Machines용 SLA][sla-vm] | [App Service용 SLA][sla-app-service] | [Service Fabric용 SLA][sla-sf] | [Functions용 SLA][sla-functions] | [Azure Container Service용 SLA][sla-acs] | [Cloud Services용 SLA][sla-cloud-service] | [Azure Batch용 SLA][sla-batch] |
| 다중 지역 장애 조치(failover) | 트래픽 관리자 | 트래픽 관리자 | 트래픽 관리자, 다중 지역 클러스터 | 지원되지 않음  | 트래픽 관리자 | 트래픽 관리자 | 지원되지 않음 |

## <a name="security"></a>보안

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| SSL | VM에 구성됨 | 지원됨 | 지원됨  | 지원됨 | VM에 구성됨 | 지원됨 | 지원됨 |
| RBAC | 지원됨 | 지원됨 | 지원됨 | 지원됨 | 지원됨 | 지원되지 않음 | 지원됨 |

## <a name="other"></a>기타

| 조건 | Virtual Machines | App Service | Service Fabric | Azure 기능 | Azure Container Service | Cloud Services | Azure Batch |
|----------|-----------------|-------------|----------------|-----------------|-------------------------|----------------|-------------|
| 비용 | [Windows][cost-windows-vm], [Linux][cost-linux-vm] | [App Service 가격][cost-app-service] | [Service Fabric 가격][cost-service-fabric] | [Azure Functions 가격][cost-functions] | [Azure Container Service 가격][cost-acs] | [Cloud Services 가격][cost-cloud-services] | [Azure Batch 가격][cost-batch]
| 적합한 아키텍처 스타일 | N 계층, 다량 계산(HPC) | 웹-큐-작업자 | 마이크로 서비스, 이벤트 기반 아키텍처(EDA) | 마이크로 서비스, EDA | 마이크로 서비스, EDA | 웹-큐-작업자 | 다량 계산 |

[cost-linux-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/linux/
[cost-windows-vm]: https://azure.microsoft.com/pricing/details/virtual-machines/windows/
[cost-app-service]: https://azure.microsoft.com/pricing/details/app-service/
[cost-service-fabric]: https://azure.microsoft.com/pricing/details/service-fabric/
[cost-functions]: https://azure.microsoft.com/pricing/details/functions/
[cost-acs]: https://azure.microsoft.com/pricing/details/container-service/
[cost-cloud-services]: https://azure.microsoft.com/pricing/details/cloud-services/
[cost-batch]: https://azure.microsoft.com/pricing/details/batch/

[function-plans]: /azure/azure-functions/functions-scale
[sla-acs]: https://azure.microsoft.com/support/legal/sla/container-service/
[sla-app-service]: https://azure.microsoft.com/support/legal/sla/app-service/
[sla-batch]: https://azure.microsoft.com/support/legal/sla/batch/
[sla-cloud-service]: https://azure.microsoft.com/support/legal/sla/cloud-services/
[sla-functions]: https://azure.microsoft.com/support/legal/sla/functions/
[sla-sf]: https://azure.microsoft.com/support/legal/sla/service-fabric/
[sla-vm]: https://azure.microsoft.com/support/legal/sla/virtual-machines/

[resource-manager-supported-services]: /azure/azure-resource-manager/resource-manager-supported-services