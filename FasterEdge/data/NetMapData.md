## NetMapData

### 简介

`NetMapData` 用于记录本节点的网络拓扑信息，包括节点名称、默认网络接口以及所有检测到的网络接口及其 IPv4 地址。该信息在节点内部使用，不涉及远端节点的拓扑数据（后者由 `NetMapAbility` 维护）。

### 支持的命令

| 命令字符串 | Go 常量名 | 说明 |
|-----------|----------|------|
| `info`               | `NetMapCommandInfo`            | 返回当前节点的 `NetMapLocalInfo` 快照（结构体见下文）。 |
| `set_node_name`      | `NetMapCommandSetNodeName`    | 设置本节点的 `NodeName`，参数 `NetMapSetNodeNameArgs{Name string}`。 |
| `interfaces`        | `NetMapCommandInterfaces`     | 重新扫描本机网络接口并返回 `[]NetMapInterface`（每个包含名称、MAC、IPv4 列表）。 |
| `set_default_iface` | `NetMapCommandSetDefaultIface`| 设置默认出网接口，参数 `NetMapSetDefaultIfaceArgs{Name string}`。 |

### 参数类型

```go
// set_node_name 参数
type NetMapSetNodeNameArgs struct{ Name string }

// set_default_iface 参数
type NetMapSetDefaultIfaceArgs struct{ Name string }
```

### 返回值

所有命令返回 `types.CommandOutput`：

- `info`：`Value` 为 `NetMapLocalInfo`，包含 `NodeName、DefaultIface、Interfaces、HostAddresses、ScannedAt`。
- `set_node_name`：`Value` 为设置后的节点名称字符串。
- `interfaces`：`Value` 为 `[]NetMapInterface` 切片，每个元素包含 `Name、MAC、IPv4`（已排序）。
- `set_default_iface`：`Value` 为设置成功的接口名称。

### 持久化/状态说明

`NetMapData` 只在内存中维护状态，持久化由对应 Ability（如 `NetMapAbility`）负责。快照通过 `Snapshot()` 方法返回 `NetMapLocalInfo` 的深拷贝，可用于日志或调试。

### 示例代码

```go
package main

import (
    "fmt"
    "github.com/FasterEdge/FasterEdge/data"
)

func main() {
    var nd data.NetMapData
    // 初始化（会自动扫描接口）
    nd.Mount(nil)

    // 获取网络信息快照
    out := nd.Command(nil, data.NetMapCommandInfo, nil)
    if out.Err != nil { panic(out.Err) }
    info := out.Value.(data.NetMapLocalInfo)
    fmt.Printf("Node: %s, Default iface: %s\n", info.NodeName, info.DefaultIface)
    fmt.Println("Interfaces:")
    for _, iface := range info.Interfaces {
        fmt.Printf("- %s (%s) IPv4: %v\n", iface.Name, iface.MAC, iface.IPv4)
    }

    // 设置节点名称
    out = nd.Command(nil, data.NetMapCommandSetNodeName, data.NetMapSetNodeNameArgs{Name: "edge-01"})
    if out.Err != nil { panic(out.Err) }
    fmt.Println("Set node name to", out.Value)

    // 设置默认接口（假设有 eth0）
    out = nd.Command(nil, data.NetMapCommandSetDefaultIface, data.NetMapSetDefaultIfaceArgs{Name: "eth0"})
    if out.Err != nil { panic(out.Err) }
    fmt.Println("Default iface set to", out.Value)
}
```
