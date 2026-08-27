# DockerAbility

## 简介
`DockerAbility` 为 FasterEdge 提供容器生命周期管理能力。通过抽象 `DockerTransport` 桥接真实 Docker daemon，它支持容器的列出、启动、停止、重启、删除、检查、日志获取、镜像拉取以及容器创建等操作。所有底层 API 交互均通过注入的 Transport 完成。

## 支持的命令
| 命令字符串 | Go 常量名 | 目的 |
|---|---|---|
| `set_endpoint` | `DockerCommandSetEndpoint` | 设置 Docker daemon 端点 |
| `get_endpoint` | `DockerCommandGetEndpoint` | 获取当前端点 |
| `list_containers` | `DockerCommandListContainers` | 列出容器 |
| `start_container` | `DockerCommandStart` | 启动容器 |
| `stop_container` | `DockerCommandStop` | 停止容器 |
| `restart_container` | `DockerCommandRestart` | 重启容器 |
| `remove_container` | `DockerCommandRemove` | 删除容器 |
| `pull_image` | `DockerCommandPullImage` | 拉取镜像 |
| `inspect_container` | `DockerCommandInspect` | 检查容器详情 |
| `get_logs` | `DockerCommandGetLogs` | 获取容器日志 |
| `create_container` | `DockerCommandCreate` | 创建容器 |

## 参数类型
```go
// set_endpoint
type DockerEndpointArgs struct {
    URL string // unix:///var/run/docker.sock 或 tcp://host:2375
}

// start / stop / restart / remove / inspect / get_logs 的通用参数
type DockerContainerArgs struct {
    IDOrName string        // 容器 ID 或名称，必填非空
    Timeout  time.Duration // 用于 stop/restart 的等待超时
}

// pull_image
type DockerPullImageArgs struct {
    Reference string // 镜像引用，如 nginx:latest
}

// create_container
type DockerCreateArgs struct {
    Name    string   // 容器名称
    Image   string   // 镜像，必填
    Command []string // 启动命令
    Env     []string // 环境变量
    Ports   []string // 端口映射
}
```

## 返回值
`Command` 返回 `types.CommandOutput`，`Value` 根据不同命令可能为：
- `string`（endpoint、容器 ID/名称、镜像引用、日志内容）
- `[]DockerContainer`（`list_containers` 返回容器快照切片）
- `DockerContainer`（`inspect_container` 返回单个容器快照）
- `bool`（`list_containers` 的 `All` 标记仅在入参时使用）
- `nil`（仅表示执行成功）

## 依赖与前置条件
- 必须调用 `SetTransport(t DockerTransport)` 注入实现 `List、Start、Stop、Restart、Remove、Pull、Inspect、Logs、Create` 的 Transport。
- 使用任何命令前需先 `set_endpoint` 指定有效端点（`unix://` 或 `tcp://`、`tls://`，并拒绝本地回环/私网地址）。
- Docker daemon 需处于运行状态。

## 示例代码
```go
package main

import (
    "time"
    "github.com/FasterEdge/FasterEdge/ability"
    "github.com/FasterEdge/FasterEdge/types"
)

// mockTransport 仅用于演示接口。
 type mockTransport struct{}
 func (m *mockTransport) List(all bool) ([]ability.DockerContainer, error) { return nil, nil }
 func (m *mockTransport) Start(idOrName string) error { return nil }
 func (m *mockTransport) Stop(idOrName string, timeout time.Duration) error { return nil }
 func (m *mockTransport) Restart(idOrName string, timeout time.Duration) error { return nil }
 func (m *mockTransport) Remove(idOrName string, force bool) error { return nil }
 func (m *mockTransport) Pull(reference string) error { return nil }
 func (m *mockTransport) Inspect(idOrName string) (ability.DockerContainer, error) { return ability.DockerContainer{}, nil }
 func (m *mockTransport) Logs(idOrName string, tail int) (string, error) { return "", nil }
 func (m *mockTransport) Create(args ability.DockerCreateArgs) (string, error) { return "container_id", nil }

func main() {
    docker := ability.NewDockerAbility()
    docker.SetTransport(&mockTransport{})

    // 设置端点并拉取镜像
    docker.Command(&types.Atom{}, ability.DockerCommandSetEndpoint, ability.DockerEndpointArgs{URL: "unix:///var/run/docker.sock"})
    docker.Command(&types.Atom{}, ability.DockerCommandPullImage, ability.DockerPullImageArgs{Reference: "nginx:latest"})

    // 创建并启动容器
    createOut := docker.Command(&types.Atom{}, ability.DockerCommandCreate, ability.DockerCreateArgs{Name: "web", Image: "nginx:latest", Ports: []string{"80:80"}})
    containerID := createOut.Value.(string)
    docker.Command(&types.Atom{}, ability.DockerCommandStart, ability.DockerContainerArgs{IDOrName: containerID})

    // 查看容器列表与日志
    docker.Command(&types.Atom{}, ability.DockerCommandListContainers, nil)
    logsOut := docker.Command(&types.Atom{}, ability.DockerCommandGetLogs, ability.DockerContainerArgs{IDOrName: containerID})
    println(logsOut.Value.(string))
}
```
