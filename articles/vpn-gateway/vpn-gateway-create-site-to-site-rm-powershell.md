---
title: "将本地网络连接到 Azure 虚拟网络：站点到站点 VPN：PowerShell | Microsoft 文档"
description: "通过公共 Internet 创建从本地网络到 Azure 虚拟网络的 IPsec 连接的步骤。 这些步骤将帮助你使用 PowerShell 创建跨界站点到站点 VPN 网关连接。"
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: fcc2fda5-4493-4c15-9436-84d35adbda8e
ms.service: vpn-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/24/2017
ms.author: cherylmc
translationtype: Human Translation
ms.sourcegitcommit: b0c27ca561567ff002bbb864846b7a3ea95d7fa3
ms.openlocfilehash: 4ec11ffeae94b4a8e5a65566f0f0c067f45a0134
ms.lasthandoff: 04/25/2017


---
# <a name="create-a-vnet-with-a-site-to-site-vpn-connection-using-powershell"></a>使用 PowerShell 创建具有站点到站点 VPN 连接的 VNet

本文介绍如何使用 PowerShell 创建站点到站点 VPN 网关连接，以便从本地网络连接到 VNet。 本文中的步骤适用于 Resource Manager 部署模型。 也可使用不同的部署工具或部署模型来创建此配置，方法是从以下列表中选择另一选项：

> [!div class="op_single_selector"]
> * [Resource Manager - Azure 门户](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
> * [Resource Manager - PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md)
> * [Resource Manager - CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md)
> * [经典 - Azure 门户](vpn-gateway-howto-site-to-site-classic-portal.md)
> * [经典 - 经典门户](vpn-gateway-site-to-site-create.md)
> 
>


![站点到站点 VPN 网关跨界连接示意图](./media/vpn-gateway-create-site-to-site-rm-powershell/site-to-site-connection-diagram.png)

使用站点到站点 VPN 网关连接，通过 IPsec/IKE（IKEv1 或 IKEv2）VPN 隧道将本地网络连接到 Azure 虚拟网络。 此类型的连接要求位于本地的 VPN 设备分配有一个面向外部的公共 IP 地址。 有关 VPN 网关的详细信息，请参阅[关于 VPN 网关](vpn-gateway-about-vpngateways.md)。

## <a name="before-you-begin"></a>开始之前

在开始配置之前，请验证你是否符合以下条件：

* 验证你是否需要使用 Resource Manager 部署模型。 [!INCLUDE [deployment models](../../includes/vpn-gateway-deployment-models-include.md)] 
* 一台兼容的 VPN 设备和能够对其进行配置的人员。 有关兼容的 VPN 设备和设备配置的详细信息，请参阅[关于 VPN 设备](vpn-gateway-about-vpn-devices.md)。
* 一个用于 VPN 设备的面向外部的公共 IPv4 地址。 此 IP 地址不得位于 NAT 之后。
* 如果不熟悉本地网络配置中的 IP 地址范围，则需咨询能够为你提供此类详细信息的人员。 创建此配置时，必须指定 IP 地址范围前缀，Azure 会将该前缀路由到本地位置。 本地网络的任何子网都不得与要连接到的虚拟网络子网重叠。
* 最新版本的 Azure Resource Manager PowerShell cmdlet。 有关安装 PowerShell cmdlet 的详细信息，请参阅 [如何安装和配置 Azure PowerShell](/powershell/azureps-cmdlets-docs) 。

### <a name="example-values"></a>示例值

本文中的示例使用以下值。 可使用这些值创建测试环境，或参考这些值以更好地理解本文中的示例。

```
#Example values

VnetName                = testvnet 
ResourceGroup           = testrg 
Location                = West US 
AddressSpace            = 10.0.0.0/16 
SubnetName              = Subnet1 
Subnet                  = 10.0.1.0/28 
GatewaySubnet           = 10.0.0.0/27
LocalNetworkGatewayName = LocalSite
LNG Public IP           = <VPN device IP address> 
Local Address Prefixes  = 10.0.0.0/24','20.0.0.0/24
Gateway Name            = vnetgw1
PublicIP                = gwpip
Gateway IP Config       = gwipconfig1 
VPNType                 = RouteBased 
GatewayType             = Vpn 
ConnectionName          = myGWConnection
```

## <a name="Login"></a>1.连接到订阅
确保切换到 PowerShell 模式，以便使用资源管理器 cmdlet。 有关详细信息，请参阅 [将 Windows PowerShell 与资源管理器配合使用](../powershell-azure-resource-manager.md)。

1. 打开 PowerShell 控制台并连接到你的帐户。 使用下面的示例来帮助连接：

  ```powershell
  Login-AzureRmAccount
  ```
2. 检查该帐户的订阅。

  ```powershell
  Get-AzureRmSubscription
  ```
3. 指定要使用的订阅。

  ```powershell
  Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"
  ```

## <a name="VNet"></a>2.创建虚拟网络和网关子网

如果还没有虚拟网络，请创建。 创建虚拟网络时，请确保你指定的地址空间不与本地网络的任一个地址空间相重叠。 对于此配置，你还需要一个网关子网。 虚拟网关使用的网关子网包含 VPN 网关服务使用的 IP 地址。 创建网关子网时，必须将其命名为“GatewaySubnet”。 如果将其命名为其他名称，也将创建子网，但 Azure 不将它视为网关子网。

指定的网关子网的大小取决于要创建的 VPN 网关配置。 尽管创建的网关子网最小可为 /29，但建议选择 /27 或 /28，创建包含更多地址的更大子网。 使用更大的网关子网可以有足够的 IP 地址来应对未来可能会有的配置。

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

### <a name="to-create-a-virtual-network-and-a-gateway-subnet"></a>若要创建虚拟网络和网关子网

此示例创建虚拟网络和网关子网。 如果已经有一个虚拟网络且需要向其添加网关子网，请参阅[向已创建的虚拟网络添加网关子网](#gatewaysubnet)。

创建资源组：

```powershell
New-AzureRmResourceGroup -Name testrg -Location 'West US'
```

创建虚拟网络。 

1. 设置变量。

  ```powershell
  $subnet1 = New-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.0.0/27
  $subnet2 = New-AzureRmVirtualNetworkSubnetConfig -Name 'Subnet1' -AddressPrefix '10.0.1.0/28'
  ```
2. 创建 VNet。

  ```powershell
  New-AzureRmVirtualNetwork -Name testvnet -ResourceGroupName testrg `
  -Location 'West US' -AddressPrefix 10.0.0.0/16 -Subnet $subnet1, $subnet2
  ```

### <a name="gatewaysubnet"></a>将网关子网添加到已创建的虚拟网络

1. 设置变量。

  ```powershell
  $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName testrg -Name testvnet
  ```
2. 创建网关子网。

  ```powershell
  Add-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.0.0/27 -VirtualNetwork $vnet
  ```
3. 设置配置。

  ```powershell
  Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
  ```

## 3.<a name="localnet"></a>创建本地网关

本地网络网关通常是指本地位置。 你可以为站点提供一个名称供 Azure 引用，然后指定本地 VPN 设备的 IP 地址，以便创建一个连接来连接到该设备。 此外还可指定 IP 地址前缀，以便通过 VPN 网关将其路由到 VPN 设备。 指定的地址前缀是位于本地网络的前缀。 如果本地网络出现变化，可以轻松更新这些前缀。

使用以下值：

* *GatewayIPAddress* 是本地 VPN 设备的 IP 地址。 VPN 设备不能位于 NAT 之后。
* *AddressPrefix* 是本地地址空间。

- 若要添加具有单个地址前缀的局域网网关：

  ```powershell
  New-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
  -Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix   '10.0.0.0/24'
  ```

- 若要添加具有多个地址前缀的局域网网关：

  ```powershell
  New-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
  -Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix @('10.0.0.0/24','20.0.0.0/24')
  ```

- 若要为本地网关修改 IP 地址前缀：<br>
有时你的局域网网关前缀会有变化。 修改你的 IP 地址前缀时采取的步骤取决于是否已创建 VPN 网关连接。 请参阅本文的 [修改本地网关的 IP 地址前缀](#modify) 部分。

## <a name="PublicIP"></a>4.请求公共 IP 地址

请求一个公共 IP 地址，该地址将分配给虚拟网络 VPN 网关。 这是 IP 地址，你将 VPN 设备配置为连接到该地址。

Resource Manager 部署模型的虚拟网关目前使用动态分配方法，仅支持公共 IP 地址。 但是，这并不意味着 IP 地址会更改。 VPN 网关 IP 地址只在删除或重新创建网关时更改。 虚拟网关公共 IP 地址不会因为重新调整大小、重置或其他 VPN 网关内部维护/升级而更改。

使用以下 PowerShell 示例：

```powershell
$gwpip= New-AzureRmPublicIpAddress -Name gwpip -ResourceGroupName testrg -Location 'West US' -AllocationMethod Dynamic
```

## <a name="GatewayIPConfig"></a>5.创建网关 IP 寻址配置
网关配置定义要使用的子网和公共 IP 地址。 使用以下示例创建网关配置：

```powershell
$vnet = Get-AzureRmVirtualNetwork -Name testvnet -ResourceGroupName testrg
$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
$gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id
```

## <a name="CreateGateway"></a>6.创建 VPN 网关

创建虚拟网络 VPN 网关。 创建 VPN 网关可能需要长达 45 分钟或更长时间才能完成。

使用以下值：

* 站点到站点配置的 *-GatewayType* 为 *Vpn*。 网关类型永远是你要实现的配置的特定类型。 例如，其他网关配置可能需要 -GatewayType ExpressRoute。
* *-VpnType* 可以是 *RouteBased*（在某些文档中称为动态网关）或 *PolicyBased*（在某些文档中称为静态网关）。 有关 VPN 网关类型的详细信息，请参阅[关于 VPN 网关](vpn-gateway-about-vpngateways.md)。
* *-GatewaySku* 可以是 Basic、Standard 或 HighPerformance。 某些 SKU 存在配置限制。 有关详细信息，请参阅[网关 SKU](vpn-gateway-about-vpngateways.md#gateway-skus)。

```powershell
New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn `
-VpnType RouteBased -GatewaySku Standard
```

## <a name="ConfigureVPNDevice"></a>7.配置 VPN 设备

[!INCLUDE [vpn-gateway-configure-vpn-device-rm](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]

若要使用 PowerShell 查找虚拟网关的公共 IP 地址，请使用以下示例：

```powershell
Get-AzureRmPublicIpAddress -Name GW1PublicIP -ResourceGroupName TestRG
```

## <a name="CreateConnection"></a>8.创建 VPN 连接
接下来，将在虚拟网络网关和 VPN 设备之间创建站点到站点 VPN 连接。 请务必替换为你自己的值。 共享密钥必须与你用于 VPN 设备配置的值匹配。 请注意，站点到站点的“-ConnectionType”为 *IPsec*。

1. 设置变量。
  ```powershell
  $gateway1 = Get-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
  $local = Get-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg
  ```

2. 创建连接。
  ```powershell
  New-AzureRmVirtualNetworkGatewayConnection -Name MyGWConnection -ResourceGroupName testrg `
  -Location 'West US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
  -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
  ```

在一小段时间后，将建立该连接。

## <a name="toverify"></a>9.验证 VPN 连接
VPN 连接有几种不同的验证方式。

[!INCLUDE [vpn-gateway-verify-connection-ps-rm](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="modify"></a>修改本地网关的 IP 地址前缀
如果需要更改局域网网关的前缀，请使用以下说明。 提供了两套说明。 要选择哪套说明取决于您是否已创建了网关连接。

[!INCLUDE [vpn-gateway-modify-ip-prefix-rm](../../includes/vpn-gateway-modify-ip-prefix-rm-include.md)]

## <a name="modifygwipaddress"></a>修改本地网关的 IP 地址
[!INCLUDE [vpn-gateway-modify-lng-gateway-ip-rm](../../includes/vpn-gateway-modify-lng-gateway-ip-rm-include.md)]

## <a name="next-steps"></a>后续步骤
*  连接完成后，即可将虚拟机添加到虚拟网络。 有关详细信息，请参阅[虚拟机](https://docs.microsoft.com/azure/#pivot=services&panel=Compute)。
* 有关 BGP 的信息，请参阅 [BGP 概述](vpn-gateway-bgp-overview.md)和[如何配置 BGP](vpn-gateway-bgp-resource-manager-ps.md)。

