# ShAbility 文档

## 1. 简介

`ShAbility` 是在 **CmdAbility** 之上提供的高级抽象，负责把以单字符串形式给出的命令包装为 `sh -c <command>` 并交由底层的 `CmdAbility` 执行。它复用了 CmdAbility 的任务管理、allowlist、并发限制等机制，并提供了对 **shell 名称**（如 `sh`、`bash`）的可配置化，使得跨平台或自定义 shell 成为可能。

## 2. 支持的命令

| 命令字符串 | Go 常量名 | 用途 |
|---|---|---|
| `run` | `ShCommandRun` | 同步执行 `sh -c <command>`，返回 `CmdResult`。
| `start` | `ShCommandStart` | 异步启动 `sh -c <command>`，返回 job ID（`string`）。
| `wait` | `ShCommandWait` | 等待指定 job 完成，返回 `CmdResult`。
| `kill` | `ShCommandKill` | 终止指定 job，返回 `bool`（是否成功）。
| `list` | `ShCommandList` | 列出当前所有 job，返回 `[]CmdJob`。
| `set_allowlist` | `ShCommandSetAllowlist` | 设置子命令白名单（`ShSetAllowlistArgs`），返回 `ShAllowlist` 快照。
| `get_allowlist` | `ShCommandGetAllowlist` | 获取当前白名单快照，返回 `ShAllowlist`。

## 3. 参数类型

| 参数结构 | 字段 | 类型 | 必填 / 可选 |
|---|---|---|---|
| `ShRunArgs` | `Command` | `string` | 必填，要执行的完整命令字符串。
| | `Timeout` | `time.Duration` | 可选，若 ≤0 使用默认超时。
| | `Env` | `[]string` | 可选，仅透传白名单环境变量。
| `ShWaitArgs` | `JobID` | `string` | 必填。
| | `Wait` | `time.Duration` | 必填，等待时长（0 表示无限阻塞）。
| `ShKillArgs` | `JobID` | `string` | 必填。
| `ShSetAllowlistArgs` | `Allowed` | `[]string` | 必填，允许执行的子命令前缀列表（空切片表示不限制）。
| `ShAllowlist` | `Shell` | `string` | 返回当前使用的 shell 名称（默认 `sh`）。
| | `Allowed` | `[]string` | 当前子命令白名单。

## 4. 返回值

- `CmdResult`：`run`、`wait` 成功时返回，包含 job 信息、退出码、输出等。
- `string`（job ID）：`start` 成功返回。
- `bool`：`kill` 成功返回 true，若 job 已完成返回 false。
- `[]CmdJob`：`list` 返回当前所有 job（公开视图）。
- `ShAllowlist`：`set_allowlist` / `get_allowlist` 返回当前子命令白名单快照。
- 错误 `error`：通过 `CommandOutput.Err` 传播。

## 5. 依赖与前置条件

- 依赖 **BaseData**（通过 `atom.Data("BaseData")` 检查），必须在 `atom` 中挂载。
- 内部持有一个 **CmdAbility** 实例，所有实际的进程启动、输出捕获、并发控制均委派给它。
- 通过 `SetShell(name string)` 可以自定义底层 shell（例如使用 `bash`、`zsh`），默认 `sh`。

## 6. 可直接编译的 Go 示例

```go
package main

import (
    "time"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

func main() {
    // 创建 ShAbility（内部会创建一个 CmdAbility）
    sh := ability.NewShAbility()

    // 示例 atom（实际使用时需要提供 BaseData）
    var atom *types.Atom = nil // TODO: 初始化并挂载 BaseData

    // 同步运行 `echo fastedge`
    runArgs := ability.ShRunArgs{Command: "echo fastedge", Timeout: 3 * time.Second}
    out := sh.Command(atom, ability.ShCommandRun, runArgs)
    if out.Err != nil {
        panic(out.Err)
    }
    result := out.Value.(ability.CmdResult)
    println("stdout:", result.Stdout)

    // 异步启动一个长时间任务 `sleep 5`
    startArgs := ability.ShRunArgs{Command: "sleep 5"}
    out = sh.Command(atom, ability.ShCommandStart, startArgs)
    jobID := out.Value.(string)
    println("started job", jobID)

    // 等待该任务完成
    waitArgs := ability.ShWaitArgs{JobID: jobID, Wait: 0}
    out = sh.Command(atom, ability.ShCommandWait, waitArgs)
    if out.Err == nil {
        res := out.Value.(ability.CmdResult)
        println("sleep exited with code", res.ExitCode)
    }
}
```
