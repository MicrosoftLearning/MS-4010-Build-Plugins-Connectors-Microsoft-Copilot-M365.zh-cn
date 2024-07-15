---
lab:
  title: 练习 4 - 扩展和优化消息扩展以与适用于 Microsoft 365 的 Copilot 一起使用
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 练习 4 - 扩展和优化消息扩展以与适用于 Microsoft 365 的 Copilot 一起使用

在本练习中，你将扩展和优化消息扩展，以便与适用于 Microsoft 365 的 Copilot 一起使用。 添加名为“目标受众”的新参数，并更新消息扩展逻辑以处理多个参数。 最后，运行并调试消息扩展，并在 Microsoft Teams 的 Copilot 中对其进行测试。

## 任务 1 - 更新应用部件清单

在应用部件清单中简洁准确地列出说明对确保 Copilot 知道何时以及如何调用插件至关重要。 更新应用部件清单中的应用、命令和参数说明。

打开 Visual Studio：

1. 在 **appPackage** 文件夹中，打开名为 **manifest.json** 的文件
1. 更新描述**** 对象

    ```json
    {
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
        },
    }
    ```

    由于我们将向命令添加另一个参数，因此还要更新命令说明以包含新参数。

1. 在命令**** 数组中，更新命令的说明****

    ```json
    {
        "commands": [
            {
                "id": "Search",
                "type": "query",
                "title": "Products",
                "description": "Find products by name or by target audience",
                "initialRun": true,
                "fetchTask": false,
                "context": [...],
                "parameters": [...]
            }
        ]
    }
    ```

    现在，添加 Copilot 可以使用的新参数。 这个新参数可帮助用户使用 Copilot 查找面向不同受众（如个人和企业）的产品。

1. 在参数**** 数组中，在 ProductName**** 参数后添加 TargetAudience**** 参数。

    ```json
    {    
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
    }
    ```

1. 保存所做的更改

TargetAudience**** 参数描述其内容，并说明该参数应接受使用者**** 或企业**** 为允许值。

## 任务 2 - 更新消息扩展逻辑

要支持新参数并支持复杂的提示，请更新机器人活动处理程序中的 OnTeamsMessagingExtensionQueryAsync 方法以处理多个参数。

假设用户输入提示“查找面向名称为 Mark8 的个人的 Contoso 产品”。 给定参数说明，“面向个人”将转换为使用者****，并作为 TargetAudience** **参数的值传递。 “Mark8”作为 ProductName**** 参数的值传递。

在 Visual Studio 中继续操作：

1. 在“搜索”**** 文件夹中，打开名为“SearchApp.cs”**** 的文件
1. 在“OnTeamsMessagingExtensionQueryAsync”**** 方法中，查找以下代码块：

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters); 
    ```

1. 更新代码块以获取 TargetAudience**** 参数的值，并创建在查询 SharePoint Online 列表时要使用的筛选器查询。

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var retailCategory = GetQueryData(query.Parameters, "TargetAudience");
    
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var retailCategoryFilter = !string.IsNullOrEmpty(retailCategory) ? $"fields/RetailCategory eq '{retailCategory}'" : string.Empty;
    var filters = new List<string> { nameFilter };
    filters.RemoveAll(f => string.IsNullOrEmpty(f));
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. 保存所做的更改

## 任务 3 - 预配资源

运行“准备 Teams 应用依赖项”进程以预配资源。

在 Visual Studio 中继续操作：

1. 在**解决方案资源管理器**中，右键单击 **MsgExtProductSupport** 项目
1. 展开 **Teams 工具包**菜单，选择 **“准备 Teams 应用依赖项”**
1. 在**Microsoft 365 帐户**对话框中，选择“继续”****
1. 在**预配**对话框中，选择“预配”****
1. 在 **Teams 工具包警告**对话框中，选择“预配”****
1. 在“Teams 工具包信息”**** 对话框中， 关闭**** 提示

## 任务 4 - 运行和调试

现在，启动 Web 服务并在适用于 Microsoft 365 的 Copilot 中测试消息扩展。

1. 按 **F5** 启动调试会话，并打开导航到 Microsoft Teams Web 客户端的新浏览器窗口。
1. 输入 Microsoft 365 帐户凭据并继续使用 Microsoft Teams。
1. 在应用安装对话框中，选择**添加**
1. 从 Microsoft Teams 打开 Copilot**** 应用
1. 在撰写消息区域中，打开“插件”**** 浮出控件
1. 在插件列表中，切换“Contoso 产品”**** 插件以启用它
1. 输入“查找面向个人的 Contoso 产品”**** 作为消息并发送
1. 在 Copilot 响应中，返回登录按钮，选择“登录”**** 按钮进行身份验证
1. 身份验证成功后，**输入“查找面向个人的 Contoso 产品”** 作为消息发送
1. 在 Copilot 响应中，将显示插件响应中返回的数据，以及响应中引用的插件
1. 要查看与结果相关的自适应卡片，请将鼠标悬停在 Copilot 响应中的引用上

关闭浏览器，以停止调试会话。

[继续学习实验室摘要...](./6-summary.md)