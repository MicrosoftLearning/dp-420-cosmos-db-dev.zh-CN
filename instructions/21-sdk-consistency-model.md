---
lab:
  title: 在门户和 Azure Cosmos DB for NoSQL SDK 中配置一致性模型
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# 在门户和 Azure Cosmos DB for NoSQL SDK 中配置一致性模型

新 Azure Cosmos DB for NoSQL 帐户的默认一致性级别是会话一致性。 此默认设置可以进行修改以满足所有将来的请求。 在单个请求级别，可以更进一步并放宽该特定请求的一致性级别。

在此实验室中，我们将为 Azure Cosmos DB for NoSQL 帐户配置默认一致性级别，然后使用 SDK 为单个操作配置一致性级别。

## 准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅[入门文档][code.visualstudio.com/docs/getstarted]

1. 打开命令面板并运行 Git: Clone，将 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 存储库克隆到你选择的本地文件夹中。

    > &#128161; 你可以使用 Ctrl+Shift+P 键盘快捷方式打开命令面板。

1. 克隆存储库后，打开在 Visual Studio Code 中选择的本地文件夹。

## 创建 Azure Cosmos DB for NoSQL 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 在首次预配 Azure Cosmos DB 帐户时，可以选择希望该帐户支持的 API（例如 Mongo API 或 NoSQL API）。 完成 Azure Cosmos DB for NoSQL 帐户预配后，可以检索终结点和密钥，并使用它们通过 Azure SDK for .NET 或所选择的任何其他 SDK 连接到 Azure Cosmos DB for NoSQL 帐户。

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
    | **全球分发** &vert; **异地冗余** | *启用* |
    | **应用免费分级折扣** | *不应用* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“全局复制数据”窗格。

1. 在“全局复制数据”窗格中，向帐户添加两个额外的读取区域，然后保存所做的更改。

    > &#128221; 在几个步骤中，系统会要求将一致性级别更改为“强”，但请注意，由于写入延迟较高，默认情况下会阻止跨 5000 英里（8000 公里）以上的帐户的强一致性。 确保选择更靠近的区域。  在生产环境中，要启用此功能，请联系支持人员。

1. 等待复制任务完成，然后继续执行此任务。

    > &#128221; 此操作可能需要大约 5-10 分钟，并导航到“默认一致性”窗格****。

1. 在资源边栏选项卡中，导航到“默认一致性”窗格****。

1. 在“默认一致性”窗格中，选择“强”选项，然后选择“保存”以保持更改************。

1. 等待保存对默认一致性级别的更改，然后再继续执行此任务。

1. 在资源边栏选项卡中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”窗格中，选择“新建容器” 。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定” ：

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 新建 &vert; ``cosmicworks``**** |
    | **在容器之间共享吞吐量** | 请不要选择 |
    | **容器 ID** | *``products``* |
    | **分区键** | *``/categoryId``* |
    | **容器吞吐量** | 手动 &vert; 400 |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“products”容器节点。

1. 在“数据资源管理器”窗格中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“项”。

1. 同样在“数据资源管理器”窗格中，从命令栏中选择“新建项”。 在编辑器中，将占位符 JSON 项替换为以下内容：

    ```
    {
      "id": "7d9273d9-5d91-404c-bb2d-126abb6e4833",
      "categoryId": "78d204a2-7d64-4f4a-ac29-9bfc437ae959",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. 从命令栏中选择“保存”，添加 JSON 项：

1. 在“项”选项卡中，观察“项”窗格中的新项。

1. 在资源边栏选项卡中，导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 请注意“URI”**** 字段。 稍后在本练习中将用到此终结点值。

    1. 请注意“主键”**** 字段。 稍后在本练习中将用到此键值。

1. 返回到 **Visual Studio Code**。

## 从 SDK 连接到 Azure Cosmos DB for NoSQL 帐户

使用新创建帐户中的凭据，你将连接到 SDK 类，并创建新的数据库和容器实例。 然后，你将使用数据资源管理器验证这些实例是否存在于 Azure 门户中。

1. 在“资源管理器”**** 窗格中，浏览到“21-sdk-consistency-model”**** 文件夹。

1. 打开 21-sdk-consistency-model 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开新的终端实例********。

    > &#128221; 此命令将打开起始目录已设置为 21-sdk-consistency-model 文件夹的终端****。

1. 使用 [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 命令生成项目：

    ```
    dotnet build
    ```

    > &#128221; 你可能会看到编译器警告，指出当前未使用 endpoint 和 key 变量。 可以安全地忽略此警告，因为将在此任务中使用这些变量。

1. 关闭集成终端。

1. 打开 product.cs 代码文件。

1. 观察 Product**** 记录及其对应的属性。 具体而言，此实验室将使用 id****、name**** 和 categoryId**** 属性。

1. 返回到 Visual Studio Code 的“资源管理器”窗格，打开 script.cs 代码文件。

    > &#128221; **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.49.0]** 库已从 NuGet 中预先导入。

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

## 为点操作配置一致性级别

ItemRequestOptions 类包含每个请求的配置属性****。 使用此类，可以放宽一致性级别，从当前默认的强一致性放宽到最终一致性。

1. 创建一个名为 id 的字符串变量，其值为 7d9273d9-5d91-404c-bb2d-126abb6e4833********：

    ```
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    ```

1. 创建一个名为 categoryId 的字符串变量，其值为 78d204a2-7d64-4f4a-ac29-9bfc437ae959********：

    ```
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    ```

1. 创建一个名为 partitionKey 的变量，其类型为 **PartitionKey**，将 categoryId 变量作为构造函数参数传入：

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. 异步调用 container 变量的泛型 ReadItemAsync\<\> 方法，传入 id 和 partitionkey 变量作为方法参数，使用 Product 作为泛型类型，并将结果存储在名为 response 的 ItemResponse\<Product\> 类型的变量中****************************：

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. 调用 Console.WriteLine 静态方法，使用格式化的输出字符串输出请求费用****：

    ```
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 完成后，代码文件现在应包含：

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 21-sdk-consistency-model 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例************。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 查看终端输出。 请求费用 (RU) 应输出到控制台。

    > &#128221; 当前请求费用应为 2 RU****。 这是由于强一致性要求从至少两个副本中读取，以确保它有最新的写入。

1. 关闭集成终端。

1. 返回到 script.cs 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 创建一个名为 options 的 [ItemRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions] 类型的新变量，并将 [ConsistencyLevel][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel] 属性设置为 [ConsistencyLevel.Eventual][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel] 枚举值****：

    ```
    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    ```

1. 异步调用 container 变量的泛型 ReadItemAsync\<\> 方法，传入 partitionKey 和 options 变量作为方法参数，使用 Product 作为泛型类型，并将结果存储在名为 response 的 ItemResponse\<Product\> 类型的变量中********************************：

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    ```

1. 调用 Console.WriteLine 静态方法，使用格式化的输出字符串输出请求费用****：

    ```
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 完成后，代码文件现在应包含：

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);

    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 21-sdk-consistency-model 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例************。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 查看终端输出。 请求费用 (RU) 应输出到控制台。

    > &#128221; 当前请求费用应为 1 RU****。 这是因为最终一致性仅要求从单个副本读取。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
