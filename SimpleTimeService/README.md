# SimpleTimeService

- 版本：`1.0.20260826`
- 源码：https://github.com/FasterEdge/SimpleTimeService
- 默认端口：8080

提供本机 UTC 时间、偏移时间以及 `pool.ntp.org` NTP 时间，响应同时包含三种 DateTime 大小写字段。

## 启动

```bash
go run . --port 8080 --offset 100ns
```

| 参数 | 默认值 | 说明 |
|---|---:|---|
| `--port` | 8080 | 监听端口 |
| `--offset` | 0 | Go duration，例如 `100ns`, `50ms`, `1s` |

## 文档

- [API](api.md)
- [部署](deployment.md)
