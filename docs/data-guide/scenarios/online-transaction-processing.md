---
title: OLTP(온라인 트랜잭션 처리)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 07e7f680c8ee5e8589ff7cd2236ff95f6ee84f4c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/31/2018
---
# <a name="online-transaction-processing-oltp"></a>OLTP(온라인 트랜잭션 처리)

컴퓨터 시스템을 사용하는 [트랜잭션 데이터](../concepts/transactional-data.md) 관리를 OLTP(온라인 트랜잭션 처리)라고 합니다. OLTP 시스템은 조직의 일상적인 작업에서 발생하는 비즈니스 상호 작용을 기록하고, 이 데이터를 쿼리하여 유추할 수 있도록 지원합니다.

![Azure의 OLTP](./images/oltp-data-pipeline.png)

## <a name="when-to-use-this-solution"></a>이 솔루션을 사용해야 하는 경우

비즈니스 트랜잭션을 효율적으로 처리 및 저장하고, 클라이언트 응용 프로그램에 일관된 방식에서 사용할 수 있게 하려는 경우 OLTP를 선택합니다. 또한 분명한 처리 지연이 비즈니스의 일상 작업에 부정적인 영향을 미칠 때 이 아키텍처를 사용합니다.

OLTP 시스템은 트랜잭션을 효율적으로 처리 및 저장할 뿐만 아니라 트랜잭션 데이터를 쿼리하도록 디자인되었습니다. OLTP 시스템에서 개별 트랜잭션을 효율적으로 처리하고 저장한다는 목적은 데이터 정규화, 즉 데이터를 덜 중복되는 좀 더 작은 청크로 분할함으로써 어느 정도 달성됩니다. 이 경우 OLTP 시스템이 많은 수의 트랜잭션을 독립적으로 처리할 수 있도록 하고, 중복 데이터가 있을 때 데이터 무결성을 유지하기 위해 필요한 추가 처리가 해소되므로 효율성이 유지됩니다.

## <a name="challenges"></a>과제
OLTP 시스템을 구현 및 사용할 경우 다음과 같은 몇 가지 해결 과제가 발생할 수 있습니다.

- 잘 계획된 SQL Server 기반 솔루션과 같은 예외도 있지만, OLTP 시스템이 대량의 데이터에 대한 집계를 처리하는 데 항상 적절한 것은 아닙니다. 수백만 개의 개별 트랜잭션에 대한 집계 계산에 의존하는 데이터 분석은 OLTP 시스템의 리소스를 과도하게 사용합니다. 따라서 실행이 느려질 수 있으며, 데이터베이스의 다른 트랜잭션을 차단하게 되어 속도 저하를 야기할 수 있습니다.
- 고도로 정규화된 데이터에 대해 분석 및 보고 작업을 수행할 경우, 조인을 사용해서 대부분의 쿼리를 비정규화해야 하므로 쿼리가 복잡해질 수 있습니다. 또한 OLTP 시스템에서 데이터베이스 개체에 대한 명명 규칙은 간결하고 간단한 편입니다. 명명 규칙이 간결하고 정규화가 높아지면서 비즈니스 사용자가 DBA 또는 데이터 개발자의 도움 없이 OLTP 시스템을 쿼리하는 것이 어려워집니다.
- 트랜잭션 기록을 무기한 저장하고, 하나의 테이블에 너무 많은 데이터를 저장하게 되면 저장된 트랜잭션 수에 따라 쿼리 성능이 저하될 수 있습니다. 일반적인 해결 방법은 OLTP 시스템에서 적절한 기간(예: 현재 회계 연도) 동안 두었다가 기록 데이터를 데이터 마트 또는 [데이터 웨어하우스](../technology-choices/data-warehouses.md) 등의 다른 시스템으로 오프로드하는 것입니다.

## <a name="oltp-in-azure"></a>Azure의 OLTP

[App Service Web Apps](/azure/app-service/app-service-web-overview)에 호스트되는 웹 사이트, App Service에서 실행되는 REST API 등의 응용 프로그램이나 모바일 또는 데스크톱 응용 프로그램은 일반적으로 REST API를 매개자로 사용해서 OLTP 시스템과 통신합니다.

실제로 대부분의 워크로드는 순수한 OLTP가 아닙니다. [분석 구성 요소](../scenarios/online-analytical-processing.md)의 역할을 하기도 합니다. 또한, 운영 체제에 대한 보고서 실행 등, 실시간 보고에 대한 요구도 높아지고 있습니다. 이것을 HTAP(하이브리드 트랜잭션 및 분석 처리)라고도 합니다. 자세한 내용은 [OLAP(온라인 분석 처리) 데이터 저장소](../technology-choices/olap-data-stores.md)를 참조하세요.

## <a name="technology-choices"></a>기술 선택

데이터 저장소:

- [Azure SQL Database](/azure/sql-database/)
- [Azure VM의 SQL Server](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database for PostgreSQL](/azure/postgresql/)

자세한 내용은 [OLTP 데이터 저장소 선택](../technology-choices/oltp-data-stores.md)을 참조하세요.

데이터 원본:

- [App service](/azure/app-service/)
- [Mobile Apps](/azure/app-service-mobile/)

