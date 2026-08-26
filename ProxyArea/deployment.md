# ProxyArea 部署

## 直接运行

```bash
CGO_ENABLED=0 go build -trimpath -ldflags='-s -w' -o ProxyArea .
./ProxyArea --addr=:8080 --key=<secret> --allow-hosts=api.example.com
```

## Docker

仓库 Dockerfile 使用多阶段构建：

```bash
docker build -t proxyarea:1.0.20260826 .
docker run -d --name proxyarea \
  -p 127.0.0.1:8080:8080 \
  proxyarea:1.0.20260826 \
  --key=<secret> --allow-hosts=api.example.com
```

## DontCrack 托管

```bash
DontCrack \
  -path /usr/local/bin/ProxyArea \
  -args "--addr=:8080 --key=<secret> --allow-hosts=api.example.com" \
  -start-now -auto-restart -max-retries 3 \
  -port 11884 \
  -probe-cmd "wget -q --spider http://127.0.0.1:8080/"
```

## TLS

```bash
ProxyArea --addr=:8443 --https \
  --cert_file=/run/secrets/tls.crt \
  --key_file=/run/secrets/tls.key \
  --key=<secret> --allow-hosts=api.example.com
```

证书私钥不要提交到仓库或镜像层。
