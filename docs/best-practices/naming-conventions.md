---
title: Naming conventions for Azure resources
description: Naming conventions for Azure resources. How to name virtual machines, storage accounts, networks, virtual networks, subnets and other Azure entities
author: dragon119
ms.service: guidance
ms.topic: article
ms.date: 10/31/2016
ms.author: pnp

pnp.series.title: Best Practices
---
# 명명 규칙

[!INCLUDE [header](../_includes/header.md)]

이 문서는 Azure 리소스에 대한 명명 규칙 및 제한 사항의 요약과 명명 규칙에 대한 기본 권장 사항을 수록하고 있습니다. 이러한 권장 사항을 자신의 필요에 맞는 고유한 규칙을 정하기 위한 시작점으로 사용할 수 있습니다.  

Microsoft Azure의 모든 리소스에 대한 이름 선택은 다음과 같은 이유로 중요합니다. 

* 나중에 이름을 변경하기가 어렵습니다.
* 이름은 해당 리소스 유형의 요구 사항을 충족시켜야 합니다.

일관된 명명 규칙을 사용하면 리소스를 보다 쉽게찾을 수 있습니다. 또한 솔루션에서 리소스의 역할을 나타낼 수도 있습니다.   

명명 규칙을 성공으로 이끄는 열쇠는 응용 프로그램과 조직 전체에서 이를 수립하고 따르는 것입니다.   

## 구독 이름 지정
Azure 구독의 이름을 지정할 때 자세한 이름을 사용하면 각 구독의 컨텍스트와 목적을 명확하게 이해할 수 있습니다. 구독이 많은 환경에서 작업할 때 공유 명명 규칙을 따르면 명확성을 높일 수 있습니다. 

구독 이름 지정에 권장되는 패턴은 다음과 같습니다. 

`<회사> <부서(선택 사항)> <제품 라인(선택 사항)> <환경>`

* 회사는 일반적으로 각 구독마다 동일합니다. 그러나 일부 회사는 조직 구조 내 하위 회사를 보유하고 있을 수 있습니다. 이러한 회사는 중앙 IT 그룹에서 관리할 수 있습니다. 이 경우, 부모 회사 이름(*Contoso*) 과 자식 회사 이름(*North Wind*) 을 모두 구분하여 차별화할 수 있습니다.
* 부서는 개인 그룹이 활동하는 조직 내의 이름입니다. 네임스페이스 내의 이 항목은 선택 사항입니다.
* 제품 라인은 부서 내에서 수행되는 기능 또는 제품의 특정 이름입니다. 이는 일반적으로 내부용 서비스 및 응용 프로그램의 경우 선택 사항입니다. 그러나 쉽게 분리 및 식별해야 하는 공공 서비스(예: 청구 기록의 분명한 분리)에 사용하는 것이 매우 좋습니다.
* 환경이란 Dev, QA 또는 Prod와 같은 응용 프로그램이나 서비스의 배포 수명 주기를 설명하는 이름입니다.

| 회사 | 부서 | 제품 라인 또는 서비스 | 환경 | 전체 이름 |
| --- | --- | --- | --- | --- |
| Contoso |SocialGaming |AwesomeService |Production |Contoso SocialGaming AwesomeService Production |
| Contoso |SocialGaming |AwesomeService |Dev |Contoso SocialGaming AwesomeService Dev |
| Contoso |IT |InternalApps |Production |Contoso IT InternalApps Production |
| Contoso |IT |InternalApps |Dev |Contoso IT InternalApps Dev |

<!-- TODO; include more information about organizing subscriptions for application deployment, pods, etc. -->

## 모호함을 피하기 위해 접속어 사용
Azure에서 리소스의 이름을 지정할 때 공통 접두어나 접미어를 사용하여 리소스의 유형과 컨텍스트를 식별하는 것이 좋습니다. 유형, 메타데이터, 컨텍스트에 대한 모든 정보는 프로그래밍 방식으로 사용할 수 있지만 공통 접미어를 적용하면 시각적 식별이 간단해집니다. 접속어를 명명 규칙에 포함시키려면 접속어가 이름의 시작 부분(접두사) 또는 끝에 있는지(접미어) 명확하게 지정하는 것이 중요합니다.  

예를 들어, 다음은 계산 엔진을 호스팅하는 서비스의 두 가지 가능한 이름입니다. 

* SvcCalculationEngine (접두어)
* CalculationEngineSvc (접미어)

접속어는 특정 리소스를 설명하는 다양한 측면을 나타낼 수 있습니다. 다음 표는 일반적으로 사용되는 몇 가지 예를 보여줍니다. 

| 측면 | 예 | 참고 |
| --- | --- | --- |
| 환경 |dev, prod, QA |리소스의 환경을 식별합니다. |
| 위치 |uw (US West), ue (US East) |리소스가 배포되는 지역을 식별합니다. |
| 인스턴스 |01, 02 |둘 이상의 명명된 인스턴스(웹 서버 등)가 있는 리소스의 경우 |
| 제품 또는 서비스 |서비스 |리소스가 지원하는 제품, 응용 프로그램 또는 서비스를 식별합니다. |
| 역할 |sql, web, messaging |관련 리소스의 역할을 식별합니다. |

회사나 프로젝트에 대한 특정 명명 규칙을 개발할 때는 공통 접속어 세트와 위치(접미어 또는 접두어)를 선택하는 것이 중요합니다. 

## 명명 규칙 및 제한 사항
Azure의 각 리소스 또는 서비스 유형은 일련의 명명 제한과 범위를 시행합니다. 모든 명명 규칙 또는 패턴은 필수 명명 규칙과 범위를 준수해야 합니다. 예를 들어, VM의 이름이 DNS 이름에 매핑되기 때문에(따라서 모든 Azure에서 고유해야 함) VNET의 이름은 해당 호스트가 생성된 리소스 그룹으로 범위 지정됩니다. 

일반적으로 특수 문자(`-` 또는 `_`)를 어떤 이름의 첫 번째 또는 마지막 문자로 사용하지 마십시오. 이러한 문자로 인해 대부분의 유효성 검사 규칙이 실패하게 됩니다. 

| 카테고리 | 서비스 또는 엔터티 | 범위 | 길이 | 대/소문자 | 유효한 문자 | 제안 패턴 | 예 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 리소스 그룹 |리소스 그룹 |전역 |1-64 |대소문자 구분 안 함 |영숫자, 밑줄 및 하이픈 |`<서비스 약식 이름>-<환경>-rg` |`profx-prod-rg` |
| 리소스 그룹 |가용성 설정 |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 밑줄 및 하이픈 |`<서비스 약식 이름>-<컨텍스트>-as` |`profx-sql-as` |
| 일반 |태그 |관련 엔터티 |512 (이름), 256 (값) |대소문자 구분 안 함 |영숫자 |`"키" : "값"` |`"부서" : "중앙 IT"` |
| 컴퓨팅 |가상 컴퓨터 |리소스 그룹 |1-15 |대소문자 구분 안 함 |영숫자, 밑줄 및 하이픈 |`<이름>-<역할>-vm<번호>` |`profx-sql-vm1` |
| 저장소 |저장소 계정 이름(데이터) |전역 |3-24 |소문자 |영숫자 |`<전역으로 고유한 이름><번호>` (저장소 계정 이름 지정을 위한 고유한 GUID를 계산하는 함수 사용) |`profxdata001` |
| 저장소 |저장소 계정 이름(디스크) |전역 |3-24 |소문자 |영숫자 |`<대시를 제외한 vm 이름>st<번호>` |`profxsql001st0` |
| 저장소 |컨테이너 이름 |저장소 계정 |3-63 |소문자 |영숫자 및 대시 |`<컨텍스트>` |`로그` |
| 저장소 |Blob 이름 |컨테이너 |1-1024 |대소문자 구분 |모든 URL 문자 |`<Blob 사용에 따라 가변적>` |`<Blob 사용에 따라 가변적>` |
| 저장소 |대기열 이름 |저장소 계정 |3-63 |소문자 |영숫자 및 대시 |`<서비스 약식 이름>-<컨텍스트>-<번호>` |`awesomeservice-messages-001` |
| 저장소 |테이블 이름 |저장소 계정 |3-63 |대소문자 구분 안 함 |영숫자 |`<서비스 약식 이름>-<컨텍스트>` |`awesomeservice-logs` |
| 저장소 |파일 이름 |저장소 계정 |3-63 |소문자 |영숫자 |`<Blob 사용에 따라 가변적>` |`<Blob 사용에 따라 가변적>` |
| 네트워킹 |가상 네트워크(VNet) |리소스 그룹 |2-64 |대소문자 구분 안 함 |영숫자, 밑줄, 대시 및 마침표 |`<서비스 약식 이름>-[섹션]-vnet` |`profx-vnet` |
| 네트워킹 |서브넷 |상위 VNet |2-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<설명적 텍스트>` |`웹` |
| 네트워킹 |네트워크 인터페이스 |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<vmname>-nic<num>` |`profx-sql1-nic1` |
| 네트워킹 |네트워크 보안 그룹 |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<서비스 약식 이름>-<컨텍스트>-nsg` |`profx-app-nsg` |
| 네트워킹 |네트워크 보안 그룹 역할 |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<설명적 텍스트>` |`sql-allow` |
| 네트워킹 |공용 IP 주소 |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<vm 또는 서비스 이름>-pip` |`profx-sql1-pip` |
| 네트워킹 |부하 분산 장치 |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<서비스 또는 역할>-lb` |`profx-lb` |
| 네트워킹 |부하 분산된 규칙 구성 |부하 분산 장치 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<설명적 텍스트>` |`http` |
| 네트워킹 |Azure Application Gateway |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<서비스 또는 역할>-aag` |`profx-agw` |
| 네트워킹 |Traffic Manager Profile |리소스 그룹 |1-63 |대소문자 구분 안 함 |영숫자, 대시 및 마침표 |`<설명적 텍스트>` |`app1` |


## 태그를 사용하여 리소스 구성
Azure Resource Manager는 컨텍스트를 식별하고 자동화를 능률화하기 위해 임의의 텍스트 문자열로 태그 지정된 엔터티를 지원합니다. 예를 들어, `"sqlVersion: "sql2014ee"` 태그는 SQL Server 2014 Enterprise Edition을 실행하는 배포에서 자동 스크립트를 실행하는 VM을 식별할 수 있습니다. 선택한 명명 규칙의 측면에서 컨텍스트를 보강하고 향상시키는 데 태그를 사용해야 합니다. 

> [!팁]
> 태그의 또 다른 장점 중 하나는 태그가 여러 리소스 그룹에 걸쳐져 있어서 서로 다른 배포 사이에 엔터티를 연결하고 상호 연관시킬 수 있다는 것입니다. 
> 
> 

각 리소스 또는 리소스 그룹은 최대 **15** 개의 태그를 가질 수 있습니다. 태그 이름은 512자로 제한되며 태그 값은 256자로 제한됩니다.

리소스 태그 지정에 대한 자세한 내용은 [태그를 사용하여 Azure 리소스 구성](/azure/azure-resource-manager/resource-group-using-tags/)을 참조하십시오. 

몇 가지 일반적인 태그 지정의 사용 사례는 다음과 같습니다. 

* **청구**; 리소스를 그룹화하고 청구 또는 환급 코드와 연관시킵니다.
* **서비스 컨텍스트 식별**; 일반적인 작업 및 그룹화를 위해 전체 리소스 그룹에서 리소스의 그룹을 식별합니다.
* **액세스 제어 및 보안 컨텍스트**; 포트폴리오, 시스템, 서비스, 응용 프로그램, 인스턴스 등을 기반으로 하는 관리 역할 식별

> [!팁]
> 태그를 일찍, 자주 처리하십시오. 사실에 기반하여 사후 처리하는 것보다 기본 태그 처리 방침을 마련해 놓고 시간에 따라 조정하는 것이 좋습니다.  
> 
> 

일반적인 태그 처리 접근법의 예: 

| 태그 이름 | 키 | 예 | 설명 |
| --- | --- | --- | --- |
| 청구 주소/내부 환급 ID |billTo |`IT-Chargeback-1234` |내부 I/O 또는 청구 코드 |
| 운영자 또는 직속 담당자(DRI) |managedBy |`joe@contoso.com` |별칭 또는 이메일 주소 |
| 프로젝트 이름 |project-name |`myproject` |프로젝트 또는 제품 라인 이름 |
| 프로젝트 버전 |project-version |`3.4` |프로젝트 또는 제품 라인 버전 |
| 환경 |환경 |`<생산, 단계, QA >` |환경 식별자 |
| 계층 |계층 |`전면 끝, 후면 끝, 데이터` |계층 또는 역할/컨텍스트 식별 |
| 데이터 프로파일 |dataProfile |`공용, 기밀, 제한, 내부` |리소스에 저장된 데이터의 민감도 |

## 팁과 트릭
일부 리소스 유형은 명명 및 규칙과 관련해 추가적인 주의가 필요할 수 있습니다. 

### 가상 컴퓨터
특히 규모가 큰 토폴로지에서는 가상 컴퓨터를 신중하게 명명하면 각 컴퓨터의 역할과 용도를 보다 효과적으로 파악하고 스크립트를 예측 가능하게 작성할 수 있습니다. 

> [!경고]
> Azure의 모든 가상 컴퓨터에는 Azure 리소스 이름과 운영 체제 호스트 이름이 둘 다 있습니다. 
> 리소스 이름과 호스트 이름이 다른 경우 VM 관리가 어려울 수 있으므로 이런 경우가 발생하지 않도록 주의해야 합니다. 
> 호스트 이름이 구성된 운영 체제를 이미 포함하고 있는 .vhd에서 가상 컴퓨터를 생성하는 경우를 예로 들 수 있습니다. 
> 
> 

* [Microsoft NetBIOS 컴퓨터 명명 규칙](https://support.microsoft.com/en-us/help/188997/microsoft-netbios-computer-naming-conventions)

### 저장소 계정 및 저장소 엔터티
저장소 계정에는 VM용 디스크를 백업하고 BLOB, 대기열 및 테이블에 데이터를 저장하는 두 가지 주요 사용 사례가 있습니다. VM 디스크에 사용되는 저장소 계정은 상위 VM 이름과 연관시키는 명명 규칙을 따라야 합니다(및 하이엔드 VM SKU의 경우 여러 저장소 계정이 필요할 수 있으며 번호 접미어도 추가해야 함). 

> [!팁]
> 저장소 계정(데이터 또는 디스크)은 여러 저장소 계정을 활용할 수 있도록 하는 명명 규칙(즉, 항상 숫자 접미어 사용)을 따라야 합니다. 
> 
> 

Azure 저장소 계정의 Blob 데이터에 액세스하기 위한 사용자 지정 도메인 이름을 구성할 수 있습니다. Blob 서비스의 기본 끝점은 `https://mystorage.blob.core.windows.net` 입니다. 

그러나 사용자 지정 도메인(예: www.contoso.com) 을 저장소 계정의 Blob 끝점에 매핑하면 해당 도메인을 사용하여 저장소 계정의 BLOB 데이터에 액세스할 수도 있습니다. 예를 들어, 사용자 지정 도메인 이름을 사용하면 `http://mystorage.blob.core.windows.net/mycontainer/myblob` 에
`http://www.contoso.com/mycontainer/myblob` 로 액세스할 수 있습니다. 

이 기능 구성에 대한 자세한 내용은 [Blob 저장소 끝점에 대한 사용자 지정 도메인 이름 구성](/azure/storage/storage-custom-domain-name/)을 참조하십시오. 

Blob, 컨테이너 및 테이블의 이름 지정에 대한 자세한 내용은 다음을 참조하십시오. 

* [컨테이너, Blob 및 메타데이터 명명 및 참조](https://msdn.microsoft.com/library/dd135715.aspx)
* [대기열 및 메타데이터 명명](https://msdn.microsoft.com/library/dd179349.aspx)
* [명명 테이블](https://msdn.microsoft.com/library/azure/dd179338.aspx)

Blob 이름은 임의의 문자 조합을 포함할 수 있지만 예약된 URL 문자는 올바르게 이스케이프시켜야 합니다. 마침표(.), 슬래시(/) 또는 시퀀스, 또는 이들의 두 조합으로 끝나는 Blob 이름은 사용하지 마십시오. 규칙에 따라 슬래시는 **가상** 디렉터리 구분 기호입니다. Blob 이름에 백슬래시(\)를 사용하지 마십시오. 클라이언트 API가 이를 허용할 수 있지만 제대로 해시되지 못하면 서명이 일치하지 않습니다. 

저장소 계정 또는 컨테이너를 만든 후에는 이름을 수정할 수 없습니다. 새 이름을 사용하려면 일단 삭제하고 새 이름을 만들어야 합니다. 

> [!팁]
> 새 서비스 또는 응용 프로그램을 개발하기 전에 모든 저장소 계정 및 유형에 대한 명명 규칙을 수립하는 것이 좋습니다.
> 
> 