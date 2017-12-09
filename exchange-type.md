


RabbitMQ中，所有生产者提交的消息都由Exchange来接受，然后Exchange按照特定的策略转发到Queue

RabbitMQ提供了四种Exchange：`direct`、`fanout`、`topic`、`headers`



# direct

- 任何发送到`Direct Exchange`的消息都会被转发到`RouteKey`中指定的`Queue`，可简单的认为`RouteKey`就是队列名
- RabbitMQ自带一个`(AMQP default)`Exchange，名字是`""`空字符串，称为 `default Exchange`
- 不需要将 Queue 与 Exchange 进行绑定(binding)操作
- 如果vhost中**不存指定的Queue**，则该**消息会被抛弃**


ＴＯＤＯ　direct&default 对比 ：

> [RabbitMQ exchanges: default vs. direct](https://stackoverflow.com/questions/14480052/rabbitmq-exchanges-default-vs-direct)



# fanout

- 任何发送到`Fanout Exchange`的消息都会被转发到与该Exchange绑定(Binding)的`所有Queue`上
- 这种模式**不需要RouteKey**
- 需要提前将Exchange与Queue进行绑定，**一个Exchange可以绑定多个Queue**，**一个Queue可以同多个Exchange进行绑定**
- 如果接受到消息的**Exchange没有与任何Queue绑定，则消息会被抛弃**。





# topic

- 任何发送到`Topic Exchange`的消息都会被转发到所有**关心RouteKey中指定话题的Queue上**
- 简单来说，就是每个队列都有其关心的主题，所有的**消息都带有一个“标题”(RouteKey)**，Exchange会将消息转发到所有关注主题能与RouteKey模糊匹配的队列
- 这种模式**需要RouteKey**，也需要提前绑定Exchange与Queue。
- 在进行绑定时，要提供一个该队列关心的主题
    - `#.log.#`表示该队列关心所有涉及log的消息，RouteKey为`MQ.log.error`的消息会被转发到该队列
    - `#`表示0个或若干个关键字，`*`表示一个关键字。如“log.*”能与“log.warn”匹配，无法与“log.warn.timeout”匹配；但是`log.#`能与上述两者匹配。
- 如果Exchange没有发现能够与RouteKey匹配的Queue，则会抛弃此消息。





# headers

- 发送者在发送的时候定义一些键值对，接收者在绑定时候传入一些键值对，两者匹配的话，则对应的队列就可以收到消息
- 匹配有两种方式`all`和`any`，这两种方式是在接收端必须要用键值`x-mactch`来定义
    - all代表定义的多个键值对都要满足
    - any则代码只要满足一个就可以了
- 其他 Exchange类型的routingKey都需要要字符串形式的，而headers Exchange键值对的值可以是任何类型
- 不需要 routingKey，使用Headers来匹配




## 对比
// TODO Table



# Exchange 属性

# 常见队列属性
- 延迟队列
- 超时
- 持久化
- Oterh


# 拓展阅读
> [AMQP 0-9-1 Model Explained](http://www.rabbitmq.com/tutorials/amqp-concepts.html)
> [Clients & Developer Tools](http://www.rabbitmq.com/devtools.html#java-dev)
> [Java Tutorial](https://www.rabbitmq.com/tutorials/tutorial-one-java.html)
> [Documentation: Table of Contents](https://www.rabbitmq.com/documentation.html)