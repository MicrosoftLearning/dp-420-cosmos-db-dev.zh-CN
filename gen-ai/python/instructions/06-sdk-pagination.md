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

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **python/06-sdk-pagination** 文件夹。

1. 打开 **python/06-sdk-pagination** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为 **python/06-sdk-pagination** 文件夹的终端。

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

## 使用 SDK 对小型 SQL 查询结果集进行分页

处理查询结果时，必须确保代码处理所有结果页面，并在发出后续请求之前检查是否还有任何其他页面。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **python/06-sdk-pagination** 文件夹。

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

1. 使用 **SELECT * FROM products WHERE products.price > 500** 的值创建一个名为 **sql** 的 *string* 类型的新变量：

   ```python
   sql = "SELECT * FROM products WHERE products.price > 500"
   ```

1. 使用`sql`变量作为构造函数的参数，调用[`query_items`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-query-items)方法。 将 `max_item_count` 设置为 `50` 以限制每页中返回的项目数。

   ```python
   iterator = container.query_items(
       query=sql,
       max_item_count=50  # Set maximum items per page
   )
   ```

1. 创建异步 **for** 循环，该循环将以异步方式在迭代器对象上调用 [`by_page`](https://learn.microsoft.com/python/api/azure-core/azure.core.paging.itempaged?view=azure-python#azure-core-paging-itempaged-by-page) 方法。 每次调用时，此方法都会返回一个结果页。

   ```python
   async for page in iterator.by_page():
   ```

1. 在异步 **for** 循环中，将以异步方式循环访问分页结果并输出每个项目的 `id`、`name` 和 `price`。

   ```python
   async for product in page:
       print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")
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
           # Get database and container clients
           database = client.get_database_client("cosmicworks-full")
           container = database.get_container_client("products")
    
           sql = "SELECT * FROM products WHERE products.price > 500"
        
           iterator = container.query_items(
               query=sql,
               max_item_count=50  # Set maximum items per page
           )
        
           async for page in iterator.by_page():
               async for product in page:
                   print(f"[{product['id']}]    {product['name']}   ${product['price']:.2f}")

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

1. 该脚本现在一次将输出 50 个项目的页面。

    > &#128161; 查询将匹配 products 容器中的数百个项。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
