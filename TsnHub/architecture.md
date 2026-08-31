# TsnHub 架构与实现状态

## 当前架构

```text
UDP / UADP 输入
       │
       ▼
PortablePubSub 或 open62541 DataSetReader
       │
       ▼
SimulatorNode
       │
       ▼
用户态 TSN Scheduler
  ├── 8 个优先级队列
  ├── Gate Control List
  ├── 固定时延与随机抖动
  ├── 丢包概率
  └── 队列容量限制
       │
       ▼
PortablePubSub 或 open62541 DataSetWriter
       │
       ▼
UDP / UADP 输出
```

## 已实现模块

| 模块 | 责任 | 当前状态 |
|---|---|---|
| `main.cpp` | CLI、能力输出、信号处理和节点运行入口 | 已实现 |
| `tsn/Scheduler.*` | 八级队列、GCL、时延、抖动、丢包和统计 | 已实现并有 CTest |
| `tsn/SimulatorNode.*` | 连接 UDP 输入、调度器和 UDP 输出 | 已实现并有 CTest |
| `transport/UdpSocket.*` | BSD Socket / Winsock2 跨平台 UDP | 已实现并有 loopback 测试 |
| `transport/PortablePubSub.*` | `TSN1` envelope 编解码 | 已实现并有 CTest |
| `pubsub/Open62541Bridge.*` | Linux 原生 open62541 UDP/UADP DataSetReader/DataSetWriter | 可选构建模式 |
| `docker/PubSubFixture.cpp` | 原生 Publisher / Subscriber 集成夹具 | 已实现 |

## 构建模式

- **Portable 模式（默认）**：使用自有 UDP envelope，可在 Linux、macOS 和 Windows 构建。
- **Native PubSub 模式**：启用 `TSNHUB_USE_OPEN62541_PUBSUB=ON`，使用 open62541 原生 UDP/UADP，目标环境为 Linux/Docker。
- 非 Linux 环境默认拒绝 Native PubSub；`TSNHUB_ALLOW_NATIVE_PUBSUB_NON_LINUX` 仅用于 API/编译验证，不表示运行支持。

## 验证边界

仓库包含 Scheduler、Portable PubSub、UDP loopback 和 SimulatorNode 的自动测试，以及 Linux/Docker 原生 PubSub 集成脚本。发布前仍应在目标工具链中重新执行 Conan/CMake 构建与 CTest；存在本地构建产物不等同于所有目标平台均已验证。
