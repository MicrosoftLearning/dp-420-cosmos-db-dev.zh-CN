---
title: 01 - 通过 SDK 连接 Azure Cosmos DB for NoSQL
lab:
  title: 01 - 通过 SDK 连接 Azure Cosmos DB for NoSQL
  module: Use the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 4
parent: JavaScript SDK labs
---

# 通过 SDK 连接 Azure Cosmos DB for NoSQL

Azure SDK for JavaScript（Node.js 和浏览器）是一套客户端库，提供一致的开发人员界面来与多种 Azure 服务进行交互。 客户端库是用于使用这些资源并与这些资源进行交互的包。

在此实验室中，你将使用 Azure SDK for JavaScript 连接到 Azure Cosmos DB for NoSQL 帐户。

## 准备开发环境

如果你还没有克隆**使用 Azure Cosmos DB 生成助手**的实验室代码存储库并设置你的本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明进行操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的**使用 Azure Cosmos DB 生成助手**实验室创建了 Azure Cosmos DB for NoSQL 帐户，则可以将其用于此实验室，并跳转到[下一部分](#import-the-azurecosmos-library)。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，创建一个将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并通过向帐户分配 **Cosmos DB 内置数据参与者**角色，授予你的用户标识访问权限，以管理帐户中的数据。

## 导入 @azure/cosmos 库

**@azure/cosmos** 库在 **npm** 上可用，可轻松安装到 JavaScript 项目中。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **javascript/01-sdk-connect** 文件夹。

1. 打开 **javascript/01-sdk-connect** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为 **javascript/01-sdk-connect** 文件夹的终端。

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

## 使用 @azure/cosmos 库

导入 Azure SDK for JavaScript 中的 Azure Cosmos DB 库后，可以立即使用 其类连接到 Azure Cosmos DB for NoSQL 帐户。 **CosmosClient** 类是一个核心类，用于与 Azure Cosmos DB for NoSQL 帐户建立初始连接。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **javascript/01-sdk-connect** 文件夹。

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

1. 添加名为 **main** 的 `async` 函数以读取和输出帐户属性：

    ```javascript
    async function main() {
        const { resource: account } = await client.getDatabaseAccount();
        console.log(`Consistency Policy: ${account.consistencyPolicy}`);
        console.log(`Primary Region: ${account.writableLocations[0].name}`);
    }
    ```

1. 调用 **main** 函数：

    ```javascript
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
        const { resource: account } = await client.getDatabaseAccount();
        console.log(`Consistency Policy: ${account.consistencyPolicy}`);
        console.log(`Primary Region: ${account.writableLocations[0].name}`);
    }

    main().catch((error) => console.error(error));
    ```

1. **保存** **script.cs** 文件。

## 测试脚本

现在用于连接到 Azure Cosmos DB for NoSQL 帐户的 JavaScript 代码已完成，可以测试脚本。 此脚本将输出默认一致性级别和第一个可写区域的名称。 创建帐户后，你指定了一个位置，应会看到输出为此脚本结果的同一位置值。

1. 在 **Visual Studio Code** 中，打开 **javascript/01-sdk-connect** 文件夹的关联菜单，然后选择“**在集成终端中打开**”以打开一个新的终端实例。

1. 运行脚本之前，必须使用`az login`命令登录 Azure。 在“终端”窗口中，运行：

    ```bash
    az login
    ```

1. 使用 `node` 命令运行脚本：

    ```bash
    node script.js
    ```

1. 该脚本现在将输出帐户的确认一致性级别，以及第一个可写区域。 例如，如果帐户的默认一致性级别为“**会话**”，并且第一个可写区域为“**美国东部**”，脚本将输出：

    ```text
    Consistency Policy: Session
    Primary Region: East US
    ```

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
