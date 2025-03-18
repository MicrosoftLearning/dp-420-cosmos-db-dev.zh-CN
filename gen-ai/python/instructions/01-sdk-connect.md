---
lab:
  title: 01 - 使用 SDK 连接到 Azure Cosmos DB for NoSQL
  module: Use the Azure Cosmos DB for NoSQL SDK
---

# 通过 SDK 连接 Azure Cosmos DB for NoSQL

Azure SDK for Python 是一套客户端库，提供一致的开发人员界面来与多种 Azure 服务进行交互。 客户端库是用于使用这些资源并与这些资源进行交互的包。

在此实验室中，你将使用 Azure SDK for Python 连接到 Azure Cosmos DB for NoSQL 帐户。

## 准备开发环境

如果尚未为**使用 Azure Cosmos DB 生成 Copilot** 克隆实验室代码存储库并设置本地环境，请查看[设置本地实验室环境](00-setup-lab-environment.md)说明以执行此操作。

## 创建 Azure Cosmos DB for NoSQL 帐户

如果已为此站点上的 **使用 Azure Cosmos DB 生成 Copilot** 实验室创建 Azure Cosmos DB for NoSQL 帐户，则可以将其用于此实验室，并跳到“[下一部分](#install-the-azure-cosmos-library)”。 否则，请查看[设置 Azure Cosmos DB](../../common/instructions/00-setup-cosmos-db.md) 说明，创建一个将在整个实验室模块中使用的 Azure Cosmos DB for NoSQL 帐户，并通过向帐户分配 **Cosmos DB 内置数据参与者**角色，授予你的用户标识访问权限，以管理帐户中的数据。

## 安装 azure-cosmos 库

**azure-cosmos** 库在 **PyPI** 上可用，可轻松安装到 Python 项目中。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **python/01-sdk-connect** 文件夹。

1. 打开 **python/01-sdk-connect** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开新的终端实例。

    > &#128221; 此命令将打开起始目录已设置为 **python/01-sdk-connect** 文件夹的终端。

1. 创建并激活虚拟环境以管理依赖项。

   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
   ```

1. 运行以下命令安装 [azure-cosmos][pypi.org/project/azure-cosmos] 包。

   ```bash
   pip install azure-cosmos
   ```

1. 使用以下命令安装 [azure-identity][pypi.org/project/azure-identity] 库，支持使用 Azure 身份验证连接 Azure Cosmos DB 工作区：

   ```bash
   pip install azure-identity
   ```

1. 关闭集成终端。

## 使用 azure-cosmos 库

导入 Azure SDK for Python 中的 Azure Cosmos DB 库后，可以立即使用其类连接到 Azure Cosmos DB for NoSQL 帐户。 **CosmosClient** 类是一个核心类，用于与 Azure Cosmos DB for NoSQL 帐户建立初始连接。

1. 在 **Visual Studio Code** 的“**资源管理器**”窗格中，浏览到 **python/01-sdk-connect** 文件夹。

1. 打开名为 **script.py** 的空 Python 文件。

1. 添加以下 `import` 语句以导入 **CosmosClient** 和 **DefaultAzureCredential** 类：

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential
   ```

1. 更新名为 **endpoint** 和 **credential** 的变量，将 **endpoint** 值设置为之前创建的 Azure Cosmos DB 帐户的**终结点**。 **credential** 变量应设置为 **DefaultAzureCredential** 类的新实例：

   ```python
   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()
   ```

    > &#128221；例如，如果终结点为：**https://dp420.documents.azure.com:443/**，则语句为：**endpoint = "https://dp420.documents.azure.com:443/"**。

1. 添加名为 **client** 的新变量，并使用 **endpoint** 和 **credential** 变量将其初始化为 **CosmosClient** 类的新实例：

   ```python
   client = CosmosClient(endpoint, credential=credential)
   ```

1. 添加名为 **main** 的函数以读取和打印帐户属性：

   ```python
   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
   ```

1. **script.py** 文件现在应如下所示：

   ```python
   from azure.cosmos import CosmosClient
   from azure.identity import DefaultAzureCredential

   endpoint = "<cosmos-endpoint>"
   credential = DefaultAzureCredential()

   client = CosmosClient(endpoint, credential=credential)

   def main():
       account_info = client.get_database_account()
       print(f"Consistency Policy:  {account_info.ConsistencyPolicy}")
       print(f"Primary Region: {account_info.WritableLocations[0]['name']}")

   if __name__ == "__main__":
       main()
    ```

1. **保存** **script.py** 文件。

## 测试脚本

现在用于连接到 Azure Cosmos DB for NoSQL 帐户的 Python 代码已完成，可以测试脚本。 此脚本将输出默认一致性级别和第一个可写区域的名称。 创建帐户后，你指定了一个位置，应会看到输出为此脚本结果的同一位置值。

1. 在 **Visual Studio Code** 中，打开 **python/01-sdk-connect** 文件夹的上下文菜单，然后选择“**在集成终端中打开**”以打开一个新的终端实例。

1. 运行脚本之前，必须使用`az login`命令登录 Azure。 在终端窗口处，运行：

   ```bash
   az login
   ```

1. 使用 `python` 命令运行脚本：

   ```bash
   python script.py
   ```

1. 该脚本现在将输出默认一致性级别和第一个可写区域。 例如，如果帐户的默认一致性级别为“**会话**”，并且第一个可写区域为“**美国东部**”，则脚本将输出：

   ```text
   Consistency Policy:   {'defaultConsistencyLevel': 'Session'}
   Primary Region: East US
   ```

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

[pypi.org/project/azure-cosmos]: https://pypi.org/project/azure-cosmos
[pypi.org/project/azure-identity]: https://pypi.org/project/azure-identity
