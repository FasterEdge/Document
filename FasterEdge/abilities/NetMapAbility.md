# NetMapAbility 文档

## 1. 简介

`NetMapAbility` 提供对等节点拓扑管理能力：注册、更新、查询、移除对端节点，并生成本节点 + 对等节点的拓扑快照。它本身不负责真实的网络发现，只管理节点上的拓扑元数据表，网卡枚举与拓扑同步可由 NetMapData 或外部 Transport 完成。

## 2. 支持的命令

| 命令字符串 | 常量名称 | 用途 |
|-----------|----------|------|
| `register_peer` | `NetMapCommandRegisterPeer` | 注册一个对等节点（不可重复） |
| `unregister_peer` | `NetMapCommandUnregisterPeer` | 移除一个对等节点 |
| `update_peer` | `NetMapCommandUpdatePeer` | 更新对等节点的地址 / 角色 / 最近活跃时间 |
| `list_peers` | `NetMapCommandListPeers` | 列出全部对等节点（按名称排序） |
| `lookup_peer` | `NetMapCommandLookupPeer` | 按名称或地址精确查询单个对等节点 |
| `get_topology` | `NetMapCommandGetTopology` | 生成本节点 + 对等节点拓扑快照 |

## 3. 参数类型

```go
// register_peer
type NetMapRegisterPeerArgs struct {
    Name    string // 必填，对等节点名
    Address string // 必填，host:port / 纯 IP / 主机名（不含协议前缀与路径）
    Role    string // 可选，角色标识
}

// update_peer —— 零值字段不会被应用
type NetMapUpdatePeerArgs struct {
    Name          string // 必填，对等节点名
    NewAddress    string // 可选，空则不修改
    NewRole       string // 可选，空则不修改
    TouchLastSeen bool   // 为 true 时刷新 LastSeen 为当前时间
}

// unregister_peer / lookup_peer
type NetMapLookupPeerArgs struct {
    Name    string // lookup 时可选，优先按名称精确匹配
    Address string // lookup 时可选，名称未命中时按地址匹配
}
```

## 4. 返回值

| 命令 | `CommandOutput.Value` |
|------|----------------------|
| `register_peer` | `NetMapPeer{Name, Address, Role, LastSeen}` |
| `unregister_peer` | 被移除前的 `NetMapPeer` |
| `update_peer` | 更新后的 `NetMapPeer` |
| `list_peers` | `[]NetMapPeer`（按名称排序） |
| `lookup_peer` | 命中的 `NetMapPeer` |
| `get_topology` | `NetMapTopology{Self data.NetMapLocalInfo, Peers []NetMapPeer}` |

```go
type NetMapPeer struct {
    Name     string
    Address  string
    Role     string
    LastSeen time.Time
}

type NetMapTopology struct {
    Self  data.NetMapLocalInfo `json:"self"`
    Peers []NetMapPeer         `json:"peers"`
}
```

## 5. 依赖与前置条件

- 依赖 `BaseData` 与 `NetMapData`，挂载前两者必须已注册，否则返回 `types.ErrMissingDependency`。
- 地址校验规则（`isValidPeerAddress`）：允许 `host:port`（端口可解析）、纯 IP、主机名（仅字母数字、点、连字符、下划线）；含 `/`、`\`、空格、`?`、`#` 的地址一律拒绝。
- `register_peer` 重复注册同名节点会返回错误；`lookup_peer` 传入的 Name 和 Address 至少一项非空。

## 6. 可直接编译的示例

```go
package main

import (
    "fmt"

    "github.com/FasterEdge/FasterEdge/ability"
    fasteredge "github.com/FasterEdge/FasterEdge"
    "github.com/FasterEdge/FasterEdge/types"
)

func main() {
    atom := fasteredge.InitStandardAtom()
    if err := fasteredge.PreRunAtom(atom); err != nil {
        panic(err)
    }

    ab, ok := atom.Ability("NetMapAbility")
    if !ok {
        panic("NetMapAbility missing")
    }

    // 注册一个对等节点
    out := ab.Command(atom, ability.NetMapCommandRegisterPeer, ability.NetMapRegisterPeerArgs{
        Name: "edge-2", Address: "192.0.2.20:7000", Role: "edge",
    })
    if out.Err != nil {
        panic(out.Err)
    }
    fmt.Printf("registered: %+v\n", out.Value.(ability.NetMapPeer))

    // 查询
    out = ab.Command(atom, ability.NetMapCommandLookupPeer, ability.NetMapLookupPeerArgs{Name: "edge-2"})
    if out.Err != nil {
        panic(out.Err)
    }
    fmt.Printf("lookup: %+v\n", out.Value.(ability.NetMapPeer))

    // 拓扑快照
    out = ab.Command(atom, ability.NetMapCommandGetTopology, nil)
    if out.Err != nil {
        panic(out.Err)
    }
    topo := out.Value.(ability.NetMapTopology)
    fmt.Printf("topology: self=%+v peers=%d\n", topo.Self, len(topo.Peers))

    _ = types.ErrInvalidArguments // 示例引用 types 包
}
```