---
ms.openlocfilehash: fbd43b4708cd1aeaad93b70fcf151869bd9e543e
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822577"
---
<!-- markdownlint-disable MD002 MD041 -->

在本练习中，你将了解示例 Azure 函数为准备 [发布到 Azure 函数应用程序](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)所需的更改。

## <a name="update-code"></a>更新代码

将从用户密钥存储中读取配置，这仅适用于开发计算机。 在发布到 Azure 之前，你需要更改存储配置的位置，并相应地更新 **Startup.cs** 中的代码。

应用程序密码应存储在安全存储中，如 [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview)。

## <a name="update-cors-setting-for-azure-function"></a>更新 Azure 函数的 CORS 设置

在此示例中，我们将 "CORS" 配置 **local.settings.js"启用** "，以允许测试应用程序调用函数。 您需要将已发布的函数配置为允许任何将调用它的 SPA 应用程序。

## <a name="update-app-registrations"></a>更新应用注册

将  `knownClientApplications` 需要使用将调用 Azure 函数的任何应用程序的应用程序 id 更新用于 **图 Azure 函数** 应用注册的清单中的属性。

## <a name="recreate-existing-subscriptions"></a>重新创建现有订阅

使用您的本地计算机或 ngrok 上的 webhook URL 创建的任何订阅都应使用 Azure 函数的生产 URL 重新创建 `Notify` 。
