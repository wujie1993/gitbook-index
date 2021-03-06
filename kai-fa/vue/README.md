# Vue

## 官方网站

{% embed url="https://cn.vuejs.org/" %}

## 项目地址

{% embed url="https://github.com/vuejs/vue" %}

## 快速开始

### 安装nodejs

{% embed url="https://nodejs.org/en/download/" %}

1. 下载nodejs二进制压缩包

```text
wget https://nodejs.org/dist/v14.15.0/node-v14.15.0-linux-x64.tar.xz
```

2. 解压至安装目录

```text
sudo mkdir -p /usr/local/lib/nodejs
sudo tar -xJvf node-v14.15.0-linux-x64.tar.xz -C /usr/local/lib/nodejs
```

3. 在/etc/profile中添加环境变量初始化

```text
export PATH=$PATH:/usr/local/lib/nodejs/node-v14.15.0-linux-x64/bin
```

4. 更新环境变量

```text
source /etc/profile
```

5. 查看nodejs与npm版本

```text
node -v
npm -v
```

6. 配置国内npm源

```text
npm config set registry http://registry.npm.taobao.org
```

### 安装vue-cli

```text
npm install -g @vue/cli
```

