# SimpleWebShell 部署

## DontCrack 托管

```bash
DontCrack \
  -path /usr/local/bin/SimpleWebShell \
  -args "-key <secret> -shell /bin/bash -port 8878" \
  -start-now -auto-restart \
  -port 11885
```

## 反向代理

推荐入口链路：

```text
管理员 → VPN / TLS Reverse Proxy → SimpleWebShell:8878
```

反向代理至少配置：

- TLS。
- IP allowlist。
- 请求体/文件大小上限。
- 访问日志 key 脱敏。
- 连接、读取和上传超时。

## Windows

使用 `cmd` 或 PowerShell 包装脚本。作为服务运行时确认工作目录和控制台行为，建议 NSSM 或受控计划任务。
