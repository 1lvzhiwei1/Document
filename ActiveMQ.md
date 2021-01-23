# ActiveMQ

## 基础操作

==普通启动==

bin目录下，**./activemq statrt**,默认端口61616

==关闭服务==

bin目录下，**./activemq stop**

==重启服务==

bin目录下，**./activemq restart**

==日志记录==

将日志存入文件，**./activemq statrt > /activemq/xxx.log**

## ActiveMQ控制页面

==访问路径：==

**服务器地址:8161   ====>**   **192.168.17.129:8161**

* 用户名：admin
* 密码：admin

## Queue

==目的地Destination==

* **点对点的消息传递域中，目的地被称为队列**
* **发布订阅消息传递域中，目的地被称为主题**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210118203436019.png" alt="image-20210118203436019" style="zoom:80%;" />

### 消息生产者

* 创建连接工厂

  ```java
  public static final String ACTIVEMQ_URL = "tcp://192.168.17.129:61616"
  
  ActiveMQConnectFactory activeMQConnectFactory = new ActiveMQConnectFactory(ACTIVEMQ_URL);
  ```

* 获得连接connection并启动访问

  ```java
  Connection connection = activeMQConnectionFactory.createConnection();
  connection.start();
  ```

* 创建会话session,两个参数

  * transacted：事务
  * acknowledge：签收

  ```java
  Session session = connection.createSession(transacted,acknowledge);
  ```

* 创建目的地

  * 队列

    ```java
    Queue queue = session.createQueue(queuename);
    ```

* 创建消息的生产者

  ```java
  MessageProducer messageProducer = session.creatProducer(queue);
  ```

* 通过messageProducer生产消息发送到MQ的队列中

  ```java
  TextMessage textMessage = session.createTextMessage("xxx");
  
  messageProducer.send(textMessage);
  ```

* 关闭资源

  ```java
  messageProducer.close();
  session.close();
  connection.close();
  ```


### 消息消费者

* 创建连接工厂

  ```java
  public static final String ACTIVEMQ_URL = "tcp://192.168.17.129:61616"
  
  ActiveMQConnectFactory activeMQConnectFactory = new ActiveMQConnectFactory(ACTIVEMQ_URL);
  ```

* 获得连接connection并启动访问

  ```java
  Connection connection = activeMQConnectionFactory.createConnection();
  connection.start();
  ```

* 创建会话session,两个参数

  * transacted：事务
  * acknowledge：签收

  ```java
  Session session = connection.createSession(transacted,acknowledge);
  ```

* 创建目的地

  * 队列

    ```java
    Queue queue = session.createQueue(queuename);
    ```

* 创建消费者

  ```java
  MessageConsumer messageConsumer = session.createConsumer(queue);
  ```

#### 同步阻塞方式receive()

* 通过 messaConsumer 的receive()方法接收消息,并取出消息

  ```java
  TextMessage message = (TextMessage) messageConsumer.receive();
  
  message.getText();
  ```

* 关闭资源

  ```java
  messageProducer.close();
  session.close();
  connection.close();
  ```

==receive==

* recevie()：一直等待接收，不会结束接收
* receive(timeout)：具有超时参数，超时则结束接收

#### 异步非阻塞方式(监听器)

* 创建监听器内部类

  ==System.in.read();==  

  **保持控制台不结束，监听器一直监听**

  ```java
  messageConsumer.setMessageListener(new MessageListener()){
      @Override
      public void onMessage(Message message){
          if(null != message && message instanceof TextMessage){
              TextMessage textmMessage = (TextMessage) message;
              try{
                  System.out.println("***消费者接收消息："+textmMessage.getText()); 	
              }catch(JMSException e){
                  e.printStackTrace();
              }
          }
      }
  });
  
  System.in.read();
  ```

* 关闭资源

  ```java
  messageProducer.close();
  session.close();
  connection.close();
  ```

#### 三大消费情况

==先生产消息，启动消费者1==

**消费者1消费消息**

==先生产消息，启动消费者1，再启动消费者2==

**先到先得，消费者1消费消息**

==先启动两个消费者，再生产消息==

**消费者平均分配消息数**

## Topic

```java
Topic topic = session.createTopic(TOPIC_NAME);
```

* **生产者和消费者是一对多关系，类似微信公众号**

* **生产者和消费者具有时间相关性，消费者只能消费自它订阅之后发布的消息**
* **先要有消费者再有生产者，topic只管发布，不管消费者是否接收成功，会产生废消息**

## Queue Vs Topic

![image-20210119203600014](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210119203600014.png)

## JMS四大组成元素

* JMS Provider
* JMS Producer
* JMS Consumer

### JMS Message

==消息头==

* JMSDestination：消息发送目的地，主要指Queue和Topic
* JMSDeliveryMode：持久和非持久模式
  * 持久消息：服务器出现故障，消息并不会丢失，会在服务器恢复之后再次传递
  * 非持久消息：服务器出现故障，该消息将永远消失
* JMSExpiration：消息的过期时间，默认永不过期
* JMSPriority：消息优先级，0-4级别是普通消息，5-9级别是加急消息，默认是4级  **JMS不要求MQ严格按照优先级发送消息，但必须保证加急消息要先于普通消息到达**
* JMSMessageID：MQ产生的消息唯一标识

==消息属性==

**识别、去重、重点标注等操作**

==消息体==

**封装具体的消息数据**

**发送和接受的消息体类型必须一致**

消息体格式：

* **TextMessage：普通字符串消息，包含一个字符串**
* **MapMessage：Map类型的消息，key为String类型，value为java基本类型**
* BytesMessage：二进制数组消息，包含一个byte[]
* StreamMessage：Java数据流消息，用标准流操作来顺序的填充和读取
* ObjectMessage：对象消息，包含一个可序列化的Java对象

## JMS的可靠性

### PERSISTENT(持久化)

==参数设置==

* 持久化：当服务器宕机，消息依然存在

  ```java
  messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
  ```

* 非持久化：当服务器宕机，消息永久丢失

  ```java
  messageProducer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
  ```

==持久化Queue==

**队列默认持久化消息，保证消息只被传送一次和成功使用一次，可靠性是优先考虑的因素**

==持久化Topic==

*  先运行一次消费者，向MQ注册
* 再运行生产者发送消息，无论消费者是否在线，都会收到消息

### Transaction(事务)

 ==false==

**只要执行send()，就进入队列中**

**关闭事务，则签收参数需要设置成有效**

==true==

**先执行send()，再执行commit()，消息才被真正的提交到队列中**

**消息需要批量发送，需要缓冲区处理**

### Acknowledge(签收)

==非事务==

* **自动签收**

  **默认**

  ```java
  Session.AUTO_ACKNOWLEDGE
  ```

* **手动签收**

  **客户端调用message.acknowledge()手动签收**

  ```java
  Session.CLIENT_ACKNOWLEDGE
  ```

* 允许重复消息

  ```java
  Session.DUPS_OK_ACKNOWLEDGE
  ```

==事务==

* **生产事务开启，只有commit后才能将全部消息变为已消费**

* **事务性会话中，当一个事务被成功提交则消息自动签收**
* **如果事务回滚，则消息会被再次传送**

## Broker

**相当于一个ActiveMQ服务器实例，用代码的形式启动ActiveMQ将MQ嵌入到Java代码中，随用随启**

==基于指定activemq.xml配置文件启动==

```shell
./activemq start xbean:file:/xxx/xxx/xx.xml
```

### 嵌入式Broker

**用ActiveMQ Broker作为独立的消息服务器来构建Java应用**

==pom.xml引入依赖==

```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.5</version>
</dependency>
```

==编写嵌入式Broker==

```java
BrokerService brokerService = new BrokerService();
        brokerService.setUseJmx(true);
        brokerService.addConnector("tcp://localhost:61616");
        brokerService.start();
```

## Spring整合ActiveMQ

==pom.xml导入依赖==

```xml
<dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jms</artifactId>
            <version>4.3.23.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-pool</artifactId>
            <version>5.15.9</version>
        </dependency>
```

==配置生产者==

```xml
<!-- 配置生产者 -->
    <bean id="jmsFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
        <property name="connectionFactory">
            <bean class="org.apache.activemq.ActiveMQConnectionFactory">
                <property name="brokerURL" value="tcp://http://192.168.17.129:61616"></property>
            </bean>
        </property>
        <property name="maxConnections" value="100"></property>
    </bean>
```

==队列目的地==

```java
<!-- 队列目的地，点对点 -->
    <bean id="destinationQueue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg index="0" value="spring-active-queue"/>
    </bean>
```

==JMS工具类，进行消息发送、接收等==

```java
<!-- JMS工具类 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <property name="connectionFactory" ref="jmsFactory"/>
        <property name="defaultDestination" ref="destinationQueue"/>
        <property name="messageConverter">
            <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
        </property>
    </bean>
```

### Queue

==编写消息生产者==

```java
@Service
public class SpringMQ_Producer {

    @Autowired
    private JmsTemplate jmsTemplate;

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        SpringMQ_Producer producer = (SpringMQ_Producer) context.getBean("springMQ_Producer");
        producer.jmsTemplate.send(new MessageCreator() {
            public Message createMessage(Session session) throws JMSException {
                TextMessage textMessage = session.createTextMessage("******Spring和ActiveMQ的整合case....");
                return textMessage;
            }
        });
        System.out.println("send task over");
    }
}
```

==编写消息消费者==

```java
@Service
public class SpringMQ_Consumer {

    @Autowired
    private JmsTemplate jmsTemplate;

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        SpringMQ_Consumer springMQ_consumer = (SpringMQ_Consumer) context.getBean("springMQ_Consumer");
        String retValue = (String) springMQ_consumer.jmsTemplate.receiveAndConvert();
        System.out.println("*******消费者收到的消息:"+retValue);
    }
}
```

### Topic

==修改applicationContext.xml==

```xml
<!-- 主题目的地，发布订阅 -->
    <bean id="destinationTopic" class="org.apache.activemq.command.ActiveMQTopic">
        <constructor-arg index="0" value="spring-active-topic"/>
    </bean>

<!-- JMS工具类 -->
    <bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
        <property name="connectionFactory" ref="jmsFactory"/>
        <property name="defaultDestination" ref="destinationTopic"/>
        <property name="messageConverter">
            <bean class="org.springframework.jms.support.converter.SimpleMessageConverter"/>
        </property>
    </bean>
```

### 监听器配置

==applicationContext.xml==

```xml
    <bean id="jmsContainer" class="org.springframework.jms.listener.DefaultMessageListenerContainer">
        <property name="connectionFactory" ref="jmsFactory"/>
        <property name="destination" ref="destinationTopic"/>
        <property name="messageListener" ref="myMessageListener"/>
    </bean>
```

==编写MessageListener接口实现类==

```java
@Service
public class MyMessageListener implements MessageListener {
    public void onMessage(Message message) {
        if(null != message && message instanceof TextMessage){
            TextMessage textMessage = (TextMessage) message;
            try {
                System.out.println(textMessage.getText());
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 传输协议

**ActiveMQ支持的通讯协议：TCP、NIO、UDP、SSL、Http、VM** ==默认是TCP，监听接口是61616==

**配置Transport Connector的文件在ActiveMQ安装目录的conf/activemq.xml的<transportConnectors\>标签中**

### NIO

**NIO更注重底层的访问操作，比TCP的性能更好**

==连接的URL形式==

```xml
nio://hostname:port?key=value
```

==添加NIO协议连接==

```xml
<transportConnectors>
	<transportConnector name="nio" uri="nio://0.0.0.0:61618?trace=true"/>
</transportConnectors>
```

### NIO加强

```xml
<transportConnectors>
	<transportConnector name="auto+nio" uri="auto+nio://0.0.0.0:61608?trace=true"/>
</transportConnectors>
```

## 消息持久化

**ActiveMQ的消息持久化机制有KahaDB和LevelDB**

==逻辑==

**生产者发送消息后，消息中心先将消息存储到本地数据文件、内存数据库或者远程数据库再将消息发送给消费者，发送成功则将消息从存储中删除，失败则继续尝试发送**

**消息中心启动之后先检查指定的存储位置，如果有未发生成功的消息，则需要把消息发送出去**

### KahaDB

**基于日志文件，从ActiveMQ5.4开始默认的持久化插件**

**消息存储使用一个事务日志和仅仅用一个索引文件来存储所有的地址**

==文件解释==

* db-<Number\>.log：kahaDB存储消息到预定义大小(32MB)的数据记录文件中，当数据文件已满时，一个新的文件随之创建Number数值递增	**不再有引用到数据文件中的任何消息的时，文件会被删除或归档**
* db.data：该文件包含了持久化的BTree索引，索引了消息数据记录中的消息
* db.free：记录db.data文件中空闲页面的ID
* db.redo：用来进行消息恢复，如果KahaDB消息存储在强制退出后启动，恢复BTree索引
* lock：文件锁，表示当前获得KahaDB读写权限的broker

### LevelDB

**ActiveMQ5.8之后引进，基于文件的本地数据库存储形式，比KahaDB更快的持久性**

**使用基于LevelDB的索引**

###  JDBC

* 添加mysql数据库的驱动包到lib文件夹

* 修改activemq.xml

  ```xml
  <persistenceAdapter>
  	<jdbcPersistenceAdapter dataSource="#mysql-ds"/>
  </persistenceAdapter>
  ```

* 设置createTablesOnStartup属性(是否在启动的时候创建数据表)，第一次启动设置为true之后改为false

* 配置数据库连接

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210123161519762.png" alt="image-20210123161519762" style="zoom:80%;" />

* 数据库建表,自动生成三张表

  * ACTIVEMQ_MSGS：消息表 
  * ACTIVEMQ_ACKS：存储订阅关系
  * ACTIVEMQ_LOCK：用于集群环境，记录Master Broker