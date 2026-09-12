[**【`GeekHour`】**](https://space.bilibili.com/102438649)一小时Maven教程
## Maven坐标
![[Pasted image 20250904174114.png]]

## Maven工程的目录结构
Maven的工程目录结构是有一定的规范的，这样可以方便Maven来自动的构建项目，下面是一个标准的Maven工程目录结构：
![[Pasted image 20250904171904.png]]
```bash
project                     # 项目根目录
|-- src                     # 源代码目录
|   |-- main                # 主目录
|   |   |-- java            # Java源代码目录
|   |   |-- resources       # 资源文件目录
|   |   |-- webapp          # Web应用目录
|   |-- test                # 测试目录
|       |-- java            # 测试源代码目录
|       |-- resources       # 测试资源文件目录
|-- target                  # 项目构建目录
|-- pom.xml                 # 项目配置文件
```
例如：

![](https://neucms.com/img/20240805231706.png)
## POM文件
POM文件是Maven项目的核心文件，它是一个XML文件，定义了项目的配置、依赖、插件以及构建的过程。

以下是一个简单的POM文件示例：

```xml
<project xmlns = "http://maven.apache.org/POM/4.0.0"
    xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <!-- 模型版本 -->
  <modelVersion>4.0.0</modelVersion>
  <!-- 公司或者组织的唯一标志，并且配置时生成的路径也是由此生成，
        如：com.companyname.project-group，
        maven会将该项目打成的jar包放本地路径：
        /com/companyname/project-group -->
  <groupId>com.companyname.project-group</groupId>

  <!-- 项目的唯一ID，一个groupId下面可能多个项目，就是靠artifactId来区分的 -->
  <artifactId>project</artifactId>

  <!-- 版本号 -->
  <version>1.0</version>

  <!-- 属性变量 -->
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <!-- 依赖 -->
  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>5.3.9</version>
    </dependency>
  </dependencies>

  <!-- 依赖管理 -->
  <dependencyManagement>
    <dependencies>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-core</artifactId>
          <version>5.3.9</version>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <!-- 仓库管理 -->
  <repositories>
    <repository>
        <id>central</id>
        <url>https://repo.maven.apache.org/maven2</url>
    </repository>
  </repositories>

  <!-- 构建 -->
  <build>
    <!-- 插件管理 -->
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>
```

## 构建生命周期
Maven提供了三种主要的生命周期：`Clean`、`Default`和`Site`
- clean：清理工作。
- default：核心工作，如：编译、测试、打包、安装、部署等。
- site：生成报告、发布站点等。
![[Pasted image 20250904180539.png]]

### `Clean`：用于项目清理（`mvn clean`）

执行`clean`生命周期，会删除`target`目录下的所有文件，包括编译后的字节码文件、打包后的jar包、生成的站点等等。

```bash
mvn clean
```

### `Default` ：用于项目部署
```
validate` => `compile` => `test` => `package` => `verify` => `install` => `deploy
```

| 阶段             | 处理   | 描述                            |
| -------------- | ---- | ----------------------------- |
| `mvn validate` | 验证项目 | 验证项目是否正确且所有必须信息是可用的           |
| `mvn compile`  | 执行编译 | 源代码编译在此阶段完成                   |
| `mvn test`     | 测试   | 使用适当的单元测试框架（例如JUnit）运行测试。     |
| `mvn package`  | 打包   | 将编译后的代码打包成可分发的格式，例如 JAR 或 WAR |
| `mvn verify`   | 检查   | 对集成测试的结果进行检查，以保证质量达标          |
| `mvn install`  | 安装   | 安装打包的项目到  本地仓库  ，以供其他项目使用     |
| `mvn deploy`   | 部署   | 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程  |

#### 打包细节
- Maven默认的`jar`打包方式不会将依赖的`jar`包包含在最终的`jar`文件中。
- 如果需要将依赖的`jar`包也打包进去，可以通过`maven-assembly-plugin`或`maven-shade-plugin`来实现。即在pom.xml文件中额外配置一个打包插件。

### Site：用于生成项目站点

用于生成项目站点，包括项目的文档、报告、API文档等等。

```bash
# 生成站点文档
mvn site
# 部署站点文档
mvn site:deploy
```

### 插件命令
Maven插件扩展了Maven的功能，可以用来完成一些特定的任务，
1. `mvn archetype:generate`：创建一个新的Maven项目，并生成项目骨架
2. `mvn dependency:tree`：查看项目依赖树
3. `mvn dependency:analyze`：分析项目依赖
4. `mvn dependency:resolve`：解析项目依赖
5. `mvn dependency:copy-dependencies`：复制项目依赖
6. `mvn versions:display-dependency-updates`：显示项目依赖的更新
## 依赖管理
### 依赖的范围
- `compile`：默认范围，编译、测试、运行时都有效
- `provided`：编译、测试有效，运行时无效，比如servlet-api
- `runtime`：测试、运行时有效，编译时无效
- `test`：测试时有效，编译、运行时无效
- `system`：类似`provided`，但是需要指定jar包的路径
- `import`：导入依赖的范围
### 依赖的传递性
Maven会自动的解决依赖的传递性，比如说A依赖B，B依赖C，那么Maven会自动的将C也导入到A中，这样就不需要我们手动的去导入C了。
只有当依赖的范围是`compile`或者`runtime`的时候，依赖才会被传递，如果依赖的范围是`provided`或者`test`的时候，依赖是不会被传递的。
![[Pasted image 20250904174610.png]]
### 依赖的排除
有时候我们引入的依赖包中可能会包含一些我们不需要的依赖，
这个时候我们可以使用`<exclusions>`标签来排除这些依赖。

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>6.1.11</version>
    <exclusions>
        <exclusion>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
        </exclusion>
    </exclusions>
    <-true为可选，false是关了的->
    <optional>flase</optional>
</dependency>
```
也可以通过`<optional>`标签来指定依赖是否可选，如果依赖是可选的，那么在引入这个依赖的时候，可以不用引入这个依赖的依赖。

### 依赖的版本冲突
当通过依赖传递导入的两个依赖包版本不一致时，Maven会根据一定的规则来解决这个冲突：
- 路径优先：当依赖中出现相同的资源时，层级越深，优先级越低，层级越浅，优先级越高
- 声明优先：当资源在相同层级被依赖时，配置顺序靠前的覆盖配置顺序靠后的
- 特殊优先：当同级配置了相同资源的不同版本，后配置的覆盖先配置的
### 依赖范围
![[Pasted image 20250904175204.png]]
### 上传jar包到私服仓库
执行一个`mvn deploy`命令，就可以将jar包上传到私服仓库中，
上传之后在Nexus的管理界面中就可以看到对应的jar包。