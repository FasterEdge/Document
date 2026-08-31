# TsnHub

- 项目版本：`1.0.20260831`
- 源码：https://github.com/FasterEdge/TsnHub
- 语言：C++17、CMake、CLI11、open62541

## 当前状态

当前源码已实现以下能力：

- 八级优先级队列、Gate Control List、固定时延、抖动、丢包和队列容量仿真；
- Linux/macOS BSD Socket 与 Windows Winsock2 的跨平台 UDP 封装；
- Portable PubSub envelope 与 `SimulatorNode` 转发链路；
- Linux 下基于 open62541 1.5 的原生 UDP/UADP DataSetReader/DataSetWriter；
- Scheduler、Portable PubSub、UDP loopback、SimulatorNode 四组 CTest；
- Publisher → TsnHub → Subscriber 的 Docker 原生 PubSub 集成环境。

Portable 模式可在 Linux、macOS 和 Windows 构建；原生 open62541 PubSub 模式以 Linux/Docker 为目标。构建时需要 CMake 3.23+、Conan 2.x、CLI11 2.6 和 open62541 1.5。

## 文档

- [架构与状态](architecture.md)
- [配置协议](configuration.md)
- [构建与部署](deployment.md)
