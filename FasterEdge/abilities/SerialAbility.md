# SerialAbility 文档

## 简介
`SerialAbility` 提供串口管理能力，支持串口的打开、关闭、读取、写入，以及运行时获取和更新配置。底层字节流通过注入的 `SerialTransport` 完成；端口枚举则由 `SerialPortLister` 负责（默认实现返回空列表以避免跨平台差异）。若未注入 Transport，任何读写相关命令均返回错误。

## 支持的命令
| 命令字符串 | Go 常量名 | 目的 |
|---|---|---|
| `open` | `SerialCommandOpen` | 按配置打开串口 |
| `close` | `SerialCommandClose` | 关闭已打开的串口 |
| `write` | `SerialCommandWrite` | 向已打开串口写入数据 |
| `read` | `SerialCommandRead` | 从已打开串口读取数据 |
| `is_open` | `SerialCommandIsOpen` | 查询指定串口是否处于打开状态 |
| `set_config` | `SerialCommandSetConfig` | 修改已打开串口的配置 |
| `get_config` | `SerialCommandGetConfig` | 获取已打开串口的当前配置 |
| `list_ports` | `SerialCommandListPorts` | 枚举系统上的可用串口 |

## 参数类型
### `SerialConfig`
```go
type SerialConfig struct {
    Baud     int    // 波特率，如 9600
    DataBits int    // 数据位，5/6/7/8
    StopBits int    // 停止位，1 或 2
    Parity   string // 校验位，"N"=无、"O"=奇、"E"=偶
}
```
### `SerialOpenArgs`
```go
type SerialOpenArgs struct {
    Port   string      // 必填，如 "/dev/ttyUSB0" / "COM3"
    Config SerialConfig // 必填，打开时的初始配置
}
```
### `SerialPortArg`
用于 `close` / `is_open` / `get_config`。
```go
type SerialPortArg struct {
    Port string // 必填，串口路径
}
```
### `SerialWriteArgs`
```go
type SerialWriteArgs struct {
    Port   string // 必填，已打开的串口
    Data   []byte // 必填，写入字节
    Length int    // 可选；0 表示写入全部 Data，否则截断到指定长度
}
```
### `SerialReadArgs`
```go
type SerialReadArgs struct {
    Port    string        // 必填，已打开的串口
    Length  int           // 必填，要读取的字节数（必须为 >0）
    Timeout time.Duration // 必填，单次读等待超时
}
```
### `SerialSetConfigArgs`
```go
type SerialSetConfigArgs struct {
    Port   string      // 必填，仅对已打开的串口生效
    Config SerialConfig // 必填，新的配置
}
```

## 返回值
所有命令返回 `types.CommandOutput`。
- `open` → `string`（被打开串口的路径）
- `close` → `string`（被关闭的串口路径）
- `write` → `int`（实际写入的字节数）
- `read` → `[]byte`（读取到的字节切片）
- `is_open` → `bool`
- `set_config` / `get_config` → `SerialConfig`
- `list_ports` → `[]string`（可用串口路径列表）
- 其它命令返回 `nil` 表示仅成功执行。

## 依赖与前置条件
- **依赖**：`BaseData`。
- **前置条件**：调用 `write` / `read` / `set_config` / `get_config` / `close` 前必须先通过 `open` 打开目标串口。
- 必须通过 `SetTransport(t SerialTransport)` 注入实现 `SerialTransport`（包含 `Open`、`Close`、`Write`、`Read`），否则所有操作均报错。
- 端口路径须符合格式：Linux 前缀 `/dev/tty` 或 Windows `COM` 后跟数字。
- 配置 `Baud`、`DataBits`、`StopBits`、`Parity` 均须通过合法值校验，不满足则直接返回错误。

## 示例代码（可直接编译）
```go
package main

import (
    "fmt"
    "time"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

// mockTransport 模拟串口字节流。
type mockTransport struct {
    openPorts map[string]ability.SerialConfig
}

func (m *mockTransport) Open(port string, cfg ability.SerialConfig) error {
    if m.openPorts == nil {
        m.openPorts = make(map[string]ability.SerialConfig)
    }
    m.openPorts[port] = cfg
    return nil
}
func (m *mockTransport) Close(port string) error {
    delete(m.openPorts, port)
    return nil
}
func (m *mockTransport) Write(port string, data []byte) (int, error) {
    return len(data), nil
}
func (m *mockTransport) Read(port string, length int, timeout time.Duration) ([]byte, error) {
    return []byte("ok"), nil
}

func main() {
    serial := ability.NewSerialAbility()
    tr := &mockTransport{}
    serial.SetTransport(tr)
    var atom *types.Atom = &types.Atom{}

    // 打开串口
    openOut := serial.Command(atom, ability.SerialCommandOpen, ability.SerialOpenArgs{
        Port:   "/dev/ttyUSB0",
        Config: ability.SerialConfig{Baud: 115200, DataBits: 8, StopBits: 1, Parity: "N"},
    })
    fmt.Printf("open: %v, err: %v\n", openOut.Value, openOut.Err)

    // 写入数据
    writeOut := serial.Command(atom, ability.SerialCommandWrite, ability.SerialWriteArgs{
        Port: "/dev/ttyUSB0",
        Data: []byte("hello"),
    })
    fmt.Printf("write: %v, err: %v\n", writeOut.Value, writeOut.Err)

    // 读取数据
    readOut := serial.Command(atom, ability.SerialCommandRead, ability.SerialReadArgs{
        Port:    "/dev/ttyUSB0",
        Length:  10,
        Timeout: 100 * time.Millisecond,
    })
    fmt.Printf("read: %v, err: %v\n", readOut.Value, readOut.Err)

    // 关闭串口
    closeOut := serial.Command(atom, ability.SerialCommandClose, ability.SerialPortArg{Port: "/dev/ttyUSB0"})
    fmt.Printf("close: %v, err: %v\n", closeOut.Value, closeOut.Err)
}
```