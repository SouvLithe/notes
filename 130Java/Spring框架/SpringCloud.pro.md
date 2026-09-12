# 微服务保护
微服务保护会基于Sentinel框架实现。
## 雪崩问题
微服务调用链路中的某个服务故障（导致微服务任务阻塞，不释放tomcat链接，阻塞服务过多导致微服务宕机，而其它微服务也因访问不到这个微服务宕机），引起整个链路中的所有微服务都不可用，这就是雪崩。

解决雪崩问题的常见方式有四种：
1. 超时处理：设定超时时间，请求超过一定时间没有响应就返回错误信息，不会无休止等待（缓解）
2. 舱壁模式：限定每个业务能使用的线程数，避免耗尽整个tomcat的资源，因此也叫  **线程隔离**。
![[Pasted image 20251005180951.png]]
3. 熔断降级：由  **断路器**  统计业务执行的异常比例，如果超出阈值则会  **熔断**  该业务，拦截访问该业务的一切请求。
4. 流量控制：限制业务访问的QPS，避免服务因流量的突增而故障。（预防机制，前面的是处理方法）
![[Pasted image 20251005181232.png]]
### 小问题
如何避免因瞬间高并发流量而导致服务故障？
- 流量控制
如何避免因服务故障引起的雪崩问题？
- 超时处理、线程隔离、降级熔断
## 微服务保护框架
实现雪崩问题最好使用现有框架。
![[Pasted image 20251005181741.png]]
### Sentinel整合
Sentinel是阿里巴巴开源的一款微服务流量控制组件。官网地址:
[`sentinelguard`](https://sentinelguard.io/zh-cn/index.html)
在`D:\Worksorts\sentinel-1.8.8`目录下已经下好了，里面的md文件是使用教程。

在order-service中整合Sentinel，并且连接Sentinel的控制台，步骤如下：
1. 引入sentinel依赖：
```xml
<!--sentinel-->  
<dependency>  
    <groupId>com.alibaba.cloud</groupId>    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>  
</dependency>
```
2. 配置控制台地址：
```yaml
spring:  
  cloud:    
    sentinel:      
      transport:        
        dashboard: localhost:8080  #sentinel控制台地址
```
sentinel控制台地址如果加了参数改了的话，记得要和参数端口保持一致。
3. 访问微服务的任意端点，触发sentinel监控
## 限流规则
簇点链路：就是项目内的调用链路，链路中被监控的每个接口就是一个资源。
默认情况下sentinel会监控SpringMVC的每一个端点（Endpoint，可以理解为Controller方法），因此SpringMVC的每一个端点（Endpoint）就是调用链路中的一个资源。

**后面都是基于Sentinel控制台进行操作的，不是的会标记。**
### 流控模式
流控、熔断等都是  **针对簇点链路中的资源**  来设置的，因此我们可以点击对应资源后面的按钮来设置规则：
![[Pasted image 20251005184943.png]]
在添加限流规则时，点击高级选项，可以选择三种流控模式：
1. 直接：统计当前资源的请求，触发阈值时对当前资源直接限流（默认）。
2. 关联：统计与当前资源相关的另一个资源，触发阈值时，对当前资源限流
满足下面条件可以使用关联模式：
- 两个有竞争关系的资源
- 一个优先级较高，一个优先级较低

3. 链路：统计从指定链路访问到本资源的请求，触发阈值时，对指定链路限流
对链接来源做限流。
![[Pasted image 20251005190422.png]]
#### 小结
流控模式有哪些？（简化定义方便理解）
- 直接：对当前资源限流
- 关联：高优先级资源触发阈值，对低优先级资源限流。
- 链路：阈值统计时，只统计从指定资源进入当前资源的请求，是对请求来源的限流
### 流控效果
流控效果是指请求达到流控阈值时应该采取的措施，包括三种：
- 快速失败：达到阈值后，新的请求会被立即拒绝并抛出FlowException异常。是默认的处理方式。
- warm up：预热模式，对超出阈值的请求同样是拒绝并抛出异常。但这种模式阈值会动态变化，从一个较小值逐渐增加到最大阈值。
- 排队等待：让所有的请求按照先后次序排队执行，两个请求的间隔不能小于指定时长
#### warm up
warm up也叫预热模式，是应对服务冷启动的一种方案。
请求阈值初始值是 threshold / coldFactor，持续指定时长后，逐渐提高到threshold值。而coldFactor的默认值是3.

好处：
	QPS阈值是逐渐提升的，可以避免冷启动时高并发导致服务宕机。
#### 排队等待
当请求超过QPS阈值时，快速失败  和  warm up 会拒绝新的请求并抛出异常。
而排队等待则是让所有请求进入一个队列中，然后按照阈值允许的时间间隔依次执行。后来的请求必须等待前面执行完成，如果请求预期的等待时间超出最大时长，则会被拒绝。
适用于：
	把波动的流量转为均衡的流量，起到一个流量整形的效果
### 热点参数限流
之前的限流是统计访问某个资源的所有请求，判断是否超过QPS阈值。
而热点参数限流是分别统计  **参数值相同**  的请求，判断是否超过QPS阈值。
![[Pasted image 20251005192012.png]]
示例：
![[Pasted image 20251005192118.png]]

在热点参数限流的高级选项中，可以对部分参数设置例外配置：
![[Pasted image 20251005192211.png]]
注意：
	热点参数限流对默认的SpringMVC资源无效。只有通过`@SentinelResource("hot")`注解去声明的资源，才能去配置热点参数无限流。(“”里面的hot是资源名称。可以随便取)
## 隔离和降级
不管是线程隔离还是熔断降级，都是对客户端（调用方）的保护。
### Feign整合Sentinel
SpringCloud中，微服务调用都是通过Feign来实现的，因此做客户端保护必须整合Feign和Sentinel。
1. 修改OrderService的application.yml文件，开启Feign的Sentinel功能
```xml
feign:  
  sentinel:    
    enabled: true # 开启Feign的Sentinel功能
```
2. 给FeignClient编写失败后的降级逻辑
降级逻辑，即想办法给它写一个备用方案。
方式一：FallbackClass，无法对远程调用的异常做处理
方式二：FallbackFactory，可以对远程调用的异常做处理，我们选择这种
#### 编写降级逻辑
实现步骤二。
1. 在feing-api项目中定义类，实现FallbackFactory：
![[Pasted image 20251005204922.png]]
2. 在feing-api项目中的DefaultFeignConfiguration类中将UserClientFallbackFactory注册为一个Bean：
![[Pasted image 20251005205006.png]]
3. 在feing-api项目中的UserClient接口中使用UserClientFallbackFactory：
![[Pasted image 20251005205032.png]]
### 线程隔离
线程隔离有两种方式实现：
- 线程池隔离
- 信号量隔离（Sentinel默认采用）
![[Pasted image 20251005205846.png]]

| 特性  | 信号量隔离              | 线程池隔离            |
| --- | ------------------ | ---------------- |
| 优点  | 轻量级，无额外开销          | 支持主动超时<br>支持异步调用 |
| 缺点  | 不支持主动超时<br>不支持异步调用 | 线程的额外开销比较大       |
| 场景  | 高频调用<br>高扇出        | 低扇出              |
扇出：一个服务依赖于n个其它的服务。（扇出越高，调用的依赖服务越多，需要开启的线程也越多，消耗越大）
#### 实现
![[Pasted image 20251005210641.png]]
### 熔断降级
熔断降级其思路：由  **断路器**  统计服务调用的异常比例、慢请求比例，如果超出阈值则会  **熔断**  该服务。
即拦截访问该服务的一切请求；而当服务恢复时，断路器会放行访问该服务的请求。
![[Pasted image 20251005211032.png]]
熔断的条件在Sentinel也被称为熔断策略。
断路器熔断策略有三种：慢调用、异常比例、异常数
#### 慢调用
慢调用：业务的响应时长（RT）大于指定时长的请求认定为慢调用请求。
在指定时间内，如果请求数量超过设定的最小数量，慢调用比例大于设定的阈值，则触发熔断。例如：
![[Pasted image 20251005211304.png]]
#### 异常比例、异常数
异常比例或异常数：统计指定时间内的调用，如果调用次数超过指定请求数，并且出现异常的比例达到设定的比例阈值（或超过指定异常数），则触发熔断。例如：
![[Pasted image 20251005211640.png]]
解读：统计最近1000ms内的请求，如果请求量超过10次，并且异常比例不低于0.5，则触发熔断，熔断时长为5秒。然后进入half-open状态，放行一次请求做测试。
## 授权规则
对请求者的身份进行一个判断。
授权规则可以对调用方的来源做控制，有白名单和黑名单两种方式。
- 白名单：来源（origin）在白名单内的调用者允许访问
- 黑名单：来源（origin）在黑名单内的调用者不允许访问
![[Pasted image 20251005212427.png]]
### 实现授权规则
Sentinel是通过RequestOriginParser（请求来源解析器）这个接口的parseOrigin来获取请求的来源的。
```java
public interface RequestOriginParser {    
	/**  
     * 从请求request对象中获取origin，获取方式自定义
     */    
     String parseOrigin(HttpServletRequest request);  
}
```
1. 自定义实现业务，解析来源名称。
请求头、请求参数、cookie都行，只要能区分来源就行。
例如，尝试从request中获取一个名为origin的请求头，作为origin的值：
```java
@Component  
public class HeaderOriginParser implements RequestOriginParser {    
	@Override    
	public String parseOrigin(HttpServletRequest request) {  
        String origin = request.getHeader("origin");        
        if(StringUtils.isEmpty(origin)){            
	        return "blank";  
        }        
        return origin;  
    }  
}
```
2. 给允许的服务，加上这个请求头：
我们还需要在gateway服务中，利用网关的过滤器添加名为gateway的origin头：
![[Pasted image 20251005213238.png]]
3. 给`/order/{orderId}` 配置授权规则：
![[Pasted image 20251005213310.png]]
### 自定义异常结果
默认情况下，发生限流、降级、授权拦截时，都会抛出异常到调用方。

如果要自定义异常时的返回结果，需要实现BlockExceptionHandler接口：
![[Pasted image 20251005214103.png]]

而BlockException包含很多个子类，分别对应不同的场景：
![[Pasted image 20251005214157.png]]
### 小结
记住这两个接口会用就行。
```text
获取请求来源的接口是什么？
•RequestOriginParser
处理BlockException的接口是什么？
•BlockExceptionHandler
```
## 规则持久化
- 原始模式：保存在内存(默认)
- pull模式：保存在本地文件或数据库，定时去读取
- push模式：保存在nacos，监听变更实时更新
### 详细
![[Pasted image 20251005214646.png]]
- 原始模式：控制台配置的规则直接推送到Sentinel客户端，也就是我们的应用。然后保存在内存中，服务重启则丢失.
- pull模式：控制台将配置的规则推送到Sentinel客户端，而客户端会将配置规则保存在本地文件或数据库中。以后会定时去本地文件或数据库中查询，更新本地规则。
![[Pasted image 20251005214923.png]]
- push模式：控制台将配置规则推送到远程配置中心，例如Nacos。Sentinel客户端监听Nacos，获取配置变更的推送消息，完成本地配置更新。
![[Pasted image 20251005215055.png]]
### push实现
Sentinel要推向远程配置中心。
但在Sentinel的默认实现中都是推向客户端的，要实现push模式不得不去改源代码或者买服务。
而且客户端也要去监听Nacos，因此也要去改微服务端。
# 分布式事务
事务必须要满足ACID原则。
在分布式系统下，一个业务跨越多个服务或数据源，每个服务都是一个分支事务，要保证所有分支事务最终状态一致，这样的事务就是  **分布式事务**。

事务问题：
各个事务之间互相是感知不到的，各提交各的所以无法回滚，导致状态无法一致。
## 理论基础
### CAP定理
1998年，加州大学的计算机科学家 Eric Brewer 提出，分布式系统有三个指标：
- Consistency（一致性）：用户访问分布式系统中的任意节点，得到的数据必须一致
![[Pasted image 20251005221730.png]]
- Availability（可用性）：用户访问集群中的任意健康节点，必须能得到响应，而不是超时或拒绝
![[Pasted image 20251005221803.png]]
- Partition tolerance （分区容错性）：因为网络故障或其它原因导致分布式系统中的部分节点与其它节点失去连接，形成独立分区。
Tolerance（容错）：在集群出现分区时，整个系统也要持续对外提供服务
![[Pasted image 20251005221545.png]]
02可以给01，但因为网络问题无法给03，导致03无法得到最新数据

Eric Brewer 说，分布式系统无法同时满足这三个指标。
这个结论就叫做 CAP 定理。
![[Pasted image 20251005221329.png]]
#### 小结
简述CAP定理内容？
- 分布式系统节点通过网络连接，一定会出现分区问题（P）
- 当分区出现时，系统的一致性（C）和可用性（A）就无法同时满足
### BASE理论
BASE理论是对CAP的一种解决思路，包含三个思想：
- **Basically Available （基本可用)**：分布式系统在出现故障时，允许损失部分可用性，即  **保证核心可用**。
- **Soft State（软状态)**：在一定时间内，允许出现中间状态，比如  **临时**  的不一致状态。
- **Eventually Consistent（最终一致性)**：虽然无法保证强一致性，但是在软状态结束后，最终达到数据一致。

而分布式事务最大的问题是各个子事务的一致性问题，因此可以借鉴CAP定理和BASE理论：
- AP模式：各子事务分别执行和提交，允许出现结果不一致，然后采用弥补措施恢复数据即可，实现  **最终一致**。
- CP模式：各个子事务执行后互相等待，同时提交，同时回滚，达成  **强一致**。但事务等待过程中，处于弱可用状态。


解决分布式事务，  **各个子系统之间必须能感知到彼此的事务状态**  ，才能保证状态一致，
因此需要一个事务协调者来协调每一个事务的参与者（子系统事务）。

这里的子系统事务，称为  **分支事务**  ；
有关联的各个分支事务在一起称为 **全局事务**。
## 初始Seata
[`Seata.`](http://seata.io/)是 2019 年 1 月份蚂蚁金服和阿里巴巴共同开源的分布式事务解决方案。致力于提供高性能和简单易用的分布式事务服务，为用户打造一站式的分布式解决方案。
### Seata架构
Seata事务管理中有三个重要的角色：
- **TC (Transaction Coordinator) - 事务协调者**：维护全局和分支事务的状态，协调全局事务提交或回滚。
- **TM (Transaction Manager) - 事务管理器**：定义全局事务的范围、开始全局事务、提交或回滚全局事务。
- **RM (Resource Manager) - 资源管理器**：管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。
![[Pasted image 20251005223418.png]]
tc：tm和rm的老大。 tm：入口事务的老大 rm：分支事务的老大
### 分布式事务解决方案
Seata提供了四种不同的分布式事务解决方案：
- XA模式：强一致性分阶段事务模式，牺牲了一定的可用性，无业务侵入
- TCC模式：最终一致的分阶段事务模式，有业务侵入
- AT模式：最终一致的分阶段事务模式，无业务侵入，也是Seata的默认模式
- SAGA模式：长事务模式，有业务侵入
## 部署TC服务
配置有点复杂，去网上找吧。或者黑马资料的《seata的部署和集成.md》这个文件看看。
## 微服务集成Seata
1. 引入seata相关依赖：
```xml
<dependency>  
    <groupId>com.alibaba.cloud</groupId>  
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId> 
    <exclusions>        
    <!--版本较低，1.3.0，因此排除-->        
	    <exclusion>  
            <artifactId>seata-spring-boot-starter</artifactId>  
            <groupId>io.seata</groupId>  
        </exclusion>  
    </exclusions>  
</dependency>  
<!--seata starter 采用1.4.2版本-->  
<dependency>  
    <groupId>io.seata</groupId>  
    <artifactId>seata-spring-boot-starter</artifactId>  
    <version>${seata.version}</version>  
</dependency>
```
2. 配置application.yml，让微服务通过注册中心找到seata-tc-server：
![[Pasted image 20251005224955.png]]
### 小结
nacos服务名称组成包括？
- `namespace + group + serviceName + cluster`
seata客户端获取tc的cluster名称方式？
- 以`tx-group-service`的值为key到`vgroupMapping`中查找
## Seata四种解决分布式事务的方案
TC (Transaction Coordinator) - 事务协调者：维护全局和分支事务的状态
TM (Transaction Manager) - 事务管理器：定义全局事务的范围
RM (Resource Manager) - 资源管理器：管理分支事务处理的资源
### XA模式原理
XA 规范 是 X/Open 组织定义的分布式事务处理（DTP，Distributed Transaction Processing）标准，XA 规范 描述了全局的TM与局部的RM之间的接口，几乎所有主流的数据库都对 XA 规范 提供了支持。
它是强一致性的。
![[Pasted image 20251006095438.png]]
seata的XA模式做了一些调整，但大体相似：
RM一阶段的工作：
- 注册分支事务到TC
- 执行分支业务sql但不提交
- 报告执行状态到TC
![[Pasted image 20251006095800.png]]
TC二阶段的工作：
TC检测各分支事务执行状态
	a.如果都成功，通知所有RM提交事务
	b.如果有失败，通知所有RM回滚事务
RM二阶段的工作：
接收TC指令，提交或回滚事务
#### 优缺点
XA模式的优点是什么？
- 事务的强一致性，满足ACID原则。
- 常用数据库都支持，实现简单，并且没有代码侵入

XA模式的缺点是什么？
- 因为一阶段需要锁定数据库资源，等待二阶段结束才释放，性能较差
- 依赖关系型数据库实现事务
#### 实现
Seata的starter已经完成了XA模式的自动装配，实现非常简单，步骤如下：

1. 修改application.yml文件（每个参与事务的微服务），开启XA模式：
```xml
seata:  
  data-source-proxy-mode: XA # 开启数据源代理的XA模式
```
2. 给发起全局事务的入口方法添加@GlobalTransactional注解，本例中是OrderServiceImpl中的create方法：
```java
@Override  
@GlobalTransactional  
public Long create(Order order) {    
	// 创建订单    
	orderMapper.insert(order);    
	// 扣余额 ...略
    // 扣减库存 ...略    
    return order.getId();  
}
```
3. 重启服务并测试
### AT模式原理
AT模式同样是分阶段提交的事务模型，不过缺弥补了XA模型中资源锁定周期过长的缺陷。
阶段一RM的工作：
- 注册分支事务
- **记录undo-log（数据快照）**
- 执行业务sql并提交
- 报告事务状态
![[Pasted image 20251006100924.png]]
阶段二提交时RM的工作：
- 删除undo-log即可
阶段二回滚时RM的工作：
- 根据undo-log恢复数据到更新前
#### XA与AT区别
简述AT模式与XA模式最大的区别是什么？
1. XA模式一阶段不提交事务，锁定资源；AT模式一阶段直接提交，不锁定资源。
2. XA模式依赖数据库机制实现回滚；AT模式利用数据快照实现数据回滚。
3. XA模式强一致；AT模式最终一致
#### AT模式的脏写问题
![[Pasted image 20251006101946.png]]
解决方法：
1. 写隔离:
![[Pasted image 20251006102058.png]]
隔离不太彻底：在修改这个money的过程中，有一个seata没有管理的事务它也来修改money字段，人家也不需要获取全局锁，也有可能出现脏写问题。（概率小）
![[Pasted image 20251006102727.png]]
#### 优缺点
AT模式的优点：
- 一阶段完成直接提交事务，释放数据库资源，性能比较好
- 利用全局锁实现读写隔离
- 没有代码侵入，框架自动完成回滚和提交

AT模式的缺点：
- 两阶段之间属于软状态，属于最终一致
- 框架的快照功能会影响性能，但比XA模式要好很多
它的性能比XA模式好很多，所以它是用的最多的一种。
#### 实现AT模式
AT模式中的快照生成、回滚等动作都是由框架自动完成，没有任何代码侵入，因此实现非常简单。
1. 两张表，其中lock_table（记录全局锁）导入到TC服务关联的数据库，undo_log（记录快照信息）表导入到微服务关联的数据库：
2. 修改application.yml文件，将事务模式修改为AT模式即可：
```yml
seata:  
  data-source-proxy-mode: AT # 开启数据源代理的AT模式
```
注意要使用`@GlobalTransactional`声明一个方法为全局事务方法。
3. 重启服务并测试
要看undo_log日志可用在代码上打上断点查看。
### TCC模式原理
TCC模式与AT模式非常相似，每阶段都是独立事务，不同的是TCC通过人工编码来实现数据恢复。需要实现三个方法：
- Try：资源的检测和预留；
- Confirm：完成资源操作业务；要求 Try 成功 Confirm 一定要能成功。
- Cancel：预留资源释放，可以理解为try的反向操作。
![[Pasted image 20251006103911.png]]
#### TCC模式原理
![[Pasted image 20251006104221.png]]
#### TCC的空回滚和业务悬挂
当某分支事务的try阶段阻塞时，可能导致全局事务超时而触发二阶段的cancel操作。在未执行try操作时先执行了cancel操作，这时cancel不能做回滚，就是空回滚。
![[Pasted image 20251006104907.png]]
对于那个没有执行try而进行空回滚的业务，若以后继续执行try，就永远不可能confirm或cancel，这就是业务悬挂。应当阻止执行空回滚后的try操作，避免悬挂 

解决加业务分析：
为了实现空回滚、防止业务悬挂，以及幂等性要求。我们必须在数据库记录冻结金额的同时，记录当前事务id和执行状态，为此我们设计了一张数据库表：
![[Pasted image 20251006105544.png]]
#### 实现
TCC的Try、Confirm、Cancel方法都需要在接口中基于注解来声明，语法如下：
![[Pasted image 20251006105805.png]]
1. @TwoPhaseBusinessAction写在哪个方法上哪个就是try，参数中声明的是那三个方法的名字。
2. 这三个方法有时需要共享参数，这时就要用`@BusinessActionContextParameter(paramName = "param")`，用这个注解标记出来的参数将来会放在BusinessActionContext这个上下文对象中。
3. 可以用这个上下文对象拿到这个参数。
#### 小结
TCC模式的每个阶段是做什么的？
- Try：资源检查和预留
- Confirm：业务执行和提交
- Cancel：预留资源的释放

TCC的优点是什么？
- 一阶段完成直接提交事务，释放数据库资源，性能好
- 相比AT模型，无需生成快照，无需使用全局锁，性能最强
- 不依赖数据库事务，而是依赖补偿操作，可以用于非事务型数据库

TCC的缺点是什么？
- 有代码侵入，需要人为编写try、Confirm和Cancel接口，太麻烦
- 软状态，事务是最终一致
- 需要考虑Confirm和Cancel的失败情况，**做好幂等处理**
- 会出现空回滚和业务悬挂的情况
### Saga模式
Saga模式是SEATA提供的长事务解决方案。也分为两个阶段：
- 一阶段：直接提交本地事务
- 二阶段：成功则什么都不做；失败则通过编写补偿业务来回滚

Saga模式优点：
- 事务参与者可以基于事件驱动实现异步调用，吞吐高
- 一阶段直接提交事务，无锁，性能好
- 不用编写TCC中的三个阶段，实现简单
缺点：
- 软状态持续时间不确定，时效性差
- 没有锁，没有事务隔离，会有脏写
### 四种模式对比
![[Pasted image 20251006112221.png]]
自动挡：XA模式AT模式，手动挡：tcc模式。at模式较xa模式性能更好。
## TC的异地多机房容灾架构
高可用 == 集群
![[Pasted image 20251006112821.png]]
# 分布式缓存
这个和[[Redis高级篇#分布式缓存]]的内容一样，可以去看看。
多级缓存也在这里讲过了,链接[[Redis高级篇#多级缓存]]。
# 服务异步通讯
主要就是就是rabbitmq的高级特性。
![[Pasted image 20251006171108.png]]
## 消息可靠性问题
mq是内存存储，一宕机内存数据直接消失。
消息从生产者发送到exchange，再到queue，再到消费者，有哪些导致消息丢失的可能性？
1. 发送时丢失：
- 生产者发送的消息未送达exchange
- 消息到达exchange后未到达queue
1. MQ宕机，queue将消息丢失
2. consumer接收到消息后未消费就宕机
![[Pasted image 20251006171540.png]]
### 生产者确认机制
RabbitMQ提供了publisher confirm机制来避免消息发送到MQ过程中丢失。消息发送到MQ以后，会返回一个结果给发送者，表示消息是否处理成功。结果有两种请求：
1、publisher-confirm，发送者确认
- 消息成功投递到交换机，返回ack
- 消息未投递到交换机，返回nack

2、publisher-return，发送者回执
- 消息投递到交换机了，但是没有路由到队列。返回ACK，及路由失败原因。
![[Pasted image 20251006171808.png]]
注意：
	确认机制发送消息时，需要给每个消息设置一个全局唯一id，以区分不同消息，避免ack冲突
#### SpringAMQP实现生产者确认
1. 在publisher这个微服务的application.yml中添加配置（开关）：
```yml
spring:  
  rabbitmq:  
    publisher-confirm-type: correlated  # 选择回调模式
	publisher-returns: true           # 把return功能打开
	template:      
	  mandatory: true   #表示路由失败以后要去回调，而不是抛弃
```
配置说明：
- publish-confirm-type：开启publisher-confirm，这里支持两种类型：
simple：同步等待confirm结果，直到超时
correlated：异步回调，定义ConfirmCallback，MQ返回结果时会回调这个ConfirmCallback
- publish-returns：开启publish-return功能，同样是基于callback机制，不过是定义ReturnCallback
- template.mandatory：定义消息路由失败时的策略。true，则调用ReturnCallback；false：则直接丢弃消息
2. 每个RabbitTemplate  **只能配置一个**  ReturnCallback，因此需要在项目启动过程中配置：
![[Pasted image 20251006172255.png]]
3. 发送消息，指定消息ID、消息ConfirmCallback
![[Pasted image 20251006172607.png]]
- 因为yml配置了correlated所以rabbitTemplate调用方法convertAndSend时，需要传递correlationData（里面封装了消息ID和callback）
- `correlationData.getFuture().addCallback`中的result指的时成功的回调函数，ex则是失败的回调函数。

SpringAMQP中处理消息确认的几种情况：
- `publisher-comfirm`：
•消息成功发送到exchange，返回ack
•消息发送失败，没有到达交换机，返回nack
•消息发送过程中出现异常，没有收到回执
- 消息成功发送到exchange，但没有路由到queue，调用ReturnCallback
### 消息持久化
MQ默认是内存存储消息，开启持久化功能可以确保缓存在MQ中的消息不丢失。
1.交换机持久化：
```java
@Bean  
public DirectExchange simpleExchange(){    
	// 三个参数：交换机名称、是否持久化、当没有queue与其绑定时是否自动删除    
	return new DirectExchange("simple.direct", true, false);  
}
```
2.队列持久化：
```java
@Bean  
public Queue simpleQueue(){    
	// 使用QueueBuilder构建队列，durable就是持久化的    
	return QueueBuilder.durable("simple.queue").build();  
}
```
3.消息持久化，SpringAMQP中的的消息默认是持久的，可以通过MessageProperties中的DeliveryMode来指定的：
```java
Message msg = MessageBuilder  
	        .withBody(message.getBytes(StandardCharsets.UTF_8)) // 消息体        
	        .setDeliveryMode(MessageDeliveryMode.PERSISTENT) // 持久化        
	        .build();
```
默认情况下交换机、队列、消息也好都是持久的。
### 消费者确认
RabbitMQ支持消费者确认机制，即：消费者处理消息后可以向MQ发送ack回执，MQ收到ack回执后才会删除该消息。而SpringAMQP则允许配置三种确认模式：
- manual：手动ack，需要在业务代码结束后，调用api发送ack。
- auto：自动ack，由spring监测listener代码是否出现异常，没有异常则返回ack；抛出异常则返回nack  (推荐使用这个)
- none：关闭ack，MQ假定消费者获取消息后会成功处理，因此消息投递后立即被删除

配置方式是修改application.yml文件，添加下面配置：
```yml
spring:  
  rabbitmq:    
    listener:      
      simple:        
        prefetch: 1        
        acknowledge-mode: none # none，关闭ack；manual，手动ack；auto：自动ack
```
### 消费者失败重试
当消费者出现异常后，消息会不断requeue（重新入队）到队列，再重新发送给消费者，然后再次异常，再次requeue，无限循环，导致mq的消息处理飙升，带来不必要的压力。

我们可以利用Spring的retry机制，在消费者出现异常时利用本地重试，而不是无限制的requeue到mq队列：
```yml
spring:  
  rabbitmq:    
    listener:      
      simple:        
        prefetch: 1        
		retry:          
		  enabled: true # 开启消费者失败重试          
		  initial-interval: 1000 # 初始的失败等待时长为1秒          
		  multiplier: 1 # 下次失败的等待时长倍数，下次等待时长 = multiplier * last-interval          
		  max-attempts: 3 # 最大重试次数          
		  stateless: true # true无状态；false有状态。如果业务中包含事务，这里改为false
```
stateless: true # true无状态，如果业务中包含事务，这里改为false
#### 消费者失败消息处理策略
在开启重试模式后，重试次数耗尽，如果消息依然失败，则需要有MessageRecoverer接口来处理，它包含三种不同的实现：
- RejectAndDontRequeueRecoverer：重试耗尽后，直接reject，丢弃消息。默认就是这种方式
- ImmediateRequeueMessageRecoverer：重试耗尽后，返回nack，消息重新入队
- RepublishMessageRecoverer：重试耗尽后，将失败消息投递到指定的交换机
![[Pasted image 20251006180453.png]]

测试下RepublishMessageRecoverer处理模式：
- 首先，定义接收失败消息的交换机、队列及其绑定关系：
```java
@Bean  
public DirectExchange errorMessageExchange(){    
	return new DirectExchange("error.direct");  
}  
@Bean  
public Queue errorQueue(){    
	return new Queue("error.queue", true);  
}  
@Bean  
public Binding errorBinding(){    
	return BindingBuilder.bind(errorQueue()).to(errorMessageExchange()).with("error");  
}
```
- 然后，定义RepublishMessageRecoverer：
```java
@Bean  
public MessageRecoverer republishMessageRecoverer(RabbitTemplate rabbitTemplate){    return new RepublishMessageRecoverer(rabbitTemplate, 
								"error.direct", "error");
								//这两个字符串要和上面的一致，将来的消息才能发到对的位置 
}
```
#### 小结
如何确保RabbitMQ消息的可靠性？
- 开启生产者确认机制，确保生产者的消息能到达队列
- 开启持久化功能，确保消息未消费前在队列中不会丢失
- 开启消费者确认机制为auto，由spring确认消息处理成功后完成ack
- 开启消费者失败重试机制，并设置MessageRecoverer，多次重试失败后将消息投递到异常交换机，交由人工处理
## 死信交换机
MQ的延迟消息问题。
### 初识死信交换机
当一个队列中的消息满足下列情况之一时，可以成为死信（dead letter）：

- 消费者使用basic.reject或 basic.nack声明消费失败，并且消息的requeue参数设置为false
- 消息是一个过期消息，超时无人消费
- 要投递的队列消息堆积满了，最早的消息可能成为死信

如果该队列配置了  **dead-letter-exchange属性**  ，指定了一个交换机，那么队列中的死信就会投递到这个交换机中，而这个交换机称为  **死信交换机**（Dead Letter Exchange，简称DLX）。
![[Pasted image 20251006195353.png]]
#### 小结
什么样的消息会成为死信？
- 消息被消费者reject或者返回nack
- 消息超时未消费
- 队列满了

如何给队列绑定死信交换机？
- 给队列设置dead-letter-exchange属性，指定一个交换机
- 给队列设置dead-letter-routing-key属性，设置死信交换机与死信队列的RoutingKey
### TTL
TTL，也就是Time-To-Live。如果一个队列中的消息TTL结束仍未消费，则会变为死信，ttl超时分为两种情况：
- 消息所在的队列设置了存活时间
- 消息本身设置了存活时间

我们声明一组死信交换机和队列，基于注解方式：
```java
@RabbitListener(bindings = @QueueBinding(  
        value = @Queue(name = "dl.queue", durable = "true"),  
        exchange = @Exchange(name = "dl.direct"),  
        key = "dl"  
))  
public void listenDlQueue(String msg){    
	log.info("接收到 dl.queue的延迟消息：{}", msg);  
}
```
1. 要给队列设置超时时间，需要在声明队列时配置x-message-ttl属性：
```java
@Bean   
public DirectExchange ttlExchange(){    
	return new DirectExchange("ttl.direct");   // 声明TTL交换机Bean
}  
@Bean  
public Queue ttlQueue(){    
	return QueueBuilder.durable("ttl.queue") // 指定队列名称，并持久化  
	          .ttl(10000) // 设置队列的超时时间，10秒         
	          .deadLetterExchange("dl.direct") // 指定死信交换机
	          // 指定死信RoutingKey,声明死信路由键 
	          .deadLetterRoutingKey("dl")            
	          .build();  
}  
@Bean  
public Binding simpleBinding(){    
	return BindingBuilder.bind(ttlQueue())  // 绑定队列
	             .to(ttlExchange())  // 绑定到交换机
	             .with("ttl");  // 指定路由键
}
```
2. 发送消息时，给消息本身设置超时时间
![[Pasted image 20251006201727.png]]
### 延迟队列
利用TTL结合死信交换机，我们实现了消息发出后，消费者延迟收到消息的效果。这种消息模式就称为延迟队列（Delay Queue）模式。

延迟队列的使用场景包括：
- 延迟发送短信
- 用户下单，如果用户在15 分钟内未支付，则自动取消
- 预约工作会议，20分钟后自动通知所有参会人员
因为延迟队列的需求非常多，所以RabbitMQ的官方也推出了一个插件叫[`DelayExchange`](https://blog.rabbitmq.com/posts/2015/04/scheduling-messages-with-rabbitmq)，原生支持延迟队列效果。
## 惰性队列
### 消息堆积问题
当生产者发送消息的速度超过了消费者处理消息的速度，就会导致队列中的消息堆积，直到队列存储消息达到上限。最早接收到的消息，可能就会成为死信，会被丢弃，这就是消息堆积问题。
![[Pasted image 20251006202905.png]]
解决消息堆积有三种种思路：
1. 增加更多消费者，提高消费速度
2. 在消费者内开启线程池加快消息处理速度
(开启线程越大，CPU需要做更多的上下文切换，导致耗费大量资源，适合消息业务处理时间较长的情况)
3. 扩大队列容积，提高堆积上限
### 惰性队列
从RabbitMQ的3.6.0版本开始，就增加了Lazy Queues的概念，也就是惰性队列。

惰性队列的特征如下：
- 接收到消息后直接存入磁盘而非内存
- 消费者要消费消息时才会从磁盘中读取并加载到内存
- 支持数百万条的消息存储

1. 要设置一个队列为惰性队列，只需要在声明队列时，指定x-queue-mode属性为lazy即可。可以通过命令行将一个运行中的队列修改为惰性队列：
```cmd
rabbitmqctl set_policy Lazy "^lazy-queue$" '{"queue-mode":"lazy"}' --apply-to queues
```
2. 用SpringAMQP声明惰性队列分两种方式：
@Bean的方式：
![[Pasted image 20251006203611.png]]
注解方式：
![[Pasted image 20251006203621.png]]
惰性队列的优点有哪些？
- 基于磁盘存储，消息上限高
- (直接写磁盘上，而不是先内存后磁盘)没有间歇性的page-out，性能比较稳定
惰性队列的缺点有哪些？
- 基于磁盘存储，消息时效性会降低
- 性能受限于磁盘的IO
