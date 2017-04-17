---
title: Deploy a high availability network virtual appliances
description: How to deploy network virtual appliances in high availability.
author: telmosampaio
ms.service: guidance
ms.topic: article
ms.date: 12/06/2016
ms.author: pnp

pnp.series.title: Network DMZ
pnp.series.prev: secure-vnet-dmz
cardTitle: Deploy highly available network virtual appliances
---
# 고가용성 네트워크 가상 어플라이언스 배포

이 문서는 Azure에서 고가용성을 위한 일련의 네트워크 가상 어플라이언스(NVA)를 배포하는 방법을 보여줍니다. NVA는 DMZ로 불리는 경계 네트워크로부터 다른 네트워크나 서브넷으로의 네트워크 트래픽 흐름을 제어하기 위해 주로 사용됩니다. Azure에서 DMZ를 구현하는 방법에 대한 자세한 내용은 [Microsoft 클라우드 서비스 및 네트워크 보안][cloud-security]을 참조하시기 바랍니다. 이 문서에는 수신 전용, 송신 전용, 송수신용 등 세가지 참조 아키텍처가 포함됩니다.

**선행 요구 사항:** 이 문서는 Azure 네트워킹, [Azure 부하 분산 장치][lb-overview], [사용자 정의 루트][udr-overview] (UDR)에 대한 기본적인 이해를 전제로 합니다.  


## 아키텍처 다이어그램

NVA는 다양한 아키텍처의 DMZ에 배포할 수 있습니다. 예를 들어, 다음 그림은 수신을 위한 [단일 NVA][nva-scenario]의 사용을 보여줍니다. 

![[0]][0]

이 아키텍처에서 NVA는 모든 들어오고 나가는 네트워크 트래픽을 검사하여 네트워크 보안 규칙을 만족하는 트래픽만을 통과시킴으로써 안전한 네트워크 경계를 제공합니다. 그러나 모든 네트워크 트래픽이 NVA를 거쳐야 하므로 NVA는 네트워크 내 단일 장애 지점이 됩니다. NVA에 장애가 발생할 경우 네트워크 트래픽을 위한 다른 경로가 존재하지 않고 모든 백엔드 서브넷은 이용할 수 없게 됩니다. 

NVA의 가용성을 높이려면 가용성 집합에 여러 NVA를 배포합니다. 

다음 아키텍처는 고가용성 NVA에 필요한 리소스와 구성을 보여줍니다. 

| 솔루션 | 이점 | 고려사항 |
| --- | --- | --- |
| [수신용 레이어7 NVA][ingress-with-layer-7] |모든 NVA 노드는 액티브 상태 |연결을 종료하고 SNAT을 사용할 수 있는 NVA가 필요</br> 인터넷과 Azure로부터 오는 트래픽을 위한 별도의 NVA 집합이 필요 </br> Azure 외부에서 오는 트래픽에 대해서만 사용 가능 |
| [송신용 레이어7 NVA][egress-with-layer-7] |모든 NVA 노드는 액티브 상태 | 연결을 종료하고 소스 네트워크 주소 변환(SNAT)을 수행할 수 있는 NVA가 필요
| [송수신용 레이어7 NVA][ingress-egress-with-layer-7] |모든 노드는 액티브 상태.<br/>Azure에서 발생한 트래픽을 처리. |연결을 종료하고 SNAT을 사용할 수 있는 NVA가 필요.<br/>인터넷과 Azure로부터 오는 트래픽을 위한 별도의 NVA 집합이 필요. |
| [PIP-UDR 변경][pip-udr-switch] |단일 NVA 집합으로 모든 트래픽 처리.<br/>(포트 규칙 제한 없이) 모든 트래픽 처리. |액티브-패시브.<br/>장애조치(failover) 프로세스가 필요. |

## 수신용 레이어7 NVA 

다음 그림은 인터넷 연결 부하 분산 장치 뒤에 수신 DMZ를 구현하는 고가용성 아키텍처를 보여줍니다. 이 아키텍처는 HTTP나 HTTPS와 같은 레이어7 트래픽을 위한 Azure 워크로드에 대한 연결성을 제공하도록 설계된 아키텍처입니다. 

![[1]][1]

이 아키텍처의 장점은 모든 NVA가 액티브 상태이고, 하나의 NVA에 장애가 발생하면 부하 분산 장치가 네트워크 트래픽을 다른 NVA로 보내준다는 점입니다. 두 NVA 모두 트래픽을 내부 부하 분산 장치로 라우팅하므로 하나의 NVA가 액티브 상태를 유지하기만 하면 트래픽은 중단 없이 흐르게 됩니다. 이 NVA들은 웹 계층 VM을 위해 의도된 SSL 트래픽을 종료해야 합니다. 온-프레미스를 처리하기 위해 이 NVA들을 확장해서는 안 됩니다. 온-프레미스 트래픽에 대해서는 자체 네트워크 루트를 가진 다른 전용 NVA 집합이 요구되기 때문입니다.  

> [!참고]
> 이 아키텍처는 [Azure와 온-프레미스 데이터 센터 간 DMZ][dmz-on-prem] 참조 아키텍처와 [Azure와 인터넷 간 DMZ][dmz-internet] 참조 아키텍처에 사용됩니다. 이 참조 아키텍처들은 각각 실제로 사용할 수 있는 배포 솔루션을 포함합니다. 자세한 내용은 아래 링크를 클릭하여 확인하시기 바랍니다. 

## 송신용 레이어7 NVA

앞서 소개한 아키텍처는 Azure 워크로드에서 발생하는 요청을 위한 송신 DMZ를 제공하도록 확장할 수 있습니다.  다음 아키텍처는 HTTP나 HTTPS와 같은 레이어7 트래픽을 위한 DMZ 내 NVA의 고가용성을 제공하도록 설계된 아키텍처입니다. 

![[2]][2]

이 아키텍처는 Azure에서 발생하는 모든 트래픽을 내부 부하 분산 장치로 라우팅합니다. 부하 분산 장치는 나가는 요청을 하나의 집합 내에 있는 NVA들 사이에 분산시킵니다. 이 NVA들은 개별 공용 IP 주소를 사용하여 트래픽을 인터넷으로 보냅니다.  

> [!참고]
> 이 아키텍처는 [Azure와 온-프레미스 데이터 센터 간 DMZ][dmz-on-prem] 참조 아키텍처와 [Azure와 인터넷 간 DMZ][dmz-internet] 참조 아키텍처에 사용됩니다. 이 참조 아키텍처들은 각각 실제로 사용할 수 있는 배포 솔루션을 포함합니다. 자세한 내용은 아래 링크를 클릭하여 확인하시기 바랍니다. 

## 송수신용 레이어7 NVA 

앞의 두 아키텍처의 경우 수신 DMZ와 송신 DMZ가 각각 존재했습니다. 다음 아키텍처는 HTTP나 HTTPS와 같은 레이어7 트래픽의 송수신에 모두 사용되는 DMZ를 생성하는 방법을 보여줍니다.

![[4]][4]

이 아키텍처에서 네트워크 가상 어플라이언스(NVA)는 응용 프로그램 게이트웨이로부터 오는 요청을 처리합니다. 또한 NVA는 부하 분산 장치의 백엔드 풀의 워크로드 VM으로부터 나가는 요청도 처리합니다. 들어오는 트래픽은 응용 프로그램 게이트웨이를 통해 라우팅되고 나가는 트래픽은 부하 분산 장치를 통해 라우팅되므로 NVA는 세션 선호도를 유지하는 역할을 합니다. 즉, 응용 프로그램 게이트웨이는 수신 및 송신 요청의 매핑을 유지하므로 정확한 응답을 요청자에게 전달할 수 있습니다. 그러나 내부 부하 분산 장치는 응용 프로그램 게이트웨이 매핑에 액세스할 수 있고 자체 로직을 통해 NVA에 응답을 전송합니다. 부하 분산 장치는 처음에 응용 프로그램 게이트웨이로부터 요청을 받지 않은 NVA에 응답을 전송할 수 있습니다. 이 경우 NVA들이 서로 응답을 통신하고 전송하여 정확한 NVA가 응답을 응용 프로그램 게이트웨이로 전달할 수 있어야 합니다. 

## 레이어4 NVA PIP-UDR 변경

다음 아키텍처는 액티브 NVA와 패시브 NVA가 각각 하나씩 있는 아키텍처입니다. 이 아키텍처는 레이어4 트래픽의 송수신을 모두 처리합니다. 

![[3]][3]

이 아키텍처는 이 문서에서 소개된 첫 번째 아키텍처와 유사합니다. 첫 번째 아키텍처는 들어오는 레이어4 요청을 수용하고 필터링하는 단일 NVA를 포함합니다. 이 아키텍처의 경우 가용성을 높이기 위해 또 다른 패시브 NVA를 추가했습니다. 액티브 NVA에 장애가 발생하면 패시브 NVA가 액티브 상태가 되고 UDR과 PIP는 액티브 상태가 된 NVA의 네트워크 인터페이스를 가리키도록 변경됩니다. 이러한 UDR 및 PIP의 변경은 수동으로 또는 자동화된 프로세스를 통해 수행할 수 있습니다. 자동화된 프로세스는 주로 디먼 또는 Azure에서 실행되는 다른 모니터링 서비스입니다. 이 프로세스는 액티브 NVA의 상태 프로브에 쿼리를 수행하고 NVA 장애 감지 시 UDR 및 PIP 변경을 수행합니다. 

앞의 그림은 고가용성 디먼을 제공하는 [ZooKeeper][zookeeper] 클러스터를 보여줍니다. ZooKeeper 클러스터 내에서 노드 쿼럼으로 리더를 선출합니다. 리더에 장애가 발생할 경우, 나머지 노드들이 새 리더를 선출합니다. 이 아키텍처에서 리더 노드는 NVA의 상태 끝점에 쿼리를 수행하는 디먼을 실행합니다. NVA가 상태 프로브 응답에 실패할 경우, 디먼은 패시브 NVA를 액티브 상태로 변경합니다. 그런 다음 디먼이 Azure REST API를 호출하여 실패한 NVA로부터 PIP를 제거하여 액티브 상태로 변경된 NVA에 연결합니다. 다시 디먼은 UDR이 새로운 액티브 NVA의 내부 IP 주소를 가리키도록 UDR을 변경합니다. 

> [!참고]
> ZooKeeper 노드를 해당 NVA를 포함하는 루트를 사용해서만 액세스할 수 있는 서브넷에 포함시키면 안 됩니다. 그렇지 않으면 NVA 장애 시 ZooKeeper 노드에 액세스할 수 없습니다. 무슨 이유로든 디먼에 장애가 발생한다면 문제 진단을 위해 ZooKeepr 노드에 액세스할 수 없게 됩니다. 

<!--### Solution Deployment-->

<!-- instructions for deploying this solution here --> 

## 다음 단계
* 레이어7 NVA를 사용하여 [Azure와 온-프레미스 데이터센터 간 DMZ를 구현][dmz-on-prem]하는 방법에 대해 알아보시기 바랍니다.
* 레이어7 NVA를 사용하여 [Azure와 인터넷 간 DMZ를 구현][dmz-internet]하는 방법에 대해 알아보시기 바랍니다.

<!-- links -->
[cloud-security]: /azure/best-practices-network-security
[dmz-on-prem]: ./secure-vnet-hybrid.md
[dmz-internet]: ./secure-vnet-dmz.md
[egress-with-layer-7]: #egress-with-layer-7-nvas
[ingress-with-layer-7]: #ingress-with-layer-7-nvas
[ingress-egress-with-layer-7]: #ingress-egress-with-layer-7-nvas
[lb-overview]: /azure/load-balancer/load-balancer-overview/
[nva-scenario]: /azure/virtual-network/virtual-network-scenario-udr-gw-nva/
[pip-udr-switch]: #pip-udr-switch-with-layer-4-nvas
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview/
[zookeeper]: https://zookeeper.apache.org/

<!-- images -->
[0]: ./images/nva-ha/single-nva.png "Single NVA architecture"
[1]: ./images/nva-ha/l7-ingress.png "Layer 7 ingress"
[2]: ./images/nva-ha/l7-ingress-egress.png "Layer 7 egress"
[3]: ./images/nva-ha/active-passive.png "Active-Passive cluster"
[4]: ./images/nva-ha/l7-ingress-egress-ag.png
