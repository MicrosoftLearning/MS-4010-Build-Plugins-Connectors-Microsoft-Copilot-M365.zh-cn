---
lab:
  title: 简介
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft 365 Copilot'
---

# 介绍

在此项目中，你将了解在 Microsoft 365 Copilot 中如何将 Teams 消息扩展用作插件。 该项目基于同一 GitHub 存储库[](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/tree/main/samples/msgext-northwind-inventory-ts)中包含的“Northwind Inventory”示例。 通过使用成熟的 [Northwind 数据库](https://learn.microsoft.com/dotnet/framework/data/adonet/sql/linq/downloading-sample-databases)，你将拥有大量的模拟企业数据可供使用。

Northwind 在华盛顿州斯波坎市经营一家特色食品电子商务公司。 在本实验室中，你将使用 Northwind Inventory 应用程序，该应用程序提供对产品库存和财务信息的访问权限。

完成此练习大约需要 60 分钟。

## 准备工作

- 首先通过设置开发环境并让应用程序运行来[**做好准备**](./2-prepare-development-environment.md)。

- 在练习 1 [****](./3-exercise-1-run-message-extension.md)中，你将在 Microsoft Teams 和 Outlook 中运行与 消息扩展[](https://learn.microsoft.com/microsoftteams/platform/messaging-extensions/what-are-messaging-extensions)相同的应用程序。

- 在[**练习 2**](./4-exercise-2-run-copilot-plugin.md) 中，将把应用程序作为 Microsoft 365 Copilot 的插件运行。 你将尝试各种提示，并观察如何使用不同的参数调用插件。 与 Copilot 聊天时，可以观看开发人员控制台来查看其生成的查询。

- 在练习 3[****](./5-exercise-3-add-new-command.md) 中，你将了解如何向应用程序添加新命令，以便扩展插件功能并执行更多任务。

  ![显示产品的自适应卡片的屏幕截图。](../media/1-00-product-card-only.png)

- 最后，在练习 4 中[****](./6-exercise-4-explore-plugin-source-code.md)，你将浏览代码，以更深入地了解其工作原理。 如果还没有 Copilot，其他所有内容仍将用作 Microsoft 365 的消息扩展。

准备好开始时，[继续下一个练习...](./2-prepare-development-environment.md)