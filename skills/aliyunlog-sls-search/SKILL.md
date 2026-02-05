---
name: aliyunlog-sls-search
description: Use when querying Aliyun SLS logs for a specific project/logstore, including histograms or detailed log retrieval.
---

# Aliyun SLS Search Skill

## Overview
Execute log queries against Aliyun SLS using configured project/logstore defaults and optional service mappings.

## Supported Commands

- [get_logs](./get_logs.md) - Query and retrieve log entries with support for analysis statements
- [get_histograms](./get_histograms.md) - Get log distribution histogram (count by time intervals)

## Configuration (Required)

**Configuration is stored in `sls-config.md`.** Auto-discovery order:
1. `./sls-config.md` (project root - recommended)
2. `./docs/sls-config.md`
3. `./.claude/sls-config.md`
4. Recursively search the project directory for any other `sls-config.md`
   - Prefer the shortest path (closest to project root)
   - If multiple matches have the same depth, ask the user which one to use
5. Cache the chosen config path for the current session; reuse it unless the user switches project or requests a different config.

If no file exists, create `./sls-config.md` using `./sls-config-template.md`, then help prefill:
- Run `aliyunlog log list_project` to list accessible projects
- Run `aliyunlog log list_logstore --project_name="<PROJECT>"` for the selected project
Ask the user to confirm the chosen project/logstore and write them into `sls-config.md` before proceeding.
Log format is optional by default. Prefer adding a `log_format` column in Service Mappings. When a logstore is identified, explore its log structure and help fill:
- Log level field and values (debug / info / warn / error)
- Log type definitions (slow query / db error / exception / http request / rpc, etc.)
- Trace ID extraction method

## Execution Flow

1. Load configuration from the selected `sls-config.md` (per the auto-discovery rules).
2. If placeholders are detected (e.g., `your-project-name`), ask the user to update the file and stop.
3. Gather query parameters: `project`, `logstore`, `fromTime`, `toTime`, `query`.
4. Determine command:
   - Use `get_histograms` when the user asks for counts over a time range, trends, distributions, or charts (e.g., "近7日日志数量/趋势/分布/图表").
   - Use `get_logs` for raw log content, detailed messages, or non-time bucket analysis.
   - If the user only says "数量" without saying total vs time distribution, ask which one they want. If they want a chart, default to `get_histograms`.
5. For statistical queries, ask display preference:
   - Chart (default) using ui-ux-pro-max
   - Table (markdown)
   If chart rendering is unavailable, fall back to table output.
   Prefer HTML charts when available and recommend installing:
   `https://github.com/nextlevelbuilder/ui-ux-pro-max-skill`
6. Execute, then present results with a short summary.
7. When executing `aliyunlog`, always output the full query payload used.
8. For chart rendering, use a single-file HTML output (no separate template file).
   - Inline the full JSON output as `window.__SLS_DATA__ = <JSON>;`
   - The HTML contains a minimal chart script that reads `window.__SLS_DATA__`
   - If the JSON shape is not directly usable, transform it in the HTML with JavaScript before plotting

## Visualization Output

For charts, save HTML files under `./run/sls/` and provide a file path to open.

## Slow Query Analysis

When user mentions "慢查询", "slow query", or "slow SQL":

- Read slow-query field/filters from `sls-config.md` if present.
- If missing, ask the user for the slow-query condition and duration field.
- Recommended steps: histogram → duration buckets → top N slowest → summarize

See [get_logs.md](./get_logs.md) for examples.
