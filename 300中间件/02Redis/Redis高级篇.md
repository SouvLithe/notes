# 分布式缓存
单点Redis的问题：
![[Pasted image 20251002152452.png]]
## Redis持久化
### RDB：
RDB全称Redis Database Backup file（Redis数据备份文件），也被叫做Redis数据快照。简单来说就是把内存中的所有数据都记录到磁盘中。当Redis实例故障重启后，从磁盘读取快照文件，恢复数据。
快照文件称为RDB文件，默认是保存在当前运行目录。
![[Pasted image 20251002152912.png]]
Redis停机时会执行一次RDB。
Redis内部有触发RDB的机制，可以在redis.conf文件中找到，格式如下：
```conf
# 900秒内，如果至少有1个key被修改，则执行bgsave ， 如果是save "" 则表示禁用RDB
save 900 1 
save 300 10 
save 60 10000

# 是否压缩 ,建议不开启，压缩也会消耗cpu，磁盘的话不值钱
rdbcompression yes
# RDB文件名称
dbfilename dump.rdb 
# 文件保存的路径目录
dir ./
```
- `bgsave:`
bgsave开始时会fork主进程得到子进程，子进程共享主进程的内存数据。完成fork后读取内存数据并写入 RDB 文件。
![[Pasted image 20251002155123.png]]
fork采用的是copy-on-write技术：
•当主进程执行读操作时，访问共享内存；
•当主进程执行写操作时，则会拷贝一份数据，执行写操作。
#### 小结
RDB方式bgsave的基本流程？
- fork主进程得到一个子进程，共享内存空间
- 子进程读取内存数据并写入新的RDB文件
- 用新RDB文件替换旧的RDB文件。

RDB会在什么时候执行？save 60 1000代表什么含义？
- 默认是服务停止时。
- 代表60秒内至少执行1000次修改则触发RDB

RDB的缺点？
- RDB执行间隔时间长，两次RDB之间写入数据有丢失的风险
- fork子进程、压缩、写出RDB文件都比较耗时
### AOF
AOF全称为Append Only File（追加文件）。Redis处理的每一个写命令都会记录在AOF文件，可以看做是命令日志文件。
AOF默认是关闭的，需要修改redis.conf配置文件来开启AOF：
```conf
# 是否开启AOF功能，默认是no
appendonly yes
# AOF文件的名称
appendfilename "appendonly.aof"

# 表示每执行一次写命令，立即记录到AOF文件
appendfsync always
# 写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件，是默认方案
appendfsync everysec
# 写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘(性能最好，效率最高不如RDB)
appendfsync no
```
它有大量的冗余数据。
![[Pasted image 20251002160509.png]]
### 小结
RDB和AOF各有自己的优缺点，如果对数据安全性要求较高，在实际开发中往往会结合两者来使用。

| 特性             | RDB                                   | AOF                                   |
|------------------|---------------------------------------|---------------------------------------|
| 持久化方式       | 定时对整个内存做快照                   | 记录每一次执行的命令                   |
| 数据完整性       | 不完整，两次备份之间会丢失             | 相对完整，取决于刷盘策略               |
| 文件大小         | 会有压缩，文件体积小                   | 记录命令，文件体积很大                 |
| 宕机恢复速度     | 很快                                   | 慢                                     |
| 数据恢复优先级   | 低，因为数据完整性不如AOF              | 高，因为数据完整性更高                 |
| 系统资源占用     | 高，大量CPU和内存消耗                  | 低，主要是磁盘IO资源                   |
|                  |                                       | 但AOF重写时会占用大量CPU和内存资源     |
| 使用场景         | 可以容忍数分钟的数据丢失，追求更快的启动速度 | 对数据安全性要求较高常见               |
## Redis主从
搭建主从集群，实现读写分离。
![[Pasted image 20251002160951.png]]
slave（5.0前）和replica（5.0后）都是指的从结点
后面教了一下怎么搭建Redis主从。（可以在redis-cli中用命令INFO replication查看集群状态）

假设有A、B两个Redis实例，如何让B作为A的slave节点？
- 在B节点执行命令：slaveof A的IP A的port
### 数据同步原理
主从第一次同步是全量同步：
![[Pasted image 20251002165611.png]]
master如何判断slave是不是第一次来同步数据？这里会用到两个很重要的概念：
- **`Replication Id`**：简称replid，是数据集的标记，id一致则说明是同一数据集。每一个master都有唯一的replid，slave则会继承master节点的replid
- **`offset`**：偏移量，随着记录在repl_baklog中的数据增多而逐渐增大。slave完成同步时也会记录当前同步的offset。如果slave的offset小于master的offset，说明slave数据落后于master，需要更新。
因此slave做数据同步，必须向master声明自己的replication id 和offset，master才可以判断到底需要同步哪些数据
![[Pasted image 20251002165932.png]]
 第一次连接时，从结点的replid与主结点的不一致 
 ```
 简述全量同步的流程？
•slave节点请求增量同步
•master节点判断replid，发现不一致，拒绝增量同步
•master将完整内存数据生成RDB，发送RDB到slave
•slave清空本地数据，加载master的RDB
•master将RDB期间的命令记录在repl_baklog，并持续将log中的命令发送给slave
•slave执行接收到的命令，保持与master之间的同步
 ```

增量同步：
主从第一次同步是全量同步，但如果slave重启后同步，则执行增量同步:
![[Pasted image 20251002170620.png]]
注意：
- repl_baklog底层是数组，采用的是循环写，即超出数组部分会从数组0索引处覆盖原始数据继续写
- 当从结点太久没有从宕机状态回归的话，从结点的offset会在repl_baklog找不到对应offset数据，这时会再次进行一次全量同步。

可以从以下几个方面来优化Redis主从就集群：
- 在master中配置repl-diskless-sync yes启用无磁盘复制，避免全量同步时的磁盘IO。（不走磁盘IO、走网络IO）
- Redis单节点上的内存占用不要太大，减少RDB导致的过多磁盘IO
- 适当提高repl_baklog的大小，发现slave宕机时尽快实现故障恢复，尽可能避免全量同步
- 限制一个master上的slave节点数量，如果实在是太多slave，则可以采用主-从-从链式结构，减少master压力
![[Pasted image 20251002171320.png]]
## Redis哨兵
Redis提供了哨兵（Sentinel）机制来实现主从集群的自动故障恢复。哨兵的结构和作用如下：
- 监控：Sentinel 会不断检查您的master和slave是否按预期工作
- 自动故障恢复：如果master故障，Sentinel会将一个slave提升为master。当故障实例恢复后也以新的master为主
- 通知：Sentinel充当Redis客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给Redis的客户端
![[Pasted image 20251002171705.png]]
1. Sentinel基于心跳机制监测服务状态，每隔1秒向集群的每个实例发送ping命令：
通过ping-pong命令检测状态（主观下线），当多数sentinel认为你挂掉了（客观下线）就会去选新的主结点。
![[Pasted image 20251002171935.png]]
2. sentinel在salve中选择新master的依据是这样的：
端口时间长了不选 > 选slave-priority值最小的哪个,为0永不参与 > 运行id大小，越小优先级越高。
3. 如何实现故障转移
- sentinel给备选的slave1节点发送slaveof no one命令，让该节点成为master
- sentinel给所有其它slave发送slaveof 192.168.150.101 7002 命令，让这些slave成为新master的从节点，开始从新的master上同步数据。
- 最后，sentinel将故障节点标记为slave，当故障节点恢复后会自动成为新的master的slave节点
通知从变主，告知其余从结点主节点信息引导更换，故障主结点转为从结点

## `RedisTemplate`
在JAVA代码中创建Redis哨兵机制和进行Redis分片集群。
## Redis分片集群
将原来的一个主结点进行功能封装封装，（解耦）打散成多个主节点

主从和哨兵可以解决高可用、高并发读的问题。但是依然有两个问题没有解决：
- 海量数据存储问题
- 高并发写的问题
使用分片集群可以解决上述问题，分片集群特征：
- 集群中有多个master，每个master保存不同数据
- 每个master都可以有多个slave节点
- master之间通过ping监测彼此健康状态
- 客户端请求可以访问集群任意节点，节点之间会做一个自动的路由，最终都会被转发到正确节点
### 散列插槽
数据key不是与节点绑定，而是与插槽绑定。redis会根据key的有效部分计算插槽值，分两种情况：
- key中包含"{}"，且“{}”中至少包含1个字符，“{}”中的部分是有效部分
- key中不包含“{}”，整个key都是有效部分
例如：key是num，那么就根据num计算，如果是`{itcast}num`，则根据itcast计算。计算方式是利用CRC16算法得到一个hash值，然后对16384取余，得到的结果就是slot值。

通过将键映射到特定的哈希槽，再将哈希槽分配给不同的节点(Redis服务器),节点存储数据.
### 集群伸缩
就是动态地扩展和减少主节点和从结点。
### 故障转移
自动转移和手动转移两种
当集群中有一个master宕机会发生什么呢？
1. 首先是该实例与其它实例失去连接
2. 然后是疑似宕机
3. 最后是确定下线，自动提升一个slave为新的master

利用cluster failover命令可以手动让集群中的某个master宕机，切换到执行cluster failover命令的这个slave节点，实现无感知的数据迁移。其流程如下：
手动的Failover支持三种不同模式：
- 缺省：默认的流程，如图1~6歩
- force：省略了对offset的一致性校验
- takeover：直接执行第5歩，忽略数据一致性、忽略master状态和其它master的意见
![[Pasted image 20251002180735.png]]
# 多级缓存
传统的缓存策略一般是请求到达Tomcat后，先查询Redis，如果未命中则查询数据库，存在下面的问题：
- 请求要经过Tomcat处理，Tomcat的性能成为整个系统的瓶颈
- Redis缓存失效时，会对数据库产生冲击
![[Pasted image 20251002181323.png]]
多级缓存就是充分利用请求处理的每个环节，分别添加缓存，减轻Tomcat压力，提升服务性能：
静态资源可以缓存起来，动态资源不得不访问到服务器端。
![[Pasted image 20251002181656.png]]
## JVM进程缓存
Mysql数据库最好控制一下表的大小，这个会影响查询效率
`TomCat`是JVM的进程缓存。

进程缓存时不能共享的

缓存在日常开发中启动至关重要的作用，由于是存储在内存中，数据的读取速度是非常快的，能大量减少对数据库的访问，减少数据库的压力。我们把缓存分为两类：

- 分布式缓存，例如Redis：
1. 优点：存储容量更大、可靠性更好、可以在集群间共享
2. 缺点：访问缓存有网络开销
3. 场景：缓存数据量较大、可靠性要求较高、需要在集群间共享

- 进程本地缓存，例如HashMap、GuavaCache：
1. 优点：读取本地内存，没有网络开销，速度更快
2. 缺点：存储容量有限、可靠性较低、无法共享
3. 场景：性能要求较高，缓存数据量较小
### 本地进程缓存
Caffeine是一个基于Java8开发的，提供了近乎最佳命中率的高性能的本地缓存库。目前Spring内部的缓存使用的就是Caffeine。GitHub地址：[caffeine](https://github.com/ben-manes/caffeine)
```java
@Test  
void testBasicOps() {    
	// 创建缓存对象    
	Cache<String, String> cache = Caffeine.newBuilder().build();    
	// 存数据    
	cache.put("gf", "迪丽热巴");    
	// 取数据，不存在则返回null    S
	tring gf = cache.getIfPresent("gf");  
    System.out.println("gf = " + gf);  
  
    // 取数据，包含两个参数：
    // 参数一：缓存的key
    // 参数二：Lambda表达式，表达式参数就是缓存的key，方法体是查询数据库的逻辑
    // 优先根据key查询JVM缓存，如果未命中，则执行参数二的Lambda表达式  
    String defaultGF = cache.get("defaultGF", key -> {        
	    // 这里可以去数据库根据 key查询value 将结果存入缓存，并且把结果返回给用户       
	    return "柳岩";  
    });  
    System.out.println("defaultGF = " + defaultGF);  
}
```
### Caffeine提供了三种缓存驱逐策略：
1. **基于容量**：设置缓存的数量上限
清理的时候使用的是LRU策略（最近最少使用）。
```java
// 创建缓存对象  
Cache<String, String> cache = Caffeine.newBuilder()        
		.maximumSize(1) 
		// 设置缓存大小上限为 1        
		.build();
```
2. **基于时间**：设置缓存的有效时间
```java
// 创建缓存对象  
Cache<String, String> cache = Caffeine.newBuilder()  
        .expireAfterWrite(Duration.ofSeconds(10)) 
        // 设置缓存有效期为 10 秒，从最后一次写入开始计时        
        .build();
```
3. **基于引用**：设置缓存为软引用或弱引用，利用GC来回收缓存数据。性能较差，不建议使用。
在默认情况下，当一个缓存元素过期的时候，Caffeine  **不会自动立即将其清理和驱逐**  。而是在一次读或写操作后，或者在空闲时间完成对失效数据的驱逐。

## 多级缓存
### `OpenResty`
`OpenResty` 是一个基于 Nginx的高性能 Web 平台，用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。具备下列特点：
- 具备Nginx的完整功能
- 基于Lua语言进行扩展，集成了大量精良的 Lua 库、第三方模块
- 允许使用Lua自定义业务逻辑、自定义库
### 快速入门
1. 在项目环境中的nginx.conf配置：
在nginx.conf的http下面，添加对OpenResty的Lua模块的加载：
```nginx.conf
 # 加载lua 模块  
 lua_package_path "/usr/local/openresty/lualib/?.lua;;";  
 # 加载c模块 
 lua_package_cpath "/usr/local/openresty/lualib/?.so;;";
 
2.在nginx.conf的server下面，添加对/api/item这个路径的监听：
 location /api/item {
     # 响应类型，这里返回json
     default_type application/json;

     # 响应数据由 lua/item.lua这个文件来决定
     content_by_lua_file lua/item.lua;

 }
```
2. 编写item.lua文件
对着第1步的配置改。
![[Pasted image 20251003102613.png]]
### OpenResty获取请求参数
OpenResty提供了各种API用来获取不同类型的请求参数：
![[Pasted image 20251003102759.png]]
### 查询Tomcat
不管你的Linux虚拟机地址是什么，只需要配置前三位，最后一位替换成1，一定得到的就是你windows电脑地址。（防火墙要关了）

需要再看吧 。主要讲的是：
1. 如何在OpenResty里面发起一个http请求：
利用api `location.capture`发请求（指定请求路径、请求方式、请求参数）->这个请求是被nignx自己捕获了，需要编写一个反向代理把请求拦截下来，代理到目标服务器（Tomcat服务器）
2. 如何封装一个工具类
放在对应的目录下，能被OpenResty加载，就是一个通用的工具模块了。这个路径在：本章快速入门的步骤1中自行配置
使用时只要导入这个包就可以了。
### JSON结果处理
OpenResty提供了一个cjson的模块用来处理JSON的序列化和反序列化。
官方地址： [`cjson`](https://github.com/openresty/lua-cjson/)

- 引入cjson模块：
```lua
local cjson = require "cjson"
```
- 序列化：
```lua
local obj = {
    name = 'jack',
    age = 21
}
local json = cjson.encode(obj)
```
- 反序列化：
```lua
local json = '{"name": "jack", "age": 21}'
-- 反序列化
local obj = cjson.decode(json);
print(obj.name)
```
### Tomcat集群的负载均衡
![[Pasted image 20251003110144.png]]
```nginx.conf
# 反向代理配置，将/item路径的请求代理到tomcat集群        
location /item {
#	proxy_pass http://192.168.150.1:8088;
    proxy_pass http://tomcat-cluster;
}


# tomcat集群配置
upstream tomcat-cluster{
	#原来的查询命中率低，加入hash算法，由轮询转为指定节点，提高命中率
	hash $request_uri; 
    server 192.168.150.1:8081;
    server 192.168.150.1:8082;
}
```
`hash $request_uri;`这句目的：不加的话默认采用轮询的方式得到目标服务器，用于tomcat进程缓存不能共享，查询命中率低。
加入hash算法，由轮询转为指定节点，提高命中率（有点类似于Redis分片集群的散列插槽）
### Redis缓存预热
在java项目的service层中调用StringRedisTemple完成。
![[Pasted image 20251003111420.png]]
- 冷启动：服务刚刚启动时，Redis中并没有缓存，如果所有商品数据都在第一次查询时添加缓存，可能会给数据库带来较大压力。

- 缓存预热：在实际开发中，我们可以利用大数据统计用户访问的热点数据，在项目启动时将这些热点数据提前查询并保存到Redis中。
1.使用了InitializingBean接口，要实现afterPropertiesSet方法，这个方法   会在Bean创建完Autowired注入之后执行。
 这样，就可以实现在项目创建之后执行，实现缓存预热功能。
2.使用了Spring里面的默认Json处理工具ObjectMapper，做Json序列化，  使用了一个叫writeValueAsString地一个函数
### OpenResty的Redis模块
实现优先查Redis查不到再去差Tomcat。

OpenResty提供了操作Redis的模块，我们只要引入该模块就能直接使用：
- 引入Redis模块，并初始化Redis对象
```lua
-- 引入redis模块
local redis = require("resty.redis")

-- 初始化Redis对象
local red = redis:new()

-- 设置Redis超时时间,单位毫秒
red:set_timeouts(1000, 1000, 1000)
```
- 封装函数，用来释放Redis连接，其实是放入连接池
放入连接池中，减少读取后 获取释放 资源地频次，提升性能。
```lua
-- 关闭redis连接的工具方法，其实是放入连接池
local function close_redis(red)  
    local pool_max_idle_time = 10000 -- 连接的空闲时间，单位是毫秒  
    local pool_size = 100 --连接池大小  
    local ok, err = red:set_keepalive(pool_max_idle_time, pool_size)  

    if not ok then  
        ngx.log(ngx.ERR, "放入Redis连接池失败: ", err)  
    end  
end
```
- 封装函数，从Redis读数据并返回
```lua
-- 查询redis的方法 ip和port是redis地址，key是查询的key
local function read_redis(ip, port, key)  

    -- 获取一个连接
    local ok, err = red:connect(ip, port)  

    if not ok then  
        ngx.log(ngx.ERR, "连接redis失败 : ", err)  
        return nil
    end

    -- 查询redis
    local resp, err = red:get(key) 

    -- 查询失败处理
    if not resp then  
        ngx.log(ngx.ERR, "查询Redis败: ", err, ", key = " , key)
    end  

    --得到的数据为空处理  
    if resp == ngx.null then
        resp = nil
        ngx.log(ngx.ERR, "查询Redis数据为空, key = ", key)
    end  

    close_redis(red)  
    return resp
end
```
### Nginx本地缓存
![[Pasted image 20251003113617.png]]
OpenResty为Nginx提供了shard dict的功能，可以在nginx的多个worker之间共享数据，实现缓存功能。

- 开启共享字典，在nginx.conf的http下添加配置：
```nginx.conf
 # 共享字典，也就是本地缓存，名称叫做：item_cache，大小150m
 lua_shared_dict item_cache 150m;
```
- 操作共享字典：
```lua
-- 获取本地缓存对象
local item_cache = ngx.shared.item_cache

-- 存储, 指定key、value、过期时间，单位s，默认为0代表永不过期
item_cache:set('key', 'value', 1000)

-- 读取
local val = item_cache:get('key')
```
注意：
	1. 修改item.lua中的read_data函数，优先查询本地缓存，未命中时再查询Redis、Tomcat
	2. 查询Redis或Tomcat成功后，将数据写入本地缓存，并设置有效期
## 缓存同步
### 缓存同步策略：
**设置有效期**：给缓存设置有效期，到期后自动删除。再次查询时更新（例如：OpenResty的本地缓存）
	优势：简单、方便
	缺点：时效性差，缓存过期之前可能不一致
	场景：更新频率较低，时效性要求低的业务

**同步双写**：在修改数据库的同时，直接修改缓存
	优势：时效性强，缓存与数据库强一致
	缺点：有代码侵入，耦合度高；
	场景：对一致性、时效性要求较高的缓存数据

**异步通知**：修改数据库时发送事件通知，相关服务监听到通知后修改缓存数据(利用MQ实现)
	优势：低耦合，可以同时通知多个缓存服务
	缺点：时效性一般，可能存在中间不一致状态
	场景：时效性要求一般，有多个服务需要同步
#### 异步通知：
基于MQ
![[Pasted image 20251003115101.png]]
基于Canal
![[Pasted image 20251003115134.png]]
### Canal简介：
Canal是基于mysql的主从同步来实现的，MySQL主从同步的原理如下：
![[Pasted image 20251003115342.png]]
- MySQL master 将数据变更写入二进制日志( binary log），其中记录的数据叫做binary log events
- MySQL slave 将 master 的 binary log events拷贝到它的中继日志(relay log)
- MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据

Canal就是把自己伪装成MySQL的一个slave节点，从而监听master的binary log变化。再把得到的变化信息通知给Canal的客户端，进而完成对其它数据库的同步。
![[Pasted image 20251003115526.png]]
#### 监听Canal
Canal提供了各种语言的客户端，当Canal监听到binlog变化时，会通知Canal的客户端。不过这里使用GitHub上的[第三方开源的canal-starter](https://github.com/NormanGyllenhaal/canal-client),
以此简化Canal配置。
- 引入依赖
```pom.xml
  <dependency>
      <groupId>top.javatool</groupId>
      <artifactId>canal-spring-boot-starter</artifactId>
      <version>1.2.6-RELEASE</version>
  </dependency>
```
- 编写配置
```yml
canal:  
 # canal实例名称，要跟canal-server运行时设置的destination一致  
 destination: heima 
 server: 192.168.150.101:11111 # canal地址
```
### 小结
![[Pasted image 20251003121354.png]]
# Redis最佳实践
## 键值设计
### 优雅的key结构
Redis的Key虽然可以自定义，但最好遵循下面的几个最佳实践约定：
- 遵循基本格式：`[业务名称]:[数据名]:[id]`
- 长度不超过44字节
- 不包含特殊字符
例如：我们的登录业务，保存用户信息，其key是这样的：
![[Pasted image 20251003142041.png]]
1. 可读性强
2. 避免key冲突
3. 方便管理
4. 更节省内存： key是string类型，底层编码包含int、embstr和raw三种。embstr在小于44字节使用，采用连续内存空间，内存占用更小
redis-cli中用 object encoding key -> 可以查看底层实现
### 拒绝BigKey
BigKey通常以Key的大小和Key中成员的数量来综合判定，例如：
- Key本身的数据量过大：一个String类型的Key，它的值为5 MB。
- Key中的成员数过多：一个ZSET类型的Key，它的成员数量为10,000个。
- Key中成员的数据量过大：一个Hash类型的Key，它的成员数量虽然只有1,000个但这些成员的Value（值）总大小为100 MB。
推荐值：
- 单个key的value小于10KB
- 对于集合类型的key，建议元素数量小于1000


BigKey的危害:
1. 网络阻塞
对BigKey执行读请求时，少量的QPS就可能导致带宽使用率被占满，导致Redis实例，乃至所在物理机变慢
2. 数据倾斜
BigKey所在的Redis实例内存使用率远超其他实例，无法使数据分片的内存资源达到均衡
3. Redis阻塞
对元素较多的hash、list、zset等做运算会耗时较旧，使主线程被阻塞
4. CPU压力
对BigKey的数据序列化和反序列化会导致CPU的使用率飙升，影响Redis实例和本机其它应用


如何发现BigKey:
1. `redis-cli --bigkeys`
利用redis-cli提供的--bigkeys参数，可以遍历分析所有key，并返回Key的整体统计信息与每个数据的Top1的big key
2. scan扫描
自己编程，利用scan扫描Redis中的所有key，利用strlen、hlen等命令判断key的长度（此处不建议使用MEMORY USAGE）
3. 第三方工具
利用第三方工具，如 [`Redis-Rdb-Tools`](https://github.com/sripathikrishnan/redis-rdb-tools?spm=a2c4g.11186623.0.0.14073c9cldKVDv)分析RDB快照文件，全面分析内存使用情况(离线分析)
4. 网络监控
自定义工具，监控进出Redis的网络数据，超出预警值时主动告警


如何删除BigKey:
BigKey内存占用较多，即便时删除这样的key也需要耗费很长时间，导致Redis主线程阻塞，引发一系列问题。
- Redis 3.0 及以下版本
如果是集合类型，则遍历BigKey的元素，先逐个删除子元素，最后删除BigKey
- Redis 4.0以后
 Redis在4.0后提供了异步删除的命令：unlink
#### 恰当的数据类型
例1：比如存储一个User对象，我们有三种存储方式：
![[Pasted image 20251003144245.png]]
存在的问题：
- hash的entry数量超过500时，会使用哈希表而不是ZipList，内存占用较多。
- 可以通过hash-max-ziplist-entries配置entry上限。但是如果entry过多就会导致BigKey问题(entry数量不要超过1000)


如何优化？
1. 拆分为string类型
存在的问题：
- string结构底层没有太多内存优化，内存占用较多。
- 想要批量获取这些数据比较麻烦
1. 拆分为小的hash，将 id / 100 作为key， 将id % 100 作为field


1.Key的最佳实践：
- 固定格式：`[业务名]:[数据名]:[id]`
- 足够简短：不超过44字节
- 不包含特殊字符
2.Value的最佳实践：
- 合理的拆分数据，拒绝BigKey
- 选择合适数据结构
- Hash结构的entry数量不要超过1000
- 设置合理的超时时间
## 批处理优化
### MSET
不要在一次批处理中传输太多命令，否则单次命令占用带宽过多，会导致网络阻塞。
Redis提供了很多Mxxx这样的命令，可以实现批量插入数据，例如：
- `mset`
- `hmset`
利用mset批量插入10万条数据：
```java
@Test  
void testMxx() {  
    String[] arr = new String[2000];    
    int j;    
    for (int i = 1; i <= 100000; i++) {  
        j = (i % 1000) << 1;  //左位移一位
        arr[j] = "test:key_" + i;       
        arr[j + 1] = "value_" + i;        
        if (j == 0) {            
	        jedis.mset(arr);  
        }  
    }  
}
```
左位移一位 , 保证对低位为0 , 结果永远是偶数位
### Pipeline
MSET虽然可以批处理，但是却只能操作部分数据类型，因此如果有对复杂数据类型的批处理需要，建议使用Pipeline功能：
可以做所有类型的批处理。
```java
@Test  
void testPipeline() {    
	// 创建管道    
	Pipeline pipeline = jedis.pipelined();    
	for (int i = 1; i <= 100000; i++) {        
		// 放入命令到管道        
		pipeline.set("test:key_" + i, "value_" + i);        
		if (i % 1000 == 0) {            
			// 每放入1000条命令，批量执行            
			pipeline.sync();  
        }  
    }  
}
```
mset（Redis原生）原子操作,
pipeline是缓存了多条命令一起发送(并不具备原子性)
### 集群下的批处理
如MSET或Pipeline这样的批处理需要在一次请求中携带多条命令，而此时如果Redis是一个集群，那批处理命令的多个key必须落在一个插槽中，否则就会导致执行失败。
![[Pasted image 20251003151631.png]]
推荐使用并行slot，不推荐hash_tag是因为它会数据倾斜。

Redis计算key的插槽的时候是根据key的有效部分计算的 -->`mset {prefix_id}name jack`
没有{ }中内容时，根据name计算；若有了prefix_id前缀则根据前缀计算（这个prefix_id也叫hash_tag）

java中的jedisCluster没有解决集群模式下的批处理问题
`stringRedisTemplate.opsForValue().multiSet()`可以
## 服务端配置
### 持久化配置
Redis的持久化虽然可以保证数据安全，但也会带来很多额外的开销，因此持久化请遵循下列建议：
1. 用来做缓存的Redis实例尽量不要开启持久化功能
2. 建议关闭RDB持久化功能，使用AOF持久化
3. 利用脚本定期在slave节点做RDB，实现数据备份
4. 设置合理的rewrite阈值，避免频繁的bgrewrite
5. 配置`no-appendfsync-on-rewrite = yes`，禁止在rewrite期间做aof，避免因AOF引起的阻塞
![[Pasted image 20251003154043.png]]
部署有关建议：
6. Redis实例的物理机要预留足够内存，应对fork和rewrite
7. 单个Redis实例内存上限不要太大，例如4G或8G。可以加快fork的速度、减少主从同步、数据迁移压力
8. 不要与CPU密集型应用部署在一起
9. 不要与高硬盘负载应用一起部署。例如：数据库、消息队列

总结就是单独放一个服务器。
### 慢查询
慢查询：在Redis执行时耗时超过某个阈值的命令，称为慢查询。

慢查询的阈值可以通过配置指定：
- slowlog-log-slower-than：慢查询阈值，单位是微秒。默认是10000，建议1000
慢查询会被放入慢查询日志中，日志的长度有上限，可以通过配置指定：
- slowlog-max-len：慢查询日志（本质是一个队列）的长度。默认是128，建议1000
修改这两个配置可以使用：config set命令（临时的）、配置文件上配（永久的）

查看慢查询日志列表：
- `slowlog len`：查询慢查询日志长度
- `slowlog get [n]`：读取n条慢查询日志
- `slowlog reset`：清空慢查询列表
### 命令及安全配置
Redis会绑定在0.0.0.0:6379，这样将会将Redis服务暴露到公网上，而Redis如果没有做身份认证，会出现严重的安全漏洞.[漏洞重现方式](https://cloud.tencent.com/developer/article/1039000)
漏洞出现的核心的原因有以下几点：
- Redis未设置密码
- 利用了Redis的config set命令动态修改Redis配置
- 使用了Root账号权限启动Redis

为了避免这样的漏洞，这里给出一些建议：
- Redis一定要设置密码
- 禁止线上使用下面命令：keys、flushall、flushdb、config set等命令。可以利用rename-command禁用(就是一个重命名命令，把命令改的无敌长)。
- bind：限制网卡，禁止外网网卡访问
- 开启防火墙
- 不要使用Root账户启动Redis
- 尽量不是有默认的端口
### 内存配置
当Redis内存不足时，可能导致Key频繁被删除、响应时间变长、QPS不稳定等问题。当内存使用率达到90%以上时就需要我们警惕，并快速定位到内存占用的原因。
![[Pasted image 20251003161729.png]]
Redis提供了一些命令，可以查看到Redis目前的内存分配状态：
- info memory   
- memory xxx    
桌面端可以看。

内存缓冲区常见的有三种：
- 复制缓冲区：主从复制的repl_backlog_buf，如果太小可能导致频繁的全量复制，影响性能。通过repl-backlog-size来设置，默认1mb
- AOF缓冲区：AOF刷盘之前的缓存区域，AOF执行rewrite的缓冲区。无法设置容量上限
- 客户端缓冲区(主要看这个)：分为  输入缓冲区  和  输出缓冲区（相对于redis的），输入缓冲区最大1G且不能设置。输出缓冲区可以设置
![[Pasted image 20251003162344.png]]
client list可以看到连接当前redis的所有客户端。
## 集群最佳实践
集群虽然具备高可用特性，能实现自动故障恢复，但是如果使用不当，也会存在一些问题：
1. 集群完整性问题 
2. 集群带宽问题
3. 数据倾斜问题   （`bigkey`、批处理）
4. 客户端性能问题   (节点选择、读写分离的判断、插槽的判断)
5. 命令的集群兼容性问题  (PipeLine解决的数据类型问题)
6. lua和事务问题   （集群模式下无法运行lua和事务 <--> 无法保证原子性）
在 Redis 集群中，事务不能跨多个节点执行。如果尝试在一个事务中包含多个分片的键，事务将失败。同样，执行 Lua 脚本时，脚本中访问的所有键也必须位于同一节点。

注意：（尽量不要做集群）
单体Redis（主从Redis）已经能达到万级别的QPS，并且也具备很强的高可用特性。如果主从能满足业务需求的情况下，尽量不搭建Redis集群。
### 集群完整性问题
在Redis的默认配置中，如果发现任意一个插槽不可用，则整个集群都会停止对外服务：
redis.conf文件
![[Pasted image 20251003163045.png]]
为了保证高可用特性，这里建议将 cluster-require-full-coverage配置为false
### 集群带宽问题
集群节点之间会不断的互相Ping来确定集群中其它节点的状态。每次Ping携带的信息至少包括：
- 插槽信息
- 集群状态信息

集群中节点越多，集群状态信息数据量也越大，10个节点的相关信息可能达到1kb，此时每次集群互通需要的带宽会非常高。

解决途径：
1. 避免大集群，集群节点数不要太多，最好少于1000，如果业务庞大，则建立多个集群。
2. 避免在单个物理机中运行太多Redis实例  (10个左右就可以了)
3. 配置合适的cluster-node-timeout值   (集群之间做客观下线的超时时间)
# 后记
后接[[Redis原理篇]]