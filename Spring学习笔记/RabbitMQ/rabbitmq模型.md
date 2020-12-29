#### 直连模型

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200904110719.png" alt="image-20200904110714767" style="zoom:50%;" />

在上图的模型中，有以下概念：

- P:生产者，也就是需要发送消息的程序
- C:消费者，消息的接收者
- queue：消息队列，可以缓存消息，生产者往其中投递消息，消费者从其中取出消息



🌰

Provider

```java
ConnectionFactory factory = new ConnectionFactory();
//设置连接主机
factory.setHost("127.0.0.1");
//设置连接端口
factory.setPort(5672);
//设置连接虚拟主机
factory.setVirtualHost("/ems");
//设置账户密码
factory.setUsername("ems");
factory.setPassword("123");

//获取连接对象
Connection connection = factory.newConnection();

//获取连接通道
Channel channel = connection.createChannel();

//绑定对应的消息队列
/**
 * 参数s 队列名称  不存在则自动创建
 * 参数b 队列是否需要持久化  （不包括消息持久化）
 * 参数b1 是否需要独占队列(当前队列只运行当前连接可用？)
 * 参数b2 是否在消费完成后自动删除队列
 * 参数 map  额外的附加参数
 */
channel.queueDeclare("hello",false,false,false,null);

/**
 * 参数s 交换机名称
 * 参数s1 路由规则
 * 参数3 传递消息额外设置  (例如消息持久化 MessageProperties.PERSISTENT_TEXT_PLAIN)
 * 参数4  消息的具体内容
 */
channel.basicPublish("","hello",null,"hello rabbotmq".getBytes());

channel.close();
connection.close();
```

Consumer

```java
 ConnectionFactory factory = new ConnectionFactory();
 //设置连接主机
 factory.setHost("127.0.0.1");
 //设置连接端口
 factory.setPort(5672);
 //设置连接虚拟主机
 factory.setVirtualHost("/ems");
 //设置账户密码
 factory.setUsername("ems");
 factory.setPassword("123");

 //获取连接对象
 Connection connection = factory.newConnection();

 //获取连接通道
 Channel channel = connection.createChannel();

 //绑定对应的消息队列  要保证生产者和消费者各项参数严格对应上 ，比如说生产者持久化了，那么消费者也需要持久化
 /**
  * 参数s 队列名称  不存在则自动创建
  * 参数b 队列是否需要持久化
  * 参数b1 是否需要独占队列(当前队列只运行当前连接可用？)
  * 参数b2 是否在消费完成后自动删除队列  （消费者断开连接之后队列才会被删除掉）
  * 参数 map  额外的附加参数
  */
 channel.queueDeclare("hello",false,false,false,null);

 //接受是一个多线程，持续监听队列中的消息 最好不要关闭连接  在测试的时候不能再@Test注释的方法中写，因为@Test修饰的方法不支持多线程
 /**
  * 参数1 队列 名称
  * 参数2 开始消息的自动确认
  * 参数3 消费时的回调接口
  */
channel.basicConsume("hello",true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println(new String(body));
    }
});
```



#### 任务模型

​	当消息处理比较耗时时，生成消息的速度会远远大于消息的消费速度，长此以往，消息就会堆积越来越多，无法及时处理。此时可以使用任务模型（work模型）：**让多个消费者绑定到一个队列，共同消费队列中的信息**。队列中的消息一旦消费就会消失，因此任务不会被重复执行。

​	任务队列默认在分发任务的时候是平均分配任务的，比如有两个消费者，那么每个消费者接收到任务是总任务的一般（默认情况下即使两个消费者的工作效率不相同也会平均任务）

- P: 生产者 任务的发布者
- C1: 消费者-1 领取任务并且完成任务，假设完成速度较慢
- C2: 消费者-2 领取任务并且完成任务，假设完成速度较快

##### 基础班（平均主义）

🌰

Provider

```java
 Channel channel = utils.getChannel();

    //绑定对应的消息队列
    /**
     * 参数s 队列名称  不存在则自动创建
     * 参数b 队列是否需要持久化  （不包括消息持久化）
     * 参数b1 是否需要独占队列(当前队列只运行当前连接可用？)
     * 参数b2 是否在消费完成后自动删除队列
     * 参数 map  额外的附加参数
     */
    channel.queueDeclare("hello",false,false,false,null);

    for(int i =0;i<20;i++)
    {
        channel.basicPublish("","hello",null,(i+"   hello rabbotmq").getBytes());
    }

}
```

Consumer1，2代码都相同

```java
RabbitMQUtils utils = new RabbitMQUtils();
Channel channel = utils.getChannel();

channel.queueDeclare("hello",false,false,false,null);

//接受是一个多线程，持续监听队列中的消息 最好不要关闭连接  在测试的时候不能再@Test注释的方法中写，因为@Test修饰的方法不支持多线程
/**
 * 参数1 队列 名称
 * 参数2 开始消息的自动确认
 * 参数3 消费时的回调接口
 */
channel.basicConsume("hello",true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println(new String(body));
    }
});
```



**缺陷：**

​		如果两个消费者有一个消费者突然宕机，但是它才处理了自己消息总量的一半，那么剩下的消息就会丢失掉。



##### 进阶版（能者多劳）

`channel.basicConsume("hello",true,new DefaultConsumer(channel))`

**此处的的第二个参数是autoAck，如果为true则自动帮消费者确认收到消息（也就是不管你消费者有没有执行完都一股脑全部都把消息给你，不管你有没有有消化完），否则不自动确认必须手动确认才会接受下一个消息。**

Provdeir和上面一样

Consumer1

```java
public static void main(String[] args) throws IOException, TimeoutException {
    RabbitMQUtils utils = new RabbitMQUtils();
    Channel channel = utils.getChannel();

    channel.queueDeclare("hello",false,false,false,null);
    channel.basicQos(1);//每次只能消费一个，不一股脑全部接受

    //接受是一个多线程，持续监听队列中的消息 最好不要关闭连接  在测试的时候不能再@Test注释的方法中写，因为@Test修饰的方法不支持多线程
    /**
     * 参数1 队列 名称
     * 参数2 开始消息的自动确认
     * 参数3 消费时的回调接口
     */
    channel.basicConsume("hello",false,new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(new String(body));
            /**
             * 手动确认
             * 参数1 确认队列中的哪个具体消息的标志
           	 * 参数2 false 是否开启多个消息同时确认
             */
            channel.basicAck(envelope.getDeliveryTag(),false);
        }
    });
}
```

Consumer2

```java
public static void main(String[] args) throws IOException, TimeoutException {
    RabbitMQUtils utils = new RabbitMQUtils();
    Channel channel = utils.getChannel();

    channel.queueDeclare("hello",false,false,false,null);
    channel.basicQos(1);//每次只能消费一个，不一股脑全部接受

    //接受是一个多线程，持续监听队列中的消息 最好不要关闭连接  在测试的时候不能再@Test注释的方法中写，因为@Test修饰的方法不支持多线程
    /**
     * 参数1 队列 名称
     * 参数2 开始消息的自动确认
     * 参数3 消费时的回调接口
     */
    channel.basicConsume("hello",false,new DefaultConsumer(channel){
        @Override
        public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(new String(body));
            /**
             * 手动确认
             * 参数1 确认队列中的哪个具体消息的标志
           	 * 参数2 false 是否开启多个消息同时确认
             */
            channel.basicAck(envelope.getDeliveryTag(),false);
        }
    });
}
```



#### 广播模型

![image-20200904094726571](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200904094730.png)

​	在广播模式下，消息发送流程如下：

	- 可以有多个消费者
	- **每个消费者有自己的queue**
	- **每个队列都要绑定到exchange**
	- **生产者发送的消息只能发送到交换机**，交换机决定要发送给哪个队列，生产者无法决定
	- 交换机把消息发送给绑定过的所有队列
	- 队列的消费者都能拿到消息，实现一条消息被多个消费者消费

​	

🌰

Provider

```java
Channel channel = utils.getChannel();
/**
 * 通道绑定交换机
 * 参数1 交换机名称
 * 参数2 交换机类型
 * 如果指定交换机不存在会进行自动创建
 */
channel.exchangeDeclare("logs","fanout");

channel.basicPublish("logs","",null,"fanout hello".getBytes());
utils.closeChannelAndConnection();
```

ConsumerX

```java
RabbitMQUtils utils = new RabbitMQUtils();
Channel channel = utils.getChannel();
/**
 * 通道绑定交换机
 * 参数1 交换机名称
 * 参数2 交换机类型
 */
channel.exchangeDeclare("logs","fanout");
//获取临时队列
String queue = channel.queueDeclare().getQueue();
//绑定交换机和队列
/**
 * 参数1 队列名称
 * 参数2 交换机名称
 * 参数3 路由规则  此处是扇形模型，因此不需要添加路由规则
 */
channel.queueBind(queue,"logs","");

channel.basicConsume(queue,true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者1"+new String(body));
    }
});
```



#### 路由模型

##### 	Direct

​		定义：在某些场景中我们希望不同的消息被不同的队列消费

​		在Direct模型下：

  - 队列与交换机的绑定不能是任意的绑定，而是需要指定一个RoutingKey（路由key）

  - 消息的发送方在向exchange发送消息时，也必须指定消息的RoutingKey

  - Exchange不再把消息交给每一个绑定的队列，而是根据消息的RoutingKey进行判断，只有队列的RoutingKey与消息的RoutingKey完全一致才会接收到下消息

    流程如下：

    ![image-20200904094810078](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200904094812.png)

​		图解

		- P:生产者，向exchange发送消息，发送消息时，会指定一个RoutingKey
		- X:交换机，接收生产者的消息，然后把消息传递给与RoutingKey完全匹配的队列
		- C1:消费者，其所在队列指定了需要RoutingKey为error的消息
		- C2：消费者，其所在队列指定了需要RoutingKey为info、error、warning的消息

​		

🌰

Provider

```java
Channel channel = utils.getChannel();
/**
 * 通道绑定交换机
 * 参数1 交换机名称
 * 参数2 交换机类型
 * 如果指定交换机不存在会进行自动创建
 */
channel.exchangeDeclare("logs_direct","direct");

String routingKey = "info";
channel.basicPublish("logs_direct",routingKey,null,("hello"+routingKey).getBytes());
utils.closeChannelAndConnection();
```

Consumer

```java
RabbitMQUtils utils = new RabbitMQUtils();
Channel channel = utils.getChannel();
/**
 * 通道绑定交换机
 * 参数1 交换机名称
 * 参数2 交换机类型
 */
channel.exchangeDeclare("logs_direct","direct");
//获取临时队列
String queue = channel.queueDeclare().getQueue();


//绑定交换机和队列
/**
 * 参数1 队列名称
 * 参数2 交换机名称
 * 参数3 路由规则
 */
channel.queueBind(queue,"logs_direct","info");
channel.queueBind(queue,"logs_direct","error");
channel.queueBind(queue,"logs_direct","track");

channel.basicConsume(queue,true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者"+new String(body));
    }
});
```

​	

##### 	Topic

​		Topic类型与Exchange与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过Topic类型的Exchange可以让队列在绑定RoutingKey的时候使用通配符。这种模型RoutingKey一般都是由一个或多个单词组成，多个单词之间以 “.”分割。

​		<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200904110731.png" alt="image-20200904105708765" style="zoom:50%;" />

​	通配符类型

 - *

   匹配不多不少恰好一个单词

   如audit.*  只能匹配audit.irs 也就是audit后面只跟一个单词

- #

  匹配一个或多个词

  如audit.# 匹配audit.irs.corporation或者audit.irs等

🌰

Provider

```java
Channel channel = utils.getChannel();
/**
 * 通道绑定交换机
 * 参数1 交换机名称
 * 参数2 交换机类型
 * 如果指定交换机不存在会进行自动创建
 */
channel.exchangeDeclare("logs_topic","topic");

String routingKey = "user.save";
channel.basicPublish("logs_topic",routingKey,null,("hello"+routingKey).getBytes());
utils.closeChannelAndConnection();
```

Consumer

```java
RabbitMQUtils utils = new RabbitMQUtils();
Channel channel = utils.getChannel();
/**
 * 通道绑定交换机
 * 参数1 交换机名称
 * 参数2 交换机类型
 */
channel.exchangeDeclare("logs_topic","topic");
//获取临时队列
String queue = channel.queueDeclare().getQueue();


//绑定交换机和队列
/**
 * 参数1 队列名称
 * 参数2 交换机名称
 * 参数3 路由规则
 */
channel.queueBind(queue,"logs_topic","user.*");

channel.basicConsume(queue,true,new DefaultConsumer(channel){
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消费者"+new String(body));
    }
});
```

