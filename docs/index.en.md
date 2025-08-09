# LeanExplore Documentation

Welcome to the LeanExplore documentation! This is your central hub for understanding how to navigate and utilize the LeanExplore Python package effectively. Find the project on [GitHub](https://github.com/justincasher/lean-explore).

LeanExplore is a Python package offering a powerful semantic search engine and versatile toolkit for Lean 4 projects. It assists Lean users in efficiently finding code through semantic search—leveraging vector-based similarity from embeddings of both formal Lean statements and their informal counterparts—and in exploring the dependencies of these statements to gain deeper insights. LeanExplore serves as a valuable component for building custom tools, enhancing proof development workflows, and integrating Lean's structured knowledge into AI applications.

## Core Features

* **Versatile Search Capabilities:** Find Lean statements with precision. Search semantically to discover code based on conceptual meaning, or search directly by known declaration names for targeted results.
* **Interactive AI Assistance:** Utilize an AI-powered chat interface (via the CLI) to ask questions about Lean code, receive explanations, and explore dependencies conversationally.
* **Flexible Data Backends:** Choose to operate with a fully local dataset for complete offline access and control, or leverage our convenient remote API for zero-setup searching capabilities.
* **Comprehensive CLI Toolkit:** Manage local data, configure settings, perform searches, and launch AI interactions directly from your terminal.

## Getting Started

The best way to begin your journey with LeanExplore is by following our **[Quickstart Guide](getting-started/quickstart.md)**. This guide will walk you through the initial setup and your first key interactions with the package, ensuring you get up and running smoothly.

Once you're familiar with the basics, you can explore these areas for more detailed information on specific functionalities:

* [Using the Command-Line Interface (CLI)](cli/usage.md): Master the full range of available commands and options for various tasks from data management to AI chat.
* [Performing Searches](cli/search.md): Learn how to search programmatically, whether you're using local data resources or interacting with the remote API.
* [MCP (AI Agents)](mcp/agents.md): Discover how to integrate LeanExplore with AI agent systems for advanced applications and custom tool development.

For a detailed look at every function, class, and module within the LeanExplore package, including their parameters and return types, please browse the **API Reference** section available in the sidebar.