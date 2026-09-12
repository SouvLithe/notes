# 分模块设计与开发
将项目按照功能拆分成若干个子模块，方便项目的管理维护、扩展，也方便模块间的相互调用，资源共享。
![[Pasted image 20250907112709.png]]
## 实践构造图
![[Pasted image 20250907112952.png]]
**步骤 · 分模块开发**
- 创建maven模块 tlias-pojo，存放实体类。
- 创建maven模块 tlias-utils，存放相关工具类。
**注意事项**
- 分模块开发需要先针对模块功能进行设计，再进行编码。不会先将工程开发完毕，然后进行拆分。
# 继承与聚合
两者区别：
![[Pasted image 20250913155404.png]]
## 继承
概念：继承描述的是两个工程间的关系，与java中的继承相似，子工程可以继承父工程中的配置信息，常见于依赖关系的继承。
作用：简化依赖配置、统一管理依赖。
实现：`<parent> </parent>`
![[Pasted image 20250907114348.png]]
注意：
- maven和Java一样只能单继承（只认一个爹），但可以依靠多重继承实现。
### 继承关系实现:
①. 创建maven模块 tlias-parent，该工程为父工程，设置打包方式pom(默认jar)。
②. 在子工程的pom.xml文件中，配置继承关系。
③. 在父工程中配置各个工程共有的依赖（子工程会自动继承父工程的依赖）。

- jar：普通模块打包，springboot项目基本都是jar包（内嵌tomcat运行）
- war：普通web程序打包，需要部署在外部的tomcat服务器中运行
- pom：父工程或聚合工程，该模块不写代码，仅进行依赖管理
![[Pasted image 20250907114756.png]]
packaging设置父工程打包方式为pom。
注意事项：
- 在子工程中，配置了继承关系之后，坐标中的groupId是可以省略的，因为会自动继承父工程的。
- relativePath指定父工程的pom文件的相对位置（如果不指定，将从本地仓库/远程仓库查找该工程）。
- 若父子工程都配置了同一个依赖的不同版本，以子工程为标准
#### 项目结构
两种都可以，，建议右边结构清晰点。
![[Pasted image 20250907115717.png]]
### 版本锁定
- 在maven中，可以在父工程的pom文件中通`<dependencyManagement>`来统一管理依赖版本。
![[Pasted image 20250907120041.png]]
注意事项
- 子工程引入依赖时，无需指定` <version> `版本号，父工程统一管理。变更依赖版本，只需在父工程中统一变更。
#### 自定义属性
![[Pasted image 20250907120355.png]]
`<dependencyManagement>` 与 `<dependencies>`的区别是什么？
- `<dependencies>` 是直接依赖，在父工程配置了依赖，子工程会直接继承下来。
- `<dependencyManagement>` 是统一管理依赖版本，不会直接依赖，还需要在子工程中引入所需依赖(无需指定版本)。
#### 引用pom中的属性
这是[[SSM#Maven]]中讲到的，不主流但偶尔。
步骤：
1. 定义属性
![[Pasted image 20250913160248.png]]
2. 配置文件中引用属性
![[Pasted image 20250913160411.png]]
3. 开启资源文件目录加载属性的过滤器（使得可以在2中解析${}这个符号）
![[Pasted image 20250913160549.png]]
4. 配置maven打war包时，忽略web.xml检查
![[Pasted image 20250913160643.png]]
#### 其它属性
![[Pasted image 20250913160817.png]]
读取方式：
首先cd到maven安装路径下的bin目录中，输入以下指令
```shell
mvn help:system
```
## 聚合
所遇问题：
	分模块开发后，对某一个模块进行打包
需要将这个模块的父工程和这个模块所依赖的工程，要先按照顺序安装到maven的本地仓库，才可以对该模块打包
![[Pasted image 20250907141403.png]]
### 优化：
● maven中可以通过 `<modules> `设置当前聚合工程所包含的子模块名称
![[Pasted image 20250913154233.png]]
构建项目时会根据依赖关系依次构建，与`<module>`的次序无关
```xml
<!-- 聚合 -->
<modules>
    <module>../tlias-pojo</module>
    <module>../tlias-utils</module>
    <module>../tlias-web-management</module>
</modules>
```
注意事项
- 聚合工程中所包含的模块，在构建时，会自动根据模块间的依赖关系设置构建顺序，与聚合工程中模块的配置书写位置无关。

# 私服
私服是一种特殊的远程仓库，它是架设在局域网内的仓库服务，用来代理位于外部的中央仓库，用于解决团队内部的资源共享与资源同步问题。
![[Pasted image 20250907142004.png]]
依赖查找顺序：本地->私服->中央仓库
私服仓库分类：
![[Pasted image 20250913163642.png]]
## 资源的下载和上传
![[Pasted image 20250907142323.png]]
项目版本：
- RELEASE（发行版本）：功能趋于稳定、当前更新停止，可以用于发行的版本，存储在私服中的RELEASE仓库中。
- SNAPSHOT（快照版本）：功能不稳定、尚处于开发中的版本，即快照版本，存储在私服的SNAPSHOT仓库中。

在maven文件夹中找到中配置
1. 设置私服的访问用户名/密码（maven的settings.xml中的servers中配置）
```xml
<server>
    <id>maven-releases</id>
    <username>admin</username>
    <password>admin</password>
</server>
<server>
    <id>maven-snapshots</id>
    <username>admin</username>
    <password>admin</password>
</server>
```
2. 设置私服依赖下载的仓库组地址（settings.xml中的mirrors、profiles中配置）<如果配置过阿里云的地址，需要将其替换掉>
```xml
<mirror>
    <id>maven-public</id>
    <mirrorOf>*</mirrorOf>
    <url>http://192.168.150.101:8081/repository/maven-public/</url>
</mirror>
```
3. 需要在 profiles 中，增加如下配置，来指定snapshot快照版本的依赖，依然允许使用
```xml
<profile>
    <id>allow-snapshots</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <repositories>
        <repository>
            <id>maven-public</id>
            <url>http://192.168.150.101:8081/repository/maven-public/</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>
</profile>
```
4. IDEA的maven工程的pom文件中配置上传（发布）地址
```xml
<distributionManagement>
    <repository>
        <id>maven-releases</id>
        <url>http://192.168.150.101:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>maven-snapshots</id>
        <url>http://192.168.150.101:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```
上面配置好后，执行生命周期deploy就可以了。1、4的id要一致。