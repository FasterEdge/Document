# DontCrack 主动健康探针

探针仅在 `-probe-cmd` 非空时启用。

```bash
DontCrack \
  -path /usr/local/bin/service \
  -start-now -auto-restart \
  -probe-cmd "curl -sf http://127.0.0.1:8080/health" \
  -probe-interval 30 \
  -probe-timeout 5 \
  -probe-failure-limit 3
```

## 行为

1. 子进程运行时按间隔执行命令。
2. 成功退出则失败计数归零。
3. 连续失败达到阈值后 Kill 子进程。
4. `monitor` 获取退出结果，再按 `-auto-restart` 和 `-max-retries` 决定是否重启。
5. 子进程未运行时不累计探针失败。

## 平台 shell

- OpenHarmony/ManyLinux：`/bin/sh -c`。
- Android：当前探针代码使用 `/bin/sh -c`，但部分 Android 仅有 `/system/bin/sh`；部署前应验证。启动前命令和脚本执行已具备 Android shell 探测。
- Windows：`cmd.exe /C`。

## 选择探针

优先探测业务真实可用性，例如 HTTP 业务端点、Unix socket 响应或本地命令，而不是只检查 PID。
