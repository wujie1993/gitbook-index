# Java

## 安装

### mac

1、安装openjdk8

```text
brew tap AdoptOpenJDK/openjdk
brew install adoptopenjdk8
```

2、安装maven

```text
brew install maven
```

## Q&A

Q：拉取jar包时maven-default-http-blocker报错

A：编辑maven配置文件（可通过mvn -v获取安装目录），将下方部分注释

```text
<mirror>
    <id>maven-default-http-blocker</id>
    <mirrorOf>external:http:*</mirrorOf>
    <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
    <url>http://0.0.0.0/</url>
    <blocked>true</blocked>
</mirror>
```

