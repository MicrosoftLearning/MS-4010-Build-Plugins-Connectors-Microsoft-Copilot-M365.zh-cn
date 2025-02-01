---
lab:
  title: 练习 3 - 将对话开场白添加到声明性代理
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# 练习 3 - 将对话开场白添加到声明性代理

在本练习中，你将更新声明性代理，以包含为用户提供示例提示的对话开场白，帮助用户了解他们可提出的问题类型。

### 练习用时

- **估计完成时间：** 5 分钟

## 任务 1 - 添加对话开场白

在 Visual Studio Code 中：

1. 在 **appPackage** 文件夹中，打开 **declarativeAgent.json** 文件。
1. 将以下代码片段添加到文件：

   ```json
   "conversation_starters": [
       {
           "title": "Product information",
           "text": "Tell me about Eagle Air"
       },
       {
           "title": "Returns policy",
           "text": "What is the returns policy?"
       },
       {
           "title": "Product information",
           "text": "Can you provide information on a specific product?"
       },
       {
           "title": "Product troubleshooting",
           "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
       },
       {
           "title": "Repair information",
           "text": "Can you provide information on how to get a product repaired?"
       },
       {
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
   ]
   ```

1. 保存所做更改。

**declarativeAgent.json**文件应如下所示：

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
  "version": "v1.0",
  "name": "Product support",
  "description": "Product support agent that can help answer customer queries about Contoso Electronics products",
  "instructions": "$[file('instruction.txt')]",
  "capabilities": [
    {
      "name": "OneDriveAndSharePoint",
      "items_by_url": [
        {
          "url": "https://{tenant}-my.sharepoint.com/personal/{user}/Documents/Products"
        }
      ]
    }
  ],
  "conversation_starters": [
    {
      "title": "Product information",
      "text": "Tell me about Eagle Air"
    },
    {
      "title": "Returns policy",
      "text": "What is the returns policy?"
    },
    {
      "title": "Product information",
      "text": "Can you provide information on a specific product?"
    },
    {
      "title": "Product troubleshooting",
      "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
    },
    {
      "title": "Repair information",
      "text": "Can you provide information on how to get a product repaired?"
    },
    {
      "title": "Contact support",
      "text": "How can I contact support for help?"
    }
  ]
}
```

## 任务 2 - 在 Microsoft 365 Copilot 中测试声明性代理

接下来，上传更改并启动调试会话。

在 Visual Studio Code 中：

1. 在 **活动栏**中，打开“**Teams 工具包**”扩展。
1. 在“**生命周期**”部分中，选择“**预配**”。
1. 等待上传完成。
1. 在“**活动栏**”中，切换到“**运行和调试**”视图。
1. 选择配置下拉列表旁边的“**开始调试**”按钮，或按 <kbd>F5</kbd>。 将启动新的浏览器窗口并导航到 Microsoft 365 Copilot。

接下来，在 Microsoft 365 中测试声明性代理并验证结果。

在 Web 浏览器中继续操作：

1. 在 **Microsoft 365 Copilot** 中，选择右上角的图标以**展开 Copilot 侧面板**。
1. 在代理列表中查找“**产品支持**”并选择它，以输入沉浸式体验，直接与代理聊天。 请注意，在清单中定义的对话开场白已显示在用户界面中。

![Microsoft Edge 的屏幕截图，其中显示了在沉浸式体验中使用自定义对话开场白的产品支持声明性代理。](../media/LAB_01/test-conversation-starters.png)

关闭浏览器，停止 Visual Studio Code 中的调试会话。