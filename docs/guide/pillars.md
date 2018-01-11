---
title: "소프트웨어 품질 핵심 요소"
description: "확장성, 가용성, 복원력, 관리 및 보안이라는 5가지 소프트웨어 품질 핵심 요소를 설명합니다."
author: MikeWasson
ms.openlocfilehash: 78e613368a07718f5923d619ace335d399b0cc80
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
# <a name="pillars-of-software-quality"></a>소프트웨어 품질 핵심 요소 

성공적인 클라우드 응용 프로그램은 확장성, 가용성, 복원력, 관리 및 보안이라는 5가지 소프트웨어 품질 핵심 요소에 중점을 둡니다.

| 핵심 요소 | 설명 |
|--------|-------------|
| 확장성 | 증가된 부하를 처리하는 시스템의 기능입니다. |
| Availability | 시스템이 기능하고 작동하는 시간의 비율입니다. |
| 복원력 | 오류를 복구하여 계속 작동하는 시스템 기능입니다. |
| 관리 | 프로덕션에서 시스템을 실행하는 작업 프로세스입니다. |
| 보안 | 위협으로부터 응용 프로그램 및 데이터를 보호합니다. |

## <a name="scalability"></a>확장성

확장성은 증가된 부하를 처리하는 시스템의 기능입니다. 응용 프로그램은 다음 두 가지 주요 방법으로 확장될 수 있습니다. 예를 들어, 수직적 크기 조정(*강화*)이란 더 큰 VM 크기를 사용하여 리소스의 용량을 늘리는 것입니다. 수평적 크기 조정(*확장*)이란 VM 또는 데이터베이스 복제본과 같은 리소스의 새 인스턴스를 추가하는 것입니다. 

수평적 크기 조정에는 수직적 크기 조정에 비해 많은 장점이 있습니다.

- 진정한 클라우드 규모입니다. 단일 노드에서 가능하지 않은 규모에 이르는 수백 또는 수천 개의 노드에서 실행되도록 응용 프로그램을 디자인할 수 있습니다.
- 수평적 크기 조정은 탄력적입니다. 부하가 증가하면 더 많은 인스턴스를 추가하거나 더 조용한 기간 동안 제거할 수 있습니다.
- 일정에 따라 또는 부하의 변화에 대한 응답으로 확장이 자동으로 트리거될 수 있습니다. 
- 확장은 강화보다 저렴할 수 있습니다. 여러 작은 VM을 실행하면 큰 단일 VM보다 비용이 저렴할 수 있습니다. 
- 수평적 크기 조정은 중복성을 추가하여 복원력을 향상할 수도 있습니다. 인스턴스가 중지되면 응용 프로그램이 계속 실행됩니다.

수직적 크기 조정의 장점은 응용 프로그램을 변경하지 않고도 수행할 수 있다는 점입니다. 하지만 특정 시점에 더 이상을 강화할 수 없는 제한에 도달할 수 있습니다. 이 시점에서 추가 크기 조정은 수평적이어야 합니다. 

시스템에 수평적 크기 조정이 포함되어야 합니다. 예를 들어 VM을 부하 분산 장치 뒤에 배치하여 확장할 수 있습니다. 그러나 풀의 각 VM은 모든 클라이언트 요청을 처리할 수 있어야 합니다. 따라서 응용 프로그램이 상태 비저장이거나 상태를 외부적으로(즉, 분산된 캐시에서) 저장해야 합니다. PaaS 관리 서비스는 수평적 크기 조정 및 자동 크기 조정이 기본 제공됩니다. 이러한 서비스를 크기 조정하는 용이성이 PaaS 서비스를 사용하는 주요 이점입니다.

하지만 더 많은 인스턴스를 추가하는 것은 응용 프로그램의 크기 조정이 아닙니다. 다른 곳에 병목 상태를 푸시할 수 있습니다. 예를 들어 웹 프런트 엔드의 크기를 조정하여 추가 클라이언트 요청을 처리하는 경우 데이터베이스에서 잠금 경합을 트리거할 수 있습니다. 그러면 데이터베이스에 대한 처리량을 높일 수 있도록 낙관적 동시성 또는 분할 데이터와 같은 추가 조치를 사용해야 합니다.

항상 성능 및 부하 테스트를 구축하여 이러한 잠재적인 병목 상태를 찾습니다. 데이터베이스와 같은 시스템의 상태 저장 파트는 가장 일반적인 병목 현상의 원인이며 수평적으로 규모를 조정하도록 신중하게 디자인해야 합니다. 하나의 병목 상태를 확인하면 다른 곳에서도 병목 상태를 발견할 수 있습니다.

[확장성 검사 목록][scalability-checklist]을 사용하여 확장성 관점에서 디자인을 검토합니다.

### <a name="scalability-guidance"></a>확장성 지침

- [확장성 및 성능을 위한 디자인 패턴][scalability-patterns]
- 모범 사례: [자동 크기 조정][autoscale], [배경 작업][background-jobs], [캐싱][caching], [CDN][cdn], [데이터 분할][data-partitioning]

## <a name="availability"></a>Availability

가용성은 시스템이 기능하고 작동하는 시간의 비율입니다. 일반적으로 가동 시간의 백분율로 측정됩니다. 응용 프로그램 오류, 인프라 문제 및 시스템 부하는 모두 가용성을 저하시킬 수 있습니다. 

클라우드 응용 프로그램에는 예상되는 가용성 및 가용성을 측정하는 방법을 명확하게 정의하는 SLO(서비스 수준 목표)이 있어야 합니다. 가용성을 정의할 때 중요 경로를 확인합니다. 웹 프런트 엔드는 클라이언트 요청을 처리할 수 있지만 모든 트랜잭션이 데이터베이스에 연결할 수 없기 때문에 실패하는 경우 사용자는 응용 프로그램을 사용할 수 없습니다. 

가용성은 "9" &mdash;라는 측면에서 설명합니다. 예를 들어 "4개의 9"는 99.99%의 가동 시간을 가리킵니다. 다음 표는 다양한 가용성 수준에서 잠재적인 누적 가동 중지 시간을 보여줍니다.

| % 작동 시간 | 주간 가동 중지 시간 | 월간 가동 중지 시간 | 연간 가동 중지 시간 |
|----------|-------------------|--------------------|-------------------|
| 99% | 1.68시간 | 7.2시간 | 3.65일 |
| 99.9% | 10분 | 43.2분 | 8.76시간 |
| 99.95% | 5분 | 21.6분 | 4.38시간 |
| 99.99% | 1분 | 4.32분 | 52.56분 |
| 99.999% | 6초 | 26초 | 5.26분 |

99% 가동 시간은 주당 거의 2시간 동안 서비스가 중단된다는 의미일 수 있습니다. 많은 응용 프로그램, 특히 소비자 지향 응용 프로그램의 경우 허용 가능한 SLO가 아닙니다. 반면에 5개의 9(99.999%)는 *일년*에 5분 이내의 가동 중지 시간을 의미합니다. 가동 중단을 감시하여 문제를 해결하는 것은 어려운 문제입니다. 가용성을 많이 높이려면(99.99% 이상) 오류를 복구하는 수동 개입을 사용할 수 없습니다. 응용 프로그램은 자체 진단이며 자동 복구여야 합니다. 여기서는 복원력이 중요합니다.

Azure에서 Service Level Agreement(서비스 수준 약정)는 작동 시간 및 연결에 대한 Microsoft의 정책을 설명합니다. 특정 서비스의 SLA가 99.95%라는 것은 시간의 99.95% 동안 서비스를 사용할 수 있다는 의미입니다.

응용 프로그램은 종종 여러 서비스를 사용합니다. 일반적으로 서비스에 가동 중지 시간이 있을 확률은 독립적입니다. 예를 들어 응용 프로그램은 각각 99.9% SLA를 가진 두 개의 서비스에 따라 달라집니다. 두 서비스의 복합 SLA는 99.9% &times; 99.9% &asymp; 99.8% 또는 각각의 서비스 자체보다 약간 짧습니다. 

[가용성 검사 목록][availability-checklist]을 사용하여 가용성 관점에서 디자인을 검토합니다.

### <a name="availability-guidance"></a>가용성 지침

- [가용성을 위한 디자인 패턴][availability-patterns]
- 모범 사례: [자동 크기 조정][autoscale], [배경 작업][background-jobs]

## <a name="resiliency"></a>복원력

복원력은 오류를 복구하여 계속 작동하는 시스템 기능입니다. 복원력의 목표는 오류가 발생한 후에 응용 프로그램을 완전히 작동하는 상태로 되돌리기 위한 것입니다. 복원력은 가용성과 밀접한 관련이 있습니다.

기존의 응용 프로그램 개발 시 MTBF(오류 간 평균 시간)을 단축하는 데 집중했습니다. 시스템이 실패하지 않도록 노력했습니다. 여러 가지 요인으로 인해 클라우드 컴퓨팅에서 다른 태도가 필요합니다.

- 분산 시스템이 복잡하면 한 지점의 오류는 시스템 전체에서 잠재적으로 중첩될 수 있습니다.
- 클라우드 환경의 비용은 상용 하드웨어를 사용하는 내내 낮은 수준으로 유지됩니다. 따라서 하드웨어 오류가 종종 발생될 수 있습니다. 
- 응용 프로그램은 종종 외부 서비스를 사용합니다. 여기에서는 일시적으로 대량 사용자를 사용할 수 없거나 제한하게 됩니다. 
- 오늘날 사용자는 응용 프로그램을 오프라인으로 전환하지 않고 언제든지 사용 가능합니다.

이러한 요인으로 인해 모두 클라우드 응용 프로그램은 종종 오류가 발생하고 여기서 복구하도록 디자인되어야 합니다. Azure에는 플랫폼에 기본 제공된 여러 복원력 기능이 있습니다. 예를 들면 다음과 같습니다. 

- Azure Storage, SQL Database 및 Cosmos DB는 모두 지역 내 및 지역 간에 기본 제공 데이터 복제를 제공합니다.
- Azure Managed Disks는 하드웨어 오류의 영향을 제한하도록 여러 저장소 배율 단위에 자동으로 배치됩니다.
- 가용성 집합의 VM은 여러 장애 도메인에 분산됩니다. 장애 도메인은 공통된 전원 및 네트워크 스위치를 공유하는 VM 그룹입니다. 장애 도메인에 VM을 분산하면 물리적 하드웨어 오류, 네트워크 가동 중단 또는 전원 중단의 영향을 제한합니다.

즉, 응용 프로그램에 복원력을 빌드해야 합니다. 아키텍처의 모든 수준에서 복원력 전략을 적용할 수 있습니다. 일부 완화는 &mdash;에서 더 전술적입니다. 예를 들어 일시적인 네트워크 오류가 발생한 후에 원격 호출을 다시 시도합니다. 다른 완화는 보조 지역에 전체 응용 프로그램을 장애 조치하는 작업과 같이 더 전략적입니다. 전술적 완화로 인해 큰 차이가 발생할 수 있습니다. 전체 지역이 중단되는 경우는 드물지만 네트워크 정체와 같은 일시적인 문제는 일반적이므로 &mdash; 먼저 이러한 문제를 대상으로 합니다. 올바른 모니터링 및 진단 작업은 오류가 발생하는 경우에 감지하고 근본 원인을 찾는 데 중요합니다.

복원력 있는 응용 프로그램을 디자인할 경우 가용성 요구 사항을 이해해야 합니다. 허용되는 가동 중지 시간이 얼마나 됩니까? 이것은 비용 함수의 일종입니다. 가동 중지로 인한 잠재적 비용이 얼마나 발생할까요? 고가용성 응용 프로그램을 만들려면 얼마를 투자해야 할까요?

[복원력 검사 목록][resiliency-checklist]을 사용하여 복원력 관점에서 디자인을 검토합니다.

### <a name="resiliency-guidance"></a>복원력 지침

- [Azure용 복원 응용 프로그램 디자인][resiliency]
- [복원력을 위한 디자인 패턴][resiliency-patterns]
- 모범 사례: [일시적인 오류 처리][transient-fault-handling], [특정 서비스에 대한 다시 시도 지침][retry-service-specific]

## <a name="management-and-devops"></a>관리 및 DevOps

이 핵심 요소에서는 프로덕션에서 응용 프로그램을 계속 실행하는 작업 프로세스를 다룹니다.

배포는 안정적이고 예측이 가능해야 합니다. 사용자 오류의 발생 가능성을 줄이기 위해 자동화되어야 합니다. 새로운 기능 또는 버그 수정의 릴리스 속도를 저하하지 않도록 빠르고 일상적인 프로세스여야 합니다. 또한 업데이트에 문제가 있는 경우 신속하게 롤백 또는 롤포워드할 수 있어야 합니다.

모니터링 및 진단은 매우 중요한 요소입니다. 클라우드 응용 프로그램은 개발자가 인프라 전체에 대한 제어 권한을 갖고 있지 않은 원격 데이터 센터에서 실행되거나, 또는 경우에 따라 운영 체제에서 실행됩니다. 대규모 응용 프로그램에서 문제를 해결하거나 로그 파일을 이동하기 위해 VM에 로그인하는 것은 실용적이지 않습니다. PaaS 서비스를 사용하여 로그인할 전용 VM이 없을 수도 있습니다. 모니터링 및 진단을 통해 오류가 발생하는 시기 및 위치를 알 수 있도록 시스템에 대한 정보를 제공합니다. 모든 시스템은 관찰 가능해야 합니다. 시스템에서 이벤트를 상호 연결할 수 있도록 공통적이며 일관된 로깅 스키마를 사용합니다.

모니터링 및 진단 프로세스에는 고유 단계가 있습니다.

- 계측 응용 프로그램 로그, 웹 서버 로그, Azure 플랫폼에 기본 제공되는 진단 및 기타 소스에서 원시 데이터를 생성합니다.
- 수집 및 저장 데이터를 한 위치에 통합합니다.
- 분석 및 진단 문제를 해결하고 전반적인 상태를 확인합니다.
- 시각화 및 경고 원격 분석 데이터를 사용하여 추세를 파악하거나 운영 팀에 경고합니다.

[DevOps 검사 목록][devops-checklist]을 사용하여 관리 및 DevOps 관점에서 디자인을 검토합니다.

### <a name="management-and-devops-guidance"></a>관리 및 DevOps 지침

- [관리 및 모니터링을 위한 디자인 패턴][management-patterns]
- 모범 사례: [통합 모니터링 및 진단][monitoring]

## <a name="security"></a>보안

디자인 및 구현에서 배포 및 작업까지 응용 프로그램의 전체 수명 주기 동안 보안을 고려해야 합니다. Azure 플랫폼은 네트워크 침입 및 DDoS 공격 등 다양한 위협으로부터 보호를 제공합니다. 하지만 여전히 응용 프로그램 및 DevOps 프로세스에 보안을 빌드해야 합니다.

고려해야 할 몇 가지 광범위한 보안 영역은 다음과 같습니다. 

### <a name="identity-management"></a>ID 관리

Azure AD(Azure Active Directory)를 사용하여 사용자를 인증하고 권한을 부여하는 것이 좋습니다. Azure AD는 완전히 관리되는 ID 및 액세스 관리 서비스입니다. Azure에서 순수하게 존재하는 도메인을 만들거나 온-프레미스 Active Directory ID와 통합하는 데 사용할 수 있습니다. 또한 Azure AD는 Office365, Dynamics CRM Online 및 여러 타사 SaaS 응용 프로그램과 통합됩니다. 소비자 지향 응용 프로그램의 경우 Azure Active Directory B2C를 통해 사용자는 기존 소셜 계정(예: Facebook, Google 또는 LinkedIn)을 사용하여 인증하거나 Azure AD에서 관리되는 새 사용자 계정을 만들 수 있습니다.

Azure 네트워크와 온-프레미스 Active Directory 환경을 통합하려면 요구 사항에 따라 여러 가지 방법을 사용할 수 있습니다. 자세한 내용은 [ID 관리][identity-ref-arch] 참조 아키텍처를 참조하세요.

### <a name="protecting-your-infrastructure"></a>인프라 보호 

배포하는 Azure 리소스에 대한 액세스를 제어합니다. 모든 Azure 구독은 Azure AD 테넌트와 [트러스트 관계][ad-subscriptions]가 있습니다. RBAC([역할 기반 액세스 제어][rbac])를 사용하여 조직 내의 사용자에게 Azure 리소스에 대한 올바른 사용 권한을 부여합니다. 특정 범위에서 사용자 또는 그룹에 RBAC 역할을 할당하여 액세스를 부여합니다. 범위는 구독, 리소스 그룹 또는 단일 리소스일 수 있습니다. 인프라의 모든 변경 내용을 [감사][resource-manager-auditing]합니다. 

### <a name="application-security"></a>응용 프로그램 보안

일반적으로 응용 프로그램 개발에 대한 보안 모범 사례는 여전히 클라우드에 적용됩니다. 여기에는 어디서든 SSL을 사용하고, CSRF 및 XSS 공격으로부터 보호하고, SQL 삽입 공격을 방지하는 것과 같은 작업이 포함됩니다. 

종종 클라우드 응용 프로그램은 액세스 키가 있는 관리되는 서비스를 사용합니다. 소스 제어에서 이러한 항목을 확인하지 않습니다. Azure Key Vault에 응용 프로그램 암호를 저장하는 것이 좋습니다.

### <a name="data-sovereignty-and-encryption"></a>데이터 주권 및 암호화

Azure를 항상 사용 가능한 경우 데이터가 올바른 지리적 영역에서 유지되는지 확인합니다. Azure 지역에서 복제된 저장소는 동일한 지역에서 [쌍을 이루는 지역][paired-region]이라는 개념을 사용합니다. 

Key Vault를 사용하여 암호화 키 및 암호를 보호합니다. Key Vault를 사용하여 HSM(하드웨어 보안 모듈)에서 보호하는 키를 사용하여 키 및 암호를 암호화할 수 있습니다. 많은 Azure Storage 및 DB 서비스는 [Azure Storage][storage-encryption], [Azure SQL Database][sql-db-encryption], [Azure SQL Data Warehouse][data-warehouse-encryption] 및 [Cosmos DB][documentdb-encryption]를 비롯한 미사용 데이터 암호화를 지원합니다.

### <a name="security-resources"></a>보안 리소스

- [Azure Security Center][security-center]는 Azure 구독에서 통합된 보안 모니터링 및 정책 관리를 제공합니다. 
- [Azure 보안 설명서][security-documentation]
- [Microsoft 보안 센터][trust-center]



<!-- links -->

[dr-guidance]: ../resiliency/disaster-recovery-azure-applications.md
[identity-ref-arch]: ../reference-architectures/identity/index.md
[resiliency]: ../resiliency/index.md

[ad-subscriptions]: /azure/active-directory/active-directory-how-subscriptions-associated-directory
[data-warehouse-encryption]: /azure/data-lake-store/data-lake-store-security-overview#data-protection
[documentdb-encryption]: /azure/documentdb/documentdb-nosql-database-security
[rbac]: /azure/active-directory/role-based-access-control-what-is
[paired-region]: /azure/best-practices-availability-paired-regions
[resource-manager-auditing]: /azure/azure-resource-manager/resource-group-audit
[security-blog]: https://azure.microsoft.com/blog/tag/security/
[security-center]: https://azure.microsoft.com/services/security-center/
[security-documentation]: /azure/security/
[sql-db-encryption]: /azure/sql-database/sql-database-always-encrypted-azure-key-vault
[storage-encryption]: /azure/storage/storage-service-encryption
[trust-center]: https://azure.microsoft.com/support/trust-center/
 

<!-- patterns -->
[availability-patterns]: ../patterns/category/availability.md
[management-patterns]: ../patterns/category/management-monitoring.md
[resiliency-patterns]: ../patterns/category/resiliency.md
[scalability-patterns]: ../patterns/category/performance-scalability.md


<!-- practices -->
[autoscale]: ../best-practices/auto-scaling.md
[background-jobs]: ../best-practices/background-jobs.md
[caching]: ../best-practices/caching.md
[cdn]: ../best-practices/cdn.md
[data-partitioning]: ../best-practices/data-partitioning.md
[monitoring]: ../best-practices/monitoring.md
[retry-service-specific]: ../best-practices/retry-service-specific.md
[transient-fault-handling]: ../best-practices/transient-faults.md


<!-- checklist -->
[availability-checklist]: ../checklist/availability.md
[devops-checklist]: ../checklist/dev-ops.md
[resiliency-checklist]: ../checklist/resiliency.md
[scalability-checklist]: ../checklist/scalability.md
