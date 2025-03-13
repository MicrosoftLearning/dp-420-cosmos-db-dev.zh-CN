---
title: 07.4 - 使用 LangChain 和 Azure Cosmos DB for NoSQL 矢量搜索实现 RAG
lab:
  title: 07.4 - 使用 LangChain 和 Azure Cosmos DB for NoSQL 矢量搜索实现 RAG
  module: Build copilots with Python and Azure Cosmos DB for NoSQL
layout: default
nav_order: 13
parent: Python SDK labs
---

# 使用 LangChain 和 Azure Cosmos DB for NoSQL 矢量搜索实现 RAG

LangChain 的业务流程功能通过直接使用 Azure OpenAI 客户端实现 Copilot 的 LLM 集成带来许多好处。 LangChain 允许与各种数据源（包括 Azure Cosmos DB）更紧密集成，从而实现高效的矢量搜索，进而增强检索过程。 LangChain 提供强大的工具来管理和优化工作流，简化使用模块化和可重用组件生成复杂应用程序的过程。 这种灵活性不仅简化开发过程，还可确保可扩展性和可维护性。

在此实验室中，从使用 Azure OpenAI 客户端改为利用 LangChain 功能强大的业务流程功能来转换 API 的`/chat`终结点，从而增强 Copilot 的功能。 此转变通过将矢量搜索功能与 Azure Cosmos DB for NoSQL 集成，实现更高效的数据检索并提高性能。 无论是希望优化应用的信息检索流程还是仅探索 RAG 的潜能，本模块都将指导你完成无缝转换，演示 LangChain 如何简化和提升应用的功能。 让我们踏上这段旅程，利用 LangChain 和 Azure Cosmos DB 解锁新的效率和见解！

> #128721；本模块中前面的练习是本实验室的先决条件。 如果仍需完成上述任何练习，请先完成这些练习再继续，因为它们为本实验室提供必要的基础结构和起始代码。

## 安装 LangChain 库

1. 使用 Visual Studio Code，打开将**使用 Azure Cosmos DB 生成助手**学习模块的实验室代码存储库克隆到的文件夹。

2. 在 Visual Studio Code 的“**资源管理器**”窗格中，浏览到 **python/07-build-copilot** 文件夹并打开在其中找到的`requirements.txt`文件。

3. 更新`requirements.txt`文件以包含所需的 LangChain 库：

   ```ini
   langchain==0.3.9
   langchain-openai==0.2.11
   ```

4. 在 Visual Studio Code 中启动新的集成终端窗口，并将目录更改为`python/07-build-copilot`。

5. 确保集成终端窗口在 Python 虚拟环境中运行，方法是使用下表中 OS 和 shell 的相应命令将其激活：

    | 平台 | Shell | 激活虚拟环境的命令 |
    | -------- | ----- | --------------------------------------- |
    | POSIX | bash/zsh | `source .venv/bin/activate` |
    | | fish | `source .venv/bin/activate.fish` |
    | | csh/tcsh | `source .venv/bin/activate.csh` |
    | | pwsh | `.venv/bin/Activate.ps1` |
    | Windows | cmd.exe | `.venv\Scripts\activate.bat` |
    | | PowerShell | `.venv\Scripts\Activate.ps1` |

6. 在集成终端提示符处执行以下命令并使用 LangChain 库更新虚拟环境：

   ```bash
   pip install -r requirements.txt
   ```

7. 关闭集成终端。

## 更新后端 API

在上一个实验室中，你使用 Azure OpenAI 客户端和 Azure Cosmos DB 中的数据执行了 RAG 模式。 现在，你将更新后端 API，以将 LangChain 代理与工具配合使用来执行相同的操作。

从代码的角度来看，使用 LangChain 与 Azure OpenAI 服务中部署的语言模型交互要稍微简单一些...

1. 移除`main.py`文件顶部的`from openai import AzureOpenAI` import 语句。 不再需要该客户端库，因为与 Azure OpenAI 的所有交互都将通过 LangChain 提供的类进行。

2. 删除`main.py`文件顶部的以下 import 语句，因为不再需要这些语句：

   ```python
   from openai import AsyncAzureOpenAI
   import json
   ```

### 更新嵌入终结点

1. 在`main.py`文件顶部添加以下 import 语句，以从`langchain_openai`库中导入`AzureOpenAIEmbeddings`类：

   ```python
   from langchain_openai import AzureOpenAIEmbeddings
   ```

2. 在文件中找到`generate_embeddings`方法并使用以下方法将其覆盖，该方法使用`AzureOpenAIEmbeddings`类处理与 Azure OpenAI 的交互：

   ```python
   async def generate_embeddings(text: str):
       """Generates embeddings for the provided text."""
       # Use LangChain's Azure OpenAI Embeddings class
       azure_openai_embeddings = AzureOpenAIEmbeddings(
           azure_deployment = EMBEDDING_DEPLOYMENT_NAME,
           azure_endpoint = AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider = get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default")
       )
       return await azure_openai_embeddings.aembed_query(text)
   ```

    该`AzureOpenAIEmbeddings`类提供一个接口，用于与 Azure OpenAI 嵌入 API 交互，并返回仅包含生成的矢量的简化响应对象。

### 更新聊天终结点

1. 更新`lanchain_openai` import 语句以追加`AzureChatOpenAI`类：

   ```python
   from langchain_openai import AzureOpenAIEmbeddings, AzureChatOpenAI
   ```

1. 导入生成修改后的`/chat`终结点时将使用的以下其他 LangChain 对象：

   ```python
   from langchain.agents import AgentExecutor, create_openai_functions_agent
   from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
   from langchain_core.tools import StructuredTool
   ```

1. 聊天历史记录将使用 LangChain 代理以不同的方式注入到 copilot 对话中，因此请删除紧随`system_prompt`定义之后的代码行。 应删除的行包括：

   ```python
   # Provide the copilot with a persona using the system prompt.
   messages = [{ "role": "system", "content": system_prompt }]

   # Add the chat history to the messages list
   for message in request.chat_history[-request.max_history:]:
       messages.append(message)

   # Add the current user message to the messages list
   messages.append({"role": "user", "content": request.message})
   ```

1. 使用 LangChain 的`ChatPromptTemplate`类定义`prompt`对象来代替刚刚删除的代码：

   ```python
   prompt = ChatPromptTemplate.from_messages(
       [
           ("system", system_prompt),
           MessagesPlaceholder("chat_history", optional=True),
           ("user", "{input}"),
           MessagesPlaceholder("agent_scratchpad")
       ]
   )
   ```

    正使用多个组件按特定顺序创建`ChatPromptTemplate`。 以下是将这些组件组合在一起的方法：

    - **系统消息**：使用`system_prompt`为 Copilot 赋予角色，提供有关助手的行为方式和如何与用户交互的说明。
    - **聊天历史记录**：允许将包含对话中过往消息列表的`chat_history`合并到 LLM 正在处理的上下文中。
    - **用户输入**：当前用户消息。
    - **代理暂存板**：允许代理执行中间笔记或步骤。

    生成的提示为对话式 AI 代理提供结构化输入，帮助它基于给定上下文生成响应。

1. 接下来，将`tools`数组定义替换为以下内容，后者使用 LangChain 的`StructuredTool`类将函数定义提取为正确的格式：

   ```python
   tools = [
       StructuredTool.from_function(coroutine=apply_discount),
       StructuredTool.from_function(coroutine=get_category_names),
       StructuredTool.from_function(coroutine=get_similar_products)
   ]
   ```

    LangChain 中的`StructuredTool.from_function`方法使用输入参数和函数的 DocString 说明并根据给定函数创建工具。 要将其与异步方法配合使用，请指定将函数名称传递给`coroutine`输入参数。

    在 Python 中，DocString（文档字符串的缩写）是一种用于记录函数、方法、类或模块的特殊字符串。 它提供将文档与 Python 代码相关联的便捷方式，通常括在三引号（""" 或 '''）中。 Docstring 紧跟在它们记录的函数（或方法、类或模块）定义之后。

    使用此函数可自动创建必须使用 Azure OpenAI 客户端手动创建的 JSON 函数定义，从而简化函数调用过程。

1. 删除上面完成的`tools`数组定义与函数末尾的`return`语句之间的所有代码。 使用 Azure OpenAI 客户端时，必须调用语言模型两次。 第一次调用使其能够确定需要调用哪些函数（如果有）以增强提示，第二次调用请求进行 RAG 补全。 在此期间，必须使用代码检查第一次调用的响应，以确定是否需要调用函数，然后写入代码以“处理”对这些函数的调用。 然后，必须将这些函数调用的输出插入到发送到 LLM 的消息中，使其可以在生成补全响应时有丰富的提示进行推理。 LangChain 极大简化了使用 RAG 模式调用 LLM 的过程，如下所示。 应移除的代码包括：

   ```python
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

   # Second API call, asking the model to generate a response
   final_response = await aoai_client.chat.completions.create(
       model = COMPLETION_DEPLOYMENT_NAME,
       messages = messages
   )

   return final_response.choices[0].message.content
   ```

1. 从`tools`数组定义下方开始，使用 LangChain 中的`AzureChatOpenAI`类创建对 Azure OpenAI API 的引用：

   ```python
   # Connect to Azure OpenAI API
   azure_openai = AzureChatOpenAI(
       azure_deployment=COMPLETION_DEPLOYMENT_NAME,
       azure_endpoint=AZURE_OPENAI_ENDPOINT,
       azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
       api_version=AZURE_OPENAI_API_VERSION
   )
   ```

1. 要允许 LangChain 代理与已定义的函数交互，请使用`create_openai_functions_agent`方法创建代理，你将为其提供`AzureChatOpenAI`对象、`tools`数组和`ChatPromptTemplate`对象：

   ```python
   agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
   ```

    LangChain 中的`create_openai_functions_agent`函数可创建代理，该代理可以调用外部函数以使用指定的语言模型和工具执行任务。 这样就可以将各种服务和功能集成到代理的工作流中，从而提供灵活性和增强功能。

1. 在 LangChain 中，`AgentExecutor`类用于管理代理的执行流，例如使用`create_openai_functions_agent`方法创建的代理。 它处理输入的处理、工具或模型的调用以及输出的处理。 使用以下代码为代理创建代理执行程序：

   ```python
   agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
   ```

    `AgentExecutor`确保按正确的顺序执行生成响应所需的所有步骤。 它抽象化代理执行的复杂性，提供额外的功能和结构层，并简化生成、管理和缩放复杂代理的过程。

1. 你将使用代理执行程序的`invoke`方法将传入的用户消息发送到 LLM。 还将包含聊天历史记录。 在`agent_executor`定义下方插入以下代码：

   ```python
   completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
   ```

   `input`和`chat_history`标记是在使用`ChatPromptTemplate`创建的提示对象中定义的。 使用`invoke`方法时，它们将注入到提示中，从而允许 LLM 在创建响应时使用该信息。

1. 最后，更新 return 语句以使用代理补全对象的`output`：

   ```python
   return completion["output"]
   ```

1. 保存 `main.py` 文件。 更新后的`/chat`终结点函数应如下所示：

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
       When asked to provide a list of products, you should:
           - Provide at least 3 candidate products unless the user asks for more or less, then use that number. Always include each product's name, description, price, and SKU. If the product has a discount, include it as a percentage and the associated sale price.
       """
       prompt = ChatPromptTemplate.from_messages(
           [
               ("system", system_prompt),
               MessagesPlaceholder("chat_history", optional=True),
               ("user", "{input}"),
               MessagesPlaceholder("agent_scratchpad")
           ]
       )
    
       # Define function calling tools
       tools = [
           StructuredTool.from_function(apply_discount),
           StructuredTool.from_function(get_category_names),
           StructuredTool.from_function(get_similar_products)
       ]
    
       # Connect to Azure OpenAI API
       azure_openai = AzureChatOpenAI(
           azure_deployment=COMPLETION_DEPLOYMENT_NAME,
           azure_endpoint=AZURE_OPENAI_ENDPOINT,
           azure_ad_token_provider=get_bearer_token_provider(credential, "https://cognitiveservices.azure.com/.default"),
           api_version=AZURE_OPENAI_API_VERSION
       )
    
       agent = create_openai_functions_agent(llm=azure_openai, tools=tools, prompt=prompt)
       agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True, return_intermediate_steps=True)
        
       completion = await agent_executor.ainvoke({"input": request.message, "chat_history": request.chat_history[-request.max_history:]})
            
       return completion["output"]
   ```

## 启动 API 和 UI 应用

1. 要启动 API，请在 Visual Studio Code 中打开新的集成终端窗口。

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

1. 打开新的集成终端窗口，将目录更改为 `python/07-build-copilot` 以激活 Python 环境，然后将目录更改为 `ui` 文件夹，并运行以下命令来启动 UI 应用：

   ```bash
   python -m streamlit run index.py
   ```

1. 如果 UI 未在浏览器窗口中自动打开，请启动新的浏览器选项卡或窗口并导航到 <http://localhost:8501> 以打开 UI。

## 测试 Copilot

1. 在将消息发送到 UI 之前，请返回到 Visual Studio Code 并选择与 API 应用关联的集成终端窗口。 在此窗口中，你将看到 LangChain 代理执行程序生成的“详细”输出，该执行程序将提供对 LangChain 如何处理你所发送请求的见解。 在你发送以下请求时，请注意此窗口中的输出，并在每次调用后重新检查。

1. 在 UI 中的聊天提示下，输入“应用折扣”并发送消息。

    你应该会收到回复，询问要应用的折扣百分比以及产品类别。

1. 回复，“手套。”

    你将收到一个响应，询问要对“手套”类别应用的折扣百分比。

1. 发送消息“25%。”

    你应该会收到一条响应，表明“25% 的折扣已成功应用于“手套”类别中的所有产品。”

1. 请助手“向我展示所有的手套。”

    在回复中，你应该会看到数据库中所有手套的列表，其中将包括 25% 的折扣价格。

1. 最后，询问“什么手套最适合寒冷天气骑行？” 以执行矢量搜索。 这涉及到对 `get_similar_items` 方法的函数调用，然后会调用你所更新的 `generate_embeddings` 方法以使用 LangChain 实现，同时调用 `vector_search` 函数。

1. 关闭集成终端。

1. 关闭 Visual Studio Code。
