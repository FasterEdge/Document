# DontCrack for ManyLinux

- 源码：https://github.com/FasterEdge/DontCrack4ManyLinux
- 平台：常见 Linux 发行版，amd64/arm64/arm
- shell：`/bin/sh`
- 默认日志：`./logs/proc_manager/`

## 构建

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags='-s -w' -o DontCrack .
```

## systemd

仓库提供 `example/dontcrack-edgecore.service`：

```bash
sudo cp example/dontcrack-edgecore.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now dontcrack-edgecore.service
sudo systemctl status dontcrack-edgecore.service
```

调整：

- `User`/`Group` 使用最低权限账户。
- `WorkingDirectory` 和 `ExecStart` 使用绝对路径。
- 日志路径提前创建并授权。
- 根据业务启用 `ProtectSystem`、`ProtectHome`、`PrivateTmp` 等 hardening。

## Kubernetes

用 `/healthz` 和 `/readyz` 配置探针，用 `/metrics` 供 Prometheus 抓取。若配置 `-password`，需通过代理注入凭证或仅在 sidecar/本机网络访问。
