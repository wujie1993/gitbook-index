# Nexus3

### 发布jar包到maven仓库

1、配置setting.xml文件，在&lt;servers&gt;&lt;/servers&gt;块中添加如下内容：

```text
<server>
    <id>maven-releases</id>
    <username>账号</username>
    <password>密码</password>
</server>
```

2、使用 mvn deploy 命令发布 jar 包与 pom.xml 依赖描述文件

```text
mvn deploy:deploy-file -DgroupId=<groupId> -DartifactId=<artifactId> -Dversion=<version> -Dpackaging=jar -Dfile=<jar包路径>  -DrepositoryId=maven-releases  -Durl=<nexus3 URL>/repository/maven-releases/ -DgeneratePom=true -DpomFile=<pom.xml路径>
```

{% hint style="info" %}
如果不指定上传 pom.xml 文件，会自动生成一个空的 pom.xml，导致依赖包信息丢失
{% endhint %}

