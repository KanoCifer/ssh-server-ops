# ssh-server-ops

Claude Code 技能集合，通过 SSH 连接远程服务器执行运维操作。凭据全部走环境变量，仓库中不含任何 IP / 密码。

## 技能列表

| 技能 | 用途 | 触发 |
| --- | --- | --- |
| `ssh-server-ops` | 在远程服务器执行运维操作（状态 / 日志 / 重启 / 容器 / nginx / 数据库） | 用户说"服务器/运维/重启/日志/容器/…" |
| `setup-ssh` | 初始化 / 更新 SSH 连接信息，引导配置环境变量 | 用户说"配置连接/添加服务器/初始化 ssh" |

## 安装

### 手动安装

把技能目录复制到 Claude Code 的技能路径：

```bash
# 作为 user-level 技能（全局可用）
cp -r skills/ssh-server-ops ~/.claude/skills/
cp -r skills/setup-ssh ~/.claude/skills/

# 或作为 project-level 技能（放在项目 .claude/skills/ 下）
mkdir -p .claude/skills
cp -r skills/ssh-server-ops .claude/skills/
cp -r skills/setup-ssh .claude/skills/
```

### 通过 Claude Code Plugin 安装

```bash
# 添加市场
claude plugins marketplace add KanoCifer/ssh-server-ops

# 安装插件
claude plugins install ssh-server-ops@ssh-server-ops
```

或在 Claude 对话框 `/plugin` 中交互完成：

```
/plugin marketplace add KanoCifer/ssh-server-ops
/plugin install ssh-server-ops@ssh-server-ops
```

### 通过 npx skills 安装（跨 Agent）

支持 70+ 编码 Agent（Claude Code、Codex、Cursor 等）：

```bash
# 交互式安装
npx skills add KanoCifer/ssh-server-ops
```

## 配置

真实凭据通过环境变量提供，在 `~/.zshrc`（或 `~/.bash_profile`）中设置：

```bash
export SERVER_IP='<服务器IP或域名>'
export SERVER_USER='<SSH用户名>'
export SERVER_SSH_KEY='~/.ssh/id_ed25519'   # 密钥路径
export SERVER_SSH_PORT='22'                  # 可选，默认 22
export SUDO_SSH_PASSWORD='<sudo密码>'         # 如需 root 权限
```

切换到另一台服务器时，覆盖对应的 `export` 即可。

## 使用

```text
# 先配置连接信息
> 添加一台服务器                    # 触发 /setup-ssh

# 然后执行运维操作
> 帮我看下服务器上跑哪些 Docker 容器   # 触发 ssh-server-ops → 环境探索流程
> nginx 报 502 了，帮我看下            # 触发 ssh-server-ops → 日志排查 + 应急响应
> 重启后端服务                        # 触发 ssh-server-ops → 服务重启流程
```

## 工作流程

```
shell env ($SERVER_IP / $SERVER_USER / $SERVER_SSH_KEY / $SUDO_SSH_PASSWORD)
        │
        ▼
ssh-server-ops 技能
        │
        ├── 环境探索   — docker / systemd / supervisor / 磁盘
        ├── 日志排查   — 站点日志 / docker logs / journalctl
        ├── 服务重启   — docker compose / systemctl / supervisor
        ├── 容器操作   — exec / inspect / 导出 compose
        ├── Nginx 配置 — 先 nginx -t 再 reload
        ├── 数据库     — mysql / psql（密码运行时询问）
        └── 应急响应   — 并行 legwork → 定位根因 → 恢复
```

## 目录结构

```
.
├── LICENSE
├── CLAUDE.md              # 项目内部开发规范
├── README.md
└── skills/
    ├── setup-ssh/
    │   └── SKILL.md      # 连接信息初始化
    └── ssh-server-ops/
        ├── SKILL.md      # 运维操作（流程 + RED gate）
        ├── evals/
        │   └── evals.json # 评估用例
        └── references/
            ├── server-info.md   # 连接信息模板（占位符，入库）
            ├── server-info.local.md  # 真实凭据（gitignore）
            └── bt-panel.md      # 宝塔面板命令速查
```

## 安全

- IP / 用户名 / 密钥路径 / sudo 密码 **全部走环境变量**，不在仓库中保存
- `[RED]` 操作（`rm -rf` / `DROP` / `iptables`）必须用户明确确认
- 改 nginx 前先 `nginx -t`

## License

[MIT](LICENSE)
