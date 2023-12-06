---
lab:
  title: 通过 SDK 连接 Azure Cosmos DB for NoSQL
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# 通过 SDK 连接 Azure Cosmos DB for NoSQL

Azure SDK for .NET 是一套库，提供一致的开发人员界面来与多种 Azure 服务进行交互。 Azure SDK for .NET 是根据 .NET Standard 2.0 规范构建的，确保它可用于 .NET 框架（4.6.1 或更高版本）、.NET Core（2.1 或更高版本）和 .NET（5 或更高版本）应用程序。

在此实验室中，你将使用 Azure SDK for .NET 连接到 Azure Cosmos DB for NoSQL 帐户。

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

    | **设置** | 值 |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **帐户名** | 输入全局唯一名称 |
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *预配的吞吐量* |
    | **应用免费分级折扣** | *不应用* |
    | 限制可在此帐户上预配的总吞吐量 | *未选中* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 请注意“URI”**** 字段。 稍后在本练习中将用到此终结点值。

    1. 请注意“主键”**** 字段。 稍后在本练习中将用到此键值。

1. 使浏览器选项卡保持打开状态，因为我们稍后将返回这里。

## 查看 NuGet 上的 Microsoft.Azure.Cosmos 库

NuGet 网站包含可导入 .NET 应用程序的包的可搜索索引。 若要导入预发行包（如 Microsoft.Azure.Cosmos），可以使用 NuGet 网站获取相应的版本和命令以将包导入应用程序****。

1. 在新浏览器选项卡中，导航到 NuGet 网站 (``nuget.org``)。

1. 查看有关 NuGet、.NET 的包管理器及其功能的描述。

1. 在 NuGet.org 上搜索 Microsoft.Azure.Cosmos 库****。

1. 选择“.NET CLI”选项卡，观察将此库的最新版本导入 .NET 项目所需的命令****。

    > &#128161;无需记录此命令。 你将在此练习稍后使用特定版本的库。

1. 关闭 Web 浏览器窗口或选项卡。

## 将 Microsoft.Azure.Cosmos 库导入 .NET 项目

.NET CLI 包含 [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] 命令，用于从预配置的包源导入包。 .NET 安装使用 NuGet 作为默认包源。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 04-sdk-connect 文件夹************。

1. 打开 04-sdk-connect 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开新的终端实例********。

    > &#128221; 此命令将打开起始目录已设置为 04-sdk-connect 文件夹的终端****。

1. 使用以下命令从 NuGet 添加 [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos] 包：

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.*
    ```

1. 关闭集成终端。

## 使用 Microsoft.Azure.Cosmos 库

导入 Azure SDK for .NET 中的 Azure Cosmos DB 库后，可以立即使用 [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] 命名空间中的类连接到 Azure Cosmos DB for NoSQL 帐户。 [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] 类是一个核心类，用于与 Azure Cosmos DB for NoSQL 帐户建立初始连接。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 04-sdk-connect 文件夹************。

1. 打开空的 script.cs 代码文件****。

1. 为内置的 System 和 System.Linq 命名空间添加 using 块********：

    ```
    using System;
    using System.Linq;
    ```

1. 为 [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] 命名空间添加 using 块：

    ```
    using Microsoft.Azure.Cosmos;
    ```

1. 添加名为 endpoint 的 string 变量，将其值设置为前面创建的 Azure Cosmos DB 帐户的终结点************。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，则 C# 语句将为：string endpoint = "https&shy;://dp420.documents.azure.com:443/";。

1. 添加名为 key 的 string 变量，将其值设置为前面创建的 Azure Cosmos DB 帐户的密钥************。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; 例如，如果键为：fDR2ci9QgkdkvERTQ==，则 C# 语句应为：string key = "fDR2ci9QgkdkvERTQ==";。

1. 使用构造函数中的 endpoint 和 key 变量添加一个名为 client 的 [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] 类型的新变量************：
  
    ```
    CosmosClient client = new (endpoint, key);
    ```

1. 使用调用 client 变量的 [ReadAccountAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync] 方法的异步结果，添加一个名为 account 的 [AccountProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties] 类型的新变量********：

    ```
    AccountProperties account = await client.ReadAccountAsync();
    ```

1. 使用内置的 Console.WriteLine 静态方法来输出 AccountProperties 类的 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id] 属性，并标题设为 Account Name********：

    ```
    Console.WriteLine($"Account Name:\t{account.Id}");
    ```

1. 使用内置的 Console.WriteLine 静态方法查询 AccountProperties 类的 [WritableRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions] 属性，然后输出第一个结果的 [Name][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name] 属性，并将标题设为 Primary Region********：

    ```
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using System.Linq;
    
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    AccountProperties account = await client.ReadAccountAsync();

    Console.WriteLine($"Account Name:\t{account.Id}");
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. 保存 script.cs 代码文件 。

## 测试脚本

现在用于连接到 Azure Cosmos DB for NoSQL 帐户的 .NET 代码已完成，可以测试脚本。 此脚本将输出帐户的名称，以及第一个可写区域的名称。 创建帐户后，你指定了一个位置，应会看到输出为此脚本结果的同一位置值。

1. 在 Visual Studio Code 中，打开 04-sdk-connect 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例************。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 该脚本现在将输出帐户的名称，以及第一个可写区域。 例如，如果将帐户命名为 dp420，并且第一个可写区域是 West US 2，脚本就会输出********：

    ```
    Account Name:   dp420
    Primary Region: West US 2
    ```

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos
