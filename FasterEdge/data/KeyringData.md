## KeyringData

### 简介

`KeyringData` 保存本节点用于加密访问的共享密钥以及已签发的令牌表。它本身不实现签名/校验逻辑，实际的 HMAC 计算由 `OneKeyAbility` 调用 `Sign`、`Verify` 等方法完成。支持持久化快照（包括密钥）以便在节点重启后恢复状态。

### 支持的命令

| 命令字符串 | Go 常量名 | 说明 |
|-----------|----------|------|
| `status`      | `KeyringCommandStatus` | 返回 `KeyringStatus`（算法、密钥指纹、活跃令牌数、累计签发数、最近一次轮换时间）。 |
| `set_secret`  | `KeyringCommandSetSecret` | 设置（覆盖）当前密钥。参数 `KeyringSetSecretArgs{Secret string}`，`Secret` 可以是 Base64 编码或原始字符串。返回新密钥的指纹。 |
| `rotate`      | `KeyringCommandRotate` | 生成随机 32 字节新密钥并清空现有令牌表。返回新密钥指纹。 |
| `list_tokens`| `KeyringCommandListTokens` | 列出所有令牌（已排序），返回 `[]KeyringToken`。 |
| `issue_token`| `KeyringCommandIssueToken` | 为指定 `subject` 签发新令牌。参数 `KeyringIssueTokenArgs{Subject string, TTL time.Duration, Algorithm string}`（`Algorithm` 可空，使用默认）。返回新建的 `KeyringToken`。 |
| `revoke_token`| `KeyringCommandRevokeToken` | 吊销指定 `subject` 的令牌。参数 `KeyringRevokeTokenArgs{Subject string}`，返回被吊销的 `KeyringToken`。 |
| `revoke_all` | `KeyringCommandRevokeAll` | 吊销所有未被吊销的令牌，返回被吊销的数量。 |

### 参数类型

```go
// set_secret 参数
type KeyringSetSecretArgs struct { Secret string }

// issue_token 参数
type KeyringIssueTokenArgs struct {
    Subject   string
    TTL       time.Duration // 可为 0，使用默认 TTL
    Algorithm string        // 可空，留空使用默认 HMAC-SHA256
}

// revoke_token 参数
type KeyringRevokeTokenArgs struct { Subject string }
```

### 返回值

所有命令返回 `types.CommandOutput`：

- `status`：`Value` 为 `KeyringStatus`（`Algorithm, SecretFinger, ActiveTokens, TotalIssued, LastRotatedAt`）。
- `set_secret`、`rotate`：`Value` 为新密钥的指纹 `string`。
- `list_tokens`：`Value` 为 `[]KeyringToken`（包含 `Subject, IssuedAt, ExpiresAt, Revoked`）。
- `issue_token`：`Value` 为新建的 `KeyringToken`（未签名的原始结构体）。
- `revoke_token`：`Value` 为已吊销的 `KeyringToken`。
- `revoke_all`：`Value` 为被吊销的令牌数量 `int`。

### 持久化/状态说明

- **密钥持久化**：`KeyringData` 可通过 `NewPersistentKeyringData(path)` 或 `SetSnapshotPath(path)` 启用磁盘快照。`Mount` 时会读取快照文件（JSON），若文件不存在则生成随机密钥。`Unmount`（或 Ability 在退出时）会调用 `SaveSnapshot` 将完整状态（密钥、令牌、元数据）写入 `path`，文件权限 `0600`，确保安全。
- **轮换语义**：`rotate` 与 `set_secret` 都会生成新密钥并清空令牌表 (`tokens = make(map[string]*KeyringToken)`)。因此旧令牌失效，`ActiveTokens` 变为 0，`TotalIssued` 计数保持累计历史。
- **快照结构**：内部 `keyringSnapshot` 包含 `Version, SavedAt, Secret (base64), LastRotatedAt, TotalIssued, RevokedCount, Tokens, DefaultAlgo, DefaultTokenTTL`。快照仅在持久化时写入磁盘，公开 API (`status`, `Snapshot`) 不会泄露原始密钥。

### 示例代码

```go
package main

import (
    "fmt"
    "time"

    "github.com/FasterEdge/FasterEdge/data"
)

func main() {
    // 使用持久化路径（可选）
    kr := data.NewPersistentKeyringData("/tmp/keyring_snapshot.json")
    // 挂载会加载已有快照或生成随机密钥
    if err := kr.Mount(nil); err != nil { panic(err) }

    // 查看当前状态
    out := kr.Command(nil, data.KeyringCommandStatus, nil)
    if out.Err != nil { panic(out.Err) }
    fmt.Printf("Status: %+v\n", out.Value)

    // 发行一个令牌，有效期 12 小时
    issueArgs := data.KeyringIssueTokenArgs{Subject: "peerA", TTL: 12 * time.Hour}
    out = kr.Command(nil, data.KeyringCommandIssueToken, issueArgs)
    if out.Err != nil { panic(out.Err) }
    token := out.Value.(data.KeyringToken)
    fmt.Printf("Issued token: %+v\n", token)

    // 轮换密钥（会清空令牌）
    out = kr.Command(nil, data.KeyringCommandRotate, nil)
    if out.Err != nil { panic(out.Err) }
    fmt.Println("New secret fingerprint:", out.Value)

    // 保存快照（通常在程序退出时调用）
    if err := kr.SaveSnapshot("/tmp/keyring_snapshot.json"); err != nil { panic(err) }
}
```
