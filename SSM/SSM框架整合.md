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

