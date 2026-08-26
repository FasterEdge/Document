# DontCrack HTTP API

管理地址示例：`http://127.0.0.1:11883`

配置 `-password` 后，请求需携带 `?password=<secret>`。现有错误认证行为为返回根路径文本，而不是标准 401；调用方应检查响应内容。

| 路径 | 方法 | 说明 |
|---|---|---|
| `/` | 任意 | 返回平台横幅 |
| `/ui`, `/ui/` | GET | 内嵌 Web 控制台 |
| `/healthz` | GET | 子进程运行返回 200，否则 503 |
| `/readyz` | GET | 子进程就绪返回 200，否则 503 |
| `/metrics` | GET | Prometheus 文本指标 |
| `/startup` | GET/POST | 重置重试次数并启动子进程 |
| `/heartbeat` | GET | 返回状态并取走当前日志缓存 |
| `/shutdown` | GET/POST | 主动停止子进程，不触发自动重启 |

## heartbeat 响应

```json
{
  "version": "1.0.20260826",
  "state": "running",
  "info": "进程管理器正常运行",
  "timestamp": "2026-08-26 12:00:00",
  "logs": ["[STDOUT] started"],
  "process_pid": 1234,
  "process_path": "/usr/local/bin/service",
  "restart_count": 0,
  "file_type": "binary_executable",
  "last_exit_code": 0,
  "program_args": "--port 8080",
  "extra_env_raw": "FOO=bar"
}
```

`/heartbeat` 会清空已返回的内存日志；监控系统若需要保留日志，应同步启用 `-file-log` 或外部日志采集。

## 示例

```bash
curl "http://127.0.0.1:11883/heartbeat?password=<secret>"
curl -X POST "http://127.0.0.1:11883/shutdown?password=<secret>"
curl -X POST "http://127.0.0.1:11883/startup?password=<secret>"
```
