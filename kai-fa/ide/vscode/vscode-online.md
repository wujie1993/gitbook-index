# code-server

基于vscode的补丁增强

## 项目地址

{% embed url="https://github.com/cdr/code-server" %}

## 快速开始

### Docker方式

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

### 二进制方式

1、在服务器上运行安装脚本

```text
curl -fsSL https://code-server.dev/install.sh | sh
```

可附加参数 `-s -- --dry-run` 看安装过程中会运行的指令而不实际安装

2、修改 ~/.config/code-server/config.yaml 文件中的服务监听地址，也可修改浏览器端访问时所需的密码

```text
bind-addr: 0.0.0.0:8080
password: ******
```

3、配置 ~/.local/share/code-server/User/settings.json 文件

```text
{
    "git.path": "/usr/local/git/bin/git",
    "terminal.integrated.shell.linux": "/bin/bash",
    "terminal.integrated.env.linux": {
        "PATH": "$PATH:/usr/bin:/usr/local/go/bin:/root/go/bin:/usr/local/git/bin",
        "GOPATH": "/root/go",
    }
}
```

* git.path设置git二进制文件路径，使编辑器的git插件生效
* terminal.integrated.shell.linux设置终端所使用的命令行工具
* terminal.integrated.env.linux设置编辑器启动时加载的环境变量，使用systemd方式启动的code-server不会加载profile中的环境变量，因此需要在此添加

4、设置服务开机自启动

```text
systemctl enable code-server@$User
```

5、启动服务，通过浏览器访问 `http://<ip>:8080/` 打开编辑器，输入步骤 2 中配置的密码

```text
systemctl start code-server@$User
```

## 在线试用

{% embed url="https://cloudstudio.net/" %}



