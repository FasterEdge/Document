# DontCrack CLI 配置

| 参数 | 默认值 | 说明 |
|---|---:|---|
| `-path` | 空 | 必填，目标二进制或脚本路径 |
| `-args` | 空 | 目标进程参数字符串 |
| `-pre` | 空 | 启动前由平台 shell 执行的命令 |
| `-env` | 空 | 追加/覆盖环境变量 |
| `-auto-restart` | false | 子进程退出后自动重启 |
| `-max-retries` | 3 | 最大自动重启次数，`-1` 无限 |
| `-start-now` | false | 管理器启动时立即启动子进程 |
| `-port` | 11883 | HTTP 管理端口 |
| `-password` | 空 | 管理 API 密码，空则不保护 |
| `-log-capacity` | 200 | 内存日志最大行数 |
| `-log-max-line-bytes` | 1048576 | 单行日志扫描上限 |
| `-file-log` | false | 开启日志落盘 |
| `-log-path` | 平台相关 | 日志根目录 |
| `-log-life-day` | 7 | 日志保留天数 |
| `-probe-cmd` | 空 | 健康探针命令，空则禁用 |
| `-probe-interval` | 30 | 探针间隔秒数 |
| `-probe-timeout` | 5 | 单次探针超时秒数 |
| `-probe-failure-limit` | 3 | 连续失败阈值 |

## 环境变量格式

Linux 系列：

```bash
-env "PATH=/usr/local/bin:/usr/bin FOO=bar"
```

Windows：

```cmd
-env "PATH=C:\Windows\System32;FOO=bar"
```

自定义环境位于继承环境之前，因此可以覆盖原有 PATH。

## 参数解析限制

当前 `-args` 使用空白切分，不完整支持含空格的嵌套引号参数。复杂参数建议封装为脚本，或避免在单个参数值中嵌套空格。
