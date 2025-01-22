---
lab:
  title: 练习 2 - 生成 API 插件定义
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# 练习 2 - 生成 API 插件定义

下一步是将插件定义添加到项目。 属性定义包含以下信息：

- 插件可以执行哪些操作。
- 它期望和返回的数据是什么形状。
- 声明性代理必须如何调用基础 API。

### 练习用时

- **估计完成时间：** 10 分钟

## 任务 1 - 添加基本插件定义结构

在 Visual Studio Code 中：

1. 在 **appPackage** 文件夹中，添加名为 **ai-plugin.json** 的新文件。
1. 粘贴以下内容：

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [ 
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

    文件包含 API 插件的基本结构，其中包含人类和模型的说明。 **description_for_model** 包含有关插件功能的详细信息，以帮助代理了解何时应考虑调用该插件。
1. 保存所做更改。

## 任务 2 - 定义函数

API 插件定义一个或多个函数，这些函数映射到 API 规范中定义的 API 操作。 每个函数都包含名称、说明以及指示代理如何向用户显示数据的响应定义。

### 定义用于检索菜单的函数

首先定义函数以检索有关今日菜单的信息。

在 Visual Studio Code 中：

1. 打开 **appPackage/ai-plugin.json** 文件。
1. 在**函数**数组中，添加以下代码段：

    ```json
    {
      "name": "getDishes",
      "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
      "capabilities": {
        "response_semantics": {
          "data_path": "$.dishes",
          "properties": {
            "title": "$.name",
            "subtitle": "$.description"
          }
        }
      }
    }
    ```

    首先，定义从 API 规范调用 **getDishes** 操作的函数。 接下来，提供函数说明。 此说明很重要，因为 Copilot 使用它来确定要为用户的提示调用哪个函数。

    在 response_semantics 属性中，指定 Copilot 如何显示从 API 接收的数据。 由于 API 返回有关 **dishes** 属性中菜单上的菜肴信息，因此将 **data_path** 属性设置为 `$.dishes` JSONPath 表达式。

    接下来，在**属性**部分中，映射 API 响应中表示 title、description 和 URL的属性。 因为在这种情况下，菜肴没有 URL，所以只映射 **title** 和 **description**。

1. 完整代码片段如下所示：

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          "capabilities": {
            "response_semantics": {
              "data_path": "$.dishes",
              "properties": {
                "title": "$.name",
                "subtitle": "$.description"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. 保存所做更改。

### 定义下订单的函数

接下来，定义下订单的函数。

在 Visual Studio Code 中：

1. 打开 **appPackage/ai-plugin.json** 文件。
1. 将以下代码添加到 **functions** 数组末尾：

    ```json
    {
      "name": "placeOrder",
      "description": "Places an order and returns the order details",
      "capabilities": {
        "response_semantics": {
          "data_path": "$",
          "properties": {
            "title": "$.order_id",
            "subtitle": "$.total_price"
          }
        }
      }
    }
    ```

    首先，使用 ID **placeOrder** 引用 API 操作。 然后，提供 Copilot 用于将此函数与用户的提示匹配的说明。 接下来，指示 Copilot 如何返回数据。 下面是 API 下订单后返回的数据：

    ```json
    {
      "order_id": 6532,
      "status": "confirmed",
      "total_price": 21.97
    }
    ```

    由于要显示的数据直接位于响应对象的根目录中，因此需要将 **data_path** 设置为 **$**，这表示 JSON 对象的顶部节点。 定义标题，以显示订单的编号及其价格副标题。

1. 完整文件如下所示：

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          "description": "Places an order and returns the order details",
          "capabilities": {
            "response_semantics": {
              "data_path": "$",
              "properties": {
                "title": "$.order_id",
                "subtitle": "$.total_price"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. 保存所做更改。

## 任务 3 - 定义运行时

定义 Copilot 调用的函数后，下一步是指示它应如何调用这些函数。 在插件定义的**运行时**部分中执行此操作。

在 Visual Studio Code 中：

1. 打开 **appPackage/ai-plugin.json** 文件。
1. 在运行时数组中，添加以下代码：

    ```json
    {
      "type": "OpenApi",
      "auth": {
        "type": "None"
      },
      "spec": {
        "url": "apiSpecificationFile/ristorante.yml"
      },
      "run_for_functions": [
        "getDishes",
        "placeOrder"
      ]
    }
    ```

    首先，指示 Copilot 你要向其提供有关要调用的 API 的 OpenAPI 信息 (**type: OpenApi**)，并且这些信息是匿名的 (**auth.type: None**)。 接下来，在“**规范**”部分中，指定位于项目中的 API 规范的相对路径。 最后，在 **run_for_functions** 属性中，列出属于此 API 的所有函数。

1. 完整文件如下所示：

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          ...trimmed for brevity
        }
      ],
      "runtimes": [
        {
          "type": "OpenApi",
          "auth": {
            "type": "None"
          },
          "spec": {
            "url": "apiSpecificationFile/ristorante.yml"
          },
          "run_for_functions": [
            "getDishes",
            "placeOrder"
          ]
        }
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. 保存所做更改。

