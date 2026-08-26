# 总体架构

## 核心抽象

- **Atom**：一个节点内所有 Data 与 Ability 的容器，负责注册、查找和生命周期编排。
- **Data**：节点状态与配置，包括基础元信息、网络拓扑、密钥环和配置数据。
- **Ability**：节点能力，通过统一 `Command(atom, act, args)` 入口暴露严格类型命令。
- **CommandOutput**：统一命令返回，包含 `Name`、`Value` 和 `Err`。
- **Transport**：外部依赖抽象，由使用方注入真实客户端或驱动，避免框架绑定具体 SDK。

## 生命周期

```text
InitAtom / InitStandardAtom
            │
            ▼
       注册 Data/Ability
            │
            ▼
         PreRunAtom
        Check → Mount
            │
            ▼
          RunAtom
     Runner 并发监督运行
            │ context cancel / error
            ▼
     逆序 Unmount + 超时控制
```

## 可靠性边界

- FasterEdge 管理组件注册、命令调用和 Runner 生命周期。
- DontCrack 管理独立操作系统进程的启动、退出、探针和自动重启。
- Docker/Kubernetes 等外部操作依赖注入的 Transport。
- ProxyArea 与 SimpleWebShell 属于高权限网络组件，必须配置认证和网络边界。

## 数据流示例

```text
NetMapAbility 注册节点
       │
       ▼
OneKeyAbility 签发短期令牌
       │
       ▼
FileTransferAbility 上传算法包
       │
       ▼
AlgorithmDistributionAbility 记录并分发版本
       │
       ▼
DontCrack 托管目标业务进程
```

## 线程安全

核心组件使用 `sync.RWMutex` / `atomic` 保护共享状态。命令参数采用明确结构体，nil、类型错误和空白必填字段统一返回参数错误。外部调用的重试、连接池和具体并发模型由对应 Transport 负责。
