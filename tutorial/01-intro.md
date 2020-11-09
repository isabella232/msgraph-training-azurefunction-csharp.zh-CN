---
ms.openlocfilehash: 1e4267a5d8bdfe02dbc0170122113c35df822659
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822570"
---
<!-- markdownlint-disable MD002 MD041 -->

本教程向您介绍如何构建使用 Microsoft Graph API 检索用户的日历信息的 Azure 函数。

> [!TIP]
> 如果您只想下载已完成的教程，可以下载或克隆 [GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)。 有关使用应用 ID 和密码配置应用程序的说明，请参阅 **demo** 文件夹中的自述文件。

## <a name="prerequisites"></a>必备条件

在开始本教程之前，您应在开发计算机上安装以下工具。

- [.NET Core SDK](https://dotnet.microsoft.com/download)
- [Azure 函数核心工具](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [ngrok](https://ngrok.com/)

此外，还应具有 Microsoft 工作或学校帐户，并访问同一组织中的全局管理员帐户。 如果你没有 Microsoft 帐户，可以 [注册 office 365 开发人员计划](https://developer.microsoft.com/office/dev-program) 以获取免费的 office 365 订阅。

> [!NOTE]
> 本教程是使用上述工具的以下版本编写的。 本指南中的步骤可能适用于其他版本，但尚未经过测试。
>
> - .NET Core SDK 3.1.301
> - Azure 函数 Core Tools 3.0.2630
> - Azure CLI 2.8。0
> - ngrok 2.3.35

## <a name="feedback"></a>反馈

请在 [GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)中提供有关本教程的任何反馈。
