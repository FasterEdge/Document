# InfluxDBAbility

## 简介
`InfluxDBAbility` 为 FasterEdge 提供基于抽象 `InfluxTransport` 的 InfluxDB 客户端能力。它负责管理连接配置、执行 Ping、写入时序点、执行 Flux 查询以及维护已写入的 measurement 列表。所有底层网络交互均通过注入的 `InfluxTransport` 完成，可灵活替换为任何实现（如官方 Go 客户端）。

## 支持的命令
| 命令字符串 | Go 常量名 | 目的 |
|---|---|---|
| `set_endpoint` | `InfluxCommandSetEndpoint` | 设置 InfluxDB HTTP API 端点 |
| `get_endpoint` | `InfluxCommandGetEndpoint` | 获取当前端点 |
| `set_token` | `InfluxCommandSetToken` | 设置访问 Token |
| `set_org` | `InfluxCommandSetOrg` | 设置组织名称 |
| `set_bucket` | `InfluxCommandSetBucket` | 设置默认 bucket |
| `get_config` | `InfluxCommandGetConfig` | 获取完整配置信息 |
| `write` | `InfluxCommandWrite` | 写入时序点 |
| `query` | `InfluxCommandQuery` | 执行 Flux 查询 |
| `ping` | `InfluxCommandPing` | 检测连通性 |
| `list_series` | `InfluxCommandListSeries` | 列出已写入的 measurement |
| `delete_series` | `InfluxCommandDeleteSeries` | 从本地索引中删除 measurement（不影响远端） |

## 参数类型
```go
// set_endpoint、set_token、set_org、set_bucket 的通用参数
type InfluxConfigArgs struct {
    Value string // 具体含义取决于对应命令
}

// write 命令的参数
type InfluxPoint struct {
    Measurement string            // 必填，符合 identifier 规则
    Tags        map[string]string // 可选标签集合
    Fields      map[string]any    // 至少包含一个字段
    Time        time.Time          // 时间戳
}

type InfluxWriteArgs struct {
    Points []InfluxPoint // 待写入的点集合，不能为空
}

// query 命令的参数
type InfluxQueryArgs struct {
    Query string // Flux 查询语句，不能为空
}

// list_series / delete_series 的参数
type InfluxSeriesArgs struct {
    Measurement string // measurement 名称，不能为空
}
```

## 返回值
`Command` 返回 `types.CommandOutput`，`Value` 根据不同命令可能为：
- `string`（如 endpoint、token、org、bucket）
- `InfluxConfig`（`get_config` 返回完整配置结构体）
- `bool`（`ping` 成功标记）
- `int`（`write` 返回写入点数量）
- `[]map[string]any`（`query` 返回查询结果行）
- `[]string`（`list_series` 返回已记录的 measurement 列表）
- `nil`（仅表示执行成功）

## 依赖与前置条件
- 必须先调用 `SetTransport(t InfluxTransport)` 注入实现 `Ping、Write、Query` 的 Transport。
- 在使用 `write`、`query`、`ping` 前，需要通过 `set_endpoint` 指定有效的 HTTP(s) URL（`isAcceptableInfluxURL` 进行校验），并使用 `set_token` 设置访问令牌。
- `set_org` 与 `set_bucket` 为可选，但若使用 InfluxDB 写入，则 `bucket` 必须已配置；`org` 在多数 InfluxDB 部署中也是必需的。

## 示例代码
```go
package main

import (
    "time"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

// mockTransport 用于演示，仅实现接口方法而不实际请求。
 type mockTransport struct{}
 func (m *mockTransport) Ping() error { return nil }
 func (m *mockTransport) Write(points []ability.InfluxPoint) error { return nil }
 func (m *mockTransport) Query(flux string) ([]map[string]any, error) { return nil, nil }

func main() {
    influx := ability.NewInfluxAbility()
    influx.SetTransport(&mockTransport{})

    // 配置端点、token、org、bucket
    influx.Command(&types.Atom{}, ability.InfluxCommandSetEndpoint, ability.InfluxConfigArgs{Value: "https://influx.example.com"})
    influx.Command(&types.Atom{}, ability.InfluxCommandSetToken, ability.InfluxConfigArgs{Value: "my-secret-token"})
    influx.Command(&types.Atom{}, ability.InfluxCommandSetOrg, ability.InfluxConfigArgs{Value: "my-org"})
    influx.Command(&types.Atom{}, ability.InfluxCommandSetBucket, ability.InfluxConfigArgs{Value: "my-bucket"})

    // Ping 检测连通性
    pingOut := influx.Command(&types.Atom{}, ability.InfluxCommandPing, nil)
    println("ping success:", pingOut.Value.(bool))

    // 写入时序点
    pt := ability.InfluxPoint{
        Measurement: "temperature",
        Tags:        map[string]string{"location": "room1"},
        Fields:      map[string]any{"value": 22.5},
        Time:        time.Now(),
    }
    writeOut := influx.Command(&types.Atom{}, ability.InfluxCommandWrite, ability.InfluxWriteArgs{Points: []ability.InfluxPoint{pt}})
    println("points written:", writeOut.Value.(int))

    // 执行查询
    queryOut := influx.Command(&types.Atom{}, ability.InfluxCommandQuery, ability.InfluxQueryArgs{Query: "from(bucket:\"my-bucket\") |> range(start: -1h)"})
    rows := queryOut.Value.([]map[string]any)
    println("query rows:", len(rows))
}
```
