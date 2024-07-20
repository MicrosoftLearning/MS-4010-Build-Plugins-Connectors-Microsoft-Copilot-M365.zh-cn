---
lab:
  title: 练习 1 - 创建消息扩展
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 练习 1 - 创建消息扩展

在本练习中，你将使用搜索命令创建消息扩展。 首先使用 Teams 工具包项目模板搭建项目基架，然后更新项目，并使用 Azure AI 机器人服务资源配置项目，以进行本地开发。 创建开发隧道以启用机器人服务和本地运行的 Web 服务之间的通信。 然后，准备好应用来预配所需的资源。 最后，运行并调试消息扩展，并在 Microsoft Teams 中对其进行测试。

:::image type="content" source="../media/2-search-results-nuget.png" alt-text="Microsoft Teams 中基于搜索的消息扩展返回的搜索结果屏幕截图。" lightbox="../media/2-search-results-nuget.png":::

## 任务 1 - 使用适用于 Visual Studio 的 Teams 工具包创建新项目

首先创建一个新项目。

1. 打开 **Visual Studio 2022**
1. 打开 **“文件”** 菜单，展开 **“新建”** 菜单，选择 **“新建项目”**。
1. 在“创建新项目”屏幕中，展开 **“所有平台”** 下拉列表，然后选择 **“Microsoft Teams”**。 选择“下一步”继续操作。
1. 在“配置新项目”屏幕中。 指定以下值：
    1. **项目名称**：MsgExtProductSupport
    1. **位置**：选择所需的位置
    1. **将解决方案和项目放在同一目录中**：已选中
1. 通过选择 **“创建”** 来搭建项目基架
1. 在“创建新的 Teams 应用程序”对话框中，展开 **“所有应用类型”** 下拉列表，然后选择 **“消息扩展”**
1. 在模板列表中，选择 **“自定义搜索结果”**
1. 通过选择 **“创建”** 来搭建应用基架

## 任务 2 - 配置 Azure AI 机器人服务

机器人服务资源可以作为资源在 Azure 中创建，也可以通过 dev.botframework.com 创建。 默认情况下，自定义搜索结果模板使用 dev.botframework.com 注册机器人。 目前，使用 dev.botframework.com 注册机器人与适用于 Microsoft 365 的 Copilot 不兼容。

若要支持适用于 Microsoft 365 的 Copilot，请更新项目以在 Azure 中预配 Azure AI 机器人服务资源并将其用于本地开发。

首先，让我们创建一个环境变量来集中管理应用的内部名称，以便在文件中重复使用，并在预配资源时使用。

在 Visual Studio 中：

1. 在 **env** 文件夹中，打开 **.env.local**
1. 在  文件中添加以下代码：

    ```text
    APP_INTERNAL_NAME=msgext-product-support
    ```

1. 保存所做的更改

例如`${{APP_INTERNAL_NAME}}`，可以使用数据绑定表达式，这样就能在使用 Teams 工具包预配资源时将环境变量值注入文件中。

若要预配 Azure AI 机器人服务资源，需要 Microsoft Entra 应用注册。 创建 Teams 工具包用于预配应用注册的应用注册清单文件。

在 Visual Studio 中继续操作：

1. 在 **infra** 文件夹中，创建名为 **entra** 的新文件夹
1. 在文件夹中，创建名称为 **entra.bot.manifest.json** 的文件
1. 在  文件中添加以下代码：

    ```json
    {
      "id": "${{BOT_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{BOT_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-bot-${{TEAMSFX_ENV}}",
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
      "requiredResourceAccess": [],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": []
    }
    ```

1. 保存所做更改。

Teams 工具包使用 Bicep 文件在 Azure 中预配和配置资源。 首先，创建参数文件。 参数文件用于将环境变量传递到 Bicep 模板。

在 Visual Studio 中继续操作：

1. 在 **infra** 文件夹中，创建名为 **azure.parameters.local.json 的新文件**
1. 在  文件中添加以下代码：

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
        }
      }
    }
    ```

1. 保存所做更改。

现在，创建与参数文件一起使用的 Bicep 文件。

1. 在 **infra** 文件夹中，创建一个名为 **azure.local.bicep** 的新文件
1. 在文件中，添加以下代码：

    ```bicep
    @maxLength(20)
    @minLength(4)
    @description('Used to generate names for all resources in this file')
    param resourceBaseName string
    
    @description('Required when create Azure Bot service')
    param botEntraAppClientId string
    @maxLength(42)
    param botDisplayName string
    param botAppDomain string
    
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
      }
    }
    ```

1. 保存所做更改。

最后一步是更新 Teams 工具包项目文件。 替换使用 Bot Framework 操作的步骤，以便使用清单文件预配机器人 Microsoft Entra 应用程序注册，并使用 Bicep 文件预配 Azure AI 机器人服务资源。

在 Visual Studio 中继续操作：

1. 在项目根文件夹中，打开 **teamsapp.local.yml**
1. 在文件中，找到使用 **botAadApp/create** 操作的步骤，并将其替换为：

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-bot-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: BOT_ID
          clientSecret: SECRET_BOT_PASSWORD
          objectId: BOT_ENTRA_APP_OBJECT_ID
          tenantId: BOT_ENTRA_APP_TENANT_ID
          authority: BOT_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: BOT_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.bot.manifest.json"
          outputFilePath : "./build/entra.bot.manifest.${{TEAMSFX_ENV}}.json"
    
      - uses: arm/deploy
        with:
          subscriptionId: ${{AZURE_SUBSCRIPTION_ID}}
          resourceGroupName: ${{AZURE_RESOURCE_GROUP_NAME}}
          templates:
            - path: ./infra/azure.local.bicep
              parameters: ./infra/azure.parameters.local.json
              deploymentName: Create-resources-for-${{APP_INTERNAL_NAME}}-${{TEAMSFX_ENV}}
          bicepCliVersion: v0.9.1
    ```

1. 在文件中，删除使用 **botFramework/create** 操作的步骤
1. 保存所做更改。

应用注册通过两个步骤进行预配，首先 **aadApp/create** 操作使用客户端密码创建一个新的多租户应用注册，将其输出作为环境变量写入 **.env.local** 文件。 然后，**aadApp/update** 操作使用 **entra.bot.manifest.json** 文件更新应用注册。

最后一步使用 **arm/deploy** 操作通过 **azure.parameters.local.json** 文件和 **azure.local.bicep** 文件将 Azure AI 机器人服务资源预配到资源组。

## 任务 3 - 创建开发隧道

当用户与消息扩展交互时，机器人服务会将请求发送到 Web 服务。 在开发过程中，Web 服务在本地计算机上运行。 若要允许机器人服务访问 Web 服务，需要使用开发隧道将其暴露在计算机之外。

:::image type="content" source="../media/18-select-dev-tunnel.png" alt-text="Visual Studio 中展开的“开发隧道”菜单的屏幕截图。" lightbox="../media/18-select-dev-tunnel.png":::

在 Visual Studio 中继续操作：

1. 在工具栏上，通过**选择 Microsoft Teams（浏览器）按钮旁边的下拉列表**展开调试配置文件菜单
1. 展开 **“开发隧道”（无活动隧道）** 菜单，并选择 **“创建隧道...”**
1. 在对话框中，指定以下值：
    1. **帐户**：选择所需的帐户
    1. **名称**：MsgExtProductSupport
    1. **隧道类型**：临时
    1. **访问**：公开
1. 通过选择 **“确定”** 创建隧道，此时会出现提示，指出新隧道已成为当前活动隧道
1. 通过选择 **“确定”** 关闭提示

## 任务 4 - 更新应用清单

应用清单描述了应用的特性和功能。 更新应用清单中的属性，以更好地描述应用的功能及其特性。

首先，下载应用图标并将其添加到项目中。

:::image type="content" source="../media/app/color-local.png" alt-text="用于本地开发的颜色图标。" lightbox="../media/app/color-local.png":::

:::image type="content" source="../media/app/color-dev.png" alt-text="用于远程开发的颜色图标。" lightbox="../media/app/color-dev.png":::

1. 下载 **color-local.png** 和 **color-dev.png**
1. 在 **appPackage** 文件夹中，添加 **color-local.png** 和 **color-dev.png**
1. 在文件夹中，删除名为 **color.png** 的文件

由于应用名称在项目的不同位置重复，因此需要创建新的环境变量来集中存储该值。

在 Visual Studio 中继续操作：

1. 在 **env** 文件夹中，打开名为 **.env.local** 的文件
1. 在文件中，添加以下代码：

    ```text
    APP_DISPLAY_NAME=Contoso products
    ```

1. 保存所做的更改

最后，更新应用清单文件中的图标、名称和说明对象。

1. 在 **appPackage** 文件夹中，打开名为 **manifest.json** 的文件
1. 在文件中，使用以下内容更新**图标**、**名称和**说明**对象：

    ```json
    {
        "icons": {
            "color": "color-${{TEAMSFX_ENV}}.png",
            "outline": "outline.png"
        },
        "name": {
            "short": "${{APP_DISPLAY_NAME}}",
            "full": "${{APP_DISPLAY_NAME}}"
        },
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation."
        }
    }
    ```

1. 保存所做的更改

## 任务 6 - 预配资源

一切就绪后，使用 Teams 工具包，运行“准备 Teams 应用依赖项”流程来预配所需的资源。

:::image type="content" source="../media/19-prepare-teams-app-dependencies.png" alt-text="Visual Studio 中展开的 Teams 工具包菜单的屏幕截图。" lightbox="../media/19-prepare-teams-app-dependencies.png":::

“准备 Teams 应用依赖项”流程使用活动开发隧道 URL 更新 .env.local 文件中的 **BOT_ENDPOINT** 和 **BOT_DOMAIN** 环境变量，并执行 **teamsapp.local.yml** 文件中所述的操作。

在 Visual Studio 中继续操作：

1. 在解决方案资源管理器中，右键单击 **MsgExtProductSupport** 项目
1. 展开 **Teams 工具包**菜单，选择 **“准备 Teams 应用依赖项”**
1. 在 **Microsoft 365 帐户**对话框中，选择开发人员租户的帐户，然后选择**继续**
1. 在 **“预配”** 对话框中，选择要用于将资源部署到 Azure 的帐户，并指定以下值：
    1. **订阅名称**：从下拉列表中选择订阅
    1. **资源组**：选择“新建...”，打开对话框，输入 **rg-msgext-product-support-local，然后选择 **“确定”**
    1. **区域**：从下拉列表中选择离你最近的区域
1. 通过选择 **“预配”** 在 Azure 中预配资源
1. 在 Teams 工具包警告提示中，选择 **“预配”**
1. 在 Teams 工具包信息提示中，选择 **“查看预配的资源**”以打开新的浏览器窗口。

花点时间探索在 Azure 中创建的资源。

## 任务 7 - 运行和调试  

现在启动 Web 服务并测试消息扩展。 使用 Teams 工具包上传应用清单并在 Microsoft Teams 中测试消息扩展。

在 Visual Studio 中继续操作：

1. 按 F5 启动调试会话，并打开导航 Microsoft Teams Web 客户端的新浏览器窗口。
1. 出现提示时，输入 Microsoft 365 帐户凭据

  > [!IMPORTANT]
  > 如果 Microsoft Teams 中出现一个对话框，其中显示“找不到此应用”消息，请按照以下步骤手动上传应用包：
  >
  >  1. 关闭对话
  >  2. 在侧边栏上，转到**应用**
  >  3. 在左侧菜单中，选择 **“管理应用”**
  >  4. 在命令栏中，选择**上传应用**
  >  5. 在对话框中，选择 **“上传自定义应用”**
  >  6. 在文件资源管理器中，导航到解决方案文件夹，打开 **appPackage\build** 文件夹并选择 **appPackage.local.zip**，然后选择**添加**

继续安装应用：

1. 在应用安装对话框中，选择 **“添加”**
1. 打开新的或现有的 Microsoft Teams 聊天
1. 在消息撰写区域中，选择 **“...”** 打开应用浮出控件
1. 在应用列表中，选择 **“Contoso 产品”**，以打开消息扩展
1. 在文本框中，输入 **Bot Builder** 以开始搜索
1. 在结果列表中，选择一个结果，以将卡片嵌入撰写消息框中

关闭浏览器，以停止调试会话。

[继续下一个练习...](./3-exercise-add-single-sign-on.md)