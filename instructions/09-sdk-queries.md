---
lab:
  title: 使用 Azure Cosmos DB for NoSQL SDK 执行查询
  module: Module 5 - Execute queries in Azure Cosmos DB for NoSQL
---

# <a name="execute-a-query-with-the-azure-cosmos-db-for-nosql-sdk"></a>使用 Azure Cosmos DB for NoSQL SDK 执行查询

利用最新版本的用于 Azure Cosmos DB for NoSQL 的 .NET SDK 可以更容易地查询容器，并使用 C# 中的最新最佳做法和语言功能异步遍历结果集。

> &#128161; 此实验室使用 NuGet 上的 4.0.0-preview3 版本的 [Azure.Cosmos][nuget.org/packages/azure.cosmos/4.0.0-preview3] 库。 此库具有特殊功能，让你可以更轻松地使用[异步流][docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams]查询 Azure Cosmos DB。

在此实验室中，你将使用异步流来遍历从 Azure Cosmos DB for NoSQL 返回的大型结果集。 你将使用 .NET SDK 查询和遍历结果。

## <a name="prepare-your-development-environment"></a>准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅 [Visual Studio Code 入门指南][code.visualstudio.com/docs/getstarted]

1. 打开命令面板并运行 Git: Clone，将 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 存储库克隆到你选择的本地文件夹中。

    > &#128161; 你可以使用 Ctrl+Shift+P 键盘快捷方式打开命令面板。

1. 克隆存储库后，打开在 Visual Studio Code 中选择的本地文件夹。

## <a name="create-an-azure-cosmos-db-for-nosql-account"></a>创建 Azure Cosmos DB for NoSQL 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 在首次预配 Azure Cosmos DB 帐户时，可以选择希望该帐户支持的 API（例如 Mongo API 或 NoSQL API）。 完成 Azure Cosmos DB for NoSQL 帐户预配后，可以检索终结点和密钥，并使用它们通过 Azure SDK for .NET 或所选择的任何其他 SDK 连接到 Azure Cosmos DB for NoSQL 帐户。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭证登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB for NoSQL”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **帐户名** | 输入全局唯一名称 |
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *预配的吞吐量* |
    | **应用免费分级折扣** | *不应用* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="seed-the-azure-cosmos-db-for-nosql-account-with-data"></a>使用数据为 Azure Cosmos DB for NoSQL 帐户设定种子

[cosmicworks][nuget.org/packages/cosmicworks] 命令行工具将示例数据部署到任何 Azure Cosmos DB for NoSQL 帐户。 该工具是开源工具，可通过 NuGet 获得。 将此工具安装到 Azure Cloud Shell，然后使用它来设定数据库种子。

1. 在 Visual Studio Code 中，打开“终端”菜单，然后选择“新建终端”以打开新的终端实例  。

1. 在计算机上安装可全局使用的 [cosmicworks][nuget.org/packages/cosmicworks] 命令行工具。

    ```
    dotnet tool install --global cosmicworks
    ```

    > &#128161; 此命令可能需要几分钟时间才能完成。 如果你过去已经安装了此工具的最新版本，此命令将输出警告消息（*工具 "cosmicworks" 已安装）。

1. 使用以下命令行选项运行 cosmicworks 以设置 Azure Cosmos DB 种子帐户：

    | **选项** | **值** |
    | ---: | :--- |
    | **--endpoint** | 之前在本实验室中复制的终结点值 |
    | **--key** | 之前在本实验室中复制的键值 |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，密钥为：fDR2ci9QgkdkvERTQ==，则命令为：``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. 等待 cosmicworks 命令完成对帐户的数据库、容器和项的填充。

1. 关闭集成终端。

## <a name="iterate-over-the-results-of-a-sql-query-using-the-sdk"></a>使用 SDK 来遍历 SQL 查询的结果

现在你将使用一个异步流在 Azure Cosmos DB 的分页结果上创建一个简单易懂的 foreach 循环。 在后台，SDK 将管理源迭代器，确保正确调用后续请求。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 09-execute-query-sdk 文件夹  。

1. 打开 product.cs 代码文件。

1. 观察 Product 类及其对应的属性。 具体而言，此实验室将使用 id、name 和 price 属性。

1. 返回到 Visual Studio Code 的“资源管理器”窗格，打开 script.cs 代码文件。  

1. 更新名为 endpoint 的现有变量，将其值设置为前面创建的 Azure Cosmos DB 帐户的终结点 。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，则 C# 语句将为：string endpoint = "https&shy;://dp420.documents.azure.com:443/";。

1. 更新名为 key 的现有变量，将其值设置为前面创建的 Azure Cosmos DB 帐户的键 。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; 例如，如果键为：fDR2ci9QgkdkvERTQ==，则 C# 语句应为：string key = "fDR2ci9QgkdkvERTQ=="; 。

1. 使用 SELECT * FROM products p 的值创建一个名为 sql 的 string 类型的新变量：

    ```
    string sql = "SELECT * FROM products p";
    ```

1. 创建一个 [QueryDefinition][docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition] 类型的新变量，并传入 sql 变量作为构造函数的参数：

    ```
    QueryDefinition query = new (sql);
    ```

1. 通过调用 [CosmosContainer][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer] 类的泛型 [GetItemQueryIterator][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator] 方法创建一个新的 await foreach 循环，并传入 query 变量作为参数，然后通过使用 product 变量代表 Product 类型的实例来异步遍历结果   ：

    ```
    await foreach (Product product in container.GetItemQueryIterator<Product>(query))
    {
    }
    ```

1. 在 await foreach 循环中，使用内置的 Console.WriteLine 静态方法格式化并输出 product 变量的 id、name 和 price 属性     ：

    ```
    Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    CosmosDatabase database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    CosmosContainer container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    string sql = "SELECT * FROM products p";
    QueryDefinition query = new (sql);

    await foreach (Product product in container.GetItemQueryIterator<Product>(query))
    {
        Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    }
    ```

1. 保存 script.cs 文件 。

1. 在 Visual Studio Code 中，打开 09-execute-query-sdk 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例  。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 该脚本现在将输出容器中的每一个产品

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator
[docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams]: https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/azure.cosmos/4.0.0-preview3]: https://www.nuget.org/packages/azure.cosmos/4.0.0-preview3
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
