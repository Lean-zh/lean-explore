# Using the LeanExplore CLI

The LeanExplore Command-Line Interface (CLI), invoked using the `leanexplore` command, is your primary tool for interacting with the LeanExplore ecosystem. It empowers you to configure settings, manage local data, search the LeanExplore API directly, explore code with an AI assistant, and control the Model Context Protocol (MCP) server for more advanced integrations.

## Contents

* [Understanding CLI Basics](#understanding-cli-basics)
* [Getting Started: Initial Setup](#getting-started-initial-setup)
* [Configuring Your API Keys](#configuring-your-api-keys)
* [(Optional) Preparing for Local Data Exploration](#optional-preparing-for-local-data-exploration)
* [Finding and Inspecting Lean Code (via API)](#finding-and-inspecting-lean-code-via-api)
* [Searching for Lean Statements](#searching-for-lean-statements)
* [Viewing Detailed Information](#viewing-detailed-information)
* [Exploring Code Dependencies](#exploring-code-dependencies)
* [Interactive Exploration with the AI Chat Assistant](#interactive-exploration-with-the-ai-chat-assistant)
* [Launching the Chat Session](#launching-the-chat-session)
* [Understanding Key Chat Options](#understanding-key-chat-options)
* [Integrating with AI Agents via the MCP Server](#integrating-with-ai-agents-via-the-mcp-server)
* [Running the Server](#running-the-server)
* [Key Server Options](#key-server-options)

## Understanding CLI Basics

Most CLI commands follow the structure: `leanexplore [OPTIONS] COMMAND [ARGS]...`. To get help at any point, whether for the main command or a specific subcommand, simply append `--help`:

```bash
leanexplore --help
```

```bash
leanexplore configure --help
```

```bash
leanexplore data fetch --help
```

## Getting Started: Initial Setup

Before diving into all of LeanExplore's features, a few initial setup steps can ensure everything runs smoothly, especially when interacting with online services or the AI assistant.

### Configuring Your API Keys

LeanExplore uses API keys to authenticate access to certain services. These are stored securely in a local configuration file once set up.

**LeanExplore API Key:** This key is essential for features that communicate with the remote LeanExplore API, such as direct searches (`search`, `get`, `dependencies` commands) and using the AI chat with its default API backend.

You can obtain your LeanExplore API key from <https://www.leanexplore.com/api-keys>. Once you have it, configure it by running:

```bash
leanexplore configure api-key
```

You will be prompted to enter your key. This is typically a one-time setup.

**OpenAI API Key:** If you plan to use the AI-powered chat features (`leanexplore chat`), you'll also need an OpenAI API key. Configure it with:

```bash
leanexplore configure openai-key
```

Follow the prompts to enter your OpenAI key.

### (Optional) Preparing for Local Data Exploration

For users who prefer to work offline or have direct control over the dataset, LeanExplore supports a local data toolchain. This involves downloading the necessary data assets to your machine.

To download and set up the primary local data toolchain, use the command:

```bash
leanexplore data fetch
```

This command fetches the SQLite database (containing Lean project information), FAISS search indexes for semantic search, and associated mapping files. These are installed into a local directory, typically within `~/.lean_explore/data/toolchains/`.

**Note:** The data toolchain can be several gigabytes. The initial download may take some time depending on your internet connection. This step is a prerequisite for using features that rely on a local backend, such as `leanexplore chat --backend local` or `leanexplore mcp serve --backend local`.

## Finding and Inspecting Lean Code (via API)

Once your LeanExplore API key is configured, you can directly query the remote API to find and inspect Lean statements. These commands provide quick access to the wealth of information indexed by LeanExplore.

### Searching for Lean Statements

To search for Lean statement groups based on a natural language query, use the `leanexplore search` command:

```bash
leanexplore search "your query string here" [OPTIONS]
```

For example, to find statements related to the "fundamental theorem of calculus", limit the results, and filter by the "Mathlib" package:

```bash
leanexplore search "fundamental theorem of calculus" --package Mathlib --limit 3
```

**Key Options for Searching:**

* `QUERY_STRING`: Your search terms. Enclose in quotes if it contains spaces.
* `--package TEXT` (or `-p TEXT`): Filter results by specific package names (e.g., `Mathlib`, `Std`). This option can be used multiple times to include several packages.
* `--limit INTEGER` (or `-n INTEGER`): Specify the maximum number of search results to display. Defaults to 5.

The command will display a list of matching statement groups, including their ID, Lean name, source file location, and relevant code or docstring snippets.

### Viewing Detailed Information

If you have a specific statement group ID (often obtained from search results), you can retrieve its full details using `leanexplore get`:

```bash
leanexplore get <GROUP_ID>
```

For instance:

```bash
leanexplore get 12345
```

This shows comprehensive information for the group, such as its full statement text, docstring, and any informal descriptions, usually presented in easy-to-read formatted panels.

### Exploring Code Dependencies

To understand how a statement group connects to others, you can fetch its direct dependencies (items it cites) using `leanexplore dependencies`:

```bash
leanexplore dependencies <GROUP_ID>
```

Example:

```bash
leanexplore dependencies 12345
```

This command lists the statement groups that the specified group depends on, typically in a table format showing each dependency's ID, Lean name, and source location.

## Interactive Exploration with the AI Chat Assistant

LeanExplore provides an AI-powered chat assistant (`leanexplore chat`) for a conversational way to search, understand, and explore Lean code.

**Prerequisites:**

* Your OpenAI API key must be configured (using `leanexplore configure openai-key`).
* For the default API backend: Your LeanExplore API key must be configured.
* For the local backend: Your local data toolchain must be fetched (using `leanexplore data fetch`).

### Launching the Chat Session

To start a chat session using the default API backend (recommended for most users, providing access to the latest data):

```bash
leanexplore chat
```

If you've set up local data and prefer to use that (e.g., for offline access):

```bash
leanexplore chat --backend local
```

### Understanding Key Chat Options

* `--backend {api|local}` (alias: `-lb`): Specifies the data source for the AI's tools.
  + `api` (default): The agent queries the remote LeanExplore API.
  + `local`: The agent uses your downloaded local data.
* `--lean-api-key TEXT`: (Optional) If using the API backend, you can provide a LeanExplore API key for the current session, overriding any configured key.
* `--debug`: Enables detailed debug logging for both the chat client and the underlying MCP server it manages. This is useful for troubleshooting.

Once in the chat, you can ask the assistant to perform tasks like "Find definitions of 'monoid' in Mathlib" or "Show me the dependencies of `Nat.add`".

## Integrating with AI Agents via the MCP Server

This is an advanced feature for developers aiming to integrate LeanExplore's search and retrieval capabilities as tools within their own custom AI agent applications or other programmatic setups.

The `leanexplore mcp serve` command launches the LeanExplore Model Context Protocol (MCP) server. This server communicates via standard input/output (stdio) using JSON-RPC 2.0 and exposes LeanExplore's functionalities as "tools" that an MCP-compatible agent client can call.

### Running the Server

To serve tools using the remote API backend (ensure LeanExplore API key is configured or provide it with `--api-key`):

```bash
leanexplore mcp serve --backend api
```

To serve tools using your local data backend (ensure local data has been fetched):

```bash
leanexplore mcp serve --backend local
```

### Key Server Options

* `--backend {api|local}` (alias: `-b`): Determines if the server's tools will use the remote API or local data. Defaults to `api`.
* `--api-key TEXT`: (Required if `--backend api` and no key is configured) Directly provide the LeanExplore API key for the server to use.

**Note:** The `leanexplore chat` command internally manages an instance of this MCP server. Running `leanexplore mcp serve` directly is typically for scenarios where you are connecting a different MCP client or agent framework.

By familiarizing yourself with these commands and workflows, you can effectively leverage the LeanExplore CLI for your mathematical explorations and development in Lean 4.