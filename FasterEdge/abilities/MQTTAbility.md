# MQTTAbility

## 简介
`MQTTAbility` 为 FasterEdge 提供基于抽象 `MQTTTransport` 的 MQTT 客户端能力。它支持连接、断开、发布、订阅以及本地消息队列管理，所有真实网络交互均通过注入的 `MQTTTransport` 实现，可自由替换底层 MQTT 客户端实现。

## 支持的命令
| 命令字符串 | Go 常量名 | 目的 |
|---|---|---|
| `set_broker` | `MQTTCommandSetBroker` | 设置 broker URL |
| `get_broker` | `MQTTCommandGetBroker` | 获取当前 broker URL |
| `set_credentials` | `MQTTCommandSetCredentials` | 设置用户名/密码 |
| `connect` | `MQTTCommandConnect` | 建立连接 |
| `disconnect` | `MQTTCommandDisconnect` | 断开连接 |
| `is_connected` | `MQTTCommandIsConnected` | 查询连接状态 |
| `publish` | `MQTTCommandPublish` | 发布消息 |
| `subscribe` | `MQTTCommandSubscribe` | 订阅主题并创建本地队列 |
| `unsubscribe` | `MQTTCommandUnsubscribe` | 取消订阅 |
| `list_subscriptions` | `MQTTCommandListSubs` | 列出已订阅主题 |
| `drain` | `MQTTCommandDrain` | 从本地队列拉取消息 |
| `set_client_id` | `MQTTCommandSetClientID` | 设置客户端 ID |

## 参数类型
```go
// set_broker
type MQTTBrokerArgs struct {
    URL string // 如 tcp://host:1883、ssl://...
}

// set_credentials
type MQTTCredentialsArgs struct {
    Username string
    Password string
}

// set_client_id
type MQTTClientIDArgs struct {
    ClientID string // 必填，非空
}

// publish
type MQTTPublishArgs struct {
    Topic   string   // 必填，非空
    Payload []byte   // 消息体
    Qos     MQTTQos  // 0/1/2
    Retain  bool
}

// subscribe
type MQTTSubscribeArgs struct {
    Topic    string // 必填，非空
    Qos      MQTTQos
    MaxQueue int    // 队列容量，0 表示默认 256
}

// unsubscribe / drain 共用
type MQTTTopicArg struct {
    Topic string // 必填，非空
}

// drain
type MQTTPullArgs struct {
    Topic   string        // 必填，非空
    Max     int           // 最大拉取条数，0 表示全部
    Timeout time.Duration // 超时，0 表示立即返回已缓存的消息
}
```

## 返回值
`Command` 方法返回 `types.CommandOutput`，`Value` 字段根据命令不同可能是：
- `string`（如 broker URL、client ID、主题名）
- `bool`（连接状态、操作成功标记）
- `[]string`（已订阅主题列表）
- `[]ability.MQTTMessage`（`drain` 拉取的消息）
- `nil`（仅用于执行无返回值的命令）

## 依赖与前置条件
- 必须调用 `SetTransport(t MQTTTransport)` 注入实现 `Connect、Disconnect、Publish、Subscribe、Unsubscribe` 的 Transport。
- 在使用 `publish`、`subscribe` 前需先 `set_broker` 并 `connect` 成功。
- `SetCredentials`、`SetClientID` 为可选，用于需要身份验证或自定义 client ID 的场景。

## 示例代码
```go
package main

import (
    "time"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

// mockTransport 实现 MQTTTransport，仅演示接口用。
 type mockTransport struct{}
 func (m *mockTransport) Connect() error { return nil }
 func (m *mockTransport) Disconnect() error { return nil }
 func (m *mockTransport) IsConnected() bool { return true }
 func (m *mockTransport) Publish(topic string, payload []byte, qos ability.MQTTQos, retain bool) error { return nil }
 func (m *mockTransport) Subscribe(topic string, qos ability.MQTTQos) error { return nil }
 func (m *mockTransport) Unsubscribe(topic string) error { return nil }

func main() {
    // 创建 ability 并注入 transport
    mqtt := ability.NewMQTTAbility()
    mqtt.SetTransport(&mockTransport{})

    // 设置 broker 并连接
    out := mqtt.Command(&types.Atom{}, ability.MQTTCommandSetBroker, ability.MQTTBrokerArgs{URL: "tcp://broker.example.com:1883"})
    println(out.Value.(string))
    mqtt.Command(&types.Atom{}, ability.MQTTCommandConnect, nil)

    // 订阅主题
    mqtt.Command(&types.Atom{}, ability.MQTTCommandSubscribe, ability.MQTTSubscribeArgs{Topic: "sensor/temperature", Qos: ability.MQTTQos0, MaxQueue: 100})

    // 发布一条消息
    mqtt.Command(&types.Atom{}, ability.MQTTCommandPublish, ability.MQTTPublishArgs{Topic: "sensor/temperature", Payload: []byte("22.5"), Qos: ability.MQTTQos0, Retain: false})

    // 拉取消息（最多 10 条，等待 2 秒）
    pullOut := mqtt.Command(&types.Atom{}, ability.MQTTCommandDrain, ability.MQTTPullArgs{Topic: "sensor/temperature", Max: 10, Timeout: 2 * time.Second})
    msgs := pullOut.Value.([]ability.MQTTMessage)
    for _, m := range msgs {
        println(string(m.Payload))
    }
}
```
