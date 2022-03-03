---
lab:
  title: 通过 Azure Cosmos DB SQL API SDK 连接到不同区域
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: 8d6439943bc1baac6e5c1cdb8e40f98cf6665119
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2022
ms.locfileid: "138024942"
---
# <a name="connect-to-different-regions-with-the-azure-cosmos-db-sql-api-sdk"></a>通过 Azure Cosmos DB SQL API SDK 连接到不同区域

为 Azure Cosmos DB SQL API 帐户启用异地冗余时，可以使用 SDK 按你配置的任何顺序从区域中读取数据。 当你在所有可用的读取区域分发读取请求时，此方法非常有用。

在本实验室中，你将配置 CosmosClient 类，按手动配置的回退顺序连接到读取区域。

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
    | **容量模式** | *预配的吞吐量* |
    | **应用免费分级折扣** | *不应用* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用预先创建的现有资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“全局复制数据”窗格。

1. 在“全局复制数据”窗格中，向帐户添加两个额外的读取区域，然后保存所做的更改。

1. 等待复制任务完成，然后继续执行此任务。

    > &#128221; 此操作大约需要 5-10 分钟。

1. 记录“写入”（主要）区域和两个“读取”区域的名称。 稍后在本练习中将用到这些区域名称。

    > &#128221; 例如，如果主要区域是“北欧”，并且两个读取的次要区域是“美国东部 2”和“南非北部”；你将按原样记录这三个名称。

1. 在资源边栏选项卡中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”窗格中，选择“新建容器” 。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定”： 

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 新建 &vert; cosmicworks |
    | **在容器之间共享吞吐量** | 请不要选择 |
    | **容器 ID** | products |
    | **分区键** | /categoryId |
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

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="connect-to-the-azure-cosmos-db-sql-api-account-from-the-sdk"></a>从 SDK 连接到 Azure Cosmos DB SQL API 帐户

使用新创建帐户中的凭据，你将连接到 SDK 类，并访问不同区域的数据库和容器实例。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到“20-sdk-regions”文件夹。

1. 打开“20-sdk-regions”文件夹的上下文菜单，然后选择“在集成终端中打开”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为“20-sdk-regions”文件夹的终端。

1. 使用 [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 命令生成项目：

    ```
    dotnet build
    ```

    > &#128221; 你可能会看到编译器警告，指出当前未使用 endpoint 和 key 变量。 可以安全地忽略此警告，因为将在此任务中使用这些变量。

1. 关闭集成终端。

1. 在 20-sdk-regions 文件夹内打开 script.cs 代码文件。

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

    > &#128221; 例如，如果键为：fDR2ci9QgkdkvERTQ==，则 C# 语句应为：string key = "fDR2ci9QgkdkvERTQ==";。

1. 保存 script.cs 代码文件 。

## <a name="configure-the-net-sdk-with-a-preferred-region-list"></a>使用首选区域列表配置 .NET SDK

CosmosClientOptions 类包含一个属性，用于配置要使用 SDK 连接到的区域列表。 列表按故障转移优先级排序，尝试按你配置的顺序连接到每个区域。

1. 创建一个泛型类型 List\<string\> 的新变量，其中包含你为帐户配置的区域列表，从第三个区域开始，到第一个（主要）区域结束。 例如，如果你在“美国西部”区域中创建了 Azure Cosmos DB SQL API 帐户，然后添加了“南非北部”，最后添加了“东亚”；那么，你的列表变量将包含：

    ```
    List<string> regions = new()
    {
        "East Asia",
        "South Africa North",
        "West US"
    };
    ```

    > &#128161; 或者，可以使用 [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] 静态类，其中包括各种 Azure 区域的内置字符串属性。

1. 创建名为 options 的 CosmosClientOptions 类的新实例，并将 [ApplicationPreferredRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions] 属性设置为 regions 变量：

    ```
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    ```

1. 创建名为 client 的 CosmosClient 类的新实例，将 endpoint、key 和 options 变量作为构造函数参数传入：

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. 使用 client 变量的 GetContainer 方法，使用数据库名称 (cosmicworks) 和容器名称 (products) 检索现有容器：

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. 使用 container 变量的 [ReadItemAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] 方法从服务器中检索特定项，并将结果存储在名为 response 的可为 null 的 [ItemResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse] 类型的变量中：

    ```
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    ```

1. 调用静态 Console.WriteLine 方法，以输出当前项标识符和 JSON 诊断数据：

    ```
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine($"Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    List<string> regions = new()
    {
        "<read-region-2>",
        "<read-region-1>",
        "<write-region>"
    };
    
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    
    using CosmosClient client = new(endpoint, key, options);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine("Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 20-sdk-regions 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 查看终端输出。 容器名称和 JSON 诊断数据应打印到控制台输出。

1. 查看 JSON 诊断数据。 搜索名为 HttpResponseStats 的属性和名为 RequestUri 的子属性。 此属性的值应该是一个 URI，其中包含先前在本实验室中配置的名称和区域。

    > &#128221; 例如，如果你的帐户名称为：dp420，而你配置的第一个区域为“东亚”；则 JSON 属性的值将为：dp420-eastasia.documents.azure.com/dbs/cosmicworks/colls/products。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
