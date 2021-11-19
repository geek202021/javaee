#  Spring动态代理开发

#  代理设计模式

- 代理类 = 目标类(原始类) + 额外功能 + 原始类(目标类)实现相同的接口

>:pencil:搭建开发环境

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aop</artifactId>
  <version>5.1.14.RELEASE</version>
</dependency>
<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjrt</artifactId>
  <version>1.8.9</version>
</dependency>
<dependency>
  <groupId>org.aspectj</groupId>
  <artifactId>aspectjweaver</artifactId>
  <version>1.8.13</version>
</dependency>
```

2021年10月27日22:25:22

>>:pencil:开发步骤
>
>1.创建原始对象（目标对象）

```java
public interface UserService {
    void register(User user);
    boolean login(String name, String password);
public class UserServiceImpl implements UserService{
    @Override
    public void register(User user) {
        System.out.println("UserServiceImpl.register 业务运算 + DAO");
    }
    @Override
    public boolean login(String name, String password) {
        System.out.println("UserServiceImpl.login 业务运算 + DAO");
        return true;    
```

>2.添加额外功能：实现 `MethodBeforeAdvice` 接口（额外功能运行在原始方法执行之前）
>
>-  Method: 额外功能所增加给的那个原始方法
>- Object[]:  额外功能所增加给的那个原始方法的参数
>- Object: 额外功能所增加给的那个原始对象：UserServiceImpl

```java
public class Before implements MethodBeforeAdvice {
	//作用: 把需要运行在原始方法执行之前运行的额外功能, 书写在 before 方法中
    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println("---method before advice log---");
    }
}
```

- 在配置文件中引入接口的实现类<bean id="before" class="com.jun.dynamic.Before"/>

>3.定义 **切入点**：额外功能的加入

```xml
<aop:config>
   <aop:pointcut id="pc" expression="execution(* * (..))"/>
</aop:config>
```

>4.组装：2.3整合

```xml
<bean id="userService"  class="com.jun.proxy.UserServiceImpl"/>
<!-- 额外功能 -->
<bean id="before" class="com.jun.dynamic.Before"/>

<aop:config>
<!--切入点:额外功能的加入-->
<!--目的：由程序员根据自己的需要，决定额外功能加入给哪个原始方法(register、login)-->
<!-- 简单的测试：所有方法都做为切入点，都加入额外的功能-->  
    <aop:pointcut id="pc" expression="execution(* *(..))"/>
<!--组装：表达的含义: 所有的方法 都加入before的额外功能-->
    <aop:advisor advice-ref="before" pointcut-ref="pc"/>
</aop:config>
   

```

#  MethodInterceptor

- 参数：`MethodInvocation`：额外功能所增加给的那个原始方法 (login, register)
- 返回值：`Object`：原始方法的返回值 (没有就返回 null)
- `invocation.proceed()`：原始方法运行

```java 
public class Around implements MethodInterceptor{
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable{
        System.out.println("--额外功能运行在原始方法执行之前--");
        Object ret = methodInvocation.proceed();
        return ret;
    }
}
```

#  切入点函数

##  @Annotation

- 为具有特殊注解的 **方法** 加入额外功能。

>>元注解的种类
>
>1. Target：用于描述注解的使用范围
>   1. 类或接口：ElementType.TYPE；方法：ElementType.METHOD；构造方法：ElementType.CONSTRUCTOR；
>   2. 字段：ElementType.FIELD；方法参数：ElementType.PARAMETER。
>2. Retention：表示需要在什么级别保存该注释信息，用于描述注解的生命周期。
>   1. SOURCE：表示该注解只被保留在java原文件中
>   2. CLASS：表示该注解被保存在class文件中
>   3. RUNTIME：表示该注解被保存在class文件中，并且可以被反射机制读取到。

- 例如我们自定义了一个注解：`Log`，用在register方法上。

```java 
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {}

@Log
@Override
public void register(User user){
    System.out.println("UserServiceImpl.register 业务运算 + DAO");
}
```

- 然后我们要为使用了 `Log` 注解的方法加入额外功能。

```xml
<aop:pointcut id="pc" expression="@annotation(com.jun.Log)"/>
```

#  AOP底层实现原理

##  JDK的动态代理

```java
public class TestJDKProxy {
    public static void main(String[] args) {
        //1.创建原始对象
        UserService userService = new UserServiceImpl();
        /**
         * 2.JDK创建动态代理
         *      类加载器、原始对象所创建的接口、额外的功能
         *      借用一个类加载器
         */
        InvocationHandler handler = new InvocationHandler(){
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("--额外功能在原始方法之前运行--");
                //先让原始方法运行:动态代理中的为invocation.proceed();
                Object ret = method.invoke(userService, args);
                return ret;
            }
        };
        UserService userServiceProxy = (UserService) Proxy.newProxyInstance(TestJDKProxy.class.getClassLoader(), userService.getClass().getInterfaces(), handler);
         /*UserService userServiceProxy = (UserService) Proxy.newProxyInstance(userService.getClass().getClassLoader(),userService.getClass().getInterfaces(), (proxy, method, argss) -> {
            System.out.println("----log-----");
            return method.invoke(userService, argss);
        });*/
        userServiceProxy.login("jun", "1234");
        userServiceProxy.register(new User());
    }
}
```

##   Cglib的动态代理

```java
public class TestCglib {
    public static void main(String[] args) {
        //1.创建原始对象
        UserService userService = new UserService();
		//2.通过cglib方式创建动态代理对象
		//对比 jdk 动态代理 ---> Proxy.newProxyInstance(classLoader, interfaces, 			invocationHandler);
        //Enhancer：通过继承父类创建的代理类
        Enhancer enhancer = new Enhancer();
        enhancer.setClassLoader(TestCglib.class.getClassLoader());
        enhancer.setSuperclass(userService.getClass());  //== interfaces
        MethodInterceptor interceptor = new MethodInterceptor() { 
            //MethodInterceptor(cglib包中的)
            //等同于JDK动态代理中的 InvocationHandler中的invoke方法
            @Override
            public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                System.out.println("--cglib动态代理--");
                Object ret = method.invoke(userService, args);
                return ret;
            }
        };
        //设置额外功能
        enhancer.setCallback(interceptor);
        //创建代理对象
        UserService userServiceProxy = (UserService) enhancer.create();
        userServiceProxy.login("jun", "1234");
        userServiceProxy.register(new User());
    }
}
```

#  基于注解的AOP编程

>开发步骤：1.原始对象、2.额外功能、3.切入点、4.组装切面
>
>- 2.3.4步骤主要是为了创建切面。切面也可以想象成一个类=》切面类
>
>>体现额外功能
>
>- @Around 等同于原来写的public class MyAround implements MethodInterceptor{}
>- 自定义的pubic Object around()方法 等同于 原来的 public Object invoke(MethodInvocation invocation) 方法
>- 参数：ProceedingJoinPoint joinPoint代表的是额外功能所增加给的那个原始方法
>
>>切入点

```java
@Aspect //表示切面类
public class MyAspect {
//    @Pointcut("execution(* login(..))")
    @Pointcut("execution(* *..UserServiceImpl.*(..))")//指定 特定类作为切入点(额外功能加入的位置)，这个类中的所有方法，都会加上对应的额外功能。
    public void myPointcut(){}
    
//    @Around("execution(* login(..))")    
    @Around(value="myPointcut()") //切入点
    //返回值Object代表原始方法的返回值，MethodInvocation invocation代表额外功能所增加给的那个原始方法
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("=======aspectJ log=======");
        Object ret = joinPoint.proceed(); //它的返回值就是原始方法的返回值
        return ret;
    }
    @Around(value="myPointcut()") //切入点复用
    public Object around1(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("=======aspectJ tx=======");
        Object ret = joinPoint.proceed();
        return ret;
    }
}
```

```xml
<bean id="userService" class="com.jun.aspect.UserServiceImpl"/>
<bean id="myAspect" class="com.jun.aspect.MyAspect"/>
<!--告知 Spring 基于注解进行 AOP 编程-->
<aop:aspectj-autoproxy/>
```

>默认情况 AOP 编程 底层应用 JDK动态代理创建方式。基于注解的 AOP 开发 中切换为 Cglib：
>
>- <aop:aspectj-autoproxy proxy-target-class="true"/>

