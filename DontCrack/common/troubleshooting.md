# DontCrack 故障排查

## 配置检查失败：Path 不存在

- 使用绝对路径。
- 检查文件是否为普通文件。
- 检查运行 DontCrack 的系统账户是否可访问路径。

## permission denied

Linux/Android/OH：

```bash
chmod 755 /path/to/program
```

同时检查 SELinux、挂载点 `noexec` 和 systemd sandbox。

## `/shutdown` 后又重启

当前版本 `1.0.20260826` 已通过 `StoppedByRequest` 阻止主动停止后的自动重启。确认运行的是新二进制。

## 探针持续失败

- 手动在相同用户/环境下运行 `-probe-cmd`。
- Android 检查 shell 路径。
- 增大 `-probe-timeout` 或 `-probe-failure-limit`。

## heartbeat 没有日志

heartbeat 会取走并清空内存日志。启用 `-file-log`，或降低监控抓取频率。

## 达到最大重试次数

`restart_count` 达到 `-max-retries` 后停止。调用 `/startup` 会重置计数并重新启动。
