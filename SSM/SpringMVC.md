#  SpringMVC

# 1.第一个环境搭建

- 引入依赖：Spring、Spring-webmvc、Servlet-api规范

##  编写SpringMVC配置文件

```xml
<!--1.开启注解扫描：包扫描只扫描控制器所在包-->
<context:component-scan base-package="com.jun.controller"/>
<!--2.配置处理器映射器-->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
<!--3.配置处理器适配器-->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
<!--4.配置视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
<!--也就是转发到什么样的页面，比如页面的逻辑名字（Controller返回的字符串）为index则组装出来的Url为 /index.jsp-->
    <!--注入前缀和后缀:固定写死，值可以根据项目页面目录动态变化-->
    <property name="prefix" value="/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

##  mvc:annotation-driven

但是SpringMVC加入< mvc:annotation-driven/>配置方案(一定得是mvc结尾的)，可以让初学都快速应用默认配置方案，包括了:

- 会自动注册RequestMappingHandlerMapping与RequestMappingHandlerAdapter
- 提供了数据绑定支持
- @NumberFormatannotation支持，@DateTimeFormat支持,@Valid支持读写XML的支持（JAXB）和读写JSON的支持（默认Jackson）等功能，所以我们可以写成：

```xml
<context:component-scan base-package="#方式自定"/>
<mvc:annotation-driven/>
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
	<property name="prefix" value="/"/>
	<property name="suffix" value=".jsp"/>
</bean>
```

##  配置springmvc的核心Servlet

- web.xml

```xml
<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <!-- 配置springmvc的核心Servlet-->
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--指定springmvc配置文件位置-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
      <!--固定写死用来加载SpringMVC配置文件位置-->
    </init-param>
  </servlet>

  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
    <!--指定相对于Servlet的URL的路径，拦截所有请求，交给springMVC处理-->
  </servlet-mapping>
</web-app>
```

##  / 和 /* 细节：

```markdown
在指定指定相对于Servlet的URL的路径时，SpringMVC使用的是“/”,而不是“/*”
/ 模式下，Servlet不会拦截 .jsp（仅限于此，.html仍会被拦截）格式的请求；
而 /* 模式才是真正意义上的拦截所有形式的请求。
查看Tomcat的配置文件
发现web容器拦截了.jsp（Tomcat默认的servlet）
所以使用/*会发现转发不了页面，出现404
```

```java
@Controller
@RequestMapping(value = "/hello")
public class HelloController {
    /**
     * 修饰范围：用在类上 和 方法上
     * 作用：用来指定类以及类中方法的请求路径
     *  请求 URL 的第一级访问目录。此处不写的话，就相当于应用的根目录。写的话需要以/开头。
     *  它出现的目的是为了使我们的 URL 可以按照模块化管理:
     *  例如，账户模块：/account/add
     */
    @RequestMapping(value = "/hello")
    public String hello() {
        //1.收集数据
        //2.调用业务方法
        System.out.println("hello spring mvc");
        //3.处理响应
        return "index"; //页面逻辑名（解析结果:前缀+返回值+后缀）
    }
}
```

#  2.post解决springmvc中文乱码

web.xml

```xml
<!--配置post请求方式中文乱码的Filter-->
<filter>
  <filter-name>charset</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>charset</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

# 3.forward跳转页面数据传递

```java
@Controller
@RequestMapping(value = "req")
public class RequestController {
    /**
     * 使用forward跳转页面数据传递
     *  1.传递零散类型数据、2.传递对象类型数据、3.传递集合类型数据
     *  使用request对象传递数据 ===》 Model对象：底层封装就是request对象
     *  import org.springframework.ui.Model;
     */
    @RequestMapping(value = "test")
    public String test(Model model,HttpServletRequest request, HttpServletResponse response) {
        //1.收集参数
        //2.调用业务方法
        String name = "张三";
        User user = new User(2,"李华",23,new Date());
        User user1 = new User(3,"zhangsan",22,new Date());
        User user2 = new User(4,"lisi",24,new Date());
        List<User> users = Arrays.asList(user1, user2);
//        request.setAttribute("username", name);
//        request.setAttribute("user",user);
        request.setAttribute("users",users);
        model.addAttribute("username", name);
        model.addAttribute("user", user);
        //3.流程跳转
        return "req";
    }
    /**
     * 使用redirect跳转传递数据
     * 传递数据的两种方式：1.地址栏?拼接数据；2.session对象
     *
     */
    @RequestMapping(value = "test1")
    public String test1(HttpServletRequest request) throws UnsupportedEncodingException {
        String name = "李四";
        User user = new User(1, "杰森", 34, new Date());
        request.getSession().setAttribute("user", user);
        return "redirect:/req.jsp?name=" + URLEncoder.encode(name,"UTF-8");
    }
}
```

req.jsp

```jsp
<body>
    <h1>用来测试request作用域数据传递</h1>
    <h3>获取request作用域数据：${requestScope.username}</h3>
    <h3>获取request作用域数据：${username}</h3>
    <hr color="red"/>
    <h3>id:${requestScope.user.id}</h3>
    <h3>name:${requestScope.user.name}</h3>
    <h3>age:${requestScope.user.age}</h3>
<%--    <h3>birthday:${requestScope.user.birthday}</h3>--%>
    <h3>birthday: <fmt:formatDate value="${requestScope.user.birthday}" pattern="yyy-MM-dd"/></h3>
    <hr color="red"/>
    <c:forEach items="${requestScope.users}" var="user">
        id:${user.id} === ${user.name} === ${user.age} === ${user.birthday} <br/>
    </c:forEach>

    <hr color="green"/>
    <h3>获取地址栏数据：${param.name}</h3>
    <h3>id：${sessionScope.user.id}</h3>
    <h3>name：${sessionScope.user.name}</h3>
    <h3>age：${sessionScope.user.age}</h3>
    <h3>birthday：${sessionScope.user.birthday}</h3>
</body>
```

#  spring 与 mvc整合

2021年11月5日23:37:32

##  web.xml

```xml
<!--1.配置工厂监听器-->
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<!--2.工厂配置文件-->
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:spring.xml</param-value>
</context-param>
<!--3.配置springMVC核心Servlet-->
<servlet>
  <servlet-name>springMVC</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc.xml</param-value>
  </init-param>
</servlet>
<servlet-mapping>
  <servlet-name>springMVC</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
<!--4.配置post请求参数的中文乱码问题-->
<filter>
  <filter-name>charset</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>charset</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

##  springmvc.xml

```xml
<!--1.开启注解扫描-->
<context:component-scan base-package="com.jun.controller"/>
<!--2.配置处理器映射器和处理器适配器-->
<mvc:annotation-driven/>
<!--3.配置视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/"/>
    <property name="suffix" value=".jsp"/>
</bean>
<!--配置静态资源拦截-->
<mvc:default-servlet-handler/>
```

##  UserController.java

```java
@Controller
@RequestMapping(value = "user")
public class UserController {
    @Autowired
    private UserService userService;
    @RequestMapping(value = "findAll")
    public String findAll(HttpServletRequest request) {
        List<User> users = userService.findAll();
        request.setAttribute("users",users);
        return "findAll";
    }
    //添加用户
    @RequestMapping("save")
    public String save(User user) {
        //调用业务方法
        try {
            userService.save(user);
            return "redirect:/user/findAll";
        } catch (Exception e) {
            e.printStackTrace();
            return "redirect:/add.jsp";
        }
    }
}
```

##  findAll.jsp & add.jsp

```jsp
<body>
    <h1>展示用户列表</h1>
    <c:forEach items="${requestScope.users}" var="user">
        ${user.id} === ${user.name} === ${user.age} === ${user.bir} <br/>
    </c:forEach>
    <a href="${pageContext.request.contextPath}/add.jsp">添加用户信息</a>
</body>

<body>
    <h1>添加用户信息</h1>
    <form action="${pageContext.request.contextPath}/user/save" method="post">
        用户名：<input type="text" name="name"/> <br/>
        年龄：<input type="text" name="age"/> <br/>
        生日：<input type="text" name="bir"/> <br/>
        <input type="submit" value="保存用户信息">
    </form>
</body>
```

2021年11月6日09:34:09

#  文件上传与下载

在springmvc的基础上添加想换依赖

```xml
<!--commons-fileUpload文件上传-->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
```

在springmvc.xml中添加

```xml
<!--配置静态资源拦截-->
<mvc:default-servlet-handler/>
<!--配置文件上传解析器
	id：必须指定为 multipartResolver-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	<!--注入文件上传下载大小限制，单位字节 2M = 20971520字节 ；默认没有限制-->
	<property name="maxUploadSize" value="20971520"/>
</bean>
```

upload.jsp

```jsp
<body>
<h1>文件上传</h1>
<form action="${pageContext.request.contextPath}/file/upload" method="post" enctype="multipart/form-data">
	<input type="file" name="img"/>
	<input type="submit" value="上传文件"/>
</form>
<h1>文件下载</h1>
<a href="${pageContext.request.contextPath}/file/download?fileName=down.txt">下载down.txt</a>    
</body>
```

获取文件名后缀

 >问题：xx.getOriginalFilename()获取的是原文件名，直接使用文件的原始名作为上传文件的最终名称，会导致重名文件覆盖的问题。
 >
 >解决：UUID+文件名后缀

```java
String fileName = "xxx.txt";
//分隔符
String[] split = fileName.split("\\.");
System.out.println(split[split.length-1]); //txt
//2.字符串subString
int i = fileName.lastIndexOf(".");
String substring = fileName.substring(i);
System.out.println(substring); //.txt
//3.工具类型 common-fileupload工具类
String extension = FileNameUtils.getExtension(fileName);
System.out.println(extension);
```

##  LodalDate

```java
sout(new Date());
String format = new SimpleDateFormat("yyyy-MM-dd").format(new Date());
sout(format); //2021-11-06
sout(LocalDate.now()); //2021-11-06
```

##  FileController.java

```java
@Controller
@RequestMapping("file")
public class FileController {
    @RequestMapping("upload")
    public String upload(MultipartFile img, HttpServletRequest request) throws IOException {
        System.out.println("文件名：" + img.getOriginalFilename());
        System.out.println("文件类型：" + img.getContentType());
        System.out.println("文件大小：" + img.getSize());
        //处理文件上传:
        //1.根据upload相对路径获取部署到服务器之后的绝对路径
        String realPath = request.getSession().getServletContext().getRealPath("/upload");
        //2.修改文件的原始名称
        String extension = FilenameUtils.getExtension(img.getOriginalFilename());
        String newFileName = UUID.randomUUID().toString().replace("-","") + "." + extension;
        //3.生成当天日期目录
        LocalDate now = LocalDate.now();
        File dateDir = new File(realPath, now.toString());
        if(!dateDir.exists()) dateDir.mkdirs();
        //4.将文件上传到upload对应日期的目录中
        img.transferTo(new File(realPath, newFileName));
        return "index";
    }
}
```

文件下载步骤：

1. 定位系统中哪些文件需要用户下载
2. 将需要下载的文件放入指定下载目录中
3. 开发夜歌页面提供一个文件下载链接
4. 开发下载controller

```java
//用来处理文件下载，请求对应响应输出流
@RequestMapping("download")
public void download(String fileName, HttpServletRequest request, HttpServletResponse response) throws IOException {
    System.out.println("下载文件的名称：" + fileName);
    //1.根据下载相对目录获取下载目录在服务器部署之后的绝对目录
    String realPath = request.getSession().getServletContext().getRealPath("/down");
    //2.通过文件输入流读取文件
    FileInputStream fis = new FileInputStream(new File(realPath, fileName));
    //3.获取响应输出流
    response.setContentType("text/plain;charset=UTF-8");
    //4.附件下载:attachment，inline：在线打开
    response.setHeader("content-disposition", "attachment;fileName=" + URLEncoder.encode(fileName,"UTF-8"));
    ServletOutputStream os = response.getOutputStream();
    //5.处理下载流复制
    int len;
    byte[] b = new byte[1024];
    while (true) {
        len = fis.read(b);
        if(len == -1) break;
        os.write(b,0,len);
    }
    //释放资源
    fis.close();
}
```

简便写法：操作io流用IOUtils，操作file用FileUtils

```java
//5.处理下载流复制
//commons.io包下的工具类
IOUtils.copy(fis, os);
IOUtils.closeQuietly(fis);
IOUtils.closeQuietly(os);
```

#  springMVC与ajax的集成

>> 在springmvc控制器中如何将对应数据转为JSON？
>
>a).使用fastjson转换为json格式字符串
>
>b).使用springmvc提供注解@ResponseBody
>
>- 作用范围：用在方法或者方法的返回值上
>- 作用：用来将方法的返回值自动转为json格式字符串并响应到前台

##  fastjson

引入依赖

```xml
<!--fastJSON-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.62</version>
</dependency>
```

json.jsp

```jsp
<script src="${pageContext.request.contextPath}/js/jquery-3.5.1.min.js"></script>
    <script>
        $(function () {
            $("#btn").click(function () {
                $.get("${pageContext.request.contextPath}/json/showAll", function (res) {
                    console.log(res);
                },"JSON");
            });
        });
    </script>
</head>
<body>
    <button id="btn">显示一群人信息</button>
</body>
```

JsonController.java

```java
@Controller
@RequestMapping("json")
public class JsonController {
    //1.使用阿里的fastJSON转换json(原始方法响应json格式字符串)
    @RequestMapping("findAll")
    public void findAll(HttpServletResponse res) throws IOException {
        List<User> users = new ArrayList<>();
        users.add(new User(5, "李华", 23, new Date()));
        users.add(new User(7, "张三", 13, new Date()));
        users.add(new User(8, "李四", 55, new Date()));
        //fastjson
        String s = JSONObject.toJSONStringWithDateFormat(users, "yyyy-MM-dd");
        res.setContentType("application/json;charset=UTF-8");
        res.getWriter().println(s);
    }
```

##  @ResponseBody

2.使用springmvc提供注解@ResponseBody

```xml
<!--@ResponseBody注解在转换json时使用的jackson依赖-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
```

```java
//2.使用springmvc提供注解@ResponseBody
    //作用：用来将控制器方法的返回值转为json，并响应给前台
    @RequestMapping("showAll")
//    @ResponseBody
    public @ResponseBody List<User> showAll() {
        List<User> users = new ArrayList<>();
        users.add(new User(5, "李华1", 23, new Date()));
        users.add(new User(7, "张三2", 13, new Date()));
        users.add(new User(8, "李四3", 55, new Date()));
        return users;
    }
}
```

##  日期格式问题

- {id: 5, name: '李华1', age: 23, bir: 1636176919306}

```java
public class User {
    private Integer id;
    private String name;
    private Integer age;
    @JsonFormat(pattern = "yyyy-MM-dd")
    private Date bir;
```

```jsp
$(function () {
    $("#btn").click(function () {
        $.get("${pageContext.request.contextPath}/json/showAll", function (res) {
            $.each(res, function (i, user) {
                var ul = $("<ul/>");
                var idLi = $("<li/>").text(user.id);
                var nameLi = $("<li/>").text(user.name);
                var ageLi = $("<li/>").text(user.age);
                var birLi = $("<li/>").text(user.bir);
                ul.append(idLi).append(nameLi).append(ageLi).append(birLi);
                $("#bd").append(ul);
            })
        },"JSON");
    });
});
```

#  拦截器

>- 类似于javaweb中的Filter,用来对请求进行拦截,可以将多个Controller中执行的共同代码放入拦截器中执行,减少Controller类中代码的冗余。
>
>- 底层依然是aop,额外功能是Controller中的方法。

**特点**：

- 拦截器器只能拦截Controller的请求,不能拦截jsp
- 拦截器可中断用户的请求轨迹

- 请求先经过拦截器,之后还会经过拦截器

springmvc.xml配置

```xml
<!--注册拦截器-->
<bean id="myInterceptor" class="com.jun.interceptors.MyInterceptor"/>
<!--配置拦截器-->
<mvc:interceptors>
    <!--配置一个拦截器-->
    <mvc:interceptor>
        <!--mvc:mapping 代表拦截哪个请求路径-->
        <mvc:mapping path="/json/test"/>
        <!--mvc:exclude-mapping 排除拦截哪个请求-->
        <mvc:exclude-mapping path="/json/showAll"/>
        <!--使用哪个拦截器-->
        <ref bean="myInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

1. 请求经过拦截器会优先进入拦截器中的 `preHandler` 方法执行preHandler方法中的内容
2. 如果preHandler返回值为true代表放行请求，如果返回值为false，中断请求
3. 如果preHandler返回值为true，会执行当前请求对应的控制器中方法
4. 当控制器方法执行结束之后，会返回拦截器中执行拦截器中`postHandler `方法
5. postHandler执行完成之后响应请求，在响应请求完成之后会执行`afterCompletion`方法

>如果有业务 是在控制器之前需要完成，那么就放在preHandle中去完成；如过有业务需求需要在控制器执行完成之后在做，就放在postHandle中完成；如果有业务需求无论控制器出错与否都执行，就放在faterCompletion中完成。

```java
public class MyInterceptor implements HandlerInterceptor {
    /**
     * preHandle方法是进行处理器拦截用的，顾名思义，该方法将在Controller处理之前进行调用，
     * SpringMVC中的Interceptor拦截器是链式的，可以同时存在多个Interceptor，
     * 然后SpringMVC会根据声明的前后顺序一个接一个的执行，
     * 而且所有的Interceptor中的preHandle方法都会在Controller方法调用之前调用。
     * SpringMVC的这种Interceptor链式结构也是可以进行中断的，
     * 这种中断方式是令preHandle的返回值为false，当preHandle的返回值为false的时候整个请求就结束了。
     * 【参数：Object o】代表当前请求的控制器对应的方法对象
     *  HttpServletRequest request, HttpServletResponse response为当前的请求和响应
     * 1. 请求经过拦截器会优先进入拦截器中的 `preHandler` 方法执行preHandler方法中的内容
     * 2. 如果preHandler返回值为true代表放行请求，如果返回值为false，中断请求
     * 3. 如果preHandler返回值为true，会执行当前请求对应的控制器中方法
     * 4. 当控制器方法执行结束之后，会返回拦截器中执行拦截器中`postHandler `方法
     * 5. postHandler执行完成之后响应请求，在响应请求完成之后会执行`afterCompletion`方法
     */
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object o) throws Exception {
        System.out.println(((HandlerMethod)o).getMethod().getName()); //test
        System.out.println("========1========");
        //强制用户登录
//        Object user = req.getSession().getAttribute("user");
//        if (user == null) {
//            //重定向回登录页面
//            res.sendRedirect(req.getContextPath() + "/login.jsp");
//            return false;
//        }
        return true;
    }
    /**
     * 这个方法只会在当前这个Interceptor的preHandle方法返回值为true的时候才会执行。
     * postHandle是进行处理器拦截用的，它的执行时间是在处理器进行处理之后， 也就是在Controller的方法调用之后执行，
     * 但是它会在DispatcherServlet进行视图的渲染之前执行，也就是说在这个方法中你可以对ModelAndView进行操作。
     * 这个方法的链式结构跟正常访问的方向是相反的，也就是说先声明的Interceptor拦截器该方法反而会后调用，
     * 【ModelAndView modelAndView】
     *  为当前返回的视图和数据模型：视图也就是Controller中返回值的字符串之后被底层组装
     *  数据模型也就是比如：request.setAttribute设置的k-v
     */
    @Override
    public void postHandle(HttpServletRequest req, HttpServletResponse res, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println(modelAndView);
        System.out.println("========3========");
    }
    /**
     * 该方法也是需要当前对应的Interceptor的preHandle方法的返回值为true时才会执行。
     * 该方法将在整个请求完成之后，也就是DispatcherServlet渲染了视图执行，这个方法的主要作用是用于清理资源的
     * 也用来执行异常对象
     */
    @Override
    //无论正确还是失败都会执行
    public void afterCompletion(HttpServletRequest req, HttpServletResponse res, Object o, Exception ex) throws Exception {
        if (ex != null) {
            System.out.println(ex.getMessage());
        }
        System.out.println("========4========");
    }
}
```

