# 宝塔面板 (BT Panel) 参考

> 来源：[宝塔官方命令大全](https://www.bt.cn/new/btcode.html)

## `bt` 命令行工具箱

直接在 SSH 中执行 `bt` 进入交互式菜单，或 `bt <编号>` 单命令执行：

| 编号 | 功能 | 编号 | 功能 |
| --- | --- | --- | --- |
| 1 | 重启面板 | 13 | 查看默认登录信息 |
| 2 | 停止面板 | 14 | 清理系统垃圾 |
| 3 | 启动面板 | 15 | 修复面板（更新/修复） |
| 4 | 重载面板 | 16 | 设置日志切割压缩 |
| 5 | 修改面板密码（交互式） | 17 | 设置面板自动备份 |
| 6 | 修改面板用户名（交互式） | 18 | （保留） |
| 7 | 修改 MySQL root 密码 | 19-21 | （保留） |
| 8 | 修改面板端口（交互式） | 22 | 显示面板错误日志 |
| 9 | 清除面板缓存 | 23 | 关闭 BasicAuth |
| 10 | 清除登录限制 | 24 | 关闭动态口令 |
| 11 | （保留） | 25 | 设置文件历史副本 |
| 12 | 取消域名绑定限制 | | |

## 面板自身管理

```bash
# 查看面板端口
cat /www/server/panel/data/port.pl

# 查看面板默认登录信息
bt 14

# 查看面板错误日志
cat /tmp/panelBoot.pl

# 查看软件安装日志
cat /tmp/panelExec.log

# 面板数据目录
/www/server/panel/
├── data/           # 端口、域名、授权IP、SSL配置
│   ├── port.pl
│   ├── domain.conf
│   ├── limitip.conf
│   └── ssl.pl
├── vhost/          # 站点 vhost 配置导出
│   ├── nginx/
│   └── apache/
└── logs/           # 面板访问/错误日志
```

## 服务管理命令速查

```bash
# Nginx
/etc/init.d/nginx start|stop|restart|reload
# 配置: /www/server/nginx/conf/nginx.conf

# MySQL
/etc/init.d/mysqld start|stop|restart|reload
# 配置: /etc/my.cnf  数据: /www/server/data/

# PHP-FPM（按版本替换 74/80/81/82）
/etc/init.d/php-fpm-82 start|stop|restart|reload
# 配置: /www/server/php/82/etc/php.ini

# Redis（宝塔安装版）
/etc/init.d/redis start|stop|restart
# 配置: /www/server/redis/redis.conf

# PostgreSQL（宝塔安装）
# 数据目录: /www/server/pgsql/data/
# 二进制: /www/server/pgsql/bin/
# ⚠️ 注意不是 /www/server/postgresql/

# Pure-FTPd
/etc/init.d/pure-ftpd start|stop|restart

# Apache
/etc/init.d/httpd start|stop|restart|reload
```

## 关键路径速查

| 用途 | 路径 |
| --- | --- |
| 站点根目录 | `/www/wwwroot/` |
| Nginx vhost | `/www/server/panel/vhost/nginx/` |
| Nginx 主配置 | `/www/server/nginx/conf/nginx.conf` |
| Nginx 安装目录 | `/www/server/nginx/` |
| Apache vhost | `/www/server/panel/vhost/apache/` |
| Apache 安装目录 | `/www/server/httpd/` |
| MySQL 数据 | `/www/server/data/` |
| phpMyAdmin | `/www/server/phpmyadmin/` |
| PostgreSQL 数据 | `/www/server/pgsql/data/` |
| PHP 安装目录 | `/www/server/php/` |
| Redis 配置 | `/www/server/redis/redis.conf` |
| 数据库备份 | `/www/backup/database/` |
| 站点备份 | `/www/backup/site/` |
| 网站访问日志 | `/www/wwwlogs/` |
| MySQL 错误日志 | `/www/server/data/*.err` |
| 面板日志 | `/tmp/panelBoot.pl` |
| SSL 证书（面板面板内置） | `/www/server/panel/ssl/` |

## 站点/域名相关

```bash
# 查看绑定到面板的域名
cat /www/server/panel/data/domain.conf

# 删除域名绑定（网站不在面板域名栏出现）
rm -f /www/server/panel/data/domain.conf

# 查看授权 IP 白名单
cat /www/server/panel/data/limitip.conf

# 关闭访问限制（慎用，会暴露面板到所有 IP）
rm -f /www/server/panel/data/limitip.conf

# 清理登录限制（连续输错密码后的临时封禁）
rm -f /www/server/panel/data/*.login

# 关闭面板 SSL（HTTPS 访问）
rm -f /www/server/panel/data/ssl.pl && bt restart
```

## 常用排查

```bash
# 面板卡死/502 → 重启面板
bt 1

# 面板加载慢 → 清理缓存
bt 9

# 登不上了 → 先看错误日志 + 清理登录限制
cat /tmp/panelBoot.pl
rm -f /www/server/panel/data/*.login

# 面板挂了 → 修复面板
bt 15

# 软件安装失败 → 看安装日志
cat /tmp/panelExec.log

# 查看 MySQL 错误
cat /www/server/data/*.err
```

## 安装方式（如需重装）

```bash
# Ubuntu/Debian
wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh

# CentOS
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```
