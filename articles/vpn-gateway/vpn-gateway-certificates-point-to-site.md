---
title: "为点到站点连接创建自签名证书：PowerShell：Azure | Microsoft 文档"
description: "本文包含使用 PowerShell 在 Windows 10 上创建自签名根证书、导出公钥和生成客户端证书的步骤。"
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 27b99f7c-50dc-4f88-8a6e-d60080819a43
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/24/2017
ms.author: cherylmc
translationtype: Human Translation
ms.sourcegitcommit: b0c27ca561567ff002bbb864846b7a3ea95d7fa3
ms.openlocfilehash: 72fc6eb93c77dd5a0a7ce55897f4c06fb98c0721
ms.lasthandoff: 04/25/2017


---
# <a name="create-a-self-signed-root-certificate-for-point-to-site-connections-using-powershell"></a>使用 PowerShell 为点到站点连接创建自签名根证书

点到站点连接使用证书进行身份验证。 配置点到站点连接时，需要将根证书的公钥（.cer 文件）上载到 Azure。 必须从根证书生成客户端证书，并将其安装在连接到 VNet 的每台客户端计算机上。 客户端证书允许客户端进行身份验证。 本文将说明创建自签名根证书、导出公钥，以及生成客户端证书。 本文不包含点到站点配置说明或点到站点常见问题解答。 可以通过从以下列表中选择一篇文章来查找该信息：

> [!div class="op_single_selector"]
> * [创建自签名证书](vpn-gateway-certificates-point-to-site.md)
> * [配置点到站点 - Resource Manager - Azure 门户](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
> * [配置点到站点 - Resource Manager - PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md)
> * [配置点到站点 - 经典 - Azure 门户](vpn-gateway-howto-point-to-site-classic-azure-portal.md)
> 
> 

## <a name="rootcert"></a>创建自签名根证书

以下步骤将引导你完成在 Windows 10 上使用 PowerShell 创建自签名根证书的过程。 这些步骤中使用的 cmdlet 和参数是 Windows 10 操作系统的一部分，而不是 PowerShell 版本的一部分。 这并不意味着，所创建的证书只能在 Windows 10 上安装。 有关支持的客户端的信息，请参阅[点到站点常见问题解答](vpn-gateway-howto-point-to-site-resource-manager-portal.md#faq)。

1. 在运行 Windows 10 的计算机上，使用提升的特权打开 Windows PowerShell 控制台。
2. 使用以下示例创建自签名根证书。 以下示例创建名为“P2SRootCert”、将自动安装在“Certificates-Current User\Personal\Certificates”中的自签名根证书。 打开 *certmgr.msc* 即可查看该证书。

  ```powershell
  $cert = New-SelfSignedCertificate -Type Custom -KeySpec Signature `
  -Subject "CN=P2SRootCert" -KeyExportPolicy Exportable `
  -HashAlgorithm sha256 -KeyLength 2048 `
  -CertStoreLocation "Cert:\CurrentUser\My" -KeyUsageProperty Sign -KeyUsage CertSign
  ```

### <a name="cer"></a>获取公钥

点到站点连接要求将公钥 (.cer) 上载到 Azure。 以下步骤可帮助你导出自签名根证书的 .cer 文件：

1. 若要获取证书 .cer 文件，请打开“管理用户证书”。 找到自签名根证书（通常位于“Certificates - Current User\Personal\Certificates”中），然后右键单击。 单击“所有任务”，然后单击“导出”。 此操作将打开“证书导出向导”。
2. 在向导中，单击“下一步”。 选择“否，不导出私钥”，然后单击“下一步”。
3. 在“导出文件格式”页上，选择“Base-64 编码的 X.509 (.CER)”，然后单击“下一步”。 
4. 在“要导出的文件”中，“浏览”到要将证书导出的目标位置。 在“文件名”中，为证书文件命名。 然后单击“下一步”。
5. 单击“完成”导出证书。 会看到“导出成功”。 单击“确定”关闭向导。

### <a name="to-export-a-self-signed-root-certificate-optional"></a>导出自签名根证书（可选）

你可能想要导出自签名根证书并将它存储在安全位置。 如果需要，可以稍后在另一台计算机上安装此自签名证书，然后生成更多客户端证书，或导出另一个 .cer 文件。 若要将自签名根证书导出为 .pfx，请选择该根证书，然后使用[导出客户端证书](#clientexport)中所述的步骤导出。

## <a name="clientcert"></a>生成客户端证书

在使用点到站点连接连接到 VNet 的每台客户端计算机上，必须安装客户端证书。 可以从自签名根证书生成客户端证书，然后导出并安装该客户端证书。 如果不安装客户端证书，身份验证将会失败。 

以下步骤引导你完成从自签名根证书生成客户端证书的过程。 可以从相同根证书生成多个客户端证书。 使用以下步骤生成客户端证书时，客户端证书将自动安装在用于生成该证书的计算机上。 如果想要在另一台客户端计算机上安装客户端证书，可以导出该证书。

使用以下 PowerShell 步骤生成客户端证书时，需要 Windows 10。 这些步骤中使用的 cmdlet 和参数是 Windows 10 操作系统的一部分，而不是 PowerShell 版本的一部分。 这并不意味着，所创建的证书只能在 Windows 10 上安装。 有关支持的客户端的信息，请参阅[点到站点常见问题解答](vpn-gateway-howto-point-to-site-resource-manager-portal.md#faq)。

### <a name="example-1"></a>示例 1

此示例使用上一部分中声明的“$cert”变量。 如果创建自签名根证书后关闭了 PowerShell 控制台，或者要在新的 PowerShell 控制台会话中创建其他客户端证书，请使用[示例 2](#ex2) 中的步骤。

修改并运行示例以生成客户端证书。 如果在未经修改的情况下直接运行以下示例，将会生成名为“P2SChildCert”的客户端证书。  如果想要为子证书指定其他名称，请修改 CN 值。 运行此示例时，请不要更改 TextExtension。 生成的客户端证书将自动安装在计算机上的“Certificates - Current User\Personal\Certificates”中。

```powershell
New-SelfSignedCertificate -Type Custom -KeySpec Signature `
-Subject "CN=P2SChildCert" -KeyExportPolicy Exportable `
-HashAlgorithm sha256 -KeyLength 2048 `
-CertStoreLocation "Cert:\CurrentUser\My" `
-Signer $cert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")
```

### <a name="ex2"></a>示例 2

若要创建其他客户端证书，或者不想要使用创建自签名根证书时所用的同一个 PowerShell 会话，请使用以下步骤：

1. 识别安装在计算机上的自签名根证书。 此 cmdlet 将返回计算机上安装的证书列表。

  ```powershell
  Get-ChildItem -Path “Cert:\CurrentUser\My”
  ```
2. 从返回的列表中找到使用者名称，然后将该名称旁边的指纹复制到文本文件中。 以下示例显示了两个证书。 CN 名称是要从中生成子证书的自签名根证书的名称。 在本例中，该根证书为“P2SRootCert”。

        Thumbprint                                Subject

        AED812AD883826FF76B4D1D5A77B3C08EFA79F3F  CN=P2SChildCert4
        7181AA8C1B4D34EEDB2F3D3BEC5839F3FE52D655  CN=P2SRootCert


3. 使用上一步骤中获取的指纹为根证书声明变量。 将 THUMBPRINT 替换为要从中生成子证书的根证书的指纹。

  ```powershell
  $cert = Get-ChildItem -Path "Cert:\CurrentUser\My\THUMBPRINT"
  ```

  例如，使用上一步骤中 P2SRootCert 的指纹，该变量将如下所示：

  ```powershell
  $cert = Get-ChildItem -Path "Cert:\CurrentUser\My\7181AA8C1B4D34EEDB2F3D3BEC5839F3FE52D655"
  ```    

4.  修改并运行示例以生成客户端证书。 如果在未经修改的情况下直接运行以下示例，将会生成名为“P2SChildCert”的客户端证书。 如果想要为子证书指定其他名称，请修改 CN 值。 运行此示例时，请不要更改 TextExtension。 生成的客户端证书将自动安装在计算机上的“Certificates - Current User\Personal\Certificates”中。

  ```powershell
  New-SelfSignedCertificate -Type Custom -KeySpec Signature `
  -Subject "CN=P2SChildCert" -KeyExportPolicy Exportable `
  -HashAlgorithm sha256 -KeyLength 2048 `
  -CertStoreLocation "Cert:\CurrentUser\My" `
  -Signer $cert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")
  ```

## <a name="clientexport"></a>导出客户端证书   

生成客户端证书时，该证书会自动安装在用于生成它的计算机上。 如果想要在另一台客户端计算机上安装客户端证书，需要导出生成的客户端证书。                              

1. 若要导出客户端证书，请打开“管理用户证书”。 生成的客户端证书默认位于“Certificates - Current User\Personal\Certificates”中。 右键单击要导出的客户端证书，单击“所有任务”，然后单击“导出”打开“证书导出向导”。
2. 在向导中，单击“**下一步**”，选择“**是，导出私钥**”，然后单击“**下一步**”。
3. 在“导出文件格式”页上，保留选择默认值。 请务必选中“包括证书路径中的所有证书(如果可能)”。 选择此项也会导出成功身份验证所需的根证书信息。 然后单击“下一步”。
4. 在“**安全性**”页上，必须保护私钥。 如果选择使用密码，请务必记下或牢记为此证书设置的密码。 然后单击“下一步”。
5. 在“要导出的文件”中，“浏览”到要将证书导出的目标位置。 在“文件名”中，为证书文件命名。 然后单击“下一步”。
6. 单击“完成”导出证书。    

## <a name="install"></a>安装已导出的客户端证书

如果想要从另一台客户端计算机（而不是用于生成客户端证书的计算机）创建 P2S 连接，需要安装客户端证书。 安装客户端证书时，需要使用导出客户端证书时创建的密码。

1. 找到 *.pfx* 文件并将其复制到客户端计算机。 在客户端计算机上，双击 *.pfx* 文件以进行安装。 将“存储位置”保留为“当前用户”，然后单击“下一步”。
2. 在“要导入的**文件**”页上，不要进行任何更改。 单击“下一步”。
3. 在“私钥保护”页上，输入证书的密码，或验证安全主体是否正确，然后单击“下一步”。
4. 在“**证书存储**”页上，保留默认位置，然后单击“**下一步**”。
5. 单击“**完成**”。 在证书安装的“**安全警告**”上，单击“**是**”。 可随时单击“是”，因为证书已生成。 现已成功导入证书。

## <a name="next-steps"></a>后续步骤

继续使用点到站点配置。 

* 有关 **Resource Manager** 部署模型步骤，请参阅 [Configure a Point-to-Site connection to a VNet](vpn-gateway-howto-point-to-site-resource-manager-portal.md)（配置与 VNet 的点到站点连接）。 
* 有关**经典**部署模型步骤，请参阅 [Configure a Point-to-Site VPN connection to a VNet (classic)](vpn-gateway-howto-point-to-site-classic-azure-portal.md)（配置与 VNet 的点到站点 VPN 连接（经典））。
