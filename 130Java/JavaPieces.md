问 Java表达式1/0和1.0/0.0的值是什么？ 
答 第一个表达式会产生一个运行时除以零异常（它会终止程序，因为这个值是未定义的）；第二个表 达式的值是Infinity（无穷大）。

问 负数的除法和余数的结果是什么？ 
答 表达式a/b的商会向0取整；a % b的余数的定义是(a/b)*b + a % b恒等于a。例如-14/3和 14/-3 的商都是-4，但-14 % 3是-2，而14 % -3是2。

new接口但是要实现这个接口的方法
![[Pasted image 20251012094939.png]]
## 布隆过滤器
利用hash函数。
是一种节省空间的概率数据结构，用于回答这个元素在集合中吗？
	会给出肯定的否 和 可能的是的回答
其可以减少内存占用。
许多NoSQL的数据库使用布隆过滤器，来减少对不存在键的磁盘读取。
![[Pasted image 20251130093616.png]]
解决布隆过滤器误报问题的方法：

| 方法              | 原理                               | 优点            | 缺点                     |
| --------------- | -------------------------------- | ------------- | ---------------------- |
| **优化参数 (m, k)** | 根据期望的误报率和元素数量，计算最优的位数组大小和哈希函数个数。 | 最常用、最有效、成本可控。 | 需要提前预估数据规模。            |
| **使用优质哈希函数**    | 减少因哈希函数性能差导致的额外冲突。               | 提升效果明显，实现简单。  | 有选择成本。                 |
| **计数布隆过滤器**     | 用计数器代替比特位，支持删除。                  | 支持删除操作。       | 内存消耗增加 3-4 倍。          |
| **布谷鸟过滤器**      | 使用不同的算法和数据结构。                    | 支持删除，空间效率更高。  | 实现复杂，满载时性能下降。          |
| **替代数据结构**      | 使用完全精确的存储（如HashSet）。             | 零误报。          | 内存消耗巨大，失去布隆过滤器空间效率的优势。 |
误报出现情况：所以是可能的有，否定的无。
![[Pasted image 20251130094115.png]]


要运行包内的Java文件，必须输入父目录（在本例中为lab6）并使用完全规范的名称。
![[Pasted image 20251224184402.png]]
Javac编译后，可能会注意到
- IntelliJ中的一堆.class文件与你的代码在同一个文件夹中。
- 而之前没有找到这种文件，是因为IntelliJ将生成的.class文件存储在另一个文件夹中(通常称为out或target)

查找当前工作目录：
![[Pasted image 20251224191440.png]]
就是shell命令的pwd

路径是文件或目录的位置。有两种路径：绝对路径和相对路径。  

- 绝对路径是文件或目录相对于文件系统根目录的位置。  
	example .java的绝对路径为C:/Users/Michelle/example/ example .java （Windows）或/home/Michelle/example/ example .java （Mac/Linux）。注意，这些路径以根目录C:/ (Windows)和/ (Mac/Linux)开始。
	
- 相对路径是文件或目录相对于程序CWD的位置。
	1. 如果我在C:/Users/Michelle/example/ (Windows)或/home/Michelle/example/ (Mac/Linux)文件夹中，那么example .java的相对路径将只是example .java。  
	2. 如果我在C:/Users/Michelle/或/home/Michelle/，那么example .java的相对路径将是example/ example .java。
```shell
cd ../.. # 去父目录的父目录
```

`java.io.Serializable` 接口没有方法；它只是标记它的子类型，以便一些特殊的Java类在对象上执行I/O
- 序列化
```java
Model m = ....;
File outFile = new File(saveFileName);
try {
    ObjectOutputStream out =
        new ObjectOutputStream(new FileOutputStream(outFile));
    out.writeObject(m);
    out.close();
} catch (IOException excp) {
    ...
}
```
- 反序列化
```java
Model m;
File inFile = new File(saveFileName);
try {
    ObjectInputStream inp =
        new ObjectInputStream(new FileInputStream(inFile));
    m = (Model) inp.readObject();
    inp.close();
} catch (IOException | ClassNotFoundException excp) {
    ...
    m = null;
}
```
上面的写法很烦人，可以使用Util包下的简化工具：
![[Pasted image 20251224193211.png]]
