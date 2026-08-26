# DontCrack for Android

- 源码：https://github.com/FasterEdge/DontCrack4AndroidLinuxKernelSide
- 运行形式：adb shell 中的 Linux ELF，不是 APK
- shell 探测：`/system/bin/sh` → `/bin/sh` → `/system/bin/ash`

## 构建与部署

```bash
GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -ldflags='-s -w' -o DontCrack .
adb push DontCrack /data/local/tmp/DontCrack
adb shell chmod 755 /data/local/tmp/DontCrack
```

启动：

```bash
adb shell /data/local/tmp/DontCrack \
  -path /data/local/tmp/childproc \
  -start-now -auto-restart \
  -port 11883
```

## USB 访问管理端口

```bash
adb reverse tcp:11883 tcp:11883
curl http://127.0.0.1:11883/heartbeat
```

## 开机自启

仓库 `example/example_init.rc` 使用 Android Init Language。修改 init.rc 需要 root/自定义镜像，并可能需要 sepolicy。

## SELinux

`/data/local/tmp` 通常适合调试。生产镜像应将二进制放入批准目录并配置最小 sepolicy，不建议长期 `setenforce 0`。
