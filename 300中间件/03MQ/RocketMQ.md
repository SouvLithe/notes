# Base
MQ全称 Message Queue（消息队列），是在消息的传输过程中保存消息的容器。多用于分布式系统之间进行通信。队列：数据结构的一种，特征为“先进先出”
![[Pasted image 20251007200007.png]]
系统的耦合性越高，容错率就越低，可维护性就越低。
## 优劣势对比：
### 优势

1. 应用解耦：MQ可以把得到的消息复制成多份，分发给目标业务群。
2. 异步提速：MQ可以优先把数据消息变更通知到数据库的一张专门表中，就立刻返回结果响应前端，后端其它业务慢慢去调用就行。
3. 削峰填谷：把MQ当作一个缓存，响应业务系统再慢慢去MQ中拿。
![[Pasted image 20251007201438.png]]
### 劣势：
1. 系统可用性降低 （只要引第三方技术，都会出现这个问题）
系统引入的外部依赖越多，系统稳定性越差。一旦 MQ 宕机，就会对业务造成影响。
如何保证MQ的高可用？

2. 系统复杂度提高 
MQ 的加入大大增加了系统的复杂度，以前系统间是同步的远程调用，现在是通过 MQ 进行异步调用。
如何保证消息没有被重复消费？怎么处理消息丢失情况？那么保证消息传递的顺序性？

3. 一致性问题 
A 系统处理完业务，通过 MQ 给B、C、D三个系统发消息数据，如果 B 系统、C 系统处理成功，D 系统处理失败。
如何保证消息数据处理的一致性？
## 用什么MQ
1.ActiveMQ
- java语言实现，万级数据吞吐量，处理速度ms级，主从架构，成熟度高

2.RabbitMQ
- erlang语言实现，万级数据吞吐量，处理速度us级，主从架构

3.RocketMQ
- java语言实现，十万级数据吞吐量，处理速度ms级，分布式架构，功能强大，扩展性强

4.kafka
- scala语言实现，十万级数据吞吐量，处理速度ms级，分布式架构，功能较少，应用于大数据较多
## 工作原理
部署着MQ的机器就叫Broker。
![[Pasted image 20251007204530.png]]
消费者是定时拉取，但性能很差
消息（对象）定义一个Topic用于区分消息类型，如：用户消息、商家消息等
	定义一个Tag同一个Topic下，继续细分。如：vip和普通用户。
假如有多个Broker Cluster用存在相同的Topic类型，那么由NameServer提供具体存放队列路径。
拉去消息比较耗费资源，一般都是用监听器模式。

# 使用MQ
```bash
cd /tools/rocketmq
./rocketmq_jdk17_start.sh
```
这个脚本会自动：停止现有服务、使用JDK17兼容性参数启动NameServer和Broker、 显示启动状态
### 停止服务
```bash
# 方法1：使用停止脚本
cd /tools/rocketmq
sh bin/mqshutdown broker
sh bin/mqshutdown namesrv

# 方法2：使用pkill
pkill -f broker
pkill -f namesrv
```
### 检查状态
```bash
# 检查进程
ps aux | grep -E "namesrv|broker"
jps -l

# 检查端口
netstat -tlnp | grep -E "9876|10911"
```
### 常用管理操作
```bash
cd /tools/rocketmq

# 查看集群状态
sh bin/mqadmin clusterList -n localhost:9876

# 创建Topic
sh bin/mqadmin updateTopic -n localhost:9876 -t YourTopic -c DefaultCluster

# 查看Topic列表
sh bin/mqadmin topicList -n localhost:9876

# 查看消费者进度
sh bin/mqadmin consumerProgress -n localhost:9876

# 发送测试消息
sh bin/mqadmin sendMessage -n localhost:9876 -t YourTopic -p "test message"
```
### 日志查看
```bash
# Broker启动日志
tail -f /tools/rocketmq/logs/broker.out

# NameServer启动日志
tail -f /tools/rocketmq/logs/namesrv.out

# Broker运行日志
tail -f ~/logs/rocketmqlogs/broker.log

# NameServer运行日志
tail -f ~/logs/rocketmqlogs/namesrv.log
```
### 测试消息发送和接收
```bash
cd /tools/rocketmq

# 终端1：启动消费者
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer

# 终端2：启动生产者
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```
# 单生产多消费者
![[Pasted image 20251012104413.png]]
默认情况下：
	每个group都会得到生产者发送的完整消息。
	而每个group中的不同consumer会因为分配策略的不同，可能得到  部分  或者  全部  的消息。

用下面的代码设置，最前面的是Cosumer对象创建的示例。
```java
consumer.setMessageModel(MessageModel.BROADCASTING);
```
# 消息类别
### 同步消息：
特征：时效性强，重要的消息且必须有回执的消息会用。
![[Pasted image 20251012105808.png]]
### 异步消息:
特征：即时性弱，但需要有回执的消息。常用于提高系统服务的吞吐量。
![[Pasted image 20251012105845.png]]
异步调用的时候不要关掉生产者，不然拿不到mq返回的数据会一直认为自己回调失败
### 单向消息：
特征：不需要有回执的消息，例如：日志类的消息
![[Pasted image 20251012111245.png]]
### 延时消息：
特征：消息时并不直接发送到消息服务器，而是根据设定的等待时间到达，起到延时到达的缓冲作用。
（如游戏好久登录的邀请回归消息）
![[Pasted image 20251012112338.png]]
可以根据每一条消息设置其延时等级。
### 批量消息：
特征：一次发送多条消息，节省网络开销。
![[Pasted image 20251012112641.png]]
注意点：
1. 这些批量消息应该(必须)有相同的topic
2. 相同的waitStoreMsgOK,尽量消息类型一样
3. 不能是延时消息
4. 消息内容总长度不超过4M

消息内容总长度包含如下：
1. topic（字符串字节数）
2. body（字节数组长度）
3. 消息追加的属性（key与value对应字符串字节数）
4. 日志（固定20字节）
# 消息过滤
### 按照tag过滤消息
消费者tag要和所监听的生产者的tag一致，
可以监听多个用`||`分开，`*` 表示监听该topic的所有tag。
![[Pasted image 20251012113745.png]]

### 按照sql过滤消息
语法过滤（属性过滤/语法过滤/SQL过滤）：按照消息的某些属性过滤
基本语法：
- 数值比较，比如：>, >=, <, <=, BETWEEN, =;
- 字符串比较，比如：=, <>, IN;
- IS NULL 或者 IS NOT NULL;
- 逻辑符号 AND, OR, NOT

常量支持类型为：
- 数值，比如：123, 3.1415;
- 字符串，比如：`'abc'`，必须用单引号包裹起来;
- NULL，特殊的常量
- 布尔值，TRUE 或 FALSE
![[Pasted image 20251012115626.png]]
注意：
	运维部署rocket的时候需要把sql过滤开启起来。
	在broker配置中追加  `enablePropertyFilter=true`
# 消息的特殊处理
## 消息顺序
队列内有序，而队列外无序。
讲的不好回头去网上找吧，rocketmq没有做这个，是依靠算法解决的。
## 消息事务
![[Pasted image 20251012182337.png]]
 蓝色：事务补偿过程。
 红色：正常事务过程。
- 提交状态：允许进入队列，此消息与非事务消息无区别
- 回滚状态：不允许进入队列，此消息等同于未发送过
- 中间状态：完成了half消息的发送，未对MQ进行二次状态确认
 注意：
	 事务消息仅与生产者有关，与消费者无关
# 集群搭建
- 单机
    - 一个broker提供服务（宕机后服务瘫痪）
- 集群
    - 多个broker提供服务（单机宕机后消息无法及时被消费）
    - 多个master多个slave
        - master到slave消息同步方式为同步（较异步方式性能略低，消息无延迟）
        - master到slave消息同步方式为异步（较同步方式性能略高，数据略有延迟）
![[Pasted image 20251012184825.png]]
在配置文件中:
	只有主节点的brokerId =0。
	通过将多个broker Name 写成一样的，来分辨是否是同一个broker cluster。
![[Pasted image 20251012185356.png]]
主节点和副节点在物理意义上要离的远一点，防止一个挂了影响另一个。
![[Pasted image 20251012191004.png]]
# 高级特性
## 消息的存储
1. 消息生成者发送消息到MQ
2. MQ收到消息，将消息进行持久化，存储该消息
3. MQ返回ACK给生产者
4. MQ push 消息给对应的消费者
5. 消息消费者返回ACK给MQ
6. MQ删除消息
![[Pasted image 20251012193942.png]]
新型的RocketMQ、Kafka、RabbitMQ都是直接掠过数据库，直接与文件系统交互。 
注意：
-  第5步MQ在指定时间内接到消息消费者返回ACK，MQ认定消息消费成功，执行6
-  第5步MQ在指定时间内未接到消息消费者返回ACK，MQ认定消息消费失败，重新执行456步

直接往文件系统中存可以有效避免数据库的额外系统开销。
数据库
- ActiveMQ
- 缺点：数据库瓶颈将成为MQ瓶颈
文件系统
- `RocketMQ/Kafka/RabbitMQ`
- 解决方案：采用消息刷盘机制进行数据存储
## 消息存储与读写方式
- SSD(Solid State Disk)
	随机写    100kb/s
	顺序写    600-3000m/s
rocketmq高效读写的原理：预先申请了连续的磁盘空间，进行了顺序读写。
![[Pasted image 20251012195419.png]]
- “零拷贝”技术
零拷贝是Linux系统发送数据的方式：
- 数据传输由传统的4次复制简化成3次复制，减少1次复制过程
- Java语言中使用MappedByteBuffer类实现了该技术
- 要求：预留存储空间，用于保存数据（1G存储空间起步）
## 消息存储结构
![[Pasted image 20251012200006.png]]
消息存在commitlog中，MessageQueue中存着topic、queueId、message。
消费者逻辑队列：记录消费者的读取记录，记录会保存一段时间。
索引：存着queue的创建时间，消费者的读取记录信息也在这里。
## 刷盘机制
同步刷盘：
1. 生产者发送消息到MQ，MQ接到消息数据
2. MQ挂起生产者发送消息的线程
3. MQ将消息数据写入内存
4. 内存数据写入硬盘
5. 磁盘存储后返回SUCCESS
6. MQ恢复挂起的生产者线程
7. 发送ACK到生产者
![[Pasted image 20251012200821.png]]
异步刷盘：
只有1、3、7步，异步消息都是在内存中存折，只有当积累了又一批消息的时候，才写入磁盘。

- 同步刷盘：安全性高，效率低，速度慢（适用于对数据安全要求较高的业务）
- 异步刷盘：安全性低，效率高，速度快（适用于对数据处理速度要求较高的业务）
配置方式：
在brocker主节点的配置文件中进行配置：
```bash
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=SYNC_FLUSH
```
## 高可用
### 四方面
- nameserver
    - 无状态+全服务器注册
- 消息服务器
    - 主从架构（2M-2S）
- 消息生产
    - 生产者将相同的topic绑定到多个group组，保障master挂掉后，其他master仍可正常进行消息接收
- 消息消费
    - RocketMQ自身会根据master的压力确认是否由master承担消息读取的功能，当master繁忙时候，自动切换由slave承担数据读取的工作  (读写分离)
### 主从数据复制
- 同步复制
    - master接到消息后，先复制到slave，然后反馈给生产者写操作成功
    - 优点：数据安全，不丢数据，出现故障容易恢复
    - 缺点：影响数据吞吐量，整体性能低
- 异步复制
    - master接到消息后，立即返回给生产者写操作成功，当消息达到一定量后再异步复制到slave
    - 优点：数据吞吐量大，操作延迟低，性能高
    - 缺点：数据不安全，会出现数据丢失的现象，一旦master出现故障，从上次数据同步到故障时间的数据将丢失
- 配置方式（主节点配置文件）
```bash
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=SYNC_MASTER
```
### 负载均衡
默认每个topic都有四个队列
Producer负载均衡：
	只要broker中有Producer需要的消息队列，Producer会采取轮询的方式发送。
Consumer负载均衡：
- 平均分配
![[Pasted image 20251012202551.png]]
问题：当broker-A挂了之后，蓝色的就拿不到消息了，黄色的还可以，压力全给到了绿色
- 循环平均分配（解决宕机问题）
![[Pasted image 20251012202721.png]]
## 消息重试
当消息消费后未正常返回消费成功的信息将启动消息重试机制
1. 顺序消息
当消费者消费消息失败后，RocketMQ会自动进行消息重试（每次间隔时间为 1 秒）
注意：
	应用会出现消息消费被阻塞的情况，因此，要对顺序消息的消费情况进行监控，避免阻塞现象的发生

2. 无序消息
- 无序消息包括普通消息、定时消息、延时消息、事务消息
- 无序消息重试仅适用于负载均衡（集群）模型下的消息消费，不适用于广播模式下的消息消费
- 为保障无序消息的消费，MQ合理的消息重试间隔时长 (默认重试16次，总时长4h45min40s)
### 死信队列
当消息消费重试到达了指定次数（默认16次）后，MQ将无法被正常消费的消息称为死信消息（Dead-Letter Message）。
死信消息不会被直接抛弃，而是保存到了一个全新的队列中，该队列称为死信队列（Dead-Letter Queue）

死信队列特征
- 归属某一个组（Gourp Id），而不归属Topic，也不归属消费者
- 一个死信队列中可以包含同一个组下的多个Topic中的死信消息
- 死信队列不会进行默认初始化，当第一个死信出现后，此队列首次初始化

死信队列中消息特征
- 不会被再次重复消费
- 死信队列中的消息有效期为3天，达到时限后将被清除

处理死信队列，不同的企业处理方式不同。
一个例子：
	死信处理
	在监控平台中，通过查找死信，获取死信的messageId，然后通过id对死信进行精准消费
## 消息的重复消费
### 消息重复消费原因：
1、生产者发送了重复的消息
- 网络闪断
- 生产者宕机    
![[Pasted image 20251012204016.png]]
2、消息服务器投递了重复的消息
- 网络闪断

3、动态的负载均衡过程
- 络闪断/抖动、broker重启、订阅方应用重启（消费者）、客户端扩容、客户端缩容
### 消息幂等性
**对同一条消息，无论消费多少次，结果保持一致，称为消息幂等性**

解决方案:
- 使用业务id作为消息的key
- 在消费消息时，客户端对key做判定，未使用过放行，使用过抛弃
注意：messageId由RocketMQ产生，messageId并不具有唯一性，不能作用幂等判定条件

| 操作类型 | 幂等性 | SQL语句 |
|----------|--------|---------|
| 新增     | 不幂等 | `insert into order values (……)` |
| 查询     | 幂等   |          |
| 删除     | 幂等   | `delete from 表 where id = 1` |
| 修改     | 不幂等 | `update account set balance = balance+100 where no=1` |
| 修改     | 幂等   | `update account set balance = 100 where no=1` |