# CmdAbility 文档

## 1. 简介

`CmdAbility` 提供受 *allowlist* 约束的系统命令执行能力，支持同步运行 (`run`)、异步启动 (`start`)、等待任务完成 (`wait`)、结束任务 (`kill`)、列出任务 (`list`)、获取任务信息 (`get_job`)、清理已完成任务 (`clear_finished`) 以及对可执行命令的白名单管理 (`set_allowlist` / `get_allowlist`)。它限制最大输出大小、最大运行超时以及并发数量，防止资源滥用。

## 2. 支持的命令

| 命令字符串 | Go 常量名 | 用途 |
|---|---|---|
| `run` | `CmdCommandRun` | 同步执行命令并返回 `CmdResult`。
| `start` | `CmdCommandStart` | 异步启动命令返回 job ID。
| `wait` | `CmdCommandWait` | 等待指定 job 完成并返回 `CmdResult`。
| `kill` | `CmdCommandKill` | 终止指定 job。
| `list` | `CmdCommandList` | 列出当前所有 job（`[]CmdJob`）。
| `get_job` | `CmdCommandGetJob` | 获取指定 job 的快照（`CmdJob`）。
| `clear_finished` | `CmdCommandClearJobs` | 清除已完成的 job，返回清除数量。
| `set_allowlist` | `CmdCommandSetAllowlist` | 设置命令白名单（`CmdSetAllowlistArgs`）。
| `get_allowlist` | `CmdCommandGetAllowlist` | 获取当前白名单快照（`[]CmdAllowlistEntry`）。

## 3. 参数类型

| 参数结构 | 字段 | 类型 | 必填 / 可选 |
|---|---|---|---|
| `CmdRunArgs` | `Name` | `string` | 必填，命令可执行文件名。
| | `Args` | `[]string` | 必填，命令参数。
| | `Timeout` | `time.Duration` | 可选，若 <=0 使用默认超时。
| | `Env` | `[]string` | 可选，仅会透传白名单环境变量。
| `CmdWaitArgs` | `JobID` | `string` | 必填。
| | `Wait` | `time.Duration` | 必填，等待时长（0 表示无限等待）。
| `CmdKillArgs` | `JobID` | `string` | 必填。
| `CmdJobIDArg` | `JobID` | `string` | 必填，用于 `get_job`、`clear_finished`（后者不需要字段）。
| `CmdSetAllowlistArgs` | `Entries` | `[]CmdAllowlistEntry` | 必填，白名单条目列表。
| `CmdAllowlistEntry` | `Name` | `string` | 必填，可执行文件名（精确匹配）。
| | `ArgsPrefix` | `[]string` | 可选，必须匹配的前置参数。
| | `MaxArgs` | `int` | 可选，0 表示不限制。

## 4. 返回值

- `CmdResult`：同步 `run`、`wait`、`kill`（成功）返回，包含 `JobID、Name、Args、ExitCode、Stdout、Stderr、Started、Duration、Truncated`。
- `string`（job ID）：`start` 成功返回。
- `[]CmdJob`：`list` 返回当前所有 job（已公开的视图）。
- `CmdJob`：`get_job` 返回指定 job 的快照。
- `int`：`clear_finished` 返回被清除的 job 数量。
- `[]CmdAllowlistEntry`：`get_allowlist` 返回当前白名单。
- 错误 `error`：任何异常情况均通过 `CommandOutput.Err` 返回。

## 5. 依赖与前置条件

- 依赖 `BaseData`（通过 `atom.Data("BaseData")` 检查）。
- 需要在 `atom` 中挂载 `BaseData` 实例后才能使用。

## 6. 可直接编译的 Go 示例

```go
package main

import (
    "time"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

func main() {
    // 创建 ability 实例
    cmd := ability.NewCmdAbility()

    // 示例 atom（这里使用 nil 仅示例，实际应提供 BaseData）
    var atom *types.Atom = nil // TODO: 初始化并挂载 BaseData

    // 同步运行 `echo hello`
    runArgs := ability.CmdRunArgs{
        Name:    "echo",
        Args:    []string{"hello"},
        Timeout: 5 * time.Second,
    }
    out := cmd.Command(atom, ability.CmdCommandRun, runArgs)
    if out.Err != nil {
        panic(out.Err)
    }
    result := out.Value.(ability.CmdResult)
    println("stdout:", result.Stdout)

    // 异步启动 `sleep 10`
    startArgs := ability.CmdRunArgs{Name: "sleep", Args: []string{"10"}}
    out = cmd.Command(atom, ability.CmdCommandStart, startArgs)
    jobID := out.Value.(string)
    println("started job", jobID)

    // 等待该 job 完成
    waitArgs := ability.CmdWaitArgs{JobID: jobID, Wait: 0}
    out = cmd.Command(atom, ability.CmdCommandWait, waitArgs)
    if out.Err == nil {
        res := out.Value.(ability.CmdResult)
        println("sleep exited with code", res.ExitCode)
    }
}
```
