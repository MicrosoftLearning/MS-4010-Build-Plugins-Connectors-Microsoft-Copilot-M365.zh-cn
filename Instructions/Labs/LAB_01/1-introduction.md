---
lab:
  title: 简介
  module: 'LAB 01: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# 介绍

消息扩展允许用户使用来自 Microsoft Teams 和 Microsoft Outlook 的外部系统。 用户可以使用消息扩展来查找和更改数据，并在消息和电子邮件中以格式丰富的卡片形式共享来自这些系统的数据。

假设你有一个自定义 API，用于访问与组织相关的最新产品信息。 想要跨 Microsoft 365 搜索和共享此信息。 还希望适用于 Microsoft 365 的 Copilot 在其答案中使用此信息。

在本模块中，将创建消息扩展。 消息扩展使用机器人与 Microsoft Teams、Microsoft Outlook 和适用于 Microsoft 365 的 Copilot 进行通信。

![Microsoft Teams 中基于搜索的消息扩展返回的搜索结果屏幕截图。](../media/1-search-results.png)

它使用 Microsoft Entra 对用户进行身份验证，这样就能代表用户从 API 返回数据。

用户进行身份验证后，消息扩展将从 API 获取数据，并返回搜索结果，这些搜索结果可以作为格式丰富的卡片嵌入到消息和电子邮件中，然后进行共享。

![Microsoft Teams 中使用外部 API 中数据的搜索结果的屏幕截图。](../media/3-search-results-api.png)

![Microsoft Teams 中嵌入消息中的搜索结果的屏幕截图。](../media/4-adaptive-card.png)

它与适用于 Microsoft 365 的 Copilot 作为插件一起使用，使它能够代表用户查询产品数据，并在其答案中使用返回的数据。

![适用于 Microsoft 365 的 Copilot 中答案的屏幕截图，其中包含消息扩展插件返回的信息。 显示产品信息的自适应卡片。](../media/5-copilot-answer.png)

在本模块结束时，可以创建用 C# 编写的消息扩展（在 .NET 上运行）。 可用于 Microsoft Teams、Microsoft Outlook 和适用于 Microsoft 365 的 Copilot。 可以在受保护的 API 后面查询数据，并将结果作为格式丰富的卡片返回。

## 先决条件

- C# 基础知识
- Bicep 基础知识
- 身份验证基础知识
- 对 Microsoft 365 租户的管理员访问权限。
- 可访问 Azure 订阅
- 适用于 Microsoft 365 的 Copilot 访问权限是可选的，只需完成**练习 4：任务 5**即可。
- 安装了 [Teams 工具包](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs) 的 Visual Studio 2022 17.10+（Microsoft Teams 开发工具组件）
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)
- [Dev Proxy 0.19.1+](https://aka.ms/devproxy)

> [!NOTE]
> 此实验室中唯一需要 Microsoft 365 Copilot 许可证的练习是**练习 4：任务 5**。 无论租户是否具有 Copilot，都应完成该时间点之前的一切内容。

## 实验室用时

  - **估计完成时间：** 150 分钟

## 学习目标

学完本模块后，你应该能够：

- 了解什么是消息扩展以及如何生成消息扩展。
- 创建一个消息扩展。
- 了解如何使用单一登录对用户进行身份验证，并调用受 Microsoft Entra 身份验证保护的自定义 API。
- 了解如何扩展和优化消息扩展以与 Copilot for Microsoft 365 一起使用。

准备好开始时，[继续第一个练习...](./2-exercise-create-a-message-extension.md)
