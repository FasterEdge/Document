## BaseData

### 简介

`BaseData` 仅用于存放框架的基础信息。当前实现几乎不持有任何状态，仅提供框架版本与 Logo 的展示能力。

### 支持的命令

| 命令字符串 | Go 常量名 | 说明 |
|-----------|----------|------|
| `logo`    | `CommandLogo` | 返回框架的 ASCII Logo。
| `info`    | `CommandInfo` | 返回框架版本信息描述字符串。

### 参数类型

`BaseData.Command` 不接受任何参数，`args` 必须为 `nil`，否则返回 `ErrInvalidArguments`。

### 返回值

返回 `types.CommandOutput{ Name: <command>, Value: <string>, Err: nil }`。

- `logo`：`Value` 为多行字符串的 ASCII Logo。
- `info`：`Value` 为 `"FasterEdge v<version> - 对称、可靠、安全的多场景边缘计算框架"`。

### 持久化/状态说明

`BaseData` 不持有持久化状态，也不参与快照或磁盘存储。

### 示例代码

```go
package main

import (
    "fmt"
    "github.com/FasterEdge/FasterEdge/data"
)

func main() {
    var d data.BaseData
    // 获取 Logo
    out := d.Command(nil, data.CommandLogo, nil)
    if out.Err != nil {
        panic(out.Err)
    }
    fmt.Println(out.Value)

    // 获取版本信息
    out = d.Command(nil, data.CommandInfo, nil)
    if out.Err != nil {
        panic(out.Err)
    }
    fmt.Println(out.Value)
}
```
