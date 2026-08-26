# TsnHub

- 项目版本：`1.0.20260826`
- 源码：https://github.com/FasterEdge/TsnHub
- 语言：C++17、CMake、CLI11、open62541

## 当前状态

仓库 README 和 CMakeLists 描述目标为三种本地 IPC 与 OPC UA 桥接：UnixSocket、Windows NamedPipe、CommandLine。

**但当前可见源码树只有 7 行入口骨架 `main.cpp`，CMakeLists 引用的以下文件不存在：**

- `unix_socket/UnixSocketBridge.cpp`
- `name_pipe/NamedPipeBridge.cpp`
- `command_line/CommandLineBridge.cpp`
- `tsn/Open62541Adapter.cpp`

因此当前仓库不能按 CMakeLists 成功构建。下列文档区分“设计接口”与“已验证实现”，避免把 README 目标当作完成状态。

## 设计目标

- UnixSocket 模式：Linux 本地 Unix socket。
- NamedPipe 模式：Windows Named Pipe。
- CommandLine 模式：stdin/stdout。
- 运行时 CFG 指定 OPC UA endpoint、tx/rx NodeId。
- open62541 连接、订阅、写入和重连。

## 文档

- [架构与状态](architecture.md)
- [配置协议](configuration.md)
- [构建与部署](deployment.md)
