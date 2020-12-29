#### 安装方式

1. MiniKube

   相当于是一个模拟器，单机环境

2. 二进制安装，将编译好的二进制文件进行安装，需要设置参数，安装难度较大

3. Kubeadm 

   自动化安装工具，以镜像的方式进行部署，镜像在谷歌仓库，容易失败



#### 以Kubeadm为例

##### 安装Container runtimes

该组件是运行容器使用的，除了docker之外有多种选择

1. 设置镜像源

   设置docker-ce源 [阿里云镜像源](https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo)

   在/etc/yum.repos.d目录下

   `curl -o docker -ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`

   下载下来之后将其拷贝到另外两台节点上面去

   (有必要的话基础的源也需要指向国内，就是使用镜像的源，比如centos的源)

   

2. 安装docker（每个机器上面都要安装）

   ```shell
   sudo yum update -y && sudo yum install -y \
     containerd.io-1.2.13 \
     docker-ce-19.03.11 \
     docker-ce-cli-19.03.11
   ```

   重新docker服务并设置开机生效

   `systemctl restart docker && systemctl enable docker`

   查看docker服务启动状况

   `systemctl status docker`

   查看引擎详情(看cgroup driver是否是systemd，不是的需要进行修改)

   `docker info`

   生成docker配置文件

   ```shell
   cat <<EOF | sudo tee /etc/docker/daemon.json
   {
     "exec-opts": ["native.cgroupdriver=systemd"],
     "log-driver": "json-file",
     "log-opts": {
       "max-size": "100m"
     },
     "storage-driver": "overlay2",
     "storage-opts": [
       "overlay2.override_kernel_check=true"
     ]
   }
   EOF
   ```

   重启docker服务

   `systemctl restart docker`

   将该文件复制到其余节点上,同时重启docker 服务

   `scp /etc/docker/daemon.json {node}:/etc/docker/`

   

##### 安装kubeadm（保证机器配置最起码2G内存和2核心）

1. 将ipatbles和bridged traffic 结合起来 (每台机器都要)

   ```bash
   cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   sudo sysctl --system
   ```

   查看状态

   sysctl -a|grep net.bridge.bridge-nf

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201122210000.png" alt="image-20201122205954540" style="zoom:50%;" />

2. 放开对应的端口

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201122204655.png" alt="image-20201122204641520" style="zoom:50%;" />

   如果方便的话直接将防火墙关闭掉

   `systemctl disable firewalld && systemctl stop firewalld`

   还有selinux也需要关掉	

   

   3. 安装kubeadm（部署k8s集群）、安装kubelet（管理容器的）、安装kubectl（命令行工具）

      部署源(节点都需要布置)

      ```shell
      cat <<EOF > /etc/yum.repos.d/kubernetes.repo
      [kubernetes]
      name=Kubernetes
      baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
      enabled=1
      gpgcheck=1
      repo_gpgcheck=1
      gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
      EOF
      ```

       安装(每个节点都需要安装)

      `yum install kubeadm kubelet kubectl -y`

      <font color=red>安装完毕之后强拉kubelet服务的话是不行的，因为环境没有准备好了</font>

      将kubelet服务开机启动,否则下次开机的话无法启动k8s服务

      `systemctl enable kubelet`

      

   4. 初始化

      指定pod网络进行初始化(必须要指定16位)

      `kubeadm init --pod-network-cidr 192.168.0.0/16`

      会遇到错误，因为swap是不支持的

      ![image-20201122215155883](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201122215157.png)

      需要禁用swap

       1.  安装工具包

           `yum install util-linux`

      	2.  查看swap空间位置

          `cat /etc/fstab`

      	3. 禁用swap空间(每台机器都需要)

          `swapoff /dev/mapper/centos-swap`

          将/etc/fstab 文件中的swap注释掉，否则下次开机会自动启用swap空间

          ![image-20201122215650622](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201122215736.png)

   

   ​	再次执行``kubeadm init --pod-network-cidr 192.168.0.0/16`

   ​	但是还是会失败，因为它会自动拉取google的镜像

   ​	两种方法解决[方法](https://segmentfault.com/a/1190000038248999)

   ​	查看需要的镜像命令 `kubeadm config images list`

   ​	<font color=red>一定要版本对应</font>

   ```shell
   [root@node3 ~]# docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.19.4 k8s.gcr.io/kube-apiserver:v1.19.4
   
   [root@node3 ~]# docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.19.4 k8s.gcr.io/kube-controller-manager:v1.19.4
   
   [root@node3 ~]# docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.19.4 k8s.gcr.io/kube-scheduler:v1.19.4
   
   [root@node3 ~]# docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.19.4 k8s.gcr.io/kube-proxy:v1.19.4
   
   [root@node3 ~]# docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.4.3-0 k8s.gcr.io/etcd:3.4.3-0
   
   [root@node3 ~]# docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.6.7 k8s.gcr.io/coredns:1.6.7
   
   [root@node3 ~]# docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2 k8s.gcr.io/pause:3.2
   ```

   镜像导入成功之后再次执行`kubeadm init --pod-network-cidr 192.168.0.0/16`

   执行成功之后需要记录下信息

   ![image-20201122230410476](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201122230411.png)

   可能遇到的问题 

   查看当前节点：`kbuectl get nodes`

   The connection to the server localhost:8080 was refused - did you specify the right host or port?

   ![image-20201122230225816](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201122230227.png)

   原因：kubenetes master没有与本机绑定，集群初始化的时候没有设置

   解决办法： `export KUBECONFIG=/etc/kubernetes/admin.conf`

5. 其他节点加入集群

   查看ip_forward是否开启，确保其开启

   ![image-20201122231039260](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201122231040.png)

   输入之前成功初始化的命令

   ![image-20201122231011415](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201122231012.png)

   

   查看集群情况

   `kubectl get nodes`

6. 安装网络插件 （让宿主机之间都联系起来）

   获取yaml文件

   `curl https://docs.projectcalico.org/manifests/calico-etcd.yaml -o calico.yaml`

   执行

   `kubectl apply -f calico.yaml`

   查看集群健康状态

   `kubectl get ds n kube-system`

   `kubectl get pods -o wide -n kube-system`

   `kubectl get cs`

7. 命令补全

   安装包

   `yum install bash-completion`

   `source /usr/share/bash-completion/bash_completion`

   配置包

   `kubectl completion bash >/etc/profile.d/kubectl.sh`

   加载配置文件

   `source /etc/profile.d/kubectl.sh`

   `vim /root/.bashrc`  在里面添加上面这句话，这样的话每次开机就可以生效

   完成！



