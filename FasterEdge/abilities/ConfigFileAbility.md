# ConfigFileAbility 文档

## 1. 简介

`ConfigFileAbility` 在 **ConfigData** 之上提供 JSON 配置文件的加载与保存能力。它支持扁平的点号键（如 `"a.b": "v"`）、路径管理（`set_path` / `get_path`）、文件存在性检查（`exists`）以及带严格模式（`Strict`）和覆盖模式（`Overwrite`）的读写。

## 2. 支持的命令

| 命令字符串 | Go 常量名 | 用途 |
|---|---|---|
| `load` | `ConfigFileCommandLoad` | 从文件加载 JSON 到 ConfigData，返回配置快照。
| `save` | `ConfigFileCommandSave` | 将 ConfigData 保存为 JSON 文件，返回保存路径。
| `set_path` | `ConfigFileCommandSetPath` | 设置默认配置路径，返回清理后的路径。
| `get_path` | `ConfigFileCommandGetPath` | 获取当前配置路径。
| `exists` | `ConfigFileCommandExists` | 检查指定文件是否存在，返回 `bool`。

## 3. 参数类型

| 参数结构 | 字段 | 类型 | 必填 / 可选 |
|---|---|---|---|
| `ConfigFileLoadArgs` | `Path` | `string` | 必填，要加载的 JSON 文件路径。
| | `Strict` | `bool` | 可选，严格模式下文件不存在或解析失败返回错误；否则静默返回当前快照。
| `ConfigFileSaveArgs` | `Path` | `string` | 可选，保存路径，若为空使用已设置的默认路径。
| | `Overwrite` | `bool` | 可选，为 true 时覆盖已有文件；为 false 时文件已存在会返回错误。
| `ConfigFilePathArg` | `Path` | `string` | 必填，用于 `set_path` 与 `exists` 的文件路径（`get_path` 不需要参数）。

## 4. 返回值

- `map[string]string`（配置快照）：`load`、`get_allowlist` 无；`load` 成功返回 ConfigData 的快照。
- `string`（路径）：`save`、`set_path` 返回清理后的路径；`get_path` 返回当前路径。
- `bool`：`exists` 返回文件是否存在。
- 错误 `error`：通过 `CommandOutput.Err` 传播（如文件不存在、解析失败、写入失败等）。

## 5. 依赖与前置条件

- 依赖 **ConfigData**（通过 `atom.Data("ConfigData")` 获取），必须挂载 `*data.ConfigData` 实例。
- 依赖 **BaseData**（通过 `atom.Data("BaseData")` 检查）。
- 通过 `Dependencies()` 声明 `BaseData` 与 `ConfigData` 两个数据依赖。
- 加载的 JSON 必须是扁平结构（键为点号路径，值为字符串），不支持嵌套对象。

## 6. 可直接编译的 Go 示例

```go
package main

import (
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/data"
    "github.com/FasterEdge/FasterEdge/types"
)

func main() {
    // 创建 ConfigFileAbility
    cfgFile := ability.NewConfigFileAbility()

    // 构造 atom 并挂载 BaseData 与 ConfigData
    atom := types.NewAtom()
    _ = atom.SetData("BaseData", &data.BaseData{})  // 示例，按实际类型初始化
    cfg := data.NewConfigData()
    _ = atom.SetData("ConfigData", cfg)

    // 设置保存路径
    setPath := ability.ConfigFilePathArg{Path: "/tmp/fe_config.json"}
    out := cfgFile.Command(atom, ability.ConfigFileCommandSetPath, setPath)
    if out.Err != nil {
        panic(out.Err)
    }
    println("path set to", out.Value.(string))

    // 写入一些配置
    cfg.Set("app.name", "fasteredge")
    cfg.Set("app.debug", "true")

    // 保存配置到文件
    saveArgs := ability.ConfigFileSaveArgs{Path: "", Overwrite: true}
    out = cfgFile.Command(atom, ability.ConfigFileCommandSave, saveArgs)
    if out.Err != nil {
        panic(out.Err)
    }
    println("saved to", out.Value.(string))

    // 加载配置
    loadArgs := ability.ConfigFileLoadArgs{Path: "/tmp/fe_config.json", Strict: true}
    out = cfgFile.Command(atom, ability.ConfigFileCommandLoad, loadArgs)
    if out.Err != nil {
        panic(out.Err)
    }
    snapshot := out.Value.(map[string]string)
    println("app.name =", snapshot["app.name"])
}
```
