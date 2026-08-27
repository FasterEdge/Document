# 安全基线

FasterEdge 组件可以执行命令、传输文件、代理网络请求、管理容器或控制远程进程。生产部署必须明确可信边界。

## 通用要求

1. 不把示例密码用于生产，密钥使用随机生成并通过安全配置注入。
2. 管理端口默认仅监听可信网络，公网暴露时使用 TLS、VPN、防火墙或反向代理。
3. 日志、命令参数、环境变量和 `/heartbeat` 响应可能包含路径或敏感信息，分享前脱敏。
4. 以最低权限账户运行；只有确实需要 init/systemd/驱动权限时才使用 root/SYSTEM。
5. 生产环境固定依赖版本并保留升级/回滚路径。

## 组件风险重点

| 组件 | 重点风险 | 最低措施 |
|---|---|---|
| FasterEdge | 命令执行、外部 Transport、SSRF | 命令 allowlist；校验目标；最小 Transport 权限 |
| DontCrack | 启停任意进程、日志/环境暴露 | 设置 `-password`；限制管理端口；使用最小系统账户 |
| ProxyArea | SSRF、开放代理、查询串密钥泄露 | 设置 `--key` 和 `--allow-hosts`；使用 Bearer/X-Proxy-Key；启用 TLS；限制网络入口 |
| SimpleWebShell | 远程 Shell、文件读写 | 强密钥、TLS/VPN、非 root、网络白名单、审计日志 |
| SimpleTimeService | 外部 NTP 依赖 | 限制服务入口，配置 NTP 出站策略 |
| TsnHub | IPC/OPC UA 控制 | 限制 socket/pipe 权限和 OPC UA 身份 |

## 查询串凭证

现有 DontCrack 和 SimpleWebShell 接口，以及 ProxyArea 的兼容接口，可以用 query 参数传递密码或密钥。ProxyArea 已支持并优先采用 `Authorization: Bearer` 与 `X-Proxy-Key`，query `key` 仅用于旧客户端兼容。查询串会进入浏览器历史、访问日志和代理日志。部署时：

- 强制 HTTPS 或可信内网。
- 关闭或脱敏 URL 日志。
- 不在截图、文档和监控标签中展示真实密钥。
- 对 ProxyArea 使用 Bearer 或 X-Proxy-Key，避免在 URL 中携带 key；其他组件如需保留 query 兼容，应规划 Header 认证迁移窗口。

## 私有资料边界

`PrivateDocument` 属于内部专利、软著和登记资料，禁止复制到公开 `Document` 仓库或公开示例。
