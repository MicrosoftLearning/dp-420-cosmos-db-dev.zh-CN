---
lab:
  title: 配置 Azure Cosmos DB for NoSQL SDK 进行脱机开发
  module: Module 3 - Connect to Azure Cosmos DB for NoSQL with the SDK
---

# <a name="configure-the-azure-cosmos-db-for-nosql-sdk-for-offline-development"></a>配置 Azure Cosmos DB for NoSQL SDK 进行脱机开发

Azure Cosmos DB 仿真器是一个本地工具，可以模拟 Azure Cosmos DB 服务，用于开发和测试。 该仿真器支持 NoSQL，在使用 Azure SDK for .NET 开发代码时，可以使用它来代替云服务。

在本实验室中，你将从 Azure SDK for .NET 连接到 Azure Cosmos DB 仿真器。

## <a name="prepare-your-development-environment"></a>准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅[入门文档][code.visualstudio.com/docs/getstarted]

1. 打开命令面板并运行 Git: Clone，将 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 存储库克隆到你选择的本地文件夹中。

    > &#128161; 你可以使用 Ctrl+Shift+P 键盘快捷方式打开命令面板。

1. 克隆存储库后，打开在 Visual Studio Code 中选择的本地文件夹。

## <a name="start-the-azure-cosmos-db-emulator"></a>启动 Azure Cosmos DB 仿真器

你的环境应已预安装了仿真器。 如果没有，请参阅[安装说明][docs.microsoft.com/azure/cosmos-db/local-emulator]来安装 Azure Cosmos DB 仿真器。 启动仿真器后，可以检索连接字符串，并使用 Azure SDK for .NET 或所选的任何其他 SDK 将其连接到仿真器。

1. 启动 Azure Cosmos DB 仿真器。

    > &#128221; 系统可能会提示你授予管理员访问权限以启动仿真器。 在实验室环境中，Admin 帐户密码与 Student 帐户密码相同。

    > &#128161; Azure Cosmos DB 仿真器已固定到 Windows 任务栏和“开始”菜单。 如果仿真器未从固定图标启动，请尝试通过双击 C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe 文件来打开它。 请注意，仿真器需要 20-30 秒才能启动。

1. 等待仿真器自动打开默认浏览器并导航到 localhost:8081/_explorer/index.html 登陆页面。

1. 在 Azure Cosmos DB 仿真器登陆页面中，导航到“快速入门”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“主连接字符串”字段的值。 你将在本练习的后面部分用到此连接字符串值。

1. 导航到“资源管理器”窗格。

1. 在“数据资源管理器”中，观察“NoSQL API”导航树中是否没有节点。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="connect-to-the-emulator-from-the-sdk"></a>从 SDK 连接到仿真器

Microsoft.Azure.Cosmos 库已预安装在你将在本练习中使用的 .NET 脚本中。 此外，已编写一些样本代码以节省你的时间。 你将需要更新样本连接字符串值并编写几行代码才能完成脚本。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 05-sdk-offline 文件夹。

1. 在 05-sdk-offline 文件夹内打开 script.cs 代码文件。

1. 更新名为 connectionString 的现有变量，将值设置为 Azure Cosmos DB 仿真器的连接字符串。
  
    ```
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    ```

    > &#128221; 仿真器的 URI 通常是使用 SSL 的 localhost:[port]，默认端口设置为 8081。

    > &#128221; C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw== 是仿真器所有安装的默认键。 可以使用命令行选项更改此键。

1. 异步调用 client 变量的 [CreateDatabaseIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync] 方法，传入要在仿真器中创建的新数据库的名称 (cosmicworks)，并将结果存储在 [Database][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database] 类型的变量中：

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. 使用内置的 Console.WriteLine 静态方法来输出 Database 类的 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id] 属性，并将标题设置为 New Database：

    ```
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 05-sdk-offline 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为“05-sdk-offline”文件夹的终端。

1. 使用以下命令从 NuGet 添加 [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 包：

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 关闭集成终端。

## <a name="view-the-changes-in-the-emulator"></a>在仿真器中查看更改

在 Azure Cosmos DB 仿真器中创建了一个新数据库后，将使用联机数据资源管理器来观察仿真器中的 NoSQL API 数据库。

1. 导航到 Windows 系统托盘中的仿真器图标，打开上下文菜单，然后选择“打开数据资源管理器...”，使用默认浏览器导航到 localhost:8081/_explorer/ 登陆页面。

1. 在 Azure Cosmos DB 仿真器登陆页面中，导航到“资源管理器”窗格。

1. 在“数据资源管理器”中，观察“NoSQL API”导航树中的新“cosmicworks”数据库节点。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="create-and-view-a-new-container"></a>创建和查看新容器

创建新容器的模式与创建新数据库类似。 无论是在云中还是在仿真器中创建资源，你在这里学到的代码都是相关的，你只需更改连接字符串。 进一步展开脚本文件，创建新容器以及数据库。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 05-sdk-offline 文件夹。

1. 在 05-sdk-offline 文件夹内，再次打开 script.cs 代码文件。

1. 异步调用 database 变量的 [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] 方法，传入新容器的名称 (products)、分区键路径 (/categoryId)，以及要在 cosmicworks 中创建的吞吐量 (400)，并将结果存储在 [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 类型的变量中：

    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. 使用内置的 Console.WriteLine 静态方法来输出 Container 类的 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] 属性，并将标题设置为 New Container：

    ```
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. 保存 script.cs 代码文件 。

1. 在 Visual Studio Code 中，打开 05-sdk-offline 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

1. 导航到 Windows 系统托盘中的仿真器图标，打开上下文菜单，然后选择“打开数据资源管理器...”，使用默认浏览器导航到 localhost:8081/_explorer/ 登陆页面。

1. 在 Azure Cosmos DB 仿真器登陆页面中，导航到“资源管理器”窗格。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“NoSQL API”导航树中观察新“products”容器节点。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="stop-the-azure-cosmos-db-emulator"></a>停止 Azure Cosmos DB 仿真器

使用完仿真器后，必须将其停止，因为它会在你的环境中使用系统资源。 使用系统托盘图标停止仿真器以及所有正在运行的实例。

1. 导航到 Windows 系统托盘中的仿真器图标，打开上下文菜单，然后选择“退出”以关闭仿真器。

    > &#128221; 退出仿真器的所有实例可能需要一分钟的时间。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
