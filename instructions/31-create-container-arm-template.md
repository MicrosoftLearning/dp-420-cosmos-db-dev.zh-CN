---
lab:
  title: 使用 Azure 资源管理器模板创建 Azure Cosmos DB for NoSQL 容器
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# <a name="create-an-azure-cosmos-db-for-nosql-container-using-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板创建 Azure Cosmos DB for NoSQL 容器

Azure 资源管理器模板是 JSON 文件，以声明方式定义要部署到 Azure 的基础结构。 Azure 资源管理器模板是一种常见的基础结构即代码解决方案，用于将服务部署到 Azure。 Bicep 通过定义一种更易于读取的域特定语言（该语言可用来创建 JSON 模板），进一步发展了此概念。

在此实验室中，你将使用 Azure 资源管理器模板创建新的 Azure Cosmos DB 帐户、数据库和容器。 首先从原始 JSON 创建模板，然后使用 Bicep 域特定语言创建模板。

## <a name="prepare-your-development-environment"></a>准备开发环境

如果你还没有将 DP-420 的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅 [Visual Studio Code 入门指南][code.visualstudio.com/docs/getstarted]

1. 打开命令面板并运行 Git: Clone，将 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 存储库克隆到你选择的本地文件夹中。

    > &#128161; 你可以使用 Ctrl+Shift+P 键盘快捷方式打开命令面板。

1. 克隆存储库后，打开在 Visual Studio Code 中选择的本地文件夹。

## <a name="create-azure-cosmos-db-for-nosql-resources-using-azure-resource-manager-templates"></a>使用 Azure 资源管理器模板创建 Azure Cosmos DB for NoSQL 资源

借助 Azure 资源管理器中的 Microsoft.DocumentDB 资源提供程序，可以使用 JSON 文件部署帐户、数据库和容器。 虽然文件可能很复杂，但它们遵循可预测的格式，并且可以在 Visual Studio Code 扩展的帮助下编写。

> &#128161; 如果遇到问题，并且无法找出模板的语法错误，请使用此[解决方案 Azure 资源管理器模板][github.com/arm-template-guide]作为指南。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 31-create-container-arm-template 文件夹  。

1. 打开 deploy.json 文件。

1. 观察空的 Azure 资源管理器模板：

    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        ]
    }
    ```

1. 在 resources 数组中，添加新的 JSON 对象以创建新的 Azure Cosmos DB 帐户：

    ```
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id))]",
        "location": "[resourceGroup().location]",
        "properties": {
            "databaseAccountOfferType": "Standard",
            "locations": [
                {
                    "locationName": "westus"
                }
            ]
        }
    }
    ```

    对象配置有以下设置：

    | **设置** | **值** |
    | ---: | :--- |
    | **资源类型** | *Microsoft.DocumentDB/databaseAccounts* |
    | **API 版本** | 2021-05-15 |
    | **帐户名称** | 从帐户名称生成的 csmsarm 和 unique 字符串   |
    | **位置** | 资源组的当前位置 |
    | **帐户产品/服务类型** | *标准* |
    | **位置** | 仅美国西部 |

1. 保存 deploy.json 文件。

1. 打开 31-create-container-arm-template 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例 。

    > &#128221; 此命令将打开起始目录已设置为 31-create-container-arm-template 文件夹的终端。

1. 使用以下命令开始 Azure CLI 的交互式登录过程：

    ```
    az login
    ```

1. Azure CLI 将自动打开 Web 浏览器窗口或选项卡。在浏览器实例中，使用与订阅关联的 Microsoft 凭据登录 Azure CLI。

1. 关闭 Web 浏览器窗口或选项卡。

1. 检查实验室提供商是否为你创建了资源组，如果已创建，请记录其名称，因为你将在下一部分中用到它。

    ```
    az group list --query "[].{ResourceGroupName:name}" -o table
    ```
    
    此命令可能会返回多个资源组名称。

1. （可选）如果没有为你创建资源组，请选择资源组名称并进行创建。 请注意，某些实验室环境可能被锁定，你需要管理员为你创建资源组。

    i. 从此列表中获取离你最近的位置名称

    ```
    az account list-locations --query "sort_by([].{YOURLOCATION:name, DisplayName:regionalDisplayName}, &YOURLOCATION)" --output table
    ```

    ii. 创建资源组。  请注意，某些实验室环境可能被锁定，你需要管理员为你创建资源组。
    ```
    az group create --name YOURRESOURCEGROUPNAME --location YOURLOCATION
    ```

1. 使用以下命令，使用此实验中之前创建或查看的资源组的名称创建一个新的变量名 resourceGroup：

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; 例如，如果资源组名为 dp420，则命令将为 $resourceGroup="dp420" 。

1. 使用 echo cmdlet，通过以下命令将 $resourceGroup 变量的值写入到终端输出中 ：

    ```
    echo $resourceGroup
    ```

1. 使用 [az deployment group create][docs.microsoft.com/cli/azure/deployment/group] 命令部署 Azure 资源管理器模板：

    ```
    az deployment group create --name "arm-deploy-account" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 将集成终端保持打开，然后返回到 deploy.json 文件的编辑器。

1. 在 resources 数组中，添加另一个新的 JSON 对象以创建新的 Azure Cosmos DB for NoSQL 数据库：

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]"
        ],
        "properties": {
            "resource": {
                "id": "cosmicworks"
            }
        }
    }
    ```

    对象配置有以下设置：

    | **设置** | **值** |
    | ---: | :--- |
    | **资源类型** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API 版本** | 2021-05-15 |
    | **帐户名称** | 从帐户名称和 /cosmicworks 生成的 csmsarm 和 unique 字符串  |
    | **资源 ID** | cosmicworks |
    | **依赖项** | 之前在模板中创建的 databaseAccount |

1. 保存 deploy.json 文件。

1. 返回到集成终端。

1. 使用 az deployment group create 命令部署 Azure 资源管理器模板：

    ```
    az deployment group create --name "arm-deploy-database" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 将集成终端保持打开，然后返回到 deploy.json 文件的编辑器。

1. 在 resources 数组中，添加另一个新的 JSON 对象以创建新的 Azure Cosmos DB for NoSQL 容器：

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks/products')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]",
            "[resourceId('Microsoft.DocumentDB/databaseAccounts/sqlDatabases', concat('csmsarm', uniqueString(resourceGroup().id)), 'cosmicworks')]"
        ],
        "properties": {
            "options": {
                "throughput": 400
            },
            "resource": {
                "id": "products",
                "partitionKey": {
                    "paths": [
                        "/categoryId"
                    ]
                }
            }
        }
    }
    ```

    对象配置有以下设置：

    | **设置** | **值** |
    | ---: | :--- |
    | **资源类型** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers* |
    | **API 版本** | 2021-05-15 |
    | **帐户名称** | 从帐户名称和 /cosmicworks/products 生成的 csmsarm 和 unique 字符串&amp;&amp;  |
    | **资源 ID** | products |
    | **吞吐量** | *400* |
    | **分区键** | /categoryId |
    | **依赖项** | 之前在模板中创建的帐户和数据库 |

1. 保存 deploy.json 文件。

1. 返回到集成终端。

1. 使用 az deployment group create 命令部署最终 Azure 资源管理器模板：

    ```
    az deployment group create --name "arm-deploy-container" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 关闭集成终端。

## <a name="observe-deployed-azure-cosmos-db-resources"></a>观察已部署的 Azure Cosmos DB 资源

部署 Azure Cosmos DB for NoSQL 资源后，可以在 Azure 门户中导航到该资源。 你将使用数据资源管理器，验证帐户、数据库和容器是否已全部部署并正确配置。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，然后选择在此实验室中创建或查看的资源组，然后选择在此实验室中创建的带有“csmsarm”前缀的 Azure Cosmos DB 帐户资源  。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“NoSQL API”导航树中观察新“products”容器节点。

1. 选择“NoSQL API”导航树中的“products”容器节点，然后选择“缩放和设置”  。

1. 观察“缩放”部分中的值。 特别请注意“吞吐量”部分中选择了“手动”选项，并且预配的吞吐量设置为 400 RU/s  。

1. 观察“设置”部分中的值。 特别请注意，“分区键”值设置为“/categoryId” 。

1. 关闭 Web 浏览器窗口或选项卡。

## <a name="create-azure-cosmos-db-for-nosql-resources-using-bicep-templates"></a>使用 Bicep 模板创建 Azure Cosmos DB for NoSQL 资源

Bicep 是一种高效的域特定语言，与使用 Azure 资源管理器模板相比，使用该语言可以更简单、更轻松地部署 Azure 资源。 你将使用 Bicep 和不同的名称部署相同的资源，以说明\[差别\]。

> &#128161; 如果遇到问题，并且无法找出模板的语法错误，请使用此[解决方案 Bicep 模板][github.com/bicep-template-guide]作为指南。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 31-create-container-arm-template 文件夹  。

1. 打开空的 deploy.bicep 文件。

1. 在该文件中，添加一个新的对象来创建新的 Azure Cosmos DB 帐户：

    ```
    resource Account 'Microsoft.DocumentDB/databaseAccounts@2021-05-15' = {
      name: 'csmsbicep${uniqueString(resourceGroup().id)}'
      location: resourceGroup().location
      properties: {
        databaseAccountOfferType: 'Standard'
        locations: [
          { 
            locationName: 'westus' 
          }
        ]
      }
    }
    ```

    对象配置有以下设置：

    | **设置** | **值** |
    | ---: | :--- |
    | **Alias** | *帐户* |
    | **名称** | 从帐户名称生成的 csmsarm 和 unique 字符串  |
    | **资源类型** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API 版本** | 2021-05-15 |
    | **位置** | 资源组的当前位置 |
    | **帐户产品/服务类型** | *标准* |
    | **位置** | 仅美国西部 |

1. 保存 deploy.bicep 文件。

1. 打开 31-create-container-arm-template 文件夹的上下文菜单，然后选择“在集成终端中打开”以打开一个新的终端实例 。

1. 使用以下命令，使用此实验中之前创建或查看的资源组的名称创建一个新的变量名 resourceGroup：

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; 例如，如果资源组名为 dp420，则命令将为 $resourceGroup="dp420" 。

1. 使用 az deployment group create 命令部署 Bicep 模板：

    ```
    az deployment group create --name "bicep-deploy-account" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 将集成终端保持打开，然后返回到 deploy.bicep 文件的编辑器。

1. 在该文件中，添加另一个新的对象以创建新的 Azure Cosmos DB 数据库：

    ```
    resource Database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2021-05-15' = {
      parent: Account
      name: 'cosmicworks'
      properties: {
        resource: {
            id: 'cosmicworks'
        }
      }
    }
    ```

    对象配置有以下设置：

    | **设置** | **值** |
    | ---: | :--- |
    | **父级** | 之前在模板中创建的帐户 |
    | **Alias** | *Database* |
    | **名称** | cosmicworks  |
    | **资源类型** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API 版本** | 2021-05-15 |
    | **资源 ID** | cosmicworks |

1. 保存 deploy.bicep 文件。

1. 返回到集成终端。

1. 使用 az deployment group create 命令部署 Bicep 模板：

    ```
    az deployment group create --name "bicep-deploy-database" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 将集成终端保持打开，然后返回到 deploy.bicep 文件的编辑器。

1. 在该文件中，添加另一个新的对象以创建新的 Azure Cosmos DB 容器：

    ```
    resource Container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2021-05-15' = {
      parent: Database
      name: 'products'
      properties: {
        options: {
          throughput: 400
        }
        resource: {
          id: 'products'
          partitionKey: {
            paths: [
              '/categoryId'
            ]
          }
        }
      }
    }
    ```

    对象配置有以下设置：

    | **设置** | **值** |
    | ---: | :--- |
    | **父级** | 之前在模板中创建的数据库 |
    | **Alias** | *容器* |
    | **名称** | products  |
    | **资源 ID** | products |
    | **吞吐量** | *400* |
    | **分区键路径** | /categoryId |

1. 保存 deploy.bicep 文件。

1. 返回到集成终端。

1. 使用 az deployment group create 命令部署最终 Bicep 模板：

    ```
    az deployment group create --name "bicep-deploy-container" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 关闭集成终端。

1. 关闭 Visual Studio Code。

## <a name="observe-bicep-template-deployment-results"></a>观察 Bicep 模板部署结果

可以使用与 Azure 资源管理器部署相同的许多技术验证 Bicep 部署。 你不仅会验证你的帐户、数据库和容器是否已成功部署，还将查看所有六个部署中的部署历史记录。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，然后选择之前在此实验室中创建或查看的资源组。

1. 在资源组中，导航到“部署”窗格。

1. 观察 Azure 资源管理器模板和 Bicep 文件中的六个部署。

1. 仍在资源组中，导航到“概述”窗格。

1. 仍在资源组中，选择在此实验室中创建的 Azure Cosmos DB 帐户资源，其中包含 csmsbicep 前缀 。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“NoSQL API”导航树中观察新“products”容器节点。

1. 选择“NoSQL API”导航树中的“products”容器节点，然后选择“缩放和设置”  。

1. 观察“缩放”部分中的值。 特别请注意“吞吐量”部分中选择了“手动”选项，并且预配的吞吐量设置为 400 RU/s  。

1. 观察“设置”部分中的值。 特别请注意，“分区键”值设置为“/categoryId” 。

1. 关闭 Web 浏览器窗口或选项卡。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/cli/azure/deployment/group]: https://docs.microsoft.com/cli/azure/deployment/group
[github.com/arm-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.json
[github.com/bicep-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.bicep
