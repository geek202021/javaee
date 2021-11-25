#  SSM-CRUD

2021年11月7日20:10:29 (尚硅谷视频中的整合项目)

>功能点：

1. 分页查询
2. 数据校验：JQuery前端校验+JSR-303后端校验
3. Ajax请求
4. REST风格的URI：GET查询、POST新增、DELETE删除、PUT修改

>技术点：

- 基础框架-SSM（Spring+SpringMVC+Mybatis）
- 数据库-MySQL
- 前端框架-Bootstrap
- 依赖管理-Maven
- 分页查询-PageHelper
- 逆向工程-Mybatis Generator

#  web.xml

```xml
<!--1.启动Spring的容器-->
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:spring.xml</param-value>
</context-param>
<!--2.配置springMVC的前端控制器,拦截所有请求-->
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
<!--3.字符编码过滤器：一定要放在所有过滤器之前-->
<filter>
  <filter-name>charset</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>utf-8</param-value>
  </init-param>
  <init-param>
    <param-name>forceRequestEncoding</param-name>
    <param-value>true</param-value>
  </init-param>
  <init-param>
    <param-name>forceResponseEncoding</param-name>
    <param-value>true</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>charset</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
<!--4.使用Rest风格的URI，将页面普通的post请求转为指定的delete或者put请求-->
<filter>
  <filter-name>hiddenHttpMethodFilter</filter-name>
  <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>hiddenHttpMethodFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

#  dispatcherServlet.xml

```xml
<!--SpringMVC的配置文件，包含网站跳转逻辑的控制，配置-->
<context:component-scan base-package="com.jun" use-default-filters="false">
    <!--只扫描@Controller控制器-->
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
<!--配置适配器,映射动态请求-->
<mvc:annotation-driven/>
<!--配置视图解析器，方便页面返回-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/views/"/>
    <property name="suffix" value=".jsp"/>
</bean>
<!--将springMVC不能处理的请求交给Tomcat-->
<mvc:default-servlet-handler/>
```

#  applicationContext.xml

```xml
 <context:component-scan base-package="com.jun">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <!--Spring的配置文件，主要配置和业务逻辑有关的-->
    <!--数据源，事务控制-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--指定Mybatis全局配置文件的位置-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
<!--        <property name="typeAliasesPackage" value="com.jun.bean"/>-->
        <property name="dataSource" ref="dataSource"/>
        <!--指定mapper文件Mybatis的位置-->
        <property name="mapperLocations" value="classpath:com/jun/mapper/*.xml"/>
    </bean>
    <!--配置扫描器，将Mybatis接口的实现类加入到IOC容器中-->
    <bean id="scanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
<!--        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactoryBean"/>-->
        <!--扫描所有dao接口的实现，加入到ioc容器中-->
        <property name="basePackage" value="com.jun.dao"/>
    </bean>
    <!--事务控制的配置-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--控制住数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--使用xml配置形式的事务（推荐），也可以使用基于注解形式的事务控制-->
    <!--配置事务增强，事务如何切入-->
    <tx:advice id="txAdvice">
        <tx:attributes>
            <!--所有方法都是事务方法-->
            <tx:method name="*"/>
            <!--以get开始的所有方法-->
            <tx:method name="get*" read-only="true"/>
        </tx:attributes>
    </tx:advice>
    <aop:config>
        <!--切入点表达式-->
        <aop:pointcut id="txPoint" expression="execution(* com.jun.service..*(..))"/>
        <!--配置事务增强-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPoint"/>
    </aop:config>
```

#  mabatis-config.xml

```xml
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
    <typeAliases>
        <package name="com.jun.bean"/>
    </typeAliases>
</configuration>
```

#  实体表

```sql
CREATE DATABASE IF NOT EXISTS ssm_crud;
USE ssm_crud;

CREATE TABLE `t_dep`  (
  `dep_id` INT(11) PRIMARY KEY AUTO_INCREMENT,
  `dep_name` VARCHAR(255) NOT NULL
) ENGINE = INNODB CHARACTER SET = utf8;
CREATE TABLE `t_emp`  (
  `emp_id` int(11) primary key auto_increment,
  `emp_name` varchar(255) NOT NULL,
  `gender` char(1) NOT NULL,
  `email` varchar(255) DEFAULT NULL,
  `d_id` int(11) NOT NULL,
  FOREIGN KEY (`d_id`) REFERENCES `t_dep` (`dep_id`) 
) ENGINE = InnoDB CHARACTER SET = utf8;
```

#  逆向工程

```xml
<dependency>
  <groupId>org.mybatis.generator</groupId>
  <artifactId>mybatis-generator-core</artifactId>
  <version>1.3.7</version>
</dependency>
```

mbg.xml

```xml
<generatorConfiguration>
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <!-- 自定义的注释规则，继承 DefaultCommentGenerator 重写一些方法 -->
        <commentGenerator>
            <!-- 是否去除自动生成日期的注释-->
            <property name="suppressDate" value="true"/>
            <!-- 是否去除所有自动生成的注释-->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--配置数据库连接信息-->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/ssm_crud?useSSL=false"
                        userId="root"
                        password="hrj">
        </jdbcConnection>
        <!--生成entity类存放位置-->
        <javaModelGenerator targetPackage="com.jun.bean"
                            targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>
        <!--生成sql映射文件存放位置-->
        <sqlMapGenerator targetPackage="com.jun.mapper"
                         targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!--生成Dao接口类存放位置-->
        <javaClientGenerator type="XMLMAPPER"
                             targetPackage="com.jun.dao"
                             targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <!--table指定每个表的生成策略:domainObjectName=生成实体的类名-->
        <table tableName="t_emp" domainObjectName="Employee"></table>
        <table tableName="t_dep" domainObjectName="Department"></table>
    </context>
</generatorConfiguration>
```

MBGenerator.java

```java
public class MBGenerator {
    @Test
    public void testGenerator() throws Exception {
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        File configFile = new File("mbg.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }
}
```

#  EmployeeMapper.xml

```xml
<resultMap id="WithDeptResultMap" type="com.jun.bean.Employee">
    <id column="emp_id" jdbcType="INTEGER" property="empId" />
    <result column="emp_name" jdbcType="VARCHAR" property="empName" />
    <result column="gender" jdbcType="CHAR" property="gender" />
    <result column="email" jdbcType="VARCHAR" property="email" />
    <result column="d_id" jdbcType="INTEGER" property="dId" />
    <association property="department" javaType="com.jun.bean.Department">
      <id column="dep_id" property="depId"/>
      <result column="dep_name" property="depName"/>
    </association>
</resultMap>
<sql id="WithDept_Column_List">
    e.emp_id, e.emp_name, e.gender, e.email, e.d_id,d.dep_id,d.dep_name
</sql>
<!--查询员工的同时，带部门信息-->
<!--  List<Employee> selectByExampleWithDept(EmployeeExample example);-->
  <select id="selectByExampleWithDept" resultMap="WithDeptResultMap">
    select
    <if test="distinct">
      distinct
    </if>
    <include refid="WithDept_Column_List" />
    FROM t_emp e
    LEFT JOIN t_dep d ON e.`d_id` = d.`dep_id`
    <if test="_parameter != null">
      <include refid="Example_Where_Clause" />
    </if>
    <if test="orderByClause != null">
      order by ${orderByClause}
    </if>
  </select>
<!--  Employee selectByPrimaryKeyWithDept(Integer empId);-->
  <select id="selectByPrimaryKeyWithDept" resultMap="WithDeptResultMap">
    select
    <include refid="WithDept_Column_List" />
    FROM t_emp e
    LEFT JOIN t_dep d ON e.`d_id` = d.`dep_id`
    where emp_id = #{empId,jdbcType=INTEGER}
  </select>
```

#  测试DepartmentMapper

>测试dao层的工作
> *  推荐Spring项目可以使用Spring的单元测试，自动注入我们需要的组件
> *      1.导入Spring单元测试模块spring-test（需要将juit的scope标签删除）

- 在applicationContext.xml里面添加

```xml
<!--配置一个可以执行批量的sqlSession-->
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactoryBean"/>
    <constructor-arg name="executorType" value="BATCH"/>
</bean>
```

##  批量插入

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class TestMapper {
    /**
     * 推荐Spring项目可以使用Spring的单元测试，自动注入我们需要的组件
     * 1.导入Spring单元测试模块spring-test
     * 2.@ContextConfiguration指定Spring配置文件的位置
     * 3.@RunWith是Junit里面的注解
     * 4.直接@AutoWired要使用的组件即可
     */
    @Autowired
    DepartmentMapper departmentMapper;
    @Autowired
    EmployeeMapper employeeMapper;
    @Autowired
    SqlSession sqlSession;
    @Test
    public void testCRUD() {
        //1.创建SpringIOC容器
        //ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        //2.从容器中获取Mapper
        //DepartmentMapper dept = ctx.getBean(DepartmentMapper.class);

        System.out.println(departmentMapper);
        //a.测试插入几个部门：一定要生成有参，无参构造器
//        departmentMapper.insertSelective(new Department(null,"开发部"));
//        departmentMapper.insertSelective(new Department(null,"测试部"));
        //b.测试生成员工数据，测试员工插入
//        employeeMapper.insertSelective(new Employee(null,"Jelly","M","Jerry@qq.com",1));
        //c.批量插入多个员工，批量，使用可以执行批量操作的sqlSession
        EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
        for (int i = 0; i < 1000; i++) {
            String uid = UUID.randomUUID().toString().substring(0, 5) + i;
            mapper.insertSelective(new Employee(null,uid,"M",uid+"@qq.com",1));
        }
        System.out.println("批量完成");
    }
}
```

#  分页查询

##  EmployeeController

EmployeeService.java

```java
@Service
public class EmployeeService {
    @Autowired
    EmployeeMapper employeeMapper;
    //查询所有员工
    public List<Employee> getAll() {
        return employeeMapper.selectByExampleWithDept(null);
    }
}
```

```java
@Controller
public class EmployeeController {
    @Autowired
    EmployeeService employeeService;
    /**
     *查询员工数据：分页查询
     */
    @RequestMapping("/emps")
    public String getEmps(@RequestParam(value="pn",defaultValue = "1")Integer pn, Model model) {
        //引入PageHelper分页插件,查询之前只需要调用startPage(pageNum,pageSize)
        PageHelper.startPage(pn, 5);
        //startPage后面跟着的这个查询就是一个分页查询
        List<Employee> emps =  employeeService.getAll();
        //使用pageInfo包装查询后的结果
        //PageInfo就是一个分页Bean
        PageInfo page = new PageInfo(emps,5);
        model.addAttribute("pageInfo", page);
        return "list";
    }
}
```

##  测试分页查询

```java
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration //此注解为WebApplicationContext从springmvc IOC中取出赋值
@ContextConfiguration(locations = {"classpath:applicationContext.xml","classpath:springmvc.xml"})
public class TestMVC {
    //传入Springmvc的ioc
    //@WebAppConfiguration注解的目的是为了将WebApplicationContext获取过来
    @Autowired
    private WebApplicationContext context;
    //虚拟mvc请求，获取到处理结果
    private MockMvc mockMvc;
    @Before //初始化mockMvc
    public void initMockMvc() {
        mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
    }
    @Test
    public void testPage() throws Exception { //andReturn()拿到返回值
        //模拟请求拿到返回值
        MvcResult result = mockMvc.perform(MockMvcRequestBuilders.get("/emps").param("pn", "1")).andReturn();
        //请求成功以后，请求域中会有pageInfo，我们可以取出pageInfo进行验证
        MockHttpServletRequest request = result.getRequest();
        PageInfo pageInfo = (PageInfo) request.getAttribute("pageInfo");
        System.out.println("当前页码：" + pageInfo.getPageNum());
        System.out.println("总页码：" + pageInfo.getPages());
        System.out.println("总记录数：" + pageInfo.getTotal());
        System.out.println("在页面需要连续显示的页码：");
        int[] nums = pageInfo.getNavigatepageNums();
        for (int i : nums) {
            System.out.println(" " + i);
        }
        //获取员工数据
        List<Employee> list = pageInfo.getList();
        for (Employee employee : list) {
            System.out.println("ID:" + employee.getEmpId()+ ", Name=" + employee.getEmpName());
        }

    }
}
```

#  原始list.jsp页面

- 注意：isELIgnored="false"

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>员工列表页面</title>
    <% 
        pageContext.setAttribute("APP_PATH", request.getContextPath());
    %>
    <%--/开始的相对路径，找资源，是以服务器的路径为标准http://localhost:3306--%>
    <script type="text/javascript" src="${APP_PATH}/static/js/jquery-1.12.4.min.js"></script>
    <%--引入Bootstrap的CSS、JS资源--%>
<%--    <link href="${APP_PATH}/static/bootstrap-3.4.1-dist/css/bootstrap.min.css" rel="stylesheet"/>--%>
<%--    <script src="${APP_PATH}/static/bootstrap-3.4.1-dist/js/bootstrap.min.js"></script>--%>

    <!-- 最新版本的 Bootstrap 核心 CSS 文件 -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css"
          integrity="sha384-HSMxcRTRxnN+Bdg0JdbxYKrThecOKuH5zCYotlSAcp1+c8xmyTe9GYg1l9a69psu"
          crossorigin="anonymous">
    <!-- 最新的 Bootstrap 核心 JavaScript 文件 -->
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/js/bootstrap.min.js"
            integrity="sha384-aJ21OjlMXNL5UyIl/XNwTMqvzeRMZH2w8c5cRVpzpU8Y5bApTppSuUkhZXN0VxHd"
            crossorigin="anonymous"></script>
</head>
<body>
    <%--搭建显示页面--%>
    <div class="container">
        <!--标题-->
        <div class="row">
            <div class="col-md-12">
                <h1>SSM-CRUD</h1>
            </div>
        </div>
        <!-- 按钮 -->
        <div class="row">
            <div class="col-md-4 col-md-offset-8">
                <button class="btn btn-primary">新增</button>
                <button class="btn btn-danger">删除</button>
            </div>
        </div>
        <!-- 显示表格数据 -->
        <div class="row">
            <div class="col-md-12">
                <table class="table table-hover">
                    <tr>
                        <th>编号</th>
                        <th>姓名</th>
                        <th>性别</th>
                        <th>邮箱</th>
                        <th>部门</th>
                        <th>操作</th>
                    </tr>
                    <c:forEach items="${pageInfo.list}" var="emp">
                        <tr>
                            <th>${emp.empId}</th>
                            <th>${emp.empName}</th>
                            <th>${emp.gender=="M"?"男":"女" }</th>
                            <th>${emp.email}</th>
                            <th>${emp.department.depName}</th>
                            <th>
                                <button class="btn btn-primary btn-sm">
                                    <span class="glyphicon glyphicon-pencil" aria-hidden="true"></span>编辑
                                </button>
                                <button class="btn btn-danger btn-sm">
                                    <span class="glyphicon glyphicon-trash" aria-hidden="true"></span>删除
                                </button>
                            </th>
                        </tr>
                    </c:forEach>
                </table>
            </div>
        </div>
        <!-- 显示分页信息 -->
        <div class="row">
            <!--分页文字信息  -->
            <div class="col-md-6">
                当前 ${pageInfo.pageNum}页,总${pageInfo.pages}页,总 ${pageInfo.total} 条记录
            </div>
            <!-- 分页条信息 -->
            <div class="col-md-6">
                <nav aria-label="Page navigation">
                    <ul class="pagination">
                        <li>
                            <a href="${APP_PATH}/emps?pn=1">首页</a>
                        </li>
                        <c:if test="${pageInfo.hasPreviousPage}">
                            <li>
                                <a href="${APP_PATH}/emps?pn=${pageInfo.pageNum-1}" aria-label="Previous">
                                    <span aria-hidden="true">&laquo;</span>
                                </a>
                            </li>
                        </c:if>
                        <c:forEach items="${pageInfo.navigatepageNums}" var="page_Num">
                            <c:if test="${page_Num == pageInfo.pageNum}">
                                <li class="active">
                                    <a href="#">${page_Num}</a>
                                </li>
                            </c:if>
                            <c:if test="${page_Num != pageInfo.pageNum}">
                                <li>
                                    <a href="${APP_PATH}/emps?pn=${page_Num}">${page_Num}</a>
                                </li>
                            </c:if>
                        </c:forEach>
                        <c:if test="${pageInfo.hasNextPage }">
                            <li>
                                <a href="${APP_PATH}/emps?pn=${pageInfo.pageNum+1}" aria-label="Next">
                                    <span aria-hidden="true">&raquo;</span>
                                </a>
                            </li>
                        </c:if>
                        <li><a href="${APP_PATH}/emps?pn=${pageInfo.pages}">末页</a></li>
                    </ul>
                </nav>
            </div>
        </div>
    </div>
</body>
</html>
```

#  查询-ajax

1. index.jsp页面直接发送ajax请求进行员工分页数据的查询
2. 服务器将查出的数据，以JSON字符串的形式返回给浏览器
3. 浏览器收到js字符串。可以使用js对JSON进行解析，使用js通过dom增删改改变页面。
4. 返回JSON。实现客户端的无关性。

>a、之前的index.jsp页面
>
>```jsp
><%@ page contentType="text/html;charset=UTF-8" language="java" %>
><jsp:forward page="/emps"></jsp:forward>
>```
>
>b、转到emps请求，控制器EmployeeController收到emps请求后，转发到（return "list"）list.jsp页面。
>
>c、如今将分页的数据以JSON字符串的形式返回

##  @ResponseBody

>自动将返回的对象转为JSON字符串（springmvc）,需导入jackson包，负责将PageInfo对象转换成json字符串。

- Msg类引入链式编程

```java
@Data
public class Msg {
    //JSON数据通用的返回类
    private int code;//状态码 100成功 200失败
    private String msg;//提示信息
    //用户要返回给浏览器的数据
    private Map<String, Object> data = new HashMap<>();

    public static Msg success() {
        Msg result = new Msg();
        result.setCode(100);
        result.setMsg("处理成功");
        return result;
    }

    public static Msg fail() {
        Msg result = new Msg();
        result.setCode(200);
        result.setMsg("处理失败");
        return result;
    }

    //链式编程
    public Msg add(String key, Object value) {
        this.getData().put(key, value);
        return this;
    }
}
```

- 修改EmployeeController：改为使用 JSON 格式传输数据，需要 Jackson 的依赖。

```java
@Controller
public class EmployeeController {
    @Autowired
    EmployeeService employeeService;

    @RequestMapping("/emps")
    @ResponseBody
    //自动将返回的对象转为JSON字符串（springmvc）,需导入jackson包，负责将PageInfo对象转换成json字符串
    public Msg getEmpsWithJson(@RequestParam(value = "pn", defaultValue = "1")Integer pn) {
        PageHelper.startPage(pn, 5);
        List<Employee> emps =  employeeService.getAll();
        PageInfo page = new PageInfo(emps,5);
        return Msg.success().add("pageInfo",page);
    }
}    
```

##  修改 index.jsp 采用 Ajax 请求

>页面加载完成以后，直接去发送一个ajax请求，要到分页数据

