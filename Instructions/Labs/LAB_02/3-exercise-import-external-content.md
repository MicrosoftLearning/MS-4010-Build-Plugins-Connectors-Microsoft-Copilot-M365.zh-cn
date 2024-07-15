---
lab:
  title: 练习 2 - 导入外部内容
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# 练习 - 导入外部内容

在本练习中，你将使用代码扩展自定义 Microsoft Graph 连接器，以将本地 Markdown 文件导入到 Microsoft 365。

## 准备工作

完成本练习大约需要** XX 分钟**。

## 任务 1 - 下载外部内容

若要完成本练习，请从 [GitHub](https://pnp.github.io/download-partial/?url=https://github.com/pnp/graph-connectors-samples/tree/main/samples/dotnet-csharp-graphdocs/content) 复制本练习中使用的示例内容文件，并将其存储到项目中名为**内容**的文件夹。

:::image type="content" source="../media/6-content-files.png" alt-text="显示了本练习中使用的内容文件的代码编辑器屏幕截图。":::

若要使代码正常运行，必须将**内容**文件夹及其内容复制到生成输出文件夹。

在代码编辑器中：

1. 打开 **.csproj** 文件，在 `</Project>` 标记前添加以下代码：

   ```xml
   <ItemGroup>
     <ContentFiles Include="content\**"    CopyToOutputDirectory="PreserveNewest" />
   </ItemGroup>
   
   <Target Name="CopyContentFolder" AfterTargets="Build">
     <Copy SourceFiles="@(ContentFiles)" DestinationFiles="@   (ContentFiles->'$(OutputPath)\content\%(RecursiveDir)%(Filename)%   (Extension)')" />
   </Target>
   ```

1. 保存所做更改。

## 任务 2 - 添加库以分析 Markdown 和 YAML

生成的 Microsoft Graph 连接器会将本地 Markdown 文件导入到 Microsoft 365。 每个文件都包含一个带有 YAML 格式元数据的标头，也称为 frontmatter。 此外，每个文件的内容都用 Markdown 编写。 若要提取元数据并将正文转换为 HTML，请使用自定义库：

1. 打开终端，将工作目录更改为项目。
1. 若要添加 Markdown 处理库，请运行以下命令：`dotnet add package Markdig`。
1. 若要添加 YAML 处理库，请运行以下命令：`dotnet add package YamlDotNet`。

## 任务 3 - 定义表示导入文件的类

为了简化使用导入的 Markdown 文件及其内容，让我们定义一个具有必要属性的类。

在代码编辑器中：

1. 创建名为 **ContentService.cs** 的新文件。
1. 添加以下代码：

   ```csharp
   using YamlDotNet.Serialization;
   
   public interface IMarkdown
   {
     string? Markdown { get; set; }
   }
   
   class DocsArticle : IMarkdown
   {
     [YamlMember(Alias = "title")]
     public string? Title { get; set; }
     [YamlMember(Alias = "description")]
     public string? Description { get; set; }
     public string? Markdown { get; set; }
     public string? Content { get; set; }
     public string? RelativePath { get; set; }
   }
   ```

   `IMarkdown` 接口表示本地 Markdown 文件的内容。 需要单独定义它，以支持反序列化文件内容。 `DocsArticle` 类，表示具有分析的 YAML 属性和 HTML 内容的最终文档。 `YamlMember` 属性将属性映射到每个文档标头中的元数据。

1. 保存所做更改。

## 任务 4 - 定义 `ContentService` 类

接下来，生成一个类，其中包含用于加载本地 Markdown 文件、将其转换为外部项和将其加载到 Microsoft 365 的代码。

在代码编辑器中：

1. 验证是否正在编辑 **ContentService.cs** 文件。
1. 在该文件的顶部，添加以下 using 语句：

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. 然后，在文件末尾添加以下代码：

   ```csharp
   static class ContentService
   {
     static IEnumerable<DocsArticle> Extract()
     {}
   
     static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
     {}
   
     static async Task Load(IEnumerable<ExternalItem> items)
     {}
   
     public static async Task LoadContent()
     {
       var content = Extract();
       var transformed = Transform(content);
       await Load(transformed);
     }
   }
   ```

   `ContentService` 类定义三个表示内容处理过程的方法：

   1. `Extract` 可以加载本地 Markdown 文件并将其分析为 `DocsArticle` 类的实例，以便于处理。
   1. `Transform` 可以将 `DocsArticle` 对象转换为 `ExternalItems` 类的实例，该实例是 Microsoft Graph .NET SDK 的一部分，表示要加载到 Microsoft 365 的外部项。
   1. `Load` 使用 Microsoft Graph API 将外部项加载到 Microsoft 365。

   这些方法按特定顺序从 `LoadContent` 方法中调用。

1. 保存所做更改。

## 任务 5 - 配置 Markdown 处理

首先，从本地 Markdown 文件中提取内容。

首先，添加帮助程序方法来轻松使用 `Markdig` 和 `YamlDotNet` 库。

在代码编辑器中：

1. 创建名为 **MarkdownExtensions.cs** 的新文件。
1. 在  文件中添加以下代码：

   ```csharp
   // from: https://khalidabuhakmeh.com/parse-markdown-front-matter-with-csharp
   using Markdig;
   using Markdig.Extensions.Yaml;
   using Markdig.Syntax;
   using YamlDotNet.Serialization;
   
   public static class MarkdownExtensions
   {
     private static readonly IDeserializer YamlDeserializer =
         new DeserializerBuilder()
         .IgnoreUnmatchedProperties()
         .Build();
         
     private static readonly MarkdownPipeline Pipeline
         = new MarkdownPipelineBuilder()
         .UseYamlFrontMatter()
         .Build();
   }
   ```

   `YamlDeserializer` 属性为要提取的每个 Markdown 文件中的 YAML 块定义一个新的反序列化程序。 配置反序列化程序，以忽略不属于文件反序列化的类的所有属性。

   `Pipeline` 属性定义 Markdown 分析器的处理管道。 对其进行配置以分析 YAML 标头。 如果没有此配置，标头中的信息将被丢弃。

1. 接下来，使用以下代码扩展 `MarkdownExtensions` 类：

   ```csharp
   public static T GetContents<T>(this string markdown) where T :    IMarkdown, new()
   {
     var document = Markdown.Parse(markdown, Pipeline);
     var block = document
         .Descendants<YamlFrontMatterBlock>()
         .FirstOrDefault();
   
     if (block == null)
       return new T { Markdown = markdown };
   
     var yaml =
         block
         // this is not a mistake
         // we have to call .Lines 2x
         .Lines // StringLineGroup[]
         .Lines // StringLine[]
         .OrderByDescending(x => x.Line)
         .Select(x => $"{x}\n")
         .ToList()
         .Select(x => x.Replace("---", string.Empty))
         .Where(x => !string.IsNullOrWhiteSpace(x))
         .Aggregate((s, agg) => agg + s);
   
     var t = YamlDeserializer.Deserialize<T>(yaml);
     t.Markdown = markdown.Substring(block.Span.End + 1);
     return t;
   }
   ```

   `GetContents` 方法将标头中带有 YAML 元数据的 Markdown 字符串转换为实现 `IMarkdown` 接口的指定类型。 从 Markdown 字符串中提取 YAML 标头并将其反序列化为指定类型。 然后，它会提取文章正文，并将其设置为 `Markdown` 属性，以便进一步处理。

1. 保存所做更改。

## 任务 6 - 提取 Markdown 和 YAML 内容

帮助程序方法就绪后，实现 Extract 方法以加载本地 Markdown 文件并从中提取信息。

在代码编辑器中：

1. 打开 **ContentService.cs** 文件。
1. 在该文件的顶部，添加以下 using 语句：

   ```csharp
   using Markdig;
   ```

1. 接下来，在 `ContentService` 类中，使用以下代码实现 `Extract` 方法：

   ```csharp
   static IEnumerable<DocsArticle> Extract()
   {
     var docs = new List<DocsArticle>();
   
     var contentFolder = "content";
     var contentFolderPath = Path.Combine(Directory.GetCurrentDirectory(),    contentFolder);
     var files = Directory.GetFiles(contentFolder, "*.md", SearchOption.   AllDirectories);
   
     foreach (var file in files)
     {
       try
       {
         var contents = File.ReadAllText(file);
         var doc = contents.GetContents<DocsArticle>();
         doc.Content = Markdown.ToHtml(doc.Markdown ?? "");
         doc.RelativePath = Path.GetRelativePath(contentFolderPath, file);
         docs.Add(doc);
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   
     return docs;
   }
   ```

   该方法首先从**内容**文件夹中加载 Markdown 文件。 对于每个文件，它会将其内容作为字符串加载。 它使用前面在 `MarkdownExtensions` 类中定义的 `GetContents` 扩展方法将字符串转换为对象，并将元数据和内容存储在单独属性中。 接下来，它将 Markdown 字符串转换为 HTML。 最后，它将存储文件的相对路径，并将对象添加到集合以供进一步处理。

1. 保存所做更改。

## 任务 7 - 将内容转换为外部项

读取外部内容后，下一步是将其转换为外部项，并加载到 Microsoft 365。

首先，添加一个帮助程序方法，该方法基于其相对文件路径为每个外部项生成唯一 ID。

在代码编辑器中：

1. 确认正在编辑 **ContentService.cs** 文件。
1. 在 `ContentService` 类中，添加以下方法：

   ```csharp
   static string GetDocId(string relativePath)
   {
     var id = relativePath.Replace(Path.DirectorySeparatorChar.ToString(),    "__").Replace(".md", "");
     return id;
   }
   ```

   `GetDocId` 方法采用相对文件路径，并将所有目录分隔符替换为双下划线。 这是必要的，因为路径分隔符不能用于外部项 ID。

1. 保存所做更改。

现在，实现 `Transform` 方法，将表示本地 Markdown 文件的对象从 Microsoft Graph 转换为外部项。

在代码编辑器中：

1. 确认你位于 **ContentService.cs** 文件中。
1. 使用以下代码实现 `Transform` 方法：

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var docId = GetDocId(a.RelativePath ?? '');
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
               { "title", a.Title ?? "" },
               { "description", a.Description ?? "" },
               { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md",    "")).ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
             new()
             {
               Type = AclType.Everyone,
               Value = "everyone",
               AccessType = AccessType.Grant
             }
         }
       };
     });
   }
   ```

   首先，定义基 URL。 使用此 URL 为每个项生成完整的 URL，以便当该项显示给用户时，他们可以导航到原始项。 接下来，将每个项从 `DocsArticle` 转换为 `ExternalItem`。 首先，根据每个项的相对文件路径获取其唯一 ID。 然后，创建一个新的 `ExternalItem` 实例，并使用来自 `DocsArticle` 的信息填充其属性。 然后，将项的内容设置为从本地文件中提取的 HTML 内容，并将项目内容类型设置为 HTML。 最后，配置项目的权限，使其对组织中的每个人可见。

1. 保存所做更改。

## 任务 8 - 将外部项加载到 Microsoft 365

处理内容的最后一步是将转换的外部项加载到 Microsoft 365。

在代码编辑器中：

1. 验证是否正在编辑 **ContentService.cs** 文件。
1. 在 `ContentService` 类中，使用以下代码实现 `Load` 方法：

   ```csharp
   static async Task Load(IEnumerable<ExternalItem> items)
   {
     foreach (var item in items)
     {
       Console.Write(string.Format("Loading item {0}...", item.Id));
   
       try
       {
         await GraphService.Client.External
           .Connections[Uri.EscapeDataString(ConnectionConfiguration.   ExternalConnection.Id!)]
           .Items[item.Id]
           .PutAsync(item);
   
         Console.WriteLine("DONE");
       }
       catch (Exception ex)
       {
         Console.WriteLine("ERROR");
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

   对于每个外部项，请使用 Microsoft Graph .NET SDK 调用 Microsoft Graph API 并上传该项。 在请求中，指定此前创建的外部连接的 ID、要上传的项目 ID 和完整的项目内容。

1. 保存所做更改。

## 任务 9 - 添加内容加载命令

在测试代码之前，需要使用调用内容加载逻辑的命令扩展控制台应用程序。

在代码编辑器中：

1. 打开 **Program.cs** 文件。
1. 使用以下代码添加一个新命令来加载内容：

    ```csharp
    var loadContentCommand = new Command("load-content", "Loads content   into the external connection");
    loadContentCommand.SetHandler(ContentService.LoadContent);
    ```

1. 使用根命令注册新定义的命令，以便可以使用以下代码调用它：

     ```csharp
     rootCommand.AddCommand(loadContentCommand);
     ```

1. 保存所做更改。

## 任务 10 - 测试代码

最后一件事是测试 Microsoft Graph 连接器是否正确导入外部内容。

1. 打开终端。
1. 将工作目录更改为项目。
1. 通过运行 `dotnet build` 命令生成项目。
1. 通过运行 `dotnet run -- load-content` 命令开始加载内容。
1. 等待命令完成并加载内容。

[继续下一个练习...](./4-exercise-ensure-secure-access.md)