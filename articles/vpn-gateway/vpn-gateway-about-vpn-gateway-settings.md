---
title: "跨界 Azure 连接的 VPN 网关设置 | Microsoft Docs"
description: "了解 Azure 虚拟网络网关的 VPN 网关设置。"
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: 
tags: azure-resource-manager,azure-service-management
ms.assetid: ae665bc5-0089-45d0-a0d5-bc0ab4e79899
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/13/2017
ms.author: cherylmc
translationtype: Human Translation
ms.sourcegitcommit: b902d2e79633959a6f76ddd45b1193177b0e8465
ms.openlocfilehash: 1ac5a78c8d9419e4c641bf66f8dac7aa8cbcd179


---
# <a name="about-vpn-gateway-configuration-settings"></a>关于 VPN 网关配置设置
VPN 网关是一种虚拟网络网关，可跨公共连接在虚拟网络和本地位置发送加密的流量。 还可使用 VPN 网关跨 Azure 主干网在虚拟网络间发送流量。

VPN 网关连接依赖于多个资源配置，其中每个资源包含可配置的设置。 本文的各部分介绍了与在 Resource Manager 部署模型中创建的虚拟网络的 VPN 网关相关的资源和设置。 可在 [关于 VPN 网关](vpn-gateway-about-vpngateways.md)一文中找到每种连接解决方案的介绍和拓扑图。  

## <a name="a-namegwtypeagateway-types"></a><a name="gwtype"></a>网关类型
每个虚拟网络只能有一种类型的虚拟网络网关。 创建虚拟网络网关时，必须确保用于配置的网关类型正确。

-GatewayType 的可用值为： 

* Vpn
* ExpressRoute

VPN 网关需要 `-GatewayType` *Vpn*。  

示例：

    New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
    -Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn `
    -VpnType RouteBased


## <a name="a-namegwskuagateway-skus"></a><a name="gwsku"></a>网关 SKU
[!INCLUDE [vpn-gateway-gwsku-include](../../includes/vpn-gateway-gwsku-include.md)]

### <a name="configuring-the-gateway-sku"></a>配置网关 SKU
####<a name="specifying-the-gateway-sku-in-the-azure-portal"></a>在 Azure 门户中指定网关 SKU

如果使用 Azure 门户创建资源管理器虚拟网络网关，可以使用下拉列表选择网关 SKU。 显示的选项对应于所选的网关类型和 VPN 类型。

例如，如果选择“VPN”作为网关类型，选择“基于策略”作为 VPN 类型，只能看到“基本”SKU，因为这是“基于策略”SKU 唯一可用的 SKU。 如果选择“基于路由”，可以从“基本”、“标准”和“高性能”SKU 中选择。 

####<a name="specifying-the-gateway-sku-using-powershell"></a>使用 PowerShell 指定网关 SKU

以下 PowerShell 示例将 `-GatewaySku` 指定为*标准*。

    New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
    -Location 'West US' -IpConfigurations $gwipconfig -GatewaySku Standard `
    -GatewayType Vpn -VpnType RouteBased

####<a name="changing-a-gateway-sku"></a>更改网关 SKU

如果想要将网关 SKU 升级到更强大的 SKU（从“基本”/“标准”升级到“高性能”），可以使用 `Resize-AzureRmVirtualNetworkGateway` PowerShell cmdlet。 也可以使用此 cmdlet 下调网关 SKU 大小。

以下 PowerShell 示例演示如何将网关 SKU 的大小调整为“高性能”。

    $gw = Get-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg
    Resize-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku HighPerformance

### <a name="estimated-aggregate-throughput-by-gateway-sku-and-type"></a>按网关 SKU 和类型列出的估计聚合吞吐量
[!INCLUDE [vpn-gateway-table-gwtype-aggthroughput](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]

## <a name="a-nameconnectiontypeaconnection-types"></a><a name="connectiontype"></a>连接类型
在资源管理器部署模型中，每个配置都需要特定的虚拟网络网关连接类型。 `-ConnectionType` 的可用 Resource Manager PowerShell 值为：

* IPsec
* Vnet2Vnet
* ExpressRoute
* VPNClient

在以下 PowerShell 示例中，我们将创建需要 *IPsec* 连接类型的 S2S 连接。

    New-AzureRmVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg `
    -Location 'West US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
    -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'


## <a name="a-namevpntypeavpn-types"></a><a name="vpntype"></a>VPN 类型
为 VPN 网关配置创建虚拟网络网关时，必须指定 VPN 类型。 选择的 VPN 类型取决于要创建的连接拓扑。 例如，P2S 连接需要 RouteBased VPN 类型。 VPN 类型还取决于要使用的硬件。 S2S 配置需要 VPN 设备。 有些 VPN 设备仅支持特定的 VPN 类型。

选择的 VPN 类型必须满足所要创建的解决方案的所有连接要求。 例如，如果要为同一虚拟网络创建 S2S VPN 网关连接和 P2S VPN 网关连接，应使用 VPN 类型*基于路由*，因为 P2S 需要“基于路由”VPN 类型。 此外，需确认 VPN 设备支持 RouteBased VPN 连接。 

创建虚拟网络网关后，无法更改 VPN 类型。 必须删除虚拟网络网关，然后新建一个。 有两种 VPN 类型：

[!INCLUDE [vpn-gateway-vpntype](../../includes/vpn-gateway-vpntype-include.md)]

以下 PowerShell 示例将 `-VpnType` 指定为*基于路由*。 在创建网关时，你必须确保用于配置的 -VpnType 正确。 

    New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
    -Location 'West US' -IpConfigurations $gwipconfig `
    -GatewayType Vpn -VpnType RouteBased

## <a name="a-namerequirementsagateway-requirements"></a><a name="requirements"></a>网关要求
[!INCLUDE [vpn-gateway-table-requirements](../../includes/vpn-gateway-table-requirements-include.md)]

## <a name="a-namegwsubagateway-subnet"></a><a name="gwsub"></a>网关子网
若要为 VNet 配置虚拟网络网关，需要先创建网关子网。 网关子网包含虚拟网络网关服务使用的 IP 地址。 网关子网必须命名为 *GatewaySubnet* 才能正常工作。 此名称可以让 Azure 知道此子网将用于网关。

创建网关子网时，请指定子网包含的 IP 地址数。 网关子网中的 IP 地址分配到网关服务。 相对于其他配置，某些配置需要将更多 IP 地址分配到网关服务。 需确保网关子网包含足够多的 IP 地址，以便应对将来的业务增长以及可能会添加的新连接配置。 因此，尽管网关子网最小可创建为 /29，但建议创建 /28 或更大（/28、/27 和 /26 等）的网关子网。 查看要创建的配置的要求，并验证网关子网是否会满足这些要求。

以下 Resource Manager PowerShell 示例显示名为 GatewaySubnet 的网关子网。 可以看到，CIDR 表示法指定了 /27，这可提供足够的 IP 地址供大多数现有配置使用。

    Add-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/27

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="a-namelngalocal-network-gateways"></a><a name="lng"></a>本地网关
创建 VPN 网关配置时，本地网络网关通常代表本地位置。 在经典部署模型中，本地网络网关称为本地站点。 

指定本地网络网关的名称（即本地 VPN 设备的公共 IP 地址），并指定位于本地位置的地址前缀。 Azure 将查看网络流量的目标地址前缀、查阅针对本地网络网关指定的配置，并相应地路由数据包。 也应该针对使用 VPN 网关连接的 VNet 到 VNet 配置指定本地网络网关。

以下 PowerShell 示例创建新的本地网络网关：

    New-AzureRmLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
    -Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.5.51.0/24'

有时需要修改本地网络网关设置。 例如，在添加或修改地址范围时，或 VPN 设备的 IP 地址发生变化时。 对于经典 VNet，可以在经典门户上的“局域网”页上更改这些设置。 对于资源管理器，请参阅[使用 PowerShell 修改本地网络网关设置](vpn-gateway-modify-local-network-gateway.md)。

## <a name="a-nameresourcesarest-apis-and-powershell-cmdlets"></a><a name="resources"></a>REST API 和 PowerShell cmdlet
有关将 REST API 和 PowerShell cmdlet 用于 VPN 网关配置的其他技术资源和具体语法要求，请参阅以下页面：

| **经典** | **资源管理器** |
| --- | --- |
| [PowerShell](https://msdn.microsoft.com/library/mt270335.aspx) |[PowerShell](https://msdn.microsoft.com/library/mt163510.aspx) |
| [REST API](https://msdn.microsoft.com/library/jj154113.aspx) |[REST API](https://msdn.microsoft.com/library/mt163859.aspx) |

## <a name="next-steps"></a>后续步骤
有关可用连接配置的详细信息，请参阅[关于 VPN 网关](vpn-gateway-about-vpngateways.md)。 




<!--HONumber=Feb17_HO2-->


