# 用于 AI 代理的 LeanExplore MCP 服务器

LeanExplore 包含一个模型上下文协议 (MCP) 服务器，旨在将其强大的搜索和数据检索功能作为 **工具** 公开给 AI 代理使用。这使开发人员能够构建智能应用程序——例如，使用 `openai-agents` 库或其他代理框架的应用程序——可以通过 LeanExplore 以编程方式与 Lean 4 代码库交互并进行推理。

## 目录

- [用于 AI 代理的 LeanExplore MCP 服务器](#用于-ai-代理的-leanexplore-mcp-服务器)
  - [目录](#目录)
  - [LeanExplore MCP 服务器概述](#leanexplore-mcp-服务器概述)
  - [与 MCP 客户端应用程序集成](#与-mcp-客户端应用程序集成)
  - [运行 LeanExplore MCP 服务器](#运行-leanexplore-mcp-服务器)
    - [命令调用](#命令调用)
    - [关键命令行选项](#关键命令行选项)
    - [服务器行为](#服务器行为)
  - [为 AI 代理公开的工具](#为-ai-代理公开的工具)
    - [工具：`search`](#工具search)
    - [工具：`get_by_id`](#工具get_by_id)
    - [工具：`get_dependencies`](#工具get_dependencies)
  - [与 `leanexplore chat` 的关系](#与-leanexplore-chat-的关系)
  - [自定义代理开发人员注意事项](#自定义代理开发人员注意事项)

## LeanExplore MCP 服务器概述

LeanExplore MCP 服务器充当 AI 代理的专用接口。它监听请求，通常格式化为 JSON-RPC 2.0 消息，通过 **标准输入/输出 (stdio)**。当代理发送使用工具的请求时，服务器将其转换为 LeanExplore 系统内的操作——根据其配置查询远程 LeanExplore API 或您的本地数据后端。

处理请求后，服务器将结果返回给代理，允许动态的编程交互。此服务器使用 `FastMCP` 库构建，该库是更广泛的 [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) 的一部分。

## 与 MCP 客户端应用程序集成

LeanExplore MCP 服务器可以作为工具提供者与各种兼容 MCP 的客户端应用程序集成。这些客户端允许您从统一界面管理多个 AI 工具和模型并与之交互。此类应用程序的一个示例是 Claude Desktop。

要配置像 Claude Desktop 这样的 MCP 客户端以使用 LeanExplore MCP 服务器，您通常需要提供指定如何启动服务器的配置。以下是此类配置在客户端应用程序设置（通常是 JSON 文件）中的示例：

```json
{
  "mcpServers": {
    "leanexploreAPI": {
      "command": "/path/to/your/leanexplore/package",
      "args": [
        "mcp",
        "serve",
        "--backend",
        "api",
        "--api-key",
        "YOUR_ACTUAL_LEANEXPLORE_API_KEY"
      ]
    }
  }
}
```

在 MCP 客户端应用程序中设置此配置后，它应该能够将 LeanExplore 列为可用的工具提供者，并使用服务器公开的工具（`search`、`get_by_id`、`get_dependencies`）。

有关设置像 Claude Desktop 这样的应用程序与 MCP 服务器的更多详细信息，请参阅其特定文档。对于 Claude Desktop，您可以在其[快速入门指南](https://modelcontextprotocol.io/quickstart/user)中找到信息。

## 运行 LeanExplore MCP 服务器

您可以使用 `leanexplore mcp serve` 命令直接从命令行启动 MCP 服务器。此命令适用于构建或连接自己的自定义 MCP 客户端应用程序或 AI 代理的开发人员。

### 命令调用

以下是启动服务器的典型方式：

```bash
# 使用 API 后端启动服务器（默认）
# 需要配置 LeanExplore API 密钥或通过 --api-key 传递
leanexplore mcp serve --backend api

# 使用您的本地数据后端启动服务器
# 需要通过 'leanexplore data fetch' 获取本地数据
leanexplore mcp serve --backend local

# 使用特定 API 密钥和 API 后端调试日志记录的示例
leanexplore mcp serve --backend api --api-key YOUR_LE_API_KEY --log-level DEBUG
```

### 关键命令行选项

* `--backend {api|local}`（别名：`-b`）确定服务器工具的数据源：
  + `api`：工具将查询远程 LeanExplore API。**先决条件：** 必须配置有效的 LeanExplore API 密钥（通过 `leanexplore configure api-key`）或使用 `--api-key` 选项直接提供。此后端通常提供对最新数据的访问并卸载计算。
  + `local`：工具将使用您本地下载的数据资源（SQLite 数据库、FAISS 索引）。**先决条件：** 您必须首先使用 `leanexplore data fetch` 下载数据工具链。此后端允许离线使用并完全控制数据版本。
* `--api-key TEXT`（可选）如果使用 `--backend api`，此选项允许您直接为当前服务器会话提供 LeanExplore API 密钥，覆盖任何全局配置的密钥。
* `--log-level {DEBUG|INFO|WARNING|ERROR|CRITICAL}` 设置服务器的日志记录详细程度。使用 `DEBUG` 有助于故障排除。默认通常是 `ERROR` 或 `WARNING` 以最小化噪音。

### 服务器行为

启动时，`leanexplore mcp serve` 命令在前台连续运行服务器。它接管您终端的标准输入和输出 (stdio) 以与连接的 MCP 客户端通信。

要停止服务器，连接的客户端应用程序通常应启动断开连接。或者，您可以在终端中手动终止服务器进程（通常使用 Ctrl+C）。

## 为 AI 代理公开的工具

LeanExplore MCP 服务器使其核心功能作为可调用的"工具"对 AI 代理可用。这些工具允许代理以编程方式搜索和检索有关 Lean 语句的信息。返回数据项的结构（下面称为 `ResultItem`）在返回语句组信息的工具之间是一致的。

`ResultItem` 对象包括以下字段：

* `id: integer` - 语句组的唯一标识符。
* `primary_declaration: object | null` - 有关主要声明的信息：
  + `lean_name: string | null` - 完整的 Lean 名称（例如，"Nat.add"）。
* `source_file: string` - 源文件路径（例如，"Mathlib/Data/Nat/Basic.lean"）。
* `range_start_line: integer` - 语句组在源文件中的起始行号。
* `statement_text: string` - 语句组的完整规范 Lean 代码文本。
* `docstring: string | null` - 与语句组关联的文档字符串（如果可用）。
* `informal_description: string | null` - 非形式的、人类可读的描述（如果可用）。

### 工具：`search`

* **目的：** 使代理能够基于自然语言查询查找 Lean 语句组。
* **关键参数：**
  + `query: string`（必需）- 自然语言搜索查询（例如，"连续函数"）。
  + `package_filters: string[]`（可选）- 要按其过滤结果的包名称列表（例如，`["Mathlib.Analysis", "Mathlib.Order"]`）。如果省略或为空，则不应用包过滤器。
  + `limit: integer`（可选，默认：10）- 要返回的最大搜索结果数。必须是正整数。
* **返回：** 包含搜索结果和元数据的对象，具有以下字段：
  + `query: string` - 提交的原始搜索查询字符串。
  + `packages_applied: string[] | null` - 实际应用于搜索的包过滤器列表。
  + `results: ResultItem[]` - 与查询匹配的 `ResultItem` 对象列表，结构如上所述。
  + `count: integer` - `results` 列表中提供的结果数（遵守 `limit` 参数）。
  + `total_candidates_considered: integer` - 在工具应用 `limit` 之前后端找到的潜在候选结果总数。
  + `processing_time_ms: integer` - 搜索请求的服务器端处理时间（以毫秒为单位）。

### 工具：`get_by_id`

* **目的：** 允许代理使用其唯一 ID 检索特定语句组的详细信息。
* **关键参数：**
  + `group_id: integer`（必需）- 要检索的语句组的唯一标识符（例如，`12345`）。
* **返回：** 如果找到具有给定 ID 的语句组，则返回单个 `ResultItem` 对象（结构如上所述）。如果不存在此类组，则返回 `null`。

### 工具：`get_dependencies`

* **目的：** 使代理能够获取给定语句组 ID 的直接依赖关系（即，被引用或依赖的项目）。
* **关键参数：**
  + `group_id: integer`（必需）- 要获取依赖关系的语句组的唯一标识符（例如，`12345`）。
* **返回：** 如果找到，则包含依赖关系的对象，否则为 `null`。该对象具有以下字段：
  + `source_group_id: integer` - 请求依赖关系的语句组的 ID。
  + `citations: ResultItem[]` - 表示直接依赖关系的 `ResultItem` 对象列表。每个项目的结构如上所述。
  + `count: integer` - 在 `citations` 列表中找到并返回的直接依赖关系数。

如果未找到源语句组或没有依赖关系，则返回 `null`。

**说明性代理工作流程：** AI 代理可能首先使用 `search` 工具发现与概念相关的语句（例如，"Frobenius 同态"）。从结果中，它可以选择一个看起来最相关的语句 ID。然后，它可能调用 `get_by_id` 来检索该语句的完整 Lean 代码，然后调用 `get_dependencies` 来了解其直接上下文和它依赖的定义。这个序列允许代理收集全面的信息用于推理或解释任务。

## 与 `leanexplore chat` 的关系

`leanexplore chat` 命令提供了一个即用型 AI 助手用于交互式探索。重要的是要理解，此聊天命令 **在内部管理其自己的 LeanExplore MCP 服务器实例** 并充当其 MCP 客户端。

因此，您不需要单独运行 `leanexplore mcp serve` 来使用 `leanexplore chat` 功能。`leanexplore mcp serve` 命令专门用于希望将他们 *自己的* 自定义 AI 代理或其他 MCP 客户端应用程序连接到 LeanExplore 工具集的开发人员。

## 自定义代理开发人员注意事项

* **通信协议：** LeanExplore MCP 服务器使用通过 **标准输入/输出 (stdio)** 交换的 JSON-RPC 2.0 消息进行通信。您的自定义代理客户端必须能够生成服务器进程并通过其 stdin/stdout 流与其通信。
* **客户端实现：** 要与此服务器交互，您需要一个 MCP 客户端。[MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)（提供 LeanExplore 使用的 `mcp` 库）包括用于构建此类客户端的实用程序，如 `ClientSession` 和 `stdio_client`。像 `openai-agents` 这样的框架也设计为与兼容 MCP 的服务器一起工作。
* **工具模式（Pydantic 模型）：** 虽然此页面提供了工具参数和返回结构的概述，但精确定义实现为 Pydantic 模型。需要确切模型模式的开发人员（例如，用于强大的客户端验证或代码生成）应参考 LeanExplore Python 包中 `lean_explore.shared.models.api.py` 模块（用于响应数据结构，如 `APISearchResultItem`、`APISearchResponse`、`APICitationsResponse`）和 `lean_explore.mcp.tools.py`（用于工具函数签名）中的源代码。代理调用的工具名称（例如，"search"）对应于注册为工具的函数名称。
* **服务器问题故障排除：** 如果您在自定义客户端与服务器交互时遇到问题，使用 `--log-level DEBUG` 选项运行 `leanexplore mcp serve` 可以提供来自服务器端的详细日志，这对于诊断与服务器启动、后端连接或工具执行相关的问题非常宝贵。

通过提供此 MCP 接口，LeanExplore 旨在成为数学研究和 Lean 开发的 AI 工具生态系统中的宝贵组件，使对其索引的 Lean 代码知识库的复杂编程访问成为可能。