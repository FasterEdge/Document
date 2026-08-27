## ConfigData

### 简介

`ConfigData` 以键值树形式存储节点配置项，采用扁平点号路径命名（如 `server.port`）。它只负责内存态管理，实际的持久化文件读写由 `ConfigFileAbility` 负责。

### 支持的命令

| 命令字符串 | Go 常量名 | 说明 |
|-----------|----------|------|
| `get`     | `ConfigCommandGet`     | 读取指定键的值。参数 `ConfigGetArgs{Key string}`。 |
| `set`     | `ConfigCommandSet`     | 写入键值。参数 `ConfigSetArgs{Key, Value string}`。 |
| `delete`  | `ConfigCommandDelete`  | 删除指定键。参数 `ConfigDeleteArgs{Key string}`。 |
| `list`    | `ConfigCommandList`    | 返回所有键（字典序排序），`[]string`。 |
| `snapshot`| `ConfigCommandSnapshot`| 返回全部键值的深拷贝，`map[string]string`。 |

### 参数类型

```go
// get 参数
type ConfigGetArgs struct { Key string }

// set 参数
type ConfigSetArgs struct {
    Key   string
    Value string
}

// delete 参数
type ConfigDeleteArgs struct { Key string }
```

### 返回值

所有命令返回 `types.CommandOutput`：

- `get`：`Value` 为命中的配置值 `string`；键不存在时返回 `ErrInvalidArguments` 包装错误。
- `set`：`Value` 为写入的值 `string`。
- `delete`：`Value` 为被删除的键 `string`；键不存在时返回 `ErrInvalidArguments` 包装错误。
- `list`：`Value` 为按字典序排序的键切片 `[]string`。
- `snapshot`：`Value` 为键值快照 `map[string]string`。

### 持久化/状态说明

- **键校验**：键必须非空，且仅允许字母数字、`.`、`_`、`-` 字符；非法键会返回 `ErrInvalidArguments`。
- **持久化**：`ConfigData` 本身只保留内存态。`JSONMarshal()` 以 `json.MarshalIndent` 输出整个配置的 JSON 表示，供 `ConfigFileAbility` 保存到文件。`ConfigFileAbility` 负责读取文件后调用 `ReplaceAll()` 一次性载入配置。
- **Snapshot / GetSet**：`Snapshot()` 返回键值的深拷贝，`GetSet()` 是其别名（供 Ability 就地应用解析后的文件时使用）。

### 示例代码

```go
package main

import (
    "fmt"
    "github.com/FasterEdge/FasterEdge/data"
)

func main() {
    cd := data.NewConfigData()

    // 写入配置
    out := cd.Command(nil, data.ConfigCommandSet, data.ConfigSetArgs{Key: "server.port", Value: "8080"})
    if out.Err != nil { panic(out.Err) }
    fmt.Println("set:", out.Value)

    // 读取配置
    out = cd.Command(nil, data.ConfigCommandGet, data.ConfigGetArgs{Key: "server.port"})
    if out.Err != nil { panic(out.Err) }
    fmt.Println("get:", out.Value)

    // 列出全部键
    out = cd.Command(nil, data.ConfigCommandList, nil)
    if out.Err != nil { panic(out.Err) }
    fmt.Printf("keys: %v\n", out.Value)

    // 获取全量快照
    out = cd.Command(nil, data.ConfigCommandSnapshot, nil)
    if out.Err != nil { panic(out.Err) }
    fmt.Printf("snapshot: %v\n", out.Value)

    // 序列化为 JSON（供 ConfigFileAbility 持久化）
    if b, err := cd.JSONMarshal(); err == nil {
        fmt.Println(string(b))
    }
}
```