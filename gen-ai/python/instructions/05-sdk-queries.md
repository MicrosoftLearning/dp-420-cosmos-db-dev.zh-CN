---
title: 05 - 使用 Azure Cosmos DB for NoSQL SDK 执行查询
lab:
  title: 05 - 使用 Azure Cosmos DB for NoSQL SDK 执行查询
  module: Query the Azure Cosmos DB for NoSQL
layout: default
nav_order: 8
parent: Python SDK labs
---

# 使用 Azure Cosmos DB for NoSQL SDK 执行查询

最新版本的适用于 Azure Cosmos DB for NoSQL 的 Python SDK 简化容器查询，并利用 Python 的现代特性遍历结果集。

该`azure-cosmos`库内置功能，查询 Azure Cosmos DB 高效简单。

在此实验室中，你将使用迭代器遍历从 Azure Cosmos DB for NoSQL 返回的大型结果集。 你将使用 Python SDK 执行查询并遍历结果。

## 准备开发环境

如果你还没有克隆**使用 Azure Cosmos DB 生成助手**的实验室代码存储库并设置你的本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明进行操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的**使用 Azure Cosmos DB 生成助手**实验室创建了 Azure Cosmos DB for NoSQL 帐户，你可以将其用于此实验室，并跳转至[下一部分](#create-azure-cosmos-db-database-and-container-with-sample-data)。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，创建一个将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并通过向帐户分配 **Cosmos DB 内置数据参与者**角色，授予你的用户标识访问权限，以管理帐户中的数据。

## 创建 Azure Cosmos DB 数据库和包含示例数据的容器

如果你已创建一个名为 **cosmicworks-full** 的 Azure Cosmos DB 数据库，并且其中的容器 **products** 已预加载示例数据，则可以在此实验室中使用该数据库，并跳至[下一部分](#install-the-azure-cosmos-library)。 否则，请按照以下步骤创建新的示例数据库和容器。

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

## 安装 azure-cosmos 库

**azure-cosmos** 库在 **PyPI** 上可用，可轻松安装到 Python 项目中。

1. 在 **Visual Studio Code** 中打开“资源管理器”**** 窗格，浏览 "python/05-sdk-queries" **** 文件夹。

1. 打开 **python/05-sdk-queries** 文件夹的关联菜单，然后选择 **“在集成终端中打开”**，启动新的终端实例。

    > &#128221；此命令将打开起始目录已设置为 "python/05-sdk-queries" **** 文件夹的终端。

1. 创建并激活虚拟环境以管理依赖项。

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. 运行以下命令安装 [azure-cosmos][pypi.org/project/azure-cosmos] 包。

   ```bash
   pip install azure-cosmos
   ```

1. 因为我们在使用 SDK 的异步版本，所以还需要安装`asyncio`库：

   ```bash
   pip install asyncio
   ```

1. SDK 的异步版本还需要库`aiohttp`。 使用以下命令安装它：

   ```bash
   pip install aiohttp
   ```

1. 使用以下命令安装 [azure-identity][pypi.org/project/azure-identity] 库，支持使用 Azure 身份验证连接 Azure Cosmos DB 工作区：

   ```bash
   pip install azure-identity
   ```

## 使用 SDK 来遍历 SQL 查询的结果

使用新创建帐户中的凭据，与 SDK 类建立连接，然后连接到之前步骤中配置的数据库和容器，接着使用 SDK 执行 SQL 查询并遍历结果。

现在，你将使用迭代器，在 Azure Cosmos DB 的分页结果中创建一个简单明了的循环。 SDK 会在后台管理源迭代器，确保正确调用后续请求。

1. 在 **Visual Studio Code** 中打开“资源管理器”**** 窗格，浏览 "python/05-sdk-queries" **** 文件夹。

1. 打开名为 **script.py** 的空 Python 文件。

1. 添加以下`import`语句以导入异步 **CosmosClient** 类、 **DefaultAzureCredential** 类和 **asyncio** 库：

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio
   ```

1. 更新名为 **endpoint** 和 **credential** 的变量，将 **endpoint** 值设置为之前创建的 Azure Cosmos DB 帐户的**终结点**。 **credential** 变量应设置为 **DefaultAzureCredential** 类的新实例：

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221；例如，如果终结点为：**https://dp420.documents.azure.com:443/**，则语句为：**endpoint = "https://dp420.documents.azure.com:443/"**。

1. 所有与 Cosmos DB 的交互从`CosmosClient`的实例开始。 使用异步客户端时，我们需要使用 async/await 关键字，并且只能在异步方法中使用。 创建名为 **main** 的新异步方法，并添加以下代码，使用 **endpoint** 和 **credential** 变量创建异步 **CosmosClient** 类的新实例：

   ```python
   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
   ```

    > &#128161；使用异步 **CosmosClient** 客户端时，需先预热，使用完毕后再关闭，确保正常操作。 我们建议使用上述代码中演示的 `async with` 关键字启动客户端 - 这些关键字会创建上下文管理器，自动处理客户端的预热、初始化和清理工作，无需手动操作。

1. 添加以下代码，连接到你之前创建的数据库和容器：

   ```python
   database = client.get_database_client("cosmicworks-full")
   container = database.get_container_client("products")
   ```

1. 创建名为 `sql` 且值为 `SELECT * FROM products p` 的查询字符串变量。

   ```python
   sql = "SELECT * FROM products p"
   ```

1. 使用`sql`变量作为构造函数的参数，调用[`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items)方法。

   ```python
   result_iterator = container.query_items(
       query=sql
   )
   ```

1. **query_items** 方法返回一个异步迭代器，存储在名为`result_iterator`的变量中。 这表明迭代器中的每个对象都是可等待对象，但尚未返回查询结果。 添加以下代码，创建异步 **for** 循环，在循环访问异步迭代器时等待每个查询结果，并输出每个项的`id`、`name` 和 `price`。

   ```python
   # Perform the query asynchronously
   async for item in result_iterator:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. 在`main`方法下方，添加以下代码，使用`asyncio`库运行`main`方法：

   ```python
   if __name__ == "__main__":
       asyncio.run(query_items_async())
   ```

1. **script.py** 文件现在应如下所示：

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:

           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products p"
            
           result_iterator = container.query_items(
               query=sql
           )
            
           # Perform the query asynchronously
           async for item in result_iterator:
               print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **保存** **script.py** 文件。

1. 运行脚本之前，必须使用`az login`命令登录 Azure。 在“终端”窗口中，运行：

   ```bash
   az login
   ```

1. 运行脚本创建数据库和容器

   ```bash
   python script.py
   ```

1. 该脚本现在会输出容器中的每个项

## 在逻辑分区内执行查询

在上一部分，你查询了容器中的所有项。 默认情况下，异步 **CosmosClient** 会执行跨分区查询。 因此，你执行的查询（`"SELECT * FROM products p"`）引起查询引擎扫描容器中的所有分区。 最佳做法是，始终在逻辑分区内执行查询，避免跨分区查询。 这样做最终能节省开支并提高性能。

在本部分，你将在查询中加入分区键，执行逻辑分区查询。

1. 返回到 **script.py** 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

   ```python
   result_iterator = container.query_items(
       query=sql
   )
    
   # Perform the query asynchronously
   async for item in result_iterator:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. 修改脚本，创建 **partition_key** 变量，存储 jersey 类别 ID 值。 添加 **partition_key** 参数至 **query_items** 方法。 这可确保查询在 jersey 类别的逻辑分区内执行。

   ```python
   partition_key = "C3C57C35-1D80-4EC5-AB12-46C57A017AFB"

   result_iterator = container.query_items(
       query=sql,
       partition_key=partition_key
   )
   ```

1. 在上一部分，你直接在异步迭代器上使用异步 for 循环。（`async for item in result_iterator:`） 这次，你将异步生成完整的实际查询结果列表。 此代码执行的操作与之前使用的 for-loop 示例相同。 添加以下代码行，生成结果列表并输出结果：

   ```python
   item_list = [item async for item in result_iterator]

   for item in item_list:
       print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")
   ```

1. **script.py** 文件现在应如下所示：

   ```python
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:

           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products p"
            
           partition_key = "C3C57C35-1D80-4EC5-AB12-46C57A017AFB"

           result_iterator = container.query_items(
               query=sql,
               partition_key=partition_key
           )
    
           # Perform the query asynchronously
           item_list = [item async for item in result_iterator]
    
           for item in item_list:
               print(f"[{item['id']}]   {item['name']}  ${item['price']:.2f}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **保存** **script.py** 文件。

1. 运行脚本创建数据库和容器

   ```bash
   python script.py
   ```

1. 该脚本现在会输出 jersey 类别中的每个项，有效执行分区内查询。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
