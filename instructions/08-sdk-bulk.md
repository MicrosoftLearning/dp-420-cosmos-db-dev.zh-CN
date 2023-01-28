---
lab:
  title: 使用 Azure Cosmos DB for NoSQL SDK 批量移动多个文档
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# <a name="move-multiple-documents-in-bulk-with-the-azure-cosmos-db-for-nosql-sdk"></a>使用 Azure Cosmos DB for NoSQL SDK 批量移动多个文档

要了解如何执行批量操作，最简单的方法是尝试将多个文档推送到云中的 Azure Cosmos DB for NoSQL 帐户。 使用 SDK 的批量功能，可通过 [System.Threading.Tasks][docs.microsoft.com/dotnet/api/system.threading.tasks] 命名空间提供的一些小帮助完成此操作。

在本实验室中，你将使用 NuGet 的 [Bogus][nuget.org/packages/bogus/33.1.1] 库生成虚构的数据，并将其放入 Azure Cosmos DB 帐户。

## <a name="prepare-your-development-environment"></a>准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅[入门文档][code.visualstudio.com/docs/getstarted]

1. 打开命令面板并运行 Git: Clone，将 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 存储库克隆到你选择的本地文件夹中。

    > &#128161; 你可以使用 Ctrl+Shift+P 键盘快捷方式打开命令面板。

1. 克隆存储库后，打开在 Visual Studio Code 中选择的本地文件夹。

## <a name="create-an-azure-cosmos-db-for-nosql-account-and-configure-the-sdk-project"></a>创建 Azure Cosmos DB for NoSQL 帐户并配置 SDK 项目

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭证登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB for NoSQL”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | 值 |
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

1. 同样在新创建的 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，选择“新建容器”，然后创建一个具有以下设置的新容器，并将所有其余设置保留为其默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **数据库 ID** | 新建 &vert; `cosmicworks`  |
    | **在容器之间共享吞吐量** | 请不要选择 |
    | **容器 ID** | *`products`* |
    | **分区键** | *`/categoryId`* |
    | **容器吞吐量** | 自动缩放 &vert; `4000`  |

1. 关闭 Web 浏览器窗口或选项卡。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到“08-sdk-bulk”文件夹。

1. 在 08-sdk-bulk 文件夹内打开 script.cs 代码文件。

    > &#128221; [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 库已从 NuGet 中预先导入。

1. 找到名为 endpoint 的 string 变量。 将它的值设置为之前创建的 Azure Cosmos DB 帐户的终结点。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，则 C# 语句将为：string endpoint = "https&shy;://dp420.documents.azure.com:443/";。

1. 找到名为 key 的 string 变量。 将它的值设置为之前创建的 Azure Cosmos DB 帐户的键。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; 例如，如果键为：fDR2ci9QgkdkvERTQ==，则 C# 语句应为：string key = "fDR2ci9QgkdkvERTQ=="; 。

1. 保存 script.cs 代码文件 。

1. 打开“08-sdk-bulk”文件夹的上下文菜单，然后选择“在集成终端中打开”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为“08-sdk-bulk”文件夹的终端。

1. 使用以下命令从 NuGet 添加 [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 包：

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 使用 [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 命令生成项目：

    ```
    dotnet build
    ```

1. 关闭集成终端。

## <a name="bulk-inserting-a-twenty-five-thousand-documents"></a>批量插入 25,000 个文档

接下来，我们不妨尝试大胆地插入大量文档，看看它是如何工作的。 在内部测试中，如果实验室虚拟机和 Azure Cosmos DB for NoSQL 帐户在地理上相对接近，则可能需要大约 1-2 分钟。

1. 返回到 script.cs 代码文件的“编辑器”选项卡。

1. 创建名为 options 的 [CosmosClientOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions] 类的新实例，并将 AllowBulkExecution 属性设置为值 true：

    ```
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    ```

1. 创建名为 client 的 CosmosClient 类的新实例，将 endpoint、key 和 options 变量作为构造函数参数传入：

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. 使用 client 变量的 [GetContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer] 方法，使用数据库名称 (cosmicworks) 和容器名称 (products) 检索现有容器：

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. 使用此特殊示例代码，使用从 NuGet 导入的 Bogus 库中的 Faker 类生成 25,000 个虚构产品。

    ```
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
    ```

    > &#128161; [Bogus][nuget.org/packages/bogus/33.1.1] 库是一个开源库，用于设计虚构的数据来测试用户界面应用程序，非常适合用户学习如何开发批量导入/导出应用程序。

1. 新建名为 concurrentTasks、类型为 Task 的泛型 List<>：

    ```
    List<Task> concurrentTasks = new List<Task>();
    ```

1. 创建一个 foreach 循环，该循环将循环访问此应用程序之前生成的产品列表：

    ```
    foreach(Product product in productsToInsert)
    {
    }
    ```

1. 在 foreach 循环中，创建一个 Task，以异步方式将产品插入到 Azure Cosmos DB for NoSQL，以确保显式指定分区键，并将该任务添加到名为 concurrentTasks 的任务列表：

    ```
    concurrentTasks.Add(
        container.CreateItemAsync(product, new PartitionKey(product.categoryId))
    );   
    ```

1. 在 foreach 循环后，以异步方式等待 concurrentTasks 变量上 Task.WhenAll 的结果：

    ```
    await Task.WhenAll(concurrentTasks);
    ```

1. 使用内置 Console.WriteLine 静态方法，将“批量处理任务完成”的静态消息输出到控制台：

    ```
    Console.WriteLine("Bulk tasks complete");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Bogus;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    
    CosmosClient client = new (endpoint, key, options);  
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
        
    List<Task> concurrentTasks = new List<Task>();
    
    foreach(Product product in productsToInsert)
    {    
        concurrentTasks.Add(
            container.CreateItemAsync(product, new PartitionKey(product.categoryId))
        );
    }
    
    await Task.WhenAll(concurrentTasks);   

    Console.WriteLine("Bulk tasks complete");
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 08-sdk-bulk 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 应用程序应以无提示方式运行，在以无提示方式完成之前，大约需要一到两分钟的运行时间。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

## <a name="observe-the-results"></a>观察结果

现在，你已将 25,000 个项发送到 Azure Cosmos DB，接下来查看数据资源管理器。

1. 在 Web 浏览器中，导航到 Azure 门户 (``portal.azure.com``)。

1. 选择“资源组”，选择先前在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“NoSQL API”导航树中观察“products”容器节点。

1. 展开“products”节点，然后选择“Items”节点。 查看容器中的项列表。

1. 选择“NoSQL API”导航树中的“products”容器节点，然后选择“新建 SQL 查询”。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询将返回使用批量操作创建的所有文档计数：

    ```
    SELECT COUNT(1) FROM items
    ```

1. 选择“执行查询”。

1. 查看容器中项的计数。

1. 关闭 Web 浏览器窗口或选项卡。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions
[docs.microsoft.com/dotnet/api/system.threading.tasks]: https://docs.microsoft.com/dotnet/api/system.threading.tasks
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/bogus/33.1.1]: https://www.nuget.org/packages/bogus/33.1.1
