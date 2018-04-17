---
title: Azure에서 Jenkins 서버 실행
description: 이 참조 아키텍처는 SSO(Single Sign-On)로 보호된 Azure에서 확장성 있는 엔터프라이즈급 Jenkins 서버를 배포하고 작동하는 방법을 보여 줍니다.
author: njray
ms.date: 01/21/18
ms.openlocfilehash: c07a341bbe4d0304087e4535408967c45d36199e
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="run-a-jenkins-server-on-azure"></a><span data-ttu-id="ab90b-103">Azure에서 Jenkins 서버 실행</span><span class="sxs-lookup"><span data-stu-id="ab90b-103">Run a Jenkins server on Azure</span></span>

<span data-ttu-id="ab90b-104">이 참조 아키텍처는 SSO(Single Sign-On)로 보호된 Azure에서 확장성 있는 엔터프라이즈급 Jenkins 서버를 배포하고 작동하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-104">This reference architecture shows how to deploy and operate a scalable, enterprise-grade Jenkins server on Azure secured with single sign-on (SSO).</span></span> <span data-ttu-id="ab90b-105">이 아키텍처는 또한 Jenkins 서버의 상태를 모니터링하는 데 Azure Monitor를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-105">The architecture also uses Azure Monitor to monitor the state of the Jenkins server.</span></span> [<span data-ttu-id="ab90b-106">**이 솔루션을 배포합니다**.</span><span class="sxs-lookup"><span data-stu-id="ab90b-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

![Azure에서 실행 중인 Jenkins 서버][0]

<span data-ttu-id="ab90b-108">*이 아키텍처 다이어그램이 포함된 [Visio 파일](https://archcenter.blob.core.windows.net/cdn/Jenkins-architecture.vsdx)을 다운로드합니다.*</span><span class="sxs-lookup"><span data-stu-id="ab90b-108">*Download a [Visio file](https://archcenter.blob.core.windows.net/cdn/Jenkins-architecture.vsdx) that contains this architecture diagram.*</span></span>

<span data-ttu-id="ab90b-109">이 아키텍처는 Azure 서비스를 통한 재해 복구를 지원하지만 가동 중지 시간이 없는 다중 마스터 또는 HA(고가용성)과 관련된 보다 고급의 스케일 아웃 시나리오는 다루지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-109">This architecture supports disaster recovery with Azure services but does not cover more advanced scale-out scenarios involving multiple masters or high availability (HA) with no downtime.</span></span> <span data-ttu-id="ab90b-110">Azure에서 CI/CD 파이프라인을 빌드하는 방법에 대한 단계별 자습서를 비롯하여 다양한 Azure 구성 요소에 대한 일반적인 정보는 [Azure의 Jenkins][jenkins-on-azure]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-110">For general insights about the various Azure components, including a step-by-step tutorial about building out a CI/CD pipeline on Azure, see [Jenkins on  Azure][jenkins-on-azure].</span></span>

<span data-ttu-id="ab90b-111">이 문서에서는 Azure Storage를 사용하여 빌드 아티팩트, SSO에 필요한 보안 항목, 통합 가능한 기타 서비스 및 파이프라인에 대한 확장성을 유지하는 등 Jenkins를 지원하는 데 필요한 핵심 Azure 작업에 중점을 둡니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-111">The focus of this document is on the core Azure operations needed to support Jenkins, including the use of Azure Storage to maintain build artifacts, the security items needed for SSO, other services that can be integrated, and scalability for the pipeline.</span></span> <span data-ttu-id="ab90b-112">이 아키텍처는 기존 소스 제어 리포지토리에서 작동하도록 설계되었습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-112">The architecture is designed to work with an existing source control repository.</span></span> <span data-ttu-id="ab90b-113">예를 들어, 일반적인 시나리오는 GitHub 커밋에 따라 Jenkins 작업을 시작하는 것입니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-113">For example, a common scenario is to start Jenkins jobs based on GitHub commits.</span></span>

## <a name="architecture"></a><span data-ttu-id="ab90b-114">아키텍처</span><span class="sxs-lookup"><span data-stu-id="ab90b-114">Architecture</span></span>

<span data-ttu-id="ab90b-115">이 아키텍처는 다음과 같은 구성 요소로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-115">The architecture consists of the following components:</span></span>

- <span data-ttu-id="ab90b-116">**리소스 그룹.**</span><span class="sxs-lookup"><span data-stu-id="ab90b-116">**Resource group.**</span></span> <span data-ttu-id="ab90b-117">[리소스 그룹][rg]은 Azure 자산을 수명, 소유자를 비롯한 기준으로 관리할 수 있도록 Azure 자산을 그룹화하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-117">A [resource group][rg] is used to group Azure assets so they can be managed by lifetime, owner, and other criteria.</span></span> <span data-ttu-id="ab90b-118">리소스 그룹을 사용하여 Azure 자산을 그룹 단위로 배포 및 모니터링하고, 리소스 그룹별로 청구 비용을 추적할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-118">Use resource groups to deploy and monitor Azure assets as a group and track billing costs by resource group.</span></span> <span data-ttu-id="ab90b-119">리소스를 하나의 집합으로 삭제할 수도 있습니다. 이러한 기능은 테스트 배포에서 매우 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-119">You can also delete resources as a set, which is very useful for test deployments.</span></span>

- <span data-ttu-id="ab90b-120">**Jenkins 서버**.</span><span class="sxs-lookup"><span data-stu-id="ab90b-120">**Jenkins server**.</span></span> <span data-ttu-id="ab90b-121">가상 머신은 [Jenkins][azure-market]를 자동화 서버로 실행하고 Jenkins 마스터 역할을 수행하도록 배포됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-121">A virtual machine is deployed to run [Jenkins][azure-market] as an automation server and serve as Jenkins Master.</span></span> <span data-ttu-id="ab90b-122">이 참조 아키텍처는 Azure의 Linux(Ubuntu 16.04 LTS) 가상 머신에 설치된 [Azure의 Jenkins용 솔루션 템플릿][solution]을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-122">This reference architecture uses the [solution template for Jenkins on Azure][solution], installed on a Linux (Ubuntu 16.04 LTS) virtual machine on Azure.</span></span> <span data-ttu-id="ab90b-123">다른 Jenkins 제품은 Azure Marketplace에서 제공됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-123">Other Jenkins offerings are available in the Azure Marketplace.</span></span>

  > [!NOTE]
  > <span data-ttu-id="ab90b-124">Nginx는 VM에 설치되어 Jenkins에 대한 역방향 프록시 역할을 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-124">Nginx is installed on the VM to act as a reverse proxy to Jenkins.</span></span> <span data-ttu-id="ab90b-125">Jenkins 서버에 SSL을 사용하도록 Nginx를 구성할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-125">You can configure Nginx to enable SSL for the Jenkins server.</span></span>
  > 
  > 

- <span data-ttu-id="ab90b-126">**가상 네트워크**.</span><span class="sxs-lookup"><span data-stu-id="ab90b-126">**Virtual network**.</span></span> <span data-ttu-id="ab90b-127">[가상 네트워크][vnet]는 Azure 리소스를 서로 연결하고 논리적 격리를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-127">A [virtual network][vnet] connects Azure resources to each other and provides logical isolation.</span></span> <span data-ttu-id="ab90b-128">이 아키텍처에서 Jenkins 서버는 가상 네트워크에서 실행됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-128">In this architecture, the Jenkins server runs in a virtual network.</span></span>

- <span data-ttu-id="ab90b-129">**서브넷**.</span><span class="sxs-lookup"><span data-stu-id="ab90b-129">**Subnets**.</span></span> <span data-ttu-id="ab90b-130">Jenkins 서버는 성능에 영향을 미치지 않으면서 네트워크 트래픽을 보다 쉽게 관리하고 분리할 수 있도록 [서브넷][subnet]에 격리됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-130">The Jenkins server is isolated in a [subnet][subnet] to make it easier to manage and segregate network traffic without impacting performance.</span></span>

- <span data-ttu-id="ab90b-131"><strong>NSG</strong>.</span><span class="sxs-lookup"><span data-stu-id="ab90b-131"><strong>NSGs</strong>.</span></span> <span data-ttu-id="ab90b-132">NSG([네트워크 보안 그룹][nsg])를 사용하여 네트워크 트래픽을 인터넷에서 가상 네트워크의 서브넷으로 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-132">Use [network security groups][nsg] (NSGs) to restrict network traffic from the Internet to the subnet of a virtual network.</span></span>

- <span data-ttu-id="ab90b-133">**관리 디스크**.</span><span class="sxs-lookup"><span data-stu-id="ab90b-133">**Managed disks**.</span></span> <span data-ttu-id="ab90b-134">[관리 디스크][managed-disk]는 응용 프로그램 저장소 및 Jenkins 서버의 상태를 유지하고 재해 복구를 제공하는 데 사용된 영구 VHD(가상 하드 디스크)입니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-134">A [managed disk][managed-disk] is a persistent virtual hard disk (VHD) used for application storage and also to maintain the state of the Jenkins server and provide disaster recovery.</span></span> <span data-ttu-id="ab90b-135">데이터 디스크는 Azure Storage에 저장됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-135">Data disks are stored in Azure Storage.</span></span> <span data-ttu-id="ab90b-136">고성능을 위해 [프리미엄 저장소][premium]를 사용하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-136">For high performance, [premium storage][premium] is recommended.</span></span>

- <span data-ttu-id="ab90b-137">**Azure Blob Storage**.</span><span class="sxs-lookup"><span data-stu-id="ab90b-137">**Azure Blob Storage**.</span></span> <span data-ttu-id="ab90b-138">[Windows Azure Storage 플러그 인][configure-storage]은 Azure Blob Storage를 사용하여 다른 Jenkins 빌드로 생성 및 공유되는 빌드 아티팩트를 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-138">The [Windows Azure Storage plugin][configure-storage] uses Azure Blob  Storage to store the build artifacts that are created and shared with other Jenkins builds.</span></span>

- <span data-ttu-id="ab90b-139"><strong>Azure AD(Azure Active Directory)</strong>.</span><span class="sxs-lookup"><span data-stu-id="ab90b-139"><strong>Azure Active Directory (Azure AD)</strong>.</span></span> <span data-ttu-id="ab90b-140">[Azure AD][azure-ad]는 SSO를 설정할 수 있도록 사용자 인증을 지원합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-140">[Azure AD][azure-ad] supports user authentication, allowing you to set up SSO.</span></span> <span data-ttu-id="ab90b-141">Azure AD [서비스 주체][service-principal]는 RBAC([역할 기반 액세스 제어][rbac])를 사용하여 워크플로에서 각 역할 권한 부여에 대한 정책 및 사용 권한을 정의합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-141">Azure AD [service principals][service-principal] define the policy and permissions for each role authorization in the workflow, using [role-based access control][rbac] (RBAC).</span></span> <span data-ttu-id="ab90b-142">각 서비스 주체는 Jenkins 작업과 연결됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-142">Each service principal is associated with a Jenkins job.</span></span>

- <span data-ttu-id="ab90b-143">**Azure Key Vault.**</span><span class="sxs-lookup"><span data-stu-id="ab90b-143">**Azure Key Vault.**</span></span> <span data-ttu-id="ab90b-144">비밀이 요구될 때 Azure 자원을 프로비전하는 데 사용되는 비밀 및 암호화 키를 관리하기 위해 이 아키텍처는 [Key Vault][key-vault]를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-144">To manage secrets and cryptographic keys used to provision Azure resources when secrets are required, this architecture uses [Key Vault][key-vault].</span></span> <span data-ttu-id="ab90b-145">파이프라인에 있는 응용 프로그램과 관련된 비밀을 저장하는 추가 도움말은 Jenkins의 [Azure 자격 증명][configure-credential] 플러그 인을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-145">For added help storing secrets associated with the application in the pipeline, see also the [Azure Credentials][configure-credential] plugin for Jenkins.</span></span>

- <span data-ttu-id="ab90b-146">**Azure 모니터링 서비스**.</span><span class="sxs-lookup"><span data-stu-id="ab90b-146">**Azure monitoring services**.</span></span> <span data-ttu-id="ab90b-147">이 서비스는 Jenkins를 호스팅하는 Azure 가상 머신을 [모니터링][monitor]합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-147">This service [monitors][monitor] the Azure virtual machine hosting Jenkins.</span></span> <span data-ttu-id="ab90b-148">이 배포는 가상 머신 상태 및 CPU 사용률을 모니터링하고 경고를 보냅니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-148">This deployment monitors the virtual machine status and CPU utilization and sends alerts.</span></span>

## <a name="recommendations"></a><span data-ttu-id="ab90b-149">권장 사항</span><span class="sxs-lookup"><span data-stu-id="ab90b-149">Recommendations</span></span>

<span data-ttu-id="ab90b-150">대부분의 시나리오의 경우 다음 권장 사항을 적용합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="ab90b-151">이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="azure-ad"></a><span data-ttu-id="ab90b-152">Azure AD</span><span class="sxs-lookup"><span data-stu-id="ab90b-152">Azure AD</span></span>

<span data-ttu-id="ab90b-153">Azure 구독을 위한 [Azure AD][azure-ad] 테넌트가 Jenkins 사용자를 위해 SSO를 사용하도록 설정하고, Jenkins 작업이 Azure 리소스에 액세스할 수 있도록 [서비스 주체][service-principal]를 설정하는 데 사용됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-153">The [Azure AD][azure-ad] tenant for your Azure subscription is used to enable SSO for Jenkins users and set up [service principals][service-principal] that enable Jenkins jobs to access Azure resources.</span></span>

<span data-ttu-id="ab90b-154">SSO 인증 및 권한 부여는 Jenkins 서버에 설치된 Azure AD 플러그 인에서 구현됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-154">SSO authentication and authorization are implemented by the Azure AD plugin  installed on the Jenkins server.</span></span> <span data-ttu-id="ab90b-155">SSO를 사용하면 Jenkins 서버에 로그온할 때 Azure AD에서 조직 자격 증명을 사용하여 인증할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-155">SSO allows you to authenticate using your organization credentials from Azure AD when logging on to the Jenkins server.</span></span> <span data-ttu-id="ab90b-156">Azure AD 플러그 인을 구성할 때 Jenkin 서버에 대한 권한이 있는 사용자의 수준을 지정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-156">When configuring the Azure AD plugin, you can specify the level of a user’s authorized access to the Jenkin server.</span></span>

<span data-ttu-id="ab90b-157">Azure 리소스에 대한 액세스 권한과 함께 Jenkins 작업을 제공하기 위해 Azure AD 관리자는 서비스 주체를 작성합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-157">To provide Jenkins jobs with access to Azure resources, an Azure AD administrator creates service principals.</span></span> <span data-ttu-id="ab90b-158">이러한 권한 부여 응용 프로그램(이 경우 Jenkins 작업)은 Azure 리소스에 대해 [인증되고 권한 부여된 액세스][ad-sp]를 포함합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-158">These grant applications—in this case, the Jenkins jobs—[authenticated, authorized access][ad-sp] to Azure resources.</span></span>

<span data-ttu-id="ab90b-159">[RBAC][rbac]는 할당된 역할을 통해 사용자 또는 서비스 주체의 Azure 리소스에 대한 액세스를 추가로 정의 및 제어합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-159">[RBAC][rbac] further defines and controls access to Azure resources for users or service principals through their assigned role.</span></span> <span data-ttu-id="ab90b-160">기본 제공 및 사용자 지정 역할 둘 다 지원됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-160">Both built-in and custom roles are supported.</span></span> <span data-ttu-id="ab90b-161">역할은 또한 파이프라인을 보호하고 사용자 또는 에이전트의 책임이 올바르게 할당되고 권한 부여되는지 확인하는 데 도움이 됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-161">Roles also help secure the pipeline and ensure that a user’s or agent’s responsibilities are assigned and authorized correctly.</span></span> <span data-ttu-id="ab90b-162">또한 Azure 자산에 대한 액세스를 제한하는 데 RBAC를 설정할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-162">In addition, RBAC can be set up to limit access to Azure assets.</span></span> <span data-ttu-id="ab90b-163">예를 들어 사용자는 특정 리소스 그룹의 자산으로만 작업하도록 제한할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-163">For example, a user can be limited to working with only the assets in a particular resource group.</span></span>

### <a name="storage"></a><span data-ttu-id="ab90b-164">Storage</span><span class="sxs-lookup"><span data-stu-id="ab90b-164">Storage</span></span>

<span data-ttu-id="ab90b-165">Azure Marketplace에서 설치되는 Jenkins [Windows Azure Storage 플러그 인][storage-plugin]을 사용하여 다른 빌드 및 테스트와 공유할 수 있는 빌드 아티팩트를 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-165">Use the Jenkins [Windows Azure Storage plugin][storage-plugin], which is installed from the Azure Marketplace, to store build artifacts that can be shared with other builds and tests.</span></span> <span data-ttu-id="ab90b-166">Jenkins 작업에서 이 플러그 인을 사용할 수 있으려면 Azure Storage 계정을 구성해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-166">An Azure Storage account must be configured before this plugin can be used by the Jenkins jobs.</span></span>

### <a name="jenkins-azure-plugins"></a><span data-ttu-id="ab90b-167">Jenkins Azure 플러그 인</span><span class="sxs-lookup"><span data-stu-id="ab90b-167">Jenkins Azure plugins</span></span>

<span data-ttu-id="ab90b-168">Azure에서 Jenkins를 위한 솔루션 템플릿은 여러 Azure 플러그 인을 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-168">The solution template for Jenkins on Azure installs several Azure plugins.</span></span> <span data-ttu-id="ab90b-169">Azure DevOps 팀은 솔루션 템플릿과, Azure Marketplace의 다른 Jenkins 제품은 물론 온-프레미스에 설정된 모든 Jenkins 마스터와 작동하는 플러그 인을 빌드하고 유지 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-169">The Azure DevOps Team builds and maintains the solution template and the following plugins, which work with other Jenkins offerings in Azure Marketplace as well as any Jenkins master set up on premises:</span></span>

-   <span data-ttu-id="ab90b-170">[Azure AD 플러그 인][configure-azure-ad]을 통해 Jenkins 서버가 Azure AD에 따라 사용자에 대한 SSO를 지원하도록 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-170">[Azure AD plugin][configure-azure-ad] allows the Jenkins server to support SSO for users based on Azure AD.</span></span>

-   <span data-ttu-id="ab90b-171">[Azure VM 에이전트][configure-agent] 플러그 인은 ARM(Azure Resource Manager) 템플릿을 사용하여 Azure 가상 머신에 Jenkins 에이전트를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-171">[Azure VM Agents][configure-agent] plugin uses an Azure Resource Manager (ARM) template to create Jenkins agents in Azure virtual machines.</span></span>

-   <span data-ttu-id="ab90b-172">[Azure 자격 증명][configure-credential] 플러그 인을 사용하여 Jenkins에 Azure 서비스 주체를 저장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-172">[Azure Credentials][configure-credential] plugin allows you to store Azure service principal credentials in Jenkins.</span></span>

-   <span data-ttu-id="ab90b-173">[Windows Azure Storage 플러그 인][configure-storage]은 [Azure Blob Storage][blob]로 빌드 아티팩트를 업로드하고 Azure Blob Storage에서 빌드 종속성을 다운로드합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-173">[Windows Azure Storage plugin][configure-storage] uploads build artifacts to, or downloads build dependencies from, [Azure Blob storage][blob].</span></span>

<span data-ttu-id="ab90b-174">또한 Azure 리소스를 사용하는 사용 가능한 모든 Azure 플러그 인의 증가하는 목록을 검토하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-174">We also recommend reviewing the growing list of all available Azure plugins that work with Azure resources.</span></span> <span data-ttu-id="ab90b-175">모든 최신 목록을 보려면, [Jenkins 플러그 인 인덱스][index]를 방문하여 Azure를 검색하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-175">To see all the latest list, visit [Jenkins Plugin Index][index] and search for Azure.</span></span> <span data-ttu-id="ab90b-176">예를 들어 배포를 위해 다음 플러그 인을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-176">For example, the following plugins are available for deployment:</span></span>

-   <span data-ttu-id="ab90b-177">[Azure Container 에이전트][container-agents]를 통해 컨테이너를 Jenkins에서 에이전트로 실행할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-177">[Azure Container Agents][container-agents] helps you to run a container as an agent in Jenkins.</span></span>

-   <span data-ttu-id="ab90b-178">[Kubernetes 연속 배포](https://aka.ms/azjenkinsk8s)는 리소스 구성을 Kubernetes 클러스터에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-178">[Kubernetes Continuous Deploy](https://aka.ms/azjenkinsk8s) deploys resource configurations to a Kubernetes cluster.</span></span>

-   <span data-ttu-id="ab90b-179">[Azure Container Service][acs]는 Kubernetes를 사용한 Azure Container Service, Marathon을 사용한 DC/OS 또는 Docker Swarm에 구성을 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-179">[Azure Container Service][acs] deploys configurations to Azure Container Service with Kubernetes, DC/OS with Marathon, or Docker Swarm.</span></span>

-   <span data-ttu-id="ab90b-180">[Azure Functions][functions]는 프로젝트를 Azure Function에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-180">[Azure Functions][functions] deploys your project to Azure Function.</span></span>

-   <span data-ttu-id="ab90b-181">[Azure App Service][app-service]는 Azure App Service에 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-181">[Azure App Service][app-service] deploys to Azure App Service.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="ab90b-182">확장성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ab90b-182">Scalability considerations</span></span>

<span data-ttu-id="ab90b-183">Jenkins는 매우 큰 워크로드를 지원하도록 확장할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-183">Jenkins can scale to support very large workloads.</span></span> <span data-ttu-id="ab90b-184">탄력적 빌드를 위해서는 Jenkins 마스터 서버에서 빌드를 실행하지 마세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-184">For elastic builds, do not run builds on the Jenkins master server.</span></span> <span data-ttu-id="ab90b-185">대신, 빌드 작업을 필요에 따라 탄력적으로 규모 확장 및 축소할 수 있는 Jenkins 에이전트에 오프로드하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-185">Instead, offload build tasks to Jenkins agents, which can be elastically scaled in and out as need.</span></span> <span data-ttu-id="ab90b-186">에이전트 크기 조정을 위해 두 가지 옵션을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-186">Consider two options for scaling agents:</span></span>

- <span data-ttu-id="ab90b-187">[Azure VM 에이전트][vm-agent] 플러그 인을 사용하여 Azure VM에서 실행되는 Jenkins 에이전트를 만듭니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-187">Use the [Azure VM Agents][vm-agent] plugin to create Jenkins agents that run in Azure VMs.</span></span> <span data-ttu-id="ab90b-188">이 플러그 인을 통해 에이전트에 대한 탄력적 스케일 아웃이 가능하며 가상 머신의 고유 형식을 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-188">This plugin enables elastic scale-out for agents and can use distinct types of virtual machines.</span></span> <span data-ttu-id="ab90b-189">Azure Marketplace에서 다른 기본 이미지를 선택하거나 사용자 지정 이미지를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-189">You can select a different base image from Azure Marketplace or use a custom image.</span></span> <span data-ttu-id="ab90b-190">Jenkins 에이전트를 크기 조정하는 방법에 대한 자세한 내용은 Jenkins 설명서의 [크기 조정을 위한 설계][scale]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-190">For details about how the Jenkins agents scale, see [Architecting for Scale][scale] in the Jenkins documentation.</span></span>

- <span data-ttu-id="ab90b-191">[Azure Container 에이전트][container-agents] 플러그 인을 사용하여 [Kubernetes를 사용한 Azure Container Service](/azure/container-service/kubernetes/) 또는 [Azure Container Instances](/azure/container-instances/)에서 컨테이너를 에이전트로 실행합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-191">Use the [Azure Container Agents][container-agents] plugin to run a container as an agent in either [Azure Container Service with Kubernetes](/azure/container-service/kubernetes/), or [Azure Container Instances](/azure/container-instances/).</span></span>

<span data-ttu-id="ab90b-192">가상 머신은 일반적으로 크기 조정을 위해 컨테이너보다 더 많은 비용을 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-192">Virtual machines generally cost more to scale than containers.</span></span> <span data-ttu-id="ab90b-193">그러나 크기 조정을 위해 컨테이너를 사용하려면 컨테이너와 함께 빌드 프로세스를 실행해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-193">To use containers for scaling, however, your build process must run with containers.</span></span>

<span data-ttu-id="ab90b-194">또한 Azure Storage를 사용하여 다른 빌드 에이전트에 의해 파이프라인의 다음 단계에서 사용할 수 있는 빌드 아티팩트를 공유합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-194">Also, use Azure Storage to share build artifacts that may be used in the next stage of the pipeline by other build agents.</span></span>

### <a name="scaling-the-jenkins-server"></a><span data-ttu-id="ab90b-195">Jenkins 서버 크기 조정</span><span class="sxs-lookup"><span data-stu-id="ab90b-195">Scaling the Jenkins server</span></span> 

<span data-ttu-id="ab90b-196">VM 크기를 변경하여 Jenkins 서버 VM의 규모를 확장 또는 축소할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-196">You can scale the Jenkins server VM up or down by changing the VM size.</span></span> <span data-ttu-id="ab90b-197">[Azure의 Jenkins용 솔루션 템플릿][azure-market]은 기본적으로 DS2 v2 크기(2개의 CPU, 7GB 포함)를 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-197">The [solution template for Jenkins on Azure][azure-market] specifies the DS2 v2 size (with two CPUs, 7 GB) by default.</span></span> <span data-ttu-id="ab90b-198">이 크기로 중소 규모의 팀 워크로드를 처리합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-198">This size handles a small to medium team workload.</span></span> <span data-ttu-id="ab90b-199">서버를 빌드할 때 다른 옵션을 선택하여 VM 크기를 변경합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-199">Change the VM size by choosing a different option when building out the server.</span></span> 

<span data-ttu-id="ab90b-200">올바른 서버 크기를 선택하는 것은 예상된 워크로드 크기에 따라 달라집니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-200">Selecting the correct server size depends on the size of the expected workload.</span></span> <span data-ttu-id="ab90b-201">Jenkins 커뮤니티는 요구 사항에 가장 잘 맞는 구성을 식별할 수 있도록 [선택 가이드][selection-guide]를 유지 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-201">The Jenkins community maintains a [selection guide][selection-guide] to help identify the configuration that best meets your requirements.</span></span> <span data-ttu-id="ab90b-202">Azure에서는 모든 요구 사항을 충족하도록 수많은 [Linux VM용 크기][sizes-linux]를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-202">Azure offers many [sizes for Linux VMs][sizes-linux] to meet any requirements.</span></span> <span data-ttu-id="ab90b-203">Jenkins 마스터의 크기 조정에 대한 자세한 내용은 Jenkins 마스터 크기 조정에 대한 자세한 내용을 포함하는 [모범 사례][best-practices]의 Jenkins 커뮤니티를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-203">For more information about scaling the Jenkins master, refer to the Jenkins community of [best practices][best-practices], which also includes details about scaling Jenkins master.</span></span>


## <a name="availability-considerations"></a><span data-ttu-id="ab90b-204">가용성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ab90b-204">Availability considerations</span></span>

<span data-ttu-id="ab90b-205">워크플로의 가용성 요구 사항과 Jenkin 서버가 다운되는 경우 Jenkin 상태를 복구하는 방법을 평가합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-205">Assess the availability requirements for your workflow and how to recover the Jenkin state should the Jenkin server goes down.</span></span> <span data-ttu-id="ab90b-206">가용성 요구 사항을 평가하려면 두 가지 공통 메트릭을 고려합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-206">To assess your availability requirements, consider two common metrics:</span></span>

-   <span data-ttu-id="ab90b-207">RTO(복구 시간 목표)는 Jenkins 없이 얼마나 오래 갈 수 있는지 지정합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-207">Recovery Time Objective (RTO) specifies how long you can go without Jenkins.</span></span>

-   <span data-ttu-id="ab90b-208">RPO(복구 지점 목표)는 서비스 중단으로 인해 Jenkins에 영향을 줄 경우 잃을 수 있는 데이터 양을 나타냅니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-208">Recovery Point Objective (RPO) indicates how much data you can afford to lose if a disruption in service affects Jenkins.</span></span>

<span data-ttu-id="ab90b-209">실제로 RTO와 RPO는 중복성과 백업을 의미합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-209">In practice, RTO and RPO imply redundancy and backup.</span></span> <span data-ttu-id="ab90b-210">가용성은 Azure의 일부인 하드웨어 복구의 문제가 아니라 Jenkins 서버의 상태를 유지하는 것을 보장합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-210">Availability is not a question of hardware recovery—that is part of Azure—but rather ensuring you maintain the state of your Jenkins server.</span></span> <span data-ttu-id="ab90b-211">이 참조 아키텍처는 단일 가상 머신에 대해 99.9%의 작동 시간을 보장하는 [Azure Service Level Agreement(서비스 수준 약정)][sla]를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-211">This reference architecture uses the [Azure service level agreement][sla] (SLA), which guarantees 99.9-percent uptime for a single virtual machine.</span></span> <span data-ttu-id="ab90b-212">이 SLA가 가동 시간 요구 사항을 충족시키지 못하는 경우 재해 복구 계획을 수립했는지 확인하거나 [다중 마스터 Jenkins 서버][multi-master] 배포(이 문서에서는 다루지 않음)를 사용하는 것을 고려하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-212">If this SLA doesn't meet your uptime requirements, make sure you have a plan for disaster recovery, or consider using a [multi-master Jenkins server][multi-master] deployment (not covered in this document).</span></span>

<span data-ttu-id="ab90b-213">배포 7단계에서 재해 복구 [스크립트][disaster]를 사용하여 Jenkins 서버 상태를 저장할 관리 디스크가 있는 Azure Storage 계정을 만드는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-213">Consider using the disaster recovery [scripts][disaster] in step 7 of the deployment to create an Azure Storage account with managed disks to store the Jenkins server state.</span></span> <span data-ttu-id="ab90b-214">Jenkins가 다운되면 이 별도의 저장소 계정에 저장된 상태로 복원할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-214">If Jenkins goes down, it can be restored to the state stored in this separate storage account.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="ab90b-215">보안 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ab90b-215">Security considerations</span></span>

<span data-ttu-id="ab90b-216">다음과 같은 방법을 사용하여 기본 Jenkins 서버의 보안을 잠글 수 있습니다. 기본 상태에서는 안전하지 않기 때문입니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-216">Use the following approaches to help lock down security on a basic Jenkins server, since in its basic state, it is not secure.</span></span>

-   <span data-ttu-id="ab90b-217">Jenkin 서버에 보안 로그온하는 방법을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-217">Set up a way to secure logon to the Jenkin server.</span></span> <span data-ttu-id="ab90b-218">HTTP는 기본적으로 안전하지 않으며 이 아키텍처에서는 HTTP를 사용하고 공용 IP를 보유합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-218">HTTP is not secure By default, this architecture uses HTTP and has a public IP.</span></span> <span data-ttu-id="ab90b-219">보안 로그온에 사용되는 [Nginx 서버에 HTTPS][nginx]를 설정하는 것이 좋습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-219">Consider setting up [HTTPS on the Nginx server][nginx] being used for a secure logon.</span></span>

    > [!NOTE]
    > <span data-ttu-id="ab90b-220">서버에 SSL을 추가할 경우 Jenkins 서브넷에 대해 NSG 규칙을 만들어 포트 443을 엽니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-220">When adding SSL to your server, create an NSG rule for the Jenkins subnet to open port 443.</span></span> <span data-ttu-id="ab90b-221">자세한 내용은 [Azure Portal을 사용하여 가상 머신에 대한 포털을 여는 방법][port443]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-221">For more information, see [How to open ports to a virtual machine with the Azure portal][port443].</span></span>
    > 

-   <span data-ttu-id="ab90b-222">Jenkins 구성이 교차 사이트 요청 위조를 방지하는지 확인합니다(Jenkins 관리 \> 전역 보안 구성).</span><span class="sxs-lookup"><span data-stu-id="ab90b-222">Ensure that the Jenkins configuration prevents cross site request forgery (Manage Jenkins \> Configure Global Security).</span></span> <span data-ttu-id="ab90b-223">이는 Microsoft Jenkins 서버의 기본값입니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-223">This is the default for Microsoft Jenkins Server.</span></span>

-   <span data-ttu-id="ab90b-224">[매트릭스 권한 부여 전략 플러그 인][matrix]을 사용하여 Jenkins 대시보드에 대한 읽기 전용 액세스를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-224">Configure read-only access to the Jenkins dashboard by using the [Matrix Authorization Strategy Plugin][matrix].</span></span>

-   <span data-ttu-id="ab90b-225">Key Vault를 사용하여 Azure 자산, 파이프라인의 에이전트 및 타사 구성 요소의 비밀을 처리하도록 [Azure 자격 증명][configure-credential] 플러그 인을 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-225">Install the [Azure Credentials][configure-credential] plugin to use Key Vault to handle secrets for the Azure assets, the agents in the pipeline, and third-party components.</span></span>

-   <span data-ttu-id="ab90b-226">RBAC를 사용하여 서비스 사용자의 액세스 권한을 작업 실행에 필요한 최소 수준으로 제한합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-226">Use RBAC to restrict the access of the service principal to the minimum required to run the jobs.</span></span> <span data-ttu-id="ab90b-227">이렇게 하면 rogue 작업의 손상 범위를 제한할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-227">This helps limit the scope of damage from a rogue job.</span></span>

<span data-ttu-id="ab90b-228">Jenkins 작업에서는 Azure Container Service와 같은 권한 부여를 요구하는 Azure 서비스에 액세스하는 데 보통 비밀을 요구합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-228">Jenkins jobs often require secrets to access Azure services that require authorization, such as Azure Container Service.</span></span> <span data-ttu-id="ab90b-229">[Azure 자격 증명 플러그 인][configure-credential]과 함께 [Key Vault][key-vault]를 사용하여 이러한 비밀은 안전하게 관리합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-229">Use [Key Vault][key-vault] along with the [Azure Credential plugin][configure-credential] to manage these secrets securely.</span></span> <span data-ttu-id="ab90b-230">Key Vault를 사용하여 서비스 주체 자격 증명, 암호, 토큰 및 기타 비밀을 저장합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-230">Use Key Vault to store service principal credentials, passwords, tokens, and other secrets.</span></span>

<span data-ttu-id="ab90b-231">Azure 리소스의 보안 상태를 중앙에서 살펴 보려면 [Azure Security Center][security-center]를 사용합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-231">To get a central view of the security state of your Azure resources, use [Azure Security Center][security-center].</span></span> <span data-ttu-id="ab90b-232">Security Center는 잠재적인 보안 문제를 모니터링하고 배포의 보안 상태에 대한 종합적인 그림을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-232">Security Center monitors potential security issues and provides a comprehensive picture of the security health of your deployment.</span></span> <span data-ttu-id="ab90b-233">보안 센터는 각 Azure 구독을 기준으로 구성됩니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-233">Security Center is configured per Azure subscription.</span></span> <span data-ttu-id="ab90b-234">[Azure Security Center 빠른 시작 가이드][quick-start]에 설명된 것처럼 보안 데이터 수집을 사용하도록 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-234">Enable security data collection as described in the [Azure Security Center quick start guide][quick-start].</span></span> <span data-ttu-id="ab90b-235">데이터 수집이 사용되도록 설정되면 Security Center는 해당 구독에서 만든 모든 가상 머신을 자동으로 검색합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-235">When data collection is enabled, Security Center automatically scans any virtual machines created under that subscription.</span></span>

<span data-ttu-id="ab90b-236">Jenkins 서버에는 사용자 고유의 사용자 관리 시스템이 있으며 Jenkins 커뮤니티는 [Azure에서 Jenkins 인스턴스 보안][secure-jenkins]을 위한 모범 사례를 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-236">The Jenkins server has its own user management system, and the Jenkins community provides best practices for [securing a Jenkins instance on Azure][secure-jenkins].</span></span> <span data-ttu-id="ab90b-237">Azure에서 Jenkins를 위한 솔루션 템플릿은 이러한 모범 사례를 구현합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-237">The solution template for Jenkins on Azure implements these best practices.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="ab90b-238">관리 효율성 고려 사항</span><span class="sxs-lookup"><span data-stu-id="ab90b-238">Manageability considerations</span></span>

<span data-ttu-id="ab90b-239">리소스 그룹을 사용하여 배포된 Azure 리소스를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-239">Use resource groups to organize the Azure resources that are deployed.</span></span> <span data-ttu-id="ab90b-240">프로덕션 환경 및 개발/테스트 환경을 별도의 리소스 그룹에 배포하여 각 환경의 리소스를 모니터링하고 리소스 그룹별로 청구 비용을 롤업할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-240">Deploy production environments and development/test environments in separate resource groups, so that you can monitor each environment’s resources and roll up billing costs by resource group.</span></span> <span data-ttu-id="ab90b-241">리소스를 하나의 집합으로 삭제할 수도 있습니다. 이러한 기능은 테스트 배포에서 매우 유용합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-241">You can also delete resources as a set, which is very useful for test deployments.</span></span>

<span data-ttu-id="ab90b-242">Azure는 전체 인프라를 [모니터링 및 진단][monitoring-diag]하는 여러 기능을 제공합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-242">Azure provides several features for [monitoring and diagnostics][monitoring-diag] of the overall infrastructure.</span></span> <span data-ttu-id="ab90b-243">CPU 사용량을 모니터링하기 위해 이 아키텍처는 Azure Monitor를 배포합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-243">To monitor CPU usage, this architecture deploys Azure Monitor.</span></span> <span data-ttu-id="ab90b-244">예를 들어 Azure Monitor를 사용하여 CPU 사용률을 모니터링하고 CPU 사용량이 80%를 초과하는 경우 알림을 보낼 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-244">For example, you can use Azure Monitor to monitor CPU utilization, and send a notification if CPU usage exceeds 80 percent.</span></span> <span data-ttu-id="ab90b-245">(높은 CPU 사용량은 Jenkins 서버 VM을 강화하려는 경우가 있음을 나타냅니다.) 또한 VM이 실패하거나 사용할 수 없게 되는 경우 지정된 사용자에게 알릴 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-245">(High CPU usage indicates that you might want to scale up the Jenkins server VM.) You can also notify a designated user if the VM fails or becomes unavailable.</span></span>

## <a name="communities"></a><span data-ttu-id="ab90b-246">커뮤니티</span><span class="sxs-lookup"><span data-stu-id="ab90b-246">Communities</span></span>

<span data-ttu-id="ab90b-247">커뮤니티는 질문에 대답하고 성공적인 배포를 설정하는 데 도움을 줄 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-247">Communities can answer questions and help you set up a successful deployment.</span></span> <span data-ttu-id="ab90b-248">다음을 고려해 보세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-248">Consider the following:</span></span>

-   [<span data-ttu-id="ab90b-249">Jenkins 커뮤니티 블로그</span><span class="sxs-lookup"><span data-stu-id="ab90b-249">Jenkins Community Blog</span></span>](https://jenkins.io/node/)
-   [<span data-ttu-id="ab90b-250">Azure 포럼</span><span class="sxs-lookup"><span data-stu-id="ab90b-250">Azure Forum</span></span>](https://azure.microsoft.com/support/forums/)
-   [<span data-ttu-id="ab90b-251">Stack Overflow Jenkins</span><span class="sxs-lookup"><span data-stu-id="ab90b-251">Stack Overflow Jenkins</span></span>](https://stackoverflow.com/tags/jenkins/info)

<span data-ttu-id="ab90b-252">Jenkins 커뮤니티의 더 많은 모범 사례는 [Jenkins 모범 사례][jenkins-best]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-252">For more best practices from the Jenkins community, visit [Jenkins best practices][jenkins-best].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="ab90b-253">솔루션 배포</span><span class="sxs-lookup"><span data-stu-id="ab90b-253">Deploy the solution</span></span>

<span data-ttu-id="ab90b-254">이 아키텍처를 배포하려면 아래 단계에 따라 [Azure에서 Jenkins를 위한 솔루션 템플릿][azure-market]을 설치한 다음 아래 단계에서 모니터링 및 재해 복구를 설정하는 스크립트를 설치합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-254">To deploy this architecture, follow the steps below to install the [solution template for Jenkins on Azure][azure-market], then install the scripts that set up monitoring and disaster recovery in the steps below.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="ab90b-255">필수 조건</span><span class="sxs-lookup"><span data-stu-id="ab90b-255">Prerequisites</span></span>

- <span data-ttu-id="ab90b-256">이 참조 아키텍처에는 Azure 구독이 필요합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-256">This reference architecture requires an Azure subscription.</span></span> 
- <span data-ttu-id="ab90b-257">Azure 서비스 주체를 만들려면 배포된 Jenkins 서버와 연결된 Azure AD 테넌트에 대한 관리자 권한이 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-257">To create an Azure service principal, you must have admin rights to the Azure AD tenant that is associated with the deployed Jenkins server.</span></span>
- <span data-ttu-id="ab90b-258">이러한 지침에서는 Jenkins 관리자가 참가자 권한 이상을 보유하는 Azure 사용자라고 가정합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-258">These instructions assume that the Jenkins administrator is also an Azure user with at least Contributor privileges.</span></span>

### <a name="step-1-deploy-the-jenkins-server"></a><span data-ttu-id="ab90b-259">1단계: Jenkins 서버 배포</span><span class="sxs-lookup"><span data-stu-id="ab90b-259">Step 1: Deploy the Jenkins server</span></span>

1.  <span data-ttu-id="ab90b-260">웹 브라우저에서 [Jenkins의 Azure Marketplace 이미지][azure-market]를 열고 페이지의 왼쪽에서 **지금 가져오기**를 선택합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-260">Open the [Azure marketplace image for Jenkins][azure-market] in your web browser and select **GET IT NOW** from the left side of the page.</span></span>

2.  <span data-ttu-id="ab90b-261">가격 책정 세부 정보를 검토하고 **계속**을 선택한 후 **만들기**를 선택하여 Azure Portal에서 Jenkins 서버를 구성합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-261">Review the pricing details and select **Continue**, then select **Create** to configure the Jenkins server in the Azure portal.</span></span>

<span data-ttu-id="ab90b-262">자세한 지침은 [Azure Portal에서 Azure Linux VM에 Jenkins 서버 만들기][create-jenkins]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-262">For detailed instructions, see [Create a Jenkins server on an Azure Linux VM from the Azure portal][create-jenkins].</span></span> <span data-ttu-id="ab90b-263">이 참조 아키텍처의 경우 관리자 로그온으로 서버를 시작하고 실행하는 것으로 충분합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-263">For this reference architecture, it is sufficient to get the server up and running with the admin logon.</span></span> <span data-ttu-id="ab90b-264">그런 다음 다른 여러 서비스를 사용하도록 프로비전할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-264">Then you can provision it to use various other services.</span></span>

### <a name="step-2-set-up-sso"></a><span data-ttu-id="ab90b-265">2단계: SSO 설정</span><span class="sxs-lookup"><span data-stu-id="ab90b-265">Step 2: Set up SSO</span></span>

<span data-ttu-id="ab90b-266">이 단계는 Jenkins 관리자에 의해 실행되며, 이때 Jenkins 관리자는 구독의 Azure AD 디렉터리에 사용자 계정을 포함하고 참가자 역할이 할당되어 있어야 합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-266">The step is run by the Jenkins administrator, who must also have a user account in the subscription’s Azure AD directory and must be assigned the Contributor role.</span></span>

<span data-ttu-id="ab90b-267">Jenkin 서버의 Jenkins 업데이트 센터에서 [Azure AD 플러그 인][configure-azure-ad]을 사용하고 지침에 따라 SSO를 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-267">Use the [Azure AD Plugin][configure-azure-ad] from the Jenkins Update Center in the Jenkin server and follow the instructions to set up SSO.</span></span>

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a><span data-ttu-id="ab90b-268">3단계: Azure VM 에이전트 플러그 인이 있는 Jenkins 서버 프로비전</span><span class="sxs-lookup"><span data-stu-id="ab90b-268">Step 3: Provision Jenkins server with Azure VM Agent plugin</span></span>

<span data-ttu-id="ab90b-269">이 단계는 Jenkins 관리자에 의해 실행되며 이미 설치된 Azure VM 에이전트 플러그 인을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-269">The step is run by the Jenkins administrator to set up the Azure VM Agent plugin, which is already installed.</span></span>

<span data-ttu-id="ab90b-270">[다음 단계에 따라 플러그 인을 구성합니다][configure-agent].</span><span class="sxs-lookup"><span data-stu-id="ab90b-270">[Follow these steps to configure the plugin][configure-agent].</span></span> <span data-ttu-id="ab90b-271">플러그 인에 대한 서비스 주체 설정에 대한 자습서는 [Azure VM 에이전트를 사용하여 수요에 맞게 Jenkins 배포 규모 조정][scale-agent]을 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-271">For a tutorial about setting up service principals for the plugin, see [Scale your  Jenkins deployments to meet demand with Azure VM agents][scale-agent].</span></span>

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a><span data-ttu-id="ab90b-272">4단계: Azure Storage가 있는 Jenkins 서버 프로비전</span><span class="sxs-lookup"><span data-stu-id="ab90b-272">Step 4: Provision Jenkins server with Azure Storage</span></span>

<span data-ttu-id="ab90b-273">이 단계는 Jenkins 관리자에 의해 실행되며 이미 설치된 Windows Azure Storage 플러그 인을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-273">The step is run by the Jenkins administrator, who sets up the Windows Azure Storage Plugin, which is already installed.</span></span>

<span data-ttu-id="ab90b-274">[다음 단계에 따라 플러그 인을 구성합니다][configure-storage].</span><span class="sxs-lookup"><span data-stu-id="ab90b-274">[Follow these steps to configure the plugin][configure-storage].</span></span>

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a><span data-ttu-id="ab90b-275">5단계: Azure 자격 증명 플러그 인이 있는 Jenkins 서버 프로비전</span><span class="sxs-lookup"><span data-stu-id="ab90b-275">Step 5: Provision Jenkins server with Azure Credential plugin</span></span>

<span data-ttu-id="ab90b-276">이 단계는 Jenkins 관리자에 의해 실행되며 이미 설치된 Azure 자격 증명 플러그 인을 설정합니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-276">The step is run by the Jenkins administrator to set up the Azure Credential plugin, which is already installed.</span></span>

<span data-ttu-id="ab90b-277">[다음 단계에 따라 플러그 인을 구성합니다][configure-credential].</span><span class="sxs-lookup"><span data-stu-id="ab90b-277">[Follow these steps to configure the plugin][configure-credential].</span></span>

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a><span data-ttu-id="ab90b-278">6단계: Azure Monitor 서비스로 모니터링할 Jenkins 서버 프로비전</span><span class="sxs-lookup"><span data-stu-id="ab90b-278">Step 6: Provision Jenkins server for monitoring by the Azure Monitor Service</span></span>

<span data-ttu-id="ab90b-279">Jenkins 서버에 대한 모니터링을 설정하려면 [Azure 서비스의 Azure Monitor에서 메트릭 경고 만들기][create-metric]를 참조하세요.</span><span class="sxs-lookup"><span data-stu-id="ab90b-279">To set up monitoring for your Jenkins server, follow the instructions in [Create metric alerts in Azure Monitor for Azure services][create-metric].</span></span>

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a><span data-ttu-id="ab90b-280">7단계: 재해 복구를 위해 Managed Disks로 Jenkins 서버 프로비전</span><span class="sxs-lookup"><span data-stu-id="ab90b-280">Step 7: Provision Jenkins server with Managed Disks for disaster recovery</span></span>

<span data-ttu-id="ab90b-281">Microsoft Jenkins 제품 그룹은 Jenkins 상태를 저장하는 데 사용되는 관리 디스크를 작성하는 재해 복구 스크립트를 작성했습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-281">The Microsoft Jenkins product group has created disaster recovery scripts that build a managed disk used to save the Jenkins state.</span></span> <span data-ttu-id="ab90b-282">서버가 다운되면 최신 상태로 복원할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="ab90b-282">If the server goes down, it can be restored to its latest state.</span></span>

<span data-ttu-id="ab90b-283">[GitHub][disaster]에서 재해 복구 스크립트를 다운로드하여 실행하십시오.</span><span class="sxs-lookup"><span data-stu-id="ab90b-283">Download and run the disaster recovery scripts from [GitHub][disaster].</span></span>

[acs]: https://aka.ms/azjenkinsacs
[ad-sp]: /azure/active-directory/develop/active-directory-integrating-applications
[app-service]: https://plugins.jenkins.io/azure-app-service
[azure-ad]: /azure/active-directory/
[azure-market]: https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.jenkins?tab=Overview
[best-practices]: https://jenkins.io/doc/book/architecting-for-scale/
[blob]: /azure/storage/common/storage-java-jenkins-continuous-integration-solution
[configure-azure-ad]: https://plugins.jenkins.io/azure-ad
[configure-agent]: https://plugins.jenkins.io/azure-vm-agents
[configure-credential]: https://plugins.jenkins.io/azure-credentials
[configure-storage]: https://plugins.jenkins.io/windows-azure-storage
[container-agents]: https://aka.ms/azcontaineragent
[create-jenkins]: /azure/jenkins/install-jenkins-solution-template
[create-metric]: /azure/monitoring-and-diagnostics/insights-alerts-portal
[disaster]: https://github.com/Azure/jenkins/tree/master/disaster_recovery
[functions]: https://aka.ms/azjenkinsfunctions
[index]: https://plugins.jenkins.io
[jenkins-best]: https://wiki.jenkins.io/display/JENKINS/Jenkins+Best+Practices
[jenkins-on-azure]: /azure/jenkins/
[key-vault]: /azure/key-vault/
[managed-disk]: /azure/virtual-machines/linux/managed-disks-overview
[matrix]: https://plugins.jenkins.io/matrix-auth
[monitor]: /azure/monitoring-and-diagnostics/
[monitoring-diag]: /azure/architecture/best-practices/monitoring
[multi-master]: https://jenkins.io/doc/book/architecting-for-scale/
[nginx]: https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04
[nsg]: /azure/virtual-network/virtual-networks-nsg
[quick-start]: /azure/security-center/security-center-get-started
[port443]: /azure/virtual-machines/windows/nsg-quickstart-portal
[premium]: /azure/virtual-machines/linux/premium-storage
[rbac]: /azure/active-directory/role-based-access-control-what-is
[rg]: /azure/azure-resource-manager/resource-group-overview
[scale]: https://jenkins.io/doc/book/architecting-for-scale/
[scale-agent]: /azure/jenkins/jenkins-azure-vm-agents
[selection-guide]: https://jenkins.io/doc/book/hardware-recommendations/
[service-principal]: /azure/active-directory/develop/active-directory-application-objects
[secure-jenkins]: https://jenkins.io/blog/2017/04/20/secure-jenkins-on-azure/
[security-center]: /azure/security-center/security-center-intro
[sizes-linux]: /azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json
[solution]: https://azure.microsoft.com/blog/announcing-the-solution-template-for-jenkins-on-azure/
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
[storage-plugin]: https://wiki.jenkins.io/display/JENKINS/Windows+Azure+Storage+Plugin
[subnet]: /azure/virtual-network/virtual-network-manage-subnet
[vm-agent]: https://wiki.jenkins.io/display/JENKINS/Azure+VM+Agents+plugin
[vnet]: /azure/virtual-network/virtual-networks-overview
[0]: ./images/jenkins-server.png 
