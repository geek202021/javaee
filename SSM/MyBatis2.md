#  MyBatis

- [1-百知教育笔记](https://blog.csdn.net/unique_perfect/article/details/105587367)   [2-非常详细的笔记](https://gitee.com/hong-0105/hong-note/blob/master/%E6%A1%86%E6%9E%B6/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/MyBatis/MyBatis.md#mybatis)

##  IDEA配置xml文件模板

>Settings --> Editor --> File and Code Templates --> Unnamed --> name=mybatis-mapper / mybatis-config

- 建数据库表

```sql
DROP DATABASE IF EXISTS mybatis;
CREATE DATABASE mybatis;
USE mybatis;
CREATE TABLE t_user(
	`id` INT PRIMARY KEY AUTO_INCREMENT,
	`name` VARCHAR(30),
	`password` VARCHAR(12)
);
INSERT INTO t_user(`name`,`password`) VALUE('zhangsan',1234);
SELECT * FROM t_user;
```

- 添加依赖

```xml
<!--只有这个依赖才能保证在dao文件夹下的UserDAO.xml被加载到target/classes路径下-->
<build>
    <!--资源插件：处理src/main/java目录中的xml文件-->
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>false</filtering>
        </resource>     
    </resources>
</build>
```

- mapper文件的配置

```xml
<mappers>
    <!--第一种方式，resources="mapper文件的路径"-->
    <mapper resource="com/jun/dao/UserDao.xml"/>
</mapper>
<!--第二种方式，使用package。name:包名，mapper文件所在的包名。
使用要求：mapper文件和dao接口在同一目录。mapper文件和dao接口名称完全一样。-->
<package name="com.jun.dao"/>
```

##  返回对象List集合

- 创建mybatis-config.xml配置文件
- 使用mapper的`resource`属性指定mapper文件的路径，从类路径开始的路径信息。从target/classes(类路径)开启。
- **resource="mapper文件的路径，使用 / 分割路径"**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="hrj"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="UserDAO.xml"/>
    </mappers>
</configuration>
```

- 创建UserDAO接口和UserDAO.xml配置文件

>- `namespace`:命名空间，配置接口的包名+类名（要求使用dao接口的全限定名称）
>- id就是接口的方法名；resultType:返回值类型(包名+类名),是SQL语句执行后得到ResultSet，遍历这个ResultSet得到java对象的类型。
>- sqlSession.getMapper();使用的是MyBatis的动态代理机制，getMapper能获取dao接口对应的实现类对象。

```xml
<!--
    //UserDAO接口：查询所有用户信息
    public List<User> queryAllUsers();
-->
<mapper namespace="com.jun.dao.UserDAO">
    <select id="queryAllUsers" resultType="com.jun.pojo.User">
            select * from t_user;
    </select>
</mapper>
resultType:告诉Mybatis，执行sql语句，把数据赋值给哪个类型的java对象。
```

- 测试程序

```java
public void test01() throws Exception {
    InputStream ins = Resources.getResourceAsStream("mybatis-config.xml"); //配置文件在类路径下
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(ins);
    //产生一个SqlSession会话对象连接MySql数据库
    SqlSession sqlSession = sqlSessionFactory.openSession();
    //创建一个接口:接口的方法名与Mapper文件名要一致
    //Mapper文件
    UserDAO userDAO = sqlSession.getMapper(UserDAO.class);
    List<User> userList = userDAO.queryAllUsers();
    userList.forEach(user -> {
        System.out.println(user);
    });
}
```

>重复代码提取:
>
>- @Before 注解表示任何的单元测试@Test在执行之前都要执行@Before （import org.junit.Before;）

```java
public class TestMybatis {
    private SqlSessionFactory sqlSessionFactory;
    //公共提取
    @Before
    public void beforeMybatis() throws Exception {
        InputStream ins = Resources.getResourceAsStream("mybatis-config.xml"); //配置文件在类路径下
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(ins);
    }
//方式二：
public class MyBatisUtils{
    private static SqlSessionFactory sqlSessionFactory;
    static{
        InputStream ins = Resources.getResourceAsStream("mybatis-config.xml");
        sqlSessionFactory = new SqlSessionFactoryBuilder().build(ins);    
    }
    //获取SqlSession的方法
    public static SqlSession getSqlSession(){
        SqlSession sqlSession = null;
        if(sqlSessionFactory != null){
            sqlSession=sqlSessionFactory.openSession();//非自动提交事务
        }
        return sqlSession;
    }
}    
```

##  返回单个对象

- UserMapper.xml文件

```xml
public User queryUserById(Integer id);
parameterType：形参的类型（自定义JavaBean必须加：包名+类名）,可以省略不写
#{}是Mybatis的占位符，相当于jdbc的？
<select id="queryUserById" parameterType="Integer" resultType="com.jun.pojo.User">
        select `id`,`name`,`password` from t_user where id = #{id}
</select>
```

- UserMapper接口，增加功能

>openSession()：无参数/参数为false，获取的是非自动提交事务的SqlSession对象

```java
public interface UserMapper {
    //查询所有用户信息
    public List<User> queryAllUsers();
    //根据用户ID查询一个用户信息
    public User queryUserById(Integer id);
}
public void test2() {
    SqlSession sqlSession = null;
    try {
        sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.queryUserById(1);
        System.out.println(user);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (sqlSession != null) {
            sqlSession.close();
        }
    }
}
```

##  更改用户信息

- UserMapper接口：public void updateUser(User user);  

```xml
<mapper>
    <update id="updateUser">
    update t_user set `name` = #{name}, `password` = #{password} where id = #{id}
	</update>
</mapper>
```

##  插入记录并返回主键

>insert标签配置insert的sql语句：id配置唯一标识；
>
>- parameterType配置方法的参数 类型（它是可选的，一般情况下，一个参数，且是JavaBean的时候。建议写上，提高可写性）
>- useGeneratedKeys="true": 开启获取自增主键的策略
>- keyColumn: 指定数据库主键的列名
>-  keyProperty: 指定对应的主键属性, ps(获取到主键值后, 将这个值封装给javaBean的哪个属性)

```xml
<!-- public int addUser(User user);-->
<insert id="addUser" parameterType="com.jun.pojo.User" useGeneratedKeys="true" keyProperty="id">
        insert into t_user(name,password) values (#{name},#{password})
</insert>
```

>扩展：
>
>- selectKey: 配置查询主键的sql语句
>- keyProperty:查出的主键值封装给javaBean的哪个属性
>- order: 
>  - BEFORE:当前sql在插入sql之前运行；AFTER:当前sql在插入sql之后运行

```xml
扩展二：
<insert id="addUser" parameterType="com.jun.pojo.User">
    <!--返回主键ID-->
    <selectKey keyProperty="id" resultType="Integer" order="AFTER">
    	select LAST_INSERT_ID()
    </selectKey>
    insert into t_user()values()
</insert>
```

- MyBatis默认不是自动提交事务的，所以在insert、update、delete后要手工提交事务。

```java
public void test5() {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    User user = new User();
    user.setName("root");
    user.setPassword("huawei");
    int count = mapper.addUser(user);
    System.out.println("count = " + count);
    sqlSession.commit();
    sqlSession.close();
}
```

##  返回对象Map集合

```java
UserMapper接口中的方法
//查询所有用户集合，保存到Map中。@MapKey中的参数是：User对象中的//private Integer id;
@MapKey("id")
public Map<Integer,User> queryUsersForMap();
```

```xml
<mapper>
	<select id="queryUsersForMap" resultType="Map">
        select * from t_user
	</select>
</mapper>
```

```java
public void test6() {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    //查询所有
    Map<Integer, User> map = mapper.queryUsersForMap();
    System.out.println(map);
}
```

#  Mybatis配置信息

```xml
<configuration>
	<!--设置日志-->
    <settings>
    	<setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
    <!--声明别名-->
    <typeAliases>
    	<!--第一种语法格式：
			type:java类型的全限定名称（自定义类型）
			alias:自定义别名
		-->
        <typeAlias type="com.jun.bean.User" alias="user"/>
    </typeAliases>
</configuration>
```

##  properties标签

>mybatis-config.xml  & jdbc.properties

```xml
<configuration>
    <!--读取classpath下的配置文件-->
    <properties resource="jdbc.properties"/>
    ……………………………………
    <property name="driver" value="${jdbc.driverClassName}"/>
</configuration>

jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis?useSSL=false
jdbc.username=root
jdbc.password=hrj
initialSize=5
maxActive=10
```

##  类型别名typeAliases

- 在mybatis主配置文件，使用typeAliases标签声明别名
  - <typeAlias type="com.jun.bean.User" alias="user"/>
- 在mapper文件中，resultType=“别名”
  - /<select id="selectById" resultType="user">/</select>

```xml
<typeAliases>
    <!--指定包，mybatis会把这个包中的所有类名作为别名（不区分大小写）。优点：使用方便，一次给多个类定义别名。缺点：别名不能自定义，必须是类名，容易造成类名冲突-->
    <package name="com.jun.pojo"/>
</typeAliases>
```

##  映射器mappers

```xml
<!--将包内的映射器接口实现全部注册为映射器-->
<mappers>
	<package name="org.mybatis.builder"/>
</mappers>
```

#  深入理解参数

##  多个参数-使用@Param

```java
//多个普通参数：根据用户名及性别查询用户信息@Param
public List<User> queryUsersByIdAndNameParam(@Param("id") Integer id, @Param("name") String name);

//UserMapper.xml
<select id="queryUsersByIdAndNameParam" resultType="com.jun.pojo.User">
        select id, name, password from t_user where id = #{id} and name = #{name}
</select>
```

##  Map形参

```java
//形参是Map 根据用户名及性别查询用户信息
public List<User> queryUserByIdAndNameMap(Map map);
//UserMapper.xml
<select id="queryUserByIdAndNameMap" resultType="com.jun.pojo.User">
        select id, name, password from t_user where id = #{id} and name = #{name}
</select>

//test
public void test8() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Map map = new HashMap();
        map.put("id", 3);
        map.put("name", "root");
        List<User> users = mapper.queryUserByIdAndNameMap(map);
        users.forEach(u ->{
            System.out.println(u);
        });
    }    
```

#  模糊查询${}

- #{}：占位符 ？ '张三'   **CONCAT('%',#{name},'%')**
- ${}：字符串拼接  张三  **'%${name}%'**

>第一种方式：在java程序中，把like的内容组装好。把这个类容传入到sql语句

UserMapper.xml

```xml
<select id="queryUserLikeOne" resultType="com.jun.pojo.User">
    select id,name,password from t_user where name like #{name}
</select>
```

```java
public List<User> queryUserLikeOne(@Param("name") String name);
public void test10() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> list = mapper.queryUserLikeOne("%张%");
        list.forEach(u -> {
            System.out.println(u);
        });
    }
```

>sql语句like的格式：where name like "%"空格#{name}空格"%"
>
> select id,name,password from t_user where name like "%" #{name} "%"

```xml
//UserMapper.xml
<select id="queryUsersByNameLike" resultType="com.jun.pojo.User">
        select id,name,password from t_user where name like '%${name}%'
</select>
<select id="queryUsersByNameLike" resultType="com.jun.pojo.User">
        select id,name,password from t_user where name like CONCAT('%',#{name},'%')
</select> 
```

```java
//模糊查询
public List<User> queryUsersByNameLike(@Param("name") String name); 
//test
 public void test9() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> list = mapper.queryUsersByNameLike("oo");
        list.forEach(u -> {
            System.out.println(u);
        });
    }    
```

###   #和$的区别

>#占位符：Mybatis处理#{}使用的jdbc对象是 PrepareStatement对象
>
>- 使用的PrepareStatement对象，执行sql语句，效率高。
>- 使用的PrepareStatement对象，能避免sql注入，sql语句执行更安全 

>$占位符：${}使用的

```java
@Select("select * from user where username like #{username} ")
List<User> users = userDao.findUserByName("%张%");
@Select("select * from user where username like '%${value}%' ")
List<User> users = userDao.findUserByName("张");
```

- #{}是预编译处理，${}是字符串替换。
- Mybatis 在处理#{}时，会将 sql 中的#{}替换为?号，调用 PreparedStatement 的set 方法来赋值。
- Mybatis 在处理$ {}时，就是把${}替换成变量的值。
- 使用#{}可以有效的防止 SQL 注入，提高系统安全性。

#  resultType&resultMap

- resultType：全自动映射
  - 表示mybaits执行sql后得到java对象类型，规则是数据库中的同名列赋值给同名属性。
- 复杂情况下：resultMap 半自动映射
  - 字段名称不与属性名称一致  Sql语句 别名

>列名和属性名不一样的解决方式

```java
public class Person {
    private Integer cid;
    private String cname;
    private String password;
public void test10() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        Person person = mapper.queryPersonById(3);
        System.out.println(person);
    }    
```

```xml
<!--    resultMap测试:列名和属性名不同的情况
        id：给resultMap的映射关系起个名称，唯一值。
        type：java类型的权限定名称
-->
    <resultMap id="personMap" type="com.jun.pojo.Person">
        <!--主键类型使用id标签-->
        <id column="id" property="cid"/>
        <!--非主键类型使用result标签-->
        <result column="name" property="cname"/>
        <!--列名和属性名相同不用定义-->
    </resultMap>
    <select id="queryPersonById" resultMap="personMap">
        select id,name,password from t_user where id = #{id}
    </select>
```

>使用列别名，解决列名和属性名不同的问题

```xml
<select id="queryPersonById" resultType="com.jun.pojo.User"
        select id AS cid,name AS cname,password from t_user where id = #{id}
</select>
```

#  动态sql

>同一个dao的方法，根据不同的条件可以表示不同的sql语句，主要是where部分有变化

##  If标签

```xml
<!--test:使用对象的属性值作为条件-->
<select id="selectIf" resultType="com.jun.pojo.User">
    select * from t_user where
    <if test="name != null and name !=''">
        name = #{name}
    </if>
    <if test="id > 2">
        or id = #{id}
    </if>
</select>

List<User> selectIf(User user);

SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = new User();
        user.setName("张三");
        user.setId(3);
        List<User> users = mapper.selectIf(user);
        users.forEach(u -> System.out.println(u));
        sqlSession.close();
```

##  Where标签

>使用where里面有一个或多个if标签，当有一个if标签判断条件为true，where标签会转为`WHERE`关键字附加到sql语句后面，如果if没有一个条件为true，则忽略where和里面的if
>
>- where标签删除和他最近的or或者and

```java
List<User> selectWhere(User user);
public void test13() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = new User();
        user.setName("张三");
        user.setId(1);
        List<User> users = mapper.selectWhere(user);
        users.forEach(u -> System.out.println(u));
        sqlSession.close();
    }
```

```xml
<select id="selectWhere" resultType="com.jun.pojo.User">
    select * from t_user
    <where>
        <if test="name != null and name !=''">
            or name = #{name}
        </if>
        <if test="id > 2">
            or id &lt; #{id}
        </if>
    </where>
</select>
```

##  foreach循环标签

>1.手工实现循环

```java
public void test14() {
    List<Integer> list = new ArrayList<>();
    list.add(3);
    list.add(4);
    list.add(5);
    //查询id在list中的User
    //select * from user where id in (3,4,5)
    StringBuffer sb = new StringBuffer("");
    sb.append("select * from user where id in ");
    sb.append("(");
    //使用循环，把List数据追加到sb字符串中
    for (int i = 0; i < list.size(); i++) {
        Integer item = list.get(i); //item是集合成员
        sb.append(item);//添加成员到sb字符串
        sb.append(",");//集合成员之间的分隔符
    }
    sb.deleteCharAt(sb.length() - 1);
    sb.append(")");
    System.out.println(sb);
}
```

>2.foreach循环简单类型的List

```java
List<User> selectForeachOne(List<Integer> list);
public void test15() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<Integer> list = new ArrayList<>();
        list.add(3);
        list.add(4);
        list.add(5);
        List<User> user = mapper.selectForeachOne(list);
        user.forEach(u -> System.out.println(u));
        sqlSession.close();
    }
```

```xml
<select id="selectForeachOne" resultType="com.jun.pojo.User">
   select * from t_user
   <if test="list != null and list.size>0">
    where id in
       <foreach collection="list" open="(" close=")" separator="," item="myId">
          #{myId}
       </foreach>
   </if>
</select>
```

>3.foreach循环对象类型的List

```java
List<User> selectForeachTwo(List<User> userList);

<select id="selectForeachTwo" resultType="com.jun.pojo.User">
        select * from t_user
        <if test="list != null and list.size>0">
            where id in
            <foreach collection="list" open="(" close=")" separator="," item="stu">
                #{stu.id}
            </foreach>
        </if>
</select>

@Test
    public void test16() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> list = new ArrayList<>();
        User u1 = new User();
        u1.setId(3);
        User u2 = new User();
        u2.setId(4);
        list.add(u1);
        list.add(u2);
        List<User> user = mapper.selectForeachTwo(list);
        user.forEach(u -> System.out.println(u));
        sqlSession.close();
    }            
```

##  sql标签

>sql标签定义代码片段，可以是表名，字段，where条件
>
>- 在mapper文件中定义sql代码片段<sql id="唯一字符串"> 部分sql语句《/sql》
>- 其他位置，使用include标签引用某个代码片段。

```xml
<sql id="selectUser">
	select * from t_user
</sql>
```

#  Mybatis通用分页插件

[PageHelper](https://github.com/pagehelper/Mybatis-PageHelper)

#  级联映射

- 一对一映射：<association

>数据库表

```sql
CREATE TABLE teacher(
	`id` INT PRIMARY KEY AUTO_INCREMENT,
	`name` VARCHAR(15) DEFAULT NULL
)ENGINE=INNODB DEFAULT CHARSET=utf8;
INSERT INTO teacher(`name`) VALUES ("Mike");
CREATE TABLE student(
	`id` INT PRIMARY KEY AUTO_INCREMENT,
	`name` VARCHAR(15) DEFAULT NULL,
	`tid` INT,
	FOREIGN KEY(tid) REFERENCES teacher(id)
)ENGINE=INNODB DEFAULT CHARSET=utf8;
INSERT INTO student(`name`,`tid`) VALUES ('小明',1);

CREATE TABLE employee(
	id INT PRIMARY KEY AUTO_INCREMENT,
	`name` VARCHAR(32) DEFAULT NULL
)ENGINE=INNODB DEFAULT CHARSET=utf8;
CREATE TABLE department(
	id INT(11) PRIMARY KEY AUTO_INCREMENT,
	dept_name VARCHAR(255)	
);
ALTER TABLE employee ADD COLUMN d_id INT(11);
ALTER TABLE employee ADD CONSTRAINT fk 
FOREIGN KEY(d_id) REFERENCES department(id);
```

>mybatis-config.xml & StudentMapper.xml

```xml
<mappers>
	<package name="com.jun.dao"/>
</mappers>

<mapper namespace="com.jun.dao.StudentMapper">
    <!--<resultMap id="queryAllStudents" type="com.jun.pojo.Student">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="tid" property="teacher.id"/>
        <result column="tname" property="teacher.name"/>
    </resultMap>-->
    <!--办法二：-->
    <resultMap id="queryAllStudents" type="com.jun.pojo.Student">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <association property="teacher"  javaType="Teacher">
            <id column="tid" property="id"/>
            <result column="tname" property="name"/>
        </association>
    </resultMap>
    <select id="queryStudents" resultMap="queryAllStudents">
        select s.id ,s.name ,s.tid, t.name tname ,t.id tid from student s
        inner join teacher t
        on t.id = s.tid
    </select>
</mapper>

<mapper namespace="com.jun.dao.EmployeeMapper">
    <!--<resultMap id="getEmp" type="com.jun.pojo.Employee">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <result column="d_id" property="dept.id"/>
        <result column="dept_name" property="dept.deptName"/>
    </resultMap>-->
    <resultMap id="getEmp" type="com.jun.pojo.Employee">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <!--
            association可以指定联合的javaBean对象
            property="dept"：指定哪个属性是联合的对象
            javaType：指定这个属性对象的类型【不能省略】
        -->
        <association property="dept" javaType="com.jun.pojo.Department">
            <id column="did" property="id"/>
            <result column="dept_name" property="deptName"/>
        </association>
    </resultMap>
    <select id="getEmpAndDept" resultMap="getEmp">
        SELECT e.id,e.name,e.d_id,d.id did,d.dept_name dept_name
        FROM employee e,department d WHERE e.d_id = d.id AND e.id = #{id}
    </select>
</mapper>
```

#  延迟加载，按需分配

>延迟加载分2步走：
>
>- 第一步：查询Key表，**关联第二步**
>- 第二步：查询Lock表

>1.开启Mybatis延迟加载的行为，在mybatis-config.xml文件中配置settings

```xml
<settings>
   <!--打开延迟加载开关-->
   <setting name="lazyLoadingEnabled" value="true"/>
   <!--将积极加载改为消极加载：按需加载-->
   <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

>StudentMapper.java & TeacherMapper.java

```java
//延迟加载
//1.根据id，查询Student表
public Student queryStudentById(Integer id);
//延迟查询，查询Lock表（Key表是：StudentMapper）
public Teacher queryTeacherById(Integer id);

public void test2() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        Student student = mapper.queryStudentById(1);
        System.out.println("ID = " + student.getId() + " ,姓名 = " + student.getName());
        //延迟加载
        System.out.println("======================");
        System.out.println(student.getTeacher());
    }
```

>StudentMapper.xml & TeacherMapper.xml

```xml
<mapper namespace="com.jun.dao.TeacherMapper">
<!--    //2.根据id，查询Teacher表-->
    <select id="queryTeacherById" resultType="com.jun.pojo.Teacher">
        select id,name from teacher where id = #{id}
    </select>
</mapper>
<mapper namespace="com.jun.dao.StudentMapper">
<!--    //1.根据id，查询Student表-->
    <resultMap id="queryStudent" type="com.jun.pojo.Student">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <!--延迟加载，关联teacher表
            select ：关联另一个Mapper接口及Mapper文件（包名+类名+方法名）
            column：传递参数到另一个Mapper接口
            javaType：返回值类型
        -->
        <association property="teacher" column="tid" javaType="Teacher" select="com.jun.dao.TeacherMapper.queryTeacherById"/>    
    </resultMap>
    <select id="queryStudentById" resultMap="queryStudent">
        select id,name,tid from student where id = #{id}
    </select>
</mapper>
```

#  一对多查询<collection

```java
public class Teacher {
    private int id;
    private String name;
    private List<Student> students;
}    
//TeacherMapper.java查询所有老师及老师对应的学生
public List<Teacher> queryAllTeachers();

public void test3() {
    SqlSession sqlSession = sqlSessionFactory.openSession();
    TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
    List<Teacher> teachers = mapper.queryAllTeachers();
    for (Teacher teacher : teachers) {
        System.out.println(teacher);
        List<Student> students = teacher.getStudents();
        for (Student stu : students) {
            System.out.println(stu);
        }
    }
```

>TeacherMapper.xml

```xml
<resultMap id="selectAllTeachers" type="com.jun.pojo.Teacher">
    <id column="id" property="id"/>
    <result column="name" property="name"/>
    <!--一对多映射：ofType：指定集合里面元素的类型,property="students"子对象的属性名称-->
    <collection property="students" ofType="Student">
        <id column="sid" property="id"/>
        <id column="sname" property="name"/>
    </collection>
</resultMap>
<select id="queryAllTeachers" resultMap="selectAllTeachers">
    select t.id,t.name,s.id sid,s.name sname from teacher t inner join student s on s.tid = t.id
</select>
```

