# 数据结构
## 动态字符串SDS
Redis中保存的Key是字符串，value往往是字符串或者字符串的集合。可见字符串是Redis中最常用的一种数据结构。
不过Redis没有直接使用C语言中的字符串，因为C语言字符串存在很多问题：
- 获取字符串长度的需要通过运算 （c语言的字符串是本质数组）
- 非二进制安全
- 不可修改

Redis构建了一种新的字符串结构，称为`简单动态字符串（Simple Dynamic String）`，简称SDS。
例如，我们执行命令：
```redis
set name 虎哥
```
那么Redis将在底层创建两个SDS，其中一个是包含“name”的SDS，另一个是包含“虎哥”的SDS。
![[Pasted image 20251003171307.png]]

SDS之所以叫做动态字符串，是因为它具备动态扩容的能力。
- 如果新字符串小于1M，则新空间为扩展后字符串长度的两倍+1；
- 如果新字符串大于1M，则新空间为扩展后字符串长度+1M+1。称为  **内存预分配**（减低频繁的内存分配申请造成的资源消耗问题）。
优点：
获取字符串长度的时间复杂度为O(1)、支持动态扩容、减少内存分配次数、二进制安全
## `IntSet`
IntSet是Redis中set集合的一种实现方式，基于整数数组来实现，并且具备长度可变、有序等特征。
![[Pasted image 20251003174036.png]]
为了方便查找，Redis会将intset中所有的整数按照升序依次保存在contents数组中，结构如图：
数组角标都是从0开始  ->  为了方便寻址，性能考虑
![[Pasted image 20251003174505.png]]
现在，数组中每个数字都在int16_t的范围内，因此采用的编码方式是INTSET_ENC_INT16，每部分占用的字节大小为：
encoding：4字节
length：4字节
contents：2字节 * 3  = 6字节
### IntSet升级
现在，假设有一个intset，元素为{5,10，20}，采用的编码是INTSET_ENC_INT16，则每个整数占2字节：

向该其中添加一个数字：50000，这个数字超出了int16_t的范围，intset会  **自动升级编码方式**  到合适的大小。
![[Pasted image 20251003175304.png]]
最后面元素的首尾值`*`2向前遍历做相同操作（不会出现数据覆盖），把新来的大数字放在12-16位置。

详细流程如下：
1. 升级编码为INTSET_ENC_INT32, 每个整数占4字节，并按照新的编码方式及元素个数扩容数组
2. 倒序依次将数组中的元素拷贝到扩容后的正确位置
3. 将待添加的元素放入数组末尾
4. 最后，将inset的encoding属性改为INTSET_ENC_INT32，将length属性改为4
### 小结
Intset可以看做是特殊的整数数组，具备一些特点：
1. Redis会确保Intset中的元素唯一、有序
2. 具备类型升级机制，可以节省内存空间
3. 底层采用二分查找方式来查询
推荐数据量不多的情况下使用。
## `Dict`(`HashTable`)
Redis是一个键值型（Key-Value Pair）的数据库，可以根据键实现快速的增删改查。而键与值的映射关系正是通过Dict来实现的。

Dict由三部分组成，分别是：哈希表（DictHashTable）、哈希节点（DictEntry）、字典（Dict）
![[Pasted image 20251003181426.png]]
当向Dict添加键值对时，Redis首先根据key计算出hash值（h），然后利用  `h & sizemask` (与运算与取模<余数>运算的效果相等,但性能更高),
来计算元素应该存储到数组中的哪个索引位置。
### 内存结构图
![[Pasted image 20251003231024.png]]
### Dict的扩容
Dict中的HashTable就是数组结合单向链表的实现，当集合中元素较多时，必然导致哈希冲突增多，链表过长，则查询效率会大大降低。

Dict在每次新增键值对时都会检查  **负载因子**（`LoadFactor = used/size`） ，满足以下两种情况时会触发哈希表扩容：
- 哈希表的 `LoadFactor` >= 1，并且服务器没有执行 BGSAVE 或者 BGREWRITEAOF 等后台进程；
- 哈希表的 `LoadFactor` > 5 ；
```c
static int _dictExpandIfNeeded(dict *d){
    // 如果正在rehash，则返回ok
    if (dictIsRehashing(d)) return DICT_OK;    
    // 如果哈希表为空，则初始化哈希表为默认大小：4
    if (d->ht[0].size == 0) return dictExpand(d, DICT_HT_INITIAL_SIZE);
    // 当负载因子（used/size）达到1以上，并且当前没有进行bgrewrite等子进程操作
    // 或者负载因子超过5，则进行 dictExpand ，也就是扩容
    if (d->ht[0].used >= d->ht[0].size &&
        (dict_can_resize || d->ht[0].used/d->ht[0].size > dict_force_resize_ratio){
        // 扩容大小为used + 1，底层会对扩容大小做判断，实际上找的是第一个大于等于 used+1 的 2^n
        return dictExpand(d, d->ht[0].used + 1);
    }
    return DICT_OK;
}
```
### Dict的收缩
Dict除了扩容以外，每次删除元素时，也会对负载因子做检查，当LoadFactor < 0.1 时，会做哈希表收缩：
![[Pasted image 20251003182310.png]]
可以去看看dictExpand怎么写的，视频也有讲解。
### Dict的rehash
不管是扩容还是收缩，必定会创建新的哈希表，
导致哈希表的size和sizemask变化，而key的查询与sizemask有关。因此,
对哈希表中的每一个key重新计算索引，插入新的哈希表，这个过程称为**rehash**。过程是这样的：

1、计算新hash表的realeSize，值取决于当前要做的是扩容还是收缩：
- 如果是扩容，则新size为第一个大于等于`dict.ht[0].used + 1的2^n`
- 如果是收缩，则新size为第一个大于等于`dict.ht[0].used的2^n`（不得小于4）
2、按照新的realeSize申请内存空间，创建dictht，并赋值给`dict.ht[1]`
3、设置dict.rehashidx = 0，标示开始rehash
4、将`dict.ht[0]`中的每一个dictEntry都rehash到`dict.ht[1]`
5、将`dict.ht[1]`赋值给`dict.ht[0]`，给`dict.ht[1]`初始化为空哈希表，释放原来的`dict.ht[0]`的内存

这个过程强烈去看视频ppt的动画讲解。
#### 为了避免主线程的阻塞
Dict的rehash并不是一次性完成的。试想一下，如果Dict中包含数百万的entry，要在一次rehash完成，极有可能导致主线程阻塞。
因此Dict的rehash是分多次、渐进式的完成，因此称为**渐进式rehash**。

流程和上面相似之说更改地方：
- 4步改为了：④每次执行新增、查询、修改、删除操作时，都检查一下dict.rehashidx是否大于-1，如果是则将`dict.ht[0].table[rehashidx]`的entry链表rehash到`dict.ht[1]`，并且将rehashidx++。直至`dict.ht[0]`的所有数据都rehash到`dict.ht[1]`
	也就是，每次做rehash的时候，只迁移了数组一个角标上的链表，其它的没管，每次增删改做一次（渐进式），知道ht0全部迁移为止。

后续加上
6. 将rehashidx赋值为-1，代表rehash结束
7. 在rehash过程中，新增操作，则直接写入`ht[1]`，查询、修改和删除则会在`dict.ht[0]`和`dict.ht[1]`依次查找并执行。这样可以确保`ht[0]`的数据只减不增，随着rehash最终为空
### 小结
![[Pasted image 20251003184457.png]]
## `ZipList`
`ZipList` 是一种特殊的“双端链表”（但它根本不是链表） ，由一系列特殊编码的连续内存块组成。可以在任意一端进行压入/弹出操作, 并且该操作的时间复杂度为 O(1)。
大概结构：
![[Pasted image 20251003212137.png]]

| 属性      | 类型       | 长度   | 用途                                                                                 |
| ------- | -------- | ---- | ---------------------------------------------------------------------------------- |
| zlbytes | uint32_t | 4 字节 | 记录整个压缩列表占用的内存字节数                                                                   |
| zltail  | uint32_t | 4 字节 | 记录压缩列表表尾节点距离压缩列表的起始地址有多少字节，通过这个偏移量，可以确定表尾节点的地址。                                    |
| zllen   | uint16_t | 2 字节 | 记录了压缩列表包含的节点数量。最大值为UINT16_MAX（65534），如果超过这个值，此处会记录为65535，但节点的真实数量需要遍历整个压缩列表才能计算得出。 |
| entry   | 列表节点     | 不定   | 压缩列表包含的各个节点，节点的长度由节点保存的内容决定。                                                       |
| zlend   | uint8_t  | 1 字节 | 特殊值 0xFF（十进制 255），用于标记压缩列表的末端。                                                     |
|         |          |      |                                                                                    |
### `ZipListEntry`
`ZipList`中的Entry并不像普通链表那样记录前后节点的指针，因为记录两个指针要占用16个字节，浪费内存。而是采用了下面的结构：
![[Pasted image 20251003212653.png]]
1、previous_entry_length：前一节点的长度，占1个或5个字节。
- 如果前一节点的长度小于254字节，则采用1个字节来保存这个长度值
- 如果前一节点的长度大于254字节，则采用5个字节来保存这个长度值，第一个字节为0xfe，后四个字节才是真实长度数据
2、encoding：编码属性，记录content的数据类型（字符串还是整数）以及长度，占用1个、2个或5个字节
3、contents：负责保存节点的数据，可以是字符串或整数

注意：
	ZipList中所有存储长度的数值均采用  **小端字节序**，即低位字节在前，高位字节在后。例如：数值0x1234，采用小端字节序后实际存储值为：0x3412

ZipListEntry不是通过记录指针而是通过记录长度，来推算出前一个后一个节点的位置。
#### Encoding编码
ZipListEntry中的encoding编码分为字符串和整数两种：
- 字符串：如果encoding是以“00”、“01”或者“10”开头，则证明content是字符串
![[Pasted image 20251003213434.png]]
例如，我们要保存字符串：`“ab”和 “bc”`
上面这个是ab：
![[Pasted image 20251003213805.png]]
因为前面没有ZipListEntry元素所以第一个是 0x00，ab占用了两个字节所以第二个是0x02，
0x61和0x62是a和b对应的ASCII码。

把ab和bc放在一起得到的`ZipListEntry`结构如下：
![[Pasted image 20251003214103.png]]


- 整数：如果encoding是以“11”开始，则证明content是整数，且encoding固定只占用1个字节
![[Pasted image 20251003214728.png]]
算法同上，结束标识也是1111.
注意：
	特别小的数字，直接在xxxx位置保存数值，范围从0001~1101，减1后结果为实际值
### ZipList的连锁更新问题
ZipList的每个Entry都包含previous_entry_length来记录上一个节点的大小，长度是1个或5个字节：
- 如果前一节点的长度小于254字节，则采用1个字节来保存这个长度值
- 如果前一节点的长度大于等于254字节，则采用5个字节来保存这个长度值，第一个字节为0xfe，后四个字节才是真实长度数据

现在，假设我们有N个连续的、长度为250~253字节之间的entry，因此entry的previous_entry_length属性用1个字节即可表示，如图所示：
![[Pasted image 20251003220129.png]]
导致后面的节点连续崩坏。
ZipList这种特殊情况下产生的连续多次空间扩展操作称之为  **连锁更新（Cascade Update）**。新增、删除都可能导致连锁更新的发生。
### 小结
ZipList特性：
1. 压缩列表的可以看做一种连续内存空间的"双向链表"
2. 列表的节点之间不是通过指针连接，而是记录上一节点和本节点长度来寻址，内存占用较低
3. 如果列表数据过多，导致链表过长，可能影响查询性能
4. 增或删较大数据时有可能发生连续更新问题
## `QuickList`
问题1：ZipList虽然节省内存，但申请内存必须是连续空间，如果内存占用较多，申请内存效率很低。怎么办？
- 为了缓解这个问题，我们必须限制ZipList的长度和entry大小。
问题2：但是我们要存储大量数据，超出了ZipList最佳的上限该怎么办？
- 我们可以创建多个ZipList来分片存储数据。
问题3：数据拆分后比较分散，不方便管理和查找，这多个ZipList如何建立联系？
- Redis在3.2版本引入了新的数据结构QuickList，它是一个双端链表，只不过链表中的每个节点都是一个ZipList。
![[Pasted image 20251003220612.png]]

为了避免QuickList中的每个ZipList中entry过多，Redis提供了一个配置项：list-max-ziplist-size来限制。
- 如果值为正，则代表ZipList的允许的entry个数的最大值
- 如果值为负，则代表ZipList的最大内存大小，分5种情况：

-1：每个ZipList的内存占用不能超过4kb
-2：每个ZipList的内存占用不能超过8kb
-3：每个ZipList的内存占用不能超过16kb
-4：每个ZipList的内存占用不能超过32kb
-5：每个ZipList的内存占用不能超过64kb
其默认值为 -2：


除了控制ZipList的大小，QuickList还可以对节点的ZipList做压缩。通过配置项list-compress-depth来控制。因为链表一般都是从首尾访问较多，所以首尾是不压缩的。这个参数是控制首尾不压缩的节点个数：
- 0：特殊值，代表不压缩
- 1：标示QuickList的首尾各有1个节点不压缩，中间节点压缩
- 2：标示QuickList的首尾各有2个节点不压缩，中间节点压缩
以此类推
默认值： 为0
### 结构源码
![[Pasted image 20251003221214.png]]
### 内存结构图
![[Pasted image 20251003221343.png]]
### 小结
QuickList的特点：
- 是一个节点为ZipList的双端链表
- 节点采用ZipList，解决了传统链表的内存占用问题
- 控制了ZipList大小，解决连续内存空间申请效率问题
- 中间节点可以压缩，进一步节省了内存
## `SkipList`
SkipList（跳表）首先是链表，但与传统链表相比有几点差异：
- 元素按照升序排列存储
- 节点可能包含多个指针，指针跨度不同。
![[Pasted image 20251003221858.png]]
调表中最多允许32级指针。
### 代码实现
![[Pasted image 20251003222248.png]]
### 内存结构图
![[Pasted image 20251003222853.png]]
### 小结
SkipList的特点：
- 跳跃表是一个双向链表，每个节点都包含score和ele值
- 节点按照score值排序，score值一样则按照ele字典排序
- 每个节点都可以包含多层指针，层数是1到32之间的随机数
- 不同层指针到下一个节点的跨度不同，层级越高，跨度越大
- 增删改查效率与红黑树基本一致，实现却更简单
## `RedisObject`
上面学习的都是数据结构。
Redis中的任意数据类型的键和值都会被封装为一个RedisObject，也叫做Redis对象，源码如下：
![[Pasted image 20251003223035.png]]
String类型虽然是最简单的，但每个String类型都需要一个id头部，有大量的内存浪费在头信息上了，所以不太建议使用这种类型。（使用其它类型的集合是多个数据用一个id头）
### Redis的编码方式
Redis中会根据存储的数据类型不同，选择不同的编码方式，共包含11种不同类型：
![[Pasted image 20251003223649.png]]

Redis中会根据存储的数据类型不同，选择不同的编码方式。每种数据类型的使用的编码方式如下：
![[Pasted image 20251003223727.png]]
## 五种数据类型
### String
String是Redis中最常见的数据存储类型：
- 其基本编码方式是  **RAW**  ，基于简单动态字符串（SDS）实现，存储上限为512mb。
![[Pasted image 20251003224219.png]]
- 如果存储的SDS长度小于44字节，则会采用  **EMBSTR编码**  ，此时object head与SDS是一段连续空间。申请内存时只需要调用一次内存分配函数，效率更高。
![[Pasted image 20251003224245.png]]
- 如果存储的字符串是整数值，并且大小在LONG_MAX范围内，则会采用  **INT编码**  ：直接将数据保存在RedisObject的ptr指针位置（刚好8字节），不再需要SDS了。
![[Pasted image 20251003224722.png]]
### List
记忆回顾Tip:
- LinkedList ：普通链表，可以从双端访问，内存占用较高，内存碎片较多
- `ZipList` ：压缩列表，可以从双端访问，内存占用低，存储上限低
- `QuickList`：`LinkedList + ZipList`，可以从双端访问，内存占用较低，包含多个ZipList，存储上限高

Redis的List结构类似一个双端链表，可以从首、尾操作列表中的元素：
- 在3.2版本之前，Redis采用ZipList和LinkedList来实现List，当元素数量小于512并且元素大小小于64字节时采用ZipList编码，超过则采用LinkedList编码。
- 在3.2版本之后，Redis统一采用QuickList来实现List
![[Pasted image 20251003230050.png]]
说白了这个结构就是在QuickList的基础上包了一层RedisObject头。
### Set
Set是Redis中的单列集合，满足下列特点：
- 不保证有序性
- 保证元素唯一
- 求交集、并集、差集

可以看出，Set对查询元素的效率要求非常高，思考一下，什么样的数据结构可以满足？
- HashTable，也就是Redis中的Dict，不过Dict是双列集合（可以存键、值对）

Set是Redis中的集合，不一定确保元素有序，可以满足元素唯一、查询效率要求极高。
- 为了查询效率和唯一性，set采用  **`HT编码(Dict)`** 。Dict中的key用来存储元素，value统一为null。
- 当存储的所有数据都是整数，并且元素数量不超过set-max-intset-entries时，Set会采用IntSet编码，以节省内存。
set-max-intset-entries的默认值是512
#### 内存图
![[Pasted image 20251003231712.png]]
把所有红色去掉就是原来有全数字组成的Set。
当插入一个字符串时，指针由指向IntSet转为创建dict并指向dict、并改变编码方式
### `ZSet`
ZSet也就是SortedSet，其中每一个元素都需要指定一个score值和member值：
- 可以根据score值排序后
- member必须唯一 (若重复后添加member的score值就会覆盖之前的)
- 可以根据member查询分数 

因此，zset底层数据结构必须满足  **键值存储、键必须唯一、可排序**  这几个需求。之前学习的哪种编码结构可以满足？
- SkipList：可以排序，并且可以同时存储score和ele值（member） <无法实现高效的键值唯一性检查>
- `HT（Dict`：可以键值存储，并且可以根据key找value <不满足可排序>
这两个都用了
![[Pasted image 20251003232455.png]]
#### 内存结构
![[Pasted image 20251003232821.png]]
这个版本的ZSet非常占用内存。
#### 第二种实现方式
当元素数量不多时，HT和SkipList的优势不明显，而且更耗内存。因此zset还会采用  **ZipList结构**  来节省内存，不过需要同时满足两个条件：
1. 元素数量小于zset_max_ziplist_entries，默认值128
2. 每个元素都小于zset_max_ziplist_value字节，默认值64

ziplist本身没有排序功能，而且没有键值对的概念，因此需要有zset通过编码实现：
- ZipList是连续内存，因此score和element是紧挨在一起的两个entry， element在前，score在后
- score越小越接近队首，score越大越接近队尾，按照score值升序排列
结构图：
![[Pasted image 20251003233818.png]]

若zset_max_ziplist_entries为0或者zset_max_ziplist_value初始化的元素超过64字节，就采用SkipList编码，否则就采用ZipList编码。
注意：
	这个也会有编码转换的风险。
### Hash
Hash结构与Redis中的Zset非常类似：
- 都是键值存储
- 都需求根据键获取值
- 键必须唯一
区别如下：
- zset的键是member，值是score；hash的键和值都是任意值
- zset要根据score排序；hash则无需排序
因此，Hash底层采用的编码与Zset也基本一致，只需要把排序有关的SkipList去掉即可
#### 实现
- Hash结构默认采用ZipList编码，用以节省内存。 ZipList中相邻的两个entry 分别保存field和value
![[Pasted image 20251003234359.png]]
- 当数据量较大时，Hash结构会转为HT编码，也就是Dict，触发条件有两个：
1、ZipList中的元素数量超过了hash-max-ziplist-entries（默认512）
2、ZipList中的任意entry大小超过了hash-max-ziplist-value（默认64字节）
![[Pasted image 20251003234613.png]]
转换时，指针由ZipList指向dict，更换编码格式。


# Redis网络模型
![[Pasted image 20251004082026.png]]
在《UNIX网络编程》一书中，总结归纳了5种IO模型：
- 阻塞IO（Blocking IO）
- 非阻塞IO（Nonblocking IO）
- IO多路复用（IO Multiplexing）
- 信号驱动IO（Signal Driven IO）
- 异步IO（Asynchronous IO）
不同的IO模型的差别就是在图中1，2阶段的处理上有所区别。
## 用户空间和内核空间
任何Linux发行版，其系统内核都是Linux。我们的应用都需要通过Linux内核与硬件交互。
![[Pasted image 20251003235904.png]]
为了避免用户应用导致冲突甚至内核崩溃，用户应用与内核是分离的：
- 进程的寻址空间会划分为两部分：**内核空间、用户空间**
- 用户空间只能执行受限的命令（Ring3），而且不能直接调用系统资源，必须通过内核提供的接口来访问
- 内核空间可以执行特权命令（Ring0），调用一切系统资源
![[Pasted image 20251004000858.png]]
```知识补充
寻址空间：内核和用户都无法直接访问物理内存，而是给它们分配不同的虚拟内存空间，映射到不同的物理内存
访问虚拟内存空间需要一个虚拟地址（无符号整数从0开始，最大值取决于CPU的地址总线和寄存器的带宽），这段0-最大值就是  寻址空间  。
而内存地址的每一个值就是一个存储单元，也就是一个字节
```
Linux系统为了提高IO效率，会在用户空间和内核空间都加入缓冲区：
- 写数据时，要把用户缓冲数据拷贝到内核缓冲区，然后写入设备
- 读数据时，要从设备读取数据到内核缓冲区，然后拷贝到用户缓冲区

提高IO效率的核心点：减少无效等待，减少内核态和用户态之间的数据拷贝。
## 阻塞IO
![[Pasted image 20251004082312.png]]顾名思义，阻塞IO就是两个阶段都必须阻塞等待：
阶段一：
1. 用户进程尝试读取数据（比如网卡数据）
2. 此时数据尚未到达，内核需要等待数据
3. 此时用户进程也处于阻塞状态

阶段二：
1. 数据到达并拷贝到内核缓冲区，代表已就绪
2. 将内核数据拷贝到用户缓冲区
3. 拷贝过程中，用户进程依然阻塞等待
4. 拷贝完成，用户进程解除阻塞，处理数据
可以看到，阻塞IO模型中，用户进程在两个阶段都是阻塞状态。
## 非阻塞IO
区别就是非阻塞IO查询失败后内核会返回一个错误信息。
顾名思义，非阻塞IO的recvfrom操作会立即返回结果而不是阻塞用户进程。
![[Pasted image 20251004082500.png]]
阶段一：
1. 用户进程尝试读取数据（比如网卡数据）
2. 此时数据尚未到达，内核需要等待数据
3. 返回异常给用户进程
4. 用户进程拿到error后，再次尝试读取
5. 循环往复，直到数据就绪

阶段二：
1. 将内核数据拷贝到用户缓冲区
2. 拷贝过程中，用户进程依然阻塞等待
3. 拷贝完成，用户进程解除阻塞，处理数据
可以看到，非阻塞IO模型中，用户进程在第一个阶段是非阻塞，第二个阶段是阻塞状态。虽然是非阻塞，但性能并没有得到提高。而且忙等机制会导致CPU空转，CPU使用率暴增。
## IO多路复用
无论是阻塞IO还是非阻塞IO，用户应用在一阶段都需要调用recvfrom来获取数据，差别在于无数据时的处理方案：
- 如果调用recvfrom时，恰好没有数据，阻塞IO会使CPU阻塞，非阻塞IO使CPU空转，都不能充分发挥CPU的作用。
- 如果调用recvfrom时，恰好有数据，则用户进程可以直接进入第二阶段，读取并处理数据
而在单线程情况下，只能依次处理IO事件，如果正在处理的IO事件恰好未就绪（数据不可读或不可写），线程就会被阻塞，所有IO事件都必须等待，性能自然会很差。
![[Pasted image 20251004083201.png]]

**文件描述符（File Descriptor）**：
简称FD，是一个从0 开始的无符号整数，用来关联Linux中的一个文件。在Linux中，一切皆文件，例如常规文件、视频、硬件设备等，当然也包括网络套接字（Socket）。
**IO多路复用**：
是利用单个线程来同时监听多个FD，并在某个FD可读、可写时得到通知，从而避免无效的等待，充分利用CPU资源。
![[Pasted image 20251004083352.png]]
阶段一：
1. 用户进程调用select，指定要监听的FD集合
2. 内核监听FD对应的多个socket
3. 任意一个或多个socket数据就绪则返回readable
4. 此过程中用户进程阻塞

阶段二：
1. 用户进程找到就绪的socket
2. 依次调用recvfrom读取数据
3. 内核将数据拷贝到用户空间
4. 用户进程处理数据
### 监听方式
不过监听FD的方式、通知的方式又有多种实现，常见的有：
`select、poll、epoll`
差异：
- select和poll只会通知用户进程有FD就绪，但不确定具体是哪个FD，需要用户进程逐个遍历FD来确认
- epoll则会在通知用户进程FD就绪的同时，把已就绪的FD写入用户空间
#### select
最早的IO多路复用的实现方案。
```C
// 定义类型别名 __fd_mask，本质是 long int
typedef long int __fd_mask;

/* fd_set 记录要监听的fd集合，及其对应状态 */
typedef struct {

    // fds_bits是long类型数组，长度为 1024/32 = 32
    // 共1024个bit位，每个bit位代表一个fd，0代表未就绪，1代表就绪
    __fd_mask fds_bits[__FD_SETSIZE / __NFDBITS];
    // ...

} fd_set;

// select函数，用于监听fd_set，也就是多个fd的集合
int select(
    int nfds, // 要监视的fd_set的最大fd + 1
    fd_set *readfds, // 要监听读事件的fd集合
    fd_set *writefds,// 要监听写事件的fd集合
    fd_set *exceptfds, // // 要监听异常事件的fd集合
    // 超时时间，null-用不超时；0-不阻塞等待；大于0-固定等待时间
    struct timeval *timeout
);
```
![[Pasted image 20251004085212.png]]
select模式存在的问题：
1. 需要将整个fd_set从用户空间拷贝到内核空间，select结束还要再次拷贝回用户空间
2. select无法得知具体是哪个fd就绪，需要遍历整个fd_set
3. 监听的数量很少，fd_set监听的fd数量不能超过1024
#### poll
poll模式对select模式做了简单改进，但性能提升不明显，部分关键代码如下：
```C
// pollfd 中的事件类型

#define POLLIN     //可读事件
#define POLLOUT    //可写事件
#define POLLERR    //错误事件
#define POLLNVAL   //fd未打开

// pollfd结构
struct pollfd {
    int fd;         /* 要监听的fd  */
    short int events; /* 要监听的事件类型：读、写、异常 */
    short int revents;/* 实际发生的事件类型 */

};

  
// poll函数
int poll(
    struct pollfd *fds, // pollfd数组，可以自定义大小
    nfds_t nfds, // 数组元素个数
    int timeout // 超时时间
);
```

IO流程：
1. 创建pollfd数组，向其中添加关注的fd信息，数组大小自定义
2. 调用poll函数，将pollfd数组拷贝到内核空间，转链表存储，无上限
3. 内核遍历fd，判断是否就绪
4. 数据就绪或超时后，拷贝pollfd数组到用户空间，返回就绪fd数量n
5. 用户进程判断n是否大于0
6. 大于0则遍历pollfd数组，找到就绪的fd

与select对比：
- select模式中的fd_set大小固定为1024，而pollfd在内核中采用链表，理论上无上限
- 监听FD越多，每次遍历消耗时间也越久，性能反而会下降
只解决了id的上限问题，性能反而下降了
#### epoll
![[Pasted image 20251004090926.png]]
epoll模式中如何解决这些问题的？
- 基于epoll实例中的红黑树保存要监听的FD，理论上无上限，而且增删改查效率都非常高
- 每个FD只需要执行一次epoll_ctl添加到红黑树，以后每次epol_wait无需传递任何参数，无需重复拷贝FD到内核空间
- 利用ep_poll_callback机制来监听FD状态，无需遍历所有FD，因此性能不会随监听的FD数量增多而下降
### 事件通知机制
当FD有数据可读时，调用epoll_wait（或者select、poll）可以得到通知。但是事件通知的模式有两种：
- LevelTriggered：简称LT，也叫做水平触发。只要某个FD中有数据可读，每次调用epoll_wait都会得到通知。
- EdgeTriggered：简称ET，也叫做边沿触发。只有在某个FD有状态变化时，调用epoll_wait才会被通知。

LT在数据从内核拷贝到用户空间后，重新添加回就绪队列；ET拷贝完了，直接干掉。
![[Pasted image 20251004092333.png]]
结论：
- ET模式避免了LT模式可能出现的惊群现象
- ET模式最好结合非阻塞IO读取FD数据，相比LT会复杂一些
### web服务流程
![[Pasted image 20251004093040.png]]
## 信号驱动IO
信号驱动IO是与内核建立SIGIO的信号关联并设置回调，当内核有FD就绪时，会发出SIGIO信号通知用户，期间用户应用可以执行其它业务，无需阻塞等待。
![[Pasted image 20251004093504.png]]
阶段一：
1. 用户进程调用sigaction，注册信号处理函数
2. 内核返回成功，开始监听FD
3. 用户进程不阻塞等待，可以执行其它业务
4. 当内核数据就绪后，回调用户进程的SIGIO处理函数

阶段二：
1. 收到SIGIO回调信号
2. 调用recvfrom，读取
3. 内核将数据拷贝到用户空间
4. 用户进程处理数据

不大量使用的原因：
	当有大量IO操作时，信号较多，SIGIO处理函数不能及时处理可能导致信号队列溢出，而且内核空间与用户空间的频繁信号交互性能也较低。
## 异步IO
异步IO的整个过程都是非阻塞的，用户进程调用完异步API后就可以去做其它事情，内核等待数据就绪并拷贝到用户空间后才会递交信号，通知用户进程。
![[Pasted image 20251004093742.png]]
阶段一：
1. 用户进程调用aio_read，创建信号回调函数
2. 内核等待数据就绪
3. 用户进程无需阻塞，可以做任何事情

阶段二：
1. 内核数据就绪
2. 内核数据拷贝到用户缓冲区
3. 拷贝完成，内核递交信号触发aio_read中的回调函数
4. 用户进程处理数据
可以看到，异步IO模型中，用户进程在两个阶段都是非阻塞状态。

不大量使用的原因：
	用户程序可能一直给内核任务，内核任务累积过量导致系统因占用内存过多而崩溃。（使用要进行限流 == 高复杂性）
### 同步和异步
IO操作是同步还是异步，关键看数据在内核空间与用户空间的拷贝过程（数据读写的IO操作），也就是阶段二是同步还是异步：
![[Pasted image 20251004094259.png]]
## Redis网络模型
Redis到底是单线程还是多线程？
- 如果仅仅聊Redis的核心业务部分（命令处理），答案是单线程
- 如果是聊整个Redis，那么答案就是多线程

在Redis版本迭代过程中，在两个重要的时间节点上引入了多线程的支持：
- Redis v4.0：引入多线程异步处理一些耗时较旧的任务（特别是后台进程），例如异步删除命令unlink
- Redis v6.0：在核心网络模型中引入 多线程，进一步提高对于多核CPU的利用率

1. **为什么Redis要选择单线程？**
- 抛开持久化不谈，Redis是纯内存操作，执行速度非常快，它的性能瓶颈是网络延迟而不是执行速度，因此多线程并不会带来巨大的性能提升。
- 多线程会导致过多的上下文切换，带来不必要的开销
- 引入多线程会面临线程安全问题，必然要引入线程锁这样的安全手段，实现复杂度增高，而且性能也会大打折扣
**现在依然是单线程，也就在网络处理这块加了多线程**
### Redis单线程及多线程网络模型变更
Redis通过IO多路复用来提高网络性能，并且支持各种不同的多路复用实现，并且将这些实现进行封装， 提供了统一的高性能事件库API库 AE：
![[Pasted image 20251004103312.png]]
后面的是源码解读。
#### 网络模型结构图
![[Pasted image 20251004105508.png]]
当server Socket创建完了，会将fd注册到aeEventLoop上。
当server Socket 有客户端连接上了，就会触发其读事件。将tcpAccepthandler处理器处理server Socket读事件得到客户端Socket的fd并将其注册到aeEventLoop上。

若client Socket可读，会调用readQueryFromClient。
readQueryFromClient会给每一个client Socket封装一个对应的client，并将请求读一下放到queryBuf中，后解析数据为Redis命令，后将解析得到的Redis命令写到 buf或reply缓存区 （buf写不下写到reply链表中）。
写到 buf或reply 缓存区后，将其放入`server. clients_pending_write`队列中，然后遍历队列中的client，监听FD写事件绑定写处理器sendReplyToClient。

sendReplyToClient就把缓冲区的数据取出来，一个个地写入Client的Socket中，客户端就拿到数据了。
#### 多线程的使用
可以将aeEventLoop、before sleep，aeApiPoll抽象其功能理解为： IO多路复用+事件派发。

Redis 6.0版本中引入了多线程，目的是为了提高IO读写效率。因此在解析客户端命令、写响应结果时采用了多线程。核心的命令执行、IO多路复用模块依然是由主线程执行。
![[Pasted image 20251004105709.png]]
### Redis通信协议
#### RESP协议
Redis是一个CS架构的软件，通信一般分两步（不包括pipeline和PubSub）：
1. 客户端（client）向服务端（server）发送一条命令
2. 服务端解析并执行命令，返回响应结果给客户端
因此客户端发送命令的格式、服务端响应结果的格式必须有一个规范，这个规范就是通信协议。

而在Redis中采用的是RESP（Redis Serialization Protocol）协议：
- Redis 1.2版本引入了RESP协议
- Redis 2.0版本中成为与Redis服务端通信的标准，称为RESP2
- Redis 6.0版本中，从RESP2升级到了RESP3协议，增加了更多数据类型并且支持6.0的新特性--客户端缓存
但目前，默认使用的依然是RESP2协议，也是我们要学习的协议版本（以下简称RESP）。
#### RESP协议-数据类型
在RESP中，通过首字节的字符来区分不同数据类型，常用的数据类型包括5种：
- 单行字符串：首字节是 **‘+’** ，后面跟上单行字符串，以CRLF（ **"\r\n"** ）结尾。例如返回"OK"： "+OK\r\n"
- 错误（Errors）：首字节是 ‘-’ ，与单行字符串格式一样，只是字符串是异常信息，例如："-Error message\r\n"
- 数值：首字节是 ‘:’ ，后面跟上数字格式的字符串，以CRLF结尾。例如：":10\r\n"
- 多行字符串：首字节是 **‘$’** ，表示二进制安全的字符串，最大支持512MB：
![[Pasted image 20251004112412.png]]
	如果大小为0，则代表空字符串："$0\r\n\r\n"
	如果大小为-1，则代表不存在："$-1\r\n"
- 数组：首字节是 ‘`*`’，后面跟上数组元素个数，再跟上元素，元素数据类型不限:
![[Pasted image 20251004112638.png]]
#### 有Java实现有机会实践看看
## Redis内存策略
Redis之所以性能强，最主要的原因就是基于内存存储。然而单节点的Redis其内存大小不宜过大，会影响持久化或主从同步性能。
当内存使用达到上限时，就无法存储更多数据了。为了解决这个问题，Redis提供了一些策略实现内存回收：
- 内存过期策略
- 内存淘汰策略
### 过期策略
#### DB结构
Redis本身是一个典型的key-value内存存储数据库，因此所有的key、value都保存在之前学习过的Dict结构中。
不过在其database结构体中，有两个Dict：一个用来记录key-value；另一个用来记录key-TTL。
```C
typedef struct redisDb {
    dict *dict;              /* 存放所有key及value的地方，也被称为keyspace*/
    dict *expires;          /* 存放每一个key及其对应的TTL存活时间，只包含设置了TTL的key*/
    dict *blocking_keys;        /* Keys with clients waiting for data (BLPOP)*/
    dict *ready_keys;           /* Blocked keys that received a PUSH */
    dict *watched_keys;         /* WATCHED keys for MULTI/EXEC CAS */
    int id;                     /* Database ID，0~15 */
    long long avg_ttl;          /* 记录平均TTL时长 */
    unsigned long expires_cursor; /* expire检查时在dict中抽样的索引位置. */
    list *defrag_later;         /* 等待碎片整理的key列表. */
} redisDb;
```
![[Pasted image 20251004130856.png]]
#### 问题：
有两个问题需要我们思考：
1. Redis是如何知道一个key是否过期呢？
利用两个Dict分别记录key-value对及key-ttl对
2. 是不是TTL到期就立即删除了呢？
惰性删除
周期删除
要立即删除需要设置一个定时器，到期就干掉，但大量的计时器会耗费大量的资源。
#### 惰性删除：
**惰性删除**：顾明思议并不是在TTL到期后就立刻删除，而是在访问一个key的时候，检查该key的存活时间，如果已经过期才执行删除。
![[Pasted image 20251004131302.png]]
是在访问它的时候检查，然后删除。
注意：
	如果有一个key过期很长时间了，一直没有人来访问它，它并不会被删除。
#### 周期删除
周期删除：顾明思议是通过一个定时任务，**周期性的抽样部分过期的key**，然后执行删除。执行周期有两种：
- Redis服务初始化函数initServer()中设置定时任务，按照server.hz的频率来执行过期key清理，模式为SLOW (低频，大量清理)
- Redis的每个事件循环前会调用beforeSleep()函数，执行过期key清理，模式为FAST。（高频，大量清理）
![[Pasted image 20251004131801.png]]
![[Pasted image 20251004132112.png]]
SLOW模式规则：
1. 执行频率受server.hz影响，默认为10，即每秒执行10次，每个执行周期100ms。
2. 执行清理耗时不超过一次执行周期的25%.默认slow模式耗时不超过25ms
3. 逐个遍历db，逐个遍历db中的bucket，抽取20个key判断是否过期
4. 如果没达到时间上限（25ms）并且过期key比例大于10%，再进行一次抽样，否则结束

FAST模式规则（过期key比例小于10%不执行 ）：
1. 执行频率受beforeSleep()调用频率影响，但两次FAST模式间隔不低于2ms
2. 执行清理耗时不超过1ms
3. 逐个遍历db，逐个遍历db中的bucket，抽取20个key判断是否过期
4. 如果没达到时间上限（1ms）并且过期key比例大于10%，再进行一次抽样，否则结束
#### 小结
RedisKey的TTL记录方式：
- 在RedisDB中通过一个Dict记录每个Key的TTL时间
过期key的删除策略：
- 惰性清理：每次查找key时判断是否过期，如果过期则删除
- 定期清理：定期抽样部分key，判断是否过期，如果过期则删除。
定期清理的两种模式：
- SLOW模式执行频率默认为10，每次不超过25ms
- FAST模式执行频率不固定，但两次间隔不低于2ms，每次耗时不超过1ms
### 过期策略的问题&淘汰策略的目的
如果key都没有过期，而Redis内存已经达到上限，就会考虑淘汰策略。
### 淘汰策略
内存淘汰：就是当Redis内存使用达到设置的上限时，主动挑选部分key删除以释放更多内存的流程。Redis会在处理客户端命令的方法  `processCommand()`  中尝试做内存淘汰：
```C
int processCommand(client *c) {
    // 如果服务器设置了server.maxmemory属性，并且并未有执行lua脚本
    if (server.maxmemory && !server.lua_timedout) {
        // 尝试进行内存淘汰performEvictions
        int out_of_memory = (performEvictions() == EVICT_FAIL);
        // ...
        if (out_of_memory && reject_cmd_on_oom) {
            rejectCommand(c, shared.oomerr);
            return C_OK;
        }
        // ....
    }
}
```
在任何命令执行之前去做内存的检查，尝试去淘汰一部分内存。
#### Redis支持8种不同策略来选择要删除的key：

- `noeviction`： 不淘汰任何key，但是内存满时不允许写入新数据，默认就是这种策略。
- `volatile-ttl`： 对设置了TTL的key，比较key的剩余TTL值，TTL越小越先被淘汰
- `allkeys-random`：对全体key ，随机进行淘汰。也就是直接从db->dict中随机挑选
- `volatile-random`：对设置了TTL的key ，随机进行淘汰。也就是从db->expires中随机挑选。
- `allkeys-lru`： 对全体key，基于LRU算法进行淘汰
- `volatile-lru`： 对设置了TTL的key，基于LRU算法进行淘汰
- `allkeys-lfu`： 对全体key，基于LFU算法进行淘汰
- `volatile-lfu`： 对设置了TTL的key，基于LFI算法进行淘汰
比较容易混淆的有两个：
- `LRU（Least Recently Used)`，最少最近使用。用**当前时间减去最后一次访问时间**，这个值越大则淘汰优先级越高。
- `LFU（Least Frequently Used)`，最少频率使用。会统计每个key的访问频率，值越小淘汰优先级越高。（会用255-统计的访问频率，值越大越容易被淘汰）
```C
typedef struct redisObject {
    unsigned type:4;        // 对象类型
    unsigned encoding:4;    // 编码方式
    
    unsigned lru:LRU_BITS;  // LRU：以秒为单位记录最近一次访问时间，长度24bit
    // LFU：高16位以分钟为单位记录最近一次访问时间，低8位记录逻辑访问次数
    
    int refcount;           // 引用计数，计数为0则可以回收
    void *ptr;              // 数据指针，指向真实数据
} robj;
```
LFU的访问次数之所以叫做逻辑访问次数，是因为并不是每次key被访问都计数，而是通过运算：
1. 生成0~1之间的随机数R
2. 计算 `1/(旧次数 * lfu_log_factor + 1)`，记录为P
3. 如果 R < P ，则计数器 + 1，且最大不超过255
4. 访问次数会随时间衰减，距离上一次访问时间每隔 `lfu_decay_time` 分钟（默认1），计数器 -1
如果访问频率相同的情况下就必须用访问时间做判断是否需要删除.
#### 淘汰策略内存结构图
![[Pasted image 20251004134429.png]]
Tips知识回顾：
	在mysql中淘汰脏页的LRU算法是使用一个链表，访问过的页就往头放，这样尾部的脏页就是最久没被访问过的
	为啥要升序存？ ==> 这是一个链表, 升序比较链表头就好