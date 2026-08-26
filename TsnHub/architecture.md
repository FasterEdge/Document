# TsnHub 架构与实现状态

## 设计架构

```text
Unix Socket ─┐
Named Pipe ──┼── Bridge ── Open62541Adapter ── OPC UA Server / TSN nodes
stdin/stdout ┘
```

IPC 输入作为上行消息写入 OPC UA tx 节点；订阅 rx 节点的数据回传到对应 IPC。

## 预期模块

| 模块 | 责任 | 当前源码状态 |
|---|---|---|
| main.cpp | 模式/参数解析 | 只有 include 骨架 |
| UnixSocketBridge | Unix socket 监听、CFG 和数据帧 | 文件缺失 |
| NamedPipeBridge | Windows pipe 监听、CFG 和数据帧 | 文件缺失 |
| CommandLineBridge | stdin/stdout 交互 | 文件缺失 |
| Open62541Adapter | 连接、写入、订阅、重连 | 文件缺失 |

## 发布门槛

在标记为可用前至少需要：

1. 恢复/实现 CMake 引用源码。
2. 完成模式解析和 `--version`。
3. 测试 Unix socket 清理与权限。
4. 测试 Windows pipe 命名和 ACL。
5. 测试 OPC UA 连接、NodeId、重连和状态码。
6. 增加 LICENSE 和端到端测试。
