# OneKeyAbility 文档

## 1. 简介

`OneKeyAbility` 提供节点加密访问（One-Key）能力：为对等节点签发短期 HMAC-SHA256 令牌，并支持校验、吊销与密钥轮换。它建立在 `KeyringData` 共享密钥之上——本节点签发令牌，远端节点可通过 `verify_token` 验证持有者身份，实现对称的"一键加密访问"。

## 2. 支持的命令

| 命令字符串 | 常量名称 | 用途 |
|-----------|----------|------|
| `issue_token` | `OneKeyCommandIssueToken` | 为一个 Subject 签发短期令牌 |
| `verify_token` | `OneKeyCommandVerifyToken` | 校验令牌签名、有效期与吊销状态 |
| `revoke_token` | `OneKeyCommandRevokeToken` | 吊销指定 Subject 的令牌 |
| `revoke_all` | `OneKeyCommandRevokeAll` | 吊销全部令牌 |
| `list_tokens` | `OneKeyCommandListTokens` | 列出所有令牌记录 |
| `status` | `OneKeyCommandStatus` | 查看 KeyringData 状态快照 |
| `rotate` | `OneKeyCommandRotate` | 轮换共享密钥（转发给 KeyringData 的 `rotate` 命令） |

## 3. 参数类型

```go
// issue_token
type OneKeyIssueTokenArgs struct {
    Subject   string        // 必填，令牌主体（对等节点名）
    TTL       time.Duration // 可选，0 表示使用 KeyringData 默认值；负数报错
    Algorithm string        // 可选，留空使用默认，仅支持 "HMAC-SHA256"
}

// verify_token —— 需要把令牌字段原样回传
type OneKeyVerifyTokenArgs struct {
    Subject   string
    IssuedAt  time.Time
    ExpiresAt time.Time
    Signature string // base64(HMAC-SHA256)
}

// revoke_token
type OneKeyRevokeTokenArgs struct {
    Subject string
}
```

## 4. 返回值

| 命令 | `CommandOutput.Value` |
|------|----------------------|
| `issue_token` | `OneKeyToken{Subject, IssuedAt, ExpiresAt, Signature}` |
| `verify_token` | `string`（校验通过的 Subject） |
| `revoke_token` | `string`（被吊销令牌的 Subject） |
| `revoke_all` | KeyringData `revoke_all` 的返回值 |
| `list_tokens` | KeyringData 的令牌记录列表 |
| `status` | KeyringData 状态快照 |
| `rotate` | KeyringData `rotate` 的返回值 |

```go
type OneKeyToken struct {
    Subject   string
    IssuedAt  time.Time
    ExpiresAt time.Time
    Signature string // base64(HMAC-SHA256)
}
```

## 5. 依赖与前置条件

- 依赖 `BaseData`、`NetMapData` 与 `KeyringData`，三者缺一则返回 `types.ErrMissingDependency`。
- 轮换共享密钥会影响后续签发与验证语义：已签发的旧令牌在新密钥下会被判定签名无效，调用方应规划兼容窗口。
- 传输工具函数不依赖哨兵错误，可直接使用：

```go
// 打包为 "subject.issuedNanos.expiresNanos.signature"
s := ability.EncodeForTransmission(tok)
// 解析回 OneKeyToken
tok2, err := ability.DecodeFromTransmission(s)
```

## 6. 可直接编译的示例

```go
package main

import (
    "fmt"
    "time"

    "github.com/FasterEdge/FasterEdge/ability"
    fasteredge "github.com/FasterEdge/FasterEdge"
)

func main() {
    atom := fasteredge.InitStandardAtom()
    if err := fasteredge.PreRunAtom(atom); err != nil {
        panic(err)
    }

    ok, ok2 := atom.Ability("OneKeyAbility")
    _ = ok2
    if !ok2 {
        panic("OneKeyAbility missing")
    }

    // 1. 为 edge-2 签发有效期 1 小时的令牌
    out := ok.Command(atom, ability.OneKeyCommandIssueToken, ability.OneKeyIssueTokenArgs{
        Subject: "edge-2", TTL: time.Hour,
    })
    if out.Err != nil {
        panic(out.Err)
    }
    tok := out.Value.(ability.OneKeyToken)
    fmt.Printf("token: subject=%s sig=%s\n", tok.Subject, tok.Signature)

    // 2. 打包传输
    wire := ability.EncodeForTransmission(tok)
    fmt.Println("wire:", wire)

    // 3. 远端解析并校验
    tok2, err := ability.DecodeFromTransmission(wire)
    if err != nil {
        panic(err)
    }
    out = ok.Command(atom, ability.OneKeyCommandVerifyToken, ability.OneKeyVerifyTokenArgs{
        Subject:   tok2.Subject,
        IssuedAt:  tok2.IssuedAt,
        ExpiresAt: tok2.ExpiresAt,
        Signature: tok2.Signature,
    })
    if out.Err != nil {
        panic(out.Err)
    }
    fmt.Printf("verified subject: %v\n", out.Value)
}
```