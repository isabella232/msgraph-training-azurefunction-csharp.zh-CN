---
ms.openlocfilehash: 0c9c6703a54e50f576a171a4a161305c9b9ae6ff
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822583"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="241bf-101">在本练习中，将使用 Azure Active Directory 管理中心创建三个新的 Azure AD 应用程序：</span><span class="sxs-lookup"><span data-stu-id="241bf-101">In this exercise you will create three new Azure AD applications using the Azure Active Directory admin center:</span></span>

- <span data-ttu-id="241bf-102">单页面应用程序的应用程序注册，以便能够登录用户并获取允许应用程序调用 Azure 函数的令牌。</span><span class="sxs-lookup"><span data-stu-id="241bf-102">An app registration for the single-page application so that it can sign in users and get tokens allowing the application to call the Azure Function.</span></span>
- <span data-ttu-id="241bf-103">Azure 函数的应用程序注册，它允许它将由 SPA 发送的令牌用于交换由 SPA 发送的令牌 [，以允许](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) 它调用 Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="241bf-103">An app registration for the Azure Function that allows it to use the [on-behalf-of flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to exchange the token sent by the SPA for a token that will allow it to call Microsoft Graph.</span></span>
- <span data-ttu-id="241bf-104">Azure 函数 webhook 的应用程序注册，使其可以使用 [客户端凭据流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) 在没有用户的情况下调用 Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="241bf-104">An app registration for the Azure Function webhook that allows it to use the [client credential flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow) to call Microsoft Graph without a user.</span></span>

> [!NOTE]
> <span data-ttu-id="241bf-105">此示例需要三个应用注册，因为它同时实现了代表流和客户端凭据流。</span><span class="sxs-lookup"><span data-stu-id="241bf-105">This example requires three app registrations because it is implementing both the on-behalf-of flow and the client credential flow.</span></span> <span data-ttu-id="241bf-106">如果您的 Azure 函数只使用这些流之一，则只需创建对应于该流的应用程序注册。</span><span class="sxs-lookup"><span data-stu-id="241bf-106">If your Azure Function only uses one of these flows, you would only need to create the app registrations that correspond to that flow.</span></span>

1. <span data-ttu-id="241bf-107">打开浏览器并导航到 [Azure Active Directory 管理中心](https://aad.portal.azure.com) ，并使用 Microsoft 365 租户组织管理员登录。</span><span class="sxs-lookup"><span data-stu-id="241bf-107">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com) and login using an Microsoft 365 tenant organization admin.</span></span>

1. <span data-ttu-id="241bf-108">选择左侧导航栏中的“ **Azure Active Directory** ”，再选择“ **管理** ”下的“ **应用注册** ”。</span><span class="sxs-lookup"><span data-stu-id="241bf-108">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="241bf-109">应用注册的屏幕截图</span><span class="sxs-lookup"><span data-stu-id="241bf-109">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

## <a name="register-an-app-for-the-single-page-application"></a><span data-ttu-id="241bf-110">为单页面应用程序注册应用程序</span><span class="sxs-lookup"><span data-stu-id="241bf-110">Register an app for the single-page application</span></span>

1. <span data-ttu-id="241bf-111">选择“新注册”。</span><span class="sxs-lookup"><span data-stu-id="241bf-111">Select **New registration**.</span></span> <span data-ttu-id="241bf-112">在“注册应用”页上，按如下方式设置值。</span><span class="sxs-lookup"><span data-stu-id="241bf-112">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="241bf-113">将“名称”设置为“`Graph Azure Function Test App`”。</span><span class="sxs-lookup"><span data-stu-id="241bf-113">Set **Name** to `Graph Azure Function Test App`.</span></span>
    - <span data-ttu-id="241bf-114">仅将 **受支持的帐户类型** 设置为 **此组织目录中的帐户** 。</span><span class="sxs-lookup"><span data-stu-id="241bf-114">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="241bf-115">在 " **重定向 URI** " 下，将 dropdown 更改为 " **单页面应用程序 (SPA)** 并将值设置为 `http://localhost:8080` 。</span><span class="sxs-lookup"><span data-stu-id="241bf-115">Under **Redirect URI** , change the dropdown to **Single-page application (SPA)** and set the value to `http://localhost:8080`.</span></span>

    !["注册应用程序" 页的屏幕截图](./images/register-command-line-app.png)

1. <span data-ttu-id="241bf-117">选择“ **注册** ”。</span><span class="sxs-lookup"><span data-stu-id="241bf-117">Select **Register**.</span></span> <span data-ttu-id="241bf-118">在 " **Graph Azure 函数测试应用** 程序" 页上，将应用程序的值复制 **(客户端) id** 和 **目录 (租户) id** 并保存它们，后续步骤中将需要这些值。</span><span class="sxs-lookup"><span data-stu-id="241bf-118">On the **Graph Azure Function Test App** page, copy the values of the **Application (client) ID** and **Directory (tenant) ID** and save them, you will need them in the later steps.</span></span>

    ![新应用注册的应用程序 ID 的屏幕截图](./images/aad-application-id.png)

## <a name="register-an-app-for-the-azure-function"></a><span data-ttu-id="241bf-120">为 Azure 函数注册应用程序</span><span class="sxs-lookup"><span data-stu-id="241bf-120">Register an app for the Azure Function</span></span>

1. <span data-ttu-id="241bf-121">返回到 " **应用注册** "，然后选择 " **新建注册** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-121">Return to **App Registrations** , and select **New registration**.</span></span> <span data-ttu-id="241bf-122">在“注册应用”页上，按如下方式设置值。</span><span class="sxs-lookup"><span data-stu-id="241bf-122">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="241bf-123">将“名称”设置为“`Graph Azure Function`”。</span><span class="sxs-lookup"><span data-stu-id="241bf-123">Set **Name** to `Graph Azure Function`.</span></span>
    - <span data-ttu-id="241bf-124">仅将 **受支持的帐户类型** 设置为 **此组织目录中的帐户** 。</span><span class="sxs-lookup"><span data-stu-id="241bf-124">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="241bf-125">将 **重定向 URI** 保留为空。</span><span class="sxs-lookup"><span data-stu-id="241bf-125">Leave **Redirect URI** blank.</span></span>

1. <span data-ttu-id="241bf-126">选择“ **注册** ”。</span><span class="sxs-lookup"><span data-stu-id="241bf-126">Select **Register**.</span></span> <span data-ttu-id="241bf-127">在 " **图形 Azure 函数** " 页上，将应用程序的值复制 **(客户端) ID** 并保存它，在下一步中将需要它。</span><span class="sxs-lookup"><span data-stu-id="241bf-127">On the **Graph Azure Function** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

1. <span data-ttu-id="241bf-128">选择“管理”下的“证书和密码”。</span><span class="sxs-lookup"><span data-stu-id="241bf-128">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="241bf-129">选择“新客户端密码”按钮。</span><span class="sxs-lookup"><span data-stu-id="241bf-129">Select the **New client secret** button.</span></span> <span data-ttu-id="241bf-130">在 " **说明** " 中输入一个值，然后选择 " **过期** " 选项之一，然后选择 " **添加** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-130">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

    !["添加客户端密码" 对话框的屏幕截图](./images/aad-new-client-secret.png)

1. <span data-ttu-id="241bf-132">离开此页前，先复制客户端密码值。</span><span class="sxs-lookup"><span data-stu-id="241bf-132">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="241bf-133">将在下一步中用到它。</span><span class="sxs-lookup"><span data-stu-id="241bf-133">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="241bf-134">此客户端密码不会再次显示，所以请务必现在就复制它。</span><span class="sxs-lookup"><span data-stu-id="241bf-134">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![新添加的客户端密码的屏幕截图](./images/aad-copy-client-secret.png)

1. <span data-ttu-id="241bf-136">选择 " **管理** " 下的 " **API 权限** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-136">Select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="241bf-137">选择 " **添加权限** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-137">Choose **Add a permission**.</span></span>

1. <span data-ttu-id="241bf-138">依次选择 " **Microsoft Graph** " 和 " **委派权限** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-138">Select **Microsoft Graph** , then **Delegated Permissions**.</span></span> <span data-ttu-id="241bf-139">添加 **邮件。阅读** 并选择 " **添加权限** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-139">Add **Mail.Read** and select **Add permissions**.</span></span>

    ![Azure Function 应用注册的已配置权限的屏幕截图](./images/web-api-configured-permissions.png)

1. <span data-ttu-id="241bf-141">选择 "在 **管理** 下 **公开 API** "，然后选择 " **添加范围** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-141">Select **Expose an API** under **Manage** , then choose **Add a scope**.</span></span>

1. <span data-ttu-id="241bf-142">接受默认的 **应用程序 ID URI** ，然后选择 " **保存并继续** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-142">Accept the default **Application ID URI** and choose **Save and continue**.</span></span>

1. <span data-ttu-id="241bf-143">填写 " **添加范围** " 窗体，如下所示：</span><span class="sxs-lookup"><span data-stu-id="241bf-143">Fill in the **Add a scope** form as follows:</span></span>

    - <span data-ttu-id="241bf-144">**作用域名称：** Mail. Read</span><span class="sxs-lookup"><span data-stu-id="241bf-144">**Scope name:** Mail.Read</span></span>
    - <span data-ttu-id="241bf-145">**谁可以同意？：** 管理员和用户</span><span class="sxs-lookup"><span data-stu-id="241bf-145">**Who can consent?:** Admins and users</span></span>
    - <span data-ttu-id="241bf-146">**管理员同意显示名称：** 读取所有用户的收件箱</span><span class="sxs-lookup"><span data-stu-id="241bf-146">**Admin consent display name:** Read all users' inboxes</span></span>
    - <span data-ttu-id="241bf-147">**管理员同意说明：** 允许应用读取所有用户的收件箱</span><span class="sxs-lookup"><span data-stu-id="241bf-147">**Admin consent description:** Allows the app to read all users' inboxes</span></span>
    - <span data-ttu-id="241bf-148">**用户同意显示名称：** 阅读您的收件箱</span><span class="sxs-lookup"><span data-stu-id="241bf-148">**User consent display name:** Read your inbox</span></span>
    - <span data-ttu-id="241bf-149">**用户同意说明：** 允许应用读取您的收件箱</span><span class="sxs-lookup"><span data-stu-id="241bf-149">**User consent description:** Allows the app to read your inbox</span></span>
    - <span data-ttu-id="241bf-150">**状态：** 了</span><span class="sxs-lookup"><span data-stu-id="241bf-150">**State:** Enabled</span></span>

1. <span data-ttu-id="241bf-151">选择“添加作用域”。</span><span class="sxs-lookup"><span data-stu-id="241bf-151">Select **Add scope**.</span></span>

1. <span data-ttu-id="241bf-152">复制新作用域，在后续步骤中将需要它。</span><span class="sxs-lookup"><span data-stu-id="241bf-152">Copy the new scope, you'll need it in later steps.</span></span>

    ![Azure Function 应用注册的已定义作用域的屏幕截图](./images/web-api-defined-scopes.png)

1. <span data-ttu-id="241bf-154">选择 " **管理** " 下的 " **清单** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-154">Select **Manifest** under **Manage**.</span></span>

1. <span data-ttu-id="241bf-155">`knownClientApplications`在清单中找到，并将它的当前值替换 `[]` 为 `[TEST_APP_ID]` ，其中 `TEST_APP_ID` 是 **Graph Azure 函数测试应用** 程序注册的应用程序 ID。</span><span class="sxs-lookup"><span data-stu-id="241bf-155">Locate `knownClientApplications` in the manifest, and replace it's current value of `[]` with `[TEST_APP_ID]`, where `TEST_APP_ID` is the application ID of the **Graph Azure Function Test App** app registration.</span></span> <span data-ttu-id="241bf-156">选择“ **保存** ”。</span><span class="sxs-lookup"><span data-stu-id="241bf-156">Select **Save**.</span></span>

> [!NOTE]
> <span data-ttu-id="241bf-157">将测试应用程序的应用程序 ID 添加到 `knownClientApplications` Azure 函数的清单中的属性，可以使测试应用程序触发 [组合同意流](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent)。</span><span class="sxs-lookup"><span data-stu-id="241bf-157">Adding the test application's app ID to the `knownClientApplications` property in the Azure Function's manifest allows the test application to trigger a [combined consent flow](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow#default-and-combined-consent).</span></span> <span data-ttu-id="241bf-158">这对于代表流运行是必需的。</span><span class="sxs-lookup"><span data-stu-id="241bf-158">This is necessary for the on-behalf-of flow to work.</span></span>

## <a name="add-azure-function-scope-to-test-application-registration"></a><span data-ttu-id="241bf-159">添加 Azure 函数作用域以测试应用程序注册</span><span class="sxs-lookup"><span data-stu-id="241bf-159">Add Azure Function scope to test application registration</span></span>

1. <span data-ttu-id="241bf-160">返回到 **Graph Azure 函数测试应用** 注册，然后选择 " **管理** " 下的 " **API 权限** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-160">Return to the **Graph Azure Function Test App** registration, and select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="241bf-161">选择 “ **添加权限** ”。</span><span class="sxs-lookup"><span data-stu-id="241bf-161">Select **Add a permission**.</span></span>

1. <span data-ttu-id="241bf-162">选择 **"我的 api** "，然后选择 " **加载更多** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-162">Select **My APIs** , then select **Load more**.</span></span> <span data-ttu-id="241bf-163">选择 " **Graph Azure 函数** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-163">Select **Graph Azure Function**.</span></span>

    !["请求 API 权限" 对话框的屏幕截图](./images/test-app-add-permissions.png)

1. <span data-ttu-id="241bf-165">选择 " **邮件读取** " 权限，然后选择 " **添加权限** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-165">Select the **Mail.Read** permission, then select **Add permissions**.</span></span>

1. <span data-ttu-id="241bf-166">在 **配置的权限** 中，删除 **用户。** 在 **Microsoft Graph** 下，选择权限右侧的 " **...** "，然后选择 " **删除权限** " 来读取权限。</span><span class="sxs-lookup"><span data-stu-id="241bf-166">In the **Configured permissions** , remove the **User.Read** permission under **Microsoft Graph** by selecting the **...** to the right of the permission and selecting **Remove permission**.</span></span> <span data-ttu-id="241bf-167">选择 **"是，删除** 以确认"。</span><span class="sxs-lookup"><span data-stu-id="241bf-167">Select **Yes, remove** to confirm.</span></span>

    ![测试应用程序应用注册的已配置权限的屏幕截图](./images/test-app-configured-permissions.png)

## <a name="register-an-app-for-the-azure-function-webhook"></a><span data-ttu-id="241bf-169">为 Azure 函数 webhook 注册应用程序</span><span class="sxs-lookup"><span data-stu-id="241bf-169">Register an app for the Azure Function webhook</span></span>

1. <span data-ttu-id="241bf-170">返回到 " **应用注册** "，然后选择 " **新建注册** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-170">Return to **App Registrations** , and select **New registration**.</span></span> <span data-ttu-id="241bf-171">在“注册应用”页上，按如下方式设置值。</span><span class="sxs-lookup"><span data-stu-id="241bf-171">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="241bf-172">将“名称”设置为“`Graph Azure Function Webhook`”。</span><span class="sxs-lookup"><span data-stu-id="241bf-172">Set **Name** to `Graph Azure Function Webhook`.</span></span>
    - <span data-ttu-id="241bf-173">仅将 **受支持的帐户类型** 设置为 **此组织目录中的帐户** 。</span><span class="sxs-lookup"><span data-stu-id="241bf-173">Set **Supported account types** to **Accounts in this organizational directory only**.</span></span>
    - <span data-ttu-id="241bf-174">将 **重定向 URI** 保留为空。</span><span class="sxs-lookup"><span data-stu-id="241bf-174">Leave **Redirect URI** blank.</span></span>

1. <span data-ttu-id="241bf-175">选择“ **注册** ”。</span><span class="sxs-lookup"><span data-stu-id="241bf-175">Select **Register**.</span></span> <span data-ttu-id="241bf-176">在 " **图形 Azure 函数 webhook** " 页面上，将应用程序的值复制 **(客户端) ID** 并保存它，在下一步中将需要它。</span><span class="sxs-lookup"><span data-stu-id="241bf-176">On the **Graph Azure Function webhook** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

1. <span data-ttu-id="241bf-177">选择“管理”下的“证书和密码”。</span><span class="sxs-lookup"><span data-stu-id="241bf-177">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="241bf-178">选择“新客户端密码”按钮。</span><span class="sxs-lookup"><span data-stu-id="241bf-178">Select the **New client secret** button.</span></span> <span data-ttu-id="241bf-179">在 " **说明** " 中输入一个值，然后选择 " **过期** " 选项之一，然后选择 " **添加** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-179">Enter a value in **Description** and select one of the options for **Expires** and select **Add**.</span></span>

1. <span data-ttu-id="241bf-180">离开此页前，先复制客户端密码值。</span><span class="sxs-lookup"><span data-stu-id="241bf-180">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="241bf-181">将在下一步中用到它。</span><span class="sxs-lookup"><span data-stu-id="241bf-181">You will need it in the next step.</span></span>

1. <span data-ttu-id="241bf-182">选择 " **管理** " 下的 " **API 权限** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-182">Select **API Permissions** under **Manage**.</span></span> <span data-ttu-id="241bf-183">选择 " **添加权限** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-183">Choose **Add a permission**.</span></span>

1. <span data-ttu-id="241bf-184">依次选择 " **Microsoft Graph** " 和 " **应用程序权限** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-184">Select **Microsoft Graph** , then **Application Permissions**.</span></span> <span data-ttu-id="241bf-185">添加 **User. read. All** 和 **Mail. read** ，然后选择 " **添加权限** "。</span><span class="sxs-lookup"><span data-stu-id="241bf-185">Add **User.Read.All** and **Mail.Read** , then select **Add permissions**.</span></span>

1. <span data-ttu-id="241bf-186">在 **配置的权限** 中，删除委派的 **用户。** 在 **Microsoft Graph** 下，选择权限右侧的 " **...** "，然后选择 " **删除权限** " 来读取权限。</span><span class="sxs-lookup"><span data-stu-id="241bf-186">In the **Configured permissions** , remove the delegated **User.Read** permission under **Microsoft Graph** by selecting the **...** to the right of the permission and selecting **Remove permission**.</span></span> <span data-ttu-id="241bf-187">选择 **"是，删除** 以确认"。</span><span class="sxs-lookup"><span data-stu-id="241bf-187">Select **Yes, remove** to confirm.</span></span>

1. <span data-ttu-id="241bf-188">选择 " **授予管理员同意 ...** " 按钮，然后选择 **"是"** 授予管理员同意配置的应用程序权限。</span><span class="sxs-lookup"><span data-stu-id="241bf-188">Select the **Grant admin consent for...** button, then select **Yes** to grant admin consent for the configured application permissions.</span></span> <span data-ttu-id="241bf-189">" **已配置权限** " 表中的 " **状态** " 列更改为 "已 **授予"。**</span><span class="sxs-lookup"><span data-stu-id="241bf-189">The **Status** column in the **Configured permissions** table changes to **Granted for ...**.</span></span>

    ![授予了管理员同意的 webhook 的已配置权限的屏幕截图](./images/webhook-configured-permissions.png)
