# ProxyArea

- 版本：`1.0.20260826`
- 源码：https://github.com/FasterEdge/ProxyArea
- 实现：Go 标准库 `net/http`，免 CGO

ProxyArea 将 HTTP 请求转发到指定目标，提供兼容 GET/POST 接口与通用方法透传接口。

## 启动

```bash
ProxyArea \
  --addr=:8080 \
  --key=<replace-me> \
  --allow-hosts=api.example.com,192.0.2.20 \
  --timeout=30s
```

## 文档

- [API](api.md)
- [安全](security.md)
- [部署](deployment.md)
- [测试](testing.md)

## 参数

| 参数 | 默认值 | 说明 |
|---|---:|---|
| `--addr` | `:8080` | 监听地址 |
| `--key` | 空 | 访问密钥，空则不验证 |
| `--https` | false | 本服务启用 HTTPS |
| `--cert_file` | 空 | TLS 证书 |
| `--key_file` | 空 | TLS 私钥 |
| `--timeout` | `30s` | 上游请求超时 |
| `--target-scheme` | `http` | 目标缺少协议时使用的协议 |
| `--allow-hosts` | 空 | 目标主机白名单，逗号分隔 |
| `--insecure-skip-verify` | false | 跳过上游 TLS 校验 |

生产环境必须设置 `--key` 和 `--allow-hosts`。
