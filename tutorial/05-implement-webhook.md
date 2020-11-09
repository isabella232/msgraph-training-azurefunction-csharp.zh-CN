---
ms.openlocfilehash: f55eecd4d157c945d0e95870d4e481653b26c9ec
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822560"
---
<!-- markdownlint-disable MD002 MD041 -->

在本练习中，你将完成实现 Azure 功能 `SetSubscription` 和 `Notify` 更新测试应用程序以订阅和取消订阅用户收件箱中的更改。

- `SetSubscription`函数将充当 API，允许测试应用创建或删除对用户的收件箱中的更改的[订阅](https://docs.microsoft.com/graph/webhooks)。
- 该 `Notify` 函数将充当接收由订阅生成的更改通知的 webhook。

这两个函数都将使用 [客户端凭据授予流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) 获取用于调用 Microsoft Graph 的仅应用程序令牌。 由于管理员向管理员授予所需的权限范围，因此无需任何用户交互即可获取令牌。

## <a name="add-client-credentials-authentication-to-the-azure-functions-project"></a>将客户端凭据身份验证添加到 Azure 函数项目

在本节中，你将在 Azure 函数项目中实现客户端凭据流，以获取与 Microsoft Graph 兼容的访问令牌。

1. 在包含 **GraphTutorial** 的目录中打开您的 CLI。

1. 使用以下命令将 webhook 应用程序 ID 和密码添加到密钥存储中。 将替换为 `YOUR_WEBHOOK_APP_ID_HERE` **Graph Azure 函数 Webhook** 的应用程序 ID。 将替换为 `YOUR_WEBHOOK_APP_SECRET_HERE` 在 azure 门户中为 **Graph Azure 函数 Webhook** 创建的应用程序密码。

    ```Shell
    dotnet user-secrets set webHookId "YOUR_WEBHOOK_APP_ID_HERE"
    dotnet user-secrets set webHookSecret "YOUR_WEBHOOK_APP_SECRET_HERE"
    ```

### <a name="create-a-client-credentials-authentication-provider"></a>创建客户端凭据身份验证提供程序

1. 在名为 **ClientCredentialsAuthProvider.cs** 的 **/GraphTutorial/Authentication** 目录中创建一个新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/ClientCredentialsAuthProvider.cs" id="AuthProviderSnippet":::

请花点时间考虑 **ClientCredentialsAuthProvider.cs** 中的代码的功能。

- 在构造函数中，它从包中初始化 **ConfidentialClientApplication** `Microsoft.Identity.Client` 。 它使用 `WithAuthority(AadAuthorityAudience.AzureAdMyOrg, true)` 和 `.WithTenantId(tenantId)` 函数将登录访问群体限制为仅指定的 Microsoft 365 组织。
- 在 `GetAccessToken` 函数中，它调用 `AcquireTokenForClient` 以获取应用程序的令牌。 客户端凭据令牌流始终是非交互式的。
- 它实现 `Microsoft.Graph.IAuthenticationProvider` 接口，允许在中的构造函数中传递此类以对 `GraphServiceClient` 传出请求进行身份验证。

## <a name="update-graphclientservice"></a>更新 GraphClientService

1. 打开 **GraphClientService.cs** ，并将以下属性添加到类中。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

1. 将现有的 `GetAppGraphClient` 函数替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="AppGraphClientMembers":::

## <a name="implement-notify-function"></a>实现通知函数

在本节中，您将实现 `Notify` 将用作更改通知的通知 URL 的函数。

1. 在名为 " **模型** " 的 **GraphTutorials** 目录中创建一个新目录。

1. 在名为 " **ResourceData.cs** " 的 **模型** 目录中创建一个新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/ResourceData.cs" id="ResourceDataSnippet":::

1. 在名为 " **ChangeNotification.cs** " 的 **模型** 目录中创建一个新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/ChangeNotification.cs" id="ChangeNotificationSnippet":::

1. 在名为 " **NotificationList.cs** " 的 **模型** 目录中创建一个新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/NotificationList.cs" id="NotificationListSnippet":::

1. 打开 **/GraphTutorial/Notify.cs** ，并将其全部内容替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/Notify.cs" id="NotifySnippet":::

请花点时间考虑 **Notify.cs** 中的代码的功能。

- `Run`函数检查是否存在 `validationToken` 查询参数。 如果存在此参数，则它会将请求作为 [验证请求](https://docs.microsoft.com/graph/webhooks#notification-endpoint-validation)进行处理，并相应地做出响应。
- 如果请求不是验证请求，则 JSON 有效负载被反序列化为 `NotificationList` 。
- 将检查列表中的每个通知以查找预期的客户端状态值，并对其进行处理。
- 将使用 Microsoft Graph 检索触发通知的消息。

## <a name="implement-setsubscription-function"></a>实现 SetSubscription 函数

在本节中，您将实现 SetSubscription 函数。 此函数将充当测试应用程序调用的 API，以在用户的收件箱中创建或删除订阅。

1. 在名为 " **SetSubscriptionPayload.cs** " 的 **模型** 目录中创建一个新文件，并添加以下代码。

    :::code language="csharp" source="../demo/GraphTutorial/Models/SetSubscriptionPayload.cs" id="SetSubscriptionPayloadSnippet":::

1. 打开 **/GraphTutorial/SetSubscription.cs** ，并将其全部内容替换为以下内容。

    :::code language="csharp" source="../demo/GraphTutorial/SetSubscription.cs" id="SetSubscriptionSnippet":::

请花点时间考虑 **SetSubscription.cs** 中的代码的功能。

- `Run`函数读取在 POST 请求中发送的 JSON 有效负载，以确定请求类型 (订阅或取消订阅) 、订阅的用户 id 以及订阅 ID 取消订阅。
- 如果请求是订阅请求，则使用 Microsoft Graph SDK 在指定用户的收件箱中创建新的订阅。 在创建或更新邮件时，订阅将会发出通知。 新订阅在响应的 JSON 有效负载中返回。
- 如果请求是取消订阅请求，则使用 Microsoft Graph SDK 删除指定的订阅。

## <a name="call-setsubscription-from-the-test-app"></a>从测试应用程序调用 SetSubscription

在本节中，您将实现在测试应用程序中创建和删除订阅的函数。

1. 打开 **/TestClient/azurefunctions.js** 并添加以下函数。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="createSubscriptionSnippet":::

    此代码调用 `SetSubscription` Azure 函数来订阅，并将新订阅添加到会话中的订阅数组中。

1. 将以下函数添加到 **azurefunctions.js** 。

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="deleteSubscriptionSnippet":::

    此代码调用 `SetSubscription` Azure 函数，从会话中的订阅数组中删除订阅。

1. 如果您没有 ngrok 在运行，请运行 ngrok (`ngrok http 7071`) 并复制 HTTPS 转发 URL。

1. 运行以下命令，将 ngrok URL 添加到用户密码存储中。

    ```Shell
    dotnet user-secrets set ngrokUrl "YOUR_NGROK_URL_HERE"
    ```

    > [!IMPORTANT]
    > 如果重新启动 ngrok，您将需要重复此命令来更新 ngrok URL。

1. 将您的 CLI 中的当前目录更改为 **/GraphTutorial** 目录，并运行以下命令以在本地启动 Azure 函数。

    ```Shell
    func start
    ```

1. 刷新 SPA 并选择 " **订阅** " 导航项。 输入包含 Exchange Online 邮箱的 Microsoft 365 组织中的用户的用户 ID。 这可以是 `id` 来自 Microsoft Graph 的用户 () 或用户的 `userPrincipalName` 。 单击 " **订阅** "。

1. 页面刷新显示表中的新订阅。

1. 向用户发送电子邮件。 经过一小段时间后， `Notify` 应调用该函数。 您可以在 ngrok web 界面 (`http://localhost:4040`) 或在 Azure Function 项目的调试输出中进行验证。

    ```Shell
    ...
    [7/8/2020 7:33:57 PM] The following message was created:
    [7/8/2020 7:33:57 PM] Subject: Hi Megan!, ID: AAMkAGUyN2I4N2RlLTEzMTAtNDBmYy1hODdlLTY2NTQwODE2MGEwZgBGAAAAAAA2J9QH-DvMRK3pBt_8rA6nBwCuPIFjbMEkToHcVnQirM5qAAAAAAEMAACuPIFjbMEkToHcVnQirM5qAACHmpAsAAA=
    [7/8/2020 7:33:57 PM] Executed 'Notify' (Succeeded, Id=9c40af0b-e082-4418-aa3a-aee624f30e7a)
    ...
    ```

1. 在测试应用程序中，单击订阅的表行中的 " **删除** "。 页面将刷新，且订阅不再位于表中。
