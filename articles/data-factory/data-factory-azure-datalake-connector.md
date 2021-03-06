---
title: "将数据移入/移出 Azure Data Lake Store | Microsoft Docs"
description: "了解如何使用 Azure 数据工厂将数据移入/移出 Azure Data Lake Store"
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: monicar
ms.assetid: 25b1ff3c-b2fd-48e5-b759-bb2112122e30
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/30/2017
ms.author: jingwang
translationtype: Human Translation
ms.sourcegitcommit: e851a3e1b0598345dc8bfdd4341eb1dfb9f6fb5d
ms.openlocfilehash: 3f0575a170eb20d136858bedc1f87d4f4375c812
ms.lasthandoff: 04/15/2017


---
# <a name="move-data-to-and-from-azure-data-lake-store-using-azure-data-factory"></a>使用 Azure 数据工厂将数据移入和移出 Azure Data Lake Store
本文介绍如何使用 Azure 数据工厂中的复制活动将数据移入/移出 Azure Data Lake Store。 它基于[数据移动活动](data-factory-data-movement-activities.md)一文，其中总体概述了如何使用复制活动移动数据。

可将数据从任一支持的源数据存储移到 Azure Data Lake Store，或从 Azure Data Lake Store 移到任一支持的接收器数据存储。 有关复制活动支持作为源或接收器的数据存储列表，请参阅[支持的数据存储](data-factory-data-movement-activities.md#supported-data-stores-and-formats)表。  

> [!NOTE]
> 在使用复制活动创建管道之前，创建 Azure Data Lake Store 帐户，将数据移入/移出 Azure Data Lake store。 若要了解 Azure Data Lake Store，请参阅 [Azure Data Lake Store 入门](../data-lake-store/data-lake-store-get-started-portal.md)。

## <a name="supported-authentication-types"></a>支持的身份验证类型
Azure Data Lake Store 连接器支持**服务主体**身份验证和**用户凭据** (OAuth) 身份验证。 推荐使用前者（尤其对于计划的数据副本），以避免后者的令牌过期行为。 有关配置详细信息，请参阅[链接服务属性](#linked-service-properties)。

## <a name="getting-started"></a>入门
可以使用不同的工具/API 创建包含复制活动的管道，以将数据移入/移出 Azure Data Lake Store。

创建管道的最简单方法是使用**复制向导**。 请参阅[教程：使用复制向导创建管道](data-factory-copy-data-wizard-tutorial.md)，以快速了解如何使用复制数据向导创建管道。

也可以使用以下工具创建管道：**Azure 门户**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 模板**、**.NET API** 和 **REST API**。 有关创建包含复制活动的管道的分步说明，请参阅[复制活动教程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

无论使用工具还是 API，执行以下步骤都可创建管道，以便将数据从源数据存储移到接收器数据存储：

1. 创建**链接服务**可将输入和输出数据存储链接到数据工厂。
2. 创建**数据集**以表示复制操作的输入和输出数据。
3. 创建包含复制活动的**管道**，该活动将一个数据集作为输入，将一个数据集作为输出。

使用向导时，将自动为你创建这些数据工厂实体（链接服务、数据集和管道）的 JSON 定义。 使用工具/API（.NET API 除外）时，使用 JSON 格式定义这些数据工厂实体。  有关用于向/从 Azure Data Lake Store 复制数据的数据工厂实体的 JSON 定义示例，请参阅本文的 [JSON 示例](#json-examples)部分。

对于特定于 Azure Data Lake Store 的数据工厂实体，以下部分提供了有关用于定义这些实体的 JSON 属性的详细信息：

## <a name="linked-service-properties"></a>链接服务属性
链接服务可将数据存储链接到数据工厂。 创建 **AzureDataLakeStore** 类型的链接服务，以便将 Azure Data Lake Store 链接到数据工厂。 下表提供了特定于 Azure Data Lake Store 链接服务的 JSON 元素的说明，你可在**服务主体**和**用户凭据**身份验证之间进行选择。

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| type | type 属性必须设置为：**AzureDataLakeStore** | 是 |
| dataLakeStoreUri | 指定 Azure Data Lake Store 帐户相关信息。 其格式如下：`https://[accountname].azuredatalakestore.net/webhdfs/v1` 或 `adl://[accountname].azuredatalakestore.net/`。 | 是 |
| subscriptionId | Data Lake Store 所属的 Azure 订阅 ID。 | 接收器所需 |
| resourceGroupName | Data Lake Store 所属的 Azure 资源组名称。 | 接收器所需 |

### <a name="using-service-principal-authentication-recommended"></a>使用服务主体身份验证（推荐）
若要使用服务主体身份验证，请在 Azure Active Directory (AAD) 中注册应用程序实体并授予其访问 Data Lake Store 的权限。 有关详细步骤，请参阅[服务到服务身份验证](../data-lake-store/data-lake-store-authenticate-using-active-directory.md)。 请记下以下值：**应用程序 ID**、**应用程序密钥**和**租户 ID**。 定义链接的服务时需使用此信息。

> [!IMPORTANT]
> 如果使用“复制向导”创作数据管道，请确保在访问控制 (IAM) 中向服务主体至少授予对 Data Lake Store 帐户的“读取者”角色以及对 Data Lake Store 根 ("/") 及其子级的“读取+执行”权限。 否则，可能会看到“提供的凭据无效”错误。
>
> 在 AAD 中创建/更新服务主体后，所作更改可能需要几分钟才能生效。 首先，仔细检查服务主体和 Data Lake Store ACL 配置。 如果仍看到错误：“提供的凭据无效”，请稍等一会儿，然后重试。

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| servicePrincipalId | 指定应用程序的客户端 ID。 | 是 |
| servicePrincipalKey | 指定应用程序的密钥。 | 是 |
| tenant | 指定应用程序的租户信息（域名或租户 ID）。 可将鼠标悬停在 Azure 门户右上角进行检索。 | 是 |

**示例：使用服务主体身份验证**
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

### <a name="using-user-credential-authentication"></a>使用用户凭据身份验证
或者，通过指定以下属性，可使用用户凭据身份验证复制出/到 Data Lake Store：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| authorization | 单击“数据工厂编辑器”中的“授权”按钮，并输入将自动生成的授权 URL 分配给此属性的凭据。 | 是 |
| sessionId | OAuth 授权会话中的 OAuth 会话 ID。 每个会话 ID 都是唯一的，并且可能仅可使用一次。 使用数据工厂编辑器时，将自动生成此设置。 | 是 |

**示例：使用用户凭据身份验证**
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<session ID>",
            "authorization": "<authorization URL>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

#### <a name="token-expiration"></a>令牌过期
通过使用“授权”按钮生成的授权代码在一段时间后过期。 请参阅下表，了解不同类型用户帐户的过期时间。 身份验证**令牌过期**时，可能会看到以下错误消息：

```
"Credential operation error: invalid_grant - AADSTS70002: Error validating credentials. AADSTS70008: The provided access grant is expired or revoked. Trace ID: d18629e8-af88-43c5-88e3-d8419eb1fca1 Correlation ID: fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Timestamp: 2015-12-15 21-09-31Z".
```

| 用户类型 | 过期时间 |
|:--- |:--- |
| 不由 Azure Active Directory 管理的用户帐户（@hotmail.com、@live.com，等等）。 |12 小时 |
| 由 Azure Active Directory (AAD) 管理的用户帐户 |最后一次运行切片后的 14 天。 <br/><br/>如果以基于 OAuth 的链接服务为基础的切片每 14 天至少运行一次，则为 90 天。 |

如果在此令牌的过期时间之前更改密码，则该令牌将立即过期，并且你将看到此部分中所提到的错误。

若要避免/解决此错误，**令牌过期**时，使用“授权”按钮重新授权，并重新部署链接服务。 还可以使用以下部分中的代码以编程方式生成 **sessionId** 和 **authorization** 属性的值：

#### <a name="to-programmatically-generate-sessionid-and-authorization-values"></a>以编程方式生成 sessionId 和 authorization 值

```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```
请参阅 [AzureDataLakeStoreLinkedService 类](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice.aspx)、[AzureDataLakeAnalyticsLinkedService 类](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice.aspx)和 [AuthorizationSessionGetResponse 类](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse.aspx)主题，了解关于代码中使用的数据工厂类的详细信息。 针对代码中使用的 WindowsFormsWebAuthenticationDialog 类，向 **2.9.10826.1824** 版的 **Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll** 添加引用。

## <a name="dataset-properties"></a>数据集属性
若要指定数据集表示 Azure Data Lake Store 中的输入数据，可以将该数据集的类型属性设置为：**AzureDataLakeStore**。 将数据集的 **linkedServiceName** 属性设置为 Azure Data Lake Store 链接服务的名称。 有关可用于定义数据集的 JSON 部分和属性的完整列表，请参阅[创建数据集](data-factory-create-datasets.md)一文。 对于所有数据集类型（Azure SQL、Azure Blob、Azure 表等），结构、可用性和数据集 JSON 的策略等部分均类似。 每种数据集的 **typeProperties** 节有所不同，该部分提供有关数据在数据存储区中的位置、格式等信息。 **AzureDataLakeStore** 数据集类型的 typeProperties 部分具有以下属性：

| 属性 | 说明 | 必选 |
|:--- |:--- |:--- |
| folderPath |到 Azure Data Lake Sore 中的容器和文件夹的路径。 |是 |
| fileName |Azure Data Lake Sore 中文件的名称。 fileName 可选，并且区分大小写。 <br/><br/>如果指定 filename，则活动（包括复制）将对特定文件起作用。<br/><br/>如果未指定 fileName，则复制将包括输入数据集的 folderPath 中的所有文件。<br/><br/>如果没有为输出数据集指定 fileName，生成的文件的名称将采用以下格式：Data<Guid>.txt (for example: : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt |否 |
| partitionedBy |partitionedBy 是一个可选属性。 它可用于指定时序数据的动态 folderPath 和 filename。 例如，folderPath 可针对每小时的数据参数化。 请参阅[使用 partitionedBy 属性](#using-partitionedby-property)部分了解详细信息和示例。 |否 |
| 格式 | 支持以下格式类型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat** 和 **ParquetFormat**。 请将格式中的 **type** 属性设置为上述值之一。 有关详细信息，请参阅[文本格式](data-factory-supported-file-and-compression-formats.md#text-format)、[Json 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[Orc 格式](data-factory-supported-file-and-compression-formats.md#orc-format)和 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)部分。 <br><br> 如果想要在基于文件的存储之间**按原样复制文件**（二进制副本），可以在输入和输出数据集定义中跳过格式节。 |否 |
| compression | 指定数据的压缩类型和级别。 支持的类型为：**GZip**、**Deflate**、**BZip2** 和 **ZipDeflate**。 支持的级别为：**最佳**和**最快**。 有关详细信息，请参阅 [Azure 数据工厂中的文件和压缩格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

### <a name="using-partitionedby-property"></a>使用 partitionedBy 属性
如上一部分所述，可以使用 **partitionedBy** 属性、[数据工厂函数和系统变量](data-factory-functions-variables.md)指定时序数据的动态 folderPath 和 filename。

请参阅[创建数据集](data-factory-create-datasets.md)和[计划和执行](data-factory-scheduling-and-execution.md)文章，了解更多有关时序数据集、计划和切片的详细信息。

#### <a name="sample-1"></a>示例 1

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```
在本示例中，{Slice} 已替换为数据工厂系统变量 SliceStart 的值，格式为指定的指定的格式 (YYYYMMDDHH)。 SliceStart 指切片开始时间。 每个切片的 folderPath 不同。 例如：wikidatagateway/wikisampledataout/2014100103 or wikidatagateway/wikisampledataout/2014100104

#### <a name="sample-2"></a>示例 2
```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
 [
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```
在本示例中，SliceStart 的年、月、日和时间已提取到 folderPath 和 fileName 属性使用的各个变量。

## <a name="copy-activity-properties"></a>复制活动属性
有关可用于定义活动的各节和属性的完整列表，请参阅[创建管道](data-factory-create-pipelines.md)一文。 名称、说明、输入和输出表格等属性和策略可用于所有类型的活动。

而可用于此活动的 typeProperties 节的属性因每个活动类型而异。 对于复制活动，这些属性则因源和接收器的类型而异

**AzureDataLakeStoreSource** 支持以下属性 **typeProperties** 部分：

| 属性 | 说明 | 允许的值 | 必选 |
| --- | --- | --- | --- |
| recursive |指示是要从子文件夹中以递归方式读取数据，还是只从指定的文件夹中读取数据。 |True（默认值）、False |否 |

**AzureDataLakeStoreSink** 支持以下属性 **typeProperties** 部分：

| 属性 | 说明 | 允许的值 | 必选 |
| --- | --- | --- | --- |
| copyBehavior |指定复制行为。 |<b>PreserveHierarchy</b>：保留目标文件夹中的文件层次结构。 从源文件到源文件夹的相对路径与从目标文件到目标文件夹的相对路径相同。<br/><br/><b>FlattenHierarchy</b>：源文件夹中的所有文件在目标文件夹的第一个级别中创建。 创建目标文件时，自动生成名称。<br/><br/><b>MergeFiles</b>：将源文件夹的所有文件合并到一个文件中。 如果指定文件/Blob 名称，则合并的文件名称将为指定的名称；否则，将会自动生成文件名。 |否 |

## <a name="supported-file-and-compression-formats"></a>支持的文件和压缩格式
请参阅 [Azure 数据工厂中的文件和压缩格式](data-factory-supported-file-and-compression-formats.md)一文了解详细信息。

## <a name="json-examples"></a>JSON 示例
以下示例提供示例 JSON 定义，可使用该定义通过 [Azure 门户](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 创建管道。 它们演示如何将数据复制到 Azure Data Lake Store 和 Azure Blob 存储，以及如何复制其中的数据。 但是，可以从任何源将数据**直接**复制到任何受支持的接收器。 有关详细信息，请参阅[使用复制活动移动数据](data-factory-data-movement-activities.md)中的“支持的数据存储和格式”部分。  
## <a name="example-copy-data-from-azure-blob-to-azure-data-lake-store"></a>示例：将数据从 Azure Blob 复制到 Azure Data Lake Store
以下示例显示：

1. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties) 类型的链接服务。
2. [AzureDataLakeStore](#linked-service-properties) 类型的链接服务。
3. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 类型的输入[数据集](data-factory-create-datasets.md)。
4. [AzureDataLakeStore](#dataset-properties) 类型的输出[数据集](data-factory-create-datasets.md)。
5. 包含使用 [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) 和 [AzureDataLakeStoreSink](#copy-activity-properties) 的复制活动的[管道](data-factory-create-pipelines.md)。

此示例每隔一小时将时序数据从 Azure Blob 存储复制到 Azure Data Lake Store。 示例后续部分描述了这些示例中使用的 JSON 属性。

**Azure 存储链接服务：**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

**Azure Data Lake Store 链接服务**

```JSON
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

> [!NOTE]
> 有关配置详细信息，请参阅 [Azure Data Lake Store 链接服务属性](#linked-service-properties)部分中的步骤。
>

**Azure Blob 输入数据集：**

从新 blob 获取数据，每隔一小时进行一次（频率：小时，间隔：1）。 根据正在处理的切片的开始时间，将对 blob 的文件夹路径和文件名进行动态计算。 文件夹路径使用开始时间的年、月和日部分，文件名使用开始时间的小时部分。 “external”: “true” 设置将告知数据工厂服务：表位于数据工厂外且不由数据工厂中的活动生成。

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ]
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Azure Data Lake Store 输出数据集：**

此示例将数据复制到 Azure Data Lake Store。 每隔一小时将新的数据复制到 Data Lake Store。

```JSON
{
    "name": "AzureDataLakeStoreOutput",
      "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/output/"
        },
        "availability": {
              "frequency": "Hour",
              "interval": 1
        }
      }
}
```


**管道中使用 Blob 源和 Azure Data Lake Store 接收器的复制活动：**

管道包含配置为使用输入和输出数据集、且计划每小时运行一次的复制活动。 在管道 JSON 定义中，将 **source** 类型设置为 **BlobSource**，将 **sink** 类型设置为 **AzureDataLakeStoreSink**。

```json
{  
    "name":"SamplePipeline",
    "properties":
    {  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":
        [  
              {
                "name": "AzureBlobtoDataLake",
                "description": "Copy Activity",
                "type": "Copy",
                "inputs": [
                  {
                    "name": "AzureBlobInput"
                  }
                ],
                "outputs": [
                  {
                    "name": "AzureDataLakeStoreOutput"
                  }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                      },
                      "sink": {
                        "type": "AzureDataLakeStoreSink"
                      }
                },
                   "scheduler": {
                      "frequency": "Hour",
                      "interval": 1
                },
                "policy": {
                      "concurrency": 1,
                      "executionPriorityOrder": "OldestFirst",
                      "retry": 0,
                      "timeout": "01:00:00"
                }
              }
        ]
    }
}
```

## <a name="example-copy-data-from-azure-data-lake-store-to-azure-blob"></a>示例：将数据从 Azure Data Lake Store 复制到 Azure Blob
以下示例显示：

1. [AzureDataLakeStore](#linked-service-properties) 类型的链接服务。
2. [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties) 类型的链接服务。
3. [AzureDataLakeStore](#dataset-properties) 类型的输入[数据集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 类型的输出[数据集](data-factory-create-datasets.md)。
5. 包含使用 [AzureDataLakeStoreSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 的复制活动的[管道](data-factory-create-pipelines.md)

此示例每隔一小时将时序数据从 Azure Data Lake Store 复制到 Azure blob。 示例后续部分描述了这些示例中使用的 JSON 属性。

**Azure Data Lake Store 链接服务：**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>"
        }
    }
}
```

> [!NOTE]
> 有关配置详细信息，请参阅 [Azure Data Lake Store 链接服务属性](#linked-service-properties)部分中的步骤。
>

**Azure 存储链接服务：**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
**Azure Data Lake 输入数据集：**

设置 **"external": true** 将告知数据工厂服务：表位于数据工厂外且不由数据工厂中的活动生成。

```json
{
    "name": "AzureDataLakeStoreInput",
      "properties":
    {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            }
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
              "interval": 1
        },
        "policy": {
              "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
              }
        }
      }
}
```
**Azure Blob 输出数据集：**

数据将写入到新 blob，每隔一小时进行一次（频率：小时，间隔：1）。 根据正在处理的切片的开始时间，动态计算 blob 的文件夹路径。 文件夹路径使用开始时间的年、月、日和小时部分。

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**管道中使用 Azure Data Lake Store 源和 Blob 接收器的复制活动：**

管道包含配置为使用输入和输出数据集、且计划每小时运行一次的复制活动。 在管道 JSON 定义中，将 **source** 类型设置为 **AzureDataLakeStoreSource**，将 **sink** 类型设置为 **BlobSink**。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[  
              {
                "name": "AzureDakeLaketoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                  {
                    "name": "AzureDataLakeStoreInput"
                  }
                ],
                "outputs": [
                  {
                    "name": "AzureBlobOutput"
                  }
                ],
                "typeProperties": {
                    "source": {
                        "type": "AzureDataLakeStoreSource",
                      },
                      "sink": {
                        "type": "BlobSink"
                      }
                },
                   "scheduler": {
                      "frequency": "Hour",
                      "interval": 1
                },
                "policy": {
                      "concurrency": 1,
                      "executionPriorityOrder": "OldestFirst",
                      "retry": 0,
                      "timeout": "01:00:00"
                }
              }
         ]
    }
}
```

还可以在复制活动定义中将源数据集中的列映射到接收器数据集中的列。 有关详细信息，请参阅[映射 Azure 数据工厂中的数据集列](data-factory-map-columns.md)。

## <a name="performance-and-tuning"></a>性能和优化
请参阅[复制活动性能和优化指南](data-factory-copy-activity-performance.md)，了解影响 Azure 数据工厂中数据移动（复制活动）性能的关键因素以及各种优化方法。

