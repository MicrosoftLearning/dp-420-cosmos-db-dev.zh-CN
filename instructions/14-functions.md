---
lab:
  title: 使用 Azure Functions 处理 Azure Cosmos DB for NoSQL 数据
  module: Module 7 - Integrate Azure Cosmos DB for NoSQL with Azure services
---

# 使用 Azure Functions 处理 Azure Cosmos DB for NoSQL 数据

Azure Functions 的 Azure Cosmos DB 触发器是使用更改源处理器实现的。 利用这些知识，可以在 Azure Cosmos DB for NoSQL 容器中创建响应创建和更新操作的函数。 如果手动实现了一个更改源处理器，则 Azure Functions 的设置与此类似。

在此实验室中，你将创建一个函数应用及其所有必要的资源，用于监视其中检测到的每个操作的数据库和输出日志信息。

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
    | **容量模式** | *无服务器* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 请注意“URI”**** 字段。 稍后在本练习中将用到此终结点值。

    1. 请注意“主键”**** 字段。 稍后在本练习中将用到此键值。

1. 从资源菜单中选择“数据资源管理器”。

1. 在“数据资源管理器”窗格中，展开“新建容器”，然后选择“新建数据库”。

1. 在“新建数据库”弹出窗口中，为每个设置输入以下值，然后选择“确定”：

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | *``cosmicworks``* |

1. 返回到“数据资源管理器”窗格，观察层次结构中的“cosmicworks”数据库节点。

1. 在“数据资源管理器”窗格中，选择“新建容器” 。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定” ：

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 使用现有 &vert; cosmicworks |
    | **容器 ID** | *``products``* |
    | **分区键** | *``/categoryId``* |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“products”容器节点。

1. 在“数据资源管理器”窗格中，再次选择“新建容器”********。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定” ：

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 使用现有 &vert; cosmicworks |
    | **容器 ID** | *``productslease``* |
    | **分区键** | *``/id``* |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“productslease”容器节点。************

1. 转到 Azure 门户的主页。

## 创建 Application Insight

在创建 Azure Funtion Application 之前，需要启用 Azure Application Insight，以便可以监视应用程序功能****。 Application Insight 首先需要 Log Analytics 工作区**。

1. 在搜索框中搜索“Log Analytics 工作区”****。

1. 选择“+创建”以创建新的 Log Analytics 工作区******。

1. 在“Log Analytics 工作区”对话框中，为每个设置输入以下值，然后选择“查看+创建”，然后选择“创建”************：

    | **设置** | 值 |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **Name** | *``lab14laworkspace``* |
    | **位置** | 选择任何可用区域 |

1. 创建 Log Analytics 工作区后，在搜索框中搜索“Application Insights”******。

1. 选择“+创建”以创建新的 Application Insight******。

1. 在“Application Insights”对话框中，为每个设置输入以下值，然后选择“查看+创建”，然后选择“创建”************：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅（两个条目）** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **Name** | *``lab14appinsight``* |
    | **位置** | 选择任何可用区域 |
    | **Log Analytics 工作区** | lab14laworkspace** |

现在应该能够监视应用程序函数。

## 创建 Azure Function 应用和 Azure Cosmos DB 触发的函数

在开始编写代码之前，需要使用创建向导创建 Azure Functions 资源及其相关资源（Application Insights、存储）。

1. 选择“**+ 创建资源**”，搜索“*函数*”，然后创建新的“**函数应用**”帐户资源。 选择“**消耗**”作为托管选项，并使用以下设置来设置应用，并将所有剩余设置保留为其默认值：

    | **设置** | 值 |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **名称** | 输入全局唯一名称 |
    | **发布** | *代码* |
    | **运行时堆栈** | *.NET* |
    | **版本** | *8 (LTS) 进程内* |
    | **区域** | 选择任何可用区域 |
    | **存储帐户** | *新建存储帐户* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 **Azure Functions** 帐户资源，然后在概述页面中选择“**创建函数**”。

1. 在“创建函数”弹出窗口中，使用以下设置创建一个新函数，并将所有剩余设置保留为默认值****：

    | **设置** | **值** |
    | ---: | :--- |
    | **Select a template** | Azure Cosmos DB 触发器 |
    | **函数名** | *``ItemsListener``* |
    | Cosmos DB 帐户连接**** | 选择“新建”&vert; 选择“Azure Cosmos DB 帐户”&vert; 选择之前创建的 Azure Cosmos DB 帐户****** |
    | **数据库名称** | *``cosmicworks``* |
    | 集合名称 | *``products``* |
    | **租约的集合名称** | *``productslease``* |
    | **不存在租约集合时创建一个** | *否* |

## 在 .NET 中实现函数代码

前面创建的函数是在门户内编辑的 C# 脚本。 现在，你将使用门户编写一个简短的函数以输出在容器中插入或更新的任何项的唯一标识符。

1. 在“**ItemsListener**&vert;**代码 + 测试**”窗格中，导航到 **run.csx** 脚本的编辑器并删除其内容。

1. 在编辑器区域中，引用 Microsoft.Azure.DocumentDB.Core 库****：

    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    ```

1. 为 System、System.Collections.Generic, 和 [Microsoft.Azure.Documents][docs.microsoft.com/dotnet/api/microsoft.azure.documents] 命名空间添加 using 块********：

    ```
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    ```

1. 创建一个名为 Run 的新静态方法，该方法具有两个参数****：

    1. 类型为 IReadOnlyList\<\>、名为 input 的参数，其泛型类型为 [Document][docs.microsoft.com/dotnet/api/microsoft.azure.documents.document]********。

    1. 名为 log 类型为 ILogger 的参数********。

    ```
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
    }
    ```

1. 在 Run 方法中，调用 log 变量的 LogInformation 方法，传入一个字符串，该字符串计算当前批处理中的项目数************：

    ```
    log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}"); 
    ```

1. 仍然在 Run 方法中，创建一个 foreach 循环，循环访问 input 变量，并使用变量 item 来表示 Document 类型的实例****************：

    ```
    foreach(Document item in input)
    {
    }
    ```

1. 在 Run 方法的 foreach 循环中，调用 log 变量的 LogInformation 方法，传递一个字符串，该字符串打印 item 变量的 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id] 属性****************：

    ```
    log.LogInformation($"Detected Operation:\t{item.Id}");
    ```

1. 完成后，代码文件现在应包含：
  
    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
        log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}");
    
        foreach(Document item in input)
        {
            log.LogInformation($"Detected Operation:\t{item.Id}");
        }
    }
    ```

1. 展开页面底部的“**日志**”部分，展开“**App Insights 日志**”，然后选择“**文件系统日志**”以连接到当前函数的流式处理日志。

    > &#128161; 连接到流日志服务可能需要几秒钟的时间。 连接后，会在日志输出中看到一条消息。

1. 保存当前函数代码****。

1. 观察 C# 代码编译的结果。 应会在日志输出的末尾看到“编译成功”消息****。

    > &#128221; 日志输出中可能会显示警告消息。 这些警告不会影响此实验。

1. 最大化日志部分，以扩展输出窗口和最大程度填充可用空间****。

    > &#128221; 将使用另一个工具在 Azure Cosmos DB for NoSQL 容器中生成项。 生成项后，将返回此浏览器窗口查看输出。 不要提前关闭浏览器窗口。

## 使用示例数据为 Azure Cosmos DB for NoSQL 帐户设定种子

你将使用命令行实用工具来创建 cosmicworks 数据库和 products 容器。 然后，该工具将创建一组项，你将使用终端窗口中运行的更改源处理器来观察它们。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅 [Visual Studio Code 入门指南][code.visualstudio.com/docs/getstarted]

1. 在 Visual Studio Code 中，打开“终端”菜单，然后选择“新建终端”以打开新的终端实例。

1. 在计算机上安装可全局使用的 [cosmicworks][nuget.org/packages/cosmicworks] 命令行工具。

    ```
    dotnet tool install cosmicworks --global --version 1.*
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

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

1. 返回当前打开的浏览器窗口或选项卡，并展开 Azure Functions 日志部分。

1. 查看函数的日志输出。 终端使用更改源为收到的每个更改输出一条“检测到操作”消息****。 操作按 ~100 个操作一组分批成组。

1. 关闭 Web 浏览器窗口或选项卡。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.documents]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.document]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.document
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id
