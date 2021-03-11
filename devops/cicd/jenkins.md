# Jenkins

## 项目地址

{% embed url="https://github.com/jenkinsci/jenkins" %}

## 快速开始

1、创建 Dockerfile，在 Jenkins 镜像中预装 Blue Ocean 插件（可选）

```text
mkdir /opt/jenkins && cd /opt/jenkins
```

{% code title="Dockerfile" %}
```text
FROM jenkins/jenkins:2.263.4-lts-jdk11
USER root
RUN apt-get update && apt-get install -y apt-transport-https \
       ca-certificates curl gnupg2 \
       software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN apt-key fingerprint 0EBFCD88
RUN add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/debian \
       $(lsb_release -cs) stable"
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins blueocean:1.24.4
```
{% endcode %}

2、构建 Jenkins 镜像（可选）

```text
docker build -t jenkins:2.263.4-lts-jdk11-blueocean .
```

3、创建 docker-compose.yml 文件

{% code title="docker-compose.yml" %}
```text
services:
  jenkins:
    # 如果无需预装 Blue Ocean 插件，可以直接使用官方镜像
    image: 'jenkins:2.263.4-lts-jdk11-blueocean'
    restart: always
    environment:
      DOCKER_HOST: 'tcp://docker:2376'
    ports:
      - '80:8080'
      - '50000:50000'
    volumes:
      - '/data/jenkins/certs:/certs/client:ro'
      - '/data/jenkins/home:/var/jenkins_home'
    links:
      - "dind:docker"
  dind:
    image: 'docker:dind'
    restart: always
    environment:
      DOCKER_TLS_CERTDIR: '/certs'
    ports:
      - '2376:2376'
    volumes:
      - '/data/jenkins/certs:/certs/client'
      - '/data/jenkins/home:/var/jenkins_home'
    privileged: true
```
{% endcode %}

4、创建 Jenkins 数据目录并授权

```text
mkdir -p /opt/jenkins/home
chown 1000:1000 /opt/jenkins/home
```

5、启动 Jenkins

```text
docker-compose up -d
```

6、获取 Jenkins 管理员初始密码

```text
cat /data/jenkins/home/secrets/initialAdminPassword
```

7、浏览器访问 Jenkins 地址 http://&lt;ip&gt;:80，使用管理员初始密码登录

