---
lab:
  title: 练习 3 - 确保使用访问控制列表进行安全访问
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# 练习 3 - 确保使用访问控制列表进行安全访问

在本练习中，你将更新负责导入本地 markdowns 文件的代码，并在所选项目上配置 ACL。

## 准备工作

完成本练习大约需要** XX 分钟**。

## 任务 1 - 导入组织中每个人可用的内容

在上一练习中实现导入外部内容的代码时，已将其配置为可供组织中的所有人使用。 下面是你使用的代码：

```csharp
static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
{
  var baseUrl = new Uri("https://learn.microsoft.com/graph/");

  return content.Select(a =>
  {
    var docId = GetDocId(a.RelativePath ?? "");

    return new ExternalItem
    {
      Id = docId,
      Properties = new()
      {
        AdditionalData = new Dictionary<string, object> {
            { "title", a.Title ?? "" },
            { "description", a.Description ?? "" },
            { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).ToString() }
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

你已将 ACL 配置为向所有人授予访问权限。 让我们调整一下要导入的选定 Markdown 页面。

## 任务 2 - 导入可用于选择用户的内容

首先，配置要导入的页面之一，以便只能访问特定用户。

在 Web 浏览器中：

1. 导航到位于 [https://portal.azure.com](https://portal.azure.com) 的 Azure 门户并使用你的工作或学校帐户登录。
1. 从边栏中，选择“Microsoft Entra ID”****。
1. 从导航中选择“用户”****。
1. 从用户列表中，通过选择其名称打开其中一个用户。
1. 复制**对象 ID** 属性的值。

   :::image type="content" source="../media/8-user.png" alt-text="打开用户配置文件的 Azure 门户屏幕截图。":::

使用此值为特定 Markdown 页面定义新的 ACL。

在代码编辑器中：

1. 打开 **ContentService.cs** 文件，查找 `Transform` 方法。
1. 在 `Select` 委托中，定义适用于所有导入项的默认 ACL：

   ```csharp
   var acl = new Acl
   {
     Type = AclType.Everyone,
     Value = "everyone",
     AccessType = AccessType.Grant
   };
   ```

1. 接下来，用名称以 `use-the-api.md` 结尾 的 markdown 文件重写默认 ACL：

   ```csharp
   if (a.RelativePath!.EndsWith("use-the-api.md"))
   {
     acl = new()
     {
       Type = AclType.User,
       // AdeleV
       Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
       AccessType = AccessType.Grant
     };
   }
   ```

1. 最后，更新返回外部项的代码以使用定义的 ACL：

   ```csharp
   return new ExternalItem
   {
     Id = docId,
     Properties = new()
     {
       AdditionalData = new Dictionary<string, object> {
         { "title", a.Title ?? "" },
         { "description", a.Description ?? "" },
         { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).   ToString() }
       }
     },
     Content = new()
     {
       Value = a.Content ?? "",
       Type = ExternalItemContentType.Html
     },
     Acl = new()
     {
       acl
     }
   };
   ```

1. 更新的 `Transform` 方法如下所示：

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var acl = new Acl
       {
         Type = AclType.Everyone,
         Value = "everyone",
         AccessType = AccessType.Grant
       };
   
       if (a.RelativePath!.EndsWith("use-the-api.md"))
       {
         acl = new()
         {
           Type = AclType.User,
           // AdeleV
           Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
           AccessType = AccessType.Grant
         };
       }
   
       var docId = GetDocId(a.RelativePath ?? "");
   
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
             { "title", a.Title ?? "" },
             { "description", a.Description ?? "" },
             { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).   ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
           acl
         }
       };
     });
   }
   ```

1. 保存所做更改。

## 任务 3 - 导入可用于选择组的内容

现在，让我们扩展代码，以便另一个页面只能由一组选定的用户访问。

在 Web 浏览器中：

1. 导航到位于 [https://portal.azure.com](https://portal.azure.com) 的 Azure 门户并使用你的工作或学校帐户登录。
1. 从边栏中，选择“Microsoft Entra ID”****。
1. 从导航中选择**组**。
1. 从组列表中，通过选择组名称打开其中一个组。
1. 复制**对象 ID** 属性的值。

:::image type="content" source="../media/8-group.png" alt-text="打开组页的 Azure 门户屏幕截图。":::

使用此值为特定 Markdown 页面定义新的 ACL。

在代码编辑器中：

1. 打开 **ContentService.cs** 文件，查找 `Transform` 方法
1. 扩展以前定义的 `if` 子句，并附加条件，用于为名称以 `traverse-the-graph.md` 结尾的 markdown 文件定义 ACL：

   ```csharp
   if (a.RelativePath!.EndsWith("use-the-api.md"))
   {
     acl = new()
     {
       Type = AclType.User,
       // AdeleV
       Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
       AccessType = AccessType.Grant
     };
   }
   else if (a.RelativePath.EndsWith("traverse-the-graph.md"))
   {
     acl = new()
     {
       Type = AclType.Group,
       // Sales and marketing
       Value = "a9fd282f-4634-4cba-9dd4-631a2ee83cd3",
       AccessType = AccessType.Grant
     };
   }
   ```

1. 更新的 `Transform` 方法如下所示：

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var acl = new Acl
       {
         Type = AclType.Everyone,
         Value = "everyone",
         AccessType = AccessType.Grant
       };
   
       if (a.RelativePath!.EndsWith("use-the-api.md"))
       {
         acl = new()
         {
           Type = AclType.User,
           // AdeleV
           Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
           AccessType = AccessType.Grant
         };
       }
       else if (a.RelativePath.EndsWith("traverse-the-graph.md"))
       {
         acl = new()
         {
           Type = AclType.Group,
           // Sales and marketing
           Value = "a9fd282f-4634-4cba-9dd4-631a2ee83cd3",
           AccessType = AccessType.Grant
         };
       }
   
       var docId = GetDocId(a.RelativePath ?? "");
   
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
             acl
         }
       };
     });
   }
   ```

1. 保存所做更改。

## 任务 4 - 应用新的 ACL

最后一步是应用新配置的 ACL。

1. 打开终端，将工作目录更改为项目。
1. 通过运行 `dotnet build` 命令生成项目。
1. 通过运行 `dotnet run -- load-content` 命令开始加载内容。

[继续下一个练习...](./5-exercise-enable-inline-results.md)