# SimpleWebShell API

所有接口使用 `key` 参数认证。

| 路径 | 方法 | 参数 | 说明 |
|---|---|---|---|
| `/` | GET | `key` | 正确 key 返回 UI；错误 key 返回版本文本 |
| `/get` | GET | `key`, `cmd`, `session?` | 执行命令 |
| `/post` | POST | `key`, `cmd`, `session?` | JSON/Form 命令 |
| `/get_current_path` | GET | `key`, `session?` | 当前目录 |
| `/file_send` | POST | `key`, multipart `file`, `path?` | 上传文件 |
| `/file_receive` | GET | `key`, `path` | 下载文件 |
| `/session_create` | GET | `key` | 新建 session |
| `/session_list` | GET | `key` | session 列表 |
| `/session_delete` | GET | `key`, `session` | 删除 session |
| `/session_get` | GET | `key`, `session` | session 详细 JSON |

## GET 命令

`cmd` 为 URL 编码后的命令：

```bash
curl "http://127.0.0.1:8878/get?key=<secret>&cmd=pwd"
```

## POST 命令

```bash
curl -X POST http://127.0.0.1:8878/post \
  -d 'key=<secret>' -d 'cmd=uname -a'
```

## 上传/下载

```bash
curl -X POST "http://127.0.0.1:8878/file_send?key=<secret>" \
  -F 'file=@./artifact.bin' -F 'path=/tmp/artifact.bin'

curl -o artifact.bin \
  "http://127.0.0.1:8878/file_receive?key=<secret>&path=/tmp/artifact.bin"
```

实现目前未设置明确上传/下载大小上限，生产反向代理应配置限制。
