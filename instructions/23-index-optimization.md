---
lab:
  title: 优化 Azure Cosmos DB SQL API 容器的写入操作索引策略
  module: Module 10 - Optimize query performance in Azure Cosmos DB SQL API
ms.openlocfilehash: d83af5c3783a7502a2f2e220b3b2ae566c0338fe
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2022
ms.locfileid: "138024940"
---
# <a name="optimize-an-azure-cosmos-db-sql-api-containers-indexing-policy-for-write-operations"></a>优化 Azure Cosmos DB SQL API 容器的写入操作索引策略

对于写入量很大的工作负载或具有大型 JSON 对象的工作负载，优化索引策略以只索引你知道要用于查询的属性可能更有利。

在本实验室中，我们将使用测试 .NET 应用程序，使用默认索引策略将大型 JSON 项插入到 Azure Cosmos DB SQL API 容器中，然后使用经过微调的索引策略。

## <a name="prepare-your-development-environment"></a>准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅[入门文档][code.visualstudio.com/docs/getstarted]

1. 打开命令面板并运行 Git: Clone，将 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 存储库克隆到你选择的本地文件夹中。

    > &#128161; 可以使用 CTRL+SHIFT+P 键盘快捷方式打开命令面板。

1. 克隆存储库后，打开在 Visual Studio Code 中选择的本地文件夹。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>创建 Azure Cosmos DB SQL API 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 在首次预配 Azure Cosmos DB 帐户时，可以选择希望该帐户支持的 API（例如 Mongo API 或 SQL API）。 完成 Azure Cosmos DB SQL API 帐户预配后，可以检索终结点和密钥，并使用它们通过 Azure SDK for .NET 或所选择的任何其他 SDK 连接到 Azure Cosmos DB SQL API 帐户。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB SQL API”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **帐户名** | 输入一个全局唯一的名称 |
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *无服务器* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用预先创建的现有资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”窗格中，选择“新建容器” 。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定”： 

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 新建 &vert; cosmicworks |
    | **容器 ID** | products |
    | **分区键** | /categoryId |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“products”容器节点。  

1. 在资源边栏选项卡中，导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="run-the-test-net-application-using-the-default-indexing-policy"></a>使用默认索引策略运行测试 .NET 应用程序

此实验室有一个预生成的测试 .NET 应用程序，该应用程序将采用大型 JSON 对象，并在 Azure Cosmos DB SQL API 容器中创建新项。 单次写入操作完成后，应用程序将项的唯一标识符和 RU 费用输出到控制台窗口。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 23-index-optimization 文件夹。

1. 打开 23-index-optimization 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为 23-index-optimization 文件夹的终端。

1. 使用 [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 命令生成项目：

    ```
    dotnet build
    ```

    > &#128221; 你可能会看到编译器警告，指出当前未使用 endpoint 和 key 变量。 可以安全地忽略此警告，因为将在此任务中使用这些变量。

1. 关闭集成终端。

1. 打开 script.cs 代码文件。

1. 找到名为 endpoint 的 string 变量。 将它的值设置为之前创建的 Azure Cosmos DB 帐户的终结点。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，则 C# 语句将为：string endpoint = "https&shy;://dp420.documents.azure.com:443/";。

1. 找到名为 key 的 string 变量。 将它的值设置为之前创建的 Azure Cosmos DB 帐户的键。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; 例如，如果键为：fDR2ci9QgkdkvERTQ==，则 C# 语句应为：string key = "fDR2ci9QgkdkvERTQ==";。

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 23-index-optimization 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 查看终端输出。 项的唯一标识符和操作的请求费用（按 RU 计）应输出到控制台。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令至少再生成并运行项目两次。 观察控制台输出中的 RU 费用：

    ```
    dotnet run
    ```

1. 使集成终端保持打开状态。

    > &#128221; 你将在此练习的后面部分再次使用此终端。 务必要使终端保持打开状态，以便可以比较原始 RU 费用和更新的 RU 费用。

## <a name="update-the-indexing-policy-and-rerun-the-net-application"></a>更新索引策略并重新运行 .NET 应用程序

本实验室方案假定我们未来的查询主要关注 name 和 categoryName 属性。 若要针对大型 JSON 项进行优化，请通过创建索引策略（从排除所有路径开始）从索引中排除所有其他字段。 然后，策略将选择性地包含特定路径。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，选择先前在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“设置”。

1. 在“设置”选项卡中，导航到“索引策略”部分。

1. 观察默认索引策略：

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }    
    ```

1. 将索引策略替换为修改后的 JSON 对象，然后保存更改：

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        },
        {
          "path": "/categoryName/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        },
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

1. 关闭 Web 浏览器窗口或选项卡。

1. 返回到 Visual Studio Code。 返回到打开的终端。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令至少再生成并运行项目两次。 观察控制台输出中的新 RU 费用，该费用应显著小于原始费用：

    ```
    dotnet run
    ```

    > &#128221; 如果没有看到更新的 RU 费用，可能需要等待几分钟。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，选择先前在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“设置”。

1. 在“设置”选项卡中，导航到“索引策略”部分。

1. 将索引策略替换为修改后的 JSON 对象，然后保存更改：

    ```
    {
      "indexingMode": "none"
    }
    ```

1. 关闭 Web 浏览器窗口或选项卡。

1. 返回到 Visual Studio Code。 返回到打开的终端。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令至少再生成并运行项目两次。 观察控制台输出中的新 RU 费用，该费用应略微小于原始费用：

    ```
    dotnet run
    ```

    > &#128221; 如果没有看到更新的 RU 费用，可能需要等待几分钟。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
