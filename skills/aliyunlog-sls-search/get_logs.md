# get_logs Command

Query and retrieve log entries with support for SQL-like analysis statements.

## When to Use

Use `get_logs` when user wants:
- Actual log content/entries
- Detailed error messages
- SQL-like analysis queries
- Aggregated statistics

## Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| project | Yes | SLS Project name (from config) |
| logstore | Yes | Logstore name (from config) |
| fromTime | Yes | Start time (YYYY-MM-DD HH:mm:ss) |
| toTime | Yes | End time (YYYY-MM-DD HH:mm:ss) |
| query | Yes | Query statement with optional SQL analysis |
| topic | No | Log topic filter (default: `""`) |
| line | No | Max logs to return (default: `100`) |
| offset | No | Pagination offset (default: `0`) |
| reverse | No | Reverse chronological order (default: `true`) |

## Command Format

```bash
aliyunlog log get_logs \
  --request='{"project":"<PROJECT>","logstore":"<LOGSTORE>","fromTime":"<FROM_TIME>","toTime":"<TO_TIME>","query":"<QUERY>","topic":"","line":"<LINE>","offset":"<OFFSET>","reverse":"<REVERSE>"}' \
  --format-output=json
```

Replace placeholders with actual values from `sls-config.md` or user input.

### Parameter Placeholders

Use these when the exact query condition is project-specific:
- `{{CONFIG:<key>}}`: read from `sls-config.md`
- `{{ASK:<key>}}`: ask the user, then write the answer back to `sls-config.md`

If a `{{CONFIG:<key>}}` value is missing, ask the user for it and persist to `sls-config.md` before querying.

## Query Syntax

### Basic Filters (Project-Specific)
```
<LEVEL_FIELD>:<ERROR_LEVEL>
<KEY_FIELD>:"<KEYWORD>"
```

### Analysis Queries (SQL-like)

Use `|` to separate filter and analysis:

```
{{CONFIG:base_filter}} | SELECT COUNT(*) as count
{{CONFIG:base_filter}} | SELECT <FIELD>, COUNT(*) as pv GROUP BY <FIELD> ORDER BY pv DESC LIMIT 10
{{CONFIG:error_filter}} | SELECT __time__, message LIMIT 100
```

## Examples

### Example 1: Query by a configured filter
User: "查询最近 30 分钟的错误日志"

```bash
aliyunlog log get_logs \
  --request='{"project":"<PROJECT>","logstore":"<LOGSTORE>","fromTime":"<FROM_TIME>","toTime":"<TO_TIME>","query":"{{CONFIG:error_filter}}","topic":"","line":"<LINE>","offset":"<OFFSET>","reverse":"<REVERSE>"}' \
  --format-output=json
```

### Example 2: Aggregate by a user-specified field
User: "统计各状态码的数量"

```bash
aliyunlog log get_logs \
  --request='{"project":"<PROJECT>","logstore":"<LOGSTORE>","fromTime":"<FROM_TIME>","toTime":"<TO_TIME>","query":"{{CONFIG:base_filter}} | SELECT {{ASK:group_by_field}}, COUNT(*) as count GROUP BY {{ASK:group_by_field}} ORDER BY count DESC","topic":"","line":"<LINE>","offset":"<OFFSET>","reverse":"<REVERSE>"}' \
  --format-output=json
```

### Example 3: Slow query (project-specific)
User: "查看慢查询耗时分布 / 最慢的10条查询"

**Note:** Slow query filters and duration fields are project-specific.  
If the configuration doesn't define them yet:
1. Ask the user for the slow-query filter and duration field
2. Write the answer into `sls-config.md`
3. Use those values in the query

## Result Parsing

### Standard Log Fields
- `__time__`: Log timestamp
- `log_level`: Log level (info, warn, error)
- `message`: Log message content

### Slow Query Message Format
Slow-query log schema is project-specific. Prefer a config-defined parser or field map from `sls-config.md`.  
If missing, ask the user how duration / SQL / code location / rows are represented, then update `sls-config.md`.

Present slow query results in table format:

| Duration(ms) | Time | Code Location | Rows |
|--------------|------|---------------|------|
| xxx | HH:mm | file:line | rows |

## Output Rules

- Always print the full `aliyunlog` request payload used for the query.
- For charting, output a single HTML file (no external template).
- Inline the full JSON output as `window.__SLS_DATA__ = <JSON>;`
- Use the chart script to read `window.__SLS_DATA__` and transform fields if needed.
