## MAVEN
> 项目构建工具

## 为什么使用
* 简化项目构建，不用再手动寻找jar包，配置jar包
* 解决项目中的依赖冲突问题

### 依赖冲突？
#### 产生原因
1. 项目中的依赖A和依赖B同时引入了依赖C
2. 依赖C在A和B中的版本不一致就可能产生依赖冲突
3. maven如果选择高版本C（1.1）来导入（这个选择 maven会根据不等路径短路径原则和同等路径第一声明原则选取），C（1.）中的类c在C（1.1）中被修改而不存在了。
4. 在编译期可能并不会报错，因为编译的目的只是把业务源代码编译成class文件，所以如果项目源代码中没有引入共有依赖C因升级而缺失的类c，就不会岀现编译失败。除非源代码就引入了共有依赖C，因升级而缺失的类C则会直接编译失败。
5. 在运行期，很有可能出现依赖A在执行过程中调用C（1.8）以前有但是升级到c（1.1）就缺失的类C，导致运行期失败，出现很典型的依赖冲突时的`NoClassDefFoundError`错误。
6. 如果是升级后出现原有的方法被修改而不存在的情况时，就会抛岀`NoSuchMethdError`错误。


#### 解决方式
1. 找到可能冲突的依赖（maven helper）
2. 调整依赖致使版本统一
3. 使用exclusion标签排除依赖
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

2. 删除.lastUpdated文件
3. Reimport All Maven Projects
