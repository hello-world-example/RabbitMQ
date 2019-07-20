# 资源
> 官方 [Work Queues](http://www.rabbitmq.com/tutorials/tutorial-two-java.html)
> [Python 版中文教程](http://rabbitmq.mr-ping.com/tutorials_with_python/[2]Work_Queues.html)
> 
> > Client 教程 [Hello World](tutorials/hello-world.md)

# NewTask 生产者

```java

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

public class NewTask {

    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);

        String[] messages = new String[]{
                "Hello World1",
                "Hello World2",
                "Hello World3",
                "Hello World4",
                "Hello World5",
                "Hello World6"
        };

        for (String message : messages) {
            // 发布消息
            // 参数3：设置消息的属性（Content-type "text/plain", deliveryMode 2 (persistent), priority zero）
            channel.basicPublish("", TASK_QUEUE_NAME, true, true, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        }

        channel.close();
        connection.close();
    }
}
```

# Worker 消费者

```java
import com.rabbitmq.client.*;
import java.io.IOException;

public class Worker {

    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        /*
         * Qos = quality of service (服务质量)
         * 这里参数 prefetchCount 设置为1，global参数默认是false（限制级别是这个信道下面的每一个消费者）
         *
         * 作用是：如果有多个消费者(消费同一个队列)，RabbitMQ 会遍历每个消费者，
         *        如果当前遍历到的消费者 unack 的数量是1(prefetchCount=1)，则不给改消费者推送消息，自动跳到下一个消费者
         *
         * 注意：只有在 autoAck 设为false 的时候才有作用
         */
        channel.basicQos(1);

        // 第二个参数 global 默认是 false
        // 如果是false：代表对每个消费者(consumer)进行限制
        // 如果是 true：代表对信道(channel)进行限制
        // channel.basicQos(1, false);

        final Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String message = new String(body, "UTF-8");

                System.out.println(" [x] Received '" + message + "'");
                try {
                    doWork(message);
                } finally {
                    System.out.println(" [x] Done");

                    /*
                     * 消息确认
                     * 参数1：deliveryTag 代表一个消息的标识
                     * 参数2：multiple false:仅对当前消息ack，true:一次性ack所有小于deliveryTag的消息
                     * http://www.rabbitmq.com/confirms.html --> Acknowledging Multiple Deliveries at Once
                     */
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            }
        };
        // 参数2：autoAck 设为false，表示需要手动确认消息
        channel.basicConsume(TASK_QUEUE_NAME, false, consumer);
    }

    /**
     * 逻辑是如果 参数message 有几个点就 sleep 多少秒
     */
    private static void doWork(String massage) {
        for (char ch : massage.toCharArray()) {
            if (ch == '.') {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException _ignored) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }
}

```

# 运行方式
- 先启动两个 Worker
- 再启动一个 NewTask
- 可以看到两个 一个Worker输出 1、3、5，另一个Worker输出 2、4、6


# 扩展阅读
> [Consumer Prefetch](http://www.rabbitmq.com/consumer-prefetch.html)
> [rabbitmq channel参数详解](http://www.cnblogs.com/piaolingzxh/p/5448927.html)
> [rabbitmq——prefetch count](https://my.oschina.net/hncscwc/blog/195560)