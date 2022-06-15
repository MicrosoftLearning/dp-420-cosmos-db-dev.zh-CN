---
lab:
  title: 使用 Azure 门户创建存储过程
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB SQL API
ms.openlocfilehash: 0f390ab5e3bd796c3bfb17a060e2db4424e222e2
ms.sourcegitcommit: 58caf52fefd9f9cbeeef3629e98245544a299b44
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/02/2022
ms.locfileid: "146027670"
---
# <a name="create-a-stored-procedure-with-the-azure-portal"></a>使用 Azure 门户创建存储过程

存储过程是你可以在 Azure Cosmos DB 中执行业务逻辑服务器端的方法之一。 使用存储过程，可以在单个事务范围内使用容器对多个文档执行基本的 CRUD（创建、读取、更新、删除）操作。

在本实验室中，你将创作一个在容器内创建文档的存储过程。 然后，使用 SQL 查询来验证存储过程的结果。

## <a name="author-a-stored-procedure"></a>创作存储过程

存储过程是用语言集成的 JavaScript 编写的，支持在数据库引擎内执行基本 CRUD 操作。 在数据库引擎内运行的 JavaScript 是通过 Azure Cosmos DB 的服务器端 JavaScript SDK 和一系列辅助方法实现的。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“+ 创建资源”，搜索“Cosmos DB”，然后使用以下设置创建新的“Azure Cosmos DB SQL API”帐户资源，并将所有其余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 选择现有资源组，或创建新资源组 |
    | **帐户名** | 输入一个全局唯一的名称 |
    | **位置** | 选择任何可用区域 |
    | **容量模式** | *预配的吞吐量* |
    | **应用免费分级折扣** | *不应用* |

    > &#128221; 你的实验室环境可能存在阻止你创建新资源组的限制。 如果是这种情况，请使用预先创建的现有资源组。

1. 等待部署任务完成，然后继续执行此任务。

1. 转到新创建的 Azure Cosmos DB 帐户资源，并导航到“数据资源管理器”窗格。

1. 在“数据资源管理器”中，选择“新建容器”，然后创建一个具有以下设置的新容器，并将所有其余设置保留为其默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **数据库 ID** | 新建 &vert; cosmicworks |
    | **在容器之间共享吞吐量** | 选择此选项 |
    | **数据库吞吐量** | 手动 &vert; 400 |
    | **容器 ID** | products |
    | **索引** | *自动* |
    | **分区键** | /categoryId |

1. 仍然是在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在 SQL API 导航树中选择新的“products”容器节点。   

1. 选择“新建存储过程”。

1. 在“存储过程 ID”字段中，输入值“createDoc”。 

1. 删除编辑器区域的内容。

1. 创建一个名为 createDoc 的新 JavaScript 函数，该函数没有输入参数：

    ```
    function createDoc() {
        
    }
    ```

1. 在 createDoc 函数中，调用内置的 [getContext][azure.github.io/azure-cosmosdb-js-server/global.html] 方法，将结果存储在名为 context 的变量中：

    ```
    var context = getContext();
    ```

1. 调用上下文对象的 [getCollection][azure.github.io/azure-cosmosdb-js-server/context.html] 方法，并将结果存储在名为 container 的变量中：

    ```
    var container = context.getCollection();
    ```

1. 创建一个名为 doc 的新对象，包含以下两个属性：

    | **属性** | **值** |
    | ---: | :--- |
    | **名称** | first document |
    | **类别 ID** | demo |

    ```
    var doc = {
        name: 'first document',
        categoryId: 'demo'
    };
    ```

1. 调用容器对象的 createDocument 方法，将调用容器对象的 getSelfLink 方法的结果和新文档作为参数传入：

    ```
    container.createDocument(
      container.getSelfLink(),
      doc
    );
    ```

1. 完成后，存储过程代码现在应包括：

    ```
    function createDoc() {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: 'first document',
        categoryId: 'demo'
      };
      container.createDocument(
        container.getSelfLink(),
        doc
      );
    }
    ```

1. 选择“保存”以保留对存储过程所做的更改。

1. 选择“执行”，然后使用以下输入参数执行存储过程：

    | **设置** | **Key** | **值** |
    | ---: | :--- | :--- |
    | **分区键值** | *字符串* | demo |

1. 看到空的结果。 虽然存储过程已成功执行，但 JavaScript 代码从未返回人类可读的响应。

## <a name="implement-best-practices-for-a-stored-procedure"></a>实现存储过程的最佳做法

虽然之前在本实验室中创作的存储过程具有基本的功能，但它也缺少一些应在所有存储过程中实现的常见错误处理技巧。 首先，存储过程假定它将始终有时间完成操作，并且不会检查 createDocument 方法的返回值以确保它有足够的时间。 其次，存储过程假定已成功插入所有文档，而不会检查或抛出任何潜在的错误消息。 最后，存储过程不会将新创建的文档作为最初调用存储过程的请求的 HTTP 响应返回。 你将对存储过程进行这三项更改，以实现常见的最佳做法。

1. 返回到 createDoc 存储过程的编辑器。

1. 在定义 createDoc 函数的代码中找到第 1 行：

    ```
    function createDoc() {
    ```

    更新代码行以包含名为 title 的参数：

    ```
    function createDoc(title) {
    ```

1. 在设置 doc 对象的 name 属性的代码中找到第 5 行：

    ```
    name: 'first document',
    ```

    更新代码行以使用 title 参数的值：

    ```
    name: title,
    ```

1. 在调用 createDocument 方法的代码中找到第 8 行：

    ```
    container.createDocument(
    ```

    更新代码行，将方法调用的结果存储在名为 accepted 的变量中

    ```
    var accepted = container.createDocument(
    ```

1. 在 createDocument 方法调用后添加一个新的代码行，以检查 accepted 变量的值，如果不为 true，则返回该方法：

    ```
    if (!accepted) return;
    ```

1. 最后，向 createDocument 方法调用添加第三个参数，该参数是一个函数，它采用 error 和 newDoc 这两个参数，检查错误是否为 null，然后将 newDoc 设置为存储过程的响应正文：

    ```
    (error, newDoc) => {
      if (error) throw new Error(error.message);
      context.getResponse().setBody(newDoc);
    }
    ```

1. 完成后，存储过程代码现在应包括：

    ```
    function createDoc(title) {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: title,
        categoryId: 'demo'
      }
      var accepted = container.createDocument(
        container.getSelfLink(),
        doc,
        (error, newDoc) => {
          if (error) throw new Error(error.message);
          context.getResponse().setBody(newDoc);
        }
      );
      if (!accepted) return;
    }
    ```

1. 选择“更新”以保留对存储过程所做的更改。

1. 选择“执行”，然后使用以下输入参数执行存储过程：

    | **设置** | **Key** | **值** |
    | ---: | :--- | :--- |
    | **分区键值** | *字符串* | demo |
    | **输入参数** | *字符串* | second document |

1. 看到 JSON 结果。 成功执行存储过程后，新创建的文档已作为原始 HTTP 请求的响应返回。

## <a name="query-documents"></a>查询文档

最后，你将使用数据资源管理器发出 SQL 查询，该查询将返回在本实验室中创建的两个文档。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“SQL API”导航树中选择“products”容器节点。

1. 选择“新建 SQL 查询”。

1. 删除编辑器区域的内容。

1. 创建一个新的 SQL 查询，该查询将返回 categoryId 为 demo 的所有文档：

    ```
    SELECT * FROM docs WHERE docs.categoryId = 'demo'
    ```

1. 选择“执行查询”。

1. 看到在本实验室中创建的两个文档作为执行此查询的结果。

1. 关闭 Web 浏览器窗口或选项卡。

[azure.github.io/azure-cosmosdb-js-server/context.html]: https://azure.github.io/azure-cosmosdb-js-server/Context.html
[azure.github.io/azure-cosmosdb-js-server/global.html]: https://azure.github.io/azure-cosmosdb-js-server/global.html
