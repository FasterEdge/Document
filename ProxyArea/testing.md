# ProxyArea 测试

仓库提供 `examples/testserver/main.go`，会回显 method、path、query、header 和 body。

```bash
# 上游测试服务
go run examples/testserver/main.go -addr=:8081

# 代理
go run . --addr=:8080 --key=test-secret --allow-hosts=127.0.0.1
```

## 测试矩阵

```bash
curl "http://127.0.0.1:8080/get?url=http://127.0.0.1:8081/test/get&key=test-secret"

curl -X POST \
  "http://127.0.0.1:8080/post?url=http://127.0.0.1:8081/test/post&key=test-secret" \
  -H 'Content-Type: application/json' -d '{"x":1}'

curl -X PUT \
  "http://127.0.0.1:8080/proxy?url=http://127.0.0.1:8081/test/put&key=test-secret" \
  -d 'payload'

curl "http://127.0.0.1:8080/get?url=http://example.net/&key=test-secret"
# 预期：403，不在 allow-hosts
```

同时测试缺 key、错误 key、空 URL、超时、TLS 上游和大响应流式转发。
