#### docker compose

##### 安装

```shell
#国内镜像源
curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
#赋予权限
chmod +x /usr/local/bin/docker-compose
```



##### 启动

在有docker-compose.yml的文件目录下启动docker-compose

- 启动`docker-compose up`

- 停止 `docker-compse down`

- 重新部署打包  `docker-compose up --build`

docker-compose 每次启动之后都会自己创建一个网络，用于内部的通信，项目中的所有容器都在同一个网络下，这样的话彼此之间可以通过域名访问

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200923225137.png" alt="image-20200923222332638" style="zoom:50%;" />



##### yaml文件规则

```yaml
#3层
version: 版本 需要与docker引擎对应
service: 服务
	服务1：web
	服务2：redis
	...
	服务n：
	
volumes:
networks:
config:
```



 

#### docker swarm

##### 初始化节点

`docker swarm init`

```shell
#获取令牌
docker swarm join-token manager  #作为管理者结点加入集群
docker swarm join-token worker	#作为工作结点加入集群
```

如果这些机器不在同一个内网中，需要彼此之间做认证才行

只有管理者节点才可以执行命令，worker节点无法执行



##### Raft协议（保证大多数结点存活才可以使用）

比如双主双从，其中还只要有一个主节点挂机，那么另外一个主节点也将不可以用

可以将其他结点从集群中离开，只需要在该离开的结点中输入：`docker swarm leave`



#### service

服务可以动态的扩缩容，对于上层的调用者来说他就是一个

`docker service`

`docker service ps 服务名`

- `docker service update --replicas {副本数量} {服务名}` 或者 `docker service scale {服务名}={副本数量}`

通过该命令创建的副本会随机分布在集群的主机上，并且通过集群中的每一个节点都可以对服务进行访问，不管该节点有没有该服务



##### docker swarm 网络

当docker swarm 搭建集群成功之后，会自动生成一个ingress网络，该网络的类型是overlay

<img src="/Users/cooper/Library/Application Support/typora-user-images/image-20200924093337761.png" alt="image-20200924093337761" style="zoom:50%;" />

overlay可以使得不同docker的内部网络相互ping通，使得集群间的网络成为了一个整体