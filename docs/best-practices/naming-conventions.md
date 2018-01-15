---
title: "Azure 리소스에 대한 명명 규칙"
description: "Azure 리소스에 대한 명명 규칙 가상 머신, 저장소 계정, 네트워크, 가상 네트워크, 서브넷 및 기타 Azure 엔터티의 이름을 지정하는 방법"
author: telmosampaio
ms.date: 05/18/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 364735dec9658b4d2a9d21330f38c57f6fa694bd
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/08/2018
---
# <a name="naming-conventions"></a>명명 규칙

[!INCLUDE [header](../_includes/header.md)]

이 문서는 Azure 리소스에 대한 명명 규칙 및 제한 사항에 대한 요약과, 명명 규칙에 대한 권장 사항의 기준 집합을 제공합니다.  필요에 따라 사용자 고유의 규칙에 대한 시작 지점으로 이러한 권장 사항을 사용할 수 있습니다.

Microsoft Azure에서 모든 리소스에 대한 이름 선택은 다음 이유로 인해 중요합니다.

* 나중에 이름을 변경하는 것은 어렵습니다.
* 이름은 해당 특정 리소스 유형의 요구 사항을 충족해야 합니다.

일관된 명명 규칙은 리소스를 더 쉽게 찾을 수 있도록 합니다. 솔루션에서 리소스의 역할을 표시할 수도 있습니다.

성공적인 명명 규칙이 되기 위한 핵심은 응용 프로그램과 조직 전체에서 규칙을 정하여 따르는 것입니다.

## <a name="naming-subscriptions"></a>구독 명명
Azure 구독을 명명하는 경우 자세한 이름은 각 구독의 컨텍스트 및 용도에 대한 이해를 명확하게 합니다.  많은 구독이 있는 환경에서 작업하는 경우 공유 명명 규칙을 따르면 명확성을 크게 높일 수 있습니다.

구독 명명에 대한 권장되는 패턴은 다음과 같습니다.

`<Company> <Department (optional)> <Product Line (optional)> <Environment>`

* 일반적으로 회사는 각 구독에 대해 동일할 것입니다. 그러나 일부 회사에서는 조직 구조 내에 자식 회사가 있을 수 있습니다. 이러한 회사는 중앙 IT 그룹에서 관리될 수 있습니다. 이 경우 부모 회사 이름(*Contoso*) 및 자식 회사 이름(*Northwind*)을 모두 사용하여 구분할 수 있습니다.
* 부서는 개인 작업의 그룹인 조직 내에서 이름입니다. 네임스페이스 내에서 이 항목은 선택적입니다.
* 제품 라인은 부서 내에서 수행되는 제품 또는 기능에 대한 특정 이름입니다. 일반적으로 내부 전용 서비스 및 응용 프로그램에는 선택 사항입니다. 쉬운 분리 및 ID(예: 청구 레코드의 명확한 분리)가 필요한 공용 서비스에 사용하는 것이 좋습니다.
* 환경은 개발, QA 또는 프로덕션 등의 응용 프로그램 또는 서비스의 배포 수명 주기를 설명하는 이름입니다.

| 회사 | 부서 | 제품 라인 또는 서비스 | 환경 | 전체 이름 |
| --- | --- | --- | --- | --- |
| Contoso |SocialGaming |AwesomeService |프로덕션 |Contoso SocialGaming AwesomeService 프로덕션 |
| Contoso |SocialGaming |AwesomeService |개발 |Contoso SocialGaming AwesomeService 개발 |
| Contoso |IT |InternalApps |프로덕션 |Contoso IT InternalApps 프로덕션 |
| Contoso |IT |InternalApps |개발 |Contoso IT InternalApps 개발 |

대규모 기업에 대해 구독을 구성하는 방법에 대한 자세한 내용은 [규범적인 구독 거버넌스 지침][scaffold]을 참조하세요.

## <a name="use-affixes-to-avoid-ambiguity"></a>모호성을 예방하기 위해 접사 사용

Azure에서 리소스를 명명하는 경우 리소스의 유형 및 컨텍스트를 확인하도록 공통 접두사 또는 접미사를 사용하는 것이 좋습니다.  유형, 메타데이터, 컨텍스트에 대한 모든 정보는 프로그래밍 방식으로 사용할 수 있지만 공통 접사를 적용하면 시각적 식별이 간편해집니다.  명명 규칙에 접사를 통합할 때 접사를 이름의 시작 부분(접두사) 또는 끝 부분(접미사)에 붙여야 하는지 여부를 명확하게 지정하는 것이 중요합니다.

예를 들어 다음은 계산 엔진을 호스팅하는 서비스에 가능한 두 이름입니다.

* SvcCalculationEngine(접두사)
* CalculationEngineSvc(접미사)

접사는 특정 리소스를 설명하는 다양한 측면을 참조할 수 있습니다. 다음 표에서 일반적으로 사용하는 일부 예를 보여줍니다.

| 측면 | 예 | 메모 |
| --- | --- | --- |
| Environment |개발, 프로덕션, QA |리소스에 대한 환경 식별 |
| 위치 |uw(미국 서부), ue(미국 동부) |리소스가 배포되는 지역 식별 |
| 인스턴스 |01, 02 |둘 이상의 명명된 인스턴스가 있는 리소스의 경우(웹 서버 등). |
| 제품 또는 서비스 |서비스 |리소스가 지원하는 제품, 응용 프로그램 또는 서비스 식별 |
| 역할 |sql, 웹, 메시징 |연결된 리소스의 역할 식별 |

회사 또는 프로젝트에 대한 특정 명명 규칙을 개발할 때 접사의 공통 집합과 해당 위치(접미사 또는 접두사)를 선택하는 것이 중요합니다.

## <a name="naming-rules-and-restrictions"></a>명명 규칙 및 제한 사항

Azure의 각 리소스 또는 서비스 유형은 명명 제한 및 범위 집합을 적용하며 모든 명명 규칙이나 패턴은 필수 명명 규칙과 범위를 따라야 합니다.  예를 들어 VM의 이름은 DNS 이름에 매핑되며(따라서 Azure 전체에서 고유해야 함) VNET의 이름은 해당 항목이 생성된 리소스 그룹 안으로 범위가 지정됩니다.

일반적으로 모든 이름의 처음과 마지막 글자로 특수 문자(`-` 또는 `_`)는 사용하지 않습니다. 대부분의 유효성 검사 실패는 특수 문자로 인해 발생합니다.

### <a name="general"></a>일반

| 엔터티 | 범위 | 길이 | 대/소문자 구분 | 사용할 수 있는 문자 | 제안된 패턴 | 예 |
| --- | --- | --- | --- | --- | --- | --- |
|리소스 그룹 |구독 |1-90 |대/소문자 구분하지 않음 |영숫자, 밑줄, 괄호, 하이픈 및 마침표(맨 끝 제외) |`<service short name>-<environment>-rg` |`profx-prod-rg` |
|가용성 집합 |리소스 그룹 |1-80 |대/소문자 구분하지 않음 |영숫자, 밑줄 및 하이픈 |`<service-short-name>-<context>-as` |`profx-sql-as` |
|태그 |연결된 엔터티 |512(이름), 256(값) |대/소문자 구분하지 않음 |영숫자 |`"key" : "value"` |`"department" : "Central IT"` |

### <a name="compute"></a>컴퓨팅

| 엔터티 | 범위 | 길이 | 대/소문자 구분 | 사용할 수 있는 문자 | 제안된 패턴 | 예 |
| --- | --- | --- | --- | --- | --- | --- |
|Virtual Machine |리소스 그룹 |1-15(Windows), 1-64(Linux) |대/소문자 구분하지 않음 |영숫자, 밑줄 및 하이픈 |`<name>-<role>-vm<number>` |`profx-sql-vm1` |
|함수 앱 | 전역 |1-60 |대/소문자 구분하지 않음 |영숫자 및 하이픈 |`<name>-func` |`calcprofit-func` |

> [!NOTE]
> Azure의 가상 머신에는 가상 머신 이름과 호스트 이름 등 두 가지 이름이 있습니다. 포털에서 VM을 만들 때 호스트 이름과 가상 머신 리소스 이름 모두에 동일한 이름을 사용합니다. 위의 제한은 호스트 이름에 대한 것입니다. 실제 리소스 이름은 최대 64자까지 가능합니다.

### <a name="storage"></a>Storage

| 엔터티 | 범위 | 길이 | 대/소문자 구분 | 사용할 수 있는 문자 | 제안된 패턴 | 예 |
| --- | --- | --- | --- | --- | --- | --- |
|Storage 계정 이름(데이터) |전역 |3-24 |소문자 |영숫자 |`<globally unique name><number>`(저장소 계정 명명을 위한 고유 GUID를 계산하는 함수 사용) |`profxdata001` |
|Storage 계정 이름(디스크) |전역 |3-24 |소문자 |영숫자 |`<vm name without dashes>st<number>` |`profxsql001st0` |
| 컨테이너 이름 |Storage 계정 |3-63 |소문자 |영숫자 및 대시 |`<context>` |`logs` |
|Blob 이름 | 컨테이너 |1-1024 |대/소문자 구분 |모든 URL 문자 |`<variable based on blob usage>` |`<variable based on blob usage>` |
|큐 이름 |Storage 계정 |3-63 |소문자 |영숫자 및 대시 |`<service short name>-<context>-<num>` |`awesomeservice-messages-001` |
|테이블 이름 | Storage 계정 |3-63 |대/소문자 구분하지 않음 |영숫자 |`<service short name><context>` |`awesomeservicelogs` |
|파일 이름 | Storage 계정 |3-63 |소문자 | 영숫자 |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Data Lake Store | 전역 |3-24 |소문자 | 영숫자 |`<name>-dls` |`telemetry-dls` |

### <a name="networking"></a>네트워킹

| 엔터티 | 범위 | 길이 | 대/소문자 구분 | 사용할 수 있는 문자 | 제안된 패턴 | 예 |
| --- | --- | --- | --- | --- | --- | --- |
|Virtual Network(VNet) |리소스 그룹 |2-64 |대/소문자 구분하지 않음 |영숫자, 대시, 밑줄 및 마침표 |`<service short name>-vnet` |`profx-vnet` |
|서브넷 |부모 VNet |2-80 |대/소문자 구분하지 않음 |영숫자, 대시, 밑줄 및 마침표 |`<descriptive context>` |`web` |
|네트워크 인터페이스 |리소스 그룹 |1-80 |대/소문자 구분하지 않음 |영숫자, 대시, 밑줄 및 마침표 |`<vmname>-nic<num>` |`profx-sql1-nic1` |
|네트워크 보안 그룹 |리소스 그룹 |1-80 |대/소문자 구분하지 않음 |영숫자, 대시, 밑줄 및 마침표 |`<service short name>-<context>-nsg` |`profx-app-nsg` |
|네트워크 보안 그룹 규칙 |리소스 그룹 |1-80 |대/소문자 구분하지 않음 |영숫자, 대시, 밑줄 및 마침표 |`<descriptive context>` |`sql-allow` |
|공용 IP 주소 |리소스 그룹 |1-80 |대/소문자 구분하지 않음 |영숫자, 대시, 밑줄 및 마침표 |`<vm or service name>-pip` |`profx-sql1-pip` |
|Load Balancer |리소스 그룹 |1-80 |대/소문자 구분하지 않음 |영숫자, 대시, 밑줄 및 마침표 |`<service or role>-lb` |`profx-lb` |
|부하 분산된 규칙 구성 |Load Balancer |1-80 |대/소문자 구분하지 않음 |영숫자, 대시, 밑줄 및 마침표 |`<descriptive context>` |`http` |
|Azure Application Gateway |리소스 그룹 |1-80 |대/소문자 구분하지 않음 |영숫자, 대시, 밑줄 및 마침표 |`<service or role>-agw` |`profx-agw` |
|Traffic Manager 프로필 |리소스 그룹 |1-63 |대/소문자 구분하지 않음 |영숫자, 대시 및 마침표 |`<descriptive context>` |`app1` |

## <a name="organize-resources-with-tags"></a>태그로 리소스 정리

Azure Resource Manager는 임의적인 텍스트 문자열로 태그 지정 엔터티를 지원하여 컨텍스트를 식별하고 자동화를 간소화합니다.  예를 들어 `"sqlVersion: "sql2014ee"` 태그는 자동화된 스크립트 실행을 위해 SQL Server 2014 Enterprise Edition을 실행하는 배포의 모든 VM을 식별할 수 있습니다.  태그는 선택한 명명 규칙에 대한 컨텍스트를 보강 및 향상하는 데 사용되어야 합니다.

> [!TIP]
> 태그의 다른 한 장점은 태그는 서로 다른 배포 전체에서 엔터티를 연결 및 상관 관계를 지정할 수 있도록 하는 리소스 그룹을 확장한다는 점입니다.

각 리소스 또는 리소스 그룹에는 최대 **15** 개의 태그가 포함될 수 있습니다. 태그 이름은 512자로 제한되며 태그 값은 256자로 제한됩니다.

리소스 태그 지정에 대한 자세한 내용은 [태그를 사용하여 Azure 리소스 구성](/azure/azure-resource-manager/resource-group-using-tags/)을 참조하세요.

일반적인 태그 사용 사례는 다음과 같습니다.

* **청구**; 리소스를 그룹화하고 청구 또는 차지 백 코드와 연결합니다.
* **서비스 컨텍스트 ID**; 일반 작업 및 그룹화에 대한 리소스 그룹의 리소스 그룹 식별
* **Access Control 및 보안 컨텍스트**; 포트폴리오, 시스템, 서비스, 앱, 인스턴스 등을 기준으로 관리 역할 식별

> [!TIP]
> 태그 초기 - 태그인 경우가 많습니다.  사후에 개/보수하는 것보다 기준 태그 지정 체계를 준비하고 시간에 따라 조정하는 것이 좋습니다.

일부 일반적인 태그 지정 방법의 예:

| 태그 이름 | 키 | 예 | 주석 |
| --- | --- | --- | --- |
| 청구 대상 / 내부 차지백 ID |청구 대상 |`IT-Chargeback-1234` |내부 I/O 또는 청구 코드 |
| 운영자 또는 DRI(직접적인 책임자) |managedBy |`joe@contoso.com` |별칭 또는 메일 주소 |
| 프로젝트 이름 |projectName |`myproject` |프로젝트 또는 제품 라인 이름 |
| 프로젝트 버전 |projectVersion |`3.4` |프로젝트 또는 제품 라인 버전 |
| Environment |환경 |`<Production, Staging, QA >` |환경 식별자 |
| 계층 |계층 |`Front End, Back End, Data` |계층 또는 역할/컨텍스트 식별 |
| 데이터 프로필 |dataProfile |`Public, Confidential, Restricted, Internal` |리소스에 저장된 데이터의 민감도 |

## <a name="tips-and-tricks"></a>팁과 요령

일부 리소스 유형은 이름 지정 및 규칙에 주의해야 합니다.

### <a name="virtual-machines"></a>가상 머신

특히 더 큰 토폴로지에서 신중하게 가상 머신을 명명하는 것은 각 컴퓨터의 역할 및 용도 식별을 간소화하고 예측 가능성이 더욱 뛰어난 스크립팅을 활성화합니다.

### <a name="storage-accounts-and-storage-entities"></a>Storage 계정 및 저장소 엔터티

저장소 계정에는 VM에 대한 디스크 지원과 Blob, 큐, 테이블에 데이터 저장 등, 두 가지 기본 사용 사례가 있습니다.  VM 디스크에 사용되는 저장소 계정은 부모 VM 이름과 연결하는 명명 규칙을 따라야 합니다(고급 VM SKU에 대해 여러 저장소 계정이 필요할 수 있음, 숫자 접미사도 사용).

> [!TIP]
> 저장소 계정(데이터 또는 디스크에 대한)은 여러 저장소 계정을 활용되도록 허용하는 명명 규칙을 따라야 합니다(즉 항상 숫자 접미사 사용).

Azure Storage 계정에서 Blob 데이터에 액세스할 수 있도록 사용자 지정 도메인 이름을 구성할 수 있습니다. Blob Service의 기본 끝점은 https://<name>.blob.core.windows.net`입니다.

그러나 사용자 지정 도메인(예: www.contoso.com)을 저장소 계정의 Blob 끝점에 매핑하는 경우 해당 도메인을 사용하여 저장소 계정의 Blob 데이터에 액세스할 수 있습니다. 예를 들어 사용자 지정 도메인 이름을 통해 `http://mystorage.blob.core.windows.net/mycontainer/myblob`에 `http://www.contoso.com/mycontainer/myblob`로 액세스할 수 있습니다.

이 기능 구성에 대한 자세한 내용은 [Blob 저장소 끝점에 대한 사용자 지정 도메인 이름 구성](/azure/storage/storage-custom-domain-name/)을 참조하세요.

Blob, 컨테이너 및 테이블 명명에 대한 자세한 내용은 다음 목록을 참조하세요.

* [컨테이너, BLOB, 메타데이터 이름 명명 및 참조](https://msdn.microsoft.com/library/dd135715.aspx)
* [큐 및 메타데이터 명명](https://msdn.microsoft.com/library/dd179349.aspx)
* [테이블 명명](https://msdn.microsoft.com/library/azure/dd179338.aspx)

Blob 이름은 문자 조합을 포함할 수 있지만 예약된 URL 문자는 적절히 이스케이프되어야 합니다. 마침표(.), 슬래시(/) 또는 시퀀스 또는 둘의 조합으로 끝나는 Blob 이름은 피합니다. 규칙에 따라 슬래시는 **가상** 디렉터리 구분 문자입니다. Blob 이름에 백슬래시(\\)를 사용하지 않습니다. 클라이언트 API는 이를 허용할 수도 있지만 이 경우 제대로 해시하지 않고 서명이 일치하지 않게 됩니다.

만든 후 저장소 계정 또는 컨테이너의 이름을 수정하는 것이 불가능합니다. 새 이름을 사용하려면 이름을 삭제하고 새로 만들어야 합니다.

> [!TIP]
> 새 서비스 또는 응용 프로그램의 개발을 시작하기 전에 모든 저장소 계정 및 유형에 대한 명명 규칙을 설정하는 것이 좋습니다.

<!-- links -->

[scaffold]: /azure/azure-resource-manager/resource-manager-subscription-governance
