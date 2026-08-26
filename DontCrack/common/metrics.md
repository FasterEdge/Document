# DontCrack Prometheus 指标

访问：

```bash
curl "http://127.0.0.1:11883/metrics?password=<secret>"
```

| 指标 | 类型 | 说明 |
|---|---|---|
| `dontcrack_up` | gauge | 管理器正在响应，固定 1 |
| `dontcrack_process_state` | gauge | 子进程运行 1，停止 0 |
| `dontcrack_process_pid` | gauge | 当前 PID，停止时 0 |
| `dontcrack_restart_count` | counter | 当前自动重启计数 |
| `dontcrack_last_exit_code` | gauge | 上次退出码 |
| `dontcrack_last_exit_time_seconds` | gauge | 上次退出 Unix 时间戳 |
| `dontcrack_log_lines` | gauge | 当前内存缓存日志行数 |
| `dontcrack_info` | gauge | 版本、文件类型和进程路径标签 |

## Prometheus 抓取

```yaml
scrape_configs:
  - job_name: dontcrack
    static_configs:
      - targets: ["127.0.0.1:11883"]
    params:
      password: ["<secret>"]
```

查询串可能出现在 Prometheus 配置、日志和 UI 中。生产环境建议仅在可信网络抓取，并限制配置文件权限。
