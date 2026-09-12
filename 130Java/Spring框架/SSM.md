# Spring Framework系统架构
系统架构讲究上层依赖下层
![[Pasted image 20250907180800.png]]
Test构不成学习的难度，SpringWeb后面再学。
# 核心容器Bean
实现：
- `IoC（Inversion of Control）`控制反转
    - 使用对象时，由主动new产生对象转换为由外部提供对象，此过程中对象创建控制权由程序转移到外部，此思想称为控制反转
- Spring技术对IoC思想进行了实现
    - Spring提供了一个容器，称为IoC容器，用来充当IoC思想中的“外部”
    - IoC容器负责对象的创建、初始化等一系列工作，被创建或被管理的对象在IoC容器中统称为Bean
- `DI（Dependency Injection）`依赖注入
    - 在容器中建立bean与bean之间的依赖关系的整个过程，称为依赖注入
![[Pasted image 20250907181900.png]]
最终目的与效果：
- 目标：充分解耦
    - 使用IoC容器管理bean（IoC）
    - 在IoC容器内将有依赖关系的bean进行关系绑定（DI）
- 最终效果
    - 使用对象时不仅可以直接从IoC容器中获取，并且获取到的bean已经绑定了所有的依赖关系
## IoC实现
先想明白以下问题：
1. **管理什么？**
    - Service和Dao对象。
2. **如何将被管理的对象告知IoC容器？**
    - 通过配置文件。
3. **如何获取到IoC容器？**
    - 通过Ioc接口调用其实现类。
4. **如何从容器中获取bean？**
    - 使用`getBean`方法。
5. **使用Spring导入哪些坐标？**
    - 导入pom.xml文件。
![[Pasted image 20250907200945.png]]
### XML版实现
1. 导入Spring坐标(maven依赖)
2. 定义Spring管理的类（实现接口）和接口
3. 创建Spring配置文件，配置对应类作为Spring管理的Bean（Bean定义时id属性不能重复，最好是相应类的名字 且首字符小写）
4. 初始化IoC容器（Spring核心容器），通过容器获取Bean
## DI实现
1. **基于IoC管理bean**
2. **Service中使用new形式创建的Dao对象是否保留？**
    - 否。（若new了实现类，耦合度必然高）
3. **Service中需要的Dao对象如何进入到Service中？**
    - 通过依赖注入。（提供一个方法，让Ioc容器往里面传递Bean对象/Dao对象）
4. **Service与Dao间的关系如何描述？**
    - 通过配置。
### XML版实现
1. 删除使用new的形式创建对象的代码。
2. 提供依赖对象的setter方法。（设置去掉new后对象的属性值）
3. 配置service与dan之间的关系。
```xml
<property name="去掉new后的属性名" ref="Bean的id"/>
//String str [= new string]; 在这里name就是str (找到注入对象的名称)
```
## Bean的配置
![[Pasted image 20250907202142.png]]
### Bean别名设置
用name属性就行
![[Pasted image 20250907202445.png]]
注意事项:
- 获取bean无论是通过id还是name获取，如果无法获取到，将抛出异常`NoSuchBeanDefinitionException`
`NoSuchBeanDefinitionException: No bean named 'bookServiceImpl' available`
### Bean作用范围
用scope属性
![[Pasted image 20250907202857.png]]
- 为什么bean默认为单例？
单例模式有助于节省资源，因为不需要为每个请求创建新的实例，同时也便于管理对象之间的依赖关系。
- 适合交给容器进行管理的bean
    - 表现层对象（**servlet**）
    - 业务层对象（**Service**）
    - 数据层对象（**dao**）
    - 工具对象（**utils**）
    指的是应用程序中通用的工具类或组件，它们可以被应用程序的其他部分重用。
- 不适合交给容器进行管理的bean
    - 封装实体的域对象（entity层）
    通常是应用程序中的实体对象，如用户、订单等，这些对象的创建和销毁通常是由业务逻辑控制的，而不是由Spring容器控制。
## Bean的实例化
### 构造方法
spring创建bean的构造方法时，会隐式调用一个无参的构造方法。
![[Pasted image 20250909113840.png]]
- 无参构造方法如果不存在，将抛出异常BeanCreationException
### 静态工厂实例化Bean
之前的方法，不要new对象用工厂造对象，可以实现代码解耦。
![[Pasted image 20250909114218.png]]
class执行工厂类，并用factory-method指明工厂类的那个方法可以造对象，否则造的是工厂Bean。
### 实例工厂实例化Bean
![[Pasted image 20250909114711.png]]
先造工厂的实例Bean，再用实例Bean调用造Bean 类。
#### 使用factorybean实例化bean
优化上一种实例工厂实例化Bean，提高代码的可读性
![[Pasted image 20250909114908.png]]
这种方法很重要。
默认情况下用的是单例Bean，若要非单例，则需要在工厂Bean中将下面方法的值改为false。
```java
public boolean isSingleton() {
    return false;
}
```
## Bean的生命周期
- 初始化容器
    1. 创建对象（内存分配）
    2. 执行构造方法
    3. 执行属性注入（set操作）
    4. 执行bean初始化方法
- 使用bean
    1. 执行业务操作
- 关闭/销毁容器
    1. 执行bean销毁方法
### Bean销毁时机
在配置文件指名init-method和destory-method属性，并赋给这两个属性方法（写在配置class属性的类中）。
但是destroy只会在容器关闭后产生，可以使用容器类.close方法强制关闭。也可以用容器类.registerShutdownHook() - 注册容器钩子这个方法，提示JVM在关闭之前，先关闭掉这个容器
IOC容器根据配置文件创建、初始化bean对象

可以实现initializingbean，disposableBean接口，覆盖执行service的afterPropertiesSet()和destroy()方法
![[Pasted image 20250909121728.png]]
## 依赖注入方式
- 思考：向一个类中传递数据的方式有几种？
    - 普通方法（set方法）
    - 构造方法
- 思考：依赖注入描述了在容器中建立bean与bean之间依赖关系的过程，如果bean运行需要的是数字或字符串呢？
    - 引用类型
    - 简单类型（基本数据类型与String）
- 依赖注入方式
    - setter注入：简单类型、引用类型
    - 构造器注入：简单类型、引用类型
### Setter方法注入
![[Pasted image 20250909122347.png]]
定义Bean后，
在Bean标签（含闭标签）中，使用property标签。
ref 用来做引用Bean；给值用value。
### 构造器注入
定义Bean后，
在Bean标签（含闭标签）中，使用constructor-arg标签。
ref 用来做引用Bean；给值用value。

使用参数名（name，与源文件形参紧密耦合）、数据类型（type，用类型数据就不能用）、利用相对位置定位注入对象（0，1.......）。
### 依赖注入方式选择
1. **强制依赖使用构造器进行**，使用setter注入有概率不进行注入导致null对象出现
2. 可选依赖使用setter注入进行，灵活性强
3. **Spring框架倡导使用构造器**，第三方框架内部大多数采用构造器注入的形式进行数据初始化，相对严谨
4. 如果有必要可以两者同时使用，使用构造器注入完成强制依赖的注入，使用setter注入完成可选依赖的注入
5. 实际开发过程中还要根据实际情况分析，如果受控对象没有提供setter方法就必须使用构造器注入
6. **自己开发的模块推荐使用setter注入**
### 自动装配
- IoC容器根据bean所依赖的资源在容器中自动查找并注入到bean中的过程称为自动装配
- 自动装配方式
    - 按类型（常用，连名字都可以不取）
    - 按名称：用set方法的名字 和 配置文件定义Bean的名字装配（耦合）。
    - 按构造方法
    - 不启用自动装配
![[Pasted image 20250909124317.png]]
注意：
- 自动装配用于引用类型依赖注入，不能对简单类型进行操作
- 使用按类型装配时（byType）必须保障容器中相同类型的bean唯一，推荐使用
- 使用按名称装配时（byName）必须保障容器中具有指定名称的bean，因变量名与配置耦合，不推荐使用
- 自动装配优先级低于setter注入与构造器注入，同时出现时自动装配配置失效
### 集合注入
注入数组对象：
```xml
<property name="array">
    <array>
        <value>100</value>
        <value>200</value>
        <value>300</value>
    </array>
</property>
```
注入List对象（重点）:
```xml
<property name="list">
    <list>
        <value>itcast</value>
        <value>itheima</value>
        <value>boxuegu</value>
    </list>
</property>
```
注入Map对象（重点）:
```xml
<property name="map">
    <map>
        <entry key="country" value="china"/>
        <entry key="province" value="henan"/>
        <entry key="city" value="kaifeng"/>
    </map>
</property>
```
注入Properties对象:
```xml
<property name="properties">
    <props>
        <prop key="country">china</prop>
        <prop key="province">henan</prop>
        <prop key="city">kaifeng</prop>
    </props>
</property>
```
## 加载properties文件

1. 在 xml 中开启 context 命名空间
2. 使用 Context 在 xml 中加载 properties 文件
	`<context:property-placeholder location=""/>`
3. ${name} 语法获取属性值,读取properties文件中的属性值
![[Pasted image 20250909125823.png]]
注意： 有时候properties中命名时会恰巧和系统属性命名一样，因为系统文件优先级 > propertise文件优先级。
```xml
- 不加载系统属性
    `<context:property-placeholder location="jdbc.properties" system-properties-mode="NEVER"/>`
- 加载多个properties文件
    `<context:property-placeholder location="jdbc.properties,msg.properties"/>`
- 加载所有properties文件 
    `<context:property-placeholder location="*.properties"/>`  
- 加载properties文件标准格式
    `<context:property-placeholder location="classpath:*.properties"/>` 
- 从类路径或jar包中搜索并加载properties文件
    `<context:property-placeholder location="classpath*:*.properties"/>`
```
## 容器
获取：
 ```java
 - 方式一：类路径加载配置文件
    `ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");`
- 方式二：文件路径加载配置文件
    `ApplicationContext ctx = new FileSystemXmlApplicationContext("D:\\applicationContext.xml");`
- 加载多个配置文件，按类型查找
    `ApplicationContext ctx = new ClassPathXmlApplicationContext("bean1.xml", "bean2.xml");`
 ```
 ApplicationContext的顶层接口是BeanFactory，BeanFactory也可以创建接口（老东西了，有缺陷被ApplicationContext修复了）
 Be按Factory是所有容器类的顶层接口。
 beanfactory创建完毕后，所有的bean均为延迟加载
![[Pasted image 20250909203548.png]]
记住这种继承接口不断向下丰富功能的思想。
## 总结

- BeanFactory是IoC容器的顶层接口，初始化BeanFactory对象时，加载的bean延迟加载
- ApplicationContext接口是Spring容器的核心接口，初始化时bean立即加载
- ApplicationContext接口提供基础的bean操作相关方法，通过其他接口扩展其功能
- ApplicationContext接口常用初始化类
    - `ClassPathXmlApplicationContext`
    - `FileSystemXmlApplicationContext`
### Bean 相关
![[Pasted image 20250909204540.png]]
### 依赖注入相关：
![[Pasted image 20250909204451.png]]
# 注解开发
## Bean的管理
- 使用@Component定义bean
```java
@Component("bookDao")
public class BookDaoImpl implements BookDao {
}

@Component
public class BookServiceImpl implements BookService {
}
```
- 在核心配置文件中通过组件扫描加载bean
```xml
<context:component-scan base-package="com.itheima"/>
```
Spring提供@Component注解的三个衍生注解
- @Controller：用于表现层bean定义
- @Service：用于业务层bean定义
- @Repository：用于数据层bean定义
```java
@Repository("bookDao")
public class BookDaoImpl implements BookDao {
}

@Service
public class BookServiceImpl implements BookService {
}
```
#### 配置文件变更为类
![[Pasted image 20250909210753.png]]
#### 读取配置类
![[Pasted image 20250909211003.png]]
#### bean的作用范围
![[Pasted image 20250909211514.png]]
#### Bean的生命周期
![[Pasted image 20250909211607.png]]
### 依赖注入(自动装配)
1. 使用@Autowired注解开启自动装配模式（按类型）
```java
@Service
public class BookServiceImpl implements BookService {
    @Autowired
    private BookDao bookDao;
    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```
- 注意：自动装配基于反射设计创建对象并暴力反射对应属性为私有属性初始化数据，因此  **无需提供setter方法**
    
- 注意：自动装配建议使  **用无参构造方法创建对象（默认）**  ，如果不提供对应构造方法，请提供唯一的构造方法

2. 使用@Qualifier注解开启指定名称装配bean
```java
@Service
public class BookServiceImpl implements BookService {
    @Autowired
    @Qualifier("bookDao")
    private BookDao bookDao;
}
```
- 注意：@Qualifier注解无法单独使用，必须配合@Autowired注解使用

3. 使用@Value实现简单类型注入
```java
@Repository("bookDao")
public class BookDaoImpl implements BookDao {
    @Value("100")
    private String connectionNum;
}
```

4. 加载properties文件
使用@PropertySource注解加载properties文件
```java
@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:jdbc.properties")
public class SpringConfig {
}
```
- 注意：路径仅支持单一文件配置，多文件请使用数组格式配置，不允许使用通配符*
### 管理第三方Bean
- 使用@Bean配置第三方Bean
```java
@Configuration
public class SpringConfig {

    @Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("root");
        return ds;
    }
}
```
- 将独立的配置类加入核心配置
方式一：导入式
```java
public class JdbcConfig {
    @Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        // 相关配置
        return ds;
    }
}
```
 使用@Import注解手动加入配置类到核心配置，此注解只能添加一次，多个数据请用数组格式
 ```java
@Configuration
//写多个配置类时，写成数组{A.class,B.class}
@Import(JdbcConfig.class) 
public class SpringConfig {
}
 ```
方式二：扫描式
![[Pasted image 20250910103520.png]]
#### 注入第三方Bean
- 简单类型依赖注入
```java
public class JdbcConfig {
    @Value("com.mysql.jdbc.Driver")
    private String driver;
    @Value("jdbc:mysql://localhost:3306/spring_db")
    private String url;
    @Value("root")
    private String userName;
    @Value("root")
    private String password;
    @Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(userName);
        return ds;
    }
}
```
- 引用类型依赖注入
```java
@Bean
public DataSource dataSource(BookService bookService){
    System.out.println(bookService);
    DruidDataSource ds = new DruidDataSource();
    // 属性设置
    return ds;
}
```
引用类型注入只需要为bean定义方法设置形参即可，容器会根据类型自动装配对象
### 小结
XML配置对比注解配置
![[Pasted image 20250910104611.png]]
## 整合Mybatis
### `MyBatis`独立开发过程
- MyBatis程序核心对象分析
![[Pasted image 20250910105149.png]]
- 整合MyBatis
![[Pasted image 20250910105304.png]]
上三块本质是初始化SqlSessionFactory，MyBatis本质上是在管理SqlSessionFactory。
### Spring整合
- SqlSessionFactory转化成一个Bean
![[Pasted image 20250910110650.png]]
数据源（dataSource）是通过注入的方式加进来的
- 映射配置
![[Pasted image 20250910110750.png]]
## 整合JUnit
```java
//类运行器要指定正确，一般不会变
@RunWith(SpringJUnit4ClassRunner.class)
//指定Spirng配置
@ContextConfiguration(classes = SpringConfig.class)
public class BookServiceTest {
		
	//想配谁，就把谁配置为一个属性，自动装配进去就行
    @Autowired
    private BookService bookService;

    @Test
    public void testSave(){
        bookService.save();
    }
}
```
## AOP
面向切面编程：
![[Pasted image 20250910112517.png]]
思路分析：
1. 导入坐标（pom.xml）<导入aspect包和  aop包（context包包含）>
![[Pasted image 20250910113752.png]]
2. 制作连接点方法（原始操作，Dao接口与实现类）
![[Pasted image 20250910113819.png]]
3. 制作共性功能（通知类与通知）
![[Pasted image 20250910113850.png]]
4. 定义切入点
![[Pasted image 20250910113917.png]]
5. 绑定切入点与通知关系（切面），并指定通知添加到原始连接点的具体执行位置
![[Pasted image 20250910114125.png]]
6. 定义通知类受Spring容器管理，并定义为当前类为切面类
![[Pasted image 20250910114430.png]]
7. 开启Spring对AOP注解驱动支持
![[Pasted image 20250910114935.png]]
### AOP工作流程
1. Spring容器启动
2. 读取所有切面配置中的切入点
3. 初始化bean，判定bean对应的类中的方法是否匹配到任意切入点
    - 匹配失败，创建对象
    - 匹配成功，创建原始对象（目标对象）的代理对象
4. 获取bean执行方法
    - 获取bean，调用方法并执行，完成操作
    - 获取的bean是代理对象时，根据代理对象的运行模式运行原始方法与增强的内容，完成操作
目标对象（Target）：代理对象所代理的对象，也叫原始对象。
代理（Proxy）：目标对象无法直接完成工作，需要对其进行功能回填，通过原始对象的代理对象实现。
### AOP切入点表达式
![[Pasted image 20250910120147.png]]
```java
切入点表达式标准格式：动作关键字（访问修饰符 返回值 包名.类/接口名.方法名（参数）异常名）
execution（public User com.itheima.service.UserService.findById(int)
```
- 动作关键字：描述切入点的行为动作，例如execution表示执行到指定切入点
- 访问修饰符：public，private等，可以省略
- 返回值
- 包名
- 类/接口名
- 方法名
- 参数 
- **异常名：方法定义中抛出指定异常，可以省略**

可以使用通配符描述切入点，快速描述：
![[Pasted image 20250910121019.png]]
#### 书写技巧
- 所有代码  **按照标准规范开发**  ，否则以下技巧全部失效
- 描述切入点通常描述接口，而不描述实现类（描述到实现类就耦合了）
- 访问控制修饰符针对接口开发均采用public描述   （**可省略访问控制修饰符描述**，就是这个public）
- 返回值类型对于增删改类使用  **精准类型**  加速匹配，对于查询类使用 * 通配快速描述
- **包名**  书写  **尽量不使用..匹配**，效率过低，常用*做单个包描述匹配，或精准匹配
- 接口名/类名书写名称与  **模块相关**  的  **采用 * 匹配**，例如UserService书写成_Service，绑定业务层接口名
- **方法名**  书写以  **动词**  进行  **精准匹配**，名词采用_匹配，例如getById书写成getBy_,selectAll书写成selectAll
- 参数规则较为复杂，根据业务方法灵活调整
- 通常  **不使用异常**  作为  **匹配**  规则
### AOP通知类型
五种：
1. 前置通知
![[Pasted image 20250910123213.png]]
2. 后置通知
![[Pasted image 20250910123248.png]]
3. 环绕通知（重点）
![[Pasted image 20250910123451.png]]
```txt
@Around注意事项

1. 环绕通知必须依赖形参 ProceedingJoinPoint 才能实现对原始方法的调用，进而实现原始方法调用前后同时添加通知
    
2. 通知中如果未使用ProceedingJoinPoint对原始方法进行调用将跳过原始方法的执行(可以利用这个特性做权限隔离)
    
3. 对原始方法的调用可以不接收返回值，通知方法  设置成void  即可，如果接收返回值，必须  设定为Object类型
    
4. 原始方法的返回值如果是void类型，通知方法的返回值类型可以设置成void，也可以设置成Object
    
5. 由于无法预知原始方法运行后是否会抛出异常，因此环绕通知方法必须抛出Throwable对象
```
4. 返回后通知（了解）
![[Pasted image 20250910123825.png]]
5. 抛出异常后通知（了解）
![[Pasted image 20250910123846.png]]
### AOP通知获取
获取参数：
![[Pasted image 20250910124526.png]]
获取返回值数据：
![[Pasted image 20250910124631.png]]
获取异常数据（了解）：
![[Pasted image 20250910124739.png]]
# Spring事务
事务开到业务层上的好处：可以将业务层方法包含的将数据层操作，放到同一个事务中让他们同成功同失败。
作用：在  数据层  或  业务层  同成功同失败。
![[Pasted image 20250911143022.png]]上面接口下面实现类。
## 步骤：
![[Pasted image 20250911143922.png]]
注意事项
Spring注解式事务通常添加在业务层接口中而不会添加到业务层实现类中，降低耦合。
注解式事务可以添加到业务方法上表示当前方法开启事务，也可以添加到接口上表示当前接口所有方法开启事务。
![[Pasted image 20250911144106.png]]
DataSourceTransactionManager上面的是接口，下面的是实现类。
MyBatis框架使用的事务是由JDBC提供的若要换的话，只能换实现类不能换接口。
![[Pasted image 20250911144516.png]]
## 事务角色
- 事务管理员：发起事务方，在Spring中通常指代业务层开启事务的方法
- 事务协调员：加入事务方，在Spring中通常指代数据层方法，也可以是业务层方法
![[Pasted image 20250911145039.png]]
通过相同的数据源进行管理的。
## 事务相关配置
点@Transactional注解进去即可看到这些属性设置。
![[Pasted image 20250911145312.png]]
只有遇到如下两种异常事务才会进行回滚：
- 第一种是Error系的
- 第二种是运行时异常。
若想遇到其它的异常也能回滚，需要在注解中加入rollbackFor属性并指定异常类。
例如：`@Transactional(rollbackFor = {IOException.class})`
try...finally...无论try中代码是否执行成功，都能执行finally中的语句。
### 事务的传播行为
- 事务传播行为：**事务协调员对事务管理员所携带事务的处理态度**
问题：
两个事务T和事务T2都加入了@Transactional事务，但我想让事务无论是否执行成功都执行finally中的语句（由于事务特性失败了）
![[Pasted image 20250911150506.png]]
解决方法：
![[Pasted image 20250911150727.png]]
常用参数如下图：
![[Pasted image 20250911151205.png]]
# `SpringMVC`
- SpringMVC技术与Servlet技术功能等同，均属于web层开发技术
目的：
1. 掌握基于SpringMVC获取请求参数与响应json数据操作
2. 熟练应用基于REST风格的请求路径设置与参数传递
3. 能够根据实际业务建立前后端开发通信协议并进行实现
4. 基于SSM整合技术开发任意业务模块功能
![[Pasted image 20250912102352.png]]
## 入门案例
@Controller是做springmvc开发的Bean的注解
步骤：
1. 使用SpringMVC技术需要先导入SpringMVC坐标和Servlet坐标
![[Pasted image 20250912103101.png]]
2. 创建SpringMVC的控制器类（等同于Servlet功能）
![[Pasted image 20250912103204.png]]
3. 初始化SpringMVC环境（属于Spring环境），所以要设定SpringMVC加载对应的bean
![[Pasted image 20250912103438.png]]
4. 初始化Servlet容器，加载SpringMVC环境，并设置SpringMVC技术处理的请求。
![[Pasted image 20250912103615.png]]
#### 两个新的注解
- 名称：@Controller
- 类型：类注解
- 位置：SpringMVC控制器类定义上方
- 作用：设定SpringMVC的核心控制器bean
- 范例：
```java
@Controller
    public class UserController {
    }
```
- 名称：@RequestMapping
- 类型：方法注解
- 位置：SpringMVC控制器方法定义上方
- 作用：设置当前控制器方法请求访问路径
- 范例：
 ```java
   @RequestMapping("/save")
    public void save(){
        System.out.println("user save ...");
    }
 ```
 - 名称：@ResponseBody
- 类型：方法注解
- 位置：SpringMVC控制器方法定义上方
- 作用：设置当前控制器方法响应内容为当前返回值，无需解析
- 范例：
```java
@RequestMapping("/save")
@ResponseBody
public String save(){
    System.out.println("user save ...");
    return "{'info':'springmvc'}";
}
```
#### SpringMVC入门程序开发总结（1+N）
- 一次性工作
    - 创建工程，设置服务器，加载工程
    - 导入坐标
    - 创建web容器启动类，加载SpringMVC配置，并设置SpringMVC请求拦截路径
    - SpringMVC核心配置类（设置配置类，扫描controller包，加载Controller控制器bean）
- 多次工作
    - 定义处理请求的控制器类
    - 定义处理请求的控制器方法，并配置映射路径（@RequestMapping）与返回json数据（`@ResponseBody`）

#### Servlet容器的配置类
- AbstractDispatcherServletInitializer类是SpringMVC提供的快速初始化Web3.0容器的抽象类
- AbstractDispatcherServletInitializer提供三个接口方法供用户实现

1. `createServletApplicationContext()`方法，创建Servlet容器时，加载SpringMVC对应的bean并放入WebApplicationContext对象范围中，而WebApplicationContext的作用范围为ServletContext范围，即整个web容器范围
```java
protected WebApplicationContext createServletApplicationContext() {
    AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
    ctx.register(SpringMvcConfig.class);
    return ctx;
}
```
2.  `getServletMappings()`方法，设定SpringMVC对应的请求映射路径，设置为“/”表示拦截所有请求，任意请求都将转入到SpringMVC进行处理(固定格式)
```java
protected String[] getServletMappings() {
    return new String[]{"/"};
}
```
3. `createRootApplicationContext()`方法，如果创建Servlet容器时需要加载非SpringMVC对应的bean，使用当前方法进行，使用方式同createServletApplicationContext()
```java
protected WebApplicationContext createRootApplicationContext() {
    return null;
}
```
### 入门案例工作流程分析
所有的SpringMVC的映射是放在一起统一管理的，并不是放在每个bean中管理。

- 启动服务器初始化过程
    1. 服务器启动，执行ServletContainersInitConfig类，初始化web容器
    2. 执行createServletApplicationContext方法，创建了WebApplicationContext对象
    3. 加载SpringMvcConfig
    4. 执行@ComponentScan加载对应的bean
    5. 加载UserController，每个@RequestMapping的名称对应一个具体的方法
    6. 执行getServletMappings方法，定义所有的请求都通过SpringMVC
- 单次请求过程
    1. 发送请求localhost/save
    2. web容器发现所有请求都经过SpringMVC，将请求交给SpringMVC处理
    3. 解析请求路径/save
    4. 由/save匹配执行对应的方法save()
    5. 执行save()
    6. 检测到有@ResponseBody直接将save()方法的返回值作为响应体返回给请求方
### bean加载控制
因为功能不同，如何避免Spring错误的加载到SpringMVC的bean？
回答：
- SpringMVC相关bean加载控制
    - SpringMVC加载的bean对应的包均在com.itheima.controller包内

- Spring相关bean加载控制
    - 方式一：Spring加载的bean设定扫描范围为com.itheima，排除掉controller包内的bean
![[Pasted image 20250912111802.png]]
    - 方式二：Spring加载的bean设定扫描范围为精准范围，例如service包、dao包等
![[Pasted image 20250912111011.png]]
### 简化开发
将入门案例中的第4步所含的内容可改为以下内容：
![[Pasted image 20250912111150.png]]
## 请求与响应
### 设置请求映射路径
团队多人开发，每人设置不同的请求路径，冲突问题如何解决？
	设置模块名作为请求路径前缀
![[Pasted image 20250912112747.png]]
#### 请求方式
代码上面的那个页面是Postman
- Get请求传递参数
![[Pasted image 20250912113148.png]]
- Post请求
![[Pasted image 20250912113211.png]]
以上的只能处理英文Post请求，中文会乱码。
##### Post请求中文乱码的处理：
![[Pasted image 20250912113522.png]]
在web容器类之中加上这个东西，也就是  入门案例章节的简化开发  那个类中
#### 参数传递
就记住一句话：默认的名称就要对上，对不上的话用@RequestParam注解给它对应上就可以了去看[[Java Web#参数的请求和封装]]

- 普通参数：请求参数名与形参变量名相同直接传就行，不同则用@RequestParam注释绑定参数关系
![[Pasted image 20250912114641.png]]
@RequestParam：
![[Pasted image 20250912114825.png]]
- Pojo参数：请求参数名和参数对象属性名相同，定义Pojo类型形参即可接收参数
![[Pasted image 20250912115017.png]]
- 嵌套的Pojo：请求参数名与形参对象属性名相同，按照对象层级结构关系即可嵌套Pojo属性参数
![[Pasted image 20250912115205.png]]
![[Pasted image 20250912115234.png]]
- 数组参数：请求参数名与形参对象属性名相同且请求参数为多个，定义数组类型形参即可接收参数。
![[Pasted image 20250912115401.png]]
- 集合保存普通参数：请求参数名与形参集合对象名相同且请求参数为多个，@RequestParam绑定参数关系
![[Pasted image 20250912115454.png]]
#### 参数传递（传递JSON数据）
1. 添加依赖
![[Pasted image 20250912120501.png]]
2. 设置发送json数据（请求body中添加json数据）
![[Pasted image 20250912120552.png]]
3. 开启自动转换json数据支持
![[Pasted image 20250912120640.png]]
注意：
@EnableWebMvc注解功能强大，该注解整合了多个功能，此处仅使用其中一部分功能，即json数据进行自动类型转换
4. 设置接收json数据集
![[Pasted image 20250912120819.png]]
- 区别
    - @RequestParam用于接收URL地址传参，表单传参`【application/x-www-form-urlencoded】`
        
    - @RequestBody用于接收json数据`【application/json】`
- 应用
    - 后期开发中，发送json格式数据为主，@RequestBody应用较广
        
    - 如果发送非json格式数据，选用@RequestParam接收请求参数
#### 新学注解：
![[Pasted image 20250912120948.png]]
![[Pasted image 20250912121003.png]]
#### 参数传递（日期类型参数传递）
他有一个默认的格式，当这个格式不适用的时候就要用@DateTineFormat()设定日期的格式,括号中写日期的格式。
![[Pasted image 20250913090713.png]]
@DateTineFormat()
![[Pasted image 20250913090822.png]]
Converter接口，它的实现类用于参数之间的转换（有默认的转换规则，如果发现转换不了把这个打开@EnableWebMvc）。

- 请求参数年龄数据（String→Integer）
- 日期格式转换（String → Date）
- @EnableWebMvc功能之一：根据类型匹配对应的类型转换器
### 响应
之前返回的都不是JSON数据而是字符串。
对象转JSON数据是jackson-databind这个依赖帮忙转的，而不是Spring。
- 响应页面
```java
@RequestMapping("/toPage")
public String toPage(){
    return "page.jsp";
}
```
- 响应文本数据
```java
@RequestMapping("/toText")
@ResponseBody
public String toText(){
    return "response text";
}
```
- 响应JSON数据（对象转JSON）
```java
@RequestMapping("/toJsonPOJO")
@ResponseBody
public User toJsonPOJO(){
    User user = new User();
    user.setName("赵云");
    user.setAge(41);
    return user;
}
```
- 响应JSON数据（对象集合转JSON）
```java
@RequestMapping("/toJsonList")
@ResponseBody
public List<User> toJsonList(){
    User user1 = new User();
    user1.setName("赵云");
    user1.setAge(41);
    User user2 = new User();
    user2.setName("master 赵云");
    user2.setAge(40);
    List<User> userList = new ArrayList<User>();
    userList.add(user1);
    userList.add(user2);
    return userList;
}
```
### `@ResponseBody：`
![[Pasted image 20250913092235.png]]
HttpMessageConverter接口：
`@ResponseBody`是HttpMessageConverter接口实现的，而不是Converter接口实现的
![[Pasted image 20250913092418.png]]
#### 接收参数由三种类型
- 区别
    - @RequestParam用于接收url地址传参或表单传参
    - @RequestBody用于接收json数据
    - @PathVariable用于接收路径参数，使用{参数名称}描述路径参数
- 应用
    - 后期开发中，发送请求参数超过1个时，以json格式为主，@RequestBody应用较广
    - 如果发送非json格式数据，选用@RequestParam接收请求参数
    - 采用RESTful进行开发，当参数数量较少时，例如1个，可以采用@PathVariable接收请求路径变量，通常用于传递id值
## Rest风格
为什么要用它？
![[Pasted image 20250913093557.png]]
如何区分请求类别：
![[Pasted image 20250913093517.png]]
通过其调用行为进行区分。

注意事项：
	上述行为是约定方式，约定不是规范，可以打破，所以称REST风格，而不是REST规范
	描述模块的名称通常使用复数，也就是加s的格式描述，表示此类资源，而非单个资源，例如：users、books、accounts……
### Restful入门案例
1. 设置http请求动作
![[Pasted image 20250913094533.png]]
2. 设定请求参数（路径变量）
![[Pasted image 20250913094705.png]]
#### 新注解
![[Pasted image 20250913094757.png]]
![[Pasted image 20250913094831.png]]
### 优化入门案例
将@RequestMapping和@ResponsBody提出并简化。
![[Pasted image 20250913095649.png]]![[Pasted image 20250913095735.png]]
### 案例（前后端连接）
看看你的SpringMVC拦截器是不是把所有的请求都拦截了（尤其是静态资源的访问<前端页面>）。
config模块中设置SpringMvcSupport文件，覆盖addResourceHandler方法，设置拦截（进行资源重定向）。
1. 制作SpringMVC控制器，并通过测试号接口功能
![[Pasted image 20250913101720.png]]
2. 设置对静态页面的访问放行
![[Pasted image 20250913101837.png]]
3. 前端页面通过异步提交访问后台控制器
![[Pasted image 20250913101935.png]]
# SSM整合
## 后端流程
SSM整合流程：
```
1. 创建工程
2. SSM整合
   ● Spring
       ■ SpringConfig
   ● MyBatis
       ■ MybatisConfig
       ■ JdbcConfig
       ■ jdbc.properties
   ● SpringMVC
       ■ ServletConfig
       ■ SpringMvcConfig
3. 功能模块
   ● 表与实体类
   ● dao（接口+自动代理）
   ● service（接口+实现类）
       ■ 业务层接口测试（整合JUnit）
   ● controller
       ■ 表现层接口测试（PostMan）
```
Spring整合MyBatis：
```
● 配置
  ■ SpringConfig
  ■ JDBCConfig、jdbc.properties
  ■ MyBatisConfig

● 模型
  ■ Book

● 数据层标准开发
  ■ BookDao

● 业务层标准开发
  ■ BookService
  ■ BookServiceImpl

● 测试接口
  ■ BookServiceTest
```
事务管理也是Spring整合MyBatis的部分之一。

Spring整合SpringMVC：
![[Pasted image 20250913110850.png]]
![[Pasted image 20250913110907.png]]

基于Restful的Controller开发：
[[SSM#Rest风格]]，按照这个里面写就行，就是一个规范。
### 实现笔记
SpringMVC可以访问Spring的容器，但Spring访问不了SpringMVC的容器。
SpringMVC是Spring的子容器。
Service层中，会出一个错误（但他不是错误）。spring中没有配置相应的bean（因为用的是自动代理，所以没有对应的Bean给它自动代理）。

有两个环节要停下来做接口测试：
1. 业务层接口开发完之后，要使用JUint做业务层接口的测试工作
![[Pasted image 20250913105647.png]]
SpringService接口是在Spring环境中的，所以是Spring的配置类。
2. 表现层接口（Controller）实现完成之后，要使用Postman做表现层接口的测试工作。

做事务管理，SpringConfig上加上@EnableTransactionManagement注解后，需要数据源对象（在JdbcConfig中创建），在业务层接口加事务（事务传播行为需求到了再进行配置）
## 表现层封装（返回结果进行统一）
前后端数据进行交互时会准备一个协议（联调协议）：
![[Pasted image 20250913115618.png]]
```
data：查询返回的数据封装
code：表示返回类型（CRUD），用于识别data内容。
	一般以0结尾的表示失败，以1结尾的表示成功
msg：为数据返回加上一段信息说明。
```
注意事项：
	Result类中的字段  并不固定 ，可以根据需要自行增减提供若干个构造方法，方便操作
### 实现
一般实现在Controller层包下：
1. Result类。构造函数、get和set方法。其构造函数要一个空的，一个带消息（msg）的，一个不带的。
2. Code类。设置统一数据返回结果编码
![[Pasted image 20250913120846.png]]
Code的常量设计也是不固定的，可根据需求自行增删。
3. 根据实际情况设定合理的Reslut
![[Pasted image 20250913121306.png]]
## 异常处理器
出现异常现象的常见位置与常见诱因如下：
- 框架内部抛出的异常：因使用不合规导致
- 数据层抛出的异常：因外部服务器故障导致（例如：服务器访问超时）<上面的都是用在东代理去做的，所以手写的问题相对较少吗，大多是数据库的问题>
- 业务层抛出的异常：因业务逻辑书写错误导致（例如：遍历业务书写操作，导致索引异常等）<最常见>
- 表现层抛出的异常：因数据收集、校验等规则导致（例如：不匹配的数据类型间导致异常）
- 工具类抛出的异常：因工具类书写不严谨不够健壮导致（例如：必要释放的连接长期未释放等）<我们实现的工具类一般都是按照最佳实践（不出事能用，一出事就挂）来写的>

各个层级均出现异常，异常处理代码书写在哪一层？
	所有的异常均抛出到表现层进行处理
表现层处理异常，每个方法中单独书写，代码书写量巨大且意义不强，如何解决
	AOP思想

异常的处理方案不一样，异常要往上抛，异常要集中在表现层处理，要用AOP思想来处理。
### Spring提供了异常处理器
实现：有RestControllerAdvice和ControllerAdvice，区别在于使没使用Rest风格
![[Pasted image 20250913122648.png]]
在后端查找，return可以将数据返回给Result交给前端看。
#### 新注解
![[Pasted image 20250913123406.png]]
![[Pasted image 20250913123627.png]]
### 项目异常处理方案
项目异常分类
■ 业务异常（BusinessException）
    ■ 规范的用户行为产生的异常
    ■ 不规范的用户行为操作产生的异常
■ 系统异常（SystemException）
    ■ 项目运行过程中可预计且无法避免的异常
■ 其他异常（Exception）
    ■ 编程人员未预期到的异常
![[Pasted image 20250913124226.png]]
#### 处理方法
1. 自定义项目系统级异常
![[Pasted image 20250913125254.png]]
2. 自定义项目业务级的异常
![[Pasted image 20250913125953.png]]
上面两个就是为了分类的，除了类名和方法名字不一样其余都一样
3. 自定义异常编码（持续补充）
![[Pasted image 20250913125541.png]]
4. 触发自定义异常
![[Pasted image 20250913125640.png]]
5. 拦截并处理异常
![[Pasted image 20250913130054.png]]
## 前端连接
想看回去看吧，那几集课的时长也不长，就是用ajax进行前后端异步操作。
![[Pasted image 20250913132335.png]]
## 拦截器
底层是用AOP实现的。
![[Pasted image 20250913132753.png]]在控制器的前和后都做一些事情，可以做一些通用处理，如前面可做权限查询。
```
● 拦截器（Interceptor）是一种动态拦截方法调用的机制，在SpringMVC中动态拦截控制器方法的执行
● 作用：
  ■ 在指定的方法调用前后执行预先设定的代码
  ■ 阻止原始方法的执行
```
拦截器与过滤器区别：
● 归属不同：Filter属于Servlet技术，Interceptor属于SpringMVC技术
● 拦截内容不同：Filter对所有访问进行增强（在Tomcat阶段进行配置），Interceptor仅针对SpringMVC的访问进行增强
## 入门案例
Interceptor最好在Controllerbiaoxiancengxia定义比较合理（SpringMVCConfig即SpringMVC可以少扫描一个包）。
1. 声明拦截器的bean，并实现HandlerInterceptor接口（注意：要加载扫描bean）
![[Pasted image 20250913142454.png]]
2. 定义配置类，继承WebMvcConfigurationSupport，实现addInterceptor方法（注意：扫描加载配置）
![[Pasted image 20250913142544.png]]
3. 添加拦截器并设定拦截的访问路径，路径可以通过可变参数设置多个
![[Pasted image 20250913142800.png]]
请求路径/books就只能在/books这个路径生效而不能在/books/*路径生效，需要单独配置。
- 还可以使用标准接口WebMvcConfigurer简化开发（注意：侵入式较强，其和Spring强绑定了和SpringAPI关联在一起了）
![[Pasted image 20250913142959.png]]
## 拦截参数
前置处理：
![[Pasted image 20250913143628.png]]
可以认为拿到了handler，就可以执行原始执行的方法。
后置处理：
![[Pasted image 20250913143706.png]]
modelAndView现在开发模式中不需要使用它。
完成后处理：
![[Pasted image 20250913144126.png]]
这个exception可以通过SpringMVC提供的异常处理机制替换这个操作。
## 多拦截器执行顺序
![[Pasted image 20250913144705.png]]

拦截器链的运行顺序
- preHandle：与配置顺序相同，必定运行
- postHandle：与配置顺序相反，可能不运行
- afterCompletion：与配置顺序相反，可能不运行
# Maven
[[Maven进阶]]有的就不跟写了，只写点没有的。
## 分模块开发
1. 创建Maven模块
![[Pasted image 20250913151309.png]]
2. 书写模块代码
注意事项：
分模块开发需要先针对模块功能进行设计，再进行编码。不会先将工程开发完毕，然后进行拆分
3. 通过maven指令安装模块到本地仓库（install指令）
注意事项：
团队内部开发需要发布模块功能到团队内部可共享的仓库中（私服）
## 多环境开发
maven提供配置多种环境的设定，帮助开发者使用过程中快速切换环境
![[Pasted image 20250913161243.png]]
1. 在父工程定义多环境：
![[Pasted image 20250913161625.png]]
2. 使用多环境（构建过程）
![[Pasted image 20250913161713.png]]
环境定义id就是profile标签下id标签中的值。
## 跳过测试
```mvn
跳过测试
mvn 指令 -D skipTests

范例：
mvn install -D skipTests
```
- 细粒度控制跳过测试
![[Pasted image 20250913162325.png]]
# `Springboot`
![[Pasted image 20250913170807.png]]
Spring程序和Springboot程序对比：
![[Pasted image 20250913165747.png]]
注意事项：
	基于idea开发SpringBoot程序需要确保联网且能够加载到程序框架结构
springboot可以java -jar运行jar包就是依赖下面的插件
![[Pasted image 20250913170339.png]]
这个插件里包含有其它jar包，还设置了一个入口程序。
## 起步依赖
就是这两个东西，可以点开看看 
![[Pasted image 20250913171410.png]]
- starter
    - SpringBoot中常见项目名称，定义了当前项目使用的所有项目坐标，以达到减少依赖配置的目的
- parent
    - 所有SpringBoot项目要继承的项目，定义了若干个坐标版本号（依赖管理，而非依赖），以达到  减少依赖冲突  的目的 
    - spring-boot-starter-parent (2.5.0) 与 spring-boot-starter-parent (2.4.6) 共计57处坐标版本不同
- 实际开发
    - 使用任意坐标时，仅书写GAV中的G和A，V由SpringBoot提供
    - 如发生坐标错误，再指定version（要小心版本冲突）
注意：
	springboot的起步依赖版本不同差别很大。
	在正式开发的时候，不一定是用的最新的，而是用的最合适的版本。
## 基础配置
可以对照着[[Java Web#sprintboot原理]]一起看，以后主写的配置格式用的是yml
注意：
SpringBoot核心配置文件名为application  
SpringBoot内置属性过多，且所有属性集中在一起修改，在使用时，通过提示键+关键字修改属性
### YAML
![[Pasted image 20250913180411.png]]
层级用空格表示，只要比上面多一个就是个层级（#注释）。
数组数据：
![[Pasted image 20250913180620.png]]
#### YAML数据读取方式（3种）
读取单个数据：
![[Pasted image 20250913181056.png]]
封装成对象读取：
一把这种方式框架内部用到的多一些。
![[Pasted image 20250913181142.png]]
自定义对象封装指定数据：
![[Pasted image 20250913181416.png]]做自定义封装的是时候会出一个警报（加下面的依赖就行）：
```xml
<dependency> 
	<groupId>org.springframework.boot</groupId> <artifactId>spring-boot-configuration-processor</artifactId> 
	<optional>true</optional>
 </dependency>
```
#### 多环境启动
![[Pasted image 20250913182013.png]]
### properties文件多环境启动：
![[Pasted image 20250913182047.png]]
### 多环境命令行启动参数配置
配置文件中有时候会有中文，这时候打包是失败的
```shell
带参数启动SpringBoot
java -jar springboot.jar --spring.profiles.active=test
#改端口
java -jar springboot.jar --server.port=88
#两个都改
aa -jar springboot.jar --server.port=88 --spring.profiles.active=test
```
test改为环境变量名。
参数优先级可以去Springboot官网上查。
### 多环境开发兼容问题
如果都配置了多环境，兼容就看maven。
![[Pasted image 20250913183148.png]]
1. Maven中设置多环境属性
![[Pasted image 20250913183536.png]]
2. Springboot中引用Maven属性
![[Pasted image 20250913183631.png]]
3. 执行Maven打包指令
这一步需要注意：
![[Pasted image 20250913183800.png]]
对资源文件开启对默认占位符的解析：
![[Pasted image 20250913183841.png]]
后：
![[Pasted image 20250913183906.png]]
### 配置文件分类
解决临时属性太多的问题：Springboot中提供了多级配置文件。
![[Pasted image 20250913184103.png]]
idea看到的配置文件是第4级。
springboot2.5.0和2.4.6版本中，config目录下必须有一个文件夹，才能运行最高级的配置文件。
目前2.4.1是主流。
## 整合第三方技术
### SpringBoot整合JUnit：
#### Spring整合JUnit：
![[Pasted image 20250913184943.png]]
bookServices无法自动装配的,改成`@Autowired(required=true)`
当 `required=true` 时，Spring 会尝试查找一个匹配的 Bean 并注入。
#### SpringBoot整合JUnit：
![[Pasted image 20250913185558.png]]
#### 新注解
![[Pasted image 20250913185656.png]]
### SpringBoot整合SSM
基于SpringBoot实现SSM整合：
- SpringBoot整合Spring（不存在）
- SpringBoot整合SpringMVC（不存在）
- SpringBoot整合MyBatis（主要）
#### Spring整合MyBatis
![[Pasted image 20250913190105.png]]
Springboot整合几乎全不用写。
#### Springboot整合MyBatis
2.4.2及以前版本的Springboot在数据库的URL要设置时区，就是表明后面跟`？serverTimezone=UTC`。
- 设置数据源参数
![[Pasted image 20250913191057.png]]
- 定义数据层接口与映射配置
![[Pasted image 20250913191154.png]]
- 测试类中注入dao接口，测试功能
![[Pasted image 20250913191230.png]]
# `MyBatisPlus`
学习目的： 基于MyBatisPlus完成标准Dao开发
- MyBatisPlus（简称MP）是基于MyBatis框架基础上开发的增强型工具，旨在简化开发、提高效率
开发方式
    ● 基于MyBatis使用MyBatisPlus
    ● 基于Spring使用MyBatisPlus
    ● 基于SpringBoot使用MyBatisPlus（直接讲这个了）
国人的东西，直接去官网看就行：[MyBatis-Plus 🚀 为简化开发而生](https://baomidou.com/)
## 入门案例
1. 手动添加map起步依赖
由于MP并未被收录到idea的系统内置配置，无法直接选择加入。
![[Pasted image 20250913193328.png]]
2. 设置JDBC的参数（application.xml）
![[Pasted image 20250913193500.png]]
3. 制作实体类与表结构（类名与表名对应，属性名与字段名对应)
![[Pasted image 20250913193559.png]]
4. 最重要：定义数据接口，继承BaseMapper`<User>`
![[Pasted image 20250913193725.png]]
测试功能就行。
## 标准CRUD操作
![[Pasted image 20250913194022.png]]
### 标准功能分页
MP提供了一个分页拦截器，配上了就有分页就有这个功能了。
```
3.5.9需要导入一个包：<artifactId>mybatis-plus-jsqlparser</artifactId>，因为功能被细分出去了
```
1. 设置分页拦截器作为Spring管理的bean
![[Pasted image 20250913195243.png]]
2. 执行分页查询
![[Pasted image 20250913195321.png]]
开日志：
![[Pasted image 20250913195346.png]]
### 条件查询
![[Pasted image 20250913200100.png]]
![[Pasted image 20250913200131.png]]
推荐格式四。
组合查询条件：
![[Pasted image 20250913200221.png]]
#### null值处理
可以用if-else但推荐下面这个：
![[Pasted image 20250913200624.png]]
### 查询投影
在 `MyBatis Plus `中，查询投影通常指的是在查询数据库时，只选择需要的字段，而不是选择整个表的所有字段。
设置查询出来的结构长什么样子。
![[Pasted image 20250913201157.png]]
#### 查询条件设置
![[Pasted image 20250913211327.png]]![[Pasted image 20250913211350.png]]
### 映射匹配兼容性
问题：
表字段与编码属性设计不同步，
编码中添加了数据库中未定义的属性，
采用默认查询开放了更多的字段查看权限：
![[Pasted image 20250913213136.png]] 
表名与编码开发设计不同步：
![[Pasted image 20250913213049.png]]
##  DML编程控制
![[Pasted image 20250914073101.png]]
- AUTO(0)：使用数据库id自增策略控制id生成
- NONE(1)：不设置id生成策略
- INPUT(2)：用户手工输入id
- ASSIGN_ID(3)：雪花算法生成id（可兼容数值型与字符串型）
- ASSIGN_UUID(4)：以UUID生成算法作为id生成策略
![[Pasted image 20250914073729.png]]
table_prefix表示匹配表名，id-type匹配@TableId注解中的type属性。
可以将文件中的这些属性提取出来形成解耦。
### 多数据操作（查询和删除）
这两种方法都需要提供表没行对应的id值。
![[Pasted image 20250914074238.png]]
#### 逻辑删除
逻辑删除其本质上并没有删除数据，只是在被删除数据后加上了一个标记。
今年合计应该是1,000,000,000
![[Pasted image 20250914074824.png]]
实现：
![[Pasted image 20250914075308.png]]
或者在实体类中添加对应字段，并设定当前字段为逻辑删除标记字段@TableLogic，使用其属性value和另一个设置deleteId。
底层会把删除语句改为更新语句。
![[Pasted image 20250914075447.png]]
## 乐观锁
修改操作涉及到的东西，请求量2000- 可用这个方案。
1. 实体类中添加对应字段，并设计当前字段为逻辑删除字段。
![[Pasted image 20250914080416.png]]
2. 配置乐观锁拦截器实现锁机制对应的动态SQL语句拼装
![[Pasted image 20250914080531.png]]
查询测试：
![[Pasted image 20250914080646.png]]
## 代码生成器
- 模板：MyBatisPlus提供
- 数据库相关配置：读取数据库获取信息
- 开发者自定义配置：手工配置


