---
lab:
  title: 通过门户查看 Azure Cosmos DB SQL API 容器的默认索引策略
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: a5918d41746f82da08d66c486aa53f782d9e8e17
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2022
ms.locfileid: "138024943"
---
# <a name="review-the-default-index-policy-for-an-azure-cosmos-db-sql-api-container-with-the-portal"></a>通过门户查看 Azure Cosmos DB SQL API 容器的默认索引策略

Azure Cosmos DB 中的每个容器都有一个索引策略，用于指示服务如何为容器中的项编制索引。 默认情况下，此索引策略会索引每个项的每一个属性。 使用默认索引策略可以快速开始使用 Azure Cosmos DB，因为不需要在项目开始时考虑索引编制、性能和管理方面的问题。

在此实验中，将使用数据资源管理器来观察和操作一些容器的默认索引策略。

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>创建 Azure Cosmos DB SQL API 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 在首次预配 Azure Cosmos DB 帐户时，可以选择希望该帐户支持的 API（例如 Mongo API 或 SQL API） 。 完成 Azure Cosmos DB SQL API 帐户预配后，可以检索终结点和密钥，并使用它们通过 Azure SDK for .NET 或所选择的任何其他 SDK 连接到 Azure Cosmos DB SQL API 帐户。

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
    | **应用免费分级折扣** | *不应用* |

    > &#128221; 你的实验环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格 。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="seed-the-azure-cosmos-db-sql-api-account-with-data"></a>使用数据为 Azure Cosmos DB SQL API 帐户设定种子

[Cosmicworks][nuget.org/packages/cosmicworks] 命令行工具将示例数据部署到任何 Azure Cosmos DB SQL API 帐户。 该工具是开源工具，可通过 NuGet 获得。 将此工具安装到 Azure Cloud Shell，然后使用它来设定数据库种子。

1. 启动 Visual Studio Code。

1. 在 Visual Studio Code 中，打开“终端”菜单，然后选择“新建终端”以打开新的终端实例  。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅 [Visual Studio Code 入门指南][code.visualstudio.com/docs/getstarted]

1. 在计算机上安装可全局使用的 [cosmicworks][nuget.org/packages/cosmicworks] 命令行工具。

    ```
    dotnet tool install --global cosmicworks
    ```
  
    > &#128161; 此命令可能需要几分钟时间才能完成。 如果你过去已经安装了此工具的最新版本，此命令将输出警告消息（*工具“cosmicworks”已安装）。

1. 使用以下命令行选项运行 cosmicworks 以设置 Azure Cosmos DB 种子帐户：

    | **选项** | **值** |
    | ---: | :--- |
    | --endpoint | 之前在本实验室中复制的终结点值 |
    | --key | 之前在本实验室中复制的键值 |
    | --datasets | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，密钥为：fDR2ci9QgkdkvERTQ==，则命令为：``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. 等待 cosmicworks 命令使用数据库、容器和项完成帐户填充。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

## <a name="view-and-manipulate-the-default-indexing-policy"></a>查看和操作默认索引策略

通过代码、门户或工具创建容器时，如果不另行指定，索引策略将设置为智能默认值。 将观察该默认索引策略，并更改该策略。

1. 在 Web 浏览器中，转到 Azure 门户 (``portal.azure.com``)。

1. 选择“资源组”，然后选择在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源 。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“SQL API”导航树中观察新“products”容器节点。

1. 选择“SQL API”导航树中的“products”容器节点，然后选择“新建 SQL 查询”。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询将返回 name 等同于 HL Headset 的所有文档：

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. 选择“执行查询”。

1. 观察查询结果。

1. 在“查询”选项卡中，选择“查询统计信息”。

1. 同样在“查询”选项卡中，观察“查询统计信息”部分中“请求费用”字段的值。

    > &#128221; 所有路径当前已编制索引，所以此查询应相对高效。

1. 在 SQL API 导航树的 products 容器节点内，选择“缩放和设置”  。

1. 在“索引策略”部分中观察默认索引策略：

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

    > &#128221; 此默认策略将为所有可能的路径编制索引，但 _etag 除外。

1. 在编辑器中，替换索引策略的内容，以仅为 /price 路径编制 索引：

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/price/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        }
      ]
    }
    ```

1. 选择“保存”以保存更改。

1. 选择“新建 SQL 查询”。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询将返回 name 等同于 HL Headset 的所有文档：

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. 选择“执行查询”。

1. 观察查询结果。

1. 在“查询”选项卡中，选择“查询统计信息”。

1. 同样在“查询”选项卡中，观察“查询统计信息”部分中“请求费用”字段的值。

    > &#128221; 由于 name 属性未编制索引，请求费用增加了。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询返回其中 price 大于 $3,000 的所有文档 ：

    ```
    SELECT * FROM p WHERE p.price > 3000
    ```

1. 选择“执行查询”。

1. 观察查询结果。

1. 在“查询”选项卡中，选择“查询统计信息”。

1. 同样在“查询”选项卡中，观察“查询统计信息”部分中“请求费用”字段的值。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
