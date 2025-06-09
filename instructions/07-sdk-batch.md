---
lab:
  title: 批处理多点操作与 Azure Cosmos DB for NoSQL SDK
  module: Module 4 - Access and manage data with the Azure Cosmos DB for NoSQL SDKs
---

# 批处理多点操作与 Azure Cosmos DB for NoSQL SDK

[TransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch] 和 [TransactionalBatchResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse] 类是将操作组合和分解为单个逻辑步骤的关键。 使用这两个类，可以编写代码来执行多个操作，然后确定这些操作是否在服务器端成功完成。

在本实验室中，你将使用 SDK 执行两个双项操作，尝试将两个项创建为单个逻辑单元。

## 准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅[入门文档][code.visualstudio.com/docs/getstarted]

1. 打开命令面板并运行 Git: Clone，将 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 存储库克隆到你选择的本地文件夹中。

    > &#128161; 你可以使用 Ctrl+Shift+P 键盘快捷方式打开命令面板。

1. 克隆存储库后，打开在 Visual Studio Code 中选择的本地文件夹。

## 创建 Azure Cosmos DB for NoSQL 帐户并配置 SDK 项目

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭证登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB for NoSQL”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | 工作负载类型**** | **学习** |
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

    1. 请注意“URI”**** 字段。 稍后在本练习中将用到此终结点值。

    1. 请注意“主键”**** 字段。 稍后在本练习中将用到此键值。

1. 返回到 **Visual Studio Code**。

1. 在“资源管理器”**** 窗格中，浏览到“07-sdk-batch”**** 文件夹。

1. 在 07-sdk-batch**** 文件夹内打开 script.cs**** 代码文件。

    > &#128221; 已从 NuGet 中预先导入 [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 库。

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

1. 打开“07-sdk-batch”**** 文件夹的上下文菜单，然后选择“在集成终端中打开”**** 以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为“07-sdk-batch”**** 文件夹的终端。

1. 使用以下命令从 NuGet 添加 [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 包：

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.49.0
    ```

1. 使用 [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 命令生成项目：

    ```
    dotnet build
    ```

1. 关闭集成终端。

## 创建事务批处理

首先，让我们创建一个简单的事务批处理，使其生成两个虚构的产品。 此批处理将使用相同的“旧配件”类别标识符，将 worn saddle 和 rusty handlebar 插入到容器中。 这两个项具有相同的逻辑分区键，确保我们将有一个成功的批处理操作。

1. 返回到 script.cs 代码文件的“编辑器”选项卡。

1. 创建一个名为 saddle**** 的 Product**** 变量，其唯一标识符为 0120****，名称为 Worn Saddle****，类别标识符为 9603ca6c-9e28-4a02-9194-51cdb7fea816****：

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 创建一个名为 handlebar**** 的 Product**** 变量，其唯一标识符为 012A****，名称为 Rusty Handlebar****，类别标识符为 9603ca6c-9e28-4a02-9194-51cdb7fea816****：

    ```
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 创建一个名为 PartitionKey**** 的变量，该变量的类型为 PartitionKey****，将 9603ca6c-9e28-4a02-9194-51cdb7fea816**** 作为构造函数参数传入：

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 调用 container**** 变量的 [CreateTransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch] 方法，将 partitionkey**** 变量作为方法参数传入，并使用熟知的语法调用 [CreateItem<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem] 泛型方法，将 saddle**** 和 handlebar**** 变量作为要在各个操作中创建的项传入，并将结果存储在类型为 TransactionalBatch****、名为 batch**** 的变量中：

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    ```

1. 在 using 语句中，以异步方式调用 batch**** 变量的 ExecuteAsync**** 方法，并将结果存储在类型为 TransactionalBatchResponse****、名为 response**** 的变量中：

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. 调用静态 Console.WriteLine**** 方法以输出 response**** 变量的 StatusCode**** 属性值：

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code**** 中，打开 07-sdk-batch**** 文件夹的上下文菜单，然后选择“在集成终端中打开”**** 以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 查看终端输出。 状态代码应为“OK”****。

1. 关闭集成终端。

## 创建错误的事务批处理

现在来创建一个故意出错的事务批处理。 此批处理将尝试插入两个具有不同逻辑分区键的项。 我们将在“旧配件”类别中创建一个 flickering strobe light，并在“新配件”类别中创建一个 new helmet。 按照定义，此请求应为错误请求，会在执行此事务时返回错误。

1. 返回到 script.cs 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 创建一个名为 light**** 的 Product**** 变量，其唯一标识符为 012B****，名称为 Flickering Strobe Light****，类别标识符为 9603ca6c-9e28-4a02-9194-51cdb7fea816****：

    ```
    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 创建一个名为 helmet**** 的 Product**** 变量，其唯一标识符为 012C****，名称为 New Helmet****，类别标识符为 0feee2e4-687a-4d69-b64e-be36afc33e74****：

    ```
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    ```

1. 创建一个名为 PartitionKey**** 的变量，该变量的类型为 PartitionKey****，将 9603ca6c-9e28-4a02-9194-51cdb7fea816**** 作为构造函数参数传入：

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 调用 container**** 变量的 **CreateTransactionalBatch** 方法，将 partitionkey**** 变量作为方法参数传入，并使用熟知的语法调用 **CreateItem<>** 泛型方法，将 light**** 和 helmet**** 变量作为要在各个操作中创建的项传入，并将结果存储在类型为 TransactionalBatch****、名为 batch**** 的变量中：

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    ```

1. 在 using 语句中，以异步方式调用 batch**** 变量的 ExecuteAsync**** 方法，并将结果存储在类型为 TransactionalBatchResponse****、名为 response**** 的变量中：

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. 调用静态 Console.WriteLine**** 方法以输出 response**** 变量的 StatusCode**** 属性值：

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code**** 中，打开 07-sdk-batch**** 文件夹的上下文菜单，然后选择“在集成终端中打开”**** 以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 查看终端输出。 状态代码应为“错误请求”**** 或“冲突”****。 出现此错误的原因是该事务中的所有项都没有与事务批处理共享相同的分区键值。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
