# EKuiperAbility

## 简介
`EKuiperAbility` 提供对 LF Edge eKuiper 流处理引擎的管理能力。通过抽象 `EKuiperTransport` 桥接真实 eKuiper REST API，它支持创建/删除流、创建/删除/启动/停止规则、查看规则状态，以及列出已定义的 stream 和 rule 等操作。

## 支持的命令
| 命令字符串 | Go 常量名 | 目的 |
|---|---|---|
| `set_endpoint` | `EKuiperCommandSetEndpoint` | 设置 eKuiper REST API 端点 |
| `get_endpoint` | `EKuiperCommandGetEndpoint` | 获取当前端点 |
| `create_stream` | `EKuiperCommandCreateStream` | 创建流（stream）定义 |
| `drop_stream` | `EKuiperCommandDropStream` | 删除流定义 |
| `list_streams` | `EKuiperCommandListStreams` | 列出所有已创建流 |
| `get_stream` | `EKuiperCommandGetStream` | 获取指定流详情 |
| `create_rule` | `EKuiperCommandCreateRule` | 创建流处理规则 |
| `drop_rule` | `EKuiperCommandDropRule` | 删除规则 |
| `start_rule` | `EKuiperCommandStartRule` | 启动规则 |
| `stop_rule` | `EKuiperCommandStopRule` | 停止规则 |
| `show_rules` | `EKuiperCommandShowRules` | 列出所有规则及其状态 |
| `get_rule_status` | `EKuiperCommandGetRuleStatus` | 获取单个规则状态 |

## 参数类型
```go
// set_endpoint
type EKuiperEndpointArgs struct {
    URL string // eKuiper REST API 端点，如 http://127.0.0.1:9081
}

// create_stream
type EKuiperStreamArgs struct {
    Name string // 流名称，符合 identifier 规则
    SQL  string // 流的 SQL 定义（例如 CREATE STREAM ...)
}

// drop_stream / get_stream
type EKuiperStreamRef struct {
    Name string // 流名称，必填非空
}

// create_rule
type EKuiperCreateRuleArgs struct {
    ID      string   // 规则唯一标识
    SQL     string   // 规则的 SQL 查询语句
    Actions []string // 输出动作名称列表
}

// drop_rule / start_rule / stop_rule / get_rule_status
type EKuiperRuleIDArg struct {
    ID string // 规则 ID，必填非空
}
```

## 返回值
`Command` 返回 `types.CommandOutput`，`Value` 根据不同命令可能为：
- `string`（endpoint、stream 名称、rule ID 等）
- `bool`（操作成功标记，如 start/stop rule）
- `[]EKuiperStream`（`list_streams` 返回所有流定义切片）
- `EKuiperStream`（`get_stream` 返回单个流定义）
- `[]EKuiperRule`（`show_rules` 返回规则切片）
- `EKuiperRuleStatus`（`get_rule_status` 返回规则状态，取值：`stopped`、`running`、`failed`）
- `nil`（无返回值的命令）

## 依赖与前置条件
- 必须调用 `SetTransport(t EKuiperTransport)` 注入实现 `CreateStream、DropStream、CreateRule、DropRule、StartRule、StopRule、Ping` 的 Transport。
- 使用任何命令前需先 `set_endpoint` 指定有效的 HTTP URL（校验规则与 InfluxDB 的 URL 校验一致）。
- eKuiper 服务器需已启动并可通过该端点访问。

## 示例代码
```go
package main

import (
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

// mockTransport 仅用于演示接口，不做真实 HTTP 请求。
 type mockTransport struct{}
 func (m *mockTransport) CreateStream(name, sql string) error { return nil }
 func (m *mockTransport) DropStream(name string) error { return nil }
 func (m *mockTransport) CreateRule(id, sql string, actions []string) error { return nil }
 func (m *mockTransport) DropRule(id string) error { return nil }
 func (m *mockTransport) StartRule(id string) error { return nil }
 func (m *mockTransport) StopRule(id string) error { return nil }
 func (m *mockTransport) Ping() error { return nil }

func main() {
    ek := ability.NewEKuiperAbility()
    ek.SetTransport(&mockTransport{})

    // 设置端点
    ek.Command(&types.Atom{}, ability.EKuiperCommandSetEndpoint, ability.EKuiperEndpointArgs{URL: "http://ekuiper:9081"})

    // 创建 stream
    ek.Command(&types.Atom{}, ability.EKuiperCommandCreateStream, ability.EKuiperStreamArgs{Name: "temperature_stream", SQL: "CREATE STREAM temperature_stream (temp float) WITH (DATASOURCE=\"temperature\", FORMAT=\"JSON\");"})

    // 创建 rule
    ruleID := "rule_temp_alert"
    sql := "SELECT temp FROM temperature_stream WHERE temp > 30"
    ek.Command(&types.Atom{}, ability.EKuiperCommandCreateRule, ability.EKuiperCreateRuleArgs{ID: ruleID, SQL: sql, Actions: []string{"mqtt_sink"}})

    // 启动 rule
    ek.Command(&types.Atom{}, ability.EKuiperCommandStartRule, ability.EKuiperRuleIDArg{ID: ruleID})

    // 查看 rule 状态
    statusOut := ek.Command(&types.Atom{}, ability.EKuiperCommandGetRuleStatus, ability.EKuiperRuleIDArg{ID: ruleID})
    println("rule status:", string(statusOut.Value.(ability.EKuiperRuleStatus)))
}
```
