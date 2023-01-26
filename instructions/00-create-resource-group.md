---
lab:
  title: 创建实验室资源组
  module: Setup
---

# <a name="create-azure-resource-group-for-lab"></a>为实验室创建 Azure 资源组

完成此实验室之前，应创建一个新[资源组][docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal]，将新部署的 Azure 资源放入其中。

1. 在新的 Web 浏览器窗口或选项卡中，导航到 Azure 门户 (``portal.azure.com``)。

1. 使用与你的订阅关联的 Microsoft 凭据登录到门户。

1. 在主页中，选择“资源组” 。

    > &#128161; 或者，展开“&#8801;”菜单，选择“所有服务”，在“所有”类别中选择“资源组”   。

1. 选择“+ 新建”。 

1. 在“创建资源组”弹出窗口中，使用以下设置创建一个新资源组，并将所有剩余设置保留为默认值：

    | **设置** | **值** |
    | ---: | :--- |
    | **订阅** | 你的现有 Azure 订阅 |
    | **资源组** | 为资源组指定唯一名称 |
    | **区域** | 选择任何可用区域 |

1. 等待部署任务完成，然后继续执行此任务。

[docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal]: https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal
