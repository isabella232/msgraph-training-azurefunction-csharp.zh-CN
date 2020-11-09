---
ms.openlocfilehash: 003d7f382a0c950d31bb84176feb1e83fbd5cf19
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822573"
---
<!-- markdownlint-disable MD002 MD041 -->

在本教程中，将创建一个简单的 Azure 函数，该函数实现调用 Microsoft Graph 的 HTTP 触发器函数。 这些函数将涵盖以下方案：

- 使用 [代表流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) 身份验证来实现 API 以访问用户的收件箱。
- 实现 API 以订阅和取消订阅用户收件箱上的通知，使用 [客户端凭据授予流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) 身份验证。
- 实现 webhook 以接收来自 Microsoft Graph 的 [更改通知](https://docs.microsoft.com/graph/webhooks) 并使用客户端凭据授予流访问数据。

您还将创建一个简单的 JavaScript 单页面应用程序 (SPA) ，以调用在 Azure 函数中实现的 Api。

## <a name="create-azure-functions-project"></a>创建 Azure 函数项目

1. 在要创建项目的目录中打开命令行界面 (CLI) 。 运行以下命令。

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. 将 CLI 中的当前目录更改为 **GraphTutorial** 目录，并运行以下命令以在项目中创建三个函数。

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. 打开 **local.settings.js** ，并将以下各项添加到文件中，以允许从 `http://localhost:8080` 测试应用程序的 URL 中进行 CORS。

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. 运行以下命令以在本地运行该项目。

    ```Shell
    func start
    ```

1. 如果一切正常，您将看到以下输出：

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. 打开浏览器并浏览到输出中显示的函数 Url，以验证这些函数是否正常工作。 您应该会在浏览器中看到以下消息： `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` 。

## <a name="create-single-page-application"></a>创建单页面应用程序

1. 在要创建项目的目录中打开您的 CLI。 创建一个名为 **TestClient** 的目录以保存您的 HTML 和 JavaScript 文件。

1. 在 **TestClient** 目录中创建一个名为 **index.html** 的新文件，并添加以下代码。

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    这将定义应用程序的基本布局，包括导航栏。 此外，它还会添加以下内容：

    - [引导数据库](https://getbootstrap.com/) 及其支持的 JavaScript
    - [FontAwesome](https://fontawesome.com/)
    - [适用于 JavaScript 的 Microsoft 身份验证库 ( # A0) 2。0](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > 页面包含一个 favicon， (`<link rel="shortcut icon" href="g-raph.png">`) 。 您可以删除此行，也可以从 [GitHub](https://github.com/microsoftgraph/g-raph)下载 **g-raph.png** 文件。

1. 在 **TestClient** 目录中创建一个名为 **style** 的新文件，并添加以下代码。

    :::code language="css" source="../demo/TestClient/style.css":::

1. 在 **TestClient** 目录中创建一个名为 **ui.js** 的新文件，并添加以下代码。

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    此代码使用 JavaScript 根据所选视图呈现当前页面。

### <a name="test-the-single-page-application"></a>测试单页面应用程序

> [!NOTE]
> 本节包含有关如何使用 [dotnet](https://github.com/natemcmaster/dotnet-serve) 来在开发计算机上运行简单测试 HTTP 服务器的说明。 不需要使用此特定工具。 您可以使用任何您希望为 **TestClient** 目录提供服务的测试服务器。

1. 在 CLI 中运行以下命令，以安装 **dotnet 服务** 。

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. 将 CLI 中的当前目录更改为 **TestClient** 目录，并运行以下命令以启动 HTTP 服务器。

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. 打开浏览器，并导航到 `http://localhost:8080`。 页面应呈现，但当前没有任何按钮可用。

## <a name="add-nuget-packages"></a>添加 NuGet 包

在继续之前，请安装稍后将使用的一些其他 NuGet 包。

- 在 Azure 函数项目中启用依赖关系注入的[扩展名](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions)。
- [Microsoft.Extensions.Configuration。UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) 从 [.net 开发密钥存储](https://docs.microsoft.com/aspnet/core/security/app-secrets)读取应用程序配置。
- 用于调用 Microsoft Graph 的[microsoft](https://www.nuget.org/packages/Microsoft.Graph/) graph。
- 用于对令牌进行身份验证和管理的 " [Microsoft 身份" 客户端](https://www.nuget.org/packages/Microsoft.Identity.Client/)。
- [Microsoft.identitymodel.dll](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) 用于检索令牌验证的 OpenID 配置的 OpenIdConnect。
- 用于验证已发送到 web API 的令牌的[Microsoft.identitymodel.dll Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) 。

1. 将 CLI 中的当前目录更改为 **GraphTutorial** 目录，并运行以下命令。

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
