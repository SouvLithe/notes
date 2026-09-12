# 进程与线程
进程:当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程。
(Java中启动了一个虚拟机，就相当于创建了一个进程)
线程:一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给 CPU 执行.
## 并行与并发
引用 Rob Pike 的一段描述： 
	并发（concurrent）是同一时间应对（dealing with）多件事情的能力 
	并行（parallel）是同一时间动手做（doing）多件事情的能力
## 同步与异步
同步: 需要等待结果返回，才能继续运行就是同步 
异步: 不需要等待结果返回，就能继续运行就是异步

结论 
- 比如在项目中，视频文件需要转换格式等操作比较费时，这时开一个新线程处理视频转换，避免阻塞主线程 
- tomcat 的异步 servlet 也是类似的目的，让用户线程处理耗时较长的操作，避免阻塞 tomcat 的工作线程 
- `ui` 程序中，开线程进行其他操作，避免阻塞 `ui` 线程
## 应用
单核cpu下，多线程并不能提高程序运行效率，它能够让多个线程轮流使用cpu。

多核cpu可以并行跑多个线程，能否提升效率分情况：
- 可以拆分的任务，并行执行可以提高效率。但并非所有任务都可以拆分。 【阿姆达尔定律】
- 如果任务的目的不同(任务2要等待任务1的结果)，拆分和效率的抉择就没有意义。

### JMH测试工具
- 基准测试工具，比较靠谱，它会执行程序预热，执行多次测试并平均
- JMH执行测试时，需要用maven把它打成jar包，运行这个jar包就行
- `cpu` 核数限制，有两种思路 
	1. 使用虚拟机，分配合适的核 
	2. 使用 msconfig，分配合适的核，需要重启比较麻烦

 - IO 操作不占用 cpu，只是当使用的是【阻塞 IO】（如拷贝文件），这时相当于线程虽然不用 cpu，但需要一 直等待 IO 结束，没能充分利用线程。所以才有后面的【非阻塞 IO】和【异步 IO】优化 
# java 线程
线程池的七大参数:核心线程数，最大线程数，任务队列，空闲线程死亡时间，时间单位，线程工厂，拒绝策略
## Thread 与 Runnable 的关系
- 方法1 是把线程和任务合并在了一起，方法2 是把线程和任务分开了 
- 用 Runnable 更容易与线程池等高级 API 配合 
- 用 Runnable 让任务类脱离了 Thread 继承体系，更灵活
## 查看线程进程的方法
windows:
```text
任务管理器可以查看进程和线程数，也可以用来杀死进程 
tasklist 查看进程  tasklist | findstr java
taskkill 杀死进程  taskkill /F /PID 【给进程id】
(/F【强制杀死】 /PID 【给进程id】)
```
`linux`:
```text
ps -fe 查看所有进程 
ps -fT -p <PID> 查看某个进程（PID）的所有线程 
kill 杀死进程 
top 按大写 H 切换是否显示线程 
top -H -p <PID> 查看某个进程（PID）的所有线程
```
Java:
```text
jps 命令查看所有 Java 进程 
jstack <PID> 查看某个 Java 进程（PID）的所有线程状态 
jconsole 来查看某个 Java 进程中线程的运行情况（图形界面）
```
`jconsole` 远程监控配置:
- 需要以如下方式运行你的 java 类
```text
java -Djava.rmi.server.hostname=`ip地址` -Dcom.sun.management.jmxremote - Dcom.sun.management.jmxremote.port=`连接端口` -Dcom.sun.management.jmxremote.ssl=是否安全连接 - Dcom.sun.management.jmxremote.authenticate=是否认证 java类
```
- 修改` /etc/hosts `文件将 127.0.0.1 映射至主机名 
- 如果要认证访问，还需要做如下步骤 
```text
复制 jmxremote.password 文件 
修改 jmxremote.password 和 jmxremote.access 文件的权限为 600 即文件所有者可读写 
连接时填入 controlRole（用户名），R&D（密码）
```
## 线程运行原理
Java Virtual Machine Stacks （Java 虚拟机栈）
### 栈帧图解：
![[Pasted image 20251018114609.png]]
### 线程上下文切换（Thread Context Switch）
由于以下一些原因导致 `cpu` 不再执行当前的线程，转而执行另一个线程的代码 
- 线程的 `cpu` 时间片用完 
- 垃圾回收 
- 有更高优先级的线程需要运行 
- 线程自己调用了 `sleep、yield、wait、join、park、synchronized、lock` 等方法

当 Context Switch 发生时，**需要由操作系统保存当前线程的状态**，并恢复另一个线程的状态，Java 中对应的概念 就是**程序计数器**（Program Counter Register,记录状态），它的作用是记住下一条 `jvm` 指令的执行地址，是线程私有的 
- 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等 
- Context Switch 频繁发生会影响性能 --> 这就是当糸统并发量上来的时候，性能下降的根本原因！！！
## 常见方法
只记录了一些不常见的，其它不懂的去查api文档
### sleep 与 yield 
sleep 
1. 调用 sleep 会让当前线程从 Running 进入 Timed Waiting 状态 **（阻塞）** 
2. 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 `InterruptedException `
3. 睡眠结束后的线程未必会立刻得到执行 
4. 建议用 `TimeUnit` 的 sleep 代替 Thread 的 sleep 来获得更好的可读性 

yield(让出&谦让) 
1. 调用 yield 会让当前线程从 Running 进入 Runnable **就绪状态**，然后调度执行其它线程 
2. 具体的实现依赖于操作系统的任务调度器
3. 如果没有其它额外线程，被yield掉的线程会直接执行，被sleep的线程则还是会等待。
#### sleep限制对 CPU 的使用
在没有利用 `cpu` 来计算时，不要让 while(true) 空转浪费 cpu，这时可以使用 yield 或 sleep 来让出 `cpu` 的使用权 给其他程序
```java
while(true) { 
	try { 
		Thread.sleep(50);      //不加地话，它会空转，不断占用cpu资源
	} catch (InterruptedException e) {
		 e.printStackTrace(); 
	 } 
}
```
- 可以用 wait 或 条件变量达到类似的效果 
- 不同的是，后两种都需要加锁，并且需要相应的唤醒操作，一般适用于要进行同步的场景 
- sleep 适用于无需锁同步的场景
### 线程优先级
- 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它 
- 如果 `cpu` 比较忙，那么优先级高的线程会获得更多的时间片，但 `cpu` 闲时，优先级几乎没作用
它们都不能真正地控制任务的调度，还是要听任务调度器的。
### join
```java
static int r = 0; 
public static void main(String[] args) throws InterruptedException { 
	test1(); 
} 
private static void test1() throws InterruptedException { 
	log.debug("开始"); 
	Thread t1 = new Thread(() -> { 
		log.debug("开始"); 
		sleep(1); 
		log.debug("结束"); 
		r = 10; 
	}); 
		t1.start(); 
		log.debug("结果为:{}", r); 
		log.debug("结束"); }
```
分析 
- 因为主线程和线程 t1 是并行执行的，t1 线程需要 1 秒之后才能算出 r=10 
- 而主线程一开始就要打印 r 的结果，所以只能打印出 r=0
用 join，加在 t1.start() 之后即可
### `interrput`
#### 打断 `sleep，wait，join` 的线程
如果被打断线程正在 `sleep，wait，join` 会导致被打断 的线程抛出 InterruptedException，并清除 打断标 记 ；
如果打断的正在运行的线程，则会设置 打断标记 ；
park 的线程被打断，也会设置 打断标记

interrupt只是通知线程中断，不是强中断,该种特性可以让被通知线程做一些善后工作，
被通知线程可以通过判断isInterrupted返回的中断标志，判断是否应该中断线程。
#### 两阶段终止模式
错误方法：
- 使用线程对象的 stop() 方法停止线程
    - stop 方法会真正杀死线程，如果这时线程锁住了共享资源，那么当它被杀死后就再也没有机会释放锁，其它线程将永远无法获取锁

- 使用 `System.exit(int)` 方法停止线程
    - 目的仅是停止一个线程，但这种做法会让整个程序都停止
### 打断 park 线程
`LockSupport.park();`它是锁的一个支持类，作用是让当前线程停下来，
特点是打断标记为真时会让park失效。
### 不推荐的方法
还有一些不推荐使用的方法，这些方法已过时，容易破坏同步代码块，造成线程死锁

| 方法名       | 功能说明       |
| --------- | ---------- |
| stop()    | 停止线程运行     |
| suspend() | 挂起（暂停）线程运行 |
| resume()  | 恢复线程运行     |
### 主线程与守护线程
默认情况下，Java 进程需要等待所有线程都运行结束，才会结束。
有一种特殊的线程叫做守护线程，只要其它  非守护线程运行结束了，即使守护线程的代码没有执行完，也会强制结束。
```java
// 设置该线程为守护线程 
t1.setDaemon(true); 
t1.start();
```
在线程start前只要调用setDaemon把它设置为true，它就是守护线程。
注意 
- 垃圾回收器线程就是一种守护线程 
- Tomcat 中的 `Acceptor` 和 `Poller` 线程都是守护线程，所以 Tomcat 接收到 shutdown 命令后，不会等 待它们处理完当前请求
## 线程状态
### 五种状态
这是从 **操作系统** 层面来描述的
![[Pasted image 20251018162043.png]]
- 【初始状态】仅是在语言层面创建了线程对象，还未与操作系统线程关联 
- 【可运行状态】（就绪状态）指该线程已经被创建（与操作系统线程关联），可以由 CPU 调度执行 
- 【运行状态】指获取了 CPU 时间片运行中的状态
	当 CPU 时间片用完，会从【运行状态】转换至【可运行状态】，会导致线程的上下文切换
- 【阻塞状态】
	- 如果调用了阻塞 API，如 BIO 读写文件，这时该线程实际不会用到 CPU，会导致线程上下文切换，进入 【阻塞状态】 
	- 等 BIO 操作完毕，会由操作系统唤醒阻塞的线程，转换至【可运行状态】 
	- 与【可运行状态】的区别是，对【阻塞状态】的线程来说只要它们一直不唤醒，调度器就一直不会考虑 调度它们
- 【终止状态】表示线程已经执行完毕，生命周期已经结束，不会再转换为其它状态
### 六种状态
笔记4-10上面有根据Java方法，分析这六种状态的讲解。
这是从 Java API 层面来描述的,根据 `Thread.State` 枚举，分为六种状态
![[Pasted image 20251018162523.png]]
- NEW 线程刚被创建，但是还没有调用 start() 方法 
- RUNNABLE 当调用了 start() 方法之后，注意，Java API 层面的 RUNNABLE 状态涵盖了 操作系统 层面的 【可运行状态】、【运行状态】和【阻塞状态】（由于 BIO 导致的线程阻塞，在 Java 里无法区分，仍然认为 是可运行） 
- BLOCKED ， WAITING ， TIMED_WAITING 都是 Java API 层面对【阻塞状态】的细分，后面会在状态转换一节 详述 
- TERMINATED 当线程代码运行结束
注意：
	WAITING 和TIMED_WAITING的区别就是一个有时限，一个没有时限

假设有线程t
#### 情况 1 NEW --> RUNNABLE
当调用 `t.start()` 方法时，由 NEW --> RUNNABLE

#### 情况 2 RUNNABLE <--> WAITING
t 线程用 synchronized(obj) 获取了对象锁后 
- 调用 `obj.wait()` 方法时，t 线程从 RUNNABLE --> WAITING 
- 调用 `obj.notify()` ， `obj.notifyAll()` ， `t.interrupt()` 时 
	竞争锁成功，t 线程从 WAITING --> RUNNABLE 
	竞争锁失败，t 线程从 WAITING --> BLOCKED
	
#### 情况 3 RUNNABLE <--> WAITING
当前线程调用 `t.join()` 方法时，当前线程从 RUNNABLE --> WAITING 
- 注意是当前线程在t 线程对象的监视器上等待 
t 线程运行结束，或调用了当前线程的 interrupt() 时，当前线程从 WAITING --> RUNNABLE

#### 情况 4 RUNNABLE <--> WAITING
- 当前线程调用 `LockSupport.park()` 方法会让当前线程从 RUNNABLE --> WAITING 
- 调用 `LockSupport.unpark(目标线程)` 或调用了线程 的 interrupt() ，会让目标线程从 WAITING --> RUNNABLE

#### 情况 5 RUNNABLE <--> TIMED_WAITING
t 线程用 synchronized(obj) 获取了对象锁后 
- 调用 `obj.wait(long n)` 方法时，t 线程从 RUNNABLE --> TIMED_WAITING 
- t 线程等待时间超过了 n 毫秒，或调用 `obj.notify() ， obj.notifyAll() ， t.interrupt()` 时 
	竞争锁成功，t 线程从 TIMED_WAITING --> RUNNABLE 
	竞争锁失败，t 线程从 TIMED_WAITING --> BLOCKED

#### 情况 6 RUNNABLE <--> TIMED_WAITING
当前线程调用 `t.join(long n)` 方法时，当前线程从 RUNNABLE --> TIMED_WAITING 
- 注意是当前线程在t 线程对象的监视器上等待 
当前线程等待时间超过了 n 毫秒，或t 线程运行结束，或调用了当前线程的 interrupt() 时，当前线程从 TIMED_WAITING --> RUNNABLE

#### 情况 7 RUNNABLE <--> TIMED_WAITING
- 当前线程调用 `Thread.sleep(long n)` ，当前线程从 RUNNABLE --> TIMED_WAITING 
- 当前线程等待时间超过了 n 毫秒，当前线程从 TIMED_WAITING --> RUNNABLE

#### 情况 8 RUNNABLE <--> TIMED_WAITING
- 当前线程调用 `LockSupport.parkNanos(long nanos)` 或 `LockSupport.parkUntil(long millis) `时，当前线 程从 RUNNABLE --> TIMED_WAITING 
- 调用 `LockSupport.unpark`(目标线程) 或调用了线程 的 interrupt() ，或是等待超时，会让目标线程从 TIMED_WAITING--> RUNNABLE

#### 情况 9 RUNNABLE <--> BLOCKED
- t 线程用 synchronized(obj) 获取了对象锁时如果竞争失败，从 RUNNABLE --> BLOCKED 
- 持 obj 锁线程的同步代码块执行完毕，会唤醒该对象上所有 BLOCKED 的线程重新竞争，如果其中 t 线程竞争 成功，从 BLOCKED --> RUNNABLE ，其它失败的线程仍然 BLOCKED

#### 情况 10 RUNNABLE <--> TERMINATED
当前线程所有代码运行完毕，进入 TERMINATED

## 小结
- 线程创建 
- 线程重要 api，如 `start，run，sleep，join，interrupt` 等 
- 线程状态

- 应用方面
	异步调用：主线程执行期间，其它线程异步执行耗时操作 
	提高效率：并行计算，缩短运算时间 
	同步等待：join 
	统筹规划：合理使用线程，得到最优效果
- 原理方面 线程运行流程：
	栈、栈帧、上下文切换、程序计数器 
	Thread 两种创建方式 的源码 
- 模式方面 
	终止模式之两阶段终止
# 共享模型管程（Monitor悲观锁）
Java的共享数据问题是怎么出现的呢？
从JVM字节码指令而言：
![[Pasted image 20251018181110.png]]
而 Java 的内存模型如下，完成静态变量的自增，自减需要在主存和工作内存（线程）中进行数据交换： 
![[Pasted image 20251018181350.png]]
如果是单线程串行执行不会出现  **指令交错问题**  问题。
## 临界区 Critical Section
一个程序运行多个线程本身是没有问题的, 
问题出在多个线程访问  **共享资源** 
- 多个线程读  **共享资源**  其实也没有问题 
- 在多个线程对  **共享资源**  读写操作时发生指令交错，就会出现问题 
一段代码块内如果存在对  **共享资源**  的多线程读写操作，称这段代码块为临界区.

竞态条件 Race Condition: 多个线程在临界区内执行，由于代码的  **执行序列不同**  而导致结果无法预测，称之为发生了  **竞态条件**
## 解决方案
- 阻塞式的解决方案：synchronized，Lock 
- 非阻塞式的解决方案：原子变量
### synchronized
synchronized 实际是用  **对象锁保证了临界区内代码的原子性**，临界区内的代码对外是不可分割的，不会被线程切 换所打断。
加锁的时候，要对使用同一个锁对象的锁，不能一个加锁一个不加锁。
```java
@Slf4j(topic = "c.Test1-ShareQuestion")  
public class Test2 {  
    static int cnt =0;  
    //创建静态锁对象  
    static Object Lock = new Object();  
    
    public static void main(String[] args) throws InterruptedException {  
        Thread t1 = new Thread(() -> {  
            for (int i = 0; i < 5000; i++) {  
                synchronized (Lock) {  
                    cnt++;  
                }  
            }  
        }, "t1");  
  
        Thread t2 = new Thread(() -> {  
            for (int i = 0; i < 5000; i++) {  
                synchronized (Lock) {  
                    cnt--;  
                }  
            }  
        }, "t2");  
        t1.start();  
        t2.start();  
        t1.join();  
        t2.join();  
        log.debug("cnt:{}",cnt);  
    }  
}
```
#### 注意：
synchronized字段只能锁对象，锁住的是同一个对象才会有互斥效果。
还有两种synchronized方法，
一种是锁在方法上（实际上是锁的是this对象），一种是锁在静态方法上（实际上锁的是整个类对象）
![[Pasted image 20251018185507.png]]
## 线程安全分析
成员变量和静态变量是否线程安全？ 
```text
如果它们没有共享，则线程安全 
如果它们被共享了，根据它们的  状态是否能够改变  ，又分两种情况 
	如果只有读操作，则线程安全 
	如果有读写操作，则这段代码是临界区，需要考虑线程安全
```
局部变量是否线程安全？
```text
局部变量是线程安全的 
但局部变量引用的对象则未必 
	如果该对象没有逃离方法的作用访问，它是线程安全的 
	如果该对象(用return)逃离方法的作用范围，需要考虑线程安全
```
private 或 final 提供【安全】的意义所在，请体会开闭原则中的【闭】
### 常见线程安全类
`String、Integer、StringBuffer、Random、Vector、Hashtable、java.util.concurrent（JUC）` 包下的类。

这里说它们是线程安全的是指，多个线程调用它们同一个实例的某个方法时，是线程安全的。也可以理解为:
```java
Hashtable table = new Hashtable(); 
new Thread(()->{ 
	table.put("key", "value1"); 
	}).start();
	 
new Thread(()->{ 
	table.put("key", "value2"); 
}).start();
```
- 它们的每个方法是原子的 
- 但注意它们多个方法的组合不是原子的

不可变类线程安全性:
`String、Integer` 等都是不可变类，因为其内部的状态不可以改变，因此它们的方法都是线程安全的
有疑问，String 有 `replace，substring` 等方法【可以】改变值啊，那么这些方法又是如何保证线程安 全的呢？ --> 创建一个新的，把引用换了一下。
## Monitor
Monitor 被翻译为监视器或管程
每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的 Mark Word 中就被设置指向 Monitor 对象的指针
### Monitor 结构如下
![[Pasted image 20251019084258.png]]
- 刚开始 Monitor 中 Owner 为 null 
- 当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2，**Monitor中只能有一 个 Owner** 
- 在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入 `EntryList BLOCKED `
- Thread-2 执行完同步代码块的内容，然后唤醒 `EntryList` 中等待的线程来竞争锁，竞争的时是**非公平的** (由jdk决定)
- 图中` WaitSet` 中的 Thread-0，Thread-1 是之前获得过锁，但条件不满足进入 WAITING 状态的线程，后面讲 wait-notify 时会分析

注意： 
	synchronized 必须是进入同一个对象的 monitor 才有上述的效果 
	不加 synchronized 的对象不会关联监视器，不遵从以上规则
## synchronized优化原理
### 轻量级锁
轻量级锁的**使用场景**：如果一个对象虽然有多线程要加锁，但加锁的时间是错开的（也就是没有竞争），那么可以 使用轻量级锁来优化。 
轻量级锁对使用者是透明的，即语法仍然是 synchronized 

假设有两个方法同步块，利用同一个对象加锁:
```java
static final Object obj = new Object(); 
public static void method1() { 
	synchronized( obj ) { 
	// 同步块 A 
	method2(); 
	} 
} 
public static void method2() { 
	synchronized( obj ) { 
	// 同步块 B 
} 
```
过程：
1. 创建锁记录（Lock Record）对象，每个线程都的栈帧都会包含一个锁记录的结构，内部可以存储锁定对象的 Mark Word
![[Pasted image 20251019091106.png]]
2. 让锁记录中 Object reference 指向锁对象，并尝试用 `cas` 替换 Object 的 Mark Word，将 Mark Word 的值存入锁记录
![[Pasted image 20251019091141.png]]
3. 如果 `cas` 替换成功，对象头中存储了 锁记录地址和状态 00 ，表示由该线程给对象加锁，这时图示如下
![[Pasted image 20251019091212.png]]
4. 如果 `cas` 失败，有两种情况
- 如果是其它线程已经持有了该 Object 的轻量级锁，这时表明有竞争，进入锁膨胀过程 
- 如果是自己执行了 synchronized 锁重入，那么再添加一条 Lock Record 作为重入的计数
![[Pasted image 20251019091324.png]]
5. 当退出 synchronized 代码块（解锁时）如果有取值为 null 的锁记录，表示有重入，这时重置锁记录，表示重 入计数减一
![[Pasted image 20251019091407.png]]
6. 当退出 synchronized 代码块（解锁时）锁记录的值不为 null，这时使用 `cas` 将 Mark Word 的值恢复给对象头 
	成功，则解锁成功 
	失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程
### 锁膨胀
如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有 竞争），这时需要进行  **锁膨胀**，将轻量级锁变为重量级锁。

锁膨胀不可逆，变不会轻量级锁了。

1. 当 Thread-1 进行轻量级加锁时，Thread-0 已经对该对象加了轻量级锁
![[Pasted image 20251019091817.png]]
2. 这时 Thread-1 加轻量级锁失败，进入锁膨胀流程 
	即为 Object 对象申请 Monitor 锁，让 Object 指向重量级锁地址 
	然后自己进入 Monitor 的 `EntryList BLOCKED`
![[Pasted image 20251019092148.png]]
3. 当 Thread-0 退出同步块解锁时，使用 `cas` 将 Mark Word 的值恢复给对象头，(它想解轻量级锁，但发现锁对象是重量级)失败。
4. 这时会进入重量级解锁 流程，即在Object对象头中，按照 Monitor 地址找到 Monitor 对象，将 Owner字段（当前锁的所有者）设置为 null，唤醒 `EntryList` 中 BLOCKED 线程
### 自旋优化
锁膨胀时，后来的线程（包括触发锁膨胀的线程）会进入自旋等待，避免阻塞。
重量级锁竞争的时候，也可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步 块，释放了锁），这时当前线程就可以避免阻塞。

自旋一定是多核CPU下才具有意义。
注意：
	1. 自旋会占用 CPU 时间，单核 CPU 自旋就是浪费，多核 CPU 自旋才能发挥优势。 
	2. 在 Java 6 之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会 高，就多自旋几次；反之，就少自旋甚至不自旋，总之，比较智能。 
	3. Java 7 之后不能控制是否开启自旋功能
### 偏向锁
轻量级锁在没有竞争时（就自己这个线程），每次重入仍然需要执行 CAS 操作。 

Java 6 中引入了偏向锁来做进一步优化：只有第一次使用 CAS 将线程 ID 设置到对象的 Mark Word 头，之后发现 这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有
#### 偏向状态
**JDK 17中完全移除了偏向锁**,**JDK 15默认关闭**
![[Pasted image 20251019094422.png]]
- 如果开启了偏向锁（默认开启），那么对象创建后，markword 值为 0x05 即最后 3 位为 101，这时它的` thread、epoch、age` 都为 0 
- 偏向锁是默认是延迟的，不会在程序启动时立即生效，如果想避免延迟，可以加 VM 参数 `- XX:BiasedLockingStartupDelay=0` 来禁用延迟 
- 如果没有开启偏向锁，那么对象创建后，markword 值为 0x01 即最后 3 位为 001，这时它的 `hashcode`、 age 都为 0，第一次用到 `hashcode` 时才会赋值

加锁顺序：偏向锁 --> 轻量级锁 --> 重量级锁
来。
#### 撤销 
1. 调用对象 `hashCode`
调用了对象的 hashCode，但偏向锁的对象 `MarkWord` 中存储的是线程 id，如果调用 `hashCode` 会导致偏向锁被撤销
注意：
	1. 调用hashcode会直接把偏向锁会变为Normal状态（因为空间不够）
	2. 为什么轻量级锁和重量级锁调用hashcode不会有这个问题呢？
		因为轻量级锁的hashcode会存在线程栈帧的锁记录中。
		重量级锁的hashcode会存在Monitor对象中，解锁时会还原回
2. 其它线程使用对象
当有其它线程使用偏向锁对象时，会将偏向锁升级为轻量级锁。
3. 调用 wait/notify
这两个机制只有重量级锁有，使用的时候会升级为重量级锁。
#### 批量重偏向
如果对象虽然被多个线程访问，但没有竞争，这时偏向了线程 T1 的对象仍有机会重新偏向 T2，重偏向会重置对象 的 Thread ID 
当撤销偏向锁阈值超过 20 次后，jvm 会这样觉得，我是不是偏向错了呢，于是会在给这些对象加锁时重新偏向至 加锁线程
#### 批量撤销
当撤销偏向锁阈值超过 40 次后，jvm 会这样觉得，自己确实偏向错了，根本就不该偏向。
于是整个类的所有对象 都会变为不可偏向的，新建的对象也是不可偏向的
### 锁消除
当JVM确定同步操作不会产生竞争时，完全移除同步代码。
- java -jar benchmarks.jar 正常执行
```text
Benchmark    Mode   Samples   Score   Score error   Units 
c.i.MyBenchmark.a   avgt   5   1.542   0.056   ns/op 
c.i.MyBenchmark.b   avgt   5   1.518   0.091   ns/op
```
- `java -XX:-EliminateLocks -jar benchmarks.jar` 加上参数关闭锁消除功能
```text
Benchmark   Mode   Samples   Score   Score error   Units 
c.i.MyBenchmark.a   avgt   5   1.507    0.108 ns/op 
c.i.MyBenchmark.b   avgt   5   16.976   1.572 ns/op
```
原理：
1. **逃逸分析**: 分析对象的作用域
 - 对象不会逃逸出当前线程 → 可以消除锁
 - 对象可能被其他线程访问 → 保留锁
2. **对象生命周期分析**:
```java
public void method() {
    // 局部对象，不会逃逸 - 锁可消除
    Object lock = new Object();
    synchronized(lock) {
        // 操作
    }
}
```
### 锁粗化: 
将多个连续的锁操作合并为一个更大范围的锁操作，减少锁的获取和释放次数。
1. **JIT编译器分析**: 在即时编译阶段，编译器检测到连续的同步块
2. **逃逸分析**: 确定锁对象不会逃逸出当前上下文
3. **范围扩展**: 将多个小范围的锁合并为一个大范围的锁
4. **性能优化**: 减少锁竞争和上下文切换开销
### wait notify 原理
![[Pasted image 20251019110139.png]]
- Owner 线程发现条件不满足，调用 wait 方法，即可进入 `WaitSet` 变为 WAITING 状态 
- BLOCKED 和 WAITING 的线程都处于阻塞状态，不占用 CPU 时间片 
- BLOCKED 线程会在 Owner 线程释放锁时唤醒 
- WAITING 线程会在 Owner 线程调用 notify 或 `notifyAll` 时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入 `EntryList` 重新竞争
#### 相关API
```text
obj.wait()        \\让进入 object 监视器的线程到 waitSet 等待 
obj.notify()      \\在 object 上正在 waitSet 等待的线程中挑一个唤醒 
obj.notifyAll()   \\让 object 上正在 waitSet 等待的线程全部唤醒
```
这三个都是线程之间进行协作的手段，都属于 Object 对象的方法。  **必须获得此对象的锁**  ，才能调用这几个方法。

注意：
	1. 休息室（wait）可以有多个线程。
	2. wait() 方法会释放对象的锁，进入 `WaitSet` 等待区，从而让其他线程就机会获取对象的锁。**无限制等待，直到 notify 为止** 
	3. wait(long n) 有时限的等待, 到 n 毫秒后结束等待，或是被 notify
#### wait notify 的正确姿势
sleep(long n) 和 wait(long n) 的区别：
	1. sleep 是 Thread 方法，而 wait 是 Object 的方法 
	2. sleep 不需要强制和 synchronized 配合使用，但 wait 需要 和 synchronized 一起用 
	3. sleep 在睡眠的同时，不会释放对象锁的，但 wait 在等待的时候会释放对象锁 
	4. 它们 的线程状态都是 TIMED_WAITING

获取锁对象的代码建议改成final，意味着引用不可变。

1. notify 只能随机唤醒一个 `WaitSet` 中的线程，这时如果有其它线程也在等待，那么就可能唤醒不了正确的线 程，称之为【虚假唤醒】 
- 解决方法，改为 `notifyAll`
2. 用 `notifyAll` 仅解决某个线程的唤醒问题，但使用 if + wait 判断仅有一次机会，一旦条件不成立，就没有重新 判断的机会了
- 解决方法，用 while + wait，当条件不成立，再次 wait
建议后面这样使用：
```java
synchronized(lock) { 
	while(条件不成立) { 
		lock.wait(); 
	} // 干活 
} 
//另一个线程 
synchronized(lock) { 
	lock.notifyAll(); 
}
```
## 设计模式
### 认识同步模式-保护者暂停
即 Guarded Suspension，用在一个线程等待另一个线程的执行结果 
要点:
	1. 有一个结果需要从一个线程传递到另一个线程，让他们关联同一个 `GuardedObject `
	2. 如果有结果不断从一个线程到另一个线程那么可以使用消息队列（见生产者/消费者）
	3. JDK 中，join 的实现、Future 的实现，采用的就是此模式 
	4. 因为要等待另一方的结果，因此归类到同步模式
![[Pasted image 20251019131156.png]]
join的实现方式也是这个模式。
`t.join()`，循环阻塞执行这条语句的线程，isAlive判断调用者t线程对象的状态
### 异步模式之生产者/消费者
要点 
- 与前面的保护性暂停中的 `GuardObject` 不同，不需要产生结果和消费结果的线程一一对应 
- 消费队列可以用来平衡生产和消费的线程资源 
- 生产者仅负责产生结果数据，不关心数据该如何处理，而消费者专心处理结果数据 
- 消息队列是有容量限制的，满时不会再加入数据，空时不会再消耗数据 
- JDK 中各种阻塞队列，采用的就是这种模式
![[Pasted image 20251019150324.png]]
## Park & Unpark
它们是 `LockSupport` 类中的方法
```java
// 暂停当前线程 
LockSupport.park(); 
// 恢复某个线程的运行 
LockSupport.unpark(暂停线程对象)
```
特点: 
	1. 与 Object 的 wait & notify 相比 `wait，notify` 和 `notifyAll` 必须配合 Object Monitor 一起使用，而 `park，unpark` 不必
	2. park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，而 notify 只能随机唤醒一个等待线程，notifyAll 是唤醒所有等待线程，就不那么【精确】
	3. park & unpark 可以先 unpark，而 wait & notify 不能先 notify
### 原理
每个线程都有自己的一个 Parker 对象，由三部分组成 `_counter` ，` _cond` 和 `_mutex` 打个比喻 
- 线程就像一个旅人，Parker 就像他随身携带的背包，条件变量就好比背包中的帐篷。_counter 就好比背包中 的备用干粮（0 为耗尽，1 为充足） 
- 调用 park 就是要看需不需要停下来歇息 
	如果备用干粮耗尽，那么钻进帐篷歇息 
	如果备用干粮充足，那么不需停留，继续前进 
- 调用 unpark，就好比令干粮充足 
	如果这时线程还在帐篷，就唤醒让他继续前进 
	如果这时线程还在运行，那么下次他调用 park 时，仅是消耗掉备用干粮，不需停留继续前进 
		因为背包空间有限，多次调用 unpark 仅会  **补充一份**  备用干粮

### park后调用unpark
![[Pasted image 20251019183303.png]]
1. 当前线程调用 `Unsafe.park()` 方法 
2. 检查 `_counter `，本情况为 0，这时，获得 `_mutex` 互斥锁 
3. 线程进入 `_cond` 条件变量阻塞 
4. 设置 `_counter = 0`

![[Pasted image 20251019183425.png]]
1. 调用 `Unsafe.unpark(Thread_0)` 方法，设置 `_counter` 为 1 
2. 唤醒 `_cond` 条件变量中的 Thread_0 
3. Thread_0 恢复运行 
4. 设置 `_counter` 为 0
### unpark后调用park
![[Pasted image 20251019184056.png]]
1. 调用 `Unsafe.unpark(Thread_0)` 方法，设置 `_counter` 为 1 
2. 当前线程调用 `Unsafe.park()` 方法 
3. 检查 `_counter` ，本情况为 1，这时线程无需阻塞，继续运行 
4. 设置 `_counter` 为 0
## 多把锁
考虑锁的粗细粒度，有时候一个大的粗粒度锁区块的性能，
不如多个细粒度锁的性能.

将锁的粒度细分 
- 好处，是可以增强并发度 
- 坏处，如果一个线程需要同时获得多把锁，就容易发生死锁
## 活跃性
饥饿、死锁、活锁都可以用ReentrantLock解决，活锁还可以通过增加随机休眠时间来解决。
### 死锁 
有这样的情况：一个线程需要同时获取多把锁，这时就容易发生死锁
即，
t1 线程 获得 A对象 锁，接下来想获取 B对象 的锁 t2 线程 获得 B对象 锁，接下来想获取 A对象 的锁 
解决方法：
	顺序加锁。
#### 定位死锁
检测死锁可以使用 jconsole工具，或者使用 `jps` 定位进程 id，再用 `jstack` 定位死锁

### 活锁
活锁出现在两个线程互相改变对方的结束条件，最后谁也无法结束.
解决办法：
	让系统执行时间有一定的交错，不要让它集中在一起就行。（随机休眠）

### 饥饿
一个线程由于优先级太低，始终得不到 CPU 调度执行，也不能够结束。
## `ReentrantLock`
相对于 synchronized 它具备如下特点 
- 可中断 
- 可以设置超时时间 
- 可以设置为公平锁 
- 支持多个条件变量
与 synchronized 一样，都支持可重入
基本语法(lock和unlock要成对出现)：
获取锁对象：
```java
// 获取锁 
reentrantLock.lock(); 
	try { 
		// 临界区 
	} finally { 
		// 释放锁 
		reentrantLock.unlock(); 
}
```
让类具备锁的功能，**每个对象实例都是一个独立的锁**:
```java
class Chopstick extends ReentrantLock 
```
- 可重入
可重入是指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权利再次获取这把锁
- 可打断
- 锁超时
- 公平锁:ReentrantLock 默认是不公平的(饥饿问题)
```java
//改为公平性(先入先得的方法实现的)
ReentrantLock lock = new ReentrantLock(true);
```
公平锁的设计目的是为了解决饥饿问题，
但公平锁一般没有必要，会降低并发度
### 条件变量
synchronized 中也有条件变量，就是我们讲原理时那个 `waitSet` 休息室(wait方法)，当条件不满足时进入 `waitSet` 等待 
`ReentrantLock` 的条件变量比 synchronized 强大之处在于，它是支持多个条件变量的，这就好比 
- synchronized 是那些不满足条件的线程都在一间休息室等消息 
- 而 `ReentrantLock` 支持多间休息室，有专门等烟的休息室、专门等早餐的休息室、唤醒时也是按休息室来唤醒
- 
使用要点： 
	1. await 前需要获得锁 
	2. await 执行后，会释放锁，进入 **`conditionObject`** 等待 
	3. await 的线程被唤醒（或打断、或超时）取重新竞争 lock 锁 
	4. 竞争 lock 锁成功后，从 await 后继续执行

注意：
	用signal -- 唤醒一个等待线程
	用signalAll -- 唤醒所有等待线程

用newCondition创建：
```java
static ReentrantLock lock = new ReentrantLock(); 
static Condition waitCigaretteQueue = lock.newCondition(); 
static Condition waitbreakfastQueue = lock.newCondition();
```
# 共享内存模型
上一章讲解的 Monitor 主要关注的是访问共享变量时，保证临界区代码的**原子性**.
这一章我们进一步深入学习共享变量在多线程间的【可见性】问题与多条指令执行时的【有序性】问题
## Java 内存模型
JMM 即 Java Memory Model，它定义了主存、工作内存抽象概念，底层对应着 CPU 寄存器、缓存、硬件内存、 CPU 指令优化等。 
JMM 体现在以下几个方面 
- 原子性 - 保证指令不会受到线程上下文切换的影响 
- 可见性 - 保证指令不会受 `cpu` 缓存的影响 
- 有序性 - 保证指令不会受 `cpu` 指令并行优化的影响
## 可见性
退不出的循环。
先来看一个现象，main 线程对 run 变量的修改对于 t 线程不可见，导致了 t 线程无法停止:
```java
static boolean run = true; 
public static void main(String[] args) throws InterruptedException { 
	Thread t = new Thread(()->{ 
		while(run){ 
		// .... 
		} 
	}); 
	t.start(); 
	sleep(1); run = false; // 线程t不会如预想的停下来 
}
```
为什么呢？分析一下： 
1. 初始状态， t 线程刚开始从主内存读取了 run 的值到工作内存。
![[Pasted image 20251021083731.png]]
 2. 因为 t 线程要频繁从主内存中读取 run 的值，JIT 编译器会将 run 的值缓存至自己工作内存中的高速缓存中， 减少对主存中 run 的访问，提高效率
![[Pasted image 20251021083813.png]]
3. 1 秒之后，main 线程修改了 run 的值，并同步至主存，而 t 是从自己工作内存中的高速缓存中读取这个变量 的值，结果永远是旧值
![[Pasted image 20251021083846.png]]

解决方法:
1. volatile（易变关键字, 更轻量，推荐使用） 
它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取 它的值，线程操作 volatile 变量都是直接操作主存
2. synchronized (加锁)
**关键机制**：当线程释放锁时，JMM（Java内存模型）保证：
- 该线程在同步块内修改的所有变量（包括 `run`）会**立即刷新到主内存**
- 当其他线程获取同一个锁时，会**强制从主内存重新读取**这些变量的最新值
3. 前面死循环加入sout即使不加 volatile 修饰符，也能保证可见性
CPU不加操作一直空转  加了就会上下文切换就会重新读取(它内部的 synchronized 块创建了内存屏障，强制进行了内存同步)

一个线程对 volatile 变量的修改对另一个线程可见， 不能保证原子性。
(只能保证看到最新值，不能解决指令交错问题)
synchronized 语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性。但缺点是 synchronized 是属于重量级操作，性能相对更低
## 有序性
JVM 会在不影响正确性的前提下，可以调整语句的执行顺序.
这种特性称之为『指令重排』，多线程下『指令重排』会影响正确性。

现代处理器会设计为一个时钟周期完成一条执行时间最长的 CPU 指令。每条指令都可以分为： 取指令 - 指令译码 - 执行指令 - 内存访问 - 数据 写回 这 5 个阶段
![[Pasted image 20251021095135.png]]
现代CPU支持多指令流水线，CPU 可以在一个时钟周期内，同时运行五条指令的不同阶段
![[Pasted image 20251021095149.png]]
本质上，流水线技术并不能缩短单条指令的执行时间，但它变相地提高了 指令地吞吐率。
指令重排的前提是，重排指令不能影响结果
### 诡异的结果
```java
int num = 0;
boolean ready = false; 
// 线程1 执行此方法 
public void actor1(I_Result r) { 
	if(ready) { 
	r.r1 = num + num; 
	} else { 
	r.r1 = 1; 
	} 
} 
// 线程2 执行此方法 
public void actor2(I_Result r) { 
	num = 2; 
	ready = true; 
}
```
结果可以是0，1，4，出现0的原因就是指令重排序把actor2中的两条指令顺序交换了。

这个现象需要通过大量测试才能复现：
借助 java 并发压测工具 [`jcstress`]( https://wiki.openjdk.java.net/display/CodeTools/jcstress)
**使用Maven原型(archetype)快速生成JCStress测试项目**
```mvn
mvn archetype:generate -DinteractiveMode=false -DarchetypeGroupId=org.openjdk.jcstress - DarchetypeArtifactId=jcstress-java-test-archetype -DarchetypeVersion=0.5 -DgroupId=cn.itcast - DartifactId=ordering -Dversion=1.0
```

禁止重排序的方法：
```java
boolean volatile ready = false;
```
这样可以防止actor2中ready执行之前的代码重排序（加了一个写屏障）。
### volatile 原理
volatile 的底层实现原理是内存屏障，`Memory Barrier(Memory Fence)` 
- 对 volatile 变量的写指令后会加入写屏障 
- 对 volatile 变量的读指令前会加入读屏障
#### 如何保证可见性
写屏障（sfence）保证在该屏障之前的，对共享变量的改动，都同步到主存当中
```java
public void actor2(I_Result r) { 
	num = 2; 
	ready = true; // ready 是 volatile 赋值带写屏障 
	// 写屏障 
}
```
而读屏障（lfence）保证在该屏障之后，对共享变量的读取，加载的是主存中最新数据
```java
public void actor1(I_Result r) { 
	// 读屏障 
	// ready 是 volatile 读取值带读屏障 
	if(ready) { 
		r.r1 = num + num; 
	} else { 
		r.r1 = 1; 
	} 
}
```
#### 如何保证有序性
写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后。
读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前。

还是那句话，不能解决指令交错： 
	写屏障仅仅是保证之后的读能够读到最新的结果，但不能保证读跑到它前面去 
	而有序性的保证也只是保证了本线程内相关代码不被重排序
###  双重锁问题
```java
public final class Singleton {
    private Singleton() { }
    private static Singleton INSTANCE = null;
    
    public static Singleton getInstance() {
        if (INSTANCE == null) {
            // 首次访问会同步，而之后的使用没有 synchronized
            synchronized(Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```
这个代码有指令重排问题。
假设有两个线程：**线程A**和**线程B**，步骤：
	1. 分配内存
	2. 赋值引用       ← 重排序到这里！
	3. 初始化对象    ← 其他线程可能在这时候访问到未初始化的对象
**时间线**：
![[Pasted image 20251021105604.png]]
JVM字节码角度，就是线程A的初始化引用后的putStatic指令，重排序到线程B的getStatic指令之后（B发现A已经创建了对象【但其实并未赋值】，直接调用了这个对象）。
### 双重锁问题解决
在Singleton变量前面加上 volatile 关键字
```java
private static volatile Singleton INSTANCE = null;
```
1. **可见性**：确保一个线程对变量的修改对其他线程立即可见
2. **禁止指令重排序**：在`volatile`写操作之前插入内存屏障，防止指令重排序跨越这个屏障
	B发现A已经创建了对象【但其实并未赋值】，这时由于  **B带读屏障**  而  **A具有写屏障** ，它会拿到null值，继续进行，这时就可以synchronized保证它的安全性了。
这样就能确保：**对象完全初始化完成后，引用才会被赋值给`INSTANCE`**。

注意:
	在 JDK 5 以上的版本的 volatile 才会真正有效
### happens-before规则(7个)
happens-before规则：一套规定了对共享变量的写操作对其它线程的读操作可见 的一套关于  **可见性与有序性** 的规则总结。
1. (synchronized)线程解锁 m 之前对变量的写，对于接下来对 m 加锁的其它线程对该变量的读可见
```java
static int x; 
static Object m = new Object(); 
new Thread(()->{ 
	synchronized(m) { 
		x = 10;
	} 
},"t1").start(); 

new Thread(()->{ 
	synchronized(m) { 
		System.out.println(x); 
	} 
},"t2").start();
```
2. 线程对 volatile 变量的写，对接下来其它线程对该变量的读可见
```java
volatile static int x; 

new Thread(()->{ 
	x = 10; 
},"t1").start(); 

new Thread(()->{ 
	System.out.println(x); 
},"t2").start();
```
3. 线程 start 前对变量的写，对该线程开始后对该变量的读可见
```java
static int x; 
x = 10; 
new Thread(()->{ 
	System.out.println(x); 
},"t2").start();
```
4. 线程结束前对变量的写，对其它线程得知它结束后的读可见（比如其它线程调用 t1.isAlive() 或 t1.join()等待 它结束）
```java
static int x; 
Thread t1 = new Thread(()->{ 
	x = 10; 
},"t1"); 
t1.start(); 
t1.join(); 
System.out.println(x);
```
5. 线程 t1 打断 t2（interrupt）前对变量的写，对于其他线程得知 t2 被打断后对变量的读可见（通过 t2.interrupted 或 t2.isInterrupted）
```java
 static int x;

public static void main(String[] args) {
    Thread t2 = new Thread(() -> {
        while (true) {
            if (Thread.currentThread().isInterrupted()) {
                System.out.println(x);
                break;
            }
        }
    }, "t2");
    t2.start();

    new Thread(() -> {
        try {
            Thread.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        x = 10;
        t2.interrupt();
    }, "t1").start();

    while (!t2.isInterrupted()) {
        Thread.yield();
    }
    System.out.println(x);
}
```
6. 对变量默认值（0，false，null）的写，对其它线程对该变量的读可见 
7. 具有传递性，如果` x hb-> y `并且 `y hb-> z `那么有` x hb-> z `，配合 volatile 的防指令重排
```java
volatile static int x; 
static int y; 
new Thread(()->{ 
	y = 10; 
	x = 20; 
},"t1").start(); 

new Thread(()->{ 
// x=20 对 t2 可见, 同时 y=10 也对 t2 可见 
	System.out.println(x); 
},"t2").start();
```
### 线程安全单例须知
单例模式有很多实现方法，饿汉、懒汉、静态内部类、枚举类，试分析每种实现下获取单例对象（即调用 getInstance）时的线程安全，并思考注释中的问题 
	饿汉式：类加载就会导致该单实例对象被创建 
	懒汉式：类加载不会导致该单实例对象被创建，而是首次使用该对象时才会创建
![[Pasted image 20251021114450.png]]
5：提供更好的封装性，内部实现一些懒惰初始化；可以拿到对象值后做一些控制；提供泛型支持
# 无锁并发（乐观锁）
## CAS 与 volatile
关键是 compareAndSet，它的简称就是 CAS （也有 Compare And Swap 的说法），它必须是原子操作。
注意 :
	其实 CAS 的底层是 `lock cmpxchg` 指令（X86 架构），在单核 CPU 和多核 CPU 下都能够保证【比较-交换】的原子性。
	在多核状态下，某个核执行到带 lock 的指令时，CPU 会让总线锁住，当这个核把此指令执行完毕，再 开启总线。这个过程中不会被线程的调度机制所打断，保证了多个线程对内存操作的准确性，是原子的。
```java
public void withdraw(Integer amount) { 
	while(true) { 
	// 需要不断尝试，直到成功为止 
		while (true) { 
			// 比如拿到了旧值 1000 
			int prev = balance.get(); 
			// 在这个基础上 1000-10 = 990 
			int next = prev - amount; 
			/* compareAndSet 正是做这个检查，在 set 前，先比较 prev 与当前值 
			- 不一致了，next 作废，返回 false 表示失败 
			  比如，别的线程已经做了减法，当前值已经被减成了 990 
			  那么本线程的这次 990 就作废了，进入 while 下次循环重试 
			- 一致，以 next 设置为新值，返回 true 表示成功 
			*/ 
			if (balance.compareAndSet(prev, next)) { 
				break; 
			} 
		} 
	} 
}
```
### volatile 
获取共享变量时，为了保证该变量的可见性，需要使用 volatile 修饰。 

它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取 它的值，线程操作 volatile 变量都是直接操作主存。即一个线程对 volatile 变量的修改，对另一个线程可见。 
	注意 
	volatile 仅仅保证了共享变量的可见性，让其它线程能够看到最新值，但不能解决指令交错问题（不能保证原子性）
 CAS **必须借助 volatile** 才能读取到共享变量的最新值来实现【比较并交换】的效果
### 为什么无锁效率高
无锁情况下，即使重试失败，线程始终在高速运行，没有停歇，而 synchronized 会让线程在没有获得锁的时 候，发生上下文切换，进入阻塞。
- 即，减少上下文切换的机会，线程始终在高速运行。
无锁编程虽然避免了线程阻塞，但在高竞争环境下，由于线程需要持续运行来重试CAS操作，会大量消耗CPU资源。当线程数量超过CPU核心数时，操作系统仍然需要进行线程调度，导致上下文切换的开销。
- 虽然避免了阻塞，但在高压环境下会大量消耗CPU资源，当线程数大于CPU核心数时，OS会线程调度，发生上下文切换。
## CAS方式实现的工具类
JUC包下的Atomic子包提供的工具类。

1. 原子整数：`AtomicBoolean 、AtomicInteger、 AtomicLong`
2. 原子引用：`AtomicReference 、AtomicMarkableReference 、AtomicStampedReference`
	ABA问题：主线程无法感知其它线程是否对变量进行了修改，主线程只是根据最终值进行对比感知是否修改
	如果目的是：只要有其它线程【动过了】共享变量，那么 `cas` 就算失败
	解决方法：这时，仅比较值是不够的，需要再加一个版本号
		使用`AtomicMarkableReference（带版本号） 、AtomicStampedReference（只关心改没改过）`--> 两个用法完全一样，只不过把  版本号数值  改成了  布尔值
3. 原子数组:  `AtomicIntegerArray、 AtomicLongArray、 AtomicReferenceArray`
4. 字段更新器:  `AtomicReferenceFieldUpdater // 域 字段、 AtomicIntegerFieldUpdater 、AtomicLongFieldUpdater` -->  有一个要求：属性值必须用volatile修饰
5. 原子累加器：LongAdder
	`LongAdder` 性能优于 `AtomicLong` 的原因是：在有竞争时，设置多个累加单元（Cell），这样它们在累加时操作不同的 Cell 变量，最后将结果汇总起来  ==> 减少了CAS的重试失败，从而提高性能。
	Cell数目  不会超过  你的CPU的核心数
### LongAdder源码
并发编程 P127.
#### 伪共享
其中 Cell 即为累加单元
```java
// 防止缓存行伪共享 
@sun.misc.Contended static final class Cell { 
	volatile long value; 
	Cell(long x) { value = x; } 
	// 最重要的方法, 用来 cas 方式进行累加, prev 表示旧值, next 表示新值 
	final boolean cas(long prev, long next) { 
		return UNSAFE.compareAndSwapLong(this, valueOffset, prev, next); 
	} 
	// 省略不重要代码 
}
```
得从缓存说起:
![[Pasted image 20251021170414.png]]
![[Pasted image 20251021170607.png]]
缓存的加入会造成数据副本的产生，即同一份数据会缓存在不同核心的缓存行中
CPU 要保证数据的一致性，如果某个 CPU 核心更改了数据，其它 CPU 核心对应的整个缓存行必须失效
![[Pasted image 20251021170941.png]]
因为 Cell 是数组形式，在内存中是连续存储的，一个 Cell 为 24 字节（16 字节的对象头和 8 字节的 value），因 此缓存行可以存下 2 个的 Cell 对象。这样问题来了： 
- Core-0 要修改 `Cell[0]` 
- Core-1 要修改 `Cell[1] `
无论谁修改成功，都会导致对方 Core 的缓存行失效，比如 Core-0 中 `Cell[0]=6000, Cell[1]=8000` 要累加 `Cell[0]=6001, Cell[1]=8000` ，这时会让 Core-1 的缓存行失效

@sun.misc.Contended 用来解决这个问题
原理:  是在使用此注解的对象或字段的前后各增加 128 字节大小的 padding，从而让 CPU 将对象预读至缓存时占用不同的缓存行，这样，不会造成对方缓存行的失效。
#### 渐进式冲突解决(LongAdder源码得来)
```java
// 伪代码表示这个逻辑
if (!collide) {  // 第一次冲突
    collide = true;  // 标记为已冲突
    // 尝试换一个Cell槽位
    if (!tryUpdateInNewCell()) {
        // 如果换槽位还不行，才真正扩容
        expandAndRetry();
    }
} else if (/* 其他条件 */) {
    // 真正的扩容逻辑
    realExpansion();
}
```
**先用尽所有低成本方案，再考虑高成本方案**。
扩容机制：
```java
Cell[] rs = new Cell[n << 1]; //扩大两倍
for (int i = 0; i < n; ++i)
    rs[i] = as[i];
als = rs;
```
### `LongAdder` 的 `LongAccumulate` 逻辑
#### 性能优势:

|场景|策略|效果|
|---|---|---|
|无竞争|直接更新 base|接近 AtomicLong|
|中等竞争|Cell 数组分散|避免 CAS 重试|
|高竞争|动态扩容|维持性能稳定|
#### 核心设计思想:
	**"空间换时间，分散竞争"** - 通过多副本存储来减少线程间的竞争冲突

#### 三级处理策略:
1. **第一级：直接 CAS 更新 base（无竞争）**
```java
if (cells == null && casBase(b = base, b + x)) {
    return; // 成功则直接返回
}
```
- **适用场景**：单线程或低并发情况
- **优势**：性能最优，只需一次 CAS 操作
- **成本**：最低

2. **第二级：使用 Cell 数组（中等竞争）**
```java
// 每个线程哈希到不同的 Cell 槽位
int index = threadHash & (cells.length - 1);
Cell cell = cells[index];
if (cell.cas(v = cell.value, v + x)) {
    return; // 成功则返回
}
```
- **适用场景**：多线程竞争但冲突不严重
- **优势**：线程各自更新不同位置，避免竞争
- **成本**：数组内存开销
3. **第三级：冲突处理与扩容（高竞争）**
```java
// 进入 longAccumulate 的复杂逻辑
if (!collide) {
    collide = true;        // 第一次冲突：标记
} else {
    // 真正扩容：创建更大数组，重新哈希
    cells = expandCells();
    collide = false;
}
```
- **适用场景**：激烈竞争导致 CAS 频繁失败
- **策略**：渐进式处理，先尝试换槽位，不行再扩容
#### **最终一致性**
```java
public long sum() {
    return base + Σcells[i].value;  // 最终求和
}
```
- 读取时不是强一致性
- 适合统计类场景，不适用需要实时精确值的场景
## Unsafe
Unsafe 对象提供了非常底层的，操作内存、线程的方法，Unsafe 对象不能直接调用，**只能通过反射获得**
# 不可变
如果一个对象是不可变的，没有可以修改它变量的程序，即使它是共享的，它也是线程安全的。
## 不可变设计
final 的使用:
类、类中所有属性都是 final 的
- 属性用 final 修饰保证了该属性是只读的，不能修改 
- 类用 final 修饰保证了该类中的方法不能被覆盖，防止子类无意间破坏不可变性

保护性拷贝:
构造新字符串对象时，会生成新的 `char[] value`，对内容进行复制 。这种通过创建副本对象来避 免共享的手段称之为【保护性拷贝（defensive copy）】
## 享元模式
Flyweight pattern. 当需要重用数量有限的同一类对象时
```text
wikipedia： A flyweight is an object that minimizes memory usage by sharing as much data as possible with other similar objects
```
### 体现
- 在JDK中 `Boolean，Byte，Short，Integer，Long，Character` 等包装类提供了 `valueOf` 方法使用的就是  享元模式。
注意：
```text
Byte, Short, Long 缓存的范围都是 -128~127 
Character 缓存的范围是 0~127 
Integer的默认范围是 -128~127 
	最小值不能变 
	但最大值可以通过调整虚拟机参数 -Djava.lang.Integer.IntegerCache.high 来改变 
Boolean 缓存了 TRUE 和 FALSE
```
- String 串池
- `BigDecimal BigInteger`
## final 原理
```java
public class TestFinal { 
	final int a = 20; 
}
```
字节码：
![[Pasted image 20251021201513.png]]
发现 final 变量的赋值也会通过 `putfield` 指令来完成，同样在这条指令之后也会加入写屏障，保证在其它线程读到 它的值时不会出现为 0 的情况
类的静态常量，链接的时候就赋值
数字比较小的时候直接在栈内存中，基本类型规定最大值，就在常量池中
## 无状态
在 web 阶段学习时，设计 Servlet 时为了保证其线程安全，都会有这样的建议，不要为 Servlet 设置成员变量，这 种  **没有任何成员变量的类是线程安全**  的
- 因为成员变量保存的数据也可以称为状态信息，因此没有成员变量就称之为【无状态】
- 总结：无状态对象线程安全 不可变对象线程安全
# 1.共享模型工具
## 1. 线程池
![[Pasted image 20251021202700.png]]
线程并非越多越好，过多导致获取不到CPU时间片的线程陷入阻塞，引起线程上下文切换问题。
拒绝策略有以下几种：
- 死等: 下面就是死等的写法
- 带超时等待: 在死等的基础上，把其await换成awaitNanos，并给定时间就行
- 放弃任务执行: 什么都不写,就是放弃执行任务
- 抛出异常: 在拒绝策略调用 `throw new RuntimeException("任务执行失败");`
- 调用者自己执行任务: 哪个线程调用并完成了拒绝策略(调用run方法)，就在哪个线程执行
```java
public void put(T task){  
    lock.lock();  
    try{  
	    //long nanos = timeUnit.toNanos(timeout); //超时等待加上
        while(queue.size()==capacity){  
            try {  
                //"等待加入任务队列{}" 
                fullWaitSet.await();  //改成awaitNanos（时间）
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
        //"加入任务队列{}"  
        queue.addLast(task);  
        emptyWaitSet.signal();  
    }finally {  
        lock.unlock();  
    }  
}
```
详细的可用去看代码。
## 2. `ThreadPoolExecutor`
JDK的线程池实现`ThreadPoolExecutor`.
#### 线程池状态：
![[Pasted image 20251025100203.png]]
从数字上比较，TERMINATED > TIDYING > STOP > SHUTDOWN > RUNNING
为什么不用两个int而是一个int的不同位表示线程状态呢？
	保证原子性。这些信息存储在一个原子变量 `ctl` 中，目的是将线程池状态与线程个数合二为一，这样就可以用一次 `cas` 原子操作 进行赋值
```java
// c 为旧值， ctlOf 返回结果为新值 
ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c)))); 
// rs 为高 3 位代表线程池状态， wc 为低 29 位代表线程个数，ctl 是合并它们 
private static int ctlOf(int rs, int wc) { return rs | wc; }
```
#### 构造方法(重要)
```java
public ThreadPoolExecutor(int corePoolSize, 
						int maximumPoolSize, 
						long keepAliveTime, 
						TimeUnit unit, 
						BlockingQueue<Runnable> workQueue, 
						ThreadFactory threadFactory, 
						RejectedExecutionHandler handler)
```
- `corePoolSize` 核心线程数目 (最多保留的线程数) 
- `maximumPoolSize` 最大线程数目 
- `keepAliveTime` 生存时间 - 针对救急线程 
- unit 时间单位 - 针对救急线程 
- `workQueue` 阻塞队列 
- `threadFactory` 线程工厂 - 可以为线程创建时起个好名字（便于定位线程和线程工厂）
- handler 拒绝策略

核心线程数 + 救急线程数 = 最大线程数
只有当核心线程和阻塞队列都放不下时，救急线程(有生存时间3,4参数控制)才会被创建并执行任务

1. 线程池中刚开始没有线程，当一个任务提交给线程池后，线程池会创建一个新线程来执行任务。 
2. 当线程数达到 `corePoolSize` 并没有线程空闲，这时再加入任务，新加的任务会被加入workQueue 队列排 队，直到有空闲的线程。 
3. 如果队列选择了**有界队列**，那么任务超过了队列大小时，会创建 `maximumPoolSize - corePoolSize` 数目的线程来救急。 (前提是配合有界队列来使用)
4. 如果线程到达 `maximumPoolSize` 仍然有新任务这时会执行拒绝策略。拒绝策略 `jdk` 提供了 4 种实现，其它 著名框架也提供了实现
	`AbortPolicy` 让调用者抛出 `RejectedExecutionException` 异常，这是默认策略
	`CallerRunsPolicy` 让调用者运行任务 
	`DiscardPolicy` 放弃本次任务 
	`DiscardOldestPolicy` 放弃队列中最早的任务，本任务取而代之 
![[Pasted image 20251025102411.png]]
	**其它组件典型实现**：
	Dubbo 的实现，在抛出 `RejectedExecutionException` 异常之前会记录日志，并 dump 线程栈信息，方便定位问题 (对`AbortPolicy`增强)
	`Netty` 的实现，是创建一个新线程来执行任务 
	`ActiveMQ` 的实现，带超时等待（60s）尝试放入队列，类似我们之前自定义的拒绝策略 
	`PinPoint` 的实现，它使用了一个**拒绝策略链**，会逐一尝试策略链中每种拒绝策略
5. 当高峰过去后，超过corePoolSize 的救急线程如果一段时间没有任务做，需要结束节省资源，这个时间由 `keepAliveTime` 和 unit 来控制。
#### Executors
根据这个构造方法，JDK Executors 类中提供了众多工厂方法来创建各种用途的线程池.

1. `newFixedThreadPool:`
特点 
- 核心线程数 == 最大线程数（没有救急线程被创建），因此也无需超时时间 
- 阻塞队列是无界的，可以放任意数量的任务
```java
public static ExecutorService newFixedThreadPool(int nThreads) { 
	return new ThreadPoolExecutor(nThreads, 
								nThreads,0L, 
								 TimeUnit.MILLISECONDS, 
								 new LinkedBlockingQueue<Runnable>()); }
```
小结: 适用于任务量已知，相对耗时的任务

2. `newCachedThreadPool`
特点 
- 核心线程数是 0， 最大线程数是 Integer.MAX_VALUE，救急线程的空闲生存时间是 60s，意味着 :
	全部都是救急线程（60s 后可以回收）
	救急线程可以无限创建
- 队列采用了 `SynchronousQueue` 实现特点是，它没有容量，没有线程来取是放不进去的（一手交钱、一手交货）
```java
public static ExecutorService newCachedThreadPool() { 
	return new ThreadPoolExecutor(0, 
						Integer.MAX_VALUE, 60L, 
						TimeUnit.SECONDS, 
						new SynchronousQueue<Runnable>()); }
```
小结: 整个线程池表现为线程数会根据任务量不断增长，没有上限，当任务执行完毕，空闲 1分钟后释放线 程。 适合任务数比较密集，但每个任务执行时间较短的情况

3.  `newSingleThreadExecutor`
使用场景: (**保证始终有一个线程可用**)
希望多个任务排队执行。线程数固定为 1，任务数多于 1 时，会放入无界队列排队。任务执行完毕，这唯一的线程 也不会被释放。
```java
public static ExecutorService newSingleThreadExecutor() { 
	return new FinalizableDelegatedExecutorService (new ThreadPoolExecutor(
										1, 1, 0L, 
										TimeUnit.MILLISECONDS,
										new LinkedBlockingQueue<Runnable>())); 
}
```
自己创建一个线程执行 和 用单线程线程池执行 串行任务,区别:
- 自己创建一个单线程串行执行任务，如果任务执行失败而终止那么没有任何补救措施，而线程池还会新建一 个线程，保证池的正常工作 
- `Executors.newSingleThreadExecutor()` 线程个数始终为1，不能修改 
	`FinalizableDelegatedExecutorService` 应用的是装饰器模式，只对外暴露了 `ExecutorService` 接口，因 此不能调用 `ThreadPoolExecutor` 中特有的方法 
- `Executors.newFixedThreadPool(1)` 初始时为1，以后还可以修改 
	对外暴露的是 `ThreadPoolExecutor` 对象，可以强转后调用 `setCorePoolSize` 等方法进行修改
### 提交任务方法
```java
// 执行任务 
void execute(Runnable command); 

// 提交任务 task，用返回值 Future 获得任务执行结果 
<T> Future<T> submit(Callable<T> task); 

// 提交 tasks 中所有任务 
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) 
		throws InterruptedException; 

// 提交 tasks 中所有任务，带超时时间 
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, 
										long timeout, TimeUnit unit) 
		throws InterruptedException; 

// 提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消 
<T> T invokeAny(Collection<? extends Callable<T>> tasks) 
		throws InterruptedException, ExecutionException;

// 提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消，带超时时间 
<T> T invokeAny(Collection<? extends Callable<T>> tasks, 
					long timeout, TimeUnit unit) 
		throws InterruptedException, ExecutionException, TimeoutException;
```
Callable 和 Runnable 的区别在于:前者有返回结果,而后者没有.
invokeAny会被线程池的线程数量影响结果，例如当线程池的线程数为1，而有多个线程想要运行，这时可能拿到非预期结果
### 关闭线程池
**shutdown** :
```java
public void shutdown() { 
	final ReentrantLock mainLock = this.mainLock; 
	mainLock.lock(); 
	try { 
		checkShutdownAccess(); 
		// 修改线程池状态 
		advanceRunState(SHUTDOWN); 
		// 仅会打断空闲线程 
		interruptIdleWorkers(); 
		onShutdown(); // 扩展点 
		ScheduledThreadPoolExecutor 
	} finally { 
		mainLock.unlock(); 
	} 
	// 尝试终结(没有运行的线程可以立刻终结，如果还有运行的线程也不会等) 
	tryTerminate(); 
}
```
线程池状态变为 SHUTDOWN :
	- 不会接收新任务 
	- 但已提交任务会执行完 
	- 此方法不会阻塞调用线程的执行 

**`shutdownNow`**:
```java
public List<Runnable> shutdownNow() {
	List<Runnable> tasks; 
	final ReentrantLock mainLock = this.mainLock; 
	mainLock.lock(); 
	try { 
		checkShutdownAccess(); 
		// 修改线程池状态 a
		anceRunState(STOP); 
		// 打断所有线程 
		interruptWorkers(); 
		// 获取队列中剩余任务 
		tasks = drainQueue(); 
	} finally { 
		mainLock.unlock(); 
	} 
	// 尝试终结 
	tryTerminate(); 
	return tasks; 
}
```
线程池状态变为 STOP 
- 不会接收新任务 
- 会将队列中的任务返回 
- 并用 interrupt 的方式中断正在执行的任务

**其它方法 :**
```java
// 不在 RUNNING 状态的线程池，此方法就返回 true 
boolean isShutdown(); 
// 线程池状态是否是 TERMINATED 
boolean isTerminated(); 
// 调用 shutdown 后，由于调用线程并不会等待所有任务运行结束，因此如果它想在线程池 TERMINATED 后做些事情，可以利用此方法等待 
boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
//awaitTermination若在等待期间，任务完成则不等剩余时间
```
### 异步模式
让有限的工作线程（Worker Thread）来轮流异步处理无限多的任务。也可以将其归类为分工模式，它的典型实现就是线程池，也体现了经典设计模式中的  **享元模式**。

固定大小线程池会有饥饿现象.
**注意**，不同任务类型应该使用不同的线程池，这样能够避免饥饿，并能提升效率
#### 创建多少线程池合适
- 过小会导致程序不能充分地利用系统资源、容易导致饥饿 
- 过大会导致更多的线程上下文切换，占用更多内存

**CPU 密集型运算**  ：
通常采用 `cpu 核数 + 1` 能够实现最优的 CPU 利用率，
+1 是保证当线程由于页缺失故障（操作系统）或其它原因 导致暂停时，额外的这个线程就能顶上去，保证 CPU 时钟周期不被浪费

**I/O 密集型运算(web应用程序)** ：
CPU 不总是处于繁忙状态，
例如，当你执行业务计算时，这时候会使用 CPU 资源，但当你执行 I/O 操作时、远程 RPC 调用时，包括进行数据库操作时，这时候 CPU 就闲下来了，你可以利用多线程提高它的利用率。 经验公式如下 
$$
线程数 = CPU核数 * 期望 CPU 利用率 * 总时间/ CPU 计算时间

$$
总时间：
$$
总时间=CPU计算时间+等待时间
$$

### 任务调度线程池
在『任务调度线程池』功能加入之前，可以使用 `java.util.Timer` 来实现定时功能，
- Timer 的优点在于简单易用
- 但 由于所有任务都是由同一个线程来调度，因此所有任务都是串行执行的，同一时间只能有一个任务在执行，前一个 任务的延迟或异常都将会影响到之后的任务。

主要方法：
- `scheduleAtFixedRate`,定时任务执行，若任务本身时间过长（大于时间间隔），它会在任务执行完毕后不等间隔直接执行。
- `scheduleWithFixedDelay`,按固定时间间隔执行
拿异常：
- 方法1：使用try-catch手动捕获异常
- 方法2：使用future,出现异常时，get拿到的是异常信息
```java
ExecutorService pool = Executors.newFixedThreadPool(1); 
Future<Boolean> f = pool.submit(() -> { 
	log.debug("task1"); 
	int i = 1 / 0; 
	return true; 
}); 
log.debug("result:{}", f.get()); 
```
### Tomcat 线程池
Tomcat 在哪里用到了线程池呢?
![[Pasted image 20251025145511.png]]
- `LimitLatch` 用来限流，可以控制最大连接个数，类似 J.U.C 中的 Semaphore 后面再讲 
- Acceptor 只负责【接收新的 socket 连接】 
- `Poller` 只负责监听 socket channel 是否有【可读的 I/O 事件】 
- 一旦可读，封装一个任务对象（socketProcessor），提交给 Executor 线程池处理 
- Executor 线程池中的工作线程最终负责【处理请求】

Tomcat 线程池扩展了 ThreadPoolExecutor，行为稍有不同 :
如果总线程数达到 `maximumPoolSize `
- 这时不会立刻抛 `RejectedExecutionException` 异常 
- 而是再次尝试将任务放入队列，如果还失败，才抛出 `RejectedExecutionException` 异常
尝试只有一次，因为底层是用if实现的。
#### Tomcat与线程池相关配置
Connector 配置 :
![[Pasted image 20251025150436.png]]
Executor 线程配置 :
Executor会覆盖掉Connector的核心线程数和最大线程数，即Executor配置的优先级高
![[Pasted image 20251025150534.png]]
队列流程：
![[Pasted image 20251025150948.png]]
Tomcat是先用救急线程，救急线程不够了，再加入队列，和JDK的线程池实现`ThreadPoolExecutor`完全相反。
##  3. Fork/Join
Fork/Join 是 JDK 1.7 加入的新的线程池实现，它体现的是一种**分治思想**，适用于能够进行任务拆分的 **`cpu` 密集型运算** 
所谓的**任务拆分**，是将一个大任务拆分为算法上相同的小任务，直至不能拆分可以直接求解。跟递归相关的一些计 算，如归并排序、斐波那契数列、都可以用分治思想进行求解 
Fork/Join 在分治的基础上加入了多线程，可以把每个任务的分解和合并交给不同的线程来完成，进一步提升了运 算效率 
Fork/Join **默认会创建与 `cpu` 核心数大小相同的线程池**
# 2. JUC
## AQS 原理
全称是 AbstractQueuedSynchronizer，是阻塞式锁和相关的同步器工具的框架 
特点：
- 用 state 属性来表示资源的状态（分独占模式和共享模式），子类需要定义如何维护这个状态，控制如何获取 锁和释放锁 
	- `getState` - 获取 state 状态 
	- `setState` - 设置 state 状态 
	- `compareAndSetState - cas` 机制设置 state 状态 
	- 独占模式是只有一个线程能够访问资源，而共享模式可以允许多个线程访问资源 
- 提供了基于 FIFO 的等待队列，类似于 Monitor 的 `EntryList `
- 条件变量来实现等待、唤醒机制，支持多个条件变量，类似于 Monitor 的 `WaitSet`

子类要去继承AQS父类主要实现这样一些方法（默认抛出 `UnsupportedOperationException`）
- `tryAcquire、tryRelease、 tryAcquireShared、 tryReleaseShared、 isHeldExclusively`

获取锁的姿势 :
```java
// 如果获取锁失败 
if (!tryAcquire(arg)) { 
	// 入队, 可以选择阻塞当前线程 ,是使用park unpark 机制实现的
}
```
释放锁的姿势 :
```java
// 如果释放锁成功 
if (tryRelease(arg)) { 
	// 让阻塞线程恢复运行 
}
```
## `ReentrantLock` 原理
在JDK11中，acquire直接由AQS实现了
不可打断模式：被打断但是不是立即响应，只有获取到锁之后，通过返回的那个打断标记，才知道被打断过，才会自我再打断一次 有延迟
## 读写锁
当读操作远远高于写操作时，这时候使用 **读写锁** 让 读-读 可以并发，提高性能。
**注意事项** :
- 读锁不支持条件变量 
- 重入时升级不支持：即持有读锁的情况下去获取写锁，会导致获取写锁永久等待
- 重入时降级支持：即持有写锁的情况下去获取读锁
利用数据库表的不同对锁进行分类，优化读写锁。
读读节点并发的原因：只要你是share的节点，那么它一唤醒就会把这些连着一串的share节点都唤醒，直到遇到一个独占节点，它才停止唤醒。
## `StampedLock`
该类自 JDK 8 加入，是为了  **进一步优化读性能**，它的特点是在使用读锁、写锁时都必须配合【戳】使用
加解读锁:
```java
long stamp = lock.readLock(); 
lock.unlockRead(stamp);
```
加解写锁:
```java
long stamp = lock.writeLock(); 
lock.unlockWrite(stamp);
```
乐观读:
`StampedLock` 支持 `tryOptimisticRead() `方法（乐观读），读取完毕后需要做一次 **戳校验** 如果校验通 过，表示这期间确实没有写操作，数据可以安全使用，如果校验没通过，需要重新获取读锁，保证数据安全。
```java
long stamp = lock.tryOptimisticRead(); 
// 验戳 
if(!lock.validate(stamp)){ 
	// 锁升级 
}
```
注意 
	`StampedLock` 不支持条件变量 (不支持wait，signal)
	`StampedLock` 不支持可重入
## Semaphore
信号量，用来  **限制**  能同时访问共享资源的**线程上限**.
```java
// 1. 创建 semaphore 对象  
Semaphore semaphore = new Semaphore(3);  
// 2. 10个线程同时运行  
for(int i = 0;i< 10;i++){  
    new Thread(() -> {  
        // 3. 获取许可  
        try {  
            semaphore.acquire();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        try {  
            log.debug("running...");  
            Thread.sleep(1000);  
            log.debug("end...");  
        } catch (InterruptedException e) {  
            throw new RuntimeException(e);  
        } finally {  
            // 4. 释放许可  
            semaphore.release();  
        }  
    }).start();
}
```
- **使用 Semaphore 限流**，在访问高峰期时，让请求线程阻塞，高峰期过去再释放许可，当然它只适合限制单机 线程数量，并且仅是限制线程数，而不是限制资源数（例如连接数，请对比 `Tomcat LimitLatch` 的实现） 
- 用 Semaphore **实现简单连接池**，对比『享元模式』下的实现（用wait notify），性能和可读性显然更好， 注意下面的实现中线程数和数据库连接数是相等的
- 当资源数和线程数一致的时候，用Semaphore去限流比较合适
## `CountdownLatch`
用来进行线程同步协作，等待所有线程完成倒计时。 
其中构造参数用来初始化等待计数值，await() 用来等待计数归零，countDown() 用来让计数减一

最好使用JUC提供的高级API完成同步功能。
使用场景：多玩家任务进度加载。多线程远程调用(远程拿链接资源 , 下载资源等).
多线程远程调用,单独说一下：
```java
RestTemplate restTemplate = new RestTemplate();
log.debug("begin");
ExecutorService service = Executors.newCachedThreadPool();
CountDownLatch latch = new CountDownLatch(4);
Future<Map<String, Object>> f1 = service.submit(() -> {
    Map<String, Object> r = restTemplate.getForObject("http://localhost:8080/order/{1}", Map.class, 1);
    return r;
});
Future<Map<String, Object>> f2 = service.submit(() -> {
    Map<String, Object> r = restTemplate.getForObject("http://localhost:8080/product/{1}", Map.class, 1);
    return r;
});

System.out.println(f1.get());
System.out.println(f2.get());
log.debug("执行完毕"); 
service.shutdown();
```
如上述代码,可以用Future拿到远程调用的结果
不适用场景:
	单个任务需要多次执行,这样的话`CountdownLatch`会频繁创建(不可重用),资源利用率下降
### `CyclicBarrier`
循环栅栏，用来进行线程协作，等待线程满足某个计数。
构造时设置『计数个数』，每个线程执 行到某个需要“同步”的时刻调用 await() 方法进行等待，当等待的线程数满足『计数个数』时，继续执行
注意：
	newFixedThreadPool线程数 要和 CyclicBarrier的数量保持一致，不然会出现意料之外的结果
## 线程安全集合类
线程安全集合类可以分为三大类： 
1. 遗留的线程安全集合如 `Hashtable` ， Vector 
2. 使用 Collections 装饰的线程安全集合，如： 
	`Collections.synchronizedCollection `
	`Collections.synchronizedList `
	`Collections.synchronizedMap` 
	`Collections.synchronizedSet `
	`Collections.synchronizedNavigableMap `
	`Collections.synchronizedNavigableSet` 
	`Collections.synchronizedSortedMap `
	`Collections.synchronizedSortedSet`
3. java.util.concurrent.*
![[Pasted image 20251026091221.png]]
重点介绍 java.util.concurrent.* 下的线程安全集合类，可以发现它们有规律，里面包含三类关键词： `Blocking、CopyOnWrite、Concurrent` 

- Blocking 大部分实现基于锁，并提供用来阻塞的方法 
- `CopyOnWrite` (写时复制一份)之类容器修改开销相对较重 
- Concurrent 类型的容器 
	内部很多操作使用 `cas` 优化，一般可以提供较高吞吐量
	弱一致性 
	- 遍历时弱一致性，例如，当利用迭代器遍历时，如果容器发生修改，迭代器仍然可以继续进行遍 历，这时内容是旧的 
	- 求大小弱一致性，size 操作未必是 100% 准确 
	- 读取弱一致性
```test
遍历时如果发生了修改，对于非安全容器来讲，使用 fail-fast 机制也就是让遍历立刻失败，抛出 ConcurrentModificationException，不再继续遍历
```
遍历失败后仍然可以运行的称之为 fail-safe.
### `ConcurrentHashMap`
累加器LongAdder,
`computeIfAbsent()`保证get和put的原子性

HashMap:在JDK8中后加入的元素会放在链表的尾部,JDK7则是头部(七上八下),数组元素超过数组长度的3/4时,会进行一次扩容(创建一个新的),并把原来数组中的值迁移到新的数组
为什么数组扩容后，链表长度会减半？
因为我们确定桶位置是数组长度与hash进行&操作，长度扩为2倍，桶位置正好会改为2倍
#### 死链
多线程环境下进行扩容时就会造成一种并发死链问题.
```IDEA
下面 Customize Data Views取消勾选Enable alternative view for collections classes就不用老view as了
```
- JDK 8 虽然将扩容算法做了调整，不再将元素加入链表头（而是保持与扩容前一样的顺序），但仍不意味着能 够在多线程环境下能够安全扩容，还会出现其它问题（如扩容丢数据）
## `LinkedBlockingQueue`
去看PDF吧
主要列举 `LinkedBlockingQueue` 与 `ArrayBlockingQueue` 的性能比较 
- Linked 支持有界，Array 强制有界 
- Linked 实现是链表，Array 实现是数组 
- Linked 是懒惰的，而 Array 需要提前初始化 Node 数组 
- Linked 每次入队会生成新 Node，而 Array 的 Node 是提前创建好的 
- Linked 两把锁，Array 一把锁
### `ConcurrentLinkedQueue`
`ConcurrentLinkedQueue` 的设计与 `LinkedBlockingQueue` 非常像，也是 
- 两把【锁】，同一时刻，可以允许两个线程同时（一个生产者与一个消费者）执行 
- dummy 节点的引入让两把【锁】将来锁住的是不同对象，避免竞争 
- 只是这【锁】使用了 `cas` 来实现 

事实上，ConcurrentLinkedQueue 应用还是非常广泛的 例如 :
之前讲的 Tomcat 的 Connector 结构时，Acceptor 作为生产者向 `Poller` 消费者传递事件信息时，正是采用了 `ConcurrentLinkedQueue` 将 `SocketChannel` 给 `Poller` 使用
![[Pasted image 20251026131220.png]]
### `CopyOnWriteArrayList`
`CopyOnWriteArraySet` 是`CopyOnWriteArrayList`的马甲 
底层实现采用了 **写入时拷贝** 的思想，增删改操作会将底层数组拷贝一份，更 改操作在新数组上执行，这时不影响其它线程的  **并发读，读写分离**

适合『读多写少』的应用场景
#### 问题：get 弱一致性
`CopyOnWrite*` 和 `Concurrent*` 这两种开头的都有弱一致性问题
#### 迭代器弱一致性
不要觉得弱一致性就不好 
- 数据库的 MVCC 都是弱一致性的表现 
- 并发高和一致性是矛盾的，需要权衡