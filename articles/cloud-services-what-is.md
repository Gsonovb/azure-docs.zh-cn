<properties linkid="manage-services-what-is-a-cloud-service" urlDisplayName="What is a Cloud Service" pageTitle="What is a cloud service - Azure service management" metaKeywords="Azure cloud services intro, cloud services overview, cloud services basics" description="An introduction to the cloud service in Azure." metaCanonical="" services="cloud-services" documentationCenter="" title="What is a cloud service?" authors="ryanwi" solutions="" manager="" editor="" />

# 什么是云服务？

当你创建应用程序以及在 Azure 中运行该应用程序时，相关代码和配置统称为 Azure 云服务（在早期版本的 Azure 中称为*托管服务*）。

通过创建云服务，你可以在 Azure 中部署多层应用程序，以便定义用于处理分发以及允许灵活扩展你的应用程序的多个角色。云服务由一个或多个 Web 角色和/或辅助角色组成，其中每个角色都具有其自己的应用程序文件和配置。

Azure 可为你维护云服务基础结构，以便执行日常维护、修补操纵系统以及尝试从服务和硬件故障中恢复。如果你为每个角色至少定义了两个实例，则可在不中断服务的情况下，完成大多数维护和服务升级任务。云服务的每个角色必须至少具有两个实例才能符合 Azure 服务级别协议，从而确保你的面向 Internet 的角色至少在 99.95% 的时间内能够建立外部连接。

每个云服务都具有两个环境，你可以将服务包和配置部署到这两个环境。在将云服务升级到生产环境之前，你可以将其部署到过渡环境以对其进行测试。将暂存的云服务升级到生产环境的过程较为简单，只需交换与这两个环境关联的虚拟 IP 地址 (VIP)。

## 概念

-   **云服务角色：**云服务角色由应用程序文件和配置组成。云服务可以具有两种类型的角色：

> -   <b>Web 角色：</b>Web 角色提供专门用于托管前端 Web 应用程序的 Internet Information Services (IIS) Web 服务器。

> -   **辅助角色：**辅助角色中托管的应用程序可运行独立于用户交互或输入的异步任务、运行时间较长的任务或永久性任务。

-   **角色实例：**角色实例是可在其上运行应用程序代码和角色配置的虚拟机。根据服务配置文件中的定义，一个角色可以具有多个实例。

-   **来宾操作系统：**云服务的来宾操作系统是安装在可在其上运行应用程序代码的角色实例（虚拟机）上的操作系统。

-   **云服务组件：**要将应用程序部署为 Azure 中的云服务，需要以下三个组件：

> -   **服务定义文件：**云服务定义文件 (.csdef) 可定义服务模型，包括角色数量。

> -   **服务配置文件：**云服务配置文件 (.cscfg) 提供云服务和各个角色的配置设置，包括角色实例的数量。

> -   **服务包：**服务包 (.cspkg) 包含应用程序代码和服务定义文件。

-   **云服务部署：**云服务部署是部署到 Azure 过渡环境或生产环境的云服务的实例。你可以同时在过渡环境和生产环境中维护部署。

-   <b>部署环境：</b>Azure 提供两种云服务部署环境：*过渡环境*：在该环境中，你可以在将部署升级到*生产环境* 之前，测试该部署。只能通过用于访问云服务的虚拟 IP 地址 (VIP) 来区分这两个环境。在过渡环境中，使用云服务的全局唯一标识符 (GUID) 在 URL 中标识该服务 (*GUID*.cloudapp.net)。在生产环境中，URL 基于分配给云服务的更加友好的 DNS 前缀（例如，*myservice*.cloudapp.net）。

-   **交换部署：**若要将 Azure 过渡环境中的部署升级到生产环境，你可以通过交换用于访问这两个部署的 VIP 来“交换”这些部署。在部署后，云服务的 DNS 名称将指向曾经位于过渡环境中的部署。

-   **最少监控与详细监控：** *最少监控* 是云服务的默认配置，该配置使用从角色实例（虚拟机）的主机操作系统中收集到的性能计数器数据。*详细监控* 根据角色实例中的性能数据收集其他度量信息，以便对处理应用程序期间出现的问题进行进一步分析。有关详细信息，请参阅[如何监视云服务][]。

-   <b>Azure 诊断：</b>Azure 诊断是一个 API，它使你能够从运行在 Azure 中的应用程序中收集诊断数据。必须为云服务角色启用 Azure 诊断，然后才能启动详细监视。

-   **链接资源：**若要揭示云服务对其他资源（如 Azure SQL Database 实例）的依赖性，你可以将资源链接到云服务。在预览版管理门户中，你可以在“链接的资源”页上查看链接资源，在仪表板上查看资源状态，以及在“扩展”页上扩展链接的 SQL Database 实例和服务角色。在这种情况下链接资源时并不会将资源连接到应用程序；你必须在应用程序代码中配置相关连接。

-   **扩展云服务：**通过增加为角色部署的角色实例（虚拟机）的数量，可以向外扩展云服务。通过减少角色实例数，可以向内扩展云服务。在预览版管理门户中，通过更改 SQL Database 版本和最大数据库大小，你还可以在扩展服务角色时扩展链接的 SQL Database 实例。

-   <b>Azure 服务级别协议 (SLA)：</b>Azure 计算 SLA 保证，当你为每个角色部署两个或两个以上角色实例时，将至少在 99.95 % 的时间内确保他人可以访问你的云服务。此外，当某角色实例过程未运行时，可确保在 99.9% 的情况下可启动检测和纠正操作。有关详细信息，请参阅[服务级别协议][]。

  [如何监视云服务]: https://www.windowsazure.com/zh-cn/manage/services/cloud-services/how-to-monitor-a-cloud-service/
  [服务级别协议]: https://www.windowsazure.com/zh-cn/support/legal/sla/