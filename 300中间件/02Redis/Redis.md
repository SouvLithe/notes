# 初识Redis
## 认识NoSQL
![[Pasted image 20250917200018.png]]
SQL：设计时没有考虑分布式的情况，只能依靠提升本台机器的性能来提升SQL的能力。
NoSQL：考虑到了数据拆分的需求（通过hash运算，依靠结果来判断其存储节点，从而进行数据的拆分）
## 认识Redis
Redis（全称Remote Dictionary Server）远程词典服务器，是一个基于内存的键值型NoSQL数据库。
特征：
- 键值（key-value）型，value支持多种不同的数据结构，功能丰富
- 单线程，每个命令具备原子性（6.0多线程仅仅作用在处理网络处理这块，核心命令执行依然是单线程）
- 延迟低、速度快（**基于内存**、IO多路复用、良好的编码）
- 支持数据的持久化
- 支持主从集群、分片集群
- 支持多语言客户端
Redis默认有16个仓库，编号从0至15. 通过配置文件可以设置仓库数量，但是不超过16，并且不能自定义仓库名称。
# Redis常见命令
Redis安装完成后就自带了命令行客户端：redis-cli，使用方式如下：
```bash
 redis-cli [options] [commonds]
 SELECT [0-16]   选择0-16号库
```
其中常见的options有：
- `-h 127.0.0.1`：指定要连接的redis节点的IP地址，默认是127.0.0.1
- `-p 6379`：指定要连接的redis节点的端口，默认是6379
- `-a 123321`：指定redis的访问密码

其中的commonds就是Redis的操作命令，例如：
- `ping`：与redis服务端做心跳测试，服务端正常会返回`pong`
## 5种常见数据结构
Redis是典型的key-value数据库，key一般是字符串，而value包含很多不同的数据类型：
![[Pasted image 20250917203545.png]]
Redis为了方便我们学习，将操作不同数据类型的命令也做了分组，在[官网][https://redis.io/commands]可以查看到不同的命令
## 通用命令
不同类型的命令称为一个group，我们也可以通过help命令来查看各种不同group的命令：
![[Pasted image 20250917204318.png]]
通用指令是部分数据类型的，都可以使用的指令，常见的有：
- KEYS：查看符合模板的所有key(不建议在大型数据环境中使用，消耗内存)
- DEL：删除一个指定的key
- EXISTS：判断key是否存在
- EXPIRE：给一个key设置有效期，有效期到期时该key会被自动删除（存在内存中，所以要提供这个命令）
  有效期==>-2删除、-1永久有效、整数 剩余存活时间
- TTL：查看一个KEY的剩余有效期

通过help `[command] `可以查看一个命令的具体用法，例如：
```txt
 # 查看keys命令的帮助信息：  
 127.0.0.1:6379> help keys  
 ​  
 KEYS pattern  
 summary: Find all keys matching the given pattern  
 since: 1.0.0  
 group: generic
```
KEYS命令，不建议在数据量特别大的设备上使用（因为其是单线程，模糊查询性能不高会阻塞Redis服务），不要在主节点上执行，可以在从节点上执行
## 不同数据结构的操作命令
### String类型
value是字符串，不过根据字符串的格式不同，又可以分为3类：
- string：普通字符串
- int：整数类型，可以做自增、自减操作
- float：浮点类型，可以做自增、自减操作

不管是哪种格式，底层都是字节数组形式存储，只不过是编码方式不同。字符串类型的最大空间不能超过512m.
数字的话会直接转为二进制编码的形式去存储（节省内存空间）。

常见命令：
- SET：添加或者修改已经存在的一个String类型的键值对
- GET：根据key获取String类型的value
- MSET：批量添加多个String类型的键值对
- MGET：根据多个key获取多个String类型的value
- INCR：让一个整型的key自增1
- INCRBY:让一个整型的key自增并指定步长，例如：incrby num 2 让num值自增2
- INCRBYFLOAT：让一个浮点类型的数字自增并指定步长
- SETNX：添加一个String类型的键值对，前提是这个key不存在，否则不执行
- SETEX：添加一个String类型的键值对，并且指定有效期
### key结构
Redis没有类似MySQL中的Table的概念，我们该如何区分不同类型的key呢？
可以通过给key添加前缀加以区分，不过这个前缀不是随便加的，有一定的规范。

Redis的key允许有多个单词形成层级结构，多个单词之间用':'隔开，格式如下：
```
推荐使用格式：
	项目名:业务名:类型:id
例如：
	user相关的key：heima:user:1
    product相关的key：heima:product:1
```
### Hash类型
Hash类型，也叫散列，其value是一个无序字典，类似于Java中的HashMap结构。
![[Pasted image 20250917213706.png]]
Hash的常见命令有：
- HSET key field value：添加或者修改hash类型key的field的值
    
- HGET key field：获取一个hash类型key的field的值
    
- HMSET：批量添加多个hash类型key的field的值
    
- HMGET：批量获取多个hash类型key的field的值
    
- HGETALL：获取一个hash类型的key中的所有的field和value
    
- HKEYS：获取一个hash类型的key中的所有的field
    
- HINCRBY:让一个hash类型key的字段值自增并指定步长
    
- HSETNX：添加一个hash类型的key的field值，前提是这个field不存在，否则不执行
![[Pasted image 20250917214719.png]]
### List类型
Redis中的List类型与Java中的LinkedList类似，可以看做是一个双向链表结构。既可以支持正向检索和也可以支持反向检索。
特征也与LinkedList类似：
- 有序
- 元素可以重复
- 插入和删除快
- 查询速度一般

常用来存储一个有序数据，例如：朋友圈点赞列表，评论列表等。

List的常见命令有：
- LPUSH key element ... ：向列表左侧插入一个或多个元素（队首）
- LPOP key：移除并返回列表左侧的第一个元素，没有则返回nil
- RPUSH key element ... ：向列表右侧插入一个或多个元素（队尾）
- RPOP key：移除并返回列表右侧的第一个元素
- LRANGE key star end：返回一段角标范围内的所有元素。key（指定链表）
- BLPOP和BRPOP：与LPOP和RPOP类似，只不过在没有元素时等待指定时间，而不是直接返回nil  （**阻塞队列的效果**）
![[Pasted image 20250917215345.png]]
### Set类型
Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap，因为也是一个hash表。
Set的常见命令有：
- SADD key member ... ：向set中添加一个或多个元素
- SREM key member ... : 移除set中的指定元素
- SCARD key： 返回set中元素的个数
- SISMEMBER key member：判断一个元素是否存在于set中
- SMEMBERS：获取set中的所有元素
	
- SINTER key1 key2 ... ：求key1与key2的交集
- SDIFF key1 key2 ... : 求差集（A有B没有）
- SUNION key1 key2 ...: 求并集
### SortedSet类型
SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但**底层数据结构却差别很大**。SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。
常见命令：
- ZADD key score member：添加一个或多个元素到sorted set ，如果已经存在则更新其score值
- ZREM key member：删除sorted set中的一个指定元素
- ZSCORE key member : 获取sorted set中的指定元素的score值
- ZRANK key member：获取sorted set 中的指定元素的排名
- ZCARD key：获取sorted set中的元素个数
- ZCOUNT key min max：统计score值在给定范围内的所有元素的个数
- ZINCRBY key increment member：让sorted set中的指定元素自增，步长为指定的increment值
- ZRANGE key min max：按照score排序后，获取指定排名范围内的元素
- ZRANGEBYSCORE key min max：按照score排序后，获取指定score范围内的元素
- ZDIFF、ZINTER、ZUNION：求差集、交集、并集

注意：所有的排名默认都是升序，如果要降序则在命令的Z后面添加REV即可，例如：
- **升序**获取sorted set 中的指定元素的排名：`ZRANK key member`
- **降序**获取sorted set 中的指定元素的排名：`ZREVRANK key memeber`

# Redis的Java客户端
![[Pasted image 20250917222948.png]]
标记为*的就是推荐使用的java客户端，包括：

- Jedis和Lettuce：这两个主要是提供了Redis命令对应的API，方便我们操作Redis，而SpringDataRedis又对这两种做了抽象和封装，因此我们后期会直接以SpringDataRedis来学习。
    
- Redisson：是在Redis基础上实现了分布式的可伸缩的java数据结构，例如Map、Queue等，而且支持跨进程的同步机制：Lock、Semaphore等待，比较适合用来实现特殊的功能需求。
Redisson会在后期学习  **分布式锁**  的时候再去学习。
## `Jedis`
1. 引入依赖
```xml
<!--jedis-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.7.0</version>
</dependency>
```
2. 建立连接
```java
private Jedis jedis;

@BeforeEach
void setUp() {
    // 1.建立连接
    jedis = new Jedis("192.168.150.101", 6379); //Jedis(IP地址.端口号)
    //配置连接池后，可用这个下面方法获取Bean对象
    // jedis = JedisConnectionFactory.getJedis(); 
    // 2.设置密码
    jedis.auth("123321");
    // 3.选择库
    jedis.select(0);
}
```
3. 测试string（使用）
```java
@Test
void testString() {
    // 存入数据
    String result = jedis.set("name", "虎哥");
    System.out.println("result = " + result);
    // 获取数据
    String name = jedis.get("name");
    System.out.println("name = " + name);
}

@Test
void testHash() {
    // 插入hash数据
    jedis.hset("user:1", "name", "Jack");
    jedis.hset("user:1", "age", "21");

    // 获取
    Map<String, String> map = jedis.hgetAll("user:1");
    System.out.println(map);
}
```
如果没有指定访问修饰符，成员（包括方法和变量）的访问权限是  **包级私有**。
意味着：只有同一个包下的类可以访问这个成员。
4. 释放资源
```java
@AfterEach
void tearDown() {
    if (jedis != null) {
        jedis.close();
    }
}
```
2、3、4是在同一个测试类中的代码。
使用Jedis时，其方法名称和Redis命令一致。
### Jedis连接池
Jedis本身是线程不安全的，并且频繁的创建和销毁连接会有性能损耗，因此我们推荐大家使用Jedis连接池代替Jedis的直连方式。
```java
package com.heima.jedis.util;

import redis.clients.jedis.*;

public class JedisConnectionFactory {

    private static JedisPool jedisPool;

    static {
        // 配置连接池
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        //最大连接
        poolConfig.setMaxTotal(8);
        //最大空闲连接
        poolConfig.setMaxIdle(8);
        //最小空闲连接
        poolConfig.setMinIdle(0);
        //设置最长等待时间，ms
        poolConfig.setMaxWaitMillis(1000);
        // 创建连接池对象，参数：连接池配置、服务端ip、服务端端口、超时时间、密码
        jedisPool = new JedisPool(poolConfig, "192.168.150.101", 6379, 1000, "123321");
    }

//每当调用这个方法，就可以拿Jedis对象
    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
```
## `SpringDataRedis`
SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做SpringDataRedis。
- 提供了对不同Redis客户端的整合（Lettuce和Jedis）
- 提供了RedisTemplate统一API来操作Redis
- 支持Redis的发布订阅模型
- 支持Redis哨兵和Redis集群
- 支持基于Lettuce的响应式编程
- 支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
- 支持基于Redis的JDKCollection实现

支持数据序列化及反序列化是因为：Jedis的方法参数类型要求的是String类型，没办法传Object对象类型，这时就要对对象进行序列化，转为字符串 或 字节

SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：
![[Pasted image 20250917230643.png]]
## 实践
SpringDataRedis的使用步骤：
- 引入spring-boot-starter-data-redis依赖
- 在application.yml配置Redis信息
- 注入RedisTemplate
RedisTemplate使用的是JDK的序列化工具，默认使用的是OutputStream序列化的。
此外还有：
StringRedisSerializer就是管理转字符串的，因为底层只需要使用getBytes就可以了，key一般用的就是这个。
GenericJackson2JsonRedisSerializer这个可以将队象给序列化。

所以自定义RedisTemplate的序列化方式，代码如下：
```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory){
        // 创建RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置连接工厂
        template.setConnectionFactory(connectionFactory);
        // 创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = 
            							new GenericJackson2JsonRedisSerializer();
        // 设置Key的序列化
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // 设置Value的序列化
        template.setValueSerializer(jsonRedisSerializer);
        template.setHashValueSerializer(jsonRedisSerializer);
        // 返回
        return template;
    }
}
```
json序列化时，会自动写一个@class的属性，里面是实体类的字节码名称。
![[Pasted image 20250920160331.png]]
所以反序列化的时候才能读取到类的名称，帮助把json反序列化为User。
### `StringRedisTemplate`
尽管JSON的序列化方式可以满足我们的需求，但依然存在一些资源问题：
为了在反序列化时知道对象的类型，JSON序列化器会将类的class类型写入json结果中，存入Redis，会带来额外的内存开销。
为了减少内存的消耗，可以采用手动序列化，换句话说，就是不借助默认的序列化器，而是自己来控制序列化的动作，同时，只采用String的序列化器，这样，在存储value时，就不需要在内存中就不用多存储数据，从而节约的内存空间
![[Pasted image 20250920161445.png]]
一句话：还用String存，做一个工具类来处理JSON和String的转换（将转换标志从Redis内转到java中）
### 从对象中获取值插入Redis
还可以从对象中获得一个个的值，从而像操作Redis-cli那样一个个插入，不过这不是set而是put，其余的去百度一下吧。
![[Pasted image 20250920163254.png]]
# Redis企业实战
全图一览：
![[Pasted image 20250920164050.png]]
架构一览：
![[Pasted image 20250920165353.png]]
要考虑并发能力，也就是水平扩展能力（负载均衡集群）。
## 短信登录
### 基于Session实现登录
![[Pasted image 20250922224724.png]]
Session是基于Cookie的，每一个Session都会有一个SessionId保存在浏览器的Cookie当中，用户访问时，会携带对应的Cookie里面就有Session Id
#### 校验&登录
之前的设计思路无法适应多用户访问的情况（针对每一个用户设计Controller太复杂），
于是，可以使用拦截器进行校验，但需要将拦截器拦截到的信息传到相应的Controller中去，
于是，可以使用ThreadLocal实现这一点（注意线程安全）
![[Pasted image 20250923132015.png]]
#### 返回信息过多
存储的粒度问题，粒度越大，信息越完整，使用越方便，但内存压力同时也越大，敏感字段会返沪前端。
### 集群的session共享问题
session共享问题：多台Tomcat并不共享session存储空间，当请求切换到不同tomcat服务时导致数据丢失的问题。
![[Pasted image 20250923141838.png]]
session的替代方案应该满足（redis：就差点我名字了）：
- 数据共享
- 内存存储
- key、value结构
#### 基于Redis实现共享session登录
保存登录的用户信息，可以使用String结构，以JSON字符串来保存，比较直观：
![[Pasted image 20250923142515.png]]
Hash结构可以将对象中的每个字段独立存储，可以针对单个字段做CRUD，并且内存占用更少：
![[Pasted image 20250923142553.png]]
推荐使用hash结构
发送验证码：
![[Pasted image 20250923143027.png]]
校验登录：
![[Pasted image 20250923143234.png]]
不能使用手机号返回原因：前端可以响应数据中查到
前端如何携带token？
拿到后端返回的数据data，并将其存入sessionStorage中，然后再让axios从中拿到taken装入请求头，后端就可以从请求头中拿到taken。
登录拦截器的优化：
![[Pasted image 20250923183454.png]]
把原来的拦截器分为两个一个负责刷新，一个负责拦截
## 商户查询缓存
缓存就是数据交换的缓冲区（称作Cache），是存贮数据的临时地方，一般读写性能较高。
缓存的作用:
	降低后端负载
	提高读写效率，降低响应时间
缓存的成本:
	数据一致性成本(缓存和数据库的数据不一致)
	代码维护成本（代码复杂度上升）
	运维成本（缓存雪崩`<让缓存高可用>`）
### 添加Redis缓存
![[Pasted image 20250923221705.png]]
### 缓存更新策略
![[Pasted image 20250923224059.png]]
业务场景：
- 低一致性需求：使用内存淘汰机制。例如店铺类型的查询缓存
- 高一致性需求：主动更新，并以超时剔除作为兜底方案。例如店铺详情查询的缓存
主动更新：
![[Pasted image 20250923224449.png]]
操作缓存和数据库时有三个问题需要考虑：
1.删除缓存还是更新缓存？
- 更新缓存：每次更新数据库都更新缓存，无效写操作较多
- 删除缓存：更新数据库时让缓存失效，查询时再更新缓存（推荐这个）
2.如何保证缓存与数据库的操作的同时成功或失败？
- 单体系统，将缓存与数据库操作放在一个事务
- 分布式系统，利用TCC等分布式事务方案 
3.先操作缓存还是先操作数据库？
- 先删除缓存，再操作数据库
- 先操作数据库，再删除缓存（出问题更概率低）
### 缓存穿透
缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求都会打到数据库。（只要查询就一定会打到数据库）
常见的解决方案有两种：
缓存空对象
	优点：实现简单，维护方便(一般用这个)
	缺点：
	1. 额外的内存消耗
	2.可能造成短期的不一致
布隆过滤：把数据通过hash算法，计算成一种hash值，将这些hash值转换成二进制位存入布隆过滤器，判断是否存在时，就是判断对应的位置是0还是1，以此来判断数据是否存在。（这是一种概率上的统计不一定准确——它说不存在一定不存在，说存在也不一定存在）
	优点：内存占用较少，没有多余key
	缺点：实现复杂、存在误判可能
![[Pasted image 20250923231118.png]]
![[Pasted image 20250923232813.png]]
### 缓存雪崩
缓存雪崩是指在同一时段大量的缓存key同时失效或者Redis服务宕机，导致大量请求到达数据库，带来巨大压力。
解决方案：
- 给不同的Key的TTL添加随机值
- 利用Redis集群提高服务的可用性（哨兵机制，主从集群机器，从代主）
- 给缓存业务添加降级限流策略
- 给业务添加多级缓存
![[Pasted image 20250923233020.png]]
### 缓存击穿
缓存击穿问题也叫热点Key问题，就是一个被  **高并发访问**  并且  **缓存重建业务较复杂**  的key突然失效了，无数的请求访会在瞬间给数据库带来巨大的冲击。
![[Pasted image 20250923233705.png]]
常见解决方案：
	逻辑过期、互斥锁
![[Pasted image 20250927191144.png]]
这两种方式也是分布式系统中经常面临的一个问题：保证一致性还是可用性。
### 缓存工具封装
基于StringRedisTemplate封装一个缓存工具类，满足下列需求：
- 方法1：将任意Java对象序列化为json并存储在string类型的key中，并且可以设置TTL过期时间
- 方法2：将任意Java对象序列化为json并存储在string类型的key中，并且可以设置逻辑过期时间，用于处理缓存击穿问题
- 方法3：根据指定的key查询缓存，并反序列化为指定类型，利用缓存空值的方式解决缓存穿透问题
- 方法4：根据指定的key查询缓存，并反序列化为指定类型，需要利用逻辑过期解决缓存击穿问题
1、3针对普通缓存，2、4针对的是热点缓存。

设置逻辑过期，并不是真的过期，  而是设置个带有（JSON对象封装的data、还有一个私有变量记录时间）的一个实体类，利用这个实体类中的私有过期时间进行逻辑时间处理

函数式编程：
`Function<ID,R> dbFallback` 参数ID 返回值R
## 优惠券秒杀
### 全局唯一ID
当用户抢购时，就会生成订单并保存到tb_voucher_order这张表中，而订单表如果使用数据库自增ID就存在一些问题：
- id的规律性太明显
- 受单表数据量的限制
实现方式：
	利用Redis自增(推荐，满足下面五个)，JDK自带的工具类UUID,snowflake算法,
	数据库自增（利用数据库某一张表自增来代替Redis自增）
Redis自增ID策略：
- 每天一个key，方便统计订单量
- ID构造是 时间戳 + 计数器
全局ID生成器，是一种在分布式系统下用来生成全局唯一ID的工具，一般要满足下列特性：
![[Pasted image 20250928133551.png]]
### 实现优惠券秒杀下单
#### 超卖下单
高并发出现的问题。
![[Pasted image 20250928151623.png]]
超卖问题是典型的多线程安全问题，针对这一问题的常见解决方案就是加锁：
![[Pasted image 20250928151724.png]]
乐观锁的性能比悲观锁性能要好一点。
但乐观锁成功率太低。
#### 如何得知在我之前有没有人修改？
有两种方法：
版本号：在id修改的基础上维系一个版本号，每次访问加一，初始版本为0，通过比较版本号是否唯一，来确认修改情况。
![[Pasted image 20250928152000.png]]
CAS法：简化版本号法
利用库存代替版本：先查询stock的值是否和库中的在一致，若是而后减一，不是则 不会执行该方法。
![[Pasted image 20250928152433.png]]
#### 乐观锁成功率太低。
解决方案：
- 若乐观锁的库中数据是库存，可以有极值，只需判断库存是否大于0就行，没必要非得相等
- 若乐观锁的只能通过判断数据是否相等来判断，可以使用分段锁（把资源分成几份，可以去多张表中分别去抢锁）
### 一人一单
在这两个节点上负载均衡，再次测试下是否存在线程安全问题。
原因是：负载均衡在集群模式下会创建两个JVM对象，而一个JVM对象只有一个锁监视器。
![[Pasted image 20250928185434.png]]
### 分布式锁
分布式锁：满足分布式系统或集群模式下多进程可见并且互斥的锁。
![[Pasted image 20250928185829.png]]
分布式锁特性：多进程可见、互斥、高可用、高性能、安全性（基本特征）

分布式锁的核心是实现多进程之间互斥，而满足这一点的方式有很多，常见的有三种：
![[Pasted image 20250928190110.png]]
#### 基于Redistribution实现：
获取锁：
- 互斥：确保只能有一个线程获取锁
- 非阻塞：尝试一次，成功返回true，失败返回false
```Redis
# 添加锁，利用setnx的互斥特性
SETNX lock thread1
# 添加锁过期时间，避免服务宕机引起的死锁
EXPIRE lock 10
# 添加锁，NX是互斥、EX是设置超时时间
SET lock thread1 NX EX 10
```
释放锁：
- 手动释放
- 超时释放：获取锁时添加一个超时时间
```Redis
# 释放锁，删除即可
DEL key
```
#### Redis锁的互删问题（超时释放锁造成）
![[Pasted image 20250928194738.png]]
跟之前的区别：
1.在获取锁时存入线程标示（可以用UUID表示）
2.在释放锁时先获取锁中的线程标示，判断是否与当前线程标示一致
- 如果一致则释放锁
- 如果不一致则不释放锁
### 分布式锁的原子性问题（也是超时释放锁）：
因为判断已经过了，卡在删除的位置(因为JVM中GC的阻塞机制)
Redis提供了[[Lua]]脚本功能，在一个脚本中编写多条Redis命令，确保多条命令执行时的原子性。Lua是一种编程语言，它的基本语法大家可以参考网站：[菜鸟教程](https://www.runoob.com/lua/lua-tutorial.html)
![[Pasted image 20251001104221.png]]
释放锁的时候读取lua文件，还是提前读取好点？
提前读取好点，因为其没有频繁的IO流(减少磁盘IO)，性能好点。

静态代码块就是类加载的时候会被执行一次，就是最好的初始化方法
是这个解决方案是针对`Java jvm 的阻塞问题`的解决。
### `Redisson`
基于sentx实现的分布式锁存在下面的问题：
![[Pasted image 20251001104728.png]]
Redisson是一个在Redis的基础上实现的Java驻内存数据网格（In-Memory Data Grid）。它不仅提供了一系列的分布式的Java常用对象，还提供了许多分布式服务，其中就包含了各种分布式锁的实现。
之后使用分布式锁，直接使用Redission这个框架就行了。
#### 使用注意
配置Redisson客户端：
```java
@Configuration  
public class RedisConfig {    
@Bean    
public RedissonClient redissonClient() {        
	// 配置类        
	Config config = new Config();        
	// 添加redis地址，这里添加了单点的地址，也可以使用
	// config.useClusterServers()添加集群地址
	config.useSingleServer().
	     setAddress("redis://192.168.150.101:6379").setPassowrd("147258");        
	// 创建客户端        
	return Redisson.create(config);  
    }  
}
```
可以利用上面这种java配置方式实现（自定义配置），
也可以使用yml文件和springboot整合实现（但这种并不推荐使用，因为它会替代spring官方提供的Redis的配置和实现）
使用Redission的分布式锁：
```java
@Resource  
private RedissonClient redissonClient;  
  
@Test  
void testRedisson() throws InterruptedException {    
	// 获取锁（可重入），指定锁的名称    
	RLock lock = redissonClient.getLock("anyLock");    
	// 尝试获取锁，参数分别是：获取锁的最大等待时间（期间会重试），锁自动释放时间，时间单位
	boolean isLock = lock.tryLock(1, 10, TimeUnit.SECONDS);    
	// 判断释放获取成功    
	if(isLock){        
		try {  
            System.out.println("执行业务");  
        }finally {            
	        // 释放锁            
	        lock.unlock();  
        }  
    }  
}
```
#### 可重入锁
获取锁：
![[Pasted image 20251001113056.png]]
第二个if通过threadId判断其是否存在于这个Hash结构中就可以判断锁标识是不是自己的。
释放锁：
![[Pasted image 20251001113202.png]]
#### 优化原理
![[Pasted image 20251001115536.png]]
Redisson分布式锁原理：
- **可重入**：利用hash结构记录  线程id  和  重入次数 
- **可重试**：利用信号量和PubSub功能实现等待、唤醒，获取锁失败的重试机制（并不是无限制的重试，会有一个等待的时间，超过就不再重试了；所用的等待唤醒机制不会过多占用CPU）
- **超时续约**：利用watchDog，每隔一段时间（releaseTime / 3），重置超时时间

主从一致性问题：
![[Pasted image 20251001120700.png]]
解决方式：
![[Pasted image 20251001120559.png]]
其实就是，创建联锁 `multiLock`
#### 小结
1. 不可重入Redis分布式锁：
- 原理：利用setnx的互斥性；利用ex避免死锁；释放锁时判断线程标示
- 缺陷：不可重入、无法重试、锁超时失效
2. 可重入的Redis分布式锁：
- 原理：利用hash结构，记录线程标示和重入次数；利用watchDog延续锁时间；利用信号量控制锁重试等待
- 缺陷：redis宕机引起锁失效问题
3. Redisson的multiLock：
- 原理：多个独立的Redis节点，必须在所有节点都获取重入锁，才算获取锁成功
- 缺陷：运维成本高、实现复杂
### 优化秒杀<有点乱>
![[Pasted image 20251001132218.png]]查数据库时的并发能力本来就是比较弱的（减库存和创建订单还是写操作），并且还加了分布式锁，整个业务性能可想而知。
解决方法：
将不需要访问数据库的操作  提取解耦  出来放入Redis（写入Lua脚本保证原子性），将其余需要对数据库进行访问的写入java业务层，
业务层多线程地调用Lua（Redis）脚本，再在处理过后存入阻塞队列异步访问数据库，从而提升性能。
![[Pasted image 20251001133105.png]]
可实现异步访问数据库（原来是同步）。

秒杀业务的优化思路是什么？
1. 先利用Redis完成库存余量、一人一单判断，完成抢单业务
2. 再将下单业务放入阻塞队列，利用独立线程异步下单

基于阻塞队列的异步秒杀存在哪些问题？
- 内存限制问题
我们用的是JDK中的阻塞队列，这个阻塞队列使用的是JVM的内存，
如果不加以限制，在高并发情况下 可能导致内存溢出
- 数据安全问题（任务丢失、用户信息丢失）
#### 解决方法 => 消息队列
![[Pasted image 20251001153921.png]]
MQ较阻塞队列的优点：
1. MQ是JVM以外的一个独立的服务，不受JVM内存的限制
2. MQ不仅仅做数据存储，还保证了消息安全（给消费者后要求消费者确认）

Redis提供了三种不同的方式来实现消息队列：
- list结构：基于List结构模拟消息队列
```text
基于List的消息队列有哪些优缺点？
优点：
•利用Redis存储，不受限于JVM内存上限
•基于Redis的持久化机制，数据安全性有保证
•可以满足消息有序性
缺点：
•无法避免消息丢失
•只支持单消费者
```
- PubSub：基本的点对点消息模型
```text
基于PubSub的消息队列有哪些优缺点？
优点：
•采用发布订阅模型，支持多生产、多消费
缺点：
•不支持数据持久化
•无法避免消息丢失
•消息堆积有上限，超出时数据丢失
```
- Stream：比较完善的消息队列模型
Stream 是 Redis 5.0 引入的一种新数据类型，可以实现一个功能非常完善的消息队列。
![[Pasted image 20251001160118.png]]
![[Pasted image 20251001160052.png]]
注意：
	当我们指定起始ID为$时，代表读取最新的消息，如果我们处理一条消息的过程中，又有超过1
	条以上的消息到达队列，则下次获取时也只能获取到最新的一条，会出现漏读消息的问题。
```text
STREAM类型消息队列的XREAD命令特点：
消息可回溯
一个消息可以被多个消费者读取
可以阻塞读取
有消息漏读的风险
```
#### 基于Stream的消息队列-消费者组
消费者组（Consumer Group）：将多个消费者划分到一个组中，监听同一个队列。
![[Pasted image 20251001161113.png]]
创建消费者组：
![[Pasted image 20251001161255.png]]
从消费者组读取消息：
![[Pasted image 20251001161333.png]]
三者优势比较：

| 特性     | List                 | PubSub    | Stream                      |
| ------ | -------------------- | --------- | --------------------------- |
| 消息持久化  | 支持                   | 不支持       | 支持                          |
| 阻塞读取   | 支持                   | 支持        | 支持                          |
| 消息堆积处理 | 受限于内存空间，可以利用多消费者加快处理 | 受限于消费者缓冲区 | 受限于队列长度，可以利用消费者组提高消费速度，减少堆积 |
| 消息确认机制 | 不支持                  | 不支持       | 支持                          |
| 消息回溯   | 不支持                  | 不支持       | 支持                          |
## 达人探店
```java
//当前字段不属于Blog表  
@TableField(exist = false)
```
包装类可能为空，用BooleanUtil工具判断是否为真或是否有值。
使用map方法将每个元素转换为对应的Long类型，并使用collect方法将结果收集到一个List中。
## 关注推送
关注推送也叫做Feed流，直译为投喂。为用户持续的提供“沉浸式”的体验，通过无限下拉刷新获取新的信息。

Feed流产品有两种常见模式：
1. Timeline：不做内容筛选，简单的按照内容发布时间排序，常用于好友或关注。例如朋友圈
优点：信息全面，不会有缺失。并且实现也相对简单
缺点：信息噪音较多，用户不一定感兴趣，内容获取效率低
实现方法：拉模式、推模式、推拉结合
![[Pasted image 20251002085203.png]]
2. 智能排序：利用智能算法屏蔽掉违规的、用户不感兴趣的内容。推送用户感兴趣信息来吸引用户
优点：投喂用户感兴趣信息，用户粘度很高，容易沉迷
缺点：如果算法不精准，可能起到反作用

按角标分页，在分页过程中有数据插入导致数据混乱。
滚动分页则不会。(注意高并发的时候可能会出现时间戳一致的情况，这时可能会出现错误)
```text
滚动分页查询参数：优化后 || 优化前
max：  上一次查询的最小时间戳    || 当前时间戳
min：    0
offset：   与上一次查询最小时间戳一致的所有元素个数    ||  0
count：  3
```
时间戳的最大值是当前时间戳。
优化前问题：
1. 如果有在每页后面重复时间戳，则会出现重复读的情况 
2. 优化后其实还有问题，如果重复时间戳数量比两次分页查询数量都多的话就不会往下走了
Tips：
直接时间戳加用户唯一id拼接就完了
把时间戳后面拼上userid，也就除了时间排序外，相同时间发布的还以用户id排序。就能解决，也不会重复
## 附近商户
### GEO数据结构
GEO就是Geolocation的简写形式，代表地理坐标。Redis在3.2版本中加入了对GEO的支持，允许存储地理坐标信息，帮助我们根据经纬度来检索数据。常见的命令有：
- [GEOADD](https://redis.io/commands/geoadd)：添加一个地理空间信息，包含：经度（longitude）、纬度（latitude）、值（member）
- [GEODIST](https://redis.io/commands/geodist)：计算指定的两个点之间的距离并返回
- [GEOHASH](https://redis.io/commands/geohash)：将指定member的坐标转为hash字符串形式并返回
- [GEOPOS](https://redis.io/commands/geopos)：返回指定member的坐标
- [GEORADIUS](https://redis.io/commands/georadius)：指定圆心、半径，找到该圆内包含的所有member，并按照与圆心之间的距离排序后返回。6.2以后已废弃
- [GEOSEARCH](https://redis.io/commands/geosearch)：在指定范围内搜索member，并按照与指定点之间的距离排序后返回。范围可以是圆形或矩形。6.2.新功能（常用）
- [GEOSEARCHSTORE](https://redis.io/commands/geosearchstore)：与GEOSEARCH功能一致，不过可以把结果存储到一个指定的key。 6.2.新功能
### 附近商户搜索
按照商户类型做分组，类型相同的商户作为同一组，以typeId为key存入同一个GEO集合中即可
## 用户签到
我们按月来统计用户签到信息，签到记录为1，未签到则记录为0.
把每一个bit位对应当月的每一天，形成了映射关系。用0和1标示业务状态，这种思路就称为位图（BitMap）。
Redis中是利用string类型数据结构实现BitMap，因此最大上限是512M，转换为bit则是 2^32个bit位。


Redis中是利用string类型数据结构实现BitMap，因此最大上限是512M，转换为bit则是 2^32个bit位。
BitMap的操作命令有：
- [SETBIT](https://redis.io/commands/setbit)：向指定位置（offset）存入一个0或1
- [GETBIT](https://redis.io/commands/getbit) ：获取指定位置（offset）的bit值
- [BITCOUNT](https://redis.io/commands/bitcount) ：统计BitMap中值为1的bit位的数量
- [BITFIELD](https://redis.io/commands/bitfield) ：操作（查询、修改、自增）BitMap中bit数组中的指定位置（offset）的值
- [BITFIELD_RO](https://redis.io/commands/bitfield_ro) ：获取BitMap中bit数组，并以十进制形式返回
- [BITOP](https://redis.io/commands/bitop) ：将多个BitMap的结果做位运算（与 、或、异或）
- [BITPOS](https://redis.io/commands/bitpos) ：查找bit数组中指定范围内第一个0或1出现的位置
![[Pasted image 20251002142223.png]]
问题1：什么叫做连续签到天数？
从最后一次签到开始向前统计，直到遇到第一次未签到为止，计算总的签到次数，就是连续签到天数。
## UV
UV：全称Unique Visitor，也叫独立访客量，是指通过互联网访问、浏览这个网页的自然人。1天内同一个用户多次访问该网站，只记录1次。
PV：全称Page View，也叫页面访问量或点击量，用户每访问网站的一个页面，记录1次PV，用户多次打开页面，则记录多次PV。往往用来衡量网站的流量。

UV统计在服务端做会比较麻烦，因为要判断该用户是否已经统计过了，需要将统计过的用户信息保存。但是如果每个访问的用户都保存到Redis中，数据量会非常恐怖。
### HyperLogLog用法
`Hyperloglog(HLL)`是从Loglog算法派生的概率算法，用于确定非常大的集合的基数，而不需要存储其所有值。相关算法原理大家可以参考：
[`HyperLogLog` 算法的原理讲解以及 Redis 是如何应用它的](https://juejin.cn/post/6844903785744056333)
Redis中的HLL是基于string结构实现的，单个HLL的内存永远小于16kb，内存占用低的令人发指！作为代价，其测量结果是概率性的，有小于0.81％的误差。不过对于UV统计来说，这完全可以忽略。
![[Pasted image 20251002150608.png]]
# 后记
后接[[Redis高级篇]]


