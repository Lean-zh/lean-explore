# Performing Searches with LeanExplore

LeanExplore offers flexible and powerful ways to search for Lean mathematical statements and definitions, catering to different needs and workflows. This guide details how to perform searches programmatically using either the remote LeanExplore API for immediate access or local data resources for offline capabilities. These methods allow you to integrate LeanExplore's search engine into your custom Python tools or analyses.

## Contents

* [Searching via the Remote API](#searching-via-the-remote-api)
* [Prerequisites](#prerequisites)
* [Programmatic Access (Asynchronous)](#programmatic-access-asynchronous)
* [Internal Usage by CLI](#internal-usage-by-cli)
* [Searching with Local Data](#searching-with-local-data)
* [Prerequisites](#prerequisites-1)
* [Programmatic Access](#programmatic-access)
* [Internal Usage by CLI](#internal-usage-by-cli-1)

## Searching via the Remote API

Using the remote API is a quick way to get started, as it offloads data hosting and search computation to the LeanExplore servers. All you need is an API key and an internet connection. The methods in the API client are asynchronous. For use in IPython or Jupyter, top-level `await` can often be used directly.

### Prerequisites

* The `lean_explore` Python package installed (`pip install lean-explore`). This includes the `httpx` library required by the API client.
* Your personal LeanExplore API key. You can obtain one from [leanexplore.com/api-keys](https://www.leanexplore.com/api-keys).
* This API key should be configured using the CLI for easy loading:
  ```bash
  leanexplore configure api-key YOUR_API_KEY_HERE
  ```
  Alternatively, set the `LEANEXPLORE_API_KEY` environment variable.

### Programmatic Access (Asynchronous)

#### 1. Initializing the API Client

Import necessary modules and initialize the client. This assumes your API key is configured.

```python
import asyncio
from lean_explore.api.client import Client
from lean_explore.cli import config_utils

# Load API key (ensure it's configured via CLI or ENV variable)
api_key = config_utils.load_api_key() 
client = Client(api_key=api_key)
print("API Client initialized.")
```

#### 2. Performing a Search

Use the `client.search()` method. It returns an `APISearchResponse` object. (Requires `client` from step 1).

```python
# Define query and limit for displaying results
query_str_api = "fundamental theorem of calculus"
display_limit_api = 3

# Perform the search (use 'await' in an async context e.g. IPython, Jupyter, or async script)
search_response_api = await client.search(query=query_str_api)

print(f"\nFound {search_response_api.count} API results for '{query_str_api}':")
for item_api in search_response_api.results[:display_limit_api]:
    name_api = (item_api.primary_declaration.lean_name
                if item_api.primary_declaration else "N/A")
    print(f"  ID: {item_api.id}, Name: {name_api}")
    print(f"    File: {item_api.source_file}:{item_api.range_start_line}")

# Example: Get ID of the first result, assuming results are present
api_first_result_id = search_response_api.results[0].id
print(f"ID of the first API result: {api_first_result_id}")
```

#### 3. Retrieving a Statement Group by ID

Use `client.get_by_id()`. (Requires `client` and `api_first_result_id` from previous steps).

```python
# Use the ID obtained from the search results
item_details_api = await client.get_by_id(group_id=api_first_result_id)

name_details_api = (item_details_api.primary_declaration.lean_name
                    if item_details_api.primary_declaration else "N/A")
print(f"\nDetails for API Statement Group ID {item_details_api.id}: Name: {name_details_api}")
# print(f"  Statement: {item_details_api.statement_text}")
```

#### 4. Fetching Dependencies

Use `client.get_dependencies()`. (Requires `client` and `api_first_result_id`).

```python
# Use the ID obtained from the search results
deps_response_api = await client.get_dependencies(group_id=api_first_result_id)

print(f"\nAPI Dependencies for Group ID {deps_response_api.source_group_id}\n  ({deps_response_api.count} found):")
for citation_api in deps_response_api.citations:
    name_deps_api = (citation_api.primary_declaration.lean_name
                     if citation_api.primary_declaration else "N/A")
    print(f"  - Dep ID: {citation_api.id}, Name: {name_deps_api}")
```

**Note on Running Async Code:** The `await` keyword is used for API calls. In a standalone Python script, you would typically wrap these calls in an `async def` function and run it using `asyncio.run()`. In environments like IPython 7.0+ or Jupyter notebooks, top-level `await` is often supported directly.

### Internal Usage by CLI

The LeanExplore CLI commands such as `leanexplore search`, `get`, `dependencies`, and `leanexplore chat --backend api` (which is the default for chat if an API key is set) all utilize this `lean_explore.api.client.Client` internally.

## Searching with Local Data

This mode allows you to perform searches directly on your machine using a local dataset. It leverages a SQLite database for structured information, a FAISS vector index for efficient semantic matching, and local sentence embedding models to process your queries. This is ideal for offline use, custom data analysis, or when you prefer full control over the data assets.

### Prerequisites

* The `lean_explore` Python package installed.
* The local data toolchain must be downloaded and available. You can do this using the CLI command:
  ```bash
  leanexplore data fetch
  ```
  This ensures the database and FAISS index are present in the expected location (typically `~/.lean_explore/data/toolchains/<version>/`).

### Programmatic Access

#### 1. Initializing the Local Service

Import and instantiate the `lean_explore.local.service.Service` class. This assumes data files are correctly downloaded.

```python
from lean_explore.local.service import Service

# Assumes Service() initializes successfully (data files are present)
service_instance = Service()
print("LocalService initialized successfully.")
```

#### 2. Performing a Search

Use the `service.search()` method. It returns an `APISearchResponse` object. (Requires `service_instance` from step 1).

```python
# Define query and limit
query_str_local = "ring definition"
limit_for_local = 3

search_response_local = service_instance.search(
    query=query_str_local, 
    limit=limit_for_local
)

print(f"\nFound {search_response_local.count} local results for '{query_str_local}':")
for item_local in search_response_local.results: # Already limited by service.search
    name_local = (item_local.primary_declaration.lean_name
                  if item_local.primary_declaration else "N/A")
    print(f"  ID: {item_local.id}, Name: {name_local}")
    print(f"    File: {item_local.source_file}:{item_local.range_start_line}")

# Example: Get ID of the first result, assuming results are present
local_first_result_id = search_response_local.results[0].id
print(f"ID of the first local result: {local_first_result_id}")
```

#### 3. Retrieving a Statement Group by ID

Use `service.get_by_id()`. (Requires `service_instance` and `local_first_result_id`).

```python
# Use the ID obtained from the local search results
item_details_local = service_instance.get_by_id(
    group_id=local_first_result_id
)

name_details_local = (item_details_local.primary_declaration.lean_name
                      if item_details_local.primary_declaration else "N/A")
print(f"\nDetails for Local Statement Group ID {item_details_local.id}: Name: {name_details_local}")
# print(f"  Statement: {item_details_local.statement_text}")
```

#### 4. Fetching Dependencies

Use `service.get_dependencies()`. (Requires `service_instance` and `local_first_result_id`).

```python
# Use the ID obtained from the local search results
deps_response_local = service_instance.get_dependencies(
    group_id=local_first_result_id
)

print(f"\nLocal Dependencies for Group ID {deps_response_local.source_group_id}\n  ({deps_response_local.count} found):")
for citation_local in deps_response_local.citations:
    name_deps_local = (citation_local.primary_declaration.lean_name
                       if citation_local.primary_declaration else "N/A")
    print(f"  - Dep ID: {citation_local.id}, Name: {name_deps_local}")
```

### Internal Usage by CLI

The LeanExplore CLI commands `leanexplore chat --backend local` and `leanexplore mcp serve --backend local` make use of this `lean_explore.local.service.Service` class to interact with your local data toolchain.