“随机盐 + MD5”实现了基本的加盐哈希，但 MD5 太弱，不应用于生产环境。建议迁移到 **`bcrypt`** 或 **PBKDF2**。
`@Autowired(required = false)` 是 Spring 依赖注入中的一个**容错开关**。它的含义是：**如果找不到匹配的 Bean，就不要注入，也不要报错，保持为 `null`**。
# Config
## 类上
### `@RestControllerAdvice`
`@RestControllerAdvice` 是 Spring Framework 提供的一个**组合注解**，它本质上是 `@ControllerAdvice` 与 `@ResponseBody` 的结合体
- **全局异常处理**：配合 `@ExceptionHandler` 捕获 Controller 层抛出的异常，并将返回结果自动转换为 JSON/XML 等响应体（因为内嵌了 `@ResponseBody`），从而统一错误响应格式。
- **全局数据绑定增强**：配合 `@InitBinder` 自定义参数绑定逻辑（如日期格式转换）。
- **全局模型属性填充**：配合 `@ModelAttribute` 为所有 Controller 的 Model 添加公共属性。
---
## 方法上
### @`ExceptionHandler`
`@ExceptionHandler` 是 Spring MVC 中用于**处理控制器（Controller）层抛出的异常**的注解。当某个请求处理方法（如 `@GetMapping` 方法）抛出指定类型的异常时，被 `@ExceptionHandler` 标记的方法会接管该异常，并返回自定义的响应（如错误视图、JSON 数据等）。
- **异常集中处理**：避免在每个 Controller 方法中使用 `try-catch` 重复处理相同类型的异常。
- **自定义异常响应**：可以根据异常类型返回不同的 HTTP 状态码、错误信息或视图。
- **保持业务代码整洁**：让 Controller 方法专注于正常业务流程，异常处理逻辑被抽离到专门的方法中。
注解中的参数（通常通过 `value` 属性指定）就是**需要处理的异常类型**。
当 Controller 中的方法抛出某种异常时，Spring 会查找与该异常类型匹配的 `@ExceptionHandler` 方法（支持按继承关系匹配），然后执行该方法来处理异常。

---

# Controller
## 类上
### @`RestController`
`@RestController` 是 **`@Controller` + `@ResponseBody`** 的组合注解，用于标记 **RESTful 风格的控制器**。  
它的作用是：将控制器中每个方法的返回值**直接写入 HTTP 响应体**（通常为 JSON/XML），而不是解析为视图名称，从而简化前后端分离接口的开发。

---
## 方法上
### @Resource
`@Resource` 是 JSR-250 规范提供的**依赖注入注解**，用于按**名称**（name 属性）或**类型**自动装配 Spring Bean。  
其作用类似于 `@Autowired`，但主要区别是：
- 默认**按名称匹配**（可通过 `name` 指定），找不到名称再按类型匹配。
- 属于 Java 标准扩展，不强制依赖 Spring 框架。  
    常用于注入 `Service、Dao` 等组件。
---
## 参数中
- **`@RequestBody`**：将 HTTP 请求体（JSON/XML）中的内容**绑定到控制器方法的参数对象**，常用于接收 POST/PUT 请求提交的复杂数据。
- **`@PathVariable`**：从 URL 路径模板中**提取变量值**绑定到方法参数，如 `/user/{id}` 中的 `{id}`
- **`@RequestParam`**：获取请求参数（URL 问号后的键值对或表单数据）的值，可设置是否必需、默认值等，常用于 GET 请求的查询参数。
---
# MP
MP 的 `BaseMapper` 解决了单表基础操作的重复编码问题，但并未也不应该试图覆盖所有 SQL 场景。  当遇到 多表关联、复杂统计、特定优化、非标准返回结构 等需求时，在 Mapper 中扩展自定义方法是合理且必要的。

使用MP时，要给mybatis提供配置类
`MybatisConfig` 配置类的作用是**为 `MyBatis-Plus` 配置拦截器插件** ，具体来说就是**启用并配置分页插件**。

`@MapperScan` 是 MyBatis（以及 MyBatis-Plus）提供的注解，用于**批量扫描指定包路径下的 Mapper 接口**，并将其自动注册为 Spring 的 Bean，从而无需在每个 Mapper 接口上单独使用 `@Mapper` 注解。
```java
@Configuration
@MapperScan("com.example.demo.mapper") // 替换为你的Mapper包路径
public class MybatisPlusConfig {
    /**
     * MyBatis-Plus 核心插件配置
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();

        // 1. 多租户插件 (TenantLineInnerInterceptor)
        // 作用：自动为SQL语句添加租户ID过滤条件，实现数据隔离
        interceptor.addInnerInterceptor(new TenantLineInnerInterceptor(new TenantLineHandler() {
            @Override
            public Expression getTenantId() {
                // 模拟从上下文中获取当前租户ID，实际中应从登录信息中获取
                return new LongValue(1L); 
            }
            @Override
            public String getTenantIdColumn() {
                // 指定数据库表中的租户ID字段名
                return "tenant_id";
            }
            @Override
            public boolean ignoreTable(String tableName) {
                // 某些表不需要多租户隔离，可在此处进行排除
                return "sys_config".equalsIgnoreCase(tableName);
            }
        }));

        // 2. 动态表名插件 (DynamicTableNameInnerInterceptor)
        // 作用：根据业务逻辑动态替换SQL中的表名，常用于分表场景
        DynamicTableNameInnerInterceptor dynamicTableNameInterceptor = new DynamicTableNameInnerInterceptor();
        dynamicTableNameInterceptor.setTableNameHandler((sql, tableName) -> {
            // 示例：根据当前年份动态替换表名后缀
            String year = String.valueOf(java.time.Year.now().getValue());
            if (tableName.equals("user_log")) {
                return "user_log_" + year;
            }
            return tableName;
        });
        interceptor.addInnerInterceptor(dynamicTableNameInterceptor);

        // 3. 分页插件 (PaginationInnerInterceptor)
        // 作用：自动实现物理分页，将Page对象转换为对应的数据库方言LIMIT语句
        PaginationInnerInterceptor paginationInterceptor = new PaginationInnerInterceptor();
        paginationInterceptor.setDbType(DbType.MYSQL); // 设置数据库类型
        paginationInterceptor.setMaxLimit(1000L);      // 设置单页最大条数限制
        interceptor.addInnerInterceptor(paginationInterceptor);

        // 4. 乐观锁插件 (OptimisticLockerInnerInterceptor)
        // 作用：实现乐观锁机制，更新时会自动对version字段进行+1和条件校验，防止并发更新冲突
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());

        // 5. SQL性能规范插件 (IllegalSQLInnerInterceptor)
        // 作用：拦截并检查风险SQL，如全表更新/删除、无索引查询等，提升安全性和性能
        interceptor.addInnerInterceptor(new IllegalSQLInnerInterceptor());

        // 6. 防全表更新/删除插件 (BlockAttackInnerInterceptor)
        // 作用：终极安全防线，阻止无任何条件的update和delete语句执行，保护数据安全
        interceptor.addInnerInterceptor(new BlockAttackInnerInterceptor());
        return interceptor;
    }
}
```
有些情况下MP不再适用：
- 反过来，MP只适合简单的单表CRUD、简单的条件查询
![[Pasted image 20260418231108.png]]
# Entity
## 类上：
- **`@EqualsAndHashCode(callSuper = false)`**（Lombok）  
    自动生成 `equals()` 和 `hashCode()` 方法。`callSuper=false` 表示**不调用父类的字段**进行比较和计算，只基于当前类的非继承字段。
    
- **`@Accessors(chain = true)`**（Lombok）  
    使生成的 `setter` 方法返回当前对象（即 `return this`），从而支持**链式调用**，如 `obj.setName("a").setAge(10)`。
    
- **`@TableName("tb_blog")`**（`MyBatis-Plus`）  
    指定实体类对应的数据库表名为 `tb_blog`，用于 MP 自动生成 SQL 时识别表名。
---
## 方法上：
- **`@TableId(value = "id", type = IdType.AUTO)`**  
    `MyBatis-Plus` 注解，用于标识实体类中的主键字段。
    - `value = "id"`：指定数据库表中的主键列名为 `id`。
    - `type = IdType.AUTO`：设置主键生成策略为**数据库自增**（如 MySQL 的 `AUTO_INCREMENT`），插入数据时无需手动赋值，由数据库自动生成。
    
- **`@TableField(exist = false)`**  
    `MyBatis-Plus` 注解，标记实体类中的某个字段在数据库表中**不存在**。  
    常用于临时字段、关联查询结果字段或额外计算属性，MP 在生成 SQL 语句时会自动忽略该字段（不参与 INSERT/UPDATE/SELECT 的列映射）。
---
# Service
## 方法上 
`@Transactional` 用于**声明式事务管理**：标记在方法或类上，表示该方法（或类中所有方法）执行时被 Spring 事务切面包裹，确保数据库操作**要么全部成功提交，要么遇到运行时异常时自动回滚**，从而保证数据的一致性和完整性。
# 继承父类
`WebMvcConfigurer` 是 Spring MVC 的**配置接口**。继承（实现）它可以**自定义 Spring MVC 的底层行为**，例如：添加拦截器、配置跨域（CORS）、设置视图控制器、自定义消息转换器（如 JSON 处理）、静态资源映射、参数解析器等。

# 实现接口
实现 `HandlerInterceptor` 接口用于**自定义请求拦截逻辑**，在 Spring MVC 中可以对控制器的执行进行前置处理、后置处理及视图渲染后的处理。  
典型作用包括：**权限验证、日志记录、性能监控、参数预处理、统一响应修改**等。通过重写 `preHandle`、`postHandle`、`afterCompletion` 方法，可以在请求进入 Controller 之前或之后执行通用代码，实现横切关注点与业务逻辑的解耦。
# 使用Redission
需要配置配置类( `redis路径端口`，`密码` )。
```java
// 把文件提前加载，降低磁盘的IO流，提升效率  
private static final DefaultRedisScript<Long> SECKILL_SCRIPT;  
  
static {  
    // 静态代码块就是类加载的时候会被执行一次，就是最好的初始化方法啊  
    SECKILL_SCRIPT = new DefaultRedisScript<>();  
    SECKILL_SCRIPT.setLocation(new ClassPathResource("secKill.lua"));  
    // 配置返回值  
    SECKILL_SCRIPT.setResultType(Long.class);  
}  
  
// 创建线程池，异步处理订单  
private ExecutorService SECKILL_ORDER_EXECUTOR = Executors.newSingleThreadExecutor();  
  
// 在当前类 VoucherOrderServiceImpl 初始化后就来执行  
@PostConstruct  
private void init() {  
    SECKILL_ORDER_EXECUTOR.submit(new VoucherOrderTask());  
}  
  
  
// 定义阻塞队列  
private BlockingQueue<VoucherOrder> orderTasks = new ArrayBlockingQueue<>(1024 * 1024);
```
