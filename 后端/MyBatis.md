# MyBatis

半自动orm框架。sql与java编码分离，并由开发人员控制。

## 基础

### 使用方法

1. 根据xml配置文件（全局配置文件）创建一个SqlSessionFactory对象，有数据源的一些运行环境信息
2. sql映射文件：配置了每一个sql，以及sql的封装规则等
3. 将sql映射文件注册在全局配置文件中
4. 写代码
   1. 根据全局配置文件得到SqlSessionFactory
   2. 使用sqlSession工厂，获取sqlSession对象，使用他来执行增删改查；一个sqlSession就是代表和数据库的一次会话，用完关闭
   3. 使用sql的唯一标识来告诉MyBatis执行哪个sql，sql都是保存在sql映射文件中的

### 特点

1. Mapper --> xxMapper.xml
2. SqlSession代表和数据库的一次会话，用完必须关闭
3. SQLSession和connection都是非线程安全的，每次使用都应该获取新对象
4. mapper接口没有实现类，但是mybatis会为这个接口生成一个代理对象，将接口和xml进行绑定
5. 两个重要的配置文件
   1. mybatis全局配置文件，包含连接池信息、事务管理器等系统运行环境信息
   2. sql映射文件，保存了sql语句的映射信息

## 全局配置文件

### properties

引入外部properties配置文件内容

resource：引入类路径下资源

url：引入网络路径或者磁盘路径下的资源

### settings

运行时行为设置，包含很多重要的设置项

```xml
<setting name="xxx" value="xxx"/> 
<!-- 
name：设置名称
value：设置值
-->
```

### typeAliases

别名处理器，可以为java类型起别名。别名不区分大小写

```xml
<typeAlias type="xxxx" alias="xxx"/>
<!--
单个起别名
type：类型全类名，默认别名是类名小写
alias：指定新的别名
-->
<package name="xxx"/>
<!--
包下所有类批量起别名
name：指定包名，为包下所有类起一个默认别名
-->
<!-- 批量起别名的情况下，使用@Alias注解为某个类指定新的别名 -->
```

### typeHandlers

类型处理器，java类型与数据库类型进行映射

### plugins

插件，丰富MyBatis功能

### environments

可以配置多种环境

```xml
<environments default="xxx">
<environment id="xxx">
	<transactionManager type="xx"></transactionManager>
	<dataSource type="xx"></dataSource>
</environment>
</environments>
<!--
一般使用spring的事务管理器和数据源
default指定当前使用的环境
environment配置一个具体的环境信息，id代表当前环境的唯一标识
必须有两个标签
transactionManager：事务管理器
dataSource：数据源
-->
```

### databaseldProvider

支持数据库厂商，为不同的厂商起别名

### mappers

将sql映射注册到全局配置中

```xml
<mapper resource="xxx"></mapper>
<mapper url="xxx"></mapper>
<mapper class="xxx"></mapper>
<!--
resource：引用类路径下的sql映射文件
url：引用网络路径或者磁盘路径下的sql映射文件
class：填写全类名，引用（注册）接口
	1. 有sql映射文件，映射文件名必须和接口同名，并且放在与接口同一目录下
	2. 没有sql映射文件，所有sql都是利用注解写在接口上
推荐重要的Dao接口写sql映射文件，其他的可以使用注解能快速开发
-->
<package name="xxx"/>
<!--
批量注册，但需要接口文件和映射文件同名，且在同一目录下
-->
```

## 映射文件

```java
sqlSessionFactory.openSession(); // 手动提交
sqlSessionFactory.openSession(true); // 自动提交
```

### 参数处理

1. 单个参数：#{参数名}

2. 多个参数：多个参数会被封装成一个map，

   #{paramN}是从map中获取指定的key值

   map格式：

   * key：param1...paramN，或者参数索引
   * value：传入的参数值

3. 命名参数：使用@Param("xxx")注解，指定封装参数时map的key

   map格式：

   * key：@Param指定的key
   * value：传入的参数值

   #{指定的key}取出传入的参数值

4. 传入POJO或者map或者TO(Transfer Object)

**注意：**如果是我Collection（List、Set）类型或者数组也会特殊处理，也就是把传入的list或者数组封装在map中。

| type       | key        |
| ---------- | ---------- |
| Collection | collection |
| List       | list       |
| 数组       | array      |

#### 参数封装

1. 获取每个标了param注解的参数的@Param值，赋值给name

2. 每次解析一个参数给map中保存信息（key，参数索引，value，name的值）

   > name的值：
   >
   > * 标注了param注解，name=注解的值
   > * 没标注param注解：
   >   1. 全局配置useActualParamName（jdk1.8），name=参数名
   >   2. name=map.size()，相当于元素的索引

**ParamNameResolver.getNameParams(Object[] args)**

> 1. 参数为null直接返回
> 2. 如果只有一个元素且没有@Param注解，单个参数直接返回
> 3. 多个元素或者有@Param注解，会遍历names集合，names集合的value作为key，args的值作为value重新封装为map

#### #{}和${}区别

* #{}：以预编译的形式，将参数设置到sql语句中，相当于占位符?
* ${}：取出的值直接拼装在sql语句中，会有安全问题
* 大多数情况下使用#{}，传统jdbc不支持占位符的地方可以使用${}进行取值，如分表、排序

#### #{}更多用法

规定参数的一些规则：javaType、jdbcType、mode(存储过程)、numericScale、resultMap、typeHandler、jdbcTypeName、expression(未来准备支持的功能)

jdbcType通常在某种特定条件下被设置，如数据为null时，有些数据库不能识别mybatis对**null**的默认处理为Jdbc的**OTHER**类型，**Oracle**就不能识别。

> 全局配置中jdbcTypeForNull=NULL也可避免上述问题

### insert

1. 获取自增主键的值

   > useGeneratedKeys="true"  // 打开使用自增主键
   >
   > keyProperty="xxx" // 自增主键的字段

2. 获取非自增主键的值

   > <selectKey keyProperty="字段" resultType="type”>sql语句</selectKey>
   >
   > // keyProperty：表示将查出的主键值封装给javabean的字段
   >
   > // resultType：表示返回值类型

### select

1. 返回List

   如果返回的是一个集合，resultType要写集合中元素的类型

2. 返回map

   * 单条数据封装map

     resultType="map"，key是列名，value是对应值

   * 多条数据封装map

     resultType="元素类型"，key是方法注解@MapKey("xx")，value是对应元素集合

3. 自定义结果映射

   属性resultMap，需要自定义某个JavaBean的封装规则

   ```xml
   <resultMap type="类名" id="xxx">
   <!-- 指定主键列的封装规则
   column：指定哪一列
   property：指定对应的JavaBean属性
   -->
       <id column="xx" property="xx"/>
   <!-- 定义普通列封装规则 -->
       <result column="xx" property="xx"/>
   <!-- 其他不指定的列会自动封装，建议只要写resultMap就把全部的映射规则都写上 -->
       ...
   </resultMap>
   ```

#### 关联查询

1. 级联属性封装结果

   利用resultMap

2. association定义关联对象封装规则

   ```xml
   <!-- 指定联合的JavaBean对象
   property：指定那个属性是联合的对象
   javaType：指定这个属性对象的类型（不能省略）
   -->
   <association property="xx" javaType="类名">
   	<id column="xx" property="xx"/>
       <result column="xx" property="xx"/>
   </association>
   ```

3. association分步查询

   ```xml
   <!-- 使用select指定的方法（传入column指定的这列参数的值）查出对象，并封装给property指定的属性
   property：指定那个属性是联合的对象
   select：指定这个属性对象是由哪个查询方法查出的
   column：指定将哪一列的值传给这个方法
   -->
   <association property="xx" select="查询方法" column="xx">
   </association>
   ```

   延迟加载

   ```xml
   <!-- 在全局配置文件中配置设置属性 -->
   <setting name="lazyLoadingEnabled" value="true"/>
   <setting name="aggressiveLazyLoading" value="false"/>
   ```

4. collection定义关联集合封装规则

   ```xml
   <collection poperty="xx" ofType="元素类型">
   	<id column="xx" property="xx"/>
       <result column="xx" property="xx"/>
   </collection>
   ```

5. collection分布查询

   ```xml
   <!-- 使用select指定的方法（传入column指定的这列参数的值）查出对象，并封装给property指定的属性
   property：指定那个属性是联合的对象
   select：指定这个属性对象是由哪个查询方法查出的
   column：指定将哪一列的值传给这个方法
   -->
   <collection poperty="xx" select="查询方法" column="xx">
   </collection>
   ```

6. 分布查询多列值传递

   封装成map：column="{key1=column1, key2=column2...}"

   标签fetchType="lazy"：表示使用延迟加载

7. discriminator鉴别器

   可以使用discriminator判断某列的值，然后根据某列的值改变封装规则

   ```xml
   <discriminator javaType="类型" column="xx">
   	<case value="aa" resultType="类型"></case>
       <case value="bb" resultType="类型"></case>
   </discriminator>
   ```

## 动态sql

### if

```xml
<!-- test：判断OGNL表达式 -->
<if test=""></if>
```

> 1. 使用<if>标签可能使sql拼装错误，可以使用<where>标签，会去掉第一个多的and或or
>    * <where>：封装查询条件
>    * <set>：封装修改条件

### choose

```xml
<!-- 分支查询 -->
<choose>
	<when test="">
    查询条件
    </when>
    ...
    <otherwise>
    默认查询条件
    </otherwise>
</choose>
```

### trim

```xml
<!-- 字符串截取 
使用<trim>标签
prefix=""：给拼接后的整个字符串加一个前缀
prefixOverrides=""：前缀覆盖，去掉整个字符串前面多余的字符
suffix=""：拼接后的整个字符串加一个后缀
suffixOverrides=""：去掉整个字符串后面多余的字符
-->
<trim prefix="" prefixOverrides="" suffix="" suffixOverrides=""></trim>
```

### foreach

```xml
<!-- 遍历
collection：指定要遍历的集合
item：将当前遍历出的元素赋值给指定的变量
separator：拼接的分隔符
open：遍历出的所有结果拼接一个开始的字符
close：遍历出的所有结果拼接一个结束的字符
index：索引。
	遍历list时index是索引，item是当前值
	遍历map时表示map的key，item是map的值
#{}：取出变量的值
-->
<foreach collection="xx" item="xxx" separator="" open="" close="">
	#{xxx}
</foreach>
```

### 批量插入

1. MySQL下

   MySQL支持`insert into...values (), ()...`，使用`foreach`即可

   ```xml
   <insert id="xxx">
   	insert into ...
   	values
       <foreach collection="xx" item="xxx" separator="," >
       	(col1, col2...)
       </foreach>
   </insert>
   ```

2. Oracle下

   Oracle不支持`values`后跟多个括号的形式，所以有如下两种方式进行批量插入

   ```xml
   <!-- 第一种方式：使用多条insert，放在begin-end内 -->
   <insert id="xxx">
   	begin
       <foreach collection="xx" item="xxx">
       insert into ... values (val1, val2...);
       </foreach>
   	end
   </insert>
   
   <!-- 第二种方式：使用临时表，将插入数据union成一张表 -->
   <insert id="xxx">
       insert into ... select col1, col2 ... from(
       <foreach collection="xx" item="xxx" separator="union">
           select val1, val2...
       </foreach>
       )
   </insert>
   ```

### 内置参数

1. _parameter：代表整个参数

   单个参数时：_parameter就是这个参数

   多个参数时：参数会被封装为一个map，_parameter代表这个map

2. _databaseId：如果配置了databaseIdProvider标签，\_databaseId代表当前数据库的别名

### bind绑定

可以将OGNL表达式的值绑定到一个变量中，方便后来引用这个变量的值

```xml
<bind name="xxx" value="OGNL表达式"/>
```

### sql标签

抽取可重用的sql**片段**，方便后面引用

```xml
<!-- 对可重用的sql片段进行定义 -->
<sql id="xxx">sql片段</sql>
<!-- 在需要引用的地方引用外部定义的sql -->
<!-- property自定义属性，可以在sql标签中引用，取值方法为${}，不能使用#{} -->
<include refid="xxx">
	<property name="xx" value="xxxx"/>
</include>
```

## 缓存机制

默认定义了两级缓存

1. 默认情况下，只开启一级缓存（SqlSession级别的缓存，本地缓存）开启
2. 手动开启二级缓存，基于namaespace级别的缓存
3. 可以使用缓存接口Cache接口自定义二级缓存

### 一级缓存（本地缓存）

SqlSession级别的缓存，是一直开启的。实质上是SqlSession的一个Map

与数据库同义词会话期间查询到的数据会放在本地缓存中。

以后如果需要获取相同的数据，直接从缓存中拿，没必要再查询数据库。

**失效情况：**

1. SqlSession不同
2. SqlSession相同，但是查询条件不同（当前一级缓存中还没有该数据）
3. SqlSession相同，但两次查询期间执行增删改（因为这次增删改可能对缓存数据有影响）
4. SqlSession相同，但手动清除了，`clearcache()`

### 二级缓存（全局缓存）

基于namespace级别的缓存，一个namespace对应一个二级缓存

**工作机制：**

1. 一个会话，查询一条数据，这个数据会被放在当前会话的一级缓存中
2. 如果会话关闭，一级缓存中的数据会被保存到二级缓存，新的会话查询信息就可以参照二级缓存，不同的namespace查出的数据会放在自己对应的缓存中（map）

**使用方法：**

1. 开启全局二级缓存配置

   ```xml
   <setting name="cacheEnabled" value="true"/>
   ```

2. 去mapper.xml配置使用二级缓存

   ```xml
   <!--
   eviction：缓存回收策略
   	LRU，默认
   	FIFO
   	SOFT软引用，移除基于垃圾回收器状态和软引用规则的对象
   	WEAK弱引用，更积极地移除基于垃圾回收期状态和弱引用规则的对象
   flushInterval：缓存刷新间隔，即多长时间清空一次，默认不清空
   readOnly：
   	true：只读，认为所有从缓存中取数据的操作都是只读操作，不修改数据。直接将引用交给用户，不安全但速度快
   	false：非只读，默认，认为获取数据可能会被修改。利用序列化&反序列化技术克隆一份新数据，安全但速度慢
   type：自定义缓存的全类名
   -->
   <cache eviction="" flushInterval="" readOnly="" size="" type=""></cache>
   ```

3. POJO需要实现序列化接口

   ```java
   class POJO implements Serializable {}
   ```

### 相关设置

> 1. cacheEnabled=false：关闭的是二级缓存，一级缓存一直可用
>
> 2. select标签中的useCache="false"，关闭的是二级缓存，一级缓存一直可用
>
> 3. 增删改标签默认的flushCache="true"，一级二级缓存都清空
>
>    查询标签中默认flushCache="false"
>
> 4. clearCache()：只清除当前session的一级缓存
>
> 5. localCacheScope：本地缓存作用域
>
>    默认SESSION：当前会话的所有数据保存在会话缓存中
>
>    STATEMENT：可以禁用一级缓存

缓存的查找顺序：二级 -> 一级 -> 数据库

### 第三方缓存整合

1. 导入第三方缓存包

2. 带入与第三方缓存整合的适配包

3. mapper.xml中使用自定义缓存

   ```xml
   <!--  引用缓存，namespace：指定和哪个命名空间下的缓存一样 -->
   <cache-ref namespace=""/>
   ```

## 工作原理

### 创建SqlSessionFactory

把配置文件的信息解析并保存在Configuration对象中，返回包含了Configuration的DefaultSqlSession对象

1. sqlSessionFactoryBuilder().build(配置xml文件)
2. 解析配置xml文件
3. 解析每一个标签，把详细信息保存在Configuration
4. 解析mapper.xml文件
5. 返回封装了所有配置信息的Configuration
6. build(Configuration)
7. 返回创建的DefaultSqlSession()，包含了保存全局配置信息的Configuration

### 获取SqlSession对象

返回SqlSession的实现类DefaultSqlSession，包含Configuration和Executor，这一步创建了Executor

1. 调用openSession() -> openSessionFromDataSource()

2. 获取一些信息，创建tx

3. 创建Executor对象

4. 根据Executor在全局配置中的类型，创建出对应的Executor

5. 如果有二级缓存开启，创建CachingExecutor(executor)

6. 使用拦截器重新包装executor并返回

   `executor=(Executor)interceptorChain.pluginAll(executor)`

7. 创建DefaultSqlSession对象，包含Configuration和Executor并返回

### 获取mapper对象

getMapper()，使用MapperProxyFactory创建一个MapperProxy的代理对象，包含了DefaultSqlSession(Executor)对象

1. sqlSession.getMapper(type) -> mapperRegister.getMapper(type, sqlSession) 
2. 根据接口类型获取MapperProxyFactory
3. newInstance(sqlSession)
4. 创建MapperProxy，这是一个InvocationHandler
5. 创建MapperProxy代理对象并返回

### 执行

1. invoke() -> execute()

2. 判断增删改查类型

3. 参数封装

4. 调用SqlSession对应的方法，再调用executor的对应方法

5. 获取BoundSql，代表sql语句的详细信息

6. 如果是查询语句，缓存开启的话先到缓存中查找

7. 创建StatementHandler对象

8. 使用拦截器重新包装StatementHandler并返回，默认为PrepareStatementHandler

   `statementHandler=(statementHandler)interceptorChain.pluginAll(statementHandler)`

9. 使用拦截器重新包装ParameterHandler并返回

   `parameterHandler=(ParameterHandler)interceptorChain.pluginAll(parameterHandler)`

10. 使用拦截器重新包装ResultSetHandler并返回

    `resultSetHandler=(ResultSetHandler)interceptorChain.pluginAll(resultSetHandler)`

11. 再创建相应的Statement对象，默认为PrepareStatement

12. 调用TypeHandler给sql预编译设置参数

13. 使用resultSetHandler处理结果，使用TypeHandler获取value的值

14. 后续的连接关闭，并返回执行结果

### 总结

1. 根据配置文件（全局，sql映射）初始化出Configuration对象

2. 创建一个DefaultSqlSession对象

   里面包含了Configuration以及Executor（根据全局配置文件中的defaultExecutorType创建出对应的Executor）

3. DefaultSqlSession.getMapper()，那倒Mapper接口对应的MapperProxy

4. MapperProxy里面有DefaultSqlSession

5. 执行增删改查方法

   * 调用DefaultSqlSession的增删改查
   * 会创建一个StatementHandler对象，也会同时创建出ParameterHandler和ResultSetHandler
   * 调用StatementHandler预编译参数以及设置参数值
   * 使用ParameterHandler给sql设置参数
   * 调用StatementHandler的增删改查方法
   * ResultSetHandler封装结果

   > 四大对象每个创建的时候都会有interceptorChain.pluginAll()方法

## 插件原理

在四大对象创建的时候，

1. 每个创建出来的对象不是直接返回的，二是interceptorChain.pluginAll()

2. 获取到所有的Interceptor（拦截器）（插件需要实现的接口）

   调用interceptor.plugin(target)，返回target包装后的对象

3. 插件机制，使用插件为目标对象创建一个代理对象：AOP

   自定义插件可以为四大对象创建出代理对象，代理对象可以拦截到四大对象的每一个执行

### 编写

1. 实现Interceptor接口`implement Interceptor`
   1. intercept()：拦截目标对象的目标方法的执行
   2. plugin()：包装目标对象，即为目标对象创建一个代理对象，调用`wrap()`
   3. setProperties()：将插件注册时的property设置进来

2. 签名

   使用`@Intercepts`注解
   
   告诉MyBatis当前插件用来拦截哪个对象的哪个方法
   
   ```java
   @Intercepts({
       @Signature(type=拦截对象类名 method="方法名" args=参数类名)
   })
   ```
   
3. 将插件注册到全局配置文件中

   ```xml
   <plugin interceptor="xxx">
   	<property name="" value=""/>
   </plugin>
   ```

### 多个插件

创建动态代理时，按照插件配置顺序创建多层代理对象，最后配置的在最外层；

执行目标方法时，是按照逆序执行