---
lab:
  title: 练习 1 - 将 API 插件与使用密钥保护的 API 集成
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# 练习 1 - 将 API 插件与使用密钥保护的 API 集成

智能 Microsoft 365 Copilot 副驾驶® 的 API 插件允许与使用密钥保护的 API 集成。 通过在 Teams 保管库中注册 API 密钥来保护该密钥。 在运行时，智能 Microsoft 365 Copilot 副驾驶® 执行插件，从保管库中检索 API 密钥，并使用它调用 API。 遵循此过程后，API 密钥将保持安全状态，并且永远不会向客户端公开。

在本练习中，你将使用 API 插件创建新的声明性代理，该插件使用生成的 API 密钥进行身份验证。

### 练习用时

- **估计完成时间：** 10 分钟

## 任务 1 - 创建新项目

首先，为 智能 Microsoft 365 Copilot 副驾驶® 创建新的 API 插件。 打开 Visual Studio Code。

在 Visual Studio Code 中：

1. 在“**活动栏**”（边栏）中，激活“Teams 工具包”扩展。
1. 在“**Teams 工具包**”扩展面板中，选择“**创建新应用**”。
1. 从项目模板列表中，选择 **Copilot 智能体**。
1. 从应用功能列表中，选择“**声明性代理**”。
1. 选择“**添加插件**”选项。
1. 选择“**从新 API 开始**”选项。
1. 从身份验证类型列表中选择 **API 密钥（持有者令牌身份验证）**。
1. 对于编程语言，选择“**TypeScript**”。
1. 选择要存储项目的文件夹。
1. 将项目命名为 **da-repairs-key**。

Teams 工具包创建一个新项目，其中包含声明性代理、API 插件和使用密钥保护的 API。

## 任务 2 - 检查 API 密钥身份验证配置

在继续之前，请检查生成的项目中的 API 密钥身份验证配置。

### 检查 API 定义

首先，看看 API 密钥身份验证在 API 定义中是如何定义的。

在 Visual Studio Code 中：

1. 打开 **appPackage/apiSpecificationFile/repair.yml** 文件。 此文件包含 API 的 OpenAPI 定义。
1. 在 **components.securitySchemes** 部分中，请注意 **apiKey** 属性：

  ```yml
  components:
    securitySchemes:
    apiKey:
      type: http
      scheme: bearer
  ```

  该属性定义一个安全方案，该方案使用 API 密钥作为授权请求标头中的持有者令牌。

1. 找到 **paths./repairs.get.security** 属性。 请注意，它引用 **apiKey** 安全方案。

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - apiKey: []
  [...] 
  ```

### 检查 API 实现情况

接下来，请参阅 API 如何在每个请求上验证 API 密钥。

在 Visual Studio Code 中：

1. 打开 **src/functions/repairs.ts** 文件。
1. 在 **repairs** 处理程序函数中，找到以下行来拒绝所有未经授权的请求：

  ```typescript
  if (!isApiKeyValid(req)) {
    // Return 401 Unauthorized response.
    return {
    status: 401,
    };
  } 
  ```

1. **isApiKeyValid** 函数在 repairs.ts 文件中进一步实现：

  ```typescript
  function isApiKeyValid(req: HttpRequest): boolean {
    const apiKey = req.headers.get("Authorization")?.replace("Bearer ", "").trim();
    return apiKey === process.env.API_KEY;
  }
  ```

  该函数检查授权标头是否包含持有者令牌，并将其与 **API_KEY** 环境变量中定义的 API 密钥进行比较。

此代码演示 API 密钥安全性的简单实现，但它说明了 API 密钥安全性在实践中的工作原理。

### 检查保管库任务配置

在此项目中，你将使用 Teams 工具包将 API 密钥添加到保管库。 Teams 工具包使用项目配置中的特殊任务在保管库中注册 API 密钥。

在 Visual Studio Code 中：

1. 打开 **./teampsapp.local.yml** 文件。
1. 在 **预配** 部分中，找到 **apiKey/register** 任务。

  ```yml
  # Register API KEY
  - uses: apiKey/register
    with:
    # Name of the API Key
    name: apiKey
    # Value of the API Key
    primaryClientSecret: ${{SECRET_API_KEY}}
    # Teams app ID
    appId: ${{TEAMS_APP_ID}}
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    # Write the registration information of API Key into environment file for
    # the specified environment variable(s).
    writeToEnvironmentFile:
    registrationId: APIKEY_REGISTRATION_ID
  ```

  该任务采用存储在 **env/.env.local.user** 文件中的 **SECRET_API_KEY** 项目变量的值，并将其注册到保管库中。 然后，它会获取保管库条目 ID，并将其写入环境文件 **env/.env.local**。 此任务的结果是名为 **APIKEY_REGISTRATION_ID** 的环境变量。 Teams 工具包将此变量的值写入包含插件定义的 **appPackages/ai-plugin.json** 文件。 在运行时，加载 API 插件的声明性代理使用此 ID 从保管库中检索 API 密钥，并安全地调用 API。

## 任务 3 - 配置用于本地开发的 API 密钥

在测试项目之前，需要为 API 定义 API 密钥。 然后，将 API 密钥存储在保管库中，并在 API 插件中记录保管库条目 ID。 对于本地开发，请将 API 密钥存储在项目中，并使用 Teams 工具包在保管库中注册。

在 Visual Studio Code 中：

1. 打开“**终端**”窗格 (Ctrl + `)。
1. 使用命令行：
  1. 运行 `npm install` 以安装项目的依赖项。
  1. 通过运行：`npm run keygen`，生成新的 API 密钥。
  1. 将生成的密钥复制到剪贴板。
1. 打开 **env/.env.local.user** 文件。
1. 将 **SECRET_API_KEY** 属性更新为新生成的 API 密钥。 更新的属性如下所示：

  ```text
  SECRET_API_KEY='your_key'
  ```

1. 保存所做更改。

每次生成项目时，Teams 工具包都会自动更新保管库中的 API 密钥，并使用保管库条目 ID 更新项目。