---
title: 02 - 配置 Azure Cosmos DB Python SDK 进行脱机开发
lab:
  title: 02 - 配置 Azure Cosmos DB Python SDK 进行脱机开发
  module: Configure the Azure Cosmos DB for NoSQL SDK
layout: default
nav_order: 5
parent: Python SDK labs
---

# 配置 Azure Cosmos DB Python SDK 进行脱机开发

Azure Cosmos DB 仿真器是一个本地工具，可以模拟 Azure Cosmos DB 服务，用于开发和测试。 该仿真器支持 NoSQL API，在使用 Azure SDK for Python 开发代码时，可以使用它来代替云服务。

在本实验室中，你将从 Azure SDK for Python 连接到 Azure Cosmos DB 仿真器。

## 准备开发环境

如果你还没有克隆**使用 Azure Cosmos DB 生成助手**的实验室代码存储库并设置你的本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明进行操作。

## 启动 Azure Cosmos DB 仿真器

如果使用托管实验室环境，它应该已安装该仿真器。 如果没有，请参阅[安装说明](https://docs.microsoft.com/azure/cosmos-db/local-emulator)来安装 Azure Cosmos DB 仿真器。 启动仿真器后，你可以检索连接字符串，并使用 Azure SDK for Python，通过该连接字符串连接到仿真器。

> &#128161; 可以选择安装[新的基于 Linux 的 Azure Cosmos DB 仿真器（预览版）](https://learn.microsoft.com/azure/cosmos-db/emulator-linux)，该仿真器可用作 Docker 容器。 它支持在各种处理器和操作系统上运行。

1. 启动 Azure Cosmos DB 仿真器。

    > 💡如果使用 Windows，则 Azure Cosmos DB 仿真器已固定到 Windows 任务栏和“开始”菜单。 如果仿真器未从固定的图标启动，请尝试通过双击 **C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe** 文件来打开它。

1. 等待仿真器自动打开默认浏览器并导航到 **https://localhost:8081/_explorer/index.html** 登陆页。

1. 在“**快速入门**”窗格中，记下**主连接字符串**。 稍后将用到此连接字符串。

> &#128221; 有时，即使仿真器正在运行，登陆页也不会成功加载。 如果发生这种情况，可以使用已知的连接字符串连接到仿真器。 已知的连接字符串是：`AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==`

## 安装 azure-cosmos 库

**azure-cosmos** 库在 **PyPI** 上可用，可轻松安装到 Python 项目中。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **python/02-sdk-offline** 文件夹。

1. 打开 **python/02-sdk-offline** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为 **python/02-sdk-offline** 文件夹的终端。

1. 创建并激活虚拟环境以管理依赖项。

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. 运行以下命令安装 [azure-cosmos][pypi.org/project/azure-cosmos] 包。

   ```bash
   pip install azure-cosmos
   ```

## 从 Python SDK 连接到仿真器

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **python/02-sdk-offline** 文件夹。

1. 打开名为 **script.py** 的空 Python 文件。

1. 添加以下代码以连接到仿真器、创建数据库并输出其 ID：

   ```python
   from azure.cosmos import CosmosClient, PartitionKey
   
   # Connection string for the Azure Cosmos DB Emulator
   connection_string = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="
    
   # Initialize the Cosmos client
   client = CosmosClient.from_connection_string(connection_string)
    
   # Create a database
   database_name = "cosmicworks"
   database = client.create_database_if_not_exists(id=database_name)
    
   # Print the database ID
   print(f"New Database: Id: {database.id}")
   ```

1. **保存** **script.py** 文件。

## 运行脚本

1. 使用 **Visual Studio Code** 中为此实验室设置 Python 环境所用的同一终端窗口。 如果你关闭了该窗口，可以打开 **python/02-sdk-offline** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

1. 使用 `python` 命令运行脚本：

   ```bash
   python script.py
   ```

1. 该脚本将在仿真器中创建一个名为 `cosmicworks` 的数据库。 应会看到如下所示的输出：

   ```text
   New Database: Id: cosmicworks
   ```

## 创建和查看新容器

可以扩展脚本以在数据库中创建容器。

### 更新的代码

1. 修改 `script.py` 文件，在文件底部添加以下代码以创建容器：

   ```python
   # Create a container
   container_name = "products"
   partition_key_path = "/categoryId"
   throughput = 400
    
   container = database.create_container_if_not_exists(
       id=container_name,
       partition_key=PartitionKey(path=partition_key_path),
       offer_throughput=throughput
   )
    
   # Print the container ID
   print(f"New Container: Id: {container.id}")
   ```

### 运行更新的脚本

1. 使用以下命令运行更新的脚本：

   ```bash
   python script.py
   ```

1. 该脚本将在仿真器中创建一个名为 `products` 的容器。 应会看到如下所示的输出：

   ```text
   New Container: Id: products
   ```

### 验证结果

1. 切换到打开仿真器的数据资源管理器所用的浏览器。

1. 刷新 **NoSQL API** 以查看新的 **cosmicworks** 数据库和 **products** 容器。

## 停止 Azure Cosmos DB 仿真器

使用完仿真器后，务必将其停止，以释放系统资源。 请根据你的操作系统执行以下步骤：

### 在 macOS 或 Linux 上：

如果在终端窗口中启动仿真器，请执行以下步骤：

1. 找到仿真器运行所在的终端窗口。

1. 按 `Ctrl + C` 以终止仿真器进程。

或者，如果需要手动停止仿真器进程：

1. 打开新的终端窗口。

1. 使用以下命令来查找仿真器进程：

   ```bash
   ps aux | grep CosmosDB.Emulator
   ```

在输出中标识仿真器进程的 **PID**（进程 ID）。 使用 kill 命令终止仿真器进程：

```bash
kill <PID>
```

### 在 Windows 上：

1. 在 Windows 系统托盘（任务栏上的时钟附近）中找到 Azure Cosmos DB 仿真器图标。

1. 右键单击仿真器图标可打开上下文菜单。

1. 选择“**退出**”以关闭仿真器。

> 💡 退出仿真器的所有实例可能需要一点时间。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
