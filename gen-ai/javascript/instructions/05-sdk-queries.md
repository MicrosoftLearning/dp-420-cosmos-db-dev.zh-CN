---
title: 05 - 使用 Azure Cosmos DB for NoSQL SDK 执行查询
lab:
  title: 05 - 使用 Azure Cosmos DB for NoSQL SDK 执行查询
  module: Query the Azure Cosmos DB for NoSQL
layout: default
nav_order: 8
parent: JavaScript SDK labs
---

# 使用 Azure Cosmos DB for NoSQL SDK 执行查询

最新版本的适用于 Azure Cosmos DB for NoSQL 的 JavaScript 简化容器查询，并利用 JavaScript 的现代特性遍历结果集。

该`@azure/cosmos`库内置功能，查询 Azure Cosmos DB 高效简单。

在此实验室中，你将使用迭代器遍历从 Azure Cosmos DB for NoSQL 返回的大型结果集。 你将使用 JavaScript SDK 执行查询并遍历结果。

## 准备开发环境

如果你还没有克隆**使用 Azure Cosmos DB 生成助手**的实验室代码存储库并设置你的本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明进行操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的**使用 Azure Cosmos DB 生成助手**实验室创建了 Azure Cosmos DB for NoSQL 帐户，你可以将其用于此实验室，并跳转至[下一部分](#create-azure-cosmos-db-database-and-container-with-sample-data)。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，创建一个将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并通过向帐户分配 **Cosmos DB 内置数据参与者**角色，授予你的用户标识访问权限，以管理帐户中的数据。

## 创建 Azure Cosmos DB 数据库和包含示例数据的容器

如果已创建一个名为 **cosmicworks-full** 的 Azure Cosmos DB 数据库，其中的容器名为 **products**，且该容器预加载了示例数据，则可以将其用于此实验室，并跳转至[下一部分](#import-the-azurecosmos-library)。 否则，请按照以下步骤创建新的示例数据库和容器。

<details markdown=1>
<summary markdown="span"><strong>单击可展开/折叠步骤以创建数据库和包含示例数据的容器</strong></summary>

1. 在新创建的 **Azure Cosmos DB** 帐户资源中，导航到“**数据资源管理器**”窗格。

1. 在“**数据资源管理器**”的主页中，选择“**启动快速入门**”。

1. 在“**新容器**”窗体中，输入以下值：

    - **数据库 ID**：`cosmicworks-full`
    - **容器 ID**：`products`
    - **分区键**：`/categoryId`
    - **分析存储**：`Off`

1. 选择“**确定**”创建新容器。 此过程在创建资源，以及为容器预加载示例产品数据时将需要花费一两分钟的时间。

1. 使浏览器选项卡保持打开状态，因为我们稍后将返回这里。

1. 切回到 Visual Studio Code****。

</details>

## 导入 @azure/cosmos 库

**@azure/cosmos** 库在 **npm** 上可用，可轻松安装到 JavaScript 项目中。

1. 在 **Visual Studio Code** 中打开 **“资源管理器”** 窗格，定位到 **javascript/05-sdk-queries** 文件夹。

1. 点击 "javascript/05-sdk-queries" **** 文件夹，打开上下文菜单，然后选择“在集成终端中打开”****，启动新的终端实例。

    > &#128221；此命令将打开起始目录已设置为 "javascript/05-sdk-queries" **** 文件夹的终端。

1. 初始化新的 Node.js 项目：

    ```bash
    npm init -y
    ```

1. 使用以下命令安装 [@azure/cosmos][npmjs.com/package/@azure/cosmos] 包：

    ```bash
    npm install @azure/cosmos
    ```

1. 使用以下命令安装 [@azure/identity][npmjs.com/package/@azure/identity] 库，以便使用 Azure 身份验证连接到 Azure Cosmos DB 工作区：

    ```bash
    npm install @azure/identity
    ```

## 使用 SDK 来遍历 SQL 查询的结果

使用新创建帐户中的凭据，与 SDK 类建立连接，然后连接到之前步骤中配置的数据库和容器，接着使用 SDK 执行 SQL 查询并遍历结果。

现在，你将使用迭代器，在 Azure Cosmos DB 的分页结果中创建一个简单明了的循环。 SDK 会在后台管理源迭代器，确保正确调用后续请求。

1. 在 **Visual Studio Code** 中打开 **“资源管理器”** 窗格，定位到 **javascript/05-sdk-queries** 文件夹。

1. 打开名为 **script.js** 的空 JavaScript 文件。

1. 添加以下 `require` 语句，以导入 **@azure/cosmos** 和 **@azure/identity** 库：

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    ```

1. 更新名为 **endpoint** 和 **credential** 的变量，将 **endpoint** 值设置为之前创建的 Azure Cosmos DB 帐户的**终结点**。 **credential** 变量应设置为 **DefaultAzureCredential** 类的新实例：

    ```javascript
    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();
    ```

    > &#128221; 例如，如果终结点为：**https://dp420.documents.azure.com:443/**，则语句为：**const endpoint = "https://dp420.documents.azure.com:443/";**。

1. 添加名为 **client** 的新变量，并使用 **endpoint** 和 **credential** 变量将其初始化为 **CosmosClient** 类的新实例：

    ```javascript
    const client = new CosmosClient({ endpoint, aadCredentials: credential });
    ```

1. 创建名为 **queryContainer** 新方法，并编写代码，确保运行脚本时执行该方法。 你将添加代码以在此方法中查询容器：

    ```javascript
    async function queryContainer() {
        // Query the container
    }

    queryContainer().catch((error) => {
        console.error(error);
    });
    ```

1. 在 **queryContainer** 方法中，添加以下代码，连接到之前创建的数据库和容器：

    ```javascript
    const database = client.database("cosmicworks-full");
    const container = database.container("products");
    ```

1. 创建名为 `sql` 且值为 `SELECT * FROM products p` 的查询字符串变量。

    ```javascript
    const sql = "SELECT * FROM products p";
    ```

1. 使用`sql`变量作为构造函数的参数，调用[`query`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-query-1)方法。 `enableCrossPartitionQuery`参数设置为`true`，即可发送多个请求执行 Azure Cosmos DB 服务中的查询。 如果查询未限定为单个分区键值，则需要多个请求。

    ```javascript
    const iterator = container.items.query(
        query,
        { enableCrossPartitionQuery: true }
    );
    ```

1. 遍历分页结果并输出每个项的`id`、`name`和`price`：

    ```javascript
    while (iterator.hasMoreResults()) {
        const { resources } = await iterator.fetchNext();
        for (const item of resources) {
            console.log(`[${item.id}]   ${item.name.padEnd(35)} ${item.price.toFixed(2)}`);
        }
    }
    ```

1. **script.js** 文件现在应如下所示：

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function queryContainer() {
        const database = client.database("cosmicworks-full");
        const container = database.container("products");
        
        const query = "SELECT * FROM products p";
    
        const iterator = container.items.query(
            query,
            { enableCrossPartitionQuery: true }
        );
        
        while (iterator.hasMoreResults()) {
            const { resources } = await iterator.fetchNext();
            for (const item of resources) {
                console.log(`[${item.id}]   ${item.name.padEnd(35)} ${item.price.toFixed(2)}`);
            }
        }
    }
    
    queryContainer().catch((error) => {
        console.error(error);
    });
    ```

1. **保存** **script.cs** 文件。

1. 运行脚本之前，必须使用`az login`命令登录 Azure。 在“终端”窗口中，运行：

    ```bash
    az login
    ```

1. 运行脚本创建数据库和容器

    ```bash
    node script.js
    ```

1. 该脚本现在会输出容器中的每个项

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
