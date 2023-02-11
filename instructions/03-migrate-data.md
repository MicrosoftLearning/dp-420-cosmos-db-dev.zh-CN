---
lab:
  title: 使用 Azure 数据工厂迁移现有数据
  module: Module 2 - Plan and implement Azure Cosmos DB for NoSQL
---

# 使用 Azure 数据工厂迁移现有数据

在 Azure 数据工厂中，支持将 Azure Cosmos DB 作为数据引入的源以及数据输出的目标（接收器）。

在本实验室中，我们将使用有用的命令行实用工具填充 Azure Cosmos DB，然后使用 Azure 数据工厂将数据的子集从一个容器移到另一个容器。

## 创建 Azure Cosmos DB for NoSQL 帐户并设定种子

你将使用命令行实用工具来创建 cosmicworks 数据库和 products 容器，每秒 4,000 个请求单位（RU/秒）。 创建后，将吞吐量调整至 400 RU/秒。

为了随附 products 容器，需要手动创建 flatproducts 容器，该容器将是本实验室结束时 ETL 转换和加载操作的目标。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭证登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB for NoSQL”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **帐户名** | 输入全局唯一名称 |
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *预配的吞吐量* |
    | **应用免费分级折扣** | *不应用* |
    | 限制可在此帐户上预配的总吞吐量 | *未选中* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 记录“URI”字段的值。 稍后在本练习中将用到此终结点值。

    1. 记录“主键”字段的值。 稍后在本练习中将用到此键值。

1. 关闭 Web 浏览器窗口或选项卡。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅 [Visual Studio Code 入门指南][code.visualstudio.com/docs/getstarted]

1. 在 Visual Studio Code 中，打开“终端”菜单，然后选择“新建终端”以打开新的终端实例。

1. 在计算机上安装可全局使用的 [cosmicworks][nuget.org/packages/cosmicworks] 命令行工具。

    ```
    dotnet tool install --global cosmicworks
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

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，选择先前在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，依次展开“cosmicworks”数据库节点和“products”容器节点，然后选择“项”。

1. 查看并选择 products 容器中的各种 JSON 项。 这些是前面的步骤中使用命令行工具创建的项。

1. 选择“缩放和设置”节点。 在“缩放和设置”选项卡中，选择“手动”，将“所需吞吐量”设置从“4000 RU/秒”更新为“400 RU/秒”，然后保存更改**。

1. 在“数据资源管理器”窗格中，选择“新建容器” 。

1. 在“新建容器”弹出窗口中，为每个设置输入以下值，然后选择“确定” ：

    | **设置** | **值** |
    | --: | :-- |
    | **数据库 ID** | 使用现有 &vert; cosmicworks  |
    | **容器 ID** | *`flatproducts`* |
    | **分区键** | *`/category`* |
    | **容器吞吐量(自动缩放)** | *手动* |
    | **RU/秒** | *`400`* |

1. 返回到“数据资源管理器”窗格中，展开“cosmicworks”数据库节点，然后观察层次结构中的“flatproducts”容器节点。

1. 转到 Azure 门户的主页。

## 创建 Azure 数据工厂资源

Azure Cosmos DB for NoSQL 资源已就绪，接下来你将创建一个 Azure 数据工厂资源，并配置所有必需的组件和连接，以执行从一个 NoSQL API 容器到另一个容器的一次性数据移动，以提取数据、转换数据，并将其加载到另一个 NoSQL API 容器。

1. 选择“+ 创建资源”，搜索“数据工厂”，然后使用以下设置创建新的“Azure 数据工厂”资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **名称** | 输入全局唯一名称 |
    | **区域** | 选择任何可用区域 |
    | **版本** | *V2* |
    | **Git 配置** | 稍后配置 Git |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的“Azure 数据工厂”资源，然后选择“启动工作室”。

    > &#128161; 或者，可以导航到 (``adf.azure.com/home``)，选择新创建的数据工厂资源，然后选择主页图标。

1. 在主屏幕中。 选择“引入”选项以启动快速向导，执行一次性大规模复制数据操作，并转到向导的“属性”步骤。

1. 从向导的“属性”步骤开始，在“任务类型”部分，选择“内置复制任务”。

1. 在“任务节奏或任务计划”部分，选择“立即运行一次”，然后选择“下一步”以转到向导的“源”步骤。

1. 在向导的“源”步骤中，选择“源类型”列表中的“Azure Cosmos DB (NoSQL API)”。

1. 在“连接”部分，选择“+ 新建连接”。

1. 在“新建连接(Azure DB Cosmos (NoSQL API))”弹出窗口中，配置具有以下值的新连接，然后选择“创建”：

    | **设置** | **值** |
    | ---: | :--- |
    | **名称** | *`CosmosSqlConn`* |
    | **通过集成运行时进行连接** | AutoResolveIntegrationRuntime |
    | **身份验证方法** | 帐户密钥 &vert; 连接字符串 |
    | **帐户选择方法** | *从 Azure 订阅中* |
    | **Azure 订阅** | 你的现有 Azure 订阅 |
    | **Azure Cosmos DB 帐户名称** | 你之前在本实验室中选择的现有 Azure Cosmos DB 帐户名称 |
    | **数据库名称** | cosmicworks |

1. 返回到“源数据存储”部分，在“源表”部分中，选择“使用查询”。

1. 在“表名称”列表中，选择“产品”。

1. 在“查询”编辑器中，删除现有内容并输入以下查询：

    ```
    SELECT 
        p.name, 
        p.categoryName as category, 
        p.price 
    FROM 
        products p
    ```

1. 选择“预览数据”以测试查询的有效性。 选择“下一步”，转到向导的“目标”步骤。

1. 在向导的“目标”步骤中，选择“目标类型”列表中的“Azure Cosmos DB (NoSQL API)”。

1. 在“连接”列表中，选择“CosmosSqlConn”。

1. 在“目标”列表中，选择“flatproducts”，然后选择“下一步”，转到向导的“设置”步骤。

1. 在向导的“设置”步骤中，在“任务名称”字段中输入 `FlattenAndMoveData`  。

1. 将所有剩余字段保留为默认的空值，然后选择“下一步”，转到向导的最后一步。

1. 查看在向导中选择的步骤的“摘要”，然后选择“下一步”。

1. 观察部署中的各种步骤。 部署完成后，选择“完成”。

1. 关闭 Web 浏览器窗口或选项卡。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，选择先前在此实验室中创建或查看的资源组，然后选择在此实验室中创建的“Azure Cosmos DB 帐户”资源。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，选择“flatproducts”容器节点，然后选择“新建 SQL 查询”。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询将返回 name 等同于 HL Headset 的所有文档：

    ```
    SELECT 
        p.name, 
        p.category, 
        p.price 
    FROM
        flatproducts p
    WHERE
        p.name = 'HL Headset'
    ```

1. 选择“执行查询”。

1. 观察查询结果。

1. 关闭 Web 浏览器窗口或选项卡。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
