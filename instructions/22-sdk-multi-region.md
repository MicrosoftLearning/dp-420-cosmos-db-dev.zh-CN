---
lab:
  title: 通过 Azure Cosmos DB for NoSQL SDK 连接到多区域写入帐户
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB for NoSQL
---

# 通过 Azure Cosmos DB for NoSQL SDK 连接到多区域写入帐户

CosmosClientBuilder**** 类是一种流畅的类，设计为构建 SDK 客户端以连接到容器并执行操作。 如果你的 Azure Cosmos DB for NoSQL 帐户已配置为进行多区域写入，则可以使用生成器为写入操作配置首选应用程序区域。

在本实验室中，你将为多个区域配置 Azure Cosmos DB for NoSQL 帐户，并启用多区域写入。 然后，你将使用 SDK 针对特定区域执行操作。

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

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“全局复制数据”窗格。

1. 在“全局复制数据”**** 窗格中，将至少一个额外的区域添加到帐户。

1. 同样在“全局复制数据”**** 窗格内，启用“多区域写入”****，然后保存**** 所做的更改。

1. 等待复制任务完成，然后继续执行此任务。

    > &#128221; 此操作大约需要 5-10 分钟。

1. 请注意至少一个你创建的额外区域。 稍后在本练习中将用到此区域值。

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

1. 在资源边栏选项卡中，导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 请注意“URI”**** 字段。 稍后在本练习中将用到此终结点值。

    1. 请注意“主键”**** 字段。 稍后在本练习中将用到此键值。

1. 返回到 Visual Studio Code。

## 从 SDK 连接到 Azure Cosmos DB for NoSQL 帐户

使用新创建帐户中的凭据，你将连接到 SDK 类，并创建新的数据库和容器实例。 然后，你将使用数据资源管理器验证这些实例是否存在于 Azure 门户中。

1. 在“资源管理器”**** 窗格中，浏览到“22-sdk-multi-region”**** 文件夹。

1. 打开“22-sdk-multi-region”**** 文件夹的上下文菜单，然后选择“在集成终端中打开”**** 以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为“22-sdk-multi-region”**** 文件夹的终端。

1. 使用 [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 命令生成项目：

    ```
    dotnet build
    ```

    > &#128221; 你可能会看到编译器警告，指出当前未使用 endpoint 和 key 变量。 可以安全地忽略此警告，因为将在此任务中使用这些变量。

1. 关闭集成终端。

1. 打开 product.cs 代码文件。

1. 观察 Product**** 记录及其对应的属性。 具体而言，此实验室将使用 id****、name**** 和 categoryId**** 属性。

1. 返回到 Visual Studio Code 的“资源管理器”窗格，打开 script.cs 代码文件。

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

## 为 SDK 配置写入区域

流畅的 WithApplicationRegion**** 方法用于为使用 builder 类的后续操作配置首选区域。

1. 创建名为 builder**** 的 [CosmosClientBuilder][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder] 类的新实例，将 endpoint**** 和 key**** 变量作为构造函数参数传入：

    ```
    CosmosClientBuilder builder = new (endpoint, key);
    ```

1. 使用之前在实验室中创建的额外区域的名称，创建一个名为 region**** 的 string**** 类型的新变量。 例如，如果你在“美国东部”**** 区域创建了 Azure Cosmos DB for NoSQL 帐户，然后添加了“巴西南部”****，字符串变量将包含：

    ```
    string region = "Brazil South"; 
    ```

    > &#128161; 或者，可以使用 [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] 静态类，其中包括各种 Azure 区域的内置字符串属性。

1. 在 builder**** 变量上流畅地调用带 region**** 参数的 [WithApplicationRegion][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion] 方法和 [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build] 方法，将结果存储在名为 client**** 的 CosmosClient**** 类型变量中，该变量封装在 using 语句中：

    ```
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    ```

1. 使用 client 变量的 GetContainer 方法，使用数据库名称 (cosmicworks) 和容器名称 (products) 检索现有容器：

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. 通过生成新的 Guid**** 值并将结果存储为字符串，创建名为 id**** 和 categoryId**** 的两个 string**** 变量：

    ```
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    ```

1. 创建名为 item**** 的 Product**** 类型的新变量，将 id**** 变量、Polished Bike Frame**** 字符串值和 categoryId**** 变量作为构造函数参数传入：

    ```
    Product item = new (id, "Polished Bike Frame", categoryId);
    ```

1. 异步调用 container**** 变量的 CreateItemAsync\<\>**** 方法，将 item**** 变量作为参数传入，并将结果存储在名为 response**** 的变量中：

    ```
    var response = await container.CreateItemAsync<Product>(item);
    ```

1. 调用静态 Console.WriteLine**** 方法，输出响应的 HTTP 状态代码和请求费用（按请求单位计）：

    ```
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. 完成后，代码文件现在应包含：

    ```
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Fluent;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";    

    CosmosClientBuilder builder = new (endpoint, key);            
    
    string region = "West Europe";
    
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    Product item = new (id, "Polished Bike Frame", categoryId);
    
    var response = await container.CreateItemAsync<Product>(item);
    
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code**** 中，打开 22-sdk-multi-region**** 文件夹的上下文菜单，然后选择“在集成终端中打开”**** 以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 查看终端输出。 HTTP 状态代码和请求费用（按 RU 计）应输出到控制台。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
