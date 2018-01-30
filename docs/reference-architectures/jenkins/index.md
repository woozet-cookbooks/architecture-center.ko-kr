---
title: "Azure에서 Jenkins 서버 실행"
description: "이 참조 아키텍처는 SSO(Single Sign-On)로 보호된 Azure에서 확장성 있는 엔터프라이즈급 Jenkins 서버를 배포하고 작동하는 방법을 보여 줍니다."
author: njray
ms.date: 01/21/18
ms.openlocfilehash: d06b16c212951c629612d69b13fa2b32b1030475
ms.sourcegitcommit: 9998334bebccb86be0f715ac7dffc0c3175aea68
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/26/2018
---
# <a name="run-a-jenkins-server-on-azure"></a>Azure에서 Jenkins 서버 실행

이 참조 아키텍처는 SSO(Single Sign-On)로 보호된 Azure에서 확장성 있는 엔터프라이즈급 Jenkins 서버를 배포하고 작동하는 방법을 보여 줍니다. 이 아키텍처는 또한 Jenkins 서버의 상태를 모니터링하는 데 Azure Monitor를 사용합니다. [**이 솔루션을 배포합니다**.](#deploy-the-solution)

![Azure에서 실행 중인 Jenkins 서버][0]

*이 아키텍처 다이어그램이 포함된 [Visio 파일](https://arch-center.azureedge.net/cdn/Jenkins-architecture.vsdx)을 다운로드합니다.*

이 아키텍처는 Azure 서비스를 통한 재해 복구를 지원하지만 가동 중지 시간이 없는 다중 마스터 또는 HA(고가용성)과 관련된 보다 고급의 스케일 아웃 시나리오는 다루지 않습니다. Azure에서 CI/CD 파이프라인을 빌드하는 방법에 대한 단계별 자습서를 비롯하여 다양한 Azure 구성 요소에 대한 일반적인 정보는 [Azure의 Jenkins][jenkins-on-azure]를 참조하세요.

이 문서에서는 Azure Storage를 사용하여 빌드 아티팩트, SSO에 필요한 보안 항목, 통합 가능한 기타 서비스 및 파이프라인에 대한 확장성을 유지하는 등 Jenkins를 지원하는 데 필요한 핵심 Azure 작업에 중점을 둡니다. 이 아키텍처는 기존 소스 제어 리포지토리에서 작동하도록 설계되었습니다. 예를 들어, 일반적인 시나리오는 GitHub 커밋에 따라 Jenkins 작업을 시작하는 것입니다.

## <a name="architecture"></a>건축

이 아키텍처는 다음과 같은 구성 요소로 구성됩니다.

-   **리소스 그룹.** [리소스 그룹][rg]은 Azure 자산을 수명, 소유자를 비롯한 기준으로 관리할 수 있도록 Azure 자산을 그룹화하는 데 사용됩니다. 리소스 그룹을 사용하여 Azure 자산을 그룹 단위로 배포 및 모니터링하고, 리소스 그룹별로 청구 비용을 추적할 수 있습니다. 리소스를 하나의 집합으로 삭제할 수도 있습니다. 이러한 기능은 테스트 배포에서 매우 유용합니다.

-   **Jenkins 서버**. 가상 머신은 [Jenkins][azure-market]를 자동화 서버로 실행하고 Jenkins 마스터 역할을 수행하도록 배포됩니다. 이 참조 아키텍처는 Azure의 Linux(Ubuntu 14.04 LTS) 가상 머신에 설치된 [Azure의 Jenkins용 솔루션 템플릿][solution]을 사용합니다. 다른 Jenkins 제품은 Azure Marketplace에서 제공됩니다.

    > [!NOTE]
    > Nginx는 VM에 설치되어 Jenkins에 대한 역방향 프록시 역할을 합니다. Jenkins 서버에 SSL을 사용하도록 Nginx를 구성할 수 있습니다.
    > 
    > 

-   **가상 네트워크**. [가상 네트워크][vnet]는 Azure 리소스를 서로 연결하고 논리적 격리를 제공합니다. 이 아키텍처에서 Jenkins 서버는 가상 네트워크에서 실행됩니다.

-   **서브넷**. Jenkins 서버는 성능에 영향을 미치지 않으면서 네트워크 트래픽을 보다 쉽게 관리하고 분리할 수 있도록 [서브넷][subnet]에 격리됩니다.

-   **NSG**. NSG([네트워크 보안 그룹][nsg])를 사용하여 네트워크 트래픽을 인터넷에서 가상 네트워크의 서브넷으로 제한합니다.

-   **관리 디스크**. [관리 디스크][managed-disk]는 응용 프로그램 저장소 및 Jenkins 서버의 상태를 유지하고 재해 복구를 제공하는 데 사용된 영구 VHD(가상 하드 디스크)입니다. 데이터 디스크는 Azure Storage에 저장됩니다. 고성능을 위해 [프리미엄 저장소][premium]를 사용하는 것이 좋습니다.

-   **Azure Blob Storage**. [Windows Azure Storage 플러그 인][configure-storage]은 Azure Blob Storage를 사용하여 다른 Jenkins 빌드로 생성 및 공유되는 빌드 아티팩트를 저장합니다.

-   **Azure AD(Azure Active Directory)**. [Azure AD][azure-ad]는 SSO를 설정할 수 있도록 사용자 인증을 지원합니다. Azure AD [서비스 주체][service-principal]는 RBAC([역할 기반 액세스 제어][rbac])를 통해 워크플로에서 각 역할 권한 부여에 대한 정책 및 사용 권한을 정의합니다. 각 서비스 주체는 Jenkins 작업과 연결됩니다.

-   **Azure Key Vault.** 비밀이 요구될 때 Azure 자원을 프로비전하는 데 사용되는 비밀 및 암호화 키를 관리하기 위해 이 아키텍처는 [Key Vault][key-vault]를 사용합니다. 파이프라인에 있는 응용 프로그램과 관련된 비밀을 저장하는 추가 도움말은 Jenkins의 [Azure 자격 증명][configure-credential] 플러그 인을 참조하세요.

-   **Azure 모니터링 서비스**. 이 서비스는 Jenkins를 호스팅하는 Azure 가상 머신을 [모니터링][monitor]합니다. 이 배포는 가상 머신 상태 및 CPU 사용률을 모니터링하고 경고를 보냅니다.

## <a name="recommendations"></a>권장 사항

대부분의 시나리오의 경우 다음 권장 사항을 적용합니다. 이러한 권장 사항을 재정의하라는 특정 요구 사항이 있는 경우가 아니면 따릅니다.

### <a name="azure-ad"></a>Azure AD

Azure 구독을 위한 [Azure AD][azure-ad] 테넌트가 Jenkins 사용자를 위해 SSO를 사용하도록 설정하고, Jenkins 작업이 Azure 리소스에 액세스할 수 있도록 [서비스 주체][service-principal]를 설정하는 데 사용됩니다.

SSO 인증 및 권한 부여는 Jenkins 서버에 설치된 Azure AD 플러그 인에서 구현됩니다. SSO를 사용하면 Jenkins 서버에 로그온할 때 Azure AD에서 조직 자격 증명을 사용하여 인증할 수 있습니다. Azure AD 플러그 인을 구성할 때 Jenkin 서버에 대한 권한이 있는 사용자의 수준을 지정할 수 있습니다.

Azure 리소스에 대한 액세스 권한과 함께 Jenkins 작업을 제공하기 위해 Azure AD 관리자는 서비스 주체를 작성합니다. 이러한 권한 부여 응용 프로그램(이 경우 Jenkins 작업)은 Azure 리소스에 대해 [인증되고 권한 부여된 액세스][ad-sp]를 포함합니다.

[RBAC][rbac]는 할당된 역할을 통해 사용자 또는 서비스 주체의 Azure 리소스에 대한 액세스를 추가로 정의 및 제어합니다. 기본 제공 및 사용자 지정 역할 둘 다 지원됩니다. 역할은 또한 파이프라인을 보호하고 사용자 또는 에이전트의 책임이 올바르게 할당되고 권한 부여되는지 확인하는 데 도움이 됩니다. 또한 Azure 자산에 대한 액세스를 제한하는 데 RBAC를 설정할 수 있습니다. 예를 들어 사용자는 특정 리소스 그룹의 자산으로만 작업하도록 제한할 수 있습니다.

### <a name="storage"></a>Storage

Azure Marketplace에서 설치되는 Jenkins [Windows Azure Storage 플러그 인][storage-plugin]을 사용하여 다른 빌드 및 테스트와 공유할 수 있는 빌드 아티팩트를 저장합니다. Jenkins 작업에서 이 플러그 인을 사용할 수 있으려면 Azure Storage 계정을 구성해야 합니다.

### <a name="jenkins-azure-plugins"></a>Jenkins Azure 플러그 인

Azure에서 Jenkins를 위한 솔루션 템플릿은 여러 Azure 플러그 인을 설치합니다. Azure DevOps 팀은 솔루션 템플릿과, Azure Marketplace의 다른 Jenkins 제품은 물론 온-프레미스에 설정된 모든 Jenkins 마스터와 작동하는 플러그 인을 빌드하고 유지 관리합니다.

-   [Azure AD 플러그 인][configure-azure-ad]을 통해 Jenkins 서버가 Azure AD에 따라 사용자에 대한 SSO를 지원하도록 할 수 있습니다.

-   [Azure VM 에이전트][configure-agent] 플러그 인은 ARM(Azure Resource Manager) 템플릿을 사용하여 Azure 가상 머신에 Jenkins 에이전트를 만듭니다.

-   [Azure 자격 증명][configure-credential] 플러그 인을 사용하여 Jenkins에 Azure 서비스 주체를 저장할 수 있습니다.

-   [Windows Azure Storage 플러그 인][configure-storage]은 [Azure Blob Storage][blob]로 빌드 아티팩트를 업로드하고 Azure Blob Storage에서 빌드 종속성을 다운로드합니다.

또한 Azure 리소스를 사용하는 사용 가능한 모든 Azure 플러그 인의 증가하는 목록을 검토하는 것이 좋습니다. 모든 최신 목록을 보려면, [Jenkins 플러그 인 인덱스][index]를 방문하여 Azure를 검색하세요. 예를 들어 배포를 위해 다음 플러그 인을 사용할 수 있습니다.

-   [Azure Container 에이전트][container-agents]를 통해 컨테이너를 Jenkins에서 에이전트로 실행할 수 있습니다.

-   [Kubernetes 연속 배포](https://aka.ms/azjenkinsk8s)는 리소스 구성을 Kubernetes 클러스터에 배포합니다.

-   [Azure Container Service][acs]는 Kubernetes를 사용한 Azure Container Service, Marathon을 사용한 DC/OS 또는 Docker Swarm에 구성을 배포합니다.

-   [Azure Functions][functions]는 프로젝트를 Azure Function에 배포합니다.

-   [Azure App Service][app-service]는 Azure App Service에 배포합니다.

## <a name="scalability-considerations"></a>확장성 고려 사항

Jenkins는 매우 큰 워크로드를 지원하도록 확장할 수 있습니다. 탄력적 빌드를 위해서는 Jenkins 마스터 서버에서 빌드를 실행하지 마세요. 대신, 빌드 작업을 필요에 따라 탄력적으로 규모 확장 및 축소할 수 있는 Jenkins 에이전트에 오프로드하세요. 에이전트 크기 조정을 위해 두 가지 옵션을 고려합니다.

- [Azure VM 에이전트][vm-agent] 플러그 인을 사용하여 Azure VM에서 실행되는 Jenkins 에이전트를 만듭니다. 이 플러그 인을 통해 에이전트에 대한 탄력적 스케일 아웃이 가능하며 가상 머신의 고유 형식을 사용할 수 있습니다. Azure Marketplace에서 다른 기본 이미지를 선택하거나 사용자 지정 이미지를 사용할 수 있습니다. Jenkins 에이전트를 크기 조정하는 방법에 대한 자세한 내용은 Jenkins 설명서의 [크기 조정을 위한 설계][scale]를 참조하세요.

- [Azure Container 에이전트][container-agents] 플러그 인을 사용하여 [Kubernetes를 사용한 Azure Container Service](/azure/container-service/kubernetes/) 또는 [Azure Container Instances](/azure/container-instances/)에서 컨테이너를 에이전트로 실행합니다.

가상 머신은 일반적으로 크기 조정을 위해 컨테이너보다 더 많은 비용을 사용합니다. 그러나 크기 조정을 위해 컨테이너를 사용하려면 컨테이너와 함께 빌드 프로세스를 실행해야 합니다.

또한 Azure Storage를 사용하여 다른 빌드 에이전트에 의해 파이프라인의 다음 단계에서 사용할 수 있는 빌드 아티팩트를 공유합니다.

### <a name="scaling-the-jenkins-server"></a>Jenkins 서버 크기 조정 

VM 크기를 변경하여 Jenkins 서버 VM의 규모를 확장 또는 축소할 수 있습니다. [Azure의 Jenkins용 솔루션 템플릿][azure-market]은 기본적으로 DS2 v2 크기(2개의 CPU, 7GB 포함)를 지정합니다. 이 크기로 중소 규모의 팀 워크로드를 처리합니다. 서버를 빌드할 때 다른 옵션을 선택하여 VM 크기를 변경합니다. 

올바른 서버 크기를 선택하는 것은 예상된 워크로드 크기에 따라 달라집니다. Jenkins 커뮤니티는 요구 사항에 가장 잘 맞는 구성을 식별할 수 있도록 [선택 가이드][selection-guide]를 유지 관리합니다. Azure에서는 모든 요구 사항을 충족하도록 수많은 [Linux VM용 크기][sizes-linux]를 제공합니다. Jenkins 마스터의 크기 조정에 대한 자세한 내용은 Jenkins 마스터 크기 조정에 대한 자세한 내용을 포함하는 [모범 사례][best-practices]의 Jenkins 커뮤니티를 참조하세요.


## <a name="availability-considerations"></a>가용성 고려 사항

워크플로의 가용성 요구 사항과 Jenkin 서버가 다운되는 경우 Jenkin 상태를 복구하는 방법을 평가합니다. 가용성 요구 사항을 평가하려면 두 가지 공통 메트릭을 고려합니다.

-   RTO(복구 시간 목표)는 Jenkins 없이 얼마나 오래 갈 수 있는지 지정합니다.

-   RPO(복구 지점 목표)는 서비스 중단으로 인해 Jenkins에 영향을 줄 경우 잃을 수 있는 데이터 양을 나타냅니다.

실제로 RTO와 RPO는 중복성과 백업을 의미합니다. 가용성은 Azure의 일부인 하드웨어 복구의 문제가 아니라 Jenkins 서버의 상태를 유지하는 것을 보장합니다. 이 참조 아키텍처는 단일 가상 머신에 대해 99.9%의 작동 시간을 보장하는 [Azure Service Level Agreement(서비스 수준 약정)][sla]를 사용합니다. 이 SLA가 가동 시간 요구 사항을 충족시키지 못하는 경우 재해 복구 계획을 수립했는지 확인하거나 [다중 마스터 Jenkins 서버][multi-master] 배포(이 문서에서는 다루지 않음)를 사용하는 것을 고려하세요.

배포 7단계에서 재해 복구 [스크립트][disaster]를 사용하여 Jenkins 서버 상태를 저장할 관리 디스크가 있는 Azure Storage 계정을 만드는 것이 좋습니다. Jenkins가 다운되면 이 별도의 저장소 계정에 저장된 상태로 복원할 수 있습니다.

## <a name="security-considerations"></a>보안 고려 사항

다음과 같은 방법을 사용하여 기본 Jenkins 서버의 보안을 잠글 수 있습니다. 기본 상태에서는 안전하지 않기 때문입니다.

-   Jenkin 서버에 보안 로그온하는 방법을 설정합니다. HTTP는 기본적으로 안전하지 않으며 이 아키텍처에서는 HTTP를 사용하고 공용 IP를 보유합니다. 보안 로그온에 사용되는 [Nginx 서버에 HTTPS][nginx]를 설정하는 것이 좋습니다.

    > [!NOTE]
    > 서버에 SSL을 추가할 경우 Jenkins 서브넷에 대해 NSG 규칙을 만들어 포트 443을 엽니다. 자세한 내용은 [Azure Portal을 사용하여 가상 머신에 대한 포털을 여는 방법][port443]을 참조하세요.
    > 

-   Jenkins 구성이 교차 사이트 요청 위조를 방지하는지 확인합니다(Jenkins 관리 \> 전역 보안 구성). 이는 Microsoft Jenkins 서버의 기본값입니다.

-   [매트릭스 권한 부여 전략 플러그 인][matrix]을 사용하여 Jenkins 대시보드에 대한 읽기 전용 액세스를 구성합니다.

-   Key Vault를 사용하여 Azure 자산, 파이프라인의 에이전트 및 타사 구성 요소의 비밀을 처리하도록 [Azure 자격 증명][configure-credential] 플러그 인을 설치합니다.

-   사용자, 서비스 및 파이프라인 에이전트가 작업을 수행하는 데 필요한 리소스를 정의하지만 더 이상 필요하지 않은 리소스를 정의하는 보안 프로필을 만듭니다. 이 단계는 보안 설정을 고려할 때 중요합니다.

Jenkins 작업에서는 Azure Container Service와 같은 권한 부여를 요구하는 Azure 서비스에 액세스하는 데 보통 비밀을 요구합니다. [Azure 자격 증명 플러그 인][configure-credential]과 함께 [Key Vault][key-vault]를 사용하여 이러한 비밀은 안전하게 관리합니다. Key Vault를 사용하여 서비스 주체 자격 증명, 암호, 토큰 및 기타 비밀을 저장합니다.

Azure 리소스의 보안 상태를 중앙에서 살펴 보려면 [Azure Security Center][security-center]를 사용합니다. Security Center는 잠재적인 보안 문제를 모니터링하고 배포의 보안 상태에 대한 종합적인 그림을 제공합니다. 보안 센터는 각 Azure 구독을 기준으로 구성됩니다. [Azure Security Center 빠른 시작 가이드][quick-start]에 설명된 것처럼 보안 데이터 수집을 사용하도록 설정합니다. 데이터 수집이 사용되도록 설정되면 Security Center는 해당 구독에서 만든 모든 가상 머신을 자동으로 검색합니다.

Jenkins 서버에는 사용자 고유의 사용자 관리 시스템이 있으며 Jenkins 커뮤니티는 [Azure에서 Jenkins 인스턴스 보안][secure-jenkins]을 위한 모범 사례를 제공합니다. Azure에서 Jenkins를 위한 솔루션 템플릿은 이러한 모범 사례를 구현합니다.

## <a name="manageability-considerations"></a>관리 효율성 고려 사항

리소스 그룹을 사용하여 배포된 Azure 리소스를 구성합니다. 프로덕션 환경 및 개발/테스트 환경을 별도의 리소스 그룹에 배포하여 각 환경의 리소스를 모니터링하고 리소스 그룹별로 청구 비용을 롤업할 수 있습니다. 리소스를 하나의 집합으로 삭제할 수도 있습니다. 이러한 기능은 테스트 배포에서 매우 유용합니다.

Azure는 전체 인프라를 [모니터링 및 진단][monitoring-diag]하는 여러 기능을 제공합니다. CPU 사용량을 모니터링하기 위해 이 아키텍처는 Azure Monitor를 배포합니다. 예를 들어 Azure Monitor를 사용하여 CPU 사용률을 모니터링하고 CPU 사용량이 80%를 초과하는 경우 알림을 보낼 수 있습니다. (높은 CPU 사용량은 Jenkins 서버 VM을 강화하려는 경우가 있음을 나타냅니다.) 또한 VM이 실패하거나 사용할 수 없게 되는 경우 지정된 사용자에게 알릴 수 있습니다.

## <a name="communities"></a>커뮤니티

커뮤니티는 질문에 대답하고 성공적인 배포를 설정하는 데 도움을 줄 수 있습니다. 다음을 고려해 보세요.

-   [Jenkins 커뮤니티 블로그](https://jenkins.io/node/)
-   [Azure 포럼](https://azure.microsoft.com/support/forums/)
-   [Stack Overflow Jenkins](https://stackoverflow.com/tags/jenkins/info)

Jenkins 커뮤니티의 더 많은 모범 사례는 [Jenkins 모범 사례][jenkins-best]를 참조하세요.

## <a name="deploy-the-solution"></a>솔루션 배포

이 아키텍처를 배포하려면 아래 단계에 따라 [Azure에서 Jenkins를 위한 솔루션 템플릿][azure-market]을 설치한 다음 아래 단계에서 모니터링 및 재해 복구를 설정하는 스크립트를 설치합니다.

### <a name="prerequisites"></a>필수 조건

- 이 참조 아키텍처에는 Azure 구독이 필요합니다. 
- Azure 서비스 주체를 만들려면 배포된 Jenkins 서버와 연결된 Azure AD 테넌트에 대한 관리자 권한이 있어야 합니다.
- 이러한 지침에서는 Jenkins 관리자가 참가자 권한 이상을 보유하는 Azure 사용자라고 가정합니다.

### <a name="step-1-deploy-the-jenkins-server"></a>1단계: Jenkins 서버 배포

1.  웹 브라우저에서 [Jenkins의 Azure Marketplace 이미지][azure-market]를 열고 페이지의 왼쪽에서 **지금 가져오기**를 선택합니다.

2.  가격 책정 세부 정보를 검토하고 **계속**을 선택한 후 **만들기**를 선택하여 Azure Portal에서 Jenkins 서버를 구성합니다.

자세한 지침은 [Azure Portal에서 Azure Linux VM에 Jenkins 서버 만들기][create-jenkins]를 참조하세요. 이 참조 아키텍처의 경우 관리자 로그온으로 서버를 시작하고 실행하는 것으로 충분합니다. 그런 다음 다른 여러 서비스를 사용하도록 프로비전할 수 있습니다.

### <a name="step-2-set-up-sso"></a>2단계: SSO 설정

이 단계는 Jenkins 관리자에 의해 실행되며, 이때 Jenkins 관리자는 구독의 Azure AD 디렉터리에 사용자 계정을 포함하고 참가자 역할이 할당되어 있어야 합니다.

Jenkin 서버의 Jenkins 업데이트 센터에서 [Azure AD 플러그 인][configure-azure-ad]을 사용하고 지침에 따라 SSO를 설정합니다.

### <a name="step-3-provision-jenkins-server-with-azure-vm-agent-plugin"></a>3단계: Azure VM 에이전트 플러그 인이 있는 Jenkins 서버 프로비전

이 단계는 Jenkins 관리자에 의해 실행되며 이미 설치된 Azure VM 에이전트 플러그 인을 설정합니다.

[다음 단계에 따라 플러그 인을 구성합니다][configure-agent]. 플러그 인에 대한 서비스 주체 설정에 대한 자습서는 [Azure VM 에이전트를 사용하여 수요에 맞게 Jenkins 배포 규모 조정][scale-agent]을 참조하세요.

### <a name="step-4-provision-jenkins-server-with-azure-storage"></a>4단계: Azure Storage가 있는 Jenkins 서버 프로비전

이 단계는 Jenkins 관리자에 의해 실행되며 이미 설치된 Windows Azure Storage 플러그 인을 설정합니다.

[다음 단계에 따라 플러그 인을 구성합니다][configure-storage].

### <a name="step-5-provision-jenkins-server-with-azure-credential-plugin"></a>5단계: Azure 자격 증명 플러그 인이 있는 Jenkins 서버 프로비전

이 단계는 Jenkins 관리자에 의해 실행되며 이미 설치된 Azure 자격 증명 플러그 인을 설정합니다.

[다음 단계에 따라 플러그 인을 구성합니다][configure-credential].

### <a name="step-6-provision-jenkins-server-for-monitoring-by-the-azure-monitor-service"></a>6단계: Azure Monitor 서비스로 모니터링할 Jenkins 서버 프로비전

Jenkins 서버에 대한 모니터링을 설정하려면 [Azure 서비스의 Azure Monitor에서 메트릭 경고 만들기][create-metric]를 참조하세요.

### <a name="step-7-provision-jenkins-server-with-managed-disks-for-disaster-recovery"></a>7단계: 재해 복구를 위해 Managed Disks로 Jenkins 서버 프로비전

Microsoft Jenkins 제품 그룹은 Jenkins 상태를 저장하는 데 사용되는 관리 디스크를 작성하는 재해 복구 스크립트를 작성했습니다. 서버가 다운되면 최신 상태로 복원할 수 있습니다.

[GitHub][disaster]에서 재해 복구 스크립트를 다운로드하여 실행하십시오.

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
