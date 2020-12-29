#### 导入依赖

```xml
<!-- https://mvnrepository.com/artifact/com.github.theborakompanioni/thymeleaf-extras-shiro -->
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
```

---

#### 编写ShiroConfig配置类

UserRealm.class (授权和认证类)

```java
public class UserRealm extends AuthorizingRealm {
    @Autowired
    UserServiceImpl userService;

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {

        //建立授权信息
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();

        //添加授权
        Subject subject = SecurityUtils.getSubject();
        User user = (User) subject.getPrincipal();//从认证的的时候添加的principal获取User
        authorizationInfo.addStringPermission(user.getPerms());//添加
        return authorizationInfo;
    }

    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //用户名和密码  从数据库中取出
//        String username = "cooper";
//        String password = "123456";

        UsernamePasswordToken userToken = (UsernamePasswordToken) token;//获取用户登录的信息，也就是前端数据传到后台封装好的用户数据
        //连接数据库
        User user = userService.getUserByName(userToken.getUsername());

        if(user==null)//查无此人
        {
            return null;//用户名不对则抛出异常
        }

        //将user存放到shrio的session中
        Subject subject = SecurityUtils.getSubject();
        Session session = subject.getSession();
        session.setAttribute("loginUser",user);

        //密码认证是shiro自己做的，我们不参与 &&  认证的的时候添加principal的对象User
        return new SimpleAuthenticationInfo(user,user.getPassword(),"");
    }
}
```

ShiroConfig.class

```java
@Configuration
public class ShiroConfig {

    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager securityManager)
    {
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        //设置安全管理器
        bean.setSecurityManager(securityManager);

        //添加shiro内置过滤器
        /*
            anon:无需认证即可访问
            authc:必须认证了才可以访问
            user:必须拥有记住我功能才能用
            perms:拥有某个资源的权限才能访问
            role:拥有某个角色权限才能访问
         */
        Map<String,String> filterMap = new LinkedHashMap<>();

        //拦截 && 授权
        //filterMap.put("/user/*","authc");

        filterMap.put("/user/add","perms[user:add]");

        filterMap.put("/user/update","perms[user:update]");

        bean.setFilterChainDefinitionMap(filterMap);

        //定制未授权页面
        bean.setUnauthorizedUrl("/noauth");

        //设置登录的请求
        bean.setLoginUrl("/login");
        return bean;
    }

    @Bean(name = "securityManager") //此处的@Qualifier是为了使用正确的类
    public DefaultWebSecurityManager getDefaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm)
    {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        //关联UserRealm
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    //创建realm对象，需要自定义类
    @Bean
    public UserRealm userRealm()
    {
        return new UserRealm();
    }

    //整合ShiroDialect  用来整合thymeleaf和shiro
    @Bean
    public ShiroDialect getShiroDialect()
    {
        return new ShiroDialect();
    }
}
```

---

#### 编写pojo

```java
package com.cooper.pojo;

public class User {
    private String name;
    private String password;
    private String perms;

    public User() {
    }

    public User(String name, String password, String perms) {
        this.name = name;
        this.password = password;
        this.perms = perms;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getPerms() {
        return perms;
    }

    public void setPerms(String perms) {
        this.perms = perms;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", password='" + password + '\'' +
                ", perms='" + perms + '\'' +
                '}';
    }
}
```

---

#### 编写UserMapper和UserMapper.xml

```java
@Mapper
@Repository
public interface UserMapper {
    User getUserByName(String name);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.cooper.mapper.UserMapper">

    <select id="getUserByName" resultType="User" parameterType="String">
       select * from mybatis.test where name = #{name};
    </select>

</mapper>
```

---

#### 编写Service类

UserService接口

```java
public interface UserService {
    User getUserByName(String name);
}
```

实现类

```java
@Service
public class UserServiceImpl implements UserService{
    @Autowired
    UserMapper userMapper;

    @Override
    public User getUserByName(String name) {
        return userMapper.getUserByName(name);
    }
}
```

---

#### application.yaml配置文件

```yaml
server:
  port: 8080

#数据库设置
spring:
  datasource:
    username: root
    password: 12345678
    #?serverTimezone=UTC解决时区的报错
    url: jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver

#这个必须要配置  相当于之前Mybatis.xml
mybatis:
  mapper-locations: classpath:mybatis/mapper/*.xml
  type-aliases-package: com.cooper.pojo
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

---

#### 编写Controller类

```java
@Controller
public class UserController {

    @RequestMapping("/toLogin")
    public String toLogin(@RequestParam("username") String username, @RequestParam("password") String password, Model model)
    {
        //获取subject对象 也就是登录的对象
        Subject subject = SecurityUtils.getSubject();
        //封装用户登录的数据
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);

        try
        {
            subject.login(token);//执行登录方法，如果没有抛出异常则说明认证通过
            return "main";
        }catch (UnknownAccountException e)
        {
            model.addAttribute("msg","用户名不存在");
            return "login";
        }catch (IncorrectCredentialsException e)
        {
            model.addAttribute("msg","密码错误");
            return "login";
        }
    }

    @RequestMapping("/login")
    public String toLogin()
    {
        return "login";
    }

    @RequestMapping("/user/add")
    public String add()
    {
        return "add";
    }

    @RequestMapping("/user/update")
    public String update()
    {
        return "update";
    }

    @RequestMapping({"/"})
    public String main()
    {
        return "main";
    }

    @RequestMapping("/noauth")
    @ResponseBody
    public String noauth()
    {
        return "未经授权，无法访问";
    }

}
```

---

#### 编写页面

main.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:shiro="http://www.thymeleaf.org/thymeleaf-extras-shiro">
<head>
    <meta charset="UTF-8">
    <title>欢迎</title>
</head>
<body>
    <h1>首页</h1>
    <div th:if="${session.loginUser==null}">
        <a th:href="@{/login}">登录</a>
    </div>

    <div shiro:hasPermisson name="user:add">
        <a th:href="@{/user/add}">add</a>
    </div>
    <div shiro:hasPermisson name="user:update">
        <a th:href="@{/user/update}">update</a>
    </div>

</body>
</html>
```

login.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>login</title>
</head>
<body>
    <p style="color: red" th:text="${msg}"></p>
    <form th:action="@{/toLogin}" th:method="get">
        <input name="username" type="text" placeholder="username"><br>
        <input name="password" type="password" placeholder="password"><br>
        <button type="submit">登录</button>
    </form>
</body>
</html>
```

add.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>add</title>
</head>
<body>
    <h1>add</h1>
</body>
</html>
```

update.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>update</title>
</head>
<body>
    <h1>update</h1>
</body>
</html>
```

---

#### 数据库测试数据

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200811223222.png" alt="image-20200811223210497" style="zoom:50%;" />

---

#### 完成！

