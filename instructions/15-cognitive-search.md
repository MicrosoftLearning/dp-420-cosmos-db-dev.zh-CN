---
lab:
  title: 使用 Azure 认知搜索和 Azure Cosmos DB SQL API 来搜索数据
  module: Module 7 - Integrate Azure Cosmos DB SQL API with Azure services
ms.openlocfilehash: e61608396e31d7892168cbf29086cb16be525087
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2022
ms.locfileid: "138024908"
---
# <a name="search-data-using-azure-cognitive-search-and-azure-cosmos-db-sql-api"></a>使用 Azure 认知搜索和 Azure Cosmos DB SQL API 来搜索数据

Azure 认知搜索将搜索引擎作为一项服务与和人工智能功能的深度集成相结合，来丰富搜索索引中的信息。

在此实验中，你将构建一个 Azure 认知搜索索引，该索引会自动为 Azure Cosmos DB SQL API 容器中的数据编制索引，并使用 Azure 认知服务翻译器功能来丰富数据。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>创建 Azure Cosmos DB SQL API 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 在首次预配 Azure Cosmos DB 帐户时，可以选择希望该帐户支持的 API（例如 Mongo API 或 SQL API）。 完成 Azure Cosmos DB SQL API 帐户预配后，可以检索终结点和密钥，并使用它们通过 Azure SDK for .NET 或所选择的任何其他 SDK 连接到 Azure Cosmos DB SQL API 帐户。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB SQL API”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **帐户名** | 输入一个全局唯一名称 |
    | **位置** | 选择任意可用区域 |
    | **容量模式** | *无服务器* |

    > &#128221; 你的实验环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用预先创建的现有资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

    1. 记录“主链接字符串”字段的值。 你将在此练习的稍后部分使用此连接字符串值。

1. 从资源菜单中选择“数据资源管理器”。

1. 在“数据资源管理器”窗格中，选择“新建容器” 。

1. 在“新建容器”弹出窗口中为每个设置输入以下值，然后选择“确定”： 

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 新建 &vert; cosmicworks |
    | **容器 ID** | products |
    | **分区键** | /categoryId |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“products”容器节点。  

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="seed-your-azure-cosmos-db-sql-api-account-with-sample-data"></a>使用示例数据设置 Azure Cosmos DB SQL API 种子帐户

你将使用命令行实用工具来创建 cosmicworks 数据库和 products 容器。  然后，该工具将创建一组可使用在终端窗口中运行的更改源处理器进行观察的项。

1. 在 Visual Studio Code 中，打开“终端”菜单，然后选择“拆分终端”，以与现有实例并排打开新的终端  。

1. 在计算机上安装可全局使用的 [cosmicworks][nuget.org/packages/cosmicworks] 命令行工具。

    ```
    dotnet tool install --global cosmicworks
    ```

    > &#128161; 此命令可能需要几分钟时间才能完成。 如果你过去已经安装了此工具的最新版本，此命令将输出警告消息（*工具 'cosmicworks' 已安装'）。

1. 使用以下命令行选项运行 Cosmos 设置 Azure Cosmos DB 种子帐户：

    | **选项** | **值** |
    | ---: | :--- |
    | --endpoint | 之前在本实验中复制的终结点值 |
    | --key | 之前在本实验中复制的键值 |
    | --datasets | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; 如，如果终结点为 https&shy;://dp420.documents.azure.com:443/，密钥为：fDR2ci9QgkdkvERTQ==，则命令为：``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product`` 

1. 等待 cosmicworks 命令使用数据库、容器和项完成帐户填充。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

## <a name="create-azure-cognitive-search-resource"></a>创建 Azure 认知搜索资源

继续完成此练习之前，需要先创建新的 Azure 认知搜索实例。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“+ 创建资源”，搜索“认知搜索”，然后使用以下设置创建新的“Azure 认知搜索”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **名称** | 输入全局唯一名称 |
    | **位置** | 选择任何可用区域 |
    | **定价层** | *免费* |

    > &#128221; 你的实验环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的“Azure 认知搜索”帐户资源。

## <a name="build-indexer-and-index-for-azure-cosmos-db-sql-api-data"></a>Azure Cosmos DB SQL API 数据的生成索引器和索引

将创建一个索引器，该索引器按小时索引特定 Azure Cosmos DB SQL API 容器中数据的子集。

1. 从“Azure 认知搜索”资源边栏选项卡中，选择“导入数据” 。

1. 在“导入数据”向导的“连接到数据”步骤中，选择“数据源”列表中的“Azure Cosmos DB”   。

1. 将数据源配置为以下设置，并将所有其余设置保留为其默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **数据源名称** | products-cosmossql-source |
    | **连接字符串** | 之前创建的 Azure Cosmos DB SQL API 帐户的 _连接字符串_ |
    | **数据库** | cosmicworks |
    | **集合** | products |

1. 在“查询”字段中，输入以下 SQL 查询，以创建容器中数据子集的一个具体化视图：

    ```
    SELECT 
        p.id, 
        p.categoryId, 
        p.name, 
        p.price,
        p._ts
    FROM 
        products p 
    WHERE 
        p._ts > @HighWaterMark 
    ORDER BY 
        p._ts
    ```

1. 选中“按 _ts 对查询结果排序”复选框。

    > &#128221; 此复选框让 Azure 认知搜索功能知道查询按 _ts 字段对结果进行排序。 这种类型的排序启用增量进度跟踪。 如果索引器失败，可以直接从原先的 _ts 值处开始，因为结果按时间戳排序。

1. 选择“下一步：添加认知技能”。

1. 单击“下一步：自定义目标索引”。

1. 在向导的“自定义目标索引”步骤中，使用以下设置配置索引，将所有剩余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **索引名称** | products-index |
    | **键** | *id* |

1. 在字段表中，使用下表为每个字段配置“可检索”、“可筛选”、“可排序”、“可分面”和“可搜索”选项    ：

    | **字段** | **可检索** | **可筛选** | **可排序** | **可分面** | **可搜索** |
    | ---: | :---: | :---: | :---: | :---: | :---: |
    | **id** | &#10004; | &#10004; | &#10004; | | |
    | categoryId | &#10004; | &#10004; | &#10004; | &#10004; | |
    | name | &#10004; | &#10004; | &#10004; | | &#10004;（英语 - Microsoft） |
    | **price** | &#10004; | &#10004; | &#10004; | &#10004; | |

1. 选择“下一步:创建索引器”。

1. 在向导的“创建索引器”步骤中，使用以下设置配置索引器，将所有剩余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **名称** | products-cosmosdb-indexer |
    | **计划** | *每小时* |

1. 选择“提交”以创建数据源、索引和索引器。

    > &#128221; 创建第一个索引器后，可能需要关闭一个调查弹出窗口。

1. 在“Azure 认知搜索”资源边栏选项卡中，导航到“索引器”选项卡，观察第一个索引操作的结果 。

1. 请等待 products-cosmosdb-indexer 索引器的状态为“成功”，然后再继续执行此任务 。

    > &#128221; 如果未自动更新边栏选项卡，可能需要使用“刷新”选项来更新边栏选项卡。

1. 导航到“索引”选项卡，然后选择“products-index”索引 。

## <a name="validate-index-with-example-search-queries"></a>使用示例搜索查询来验证索引

现在，Azure Cosmos DB SQL API 数据的具体化视图已在搜索索引中，可以执行一些基本查询来利用 Azure 认知搜索中的功能。

> &#128221; 此实验的目的不是教授 Azure 认知搜索语法。 这些查询经过特别设计，专门用来展示搜索索引和引擎中提供的一些功能。

1. 在“products-index”&vert;“索引”窗格中，选择“搜索”以启动默认搜索查询，这会返回所有使用 \*（通配符）运算符的可能结果   。

1. 请注意，此搜索查询返回所有可能的结果。

1. 在“查询字符串”编辑器中，输入以下查询，然后选择“搜索” ：

    ```
    touring 3000
    ```

1. 请注意，此搜索查询返回包含词语“旅行”或“3000”的结果，并对同时包含这两个词的结果给予更高的分数 。 然后按 @search.score 字段的降序对结果进行排序。

1. 在“查询字符串”编辑器中，输入以下查询，然后选择“搜索” ：

    ```
    red&$count=true
    ```

1. 请注意，此搜索查询返回的结果带有词语“红色”，但现在还显示了一个元数据字段，指示结果的总数，即使这些结果未包含在同一页面中。

1. 在“查询字符串”编辑器中，输入以下查询，然后选择“搜索” ：

    ```
    blue&$count=true&$top=6
    ```

1. 请注意，此搜索查询一次只返回六个结果，即使服务器端有更多匹配项也是如此。

1. 在“查询字符串”编辑器中，输入以下查询，然后选择“搜索” ：

    ```
    mountain&$count=true&$top=25&$skip=50
    ```

1. 请注意，此搜索查询跳过前 50 个结果，返回了 25 个结果。 如果这是客户端应用程序中的分页视图，可以推断出这是结果的第三个“页”。

1. 在“查询字符串”编辑器中，输入以下查询，然后选择“搜索” ：

    ```
    touring&$count=true&$filter=price lt 500
    ```

1. 请注意，此搜索查询仅返回数值价格字段的值小于 500 的结果。

1. 在“查询字符串”编辑器中，输入以下查询，然后选择“搜索” ：

    ```
    road&$count=true&$top=15&facet=price,interval:500
    ```

1. 请注意，此搜索查询返回一个分面数据集合，这些数据指示属于每个类别的项数，即使这些项并非全部位于当前结果页中。 此示例中，匹配项以 500 为划分区间分解为各数值价格类别。 这通常用于填充客户端应用程序中的筛选器和进行导航辅助。

1. 关闭 Web 浏览器窗口或选项卡。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
