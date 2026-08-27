# ProxyArea 部署

## 直接运行

```bash
CGO_ENABLED=0 go build -trimpath -ldflags='-s -w' -o ProxyArea .
./ProxyArea --addr=:8080 --key=<secret> --allow-hosts=api.example.com --timeout=30s
```

CLI 保持兼容：`--addr`、`--key`、`--https`、`--cert_file`、`--key_file`、`--timeout`、`--target-scheme`、`--allow-hosts`、`--insecure-skip-verify`。认证客户端优先使用 Bearer 或 X-Proxy-Key；query key 仅用于旧客户端。

## Docker

```bash
docker build -t proxyarea:1.0.20260826 .
docker run -d --name proxyarea -p 127.0.0.1:8080:8080 proxyarea:1.0.20260826 \
  --key=<secret> --allow-hosts=api.example.com
```

## DontCrack 托管

```bash
DontCrack -path /usr/local/bin/ProxyArea \
  -args "--addr=:8080 --key=<secret> --allow-hosts=api.example.com" \
  -start-now -auto-restart -max-retries 3 -port 11884 \
  -probe-cmd "wget -q --spider http://127.0.0.1:8080/"
```

## TLS 与网络边界

```bash
ProxyArea --addr=:8443 --https --cert_file=/run/secrets/tls.crt \
  --key_file=/run/secrets/tls.key --key=<secret> --allow-hosts=api.example.com
```

私钥不得进入仓库或镜像层。白名单会覆盖重定向目标，但仍应在防火墙限制出站网段和端口。`--insecure-skip-verify` 只能用于受控自签名测试环境。
