# CloudRoleAbility

## 简介
CloudRoleAbility 扩展 RoleAbility，提供云端节点特有的「控制器注册、服务清单、心跳与状态管理」能力。它持有自己的状态（控制器地址、服务映射、运行状态、心跳次数等），通过 `Check` 要求 RoleAbility 必须存在且角色为 `"cloud"`。状态由 `sync.RWMutex` 保护，线程安全。

## 支持的命令
| 常量 | 命令字符串 | 说明 |
|------|-------------|------|
| `ability.CloudRoleCommandDescribe` | `"describe"` | 返回 `CloudRoleDescription` 结构快照（含 Role/Controller/Status/Services/心跳统计）。
| `ability.CloudRoleCommandSetController` | `"set_controller"` | 设置云端控制器 URL，须为 http/https 且非回环/私网地址。
| `ability.CloudRoleCommandGetController` | `"get_controller"` | 获取当前控制器 URL，未设置时为空字符串。
| `ability.CloudRoleCommandRegister` | `"register_service"` | 注册一个对外服务，参数为 `CloudRoleRegisterServiceArgs`。
| `ability.CloudRoleCommandUnregister` | `"unregister_service"` | 注销一个已注册服务，返回被注销的服务结构。
| `ability.CloudRoleCommandListServices` | `"list_services"` | 按名称排序返回服务列表 `[]CloudRoleService`。
| `ability.CloudRoleCommandSetStatus` | `"set_status"` | 设置运行状态（healthy/degraded/offline，含别名 ok/up/down）。
| `ability.CloudRoleCommandGetStatus` | `"get_status"` | 获取当前运行状态，初始为 `"unknown"`。
| `ability.CloudRoleCommandHeartbeat` | `"heartbeat"` | 记录一次心跳，返回本次心跳时间。

## 参数类型

```go
// set_controller 的参数
type CloudRoleSetControllerArgs struct {
    URL string // 必填，http(s)://host[:port][/path]，拒绝 localhost/127.0.0.1/::1/0.0.0.0 及私网、链路本地
}

// register_service 的参数
type CloudRoleRegisterServiceArgs struct {
    Name     string // 必填，去除首尾空白后须与原名一致（不允许首尾空格）
    Version  string // 可选，去除首尾空白后存储
    Endpoint string // 可选，去除首尾空白后存储
}

// unregister_service 的参数
type CloudRoleUnregisterServiceArgs struct {
    Name string // 必填，服务名，去除首尾空白后非空
}

// set_status 的参数
type CloudRoleSetStatusArgs struct {
    Status CloudRoleStatus // 必填，normalizeStatus 归一化后须为 healthy/degraded/offline
}
```
| 命令 | 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|------|
| `set_controller` | `URL` | `string` | 是 | 控制器地址，须通过 `isAcceptableControllerURL` 校验。 |
| `register_service` | `Name` | `string` | 是 | 服务名，`TrimSpace` 后不能为空且不能带首尾空格。 |
| `register_service` | `Version` | `string` | 否 | 服务版本，会 `TrimSpace` 后存储。 |
| `register_service` | `Endpoint` | `string` | 否 | 服务端点，会 `TrimSpace` 后存储。 |
| `unregister_service` | `Name` | `string` | 是 | 要注销的服务名，须已注册。 |
| `set_status` | `Status` | `CloudRoleStatus` | 是 | 目标状态，支持 `"healthy"/"ok"/"up"`、`"degraded"`、`"offline"/"down"`。 |

`describe`、`get_controller`、`list_services`、`get_status`、`heartbeat` 不接受参数，`args` 必须为 `nil`。

**状态常量**（`CloudRoleStatus`，package 内公开）：
```go
CloudRoleStatusUnknown  CloudRoleStatus = "unknown"
CloudRoleStatusHealthy  CloudRoleStatus = "healthy"
CloudRoleStatusDegraded CloudRoleStatus = "degraded"
CloudRoleStatusOffline  CloudRoleStatus = "offline"
```

**返回结构**：
```go
type CloudRoleService struct {
    Name      string
    Version   string
    Endpoint  string
    UpdatedAt time.Time
}

type CloudRoleDescription struct {
    Role        string
    Controller  string
    Status      CloudRoleStatus
    Services    []CloudRoleService
    LastBeatAt  time.Time
    Heartbeats  int
    GeneratedAt time.Time
}
```

## 返回值
`CommandOutput.Value` 的内容：
- `describe`：`CloudRoleDescription`，含当前 Role（从 RoleAbility 读取）、Controller、Status、排序后的 Services、LastBeatAt、Heartbeats 与 GeneratedAt。
- `set_controller`：`string`，写入的控制器 URL。
- `get_controller`：`string`，当前控制器 URL，未设置时为空字符串。
- `register_service`：`CloudRoleService`，注册后的服务结构（含 `UpdatedAt = time.Now()`）。
- `unregister_service`：`CloudRoleService`，被注销的服务结构；若不存在返回错误 `types.ErrInvalidArguments`。
- `list_services`：`[]CloudRoleService`，按 Name 排序。
- `set_status`：`CloudRoleStatus`，归一化后的状态；非法状态返回错误。
- `get_status`：`CloudRoleStatus`，当前状态，新实例初始为 `CloudRoleStatusUnknown`。
- `heartbeat`：`time.Time`，本次心跳时间，同时 `heartbeats+1` 并更新 lastBeatAt。

## 依赖与前置条件
- 依赖（`Dependencies()`）：`DependencyData/BaseData` 与 `DependencyAbility/RoleAbility`。
- `Check` 要求：Atom 非 nil、已注册 `BaseData`、已挂载 `RoleAbility`，且 `RoleAbility.Command(atom, CommandGetRole, nil)` 返回值必须为 `"cloud"`，否则返回 `types.ErrMissingDependency`。
- 所有命令执行前都会先执行 `Check`，因此使用前必须先通过 RoleAbility 的 `set_role` 将角色设为 `"cloud"`。
- `set_controller` 的 URL 校验（`isAcceptableControllerURL`）拒绝 `localhost`、`127.0.0.1`、`::1`、`0.0.0.0`，用于降低 SSRF 风险。

## 示例代码
```go
// 假设已有 *types.Atom 实例 atom，且 RoleAbility 已挂载
ab, _ := atom.Ability("CloudRoleAbility")

// 心跳
beatOut := ab.Command(atom, ability.CloudRoleCommandHeartbeat, nil)
if beatOut.Err != nil {
    // 处理错误
}
fmt.Println("heartbeat at:", beatOut.Value.(time.Time))

// 设置控制器
setCtlOut := ab.Command(atom, ability.CloudRoleCommandSetController, ability.CloudRoleSetControllerArgs{URL: "https://controller.example.com"})
if setCtlOut.Err != nil {
    // 处理错误
}
fmt.Println("controller:", setCtlOut.Value.(string))

// 注册服务
regOut := ab.Command(atom, ability.CloudRoleCommandRegister, ability.CloudRoleRegisterServiceArgs{Name: "inference", Version: "v1", Endpoint: ":9000"})
if regOut.Err != nil {
    // 处理错误
}
svc := regOut.Value.(ability.CloudRoleService)
fmt.Println("registered:", svc.Name, svc.Version)

// 查询服务列表
listOut := ab.Command(atom, ability.CloudRoleCommandListServices, nil)
if listOut.Err != nil {
    // 处理错误
}
services := listOut.Value.([]ability.CloudRoleService)
fmt.Println("services:", services)

// 设置并读取状态
stOut := ab.Command(atom, ability.CloudRoleCommandSetStatus, ability.CloudRoleSetStatusArgs{Status: ability.CloudRoleStatusHealthy})
if stOut.Err != nil {
    // 处理错误
}
fmt.Println("status:", stOut.Value.(ability.CloudRoleStatus))

// describe 快照
descOut := ab.Command(atom, ability.CloudRoleCommandDescribe, nil)
if descOut.Err != nil {
    // 处理错误
}
desc := descOut.Value.(ability.CloudRoleDescription)
fmt.Println(desc.Role, desc.Controller, desc.Status, desc.Heartbeats)
```