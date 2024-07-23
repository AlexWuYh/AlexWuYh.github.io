# netmarker安装部署


## Netmaker 介绍

Netmaker 是一个用来配置 WireGuard 全互联模式的可视化工具，它的功能非常强大，不仅支持 UDP 打洞、NAT 穿透、多租户，还可以使用 Kubernetes 配置清单来部署，客户端几乎适配了所有平台，包括 Linux, Mac 和 Windows，还可以通过 WireGuard 原生客户端连接 iPhone 和 Android。

其最新版本的基准测试结果显示，基于 Netmaker 的 WireGuard 网络速度比其他全互联模式的 VPN（例如 Tailscale 和 ZeroTier）网络速度快 50% 以上。

## Netmaker 架构

​![](https://imgup.oneone.life/app/hide.php?key=aGlLNHFKR1B3THR1V1Jralp3L0s1bFVGWTdENndCb2hoSVU9)​

Netmaker 使用的是 C/S 架构，即客户端/服务器架构。Netmaker Server 包含两个核心组件：用来管理网络的可视化界面，以及与客户端通信的 gRPC Server。你也可以可以选择部署DNS服务器（CoreDNS）来管理私有DNS。

客户端（netclient）是一个二进制文件，可以在绝大多数 Linux 客户端以及 macOS 和 Windows 客户端运行，它的功能就是自动管理 WireGuard，动态更新 Peer 的配置。

> 注意：Netmaker Server 只是用来存储虚拟网络的配置并管理各个 Peer 的状态，Peer 之间的网络流量并不会通过 Netmaker Server。

Netmaker 还有一个重要的术语叫`签到`，客户端会通过定时任务来不断向 Netmaker Server 签到，以动态更新自身的状态和 Peer 的配置，它会从 Netmaker Server 检索 Peer 列表，然后与所有的 Peer 建立点对点连接，即全互联模式。所有的 Peer 通过互联最终呈现出来的网络拓扑结构就类似于本地子网或 VPC。

## Netmarker 部署

Netmaker 支持多种部署方式，包括二进制部署和容器化部署，容器化部署还支持 docker-compose 和 Kubernetes。个人推荐使用docker-compose来部署，简单易维护。

主要的部署步骤都是参考的[官方部署文档](https://docs.netmaker.org/quick-start.html)，因为官方文档使用的是`traefik`来做的反向代理和负载，但是我个人的服务器上的其他服务已经使用了`Caddy`来做代理，所以此处就修改了下YAML文件，继续使用Caddy做代理。

### 1. 先决条件

![](https://imgup.oneone.life/app/hide.php?key=V3hRRHgyaW9OOUZlQnpFU01NdnB5bFVGWTdENndCb2hoSVU9)

主要的点：

- 开放服务器的443(tcp), 53(tcp & udp), 51821-518XX(udp)端口
  - 443 端口，Dashboard，REST API 和 gRPC
  - 53端口，CoreDNS
  - 51821-518XX，WireGuard，每一个网络需要一个端口，起始端口会使用 51821，可以根据自己的网络端数量需要设定端口范围
- 如果有防火墙的话，需要开启防火墙的允许访问
  ```bash
  sudo ufw allow proto tcp from any to any port 443 && sudo ufw allow 51821:51830/udp

  iptables --policy FORWARD ACCEPT
  ```
- 如果是使用域名模式的话，还需要增加3条域名解析（如果只用IP访问，可以在配置文件中修改网络模式为：host，此处不做详细配置说明）
  - dashboard.domain
  - api.domain
  - broker.domain

### 2. 依赖安装

分别安装 `docker` 、`docker-compose`、`wireguard`，如果已经安装则略过

```bash
sudo apt-get isntall -y docker.io docker-compose wireguard
```

### 3. 安装 Netmarker

#### a. 从官方获取docker-compose的YAML文件，并修改其中的参数

```bash
# 获取docker-compose文件
wget -O docker-compose.yml https://raw.githubusercontent.com/gravitl/netmaker/master/compose/docker-compose.traefik.yml

# 修改域名
sed -i 's/NETMAKER_BASE_DOMAIN/<your base domain>/g' docker-compose.yml

# 修改公网IP
sed -i 's/SERVER_PUBLIC_IP/<your server ip>/g' docker-compose.yml

# 修改邮件地址
sed -i 's/YOUR_EMAIL/<your email>/g' docker-compose.yml

```

#### b. 生成唯一的master key并写入yaml文件

```bash
tr -dc A-Za-z0-9 </dev/urandom | head -c 30 ; echo ''

sed -i 's/REPLACE_MASTER_KEY/<your generated key>/g' docker-compose.yml
```

#### c. 拉取MQ配置文件

```bash
#从官方获取MQ配置文件，并存放于/root/目录
wget -O /root/mosquitto.conf https://raw.githubusercontent.com/gravitl/netmaker/master/docker/mosquitto.conf
```

#### d. 增加`Caddy`的配置

在`Caddyfile`文件末尾增加netmarker的代理配置，然后并重启Caddy服务：

```bash
## netmarker conf
## 替换下方的`NETMAKER_BASE_DOMAIN`和`YOUR_EMAIL`为你的真实信息
# Dashboard
dashboard.NETMAKER_BASE_DOMAIN {
        # Apply basic security headers
        header {
                # Enable HTTP Strict Transport Security (HSTS)
                Strict-Transport-Security "max-age=31536000;"
                # Enable cross-site filter (XSS) and tell browser to block detected attacks
                X-XSS-Protection "1; mode=block"
                # Disallow the site to be rendered within a frame on a foreign domain (clickjacking protection)
                X-Frame-Options "SAMEORIGIN"
                # Prevent search engines from indexing
                X-Robots-Tag "none"
                # Remove the server name
                -Server
        }
        encode gzip
        #root * /usr/share/caddy
        tls YOUR_EMAIL
        reverse_proxy 127.0.0.1:8888
}

# API
api.NETMAKER_BASE_DOMAIN {
        encode gzip
        root * /usr/share/caddy
        tls YOUR_EMAIL
        reverse_proxy 127.0.0.1:8081
}
```

#### e. 启动服务

在`docker-compose.yml`文件目录执行下面的命令启动服务

```bash
sudo docker-compose up -d
```

然后浏览器访问 dashboard.example.com，即可打开netmarker页面，第一次登录会提示创建用户

​![](https://imgup.oneone.life/app/hide.php?key=Qmhid1VJNXQxcEl5U0VPbXVQWnV1SFA1RGlydkNTOVIxc0k9)​

#### 最后贴上我自己的`docker-compose.yml`文件内容：

```yaml

version: "3.4"

services:
  netmaker:
    container_name: netmaker
    image: gravitl/netmaker:v0.14.3
    cap_add:
      - NET_ADMIN
      - NET_RAW
      - SYS_MODULE
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv6.conf.all.disable_ipv6=0
      - net.ipv6.conf.all.forwarding=1
    restart: always
    volumes:
      - dnsconfig:/root/config/dnsconfig
      - sqldata:/root/data
      - shared_certs:/etc/netmaker
    environment:
      SERVER_NAME: "broker.NETMAKER_BASE_DOMAIN"
      SERVER_HOST: "SERVER_PUBLIC_IP"
      SERVER_API_CONN_STRING: "api.nNETMAKER_BASE_DOMAIN:443"
      COREDNS_ADDR: "SERVER_PUBLIC_IP"
      DNS_MODE: "on"
      SERVER_HTTP_HOST: "api.NETMAKER_BASE_DOMAIN"
      API_PORT: "8081"
      CLIENT_MODE: "on"
      MASTER_KEY: "HsGWuuw5I48TEFp9RjBnlLRLT89lzk"
      CORS_ALLOWED_ORIGIN: "*"
      DISPLAY_KEYS: "on"
      DATABASE: "sqlite"
      NODE_ID: "netmaker-server-1"
      MQ_HOST: "mq"
      MQ_PORT: "443"
      HOST_NETWORK: "off"
      VERBOSITY: "1"
      MANAGE_IPTABLES: "on"
      PORT_FORWARD_SERVICES: "dns"
    ports:
      - "51821-51830:51821-51830/udp"
    expose:
      - "8081"
    ports: # 增加端口映射到容器外部，为了安全只开启本地访问
      - "127.0.0.1:8081:8081"
  netmaker-ui:
    container_name: netmaker-ui
    image: gravitl/netmaker-ui:v0.14.3
    depends_on:
      - netmaker
    links:
      - "netmaker:api"
    restart: always
    environment:
      BACKEND_URL: "https://api.NETMAKER_BASE_DOMAIN"
    expose:
      - "80"
    ports: # 增加端口映射到容器外部，为了安全只开启本地访问
      - "127.0.0.1:8888:80"
  coredns:
    container_name: coredns
    image: coredns/coredns
    command: -conf /root/dnsconfig/Corefile
    depends_on:
      - netmaker
    restart: always
    volumes:
      - dnsconfig:/root/dnsconfig
  mq:
    container_name: mq
    image: eclipse-mosquitto:2.0.11-openssl
    depends_on:
      - netmaker
    restart: unless-stopped
    volumes:
      - /root/mosquitto.conf:/mosquitto/config/mosquitto.conf
      - mosquitto_data:/mosquitto/data
      - mosquitto_logs:/mosquitto/log
      - shared_certs:/mosquitto/certs
    ports:
      - "127.0.0.1:1883:1883"
    expose:
      - "8883"
volumes:
  shared_certs: {}
  sqldata: {}
  dnsconfig: {}
  mosquitto_data: {}
  mosquitto_logs: {}
```

主要修改的内容：

- 删除`traefik`相关配置
- 增加`api`、`netmaker`容器的端口映射

