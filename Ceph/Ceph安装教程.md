1. 设置三台linux主机并配置不同的ip地址以及内网地址

   我设置的ip网址为：192.168.236.121 192.168.236.122 192.168.236.123

   内网地址为：192.168.100.101 192.168.100.102  192.168.100.103

2. 设置三条主机名分别为node1、node2、node3

   `hostnamectl set-hostname node1`

   `bash #使得命名生效`

3. 编写hosts文件并传递到另外两个主机上

4. 免密钥登录

   1. ssh-keygen 生成公钥和私钥

   2. 将公钥派发到三个节点上

      `ssh-copy-id -i 公钥存放地址`

5. 关闭防火墙

   - 关闭selinux

     SELinux 主要作用就是**最大限度地减小系统中服务进程可访问的资源**（最小权限原则）

     - 将/etc/selinux下的config文件中的selinux修改为disabled

     <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003101658.png" alt="image-20201003085849002" style="zoom:50%;" />

     - 设置setenforce为0  （临时关闭selinux）

       `setenforce 0 `

       查看当前selinux状态

        `getenforce`

       <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003101652.png" alt="image-20201003090415905" style="zoom:50%;" />

   - 关闭防火墙

     ```bash
      systemctl disable firewalld
      systemctl stop firewalld
     ```

6. 设置ntp同步

   - 安装ntp

      yum install ntp -y

   - 如果需要将ntp的时间以内网为准的话需要设置ntp的文件，也就是设置以下部分

     这里默认是指向外网的，一般都需要指向公司的ntp服务器

     文件位于 /etc/ntp.conf

     <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003101645.png" alt="image-20201003091341145" style="zoom:50%;" />

   - 重启ntp服务

     systemctl restart ntpd

     systemctl enable ntpd

   - 查看是否启动

      ntpq -pn

   - 将另外两台服务器的ntp指向第一台服务器的ntp，并重新启动ntp服务

     在其对用的ntp.conf文件中将原来的server注释掉，然后添加对应的服务器

     sever  ip iburst

     如果出现*则说明同步完成

     <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003101635.png" alt="image-20201003092656801" style="zoom:50%;" />

7. 配置yum源

   - 配置centos的[yum源](https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11MsogJv)

     - 删除原来自带的yum源

       rm -f * 删除掉 /etc/yum.repos.d/下的yum源

     - 在阿里云镜像上找到与自己版本匹配的yum源

   - 配置epl的yum源

   - 配置ceph的yum源

     自己编写ceph.repo文件

     ```bash
     [norch]
     name=norch
     baseurl=https://mirrors.aliyun.com/ceph/rpm-nautilus/el7/noarch/
     enabled=1
     gpgcheck=0
     
     [x86_64]
     name=x86 64
     baseurl=https://mirrors.aliyun.com/ceph/rpm-nautilus/el7/x86_64/
     enabled=1
     gpgcheck=0
     ```
     
     创建缓存 yum makecache

8. 安装ceph-deploy

   yum install python-setuptools ceph-deploy

   需要确保版本是2.0.1或者是2.0.0

9. 创建monitor

   先创建ceph-deploy文件夹，进入之后执行创建monitor命令，创建ceph-deloy文件夹是想将让生成的文件放到包里面

   ceph-deploy new --public-network 192.168.236.0/24(这个必须有) --cluster-network 192.168.100.0/24 node1

   public-network用于对外，cluster-network用于对内

11. 安装依赖的包  （每个节点都需要安装）

    yum install ceph ceph-mon ceph-mgr ceph-radosgw ceph-mds -y

11. 初始化monitor (必须等待步骤10全部完成)

    ceph-deploy mon create-initial

12. 将admin的密钥推送到所有的节点上

    ceph-deploy admin node1 node2 node3

13. 部署监控节点

     ceph-deploy mgr create node1

14. 添加osd

    可以将主机中的磁盘作为osd添加到ceph集群中

    ceph-deploy osd create node1 --data /dev/sdb

    ceph-deploy osd create node2 --data /dev/sdb

    ceph-deploy osd create node3 --data /dev/sdb

15. 为了确保高可用的集群，需要部署多个monitor，ceph采用的是Paxos算法，因此需要部署奇数个mon

    ceph-deploy  mon add node2 --address 192.168.100.102

    ceph-deploy  mon add node2 --address 192.168.100.103

    也可以查看仲裁选举的情况

    ceph-deploy quorum_status --format json-pretty

16. 部署多个mgr节点，也是为了高可用，但是此处的mgr节点只有一个是active的状态，其余的都是standby状态

    ceph-deploy mgr create node2 node3

    



到此我们已经部署一个有3个mon，3个mgr和3个osd的高可用ceph集群！





### 附加

1. 如果安装过程中遇到问题

   - 删除安装包 

     ceph-deploy purge admin-node node1 node2 node3

   - 清除配置 

     ceph-deploy purgedata admin-node node1 node2 node3

   ​	   ceph-deploy forgetkeys

2. 如果osd安装不了 查看磁盘是否清理干净

   https://www.jianshu.com/p/c88ddc104a4a

   vgscan 查看

   

