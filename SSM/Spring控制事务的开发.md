#  Spring控制事务的开发

```markdowm
Spring是通过AOP的方式进行事务开发的
```

> 原始对象

```java 
public class xxxUserServiceImpl{
    private xxxDAO xxxDAO
    set get
     1.原始对象  --》 原始方法 --》 核心功能(业务处理+DAO调用)
     2.DAO作为Service的成员变量，依赖注入的方式进行赋值   
}
```

> 额外功能

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

>搭建开发环境

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