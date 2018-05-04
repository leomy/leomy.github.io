---
title: 使用MyBatis
date: 2018-03-24 03:44:03
categories:
- mybatis
tags: [Java, SQL, MyBatis, Usage]
toc: true
---

记录MyBatis的用法如配置文件的内容、使用接口代理开发、动态SQL的各种标签等。详情参考<a href="http://www.mybatis.org/mybatis-3/zh/index.html">官方中文文档</a>

## 接口代理开发
- 开发流程：Mapper接口开发方法只需要编写Mapper接口(相当于以前的dao接口)，由MyBatis框架根据接口定义创建接口的动态代理对象，代理对象的方法体调用dao接口实现类的方法。
- Mapper接口开发需要遵循以下规范:
    - `mapper.xml`文件中的`namespace`与 接口的 类路径相同
    - `Mapper`接口方法的输入参数类型和`mapper.xml`中定义的每个sql的`parameterType` 的类型相同
    - `Mapper`接口方法的输入参数类型和`mapper.xml`中定义的每个sql的`parameterType`的类型相同
    - `Mapper`接口方法的输出参数类型和`mapper.xml`中定义的每个sql的`resultType`的类型相同
    
## 配置文件
- `mybatis-config.xml`配置内容(有顺序，如下列出的顺序书写)
    ```
    1. properties 属性
    2. settings 设置
    3. typeAliases 类型别名
    4. typeHandlers 类型处理器
    5. objectFactory 对象工厂
    6. plugins 插件
    7. environments 环境
            environment 环境变量
                transactionManger 事务管理器
                dataSource 数据源
    8. databaseldProvider 数据库厂商标志符
    9. mappers 映射器
    ```
- `properties`引入外部配置文件
    -  `<properties resource="classpath:jdbc.properties"/>`
-  `settings`配置mybatis全局参数：如开启缓存、延迟加载等全局开关
    ```xml
    <!-- name： 全局开关属性名 -->
    <!-- value：全局开关属性值 -->
    <settings>
        <!-- 开启缓存，只是全局开关，还需要在mapper.xml下使用 <cache/> 标签开启 -->
        <setting name="cacheEnable" value="true"/>
        
        <!-- 开启延迟加载，sql语句的分步查询(一条SQL语句改写成多条SQL，通过association关联) -->
        <setting name="lazyLoadingEnable" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
        
        <!-- 使用日志，日志加载顺序方式如下:
                SLF4J
                Apache Commons Logging
                Log4j 2
                Log4j
                JDK logging 
        -->
        <!-- 启用 log4j2 日志 -->
        <setting name="logImpl" value="LOG4J2"/>  
        
        <!-- 启用驼峰映射 -->
        <setting name="mapUnderscoreToCamelCase" value="true" />  
    </settings>
    ```
- `typeAliases`定义别名
    ```xml
    <typeAliases>
    
        <!-- 
            typeAlias 定义单个别名，每次只能定义一个
            type 指定实体路径
            alias 别名
        -->
        <typeAlias type="com.leo.mybatis.domain.User" alias="user"/>
        
        <!-- package批量扫描包定义别名，实体类名为大小写都可以。扫描多个路径时使用逗号(,)分割 -->
        <package  name="com.leo.mybatis.vo,com.leo.mybatis.pojo"/>
    </typeAliases>
    ```
- `typeHandlers`通常mybatis的 `typeHandler`完成jdbc类型和Java类型的转换
- `mappers`引入mapper.xml映射文件
    ```xml
    <mappers>
        <!-- 一个一个文件的配置 -->
        <mapper resource="sqlMap/user.xml"/>
        
        <!-- 批量扫描接口 -->
        <package name="com。leo.mybatis.mapper"/>
    </mappers>
    ```
- MyBatis缓存(总共有两级缓存)
    - Mybatis执行查询sql时，首先去缓存中查找，如果命中直接返回，没有命中执行sql从数据库中查询
    - 一级缓存的作用域是`Session`，当`openSession()`后，如果执行相同的sql语句，Mybatis不会执行sql语句，而是从缓存中命中返回。当执行`insert/update/delete`会刷新缓存，并且一级缓存不能禁用但可以清空使用`sqlsession.clearCache()`
    - 二级缓存的作用域为`mapper`的`namespace`，同一个`namespace`执行的sql会缓存在二级缓存中，须要满足缓存对象能序列化(`Serializable`)。二级缓存默认是关闭的，需要手动开启
        - 开启方法(在全局配置文件中)

            ```xml
            <settings>
          
            <!-- 开启缓存-->
            <setting name="cacheEnabled" value="true"/> 
            
            <!-- 开启延迟加载 -->
            <setting name="lazyLoadingEnabled" value="true"/>
            
            <!-- 开启按需加载 -->
            <setting name="aggressiveLazyLoading" value="false"/>
            
            </settings>
            ```
        - 在mapper配置文件中要开启
            
            ```xml
            <cache/>
            ```
## 动态SQL
### 主键增长
`useGeneratedKeys`实现方式是调用JDBC中Statement的getGeneratedKeys()
- MySQL
    - 使用自动回写
        - `useGeneratedKeys`表示主键自增后自动回写id
        - `keyColumn`表示数据库表中的字段
        - `keyProperty`表示实体类中的属性
        ```xml
        <insert id="save" useGeneratedKeys="true" keyColumn="id" keyProperty="id" parameterType="user">
            insert into user(username,phonm) values(#{username},#{phone})
        </insert>
        ```
    - 使用MySQL函数`LAST_INSERT_ID()`
        - `selectKey`实现自增主键返回
        - `keyProperty`返回值对应pojo里面的属性
        - `order`表示id生成的顺序,由于MYSQL主键生成是在sql语句执行之后再进行设置,所以设置成after
        - `resultType`主键返回类型
        - `LAST_INSERT_ID()`是MYSQL函数,返回`auto_increment`自增id值
        ```xml
        <insert id="save" parameterType="user">
            <selectKey keyProperty="id" order="after" resultType="int">
                select LAST_INSERT_ID()
                insert into user(id,username,phone) values(#{id},#{username},#{phone})
            </selectKey>
        </insert>
        ```
    - 使用`uuid`实现主键自增
        MySQL的`uuid`生成必须在执行`insert` 之前进行，因为后面需要获取`uuid`的属性值进行表的插入
        ```xml
        <insert id="save" parameterType="user">
            <selectKey keyProperty="id" order="before" resultType="string">
                select uuid()
        insert insto user(id,username) values(#{id},#{username})
            </selectKey>
        </insert>
        ```
- Oracle
    Oracle自增主键是序列化自增类型，需要`before`
    ```xml
    <insert id="save" parameterType="user">
        <selectKey keyProperty="id" order="befor" resultType="int">
            select 序列.nextVal from dual
        </selectKey>
        insert into user(id,username) values(#{id},#{username})
    </insert>
    ```
### `where`
对应SQL语句中的`where`条件。MyBatis会去掉最后一个`and` 
```xml
<!-- 动态添加条件 -->
<select id="findUser" parameterType="user" resultType="user">
    select * from user
    <where>
        <!-- id精确查找 -->
        <if test="id != null">
            and id=#{id}
        </if>
        <!-- 用户名模糊查找 -->
        <if test="name != null and name != ''">
            and name like '%${name}%'
        </if>
    </where>
</select>
```
### `resultMap`
`resultMap`可以完成sql查询出来的类名与实体之间的复杂映射
```xml
<!-- autoMapping="true" 开启自动映射(当表的列和类的属性相同时不用写) -->
<resultMap id="resultMapPerson"  type="com.leo.mybatis.User" autoMapping="true">
    <!-- 
        id 代表主键
        column sql查询字段名(可以是别名)
        property 实体类型名
    -->
    <id column="id" property="id"/>
    
    <!-- 普通类型字段映射 -->
    <result column="username" property="username">                
</resultMap>
<select id="findUserById" parameterType="int" rersultMap="resultMapPerson">
    select id,username,birthday from user where id=#{id}
</select>
```
### `include`
将重复的sql提取出来，使用`include`引用即可，最终达到sql重用的目的(引用其他名称空间的sql片段，需要在引入时加上名称空间)
```xml
<sql id="userColumn">
    id,username,birthday
</sql>
<select id="findById" parameterType="int" resultType="user">
    select <include refid="userColumn"/> from user where id=#{id}
</select>
```
### `foreach`
向SQL传递数组或List
```xml
<!-- 第一种方法、在user对象定义一个 List<Integer> ids,并生成get/set。在 foreach 标签中 collection 表示类中的ids集合。最终生成的sql语句包含 id in(....) -->
<select id="findUser" parameterType="user" resultType="user">
    select * from user 
    <where>
        <if test="ids != null and ids.size > 0">
            <foreach collection="ids" item="id" open="id in(" close=")" separator=",">
                #{id} 
            </foreach>
        </if>
    </where>
</select>

<!-- 第二种方法、直接传递 List 对象，批量删除时有用。 此时的parameterType必须为java.util.List，if的test和foreach的collections必须为list -->
<select id="findByIds" parameterType="java.util.List" resultType="user">
    select * from user
    <where>
        <if test="list != null and list.size > 0">
            and id in
            <foreach collection="list" item="id" open="(" close=")" separator=",">
                #{id}
            </foreach>
        </if>
    </where>
</select>
```
### `set`
更新操作动态选择条件。MyBatis回去掉最后一个`,`
```xml
<!-- set 标签自动将 set 最后一个删掉 -->
<update id="updateUser" parameterType="user" resultType="user">
    update user 
    <set>
        <if test="name != null and name != ''">
            name=#{name},
        </if>
        <if test="phone != null and phone != ''">
            phone=#{phone}
        </if>
    </set>
    where id=#{id}
</update>
```
### `chooes`
`chooes when otherwise` 当某个条件满足则加上该条件，否则加上另外的条件
```xml
<select id="findUser" parameterType="user" resultType="user">
    select * from user
    <where>
        <chooes>
            <when test="id != null">
                and id=#{id}
            </when>
            <otherwise>
                name='%${name}%'
            </otherwise>
        </chooes>
    </where>
</select>
```
### 关联关系
- `association`一对一关联关系
    ```xml
    <!-- (新建类)添加OrdersCustomer类，继承Orders，并添加User类型成员变量 -->
    <!-- 查询所有的订单及其相关的下单人 -->
    <resultMap id="findAllMapper" type="ordersCustomer">
        <!-- orders表和Orders类映射。column 数据库表中查出来的字段(有别名,则以别名为准)，property类中的属性 -->
        <id column="id" property="id"/>
        <result column="number" property="number"/>
        
        <!-- 关联的下单用户映射。association表示关联查询的单条记录。type 表示数据库中的表。javaType表示类名 -->
        <association type="user" javaType="user">
            <id column="id" property="id"/>
            <result column="username" property="username"/>
        </association>
    </resultMap>

    <select id="findAll" resultMap="findAllMapper">
        select orders.*,user.id as userid,orders.* from user,orders where orders.user_id = user.id
        </select>
    ```
- `collection` 一对多查询
    ```xml
    <!-- 通过Id查找用户及相应的订单 -->
    <resultMap id="findUserAndOrdersByUserIdMapper" type="user">
        <id column="id" property="id"/>
        <id column="username" property="username"/>
        <id column="birthday" property="birthday"/>
        <id column="sex" property="sex"/>
        <id column="address" property="address"/>

        <collection property="orders" ofType="order">
            <id column="ordersid" property="id"/>
            <id column="number" property="number"/>
            <id column="note" property="note"/>
            <id column="createtime" property="createtime"/>
        </collection>
    </resultMap>
    <select id="findUserAndOrdersByUserId" parameterType="int" resultMap="findUserAndOrdersByUserIdMapper">
        select user.*,orders.id as ordersid, orders.* from user,orders where user.id=orders.user_id and user.id=#{id};
    </select>
    ```
