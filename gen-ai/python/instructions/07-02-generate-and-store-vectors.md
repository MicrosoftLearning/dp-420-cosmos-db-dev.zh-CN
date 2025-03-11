---
title: 07.2 - 使用 Azure OpenAI 生成矢量嵌入，存储在 Azure Cosmos DB for NoSQL 中
lab:
  title: 07.2 - 使用 Azure OpenAI 生成矢量嵌入，存储在 Azure Cosmos DB for NoSQL 中
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 11
parent: Python SDK labs
---

# 使用 Azure OpenAI 生成矢量嵌入，存储在 Azure Cosmos DB for NoSQL 中

Azure OpenAI 支持访问 OpenAI 的高级语言模型，包括`text-embedding-ada-002`、`text-embedding-3-small`和`text-embedding-3-large`模型。 利用这些模型之一，你可以生成文本数据的矢量表示，存储在类似 Azure Cosmos DB for NoSQL 的矢量存储中。 这有助于实现高效准确的相似性搜索，显著提升 copilot 检索相关信息并提供丰富上下文交互的能力。

在本实验室中，你将创建一个 Azure OpenAI 服务并部署嵌入模型。 然后，你将使用 Python 代码，利用各自的 Python SDK 创建 Azure OpenAI 和 Cosmos DB 客户端，以生成产品说明的矢量表示并写入数据库。

> &#128721；本模块中的前一个练习是本实验室的前提。 如果尚未完成该练习，请在继续前完成，它为本实验室提供必需的基础。

## 创建 Azure OpenAI 服务

Azure OpenAI 服务提供 REST API 访问 OpenAI 强大语言模型的权限。 这些模型可以轻松适应特定的任务，包括但不限于内容生成、汇总、图像理解、语义搜索和自然语言到代码的转换。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

2. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

3. 选择“创建资源”****，搜索 *Azure OpenAI*”，然后使用以下设置创建新的**Azure OpenAI**，其他设置保留默认值：

    | 设置 | 值 |
    | ------- | ----- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **区域** | *从[支持区域列表](https://learn.microsoft.com/azure/ai-services/openai/concepts/models?tabs=python-secure%2Cglobal-standard%2Cstandard-embeddings#tabpanel_3_standard-embeddings)中选择支持`text-embedding-3-small`模型的可用区域*。 |
    | **Name** | 输入全局唯一名称 |
    | **定价层** | *选择标准 0* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用现有的预先创建的资源组。

4. 等待部署任务完成，再继续执行下一个任务。

## 部署嵌入模型

使用 Azure OpenAI 生成嵌入前，必须在服务中部署所需的嵌入模型实例。

1. 在 Azure 门户（``portal.azure.com``）中导航至新创建 Azure OpenAI 服务。

2. 在 Azure OpenAI 服务的“**概述**”页上，选择工具栏上的“**转到 Azure AI Foundry 门户**”链接以启动“**Azure AI Foundry**”。

3. 在 Azure AI Foundry 中，从左侧菜单中选择“**部署**”。

4. 在“**模型部署**”页上，选择“**部署模型**”，然后从下拉列表中选择“**部署基础模型**”。

5. 从模型列表中选择“`text-embedding-3-small`”。

    > #128161；可以使用推理任务筛选器筛选列表以仅 显示*嵌入*模型。

    > #128221；如果未看到`text-embedding-3-small`模型，则可能选择了当前不支持该模型的 Azure 区域。 在这种情况下，可以对此实验室使用`text-embedding-ada-002`模型。 这两种模型都生成包含 1536 个维度的矢量，因此无需对在 Azure Cosmos DB 中的`Products`容器上定义的容器矢量策略进行更改。

6. 选择“确认”****，部署模型。

7. 在 Azure AI Foundry 的“模型部署”**** 页面，记录`text-embedding-3-small`“模型部署”的**名称**，稍后在本练习中会用到。

## 部署聊天完成模型

在嵌入模型基础上，copilot 还需部署聊天完成模型。 你将使用 OpenAI 的`gpt-4o` 大型语言模型生成 copilot 的回复。

1. 在 Azure AI Foundry 的“模型部署”**** 页面，再次点击“部署模型”**** 按钮，然后从下拉列表中选择“部署基础模型”****。

2. 从列表中选择 **gpt-4o** 聊天完成模型。

3. 选择“确认”****，部署模型。

4. 在 Azure AI Foundry 的“**模型部署**”页上，记下 `gpt-4o` 模型部署的“**名称**”，稍后在本练习中需要用到此名称。

## 分配认知服务 OpenAI 用户 RBAC 角色

要允许用户标识与 Azure OpenAI 服务交互，可以向帐户分配**认知服务 OpenAI 用户**角色。 Azure OpenAI 服务支持 Azure 基于角色的访问控制 (Azure RBAC)，这是用于管理对 Azure 资源的个人访问权限的授权系统。 使用 Azure RBAC，可以根据不同团队成员对指定项目的需求，为其分配不同级别的权限。

> &#128221; Microsoft Entra ID 的基于角色的访问控制 (RBAC) 用于对 Azure 服务（如 Azure OpenAI）进行身份验证，可通过针对用户角色定制的精确访问控制来增强安全性，从而有效降低未经授权的访问风险。 使用 Entra ID RBAC 简化安全访问管理，从而通过更高效且可缩放的解决方案来利用 Azure 服务。

1. 在 Azure 门户 (``portal.azure.com``) 中，导航到 Azure OpenAI 资源。

2. 在左侧导航窗格上，选择“**访问控制(IAM)**”。

3. 依次选择“+ 添加”和“添加角色分配” 。

4. 在“**角色**”选项卡上，选择“**认知服务 OpenAI 用户**”角色，然后选择“**下一步**”。

5. 在“**成员**”选项卡上，选择“将访问权限分配到用户、组或服务主体”，然后选择“**选择成员**”。

6. 在“**选择成员**”对话框中，搜索姓名或电子邮件地址，然后选择你的帐户。

7. 在“查看 + 分配”选项卡上，选择“查看 + 分配”，以分配角色 。

## 创建 Python 虚拟环境

Python 中的虚拟环境对于维护干净有序的开发空间至关重要，可使单个项目拥有独立于其他项目的自己的一组依赖项。 这可以防止不同项目之间的冲突，并确保开发工作流中的一致性。 通过使用虚拟环境，可以轻松管理包版本，避免依赖项冲突，并使项目顺利运行。 最佳做法是使编码环境保持稳定、可靠，使开发过程更高效且不容易出现问题。

1. 使用 Visual Studio Code，打开将**使用 Azure Cosmos DB 生成助手**学习模块的实验室代码存储库克隆到的文件夹。

2. 在 Visual Studio Code 中，打开新的终端窗口，并将目录更改为 `python/07-build-copilot` 文件夹。

3. 通过在终端提示符下运行以下命令来创建名为 `.venv` 的虚拟环境：

    ```bash
    python -m venv .venv 
    ```

    上述命令将在 `07-build-copilot` 文件夹下创建一个 `.venv` 文件夹，该文件夹将为本实验室中的练习提供专用 Python 环境。

4. 通过从下表中选择 OS 和 shell 的相应命令并在终端提示符下执行命令来激活虚拟环境。

    | 平台 | Shell | 激活虚拟环境的命令 |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

5. 安装 `requirements.txt` 中定义的库：

    ```bash
    pip install -r requirements.txt
    ```

    `requirements.txt` 文件包含一组将在整个实验室中使用的 Python 库。

    | 库 | 版本 | 说明 |
    | ------- | ------- | ----------- |
    | `azure-cosmos` | 4.9.0 | 适用于 Python 的 Azure Cosmos DB - 客户端库 |
    | `azure-identity` | 1.19.0 | 适用于 Python 的 Azure 标识 SDK |
    | `fastapi` | 0.115.5 | 用于通过 Python 生成 API 的 Web 框架 |
    | `openai` | 1.55.2 | 提供从 Python 应用访问 Azure OpenAI REST API 的权限。 |
    | `pydantic` | 2.10.2 | 使用 Python 类型提示进行数据验证。 |
    | `requests` | 2.32.3 | 发送 HTTP 请求。 |
    | `streamlit` | 1.40.2 | 将 Python 脚本转换为交互式 Web 应用。 |
    | `uvicorn` | 0.32.1 | 适用于 Python 的 ASGI Web 服务器实现。 |
    | `httpx` | 0.27.2 | 适用于 Python 的下一代 HTTP 客户端。 |

## 添加 Python 函数以矢量化文本

适用于 Azure OpenAI 的 Python SDK 提供对同步类和异步类的访问权限，这些类可用于为文本数据创建嵌入内容。 此功能可以封装在 Python 代码中的函数中。

1. 在 Visual Studio Code 的“**资源管理器**”窗格中，导航到 `python/07-build-copilot/api/app` 文件夹并打开其中的 `main.py` 文件。

    > &#128221; 此文件将用作在下一练习中生成的后端 Python API 的入口点。 在本练习中，你将提供一些异步函数，这些函数可用于将包含嵌入内容的数据导入将由 API 利用的 Azure Cosmos DB。

2. 要使用异步 Azure OpenAI SDK for Python，请通过将以下代码添加到 `main.py` 文件顶部来导入库：

   ```python
   from openai import AsyncAzureOpenAI
   ```

3. 你将使用 Azure 身份验证和之前为你的用户标识分配的 Entra ID RBAC 角色异步访问 Azure OpenAI 和 Cosmos DB。 在文件顶部的 `openai` 导入语句下方添加以下行，以从 `azure-identity` 库导入所需的类：

   ```python
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   ```

    > &#128221; 为确保可以从 API 安全地与 Azure 服务交互，你将使用适用于 Python 的 Azure 标识 SDK。 利用此方法，你可以避免存储代码中的密钥或与此类密钥交互，而是利用分配给帐户的 RBAC 角色，访问前面练习中的 Azure Cosmos DB 和 Azure OpenAI。

4. 创建变量来存储 Azure OpenAI API 版本和终结点，并将 `<AZURE_OPENAI_ENDPOINT>` 令牌替换为 Azure OpenAI 服务的终结点值。 此外，请为嵌入模型部署的名称创建变量。 在文件中的 `import` 语句下方插入以下代码：

   ```python
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   ```

    如果你的嵌入部署名称不同，请相应地更新分配给变量的值。

    > &#128161; 在撰写本文时，`2024-10-21` 的 API 版本是最新的正式发布版本。 可以使用该版本或新版本（如果其中一个可用）。 API 规范文档中有一个[包含最新 API 版本的表](https://learn.microsoft.com/azure/ai-services/openai/reference#api-specs)。

    > &#128221; `EMBEDDING_DEPLOYMENT_NAME` 是在 Azure AI Foundry 中部署 `text-embedding-3-small` 模型后记下的“**名称**”值。 如果需要引用它，请启动 Azure AI Foundry，导航到“**部署**”页，然后找到“**模型名称**”为 `text-embedding-3-small` 的部署。 之后，复制该项的“**名称**”字段值。 如果部署了 `text-embedding-ada-002` 模型，请使用该部署的名称。

5. 使用适用于 Python 的 Azure 标识 SDK 的 `DefaultAzureCredential` 类创建异步凭据，以通过在变量声明下方插入以下代码，使用 Microsoft Entra ID RBAC 身份验证来访问 Azure OpenAI 和 Azure Cosmos DB：

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

6. 要创建嵌入内容，请插入以下内容，这将添加一个函数，用于使用 Azure OpenAI 客户端生成嵌入内容：

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
   ```

    创建 Azure OpenAI 客户端不需要 `api_key` 值，因为它使用 Azure 标识 SDK 的 `get_bearer_token_provider` 类来检索持有者令牌。

7. `main.py` 文件现在应与以下类似：

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding
   ```

8. 保存 `main.py` 文件。

## 测试嵌入函数

要确保 `generate_embeddings` 文件中的 `main.py` 函数正常工作，请在文件底部添加几行代码，以便直接运行该文件。 借助这些行，你可以从命令行执行 `generate_embeddings` 函数，并传入要嵌入的文本。

1. 添加一个 **main guard** 块，其中包含对 `main.py` 文件底部的 `generate_embeddings` 的调用：

   ```python
   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

    > &#128221; `if __name__ == "__main__":` 块在 Python 中通常称为 **main guard** 或 **entry point**。 它可确保仅在直接运行脚本时执行某些代码，而不会在另一个脚本中作为模块导入时执行。 这种做法有助于组织代码，并使代码的可重用性和模块化更强。

2. 保存 `main.py` 文件，该文件现在应如下所示：

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding

   if __name__ == "__main__":
       import asyncio
       import sys
    
       async def main():
           print(await generate_embeddings(sys.argv[1]))
           # Close open async credential sessions
           await credential.close()
        
       asyncio.run(main())
   ```

3. 在 Visual Studio Code 中打开一个新的集成终端窗口。

4. 运行将请求发送到 Azure OpenAI 的 API 之前，必须使用 `az login` 命令登录到 Azure。 在“终端”窗口中，运行：

   ```bash
   az login
   ```

5. 在浏览器中完成登录过程。

6. 在终端提示符下，将目录更改为 `python/07-build-copilot`。

7. 通过使用下表中的命令激活虚拟环境，并为 OS 和 shell 选择适当的命令，来确保集成终端窗口在 Python 虚拟环境中运行。

    | 平台 | Shell | 激活虚拟环境的命令 |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

8. 在终端提示符处，将目录更改为 `api/app`，然后执行以下命令：

   ```python
   python main.py "Hello, world!"
   ```

9. 查看“终端”窗口中的输出。 你应该会看到一个浮点数数组，这是“Hello, world!”的矢量表示形式！ string。 它应该与以下缩写输出类似：

   ```bash
   [-0.019184619188308716, -0.025279032066464424, -0.0017195191467180848, 0.01884828321635723...]
   ```

## 生成用于将数据写入 Azure Cosmos DB 的函数

使用适用于 Python 的 Azure Cosmos DB SDK，可以创建一个支持将文档更新插入数据库的函数。 如果找到匹配项，更新插入操作将更新记录，如果未找到，则插入一条新记录。

1. 返回到 Visual Studio Code 中打开的 `main.py` 文件，并通过在文件中已存在的 `import` 语句正下方插入以下行，从适用于 Python 的 Azure Cosmos DB SDK 导入异步 `CosmosClient` 类：

   ```python
   from azure.cosmos.aio import CosmosClient
   ```

2. 添加另一条导入语句，以从 `api/app` 文件夹中的 *models* 模块引用 `Product` 类。 `Product` 类可定义 Cosmic Works 数据集中产品的形状。

   ```python
   from models import Product
   ```

3. 新建一组包含与 Azure Cosmos DB 关联的配置值的变量，并将其添加到 `main.py` 文件中之前插入的 Azure OpenAI 变量下方。 确保将 `<AZURE_COSMOSDB_ENDPOINT>` 令牌替换为 Azure Cosmos DB 帐户的终结点。

   ```python
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
   ```

4. 添加名为 `upsert_product` 的函数，以将文档更新插入（更新或插入）Cosmos DB 中，并将以下代码插入到 `main.py` 文件中 `generate_embeddings` 函数下方：

   ```python
   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
   ```

5. 保存 `main.py` 文件，该文件现在应如下所示：

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"

   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
    
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Create an async Azure OpenAI client
       async with AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       ) as client:
           response = await client.embeddings.create(
               input = text,
               model = EMBEDDING_DEPLOYMENT_NAME
           )
           return response.data[0].embedding

   async def upsert_product(product: Product):
       """Upserts the provided product to the Cosmos DB container."""
       # Create an async Cosmos DB client
       async with CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential) as client:
           # Load the CosmicWorks database
           database = client.get_database_client(DATABASE_NAME)
           # Retrieve the product container
           container = database.get_container_client(CONTAINER_NAME)
           # Upsert the product
           await container.upsert_item(product)
    
   if __name__ == "__main__":
       import sys
       print(generate_embeddings(sys.argv[1]))
   ```

## 矢量化示例数据

现在，你已准备好同时测试 `generate_embeddings` 函数和 `upsert_document` 函数。 为此，你将使用从 GitHub 下载包含 Cosmic Works 产品信息的示例数据文件的代码覆盖 `if __name__ == "__main__"` main guard 块，然后矢量化每个产品的 `description` 字段，并将文档更新插入到 Azure Cosmos DB 数据库中的 `Products` 容器。

> &#128221; 此方法用于演示使用 Azure OpenAI 生成嵌入内容以及在 Azure Cosmos DB 中存储嵌入内容的技术。 然而，在实际应用场景中，更加适合采用更可靠的方法（如使用由 Azure Cosmos DB 更改源触发的 Azure Function）将嵌入内容添加到现有文档和新文档。

1. 在 `main.py` 文件中，使用以下内容覆盖 `if __name__ == "__main__":` 代码块：

   ```python
   if __name__ == "__main__":
       import asyncio
       from models import Product
       import requests
    
       async def main():
           product_raw_data = "https://raw.githubusercontent.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs/refs/heads/main/data/07/products.json?v=1"
           products = [Product(**data) for data in requests.get(product_raw_data).json()]
    
           # Call the generate_embeddings function, passing in an argument from the command line.    
           for product in products:
               print(f"Generating embeddings for product: {product.name}", end="\r")
               product.embedding = await generate_embeddings(product.description)
               await upsert_product(product.model_dump())
    
           print("All products with vectorized descriptions have been upserted to the Cosmos DB container.")
           # Close open credential sessions
           await credential.close()
    
       asyncio.run(main())
   ```

2. 在 Visual Studio Code 中打开的集成终端提示符下，使用以下命令再次运行 `main.py` 文件：

   ```python
   python main.py
   ```

3. 等待代码执行完成，完成时将出现一条消息，指示带有矢量化说明的所有产品均已更新插入到 Cosmos DB 容器。 针对产品数据集中 295 条记录的矢量化和数据更新插入过程可能需要长达 10 - 15 分钟才能完成。 如果未插入所有产品，则可以使用上述命令重新运行 `main.py` 来添加剩余产品。

## 查看 Cosmos DB 中更新插入的示例数据

1. 返回到 Azure 门户 (``portal.azure.com``)，然后导航到 Azure Cosmos DB 帐户。

2. 在左侧导航菜单中，选择“**数据资源管理器**”。

3. 展开 **CosmicWorks** 数据库和 **Products** 容器，然后选择容器下的**项**。

4. 选择容器中的多个随机文档，并确保 `embedding` 字段填充了矢量数组。
