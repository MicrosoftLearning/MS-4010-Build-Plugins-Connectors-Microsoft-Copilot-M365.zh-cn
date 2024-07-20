---
lab:
  title: 练习 2 - 添加单一登录
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 练习 2 - 添加单一登录

在本练习中，你将更新消息扩展，以便提示用户登录并进行身份验证。 配置 Bot Microsoft Entra 应用注册和应用清单以启用单一登录。 将 Microsoft Entra 应用注册配置为使用 Microsoft Graph 进行身份验证，并更新消息扩展逻辑，以使用 Bot Framework 令牌服务获取访问令牌。 然后，运行并调试消息扩展，以在 Microsoft Teams 中进行测试。

## 任务 1 - 配置单一登录

首先，配置 Bot Microsoft Entra 应用注册。

在 Visual Studio 中：

1. 在 **infra\entra** 文件夹中，打开名为 **entra.bot.manifest.json** 的文件
1. 在文件中，更新 **identifierUris** 数组以设置应用程序 ID URI

    ```json
    "identifierUris": [
        "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    ]
    ```

1. 在文件中，更新 **oauth2Permissions** 数组以创建范围，允许 Teams 以管理员或用户身份调用 Web API：

    ```json
      "oauth2Permissions": [
        {
          "adminConsentDescription": "Allows Teams to call the app's web APIs as the current user.",
          "adminConsentDisplayName": "Teams can access app's web APIs",
          "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
          "isEnabled": true,
          "type": "User",
          "userConsentDescription": "Enable Teams to call this app's web APIs with the same rights that you have",
          "userConsentDisplayName": "Teams can access app's web APIs and make requests on your behalf",
          "value": "access_as_user"
        }
      ]
    ```

1. 在文件中，更新 **preAuthorizedApplications** 数组，将 Microsoft Teams、Microsoft Outlook 和适用于 Microsoft 365 的 Copilot 客户端添加到授权客户端列表中：

    ```json
      "preAuthorizedApplications": [
        {
          "appId": "1fec8e78-bce4-4aaf-ab1b-5451cc387264",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "5e3ce6c0-2b1f-4285-8d4b-75ee78787346",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "4765445b-32c6-49b0-83e6-1d93765276ca",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "0ec893e0-5785-4de6-99da-4ed124e5296c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "d3590ed6-52b3-4102-aeff-aad2292ab01c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "bc59ab01-8403-45c6-8796-ac3ef710b3e3",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "27922004-5251-4030-b22d-91ecd9a37ea4",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        }
      ]
    ```

1. 保存所做的更改

接下来，更新应用清单文件，以定义客户端在应用中启动单一登录流时应使用的资源。

在 Visual Studio 中继续操作：

1. 在 **appPackage** 文件夹中，打开名为 **manifest.json** 的文件
1. 在  文件中添加以下代码：

    ```json
    "webApplicationInfo": {
      "id": "${{BOT_ID}}",
      "resource": "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    }
    ```

1. 保存所做的更改

## 任务 2 - 为 Microsoft Graph 创建 Microsoft Entra 应用注册清单文件

若要使用 Microsoft Graph 进行身份验证，请创建新的应用注册清单文件。

在 Visual Studio 中继续操作：

1. 在 **infra\entra** 文件夹中，创建名为 **entra.graph.manifest.json** 的文件
2. 在  文件中添加以下代码：

    ```json
    {
      "id": "${{GRAPH_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{GRAPH_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}",
      "accessTokenAcceptedVersion": 2,
      "signInAudience": "AzureADMultipleOrgs",
      "optionalClaims": {
        "idToken": [],
        "accessToken": [
          {
            "name": "idtyp",
            "source": null,
            "essential": false,
            "additionalProperties": []
          }
        ],
        "saml2Token": []
      },
      "requiredResourceAccess": [
        {
          "resourceAppId": "Microsoft Graph",
          "resourceAccess": [
            {
              "id": "Sites.ReadWrite.All",
              "type": "Scope"
            }
          ]
        }
      ],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": [
        {
          "url": "https://token.botframework.com/.auth/web/redirect",
          "type": "Web"
        }
      ]
    }
    ```

1. 保存所做的更改

**requiredResourceAccess** 数组定义 API 权限范围，**replyUrlsWithType** 数组定义应用注册上的重定向 URI。

现在，使用操作更新项目文件以创建应用注册。

1. 在项目根文件夹中，打开 **teamsapp.local.yml**
1. 在文件中，找到使用 **arm/deploy** 操作的步骤
1. 在该步骤之前，添加以下代码：

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: GRAPH_ENTRA_APP_ID
          clientSecret: SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET
          objectId: GRAPH_ENTRA_APP_OBJECT_ID
          tenantId: GRAPH_ENTRA_APP_TENANT_ID
          authority: GRAPH_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: GRAPH_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.graph.manifest.json"
          outputFilePath : "./build/entra.graph.manifest.${{TEAMSFX_ENV}}.json"
    ```

1. 保存所做的更改

## 任务 3 - 创建 OAuth 连接设置

Azure AI 机器人服务连接设置用于管理机器人和消息扩展中的用户身份验证。

首先，将用于创建连接设置的 OAuth 连接设置的名称集中作为环境变量，然后添加代码以在运行时使用环境变量值。

在 Visual Studio 中继续操作：  

1. 在 **env** 文件夹中，打开 **.env.local**
1. 在  文件中添加以下代码：

    ```text
    CONNECTION_NAME=MicrosoftGraph
    ```

1. 在项目根文件夹中，打开名为 **teamsapp.local.yml** 的文件
1. 在文件中，找到面向 **./appsettings Development.json **文件，使用 **file/createOrUpdateJsonFile** 操作的步骤。
1. 更新内容数组，添加 **CONNECTION_NAME** 变量

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. 保存所做的更改
1. 在项目根文件夹中，打开 **Config.cs**
1. 在 **ConfigOptions** 类中，添加名称为 **CONNECTION_NAME** 的新字符串属性

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. 保存所做的更改
1. 在项目根文件夹中，打开名为 **Program.cs** 的文件
1. 在文件中，添加新行以将 **CONNECTION_NAME** 环境变量添加为应用配置设置：

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    ```

接下来，更新机器人活动处理程序以访问应用配置。

1. 在 **搜索** 文件夹中，打开名为 **SearchApp.cs** 的文件
1. 在 **SearchApp** 类中，使用名称 **connectionName** 创建只读字符串属性。

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    }
    ```

1. 创建一个构造函数，该构造函数注入应用配置并设置 connectionName 属性的值。

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    
      public SearchApp(IConfiguration configuration)
      {
        connectionName = configuration["CONNECTION_NAME"];
      }  
    }
    ```

1. 保存所做的更改

接下来，更新 Bicep 文件以预配 OAuth 连接设置。

首先，更新参数文件以传递 Microsoft Graph Microsoft Entra 应用注册的凭据和连接设置的名称。

1. 在 infra **文件夹**中，打开名为 **azure.parameters.local.json** 的文件
1. 在参数**对象**中，添加 **graphEntraAppClientId**、**graphEntraAppClientSecret** 和 **connectionName** 参数

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "resourceBaseName": {
          "value": "bot-${{RESOURCE_SUFFIX}}-${{TEAMSFX_ENV}}"
        },
        "botEntraAppClientId": {
          "value": "${{BOT_ID}}"
        },
        "botDisplayName": {
          "value": "${{APP_DISPLAY_NAME}}"
        },
        "botAppDomain": {
          "value": "${{BOT_DOMAIN}}"
        },
        "graphEntraAppClientId": {
          "value": "${{GRAPH_ENTRA_APP_ID}}"
        },
        "graphEntraAppClientSecret": {
          "value": "${{SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. 保存所做的更改

接下来，更新 Bicep 文件。

1. 在 **infra** 文件夹中，打开名为 **azure.local.bicep 的文件**
1. 在文件中，在 **botAppDomain** 参数声明后，添加 **graphEntraAppClientId**、**graphEntraAppClientSecret** 和 **connectionName** 参数声明

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. 在 **azureBotRegistration** 模块中 **，添加 graphEntraAppClientId**、**graphEntraAppClientSecret** 和 **connectionName** 参数

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        graphEntraAppClientId: graphEntraAppClientId
        graphEntraAppClientSecret: graphEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. 保存所做更改。

最后，更新机器人注册 Bicep 文件。

1. 在 **infra/botRegistration** 文件夹中，打开名为 **azurebot.bicep 的文件**
1. 在文件中，在 botAppDomain** 参数声明后**，添加 **graphEntraAppClientId**、**graphEntraAppClientSecret** 和 **connectionName** 参数声明

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. **在 botServiceM365ExtensionsChannel** 资源之后，为 Microsoft Graph 连接添加新资源

    ```bicep
    resource botServicesMicrosoftGraphConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: graphEntraAppClientId
        clientSecret: graphEntraAppClientSecret
        scopes: 'email offline_access openid profile Sites.ReadWrite.All'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${ botAadAppClientId}'
          }
        ]
      }
    }
    ```

1. 保存所做的更改

## 任务 4 - 验证用户查询

接下来，添加代码，以便在用户使用消息扩展启动搜索时对用户进行身份验证。

在 Visual Studio 中继续操作：

1. 在 **“搜索”** 文件夹中，打开名为 **SearchApp.cs** 的文件
1. 在文件中，首先从 Bot Framework SDK 添加 **“身份验证”** 命名空间。

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. 在 **OnTeamsMessagingExtensionQueryAsync** 方法的开头添加以下代码：

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    
    if (!HasToken(tokenResponse))
    {
        return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

上述代码块使用三种处理用户身份验证的方法。

- **GetToken** 使用令牌服务客户端获取当前用户的访问令牌
- **HasToken** 检查来自令牌服务的响应是否包含访问令牌
- 如果未返回令牌，则会调用 **CreateAuthResponse** 并返回响应，在用户界面中显示登录链接

现在，在 **SearchApp** 类中创建方法。

- 使用以下代码创建 **GetToken** 方法：

```csharp
private static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
{
  var magicCode = string.Empty;

  if (!string.IsNullOrEmpty(state))
  {
    if (int.TryParse(state, out var parsed)) 
    {
        magicCode = parsed.ToString();
    }
  }

  return await userTokenClient.GetUserTokenAsync(userId, connectionName, channelId, magicCode, cancellationToken);
}
```

首先检查状态参数是否不为 null 或空，代码会尝试使用 int.TryParse 方法将其分析为整数。 如果分析成功，则分析的值将作为字符串分配给 magicCode 变量。 然后，magicCode 与其他参数一起作为参数传递给 GetUserTokenAsync 方法。 GetUserTokenAsync 方法使用 magicCode 验证请求的真实性。 它确保由发起身份验证过程的同一实体请求用户令牌。

- 使用以下代码创建 HasToken 方法：

```csharp
private static bool HasToken(TokenResponse tokenResponse)
{
    return tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
}
```

通过检查响应上的令牌响应和令牌属性是否不为空或 null，可以检查是否从令牌服务获取了有效令牌。

- 使用以下代码创建 **CreateAuthResponse** 方法：

```csharp
private static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
{
    var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);

    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "auth",
            SuggestedActions = new MessagingExtensionSuggestedAction
            {
                Actions = new List<CardAction>
                {
                    new() {
                        Type = ActionTypes.OpenUrl,
                        Value = resource.SignInLink,
                        Title = "Sign In",
                    },
                },
            },
        },
    };
}
```

首先，使用 **GetSignInResourceAsync** 方法从令牌服务检索登录链接。 登录链接用于构造 **MessagingExtensionResponse** 对象。 创建新的对象，并将响应的 **ComposeExtension** 属性设置为新的 **MessagingExtensionResult** 对象。 结果的类型属性设置为 “auth”，指示结果为身份验证响应。 结果的 **SuggestedActions** 属性设置为新的 **MessagingExtensionSuggestedAction** 对象。 建议操作的 操作属性设置为包含单个 **CardAction** 对象的列表。 此 **CardAction** 对象表示用户可以执行的操作。 **CardAction** 的类型属性设置为 **ActionTypes.OpenUrl**，指示它是打开 URL 的操作。 值属性设置为从资源检索的登录链接。 标题属性设置为“登录”，该属性指定操作的标题。 最后，从方法返回构造的响应。

当用户遵循登录链接时，他们将被带到托管在外部域上的资源。 域必须包含在应用清单文件中。 将 Bot Framework 令牌服务域添加到应用清单。

在 Visual Studio 中继续操作：

1. 在 **appPackage** 文件夹中，打开 **manifest.json**
1. 在文件中，更新 **validDomains** 数组，添加令牌服务的域：

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]    
    ```

1. 保存所做的更改

## 任务 5：预配资源

现在一切都已准备就绪，运行“准备 Teams 应用依赖项”过程来预配所需的资源。

在 Visual Studio 中继续操作：

1. 在**解决方案资源管理器**中，右键单击 **MsgExtProductSupport** 项目
1. 展开 **Teams 工具包**菜单，选择 **“准备 Teams 应用依赖项”**
1. 在**Microsoft 365 帐户**对话框中，选择“继续”****
1. 在**预配**对话框中，选择“预配”****
1. 在 **Teams 工具包警告**对话框中，选择“预配”****
1. 在 **Teams 工具包信息** 对话框中，选择“查看预配的资源”**** 以打开新的浏览器窗口。

花点时间浏览在 Azure 中创建和更新的资源。

## 任务 6 - 运行和调试

现在，启动 Web 服务并在 Microsoft Teams 中测试消息扩展。

在 Visual Studio 中继续操作：

1. 按 **F5** 启动调试会话，并打开导航到 Microsoft Teams Web 客户端的新浏览器窗口。
1. 在浏览器中，如有必要，输入Microsoft 365 帐户凭据，然后继续 Microsoft Teams。
1. 在应用安装对话框中，选择 **“添加”**
1. 打开新的或现有的 Microsoft Teams 聊天
1. 在消息撰写区域中，选择 **“...”** 打开应用浮出控件
1. 在应用列表中，选择 **“Contoso 产品”**，以打开消息扩展
1. 在文本框中，输入 **Bot Builder** 以开始搜索
1. 在结果列表中，**选择要将卡片嵌入撰写消息框中的结果**
1. 显示一条消息，**需要登录才能使用此应用**
1. 选择“登录链接”****，打开新选项卡并启动身份验证流
1. 在“权限许可”页面，查看所请求的权限
1. 选择“接受”**，关闭选项卡并返回 Microsoft Teams
1. 在消息撰写区域中，选择 **“...”** 打开应用浮出控件
1. 在应用列表中，选择 **“Contoso 产品”**，以打开消息扩展
1. 在文本框中，输入 **Bot Builder** 以开始搜索
1. 系统会提示你再次登录。 再次按照**登录链接**开始搜索。
1. 在结果列表中，**选择要将卡片嵌入撰写消息框中的结果**

关闭浏览器，以停止调试会话。

[继续进行下一个练习...](./4-exercise-retrieve-product-information-from-sharepoint-online.md)