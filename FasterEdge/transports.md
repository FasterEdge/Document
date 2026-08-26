# Transport 注入

下列 Ability 只定义验证、元数据和生命周期，不绑定具体客户端：

- FileTransferAbility
- ModbusAbility
- SerialAbility
- MQTTAbility
- InfluxDBAbility
- EKuiperAbility
- DockerAbility
- KubernetesAbility

使用方通过组件提供的 `SetXxxTransport(...)` 注入实现。

## 实现责任

Transport 应负责：

- 连接建立与释放。
- 超时、重试、连接池和并发控制。
- 认证信息保管与日志脱敏。
- 将驱动错误转换为可理解的 error。
- 对取消 context 作出响应。

## 测试

单元测试优先注入 fake Transport，不依赖真实 Docker、MQTT、Modbus 或 Kubernetes 环境。

## 安全

网络类 Ability 默认应拒绝空地址、loopback、未授权私网或不受支持协议。确实需要访问 LAN 时，显式开启对应选项，不应通过字符串拼接绕过校验。
