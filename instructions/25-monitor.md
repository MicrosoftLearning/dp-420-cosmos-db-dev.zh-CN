---
lab:
  title: 使用 Azure Monitor 分析 Azure Cosmos DB SQL API 帐户
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB SQL API solution
ms.openlocfilehash: 55c1fe4335145a346bb1a197594a5c2d36c7fe7c
ms.sourcegitcommit: fc48219b2f9ba5cbae4b0ba00b22142246bb2195
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/26/2022
ms.locfileid: "145890692"
---
# <a name="use-azure-monitor-to-analyze-an-azure-cosmos-db-sql-api-account"></a>使用 Azure Monitor 分析 Azure Cosmos DB SQL API 帐户

Azure Monitor 是 Azure 中的全栈监视服务，它提供了一套完整的功能来监视 Azure 资源。  Azure Cosmos DB 使用 Azure Monitor 来创建监视数据。  Azure Monitor 捕获 Cosmos DB 的指标和遥测数据。

在此实验室中，你将针对 Azure Cosmos DB 容器运行一个模拟工作负载，并分析该工作负载如何影响 Azure Cosmos DB 帐户。

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
    | **帐户名** | 输入一个全局唯一的名称 |
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *预配的吞吐量* |
    | **应用免费分级折扣** | *`Do Not Apply`* |
    | 限制可在此帐户上预配的总吞吐量 | *取消选中* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用预先创建的现有资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

1. 最小化浏览器窗口，但不要关闭它。 在接下来的步骤中，我们将在启动后台工作负载几分钟后返回 Azure 门户。


## <a name="import-the-microsoftazurecosmos-and-newtonsoftjson-libraries-into-a-net-script"></a>将 Microsoft.Azure.Cosmos 和 Newtonsoft.Json 库导入 .NET 脚本

.NET CLI 包含 [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] 命令，用于从预配置的包源导入包。 .NET 安装使用 NuGet 作为默认包源。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到“25-monitor”文件夹。

1. 打开“25-monitor”文件夹的上下文菜单，然后选择“在集成终端中打开”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为“25-monitor”文件夹的终端。

1. 使用以下命令从 NuGet 添加 [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 包：

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 使用以下命令从 NuGet 添加 [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] 包：

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

## <a name="run-a-script-to-create-the-containers-and-the-workload"></a>运行脚本以创建容器和工作负载

现在，我们已准备好运行工作负载来监视 Azure Cosmos DB 帐户的使用情况。  脚本将在后台运行。 此脚本将创建三个容器，并将一些数据加载到这些容器中。 然后，该脚本会随机运行一些 SQL 查询来模拟访问 Azure Cosmos DB 帐户的多个用户应用程序。 

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到“25-monitor”文件夹。

1. 打开 Program.cs 代码文件。

1. 更新名为 endpoint 的现有变量，将其值设置为前面创建的 Azure Cosmos DB 帐户的终结点 。
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，则 C# 语句将为：private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";。

1. 更新名为 key 的现有变量，将其值设置为前面创建的 Azure Cosmos DB 帐户的键 。

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; 例如，如果你的键为：fDR2ci9QgkdkvERTQ==，则 C# 语句应为：private static readonly string key = "fDR2ci9QgkdkvERTQ==";。

1. 保存 Program.cs 文件。

1. 返回到集成终端。

1. 使用 [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 命令生成并运行项目：

    ```
    dotnet run
    ```
    > &#128221; 此脚本的第一部分将创建三个容器，并将数据加载到其中，这大约需要 2 分钟。 为了模拟某些速率限制事件，该脚本会将预配的吞吐量设置为 400 RU/秒。 然后，你应收到消息“正在创建模拟的后台工作负载”，等待 5-10 分钟，然后转到练习的下一个步骤。 由于 Azure 资源会以异步方式将监视数据上传到 Azure Monitor，因此我们需要等待一小段时间才能开始在 Azure Monitor 指标和见解中获取一些诊断数据。 5-10 分钟后，继续执行下一步。 如果要收集其他诊断数据，无需在 5-10 分钟后停止脚本，只需等待实验室结束即可。

    > &#128221; 你将注意到几个黄色警告，因为编译器检测到脚本以同步方式运行多个操作，并且不等待操作答复。 你可以忽略这些警告，因为这是同时运行多个 SQL 脚本的预期行为。

## <a name="use-azure-monitor-to-analyze-the-azure-cosmos-db-account-usage"></a>使用 Azure Monitor 分析 Azure Cosmos DB 帐户使用情况

在本练习的这一部分中，我们将返回到浏览器，并查看一些 Azure Monitor 见解和指标报告。

### <a name="azure-monitor-metricss-reports"></a>Azure Monitor 指标报告

1. 返回到前面已最小化的打开的浏览器窗口。 如果已将其关闭，请打开一个新窗口，并在 portal.azure.com 下转到 Azure Cosmos DB 帐户页。

1. 在 Azure Cosmos DB 左侧菜单的“监视”下，选择“指标”。 你会注意到“范围”和“指标命名空间”字段预填充了正确的信息。 在下面的步骤中，我们将查看一些“指标”选项以及“添加筛选器”和“应用拆分”功能。

1. 默认情况下，“指标”部分将显示过去 24 小时内的诊断信息。 我们需要更精细地查看在上一步中创建的工作负载中的指标。 在右上角，选择标记为“本地时间: 过去 24 小时(自动)”*_的按钮，随后将显示一个窗口，其中包含多个单选按钮时间范围选项。  选择标记为“过去 30 分钟”_***的单选按钮，然后选择“应用”按钮。 如果需要，可以选择“自定义”单选按钮，并选择一个开始和结束的日期和时间，以获得更精细的粒度。 

1. 现在我们有了诊断图表的时间范围，接下来让我们看看一些指标。 我们将从通用指标开始。 从“指标”下拉菜单中选择“请求单位总数”。 默认情况下，此指标显示为 RU 总和。 或者，可以将“聚合”下拉菜单更改为 avg 或 max。检查这两个聚合后，将其设置回 Sum 以执行接下来的步骤。

1. 此指标使我们可以更好地了解 Azure Cosmos DB 帐户中使用的请求单位数。 然而，当我们的帐户中有多个数据库或容器时，当前图表可能不能帮助我们找到问题所在。 让我们来改变这种情况，我们看看按数据库产生的 RU 使用量。 在图表标题下的菜单中，选择“应用拆分”，在“值”下拉菜单下选择“DatabaseName”，并选择图表的任意位置以接受更改。 现在，“拆分依据 = DatabaseName”按钮将置于图表的正上方。 

1. 好多了，现在我们知道哪个数据库正在执行大部分工作。 尽管这些信息有所帮助，但我们不知道哪个容器正在执行所有工作。  选择“拆分依据 = DatabaseName”按钮以更改 Split 条件，并从“值”下拉菜单中选择“CollectionName”。 很好，现在我们应该有了“customer”和“salesOrder”集合的数据。 此图表只存在一个问题，即 salesOrder 集合存在于两个数据库中：database-v2 和 database-v3，因此，这个值是该集合名称在两个数据库中的聚合。

1. 这应该可以简单地修复，选择“添加筛选器”按钮，在“属性”下拉菜单下选择“DatabaseName”，然后在“值”下选择“database-V3”。

1. 来看看另外两个指标。 我们将编辑现有图表，你也可以根据需要创建新图表。 在该图表上方，选择带有“Azure Cosmos DB 帐户名称”和“请求单位总数”标签的按钮。 从“指标”下拉菜单中选择“请求总数”，请注意，唯一可用的聚合是“Count”。

1. 这里有两个关键筛选器可以帮助我们排查不同类型的问题。 我们添加一个具有 StatusCode 属性的筛选器（请注意，具有不同类型详细信息的类似筛选器是 Status），对于“值”，选择“200”和“429”。 更改 Split 以使用 StatusCode。 请注意，与状态 200 成功请求相比，存在大量的 429 或速率限制请求。 出现 429 异常是因为脚本每秒发送了数千个请求，而我们将预配吞吐量设置为 400 RU/秒。 与成功的请求相比，这种大量的 429 异常在生产环境中是不正常的。在生产环境中，429 异常应很少出现在正常运行的 Azure Cosmos DB 帐户中。  你还可以使用 StautusCode 或 Status 属性对“请求单位总数”进行类似的故障排除

1. 接下来，我们来看看“请求总数”，但将拆分更改为 OperationType。  此属性可帮助我们确定哪些读取或写入操作正在执行大部分工作。 同样，此属性也可以通过类似方式用于“请求单元总数”

1. 就像我们对“请求单位总数”执行的操作一样，试验选择不同的筛选器和拆分选项。 

1. 本练习介绍的最后一个指标是“规范化 RU 消耗量”指标。 将拆分更改为 PartitionKeyRangeId。 这个指标可帮助我们确定哪个分区键范围使用量更大。 该指标可以让我们看到吞吐量向分区键范围倾斜。 继续操作，从“指标”下拉菜单选择该指标。 此图表现在应显示一个非常不正常的系统，该系统达到恒定的 100% 规范化 RU 消耗量。

> &#128221; 如果要一次查看多个图表，请单击图表名称上方的“+ 新建图表”选项。 

> &#128221; 虽然无法直接保存指标，但可以通过单击图表右上角的“固定到仪表板”按钮，创建或使用现有仪表板，并将此图表添加到仪表板。  单击按钮，选择“新建”选项卡，为其指定名称“DP-420 labs”，然后单击“创建并固定”。 要查看专用仪表板，应转到左上角的“门户”菜单，并从“Azure 资源”选项中选择“仪表板”。 首次显示仪表板可能需要几分钟时间。

> &#128221; 共享图表的另一种方法是单击“共享”下拉菜单，然后将其下载为 Excel 文件，或者选择“复制链接”选项。

### <a name="azure-monitor-insights-reports"></a>Azure Monitor 见解报告

我们可能需要花些时间来微调 Azure Monitor 指标诊断报告。  Cosmos DB Insights 提供了对 Azure Cosmos DB 资源的整体性能、故障和操作运行状况的概览。 这些见解图表将是预先生成的图表，与指标图表类似。 让我们来看看其中一些见解。

1. 在 Azure Cosmos DB 左侧菜单的“监视”下，选择“见解”。 你会发现“概述”、“管理”选项等多个选项卡。 我们将介绍其中几个“见解”图表。 第一个选项卡是“概述”选项卡，它汇总了你可以使用的最常见图表。 例如，请求总数、数据和索引使用情况、429 异常和规范化 RU 消耗量等图表。  在上一部分中，我们看到了大部分这些图表。

1. 请注意，在图表顶部，我们可以锁定在“时间范围”，因此，请选择“15”或“30”分钟来评估此练习中的工作负载。

1. 在每个图表的右上角，你会注意到“打开指标资源管理器”*_选项。 接下来，为“请求总数”图表选择“打开指标资源管理器”_*选项。 你会注意到，当你选择此选项时，将转到我们之前查看的“指标”报告。 打开“指标资源管理器”的优点在于，它已为我们生成图表的大部分内容。

1. 通过选择“指标”图表右上角的“X”，返回“见解”页面。

1. 选择“吞吐量”选项卡。这些图表非常适合用于查明吞吐量问题。  请密切注意“规范化 RU 消耗量(%) (按 PartitionKeyRangeID)”图表，该图表可用于检测热分区。

1. 选择“请求”选项卡。这些图表非常适合用于分析帐户遇到的限制事件数（429 与 200），或查看每个操作类型的请求数。  

1. 选择“存储”选项卡。这些图表显示了集合的增长情况，以及数据和索引的使用情况。  

1. 选择“系统”选项卡。如果你的应用程序经常创建、删除或查询帐户元数据，则可能会出现 429 异常。  这些图表可帮助我们确定频繁的元数据访问是否是导致 429 异常的原因。 此外，还可以确定元数据请求的状态。  

### <a name="azure-monitor-insights-reports"></a>Azure Monitor 见解报告

1. 如果程序仍在运行，请返回到 Visual Studio Code 命令终端。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[nuget.org/packages/Newtonsoft.Json/13.0.1]: https://www.nuget.org/packages/Newtonsoft.Json/13.0.1
