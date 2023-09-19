# Mybatis

[什么是Mybatis？最全Mybatis学习笔记 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/351830443)

[mybatis – MyBatis 3 | 入门](https://mybatis.org/mybatis-3/zh/getting-started.html)



### 1.Mybatis，Maven项目搭建

#### 1.什么是mabatis

* 持久层框架
* 避免JDBC代码，手动设置参数和获取结果集
* 简单的注解，xml，接口，pojo，映射源生类型

#### 2.可能遇到问题

> 1.配置文件没有注册
>
> 2.绑定接口不对
>
> 3.方法名不对
>
> 4.返回类型不对
>
> 5.Maven资源导出问题



### 2.CRUD

#### 1.注意事项

* namespace包名要与接口名称一致
* id对应方法名
* resultType返回值类型
* parameterType参数类型

注意：select不用递交事务，update、delete、insert都需要递交事务`sqlSession.commit`



#### 2.编写流程

* 编写接口

  ```java
      //获取所有用户信息
      List<User> getListUser();
  
      //根据id获取信息
      User getUserById(int id);
  
      //insert
      int addUser(User user);
  
      int update(User user);
  
      int delete(int id);
  ```

  

* 编写xml中sql语句

  ```
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
  
  <mapper namespace="com.rst.dao.Userdao">
  </mapper>
  ```

  ```xml
    <!--    查询所有用户语句-->
      <select id="getListUser" resultType="com.rst.pojo.User">
          select * from mybatis.user;
      </select>
  
      <!--    根据id查询-->
      <select id="getUserById" resultType="com.rst.pojo.User" parameterType="int">
          select * from mybatis.user where id = #{id}
      </select>
  
      <insert id="addUser" parameterType="com.rst.pojo.User">
          insert into mybatis.user (id,name,pwd) values (#{id},#{name},#{pwd});
      </insert>
  
      <update id="update" parameterType="com.rst.pojo.User">
          update mybatis.user
          set name=#{name},
              pwd=#{pwd}
          where id = #{id};
      </update>
  
      <delete id="delete" parameterType="int">
          delete
          from mybatis.user
          where id = #{id};
      </delete>
  ```

  

* test中测试 

  ```java
      @Test
      public void getUserById() {
          SqlSession sqlSession = MybatisUtil.getSqlSession();
          Userdao mapper = sqlSession.getMapper(Userdao.class);
  
          User user = mapper.getUserById(4);
          System.out.println(user);
          sqlSession.close();
      }
  
      //增删改需要提交事务
      @Test
      public void addUser() {
          SqlSession sqlSession = MybatisUtil.getSqlSession();
          Userdao mapper = sqlSession.getMapper(Userdao.class);
  
          mapper.addUser(new User(4, "大大", "123456"));
  
          //提交事务
          sqlSession.commit();
          sqlSession.close();
      }
  
      @Test
      public void update() {
          SqlSession sqlSession = MybatisUtil.getSqlSession();
          Userdao mapper = sqlSession.getMapper(Userdao.class);
  
          mapper.update(new User(4, "官方的", "12226"));
  
          //提交事务
          sqlSession.commit();
          sqlSession.close();
      }
  
      @Test
      public void delete() {
          SqlSession sqlSession = MybatisUtil.getSqlSession();
          Userdao mapper = sqlSession.getMapper(Userdao.class);
  
          mapper.delete(4);
  
          //提交事务
          sqlSession.commit();
          sqlSession.close();
      }
  ```



* mybatis配置文件

  ```
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "https://mybatis.org/dtd/mybatis-3-config.dtd">
  
  <!--核心配置文件-->
  <configuration>
      <environments default="development">
          <environment id="development">
              <transactionManager type="JDBC"/>
              <dataSource type="POOLED">
                  <property name="driver" value="com.mysql.jdbc.Driver"/>
                  <property name="url"
                            value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf-8"/>
                  <property name="username" value="root"/>
                  <property name="password" value="200218"/>
              </dataSource>
          </environment>
      </environments>
      <mappers>
          <mapper resource="com/rst/dao/UserMapper.xml"/>
      </mappers>
  </configuration>
  ```


#### 3.可能存在问题：

```xml
<mappers>
        <mapper resource="com/rst/dao/UserMapper.xml"/></mappers>

 parameterType="com.rst.pojo.User"

<!-- 两个地方分别使用/路径和.包名,不要搞错 -->
```



pom文件配置资源加载，否则resouces之外的资源不会被打包到target下。

```
<build>
    <!-- 对于项目资源文件的配置放在build中 -->
    <resources>
        <resource>
            <directory>src/main/resources/${env}</directory>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.properties</include>
                <include>**/*.tld</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```



#### 4.万能Map

> map传递参数，sql使用key
>
> 对象传递参数，使用对象的属性
>
> 基本类型参数，sql可以直接取到
>
> 多个参数使用==map==，或==注解==



 接口

```java
 User getUserByMap(Map<String,Object> map);
```

配置文件

```xml
    <!--    根据map查询-->
    <select id="getUserByMap" resultType="com.rst.pojo.User" parameterType="map">
        select *
        from mybatis.user
        where id = #{id1}
          and name = #{name1};
    </select>
```

测试

```java
    @Test
    public void getUserByMap() {
        SqlSession sqlSession = MybatisUtil.getSqlSession();
        Userdao mapper = sqlSession.getMapper(Userdao.class);

        HashMap<String, Object> map = new HashMap<>();
        map.put("id1", 1);
        map.put("name1", "a");
        User user = mapper.getUserByMap(map);
        System.out.println(user);


        sqlSession.close();
    }
```



#### 5.模糊查询

```xml
    <!--   模糊查询-->
    <select id="getUserLike" resultType="com.rst.pojo.User">
        select *
        from mybatis.user
        where name like #{value};
    </select>
```



1.直接通过value传入通配符%

2.在sql中进行拼接'%'#{value}'%'



### 3.配置解析

#### 1.核心配置文件

* `mybatis-config.xml`

  ```xml
  configuration（配置）
      properties（属性）
      settings（设置）
      typeAliases（类型别名）
      typeHandlers（类型处理器）
      objectFactory（对象工厂）
      plugins（插件）
      environments（环境配置）
      environment（环境变量）
      transactionManager（事务管理器）
      dataSource（数据源）
      databaseIdProvider（数据库厂商标识）
      mappers（映射器）
  ```

**标签要按顺序写**（xml可以规定标签顺序）

#### 2.环境配置 

mybatis可以配置适应多套环境

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

mybatis默认事务管理器 `JDBC`;默认连接池`POOLED`

#### 3.属性（properties）

这些属性可以在外部进行配置，并可以进行动态替换。你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。

`db.properties`

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf-8
username=root
password=200218
```

```xml
<properties resource="db.properties">
	<property name="password" value="200218"/>
</properties>
```



1.可以使用外部配置文件
2.可以增加一些属性
3.`properties`和`db.properties`==有相同属性优先选择配置文件里的==



#### 4.类型别名(typeAliases)

* 类型别名可为 Java 类型设置一个缩写名字。 

* 意在降低冗余的全限定类名书写。

```xml
<!--场景：实体类少-->
<typeAliases>
	<typeAlias type="com.rst.pojo.User" alias="User"/>
</typeAliases>
```

每一个在包 `domain.blog` 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 比如 `domain.blog.Author` 的别名为 `author`；若有注解，则别名为其注解值。

```xml
<!--场景：实体类多-->
<typeAliases>
	<package name="com.rst.pojo"/>
</typeAliases>
```

```java
@Alias("User")
public class User_ssda(){
    .....
}
```



#### 5.设置(settings)

```
cacheEnabled  //缓存
lazyLoadingEnabled //懒加载
mapUnderscoreToCamelCase //驼峰命名映射
logImpl //日志实现方法
```



#### 6.其他配置

* typeHandlers（类型处理器）

* objectFactory（对象工厂）    

* plugins（插件）

  * mybatis plus
  * mybatis spring
  * MyBatis Generator Core



#### 7.mapper(映射器)

注册绑定我们的mapper文件



方式一(推荐)：

```xml
<mappers>
	<mapper resource="com/rst/dao/UserMapper.xml"/>
</mappers>
```



方式二(接口类完全限定类名)：

```xml
<mappers>
	<mapper class="com.rst.dao.Userdao"/>
</mappers>
```

注意:

* ==接口文件和mapper文件必须同名==
* ==两个文件必须在同一个包下==



方式三(包名)：

```xml
<mappers>
	<package name="com.rst.dao"/>
</mappers>
```

注意:

* ==接口文件和mapper文件必须同名==



#### 8.作用域和生命周期

作用域和生命周期是至关重要的，因为错误的使用会导致非常严重的**并发**问题。

**SqlSessionFactoryBuilder**：

* 一旦创建了 SqlSessionFactory，就不再需要它了。
* 局部变量

**SqlSessionFactory** ：

* SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，好比**数据库连接池**
* SqlSessionFactory 的最佳作用域是应用作用域。
* 最简单的就是使用单例模式或者静态单例模式。

 **SqlSession**：

* 连接到连接池的一个请求
* SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。
* 用完就关闭，否则资源被占用



### 4.属性名和字段名不一致

#### 1.问题

解决方法：

* 起别名

  ```xml
  x select id, name, pwd as passwordfrom mybatis.user;
  ```



#### 2.ResultMap

结果映射集

```
id name pwd
id name password
```

```xml
<resultMap id="Users" type="user">
	<result property="password" column="pwd"/>
</resultMap>

<select id="getListUser" resultMap="Users">
	select *
	from mybatis.user;
</select>
```

* `ResultMap` 的优秀之处——你完全可以不用显式地配置它们。
* ResultMap 的设计思想是，对简单的语句做到零配置，对于复杂一点的语句，只需要描述语句之间的关系就行了。



### 5.日志

#### 1.日志工厂

如果一个数据库操作出现错误，我们需要排错。

pre：sout，debug

now：日志工厂



在settings中配置该属性

| 设置名  | 描述                                                  | 有效值                                                       | 默认值 |
| :------ | :---------------------------------------------------- | :----------------------------------------------------------- | :----- |
| logImpl | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。 | SLF4J \| LOG4J（3.5.9 起废弃） \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置 |

* LOG4J
* STDOUT_LOGGING (标准日志工厂实现)

```xml
<settings>
	<setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

```
Opening JDBC Connection
Created connection 109069556.
Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@68044f4]
==>  Preparing: select * from mybatis.user;
==> Parameters: 
<==    Columns: id, name, pwd
<==        Row: 1, 李四, 456
<==        Row: 2, 李光, 222
<==        Row: 3, 刘明, 313
<==      Total: 3
User{id=1, name='李四', password='456'}
User{id=2, name='李光', password='222'}
User{id=3, name='刘明', password='313'}
Resetting autocommit to true on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@68044f4]
Closing JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@68044f4]
Returned connection 109069556 to pool.
```



#### 2.LOG4J

什么是LOG4J：

* Log4j是[Apache](https://baike.baidu.com/item/Apache/8512995?fromModule=lemma_inlink)的一个[开源项目](https://baike.baidu.com/item/开源项目/3406069?fromModule=lemma_inlink)，通过使用Log4j，我们可以控制日志信息输送的目的地是[控制台](https://baike.baidu.com/item/控制台/2438626?fromModule=lemma_inlink)、文件、GUI组件等

* 可以控制每一条日志的输出格式
* 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程
* 这些可以通过一个[配置文件](https://baike.baidu.com/item/配置文件/286550?fromModule=lemma_inlink)来灵活地进行配置，而不需要修改应用的代码

1.导入jar包

```xml
  <!-- https://mvnrepository.com/artifact/log4j/log4j -->
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

2.log4j.properties

```properties
#将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
log4j.rootLogger=DEBUG,console,file
#控制台输出的相关设置
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Target=System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n
#文件输出的相关设置
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/logFile.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

3.配置日志实现为log4j



**简单实现**

```java
Logger logger = Logger.getLogger(UserdaoTest.class);

@Test
public void Log4j() {
    logger.info("info");
    logger.debug("debug");
    logger.error("error");
}
```



### 6.分页

为什么：

* 减少数据处理量



#### 1.limit

```
select * from user limit startIndex,pageSize;
```

pageSize = -1表示从当前位置一直查到结尾（bug被修复了）



mybatis实现：

```java
//    List<User> getBylimit(Map<String, Integer> map);
List<User> getBylimit(@Param("index") int index, @Param("size") int size);
```

```xml
<!--    分页查询-->
<select id="getBylimit" resultMap="Users">
    select *
    from mybatis.user
    limit #{index},#{size};
</select>
```

```java
@Test
public void limit() {
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    Userdao mapper = sqlSession.getMapper(Userdao.class);

    //        Map<String, Integer> map = new HashMap<>();
    //        map.put("index", 1);
    //        map.put("size", 3);

    List<User> listUser = mapper.getBylimit(1, 3);

    for (User user : listUser) {
        System.out.println(user);
    }
    sqlSession.close();
}
```



#### 2.RowBounds分页(了解)

```xml
<select id="getByRowBounds" resultMap="Users">
    select *
    from mybatis.user
</select>
```

```java
@Test
public void getByRowBounds() {
    SqlSession sqlSession = MybatisUtil.getSqlSession();

    RowBounds rowBounds = new RowBounds(1, 2);


    List<User> selectList = sqlSession.selectList("com.rst.dao.Userdao.getByRowBounds", null, rowBounds);
    for (User user : selectList) {
        System.out.println(user);
    }

    sqlSession.close();
}
```



#### 3.分页插件

Mybatis PageHelper



### 7.使用注解开发

#### 1.面向接口编程

* 解耦，可拓展，提高复用性，分层开发，开发更加规范化
* 面向接口的理解：
  * 定义与实现的分离
  * 接口本身反映系统设计人员对系统的抽象理解
* 两类接口：抽象体和抽象面
* 三个面向：
  * 面向对象，以对象为单位，考虑其方法与属性
  * 面向过程，考虑具体流程和实现
  * 接口技术针对复用技术，更多体现的时对系统整体的架构



#### 2.注解开发

本质：反射 底层：动态代理

```java
@Select("select id,name,pwd as password from user")
List<User> getListUser();
```

```xml
<mappers>
    <mapper class="com.rst.dao.Userdao"/>
</mappers>
```

注意：使用注解来映射简单语句会使代码显得更加简洁，但对于稍微复杂一点的语句，Java 注解不仅力不从心，还会让本就复杂的 SQL 语句更加混乱不堪。 因此，如果你需要做一些很复杂的操作，最好用 XML 来映射语句。



### 8.mybatis执行流程

1. Resource获取配置文件
2. 实例化SqlSessionFactoryBuilder
3. 解析流文件（xml），Configuration对象存储配置信息
4. SqlSessionFactory实例化
5. Transaction事务管理器
6. 创建执行器executor，创建sqlSession
7. 执行CRUD
8. 是否成功（是否混滚事务管理器）
9. 提交
10. 关闭sqlSession

### 9.注解CRUD

#### 1.自动提交

```Java
sqlSessionFactory.openSession(true)
```

#### 2.注解参数

* 基本类型需要加上
* 引用类型不需要
* #{}和${}的区别

```Java
List<User> getBylimit(@Param("index") int index, @Param("size") int size);
```

```xml
<select id="getBylimit" resultMap="Users">
    select *
    from mybatis.user
    limit #{index},#{size};
</select>
```

#### 3.insert

```Java
@Insert("insert into user (id,name,pwd) values (#{id},#{name},#{password})")
int insert(User user);
```



### 10.Lombok

Project Lombok is a ==java library== that automatically ==plugs== into your editor and ==build tools==, spicing up your java.==Never write another getter or equals method again==, with one ==annotation== your class has a fully featured builder, Automate your logging variables, and much more.
————————————————
版权声明：本文为CSDN博主「ThinkWon」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/ThinkWon/article/details/101392808



 使用方法:

1. 安装插件
2. 导入jar包
3. 在实体类上加注解即可（字段、类）



注解：

```
@Getter and @Setter
@FieldNameConstants
@ToString
@EqualsAndHashCode
@AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor
@Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog
@Data
@Builder
@SuperBuilder
@Singular
@Delegate
@Value
@Accessors
@Wither
@With
@SneakyThrows
@val
@var
```

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private int id;
    private String name;
    private String pwd;
}
```



### 11.多对一处理

**关联**：多对一     子查询 连表查询

**集合**：一对多

```
<association 对象
<collection 集合
```

```mysql
CONSTRAINT 约束名 FOREIGN KEY (外键名)
REFERENCES 关联表名(主键名)
```



#### 环境搭建过程中产生的问题：

1. ```
   org.apache.ibatis.reflection.ReflectionException: Error instantiating class com.rst.pojo.Student with invalid types (int,String,Teacher) or values (1,小米,1). Cause: java.lang.IllegalArgumentException: argument type mismatch
   ```

   pojo类使用lombok注解需要加上全参和无参构造

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   ```

2. LOG4J报错

   clean以下项目，并重新编译
   
   

#### 思路1：查询嵌套处理

```xml
    <resultMap id="studentteacher" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
    </resultMap>


    <select id="getAllStudent" resultMap="studentteacher">
        select *
        from mybatis.student;
    </select>

    <select id="getTeacher" resultType="Teacher">
        select *
        from mybatis.teacher
        where id = #{id};
    </select>
```

```
Student(id=1, name=小米, teacher=Teacher(id=1, name=rst))
Student(id=2, name=小红, teacher=Teacher(id=1, name=rst))
Student(id=3, name=小李, teacher=Teacher(id=1, name=rst))
Student(id=4, name=小苏, teacher=Teacher(id=1, name=rst))
```



#### 思路2：结果嵌套处理

```xml
    <select id="getAllStudent2" resultMap="studentteacher2">
        select s.id sid, s.name sname, t.name tname
        from mybatis.teacher t,
             mybatis.student s
        where t.id = s.tid;
    </select>

    <resultMap id="studentteacher2" type="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <association property="teacher" javaType="Teacher">
            <result property="name" column="tname"/>
        </association>
    </resultMap>
```

```
Student(id=1, name=小米, teacher=Teacher(id=0, name=rst))
Student(id=2, name=小红, teacher=Teacher(id=0, name=rst))
Student(id=3, name=小李, teacher=Teacher(id=0, name=rst))
Student(id=4, name=小苏, teacher=Teacher(id=0, name=rst))
```



### 12.一对多

#### 1.一个老师拥有多个学生

```
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Teacher {
    int id;
    String name;
    List<Student> students;
}
```



#### 2.按结果嵌套查询

```xml
<select id="getTeacher" resultMap="teacherstu">
    select t.id teacherid, t.name tname, s.id sid, s.name sname
    from mybatis.student s,
    mybatis.teacher t
    where t.id = s.tid
    and t.id = #{Tid}
</select>

<resultMap id="teacherstu" type="Teacher">
    <result property="id" column="teacherid"/>
    <result property="name" column="tname"/>
    <collection property="students" ofType="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <result property="tid" column="teacherid"/>
    </collection>
</resultMap>
```

```java
Teacher getTeacher(@Param("Tid") int id);
```



#### 3.嵌套子查询

```xml
    <select id="getTeacher2" resultMap="getTeacherstu2">
        select *
        from mybatis.teacher
        where id = #{Tid};
    </select>

    <resultMap id="getTeacherstu2" type="Teacher">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <collection property="students" column="id" javaType="ArrayList" ofType="Student" select="getStudentsByid"/>
    </resultMap>
    <select id="getStudentsByid" resultType="Student">
        select *
        from mybatis.student
        where tid = #{tid};
    </select>
```

*  javaType 实体类属性类型
* ofType 指定映射到集合中的pojo类型



注意：

1. 保证sql的可读性
2. 注意字段名和属性名的匹配问题
3. 多用日志排除错误



==mysql引擎==

==innodb底层原理==

==索引==

==索引优化==



### 13.动态sql

**根据不同条件生成不同sql语句**

在sql层面添加逻辑代码，本质还是sql语句

```
使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性。

如果你之前用过 JSTL 或任何基于类 XML 语言的文本处理器，你对动态 SQL 元素可能会感觉似曾相识。在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。

if
choose (when, otherwise)
trim (where, set)
foreach
```



#### 1.环境搭建

```SQL
create table `blog`
(
    `id`          varchar(20) not null comment '博客id',
    `title`       varchar(20) not null,
    `author`      varchar(20) not null,
    `create_time` datetime    not null,
    `views`       int         not null
) engine = innodb
  default charset = utf8;

insert into blog (id, title, author, create_time, views)
VALUES ('rst123', 'blog', 'rst', '2023-04-07', 666);
```



1. 导包
2. 编写配置
3. 编写实体类
4. 编写Mapper接口和Mapper.xml文件

```
<setting name="mapUnderscoreToCamelCase " value="true"/>      驼峰命名转换
```



UUID：

```
public class IDUtil {
    public static String getId() {
        return UUID.randomUUID().toString().replaceAll("-", "");
    }
}
```



#### 2.if

接口

```java
List<Blog> getBlogByif(Map map);
```

xml

```xml
<select id="getBlogByif" parameterType="map" resultType="Blog">
    select * from mybatis.blog where 1=1
    <if test="title != null">
        and title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</select>
```

测试

```
@Test
public void getBlogByif() {
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);

    HashMap<String, String> map = new HashMap<>();
    map.put("title", "rstzz");


    List<Blog> blogList = mapper.getBlogByif(map);

    for (Blog blog : blogList) {
        System.out.println(blog);
    }

    sqlSession.close();
}
```



#### 3.choose(when,otherwise)

```xml
<select id="findActiveBlogLike"
     resultType="Blog">
  SELECT * FROM BLOG WHERE state = ‘ACTIVE’
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```

* when中有满足条件走第一个满足的语句，当when都不满足时走otherwise中的语句



#### 4.trim(where,set)

```xml
    <select id="getBlogByChoose" parameterType="map" resultType="Blog">
        select *
        from mybatis.blog
        <where>
            <choose>
                <when test="title !=null">
                    and title = #{title}
                </when>
                <otherwise>
                    and views > 700
                </otherwise>
            </choose>
        </where>
    </select>
```

* *where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。



```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```

* *set* 元素会动态地在行首插入 SET 关键字，并会删掉额外的逗号



```xml
<trim prefix="" suffix="" prefixOverrides="" suffixOverrides=""></trim>
```

* 前后缀，前后缀覆盖



#### 5.sql片段

便于复用sql语句



抽取公共部分代码

```
<sql id="ifTitle">
    <if test="title != null">
        and title = #{title}
    </if>
</sql>
```

在需要的地方include

```
<select id="getBlogByif" parameterType="map" resultType="Blog">
    select * from mybatis.blog where 1=1
    <include refid="ifTitle"></include>
    <if test="author != null">
        and author = #{author}
    </if>
</select>
```



注意：

1. 基于单表设置sql片段，尽量简单
2. 不要使用where等标签



#### 6.foreach

动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在构建 IN 条件语句的时候）。



```xml
    <select id="getBlogByForeach" parameterType="map" resultType="Blog">
        select *
        from mybatis.blog
        <where>
            <foreach collection="ids" item="id" open="and (" close=")" separator="or">
                id = #{id}
            </foreach>
        </where>
    </select>
```



### 14.缓存

#### 1.简介

1. 什么是缓存
   * 存储在内存中的临时数据
   * 用户经常使用的数据存储在内存中，当用户查询时就不再访问硬盘，而是直接访问缓存，从而提高查询效率，解决高并发系统性能问题
2. 为什么使用缓存
   * 减少和数据库的交互次数，减少系统开销
3. 什么样的数据库可以使用缓存
   * 经常查询而不经常改变的数据



#### 2.mybatis缓存

* mybatis包含非常强大的查询缓存特性，它可以非常方便地制定和配置缓存。
* mybatis默认定义了两级缓存：一级和二级
  * 默认开启一级缓存（sqlsession级别，本地缓存）
  * 二级缓存需要手动配置和开启，是基于namespace的缓存
  * 为提高可扩展性，mybatis提供了缓存接口Cache，我们可以通过该接口实现自定义二级缓存

#### 3.一级缓存

```java
public void insert() {
    SqlSession sqlSession = MybatisUtil.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    User user1 = mapper.getUserById(4);
    System.out.println(user1);
    System.out.println("===============================");

    User user2 = mapper.getUserById(4);
    System.out.println(user2);

    System.out.println(user1 == user2);


    sqlSession.close();
}
```

运行结果，sql语句只执行一次

```
==>  Preparing: select * from mybatis.user where id = ?
==> Parameters: 4(Integer)
<==    Columns: id, name, pwd
<==        Row: 4, xxx, 564
<==      Total: 1
User(id=4, name=xxx, pwd=564)
===============================
User(id=4, name=xxx, pwd=564)
true
```



缓存失效：

- 查询不同语句。
- **映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。**
- 查询不同mapper。
- 手动清除缓存   `sqlSession.clearCache();`



注：一级缓存默认生效，只在一个sqlsession开启和关闭之间起作用。



#### 4.二级缓存

工作机制：

* 不同会话查询的数据放在自己的一级缓存中
* ==当会话关闭，一级缓存中的数据会转存到二级缓存中==
* 不同Mapper查询的数据会放到对应的缓存中（map），即二级缓存只在同一个Mapper下有效



步骤：

1. 开启全局缓存

   ```
   <setting name="cacheEnabled" value="true"/>
   ```

2. 在要使用的Mapper中开启二级缓存

   ```
   <cache
           eviction="FIFO"
           flushInterval="60000"
           size="512"
           readOnly="true"/>
   ```

3. 测试

   ```
   @Test
   public void insert2() {
       SqlSession sqlSession = MybatisUtil.getSqlSession();
       UserMapper mapper = sqlSession.getMapper(UserMapper.class);
   
       SqlSession sqlSession2 = MybatisUtil.getSqlSession();
       UserMapper mapper2 = sqlSession2.getMapper(UserMapper.class);
   
       User user1 = mapper.getUserById(4);
       System.out.println(user1);
   
       sqlSession.close();
   
       User user2 = mapper2.getUserById(4);
       System.out.println(user2);
   
       sqlSession2.close();
   }
   ```

测试结果:

```
Created connection 1677568775.
Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@63fdab07]
==>  Preparing: select * from mybatis.user where id = ?
==> Parameters: 4(Integer)
<==    Columns: id, name, pwd
<==        Row: 4, xxx, 564
<==      Total: 1
User(id=4, name=xxx, pwd=564)
Resetting autocommit to true on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@63fdab07]
Closing JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@63fdab07]
Returned connection 1677568775 to pool.
Cache Hit Ratio [com.rst.dao.UserMapper]: 0.5
User(id=4, name=xxx, pwd=564)
```



问题：实体类需要要序列化

**程序运行的时候，会产生很多对象，而对象信息也只是在程序运行的时候才在内存中保持其状态，一旦程序停止，内存释放，对象也就不存在了。  使用序列化可以把对象永久缓存下来**



#### 5.缓存原理

1. 先看二级缓存
2. 二级缓存没有，看一级缓存
3. 一级缓存没有，再连接数据库
4. 会话关闭，一级缓存内容转存到二级缓存中



#### 6.自定义缓存

EhCache 是一个纯Java的进程内缓存[框架](https://so.csdn.net/so/search?q=框架&spm=1001.2101.3001.7020)，具有快速、精干等特点。EhCache支持单机缓存和分布式缓存，分布式可以理解为缓存数据的共享，这就导致内存缓存数据量偏小。ehcache缓存存储和读取非常快。



要在程序中使用，先导包

```
<!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.2.3</version>
</dependency>
```



在mapper中指定缓存类

```
 <cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```



ehcache.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="https://ehcache.org/ehcache.xsd"
         updateCheck="false">

    <diskStore path="./tmpdir/Tmp_EhCache"/>

    <defaultCache
            eternal="false"
            maxElementsInMemory="10000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="259200"
            memoryStoreEvictionPolicy="LRU"/>

    <cache
            name="cloud_user"
            eternal="false"
            maxElementsInMemory="5000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="1800"
            memoryStoreEvictionPolicy="LRU"/>
</ehcache>
```

