---
lab:
  title: 04 - 批处理多点操作与 Azure Cosmos DB for NoSQL SDK
  module: Perform cross-document transactional operations with the Azure Cosmos DB for NoSQL
---

# 批处理多点操作与 Azure Cosmos DB for NoSQL SDK

`azure-cosmos` Python SDK 提供 `execute_item_batch` 方法以在单个逻辑步骤中执行多点操作。 这样，开发人员就可以高效地将多个操作捆绑在一起，并确定它们是否在服务器端成功完成。

在此实验室中，你将使用 Python SDK 执行双项批处理操作，这些操作将同时演示成功和失败的事务批处理。

## 准备开发环境

如果尚未为**使用 Azure Cosmos DB 生成 Copilot** 克隆实验室代码存储库并设置本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明以执行此操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的 **使用 Azure Cosmos DB 生成 Copilot** 实验室创建 Azure Cosmos DB for NoSQL 帐户，则可以将其用于此实验室，并跳到“[下一部分](#install-the-azure-cosmos-library)”。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，创建一个将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并通过向帐户分配 **Cosmos DB 内置数据参与者**角色，授予你的用户标识访问权限，以管理帐户中的数据。

## 安装 azure-cosmos 库

**azure-cosmos** 库在 **PyPI** 上可用，可轻松安装到 Python 项目中。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **python/04-sdk-batch** 文件夹。

1. 打开 **python/04-sdk-batch** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为 **python/04-sdk-batch** 文件夹的终端。

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
       asyncio.run(main())
   ```

1. **script.py** 文件现在应如下所示：

   ```python
   from azure.cosmos import exceptions, PartitionKey
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

## 创建事务批处理

首先，让我们创建一个简单的事务批处理，使其生成两个虚构的产品。 此批处理将使用相同的“旧配件”类别标识符，将 worn saddle 和 rusty handlebar 插入到容器中。 这两个项具有相同的逻辑分区键，确保我们将有一个成功的批处理操作。

1. 返回到 **Visual Studio Code**。 如果尚未打开，请在 **python/04-sdk-batch** 文件夹中打开 **script.py** 代码文件。

1. 创建两个代表产品的字典：**worn saddle** 和 **rusty handlebar**。 这两个项目共享相同的分区键值 **"9603ca6c-9e28-4a02-9194-51cdb7fea816"**。

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   ```

1. 定义分区键值。

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. 创建包含两个项目的批处理。

   ```python
   batch = [saddle, handlebar]
   ```

1. 使用 `container` 对象的 `execute_item_batch` 方法执行批处理，并输出批处理中每个项目的响应。

```python
try:
        # Execute the batch
        batch_response = await container.execute_item_batch(batch, partition_key=partition_key)

        # Print results for each operation in the batch
        for idx, result in enumerate(batch_response):
            status_code = result.get("statusCode")
            resource = result.get("resourceBody")
            print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
    except exceptions.CosmosBatchOperationError as e:
        error_operation_index = e.error_index
        error_operation_response = e.operation_responses[error_operation_index]
        error_operation = batch[error_operation_index]
        print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
    except Exception as ex:
        print(f"An error occurred: {ex}")
```

1. 完成后，代码文件现在应包含：
  
   ```python
   from azure.cosmos import exceptions, PartitionKey
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

           saddle = ("create", (
               {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           handlebar = ("create", (
               {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [saddle, handlebar]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
   ```

1. **保存**并再次运行脚本：

   ```bash
   python script.py
   ```

1. 输出应指示每个操作的成功状态代码。

## 创建错误的事务批处理

现在来创建一个故意出错的事务批处理。 此批处理将尝试插入两个具有不同逻辑分区键的项。 我们将在“旧配件”类别中创建一个 flickering strobe light，并在“新配件”类别中创建一个 new helmet。 按照定义，此请求应为错误请求，会在执行此事务时返回错误。

1. 返回到 **script.py** 代码文件的“编辑器”选项卡。

1. 删除以下代码行：

   ```python
   saddle = ("create", (
       {"id": "0120", "name": "Worn Saddle", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   handlebar = ("create", (
       {"id": "012A", "name": "Rusty Handlebar", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))

   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"

   batch = [saddle, handlebar]
   ```

1. 修改脚本，以使用不同的分区键值创建新的 **flickering strobe light** 和 **new helmet**。

   ```python
   light = ("create", (
       {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
   ))
   helmet = ("create", (
       {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
   ))
   ```

1. 定义批处理的分区键值。

   ```python
   partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
   ```

1. 创建包含这两个项目的新的批处理。

   ```python
   batch = [light, helmet]
   ```

1. 完成后，代码文件现在应包含：

   ```python
   from azure.cosmos import exceptions, PartitionKey
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

           light = ("create", (
               {"id": "012B", "name": "Flickering Strobe Light", "categoryId": "9603ca6c-9e28-4a02-9194-51cdb7fea816"},
           ))
           helmet = ("create", (
               {"id": "012C", "name": "New Helmet", "categoryId": "0feee2e4-687a-4d69-b64e-be36afc33e74"},
           ))
        
           partition_key = "9603ca6c-9e28-4a02-9194-51cdb7fea816"
        
           batch = [light, helmet]
            
           try:
               # Execute the batch
               batch_response = await container.execute_item_batch(batch, partition_key=partition_key)
        
               # Print results for each operation in the batch
               for idx, result in enumerate(batch_response):
                   status_code = result.get("statusCode")
                   resource = result.get("resourceBody")
                   print(f"Item {idx} - Status Code: {status_code}, Resource: {resource}")
           except exceptions.CosmosBatchOperationError as e:
               error_operation_index = e.error_index
               error_operation_response = e.operation_responses[error_operation_index]
               error_operation = batch[error_operation_index]
               print("Error operation: {}, error operation response: {}".format(error_operation, error_operation_response))
           except Exception as ex:
               print(f"An error occurred: {ex}")

   if __name__ == "__main__":
       asyncio.run(main())
    ```

1. **保存**并再次运行脚本：

   ```bash
   python script.py
   ```

1. 查看终端输出。 第二个项目（“New Helmet”）的状态代码应为 **400**，表示**错误请求**。 出现此错误的原因是该事务中的所有项都没有与事务批处理共享相同的分区键值。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
