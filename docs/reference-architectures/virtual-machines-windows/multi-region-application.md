---
title: Run Windows VMs in multiple Azure regions for high availability
description: >-
  How to deploy VMs in multiple regions on Azure for high availability and
  resiliency.

author: MikeWasson

ms.service: guidance
ms.topic: article
ms.date: 11/22/2016
ms.author: pnp

pnp.series.title: Windows VM workloads
pnp.series.prev: n-tier
---

# 고가용성을 위해 다지역에서 Windows VM 실행

이 참조 아키텍처는 가용성과 견고한 재난 복구 인프라를 확보하기 위해 여러 Aure 지역에 N계층 응용 프로그램을 실행하기 위한 일련의 검증된 사례를 보여줍니다. 

[![0]][0] 

## 아키텍처 

이 참조 아키텍처는 [N계층 응용 프로그램을 위한 Windows VM 실행](n-tier.md)에 소개된 아키텍처를 기반으로 하고 있습니다. 

* **주 지역 및 부 지역**. 고가용성을 위해 두 지역을 사용합니다. 하나는 주 지역이고 다른 하나는 장애 조치를 위한 지역입니다. 
* **Azure Traffic Manager**. [Traffic Manager][traffic-manager]는 들어오는 요청을 이 두 지역 중 하나에 라우팅합니다. 정상적으로 실행되는 경우 Traffic Manager는 요청을 주 지역으로 라우팅하는데, 주 지역이 사용 불가능한 경우에 부 지역으로 장애조치합니다. 자세한 내용은 [Traffic Manager 구성](#traffic-manager-configuration)을 참조하시기 바랍니다.
* **리소스 그룹**. 주 지역, 부 지역, Traffic Manager에 대한 별도의 [리소스 그룹][resource groups]을 생성합니다. 이를 통해 각 지역을 단일 리소스 모음으로 보다 유연하게 관리할 수 있습니다. 예를 들면 다른 지역의 실행을 중단하지 않고도 한 지역을 재배포할 수 있습니다. [리소스 그룹을 연결][resource-group-links]하여 해당 응용 프로그램에 대한 모든 리소스 목록을 생성하는 쿼리를 실행할 수 있습니다.
* **VNets**. 지역별로 별개의 VNet을 생성하는데, 이 때 주소 공간이 겹치지 않도록 주의해야 합니다. 
* **SQL 서버 Always On 가용성 그룹**. SQL 서버를 사용하는 경우에는 고가용성을 위해 [SQL Always On 가용성 그룹][sql-always-on]을 사용할 것을 권장합니다. 양 지역에 SQL 서버 인스턴스를 포함하는 단일 가용성 그룹을 만듭니다.  

    > [!참고]
    > 클라우드 서비스로서 관계형 DB를 제공하는 [Azure SQL Database][azure-sql-db]의 사용도 고려해 보시기 바랍니다. SQL Database를 사용하면 가용성 그룹을 구성하거나 장애조치를 관리할 필요가 없습니다.  
    > 

* **•	VPN 게이트웨이**. 각 VNet에 [VPN 게이트웨이][vpn-gateway]를 생성하고 [VNet-to-VNet 연결][vnet-to-vnet]을 구성하면 두 VNet 간 네트워크 트래픽을 허용하도록 설정할 수 있습니다. SQL Always On 가용성 그룹을 사용하려면 이러한 구성이 필요합니다.

You can download a [Visio file](https://aka.ms/arch-diagrams) of this architecture.

> [!참고]
> Azure는 [Resource Manager][resource-manager-overview]와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 문서에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다.
> 


## 권장사항

다지역 아키텍처는 단일 지역 배포보다 더 높은 가용성을 제공합니다. 주 지역이 지역 정전으로 영향을 받는 경우 [Traffic Manager][traffic-manager]를 사용하여 부 지역으로 장애조치할 수 있습니다. 이 아키텍처는 응용 프로그램의 개별 하위시스템 장애 발생 시에도 유용합니다. 

여러 지역에서 고가용성을 얻을 수 있는 몇 가지 일반적인 접근 방식이 있습니다. 

* 액티브/패시브 상시 대기. 트래픽이 한 지역으로 가는 동안 다른 지역은 상시 대기 모드가 됩니다. 상시 대기 모드란 부 지역의 VM이 할당되어 항상 실행되는 상태를 말합니다.
* 액티브/패시브 수동 대기. 트래픽이 한 지역으로 가는 동안 다른 지역은 수동 대기 모드가 됩니다. 수동 대기 모드란 장애조치를 위해 필요해지기 전까지 부 지역의 VM이 할당되지 않는 상태를 말합니다. 이 접근 방식은 비용을 절감할 수 있지만 장애 복구까지 더 오랜 시간이 소요됩니다.
* 액티브/액티브. 두 지역 모두 액티브 상태로서 요청 부하는 두 지역에 분산됩니다. 한 지역이 중단되면 로테이션에서 제외됩니다. 

이 참조 아키텍처는 장애조치를 위한 Traffic Manager를 사용하여 상시 대기 모드에서의 액티브/패시브에 초점을 맞추고 있습니다. 상시 대기 모드를 위해 일단 적은 수의 VM만 배포한 후 필요에 따라 확장할 수 있습니다. 

### 지역 연결

각 Azure 지역은 동일한 상위 지역 내에 위치한 다른 지역과 연결됩니다. 일반적으로 동일 지역 연결로부터 지역을 선택합니다. (예: East US 2 및 US Central.) 이렇게 함으로써 다음과 같은 이점을 얻을 수 있습니다.

* 넓은 지역에서 정전이 발생하는 경우 모든 연결 중 최소 한 지역에 대해서 우선적으로 복구를 수행합니다.
* 계획된 Azure 시스템 업데이트를 연결된 지역들에 순차적으로 발행하여 중단 시간을 최소화합니다.
* 한 쌍의 연결 지역은 동일 상위 지역 내에 위치해 데이터 상주 요구사항을 충족합니다. 

그러나 두 지역 모두 응용 프로그램에 필요한 모든 Azure 서비스를 지원해야 합니다. ([지역별 서비스][services-by-region] 참조). 지역 연결에 대한 자세한 내용은 [비즈니스 연속성 및 재난 복구(BCDR): Azure 연결 지역][regional-pairs]을 참조하시기 바랍니다.

### Traffic Manager 구성

Traffic Manager를 구성할 때는 다음과 같은 사항을 고려해야 합니다.

* **라우팅**. Traffic Manager는 몇 가지 [라우팅 알고리즘][tm-routing]을 지원합니다. 이 문서에서 설명하는 시나리오의 경우에는 (과거 *장애조치* 라우팅으로 불린) *우선순위* 라우팅을 사용합니다. 우선순위 라우팅 설정 시 Traffic Manager는 주 지역이 통신 불가능한 경우가 아니라면 모든 요청을 주 지역으로 전송합니다. 만약 주 지역이 통신 불가능한 상태라면 Traffic Manager가 부 지역으로 자동 장애조치를 수행합니다. [장애조치 라우팅 메서드 구성][tm-configure-failover]을 참조하시기 바랍니다.
* **상태 프로브**. Traffic Manager는 HTTP(또는 HTTPS) [프로브][tm-monitoring]를 사용하여 각 지역의 가용성을 모니터링합니다. 이 프로브는 특정 URL 경로에 대한 HTTP 200 응답을 확인합니다. 모범 사례 중 하나는 응용 프로그램의 전반적인 상태를 보고하는 끝점을 생성하여 상태 프로브를 위해 사용하는 것입니다. 그렇지 않으면 이 프로브는 응용 프로그램의 중요한 부분이 실제로는 고장난 경우에도 끝점의 상태를 양호하다고 보고할 수 있습니다. 자세한 내용은 [상태 끝점 모니터링 패턴][health-endpoint-monitoring-pattern]을 참조하시기 바랍니다.   

Traffic Manager가 장애 조치를 수행하는 동안 클라이언트가 해당 응용 프로그램에 접속할 수 없는 시간이 발생하는데, 이 시간은 다음과 같은 요소에 따라 달라집니다. 

* 상태 프로브는 주 지역이 통신할 수 없는 상태인 경우 이를 감지해야 합니다.
* DNS 서버는 해당 IP 주소에 대한 캐시된 DNS 기록을 업데이트해야 하는데, 이는 DNS TTL(time-to-live)의 영향을 받습니다. TTL은 300초(5분)로 기본 설정되어 있지만 Traffic Manager 프로필을 생성할 때 이 값을 구성할 수 있습니다.

자세한 내용은 [Traffic Manager 모니터링][tm-monitoring]을 참조하시기 바랍니다.

Traffic Manager가 장애조치를 실행하는 경우 자동 장애 복구가 아닌 수동 장애 복구 실행을 권장합니다. 그렇지 않으면 해당 응용 프로그램이 여러 다른 지역을 왔다 갔다 하는 상황이 발생할 수 있습니다. 장애 복구를 실시하기 전 모든 응용 프로그램 하위 시스템이 정상적인 상태인지 확인합니다.

Traffic Manager는 자동 장애 복구로 기본 설정되어 있습니다. 이를 방지하려면 장애 복구 이벤트 발생 후 주 지역의 우선순위를 수동으로 낮춰야 합니다. 예를 들어, 주 지역이 우선순위 1이고 부 지역이 우선순위 2인 경우, 장애 조치 후, 주 지역을 우선순위 3으로 설정하여 자동 장애 복구를 방지할 수 있습니다. 원상 복구할 준비가 완료되면 그 때 우선순위를 다시 1로 업데이트하면 됩니다.

우선순위는 다음과 같은 [Azure CLI][install-azure-cli] 명령어를 사용하여 업데이트할 수 있습니다.

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

또 다른 접근 방식은 장애 복구를 실시할 준비가 완료될 때까지 해당 끝점을 일시적으로 사용하지 않도록 설정하는 것입니다. 

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```

장애 조치의 원인에 따라 한 지역 내 리소스들을 재배포해야 할 수도 있습니다. 따라서 장애 복구 전에 실행 준비 테스트를 실시하는데, 이 테스트를 통해 다음과 같은 사항을 확인해야 합니다.

* VM이 올바르게 구성되었는지 여부. (모든 필요한 소프트웨어 설치 여부, IIS 실행 여부 등)
* 응용 프로그램 하위 시스템이 정상 상태인지 여부 
* 기능 테스트. (예를 들어, 웹 계층에서 DB 계층으로의 접근 가능 여부.)

### SQL 서버 Always On 구성

Windows Server 2016에 앞서 SQL 서버 Always On 가용성 그룹에는 도메인 컨트롤러가 필요하고 가용성 그룹 내 모든 노드는 동일한 Active Directory(AD) 도메인 내에 있어야 합니다.

가용성 그룹을 구성하는 방법은 다음과 같습니다.

* 각 지역에 최소 두 개의 도메인 컨트롤러를 배치합니다.
* 각 도메인 컨트롤러에 정적 IP 주소를 할당합니다.
* VNet-to-VNet 연결을 만들어 VNet 간 통신이 가능하도록 설정합니다.
* 각 VNet에 대해 (두 지역의) 도메인 컨트롤러의 IP 주소를 DNS 서버 목록에 추가합니다. 다음과 같은 CLI 명령어를 사용할 수 있습니다. 자세한 내용은 [가상 네트워크(VNet)가 사용하는 DNS 서버 관리][vnet-dns]를 참조하시기 바랍니다.

    ```bat
    azure network vnet set --resource-group dc01-rg --name dc01-vnet --dns-servers "10.0.0.4,10.0.0.6,172.16.0.4,172.16.0.6"
    ```

* 두 지역에 SQL 서버 인스턴스를 포함하는 [Windows Server Failover Clustering][wsfc] (WSFC) 클러스터를 생성합니다. 
* 주 지역과 부 지역 모두에 SQL 서버 인스턴스를 포함하는 SQL 서버 Always On 가용성 그룹을 생성합니다. 생성 단계에 관한 내용은 [Always On 가용성 그룹을 원격 Azure Datacenter (PowerShell)로 확장](https://blogs.msdn.microsoft.com/sqlcat/2014/09/22/extending-alwayson-availability-group-to-remote-azure-datacenter-powershell/)을 참조하시기 바랍니다.

    * 주 복제본을 주 지역에 배치합니다.
    * 하나 이상의 부 복제본을 주 지역에 배치합니다. 자동 장애조치 동기 커밋을 사용하도록 구성합니다.
    * 하나 이상의 부 복제본을 부 지역에 배치합니다. 성능 관련 이유로 *비동기* 커밋을 사용하도록 구성합니다. (그렇지 않으면, 모든 SQL 트랜잭션이 부 지역까지 네트워크 왕복 여행 동안 대기해야 합니다.)

    > [!참고]
    > 비동기 커밋 복제본은 자동 장애 조치를 지원하지 않습니다.
    >
    >

[Azure에서 N계층 아키텍처를 위한 Windows VM 실행](n-tier.md)을 참조하시기 바랍니다.

## 가용성 고려사항

복잡한 N계층 앱의 경우 부 지역에 응용 프로그램을 통째로 복제해야 할 수도 있습니다. 반대로 비즈니스 연속성 지원에 필요한 중요한 하위 시스템만을 복제하는 방법도 있습니다. 

이러한 시스템에서는 Traffic Manager가 장애 지점이 됩니다. Traffic Manager 서비스 장애 시, 클라이언트는 중단 시간 동안 응용 프로그램에 액세스할 수 없습니다. [Traffic Manager SLA][tm-sla]를 검토하여 Traffic Manager를 단독으로 사용하는 것이 고가용성을 위한 비즈니스 요구사항을 충족하는지 확인하시기 바랍니다. 만일 요구사항이 충족되지 않는다면, 또 다른 트래픽 관리 솔루션을 장애 복구용으로 추가할 것을 고려해 보아야 합니다. Azure Traffic Manager 서비스 장애 시, DNS의 CNAME 기록을 변경하여 다른 트래픽 관리 서비스를 지정하도록 설정합니다. (이 단계는 수동으로 수행해야 하는데, DNS 변경이 전파될 때까지는 응용 프로그램을 이용할 수 없습니다.)

SQL 서버 클러스터의 경우 고려해야 할 두 가지 장애 조치 시나리오가 있습니다.

- 주 지역의 모든 SQL 복제본이 장애 상태인 경우. 예를 들어, 이러한 경우는 지역 정전 시 발생할 수 있습니다. 이 경우에는 Traffic Manager가 프런트엔드에서 자동으로 장애조치를 실행하더라도 SQL 가용성 그룹을 수동으로 장애 조치해야 합니다. [SQL 서버 가용성 그룹의 강제 수동 장애조치 실행](https://msdn.microsoft.com/library/ff877957.aspx)에 소개된 단계를 따르시기 바랍니다. 여기에는 SQL Server Management Studio, Transact-SQL 또는 SQL Server 2016 PowerShell을 사용하여 강제 장애조치를 실행하는 방법이 설명되어 있습니다.

   > [!경고]
   > 강제 장애 조치 시 데이터 손실의 위험이 있습니다. 주 지역의 연결이 복구되면 DB의 스냅숏을 찍고 [tablediff]를 사용하여 차이점을 확인하세요.
   >
   >
- Traffic Manager가 부 지역으로 장애 조치를 수행하더라도 주 SQL 복제본은 계속해서 이용할 수 있습니다. 예를 들어, 프런트엔드 계층에 장애가 발생해도 SQL VM은 영향을 받지 않습니다. 이 경우, 내부 트래픽은 부 지역으로 라우팅되고, 해당 지역은 계속해서 주 SQL 복제본에 연결됩니다. 그러나 SQL 연결이 여러 지역에 걸쳐 이루어지므로 대기 시간이 증가할 수 있습니다. 이러한 경우에는 다음과 같이 수동으로 장애 조치를 수행해야 합니다.

   1. 부 지역의 SQL 복제본을 일시적으로 *동기* 커밋으로 변경합니다. 이를 통해 장애 조치를 실시하는 동안 데이터 손실을 방지할 수 있습니다.
   2. 해당 SQL 복제본으로 장애 조치를 실시합니다.
   3. 주 지역으로 장애 복구 시 다시 비동기 커밋으로 설정을 변경합니다.

## 관리 효율성 고려사항

배포 업데이트 시 한번에 하나의 지역씩 업데이트함으로써 응용 프로그램의 부정확한 구성 또는 오류에 의한 글로벌 장애의 위험을 완화합니다. 

시스템의 장애 복원력을 테스트해야 하는데, 일반적인 테스트 대상 장애 시나리오는 다음과 같습니다.

* VM 인스턴스를 종료합니다.
* CPU나 메모리와 같은 리소스의 부하를 증가시킵니다.
* 네트워크를 차단하거나 지연시킵니다.
* 프로세스를 충돌시킵니다.
* 인증서의 유효기간이 만료되도록 합니다.
* 하드웨어 고장을 시뮬레이션합니다.
* 도메인 컨트롤러의 DNS 서비스를 종료합니다.

복구 시간을 측정하여 비즈니스 요구사항을 충족하는지 확인하고 장애 모드의 조합도 테스트해야 합니다.



<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md

[azure-sla]: https://azure.microsoft.com/support/legal/sla/
[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-always-on]: https://msdn.microsoft.com/library/hh510230.aspx
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx
[vnet-dns]: /azure/virtual-network/virtual-networks-manage-dns-in-vnet
[vnet-to-vnet]: /azure/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps
[vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx

[0]: ./images/multi-region-application-diagram.png "Highly available network architecture for Azure N-tier applications"
