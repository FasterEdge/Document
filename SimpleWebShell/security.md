# SimpleWebShell 安全

SimpleWebShell 等价于远程 Shell 与远程文件系统入口。密码保护不等于加密传输。

## 生产基线

- 仅监听管理网络、VPN 或 loopback。
- 通过 HTTPS 反向代理提供 TLS。
- 使用长随机 key，不写入仓库、脚本或截图。
- 以专用低权限用户运行，限制 sudo、文件权限、网络出口和设备访问。
- 反向代理限制请求体、上传文件大小、超时和连接数。
- 记录审计事件但不记录 key、完整环境和敏感命令输出。

## 暴露信息

Session 详情可能包含：环境变量、用户/组、主机名、路径、Git 分支、系统资源。不要向非管理员开放 `session_get`。

## Query key

key 会进入 URL 历史和访问日志。反向代理需脱敏 query，长期建议迁移到 Authorization header。

## 合法用途

仅用于自己拥有或明确授权的系统；禁止作为未授权访问工具。
