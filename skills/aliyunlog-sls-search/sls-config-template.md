# SLS Configuration Template

project: "your-project-name"
logstore: "your-logstore-name"
endpoint: "cn-hangzhou.log.aliyuncs.com"
line: 100
offset: 0
reverse: true
log-tool: glog

## Service Mappings

| alias | project       | logstore     | log_format | description | filter              |
|------|---------------|--------------|-----------|-------------|---------------------|
| myapp| my-log-project | my-app-logs  | glog      | My service  | service_id: myapp01 |

---

## Log Format (Optional)

### glog (default)

Log Levels:
- level: debug / info / warn / error

Special Log Types:
- Slow query logs: duration_query > 0, level: warn
- Exception logs: level: error

Trace ID:
- Field: x-trace-id
- Example query: "x-trace-id": "abc123xyz"

HTTP Request Related:
- Keywords: recv, send, HTTPREQUEST, HTTPRESPONSE

---

### Request Chain Analysis

Use trace_id to find all related logs across components. Identify the log component type first, then query by its trace field.

---

## Custom Query Patterns

payment_timeout: service:payment AND message:"timeout" AND level:error
login_fail: uri:"/login" AND status:>=400
