## MAVEN
> 项目构建工具

### 使用
* settings配置本地仓库和远程仓库
* 配置环境变量
* idea中配置maven
### bug
#### 导入依赖报错
1. 切换镜像
```xml
<mirror>	
    <id>nexus-aliyun</id>	
    <mirrorOf>*</mirrorOf>	
    <name>Nexus aliyun</name>	
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>	
</mirror>

<mirror>
    <id>ibiblio</id>
    <name>Mirror from Maven ibiblio</name>
    <url>http://mirrors.ibiblio.org/pub/mirrors/maven2/</url>
    <mirrorOf>central</mirrorOf>
</mirror>

<mirror>
    <id>huaweicloud</id>
    <name>mirror from maven huaweicloud</name>
    <url>https://mirror.huaweicloud.com/repository/maven/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
<!-- 需要 配置一个server子节点 -->
<server>
    <id>huaweicloud</id>
    <username>anonymous</username>
    <password>devcloud</password>
</server>

<mirror>
    <id>repo2</id>
    <name>Mirror from Maven Repo2</name>
    <url>http://repo2.maven.org/maven2/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```
2. 删除*.lastUpdated文件
3. Reimport All Maven Projects
