---
lab:
  title: 从恢复点恢复数据库或容器
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB for NoSQL solution
---

# 从恢复点恢复数据库或容器 

Azure 自动执行数据的加密备份。 这些备份以两种模式进行：定期备份模式和连续备份模式********。

在此实验室中，你将使用连续备份模式执行备份**** 和还原****。 首先，创建一个 Azure Cosmos DB 帐户。 然后，创建两个容器，并在其中添加一些文档。 接下来，将更新这些容器中的一些文档。 最后，将帐户还原到每次删除前的某个点。

## 创建 Azure Cosmos DB for NoSQL 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 在首次预配 Azure Cosmos DB 帐户时，可以选择希望该帐户支持的 API（例如 Mongo API 或 NoSQL API） 。 Azure Cosmos DB for NoSQL 帐户完成预配后，你可以检索终结点和密钥。 使用终结点和密钥以编程方式连接到 Azure Cosmos DB for NoSQL 帐户。 在 Azure SDK for .NET 或任何其他 SDK 的连接字符串上使用终结点和密钥。

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
    | “全局分发”选项卡**** | 禁用多区域写入 |

    > &#128221; 请注意，你可以在创建 Azure Cosmos DB 帐户期间启用“连续”（7 天）模式，方法是在“备份策略”选项卡下选择该模式********。在此实验室中，可以选择在创建帐户期间或在创建帐户之后在以下可选部分中启用此功能。 但是，在创建帐户<ins>后</ins>启用该功能可能需要 5 分钟以上的时间********。

    > &#128221; 请注意，[连续备份目前不支持多区域写入帐户][/azure/cosmos-db/continuous-backup-restore-introduction]**。

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

## 向帐户添加一个数据库和两个容器

让我们创建一个数据库和几个容器。

1. 在 Azure 门户，导航到“Azure Cosmos DB 帐户”页面。

1. 在“数据资源管理器”下，添加具有以下设置的新容器****

    | **设置** | **值** |
    | ---: | :--- |
    | **数据库 ID** | “新建”** 名称：*`Sales`* |
    | **在容器之间共享吞吐量** | 请不要选择 |
    | **容器 ID** | *`customer`* |
    | **分区键** | *`/id`* |
    | **容器吞吐量（400 - 无限制 RU/s）** | 手动吞吐量：400****|

1. 在“数据资源管理器”下，添加具有以下设置的新容器****

    | **设置** | **值** |
    | ---: | :--- |
    | **数据库 ID** | 使用现有名称：Sales**** |
    | **容器 ID** | *`salesOrder`* |
    | **分区键** | *`/id`* |
    | **容器吞吐量（400 - 无限制 RU/s）** | 手动吞吐量：400****|

## 将项添加到容器

让我们向这些容器添加一些文档。

1. 在 Azure 门户，导航到“Azure Cosmos DB 帐户”页面。

1. 在“数据资源管理器”下，将以下两个文档添加到 customer 容器********。

```
  {
    "id": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "title": "",
    "firstName": "Franklin",
    "lastName": "Ye",
    "emailAddress": "franklin9@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0139",
    "creationDate": "2014-02-05T00:00:00",
    "addresses": [
      {
        "addressLine1": "1796 Westbury Dr.",
        "addressLine2": "",
        "city": "Melton",
        "state": "VIC",
        "country": "AU",
        "zipCode": "3337"
      }
    ],
    "password": {
      "hash": "GQF7qjEgMl3LUppoPfDDnPtHp1tXmhQBw0GboOjB8bk=",
      "salt": "12C0F5A5"
    }
  }
```

```
  {
    "id": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "title": "",
    "firstName": "Victor",
    "lastName": "Moreno",
    "emailAddress": "victor8@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0134",
    "creationDate": "2011-10-09T00:00:00",
    "addresses": [
      {
        "addressLine1": "Parkstr 42",
        "addressLine2": "",
        "city": "Hamburg",
        "state": "HH ",
        "country": "DE",
        "zipCode": "20354"
      }
    ],
    "password": {
      "hash": "n8l+wY/klP/hwTC3wSr8BLMA9tm3tGTyDsCgG/Q9EYI=",
      "salt": "AC22BC8C"
    }
  }
```
1. 在“数据资源管理器”下，将以下三个文档添加到 salesOrder 容器********。

```
  {
    "id": "000C23D8-B8BC-432E-9213-6473DFDA2BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2014-02-16T00:00:00",
    "shipDate": "2014-02-23T00:00:00",
    "details": [
      {
        "sku": "BK-R64Y-42",
        "name": "Road-550-W Yellow, 42",
        "price": 1120.49,
        "quantity": 1
      },
      {
        "sku": "HL-U509-B",
        "name": "Sport-100 Helmet, Blue",
        "price": 34.99,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "001676F7-0B70-400B-9B7D-24BA37B97F70",
    "customerId": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "orderDate": "2013-06-02T00:00:00",
    "shipDate": "2013-06-09T00:00:00",
    "details": [
      {
        "sku": "HL-U509-R",
        "name": "Sport-100 Helmet, Red",
        "price": 34.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "0019092E-BD25-48F5-8050-7051B2655BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2013-09-14T00:00:00",
    "shipDate": "2013-09-21T00:00:00",
    "details": [
      {
        "sku": "TI-T723",
        "name": "Touring Tire",
        "price": 28.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      },
      {
        "sku": "TT-T092",
        "name": "Touring Tire Tube",
        "price": 4.99,
        "quantity": 1
      }
    ]
  }
```

## 将默认的备份模式改为连续备份（可选，如果在创建帐户时未启用该功能）

如果在创建 Azure Cosmos DB 帐户期间未启用该功能，则需要立即启用此功能**。  更改备份模式很简单，只需将一个设置更改为“开启”****。 现在，让我们更改它。

1. 在 Azure 门户，导航到“Azure Cosmos DB 帐户”页面。

1. 在“设置”**** 部分下，选择“备份和还原”****。

1. 在“备份策略”**** 模式旁边选择“更改”****，在屏幕中选择“连续(7 天)”**** 选项，然后选择“保存”****。 启用此功能可能需要 5 分钟以上的时间。******

    > &#128221; 请注意，[连续备份目前不支持多区域写入帐户][/azure/cosmos-db/continuous-backup-restore-introduction]**。 如果在创建 Azure Cosmos DB 帐户时未禁用多区域写入，则需要立即执行此操作，否则启用连续备份功能将失败。  可以在“全局复制数据”设置部分下禁用多区域写入******。

## 删除其中一个 salesOrder 文档

1. 在“数据资源管理器”下，运行以下查询获取当前日期和时间****。 将时间戳复制到记事本。 此时间戳应采用 UTC 格式。

    ```
    SELECT GetCurrentDateTime ()
    ```

1. 在“数据资源管理器”下，找到 ID 为 `0019092E-BD25-48F5-8050-7051B2655BC5` 的 salesOrder 文档************。 删除文档，验证文档不再存在。

## 在删除 salesOrder 文档之前，将数据库还原到点

1. 在 Azure 门户，导航到“Azure Cosmos DB 帐户”页面。

1. 在“设置”部分下，选择“时间点还原”******。 使用以下设置：

    | **设置** | **值** |
    | ---: | :--- |
    | **还原点 (UTC)** | 适当转换日期和时间。 时间需要采用 AM/PM 格式|
    | **位置** | 已选择可用位置** |
    | **选择要还原的资源** | 已选择数据库/容器** |
    | **还原资源** | salesOrder** |
    | **还原目标帐户** | 选择一个新的 Azure Cosmos DB 帐户名称********** |

    > &#128221; 对于 Azure Cosmos DB 还原，切勿在现有帐户的基础上还原，而是始终必须创建一个新的 Azure Cosmos DB 帐户********。

    > &#128221; 尽管可以选择还原整个数据库甚至整个帐户，但在实际的生产环境中，数据库可能会很大。 在许多情况下，只还原容器或需要的数据库可能会更快。

1. 此还原可能需要 15 分钟或更长时间，请转到下一部分，让此还原在后台运行。

## 删除 customer 容器

1. 在“数据资源管理器”下，运行以下查询获取当前日期和时间****。 将时间戳复制到记事本。

    ```
    SELECT GetCurrentDateTime ()
    ```

1. 删除 customer 容器****。

## 在删除 salesOrder 文档之前，将数据库还原到点

1. 在 Azure 门户，导航到“Azure Cosmos DB 帐户”页面。

1. 在“设置”部分下，选择“时间点还原”******。 使用以下设置：

    | **设置** | **值** |
    | ---: | :--- |
    | **位置** | 已选择可用位置** |
    | **还原点 (UTC)** | 适当转换日期和时间。 时间需要采用 AM/PM 格式|
    | **选择要还原的资源** | 已选择数据库/容器** |
    | **还原资源** | *`customer`* |
    | **还原目标帐户** | 选择一个新的 Azure Cosmos DB 帐户名称********** |

    > &#128221; 对于 Azure Cosmos DB 还原，切勿在现有帐户的基础上还原，而是始终必须创建一个新的 Azure Cosmos DB 帐户********。

    > &#128221;尽管可以选择还原整个数据库甚至整个帐户，但在实际的生产环境中，数据库可能会很大。 在许多情况下，只还原容器或需要的数据库可能会更快。

1. 此还原可能需要 15 分钟或更长时间，请转到下一部分，让此还原在后台运行。

## 查看还原的数据

还原可能需要很长时间，具体取决于数据库的大小和其他因素。 Azure Cosmos DB 帐户还原完成后：

1. 对于我们的第一次还原，请确保第三个文档已恢复。

1. 对于第二次还原，应还原 customer 表。

## 清理

1. 删除通过帐户还原创建的两个新的 Azure Cosmos DB 帐户。

1. 删除 Sales 数据库，如果需要，请删除原始 Azure Cosmos DB 帐户。

[/azure/cosmos-db/continuous-backup-restore-introduction]:https://docs.microsoft.com/azure/cosmos-db/continuous-backup-restore-introduction

