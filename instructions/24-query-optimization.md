---
lab:
  title: 优化用于查询的 Azure Cosmos DB SQL API 容器索引策略
  module: Module 10 - Optimize query performance in Azure Cosmos DB SQL API
ms.openlocfilehash: 3556b5f9b8a3129d92dcccf65a54d6adf1ccbf32
ms.sourcegitcommit: c3778722311b55568f083480ecc69c9b3e837a18
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/19/2022
ms.locfileid: "138025012"
---
# <a name="optimize-an-azure-cosmos-db-sql-api-containers-indexing-policy-for-a-query"></a>优化用于查询的 Azure Cosmos DB SQL API 容器索引策略

在针对 Azure Cosmos DB SQL API 帐户进行规划时，了解最常见的查询有助于优化索引策略，尽可能提高查询性能。

在此实验中，将通过数据资源管理器，使用默认索引策略和包含复合索引的索引策略来测试 SQL 查询。

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
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *无服务器* |

    > &#128221; 你的实验环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”窗格中，选择“新建容器”。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定” ：

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 新建 &vert; cosmicworks |
    | **容器 ID** | products |
    | **分区键** | /categoryId |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“products”容器节点。

1. 在资源边栏选项卡中，导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="seed-your-azure-cosmos-db-sql-api-account-with-sample-data"></a>使用示例数据设置 Azure Cosmos DB SQL API 种子帐户

你将使用命令行实用工具来创建 cosmicworks 数据库和 products 容器。  然后，该工具将创建一组项，你将使用终端窗口中运行的更改源处理器来观察它们。

1. 在 Visual Studio Code 中，打开“终端”菜单，然后选择“拆分终端”，以与现有实例并排打开新的终端。  

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

## <a name="execute-sql-queries-and-measure-their-request-unit-charge"></a>执行 SQL 查询并测量其请求单位费用

在修改索引策略之前，先运行几个示例 SQL 查询，来获取以 RU 表示的基线请求单位费用。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，然后选择在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“新 SQL 查询”。

1. 选择“执行查询”以运行默认查询：

    ```
    SELECT * FROM c
    ```

1. 观察查询结果。 选择“查询统计信息”，查看以 RU 为单位的请求单位费用。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询将返回 name 等同于 HL Headset 的所有文档：

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p    
    ```

1. 选择“执行查询”。

1. 观察查询的结果和统计信息。 请求单位费用与第一个查询几乎相同。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询将返回 name 等同于 HL Headset 的所有文档：

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC
    ```

1. 选择“执行查询”。

1. 观察查询的结果和统计信息。 由于 ORDER BY 子句，请求单位费用增加了。

## <a name="create-a-composite-index-in-the-indexing-policy"></a>在索引策略中创建复合索引

现在，如果使用多个属性对项进行排序，将需要创建一个复合索引。 在此任务中，将创建一个复合索引，以便按 categoryName 对项进行排序，然后按其实际名称对项进行排序。

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“新 SQL 查询”。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询先按 categoryName 降序对结果进行排序，然后按 price 升序排序 ：

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. 选择“执行查询”。

1. 查询应会失败，并显示错误：“order by”查询没有相应的复合索引可用于提供相应服务。

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“设置”。

1. 在“设置”选项卡中，导航到“索引策略”部分。

1. 观察默认索引策略：

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

1. 将索引策略替换为修改后的 JSON 对象，然后保存更改：

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“新 SQL 查询”。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询先按 categoryName 降序对结果进行排序，然后按 price 升序排序 ：

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. 选择“执行查询”。

1. 观察查询的结果和统计信息。 现在复合索引已有，请求单位费用应会减少。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询先按 categoryName 降序对结果进行排序，然后按 name 升序排序，最后按 price 升序排序  ：

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. 选择“执行查询”。

1. 观察查询的结果和统计信息。 请求单位费用将再次增加，因为查询复杂了以及缺少支持的复合索引。

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后再次选择“设置”。

1. 在“设置”选项卡中，导航到“索引策略”部分。

1. 将索引策略替换为修改后的 JSON 对象，然后保存更改：

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ],
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/name",
            "order": "ascending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“新 SQL 查询”。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询先按 categoryName 降序对结果进行排序，然后按 name 升序排序，最后按 price 升序排序  ：

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. 选择“执行查询”。

1. 观察查询的结果和统计信息。 现在复合索引已有，请求单位费用应会降低。

1. 关闭 Web 浏览器窗口或选项卡。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
