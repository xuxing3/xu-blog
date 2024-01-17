---
title: "Error Establishing A Database Connection"
date: "2024-01-17T12:00:03+08:00"
tags:
  - Bash
draft: false
---

## 背景

我的 wordpress 站点使用的是比较老的主题， 很久没人维护了，前段时间看到有些插件需要升级， 然后就头脑发热全升级了；

升级后应该是插件和主题兼容性有些问题， 导致我的博客每过一段时间都会 report `Error Establishing A Database Connection`.

幸亏 Tencent 云提供了站点存活行检测，每次博客数据库出现问题后, 它会给我发个短信提醒我一下， 我就直接重启下我的 vps， 然后博客就恢复了。

搞了那么几次发现还是太麻烦了， 所以今天去尝试去检查为什么会报这个错误。

## 检查 mysql 服务

尝试登录下 mysql web 页面输入用户名密码发现无法正常登陆， 报`Getting error mysqli::real_connect()`

检查了下 mysql 进程， 发现不在 running，尝试重启了下 mysql 服务， 发现我的博客恢复正常了。

检查了下异常时间点的 log， 看到进程由于 OOO 被 kill 掉了, kill 掉后也没正常启动.

```bash
root@vultr:~# journalctl | grep mysql | grep Jan
Jan 16 11:43:21 vultr.guest kernel: [  763]   111   763   290638    58917   802816        0             0 mysqld
Jan 16 11:43:21 vultr.guest kernel: Out of memory: Kill process 763 (mysqld) score 234 or sacrifice child
Jan 16 11:43:21 vultr.guest kernel: Killed process 763 (mysqld) total-vm:1162552kB, anon-rss:235668kB, file-rss:0kB, shmem-rss:0kB
Jan 16 11:43:21 vultr.guest kernel: oom_reaper: reaped process 763 (mysqld), now anon-rss:0kB, file-rss:0kB, shmem-rss:0kB
Jan 16 11:43:20 vultr.guest systemd[1]: mysql.service: Main process exited, code=killed, status=9/KILL
```

鉴于这个 OOO 的排查对我来说有点难， 索性想着要不写个脚本， 周期 run 一下帮我检查下进程状态， 如果不是 active 的就直接帮我重启下进程。

脚本如下:

```bash
#!/bin/bash

LOG_FILE="/root/mysql-check.log"

MYSQL_STATUS=$(systemctl is-active mysql)

if [ "$MYSQL_STATUS" != "active" ]; then
    echo "$(date) - MySQL 服务不在运行状态，正在尝试重启..." >> "$LOG_FILE"

    systemctl restart mysql

    NEW_MYSQL_STATUS=$(systemctl is-active mysql)

    echo "$(date) - MySQL 服务状态: $NEW_MYSQL_STATUS" >> "$LOG_FILE"
else
    echo "$(date) - MySQL 服务正常运行" >> "$LOG_FILE"
fi
```

使用 crontab 每 5 分钟执行一次:

```
root@vultr:~# crontab -e
*/5 * * * * /root/mysql-check.sh
```
