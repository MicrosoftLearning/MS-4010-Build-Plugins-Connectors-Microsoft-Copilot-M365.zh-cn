---
lab:
  title: 简介
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# 简介

声明性代理可用于扩展 Microsoft 365 Copilot。 定义自定义知识，以创建可以使用权威内容回答问题的代理。

## 示例方案

假设你在客户支持团队工作。 你和你的团队处理有关组织从客户生产的产品的查询。 你想要改进响应时间并提供更好的体验。 你将文档存储在 SharePoint Online 网站上的文档库中，其中包含产品规格、常见问题，以及用于处理维修、退货和保修的各种策略。 你希望能够使用自然语言来查询这些文档中的信息，并快速获取客户查询的答案。

## 我们将执行哪些操作？

在这里，将创建一个声明性代理，该代理可以使用存储在 Microsoft 365 的文档中的信息来回答产品支持问题：

- **创建**：在 Visual Studio Code 中创建声明性代理项目，并使用 Teams 工具包。
- **自定义指令**：通过定义自定义指令来形成响应。
- **自定义上下文关联**：通过配置上下文关联数据，向代理添加额外的上下文。
- **对话开场白**：定义启动新对话的提示。
- **预配**：将声明性代理上传到 Microsoft 365 Copilot 并验证结果。

## 先决条件

- Microsoft 365 Copilot 的基本知识及其工作原理
- Microsoft 365 Copilot 的声明性代理的基础知识
- 使用 Microsoft 365 Copilot 的 Microsoft 365 租户
- 有权将自定义应用上传到 Microsoft Teams 的帐户
- 使用 Microsoft 365 Copilot 访问 Microsoft 365 租户
- 安装了 [Teams 工具包](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension)扩展的 [Visual Studio Code](https://code.visualstudio.com/)
- [Node.js v18](https://nodejs.org/en/download/package-manager)

## 实验室用时

- **估计完成时间：** 30 分钟

## 学习目标

在本模块结束时，可以创建声明性代理，将其上传到 Microsoft 365，然后在 Microsoft 365 Copilot 中使用它来验证结果。

准备好开始时，[继续第一个练习...](./2-exercise-create-declarative-agent.md)
