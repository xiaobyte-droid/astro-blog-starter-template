---
title: "常用命令"
description: "Lorem ipsum dolor sit amet"
pubDate: "Jul 15 2022"
heroImage: "/blog-placeholder-2.jpg"
---

1. 目录映射 

   ```
   mklink /j "C:\Users\Administrator\AppData\Local\go-build" "G:\AppData\go-build"
   mklink /j "C:\Users\Administrator\AppData\Local\JetBrains" "G:\AppData\JetBrains"
   mklink /j "C:\Users\Administrator\AppData\Local\Google" "G:\AppData\Google"
   mklink /j "C:\Users\Administrator\AppData\Local\JetBrains"  "G:\AppData\JetBrains"
   mklink /j "C:\Users\Administrator\go" "G:\AppData\goPkg"
   ```

2. frp

   ```
   podman cp 129d06915cc2:/etc/frp/frps.toml /root/frp
   
   ##### frps.toml #####
   bindAddr = "0.0.0.0"
   bindPort = 7000
   
   auth.method = "token"
   auth.token = "token123456"
   
   webServer.addr = "0.0.0.0"
   webServer.port = 7500
   webServer.user = "admin"
   webServer.password = "admin_M26iG5"
   ##### frps.toml #####
   
   ##### frpc.toml #####
   serverAddr = "服务器ip"
   serverPort = 7000
   
   auth.method = "token"
   auth.token = "token123456jwb"
   
   webServer.addr = "0.0.0.0"
   webServer.port = 7500
   webServer.user = "admin"
   webServer.password = "admin_M26iG5"
   webServer.pprofEnable = false
   
   # tls
   #transport.tls.certFile = "/etc/frp/ssl/client.crt"
   #transport.tls.keyFile = "/etc/frp/ssl/client.key"
   #transport.tls.trustedCaFile = "/etc/frp/ssl/ca.crt"
   
   [[proxies]]
   name = "demo"
   type = "tcp"
   localIP = "172.17.0.1"
   localPort = 8443
   remotePort = 8080
   ##### frpc.toml #####
   
   
   podman run -d --name frps \
     -p 7000:7000 \
     -p 7500:7500 \
     -p 8080:8080 \
     -v /root/frp/frps.toml:/etc/frp/frps.toml \
     5b36465b6b5f
     
     
     
     # 注意：如果用8080访问，docker端口要映射8080
   ```

3. redis

   ```
   podman pull docker.io/redis:7.4
   
   podman run -d \
     --name redis \
     --network host \
     -p 6379:6379 \
     -v ~/data/redis:/data \
     58914618496b --appendonly yes
   ```

4. jdk

   ```
   https://repo.huaweicloud.com/java/jdk/
   https://mirrors.huaweicloud.com/openjdk/
   /usr/local/jdk-17.0.2
   
   
   nano ~/.bashrc
   export JAVA_HOME=/usr/local/jdk-17.0.2
   export PATH="$JAVA_HOME/bin:$PATH"
   
   source ~/.bashrc
   ```

5. acme

   ```
   acme.sh --issue -d test.com --webroot /home/xxx/nginx/html
   
   
   ~/.acme.sh/acme.sh --install-cert -d test.com \
   --key-file       /home/xxx/nginx/ssl/key.pem  \
   --fullchain-file /home/xxx/nginx/ssl/cert.pem
   
   
   /home/xxx/nginx/ssl/cert.pem
   /home/xxx/nginx/ssl/key.pem
   ```

6. tdlib

   ```
   ## 启动命令
   java -jar -Djava.library.path=. td-app-0.0.1-SNAPSHOT.jar  ##启动命令
   
   ## 启动脚本
   #!/bin/sh
   nohup /root/jdk-17.0.2/bin/java -jar '-Djava.library.path=.' td-app-0.0.1-SNAPSHOT.jar > app.log &
   echo $! > pid.file
   
   ## 停止脚本
   #!/bin/sh
   kill $(cat pid.file) && rm pid.file
   
   ## 环境变量
   export SQLITE_APP_DB_URL="jdbc:sqlite:/root/db/app.db"
   export SQLITE_CONFIG_DB_URL="jdbc:sqlite:/root/db/config.db"
   
   ## 时区设置
   timedatectl status
   timedatectl list-timezones | grep -i shanghai
   timedatectl set-timezone Asia/Shanghai
   timedatectl status
   
   ####################################################################start########################################################################
   #!/bin/bash
   
   # 应用目录
   export SQLITE_APP_DB_URL=jdbc:sqlite:/root/db/app.db
   export TDLIB_DIR=/root/app/tdlib
   APP_DIR="/root/app"
   PID_FILE="$APP_DIR/pid.file"
   LOG_FILE="$APP_DIR/app.log"
   RESTART_LOG="$APP_DIR/restart.log"
   APP_NAME="myapp.jar"
   JAVA_HOME="/usr/local/jdk1.8.0_192/bin"
   
   # 停止应用
   echo "$(date): 停止应用程序..." >> $RESTART_LOG
   
   if [ -f $PID_FILE ]; then
       PID=$(cat $PID_FILE)
       echo "$(date): 找到PID: $PID" >> $RESTART_LOG
       
       # 发送终止信号
       kill $PID 2>/dev/null
       
       # 等待进程终止，最多等待30秒
       TIMEOUT=30
       COUNT=0
       while [ $COUNT -lt $TIMEOUT ]; do
           if ! kill -0 $PID 2>/dev/null; then
               echo "$(date): 应用程序已停止" >> $RESTART_LOG
               break
           fi
           echo "$(date): 等待进程停止... ($((COUNT+1))秒)" >> $RESTART_LOG
           sleep 1
           COUNT=$((COUNT+1))
       done
       
       # 如果超时后进程仍然存在，强制杀死
       if kill -0 $PID 2>/dev/null; then
           echo "$(date): 正常停止超时，强制杀死进程" >> $RESTART_LOG
           kill -9 $PID
           sleep 2
       fi
       
       # 删除pid文件
       rm -f $PID_FILE
       echo "$(date): PID文件已删除" >> $RESTART_LOG
   fi
   
   # 再次检查是否还有相关进程运行
   if pgrep -f $APP_NAME > /dev/null; then
       echo "$(date): 仍有相关进程运行，强制杀死" >> $RESTART_LOG
       pkill -9 -f $APP_NAME
       sleep 2
   fi
   
   # 备份旧日志
   if [ -f $LOG_FILE ]; then
       mv $LOG_FILE $LOG_FILE.$(date +%Y%m%d_%H%M%S)
       echo "$(date): 日志文件已备份" >> $RESTART_LOG
   fi
   
   # 启动应用
   echo "$(date): 启动应用程序..." >> $RESTART_LOG
   cd $APP_DIR
   nohup $JAVA_HOME/java -jar '-Djava.library.path=.' $APP_NAME > app.log 2>&1 &
   echo $! > pid.file
   
   # 验证应用是否成功启动
   sleep 3
   if [ -f $PID_FILE ] && kill -0 $(cat $PID_FILE) 2>/dev/null; then
       echo "$(date): 应用程序重启完成，PID: $(cat $PID_FILE)" >> $RESTART_LOG
   else
       echo "$(date): 警告：应用程序可能启动失败" >> $RESTART_LOG
   fi
   
   #########################################################################end#####################################################################
   crontab -e
   0 4 * * * /usr/local/bin/restart-app.sh >> /root/app/tmp.txt    
   crontab -l
   ```

7. pve

   ```
   # network interface settings; autogenerated
   # Please do NOT modify this file directly, unless you know what
   # you're doing.
   #
   # If you want to manage parts of the network configuration manually,
   # please utilize the 'source' or 'source-directory' directives to do
   # so.
   # PVE will preserve these directives, but will NOT read its network
   # configuration from sourced files, so do not attempt to move any of
   # the PVE managed interfaces into external files!
   
   auto lo
   iface lo inet loopback
   
   auto enp1s0
   iface enp1s0 inet static
       address 192.168.31.3/24
   
   auto enp2s0
   iface enp2s0 inet manual
   
   auto vmbr1
   iface vmbr1 inet dhcp
       bridge-ports enp2s0
       bridge-stp off
       bridge-fd 0
   
   # source /etc/network/interfaces.d/*
   
   
   
   
   
   systemctl restart networking
   编辑/etc/sysctl.conf   取消注释net.ipv4.ip_forward=1
   iptables -t nat -A POSTROUTING -s 192.168.31.0/24 -o vmbr1 -j MASQUERADE
   ```

8. go环境变量

   ```
   rm -rf /usr/local/go && tar -C /usr/local -xzf go1.25.4.linux-amd64.tar.gz
    
   export PATH=$PATH:/usr/local/go/bin
    
   go version
   ```

9. 3x-ui

   ```
   podman run -d \
     --name 3xui_app \
     -v /root/3xui/db:/etc/x-ui:Z \
     -v /root/3xui/cert:/root/cert:Z \
     -e XRAY_VMESS_AEAD_FORCED=false \
     -e XUI_ENABLE_FAIL2BAN=true \
     --network host \
     ghcr.io/mhsanaei/3x-ui:latest
   ```

10. openresty

    ```
    sudo apt-get update
    sudo apt-get install -y wget gnupg2 software-properties-common
    
    wget -qO - https://openresty.org/package/pubkey.gpg | sudo gpg --dearmor -o /usr/share/keyrings/openresty.gpg
    
    echo "deb [signed-by=/usr/share/keyrings/openresty.gpg] http://openresty.org/package/debian $(lsb_release -sc) openresty" | sudo tee /etc/apt/sources.list.d/openresty.list
    
    sudo apt-get update
    sudo apt-get install -y openresty
    
    
    systemctl status openresty
    
    sudo systemctl start openresty
    sudo systemctl stop openresty
    sudo openresty -t
    sudo openresty -s reload
    
    
    sudo apt install ufw
    sudo ufw allow ssh
    sudo ufw enable
    
    
    # 放行 80 端口 (HTTP)
    sudo ufw allow 80/tcp
    
    # 放行 443 端口 (HTTPS)
    sudo ufw allow 443/tcp
    ```

11. postgresql

    ```
    podman pull docker.io/postgres:16
    
    podman run -d \
      --name xxx-postgres \
      --restart=always \
      -p 5432:5432 \
      -e POSTGRES_USER=xxx \
      -e POSTGRES_PASSWORD='xxx' \
      -e POSTGRES_DB=xxx \
      -e TZ=Asia/Shanghai \
      -e PGTZ=Asia/Shanghai \
      -v ~/data/postgresql:/var/lib/postgresql/data:Z \
      c6199d53bd7e
      
      podman exec -it xxx-postgres bash
      
      psql -U xxx -d xxxdb
      
      
      #备份到另一个容器
      podman run -d \
      --name xxx-postgres \
      --restart=always \
      -p 5432:5432 \
      -v ~/data/postgresql:/var/lib/postgresql/data:Z \
      docker.io/postgres:16
    ```

12. next