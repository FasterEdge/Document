# BashAbility 文档

## 1. 简介

`BashAbility` 是 **ShAbility** 的特化实现。它把用户提供的单条命令包装为 `bash --noprofile --norc -c <cmd>`，然后交由内部的 **ShAbility** 进行实际执行。通过这种方式，既复用了 ShAbility 的任务管理、并发控制和 allowlist 机制，又确保使用 **bash** 语义（不读取用户登录配置文件），从而提供更可预测的执行环境。

## 2. 支持的命令

| 命令字符串 | Go 常量名 | 用途 |
|---|---|---|
| `run` | `BashCommandRun` | 同步执行 `bash --noprofile --norc -c <cmd>`，返回 `CmdResult`（由底层 ShAbility 产生）。
| `start` | `BashCommandStart` | 异步启动同上命令，返回 job ID（`string`）。
| `wait` | `BashCommandWait` | 等待指定 job 完成，返回 `CmdResult`（转发自 ShAbility）。
| `kill` | `BashCommandKill` | 终止指定 job，返回 `bool`（转发自 ShAbility）。
| `list` | `BashCommandList` | 列出当前所有 job，返回 `[]CmdJob`（转发自 ShAbility）。
| `set_allowlist` | `BashCommandSetAllowlist` | 设置 **BashAbility** 自身的子命令白名单（`ShSetAllowlistArgs`），返回当前白名单快照。
| `get_allowlist` | `BashCommandGetAllowlist` | 获取当前白名单快照，返回 `ShAllowlist`（只包含子命令前缀）。

## 3. 参数类型

| 参数结构 | 字段 | 类型 | 必填 / 可选 |
|---|---|---|---|
| `ShRunArgs`（复用自 ShAbility） | `Command` | `string` | 必填，要执行的原始命令字符串（在内部会包装为 bash）。
| | `Timeout` | `time.Duration` | 可选，若 ≤0 使用默认超时。
| | `Env` | `[]string` | 可选，环境变量透传。
| `ShSetAllowlistArgs`（复用自 ShAbility） | `Allowed` | `[]string` | 必填，允许的子命令前缀列表（空切片表示不限制）。
| `ShAllowlist`（复用自 ShAbility） | `Shell` | `string` | 固定为 `bash`（由实现硬编码）。
| | `Allowed` | `[]string` | 当前子命令白名单。

> **注意**：`BashAbility` 的 allowlist 只检查 **用户原始命令的首词**，而底层的 ShAbility 只允许 `bash` 本身。两层检查确保不会因包装导致的误判。

## 4. 返回值

- `CmdResult`：`run`、`wait`、`kill`（成功）返回，包含 job 信息、退出码、输出等。
- `string`（job ID）：`start` 成功返回。
- `[]CmdJob`：`list` 返回当前所有 job（只读视图）。
- `ShAllowlist`：`set_allowlist` / `get_allowlist` 返回当前子命令白名单快照。
- 错误 `error`：通过 `CommandOutput.Err` 传播。

## 5. 依赖与前置条件

- **ShAbility**：`BashAbility` 内部维护一个 `*ShAbility` 实例，用它完成实际的命令包装与执行。
- **BaseData**：通过 `atom.Data("BaseData")` 检查，必须挂载。
- **Allowlist**：`BashAbility` 通过 `SetAllowed` 管理自己的子命令白名单；底层 ShAbility 的 allowlist 永远固定为 `bash`（由 `syncUnderlyingAllowlist` 注入）。

## 6. 可直接编译的 Go 示例

```go
package main

import (
    "time"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

func main() {
    // 创建 BashAbility（内部创建 ShAbility）
    bash := ability.NewBashAbility()

    // 示例 atom（实际需要提供 BaseData）
    var atom *types.Atom = nil // TODO: 初始化并挂载 BaseData

    // 同步执行 `echo hello`（使用 bash）
    runArgs := ability.ShRunArgs{Command: "echo hello", Timeout: 2 * time.Second}
    out := bash.Command(atom, ability.BashCommandRun, runArgs)
    if out.Err != nil {
        panic(out.Err)
    }
    result := out.Value.(ability.CmdResult)
    println("stdout:", result.Stdout)

    // 异步启动一个长任务 `sleep 3`
    startArgs := ability.ShRunArgs{Command: "sleep 3"}
    out = bash.Command(atom, ability.BashCommandStart, startArgs)
    jobID := out.Value.(string)
    println("started job", jobID)

    // 等待任务完成
    waitArgs := ability.ShWaitArgs{JobID: jobID, Wait: 0}
    out = bash.Command(atom, ability.BashCommandWait, waitArgs)
    if out.Err == nil {
        res := out.Value.(ability.CmdResult)
        println("sleep exited with code", res.ExitCode)
    }
}
```
