#### 基础命令

##### 镜像

- 查看镜像

  docker images 

- 删除镜像  

  先停止该镜像的服务并删除容器才能删除该镜像

  docker  rmi  ID

- 搜索镜像

  docker search name

- 下载镜像

  docker pull imageName

  docker会进行分层下载，相同镜像不同版本之间相同的东西只需要下载一次即可

- docker 删除所有的容器

  ```bash
  docker rm -f $(docker ps -a)
  ```

容器

- 容器退出

  ```bash
  exit #直接退出容器并停止
  Ctrl + P + Q #退出但容器不停止
  ```

- 启动和停止容器操作

  ```shell
  docker start 容器id #启动容器
  docker stop 容器id #停止容器
  docker restart 容器id #重启容器
  docker kill 容器id #强制停止正在运行的容器
  ```

- 查看容器

  docker ps -a

- 删除容器

  docker rm ID

- 容器运行

  docker run -d --name 应用自定义名称 -p 宿主机端口号:容器内部端口 镜像名称

- 查看容器运行内存占用情况

  docker stats

  

##### 基本命令

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200908214754.png" alt="image-20200908214746588" style="zoom:50%;" />



	- 查看镜像的元数据

​		docker inspect 镜像id

- 进入当前正在运行的容器

  <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200908215540.png" alt="image-20200908215531669" style="zoom:50%;" />

​				

- 从容器内拷贝文件到主机

  <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200908220007.png" alt="image-20200908215958148" style="zoom:50%;" />

  

- commit镜像

  <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200909093102.png" alt="image-20200909093055112" style="zoom:50%;" />

  <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200909092946.png" alt="image-20200909092941153" style="zoom:50%;" />





#### 进阶命令

##### 挂载数据卷

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200909110158.png" alt="image-20200909110153164" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200909110934.png" alt="image-20200909110928232" style="zoom:50%;" />



##### mysql持久化

<img src="/Users/cooper/Library/Application Support/typora-user-images/image-20200909160305077.png" alt="image-20200909160305077" style="zoom:50%;" />

​				即使我们将容器删除了，数据也会保留到本地。

##### 匿名挂载和具名挂载

​	<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200909163546.png" alt="image-20200909163541812" style="zoom:50%;" />

​	<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200909163634.png" alt="image-20200909163629399" style="zoom:50%;" />

##### 	自己编写的镜像

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200911092452.png" alt="image-20200911092446985" style="zoom:50%;" />

声明volume之后，对应的卷就会匿名挂载到主机上

##### 容器数据卷共享

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200911101457.png" alt="image-20200911094132756" style="zoom:50%;" />

只要在挂载的时候声明一下 --volumes-from + 容器名称 即可

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200911093732.png" alt="image-20200911093602185" style="zoom:50%;" />

**<font color=red>如果父容器卷被删除了，也不会影响依靠他挂载容器卷的容器</font>**



结论：

- 容器之间配置信息的传递，数据卷容器生命周期一直持续到没有容器使用为止。
- 一旦持久化到本地之后，到这个时候，本地的数据是不会被删除的！





#### DockerFiler

dockfile是用来构建docker镜像的文件，命令参数脚本。



##### 基本名词定义

- DockerFile：构建文件，定义一切的步骤和源代码

- DockerImages：通过DockerFile构建生成的镜像，最终发布和运行的产品

- Docker容器：镜像运行起来提供的服务器

  

##### 构建步骤：

- 编写一个dockerfile文件
- docker build 构建成为一个镜像
- docker run 运行镜像
- docker push 发布镜像（DockerHub，阿里云镜像仓库）



##### 基础知识

- 每个关键字必须是大写字母
- 执行顺序是从上往下
- ‘#’ 表示注释
- 每一个指令都会创建并提交一个新的镜像层，并提交

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200911102718.png" alt="image-20200911102513228" style="zoom:50%;" />



##### DockerFile指令

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200911103937.png" alt="image-20200911103540477" style="zoom:50%;" />



docker history + 镜像名称 可以查看变更历史，从而了解其dockerfile是如何编写的



编写dockerfile文件

```shell
FROM centos
MAINTAINER Cooper<1789023580@qq.com>

ENV MYPATH /user/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "---end---"
CMD /bin/bash
```



<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200911105025.png" alt="image-20200911105020615" style="zoom:50%;" />

注意：

docker build -f mydockerfile_centos -t mycentos:0.1 **<font color=red>.</font>** 最后面的小点不能忘记写，这是镜像生成到哪里的目录



##### 提交镜像

docker push



#### Docker网络

##### docker间内部网络通信原理

docker0相当于是路由器，可以转发各个容器之间的消息，而并非是两个容器之间相互直接传递消息

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200911150628.png" alt="image-20200911150622189" style="zoom:50%;" />



##### --link

作用：使用名称来访问容器

```shell
docker run --name nginx02 -p 81:80 -d --link nginx01 nginx
```

该作用的命令是将nginx01的网络地址写到nginx02的hosts文件中

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200918094641.png" alt="image-20200918094630665" style="zoom:50%;" />

**<font color=red>但是nginx01却不能ping nginx02 因为没有绑定nginx02的ip地址到ngin02上</font>**



##### 自定义网络

- 查看所有的docker网络

```shell 
docker network ls
```

- 网络模式

  - bridge : 桥接模式 docker默认的模式（就是如果两个网络之间连不通，可以通过一座桥来连接对方）
  - none： 不配置网络
  - host：和宿主机共享网络
  - container：容器网络联通（用的少，局限大）

- 创建自定义网络

  ```shell
  # dievr:网络模式
  # subnet:子网
  # gateway:网关
  docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
  ```

- 将容器启动到该网络中

  ```shell
  docker run -d -P --name tomcat-01-mynet --net mynet tomcat
  ```

- 查看网络中的容器

  ```shell
  docker network inspect mynet
  ```

  <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200918101530.png" alt="image-20200918101524772" style="zoom:50%;" />

- 容器之间相互ping

  `docker exec -it tomcat-01-mynet ping tomcat-02-mynet`

  此处并没有将其主机地址写入到对方的hosts文件中，只是因为在同一个网段下所有可以ping通

- 好处

  不同的集群使用不同的网络，可以保护集群之间的相互健康和安全



##### 网络连通

- 命令

  ```shell
  docker network connect mynet tomcat
  ```

- 原理

  <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200918125826.png" alt="image-20200918110746670" style="zoom:50%;" />

相当于将tomcat分配了一个mynet中的内网地址，tomcat也就有了两个地址，一个是外网地址，一个是内网地址



##### redis集群部署实战

1. 创建集群网络

   ```shell
   docker network create --driver bridge --subnet 172.38.0.0/16 --gateway 172.38.0.1 redis
   ```

2. 通过脚本创建六个redis配置

   ```shell
   for port in $(seq 1 6);
   do
   mkdir -p /Users/cooper/Desktop/redis/node-${port}/conf
   touch /Users/cooper/Desktop/redis/node-${port}/conf/redis.conf
   cat << EOF >> /Users/cooper/Desktop/redis/node-${port}/conf/redis.conf
   port 6379
   protected-mode no
   cluster-enabled yes
   cluster-config-file nodes.conf
   cluster-node-timeout 5000
   cluster-announce-ip 172.38.0.1${port}
   cluster-announce-bus-port 16379
   appendonly yes
   EOF
   done
   
   #启动
   for port in $(seq 63 6);
     docker run -p 6376:6379 -p 16376:16379 --name redis-6 \
     -v /Users/cooper/Desktop/redis/node-6/data:/data \
     -v /Users/cooper/Desktop/redis/node-6/conf/redis.conf:/etc/redis/redis.conf \
     -d --net redis --ip 172.38.0.16 redis redis-server /etc/redis/redis.conf;
    
   
   #配置集群
   redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
   
   #进入redis客户端
   redis-cli -c
   
   #查看集群信息
   cluster info
   cluster nodes
   
   
   #接下来停掉其中的5个主机，仍然可以查询到存储到其中的值
   ```





#### SpringBoot微服务打包Docker镜像

 1. 构建springboot项目

 2. 打包应用

 3. 编写dockerfile文件

    ```dockerfile
    FROM java:8
    COPY *.jar /app.jar
    CMD ["--server.port=8080"]
    EXPOSE 8080
    ENTRYPOINT ["java","-jar","/app.jar"]
    ```

 4. 构建镜像(在dockerfile和jar包同级目录下)

    ```shell
    docker build -t {name} .
    ```

 5. 发布运行

    

    





