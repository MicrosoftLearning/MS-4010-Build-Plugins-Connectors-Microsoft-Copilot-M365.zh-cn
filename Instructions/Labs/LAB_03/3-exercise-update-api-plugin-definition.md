---
lab:
  title: 练习 2 - 更新 API 插件定义
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# 练习 2 - 更新 API 插件定义

下一步是使用 Copilot 用于向用户显示 API 中数据的自适应卡片更新 API 插件定义。

### 练习用时

- **估计完成时间：** 10 分钟

## 任务 1 - 添加自适应卡片以显示菜肴

在 Visual Studio Code 中：

1. 打开 **cards/dish.json** 文件并复制其内容。
1. 打开 **appPackage/ai-plugin.json** 文件。
1. 向 **functions.getDishes.companies.response_sisemantic**属性添加一个名为 **static_template**的新属性，并将**正文**值设置为 **dish.json** 的内容。
1. 完整代码片段如下所示：

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "Container",
            "items": [
              {
                "type": "Image",
                "url": "${image_url}",
                "size": "large"
              },
              {
                "type": "TextBlock",
                "text": "${name}",
                "weight": "Bolder"
              },
              {
                "type": "TextBlock",
                "text": "${description}",
                "wrap": true
              },
              {
                "type": "TextBlock",
                "text": "Allergens: ${if(count(allergens) > 0, join(allergens, ', '), 'none')}",
                "weight": "Lighter"
              },
              {
                "type": "TextBlock",
                "text": "**Price:** €${formatNumber(price, 2)}",
                "weight": "Lighter",
                "spacing": "None"
              }
            ]
          }
      ]
    }
    ```

1. 保存所做更改。

## 任务 2 - 添加自适应卡片模板以显示订单摘要

在 Visual Studio Code 中：

1. 打开 **cards/order.json** 文件并复制其内容。
1. 打开 **appPackage/ai-plugin.json** 文件。
1. 向 **functions.placeOrder.capabilities.response_semantics** 属性添加一个名为 **static_template** 的新属性，并将其内容设置为自适应卡。
1. 完整文件如下所示：

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "TextBlock",
            "text": "Order Confirmation 🤌",
            "size": "Large",
            "weight": "Bolder",
            "horizontalAlignment": "Center"
          },
          {
            "type": "Container",
            "items": [
              {
                "type": "TextBlock",
                "text": "Your order has been successfully placed!",
                "weight": "Bolder",
                "spacing": "Small"
              },
              {
                "type": "FactSet",
                "facts": [
                  {
                    "title": "Order ID:",
                    "value": "${order_id} "
                  },
                  {
                    "title": "Status:",
                    "value": "${status}"
                  },
                  {
                    "title": "Total Price:",
                    "value": "€${formatNumber(total_price, 2)}"
                  }
                ]
              }
            ]
          }
        ]
    }
    ```

1. 保存所做更改。
