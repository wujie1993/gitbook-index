# JumpServer

### 官方网站

{% embed url="https://www.jumpserver.org/" %}

### 项目地址

{% embed url="https://github.com/jumpserver/jumpserver" %}

### 快速开始

前置准备:

* docker
* docker-compose

1、创建工作目录

```text
mkdir /opt/jumpserver && cd /opt/jumpserver
```

2. 配置 docker-compose.yml 和环境变量

{% tabs %}
{% tab title="docker-compose.yml" %}
```text
version: '2.4'
services:
  mysql:
    image: mariadb:10
    container_name: jms_mysql
    restart: always
    tty: true
    environment:
      MYSQL_USER: $DB_USER
      MYSQL_PASSWORD: $DB_PASSWORD
      MYSQL_DATABASE: $DB_NAME
      MYSQL_ROOT_PASSWORD: jumpserver@123
    volumes:
      - /data/jumpserver/mysql-data:/var/lib/mysql
    networks:
      - net

  redis:
    image: redis:6
    container_name: jms_redis
    restart: always
    tty: true
    command:
      - sh
      - -c
      - redis-server --appendonly yes --requirepass "$REDIS_PASSWORD"
    volumes:
      - /data/jumpserver/redis-data:/var/lib/redis/
    networks:
      - net

  core:
    image: jumpserver/jms_core:${Version}
    container_name: jms_core
    restart: always
    tty: true
    command: start web
    environment:
      SECRET_KEY: $SECRET_KEY
      BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN
      DEBUG: $DEBUG
      LOG_LEVEL: $LOG_LEVEL
      DB_HOST: $DB_HOST
      DB_PORT: $DB_PORT
      DB_USER: $DB_USER
      DB_PASSWORD: $DB_PASSWORD
      DB_NAME: $DB_NAME
      REDIS_HOST: $REDIS_HOST
      REDIS_PORT: $REDIS_PORT
      REDIS_PASSWORD: $REDIS_PASSWORD
    healthcheck:
      test: "curl -f http://localhost:8080/api/health/"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 60s
    volumes:
      - /data/jumpserver/core-data:/opt/jumpserver/data
    networks:
      - net

  celery:
    image: jumpserver/jms_core:${Version}
    container_name: jms_celery
    restart: always
    tty: true
    command: start task
    environment:
      SECRET_KEY: $SECRET_KEY
      BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN
      DEBUG: $DEBUG
      LOG_LEVEL: $LOG_LEVEL
      DB_HOST: $DB_HOST
      DB_PORT: $DB_PORT
      DB_USER: $DB_USER
      DB_PASSWORD: $DB_PASSWORD
      DB_NAME: $DB_NAME
      REDIS_HOST: $REDIS_HOST
      REDIS_PORT: $REDIS_PORT
      REDIS_PASSWORD: $REDIS_PASSWORD
    depends_on:
      core:
        condition: service_healthy
    healthcheck:
      test: "/opt/py3/bin/python /opt/jumpserver/apps/manage.py check_celery"
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 30s
    volumes:
      - /data/jumpserver/core-data:/opt/jumpserver/data
    networks:
      - net

  koko:
    image: jumpserver/jms_koko:${Version}
    container_name: jms_koko
    restart: always
    privileged: true
    tty: true
    environment:
      CORE_HOST: http://core:8080
      BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN
      LOG_LEVEL: $LOG_LEVEL
    depends_on:
      core:
        condition: service_healthy
    healthcheck:
      test: "nc -z localhost 2222 && nc -z localhost 5000"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s
    volumes:
      - /data/jumpserver/koko-data:/opt/koko/data
    ports:
      - 2222:2222
    networks:
      - net

  lion:
    image: jumpserver/jms_lion:${Version}
    container_name: jms_lion
    restart: always
    tty: true
    environment:
      CORE_HOST: http://core:8080
      BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN
      LOG_LEVEL: $LOG_LEVEL
    depends_on:
      core:
        condition: service_healthy
    healthcheck:
      test: "/etc/init.d/guacd status && curl -f http://localhost:8081/lion/health/"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s
    volumes:
      - /data/jumpserver/lion-data:/opt/lion/data
    networks:
      - net

  nginx:
    image: jumpserver/jms_nginx:${Version}
    container_name: jms_nginx
    restart: always
    tty: true
    depends_on:
      core:
        condition: service_healthy
    healthcheck:
      test: "curl -f http://localhost"
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s
    volumes:
      - /data/jumpserver/core-data:/opt/jumpserver/data
    ports:
      - 80:80
    networks:
      - net

networks:
  net:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: $DOCKER_SUBNET

```
{% endtab %}

{% tab title=".env" %}
```
# 版本号可以自己根据项目的版本修改
Version=v2.11.3

# 构建参数, 支持 amd64/arm64
TARGETARCH=amd64

# Compose
COMPOSE_PROJECT_NAME=jms
COMPOSE_HTTP_TIMEOUT=3600
DOCKER_CLIENT_TIMEOUT=3600
DOCKER_SUBNET=172.16.238.0/24

# MySQL
DB_HOST=mysql
DB_PORT=3306
DB_USER=jumpserver
DB_PASSWORD=nu4x599Wq7u0Bn8EABh3J91G
DB_NAME=jumpserver

# Redis
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=8URXPL2x3HZMi7xoGTdk3Upj

# Core
SECRET_KEY=B3f2w8P2PfxIAS7s4URrD9YmSbtqX4vXdPUL217kL9XPUOWrmy
BOOTSTRAP_TOKEN=7Q11Vz6R2J6BLAdO
DEBUG=FALSE
LOG_LEVEL=ERROR

##
# SECRET_KEY 保护签名数据的密匙, 首次安装请一定要修改并牢记, 后续升级和迁移不可更改, 否则将导致加密的数据不可解密。
# BOOTSTRAP_TOKEN 为组件认证使用的密钥, 仅组件注册时使用。组件指 koko、guacamole

```
{% endtab %}
{% endtabs %}

2、启动 jumpserver

```text
docker-compose up -d
```

### 使用

浏览器访问 http://&lt;ip&gt;:80

SSH命令行访问 `ssh -p 2222 <ip>`

默认登录账密 admin/admin

