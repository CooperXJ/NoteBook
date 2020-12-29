#### 添加依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```



#### 五种Provider

```java
@Autowired
RabbitTemplate template;

@Test
public void testHelloWorld()
{
    template.convertAndSend("hello","Hello World");
}

@Test
public void testWork()
{
    for(int i =0;i<10;i++)
        template.convertAndSend("work","Hello World "+i);
}

@Test
public void testFanout()
{
    template.convertAndSend("springboot_logs","","Fanout模型");
}

@Test
public void testDirect()
{
    template.convertAndSend("direct","error","direct模型");
}

@Test
public void testTopic()
{
    template.convertAndSend("topic","error.error","topic模型");
}
```



#### 五种Consumer



##### HelloWorld

```java
@Component
@RabbitListener(queuesToDeclare = @Queue("hello"))
public class HelloConsumer {

    @RabbitHandler
    public void receive(String message)
    {
        System.out.println("message   "+message);
    }
}
```

##### Work

```java
@Component
public class WorkConsumer {

    @RabbitListener(queuesToDeclare = @Queue("work"))
    public void receive1(String message)
    {
        System.out.println("message1 "+message);
    }

    @RabbitListener(queuesToDeclare = @Queue("work"))
    public void receive2(String message)
    {
        System.out.println("message2 "+message);
    }
}
```

##### Direct

```java
@Component
public class DirectConsumer {
    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue,//未指定queue名称则创建临时queue
                    exchange = @Exchange(value = "direct",type = "direct"),//绑定交换机 默认类型时direct
                    key = {"info","error"}
            )
    })
    public void receive1(String message)
    {
        System.out.println("1 "+message);
    }

    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue,//未指定queue名称则创建临时queue
                    exchange = @Exchange(value = "direct",type = "direct"),//绑定交换机
                    key = {"info"}
            )
    })
    public void receive2(String message)
    {
        System.out.println("2 "+message);
    }
}
```

##### Topic

```java
@Component
public class TopicConsumer {
    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue,//未指定queue名称则创建临时queue
                    exchange = @Exchange(value = "topic",type = "topic"),//绑定交换机 默认类型时direct
                    key = {"info.*","error.*"}
            )
    })
    public void receive1(String message)
    {
        System.out.println("1 "+message);
    }

    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue,//未指定queue名称则创建临时queue
                    exchange = @Exchange(value = "topic",type = "topic"),//绑定交换机
                    key = {"info.#","#.error.#"}
            )
    })
    public void receive2(String message)
    {
        System.out.println("2 "+message);
    }
}
```

##### Fanout

```java
@Component
public class FanoutConsumer {
    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue,//未指定queue名称则创建临时queue
                    exchange = @Exchange(value = "springboot_logs",type = "fanout")//绑定交换机
            )
    })
    public void receive1(String message)
    {
        System.out.println("1 "+message);
    }

    @RabbitListener(bindings = {
            @QueueBinding(
                    value = @Queue,//未指定queue名称则创建临时queue
                    exchange = @Exchange(value = "springboot_logs",type = "fanout")
            )
    })
    public void receive2(String message)
    {
        System.out.println("2 "+message);
    }
}
```