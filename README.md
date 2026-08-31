# FasterEdge 文档中心

> 文档基线版本：`1.0.20260831`

本仓库集中维护 FasterEdge 主框架与配套组件的公开文档。组件源码仓库仍是实现、参数和版本的最终依据；这里提供跨仓库导航、架构说明、部署手册、安全建议和组合示例。

## 快速导航

| 组件 | 作用 | 文档 | 源码 |
|---|---|---|---|
| FasterEdge | Atom / Data / Ability / Command / Transport 主框架 | [文档](FasterEdge/README.md) | [仓库](https://github.com/FasterEdge/FasterEdge) |
| DontCrack | 跨平台进程托管、自动重启、健康探针与可观测性 | [文档](DontCrack/README.md) | [OH](https://github.com/FasterEdge/DontCrack4OpenHarmonyLinuxKernelSide) · [Android](https://github.com/FasterEdge/DontCrack4AndroidLinuxKernelSide) · [Linux](https://github.com/FasterEdge/DontCrack4ManyLinux) · [Windows](https://github.com/FasterEdge/DontCrack4Windows) |
| ProxyArea | 通用 HTTP 请求转发、目标白名单与 TLS | [文档](ProxyArea/README.md) | [仓库](https://github.com/FasterEdge/ProxyArea) |
| SimpleWebShell | 带会话和文件传输的 WebShell | [文档](SimpleWebShell/README.md) | [仓库](https://github.com/FasterEdge/SimpleWebShell) |
| SimpleTimeService | 本机/NTP UTC 时间服务 | [文档](SimpleTimeService/README.md) | [仓库](https://github.com/FasterEdge/SimpleTimeService) |
| TsnHub | 本地 IPC 与 OPC UA/TSN 桥接设计 | [文档](TsnHub/README.md) | [仓库](https://github.com/FasterEdge/TsnHub) |
| Example | 示例仓库状态与现有 demo 入口 | [文档](Example/README.md) | [仓库](https://github.com/FasterEdge/Example) |
| FasterEdgeDoctor | 本地/远程仓库与运行状态诊断工具 | — | [仓库](https://github.com/FasterEdge/FasterEdgeDoctor) |
| MCU / FPGA 移植 | Arduino、PlatformIO、Keil、MounRiver、Vivado 等硬件实现 | — | [组织仓库](https://github.com/FasterEdge) |
| RelayNode | SW2MQTT、SW2USB 硬件节点工程 | — | [组织仓库](https://github.com/FasterEdge) |

## 推荐阅读顺序

1. [生态系统](00-overview/ecosystem.md)
2. [总体架构](00-overview/architecture.md)
3. [版本策略](00-overview/versioning.md)
4. [安全基线](00-overview/security.md)
5. [FasterEdge 快速开始](FasterEdge/README.md)
6. [DontCrack 进程托管](DontCrack/README.md)
7. [ProxyArea 通用代理](ProxyArea/README.md)

## 文档边界

- 本仓库仅包含公开技术文档，不复制 `PrivateDocument` 中的专利、软著、登记材料和内部信息。
- 示例中的密钥、地址和路径均为占位值，不应直接用于生产。
- README 中的历史版本仅用于变更记录；当前运行基线统一为 `1.0.20260831`。

## 维护原则

- 先改组件源码和组件 README，再同步本仓库。
- 所有参数、端点、命令名必须可在源码中找到。
- 未在当前源码中出现的功能明确标为“设计目标”或“待实现”，不得写成已完成。
- 不提交二进制、压缩包、证书、密钥、日志、IDE 和操作系统元数据。
