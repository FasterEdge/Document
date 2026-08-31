# FasterEdge 生态系统

FasterEdge 采用“核心框架 + 外置组件”的组织方式。主框架负责统一的组件模型、生命周期和调用协议；外置组件负责进程管理、代理、WebShell、时间服务和工业协议桥接。

## 组件关系

```text
业务 / 用户
   │
   ▼
FasterEdge Atom
   ├── Data: Base / NetMap / Keyring / Config
   ├── Ability: Time / NetMap / OneKey / Role / Terminal / Industrial / Container ...
   └── Transport: 由使用方注入真实网络、MQTT、Docker、K8s、串口等驱动

外置运行组件
   ├── DontCrack ── 托管任意进程，自动重启/探针/指标/UI
   ├── FasterEdgeDoctor ── 本地/远程健康诊断与只读 HTTP 检查
   ├── ProxyArea ── HTTP 通用代理转发
   ├── SimpleWebShell ── 远程终端与文件传输
   ├── SimpleTimeService ── 本机/NTP 时间 API
   ├── TsnHub ── 本地 IPC 与 OPC UA/TSN 桥接（当前实现需核对源码完整度）
   └── RelayNode ── SW2MQTT、SW2USB 硬件节点工程
```

## 组合方式

### 原生运行

FasterEdge 的终端、文件传输和算法分发 Ability 可直接调用本机能力，获得接近原生的性能和板载驱动访问。

### 进程高可用

用 DontCrack 托管 ProxyArea、SimpleWebShell、SimpleTimeService 或业务程序：

```bash
DontCrack -path /usr/local/bin/ProxyArea \
  -args "--addr=:8080 --key=<replace-me>" \
  -start-now -auto-restart -max-retries 3 \
  -probe-cmd "wget -q --spider http://127.0.0.1:8080/ || exit 1"
```

### 容器与集群

- ProxyArea 提供多阶段 Dockerfile。
- FasterEdge 的 DockerAbility/KubernetesAbility 通过注入的 Transport 对接外部运行时。
- DontCrack 的 `/healthz`、`/readyz`、`/metrics` 可接入 Kubernetes 或 Prometheus。

## 仓库成熟度

| 组件 | 当前状态 |
|---|---|
| FasterEdge | 已有完整 Go 框架、Ability/Data 组件、单测与示例 |
| 四个平台 DontCrack | 已实现跨平台进程管理和统一 HTTP API |
| ProxyArea | 已实现纯标准库 HTTP 代理、白名单、超时、TLS 和通用方法转发 |
| SimpleWebShell | 已实现终端、会话、文件上传下载接口 |
| SimpleTimeService | 已实现本机时间、偏移时间和 NTP 时间接口 |
| TsnHub | README/CMake 描述完整桥接目标，但当前源码树只有入口骨架；见 [TsnHub 状态说明](../TsnHub/README.md) |
| Example | 独立仓库当前仅为占位；可先使用 FasterEdge 主仓库中的 `examples/demo` |
| FasterEdgeDoctor | 已实现本地仓库发现、Go 测试、远程只读检查和 OneKey 认证检查；不执行远程 shell |
| RelayNode-SW2MQTT | 已完成 ESP8266、CH340X、光耦隔离输入的 EDA 工程与 BOM；应以 EDA 工程为生产依据 |
| RelayNode-SW2USB | 已完成 CH552G USB/开关节点 EDA 工程与 BOM；主控固件能力取决于具体应用 |
