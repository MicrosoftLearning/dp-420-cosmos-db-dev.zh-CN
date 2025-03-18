---
lab:
  title: 07.1 - 启用 Azure Cosmos DB for NoSQL 的矢量搜索功能
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
---

# 启用 Azure Cosmos DB for NoSQL 的矢量搜索功能

Azure Cosmos DB for NoSQL 提供高效的矢量索引和搜索功能，旨在高效、准确地存储和查询高维矢量，适用于各种规模的应用。 使用这项功能之前，请确保帐户已启用*矢量搜索 NoSQL API* 功能。

在本实验室中，你将创建一个 Azure Cosmos DB for NoSQL 帐户，并启用矢量搜索功能，目的是准备数据库，将其用作矢量存储位置。

## 准备开发环境

如果尚未为**使用 Azure Cosmos DB 生成 Copilot** 克隆实验室代码存储库并设置本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明以执行此操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的 **使用 Azure Cosmos DB 生成 Copilot** 实验室创建 Azure Cosmos DB for NoSQL 帐户，则可以将其用于此实验室，并跳到“[下一部分](#enable-vector-search-for-nosql-api)”。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，创建一个将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并通过向帐户分配 **Cosmos DB 内置数据参与者**角色，授予你的用户标识访问权限，以管理帐户中的数据。

## 启用 NoSQL 矢量搜索功能

在本任务中，你将使用 Azure CLI 在 Azure Cosmos DB 帐户中启用*矢量搜索 NoSQL API *功能。

1. 在 [Azure 门户](https://portal.azure.com)的顶部菜单中打开 Cloud Shell。

    ![Azure 门户工具栏上，高亮显示 Cloud Shell 图标。](media/07-azure-portal-toolbar-cloud-shell.png)

2. 在 Cloud Shell 提示符下，运行`az account set -s <SUBSCRIPTION_ID>`，确保练习订阅用于后续命令，并将`<SUBSCRIPTION_ID>`占位符令牌替换为当前练习使用的订阅 ID。

3. 在 Azure Cloud Shell 中执行以下命令启用 *NoSQL API* 矢量搜索功能，替换`<RESOURCE_GROUP_NAME>`和`<COSMOS_DB_ACCOUNT_NAME>`令牌为资源组及 Azure Cosmos DB 帐户名称。

     ```bash
     az cosmosdb update \
       --resource-group <RESOURCE_GROUP_NAME> \
       --name <COSMOS_DB_ACCOUNT_NAME> \
       --capabilities EnableNoSQLVectorSearch
     ```

4. 等待命令成功执行后再退出 Cloud Shell。

5. 关闭 Cloud Shell。

## 创建数据库和容器托管矢量

1. 在 [Azure 门户](https://portal.azure.com)中的 Azure Cosmos DB 帐户左侧菜单选择“数据资源管理器”****，然后选择“新建容器”****。

2. 在“新建容器”**** 对话框中：
   1. 在“数据库 ID”下****，选择“新建”****，并在数据库 ID 字段中输入 "CosmicWorks"。
   2. 在“容器 ID”**** 框中，输入名称 "Products"。
   3. 将 "/category" 指定为**分区键**。

      ![上面指定的新容器设置已经输入到对话框中的屏幕截图。](media/07-azure-cosmos-db-new-container.png)

   4. 滚动至“新建容器”**** 话框底部，展开“容器矢量策略”****，选择“添加矢量嵌入”****。

   5. 在“容器矢量策略”**** 设置部分，配置以下内容：

      | 设置 | “值” |
      | ------- | ----- |
      | **Path** | 输入 */embedding*。 |
      | **Data type** | 选择 *float32*。 |
      | **距离函数** | 选择 *余弦*。 |
      | **Dimensions** | 输入 *1536*，匹配 OpenAI `text-embedding-3-small`模型生成的维度数。 |
      | **索引类型** | 选择 *diskANN*。 |
      | **量化字节大小** | 将其留空。 |
      | **索引搜索列表大小** | 接受默认值 *100*。 |

      ![上面指定的容器矢量策略的屏幕截图，其中输入了“新建容器”对话框。](media/07-azure-cosmos-db-container-vector-policy.png)

   6. 选择“确定”创建数据库和容器。

   7. 等待容器创建完成后再继续。 容器准确就绪可能需要几分钟时间。
