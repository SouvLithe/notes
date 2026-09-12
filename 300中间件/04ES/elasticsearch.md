# ELK技术栈
elasticsearch结合kibana、Logstash、Beats，也就是`elastic stack（ELK）`。被广泛应用在日志数据分析、实时监控等领域。
而elasticsearch是elastic stack的核心，负责存储、搜索、分析数据。
![[Pasted image 20260508151300.png]]
# 倒排索引
## 正向索引
例如给下表（tb_goods）中的id创建索引：
![[Pasted image 20260508152050.png]]
如果是根据id查询，那么直接走索引，查询速度非常快。但如果是基于title做模糊查询，只能是逐行扫描数据，流程如下：
>1）用户搜索数据，条件是title符合`"%手机%"`
>2）逐行获取数据，比如id为1的数据
>3）判断数据中的title是否符合用户搜索条件
>4）如果符合则放入结果集，不符合则丢弃。回到步骤1

逐行扫描，也就是全表扫描，随着数据量增加，其查询效率也会越来越低。当数据量达到数百万时，就是一场灾难。
## 倒排索引
两个重要概念：
- 文档（`Document`）：用来搜索的数据，其中的每一条数据就是一个文档。例如一个网页、一个商品信息
- 词条（`Term`）：对文档数据或用户搜索数据，利用某种算法分词，得到的具备含义的词语就是词条。例如：我是中国人，就可以分为：我、是、中国人、中国、国人这样的几个词条

**创建倒排索引**是对正向索引的一种特殊处理，流程如下：
- 将每一个文档的数据利用算法分词，得到一个个词条
- 创建表，每行数据包括词条、词条所在文档id、位置等信息
- 因为词条唯一性，可以给词条创建索引，例如hash表结构索引
![[Pasted image 20260508152411.png]]
搜索流程：
![[Pasted image 20260508152632.png]]
## 正排和倒排
那么为什么一个叫做正向索引，一个叫做倒排索引呢？
- **正向索引**，**根据文档找词条的过程**。
- **倒排索引**，**根据词条找文档的过程**。
**正向索引**：
- 优点：
    - 可以给多个字段创建索引
    - 根据索引字段搜索、排序速度非常快
- 缺点：
    - 根据非索引字段，或者索引字段中的部分词条查找时，只能全表扫描。
**倒排索引**：
- 优点：
    - 根据词条搜索、模糊搜索时，速度非常快
- 缺点：
    - 只能给词条创建索引，而不是字段
    - 无法根据字段做排序
# es的一些概念
## 文档和字段
elasticsearch是面向**文档（Document）** 存储的，可以是数据库中的一条商品数据，一个订单信息。文档数据会被序列化为json格式后存储在elasticsearch中：
![[Pasted image 20260508153321.png]]
而Json文档中往往包含很多的 **字段（Field）**，类似于数据库中的列。
## 索引和映射
**索引（Index）**，就是相同类型的文档的集合。
![[Pasted image 20260508153449.png]]
可以把索引当做是数据库中的表。
数据库的表会有约束信息，用来定义表的结构、字段的名称、类型等信息。因此，索引库中就有**映射（mapping）**，是索引中文档的字段约束信息，类似表的结构约束。
## mysql与elasticsearch
![[Pasted image 20260508153713.png]]
并非学习了elasticsearch就不再需要mysql了呢？
- Mysql：擅长事务类型操作，可以确保数据的安全和一致性
- Elasticsearch：擅长海量数据的搜索、分析、计算
![[Pasted image 20260508154242.png]]
# 使用ES
创建网络：
```shell
docker network create es-net
```
## 运行ES
```shell
docker run -d `
  --name es `
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" `
  -e "discovery.type=single-node" `
  -e "xpack.security.enabled=false" `
  -v es-data:/usr/share/elasticsearch/data `
  -v es-plugins:/usr/share/elasticsearch/plugins `
  --privileged `
  --network es-net `
  -p 9300:9200 `
  -p 9301:9300 `
  docker.elastic.co/elasticsearch/elasticsearch:8.17.10
```
- `-e "cluster.name=es-docker-cluster"`：设置集群名称
- `-e "http.host=0.0.0.0"`：监听的地址，可以外网访问
- `-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"`：内存大小
- `-e "discovery.type=single-node"`：非集群模式
- `-v es-data:/usr/share/elasticsearch/data`：挂载逻辑卷，绑定es的数据目录
- `-v es-logs:/usr/share/elasticsearch/logs`：挂载逻辑卷，绑定es的日志目录
- `-v es-plugins:/usr/share/elasticsearch/plugins`：挂载逻辑卷，绑定es的插件目录
- `--privileged`：授予逻辑卷访问权
- `--network es-net` ：加入一个名为es-net的网络中
- `-p 9300:9200：端口映射配置
## 运行kibana
```shell
docker run -d `
  --name kibana `
  -e ELASTICSEARCH_HOSTS=http://es:9200 `
  --network=es-net `
  -p 5601:5601 `
  docker.elastic.co/kibana/kibana:8.17.10
```
命令解释：
- `--network es-net` ：加入一个名为es-net的网络中，与elasticsearch在同一个网络中
- `-e ELASTICSEARCH_HOSTS=http://es:9200"`：设置elasticsearch的地址，因为kibana已经与elasticsearch在一个网络，因此可以用容器名直接访问elasticsearch
- `-p 5601:5601`：端口映射配置
kibana启动一般比较慢，需要多等待一会，可以通过命令：
```shell
 docker logs -f kibana
```
查看运行日志，以判断是否运行成功。
## 安装ik分词器(插件)
```shell
# 第1步：进入 ES 容器内部
docker exec -it es /bin/bash
# 第2步：下载并安装 IK 插件
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.17.10/elasticsearch-analysis-ik-8.17.10.zip
```
### 扩展词词典
IK分词器提供了扩展词汇的功能。
1）打开IK分词器config目录：
![[Pasted image 20260508192239.png]]
2）在IKAnalyzer.cfg.xml配置文件内容添加：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 *** 添加扩展词典-->
        <entry key="ext_dict">ext.dic</entry>
</properties>
```
3）新建一个 ext.dic，可以参考config目录下复制一个配置文件进行修改
```properties
传智播客
奥力给
```
4）重启elasticsearch
```shell
docker restart es

# 查看 日志
docker logs -f elasticsearch
# 可以在日志中看到新创建的词典
```
### 停用词词典
在互联网项目中，在网络间传输的速度很快，所以很多语言是不允许在网络上传递的，如：关于宗教、政治等敏感词语，那么我们在搜索时也应该忽略当前词汇。
IK分词器也提供了强大的停用词功能，让我们在索引时就直接忽略当前的停用词汇表中的内容。
1）IKAnalyzer.cfg.xml配置文件内容添加：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典-->
        <entry key="ext_dict">ext.dic</entry>
         <!--用户可以在这里配置自己的扩展停止词字典  *** 添加停用词词典-->
        <entry key="ext_stopwords">stopword.dic</entry>
</properties>
```
3）在 `stopword.dic` 添加停用词
```properties
习大大
```
4）重启elasticsearch
```shell
# 重启服务
docker restart elasticsearch
docker restart kibana

# 查看 日志
docker logs -f elasticsearch
```
日志中已经成功加载stopword.dic配置文件
# 关闭ES
```shell
# 查看运行中的容器（确认 ES 还在跑）
docker ps
# 停止 ES 容器
docker stop es
# 可选：停止 Kibana
docker stop kibana

# 一步到位，停止所有容器
# 停止所有运行中的容器
docker stop $(docker ps -q)
```
再启动:
```shell
# 启动 ES
docker start es
# 启动 Kibana
docker start kibana
```
# 部署es集群
部署es集群可以直接使用docker-compose来完成，不过要求你的Linux虚拟机至少有**4G**的内存空间

首先编写一个docker-compose.yaml文件，内容如下：
```yaml
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.12.1
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.12.1
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
    networks:
      - elastic
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.12.1
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
    networks:
      - elastic

volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local

networks:
  elastic:
    driver: bridge
```
然后运行：
```shell
docker-compose up
```
# Java API Client
这种是7.15以后才引入的。
```xml
<!-- 引入 -->  
<dependency>  
    <groupId>co.elastic.clients</groupId>  
    <artifactId>elasticsearch-java</artifactId>  
    <version>8.17.10</version> <!-- 替换成你的 ES 版本，要版本一直 -->  
</dependency>
```
且**完全采用了 Builder 模式**，如下：
```json
POST /_analyze
{
  "analyzer": "ik_smart", 
  "text": "我爱北京天安门"
}
```
对应java为：
```java
client.indices().analyze(b -> b
    .analyzer("ik_smart")     // 对应 JSON 中的 analyzer
    .text("我爱北京天安门")     // 对应 JSON 中的 text
);
```
