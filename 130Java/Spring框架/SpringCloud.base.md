# 微服务技术栈：
微服务 不等于 `SpringCloud`。
![[Pasted image 20251004114733.png]]
具体技术如下：
微服务技术、异步通信技术、缓存技术、搜索技术、DevOps。
![[Pasted image 20251004114925.png]]
# 认识微服务
## 服务架构演变
### 单体架构
单体架构：将业务的所有功能集中在一个项目中开发，打成一个包部署。
![[Pasted image 20251004161326.png]]
优点：
- 架构简单
- 部署成本低
缺点：耦合度高
### 分布式架构
分布式架构：根据业务功能对系统进行拆分，每个业务模块作为独立项目开发，称为一个服务。
![[Pasted image 20251004161503.png]]
优点：
- 降低服务耦合
- 有利于服务升级拓展

分布式架构的要考虑的问题：
- 服务拆分粒度如何？
- 服务集群地址如何维护？
- 服务之间如何实现远程调用？
- 服务健康状态如何感知？
### 微服务
微服务是一种经过良好架构设计的分布式架构方案，微服务架构特征：
- 单一职责：微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责，避免重复业务开发    (好处：每个模块业务更少了，影响范围更小了)
- 面向服务：微服务对外暴露业务接口
- 自治：团队独立、技术独立、数据独立、部署独立
- 隔离性强：服务调用做好隔离、容错、降级，避免出现  **级联问题**
实现高内聚低耦合，降低服务之间的影响或者说降低服务影响的范围，避免出现集群故障。
### 小结
单体架构特点？
- 简单方便，高度耦合，扩展性差，适合小型项目。例如：学生管理系统
分布式架构特点？
- 松耦合，扩展性好，但架构复杂，难度大。适合大型互联网项目，例如：京东、淘宝
微服务：一种良好的分布式架构方案
- 优点：拆分粒度更小、服务更独立、耦合度更低
- 缺点：架构非常复杂，运维、监控、部署难度提高
## 微服务技术对比
微服务这种方案需要技术框架来落地，全球的互联网公司都在积极尝试自己的微服务落地技术。在国内最知名的就是  `SpringCloud`  和  阿里巴巴的Dubbo。
![[Pasted image 20251004163055.png]]
`SpringCloudAlibaba`同时兼容前面两种的服务架构
### 企业要求
![[Pasted image 20251004163409.png]]
Dubbo原始模式：2012年老的技术体系（Dubbo+ZooKeeper）这套，由Dubbo原始模式升级为SpringCloudAlibaba + Dubbo这种升级代码是不用动的，只需要改外部组件。
## 关于Spring Cloud
SpringCloud是目前国内使用最广泛的微服务框架。官网地址：[spring-cloud](https://spring.io/projects/spring-cloud)。
SpringCloud集成了各种微服务功能组件，并基于SpringBoot实现了这些组件的自动装配，从而提供了良好的开箱即用体验：
![[Pasted image 20251004163905.png]]
使用SpringCloud时，要注意其与SpringBoot的版本兼容关系。
# 服务拆分及远程调用
## 服务拆分
服务拆分注意事项：
1. 单一职责：不同微服务，不要重复开发相同业务
2. 数据独立：不要访问其它微服务的数据库
3. 面向服务：将自己的业务暴露为接口，供其它微服务调用
![[Pasted image 20251004164615.png]]

这就要求做到：
1. 微服务需要根据业务模块拆分，做到单一职责,不要重复开发相同业务
2. 微服务可以将业务暴露为接口，供其它微服务使用
3. 不同微服务都应该有自己独立的数据库
## 远程调用
原理：在Java中创建Http请求给其它服务模块，类似于Ajax。

1. 注册RestTemplate
```java
@MapperScan("cn.itcast.order.mapper")  
@SpringBootApplication  
public class OrderApplication {    
	public static void main(String[] args) {  
        SpringApplication.run(OrderApplication.class, args);  
    }    
    @Bean    
    public RestTemplate restTemplate(){        
	    return new RestTemplate();  
    }  
}
```
Bean的注入只能放到配置类中，而启动类本质上也是配置类。
2. 服务远程调用RestTemplate
```java
@Service  
public class OrderService {       
	@Autowired    
	private RestTemplate restTemplate;    
	public Order queryOrderById(Long orderId) {        
		// 1.查询订单        
		Order order = orderMapper.findById(orderId);        
		// TODO 2.查询用户        
		String url = "http://localhost:8081/user/" +  order.getUserId();  
		User user = restTemplate.getForObject(url, User.class);        
		// 3.封装user信息        
		order.setUser(user);        
		// 4.返回        
		return order;  
    }  
}
```
想要发送Get请求，使用getForObject；想要发送Post请求，就使用postForObject。
里面的User.class是返回值类型，它知道返回的是JSON，但如果要的是User.class的对象类型，它会自动帮你把JSON反序列化为对象
```tips
1. 序列化（Serialization）：将对象转换成JSON字符串的过程。这通常意味着将一个对象的属性和值转换为一个JSON格式的字符串，以便可以存储或通过网络传输。
    
2. 反序列化（Deserialization）：将JSON字符串转换回对象的过程。这通常意味着将一个JSON格式的字符串解析并转换回一个对象，以便可以在程序中使用
```
微服务调用方式
- 基于RestTemplate发起的http请求实现远程调用
- http请求做远程调用是与语言无关的调用，只要知道对方的ip、端口、接口路径、请求参数即可。
### 服务调用关系
- 服务提供者：暴露接口给其它微服务调用
- 服务消费者：调用其它微服务提供的接口
提供者与消费者角色其实是相对的
如果不谈业务 , 一个服务可以同时是服务提供者和服务消费者。
# 注册中心
## Eureka注册中心
服务消费者该如何获取服务提供者的地址信息？
- 服务提供者启动时向eureka注册自己的信息
- eureka保存这些信息
- 消费者根据服务名称向eureka拉取提供者信息
如果有多个服务提供者，消费者该如何选择？
- 服务消费者利用负载均衡算法，从服务列表中挑选一个
消费者如何得知服务提供者的健康状态？
- 服务提供者会每隔30秒向EurekaServer发送心跳请求，报告健康状态
- eureka会更新记录服务列表信息，心跳不正常会被剔除
- 消费者就可以拉取到最新的信息

Eureka的作用：
![[Pasted image 20251004180715.png]]
在Eureka架构中，微服务角色有两类：
1. EurekaServer：服务端，注册中心
记录服务信息、心跳监控

2. EurekaClient：客户端
Provider：服务提供者，例如案例中的 user-service
- 注册自己的信息到EurekaServer
- 每隔30秒向EurekaServer发送心跳
consumer：服务消费者，例如案例中的 order-service
- 根据服务名称从EurekaServer拉取服务列表
- 基于服务列表做负载均衡，选中一个微服务后发起远程调用
### 搭建Eureka注册中心
搭建EurekaServer服务步骤如下：
1. 创建项目，引入spring-cloud-starter-netflix-eureka-server的依赖
```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>  
</dependency>
```
2. 编写启动类，添加@EnableEurekaServer注解
3. 添加application.yml文件，编写下面的配置：
```yml
server:
  port: 10086
spring:  
  application:    
  name: eurekaserver  
eureka:  
  client:    
    service-url:      
      defaultZone: http://127.0.0.1:10086/eureka/
```
Eureka自己也是微服务，在启动时会将自己也注册到Eureka，为了方便Eureka集群交流。
一个服务在线上部署一个就是一个实列。
### 注册user-service
将user-service服务注册到EurekaServer步骤如下：
1. 在user-service项目引入`spring-cloud-starter-netflix-eureka-client`的依赖
```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>  
</dependency>
```
2. 在application.yml文件，编写下面的配置：
```yml
spring:  
  application:    
    name: userservice  
eureka:  
  client:    
    service-url:      
      defaultZone: http://127.0.0.1:10086/eureka/
```
### 服务发现（服务拉取）
服务拉取是基于服务名称获取服务列表，然后在对服务列表做负载均衡
1. 修改OrderService的代码，修改访问的url路径，用服务名代替ip、端口：
```java
String url = "http://[userservice]/user/" + order.getUserId();
```
`[userservice]`这个东西不是固定的，可以去yml文件中找到Ctrl+cv一下。
服务注册的第2步
2. 在order-service项目的启动类OrderApplication中的RestTemplate**添加负载均衡注解**：
```java
@Bean  
@LoadBalanced  
public RestTemplate restTemplate() {    return new RestTemplate();  
}
```
## Ribbon负载均衡
Ribbon组件实现了Eureka的功能。
![[Pasted image 20251004193351.png]]
### 负载均衡策略
Ribbon的负载均衡规则是一个叫做IRule的接口来定义的，每一个子接口都是一种规则：
![[Pasted image 20251004193433.png]]
![[Pasted image 20251004193551.png]]
如果配置了Zone的值（杭州、上海），在做轮询的时候会优先选择和自己在同一个Zone内的服务然后再做轮询。
默认规则是红色那个。

通过定义IRule实现可以修改负载均衡规则，有两种方式：
1. 代码方式：在order-service中的OrderApplication类中，定义一个新的IRule：
```java
@Bean  
public IRule randomRule(){    
	return new RandomRule();  //在这一句设定修改了负载均衡规则，new 后面加方法就行。
}
```
这种方式是作用于全局的，在OrderApplication类中无论调用哪个微服务都是执行这种负载均衡规则的。
2. 配置文件方式：在order-service的application.yml文件中，添加新的配置也可以修改规则：
```yml
userservice:  #指定服务名称
  ribbon:    
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule# 负载均衡规则
```
这种方式是针对某个微服务而言的。
### 饥饿加载
饥饿加载：在项目启动时创建。
Ribbon默认是采用懒加载，即第一次访问时才会去创建LoadBalanceClient，请求时间会很长。
而饥饿加载则会在项目启动时创建，降低第一次访问的耗时，通过下面配置开启饥饿加载：
```yml
ribbon:  
  eager-load:    
    enabled: true # 开启饥饿加载    
      clients: userservice # 指定对userservice这个服务饥饿加载(只针对一个)
        
		- userservice   #针对多个进行饥饿加载（记得把client后面的给去了）
        - xxxservice
```

## Nacos注册中心
[`Nacos`](https://nacos.io/)是阿里巴巴的产品，现在是[`SpringCloud`](https://spring.io/projects/spring-cloud)中的一个组件。相比[`Eureka`](https://github.com/Netflix/eureka)功能更加丰富，在国内受欢迎程度较高。
1.在cloud-demo父工程中添加spring-cloud-alilbaba的管理依赖：
```xml
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>  
    <version>2.2.6.RELEASE</version>  
    <type>pom</type>  
    <scope>import</scope>  
</dependency>
```
2.注释掉order-service和user-service中原有的eureka依赖。
3.添加nacos的客户端依赖：
```xml
<!-- nacos客户端依赖 -->  
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
</dependency>
```
4.修改user-service&order-service中的application.yml文件，注释eureka地址，添加nacos地址：
```yml
spring:  
  cloud:    
    nacos:      
      server-addr: localhost:8848 # nacos 服务端地址
```
### Nacos服务分级存储模型
服务  <  集群  <  服务
![[Pasted image 20251004222015.png]]
服务调用尽可能选择本地集群的服务，跨集群调用延迟较高
本地集群不可访问时，再去访问其它集群
![[Pasted image 20251004222315.png]]
实现：
修改application.yml，添加如下内容：
```yml
spring:  
  cloud:    
    nacos:      
      server-addr: localhost:8848 # nacos 服务端地址      
      discovery:        
        cluster-name: HZ # 配置集群名称，也就是机房位置，例如：HZ，杭州
```
### NacosRule集群负载均衡
1.修改order-service中的application.yml，设置集群为HZ：
```yml
spring:  
  cloud:    
    nacos:      
      server-addr: localhost:8848 # nacos 服务端地址
      discovery:        
        cluster-name: HZ # 配置集群名称，也就是机房位置
```
2.然后在order-service中设置负载均衡的IRule为NacosRule，这个规则优先会寻找与自己同集群的服务：
```yml
userservice:  
  ribbon:    
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule # 负载均衡规则
```
NacosRule负载均衡策略
1. 优先选择同集群服务实例列表
2. 本地集群找不到提供者，才去其它集群寻找，并且会报警告
3. 确定了可用实例列表后，再采用随机负载均衡挑选实例
### 根据权重负载均衡
实际部署中会出现这样的场景：
- 服务器设备性能有差异，部分实例所在机器性能较好，另一些较差，我们希望性能好的机器承担更多的用户请求
Nacos提供了权重配置来控制访问频率，权重越大则访问频率越高

可以实现服务器代码的平滑升级。
![[Pasted image 20251004223501.png]]
实例的权重控制
-  Nacos控制台可以设置实例的权重值，0~1之间
- 同集群内的多个实例，权重越高被访问的频率越高
- 权重设置为0则完全不会被访问
### 环境隔离 - namespace
Nacos中服务存储和数据存储的最外层都是一个名为namespace的东西，用来做最外层隔离
1. 在Nacos控制台可以创建namespace，用来隔离不同环境
![[Pasted image 20251004224009.png]]
2. 然后填写一个新的命名空间信息：
![[Pasted image 20251004224032.png]]
3. 保存后会在控制台看到这个命名空间的id：
![[Pasted image 20251004224120.png]]
4. 修改order-service的application.yml，添加namespace：
```yml
spring:  
  datasource:   
  url: jdbc:mysql://localhost:3306/heima?useSSL=false    
  username: root    
  password: 123    
  driver-class-name: com.mysql.jdbc.Driver  
cloud:    
  nacos:      
  server-addr: localhost:8848      
  discovery:        
    cluster-name: SH # 上海        
    namespace: 492a7d5d-237b-46a1-a99a-fa8e98e4b0f9 # 命名空间，填ID
```
这个namespace的id就是上面创建得到的。
### Nacos和Eureka对比
![[Pasted image 20251004225056.png]]
服务注册到Nacos时，可以选择注册为临时或非临时实例，通过下面的配置来设置：
```yml
spring:  
  cloud:    
    nacos:      
      discovery:        
        ephemeral: false # 设置为非临时实例
```
临时实例宕机时，会从nacos的服务列表中剔除，而非临时实例则不会

1、Nacos与eureka的共同点
- 都支持服务注册和服务拉取
- 都支持服务提供者心跳方式做健康检测
2、Nacos与Eureka的区别
- Nacos支持服务端主动检测提供者状态：临时实例采用心跳模式，非临时实例采用主动检测模式
- 临时实例心跳不正常会被剔除，非临时实例则不会被剔除
- Nacos支持服务列表变更的消息推送模式，服务列表更新更及时
- Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式
## Nacos配置管理
### 统一配置管理
- Nacos可以实现配置更改热更新
![[Pasted image 20251005085037.png]]
在网页进行配置。
在内容部分不要配置文件的一切配置，而是需要更新的核心配置。
![[Pasted image 20251005090114.png]]
1. 引入Nacos的配置管理客户端依赖：
 ```xml
 <!--nacos配置管理依赖-->  
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>  
</dependency>
 ```

2. 在userservice中的resource目录添加一个bootstrap.yml文件，这个文件是引导文件，优先级高于application.yml：
```yml
spring:  
  application:    
    name: userservice # 服务名称  
  profiles:    
    active: dev #开发环境，这里是dev  
  cloud:    
    nacos:       
      server-addr: localhost:8848 # Nacos地址      
      config:        
        file-extension: yaml # 文件后缀名
```
3. 测试
在user-service中将pattern.dateformat这个属性注入到UserController中做测试：
```java
@RestController  
@RequestMapping("/user")  
public class UserController {  

    // 注入nacos中的配置属性    
    @Value("${pattern.dateformat}")    
    private String dateformat;  
 
    // 编写controller，通过日期格式化器来格式化现在时间并返回    
    @GetMapping("now")    
    public String now(){        
    return LocalDate.now().format(  
                DateTimeFormatter.ofPattern(dateformat, Locale.CHINA)  
        );  
    }
    // ... 略  
}
```
### 配置自动刷新（热启动）
Nacos中的配置文件变更后，微服务无需重启就可以感知。不过需要通过下面两种配置实现：
- 方式一：在@Value注入的变量所在类上添加注解@RefreshScope
![[Pasted image 20251005090844.png]]
- 方式二：使用@ConfigurationProperties注解
```java
@Component  
@Data  
@ConfigurationProperties(prefix = "pattern")  
public class PatternProperties {    
	private String dateformat;  
}
```
注入空指针的，看一下自己写yaml时，冒号后面加空格没，不加就空指针，加空格是yaml语法
#### 小结
Nacos配置更改后，微服务可以实现热更新，方式：
1. 通过@Value注解注入，结合@RefreshScope来刷新
2. 通过@ConfigurationProperties注入，自动刷新

注意事项：
- 不是所有的配置都适合放到配置中心，维护起来比较麻烦
- 建议将一些关键参数，需要运行时调整的参数放到nacos配置中心，一般都是自定义配置
### 多环境配置共享
微服务启动时会从nacos读取多个配置文件：
- `[spring.application.name]-[spring.profiles.active].yaml`，
	例如：userservice-dev.yaml
- `[spring.application.name].yaml`，例如：userservice.yaml
无论profile如何变化，`[spring.application.name].yaml`这个文件一定会加载，因此多环境共享配置可以写入这个文件
![[Pasted image 20251005092547.png]]
但只能改配置环境，如输入：test，即测试环境
#### 优先级
![[Pasted image 20251005093512.png]]
### Nacos集群搭建
Nacos生产环境下一定要部署为集群状态，部署方式参考文档。
![[Pasted image 20251005093645.png]]
# http客户端Feign
先来看以前利用RestTemplate发起远程调用的代码：
```java
String url = "http://userservice/user/" + order.getUserId();  
User user = restTemplate.getForObject(url, User.class);
```
存在下面的问题：
- 代码可读性差，编程体验不统一
- 参数复杂URL难以维护
## 关于Feign
Feign是一个声明式的http客户端，官方地址：[Feign](https://github.com/OpenFeign/feign)
作用是: 优雅的实现http请求的发送，解决上面提到的问题。

使用Feign的步骤如下：
1. 引入依赖：
```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-openfeign</artifactId>  
</dependency>
```
2. 在order-service的启动类添加注解开启Feign的功能：
![[Pasted image 20251005094322.png]]
3. 编写Feign客户端：
```java
@FeignClient("userservice")  
public interface UserClient {    
	@GetMapping("/user/{id}")    
	User findById(@PathVariable("id") Long id);  
}
```
主要是基于SpringMVC的注解来声明远程调用的信息，比如：
- 服务名称：userservice
- 请求方式：GET
- 请求路径：/user/{id}
- 请求参数：Long id
- 返回值类型：User
4. 用Feign客户端代替RestTemplate
![[Pasted image 20251005094635.png]]
## 自定义Feign的配置
Feign运行自定义配置来覆盖默认配置，可以修改的配置如下：
![[Pasted image 20251005095549.png]]
一般我们需要配置的就是日志级别。

配置Feign日志有两种方式：
- 方式一：配置文件方式
![[Pasted image 20251005095749.png]]
- 方式二：java代码方式，需要先声明一个Bean：
![[Pasted image 20251005095840.png]]
## Feign的性能优化
Feign底层的客户端实现：
- URLConnection：默认实现，不支持连接池
- `Apache HttpClient` ：支持连接池
- OKHttp：支持连接池

因此优化Feign的性能主要包括：
1. 使用连接池代替默认的URLConnection
2. 日志级别，最好用basic或none

Feign添加HttpClient的支持：
引入依赖：
```xml
<!--httpClient的依赖 -->  
<dependency>  
    <groupId>io.github.openfeign</groupId>  
    <artifactId>feign-httpclient</artifactId>  
</dependency>
```
配置连接池：
```yaml
feign:  
  client:    
    config:      
      default: # default全局的配置        
      loggerLevel: BASIC # 日志级别，BASIC就是基本的请求和响应信息  
  httpclient:    
    enabled: true # 开启feign对HttpClient的支持    
      max-connections: 200 # 最大的连接数    
      max-connections-per-route: 50 # 每个路径的最大连接数
```
## Feign的最佳实践
方式一（继承）：给消费者的FeignClient和提供者的controller定义统一的父接口作为标准。
![[Pasted image 20251005100804.png]]
这种方法会导致：
- 服务紧耦合（service层和Controller层绑死在一个接口上）
- 父接口参数列表中的映射不会被继承

方式二（抽取）：将FeignClient抽取为独立模块，并且把接口有关的POJO、默认的Feign配置都放到这个模块中，提供给所有消费者使用
- 原来：
![[Pasted image 20251005101129.png]]
- 现在：
![[Pasted image 20251005101056.png]]
这种方法的问题是：fegin-api可能会过度抽取，即部分service可能并不需要fegin-api的完整功能。
#### 抽取FeignClient
实现最佳实践方式二的步骤如下：
1. 首先创建一个module，命名为feign-api，然后引入feign的starter依赖
2. 将order-service中编写的UserClient、User、DefaultFeignConfiguration都复制到feign-api项目中
3. 在order-service中引入feign-api的依赖
4. 修改order-service中的所有与上述三个组件有关的import部分，改成导入feign-api中的包
5. 重启测试

当定义的FeignClient不在SpringBootApplication的扫描包范围时，这些FeignClient无法使用。有两种方式解决：
方式一：指定FeignClient所在包
```java
@EnableFeignClients(basePackages = "cn.itcast.feign.clients")
```
方式二：指定FeignClient字节码
```java
@EnableFeignClients(clients = {UserClient.class})
```
# 统一网关Gateway
## 为什么需要网关
网关功能：
- 身份认证和权限校验
- 服务路由、负载均衡
- 请求限流
![[Pasted image 20251005102454.png]]
在SpringCloud中网关的实现包括两种：
- gateway
- `zuul`
Zuul是基于Servlet的实现，属于阻塞式编程。
而SpringCloudGateway则是基于Spring5中提供的WebFlux，属于响应式编程的实现，具备更好的性能。
## 搭建网关服务
1. 创建新的module，引入SpringCloudGateway的依赖和nacos的服务发现依赖：
```xml
<!--网关依赖-->  
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-gateway</artifactId>  
</dependency>  
<!--nacos服务发现依赖-->  
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>  
</dependency>
```
2. 编写路由配置及nacos地址
请求路由是通过配置实现的。
![[Pasted image 20251005102833.png]]
`lb == loadBalance`
predicate 断言。
![[Pasted image 20251005103636.png]]
## 路由断言工厂Route Predicate Factory
网关路由可以配置的内容包括：
- 路由id：路由唯一标示
- uri：路由目的地，支持lb和http两种
- predicates：路由断言，判断请求是否符合要求，符合则转发到路由目的地
- filters：路由过滤器，处理请求或响应

在配置文件中写的断言规则只是字符串，这些字符串会被Predicate Factory读取并处理，转变为路由判断的条件
- 例如`Path=/user/**`是按照路径匹配，这个规则是由org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory类来处理的   (这个方式是Path断言工厂)

Spring提供了11种基本的Predicate工厂：
![[Pasted image 20251005103934.png]]
## 路由过滤器 `GatewayFilter`
GatewayFilter是网关中提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理：
![[Pasted image 20251005112147.png]]
Spring提供了31种不同的路由过滤器工厂。例如：
![[Pasted image 20251005112215.png]]

过滤器的作用是什么？
1. 对路由的请求或响应做加工处理，比如添加请求头
2. 配置在路由下的过滤器只对当前路由的请求生效
defaultFilters的作用是什么？
- 对所有路由都生效的过滤器
### 全局过滤器 `GlobalFilter`
全局过滤器的作用也是处理一切进入网关的请求和微服务响应，与GatewayFilter的作用一样。
区别在于GatewayFilter通过配置定义，处理逻辑是固定的。而GlobalFilter的逻辑需要自己写代码实现。
定义方式是实现GlobalFilter接口。
```java
public interface GlobalFilter {   /**  
    *  处理当前请求，有必要的话通过{@link GatewayFilterChain}将请求交给下一个过滤器处理
    * @param exchange 请求上下文，里面可以获取Request、Response等信息    
    * @param chain 用来把请求委托给下一个过滤器    
    * @return {@code Mono<Void>} 返回标示当前过滤器业务结束    */   
      Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain);  
}
```
过滤器的处理逻辑可以这么写：
```java
// @Order(-1)  
@Component  
public class AuthorizeFilter implements GlobalFilter, Ordered {  
    @Override  
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {  
        // 1.获取请求参数  
        ServerHttpRequest request = exchange.getRequest();  
        MultiValueMap<String, String> params = request.getQueryParams();  
        // 2.获取参数中的 authorization 参数  
        String auth = params.getFirst("authorization");  
        // 3.判断参数值是否等于 admin        if ("admin".equals(auth)) {  
            // 4.是，放行  
            return chain.filter(exchange);  
        }  
        // 5.否，拦截  
        // 5.1.设置状态码  
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);  
        // 5.2.拦截请求  
        return exchange.getResponse().setComplete();  
    }  
  
    @Override  
    public int getOrder() {  
        return -1;  
    }  
}
```
@Order(-1)  这个和下面那个getOrder选择一个写就行，getOrder的返回值就是优先级（越大优先级越低），使用这个方法需要实现Ordered接口。
## 过滤器执行顺序
请求进入网关会碰到三类过滤器：当前路由的过滤器、DefaultFilter、GlobalFilter
![[Pasted image 20251005113907.png]]
请求路由后，会将当前路由过滤器和DefaultFilter、GlobalFilter，合并到一个过滤器链（集合）中，排序后依次执行每个过滤器

在GatewayFilterAdapter网关路由过滤适配器把GlobalFilter做了一个适配，变成了GatewayFilter。

- 每一个过滤器都必须指定一个int类型的order值，order值越小，优先级越高，执行顺序越靠前。
- GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定
- 路由过滤器和defaultFilter的order由Spring指定，默认是按照  **声明顺序从1递增**。
- 当过滤器的order值一样时，会按照 `defaultFilter > 路由过滤器 > GlobalFilter` 的顺序执行。

可以参考下面几个类的源码来查看：
```java
org.springframework.cloud.gateway.route.RouteDefinitionRouteLocator#getFilters()
方法是先加载defaultFilters，然后再加载某个route的filters，然后合并。

org.springframework.cloud.gateway.handler.FilteringWebHandler#handle()
方法会加载全局过滤器，与前面的过滤器合并后根据order排序，组织过滤器链
```
## 跨域问题处理
跨域：域名不一致就是跨域，主要包括：
- 域名不同： `www.taobao.com` 和 `www.taobao.org` 和 `www.jd.com` 和 miaosha.jd.com
- 域名相同，端口不同：localhost:8080和localhost8081
跨域问题：**浏览器**  禁止请求的发起者与服务端发生  **跨域ajax请求**  ，请求被浏览器拦截的问题

解决方案：CORS
网关处理跨域采用的同样是CORS方案，并且只需要简单配置即可实现：
![[Pasted image 20251005114905.png]]
有效期期间内浏览器不再询问，而是直接放行。
注意：
```yml
 add-to-simple-url-handler-mapping: true 
```
网关中特有的，Ajax采用的是CORS方案（浏览器询问服务器，你让不让那个域名跨域），这个询问它的请求方式就是option，
默认情况下option请求会被网关拦截，改为true  让网关不拦截。
## 限流过滤器
限流：对应用服务器的请求做限制，避免因过多请求而导致服务器过载甚至宕机。

限流算法常见如下：
- 计数器算法，又包括窗口计数器算法、滑动窗口计数器算法
![[Pasted image 20251005115742.png]]
- 漏桶算法(Leaky Bucket)
![[Pasted image 20251005115758.png]]
- 令牌桶算法（Token Bucket）
![[Pasted image 20251005115831.png]]

