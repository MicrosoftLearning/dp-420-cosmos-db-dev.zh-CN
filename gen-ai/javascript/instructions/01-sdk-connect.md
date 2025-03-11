---
lab:
  title: 01 - 使用 SDK 连接到 Azure Cosmos DB for NoSQL
  module: Use the Azure Cosmos DB for NoSQL SDK
---

# 通过 SDK 连接 Azure Cosmos DB for NoSQL

适用于 JavaScript 的 Azure SDK（Node.js 和浏览器）是一套客户端库，提供一致的开发者界面与多种 Azure 服务交互。 客户端库是用于使用这些资源并与之交互的包。

在此实验室中，使用适用于 JavaScript 的 Azure SDK 连接到 Azure Cosmos DB for NoSQL 帐户。

## 准备开发环境

如果尚未为**使用 Azure Cosmos DB 生成 Copilot** 克隆实验室代码存储库并设置本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明以执行此操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的 **使用 Azure Cosmos DB 生成 Copilot** 实验室创建 Azure Cosmos DB for NoSQL 帐户，则可以将其用于此实验室，并跳到“[下一部分](#import-the-azurecosmos-library)”。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，以创建将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并向用户标识授予访问权限以管理帐户中的数据，方法是将其分配给 **Cosmos DB 内置数据参与者**角色。

## 导入@azure/cosmos库

该**@azure/cosmos**库可在 **NPM** 上获取，可轻松安装到 JavaScript 项目中。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **javascript/01-sdk-connect** 文件夹。

1. 打开 **javascript/01-sdk-connect** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

    > #128221；此命令将打开起始目录已设置为 **javascript/01-sdk-connect** 文件夹的终端。

1. 初始化新 Node.js 项目：

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

从适用于 JavaScript 的 Azure SDK 中导入 Azure Cosmos DB 库后，可以立即使用其类连接到 Azure Cosmos DB for NoSQL 帐户。 **CosmosClient** 类是用于与 Azure Cosmos DB for NoSQL 帐户建立初始连接的核心类。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **javascript/01-sdk-connect** 文件夹。

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

1. 添加名为 **client** 的新变量，并使用 **endpoint** 和 **credential** 变量将其初始化为 **CosmosClient** 的新实例：

    ```javascript
    const client = new CosmosClient({ endpoint, aadCredentials: credential });
    ```

1. 添加名为 **main** 的`async`函数以读取和输出帐户属性：

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

1. **保存** **script.js** 文件。

## 测试脚本

连接到 Azure Cosmos DB for NoSQL 帐户的 JavaScript 代码已完成，现在可以测试脚本。 此脚本将输出默认一致性级别和第一个可写区域的名称。 创建帐户时指定了一个位置，应会看到与此脚本结果输出的位置值相同。

1. 在 **Visual Studio Code** 中，打开 **javascript/01-sdk-connect** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

1. 运行脚本之前，必须使用`az login`命令登录到 Azure。 在终端窗口处，运行：

    ```bash
    az login
    ```

1. 使用 `node` 命令运行脚本：

    ```bash
    node script.js
    ```

1. 该脚本现在将输出帐户的默认一致性级别和第一个可写区域。 例如，如果帐户的默认一致性级别为“**会话**”，并且第一个可写区域为“**美国东部**”，则脚本将输出：

    ```text
    Consistency Policy: Session
    Primary Region: East US
    ```

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[npmjs.com/package/@azure/cosmos]: https://www.npmjs.com/package/@azure/cosmos
[npmjs.com/package/@azure/identity]: https://www.npmjs.com/package/@azure/identity
