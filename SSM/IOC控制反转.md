#  Spring5

#  传统方式

>耦合度太高：UserDAO的路径和方法发生改变，UserService要跟着变

```java 
class UserService{
    execute(){
        UserDAO dao = new UserDAO();
        dao.add();
    }
}
class UserDAO{
    add(){
        ……………………
    }
}
```

>:pencil:解耦合：`工厂模式`

```java 
class UserService{
    execute(){
        UserDAO dao = UserFactory.getDao();
        dao.add();
    }
}
class UserDAO{
    add(){……………………}
}
class UserFactory{//工厂类：降低了UserService和DAO之间的耦合度
    public static UserDAO getDao(){
        return new UserDAO();
    }
}
```

>:pencil:进一步降低耦合度：`IOC`
>
>1. xml配置文件，配置创建的对象。<bean id="dao" class="com.jun.dao.UserDao"></bean>
>2. 第二步：有service类和dao类，创建工厂类
>3. IOC 过程中，如果文件地址改变了，只需要改变xml配置文件中<bean class=""> 的地址。

```java 
class UserFactory{
    public static UserDAO getDao(){
        String classValue = class属性值；//1.xml解析
        Class clazz = Class.forName(classValue); //2.通过反射创建对象
        return (UserDAO)clazz.newInstance();
    }
}
```

#  IOC容器

- 控制反转，把对象创建和对象的调用过程交给spring进行管理。

- 1.IOC思想基于IOC容器完成，IOC容器底层就是对象工厂
- 2.Spring提供IOC容器实现的两种方式：（2个接口）
  - (1).`BeanFactory`，IOC容器基本实现，是Spring内部的使用接口，不提供给开发人员。
    - 加载配置文件的时候不会创建对象，只有在获取对象(使用)的时候才会创建对象。
  - (2).`ApplicationContext`是BeanFactory子接口，功能强大，开发人员可使用。
    - 加载配置就会创建对象。 （比如在web项目中，把耗时耗资源的操作都在项目启动的时候进行处理，在服务器启动过程中就去创建对象。）
- 3.`ApplicationContext`有一些特别的实现类：
  - (1).`ClassPathXmlApplicationContext`，在src目录下可以写文件名
  - (2).`FileSystemXmlApplicationContext`，在src目录下，必须写绝对路径

##  IOC基于xml操作Bean管理

> Bean管理

- Bean管理是指两个操作：**Spring创建对象** 和 **Spring注入属性**
- Bean管理有两种操作方式：**基于xml配置文件方式实现** 和 **基于注解方式实现**

##  基于xml方式注入属性

>1.使用set方法进行注入

>2.使用有参数构造进行注入

```java 
//（1）创建类，定义属性，创建属性对应有参数构造方法
public class Orders {
     private String oname;
     private String address;
     //有参数构造
     public Orders(String oname,String address) {
         this.oname = oname;
         this.address = address;
     }
}
//（2）在 spring 配置文件bean1.xml中进行配置
<!--有参数构造注入属性-->
<bean id="orders" class="com.atguigu.spring5.Orders">
 <constructor-arg name="oname" value="小米11"></constructor-arg>
 <constructor-arg name="address" value="China"></constructor-arg>
</bean>
```

#  Spring工厂创建复杂对象

##  FactoryBean接口

>是Spring 中用于创建复杂对象的⼀种方式，也是 Spring **原生提供的**，后续 Spring 整合其他框架时会大量应用 FactoryBean 方式。

1. 添加mysql依赖

2. 配置文件：mysql 高版本连接创建时，需要制定`SSL` 证书，否则会警告；?useSSL=false

   ```xml
   <bean id="conn" class="com.jun.factorybean.ConnectionFactory">
       <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
       <property name="url" value="jdbc:mysql://localhost:3306/book?useSSL=false"/>
       <property name="username" value="root"/>
       <property name="password" value="hrj"/>
   </bean>
   ```

```java
public class ConnectionFactory implements FactoryBean {
    private String driverClassName;
    private String url;
    private String username;
    private String password;
    set /get 方法
    @Override //用于书写创建复杂对象的代码
    public Object getObject() throws Exception {
        Class.forName(driverClassName);
        Connection connect = DriverManager.getConnection(url, username, password);
        return connect;
    }
    @Override //返回创建的复杂对象的class对象
    public Class<?> getObjectType() {
        return Connection.class;
    }
    @Override
    public boolean isSingleton() {
        return false;//返回false，每次调用都需要创建一个新的复杂对象
    }
} 
```

>如果就想获得 `FactoryBean` 类型的对象，`ctx.getBean("&conn")`

```java
public class TestFactoryBean {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        Connection conn = (Connection) ctx.getBean("conn");
        //如果就想获得 FactoryBean 类型的对象，加个 &，ctx.getBean("&conn")
        ConnectionFactory bean = (ConnectionFactory) ctx.getBean("&conn");
        System.out.println("conn = " + conn);
        System.out.println("bean = " + bean);
    }
}
```

###  配置文件参数

><!--resources 下的文件在整个程序编译完后会被放到 classpath 目录下，src.main.java中的文件也是-->

```java 
//jdbc.properties
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/book?useSSL=false
jdbc.username=root
jdbc.password=hrj
//applicationContext.xml
<context:property-placeholder location="classpath:/jdbc.properties"/>    
<bean id="conn" class="com.jun.factorybean.ConnectionFactory">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
</bean>    
```

##  实例工厂&静态工厂

[笔记1](https://luzhenyu.blog.csdn.net/article/details/106292604)    [笔记2+代码](https://blog.csdn.net/qq_36903261/article/details/108974033)

#  JDBCTemplate

2021年10月25日10:13:22

>Spring 框架对 JDBC 进行封装，使用 JdbcTemplate 方便实现对数据库操作。

1. 引入相关jar包：druid/mysql-connector-java-bin/spring-jdbc/spring-orm/spring-tx
2. 在 spring 配置文件配置数据库连接池。
3. 配置 JdbcTemplate 对象，注入 DataSource。
   1. 根据源码发现，JdbcTemplate并不是根据有参构造注入，而是根据setDataSource方法注入
4. 创建 service 类，创建 dao 类，在 dao 注入 jdbcTemplate 对象。
