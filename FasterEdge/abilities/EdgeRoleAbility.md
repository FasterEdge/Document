# EdgeRoleAbility

## 简介
EdgeRoleAbility 扩展 RoleAbility，提供边缘节点特有的「区域（zone）、本地能力清单、在线状态与延迟指标」管理能力。它持有自己的状态（区域、能力集合、在线标志、延迟汇总），通过 `Check` 要求 RoleAbility 必须存在且角色为 `"edge"`。所有状态由 `sync.RWMutex` 保护，线程安全。

## 支持的命令
| 常量 | 命令字符串 | 说明 |
|------|-------------|------|
| `ability.EdgeRoleCommandDescribe` | `"describe"` | 返回 `EdgeRoleDescription` 结构快照（含 Role/Zone/Online/Caps/Metrics）。
| `ability.EdgeRoleCommandSetZone` | `"set_zone"` | 设置区域标识，参数为 `EdgeRoleSetZoneArgs`，去除首尾空白后非空才生效。
| `ability.EdgeRoleCommandGetZone` | `"get_zone"` | 获取当前区域值，未设置时为空字符串。
| `ability.EdgeRoleCommandAddCap` | `"add_capability"` | 添加单项能力，参数为 `EdgeRoleCapabilityArg`。
| `ability.EdgeRoleCommandRemoveCap` | `"remove_capability"` | 移除单项能力，参数为 `EdgeRoleCapabilityArg`；不存在时返回错误。
| `ability.EdgeRoleCommandListCaps` | `"list_capabilities"` | 按名称排序返回当前能力列表 `[]string`。
| `ability.EdgeRoleCommandSetCaps` | `"set_capabilities"` | 整体覆盖能力集，参数为 `EdgeRoleSetCapabilitiesArgs`，返回排序后的新列表。
| `ability.EdgeRoleCommandRecordLatency` | `"record_latency"` | 记录一次延迟样本，参数为 `EdgeRoleRecordLatencyArgs`，值须为非负。
| `ability.EdgeRoleCommandGetMetrics` | `"get_metrics"` | 返回 `EdgeRoleMetrics` 结构（含 Online/Zone/Capabilities/AvgLatencyMs 等全部指标）。
| `ability.EdgeRoleCommandSetOnline` | `"set_online"` | 设置在线状态，参数为 `EdgeRoleSetOnlineArgs`。

## 参数类型

```go
// set_zone 的参数
type EdgeRoleSetZoneArgs struct {
    Zone string // 必填，去除首尾空白后非空
}

// 通用能力条目（add_capability / remove_capability 共用）
type EdgeRoleCapabilityArg struct {
    Name string // 必填，去除首尾空白后非空
}

// set_capabilities 的参数，用于整体覆盖
type EdgeRoleSetCapabilitiesArgs struct {
    Capabilities []string // 必填，每个元素去除首尾空白后不能为空
}

// record_latency 的参数
type EdgeRoleRecordLatencyArgs struct {
    LatencyMs float64 // 必填，必须 >= 0
}

// set_online 的参数
type EdgeRoleSetOnlineArgs struct {
    Online bool // 必填，true=在线，false=离线
}
```
| 命令 | 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| `set_zone` | `Zone` | `string` | 是 | 区域标识，`TrimSpace` 后非空。 |
| `add_capability` | `Name` | `string` | 是 | 能力名称，`TrimSpace` 后非空。 |
| `remove_capability` | `Name` | `string` | 是 | 要移除的能力名称，须已存在。 |
| `set_capabilities` | `Capabilities` | `[]string` | 是 | 整体覆盖的能力名列表，每个元素 `TrimSpace` 后不能为空。 |
| `record_latency` | `LatencyMs` | `float64` | 是 | 延迟值（毫秒），必须 >= 0。 |
| `set_online` | `Online` | `bool` | 是 | `true` 上线、`false` 下线。 |

`describe`、`get_zone`、`list_capabilities`、`get_metrics` 不接受参数，`args` 必须为 `nil`。

**返回结构**：
```go
type EdgeRoleMetrics struct {
    Online         bool
    Zone           string
    Capabilities   []string
    AvgLatencyMs   float64
    SampleCount    int
    LastSeenAt     time.Time
    LastLatencyAt  time.Time
    LatencySamples int
    GeneratedAt    time.Time
}

type EdgeRoleDescription struct {
    Role    string
    Zone    string
    Online  bool
    Caps    []string
    Metrics EdgeRoleMetrics
}
```

## 返回值
`CommandOutput.Value` 的内容：
- `describe`：`EdgeRoleDescription`，含当前 Role（从 RoleAbility 读取）、Zone、Online、Caps、Metrics 快照。
- `set_zone`：`string`，写入的区域值，同时更新 `lastSeenAt`。
- `get_zone`：`string`，当前区域值，未设置时为空字符串。
- `add_capability`：`string`，添加的能力名称，同时更新 `lastSeenAt`。
- `remove_capability`：`string`，被移除的能力名称；不存在时返回错误 `types.ErrInvalidArguments`。
- `list_capabilities`：`[]string`，按名称排序。
- `set_capabilities`：`[]string`，整体覆盖后的新能力列表（排序），同时更新 `lastSeenAt`。
- `record_latency`：`float64`，本次记录的延迟值，同时累加 `latencySumMs`、递增 `latencyCount`、更新 `lastLatencyAt` 与 `lastSeenAt`；负值返回错误。
- `get_metrics`：`EdgeRoleMetrics`，含 `Online`、`Zone`、`Capabilities`（排序副本）、`AvgLatencyMs`（`latencySumMs / latencyCount`，若无样本则为 0）、`SampleCount`/`LatencySamples`、`LastSeenAt`、`LastLatencyAt`、`GeneratedAt`。
- `set_online`：`bool`，写入的在线状态，同时更新 `lastSeenAt`。

## 依赖与前置条件
- 依赖（`Dependencies()`）：`DependencyData/BaseData` 与 `DependencyAbility/RoleAbility`。
- `Check` 要求：Atom 非 nil、已注册 `BaseData`、已挂载 `RoleAbility`，且 `RoleAbility.Command(atom, CommandGetRole, nil)` 返回值必须为 `"edge"`，否则返回 `types.ErrMissingDependency`。
- 所有命令执行前都会先执行 `Check`，因此使用前必须先通过 RoleAbility 的 `set_role` 将角色设为 `"edge"`。
- 延迟统计使用内部累加器（`latencySumMs` 和 `latencyCount`），通过 `get_metrics` 计算当前平均值，不持久化历史。

## 示例代码
```go
// 假设已有 *types.Atom 实例 atom，且 RoleAbility 已挂载
ab, _ := atom.Ability("EdgeRoleAbility")

// 设置区域
zoneOut := ab.Command(atom, ability.EdgeRoleCommandSetZone, ability.EdgeRoleSetZoneArgs{Zone: "cn-beijing"})
if zoneOut.Err != nil {
    // 处理错误
}
fmt.Println("zone:", zoneOut.Value.(string))

// 添加能力
capOut := ab.Command(atom, ability.EdgeRoleCommandAddCap, ability.EdgeRoleCapabilityArg{Name: "video-processing"})
if capOut.Err != nil {
    // 处理错误
}
fmt.Println("added capability:", capOut.Value.(string))

// 记录延迟样本
latOut := ab.Command(atom, ability.EdgeRoleCommandRecordLatency, ability.EdgeRoleRecordLatencyArgs{LatencyMs: 12.5})
if latOut.Err != nil {
    // 处理错误
}
fmt.Println("recorded latency:", latOut.Value.(float64))

// 整体覆盖能力集
capsOut := ab.Command(atom, ability.EdgeRoleCommandSetCaps, ability.EdgeRoleSetCapabilitiesArgs{Capabilities: []string{"ai", "storage", "compute"}})
if capsOut.Err != nil {
    // 处理错误
}
fmt.Println("capabilities:", capsOut.Value.([]string))

// 设置上线
onlineOut := ab.Command(atom, ability.EdgeRoleCommandSetOnline, ability.EdgeRoleSetOnlineArgs{Online: true})
if onlineOut.Err != nil {
    // 处理错误
}
fmt.Println("online:", onlineOut.Value.(bool))

// 获取完整指标
metricsOut := ab.Command(atom, ability.EdgeRoleCommandGetMetrics, nil)
if metricsOut.Err != nil {
    // 处理错误
}
metrics := metricsOut.Value.(ability.EdgeRoleMetrics)
fmt.Println(metrics.Online, metrics.Zone, metrics.AvgLatencyMs, metrics.SampleCount)

// describe 快照
descOut := ab.Command(atom, ability.EdgeRoleCommandDescribe, nil)
if descOut.Err != nil {
    // 处理错误
}
desc := descOut.Value.(ability.EdgeRoleDescription)
fmt.Println(desc.Role, desc.Zone, desc.Online, desc.Caps)
```