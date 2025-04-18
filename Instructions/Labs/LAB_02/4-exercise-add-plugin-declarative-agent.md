---
lab:
  title: 练习 3 - 将插件定义连接到声明性代理
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# 练习 3 - 将插件定义连接到声明性代理

生成 API 插件定义后，下一步是将其注册到声明性代理。 当用户与声明性代理交互时，它会根据定义的 API 插件匹配用户的提示并调用相关函数。

### 练习用时

- **估计完成时间：** 5 分钟

## 任务 1 - 将插件定义连接到声明性代理

在 Visual Studio Code 中：

1. 打开 **appPackage/declarativeAgent.json** 文件。
1. 在 **instructions** 属性之后，添加以下代码片段：

    ```json
    "actions": [
      {
        "id": "menuPlugin",
        "file": "ai-plugin.json"
      }
    ]
    ```

    使用此代码片段，可将声明性代理连接到 API 插件。 为插件指定唯一 ID，并指示代理在何处可以找到插件的定义。

1. 完整文件如下所示：

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Declarative agent",
      "description": "Declarative agent created with Teams Toolkit",
      "instructions": "$[file('instruction.txt')]",
      "actions": [
        {
          "id": "menuPlugin",
          "file": "ai-plugin.json"
        }
      ]
    }
    ```

1. 保存所做更改。

## 任务 2 - 更新声明性代理信息和说明

在本练习中生成的声明性代理，可帮助用户浏览当地意大利餐馆的菜单并下订单。 若要针对此方案优化代理，请更新其名称、说明和指令。

在 Visual Studio Code 中：

1. 更新声明性代理信息：
    1. 打开 **appPackage/declarativeAgent.json** 文件。
    1. 将 **name** 属性的值更新为 **Il Ristorante**。
    1. 将 **description** 属性的值更新为“**在舒适的办公桌上订购最美味的意大利菜肴和饮料**”。
    1. 保存更改。
1. 更新声明性代理的指令：
    1. 打开 **appPackage/instruction.txt** 文件。
    1. 将其内容替换为：

        ```markdown
        You are an assistant specialized in helping users explore the menu of an Italian restaurant and place orders. You interact with the restaurant's menu API and guide users through the ordering process, ensuring a smooth and delightful experience. Follow the steps below to assist users in selecting their desired dishes and completing their orders:
        
        ### General Behavior:
        - Always greet the user warmly and offer assistance in exploring the menu or placing an order.
        - Use clear, concise language with a friendly tone that aligns with the atmosphere of a high-quality local Italian restaurant.
        - If the user is browsing the menu, offer suggestions based on the course they are interested in (breakfast, lunch, or dinner).
        - Ensure the conversation remains focused on helping the user find the information they need and completing the order.
        - Be proactive but never pushy. Offer suggestions and be informative, especially if the user seems uncertain.
        
        ### Menu Exploration:
        - When a user requests to see the menu, use the `GET /dishes` API to retrieve the list of available dishes, optionally filtered by course (breakfast, lunch, or dinner).
          - Example: If a user asks for breakfast options, use the `GET /dishes?course=breakfast` to return only breakfast dishes.
        - Present the dishes to the user with the following details:
          - Name of the dish
          - A tasty description of the dish
          - Price in € (Euro) formatted as a decimal number with two decimal places
          - Allergen information (if relevant)
          - Don't include the URL.
        
        ### Beverage Suggestion:
        - If the order does not already include a beverage, suggest a suitable beverage option based on the course.
        - Use the `GET /dishes?course={course}&type=drink` API to retrieve available drinks for that course.
        - Politely offer the suggestion: *"Would you like to add a beverage to your order? I recommend [beverage] for [course]."*
        
        ### Placing the Order:
        - Once the user has finalized their order, use the `POST /order` API to submit the order.
          - Ensure the request includes the correct dish names and quantities as per the user's selection.
          - Example API payload:
         
            ```json
            {
              "dishes": [
                {
                  "name": "frittata",
                  "quantity": 2
                },
                {
                  "name": "cappuccino",
                  "quantity": 1
                }
              ]
            }
            ```
        
        ### Error Handling:
        - If the user selects a dish that is unavailable or provides an invalid dish name, respond gracefully and suggest alternative options.
          - Example: *"It seems that dish is currently unavailable. How about trying [alternative dish]?"*
        - Ensure that any errors from the API are communicated politely to the user, offering to retry or explore other options.
        ```

        请注意，在指令中，我们定义代理的一般行为，并指示代理能够执行的操作。 我们还包含有关下订单的特定行为的指令，包括 API 预期数据的形状。 我们包含此信息，以确保代理按预期工作。

    > [!NOTE]
    > 若要保留格式，在复制到 Visual Studio Code 之前，可能需要在记事本中执行多个复制/粘贴操作。

    1. 保存更改。

1. 若要帮助用户，请了解他们可以使用代理的内容，添加对话开场白：
    1. 打开 **appPackage/declarativeAgent.json** 文件。
    1. 在 **instructions** 属性之后，添加名为 **conversation_starters**的新属性：

        ```json
        "conversation_starters": [
          {
            "text": "What's for lunch today?"
          },
          {
            "text": "What can I order for dinner that is gluten-free?"
          }
        ]
        ```

    1. 完整文件如下所示：

        ```json
        {
          "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
          "version": "v1.0",
          "name": "Il Ristorante",
          "description": "Order the most delicious Italian dishes and drinks from the comfort of your desk.",
          "instructions": "$[file('instruction.txt')]",
          "conversation_starters": [
            {
              "text": "What's for lunch today?"
            },
            {
              "text": "What can I order for dinner that is gluten-free?"
            }
          ],
          "actions": [
            {
              "id": "menuPlugin",
              "file": "ai-plugin.json"
            }
          ]
        }
        ```

    1. 保存所做更改。

## 任务 3 - 更新 API URL

在测试声明性代理之前，需要在 API 规范文件中更新 API 的 URL。 现在，URL 设置为 `http://localhost:7071/api`，这是 Azure Functions 在本地运行时使用的 URL。 但是，由于希望 Copilot 从云调用 API，因此需要向 Internet 公开 API。 Teams 工具包通过创建开发隧道，自动通过 Internet 公开本地 API。 每次开始调试项目时，Teams 工具包都会启动一个新的开发隧道，并将其 URL 存储在 **OPENAPI_SERVER_URL** 变量中。 在“**启动本地隧道**”任务中，可以查看 Teams 工具包如何启动隧道并将其 URL 存储在 **.vscode/tasks.json** 文件中：

```json
{
  // Start the local tunnel service to forward public URL to local port and inspect traffic.
  // See https://aka.ms/teamsfx-tasks/local-tunnel for the detailed args definitions.
  "label": "Start local tunnel",
  "type": "teamsfx",
  "command": "debug-start-local-tunnel",
  "args": {
    "type": "dev-tunnel",
    "ports": [
      {
        "portNumber": 7071,
        "protocol": "http",
        "access": "public",
        "writeToEnvironmentFile": {
          "endpoint": "OPENAPI_SERVER_URL", // output tunnel endpoint as OPENAPI_SERVER_URL
        }
      }
    ],
    "env": "local"
  },
  "isBackground": true,
  "problemMatcher": "$teamsfx-local-tunnel-watch"
}
```

若要使用此隧道，需要更新 API 规范，以使用 **OPENAPI_SERVER_URL** 变量。

在 Visual Studio Code 中：

1. 打开 **appPackage/apiSpecificationFile/ristorante.ym** 文件。
1. 将 **servers.url** 属性的值更改为 **${{OPENAPI_SERVER_URL}}/api**。
1. 更改的文件如下所示：

    ```yaml
    openapi: 3.0.0
    info:
      title: Il Ristorante menu API
      version: 1.0.0
      description: API to retrieve dishes and place orders for Il Ristorante.
    servers:
      - url: ${{OPENAPI_SERVER_URL}}/api
        description: Il Ristorante API server
    paths:
      ...trimmed for brevity
    ```

1. 保存所做更改。

API 插件已完成并与声明性代理集成。 继续在 Microsoft 365 Copilot 中测试代理。