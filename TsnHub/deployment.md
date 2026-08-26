# TsnHub 构建与部署

## 依赖

- C++17
- CMake（当前 CMakeLists 要求 4.1）
- Conan 2.x
- CLI11
- open62541

## 当前构建状态

```bash
cmake -S . -B build
cmake --build build -j
```

当前会因 CMakeLists 引用的桥接源码缺失而失败。这是仓库状态，不是本地依赖问题。

## 完成实现后的预期运行

Unix：

```bash
TsnHub -m UnixSocket -a /tmp/tsn_hub.sock
```

Windows：

```cmd
TsnHub.exe -m NamedPipe -a tsn_hub_pipe
```

CLI：

```bash
TsnHub -m CommandLine \
  --endpoint opc.tcp://127.0.0.1:4840 \
  --tx 2:TsnTx --rx 2:TsnRx
```

## 权限

- `/var/run` 可能需要提升权限，普通用户优先 `/tmp` 或用户 runtime 目录。
- Named Pipe 应设置仅允许目标服务账户的 ACL。
- OPC UA 生产连接应配置加密、证书和用户身份。
