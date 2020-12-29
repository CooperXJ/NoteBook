1. 加载模块

   `ceph mgr module enable prometheus`

   Ceph Luminous 12.2.1的mgr中<font color=red>自带</font>了Prometheus插件，内置了 Prometheus ceph exporter，可以使用Ceph mgr内置的exporter作为Prometheus的target。

2. 默认暴露的端口是9283

   地址：ip:9283

   里面有非常多的指标

   当然端口也是可以修改的

3. 安装Prometheus

   编写yum源

   安装

   `yum install prometheus -y`

4. 修改Prometheus的配置文件并启动

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201007201106.png" alt="image-20201007200927996" style="zoom:50%;" />

5. 打开宿主机的网址以及端口号

   {ip}:9090

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201007201045.png" alt="image-20201007201040079" style="zoom:50%;" />

6. 安装grafana并启动

   `yum install https://mirrors.cloud.tencent.com/grafana/yum/el7/grafana-5.4.2-1.x86_64.rpm `

   `service grafana-server start`

7. 查看grafana端口(默认是3000端口)

   `netstat -antupl |grep grafana`

8. 登录的默认账户和密码都是admin

9. 配置datasource

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201007202503.png" alt="image-20201007202456062" style="zoom:50%;" />

10. 因为下载的时候grafana就已经提供了一些页面脚本共选择

    在 /etc/grafana/dashboards/下有较多的json文件

    <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201007213637.png" alt="image-20201007203436859" style="zoom:50%;" />

    可以直接导入到grafana中

    或者直接输入UID（也就是官方提供的一些模板）