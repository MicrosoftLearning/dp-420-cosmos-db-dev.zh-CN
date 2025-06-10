---
lab:
  title: 优化 Azure Cosmos DB for NoSQL 容器的索引策略以执行特定查询
  module: Module 10 - Optimize query and operation performance in Azure Cosmos DB for NoSQL
---

# 优化用于查询的 Azure Cosmos DB for NoSQL 容器索引策略

在针对 Azure Cosmos DB for NoSQL 帐户进行规划时，了解最常见的查询有助于优化索引策略，尽可能提高查询性能。

在此实验中，将通过数据资源管理器，使用默认索引策略和包含复合索引的索引策略来测试 SQL 查询。

## 创建 Azure Cosmos DB for NoSQL 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 在首次预配 Azure Cosmos DB 帐户时，可以选择希望该帐户支持的 API（例如 Mongo API 或 NoSQL API）。 完成 Azure Cosmos DB for NoSQL 帐户预配后，可以检索终结点和密钥，并使用它们通过 Azure SDK for .NET 或所选择的任何其他 SDK 连接到 Azure Cosmos DB for NoSQL 帐户。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭证登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB for NoSQL”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | 工作负载类型**** | **学习** |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **帐户名** | 输入全局唯一名称 |
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *预配的吞吐量* |
    | **应用免费分级折扣** | *不应用* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”窗格中，展开“新建容器”，然后选择“新建数据库”。

1. 在“新建数据库”弹出窗口中，为每个设置输入以下值，然后选择“确定”：

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | *``cosmicworks``* |
    | 预配吞吐量 | enabled |
    | **数据库吞吐量** | **手动** |
    | **数据库每秒所需 RU 数** | ``1000`` |

1. 返回到“数据资源管理器”窗格，观察层次结构中的“cosmicworks”数据库节点。

1. 在“数据资源管理器”窗格中，选择“新建容器” 。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定” ：

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 使用现有 &vert; cosmicworks |
    | **容器 ID** | *``products``* |
    | **分区键** | *``/category/name``* |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“products”容器节点。

1. 在资源边栏选项卡中，导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 请注意“主连接字符串”**** 字段。 你将在此练习的稍后部分使用此连接字符串值。

1. 打开 **Visual Studio Code**。

## 使用示例数据为 Azure Cosmos DB for NoSQL 帐户设定种子

你将使用命令行实用工具来创建 cosmicworks 数据库和 products 容器。 然后，该工具将创建一组项，你将使用终端窗口中运行的更改源处理器来观察它们。

1. 在 **Visual Studio Code** 中，打开“终端”**** 菜单，然后选择“新建终端”**** 以打开新的终端。

1. 在计算机上安装可全局使用的 [cosmicworks][nuget.org/packages/cosmicworks] 命令行工具。

    ```
    dotnet tool install --global CosmicWorks --version 2.3.1
    ```

    > &#128161; 此命令可能需要几分钟时间才能完成。 如果你过去已经安装了此工具的最新版本，此命令将输出警告消息（*工具 "cosmicworks" 已安装）。

1. 使用以下命令行选项运行 cosmicworks 以设置 Azure Cosmos DB 种子帐户：

    | **选项** | 值 |
    | ---: | :--- |
    | **-c** | *之前在本实验室中勾选的连接字符串值* |
    | **--number-of-employees** | *cosmicworks 命令会用员工容器和产品容器分别填充数据库，默认包含 1000 个和 200 个项目，除非另有指定。* |

    ```powershell
    cosmicworks -c "connection-string" --number-of-employees 0 --disable-hierarchical-partition-keys
    ```

    > &#128221; 例如，如果终结点为：https&shy;://dp420.documents.azure.com:443/，密钥为：fDR2ci9QgkdkvERTQ==，则命令为：``cosmicworks -c "AccountEndpoint=https://dp420.documents.azure.com:443/;AccountKey=fDR2ci9QgkdkvERTQ==" --number-of-employees 0 --disable-hierarchical-partition-keys``

1. 等待 cosmicworks 命令完成对帐户的数据库、容器和项的填充。

1. 关闭集成终端。

1. 关闭 Visual Studio Code**** 并返回浏览器。

## 执行 SQL 查询并测量其请求单位费用

在修改索引策略之前，先运行几个示例 SQL 查询，来获取以 RU 表示的基线请求单位费用。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“新 SQL 查询”。

1. 选择“执行查询”以运行默认查询：

    ```
    SELECT * FROM c
    ```

1. 观察查询结果。 选择“查询统计信息”，查看以 RU 为单位的请求单位费用。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询将从所有文档返回所有三个值：

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p    
    ```

1. 选择“执行查询”。

1. 观察查询的结果和统计信息。 请求单位费用与第一个查询几乎相同。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询将从所有文档按 categoryName 返回所有三个值：

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC
    ```

1. 选择“执行查询”。

1. 观察查询的结果和统计信息。 由于 ORDER BY 子句，请求单位费用增加了。

## 在索引策略中创建复合索引

现在，如果使用多个属性对项进行排序，将需要创建一个复合索引。 在此任务中，将创建一个复合索引，以便按 categoryName 对项进行排序，然后按其实际名称对项进行排序。

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“新 SQL 查询”。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，先按**类别**降序排序，再按**价格**升序排列结果：

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
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
            "path": "/category",
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
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.price ASC
    ```

1. 选择“执行查询”。

1. 观察查询的结果和统计信息。 此时，由于查询已完成，因此可以再次查看 RU 费用。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，先按**类别**降序排序，再按**名称**升序排序结果，最后按**价格**升序排序：

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.name ASC,
        p.price ASC
    ```

1. 选择“执行查询”。

1. 查询应会失败，并显示错误：“order by”查询没有相应的复合索引可用于提供相应服务。

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
            "path": "/category",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ],
        [
          {
            "path": "/category",
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

1. 创建一个新的 SQL 查询，先按**类别**降序排序，再按**名称**升序排序结果，最后按**价格**升序排序：

    ```
    SELECT 
        p.name,
        p.category,
        p.price
    FROM
        products p
    ORDER BY
        p.category DESC,
        p.name ASC,
        p.price ASC
    ```

1. 选择“执行查询”。

1. 观察查询的结果和统计信息。 此时，由于查询已完成，因此可以再次查看 RU 费用。

1. 关闭 Web 浏览器窗口或选项卡。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
