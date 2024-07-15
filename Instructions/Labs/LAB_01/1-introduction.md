---
lab:
  title: 简介
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 简介

消息扩展允许用户使用来自 Microsoft Teams 和 Microsoft Outlook 的外部系统。 用户可以使用消息扩展来查找和更改数据，并在消息和电子邮件中以格式丰富的卡片的形式共享来自这些系统的信息。

假设你有一个 SharePoint Online 列表，其中包含当前且与组织相关的产品信息。 想要跨 Microsoft 365 搜索和共享此信息。 还希望适用于 Microsoft 365 的 Copilot 在其答案中使用此信息。

:::image type="content" source="../media/1-sharepoint-online-product-support-site.png" alt-text="产品支持 SharePoint Online 团队网站主页的屏幕截图。会显示最近发布的产品列表。" lightbox="../media/1-sharepoint-online-product-support-site.png":::

在本模块中，将创建消息扩展。 消息扩展使用机器人与 Microsoft Teams、Microsoft Outlook 和适用于 Microsoft 365 的 Copilot 进行通信。

:::image type="content" source="../media/2-search-results-nuget.png" alt-text="Microsoft Teams 中基于搜索的消息扩展返回的搜索结果屏幕截图。" lightbox="../media/2-search-results-nuget.png":::

它使用 Microsoft Entra 对用户进行身份验证，这样就能代表用户使用 Microsoft Graph API 从 SharePoint Online 返回数据。

:::image type="content" source="../media/3-sign-in.png" alt-text="基于搜索的消息扩展中的身份验证质询的屏幕截图。将显示登录链接。" lightbox="../media/3-sign-in.png":::

用户进行身份验证后，消息扩展将使用 Microsoft Graph API 从 SharePoint Online 获取产品信息。 它将返回搜索结果，这些搜索结果可以嵌入到消息和电子邮件中作为格式丰富的卡片，然后共享。

:::image type="content" source="../media/4-search-results-sharepoint-online.png" alt-text="Microsoft Teams 中基于搜索的消息扩展返回的搜索结果的屏幕截图。搜索结果从 SharePoint Online 返回。每个搜索结果显示产品名称、类别和产品图像。" lightbox="../media/4-search-results-sharepoint-online.png":::

:::image type="content" source="../media/5-adaptive-card.png" alt-text="Microsoft Teams 消息中嵌入的搜索结果的屏幕截图。搜索结果将呈现为具有产品名称、类别、呼叫量和发布日期的自适应卡片。会显示标题视图的操作按钮，用户可以使用该按钮导航到 SharePoint Online 中的产品列表项。" lightbox="../media/5-adaptive-card.png":::

它与适用于 Microsoft 365 的 Copilot 作为插件一起使用，使它能够代表用户查询 SharePoint Online 列表，并在其答案中使用返回的数据。

:::image type="content" source="../media/6-copilot-answer.png" alt-text="适用于 Microsoft 365 的 Copilot 中包含消息扩展插件返回的信息的答案的屏幕截图。会显示自适应卡片，显示产品信息。" lightbox="../media/6-copilot-answer.png":::

在本模块结束时，可以创建用 C# 编写的消息扩展（在 .NET 上运行）。 可用于 Microsoft Teams、Microsoft Outlook 和适用于 Microsoft 365 的 Copilot。 可以在受保护的 API 后面查询数据，并将结果作为格式丰富的卡片返回。

## 先决条件

- C# 基础知识
- Bicep 基础知识
- 身份验证基础知识
- 对 Microsoft 365 租户的管理员访问权限。
- 可访问 Azure 订阅
- 适用于 Microsoft 365 的 Copilot 访问权限是可选的，只需完成一个练习
- 安装了 [Teams 工具包](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs) 的 Visual Studio 2022 17.9（Microsoft Teams 开发工具组件）
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

准备好开始时，[继续下一个练习......](./2-exercise-create-a-message-extension.md)
