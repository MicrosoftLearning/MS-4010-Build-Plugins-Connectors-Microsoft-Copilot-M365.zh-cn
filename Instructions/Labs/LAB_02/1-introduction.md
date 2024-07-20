---
lab:
  title: 简介
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# 简介

假设你有一个存储知识库文章的外部系统。 这些文章包含组织中不同流程的信息。 你希望能够从 Microsoft 365 中轻松查找和发现相关信息。 你还希望适用于 Microsoft 365 的 Copilot 在其响应中包含来自这些知识库文章的信息。

若要在 Microsoft 365 中公开此外部信息，你将生成自定义 Microsoft Graph 连接器。 Microsoft Graph 连接器连接到外部系统（1）以检索内容，使用来自 Microsoft Entra ID 的信息通过 Microsoft 365 进行身份验证，（2）并使用 Microsoft Graph API 将内容导入到 Microsoft 365。

:::image type="content" source="../media/1-graph-connector-concept.png" alt-text="显示 Microsoft Graph 连接器的概念工作的示意图。":::

在本模块中，你了解了什么是 Microsoft Graph 连接器，以及为何应考虑在组织中使用这些连接器。 你生成了 Microsoft Graph 连接器，它可以将本地 Markdown 文件导入到 Microsoft 365 中。 你还了解了如何确保只有具有适当分配权限的个人才能访问导入的外部内容。 最后，优化 Microsoft Graph 连接器，以便与适用于 Microsoft 365 的 Copilot 一起使用。

## 先决条件

- C# 基础知识
- 身份验证基础知识
- [Microsoft 365 开发人员租户](https://developer.microsoft.com/microsoft-365/dev-program?ocid=MSlearn)的访问权限
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

准备好开始时，[继续下一个练习...](./2-exercise-configure-connection-schema.md)