# get_histograms Command

Get log distribution histogram (count by time intervals).

## When to Use

Use `get_histograms` when user wants:
- Log count distribution over time
- Error frequency analysis
- Time-based aggregation
- Trend visualization data
- Log counts over a time range (especially when a chart is requested)

## Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| project | Yes | SLS Project name (from config) |
| logstore | Yes | Logstore name (from config) |
| fromTime | Yes | Start time (YYYY-MM-DD HH:mm:ss) |
| toTime | Yes | End time (YYYY-MM-DD HH:mm:ss) |
| query | Yes | Query filter statement |
| topic | No | Log topic filter (default: `""`) |

## Command Format

```bash
aliyunlog log get_histograms \
  --request='{"project":"<PROJECT>","logstore":"<LOGSTORE>","fromTime":"<FROM_TIME>","toTime":"<TO_TIME>","query":"<QUERY>","topic":""}' \
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
```

**Note:** Unlike `get_logs`, `get_histograms` does not support SQL-like analysis queries after `|`.

## Examples

### Example 1: Distribution for a configured filter
User: "查看错误日志的时间分布"

```bash
aliyunlog log get_histograms \
  --request='{"project":"<PROJECT>","logstore":"<LOGSTORE>","fromTime":"<FROM_TIME>","toTime":"<TO_TIME>","query":"{{CONFIG:error_filter}}","topic":""}' \
  --format-output=json
```

### Example 2: Slow query time distribution (project-specific)
User: "查询慢查询的时间分布"

```bash
aliyunlog log get_histograms \
  --request='{"project":"<PROJECT>","logstore":"<LOGSTORE>","fromTime":"<FROM_TIME>","toTime":"<TO_TIME>","query":"{{CONFIG:slow_query_filter}}","topic":""}' \
  --format-output=json
```

### Example 3: All logs distribution (project-specific)
User: "查看所有日志的时间分布"

```bash
aliyunlog log get_histograms \
  --request='{"project":"<PROJECT>","logstore":"<LOGSTORE>","fromTime":"<FROM_TIME>","toTime":"<TO_TIME>","query":"{{CONFIG:all_logs_filter}}","topic":""}' \
  --format-output=json
```

## Result Format

Returns a list of time intervals with counts:

```json
{
  "count": 1234,
  "histograms": [
    {"from": 1705395600, "to": 1705396200, "count": 100},
    {"from": 1705396200, "to": 1705396800, "count": 150}
  ],
  "progress": "Complete"
}
```

## Output Rules

- Always print the full `aliyunlog` request payload used for the query.
- For charting, output a single HTML file (no external template).
- Inline the full JSON output as `window.__SLS_DATA__ = <JSON>;`
- Use the chart script to read `window.__SLS_DATA__` and transform fields if needed.

### Fields
- `count`: Total log count in the time range
- `histograms`: Array of time buckets
  - `from`: Bucket start timestamp (Unix)
  - `to`: Bucket end timestamp (Unix)
  - `count`: Log count in this bucket
- `progress`: Query completion status ("Complete" or "Incomplete")

## Use Cases

### Identify Peak Error Times
Use histogram to find when errors spike, then use `get_logs` to investigate specific logs during those periods.

### Monitor Service Health
Track log volume trends to identify anomalies in service behavior.

### Pre-analysis Before Deep Dive
Get overview of log distribution before running detailed `get_logs` queries.
