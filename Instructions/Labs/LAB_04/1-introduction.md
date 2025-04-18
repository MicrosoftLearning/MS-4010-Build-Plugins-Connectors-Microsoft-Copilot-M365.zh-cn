---
lab:
  title: 简介
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# 简介

Microsoft 365 Copilot 代理允许创建针对特定应用场景优化的 AI 支持的助手。 使用说明，定义 Copilot 的上下文，并配置其语音的音调或响应方式。 通过配置代理的知识，可以授予它对不属于大型语言模型 (LLM) 的外部数据的访问权限，以便它可以更准确地响应。 

## 示例方案

假设你在大型组织的 IT 部门工作。 组织通过存储在专用系统中的不同策略来标准化 IT。 你和 IT 部门的同事会定期收到策略中涵盖的问题。 在策略管理系统中查找答案非常耗时。 你希望为组织提供一个 AI 支持的助手，能够使用策略中的权威信息回答同事的问题。

## 学习目标

在本模块结束时，你将能够为智能 Microsoft 365 Copilot 副驾驶® 生成声明性代理。 你将了解如何配置其说明以针对特定应用场景对其进行优化。 你还将了解如何将它们与 Microsoft Graph 连接器集成，以便它们能够访问外部数据，这不是 Microsoft 365 Copilot LLM 的一部分。

## 先决条件

- 了解什么是智能 Microsoft 365 Copilot 副驾驶® 及其在初学者级别的工作原理
- 了解如何构建智能 Microsoft 365 Copilot 副驾驶® 声明性代理
- 了解如何生成 Graph 连接器
- Microsoft 365 租户，具有智能 Microsoft 365 Copilot 副驾驶® 和租户管理员权限
- 安装了 [Teams 工具包](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension)扩展的 [Visual Studio Code](https://code.visualstudio.com/)
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)
- [Node.js v18](https://nodejs.org/)
