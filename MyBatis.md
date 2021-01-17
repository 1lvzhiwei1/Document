# MyBatis

**半自动、轻量级的框架**

**sql与java编码分离，sql是开发人员编写**

==下载地址==

**https://github.com/mybatis/mybatis-3**

==配置流程==

* 根据全局配置文件（配置数据源等信息）创建一个SqlSessionFactory对象
* 创建sql映射文件：mapper，编写业务sql语句
* 将sql映射文件注册在全局配置文件中
* 编写逻辑代码：
  * 根据全局配置文件得到SqlSessionFactory
  * 使用SqlSessionFactory获取到sqlsession对象**sqlsession代表和一次和数据库的会话，最后需要close**
  * 使用mapper中的sql的唯一标识来执行sql语句

## 接口式编程

* **Mapper  ====>  Mapper.xml**

* **SqlSession代表和数据库的一次会话，每次操作完需要close**

* **SqlSession和connection一样，都是非线程安全的，每次使用都应该获取新的对象**
* **Mapper接口没有写实现类，MyBatis会生成一个代理对象，通过代理对象去执行操作**

==两个重要的配置文件==

* mybatis的全局配置文件：包含数据库连接池信息、事务管理器信息等系统运行环境信息
* sql映射文件Mapper：保存sql的映射信息

## 全局配置文件标签

==properties==

**使用<properties\>标签来引入外部properties配置文件的内容**

==settings==

**setting：用来设置每一个设置项**

* name：设置项名
* value：设置项取值

==typeAliases==

**别名处理器：可以为java类型设置别名，<package\>标签批量设置别名**

* type：类型的全类名
* alias：指定新的别名

==environments==

**<environments\>标签配置各种系统运行环境，default指定使用某种环境，可以快速切换**

**<environment\>标签配置一个具体的环境，id代表当前环境的唯一标识,<environment\>标签中必须具有以下两个标签：**

* <transactionManager\>：事务管理器
* <dataSource\>：数据源

==databaseIdProvider==

**支持多数据库厂商**

**属性type="DB_VENDOR"：得到数据库厂商的标识，mybatis根据数据库厂商标识执行不同的sql语句**

**通过sql语句标签中的databaseId属性设置不同的数据库厂商**

==mappers==

**将sql映射注册到全局配置中**

* resource：引用类路径下的sql映射
* url：引用网络路径或者磁盘下的sql映射文件

* class：引用接口，映射sql文件名必须和接口同名，并且放在与接口同一目录下

**sql映射直接写在注解中（@Select、@Update...)**

## Sql映射文件Mapper

==增删改查==

**增删改可以定义返回值类型为INTEGER、LONG、BOOLEAN**

* <insert/\>
* <delete/\>
* <update/\>
* \<select/>

**sqlSessionFactory.openSession();  ====> 手动commit**

**sqlSessionFactory.openSession(true);  ====> 自动commit**

==自增主键==

**useGeneratedKeys="true"：使用自增主键获取主键值策略**

**keyProperty：指定对应的主键属性，将主键值赋值给这个属性**

### 参数处理

==单个参数==

**mybatis不会做参数处理，#{xxx}取出参数值**

==多个参数==

**mybatis会做特殊处理，多个参数会被封装成一个map**

* key：[param1~paramN]或者索引下标
* value：传入的参数值

**#{xxx}就是从map值获取指定的key的值**

==命名参数==

**明确指定封装参数时map的key**

**定义方法参数时添加注解@Param("paramname")**

==POJO==

**如果多个参数一一对应pojo的属性，就可以直接传入pojo**

**#{属性名}：取出传入的pojo属性值**

==Map==

**多个参数没有对应的pojo，可以封装成Map传入**

**#{key}：取出map中对应的值**

==传入参数Collection==

**mybatis做特殊处理，把传入的List或者Array封装在Map中**

* List
  * key：list
* Array
  * key：array

==参数值的获取==

* **#{}：是以预编译的形式，将参数设置到sql语句中，防止sql注入，只能取sql语句参数的值**
* **${}：取出的值直接拼装到sql语句中，会有安全问题**

**原生jdbc不支持占位符的地方我们可以使用${}进行取值，例如分表、排序**

**select * from ${tablename}_table where id = #{id}**

==Null==

**mybatis中null对应的jdbcType.OTHER，Oracle数据库无法识别**

* 全局配置文件中设置Null对应jdbcType.NULL：

  ```xml
  <settings>
  	<setting name="jdbcTypeForNull" value="NULL"></setting>
  </settings>
  ```

==resultMap==

**自定义结果集映射规则**

```xml
<resultMap type="返回类的全路径" id="xxx">
    <id column="id" property="id"></id>
	<result column="name"  property="name"></result>
</resultMap>
```

* **id定义主键列**
  * column指定哪一列
  * property指定对应得javaBean属性
* **result定义普通列**

==association==

**定义联合的单个javaBean对象**

* property：指定哪个属性是联合的对象
* JavaType：指定联合对象的类型（全路径）

**分布查询**

* select：表明当前属性是调用select指定的方法查出的结果
* column：指定将哪一列的值传给这个方法

==懒加载==

```xml
<settings>
	<setting name="lazyLoadingEnabled" value="true"/>
    <setting name="aggressivelazyLoading" value="false"/>
</settings>
```

==关联集合==

**在<resultMap\>中使用<collection\>标签定义关联集合类型的属性的封装规则**

* property：关联集合对应的属性名
* ofType：指定关联集合中的元素类型（全路径）

==鉴别器==

**mybatis可以使用discriminator判断某列的值，然后根据某列的值改变封装规则**

* column：指定判断的列名
* JavaType：列值对应得java类型

## 动态SQL

**使用OGNL表达式**

### If

**如果带了某个条件，则查询的时候加上这个条件**

```xml
<where>  
	<if test="id != null">
        <!--逻辑语句-->
        id=#{id}
    </if>
    <if test="name != null">
        and name=#{name}
    </if>
</where>
```

**查询的时候如果某些条件没带，可能导致sql拼装出错**

* 在where后面加上1=1，后面的条件都写成and xxx
* 把所有的查询条件写在<where\>标签中，**只能去除前面的and和or**

### Trim

**trim标签体中是整个字符串拼接后的结果**

* prefix：给字符串加一个前缀
* prefixOverrides：前缀覆盖，去掉字符串前面多余的字符
* suffix：给字符串加一个后缀
* suffixOverrides：后缀覆盖，去掉字符串后面多余的字符

```xml
<where>
<trim prefix="xxx" prefixOverrides="xxx" suffix="xxx" suffixOverrides="xxx">
   <if test="id != null">
		<!--逻辑语句-->
    	id=#{id}
	</if>
	<if test="name != null">
		and name=#{name}
	</if>         
</trim>
</where>
```

### Choose

**如果带了条件1，就用条件1查；如果带了条件2，就用条件2查 **	==只会有一个查询条件==

```xml
<where>
	<choose>
    	<when test="id != null">
        	id=#{id}
        </when>
     	<when test="name != null">					name=#{name}
        </when>
        <otherwise>
        	1=1
        </otherwise>
    </choose>
</where>
```

* when：判断该条件
* otherwise：所有条件都不成立，则执行

### Foreach

**遍历集合**

* collection：指定要遍历的集合，**list类型的参数会特殊处理封装在map中，key就是list**
* item：将遍历出的元素赋值给指定变量，**#{变量名}就能取出变量的值**
* separator：每个元素之间的分隔符
* open：遍历出所有结果拼接一个开始的字符
* close：遍历出所有结果拼接一个结束的字符
* index：索引
  * 遍历list的时候，是索引
  * 遍历map的时候，是map的key

```xml
<select>
	select * from xxx where id in
    <foreach collection="ids" item="item_id" separator="," open="(" close=")">
    	#{item_id}
    </foreach>
</select>
```

## 内置参数

==_parameter==

**代表整个参数**

* 单个参数：_parameter就是这个参数
* 多个参数：参数会被封装为一个map，_parameter就代表这个map

==_databaseId==

**如果配置databaseIdProvider标签，_databaseId就是代表当前数据库的别名**

==bind==

**将OGNL表达式的值绑定到一个变量中，方便后面引用**

==sql==

**抽取可重用的sql片段，方便后面引用**

* sql抽取：经常讲要查询的列名，或者插入列名抽取出来方便调用
* include：引用已经抽取的sql

```xml
<sql id="selectColumn">
	id,name,gender
</sql>

<inlude refid="selectColumn"></inlude>
```

## 缓存机制

==查出的数据默认放到一级缓存中，只有会话提交或者关闭后，一级缓存中的数据才会转移到二级缓存==

### 一级缓存（本地缓存）

**sqlSession级别的一个Map**

**与数据库同一次会话期间查询到的数据会放在本地缓存中，如果以后需要获取相同的数据，直接从缓存中拿，不需要再去数据库查询**

==一级缓存失效==

* sqlSession不同
* sqlSession相同，查询条件不同（数据还未存入一级缓存中）
* sqlSession相同，两次查询之间执行了增删改操作
* sqlSession相同，手动清除了一级缓存（缓存清空clearCache( )）

### 二级缓存（全局缓存）

**namespace级别的一个Map**

==工作机制==

* 一个会话，查询一条数据，这个数据就会被放在当前会话的一级缓存中
* 如果会话关闭，一级缓存中的数据会被保存到二级缓存中
* 新的会话可以到二级缓存中查询数据
* 不同的namespace查出的数据会放在自己对应的二级缓存中

==使用==

* 开启全局二级缓存配置：

  ```xml
  <settings>
  	<setting name="cacheEnabled" value="true"></setting>
  </settings>
  ```

* mapper.xml中配置使用二级缓存

  ```xml
  <cache></cache>
  ```

* POJO需要实现序列化接口

==缓存相关的属性==

* cacheEnabled=false：关闭二级缓存
* <select\>标签中的属性useCache=“false”,关闭二级缓存
* 增删改标签的属性flushCache=“true”,操作执行完会清除一级缓存和二级缓存
* sqlSession.clearCache()：清除一级缓存
* localCacheScope：本地缓存作用域

### 缓存原理

==缓存顺序：二级缓存 —> 一级缓存 —> 数据库==

![image-20210116205659753](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210116205659753.png)

## 逆向工程

**MBG，为MyBatis框架使用者定制的代码生成器，可以快速的根据表生成对应得映射文件、接口以及pojo类**

**https://github.com/mybatis/generator/releases**

### 标签解释

==jdbcConnection==

**指定如何连接到目标数据库**

==javaModelGenerator==

**指定JavaBean的生成策略**

* targetPackage：生成的目标包名
* targetProject：生成的目标工程

==sqlMapGenerator==

**sql映射生成策略**

* targetPackage：生成的目标包名
* targetProject：生成的目标工程

==javaClientGenerator==

**指定mapper接口所在的位置**

==table==

**指定要逆向分析哪些表**

* domainObjectName：生成的bean名

## 工作原理

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210117195019860.png" alt="image-20210117195019860" style="zoom: 67%;" />

1. 获取sqlSessionFactory对象
2. 获取sqlSession对象
3. 获取接口的代理对象（MapperProxy）
4. 执行增删改查方法

### 创建sqlSessionFactory

**Configuration对象封装了所有配置文件的详细信息**

**一个MappedStatement封装了一个增删改查标签的信息**

==总结==

**把配置文件的信息解析并封装在Configuration对象中，返回包含Configuration的DefaultSqlSessionFactory对象**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210117202119495.png" alt="image-20210117202119495" style="zoom:90%;" />

### 创建sqlSession

**返回sqlSession的实现类DefaultSqlSession对象，封装了Configuration和Executor**

==会创建一个Executor对象==

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210117210028270.png" alt="image-20210117210028270" style="zoom:80%;" />

### 创建接口的代理对象

==mapper接口的代理对象==

**包含了DeFaultSqlSession对象**

![image-20210117211619098](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210117211619098.png)

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210117211523637.png" alt="image-20210117211523637" style="zoom:90%;" />

### 增删改查流程

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210117214750950.png" alt="image-20210117214750950" style="zoom:110%;" />

==总结==

1. 调用DefaultSqlSession的增删改查（Executor）
2. 创建一个StatementHandler对象（同时也会创建ParameterHandler和ResultSetHandler）
3. 调用StatementHandler预编译参数以及设置参数值，使用ParameterHandler来给sql设置参数
4. 调用StatementHandler的增删改查方法
5. ResultSetHandler封装结果

==注意==

**四大对象每一个创建的时候都会通过拦截器配置插件interceptorChain.pluginAll(ParameterHandler)**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210117215622475.png" alt="image-20210117215622475" style="zoom:70%;" />

## 插件开发

### 插件原理

**四大对象(Executor、StatementHandler、ParameterHandler、ResultSetHandler)创建的时候：**

1. 每个创建出来的对象不是直接返回的，而是interceptorChain.pluginAll(ParameterHandler)
2. 获取到所有的Interceptor(拦截器)；调用interceptor.plugin(target)，返回target包装后的对象
3. 可以使用插件为目标对象创建一个代理对象：AOP(面向切面)

### 插件编写

#### 编写Interceptor的实现类

* intercept：拦截目标对象的目标方法的执行
* plugin：包装目标对象，为目标对象创建代理对象  **可以使用Plugin的wrap(object,interceptor)来包装目标对象**
* setProperties：将插件注册时的property属性设置进来

#### 插件签名

**使用@Intercepts注解，告诉mybatis拦截哪个对象的哪个方法**

**@Signature注解的是一个拦截对象，需要设置三个属性**

* type：拦截对象
* method：拦截对象的方法
* args：拦截方法的参数

```java
@Intercepts(
    {
     	@Signature(type,method,args)   
    })
```

#### 将插件注册到全局配置文件中

**插件的property属性会通过setProperties()加载进拦截器**

```xml
<plugins>
	<plugin interceptor="拦截器的全类名">
    <prpperty name="xxx" value="xxx"/>
    <prpperty name="xxx" value="xxx"/>
    </plugin>
</plugins>
```

### 多个插件运行

**多个插件就会产生多层代理**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210118095906026.png" alt="image-20210118095906026" style="zoom:70%;" />

**创建动态代理的时候，是按照插件配置顺序创建层层代理对象**

**执行目标方法的时候，则是按照插件配置顺序的逆序执行（先执行最外层代理对象的方法）**

![image-20210118095702534](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210118095702534.png)

## 批量操作

**设置ExecutorType为BATCH**

## 存储过程

**使用<select\>标签定义调用存储过程**

**statementType="CALLABLE",表示调用存储过程**

==mode==

* IN：输入参数
* OUT：输出参数

```xml
<select id="useProcedure" statementType="CALLABLE">
	{call procedurename(
    	#{param1,mode=IN,jdbcType=INTEGER}
    	#{param2,mode=OUT,jdbcType=INTEGER}
    )}
</select>
```

