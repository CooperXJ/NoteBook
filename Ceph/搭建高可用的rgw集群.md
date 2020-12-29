#### 安装keepalived（集群中的所有主机都安装）

​	keepalived: 一种高性能的服务器高可用或热备解决方案

1. `yum install keepalived -y`

2. - 修改配置文件（master节点）

   ```bash
   ! Configuration File for keepalived
   
   global_defs {
      notification_email {
        acassen@firewall.loc
        failover@firewall.loc
        sysadmin@firewall.loc
      }
      notification_email_from Alexandre.Cassen@firewall.loc
      smtp_server 192.168.200.1
      smtp_connect_timeout 30
      router_id LVS_DEVEL
      vrrp_skip_check_adv_addr
    #  vrrp_strict  #此处需要修改，否则可以会出现端口无法访问的情况
      vrrp_garp_interval 0
      vrrp_gna_interval 0
   }
   
   # 切换脚本  如果haproxy服务掉线，会将该主机的权重-2，使得backup成为master
   vrrp_script chk_haproxy {
   	script "killall -0 haproxy"
   	interval 2
   	weight -2
   }
   
   vrrp_instance RGW {
       state MASTER
       interface ens37 #vip配置的网卡  这个必须要真实存在  （也就是从现有的网卡中挑选一块）
       virtual_router_id 51
       priority 100
       advert_int 1
       authentication {
           auth_type PASS
           auth_pass 1111
       }
       virtual_ipaddress {
           192.168.100.104/24 #vip地址，虚拟ip地址，配置之后自动在interface选择的网卡中生成
       }
       track_script {
   				chk_haproxy #脚本
       }
   }
   
   ```

   - 备用节点其他都一样，就是将 state MASTER字段改为state BACKUP

3. 启动服务

   `systemctl restart keepalived`

4. 查看日志(报错正常，因为我们haproxy未启动)

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201004202056.png" alt="image-20201004202051674" style="zoom:50%;" />

#### 安装haproxy（集群中的主机都安装）

haproxy：提供高可用性、负载均衡以及基于TCP和HTTP应用的代理，支持虚拟主机，它是免费、快速并且可靠的一种解决方案

1. `yum install haproxy -y`

2. 修改配置文件(主要修改frontend和backend内容)

   ![image-20201004202206105](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201004202211.png)

   frontend我是和ceph中的rgw集成，因此使用htpp协议，代理端口设置为80。

   backend是设置rgw的ip地址和暴露的端口号（rgw默认端口为7480，在ceph.conf中我修改为了81）

   这样就完成了代理。

3. 启动haproxy服务

   `systemctl restart haproxy`

4. 查看状态

   `systemctl status haproxy`

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201004202734.png" alt="image-20201004202730060" style="zoom:50%;" />



#### 检查

1. 我们可以看到虚拟ip已经添加到ens37网卡上了

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201004204530.png" alt="image-20201004203856577" style="zoom:50%;" />

2. s3客户端也可以进行访问

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201004204525.png" alt="image-20201004204015207" style="zoom:50%;" />

3. 如果我们此时停止node1(Keepalived初始的master节点)的haproxy服务

   `systemctl stop haproxy`

   我们可以看到当前node1的ens37中已经没有192.168.100.104(vip)了

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201004204520.png" alt="image-20201004204351389" style="zoom:50%;" />

   node2我们可以看到vip已经转移到此节点

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201004205008.png" alt="image-20201004204455435" style="zoom:50%;" />

   至此完成了vip漂移

4. 如果配置了s3客户端，我们需要对其配置文件进行修改，使其指向我们配置的keepalived配置的vip

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201004205548.png" alt="image-20201004205543347" style="zoom:50%;" />



#### 完成高可用rgw集群的搭建

