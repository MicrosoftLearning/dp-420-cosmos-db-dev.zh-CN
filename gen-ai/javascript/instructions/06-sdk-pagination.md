---
lab:
  title: 06 - 在 Azure Cosmos DB for NoSQL SDK 中对叉积查询结果进行分页
  module: Author complex queries with the Azure Cosmos DB for NoSQL
---

# 在 Azure Cosmos DB for NoSQL SDK 中对叉积查询结果进行分页

Azure Cosmos DB 查询通常会有多页的结果。 当 Azure Cosmos DB 不能在单次执行中返回所有查询结果时，分页会在服务器端自动完成。 在许多应用程序中，你需要使用 SDK 编写代码，以高性能的方式分批处理查询结果。

在本实验室中，你将创建一个可在循环中使用的源迭代器，用来循环访问整个结果集。

## 准备开发环境

如果尚未为**使用 Azure Cosmos DB 生成 Copilot** 克隆实验室代码存储库并设置本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明以执行此操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的 **使用 Azure Cosmos DB 生成 Copilot** 实验室创建 Azure Cosmos DB for NoSQL 帐户，则可以将其用于此实验室，并跳到“[下一部分](#create-azure-cosmos-db-database-and-container-with-sample-data)”。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，创建一个将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并通过向帐户分配 **Cosmos DB 内置数据参与者**角色，授予你的用户标识访问权限，以管理帐户中的数据。

## 创建 Azure Cosmos DB 数据库和包含示例数据的容器

如果你已创建一个名为 **cosmicworks-full** 的 Azure Cosmos DB 数据库，并且其中的容器 **products** 已预加载示例数据，则可以在此实验室中使用该数据库，并跳至[下一部分](#import-the-azurecosmos-library)。 否则，请按照以下步骤创建新的示例数据库和容器。

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

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **javascript/06-sdk-pagination** 文件夹。

1. 打开 **javascript/06-sdk-pagination** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为 **javascript/06-sdk-pagination** 文件夹的终端。

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

处理查询结果时，必须确保代码处理所有结果页面，并在发出后续请求之前检查是否还有任何其他页面。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **javascript/06-sdk-pagination** 文件夹。

1. 打开名为 **script.js** 的空 JavaScript 文件。

1. 添加以下`require`语句以导入**@azure/cosmos**和**@azure/identity**库：

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0
    ```

1. 添加名为 **endpoint** 和 **credential** 的变量，并将 **endpoint** 值设置为之前创建的 Azure Cosmos DB 帐户的 **endpoint**。 **credential** 变量应设置为 **DefaultAzureCredential** 类的新实例：

    ```javascript
    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();
    ```

    > #128221；例如，如果终结点为：**https://dp420.documents.azure.com:443/**，则语句为：**const endpoint = "https://dp420.documents.azure.com:443/";**。

1. 添加名为 **client** 的新变量，并使用 **endpoint** 和 **credential** 变量将其初始化为 **CosmosClient** 类的新实例：

    ```javascript
    const client = new CosmosClient({ endpoint, aadCredentials: credential });
    ```

1. 创建一个名为 **paginateResults** 的新方法，并创建在运行脚本时用于执行该方法的代码。 你将添加代码以在此方法中查询容器：

    ```javascript
    async function paginateResults() {
        // Query the container
    }

    paginateResults().catch((error) => {
        console.error(error);
    });
    ```

1. 在 **paginateResults** 方法中，添加以下代码以连接到之前创建的数据库和容器：

    ```javascript
    const database = client.database("cosmicworks-full");
    const container = database.container("products");
    ```

1. 创建名为 `sql` 且值为 `SELECT * FROM products p` 的查询字符串变量。

    ```javascript
    const sql = "SELECT * FROM products p";
    ```

1. 创建一个名为 `options` 的新变量，并将其设置为一个对象，其中对象的 `enableCrossPartitionQuery` 属性设置为 `true`。 此属性允许发送多个请求以在 Azure Cosmos DB 服务中执行查询。 如果查询未限定为单个分区键值，则需要多个请求。 将 `maxItemCount` 属性设置为 `50` 以限制每页的项数：

    ```javascript
    const options = {
        enableCrossPartitionQuery: true,
        maxItemCount: 50 // Set the maximum number of items per page
    };
    ```

1. 将 `sql` 和 `options` 变量作为构造函数的参数来调用 [`query`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-query-1) 方法。 此方法将返回一个可用于提取下一页结果的迭代器：

    ```javascript
    const iterator = container.items.query(query, options);
    ```

1. 循环访问分页结果并输出每个项目的 `id`、`name` 和 `price`。 `iterator.getAsyncIterator` 方法将返回一个可用于提取 `for await...of` 循环以检索每页结果的异步迭代器。 此循环会自动处理页面的异步迭代。

    ```javascript
    for await (const page of iterator.getAsyncIterator()) {
        page.resources.forEach(product => {
            console.log(`[${product.id}] ${product.name} $${product.price.toFixed(2)}`);
        });
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

    async function paginateResults() {
        const database = client.database("cosmicworks-full");
        const container = database.container("products");
        
        const query = "SELECT * FROM products WHERE products.price > 500";

        const options = {
            enableCrossPartitionQuery: true,
            maxItemCount: 50 // Set the maximum number of items per page
        };
        
        const iterator = container.items.query(query, options);

        for await (const page of iterator.getAsyncIterator()) {
            page.resources.forEach(product => {
                console.log(`[${product.id}] ${product.name} $${product.price.toFixed(2)}`);
            });
        }
    }
    
    paginateResults().catch((error) => {
        console.error(error);
    });
    ```

1. **保存** **script.cs** 文件。

1. 运行脚本之前，必须使用 `az login` 命令登录 Azure。 在“终端”窗口中，运行：

    ```bash
    az login
    ```

1. 运行脚本创建数据库和容器

    ```bash
    node script.js
    ```

1. 该脚本现在一次将输出 50 个项目的页面。

    > &#128161; 查询将匹配 products 容器中的数百个项。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
