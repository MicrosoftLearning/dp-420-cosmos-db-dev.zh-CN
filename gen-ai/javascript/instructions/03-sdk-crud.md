---
lab:
  title: 03 - 使用 Azure Cosmos DB for NoSQL SDK 创建和更新文档
  module: Implement Azure Cosmos DB for NoSQL point operations
---

# 使用 Azure Cosmos DB for NoSQL SDK 创建和更新文档

该`@azure/cosmos`库包括在 Azure Cosmos DB for NoSQL 容器中创建、检索、更新和删除 (CRUD) 项的操作方法。 这些方法共同执行 NoSQL API 容器中各种项的最常见“CRUD”操作。

在本实验室中，你将在 Azure Cosmos DB for NoSQL 容器中使用 JavaScript SDK 对项执行日常 CRUD 操作。

## 准备开发环境

如果尚未为**使用 Azure Cosmos DB 生成 Copilot** 克隆实验室代码存储库并设置本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明以执行此操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的 **使用 Azure Cosmos DB 生成 Copilot** 实验室创建 Azure Cosmos DB for NoSQL 帐户，则可以将其用于此实验室，并跳到“[下一部分](#import-the-azurecosmos-library)”。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，以创建将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并向用户标识授予访问权限以管理帐户中的数据，方法是将其分配给 **Cosmos DB 内置数据参与者**角色。

## 导入@azure/cosmos库

**@azure/cosmos** 库在 **npm** 上可用，可轻松安装到 JavaScript 项目中。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，访问 **javascript/03-sdk-crud** 文件夹。

1. 打开 **javascript/03-sdk-crud** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为 **javascript/03-sdk-crud** 文件夹的终端。

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

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，访问 **javascript/03-sdk-crud** 文件夹。

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
    const { CosmosClient } = require("@azure/cosmos");
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

## 使用 SDK 对项执行创建和读取点操作

现在，你将在 **Container** 类中使用这组方法，对 NoSQL API 容器中的项执行常见操作。

1. 返回到 **Visual Studio Code**。 如果尚未打开，请打开 **javascript/03-sdk-crud** 文件夹中的 **script.js** 代码文件。

1. 创建一个新产品项，并将其分配给具有以下属性的名为 **saddle** 的变量。 确保将以下代码添加到 `main` 函数中：

    | 属性 | 值 |
    | ---: | :--- |
    | **id** | 706cd7c6-db8b-41f9-aea2-0e0c7e8eb009 |
    | categoryId | 9603ca6c-9e28-4a02-9194-51cdb7fea816 |
    | **name** | *Road Saddle* |
    | **price** | 45.99d |
    | **tags** | { tan, new, crisp } |

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };
    ```

1. 调用容器 **items** 类的[`create`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#@azure-cosmos-items-create)方法，并将 **saddle** 变量作为方法参数传入：

    ```javascript
    const { resource: item } = await container
        .items.create(saddle);
    ```

1. 完成后，代码文件现在应包含：
  
    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const saddle = {
            id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
            categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
            name: "Road Saddle",
            price: 45.99,
            tags: ["tan", "new", "crisp"]
        };
    
        const { resource: item } = await container
                .items.create(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **保存**并再次运行脚本：

    ```bash
    node script.js
    ```

1. 观察**数据资源管理器**中的新项。

1. 返回到 **Visual Studio Code**。

1. 返回到 **script.js** 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

    ```javascript
    const saddle = {
        id: "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId: "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name: "Road Saddle",
        price: 45.99,
        tags: ["tan", "new", "crisp"]
    };

    const { resource: item } = await container
            .items.create(saddle);
    ```

1. 创建一个名为 **item_id** 的字符串变量，其值为 **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**：

    ```javascript
    const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. 创建一个名为 **partition_key** 的字符串变量，其值为 **9603ca6c-9e28-4a02-9194-51cdb7fea816**：

    ```javascript
    const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. 调用容器 **item** 类的[`read`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-read)方法，并将 **itemId** 和 **partitionKey** 变量作为方法参数传入：

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();
    ```

    > &#128161；`read` 方法支持你对容器中的项执行点读取操作。 该方法需要 `itemId` 和 `partitionKey` 参数来标识要读取的项。 相比于使用 Cosmos DB 的 SQL 查询语言执行查询来查找单个项，`read` 方法可以更加高效且经济有效地检索单个项。 点读取可以直接读取数据，不需要查询引擎来处理请求。

1. 使用格式化输出字符串打印 saddle 对象：

    ```javascript
    print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
    ```

1. 完成后，代码文件现在应包含：

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();
    
        console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **保存**并再次运行脚本：

    ```bash
    node script.js
    ```

1. 查看终端输出。 具体而言，请查看带有项的 id、Name和Price的格式化输出文本。

## 使用 SDK 执行更新和删除点操作

在学习 SDK 期间，使用联机 Azure Cosmos DB 帐户或仿真器来更新项，在执行操作时在数据资源管理器与所选的 IDE 之间来回切换，以及查看更改是否已应用，是很常见的情况。 在本实验室中，当你使用 SDK 更新和删除项时，你就会这样做。

1. 返回 Web 浏览器窗口或选项卡。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”**** 中，展开“cosmicworks”**** 数据库节点，然后在“NoSQL API”**** 导航树中展开新“products”**** 容器节点。

1. 选择“Items”节点。 选择容器中的唯一项，然后观察项的 name 和 price 属性的值。

    | **属性** | **值** |
    | ---: | :--- |
    | **名称** | *Road Saddle* |
    | **价格** | *45.99 美元* |

    > &#128221; 此时，自从你创建了这个项后，这些值不应发生更改。 在本练习中，你将更改这些值。

1. 返回到 Visual Studio Code。 返回到 **script.js** 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

    ```javascript
    console.log(`[${saddle.id}]\t${saddle.name} (${saddle.price})`);
    ```

1. 通过将 price 属性的值设置为 32.55 来更改 saddle 变量：

    ```javascript
    // Update the item
    saddle.price = 32.55;
    ```

1. 通过将 name 属性的值设置为 Road LL Saddle，再次修改 saddle 变量：

    ```javascript
    saddle.name = "Road LL Saddle";
    ```

1. 调用容器 **items** 类的[`replace`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-replace)方法，并将 **saddle** 变量作为方法参数传入：

    ```javascript
    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. 完成后，代码文件现在应包含：

    ```javascript
    const { CosmosClient } = require("@azure/cosmos");
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
    
        const itemId = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
        const partitionKey = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    
        // Read the item
        const { resource: saddle } = await container.item(itemId, partitionKey).read();

        // Update the item
        saddle.price = 32.55;
        saddle.name = "Road LL Saddle";
    
        await container.item(saddle.id, partitionKey).replace(saddle);
    }
    
    main().catch((error) => console.error(error));
    ```

1. **保存**并再次运行脚本：

    ```bash
    node script.js
    ```

1. 返回 Web 浏览器窗口或选项卡。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”**** 中，展开“cosmicworks”**** 数据库节点，然后在“NoSQL API”**** 导航树中展开新“products”**** 容器节点。

1. 选择“Items”节点。 选择容器中的唯一项，然后观察项的 name 和 price 属性的值。

    | **属性** | **值** |
    | ---: | :--- |
    | **名称** | *Road LL Saddle* |
    | **价格** | *32.55 美元* |

    > &#128221; 此时，自从你观察到这个项后，这些值应发生更改。

1. 返回到 Visual Studio Code。 返回到 **script.js** 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

    ```javascript
    // Read the item
    const { resource: saddle } = await container.item(itemId, partitionKey).read();

    // Update the item
    saddle.price = 32.55;
    saddle.name = "Road LL Saddle";

    await container.item(saddle.id, partitionKey).replace(saddle);
    ```

1. 调用容器 **item** 类的[`delete`](https://learn.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#@azure-cosmos-item-delete)方法，并将 **itemId** 和 **partitionKey** 变量作为方法参数传入：

    ```javascript
    // Delete the item
    await container.item(itemId, partitionKey).delete();
    ```

1. 保存并再次运行脚本：

    ```bash
    node script.js
    ```

1. 关闭集成终端。

1. 返回 Web 浏览器窗口或选项卡。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”**** 中，展开“cosmicworks”**** 数据库节点，然后在“NoSQL API”**** 导航树中展开新“products”**** 容器节点。

1. 选择“Items”节点。 应会看到项列表现在为空。

1. 关闭 Web 浏览器窗口或选项卡。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
