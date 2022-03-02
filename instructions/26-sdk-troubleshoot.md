---
lab:
  title: 使用 Azure Cosmos DB SQL API SDK 对应用程序进行故障排除
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB SQL API solution
ms.openlocfilehash: e87e27c83e9aa41ed7494097ce3e0e64a2b46a2f
ms.sourcegitcommit: c3778722311b55568f083480ecc69c9b3e837a18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/19/2022
ms.locfileid: "138025014"
---
# <a name="troubleshoot-an-application-using-the-azure-cosmos-db-sql-api-sdk"></a>使用 Azure Cosmos DB SQL API SDK 对应用程序进行故障排除

Azure Cosmos DB 提供了一组广泛的响应代码，可帮助我们轻松解决不同操作类型产生的问题。 捕获是为了确保在为 Azure Cosmos DB 创建应用时，我们创建了正确的错误处理程序。

在此实验中，将创建一个菜单驱动程序，用于支持插入或删除两个文档中的一个。 此实验的主要目的是介绍如何使用一些最常见的响应代码以及如何在应用的错误处理代码中使用它们。  虽然我们将为多个响应代码的错误处理进行编码，但只会在两种不同类型的条件下触发。  此外，错误处理中不执行任何复杂的操作，可能在屏幕上显示一条消息或等待 10 秒然后重试操作一次，具体取决于响应代码。 

## <a name="prepare-your-development-environment"></a>准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅 [Visual Studio Code 入门指南][code.visualstudio.com/docs/getstarted]

1. 打开命令面板并运行 Git: Clone，将 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 存储库克隆到你选择的本地文件夹中。

    > &#128161; 可以使用 CTRL+SHIFT+P 键盘快捷方式打开命令面板。

1. 克隆存储库后，打开在 Visual Studio Code 中选择的本地文件夹。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>创建 Azure Cosmos DB SQL API 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 在首次预配 Azure Cosmos DB 帐户时，可以选择希望该帐户支持的 API（例如 Mongo API 或 SQL API） 。 Azure Cosmos DB SQL API 帐户完成预配后，你可以检索终结点和密钥。 使用终结点和密钥以编程方式连接到 Azure Cosmos DB SQL API 帐户。 在 Azure SDK for .NET 或任何其他 SDK 的连接字符串上使用终结点和密钥。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB SQL API”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **帐户名** | 输入一个全局唯一名称 |
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *预配的吞吐量* |
    | **应用免费分级折扣** | *`Do Not Apply`* |

    > &#128221; 实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

1. 最小化浏览器窗口，但不要关闭它。 在接下来的步骤中，我们将在启动后台工作负载几分钟后返回 Azure 门户。


## <a name="import-the-microsoftazurecosmos-library-into-a-net-script"></a>将 Microsoft.Azure.Cosmos 库导入 .NET 脚本

.NET CLI 包含 [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] 命令，用于从预配置的包源导入包。 .NET 安装使用 NuGet 作为默认包源。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 26-sdk-troubleshoot  。

1. 打开 26-sdk-troubleshoot 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开新的终端实例 。

    > &#128221; 此命令将打开起始目录已设置为 26-sdk-troubleshoot 文件夹的终端。

1. 使用以下命令从 NuGet 添加 [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 包：

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

## <a name="run-a-script-to-create-menu-driven-options-to-insert-and-delete-documents"></a>运行脚本以创建菜单驱动的选项，用于插入和删除文档。

在应用程序能够运行之前，需要将其连接到 Azure Cosmos DB 帐户。 

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 26-sdk-troubleshoot  。

1. 打开 Program.cs 代码文件。

1. 更新名为 endpoint 的现有变量，将其值设置为前面创建的 Azure Cosmos DB 帐户的终结点。
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，则 C# 语句将为：private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";。

1. 更新名为 key 的现有变量，将其值设置为前面创建的 Azure Cosmos DB 帐户的键。

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; 例如，如果你的键为：fDR2ci9QgkdkvERTQ==，则 C# 语句应为：private static readonly string key = "fDR2ci9QgkdkvERTQ==";。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```
    > &#128221; 这是一个非常简单的程序。  它将显示一个菜单，其中包含五个选项，如下所示。 两个选项用于插入预定义的文档，两个用于删除预定义的文档，还有一个选项用于退出程序。

    >```
    >1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >5) Exit
    >Select an option:
    >```

## <a name="time-to-insert-and-delete-documents"></a>插入和删除文档的时间。

1. 选择“1”并按 ENTER 以插入第一个文档 。 该程序将插入第一个文档并返回以下消息。

    ```
    Insert Successful.
    Document for customer with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828' Inserted.
    Press [ENTER] to continue
    ```

1. 再次选择“1”并按 ENTER 以插入第一个文档 。 这次程序将崩溃，并引发异常。 通过查看错误堆栈，可以找到程序失败的原因。 通过从错误堆栈中提取的消息可以看出，遇到了一个未处理的异常“冲突 (409)”

    ```
    Unhandled exception. Microsoft.Azure.Cosmos.CosmosException : Response status code does not indicate success: Conflict (409);
    ```

1. 由于要插入文档，因此需要查看创建文档时返回的常用的[创建文档状态代码][/rest/api/cosmos-db/create-a-document#status-codes]的列表。 此代码的说明是：为新文档提供的 ID 已被现有文档使用。 这是显而易见的，因为刚刚才运行了菜单选项，创建了同一个文档。

1. 进一步深入堆栈中，可以看到这个异常是从第 99 行调用的，而这又是从第 52 行调用的。

    ```
    at Program.CreateDocument1(Container Customer) in C:\WWL\Git\DP-420\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 101   at Program.Main(String[] args) in C:\WWL\Git\DP-420\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 47
    ```

1. 检查第 101 行，可以看到此错误是由 CreateItemAsync 操作引起的，与预期一致。 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
    ```

1. 此外，通过查看 101 到 104 行，可以明显看到此代码没有相关的错误处理。 需要解决这个问题。 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
        Console.WriteLine("Insert Successful.");
        Console.WriteLine("Document for customer with id = '" + customerID + "' Inserted.");
    ```

1. 需要确定错误处理代码应执行的操作。 通过查看[创建文档状态代码][/rest/api/cosmos-db/create-a-document#status-codes]，可以选择为此操作的每个可能的状态代码创建错误处理代码。  在此实验中，将把此列表中的状态代码 403 视为 409。  返回的所有其他状态代码将直接显示系统错误消息。

    > &#128221; 请注意，虽然要编写一个任务代码，在遇到 403 异常时执行相应的任务，但本实验中不会生成这种类型的异常错误。

1. 现在为名为 `CompeteTaskOnCosmosDB` 的 top 函数添加错误处理。 在函数 `Main` 中找到第 45 行附近的 `while` 循环，并用错误处理收捲 `CompeteTaskOnCosmosDB` 的调用。 在这段新代码中首先要注意的是，在 catch 上，将捕获 `CosmosException` 类型的类的异常。  此类包含属性 `StatusCode`，该属性可从 Azure Cosmos DB 服务获取请求完成状态代码。 `StatusCode` 属性的类型为 `System.Net.HttpStatusCode`，因此实际上是在将其值与 .Net [HTTP 状态代码][dotnet/api/system.net.httpstatuscode]中的字段名称比较。  

    ```C#
        try
        {
            await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
        }
        catch (CosmosException e)
        {
                    switch (e.StatusCode.ToString())
                    {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                    }

        }

    ```

1. 保存文件，由于崩溃了，需要再次运行 Menu 程序，因此运行命令：

    ```
    dotnet run
    ```
 
1. 再次选择“1”并按 ENTER 以插入第一个文档 。 这次不会发生故障，而是会获得用户友好的消息。

    ```
    Insert Failed. 
    Response Code (409).
    Can not insert a duplicate partition key, customer with the same ID already exists.
    Press [ENTER] to continue
    ```

1. 在创建文档时需要注意两个特定的异常 403 和 409，同时任何类型的操作还有可能会遇到其他三种通信类型的异常。  这些异常是 429、503 和 408，分别对应“请求过多”、“服务不可用”和“请求超时“。 将添加一些代码来执行以下操作，如果发现任何异常，请等待 10 分钟，然后再次尝试插入文档。  现在在代码中添加更多内容：

    ```C#
                    default:
                        Console.WriteLine(e.Message);
                        break;
    ```


    用于处理异常的代码：


    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
    ```

    > &#128221; 请注意，虽然要编写一个任务代码，在遇到 429、503 或 408 异常时执行相应的任务，但本实验中不会生成相应类型的异常错误。

1. `Main` 函数现在应如下所示：

    ```C#
        public static async Task Main(string[] args)
        {

            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                    try
                    {
                        await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

1. 请注意，仍然通过上面的更改来解决此 `CreateDocument2` 函数的问题。

1. 最后，函数 `DeleteDocument1` 和 `DeleteDocument2` 还需要将以下代码替换为类似于 `CreateDocument1` 函数的适当错误处理代码。 除了使用 DeleteItemAsync 而不是 CreateItemAsync 以外，这些函数的唯一区别在于，[“删除”状态代码][/rest/api/cosmos-db/delete-a-document]与“插入”状态代码不同 。 对于删除，我们只关心 404 状态代码。 现在来了解一下其他情况的 `CompeteTaskOnCosmosDB` 错误处理：


    ```C#
                    case ("NotFound"):
                        Console.WriteLine("Delete Failed. Response Code (404).");
                        Console.WriteLine("Can not delete customer, customer not found.");
                        break;         
    ```

1.  继续操作，添加下面关于处理删除函数的代码，其中包含一些错误处理代码。 同时包含状态代码 429、503 和 508 的重试逻辑  。 需要在 `default` 情况中添加以下代码：

    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;       
    ```

1. 完成所有函数的修复后，多次测试所有菜单选项，以确保应用程序在遇到异常时返回消息并不会崩溃。  如果应用崩溃，则解决错误，然后只重新运行以下命令：

    ```
    dotnet run
    ```


1. 不要速览，完成后，`Main` 代码应与此类似。

    ```C#
        public static async Task Main(string[] args)
        {
            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                    try
                    {
                        await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            case ("NotFound"):
                                Console.WriteLine("Delete Failed. Response Code (404).");
                                Console.WriteLine("Can not delete customer, customer not found.");
                                break; 
                            case ("TooManyRequests"):
                            case ("ServiceUnavailable"):
                            case ("RequestTimeout"):
                                // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                                await Task.Delay(10000); // Wait 10 seconds
                                try
                                {
                                    Console.WriteLine("Try one more time...");
                                    await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                                }
                                catch (CosmosException e2)
                                {
                                    Console.WriteLine("Insert Failed. " + e2.Message);
                                    Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                    break;
                                }
                                break;    
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

## <a name="conclusion"></a>结束语

即使是最初级的开发人员也知道需要向所有代码添加正确的错误处理。 这段代码中的错误处理很简单，但通过它你应该了解了有关 Azure Cosmos DB 异常组件的基础知识，通过这些组件可在代码中创建可靠的错误处理解决方案。


[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[/rest/api/cosmos-db/create-a-document#status-codes]:https://docs.microsoft.com/rest/api/cosmos-db/create-a-document#status-codes
[dotnet/api/system.net.httpstatuscode]:https://docs.microsoft.com/dotnet/api/system.net.httpstatuscode?view=net-6.0
[/rest/api/cosmos-db/delete-a-document]:https://docs.microsoft.com/rest/api/cosmos-db/delete-a-document#status-codes

