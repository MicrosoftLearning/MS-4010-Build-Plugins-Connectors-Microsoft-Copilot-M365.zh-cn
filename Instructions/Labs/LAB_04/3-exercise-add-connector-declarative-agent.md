---
lab:
  title: 练习 2 - 创建声明性代理并集成 Graph 连接器
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# 练习 2 - 创建声明性代理并集成 Graph 连接器

在本练习中，你将从头开始创建新的声明性代理，并将其配置为使用外部连接作为其基础设置源。

### 练习用时

- **估计完成时间：** 10 分钟

## 任务 1 - 创建新的声明性代理

创建声明性代理的一种方法是使用 Teams 工具包。 Teams 工具包提供了一个用于创建声明性代理的模板项目，这为你配置代理设置和添加额外功能提供了一个很好的起点。

要创建新的声明性代理，请打开 Visual Studio Code。

在 Visual Studio Code 中：

1. 在“**活动栏**”（边栏）中，打开“**Teams 工具包**”扩展。
1. 在 Teams 工具包面板中，选择“**创建新应用**”按钮。
1. 在“**新建项目**”对话框中，选择“**代理**”选项。
1. 在下一个对话框中，选择“**声明性代理**”选项。
1. 不要通过选择“**无插件**”选项来添加插件。
1. 选择要将项目存储在计算机上的文件夹。
1. 将项目命名为 `da-it-policies`。

## 任务 2 - 配置声明性代理指令

Teams 工具包创建新的声明性代理项目。 若要将其范围限定到应用场景，请更新代理的说明和指令。

在 Visual Studio Code 中：

1. 打开 **appPackage/declarativeAgent.json** 文件
1. 将 **name** 属性的值更新为 `Contoso IT policies`。
1. 将 **description** 属性的值更新为 `Assistant specialized in Contoso IT policies`。
1. 保存所做更改。
1. 更新的文件内容如下所示：

    ```json
    {
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
    }
    ```

1. 打开 **appPackage/instruction.txt** 文件。
1. 将内容更新为：

    ```text
    You are a helpful assistant that can answer questions from users in a friendly manner. You should take your time to respond. Your responses should be accurate and helpful. If you don't have the information, you should say so in your response. When answering follow-up questions, you should review the information you gathered from external sources, if you don't already have the information to give an accurate answer, you should search for more information. Only answer when you have the information to give an accurate response.
    ```

1. 保存所做更改。

## 任务 3 - 将 Microsoft Graph 连接器与声明性代理集成

创建声明性代理后，下一步是将其与 Microsoft Graph 连接器集成，以便它可以访问外部数据。

在 Visual Studio Code 中：

1. 打开 **appPackage/declarativeAgent.json** 文件。
1. 在 **instructions** 属性之后，使用以下代码添加一个名为 **capabilities** 的新属性：

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": ""
            }
          ]
        }
      ]
    } 
    ```

1. 在 **connection_id** 属性中，指定 `policieslocal` 为外部连接的 ID。 `policieslocal` 是上一步骤中创建的 Graph 连接器的外部连接的 ID。
1. 更新的文件内容如下所示：

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": "policieslocal"
            }
          ]
        }
      ]
    } 
    ```

1. 保存所做更改。
