# SimpleWebShell

- 版本：`1.0.20260826`
- 源码：https://github.com/FasterEdge/SimpleWebShell
- 默认端口：8878

SimpleWebShell 提供受 key 保护的远程命令、会话与文件传输。它具有直接执行 Shell 和读写文件的高权限能力，只适用于明确授权的运维场景。

## 启动

```bash
./SimpleWebShell -key <replace-with-strong-secret> -port 8878 -shell /bin/bash
```

Windows 推荐：

```cmd
SimpleWebShell.exe -key <replace-with-strong-secret> -port 8878 -shell cmd
```

## 参数

| 参数 | 默认值 | 说明 |
|---|---:|---|
| `-key` | 空 | 必填访问密钥 |
| `-shell` | `/bin/bash` | Shell 路径 |
| `-port` | `8878` | HTTP 监听端口 |

## 文档

- [API](api.md)
- [安全](security.md)
- [部署](deployment.md)

## 会话

Session 保存当前目录、环境快照、Shell、历史、用户/组、主机、CPU/内存/磁盘和 Git 分支等信息。命令成功后写入限长历史。
