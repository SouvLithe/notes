# 基础
`mysql [-h 127.0.0.1] [-P 3306] -u root -p`
-h指定主机地址IP或者域名，默认连接localhost**127.0.0.1**。  
-P是端口号，默认是3306
方括号 `[]` 表示参数是可选的，可以省略。  root是用户名。
## SQL
注释：
```
单行注释：--注释内容 或 #注释内容(MySQL特有)
多行注释：/*注释内容*/
```
分类：
![[Pasted image 20250817113852.png]]
### DDL（数据定义语言）
#### 数据库操作：
![[Pasted image 20250817153702.png]]
#### 表操作：
创建：
![[Pasted image 20250817154037.png]]

查询：
![[Pasted image 20250817154631.png]]

修改：
![[Pasted image 20250817160711.png]]
![[Pasted image 20250817160830.png]]
![[Pasted image 20250817160949.png]]

删除：
![[Pasted image 20250817160915.png]]
![[Pasted image 20250817161400.png]]
删除表时不论哪种会删掉全部数据。
#### 数据（字段）类型：
![[Pasted image 20250817155308.png]]
![[Pasted image 20250817155347.png]]
![[Pasted image 20250817160204.png]]

**定长 CHAR 用空间换时间，适合短、固定长度字段；
变长 VARCHAR 用时间换空间，适合长度波动大的文本。**
![[Pasted image 20260309223121.png]]
#### 小结
![[Pasted image 20250817161924.png]]

### DML（数据操作语言）
#### 添加：
![[Pasted image 20250817162907.png]]
注意：
- 插入数据时，指定的字段顺序需要与值的顺序是一一对应的。
- 字符串和日期型数据应该包含在引号中。
- 插入的数据大小，应该在字段的规定范围内。
#### 修改：
![[Pasted image 20250817164254.png]]
#### 删除：
![[Pasted image 20250817164808.png]]
DELETE语句不能删除某一个字段的值（可以使用UPDATE将这个字段的值设置为NULL）

### DQL（数据查询语言）
编写顺序：
![[Pasted image 20250817165837.png]]
分组查询通常会和聚合查询一起使用。
执行顺序：
	可以利用别名来验证执行顺序。
	from > where >group by > having > select > order by > limit 
#### 基本查询
![[Pasted image 20250817170149.png]]
**在所有主流 SQL 方言里，只要“别名”紧跟在列或表之后、且没有任何可能引起歧义的关键字/标点，都可以把关键字 `AS` 省掉。**
```sql
height     AS 'h',     --  字符串单引号别名，必须写 AS（否则会被当成字符串常量）
    weight  AS   "w"         --  双引号别名，必须写 AS（同上）
```
#### 条件查询
![[Pasted image 20250817170659.png]]
![[Pasted image 20250817170723.png]]
between...and...前小后大不能写反，否则报错。
![[Pasted image 20250817170738.png]]
多个条件就在where后面用多个and连接就行。
#### 聚合函数
将一列数据作为一个整体，进行纵向计算。
![[Pasted image 20250817171614.png]]
![[Pasted image 20250817171810.png]]
#### 分组查询
![[Pasted image 20250817174146.png]]
![[Pasted image 20250817174500.png]]
#### 排序查询
![[Pasted image 20250817174058.png]]
#### 分页查询
![[Pasted image 20250817175101.png]]
![[Pasted image 20250817175111.png]]
#### 小结
![[Pasted image 20250817181401.png]]

### DCL（数据控制语言）
#### 管理用户：
![[Pasted image 20250817181848.png]]
主机名设为： `%` 时，意味所有用户都可访问。
用户名、主机名、密码的单引号不能省去。
#### 权限控制：
![[Pasted image 20250817182350.png]]
注意：
- 多个权限之间，使用逗号分隔。
- 授权时，数据库名和表名可以使用 * 进行通配，代表所有。

## 函数
### 字符串函数
常用的。
![[Pasted image 20250818155229.png]]
### 数值函数
![[Pasted image 20250818160031.png]]
### 日期函数
![[Pasted image 20250818160520.png]]
### 流程函数
![[Pasted image 20250818162602.png]]

## 约束
概念：约束时作用在表中字段上的规则，用于限制存储在表中的数据。
![[Pasted image 20250818163500.png]]
约束是作用在表中字段上的，可以在创建/修改表的时候添加约束。
多个元素之间用空格分开就行。
auto_increment   自增(mysql中)

### 外键约束
作用：用于在两张表之间建立连接。
![[Pasted image 20250818165240.png]]
![[Pasted image 20250818165536.png]]

![[Pasted image 20250818170027.png]]前两个是系统默认的作用相同。主要看后三个。

## 多表查询
关系：
多对多的实现：借用第三张中间表(至少包含两个外键，分别关联两方主键)。
一对多的实现：在多的一方建立外键，指向一的一方的主键。
一对一的关系，多用于单表拆分，以提高效率。
	实现：在任意一方加入外键，关联另外一方的主键，并设置外键唯一
![[Pasted image 20250818171850.png]]
查询分类：
![[Pasted image 20250818172135.png]]
### 自连接
![[Pasted image 20250818182937.png]]
**显式内连接把“连接关系”和“过滤条件”分离，语法清晰、易维护；隐式内连接写起来短，但可读性差、扩展难。**
### 外连接
外连接和内连接的区别是：外连接会分主副表，主表的信息会全部输出，左外即左边是主表
![[Pasted image 20250818183531.png]]
左外用的多一点，本质山两个没什么区别。
### 自连接
![[Pasted image 20250818183845.png]]
自连接的别名很重要。
**把“行与行之间的关系”转换成“表与表之间的关系”**，从而用一次查询就能找出这些关系。

### 联合查询
![[Pasted image 20250818184552.png]]
加all的话是全部表合在一起不做更改，去掉后的作用等同与去重。
### 子查询
![[Pasted image 20250818185255.png]]
根据子查询结果不同，分为：
- 标量子查询（子查询结果为单个值）
- 列子查询（子查询结果为一列）
- 行子查询（子查询结果为一行）
- 表子查询（子查询结果为多行多列）
根据子查询位置，分为：WHERE 之后、FROM 之后、SELECT 之后。
![[Pasted image 20250819162131.png]]

![[Pasted image 20250819162100.png]]
### 小结
![[Pasted image 20250820180101.png]]

## 事物
![[Pasted image 20250820180140.png]]
``` sql
SELECT @@autocommit;
	 返回 `1`：每条 SQL 执行完 自动 提交事务。
	 返回 `0`：需要 手动 `COMMIT` 或 `ROLLBACK`。

SET @@autocommit = 0;
	 此后，所有 DML（`INSERT/UPDATE/DELETE`）都只是 **暂存在事务里**，对其他人不可见，直到你显式 `COMMIT` 或 `ROLLBACK`。
COMMIT;
	 把当前事务里的所有修改 **一次性永久生效**。
ROLLBACK;
	 撤销当前事务里的所有修改，回到事务开始前的状态。
```
![[Pasted image 20250820181816.png]]
### 并发事务三个问题：
![[Pasted image 20250820182049.png]]
### 事务的隔离级别
| 隔离级别                     |  脏读 | 不可重复读 |  幻读 |
| ------------------------ | :-: | :---: | :-: |
| Read uncommitted         |  √  |   √   |  √  |
| Read committed           |  ×  |   √   |  √  |
| Repeatable Read（mysql默认） |  ×  |   ×   |  √  |
| Serializable             |  ×  |   ×   |  ×  |
从上到下隔离级别越来越高，数据越来越安全，但性能越来越低。
![[Pasted image 20250820183847.png]]

# 进阶
## MySQL体系结构
![[Pasted image 20250820191608.png]]
- **连接层**  
    最上层是一些客户端和链接服务，主要完成连接处理、授权认证及相关安全方案。服务器也会为安全接入的每个客户端验证其所具有的操作权限。
- **服务层**  
    第二层架构主要完成大多数核心服务功能，如 SQL 接口，并完成缓存查询、SQL 分析与优化以及部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。
- **引擎层**  
    存储引擎真正负责 MySQL 中数据的存储与提取，服务器通过 API 与存储引擎进行通信。不同存储引擎具有不同功能，可根据需要选择合适的存储引擎。
- **存储层**  
    主要将数据存储在文件系统之上，并完成与存储引擎的交互。
## 存储引擎
存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式。存储引擎是  基于表的，而不是基于库的  ，所以存储引擎也可被称为表类型。
MySQL5.5后，存储引擎默认版本为InnoDB。
![[Pasted image 20250820192322.png]]
常见的三个引擎：
![[Pasted image 20250820194948.png]]
### InnoDB
`xxx.ibd`：xxx 代表表名，InnoDB 引擎的每张表都会对应这样一个表空间文件，用于存储该表的
表结构（`.frm` 或 `.sdi`），数据（行记录），索引
控制参数：`innodb_file_per_table = 1` 时，每张 InnoDB 表独占一个 `xxx.ibd` 文件；值为 0 时，所有表共用系统表空间 `ibdata1`。

![[Pasted image 20250820194214.png]]
### MyISAM
MyISAM 是 MySQL 早期的默认存储引擎。
特点
- 不支持事务，不支持外键
- 支持表锁，不支持行锁
- 访问速度快
```
文件类型
tb_book.MYD  存储数据
tb_book.MYI  存储索引
tb_book_448.sdi 存放表结构信息
```
### Memory
介绍  
Memory 引擎的表数据存储在内存中，受硬件故障或断电影响，只能作为临时表或缓存使用。
特点
- 数据存放在内存
- 默认使用 hash 索引

文件  
xxx.sdi：仅存储表结构信息
### 存储引擎的选择
- **InnoDB**（MySQL 默认）  
    支持事务、外键，适合高并发、强一致性场景，频繁更新、删除、插入与查询混合操作。
- **MyISAM**  
    以读和插入为主，更新/删除极少，对事务完整性、并发要求不高时选用。
- **MEMORY**  
    数据全放内存，访问极快，仅作临时表或缓存；受内存容量限制且断电即失，安全性低。
MyISAM现被MongoDB取代，MEMORY现在被Redis取代。
所以常用的环视InnoDB引擎。

## 索引
索引（index）是帮助 MySQL 高效获取数据的（有序）数据结构。
![[Pasted image 20250825142744.png]]
优劣：
![[Pasted image 20250821150216.png]]
### 索引结构
MySQL的索引是在存储引擎层实现的，不同的存储引擎有不同的结构
![[Pasted image 20250825143306.png]]
索引结构的支持情况：
![[Pasted image 20250825143518.png]]
平常所说的索引，如果没有特别说明，都指的是B+树结构组织的索引。
### B-Tree
![[Pasted image 20250825144050.png]]
之前树结构的问题：层级较深，检查速度较慢。
![[Pasted image 20250825144422.png]]
值在两数之间，找相应指针。
五阶节点：4个key5个指针

创建过程：
在五阶节点的情况下，不断加数据，当数据个数大于4时中间数向上分裂。
中间数向上分裂是要保证左节点比中间值小，右节点比中间的大
如果阶数是偶数，加了一位数后，刚好取中心元素向上裂变（12345，取3向上裂变） 如果阶数是奇数，加了一位数后，那么取中心偏左一位元素向上裂变（1234，取2向上裂变）
### B+Tree
![[Pasted image 20250825145604.png]]
特点：
	所有的元素都会出现在叶子节点。
	叶子节点形成了一个单向链表。
所有非叶子节点起到索引作用，叶子节点存储数据。
所有叶节点包含所有关键字，叶节点将关键字按大小排列，并且相邻叶节点按大小顺序链接起来

Mysql对B+树进行了优化，在原B+Tree的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的B+Tree，提高区间访问的性能。变为了双向链表
![[Pasted image 20250825150900.png]]
### 哈希结构
![[Pasted image 20250825151259.png]]
**Hash索引特点**
1. Hash索引只能用于对等比较（=，in），不支持范围查询（between，>，<，…）
2. 无法利用索引完成排序操作
3. 查询效率高，通常(不发生hash碰撞)只需要一次检索就可以了，效率通常要高于B+tree索引
在MySQL中，支持hash索引的是Memory引擎，而InnoDB中具有  自适应hash功能  ，hash索引是存储引擎根据B+Tree索引在指定条件下自动构建的。
自适应hash功能：
	InnoDB 会 **自动识别** 哪些索引值被频繁以 **等值查询（= 或 IN）** 访问，然后在内存中为这些热点值构建 **哈希索引**，从而把查询复杂度从 B+ 树的 **O(log n)** 降到哈希的 **O(1)**。
### 好问题
![[Pasted image 20250825152151.png]]
hash索引只支持等值匹配。
![[Pasted image 20250825154204.png]]
### 索引分类
![[Pasted image 20250825152419.png]]
在InnoDB存储引擎中，根据索引的  存储  形式，又可以分为以下两种：
![[Pasted image 20250825152623.png]]
聚集索引选取规则
1. 如果存在主键，主键索引就是聚集索引。
2. 如果不存在主键，将使用第一个唯一（UNIQUE）索引作为聚集索引。
3. 如果表没有主键，也没有合适的唯一索引，则 InnoDB 会自动生成一个 rowid 作为隐藏的聚集索引。
![[Pasted image 20250825152902.png]]
数据库默认有一个主键用来对应行找，你给主键id就直接找，如果不是id就利用他找到id，再用id找
### 语法
![[Pasted image 20250825154431.png]]
### SQL性能分析：
#### 查执行频率：
![[Pasted image 20250825170900.png]]
查查操作频率最高的，有针对性地设计数据结构。
Window系统的my.ini配置文件可能在这里C:\ProgramData\MySQL\MySQL Server 8.0，注意：ProgramData是隐藏起来的文件
#### 慢查询：
在mysql环境下，用`show variables like 'slow_query_log'`代码查看是否开启。
![[Pasted image 20250825171535.png]]
Window系统的my.ini配置文件可能在这里C:\ProgramData\MySQL\MySQL Server 8.0，注意：ProgramData是隐藏起来的文件
Linux是查看慢日志文件中记录的信息  `/var/lib/mysql/localhost-slow.log`
#### profile详情
查看sql执行地耗时。
开启和设置：
![[Pasted image 20250825172724.png]]
使用：
![[Pasted image 20250825172748.png]]
#### explain执行计划
![[Pasted image 20250825173157.png]]
EXPLAIN 执行计划各字段含义：
- **id**  （重要）
    select 查询的序列号，表示查询中执行 select 子句或操作表的顺序（id 相同，执行顺序从上到下；id 不同，值越大，越先执行）。
- **select_type**  
    表示 SELECT 的类型，常见取值：
    - SIMPLE：简单表，即不使用表连接或子查询
    - PRIMARY：主查询，即外层查询
    - UNION：UNION 中的第二个或后面的查询语句
    - SUBQUERY：SELECT/WHERE 之后包含的子查询
- **type**  （重要）
    表示连接类型，性能由好到差依次为：NULL、system、const、eq_ref、ref、range、index、all。
- **possible_keys** (关注) 
    显示可能应用在这张表上的索引，一个或多个。
- **Key**  （关注）
    实际使用的索引，如果为 NULL，则没有使用索引。
- **Key_len**  （关注）
    表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度；在不损失精确性的前提下，长度越短越好。
- **rows**  
    MySQL 认为必须执行查询的行数。在 InnoDB 引擎的表中，这是一个估计值，可能并不总是准确。
- **filtered**  
    表示返回结果的行数占需读取行数的百分比，filtered 的值越大越好。
- **Extra**  （关注）
    额外的信息。
### 索引使用
#### 验证索引效率
![[Pasted image 20250825175056.png]]
后面加上\G可以解决查询后，框挤压地效果。
#### 最左前缀法则
联合索引的使用要保证联合索引中最左存在,中间不能跳过
![[Pasted image 20250825175459.png]]
#### 范围查询：
![[Pasted image 20250825180501.png]]
有时你会误以为使用了更多字段，其实是“被用来排序或返回了”，不一定是“用于查找条件”。
key_len 是一个 预估值，表示能用多少联合索引字段；
如果范围查询阻断了后续字段的使用，key_len 会对应地“截断”；
索引失效和字段放的位置无关，只要存在就有效！！
解决方法：
	如果业务允许的情况下将>,< 改为 >=,<=即可
#### 索引失效情况
不要在索引列上进行运算操作，索引将失效。
字符串类型字段使用时，不加引号，索引将失效（字符串不加单引号，隐式类型转换）。
如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。
	原因：**% 在左边，MySQL 不知道从索引哪个位置开始找，只能全表扫描，索引就失效了。**
用 or 分割开的条件，如果 or 前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。
数据分布影响 :如果MySQL评估使用索引比全表更慢，则不使用索引。
#### SQL提示
![[Pasted image 20250826014331.png]]
#### 覆盖索引
尽量使用覆盖索引（查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到），减少 select *
![[Pasted image 20250826015942.png]]
覆盖索引：在辅助索引处直接得到目标值，不需要进行回表。
下图是explain语句后Extra栏中的数据：
![[Pasted image 20250826020310.png]]
#### 前缀索引
![[Pasted image 20250826020725.png]]
大致实现原理：
![[Pasted image 20250826021315.png]]
#### 单例&联合索引
要尽量使用单列索引而非联合索引，因为其性能高，且有可能避免回表查询
但需注意创建联合索引的次序，应符合最左前缀法则。
![[Pasted image 20250826021658.png]]
### 设计原则
1．针对于数据量较大，且查询比较频繁的表建立索引。
2．针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引。
3．尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
4．如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
5．尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
6．要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。
7．如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。当优化器知道每列是否包含NULL值时，它可以更好地确定哪个索引最有效地用于查询。
## SQL优化
### 插入数据
批量查入时,插入500条到1000条比较好
![[Pasted image 20250826023109.png]]大批量插入数据：
如果一次性需要插入大批量数据，使用insert语句插入性能较低，此时可以使用MySQL数据库提供的load指令进行插入。
![[Pasted image 20250826023519.png]]
也要注意顺序插入高于乱序。
### 主键优化
![[Pasted image 20250826023901.png]]
![[Pasted image 20250826024149.png]]
主键乱序插入时，容易出现也分裂现象。
![[Pasted image 20250826024447.png]]
MERGE_THRESHOLD：合并页的阈值，可以自己设置，在创建表或者创建索引时指定。
![[Pasted image 20250826024634.png]]
2、3的原因都是：减少页分裂出现的可能。
### order by优化
前提用的时覆盖索引。
Using indes性能 > Using filesort性能
![[Pasted image 20250826151502.png]]
一升一降会出现using filefort
![[Pasted image 20250826151646.png]]
#### 小结
![[Pasted image 20250826152050.png]]
### group by优化
主要时针对索引的。
- 在分组操作时，可以通过索引来提高效率。
- 分组操作时，索引的使用也是满足最左前缀法则的。
### limit优化
关于大数据分页查询，越往后面查询效率越低。
一个常见又非常头疼的问题就是 limit 2000000,10 ，此时需要MySQL排序前2000010 记录，仅仅返回2000000 - 2000010 的记录，其他记录丢弃，查询排序的代价非常大。

官方给的方法是通过  覆盖索引+子查询  的方式优化
![[Pasted image 20250826153359.png]]
### count优化
由存储引擎决定的。
![[Pasted image 20250827020357.png]]
count的用法
![[Pasted image 20250827020819.png]]
- count（主键） InnoDB 引擎会遍历整张表，把每一行的主键id 值都取出来，返回给服务层。服务层拿到主键后，直接按行进行累加(主键不可能为null)。
- count（字段） 没有not null 约束：InnoDB 引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，服务层判断是否为null，不为null，计数累加。 有not null 约束：InnoDB 引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，直接按行进行累加。
- count（1） InnoDB 引擎遍历整张表，但不取值。服务层对于返回的每一行，放一个数字“1”进去，直接按行进行累加。
- count（*） InnoDB引擎并不会把全部字段取出来，而是专门做了优化，不取值，服务层直接按行进行累加。
![[Pasted image 20250827021257.png]]
update优化
有索引就是行锁，没有索引就是表锁
总结：更新某个字段是一定要走索引，否则走全表扫描会变成表级锁
![[Pasted image 20250827021754.png]]
### 小结
![[Pasted image 20250827022534.png]]

## 视图/存储过程/触发器
### 视图
视图是数据库中基于SQL查询结果的虚拟表。
它不存储数据，只存储查询逻辑，可以用来简化复杂查询、提供数据抽象、增强安全性和实现数据逻辑独立性。视图使得数据访问更加灵活和安全，同时便于数据管理和维护。
#### 基本语法
![[Pasted image 20250827023719.png]]
#### 查询选项
多看看
![[Pasted image 20250827024952.png]]
 **CASCADED**
- **特点**：当选择`CASCADED`选项时，MySQL不仅会检查当前视图定义中的条件，还会递归地检查所有依赖于当前视图的视图。这意味着如果一个视图依赖于另一个视图，那么在进行数据修改时，MySQL会确保这些修改不仅满足当前视图的条件，还要满足所有依赖视图的条件。
- **用途**：适用于需要确保数据修改在整个视图层次结构中都保持一致性的场景。
 **LOCAL**
- **特点**：选择`LOCAL`选项时，MySQL只检查当前视图定义中的条件，而不检查依赖于当前视图的其他视图。这意味着数据修改只需要满足当前视图的条件即可。
- **用途**：适用于不需要或不希望检查依赖视图条件的场景，可以提供更灵活的数据修改操作。
**区别**
- **检查范围**：`CASCADED`会检查所有依赖视图的条件，而`LOCAL`只检查当前视图的条件。
- **默认值**：MySQL的默认值是`CASCADED`，这意味着如果不明确指定，MySQL会默认进行更严格的检查。
#### 更新&作用
![[Pasted image 20250827025602.png]]
作用：
![[Pasted image 20250827025905.png]]
### 存储过程
对SQL语句进行封装与重用，用于减少数据在库和应用服务器之间的传输，提高数据的处理效率。
#### 基本语法
 ![[Pasted image 20250827030807.png]]
![[Pasted image 20250827031003.png]]
![[Pasted image 20250827031014.png]]
注意: 在命令行中，执行创建存储过程的SQL时，需要通过关键字 delimiter 指定SQL语句的结束符。
#### 变量
1. 系统变量
![[Pasted image 20250827031715.png]]
事物提交默认是开启的,默认为1
![[Pasted image 20250827032306.png]]
2. 用户自定义变量
![[Pasted image 20250827032426.png]]
注意：用户定义的变量无需对其进行声明或初始化，只不过获取到的值为NULL。
3. 局部变量
![[Pasted image 20250827032851.png]]
#### 参数
![[Pasted image 20250827033614.png]]
#### 函数
1. if
![[Pasted image 20250827033149.png]]
2. case
![[Pasted image 20250827034335.png]]
concat函数的作用是字符串拼接。
3. while
![[Pasted image 20250827034621.png]]
4. repeat
![[Pasted image 20250827035057.png]]
5. loop
- LOOP 实现简单的循环，如果不在SQL逻辑中增加退出循环的条件，可以用其来实现简单的死循环。LOOP可以配合一下两个语句使用：
    - LEAVE：配合循环使用，退出循环。
    - ITERATE：必须用在循环中，作用是跳过当前循环剩下的语句，直接进入下一次循环。
```sql
[begin_label:] LOOP
    SQL逻辑...
END LOOP [end_label];

LEAVE label; -- 退出指定标记的循环体

ITERATE label; -- 直接进入下一次循环
```
#### 游标
之前讲的内容只能处理单行单列的数据。
![[Pasted image 20250827040332.png]]
逻辑： A. 声明游标，存储查询结果集  -->  B. 准备：创建表结构  -->  C. 开启游标  -->
D. 获取游标中的记录  -->  E. 插入数据到新表中  -->  F. 关闭游标
声明时，要先声明局部变量，再声明游标。
#### 异常处理
![[Pasted image 20250827041642.png]]
### 存储函数
存储函数必须由返回值。
当二进制日志开启了，则要求指出characteristic。
![[Pasted image 20250827042110.png]]
存储函数用的比较少。
1. 存储函数能做的，存储过程都能做。
2. 存储函数有关弊端，它必须有返回值
存储函数有个重要的用途,就是,跟select 组合 生成分析表.存储过程不适合.
### 触发器
触发器是与表有关的数据库对象，指在 insert/update/delete 之前或之后，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作。
使用别名 OLD 和 NEW 来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在mysql中还只支持行级触发，不支持语句级触发。
![[Pasted image 20250827043218.png]]
行级触发器： 对于表中每一行数据变化触发一次触发器指定操作
语句级触发器：在执行一条DML语句时，无论其影响了多少行数据，只进行一次触发器定义的操作。
#### 语法
![[Pasted image 20250827160710.png]]
![[Pasted image 20250827162118.png]]
### 小结
存储函数有个重要的用途,就是,跟select 组合 生成分析表.存储过程不适合.
![[Pasted image 20250827162637.png]]
## 锁
锁是计算机协调多个进程或线程并发访问某一资源的机制。
在数据库中，除传统的计算资源（CPU、RAM、I/O）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。

MySQL中的锁，按照锁的粒度分，分为以下三类：
1. 全局锁：锁定数据库中的所有表。
2. 表级锁：每次操作锁住整张表。
3. 行级锁：每次操作锁住对应的行数据。
### 全局锁
加上全局锁后，除查询语句外所有语句进入阻塞状态。
![[Pasted image 20250827162948.png]]
`flush tables with read lock;`对当前数据库加上全局锁。 
用mysqldump这个工具进行数据备份。
`mysql -u<访问数据库时的用户名> -p<密码> itcast>itcast.sql`
itcast备份的那个数据，itcast.sql存到那个sql文件中
`unlock tables;`   解锁
![[Pasted image 20250827163908.png]]
问题：
![[Pasted image 20250827164303.png]]
### 表级锁
 表级锁，每次操作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在MyISAM、InnoDB、BDB等存储引擎中。
对于表级锁，主要分为以下三类：
1. 表锁
2. 元数据锁（meta data lock，MDL）
3. 意向锁
#### 表锁
对于表锁，分为两类：
1. 表共享读锁（read lock）会阻塞其他客户端的写，不会阻塞读
![[Pasted image 20250827164816.png]]
2. 表独占写锁（write lock）会阻塞其他客户端的读写，当前可读可写
![[Pasted image 20250827165312.png]]
上面两张图的结构是一样的。
#### 元数据锁（meta data lock，MDL）
元数据锁（Metadata Lock，MDL）的主要作用是防止在读取表元数据时对表结构进行修改，从而避免数据不一致的问题。MDL确保了当一个事务正在访问表（例如，读取或写入数据）时，另一个事务不能修改表的结构。
![[Pasted image 20250827165913.png]]
查看元数据锁：
```sql
select object_type, object_schema, object_name, lock_type, lock_duration 
from performance_schema.metadata_locks;
```
#### 意向锁
为了避免DML在执行时，加的行锁与表锁的冲突，在InnoDB中引入了意向锁，使得表锁不用检查每行数据是否加锁，使用意向锁来减少表锁的检查。

若加表锁时，发现有意向锁，则判断与其是否兼容，兼容则直接加上表锁，不兼容则进入阻塞状态。
1. 意向共享锁（IS）：由语句 select ... lock in share mode添加。
与表锁共享锁（read）兼容，与表锁排它锁（write）互斥。
2. 意向排他锁（IX）：由insert、update、delete、select ... for update 添加。
与表锁共享锁（read）及排它锁（write）都互斥。意向锁之间不会互斥。

可以通过以下SQL，查看意向锁及行锁的加锁情况：
```sql
select object_schema, object_name, index_name, lock_type, lock_mode, lock_data 
from performance_schema.data_locks;
```
### 行级锁
![[Pasted image 20250827172141.png]]
InnoDB实现了以下两种类型的行锁：
1. 共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排它锁。
2. 排他锁（X）：允许获取排他锁的事务更新数据，阻止其他事务获得相同数据集的共享锁和排他锁。
![[Pasted image 20250827172307.png]]
加锁情况：
![[Pasted image 20250827172408.png]]

![[Pasted image 20250827173107.png]]
![[Pasted image 20250827172535.png]]
#### 间隙锁&临建锁
默认情况下，InnoDB在REPEATABLE READ事务隔离级别运行，InnoDB使用next-key锁进行搜索和索引扫描，以防止幻读。
1. 索引上的等值查询(唯一索引)，给不存在的记录加锁时，优化为间隙锁。
2. 索引上的等值查询(普通索引)，向右遍历时最后一个值不满足查询需求时，next-key lock 退化为间隙锁。
3. 索引上的范围查询(唯一索引)--会访问到不满足条件的第一个值为止。

注意：间隙锁唯一目的是防止其他事务插入间隙,造成幻读现象。间隙锁可以共存，一个事务采用的间隙锁不会阻止另一个事务在同一间隙上采用间隙锁。
### 小结
![[Pasted image 20250827174326.png]]
## InnoDB引擎
### 逻辑存储结构
![[Pasted image 20250827175047.png]]
### 架构
MySQL5.5 版本开始，默认使用InnoDB存储引擎，它擅长事务处理，具有崩溃恢复特性，在日常开发中使用非常广泛。下面是InnoDB架构图，左侧为内存结构，右侧为磁盘结构。
![[Pasted image 20250827175317.png]]
#### 内存结构
Buffer Pool：
![[Pasted image 20250827175548.png]]
Change Buffer：
![[Pasted image 20250827234830.png]]
自适应hash索引：
![[Pasted image 20250827235025.png]]
Log Buffer：
![[Pasted image 20250827235200.png]]
#### 磁盘结构
![[Pasted image 20250827235932.png]]
![[Pasted image 20250828000040.png]]
![[Pasted image 20250828000205.png]]
#### 后台线程
作用：将InnoDB存储引擎的缓冲池中的数据，在合适的时候刷新到本地磁盘中。
![[Pasted image 20250828000705.png]]
### 事物原理
事务是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。
特性：ACID。
实现方式：
![[Pasted image 20250828001004.png]]
- 原子性 - undo log
- 持久性 - redo log
- 一致性 - undo log + redo log
- 隔离性 - 锁 + MVCC
#### RedoLog：
![[Pasted image 20250828002025.png]]
若操作数据不在内存中时，才会进行1，2操作。
事务commit时才会执行4.
每过一段时间会去清理磁盘中的RedoLog日志。
为什么要把RedoLog刷新到磁盘中？
	若只用BufferPool事务的提交是随机的，对磁盘进行随机磁盘IO
	日志时顺序磁盘IO，性能是要高于随机磁盘IO的。
	内存在断电后会丢失数据。
#### UndoLog：
MVCC和事务回滚操作时会用到这个。
![[Pasted image 20250828002929.png]]
### MVCC
1.  当前读：
读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，如：`select ... lock in share mode(共享锁)，select ... for update、update、insert、delete(排他锁)`都是一种当前读。
2.  快照读
简单的select（不加锁）就是快照读，快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读。
- Read Committed：每次select，都生成一个快照读。
- Repeatable Read：开启事务后第一个select语句才是快照读的地方。
- Serializable：快照读会退化为当前读。
3.  MVCC
全称 Multi-Version Concurrency Control，多版本并发控制。指维护一个数据的多个版本，使得读写操作没有冲突，快照读为MySQL实现MVCC提供了一个非阻塞读功能。

MVCC的具体实现，还需要依赖于数据库记录中的三个隐式字段、undo log日志、readView。
#### 隐藏字段
![[Pasted image 20250828003901.png]]
可以查看表空间文件来进行验证。
#### undo log
回滚日志，在insert、update、delete的时候产生的便于数据回滚的日志。

当insert的时候，产生的undo log日志只在回滚时需要，在事务提交后，可被立即删除。 而update、delete的时候，产生的undo log日志不仅在回滚时需要，在快照读时也需要，不会立即被删除。
![[Pasted image 20250828004818.png]]
不同事务或相同事务对同一条记录进行修改，会导致该记录的undo log生成一条记录版本链表，链表的头部是最新的旧记录，链表尾部是最早的旧记录。
#### readView
作用：undo log 的回滚版本。
![[Pasted image 20250828005051.png]]
规则：
![[Pasted image 20250828005256.png]]
不同的隔离级别，生成ReadView的时机不同：
- READ COMMITTED（RC）：在事务中每一次执行快照读时生成ReadView。
- REPEATABLE READ（RR）：仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView。
##### RC
READ COMMITTED（RC）：在事务中每一次执行快照读时生成ReadView。
![[Pasted image 20250828010351.png]]
##### RR
REPEATABLE READ（RR）：仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView。
![[Pasted image 20250828010620.png]]
所得到的ReadView相同，根据条件得到的结果便也是相同的，即得到的回滚版本也是相同的
## MySQL管理
### 系统数据库
![[Pasted image 20250828011829.png]]
### 常用工具
![[Pasted image 20250828014444.png]]
#### mysql
![[Pasted image 20250828011955.png]]
#### mysqladmin
mysqladmin 是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态、创建并删除数据库等。
```sql
通过帮助文档查看选项：
    mysqladmin --help
示例：
    mysqladmin -uroot -p123456 drop "test01";
    mysqladmin -uroot -p123456 version;
```
#### mysqlbinlog
![[Pasted image 20250828012641.png]]
#### mysqlshow
![[Pasted image 20250828012958.png]]
#### mysqldump
![[Pasted image 20250828013301.png]]
备份文件应存在/var/lib/mysql-files/
#### mysqlumport/source
导入工具。mysqlumport 只能导入文本文件；source导入sql文件
![[Pasted image 20250828014020.png]]
## 小结
# 运维
## 日志
### 错误日志
错误日志是 MySQL 中最重要的日志之一，它记录了当 mysqld 启动和停止时，以及服务器在运行过程中发生任何严重错误的相关信息。
当数据库出现任何故障导致无法正常使用时，建议首先查看此日志。

该日志是默认开启的，默认存放目录 /var/log/，默认的日志文件名为 mysqld.log。查看日志位置：
`show variables like '%log_error%'`
### 二进制日志
二进制日志（BINLOG）记录了所有的 DDL（数据定义语言）语句和 DML（数据操纵语言）语句，但不包括数据查询（SELECT、SHOW）语句。
作用：①. 灾难时的数据恢复；②. MySQL的主从复制。在MySQL8版本中，默认二进制日志是开启着的，涉及到的参数如下：
`show variables like '%log_bin%'`

日志各式：
![[Pasted image 20250828114415.png]]
日志查看：
![[Pasted image 20250828114723.png]]
日志删除：
![[Pasted image 20250828115141.png]]
也可以在mysql的配置文件中配置二进制日志的过期时间（默认是30天），设置了之后，二进制日志过期会自动删除。
`show variables like '%binlog_expire_logs_seconds%';`
### 查询日志
![[Pasted image 20250828115819.png]]
这个文件比较大，如果用不上最好给它关掉，默认也是关掉的。
### 慢查询日志
记录了执行效率低、速度慢的sql语句。
慢查询日志记录了所有执行时间超过参数 `long_query_time` 设置值并且扫描记录数不小于 `min_examined_row_limit` 的所有的SQL语句的日志，默认未开启。`long_query_time` 默认为 10 秒，最小为 0，精度可以到微秒。
```shell
#慢查询日志
slow_query_log=1
#执行时间参数
long_query_time=2

#记录执行较慢的管理语句
log_slow_admin_statements = 1
#记录执行较慢的未使用索引的语句
log_queries_not_using_indexes = 1

```
默认情况下，不会记录管理语句，也不会记录不使用索引进行查找的查询。可以使用`log_slow_admin_statements`和更改此行为 `log_queries_not_using_indexes`，如下所述。
## 主从复制
将主库的数据变更同步到从库，从而保证主库和从库数据一致。
数据备份、失败迁移，读写分离（增删改走主库，查询走从库），降低单库读写压力。
![[Pasted image 20250828180656.png]]
MySQL 复制的优点主要包含以下三个方面：
1. 主库出现问题，可以快速切换到从库提供服务。
2. 实现读写分离，降低主库的访问压力。
3. 可以在从库中执行备份，以避免备份期间影响主库服务。
### 原理
![[Pasted image 20250828181048.png]]
从上图来看，复制分成三步：
1. Master 主库在事务提交时，会把数据变更记录在二进制日志文件 Binlog 中。
2. 从库读取主库的二进制日志文件 Binlog，写入到从库的中继日志 Relay Log。
3. Slave重做中继日志中的事件，将改变反映它自己的数据。
### 搭建：
#### 服务器环境：
![[Pasted image 20250828181458.png]]
#### 主库配置
1. 修改主库配置文件 `/etc/my.cnf`
![[Pasted image 20250828181912.png]]
2. 重启MySQL服务
```shell
systemctl restart mysqld
```
如果没有返回任何信息，则配置成功；若有则要去查看配置文件。
3. 登录mysql，创建远程连接的账号，并授予主从复制权限
![[Pasted image 20250828182343.png]]
4. 通过指令，查看二进制日志坐标
![[Pasted image 20250828182725.png]]
#### 从库配置
1. 修改从库配置文件 `/etc/my.cnf`
![[Pasted image 20250828182834.png]]
对普通用户只读，可以用`super-read-only=1`设置super用户只读
2. 重启MySQL服务
```shell
systemctl restart mysqld
```
3. 登录mysql，设置主库配置，关联主从两库<在文件中配置>
```shell
CHANGE REPLICATION SOURCE TO SOURCE_HOST='xxx.xxx', SOURCE_USER='xxx', SOURCE_PASSWORD='xxx', SOURCE_LOG_FILE='xxx', SOURCE_LOG_POS=xxx;
```
上述是8.0.23中的语法。如果mysql是 8.0.23 之前的版本，执行如下SQL：
```shell
CHANGE MASTER TO MASTER_HOST='xxx.xxx.xxx.xxx', MASTER_USER='xxx', MASTER_PASSWORD='xxx', MASTER_LOG_FILE='xxx', MASTER_LOG_POS=xxx;
```
![[Pasted image 20250828183345.png]]
4. 开启同步操作
```shell
start replica;    #8.0.22之后
start slave;      #8.0.22之前
```
5. 查看主从同步状态
```shell
show replica status;    #8.0.22之后
show slave status;      #8.0.22之前
```
![[Pasted image 20250828184154.png]]
这两个为Yes，表示状态连接是正常的。
## 分库分表
问题：
随着互联网及移动互联网的发展，应用系统的数据量也是成指数式增长，若采用单数据库进行数据存储，存在以下性能瓶颈：
1. IO瓶颈：热点数据太多，数据库缓存不足，产生大量磁盘IO，效率较低。请求数据太多，带宽不够，网络IO瓶颈。
2. CPU瓶颈：排序、分组、连接查询、聚合统计等SQL会耗费大量的CPU资源，请求数太多，CPU出现瓶颈。
![[Pasted image 20250829003618.png]]
分库分表的中心思想都是将数据分散存储，使得单一数据库/表的数据量变小来缓解单一数据库的性能问题，从而达到提升数据库性能的目的。
### 拆分方式
差分策略：
从粒度上分为：分库和分表；            维度上分为：垂直和水平。
![[Pasted image 20250829003859.png]]
水平拆数据，垂直拆结构
垂直拆分：
![[Pasted image 20250829004334.png]]
水平拆分：
![[Pasted image 20250829005920.png]]
### 实现方式
- shardingJDBC：基于AOP原理，在应用程序中对本地执行的SQL进行拦截，解析、改写、路由处理。需要自行编码配置实现，只支持java语言，性能较高。
- MyCat：数据库分库分表  中间件  ，不用调整代码即可实现分库分表，支持多种语言，性能不及前者。
![[Pasted image 20250829010351.png]]
### Mycat概述
Mycat是开源的、活跃的、基于Java语言编写的MySQL数据库中间件。可以像使用mysql一样来使用mycat，对于开发人员来说根本感觉不到mycat的存在。

优势：性能可靠稳定、强大的技术团队、体系完善、社区活跃
![[Pasted image 20250829010723.png]]
#### 结构图
MyCat是不存具体数据的，只是在逻辑上进行分片处理而已。
![[Pasted image 20250829011725.png]]
### Mycat入门
这个是水平分表。
#### 环境结构：
![[Pasted image 20250829012048.png]]
#### 分片配置文件（schema.xml）：
分片规则在rule.xml文件中。
![[Pasted image 20250829012200.png]]
#### server.xml文件配置：
![[Pasted image 20250829013110.png]]
#### 启动服务
![[Pasted image 20250829013354.png]]
#### 连接并登录
h 主机（IP）；P 端口。
![[Pasted image 20250829013615.png]]
### Mycat配置
#### schema.xml 
schema.xml 作为MyCat中最重要的配置文件之一，涵盖了MyCat的逻辑库、逻辑表、分片规则、分片节点及数据源的配置。
主要包含以下三组标签：
- schema标签
![[Pasted image 20250829014620.png]]
![[Pasted image 20250829014834.png]]
- datanode标签
![[Pasted image 20250829015029.png]]
- datahost标签
![[Pasted image 20250829015141.png]]
#### rule.xml
tableRule 分片规则。
![[Pasted image 20250829015513.png]]
#### server.xml
server.xml配置文件包含了MyCat的系统配置信息，主要有两个重要的标签：system、user。
- system标签
![[Pasted image 20250829015655.png]]
- user标签
![[Pasted image 20250829015902.png]]
### Mycat分片
#### 垂直拆分：
![[Pasted image 20250829020528.png]]
- 准备
![[Pasted image 20250829021126.png]]
- 配置
![[Pasted image 20250829021325.png]]
![[Pasted image 20250829021623.png]]
#### 水平拆分
之前和准备和垂直一样，直接到配置了。
- 配置
![[Pasted image 20250829022631.png]]
### 分片规则
rule.xml可以添加新的规则。
- 1.范围分片
![[Pasted image 20250829023249.png]]
配置：
![[Pasted image 20250829023452.png]]
- 2.取模分片
![[Pasted image 20250829023702.png]]
配置：改两个就行
![[Pasted image 20250829023908.png]]
- 3.一致性hash分片
![[Pasted image 20250829024101.png]]
配置：
![[Pasted image 20250829024157.png]]
- 4.枚举分片
可以指定多个枚举类型，但一张表只能用一种。
![[Pasted image 20250829155329.png]]
配置：
![[Pasted image 20250829155438.png]]
- 5.应用指定算法分片
![[Pasted image 20250829160102.png]]
配置：
![[Pasted image 20250829160215.png]]
超出指定的分片范围时，会走默认分片。
- 6.固定分片hash算法
![[Pasted image 20250829161151.png]]
特点：
- 如果是求模，连续的值，分别分配到各个不同的分片；但是此算法会将连续的值可能分配到相同的分片，降低事务处理的难度。
- 可以均匀分配，也可以非均匀分配。
- 分片字段必须为数字类型。
配置：
![[Pasted image 20250829161630.png]]
与运算后在数组中找响应值，决定去哪个分片。
- 7.字符串hash解析
![[Pasted image 20250829162517.png]]
配置：
![[Pasted image 20250829162831.png]]
- 8.按（天）日期分片
超过end时，并没有停止而是按照end-begin的时间差重新分配一轮。
![[Pasted image 20250829163501.png]]
配置：
![[Pasted image 20250829163839.png]]
- 9.按自然月分片
超过end时，并没有停止而是按照end-begin的时间差重新分配一轮。
![[Pasted image 20250829164703.png]]
配置：
![[Pasted image 20250829164939.png]]
### Mycat管理及监控
原理：
查询时，如果sql语句中有status字段则找到表直接用；没有，则在三张表都进行sql语句，结果返回MyCat。
![[Pasted image 20250829172608.png]]
现在MyCat-Web已经改名为MyCat-eye
#### MyCat管理工具
Mycat默认开通2个端口，可以在server.xml中进行修改。
- 8066 数据访问端口，即进行DML和DDL操作。
- 9066 数据库管理端口，即mycat服务管理控制功能，用于管理mycat的整个集群状态。
```shell
mysql -h 192.168.200.210 -p 9066 -uroot -p123456
```
MyCat管理指令：
![[Pasted image 20250829173146.png]]
#### MyCat管理与监控
● `Mycat-eye`
`Mycat-web(Mycat-eye)`是对mycat-server提供监控服务，功能不局限于对mycat-server使用。他通过JDBC连接对Mycat、Mysql监控，监控远程服务器(目前仅限于linux系统的cpu、内存、网络、磁盘。
Mycat-eye运行过程中需要依赖zookeeper，因此需要先安装zookeeper。
```url
启动MyCat-Web后访问下面网站（注意端口被占的情况）：
http://192.168.200.210:8082/mycat
```
## 读写分离
读写分离，简单地说是把对数据库的读和写操作分开，以对应不同的数据库服务器。主数据库提供写操作，从数据库提供读操作，这样能有效地减轻单台数据库的压力。

通过MyCat即可轻易实现上述功能，不仅可以支持MySQL，也可以支持Oracle和SQL Server。
![[Pasted image 20250829175733.png]]
这里的MyCat中间件不是必要的，但因为读写管理的复杂度太高，所以加了这么一层，也可以用别的。
红色的是两个组件writeHost和readHost，使用时需要配置一下。
### 一主一从
原理：
![[Pasted image 20250829180029.png]]
#### 读写分离
![[Pasted image 20250829180513.png]]
balance属性负责负载均衡策略，目前取值有4种：
![[Pasted image 20250829180629.png]]
1、3均可实现读写分离。
一主一从这种架构实现读写分离会遇到的问题：
	当主节点宕机后，业务系统就只可以读，而不能够写入数据了。
### 双主双从
一个主机 Master1 用于处理所有写请求，它的从机 Slave1 和另一台主机 Master2 还有它的从机 Slave2 负责所有读请求。当 Master1 主机宕机后，Master2 主机负责写请求，Master1、Master2 互为备机。架构图如下：
![[Pasted image 20250829181753.png]]
需要5台服务器。
#### 搭建
- 主库配置
![[Pasted image 20250829182015.png]]
- 第二个主库配置（3号服务器）
只需要将server-id改为3就可以了。其余和主库1配置一样。
- 搭建两主库间的主从复制
![[Pasted image 20250829182513.png]]
两个服务器都要执行上述内容。
- 从库2配置
![[Pasted image 20250829182809.png]]
- 从库4配置
从库4配置和从库2配置一样，只需要将server-id改为4 即可
- 搭建两从库与主库之间的关联
![[Pasted image 20250829183114.png]]
- 两主库相互复制
![[Pasted image 20250829183517.png]]
#### 读写分离
配置：
![[Pasted image 20250829184220.png]]
![[Pasted image 20250829184418.png]]
switchType指的是：当writeHost1挂掉之后，会不会自动切换到writeHost2.
### 小结
![[Pasted image 20250829185203.png]]

