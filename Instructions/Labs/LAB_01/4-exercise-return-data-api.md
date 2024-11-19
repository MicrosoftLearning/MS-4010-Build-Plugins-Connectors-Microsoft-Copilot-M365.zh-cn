---
lab:
  title: 练习 3 - 从 Microsoft Entra 保护的 API 返回产品数据
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 练习 3 - 从 Microsoft Entra 保护的 API 返回产品数据

在本练习中，你将更新消息扩展插件以从自定义 API 检索数据。 基于用户查询从自定义 API 获取数据，并将搜索结果中的数据返回给用户。

![Microsoft Teams 中基于搜索的消息扩展返回的搜索结果屏幕截图。](../media/3-search-results-api.png)

### 练习用时

  - **估计完成时间：** 50 分钟

## 任务 1 - 安装和配置 Dev Proxy

在本练习中，你将使用 Dev Proxy，这是一款可以模拟 API 的命令行工具。 当你想要测试应用而无需创建实际 API 时，它非常有用。

要完成本练习，你需要安装[最新版本的 Dev Proxy](/microsoft-cloud/dev/dev-proxy/get-started)，并下载本模块的 Dev Proxy 预设。

预设使用内存中数据存储模拟 CRUD（创建、读取、更新、删除）API，该数据存储受 Microsoft Entra 保护。 这意味着你可以测试应用，如同它调用需要身份验证的实际 API 一样。

1. 要安装 Dev Proxy，请**以管理员身份打开新的命令提示符窗口**：

    ```bash
    winget install Microsoft.DevProxy --silent
    ```

1. 如需下载预测，接下来请运行以下命令：

    ```bash
    devproxy preset get learn-copilot-me-plugin
    ```

1. 将命令提示符窗口保持打开状态，以便以后再次使用。

## 任务 4 - 获取用户查询值

创建一个按参数名称获取用户查询值的方法。

在 Visual Studio 和 **ProductsPlugin** 项目中：

1. 在 **Helpers** 文件夹中，创建名为 **MessageExtensionHelpers.cs** 的新文件。

1. 用以下内容替换文件中的代码：

   ```csharp
   using Microsoft.Bot.Schema.Teams;
   internal class MessageExtensionHelpers
   {
       internal static string GetQueryParameterValueByName(IList<MessagingExtensionParameter> parameters, string name) => parameters.FirstOrDefault(p => p.Name == name)?.Value as string ?? string.Empty;
   }
   ```

1. 保存所做更改。

接下来，更新 SearchApp 类中的 **OnTeamsMessagingExtensionQueryAsync** 方法以使用新的帮助程序方法。

1. 在 **Search** 文件夹中，打开 **SearchApp.cs**。

1. 在 **OnTeamsMessagingExtensionQueryAsync** 方法中，替换以下代码：

   ```csharp
   var text = query?.Parameters?[0]?.Value as string ?? string.Empty;
   ```

   替换为

   ```csharp
   var text = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
   ```

1. 将光标移动到 **text** 变量，使用 `Ctrl + R` 和 `Ctrl + R`，并将该变量重命名为 **name**。

1. 按 **Enter** 以在 3 个文件中重命名该变量。

1. 保存所做更改。

**OnTeamsMessagingExtensionQueryAsync** 方法现在应如下所示：

```csharp
protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
{
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
    var template = new AdaptiveCardTemplate(card);
    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "result",
            AttachmentLayout = "list",
            Attachments = [
                new MessagingExtensionAttachment
                    {
                        ContentType = AdaptiveCard.ContentType,
                        Content = JsonConvert.DeserializeObject(template.Expand(new { title = name })),
                        Preview = new ThumbnailCard { Title = name }.ToAttachment()
                    }
            ]
        }
    };
}
```

## 任务 3 - 从自定义 API 获取数据

要从自定义 API 获取数据，你需要在请求的授权标头中发送访问令牌，并将响应反序列化为表示产品数据的模型。

首先，创建一个模型，表示从自定义 API 返回的产品数据。

在 Visual Studio 和 **ProductsPlugin** 项目中：

1. 创建名为 **Models** 的文件夹。

1. 在 **Models** 文件夹中，新建名为 **Product.cs** 的文件。

1. 在该文件中，将现有代码替换为以下内容：

   ```csharp
   using System.Text.Json.Serialization;
   internal class Product
   {
       [JsonPropertyName("productId")]
       public int Id { get; set; }
       [JsonPropertyName("imageUrl")]
       public string ImageUrl { get; set; }
       [JsonPropertyName("name")]
       public string Name { get; set; }
       [JsonPropertyName("category")]
       public string Category { get; set; }
       [JsonPropertyName("callVolume")]
       public int CallVolume { get; set; }
       [JsonPropertyName("releaseDate")]
       public string ReleaseDate { get; set; }
   }
   ```

1. 保存所做更改。

接下来，创建一个服务类，用于从自定义 API 检索产品数据。

在 Visual Studio 和 **ProductsPlugin** 项目中：

1. 创建名为 **Services** 的文件夹。

1. 在 **Services** 文件夹中，新建名为 **ProductService.cs** 的文件。

1. 在该文件中，将现有代码替换为以下内容：

    ```csharp
    using System.Net.Http.Headers;
    internal class ProductsService
    {
        private readonly HttpClient _httpClient;
        private readonly string _baseUri = "https://api.contoso.com/v1/";
        internal ProductsService(string token)
        {
            _httpClient = new HttpClient();
            _httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
        }
        internal async Task<Product[]> GetProductsByNameAsync(string name)
        {
            var response = await _httpClient.GetAsync($"{_baseUri}products?name={name}");
            response.EnsureSuccessStatusCode();
            var jsonString = await response.Content.ReadAsStringAsync();
            return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
        }
    }
    ```

1. 保存所做更改。

**ProductsService** 类包含从自定义 API 获取产品数据的方法。 类构造函数采用访问令牌作为参数，并使用授权标头中的访问令牌设置 **HttpClient** 实例。

接下来，更新 **OnTeamsMessagingExtensionQueryAsync** 方法，以使用 **ProductsService** 类从自定义 API 获取产品数据。

1. 在 **Search** 文件夹中，打开 **SearchApp.cs**。

1. 在 **OnTeamsMessagingExtensionQueryAsync** 方法中，在 **name** 变量声明后添加以下代码，以从自定义 API 获取产品数据：

   ```csharp
   var productService = new ProductsService(tokenResponse.Token);
   var products = await productService.GetProductsByNameAsync(name);
   ```

1. 保存所做更改。

## 任务 4 - 创建搜索结果

拥有产品数据后，即可将这些数据包含在返回给用户的搜索结果中。

首先，更新现有的自适应卡片模板以显示产品信息。

在 Visual Studio 和 **ProductsPlugin** 项目中继续操作：

1. 在 **Resources** 文件夹中，将 **card.json** 重命名为 **Product.json**。

1. 在 **Resources** 文件夹中，新建名为 **Product.data.json** 的文件。 此文件包含 Visual Studio 用于生成自适应卡片模板预览的示例数据。

1. 将以下 JSON 添加到该文件：

    ```json
    {
      "callVolume": 36,
      "category": "Enterprise",
      "imageUrl": "https://raw.githubusercontent.com/SharePoint/sp-dev-provisioning-templates/master/tenant/productsupport/source/Product%20Imagery/Contoso4.png",
      "name": "Contoso Quad",
      "productId": 1,
      "releaseDate": "2019-02-09"
    }
    ```

1. 保存所做更改。

1. 在 **Resources** 文件夹中，打开 **Product.json**。

1. 将文件内容替换为以下 JSON：

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.5",
      "body": [
        {
          "type": "TextBlock",
          "text": "${name}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${category}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${imageUrl}",
              "altText": "${name}"
            }
          ],
          "minHeight": "350px",
          "verticalContentAlignment": "Center",
          "horizontalAlignment": "Center"
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "Call Volume",
              "value": "${formatNumber(callVolume,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(releaseDate,'dd/MM/yyyy')}"
            }
          ]
        }
      ]
    }
    ```

1. 保存所做更改。

自适应卡片模板使用绑定表达式来显示产品信息。 **\$\{name\}**、**\$\{category\}**、**\$\{imageUrl\}**、**\$\{callVolume\}** 和 **\$\{releaseDate\}** 表达式将替换为产品数据中的相应值。 **formatNumber** 和 **formatDateTime** 模板函数用于将 **callVolume** 和 **releaseDate** 值分别格式化为数字和日期。

花点时间浏览 Visual Studio 中的自适应卡片预览。 预览显示了将产品数据绑定到模板时自适应卡片模板的外观。 它使用 **Product.data.json** 文件中的示例数据来生成预览。

接下来，更新应用部件清单中的 **validDomains** 属性以包含 **raw.githubusercontent.com** 域，以便在 Microsoft Teams 中显示自适应卡片模板中的图像。

在 **TeamsApp** 项目中：

1. 在 **appPackage** 文件夹中，打开 **manifest.json**。

1. 在该文件中，将 GitHub 域添加到 **validDomains** 属性：

    ```json
      "validDomains": [
        "token.botframework.com",
        "raw.githubusercontent.com",
        "${{BOT_DOMAIN}}"
      ],
    ```

1. 保存所做更改。

接下来，更新 **OnTeamsMessagingExtensionQueryAsync** 方法，以创建包含产品信息的附件列表。

在 **ProductsPlugin** 项目中：

1. 在 **Search** 文件夹中，打开 **SearchApp.cs**。

1. 将 **card.json** 更新为 **Product.json**，以反映文件名中的更改。 替换第 34 行的以下代码：

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
   ```

   替换为

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "Product.json"), cancellationToken);
   ```

1. 在 **template** 变量声明后添加以下代码以创建附件列表：

   ```csharp
    var attachments = products.Select(product =>
    {
        var content = template.Expand(product);
        return new MessagingExtensionAttachment
        {
            ContentType = AdaptiveCard.ContentType,
            Content = JsonConvert.DeserializeObject(content),
            Preview = new ThumbnailCard
            {
                Title = product.Name,
                Subtitle = product.Category,
                Images = [new() { Url = product.ImageUrl }]
            }.ToAttachment()
        };
    }).ToList();
   ```

1. 更新 **return** 语句以包含 **attachments** 变量：

   ```csharp
   return new MessagingExtensionResponse
   {
       ComposeExtension = new MessagingExtensionResult
       {
           Type = "result",
           AttachmentLayout = "list",
           Attachments = attachments
       }
   };
   ```

1. 保存更改

## 任务 5 - 创建和更新资源

现在一切就绪后，运行**准备 Teams 应用依赖项**进程以新建资源并更新现有资源。

在 Visual Studio 中继续操作：

1. 在**解决方案资源管理器**中，右键单击 **TeamsApp** 项目。

1. 展开“**Teams 工具包**”菜单，选择“**准备 Teams 应用依赖项**”。

1. 在“**Microsoft 365 帐户**”对话框中，选择“**继续**”。

1. 在“**预配**”对话框中，选择“**预配**”。

1. 在“**Teams 工具包警告**”对话框中，选择“**预配**”。

1. 在“**Teams 工具包信息**”对话框中，选择交叉图标以关闭对话框。

## 任务 6 - 运行和调试

预配资源后，启动调试会话以测试消息扩展。

首先，启动 Dev Proxy 来模拟自定义 API。

1. 在仍然打开的**命令提示符窗口**中，运行以下命令以启动 Dev Proxy：

   ```bash
   devproxy --config-file "~appFolder/presets/learn-copilot-me-plugin/products-api-config.json"
   ```

1. 如果出现提示，请接受任何证书警告。

> [!NOTE]
> 当 Dev Proxy 运行时，它充当系统级代理。

接下来，在 Visual Studio 中启动调试会话：

1. 要启动新的调试会话，请按 <kbd>F5</kbd> 或从工具栏中选择“**开始**”。

1. 等待浏览器窗口打开，应用安装对话框将显示在 Microsoft Teams Web 客户端中。 出现提示时，输入 Microsoft 365 帐户凭据。

1. 在应用安装对话框中，选择“**添加**”。

1. 打开新的或现有的 Microsoft Teams 聊天。

1. 在消息撰写区域中，键入 **/apps** 以打开应用选取器。

1. 在应用列表中，选择“**Contoso 产品**”，以打开消息扩展。

1. 在文本框中输入 **Mark8**。 可能需要多次输入搜索查询。

1. 等待搜索完成并显示结果。

    ![Microsoft Teams 中基于搜索的消息扩展返回的搜索结果屏幕截图。](../media/3-search-results-api.png)

1. 在结果列表中，选择一条搜索结果，以将卡片嵌入到撰写消息框中

返回到 Visual Studio，然后从工具栏中选择“**停止**”，或按 <kbd>Shift</kbd> + <kbd>F5</kbd> 停止调试会话。 同时，使用 <kbd>Ctrl</kbd> + <kbd>C</kbd> 关闭 Dev Proxy。

[继续进行下一个练习...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)