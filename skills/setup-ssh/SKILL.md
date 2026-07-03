---
name: setup-ssh
description: |
  初始化或更新 SSH 连接信息 / 服务器环境信息。
disable-model-invocation: true
---

# Setup — SSH 连接信息初始化

本技能为 `ssh-server-ops` 技能生成/更新运行时凭据文件。真实连接信息保存在
`skills/ssh-server-ops/references/server-info.local.md`

## 流程

### 1. 收集非敏感连接信息

先检查 `server-info.local.md` 是否存在。如果存在，读取并展示现有值，问用户要修改哪些项（增量编辑）；如果不存在，按下列清单收集全部项。

按下列清单向用户询问（只收不写入文件/不入库的项）：

```
[必需 — 写入环境变量]
  SSH 用户名:
  SSH 密钥路径:        （默认 ~/.ssh/id_ed25519）

[可选 — 写入环境变量]
  SSH 端口:            （默认 22）

[必需 — 环境变量，不在本环节收集]
  IP / 域名:           → 由用户后续在 shell 中 export（见步骤 3）
  sudo 密码:           → 由用户后续在 shell 中 export（见步骤 4）

[环境 — 决定哪些运维命令可用]
  管理面板:
    □ 宝塔面板 (BT Panel)    → bt-panel.md 命令可用
    □ 无面板
  服务管理:
    □ Docker Compose
    □ systemd
    □ Supervisor
  Web 服务器:
    □ Nginx
    □ Apache
```

完成标准：用户名 + 密钥路径拿到。

### 2. 写入 server-info.local.md

生成 `skills/ssh-server-ops/references/server-info.local.md`。**文件中不含 IP / sudo 密码**：

```markdown
# Server Connection Info (local — gitignored — 无 IP / 无密码)

| 项目 | 值                                            |
| ---- | --------------------------------------------- |
| 用户 | <用户>                                        |
| 密钥 | <密钥>                                        |
| 端口 | <端口>                                        |
| IP   | 环境变量 $SERVER_IP（不出现在本文件）         |
| sudo | 环境变量 $SUDO_SSH_PASSWORD（不出现在本文件） |

## 服务器环境

- 管理面板: <面板>
- 服务管理: <管理工具列表>
- Web 服务器: <Web 服务器>

## 备注

<用户补充的额外信息>
```

完成标准：文件落盘 + 确认路径。

### 3. 引导配置连接环境变量

在 shell 配置中设置 IP、用户名、密钥、端口：

```bash
# ~/.zshrc 或 ~/.bash_profile
export SERVER_IP='<服务器IP或域名>'
export SERVER_USER='<SSH用户名>'
export SERVER_SSH_KEY='<密钥路径>'     # 默认 ~/.ssh/id_ed25519
export SERVER_SSH_PORT='22'             # 非默认时设置
```

说明：

- 这些变量不在仓库、不在大模型上下文
- 切换服务器时覆盖 export 即可（如 `export SERVER_IP='other.host'`）

完成标准：用户确认已在 shell 配置中设置连接变量。

### 4. 引导配置 sudo 密码环境变量

```bash
export SUDO_SSH_PASSWORD='<sudo密码>'
```

说明：密码只存在于本机 shell，`ssh-server-ops` 技能每次读取它。

完成标准：用户确认已设置。

### 5. 验证连接

```bash
ssh -i "$SERVER_SSH_KEY" [-p "$SERVER_SSH_PORT"] "$SERVER_USER"@"$SERVER_IP" \
  "echo ok && uname -a && docker ps --format 'table {{.Names}}\t{{.Status}}'"
```

- 成功 → 打印连接摘要
- 失败 → 按故障树提示用户：
  - `Connection timed out` → 检查安全组、IP 是否正确
  - `Permission denied` → 检查密钥 `chmod 600`、用户名是否正确
  - `Connection refused` → 检查 SSH 端口、IP 是否正确

完成标准：拿到 `echo ok`。

### 6. 配置完成总结

连接成功后，打印摘要：

```
✅ 连接就绪
  用户: $SERVER_USER
  主机: $SERVER_IP (port $SERVER_SSH_PORT)
  环境: <管理面板> / <服务管理> / <Web 服务器>
  
切换服务器时覆盖对应 export 即可。
```

## 安全规则

- `server-info.local.md` 不含 IP 和密码，推公共仓库也安全
- 连接变量（`$SERVER_IP`、`$SERVER_USER`、`$SERVER_SSH_KEY`）**只**通过环境变量使用
- sudo 密码 **只** 通过 `$SUDO_SSH_PASSWORD`，不写入任何文件
- 不要把 `server-info.local.md` 的路径和内容透露给大模型外的第三方
