### Linux环境下安装RabbitMQ

##### 下载对应的包（3个包）

 1. erlang

    下载的版本要与RabbitMQ对应*https://www.rabbitmq.com/which-erlang.html*

    [erlang版本](https://dl.bintray.com/rabbitmq-erlang/rpm/erlang/22/el/7/x86_64/)

	2. RabbitMQ

    https://www.rabbitmq.com/download.html

	3. 依赖包

    rabbitmq安装依赖于socat，所以需要下载socat。

　　socat下载地址：http://repo.iotti.biz/CentOS/6/x86_64/[socat-1.7.3.2-1.el6.lux.x86_64.rpm](http://repo.iotti.biz/CentOS/6/x86_64/socat-1.7.3.2-1.el6.lux.x86_64.rpm)

　　根据自身需求下载对应系统socat依赖：(http://repo.iotti.biz/CentOS/)



##### 安装包（一定要注意安装顺序）

**一定按照以下顺序安装**：

　　① rpm -ivh erlang-22.2.4-1.el7.x86_64.rpm 

　　②rpm -ivh socat-1.7.3.2-1.el6.lux.x86_64.rpm

　　③ rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm 

[如果安装版本失败了，则可以卸载重新安装](https://www.cnblogs.com/qinghuaL/p/11597695.html)



##### 配置Rabbit.config文件

- 有可能在/usr/share/doc/rabbitmq-server-3.8.8/目录下有Rabbit.config的模板

- 找到后修改第61行

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200907213456.png" alt="image-20200907213450706" style="zoom:50%;" />

​		%%去除，并将其后的“,”去除

- 将其复制到 /etc/rabbitmq/目录下

  如果没有找到，则自己复制一份config（https://blog.csdn.net/xiaoming1563/article/details/82385486）



##### 安装RabbitMQ的WEB插件

`rabbitmq-plugins enable rabbitmq_management`



##### 如果无法通过guest进行访问

新增用户进行访问即可

- 查询用户

  ```bash
  rabbitmqctl list_users
  ```

- 新增用户

  ```bash
  rabbitmqctl add_user 'username' 'password'
  ```

- 授权（虚拟机授权）

  ```bash
  rabbitmqctl set_permissions -p "/" "username" ".*" ".*" ".*"
  ```

- 设置用户角色（必须设置，否则无法登陆）

  ```bash 
  rabbitmqctl set_user_tags username administrator
  ```



##### 访问端口

http://ip:15672





​		