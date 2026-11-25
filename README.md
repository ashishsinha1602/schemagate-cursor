# schemagate for Cursor

Text-to-SQL with ~91% fewer schema tokens. schemagate is a local, read-only
MCP server that hands the model only the tables a question needs, and only
the ones the caller may read. Works on PostgreSQL, MySQL, SQLite,
SQL Server, Oracle, Snowflake, BigQuery and DuckDB.

## Benchmarks

| Benchmark | Result |
|---|---|
| BIRD, hidden test set | 74.34 EX |
| BIRD Mini-Dev | #1 in SQLite, PostgreSQL and MySQL |
| UniQL, Oracle | #1 (64.93) |
| LiveSQLBench-Large | listed, 34.58% |
| Spider-ACL (access control) | 0.984 F1 |
| Prompt tokens, 50+ table warehouses | 91% fewer than the full schema |

Every number, with the scripts that produce it, is in
[BENCHMARKS.md](https://github.com/ashishsinha1602/schemagate/blob/main/BENCHMARKS.md).

## Where else it is listed

LangChain and LangGraph ([langchain-schemagate](https://pypi.org/project/langchain-schemagate/),
in the LangChain retrievers table), LlamaIndex
([llama-index-retrievers-schemagate](https://pypi.org/project/llama-index-retrievers-schemagate/)),
Pydantic AI (`schemagate[pydantic-ai]`), a Claude Code plugin and a GitHub
Copilot plugin in the main repo, the official MCP registry, Glama and the
awesome-mcp-servers lists. 44k PyPI downloads since the first release in
September 2026.

Source: [ashishsinha1602/schemagate](https://github.com/ashishsinha1602/schemagate).

## Install

Install the plugin from the Cursor Marketplace. Cursor asks for one value:

| Variable | What to enter |
|---|---|
| `SCHEMAGATE_DATABASE_URL` | A SQLAlchemy URL such as `postgresql+psycopg://user:password@host/db`, or `demo` to try the bundled 42-object sample schema |

The value is stored by Cursor and passed to the server as an environment
variable. It never appears in this repository or in the command line.

The server runs with [uv](https://docs.astral.sh/uv/), which must be on the
PATH: `uvx --from "schemagate[mcp,databases]==1.19.2" schemagate-mcp`. The
first start downloads the package and the database drivers.

Try it on `demo`: ask "which tables hold salary data?" as an anonymous caller,
then again with the `payroll` role.

## What is included

- **MCP server** `schemagate` with nine read-only tools: `select_schema`,
  `list_objects`, `describe_object`, `run_query`, `semantic_query`, `answer`,
  `deep_answer`, `refresh_catalog`, `health`. `run_query` refuses writes,
  more than one statement, and any object the caller may not read.
- **Skill** `schemagate-sql`: the select-schema, write-SQL, run-query
  workflow, including how to report a refusal as a refusal.

Every tool takes `principal` and `roles`. A caller that omits them is
anonymous and sees only unrestricted objects.

## Restrictions and hints

Permissions come from the database's own grants. To add role-based
restrictions, column hints or business rules, point `SCHEMAGATE_CATALOG_CONFIG`
at a JSON file as described in the
[main README](https://github.com/ashishsinha1602/schemagate#readme).

## Trust boundary

Over stdio the server believes the `principal` and `roles` the client sends.
It limits what the model is shown; it is not authentication. For a shared
HTTP deployment, use the OAuth mode described in the main README.

## Privacy

The server runs on your machine. The only network traffic is the package
download from PyPI on first start and your own database connection. No
telemetry. The `answer`, `semantic_query` and `deep_answer` tools call a model
provider only if you configure one on the server.

Apache-2.0.
