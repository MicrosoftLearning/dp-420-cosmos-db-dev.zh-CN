---
lab:
  title: 设置实验室环境
  module: Setup
---

# 设置本地实验室环境

理想情况下，应在托管实验室环境中完成这些实验室。 如果要在你自己的计算机上完成它们，可以通过安装以下软件来完成。 使用自己的环境时，可能会遇到意外的对话和行为。 由于可能的本地配置范围广泛，课程团队无法支持你在自己的环境中可能遇到的问题。

## Azure 命令行工具

1. [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest) 或 [Azure Cloud Shell](https://shell.azure.com) - 如果要通过 CLI 而不非 Azure 门户执行命令，请安装。

## Node.js

1. 从 [nodejs.org/en/download] 下载并安装 Node.js v18.0.0 或更高版本。

1. 从 [npmjs.com/get-npm] 下载并安装 NPM v10.2.3 或更高版本。

在 Windows 上安装最新版本的 NPM 和node.js 的建议方式为：

- 从 [github.com/coreybutler/nvm-windows] 安装 NVM
- 运行 nvm install latest
- 运行 nvm list（以查看可用的 NPM/node.js 版本）
- 运行 nvm use latest（使用最新的可用版本）

### Git

1. 从 [git-scm.com/downloads] 下载和安装。

    - 使用安装程序中的默认选项。

### Visual Studio Code（和扩展）

1. 从 [code.visualstudio.com/download] 下载和安装。

    - 使用安装程序中的默认选项。

1. 安装后，启动 Visual Studio Code。

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
[nodejs.org/en/download]: https://nodejs.org/en/download
[npmjs.com/get-npm]: https://npmjs.com/get-npm
[github.com/coreybutler/nvm-windows]: https://github.com/coreybutler/nvm-windows
