---
title: "常用命令"
description: "Lorem ipsum dolor sit amet"
pubDate: "Jul 15 2022"
---

```
# ssh是否被爆破
sudo lastb -n 20

systemctl status openresty
systemctl start openresty
systemctl stop openresty
openresty -t
openresty -s reload

# 看端口占用
ss -tulnp

# 进入容器
podman exec -it xxx bash

# pg连接数据库
psql -U xxxusername -d xxxdb

# 修改ssh配置
nano /etc/ssh/sshd_config

# 生成ssh key
ssh-keygen -t rsa -b 4096 -f C:\Users\Administrator\.ssh\xxx_key

```

