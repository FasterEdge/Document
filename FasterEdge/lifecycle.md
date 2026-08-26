# 生命周期与优雅退出

## 挂载

`PreRunAtom` 调用 Atom 的 `PreRun`，依次对组件执行检查与挂载。依赖缺失时返回结构化错误，不继续运行。

## 运行

`RunAtom` 只监督实现 `types.Runner` 的 Ability：

- 在 goroutine 中运行 Runner。
- 任一 Runner 返回错误时结束监督。
- 外部 context 取消时开始清理。
- 清理使用新的超时 context，不复用已取消 context。

## 卸载

组件按注册顺序的逆序卸载，降低依赖先于被依赖项退出的风险。

```go
err := fasteredge.RunAtom(
    ctx,
    atom,
    fasteredge.WithShutdownTimeout(5*time.Second),
)
```

## 错误处理

- panic、Runner 错误、卸载失败和超时均应通过 error 返回或汇总。
- 调用方不应只忽略 `CommandOutput.Err`。
- Transport 级连接、重试和限流由具体 Transport 实现负责。
