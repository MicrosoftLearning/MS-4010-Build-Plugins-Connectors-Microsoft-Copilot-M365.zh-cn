---
lab:
  title: 练习 2 - 添加单一登录
  module: 'LAB 01: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 练习 2 - 添加单一登录

在本练习中，将单一登录添加到消息扩展以对用户查询进行身份验证。

![基于搜索的消息扩展中身份验证质询的屏幕截图。 将显示登录链接。](../media/2-sign-in.png)

### 练习用时

  - **估计完成时间：** 40 分钟

## 任务 1 - 配置后端 API 应用注册

首先，为后端 API 创建 Microsoft Entra 应用注册。 在本练习中，你将创建新的应用注册。但是，在生产环境中，将使用现有的应用注册。

在浏览器窗口中：

1. 导航到 [Azure 门户](https://portal.azure.com)。

1. 打开门户菜单，然后选择 **Microsoft Entra ID**。

1. 选择“**管理 > 应用注册**”，然后选择“**新建注册**”。

1. 在“注册应用程序”窗体中，指定以下值：

    1. **名称**：产品 API

    1. **支持帐户类型**：任何组织目录中的帐户（任何 Microsoft Entra ID 租户 - 多租户）

1. 选择“注册”以创建应用注册。****

1. 在应用注册左侧菜单中，选择“**管理 > 公开 API**”。

1. 在“**应用程序 ID URI**”旁边，选择“**添加**”并“**保存**”以创建新的应用程序 ID URI。

1. 在“此 API 定义的范围”部分，选择“**添加范围**”。

1. 在“添加范围”窗体中，指定以下值：

    1. **范围名称**：Product.Read

    1. **谁能同意？**：管理员和用户

    1. **管理员同意显示名称**：读取产品

    1. **管理员同意说明**：允许应用读取产品数据

    1. **用户同意显示名称**：读取产品

    1. **用户同意说明**：允许应用读取产品数据

    1. **状态**: 已启用

1. 选择“**添加范围**”以创建范围。

接下来，记下应用注册 ID 和范围 ID。 需要这些值来配置用于获取后端 API 访问令牌的应用注册。

1. 在应用注册左侧菜单中，选择“**清单**”。

1. 复制 **appId** 属性值并保存供以后使用。

1. 复制 ** oauth2Permissions.id** 属性值并保存供以后使用。

由于我们需要项目中的这些值，请将它们添加到环境文件。

在 Visual Studio 和 **TeamsApp** 项目中：

1. 在 **env** 文件夹中，打开 **.env.local**

1. 在文件中，添加以下环境变量，并将值设置为之前保存的**应用注册 ID** 和**范围 ID**：

    ```text
    BACKEND_API_ENTRA_APP_ID=<app-registration-id>
    BACKEND_API_ENTRA_APP_SCOPE_ID=<scope-id>
    ```

1. 保存所做更改。

## 任务 2 - 创建应用注册清单文件以使用后端 API 进行身份验证

若要使用后端 API 进行身份验证，需要应用注册才能获取用于调用 API 的访问令牌。

接下来，创建应用注册清单文件。 清单定义应用注册上的 API 权限范围和重定向 URI。

在 Visual Studio 和 **TeamsApp** 项目中：

1. 在 **infra\entra** 文件夹中，创建名为 **entra.products.api.manifest.json** 的新文件 (<kbd>Ctrl+Shift+A</kbd>)。

1. 在文件中，添加以下代码：

    ```json
    {
      "id": "${{PRODUCTS_API_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{PRODUCTS_API_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-product-api-${{TEAMSFX_ENV}}",
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
          "resourceAppId": "${{BACKEND_API_ENTRA_APP_ID}}",
          "resourceAccess": [
            {
              "id": "${{BACKEND_API_ENTRA_APP_SCOPE_ID}}",
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

1. 保存所做更改。

**requiredResourceAccess** 属性指定应用注册 ID 和后端 API 的范围 ID。

**replyUrlsWithType** 属性指定 Bot Framework 令牌服务用于在用户进行身份验证后将访问令牌返回到令牌服务的重定向 URI。

接下来，更新自动化工作流以创建和更新应用注册。

在 **TeamsApp** 项目中：

1. 打开 **teamsapp.local.yml**。

1. 在文件中，找到使用 **arm/deploy** 操作的步骤。

1. 操作后，添加 **aadApp/create** 和 **aadApp/update** 操作以创建和更新应用注册（从**第 31 行**开始）：

    ```yml
      - uses: aadApp/create
        with:
            name: ${{APP_INTERNAL_NAME}}-products-api-${{TEAMSFX_ENV}}
            generateClientSecret: true
            signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
            clientId: PRODUCTS_API_ENTRA_APP_ID
            clientSecret: SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET
            objectId: PRODUCTS_API_ENTRA_APP_OBJECT_ID
            tenantId: PRODUCTS_API_ENTRA_APP_TENANT_ID
            authority: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY
            authorityHost: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
            manifestPath: "./infra/entra/entra.products.api.manifest.json"
            outputFilePath : "./infra/entra/build/entra.products.api.${{TEAMSFX_ENV}}.json"
    ```

1. 保存所做的更改

**aadApp/create** 操作使用指定的名称、受众创建新的应用注册，并生成客户端密码。 **writeToEnvironmentFile** 属性将应用注册 ID、客户端密码、对象 ID、租户 ID、颁发机构和颁发机构主机写入环境文件。 客户端密码已加密并安全地存储在 **env.local.user** 文件中。 客户端密码的环境变量名称以 **SECRET_** 为前缀，它告知 Teams 工具包不要在日志中写入值。

**aadApp/update** 操作使用指定的清单文件更新应用注册。

## 任务 3 - 集中化连接设置名称

接下来，将环境文件中的连接设置名称进行集中，并更新应用配置，以便在运行时访问环境变量。

在 Visual Studio 和 **TeamsApp** 项目中继续操作：

1. 在 **env** 文件夹中，打开 **.env.local**

1. 在文件中，添加以下代码：

    ```text
    CONNECTION_NAME=ProductsAPI
    ```

1. 打开 **teamsapp.local.yml**。

1. 在文件中，找到面向 **./appsettings Development.json ** 文件，使用 **file/createOrUpdateJsonFile** 操作的步骤。 更新内容数组以包含 **CONNECTION_NAME** 环境变量，并将该值写入 **appsettings.Development.json**文件：

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ../ProductsPlugin/appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. 保存所做更改。

接下来，更新应用配置以访问 **CONNECTION_NAME** 环境变量。

在 **ProductsPlugin** 项目中：

1. 打开 **Config.cs**。

1. 在 **ConfigOptions** 类中，添加名为 **CONNECTION_NAME** 的新属性：

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. 保存所做更改。

1. 打开 Program.cs。

1. 在文件中，更新读取应用程序配置的代码以包含 **CONNECTION_NAME** 属性：

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["ConnectionName"] = config.CONNECTION_NAME;
    ```

1. 保存所做更改。

接下来，更新机器人代码以在运行时使用连接设置名称。

1. 在 **Search** 文件夹中，打开 **SearchApp.cs**。

1. 在 **SearchApp** 类的开头，创建接受 **IConfiguration** 对象的构造函数，并将 **CONNECTION_NAME** 属性的值分配给名为 **connectionName** 的专用字段：

    ```csharp
    private readonly string connectionName;
    public SearchApp(IConfiguration configuration)
    {
      connectionName = configuration["CONNECTION_NAME"];
    }  
    ```

1. 保存所做更改。

## 任务 4 - 配置产品 API 连接设置

要使用后端 API 进行身份验证，需要在 Azure 机器人资源中配置连接设置。

在 Visual Studio 和 **TeamsApp** 项目中继续操作：

1. 在 **infra** 文件夹中，打开名为 **azure.parameters.local.json** 的文件。

1. 在文件中，添加 **backendApiEntraAppClientId**、**productsApiEntraAppClientId**、**productsApiEntraAppClientSecret** 和 **connectionName** 参数：

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
        "backendApiEntraAppClientId": {
          "value": "${{BACKEND_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientId": {
          "value": "${{PRODUCTS_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientSecret": {
          "value": "${{SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. 保存所做更改。

接下来，更新 Bicep 文件以包含新参数并将其传递给 Azure 机器人资源。

1. 在 **infra** 文件夹中，打开名为 **azure.local.bicep** 的文件。

1. 在文件中，在 **botAppDomain** 参数声明后，添加 **backendApiEntraAppClientId**、**productsApiEntraAppClientId**、**productsApiEntraAppClientSecret** 和 **connectionName** 参数声明：

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. 在 **azureBotRegistration** 模块声明中，添加新参数：

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botEntraAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        backendApiEntraAppClientId: backendApiEntraAppClientId
        productsApiEntraAppClientId: productsApiEntraAppClientId
        productsApiEntraAppClientSecret: productsApiEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. 保存所做更改。

最后，更新机器人注册 Bicep 文件以包含新的连接设置。

1. 在 **infra/botRegistration** 文件夹中，打开 **azurebot.bicep**

1. 在文件中，在 **botAppDomain** 参数声明后，添加 **backendApiEntraAppClientId**、**productsApiEntraAppClientId**、**productsApiEntraAppClientSecret** 和 **connectionName** 参数声明：

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. 在文件中，将名为 **botServicesProductsApiConnection** 的新资源添加到文件末尾：

    ```bicep
    resource botServicesProductsApiConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: productsApiEntraAppClientId
        clientSecret: productsApiEntraAppClientSecret
        scopes: 'api://${backendApiEntraAppClientId}/Product.Read'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${botEntraAppClientId}'
          }
        ]
      }
    }
    ```

1. 保存所做更改。

## 任务 5 - 在消息扩展中配置身份验证

若要在消息扩展中对用户查询进行身份验证，请使用 Bot Framework SDK 从 Bot Framework 令牌服务获取用户的访问令牌。 然后，可以使用访问令牌从外部服务访问数据。

要简化代码，请创建一个用于处理用户身份验证的帮助程序类。

在 Visual Studio 和 **ProductsPlugin** 项目中继续操作：

1. 创建名为 **Helpers** 的新文件夹。

1. 在 **Helpers** 文件夹中，创建名为 **AuthHelpers.cs** 的新类文件

1. 在文件中，添加以下代码：

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    using Microsoft.Bot.Schema.Teams;
    internal static class AuthHelpers
    {
        internal static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
        {
            var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);
            return new MessagingExtensionResponse
            {
                ComposeExtension = new MessagingExtensionResult
                {
                    Type = "auth",
                    SuggestedActions = new MessagingExtensionSuggestedAction
                    {
                        Actions = [
                            new() {
                                Type = ActionTypes.OpenUrl,
                                Value = resource.SignInLink,
                                Title = "Sign In",
                            },
                        ],
                    },
                },
            };
        }
        internal static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
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
        internal static bool HasToken(TokenResponse tokenResponse) => tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
    }
    ```

1. 保存所做更改。

**AuthHelpers** 类中的三个帮助程序方法处理消息扩展中的用户身份验证。

- **CreateAuthResponse** 方法构造一个响应，在用户界面中呈现登录链接。 首先，使用 **GetSignInResourceAsync** 方法从令牌服务检索登录链接。

- **GetToken** 方法使用令牌服务客户端获取当前用户的访问令牌。 该方法使用魔码验证请求的真实性。

- **HasToken** 方法检查来自令牌服务的响应是否包含访问令牌。 如果令牌不为 null 或空，则该方法返回 true。

接下来，更新消息扩展代码以使用帮助程序方法对用户查询进行身份验证。

1. 在 **Search** 文件夹中，打开 **SearchApp.cs**。

1. 在该文件的顶部，添加以下 using 语句：

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. 在 **OnTeamsMessagingExtensionQueryAsync** 方法的开头添加以下代码：

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

1. 保存所做更改。

接下来，将令牌服务域添加到应用清单文件，以确保客户端在启动单一登录流时可以信任域。

在 **TeamsApp** 项目中：

1. 在 **appPackage** 文件夹中，打开 **manifest.json**。

1. 在文件中，更新 **validDomains** 数组，添加令牌服务的域：

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]
    ```

1. 保存所做更改。

## 任务 6 - 创建和更新资源

现在一切就绪后，运行**准备 Teams 应用依赖项**进程以新建资源并更新现有资源。

> [!NOTE]
> 如果预配无法准备依赖项，请确保在 **env.local** 中具有正确的 **BACKEND_API_ENTRA_APP_ID** 和 **BACKEND_API_ENTRA_APP_SCOPE_ID** 值。

在 Visual Studio 中继续操作：

1. 在**解决方案资源管理器**中，右键单击 **TeamsApp** 项目。

1. 展开“**Teams 工具包**”菜单，选择“**准备 Teams 应用依赖项**”。

1. 在“**Microsoft 365 帐户**”对话框中，选择“**继续**”。

1. 在“**预配**”对话框中，选择“**预配**”。

1. 在“**Teams 工具包警告**”对话框中，选择“**预配**”。

1. 在“**Teams 工具包信息**”对话框中，选择交叉图标以关闭对话框。

## 任务 7 - 运行和调试

预配资源后，启动调试会话以测试消息扩展。

1. 要启动新的调试会话，请按 <kbd>F5</kbd> 或从工具栏中选择“**开始**”。

1. 等待浏览器窗口打开，应用安装对话框将显示在 Microsoft Teams Web 客户端中。 出现提示时，输入 Microsoft 365 帐户凭据。

1. 在应用安装对话框中，选择“**添加**”。

1. 打开新的或现有的 Microsoft Teams 聊天。

1. 在消息撰写区域中，开始键入 **/apps** 以打开应用选取器。

1. 在应用列表中，选择“**Contoso 产品**”，以打开消息扩展。

1. 在文本框中输入 **hello**。 可能需要多次输入搜索。

1. 显示一条消息：**需要登录才能使用此应用**

    ![基于搜索的消息扩展中身份验证质询的屏幕截图。 将显示登录链接。](../media/2-sign-in.png)

1. 按照**登录**链接启动身份验证流。

1. 同意请求的权限并返回到 Microsoft Teams：

    ![Microsoft Entra API 权限许可对话框的屏幕截图。](../media/18-api-permission-consent.png)

1. 等待搜索完成并显示结果。

1. 在结果列表中，选择 **hello**，以将卡片嵌入撰写消息框中

返回到 Visual Studio，然后从工具栏中选择“**停止**”，或按 <kbd>Shift</kbd> + <kbd>F5</kbd> 停止调试会话。

[继续进行下一个练习...](./4-exercise-return-data-api.md)