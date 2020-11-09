---
ms.openlocfilehash: 728138b5d298f9469a566defed6ee2e457896c0b
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822585"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="8cab8-101">在本练习中，你将完成 Azure 函数的实现 `GetMyNewestMessage` 并更新测试客户端以调用函数。</span><span class="sxs-lookup"><span data-stu-id="8cab8-101">In this exercise you will finish implementing the Azure Function `GetMyNewestMessage` and update the test client to call the function.</span></span>

<span data-ttu-id="8cab8-102">Azure 函数使用 [代表流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)。</span><span class="sxs-lookup"><span data-stu-id="8cab8-102">The Azure Function uses the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow).</span></span> <span data-ttu-id="8cab8-103">此流中事件的基本顺序为：</span><span class="sxs-lookup"><span data-stu-id="8cab8-103">The basic order of events in this flow are:</span></span>

- <span data-ttu-id="8cab8-104">测试应用程序使用交互式身份验证流，以允许用户登录并授予同意。</span><span class="sxs-lookup"><span data-stu-id="8cab8-104">The test application uses an interactive auth flow to allow the user to sign in and grant consent.</span></span> <span data-ttu-id="8cab8-105">它将返回范围限定为 Azure 函数的令牌。</span><span class="sxs-lookup"><span data-stu-id="8cab8-105">It gets back a token that is scoped to the Azure Function.</span></span> <span data-ttu-id="8cab8-106">令牌 **不包含任何** Microsoft Graph 作用域。</span><span class="sxs-lookup"><span data-stu-id="8cab8-106">The token does **NOT** contain any Microsoft Graph scopes.</span></span>
- <span data-ttu-id="8cab8-107">测试应用程序将调用 Azure 函数，并在标头中发送其访问令牌 `Authorization` 。</span><span class="sxs-lookup"><span data-stu-id="8cab8-107">The test application invokes the Azure Function, sending its access token in the `Authorization` header.</span></span>
- <span data-ttu-id="8cab8-108">Azure 函数验证令牌，然后将该令牌与包含 Microsoft Graph 作用域的第二个访问令牌交换。</span><span class="sxs-lookup"><span data-stu-id="8cab8-108">The Azure Function validates the token, then exchanges that token for a second access token that contains Microsoft Graph scopes.</span></span>
- <span data-ttu-id="8cab8-109">Azure 函数代表用户使用第二个访问令牌调用 Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="8cab8-109">The Azure Function calls Microsoft Graph on the user's behalf using the second access token.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="8cab8-110">为了避免在源中存储应用程序 ID 和密码，您将使用 [.Net 密钥管理器](https://docs.microsoft.com/aspnet/core/security/app-secrets) 来存储这些值。</span><span class="sxs-lookup"><span data-stu-id="8cab8-110">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](https://docs.microsoft.com/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="8cab8-111">机密管理器仅用于开发目的，而生产应用程序应使用受信任密钥管理器来存储机密信息。</span><span class="sxs-lookup"><span data-stu-id="8cab8-111">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

## <a name="add-authentication-to-the-single-page-application"></a><span data-ttu-id="8cab8-112">向单页面应用程序添加身份验证</span><span class="sxs-lookup"><span data-stu-id="8cab8-112">Add authentication to the single page application</span></span>

<span data-ttu-id="8cab8-113">首先向 SPA 添加身份验证。</span><span class="sxs-lookup"><span data-stu-id="8cab8-113">Start by adding authentication to the SPA.</span></span> <span data-ttu-id="8cab8-114">这将允许应用程序获取访问令牌，授予访问权限以调用 Azure 函数。</span><span class="sxs-lookup"><span data-stu-id="8cab8-114">This will allow the application to get an access token granting access to call the Azure Function.</span></span> <span data-ttu-id="8cab8-115">因为这是一个 SPA，所以它将对 [PKCE 使用授权代码流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow)。</span><span class="sxs-lookup"><span data-stu-id="8cab8-115">Because this is a SPA, it will use the [authorization code flow with PKCE](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-auth-code-flow).</span></span>

1. <span data-ttu-id="8cab8-116">在名为 **config.js** 的 **TestClient** 目录中创建一个新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="8cab8-116">Create a new file in the **TestClient** directory named **config.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/config.example.js" id="msalConfigSnippet":::

    <span data-ttu-id="8cab8-117">`YOUR_TEST_APP_APP_ID_HERE`将替换为您在 azure 门户中为 **Graph Azure 函数测试应用** 程序创建的应用程序 ID。</span><span class="sxs-lookup"><span data-stu-id="8cab8-117">Replace `YOUR_TEST_APP_APP_ID_HERE` with the application ID you created in the Azure portal for the **Graph Azure Function Test App**.</span></span> <span data-ttu-id="8cab8-118">`YOUR_TENANT_ID_HERE`将替换为您从 Azure 门户复制的 **目录 (租户) ID** 值。</span><span class="sxs-lookup"><span data-stu-id="8cab8-118">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span> <span data-ttu-id="8cab8-119">将替换为 `YOUR_AZURE_FUNCTION_APP_ID_HERE` **Graph Azure 函数** 的应用程序 ID。</span><span class="sxs-lookup"><span data-stu-id="8cab8-119">Replace `YOUR_AZURE_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="8cab8-120">如果您使用的是源代码管理（如 git），现在可以从源代码管理中排除 **config.js** 文件，以避免在无意中泄漏您的应用程序 id 和租户 ID。</span><span class="sxs-lookup"><span data-stu-id="8cab8-120">If you're using source control such as git, now would be a good time to exclude the **config.js** file from source control to avoid inadvertently leaking your app IDs and tenant ID.</span></span>

1. <span data-ttu-id="8cab8-121">在名为 **auth.js** 的 **TestClient** 目录中创建一个新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="8cab8-121">Create a new file in the **TestClient** directory named **auth.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="signInSignOutSnippet":::

    <span data-ttu-id="8cab8-122">请考虑此代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="8cab8-122">Consider what this code does.</span></span>

    - <span data-ttu-id="8cab8-123">它 `PublicClientApplication` 使用存储在 **config.js** 中的值对 a 进行初始化。</span><span class="sxs-lookup"><span data-stu-id="8cab8-123">It initializes a `PublicClientApplication` using the values stored in **config.js**.</span></span>
    - <span data-ttu-id="8cab8-124">使用 `loginPopup` Azure 函数的权限范围在中对用户进行签名。</span><span class="sxs-lookup"><span data-stu-id="8cab8-124">It uses `loginPopup` to sign the user in, using the permission scope for the Azure Function.</span></span>
    - <span data-ttu-id="8cab8-125">它将用户的用户名存储在会话中。</span><span class="sxs-lookup"><span data-stu-id="8cab8-125">It stores the user's username in the session.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="8cab8-126">由于应用程序使用 `loginPopup` ，您可能需要将浏览器的弹出窗口阻止程序更改为允许弹出窗口阻止程序 `http://localhost:8080` 。</span><span class="sxs-lookup"><span data-stu-id="8cab8-126">Since the app uses `loginPopup`, you may need to change your browser's pop-up blocker to allow pop-ups from `http://localhost:8080`.</span></span>

1. <span data-ttu-id="8cab8-127">刷新页面并登录。</span><span class="sxs-lookup"><span data-stu-id="8cab8-127">Refresh the page and sign in.</span></span> <span data-ttu-id="8cab8-128">页面应使用用户名进行更新，这表明登录成功。</span><span class="sxs-lookup"><span data-stu-id="8cab8-128">The page should update with the user name, indicating that the sign in was successful.</span></span>

> [!TIP]
> <span data-ttu-id="8cab8-129">您可以在中分析访问令牌， [https://jwt.ms](https://jwt.ms) 并确认 `aud` 声明是 azure 函数的应用程序 ID，并且 `scp` 声明包含 azure 函数的权限范围，而不是 Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="8cab8-129">You can parse the access token at [https://jwt.ms](https://jwt.ms) and confirm that the `aud` claim is the app ID for the Azure Function, and that the `scp` claim contains the Azure Function's permission scope, not Microsoft Graph.</span></span>

## <a name="add-authentication-to-the-azure-function"></a><span data-ttu-id="8cab8-130">向 Azure 函数添加身份验证</span><span class="sxs-lookup"><span data-stu-id="8cab8-130">Add authentication to the Azure Function</span></span>

<span data-ttu-id="8cab8-131">在本节中，你将在 Azure 函数中实施代表流 `GetMyNewestMessage` ，以获取与 Microsoft Graph 兼容的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="8cab8-131">In this section you'll implement the on-behalf-of flow in the `GetMyNewestMessage` Azure Function to get an access token compatible with Microsoft Graph.</span></span>

1. <span data-ttu-id="8cab8-132">通过在包含 **GraphTutorial** 的目录中打开 CLI 并运行以下命令来初始化 .net 开发密钥存储。</span><span class="sxs-lookup"><span data-stu-id="8cab8-132">Initialize the .NET development secret store by opening your CLI in the directory that contains **GraphTutorial.csproj** and running the following command.</span></span>

    ```Shell
    dotnet user-secrets init
    ```

1. <span data-ttu-id="8cab8-133">使用以下命令将应用程序 ID、密码和租户 ID 添加到密钥存储。</span><span class="sxs-lookup"><span data-stu-id="8cab8-133">Add your application ID, secret, and tenant ID to the secret store using the following commands.</span></span> <span data-ttu-id="8cab8-134">将替换为 `YOUR_API_FUNCTION_APP_ID_HERE` **Graph Azure 函数** 的应用程序 ID。</span><span class="sxs-lookup"><span data-stu-id="8cab8-134">Replace `YOUR_API_FUNCTION_APP_ID_HERE` with the application ID for the **Graph Azure Function**.</span></span> <span data-ttu-id="8cab8-135">`YOUR_API_FUNCTION_APP_SECRET_HERE`将替换为在 azure 门户中为 **Graph azure 函数** 创建的应用程序密码。</span><span class="sxs-lookup"><span data-stu-id="8cab8-135">Replace `YOUR_API_FUNCTION_APP_SECRET_HERE` with the application secret you created in the Azure portal for the **Graph Azure Function**.</span></span> <span data-ttu-id="8cab8-136">`YOUR_TENANT_ID_HERE`将替换为您从 Azure 门户复制的 **目录 (租户) ID** 值。</span><span class="sxs-lookup"><span data-stu-id="8cab8-136">Replace `YOUR_TENANT_ID_HERE` with the **Directory (tenant) ID** value you copied from the Azure portal.</span></span>

    ```Shell
    dotnet user-secrets set apiFunctionId "YOUR_API_FUNCTION_APP_ID_HERE"
    dotnet user-secrets set apiFunctionSecret "YOUR_API_FUNCTION_APP_SECRET_HERE"
    dotnet user-secrets set tenantId "YOUR_TENANT_ID_HERE"
    ```

### <a name="process-the-incoming-bearer-token"></a><span data-ttu-id="8cab8-137">处理传入持有者令牌</span><span class="sxs-lookup"><span data-stu-id="8cab8-137">Process the incoming bearer token</span></span>

<span data-ttu-id="8cab8-138">在本节中，您将实现一个类，用于验证和处理从 SPA 发送到 Azure 函数的载荷令牌。</span><span class="sxs-lookup"><span data-stu-id="8cab8-138">In this section you'll implement a class to validate and process the bearer token sent from the SPA to the Azure Function.</span></span>

1. <span data-ttu-id="8cab8-139">在名为 **Authentication** 的 **GraphTutorial** 目录中创建一个新目录。</span><span class="sxs-lookup"><span data-stu-id="8cab8-139">Create a new directory in the **GraphTutorial** directory named **Authentication**.</span></span>

1. <span data-ttu-id="8cab8-140">在 **./GraphTutorial/Authentication** 文件夹中创建一个名为 **TokenValidationResult.cs** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="8cab8-140">Create a new file named **TokenValidationResult.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidationResult.cs" id="TokenValidationResultSnippet":::

1. <span data-ttu-id="8cab8-141">在 **./GraphTutorial/Authentication** 文件夹中创建一个名为 **TokenValidation.cs** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="8cab8-141">Create a new file named **TokenValidation.cs** in the **./GraphTutorial/Authentication** folder, and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/TokenValidation.cs" id="TokenValidationSnippet":::

<span data-ttu-id="8cab8-142">请考虑此代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="8cab8-142">Consider what this code does.</span></span>

- <span data-ttu-id="8cab8-143">它确保标头中存在持有者令牌 `Authorization` 。</span><span class="sxs-lookup"><span data-stu-id="8cab8-143">It ensure there is a bearer token in the `Authorization` header.</span></span>
- <span data-ttu-id="8cab8-144">它将验证来自 Azure 的已发布 OpenID 配置的签名和颁发者。</span><span class="sxs-lookup"><span data-stu-id="8cab8-144">It verifies the signature and issuer from Azure's published OpenID configuration.</span></span>
- <span data-ttu-id="8cab8-145">它验证访问群体 (`aud` 声明是否与 Azure 函数的应用程序 ID) 匹配。</span><span class="sxs-lookup"><span data-stu-id="8cab8-145">It verifies that the audience (`aud` claim) matches the Azure Function's application ID.</span></span>
- <span data-ttu-id="8cab8-146">它分析令牌并生成 MSAL 帐户 ID，这将需要利用令牌缓存。</span><span class="sxs-lookup"><span data-stu-id="8cab8-146">It parses the token and generates an MSAL account ID, which will be needed to take advantage of token caching.</span></span>

### <a name="create-an-on-behalf-of-authentication-provider"></a><span data-ttu-id="8cab8-147">创建代表身份验证提供程序</span><span class="sxs-lookup"><span data-stu-id="8cab8-147">Create an on-behalf-of authentication provider</span></span>

1. <span data-ttu-id="8cab8-148">在名为 " **OnBehalfOfAuthProvider.cs** " 的 **身份验证** 目录中创建一个新文件，并将以下代码添加到该文件中。</span><span class="sxs-lookup"><span data-stu-id="8cab8-148">Create a new file in the **Authentication** directory named **OnBehalfOfAuthProvider.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Authentication/OnBehalfOfAuthProvider.cs" id="AuthProviderSnippet":::

<span data-ttu-id="8cab8-149">请花点时间考虑 **OnBehalfOfAuthProvider.cs** 中的代码的功能。</span><span class="sxs-lookup"><span data-stu-id="8cab8-149">Take a moment to consider what the code in **OnBehalfOfAuthProvider.cs** does.</span></span>

- <span data-ttu-id="8cab8-150">在 `GetAccessToken` 函数中，它首先尝试使用从令牌缓存中获取用户令牌 `AcquireTokenSilent` 。</span><span class="sxs-lookup"><span data-stu-id="8cab8-150">In the `GetAccessToken` function, it first attempts to get a user token from the token cache using `AcquireTokenSilent`.</span></span> <span data-ttu-id="8cab8-151">如果此操作失败，它会将测试应用程序发送的持有者令牌用于 Azure 函数，以生成用户断言。</span><span class="sxs-lookup"><span data-stu-id="8cab8-151">If this fails, it uses the bearer token sent by the test app to the Azure Function to generate a user assertion.</span></span> <span data-ttu-id="8cab8-152">然后，它使用该用户断言获取与图形兼容的令牌（使用） `AcquireTokenOnBehalfOf` 。</span><span class="sxs-lookup"><span data-stu-id="8cab8-152">It then uses that user assertion to get a Graph-compatible token using `AcquireTokenOnBehalfOf`.</span></span>
- <span data-ttu-id="8cab8-153">它实现 `Microsoft.Graph.IAuthenticationProvider` 接口，允许在中的构造函数中传递此类以对 `GraphServiceClient` 传出请求进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="8cab8-153">It implements the `Microsoft.Graph.IAuthenticationProvider` interface, allowing this class to be passed in the constructor of the `GraphServiceClient` to authenticate outgoing requests.</span></span>

### <a name="implement-a-graph-client-service"></a><span data-ttu-id="8cab8-154">实现 Graph 客户端服务</span><span class="sxs-lookup"><span data-stu-id="8cab8-154">Implement a Graph client service</span></span>

<span data-ttu-id="8cab8-155">在本节中，你将实现可注册 [依赖项注入](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection)的服务。</span><span class="sxs-lookup"><span data-stu-id="8cab8-155">In this section you'll implement a service that can be registered for [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection).</span></span> <span data-ttu-id="8cab8-156">该服务将用于获取经过身份验证的 Graph 客户端。</span><span class="sxs-lookup"><span data-stu-id="8cab8-156">The service will be used to get an authenticated Graph client.</span></span>

1. <span data-ttu-id="8cab8-157">在名为 " **服务** " 的 **GraphTutorial** 目录中创建一个新目录。</span><span class="sxs-lookup"><span data-stu-id="8cab8-157">Create a new directory in the **GraphTutorial** directory named **Services**.</span></span>

1. <span data-ttu-id="8cab8-158">在名为 " **IGraphClientService.cs** " 的 **服务** 目录中创建一个新文件，并将以下代码添加到该文件中。</span><span class="sxs-lookup"><span data-stu-id="8cab8-158">Create a new file in the **Services** directory named **IGraphClientService.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/IGraphClientService.cs" id="IGraphClientServiceSnippet":::

1. <span data-ttu-id="8cab8-159">在名为 " **GraphClientService.cs** " 的 **服务** 目录中创建一个新文件，并将以下代码添加到该文件中。</span><span class="sxs-lookup"><span data-stu-id="8cab8-159">Create a new file in the **Services** directory named **GraphClientService.cs** and add the following code to that file.</span></span>

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

1. <span data-ttu-id="8cab8-160">将以下属性添加到 `GraphClientService` 类中。</span><span class="sxs-lookup"><span data-stu-id="8cab8-160">Add the following properties to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientMembers":::

1. <span data-ttu-id="8cab8-161">将以下函数添加到 `GraphClientService` 类中。</span><span class="sxs-lookup"><span data-stu-id="8cab8-161">Add the following functions to the `GraphClientService` class.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Services/GraphClientService.cs" id="UserGraphClientFunctions":::

1. <span data-ttu-id="8cab8-162">为函数添加占位符实现 `GetAppGraphClient` 。</span><span class="sxs-lookup"><span data-stu-id="8cab8-162">Add a placeholder implementation for the `GetAppGraphClient` function.</span></span> <span data-ttu-id="8cab8-163">您将在后面的部分中实现。</span><span class="sxs-lookup"><span data-stu-id="8cab8-163">You will implement that in later sections.</span></span>

    ```csharp
    public GraphServiceClient GetAppGraphClient()
    {
        throw new System.NotImplementedException();
    }
    ```

    <span data-ttu-id="8cab8-164">`GetUserGraphClient`函数采用令牌验证的结果，并为用户生成已通过身份验证的结果 `GraphServiceClient` 。</span><span class="sxs-lookup"><span data-stu-id="8cab8-164">The `GetUserGraphClient` function takes the results of token validation and builds an authenticated `GraphServiceClient` for the user.</span></span>

1. <span data-ttu-id="8cab8-165">在名为 **Startup.cs** 的 **GraphTutorial** 目录中创建一个新文件，并将以下代码添加到该文件中。</span><span class="sxs-lookup"><span data-stu-id="8cab8-165">Create a new file in the **GraphTutorial** directory named **Startup.cs** and add the following code to that file.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="StartupSnippet":::

    <span data-ttu-id="8cab8-166">此代码将在 Azure 函数中启用 [依赖关系注入](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) ， `IConfiguration` 从而公开对象和 `GraphClientService` 服务。</span><span class="sxs-lookup"><span data-stu-id="8cab8-166">This code will enable [dependency injection](https://docs.microsoft.com/azure/azure-functions/functions-dotnet-dependency-injection) in your Azure Functions, exposing the `IConfiguration` object and the `GraphClientService` service.</span></span>

### <a name="implement-getmynewestmessage-function"></a><span data-ttu-id="8cab8-167">实现 GetMyNewestMessage 函数</span><span class="sxs-lookup"><span data-stu-id="8cab8-167">Implement GetMyNewestMessage function</span></span>

1. <span data-ttu-id="8cab8-168">打开 **/GraphTutorial/GetMyNewestMessage.cs** ，并将其全部内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="8cab8-168">Open **./GraphTutorial/GetMyNewestMessage.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/GetMyNewestMessage.cs" id="GetMyNewestMessageSnippet":::

#### <a name="review-the-code-in-getmynewestmessagecs"></a><span data-ttu-id="8cab8-169">查看 GetMyNewestMessage.cs 中的代码</span><span class="sxs-lookup"><span data-stu-id="8cab8-169">Review the code in GetMyNewestMessage.cs</span></span>

<span data-ttu-id="8cab8-170">请花点时间考虑 **GetMyNewestMessage.cs** 中的代码的功能。</span><span class="sxs-lookup"><span data-stu-id="8cab8-170">Take a moment to consider what the code in **GetMyNewestMessage.cs** does.</span></span>

- <span data-ttu-id="8cab8-171">在构造函数中，它 `IConfiguration` 通过依赖关系注入保存传入的对象。</span><span class="sxs-lookup"><span data-stu-id="8cab8-171">In the constructor, it saves the `IConfiguration` object passed in via dependency injection.</span></span>
- <span data-ttu-id="8cab8-172">在 `Run` 函数中，它执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="8cab8-172">In the `Run` function, it does the following:</span></span>
  - <span data-ttu-id="8cab8-173">验证对象中是否存在所需的配置值 `IConfiguration` 。</span><span class="sxs-lookup"><span data-stu-id="8cab8-173">Validates the required configuration values are present in the `IConfiguration` object.</span></span>
  - <span data-ttu-id="8cab8-174">验证持有者令牌，并在 `401` 令牌无效时返回状态代码。</span><span class="sxs-lookup"><span data-stu-id="8cab8-174">Validates the bearer token and returns a `401` status code if the token is invalid.</span></span>
  - <span data-ttu-id="8cab8-175">从发出此请求的用户处获取图形客户端 `GraphClientService` 。</span><span class="sxs-lookup"><span data-stu-id="8cab8-175">Gets a Graph client from the `GraphClientService` for the user that made this request.</span></span>
  - <span data-ttu-id="8cab8-176">使用 Microsoft Graph SDK 从用户的收件箱获取最新邮件，并在响应中将其作为 JSON 正文返回。</span><span class="sxs-lookup"><span data-stu-id="8cab8-176">Uses the Microsoft Graph SDK to get the newest message from the user's inbox and returns it as a JSON body in the response.</span></span>

## <a name="call-the-azure-function-from-the-test-app"></a><span data-ttu-id="8cab8-177">从测试应用程序调用 Azure 函数</span><span class="sxs-lookup"><span data-stu-id="8cab8-177">Call the Azure Function from the test app</span></span>

1. <span data-ttu-id="8cab8-178">打开 **auth.js** 并添加以下函数以获取访问令牌。</span><span class="sxs-lookup"><span data-stu-id="8cab8-178">Open **auth.js** and add the following function to get an access token.</span></span>

    :::code language="javascript" source="../demo/TestClient/auth.js" id="getTokenSnippet":::

    <span data-ttu-id="8cab8-179">请考虑此代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="8cab8-179">Consider what this code does.</span></span>

    - <span data-ttu-id="8cab8-180">它首先尝试在没有用户交互的情况下以无提示方式获取访问令牌。</span><span class="sxs-lookup"><span data-stu-id="8cab8-180">It first attempts to get an access token silently, without user interaction.</span></span> <span data-ttu-id="8cab8-181">由于用户应已登录，因此 MSAL 应具有用户在其缓存中的令牌。</span><span class="sxs-lookup"><span data-stu-id="8cab8-181">Since the user should already be signed in, MSAL should have tokens for the user in its cache.</span></span>
    - <span data-ttu-id="8cab8-182">如果失败，并显示指示用户需要交互的错误，它将尝试以交互方式获取令牌。</span><span class="sxs-lookup"><span data-stu-id="8cab8-182">If that fails with an error that indicates the user needs to interact, it attempts to get a token interactively.</span></span>

1. <span data-ttu-id="8cab8-183">在名为 **azurefunctions.js** 的 **TestClient** 目录中创建一个新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="8cab8-183">Create a new file in the **TestClient** directory named **azurefunctions.js** and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/azurefunctions.js" id="getLatestMessageSnippet":::

1. <span data-ttu-id="8cab8-184">将您的 CLI 中的当前目录更改为 **/GraphTutorial** 目录，并运行以下命令以在本地启动 Azure 函数。</span><span class="sxs-lookup"><span data-stu-id="8cab8-184">Change the current directory in your CLI to the **./GraphTutorial** directory and run the following command to start the Azure Function locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="8cab8-185">如果尚未为 SPA 提供服务，请打开第二个 CLI 窗口，并将当前目录更改为 **/TestClient** 目录。</span><span class="sxs-lookup"><span data-stu-id="8cab8-185">If not already serving the SPA, open a second CLI window and change the current directory to the **./TestClient** directory.</span></span> <span data-ttu-id="8cab8-186">运行以下命令以运行测试应用程序。</span><span class="sxs-lookup"><span data-stu-id="8cab8-186">Run the following command to run the test application.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="8cab8-187">打开浏览器，并导航到 `http://localhost:8080`。</span><span class="sxs-lookup"><span data-stu-id="8cab8-187">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="8cab8-188">登录并选择 **最新的邮件** 导航项。</span><span class="sxs-lookup"><span data-stu-id="8cab8-188">Sign in and select the **Latest Message** navigation item.</span></span> <span data-ttu-id="8cab8-189">应用程序将显示有关用户的收件箱中最新邮件的信息。</span><span class="sxs-lookup"><span data-stu-id="8cab8-189">The app displays information about the newest message in the user's inbox.</span></span>
