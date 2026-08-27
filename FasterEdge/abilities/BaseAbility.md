# BaseAbility

## 简介
BaseAbility 是 FasterEdge 框架提供的基础能力，用于在 Atom 上列举已挂载的 Data 与 Ability 名称。它在挂载前会检查 `BaseData` 是否已存在，确保框架的基础数据已准备就绪。

## Overview (English)
`BaseAbility` is a fundamental ability in the FasterEdge framework that lists the names of all mounted Data and Ability components on an Atom. It checks for the presence of `BaseData` before mounting to ensure core data is initialized.

## 支持的命令
| 常量 | 命令字符串 | 说明 |
|------|-------------|------|
| `ability.CommandListDataNames` | `"list_data_names"` | 列举当前 Atom 中已挂载的所有 Data 名称，返回 `[]string`。
| `ability.CommandListAbilityNames` | `"list_ability_names"` | 列举当前 Atom 中已挂载的所有 Ability 名称，返回 `[]string`。

## Commands
| Constant | Command String | Description |
|----------|----------------|-------------|
| `ability.CommandListDataNames` | `"list_data_names"` | Lists all Data names mounted on the current Atom, returns `[]string`.
| `ability.CommandListAbilityNames` | `"list_ability_names"` | Lists all Ability names mounted on the current Atom, returns `[]string`.

## 参数类型
BaseAbility 的所有命令均不接受参数，`args` 必须为 `nil`。如果传入非 nil 参数，会返回 `types.ErrInvalidArguments`。

## Parameter Types
All commands of `BaseAbility` accept no arguments; the `args` argument must be `nil`. Supplying a non‑nil value returns `types.ErrInvalidArguments`.

## 返回值
`CommandOutput.Value` 为 `[]string`，分别对应 Data 名称列表或 Ability 名称列表。

## Return Value
`CommandOutput.Value` is an `[]string` containing either the list of Data names or Ability names.

## 依赖与前置条件
- 挂载前必须已注册 `BaseData`，否则 `Check` 会返回 `types.ErrMissingDependency`。

## Dependencies & Preconditions
- `BaseData` must be registered before mounting; otherwise `Check` returns `types.ErrMissingDependency`.

## 示例代码
```go
// 假设已有 *types.Atom 实例 atom
ab, _ := atom.Ability("BaseAbility")
// 列举 Data 名称
out := ab.Command(atom, ability.CommandListDataNames, nil)
if out.Err != nil {
    // 处理错误
}
names := out.Value.([]string)
fmt.Println("Data names:", names)

// 列举 Ability 名称
out2 := ab.Command(atom, ability.CommandListAbilityNames, nil)
if out2.Err != nil {
    // 处理错误
}
abilityNames := out2.Value.([]string)
fmt.Println("Ability names:", abilityNames)
```

## Example (English)
```go
// Assume you have an *types.Atom instance called atom
ab, _ := atom.Ability("BaseAbility")
// List Data names
out := ab.Command(atom, ability.CommandListDataNames, nil)
if out.Err != nil {
    // handle error
}
names := out.Value.([]string)
fmt.Println("Data names:", names)

// List Ability names
out2 := ab.Command(atom, ability.CommandListAbilityNames, nil)
if out2.Err != nil {
    // handle error
}
abilityNames := out2.Value.([]string)
fmt.Println("Ability names:", abilityNames)
```


## 简介
BaseAbility 是 FasterEdge 框架提供的基础能力，用于在 Atom 上列举已挂载的 Data 与 Ability 名称。它在挂载前会检查 `BaseData` 是否已存在，确保框架的基础数据已准备就绪。

## 支持的命令
| 常量 | 命令字符串 | 说明 |
|------|-------------|------|
| `ability.CommandListDataNames` | `"list_data_names"` | 列举当前 Atom 中已挂载的所有 Data 名称，返回 `[]string`。
| `ability.CommandListAbilityNames` | `"list_ability_names"` | 列举当前 Atom 中已挂载的所有 Ability 名称，返回 `[]string`。

## 参数类型
BaseAbility 的所有命令均不接受参数，`args` 必须为 `nil`。如果传入非 nil 参数，会返回 `types.ErrInvalidArguments`。

## 返回值
`CommandOutput.Value` 为 `[]string`，分别对应 Data 名称列表或 Ability 名称列表。

## 依赖与前置条件
- 挂载前必须已注册 `BaseData`，否则 `Check` 会返回 `types.ErrMissingDependency`。

## 示例代码
```go
// 假设已有 *types.Atom 实例 atom
ab, _ := atom.Ability("BaseAbility")
// 列举 Data 名称
out := ab.Command(atom, ability.CommandListDataNames, nil)
if out.Err != nil {
    // 处理错误
}
names := out.Value.([]string)
fmt.Println("Data names:", names)

// 列举 Ability 名称
out2 := ab.Command(atom, ability.CommandListAbilityNames, nil)
if out2.Err != nil {
    // 处理错误
}
abilityNames := out2.Value.([]string)
fmt.Println("Ability names:", abilityNames)
```