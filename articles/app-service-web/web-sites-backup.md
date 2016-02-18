<properties 
	pageTitle="在 Azure 网站中备份网站" 
	description="了解如何在 Azure 网站中创建网站的备份。" 
	services="app-service" 
	documentationCenter="" 
	authors="cephalin" 
	manager="wpickett" 
	editor="jimbe"/>

<tags
	ms.service="app-service"
	ms.date="12/10/2015"
	wacn.date=""/>

# 在 Azure 网站中备份网站


利用 [Azure 网站](/documentation/services/web-sites/)中的备份和还原功能，你可以轻松地手动或自动创建网站备份。你可以将网站还原到以前的状态，或者基于原始应用的备份之一创建新网站。

有关从备份中还原 Azure 网站的信息，请参阅[还原网站](/documentation/articles/web-sites-restore)。
##本文内容

- [备份的内容](#whatsbackedup)
- [要求和限制](#requirements)
- [创建手动备份](#manualbackup)
- [配置自动化的备份](#automatedbackups)
- [如何存储备份](#aboutbackups)
- [说明](#notes)
- [后续步骤](#nextsteps)
	- [有关存储帐户的详细信息](#moreaboutstorage)

<a name="whatsbackedup"></a>
## 备份的内容 
网站可以备份以下信息：

* 网站配置
* 网站文件内容
* 任何连接到你的应用的 Azure SQL 数据库或 MySQL 数据库（你可以选择备份中要包括哪些数据库）

此信息会备份到你指定的 Azure 存储帐户和容器。

> [AZURE.NOTE] 每个备份都是你的应用的完整脱机副本，而不是增量更新。

<a name="requirements"></a>
## 要求和限制

* 备份和还原功能要求 App Service 计划位于“标准”层中。有关缩放 App Service 计划以使用更高层的详细信息，请参阅[在 Azure 网站中缩放网站](/documentation/articles/web-sites-scale)。

* 备份和还原功能要求 Azure 存储帐户和容器与要备份的网站必须同属一个订阅。如果你还没有存储帐户，可以创建一个，方法是：单击 Azure 管理门户左窗格中的“存储”按钮（网格图标），然后选择底部命令栏中的“新建”。有关 Azure 存储帐户的详细信息，请参阅本文结尾处的[链接](#moreaboutstorage)。

* 备份和还原功能最多支持 10GB 的网站和数据库内容。如果因有效负载超出此限制备份功能无法继续执行，将指示错误。

<a name="manualbackup"></a>
## 创建手动备份
1. 在网站的 Azure 管理门户中，选择“备份”选项卡。
	
	![“备份”页面][ChooseBackupsPage]
	
2. 选择网站要备份到其中的存储帐户。该存储帐户必须与要备份的网站同属一个订阅。
	
	![选择存储帐户][ChooseStorageAccount]
	
3. 在“包含的数据库”选项中，选择要备份的连接到你的网站的数据库（SQL Server 或 MySQL）。
	
	![选择要包含的数据库][IncludedDatabases]

	> [AZURE.NOTE] 若要使数据库显示在此列表中，其连接字符串必须位于门户中“配置”选项卡的“连接字符串”部分。
	
4. 在命令栏上，单击“立即备份”。
	
	![BackUpNow 按钮][BackUpNow]
	
	在备份过程中你将看到进度消息：
	
	![备份进度消息][BackupProgress]
	
你可以随时进行手动备份。在预览期间，每 24 小时内的手动备份不能超过 2 个（此限制可能会变更）。
<a name="automatedbackups"></a>
## 配置自动执行的备份
1. 在“备份”页上，将“自动化的备份”设置为“打开”。
	
	![启用自动化的备份][SetAutomatedBackupOn]
	
2. 选择网站要备份到其中的存储帐户。该存储帐户必须与要备份的网站同属一个订阅。
	
	![选择存储帐户][ChooseStorageAccount]
	
3. 在“频率”框中，指定你希望多久进行一次自动化的备份。（在预览版期间，天数是唯一可用的时间单位。）
	
	![选择备份频率][Frequency]
	
	天数必须介于 1 到 90 之间（含 1 和 90，从每天 1 次到每 90 天一次）。
	
4. 使用“开始日期”选项指定你希望自动化的备份开始的日期和时间。
	
	![选择开始日期][StartDate]
	
	时间以半小时为增量。
	
	![选择开始时间][StartTime]
	
	> [AZURE.NOTE] Azure 存储备份时间时采用的是 UTC 格式，但显示时根据的是用于显示门户的计算机上的系统时间。
	
5. 在“包含的数据库”部分，选择要备份的连接到你的网站的数据库（SQL Server 或 MySQL）。若要使数据库显示在此列表中，其连接字符串必须位于门户中“配置”选项卡的“连接字符串”部分。
	
	![选择要包含的数据库][IncludedDatabases]
	
	> [AZURE.NOTE] 如果你选择在备份中包含一个或多个数据库，并且指定了小于 7 天的频率，就会收到频繁备份会增加数据库成本的警告。
	
6. 在命令栏中，单击“保存”按钮以保存你的配置更改（或者，如果你决定不保存，请选择“放弃”）。
	
	![“保存”按钮][SaveIcon]
<a name="notes"></a>
## 说明

* 请确保在网站的“设置”中的“网站设置”边栏选项卡中为你的每个数据库正确设置连接字符串，这样备份和还原功能才会包括这些数据库。


<a name="partialbackups"></a>
## 仅备份网站的一部分

有时你不想备份网站中的所有内容。以下是一些示例：

-	你[设置每周备份](/documentation/articles/web-sites-backup#configure-automated-backups)网站，其中包含永远不会更改的静态内容，例如旧的博客文章或图像。
-	网站的内容超过 10GB（这是一次可以备份的最大数量）。
-	你不想备份日志文件。

使用部分备份可以精确地选择想要备份的文件。

### 从备份中排除文件

若要从备份中排除文件和文件夹，在网站的 wwwroot 文件夹中创建 `_backup.filter` 文件，并指定要在该处排除的文件和文件夹列表。通过 [Kudu 控制台](https://github.com/projectkudu/kudu/wiki/Kudu-console)，即可轻松访问它。

假设你的网站中包含永远不会更改的历年的日志文件和静态图像。你已有包括旧图像的网站的完整备份。现在你想要每天都备份网站，但不想为存储永远不会更改的日志文件或静态图像文件支付费用。

![日志文件夹][LogsFolder]
![映像文件夹][ImagesFolder]
	
以下步骤展示了如何从备份中排除这些文件。

1. 转到 `http://{yourapp}.scm.chinacloudsites.cn/DebugConsole` 并确定想要从备份中排除的文件夹。在此示例中，你想要排除该用户界面中所示的以下文件和文件夹：

		D:\home\site\wwwroot\Logs
		D:\home\LogFiles
		D:\home\site\wwwroot\Images\2013
		D:\home\site\wwwroot\Images\2014
		D:\home\site\wwwroot\Images\brand.png

	> [AZURE.NOTE] 最后一行说明了你可以排除单个文件以及文件夹。

2. 创建名为 `_backup.filter` 的文件并将上述列表放在文件中，但请删除 `D:\home`。每行列出一个目录或文件。文件的内容应为：

    \\site\\wwwroot\\Logs
    \\LogFiles
    \\site\\wwwroot\\Images\\2013
    \\site\\wwwroot\\Images\\2014
    \\site\\wwwroot\\Images\\brand.png

3. 使用 [ftp](/documentation/articles/web-sites-deploy#ftp) 或任何其他方法将此文件上载到站点的 `D:\home\site\wwwroot` 目录。如果你愿意，则可以直接在 `http://{yourapp}.scm.chinacloudsites.cn/DebugConsole` 中创建文件并在那里插入内容。

4. 采用通常使用的相同方式运行备份，即[手动](#create-a-manual-backup)或[自动](#configure-automated-backups)。

现在，`_backup.filter` 中指定的所有文件和文件夹都会从备份中排除。在此示例中，日志文件和 2013 年以及 2014 年的映像文件以及 brand.png 将不再进行备份。

>[AZURE.NOTE] 采用[还原定期备份](/documentation/articles/web-sites-restore)的相同方式还原站点的部分备份。还原过程会执行正确的操作。
>
>还原完整备份后，站点上的所有内容都被替换为备份中的任何内容。如果文件在站点上但不在备份中，则会将其删除。但是当部分备份还原时，位于其中一个黑名单目录或任何黑名单文件中的任何内容都保持不变。

<a name="aboutbackups"></a>

## 如何存储备份

对网站进行一个或多个备份后，则会在你的存储帐户的“容器”边栏选项卡中看到这些备份以及你的网站。在存储帐户中，每个备份都由一个 .zip 文件和一个 .xml 文件组成，前者包含备份数据，后者包含 .zip 文件内容的清单。如果你想要在无需实际执行网站还原的情况下访问备份，则可以解压缩并浏览这些文件。

网站的数据库备份存储在 .zip 文件的根目录中。对于 SQL 数据库，这是 BACPAC 文件（无文件扩展名），并且可以导入。若要基于 BACPAC 导出创建新的 SQL 数据库，请参阅[导入 BACPAC 文件以创建新的用户数据库](http://technet.microsoft.com/zh-cn/library/hh710052.aspx)。

> [AZURE.WARNING] 改动“websitebackups”容器中的任何文件都会导致备份无效，进而无法还原。

<a name="nextsteps"></a>
## 后续步骤
有关从备份中还原网站的信息，请参阅[在 Azure 网站中还原网站](/documentation/articles/web-sites-restore)。

若要开始使用 Azure，请参阅 [Microsoft Azure 试用版](/pricing/1rmb-trial/)。


<!-- IMAGES -->
[ChooseBackupsPage]: ./media/web-sites-backup/01ChooseBackupsPage.png
[ChooseStorageAccount]: ./media/web-sites-backup/02ChooseStorageAccount.png
[IncludedDatabases]: ./media/web-sites-backup/03IncludedDatabases.png
[BackUpNow]: ./media/web-sites-backup/04BackUpNow.png
[BackupProgress]: ./media/web-sites-backup/05BackupProgress.png
[SetAutomatedBackupOn]: ./media/web-sites-backup/06SetAutomatedBackupOn.png
[Frequency]: ./media/web-sites-backup/07Frequency.png
[StartDate]: ./media/web-sites-backup/08StartDate.png
[StartTime]: ./media/web-sites-backup/09StartTime.png
[SaveIcon]: ./media/web-sites-backup/10SaveIcon.png
[ImagesFolder]: ./media/web-sites-backup/11Images.png
[LogsFolder]: ./media/web-sites-backup/12Logs.png
[GhostUpgradeWarning]: ./media/web-sites-backup/13GhostUpgradeWarning.png
 

<!---HONumber=Mooncake_0118_2016-->