---
title: 03 - 使用 Azure Cosmos DB for NoSQL SDK 创建和更新文档
lab:
  title: 03 - 使用 Azure Cosmos DB for NoSQL SDK 创建和更新文档
  module: Implement Azure Cosmos DB for NoSQL point operations
layout: default
nav_order: 6
parent: Python SDK labs
---

# 使用 Azure Cosmos DB for NoSQL SDK 创建和更新文档

该`azure-cosmos`库包括在 Azure Cosmos DB for NoSQL 容器中创建、检索、更新和删除 (CRUD) 项的操作方法。 这些方法共同执行 NoSQL API 容器中各种项的最常见“CRUD”操作。

在本实验室中，你将在 Azure Cosmos DB for NoSQL 容器中使用 SDK 对项执行日常 CRUD 操作。

## 准备开发环境

如果你还没有克隆**使用 Azure Cosmos DB 生成助手**的实验室代码存储库并设置你的本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明进行操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的“使用 Azure Cosmos DB 构建 Copilots”**** 实验室创建 Azure Cosmos DB for NoSQL 帐户，你可以直接在此实验室中使用它，并跳至[下一部分](#install-the-azure-cosmos-library)。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，创建一个将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并通过向帐户分配 **Cosmos DB 内置数据参与者**角色，授予你的用户标识访问权限，以管理帐户中的数据。

## 安装 azure-cosmos 库

**azure-cosmos** 库在 **PyPI** 上可用，可轻松安装到 Python 项目中。

1. 在 **Visual Studio Code** 中打开“资源管理器”**** 窗格，浏览 "python/03-sdk-crud" **** 文件夹。

1. 点击“python/03-sdk-crud”**** 文件夹，打开上下文菜单，然后选择“在集成终端中打开”****，启动新的终端实例。

    > &#128221；此命令将打开起始目录已设置为 "python/03-sdk-crud" **** 文件夹的终端。

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

## 使用 azure-cosmos 库

使用新创建帐户中的凭据，你将连接到 SDK 类，并创建新的数据库和容器实例。 然后，你将使用数据资源管理器验证这些实例是否存在于 Azure 门户中。

1. 在 **Visual Studio Code** 中打开“资源管理器”**** 窗格，浏览 "python/03-sdk-crud" **** 文件夹。

1. 打开名为 **script.py** 的空 Python 文件。

1. 添加以下`import`语句以导入 **PartitionKey** 类：

   ```python
   from azure.cosmos import PartitionKey
   ```

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

1. 添加以下代码，若数据库和容器尚未存在，则进行创建：

   ```python
   # Create database
   database = await client.create_database_if_not_exists(id="cosmicworks")
    
   # Create container
   container = await database.create_container_if_not_exists(
       id="products",
       partition_key=PartitionKey(path="/categoryId"),
       offer_throughput=400
   )
   ```

1. 在`main`方法下方，添加以下代码，使用`asyncio`库运行`main`方法：

   ```python
   if __name__ == "__main__":
       asyncio.run(query_items_async())
   ```

1. **script.py** 文件现在应如下所示：

   ```python
   from azure.cosmos import PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )

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

1. 切回到 Web 浏览器窗口。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”**** 中，展开“cosmicworks”**** 数据库节点，然后在“NoSQL API”**** 导航树中观察新“products”**** 容器节点。

## 使用 SDK 对项执行创建和读取点操作

现在，你将在 **ContainerProxy** 类中使用一组方法，对 NoSQL API 容器中的项执行常见操作。

1. 返回到 **Visual Studio Code**。 如果尚未打开，请打开 **python/03-sdk-crud** 文件夹下的 **script.py** 代码文件。

1. 创建新产品项并将其分配给具有以下属性的名为 **saddle** 的变量：

    | 属性 | 值 |
    | ---: | :--- |
    | **id** | 706cd7c6-db8b-41f9-aea2-0e0c7e8eb009 |
    | categoryId | 9603ca6c-9e28-4a02-9194-51cdb7fea816 |
    | **name** | *Road Saddle* |
    | **price** | 45.99d |
    | **tags** | { tan, new, crisp } |

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
   ```

1. 调用作为方法参数传入 **saddle** 变量的 **container** 变量的 [`create_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-create-item) 方法。

   ```python
   await container.create_item(body=saddle)
   ```

1. 完成后，代码文件现在应包含：
  
   ```python
   from azure.cosmos import PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )
        
           saddle = {
               "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
               "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
               "name": "Road Saddle",
               "price": 45.99,
               "tags": ["tan", "new", "crisp"]
           }
            
           await container.create_item(body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **保存**并再次运行脚本：

   ```bash
   python script.py
   ```

1. 观察**数据资源管理器**中的新项。

1. 返回到 **Visual Studio Code**。

1. 返回到 **script.py** 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

   ```python
   saddle = {
       "id": "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
       "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816",
       "name": "Road Saddle",
       "price": 45.99,
       "tags": ["tan", "new", "crisp"]
   }
    
   await container.create_item(body=saddle)
   ```

1. 创建一个名为 **item_id** 的字符串变量，其值为 **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009**：

   ```python
   item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
   ```

1. 创建一个名为 **partition_key** 的字符串变量，其值为 **9603ca6c-9e28-4a02-9194-51cdb7fea816**：

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. 调用作为方法参数传入 **item_id** 和 **partition_key** 变量的 **container** 变量的 [`read_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-read-item) 方法。

   ```python
   # Read item    
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
   ```

    > &#128161; `read_item` 方法支持你对容器中的项执行点读取操作。 该方法需要 `item_id` 和 `partition_key` 参数来标识要读取的项。 相比于使用 Cosmos DB 的 SQL 查询语言执行查询来查找单个项，`read_item` 方法可以更加高效且经济有效地检索单个项。 点读取可以直接读取数据，不需要查询引擎来处理请求。

1. 使用格式化输出字符串打印 saddle 对象：

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. 完成后，代码文件现在应包含：

   ```python
   from azure.cosmos import PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )
       
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **保存**并再次运行脚本：

   ```bash
   python script.py
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

1. 返回到 Visual Studio Code。 返回到 **script.py** 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

   ```python
   print(f'[{saddle["id"]}]\t{saddle["name"]} ({saddle["price"]})')
   ```

1. 通过将 price 属性的值设置为 32.55 来更改 saddle 变量：

   ```python
   saddle["price"] = 32.55
   ```

1. 通过将 name 属性的值设置为 Road LL Saddle，再次修改 saddle 变量：

   ```python
   saddle["name"] = "Road LL Saddle"
   ```

1. 调用作为方法参数传入 **item_id** 和 **saddle** 变量的 **container** 变量的 [`replace_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-replace-item) 方法。

   ```python
   await container.replace_item(item=item_id, body=saddle)
   ```

1. 完成后，代码文件现在应包含：

   ```python
   from azure.cosmos import PartitionKey
   from azure.cosmos.aio import CosmosClient
   from azure.identity.aio import DefaultAzureCredential
   import asyncio

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   async def main():
       async with CosmosClient(endpoint, credential=credential) as client:
           # Create database
           database = await client.create_database_if_not_exists(id="cosmicworks")
    
           # Create container
           container = await database.create_container_if_not_exists(
               id="products",
               partition_key=PartitionKey(path="/categoryId")
           )
        
           item_id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009"
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
    
           # Read item
           saddle = await container.read_item(item=item_id, partition_key=partition_key)
            
           saddle["price"] = 32.55
           saddle["name"] = "Road LL Saddle"
    
           await container.replace_item(item=item_id, body=saddle)

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. **保存**并再次运行脚本：

   ```bash
   python script.py
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

1. 返回到 Visual Studio Code。 返回到 **script.py** 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

   ```python
   # Read item
   saddle = await container.read_item(item=item_id, partition_key=partition_key)
    
   saddle["price"] = 32.55
   saddle["name"] = "Road LL Saddle"
    
   await container.replace_item(item=item_id, body=saddle)
   ```

1. 调用作为方法参数传入 **item_id** 和 **partition_key** 变量的 **container** 变量的 [`delete_item`](https://learn.microsoft.com/python/api/azure-cosmos/azure.cosmos.container.containerproxy?view=azure-python#azure-cosmos-container-containerproxy-delete-item) 方法。

   ```python
   # Delete the item
   await container.delete_item(item=item_id, partition_key=partition_key)
   ```

1. 保存并再次运行脚本：

   ```bash
   python script.py
   ```

1. 关闭集成终端。

1. 返回 Web 浏览器窗口或选项卡。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”**** 中，展开“cosmicworks”**** 数据库节点，然后在“NoSQL API”**** 导航树中展开新“products”**** 容器节点。

1. 选择“Items”节点。 应会看到项列表现在为空。

1. 关闭 Web 浏览器窗口或选项卡。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
