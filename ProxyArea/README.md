# ProxyArea

- 版本：`1.0.20260826`
- 源码：https://github.com/FasterEdge/ProxyArea
- 实现：Go 标准库 `net/http`，免 CGO

ProxyArea 是完整保留 HTTP 方法、body、端到端 Header 与上游响应的 REST 兼容转发器。提供旧版 `/get`、`/post`，通用 `/proxy`、`/proxy/`，以及七个精确方法别名。

```bash
ProxyArea --addr=:8080 --key=<replace-me> --allow-hosts=api.example.com,192.0.2.20 --timeout=30s
```

生产环境必须设置 `--key` 和 `--allow-hosts`，优先使用 `Authorization: Bearer` 或 `X-Proxy-Key`，并启用 TLS/可信反向代理。

## 文档

- [API](api.md)：路由、控制来源、envelope、错误
- [安全](security.md)：认证、SSRF、白名单与重定向
- [部署](deployment.md)：CLI、Docker、TLS
- [测试](testing.md)：自动和手工矩阵

CLI 参数保持兼容：`--addr`、`--key`、`--https`、`--cert_file`、`--key_file`、`--timeout`、`--target-scheme`、`--allow-hosts`、`--insecure-skip-verify`。
