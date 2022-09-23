---
lab:
  title: 将 Azure Cosmos DB SQL API 帐户密钥存储到 Azure 密钥保管库
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB SQL API solution
ms.openlocfilehash: 56ecce7cf2b6460419f54813af06c0e0e01738b5
ms.sourcegitcommit: b6d75bce14482279e6b4b3c8eb9d792a07516916
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/26/2022
ms.locfileid: "145893422"
---
# <a name="store-azure-cosmos-db-sql-api-account-keys-in-azure-key-vault"></a>将 Azure Cosmos DB SQL API 帐户密钥存储到 Azure 密钥保管库

将 Azure Cosmos DB 帐户连接代码添加到应用程序就像提供帐户的 URI 和密钥一样简单。 有时，此安全信息可能会硬编码到应用程序代码中。 但是，如果将应用程序部署到 Azure 应用服务，则可以将加密连接信息保存到 Azure 密钥保管库。

在此实验室中，我们将加密 Azure Cosmos DB 帐户连接字符串，并将其存储到 Azure 密钥保管库。 然后，我们将创建一个 Azure 应用服务 Web 应用，该应用将从 Azure 密钥保管库中检索这些凭据。 应用程序将使用这些凭据并连接到 Azure Cosmos DB 帐户。 然后，应用程序将在 Azure Cosmos DB 帐户容器中创建一些文档，并将其状态返回到网页。

## <a name="prepare-your-development-environment"></a>准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果尚不熟悉 Visual Studio Code 界面，请查看 [Visual Studio Code 入门指南][code.visualstudio.com/docs/getstarted]

1. 打开命令面板并运行 Git: Clone 以将 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 存储库克隆到你选择的本地文件夹中。

    > &#128161; 你可以使用 Ctrl+Shift+P 键盘快捷方式打开命令面板。

1. 克隆存储库后，关闭 Visual Studio Code__*。 稍后，直接指向 28-key-vault，以打开该文件夹。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>创建 Azure Cosmos DB SQL API 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 在首次预配 Azure Cosmos DB 帐户时，可以选择希望该帐户支持的 API（例如 Mongo API 或 SQL API） 。 Azure Cosmos DB SQL API 帐户完成预配后，你可以检索终结点和密钥。 使用终结点和密钥以编程方式连接到 Azure Cosmos DB SQL API 帐户。 在 Azure SDK for .NET 或任何其他 SDK 的连接字符串上使用终结点和密钥。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB SQL API”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **帐户名** | 输入一个全局唯一的名称 |
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *预配的吞吐量* |
    | **应用免费分级折扣** | *不应用* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用预先创建的现有资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“主链接字符串”字段的值。 你将在此练习的稍后部分使用此连接字符串值。

## <a name="create-an-azure-key-vault-and-store-the-azure-cosmos-db-account-credentials-as-a-secret"></a>创建 Azure 密钥保管库并将 Azure Cosmos DB 帐户凭据存储为机密

在创建 Web 应用之前，我们会通过将 Azure Cosmos DB 帐户连接字符串复制到 Azure 密钥保管库加密机密来保护它们 。 现在就让我们执行此操作。

1. 在 Azure 门户，导航到“密钥保管库”页面。

1. 选择“+ 创建”按钮添加保管库，并使用以下设置填写保管库，将所有其余设置保留为其默认值，然后选择创建保管库*__：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **密钥保管库名称** | 输入全局唯一名称 |
    | **区域** | 选择任何可用区域 |

1. 创建保管库后，导航到保管库。

1. 在“设置”部分下，选择“机密”。

1. 选择“+ 生成/导入”加密我们的凭据连接字符串并使用以下设置填写机密值，将所有其余设置保留为默认值，然后选择创建机密 ：

    | **设置** | **值** |
    | ---: | :--- |
    | **上传选项** | *手动* |
    | **名称** | 用于标记机密的名称 |
    | **值** | 此字段是最重要的字段，必须填写。此值是你之前从 Azure Cosmos DB 帐户的密钥部分复制的主连接字符串。此值将转换为机密。 |
    | **已启用** | *是* |
 
1. 在“机密”下，现在应该会看到新机密已列出。 我们需要获取将添加到 Web 应用代码中的机密标识符。 选择你创建的机密。

1. 在 Azure 密钥保管库中，你可以创建多个版本的机密，但就此实验室而言，我们只需要一个版本。 选择“当前版本”。

1. 记录“机密标识符”字段的值。 我们将在应用程序代码中使用此值从密钥保管库获取机密。  请注意，此值是 URL。 要让此机密起作用，我们还需要执行一个步骤，我们将在稍后执行此步骤。

## <a name="create-an-azure-app-service-webapp"></a>创建 Azure 应用服务 Web 应用

我们将创建一个 Web 应用，用于连接到 Azure Cosmos DB 帐户，并创建一些容器和文档。 我们不会在此应用中对 Azure Cosmos DB 凭据进行硬编码，而是对密钥保管库中的机密标识符进行硬编码。 我们将看到，如果没有向 Azure 层上的 Web 应用分配适当的权限，此标识符将无用。 现在，我们开始编码。



1. 打开 **Visual Studio Code**。  选择“文件”->“打开文件夹”，一直浏览到 28-key-vault 文件夹，以打开 28-key-vault 文件夹 。

    > &#128221; 请注意，你应该只能在资源管理器树中看到 28-key-vault 文件夹及其文件和子文件夹 。 如果可以看到我们之前克隆的整个 GitHub 存储库，则 Web 应用将无法正常运行，因此，请确保只能在资源管理器树中看到 28-key-vault 文件夹及其文件和子文件夹 。

1. 打开 28-key-vault 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例 。

    > &#128221; 此命令将打开起始目录已设置为 28-key-vault 文件夹的终端。

1. 现在，我们来创建和 MVC Web 应用外壳。 稍后我们将替换几个生成的文件。 运行以下命令来创建 Web 应用：

    ```
    dotnet new mvc
    ```


1. 该命令创建了 Web 应用的外壳，因此它添加了几个文件和目录。 我们已经有几个文件，其中包含我们需要的所有代码。 将文件 .\Controllers\HomeController.cs 和 .\Views\Home\Index.cshtml 替换为 .\KeyvaultFiles 目录中各自的文件  。

1. 替换文件后，请删除 .\KeyvaultFiles 目录。

## <a name="import-the-multiple-missing-libraries-into-the-net-script"></a>将多个缺少的库导入 .NET 脚本

.NET CLI 包含 [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] 命令，用于从预配置的包源导入包。 .NET 安装使用 NuGet 作为默认包源。

1. 如果尚未这样做，请在 Visual Studio Code 的“资源管理器”窗格中，浏览到 28-key-vault 文件夹  。

1. 如果尚未这样做，请打开 28-key-vault 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例 。

    > &#128221; 此命令将打开起始目录已设置为 28-key-vault 文件夹的终端。

1. 使用以下命令从 NuGet 添加 [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 包：

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 使用以下命令从 NuGet 添加 [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] 包：

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

1. 使用以下命令从 NuGet 添加 [Microsoft.Azure.KeyVault][nuget.org/packages/Microsoft.Azure.KeyVault] 包：

    ```
    dotnet add package Microsoft.Azure.KeyVault
    ```

1. 使用以下命令从 NuGet 添加 [Microsoft.Azure.Services.AppAuthentication][nuget.org/packages/Microsoft.Azure.Services.AppAuthentication] 包：

    ```
    dotnet add package Microsoft.Azure.Services.AppAuthentication
    ```

## <a name="adding-the-secret-identifier-to-your-webapp"></a>将机密标识符添加到 Web 应用

1. 在 Visual Studio 中，打开 `.\Controllers\HomeControler.cs` 文件

1. GetKeyVaultSecret 用户定义函数将获取 Azure Cosmos DB 帐户密码。 函数从第 98 行开始，应该类似于下面的脚本。

```
        private static async Task<Tuple<bool,string>>  GetKeyVaultSecret()
        {
            AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider("RunAs=App;");

            try
            {
                var KVClient = new KeyVaultClient(
                    new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));

                var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
                    .ConfigureAwait(false);

                return new Tuple<bool,string>(true, KeyVaultSecret.Value.ToString());

            }
            catch (Exception exp)
            {
                return new Tuple<bool,string>(false, exp.Message);
            }

        }
```

3. 让我们了解一下此函数进行的重要调用。

    - 在第 100 行，我们定义了当前 Web 应用的令牌。 此令牌将提供给 Azure 密钥保管库，以标识正在尝试访问该保管库的应用。 
    - 在第 104-105 行中，我们准备了将连接到 Azure 密钥保管库的密钥保管库客户端 。 请注意，我们将 Web 应用令牌作为参数发送。 
    - 在第 107-108 行中，我们向密钥保管库客户端提供了我们的机密标识符的 URL 地址，该地址将返回存储在该密钥保管库中的机密。 

1.  在部署 Web 应用之前，我们仍然需要发送机密标识符 URL。  在第 107 行，将字符串 <Key Vault Secret Identifier> 替换为我们在“机密”部分记录的“机密标识符”URL 并保存文件 。

```
        var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
```

## <a name="optional-install-the-azure-app-services-extension"></a>（可选）安装 Azure 应用服务扩展

在 Visual Studio 中，如果调出命令面板 (Ctrl+Shift+P)，并且在搜索 Azure 应用资源命令时它没有返回任何内容，则需要安装扩展。

1. 在 Visual Studio Code 左侧菜单中，选择“扩展”选项。

1. 在搜索栏中，搜索 Azure 应用服务并选择它。

1. 选择“安装”按钮进行安装。

1. 关闭“扩展”选项卡并返回到代码。

## <a name="deploy-your-application-to-azure-app-services"></a>将应用程序部署到 Azure 应用服务

其余代码很简单，获取连接字符串，连接到 Azure Cosmos DB，并添加一些文档。 应用程序还应就任何问题向我们提供一些反馈。 在部署应用程序后，我们不应再执行任何其他更改。 让我们开始吧。 

> &#128221; 下面的大部分步骤将在 Visual Studio 屏幕中上部的命令面板中运行。

1. 在 Visual Studio Code 中，打开命令面板，搜索“Azure 应用服务: 创建新的 Web 应用…(高级)”

1. 选择“登录 Azure…”。此选项将打开一个 Web 浏览器窗口，按照登录流程进行操作，完成后关闭浏览器。

1. （可选）如果要求提供订阅，请选择你的订阅。

1. 为 Web 应用输入全局唯一名称。

1. 选择现有资源组，或创建一个新组（如果需要）。

1. 选择 .NET 6 (LTS)。

1. 选择“Windows”。

1. 选择可用位置。

1. 选择“+ 创建新的应用服务计划”。

1. 接受应用服务计划的默认名称（它应该与你的 Web 应用名称相同），或选择一个新名称。

1. 选择“免费(F1)免费试用 Azure”。

1. 为 Application Insights 选择“暂时跳过”。

1. 部署现在应该运行，右下角有一个状态栏。 

1. 出现提示时，选择“部署”。

1. 选择“浏览”，你应该位于 28-key-vault 文件夹中，选择该文件夹 。

1. 此时出现一个弹出窗口，其中显示“28-key-vault 中缺少进行部署所需的配置”，选择“添加配置”按钮。  此选项将创建缺少的 `.vscode` 文件夹。

    > &#128221; 非常重要的是，如果首次部署应用时未出现此弹出窗口，则上传到 Azure 应用服务将丢失文件。 部署成功，但网站总是返回消息“你没有权限查看此目录或页面”。 最可能的原因是，Visual Studio Code 是在 GitHub 克隆存储库上打开的，而不仅仅是 28-key-vault 文件夹。

1. 当提示是否始终部署到该工作区时，选择“是”。

1. 系统提示时，选择“浏览网站”。  或者，打开浏览器并转到 `https://<yourwebappname>.azurewebsites.net`。 在这两种情况下，我们都有一个问题。 我们应该在网页上得到用户定义的消息。 消息应该是“密钥保管库无法访问”，并带有扩展的错误消息。 现在来解决此问题。

## <a name="allow-our-app-to-use-a-managed-identity"></a>允许应用使用托管标识

我们需要解决的第一个问题是允许应用使用托管标识。 使用托管标识将允许应用使用 Azure 服务，例如 Azure 密钥保管库。

1. 打开浏览器并登录 Azure 门户。

1. 打开“应用服务”页面。 你的 Web 应用名称应该列出，请选择它。

1. 在“设置”部分下，选择“标识”。

1. 在“状态”下，选择“打开”和“保存” 。  如果提示是否启用分配的托管标识，请选择“是”。

1. 现在，再次尝试我们的 Web 应用。  在浏览器中转到 `https://<yourwebappname>.azurewebsites.net`。

1. 仍有一个问题。 虽然第一条消息是程序正在发送的用户定义消息，但第二条消息是系统生成的消息。 第二条消息的意思是，我们已被授予对密钥保管库的连接访问权限，但尚未授予我们查看保管库中机密的权限。  现在，我们来进行最后一个设置，以解决此问题。

## <a name="granting-our-web-application-an-access-policy-to-the-key-vault-secrets"></a>向 Web 应用程序授予对密钥保管库机密的访问策略

此实验室的最初目标是防止我们的 Azure Cosmos DB 帐户在应用程序中被硬编码。 但是，我们确实对任何人都可以看到的机密标识符 URL 进行了硬编码。 那么，我们如何保护凭据呢？ 好消息是机密标识符本身没有用处。 机密标识符只能将你带到 Azure 密钥保管库门口，但允许谁进入保管库则由保管库自己决定。 这意味着，我们需要为应用程序创建密钥保管库访问策略，以便它可以看到该保管库中的机密。 我们来看看该解决方案。

1. （可选）在创建策略之前，我们先查看 Azure Cosmos DB 数据库的当前内容。  在 Azure 门户中，转到你 Azure Cosmos DB 帐户，看看帐户下是否有 GlobalCustomers 数据库？ 如果没有，成功运行 Web 应用后会创建该数据库。 如果有，请查看数据库中的项数，成功运行 Web 应用后会添加更多项。

1. 在 Azure 门户中，转到我们之前创建的密钥保管库。

1. 在“设置”部分下，选择“访问策略”。

1. 选择“+ 添加访问策略”。

1. 使用以下设置填写“访问策略”值，将所有其余设置保留为默认值，然后选择添加策略 ：

    | **设置** | **值** |
    | ---: | :--- |
    | 密钥权限 | *Get* |
    | **机密权限** | *Get* |
    | **选择主体** | 搜索并选择你的应用程序名称 |

    > &#128221; 不要选择授权的应用程序。

1. 保存新策略。

1. 现在，再次尝试我们的 Web 应用。  在浏览器中转到 `https://<yourwebappname>.azurewebsites.net`。

1. 成功！ 我们的网页应该表明已将新项目插入到了客户容器中。 我们还可以看到正在显示实际的机密。

    > &#128221; 在生产环境中，不会显示机密，这里只是为了说明。


1. 转到 Azure Cosmos DB 帐户，并验证是否有一个包含数据的新 GlobalCustomers 数据库，或者该数据库中的项是否增加（如果该数据库已经存在）。

我们现在已成功使用 Azure 密钥保管库来保护你的 Azure Cosmos DB 帐户的密钥。
