---
title: 컨테이너 기반 작업에 대한 CI/CD 파이프라인
description: Jenkins, Azure Container Registry, Azure Kubernetes Service, Cosmos DB 및 Grafana를 사용하는 Node.js 웹앱용 DevOps 파이프라인을 구축하는 데 입증된 시나리오입니다.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: d9f6571234a0c3e67a233cfda1a37f6fb32929a3
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060764"
---
# <a name="cicd-pipeline-for-container-based-workloads"></a>컨테이너 기반 작업에 대한 CI/CD 파이프라인

이 예제 시나리오는 컨테이너 및 DevOps 흐름을 사용하여 응용 프로그램 개발을 현대화하려는 비즈니스에 적용할 수 있습니다. 이 시나리오에서는 Jenkins에서 Node.js 웹앱을 Azure Container Registry 및 Azure Kubernetes Service에 구축하고 배포합니다. 전역 분산 데이터베이스 계층에는 Azure Cosmos DB가 사용됩니다. Azure Monitor는 응용 프로그램 성능을 모니터링하고 문제를 해결하기 위해 Grafana 인스턴스 및 대시보드와 통합됩니다.

예제 응용 프로그램 시나리오에는 자동화된 개발 환경 제공, 새 코드 커밋의 유효성 검사 및 준비 또는 프로덕션 환경으로 새 배포 푸시가 포함됩니다. 일반적으로 기업에서는 응용 프로그램과 업데이트를 수동으로 빌드 및 컴파일하고, 대규모의 모놀리식 코드베이스를 유지해야 했습니다. CI(지속적인 통합) 및 CD(지속적인 배포)를 사용하는 응용 프로그램 개발에 대한 현대적인 접근 방식을 사용하면, 서비스를 더 빠르게 빌드, 테스트 및 배포할 수 있습니다. 이러한 현대적인 접근 방식을 통해 고객에게 응용 프로그램과 업데이트를 더 빨리 릴리스하고, 변화하는 비즈니스 요구 사항에 더 민첩하게 대응할 수 있습니다.

Azure Kubernetes Service, Container Registry 및 Cosmos DB와 같은 Azure 서비스를 사용하면, 최신의 응용 프로그램 개발 기술과 도구를 사용하여 고가용성 구현 프로세스를 간소화할 수 있습니다.

## <a name="related-use-cases"></a>관련 사용 사례

이 시나리오에 적합한 사용 사례는 다음과 같습니다.

* 응용 프로그램 개발 사례를 마이크로 서비스, 컨테이너 기반 접근 방식으로 현대화
* 응용 프로그램 개발 및 배포 수명 주기 가속화
* 유효성 검사를 위해 테스트 또는 수용 환경에 대한 배포 자동화

## <a name="architecture"></a>아키텍처

![Jenkins, Azure Container Registry 및 Azure Kubernetes Service를 사용하는 DevOps 시나리오와 관련된 Azure 구성 요소 아키텍처에 대한 개요][architecture]

이 시나리오에서는 Node.js 웹 응용 프로그램 및 데이터베이스 백 엔드용 DevOps 파이프라인에 대해 설명합니다. 시나리오를 통한 데이터 흐름은 다음과 같습니다.

1. 개발자는 Node.js 웹 응용 프로그램 소스 코드를 변경합니다.
2. 코드 변경은 GitHub와 같은 소스 제어 리포지토리에 커밋됩니다.
3. CI 프로세스를 시작하기 위해 GitHub 웹후크에서 Jenkins 프로젝트 빌드를 시작합니다.
4. Jenkins 빌드 작업은 Azure Kubernetes Service의 동적 빌드 에이전트를 사용하여 컨테이너 빌드 프로세스를 수행합니다.
5. 컨테이너 이미지는 소스 제어의 코드에서 만들어진 다음, Azure Container Registry로 푸시됩니다.
6. Jenkins는 CD를 통해 업데이트된 이 컨테이너 이미지를 Kubernetes 클러스터에 배포합니다.
7. Node.js 웹 응용 프로그램은 Azure Cosmos DB를 백 엔드로 사용합니다. Cosmos DB와 Azure Kubernetes Service는 모두 Azure Monitor에 메트릭을 보고합니다.
8. Grafana 인스턴스는 Azure Monitor의 데이터를 기반으로 하여 응용 프로그램 성능의 시각적 대시보드를 제공합니다.

### <a name="components"></a>구성 요소

* [Jenkins][jenkins]는 Azure 서비스와 통합하여 CI 및 CD를 수행할 수 있게 하는 오픈 소스 자동화 서버입니다. 이 시나리오에서 Jenkins는 소스 제어에 대한 커밋에 따라 새 컨테이너 이미지를 만들도록 오케스트레이션하고, Azure Container Registry에 해당 이미지를 푸시한 다음, Azure Kubernetes Service에서 응용 프로그램 인스턴스를 업데이트합니다.
* [Azure Linux Virtual Machines][azurevm-docs]는 Jenkins 및 Grafana 인스턴스를 실행하는 데 사용됩니다.
* [Azure Container Registry][azureacr-docs]는 Azure Kubernetes Service 클러스터에서 사용되는 컨테이너 이미지를 저장하고 관리합니다. 이미지는 안전하게 저장되며, Azure 플랫폼을 통해 다른 지역으로 복제하여 배포 시간을 단축할 수 있습니다.
* [Azure Kubernetes Service][azureaks-docs]는 컨테이너 오케스트레이션에 대한 전문 지식이 없어도 컨테이너화된 응용 프로그램을 배포하고 관리할 수 있는 관리되는 Kubernetes 플랫폼입니다. 호스팅되는 Kubernetes 서비스인 Azure는 상태 모니터링 및 유지 관리 같은 중요 작업을 처리합니다.
* [Azure Cosmos DB][azurecosmosdb-docs]는 요구 사항에 맞게 다양한 데이터베이스 및 일관성 모델 중에서 선택할 수 있는 전역으로 분산된 다중 모델 데이터베이스입니다. Cosmos DB를 사용하면 데이터를 전역으로 복제할 수 있으며, 배포 및 구성할 클러스터 관리 또는 복제 구성 요소가 없습니다.
* [Azure Monitor][azuremonitor-docs]는 성능을 추적하고, 보안을 유지하며, 추세를 파악하는 데 도움이 됩니다. Monitor에서 얻은 메트릭은 Grafana와 같은 다른 리소스 및 도구에서 사용할 수 있습니다.
* [Grafana][grafana]는 메트릭을 쿼리하고, 시각화하고, 경고하고, 이해할 수 있는 오픈 소스 솔루션입니다. Azure Monitor용 데이터 원본 플러그 인을 사용하면 Grafana가 Azure Kubernetes Service에서 실행되고 Cosmos DB를 사용하는 응용 프로그램의 성능을 모니터링하는 시각적 대시보드를 만들 수 있습니다.

### <a name="alternatives"></a>대안

* [Visual Studio Team Services][vsts] 및 Team Foundation Server를 사용하면 모든 응용 프로그램에 대한 지속적인 통합(CI), 테스트 및 배포(CD) 파이프라인을 구현할 수 있습니다.
* [Kubernetes][kubernetes]는 클러스터에 대해 더 자세히 제어하려는 경우에 관리 서비스 대신 Azure VM에서 직접 실행할 수 있습니다.
* [Service Fabric][service-fabric]은 AKS를 대체할 수 있는 대체 컨테이너 오케스트레이터입니다.

## <a name="considerations"></a>고려 사항

### <a name="availability"></a>가용성

응용 프로그램 성능을 모니터링하고 문제를 보고하기 위해 이 시나리오에서는 시각적 대시보드에서 Azure Monitor와 Grafana를 결합합니다. 이러한 도구를 사용하면 코드를 업데이트해야 하는 성능 문제를 모니터링하고 문제를 해결할 수 있습니다. 그런 다음, CI/CD 파이프라인을 통해 모두 배포할 수 있습니다.

Azure Kubernetes Service 클러스터의 일부인 부하 분산 장치는 응용 프로그램을 실행하는 하나 이상의 컨테이너(포드)에 응용 프로그램 트래픽을 분산시킵니다. Kubernetes에서 컨테이너화된 응용 프로그램을 실행하는 이 접근 방식은 고객에게 고가용성 인프라를 제공합니다.

다른 가용성 항목에 대해서는 아키텍처 센터에서 사용할 수 있는 [가용성 검사 목록][availability]을 참조하세요.

### <a name="scalability"></a>확장성

Azure Kubernetes Service를 사용하면 응용 프로그램의 요구 사항에 맞게 클러스터 노드 수를 크기 조정할 수 있습니다. 응용 프로그램이 증가함에 따라 서비스를 실행하는 Kubernetes 노드의 수를 확장할 수 있습니다.

응용 프로그램 데이터는 전역으로 크기 조정할 수 있는 전역 분산형 다중 모델 데이터베이스인 Azure Cosmos DB에 저장됩니다. Cosmos DB는 기존 데이터베이스 구성 요소와 마찬가지로 인프라의 크기를 조정하기 위한 요구 사항을 추상화하고, 고객의 요구를 충족하기 위해 Cosmos DB를 전역으로 복제하도록 선택할 수 있습니다.

다른 확장성 항목에 대해서는 아키텍처 센터에서 사용할 수 있는 [확장성 검사 목록][scalability]을 참조하세요.

### <a name="security"></a>보안

공격 공간을 최소화하기 위해 이 시나리오에서는 Jenkins VM 인스턴스가 HTTP를 통해 노출되지 않습니다. Jenkins와 상호 작용해야 하는 모든 관리 작업의 경우 로컬 컴퓨터에서 SSH 터널을 사용하여 보안 원격 연결을 만듭니다. Jenkins 및 Grafana VM 인스턴스에는 SSH 공개 키 인증만 허용됩니다. 암호 기반 로그인은 사용할 수 없습니다. 자세한 내용은 [Azure에서 Jenkins 서버 실행](../../reference-architectures/jenkins/index.md)을 참조하세요.

자격 증명과 권한을 분리하기 위해 이 시나리오에서는 전용 Azure AD(Active Directory) 서비스 사용자를 사용합니다. 이 서비스 사용자에 대한 자격 증명은 Jenkins에서 보안 자격 증명 개체로 저장되어 스크립트 또는 빌드 파이프라인 내에서 직접 노출되거나 볼 수 없습니다.

보안 솔루션 설계에 대한 일반적인 지침은 [Azure 보안 설명서][security]를 참조하세요.

### <a name="resiliency"></a>복원력

이 시나리오에서는 응용 프로그램에 Azure Kubernetes Service를 사용합니다. Kubernetes에는 문제가 있는 경우 컨테이너(포드)를 모니터링하고 다시 시작하는 복원력 있는 구성 요소가 기본적으로 제공됩니다. 여러 Kubernetes 노드를 실행하는 것과 결합하여 응용 프로그램에서 사용할 수 없는 노드 또는 노드를 허용할 수 있습니다.

복원력 있는 솔루션 설계에 대한 일반적인 지침은 [복원력 있는 Azure 응용 프로그램 디자인][resiliency]을 참조하세요.

## <a name="deploy-the-scenario"></a>시나리오 배포

**필수 조건.**

* 기존 Azure 계정이 있어야 합니다. Azure 구독이 아직 없는 경우 시작하기 전에 [무료 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.
* SSH 공개 키 쌍이 필요합니다. 공개 키 쌍을 만드는 방법에 대한 단계는 [Linux VM용 SSH 키 쌍 만들기 및 사용][sshkeydocs]을 참조하세요.
* 서비스 및 리소스의 인증에는 Azure AD(Active Directory) 서비스 사용자가 필요합니다. 필요한 경우 [az ad sp create-for-rbac][createsp]를 사용하여 서비스 사용자를 만듭니다.

    ```azurecli-interactive
    az ad sp create-for-rbac --name myDevOpsScenario
    ```

    이 명령의 출력에서 나온 *appId* 및 *password*를 적어 둡니다. 시나리오를 배포할 때 이러한 값을 템플릿에 제공합니다.

Azure Resource Manager 템플릿을 사용하여 이 시나리오를 배포하려면 다음 단계를 수행합니다.

1. **Azure에 배포** 단추를 클릭합니다.<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fdevops-with-aks%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Azure Portal에서 템플릿 배포가 열릴 때까지 기다린 후에 다음 단계를 수행합니다.
   * 리소스 그룹 **새로 만들기**를 선택한 다음, 텍스트 상자에서 이름(예: *myAKSDevOpsScenario*)을 입력합니다.
   * **위치** 드롭다운 상자에서 지역을 선택합니다.
   * `az ad sp create-for-rbac` 명령에서 서비스 사용자 앱 ID 및 암호를 입력합니다.
   * Jenkins 인스턴스 및 Grafana 콘솔에 대한 사용자 이름과 보안 암호를 제공합니다.
   * Linux VM에 대한 로그인을 보호하기 위한 SSH 키를 제공합니다.
   * 사용 약관을 검토한 다음, **위에 명시된 사용 약관에 동의함**을 선택합니다.
   * **구매** 단추를 선택합니다.

배포가 완료되는 데 15-20분이 걸릴 수 있습니다.

## <a name="pricing"></a>가격

이 시나리오를 실행하는 데 들어가는 비용을 알아보기 위해 모든 서비스가 비용 계산기에서 미리 구성됩니다. 특정 사용 사례에 대한 가격이 변경되는 정도를 확인하려면 필요한 트래픽에 맞게 적절한 변수를 변경합니다. 필요한 트래픽과 일치해야 합니다.

저장할 컨테이너 이미지 및 Kubernetes 노드의 수를 기준으로 다음 세 가지 샘플 비용 프로필을 제공하여 응용 프로그램을 실행했습니다.

* [소량][small-pricing]: 매월 1,000개의 컨테이너 빌드와 관련이 있습니다.
* [중간][medium-pricing]: 매월 100,000개의 컨테이너 빌드와 관련이 있습니다.
* [대량][large-pricing]: 매월 1,000,000개의 컨테이너 빌드와 관련이 있습니다.

## <a name="related-resources"></a>관련 리소스

이 시나리오에서는 Azure Container Registry와 Azure Kubernetes Service를 사용하여 컨테이너 기반 응용 프로그램을 저장하고 실행했습니다. Azure Container Instances는 오케스트레이션 구성 요소를 프로비전하지 않고 컨테이너 기반 응용 프로그램을 실행하는 데에도 사용할 수 있습니다. 자세한 내용은 [Azure Container Instances 개요][azureaci-docs]를 참조하세요.

<!-- links -->
[architecture]: ./media/devops-with-aks/architecture-devops-with-aks.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[azureaci-docs]: /azure/container-instances/container-instances-overview
[azureacr-docs]: /azure/container-registry/container-registry-intro
[azurecosmosdb-docs]: /azure/cosmos-db/introduction
[azureaks-docs]: /azure/aks/intro-kubernetes
[azuremonitor-docs]: /azure/monitoring-and-diagnostics/monitoring-overview
[azurevm-docs]: /azure/virtual-machines/linux/overview
[createsp]: /cli/azure/ad/sp#az-ad-sp-create
[grafana]: https://grafana.com/
[jenkins]: https://jenkins.io/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sshkeydocs]: /azure/virtual-machines/linux/mac-create-ssh-keys
[vsts]: /vsts/?view=vsts
[kubernetes]: https://kubernetes.io/
[service-fabric]: /azure/service-fabric/

[small-pricing]: https://azure.com/e/841f0a75b1ea4802ba1ac8f7918a71e7
[medium-pricing]: https://azure.com/e/eea0e6d79b4e45618a96d33383ec77ba
[large-pricing]: https://azure.com/e/3faab662c54c473da55a1e93a27e0e64