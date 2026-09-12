## 简介
JDBC 是java语言操作关系型数据库[[`sqlStart`]]的一套API接口。其它sql厂商只需要对其做一个实现类，便可以用一套代码操作不同的 关系型数据库。
全称：`（Java DataBase Connectivity`）Java 数据库连接
![[Pasted image 20250830001751.png]]
真正执行代码是驱动jar包中的实现类。
### 快速入门：
![[Pasted image 20250830002324.png]]
mysql的url是 ：    `jdbc:mysql://localhostIP:3306（端口）/mysql-db（数据库名称）`
## API详解
### `DriverManager`
- `DriverManager`(驱动管理类)作用：
1. 注册驱动
![[Pasted image 20250830145131.png]]
2. 获取数据库连接
![[Pasted image 20250830145241.png]]
IP地址可以写域名，本地域名是localhost
若`jdbc:mysql://localhost:3306/db1_jdbc?useSSL=false在`?useSSL=false后面还要加参数只需要用  &  隔开就行
### `Connection`
Connection(数据库连接对象)作用：
1. 获取执行 SQL 的对象
- 普通执行SQL对象<重点>
    - `Statement createStatement()`
- 预编译SQL的执行SQL对象：防止SQL注入<重点>
    - `PreparedStatement preparedStatement(sql)`
- 执行存储过程的对象
    - `CallableStatement prepareCall(sql)`

2. 管理事务
![[Pasted image 20250830151739.png]]
java中通过异常处理的方式进行事务管理。
### `Statement`
Statement作用：
1. 执行SQL语句
`int executeUpdate(sql)`：执行DML、DDL语句
➢ 返回值：(1) DML语句影响的行数 (2) DDL语句执行后，执行成功也可能返回 0

`ResultSet executeQuery(sql)`：执行DQL 语句
➢ 返回值：ResultSet 结果集对象
### `ResultSet`
`ResultSet`(结果集对象)作用：封装了DQL查询语句的结果

`ResultSet stmt.executeQuery(sql)`：执行DQL语句，返回ResultSet对象
获取查询结果
`boolean next()`: (1) 将光标从当前位置向前移动一行 (2) 判断当前行是否为有效行
➢ 返回值：
    • true：有效行，当前行有数据
    • false：无效行，当前行没有数据
`xxx getXxx`(参数)：获取数据
➢ xxx：数据类型，如：`int getInt(参数)；String getString(参数)`
➢ 参数：
    • int：列的编号，从1开始
    • String：列的名称

使用方法：
![[Pasted image 20250830155632.png]]
### 数据操作的主要原理
![[Pasted image 20250830161547.png]]
将数据库的每行封装为java对象，将其塞到ArrayList中返回，再给它拆开就行。
### `PreparedStatement`
继承自Statement。
作用： 预编译SQL语句并执行，即预防SQL注入问题。
底层是凭借转义语句中的  单引号  ，解决sql注入的问题（将敏感字符转义）。
sout掉sql语句即可得到sql注入时可利用的语句。

使用过程：
![[Pasted image 20250830222027.png]]
### `PreparedStatement`原理
![[Pasted image 20250830230246.png]]
预编译功能：将蓝色框代码执行后，MySQL就已经把  检查SQL语法  和  编译SQL 已经处理过了，只待执行。
预编译只会执行一次。
PreparedStatement的预编译功能默认是关闭的，若要开启，需在url的  ？后面加上`useServerPrepStmts=true`这行参数
在my.ini文件下，添加配置文件，文件目录可以更改（图中以 D:\ 开头的）
配置日志就是为了让你看懂原理，真正使用的时候可以不配置
### 传统的JDBC的问题
![[Pasted image 20250906135513.png]]
硬编码：配置写死在Java文件中
繁琐：要知道数据库所有的栏目值
资源浪费、性能降低：频繁的申请和释放资源。
## 数据库连接池
- 数据库连接池是个容器，负责分配、管理数据库连接(Connection)
- 它允许应用程序重复使用一个现有的数据库连接，而不是再重新建立一个；
- 释放空闲时间超过最大空闲时间的数据库连接，来避免因为没有释放数据库连接而引起的数据库连接泄漏
所有的数据库连接池都必须实现datasource接口
![[Pasted image 20250830234230.png]]
- 好处：
    - 资源重用
    - 提升系统响应速度
    - 避免数据库连接泄漏
### 数据库连接池实现
![[Pasted image 20250830234733.png]]
![[Pasted image 20250906140308.png]]
#### Druid使用
可以用maven配置。
使用步骤：
	1. 导入jar包 druid-1.1.12.jar
	2. 定义配置文件
	3. 加载配置文件
	4. 获取数据库连接池对象
	5. 获取连接
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.21</version>
</dependency>
```
![[Pasted image 20250830235138.png]]
### 配置路径
```java
//直接在psvm中执行这串代码，会返回当前路径
System.out.println(System.getProperty("user.dir"));
```
返回内容点开后，在那个目录下找到想找文件的路径，复制并去掉代码返回的内容。
