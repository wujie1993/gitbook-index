# Vue

## 官方网站

{% embed url="https://cn.vuejs.org/" %}

## 项目地址

{% embed url="https://github.com/vuejs/vue" %}

## 快速开始

### 安装nodejs

{% embed url="https://nodejs.org/en/download/" %}

1\. 下载nodejs二进制压缩包

```
wget https://nodejs.org/dist/v14.15.0/node-v14.15.0-linux-x64.tar.xz
```

2\. 解压至安装目录

```
sudo mkdir -p /usr/local/lib/nodejs
sudo tar -xJvf node-v14.15.0-linux-x64.tar.xz -C /usr/local/lib/nodejs
```

3\. 在/etc/profile中添加环境变量初始化

```
export PATH=$PATH:/usr/local/lib/nodejs/node-v14.15.0-linux-x64/bin
```

4\. 更新环境变量

```
source /etc/profile
```

5\. 查看nodejs与npm版本

```
node -v
npm -v
```

6\. 配置国内npm源

```
npm config set registry http://registry.npm.taobao.org
```

### 安装vue-cli

```
npm install -g @vue/cli
```

## Q\&A

1、配置vue.config.js代理devserver.proxy结果返回404

{% embed url="https://juejin.cn/post/6844904122672480269" %}

2、数据更新页面内容没更新

{% embed url="https://segmentfault.com/a/1190000022772025" %}

3、执行 npm install 提示错误 gyp ERR! stack Error: EACCES: permission denied

npm install 命令添加参数 `--unsafe-perm=true --allow-root`
