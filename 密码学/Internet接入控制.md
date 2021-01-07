1. 接入过程

   - 建立终端A与接入控制设备之间的传输路径
   - 接入控制设备完成身份鉴别过程
   - 动态配置终端A的网络信息
   - 动态创建终端A的网络路由项

2. 控制接入协议

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20210106202245.png" alt="image-20210106202241082" style="zoom:50%;" />

3. 扩展鉴别协议EAP产生背景

   由于各种链路层协议需要支持每一中鉴别协议，导致现阶段这种解决方案越来越复杂。

   EAP是一种与应用环境无关的，用户传输鉴别协议消息的载体协议，所有应用环境与鉴别协议都和这种载体协议绑定。

   作用：它的出现使得链路层载体协议与鉴别协议机制得以解耦。

   

   EAP报文格式 ：

   

   <img src="/Users/cooper/Library/Application Support/typora-user-images/image-20210106212050316.png" alt="image-20210106212050316" style="zoom:50%;" />

   在标识符字段中需要注意：

   要求两次相邻请求的报文中的标识符不一样，用来应对重放攻击。

4. 802.1X鉴别接入用户身份过程

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20210106212917.png" alt="image-20210106212912871" style="zoom:50%;" />

5. 以太网接入控制过程
   1. 当某个用户希望接入Internet时，通过发送EAPOL-Start发起鉴别过程
   2. 一旦成功完成用户身份鉴别，交换机将该终端的MAC地址列入接收到EAP报文端口对应的访问控制列表并将访问控制设置为允许访问。