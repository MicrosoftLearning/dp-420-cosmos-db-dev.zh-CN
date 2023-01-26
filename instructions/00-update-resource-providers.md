---
lab:
  title: 启用资源提供程序
  module: Setup
---

# <a name="enable-azure-resource-providers"></a>启用 Azure 资源提供程序

有一些资源提供程序必须在 Azure 订阅中注册。 请按照以下步骤操作，确保已注册它们。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 在“主页”上，选择“订阅”。

    > &#128161; 或者;展开“&#8801;”菜单，选择“所有服务”，在“所有”类别中选择“订阅”。

1. 选择 Azure 订阅。

    > &#128221; 如果有多个订阅，请选择通过兑换 Azure Pass 创建的订阅。

1. 在订阅的边栏选项卡的“设置”部分，选择“资源提供程序”。

1. 在资源提供程序列表中，确保注册以下提供程序：
    - [Microsoft.DocumentDB][docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]
    - [Microsoft.Insights][docs.microsoft.com/azure/templates/microsoft.insights/components]
    - [Microsoft.KeyVault][docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]
    - [Microsoft.Search][docs.microsoft.com/azure/templates/microsoft.search/searchservices]
    - [Microsoft.Web][docs.microsoft.com/azure/templates/microsoft.web/sites]

    > &#128221; 如果有提供程序未注册，请选择该提供程序，然后选择“注册”。

1. 关闭 Web 浏览器窗口或选项卡。

[docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]: https://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts
[docs.microsoft.com/azure/templates/microsoft.insights/components]: https://docs.microsoft.com/azure/templates/microsoft.insights/components
[docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]: https://docs.microsoft.com/azure/templates/microsoft.keyvault/vaults
[docs.microsoft.com/azure/templates/microsoft.search/searchservices]: https://docs.microsoft.com/azure/templates/microsoft.search/searchservices
[docs.microsoft.com/azure/templates/microsoft.web/sites]: https://docs.microsoft.com/azure/templates/microsoft.web/sites
