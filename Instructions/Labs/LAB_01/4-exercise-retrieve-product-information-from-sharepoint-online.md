---
lab:
  title: 练习 3 - 从 SharePoint Online 检索产品信息
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 练习 3 - 从 SharePoint Online 检索产品信息

在本练习中，你将预配和配置 SharePoint Online 站点以将产品信息存储为列表中的项。 使用 Microsoft Graph SDK 更新消息扩展代码以从 SharePoint Online 检索列表项，并在搜索结果中返回列表项数据。 最后，运行并调试消息扩展，并在 Microsoft Teams 中对其进行测试。

:::image type="content" source="../media/4-search-results-sharepoint-online.png" alt-text="Microsoft Teams 中基于搜索的消息扩展返回的搜索结果的屏幕截图。搜索结果是从 SharePoint Online 返回的。每个搜索结果显示产品名称、类别和产品图像。" lightbox="../media/4-search-results-sharepoint-online.png":::

## 任务 1 - 预配和配置产品营销 SharePoint 站点

首先，使用 SharePoint Look Book 服务创建 SharePoint Online 站点。

在 Web 浏览器中：

1. 转到 **SharePoint Look Book**，网址为：[https://lookbook.microsoft.com](https://lookbook.microsoft.com)
1. 在顶部导航上，展开“查看设计”****
1. 在“查看设计”**** 菜单中，展开“团队”**** 并选择“产品支持”****
1. 选择“添加租户”****
1. 出现提示时，请登录到租户。
1. 在权限许可屏幕上，查看所需的权限，然后选择“接受”**** 以返回到 SharePoint Look Book 服务。
1. 在窗体中，接受默认值并选择“预配”****

站点预配完成后，将向你的电子邮件地址发送电子邮件通知你。 此过程可能需要几分钟才能完成。

:::image type="content" source="../media/1-sharepoint-online-product-support-site.png" alt-text="产品支持 SharePoint Online 团队网站主页的屏幕截图。会显示最近发布的产品列表。" lightbox="../media/1-sharepoint-online-product-support-site.png":::

要在使用 Microsoft Graph API 查询列表时启用对“标题”和“零售类别”列的筛选，请在列表中创建索引。

在 Web 浏览器中继续操作：

1. 转到“产品支持”**** 站点，网址为：**<https://tenant.sharepoint.com/sites/productmarketing>**，将“租户”**** 替换为 SharePoint Online 实例的名称
1. 在“Microsoft 365 套件栏”**** 上，选择“设置齿轮”**** 以打开“设置”侧面板。
1. 在 SharePoint **** 标题下，选择“站点内容”****
1. 在“列表和库”列表中，将鼠标悬停在“产品”**** 列表上，选择“垂直三点”**** 图标以展开“显示操作”**** 菜单，然后选择“设置”****。
1. 在“列”**** 部分的列列表下，选择“索引列”****
1. 选择“新建索引”****

## 任务 2 - 添加 SharePoint 主机名和站点 URL 环境变量

接下来，我们将 SharePoint Online 实例的主机名和产品支持站点 URL 集中为环境变量。 然后，将值公开为环境变量，以在运行时使用，并更新代码以读取值。

打开 Visual Studio：

1. 在 **env** 文件夹中，打开名为 **.env.local** 的文件
1. 在文件中，添加 SPO_HOSTNAME**** 和 SPO_SITE_URL**** 环境变量，将“租户”**** 替换为 SharePoint Online 实例的名称：

    ```text
    SPO_HOSTNAME=tenant.sharepoint.com
    SPO_SITE_URL=sites/productmarketing
    ```

1. 保存所做的更改

接下来，更新操作以将环境变量写入应用设置文件。

1. 在项目根文件夹中，打开名为 **teamsapp.local.yml** 的文件
1. 查找使用 file/createOrUpdateJsonFile**** 操作的步骤，该操作面向 ./appsettings.Development.json**** 文件
1. 在文件中，更新“内容”**** 数组，添加 SPO_HOSTNAME**** 和 SPO_SITE_URL**** 变量：

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
            SPO_HOSTNAME: ${{SPO_HOSTNAME}}
            SPO_SITE_URL: ${{SPO_SITE_URL}}
    ```

1. 保存所做的更改

现在，更新 ConfigOptions 类以包含新的环境变量

1. 在项目根文件夹中，打开 Config.cs
1. 在 ConfigOptions 类中，添加名称为 SPO_HOSTNAME 和 SPO_SITE_URL 的新字符串属性

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
      public string SPO_HOSTNAME { get; set; }
      public string SPO_SITE_URL { get; set; }
    }
    ```

1. 保存所做的更改

接下来，使用两个环境变量更新应用配置。

1. 在项目根文件夹中，打开 Program.cs
1. 添加新行以将 SPO_HOSTNAME 和 SPO_SITE_URL 环境变量添加为应用配置设置。

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    builder.Configuration["SPO_HOSTNAME"] = config.SPO_HOSTNAME;
    builder.Configuration["SPO_SITE_URL"] = config.SPO_SITE_URL;
    ```

1. 保存所做的更改

最后一步是更新机器人活动处理程序，以从应用配置中读取值，并将值存储在只读属性中。

1. 在“搜索”文件夹中，打开名为 SearchApp.cs 的文件
1. 在 SearchApp 类中，创建名称为 spoHostname 和 spoSiteUrl 的只读字符串属性

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
      private readonly string spoHostname;
      private readonly string spoSiteUrl;
    }
    ```

1. 更新构造函数以使用注入的应用配置设置属性值：

    ```csharp
    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    } 
    ```

1. 保存所做更改。

## 任务 3 - 更新搜索命令

当消息扩展返回产品信息时，请更新搜索命令标题和说明，同时更新参数名称和说明。

在 Visual Studio 中继续操作：

1. 在 **appPackage** 文件夹中，打开名为 **manifest.json** 的文件
1. 在 **composeExtensions** 数组中，使用以下内容更新命令对象：

    ```json
    "composeExtensions": [
      {
        "botId": "${{BOT_ID}}",
        "commands": [
          {
            "id": "Search",
            "type": "query",
            "title": "Products",
            "description": "Find products by name",
            "initialRun": false,
            "fetchTask": false,
            "context": [
              "commandBox",
              "compose",
              "message"
            ],
            "parameters": [
              {
                "name": "ProductName",
                "title": "Product name",
                "description": "The name of the product as a keyword",
                "inputType": "text"
              }
            ]
          }
        ]
      }
    ],
    ```

1. 保存所做的更改

## 任务 4 - 获取用户查询值

执行 OnTeamsMessagingExtensionQueryAsync 方法时，我们要做的第一件事就是了解用户在搜索框中输入的内容。

首先，让我们删除现有代码。

在 Visual Studio 中继续操作：

1. 在**搜索**文件夹中，打开名为 **SearchApp.cs** 的文件
1. 在 **OnTeamsMessagingExtensionQueryAsync** 方法中，删除检查访问令牌的 **if** 语句**之后**的所有代码
1. 在 **SearchApp** 类中，删除 **FindPackages** 方法和 **_adaptiveCardFilePath** 属性。

删除现有代码后，**SearchApp** 类应如以下代码片段所示：

```csharp
public class SearchApp : TeamsActivityHandler
{
    private readonly string connectionName;
    private readonly string spoHostname;
    private readonly string spoSiteUrl;

    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    }

    protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
    {
        var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
        var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);

        if (!HasToken(tokenResponse))
        {
            return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
        }
    }
}
```

现在，让我们编写代码以获取 **ProductName** 参数的值。

1. 在 **OnTeamsMessagingExtensionQueryAsync** 方法中，添加代码以从 **MessagingExtensionQuery** 对象的 **Parameters** 数组中检索 **ProductName** 参数的值

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    ```

1. 在 **SearchApp** 类中，实现 **GetQueryData** 方法

    ```csharp
    private static string GetQueryData(IList<MessagingExtensionParameter> parameters, string key)
    {
      if (parameters.Any() != true)
      {
        return string.Empty;
      }
    
      var foundPair = parameters.FirstOrDefault(pair => pair.Name == key);
      return foundPair?.Value?.ToString() ?? string.Empty;
    }
    ```

1. 保存所做的更改

**GetQueryData** 方法用于从 **MessagingExtensionParameter** 对象列表中检索与特定键关联的值。 它提供了一种从 **MessagingExtensionQuery** 对象中的参数数组提取数据的便捷方法。

## 任务 5 - 创建 SharePoint 列表查询 OData 筛选器

现在，我们已获得用户传入的值，请使用此值创建 OData 查询筛选器。 筛选器用于按“标题”列查询 SharePoint Online 列表，其中包含产品名称。

在 Visual Studio 中继续操作：

1. 在 **OnTeamsMessagingExtensionQueryAsync** 方法中，添加代码以创建 **filterQuery** 变量。

    ```csharp
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. 保存所做的更改

该代码基于**名称**参数构造筛选器查询。 如果提供了名称参数，则会创建一个筛选器表达式，搜索以提供的名称开头的**标题**字段的项目。 如果未提供名称参数，则会将空字符串指定为筛选器查询。 生成的筛选器查询稍后在代码中用于从 SharePoint 网站检索筛选的项目。

## 任务 6 - 安装和配置 Microsoft Graph SDK

若要对 Microsoft Graph 执行经过身份验证的请求，请使用 **Microsoft Graph SDK**。

从 NuGet 安装 Microsoft Graph SDK 包，然后创建一个 **TokenProvider** 类，该类允许你使用从令牌服务获取的访问令牌，然后初始化新的 **GraphServiceClient**。

在 Visual Studio 中继续操作：

1. 在解决方案资源管理器中，右键单击 **MsgExtProductSupport** 项目
1. 选择“管理 NuGet 包…”****
1. 选择**浏览**选项卡，搜索 **Microsoft.Graph**
1. 在结果列表中，选择**Microsoft.Graph**
1. 在 **“版本”** 下拉列表中，选择 **“5.42.0”**
1. 选择“安装”
1. 在**许可证接受**对话框中，选择“我接受“**** 以安装 SDK

安装包后，为 Microsoft Graph SDK 创建令牌提供程序。

1. 在 **搜索** 文件夹中，创建名为 **TokenProvider.cs 的新文件**
1. 在  文件中添加以下代码：

    ```csharp
    using Microsoft.Kiota.Abstractions.Authentication;
    
    namespace MsgExtProductSupport.Search
    {
       public class TokenProvider : IAccessTokenProvider
        {
            public string Token { get; set; }
            public AllowedHostsValidator AllowedHostsValidator => throw new NotImplementedException();
    
            public Task<string> GetAuthorizationTokenAsync(Uri uri, Dictionary<string, object>? additionalAuthenticationContext = null, CancellationToken cancellationToken = default)
            {
                return Task.FromResult(Token);
            }
        }
    }
    ```

1. 保存所做的更改

现在创建一个方法来创建新的 **GraphServiceClient** 实例。

1. 在**搜索**文件夹中，打开名为 **SearchApp.cs** 的文件
1. 在文件中，导入所需的命名空间：

    ```csharp
    using Microsoft.Graph;
    using Microsoft.Kiota.Abstractions.Authentication;
    ```

1. 在 **OnTeamsMessagingExtensionQueryAsync** 方法中，添加代码以创建新的 Graph 客户端，用于将请求发送到 Microsoft Graph

    ```csharp
    var graphClient = CreateGraphClient(tokenResponse);
    ```

1. 在 **SearchApp** 类中，实现 **CreateGraphClient** 方法

    ```csharp
    private static GraphServiceClient CreateGraphClient(TokenResponse tokenResponse)
    {
      TokenProvider provider = new() { Token = tokenResponse.Token };
      var authenticationProvider = new BaseBearerTokenAuthenticationProvider(provider);
      var graphClient = new GraphServiceClient(authenticationProvider);
      return graphClient;
    }
    ```

1. 保存所做的更改

此代码设置身份验证提供程序，并创建一个客户端对象，该对象可用于使用提供的访问令牌与 Microsoft Graph API 进行交互。

## 任务 7 - 查询产品列表

若要查询产品列表中的项及更高版本，请创建搜索结果，请使用 GraphServiceClient 发送请求，以便从 SharePoint Online 获取产品数据。

在 Visual Studio 中继续操作：

在 **OnTeamsMessagingExtensionQueryAsync** 方法中，添加代码以获取 SharePoint 数据：

  ```csharp
  var site = await GetSharePointSite(graphClient, spoHostname, spoSiteUrl, cancellationToken);
  var drive = await GetSharePointDrive(graphClient, site.SharepointIds.SiteId, "Product Imagery", cancellationToken);
  var items = await GetProducts(graphClient, site.SharepointIds.SiteId, filterQuery, cancellationToken);
  ```

此代码将：

- **获取产品营销站点**，网站对象包含 SharePoint 站点的 ID，该 ID 用于返回和查询网站中的对象
- **获取产品图像驱动器**，该驱动器表示包含产品图像的文档库。 稍后使用驱动器获取在搜索结果中显示的产品图像
- **获取产品**，根据用户查询使用该产品查询产品列表

在 **SearchApp** 类中实现三种方法。

- 实现 **GetSharePointSite** 方法

    ```csharp
    private static async Task<Site> GetSharePointSite(GraphServiceClient graphClient, string hostName, string siteUrl, CancellationToken cancellationToken)
    {
        return await graphClient.Sites[$"{hostName}:/{siteUrl}"].GetAsync(r => r.QueryParameters.Select = new string[] { "sharePointIds" }, cancellationToken);
    }
    ```

此方法使用 GraphServiceClient 向 Microsoft Graph 发送请求，以使用路径返回站点对象。 路径是通过组合 SharePoint Online 主机名和网站 URL 创建的。 由于只需要 sharePointIds 属性值，选择查询参数配置为仅在响应中返回此属性。

- 实现 **GetSharePointDrive** 方法

    ```csharp
    private static async Task<Drive> GetSharePointDrive(GraphServiceClient graphClient, string siteId, string name, CancellationToken cancellationToken)
    {
        var drives = await graphClient.Sites[siteId].Drives.GetAsync(r => r.QueryParameters.Select = new string[] { "id", "name" }, cancellationToken);
        var drive = drives.Value.Find(d => d.Name == name);
        return drive;
    }
    ```

此方法使用 GraphServiceClient 和网站 ID 从网站返回文档库的集合，并返回每个文档库的 ID 和名称属性。 然后筛选库的集合以返回驱动器，该驱动器的名称与名称方法参数的名称相同。

- 实现 **GetProducts** 方法

    ```csharp
    private static async Task<SiteCollectionResponse> GetProducts(GraphServiceClient graphClient, string siteId, string filterQuery, CancellationToken cancellationToken)
    {
        var fields = new string[]
        {
            "fields/Id",
            "fields/Title",
            "fields/RetailCategory",
            "fields/PhotoSubmission",
            "fields/CustomerRating",
            "fields/ReleaseDate"
        };
    
        var request = graphClient.Sites.WithUrl($"https://graph.microsoft.com/v1.0/sites/{siteId}/lists/Products/items?expand={string.Join(",", fields)}&$filter={filterQuery}");
        return await request.GetAsync(null, cancellationToken);
    }
    ```

此方法使用 GraphServiceClient 通过传入筛选器查询从“产品”列表中返回筛选的列表项，并返回在字段数组中定义的列表项数据。

## 任务 8 - 创建搜索结果

从 SharePoint 获取产品后，将创建搜索结果，这些搜索结果将返回给用户。

创建搜索结果包括循环访问项数组，为每个项创建 MessagingExtensionAttachment，其中包含预览卡和内容卡。

循环访问项之前，请创建可在循环中使用的自适应卡片模板。

在 Visual Studio 中继续操作：

1. 在“资源”**** 文件夹中，新建名为 Product.json**** 的文件

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.6",
      "body": [
        {
          "type": "TextBlock",
          "text": "${Product.Title}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${Product.RetailCategory}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${ProductImage}",
              "altText": "${Product.Title}"
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
              "value": "${formatNumber(Product.CustomerRating,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(Product.ReleaseDate,'dd/MM/yyyy')}"
            }
          ]
        },
        {
          "type": "ActionSet",
          "actions": [
            {
              "type": "Action.OpenUrl",
              "title": "View",
              "url": "https://${SPOHostname}/${SPOSiteUrl}/Lists/Products/DispForm.aspx?ID=${Product.Id}"
            }
          ]
        }
      ]
    }
    ```

该模板使用数据绑定表达式，这些表达式在呈现自适应卡片时会替换为实际值。 呈现卡片时，它包含一些产品信息、产品图像和操作按钮。 操作按钮将打开浏览器并导航到产品项的 SharePoint 列表显示窗体。

接下来，添加代码以将 JSON 文件转换为自适应卡片模板。

1. 在“搜索”**** 文件夹中，打开“SearchApp.cs”****
1. 在文件中，导入所需的命名空间：

    ```csharp
    using AdaptiveCards.Templating;
    ```

1. 在 OnTeamsMessagingExtensionQueryAsync**** 方法中，添加代码以读取 JSON 文件的内容并新建 AdaptiveCardTemplate**** 对象。

    ```csharp
    var card = File.ReadAllText(@"Resources\Product.json");
    var template = new AdaptiveCardTemplate(card);
    ```

1. 保存所做的更改

接下来，创建循环以循环访问列表项。 每次迭代都将：

- 将当前项数据反序列化为产品对象
- 获取产品图像缩略图
- 创建内容卡
- 创建预览卡
- 创建将内容卡和预览卡组合在一起的 MessagingExtensionAttachment
- 将 MessagingExtensionAttachment 添加到列表中

循环完成后，将向你呈现可返回给用户的附件列表。

1. 在“搜索”**** 文件夹中，打开“SearchApp.cs”****
1. 在 OnTeamsMessagingExtensionQueryAsync**** 方法中，添加代码以新建列表，以在以下位置存储 MessagingExtensionAttachment**** 对象

    ```csharp
    var attachments = new List<MessagingExtensionAttachment>();
    ```

1. 创建 foreach 循环以循环访问列表项。

    ```csharp
    foreach (var item in items.Value) { 
            
    }
    ```

1. 将以下代码添加到 foreach 循环中。

    ```csharp
    var product = JsonConvert.DeserializeObject<Product>(item.AdditionalData["fields"].ToString());
    product.Id = item.Id;
    
    var thumbnails = await GetThumbnails(graphClient, drive.Id, product.PhotoSubmission, cancellationToken);
    
    var resultCard = template.Expand(new
    {
      Product = product,
      ProductImage = thumbnails.Large.Url,
      SPOHostname = spoHostname,
      SPOSiteUrl = spoSiteUrl,
    });
    
    var previewcard = new ThumbnailCard
    {
      Title = product.Title,
      Subtitle = product.RetailCategory,
      Images = new List<CardImage> { new() { Url = thumbnails.Small.Url } }
    }.ToAttachment();
    
    var attachment = new MessagingExtensionAttachment
    {
      Content = JsonConvert.DeserializeObject(resultCard),
      ContentType = AdaptiveCard.ContentType,
      Preview = previewcard
    };
    
    attachments.Add(attachment);
    ```

1. 保存所做的更改

要强键入列表项数据，请创建表示产品**** 的模型。

1. 在项目根文件夹中，新建名为模型**** 的文件夹
1. 在模型**** 文件夹中，新建名为 Product.cs**** 的文件

    ```csharp
    namespace MsgExtProductSupport.Models
    {
        public class Product
        {
            public string Title { get; set; }
            public string RetailCategory { get; set; }
            public Link Specguide { get; set; }
            public string PhotoSubmission { get; set; }
            public double CustomerRating { get; set; }
            public DateTime ReleaseDate { get; set; }
            public string Id { get; set; }
            public string ContentType { get; set; }
            public DateTime Modified { get; set; }
            public DateTime Created { get; set; }
        }
    
        public class Link
        {
            public string Description { get; set; }
            public string Url { get; set; }
        }
    }
    ```

1. 保存所做的更改

接下来，实现 GetThumbnails**** 方法，从产品的 Microsoft Graph 检索缩略图图像。

1. 在**搜索**文件夹中，打开名为 **SearchApp.cs** 的文件
1. 在“SearchApp”**** 类中，创建“GetThumbnails”**** 方法

    ```csharp
    private static async Task<ThumbnailSet> GetThumbnails(GraphServiceClient graphClient, string driveId, string photoUrl, CancellationToken cancellationToken)
    {
        var fileName = photoUrl.Split('/').Last();
        var driveItem = await graphClient.Drives[driveId].Root.ItemWithPath(fileName).GetAsync(null, cancellationToken);
        var thumbnails = await graphClient.Drives[driveId].Items[driveItem.Id].Thumbnails["0"].GetAsync(r => r.QueryParameters.Select = new string[] { "small", "large" }, cancellationToken);
        return thumbnails;
    }
    ```

1. 保存所做的更改

**GetThumbnails** 方法使用 Microsoft Graph API 中的 Thumbnails 终结点返回存储在 SharePoint 中的产品图像的小型和大型缩略图。

## 任务 9 - 返回搜索结果

现在，我们有 MessagingExtensionResult 对象的集合，我们可以将其作为搜索结果返回给用户。

- 在 OnTeamsMessagingExtensionQueryAsync**** 方法中，添加代码以将搜索结果作为消息扩展响应返回。

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

## 任务 10：预配资源

运行“准备 Teams 应用依赖项”进程以预配资源。

在 Visual Studio 中继续操作：

1. 在**解决方案资源管理器**中，右键单击 **MsgExtProductSupport** 项目
1. 展开 **Teams 工具包**菜单，选择 **“准备 Teams 应用依赖项”**
1. 在**Microsoft 365 帐户**对话框中，选择“继续”****
1. 在**预配**对话框中，选择“预配”****
1. 在 **Teams 工具包警告**对话框中，选择“预配”****
1. 在“Teams 工具包信息”**** 对话框中， 关闭**** 提示

## 任务 11 - 运行和调试

现在，启动 Web 服务并在 Microsoft Teams 中测试消息扩展。

在 Visual Studio 中继续操作：

1. 按 **F5** 启动调试会话，并打开导航到 Microsoft Teams Web 客户端的新浏览器窗口。
1. 输入 Microsoft 365 帐户凭据并继续使用 Microsoft Teams。
1. 在应用安装对话框中，选择**添加**
1. 打开新的或现有的 Microsoft Teams 聊天
1. 在消息撰写区域中，选择 **...** 打开应用浮出控件
1. 在应用列表中，选择**Contoso 产品**，以打开消息扩展
1. 在文本框中输入 Mark8****。 显示两个结果：Mark8**** 和 Mark8 控制器****
1. 选择“Mark8”**** 将卡片嵌入撰写消息框中
1. **发送包含卡片的消息**
1. 在提交的卡片中，选择“视图”**** 按钮，在新选项卡的“产品”列表中查看产品的 SharePoint 列表项

关闭浏览器，以停止调试会话。

[继续下一个练习...](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)