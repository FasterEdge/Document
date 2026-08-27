# TimeAbility 文档

## 1. 简介

`TimeAbility` 为系统提供可靠的时间同步能力。它支持四种同步来源：手动指定、系统时钟、HTTP 接口以及 NTP 服务，并提供获取最近一次同步信息、配置运行模式的能力。

## 2. 支持的命令

| 命令字符串 | 常量名称 | 用途 |
|-----------|----------|------|
| `sync_manual` | `TimeCommandSyncManual` | 手动指定时间（RFC3339Nano） |
| `sync_system` | `TimeCommandSyncSystem` | 使用本地系统时钟同步 |
| `sync_net` | `TimeCommandSyncNetwork` | 通过 HTTP API 获取网络时间 |
| `sync_ntp` | `TimeCommandSyncNTP` | 通过 NTP 服务器获取时间 |
| `last` | `TimeCommandLastSync` | 查询最近一次同步的来源与时间 |
| `get_time` | `TimeCommandGetTime` | 获取当前时间（基于最近一次同步） |
| `configure_run` | `TimeCommandConfigureRun` | 配置内部运行模式（monotonic / ticker）及间隔 |

## 3. 参数类型

```go
// sync_manual
type TimeSyncManualArgs struct {
    Value string // RFC3339Nano 格式的时间字符串，必填
}

// sync_net
type TimeSyncNetworkArgs struct {
    URL string // 可选，HTTP 接口地址，空则使用默认
}

// sync_ntp
type TimeSyncNTPArgs struct {
    Address string // 可选，NTP 服务器地址，空则使用默认
}

// configure_run
type TimeConfigureRunArgs struct {
    Mode     TimeRunMode   // "monotonic" 或 "ticker"
    Interval time.Duration // 正数， ticker 模式的周期，monotonic 模式只能为 0
}
```

## 4. 返回值

- `sync_manual`, `sync_system`, `sync_net`, `sync_ntp`, `last`, `get_time`：`CommandOutput.Value` 为 `TimeSnapshot`，包含字段 `Source string`（同步来源标识）和 `Time time.Time`（同步得到的时间）。
- `configure_run`：无返回值，`CommandOutput.Value` 为 `nil`。

## 5. 依赖与前置条件

- 依赖 `BaseData`（通过 `Dependencies()` 返回）。在挂载前必须确保原子（`*types.Atom`）中已提供 `BaseData`。
- 网络同步受 `addressPolicy` 控制：默认不允许私有、回环、链路本地、多播地址；可以通过 `WithPrivateNetworkTimeSources()` 选项放宽。
- HTTP 同步限制：最大响应体 64 KiB、禁用系统代理、HTTPS 降级检查、重定向校验。
- NTP 同步会验证偏移并拒绝异常响应。

## 6. 可直接编译的示例

```go
package main

import (
    "fmt"
    "time"
    "github.com/FasterEdge/FasterEdge/ability"
)

func main() {
    // 创建 Ability（使用默认配置）
    ta, err := ability.NewTimeAbility()
    if err != nil {
        panic(err)
    }

    // 手动同步示例（同步到 2026-01-01T00:00:00Z）
    manualArgs := ability.TimeSyncManualArgs{Value: "2026-01-01T00:00:00Z"}
    out := ta.Command(nil, ability.TimeCommandSyncManual, manualArgs)
    if out.Err != nil {
        fmt.Printf("manual sync error: %v\n", out.Err)
    } else {
        fmt.Printf("手动同步成功: %+v\n", out.Value)
    }

    // HTTP 同步（使用默认 URL）
    out = ta.Command(nil, ability.TimeCommandSyncNetwork, ability.TimeSyncNetworkArgs{})
    if out.Err != nil {
        fmt.Printf("network sync error: %v\n", out.Err)
    } else {
        fmt.Printf("网络同步成功: %+v\n", out.Value)
    }

    // 获取当前时间（基于最近一次同步）
    out = ta.Command(nil, ability.TimeCommandGetTime, nil)
    if out.Err != nil {
        fmt.Printf("get_time error: %v\n", out.Err)
    } else {
        fmt.Printf("当前时间: %+v\n", out.Value)
    }

    // 配置运行模式为 ticker，间隔 2 秒
    cfg := ability.TimeConfigureRunArgs{Mode: ability.TimeRunModeTicker, Interval: 2 * time.Second}
    out = ta.Command(nil, ability.TimeCommandConfigureRun, cfg)
    if out.Err != nil {
        fmt.Printf("configure run error: %v\n", out.Err)
    } else {
        fmt.Println("运行模式配置成功")
    }
}
```
