---
lab:
  title: 04 - 批处理多点操作与 Azure Cosmos DB for NoSQL SDK
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
---

# 批处理多点操作与 Azure Cosmos DB for NoSQL SDK

适用于 Azure Cosmos DB 的 JavaScript SDK 中的 `TransactionalBatch` 类提供在同一逻辑分区键中撰写和执行批处理操作的功能。 利用此功能，你可以在单个事务中执行多个操作，并确保要么完成所有操作，要么任何操作均未完成。

在本实验室中，你将使用 JavaScript SDK 执行双项批处理操作，尝试将两个项创建为单个逻辑单元。

## 准备开发环境

如果尚未为**使用 Azure Cosmos DB 生成 Copilot** 克隆实验室代码存储库并设置本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明以执行此操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的 **使用 Azure Cosmos DB 生成 Copilot** 实验室创建 Azure Cosmos DB for NoSQL 帐户，则可以将其用于此实验室，并跳到“[下一部分](#import-the-azurecosmos-library)”。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，以创建将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并向用户标识授予访问权限以管理帐户中的数据，方法是将其分配给 **Cosmos DB 内置数据参与者**角色。

## 导入@azure/cosmos库

**@azure/cosmos** 库在 **npm** 上可用，可轻松安装到 JavaScript 项目中。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **javascript/04-sdk-batch** 文件夹。

1. 打开 **javascript/04-sdk-batch** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为 **javascript/04-sdk-batch** 文件夹的终端。

1. 初始化新的 Node.js 项目：

    ```bash
    npm init -y
    ```

1. 使用以下命令安装 [@azure/cosmos][npmjs.com/package/@azure/cosmos] 包：

    ```bash
    npm install @azure/cosmos
    ```

1. 使用以下命令安装[@azure/identity][npmjs.com/package/@azure/identity]库，以便使用 Azure 身份验证连接到 Azure Cosmos DB 工作区：

    ```bash
    npm install @azure/identity
    ```

## 使用@azure/cosmos库

从适用于 JavaScript 的 Azure SDK 中导入 Azure Cosmos DB 库后，可以立即使用其类连接到 Azure Cosmos DB for NoSQL 帐户。 **CosmosClient** 类是一个核心类，用于与 Azure Cosmos DB for NoSQL 帐户建立初始连接。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **javascript/04-sdk-batch** 文件夹。

1. 打开名为 **script.js** 的空 JavaScript 文件。

1. 添加以下`require`语句以导入**@azure/cosmos**和**@azure/identity**库：

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
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

1. 添加以下代码，若数据库和容器尚未存在，则进行创建：

    ```javascript
    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
    ```

1. **script.js** 文件现在应如下所示：

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
        
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    }
    
    main().catch((error) => console.error(error));
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

1. 切回到 Web 浏览器窗口。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”**** 中，展开“cosmicworks”**** 数据库节点，然后在“NoSQL API”**** 导航树中观察新“products”**** 容器节点。

## 创建事务批处理

首先，我们创建一个简单的事务批处理，该批处理向容器中添加两个虚构的产品。 此批处理将使用相同的“旧配件”类别标识符，将 worn saddle 和 rusty handlebar 插入到容器中。 这两个项具有相同的逻辑分区键，确保批处理操作成功。

1. 返回到 **Visual Studio Code**。 如果尚未打开，请打开 **javascript/04-sdk-batch** 文件夹下的 **script.js** 代码文件。

1. 定义要在事务批处理中插入的两个产品项：

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    ```

1. 为同一逻辑分区键创建事务批处理并添加项：

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = container.items.batch(partitionKey)
        .create(saddle)
        .create(handlebar);
    ```

1. 执行批处理并打印操作的状态：

    ```javascript
    const response = await batch.execute();
    console.log(`Status: ${response.statusCode}`);
    ```

1. 完成后，代码文件现在应包含：
  
    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: saddle },
            { operationType: BulkOperationType.Create, resourceBody: handlebar },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. **保存**并再次运行脚本：

    ```bash
    node script.js
    ```

1. 输出应指示每个操作的成功状态代码。

## 创建错误的事务批处理

现在来创建一个故意出错的事务批处理。 此批处理将尝试插入两个具有不同逻辑分区键的项。 我们将在“旧配件”类别中创建一个 flickering strobe light，并在“新配件”类别中创建一个 new helmet。 按照定义，此请求应为错误请求，会在执行此事务时返回错误。

1. 返回到 **script.js** 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

    ```javascript
    const saddle = { id: "0120", name: "Worn Saddle", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const handlebar = { id: "012A", name: "Rusty Handlebar", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };

    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: saddle },
        { operationType: BulkOperationType.Create, resourceBody: handlebar },
    ];
    ```

1. 修改脚本，以使用不同的分区键值创建新的 **flickering strobe light** 和 **new helmet**。

    ```javascript
    const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
    const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    ```

1. 创建一个名为 **partition_key** 的字符串变量，其值为 **9603ca6c-9e28-4a02-9194-51cdb7fea816**：

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. 创建包含 **light** 和 **helmet** 项的新的批处理：

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    const batch = [
        { operationType: BulkOperationType.Create, resourceBody: light },
        { operationType: BulkOperationType.Create, resourceBody: helmet },
    ];
    ```

1. 完成后，代码文件现在应包含：

    ```javascript
    const { CosmosClient, BulkOperationType } = require("@azure/cosmos");
    const { DefaultAzureCredential  } = require("@azure/identity");
    process.env.NODE_TLS_REJECT_UNAUTHORIZED = 0

    const endpoint = "<cosmos-endpoint>";
    const credential = new DefaultAzureCredential();

    const client = new CosmosClient({ endpoint, aadCredentials: credential });

    async function main() {
        // Create database
        const { database } = await client.databases.createIfNotExists({ id: "cosmicworks" });
            
        // Create container
        const { container } = await database.containers.createIfNotExists({
            id: "products",
            partitionKey: { paths: ["/categoryId"] },
            throughput: 400
        });
    
        const light = { id: "012B", name: "Flickering Strobe Light", categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816" };
        const helmet = { id: "012C", name: "New Helmet", categoryId: "0feee2e4-687a-4d69-b64e-be36afc33e74" };
    
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
        const batch = [
            { operationType: BulkOperationType.Create, resourceBody: light },
            { operationType: BulkOperationType.Create, resourceBody: helmet },
        ];
    
        try {
            const response = await container.items.batch(batch, partitionKey);
    
            response.result.forEach((operationResult, index) => {
                const { statusCode, requestCharge, resourceBody } = operationResult;
                console.log(`Operation ${index + 1}: Status code: ${statusCode}, Request charge: ${requestCharge}, Resource: ${JSON.stringify(resourceBody)}`);
            });
        } catch (error) {
            if (error.code === 400) {
                console.error("Bad Request: Check the structure of the batch.");
            } else if (error.code === 409) {
                console.error("Conflict: One of the items already exists.");
            } else if (error.code === 429) {
                console.error("Too Many Requests: Throttling limit reached.");
            } else {
                console.error(`Batch operation failed. Error code: ${error.code}, message: ${error.message}`);
            }
        }
    }
    
    main().catch((error) => console.error(error));
    ```

1. **保存**并再次运行脚本：

    ```bash
    node script.js
    ```

1. 查看终端输出。 项的状态代码应为 **424**（对于**失败的依赖项**）或 **400**（对于**错误的请求**）。 出现此错误的原因是该事务中的所有项都没有与事务批处理共享相同的分区键值。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
