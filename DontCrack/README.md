# DontCrack 进程管理器

- 版本：`1.0.20260831`
- 平台：OpenHarmony、Android、通用 Linux、Windows

DontCrack 托管一个目标进程，捕获 stdout/stderr，提供自动重启、主动启停、日志缓存/落盘、健康探针、Prometheus 指标和 Web UI。

## 公共文档

- [CLI 配置](common/configuration.md)
- [HTTP API](common/api.md)
- [主动健康探针](common/health-probe.md)
- [Prometheus 指标](common/metrics.md)
- [Web UI](common/web-ui.md)
- [故障排查](common/troubleshooting.md)

## 平台文档

- [OpenHarmony](OpenHarmony.md)
- [Android](Android.md)
- [ManyLinux](ManyLinux.md)
- [Windows](Windows.md)

## 处理流程

```text
启动管理器
  ├── 校验目标路径
  ├── 识别二进制/脚本
  ├── 可选执行 -pre
  ├── 合并 -env
  └── 启动子进程
         ├── 抓取 stdout/stderr
         ├── HTTP 管理和 Web UI
         ├── 健康探针
         └── 退出后按 auto-restart 策略重启
```

`monitor()` 是 `cmd.Wait()` 的唯一调用者，避免双 Wait 竞态。主动 `/shutdown` 会设置 `StoppedByRequest`，因此不会被自动重启再次拉起。
