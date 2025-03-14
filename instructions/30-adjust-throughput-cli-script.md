---
lab:
  title: 使用 Azure CLI 脚本调整预配的吞吐量
  module: Module 12 - Manage an Azure Cosmos DB for NoSQL solution using DevOps practices
---

# 使用 Azure CLI 脚本调整预配的吞吐量

Azure CLI 是一组命令，可用于管理 Azure 中的各种资源。 Azure Cosmos DB 具有一组丰富的命令，可用于管理 Azure Cosmos DB 帐户的各个方面，而不用考虑所选的 API。

在此实验中，将使用 Azure CLI 创建 Azure Cosmos DB 帐户、数据库和容器。 然后，将使用 Azure CLI 调整预配吞吐量。

## 登录 Azure CLI

使用 Azure CLI 之前，需要先检查 CLI 版本，并使用 Azure 凭据登录。

1. 启动 Visual Studio Code。

1. 打开“终端”菜单，然后选择“新建终端”以打开新的终端实例********。

1. 使用以下命令查看 Azure CLI 的版本：

    ```
    az --version
    ```

1. 使用以下命令查看最常见的 Azure CLI 命令组：

    ```
    az --help
    ```

1. 在登录到 Azure 之前安装 tls/ssl 证书：

    ```
    cd "C:\Program Files\Microsoft SDKs\Azure\CLI2\"
    .\python.exe -m pip install pip-system-certs
    ```

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

## 使用 Azure CLI 创建 Azure Cosmos DB 帐户

Cosmosdb 命令组包含使用 CLI 创建和管理 Azure Cosmos DB 帐户的基本命令****。 由于 Azure Cosmos DB 帐户有可寻址的 URI，因此务必要为新帐户创建全局唯一名称，即使是通过脚本创建的也是如此。

1. 返回到已在 Visual Studio Code 中打开的终端实例****。

1. 使用以下命令查看与 **Azure Cosmos DB** 相关的最常见的 Azure CLI 命令：

    ```
    az cosmosdb --help
    ```

1. 通过 [Get-Random][docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random] PowerShell cmdlet 使用以下命令，创建名为“suffix”的新变量****：

    ```
    $suffix=Get-Random -Maximum 1000000
    ```

    > &#128221; Get-Random cmdlet 会生成一个介于 0 到 1,000,000 之间的随机整数。 这很有用，因为我们的服务需要全局唯一的名称。

1. 使用以下命令，通过硬编码的字符串“csms”和变量替换，创建另一个新的变量名称 accountName，用于注入 $suffix 变量的值************：

    ```
    $accountName="csms$suffix"
    ```

1. 使用以下命令，使用此实验中之前创建或查看的资源组的名称创建另一个新的变量名 resourceGroup****：

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; 例如，如果资源组名为 dp420，则命令将为 $resourceGroup="dp420" 。

1. 使用 echo cmdlet，通过以下命令将 $accountName 和 $resourceGroup 变量的值写入到终端输出中************：

    ```
    echo $accountName
    echo $resourceGroup
    ```

1. 使用以下命令查看 az cosmosdb create 的选项****：

    ```
    az cosmosdb create --help
    ```

1. 使用预定义的变量和以下命令创建新的 Azure Cosmos DB 帐户：

    ```
    az cosmosdb create --name $accountName --resource-group $resourceGroup
    ```

1. 等待 create 命令完成执行并返回，然后再继续此实验****。

    > &#128161; create 命令平均需要两到十二分钟的时间完成****。

## 使用 Azure CLI 创建 Azure Cosmos DB for NoSQL 资源

cosmosdb sql 命令组包含用于管理 Azure Cosmos DB 的 NoSQL API 特定资源的命令****。 始终可以使用 --help 标志来查看这些命令组的选项****。

1. 返回到已在 Visual Studio Code 中打开的终端实例****。

1. 使用以下命令查看与 **Azure Cosmos DB for NoSQL** 相关的最常见的 Azure CLI 命令组：

    ```
    az cosmosdb sql --help
    ```

1. 使用以下命令查看用于管理 Azure Cosmos DB for NoSQL 数据库的 Azure CLI 命令****：

    ```
    az cosmosdb sql database --help
    ```

1. 使用预定义的变量、数据库名称 cosmicworks 和以下命令创建新的 Azure Cosmos DB 数据库****：

    ```
    az cosmosdb sql database create --name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. 等待 create 命令完成执行并返回，然后再继续此实验****。

1. 使用以下命令查看用于管理 Azure Cosmos DB for NoSQL 容器的 Azure CLI 命令****：

    ```
    az cosmosdb sql container --help
    ```

1. 使用预定义的变量、数据库名称 cosmicworks、容器名称 products 和以下命令创建新的 Azure Cosmos DB 容器********：

    ```
    az cosmosdb sql container create --name "products" --throughput 400 --partition-key-path "/categoryId" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. 等待 create 命令完成执行并返回，然后再继续此实验****。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，然后选择在此实验室中创建或查看的资源组，然后选择在此实验室中创建的带有“csms”前缀的 Azure Cosmos DB 帐户资源************。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“NoSQL API”导航树中观察新“products”容器节点。

1. 选择“NoSQL API”导航树中的“products”容器节点，然后选择“缩放和设置”  。

1. 观察“缩放”选项卡中的值。特别请注意“吞吐量”部分中选择了“手动”选项，并且预配的吞吐量设置为“400”RU/s****************。

1. 关闭 Web 浏览器窗口或选项卡。

## 使用 Azure CLI 调整现有容器的吞吐量

Azure CLI 可用于在手动和自动缩放的吞吐量预配之间迁移容器。 如果容器使用自动缩放吞吐量，则可以使用 CLI 动态调整允许的最大吞吐量值。

1. 返回到已在 Visual Studio Code 中打开的终端实例****。

1. 使用以下命令查看用于管理 Azure Cosmos DB for NoSQL 容器吞吐量的 Azure CLI 命令****：

    ```
    az cosmosdb sql container throughput --help
    ```

1. 使用以下命令将“products”容器吞吐量从手动预配迁移到自动缩放预配****：

    ```
    az cosmosdb sql container throughput migrate --name "products" --throughput-type autoscale --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. 等待 migrate 命令完成执行并返回，然后再继续进行本实验****。

1. 使用以下命令查询 products 容器以确定最小可能吞吐量值****：

    ```
    az cosmosdb sql container throughput show --name "products" --query "resource.minimumThroughput" --output "tsv" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. 使用以下命令将 **products** 容器的最大自动缩放吞吐量从当前默认值 **1000** 更新为新值 **5000**：

    ```
    az cosmosdb sql container throughput update --name "products" --max-throughput 5000 --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. 等待 update 命令完成执行并返回，然后再继续进行本实验****。

1. 关闭 Visual Studio Code。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 选择“资源组”，然后选择在此实验室中创建或查看的资源组，然后选择在此实验室中创建的带有“csms”前缀的 Azure Cosmos DB 帐户资源************。

1. 在 Azure Cosmos DB 帐户资源中，导航到“数据资源管理器”窗格 。

1. 在“数据资源管理器”中，展开“cosmicworks”数据库节点，然后在“NoSQL API”导航树中观察新“products”容器节点。

1. 选择“NoSQL API”导航树中的“products”容器节点，然后选择“缩放和设置”  。

1. 观察“缩放”选项卡中的值。特别请注意“吞吐量”部分中选择了“自动缩放”选项，并且预配的吞吐量设置为“5,000”RU/s****************。

1. 关闭 Web 浏览器窗口或选项卡。

[docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random]: https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random
