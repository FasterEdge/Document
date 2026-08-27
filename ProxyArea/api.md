# ProxyArea API

## 路由

| 路由 | 上游方法 |
|---|---|
| `/get`、`/post` | 固定 GET、POST |
| `/proxy`、`/proxy/` | 保持客户端方法（含 PROPFIND 等扩展方法） |
| `/proxy/get`、`post`、`put`、`patch`、`delete`、`head`、`options` | 路径指定的方法 |

别名可由任意客户端方法调用；未知或更深的 `/proxy/...` 返回 404。所有方法均按字节保留 body，包括 GET、HEAD、OPTIONS、DELETE。空 body 不自动产生 Content-Type。

## 控制字段与优先级

字段：`url`（必填）、`params`、`https`、`key`。非认证字段优先级为 **query → 显式 envelope → urlencoded/multipart form**。认证优先级为 **Bearer → X-Proxy-Key → query → envelope → form**。字段一旦在较高优先级来源出现，空值或错误值也不回退。

普通 `application/json` 始终作为业务 JSON 转发。form/multipart 可携带控制字段，同时原始 body（包括控制字段）完整转发；业务表单建议使用 query/header 控制。

```bash
curl -X PATCH 'http://127.0.0.1:8080/proxy?url=https%3A%2F%2Fapi.example.com%2Fitems%2F1' \
  -H 'Authorization: Bearer <secret>' -H 'Content-Type: application/json' \
  -d '{"enabled":true}'
```

## JSON envelope

媒体类型 `application/vnd.proxyarea.proxy+json`，顶层支持 `url`、`params`、`https`、`key`、`encoding`、`body`、`contentType`。

```json
{"url":"https://api.example.com/items","encoding":"json","body":{"name":"sensor-1"},"contentType":"application/json"}
```

`encoding`：`none`（默认且不能带 body）、`json`、`text`、`base64`。未知字段、额外 JSON 值及无效 encoding/base64/contentType 被拒绝。控制 body 最大 8 MiB、表单最多 1024 个字段。

## URL 与 Header

目标仅限绝对 http/https；无协议时按 `https` 或 `--target-scheme` 补齐。拒绝 userinfo、控制字符、缺 hostname、非法端口。`params` 通过标准解析合并，保留重复键且只编码一次。

双向移除标准及 `Connection` 动态 hop-by-hop Header；不转发 Host、Content-Length、Authorization、X-Proxy-Key；保留 Set-Cookie 等多值端到端 Header。每次重定向重新检查 scheme/白名单，默认最多 10 跳。

## 错误

统一 JSON：`{"error":"..."}`。400 控制/URL 错误；401 认证失败；403 初始目标白名单拒绝；404 未知别名；413 控制 body 超限；502 上游/重定向失败；504 超时。
