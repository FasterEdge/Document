# SimpleTimeService API

| 路径 | 方法 | 说明 |
|---|---|---|
| `/time` | GET | 当前本机 UTC |
| `/offset_time` | GET | 本机 UTC + `--offset` |
| `/ntp_time` | GET | `pool.ntp.org` UTC |
| `/offset_npt_time` | GET | NTP UTC + `--offset` |

`/offset_npt_time` 中的 `npt` 是现有兼容拼写，不应静默改名。若未来增加正确拼写 `/offset_ntp_time`，应保留旧接口一段兼容期。

## 响应

```json
{
  "DateTime": "2026-08-26T12:00:00.123456789Z",
  "dateTime": "2026-08-26T12:00:00.123456789Z",
  "datetime": "2026-08-26T12:00:00.123456789Z"
}
```

## 失败

NTP 查询失败返回 HTTP 500：

```json
{"error":"failed to fetch ntp time"}
```

当前 NTP 地址固定为 `pool.ntp.org`，未暴露服务器和超时配置。
