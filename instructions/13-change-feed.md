---
lab:
  title: 使用 Azure Cosmos DB SQL API SDK 处理更改源事件
  module: Module 7 - Integrate Azure Cosmos DB SQL API with Azure services
ms.openlocfilehash: 6dbff97f3a587513714610617007080ccd371778
ms.sourcegitcommit: 694767b3c7933a8ee84beca79da880d5874486bc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/17/2022
ms.locfileid: "139057405"
---
# <a name="process-change-feed-events-using-the-azure-cosmos-db-sql-api-sdk"></a>使用 Azure Cosmos DB SQL API SDK 处理更改源事件

Azure Cosmos DB SQL API 更改源是创建由平台中的事件驱动的补充应用程序的关键。 适用于 Azure Cosmos DB SQL API 的 .NET SDK 附带了一套类，用于生成应用程序以与更改源集成并侦听有关容器内操作的通知。

在本实验中，你将使用 .NET SDK 中的更改源处理器功能创建一个应用程序，当对指定容器中的项执行创建或更新操作时，该应用程序会收到通知。

## <a name="prepare-your-development-environment"></a>准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅 [Visual Studio Code 入门指南][code.visualstudio.com/docs/getstarted]

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

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

1. 从资源菜单中选择“数据资源管理器”。

1. 在“数据资源管理器”窗格中，展开“新建容器”，然后选择“新建数据库”。  

1. 在“新建数据库”弹出窗口中，为每个设置输入以下值，然后选择“确定”： 

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | cosmicworks |

1. 返回到“数据资源管理器”窗格，观察层次结构中的“cosmicworks”数据库节点。 

1. 在“数据资源管理器”窗格中，选择“新建容器” 。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定”： 

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 使用现有 &vert; cosmicworks  |
    | **容器 ID** | products |
    | **分区键** | /categoryId |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“products”容器节点。  

1. 在“数据资源管理器”窗格中，再次选择“新建容器” 。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定”： 

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 使用现有 &vert; cosmicworks  |
    | **容器 ID** | productslease |
    | **分区键** | /partitionKey |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“productslease”容器节点。  

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="implement-the-change-feed-processor-in-the-net-sdk"></a>在 .NET SDK 中实现更改源处理器

Microsoft.Azure.Cosmos.Container 类附带一系列方法，用于流畅地生成更改源处理器。 首先，需要引用受监视的容器、租赁容器和一个 C\# 委托（用于处理每批更改）。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到“13-change-feed”文件夹。  

1. 打开 product.cs 代码文件。

1. 观察 Product 类及其对应的属性。 具体而言，此实验室将使用 id 和 name  属性。

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

    > &#128221; 例如，如果键为：fDR2ci9QgkdkvERTQ==，则 C# 语句应为：string key = "fDR2ci9QgkdkvERTQ==";。

1. 使用 client 变量的 GetContainer 方法，使用数据库的名称 (cosmicworks) 和容器的名称 (products) 检索现有容器，并将结果存储在名为 sourceContainer、类型为 Container 的变量中：   

    ```
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    ```

1. 使用 client 变量的 GetContainer 方法，使用数据库的名称 (cosmicworks) 和容器的名称 (productslease) 检索现有容器，并将结果存储在名为 leaseContainer、类型为 Container 的变量中：   

    ```
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    ```

1. 使用具有两个输入参数的空异步匿名函数创建类型为 [ChangesHandler<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1]、名为 handleChanges 的新委托变量：

    1. 类型为 IReadOnlyCollection\<Product\>、名为 changes 的参数。 

    1. 类型为 CancellationToken、名为 cancellationToken 的参数。 

    ```
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes, 
        CancellationToken cancellationToken
    ) => {
    };
    ```

1. 在匿名函数中，使用内置的 Console.WriteLine 静态方法打印原始字符串 START\tHandling batch of changes...： 

    ```
    Console.WriteLine($"START\tHandling batch of changes...");
    ```

1. 仍然是在匿名函数中，创建一个 foreach 循环，该循环使用变量 product 循环访问 changes 变量，以表示类型为 Product 的实例：  

    ```
    foreach(Product product in changes)
    {
    }
    ```

1. 在匿名函数的 foreach 循环中，使用内置的异步 Console.WriteLineAsync 静态方法打印 product 变量的 id 和 name 属性：   

    ```
    await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
    ```

1. 在 foreach 循环和匿名函数之外，创建名为 builder 的新变量，该变量使用以下参数将调用 [GetChangeFeedProcessorBuilder<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder] 的结果存储在 sourceContainer 变量上： 

    | **参数** | **值** |
    | ---: | :--- |
    | processorName | productsProcessor |
    | onChangesDelegate | handleChanges |

    ```
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
        processorName: "productsProcessor",
        onChangesDelegate: handleChanges
    );
    ```

1. 在 builder 变量上流畅地调用参数为 consoleApp 的 [WithInstanceName][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename] 方法、参数为 leaseContainer 的 [WithLeaseContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer] 方法以及 [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build] 方法，将结果存储在类型为 [ChangeFeedProcessor][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]、名为 processor 的变量中：   

    ```
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    ```

1. 异步调用 processor 变量的 [StartAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync]：

    ```
    await processor.StartAsync();
    ```

1. 使用内置的 Console.WriteLine 和 Console.ReadKey 静态方法将输出打印到控制台，让应用程序等待按键操作： 

    ```
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();  
    ```

1. 异步调用 processor 变量的 [StopAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync]：

    ```
    await processor.StopAsync();
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using Microsoft.Azure.Cosmos;
    using static Microsoft.Azure.Cosmos.Container;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClientOptions clientoptions = new CosmosClientOptions()
    {
        RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };

    CosmosClient client = new CosmosClient(endpoint, key, clientoptions);
    
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes, 
        CancellationToken cancellationToken
    ) => {
        Console.WriteLine($"START\tHandling batch of changes...");
        foreach(Product product in changes)
        {
            await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
        }
    };
    
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
            processorName: "productsProcessor",
            onChangesDelegate: handleChanges
        );
    
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    
    await processor.StartAsync();
    
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();
    
    await processor.StopAsync();
    ```

1. 保存 script.cs 文件。 

1. 在 Visual Studio Code 中，打开 13-change-feed 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。 

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 让 Visual Studio Code 和终端保持打开状态。

    > &#128221; 你将使用另一个工具在 Azure Cosmos DB SQL API 容器中生成项。 生成项后，将返回到此终端以观察输出。 不要提前关闭该终端。

## <a name="seed-your-azure-cosmos-db-sql-api-account-with-sample-data"></a>使用示例数据设置 Azure Cosmos DB SQL API 种子帐户

你将使用命令行实用工具来创建 cosmicworks 数据库和 products 容器。  然后，该工具将创建一组项，你将使用终端窗口中运行的更改源处理器来观察它们。

1. 在 Visual Studio Code 中，打开“终端”菜单，然后选择“拆分终端”，以与现有实例并排打开新的终端。  

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

1. 观察来自 .NET 应用程序的终端输出。 终端使用更改源为发送到它的每个更改输出一条“检测到操作”消息。

1. 关闭这两个集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
