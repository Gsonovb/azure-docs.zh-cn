---
title: "创建 SQL Server 可用性组侦听器 - Azure 虚拟机 | Microsoft 文档"
description: "有关为 Azure 虚拟机中的 SQL Server AlwaysOn 可用性组创建侦听器的分步说明"
services: virtual-machines
documentationcenter: na
author: MikeRayMSFT
manager: jhubbard
editor: monicar
ms.assetid: d1f291e9-9af2-41ba-9d29-9541e3adcfcf
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 12/28/2016
ms.author: mikeray
translationtype: Human Translation
ms.sourcegitcommit: 407b189af12116d633ed505facf4bcfde9be5822
ms.openlocfilehash: 6a37e9e786a4e399c49cb77758a23793790888c9


---
# <a name="configure-an-internal-load-balancer-for-an-always-on-availability-group-in-azure"></a>在 Azure 中为 Always On 可用性组配置内部负载均衡器
本主题说明如何在 Resource Manager 模型中运行的 Azure 虚拟机上为 SQL Server Always On 可用性组创建内部负载均衡器。 当 SQL Server 实例位于 Azure 虚拟机上时，AlwaysOn 可用性组需要负载均衡器。 负载平衡器存储可用性组侦听器的 IP 地址。 如果可用性组跨多个区域，则每个区域都需要一个负载均衡器。

若要完成此任务，需要在 Resource Manager 模型中的 Azure 虚拟机上部署 SQL Server 可用性组。 这两个 SQL Server 虚拟机必须属于同一个可用性集。 可以使用 [Microsoft 模板](virtual-machines-windows-portal-sql-alwayson-availability-groups.md)在 Azure Resource Manager 中自动创建可用性组。 此模板将自动为你创建内部负载平衡器。 

如果需要，可以[手动配置可用性组](virtual-machines-windows-portal-sql-alwayson-availability-groups-manual.md)。

本主题要求事先配置可用性组。  

相关主题包括：

* [在 Azure VM (GUI) 中配置 Always On 可用性组](virtual-machines-windows-portal-sql-alwayson-availability-groups-manual.md)   
* [使用 Azure Resource Manager 和 PowerShell 配置 VNet 到 VNet 连接](../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)

执行本文档中的每个步骤可以在 Azure 门户中创建和配置负载均衡器。 完成这些步骤后，你将配置群集，以将负载均衡器中的 IP 地址用于可用性组侦听器。

## <a name="create-and-configure-the-load-balancer-in-the-azure-portal"></a>在 Azure 门户中创建并配置负载平衡器
在任务的这一部分，将在 Azure 门户中执行以下步骤：

1. 创建负载平衡器并配置 IP 地址
2. 配置后端池
3. 创建探测 
4. 设置负载平衡规则

> [!NOTE]
> 如果 SQL Server 位于不同的资源组和区域，则需要执行上述所有步骤两次：在每个资源组中各执行一次。
> 
> 

### <a name="1-create-the-load-balancer-and-configure-the-ip-address"></a>1.创建负载平衡器并配置 IP 地址
第一步是创建负载平衡器。 在 Azure 门户中，打开包含 SQL Server 虚拟机的资源组。 在资源组中，单击“添加”。

1. 搜索“负载均衡器”。 从搜索结果中选择 **Microsoft** 发布的**负载均衡器**。
1. 在“负载均衡器”边栏选项卡上，单击“创建”。
1. 在“创建负载均衡器”中配置负载均衡器，如下所示：

   | 设置 | 值 |
   | --- | --- |
   | **Name** |表示负载平衡器的文本名称。 例如 **sqlLB**。 |
   | **类型** |**内部** |
   | **虚拟网络** |选择 SQL Server 所在的虚拟网络。 |
   | **子网** |选择 SQL Server 所在的子网。 |
   | **IP 地址分配** |**静态** |
   | **专用 IP 地址** |指定子网中的某个可用 IP 地址。 在群集上创建侦听器时，将使用此 IP 地址。 本文稍后的 PowerShell 脚本会将此地址用于 `$ILBIP` 变量。 |
   | **订阅** |如果你有多个订阅，可能会显示此字段。 选择要与此资源关联的订阅。 它通常是与可用性组的所有资源相同的订阅。 |
   | **资源组** |选择 SQL Server 所在的资源组。 |
   | **位置** |选择 SQL Server 所在的 Azure 位置。 |

1. 单击“创建” 。 

Azure 将创建负载均衡器。 该负载平衡器属于特定的网络、子网、资源组和位置。 在 Azure 完成操作后，请在 Azure 中验证负载平衡器设置。 

### <a name="2-configure-the-backend-pool"></a>2.配置后端池
下一步是创建后端地址池。 Azure 将后端地址池称作*后端池*。 在本例中，后端池是可用性组中两个 SQL Server 的地址。 

1. 在资源组中，单击你创建的负载均衡器。 
1. 在“设置”中，单击“后端池”。
1. 在“后端池”中，单击“添加”创建后端地址池。 
1. 在“添加后端池”中的“名称”下，键入后端池的名称。
1. 在“虚拟机”下，单击“+ 添加虚拟机”。 
1. 在“选择虚拟机”下，单击“选择可用性集”，然后指定 SQL Server 虚拟机所属的可用性集。
1. 选择可用性集后，请单击“选择虚拟机”。 单击在可用性组中托管 SQL Server 实例的两个虚拟机。 单击“选择”。 
1. 单击“确定”，关闭“选择虚拟机”和“添加后端池”边栏选项卡。 

Azure 将更新后端地址池的设置。 现在，可用性集具有包含两个 SQL 服务器的池。

### <a name="3-create-a-probe"></a>3.创建探测
下一步是创建探测。 探测定义 Azure 如何确认哪一个 SQL Server 当前拥有可用性组侦听器。 Azure 根据创建探测时定义的端口上的 IP 地址来探测服务。

1. 在负载均衡器的“设置”边栏选项卡上，单击“运行状况探测”。 
1. 在“运行状况探测”边栏选项卡上，单击“添加”。
1. 在“添加探测”边栏选项卡上配置探测。 使用以下值配置探测：

   | 设置 | 值 |
   | --- | --- |
   | **Name** |表示探测的文本名称。 例如 **SQLAlwaysOnEndPointProbe**。 |
   | **协议** |**TCP** |
   | **端口** |可以使用任何可用的端口。 例如 *59999*。 |
   | **间隔** |*5* |
   | **不正常阈值** |*2* |

1.  单击 **“确定”**。 

> [!NOTE]
> 确保指定的端口已在两个 SQL Server 的防火墙上打开。 这两个服务器需要所用 TCP 端口的入站规则。 有关详细信息，请参阅 [Add or Edit Firewall Rule](http://technet.microsoft.com/library/cc753558.aspx)（添加或编辑防火墙规则）。 
> 
> 

Azure 将创建探测。 Azure 使用探测来测试哪个 SQL Server 具有可用性组的侦听器。

### <a name="4-set-the-load-balancing-rules"></a>4.设置负载平衡规则
设置负载平衡规则。 负载平衡规则设置负载平衡器将流量路由到 SQL Server 的方式。 对此负载均衡器，需要启用直接服务器返回，因为在两个 SQL Server 中，每次只有一个拥有可用性组侦听器资源。

1. 在负载均衡器的“设置”边栏选项卡上，单击“负载均衡规则”。 
1. 在“负载均衡规则”边栏选项卡上，单击“添加”。
1. 使用“添加负载均衡规则”边栏选项卡配置负载均衡规则。 使用以下设置： 

   | 设置 | 值 |
   | --- | --- |
   | **Name** |表示负载平衡规则的文本名称。 例如 **SQLAlwaysOnEndPointListener**。 |
   | **协议** |**TCP** |
   | **端口** |*1433* |
   | **后端端口** |*1433*.将忽略此值，因为此规则使用“浮动 IP (直接服务器返回)”。 |
   | **探测** |使用为此负载平衡器创建的探测的名称。 |
   | **会话暂留** |**无** |
   | **空闲超时(分钟)** |*4* |
   | **浮动 IP (直接服务器返回)** |**已启用** |

   > [!NOTE]
   > 可能需要在边栏选项卡中向下滚动才能看到所有设置。
   > 

1. 单击 **“确定”**。 
1. Azure 将配置负载平衡规则。 负载平衡器现已配置为将流量路由到托管可用性组侦听器的 SQL Server。 

此时，资源组有一个连接到这两个 SQL Server 计算机的负载均衡器。 负载均衡器还包含 SQL Server AlwaysOn 可用性组侦听器的 IP 地址，以便任一计算机可以响应针对可用性组的请求。

> [!NOTE]
> 如果 SQL Server 位于两个不同的区域，请在另一个区域重复上述步骤。 每个区域都需要一个负载平衡器。 
> 
> 

## <a name="configure-the-cluster-to-use-the-load-balancer-ip-address"></a>将群集配置为使用负载平衡器 IP 地址
下一步是在群集上配置侦听器，然后将侦听器联机。 请执行以下步骤： 

1. 在故障转移群集上创建可用性组侦听器 
2. 使侦听器联机

### <a name="5-create-the-availability-group-listener-on-the-failover-cluster"></a>5.在故障转移群集上创建可用性组侦听器
在此步骤中，你在故障转移群集管理器和 SQL Server Management Studio (SSMS) 中手动创建可用性组侦听器。

[!INCLUDE [ag-listener-configure](../../../../includes/virtual-machines-ag-listener-configure.md)]

<!---------------------------
* Use RDP to connect to the Azure virtual machine that hosts the primary replica. 
* Open Failover Cluster Manager.
* Select the **Networks** node, and note the cluster network name. This name will be used in the `$ClusterNetworkName` variable in the PowerShell script.
* Expand the cluster name, and then click **Roles**.
* In the **Roles** pane, right-click the availability group name and then select **Add Resource** > **Client Access Point**.
* In the **Name** box, create a name for this new listener, then click **Next** twice, and then click **Finish**. Do not bring the listener or resource online at this point.
  
  > [!NOTE]
  > The name for the new listener is the network name that applications will use to connect to databases in the SQL Server availability group.
  > 
  > 
* Click the **Resources** tab, then expand the Client Access Point you just created. Right-click the IP resource and click properties. Note the name of the IP address. You will use this name in the `$IPResourceName` variable in the PowerShell script.
* Under **IP Address** click **Static IP Address** and set the static IP address to the same address that you used when you set the load balancer IP address on the Azure portal. Enable NetBIOS for this address and click **OK**. Repeat this step for each IP resource if your solution spans multiple Azure VNets. 
* On the cluster node that currently hosts the primary replica, open an elevated PowerShell ISE and paste the following commands into a new script.
  
        $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
        $IPResourceName = "<IPResourceName>" # the IP Address resource name
        $ILBIP = “<X.X.X.X>” # the IP Address of the Internal Load Balancer (ILB). This is the static IP address for the load balancer you configured in the Azure portal.
  
        Import-Module FailoverClusters
  
        Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ILBIP";"ProbePort"="59999";"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
* Update the variables and run the PowerShell script to configure the IP address and port for the new listener.
  
  > [!NOTE]
  > If your SQL Servers are in separate regions, you need to run the PowerShell script twice. The first time use the cluster network name, cluster IP resource name, and load balancer IP address from the first resource group. The second time use the cluster network name, cluster IP resource name, and load balancer IP address from the second resource group.
  > 
  > 

Now the cluster has an availability group listener resource.

### 2. Bring the listener online
With the availability group listener resource configured, you can bring the listener online so that applications can connect to databases in the availability group with the listener.

* Navigate back to Failover Cluster Manager. Expand **Roles** and then highlight your Availability Group. On the **Resources** tab, right-click the listener name and click **Properties**.
* Click the **Dependencies** tab. If there are multiple resources listed, verify that the IP addresses have OR, not AND, dependencies. Click **OK**.
* Right-click the listener name and click **Bring Online**.
* Once the listener is online, from the **Resources** tab, right-click the availability group and click **Properties**.
* Create a dependency on the listener name resource (not the IP address resources name). Click **OK**.
* Launch SQL Server Management Studio and connect to the primary replica.
* Navigate to **AlwaysOn High Availability** | **Availability Groups** | **Availability Group Listeners**. 
* You should now see the listener name that you created in Failover Cluster Manager. Right-click the listener name and click **Properties**.
* In the **Port** box, specify the port number for the availability group listener by using the $EndpointPort you used earlier (1433 was the default), then click **OK**.

------------------------------->

### <a name="verify-the-configuration-of-the-listener"></a>验证侦听器的配置

如果正确配置了群集资源和依赖项，应能够看到 SQL Server Management Studio 中的侦听器。 执行以下步骤设置侦听器端口：

1. 启动 SQL Server Management Studio 并连接到主副本。
2. 导航到“AlwaysOn 高可用性” | “可用性组” | “可用性组侦听器”。 
1. 你现在应看到在故障转移群集管理器中创建的侦听器名称。 右键单击侦听器名称，然后单击“属性”。
1. 在“端口”框中，通过使用先前使用过的 $EndpointPort 为可用性组侦听器指定端口号（默认值为&1433;），然后单击“确定”。

现在，你在 Resource Manager 模式下运行的 Azure 虚拟机中有了一个 SQL Server AlwaysOn 可用性组。 

## <a name="test-the-connection-to-the-listener"></a>测试与侦听器的连接
若要测试连接，请执行以下操作：

1. 通过 RDP 连接到同一虚拟网络中不拥有副本的 SQL Server。 这可以是群集中的其他 SQL Server。
2. 使用 **sqlcmd** 实用工具测试连接。 例如，以下脚本通过侦听器与 Windows 身份验证来与主副本建立 **sqlcmd** 连接：
   
        sqlcmd -S <listenerName> -E

SQLCMD 连接将自动连接到托管主副本的 SQL Server 实例。 

## <a name="guidelines-and-limitations"></a>指导原则和限制
请注意有关 Azure 中使用内部负载平衡器的可用性组侦听器的以下准则：

* 每个云服务只支持一个内部可用性组侦听器，因为该侦听器将配置给负载均衡器，并且只有一个内部负载均衡器。 但是，可以创建多个外部侦听器。 
* 使用内部负载均衡器只能从同一个虚拟网络中访问侦听器。


<!--HONumber=Jan17_HO2-->


