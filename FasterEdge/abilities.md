# Ability 组件

## 基础与身份

| Ability | 命令摘要 |
|---|---|
| BaseAbility | `list_data_names`, `list_ability_names` |
| RoleAbility | `describe`, `set_role`, `get_role` |
| CloudRoleAbility | `describe`, `set_controller`, `register_service`, `set_status`, `heartbeat` |
| EdgeRoleAbility | `describe`, `set_zone`, `add_capability`, `record_latency`, `get_metrics`, `set_online` |
| TimeAbility | `sync_manual`, `sync_system`, `sync_net`, `sync_ntp`, `get_time`, `configure_run` |
| NetMapAbility | `register_peer`, `unregister_peer`, `update_peer`, `list_peers`, `lookup_peer`, `get_topology` |
| OneKeyAbility | `issue_token`, `verify_token`, `revoke_token`, `revoke_all`, `list_tokens`, `status`, `rotate` |

## 终端

| Ability | 说明 |
|---|---|
| CmdAbility | `run/start/wait/kill/list`，支持 allowlist |
| ShAbility | 基于 CmdAbility，以 `sh -c` 执行 |
| BashAbility | 基于 ShAbility，以无 profile 的 bash 执行 |

## 文件与算法

| Ability | 命令摘要 |
|---|---|
| ConfigFileAbility | `set_path`, `load`, `save`, `exists` |
| FileTransferAbility | `set_target`, `upload`, `download`, `list`, `get_transfer`, `cancel` |
| AlgorithmDistributionAbility | `register_algorithm`, `unregister_algorithm`, `distribute`, `list_distributions`, `cancel` |

## 工业协议

| Ability | 命令摘要 |
|---|---|
| ModbusAbility | `set_endpoint`, `set_unit_id`, `read_holding`, `read_input`, `read_coils`, `read_discrete`, `write_*` |
| SerialAbility | `open`, `close`, `read`, `write`, `set_config`, `list_ports` |
| TSNAbility | `set_interface`, `register_talker`, `register_listener`, `unregister`, `set_priority_map`, `set_time_aware` |

## 数据交互

| Ability | 命令摘要 |
|---|---|
| MQTTAbility | `set_broker`, `connect`, `disconnect`, `publish`, `subscribe`, `drain`, `list_subscriptions` |
| InfluxDBAbility | `set_endpoint`, `set_token`, `set_org`, `set_bucket`, `ping`, `write`, `query`, `list_series`, `delete_series` |
| EKuiperAbility | `set_endpoint`, `create_stream`, `drop_stream`, `create_rule`, `start_rule`, `stop_rule`, `show_rules` |

## 容器编排

| Ability | 命令摘要 |
|---|---|
| DockerAbility | `set_endpoint`, `list_containers`, `start`, `stop`, `restart`, `remove`, `pull_image`, `inspect`, `get_logs`, `create` |
| KubernetesAbility | `set_context`, `apply`, `delete`, `list`, `get`, `scale`, `get_logs` |

详细参数类型以各 Ability 源码中的 `...Args` 结构体为准。
