---
title: "기술 지침: 온-프레미스에서 Azure로 복구"
description: "온-프레미스 인프라에서 Azure에 이르는 복구 시스템의 이해 및 설계에 대한 문서"
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: f5ce86dbd605fa7dc74e6a7cc97f0d6c6acd79e5
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-on-premises-to-azure"></a>Azure 복원력 기술 지침: 온-프레미스에서 Azure로 복구
Azure는 고가용성 및 재해 복구를 위해서 Azure에 대한 온-프레미스 데이터 센터의 확장을 사용하도록 설정한 일련의 포괄적인 서비스를 제공합니다.

* **네트워킹**: 가상 개인 네트워크를 통해 온-프레미스 네트워크를 클라우드로 안전하게 확장할 수 있습니다.
* **Compute**: Hyper-V 온-프레미스를 사용하는 고객은 기존 VM(가상 머신)을 Azure로 “전환”할 수 있습니다.
* **저장소**: StorSimple을 사용하여 파일 시스템을 Azure Storage로 확장할 수 있습니다. Azure Backup 서비스는 파일 및 SQL 데이터베이스를 Azure Storage에 백업하는 기능을 제공합니다.
* **데이터베이스 복제**: SQL Server 2014 이상 가용성 그룹을 통해 온-프레미스 데이터에 대한 고가용성 및 재해 복구를 구현할 수 있습니다.

## <a name="networking"></a>네트워킹
Azure Virtual Network를 사용하면 Azure에 논리적으로 분리된 섹션을 만들 수 있으며 IPsec 연결을 사용하여 온-프레미스 데이터 센터 또는 단일 클라이언트 컴퓨터에 안전하게 연결할 수 있습니다. Virtual Network를 사용하면 Windows Server, 메인프레임 및 UNIX에서 실행되는 시스템을 비롯한 데이터 및 응용 프로그램 온-프레미스에 연결을 제공하는 동시에 Azure에서 확장성 있는 주문형 인프라를 쉽게 이용할 수 있습니다. 자세한 내용은 [Azure 네트워킹 설명서](/azure/virtual-network/virtual-networks-overview/) 를 참조하세요.

## <a name="compute"></a>컴퓨팅
Hyper-V를 사용하는 경우 온-프레미스는 VM을 변경하거나 VM 형식을 변환하지 않고 기존 가상 머신을 Windows Server 2012(또는 이상)를 실행하는 Azure 및 서비스 공급자에 “전환”할 수 있습니다. 자세한 내용은 [Azure 가상 머신용 디스크 및 VHD 정보](/azure/virtual-machines/virtual-machines-linux-about-disks-vhds/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)를 참조하세요.

## <a name="azure-site-recovery"></a>Azure Site Recovery
DRaaS(disaster recovery as a service)를 선호하는 경우 Azure는 [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/)를 제공합니다. Azure Site Recovery는 VMware, Hyper-V 및 물리적 서버에 대한 포괄적인 보호 기능을 제공합니다. Azure Site Recovery를 사용하면 다른 온-프레미스 서버 또는 Azure를 복구 사이트로 사용할 수 있습니다. Azure Site Recovery에 대한 자세한 내용은 [Azure Site Recovery 설명서](https://azure.microsoft.com/documentation/services/site-recovery/)를 참조하세요.

## <a name="storage"></a>Storage
온-프레미스 데이터에 대한 백업 사이트로 Azure를 사용하는 경우 몇 가지 옵션이 있습니다.

### <a name="storsimple"></a>StorSimple
StorSimple는 온-프레미스 응용 프로그램에 대한 클라우드 저장소를 안전하고 투명하게 통합합니다. 또한 고성능 계층화된 로컬 및 클라우드 저장소, 라이브 보관, 클라우드 기반 데이터 보호 및 재해 복구를 구현하는 단일 어플라이언스를 제공합니다. 자세한 내용은 [StorSimple 제품 페이지](https://azure.microsoft.com/services/storsimple/)를 참조하세요.

### <a name="azure-backup"></a>Azure Backup
Azure Backup을 통해 Windows Server 2012(또는 이상), Windows Server 2012 Essentials(또는 이상) 및 System Center 2012 Data Protection Manager(또는 이상)에서 익숙한 백업 도구를 사용하여 클라우드 백업을 활성화할 수 있습니다. 이러한 도구는 로컬 디스크 또는 Azure Storage 여부에 상관 없이 백업의 저장 위치와 독립적인 백업 관리에 대한 워크플로를 제공합니다. 클라우드로 데이터를 백업한 후에는 권한 있는 사용자가 서버로 백업을 쉽게 복구할 수 있습니다.

증분 백업으로, 변경된 파일만 클라우드에 전송됩니다. 이렇게 하면 저장소 공간을 효율적으로 사용하고 대역폭 사용을 줄일 수 있으며 여러 버전의 데이터에 대해 지정 시간 복구를 지원할 수 있습니다. 또한 데이터 보존 정책, 데이터 압축 및 데이터 전송 제한 등의 추가 기능을 사용할 수 있습니다. Azure를 백업 위치로 사용하면 백업이 자동으로 "오프사이트"가 된다는 장점이 있습니다. 이렇게 하면 추가 요구 사항을 제거하여 온사이트 백업 미디어를 보호할 수 있습니다.

자세한 내용은 [Azure Backup이란?](/azure/backup/backup-introduction-to-azure-backup/) 및 [DPM 데이터를 위한 Azure Backup 구성](https://technet.microsoft.com/library/jj728752.aspx)을 참조하세요.

## <a name="database"></a>데이터베이스
AlwaysOn 가용성 그룹, 데이터베이스 미러링, 로그 전달, Azure Blob 저장소를 사용한 백업 및 복원을 사용하여 하이브리드 IT 환경 내에 SQL Server 데이터베이스에 대한 재해 복구 솔루션을 구축할 수 있습니다. 다음 솔루션은 모두 Azure Virtual Machines에서 실행되는 SQL Server를 사용합니다.

데이터베이스 복제본이 온-프레미스와 클라우드 모두에 존재하는 하이브리드 IT 환경에서 AlwaysOn 가용성 그룹을 사용할 수 있습니다. 다음 다이어그램에 나와 있습니다.

![하이브리드 클라우드 아키텍처에서 SQL 서버 AlwaysOn 가용성 그룹](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-3.png)

데이터베이스 미러링은 온-프레미스 서버 및 인증서 기반 설정의 클라우드에 걸쳐 있을 수도 있습니다. 아래 다이어그램은 이 개념을 보여 줍니다.

![하이브리드 클라우드 아키텍처에서 SQL 서버 데이터베이스 미러링](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-4.png)

로그 전달을 사용하여 Azure 가상 머신에서 SQL Server 데이터베이스를 포함한 온-프레미스 데이터베이스를 동기화할 수 있습니다.

![하이브리드 클라우드 아키텍처에서 SQL 서버 로그 전달](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-5.png)

마지막으로 Azure Blob 저장소에 직접 온-프레미스 데이터베이스를 백업할 수 있습니다.

![하이브리드 클라우드 아키텍처의 Azure Blob 저장소에 SQL 서버 백업](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-6.png)

자세한 내용은 [Azure 가상 머신에서 SQL Server의 고가용성 및 재해 복구](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/) 및 [Azure 가상 머신에서 SQL Server Backup 및 복원](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery/)을 참조하세요.

## <a name="checklists-for-on-premises-recovery-in-microsoft-azure"></a>Microsoft Azure에서 온-프레미스 복구를 위한 검사 목록
### <a name="networking"></a>네트워킹
1. 이 문서의 네트워킹 섹션을 검토합니다.
2. Virtual Network를 사용하여 온-프레미스를 클라우드로 안전하게 연결합니다.

### <a name="compute"></a>컴퓨팅
1. 이 문서의 Compute 섹션을 검토합니다.
2. Hyper-V와 Azure 간에 VM을 재배치합니다.

### <a name="storage"></a>Storage
1. 이 문서의 저장소 섹션을 검토합니다.
2. 클라우드 저장소를 사용하기 위해 StorSimple 서비스를 활용합니다.
3. Azure Backup 서비스를 사용합니다.

### <a name="database"></a>데이터베이스
1. 이 문서의 데이터베이스 섹션을 검토합니다.
2. Azure VM에서 SQL Server를 백업으로 사용하도록 고려합니다.
3. AlwaysOn 가용성 그룹을 설정합니다.
4. 인증서 기반 데이터베이스 미러링을 구성합니다.
5. 로그 전달을 사용합니다.
6. 온-프레미스 데이터베이스를 Azure Blob 저장소에 백업합니다.


