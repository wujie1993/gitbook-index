# code-server

基于vscode的补丁增强

## 项目地址

{% embed url="https://github.com/cdr/code-server" %}

## 快速开始

1.在宿主机创建配置目录，用于挂载与快速修改vscode的配置文件

```text
mkdir -p ~/.config
```

2.运行vscode容器服务

```text
docker run -d --rm --name code-server -p 8080:8080 \
   -v "$PWD/.config:$HOME/.config" \
   -v "$PWD/project:/home/coder/project" \
   -u "$(id -u):$(id -g)" \
   -e "DOCKER_USER=$USER" \
   codercom/code-server:latest
```

3.浏览器访问http://&lt;ip&gt;:8080,密码可通过~/.config/code-server/config.yaml文件获取

## 在线试用

{% embed url="https://cloudstudio.net/" %}



