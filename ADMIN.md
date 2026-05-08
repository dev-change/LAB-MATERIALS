# 仓库管理维护手册

> 本文档面向仓库管理员与维护人员。

---

## 一、仓库基本信息

| 项目 | 内容 |
|------|------|
| 仓库路径 | `D:\GitRepos\lab-materials.git` |
| 访问地址 | `ssh://Administrator@192.168.1.114/D:/GitRepos/lab-materials.git` |
| 服务端 | Windows OpenSSH + Git Bare Repository |
| 分支保护 | `main`/`master` 禁止直接 push，强制 squash merge |

---

## 二、日常维护操作

### 2.1 检查服务状态

```powershell
# 检查 SSH 服务
Get-Service sshd | Select-Object Name, Status, StartType

# 检查防火墙规则
Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" | Select-Object Name, Enabled, Action

# 检查磁盘空间（D 盘）
Get-PSDrive D | Select-Object Used, Free
```

### 2.2 备份仓库

**手动备份**：
```powershell
$date = Get-Date -Format "yyyyMMdd_HHmmss"
$backupDir = "D:\GitRepos\backups"
if (!(Test-Path $backupDir)) { New-Item -ItemType Directory -Path $backupDir -Force }

# 打包备份
Compress-Archive -Path "D:\GitRepos\lab-materials.git" `
    -DestinationPath "$backupDir\lab-materials.git_$date.zip" -Force

Write-Host "备份完成: $backupDir\lab-materials.git_$date.zip"
```

**自动备份脚本**（保存为 `D:\GitRepos\backup.ps1`，配合任务计划程序每周运行）：
```powershell
# backup.ps1
$date = Get-Date -Format "yyyyMMdd"
$backupDir = "D:\GitRepos\backups"
$repoPath = "D:\GitRepos\lab-materials.git"
$retentionDays = 30

# 创建备份目录
if (!(Test-Path $backupDir)) { New-Item -ItemType Directory -Path $backupDir -Force }

# 执行备份
Compress-Archive -Path $repoPath `
    -DestinationPath "$backupDir\lab-materials.git_$date.zip" -Force

# 清理旧备份
Get-ChildItem $backupDir -Filter "*.zip" | Where-Object {
    $_.LastWriteTime -lt (Get-Date).AddDays(-$retentionDays)
} | Remove-Item -Force

Write-Host "$(Get-Date) 备份完成，已清理 $retentionDays 天前的旧备份。"
```

---

## 三、用户管理

### 3.1 添加队友 SSH 公钥

当队友发送公钥时：

```powershell
$teammateKey = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... teammate@device"
Add-Content -Path "$env:USERPROFILE\.ssh\authorized_keys" -Value $teammateKey -Encoding UTF8
```

**查看当前已授权密钥**：
```powershell
Get-Content "$env:USERPROFILE\.ssh\authorized_keys"
```

**撤销某人的访问权限**：编辑 `authorized_keys`，删除对应公钥行。

### 3.2 创建专用 Git 用户（推荐用于正式场景）

如需隔离权限，可创建 Windows 本地用户 `git`：
```powershell
# 创建用户（密码自行设置）
$password = Read-Host "输入密码" -AsSecureString
New-LocalUser -Name "git" -Password $password -FullName "Git Service" -Description "Git SSH Access"

# 创建该用户的 .ssh 目录
$gitHome = "C:\Users\git"
$sshDir = "$gitHome\.ssh"
New-Item -ItemType Directory -Path $sshDir -Force

# 设置目录权限（仅 git 用户可访问）
icacls $sshDir /inheritance:r
icacls $sshDir /grant:r "git:(OI)(CI)F"
```

---

## 四、Hook 维护

### 4.1 修改 update hook

Hook 文件位置：`D:\GitRepos\lab-materials.git\hooks\update`

当前规则：
- 拦截 `main`/`master` 的直接 push
- 允许 squash merge
- 允许首次初始化 push
- 允许删除分支

如需调整规则，编辑该文件后无需重启任何服务，即时生效。

### 4.2 添加其他 Hook（可选）

- **`pre-receive`**：在 update 之前执行，可全局拒绝某些提交
- **`post-receive`**：push 完成后触发，可用于发送通知、自动部署等

---

## 五、故障排查

### 5.1 队友无法连接

| 现象 | 排查步骤 |
|------|---------|
| 连接超时 | 检查本机 IP 是否变动；检查防火墙是否放行 22 端口；确认台式机开机 |
| 权限拒绝 | 检查 `authorized_keys` 中是否包含对方公钥；检查 `.ssh` 目录权限 |
| 认证失败 | 确认队友使用正确的用户名（`Administrator`）；检查私钥是否匹配 |

### 5.2 仓库损坏修复

```bash
# 进入裸仓库
cd /d D:\GitRepos\lab-materials.git

# 运行一致性检查
git fsck --full

# 如果发现问题，尝试用备份恢复
```

### 5.3 服务重启

```powershell
Restart-Service sshd
```

---

## 六、IP 变动处理

如果台式机 IP 发生变化：

1. 获取新 IP：
   ```powershell
   (Get-NetIPAddress -AddressFamily IPv4 | Where-Object {
       $_.IPAddress -like '192.168.*' -and $_.IPAddress -ne '127.0.0.1'
   }).IPAddress | Select-Object -First 1
   ```

2. 通知所有队友更新仓库 remote URL：
   ```bash
   git remote set-url origin ssh://Administrator@新IP/D:/GitRepos/lab-materials.git
   ```

3. 建议：在路由器中为台式机设置**静态 IP 绑定**，避免频繁变动。

---

## 七、关于文档格式的决策（维护人视角）

**结论：要求提交人直接提供 Markdown 文件。**

理由：
- Markdown 是 Git 原生友好格式，diff 清晰、版本控制高效
- Word/PDF 是二进制格式，Git 无法有效追踪变更，仓库体积膨胀快
- 维护人没有义务为每个人的文档做格式转换，这会形成单点瓶颈
- 现代编辑器（VS Code、Typora、Obsidian）对 Markdown 的支持已经非常完善

**例外情况处理**：
- 如果某些资料只有 Word/PDF 版本（如老师下发的课件），允许直接放入仓库，但需在文件名中标注格式，例如：`数字电路_课件_v1.2.pdf`
- 鼓励将核心内容（如八股、面经）整理为 Markdown，便于迭代更新

---

## 八、关于镜像仓库

**是否需要镜像？**

建议设置**单向镜像备份**（到另一台机器、NAS 或云端），而非双向同步的多主镜像。原因：
- 裸仓库本来就是协作中心，多主镜像会增加冲突复杂度
- 单向备份足够满足"防止台式机硬盘损坏"的容灾需求

**简易镜像方案**（定期执行）：
```bash
# 在另一台机器上执行
git clone --mirror ssh://Administrator@192.168.1.114/D:/GitRepos/lab-materials.git
cd lab-materials.git
git remote update
```

或者使用 GitHub/GitLab 作为冷备份，定期 push：
```bash
git remote add backup https://github.com/your-org/lab-materials-mirror.git
git push backup --mirror
```

---

*最后更新：2026-05-07*
