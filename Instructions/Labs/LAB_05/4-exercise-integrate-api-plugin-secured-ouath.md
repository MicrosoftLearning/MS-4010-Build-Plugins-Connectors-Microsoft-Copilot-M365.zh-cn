---
lab:
  title: 练习 3 - 将 API 插件与使用 OAuth 保护的 API 集成
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# 练习 3 - 将 API 插件与使用 OAuth 保护的 API 集成

智能 Microsoft 365 Copilot 副驾驶® 的 API 插件允许与使用 OAuth 保护的 API 集成。 通过在 Teams 保管库中注册 API 来保护 API 安全的应用的客户端 ID 和机密。 在运行时，Microsoft 365 Copilot 执行插件，从保管库中检索信息，并使用它获取访问令牌并调用 API。 遵循此过程后，客户端 ID 和机密将保持安全状态，并且永远不会向客户端公开。

### 练习用时

- **估计完成时间：** 10 分钟

## 任务 1 - 打开示例项目并检查身份验证配置

首先下载示例项目：

1. 在网页浏览器中，导航到 [https://aka.ms/learn-da-api-ts-repairs](https://aka.ms/learn-da-api-ts-repairs)。 系统会提示使用示例项目下载 ZIP 文件。
1. 将该 zip 文件保存在计算机上。
1. 提取 zip 文件内容。
1. 在 Visual Studio Code 中打开  文件夹，

示例项目是一个 Teams 工具包项目，其中包含一个声明性代理、一个 API 插件以及一个通过 Microsoft Entra ID 进行保护的 API。 该 API 在 Azure Functions 上运行，并使用 Azure Functions 的内置身份验证和授权功能（有时称为 Easy Auth）实现安全性。

## 任务 2 - 检查 OAuth2 授权配置

在继续之前，请检查示例项目中的 OAuth2 授权配置。

### 检查 API 定义

首先，查看项目中包含的 API 定义的安全配置。

在 Visual Studio Code 中：

1. 打开 **appPackage/apiSpecificationFile/repair.yml** 文件。
1. 在 **components.securitySchemes** 部分中，请注意 **oAuth2AuthCode** 属性：

  ```yml
  components:
    securitySchemes:
    oAuth2AuthCode:
      type: oauth2
      description: OAuth configuration for the repair service
      flows:
      authorizationCode:
        authorizationUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/authorize
        tokenUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/token
        scopes:
        api://${{AAD_APP_CLIENT_ID}}/repairs_read: Read repair records 
  ```

  该属性定义了一个 OAuth2 安全方案，并包含获取访问令牌所需调用的 URL 以及 API 使用的范围等信息。

  > **重要提示**请注意，作用域使用应用程序 ID URI (**api://...**) 完全限定。使用 Microsoft Entra 时，需要完全限定自定义范围。 当 Microsoft Entra 看到一个不限定的范围时，它会假定其属于 Microsoft Graph，这会导致授权流错误。

1. 找到 **paths./repairs.get.security** 属性。 请注意，它引用客户端执行操作所需的 **oAuth2AuthCode** 安全方案和范围。

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - oAuth2AuthCode:
        - api://${{AAD_APP_CLIENT_ID}}/repairs_read
  [...]
  ```

  > **重要提示** 在 API 规范中列出必要的范围纯粹是信息性的。 实现 API 时，你负责验证令牌并检查其是否包含必要的范围。

### 检查 API 实现情况

接下来，查看 API 实现。

在 Visual Studio Code 中：

1. 打开 **src/functions/repairs.ts** 文件。
1. 在 **repairs** 处理程序函数中，找到以下行，用于检查请求是否包含具有必要范围的访问令牌：

  ```typescript
  if (!hasRequiredScopes(req, 'repairs_read')) {
    return {
    status: 403,
    body: "Insufficient permissions",
    };
  }
  ```

1. **hasRequiredScopes** 函数在 **repairs.ts** 文件中进一步实现：

  ```typescript
  function hasRequiredScopes(req: HttpRequest, requiredScopes: string[] | string): boolean {
    if (typeof requiredScopes === 'string') {
    requiredScopes = [requiredScopes];
    }
  
    const token = req.headers.get("Authorization")?.split(" ");
    if (!token || token[0] !== "Bearer") {
    return false;
    }
  
    try {
    const decodedToken = jwtDecode<JwtPayload & { scp?: string }>(token[1]);
    const scopes = decodedToken.scp?.split(" ") ?? [];
    return requiredScopes.every(scope => scopes.includes(scope));
    }
    catch (error) {
    return false;
    }
  }
  ```

  该函数首先从授权请求头中提取持有者令牌。 接下来，它使用 **jwt-decode** 包解码令牌并从 **scp** 声明中获取范围列表。 最后，它会检查 **scp** 声明是否包含所有必需的范围。

  请注意，该函数未验证访问令牌。 相反，它只会检查访问令牌是否包含所需的范围。 在此模板中，API 在 Azure Functions 上运行，并使用 Easy Auth 实现安全性，后者负责验证访问令牌。 如果请求不包含有效的访问令牌，则 Azure Functions 运行时在到达代码之前会拒绝它。 虽然 Easy Auth 会验证令牌，但它不会检查所需的范围，这需要你自己完成。

### 检查保管库任务配置

在此项目中，将使用 Teams 工具包将 OAuth 信息添加到保管库。 Teams 工具包使用项目配置中的特殊任务，在保管库中注册 OAuth 信息。

在 Visual Studio Code 中：

1. 打开 **./teampsapp.local.yml** 文件。
1. 在 **预配** 部分中，找到 **oauth/register** 任务。

  ```yml
  - uses: oauth/register
    with:
    name: oAuth2AuthCode
    flow: authorizationCode
    appId: ${{TEAMS_APP_ID}}
    clientId: ${{AAD_APP_CLIENT_ID}}
    clientSecret: ${{SECRET_AAD_APP_CLIENT_SECRET}}
    isPKCEEnabled: true
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    writeToEnvironmentFile:
    configurationId: OAUTH2AUTHCODE_CONFIGURATION_ID
  ```

  该任务采用存储在 **env/.env.local** 和 **env/.env.local.user** 文件中的 **TEAMS_APP_ID**、**AAD_APP_CLIENT_ID** 和 **SECRET_AAD_APP_CLIENT_SECRET** 项目变量的值，并将其注册到保管库中。 还启用代码交换的证明密钥 (PKCE)，以提高安全性。 然后，它会获取保管库条目 ID，并将其写入环境文件 **env/.env.local**。 此任务的结果是名为 **OAUTH2AUTHCODE_CONFIGURATION_ID** 的环境变量。 Teams 工具包将此变量的值写入包含插件定义的 **appPackages/ai-plugin.json** 文件。 在运行时，加载 API 插件的声明性代理使用此 ID 从保管库中检索 OAuth 信息，并以安全方式调用 API。

  > **重要说明**：**oauth/register** 任务仅负责在保管库中注册 OAuth 信息（如果尚不存在）。 如果信息已存在，Teams 工具包将跳过运行此任务。

1. 接下来，找到 **oauth/update** 任务。

  ```yml
  - uses: oauth/update
    with:
    name: oAuth2AuthCode
    appId: ${{TEAMS_APP_ID}}
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    configurationId: ${{OAUTH2AUTHCODE_CONFIGURATION_ID}}
    isPKCEEnabled: true
  ```

  该任务使保管库中的 OAuth 信息与项目保持同步。 项目必须正常工作。 其中一个关键属性是 API 插件可用的 URL。 每次启动项目时，Teams 工具包都会在新 URL 上打开开发隧道。 保管库中的 OAuth 信息需要引用此 URL，以便 Copilot 能够访问 API。

### 检查身份验证和授权配置

要探索的下一部分是 Azure Functions 的身份验证和授权设置。 本练习中的 API 使用 Azure Functions 的内置身份验证和授权功能。 Teams 工具包在将 Azure Functions 预配到 Azure 时配置这些功能。

在 Visual Studio Code 中：

1. 打开 **infra/azure.bicep** 文件。
1. 找到 **authSettings** 资源：

  ```bicep
  resource authSettings 'Microsoft.Web/sites/config@2021-02-01' = {
    parent: functionApp
    name: 'authsettingsV2'
    properties: {
    globalValidation: {
      requireAuthentication: true
      unauthenticatedClientAction: 'Return401'
    }
    identityProviders: {
      azureActiveDirectory: {
      enabled: true
      registration: {
        openIdIssuer: oauthAuthority
        clientId: aadAppClientId
      }
      validation: {
        allowedAudiences: [
        aadAppClientId
        aadApplicationIdUri
        ]
      }
      }
    }
    }
  }
  ```

  此资源在 Azure Functions 应用中启用内置身份验证和授权功能。 首先，在 **globalValidation** 部分中，它定义应用仅允许经过身份验证的请求。 如果应用收到未经身份验证的请求，它会拒绝 401 HTTP 错误。 然后，在 **identityProviders** 部分中，配置定义它使用 Microsoft Entra ID（以前称为 Azure Active Directory）来授权请求。 指定用于保护 API 安全的 Microsoft Entra 应用注册，并指定允许调用 API 的访问群体。

### 检查 Microsoft Entra 应用程序注册

要检查的最后一部分是项目用来保护 API 的 Microsoft Entra 应用程序注册。 使用 OAuth 时，可以使用应用程序保护对资源的访问。 应用程序通常定义获取访问令牌所需的凭据，例如客户端机密或证书。 还指定客户端在调用 API 时可以请求的不同权限（也称为范围）。 Microsoft Entra 应用程序注册表示 Microsoft 云中的应用程序，并定义用于 OAuth 授权流的应用程序。

在 Visual Studio Code 中：

1. 打开 **./aad.manifest.json** 文件。
1. 找到 **oauth2Permissions** 属性。

  ```json
  "oauth2Permissions": [
    {
    "adminConsentDescription": "Allows Copilot to read repair records on your behalf.",
    "adminConsentDisplayName": "Read repairs",
    "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
    "isEnabled": true,
    "type": "User",
    "userConsentDescription": "Allows Copilot to read repair records.",
    "userConsentDisplayName": "Read repairs",
    "value": "repairs_read"
    }
  ],
  ```

  该属性定义名为 **repairs_read** 的自定义范围，以授予客户端从修复 API 读取修复的权限。

1. 找到 **identifierUris** 属性。

  ```json
  "identifierUris": [
    "api://${{AAD_APP_CLIENT_ID}}"
  ]
  ```

  **identifierUris** 属性定义用于完全限定范围的标识符。
