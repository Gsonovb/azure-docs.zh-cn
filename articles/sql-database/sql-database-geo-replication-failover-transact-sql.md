---
title: "使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外故障转移 | Microsoft 文档"
description: "使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外故障转移"
services: sql-database
documentationcenter: 
author: CarlRabeler
manager: jhubbard
editor: 
ms.assetid: 5eb2d256-025d-4f5a-99d4-17f702b37f14
ms.service: sql-database
ms.custom: business continuity
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-management
ms.date: 01/10/2017
ms.author: carlrab
translationtype: Human Translation
ms.sourcegitcommit: 8d988aa55d053d28adcf29aeca749a7b18d56ed4
ms.openlocfilehash: 6d5ee44b57ce3e60b72ff2a2d182f2b8a39ecf81
ms.lasthandoff: 02/16/2017


---
# <a name="initiate-a-planned-or-unplanned-failover-for-azure-sql-database-with-transact-sql"></a>使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外故障转移

本文介绍了如何使用 Transact-SQL 为辅助 SQL 数据库启动故障转移。 若要配置异地复制，请参阅[为 Azure SQL 数据库配置异地复制](sql-database-geo-replication-transact-sql.md)。

若要启动故障转移，需要提供以下各项：

* 主数据库上 DBManager 的登录名，拥有你要异地复制的本地数据库的 db_ownership 权限，并且是你要配置异地复制的伙伴服务器上的 DBManager。
* SQL Server Management Studio (SSMS)

> [!IMPORTANT]
> 建议始终使用最新版本的 Management Studio 以保持与 Microsoft Azure 和 SQL 数据库的更新同步。 [更新 SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx)。
> 
> 

## <a name="initiate-a-planned-failover-promoting-a-secondary-database-to-become-the-new-primary"></a>启动计划的故障转移，将辅助数据库升级为新的主数据库
可以使用 **ALTER DATABASE** 语句来升级辅助数据库，以计划好的方式使它成为新的主数据库，并将现有的主数据库降级为辅助数据库。 此语句将在要升级的异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器中的 master 数据库上运行。 此功能适用于计划的故障转移（例如灾难恢复演练期间），它要求主数据库可用。

该命令将执行以下工作流：

1. 暂时将复制切换为同步模式，导致所有未完成的事务被排送到辅助数据库并阻止所有新事务；
2. 切换异地复制合作关系中两个数据库的角色。  

此序列保证在角色切换前同步这两个数据库，因此不会发生数据丢失。 切换角色时，有一小段时间无法使用这两个数据库（大约为 0 到 25 秒）。 如果主数据库具有多个辅助数据库，则该命令将自动重新配置其他辅助数据库以连接到新的主数据库。  在正常情况下，完成整个操作所需的时间应该少于一分钟。 有关详细信息，请参阅 [ALTER DATABASE (Transact-SQL) ](https://msdn.microsoft.com/library/mt574871.aspx)和[服务层](sql-database-service-tiers.md)。

使用以下步骤来启动计划的故障转移。

1. 在 Management Studio 中，连接到异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器。
2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。
3. 使用以下 **ALTER DATABASE** 语句以将辅助数据库切换为主角色。
   
        ALTER DATABASE <MyDB> FAILOVER;
4. 单击“执行”运行查询。

> [!NOTE]
> 在少数情况下，操作无法完成并且可能会出现停滞。 在此情况下，用户可以执行强制故障转移命令并接受数据丢失。
> 
> 

## <a name="initiate-an-unplanned-failover-from-the-primary-database-to-the-secondary-database"></a>启动从主数据库到辅助数据库的非计划故障转移
可以使用 **ALTER DATABASE** 语句来升级辅助数据库，以非计划的方式使它成为新的主数据库，当主数据库不可用时，强制将现有主数据库降级为辅助数据库。 此语句将在要升级的异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器中的 master 数据库上运行。

此功能适用于还原数据库的可用性至关重要并且可接受部分数据丢失的灾难恢复场合。 调用强制故障转移时，指定的辅助数据库将立即成为主数据库，并开始接受写入事务。 一旦原始主数据库能够与此新主数据库重新连接，将在原始主数据库上创建增量备份，旧的主数据库将变成新主数据库的辅助数据库；然后，它只是新主数据库的同步副本。

但是，由于辅助数据库上不支持时间点还原，发生强制故障转移之前，如果用户想要恢复已提交到旧主数据库但尚未复制到新主数据库的数据，则应该咨询支持人员来恢复这些丢失的数据。

如果主数据库具有多个辅助数据库，则该命令将自动重新配置其他辅助数据库以连接到新的主数据库。

使用以下步骤来启动计划外故障转移。

1. 在 Management Studio 中，连接到异地复制辅助数据库所在的 Azure SQL 数据库逻辑服务器。
2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。
3. 使用以下 **ALTER DATABASE** 语句以将辅助数据库切换为主角色。
   
        ALTER DATABASE <MyDB>   FORCE_FAILOVER_ALLOW_DATA_LOSS;
4. 单击“执行”运行查询。

> [!NOTE]
> 如果在主数据库和辅助数据库在线时发出此命令，旧的主数据库将立即变为新的辅助数据库，但不会进行数据同步。 如果发出命令时主数据库正在提交事务，则可能会丢失某些数据。
> 
> 

## <a name="next-steps"></a>后续步骤
* 故障转移后，请确保在新的主机上配置服务器和数据库的身份验证要求。 有关详细信息，请参阅[灾难恢复后的 Azure SQL 数据库安全性](sql-database-geo-replication-security-config.md)。
* 若要了解如何使用活动异地复制进行灾难恢复，包括恢复前后步骤和执行灾难恢复演练，请参阅[灾难恢复](sql-database-disaster-recovery.md)
* 有关活动异地复制的 Sasha Nosov 博客文章，请参阅[聚焦异地复制新功能](https://azure.microsoft.com/blog/spotlight-on-new-capabilities-of-azure-sql-database-geo-replication/)
* 若要了解如何设计云应用程序以使用活动异地复制，请参阅[设计云应用程序以使用异地复制实现业务连续性](sql-database-designing-cloud-solutions-for-disaster-recovery.md)
* 有关将活动异地复制与弹性池配合使用的信息，请参阅[弹性池灾难恢复策略](sql-database-disaster-recovery-strategies-for-applications-with-elastic-pool.md)。
* 有关业务连续性概述，请参阅[业务连续性概述](sql-database-business-continuity.md)


