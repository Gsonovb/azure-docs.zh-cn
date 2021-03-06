---
title: "Azure 订阅与 Azure Active Directory 的关联方式 | Microsoft Docs"
description: "登录 Microsoft Azure 及相关问题（例如，Azure 订阅与 Azure Active Directory 之间的关系）。"
services: active-directory
documentationcenter: 
author: curtand
manager: femila
editor: 
ms.assetid: bc4773c2-bc4a-4d21-9264-2267065f0aea
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 02/13/2017
ms.author: curtand
translationtype: Human Translation
ms.sourcegitcommit: 2d428e0e3aaf8fd4a2138648411da644ccd308f6
ms.openlocfilehash: 4eee1b3f30c9e47983af5c1dfa60fe0deb8c2cc8


---
# <a name="how-azure-subscriptions-are-associated-with-azure-active-directory"></a>Azure 订阅与 Azure Active Directory 的关联方式
本文介绍有关登录 Microsoft Azure 及相关问题（例如，Azure 订阅与 Azure Active Directory (Azure AD) 之间的关系）的信息。

## <a name="accounts-that-you-can-use-to-sign-in"></a>可用于登录的帐户
首先，让我们了解可用于登录的帐户。 有两种类型的帐户：Microsoft 帐户（以前称为 Microsoft Live ID）和工作或学校帐户，后一种类型的帐户存储在 Azure AD 中。

| Microsoft 帐户 | Azure AD 帐户 |
| --- | --- |
| Microsoft 运行的使用者标识系统 |Microsoft 运行的企业标识系统 |
| 对面向使用者的服务（例如 Hotmail 和 MSN）进行身份验证 |对面向企业的服务（例如 Office 365）进行身份验证 |
| 使用者创建其自己的 Microsoft 帐户（例如，在注册电子邮件时） |公司和组织创建并管理其自己的工作或学校帐户 |
| 创建标识并将其存储在 Microsoft 帐户系统中 |使用 Azure 或其他服务（例如 Office 365）创建标识，并将其存储在分配给组织的 Azure AD 实例中 |

尽管 Azure 最初只允许 Microsoft 帐户用户的访问，但现在它允许来自 *两个* 系统的用户的访问。 这种变化是通过以下方式实现的：让所有 Azure 属性信任 Azure AD 执行身份验证，让 Azure AD 对组织用户进行身份验证，并创建一种联合关系，在这种关系中，Azure AD 将信任 Microsoft 帐户使用者标识系统对使用者用户执行身份验证。 因此，Azure AD 能够对“来宾”Microsoft 帐户以及“本机”Azure AD 帐户进行身份验证。

例如，下面这个具有 Microsoft 帐户的用户想要登录到 Azure 经典门户。

> [!NOTE]
> 若要登录到 Azure 经典门户，msmith@hotmail.com 必须拥有 Azure 订阅。 该帐户必须是服务管理员，或者订阅的协同管理员。
>
>

![][1]

由于此 Hotmail 地址是一个使用者帐户，因此，将由 Microsoft 帐户使用者标识系统对登录进行身份验证。 Azure AD 标识系统信任 Microsoft 帐户系统执行的身份验证，并会颁发一个令牌用于访问 Azure 服务。

## <a name="how-an-azure-subscription-is-related-to-azure-ad"></a>Azure 订阅与 Azure AD 的关系
每个 Azure 订阅都与某个 Azure AD 实例存在信任关系。 这意味着，此订阅信任该目录对用户、服务和设备执行身份验证。 多个订阅可以信任同一个目录，但一个订阅只能信任一个目录。 可以在“设置”选项卡下查看你的订阅信任的目录。 可以 [编辑订阅设置](active-directory-understanding-resource-access.md) 以更改它信任的目录。

订阅与目录之间的这种信任关系不同于订阅与 Azure 中所有其他资源（网站、数据库等）之间的信任关系，在后一种关系中，这些资源更像是订阅的子资源。 如果某个订阅过期，则对该订阅关联的其他那些资源的访问权限也将终止。 但是，目录将保留在 Azure 中，并且你可以将另一个订阅与该目录相关联，然后继续管理目录用户。

同样，订阅中显示的 Azure AD 扩展将不会像 Azure 经典门户中的其他扩展那样工作。 Azure 经典门户中的其他扩展将划归到 Azure 订阅范围内。 Azure AD 扩展中显示的内容不会因订阅而异 - 它只会根据登录用户显示目录。

所有用户都只有一个用于验证其身份的主目录，但他们也可以是其他目录中的来宾。 在 Azure AD 扩展中，你将会看到你的用户帐户所属的每个目录。 你的帐户不是其成员的任何目录都不会显示。 目录可以为 Azure AD 中的工作或学校帐户或者为 Microsoft 帐户用户颁发令牌（因为 Azure AD 已与 Microsoft 帐户系统联合）。

下图显示了 Michael Smith 在使用 Contoso 的工作帐户注册后获得的订阅。

![][2]

## <a name="how-to-manage-a-subscription-and-a-directory"></a>如何管理订阅和目录
Azure 订阅的管理角色可以管理与 Azure 订阅绑定的资源。 [在 Azure Active Directory 中分配管理员角色](active-directory-assign-admin-roles.md)中介绍了这些角色以及订阅管理最佳实践。

默认情况下，当你注册时，系统将为你分配服务管理员角色。 如果其他人需要使用同一个订阅登录和访问服务，则你可以将他们添加为协同管理员。 服务管理员和协同管理员可以是 Microsoft 帐户，也可以是 Azure 订阅信任的 Azure AD 组织内部的工作或学校帐户。

Azure AD 提供一组不同的管理角色，用于管理目录和标识相关的功能。 例如，目录的全局管理员可以将用户和组添加到目录，或者要求对用户执行多重身份验证。 将为创建目录的用户分配全局管理员角色，而他们又可以向其他用户分配管理员角色。

与订阅管理员一样，Azure AD 管理角色可以是 Microsoft 帐户，也可以是工作或学校帐户。 Office 365 和 Microsoft Intune 等其他服务也会使用 Azure AD 管理角色。 有关详细信息，请参阅 [分配管理员角色](active-directory-assign-admin-roles.md)。

但必须注意，Azure 订阅管理员和 Azure AD 目录管理员是两个不同的概念。 Azure 订阅管理员可以管理 Azure 中的资源，还可以在 Azure 经典门户中查看 Active Directory 扩展（因为 Azure 经典门户是一种 Azure 资源）。 Directory 管理员可以管理目录中的属性。

一个人可以充当两种角色，但是没有这个必要。 可将某个用户分配到目录全局管理员角色，但不能将其分配为 Azure 订阅的服务管理员或协同管理员。 如果不是订阅的管理员，此用户将无法登录到 Azure 经典门户。 但是，该用户可以使用其他工具（例如 Azure AD PowerShell 或 Office 365 管理中心）执行目录管理任务。

## <a name="why-cant-i-manage-the-directory-with-my-current-user-account"></a>我为何无法使用我当前的用户帐户管理目录？
有时，用户在注册 Azure 订阅之前，可能会尝试使用工作或学校帐户登录到 Azure 经典门户。 在这种情况下，该用户将会收到一条消息，指明该帐户没有订阅。 该消息包含一个用于开始免费试用订阅的链接。

在注册免费试用订阅之后，该用户将在 Azure 经典门户中看到组织的目录，但无法管理该目录（也就是说，无法添加用户或编辑任何现有用户属性），因为该用户不是目录全局管理员。 订阅后，该用户可以使用 Azure 经典门户并查看 Azure Active Directory 扩展，但若要管理目录，则需要拥有全局管理员的其他权限。

## <a name="using-your-work-or-school-account-to-manage-an-azure-subscription-that-was-created-by-using-a-microsoft-account"></a>使用工作或学校帐户管理使用 Microsoft 帐户创建的 Azure 订阅
最佳实践是，你应该 [以组织身份注册 Azure](sign-up-organization.md) 并使用工作或学校帐户来管理 Azure 中的资源。 之所以首选工作或学校帐户，是因为它们由颁发它们的组织集中管理，比 Microsoft 帐户的功能更多，并且直接由 Azure AD 进行身份验证。 使用同一个帐户可以访问提供给企业和组织的其他 Microsoft Online Services，例如 Office 365 或 Microsoft Intune。 如果你已经拥有了一个与其他属性配合使用的帐户，则可能想要对 Azure 使用这同一个帐户。 在这种情况下，你还会获得一个 Active Directory 实例，该实例可以支持你希望 Azure 订阅信任的那些属性。

此外，与 Microsoft 帐户相比，工作或学校帐户可通过更多方式进行管理。 例如，管理员可以重置工作或学校帐户的密码，或者要求对它进行多重身份验证。

在某些情况下，你可能希望你组织中的用户能够管理与 Microsoft 使用者帐户的 Azure 订阅关联的资源。 有关如何改为使用不同帐户管理订阅或目录的详细信息，请参阅 [在 Azure 中管理 Office 365 订阅的目录](#manage-the-directory-for-your-office-365-subscription-in-azure)。

## <a name="signing-in-when-you-used-your-work-email-for-your-microsoft-account"></a>在使用 Microsoft 帐户的工作电子邮件时登录
如果在过去的某个时间点，你使用工作电子邮件作为用户标识符创建了一个 Microsoft 使用者帐户，则可能会看到一个页面，要求你在 Microsoft Azure 帐户系统和 Microsoft 帐户系统中选择其一。

![][3]

你有两个同名的用户帐户，一个在 Azure AD 中，而另一个在 Microsoft 使用者帐户系统中。 应该选取与要使用的 Azure 订阅关联的帐户。 如果出现错误，指明此用户不存在订阅，则可能是你选择了错误的选项。 请注销后重试。 有关可能会阻止登录的错误的详细信息，请参阅 [Troubleshooting "We were unable to find any subscriptions associated with your account" errors](https://social.msdn.microsoft.com/Forums/en-US/f952f398-f700-41a1-8729-be49599dd7e2/troubleshooting-we-were-unable-to-find-any-subscriptions-associated-with-your-account-errors-in?forum=windowsazuremanagement)（排查“找不到任何与你的帐户关联的订阅”错误）。

## <a name="manage-the-directory-for-your-office-365-subscription-in-azure"></a>在 Azure 中管理 Office 365 订阅的目录
假设你在注册 Azure 之前注册了 Office 365。 现在，你想要在 Azure 经典门户中管理 Office 365 订阅的目录。 可以通过两种方式进行管理，具体取决于你是否注册了 Azure。

### <a name="i-do-not-have-a-subscription-for-azure"></a>我没有订阅 Azure
在这种情况下，你只需使用登录 Office 365 时所用的同一个工作帐户或学校帐户来 [注册 Azure](sign-up-organization.md) 。 Office 365 帐户中的相关信息将预填充到 Azure 注册表单中。 你的帐户将被指定为订阅的服务管理员角色。  

### <a name="i-do-have-a-subscription-for-azure-using-my-microsoft-account"></a>我已使用我的 Microsoft 帐户订阅了 Azure
如果你已使用工作或学校帐户注册了 Office 365 并随后使用 Microsoft 帐户注册了 Azure，那么，你将获得两个目录：在工作时或学校中使用的目录，以及在你注册 Azure 时创建的默认目录。

若要在 Azure 经典门户中管理这两个目录，请完成以下步骤。

> [!NOTE]
> 只有当用户使用 Microsoft 帐户登录时才能完成这些步骤。 如果用户是使用工作或学校帐户登录的，则“使用现有目录”选项不可用，因为工作或学校帐户只能通过其主目录（也就是存储工作或学校帐户的目录，该目录归工作单位或学校拥有）进行身份验证。
>
>

1. 使用你的 Microsoft 帐户登录到 Azure 经典门户。
2. 单击“新建” > “应用程序服务” > “Active Directory” > “目录” > “自定义创建”。
3. 单击“使用现有目录”并选中“已准备好立即注销”，然后单击复选标记以完成操作。
4. 使用对工作或学校目录具有全局管理员权限的帐户登录到 Azure 经典门户。
5. 当系统提示“是否要将 Contoso 目录用于 Azure?”时，单击“继续”。
6. 单击“立即注销”。
7. 使用你的 Microsoft 帐户重新登录到 Azure 经典门户。 这两个目录将显示在“Active Directory”扩展中。

## <a name="next-steps"></a>后续步骤
* 若要了解有关如何更改 Azure 订阅管理员的详细信息，请参阅 [如何添加或更改 Azure 管理员角色](../billing/billing-add-change-azure-subscription-administrator.md)
* 若要了解有关如何在 Microsoft Azure 中控制资源访问的详细信息，请参阅 [了解 Azure 中的资源访问权限](active-directory-understanding-resource-access.md)
* 有关如何在 Azure AD 中分配角色的详细信息，请参阅 [在 Azure Active Directory 中分配管理员角色](active-directory-assign-admin-roles.md)
* [以组织身份注册 Azure](sign-up-organization.md)

<!--Image references-->
[1]: ./media/active-directory-how-subscriptions-associated-directory/WAAD_PassThruAuth.png
[2]: ./media/active-directory-how-subscriptions-associated-directory/WAAD_OrgAccountSubscription.png
[3]: ./media/active-directory-how-subscriptions-associated-directory/WAAD_SignInDisambiguation.PNG



<!--HONumber=Feb17_HO2-->


