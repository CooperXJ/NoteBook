1. 安装包

   `yum install ceph-mgr-dashboard`

2. 启用模块

   `ceph mgr module enable dashboard`

   有可能需要强制启动

   `ceph mgr module enable dashboard --force`

3. 查看ceph mgr 已经挂载的模块

   `ceph mgr module ls | less`

4. 认证

   - 可以使用自签
     - `ceph dashboard create-self-signed-cert`
   - 可以使用CA证书
   - 也可以直接使用http

5. 分配端口和地址

   如果启用了证书签名的话，默认走8443端口，没有用证书签名的话，默认走8080端口

   - 地址

     `ceph config set mgr mgr/dashboard/server_addr 192.168.236.121`

   - http端口

     `ceph config set mgr mgr/dashboard/server_port 8080`

   - ssl端口

     `ceph config set mgr mgr/dashboard/ssl_server_port 8443`

6. 查看当前配置

   `ceph mgr services`

7. 创建用户

   `ceph dashboard ac-user-create {username} {password} administrator`

8. 然后登录mgr对应的网站，记住一定要是对应mgr node的网站,并且开头一定要加上<font color=red>https://</font>，直接在地址栏输入是无效的

