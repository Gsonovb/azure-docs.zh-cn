---
title: "为高级 Azure Redis 缓存配置虚拟网络 | Microsoft 文档"
description: "了解如何为高级层 Azure Redis 缓存实例创建和管理虚拟网络支持"
services: redis-cache
documentationcenter: 
author: steved0x
manager: douge
editor: 
ms.assetid: 8b1e43a0-a70e-41e6-8994-0ac246d8bf7f
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: article
ms.date: 04/12/2017
ms.author: sdanie
translationtype: Human Translation
ms.sourcegitcommit: c300ba45cd530e5a606786aa7b2b254c2ed32fcd
ms.openlocfilehash: cf3c1a3c669e0da810c32939492cb262e76492c7
ms.lasthandoff: 04/14/2017


---
# <a name="how-to-configure-virtual-network-support-for-a-premium-azure-redis-cache"></a>如何为高级 Azure Redis 缓存配置虚拟网络支持
Azure Redis 缓存具有不同的缓存产品/服务，从而在缓存大小和功能的选择上具有灵活性，其中包括高级层功能，如群集、暂留和虚拟网络支持。 VNet 是云中的专用网络。 为 Azure Redis 缓存实例配置了 VNet 后，该实例不可公开寻址，而只能从 VNet 中的虚拟机和应用程序进行访问。 本文说明如何为高级 Azure Redis 缓存实例配置虚拟网络支持。

> [!NOTE]
> Azure Redis 缓存同时支持经典 VNet 和资源管理器 VNet。
> 
> 

有关其他高级缓存功能的信息，请参阅 [Azure Redis 缓存高级层简介](cache-premium-tier-intro.md)。

## <a name="why-vnet"></a>为何使用 VNet？
[Azure 虚拟网络 (VNet)](https://azure.microsoft.com/services/virtual-network/) 部署增强了 Azure Redis 缓存的安全性和隔离性，并提供子网、访问控制策略以及其他进一步限制访问的功能。

## <a name="virtual-network-support"></a>虚拟网络支持
在创建缓存期间，可在“新建 Redis 缓存”边栏选项卡中配置虚拟网络 (VNet) 支持。 

[!INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]

选择高级定价层后，可以通过选择与缓存相同的订阅和位置的 VNet 来配置 Redis VNet 集成。 若要使用新 VNet，请先创建 VNet，方法是遵循[使用 Azure 门户创建虚拟网络](../virtual-network/virtual-networks-create-vnet-arm-pportal.md)或[使用 Azure 门户创建虚拟网络（经典））](../virtual-network/virtual-networks-create-vnet-classic-portal.md)中的步骤，然后返回“新 Redis 缓存”边栏选项卡来创建和配置高级缓存。

若要为新缓存配置 VNet，请单击“新建 Redis 缓存”边栏选项卡上的“虚拟网络”，然后从下拉列表中选择所需的 VNet。

![虚拟网络][redis-cache-vnet]

从“子网”下拉列表中选择所需的子网，然后指定所需的“静态 IP 地址”。 如果使用经典 VNet，则“静态 IP 地址”字段是可选的；如果未指定任何地址，将从选定的子网中选择一个。

> [!IMPORTANT]
> 将 Azure Redis 缓存部署到资源管理器 VNet 时，缓存必须位于专用子网中，其中只能包含 Azure Redis 缓存实例，而不能包含其他任何资源。 如果尝试将 Azure Redis 缓存部署到包含其他资源的资源管理器 VNet 子网，部署将会失败。
> 
> 

![虚拟网络][redis-cache-vnet-ip]

> [!IMPORTANT]
> Azure 会保留每个子网中的某些 IP 地址，但是这些地址不能使用。 子网的第一个和最后一个 IP 地址仅为协议一致性而保留，其他三个地址用于 Azure 服务。 有关详细信息，请参阅[使用这些子网中的 IP 地址是否有任何限制？](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)
> 
> 除了 Azure VNET 基础结构使用的 IP 地址外，子网中的每个 Redis 实例还会针对负载均衡器使用每分片两个 IP 地址和一个附加 IP 地址。 非群集缓存视为具有一个分片。
> 
> 

创建缓存之后，可以在“资源菜单”中单击“虚拟网络”，查看 VNet 的配置。

![虚拟网络][redis-cache-vnet-info]

若要在使用 VNet 时连接到 Azure Redis 缓存实例，请在连接字符串中指定缓存的主机名，如以下示例所示：

    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
    {
        return ConnectionMultiplexer.Connect("contoso5premium.redis.cache.windows.net,abortConnect=false,ssl=true,password=password");
    });

    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }

## <a name="azure-redis-cache-vnet-faq"></a>Azure Redis 缓存 VNet 常见问题
以下列表包含有关 Azure Redis 缓存缩放的常见问题的解答。

* [Azure Redis 缓存和 VNet 有哪些常见的错误配置问题？](#what-are-some-common-misconfiguration-issues-with-azure-redis-cache-and-vnets)
* [是否可以通过标准或基本缓存使用 VNet？](#can-i-use-vnets-with-a-standard-or-basic-cache)
* [为什么在某些子网中创建 Redis 缓存失败，而在其他子网中不会失败？](#why-does-creating-a-redis-cache-fail-in-some-subnets-but-not-others)
* [子网地址空间有哪些要求？](#what-are-the-subnet-address-space-requirements)
* [在 VNET 中托管缓存时，是否可以使用所有缓存功能？](#do-all-cache-features-work-when-hosting-a-cache-in-a-vnet)

## <a name="what-are-some-common-misconfiguration-issues-with-azure-redis-cache-and-vnets"></a>Azure Redis 缓存和 VNet 有哪些常见的错误配置问题？
在 VNet 中托管 Azure Redis 缓存时，会使用下表中的端口。 如果这些端口受阻，则缓存可能无法正常工作。 在 VNet 中使用 Azure Redis 缓存时，阻止这些端口中的一个或多个是最常见的错误配置问题。

| 端口 | Direction | 传输协议 | 目的 | 远程 IP |
| --- | --- | --- | --- | --- |
| 80、443 |出站 |TCP |Azure 存储空间/PKI (Internet) 上的 Redis 依赖关系 |* |
| 53 |出站 |TCP/UDP |DNS (Internet/VNet) 上的 Redis 依赖关系 |* |
| 6379、6380 |入站 |TCP |与 Redis 的客户端通信、Azure 负载均衡 |VIRTUAL_NETWORK、AZURE_LOADBALANCER |
| 8443 |入站/出站 |TCP |Redis 的实现详细信息 |VIRTUAL_NETWORK |
| 8500 |入站 |TCP/UDP |Azure 负载均衡 |AZURE_LOADBALANCER |
| 10221-10231 |入站/出站 |TCP |Redis 的实现详细信息（可以将远程终结点限制为 VIRTUAL_NETWORK） |VIRTUAL_NETWORK、AZURE_LOADBALANCER |
| 13000-13999 |入站 |TCP |与 Redis 群集的客户端通信、Azure 负载均衡 |VIRTUAL_NETWORK、AZURE_LOADBALANCER |
| 15000-15999 |入站 |TCP |与 Redis 群集的客户端通信、Azure 负载均衡 |VIRTUAL_NETWORK、AZURE_LOADBALANCER |
| 16001 |入站 |TCP/UDP |Azure 负载均衡 |AZURE_LOADBALANCER |
| 20226 |入站+出站 |TCP |Redis 群集的实现详细信息 |VIRTUAL_NETWORK |

在虚拟网络中，可能一开始不符合 Azure Redis 缓存的网络连接要求。 在虚拟网络中使用时，Azure Redis 缓存需要以下所有项才能正常运行。

* 与全球 Azure 存储空间终结点建立的出站网络连接。 这包括位于 Azure Redis 缓存实例区域的终结点，以及位于**其他** Azure 区域的存储终结点。 Azure 存储终结点在以下 DNS 域下解析：*table.core.windows.net*、*blob.core.windows.net*、*queue.core.windows.net* 和 *file.core.windows.net*。 
* 与 *ocsp.msocsp.com*、*mscrl.microsoft.com* 和 *crl.microsoft.com* 建立的出站网络连接。 需要此连接才能支持 SSL 功能。
* 虚拟网络的 DNS 设置必须能够解析前面几点所提到的所有终结点和域。 确保已针对虚拟网络配置并维护有效的 DNS 基础结构即可符合这些 DNS 要求。
* 到以下 Azure 监视终结点的出站网络连接，这些终结点在下列 DNS 域下进行解析：shoebox2-black.shoebox2.metrics.nsatc.net、north-prod2.prod2.metrics.nsatc.net、azglobal-black.azglobal.metrics.nsatc.net、shoebox2-red.shoebox2.metrics.nsatc.net、east-prod2.prod2.metrics.nsatc.net、azglobal-red.azglobal.metrics.nsatc.net。

### <a name="can-i-use-vnets-with-a-standard-or-basic-cache"></a>是否可以对标准或基本缓存使用 VNet？
只能对高级缓存使用 VNet。

### <a name="why-does-creating-a-redis-cache-fail-in-some-subnets-but-not-others"></a>为什么在某些子网中创建 Redis 缓存失败，而在其他子网中不会失败？
如果要将 Azure Redis 缓存部署到资源管理器 VNet，缓存必须位于不包含任何其他资源类型的专用子网中。 如果尝试将 Azure Redis 缓存部署到包含其他资源的资源管理器 VNet 子网，部署将失败。 必须先删除该子网中的现有资源，然后才能创建新的 Redis 缓存。

只要你有足够的可用 IP 地址，就可以将多种类型的资源部署到经典 VNet。

### <a name="what-are-the-subnet-address-space-requirements"></a>子网地址空间有哪些要求？
Azure 会保留每个子网中的某些 IP 地址，但是这些地址不能使用。 子网的第一个和最后一个 IP 地址仅为协议一致性而保留，其他三个地址用于 Azure 服务。 有关详细信息，请参阅[使用这些子网中的 IP 地址是否有任何限制？](../virtual-network/virtual-networks-faq.md#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)

除了 Azure VNET 基础结构使用的 IP 地址外，子网中的每个 Redis 实例还会针对负载均衡器使用每分片两个 IP 地址和一个附加 IP 地址。 非群集缓存视为具有一个分片。

### <a name="do-all-cache-features-work-when-hosting-a-cache-in-a-vnet"></a>在 VNET 中托管缓存时，是否可以使用所有缓存功能？
如果缓存是 VNET 的一部分，则只有 VNET 中的客户端可以访问缓存。 因此，以下缓存管理功能目前不起作用。

* Redis 控制台 - 由于 Redis 控制台在本地浏览器中运行（这在 VNET 的外部），因此它无法连接到缓存。

## <a name="use-expressroute-with-azure-redis-cache"></a>将 ExpressRoute 用于 Azure Redis 缓存
客户可以将 [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/) 线路连接到虚拟网络基础结构，从而将其本地网络扩展到 Azure。 

默认情况下，新创建的 ExpressRoute 线路将会通告允许出站 Internet 连接的默认路由。 使用此配置，客户端应用程序将能够连接到其他 Azure 终结点（包括 Azure Redis 缓存）。

但是，常见的客户配置是定义其自身的默认路由 (0.0.0.0/0)，以强制出站 Internet 流量改为流向本地。 如果出站流量在本地遭到阻止，此流量将断开与 Azure Redis 缓存的连接，这样 Azure Redis 缓存实例将无法与其依赖项通信。

解决方法是在包含 Azure Redis 缓存的子网上定义一个或多个用户定义的路由 (UDR)。 UDR 定义了要遵循的子网特定路由，而不是默认路由。

如果可能，建议使用以下配置：

* ExpressRoute 配置播发 0.0.0.0/0 并默认使用强制隧道将所有输出流量发送到本地。
* 已应用到包含 Azure Redis 缓存的子网的 UDR 使用公共 Internet 的 TCP/IP 流量工作路由来定义 0.0.0.0/0；例如，通过将下一个跃点类型设置为“Internet”。

这些步骤的组合效应是子网级 UDR 将优先于 ExpressRoute 强制隧道，从而确保来自 Azure Redis 缓存的出站 Internet 访问。

由于性能原因，从本地应用程序使用 ExpressRoute 连接到 Azure Redis 缓存实例不是典型使用方案（为了获得最佳性能，Azure Redis 缓存客户端应与 Azure Redis 缓存位于同一区域中）。

>[!IMPORTANT] 
>UDR 中定义的路由**必须**足够明确，以便优先于 ExpressRoute 配置所播发的任何路由。 以下示例使用广泛 0.0.0.0/0 地址范围，因此使用更明确的地址范围，有可能意外地被路由播发重写。

>[!WARNING]  
>**从公共对等路径到专用对等路径未正确交叉播发路由**的 ExpressRoute 配置不支持 Azure Redis 缓存。 已配置公共对等互连的 ExpressRoute 配置会收到来自 Microsoft 的大量 Microsoft Azure IP 地址范围的路由播发。 如果这些地址范围在专用对等路径上未正确交叉播发，则结果是来自 Azure Redis 缓存实例子网的所有出站网络数据包都不会正确地使用强制隧道发送到客户的本地网络基础结构。 此网络流将破坏 Azure Redis 缓存。 此问题的解决方法是停止从公共对等路径到专用对等路径的交叉播发路由。


有关用户定义路由的背景信息，请参阅此[概述](../virtual-network/virtual-networks-udr-overview.md)。

有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 技术概述](../expressroute/expressroute-introduction.md)。

## <a name="next-steps"></a>后续步骤
了解如何使用更多的高级版缓存功能。

* [Azure Redis 缓存高级层简介](cache-premium-tier-intro.md)

<!-- IMAGES -->

[redis-cache-vnet]: ./media/cache-how-to-premium-vnet/redis-cache-vnet.png

[redis-cache-vnet-ip]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-ip.png

[redis-cache-vnet-info]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-info.png


