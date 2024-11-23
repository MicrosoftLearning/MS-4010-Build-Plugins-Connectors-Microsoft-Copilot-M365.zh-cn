---
lab:
  title: 练习 1 - 创建消息扩展
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 练习 1 - 创建消息扩展

在本练习中，你将创建一个消息扩展解决方案。 在 Visual Studio 中使用 Teams 工具包创建所需的资源，然后在 Microsoft Teams 中启动调试会话并进行测试。

![Microsoft Teams 中基于搜索的消息扩展返回的搜索结果屏幕截图。](../media/1-search-results.png)

### 练习用时

  - **估计完成时间：** 25 分钟

## 任务 1 - 使用适用于 Visual Studio 的 Teams 工具包创建新项目

首先，创建一个新的 Microsoft Teams 应用项目，该项目配置了包含搜索命令的消息扩展。 虽然可以使用 Teams Toolkit for Visual Studio 项目模板创建项目，但需要对基架项目进行更改才能完成本模块。 你可以改用作为 NuGet 包提供的自定义项目模板。 使用自定义模板的好处是，它会创建包含必要文件和依赖项的解决方案，从而节省时间。

1. 以管理员身份打开新的 PowerShell 会话。

1. 通过运行以下命令更改为适当的工作目录：

    ```Powershell
    cd ~\Documents
    ```

1. 首先，通过运行以下命令从 NuGet 安装模板包：

    ```PowerShell
    dotnet new install M365Advocacy.Teams.Templates
    ```

1. 通过运行以下命令来创建新项目：

    ```PowerShell
    dotnet new teams-msgext-search --name "ProductsPlugin" `
      --internal-name "msgext-products" `
      --display-name "Contoso products" `
      --short-description "Product look up tool." `
      --full-description "Get real-time product information and share them in a conversation." `
      --command-id "Search" `
      --command-description "Find products by name" `
      --command-title "Products" `
      --parameter-name "ProductName" `
      --parameter-title "Product name" `
      --parameter-description "The name of the product as a keyword" `
      --allow-scripts Yes
    ```

1. 等待创建项目。

1. 通过运行 `cd ProductsPlugin` 更改为项目目录。

1. 通过运行 `.\ProductsPlugin.sln` 在 Visual Studio 中打开解决方案。

1. 从应用程序选择窗口中选择 **Visual Studio 2022**，然后选择“**始终**”。

## 任务 2 - 创建开发隧道

当用户与消息扩展交互时，机器人服务会将请求发送到 Web 服务。 在开发过程中，Web 服务在本地计算机上运行。 若要允许机器人服务访问 Web 服务，需要使用开发隧道将其暴露在计算机之外。

![Visual Studio 中“开发隧道”窗口的屏幕截图。](../media/14-select-dev-tunnel.png)

在 Visual Studio 中继续操作：

1. 在工具栏上，选择“**开始**”按钮旁边的下拉列表，展开“**开发隧道（无活动隧道）**”菜单，然后选择“**创建隧道**”。

1. 在对话框中，指定以下值：

    1. **帐户**：使用提供给你的 Microsoft 365 帐户登录。 选择“工作或学校帐户”****。

    1. **名称**：msgext-products

    1. **隧道类型**：临时

    1. **访问**：公开

1. 通过选择“**确定**”创建隧道。 此时会出现提示，指出新隧道现在已成为当前活动隧道。

1. 通过选择“**确定**”关闭提示。

## 任务 3 - 准备资源

现在一切就绪后，使用 Teams 工具包，运行**准备 Teams 应用依赖项**进程来创建所需的资源。

![Visual Studio 中展开的 Teams 工具包菜单的屏幕截图。](../media/15-prepare-teams-app-dependencies.png)

“准备 Teams 应用依赖项”进程使用活动开发隧道 URL 更新 **TeamsApp\\env\\.env.local** 文件中的 **BOT_ENDPOINT** 和 **BOT_DOMAIN** 环境变量，并执行 **TeamsApp\\teamsapp.local.yml** 文件中所述的操作。

花点时间浏览  **teamsapp.local.yml** 文件中的步骤。

在 Visual Studio 中继续操作：

1. 打开“**项目**”菜单（或者，可以在解决方案资源管理器中选择 **TeamsApp** 项目），展开 **Teams 工具包**菜单，然后选择“**准备 Teams 应用依赖项**”。

1. 在“**Microsoft 365 帐户**”对话框中，登录或选择现有帐户以访问 Microsoft 365 租户，然后选择“**继续**”。

1. 在“**预配**”对话框中，登录或选择要用于将资源部署到 Azure 的一个现有帐户，并指定以下值：

      1. **订阅名称**：从下拉列表中选择订阅。

      1. **资源组**：从下拉列表中选择预填充的资源组。

1. 通过选择“**预配**”在 Azure 中创建资源。

1. 在 Teams 工具包警告提示中，选择“**预配**”。

1. 在 Teams 工具包信息提示中，选择 **“查看预配的资源**”以打开新的浏览器窗口。

花点时间浏览在 Azure 中创建的资源，并查看在 **.env.local** 文件中创建的环境变量。

> [!NOTE]
> 关闭并重新打开 Visual Studio 时，开发隧道 URL 将更改，将不再选择该隧道作为活动隧道。 如果发生这种情况，则需要再次选择隧道，并运行**准备 Teams 应用依赖项**进程以反映应用部件清单中更新的 URL。

## 任务 4 - 运行和调试

Teams 工具包使用多项目启动配置文件。 要运行项目，需要在 Visual Studio 中启用预览功能。

在 Visual Studio 中：

1. 打开“**工具**”菜单，然后选择“**选项...**”

1. 在搜索框中，输入“**multi-project**”。

1. 在“**环境**”下，选择“**预览功能**”。

1. 选中“**启用多项目启动配置文件**”旁边的框，然后选择“**确定**”保存更改。

要启动调试会话并在 Microsoft Teams 中安装应用，请执行以下操作：

1. 按 <kbd>F5</kbd> 或从工具栏中选择“**开始**”。

1. 信任或批准首次启动应用时弹出的任何 SSL 认证警告。

1. 等待浏览器窗口打开，应用安装对话框将显示在 Microsoft Teams Web 客户端中。 出现提示时，输入 Microsoft 365 帐户凭据。

1. 在应用安装对话框中，选择“**添加**”。

要测试消息扩展，请执行以下操作：

1. 打开新聊天 (<kbd>Alt+N</kbd>)，首先在“**聊天对象**”框中键入“**Contoso**”，然后选择“**Contoso 产品支持**”。

    > [!NOTE]
    > 如果与自己的用户帐户聊天，则不起作用。 必须是另一个用户或组。

1. 在消息撰写区域中，键入“**/apps**”以打开应用选取器。

1. 在应用列表中，选择“**Contoso 产品**”，以打开消息扩展。

1. 在文本框中输入 **hello**。 可能需要多次输入搜索。

1. 等待搜索结果显示。

1. 在结果列表中，选择 **hello**，以将卡片嵌入撰写消息框中

![Microsoft Teams 中基于搜索的消息扩展返回的搜索结果屏幕截图。](../media/1-search-results.png)

返回到 Visual Studio，然后从工具栏中选择“**停止**”，或按 <kbd>Shift</kbd> + <kbd>F5</kbd> 停止调试会话。

[继续进行下一个练习...](./3-exercise-add-single-sign-on.md)