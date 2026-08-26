# TsnHub 配置协议（设计）

README 定义 UnixSocket/NamedPipe 单行 CFG：

```text
CFG endpoint=opc.tcp://127.0.0.1:4840 tx=2:TsnTx rx=2:TsnRx
```

字段：

| 字段 | 说明 |
|---|---|
| `endpoint` | OPC UA endpoint URL |
| `tx` | 上行写入 NodeId |
| `rx` | 下行订阅 NodeId |

NodeId 示例 `2:TsnTx` 表示 namespace 2、字符串标识 `TsnTx`。

Unix socket 设计示例：

```bash
printf 'CFG endpoint=opc.tcp://127.0.0.1:4840 tx=2:TsnTx rx=2:TsnRx\n' \
  | nc -U /tmp/tsn_hub.sock
```

## 待明确

当前源码未实现，发布前需定义：

- 字段转义和空格处理。
- 缺字段/重复字段错误格式。
- CFG 成功确认。
- 数据帧边界、编码、最大长度。
- 重配置时现有连接和订阅的行为。
