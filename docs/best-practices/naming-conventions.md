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

`<회사> <부서(선택 사항))> <제품 라인(선택 사항)> <환경>`

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
| 네트워킹 |가상 네트워크(VNet) |리소스 그룹 |2-64 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<서비스 약식 이름>-[섹션]-vnet` |`profx-vnet` |
| 네트워킹 |서브넷 |상위 VNet |2-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<설명적 텍스트>` |`웹` |
| 네트워킹 |네트워크 인터페이스 |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<vmname>-nic<num>` |`profx-sql1-nic1` |
| 네트워킹 |네트워크 보안 그룹 |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<서비스 약식 이름>-<컨텍스트>-nsg` |`profx-app-nsg` |
| 네트워킹 |네트워크 보안 그룹 역할 |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<설명적 텍스트>` |`sql-allow` |
| 네트워킹 |공용 IP 주소 |리소스 그룹 |1-80 |Case-insensitive |영숫자, 대시, 밑줄 및 마침표 |`<vm 또는 서비스 이름>-pip` |`profx-sql1-pip` |
| 네트워킹 |부하 분산 장치 |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<서비스 또는 역할>-lb` |`profx-lb` |
| 네트워킹 |부하 분산된 규칙 구성 |부하 분산 장치 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<설명적 텍스트>` |`http` |
| 네트워킹 |Azure Application Gateway |리소스 그룹 |1-80 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<서비스 또는 역할>-aag` |`profx-agw` |
| 네트워킹 |Traffic Manager Profile |리소스 그룹 |1-63 |대소문자 구분 안 함 |영숫자, 대시, 밑줄 및 마침표 |`<설명적 텍스트>` |`app1` |


## Organizing resources with tags
The Azure Resource Manager supports tagging entities with arbitrary
text strings to identify context and streamline automation.  For example, the tag `"sqlVersion: "sql2014ee"` could identify VMs in a deployment running SQL Server 2014 Enterprise Edition for running an automated script against them.  Tags should be used to augment and enhance context along side of the naming conventions chosen.

> [!TIP]
> One other advantage of tags is that tags span resource groups, allowing you to link and correlate entities across
> disparate deployments.
> 
> 

Each resource or resource group can have a maximum of **15** tags. The tag name is limited to 512 characters, and the tag 
value is limited to 256 characters.

For more information on resource tagging, refer to [Using tags to organize your Azure resources](/azure/azure-resource-manager/resource-group-using-tags/).

Some of the common tagging use cases are:

* **Billing**; Grouping resources and associating them with billing or charge back codes.
* **Service Context Identification**; Identify groups of resources across Resource Groups for common operations and grouping
* **Access Control and Security Context**; Administrative role identification based on portfolio, system, service, app, instance, etc.

> [!TIP]
> Tag early - tag often.  Better to have a baseline tagging scheme in place and adjust over time rather than having
> to retrofit after the fact.  
> 
> 

An example of some common tagging approaches:

| Tag Name | Key | Example | Comment |
| --- | --- | --- | --- |
| Bill To / Internal Chargeback ID |billTo |`IT-Chargeback-1234` |An internal I/O or billing code |
| Operator or Directly Responsible Individual (DRI) |managedBy |`joe@contoso.com` |Alias or email address |
| Project Name |project-name |`myproject` |Name of the project or product line |
| Project Version |project-version |`3.4` |Version of the project or product line |
| Environment |environment |`<Production, Staging, QA >` |Environmental identifier |
| Tier |tier |`Front End, Back End, Data` |Tier or role/context identification |
| Data Profile |dataProfile |`Public, Confidential, Restricted, Internal` |Sensitivity of data stored in the resource |

## Tips and tricks
Some types of resources may require additional care on naming and conventions.

### Virtual machines
Especially in larger topologies, carefully naming virtual machines streamlines identifying the
role and purpose of each machine, and enabling more predictable scripting.

> [!WARNING]
> Every virtual machine in Azure has both an Azure resource name, and an operating
> system host name.  
> If the resource name and host name are different, managing the VMs may be challenging and should be avoided.
> For example, if a virtual machine is created from a .vhd that already contains a 
> configured operating system with a hostname.
> 
> 

* [Microsoft NetBIOS Computer Naming Conventions](https://support.microsoft.com/en-us/help/188997/microsoft-netbios-computer-naming-conventions)

### Storage accounts and storage entities
There are two primary use cases for storage accounts - backing disks for VMs, and storing 
data in blobs, queues and tables.  Storage accounts used for VM disks should follow the naming
convention of associating them with the parent VM name (and with the potential need for multiple 
storage accounts for high-end VM SKUs, also apply a number suffix).

> [!TIP]
> Storage accounts - whether for data or disks - should follow a naming convention that 
> allows for multiple storage accounts to be leveraged (i.e. always using a numeric suffix).
> 
> 

It possible to configure a custom domain name for accessing blob data in your Azure Storage account.
The default endpoint for the Blob service is `https://mystorage.blob.core.windows.net`.

But if you map a custom domain (such as www.contoso.com) to the blob endpoint for your storage account,
you can also access blob data in your storage account by using that domain. For example, with a custom
domain name, `http://mystorage.blob.core.windows.net/mycontainer/myblob` could be accessed as
`http://www.contoso.com/mycontainer/myblob`.

For more information about configuring this feature, refer to [Configure a custom domain name for your Blob storage endpoint](/azure/storage/storage-custom-domain-name/).

For more information on naming blobs, containers and tables:

* [Naming and Referencing Containers, Blobs, and Metadata](https://msdn.microsoft.com/library/dd135715.aspx)
* [Naming Queues and Metadata](https://msdn.microsoft.com/library/dd179349.aspx)
* [Naming Tables](https://msdn.microsoft.com/library/azure/dd179338.aspx)

A blob name can contain any combination of characters, but reserved URL characters must be properly
escaped. Avoid blob names that end with a period (.), a forward slash (/), or a sequence or combination
of the two. By convention, the forward slash is the **virtual** directory separator. Do not use a backward 
slash (\) in a blob name. The client APIs may allow it, but then fail to hash properly, and the 
signatures will not match.

It is not possible to modify the name of a storage account or container after it has been created.
If you want to use a new name, you must delete it and create a new one.

> [!TIP]
> We recommend that you establish a naming convention for all storage accounts and types
> before embarking on the development of a new service or application.
> 
> 
