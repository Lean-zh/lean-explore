# LeanExplore MCP Server for AI Agents

LeanExplore includes a Model Context Protocol (MCP) server designed to expose its powerful search and data retrieval functionalities as **Tools** that AI agents can utilize. This enables developers to build intelligent applications—for example, those using the `openai-agents` library or other agentic frameworks—that can programmatically interact with and reason about Lean 4 codebases through LeanExplore.

## Contents

* [Overview of the LeanExplore MCP Server](#overview-of-the-leanexplore-mcp-server)
* [Integrating with MCP Client Applications](#integrating-with-mcp-client-applications)
* [Running the LeanExplore MCP Server](#running-the-leanexplore-mcp-server)
* [Command Invocation](#command-invocation)
* [Key Command-Line Options](#key-command-line-options)
* [Server Behavior](#server-behavior)
* [Exposed Tools for AI Agents](#exposed-tools-for-ai-agents)
* [Tool: search](#tool-search)
* [Tool: get_by_id](#tool-get_by_id)
* [Tool: get_dependencies](#tool-get_dependencies)
* [Relationship with leanexplore chat](#relationship-with-leanexplore-chat)
* [Notes for Custom Agent Developers](#notes-for-custom-agent-developers)

## Overview of the LeanExplore MCP Server

The LeanExplore MCP server acts as a dedicated interface for AI agents. It listens for requests, typically formatted as JSON-RPC 2.0 messages, over **standard input/output (stdio)**. When an agent sends a request to use a tool, the server translates this into an action within the LeanExplore system—querying either the remote LeanExplore API or your local data backend, depending on its configuration.

After processing the request, the server returns the results to the agent, allowing for a dynamic, programmatic interaction. This server is built using the `FastMCP` library, part of the broader [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk).

## Integrating with MCP Client Applications

The LeanExplore MCP server can be integrated as a tool provider with various MCP-compatible client applications. These clients allow you to manage and interact with multiple AI tools and models from a unified interface. An example of such an application is Claude Desktop.

To configure an MCP client like Claude Desktop to use the LeanExplore MCP server, you typically need to provide a configuration that specifies how to launch the server. Here's an example of what such a configuration would look like in the client application's settings (often a JSON file):

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

After setting up this configuration in your MCP client application, it should be able to list LeanExplore as an available tool provider and use the tools (`search`, `get_by_id`, `get_dependencies`) exposed by the server.

For more details on setting up applications like Claude Desktop with MCP servers, refer to their specific documentation. For Claude Desktop, you can find information at their [quickstart guide](https://modelcontextprotocol.io/quickstart/user).

## Running the LeanExplore MCP Server

You can launch the MCP server directly from the command line using the `leanexplore mcp serve` command. This command is intended for developers who are building or connecting their own custom MCP client applications or AI agents.

### Command Invocation

Here are typical ways to start the server:

```bash
# Start server using the API backend (default)
# Requires LeanExplore API key to be configured or passed via --api-key
leanexplore mcp serve --backend api

# Start server using your local data backend
# Requires local data to be fetched via 'leanexplore data fetch'
leanexplore mcp serve --backend local

# Example with specific API key and debug logging for the API backend
leanexplore mcp serve --backend api --api-key YOUR_LE_API_KEY --log-level DEBUG
```

### Key Command-Line Options

* `--backend {api|local}` (alias: `-b`) Determines the data source for the server's tools:
  + `api`: Tools will query the remote LeanExplore API. **Prerequisite:** A valid LeanExplore API key must be configured (via `leanexplore configure api-key`) or provided directly using the `--api-key` option. This backend typically provides access to the most current data and offloads computation.
  + `local`: Tools will use your locally downloaded data assets (SQLite database, FAISS index). **Prerequisite:** You must first download the data toolchain using `leanexplore data fetch`. This backend allows for offline use and full control over the data version.
* `--api-key TEXT` (Optional) If using `--backend api`, this option allows you to provide the LeanExplore API key directly for the current server session, overriding any globally configured key.
* `--log-level {DEBUG|INFO|WARNING|ERROR|CRITICAL}` Sets the logging verbosity for the server. Using `DEBUG` is helpful for troubleshooting. Default is typically `ERROR` or `WARNING` to minimize noise.

### Server Behavior

When started, the `leanexplore mcp serve` command runs the server continuously in the foreground. It takes over your terminal's standard input and output (stdio) to communicate with the connected MCP client.

To stop the server, the connected client application should typically initiate a disconnect. Alternatively, you can manually terminate the server process in your terminal (usually with Ctrl+C).

## Exposed Tools for AI Agents

The LeanExplore MCP server makes its core functionalities available to AI agents as callable "tools". These tools allow an agent to programmatically search and retrieve information about Lean statements. The structure of the returned data items (referred to as `ResultItem` below) is consistent across tools that return statement group information.

A `ResultItem` object includes the following fields:

* `id: integer` - Unique identifier of the statement group.
* `primary_declaration: object | null` - Information about the primary declaration:
  + `lean_name: string | null` - The full Lean name (e.g., "Nat.add").
* `source_file: string` - The source file path (e.g., "Mathlib/Data/Nat/Basic.lean").
* `range_start_line: integer` - The starting line number of the statement group in the source file.
* `statement_text: string` - The full canonical Lean code text of the statement group.
* `docstring: string | null` - The docstring associated with the statement group, if available.
* `informal_description: string | null` - An informal, human-readable description, if available.

### Tool: `search`

* **Purpose:** Enables the agent to find Lean statement groups based on a natural language query.
* **Key Parameters:**
  + `query: string` (required) - The natural language search query (e.g., "continuous function").
  + `package_filters: string[]` (optional) - A list of package names to filter results by (e.g., `["Mathlib.Analysis", "Mathlib.Order"]`). If omitted or empty, no package filter is applied.
  + `limit: integer` (optional, default: 10) - The maximum number of search results to return. Must be a positive integer.
* **Returns:** An object containing the search results and metadata, with the following fields:
  + `query: string` - The original search query string submitted.
  + `packages_applied: string[] | null` - List of package filters that were actually applied to the search.
  + `results: ResultItem[]` - A list of `ResultItem` objects matching the query, structured as described above.
  + `count: integer` - The number of results provided in the `results` list (respecting the `limit` parameter).
  + `total_candidates_considered: integer` - The total number of potential candidate results found by the backend before the `limit` was applied by the tool.
  + `processing_time_ms: integer` - Server-side processing time for the search request in milliseconds.

### Tool: `get_by_id`

* **Purpose:** Allows the agent to retrieve detailed information for a specific statement group using its unique ID.
* **Key Parameters:**
  + `group_id: integer` (required) - The unique identifier of the statement group to retrieve (e.g., `12345`).
* **Returns:** A single `ResultItem` object (structured as described above) if a statement group with the given ID is found. Returns `null` if no such group exists.

### Tool: `get_dependencies`

* **Purpose:** Enables the agent to fetch the direct dependencies (i.e., items cited by or relied upon) for a given statement group ID.
* **Key Parameters:**
  + `group_id: integer` (required) - The unique identifier of the statement group for which to fetch dependencies (e.g., `12345`).
* **Returns:** An object containing the dependencies if found, otherwise `null`. The object has the following fields:
  + `source_group_id: integer` - The ID of the statement group for which dependencies were requested.
  + `citations: ResultItem[]` - A list of `ResultItem` objects representing the direct dependencies. Each item is structured as described above.
  + `count: integer` - The number of direct dependencies found and returned in the `citations` list.

Returns `null` if the source statement group is not found or has no dependencies.

**Illustrative Agent Workflow:** An AI agent might first use the `search` tool to discover statements related to a concept (e.g., "Frobenius homomorphism"). From the results, it could pick a statement ID that seems most relevant. Then, it might call `get_by_id` to retrieve the full Lean code for that statement, followed by a call to `get_dependencies` to understand its immediate context and the definitions it relies on. This sequence allows the agent to gather comprehensive information for reasoning or explanation tasks.

## Relationship with `leanexplore chat`

The `leanexplore chat` command provides a ready-to-use AI assistant for interactive exploration. It's important to understand that this chat command **internally manages its own instance of the LeanExplore MCP server** and acts as an MCP client to it.

Therefore, you do not need to run `leanexplore mcp serve` separately to use the `leanexplore chat` feature. The `leanexplore mcp serve` command is specifically for developers who wish to connect their *own* custom AI agents or other MCP client applications to LeanExplore's toolset.

## Notes for Custom Agent Developers

* **Communication Protocol:** The LeanExplore MCP server communicates using JSON-RPC 2.0 messages exchanged over **standard input/output (stdio)**. Your custom agent client must be able to spawn the server process and communicate with it via its stdin/stdout streams.
* **Client Implementation:** To interact with this server, you'll need an MCP client. The [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk) (which provides the `mcp` library used by LeanExplore) includes utilities like `ClientSession` and `stdio_client` for building such clients. Frameworks like `openai-agents` are also designed to work with MCP-compliant servers.
* **Tool Schemas (Pydantic Models):** While this page provides an overview of tool parameters and return structures, the precise definitions are implemented as Pydantic models. Developers needing the exact model schemas (e.g., for robust client-side validation or code generation) should refer to the source code in the `lean_explore.shared.models.api.py` module (for response data structures like `APISearchResultItem`, `APISearchResponse`, `APICitationsResponse`) and `lean_explore.mcp.tools.py` (for tool function signatures) within the LeanExplore Python package. The tool names called by the agent (e.g., "search") correspond to the function names registered as tools.
* **Troubleshooting Server Issues:** If you encounter problems when your custom client interacts with the server, running `leanexplore mcp serve` with the `--log-level DEBUG` option can provide verbose logs from the server side, which can be invaluable for diagnosing issues related to server startup, backend connections, or tool execution.

By providing this MCP interface, LeanExplore aims to be a valuable component in the ecosystem of AI tools for mathematical research and Lean development, enabling sophisticated programmatic access to its indexed knowledge base of Lean code.