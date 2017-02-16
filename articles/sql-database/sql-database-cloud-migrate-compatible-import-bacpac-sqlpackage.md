---
title: "SqlPackage：从 BACPAC 文件导入到 Azure SQL 数据库 | Microsoft Docs"
description: "本文说明如何使用 SqlPackage 命令行实用程序从 BACPAC 文件导入到 SQL 数据库。"
keywords: "Microsoft Azure SQL 数据库, 数据库迁移, 导入数据库, 导入 BACPAC 文件, sqlpackage"
services: sql-database
documentationcenter: 
author: CarlRabeler
manager: jhubbard
editor: 
ms.assetid: 424afa27-5f13-4ec3-98f6-99a511a6a2df
ms.service: sql-database
ms.custom: migrate and move
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: sqldb-migrate
ms.date: 11/08/2016
ms.author: carlrab
translationtype: Human Translation
ms.sourcegitcommit: 86bc7d89bb5725add8ba05b6f0978467147fd3ca
ms.openlocfilehash: 048a8f333eb555d860df0fd485ec8d53c27e3cd6


---
# <a name="import-to-sql-database-from-a-bacpac-file-using-sqlpackage"></a>使用 SqlPackage 从 BACPAC 文件导入到 SQL 数据库
> [!div class="op_single_selector"]
> * [SSMS](sql-database-cloud-migrate-compatible-import-bacpac-ssms.md)
> * [SqlPackage](sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage.md)
> * [Azure 门户](sql-database-import.md)
> * [PowerShell](sql-database-import-powershell.md)
> 
> 

本文说明如何使用 [SqlPackage](https://msdn.microsoft.com/library/hh550080.aspx) 命令行实用程序从 [BACPAC](https://msdn.microsoft.com/library/ee210546.aspx#Anchor_4) 文件导入到 SQL 数据库。 此实用程序附带最新版本的 [SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) 和[适用于 Visual Studio 的 SQL Server Data Tools](https://msdn.microsoft.com/library/mt204009.aspx)，也可以直接从 Microsoft 下载中心下载最新版本的 [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876)。

> [!NOTE]
> 以下步骤假定用户已预配 SQL 数据库服务器，手头有连接信息，并且已验证源数据库兼容。
> 
> 

## <a name="import-from-a-bacpac-file-into-azure-sql-database-using-sqlpackage"></a>使用 SqlPackage 从 BACPAC 文件导入到 Azure SQL 数据库
使用以下步骤通过 [SqlPackage.exe](https://msdn.microsoft.com/library/hh550080.aspx) 命令行实用程序从 BACPAC 文件导入兼容的 SQL Server 数据库（或 Azure SQL 数据库）。

> [!NOTE]
> 以下步骤假定用户已预配 Azure SQL 数据库服务器并且手头有连接信息。
> 
> 

1. 打开命令提示符并更改包含 sqlpackage.exe 命令行实用程序的目录 - 此实用程序与 Visual Studio 和 SQL Server 一起提供。
2. 结合环境的以下参数运行以下 sqlpackage.exe 命令：
   
    `sqlpackage.exe /Action:Import /tsn:< server_name > /tdn:< database_name > /tu:< user_name > /tp:< password > /sf:< source_file >`
   
   | 参数 | 说明 |
   | --- | --- |
   | < server_name > |目标服务器名称 |
   | < database_name > |目标数据库名称 |
   | < user_name > |目标服务器中的用户名 |
   | < password > |用户的密码 |
   | < source_file > |要导入的 BACPAC 文件的文件名和位置 |
   
    ![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage01c.png)

## <a name="next-steps"></a>后续步骤
* [最新版本的 SSDT](https://msdn.microsoft.com/library/mt204009.aspx)
* [最新版本的 SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx)

## <a name="additional-resources"></a>其他资源
* [SQL 数据库功能](sql-database-features.md)
* [Transact-SQL 部分支持或不支持的函数](sql-database-transact-sql-information.md)
* [使用 SQL Server 迁移助手迁移非 SQL Server 数据库](http://blogs.msdn.com/b/ssma/)




<!--HONumber=Jan17_HO1-->

