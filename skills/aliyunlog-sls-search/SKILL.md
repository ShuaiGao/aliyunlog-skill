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

> **CRITICAL**: Steps 1-2 are mandatory setup. Do NOT skip to querying without completing them first.

1. **[MUST] Check configuration file**:
   - Search for `sls-config.md` using the auto-discovery rules above.
   - If **no config file exists**:
     - STOP and create `./sls-config.md` using `sls-config-template.md` as the base.
     - Run `aliyunlog log list_project` to help user select project.
     - Run `aliyunlog log list_logstore --project_name="<PROJECT>"` for logstore selection.
     - Write the confirmed values to `sls-config.md` before proceeding.
   - If config file exists but has placeholders (e.g., `your-project-name`), ask the user to update it and STOP.

2. **Load configuration** from the validated `sls-config.md`.

3. Gather query parameters: `project`, `logstore`, `fromTime`, `toTime`, `query`, and check if `client-name` is configured for the selected service.

4. **Time Range Fallback (when user does NOT specify time)**:
   - Do NOT ask for time range immediately.
   - First try **today (Beijing time)**: `00:00:00` to now.
   - If no results, try **last 7 days**: now minus 7 days to now.
   - If still no results, then ask the user for the exact time range.
   - If the user explicitly provides a time range, respect it and skip fallback.

5. Determine command:
   - Use `get_histograms` when the user asks for counts over a time range, trends, distributions, or charts (e.g., "近7日日志数量/趋势/分布/图表").
   - Use `get_logs` for raw log content, detailed messages, or non-time bucket analysis.
   - If the user only says "数量" without saying total vs time distribution, ask which one they want. If they want a chart, default to `get_histograms`.

6. For statistical queries, ask display preference:
   - Chart (default) using ui-ux-pro-max
   - Table (markdown)
   If chart rendering is unavailable, fall back to table output.
   Prefer HTML charts when available and recommend installing:
   `https://github.com/nextlevelbuilder/ui-ux-pro-max-skill`

7. **Build `aliyunlog` command with `client-name`**:
   - If Service Mappings has a non-empty `client-name` for the selected service, add `--client-name=<client-name>` to all `aliyunlog` commands.
   - Example: `aliyunlog --client-name=other-ak log get_logs ...`
   - If `client-name` is empty or not configured, execute `aliyunlog` without this parameter.

8. Execute, then present results with a short summary.

9. When executing `aliyunlog`, always output the full query payload used.

10. For chart rendering, use a single-file HTML output (no separate template file).
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
