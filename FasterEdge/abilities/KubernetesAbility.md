# KubernetesAbility

## 简介
`KubernetesAbility` 为 FasterEdge 提供 Kubernetes 资源管理能力。通过抽象 `K8sTransport`（可基于 client-go 或 REST mapper 实现）桥接真实 K8s API，它支持资源清单的 apply/delete、资源的 list/get、Deployment 的 scale，以及 Pod 日志获取等操作，并能维护当前集群上下文。

## 支持的命令
| 命令字符串 | Go 常量名 | 目的 |
|---|---|---|
| `set_context` | `K8sCommandSetContext` | 设置集群上下文 |
| `get_context` | `K8sCommandGetContext` | 获取当前上下文 |
| `apply` | `K8sCommandApply` | 应用资源清单（YAML/JSON） |
| `delete` | `K8sCommandDelete` | 删除资源 |
| `list` | `K8sCommandList` | 列出指定 Kind 的资源 |
| `get` | `K8sCommandGet` | 获取单个资源详情 |
| `scale` | `K8sCommandScale` | 伸缩 Deployment 副本数 |
| `get_logs` | `K8sCommandGetLogs` | 获取 Pod 日志 |

## 参数类型
```go
// set_context
type K8sContext struct {
    Cluster    string
    Namespace  string // 可留空，缺省补为 "default"
    Kubeconfig string // 可选，指定 kubeconfig 路径
}

type K8sContextArgs struct {
    K8sContext // 内嵌，Cluster 必填非空
}

// apply
type K8sApplyArgs struct {
    Manifest string // YAML/JSON 资源清单，必填非空
}

// delete
type K8sDeleteArgs struct {
    Kind      string // 资源类型，需合法
    Name      string // 资源名，必填非空
    Namespace string // 可留空，回退到上下文 Namespace
}

// list
type K8sListArgs struct {
    Kind      string // 资源类型，需合法
    Namespace string // 可留空，回退到上下文 Namespace
}

// get
type K8sGetArgs struct {
    Kind      string // 资源类型，需合法
    Name      string // 资源名，必填非空
    Namespace string // 可留空，回退到上下文 Namespace
}

// scale
type K8sScaleArgs struct {
    Deployment string        // Deployment 名，必填非空
    Namespace  string        // 可留空，回退到上下文 Namespace
    Replicas   int32         // 目标副本数，>= 0
}

// get_logs
type K8sLogsArgs struct {
    Pod       string // Pod 名，必填非空
    Namespace string // 可留空，回退到上下文 Namespace
    Tail      int    // 末尾行数，>= 0
}
```

## 返回值
`Command` 返回 `types.CommandOutput`，`Value` 根据不同命令可能为：
- `K8sContext`（`set_context` / `get_context` 返回完整上下文结构体）
- `bool`（`apply` 成功标记）
- `string`（`delete` 返回资源名，`scale` 返回副本数，`get_logs` 返回日志文本）
- `[]K8sResource`（`list` 返回资源快照切片，按名称排序）
- `K8sResource`（`get` 返回单个资源快照）
- `nil`（仅表示执行成功）

## 依赖与前置条件
- 必须调用 `SetTransport(t K8sTransport)` 注入实现 `Apply、Delete、List、Get、Scale、Logs` 的 Transport。
- 使用任何命令前需先 `set_context` 指定 Cluster（`set_context` 要求 `Cluster` 非空，`Namespace` 留空时默认补为 `default`）。
- 提供了 `currentNamespace()` 回退逻辑：在各操作中若未指定 Namespace，将自动使用上下文中的 Namespace，上下文无值时回退到 `"default"`。

## 示例代码
```go
package main

import (
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

// mockTransport 仅用于演示接口。
 type mockTransport struct{}
 func (m *mockTransport) Apply(manifest string) error { return nil }
 func (m *mockTransport) Delete(kind, name, namespace string) error { return nil }
 func (m *mockTransport) List(kind, namespace string) ([]ability.K8sResource, error) { return nil, nil }
 func (m *mockTransport) Get(kind, name, namespace string) (ability.K8sResource, error) { return ability.K8sResource{}, nil }
 func (m *mockTransport) Scale(deployment, namespace string, replicas int32) error { return nil }
 func (m *mockTransport) Logs(pod, namespace string, tail int) (string, error) { return "log", nil }

func main() {
    k8s := ability.NewK8sAbility()
    k8s.SetTransport(&mockTransport{})

    // 设置上下文（Namespace 缺省为 default）
    k8s.Command(&types.Atom{}, ability.K8sCommandSetContext, ability.K8sContextArgs{
        K8sContext: ability.K8sContext{Cluster: "prod-cluster", Namespace: "prod"},
    })

    // apply 资源清单
    k8s.Command(&types.Atom{}, ability.K8sCommandApply, ability.K8sApplyArgs{
        Manifest: "apiVersion: apps/v1\nkind: Deployment\nmetadata:\n  name: web\nspec:\n  replicas: 1\n",
    })

    // 列出 Pod
    k8s.Command(&types.Atom{}, ability.K8sCommandList, ability.K8sListArgs{Kind: "pod"})

    // 获取日志并缩放
    k8s.Command(&types.Atom{}, ability.K8sCommandGetLogs, ability.K8sLogsArgs{Pod: "web-0", Tail: 50})
    scaleOut := k8s.Command(&types.Atom{}, ability.K8sCommandScale, ability.K8sScaleArgs{Deployment: "web", Replicas: 3})
    println("replicas:", scaleOut.Value.(int32))
}
```