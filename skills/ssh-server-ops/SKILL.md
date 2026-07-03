---
name: ssh-server-ops
description: |
  SSH 连接远程服务器执行运维操作。当用户提到服务器、运维、重启服务、查看日志、
  检查状态、操作容器、修改 nginx 配置、数据库查询、部署、宝塔面板时使用此技能。
  即使没有明确说"SSH"，只要涉及对远程服务器的任何操作都应触发。
---

# SSH Server Ops

> server-info.md     — 连接信息模板（运行时真实值由环境变量提供）
> bt-panel.md        — 宝塔 `bt` 工具箱 + 服务管理 + 站点排查速查

### 环境变量（全部不在仓库中保存）

| 变量 | 用途 | 必需 |
| --- | --- | --- |
| `$SERVER_IP` | 服务器 IP 或域名 | ✅ |
| `$SERVER_USER` | SSH 用户名 | ✅ |
| `$SERVER_SSH_KEY` | SSH 密钥路径（默认 `~/.ssh/id_ed25519`） | ✅ |
| `$SERVER_SSH_PORT` | SSH 端口（默认 `22`） | 否 |
| `$SUDO_SSH_PASSWORD` | sudo 密码 | ✅（如需 root） |

以下 `$SSH` 由环境变量拼出：`ssh -i "$SERVER_SSH_KEY" [-p "$SERVER_SSH_PORT"] "$SERVER_USER"@"$SERVER_IP"`；需要 root 时加 `echo "$SUDO_SSH_PASSWORD" | sudo -S` 前缀。

## 原则

- **legwork**: 每个操作前做完整 legwork — 状态 + 日志 + 影响范围 — 然后再动手，避免凭猜测直接重启服务。
- **[RED]**: 动到数据或边界的命令 — `rm -rf`、`DROP`、`DELETE`、`iptables` — 是 **[RED]** 操作，必须明确问用户确认后才执行。

## 会话初始化

本会话未连接时先做：

1. 检查环境变量是否已设置：`$SERVER_IP`、`$SERVER_USER`、`$SUDO_SSH_PASSWORD`。**任一缺失则停止**并提示用户配置，或运行 `/setup-ssh` 引导
2. 读 `server-info.md`，如有 `server-info.local.md` 优先读它
3. 执行 `$SSH "echo ok"` 验证连接。失败则提示用户检查凭据
4. 规划本次操作的范围边界

## 流程 — 环境探索

先回答：这个服务器上跑什么、状态如何。

```bash
$SSH "docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'"
$SSH "systemctl is-active nginx docker containerd 2>/dev/null"
$SSH "echo "$SUDO_SSH_PASSWORD" | sudo -S supervisorctl status"
$SSH "df -h / && echo --- && free -h"
```

完成标准：以上四条都执行并拿到结果。

## 流程 — 服务重启

确认：要重启哪个服务 + 由什么管理（docker/systemd/supervisor）。

```bash
# Docker Compose
$SSH "cd /www/wwwroot/<项目> && docker compose restart <服务>"
# systemd / Supervisor
$SSH "systemctl restart <服务>"
$SSH "echo "$SUDO_SSH_PASSWORD" | sudo -S supervisorctl restart <进程>"
# Nginx — 先测试
$SSH "nginx -t && nginx -s reload"
```

完成标准：重启后验证端口监听和进程 isInActive=active。

## 流程 — 日志排查

确认：问题描述或时间段。

```bash
$SSH "tail -n 100 /www/wwwlogs/<站点>.log"
$SSH "tail -n 50 /www/wwwlogs/<站点>.error.log"
$SSH "docker logs --tail 100 <容器>"
$SSH "echo "$SUDO_SSH_PASSWORD" | sudo -S supervisorctl tail -100 <进程>"
$SSH "journalctl -u <服务> --no-pager -n 50"
```

完成标准：错误原因定位 + 摘要告知用户。

## 流程 — 容器操作

确认：目标容器 + 操作意图（进入/导出配置）。

```bash
$SSH "docker exec -it <容器> /bin/sh"
$SSH "docker inspect <容器> | jq '.[0] | {Image, Mounts:[.Mounts[]|{Type,Source,Destination}], Ports:[.NetworkSettings.Ports|to_entries[]|{key,value}]}'"
```

## 流程 — Nginx 配置

确认：站点名 + 改动目标。**此处任何修改前必须 `nginx -t`。**

```bash
$SSH "cat /www/server/panel/vhost/nginx/<站点>.conf"       # 1. 先看现有配置
$SSH "nginx -t && nginx -s reload"                          # 2. 改完必须 test
```

自定义配置放到 `/www/server/panel/vhost/nginx/<站点>.conf`，不要直接改 nginx.conf — 宝塔重启会覆盖。

## 流程 — 数据库

确认：目标库 + SQL（密码不入库，询问用户）。

```bash
$SSH "docker exec <mysql容器> mysql -u<用户> -p'<密码>' -e '<SQL>'"
$SSH "docker exec <pg容器> psql -U <用户> -c '<SQL>'"
```

## 流程 — 服务应急响应

先排查再恢复 — 并行 legwork，不要跳步。

```bash
# 1. 端口与资源（并行执行）
$SSH "ss -tlnp | grep -E ':(80|443|<业务端口>)'"
$SSH "uptime && echo --- && free -h && echo --- && df -h /"
$SSH "docker stats --no-stream"
# 2. 进程与日志
$SSH "docker ps -a --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'"
$SSH "docker logs --tail 50 <容器>"
$SSH "tail -n 50 /www/wwwlogs/<站点>.error.log"
```

完成标准：定位根因 + 摘要给用户，再决定恢复动作。

## 流程 — 容器导出（迁移/备份）

确认：目标容器。保留 named volume 以保留数据。

```bash
$SSH "docker inspect <容器> | jq '.[0] | {Name, Image, State, Mounts:[.Mounts[]|{Type,Source,Destination}], Ports:[.NetworkSettings.Ports|to_entries[]|{key,value}]}'"
```

完成标准：产出包含 image/ports/volumes 的 compose 片段，标注数据保留策略。

## [RED] 需要确认的操作

以下只能在用户明确确认后执行：

- `rm -rf`、`DROP`、`DELETE`
- `iptables` 规则修改
- `rm -f /www/server/panel/data/*.login`（清理登录封禁）
- 任何影响对外服务的重启

## 安全规则

- [RED] 操作必须询问
- sudo 密码 **只** 通过 `$SUDO_SSH_PASSWORD` 环境变量使用，不写入文件、不当明文传输
- 改 nginx 前先 `nginx -t`
- 操作完清理临时文件
