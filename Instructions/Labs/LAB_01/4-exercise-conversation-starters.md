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
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
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
    "name": "Microsoft 365 Knowledge Expert",
    "description": "Microsoft 365 Knowledge Expert that can answer any question you have about Microsoft 365",
    "instructions": "$[file('instruction.txt')]",
    "capabilities": [
        {
            "name": "WebSearch",
            "sites": [
                {
                    "url": "https://learn.microsoft.com/microsoft-365/"
                }
            ]
        }
    ],
  "conversation_starters": [
       {
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
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
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
  ]
}
```

## 任务 2 - 在智能 Microsoft 365 Copilot 副驾驶® 对话助手中测试声明性代理

接下来，上传更改并启动调试会话。

在 Visual Studio Code 中：

1. 在 **活动栏**中，打开“**Teams 工具包**”扩展。
1. 在“**生命周期**”部分，选择“**预配**”，然后选择“**发布**”。
1. **确认**你要向应用程序目录提交更新。
1. 等待上传完成。

在 Web 浏览器中继续操作：

1. 在 **Microsoft 365 Copilot** 中，选择右上角的图标以**展开 Copilot 侧面板**。
1. 在代理列表中查找“**产品支持**”并选择它，以输入沉浸式体验，直接与代理聊天。 请注意，在清单中定义的对话开场白已显示在用户界面中。

![Microsoft Edge 的屏幕截图，其中显示在沉浸式体验中使用自定义对话开场白的 Microsoft 365 知识专家声明性代理。](../media/LAB_01/test-conversation-starters.png)

关闭浏览器，停止 Visual Studio Code 中的调试会话。