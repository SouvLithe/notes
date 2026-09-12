# Web网站的开发模式：
![[Pasted image 20250831004019.png]]
# Web前端开发：
前端的代码是如何转换成用户眼中的网页的？
- 通过浏览器转化（解析和渲染）成用户看到的网页
- 浏览器中对代码进行解析渲染的部分，称为浏览器内核
Web标准
- [[html]]：负责网页的结构
- [[CSS]]：负责网页的表现
- [[JavaScript(JS)]]：负责网页的行为

# Web入门
## HTTP协议
1. 基于请求-响应模型的：一次请求对应一次响应
2. HTTP协议是无状态的协议：对于事务处理没有记忆能力。每次请求-响应都是独立的。
    - 缺点：多次请求间不能共享数据。
    - 优点：速度快
### 请求协议（数据格式）
![[Pasted image 20250905211620.png]]
请求头相关参数，如下：

| 字段名             | 描述                                                                                                |
| --------------- | ------------------------------------------------------------------------------------------------- |
| Host            | 请求的主机名                                                                                            |
| User-Agent      | 浏览器版本，例如Chrome浏览器的标识类似Mozilla/5.0 ... Chrome/79，IE浏览器的标识类似Mozilla/5.0 (Windows NT ...) like Gecko |
| Accept          | 表示浏览器能接收的资源类型，如`text/*`，`image/*`或者`*/*`表示所有；                                                     |
| Accept-Language | 表示浏览器偏好的语言，服务器可以据此返回不同语言的网页；                                                                      |
| Accept-Encoding | 表示浏览器可以支持的压缩类型，例如gzip, deflate等。                                                                  |
| Content-Type    | 请求主体的数据类型。                                                                                        |
| Content-Length  | 请求主体的大小（单位：字节）。                                                                                   |
请求方式-GET: 请求参数在请求行中，没有请求体，如：`/brand/findAll?name=OPPO&status=1`。GET请求大小是有限制的。
请求方式-POST: 请求参数在请求体中，POST请求大小是没有限制的。
### 响应协议（数据格式）
![[Pasted image 20250905212056.png]]
#### 响应行
```text
| 状态码 | 描述 |
| 1xx | 响应中-临时状态码，表示请求已经接收，告诉客户端应该继续请求或者如果它已经完成则忽略它。 （后面学习WebSocket的状态码就是1xx）
| 2xx | 成功-表示请求已经被成功接收，处理已完成。 
| 3xx | 重定向-重定向到其他地方；让客户端再发起一次请求以完成整个处理。 
| 4xx | 客户端错误-处理发生错误，责任在客户端。如：请求了不存在的资源、客户端未被授权、禁止访问等。 
| 5xx | 服务器错误-处理发生错误，责任在服务端。如：程序抛出异常等。 
```
具体的值去查一查[状态码大全](https://cloud.tencent.com/developer/chapter/13553)
#### 响应头

| 字段名              | 描述                                       |
| ---------------- | ---------------------------------------- |
| Content-Type     | 表示该响应内容的类型，例如text/html，application/json。 |
| Content-Length   | 表示该响应内容的长度（字节数）。                         |
| Content-Encoding | 表示该响应压缩算法，例如gzip。                        |
| Cache-Control    | 指示客户端应如何缓存，例如max-age=300表示可以最多缓存300秒。    |
| Set-Cookie       | 告诉浏览器为当前页面所在的域设置cookie。                  |
## Web服务器
Web服务器是一个软件程序，对HTTP协议的操作进行封装，使得程序员不必直接对协议进行操作，让Web开发更加便捷。主要功能是"提供网上信息浏览服务"。

用的是Tomcat进行讲解。
这边的tomcat只是做介绍用，springboot内置整合了tomcat
部署一个Web应用程序需要将程序复制到Tomcat软件目录下的，一个叫webapps的目录下。
启动后，在localhost：8080端口后加上所复制项目中html文件的路经就行（包含复制项目的层级）
nginx的使用场景是分发服务（代理），tomcat的使用场景是服务部署
### Tomcat
轻量级服务器，springboot集成这个东西。所以需要的话回来看吧。
概念：Tomcat是Apache软件基金会一个核心项目，是一个开源免费的轻量级Web服务器，支持Servlet/JSP少量JavaEE规范。
Tomcat也被称为Web容器、Servlet容器。Servlet程序需要依赖于Tomcat才能运行

`JavaEE：Java Enterprise Edition，Java企业版。指Java企业级开发的技术规范总和。包含13项技术规范：JDBC、JNDI、EJB、RMI、JSP、Servlet（现在常用的是其封装的高级框架）、XML、JMS、Java IDL、JTS、JTA、JavaMail、JAF`

起步依赖：利用[[Maven]]的传递特性，会将一个依赖下的所有依赖都打包下号。
# 请求响应
spingboot提供了DispatcherServlet，实现类Servlet接口（这个接口才可被tomcat识别）
![[Pasted image 20250905222428.png]]
前端控制器，是整体流程控制的中心，由它调用其他组件处理用户请求，降低耦合性
过程分析：
1. **浏览器发送请求**：浏览器通过HTTP协议向Web服务器发送请求。
2. **请求到达DispatcherServlet**：Web服务器（如Tomcat）接收到请求后，将请求封装为`HttpServletRequest`对象，并将其传递给`DispatcherServlet`。
3. **DispatcherServlet分发请求**：`DispatcherServlet`作为前端控制器，根据请求的URL或其他参数，决定将请求分发给哪个`Controller`（如`XxxController`）进行处理。
4. **Controller处理请求**：`DispatcherServlet`将`HttpServletRequest`对象传递给相应的`Controller`。`Controller`处理请求，执行业务逻辑，并可能与模型（Model）交互。
5. **Controller返回结果**：`Controller`处理完请求后，将结果（如视图名、数据模型等）返回给`DispatcherServlet`。
6. **DispatcherServlet封装响应**：`DispatcherServlet`根据`Controller`返回的结果，将数据封装到`HttpServletResponse`对象中。
7. **响应返回给浏览器**：`DispatcherServlet`将填充好的`HttpServletResponse`对象返回给Web服务器，Web服务器再将响应发送回浏览器。
8. **浏览器渲染页面**：浏览器接收到响应后，根据响应中的内容（如HTML、JSON等）渲染页面。
## 参数的请求和封装
都要满足前端的  请求参数名  与  后端的形参  中数组变量名相同
### 原始方式获取请求参数
● Controller方法形参中声明HttpServletRequest对象
● 调用对象的getParameter(参数名)
 现在大多已经不用这个了
### 简单参数：
要求：参数名与形参变量名相同，定义形参即可接收参数。
如果方法形参名称与请求参数名称不匹配，可以使用 @RequestParam 完成映射。
但要注意：
	 @RequestParam中的required属性默认为true，代表该请求参数必须传递，如果不传递将报错。如果该参数是可选的，可以将required属性设置为false。
###  实体参数
pojo包下是专门用来封装实体类的。要get和set加上一个toString方法。
将所有前端请求的参数封装到一个实体类中。
单层实体对象：
![[Pasted image 20250906084215.png]]
嵌套实体对象：
![[Pasted image 20250906084757.png]]
### 数组、集合参数
![[Pasted image 20250906085248.png]]
![[Pasted image 20250906085429.png]]
### 日期参数
红框格式要一样
![[Pasted image 20250906090017.png]]
### JSON参数
![[Pasted image 20250906090234.png]]
###  路径参数
![[Pasted image 20250906090805.png]]
## 响应数据
![[Pasted image 20250906091521.png]]
`@RestController`=`@Controller` + `@ResponseBody`
但这种写法会响应多种类型数据，导致代码冗长
统一响应结果：
![[Pasted image 20250906092228.png]]
# 分层解耦
内聚：软件中各个功能模块内部的功能联系。
耦合：衡量软件中各个层/模块之间的依赖、关联的程度。
软件设计的原则：高内聚低耦合。
## 三层架构：
![[Pasted image 20250906094949.png]]
![[Pasted image 20250906095239.png]]
类加载器的主要作用是  **动态加载类文件到Java虚拟机（JVM）中**，并将其转换为`java.lang.Class`的实例
## IOC&DI
为了解上面代码的耦合（controller需要service的对象，service需要Dao的对象），需要用到这两个。
![[Pasted image 20250906095846.png]]
### 入门
![[Pasted image 20250906100603.png]]
1. Service层及Dao层的实现类，交给IOC容器管理。（Component ）
2. 为Controller及Service注入运行时，依赖的对象。（Autowired）
@Component //将当前类交给IOC容器管理，成为IOC容器中的bean
@Autowired //运行时，IOC容器会提供该类型的bean对象，并赋值给该变量 - 依赖注入
两个都标注解Component会报错，因为只需要一个单一的 bean，多了系统无法分辨

### IOC详解
| 注解          | 说明              | 位置                          |
| ----------- | --------------- | --------------------------- |
| @Component  | 声明bean的基础注解     | 不属于以下三类时，用此注解               |
| @Controller | @Component的衍生注解 | 标注在控制器类上                    |
| @Service    | @Component的衍生注解 | 标注在业务类上                     |
| @Repository | @Component的衍生注解 | 标注在数据访问类上（由于与mybatis整合，用的少） |
`@RestController`=`@Controller` + `@ResponseBody`
Repository很少用，在讲解了Mybatis之后会用另外一个注解替代。

IEDA中view->tool windows->Endpoints，可以找到项目创建的bean对象（首字母小写的）
#### Bean组件扫描
- 前面声明bean的四大注解，要想生效，还需要被组件扫描注解@ComponentScan扫描。
- @ComponentScan注解虽然没有显式配置，但是实际上已经包含在了启动类声明注解@SpringBootApplication中，默认扫描的范围是启动类所在包及其子包。
### DI详解
![[Pasted image 20250906103733.png]]
@Autowired和@Resource同一级，区别如下：
	@Autowired默认是按照类型进行注入的。由Sping提供的
	@Resource默认是按照名称进行注入的。由jdk提供的。
# `Mybatis`
Mybatis是java用于操作数据库的技术，是一款优秀的持久层（dao）框架，用于简化JDBC的开发。
前身[[JDBC]]（数据库连接池技术[[JDBC#数据库连接池]]]）
如果mapper接口方法形参只有一个普通类型的参数，#{...} 里面的属性名可以随便写，如：#{id}、#{value}。
`@Mapper`：标识一个接口为MyBatis的Mapper接口。
```application.properties
#配置mybatis的日志，指定输出到控制台
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```
mybatis的配置信息不用背，利用IDEA的联想补全就行。例如：mybatis.log直接加Tab就可以。

## 数据封装：
- 实体类属性名 和 数据库表查询返回的字段名一致，mybatis会自动封装。
- 如果实体类属性名 和 数据库表查询返回的字段名不一致，不能自动封装。
![[Pasted image 20250906150845.png]]
 1. 起别名：在SQL语句中，对不一样的列名起别名，别名和实体类属性名一样。
 2. 手动结果映射：通过 @Results及@Result 进行手动结果映射。
 3. 改配置文件
```plaintext
#开启mybatis的驼峰命名自动映射开关 a_column -----> aColumn
mybatis.configuration.map-underscore-to-camel-case=true
```
## Lombok
Lombok是一个实用的java类库，能通过注解的形式自动生成构造器、getter/setter、equals、hashcode、toString等方法，并可以自动化生成日志变量，简化java开发、提高效率。
![[Pasted image 20250906141701.png]]
Lombok会在编译时，自动生成对应的java代码。我们使用lombok时，还需要安装一个lombok的插件(idea自带)。
`@Slf4j` 是 Lombok 库提供的一个注解，用于自动生成日志记录功能。
## 参数传递注意
![[Pasted image 20250906144829.png]]
![[Pasted image 20250906144517.png]]
```java
@Options(keyProperty = "id", useGeneratedKeys = true)
#会自动将生成的主键值，赋值给emp对象的id属性
```
## 条件查询
![[Pasted image 20250906152009.png]]
## XML映射文件
Mybatis配置
规范sql语句有两种方式：注解  和  XML配置文件
● XML映射文件的名称与Mapper接口名称一致，并且将XML映射文件和Mapper接口放置在相同包下（同包同名）。
● XML映射文件的namespace属性为Mapper接口全限定名一致。
● XML映射文件中sql语句的id与Mapper接口中的方法名一致，并保持返回类型一致。
![[Pasted image 20250906152422.png]]
如果你需要做一些很复杂的操作，最好用 XML 来映射语句。
选择何种方式来配置映射，以及认为是否应该要统一映射语句定义的形式，完全取决于你和你的团队。 换句话说，永远不要拘泥于一种方式，你可以很轻松的在基于注解和 XML 的语句映射方式间自由移植和切换。
## 动态SQL
随着用户的输入或外部条件的变化而变化的SQL语句，我们称为 动态SQL。
### if
![[Pasted image 20250906154727.png]]
where会自动删除多余的and
set标签会删除掉字段中间多余的逗号
### foreach
![[Pasted image 20250906160029.png]]
### `sql&include`
作用：提高代码的复用性。
![[Pasted image 20250906160307.png]]
# 异常处理
![[Pasted image 20250906161202.png]]`exception(与control同级的文件夹).GlobalExceptionHandler`
```java
/**
 * 全局异常处理器
 */
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class) //捕获所有异常
    public Result ex(Exception ex) {
        ex.printStackTrace();
        return Result.error("对不起，操作失败，请联系管理员");
    }
}
```
# spring框架核心
## 事务
注解:
	● 注解：@Transactional
	● 位置：业务（service）层的方法上、类上、接口上
	● 作用：将当前方法交给spring进行事务管理，方法执行前，开启事务；成功执行完毕，提交事务；出现异常，回滚事务
```yml
logging:
  level:
    org.springframework.jdbc.support.JdbcTransactionManager: debug
```
### 事务属性-回滚
`rollbackFor`
	● 默认情况下，只有出现 `RuntimeException` 才回滚异常。rollbackFor属性用于控制出现何种异常类型，回滚事务。
`@Transactional(rollbackFor = Exception.class)`
即，在  @Transactional  中加上参数  `rollbackFor = Exception.class`
### 事务属性-传播行为
● 事务传播行为：指的就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行事务控制。
![[Pasted image 20250906172231.png]]

| 属性值                | 含义                                 |
| ------------------ | ---------------------------------- |
| ***REQUIRED***     | 【默认值】需要事务，有则加入，无则创建新事务             |
| ***REQUIRES_NEW*** | 需要新事务，无论有无，总是创建新事务                 |
| SUPPORTS           | 支持事务，有则加入，无则在无事务状态中运行              |
| NOT_SUPPORTED      | 不支持事务，在无事务状态下运行,如果当前存在已有事务,则挂起当前事务 |
| MANDATORY          | 必须有事务，否则抛异常                        |
| NEVER              | 必须没事务，否则抛异常                        |
例如：`@Transactional(propagation = Propagation.REQUIRED)`
REQUIRES_NEW将当前事务挂起，新建并执行新事物
## AOP
具体概念看[[SSM#AOP]]这一小结 。
***需要对所有业务类中的增、删、改方法添加  统一功能  ，使用AOP技术最为方便***
`AOP：Aspect Oriented Programming`（面向切面编程、面向方面编程）

动态代理是面向切面编程最主流的实现。而SpringAOP是Spring框架的高级技术，旨在管理bean对象的过程中，主要通过底层的动态代理机制，对特定的方法进行编程。
### 快速入门
在pom.xml中导入AOP的依赖
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
编写AOP程序：
- 针对于特定方法根据业务需要进行编程,如针对系统运行时间：

```java
@Component
@Aspect    //表示这个是AOP类
public class TimeAspect {
	//指定特定方法范围，service后面的*依次是类名/接口名，方法名
	//com.itheima.service包名
	@Around("execution(* com.itheima.service.*.*(..))")

    public Object recordTime(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
	    //开始任务前时间
        long begin = System.currentTimeMillis();
        Object object = proceedingJoinPoint.proceed(); // 调用原始方法运行
        //结束任务后的时间
        long end = System.currentTimeMillis();
        Log.info(proceedingJoinPoint.getSignature()+"执行耗时："+end - begin);
        return object;
    }
}
```
`@Slf4j` 是 Lombok 库提供的一个注解，用于自动生成日志记录功能。
场景：
记录操作日志、权限控制、事务管理……
优势：
代码无侵入、 减少重复代码、 提高开发效率、维护方便
### AOP进阶
一旦执行AOP程序开发，最终执行的就不是目标对象，而是基于目标对象形成的代理对象。
#### 通知类型：
1. @Around：环绕通知，此注解标注的通知方法在目标方法前、后都被执行
2. @Before：前置通知，此注解标注的通知方法在目标方法前被执行
3. @After：后置通知，此注解标注的通知方法在目标方法后被执行，无论是否有异常都会执行
4. @AfterReturning：返回后通知，此注解标注的通知方法在目标方法后被执行，有异常不会执行
5. @AfterThrowing：异常后通知，此注解标注的通知方法发生异常后执行
#### 注意事项：
◆ @Around环绕通知需要自己调用 `ProceedingJoinPoint.proceed() `来让原始方法执行，其他通知不需要考虑目标方法执行
◆ @Around环绕通知方法的返回值，必须指定为Object，来接收原始方法的返回值。
#### pointcut：
![[Pasted image 20250906200349.png]]
#### 通知执行顺序
1. 不同切面类中，默认按照切面类的类名字母排序：
    - 目标方法前的通知方法：字母排名靠前的先执行
    - 目标方法后的通知方法：字母排名靠前的后执行
2. 用 @Order(数字) 加在切面类上来控制顺序
    - 目标方法前的通知方法：数字小的先执行
    - 目标方法后的通知方法：数字小的后执行
```java
@Component
@Aspect
@Slf4j
@Order(5)
public class TimeAspect {
    // 类的内容
}
```
仅供参考，以实际运行为准。
### 切入点表达式
- 切入点表达式：描述切入点方法的一种表达式
- 作用：主要用来决定项目中的哪些方法需要加入通知
#### execution
![[Pasted image 20250906203105.png]]
一般异常抛出给execution表达式中的方法抛出（向上抛出）。
可以使用通配符描述切入点：
![[Pasted image 20250906203714.png]]
注意事项
- 根据业务需要，可以使用 且（&&）、或（||）、非（!）来组合比较复杂的切入点表达式。
书写建议
- 所有业务  方法名  在 命名 时尽量规范  ，方便切入点表达式快速匹配。如：查询类方法都是 find 开头，更新类方法都是 update 开头。
- 描述切入点方法通常  基于接口描述，而不是直接描述实现类， 增强拓展性 。
- 在满足业务需要的前提下， 尽量缩小切入点的匹配范围 。如：包名匹配尽量不使用 ..，使用 * 匹配单个包。
####  annotation
![[Pasted image 20250906205148.png]]
1. `@Retention(RetentionPolicy.RUNTIME)`：这个元注解指定了自定义注解的保留策略。`RetentionPolicy.RUNTIME` 表示该注解在运行时仍然可用，即可以通过反射机制读取到该注解的信息。
2. `@Target(ElementType.METHOD)`：这个元注解指定了自定义注解可以应用的目标。`ElementType.METHOD` 表示该注解只能用于方法上。
### 连接点
- 在Spring中用JoinPoint抽象了连接点，用它可以获得方法执行时的相关信息，如目标类名、方法名、方法参数等。
    - 对于 @Around 通知，获取连接点信息只能使用 `ProceedingJoinPoint`
![[Pasted image 20250906211430.png]]
    - 对于其他四种通知，获取连接点信息只能使用 JoinPoint，它是 `ProceedingJoinPoint` 的父类型
![[Pasted image 20250906211459.png]]
获取当前登录用户
- 获取request对象，从请求头中获取到jwt令牌，解析令牌获取出当前用户的id。
# sprintboot原理
## 配置优先级
spingboot支持以下三种类型的配置文件：
`application.properties`、`application.yml`、`application.yaml`
优先级顺序：`properties>yml>yaml`
虽然springboot支持多种格式配置文件，但是在项目开发时，推荐统一使用一种格式的配置（yml是主流）。

了解就行：
命令行参数 > 系统属性
![[Pasted image 20250906221749.png]]
项目打包之后如何设置这两个属性：
![[Pasted image 20250906221952.png]]
注意事项
- Springboot项目进行打包时，需要引入插件 `spring-boot-maven-plugin`（基于官网骨架创建项目，会自动添加该插件）
总：
	命令行参数 > 系统属性>`properties>yml>yaml`
## Bean管理
### 获取Bean
默认情况下，Spring项目启动时，会把bean都创建好放在IOC容器中，如果想要主动获取这些bean，可以通过如下方式：
- 根据name获取bean：`Object getBean(String name)`
    
- 根据类型获取bean：`<T> T getBean(Class<T> requiredType)`
    
- 根据name获取bean（带类型转换）：`<T> T getBean(String name, Class<T> requiredType)`
![[Pasted image 20250906223116.png]]
ApplicationContext对象其实就是IOC容器类对象。

注意事项：
	上述所说的【Spring项目启动时，会把其中的bean都创建好】还会受到作用域及延迟初始化影响，这里主要针对于默认的单例非延迟加载的bean而言。
### Bean的作用域
| 作用域             | 说明                        |
| --------------- | ------------------------- |
| ***singleton*** | 容器内同名称的bean只有一个实例（单例）（默认） |
| ***prototype*** | 每次使用该bean时会创建新的实例（非单例）    |
| request         | 每个请求范围内会创建新的实例（web环境中，了解） |
| session         | 每个会话范围内会创建新的实例（web环境中，了解） |
| application     | 每个应用范围内会创建新的实例（web环境中，了解） |
容器内同名称的bean只有一个实例（单例），用一个物理地址，即操作的是一个。
![[Pasted image 20250906223319.png]]
注意事项
- 默认singleton的bean，在容器启动时被创建，可以使用@Lazy注解来延迟初始化（延迟到第一次使用时，可降低内存占用）。
    
- prototype的bean，每一次使用该bean的时候都会创建一个新的实例。
    
- 实际开发当中，绝大部分的Bean是单例的，也就是说绝大部分Bean不需要配置scope属性。
### 第三方Bean
- 如果要管理的bean对象来自于第三方（不是自定义的），是无法用@Component及衍生注解声明bean的，就需要用到@Bean注解。
- 若要管理的第三方bean对象，建议对这些bean进行集中分类配置，可以通过@Configuration注解声明一个配置类。
![[Pasted image 20250906224658.png]]
ApplicationContext对象其实就是IOC容器类对象。
`applicationContext.getBean()`;  从IOC容器中获取类
@Bean注释：
1. 将当前方法的返回值对象交给IOC容器管理，成为IOC容器bean 
2. 通过@Bean注解的name/value属性指定bean名称，如果未指定，默认是方法名
3. 如果第三方bean需要依赖其它bean对象，直接在bean定义方法中设置形参即可，容器会根据类型自动装配。
```warn
 @Component 及衍生注解 与 @Bean注解使用场景？
    - 项目中自定义的，使用@Component及其衍生注解
    - 项目中引入第三方的，使用@Bean注解
```
## Springboot原理分析
主要优势：
1. **起步依赖** ：依靠依赖传递。
2. **自动配置**
### 自动配置
定义注解声明Bean对象，有时不会生效。
SpingBootApplication有包扫描的作用，但其扫描范围只有当前包和子包

解决方法：
方案一：`@ComponentScan` 组件扫描
```Java
#一旦声明@ComponentScan这个注解，原来默认扫描的就会被覆盖掉
@ComponentScan({"com.example","com.itheima"})
@SpringBootApplication
public class SpringbootWebConfig2Application {
}
```
方案二：@Import 导入。使用@Import导入的类会被Spring加载到IOC容器中，导入形式主要有以下几种：
- 导入 普通类
- 导入 配置类
- 导入` ImportSelector` 接口实现类
```Java
@Import({TokenParser.class, HeaderConfig.class})
@SpringBootApplication
public class SpringbootWebConfig2Application {
}
```
第三方依赖jar包使用 @EnableXxxx注解（封装@Import注解）将最有可能使用的Bean和配置类指定出来。
### 回头一定要来看看这个自动配置原理分析
Day14-08.SpringBoot原理-自动配置-原理分析-源码跟踪
点开@SpringbootApplication可以看到：
![[Pasted image 20250907104042.png]]
@EnableAutoConfiguration：
![[Pasted image 20250907104255.png]]
并非所有的Bean都会注入到IOC容器当中。@ConditionalOnMissingBean管着的
#### @Conditional
- 作用：按照一定的条件进行判断，在满足给定条件后才会注册对应的bean对象到Spring IOC容器中。
- 位置：方法、类
- @Conditional 本身是一个父注解，派生出大量的子注解：
  1. @ConditionalOnClass：判断环境中是否有对应字节码文件，若有才注册bean到IOC容器。
  2. @ConditionalOnMissingBean：判断环境中没有对应的bean（类型或名称），才注册bean到IOC容器。<主要用于声明默认的Bean对象，若声明了用你的，否则使用我提供的默认的Bean对象>
  3. @ConditionalOnProperty：判断配置文件中有对应属性和值，才注册bean到IOC容器。
#### 自定义Starter
场景
- 在实际开发中，经常会定义一些公共组件，提供给各个项目团队使用。而在SpringBoot的项目中，一般会将这些公共组件封装为SpringBoot 的 starter。
![[Pasted image 20250907110353.png]]
步骤：
![[Pasted image 20250907110602.png]]
# Web小结
![[Pasted image 20250907112403.png]]
