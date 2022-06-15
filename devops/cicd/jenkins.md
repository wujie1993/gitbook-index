# Jenkins

## 项目地址

{% embed url="https://github.com/jenkinsci/jenkins" %}

## 快速开始

1、创建 Dockerfile，在 Jenkins 镜像中预装 Blue Ocean 插件（可选）

```
mkdir /opt/jenkins && cd /opt/jenkins
```

{% code title="Dockerfile" %}
```
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

```
docker build -t jenkins:2.263.4-lts-jdk11-blueocean .
```

3、创建 docker-compose.yml 文件

{% code title="docker-compose.yml" %}
```
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

```
mkdir -p /opt/jenkins/home
chown 1000:1000 /opt/jenkins/home
```

5、启动 Jenkins

```
docker-compose up -d
```

6、获取 Jenkins 管理员初始密码

```
cat /data/jenkins/home/secrets/initialAdminPassword
```

7、浏览器访问 Jenkins 地址 http://\<ip>:80，使用管理员初始密码登录

### Q\&A

#### 如何配置gitlab webhook触发流水线执行

在 Jenkins 的插件管理中安装 Gitlab 插件，然后在流水线中勾选启用 GitLab webhook，记录下方的 URL，为了安全起见，展开高级选项并生成一串 token 密钥。

![](<../../.gitbook/assets/image (35).png>)

![](<../../.gitbook/assets/image (37).png>)

在 Gitlab 项目中进入 Setting -> Webhook，填入上方生成的 URL 与 Token 密钥，保存后即可发送测试

![](<../../.gitbook/assets/image (34).png>)

![](<../../.gitbook/assets/image (33).png>)

![](<../../.gitbook/assets/image (32).png>)

在 Webhook 条件触发时，可在 Jenkins 流水线上看到任务被调用执行

![](<../../.gitbook/assets/image (30).png>)

