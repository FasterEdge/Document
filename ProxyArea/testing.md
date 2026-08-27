# ProxyArea 测试

自动验证：

```bash
gofmt -d .
go vet ./...
go test ./...
go test -race ./...
go test -cover ./...
```

`proxy_test.go` 使用 `httptest` 覆盖旧路由、通用路由、七个别名和扩展方法；各方法二进制 body；query/form/multipart/envelope；Bearer/X-Proxy-Key 认证优先级；URL 单次解码和重复参数；动态 hop-by-hop Header；多值响应；白名单重定向；错误与超时映射。

手工测试：

```bash
go run ./examples/testserver -addr=:8081
go run . --addr=:8080 --key=test-secret --allow-hosts=127.0.0.1,localhost
curl -X PROPFIND -H 'Authorization: Bearer test-secret' \
  'http://127.0.0.1:8080/proxy?url=http%3A%2F%2Flocalhost%3A8081%2Ftest%2Fall' -d 'payload'
curl -X GET -H 'X-Proxy-Key: test-secret' \
  'http://127.0.0.1:8080/proxy/patch?url=http%3A%2F%2Flocalhost%3A8081%2Ftest%2Fall' -d 'patch-body'
```

测试服务器回显原始 body 文本与 base64、方法、query 和 Header，并提供 `/redirect?to=...`、`/delay?duration=100ms`、`/headers`。仓库 `test_usage.md` 包含 form、multipart、四种 envelope 编码及预期错误的完整命令。
