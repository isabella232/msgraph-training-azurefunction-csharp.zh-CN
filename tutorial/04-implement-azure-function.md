---
ms.openlocfilehash: 728138b5d298f9469a566defed6ee2e457896c0b
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822585"
---
<!-- markdownlint-disable MD002 MD041 -->

在本练习中，你将完成 Azure 函数的实现 `GetMyNewestMessage` 并更新测试客户端以调用函数。

Azure 函数使用 [代表流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)。 此流中事件的基本顺序为：

- 测试应用程序使用交互式身份验证流，以允许用户登录并授予同意。 它将返回范围限定为 Azure 函数的令牌。 令牌 **不包含任何** Microsoft Graph 作用域。
- 测试应用程序将调用 Azure 函数，并在标头中发送其访问令牌 `Authorization` 。
- Azure 函数验证令牌，然后将该令牌与包含 Microsoft Graph 作用域的第二个访问令牌交换。
- Azure 函数代表用户使用第二个访问令牌调用 Microsoft Graph。

> [!IMPORTANT]
> 为了避免在源中存储应用程序 ID 和密码，您将使用 [.Net 密钥管理器](https://docs.microsoft.com/aspnet/core/security/app-secrets) 来存储这些值。 机密管理器仅用于开发目的，而生产应用程序应使用受信任密钥管理器来存储机密信息。

## <a name="add-authentication-to-the-single-page-application"></a>向单页面应用程序添加身份验证

首先向 SPA 添加身份验证。 这将允许应用程序获取访问令牌，授予访问权限以调用 Azure 函数。 因为这是一个 SPA，所以它将对 [PKCE 使用授权代码流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)。

1. 在名为 **config.js** 的 **TestClient** 目录中创建一个新文件，并添加以下代码。

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    `YOUR_TEST_APP_APP_ID_HERE`将替换为您在 azure 门户中为 **Graph Azure 函数测试应用** 程序创建的应用程序 ID。 `YOUR_TENANT_ID_HERE`将替换为您从 Azure 门户复制的 **目录 (租户) ID** 值。 将替换为 `YOUR_AZURE_FUNCTION_APP_ID_HERE` **Graph Azure 函数** 的应用程序 ID。

    > [!IMPORTANT]
    > 如果您使用的是源代码管理（如 git），现在可以从源代码管理中排除 **config.js** 文件，以避免在无意中泄漏您的应用程序 id 和租户 ID。

1. 在名为 **auth.js** 的 **TestClient** 目录中创建一个新文件，并添加以下代码。

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    请考虑此代码执行的操作。

    - 它 `PublicClientApplication` 使用存储在 **config.js** 中的值对 a 进行初始化。
    - 使用 `loginPopup` Azure 函数的权限范围在中对用户进行签名。
    - 它将用户的用户名存储在会话中。

    > [!IMPORTANT]
    > 由于应用程序使用 `loginPopup` ，您可能需要将浏览器的弹出窗口阻止程序更改为允许弹出窗口阻止程序 `http://localhost:8080` 。

1. 刷新页面并登录。 页面应使用用户名进行更新，这表明登录成功。

> [!TIP]
> 您可以在中分析访问令牌， [https://jwt.ms](https://jwt.ms) 并确认 `aud` 声明是 azure 函数的应用程序 ID，并且 `scp` 声明包含 azure 函数的权限范围，而不是 Microsoft Graph。

## <a name="add-authentication-to-the-azure-function"></a>向 Azure 函数添加身份验证

在本节中，你将在 Azure 函数中实施代表流 `GetMyNewestMessage` ，以获取与 Microsoft Graph 兼容的访问令牌。

1. 通过在包含 **GraphTutorial** 的目录中打开 CLI 并运行以下命令来初始化 .net 开发密钥存储。

    ```Shell
    dotnet user-secrets init
    ```

1. 使用以下命令将应用程序 ID、密码和租户 ID 添加到密钥存储。 将替换为 `YOUR_API_FUNCTION_APP_ID_HERE` **Graph Azure 函数** 的应用程序 ID。 `YOUR_API_FUNCTION_APP_SECRET_HERE`将替换为在 azure 门户中为 **Graph azure 函数** 创建的应用程序密码。 `YOUR_TENANT_ID_HERE`将替换为您从 Azure 门户复制的 **目录 (租户) ID** 值。

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a>处理传入持有者令牌

在本节中，您将实现一个类，用于验证和处理从 SPA 发送到 Azure 函数的载荷令牌。

1. 在名为 **Authentication** 的 **GraphTutorial** 目录中创建一个新目录。

1. 在 **./GraphTutorial/Authentication** 文件夹中创建一个名为 **TokenValidationResult.cs** 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. 在 **./GraphTutorial/Authentication** 文件夹中创建一个名为 **TokenValidation.cs** 的新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

请考虑此代码执行的操作。

- 它确保标头中存在持有者令牌 `Authorization` 。
- 它将验证来自 Azure 的已发布 OpenID 配置的签名和颁发者。
- 它验证访问群体 (`aud` 声明是否与 Azure 函数的应用程序 ID) 匹配。
- 它分析令牌并生成 MSAL 帐户 ID，这将需要利用令牌缓存。

### <a name="create-an-on-behalf-of-authentication-provider"></a>创建代表身份验证提供程序

1. 在名为 " **OnBehalfOfAuthProvider.cs** " 的 **身份验证** 目录中创建一个新文件，并将以下代码添加到该文件中。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

请花点时间考虑 **OnBehalfOfAuthProvider.cs** 中的代码的功能。

- 在 `GetAccessToken` 函数中，它首先尝试使用从令牌缓存中获取用户令牌 `AcquireTokenSilent` 。 如果此操作失败，它会将测试应用程序发送的持有者令牌用于 Azure 函数，以生成用户断言。 然后，它使用该用户断言获取与图形兼容的令牌（使用） `AcquireTokenOnBehalfOf` 。
- 它实现 `Microsoft.Graph.IAuthenticationProvider` 接口，允许在中的构造函数中传递此类以对 `GraphServiceClient` 传出请求进行身份验证。

### <a name="implement-a-graph-client-service"></a>实现 Graph 客户端服务

在本节中，你将实现可注册 [依赖项注入](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)的服务。 该服务将用于获取经过身份验证的 Graph 客户端。

1. 在名为 " **服务** " 的 **GraphTutorial** 目录中创建一个新目录。

1. 在名为 " **IGraphClientService.cs** " 的 **服务** 目录中创建一个新文件，并将以下代码添加到该文件中。

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. 在名为 " **GraphClientService.cs** " 的 **服务** 目录中创建一个新文件，并将以下代码添加到该文件中。

    ```csharp
    using GraphTutorial.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Client;
    using Microsoft.Graph;

    namespace GraphTutorial.Services
    {
        // Service added via dependency injection
        // Used to get an authenticated Graph client
        public class GraphClientService : IGraphClientService
        {
        }
    }
    ```

1. 将以下属性添加到 `GraphClientService` 类中。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. 将以下函数添加到 `GraphClientService` 类中。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. 为函数添加占位符实现 `GetAppGraphClient` 。 您将在后面的部分中实现。

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    `GetUserGraphClient`函数采用令牌验证的结果，并为用户生成已通过身份验证的结果 `GraphServiceClient` 。

1. 在名为 **Startup.cs** 的 **GraphTutorial** 目录中创建一个新文件，并将以下代码添加到该文件中。

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="StartupSnippet":::

    此代码将在 Azure 函数中启用 [依赖关系注入](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) ， `IConfiguration` 从而公开对象和 `GraphClientService` 服务。

### <a name="implement-getmynewestmessage-function"></a>实现 GetMyNewestMessage 函数

1. 打开 **/GraphTutorial/GetMyNewestMessage.cs** ，并将其全部内容替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a>查看 GetMyNewestMessage.cs 中的代码

请花点时间考虑 **GetMyNewestMessage.cs** 中的代码的功能。

- 在构造函数中，它 `IConfiguration` 通过依赖关系注入保存传入的对象。
- 在 `Run` 函数中，它执行以下操作：
  - 验证对象中是否存在所需的配置值 `IConfiguration` 。
  - 验证持有者令牌，并在 `401` 令牌无效时返回状态代码。
  - 从发出此请求的用户处获取图形客户端 `GraphClientService` 。
  - 使用 Microsoft Graph SDK 从用户的收件箱获取最新邮件，并在响应中将其作为 JSON 正文返回。

## <a name="call-the-azure-function-from-the-test-app"></a>从测试应用程序调用 Azure 函数

1. 打开 **auth.js** 并添加以下函数以获取访问令牌。

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    请考虑此代码执行的操作。

    - 它首先尝试在没有用户交互的情况下以无提示方式获取访问令牌。 由于用户应已登录，因此 MSAL 应具有用户在其缓存中的令牌。
    - 如果失败，并显示指示用户需要交互的错误，它将尝试以交互方式获取令牌。

1. 在名为 **azurefunctions.js** 的 **TestClient** 目录中创建一个新文件，并添加以下代码。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. 将您的 CLI 中的当前目录更改为 **/GraphTutorial** 目录，并运行以下命令以在本地启动 Azure 函数。

    ```Shell
    func start
    ```

1. 如果尚未为 SPA 提供服务，请打开第二个 CLI 窗口，并将当前目录更改为 **/TestClient** 目录。 运行以下命令以运行测试应用程序。

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. 打开浏览器，并导航到 `http://localhost:8080`。 登录并选择 **最新的邮件** 导航项。 应用程序将显示有关用户的收件箱中最新邮件的信息。
