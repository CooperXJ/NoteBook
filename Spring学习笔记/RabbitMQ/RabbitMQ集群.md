#### 集群架构

##### 	普通集群（副本集群）

​	默认情况下：RabbitMQ代理所有操作所需的所有数据/状态将跨所有节点复制，这方面的一个例外是消息队列，默认情况下，消息队列位于一个节点上，尽管它们是可以从所有节点看到和访问。

​	也就说从节点只复制了除了消息队列以外的所有信息（例如交换机的信息），但是就是没有复制消息队列的信息，<font color=red>**那么当主节点宕机之后，消息队列也就会随主节点的消失而消失了，无法通过从节点进行访问**</font>。

​	架构图：

​		<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200907142249.png" alt="image-20200907142236833" style="zoom:50%;position=center" />

​	核心缺陷：

​		**主节点宕机之后，消息队列也就会随主节点的消失而消失了，无法通过从节点进行访问**

​	

##### 	集群搭建

​		按照Rabbit安装教程安装好之后

- 编辑/etc/hosts  添加主从节点

  - 同步erlang cookie

    scp /var/lib/rabbitmq/.erlang.cookie  root@ip:/var/lib/rabbitmq/

- master后台启动

- 从节点停止启动

  `rabbitmq stop_app`

- 从节点加入集群

  `rabbitmqctl join_cluster rabbit@{主节点}`

- 启动从节点

  `rabbitmq start_app`



##### 镜像集群

<font color=red>**当主节点宕机之后，消息队列不会随主节点的消失而消失了，可以通过从节点进行访问**</font>

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200908103655.png" alt="image-20200908103649744" style="zoom:50%;" />

在原来普通集群的基础上进行配置

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200908105025.png" alt="image-20200908104515651" style="zoom:50%;" />

