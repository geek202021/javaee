# Spring注解开发

start from P151 2021年11月26日01:05:39

`https://cdn.jsdelivr.net/gh/gwbiubiu/images/20210711093946.png`

# 第一章 注解的基本概念

##  1.什么是注解编程？

```java
	指的是在类或者方法上加入特定的注解（@XXX)，完成特定功能的开发。
	@Component
	public class XXX{}
```

##  2.为什么要讲解注解编程

```markdown
1. 注解开发方便
	代码简洁，开发速度大大提高
2. Spring开发的潮流
	Spring 2.x引入注解 Spring 3.x完善注解 SpringBoot普及 推广注解编程。
```

##  3.注解的作用

- 替换XML这种配置形式，简化配置

<img src="ssm-imgs\p151-annotation1.png" align="left" />

- 替换接口，实现调用双方的契约性

<img src="ssm-imgs\p151-annotation2.png" align="left" />

```markdown
1. 通过注解的方式，在功能的调用者和提供者之间达成约定，进而进行功能的调用。因为注解更为方便灵活，所以在现实的开发中，更推荐使用注解的形式完成
```

<img src="ssm-imgs\p151-annotation3.png" align="left" />

##  4.Spirng注解的发展历程

1. Spirng2.x开始支持注解编程 @Component @Service @Scope
	目的：提供这些注解只是为了在某些情况下简化XML的配置，作为XML的有益补充
2. Spring3.x @Configuration @Bean
	目的：彻底替换XML，基于纯注解编程
3. Spirng4.x SpringBoot
	提倡使用注解进行常见开发

##  5.Spring注解开发的一个问题

- Spring基于注解进行配置后，还能否解耦合呢？

- 在Spring框架应用注解时，如果对注解配置的内容不满意，可以通过Spring配置文件进行覆盖。

#  第二章 Spring基础注解（Spring 2.x）

1. 这个阶段的注解，仅仅是简化XML的配置，并不能完全替代XML。

- 搭建开发环境

```xml
<context:component-scan base-package="com.gewei"/>

作用：让Spring框架在设置包及其子包扫描对应的注解，使其生效。
```

##  1.对象创建相关注解@Component

```markdown
作用：替换Spring配置文件中的Bean标签

注意：id属性 component注解提供了默认的设置方式，即单词首字母小写
	 class属性，通过反射获得class内容
```

<img src="ssm-imgs\p155-@component.png" align="left" />

- 测试：引入Junit测试单元

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

```java
@Test
public void test1(){
    ApplicationContext ctx = new ClassPathXmlApplicationContext("/SpringContext.xml");
    User user = (User) ctx.getBean("user");
    System.out.println("user+"+user);
}
```

###  1-1. @Component注解使用细节

- 如何显示的指定工厂创建对象的id值

```java
@Component("u") //此时User对象的id值就不再是user了
```

- Spring配置文件覆盖注解配置内容

```xml
applicationContext.xml

<bean id = "u" class = "com.gewei.bean.User" />
# id的值和class的值要和注解中的设置保持一致，这样才能达到覆盖的功能，如果id值不一致，那么会创建出一个新的对象，并不能实现覆盖的功能。
```

##  2 @Component衍生注解

```markdown
@Repository --> XXXDAO
@Service
@Controller
	注意：本质上这些衍生注解就是@Component 作用，细节以及用法完全一致
	目的：更加准确的表达一个类型的作用。
	
Spring整合Mybatis开发过程中，不适用@Repository和Component。因为spring整合mybatis过程中，不会写XXXDAO的类，DAO的对象也是由MapperScannerConfigurer帮我们动态创建的。 
```

###  2-1@Scope控制简单对象的创建次数

```xml
作用：控制简单对象的创建次数
	注意：不添加@Scope Spring提供的默认值为singleton
<bean id=" " class = " " scope = "single|prototype"/>
```

```java
@Component  //指定默认的id值，类名的首单词首字母小写
@Scope("singleton")
public class Customer {

}
```

```java
@Test
public void test2() {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("/SpringContext.xml");
    Customer user1 = (Customer) ctx.getBean("customer");
    Customer user2 = (Customer) ctx.getBean("customer");
    System.out.println("customer+" + user1);
    System.out.println("customer+" + user2);

}
```

- 结果：

```
customer+com.gewei.scope.Customer@78a773fd
customer+com.gewei.scope.Customer@78a773fd
```

###  2-2 @Lazy延迟创建单实例对象

- 生命周期中：对于单实例对象，spring默认会在工厂创建的同时创建这个对象

```
作用：延迟创建单实例对象
<bean id = " " class = " " lazy = "false"/>	
```

```java
@Component
public class Account {
    public Account(){
        System.out.println("Account.account");
    }
}
```

```java
@Test
public void test3(){
    ApplicationContext ctx = new ClassPathXmlApplicationContext("/SpringContext.xml"); //代表spring工厂被创建了
}
```

输出

```
Account.account
```

- 默认情况下，Spring工厂会直接创建需要加载的对象，如果我们想延迟创建单实例对象，这边可以直接使用@Lazy注解，在使用到需要对象时，才回去创建。

```java
@Component
@Lazy
public class Account {
    public Account(){
        System.out.println("Account.account");
    }
}
@Test
public void test3(){
    ApplicationContext ctx = new ClassPathXmlApplicationContext("/SpringContext.xml");
    Account account = (Account) ctx.getBean("account"); //使用的时候才会创建该对象
}
```

## 3 生命周期方法相关注解

```markdown
1. 初始化相关方法 @PostConstruct
	原来在进行初始化定义的时候，必须实现接口InitalizingBean
	或者在标签中定义<bean init-method = "" />
2. 销毁方法 @PreDestroy
	实现接口DisposableBean
	或者在标签中定义<bean destory-method = ""/>
注意：1.上述的两个注解并不是Spring提供的，而是由JSR520(JavaEE规范)提供的。	
	 2.再一次的验证，通过注解实现了接口的契约性。
```

-  @PostConstruct和@PreDestroy都在javax.annotation包下，这是java的一个扩展包。

```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

```java
@Component
public class Product {
    @PostConstruct
    public void myInit(){
        System.out.println("Product.myInit");
    }
    @PreDestroy
    public void myDestory(){  //只有工厂关闭的时候，该方法才会被调用
        System.out.println("Product.Destory");
    }
}
```

```java
@Test
public void test4(){
    ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/SpringContext.xml");
    ctx.close(); //用于工厂的关闭
}
```

```
Product.myInit
Product.Destory
```

##  4.注入相关注解

1. 用户自定义类型的注入：@Autowired
2. JDK类型

###  4-1@Autowire

传统的配置文件开发

<img src="ssm-imgs\p161-@autowired.png" align="left" />

基于注解开发

<img src="ssm-imgs\p161-@autowired2.png" align="left" />

```java
@Repository
public class UserDaoImpl implements UserDao{
    @Override
    public void save() {
        System.out.println("UserDao.Impl");
    }
}
```

```java
@Service
public class UserServiceImpl implements UserService{
    private UserDao userDao;
    public UserDao getUserDao() {
        return userDao;
    }
    @Autowired
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    @Override
    public void register() {
        userDao.save();
    }
}
```

```java
@Test
public void test5() {
    ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("/SpringContext.xml");
    UserService userServiceImpl = (UserService) ctx.getBean("userServiceImpl");
    userServiceImpl.register();
}
```

- 问题：@Autowired注解为对应的成员变量（userDAO）进行注入的过程中，我们会把注解放在对应的set方法上，此时Spring会帮我们找到对应的DAO（UserDAOImpl）。如果有两个DAO实现类，比如：UserDAOImpl、ProductDAOImpl；此时是怎么保证只注入UserDAO？而不会注入ProductDAO？

1. 基于类型注入：
2. 基于名字进行注入：注入对象的id值必须与Qualifier注解中设置的名字相同。

<img src="ssm-imgs\p163-@autowired-@qualifier.png" align="left" />

####  @AutoWired细节

```markdown
@AutoWired细节
	1.AutoWired注解基于类型进行注入
	基于类型注入：注入对象的类型，必须与目标成员变量类型相同或者是其子类（实现类）
	2.AutoWired 配合Qualifier注解基于名称进行注入  {了解即可}
			注入对象的id值必须与Qualifier注解中设置的名字相同
@Autowired
@Qualifier("userDaoImpl")
public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
}
	3.@AutoWried注解放置位置
	a) 放置在对应成员变量的set方法上
	b) 直接把这个注解放置在成员变量上，Spring通过反射直接对成员变量进行注入（没有调用set方法）【推荐】
	
	4. JavaEE规范中类似功能的注解
	JSR 250 @Resource(name = "userDaoImpl") 基于名字进行注入
	@Autowired
	@Qualifier("userDaoImpl")
	注意：在使用Resource这个注解时，名字没有配对成功，那么会继续按照类型进行注入
	JSR330 @Inject作用@Autowired完全一致，基于类型进行注入【应用不多，一般用在EJB3.0中，了解即可】
	<dependency>
            <groupId>javax.inject</groupId>
            <artifactId>javax.inject</artifactId>
            <version>1</version>
        </dependency>
```

###  4-2@Value

2. JDK类型成员变量注入过程中进行开发的方式：

<img src="ssm-imgs\p166-@value.png" align="left" />

创建出init.properties文件，放在resource目录下

```properties
id = 24
name = gewei
```

在配置文件SpringContext.xml文件下配置：

==`<context:property-placeholder location="classpath:init.properties"/>`==

```java
@Conponent
public class Category{
    @Value("${id}")
    private Integer id;
    @Value("${name}")
    private String name;  
}
//测试
@Test
public void test1(){
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    Category category = (Category)ctx.getBean("category");
    System.out.println("category.getId() = " + category.getId());
}
```

###  @propertySource

- 作用：==替换了spring中 <context:property-placeholder location="classpath:init.properties"/>标签==

```java
@Component
@PropertySource("classpath:/init.properties")
public class Category {}
```

####  @Value使用细节

```
@Value不可以应用到静态成员变量上
	如果应用(赋值)到静态成员变量上，那么会造成注入失败
@Value+properties这种方式，不可以注入集合类型
	这边Spring提供了一种新的配置形式，YAML或YML进行集合的注入（应用在SpringBoot中）
```

<img src="ssm-imgs\p169-@value+properties.png" align="left" />

##  5注解扫描详解

- 问题：spring框架是如何扫描到对应的注解？
- `<context:component-scan base-package="com.jun"/>`   针对于当前包，及其子包（如果不想扫描包中的某些注解，则提供了排除或包含方式）

###  1.排除方式

```xml
<context:component-scan base-package="com.gewei">
    <context:exclude-filter type="" expression=""/>
</context:component-scan>

spring的type提供了5种排除方式，分别为
    1.assignable：排除特定类型不进行扫描
	<context:component-scan base-package="com.gewei">
        <context:exclude-filter type="assignable" expression="com.gewei.bean.User"/>
    </context:component-scan>
    2.anntation：排除特定的注解不进行扫描
	<context:component-scan base-package="com.gewei">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
    </context:component-scan>
    3.aspectj：切入点表达式 【开发中用的较多】
        <context:exclude-filter type="aspectj" expression="com.gewei.bean..*"/> //bean包及其子包
		包切入点：com.gewei.bean..*
		类切入点：*..User
    4.regex：正则表达式
    5.custom：自定义排除策略，一般在框架的底层开发中会用到。
【注意：排除策略可以叠加使用】
<context:component-scan base-package="com.jun">
    <context:exclude-filter type="assignable" expression="com.jun.bean.User"/>
    <context:exclude-filter type="aspectj" expression="com.jun.injection..*"/> 
</context:component-scan>
```

- 测试排除策略

```java
@Test
public void test1(){
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    String[] beanDefinitionNames = ctx.getBeanDefinitionNames();
    for(String beanDefinitionName : beanDefinitionNames){
        System.out.println("beanDefinitionName = " + beanDefinitionName);
    }
}
```

###  2.包含方式

```xml
<context:component-scan base-package="com.gewei" use-default-filters="false"> //让Spring默认的扫描方式失效
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/> //作用：指定扫描哪些注解
</context:component-scan>
【注意：包含策略也可以叠加使用】
```

##  6对于注解开发的思考

- 配置互通

```xml
Spring注解配置与配置文件的配置互通

@Repository
public class UserDAOImpl{}

public class UserServiceImpl{
	private UserDAO userDAO;
	set;  get;
}

<bean id = "userService" class =  "com.gewei.UserServiceImpl"
	<property name = "userDAO" ref = "userDAOImpl"/>  //引用过程中id值要按照@Repository注解所设置的id值进行引用
</bean>
```

- 问题：什么情况下使用注解，什么情况下使用配置文件？
  1. 在程序员开发的类型上，可以加入对应注解，进行对象的创建
  1. 应用于其他非程序员开发的类型时，还是需要`<bean>`进行配置的。如：`SqlSessionFactoryBean`,`MapperScannerConfigurer`

# Spring高级注解（Spring 3.x及以上)

##  1.配置Bean

```java
1.Spring在3.x提供的一个注解，用于替换XML配置文件。

@Configuration
public class Appconfig{}
```

1.配置bean在应用的过程中替换了XML具体什么内容呢？

<img src="ssm-imgs\p175-@configuration.png" align="left" />

2. AnnotationConfigApplicationContext
   1. 创建工厂代码：`AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();`
   2. 指定配置文件（两种方式）：
      1. 指定配置bean的class：`AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);`
      2. 指定配置bean所在的路径（包扫描的方式）：

<img src="ssm-imgs\p175-@AnnotationConfigApplicationContext.png" align="left" />

###  配置Bean开发的细节分析

- 基于注解开发使用日志

```
<dependency>  
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.25</version>
</dependency>
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>1.2.17</version>
</dependency>
使用注解开发，不能集成log4j，更倾向于使用更新的技术；如：logback
org.slf4j 日志门面保留。
```

- 使用logback，引入相关jar包

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.25</version>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>jcl-over-slf4j</artifactId>
  <version>1.7.25</version>
</dependency>

<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.2.3</version>
</dependency>

<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-core</artifactId>
  <version>1.2.3</version>
</dependency>

<dependency>
  <groupId>org.logback-extensions</groupId>
  <artifactId>logback-ext-spring</artifactId>
  <version>0.1.4</version>
</dependency>
```

- 引入 logback 配置文件：logback.xml，设置级别为level="DEBUG"，后续也可以切换成：INFO

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="DEBUG">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

- @Configuration注解的本质：@Configuration也是 `@Component` 的衍生注解，同样的也可以使用<context:component-scan进行扫描。

##  2.@Bean注解

- @Bean注解在bean中进行使用，等同于XML配置文件中的<bean 标签

###  2-1@Bean注解的基本使用

- 对象的创建

1. 简单对象：直接通过new方式创建出的对象。如：User、UserService
2. 复杂对象：不能通过new方式创建出来对象。如：Connection、SqlSessionFactory

<img src="ssm-imgs\p178-@Bean.png" align="left" />  

- 创建复杂对象：不能直接通过new的方式来创建。

```java
@Bean
public Connection conn(){
	Connection conn = null;
	try{
		class.forName("com.mysql.jdbc.Driver");
		conn = DriverManager.getConnection("jdbc:mysql://locolhost:3306/suns?useSSL=false,"root","123456");
	}catch(ClassNotFoundException e){
		e.printStackTrace();
	}catch(SQLException e){
		e.printStackTrace();
	}
	return conn;
}
```

- 创建复杂对象在Spring中通常使用实现FactoryBean的方式

```java
public class ConnectionFactoryBean implements FactoryBean<Connection> {
    @Override
    public Connection getObject() throws Exception {
        Class.forName("com.mysql.jdbc.Driver");
        Connection conn = DriverManager.getConnection("jdbc:mysql://locolhost:3306/suns?useSSL=false","root","root");
        return conn;
    }

    @Override
    public Class<?> getObjectType() {
        return Connection.class;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }
}
```

- 这里我们在配置类中可以这样去实现。(P178没懂)

```java
@Configuration
public class AppConfig{
    //1.简单对象
    @Bean
    public User user(){
        return new User();
    }
    //2.创建复杂对象
    @Bean
    public Connection conn(){
        Connection conn = null;
        try{
            Class.forName("com.mysql.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://locolhost:3306/suns?useSSL=false","root","root");           
        } catch(Exception e){
            e.printStackTrace();
        }
        return conn;
    }
    //3.@Bean和FactoryBean集成的方式：【遗留系统较常见】
	@Bean
	public Connection conn1(){
    	ConnectionFactoryBean factoryBean = new ConnectionFactoryBean();
    	Connection conn = null;
    try {
        conn = factoryBean.getObject();
    } catch (Exception e) {
        e.printStackTrace();
    }
    return conn;
}
}

```

- 自定义id值
  - 注意：@Bean注释的方法名字就是bean的id值（如果我们想自定义id值，只需要在@Bean("u")中命名即可。）

- 控制对象创建次数

```java
<bean id="user" class="com.jun.bean.User" scope="prototype"/>
@Configuration
public class AppConfig{
    @Bean("u")
	@Scope("prototype") 默认值是singleton
    public User user(){return new User();}    
}    
//测试
@Test
public void test3(){
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    User u = (User)ctx.getBean("U");
    User u1 = (User)ctx.getBean("u");
    System.out.println("u1 = " + u1);
    System.out.println("u = " + u);
}
```

###  2-2.@Bean注解的注入

####  自建类型的注入

<img src="ssm-imgs\p182-@Bean.png" align="left" />

```java
@Configuration
public class AppConfig {
    @Bean
    public UserDao userDao(){
        return new UserDaoImpl();
    }

    @Bean
    public UserService userService(UserDao userDao){
        UserServiceImpl userService = new UserServiceImpl();
        userService.setUserDao(userDao);
        return userService;
    }
}

//测试：
@Test
public void test3() {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    UserService userService = (UserService) ctx.getBean("userService");
    userService.register();
}
```

- 这里还有个简化写法（不需要把userDao当做形参了）

```
@Bean
    public UserService userService(){
        UserServiceImpl userService = new UserServiceImpl();
        userService.setUserDao(userDao());
        return userService;
    }
```

####  JDK类型的注入

<img src="ssm-imgs\p183-JDK@Bean.png" align="left" />

#### JDK类型注入的细节分析

首先还是先创建init.properties，然后再这个配置类的内部，对init.properties文件进行读取

```java
如果直接在代码中进行set方法的调用，会存在耦合的问题
@Configuration
@PropertySource("classpath:/init.properties")
public class AppConfig {
    
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
```

##  3.@ComponentScan注解

```markdown
@ComponentScan注解在配置bean中使用，相当于XML文件中的<context:component-scan>标签
目的：进行相关注解的扫描（@Component @Value @Autowired）
```

```java
//XML的配置方式：
<context:component-scan base-package="com.jun"/>
    
@Configuration
@PropertySource("classpath:/init.properties")
//对com.jun.scan包下带有@Component注解的类进行扫描
@ComponentScan(basePackages = "com.jun.scan")
public class AppConfig {}
```

###  3-1排除和包含相关

1. 排除

<img src="ssm-imgs\p186-@ComponentScan.png" align="left" />

2. 包含

<img src="ssm-imgs\p187-@ComponentScan.png" align="left" />

##  4.Spring工厂创建对象的多种配置方式

###  4-1多种配置方式的应用场景

<img src="ssm-imgs\p188-@ComponentScan@Bean.png" align="left" />

###  4-2.配置优先级

- @Component及其衍生注解< @Bean < 配置文件bean标签：
  - 优先级高的配置，可以覆盖优先级低的配置。【配置覆盖：id值必须保持一致】

- 问题：在基于注解的开发环境中，怎么将Spring的配置文件与spring框架进行集成呢？进行导入呢？
  - 原来的集成方式：`new ClassPathXmlApplicationContext("applicationContext.xml");`
  - 基于注解开发的工厂：`ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);`参数是配置Bean的class。此时引入一个新的注解 `@ImportResource(""applicationContext.xml"")`

```java
//基于注解开发,applicationContext.xml配置文件中的内容：
<bean id="user" class="com.jun.bean.User">
	<property name="id" value="2"/>
	<property name="name" value="zhangsan"/>
</bean>	
这边要学习一个新的注解
@Configuration
@ComponentScan(basePackages="com.jun.bean")
@ImportResource("applicationContext.xml")
public class AppConfig{
	@Bean
	public User user(){
		User user = new User();
		user.setId(1);
		user.setName("zhangsan");
		return user;
	}
}
// 通过这个注解我们可以在applicationContext.xml这个配置文件中进行配置，同时只需要在代码中只导入注解类文件。
//测试：
@Test
public void test1(){
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    User user = (User)ctx.getBean("user");
    System.out.println("user.getId() = "+ user.getId());
}
```

- 利用这个方式，我们也可以解决配置Bean中的耦合问题，只需要覆盖即可：（P190没懂）

<img src="ssm-imgs\p190-@ImportResource.png" align="left" />

```markdown
# 这边考虑一个问题，虽然在配置Bean中并没有修改类中的代码，但是在这个Bean中加入了新的注解@ImportResource，这算不算耦合呢？
	解决这个问题我们可以通过，新创建一个配置Bean，然后对其添加@ImportResource注解，然后在加载配置文件的过程中，加载进去
	ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig1.class， AppConfig2.class);
	或者：ApplicationContext ctx = new AnnotationConfigApplicationContext("com.jun");  //com.jun包下都是APPConfigx.class
```

```java
//为了不修改原来的配置（AppConfig1）信息,重新申明一个配置APPConfig2
@Configuration
@ImportResource("SpringContext.xml")
public class AppConfig2 {
}
```

###  4-3整合多个配置信息

- 如果我们只在一个文件中配置Bean，那么这个文件中可能有上千行代码，不利于维护，所以我们对其进行拆分。
- 为什么会有多个配置信息?
  - 拆分多个配置bean的开发，是一种模块化开发的形式，也体现了面向对象中各司其职的设计思想。

<img src="ssm-imgs\p191-mybatis-@Bean.png" align="left" />

- 多配置Bean的整合方式

1. 多个配置Bean的整合
2. 配置Bean和@Component相关注解的整合
3. 配置Bean与Spring.xml配置文件的整合

- 整合多种配置需要关注哪些要点：
  - 如何使多配置信息整合成一个整体
  - 如何实现跨配置的注入


####  1）多个配置Bean的整合

- 如何使多配置信息整合成一个整体
- 如何实现跨配置的注入

#####  ①base-package

<img src="ssm-imgs\p193-多配制Bean信息整合.png" align="left" />

#####  ②@import

- 此外，还可以通过@Import的方式直接在AppConfig上添加注解

```java
@Configuration
@Import(AppConfig2.class)
public class AppConfig1 {
    @Bean
    public UserDao userDao(){
        return new UserDaoImpl();
    }
}

//测试
@Test
public void test4() {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig1.class);
    UserDaoImpl userDao = (UserDaoImpl) ctx.getBean("userDao");
    UserServiceImpl userService = (UserServiceImpl) ctx.getBean("userService");
    System.out.println(userDao);
    System.out.println(userService);
}
```

- 这个@Import注解和XML文件中`<import resource = " ">`标签一样的

##### ③指定多个配置Bean的Class对象：

最后一种方式是在工厂创建时，指定多个配置Bean的Class对象：（用的较少，了解就行）

- `ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig1.class，AppConfig2.class);`

#####  跨配置进行注入

- 此方式适用于应用配置Bean的所有场景(P194没听懂)

<img src="ssm-imgs\p194-跨配置进行注入.png" align="left" />

```markdown
# 在应用配置Bean的过程中，不管使用哪种方式进行配置信息的汇总，其操作方式都是通过成员变量加@AutoWired注解完成的
```

#### 2）配置Bean与@Component相关注解的整合

<img src="ssm-imgs\p195-配置Bean与@Component相关注解的整合.png" align="left" />

整合后对配置Bean进行DAO注入：

直接使用@AutoWired进行注入

```java
@Configuration
@ComponentScan(basePackages="com.jun.injection")
public class AppConfig3{
    @Autowired //把需要注入的内容作为成员变量
    private UserDAO userDAO;
    @Bean
    public UserService userService(){
        UserServiceImpl userService = new UserServiceImpl();
        userSercvice.setUserDAO(userDAO);
        return userService;
    }
}
```

####  3） 配置Bean与配置文件整合

<img src="ssm-imgs\p196-配置Bean与配置文件的整合.png" align='left' />

```markdown
# 注意在配置Bean注入的过程中，使用@AutoWried这个方式可能会出现一个红色的下波浪线警告，这是因为idea这个工具联想不到，属于正常情况。
```

##  5.配置Bean的底层实现

(p197没懂)

<img src="ssm-imgs\p197-配置Bean的底层实现原理.png" align="left" />

```markdown
# Spring在配置Bean中加入@Configuration注解后，底层会通过Cglib的方式，来进行相关的配置、处理。
```

##  6.四位一体的开发思想

1. 什么是四维一体

- Spring开发一个功能的4种形式，虽然开发的方式不同，但是最终的效果是一样的。
  - 基于schema
  - 基于特定功能注解 `@PropertySource` 【推荐】
  - 基于原始 `<bean id="" class="PropertySourcePlaceholderConfigure"/>`
  - 基于@Bean注解  【推荐】

<img src="ssm-imgs\p198-四位一体的开发思想.png" align="left" />

##  7.纯注解版AOP编程

- 搭建环境

1. 应用配置Bean
2. 注解扫描

###  开发步骤

```java
1. 原始对象
    @Service
    public class UserServiceImpl implements UserService{}
2. 创建切面类（额外功能、切入点、组装切面）
    @Aspect
    @Component 
    public class MyAspect { 
        @Pointcut("execution(* com.jun.aop..*.*(..))")
        public void pointCut(){}
        @Around("pointCut()")  // 额外功能 + 切入点
        public Object around(ProceedingJoinPoint joinPoint) throws Throwable{
            System.out.println("------aspectj log ----");
            Object ret = joinPoint.proceed();
            return ret;
        }
    }
3.Spring配置文件中配置一个特殊的标签：
    <aop:aspectj-autoproxy> 和 @EnableAspectjAutoProxy等价（写在配置Bean当中）
        
	@Configuration
	@ComponentScan(basePackages = "com.jun.aop")
	@EnableAspectJAutoProxy
	public class AppConfig {
	}
```

###  AOP细节分析

####  @EnableAspectjAutoProxy

```markdown
1. 创建代理的切换 (底层动态代理的创建方式有两种：JDK CGlib)
<aop:aspectj-autoproxy proxy-target-clsss = true|false>  //true->cglib ；false(默认)->jdk
基于纯注解：
@Configuration
@ComponentScan(basePackages = "com.jun.aop")
@EnableAspectjAutoProxy(proxyTargetClass = true) //将jdk代理创建的方式切换成了cglib创建方式
public class AppConfig{}

2.SpringBoot AOP的开发方式
	@EnableAspectjAutoProxy已经设置好了
只需要：
	1.创建原始对象
	2.创建切面类（额外功能、切入点、组装切面）
# Spring AOP代理的默认实现是JDK，而SpringBoot AOP代理的默认实现是 Cglib
```

<img src="ssm-imgs\p200-springboot针对于AOP开发过程中的一段源码.png" align="left" />

##  8.纯注解版Spring+MyBatis整合

<img src="ssm-imgs\p201-纯注解版spring与mybatis整合-连接池.png" align="left" />

- 注意：上面的public DruidDataSource dataSource() 中的返回值最好定义成：`public DataSource dataSource`

<img src="ssm-imgs\p201-纯注解版spring与mybatis整合-SqlSessionFactoryBean.png" align="left" />

### @MapperScan

<img src="ssm-imgs\p201-纯注解版spring与mybatis整合-MapperScannerConfigurer.png" align="left" />

###  纯注解spring+mybatis整合编码

<img src="ssm-imgs\p202-纯注解spring+mybatis整合编码.png" align="left" />

###  MapperLocations编码时通配的写法

####  ResourcePatternResolver

```java
//设置Mapper文件的路径
sqlSessionFactoryBean.setMapperLocations(Resource..);
Resource resource = new ClassPathResource("UserDAOMapper.xml");

sqlSessionFactoryBean.setMapperLocations(new ClassPathResource("UserDAOMapper.xml"));


<property name = "mapperLocations">
	<list>
		<value> classpath:com.jun.mapper/*Mapper.xml</value>
	<list>
</property>
在配置Bean中可以这样书写:（将一组Mapper文件传递给sqlSessionFactoryBean）【开发中的标准写法】
ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
Resource[] resources = resolver.getResources("com.jun.mapper/*Mapper.xml");
sqlSessionFactoryBean.setMapperLocations(resources);
```

<img src="ssm-imgs\p203-mapperLocations通配写法.png" align="left" />

##  解决配置Bean数据耦合问题

<img src="ssm-imgs\p204-解决配置Bean数据耦合的问题.png" align="left" />

然后在配置Bean中将MybatisProperties注入。

<img src="ssm-imgs\p204-解决配置Bean数据耦合的问题2.png" align="left" />

##  9.纯注解版事务编程

<img src="ssm-imgs\p205-纯注解版事务编程.png" align="left" />

- 目前还无法完成注解版控制器开发（注解版MVC整合）

##  10.Spring框架中YML的使用

- YML(YAML)是一种新形式的配置文件，比XML更简单，比 properties 更强大。

- properties配置文件存在问题

1. properties表达过于繁琐，无法表达数据的内在联系。
2. properties无法表达对象 集合类型 。

<img src="ssm-imgs\p206-spring与YAML整合分析.png" align="left" />

- YML语法

```markdown
1. 定义yml文件：xxx.yml、xxx.yaml
2. 语法：
   1. 基本语法：
   name: jun
   password: 1324
   2.对象概念
   account:
   	id: 1
   	password: 14324
   3.定义集合(如List集合)
   service:
   	- 1111
   	- 2222 
```

###  Spring与YML文件整合

```
# 首先Spring是不支持YML文件的
首先是YamlPropertiesFactoryBean这个创建复杂对象的工厂将YML文件创建成properties文件，然后交给PropertySourcePlaceholderConfigurer进行处理。
```

<img src="ssm-imgs\p208-spring与YML集成思路的分析.png" align="left" />

###  Spring与YML集成编码

1. 环境搭建：引入snakeyaml的jar包（因为`YamlPropertiesFactoryBean`对yml进行相应的解析处理）【注意最低version为1.18】

```xml
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.28</version>
</dependency>
```

2. 编码

```java
1.准备yml配置文件
2.配置Bean中操作，完成YAML读取，与PropertySourcePlaceholderConfigurer的创建
  
@Configuration
@ComponentScan(basePackages = "com.jun.yml")
public class YmlAutoConfiguration {

    @Bean
    public PropertySourcesPlaceholderConfigurer configurer(){
        YamlPropertiesFactoryBean yamlPropertiesFactoryBean = new YamlPropertiesFactoryBean();
        yamlPropertiesFactoryBean.setResources(new ClassPathResource("init.yml"));
        Properties properties = yamlPropertiesFactoryBean.getObject();
        PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
        configurer.setProperties(properties);
        return configurer;
    }
}
3.在特定的类上加@Value注解 
@Component
public class Account {
    @Value("${name}")
    private String name;
    @Value("${password}")
    private String password;
    set…… get……
}
//测试
@Test
public void test(){
    ApplicationContext ctx = new AnnotationConfigApplicationContext(YmlAutoConfiguration.class);
    Account account = (Account) ctx.getBean("account");
    System.out.println("account.getName() = " + account.getName());
    System.out.println("account.getPassword() = " + account.getPassword());
}
//如果yml配置文件中是对象的形式：则：
@Value("${account.name}")
```

####  Spring与YML集成的问题

```markdown
age =
    -11
    -12
    -13
Spring的YamlPropertiesFactoryBean还是解决不了这个问题
   # 利用Spring的EL表达式进行解决
list: 11,12,13
    
    @Value(#{${age}.split(',')})
    private List<String> list;
//测试：
@Test
public void test(){
    ApplicationContext ctx = new AnnotationConfigApplicationContext(YmlAutoConfiguration.class);
    Account account = (Account) ctx.getBean("account");
   	List<String> list = accout.getList();
   	for(String s : list){
   		System.out.println("s = " + s);
   	}
}
2.对象类型的YAML配置时，@Value("${accout.name}")过于繁琐
	# springBoot 提供的@ConfigrationProperties更好的解决了这个问题
```