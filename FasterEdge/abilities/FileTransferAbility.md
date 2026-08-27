# FileTransferAbility 文档

## 简介
`FileTransferAbility` 提供节点间文件传输能力。它通过可插拔的 `FileTransferTransport` 接口（如 HTTP、gRPC、MQTT 等）完成实际字节流的发送与接收，仅记录元数据并跟踪每次传输的状态、大小与错误。该能力依赖 `BaseData` 与 `NetMapData`（用于解析对等节点名称）。

## 支持的命令
| 命令字符串 | Go 常量名 | 目的 |
|---|---|---|
| `set_target` | `FileTransferCommandSetTarget` | 设置默认目标对等节点名称 |
| `get_target` | `FileTransferCommandGetTarget` | 获取当前默认目标 |
| `upload` | `FileTransferCommandUpload` | 上传本地文件到远端目标 |
| `download` | `FileTransferCommandDownload` | 从远端目标下载文件到本地 |
| `list` | `FileTransferCommandList` | 列出所有传输任务 |
| `get_transfer` | `FileTransferCommandGet` | 根据 ID 查询单个传输任务 |
| `cancel` | `FileTransferCommandCancel` | 取消指定 ID 的传输任务 |
| `clear_finished` | `FileTransferCommandClearFinished` | 清除已完成/失败/取消的任务记录 |

## 参数类型
### `FileTransferTargetArgs`
```go
type FileTransferTargetArgs struct {
    PeerName string // 必填，目标对等节点名称
}
```
### `FileTransferUploadArgs`
```go
type FileTransferUploadArgs struct {
    LocalPath  string // 必填，本地文件路径
    RemotePath string // 必填，远端存储路径
    Target     string // 可选，覆盖本次传输的目标对等节点名称
}
```
### `FileTransferDownloadArgs`
```go
type FileTransferDownloadArgs struct {
    RemotePath string // 必填，远端文件路径
    LocalPath  string // 必填，本地保存路径
    Target     string // 可选，覆盖本次传输的目标对等节点名称
}
```
### `FileTransferIDArg`
```go
type FileTransferIDArg struct {
    ID string // 必填，传输任务的唯一 ID
}
```

## 返回值
所有命令返回 `types.CommandOutput`。`Value` 的类型视命令而定：
- `set_target` / `get_target` → `string`（目标节点名称）
- `upload` / `download` → `string`（新建的传输任务 ID）
- `list` → `[]FileTransfer`（所有任务快照）
- `get_transfer` → `FileTransfer`（单个任务详情）
- `cancel` / `clear_finished` → `string`（被取消或清除的 ID）
- 其余返回 `nil` 表示仅执行成功。

## 依赖与前置条件
- **依赖**：`BaseData`、`NetMapData`（用于验证对等节点）以及 `NetMapAbility`（实际进行节点名称查找）。
- **前置条件**：在使用传输相关命令前必须调用 `SetTransport` 注入实现 `FileTransferTransport`。若未注入，则上传/下载仅进入 **骨架模式**：立即标记任务为完成且不产生实际流量。
- `set_target` 必须成功并且对应的对等节点在 `NetMapAbility` 中可查找到，才能进行上传/下载。

## 示例代码（可直接编译）
```go
package main

import (
    "fmt"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

// mockTransport 只演示接口实现，实际可换成 HTTP、gRPC 等。
type mockTransport struct{}

func (m *mockTransport) Upload(ctx ability.FileTransferContext) error { return nil }
func (m *mockTransport) Download(ctx ability.FileTransferContext) error { return nil }

func main() {
    // 创建 Ability 并注入 Transport
    ft := ability.NewFileTransferAbility()
    ft.SetTransport(&mockTransport{})

    // 假设 atom 已经包含 BaseData、NetMapData 与 NetMapAbility（此处使用空 Atom 仅演示调用）
    var atom *types.Atom = &types.Atom{}

    // 设置默认目标节点
    ft.Command(atom, ability.FileTransferCommandSetTarget, ability.FileTransferTargetArgs{PeerName: "nodeA"})

    // 发起一次上传
    out := ft.Command(atom, ability.FileTransferCommandUpload, ability.FileTransferUploadArgs{
        LocalPath:  "/tmp/example.txt",
        RemotePath: "/data/example.txt",
    })
    fmt.Printf("Upload task ID: %v, err: %v\n", out.Value, out.Err)

    // 查询任务列表
    list := ft.Command(atom, ability.FileTransferCommandList, nil)
    fmt.Printf("All transfers: %+v\n", list.Value)
}
```
