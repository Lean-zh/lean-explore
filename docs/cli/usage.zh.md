# 使用 LeanExplore CLI

LeanExplore 命令行界面 (CLI) 通过 `leanexplore` 命令调用，是您与 LeanExplore 生态系统交互的主要工具。它使您能够配置设置、管理本地数据、直接搜索 LeanExplore API、使用 AI 助手探索代码，以及控制模型上下文协议 (MCP) 服务器以进行更高级的集成。

## 目录

- [使用 LeanExplore CLI](#使用-leanexplore-cli)
  - [目录](#目录)
  - [理解 CLI 基础](#理解-cli-基础)
  - [入门：初始设置](#入门初始设置)
    - [配置您的 API 密钥](#配置您的-api-密钥)
    - [（可选）准备本地数据探索](#可选准备本地数据探索)
  - [查找和检查 Lean 代码（通过 API）](#查找和检查-lean-代码通过-api)
    - [搜索 Lean 语句](#搜索-lean-语句)
    - [查看详细信息](#查看详细信息)
    - [探索代码依赖关系](#探索代码依赖关系)
  - [使用 AI 聊天助手进行交互式探索](#使用-ai-聊天助手进行交互式探索)
    - [启动聊天会话](#启动聊天会话)
    - [关键聊天选项](#关键聊天选项)
  - [通过 MCP 服务器与 AI 代理集成](#通过-mcp-服务器与-ai-代理集成)
    - [运行服务器](#运行服务器)
    - [关键服务器选项](#关键服务器选项)

## 理解 CLI 基础

大多数 CLI 命令遵循结构：`leanexplore [OPTIONS] COMMAND [ARGS]...`。要在任何时候获得帮助，无论是主命令还是特定子命令，只需附加 `--help`：

```bash
leanexplore --help
```

```bash
leanexplore configure --help
```

```bash
leanexplore data fetch --help
```

## 入门：初始设置

在深入了解 LeanExplore 的所有功能之前，一些初始设置步骤可以确保一切顺利运行，特别是在与在线服务或 AI 助手交互时。

### 配置您的 API 密钥

LeanExplore 使用 API 密钥来验证对某些服务的访问。一旦设置，这些密钥将安全地存储在本地配置文件中。

**LeanExplore API 密钥：** 此密钥对于与远程 LeanExplore API 通信的功能至关重要，例如直接搜索（`search`、`get`、`dependencies` 命令）和使用具有默认 API 后端的 AI 聊天。

您可以从 <https://www.leanexplore.com/api-keys> 获取您的 LeanExplore API 密钥。一旦您有了它，通过运行以下命令配置它：

```bash
leanexplore configure api-key
```

系统将提示您输入密钥，通常只需设置一次。

**OpenAI API 密钥：** 如果您计划使用 AI 驱动的聊天功能（`leanexplore chat`），您还需要一个 OpenAI API 密钥。使用以下命令配置它：

```bash
leanexplore configure openai-key
```

按照提示输入您的 OpenAI 密钥。

### （可选）准备本地数据探索

对于喜欢离线工作或对数据集有直接控制的用户，LeanExplore 支持本地数据工具链。这涉及将必要的数据资产下载到您的机器上。

要下载和设置主要本地数据工具链，请使用命令：

```bash
leanexplore data fetch
```

此命令获取 SQLite 数据库（包含 Lean 项目信息）、用于语义搜索的 FAISS 搜索索引以及相关的映射文件。这些安装到本地目录中，通常在 `~/.lean_explore/data/toolchains/` 内。

**注意：** 数据工具链可能有几个 GB。根据您的互联网连接，初始下载可能需要一些时间。此步骤是使用依赖本地后端的功能的先决条件，例如 `leanexplore chat --backend local` 或 `leanexplore mcp serve --backend local`。

## 查找和检查 Lean 代码（通过 API）

一旦配置了 LeanExplore API 密钥，您可以直接查询远程 API 来查找和检查 Lean 语句。这些命令提供对 LeanExplore 索引数据的快速访问。

### 搜索 Lean 语句

要基于自然语言查询搜索 Lean 语句组，请使用 `leanexplore search` 命令：

```bash
leanexplore search "your query string here" [OPTIONS]
```

例如，要查找与"微积分基本定理"相关的语句，限制结果，并按"Mathlib"包过滤：

```bash
leanexplore search "fundamental theorem of calculus" --package Mathlib --limit 3
```

**搜索的关键选项：**

* `QUERY_STRING`：您的搜索词。如果包含空格，请用引号括起来。
* `--package TEXT`（或 `-p TEXT`）：按特定包名过滤结果（例如，`Mathlib`、`Std`）。此选项可以多次使用以包含多个包。
* `--limit INTEGER`（或 `-n INTEGER`）：指定要显示的最大搜索结果数。默认为 5。

该命令将显示匹配的语句组列表，包括它们的 ID、Lean 名称、源文件位置以及相关的代码或文档字符串片段。

### 查看详细信息

如果您有特定的语句组 ID（通常从搜索结果中获得），您可以使用 `leanexplore get` 检索其完整详细信息：

```bash
leanexplore get <GROUP_ID>
```

例如：

```bash
leanexplore get 12345
```

这显示了组的全面信息，例如其完整的语句文本、文档字符串和任何非形式描述，通常以易于阅读的格式化面板呈现。

### 探索代码依赖关系

要了解语句组如何连接到其他语句组，您可以使用 `leanexplore dependencies` 获取其直接依赖关系（它引用的项目）：

```bash
leanexplore dependencies <GROUP_ID>
```

示例：

```bash
leanexplore dependencies 12345
```

此命令列出指定组依赖的语句组，通常以表格格式显示每个依赖项的 ID、Lean 名称和源位置。

## 使用 AI 聊天助手进行交互式探索

LeanExplore 提供 AI 驱动的聊天助手（`leanexplore chat`），以对话方式搜索、理解和探索 Lean 代码。

**先决条件：**

* 必须配置您的 OpenAI API 密钥（使用 `leanexplore configure openai-key`）。
* 对于默认 API 后端：必须配置您的 LeanExplore API 密钥。
* 对于本地后端：必须获取您的本地数据工具链（使用 `leanexplore data fetch`）。

### 启动聊天会话

要使用默认 API 后端启动聊天会话（推荐给大多数用户，提供对最新数据的访问）：

```bash
leanexplore chat
```

如果您已设置本地数据并希望使用它（例如，用于离线访问）：

```bash
leanexplore chat --backend local
```

### 关键聊天选项

* `--backend {api|local}`（别名：`-lb`）：指定 AI 工具的数据源。
  + `api`（默认）：代理查询远程 LeanExplore API。
  + `local`：代理使用您下载的本地数据。
* `--lean-api-key TEXT`：（可选）如果使用 API 后端，您可以为当前会话提供 LeanExplore API 密钥，覆盖任何配置的密钥。
* `--debug`：为聊天客户端和它管理的底层 MCP 服务器启用详细的调试日志记录。这对故障排除很有用。

在聊天中，您可以要求助手执行诸如"在 Mathlib 中查找'monoid'的定义"或"显示 `Nat.add` 的依赖关系"等任务。

## 通过 MCP 服务器与 AI 代理集成

这是一个高级功能，适用于旨在将 LeanExplore 的搜索和检索功能作为工具集成到他们自己的自定义 AI 代理应用程序或其他编程设置中的开发人员。

`leanexplore mcp serve` 命令启动 LeanExplore 模型上下文协议 (MCP) 服务器。此服务器通过标准输入/输出 (stdio) 使用 JSON-RPC 2.0 进行通信，并将 LeanExplore 的功能公开为兼容 MCP 的代理客户端可以调用的"工具"。

### 运行服务器

要使用远程 API 后端提供工具（确保配置了 LeanExplore API 密钥或使用 `--api-key` 提供）：

```bash
leanexplore mcp serve --backend api
```

要使用您的本地数据后端提供工具（确保已获取本地数据）：

```bash
leanexplore mcp serve --backend local
```

### 关键服务器选项

* `--backend {api|local}`（别名：`-b`）：确定服务器的工具是否将使用远程 API 或本地数据。默认为 `api`。
* `--api-key TEXT`：（如果 `--backend api` 且未配置密钥则必需）直接为服务器提供要使用的 LeanExplore API 密钥。

**注意：** `leanexplore chat` 命令在内部维护了一个 MCP 服务器的实例。直接运行 `leanexplore mcp serve` 通常用于连接不同的 MCP 客户端或代理框架的场景。

通过熟悉这些命令和工作流程，您可以有效地利用 LeanExplore CLI 在 Lean 4 中进行数学探索和开发。