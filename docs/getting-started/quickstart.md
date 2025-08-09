# Quickstart Guide

Welcome to the LeanExplore Quickstart Guide! This page will walk you through the essential first steps to get up and running with LeanExplore using its powerful remote API and Command-Line Interface (CLI). Our goal is to help you perform your first meaningful actions quickly.

## Prerequisites: Installation

Before you begin, ensure you have LeanExplore installed. If not, you can install it using pip:

```bash
pip install lean-explore
```

This guide focuses on using the LeanExplore API, which is the default mode for many commands and provides a zero-setup way to access search and AI features once your API keys are configured.

## Step 1: Obtain Your LeanExplore API Key

To interact with the LeanExplore API, you'll need a personal API key. This key authenticates your requests to our servers. If you don't have one, please visit [https://www.leanexplore.com/api-keys](https://www.leanexplore.com/api-keys) to sign up or log in and obtain your API key.

## Step 2: Configure Your LeanExplore API Key

Once you have your API key, the next step is to configure it with the LeanExplore CLI. This will save your key securely for future use, so you don't have to enter it every time.

Run the following command in your terminal:

```bash
leanexplore configure api-key
```

You will be prompted to paste your API key. This is a one-time setup. If you've already done this, you can proceed to the next step.

## Step 3: Perform Your First Search

With your API key configured, you're ready to perform your first search directly from the command line. Let's try searching for a well-known theorem:

```bash
leanexplore search "fundamental theorem of calculus"
```

You should see a list of relevant Lean statements matching your query, along with details like their source file, line number, and a snippet of the code. This demonstrates the direct search capability of LeanExplore via its API.

## Step 4: Try the AI Chat (via API)

LeanExplore also offers an AI-powered chat assistant for a more interactive way to explore Lean code. By default, if your LeanExplore API key is configured, the chat will use the API backend. For the AI capabilities, you'll also need an OpenAI API key.

### 4a. Configure OpenAI API Key (If Needed)

If you haven't set up your OpenAI API key with LeanExplore yet, run:

```bash
leanexplore configure openai-key
```

Follow the prompts. If this key is already configured, you can skip this sub-step.

### 4b. Start the Chat Session

Now, launch the AI chat assistant:

```bash
leanexplore chat
```

Since your LeanExplore API key is configured and `--backend api` is the default, the assistant will use the remote API for its Lean-specific knowledge.

### 4c. Ask a Question

Once the chat interface loads and the assistant is ready, try asking it to find a specific definition and its context, for example:

```
You: Find the formal statement for the 'fundamental theorem of calculus'
and tell me about its main dependencies.
```

## Step 5: Observe the Results

The AI assistant will process your query. It uses the LeanExplore API to search for formal statements related to the 'fundamental theorem of calculus', identify the most relevant one(s), and then find and list their main dependencies. You should expect a conversational response that includes the formal Lean code for the theorem, along with explanations and other contextual information to help you understand its structure and connections within the library.

## Congratulations & Next Steps

You've now successfully used LeanExplore to perform a direct API search and to interact with the AI chat assistant! These are two of the primary ways to leverage LeanExplore's capabilities.

From here, you can:

* Dive deeper into all command-line options in the [Using the CLI](../cli/usage.md) guide.
* Learn about using local data and programmatic access (both local and API) in the [Performing Searches](../cli/search.md) section.
* If you're interested in building custom AI agent integrations, explore the [MCP (AI Agents)](../mcp/agents.md) documentation.