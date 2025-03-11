---
title: 设置实验室环境
lab:
  title: 设置实验室环境
  module: Setup
layout: default
nav_order: 2
parent: Python SDK labs
---

# 设置本地实验室环境

理想情况下，应在托管实验室环境中完成这些实验室。 如果要在你自己的计算机上完成它们，可以通过安装以下软件来完成。 使用自己的环境时，可能会遇到意外的对话和行为。 由于可能的本地配置范围广泛，课程团队无法支持你在自己的环境中可能遇到的问题。

## Azure 命令行工具

1. [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) 或 [Azure Cloud Shell](https://shell.azure.com) - 如果要通过 CLI 而不非 Azure 门户执行命令，请安装。

## Python

1. 从 [python.org/downloads] 下载并安装 Python 3.11+，并将 \<install location\>\Python36 和 \<install location>\Python36\Scripts 添加到你的路径。

    - 使用安装程序中的默认选项。

## Git

1. 从 [git-scm.com/downloads] 下载和安装。

    - 使用安装程序中的默认选项。

## Visual Studio Code（和扩展）

1. 从 [code.visualstudio.com/download] 下载和安装。

    - 使用安装程序中的默认选项。

1. 安装后，启动 Visual Studio Code。

1. 在“扩展”菜单中，搜索并安装 Microsoft 的以下扩展****：

    - [适用于 Visual Studio Code 的 Python 扩展][marketplace.visualstudio.com/mms-python.python]

### Azure Cosmos DB 模拟器

1. 从 [docs.microsoft.com/azure/cosmos-db/local-emulator] 下载和安装。
    - 使用安装程序中的默认选项。

### 克隆实验室存储库

如果你还没有将**使用 Azure Cosmos DB 生成助手**的实验室代码存储库克隆到使用此实验室的环境，请按照以下步骤操作。 否则，请在 Visual Studio Code 中打开以前克隆的文件夹。

1. 启动 Visual Studio Code。

    > &#128221; 如果你还不熟悉 Visual Studio Code 界面，请参阅 [Visual Studio Code 入门指南][code.visualstudio.com/docs/getstarted]

1. 打开命令面板并运行 Git: Clone，将 ``https://github.com/solliancenet/microsoft-learning-path-build-copilots-with-cosmos-db-labs`` GitHub 存储库克隆到你选择的本地文件夹中。

    > &#128161; 你可以使用 Ctrl+Shift+P 键盘快捷方式打开命令面板。

1. 克隆存储库后，打开在 Visual Studio Code 中选择的本地文件夹。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator#download-the-emulator
[code.visualstudio.com/download]: https://code.visualstudio.com/download
[git-scm.com/downloads]: https://git-scm.com/downloads
[python.org/downloads]: https://www.python.org/downloads/
[marketplace.visualstudio.com/mms-python.python]: https://marketplace.visualstudio.com/items?itemName=ms-python.python#overview
