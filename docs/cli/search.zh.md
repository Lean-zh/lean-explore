# 使用 LeanExplore 执行搜索

LeanExplore 提供灵活而强大的方式来搜索 Lean 数学语句和定义，满足不同的需求和工作流程。本指南详细介绍了如何使用远程 LeanExplore API 进行即时访问，或通过本地数据资源离线功能来以编程方式执行搜索。这些方法允许您将 LeanExplore 的搜索引擎集成到您的自定义 Python 工具或分析中。

## 目录

* [通过远程 API 搜索](#通过远程-api-搜索)
* [先决条件](#先决条件)
* [编程访问（异步）](#编程访问异步)
* [CLI 内部使用](#cli-内部使用)
* [使用本地数据搜索](#使用本地数据搜索)
* [先决条件](#先决条件-1)
* [编程访问](#编程访问)
* [CLI 内部使用](#cli-内部使用-1)

## 通过远程 API 搜索

使用远程 API 是快速入门的方式，因为它将数据托管和搜索计算卸载到 LeanExplore 服务器。您只需要一个 API 密钥和互联网连接。API 客户端中的方法是异步的。对于在 IPython 或 Jupyter 中使用，通常可以直接使用顶级 `await`。

### 先决条件

* 安装了 `lean_explore` Python 包（`pip install lean-explore`）。这包括 API 客户端所需的 `httpx` 库。
* 您的个人 LeanExplore API 密钥。您可以从 [leanexplore.com/api-keys](https://www.leanexplore.com/api-keys) 获取一个。
* 此 API 密钥应使用 CLI 配置以便于加载：
  ```bash
  leanexplore configure api-key YOUR_API_KEY_HERE
  ```
  或者，设置 `LEANEXPLORE_API_KEY` 环境变量。

### 编程访问（异步）

#### 1. 初始化 API 客户端

导入必要的模块并初始化客户端。这假设您的 API 密钥已配置。

```python
import asyncio
from lean_explore.api.client import Client
from lean_explore.cli import config_utils

# 加载 API 密钥（确保通过 CLI 或 ENV 变量配置）
api_key = config_utils.load_api_key() 
client = Client(api_key=api_key)
print("API 客户端已初始化。")
```

#### 2. 执行搜索

使用 `client.search()` 方法。它返回一个 `APISearchResponse` 对象。（需要步骤 1 中的 `client`）。

```python
# 定义查询和显示结果的限制
query_str_api = "微积分基本定理"
display_limit_api = 3

# 执行搜索（在异步上下文中使用 'await'，例如 IPython、Jupyter 或异步脚本）
search_response_api = await client.search(query=query_str_api)

print(f"\n为 '{query_str_api}' 找到 {search_response_api.count} 个 API 结果：")
for item_api in search_response_api.results[:display_limit_api]:
    name_api = (item_api.primary_declaration.lean_name
                if item_api.primary_declaration else "N/A")
    print(f"  ID: {item_api.id}, 名称: {name_api}")
    print(f"    文件: {item_api.source_file}:{item_api.range_start_line}")

# 示例：获取第一个结果的 ID，假设存在结果
api_first_result_id = search_response_api.results[0].id
print(f"第一个 API 结果的 ID: {api_first_result_id}")
```

#### 3. 通过 ID 检索语句组

使用 `client.get_by_id()`。（需要前面步骤中的 `client` 和 `api_first_result_id`）。

```python
# 使用从搜索结果获得的 ID
item_details_api = await client.get_by_id(group_id=api_first_result_id)

name_details_api = (item_details_api.primary_declaration.lean_name
                    if item_details_api.primary_declaration else "N/A")
print(f"\nAPI 语句组 ID {item_details_api.id} 的详细信息: 名称: {name_details_api}")
# print(f"  语句: {item_details_api.statement_text}")
```

#### 4. 获取依赖关系

使用 `client.get_dependencies()`。（需要 `client` 和 `api_first_result_id`）。

```python
# 使用从搜索结果获得的 ID
deps_response_api = await client.get_dependencies(group_id=api_first_result_id)

print(f"\n组 ID {deps_response_api.source_group_id} 的 API 依赖关系\n  （找到 {deps_response_api.count} 个）：")
for citation_api in deps_response_api.citations:
    name_deps_api = (citation_api.primary_declaration.lean_name
                     if citation_api.primary_declaration else "N/A")
    print(f"  - 依赖 ID: {citation_api.id}, 名称: {name_deps_api}")
```

**关于运行异步代码的注意事项：** `await` 关键字用于 API 调用。在独立的 Python 脚本中，您通常会将这些调用包装在 `async def` 函数中并使用 `asyncio.run()` 运行它。在 IPython 7.0+ 或 Jupyter notebook 等环境中，通常直接支持顶级 `await`。

### CLI 内部使用

LeanExplore CLI 命令，如 `leanexplore search`、`get`、`dependencies` 和 `leanexplore chat --backend api`（如果设置了 API 密钥，这是聊天的默认设置）都在内部使用此 `lean_explore.api.client.Client`。

## 使用本地数据搜索

此模式允许您直接在机器上使用本地数据集执行搜索。它利用 SQLite 数据库存储结构化信息，FAISS 向量索引进行高效的语义匹配，以及本地句子嵌入模型来处理您的查询。这对于离线使用、自定义数据分析或当您希望完全控制数据资产时是理想的。

### 先决条件

* 安装了 `lean_explore` Python 包。
* 必须下载并可用本地数据工具链。您可以使用 CLI 命令执行此操作：
  ```bash
  leanexplore data fetch
  ```
  这确保数据库和 FAISS 索引存在于预期位置（通常是 `~/.lean_explore/data/toolchains/<version>/`）。

### 编程访问

#### 1. 初始化本地服务

导入并实例化 `lean_explore.local.service.Service` 类。这假设数据文件已正确下载。

```python
from lean_explore.local.service import Service

# 假设 Service() 成功初始化（数据文件存在）
service_instance = Service()
print("本地服务已成功初始化。")
```

#### 2. 执行搜索

使用 `service.search()` 方法。它返回一个 `APISearchResponse` 对象。（需要步骤 1 中的 `service_instance`）。

```python
# 定义查询和限制
query_str_local = "环定义"
limit_for_local = 3

search_response_local = service_instance.search(
    query=query_str_local, 
    limit=limit_for_local
)

print(f"\n为 '{query_str_local}' 找到 {search_response_local.count} 个本地结果：")
for item_local in search_response_local.results: # 已由 service.search 限制
    name_local = (item_local.primary_declaration.lean_name
                  if item_local.primary_declaration else "N/A")
    print(f"  ID: {item_local.id}, 名称: {name_local}")
    print(f"    文件: {item_local.source_file}:{item_local.range_start_line}")

# 示例：获取第一个结果的 ID，假设存在结果
local_first_result_id = search_response_local.results[0].id
print(f"第一个本地结果的 ID: {local_first_result_id}")
```

#### 3. 通过 ID 检索语句组

使用 `service.get_by_id()`。（需要 `service_instance` 和 `local_first_result_id`）。

```python
# 使用从本地搜索结果获得的 ID
item_details_local = service_instance.get_by_id(
    group_id=local_first_result_id
)

name_details_local = (item_details_local.primary_declaration.lean_name
                      if item_details_local.primary_declaration else "N/A")
print(f"\n本地语句组 ID {item_details_local.id} 的详细信息: 名称: {name_details_local}")
# print(f"  语句: {item_details_local.statement_text}")
```

#### 4. 获取依赖关系

使用 `service.get_dependencies()`。（需要 `service_instance` 和 `local_first_result_id`）。

```python
# 使用从本地搜索结果获得的 ID
deps_response_local = service_instance.get_dependencies(
    group_id=local_first_result_id
)

print(f"\n组 ID {deps_response_local.source_group_id} 的本地依赖关系\n  （找到 {deps_response_local.count} 个）：")
for citation_local in deps_response_local.citations:
    name_deps_local = (citation_local.primary_declaration.lean_name
                       if citation_local.primary_declaration else "N/A")
    print(f"  - 依赖 ID: {citation_local.id}, 名称: {name_deps_local}")
```

### CLI 内部使用

LeanExplore CLI 命令 `leanexplore chat --backend local` 和 `leanexplore mcp serve --backend local` 使用此 `lean_explore.local.service.Service` 类与您的本地数据工具链交互。