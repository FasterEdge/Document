# ProxyArea API

## 端点

| 端点 | 接受方法 | 上游方法 |
|---|---|---|
| `/` | GET | 不转发，返回版本 |
| `/get` | 任意进入 handler | 固定 GET |
| `/post` | 任意进入 handler | 固定 POST |
| `/proxy` | 任意 | 保持原方法 |
| `/proxy/...` | 任意 | 保持原方法 |

`/proxy` 可透传 GET、POST、PUT、PATCH、DELETE、HEAD、OPTIONS 等方法；上游是否支持由上游服务决定。

## 控制参数

| 参数 | 来源 | 说明 |
|---|---|---|
| `url` | query/form | 必填，目标 URL |
| `key` | query/form | 与 `--key` 匹配 |
| `params` | query/form | 追加到目标 URL 的原始 query |
| `https` | query/form | `true` 时为无协议 URL 补 `https://` |

## GET

```bash
curl "http://127.0.0.1:8080/get?url=https://api.example.com/data&key=<secret>"
```

## POST

建议把控制字段放 query，把业务 body 放请求体，避免 form 解析和透传语义混合：

```bash
curl -X POST \
  "http://127.0.0.1:8080/post?url=https://api.example.com/items&key=<secret>" \
  -H "Content-Type: application/json" \
  -d '{"name":"sensor-1"}'
```

## 通用方法

```bash
curl -X PATCH \
  "http://127.0.0.1:8080/proxy?url=https://api.example.com/items/1&key=<secret>" \
  -H "Content-Type: application/json" \
  -d '{"enabled":true}'

curl -X DELETE \
  "http://127.0.0.1:8080/proxy?url=https://api.example.com/items/1&key=<secret>"
```

## Header 和响应

- 转发非 hop-by-hop 请求头。
- 不转发 Host、Content-Length、Connection、Transfer-Encoding 等头。
- 上游响应状态码和非 hop-by-hop 响应头原样返回。
- body 使用流式 `io.Copy`，避免一次性读取整个响应。

## 错误

```json
{"error":"缺少 url 参数"}
```

常见状态：400 参数错误、401 密钥错误、403 白名单拒绝、502 上游请求失败。
