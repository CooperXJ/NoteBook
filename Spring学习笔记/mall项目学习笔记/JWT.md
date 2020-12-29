#### JWT可以避免csrf攻击

##### csrf攻击原理

<img src="/Users/cooper/Library/Application Support/typora-user-images/image-20200925094805513.png" alt="image-20200925094805513" style="zoom:50%;" />

##### jwt避免crsf攻击原理

前端使用JS将*JWT*放在header中手动发送给服务端,服务端验证header中的*JWT*字段,而非cookie信息,这样就避免了*CSRF*漏洞攻击



