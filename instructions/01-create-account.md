---
lab:
  title: 创建 Azure Cosmos DB SQL API 帐户
  module: Module 1 - Get started with Azure Cosmos DB SQL API
ms.openlocfilehash: 4afd40294fba387fcc1ef5dda3196cc623609227
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2022
ms.locfileid: "138024994"
---
# <a name="create-an-azure-cosmos-db-sql-api-account"></a>创建 Azure Cosmos DB SQL API 帐户

在深入了解 Azure Cosmos DB 之前，请务必了解有关创建最常使用资源的基本知识。 在大多数情况下，你需要熟悉创建帐户、数据库、容器和项的流程。 在实际情况下，还应该有几个现成的基本查询，用来测试是否正确创建了所有资源。

在此实验室中，你将使用 SQL API 创建新的 Azure Cosmos DB 帐户。 然后，你将使用数据资源管理器创建一个数据库、一个容器和两个项。 最后，将在数据库中查询已创建的项。

## <a name="create-a-new-azure-cosmos-db-account"></a>创建新的 Azure Cosmos DB 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 在首次预配 Azure Cosmos DB 帐户时，可以选择希望该帐户支持的 API（例如 Mongo API 或 SQL API）。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 在“Azure 服务”类别中，选择“创建资源”，然后选择“Azure Cosmos DB”。

    > &#128161; 或者，展开“&#8801;”菜单，选择“所有服务”，在“数据库”类别中选择“Azure Cosmos DB”，然后选择“创建”。

1. 在“选择 API 选项”窗格中，选择“核心(SQL) - 建议”部分中的“创建”选项。

1. 在“创建 Azure Cosmos DB 帐户”窗格内，查看“基本信息”选项卡。

1. 在“基本信息”选项卡上，为每个设置输入以下值：

    | **设置** | **值** |
    | --: | :-- |
    | **订阅** | 所有资源都必须属于某个资源组。每个资源组都必须属于某个订阅。在这里，使用现有的 Azure 订阅。 |
    | **资源组** | 所有资源都必须属于某个资源组。此处请选择现有资源组或创建新的资源组。 |
    | **帐户名** | 全局唯一的帐户名称。此名称将用作请求的 DNS 地址的一部分。输入任何全局唯一名称。门户将实时检查该名称。 |
    | **位置** | 选择最初托管数据库的地理区域。选择任意可用区域。 |
    | **容量模式** | 选择“预配的吞吐量” |
    | **应用免费分级折扣** | *不应用* |

1. 依次选择“审阅并创建”以导航到“审阅并创建”选项卡，然后选择“创建”。

    > &#128221; Azure Cosmos DB SQL API 帐户可能需要 10-15 分钟才能可供使用。

1. 观察“部署”窗格。 部署完成后，窗格将更新，显示“部署成功”消息。

1. 仍然是在“部署”窗格中，选择“转到资源” 。

## <a name="use-the-data-explorer-to-create-a-new-database-and-container"></a>使用数据资源管理器来创建新的数据库和容器

数据资源管理器将是用于在 Azure 门户中管理 Azure Cosmos DB SQL API 数据库和容器的主要工具。 你将创建一个要在此实验室中使用的基本数据库和容器。

1. 从“Azure Cosmos DB 帐户”窗格内，选择资源菜单中的“数据资源管理器”。

1. 在“数据资源管理器”窗格中，选择“新建容器” 。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定”： 

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | cosmicworks |
    | **在容器之间共享吞吐量** | 请不要选择 |
    | **容器 ID** | products |
    | **分区键** | /categoryId |
    | **容器吞吐量(自动缩放)** | *手动* |
    | **RU/秒** | *400* |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“products”容器节点。  

## <a name="use-the-data-explorer-to-create-new-items"></a>使用数据资源管理器创建新项

数据资源管理器还包含一套功能，可用于在 Azure Cosmos DB SQL API 容器中查询、创建和管理项。 你将在数据资源管理器中使用原始 JSON 创建两个基本项。

1. 在“数据资源管理器”窗格中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“项”。

1. 同样在“数据资源管理器”窗格中，从命令栏中选择“新建项”。 在编辑器中，将占位符 JSON 项替换为以下内容：

    ```
    {
      "categoryId": "4F34E180-384D-42FC-AC10-FEC30227577F",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. 从命令栏中选择“保存”，添加第一个 JSON 项：

1. 返回“项”选项卡，从命令栏中选择“新建项”。 在编辑器中，将占位符 JSON 项替换为以下内容：

    ```
    {
      "categoryId": "75BF1ACB-168D-469C-9AA3-1FD26BB4EA4C",
      "categoryName": "Bikes, Touring Bikes",
      "sku": "BK-T18Y-44",
      "name": "Touring-3000 Yellow, 44",
      "price": 742.35
    }
    ```

1. 从命令栏中选择“保存”，添加第二个 JSON 项：

1. 在“项”选项卡中，观察“项”窗格中的两个新项。

## <a name="use-the-data-explorer-to-issue-a-basic-query"></a>使用数据资源管理器发出基本查询

最后，数据资源管理器有一个内置查询编辑器，用于发出查询、观察结果，并按每秒请求单位数（RU/秒）来衡量影响。

1. 在“数据资源管理器”窗格中，选择“新建 SQL 查询”。

1. 在“查询”选项卡中，选择“执行查询”以查看选择不包含任何筛选器的所有项的标准查询。

1. 删除编辑器区域的内容。

1. 在“查询”选项卡中，将占位符查询替换为以下内容：

    ```
    SELECT * FROM products p WHERE p.price > 500
    ```

    > &#128221; 此查询将选择 price 大于 $500 的所有项。

1. 选择“执行查询”。

1. 观察查询结果，其中应包括单个 JSON 项及其所有属性。

1. 在“查询”选项卡中，选择“查询统计信息”。

1. 同样在“查询”选项卡中，观察“查询统计信息”部分中“请求费用”字段的值。

    > &#128221; 通常，当容器大小较小时，此简单查询的请求费用介于 2 和 3 RU/秒之间。

1. 关闭 Web 浏览器窗口或选项卡。
