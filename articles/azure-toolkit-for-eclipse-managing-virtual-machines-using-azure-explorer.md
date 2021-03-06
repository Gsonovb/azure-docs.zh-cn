---
title: "使用用于 Eclipse 的 Azure 资源管理器管理虚拟机 | Microsoft Docs"
description: "了解如何使用用于 Eclipse 的 Azure 资源管理器管理 Azure 虚拟机。"
services: 
documentationcenter: java
author: rmcmurray
manager: erikre
editor: 
ms.assetid: 
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: multiple
ms.devlang: Java
ms.topic: article
ms.date: 04/14/2017
ms.author: robmcm
translationtype: Human Translation
ms.sourcegitcommit: 9eafbc2ffc3319cbca9d8933235f87964a98f588
ms.openlocfilehash: f8631c5fe566762862f64b47843e154b68bb8031
ms.lasthandoff: 04/22/2017


---

# <a name="managing-virtual-machines-using-the-azure-explorer-for-eclipse"></a>使用用于 Eclipse 的 Azure 资源管理器管理虚拟机

Azure 资源管理器是用于 Eclipse 的 Azure 工具包的一部分，它为 Java 开发人员提供易用的解决方案，用于从 Eclipse IDE 内部管理其 Azure 帐户中的虚拟机。

[!INCLUDE [azure-toolkit-for-eclipse-prerequisites](../includes/azure-toolkit-for-eclipse-prerequisites.md)]

[!INCLUDE [azure-toolkit-for-eclipse-show-azure-explorer](../includes/azure-toolkit-for-eclipse-show-azure-explorer.md)]

## <a name="creating-a-virtual-machine-in-eclipse"></a>在 Eclipse 中创建虚拟机

以下步骤将引导你完成使用 Azure 资源管理器创建虚拟机的步骤。

1. 遵循[用于 Eclipse 的 Azure 工具包的登录说明]一文中的步骤登录到 Azure 帐户。

1. 在“Azure 资源管理器”视图中，展开 Azure 节点，右键单击“虚拟机”，然后单击“创建 VM”。
   ![“创建 VM”菜单][CR01]

1. 显示“新建虚拟机”向导时，选择订阅，然后单击“下一步”。
   ![“新建虚拟机”向导][CR02]

1. 在该向导的下一屏幕上，指定以下选项，然后单击“下一步”： ![“新建虚拟机”向导][CR03]

   a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。 **位置**：指定将创建虚拟机的位置；例如“美国西部”。

   b.保留“数据库类型”设置，即设置为“共享”。 **发布者**：指定创建了用于创建虚拟机的映像的发布者，例如“Microsoft”。

   c. **产品/服务**：指定要使用哪一个从所选发布者提供的虚拟机；例如“JDK”。

   d.单击“下一步”。 **Sku**：从所选产品/服务中指定要使用的*库存单位 (SKU)*；例如“JDK_8”。

   e.在“新建 MySQL 数据库”边栏选项卡中，接受法律条款，然后单击“确定”。 **版本号**：从所选 SKU 中指定要使用哪个版本。

1. 在该向导的下一屏幕上，指定以下选项，然后单击“下一步”：![“新建虚拟机”向导][CR04]

   a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。 **虚拟机名称**：指定新虚拟机的名称，该名称必须以字母开头并仅包含字母、数字和连字符。

   b. **大小**：指定要为虚拟机分配的内核数和内存。

   c. **用户名**：指定要创建的用于管理虚拟机的管理员帐户。

   d.单击“下一步”。 **密码**和**确认**：指定管理员帐户的密码。

1. 在向导的最后一个屏幕中指定以下选项：

   a.在“解决方案资源管理器”中，右键单击项目文件夹下的“引用”文件夹，然后单击“添加引用”。 **资源组**：为虚拟机指定资源组；需要选择以下选项之一：
      * **新建**：指定要创建新的资源组。
      * **使用现有**：指定将从列表中选择与 Azure 帐户关联的资源组。

   b.保留“数据库类型”设置，即设置为“共享”。 **存储帐户**：指定要用于存储虚拟机的存储帐户，可以选择现有存储帐户，也可以创建新帐户。 如果选择“&lt;&lt;新建&gt;&gt;”，将显示以下对话框：

      ![“新建存储帐户”对话框][CR05]

   c. **虚拟网络**和**子网**：指定虚拟机将连接到的虚拟网络和子网；可以选择现有网络和子网以用于虚拟机，也可以创建新的网络和子网。 如果选择“&lt;&lt;新建&gt;&gt;”，将显示以下对话框：<br/>

      ![“新建虚拟网络”对话框][CR06]

   d.单击“下一步”。 **公共 IP 地址**：为虚拟机指定面向外部的 IP 地址；可以选择创建新的 IP 地址，也可以选择“(无)”（如果虚拟机将不具有公共 IP 地址）。

   e.在“新建 MySQL 数据库”边栏选项卡中，接受法律条款，然后单击“确定”。 **网络安全组**：指定虚拟机将使用的可选网络防火墙；可以选择现有防火墙，也可以选择“(无)”（如果虚拟机将不使用网络防火墙）。

   f. **可用性集**：指定虚拟机可能属于的可选可用性集；可以选择现有的可用性集，也可以创建新的可用性集，或选择“(无)”（如果虚拟机将不属于可用性集）。

1. 输入上面列出的所有选项后，请单击“完成”：![“创建新虚拟机”向导][CR07]

1. 完成上述步骤后，将在“Azure 资源管理器”工具窗口中显示新的虚拟机。
   ![新建虚拟机][CR08]

## <a name="restarting-a-virtual-machine-in-eclipse"></a>在 Eclipse 中重新启动虚拟机

若要在 Eclipse 中使用 Azure 资源管理器重新启动虚拟机，请使用以下步骤：

1. 在“Azure 资源管理器”视图中，右键单击虚拟机，然后选择“重新启动”。
   ![重新启动虚拟机][RE01]

1. 出现提示时，单击“是”重新启动虚拟机。
   ![重新启动虚拟机][RE02]

## <a name="shutting-down-a-virtual-machine-in-eclipse"></a>在 Eclipse 中关闭虚拟机

若要在 Eclipse 中使用 Azure 资源管理器关闭正在运行的虚拟机，请使用以下步骤：

1. 在“Azure 资源管理器”视图中，右键单击虚拟机，然后选择“关闭”。
   ![关闭虚拟机][SH01]

1. 出现提示时，单击“是”关闭虚拟机。
   ![关闭虚拟机][SH02]

## <a name="deleting-a-virtual-machine-in-eclipse"></a>在 Eclipse 中删除虚拟机

若要在 Eclipse 中使用 Azure 资源管理器删除虚拟机，请使用以下步骤：

1. 在“Azure 资源管理器”视图中，右键单击虚拟机，然后选择“删除”。
   ![删除虚拟机][DE01]

1. 出现提示时，单击“是”删除虚拟机。
   ![删除虚拟机][DE02]

## <a name="see-also"></a>另请参阅
有关 Azure 虚拟机大小和定价的详细信息，请参阅以下链接：

* Azure 虚拟机大小
  * [Azure 中 Windows 虚拟机的大小]
  * [Azure 中 Linux 虚拟机的大小]
* Azure 虚拟机定价
  * [Windows 虚拟机定价]
  * [Linux 虚拟机定价]

有关 Azure Toolkits for Java IDE 的详细信息，请参阅以下链接：

* [用于 Eclipse 的 Azure 工具包]
  * [用于 Eclipse 的 Azure 工具包的新增功能]
  * [安装用于 Eclipse 的 Azure 工具包]
  * [用于 Eclipse 的 Azure 工具包的登录说明]
  * [在 Eclipse 中创建 Azure 的 Hello World Web 应用]
* [用于 IntelliJ 的 Azure 工具包]
  * [用于 IntelliJ 的 Azure 工具包的新增功能]
  * [安装用于 IntelliJ 的 Azure 工具包]
  * [用于 IntelliJ 的 Azure 工具包的登录说明]
  * [在 IntelliJ 中创建 Azure 的 Hello World Web 应用]

有关将 Azure 与 Java 配合使用的详细信息，请参阅 [Azure Java 开发人员中心]和[用于 Visual Studio Team Services 的 Java 工具]。

<!-- URL List -->

[用于 Eclipse 的 Azure 工具包]: ./azure-toolkit-for-eclipse.md
[用于 IntelliJ 的 Azure 工具包]: ./azure-toolkit-for-intellij.md
[在 Eclipse 中创建 Azure 的 Hello World Web 应用]: ./app-service-web/app-service-web-eclipse-create-hello-world-web-app.md
[在 IntelliJ 中创建 Azure 的 Hello World Web 应用]: ./app-service-web/app-service-web-intellij-create-hello-world-web-app.md
[安装用于 Eclipse 的 Azure 工具包]: ./azure-toolkit-for-eclipse-installation.md
[安装用于 IntelliJ 的 Azure 工具包]: ./azure-toolkit-for-intellij-installation.md
[用于 Eclipse 的 Azure 工具包的登录说明]: ./azure-toolkit-for-eclipse-sign-in-instructions.md
[用于 IntelliJ 的 Azure 工具包的登录说明]: ./azure-toolkit-for-intellij-sign-in-instructions.md
[用于 Eclipse 的 Azure 工具包的新增功能]: ./azure-toolkit-for-eclipse-whats-new.md
[用于 IntelliJ 的 Azure 工具包的新增功能]: ./azure-toolkit-for-intellij-whats-new.md

[Azure Java 开发人员中心]: https://azure.microsoft.com/develop/java/
[用于 Visual Studio Team Services 的 Java 工具]: https://java.visualstudio.com/

[Azure 中 Windows 虚拟机的大小]: /azure/virtual-machines/virtual-machines-windows-sizes
[Azure 中 Linux 虚拟机的大小]: /azure/virtual-machines/virtual-machines-linux-sizes
[Windows 虚拟机定价]: /pricing/details/virtual-machines/windows/
[Linux 虚拟机定价]: /pricing/details/virtual-machines/linux/

<!-- IMG List -->

[RE01]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/RE01.png
[RE02]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/RE02.png

[SH01]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/SH01.png
[SH02]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/SH02.png

[DE01]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/DE01.png
[DE02]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/DE02.png

[CR01]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/CR01.png
[CR02]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/CR02.png
[CR03]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/CR03.png
[CR04]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/CR04.png
[CR05]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/CR05.png
[CR06]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/CR06.png
[CR07]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/CR07.png
[CR08]: ./media/azure-toolkit-for-eclipse-managing-virtual-machines-using-azure-explorer/CR08.png

