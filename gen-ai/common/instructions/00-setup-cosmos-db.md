---
lab:
  title: 安装 Azure Cosmos DB
  module: Setup
---

# 安装 Azure Cosmos DB

在本练习中，你将创建一个 Azure Cosmos DB for NoSQL 帐户，并在整个实验室模块中使用。随后，你将该账号分配至 **Cosmos DB 内置数据参与者** 角色，授予用户标识访问权限，管理帐户数据。 支持使用 Azure 身份验证从实验室代码中访问数据库，无需存储和管理密钥。

## 创建 Azure Cosmos DB for NoSQL 帐户

Azure Cosmos DB 是一项基于云的 NoSQL 数据库服务，它支持多个 API。 首次预配 Azure Cosmos DB 帐户时，你可以选择该帐户支持的 API。 Azure Cosmos DB for NoSQL 帐户完成预配后，你可以检索终结点和密钥。然后，使用终结点和密钥，借助 Azure SDK for Python 或所选其他 SDK 连接 Azure Cosmos DB for NoSQL 帐户。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭证登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB for NoSQL”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | 值 |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **帐户名** | 输入全局唯一名称 |
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *无服务器* |
    | **应用免费分级折扣** | *不应用* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“键”窗格。

1. 此窗格包含从 SDK 连接到帐户所需的连接详细信息和凭据。 具体而言：

    1. 复制 **URI** 字段并保存到文本编辑器中，供后续使用。 稍后在本练习中将用到此终结点值。

1. 请不要关闭浏览器标签页，继续进行下一步操作。

## 为用户标识分配 Cosmos DB 内置数据参与者 RBAC（基于角色的访问控制）角色

本练习中的最后任务是将 Microsoft Entra ID 用户标识分配至 **Cosmos DB 内置数据参与者** RBAC 角色，授予访问权限，管理 Azure Cosmos DB for NoSQL 帐户的数据。 支持使用 Azure 身份验证从代码中访问数据库，无需存储和管理密钥。

> &#128221;利用 Microsoft Entra ID 基于角色的访问控制 (RBAC) 完成 Azure 服务（如 Azure Cosmos DB）完成身份验证，相较基于密钥的方法，具备更多显著优势。 Entra ID RBAC 针对用户角色定制精确的访问控制，提升安全性，有效降低未经授权的访问风险。 它还简化用户管理，管理员可以动态分配和修改权限，免去加密密钥分发与维护的麻烦。 此外，此方法遵循组织政策要求，促进全面的访问监控和审查，增强合规性与可审核性。 Entra ID RBAC 简化安全访问管理，为 Azure 服务提供更高效且可扩展的解决方案。

1. 在 [Azure 门户](https://portal.azure.com)的顶部菜单中打开 Cloud Shell。

    ![Azure 门户工具栏上，高亮显示 Cloud Shell 图标。](media/azure-portal-toolbar-cloud-shell.png)

1. 在 Cloud Shell 提示符下，运行`az account set -s <SUBSCRIPTION_ID>`，确保练习订阅用于后续命令，并将`<SUBSCRIPTION_ID>`占位符令牌替换为当前练习使用的订阅 ID。

1. 复制上述命令的输出，作为下方`az cosmosdb sql role assignment create`命令中的`<PRINCIPAL_OBJECT_ID>`令牌使用。

1. 接下来，检索 **Cosmos DB 内置数据参与者**角色的定义 ID。 运行以下命令，确保替换`<RESOURCE_GROUP_NAME>`和`<COSMOS_DB_ACCOUNT_NAME>`令牌。

    ```bash
    az cosmosdb sql role definition list --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>"
    ```

    查看输出并找到名为 Cosmos DB 内置数据参与者**** 的角色定义。 输出包含属性中 `name` 角色定义的唯一标识符。 记下此值，因为后面下一步骤中的工作分配步骤需要使用此值。

1. 现在，你已准备好向自己分配 **Cosmos DB 内置数据参与者**角色定义。 在提示符下输入以下命令，确保替换 `<RESOURCE_GROUP_NAME>` 和 `<COSMOS_DB_ACCOUNT_NAME>` 令牌。

    > &#128221; 在以下命令中，`role-definition-id` 设置为 `00000000-0000-0000-0000-000000000002`，即 **Cosmos DB 内置数据参与者**角色定义的默认值。 如果从 `az cosmosdb sql role definition list` 命令检索到的值不同，则在执行前替换以下命令中的值。 `az ad signed-in-user show` 命令会检索已登录 Entra ID 用户的对象 ID。

    ```bash
    az cosmosdb sql role assignment create --resource-group "<RESOURCE_GROUP_NAME>" --account-name "<COSMOS_DB_ACCOUNT_NAME>" --role-definition-id "00000000-0000-0000-0000-000000000002" --principal-id $(az ad signed-in-user show --query id -o tsv) --scope "/"
    ```

1. 命令运行完成后，你将能够在本地运行代码，以便与存储在 Cosmos DB NoSQL 数据库中的数据进行交互。

1. 关闭 Cloud Shell。
