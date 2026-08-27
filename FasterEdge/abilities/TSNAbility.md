# TSNAbility 文档

## 简介
`TSNAbility` 提供时间敏感网络（TSN）能力，包括 TSN 网络接口管理、talker/listener 数据流的注册、IEEE 802.1Q 优先级到队列（Queue）的映射，以及 IEEE 802.1Qbv 时间感知门控（Time-Aware Shaper）的配置。

## 支持的命令
| 命令字符串 | Go 常量名 | 目的 |
|---|---|---|
| `set_interface` | `TSNCommandSetInterface` | 设置 TSN 使用的网络接口 |
| `get_interface` | `TSNCommandGetInterface` | 获取当前 TSN 网络接口 |
| `register_talker` | `TSNCommandRegisterTalker` | 注册一条 talker（发送方）数据流 |
| `register_listener` | `TSNCommandRegisterListener` | 注册一条 listener（接收方）数据流 |
| `unregister` | `TSNCommandUnregister` | 注销一条已注册的数据流 |
| `list_streams` | `TSNCommandListStreams` | 列出所有已注册的数据流 |
| `set_priority_map` | `TSNCommandSetPriority` | 设置 802.1Q 优先级到队列的映射 |
| `get_priority_map` | `TSNCommandGetPriority` | 获取当前优先级映射 |
| `set_time_aware` | `TSNCommandSetTimeAware` | 设置 802.1Qbv 时间感知门控状态 |
| `get_time_aware` | `TSNCommandGetTimeAware` | 获取当前时间感知门控配置 |

## 参数类型
### `TSNInterfaceArg`
```go
type TSNInterfaceArg struct {
    Interface string // 必填，网络接口名，如 "eth0"
}
```
### `TSNRegisterTalkerArgs`
```go
type TSNRegisterTalkerArgs struct {
    ID         string        // 必填，数据流 ID
    MAC        string        // 必填，源 MAC 地址，如 "aa:bb:cc:dd:ee:ff"
    DestMAC    string        // 可选，目的 MAC 地址（若提供必须为合法 MAC）
    VLANID     uint16        // VLAN ID
    Priority   uint8         // 802.1Q 优先级，0..7
    PayloadLen uint16        // 负载长度
    Interval   time.Duration // 必填，发送周期，必须 > 0
}
```
### `TSNRegisterListenerArgs`
```go
type TSNRegisterListenerArgs struct {
    ID       string // 必填，数据流 ID
    MAC      string // 必填，源 MAC 地址
    DestMAC  string // 可选，目的 MAC 地址
    VLANID   uint16
    Priority uint8 // 0..7
}
```
### `TSNStreamIDArg`
用于 `unregister`。
```go
type TSNStreamIDArg struct {
    ID string // 必填，要注销的数据流 ID
}
```
### `TSNPriorityMapArgs`
```go
type TSNPriorityMapArgs struct {
    Mappings map[uint8]uint8 // 优先级 0..7 → 队列 0..7；未在 Map 中的优先级默认映射到 Queue 0
}
```
### `TSNTimeAwareArgs`
```go
type TSNTimeAwareArgs struct {
    Enabled    bool          // 是否启用时间感知门控
    BaseTime   time.Time     // Qbv 基准时间
    CycleTime  time.Duration // 循环周期，必须 >= 0
    GateStates []byte        // 每字节低 8 位对应 8 个队列的门状态
}
```

## 返回值
所有命令返回 `types.CommandOutput`。
- `set_interface` / `get_interface` → `string`（接口名）
- `register_talker` / `register_listener` → `TSNStream`（新注册的数据流结构体）
- `unregister` → `TSNStream`（被注销的数据流结构体）
- `list_streams` → `[]TSNStream`
- `set_priority_map` / `get_priority_map` → `map[uint8]uint8`
- `set_time_aware` / `get_time_aware` → `TSNTimeAwareArgs`
- 其它命令返回 `nil` 表示仅成功执行。

## 依赖与前置条件
- **依赖**：`BaseData`。
- 新建实例时默认的优先级映射为恒等映射 `{0→0, 1→1, ..., 7→7}`，可直接使用。
- 注册数据流时 `ID` 与 `MAC` 必填，MAC 必须为合法的六段十六进制格式；talker 的 `Interval` 必须为正；优先级取值范围为 `0..7`。
- 本能力为元数据管理层，不注入任何传输层；实际的门控/映射下发给网卡由外部驱动完成。

## 示例代码（可直接编译）
```go
package main

import (
    "fmt"
    "time"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

func main() {
    tsn := ability.NewTSNAbility()
    var atom *types.Atom = &types.Atom{}

    // 设置 TSN 网络接口
    tsn.Command(atom, ability.TSNCommandSetInterface, ability.TSNInterfaceArg{Interface: "eth0"})

    // 注册一个 talker 数据流
    talkerOut := tsn.Command(atom, ability.TSNCommandRegisterTalker, ability.TSNRegisterTalkerArgs{
        ID:         "stream-1",
        MAC:        "aa:bb:cc:dd:ee:01",
        DestMAC:    "aa:bb:cc:dd:ee:02",
        VLANID:     100,
        Priority:   5,
        PayloadLen: 1400,
        Interval:   1 * time.Millisecond,
    })
    fmt.Printf("register talker: %+v, err: %v\n", talkerOut.Value, talkerOut.Err)

    // 注册一个 listener 数据流
    listenerOut := tsn.Command(atom, ability.TSNCommandRegisterListener, ability.TSNRegisterListenerArgs{
        ID:       "stream-2",
        MAC:      "aa:bb:cc:dd:ee:02",
        VLANID:   100,
        Priority: 3,
    })
    fmt.Printf("register listener: %+v, err: %v\n", listenerOut.Value, listenerOut.Err)

    // 设置优先级到队列的映射
    tsn.Command(atom, ability.TSNCommandSetPriority, ability.TSNPriorityMapArgs{
        Mappings: map[uint8]uint8{0: 0, 5: 7},
    })

    // 设置 Qbv 时间感知门控
    tsn.Command(atom, ability.TSNCommandSetTimeAware, ability.TSNTimeAwareArgs{
        Enabled:    true,
        BaseTime:   time.Now(),
        CycleTime:  1 * time.Millisecond,
        GateStates: []byte{0xFF, 0x00},
    })

    // 列出所有已注册的数据流
    listOut := tsn.Command(atom, ability.TSNCommandListStreams, nil)
    fmt.Printf("streams: %+v\n", listOut.Value)
}
```