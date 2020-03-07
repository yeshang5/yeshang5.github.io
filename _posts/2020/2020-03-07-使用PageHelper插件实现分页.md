---
layout: post
title: '使用PageHelper插件实现分页'
date: 2020-03-07
author: 白皓
cover: '/assets/img/little-boy.jpg'
tags: 框架 分页   
---

## 使用PageHelper插件实现分页

### 一、引入PageHelper

#### Springboot项目

添加Maven依赖
```xml
    <!--PageHelper物理分页插件-->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper-spring-boot-starter</artifactId>
        <version>1.2.13</version>
    </dependency>
```

在application.yml中添加配置
```yaml
#数据库分页插件
pagehelper:
  helper-dialect: mysql  #使用哪种数据库语言
  reasonable: true      #配置分页参数合理化功能，默认是false。 #启用合理化时，如果pageNum<1会查询第一页，如果pageNum>总页数会查询最后一页； #禁用合理化时，如果pageNum<1或pageNum>总页数会返回空数据。
  support-methods-arguments: true  #支持通过Mapper接口参数来传递分页参数，默认值false，分页插件会从查询方法的参数值中，自动根据上面 params 配置的字段中取值，查找到合适的值时就会自动分页。
```

#### SSM项目

*添加Maven依赖*
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.1.2</version>
</dependency>
```

*配置拦截器插件*

拦截器的配置方式有两种，配置时任选一种方式即可
>   1. 在 MyBatis 配置 xml 中配置拦截器插件
```xml
<!-- 
    plugins在配置文件中的位置必须符合要求，否则会报错，顺序如下:
    properties?, settings?, 
    typeAliases?, typeHandlers?, 
    objectFactory?,objectWrapperFactory?, 
    plugins?, 
    environments?, databaseIdProvider?, mappers?
-->
<plugins>
    <!-- com.github.pagehelper为PageHelper类所在包名 -->
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <!-- 使用下面的方式配置参数，后面会有所有的参数介绍 -->
        <property name="param1" value="value1"/>
	</plugin>
</plugins>
```

>   2. 在 Spring 配置文件中配置拦截器插件
使用 spring 的属性配置方式，可以使用 plugins 属性像下面这样配置：
```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <!-- 注意其他配置 -->
  <property name="plugins">
    <array>
      <bean class="com.github.pagehelper.PageInterceptor">
        <property name="properties">
          <!--使用下面的方式配置参数，一行配置一个 -->
          <value>
            params=value1
          </value>
        </property>
      </bean>
    </array>
  </property>
</bean>
```

配置参数说明：

    `helperDialect`：分页插件会自动检测当前的数据库链接，自动选择合适的分页方式。 你可以配置helperDialect属性来指定分页插件使用哪种方言。配置时，可以使用下面的缩写值：oracle,mysql,mariadb,sqlite,hsqldb,postgresql,db2,sqlserver,informix,h2,sqlserver2012,derby
    
    `offsetAsPageNum`：默认值为 false，该参数对使用 RowBounds 作为分页参数时有效。 当该参数设置为 true 时，会将 RowBounds 中的 offset 参数当成 pageNum 使用，可以用页码和页面大小两个参数进行分页。
    
    `rowBoundsWithCount`：默认值为false，该参数对使用 RowBounds 作为分页参数时有效。 当该参数设置为true时，使用 RowBounds 分页会进行 count 查询。
    
    `pageSizeZero`：默认值为 false，当该参数设置为 true 时，如果 pageSize=0 或者 RowBounds.limit = 0 就会查询出全部的结果（相当于没有执行分页查询，但是返回结果仍然是 Page 类型）。
    
    `reasonable`：分页合理化参数，默认值为false。当该参数设置为 true 时，pageNum<=0 时会查询第一页， pageNum>pages（超过总数时），会查询最后一页。默认false 时，直接根据参数进行查询。
    
    `params`：为了支持startPage(Object params)方法，增加了该参数来配置参数映射，用于从对象中根据属性名取值， 可以配置 pageNum,pageSize,count,pageSizeZero,reasonable，不配置映射的用默认值， 默认值为pageNum=pageNum;pageSize=pageSize;count=countSql;reasonable=reasonable;pageSizeZero=pageSizeZero。
    
    `supportMethodsArguments`：支持通过 Mapper 接口参数来传递分页参数，默认值false，分页插件会从查询方法的参数值中，自动根据上面 params 配置的字段中取值，查找到合适的值时就会自动分页。 使用方法可以参考测试代码中的 com.github.pagehelper.test.basic 包下的 ArgumentsMapTest 和 ArgumentsObjTest。
    
    `autoRuntimeDialect`：默认值为 false。设置为 true 时，允许在运行时根据多数据源自动识别对应方言的分页 （不支持自动选择sqlserver2012，只能使用sqlserver），用法和注意事项参考下面的场景五。
    
    `closeConn`：默认值为 true。当使用运行时动态数据源或没有设置 helperDialect 属性自动获取数据库类型时，会自动获取一个数据库连接， 通过该属性来设置是否关闭获取的这个连接，默认true关闭，设置为 false 后，不会关闭获取的连接，这个参数的设置要根据自己选择的数据源来决定。

### 二、编辑Controller
```java
   import com.github.pagehelper.PageInfo;@RequestMapping("/user")
   @RestController
   public class UserController extends BaseController {
   
       @Autowired
       private UserService userService;
 
       @GetMapping("data")
       public PageInfo data(User user, HttpServletRequest request, HttpServletResponse response)
       {
           List<User> userList=new ArrayList<User>();
           PageInfo pageInfo=userService.findPage(user,request,response);
           return pageInfo;
       }
   }
```

### 三、编辑Service
```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public PageInfo<User> findPage(User user,HttpServletRequest request, HttpServletResponse  response){
        String pageNumStr= request.getParameter("pageNum");
        String pageSizeStr= request.getParameter("pageSize");
        Integer pageNum=StrUtil.isNotBlank(pageNumStr)?Integer.valueOf(pageNumStr):1;
        Integer pageSize=StrUtil.isNotBlank(pageSizeStr)?Integer.valueOf(pageSizeStr):10;

        //默认分页显示10条数据,当传参pageSize=0时，不分页
        PageHelper.startPage(pageNum,pageSize);
        List<User> userList=userMapper.findList(user);
        PageInfo<User> pageInfo=new PageInfo<User>(userList);
        return pageInfo;
    }
}
```

### 四、编辑Mapper
```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    public List<User> findList(User user);
}
```

### 五、编辑Mapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.baihao.modules.system.mapper.UserMapper">

    <sql id="Columns">
        a.id AS "id",
        a.name AS "name",
        a.password AS "password",
        a.phone AS "phone"
    </sql>

    <select id="findList" resultType="com.baihao.modules.system.entity.User" >
        SELECT
        <include refid="Columns"/>
        FROM sys_user a
        <where>
            <if test="name != null and name != ''">
                AND a.name = #{name}
            </if>
            <if test="phone != null and phone != ''">
                AND a.phone = #{phone}
            </if>
        </where>
    </select>
</mapper>
```

#### 六、测试
访问localhost:8080/user/data?pageNum=1&pageSize=10  即可看到分页效果
![](https://s2.ax1x.com/2020/03/07/3XyOu4.png)



