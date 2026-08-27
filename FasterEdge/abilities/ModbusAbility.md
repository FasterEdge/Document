# ModbusAbility 文档

## 简介
`ModbusAbility` 提供 Modbus TCP/RTU 主站（Master）读写能力，支持保持寄存器、输入寄存器、线圈与离散输入四类对象。所有实际的字节流收发通过注入的 `ModbusTransport` 接口完成。若未注入 Transport，所有读写命令会直接返回 `ErrInvalidArguments`（防止误用）。

## 支持的命令
| 命令字符串 | Go 常量名 | 目的 |
|---|---|---|
| `set_endpoint` | `ModbusCommandSetEndpoint` | 设置从站端点（TCP `host:port` 或 RTU `/dev/xxx:baud:format`） |
| `get_endpoint` | `ModbusCommandGetEndpoint` | 获取当前从站端点 |
| `read_holding` | `ModbusCommandReadHolding` | 读取保持寄存器（FC03） |
| `read_input` | `ModbusCommandReadInput` | 读取输入寄存器（FC04） |
| `read_coils` | `ModbusCommandReadCoils` | 读取线圈状态（FC01） |
| `read_discrete` | `ModbusCommandReadDiscrete` | 读取离散输入（FC02） |
| `write_holding` | `ModbusCommandWriteHolding` | 写单个保持寄存器（FC06） |
| `write_coil` | `ModbusCommandWriteCoil` | 写单个线圈（FC05） |
| `write_holding_multi` | `ModbusCommandWriteMultiReg` | 写多个保持寄存器（FC16/0x10） |
| `set_unit_id` | `ModbusCommandSetUnitID` | 设置从站单元地址（Unit ID） |
| `get_unit_id` | `ModbusCommandGetUnitID` | 获取当前单元地址 |

功能码常量（供读写命令构造 PDU 使用）：
`ModbusFuncReadCoils`（0x01）、`ModbusFuncReadDiscreteInputs`（0x02）、`ModbusFuncReadHolding`（0x03）、`ModbusFuncReadInput`（0x04）、`ModbusFuncWriteCoil`（0x05）、`ModbusFuncWriteHolding`（0x06）、`ModbusFuncWriteMultiReg`（0x10）。

## 参数类型
### `ModbusEndpointArgs`
```go
type ModbusEndpointArgs struct {
    Addr string // 必填，如 "192.168.1.10:502"（TCP）或 "/dev/ttyUSB0:9600:8N1"（RTU）
}
```
### `ModbusUnitIDArgs`
用于 `set_unit_id` / `get_unit_id`。
```go
type ModbusUnitIDArgs struct {
    UnitID uint8 // 必填，目标从站单元地址
}
```
### `ModbusReadArgs`
用于所有 `read_*` 命令。
```go
type ModbusReadArgs struct {
    Address  uint16 // 必填，起始地址
    Quantity uint16 // 必填，数量；范围为 1..125
}
```
### `ModbusWriteHoldingArgs`
```go
type ModbusWriteHoldingArgs struct {
    Address uint16 // 必填，寄存器地址
    Value   uint16 // 必填，要写入的值
}
```
### `ModbusWriteCoilArgs`
```go
type ModbusWriteCoilArgs struct {
    Address uint16 // 必填，线圈地址
    Value   bool   // 必填，true=ON / false=OFF
}
```
### `ModbusWriteMultiArgs`
用于 `write_holding_multi`。
```go
type ModbusWriteMultiArgs struct {
    Address uint16   // 必填，起始寄存器地址
    Values  []uint16 // 必填，要写入的值列表；长度 1..123
}
```

## 返回值
所有命令返回 `types.CommandOutput`。
- `read_*` → `ModbusReadResult`：
  ```go
  type ModbusReadResult struct {
      Function ModbusFunctionCode
      Address  uint16
      Quantity uint16
      UnitID   uint8
      Regs     []uint16 // holding/input 的数据，大端序
      Coils    []bool   // coil/discrete 的数据，每个 bit 表示一个
      Bytes    []byte   // 原始字节，供高级用法
  }
  ```
- `write_*` → `bool`（写成功返回 `true`）
- `set_endpoint` / `get_endpoint` → `string`（端点地址）
- `set_unit_id` / `get_unit_id` → `uint8`
- 其它命令返回 `nil` 表示仅成功执行。

## 依赖与前置条件
- **依赖**：`BaseData`。
- **前置条件**：调用读写命令前必须通过 `SetTransport(t ModbusTransport)` 注入实现 `ModbusTransport`。该接口包含 `Send(unitID uint8, pdu []byte) ([]byte, error)` 与 `Close() error`。
- 建议按顺序先 `set_endpoint`、`set_unit_id`，再执行读写命令。只要端点为合法格式（TCP `host:port`，或 RTU `/dev/...:baud:format`）即接受；未注入 Transport 的任何读写命令均报错。

## 示例代码（可直接编译）
```go
package main

import (
    "fmt"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

// mockTransport 模拟 Modbus 从站的响应（只回显请求 PDU）。
type mockTransport struct{}

func (m *mockTransport) Send(unitID uint8, pdu []byte) ([]byte, error) {
    return pdu, nil
}
func (m *mockTransport) Close() error { return nil }

func main() {
    modbus := ability.NewModbusAbility()
    modbus.SetTransport(&mockTransport{})
    var atom *types.Atom = &types.Atom{}

    // 设置端点与单元地址
    modbus.Command(atom, ability.ModbusCommandSetEndpoint, ability.ModbusEndpointArgs{Addr: "192.168.1.10:502"})
    modbus.Command(atom, ability.ModbusCommandSetUnitID, ability.ModbusUnitIDArgs{UnitID: 1})

    // 读取保持寄存器：从地址 0 读 10 个
    readOut := modbus.Command(atom, ability.ModbusCommandReadHolding, ability.ModbusReadArgs{
        Address:  0,
        Quantity: 10,
    })
    fmt.Printf("read result: %+v, err: %v\n", readOut.Value, readOut.Err)

    // 写单个线圈
    writeOut := modbus.Command(atom, ability.ModbusCommandWriteCoil, ability.ModbusWriteCoilArgs{
        Address: 0,
        Value:   true,
    })
    fmt.Printf("write coil: %v, err: %v\n", writeOut.Value, writeOut.Err)

    // 写多个保持寄存器
    multOut := modbus.Command(atom, ability.ModbusCommandWriteMultiReg, ability.ModbusWriteMultiArgs{
        Address: 10,
        Values:  []uint16{1, 2, 3, 4},
    })
    fmt.Printf("write multi: %v, err: %v\n", multOut.Value, multOut.Err)
}
```