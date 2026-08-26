# FasterEdge 主框架

- 版本：`1.0.20260826`
- 源码：https://github.com/FasterEdge/FasterEdge
- 语言：Go

FasterEdge 是一个对称的云边协作组件框架。每个节点可以按需组合 Data 和 Ability，并通过统一 Command 接口调用。

## 快速开始

```go
package main

import (
    "context"

    fasteredge "github.com/FasterEdge/FasterEdge"
)

func main() {
    atom := fasteredge.InitStandardAtom()
    if err := fasteredge.PreRunAtom(atom); err != nil {
        panic(err)
    }
    if err := fasteredge.RunAtom(context.Background(), atom); err != nil {
        panic(err)
    }
}
```

## 文档

- [架构](architecture.md)
- [生命周期](lifecycle.md)
- [Data 组件](data.md)
- [Ability 组件](abilities.md)
- [Transport 注入](transports.md)
- [组合示例](examples.md)
- [测试](testing.md)

## 初始化入口

| 函数 | 用途 |
|---|---|
| `InitAtom()` | 只注册 BaseData 与 BaseAbility，适合完全自定义 |
| `InitStandardAtom()` | 注册常用 Data/Ability 完整骨架 |
| `PreRunAtom(atom)` | 执行组件检查和挂载 |
| `RunAtom(ctx, atom, options...)` | 监督 Runner，取消后逆序卸载 |

## 命令调用

```go
ab, ok := atom.Ability("NetMapAbility")
if !ok {
    panic("NetMapAbility missing")
}
out := ab.Command(atom, "register_peer", ability.NetMapRegisterPeerArgs{
    Name: "edge-2", Address: "192.0.2.20:7000", Role: "edge",
})
if out.Err != nil {
    panic(out.Err)
}
```

所有命令参数为严格类型。传入 nil、错误类型或空白必填字段时，组件返回 `types.ErrInvalidArguments`。
