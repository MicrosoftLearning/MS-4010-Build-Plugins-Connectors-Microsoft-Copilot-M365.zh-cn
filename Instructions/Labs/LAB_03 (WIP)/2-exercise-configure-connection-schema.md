---
lab:
  title: 练习 1 - 配置外部连接并部署架构
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# 练习 - 配置外部连接并部署架构

在本练习中，你将生成自定义 Microsoft Graph 连接器作为控制台应用程序。 注册新的 Microsoft Entra 应用注册，并添加代码以创建外部连接并部署其架构。

## 开始之前

完成本练习大约需要** XX 分钟**。

## 任务 1 - 注册新的 Microsoft Entra 应用注册

首先，注册新的 Entra 应用注册，自定义 Graph 连接器使用该注册向 Microsoft 365 进行身份验证。

在 Web 浏览器中：

1. 转到 **Azure 门户**，地址为 [https://portal.azure.com](https://portal.azure.com)。
1. 从导航栏的“Microsoft Entra ID”**** 下方选择“查看”****。
1. 从侧边导航中展开“管理”****，然后选择“应用注册”****。
1. 在顶部导航中，选择 “新注册”****。
1. 指定以下值：
   1. **名称：** MSGraph 文档 Graph 连接器
   1. **支持的帐户类型：** 仅此组织目录中的帐户（单个租户）
1. 选择“注册“**** 以确认输入
1. 在概述屏幕中，复制**应用程序 ID** 和**目录（租户）ID** 属性的值。 稍后将用到它们。

## 任务 2 - 创建凭据

由于此自定义 Graph 连接器无需用户交互即可运行，因此需要将其配置为自动进行身份验证。 为简单起见，请创建机密。

在 Web 浏览器中继续操作：

1. 从侧边导航中，展开“管理”****，然后选择“证书和机密”****。
1. 选择“客户端密码”**** 选项卡，然后选择“新建客户端密码”****。
1. 输入 MSGraph 文档 Graph 连接器机密**** 的说明****。
1. 通过选择“添加”**** 创建机密。
1. 复制新建机密的**值**。 稍后需要用到此信息。

## 任务 3 - 授予 API 权限

配置 Entra 应用注册的最后一步是向其授予 API 权限，以便它可以创建外部连接和架构。

在 Web 浏览器中继续操作：

1. 从侧边导航中，选择“API 权限”****。
1. 选择“**添加权限**”。
1. 从 API 的列表中，选择“Microsoft Graph”****。
1. 接下来，选择“应用程序权限”****。
1. 在筛选器文本框中，输入**外部**。
1. 展开 **ExternalConnection** 部分，然后选择 **ExternalConnection.ReadWrite.OwnedBy** 权限。
1. 展开 **ExternalItem** 部分，然后选择 **ExternalItem.ReadWrite.OwnedBy** 权限。
1. 若要确认选择，请选择“添加权限”按钮****。
1. 若要完成配置，请选择“为（租户）授予管理员同意”按钮来授予管理员同意****。
1. 选择“是”确认对话****。

## 任务 4 - 创建新的控制台应用并安装依赖项

配置 Entra 应用注册后，下一步是创建控制台应用，你将在其中实现 Graph 连接器的代码。

打开 Windows 终端以新建控制台应用程序：

1. 输入 `mkdir documents\console_app` 以新建文件夹，然后输入 `cd .\documents\console_app` 以导航到新建文件夹。
1. 运行 `dotnet new console` 以新建控制台应用程序
1. 添加需要生成连接器的依赖项：
   1. 若要添加 Microsoft 365 进行身份验证所需的库，请运行 `dotnet add package Azure.Identity`。
   1. 若要添加客户端库以与 Graph API 通信，请运行 `dotnet add package Microsoft.Graph`。
   1. 若要添加处理用户机密所需的库，将在下一步中配置该机密，请运行 `dotnet add package Microsoft.Extensions.Configuration.UserSecrets`。
   1. 将 Graph 连接器实现为具有两个命令的控制台应用：一个用于创建外部连接并部署架构，另一个用于导入内容。 若要支持在应用中定义命令，请运行 `dotnet add package System.CommandLine --prerelease`。

## 任务 5 - 安全地存储 Entra 应用注册信息

创建 Entra 应用注册后，会记录其信息，例如应用程序和租户 ID 以及机密。 在连接器中使用此信息通过 Microsoft 365 进行身份验证。 若要使其在代码中可用，可以将其作为用户机密安全地与项目一起存储。

在终端中：

1. 确保工作目录设置为新创建的控制台应用。
1. 若要启动用户机密，请运行 `dotnet user-secrets init`。
1. 若要安全地存储有关应用注册的信息，请将令牌替换为之前复制的实际值并运行：

   ```dotnetcli
   dotnet user-secrets set "EntraId:ClientId" "[application id]"
   dotnet user-secrets set "EntraId:ClientSecret" "[secret value]"
   dotnet user-secrets set "EntraId:TenantId" "[directory (tenant) id]"
   ```

## 任务 6 - 创建 Microsoft Graph 客户端

自定义 Graph 连接器使用 Microsoft Graph API 来管理其外部连接和项。 首先，从项目中安装的 Microsoft.Graph**** NuGet 包中创建 `GraphServiceClient` 类的实例。

1. 在 Visual Studio 2022 中打开项目。
1. 在项目中，添加名为 **GraphService.cs** 的新代码文件。
1. 在文件中，首先添加对要使用的命名空间的引用，方法是添加：

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   ```

1. 接下来，定义名为 GraphService 的新类：

   ```csharp
   class GraphService
   {
   }
   ```

1. 在 `GraphService` 类中，定义一个单一实例来存储 `GraphServiceClient` 实例，以便与 Microsoft Graph API 进行通信：

   ```csharp
   class GraphService
   {
     static GraphServiceClient? _client;

     public static GraphServiceClient Client
     {
       get
       {
         // TODO: implement
       }
     }
   }
   ```

1. 在 `Client` 单一实例中，实现 Getter，以便创建一个新的 `GraphServiceClient` 实例（如果尚不存在）。

   ```csharp
   public static GraphServiceClient Client
   {
     get
     {
       if (_client is null)
       {
         // TODO: implement
       }
       return _client;
     }
   }
   ```

1. 在 getting 内部，使用包含此前存储的 Entra 应用注册信息的凭据新建 `GraphServiceClient` 的实例：

   ```csharp
   var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
   var config = builder.Build();

   var clientId = config["EntraId:ClientId"];
   var clientSecret = config["EntraId:ClientSecret"];
   var tenantId = config["EntraId:TenantId"];

   var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);
   _client = new GraphServiceClient(credential);
   ```

   首先，创建配置生成器以访问存储在用户机密中的 Entra 应用注册的相关信息。 接下来，使用生成器检索应用注册信息。 然后，创建新的客户端密码凭据，传递租户和客户端 ID 以及客户端密码。 最后，创建 `GraphServiceClient` 实例，传递新创建的凭据。

1. 完整的代码如下所示：

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   
   class GraphService
   {
     static GraphServiceClient? _client;
   
     public static GraphServiceClient Client
     {
       get
       {
         if (_client is null)
         {
           var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
           var config = builder.Build();
     
           var clientId = config["EntraId:ClientId"];
           var clientSecret = config["EntraId:ClientSecret"];
           var tenantId = config["EntraId:TenantId"];
           
           var credential = new ClientSecretCredential(tenantId, clientId,    clientSecret);
           _client = new GraphServiceClient(credential);
         }
   
         return _client;
       }
     }
   }
   ```

1. 保存所做的更改

## 任务 7 - 定义外部连接和架构配置

下一步是定义 Graph 连接器应使用的外部连接和架构。 由于连接器的代码需要在多个位置访问外部连接的 ID，因此将其存储在代码的中心位置。

在代码编辑器中：

1. 创建名为 **ConnectionConfiguration.cs** 的新文件。
1. 使用 Microsoft Graph 模型添加对命名空间的引用：

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. 接下来，在同一文件中，定义一个名为 `ConnectionConfiguration` 的新静态类：

   ```csharp
   static class ConnectionConfiguration
   {
   
   }
   ```

1. 在 `ConnectionConfiguration` 类中，添加名为 `ExternalConnection` 的新属性。 实现它以返回 `ExternalConnection`Microsoft Graph 模型的实例：

   ```csharp
   public static ExternalConnection ExternalConnection
   {
     get
     {
       return new ExternalConnection
       {
         Id = "msgraphdocs",
         Name = "Microsoft Graph documentation",
         Description = "Documentation for Microsoft Graph API which explains what Microsoft Graph is and how to use it."
       };
     }
   }
   ```

1. 接下来，添加另一个名为 `Schema` 的属性。 实现它以返回 `Schema`Graph 模型的实例：

   ```csharp
   public static Schema Schema
   {
     get
     {
       return new Schema
       {
         BaseType = "microsoft.graph.externalItem",
         Properties = new()
         {
           // TODO: implement
         }
       };
     }
   }
   ```

1. 在架构中，为使用连接器引入的每个外部项定义跟踪属性：

   ```csharp
   new Property
   {
     Name = "title",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true,
     Labels = new() { Label.Title }
   },
   new Property
   {
     Name = "description",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true
   },
   new Property
   {
     Name = "iconUrl",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.IconUrl }
   },
   new Property
   {
     Name = "url",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.Url }
   }
   ```

   首先，定义**标题**属性，该属性存储导入到 Microsoft 365 的外部项的标题。 项的标题是全文索引的一部分 (`IsSearchable = true`)。 用户还可以在关键字查询 (`IsQueryable = true`) 中明确查询其内容。 还可以在搜索结果 (`IsRetrievable = true`) 中检索和显示标题。 **标题**属性代表项的标题，可以使用 `Title` 语义标签来表示。

   接下来，定义**说明**属性，该属性存储外部项内容的摘要。 其定义类似于标题。 然而，说明没有语义标签，这就是你没有定义它的原因。

   接下来，定义一个属性来存储每个项的图标的 URL。 适用于 Microsoft 365 的 Copilot 需要此属性，并且需要使用`IconUrl`语义标签对其进行映射。

   最后，定义 **URL** 属性，该属性存储外部项的原始 URL。 用户使用此 URL 从搜索结果或从适用于 Microsoft 365 的 Copilot 导航到外部项。 URL 是适用于 Microsoft 365 的Copilot 所需的属性之一，这就是为什么使用`Url`语义标签对其进行映射的原因。

1. 完整的代码如下所示：

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionConfiguration
   {
     public static ExternalConnection ExternalConnection
     {
       get
       {
         return new ExternalConnection
         {
           Id = "msgraphdocs",
           Name = "Microsoft Graph documentation",
           Description = "Documentation for Microsoft Graph API which    explains what Microsoft Graph is and how to use it."
         };
       }
     }
     public static Schema Schema
     {
       get
       {
         return new Schema
         {
           BaseType = "microsoft.graph.externalItem",
           Properties = new()
           {
             new Property
             {
               Name = "title",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true,
               Labels = new() { Label.Title }
             },
             new Property
             {
               Name = "description",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true
             },
             new Property
             {
               Name = "iconUrl",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.IconUrl }
             },
             new Property
             {
               Name = "url",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.Url }
             }
           }
         };
       }
     }
   }
   ```

1. 保存所做的更改

## 任务 8 - 创建外部连接

继续添加代码，该代码使用上一部分中定义的外部连接信息在 Microsoft 365 中创建外部连接。

在代码编辑器中：

1. 创建名为 **ConnectionService.cs** 的新文件。
1. 在文件中，首先添加对命名空间的引用：

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. 接下来，在同一文件中，定义一个名为`ConnectionService`的新静态类：

   ```csharp
   static class ConnectionService
   {
   
   }
   ```

1. 在`ConnectionService`类中，添加名为`CreateConnection`的新方法：

   ```csharp
   async static Task CreateConnection()
   {

   }
   ```

1. 在 `CreateConnection`方法中，使用 Microsoft Graph 客户端实例调用 Microsoft Graph API，并使用前面定义的连接信息创建外部连接：

   ```csharp
   Console.Write("Creating connection...");
   
   await GraphService.Client.External.Connections
     .PostAsync(ConnectionConfiguration.ExternalConnection);
   
   Console.WriteLine("DONE");
   ```

1. 完整的代码如下所示：

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   }
   ```

1. 保存所做更改。

在测试此代码之前，请添加代码以创建架构。 这样，就可以测试创建和配置外部连接的完整流。

## 任务 9 - 创建外部连接架构

创建外部连接的最后一部分是创建其架构。

在代码编辑器中：

1. 打开 **ConnectionService.cs** 文件。
1. 在 ConnectionService 类中，添加新方法，名为 `CreateSchema`：

   ```csharp
   async static Task CreateSchema()
   {
   }
   ```

1. 在 `CreateSchema` 方法中，使用 Microsoft Graph 客户端实例调用Microsoft Graph API 来创建架构。 然后，等待其创建。

   ```csharp
   Console.WriteLine("Creating schema...");
   
   await GraphService.Client.External
     .Connections[ConnectionConfiguration.ExternalConnection.Id]
     .Schema
     .PatchAsync(ConnectionConfiguration.Schema);
   
   do
   {
     var externalConnection = await GraphService.Client.External
       .Connections[ConnectionConfiguration.ExternalConnection.Id]
       .GetAsync();
   
     Console.Write($"State: {externalConnection?.State.ToString()}");
   
     if (externalConnection?.State != ConnectionState.Draft)
     {
       Console.WriteLine();
       break;
     }
   
     Console.WriteLine($". Waiting 60s...");
   
     await Task.Delay(60_000);
   }
   while (true);
   
   Console.WriteLine("DONE");
   ```

1. 在同一文件中，添加新方法，名为 `ProvisionConnection`。 在其代码中，调用前面定义的 `CreateConnection` 方法和 `CreateSchema` 方法：

   ```csharp
   public static async Task ProvisionConnection()
   {
     try
     {
       await CreateConnection();
       await CreateSchema();
     }
     catch (Exception ex)
     {
       Console.WriteLine(ex.Message);
     }
   }
   ```

1. 完整的代码如下所示：

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   
     async static Task CreateSchema()
     {
       Console.WriteLine("Creating schema...");
   
       await GraphService.Client.External
         .Connections[ConnectionConfiguration.ExternalConnection.Id]
         .Schema
         .PatchAsync(ConnectionConfiguration.Schema);
   
       do
       {
         var externalConnection = await GraphService.Client.External
           .Connections[ConnectionConfiguration.ExternalConnection.Id]
           .GetAsync();
   
         Console.Write($"State: {externalConnection?.State.ToString()}");
   
         if (externalConnection?.State != ConnectionState.Draft)
         {
           Console.WriteLine();
           break;
         }
   
         Console.WriteLine($". Waiting 60s...");
   
         await Task.Delay(60_000);
       }
       while (true);
   
       Console.WriteLine("DONE");
     }
   
     public static async Task ProvisionConnection()
     {
       try
       {
         await CreateConnection();
         await CreateSchema();
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

1. 保存所做更改。

## 任务 10 - 测试代码

最后一步是在应用程序中创建入口点，这将创建连接及其架构。 为此，请通过从命令行启动应用程序来创建调用的命令。

在代码编辑器中：

1. 打开 **Program.cs** 文件。
1. 将其内容替换为以下代码：

   ```csharp
   using System.CommandLine;
   
   var provisionConnectionCommand = new Command("provision-connection",    "Provisions external connection");
   provisionConnectionCommand.SetHandler(ConnectionService.   ProvisionConnection);
   
   var rootCommand = new RootCommand();
   rootCommand.AddCommand(provisionConnectionCommand);
   Environment.Exit(await rootCommand.InvokeAsync(args));
   ```

   首先，定义名为 `provision-connection.` 的命令，此命令将调用前面定义的 `ConnectionService.ProvisionConnection` 方法。 最后，将命令注册到命令行处理器，并启动在命令行中传递的应用程序监视参数。

1. 保存所做的更改

测试应用程序:

1. 打开终端。
1. 将工作目录更改为项目文件夹。
1. 运行 `dotnet build` 以生成项目。
1. 运行 `dotnet run -- provision-connection` 启动该应用。
1. 等待几分钟才能创建连接和架构。

[继续下一个练习……](./3-exercise-import-external-content.md)