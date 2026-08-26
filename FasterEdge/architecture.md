# FasterEdge 架构

## Atom

Atom 保存 Data 和 Ability 注册表。注册名称必须唯一；查找返回接口而不是内部 map。

## Data

Data 持有状态并实现统一生命周期和 Command。当前标准骨架包含 BaseData、NetMapData、KeyringData、ConfigData。

## Ability

Ability 负责行为，可选实现 Runner/Unmounter 等扩展接口。当前覆盖角色、时间、网络拓扑、终端、文件传输、算法分发、工业协议、数据交互和容器编排。

## Command

```go
Command(atom *types.Atom, act string, args any) types.CommandOutput
```

命令以常量定义动作名，以结构体定义参数，输出统一包含 `Name`、`Value` 和 `Err`。

## Transport

涉及外部系统的 Ability 不直接绑定 SDK，而是定义 Transport 接口：

```text
Ability metadata + validation
           │
           ▼
       Transport interface
           │
           ▼
用户提供的 MQTT/Docker/K8s/串口/Modbus/HTTP 实现
```

这种结构使单测可使用 fake Transport，也允许按平台选择真实驱动。

## 依赖

Ability 在 Check/Mount 阶段检查所需 Data 或 Ability。例如 CloudRoleAbility/EdgeRoleAbility 依赖 RoleAbility，AlgorithmDistributionAbility 建立在 FileTransferAbility 之上。
