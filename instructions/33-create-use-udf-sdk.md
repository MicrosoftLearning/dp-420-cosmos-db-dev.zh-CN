---
lab:
  title: 通过 SDK 实现和使用用户定义的函数
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB for NoSQL
---

# 通过 SDK 实现和使用用户定义的函数

适用于 Azure Cosmos DB for NoSQL 的 .NET SDK 可用于直接从容器管理和调用服务器端编程结构。 准备新容器时，建议使用 .NET SDK 将 UDF 直接发布到容器，而不是使用数据资源管理器手动执行任务。

在本实验室中，你将使用 .NET SDK 创建新的 UDF，然后使用数据资源管理器验证 UDF 是否正常工作。

## 准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅 [Visual Studio Code 入门指南][code.visualstudio.com/docs/getstarted]

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
    | **应用免费分级折扣** | *不应用* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 请注意“URI”**** 字段。 稍后在本练习中将用到此终结点值。

    1. 请注意“主键”**** 字段。 稍后在本练习中将用到此键值。

    1. 请注意“主连接字符串”**** 字段。 你将在此练习的稍后部分使用此连接字符串值。

1. 在不关闭浏览器窗口的情况下，打开 Visual Studio Code****。

## 使用数据为 Azure Cosmos DB for NoSQL 帐户设定种子

[cosmicworks][nuget.org/packages/cosmicworks] 命令行工具将示例数据部署到任何 Azure Cosmos DB for NoSQL 帐户。 该工具是开源工具，可通过 NuGet 获得。 将此工具安装到 Azure Cloud Shell，然后使用它来设定数据库种子。

1. 在 Visual Studio Code 中，打开“终端”菜单，然后选择“新建终端”以打开新的终端实例  。

1. 在计算机上安装可全局使用的 [cosmicworks][nuget.org/packages/cosmicworks] 命令行工具。

    ```
    dotnet tool install --global CosmicWorks --version 2.3.1
    ```

    > &#128161; 此命令可能需要几分钟时间才能完成。 如果你过去已经安装了此工具的最新版本，此命令将输出警告消息（*工具 "cosmicworks" 已安装）。

1. 使用以下命令行选项运行 cosmicworks 以设置 Azure Cosmos DB 种子帐户：

    | **选项** | 值 |
    | ---: | :--- |
    | **-c** | *之前在本实验室中勾选的连接字符串值* |
    | **--number-of-employees** | *cosmicworks 命令会用员工容器和产品容器分别填充数据库，默认包含 1000 个和 200 个项目，除非另有指定。* |

    ```powershell
    cosmicworks -c "connection-string" --number-of-employees 0 --disable-hierarchical-partition-keys
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，密钥为：fDR2ci9QgkdkvERTQ==，则命令为：``cosmicworks -c "AccountEndpoint=https://dp420.documents.azure.com:443/;AccountKey=fDR2ci9QgkdkvERTQ==" --number-of-employees 0 --disable-hierarchical-partition-keys``

1. 等待 cosmicworks 命令完成对帐户的数据库、容器和项的填充。

1. 关闭集成终端。

## 使用 .NET SDK 创建用户定义函数 (UDF)

.NET SDK 中的 [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 类包括一个 [Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] 属性，该属性用于直接从 SDK 对存储过程、UDF 和触发器执行 CRUD 操作。 你将使用此属性创建新的 UDF，然后将该 UDF 推送到 Azure Cosmos DB for NoSQL 容器。 我们使用 SDK 创建的 UDF 将计算含税产品价格，这样，我们就可以使用产品的含税价格对产品运行 SQL 查询。

1. 在 Visual Studio Code的“资源管理器”窗格中，浏览到“33-create-use-udf-sdk”文件夹。

1. 打开 script.cs 代码文件。

1. 为 [Microsoft.Azure.Cosmos.Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts] 命名空间添加 using 块：

    ```
    using Microsoft.Azure.Cosmos.Scripts;
    ```

1. 更新名为 endpoint 的现有变量，将其值设置为前面创建的 Azure Cosmos DB 帐户的终结点 。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，则 C# 语句将为：string endpoint = "https&shy;://dp420.documents.azure.com:443/";。

1. 更新名为 key 的现有变量，将其值设置为前面创建的 Azure Cosmos DB 帐户的键 。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; 例如，如果键为：fDR2ci9QgkdkvERTQ==，则 C# 语句应为：string key = "fDR2ci9QgkdkvERTQ==";。

1. 使用默认空构造函数创建名为 props 的 [UserDefinedFunctionProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties] 类型的新变量：

    ```
    UserDefinedFunctionProperties props = new ();
    ```

1. 将 props 变量的 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id] 属性值设置为 tax：

    ```
    props.Id = "tax";
    ```

1. 将 props 变量的 [Body][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body] 属性值设置为 props.Body = "function tax(i) { return i * 1.25; }";：

    ```
    props.Body = "function tax(i) { return i * 1.25; }";
    ```

1. 异步调用 container 变量的 [Scripts.CreateUserDefinedFunctionAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] 方法，将 props 变量作为参数传入，并将结果保存在 [UserDefinedFunctionResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse] 类型的名为 udf 的变量中：

    ```
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    ```

1. 使用内置的 Console.WriteLine 静态方法输出 UserDefinedFunctionResponse 类的 [Resource.Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource] 属性，并将标题设置为 Created UDF：

    ```
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Scripts;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name");

    UserDefinedFunctionProperties props = new ();
    props.Id = "tax";
    props.Body = "function tax(i) { return i * 1.25; }";
    
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. 保存 script.cs 文件。

1. 在 Visual Studio Code 中，打开 33-create-use-udf-sdk 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 该脚本现在将输出新创建的 UDF 的名称：

    ```
    Created UDF [tax]
    ```

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

## 使用数据资源管理器测试 UDF

在 Azure Cosmos DB 容器中创建了新的 UDF 后，使用数据资源管理器验证 UDF 是否正常工作。

1. 返回 Web 浏览器。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”**** 中，展开“cosmicworks”**** 数据库节点，然后在“NoSQL API”**** 导航树中观察新“products”**** 容器节点。

1. 选择“NoSQL API”**** 导航树中的“products”**** 容器节点，然后选择“新建 SQL 查询”****。

1. 在“查询”选项卡中，选择“执行查询”以查看选择不包含任何筛选器的所有项的标准查询。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询将返回预测有两个价格值的所有文档。 第一个值是容器中的原始价格值，第二个值是 UDF 计算的价格值：

    ```
    SELECT p.id, p.price, udf.tax(p.price) AS priceWithTax FROM products p
    ```

1. 选择“执行查询”。

1. 查看文档并比较其 price 和 priceWithTax 字段。

    > &#128221; priceWithTax 字段值应比 price 字段值大 25%。

1. 关闭 Web 浏览器窗口或选项卡。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
