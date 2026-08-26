# 跨能力组合示例

## 网络节点 + 令牌

```go
atom := fasteredge.InitStandardAtom()
if err := fasteredge.PreRunAtom(atom); err != nil { panic(err) }

nm, _ := atom.Ability("NetMapAbility")
nm.Command(atom, ability.NetMapCommandRegisterPeer, ability.NetMapRegisterPeerArgs{
    Name: "edge-2", Address: "192.0.2.20:7000", Role: "edge",
})

oneKey, _ := atom.Ability("OneKeyAbility")
out := oneKey.Command(atom, ability.OneKeyCommandIssueToken, ability.OneKeyIssueTokenArgs{
    Subject: "edge-2", TTL: time.Hour,
})
if out.Err != nil { panic(out.Err) }
```

## 算法分发

1. NetMapAbility 注册目标节点。
2. FileTransferAbility 注入传输实现。
3. AlgorithmDistributionAbility 注册算法名称、版本和源路径。
4. 调用 `distribute` 将指定版本发送到目标。
5. 在目标端用 DontCrack 托管算法进程。

## 完整 demo

FasterEdge 主仓库提供：

```bash
cd examples/demo
go run .
```

独立 Example 仓库当前仍是占位；见 [Example 状态](../Example/README.md)。
