2021年11月19日23:51:20 start

#  springboot

UP：编程不良人

#  数据库

1. 名称：ssm，基字符集：utf8mb4（和utf8的区别是：它可以存表情等符号）

```sql
CREATE TABLE emp(
	`id` INT(11) PRIMARY KEY AUTO_INCREMENT,
	`name` VARCHAR(40),
	`birthday` TIMESTAMP,
	`salary` DOUBLE(10,2)
);
```

#  相关依赖

1. new project =》选择空项目=》选择电脑上的文件夹
2. spring相关的依赖：`spring-core、beans、aop、context、context-support、jdbc、web` 版本都是5.3.6（无论选择哪个 版本，spring相关依赖建议都设置一样的）groupid：org.springframework
3. springwebmvc相关依赖：spring-webmvc 5.3.6
4. mybatis：3.5.6、groupid：org.mybatis  ；mybatis整合spring：mybatis-spring：2.0.6
5. mysql-connector-java  5.1.38  ;  druid:1.2.4
1. 转换json：com.fasterxml.jackson.core：jackson-databind：2.9.3
1. 切面相关的依赖org.aspectj：aspectjrt、aspectjweaver：1.9.5

#  spring整合mybatis

1. ==问题一：避免父子容器的问题，只扫描service？什么是父子容器？==
2. resouces文件夹的创建必须使用 / ，而不能是`.` 否则会出现问题。
3. 为避免输入数据库的数据有问题：设置characterEncoding=utf-8
4. 开启注解事务生效：一定要是tx结尾

```xml
<!--1.开启注解扫描：避免父子容器的问题，只扫描service？什么是父子容器？--> 
<!--2.创建数据源-->
<!--3.创建sqlSessionFactory-->
	<!--mapper配置文件的位置-->
	<!--实体别名 类名 类名小写-->
<!--4.创建DAO-->
<!--5.创建事务管理器-->
<!--6.开启注解事务生效：业务层的@Transactional-->
```

##  xxxMapper.xml

```xml
<!--已经起了别名，所以insert标签里的parameterType可是使用别名-->
<!--明确插入的时候要声明主键，所以需要添加useGeneratedKeys="true"，使用mysql中的主键默认生成
只有在mysql数据库中生效。声明的主键赋给：Emp中的id-->
<insert id="save" parameterType="Emp" useGeneratedKeys="true" keyProperty="id">
```

- EmpServiceImpl

```java
@Service
@Transactional
public class EmpServiceImpl implements EmpService{
    //@Autowired //官方不推荐这样写
    private EmpDAO empDAO;

    @Autowired //官方推荐的写法，必须加构造器
    public EmpServiceImpl(EmpDAO empDAO) {
        this.empDAO = empDAO;
    }
```

#  springmvc.xml

```xml
<!--1.开启注解扫描：此时的springmvc是子容器，子工厂，所以只扫描controller-->
<!--2.开启mvc注解驱动-->
<!--3.配置视图解析器-->
 
```

#  web.xml

```xml
<!--加载spring.xml-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring.xml</param-value>
  </context-param>
  <!--1.配置spring工厂启动-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>

  <!--配置springmvc-->
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

- 遇到的问题：

```java
错误: 代理抛出异常错误: java.rmi.server.ExportException: Port already in use: 1099; nested exception is: 
	java.net.BindException: Address already in use: JVM_Bind
Caused by: java.rmi.server.ExportException: Port already in use: 1099; nested exception is: 
	java.net.BindException: Address already in use: JVM_Bind
Caused by: java.net.BindException: Address already in use: JVM_Bind
    
1、可以查出占用的端口，然后杀掉
netstat -aon|findstr 1099
taskkill -f -pid 3756
```

- 解决日期输出的格式问题：

   @JsonFormat(pattern = "yyyy-MM-dd")
      private Date birthday;

- `http://localhost:8080/ch01_ssm/save?name=xiaoming&birthday=2020/12/12&salary=23.34`

2021年11月21日00:02:25

#  现有SSM开发存在的问题

1. 大量maven臃余配置
2. 每次构建项目都要书写大量相同配置极大浪费了项目开发时间
3. 每次整合第三方技术都需要编写相关配置文件
4. 项目测试每次都需要部署到tomcat

#  Springboot

>springboot(微框架)  = springmvc（控制器） + spring core（项目管理）

- 优势：
  - 创建完整独立的spring应用程序
  - 嵌入的tomcat，无需部署war文件
  - 简化maven配置，自动配置spring springmvc，没有xml配置

- 手工创建springboot

##   springboot 的约定

<img src="ssm-imgs\springboot-appointment.png" align="left" />

>创建springboot：

1. pom文件引入依赖
2. resources生成application.yml
3. 创建入口类加入@SpringBootApplication注解，在main中启动应用

##  引入相关依赖

```xml
<!--继承springboot的父项目:便于维护版本-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
</parent>

<dependencies>
    <!--web依赖：spring、springmvc-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

##  相关注解说明

- ch02Application.java

```java
@SpringBootApplication //这个注解修饰范围:用在类上，只能用在入口类上，只能出现一次
//标识这个类是一个springboot入口类，启动整个springboot项目总入口
public class ch02Application {
    public static void main(String[] args) {
        //启动springboot应用，参数1：指定入口类的类对象.class（类的对象：new出来的）；参数2：main函数的参数
        SpringApplication.run(ch02Application.class, args);
    }
}

/**
 * 1）@SpringBootApplication：注解
 *  @Target：指定注解作用范围，@Retention:指定注解什么时候有效。
 *  @SpringBootConfiguration:这个注解就是用来配置spring、springmvc的相关环境
 *  @EnableAutoConfiguration:开启自动配置核心注解，自动配置spring相关环境，
 *  自动与项目中引入第三方技术自动配置其环境
 *  @ComponentScan:组件扫描，根据注解发挥注解作用，默认扫描当前包及其子包。
 *
 * 2）args：启动springboot应用时候需要传递main函数参数作为启动的第二个参数
 *     在VM options处输入参数： -Dserver.port=8082可以修改默认的端口号
 */
```

- HelloController.java

```java
@RestController
public class HelloController {
    //测试控制器
    //注意：springboot项目默认启动没有项目名：http://localhost:8080/hello
    @RequestMapping("hello")
    public String hello() {
        System.out.println("hello springboot!");
        return "Hello SpringBoot!";
    }
}
```

- application.yml

```yaml
server:
  port: 8081 # 修改内嵌服务器的端口号
  servlet:
    context-path: /ch02_springboot #修改项目名（项目名必须以/开头）http://localhost:8081/ch02_springboot/hello
```

##  配置文件拆分以及加载外部配置

[【编程不良人】2021年SpringBoot第6集](https://www.bilibili.com/video/BV1Cv411p7cQ?p=6)

<img src="ssm-imgs\devprod.png" align="left" />

- application.yml

```yaml
#公共配置
server:
  port: 8080
# 激活哪个环境配置
spring:
  profiles:
    #active: dev  # 让dev环境生效
    active: prod # 让prod环境生效
```

- application-dev.yml & application-prod.yml

```yaml
server:
  servlet:
    context-path: /ch02_springboot  #测试项目名
    
server:
  servlet:
    context-path: /springboot  #生产项目名    
```

- **企业中**：springboot还可以单独引入一个外部配置文件：里面有各种重要的配置（生产配置）文件
- 在启动界面：Program arguments 里写入：--spring.config.location=文件的绝对路径

<img src="ssm-imgs\localprodyml.png" align="left" />

- `java -jar --spring.config.location=绝对路径  xxx.jar`

>自动创建项目

1. New Moudle ：Spring Initializr：Name：ch03-springboot2、Location：D:\Java_code\springboot1\ch03-springboot2、

2. Group：com.jun、Artifact：ch03-springboot2Package name：com.jun、Java：8

3. Dependencies：Web：Spring Web

4. pom.xml文件中的插件：打jar包的时候一定要有这个插件，自己运行的时候不需要这个插件。

##  工厂创建对象

1. 基于配置文件形式创建对象 spring.xml
   1. <bean id="工厂中名称为：类名首字母小写" class="xxx.xxServiceImpl"
2. 基于注解方式创建对象
   - @Component：在工厂中创建对象，通用对象创建注解
     - @Controller、@Service、@Repository（创建dao层注解）

>SpringBoot中如何创建对象？

1. 使用原始spring框架中注解创建对象：@Component、@Controller、@Service、@Repository：(直接加在类上)**只能创建单个对象。**
2. 使用配置方式创建对象：**可以创建多个对象**
   1. ==`@Configuration`==：修饰范围，用在**类**上。（相当于以前写的spring.xml配置文件）
      1. 作用：代表这个类是一个springboot中的配置类。
   2. ==`@Bean`== 注解：创建对象，相当于 spring.xml 书写 bean

```java
@Configuration
public class BeanConfig{
    @Bean
    public User user(){
        return new User();
    }
    @Bean
    public Emp emp(){
        return new Emp();
    }
    .......
}
```

- DemoServiceImpl.java

```java
@Service("demoService") //修饰范围：用在类上 作用：在工厂中创建对象，
// 默认：工厂中名称为：类名首字母小写demoServiceImpl
//value属性：用来指定当前创建对象在工厂中的名称。
public class DemoServiceImpl implements DemoService{
    @Override
    public void demo() {
        System.out.println("demo Service ok");
    }
}
```

- DemoController.java

```java
@RestController
public class DemoController {
    @Autowired  //如果想用注解去注入的话，先得在工厂中创建
    @Qualifier(value = "demoService") //作用：用来修改autoWired默认不再根据类型注入，修改为根据名称注入
    private DemoService demoService;

    @Autowired
    private Calendar calendar;

    @RequestMapping("demo")
    public String demo() {
        System.out.println("demo ok");
        demoService.demo(); //如果工厂中没有创建demoService的话，就不能调用demo方法
        //Calendar instance = Calendar.getInstance();没有交给springboot创建之前，每次用都要调用
        //Calendar是在java.util包下，不能加注解，所以通过创建多个对象的方式创建。
        System.out.println("工厂中获取日历对象，当前时间为：" + calendar.getTime());
        return "demo ok";
    }
}
```

- BeanConfig.java

```java
@Configuration //修饰范围：只能用在类上，作用：代表这个类是一个配置类 ==》相当于spring.xml
public class BeanConfig {
    @Bean //修饰范围：用在方法或者注解上，作用：用来将方法返回值交给工厂管理
    //方法名：推荐返回值首字母小写，代表当前创建对象在工厂中的名称
    public Calendar calendar() {
        return Calendar.getInstance();
    }
}
```

##  属性注入

>spring 框架中属性注入

- 引用类型注入：
  - @Autowired：spring框架提供，默认根据类型注入。
  - @Resource：javaEE规范提供，默认根据名称注入，如果名称找不到自动根据类型注入
  - 以上两种注解可以用在成员变量上，也可以用在成员变量的set方法上

- 八种基本类型+String类型+日期类型 value + 数组 + set list map标签集合类型属性注入：

```xml
<bean>
	<property name="name" value="xiaochen"/>
</bean>

spring框架也提供了注解的方式
@Value("xiaochen")
private String name;
```

>SpringBoot升级

- 属性注入：
  - @Autowired、@Resource、@Value

- 注入mep集合时，配置文件中要使用json格式复制，在注入时必须使用"#{${xxx}}" 进行获取

```java
@RestController
public class InjectionController {
//    @Value("xiaochen") //但是这样写就写死了，所以要去配置文件中配置
    @Value("${name}")
    private String name;
    @Value("${salary}")
    private Double salary;
    @Value("${birthday}")
    private Date birthday;
    @Value("${sex}")
    private Boolean sex;
    @Value("${arr}")
    private String[] arr;
    @Value("${lists}")
    private List<String> lists;
    @Value("#{${maps}}")
    private Map<String,String> maps;

    @RequestMapping("inject")
    public String inject() {
        System.out.println("inject ok");
        System.out.println("name = " + name);
        System.out.println("birthday = " + birthday);
        System.out.println("遍历list");
        lists.forEach(li -> System.out.println("li = " + li));
        System.out.println("遍历数组");
        for(String arrs : arr){
            System.out.println("arr = " + arrs);
        }
        System.out.println("遍历Map");
        maps.forEach((key,value) -> System.out.println("key = " + key + " value = " + value));
        return "inject ok";
    }
}
```

- application.yml

```yaml
server:
  port: 8080
  servlet:
    context-path: /springboot2

#声明注入值
name: 小陈
salary: 100.23
birthday: 2012/12/12 12:13:12  #默认的日期格式为 yyyy/mm/dd  HH:MM:ss
sex: true
arr: 23,32,45  #注入数组的时候，多个元素用"," 逗号隔开
lists: xiaosong,xiaochen,xiaohong
maps: "{'aa':'小红','bb':'xiaozhang'}"
#注入map集合可以使用json形式进行注入，注意使用@Value注入时必须加入：#{${属性}}
```

###  @ConfigurationProperties

- 使用这个注解为属性一次性赋值，必须为属性提供SET、get方法

```java
@RestController
@ConfigurationProperties("orders") //修饰范围：用在类上，
// 作用：用来指定前缀的属性，注入到当前对象中属性名一致的属性中
public class InjectObjectController {
    private Integer id;
    private String name;
    private Double price;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    @RequestMapping("injectObject")
    public String injectObject() {
        System.out.println("injectObject ok!");
        System.out.println("id = " + id);
        System.out.println("name = " + name);
        System.out.println("price = " + price);
        return "injectObject ok!";
    }
}
```

- application.yml

```yml
#声明对象方式注入
orders:
  id: 21
  name: xiaohua
  price: 23.34

alipay:
    key: xxx
    secrect: xxx

# 配置jsp视图前缀和后缀
spring:
  mvc:
    view:
      prefix: /
      suffix: .jsp
```

##  JSP模板集成

- 引入jsp的集成jar包：jstl、jstl、1.2  / org.apache.tomcat.embed、tomcat-embed-jasper (让内嵌tomcat具有解析jsp功能)

- 配置jsp视图前缀和后缀：
- IDEA中springboot项目有一个小问题：找不到.jsp文件：
  - 解决办法是：EditConfigurations-》WorkingDirectory-》$MODULE_DIR$
  - 解决办法二：配置artifactID：pring-boot-maven-plugin，groupID：org.springframework.boot
  - 通过插件：spring-boot：run启动
- 修改jsp无需重启应用：作用：修改jsp页面无需重启springboot应用

```yaml
# 修改jsp无需重启应用
server:
    servlet:
      jsp:
        init-parameters:
          development: true
```

```java
@Controller
public class JSPController {
    @RequestMapping("jsp")
    public String jsp() {
        System.out.println("jsp ok");
        return "index";
    }
}
```

##  整合mybatis思路分析

- `17分26秒`
- springboot 微框架 = spring 工厂 + springmvc控制器
- 数据库访问框架：hibernate、jpa、mybatis
- 

##  整合mybatis编码实现

###  环境搭建：

1. 引入依赖：
   - spring-boot-starter-web
   - mysql相关（druid,1.2.4/mysql-connector-java 5.1.38）
   - mybatis相关（mybatis-spring-boot-stater 2.1.4）(相当于整合了mybatis、mybatis-spring)
   - 注意：springboot命名规范：ch04springboot-day1；com.package
2. application.yml
3. 在入口文件Ch04IntegrationApplicaiton.java处加@MapperScan(value="com.ju.dao")注解创建所有dao

```yaml
server:
  port: 8989
  servlet:
    context-path: /springboot_integration  #指定应用名称

# 1.整合mybatis相关配置
# 数据源
spring:
    datasource:
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: com.mysql.jdbc.Driver # mysql5.x版本驱动
      url: jdbc:mysql://localhost:3306/ssm?useSSL=false&characterEncoding=UTF-8
      username: root
      password: hrj
# 指定mapper配置文件的位置、相当于sqlSessionFactory
mybatis:
  mapper-locations: classpath:com/jun/mapper/*.xml
  type-aliases-package: com.jun.entity # 指定实体类的包名，默认别名：类名，类名首字母小写
```

###  @MapperScan

- 修饰范围：用在类上，作用：用来扫描dao接口所在的包，同时将所有dao接口在工厂中创建对象。

```java
@SpringBootApplication
@MapperScan(value="com.jun.dao") //修饰范围：用在类上，作用：用来扫描dao接口所在的包，同时将所有dao接口在工厂中创建对象。
public class Ch04IntegrationApplication {
    public static void main(String[] args) {
        SpringApplication.run(Ch04IntegrationApplication.class, args);
    }
}
```

###  建表

```sql
CREATE TABLE `user`(		 
  `id` INT(11) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `name` VARCHAR(40) COMMENT '姓名',
  `birthday` TIMESTAMP COMMENT '生日',
  `salary` DOUBLE(10,2) COMMENT '工资',
  PRIMARY KEY (`id`) 
);
```

