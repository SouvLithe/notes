# IO流
File类，只能对文件本身进行操作，不能读写文件里面存储的内容。
IO流，用于读写文件中的数据（可以写文件，或者网络中的数据，也可用于将文件从内存写入存储器中）
两种分类方式：
![[Pasted image 20250812101945.png]]
纯文本文件：
	用Windows自带的记事本打开能读懂的文件，就是纯文本文件（txt、md、xml、lrc）
输入输出流：
![[Pasted image 20250812102248.png]]
## 字节流
![[Pasted image 20250812103852.png]]
### FileOutPutStream：
![[Pasted image 20250812111011.png]]
写数据：
![[Pasted image 20250812111209.png]]
输入中文可以这样
	`byte[] b = "我是中文".getBytes();` 
	`fos.write(b);`
#### 两个问题
换行写：
	再写一个换行符就行
		windows：  \r\n   ==>回车（光标到一行开头）、换行（光标到下一行）
		Linux：  \n                           Mac：   \r
java可以直接用  \r  或  \n  也可以实现换行，因为Java底层会自动补全，但建议不要省略，写全。

续写：
	创建对象时，在路径对象后面加一个True就行，默认是false不支持续写。

#### 小结：
![[Pasted image 20250812113813.png]]
使用的绝大多数IO流都要关闭通道。

### FileIntPutStream：
 格式与输出流一致
主要区别：
	write - > read
![[Pasted image 20250812114739.png]]
#### 循环读取：
![[Pasted image 20250812115129.png]]
while中的第三方变量必须定义，否则把它当作值赋给sout中时，得到的效果实际上是 间隔读取  间隔一个字符。


### 文件拷贝
用IO流拷贝文件过大时，因一次只读写一个字节，速度会变慢。
![[Pasted image 20250812172134.png]]
解决方法重载read方法，一次读多个字节：
![[Pasted image 20250812172259.png]]
一般是1024的整数倍，可以是5MB~10MB（图中蓝色是5MB）。返回值是本次读取的字节数 
#### 小细节
在装填数组的时候，会尽可能将数组装满。所以不是数组长度的倍数的话会发生下面情况。
![[Pasted image 20250812173356.png]]
在new string的时候，方法里面进行了强转。
去掉残留数据的方法：
在String的构造方法里面除了可以将字节数组变成字符串以外，还可以将字节数组的一部分变成字符串。即将  new String里面的参数从 bytes --> bytes，0，len  （数组，从零索引开始，将len个元素装满）
### 捕获异常
![[Pasted image 20250812175206.png]]
这种情况下，可能到write时出异常并抛出这样会导致，释放资源的代码执行不到。
处理方法：
在try外面给fos初始化： `FileOnputStream fis = null；` 
下图的FileOnputStream要省略。
![[Pasted image 20250812175351.png]]
JVM退出例子：在try中写 `System.exit(0);`
了解：
![[Pasted image 20250812180449.png]]
阅读性很差。

## 字符集
### ASCII、GBK
![[Pasted image 20250812181248.png]]
GB2312字符集、BIG5字符集 => JBK字符集=ANSI
![[Pasted image 20250812182105.png]]
汉字是两个字节，前面的成为高位字节，后面的成为低位字节。
注意：
	这里的高位第一位并不是符号位，是可以存储数据的，每个高位字节都可以是0x80到0xFF之间的值，即二进制的10000000到11111111
### Unicode
遵循[[字符编码]]特性。
![[Pasted image 20250812182823.png]]
![[Pasted image 20250812183002.png]]

### 为什么存在乱码
#### 原因1：
![[Pasted image 20250812183842.png]]
#### 原因2：
![[Pasted image 20250812184100.png]]
如何不产生乱码？
1. 不要用字节流读取文本文件
2. 编码解码时使用同一个码表，同一个编码方式

### java中的相关方法：
![[Pasted image 20250812184418.png]]
## 字符流
关系图：
![[Pasted image 20250816162553.png]]
构造方法：
![[Pasted image 20250816162702.png]]
成员方法：
![[Pasted image 20250816162718.png]]
书写规范：和字节输出流一样
![[Pasted image 20250816163132.png]]

### 底层原理
#### 输入流：
字节流没有缓冲区。
![[Pasted image 20250816164333.png]]
 如果文件字节数大于8192字节，缓冲区会从数组开头依次覆盖其中数据并不断循环。
	 8193 --> 0 , 8194 --> 1.....
问题（写过这个StreamIO -->demo3）
![[Pasted image 20250816170756.png]]
#### 输出流
![[Pasted image 20250816172821.png]]
数据保存到目的地的三种情况：
	1. 缓冲区装满了
	2. 手动刷新 flush                 3. 释放资源  close
![[Pasted image 20250816173036.png]]
flush调用后，会把缓冲区的数据刷新到文件。
close断开之前会检查缓冲区中有没有数据，如果有会写入文件。

## 高级流
![[Pasted image 20250816174735.png]]
高级流：对基本流进行封装，额外添加一些新的功能。

### 缓冲流
![[Pasted image 20250816175106.png]]
因为：`char` 在 Java 中占 2 字节，而 `byte` 占 1 字节
所以：	字节缓冲流的缓冲区的数据类型是  byte类型  的长度是  8k
	字符缓冲流的缓冲区的数据类型是  char类型  的长度是  16k
#### 字节缓冲流
包装：**把原始流作为参数传进去，构造一个新的带缓冲功能的流对象**
![[Pasted image 20250816175201.png]]
缓冲流的close方法在底层可以直接关闭基本流。

提高效率的原理：
![[Pasted image 20250816181903.png]]
图中的两个缓冲区不是一个东西。如果是一个个数字就int b，数组就int[ ] byte.
提高效率指的是：基本流和缓冲输入流的时间。即硬盘和内存的交互的时间

#### 字符缓冲流
它的基本流已经附带缓冲区了，因此效率提升不是很明显。
构造方法：
![[Pasted image 20250816182826.png]]
与字节流类似。
特有方法：
![[Pasted image 20250816183011.png]]

### 转换流
属于字符流，本身也是高级流。是字符流和字节流之间的桥梁。
![[Pasted image 20250821150634.png]]
输出流OutPutStreamWriter的顺序与输入流相反。
作用：
	指定字符集续写（jdk11后被Charset.forName淘汰了）
	字节流想要使用字符流中的方法
### 序列化流
![[Pasted image 20250821153045.png]]
![[Pasted image 20250821153258.png]]
![[Pasted image 20250821153457.png]]
Serializable 接口里面没有抽象方法，是标记型接口。  
只要实现了该接口，就表示当前类可以被序列化。
#### 反序列化流
![[Pasted image 20250821153723.png]]
#### 两流小细节
继承了Serializable接口，java会根据类中的所有内容计算出一个long类型的序列号（版本号）。
创建对象并写入文件是也会把 版本号 写入文件。
若此时修改这个类中的代码，java会重新计算出一个版本号。
这时读取标记着原版本号的文件时，会报错。
即：
![[Pasted image 20250821154716.png]]

解决办法1：固定版本号，若先定义出来这个版本号，java就不会额外计算了。
格式：
![[Pasted image 20250821155036.png]]
这时版本号就是1，serialVersionUID这个名字要不变。
办法2：可能版本不同换了位置 不显示的可以在File > Settings > Editor > Inspections > JVM languages 下找到并勾选应用就可以了
推荐用方法2.

transient：  瞬态关键字
作用：不会把当前属性序列化到本地文件当中。
#### 小结
![[Pasted image 20250821170131.png]]

### 打印流
![[Pasted image 20250821170340.png]]
只有输出流。
分类：打印流一般指 PrintStream、PrintWriter 两个类。  
特点 1：只操作文件目的地，不操作数据源。  
特点 2：特有写出方法可实现数据原样写出。
特点 3： 打印流特有写出方法可实现 **自动刷新 + 自动换行**。  
	打印一次数据 = 写出 + 换行 + 刷新。
#### 字节打印流
![[Pasted image 20250821170739.png]]
![[Pasted image 20250821170843.png]]
自动刷新是让每次调用print或者println写入的时候刷新OutputStream的缓冲区
#### 字符打印流
字符打印流底层有缓冲区，想要自动刷新需要开启。
成员方法和构造方法和字节打印流一样。
#### sout
JVM启动的时候会调用本地方法，即用更底层的语言实现的方法，直接修改跑到JVM方法区去System类静态字段中的值
```java
// 获取打印流对象；JVM 启动时创建，默认指向控制台  
// 这是系统标准输出流，不能关闭，全局唯一  
PrintStream ps = System.out;

// 调用打印流的 println 方法  
// 写出数据 + 自动换行 + 自动刷新  
ps.println("123");
// ps.close();        // 不要关闭

ps.println("你好你好");

System.out.println("456");
```
#### 小结
![[Pasted image 20250821172458.png]]
### 解压缩流和压缩流
![[Pasted image 20250821172644.png]]
#### 解压缩流
压缩包里的每一个文件/目录都用 ZipEntry 表示。  
解压的本质：按层级把每个 ZipEntry 拷贝到本地另一个文件夹中

Java 自带的 `java.util.zip` 包只能直接读写 **ZIP 格式**；如果要处理 **GZIP、JAR、7Z、RAR、TAR** 等格式，需要额外引入第三方库，例如：
- GZIP：`java.util.zip.GZIPInputStream / GZIPOutputStream`
- TAR：Apache Commons Compress
- 7Z / RAR：SevenZipJBinding、junrar 等
所以 Java 并不仅限于 ZIP，只是 ZIP 是标准库开箱即用的格式。
![[Pasted image 20250821174139.png]]
#### 压缩流
![[Pasted image 20250821174216.png]]
压缩文件：
![[Pasted image 20250821174631.png]]
压缩文件夹：



这一点有些看不懂。


### 常见工具包
这两个包主要讲的是IO相关的工具类。
#### Commons-io
![[Pasted image 20250821211159.png]]
**Commons-io使用步骤**
1. 在项目中创建一个文件夹：lib
2. 将jar包复制粘贴到lib文件夹
3. 右键点击jar包，选择 **Add as Library** → 点击 **OK**
4. 在类中导包使用
常见方法：
![[Pasted image 20250821211405.png]]
![[Pasted image 20250821211439.png]]
导包：
![[Pasted image 20250821211844.png]]
maven可能更好用一点。
#### Hutool
![[Pasted image 20250821212413.png]]
IO工具类：
![[Pasted image 20250821212536.png]]

# 多线程&JUC
应用软件中互相独立，包含在进程中，可以同时运行的功能——有了多线程，我们就可以让程序同时做多件事情。
之前写的代码都是单线程。

应用场景：
只要你想让多个事情同时运行就需要用到多线程  
比如：软件中的耗时操作、所有的聊天软件、所有的服务器
## 并发和并行
![[Pasted image 20250821224507.png]]
## 多线程的实现方式
![[Pasted image 20250821231049.png]]
### 1.继承 Thread 类的方式进行实现  
1. 自己定义一个类继承 Thread
2. 重写 run 方法（在run方法中书写线程要执行的代码）
3. 创建子类的对象，并启动线程
因为是继承关系，线程对象.setName（“name”）方法给线程起名字<在调用类中>
并在run方法中调用getName方法<在继承类中>，用于区分线程。
### 2.实现 Runnable 接口的方式进行实现  
1. 自己定义一个类实现 Runnable 接口
2. 重写里面的 run 方法
3. 创建自己的类的对象
4. 创建一个 Thread 类的对象，并开启线程
在<接口类>中，调用Thread.currentThread().getName()。
调用类和 1 一样
### 3.利用 Callable 接口和 Future 接口方式实现
1. 创建一个类 MyCallable 实现 Callable 接口
2. 重写 call（是有返回值的，表示多线程运行的结果）
3. 创建 MyCallable 的对象（表示多线程要执行的任务）
4. 创建 FutureTask 的对象（作用：管理多线程运行的结果）
5. 创建 Thread 类的对象，并启动（表示线程）
**特点：可以获取到多线程运行的结果。**

## 常见的成员方法
![[Pasted image 20250821231309.png]]
java中线程的优先级最小为1，最大为10，默认是5.
优先级越大，抢占到CPU的概率是越高的。

setName细节：
1.如果没有给线程设置名字，线程也是有默认的名字的  
格式：Thread-X（X 序号，从 0 开始）
2.如果要给线程设置名字，可以用set方法进行设置，也可以用构造方法设置。

currentThread细节：
`Thread.currentThread()` 返回的是“当前正在执行这段代码的线程”。即使上面（或别处）已经启动了两个线程，只要 `currentThread()` 这句代码本身是在主线程里执行的，它就会返回主线程并输出 main；如果你在子线程的 run 方法里调用 `currentThread()`，则会输出对应子线程的名字。
因此，输出什么完全取决于你把 `currentThread()` 写在哪个线程的上下文中。
```
当 JVM 虚拟机启动之后，会自动地启动多条线程。  
其中有一条线程就叫做 main 线程。  
它的作用就是去调用 main 方法，并执行里面的代码。  
在以前，我们写的所有代码其实都是运行在 main 线程当中。
```

sleep细节：
1. 哪条线程执行到这个方法，哪条线程就会在这里停留对应的时间。
2. 方法的参数表示睡眠的时间，单位毫秒（1 秒 = 1000 毫秒）。
3. 当时间到了之后，线程会自动醒来，继续执行下面的其他代码。
### 线程优先级
java中采取了抢占式调度
java中线程的优先级最小为1，最大为10，默认是5.
下图在本章首图：
![[Pasted image 20250821232938.png]]
### 守护线程
线程对象.方法（true）；  将该方法设为u守护线程
当其它的非守护线程执行完毕后，守护线程会陆续结束。
应用场景：聊天 和 传输文件。当退出聊天框，传输文件也就没有存在的必要了
### 礼让线程&插入线程
`Thread.yield();` 表示出让当前CPU的执行权，是写在run方法中的。
作用：让线程执行尽可能均匀一点（不常用）
插入线程（不常用）：
![[Pasted image 20250822103403.png]]
这段的代码的意思：将  土豆线程  插入到main线程之前，执行完土豆再执行main
## 线程的生命周期
![[Pasted image 20250822103827.png]]

## 线程安全问题
![[Pasted image 20250822111120.png]]
想让多个线程公用一个最大值（如，多窗口总共卖只n张票），要将n设置为static静态变量，否则会出现多线程每个线程都卖n张票。
但同时也会出现多个线程用同一张票，和超出总票数n的情况。

出现上面的情况是因为：线程执行时，具有随机性
- 多个线程读到相同的 `ticket` 值（导致卖同一张票）
- 多个线程执行 `ticket++` 后，值超过 100（导致超卖）
解决方法：将操作共享数据的代码锁起来。
### 同步代码块
![[Pasted image 20250822111743.png]]
锁对象一定要是唯一的，在对象前加一个static静态关键字即可
将上面的  if...else...  作为操作共享数据的代码。
不写再while循环的外面，是因为会一个窗口一直卖。
一般锁对象会用  当前类名的.class  字节码文件对象
### 同步方法：
![[Pasted image 20250822144228.png]]
如果是单线程则考虑使用StringBuilder，多线程则使用StringBuffer（因为其中用到了synchronized关键字）。
### Lock锁
![[Pasted image 20250822145455.png]]
这种写法中，最好把unlock方法写到try...catch...finally中的finally方法中，利用无论如何finally都会被执行的特性。
否则会出现一个线程从头到尾执行，而其他线程一直在等待调度的情况
### 死锁
是一种错误写法，不要出现。
本质就是：嵌套锁
![[Pasted image 20250822150844.png]]
## 生产者和消费者（等待唤醒机制）
生产者消费者模式是一种多线程协作的模式。
消费者等待、生产者等待。
![[Pasted image 20250822151603.png]]
常见方法：
![[Pasted image 20250822151630.png]]
控制线程的数量：boolean --> int由只能控制两种转为了多种。
### 用阻塞队列的方式实现
![[Pasted image 20250822162915.png]]
继承结构：
![[Pasted image 20250822163022.png]]
生产者和消费者必须使用一个阻塞队列。
代码多看看
## 线程的六种状态
![[Pasted image 20250822164104.png]]
java实际上是没有定义  运行  状态的，当线程抢到CPU的执行权的时候，这是JVM会把执行权交给操作系统管理了。
![[Pasted image 20250822164329.png]]
## 线程池
以前写多线程的弊端：
![[Pasted image 20250822164555.png]]
为了优化这个 -->线程池
核心原理：
![[Pasted image 20250822164832.png]]
代码实现：
```
1. 创建线程池
2. 提交任务
3. 所有的任务全部执行完毕，关闭线程池
```
![[Pasted image 20250822165036.png]]
```java
// 1. 获取线程池对象
ExecutorService pool1 = Executors.newCachedThreadPool();
// 2. 提交任务
pool1.submit(new MyRunnable());
// 3. 销毁线程池
pool1.shutdown();
```
### 自定义线程池
元素：
![[Pasted image 20250822170334.png]]
只有当 核心线程 和 队伍长度 占满时，临时线程才会被创建和使用。
先提交的任务不一定限制性。
![[Pasted image 20250822170823.png]]
当核心线程、队伍和临时线程都满时，就会触发java的拒绝策略。
![[Pasted image 20250822171104.png]]
![[Pasted image 20250822171529.png]]
### 最大并行数
最大并行数跟CPU的型号有关系。
例：4核8线程 的最大并行数为8
![[Pasted image 20250822172604.png]]直接用这个端返回的就是最大并行数。
### 线程池多大合适
![[Pasted image 20250822171636.png]]
CPU密集型运算：项目中计算较多。
I/O密集型运算：（大多数）项目中读取本地文件、数据库较多。
用工具（如thread dump）测出  CPU计算时间  和  等待时间。

### 开发使用较少but面试喜欢问
![[Pasted image 20250822173732.png]]
# 网络编程
在网络编程通信协议下，不同计算机上运行的程序，进行的数据传输。
![[Pasted image 20250822205949.png]]
B/S优缺点：
- 不需要开发客户端，只需要页面 + 服务端
- 用户不需要下载，打开浏览器就能使用
- 如果应用过大，用户体验受到影响
C/S优缺点：
- 画面可以做得非常精美，用户体验好
- 需要开发客户端，也需要开发服务端
- 用户需要下载和更新的时候太麻烦
## 网络编程三要素
- IP：设备在网络中的地址，是唯一的标识。
![[Pasted image 20250822211619.png]]
- 端口号：应用程序在设备中唯一的标识。
![[Pasted image 20250822214913.png]]
- 协议：数据在网络中传输的规则，常见的协议有 UDP、TCP、HTTP、HTTPS、FTP。

## InetAddress
java中用来表示IP的类。
![[Pasted image 20250822214827.png]]
## UDP协议
### 发送数据
![[Pasted image 20250822215820.png]]
构建`DatagramSocket ds = new DatagramSocket();`的参数若为空，则发送方在所有可用端口号随机一个使用。
和接收数据代码共同使用可以进行交互。
先执行接收方，再执行发送方。
### 接收数据
![[Pasted image 20250822222853.png]]
发送端和接收端要彼此验证时，要先执行接收端是因为：接收端会卡在调用receive方法，减少漏接数据可能。
### UDP的三种通信方式
- 单播：以前的代码就是单播
- 组播
    - 组播地址：224.0.0.0 ~ 239.255.255.255
    - 其中 224.0.0.0 ~ 224.0.0.255 为预留的组播地址
- 广播
    - 广播地址：255.255.255.255
组播：将DatagramSocket对象改为MulticastSocket对象，
发送方在发送的时候要指定组播地址，接收方要将当前本机划分为组播地址中
其余和之前一样。
广播：只需要将单播发送方目标地址改为255.255.255.255即可。
## TCP协议
![[Pasted image 20250822225211.png]]
代码实现：
发送端和接收端要彼此验证时，要先执行接收端是因为：
	1.接收端会卡在调用accept方法，减少漏接数据可能。
	2.若是发送端先运行，他找不到接收端的IP
读写通过IO流完成。
其中流可以关也可以不关，因为流是在socket中的，socket关了流自然也关了
![[Pasted image 20250822225649.png]]
问题：当传输的数据是中文是会出现乱码。
### 中文乱码问题
因为IDEA使用的utf-8编码下，中文占三个字节，而读取的时候是一个一个字节读的。
解决方法：利用转换流，将is字节流转变为字符流
![[Pasted image 20250822230059.png]]
# 反射
这是给框架用的，Mybatis就是通过反射来个对象里面的属性赋值，从对象里面获取属性，映射到sql进行sql操作
![[Pasted image 20250822231803.png]]
获取的时候不是从java文件中获取出来的，而是从class文件中获取出来的。
## 反射作用
1. 获取一个类里面所有的信息，获取到了之后，再执行其他的业务逻辑  
2. 结合配置文件，动态的创建对象并调用方法
![[Pasted image 20250823121951.png]]
## 获取class对象的三种方式
Java中有一个叫Class类 用于 描述字节码文件
![[Pasted image 20250822232742.png]]
全类名=包名+类名。
选中类名右键 --> Copy/Paste Special --> Copy Reference --> Ctrl+v
粘贴的位置不同有时不会是全类名
![[Pasted image 20250822233434.png]]
![[Pasted image 20250822233458.png]]
## 获取方法
| 获取  | 获取class对象 |    构造方法     | 字段(成员变量) |  成员方法  |
| :-: | :-------: | :---------: | :------: | :----: |
| 相应类 |   class   | Constructor |  Field   | Method |
### 获取构造方法
![[Pasted image 20250823113853.png]]
当有多个构造方法时，获取单个构造方法是根据  方法中传递的参数类型  决定获取哪个构造方法。
private修饰的构造函数一般是工具类防止被实例化的，暴力构造也没有意义
`setAccessible（true）`方法是 暴力反射，临时取消private的权限修饰。
### 获取成员变量
![[Pasted image 20250823115242.png]]
### 获取成员方法
![[Pasted image 20250823115733.png]]
## 小结
![[Pasted image 20250823122140.png]]
# 动态代理
特点：无侵入式地给代码增加额外功能。
对象如果嫌身上干的事太多的话，可以通过代理来转移部分职责。
对象有什么方法想被代理，代理就一定要有对应的方法。
![[Pasted image 20250823133510.png]]
## 创建代理对象
![[Pasted image 20250823133812.png]]
## 问题
![[Pasted image 20250823143008.png]]
答：
```
1. java.lang.reflect.Proxy
2. 3 个：ClassLoader（加载代理类）、Class<?>[]（代理要实现的接口）、InvocationHandler（代理要做的事情）  
3. JVM 自动调用；接收 3 个参数：proxy（代理对象自身）、method（被调用的方法）、args（实参）
```
# 注解
### 1. 注解（Annotation）是什么
注解是Java中的一种特殊标记，它可以被附加到代码的某些元素上（如类、方法、字段等），用来提供额外的信息。注解本身不会改变代码的逻辑，但可以被其他工具或框架读取并执行特定的操作。

举个简单的例子，你可能见过`@Override`注解，它用来标记一个方法是覆盖父类的方法。这个注解不会改变方法的逻辑，但可以帮助编译器检查是否正确覆盖了父类的方法。
### 2. 自定义注解

Java允许你创建自己的注解，这叫做自定义注解。自定义注解可以用来提供特定的信息，比如标记一个方法是否需要进行日志记录，或者标记一个类是否是一个控制器。
#### 创建自定义注解的步骤
1. **定义注解**：使用`@interface`关键字定义一个注解。
2. **指定元注解**：使用元注解来定义注解的属性和行为。
### 3. 元注解
元注解是用于定义注解的注解。Java提供了几个标准的元注解，用来描述自定义注解的属性和行为。常见的元注解有：

- **`@Target`**：指定注解可以作用的目标类型（如类、方法、字段等）。
    
- **`@Retention`**：指定注解的保留策略（如仅在源代码中保留、编译时保留、运行时保留）。
    
- **`@Documented`**：指定注解是否被包含在JavaDoc文档中。
    
- **`@Inherited`**：指定注解是否可以被子类继承。
后面两个元注解没有参数。下面是前两个注解的参数

| @Target 参数值                 | 描述                       |
| --------------------------- | ------------------------ |
| ElementType.TYPE            | 注解可以应用于类、接口（包括注解类型）或枚举声明 |
| ElementType.FIELD           | 注解可以应用于字段声明              |
| ElementType.METHOD          | 注解可以应用于方法声明              |
| ElementType.PARAMETER       | 注解可以应用于方法的参数             |
| ElementType.CONSTRUCTOR     | 注解可以应用于构造函数声明            |
| ElementType.LOCAL_VARIABLE  | 注解可以应用于局部变量              |
| ElementType.ANNOTATION_TYPE | 注解可以应用于注解类型              |
| ElementType.PACKAGE         | 注解可以应用于包声明               |

| @Retention 参数值          | 描述                         |
| ----------------------- | -------------------------- |
| RetentionPolicy.SOURCE  | 注解仅保留在源代码中，在类加载时丢弃         |
| RetentionPolicy.CLASS   | 注解保留到类文件中，在类加载时丢弃，但可通过反射获取 |
| RetentionPolicy.RUNTIME | 注解保留在运行时，可通过反射获取           |
### 4. 一个简单的例子

假设你正在开发一个日志系统，你希望标记某些方法，以便在这些方法执行时记录日志。你可以创建一个自定义注解`@Loggable`，并使用元注解来定义它的行为。
#### 定义自定义注解`@Loggable`
```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

// 使用元注解定义注解
@Target(ElementType.METHOD) // 指定注解可以作用于方法
@Retention(RetentionPolicy.RUNTIME) // 指定注解在运行时保留
public @interface Loggable {
    String message() default "Method executed"; // 定义注解的属性，默认值为"Method executed"
}
```
#### 使用自定义注解
```java
public class UserService {

    @Loggable(message = "User created")
    public void createUser(String username) {
        System.out.println("Creating user: " + username);
    }

    @Loggable // 使用默认值
    public void deleteUser(String username) {
        System.out.println("Deleting user: " + username);
    }
}
```
#### 读取注解信息
你可以通过反射来读取注解的信息，并执行相应的操作。例如，你可以检查一个方法是否被`@Loggable`注解标记，并在方法执行时记录日志。
```java
import java.lang.reflect.Method;

public class LoggableProcessor {

    public static void process(Object object) throws Exception {
        Class<?> clazz = object.getClass();
        Method[] methods = clazz.getDeclaredMethods();

        for (Method method : methods) {
            if (method.isAnnotationPresent(Loggable.class)) {
                Loggable loggable = method.getAnnotation(Loggable.class);
                System.out.println("Log: " + loggable.message());
                method.invoke(object, "exampleUser"); // 调用方法
            }
        }
    }

    public static void main(String[] args) throws Exception {
        UserService userService = new UserService();
        process(userService);
    }
}
```
### 5. 总结

- **注解**：一种特殊的标记，用来提供额外的信息。
    
- **自定义注解**：自己创建的注解，用来满足特定的需求。
    
- **元注解**：用来定义注解的属性和行为的注解，如`@Target`、`@Retention`等。