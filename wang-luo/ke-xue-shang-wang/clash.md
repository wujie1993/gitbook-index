# Clash

### 官方文档

{% embed url="https://docs.cfw.lbyczf.com/" %}

### 项目地址

{% embed url="https://github.com/Dreamacro/clash" %}

### 接口文档

{% embed url="https://clash.gitbook.io/doc/restful-api" %}

### 快速开始

安装前准备：docker与docker-compose

```text
mkdir /usr/local/clash && cd /usr/local/clash
```

下载代理商提供的配置文件，以文件名config.yaml保存，修改以下项

```text
# http代理端口
port: 7890
# socks5代理端口
socks-port: 7891
# 开启代理端口侦听
allow-lan: true
# api接口侦听地址与端口
external-controller: 0.0.0.0:8080
```

创建docker-compose.yml

```text
version: '3'
services:
  clash:
    image: dreamacro/clash
    volumes:
      - ./config.yaml:/root/.config/clash/config.yaml
      # dashboard volume
      # - ./ui:/ui
    ports:
      - "7890:7890"
      - "7891:7891"
      # If you need external controller, you can export this port.
      # - "8080:8080"
    restart: always
    # When your system is Linux, you can use `network_mode: "host"` directly.
    network_mode: "bridge"
    container_name: clash
```

启动clash

```text
docker-compose up -d
```

配置http代理

```text
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
```

或者配置socks5代理

```text
set http_proxy=socks5://127.0.0.1:7891
set https_proxy=socks5://127.0.0.1:7891
```

