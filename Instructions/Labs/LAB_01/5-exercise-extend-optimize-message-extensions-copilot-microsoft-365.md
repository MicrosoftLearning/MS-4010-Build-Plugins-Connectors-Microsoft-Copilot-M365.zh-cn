---
lab:
  title: 练习 4 - 扩展和优化消息扩展以与Microsoft 365 Copilot 一起使用
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 练习 4 - 扩展和优化消息扩展以与Microsoft 365 Copilot 一起使用

在本练习中，你将扩展和优化消息扩展，以便与 Microsoft 365 Copilot 一起使用。 添加名为“目标受众”的新参数，并更新消息扩展逻辑以处理多个参数。 最后，运行并调试消息扩展，并在 Microsoft Teams 的 Copilot 中对其进行测试。

![Microsoft 365 Copilot 中答案的屏幕截图，其中包含消息扩展插件返回的信息。 显示产品信息的自适应卡片。](../media/5-copilot-answer.png)

> [!NOTE]
> 本练习中唯一需要 Microsoft 365 Copilot 许可证的任务是任务 5。 无论租户是否具有 Copilot，都应完成以前的任务。

### 练习用时

  - **估计完成时间：** 40 分钟

## 任务 1 - 更新应用说明

在应用部件清单中简洁准确地列出说明对确保 Copilot 知道何时以及如何调用插件至关重要。 更新应用部件清单中的应用、命令和参数说明。

打开 Visual Studio，在 **TeamsApp** 项目中：

1. 在 **appPackage** 文件夹中，打开 **manifest.json**。

1. 更新描述**** 对象

    ```json
    "description": {
        "short": "Product look up tool.",
        "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
    },
    ```

## 任务 2 - 添加新参数

添加 Copilot 可以使用的新参数。 这个新参数可帮助用户使用 Copilot 查找面向不同受众（如个人和企业）的产品。

在 Visual Studio 和 **TeamsApp** 项目中继续操作：

1. 在 **manifest.json** 中的 **parameters** 数组，在 **ProductName** 参数后添加 **TargetAudience** 参数。

    ```json
    "parameters": [
        {
            "name": "ProductName",
            "title": "Product name",
            "description": "The name of the product as a keyword",
            "inputType": "text"
        },
        {
            "name": "TargetAudience",
            "title": "Target audience",
            "description": "Audience that the product is aimed at. Consumer products are sold to individuals. Enterprise products are sold to businesses",
            "inputType": "text"
        }
    ]
    ```

1. 保存所做更改。

TargetAudience**** 参数描述其内容，并说明该参数应接受使用者**** 或企业**** 为允许值。

接下来，更新命令说明以包含新参数。

1. 在**命令**数组中，更新**第 36 行**的命令**说明**：

    ```json
    "description": "Find products by name or by target audience.",
    ```

## 任务 3 - 更新消息扩展逻辑

要支持新参数并支持复杂的提示，请更新机器人活动处理程序中的 **OnTeamsMessagingExtensionQueryAsync** 方法以处理多个参数。

首先，更新 **ProductService** 类以基于名称和受众参数检索产品。

在 Visual Studio、在 **ProductsPlugin** 项目中继续操作：

1. 在 **Services** 文件夹中，打开 **ProductsService.cs**。

1. 在文件中，创建名为 **GetProductsByCategoryAsync** 和 **GetProductsByNameAndCategoryAsync** 的新方法：

    ```csharp
    internal async Task<Product[]> GetProductsByCategoryAsync(string category)
    {
        var response = await _httpClient.GetAsync($"{_baseUri}products?category={category}");
        response.EnsureSuccessStatusCode();
        var jsonString = await response.Content.ReadAsStringAsync();
        return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
    }
    internal async Task<Product[]> GetProductsByNameAndCategoryAsync(string name, string category)
    {
        var response = await _httpClient.GetAsync($"{_baseUri}?name={name}&category={category}");
        response.EnsureSuccessStatusCode();
        var jsonString = await response.Content.ReadAsStringAsync();
        return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
    }
    ```

1. 保存所做更改。

接下来，向 **MessageExtensionHelper** 类添加新方法，以基于名称和受众参数检索产品。

1. 在 **Helpers** 文件夹中，打开 **MessageExtensionHelper.cs**。

1. 在文件中，创建名为 **RetrieveProducts** 的新方法，该方法基于名称和受众参数检索产品：

    ```csharp
    internal static async Task<IList<Product>> RetrieveProducts(string name, string audience, ProductsService productsService)
    {
        IList<Product> products;
        if (string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByCategoryAsync(audience);
        }
        else if (!string.IsNullOrEmpty(name) && string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByNameAsync(name);
        }
        else if (!string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByNameAndCategoryAsync(name, audience);
        }
        else
        {
            products = [];
        }
        return products;
    }
    ```

1. 保存所做更改。

**RetrieveProduct** 方法基于名称和受众参数检索产品。 如果名称参数为空且受众参数不为空，则该方法会基于受众参数检索产品。 如果名称参数不为空，受众参数为空，则该方法会基于名称参数检索产品。 如果名称和受众参数都不为空，则该方法会基于这两个参数检索产品。 如果两个参数均为空，该方法将返回空列表。

接下来，更新 **SearchApp** 类以处理新参数。

1. 在 **Search** 文件夹中，打开 **SearchApp.cs**

1. 在 **OnTeamsMessagingExtensionQueryAsync** 方法中，替换从**第 30 行**开始的以下代码：

    ```csharp
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var productService = new ProductsService(tokenResponse.Token);
    var products = await productService.GetProductsByNameAsync(name);
    ```

    替换为：

    ```csharp
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var audience = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "TargetAudience");
    var productService = new ProductsService(tokenResponse.Token);
    var products = await MessageExtensionHelpers.RetrieveProducts(name, audience, productService);
    ```

1. 保存所做更改。

**OnTeamsMessagingExtensionQueryAsync** 方法现在从查询参数中检索名称和受众参数。 然后，使用 **RetrieveProducts** 方法基于名称和受众参数检索产品。

## 任务 4 - 创建和更新资源

现在一切就绪后，运行**准备 Teams 应用依赖项**进程以新建资源并更新现有资源。

在 Visual Studio 中继续操作：

1. 在**解决方案资源管理器**中，右键单击 **TeamsApp** 项目。

1. 展开“**Teams 工具包**”菜单，选择“**准备 Teams 应用依赖项**”。

1. 在“**Microsoft 365 帐户**”对话框中，选择“**继续**”。

1. 在“**预配**”对话框中，选择“**预配**”。

1. 在“**Teams 工具包警告**”对话框中，选择“**预配**”。

1. 在“**Teams 工具包信息**”对话框中，选择交叉图标以关闭对话框。

## 任务 5 - 运行和调试

预配资源后，启动调试会话以测试消息扩展。

首先，启动 **开发代理** 来模拟自定义 API。

1. 在仍然打开的**命令提示符窗口**中，运行以下命令以启动 Dev Proxy：

1. 运行以下命令以启动开发代理：

   ```bash
   devproxy --config-file "~appFolder/presets/learn-copilot-me-plugin/products-api-config.json"
   ```

1. 如果出现提示，请接受证书警告。

> [!NOTE]
> 当 Dev Proxy 运行时，它充当系统级代理。

接下来，在 Visual Studio 中启动调试会话：

1. 要启动新的调试会话，请按 <kbd>F5</kbd> 或从工具栏中选择“**开始**”。

1. 等待浏览器窗口打开，应用安装对话框将显示在 Microsoft Teams Web 客户端中。 出现提示时，输入 Microsoft 365 帐户凭据。

1. 在应用安装对话框中，选择“**添加**”。

1. 从 Microsoft Teams 打开 **Copilot** 应用。

1. 在撰写消息区域中，打开“**插件**”浮出控件。

1. 在插件列表中，切换 **Contoso 产品**插件以启用它。

    ![Microsoft Teams 中启用了 Contoso 产品插件的 Microsoft 365 Copilot 的屏幕截图。](../media/20-copilot-plugin-enabled.png)

1. 输入“**查找面向个人的 Contoso 产品**”作为消息并发送。

1. 等待 Copilot 做出响应：

    ![Microsoft Teams 中 Microsoft 365 Copilot 的屏幕截图，其中显示了处理用户请求时显示的助手消息。](../media/21-copilot-thinking.png)

1. 在 Copilot 响应中，将显示插件响应中返回的数据，以及响应中引用的插件：

    ![Microsoft 365 Copilot 中答案的屏幕截图，其中包含消息扩展插件返回的信息。 显示产品信息的自适应卡片。](../media/5-copilot-answer.png)

1. 要查看与结果相关的自适应卡片，请将鼠标悬停在 Copilot 响应中的引用上：

    ![Microsoft Teams 中 Microsoft 365 Copilot 的屏幕截图，其中显示了显示产品信息的自适应卡片。 当用户将鼠标悬停在 Copilot 响应中的引用上时，将显示卡片。](../media/22-copilot-reference.png)

返回到 Visual Studio，然后从工具栏中选择“**停止**”，或按 <kbd>Shift</kbd> + <kbd>F5</kbd> 停止调试会话。 同时，使用 <kbd>Ctrl</kbd> + <kbd>C</kbd> 关闭 Dev Proxy。

[继续学习实验室摘要...](./6-summary.md)