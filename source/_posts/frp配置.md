---
title: frp配置
tags: linux
abbrlink: 50bb2937
date: 2024-04-04 15:03:13
---

# frp 简介

frp 是一款高性能的反向代理应用，专注于内网穿透。它支持多种协议，包括 TCP、UDP、HTTP、HTTPS 等，并且具备 P2P 通信功能。使用 frp，您可以安全、便捷地将内网服务暴露到公网，通过拥有公网 IP 的节点进行中转。

![1.jpg](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/231011/1.jpg)

项目地址 https://github.com/fatedier/frp

---

# 环境

frp：`frp_0.51.3_linux_amd64`
VFS：`CentOS 7.6 64bit`

```shell
$ tree frp_0.51.3_linux_amd64
frp_0.51.3_linux_amd64
├── frpc                # 客户端
├── frpc_full.ini       # 客户端配置文件模板
├── frpc.ini
├── frps                # 服务器
├── frps_full.ini       # 服务器配置文件模板
├── frps.ini
└── LICENSE
```

>`frp` 从 `0.51.3` 版本之后，配置文件使用了 `.toml`，防止 dorker 镜像版本太低，这里 `frp` 使用 `0.51.3`

---
# 配置

## 服务端


服务端拥有公网IP，为内网服务做中转。

`frps.ini`
```shell
[common]
# 服务端与客户端通信的端口
bind_port = 7000 

# frps 后台仪表盘端口 账号
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin

# console or real logFile path like ./frps.log
log_file = ./frps.log
# trace, debug, info, warn, error
log_level = info
log_max_days = 3

# auth token
token = 12345678
```

---

## 客户端

`frpc.ini`
```shell
[common]
# 服务端 ip 与 端口
server_addr = xxx.xxx.xxx.xxx
server_port = 7000

# console or real logFile path like ./frpc.log
log_file = ./frpc.log
# trace, debug, info, warn, error
log_level = info
log_max_days = 3

# auth token
token = 12345678

[ssh]
# 需要服务端中转的内网服务
type = tcp
# 服务在本地的 ip 和端口
local_ip = 127.0.0.1
local_port = 22

# remote port listen by frps
# 通过服务端公网 ip 与这个端口访问内网服务
remote_port = 6000
```


---

# VFS配置

## 防火墙

![2.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/231011/2.png)

## 服务开机启动

创建一个启动脚本

`frps.service`
```bash
[Unit]
Description=frps
After=network.target
 
[Service]
TimeoutStartSec=30
ExecStart=/usr/local/bin/frps -c /etc/frp/frps.ini
ExecStop=/bin/kill $MAINPID
 
[Install]
WantedBy=multi-user.target
```

使用软连接配置到相关目录

```shell
vim /home/evan/frp/frps.service
sudo ln -s /home/evan/frp/frps.service /usr/lib/systemd/system/frps.service
sudo ln -s /home/evan/frp/frp_0.51.3_linux_amd64/frps /usr/local/bin/frps
sudo mkdir -p /etc/frp
sudo ln -s /home/evan/frp/frp_0.51.3_linux_amd64/frps.ini /etc/frp/frps.ini
```

```shell
# 启动 frp 并设置开机启动 
systemctl enable frps
systemctl start frps
systemctl status frps
```

## 客户端开机启动


`frpc.service`
```bash
[Unit]
Description=frpc
After=network.target
 
[Service]
TimeoutStartSec=30
ExecStart=/usr/local/bin/frpc -c /etc/frp/frpc.ini
ExecStop=/bin/kill $MAINPID
 
[Install]
WantedBy=multi-user.target
```

```shell
vim /home/evan/frp/frpc.service
sudo ln -s /home/evan/frp/frpc.service /usr/lib/systemd/system/frpc.service
sudo ln -s /home/evan/frp/frp_0.51.3_linux_amd64/frpc /usr/local/bin/frpc
sudo mkdir -p /etc/frp
sudo ln -s /home/evan/frp/frp_0.51.3_linux_amd64/frpc.ini /etc/frp/frpc.ini
```

```shell
# 启动 frp 并设置开机启动 
systemctl enable frps
systemctl start frps
systemctl status frps
```


---

# 帮助信息

```shell
$ ./frpc -h
frpc is the client of frp (https://github.com/fatedier/frp)

Usage:
  frpc [flags]
  frpc [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  http        Run frpc with a single http proxy
  https       Run frpc with a single https proxy
  nathole     Actions about nathole
  reload      Hot-Reload frpc configuration
  status      Overview of all proxies status
  stcp        Run frpc with a single stcp proxy
  stop        Stop the running frpc
  sudp        Run frpc with a single sudp proxy
  tcp         Run frpc with a single tcp proxy
  tcpmux      Run frpc with a single tcpmux proxy
  udp         Run frpc with a single udp proxy
  verify      Verify that the configures is valid
  xtcp        Run frpc with a single xtcp proxy

Flags:
  -c, --config string       config file of frpc (default "./frpc.ini")
      --config_dir string   config directory, run one frpc service for each file in config directory
  -h, --help                help for frpc
  -v, --version             version of frpc

Use "frpc [command] --help" for more information about a command.
```

```shell
$ ./frps --help
frps is the server of frp (https://github.com/fatedier/frp)

Usage:
  frps [flags]
  frps [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  help        Help about any command
  verify      Verify that the configures is valid

Flags:
      --allow_ports string               allow ports
      --bind_addr string                 bind address (default "0.0.0.0")
  -p, --bind_port int                    bind port (default 7000)
  -c, --config string                    config file of frps
      --dashboard_addr string            dashboard address (default "0.0.0.0")
      --dashboard_port int               dashboard port
      --dashboard_pwd string             dashboard password (default "admin")
      --dashboard_tls_cert_file string   dashboard tls cert file
      --dashboard_tls_key_file string    dashboard tls key file
      --dashboard_tls_mode               dashboard tls mode
      --dashboard_user string            dashboard user (default "admin")
      --disable_log_color                disable log color in console
      --enable_prometheus                enable prometheus dashboard
  -h, --help                             help for frps
      --kcp_bind_port int                kcp bind udp port
      --log_file string                  log file (default "console")
      --log_level string                 log level (default "info")
      --log_max_days int                 log max days (default 3)
      --max_ports_per_client int         max ports per client
      --proxy_bind_addr string           proxy bind address (default "0.0.0.0")
      --subdomain_host string            subdomain host
      --tls_only                         frps tls only
  -t, --token string                     auth token
  -v, --version                          version of frps
      --vhost_http_port int              vhost http port
      --vhost_http_timeout int           vhost http response header timeout (default 60)
      --vhost_https_port int             vhost https port

Use "frps [command] --help" for more information about a command.
```

