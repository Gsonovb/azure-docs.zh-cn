---
title: "在 Azure Active Directory 中为预先集成的应用自定义 SAML 令牌中颁发的声明 | Microsoft Docs"
description: "了解如何在 Azure Active Directory 中为预先集成的应用自定义 SAML 令牌中颁发的声明"
services: active-directory
documentationcenter: 
author: asmalser-msft
manager: femila
editor: 
ms.assetid: f1daad62-ac8a-44cd-ac76-e97455e47803
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/26/2016
ms.author: asmalser
translationtype: Human Translation
ms.sourcegitcommit: 785d3a8920d48e11e80048665e9866f16c514cf7
ms.openlocfilehash: d3952e65aaf3ca89f83e99409e196840dd692a01
ms.lasthandoff: 04/12/2017


---
# <a name="customizing-claims-issued-in-the-saml-token-for-pre-integrated-apps-in-azure-active-directory"></a>在 Azure Active Directory 中为预先集成的应用自定义 SAML 令牌中颁发的声明
Azure Active Directory 目前支持 Azure AD 应用库中数千个预先集成的应用程序，其中有 150 个以上的应用程序支持使用 SAML 2.0 协议的单一登录。 当用户使用 SAML 通过 Azure AD 对某个应用程序进行身份验证时，Azure AD 将该应用程序发送令牌（通过 HTTP 302 重定向），然后，该应用程序将执行验证并使用该令牌来登录用户，而不是提示用户输入用户名和密码。 这些 SAML 令牌包含有关用户的信息片段（称为“声明”）。

在标识交谈中，“声明”是标识提供者在为某个用户颁发的令牌中陈述的有关该用户的信息。 在 [SAML 令牌](http://en.wikipedia.org/wiki/SAML_2.0)中，此数据通常包含在 SAML 属性语句中，而用户的唯一 ID 通常显示在 SAML 主题中。

默认情况下，Azure AD 向应用程序颁发 SAML 令牌，其中包含 NameIdentifier 声明，以及用户在 Azure AD 中的用户名值（此值唯一标识用户）。 SAML 令牌还含有其他声明，其中包含用户的电子邮件地址、名字和姓氏。

若要查看或编辑 SAML 令牌中颁发给应用程序的声明，请在 Azure 管理门户中打开应用程序记录，并选择应用程序下面的“属性”选项卡。

![][1]

可能出于两个原因编辑 SAML 令牌中颁发的声明：•被写入的应用程序需要一组不同的声明 URI 或声明值 •应用程序的部署方式需要 NameIdentifier 声明不同于存储在 Azure Active Directory 中的用户名（也称为用户主体名称）。 

可以编辑任一默认声明值，方法是将鼠标光标移到 SAML 令牌属性表中的某个行上，然后单击显示在右侧的铅笔图标。 还可使用 **X** 图标删除声明（NameIdentifier 除外），并使用“添加用户属性”按钮添加新声明。

## <a name="editing-the-nameidentifier-claim"></a>编辑 NameIdentifier 声明
要解决使用不同用户名部署应用程序的问题，请单击 NameIdentifier 声明旁的铅笔图标。 这会提供包含多个不同选项的对话框：

![][2]

在“属性值”菜单中，选择“user.mail”以将 NameIdentifier 声明设置为目录中用户的电子邮件地址，或选择“user.onpremisessamaccountname”以设置为从本地 Azure AD 同步的用户 SAM 帐户名称。 

还可以使用特殊的 ExtractMailPrefix() 函数删除电子邮件地址或用户主体名称的域后缀，使得只有用户名的第一个部分可以通过（例如“joesmith”而不是 joesmith@contoso.com）。

![][3]

## <a name="adding-claims"></a>添加声明
添加新声明时，可以指定属性名称（不需要严格按照 SAML 规范遵循 URI 模式），以及将该值设置为存储在目录中的任何用户属性。

![][4]

例如，如果需要以声明方式发送用户在组织中所属的部门（如销售部），可以输入应用程序所需的任何声明值，然后选择 **user.department** 作为值。

如果指定的用户没有针对选定属性存储的值，则不会在令牌中颁发该声明。

**注意：** 仅在使用 [Azure AD Connect 工具](../active-directory-aadconnect.md)从本地 Active Directory 同步用户数据时，才支持 **user.onpremisesecurityidentifier** 和 **user.onpremisesamaccountname**。

## <a name="related-articles"></a>相关文章
* [有关 Azure Active Directory 中应用程序管理的文章索引](../active-directory-apps-index.md)
* [针对不在 Azure Active Directory 应用程序库中的应用程序配置单一登录](../active-directory-saas-custom-apps.md)
* [排查基于 SAML 的单一登录问题](active-directory-saml-debugging.md)

<!--Image references-->
[1]: ../media/active-directory-saml-claims-customization/claimscustomization1.png
[2]: ../media/active-directory-saml-claims-customization/claimscustomization2.png
[3]: ../media/active-directory-saml-claims-customization/claimscustomization3.png
[4]: ../media/active-directory-saml-claims-customization/claimscustomization4.png
