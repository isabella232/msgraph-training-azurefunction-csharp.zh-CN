---
ms.openlocfilehash: fbd43b4708cd1aeaad93b70fcf151869bd9e543e
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822577"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="da4f8-101">在本练习中，你将了解示例 Azure 函数为准备 [发布到 Azure 函数应用程序](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish)所需的更改。</span><span class="sxs-lookup"><span data-stu-id="da4f8-101">In this exercise you'll learn about what changes are needed to the sample Azure Function to prepare for [publishing to an Azure Functions app](https://docs.microsoft.com/azure/azure-functions/functions-run-local#publish).</span></span>

## <a name="update-code"></a><span data-ttu-id="da4f8-102">更新代码</span><span class="sxs-lookup"><span data-stu-id="da4f8-102">Update code</span></span>

<span data-ttu-id="da4f8-103">将从用户密钥存储中读取配置，这仅适用于开发计算机。</span><span class="sxs-lookup"><span data-stu-id="da4f8-103">Configuration is read from the user secret store, which only applies to your development machine.</span></span> <span data-ttu-id="da4f8-104">在发布到 Azure 之前，你需要更改存储配置的位置，并相应地更新 **Startup.cs** 中的代码。</span><span class="sxs-lookup"><span data-stu-id="da4f8-104">Before you publish to Azure, you'll need to change where you store your configuration, and update the code in **Startup.cs** accordingly.</span></span>

<span data-ttu-id="da4f8-105">应用程序密码应存储在安全存储中，如 [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview)。</span><span class="sxs-lookup"><span data-stu-id="da4f8-105">Application secrets should be stored in secure storage, such as [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview).</span></span>

## <a name="update-cors-setting-for-azure-function"></a><span data-ttu-id="da4f8-106">更新 Azure 函数的 CORS 设置</span><span class="sxs-lookup"><span data-stu-id="da4f8-106">Update CORS setting for Azure Function</span></span>

<span data-ttu-id="da4f8-107">在此示例中，我们将 "CORS" 配置 **local.settings.js"启用** "，以允许测试应用程序调用函数。</span><span class="sxs-lookup"><span data-stu-id="da4f8-107">In this sample we configured CORS in **local.settings.json** to allow the test application to call the function.</span></span> <span data-ttu-id="da4f8-108">您需要将已发布的函数配置为允许任何将调用它的 SPA 应用程序。</span><span class="sxs-lookup"><span data-stu-id="da4f8-108">You'll need to configure your published function to allow any SPA apps that will call it.</span></span>

## <a name="update-app-registrations"></a><span data-ttu-id="da4f8-109">更新应用注册</span><span class="sxs-lookup"><span data-stu-id="da4f8-109">Update app registrations</span></span>

<span data-ttu-id="da4f8-110">将  `knownClientApplications` 需要使用将调用 Azure 函数的任何应用程序的应用程序 id 更新用于 **图 Azure 函数** 应用注册的清单中的属性。</span><span class="sxs-lookup"><span data-stu-id="da4f8-110">The  `knownClientApplications` property in the manifest for the **Graph Azure Function** app registration will need to be updated with the application IDs of any apps that will be calling the Azure Function.</span></span>

## <a name="recreate-existing-subscriptions"></a><span data-ttu-id="da4f8-111">重新创建现有订阅</span><span class="sxs-lookup"><span data-stu-id="da4f8-111">Recreate existing subscriptions</span></span>

<span data-ttu-id="da4f8-112">使用您的本地计算机或 ngrok 上的 webhook URL 创建的任何订阅都应使用 Azure 函数的生产 URL 重新创建 `Notify` 。</span><span class="sxs-lookup"><span data-stu-id="da4f8-112">Any subscriptions created using the webhook URL on your local machine or ngrok should be recreated using the production URL of the `Notify` Azure Function.</span></span>
