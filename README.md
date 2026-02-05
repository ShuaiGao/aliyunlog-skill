## aliyunlog-sls-search 技能

该技能用于查询阿里云 SLS（Simple Log Service）日志，支持日志查询与统计，并可选图表化展示。

### 功能

- 查询日志（`get_logs`）
- 统计分布（`get_histograms`）
- 慢查询分析流程
- 可视化图表（可选）

### 安装

注意：技能不会自动更新，需要自行关注版本。

**方式一：Claude Code /plugin（推荐）**

本仓库已包含插件市场与插件清单文件，可直接通过 `/plugin` 安装：

```
/plugin marketplace add ShuaiGao/aliyunlog-skill
/plugin install aliyunlog-sls-search@aliyunlog-skill
```

安装后需要重启 Claude Code 才会生效。

**方式二：Codex（手动拷贝）**

```bash
# 1) 克隆仓库
git clone https://github.com/ShuaiGao/aliyunlog-skill

# 2) 项目级
mkdir -p .codex/skills
cp -r ./aliyunlog-skill/skills/aliyunlog-sls-search .codex/skills/

# 3) 用户级（可选）
mkdir -p ~/.codex/skills
cp -r ./aliyunlog-skill/skills/aliyunlog-sls-search ~/.codex/skills/
```

**方式三：Cursor（可以直接读取claude的skills）**

```bash
# 1) 克隆仓库
git clone https://github.com/ShuaiGao/aliyunlog-skill

# 2) 用户级（Cursor 会从 ~/.claude/skills 读取）
mkdir -p ~/.claude/skills
cp -r ./aliyunlog-skill/skills/aliyunlog-sls-search ~/.claude/skills/
```

如果 Cursor 版本未显示技能入口，说明当前版本不支持或支持不完整，可继续使用手动拷贝到 Codex 或 Claude Code /plugin 安装方式。

### 依赖

```bash
pip3 install aliyun-log-python-sdk aliyun-log-cli -U --no-cache
```

可视化推荐：优先生成 HTML 图表，并建议安装可视化技能：
```
https://github.com/nextlevelbuilder/ui-ux-pro-max-skill
```

### 阿里云 CLI 配置

```bash
aliyunlog configure "<AccessKey ID>" "<AccessKey Secret>" <Endpoint>
```

常见 Endpoint：
- cn-hangzhou.log.aliyuncs.com
- cn-shanghai.log.aliyuncs.com
- cn-beijing.log.aliyuncs.com

### 技能配置

技能会自动搜索 `sls-config.md`（按顺序）：
1. `./sls-config.md`
2. `./docs/sls-config.md`
3. `./.claude/sls-config.md`
4. 递归搜索项目目录下其他 `sls-config.md`（优先最短路径；同层级多个则询问选择）
5. 在当前会话内缓存已选路径；除非用户切换项目或要求更换配置，否则不重复搜索

未找到时会创建配置文件模板，并可先用 CLI 自动预填项目与日志库：
- `aliyunlog log list_project`
- `aliyunlog log list_logstore --project_name="<PROJECT>"`
确认后写入 `sls-config.md` 再执行查询。模板参考：`skills/aliyunlog-sls-search/sls-config-template.md`。

**Log Format 可选**：默认可以不填写；如果不填，按 `glog` 作为默认格式处理。推荐在 Service Mappings 里新增 `log_format` 列；当明确某个 logstore 被使用时，再探索其日志结构后补全。探索重点：
1. 日志等级字段（debug / info / warn / error）
2. 日志类型字段（慢查询 / db 报错 / 异常 / http 请求 / rpc 等）
3. trace id 的提取方式

**最小配置示例：**

```yaml
project: "your-project-name"
logstore: "your-logstore-name"
endpoint: "cn-hangzhou.log.aliyuncs.com"
```

### 使用示例

查询错误日志：
```
查询最近 30 分钟错误日志
```

统计错误分布：
```
展示今天错误日志分布
```

慢查询分析：
```
分析过去 1 小时慢查询
```

示例（结合代码分析）：
```
通过查询 SLS 日志，统计今日报错最多的日志，展示前 10 条并结合代码位置分析优化建议
```

### 可视化

统计类查询可选择展示方式：
- Chart（默认，HTML 图表）
- Table（Markdown 表格）

图表输出目录：`./run/sls/`。

执行 `aliyunlog` 查询时会输出完整的请求 payload；绘图时使用完整 JSON 输出作为数据源。如果 JSON 结构不适配图表，将在 HTML 内用 JavaScript 进行转换后再绘制。


### 故障排查

命令不存在：
```bash
pip3 install aliyun-log-cli -U --no-cache
```

认证失败：
```bash
aliyunlog configure "<AccessKey ID>" "<AccessKey Secret>" <Endpoint>
```

权限不足需具备：
- `log:GetLogs`
- `log:GetHistograms`

## 目录结构

```
.claude-plugin/
├── marketplace.json        # 插件市场入口
└── plugin.json             # 插件清单

skills/aliyunlog-sls-search/
├── SKILL.md                # 模型执行规则
├── get_logs.md             # get_logs 文档
├── get_histograms.md       # get_histograms 文档
├── sls-config-template.md  # 配置模板
```
