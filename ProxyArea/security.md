# ProxyArea 安全

ProxyArea 能访问服务所在网络可达的 URL，部署不当会成为 SSRF 或开放代理入口。

```bash
ProxyArea --addr=127.0.0.1:8080 --key=<strong-random-secret> --allow-hosts=api.example.com,192.0.2.20
```

## 认证

已支持（按优先级）：`Authorization: Bearer`、`X-Proxy-Key`、query、显式 JSON envelope、form 中的 `key`。高优先级 Header 即使为空/格式错误也不回退；密钥使用常量时间比较。Bearer 和 X-Proxy-Key 都是代理控制 Header，不会发给上游。query/form key 仅为兼容方式，可能进入历史或访问日志，不应作为生产首选。

## 白名单与重定向

`--allow-hosts` 为空时允许任意目标；配置后按 `URL.Hostname()` 大小写无关精确匹配，不支持通配符且不会放行子域。初始目标和每次重定向都重新检查 http/https scheme 与白名单，默认最多十跳；重定向请求也移除代理凭据。

白名单基于 URL hostname，不固定 DNS 结果。高安全部署还应使用出口防火墙、固定 DNS/静态解析或校验最终拨号 IP。ProxyArea 有意保留内部服务用途，因此不无条件禁止私网地址。

## Header、TLS 与日志

双向剥离标准 hop-by-hop Header 和 `Connection` 指定的动态 Header，并移除 Host、Content-Length 与代理认证 Header。日志使用脱敏 URL，不记录 key。公网入口应使用 `--https` 或可信 TLS 反向代理；`--insecure-skip-verify` 仅限受控测试。
