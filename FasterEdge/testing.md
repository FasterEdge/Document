# 测试与质量

在 FasterEdge 主仓库执行：

```bash
go test ./...
go test -race ./...
go test ./... -run Integration -v
go vet ./...
```

## 测试类型

- 单元测试：每个 Data/Ability 的命令和参数校验。
- 集成测试：跨能力组合、注册和生命周期。
- Race：共享状态与并发命令。
- Fuzz：地址、令牌、协议和参数解析。
- Benchmark：关键注册、查找和命令路径。

## 新组件最低要求

1. 正常路径测试。
2. nil、错误类型、空白必填字段测试。
3. 并发读写测试。
4. Runner 取消和卸载测试。
5. Transport 错误与超时测试。
