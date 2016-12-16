---
title: "监视 XTP 内存中存储 | Microsoft 文档"
description: "估算和监视 XTP 内存中存储用量与容量；解决容量错误 41823"
services: sql-database
documentationcenter: 
author: jodebrui
manager: jhubbard
editor: 
ms.assetid: b617308e-692c-4938-8fa2-070034a3ecef
ms.service: sql-database
ms.custom: db development; monitor and tune
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/03/2016
ms.author: jodebrui
translationtype: Human Translation
ms.sourcegitcommit: 5a101aa78dbac4f1a0edb7f414b44c14db392652
ms.openlocfilehash: 475af239e4c77626d4c069f1238bc32d3b440a0e


---
# <a name="monitor-in-memory-oltp-storage"></a>监视 In-Memory OLTP 存储
使用[内存中 OLTP](sql-database-in-memory.md) 时，内存优化表中的数据和表变量将驻留在内存中 OLTP 存储内。 每个高级服务层都有最大的内存中 OLTP 存储大小，如 [SQL 数据库服务层](sql-database-service-tiers.md#standalone-database-service-tiers-and-performance-levels)一文中所述。 一旦超过此限制，插入和更新操作可能会开始失败（错误 41823）。 到时，你需要删除数据以回收内存，或升级数据库的性能层。

## <a name="determine-whether-data-will-fit-within-the-in-memory-storage-cap"></a>确定数据是否在内存中存储容量限制范围内
确定存储限制：有关不同高级服务层的存储限制的信息，请参阅 [SQL 数据库服务层](sql-database-service-tiers.md#standalone-database-service-tiers-and-performance-levels)一文。

估计内存优化表的内存要求，如同在 Azure SQL 数据库中估计 SQL Server 的内存要求一样。 花费几分钟时间查看 [MSDN](https://msdn.microsoft.com/library/dn282389.aspx) 上的相关主题。

请注意，表和表变量行以及索引都将计入最大用户数据大小。 此外，ALTER TABLE 需要足够的空间来创建新版的完整表及其索引。

## <a name="monitoring-and-alerting"></a>监视和警报
可以在 Azure [门户](https://portal.azure.com/)中，通过[性能层的存储限制](sql-database-service-tiers.md#standalone-database-service-tiers-and-performance-levels)百分比来监视内存中存储用量： 

* 在“数据库”边栏选项卡上，找出“资源使用率”框并单击“编辑”。
* 然后选择指标 `In-Memory OLTP Storage percentage`。
* 若要添加警报，请单击“资源使用率”框以打开“度量值”边栏选项卡，然后单击“添加警报”。

或者，使用以下查询来显示内存中存储的使用率：

    SELECT xtp_storage_percent FROM sys.dm_db_resource_stats


## <a name="correct-out-of-memory-situations---error-41823"></a>更正内存不足情况 - 错误 41823
内存不足会导致插入、更新和创建操作失败，并出现错误消息 41823。

错误消息 41823 指示内存优化表和表变量已超过大小上限。

若要解决此错误，请执行以下操作之一：

* 从内存优化表中删除数据，为此，可以将数据卸载到传统的基于磁盘的表；或者，
* 将服务层升级到具有足够内存中存储的服务层，使保存你需要保留在内存优化表中的数据。

## <a name="next-steps"></a>后续步骤
有关[使用动态管理视图监视 Azure SQL 数据库](sql-database-monitoring-with-dmvs.md)的其他资源



<!--HONumber=Nov16_HO3-->

