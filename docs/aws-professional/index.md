---
title: Introduction to Azure for AWS experts
description: Understand the basics of Microsoft Azure accounts, platform, and services. Also learn key similarities and differences between the AWS and Azure platforms.
keywords: AWS experts, Azure comparison, AWS comparison, difference between azure and aws, azure and aws
author: lbrader
ms.service: guidance
ms.topic: article
ms.date: 03/24/2017
ms.author: pnp

pnp.series.title: Azure for AWS Professionals
---

# AWS 전문가를 위한 Microsoft Azure 계정과 플랫폼 및 서비스 소개

이 문서는 Amazon Web Services (AWS) 전문가들이 Microsoft Azure 계정과 플랫폼 및 서비스를 이해하는데 도움이 됩니다. 또한 AWS와 Azure 플랫폼의 주요 유사점과 차이점을 다룹니다.

문서를 통해 다음 내용을 익힐 수 있습니다:

•	Azure에서 계정과 리소스가 구성되는 방법.

•	Azure에서 유용한 솔루션이 구조화된 방법.

•	Azure의 주요 서비스와 AWS 서비스의 다른 점.


Azure와 AWS가 점차 독립적으로 각자의 기능을 빌드함에 따라 중요한 구현 및 설계의 차이점을 갖게 되었습니다. 

## 개요

Microsoft Azure는 AWS와 마찬가지로 계산, 저장소, 데이터베이스, 네트워킹의 주요 서비스로 구성되었습니다. 대체로 두 플랫폼은 제품과 서비스에서 기본적으로 동일한 기능을 제공합니다. AWS와 Azure 모두 Windows와 Linux 호스트 기반에서 고가용성의 솔루션을 구성할 수 있습니다. 따라서, Linux와 OSS 기술을 사용하여 개발에 참여할 경우, 두 플랫폼 모두 성공할 수 있습니다.

두 플랫폼의 기능은 유사하나, 그 기능을 제공하는 리소스는 종종 다르게 구성됩니다. 솔루션 빌드에 필요한 서비스들 사이에 정확한 일대일 관계가 항상 명확한 것은 아닙니다. 또한 특정 서비스가 한 플랫폼에서는 제공되지만 다른 쪽에서는 제공되지 않을 수도 있습니다. [Azure와 AWS 서비스의 비교표](services.md)를 참조하세요.

## 계정 및 구독

Azure 서비스는 자기 조직의 규모와 요구에 따라 몇 가지 가격 옵션으로 구매할 수 있습니다. 자세한 내용은 [가격 개요](https://azure.microsoft.com/pricing/) 페이지를 참조하세요.

[Azure 구독](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/)
은 청구와 사용 권한을 관리하도록 지정된 소유자로 리소스를 그룹화한 개념입니다. AWS 계정에서 만들어진 리소스와 그 계정에 고정적인 AWS와 달리, 구독은 그 소유자의 계정과 독립해서 존재하고, 필요한 경우 새 소유자에 재지정될 수 있습니다.

![Comparison of structure and ownership of AWS accounts and Azure subscriptions](./images/azure-aws-account-compare.png "Comparison of structure and ownership of AWS accounts and Azure subscriptions")
<br/>*AWS 계정과 Azure 구독의 구조 및 소유권 비교*
<br/><br/>

구독에는 3가지 종류의 관리자 계정이 지정됩니다:

-   **계정 관리자** - 구독 소유자와 구독에 사용된 리소스에 대금 청구된 계정. 계정 관리자는 구독 소유권을 전환함으로써 변경할 수 있습니다.

-   **서비스 관리자** - 이 계정은 구독에서 리소스를 만들고 관리할 권한이 있으나, 청구 권한은 없습니다.  기본적으로, 계정 관리자와 서비스 관리자는 같은 계정에 지정됩니다. 계정 관리자는 구독의 기술적, 운영적 측면을 관리하는 서비스 관리자 계정에 별도의 사용자를 지정할 수 있습니다.  구독마다 서비스 관리자가 단 한 명 있습니다.

-   **공동관리자** - 구독에 복수의 공동관리자 계정들이 지정될 수 있습니다.  공동관리자들은 서비스 관리자를 바꿀 수 없지만, 이를 제외하고 구독 리소스와 사용자들을 전면적으로 통제합니다. 

AWS에서 IAM 사용자와 그룹에 사용 권한이 부여되는 것과 유사하게, 구독 등급 하에서도 사용자 역할과 개별적인 사용 권한은 특정 리소스에 지정될 수 있습니다.  Azure의 모든 사용자 계정은 Microsoft 계정이나 조직 계정(Azure Active Directory를 통해 관리되는 계정) 중 하나와 관련되어있습니다.

AWS 계정과 마찬가지로 구독에도 기본 서비스 할당량과 제한이 있습니다. 이 제한들의 전체 목록을 보려면 [Azure 구독 및 서비스 제한, 할당량, 제약 조건](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/)을 참조하세요. 이 제한들은 [관리 포털에서 지원 요청하기](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/)를 통해 최대로 증가 시킬 수 있습니다.

### 참고 항목

-   [Azure 관리자 역할 추가 또는 변경하기](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [Azure 청구서 및 일일 사용 데이터 다운로드하기](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## 리소스 관리

Azure에서 "리소스"는 AWS에서와 마찬가지로 계산 인스턴스, 저장소 개체, 네트워킹 장치 또는 플랫폼에서 만들거나 구성할 수 있는 기타 항목을 의미합니다.

Azure 리소스는 둘 중 한 모델을 사용하여 배포 또는 관리됩니다: Azure 리소스 관리자 또는 오래된 Azure [클래식 배포 모델](/azure/azure-resource-manager/resource-manager-deployment-model). 새 리소스는 리소스 관리자 모델을 사용하여 만듭니다.

### 리소스 그룹

Azure와 AWS는 모두 VM, 저장소, 가상 네트워킹 장치 등 리소스를 구성하는 "리소스 그룹"이라는 항목을 갖고 있습니다. 그러나, [Azure 리소스 그룹](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)은 AWS 리소스 그룹과 직접 비교할 수 없습니다.

AWS에서 리소스는 다중의 리소스 그룹으로 태그가 지정될 수 있지만, Azure 리소스는 항상 한 리소스 그룹과만 관련이 있습니다. 한 리소스 그룹에서 만들어진 리소스는 다른 그룹으로 옮겨갈 수 있지만 한 시점에 한 리소스 그룹에만 있을 수 있습니다. 리소스 그룹은 Azure 리소스 관리자에서 사용하는 기본적인 그룹입니다.

리소스는
[태그](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/)를 사용해서 구성할 수도 있습니다. 태그는 리소스 그룹 자격과 관계 없이 구독 중 리소스를 그룹으로 묶을 수 있는 키-값 쌍입니다.

### 관리 인터페이스

Azure는 리소스를 관리하는 방법들을 제공합니다.

-   [웹 인터페이스.](https://azure.microsoft.com/documentation/articles/resource-group-portal/).
    AWS 대시보드와 같이 Azure 포털은 Azure 리소스에 완전한 웹 기반 관리 인터페이스를 제공합니다.

-   [REST
    API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/).
    Azure 리소스 관리자 REST API는 Azure 포털에서 사용할 수 있는 대부분의 기능에 대한 프로그램 방식 액세스를 제공합니다.

-   [명령줄](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/).
    TAzure CLI 도구는 Azure 자원을 만들고 관리할 수 있는 명령줄 인터페이스를 제공합니다. Azure CLI는 [Windows, Linux, and
    Mac OS](https://github.com/azure/azure-xplat-cli)에서 사용할 수 있습니다.

-   [PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/).
    Azure에서 PowerShell 모듈을 사용하면 스크립트로 자동 관리 작업을 실행할 수 있습니다. PowerShell은 [Windows, Linux, and Mac
    OS](https://github.com/PowerShell/PowerShell)에서 사용할 수 있습니다.

-   [템플릿](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/).
   Azure 리소스 관리자 템플릿은 AWS CloudFormation 서비스와 비슷한 JSON 템플릿 기반의 리소스 관리 기능을 제공합니다.
   
이 인터페이스들에서 리소스 그룹은 Azure 리소스가 생성, 배포, 수정되는 작업의 중심이 됩니다. 이는 CloudFormation 배포 중 AWS 리소스를 그룹화할 때 "스택"이 하는 역할과 비슷합니다.

이 인터페이스들의 구문과 구조는 AWS와 다르지만, 비슷한 기능을 제공합니다. Azure에서는 [Hashicorp's
Terraform](https://www.terraform.io/docs/providers/azurerm/) 및 [Netflix
Spinnaker](http://www.spinnaker.io/)와 같이 AWS에서 사용된 타사의 여러 관리 도구도 사용할 수 있습니다.

### 참고 항목

-   [Azure 리소스 그룹 지침](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## 지역 및 영역 (고가용성)

AWS에서 가용성은 가용 영역이라는 개념에 중점을 둡니다. Azure에서 장애 도메인과 가용성 집합은 모두 고가용성 솔루션을 빌드하는 것과 관련이 있습니다. 쌍을 이루는 지역은 재해 복구 기능을 추가로 제공합니다.

### 가용 영역, Azure 장애 도메인, 가용성 집합

AWS에서 지역은 둘 이상의 가용 영역으로 분할됩니다. 가용 영역은 지역 내에 물리적으로 격리된 데이터센터에 해당합니다. 가용 영역들을 구분하기 위해서 응용 프로그램 서버를 배치한 경우, 한 영역에 영향을 주는 하드웨어 또는 연결 중단은 다른 영역에 호스팅된 서버에 영향을 미치지 않습니다.

Azure에서, [장애 도메인](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/)
은 물리적 전원과 네트워크 스위치를 공유하는 VM 그룹의 범위를 규정합니다. VM을 여러 장애 도메인에 VM을 분배하려면 [가용성 세트](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-manage-availability/)를 사용합니다. 인스턴스들이 같은 가용성 세트에 할당되면, Azure는 그 인스턴스들을 몇몇 장애 도메인에 골고루 배분합니다.  장애 도메인 한 곳에서 정전이나 네트워크 중단이 발생한 경우, 적어도 그 세트의 VM 일부가 다른 장애 도메인에 있어야 그러한 중단에 영향을 받지 않습니다.

![AWS Availability Zones comparison to Azure fault domains and availability sets](./images/zone-fault-domains.png "AWS Availability Zones compared with Azure fault domains and availability sets")
<br/>*AWS 가용 영역과 Azure 장애 도메인 및 가용성 세트 비교*
<br/><br/>

인스턴스가 그 역할에 맞게 운영되려면, 해당 인스턴스가 응용 프로그램에서 차지하는 역할에 따라 가용성 세트를 구성해야 합니다.  예를 들면, 표준 3층(three-tier) 웹 응용 프로그램에서 프런트 엔드, 응용 프로그램, 데이터 인스턴스에 대한 별도의 가용성 세트를 만들고자 할 수 있습니다.

![Azure availability sets for each application role](./images/three-tier-example.png "Availability sets for each application role")
<br/>*각 응용 프로그램 역할에 대한 Azure 가용성 세트*
<br/><br/>

VM 인스턴스가 가용성 세트가 추가되면,
[업데이트 도메인](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/).
도 할당됩니다. 업데이트 도메인은 계획된 유지관리 이벤트들에 동시에 설정된 VM 그룹입니다. VM을 여러 업데이트 도메인에 배분하면 계획된 업데이트와 패치 이벤트는 어느 때든 이 VM들의 하위 집합에만 영향을 미칩니다. 

### 쌍을 이루는 지역

Azure에서, 중단이 전체 Azure 지역에 영향을 미치더라도 솔루션을 계속 이용할 수 있고, 미리 정해진 두 지역 간에 중복을 지원하기 위해서는 [쌍을 이루는 지역](https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/)을 이용합니다.

데이터센터가 물리적으로 구분되었으나 지리적으로 비교적 가까운 지역에 있는 AWS 가용 영역과는 달리, 지역 쌍은 최소 300마일 떨어져 있습니다. 이는 대규모 재해가 일어났을 때 쌍을 이룬 지역 중 한 곳에만 영향을 주게 하기 위함입니다. 인접한 쌍은 데이터베이스와 저장소 서비스 데이터를 동기화하도록 설정할 수 있고, 또 플랫폼 업데이트가 한번에 지역 쌍 중 한 곳으로만 옮겨지도록 구성할 수 있습니다. 

Azure [지역 중복 저장소](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage)
는 적절한 지역 쌍으로 자동 백업됩니다. 다른 모든 리소스의 경우, 쌍을 이루는 지역을 사용한 완전한 중복 솔루션의 생성은 두 지역에서 전체 복제된 솔루션의 생성을 의미합니다.

### 참고 항목

-   [Azure에서 가상 컴퓨터의 지역 및 가용성](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [Micresoft Azure에 빌드된 응용 프로그램의 재해 복구 및 고가용성](https://azure.microsoft.com/documentation/articles/resiliency-disaster-recovery-high-availability-azure-applications/)

-   [Azure에서 Linux 가상 컴퓨터의 계획된 유지 관리](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## 서비스

모든 서비스가 플랫폼에서 어떻게 관련이 있는지에 대한 전체 목록을 보려면 [AWS와 Azure 서비스의 전체 비교표](https://aka.ms/azure4aws-services) 를 참조하세요.

모든 지역에서 Azure 제품 및 서비스를 모두 이용할 수 있는 것은 아닙니다. 자세한 내용은 [지역별 제품](https://azure.microsoft.com/regions/services/) 페이지를 참조하세요. Azure 제품 또는 서비스에 대한 가동 시간 보장률과 가동중지 시간 신용정책은 [서비스 수준 계약](https://azure.microsoft.com/support/legal/sla/) 페이지에서 참조할 수 있습니다.

다음 절에서는 공통으로 사용되는 기능과 서비스가 AWS와 Azure 플렛폼에서 어떻게 다른지 간략하게 설명합니다. 

### 계산 서비스

#### EC2 인스턴스와 Azure 가상 컴퓨터

AWS 인스턴스 종류와 Azure 가상 컴퓨터 크기가 비슷한 방식으로 분할된다고 하더라도, RAM, CPU, 저장소 기능에서는 차이가 납니다.

-   [Amazon EC2 인스턴스 유형](https://aws.amazon.com/ec2/instance-types/)

-   [Azure에서 가상 컴퓨터의 크기 (Windows)](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [Azure에서 가상 컴퓨터의 크기 (Linux)](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

AWS의 시간당 청구와 달리, Azure의 주문형 VM은 분 단위로 청구됩니다.

Azure는 EC2 스팟 인스턴스(Spot Instances), 예약 인스턴스(Reserved Instances), 전용 호스트(Dedicated Hosts)와 같지 않습니다.

#### EBS 및 VM 디스크에 대한 Azure 저장소

Azure VM에 대한 지속형 데이터 저장소는 블롭 저장소에 상주하는 [데이터 디스크](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/)
에 의해 제공됩니다. 이는 EC2 인스턴스가 Elastic Block Store(EBS)에 디스크 볼륨을 저장하는 방법과 비슷합니다. [Azure 임시 저장소](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)
또한 EC2 인스턴스 저장소와 동일한 짧은 대기시간의 임시 읽기-쓰기 저장소(사용 후 삭제 저장소라고도 함)를 VM에 제공합니다.

고성능 디스크 IO 는 [Azure 프리미엄 저장소](https://docs.microsoft.com/azure/storage/storage-premium-storage)를 통해 지원됩니다. 이는 AWS에서 제공되는 프로비저닝된 IOPS 저장소와 비슷합니다.

#### 람다, Azure 함수, Azure 웹 작업, Azure 논리 앱

[Azure 함수](https://azure.microsoft.com/services/functions/)는 서버를 사용하지 않는 주문형 코드를 제공하는 데 있어서 AWS 람다와 기본적으로 같습니다.  그러나, 람다 기능 또한 다른 Azure 서비스와 겹칩니다:

-   [웹작업](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) - 예약된 또는 계속 실행되는 백그라운드 작업을 만들 수 있습니다.

-   [논리 앱](https://azure.microsoft.com/services/logic-apps/) - 의사소통, 통합, 비즈니스 규칙 관리 서비스를 제공합니다.

#### 자동 크기 조정, Azure VM 크기 조정, Azure 앱 서비스 자동 크기 조정

Azure의 자동 크기 조정은 두 가지 서비스에 따라 처리됩니다:

-   [VM 크기 집합](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) - VM 중 동일한 세트를 배포하고 관리할 수 있습니다.  인스턴스 개수는 성능 요구를 기반으로 하여 자동 크기가 조정될 수 있습니다.

-   [App 서비스 자동 크기 조절](https://azure.microsoft.com/documentation/articles/web-sites-scale/) -Azure 앱 서비스 솔루션을 자동 크기 조정하는 기능을 제공합니다.


#### 컨테이너 서비스
[Azure 컨테이너 서비스](https://docs.microsoft.com/azure/container-service/container-service-intro)는 Docker Swarm, Kubernetes, DC/OS를 통해 관리되는 Docker 컨테이너를 지원합니다.

#### 기타 계산 서비스


Azure는 AWS에서 직접 대응하는 것이 없는 몇 가지 계산 서비스를 제공합니다:

-   [Azure 일괄 처리](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) - 확장 가능한 가상 컴퓨터들에서 계산 집약적 작업을 관리할 수 있습니다.

-   [서비스 패브릭](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) - 확장 가능한
    [마이크로서비스](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/)솔루션을 개발하고 관리하기 위한 플랫폼.
    
#### 참고 항목

-   [포털을 사용하여 Azure에서 Linux VM 만들기](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [Azure 참조 아키텍처: Azure에서 Linux VM 실행하기](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [Azure 앱 서비스에서 Node.js 웹 앱 시작하기](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [Azure 참조 아키텍처: 기본적인 웹 응용 프로그램](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [첫 번째 Azure 함수 만들기](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### 저장소

#### S3/EBS/EFS와 Azure 저장소

AWS 플랫폼에서 클라우드 저장소는 주로 3가지 서비스로 나뉩니다:

-   **단순 저장소 서비스 (S3)** - 기본 개체 저장소. 데이터를 인터넷 접근 가능한 API를 통해서 이용 가능하게 합니다.

-   **Elastic 블록 저장소 (EBS)** - 단일 VM에서 접근하기 위한 블록 수준의 저장소.

-   **Elastic 파일 시스템 (EFS)** - 수천 개의 EC2 인스턴스에 대한 공유 저장소로 사용하고자 한 파일 저장소.

Azure 저장소에서, 반드시 구독해야 하는 [저장소 계정](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/)
을 이용하여 다음과 같은 저장소 서비스를 만들고 관리할 수 있습니다:

-   [Blob 저장소](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) - 문서, 미디어 파일, 응용 프로그램 설치 관리자와 같은 모든 형식의 텍스트 또는 이진 데이터를 저장합니다. Blob 저장소를 개인 액세스용으로 설정하거나 콘텐츠를 인터넷에 공개적으로 공유할 수 있습니다. Blob 저장소는 AWS S3 및 EBS와 동일한 목적으로 사용됩니다.

-   [테이블 저장소](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) - 구조화된 데이터세트를 저장합니다. 테이블 저장소는 다량의 데이터를 신속하게 개발하고 빠르게 액세스할 수 있는 NoSQL 키-특성 데이터 저장소입니다.  AWS의 심플DB(SympleDB) 및 다이나포DB(DynamoDB) 서비스와 유사함.

-   [큐 저장소](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - 클라우드 서비스의 구성요소들에서 워크플로 처리 및 통신을 위한 메시지를 전송합니다. 

-   [파일 저장소](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) - 표준 서버 메시지 블록(SMB) 프로토콜을 사용하여 레거시 응용 프로그램을 위한 공유 저장소를 제공합니다.   파일 저장소는 AWS 플랫폼의 EFS와 유사하게 사용됩니다. 

#### Glacier와 Azure 저장소

Azure 저장소는 AWS의 장기 보존 Glacier 저장소와 동일한 기능을 제공하지 않습니다.  자주 액세스 하지 않는 수명이 긴 데이터의 경우 Azure는 [Azure cool blob 저장소 계층](https://azure.microsoft.com/documentation/articles/storage-blob-storage-tiers/)을 제공합니다. Cool 저장소는 표준 blob 저장소보다 싸고 성능이 낮은 스토리지를 제공하는데 이는 AWS의 S3에 필적합니다 - 빈번하지 않은 액세스.

#### 참고 항목

-   [Microsoft Azure 저장소 성능 및 확장성 점검표](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [Azure 저장소 보안 지침](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [패턴 및 실례 콘텐츠 전송망(CDN) 지침](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### 네트워킹

#### Elastic 부하 분산, Azure 부하 분산 장치, Azure 응용 프로그램 게이트웨이

두 가지 Elastic 부하 분산 서비스에 상응하는 Azure 기능:
-   [부하 분산 장치](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) - AWS의 클래식 부하 분산 장치와 같은 기능을 제공하며, 네트워크 수준에서 여러 VM을 위하여 트래픽을 배분합니다.  장애조치 기능도 제공합니다.

-   [응용 프로그램 게이트웨이](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) - AWS 응용 프로그램 부하 분산 장치와 비교할만한 응용 프로그램 수준의 규칙 기반 라우팅을 제공합니다.

#### 라우트 53, Azure DNS, Azure 트래픽 관리자

AWS에서, 라우트 53은 DNS 이름 관리와 DNS 수준의 트래픽 라우팅, 장애조치 서비스를 제공합니다.  Azure에서 이는 두 가지 서비스를 통해서 처리됩니다.

-   [Azure DNS](https://azure.microsoft.com/documentation/services/dns/) - 도메인 및 DNS 관리 기능을 제공합니다.

-   [트래픽 관리자](https://azure.microsoft.com/documentation/articles/traffic-manager-overview/) - DNS 수준의 트래픽 라우팅, 부하 분산, 장애조치 기능을 제공합니다.

#### 직접 연결 및 Azure ExpressRoute

Azure는
[ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/)
서비스를 통해서 사이트 간 전용 연결을 제공합니다. ExpressRoute를 이용할 경우 전용 개인 네트워크 연결을 통해서 Azure 리소스에 로컬 네트워크를 직접 연결할 수 있습니다. Azure는 보다 전통적인 [사이트 간 VPN 연결](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)을 낮은 비용으로 제공합니다.

#### 참고 항목

-   [Azure 포털을 사용한 가상 컴퓨터 만들기 ](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [Azure 가상 네트워크의 계획 및 설계](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [Azure 네트워크 보안 모범 사례](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### 데이터베이스 서비스

#### RDS 및 Azure SQL 데이터베이스 서비스

AWS와 Azure는 클라우드에서 관계형 데이터베이스 제품에 대한 접근법이 다릅니다.  AWS의 관계형 데이터베이스 서비스(RDS)는 Oracle과 MySQL 등 여러 다양한 데이터베이스 엔진을 사용하여 인스턴스 만들기를 지원합니다.

[SQL 데이터베이스](https://azure.microsoft.com/documentation/articles/sql-database-technical-overview/)는 Azure의 클라우드 데이터베이스 제품입니다. 이 제품은 관리 서비스를 통해서 확장성이 뛰어난 관계형 데이터 저장소를 제공합니다. SQL 데이터베이스는 자체 엔진을 사용하고, 다른 종류의 데이터베이스의 생성을 지원하지 않습니다.
SQL 서버](https://azure.microsoft.com/services/virtual-machines/sql-server/), [Oracle](https://azure.microsoft.com/campaigns/oracle/),
[MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/)등 다른 데이터베이스 엔진은 Azure VM 인스턴스를 써서 배포될 수 있습니다.

AWS RDS 비용은 CPU, RAM, 저장소, 네트워크 대역폭 등 인스턴스가 사용하는 하드웨어 리소스의 수량에 따라 결정됩니다.  SQL 데이터베이스 서비스에서 비용은 데이터베이스 크기, 동시 연결, 처리량 수준에 따라 달라집니다. 

#### 참고 항목

-   [Azure SQL 데이터베이스 Database 지침서](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [Azure 포털로 Azure SQL 데이터베이스의 지역 복제 구성하기 ](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [DocumentDB 소개: NoSQL JSON 데이터베이스](https://azure.microsoft.com/documentation/articles/documentdb-introduction/)

-   [Node.js에서 Azure 데이블 저장소를 사용하는 방법](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### 보안 및 신원확인

#### 디렉터리 서비스와 Azure Active Directory

Azure는 디렉터리 서비스를 다음 제품들로 나눕니다.

-   [Azure Active
    Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) - 클라우드 기반의 디렉터리 및 신원확인 관리 서비스.

-   [Azure Active Directory
    B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) - 파트너-관리 신원확인을 통해서 회사의 응용 프로그램을 액세스할 수 있습니다. 

-   [Azure Active Directory
    B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) - 소비자가 직접 대면하는 응용 프로그램에 대한 싱글 사인 온과 사용자 관리에 필요한 서비스 제공 지원.

-   [Azure Active Directory Domain
    Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) - 도메인 컨트롤러 서비스를 관리하며, Active Directory와 호환되는 도메인 가입 및 사용자 관리 기능을 허용합니다.

#### 웹 응용 프로그램 방화벽

[응용 프로그램 게이트웨이 웹 응용 프로그램 방화벽](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)뿐만 아니라, [Barracuda
Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/)와 같은 타 공급업체의 [웹 응용 프로그램 방화벽](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/)을 사용할 수 있습니다.

#### 참고 항목
-   [Microsoft Azure 보안 시작하기](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [Azure 신원확인 관리 및 출입 통제 보안의 모범 사례](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### 응용 프로그램 및 메시징 서비스

#### 심플 이메일 서비스

AWS는 알림, 업무상 또는 마케팅 이메일을 전송하기 위한 심플 이메일 서비스(SES)를 제공합니다. Azure에서,
[Sendgrid](https://sendgrid.com/partners/azure/)와 같은 타사 솔루션이 이메일 서비스를 제공합니다.

#### 심플 큐 서비스

AWS 심플 큐 서비스(SQS)는 AWS 플랫폼에서 응용 프로그램, 서비스, 장치를 연결하기 위한 메시징 시스템을 제공합니다. Azure에는 비슷한 기능을 제공하는 두 가지 서비스가 있습니다. 

-   [큐 저장소](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) - Azure 플랫폼에서 응용 프로그램 구성요소들 사이에 통신을 할 수 있는 클라우드 메시징 서비스.

-   [서비스 버스](https://azure.microsoft.com/en-us/services/service-bus/) - 응용 프로그램, 서비스, 장치를 연결하기 위한 보다 강력한 메시징 시스템. 관련된 [서비스 버스 릴레이](https://docs.microsoft.com/en-us/azure/service-bus-relay/relay-what-is-it)를 사용하여, 서비스 버스가 원격 관리되는 응용 프로그램 및 서비스에 연결될 수도 있습니다.

#### 디바이스 팜(Device Farm)

AWS 디바이스 팜은 장치 간 시험 서비스를 제공합니다. Azure에서, [Xamarin 시험 클라우드](https://www.xamarin.com/test-cloud)는 모바일 장치에 대하여 비슷한 장치 간 프런트 엔드 테스트를 제공합니다.

[Azure DevTest 랩](https://azure.microsoft.com/services/devtest-lab/)은 프런트 엔드 테스트 뿐만 아니라 Linux와 Windows 환경에 대한 백 엔드 테스트 리소스를 제공합니다.

#### 참고 항목

-   [Node.js에서 큐 저장소를 사용하는 방법](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [서비스 버스 큐 사용 방법](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/)

### 분석 및 빅 데이터

[Cortana 인텔리전스 도구 모음](https://azure.microsoft.com/suites/cortana-intelligence-suite/)은 많은 데이터를 포착, 구성, 분석, 시각화하도록 설계된 Azure의 제품 및 서비스 패키지입니다. Cortana 도구 모음은 다음 서비스들로 구성됩니다:

-   [HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) - Hadoop, Spark, Storm, HBase를 포함하는 관리형 Apache 배포판입니다.

-   [데이터 팩토리](https://azure.microsoft.com/documentation/services/data-factory/) - 데이터 통제(orchestration) 및 데이터 파이프라인 기능을 제공합니다.

-   [SQL 데이터 웨어하우스](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) - 대규모 관계형 데이터 저장소.

-   [데이터 레이크 저장소](https://azure.microsoft.com/documentation/services/data-lake-store/) - 빅 데이터 분석 작업에 최적화된 대규모 저장소.

-   [기계 학습](https://azure.microsoft.com/documentation/services/machine-learning/) - 데이터에 대한 예측 분석을 빌드하고 적용하는 데 사용됨.

-   [스트림 분석](https://azure.microsoft.com/documentation/services/stream-analytics/) - 실시간 데이터 분석.

-   [데이터 레이크 분석](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) - 데이터 레이크 저장소와 작업할 수 있도록 최적화된 대규모 분석 서비스

-   [PowerBI](https://powerbi.microsoft.com/) - 전원 데이터 시각화에 사용됨.

#### 참고 항목

-   [Cortana 인텔리전스 갤러리](https://gallery.cortanaintelligence.com/)

-   [Microsoft 빅 데이터 솔루션 이해하기](https://msdn.microsoft.com/library/dn749804.aspx)

-   [Azure 데이터 레이크 & Azure HDInsight 블로그](https://blogs.msdn.microsoft.com/azuredatalake/)

### 사물 인터넷

#### 참고 항목

-   [Azure IoT 허브 시작하기](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [IoT 허브와 이벤트 허브 비교](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### 모바일 서비스

#### 알림

알림 허브는 SMS나 Email발송을 지원하지 않습니다. 그렇기 때문에 SMS, Email을 발송하기 위해서는 서드파티 서비스를 사용하셔야 합니다.

#### 참고 항목

-   [Android 앱 만들기](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [Azure 모바일 앱의 인증 및 권한 부여](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [Azure 알림 허브로 푸시 알림 전송](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### 관리 및 모니터링

#### 참고 항목

-   [모니터링 및 진단 지침](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [Azure 리소스 관리자 템플릿 작성에 대한 모범 사례](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [Azure 리소스 관리자 빠른 시작 템플릿](https://azure.microsoft.com/documentation/templates/)


## 다음 단계

-   [AWS와 Azure 서비스 전체 비교표](https://aka.ms/azure4aws-services)

-   [대화형 Azure 플랫폼 빅 픽처](http://azureplatform.azurewebsites.net/)

-   [Azure 시작하기](https://azure.microsoft.com/get-started/)

-   [Azure 솔루션 아키텍처](https://azure.microsoft.com/solutions/architecture/)

-   [Azure 참조 아키텍처](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

-   [패턴 및 실례 Azure 지침](https://azure.microsoft.com/documentation/articles/guidance/)

-   [무료 온라인 과정 AWS 전문가를 위한 Microsoft Azure](http://aka.ms/azureforaws)
