# Data 组件

| Data | 用途 | 命令 |
|---|---|---|
| BaseData | 框架 logo 与版本信息 | `logo`, `info` |
| NetMapData | 节点名、网卡与默认接口 | `info`, `set_node_name`, `interfaces`, `set_default_iface` |
| KeyringData | 共享密钥、令牌签发和吊销 | `status`, `set_secret`, `rotate`, `issue_token`, `revoke_token`, `revoke_all` |
| ConfigData | 扁平点号路径 KV 配置 | `get`, `set`, `delete`, `list`, `snapshot` |

## BaseData

`info` 返回 FasterEdge 当前版本 `1.0.20260831` 和框架描述。

## NetMapData

保存本机网络视图，不负责真实网络发现。网卡枚举和拓扑同步可由 NetMapAbility 或外部 Transport 完成。

## KeyringData

承载 OneKeyAbility 使用的共享秘密和令牌记录。轮换密钥会影响后续签发与验证语义，调用方应规划兼容窗口。

## ConfigData

支持 `server.host` 一类点号路径键值，提供快照用于 ConfigFileAbility 持久化。
