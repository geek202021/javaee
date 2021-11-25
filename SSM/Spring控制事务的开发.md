#  Spring控制事务的开发

- start from P116

<img src="ssm-imgs\transaction.png" align="left" />

- Spring是通过AOP的方式进行事务开发的

> 1.原始对象

```java 
public class xxxUserServiceImpl{
    private xxxDAO xxxDAO
    set get
     1.原始对象  --》 原始方法 --》 核心功能(业务处理+DAO调用)
     2.DAO作为Service的成员变量，依赖注入的方式进行赋值   
}
```

> 2.额外功能

```java 
1.MethodInterceptor
  public Object invoke(MethodInvocation invocatino){
    try{
        Connection.setAutoCommit(false);
        Object ret = invocation.proceed();
        Connection.commit();
    } catch(Exception e){
        Connection.rollback();
    }
    return ret;
}  
2.@Aspect
  @Around  
```

- spring帮我们封装好的类：`org.springframework.jdbc.datasource.DataSourceTransactionManager` 控制事务的额外功能。使用时，还要注意必须：`注入DataSource`

>3.切入点

1. `@Transactional`：事务的额外功能加入给哪些业务方法。
2. 加在类上：类中的所有方法都会加入事务
3. 加在方法上：这个方法会加入事务

>4.组装切面：切入点 + 额外功能

`<tx:annotation-driven transaction-manager=""/>`

- 其中切入点 `tx` 标签会自动 扫描`@Transactional`注解，来获取切入点相关的信息。

##  Spring控制事务的编码

<img src="ssm-imgs\tx-annotation-driven.png" align="left" />

- 搭建开发环境（jar）

```xml
<dependency>
<groupId>org.springframework</groupId>
	<artifactId>spring-tx</artifactId>
	<version>5.2.6.RELEASE</version>
</dependency>
```

```java
public class UserServiceImpl implements UserService{
    //UserServiceImpl是原始对象，负责核心功能和DAO的调用
    //把UserDAO作为其成员变量，注入
    private UserDAO userDAO;
    public UserDAO getUserDAO() {
        return userDAO;
    }
    public void setUserDAO(UserDAO userDAO) {
        this.userDAO = userDAO;
    }
    
    @Override
    public void register(User user) {
        userDAO.save(user);
    }
}
```

```xml
<!--Spring控制事务-->
<bean id="userService" class="com.jun.transaction.UserServiceImpl">
    <property name="userDAO" ref="userDAO"/>
</bean>
<!--DataSourceTransactionManager-->
<bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>

<tx:annotation-driven transaction-manager="dataSourceTransactionManager"/>
```

- 测试

```java
@Test
public void test1(){
    ApplicationContext ctx =  new ClassPathXmlApplicationContext("/applicationContext.xml");
    UserService userService = (UserService) ctx.getBean("userService");
    User user = new User();
    user.setName("hrj");
    user.setPassword("321");
    userService.register(user);
}
```

- 验证register方法已经加上了事务

1. 在register中方法中抛出异常，此时如果register方法加上了事务的话，register方法是一个整体，复合事务的原子性。抛出的异常会导致register方法中的save回滚。

```java 
@Transactional
public class UserServiceImpl implements UserService{
    private UserDAO userDAO;
    
    public UserDAO getUserDAO() {
        return userDAO;
    }
    public void setUserDAO(UserDAO userDAO) {
        this.userDAO = userDAO;
    }
    
    @Override
    public void register(User user) {
        userDAO.save(user);
        throw new RuntimeException("测试");
    }
}
```

###  细节

`proxy-target-class="true"` 进行动态代理底层实现的切换

- 默认false   JDK
- true   Cglib

```java
<tx:annotation-driven transaction-manager="dataSourceTransactionManager" proxy-target-class="true"/>    
```

#  Spring中的事务属性（Transaction Attribute）

- 事务属性：描述事务特征的一系列值：
  - 隔离属性（isloation）、传播属性（propagation）、只读属性（readOnly）、超时属性(timeout)、异常属性(rollbackFor/noRollbackFor)

- `@Transactional(isloation=,propagation=)`

##  事务属性常见配置总结

<img src="ssm-imgs\transaction-config.png" align="left" />

#  基于标签的事务配置方式（事务开发的第二种形式）

<img src="ssm-imgs\tx-advice-aop-config.png" align="left" />

##  基于标签的事务配置在实战中的应用方式

<img src="ssm-imgs\tx-advice-aop-config2.png" align="left" />









