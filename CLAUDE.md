# SSH Server Ops — 技能开发仓库

Claude Code 技能开发项目，包含 SSH 运维操作 + 连接初始化两个技能。

## 技能列表

| 技能 | 入口 | 用途 | 触发 |
| --- | --- | --- | --- |
| ssh-server-ops | `skills/ssh-server-ops/SKILL.md` | 在远程服务器上执行运维操作 | 用户说服务器/运维/重启/日志/容器/… |
| setup-ssh | `skills/setup-ssh/SKILL.md` | 初始化 / 更新 SSH 连接信息 | 用户说配置连接/添加服务器/初始化 ssh |

## 目录结构

```
.
├── CLAUDE.md
├── LICENSE
├── .gitignore
├── skills/
│   ├── setup-ssh/
│   │   └── SKILL.md                  # 连接信息初始化
│   └── ssh-server-ops/
│       ├── SKILL.md                  # 运维操作（按流程分支，RED gate）
│       ├── evals/
│       │   └── evals.json
│       └── references/
│           ├── server-info.md        # 连接信息模板（占位符）
│           └── server-info.local.md  # 真实凭据（gitignore）
│           └── bt-panel.md           # 宝塔命令速查
```

## 凭据管理 — 全部走环境变量

| 变量 | 用途 | 写入位置 |
| --- | --- | --- |
| `$SERVER_IP` | 服务器 IP / 域名 | shell env |
| `$SERVER_USER` | SSH 用户名 | shell env |
| `$SERVER_SSH_KEY` | SSH 密钥路径 | shell env（默认 `~/.ssh/id_ed25519`） |
| `$SERVER_SSH_PORT` | SSH 端口（可选，默认 22） | shell env |
| `$SUDO_SSH_PASSWORD` | sudo 密码 | shell env |

- `server-info.md`：占位模板，入库
- `server-info.local.md`：不含 IP / 不含密码（gitignore），只存用户/密钥/端口/环境备注
- 真实运行时需要的 IP + 密码只存在于用户本机 shell 环境
- 暂无 helper 脚本，用户按 `/setup-ssh` 的指引在 `~/.zshrc` 中 set 变量

## 开发规范

- 每个技能一个子目录，入口 `SKILL.md`，`name` 与目录名一致
- 参考文档放 `references/`，评估用例放 `evals/evals.json`
- `SKILL.md` = 流程步骤 + 完成标准；命令速查都走 `references/`（渐进式披露）
- 搜索关键词"writing-great-skills"可查看技能方法论

## 添加新技能

1. `skills/` 下创建 `<skill-name>/SKILL.md`
2. frontmatter 写明 description + 触发词
3. 按需添加 `references/` `evals/`
4. 在本文档的技能列表里注册
