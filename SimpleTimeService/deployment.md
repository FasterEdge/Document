# SimpleTimeService 部署

## 构建

```bash
CGO_ENABLED=0 go build -trimpath -ldflags='-s -w' -o SimpleTimeService .
```

## DontCrack 托管

```bash
DontCrack \
  -path /usr/local/bin/SimpleTimeService \
  -args "--port 8080 --offset 0" \
  -start-now -auto-restart \
  -probe-cmd "curl -sf http://127.0.0.1:8080/time"
```

## 网络

- 允许 UDP/123 出站访问 NTP。
- 公网提供时间接口时，通过反向代理限流。
- 该服务不保证授时级精度；网络、DNS 和上游 NTP 状态都会影响结果。
