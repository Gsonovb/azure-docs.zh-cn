---
title: "Service Fabric 节点类型和 VM 规模集 | Microsoft 文档"
description: "介绍 Service Fabric 节点类型如何与 VM 规模集相关联，以及如何远程连接到 VM 规模集实例或群集节点。"
services: service-fabric
documentationcenter: .net
author: ChackDan
manager: timlt
editor: 
ms.assetid: 5441e7e0-d842-4398-b060-8c9d34b07c48
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/05/2017
ms.author: chackdan
translationtype: Human Translation
ms.sourcegitcommit: aaf97d26c982c1592230096588e0b0c3ee516a73
ms.openlocfilehash: 32119a6ef586d616407c69e89a0d0f05758438bc
ms.lasthandoff: 04/27/2017


---
# <a name="the-relationship-between-service-fabric-node-types-and-virtual-machine-scale-sets"></a>Service Fabric 节点类型与虚拟机规模集之间的关系
虚拟机规模集是一种 Azure 计算资源，可用于将一组 VM 作为一个集进行部署和管理。 在 Service Fabric 群集中定义的每个节点类型将设置为不同的 VM 规模集。 然后，每个节点类型可以独立扩展或缩减、打开不同的端口集，并可以有不同的容量指标。

以下屏幕截图显示了具有 FrontEnd 和 BackEnd 两个节点类型的群集。  每个节点类型各有五个节点。

![显示具有两个节点类型的群集的屏幕截图][NodeTypes]

## <a name="mapping-vm-scale-set-instances-to-nodes"></a>将 VM 规模集实例映射到节点
如上所示，VM 规模集实例从实例 0 开始逐渐递增。 编号反映在名称中。 例如，BackEnd_0 是 BackEnd VM 规模集的实例 0。 此特定 VM 规模集有五个实例，名称分别为 BackEnd_0、BackEnd_1、BackEnd_2、BackEnd_3、BackEnd_4。

当你扩展 VM 规模集时，将创建新的实例。 新 VM 规模集的名称通常是 VM 规模集名称 + 下一个实例编号。 在本示例中，即 BackEnd_5。

## <a name="mapping-vm-scale-set-load-balancers-to-each-node-typevm-scale-set"></a>将 VM 规模集负载均衡器映射到每个节点类型/VM 规模集
如果已从门户或使用提供的示例 Resource Manager 模板部署群集，则在获取资源组下所有资源的列表时，将看到每个 VM 规模集或节点类型的负载均衡器。

该名称类似于 **LB-&lt;NodeType name&gt;**。 例如，以下屏幕截图中显示了 LB-sfcluster4doc-0：

![资源][Resources]

## <a name="remote-connect-to-a-vm-scale-set-instance-or-a-cluster-node"></a>远程连接到 VM 规模集实例或群集节点
在群集中定义的每个节点类型将设置为不同的 VM 规模集。  这意味着，节点类型可以独立扩展或缩减，且可由不同的 VM SKU 组成。 不同于单实例 VM，VM 规模集的实例没有自身的虚拟 IP 地址。 因此你可能很难找到可用来远程连接到特定实例的 IP 地址和连接端口。

可以遵循以下步骤来查找 IP 地址和端口。

### <a name="step-1-find-out-the-virtual-ip-address-for-the-node-type-and-then-inbound-nat-rules-for-rdp"></a>步骤 1：找出该节点类型的虚拟 IP 地址，然后找出 RDP 的入站 NAT 规则
若要获取此信息，必须获取 **Microsoft.Network/loadBalancers** 资源定义中定义的入站 NAT 规则值。

在门户中，导航到“负载均衡器”边栏选项卡并选择“**设置**”。

![LBBlade][LBBlade]

在“**设置**”中，单击“**入站 NAT 规则**”。 随后可以获得可用来远程连接到第一个 VM 规模集实例的 IP 地址和端口。 以下屏幕截图中显示的是 **104.42.106.156** 和 **3389**

![NATRules][NATRules]

### <a name="step-2-find-out-the-port-that-you-can-use-to-remote-connect-to-the-specific-vm-scale-set-instancenode"></a>步骤 2：找出可用来远程连接到特定 VM 规模集实例/节点的端口
本文前面讨论了如何将 VM 规模集实例映射到节点。 我们将使用这种方法来找出确切的端口。

将按照 VM 规模集实例的升序分配端口。 因此，在 FrontEnd 节点类型示例中，5 个实例中的每一个实例端口如下所示。 现在需要对 VM 规模集实例执行相同的映射。

| **VM 规模集实例** | **端口** |
| --- | --- |
| FrontEnd_0 |3389 |
| FrontEnd_1 |3390 |
| FrontEnd_2 |3391 |
| FrontEnd_3 |3392 |
| FrontEnd_4 |3393 |
| FrontEnd_5 |3394 |

### <a name="step-3-remote-connect-to-the-specific-vm-scale-set-instance"></a>步骤 3：远程连接到特定 VM 规模集实例
在以下屏幕截图中，我使用了“远程桌面连接”来连接到 FrontEnd_1：

![RDP][RDP]

## <a name="how-to-change-the-rdp-port-range-values"></a>如何更改 RDP 端口范围值
### <a name="before-cluster-deployment"></a>群集部署之前
使用 Resource Manager 模板设置群集时，可以在 **inboundNatPools** 中指定范围。

转到 **Microsoft.Network/loadBalancers** 的资源定义。 在此处找到 **inboundNatPools** 的描述。  替换 *frontendPortRangeStart* 和 *frontendPortRangeEnd* 值。

![InboundNatPools][InboundNatPools]

### <a name="after-cluster-deployment"></a>群集部署之后
这稍微要复杂一点，并且可能会导致 VM 设置被回收。 你现在必须使用 Azure PowerShell 设置新值。 请确保计算机上已安装 Azure PowerShell 1.0 或更高版本。 如果尚未安装，强烈建议根据[如何安装和配置 Azure PowerShell](/powershell/azure/overview) 一文中所述的步骤安装。

登录到你的 Azure 帐户。 如果此 PowerShell 命令由于某些原因而失败，你应该检查 Azure PowerShell 是否已正确安装。

```
Login-AzureRmAccount
```

运行以下命令获取有关负载均衡器的详细信息，在值中，可以看到 **inboundNatPools** 的描述：

```
Get-AzureRmResource -ResourceGroupName <RGname> -ResourceType Microsoft.Network/loadBalancers -ResourceName <load balancer name>
```

现在，将 *frontendPortRangeEnd* 和 *frontendPortRangeStart* 设置为所需的值。

```
$PropertiesObject = @{
    #Property = value;
}
Set-AzureRmResource -PropertyObject $PropertiesObject -ResourceGroupName <RG name> -ResourceType Microsoft.Network/loadBalancers -ResourceName <load Balancer name> -ApiVersion <use the API version that get returned> -Force
```


## <a name="next-steps"></a>后续步骤
* [“随地部署”功能的概述及其与 Azure 托管群集的比较](service-fabric-deploy-anywhere.md)
* [群集安全性](service-fabric-cluster-security.md)
* [Service Fabric SDK 及其入门](service-fabric-get-started.md)

<!--Image references-->
[NodeTypes]: ./media/service-fabric-cluster-nodetypes/NodeTypes.png
[Resources]: ./media/service-fabric-cluster-nodetypes/Resources.png
[InboundNatPools]: ./media/service-fabric-cluster-nodetypes/InboundNatPools.png
[LBBlade]: ./media/service-fabric-cluster-nodetypes/LBBlade.png
[NATRules]: ./media/service-fabric-cluster-nodetypes/NATRules.png
[RDP]: ./media/service-fabric-cluster-nodetypes/RDP.png

