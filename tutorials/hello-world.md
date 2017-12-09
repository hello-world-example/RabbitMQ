# 资源
> 官方 ["Hello World" Java](http://www.rabbitmq.com/tutorials/tutorial-one-java.html)
> [Python 版中文教程](http://rabbitmq.mr-ping.com/tutorials_with_python/[1]Hello_World.html)


# 依赖
```xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>4.3.0</version>
</dependency>

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.21</version>
</dependency>
```

# Sender 

``` java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class Send {
  
  // 队列名是 hello
  private final static String QUEUE_NAME = "hello";

  public static void main(String[] argv) throws Exception {
    // 1. 创建链接工厂
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    
    // 2. 创建链接
    Connection connection = factory.newConnection();
    
    // 3. 创建信道
    Channel channel = connection.createChannel();

    // 4. 声明一个队列
    channel.queueDeclare(QUEUE_NAME, false, false, false, null);
    
    // 5. 发送消息
    String message = "Hello World!";
    channel.basicPublish("", QUEUE_NAME, null, message.getBytes("UTF-8"));
    System.out.println(" [x] Sent '" + message + "'");
    
    // 6. 关系资源
    channel.close();
    connection.close();
  }
}
```

# Receiver

``` java
import com.rabbitmq.client.*;

import java.io.IOException;

public class Recv {

  // 队列名是 hello
  private final static String QUEUE_NAME = "hello";

  public static void main(String[] argv) throws Exception {
    // 1. 创建链接工厂
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    // factory.setUsername("guest"); // 用户名 默认是 guest
    // factory.setPassword("guest"); // 密码 默认是 guest
    // factory.setVirtualHost("/"); // 虚拟主机默认是 /
    // 其他 factory 设置
    
    // 2. 创建链接
    Connection connection = factory.newConnection();
    
    // 3. 创建信道
    Channel channel = connection.createChannel();

    // 4. 声明一个队列（如果队列已经存在，这里也可以不声明）
    // 参数2：durable 是否是持久化队列
    // 参数3：exclusive
    // 参数4：autoDelete 
    // 参数5：arguments 队列扩展参数
    channel.queueDeclare(QUEUE_NAME, false, false, false, null);
    
    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");
    
    // 5. 消费者逻辑
    channel.basicConsume(QUEUE_NAME, true, new DefaultConsumer(channel) {
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body)  throws IOException {
            String message = new String(body, "UTF-8");
            System.out.println(" [x] Received '" + message + "'");
        }
    });
  }
}
```