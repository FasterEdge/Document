# ProxyArea 安全

ProxyArea 能访问服务所在网络可达的 URL，部署不当会成为 SSRF 或开放代理入口。

## 强制措施

```bash
ProxyArea \
  --addr=127.0.0.1:8080 \
  --key=<strong-random-secret> \
  --allow-hosts=api.example.com,192.0.2.20
```

- 不设置 `--allow-hosts` 时允许任意目标。
- 不设置 `--key` 时任何客户端可使用代理。
- query key 会进入日志/历史，反向代理必须脱敏。
- 公网入口使用 `--https` 或可信 TLS 反向代理。
- `--insecure-skip-verify` 仅用于开发或受控自签名环境。

## 白名单语义

实现拆分端口后，对主机名做不区分大小写的精确匹配。它不是通配符列表，也不会自动放行子域。

## DNS 重绑定

当前白名单按 URL host 字符串校验，不固定 DNS 解析结果。若攻击者控制允许域名的 DNS，仍可能解析到内部地址。高安全部署还应：

- 在出口防火墙限制目标网段。
- 使用固定 DNS/静态解析。
- 在自定义 Transport 中校验最终拨号 IP。

## 上游响应

代理会把上游响应返回给调用方。不要把拥有内部高权限凭证的 ProxyArea 暴露给低权限用户。
