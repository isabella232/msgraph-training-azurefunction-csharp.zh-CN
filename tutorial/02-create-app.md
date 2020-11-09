---
ms.openlocfilehash: 003d7f382a0c950d31bb84176feb1e83fbd5cf19
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822573"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="79248-101">在本教程中，将创建一个简单的 Azure 函数，该函数实现调用 Microsoft Graph 的 HTTP 触发器函数。</span><span class="sxs-lookup"><span data-stu-id="79248-101">In this tutorial, you will create a simple Azure Function that implements HTTP trigger functions that call Microsoft Graph.</span></span> <span data-ttu-id="79248-102">这些函数将涵盖以下方案：</span><span class="sxs-lookup"><span data-stu-id="79248-102">These functions will cover the following scenarios:</span></span>

- <span data-ttu-id="79248-103">使用 [代表流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) 身份验证来实现 API 以访问用户的收件箱。</span><span class="sxs-lookup"><span data-stu-id="79248-103">Implements an API to access a user's inbox using [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) authentication.</span></span>
- <span data-ttu-id="79248-104">实现 API 以订阅和取消订阅用户收件箱上的通知，使用 [客户端凭据授予流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) 身份验证。</span><span class="sxs-lookup"><span data-stu-id="79248-104">Implements an API to subscribe and unsubscribe for notifications on a user's inbox, using using [client credentials grant flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) authentication.</span></span>
- <span data-ttu-id="79248-105">实现 webhook 以接收来自 Microsoft Graph 的 [更改通知](https://docs.microsoft.com/graph/webhooks) 并使用客户端凭据授予流访问数据。</span><span class="sxs-lookup"><span data-stu-id="79248-105">Implements a webhook to receive [change notifications](https://docs.microsoft.com/graph/webhooks) from Microsoft Graph and access data using client credentials grant flow.</span></span>

<span data-ttu-id="79248-106">您还将创建一个简单的 JavaScript 单页面应用程序 (SPA) ，以调用在 Azure 函数中实现的 Api。</span><span class="sxs-lookup"><span data-stu-id="79248-106">You will also create a simple JavaScript single-page application (SPA) to call the APIs implemented in the Azure Function.</span></span>

## <a name="create-azure-functions-project"></a><span data-ttu-id="79248-107">创建 Azure 函数项目</span><span class="sxs-lookup"><span data-stu-id="79248-107">Create Azure Functions project</span></span>

1. <span data-ttu-id="79248-108">在要创建项目的目录中打开命令行界面 (CLI) 。</span><span class="sxs-lookup"><span data-stu-id="79248-108">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="79248-109">运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="79248-109">Run the following command.</span></span>

    ```Shell
    func init GraphTutorial --dotnet
    ```

1. <span data-ttu-id="79248-110">将 CLI 中的当前目录更改为 **GraphTutorial** 目录，并运行以下命令以在项目中创建三个函数。</span><span class="sxs-lookup"><span data-stu-id="79248-110">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands to create three functions in the project.</span></span>

    ```Shell
    func new --name GetMyNewestMessage --template "HTTP trigger" --language C#
    func new --name SetSubscription --template "HTTP trigger" --language C#
    func new --name Notify --template "HTTP trigger" --language C#
    ```

1. <span data-ttu-id="79248-111">打开 **local.settings.js** ，并将以下各项添加到文件中，以允许从 `http://localhost:8080` 测试应用程序的 URL 中进行 CORS。</span><span class="sxs-lookup"><span data-stu-id="79248-111">Open **local.settings.json** and add the following to the file to allow CORS from `http://localhost:8080`, the URL for the test application.</span></span>

    ```json
    "Host": {
      "CORS": "http://localhost:8080"
    }
    ```

1. <span data-ttu-id="79248-112">运行以下命令以在本地运行该项目。</span><span class="sxs-lookup"><span data-stu-id="79248-112">Run the following command to run the project locally.</span></span>

    ```Shell
    func start
    ```

1. <span data-ttu-id="79248-113">如果一切正常，您将看到以下输出：</span><span class="sxs-lookup"><span data-stu-id="79248-113">If everything is working, you will see the following output:</span></span>

    ```Shell
    Http Functions:

        GetMyNewestMessage: [GET,POST] http://localhost:7071/api/GetMyNewestMessage

        Notify: [GET,POST] http://localhost:7071/api/Notify

        SetSubscription: [GET,POST] http://localhost:7071/api/SetSubscription
    ```

1. <span data-ttu-id="79248-114">打开浏览器并浏览到输出中显示的函数 Url，以验证这些函数是否正常工作。</span><span class="sxs-lookup"><span data-stu-id="79248-114">Verify that the functions are working correctly by opening your browser and browsing to the function URLs shown in the output.</span></span> <span data-ttu-id="79248-115">您应该会在浏览器中看到以下消息： `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.` 。</span><span class="sxs-lookup"><span data-stu-id="79248-115">You should see the following message in your browser: `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.`.</span></span>

## <a name="create-single-page-application"></a><span data-ttu-id="79248-116">创建单页面应用程序</span><span class="sxs-lookup"><span data-stu-id="79248-116">Create single-page application</span></span>

1. <span data-ttu-id="79248-117">在要创建项目的目录中打开您的 CLI。</span><span class="sxs-lookup"><span data-stu-id="79248-117">Open your CLI in a directory where you want to create the project.</span></span> <span data-ttu-id="79248-118">创建一个名为 **TestClient** 的目录以保存您的 HTML 和 JavaScript 文件。</span><span class="sxs-lookup"><span data-stu-id="79248-118">Create a directory named **TestClient** to hold your HTML and JavaScript files.</span></span>

1. <span data-ttu-id="79248-119">在 **TestClient** 目录中创建一个名为 **index.html** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="79248-119">Create a new file named **index.html** in the **TestClient** directory and add the following code.</span></span>

    :::code language="html" source="../demo/TestClient/index.html" id="indexSnippet":::

    <span data-ttu-id="79248-120">这将定义应用程序的基本布局，包括导航栏。</span><span class="sxs-lookup"><span data-stu-id="79248-120">This defines the basic layout of the app, including a navigation bar.</span></span> <span data-ttu-id="79248-121">此外，它还会添加以下内容：</span><span class="sxs-lookup"><span data-stu-id="79248-121">It also adds the following:</span></span>

    - <span data-ttu-id="79248-122">[引导数据库](https://getbootstrap.com/) 及其支持的 JavaScript</span><span class="sxs-lookup"><span data-stu-id="79248-122">[Bootstrap](https://getbootstrap.com/) and its supporting JavaScript</span></span>
    - [<span data-ttu-id="79248-123">FontAwesome</span><span class="sxs-lookup"><span data-stu-id="79248-123">FontAwesome</span></span>](https://fontawesome.com/)
    - [<span data-ttu-id="79248-124">适用于 JavaScript 的 Microsoft 身份验证库 ( # A0) 2。0</span><span class="sxs-lookup"><span data-stu-id="79248-124">Microsoft Authentication Library for JavaScript (MSAL.js) 2.0</span></span>](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

    > [!TIP]
    > <span data-ttu-id="79248-125">页面包含一个 favicon， (`<link rel="shortcut icon" href="g-raph.png">`) 。</span><span class="sxs-lookup"><span data-stu-id="79248-125">The page includes a favicon, (`<link rel="shortcut icon" href="g-raph.png">`).</span></span> <span data-ttu-id="79248-126">您可以删除此行，也可以从 [GitHub](https://github.com/microsoftgraph/g-raph)下载 **g-raph.png** 文件。</span><span class="sxs-lookup"><span data-stu-id="79248-126">You can remove this line, or you can download the **g-raph.png** file from [GitHub](https://github.com/microsoftgraph/g-raph).</span></span>

1. <span data-ttu-id="79248-127">在 **TestClient** 目录中创建一个名为 **style** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="79248-127">Create a new file named **style.css** in the **TestClient** directory and add the following code.</span></span>

    :::code language="css" source="../demo/TestClient/style.css":::

1. <span data-ttu-id="79248-128">在 **TestClient** 目录中创建一个名为 **ui.js** 的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="79248-128">Create a new file named **ui.js** in the **TestClient** directory and add the following code.</span></span>

    :::code language="javascript" source="../demo/TestClient/ui.js" id="uiJsSnippet":::

    <span data-ttu-id="79248-129">此代码使用 JavaScript 根据所选视图呈现当前页面。</span><span class="sxs-lookup"><span data-stu-id="79248-129">This code uses JavaScript to render the current page based on the selected view.</span></span>

### <a name="test-the-single-page-application"></a><span data-ttu-id="79248-130">测试单页面应用程序</span><span class="sxs-lookup"><span data-stu-id="79248-130">Test the single-page application</span></span>

> [!NOTE]
> <span data-ttu-id="79248-131">本节包含有关如何使用 [dotnet](https://github.com/natemcmaster/dotnet-serve) 来在开发计算机上运行简单测试 HTTP 服务器的说明。</span><span class="sxs-lookup"><span data-stu-id="79248-131">This section includes instructions for using [dotnet-serve](https://github.com/natemcmaster/dotnet-serve) to run a simple testing HTTP server on your development machine.</span></span> <span data-ttu-id="79248-132">不需要使用此特定工具。</span><span class="sxs-lookup"><span data-stu-id="79248-132">Using this specific tool is not required.</span></span> <span data-ttu-id="79248-133">您可以使用任何您希望为 **TestClient** 目录提供服务的测试服务器。</span><span class="sxs-lookup"><span data-stu-id="79248-133">You can use any testing server you prefer to serve the **TestClient** directory.</span></span>

1. <span data-ttu-id="79248-134">在 CLI 中运行以下命令，以安装 **dotnet 服务** 。</span><span class="sxs-lookup"><span data-stu-id="79248-134">Run the following command in your CLI to install **dotnet-serve**.</span></span>

    ```Shell
    dotnet tool install --global dotnet-serve
    ```

1. <span data-ttu-id="79248-135">将 CLI 中的当前目录更改为 **TestClient** 目录，并运行以下命令以启动 HTTP 服务器。</span><span class="sxs-lookup"><span data-stu-id="79248-135">Change the current directory in your CLI to the **TestClient** directory and run the following command to start an HTTP server.</span></span>

    ```Shell
    dotnet serve -h "Cache-Control: no-cache, no-store, must-revalidate"
    ```

1. <span data-ttu-id="79248-136">打开浏览器，并导航到 `http://localhost:8080`。</span><span class="sxs-lookup"><span data-stu-id="79248-136">Open your browser and navigate to `http://localhost:8080`.</span></span> <span data-ttu-id="79248-137">页面应呈现，但当前没有任何按钮可用。</span><span class="sxs-lookup"><span data-stu-id="79248-137">The page should render, but none of the buttons currently work.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="79248-138">添加 NuGet 包</span><span class="sxs-lookup"><span data-stu-id="79248-138">Add NuGet packages</span></span>

<span data-ttu-id="79248-139">在继续之前，请安装稍后将使用的一些其他 NuGet 包。</span><span class="sxs-lookup"><span data-stu-id="79248-139">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="79248-140">在 Azure 函数项目中启用依赖关系注入的[扩展名](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions)。</span><span class="sxs-lookup"><span data-stu-id="79248-140">[Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions) to enable dependency injection in the Azure Functions project.</span></span>
- <span data-ttu-id="79248-141">[Microsoft.Extensions.Configuration。UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) 从 [.net 开发密钥存储](https://docs.microsoft.com/aspnet/core/security/app-secrets)读取应用程序配置。</span><span class="sxs-lookup"><span data-stu-id="79248-141">[Microsoft.Extensions.Configuration.UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) to read application configuration from the [.NET development secret store](https://docs.microsoft.com/aspnet/core/security/app-secrets).</span></span>
- <span data-ttu-id="79248-142">用于调用 Microsoft Graph 的[microsoft](https://www.nuget.org/packages/Microsoft.Graph/) graph。</span><span class="sxs-lookup"><span data-stu-id="79248-142">[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph/) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="79248-143">用于对令牌进行身份验证和管理的 " [Microsoft 身份" 客户端](https://www.nuget.org/packages/Microsoft.Identity.Client/)。</span><span class="sxs-lookup"><span data-stu-id="79248-143">[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) for authenticating and managing tokens.</span></span>
- <span data-ttu-id="79248-144">[Microsoft.identitymodel.dll](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) 用于检索令牌验证的 OpenID 配置的 OpenIdConnect。</span><span class="sxs-lookup"><span data-stu-id="79248-144">[Microsoft.IdentityModel.Protocols.OpenIdConnect](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocols.OpenIdConnect) for retrieving OpenID configuration for token validation.</span></span>
- <span data-ttu-id="79248-145">用于验证已发送到 web API 的令牌的[Microsoft.identitymodel.dll Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) 。</span><span class="sxs-lookup"><span data-stu-id="79248-145">[System.IdentityModel.Tokens.Jwt](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) for validating tokens sent to the web API.</span></span>

1. <span data-ttu-id="79248-146">将 CLI 中的当前目录更改为 **GraphTutorial** 目录，并运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="79248-146">Change the current directory in your CLI to the **GraphTutorial** directory and run the following commands.</span></span>

    ```Shell
    dotnet add package Microsoft.Azure.Functions.Extensions --version 1.0.0
    dotnet add package Microsoft.Extensions.Configuration.UserSecrets --version 3.1.5
    dotnet add package Microsoft.Graph --version 3.8.0
    dotnet add package Microsoft.Identity.Client --version 4.15.0
    dotnet add package Microsoft.IdentityModel.Protocols.OpenIdConnect --version 6.7.1
    dotnet add package System.IdentityModel.Tokens.Jwt --version 6.7.1
    ```
