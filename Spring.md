# Spring5

## IoC

**管理所有的轻量级JavaBean组件，提供的底层服务包括组件的生命周期管理、配置和组装服务、AOP支持，以及建立在AOP基础上的声明式事务服务等**

### IOC原理

**控制反转**

**IoC负责组件的创建和配置，应用程序直接使用已经创建并配置好的组件，配置组件需要依赖注入机制**

==注入的好处==

* 应用程序不再关心怎样创建组件
* 组件共享非常简单
* 测试更容易，可以使用内存数据库，不用查MySQL的配置

==依赖注入的方式==

* 属性注入，通过set方法实现

  ```java
  public class BookService {
      private DataSource dataSource;
  
      public void setDataSource(DataSource dataSource) {
          this.dataSource = dataSource;
      }
  }
  ```

* 构造方法注入，通过带参的构造方法实现

  ```java
  public class BookService {
      private DataSource dataSource;
  
      public BookService(DataSource dataSource) {
          this.dataSource = dataSource;
      }
  }
  ```

==非侵入容器==

**应用程序的组件无需实现Spring的特定接口**

* 应用程序组件既可以在Spring的IoC容器中运行，也可以自己编写代码自行组装配置
* 测试的时候并不依赖Spring容器，可单独进行测试

==IOC原理==

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210111212835725.png" alt="image-20210111212835725" style="zoom:80%;" />

**xml解析、反射、工厂模式：解耦**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210111211924507.png" alt="image-20210111211924507" style="zoom:80%;" />

### IoC接口（BeanFactory）

**Spring提供两种方式实现IOC容器**

* BeanFactory：Spring内置方式  **\*加载配置文件的时候不会创建对象，在获取对象的时候才创建**
* ApplicationContext：BeanFactory的子接口，功能更多更强大，一般由开发人员进行使用  **\*在加载配置文件的时候就会创建对象**

==ApplicationContext接口有实现类==

* FileSystemXmlApplicationContext
* ClassPathXmlApplicationContext

### Bean管理

**Bean管理指的是两个操作：创建对象、注入属性**

**Bean管理操作有两种方式：**

* 基于xml配置文件方式实现
* 基于注解方式实现

#### xml

**创建对象**

1. 在spring xml配置文件中，使用bean标签，标签中添加对应属性，就可以实现对象创建
2. bean标签属性：
   * id：bean的唯一标识
   * class：类全路径
3. 创建对象的时候，默认是执行无参构造函数方法进行创建

##### **注入属性**

* **DI：依赖注入**

  **set方法注入：在<bean\>标签中使用<property\>标签进行属性注入**

  **有参构造方法注入：在<bean\>标签中使用<constructor-arg\>标签进行属性注入**

  **p名称空间注入：**

* 添加p名称空间在配置文件中：xmlns:p="http://www.springframework.org/schema/p"

* 在bean标签中进行属性注入：<bean id="User" class="com.spring.User" p:age="abc"/\>

**注入null值：**

```xml
<property name="xxx">
	<null/>
</property>
```

**注入特殊符号属性：**

* 转义

* 使用CDATA：

  ```xml
  <property name="xxx">
  	<value><![CDATA[特殊符号属性]]></value>
  </property>
  ```

**注入外部bean**

* 在<property\>标签中使用ref定义需要注入的beanid

  ```xml
  <bean id="userService" class="com.spring.service.UserService">
          <property name="userDao" ref="userDao"/>
  </bean>
  <bean id="userDao" class="com.spring.dao.UserDaoImpl"/>
  ```

**注入内部bean**

```xml
<bean id="xxx" class="xxx">
	<property>
    	<bean id="xxx" class="xxx">
        	<property name="xxx" value="xxx"/>
        </bean>
    </property>
</bean>
```

**注入集合属性**

* 数组(array)、列表(list)、集合(set)

  ```xml
  <property name="xxx">
  	<array>
      	<value>xxx</value>
          <value>xxx</value>
          <value>xxx</value>
      </array>
  </property>
  ```

* Map

  ```xml
  <property name="xxx">
  	<map>
      	<entry key="xxx" value="xxx"></entry>
      </map>
  </property>
  ```

**注入对象集合属性**

```xml
<bean id="id1" class="xxx"/>
<bean id="id2" class="xxx"/>
<bean id="xxx" class="xxx">
	<property>
    	<list>
        	<ref bean="id1"></ref>
            <ref bean="id2"></ref>
        </list>
    </property>
</bean>
```

**提取集合注入**

1. 先创建util空间
2. 修改xsi：将所有beans替换成util
3. 使用<util\>标签创建可公共使用的集合bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       <!--创建util空间-->
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
							<!--修改xsi值-->
						   http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">

    <util:list id="List">
		<value>xxx</value>
        <value>xxx</value>
        <value>xxx</value>
	</util:list>

	<!--使用list-->
	<bean id="xxx" class="xxx">
		<property name="list" ref="List"></property>
	</bean>
</beans>
```

##### BeanFactory

**工厂bean特点：配置文件中定义的bean类型可以和返回的bean不同**

1. 创建类，让这个类作为工厂bean，实现接口FactoryBean
2. 实现接口中的方法，在方法中定义返回的bean类型

##### bean作用域

**Spring中默认创建的是单例bean**

==设置单例或多例==

**<bean\>标签中设置属性scope的值**

* 单例：singleton，在加载配置文件的时候创建对象
* 多例：prototype，在调用getBean()方法的时候创建对象

##### bean生命周期

==bean的后置处理器==

1. 通过构造器创建bean实例（无参构造方法）
2. 为bean注入属性（调用set方法）
3. **把bean的实例传给bean的后置处理器的方法**：==postProcessBeforeInitialization==
4. 调用bean的初始化方法（配置初始化方法：设置<bean\>标签的init-method属性为自定义的初始化方法）
5. **把bean的实例传给bean的后置处理器的方法**：==postProcessAfterInitialization==
6. 使用bean（获取到对象）
7. 容器关闭的时候，执行bean的销毁（配置销毁的方法：设置<bean\>标签的destory-method属性为自定义的销毁方法）**销毁需要手动调用ClassPathXmlApplicationContext对象的close()方法**

==配置bean的后置处理器==

**在配置文件中为实现BeanPostProcessor的类创建一个bean**

##### xml自动装配

**根据指定装配规则（属性名称或者属性类型，Spring自动将匹配的属性值进行注入**

**通过设置<bean\>标签的autowire属性配置自动装配**

==autowire==

* byName：根据属性名称注入，需要注入的beanid和属性名称相同
* byType：根据属性类型注入

##### 外部属性文件

==手动配置数据库连接池==

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210112202549068.png" alt="image-20210112202549068" style="zoom:80%;" />

==创建外部属性文件，properties格式文件==

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210112204147356.png" alt="image-20210112204147356" style="zoom:80%;" />

* 引入context名称空间
* 在Spring配置文件中使用<context\>标签引入外部属性文件

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210112205001750.png" alt="image-20210112205001750" style="zoom:80%;" />

#### 基于注解方式

* 注解是代码特殊标记，格式：@注解名称(属性名称=属性值，属性名称=属性值)
* 使用注解，注解作用在类上面，方法上面，属性上面
* 使用注解目的：简化xml配置

==针对创建对象的注解，都可以创建bean实例==

* @Component
* @Service
* @Controller
* @Repository

==基于注解方式实现对象创建==

1. 引入依赖：spring-aop
2. 开启组件扫描
   * 引入context命名空间
   * spring配置文件中通过<conetxt:component-scan base-package="xxx"/>开启组件扫描
3. 创建类，在类上面添加注解

==注解创建的bean名称默认采用小驼峰法==

==配置自动扫描过滤器==

**默认过滤器是扫描所有内容**

**配置扫描注解**

* use-default-filters="false" 不使用默认的过滤器
* include-filter：设置扫描哪些内容

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210113090236087.png" alt="image-20210113090236087" style="zoom:80%;" />

**配置扫描忽略注解**

* 使用默认过滤器
* exclude-filter：设置扫描忽略哪些注解

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210113090414562.png" alt="image-20210113090414562" style="zoom:80%;" />

##### 注解方式实现注入属性

* @Autowired：根据属性类型进行注入
* @Qualifier：根据属性名称进行注入
* @Resource：可以根据属性类型注入，也可以根据属性名称注入
* @Value：注入普通类型属性

##### 纯注解开发

* 创建配置类，替代xml配置文件：**使用@Configuration注解标明配置类，使用@ComponentScan注解配置扫描**

* 加载配置类：

  ```java
  ApplicationContext context = new AnnotationConfigApplicationContext(配置类.class)
  ```

## AOP

**面向切面编程，利用AOP可以对业务逻辑的各个部分进行隔离，降低各部分之间的耦合度，提高程序的可重用性，提高开发的效率**

**通俗来说：不通过修改源代码，在主干中添加新功能**

### 动态代理

==有接口情况，使用JDK动态代理==

**创建接口实现类的的代理对象，通过代理对象增强类的方法**

* 调用newProxyInstance方法
  * 参数1：类加载器
  * 参数2：增强方法类实现的接口
  * 参数3：InvocationHandler接口，创建代理对象，写增强方法

==无接口情况，使用CGLIB动态代理==

**创建当前类的子类的代理对象，通过代理对象增强类的方法**

### 术语

==连接点==

**类里面所有的可以被增强的方法**

==切入点==

**实际被增强的方法**

==通知（增强）==

* **增强的逻辑部分**
* **多种类型：**
  * 前置通知：@Before
  * 后置通知：@AfterReturning
  * 环绕通知：@Around
  * 异常通知：@AfterThrowing
  * 最终通知（finally）：@After

==切面==

**是一个动作，把通知应用到切入点的过程**

### AOP操作

**Spring框架一般都是基于AspectJ实现AOP操作**

**AspectJ不是Spring的组成部分，是独立的AOP框架**

两种方式：

* 基于xml配置文件实现
* 基于注解方式实现（常用）

#### 基于注解方式

==增强类==

**添加@Component注解，生成bean实例**

**添加@Aspect注解，生成代理对象**

==切入点表达式==

* 作用：知道对哪个类里面的哪个方法进行增强
* 语法结构：execution([权限修饰符\][返回类型\][类全路径\][方法名称\]([参数列表\]))

==重用切入点==

**创建一个方法，在方法上添加@Pointcut注解，value为切入点表达式**

**调用：直接把通知注解中的value赋值方法名**

==增强类优先级==

**在增强类上添加@Order注解，数值类型值越小优先级越高**

## jdbcTemplate

**Spring框架对JDBC进行封装，使用jdbcTemplate方便实现对数据库的操作**

### 准备工作

==开启组件扫描，配置数据连接池dataSource==

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210113163938102.png" alt="image-20210113163938102" style="zoom:80%;" />

==配置文件中创建jdbcTemplate实例，注入dataSource==

```xml
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <!--注入dataSource-->
    <property name="dataSource" ref="dataSource"></property>
</bean>
```

==在dao类注入jdbcTemplate对象==

### 操作数据库

==添加、修改、删除：==

**调用jdbcTemplate的update(参数1,参数2)方法**

* 参数1：sql语句
* 参数2：sql语句中的参数

==查询返回某个值：==

**调用jdbcTemplate的queryForObject(参数1,参数2)方法**

* 参数1：sql语句
* 参数2：返回类型.class

==查询返回对象：==

**调用jdbcTemplate的queryForObject(参数1,参数2,参数3)**

* 参数1：sql语句
* 参数2：RowMapper接口，返回不同类型数据，使用这个接口里面实现类完成数据封装
* 参数3：sql语句中的参数

==查询返回对象集合：==

**调用jdbcTemplate的query(参数1,参数2,参数3)方法**

* 参数1：sql语句
* 参数2：RowMapper接口，返回不同类型数据，使用这个接口里面实现类完成数据封装
* 参数3：sql语句中的参数

==批量操作：==

**调用jdbcTemplate的batchUpdate(参数1,参数2)方法**

* 参数1：sql语句
* 参数2：List集合，添加多条记录数据

## 事务

**事务是数据库操作最基本单元，逻辑上的一组操作，具有原子性**

==ACID==

* 原子性
* 一致性
* 隔离性
* 持久性

### 搭建环境

**service中注入dao，dao中注入jdbcTemplate，jdbcTemplate中注入dataSource**

### Spring事务管理

**事务添加到业务逻辑层，底层使用AOP原理**

==方式==

* 编程式事务管理
* 声明式事务管理（常用）
  * **基于注解方式**
  * 基于xml配置文件方式

==事务管理API==

**提供一个接口，代表事务管理器，这个接口针对不同的框架提供不同的实现类**

#### 注解实现事务管理

* Spring配置文件中配置事务管理器

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210113211020670.png" alt="image-20210113211020670" style="zoom:80%;" />

* Spring配置文件中开启事务注解

  * 引入名称空间tx

  * 开启事务注解

    ```xml
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
    ```

* 在service类上面或者service类方法上面添加事务注解@Transactional

  * 添加到类上面，这个类里面所有的方法都添加事务
  * 添加到方法上面，为这个方法添加事务

##### 注解事务管理参数配置

**在service类上面添加注解@Transactional，配置事务相关参数：**

* propagation：事务传播行为

  **对多事务方法之间互相调用的处理**

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210113214700856.png" alt="image-20210113214700856" style="zoom:70%;" />

* ioslation：事务隔离级别

  隔离性问题：

  * 脏读：一个未提交事务读取到了未提交事务的数据
  * 不可重复读：一个未提交事务读取到已提交事务修改的数据
  * 幻读：一个未提交事务读取到已提交事务添加的数据

  **隔离级别：**

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210114092259834.png" alt="image-20210114092259834" style="zoom:65%;" />

* timeout：超时时间

  * **事务需要在一定时间内进行提交，如果未提交则进行回滚**
  * **默认值是-1（永不超时），可以设置时间以秒为单位**

* readOnly：是否只读

  * 读：查询操作，写：添加修改删除
  * 默认值是false，可修改为true：只进行查询操作

* rollbackFor：回滚

  **设置出现哪些异常进行回滚**

* noRollbackFor：不回滚

  **设置出现哪些异常不进行回滚**

#### xml实现事务管理

* spring配置文件中配置事务管理器

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210114111136907.png" alt="image-20210114111136907" style="zoom:80%;" />

* 配置通知

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210114111028457.png" alt="image-20210114111028457" style="zoom:80%;" />

* 配置切入点和切面

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210114111104938.png" alt="image-20210114111104938" style="zoom:80%;" />

#### 完全注解实现事务管理

**创建配置类，使用配置类替代xml配置文件**

## Spring5新特性

**Spring5框架基于Java8开发，运行时兼容JDK9**

### Spring5封装日志功能

* Spring5移除了Log4jConfigListener，建议使用Log4j2

==Spring5整合Log4j2==

* 引入依赖

* 创建log4j2.xml配置文件

  **日志级别及优先级排序：OFF>FATAL>ERROR>WARN>INFO>DEBUG>TRACE>ALL**

### 支持@Nullable注解

* 添加在方法上：方法返回值可以为null

* 添加在方法参数上：参数值可为null

* 添加在属性上：属性值可为null

### 函数式风格GenericApplicationContext

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210114141419615.png" alt="image-20210114141419615" style="zoom:80%;" />

### JUnit5

==JUnit4==

* 引入依赖

  ![image-20210114142309647](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210114142309647.png)

*  创建测试类，使用注解方式

  * 单元测试框架：@RunWith(SpringJUnit4ClassRunner.class)
  * 加载配置文件：@ContextConfiguration("classpath:配置文件.xml")

==JUnit5==

* 引入依赖
* 创建测试类，使用注解方式
  * @SpringJUnitConfig(locations = "classpath:配置文件.xml")

### WebFlux

**Spring5添加的新模块，用于web开发，功能与SpringMVC类似，响应式编程框架**

**传统web框架，基于servlet容器，WebFlux是一种异步非阻塞框架，Servlet3.1后才支持，核心是基于Reactor的相关API实现的**

* 异步和同步针对调用者
  * 同步：调用者发送请求后，等待回复才做其他操作
  * 异步：调用者发送请求后，不需要等待回复就可以做其他操作
* 阻塞和非阻塞针对被调用者
  * 阻塞：被调用者接收请求后，先执行操作再反馈
  * 非阻塞：被调用者接收请求后，先反馈再执行操作

==特点==

* 非阻塞式
* 函数性编程

==WebFlux Vs SpringMVC==

* 都可以使用注解方式，都可以运行在Tomcat等容器上
* SpringMVC使用命令式编程，WebFlux使用异步响应式编程

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210114150221368.png" alt="image-20210114150221368" style="zoom:80%;" />

