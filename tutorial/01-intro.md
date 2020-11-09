---
ms.openlocfilehash: 1e4267a5d8bdfe02dbc0170122113c35df822659
ms.sourcegitcommit: 57a5c2e1a562d8af092a3e78786d711ce1e8f9cb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "48822570"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="7d579-101">本教程向您介绍如何构建使用 Microsoft Graph API 检索用户的日历信息的 Azure 函数。</span><span class="sxs-lookup"><span data-stu-id="7d579-101">This tutorial teaches you how to build an Azure Function that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="7d579-102">如果您只想下载已完成的教程，可以下载或克隆 [GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)。</span><span class="sxs-lookup"><span data-stu-id="7d579-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span> <span data-ttu-id="7d579-103">有关使用应用 ID 和密码配置应用程序的说明，请参阅 **demo** 文件夹中的自述文件。</span><span class="sxs-lookup"><span data-stu-id="7d579-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="7d579-104">必备条件</span><span class="sxs-lookup"><span data-stu-id="7d579-104">Prerequisites</span></span>

<span data-ttu-id="7d579-105">在开始本教程之前，您应在开发计算机上安装以下工具。</span><span class="sxs-lookup"><span data-stu-id="7d579-105">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="7d579-106">.NET Core SDK</span><span class="sxs-lookup"><span data-stu-id="7d579-106">.NET Core SDK</span></span>](https://dotnet.microsoft.com/download)
- [<span data-ttu-id="7d579-107">Azure 函数核心工具</span><span class="sxs-lookup"><span data-stu-id="7d579-107">Azure Functions Core Tools</span></span>](https://docs.microsoft.com/azure/azure-functions/functions-run-local)
- [<span data-ttu-id="7d579-108">Azure CLI</span><span class="sxs-lookup"><span data-stu-id="7d579-108">Azure CLI</span></span>](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [<span data-ttu-id="7d579-109">ngrok</span><span class="sxs-lookup"><span data-stu-id="7d579-109">ngrok</span></span>](https://ngrok.com/)

<span data-ttu-id="7d579-110">此外，还应具有 Microsoft 工作或学校帐户，并访问同一组织中的全局管理员帐户。</span><span class="sxs-lookup"><span data-stu-id="7d579-110">You should also have a Microsoft work or school account, with access to a global administrator account in the same organization.</span></span> <span data-ttu-id="7d579-111">如果你没有 Microsoft 帐户，可以 [注册 office 365 开发人员计划](https://developer.microsoft.com/office/dev-program) 以获取免费的 office 365 订阅。</span><span class="sxs-lookup"><span data-stu-id="7d579-111">If you don't have a Microsoft account, you can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="7d579-112">本教程是使用上述工具的以下版本编写的。</span><span class="sxs-lookup"><span data-stu-id="7d579-112">This tutorial was written with the following versions of the above tools.</span></span> <span data-ttu-id="7d579-113">本指南中的步骤可能适用于其他版本，但尚未经过测试。</span><span class="sxs-lookup"><span data-stu-id="7d579-113">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="7d579-114">.NET Core SDK 3.1.301</span><span class="sxs-lookup"><span data-stu-id="7d579-114">.NET Core SDK 3.1.301</span></span>
> - <span data-ttu-id="7d579-115">Azure 函数 Core Tools 3.0.2630</span><span class="sxs-lookup"><span data-stu-id="7d579-115">Azure Functions Core Tools 3.0.2630</span></span>
> - <span data-ttu-id="7d579-116">Azure CLI 2.8。0</span><span class="sxs-lookup"><span data-stu-id="7d579-116">Azure CLI 2.8.0</span></span>
> - <span data-ttu-id="7d579-117">ngrok 2.3.35</span><span class="sxs-lookup"><span data-stu-id="7d579-117">ngrok 2.3.35</span></span>

## <a name="feedback"></a><span data-ttu-id="7d579-118">反馈</span><span class="sxs-lookup"><span data-stu-id="7d579-118">Feedback</span></span>

<span data-ttu-id="7d579-119">请在 [GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp)中提供有关本教程的任何反馈。</span><span class="sxs-lookup"><span data-stu-id="7d579-119">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-azurefunction-csharp).</span></span>
