---
title: Create an AD DS resource forest in Azure
description: How to create a trusted Active Directory domain in Azure.

  guidance,vpn-gateway,expressroute,load-balancer,virtual-network,active-directory

author: telmosampaio



pnp.series.title: Identity management

ms.service: guidance

ms.topic: article


ms.date: 11/28/2016
ms.author: pnp
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
---
# Azure에 Active Directory 도메인 서비스(AD DS) 리소스 포레스트 생성
[!INCLUDE [header](../../_includes/header.md)]

이 문서는 온-프레미스 포레스트로부터 분리되었지만 온-프레미스 포레스트에 있는 도메인의 신뢰(trust)를 받는 Azure의 Active Directory 도메인을 생성하는 방법을 설명합니다.

> [!참고]
> Azure는 [Resource Manager][resource-manager-overview]와 클래식 모델의 두 가지 배포 모델을 지원합니다. 이 참조 아키텍처에서는 Microsoft가 새 배포를 위해 권장하는 Resource Manager를 사용합니다.
> 
> 

Active Directory 도메인 서비스(AD DS)는 사용자, 장치 및 기타 리소스에 관한 계정 정보를 계층 구조로 저장하는 분산형 데이터베이스 서비스입니다. 계층 구조의 최상위 노드를 포레스트(forest)라고 합니다. 포레스트는 도메인을, 도메인은 다른 유형의 개체를 담고 있습니다.

AD DS를 사용하여 최상위 포레스트 개체 간 트러스트 관계를 형성하여 도메인 간 상호운용성을 제공할 수 있습니다. 즉, 하나의 도메인에서 로그인을 하면 트러스트 관계를 바탕으로 다른 도메인의 리소스에 대한 접근 권한도 제공됩니다.

이 참조 아키텍처는 Azure에 AD DS 포레스트를 생성하고 온-프레미스 도메인과의 일방향 트러스트 관계를 구축하는 방법에 대해 보여줍니다.  Azure에 있는 포레스트는 온-프레미스에 없는 도메인을 포함하지만 트러스트 관계를 바탕으로 온-프레미스 도메인에서 한 로그인을 통해 별개의 Azure 도메인에 있는 리소스에 접속할 수 있습니다.

이 아키텍처의 일반적인 용도는 클라우드에 있는 개체와 ID에 대한 보안 분리, 온-프레미스에서 클라우드로의 개별 도메인 마이그레이션 등이 있습니다.

## 아키텍처 다이어그램

다음 다이어그램은 이 아키텍처의 중요한 요소들을 보여줍니다.  

> [Microsoft 다운로드 센터][visio-download]에서 이 아키텍처 다이어그램을 포함한 Visio 문서를 다운로드할 수 있습니다. 이 다이어그램은 "계정 - AADS(리소스 포레스트)" 페이지에서 확인할 수 있습니다.
> 
> 

[![0]][0] 

* **온-프레미스 네트워크**. 온-프레미스 네트워크는 자체 Active Directory 포레스트와 도메인을 포함합니다.
* **Active Directory 서버**. 클라우드에서 VM으로 실행되는 도메인 서비스를 구현하는 도메인 컨트롤러입니다. 이 서버들은 온-프레미스에 위치한 도메인들과 분리된 하나 이상의 도메인을 포함하는 포레스트를 호스팅합니다.
* **일방향 트러스트 관계**. 이 다이어그램 속의 예는 Azure 도메인으로부터 온-프레미스 도메인으로의 일방향 트러스트 관계를 보여줍니다.  이 관계를 통해 온-프레미스 사용자는 Azure의 도메인에 있는 리소스에 접근할 수 있지만 Azure 사용자가 온-프레미스의 리소스에 접근할 수는 없습니다. 클라우드 사용자가 온-프레미스 리소스 접근을 요구하는 경우 양방향 트러스트 관계를 생성할 수도 있습니다.
* **Active Directory 서브넷**. AD DS 서버는 별도의 서브넷에 호스팅됩니다. 네트워크 보안 그룹(NSG) 규칙은 AD DS 서버를 보호하고 예상치 못한 소스로부터의 트래픽에 대한 방화벽을 제공합니다.
* **Azure 게이트웨이**. Azure 게이트웨이는 온-프레미스 네트워크와 Azure VNet 간 연결을 제공합니다.[VPN 연결][azure-vpn-gateway] 또는 [Azure ExpressRoute][azure-expressroute]가 사용됩니다. 자세한 내용은 [Azure에 안전한 하이브리드 네트워크 아키텍처 구현][implementing-a-secure-hybrid-network-architecture]을 참조하세요.

## 추천

Azure에서의 Active Directory 구현에 관한 구체적인 권장사항은 다음 문서를 참조하세요. 

- [Active Directory Domain Services (AD DS)를 Azure로 확장][adds-extend-domain]. 
- [Azure 가상 컴퓨터에서의 Windows Server Active Directory 배포를 위한 가이드라인][ad-azure-guidelines].

### Trust

온-프레미스 도메인은 클라우드에 있는 도메인과는 다른 포레스트 내에 있습니다. 클라우드에서 온-프레미스 사용자 인증을 하려면 Azure 도메인이 온-프레미스 포레스트의 로그인 도메인을 신뢰해야 합니다. 마찬가지로, 클라우드가 외부 사용자를 위한 로그인 도메인을 제공한다면 온-프레미스 포레스트가 클라우드 도메인을 신뢰해야 합니다.

포레스트 수준에서는 [포레스트 트러스트 생성][creating-forest-trusts]을 통해, 도메인 수준에서는 [외부 트러스트 생성][creating-external-trusts]을 통해 트러스트를 구축할 수 있습니다. 포레스트 수준 트러스트는 두 개의 포레스트 내에 있는 모든 도메인들 사이에 관계를 형성합니다. 외부 도메인 수준 트러스트는 두 개의 지정된 도메인 간 관계만을 형성합니다. 서로 다른 포레스트의 도메인 간에는 외부 도메인 수준 트러스트만 형성해야 합니다.

트러스트는 일방향 또는 양방향으로 생성됩니다.

* 일방향 트러스트는 하나의 (*인바운드*) 도메인 또는 포레스트의 사용자가 다른 (*아웃바운드*) 도메인 또는 포레스트에 있는 리소스에 접근할 수 있도록 해줍니다.
* 양방향 트러스트는 어느 한 쪽의 도메인 또는 포레스트에 있는 사용자가 다른 쪽 도메인 또는 포레스트에 있는 리소스에 접근할 수 있도록 합니다.

다음 표에는 간단한 시나리오에 대한 트러스트 구성이 요약되어 있습니다. 

| 시나리오 | 온-프레미스 트러스트 | 클라우드 트러스트 |
| --- | --- | --- |
| 온-프레미스 사용자는 클라우드에 있는 리소스에 대한 액세스를 요구할 수 있지만 그 반대 방향은 불가능합니다. |일방향(인바운드) |일방향(아웃바운드) |
| 클라우드 사용자는 온-프레미스에 위치한 리소스에 대한 액세스를 요구할 수 있지만 그 반대 방향은 불가능합니다. |일방향(아웃바운드) |일방향(인바운드) |
| 클라우드와 온-프레미스 사용자 모두 클라우드와 온-프레미스에 있는 리소스에 대한 액세스를 요구할 수 있습니다. |양방향(인바운드/아웃바운드) |양방향(인바운드/아웃바운드) |

## 확장성 고려사항

Active Directory는 동일한 도메인의 일부인 도메인 컨트롤러를 위해 자동으로 확장 가능합니다. 요청은 하나의 도메인 내에 있는 모든 컨트롤러들로 분산됩니다. 도메인 컨트롤러를 추가하면 자동으로 해당 도메인과 동기화가 진행됩니다. 해당 도메인 내에서 트래픽을 컨트롤러로 디렉팅하기 위해 별도의 부하 분산 장치를 구성하지 마세요. 모든 도메인 컨트롤러는 도메인 DB를 처리하기에 충부한 메모리와 저장소 리소스를 가져야 합니다. 모든 도메인 컨트롤러 VM은 동일한 크기여야 합니다.

## 가용성 고려사항

각 도메인에 대해 최소 두 개의 도메인 컨트롤러를 프로비전합니다.  이를 통해 서버 간 자동 복제가 가능해집니다. 각 도메인을 처리하는 Active Directory 서버의 역할을 하는 VM에 대한 가용성 집합을 생성합니다. 이 가용성 집합 내에 최소 두 개의 서버를 배치합니다. 

또한 유연한 단일 마스터 작업(FSMO) 역할을 수행하는 서버에 대한 연결 실패를 대비하여 각 도메인에 있는 하나 이상의 서버를 [대기 작업 마스터][standby-operations-masters]로 지정하는 것을 고려해 보세요.

## 관리 효율성 고려사항

관리 및 모니터링 고려사항에 대한 자세한 내용은 [Active Directory를 Azure로 확장][adds-extend-domain]을 참조하세요. 
 
추가 정보는 [Active Directory 모니터링][monitoring_ad]을 참조하세요. 관리 서브넷의 모니터링 서버에 [Microsoft Systems Center][microsoft_systems_center]와 같은 도구를 설치하며 이러한 작업 수행이 수월해질 수 있습니다.

## 보안 고려사항

포레스트 수준 트러스트는 전이적(transitive)입니다. 온-프레미스 포레스트와 클라우드 포레스트 간 포레스트 수준 트러스트를 구축하는 경우 이 트러스트는 어느 하나의 포레스트에 생성된 다른 새로운 도메인들로 확장됩니다. 보안을 목적으로 도메인을 사용하여 분리를 제공하는 경우에는 도메인 수준에서만 트러스트를 생성하는 것을 고려하세요. 도메인 수준 트러스트는 비전이적(non-transitive)입니다. 

Active Directory별 보안 고려사항을 확인하려면 [Active Directory를 Azure로 확장][adds-extend-domain]의 보안 고려사항 섹션을 참조하세요.

## 솔루션 배포
[Github][github]에서 이 참조 아키텍처를 배포할 수 있는 솔루션을 이용할 수 있습니다. 이 솔루션을 배포하는 PowerShell 스크립트를 실행하려면 최신 버전의 Azure CLI가 필요합니다. 이 참조 아키텍처를 배포하려면 아래의 단계들을 수행하십시오.

1. [Github][github]의 해당 솔루션 폴더를 로컬 컴퓨터로 다운로드하거나 복제합니다. 

2. Azure CLI을 실행하여 로컬 솔루션 폴더로 갑니다.

3. 다음 명령을 실행합니다.
   
    ```Powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    Replace `<subscription id>` with your Azure subscription ID.
   
    For `<location>`, specify an Azure region, such as `eastus` or `westus`.
   
    The `<mode>` parameter controls the granularity of the deployment, and can be one of the following values:
   
   * `Onpremise`: deploys the simulated on-premises environment.
   * `Infrastructure`: deploys the VNet infrastructure and jump box in Azure.
   * `CreateVpn`: deploys the Azure virtual network gateway and connects it to the simulated on-premises network.
   * `AzureADDS`: deploys the VMs acting as Active Directory DS servers, deploys Active Directory to these VMs, and deploys the domain in Azure.
   * `WebTier`: deploys the web tier VMs and load balancer.
   * `Prepare`: deploys all of the preceding deployments. **This is the recommended option if If you do not have an existing on-premises network but you want to deploy the complete reference architecture described above for testing or evaluation.** 
   * `Workload`: deploys the business and data tier VMs and load balancers. Note that these VMs are not included in the `Prepare` deployment.

4. Wait for the deployment to complete. If you are deploying the `Prepare` deployment, it will take several hours.
     
5. If you are using the simulated on-premises configuration, configure the incoming trust relationship:
   
   1. Connect to the jump box (*ra-adtrust-mgmt-vm1* in the *ra-adtrust-security-rg* resource group). Log in as *testuser* with password *AweS0me@PW*.
   2. On the jump box open an RDP session on the first VM in the *contoso.com* domain (the on-premises domain). This VM has the IP address 192.168.0.4. The username is *contoso\testuser* with password *AweS0me@PW*.
   3. Download the [incoming-trust.ps1][incoming-trust] script and run it to create the incoming trust from the *treyresearch.com* domain.

6. If you are using your own on-premises infrastructure:
   
   1. Download the [incoming-trust.ps1][incoming-trust] script.
   2. Edit the script and replace the value of the `$TrustedDomainName` variable with the name of your own domain.
   3. Run the script.

7. From the jump-box, connect to the first VM in the *treyresearch.com* domain (the domain in the cloud). This VM has the IP address 10.0.4.4. The username is *treyresearch\testuser* with password *AweS0me@PW*.

8. Download the [outgoing-trust.ps1][outgoing-trust] script and run it to create the incoming trust from the *treyresearch.com* domain. If you are using your own on-premises machines, then edit the script first. Set the `$TrustedDomainName` variable to the name of your on-premises domain, and specify the IP addresses of the Active Directory DS servers for this domain in the `$TrustedDomainDnsIpAddresses` variable.

9. Wait a few minutes for the previous steps to complete, then connect to an on-premises VM and perform the steps outlined in the article [Verify a Trust][verify-a-trust] to determine whether the trust relationship between the *contoso.com* and *treyresearch.com* domains is correctly configured.

## 다음 단계

* [온-프레미스 AD DS 도메인을 Azure로 확장][adds-extend-domain]하기 위한 모범 사례를 살펴보세요.
* Azure에서 [AD FS 인프라 생성][adfs]하기 위한 모범 사례를 살펴보세요.

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[visio-download]: http://download.microsoft.com/download/1/5/6/1569703C-0A82-4A9C-8334-F13D0DF2F472/RAs.vsdx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[0]: ../_images/guidance-identity-aad-resource-forest/figure1.png "Secure hybrid network architecture with separate Active Directory domains"
