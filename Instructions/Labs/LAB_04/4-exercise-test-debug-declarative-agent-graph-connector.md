---
lab:
  title: 练习 3 - 测试和调试
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# 练习 3 - 测试和调试

在本练习中，你将测试声明性代理并将其部署到 Microsoft 365，并使用 Microsoft 365 Copilot Chat 对其进行测试。

### 练习用时

- **估计完成时间：** 5 分钟

## 任务 1 - 在 Microsoft 365 Copilot 中测试声明性代理

若要测试声明性代理，请将其作为应用部署到 Microsoft 365 租户。 在智能 Microsoft 365 Copilot 副驾驶® 中打开它后，请验证其是否按预期工作。

在 Visual Studio Code 中：

1. 在“**活动栏**”（边栏）中，打开“**Teams 工具包**”扩展。
1. 在“**生命周期**”窗格中，选择“**预配**”。 Teams 工具包将声明性代理项目打包为应用，并将其上传到 Microsoft 365。
1. 打开 Web 浏览器。

在 Web 浏览器中：

1. 导航到 [https://www.microsoft365.com/chat](https://www.microsoft365.com/chat)。
1. 使用属于 Microsoft 365 租户的工作帐户登录。
1. 在智能 Microsoft 365 Copilot 副驾驶® 的侧面板中，选择“**Contoso IT 策略**”代理以激活它。
1. 在聊天文本框中，询问 `What's the acceptable use policy at Contoso?`。
1. 等待代理响应。 请注意回复如何包含对 Graph 连接器引入的外部内容的引用。 每个引用中的 URL 指向存储内容的外部系统中的位置。

    ![智能 Microsoft 365 Copilot 副驾驶® 响应用户的提示的屏幕截图。](../media/LAB_04/3-copilot-response.png)