---
title: "将数据从 SQL Server 载入 Azure SQL 数据仓库 (bcp) | Microsoft 文档"
description: "对于少量的数据，可以使用 bcp 将数据从 SQL Server 导出到平面文件，然后将数据直接导入 Azure SQL 数据仓库。"
services: sql-data-warehouse
documentationcenter: NA
author: barbkess
manager: jhubbard
editor: 
ms.assetid: f87d8d7c-8276-43c5-90e7-d4ccc0e3a224
ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.custom: loading
ms.date: 10/31/2016
ms.author: barbkess
translationtype: Human Translation
ms.sourcegitcommit: c0e2324a2b2e6294df6e502f2e7a0ae36ff94158
ms.openlocfilehash: 8dc7c2fb833c1c51ecef772ba1cbe5f0405fe494
ms.lasthandoff: 01/30/2017



---
# <a name="load-data-from-sql-server-into-azure-sql-data-warehouse-flat-files"></a>将数据从 SQL Server 载入 Azure SQL 数据仓库（平面文件）
> [!div class="op_single_selector"]
> * [SSIS](sql-data-warehouse-load-from-sql-server-with-integration-services.md)
> * [PolyBase](sql-data-warehouse-load-from-sql-server-with-polybase.md)
> * [bcp](sql-data-warehouse-load-from-sql-server-with-bcp.md)
> 
> 

对于小型数据集，可以使用 bcp 命令行实用工具从 SQL Server 导出数据，然后将其直接加载到 Azure SQL 数据仓库。

在本教程中，你将使用 bcp 来执行以下操作：

* 使用 bcp out 命令从 SQL Server 导出表（或创建简单的示例文件）
* 将表从平面文件导入 SQL 数据仓库。
* 针对加载的数据创建统计信息。

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Loading-data-into-Azure-SQL-Data-Warehouse-with-BCP/player]
> 
> 

## <a name="before-you-begin"></a>开始之前
### <a name="prerequisites"></a>先决条件
若要逐步完成本教程，你需要：

* 一个 SQL 数据仓库数据库。
* 已安装 bcp 命令行实用工具
* 已安装 sqlcmd 命令行实用工具

可以从 [Microsoft 下载中心][Microsoft Download Center]下载 bcp 和 sqlcmd 实用工具。

### <a name="data-in-ascii-or-utf-16-format"></a>采用 ASCII 或 UTF-16 格式的数据
如果你使用自己的数据尝试学习本教程，则数据需要使用 ASCII 或 UTF-16 编码，因为 bcp 不支持 UTF-8。 

PolyBase 支持 UTF-8，但尚不支持 UTF-16。 请注意，如果你要结合使用 bcp 和 PolyBase，则从 SQL Server 导出数据后，需要将数据转换为 UTF-8。 

## <a name="1-create-a-destination-table"></a>1.创建目标表
在 SQL 数据仓库中定义加载操作的目标表。 该表中的列必须对应于数据文件每一行中的数据。

若要创建表，请打开命令提示符并使用 sqlcmd.exe 运行以下命令：

```sql
sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "
    CREATE TABLE DimDate2
    (
        DateId INT NOT NULL,
        CalendarQuarter TINYINT NOT NULL,
        FiscalQuarter TINYINT NOT NULL
    )
    WITH
    (
        CLUSTERED COLUMNSTORE INDEX,
        DISTRIBUTION = ROUND_ROBIN
    );
"
```


## <a name="2-create-a-source-data-file"></a>2.创建源数据文件
打开记事本，将以下几行数据复制到新文本文件，然后将此文件保存到本地临时目录，路径为 C:\Temp\DimDate2.txt。 此数据采用 ASCII 格式。

```
20150301,1,3
20150501,2,4
20151001,4,2
20150201,1,3
20151201,4,2
20150801,3,1
20150601,2,4
20151101,4,2
20150401,2,4
20150701,3,1
20150901,3,1
20150101,1,3
```

（可选）若要从 SQL Server 数据库导出自己的数据，请打开命令提示符并运行以下命令。 将 TableName、ServerName、DatabaseName、Username 和 Password 替换为你自己的信息。

```sql
bcp <TableName> out C:\Temp\DimDate2_export.txt -S <ServerName> -d <DatabaseName> -U <Username> -P <Password> -q -c -t ','
```



## <a name="3-load-the-data"></a>3.加载数据
若要加载数据，请打开命令提示符并运行以下命令，请注意将 Server Name、Database Name、Username 和 Password 替换为你自己的信息。

```sql
bcp DimDate2 in C:\Temp\DimDate2.txt -S <ServerName> -d <DatabaseName> -U <Username> -P <password> -q -c -t  ','
```

使用此命令来验证是否已正确加载数据

```sql
sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "SELECT * FROM DimDate2 ORDER BY 1;"
```

结果应如下所示：

| DateId | CalendarQuarter | FiscalQuarter |
| --- | --- | --- |
| 20150101 |1 |3 |
| 20150201 |1 |3 |
| 20150301 |1 |3 |
| 20150401 |2 |4 |
| 20150501 |2 |4 |
| 20150601 |2 |4 |
| 20150701 |3 |1 |
| 20150801 |3 |1 |
| 20150801 |3 |1 |
| 20151001 |4 |2 |
| 20151101 |4 |2 |
| 20151201 |4 |2 |

## <a name="4-create-statistics"></a>4.创建统计信息
SQL 数据仓库尚不支持自动创建或自动更新统计信息。 为了获得最佳查询性能，在首次加载数据或者在数据发生重大更改之后，必须针对所有表的所有列创建统计信息。 有关统计信息的详细说明，请参阅[统计信息][Statistics]。 

运行以下命令针对新加载的表创建统计信息。

```sql
sqlcmd.exe -S <server name> -d <database name> -U <username> -P <password> -I -Q "
    create statistics [DateId] on [DimDate2] ([DateId]);
    create statistics [CalendarQuarter] on [DimDate2] ([CalendarQuarter]);
    create statistics [FiscalQuarter] on [DimDate2] ([FiscalQuarter]);
"
```

## <a name="5-export-data-from-sql-data-warehouse"></a>5.从 SQL 数据仓库导出数据
为了增加乐趣，你可以从 SQL 数据仓库中导出刚刚加载的数据。  导出命令与从 SQL Server 导出所用的命令完全相同。

但是，结果会有所差异。 由于数据存储在 SQL 数据仓库中的分散位置，因此，当你导出数据时，每个计算节点会将数据写入输出文件。 输出文件中的数据顺序与输入文件中的数据顺序可能不同。

### <a name="export-a-table-and-compare-exported-results"></a>导出表并比较导出的结果
若要查看导出的数据，请打开命令提示符并使用自己的参数运行此命令。 ServerName 是 Azure 逻辑 SQL Server 的名称。

```sql
bcp DimDate2 out C:\Temp\DimDate2_export.txt -S <Server Name> -d <Database Name> -U <Username> -P <password> -q -c -t ','
```
你可以通过打开新文件来验证是否已正确导出数据。 文件中的数据应该与以下文本匹配，但可能以不同的顺序排序：

```
20150301,1,3
20150501,2,4
20151001,4,2
20150201,1,3
20151201,4,2
20150801,3,1
20150601,2,4
20151101,4,2
20150401,2,4
20150701,3,1
20150901,3,1
20150101,1,3
```

### <a name="export-the-results-of-a-query"></a>导出查询结果
可以使用 bcp 的 **queryout** 函数导出查询结果，而无需导出整个表。 

## <a name="next-steps"></a>后续步骤
有关加载数据的概述，请参阅[将数据载入 SQL 数据仓库][Load data into SQL Data Warehouse]。
有关更多开发技巧，请参阅 [SQL 数据仓库开发概述][SQL Data Warehouse development overview]。
有关在 SQL 数据仓库中创建表的详细信息，请参阅[表概述][Table Overview]或 [CREATE TABLE 语法][CREATE TABLE syntax]。

<!--Image references-->

<!--Article references-->

[Load data into SQL Data Warehouse]: ./sql-data-warehouse-overview-load.md
[SQL Data Warehouse development overview]: ./sql-data-warehouse-overview-develop.md
[Table Overview]: ./sql-data-warehouse-tables-overview.md
[Statistics]: ./sql-data-warehouse-tables-statistics.md

<!--MSDN references-->
[bcp]: https://msdn.microsoft.com/library/ms162802.aspx
[CREATE TABLE syntax]: https://msdn.microsoft.com/library/mt203953.aspx

<!--Other Web references-->
[Microsoft Download Center]: https://www.microsoft.com/download/details.aspx?id=36433

