# DontCrack for Windows

- 源码：https://github.com/FasterEdge/DontCrack4Windows
- 二进制格式：PE/MZ
- shell：`cmd.exe`，PowerShell 脚本单独处理
- 脚本：`.bat`、`.cmd`、`.ps1`
- 默认日志：`logs\proc_manager\`

## 构建

```powershell
$env:CGO_ENABLED = "0"
$env:GOOS = "windows"
$env:GOARCH = "amd64"
go build -ldflags='-s -w' -o DontCrack.exe .
```

## 脚本执行

- `.bat/.cmd`：`cmd.exe /C`。
- `.ps1`：`powershell.exe -NoProfile -ExecutionPolicy Bypass -File`。
- `-pre`：`cmd.exe /C`。

## 停止

Windows 没有 POSIX SIGTERM。实现先尝试 `os.Interrupt`，超时后 `Process.Kill()`。不是所有服务/控制台组合都能接收 CTRL 事件，因此业务进程必须测试真实停止行为。

## 服务化

### 计划任务

仓库 `example/install.bat` 可注册 SYSTEM 身份的开机任务。需要管理员终端。

### NSSM

```cmd
nssm install DontCrack "C:\ProgramData\DontCrack\bin\DontCrack.exe" ^
  "-path C:\ProgramData\DontCrack\bin\service.exe -start-now -auto-restart"
nssm start DontCrack
```

服务账户、工作目录、文件权限和防火墙规则需单独配置。
