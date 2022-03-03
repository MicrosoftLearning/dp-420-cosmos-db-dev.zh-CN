---
lab:
  title: 使用 Azure Cosmos DB SQL API SDK 创建和更新文档
  module: Module 4 - Implement Azure Cosmos DB SQL API point operations
ms.openlocfilehash: 5aa8e7ec314243dce08e3d14a561e45d2ac83049
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2022
ms.locfileid: "138024923"
---
# <a name="create-and-update-documents-with-the-azure-cosmos-db-sql-api-sdk"></a>使用 Azure Cosmos DB SQL API SDK 创建和更新文档

[Microsoft.Azure.Cosmos.Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 类包括一组成员方法，用于在 Azure Cosmos DB SQL API 容器中创建、检索、更新和删除项。 这些方法共同在 SQL API 容器中对各种项执行一些最常见的“CRUD”操作。

在本实验室中，你将在 Azure Cosmos DB SQL API 容器中使用 SDK 对项执行日常 CRUD 操作。

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

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="connect-to-the-azure-cosmos-db-sql-api-account-from-the-sdk"></a>从 SDK 连接到 Azure Cosmos DB SQL API 帐户

使用新创建帐户中的凭据，你将连接到 SDK 类，并创建新的数据库和容器实例。 然后，你将使用数据资源管理器验证这些实例是否存在于 Azure 门户中。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到“06-sdk-crud”文件夹。

1. 打开“06-sdk-crud”文件夹的上下文菜单，然后选择“在集成终端中打开”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为“06-sdk-crud”文件夹的终端。

1. 使用 [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 命令生成项目：

    ```
    dotnet build
    ```

1. 关闭集成终端。

1. 在 06-sdk-crud 文件夹内打开 script.cs 代码文件。

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

    > &#128221; 例如，如果键为：fDR2ci9QgkdkvERTQ==，则 C# 语句应为：string key = "fDR2ci9QgkdkvERTQ==";。

1. 异步调用 client 变量的 CreateDatabaseIfNotExistsAsync 方法，该方法传入你要在仿真器中创建的新数据库的名称 (cosmicworks)，并将结果存储在 Database 类型的变量中：

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. 异步调用 database 变量的 CreateContainerIfNotExistsAsync 方法，传入新容器的名称 (products)、分区键路径 (/categoryId)，以及要在 cosmicworks 中创建的吞吐量 (400)，并将结果存储在 Container 类型的变量中：
  
    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);    
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);  
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 06-sdk-crud 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 关闭集成终端。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，选择先前在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“SQL API”导航树中观察新“products”容器节点。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="perform-create-and-read-point-operations-on-items-with-the-sdk"></a>使用 SDK 对项执行创建和读取点操作

现在，你将在 Microsoft.Azure.Cosmos.Container 类中使用一组异步方法，对 SQL API 容器中的项执行常见操作。 这些操作都是使用 C# 中的任务异步编程模型完成的。

1. 返回到 Visual Studio Code。 在 06-sdk-crud 文件夹内打开 product.cs 代码文件。

    > &#128221; 不要关闭 script.cs 文件编辑器。

1. 查看此代码文件中的 Product 类。 此类表示将在此容器中进行存储和操作的产品项。

1. 返回到 script.cs 代码文件的“编辑器”选项卡。

1. 使用以下属性创建一个 Product 类型的名为 saddle 的新对象：

    | 属性 | Value |
    | ---: | :--- |
    | **id** | ``706cd7c6-db8b-41f9-aea2-0e0c7e8eb009`` |
    | categoryId | ``9603ca6c-9e28-4a02-9194-51cdb7fea816`` |
    | name | ``Road Saddle`` |
    | **price** | ``45.99d`` |
    | **标记** | ``{ tan, new, crisp }`` |

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };
    ```

1. 异步调用 container 变量的泛型 [CreateItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync] 方法，将 saddle 变量作为方法参数传入，并使用 Product 作为泛型类型：

    ```
    await container.CreateItemAsync<Product>(saddle);
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);  
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 06-sdk-crud 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 关闭集成终端。

1. 返回到 script.cs 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. 创建一个名为 id 的字符串变量，其值为 706cd7c6-db8b-41f9-aea2-0e0c7e8eb009：

    ```
    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. 创建一个名为 categoryId 的字符串变量，其值为 9603ca6c-9e28-4a02-9194-51cdb7fea816：

    ```
    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. 创建一个名为 partitionKey 的变量，其类型为 [PartitionKey][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey]，将 categoryId 变量作为构造函数参数传入：

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. 异步调用 container 变量的泛型 [ReadItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] 方法，将 id 和 partitionkey 变量作为方法参数传入，使用 Product 作为泛型类型，并将结果存储在 Product 类型的名为 saddle 的变量中：

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. 调用静态 Console.WriteLine 方法，使用格式化的输出字符串输出 saddle 对象：

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);  
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 06-sdk-crud 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 查看终端输出。 具体而言，请查看带有项的 id、名称和价格的格式化输出文本。

1. 关闭集成终端。

## <a name="perform-update-and-delete-point-operations-with-the-sdk"></a>使用 SDK 执行更新和删除点操作

在学习 SDK 期间，使用联机 Azure Cosmos DB SDK 帐户或仿真器来更新项，在执行操作时在数据资源管理器与所选的 IDE 之间来回切换，以及查看更改是否已应用，是很常见的情况。 在本实验室中，当你使用 SDK 更新和删除项时，你就会这样做。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，选择先前在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“SQL API”导航树中展开新“products”容器节点。

1. 选择“Items”节点。 选择容器中的唯一项，然后观察项的 name 和 price 属性的值。

    | **属性** | **值** |
    | ---: | :--- |
    | **名称** | Road Saddle |
    | **价格** | 45.99 美元 |

    > &#128221; 此时，自从你创建了这个项后，这些值不应发生更改。 在本练习中，你将更改这些值。

1. 关闭 Web 浏览器窗口或选项卡。

1. 返回到 Visual Studio Code。 返回到 script.cs 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. 通过将 price 属性的值设置为 32.55 来更改 saddle 变量：

    ```
    saddle.price = 32.55d;
    ```

1. 通过将 name 属性的值设置为 Road LL Saddle，再次修改 saddle 变量：

    ```
    saddle.name = "Road LL Saddle";
    ```

1. 异步调用 container 变量的泛型 [UpsertItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync] 方法，将 saddle 变量作为方法参数传入，并使用 Product 作为泛型类型：

    ```
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);  
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 06-sdk-crud 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 关闭集成终端。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，选择先前在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“SQL API”导航树中展开新“products”容器节点。

1. 选择“Items”节点。 选择容器中的唯一项，然后观察项的 name 和 price 属性的值。

    | **属性** | **值** |
    | ---: | :--- |
    | **名称** | Road LL Saddle |
    | **价格** | 32.55 美元 |

    > &#128221; 此时，自从你观察到这个项后，这些值应发生更改。

1. 关闭 Web 浏览器窗口或选项卡。

1. 返回到 Visual Studio Code。 返回到 script.cs 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. 异步调用 container 变量的泛型 [DeleteItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync] 方法，将 id 和 partitionkey 变量作为方法参数传入，并使用 Product 作为泛型类型：

    ```
    await container.DeleteItemAsync<Product>(id, partitionKey);
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 06-sdk-crud 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 关闭集成终端。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，选择先前在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“SQL API”导航树中展开新“products”容器节点。

1. 选择“Items”节点。 应会看到项列表现在为空。

1. 关闭 Web 浏览器窗口或选项卡。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
