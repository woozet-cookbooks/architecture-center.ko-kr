---
title: 'Technical guidance: Recovery from on-premises to Azure'
description: Article on understanding and designing recovery systems from on-premises infrastructure to Azure
author: adamglick
ms.service: guidance
ms.topic: article
ms.date: 08/18/2016
ms.author: pnp
---
[!INCLUDE [header](../_includes/header.md)]
# Azure 복원 기술 지침: 온프레미스에서 Azure로 복구
Azure는 고가용성 및 재해 복구 목적으로 온프레미스 데이터센터를 Azure로 확장하는 것을 가능케 하는 종합적 서비스 모음을 제공합니다.

* **네트워킹**: 가상 개인 네트워크를 사용하면 온프레미스 네트워크를 클라우드로 안전하게 확장할 수 있습니다.
* **컴퓨팅**: Hyper-V 온프레미스를 사용하는 고객들은 기존 가상 컴퓨터(VM)을 Azure로 이동/전환할 수 있습니다.
* **저장소**: StorSimple은 여러분의 파일 시스템을 Azure Storage로 연장할 수 있습니다. Azure Backup 서비스는 파일과 SQL 데이터베이스를 Azure Storage에 백업하는 것을 지원합니다.
* **데이터베이스 복제**: SQL Server 2014(이후 버전) 가용성 그룹을 사용하여 온프레미스 데이터에 대한 고가용성 및 재해 복구 방안을 구현할 수 있습니다.

## 네트워킹
Azure 가상 네트워크를 사용하여 Azure에 논리적으로 분리된 섹션을 만들고, IPsec 연결을 사용하여 이를 온프레미스 데이터센터 또는 단일 클라이언트 컴퓨터에 안전하게 연결할 수 있습니다. 가상 네트워크를 통해서 Azure의 확장 가능한 온디맨드 인프라를 활용할 수 있고 아울러 Windows Server, 메인프레임 및 UNIX에서 실행되는 시스템을 포함하여 온프레미스 데이터 및 응용 프로그램에 연결할 수 있습니다. 자세한 내용은 [Azure 네트워킹 설명서](/azure/virtual-network/virtual-networks-overview/)를 참조하십시오.

## 컴퓨팅
Hyper-V 온프레미스를 사용하고 있을 경우, VM을 변경하거나 VM 형식을 변환하지 않고서도 기존 가상 컴퓨터를 Azure 및 Windows Server 2012 이후 버전을 실행하는 서비스 제공자로 이동/전환할 수 있습니다. 자세한 내용은 [Azure 가상 컴퓨터를 위한 디스크 및 VHD 정보](/azure/virtual-machines/virtual-machines-linux-about-disks-vhds/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)를 참조하십시오.

## Azure Site Recovery
서비스형 재해 복구(disaster recovery as a service, DRaaS)를 원할 경우 Azure는 [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/)를 제공합니다 Azure Site Recovery는 VMware, Hyper-V 및 물리적 서버를 위한 종합적 보호 방안입니다. Azure Site Recovery를 이용하면 다른 온프레미스 서버나 Azure를 복구 사이트로 사용할 수 있습니다. Azure Site Recovery에 관한 자세한 내용은 [Azure Site Recovery 설명서](https://azure.microsoft.com/documentation/services/site-recovery/)를 참조하십시오.

## 저장소
Azure를 온프레미스 데이터용 백업 사이트로 사용하기 위한 몇 가지 옵션이 있습니다.

### StorSimple
StorSimple은 온프레미스 응용 프로그램들을 위해 클라우드 저장소를 안전하고 투명하게 통합합니다. 또한 고성능 계층형 로컬 저장소 및 클라우드 저장소, 라이브 보관, 클라우드 기반 데이터 보호 및 재해 복구를 지원하는 단일 어플라이언스를 제공합니다. 자세한 내용은 [StorSimple 제품 페이지](https://azure.microsoft.com/services/storsimple/)를 참조하십시오.

### Azure Backup
Azure Backup은 Windows Server 2012 이상, Windows Server 2012 Essentials 이상, System Center 2012 Data Protection Manager 이상 버전에 있는 친숙한 백업 도구를 사용하여 클라우드 백업을 지원합니다. 이들 도구는 로컬 디스크 또는 Azure Storage에 관계 없이 백업 저장 위치와는 독립적인 백업 관리 워크플로를 제공합니다. 데이터를 클라우드에 백업하고 나면, 공인된 사용자들은 원하는 서버에 백업을 쉽게 복구할 수 있습니다.

With incremental backups, only changes to files are transferred to the cloud. This helps to efficiently use storage space, reduce bandwidth consumption, and support point-in-time recovery of multiple versions of the data. You can also choose to use additional features, such as data retention policies, data compression, and data transfer throttling. Using Azure as the backup location has the obvious advantage that the backups are automatically “offsite”. This eliminates the extra requirements to secure and protect on-site backup media.

For more information, see [What is Azure Backup?](/azure/backup/backup-introduction-to-azure-backup/) and [Configure Azure Backup for DPM data](https://technet.microsoft.com/library/jj728752.aspx).

## Database
You can have a disaster recovery solution for your SQL Server databases in a hybrid-IT environment by using AlwaysOn Availability Groups, database mirroring, log shipping, and backup and restore with Azure Blob storage. All of these solutions use SQL Server running on Azure Virtual Machines.

AlwaysOn Availability Groups can be used in a hybrid-IT environment where database replicas exist both on-premises and in the cloud. This is shown in the following diagram.

![SQL Server AlwaysOn Availability Groups in a hybrid cloud architecture](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-3.png)

Database mirroring can also span on-premises servers and the cloud in a certificate-based setup. The following diagram illustrates this concept.

![SQL Server database mirroring in a hybrid cloud architecture](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-4.png)

Log shipping can be used to synchronize an on-premises database with a SQL Server database in an Azure virtual machine.

![SQL Server log shipping in a hybrid cloud architecture](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-5.png)

Finally, you can back up an on-premises database directly to Azure Blob storage.

![Back up SQL Server to Azure Blob storage in a hybrid cloud architecture](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-6.png)

For more information, see [High availability and disaster recovery for SQL Server in Azure virtual machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/) and [Backup and restore for SQL Server in Azure virtual machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery/).

## Checklists for on-premises recovery in Microsoft Azure
### Networking
1. Review the Networking section of this document.
2. Use Virtual Network to securely connect on-premises to the cloud.

### Compute
1. Review the Compute section of this document.
2. Relocate VMs between Hyper-V and Azure.

### Storage
1. Review the Storage section of this document.
2. Take advantage of StorSimple services for using cloud storage.
3. Use the Azure Backup service.

### Database
1. Review the Database section of this document.
2. Consider using SQL Server on Azure VMs as the backup.
3. Set up AlwaysOn Availability Groups.
4. Configure certificate-based database mirroring.
5. Use log shipping.
6. Back up on-premises databases to Azure Blob storage.


