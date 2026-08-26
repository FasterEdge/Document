# DontCrack for OpenHarmony

- 源码：https://github.com/FasterEdge/DontCrack4OpenHarmonyLinuxKernelSide
- 目标：OpenHarmony Linux Kernel 侧
- 二进制格式：ELF
- shell：`/bin/sh`
- 默认日志：`/data/logs/proc_manager/`

## 构建

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -ldflags='-s -w' -o DontCrack .
```

## init.cfg

OpenHarmony 通过 JSON 风格 `init.cfg` 定义 service。仓库中的 `example/example_init.cfg` 展示多服务托管方式。

核心参数示例：

```json
{
  "name": "edgecore-setup",
  "path": [
    "/bin/DontCrack",
    "-env", "PATH=/bin:/system/bin",
    "-path", "/bin/edgecore",
    "-start-now",
    "-auto-restart",
    "-max-retries", "3",
    "-port", "10000"
  ],
  "uid": "root",
  "gid": "root",
  "start-mode": "boot"
}
```

## 注意

- init 配置错误可能影响设备启动，修改前保留恢复通道。
- 示例中的 root/0777 仅适用于开发镜像，生产应收紧权限。
- 健康探针默认由 `/bin/sh -c` 执行。
