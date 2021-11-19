#  Mybatis

>MyBatis开发步骤：第2步实体别名：**配置繁琐**，第6步注册Mapper文件 ：**配置繁琐**、第7步MyBatisAPI调用**代码冗余**

1. 实体
2. 实体别名
3. 创建数据库表
4. 创建DAO接口
5. 实现Mapper文件
6. 注册Mapper文件
7. MyBatisAPI调用

>首先在pom文件中引入jar包

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.48</version>
</dependency>
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>3.4.6</version>
</dependency>
```

>1.实体类

```java
public class User implements Serializable {
    private Integer id;
    private String name;
    private String password;
```

>2.引入框架对应的文件：实体别名 `mybatis-config.xml`

```xml
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Confi 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <typeAlias alias="user" type="com.jun.mybatis.User"/>
    </typeAliases>

    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/book?useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="hrj"/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

>3.创建t_mybatis表

```sql
net start mysql
mysql -u root -p hrj
show databases;
use book;
create table t_mybatis(`id` int primary key auto_increment,`name` varchar(12),`password` varchar(12));
desc t_mybatis;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int(11)     | NO   | PRI | NULL    | auto_increment |
| name     | varchar(12) | YES  |     | NULL    |                |
| password | varchar(12) | YES  |     | NULL    |                |
+----------+-------------+------+-----+---------+----------------+
```

>4.创建 DAO 接口：`UserDAO`

```java 
public interface UserDAO {
    void save(User user);
}
```

>5.实现Mapper文件：`UserDAOMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.jun.mybatis.UserDAO">
    <insert id="save" parameterType="user">
        insert into t_mybatis(name, password) values (#{name}, #{password})
    </insert>
</mapper>
```

>6.注册 Mapper 映射文件 `mybatis-config.xml`

```xml
<mappers>
    <mapper resource="UserDAOMapper.xml"/>
</mappers>
```

>7.MybatisAPI 调用

```java
public class TestMyBatis {
    public static void main(String[] args) throws IOException {
        InputStream ins = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(ins);
        SqlSession session = sqlSessionFactory.openSession();
        UserDAO userDAO = session.getMapper(UserDAO.class);
        User user = new User();
        user.setName("jun");
        user.setPassword("1324");
        userDAO.save(user);
        //提交事务
        session.commit();
    }
}
```

#  Spring 与 Mybatis 整合

>搭建开发环境

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-jdbc</artifactId>
  <version>5.2.6.RELEASE</version>
</dependency>

<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis-spring</artifactId>
  <version>2.0.4</version>
</dependency>

<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid</artifactId>
  <version>1.1.12</version>
</dependency>
```

>Spring配置文件的配置
>
>- <!--classpath是Spring内置的一个关键字，它代表的是src-->

```xml
<!--1.连接池-->
    <!-- 引入外部properties文件 -->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
```

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/book?useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="hrj"/>
    </bean>

    <!--创建SqlSessionFactory SqlSessionFactoryBean-->
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 指定实体类所在的包 -->
        <property name="typeAliasesPackage" value="com.jun.entity"/>
        <!--指定配置文件（映射文件）的路径，还有通用配置-->
        <property name="mapperLocations">
            <list>
                <value>classpath:com.jun.mapper/*Mapper.xml</value>
            </list>
        </property>
    </bean>

    <!--创建DAO对象 MapperScannerConfigure-->
    <bean id="scanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryBean"/>
        <!--指定DAO接口放置的包-->
        <property name="basePackage" value="com.jun.dao"/>
    </bean>
</beans>
```

>1-在entity包中创建实体：User

```java
public class User implements Serializable {
    private Integer id;
    private String name;
    private String password;
```

>2-创建t_mybatis表

>3-创建dao包，并创建UserDAO接口

```java 
public interface UserDAO {
    public void save(User user);
```

>4-Mapper配置文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/book?useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="hrj"/>
    </bean>

    <!--创建SqlSessionFactory SqlSessionFactoryBean-->
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 指定实体类所在的包 -->
        <property name="typeAliasesPackage" value="com.jun.entity"/>
        <!--指定配置文件（映射文件）的路径，还有通用配置-->
        <property name="mapperLocations">
            <list>
                <!--classpath是Spring内置的一个关键字，它代表的是src-->
                <value>classpath:com.jun.mapper/*Mapper.xml</value>
            </list>
        </property>
    </bean>

    <!--创建DAO对象 MapperScannerConfigure-->
    <bean id="scanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryBean"/>
        <!--指定DAO接口放置的包-->
        <property name="basePackage" value="com.jun.dao"/>
    </bean>
</beans>
----------------------------------------------------------------------------------
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jun.dao.UserDAO">
    <insert id="save" parameterType="User">
        insert into t_mybatis(name, password) values (#{name}, #{password})
    </insert>
</mapper>
```

>测试

```java
public class TestMybatisSpring {
    //用于测试Spring整合Mybatis
    @Test
    public void test() {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("/mybatis-spring.xml");
        UserDAO userDAO = (UserDAO) ctx.getBean("userDAO");
        User user = new User();
        user.setName("jelly");
        user.setPassword("999");
        userDAO.save(user);
    }
}
```

