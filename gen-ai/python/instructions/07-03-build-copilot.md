---
lab:
  title: 07.3 - 使用 Python 和 Azure Cosmos DB for NoSQL 生成 Copilot
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
---

# 使用 Python 和 Azure Cosmos DB for NoSQL 生成 Copilot

利用 Python 的多功能编程功能和 Azure Cosmos DB 的可缩放 NoSQL 数据库以及矢量搜索功能，可以创建功能强大且高效的 AI Copilot，从而简化复杂的工作流。

在此实验室中，你将使用 Python 和 Azure Cosmos DB for NoSQL 生成 Copilot，创建后端 API，该 API 将提供与 Azure 服务（Azure OpenAI 和 Azure Cosmos DB）交互所需的终结点，以及前端 UI，以方便用户与 Copilot 交互。 Copilot 将作为帮助 Cosmic Works 用户管理和查找自行车相关产品的助手。 具体而言，Copilot 使用户能够从产品类别中应用和移除折扣，查找产品类别以帮助告知用户可用的产品类型，并使用矢量搜索对产品执行相似性搜索。

![高级 Copilot 体系结构关系图，显示 Python 中使用 Streamlit 开发的 UI、使用 Python 写入的后端 API 以及与 Azure Cosmos DB 和 Azure OpenAI 的交互。](media/07-copilot-high-level-architecture-diagram.png)

在 Python 中创建 Copilot 时，将应用功能拆分为专用 UI 和后端 API 提供多种优势。 首先，它增强模块化和可维护性，使你可以独立更新 UI 或后端，而不会中断另一个 UI 或后端。 Streamlit 提供直观的交互式界面，可简化用户交互，而 FastAPI 可确保高性能、异步请求处理和数据处理。 这种拆分还提升可扩展性，因为可以在多个服务器上部署不同的组件，从而优化资源使用状况。 此外，它还可实现更好的安全做法，因为后端 API 可以单独处理敏感数据和身份验证，从而减少在 UI 层公开漏洞的风险。 此方法可生成更可靠、更高效且用户友好的应用程序。

> #128721；本模块中前面的练习是本实验室的先决条件。 如果仍需完成上述任何练习，请先完成这些练习再继续，因为它们为本实验室提供必要的基础结构和起始代码。

## 构造后端 API

Copilot 的后端 API 可丰富处理复杂数据、提供实时见解以及与各种服务无缝连接的功能，使交互更加动态化，信息量更加丰富。 要为 Copilot 生成 API，请使用 FastAPI Python 库。 FastAPI 是一种现代高性能 Web 框架，旨在使你能够基于标准 Python 类型提示使用 Python 生成 API。 使用此方法将 Copilot 与后端分离，可以确保更高的灵活性、可维护性和可扩展性，从而使 Copilot 独立于后端更改而演进。

> #128721；后端 API 基于上一练习中在`python/07-build-copilot/api/app`文件夹的`main.py`文件中添加的代码生成。 如果尚未完成上一个练习，请完成此练习再继续。

1. 使用 Visual Studio Code，打开将**使用 Azure Cosmos DB 生成助手**学习模块的实验室代码存储库克隆到的文件夹。

1. 在 Visual Studio Code 的“**资源管理器**”窗格中，浏览到 **python/07-build-copilot/api/app** 文件夹，并打开在其中找到的`main.py`文件。

1. 在`main.py`文件顶部的现有`import`语句下添加以下代码行，以引入将用于使用 FastAPI 执行异步作的库：

   ```python
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
   ```

1. 要使要创建的`/chat`终结点能够接收请求正文中的数据，请通过项目*模型*模块中定义的`CompletionRequest`对象传入内容。 更新文件顶部的 `from models import Product`import 语句，以包含`models`模块中的`CompletionRequest`类。 import 语句现在应如下所示：

   ```python
   from models import Product, CompletionRequest
   ```

1. 你需要提供在 Azure OpenAI 服务中创建的聊天补全模型的部署名称。 在 Azure OpenAI 配置变量块底部创建变量，以提供：

   ```python
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
   ```

    如果补全部署名称不同，请相应更新分配给变量的值。

1. Azure Cosmos DB 和标识 SDK 提供使用这些服务的异步方法。 API 中的多个函数都将使用这些类，因此请创建每个类的全局实例，以便在各个方法间共享同一客户端。 在 Cosmos DB 配置变量块下方插入以下全局变量声明：

   ```python
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   ```

1. 从文件中删除以下代码行，因为提供的功能将移动到将在下一步定义的`lifespan`函数中：

   ```python
   # Enable Microsoft Entra ID RBAC authentication
   credential = DefaultAzureCredential()
   ```

1. 要创建`CosmosClient`和`DefaultAzureCredentail`类的单一实例，请利用 FastAPI 中的`lifespan`对象：此方法通过 API 应用的生命周期管理这些类。 插入以下代码以定义`lifespan`：

   ```python
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
   ```

   在 FastAPI 中，生存期事件是应用程序生命周期开始和结束时运行的特殊操作。 这些操作在应用程序开始处理请求前和停止处理后执行，因此非常适合初始化和清理在整个应用程序中使用并在请求之间共享的资源。 此方法可确保在处理任何请求之前完成必要的设置，并在关闭时正确管理资源。

1. 使用以下代码创建 FastAPI 类的实例。 应将其插入到`lifespan`函数下方：

   ```python
   app = FastAPI(lifespan=lifespan)
   ```

   通过调用`FastAPI()`，可以初始化 FastAPI 应用程序的新实例。 此实例称为`app`，将作为 Web 应用程序的主要入口点。 传入`lifespan`可将生存期事件处理程序附加到应用。

1. 接下来，为 API 存根终结点。 将`api_status`方法附加到 API 的根 URL，并充当状态消息，以表明 API 已启动并正常运行。 稍后将在本练习中生成`/chat`终结点。 在用于创建 Cosmos DB 客户端、数据库和容器的代码下方插入以下代码：

   ```python
   @app.get("/")
   async def api_status():
       """Display a status message for the API"""
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

1. 从命令行运行文件时，覆盖文件底部的 main guard 块以启动 `uvicorn` ASGI（异步服务器网关接口）Web 服务器：

   ```python
   if __name__ == "__main__":
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. 保存 `main.py` 文件。 现在，它应如下所示，包括你在之前的练习中添加的 `generate_embeddings` 和 `upsert_product` 方法：

   ```python
   from openai import AsyncAzureOpenAI
   from azure.identity.aio import DefaultAzureCredential, get_bearer_token_provider
   from azure.cosmos.aio import CosmosClient
   from models import Product, CompletionRequest
   from contextlib import asynccontextmanager
   from fastapi import FastAPI
   import json
    
   # Azure OpenAI configuration
   AZURE_OPENAI_ENDPOINT = "<AZURE_OPENAI_ENDPOINT>"
   AZURE_OPENAI_API_VERSION = "2024-10-21"
   EMBEDDING_DEPLOYMENT_NAME = "text-embedding-3-small"
   COMPLETION_DEPLOYMENT_NAME = 'gpt-4o'
    
   # Azure Cosmos DB configuration
   AZURE_COSMOSDB_ENDPOINT = "<AZURE_COSMOSDB_ENDPOINT>"
   DATABASE_NAME = "CosmicWorks"
   CONTAINER_NAME = "Products"
    
   # Create a global async Cosmos DB client
   cosmos_client = None
   # Create a global async Microsoft Entra ID RBAC credential
   credential = None
   
   @asynccontextmanager
   async def lifespan(app: FastAPI):
       global cosmos_client
       global credential
       # Create an async Microsoft Entra ID RBAC credential
       credential = DefaultAzureCredential()
       # Create an async Cosmos DB client using Microsoft Entra ID RBAC authentication
       cosmos_client = CosmosClient(url=AZURE_COSMOSDB_ENDPOINT, credential=credential)
       yield
       await cosmos_client.close()
       await credential.close()
    
   app = FastAPI(lifespan=lifespan)
    
   @app.get("/")
   async def api_status():
       return {"status": "ready"}
    
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """ Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
    
   async def generate_embeddings(text: str):
       # Create Azure OpenAI client
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
       import uvicorn
       uvicorn.run(app, host="0.0.0.0", port=8000)
   ```

1. 要快速测试 API，请在 Visual Studio Code 中打开新的集成终端窗口。

1. 确保使用`az login`命令登录到 Azure。 在终端提示符处运行以下命令：

   ```bash
   az login
   ```

1. 在浏览器中完成登录过程。

1. 在终端提示符处将目录更改为`python/07-build-copilot`。

1. 确保集成终端窗口在 Python 虚拟环境中运行，方法是使用下表中的命令将其激活，并选择 OS 和 shell 的相应命令。

    | 平台 | Shell | 激活虚拟环境的命令 |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

1. 在终端提示符处，将目录更改为 `api/app`，然后执行以下命令来运行 FastAPI Web 应用：

   ```bash
   uvicorn main:app
   ```

1. 如果未自动打开，请启动新的 Web 浏览器窗口或选项卡并转到 <http://127.0.0.1:8000>。

    浏览器窗口中的 `{"status":"ready"}` 消息指示 API 正在运行。

1. 通过将 `/docs` 追加到 URL 末尾，导航到 API 的 Swagger UI：<http://127.0.0.1:8000/docs>。

    > &#128221; Swagger UI 是基于 Web 的交互式界面，用于探索和测试从 OpenAPI 规范生成的 API 终结点。 它使开发人员和用户能够可视化和调试实时 API 调用并与之交互，增强了可用性，并改进了归档。

1. 在关联的集成终端窗口中按 **Ctrl+C**，返回到 Visual Studio Code 并停止 API 应用。

## 合并 Azure Cosmos DB 中的产品数据

利用 Azure Cosmos DB 中的数据，助手可以简化复杂的工作流，并帮助用户高效完成任务。 助手可以实时更新记录和检索查找值，确保信息准确及时。 利用此功能，助手可提供高级交互，同时增强了用户快速、准确地导航和完成任务的能力。

通过函数，产品管理助手可以将折扣应用于类别中的产品。 这些函数将是助手从 Azure Cosmos DB 检索 Cosmic Works 产品数据并与此类数据交互的机制。

1. 助手将使用一个名为 `apply_discount` 的异步函数来添加和移除指定类别内产品的折扣和销售价格。 在 `main.py` 文件底部附近的 `upsert_product` 函数下方插入以下函数代码：

   ```python
   async def apply_discount(discount: float, product_category: str) -> str:
       """Apply a discount to products in the specified category."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT * FROM Products p WHERE CONTAINS(LOWER(p.category_name), LOWER(@product_category))
           """,
           parameters = [
               {"name": "@product_category", "value": product_category}
           ]
       )
    
       # Apply the discount to the products
       async for item in query_results:
           item['discount'] = discount
           item['sale_price'] = item['price'] * (1 - discount) if discount > 0 else item['price']
           await container.upsert_item(item)
    
       return f"A {discount}% discount was successfully applied to {product_category}." if discount > 0 else f"Discounts on {product_category} removed successfully."
   ```

    此函数在 Azure Cosmos DB 中执行查找，以拉取类别中的所有产品，并将请求的折扣应用于这些产品。 它还使用指定的折扣计算物料的销售价格，并将其插入数据库中。

2. 接下来，你将添加名为 `get_category_names` 的第二个函数，助手将调用该函数来帮助它了解在应用或移除产品折扣时可用的产品类别。 在文件中的 `apply_discount` 函数下方添加以下方法：

   ```python
   async def get_category_names() -> list:
       """Retrieve the names of all product categories."""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
       # Get distinct product categories
       query_results = container.query_items(
           query = "SELECT DISTINCT VALUE p.category_name FROM Products p"
       )
       categories = []
       async for category in query_results:
           categories.append(category)
       return list(categories)
   ```

    `get_category_names` 函数用于查询 `Products` 容器，以从数据库中检索非重复类别名称的列表。

3. 保存 `main.py` 文件。

## 实现聊天终结点

后端 API 上的 `/chat` 终结点充当前端 UI 与 Azure OpenAI 模型和内部 Cosmic Works 产品数据交互的接口。 此终结点充当通信网桥，支持将 UI 输入发送到 Azure OpenAI 服务，然后使用复杂的语言模型处理这些输入。 之后，结果将返回到前端，并启用实时智能对话。 利用此设置，开发人员可确保提供响应迅速的无缝用户体验，而后端可应对自然语言处理以及生成适当响应的复杂任务。 此方法还通过将前端与底层 AI 基础结构分离，支持可伸缩性和可维护性。

1. 找到之前在 `main.py` 文件中添加的 `/chat` 终结点存根。

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       raise NotImplementedError("The chat endpoint is not implemented yet.")
   ```

    该函数以参数的形式接受 `CompletionRequest`。 利用输入参数的类，可以将多个属性传递到请求正文中的 API 终结点。 `CompletionRequest` 类在*模型*模块中定义，包括用户消息、聊天历史记录和最大历史记录属性。 借助聊天历史记录，助手可以参考与用户之前的对话细节，始终对整个讨论的上下文保持了解。 利用 `max_history` 属性，你可以定义应传递到 LLM 上下文中的历史记录消息数。 这样，你便可以控制提示的令牌使用情况，并避免对请求的 TPM 限制。

2. 首先，开始实现终结点的过程时，从函数中删除 `raise NotImplementedError("The chat endpoint is not implemented yet.")` 行。

3. 在聊天终结点方法中，要执行的第一件事是提供系统提示。 此提示定义了助手的“角色”，描述了助手应如何与用户交互、回答问题，并利用可用函数来执行操作。

   ```python
   # Define the system prompt that contains the assistant's persona.
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   """
   ```

4. 接下来，创建要发送到 LLM 的消息数组，添加系统提示、聊天历史记录中的任何消息以及传入的用户消息。 此代码应直接位于函数中的系统提示声明下方：

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{"role": "system", "content": system_prompt }]
    
   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)
    
   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

    `messages` 属性会封装正在进行的对话历史记录。 其中包括整个用户输入序列和 AI 的响应，这有助于模型维护上下文。 通过参考此历史记录，AI 可以生成连贯一致且与上下文相关的回复，确保交互保持流畅和活跃。 此属性至关重要，可以让 AI 了解对话流的进度，洞悉对话中的细微差别。

5. 要允许助手使用上面定义的函数与 Azure Cosmos DB 中的数据进行交互，必须定义“工具”集合。 LLM 将作为其执行的一部分调用这些工具。 Azure OpenAI 使用函数定义来实现 AI 与各种工具或 API 之间的结构化交互。 定义函数时，描述了函数可以执行的操作、必要的参数和任何必需的输入。 要创建 `tools` 数组，请提供以下代码，其中包含之前定义的 `apply_discount` 和 `get_category_names` 方法的函数定义：

   ```python
   # Define function calling tools
   tools = [
       {
           "type": "function",
           "function": {
               "name": "apply_discount",
               "description": "Apply a discount to products in the specified category",
               "parameters": {
                   "type": "object",
                   "properties": {
                       "discount": {"type": "number", "description": "The percent discount to apply."},
                       "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                   },
                   "required": ["discount", "product_category"]
               }
           }
       },
       {
           "type": "function",
           "function": {
               "name": "get_category_names",
               "description": "Retrieves the names of all product categories"
           }
       }
   ]
   ```

    利用函数定义，Azure OpenAI 可确保 AI 与外部系统之间的交互组织有序、安全且高效。 这种结构化方法使 AI 能够无缝、可靠地执行复杂任务，从而增强其整体功能和用户体验。

6. 创建用于向聊天完成模型发出请求的异步 Azure OpenAI 客户端：

   ```python
   # Create Azure OpenAI client
   aoai_client = AsyncAzureOpenAI(
       api_version = AZURE_OPENAI_API_VERSION,
       azure_endpoint = AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
   )
   ```

7. 聊天终结点将对 Azure OpenAI 进行两次调用，以利用函数调用。 第一次将为 Azure OpenAI 客户端提供对工具的访问权限：

   ```python
   # First API call, providing the model to the defined functions
   response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages,
       tools = tools,
       tool_choice = "auto"
   )
    
   # Process the model's response and add it to the conversation history
   response_message = response.choices[0].message
   messages.append(response_message)
   ```

8. 第一次调用的响应包含 LLM 中有关已确定响应请求必需哪些工具或函数的信息。 必须包含代码来处理函数调用输出，并将其插入对话历史记录，以便 LLM 可以利用它们来针对这些输出中包含的数据制定响应：

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

    Azure OpenAI 中的函数调用支持将外部 API 或工具无缝集成到模型的输出中。 当模型检测到相关请求时，它会使用必需的参数构造 JSON 对象，以便你后续执行。 结果将返回到模型，从而让模型交付使用外部数据扩充的全面的最终响应。

9. 要利用 Azure Cosmos DB 中的扩充数据完成请求，你需要向 Azure OpenAI 发送第二个请求以生成补全内容：

   ```python
   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )
   ```

10. 最后，将完成响应返回到 UI：

   ```python
   return final_response.choices[0].message.content
   ```

11. 保存 `main.py` 文件。 `/chat`终结点`generate_chat_completion`的方法应如下所示：

   ```python
   @app.post('/chat')
   async def generate_chat_completion(request: CompletionRequest):
       """Generate a chat completion using the Azure OpenAI API."""
       # Define the system prompt that contains the assistant's persona.
       system_prompt = """
       You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
       You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
       If asked to apply a discount:
           - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
           - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
       If asked to remove discounts from a category:
           - Remove any discounts applied to products in the specified category by setting the discount value to 0.
       """
       # Provide the copilot with a persona using the system prompt.
       messages = [{ "role": "system", "content": system_prompt }]
    
       # Add the chat history to the messages list
       for message in request.chat_history[-request.max_history:]:
           messages.append(message)
    
       # Add the current user message to the messages list
       messages.append({"role": "user", "content": request.message})
    
       # Define function calling tools
       tools = [
           {
               "type": "function",
               "function": {
                   "name": "apply_discount",
                   "description": "Apply a discount to products in the specified category",
                   "parameters": {
                       "type": "object",
                       "properties": {
                           "discount": {"type": "number", "description": "The percent discount to apply."},
                           "product_category": {"type": "string", "description": "The category of products to which the discount should be applied."}
                       },
                       "required": ["discount", "product_category"]
                   }
               }
           },
           {
               "type": "function",
               "function": {
                   "name": "get_category_names",
                   "description": "Retrieves the names of all product categories"
               }
           }
       ]
       # Create Azure OpenAI client
       aoai_client = AsyncAzureOpenAI(
           api_version = AZURE_OPENAI_API_VERSION,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
    
       # First API call, providing the model to the defined functions
       response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages,
           tools = tools,
           tool_choice = "auto"
       )
    
       # Process the model's response
       response_message = response.choices[0].message
       messages.append(response_message)
    
       # Handle function call outputs
       if response_message.tool_calls:
           for call in response_message.tool_calls:
               if call.function.name == "apply_discount":
                   func_response = await apply_discount(**json.loads(call.function.arguments))
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": func_response
                       }
                   )
               elif call.function.name == "get_category_names":
                   func_response = await get_category_names()
                   messages.append(
                       {
                           "role": "tool",
                           "tool_call_id": call.id,
                           "name": call.function.name,
                           "content": json.dumps(func_response)
                       }
                   )
       else:
           print("No function calls were made by the model.")
    
       # Second API call, asking the model to generate a response
       final_response = await aoai_client.chat.completions.create(
           model = COMPLETION_DEPLOYMENT_NAME,
           messages = messages
       )
    
       return final_response.choices[0].message.content
   ```

## 生成简单的聊天 UI

Streamlit UI 提供一个界面供用户与助手交互。

1. 将使用 `python/07-build-copilot/ui` 文件夹中的 `index.py` 文件定义 UI。

2. 打开 `index.py` 文件，并将以下导入语句添加到文件顶部以开始：

   ```python
   import streamlit as st
   import requests
   ```

3. 通过在 `import` 语句下方添加以下行来配置 `index.py` 文件中定义的 Streamlit 页面：

   ```python
   st.set_page_config(page_title="Cosmic Works Copilot", layout="wide")
   ```

4. UI 将使用 `requests` 库与后端 API 交互，以调用在 API 上定义的 `/chat` 终结点。 可以将 API 调用封装在一个方法中，该方法需要当前用户消息和聊天历史记录中的消息列表。

   ```python
   async def send_message_to_copilot(message: str, chat_history: list = []) -> str:
       """Send a message to the Copilot chat endpoint."""
       try:
           api_endpoint = "http://localhost:8000"
           request = {"message": message, "chat_history": chat_history}
           response = requests.post(f"{api_endpoint}/chat", json=request, timeout=60)
           return response.json()
       except Exception as e:
           st.error(f"An error occurred: {e}")
           return""
   ```

5. 定义 `main` 函数，该函数是对应用程序进行调用的入口点。

   ```python
   async def main():
       """Main function for the Cosmic Works Product Management Copilot UI."""
    
       st.write(
           """
           # Cosmic Works Product Management Copilot
        
           Welcome to Cosmic Works Product Management Copilot, a tool for managing and finding bicycle-related products in the Cosmic Works system.
        
           **Ask the copilot to apply or remove a discount on a category of products or to find products.**
           """
       )
    
       # Add a messages collection to the session state to maintain the chat history.
       if "messages" not in st.session_state:
           st.session_state.messages = []
    
       # Display message from the history on app rerun.
       for message in st.session_state.messages:
           with st.chat_message(message["role"]):
               st.markdown(message["content"])
    
       # React to user input
       if prompt := st.chat_input("What can I help you with today?"):
           with st. spinner("Awaiting the copilot's response to your message..."):
               # Display user message in chat message container
               with st.chat_message("user"):
                   st.markdown(prompt)
                
               # Send the user message to the copilot API
               response = await send_message_to_copilot(prompt, st.session_state.messages)
    
               # Display assistant response in chat message container
               with st.chat_message("assistant"):
                   st.markdown(response)
                
               # Add the current user message and assistant response messages to the chat history
               st.session_state.messages.append({"role": "user", "content": prompt})
               st.session_state.messages.append({"role": "assistant", "content": response})
   ```

6. 最后，在文件末尾添加 **main guard** 块：

   ```python
   if __name__ == "__main__":
       import asyncio
       asyncio.run(main())
   ```

7. 保存 `index.py` 文件。

## 通过 UI 测试助手

1. 返回到在 Visual Studio Code 中为 API 项目打开的集成终端窗口，并输入以下内容以启动 API 应用：

   ```bash
   uvicorn main:app
   ```

2. 打开新的集成终端窗口，将目录更改为 `python/07-build-copilot` 以激活 Python 环境，然后将目录更改为 `ui` 文件夹，并运行以下命令来启动 UI 应用：

   ```bash
   python -m streamlit run index.py
   ```

3. 如果 UI 未在浏览器窗口中自动打开，请启动新的浏览器选项卡或窗口并导航到 <http://localhost:8501> 以打开 UI。

4. 在 UI 的聊天提示下，输入“应用折扣”并发送消息。

    由于需要为助手提供更多详细信息才能采取行动，因此响应应为请求获取更多信息，例如提供要应用的折扣百分比以及应向其应用折扣的产品类别。

5. 要了解哪些类别可用，请让助手为你提供一个产品类别列表。

    助手将使用 `get_category_names` 函数进行函数调用，并使用这些类别丰富对话消息，以便做出相应的响应。

6. 还可以请求一组更具体的类别，例如，“向我提供与服装相关类别的列表”。

7. 接下来，要求 Copilot 对所有服装产品应用 15% 的折扣。

8. 可以通过以下方法验证是否已应用定价折扣：在 Azure 门户中打开 Azure Cosmos DB 帐户，选择“**数据资源管理器**”，然后针对`Products`容器运行查询以查看“服装”类别中的所有产品，例如：

   ```sql
   SELECT c.category_name, c.name, c.description, c.price, c.discount, c.sale_price FROM c
   WHERE CONTAINS(LOWER(c.category_name), "clothing")
   ```

    请注意，查询结果中的每个项的`discount`值为`0.15`，且`sale_price`应比原始`price`小 15%。

9. 返回到 Visual Studio Code，并在运行该应用的终端窗口中按 **CTRL+C** 停止 API 应用。 可以让 UI 保持运行状态。

## 集成矢量搜索

到目前为止，你已经让助手能够执行对产品应用折扣的操作，但它仍然不了解数据库中存储的产品。 在此任务中，你将添加矢量搜索功能，使你能够请求具有特定品质的产品，并在数据库中查找类似产品。

1. 返回到`api/app`文件夹中的`main.py`文件，并提供对 Azure Cosmos DB 帐户中的`Products`容器执行矢量搜索的方法。 可以在文件底部附近的现有函数下面插入此方法。

   ```python
   async def vector_search(query_embedding: list, num_results: int = 3, similarity_score: float = 0.25):
       """Search for similar product vectors in Azure Cosmos DB"""
       # Load the CosmicWorks database
       database = cosmos_client.get_database_client(DATABASE_NAME)
       # Retrieve the product container
       container = database.get_container_client(CONTAINER_NAME)
    
       query_results = container.query_items(
           query = """
           SELECT TOP @num_results p.name, p.description, p.sku, p.price, p.discount, p.sale_price, VectorDistance(p.embedding, @query_embedding) AS similarity_score
           FROM Products p
           WHERE VectorDistance(p.embedding, @query_embedding) > @similarity_score
           ORDER BY VectorDistance(p.embedding, @query_embedding)
           """,
           parameters = [
               {"name": "@query_embedding", "value": query_embedding},
               {"name": "@num_results", "value": num_results},
               {"name": "@similarity_score", "value": similarity_score}
           ]
       )
       similar_products = []
       async for result in query_results:
           similar_products.append(result)
       formatted_results = [{'similarity_score': product.pop('similarity_score'), 'product': product} for product in similar_products]
       return formatted_results
   ```

2. 接下来，创建名为`get_similar_products`的方法，该方法将用作 LLM 用以对数据库执行矢量搜索的函数：

   ```python
   async def get_similar_products(message: str, num_results: int):
       """Retrieve similar products based on a user message."""
       # Vectorize the message
       embedding = await generate_embeddings(message)
       # Perform vector search against products in Cosmos DB
       similar_products = await vector_search(embedding, num_results=num_results)
       return similar_products
   ```

    该`get_similar_products`函数对上面定义的`vector_search`函数以及在上一练习中创建的`generate_embeddings`函数进行异步调用。 在传入的用户消息上生成嵌入，以便使用 Cosmos DB 中内置的`VectorDistance`函数将其与数据库中存储的矢量进行比较。

3. 要允许 LLM 使用新函数，必须更新之前创建的`tools`数组，为`get_similar_products`方法添加函数定义：

   ```json
   {
       "type": "function",
       "function": {
           "name": "get_similar_products",
           "description": "Retrieve similar products based on a user message.",
           "parameters": {
               "type": "object",
               "properties": {
                   "message": {"type": "string", "description": "The user's message looking for similar products"},
                   "num_results": {"type": "integer", "description": "The number of similar products to return"}
               },
               "required": ["message"]
           }
       }
   }
   ```

4. 还必须添加代码来处理新函数的输出。 将以下`elif`条件添加到处理函数调用输出的代码块：

   ```python
   elif call.function.name == "get_similar_products":
       func_response = await get_similar_products(**json.loads(call.function.arguments))
       messages.append(
           {
               "role": "tool",
               "tool_call_id": call.id,
               "name": call.function.name,
               "content": json.dumps(func_response)
           }
       )
   ```

    已完成的块现在如下所示：

   ```python
   # Handle function call outputs
   if response_message.tool_calls:
       for call in response_message.tool_calls:
           if call.function.name == "apply_discount":
               func_response = await apply_discount(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": func_response
                   }
               )
           elif call.function.name == "get_category_names":
               func_response = await get_category_names()
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
           elif call.function.name == "get_similar_products":
               func_response = await get_similar_products(**json.loads(call.function.arguments))
               messages.append(
                   {
                       "role": "tool",
                       "tool_call_id": call.id,
                       "name": call.function.name,
                       "content": json.dumps(func_response)
                   }
               )
   else:
       print("No function calls were made by the model.")
   ```

5. 最后，需要更新系统提示定义，以提供有关如何执行矢量搜索的说明。 在`system_prompt`底部插入以下内容：

   ```plaintext
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   ```

    更新的系统提示将类似于：

   ```python
   system_prompt = """
   You are an intelligent copilot for Cosmic Works designed to help users manage and find bicycle-related products.
   You are helpful, friendly, and knowledgeable, but can only answer questions about Cosmic Works products.
   If asked to apply a discount:
       - Apply the specified discount to all products in the specified category. If the user did not provide you with a discount percentage and a product category, prompt them for the details you need to apply a discount.
       - Discount amounts should be specified as a decimal value (e.g., 0.1 for 10% off).
   If asked to remove discounts from a category:
       - Remove any discounts applied to products in the specified category by setting the discount value to 0.
   When asked to provide a list of products, you should:
       - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
   """
   ```

6. 保存 `main.py` 文件。

## 测试矢量搜索功能

1. 在 Visual Studio Code 打开的集成终端窗口中针对该应用运行以下命令以重启 API 应用：

   ```bash
   uvicorn main:app
   ```

2. UI 仍应在运行，但如果停止 UI，请返回到其集成终端窗口并运行：

   ```bash
   python -m streamlit run index.py
   ```

3. 返回到运行 UI 的浏览器窗口，并在聊天提示符处输入以下内容：

   ```bash
   Tell me about the mountain bikes in stock
   ```

    此问题将返回与搜索匹配的几个产品。

4. 尝试其他一些搜索，例如“向我展示耐用踏板”，“提供 5 款时尚骑行服清单”，以及“提供所有适合热天骑行的手套的详细信息”。

    对于最后两个查询，请注意，此产品包含之前应用的 15% 折扣和销售价格。
