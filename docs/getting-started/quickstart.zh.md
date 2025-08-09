# 快速入门指南

欢迎来到 LeanExplore 快速入门指南！本页面将引导您完成使用 LeanExplore 强大的远程 API 和命令行界面 (CLI) 启动和运行的基本第一步。我们的目标是帮助您快速执行第一个有意义的操作。

## 先决条件：安装

在开始之前，请确保您已安装 LeanExplore。如果没有，您可以使用 pip 安装它：

```bash
pip install lean-explore
```

本指南专注于使用 LeanExplore API，这是许多命令的默认模式，一旦配置了 API 密钥，就提供了访问搜索和 AI 功能的零设置方式。

## 步骤 1：获取您的 LeanExplore API 密钥

要与 LeanExplore API 交互，您需要一个个人 API 密钥。此密钥验证您对我们服务器的请求。如果您没有，请访问 [https://www.leanexplore.com/api-keys](https://www.leanexplore.com/api-keys) 注册或登录并获取您的 API 密钥。

## 步骤 2：配置您的 LeanExplore API 密钥

一旦您有了 API 密钥，下一步就是使用 LeanExplore CLI 配置它。这将安全地保存您的密钥以供将来使用，因此您不必每次都输入它。

在终端中运行以下命令：

```bash
leanexplore configure api-key
```

系统将提示您粘贴您的 API 密钥。这是一次性设置。如果您已经完成了此操作，可以继续下一步。

## 步骤 3：执行您的第一次搜索

配置了 API 密钥后，您就可以直接从命令行执行第一次搜索了。让我们尝试搜索一个著名的定理：

```bash
leanexplore search "fundamental theorem of calculus"
```

您应该看到与您的查询匹配的相关 Lean 语句列表，以及源文件、行号和代码片段等详细信息。这演示了通过其 API 使用 LeanExplore 的直接搜索功能。

## 步骤 4：尝试 AI 聊天（通过 API）

LeanExplore 还提供 AI 驱动的聊天助手，以更交互的方式探索 Lean 代码。默认情况下，如果配置了您的 LeanExplore API 密钥，聊天将使用 API 后端。对于 AI 功能，您还需要一个 OpenAI API 密钥。

### 4a. 配置 OpenAI API 密钥（如果需要）

如果您尚未使用 LeanExplore 设置 OpenAI API 密钥，请运行：

```bash
leanexplore configure openai-key
```

按照提示操作。如果此密钥已配置，您可以跳过此子步骤。

### 4b. 启动聊天会话

现在，启动 AI 聊天助手：

```bash
leanexplore chat
```

由于您的 LeanExplore API 密钥已配置且 `--backend api` 是默认设置，助手将使用远程 API 获取其 Lean 特定知识。

### 4c. 提出问题

聊天界面加载并且助手准备就绪后，尝试要求它查找特定定义及其上下文，例如：

```
You: Find the formal statement for the 'fundamental theorem of calculus'
and tell me about its main dependencies.
```

## 步骤 5：观察结果

AI 助手将处理您的查询。它使用 LeanExplore API 搜索与"微积分基本定理"相关的形式陈述，识别最相关的陈述，然后查找并列出它们的主要依赖关系。您应该期待一个对话式响应，其中包括定理的 Lean 代码，以及解释和其他上下文信息，以帮助您理解其结构和在库中的连接。

## 恭喜 & 下一步

您现在已经成功使用 LeanExplore 执行了直接 API 搜索并与 AI 聊天助手交互！这些是利用 LeanExplore 功能的两种主要方式。

从这里，您可以：

* 在[使用 CLI](../cli/usage.md) 指南中深入了解所有命令行选项。
* 在[执行搜索](../cli/search.md)部分了解使用本地数据和编程访问（本地和 API）。
* 如果您有兴趣构建自定义 AI 代理集成，请探索 [MCP (AI 代理)](../mcp/agents.md) 文档。