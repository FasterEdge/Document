# AlgorithmDistributionAbility 文档

## 简介
`AlgorithmDistributionAbility` 在 `FileTransferAbility` 之上提供算法（软件模块）分发能力。它负责注册算法元数据（名称、版本、源路径、内容类型），并通过底层的 `FileTransferAbility` 把算法文件上传至目标节点。分发任务在内部会产生对应的 `FileTransfer`，状态同步由后台 watcher 完成。

## 支持的命令
| 命令字符串 | Go 常量名 | 目的 |
|---|---|---|
| `register_algorithm` | `AlgDistCommandRegister` | 注册算法元数据 |
| `unregister_algorithm` | `AlgDistCommandUnregister` | 删除已注册的算法 |
| `list_algorithms` | `AlgDistCommandList` | 列出所有已注册的算法 |
| `get_algorithm` | `AlgDistCommandGet` | 根据名称+版本查询单个算法 |
| `distribute` | `AlgDistCommandDistribute` | 将已注册的算法分发至目标节点 |
| `cancel` | `AlgDistCommandCancel` | 取消正在进行的分发任务 |
| `list_distributions` | `AlgDistCommandListDistribute` | 列出所有分发任务 |
| `clear_finished` | `AlgDistCommandClearFinished` | 清除已完成/失败/取消的分发任务记录 |

## 参数类型
### `AlgDistRegisterArgs`
```go
type AlgDistRegisterArgs struct {
    Name        string // 必填，算法名称
    Version     string // 必填，算法版本
    SourcePath  string // 必填，算法文件本地路径
    ContentType string // 可选，内容类型（如 "application/octet-stream"）
}
```
### `AlgDistAlgorithmRef`
用于 `unregister_algorithm` 与 `get_algorithm`。
```go
type AlgDistAlgorithmRef struct {
    Name    string // 必填
    Version string // 必填
}
```
### `AlgDistDistributeArgs`
```go
type AlgDistDistributeArgs struct {
    Name       string // 必填，算法名称
    Version    string // 必填，算法版本
    Target     string // 必填，目标对等节点名称
    RemotePath string // 可选，远端存放路径；若空则使用默认 "/algorithms/<name>-<version>"
}
```
### `AlgDistIDArg`
用于 `cancel` 与 `clear_finished`。
```go
type AlgDistIDArg struct {
    ID string // 任务唯一 ID
}
```

## 返回值
所有命令返回 `types.CommandOutput`。
- `register_algorithm` → `AlgDistAlgorithm`（已注册的算法结构体）
- `unregister_algorithm` → `AlgDistAlgorithm`（被删除的算法结构体）
- `list_algorithms` → `[]AlgDistAlgorithm`
- `get_algorithm` → `AlgDistAlgorithm`
- `distribute` → `string`（新创建的分发任务 ID）
- `list_distributions` → `[]AlgDistJob`
- `cancel` / `clear_finished` → `string`（被取消或清除的任务 ID）
- 其它命令返回 `nil` 表示仅成功执行。

## 依赖与前置条件
- **依赖**：`BaseData`、`NetMapData`（用于验证目标节点）以及 `FileTransferAbility`（实际的文件传输实现）。
- **前置条件**：在调用 `distribute` 之前必须先使用 `register_algorithm` 注册对应的算法，并且 `FileTransferAbility` 已通过 `SetTransport` 注入实现（如 HTTP、gRPC、MQTT 等）。若未注入 Transport，则分发进入 **骨架模式**：立即标记为完成但不实际传输。
- `set_target` 在 `FileTransferAbility` 中设置的默认目标对等节点对本能力无直接影响，`distribute` 必须在参数中显式提供 `Target`。

## 示例代码（可直接编译）
```go
package main

import (
    "fmt"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

// mockTransport 仅演示 FileTransferTransport 接口实现。
type mockTransport struct{}
func (m *mockTransport) Upload(ctx ability.FileTransferContext) error   { return nil }
func (m *mockTransport) Download(ctx ability.FileTransferContext) error { return nil }

func main() {
    // 创建底层文件传输 Ability 并注入 Transport
    ft := ability.NewFileTransferAbility()
    ft.SetTransport(&mockTransport{})

    // 创建算法分发 Ability，显式关联底层 FileTransferAbility
    alg := ability.NewAlgDistAbilityWithTransfer(ft)

    // 假设有一个已创建并挂载好的 Atom（此处简化为 nil，仅演示调用签名）
    var atom *types.Atom = &types.Atom{}

    // 注册算法元数据
    regOut := alg.Command(atom, ability.AlgDistCommandRegister, ability.AlgDistRegisterArgs{
        Name:        "myAlgo",
        Version:     "v1.0",
        SourcePath:  "/opt/algos/myAlgo.bin",
        ContentType: "application/octet-stream",
    })
    fmt.Printf("register result: %+v, err: %v\n", regOut.Value, regOut.Err)

    // 分发到目标节点 "nodeB"
    distOut := alg.Command(atom, ability.AlgDistCommandDistribute, ability.AlgDistDistributeArgs{
        Name:    "myAlgo",
        Version: "v1.0",
        Target:  "nodeB",
    })
    fmt.Printf("distribution task ID: %v, err: %v\n", distOut.Value, distOut.Err)

    // 列出所有分发任务
    listOut := alg.Command(atom, ability.AlgDistCommandListDistribute, nil)
    fmt.Printf("all jobs: %+v\n", listOut.Value)
}
```
