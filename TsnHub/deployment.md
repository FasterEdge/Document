# TsnHub 构建与部署

## 依赖

- C++17
- CMake 3.23+
- Conan 2.x
- CLI11 2.6
- open62541 1.5

## Portable 模式构建

```bash
conan install . \
  --output-folder=build-tsn \
  --build=missing \
  -s build_type=Debug

cmake -S . -B build-tsn/cmake \
  -DCMAKE_TOOLCHAIN_FILE=build-tsn/build/Debug/generators/conan_toolchain.cmake \
  -DCMAKE_BUILD_TYPE=Debug

cmake --build build-tsn/cmake -j
ctest --test-dir build-tsn/cmake --output-on-failure
```

默认构建包含跨平台 UDP、Portable PubSub、用户态 TSN Scheduler 和四组自动测试。

查看构建能力：

```bash
./build-tsn/cmake/TsnHub --capabilities
```

## Portable 模式运行

```bash
./build-tsn/cmake/TsnHub \
  --listen 127.0.0.1:4841 \
  --forward 127.0.0.1:4842 \
  --gate 1000:0x0f \
  --gate 1000:0xf0 \
  --delay-us 200 \
  --jitter-us 50 \
  --loss 0.01 \
  --queue 1024 \
  --seed 42
```

Portable 模式可面向 Linux、macOS 和 Windows；Windows 构建自动链接 Winsock2。

## Linux 原生 open62541 PubSub

原生模式使用 open62541 UDP/UADP DataSetReader/DataSetWriter：

```bash
cmake -S . -B build-tsn/native \
  -DCMAKE_TOOLCHAIN_FILE=build-tsn/build/Debug/generators/conan_toolchain.cmake \
  -DCMAKE_BUILD_TYPE=Debug \
  -DTSNHUB_USE_OPEN62541_PUBSUB=ON
cmake --build build-tsn/native -j
ctest --test-dir build-tsn/native --output-on-failure
```

该模式默认仅允许 Linux。`TSNHUB_ALLOW_NATIVE_PUBSUB_NON_LINUX=ON` 只用于 API 或编译验证，不应解释为非 Linux 运行支持。

## Docker 三节点仿真

仓库的 `docker-compose.yml` 提供 Publisher → TsnHub → Subscriber 拓扑：

```bash
docker compose up --build --abort-on-container-exit
```

发布前应在干净 Linux/Docker 环境重新执行构建和集成测试，不应仅依赖已有本地构建目录。

## 网络与安全边界

- 默认示例绑定 loopback；跨主机运行时应使用防火墙限制 UDP 入口和出口。
- 仿真调度器不提供 OPC UA 身份认证或加密策略管理；生产系统必须在 open62541 配置和外围网络中落实证书、访问控制与隔离。
- 丢包、时延和抖动参数会主动改变通信行为，只应在测试、验证或明确授权的仿真环境中启用。
