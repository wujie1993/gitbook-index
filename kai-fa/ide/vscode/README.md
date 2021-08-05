# VSCode

## 下载地址

{% embed url="https://code.visualstudio.com/download" %}

## 推荐插件

* Go。Go语言插件
* SFTP。用于将代码通过sftp快速同步到服务器端，通过服务器端进行编译构建
* Markdown All in One。Markdown编写快捷键，自动生成内容目录
* YAML。yaml语法支持，内置kubernetes yaml文件语法联想支持
* Docker。dockerfile语法高亮

{% hint style="info" %}
安装go语言插件的时候需要翻墙，否则会有部分包无法下载。
{% endhint %}

在无法翻墙的情况下，可以到github上搜索到无法下载的包的镜像同步项目，将项目下载到对应的本地路径上，如：

```bash
git clone git@github.com:golang/tools.git $GOPATH/src/golang.org/x/tools
```

## code-server

可通过浏览器端使用的 VSCode 开发环境

### 项目地址

{% embed url="https://github.com/cdr/code-server" %}

### 快速开始

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

