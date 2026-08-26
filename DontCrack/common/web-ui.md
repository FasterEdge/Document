# DontCrack Web UI

访问：`http://<host>:11883/ui`

UI 作为单个 HTML 文件通过 Go `embed` 编入二进制，不依赖 CDN 或本地静态目录。

## 功能

- 输入 password 并连接。
- 显示版本、路径、文件类型、运行状态、PID、重试次数和退出信息。
- 启动/停止子进程。
- 显示 heartbeat 返回的最近日志。
- 每 3 秒自动刷新。

## 安全

- UI 页面本身可直接打开，操作和数据读取依赖 heartbeat/startup/shutdown 的 password。
- 密码存在当前页面内存；通过 URL `?password=` 传入会留在浏览器历史，优先在输入框手动输入。
- 公网使用时必须通过 HTTPS 或可信反向代理。
