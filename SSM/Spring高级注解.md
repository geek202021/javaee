#  Spring高级注解

##  @Bean注解的基本使用

```java
//1.com.jun.bean
public class User{}
//2.com.jun
@Configuration
public class AppConfig{
    //1.简单对象的创建
    @Bean
    public User user(){
        return new User();
    }
    //2.创建复杂对象，如Connection，不能直接通过new创建
    @Bean
    public Connection connect(){
        Class.forName("com.mysql.jdbc.Driver");
        Connection connect = DriverManager.getConnection(url,user,password);
        return connect;
    }
}
//3.test
ApplicaitionContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
User user = (User) ctx.getBean("user");
soutv
//4.复杂对象的测试
……
Connection connect = (Connection)ctx.getBean("connect");
soutv
```

>@Bean注解创建复杂对象的注意事项
>
>- Spring工厂的创建中：是建议复杂对象由FactoryBean来创建的
>- FactoryBean 和 @Bean注解联合使用  (用在遗留系统较多)

 ```java
 //1.com.jun.bean
 public class ConnectionFactoryBean implements FactoryBean<Connection>{
     @Override
     public Connection getObject() throws Exception{
         Class.forName("com.mysql.jdbc.Driver");
         Connection connect = DriverManager.getConnection(url,user,password);
         return connect;
     }
     @Override
     public Class<?> getObjectType(){
         return Connection.class;
     }
     @Override
     public boolean isSingleton(){
         return false;
     }
 }
 //2.com.jun
 @Configuration
 public class AppConfig{
     @Bean
     public Connection conn1(){
         ConnectionFactoryBean factoryBean = new ConnectionFactoryBean();
         Connection conn = factoryBean.getObject();
         return conn; 
     }
 }
 //3.test
 Connection conn1 = (Connection) ctx.getBean("conn1");
 soutv
 ```

> @Bean控制对象创建的次数

@Bean 

@Scope("singleton")/ @Scope("prototype") 默认值是singleton

##  @Bean注解的注入

>用户自定义类型的注入

```java
//1.com.jun.injection
public class UserDAOImpl implements UserDAO{
    @Override
    public void save(){
        System.out.println("UserDAOImpl.save");
    }
}
public class UserServiceImpl implements UserService{
    private UserDAO userDAO;
    public UserDAO getUserDAO(){
        return userDAO;
    }
    public void setUserDAO(UserDAO userDAO){
        this.userDAO = userDAO;
    }
    @Override
    public void register(){
        userDAO.save();
    }
}
//2.com.jun
@Configuration
public class AppConfig1 {
    @Bean
    public UserDAO userDAO() {
        return new UserDAOImpl();
    } 
    // 定义形参
    @Bean
    public UserService userService(UserDAO userDAO) {
        UserServiceImpl userService = new UserServiceImpl();
        userService.setUserDAO(userDAO);
        return userService;
    }
    //简化写法
    @Bean
    public UserService userService() {
        UserServiceImpl userService = new UserServiceImpl();
        // 注入时，直接调用上面的方法即可
        userService.setUserDAO(userDAO());
        return userService;
    }
}
```

>JDK类型注入

```java
//1.com.jun.Customer
public class Customer{
    private Integer id;
    private String name;
    set get方法
}
//2.init.properties
id=2  name=jun
//3.com.jun
@Configuration
@PropertySource("classpath:/init.properties")    
public class AppConfig1{
    @Value("${id}")
    private Integer id;
    @Value("${name}")
    private String name;
    @Bean
    public Customer customer(){
        Customer customer = new Customer();
        customer.setId(id);
        customer.setName(name);
        return customer;
    }
}   
//4.test
Customer customer = (Customer)ctx.getBean("customer");
```

##  @ComponentScan

- 等同于<context:component-scan base-package=""  ；
  - 目的：进行相关注解的扫描

```java
//1.com.jun.scan
@Component
public class User1{}
@Service
public class User2{}
//2.com.jun
@Configuration
@ComponentScan(basePackages="com.jun.scan")
public class AppConfig2{
    
}
//3.test
ApplicationContext ctx = new AnnotationConfigApplicationContext("AppConfig2.class");
//返回值是Spring工厂所创建的对象的id值
String[] beanDefinitionNames=ctx.getBeanDefinitionNames();
增强for遍历输出
```

>排除策略

```xml
xml的方式：
<context:component-scan base-package="com.yusael">
	<context:exclude-filter type="assignable" expression="com.yusael.bean.User"/>
</context:component-scan>
```

- 注解的方式:排除

```java
@Configuration
@ComponentScan(basePackages="com.jun.scan",excludeFilters={@ComponentScan.Filter(type=FilterType.ANNOTATION,value={Service.class}),@ComponentScan.Filter(type= FilterType.ASPECTJ, pattern = "*..User1")})
public class AppConfig2{}

type = FilterType.ASSIGNABLE_TYPE-->	value
					 .ANNOTATION --> 	value
					 .ASPECTJ	--> 	pattern
					 .REGEX		-->		pattern
					 .CUSTOM	-->		value
```

- 包含：

```java
@ComponentScan(basePackages = "com.jun.scan",
		// 关闭Spring的默认扫描方式
        useDefaultFilters = false,
        includeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, value = {Service.class}),
                @ComponentScan.Filter(type = FilterType.ASPECTJ, pattern = {"com.jun..*", "*..User1"})
        })
```

# Spring工厂创建对象的应用方式

##  解决注解配置的耦合问题

```java
--------配置Bean：AppConfig4---------
@Configuration
public class AppConfig4 {
    @Bean
    public UserDAO userDAO() {
        return new UserDAOImpl();
    }
}
--------applicationContext.xml---------
覆盖userDAO，使用新的实现类

<bean id="userDAO" class="com.angenin.dao.UserDAOImplNew"/>

--------不修改AppConfig4，需要创建一个新的配置Bean：AppConfig5--------
@Configuration
@ImportResource("applicationContext.xml")
public class AppConfig5 {
}

------创建的工厂--------
// 可以同时引用多个配置Bean
// ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig4.class, AppConfig5.class);
// 也可以直接指向一个包
ApplicationContext ctx = new AnnotationConfigApplicationContext("com.jun.appconfig");
```

##  多配置信息的整合

###  多配置Bean信息整合

>1.基于包扫描的方式进行文件整合

```java
//1.com.jun.config
@Configuration				
public class AppConfig1{	 	
    @Bean						
    public UserService userService(){	
        UserServiceImpl userService = new UserServiceImpl();
        return userService;			
    }							
}							
@Configuration
public class AppConfig2{	
    @Bean
    public UserDAO userDAO(){
        return new UserDAOImpl();
    }
}    
//2.test
public void test1(){
    ApplicationContext ctx = new AnnotationConfigApplicationContext("com.jun.config");
    UserDAO userDAO = (UserDAO)ctx.getBean("userDAO");//.castvar
    UserService userService = (UserService)ctx.getBean("userService");
    soutv;
    soutv;
}        
```

>2.基于@import注解完成多配置bean的整合

```java
@Configuration	
@Import(AppConfig2.class)
public class AppConfig1{	 	
    @Bean						
    public UserService userService(){	
        UserServiceImpl userService = new UserServiceImpl();
        return userService;			
    }							
}
//2.test
ApplicationContext ctx = AnnotationConfigApplicationContext(AppConfig1.class);
```

>3.在工厂创建时，指定多个配置Bean的Class对象【了解】

###  跨配置进行注入

>跨配置进行注入(适用于应用配置Bean的所有场景)

```java 
@Configuration
@Import(AppConfig2.class)
public class AppConfig1{
    @AutoWired
    private UserDAO userDAO;
    @Bean 
    public UserService userService(){	
        UserServiceImpl userService = new UserServiceImpl();
        userService.setUserDAO(userDAO);
        return userService;			
    }
}
@Configuration
public class AppConfig2{	
    @Bean
    public UserDAO userDAO(){
        return new UserDAOImpl();
    }
}
//test
userService.register();
```

###  配置Bean与@Component相关注解的整合

```java 
//1.com.jun.injection
@Repository //默认名字是userDAOImpl
public class UserDAOImpl implements UserDAO{
    @Override
    public void save(){
        System.out.println("UserDAOImpl.save");
    }
}
//2.com.jun.config
@Configuration
@ComponentScan(basePackages="com.jun.injection")
public class AppConfig3{
    @Autowired
    private UserDAO userDAO;
    
    @Bean
    public UserService userService(){
        UserServiceImpl userService = new UserServiceImpl();
        userService.setUserDAO(userDAO);
        return userService;
    }
}
//3.test
ApplicationContext ctx = new AnnotationConfigApplicationContext(com.jun.config.AppConfig3.class);
UserService userService = (UserService)ctx.getBean("userService");
UserDAO userDAOImpl = (UserDAO)ctx.getBean("userDAOImpl");
System.out.println("userService="+userService);
System.out.println("userDAOImpl="+userDAOImpl);
//对配置Bean进行DAO的注入
userService.register();
```

###  配置Bean与xml配置文件的整合

- 应用场景：1.遗留系统的整合；2.配置覆盖

```java
//1.com.jun.injection
//@Repository //默认名字是userDAOImpl
public class UserDAOImpl implements UserDAO{
    @Override
    public void save(){
        System.out.println("UserDAOImpl.save");
    }
}
//2.applicationContext.xml
<bean id="userDAO" class="com.jun.injection.UserDAOImpl"/>
//3.com.jun.config
@Configuration
@ImportResource("applicationContext.xml")    
public class AppConfig4{
    @Autowired
    private UserDAO userDAO; //这里的报错并不影响
    
    @Bean
    public UserService userService(){
        UserServiceImpl userService = new UserServiceImpl();
        userService.setUserDAO(userDAO);
        return userService;
    }
}
//4.test
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig4.class);
UserService userService = (UserService)ctx.getBean("userService");
UserDAO UserDAO = (UserDAO) ctx.getBean("userDAO");//基于配置文件中的id
userService.register();
```

#  四位一体的开发思想

>1.基于schema的方式

```java 
//init.properties
//1.com.jun.four
public class Account{
    @Value("${id}")
    private Integer id;
    @Value("${name}")
    private String name;
   	get/set方法
}
//applicationContext.xml
<context:component-scan base-package="com.jun.four"/>
<context:property-placeholder location="classpath:init.properties"/> 
<bean id="account" class="com.jun.four.Account"/>  
//test
 打印getId和getName的值   
```

>2.基于注解的方式

```java
<context:component-scan base-package="com.jun.four"/>
@Component
@PropertySource("classpath:init.properties")
public class User {
    @Value("${id}")
    private Integer id;
    @Value("${name}")
    private String name;
    //get set
}    
```

#  纯注解AOP开发

```java
//1.com.jun.aop
@Configuration //应用配置Bean
@ComponentScan(basePackages="com.jun.aop") //注解扫描
public class AppConfig{}
//2.com.jun.aop.UserServiceImpl创建原始对象
@Service 
public class UserServiceImpl implements UserService{
    @Override
    public void register(){
        System.out.println("UserServiceImpl.res");
    }
    @Override
    public void login(){
        sout
    }
}
//3.com.jun.aop.MyAspect 创建切面类
@Aspect
@Component
public class MyAspect{
    @Pointcut("execution(* com.jun.aop..*.*(..))")
    public void pointCut(){}
    
    @Around("pointCut()")
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable{
        System.out.println("---Log---");
        Object proceed = joinPoint.proceed();
    }
}
//4.在APPConfig上添加注解
@Configuration
@ComponentScan(basePackages="com.jun.aop") 
@EnableAspectJAutoProxy
public class AppConfig{}
```



