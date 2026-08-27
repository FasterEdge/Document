# RoleAbility

## 简介
RoleAbility 提供角色管理能力，在 Atom 上维护一个简单的角色字符串（如 `"cloud"`、`"edge"`），并可通过 `describe`、`set_role`、`get_role` 三个命令查看与修改。它是角色类能力的基础，供 CloudRoleAbility、EdgeRoleAbility 等扩展能力派生。

## 支持的命令
| 常量 | 命令字符串 | 说明 |
|------|-------------|------|
| `ability.CommandDescribe` | `"describe"` | 返回能力描述文本（`Describe()` 返回值）。
| `ability.CommandSetRole` | `"set_role"` | 设置当前角色值，参数为 `RoleAbilityArgs`，`Role` 字段去除首尾空白后非空才生效。
| `ability.CommandGetRole` | `"get_role"` | 获取当前角色值（未设置时返回空字符串）。

## 参数类型
```go
type RoleAbilityArgs struct {
    Role string // 必填，去除首尾空白后必须非空
}
```
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `Role` | `string` | 是 | 要设置的角色名称，`strings.TrimSpace` 后不能为空。 |

`describe` 与 `get_role` 不接受参数，`args` 必须为 `nil`。

## 返回值
`CommandOutput.Value` 的内容：
- `describe`：`string`，能力描述文本。
- `set_role`：`string`，固定返回 `"角色设置成功"`。
- `get_role`：`string`，当前角色值；未设置过时为空字符串 `""`。

## 依赖与前置条件
- 挂载（`Mount`）前必须已注册 `BaseData`，否则 `Check` 返回 `types.ErrMissingDependency`。
- `set_role` 的参数必须是 `RoleAbilityArgs` 类型（强类型传入），否则返回 `types.ErrInvalidArguments`。

## 示例代码
```go
// 假设已有 *types.Atom 实例 atom
ab, _ := atom.Ability("RoleAbility")

// 设置角色
setOut := ab.Command(atom, ability.CommandSetRole, ability.RoleAbilityArgs{Role: "cloud"})
if setOut.Err != nil {
    // 处理错误
}
fmt.Println(setOut.Value.(string)) // 输出：角色设置成功

// 获取角色
getOut := ab.Command(atom, ability.CommandGetRole, nil)
if getOut.Err != nil {
    // 处理错误
}
role := getOut.Value.(string)
fmt.Println("role:", role) // 输出：role: cloud

// 描述
descOut := ab.Command(atom, ability.CommandDescribe, nil)
if descOut.Err != nil {
    // 处理错误
}
fmt.Println(descOut.Value.(string)) // 输出：提供角色管理的能力。
```