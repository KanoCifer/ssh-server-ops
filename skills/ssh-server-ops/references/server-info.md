# Server Connection Info

> 这是一个模板 — 复制到 `references/server-info.local.md` 并填入真实值。
> `server-info.local.md` 已被 gitignore，不会提交到仓库。

## SSH 连接

真实连接信息由环境变量提供，不出现在本模板文件：

```bash
# 由 $SERVER_USER / $SERVER_IP / $SERVER_SSH_KEY / $SERVER_SSH_PORT 拼出
ssh -i $SERVER_SSH_KEY [-p $SERVER_SSH_PORT] $SERVER_USER@$SERVER_IP
```

| 项目      | 来源                                   |
| --------- | -------------------------------------- |
| 用户      | 环境变量 `$SERVER_USER`                |
| 主机      | 环境变量 `$SERVER_IP`                  |
| 密钥      | 环境变量 `$SERVER_SSH_KEY`             |
| 端口      | 环境变量 `$SERVER_SSH_PORT`（默认 22） |
| sudo 密码 | 环境变量 `$SUDO_SSH_PASSWORD`          |

## 服务器环境

| 项目       | 说明                                  |
| ---------- | ------------------------------------- |
| 管理面板   | 宝塔面板 (BT Panel)                   |
| 服务管理   | Docker Compose / systemd / Supervisor |
| Web 服务器 | Nginx                                 |

## 宝塔面板关键路径

| 用途           | 路径                             |
| -------------- | -------------------------------- |
| 站点根目录     | `/www/wwwroot/`                  |
| Nginx 主配置   | `/www/server/nginx/conf/`        |
| Nginx 站点配置 | `/www/server/panel/vhost/nginx/` |
| 网站日志       | `/www/wwwlogs/`                  |
| MySQL 数据     | `/www/server/data/`              |
| PostgreSQL     | `/www/server/pgsql/data/`        |
| 宝塔 CLI       | `bt`                             |
| 面板数据目录   | `/www/server/panel/`             |
| 面板端口文件   | `/www/server/panel/data/port.pl` |
| 面板错误日志   | `/tmp/panelBoot.pl`              |
| 软件安装日志   | `/tmp/panelExec.log`             |
| 数据库备份     | `/www/backup/database/`          |
| 站点备份       | `/www/backup/site/`              |
| PHP 安装目录   | `/www/server/php/`               |
| Redis 配置     | `/www/server/redis/redis.conf`   |
| FTP 服务       | `/www/server/pure-ftpd/`         |
