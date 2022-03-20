# spring，java开发框架
## 核心：IOC、AOP
1. IOC控制反转：一种设计思想，将手动创建对象的控制权交由spring框架来管理。如果没有IOC，存在对象依赖时，程序员需要知道对象的内部细节才能创建对象，同时在修改时，会更加麻烦
   1. 控制：创建对象的权利
   2. 反转：控制权转换
   3. DI依赖注入：IOC的实现方式。
      1. 基于XML注入：构造方法、setter方法
      2. 基于注解注入@Autowired：构造方法、setter、字段、方法参数
2. AOP面向切面编程
   1. 是OOP面向对象编程的延续，OOP通过提取*类*的重复方法解决代码重复问题，但是不能解决*方法*内部的重复代码块（横切逻辑代码）。方法=横切逻辑代码+业务逻辑代码
3. spring AOP，基于动态代理，运行时增强
   1. 要代理的对象实现了某个接口，使用JDK Proxy
   2. 未实现接口，使用Cglib Proxy
4. aspectJ AOP，基于字节码操作，编译时增强。性能比spring aop好，更复杂
   1. 是最完整的AOP框架
   

5. JDK Proxy
   1. 基于反射，在invoke中调用method
   2. 只能代理实现了接口的类的原因：因为生成的代理类继承了Proxy类，java不支持多继承类，被代理的类（原来需要被代理的类）只能是接口类。
   3. 核心是InvocationHandler接口，通过Proxy 类的 newProxyInstance() 创建的代理对象在调用方法的时候，实际会调用到实现InvocationHandler 接口的类的 invoke()方法。
   4. 方法
      1. 定义一个接口及其实现类；
      2. 自定义 InvocationHandler 并重写invoke方法，在 invoke 方法中我们会调用原生方法（被代理类的方法）并自定义一些处理逻辑；
      3. 通过 Proxy.newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h) 方法创建代理对象；
``` 
newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)生成代理对象
1. loader：类加载器，用于加载代理对象
2. interfaces：被代理类实现的一些接口
3. h：实现了InvocaHandler接口的对象，
```  
   2. 调用invoke(Object proxy, Method method, Object[] args)方法，*在此处自定义逻辑*
```
proxy :动态生成的代理类
method : 与代理类对象调用的方法相对应
args : 当前 method 方法的参数
```


6. CGLIB动态代理
   1. 通过继承实现代理
   2. 核心：MethodInterceptor 接口和 Enhancer 类
   3. 方法
      1. 定义一个类；
      2. 自定义 MethodInterceptor 并重写 intercept 方法，intercept 用于拦截增强被代理类的方法，和 JDK 动态代理中的 invoke 方法类似；
      3. 通过 Enhancer 类的 create()创建代理类；
 

     

## spring bean
1. 被IOC容器管理的对象
2. bean的作用域
   1. 分类
      1. singleton：唯一实例，默认都是单例的，单例模式的应用
      2. prototype：每次请求创建一个新的实例
      3. request：每次request请求创建一个bean
      4. session：每一次来自新 session 的 HTTP 请求都会产生一个新的 bean
   2. 配置作用域
      1. xml方式
      2. 注解方式：```@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)```
   3. 单例bean存在线程安全问题
      1. 避免定义可变的成员变量
      2. ThreadLocal
   4. @Component 和 @Bean 的区别
      1. 都是注册bean到容器中
      2. @Component（@Service、@Controller、@Repository）注解作用于类，而@Bean注解作用于方法，这个方法返回一个对象
      3. @Component是通过类路径扫描来自动侦测以及自动装配到容器中，可以使用@ComponentScan定义扫描路径，@Bean告诉Spring这个方法将会返回一个对象，这个对象要注册为Spring应用上下文中的bean。通常方法体中包含了最终产生bean实例的逻辑。
      4. 引用第三方库时，只能通过@Bean实现
   5. bean的生命周期
      1. spring容器类型
         1. spring BeanFactory容器，最简单的容器
         2. ApplicationContext容器，称spring上下文，包括了BeanFactory的功能，是其子接口
      2. ApplicationContext容器中bean的生命周期
         1. Bean 容器找到配置文件中 Spring Bean 的定义。
         2. Bean 容器利用 Java Reflection API 创建一个 Bean 的实例。
         3. 如果涉及到一些属性值 利用 set()方法设置一些属性值。
         4. 调用aware相关接口
            1. 如果 Bean 实现了 BeanNameAware 接口，调用 setBeanName()方法，传入 Bean 的名字。
            2. 如果 Bean 实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader()方法，传入 ClassLoader对象的实例。
            3. 如果 Bean 实现了 BeanFactoryAware 接口，调用 setBeanFactory()方法，传入 BeanFactory对象的实例。
            4. 与上面的类似，如果实现了其他 *.Aware接口，就调用相应的方法。
         5. 执行postProcessBeforeInitialization() 方法（如果有和加载这个 Bean 的 Spring 容器相关的 BeanPostProcessor 对象）
         6. 执行afterPropertiesSet()方法。（如果 Bean 实现了InitializingBean接口）
         7. 如果 Bean 在配置文件中的定义包含 init-method 属性，执行指定的方法。
         8. 执行postProcessAfterInitialization() 方法
         9. 正常使用
         10. 销毁
             1.  如果 Bean 实现了 DisposableBean 接口，执行 destroy() 方法。
             2.  如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的方法。

## spring 事务
1. 种类
   1. 编程式事务，在代码中硬编码(不推荐使用) : 通过 TransactionTemplate或者 TransactionManager 手动管理事务，实际应用中很少使用，但是对于你理解 Spring 事务管理原理有帮助。
   2. 声明式事务，在 XML 配置文件中配置或者直接基于注解（推荐使用） : 实际是通过 AOP 实现（基于@Transactional 的全注解方式使用最多）
      1. 作用于类：当把@Transactional 注解放在类上时，表示所有该类的 public 方法都配置相同的事务属性信息。
      2. 作用于方法：当类配置了@Transactional，方法也配置了@Transactional，方法的事务会覆盖类的事务配置信息
2. 事务传播
   1. TransactionDefinition.PROPAGATION_REQUIRED，默认方式。如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务
   2. PROPAGATION_REQUIRES_NEW，创建一个新的事务，如果当前存在事务，则把当前事务挂起。
   3. PROPAGATION_NESTED，如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。
   4. PROPAGATION_MANDATORY，如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
3. 隔离级别
   1. 缺省：使用后端数据库默认的隔离级别，MySQL 默认采用的 REPEATABLE_READ 隔离级别 Oracle 默认采用的 READ_COMMITTED 隔离级别.
   2. 读未提交
   3. 读已提交
   4. 可重复读
   5. 串行
4. @Transactional(rollbackFor = Exception.class)
   1. 在 @Transactional 注解中如果不配置rollbackFor属性,那么事务只会在遇到RuntimeException的时候才会回滚，加上 rollbackFor=Exception.class,可以让事务在遇到非运行时异常时也回滚。
5. 非持久化一个字段
   1. transient String transient3; // not persistent because of transient
   2. 使用注解@Transient
              String transient4; // not persistent because of @Transient


## spring 中的设计模式
1. 工厂模式：通过BeanFactory、ApplicationContext创建Bean
2. 代理模式：AOP实现
3. 单例模式：bean作用域默认是单例
4. 模板方法：jdbcTemplate等以template结尾的数据库操作类
5. 装饰模式：
6. 观察者模式：spring事件驱动模型
7. 适配器模式：AOP的增强或通知（Advice）实现，spring MVC的controller


# spring boot
1. spring引入依赖时，需要xml配置，而springboot不需要
2. 自动装配：EnableAutoConfiguration
   1. 判断自动装配开关是否打开
   2. 获取EnableAutoConfiguration注解中的 exclude 和 excludeName
   3. 获取需要自动装配的所有配置类，读取META-INF/spring.factories
      1. 并不是每次都加载所有配置，条件注解可以筛选，@ConditionalOnXXX 中的所有条件都满足，该类才会生效。
   4. Spring Boot 通过@EnableAutoConfiguration开启自动装配，通过 SpringFactoriesLoader 最终加载META-INF/spring.factories中的自动配置类实现自动装配，自动配置类其实就是通过@Conditional按需加载的配置类，想要其生效必须引入spring-boot-starter-xxx包实现起步依赖

## @SpringBootApplication
1. 作用在主类上，看作是 @Configuration、@EnableAutoConfiguration、@ComponentScan 注解的集合。
2. @EnableAutoConfiguration：启用 SpringBoot 的自动配置机制
   1. 通过AutoConfigurationImportSeletor类（加载自动装配类）实现
3. @ComponentScan： 扫描被@Component (@Service,@Controller)注解的 bean，注解默认会扫描该类所在的包下所有的类。
4. @Configuration：允许在 Spring 上下文中注册额外的 bean 或导入其他配置类

## Spring Bean 相关
1. @Autowired：自动导入对象到类中，被注入进的类同样要被 Spring 容器管理比如：Service 类注入到 Controller 类中。
2. @Component ：通用的注解，可标注任意类为 Spring 组件。如果一个 Bean 不知道属于哪个层，可以使用@Component 注解标注。
3. @Repository : 对应持久层即 Dao 层，主要用于数据库相关操作。
4. @Service : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
5. @Controller : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。
6. @RestController注解：是@Controller和@ResponseBody的合集
7. @Scope：声明 Spring Bean 的作用域
8. @Configuration：一般用来声明配置类，可以使用 @Component注解替代
9. @PathVariable用于获取路径参数，@RequestParam用于获取查询参数。

### ymal配置文件中的内容
1. @Value：作用在对象上，使用 @Value("${property}") 读取比较简单的配置信息：
2. @ConfigurationProperties：作用在类上，读取配置信息并与 bean 绑定。

### 处理异常
1. 使用 @ControllerAdvice 和 @ExceptionHandler 处理全局异常
   1. @ControllerAdvice作用在advice类上，用以包含需要处理的controller
   2. @ExceptionHandler作用在方法上，处理异常

