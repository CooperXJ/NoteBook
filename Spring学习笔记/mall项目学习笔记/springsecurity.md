1. 动态加盐

   密码每次都是被动态加盐记录到数据库中的，也就是说不可能出现一个密码被加密之后是相同的情况。

   springsecurity会每次自动记录下加盐的位置，以便解密。

2. 对用户的账号判断是否失效、密码是否失效、用户账号是否被锁

   ![image-20201114195304582](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201114195312.png)

3. 如果开启了csrf保护

   那么需要注意一下在logout和login页面都需要使用post方式，get方式不得行

4. 在使用remember-me之后会自动生成一个token，但是该token默认是存储在cookie中的，很容易被盗取，因此需要将其持久化。

5. 关于角色写法问题

   ![image-20201116155655992](/Users/cooper/Library/Application Support/typora-user-images/image-20201116155655992.png)
   
   ROLE_XXX 其中的XXX部分必须要<font color=red>大写</font>
   
6. token持久化时候所用的表只有这么多的字段，多一个少一个都不行

   ```java
   @Data
   @Entity
   @Table(name = "persistent_logins")
   @ToString
   public class PersistentLogin {
       @Id
       private String username;
       private String series;
       private String token;
       private Date last_used;
   }
   ```

7. springmvc中的子容器和父容器

   ![image-20201116192925008](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201116192926.png)

   注解一定要放到对应的容器中去。子容器的注解支持只能在子容器中生效无法在父容器中生效。父容器的注解支持也仅仅能够在父容器中生效，在子容器中无法生效。



8. springsecurity中使用@ControllerAdviced的注意事项   [参考](https://blog.csdn.net/qq_39986681/article/details/107760997)

   <font color=red>RestControllerAdvice是处理全局异常的！！！！</font>

   <font color=red>过滤器中产生的异常不是全局异常都是无法接收到的！！！</font>

   在使用该注解的时候需要考虑到springsecurity中的ExceptionTranslationFilter对exception作出了处理

   <img src="/Users/cooper/Library/Application Support/typora-user-images/image-20201117100911575.png" alt="image-20201117100911575" style="zoom:50%;" />

   因此我们无法在@ControllerAdviced注解的类中得到异常，因为在filter中已经被拦截了。

   我们追踪源码可以看到异常已经被处理了，以AccessDeniledException为例

   <img src="/Users/cooper/Library/Application Support/typora-user-images/image-20201117102445529.png" alt="image-20201117102445529" style="zoom:50%;" />



​		如果想要解决该问题必须将异常从Filter中抛到ControllerAdvcied层

​		新建一个ErrorControllerImpl 实现ErrorController 把ErrorPath 指向`error` 再写一个方法把Error抛出 然后Controller全局统一异常处理RestControllerAdvice就能捕获到异常了

```java
@Controller
public class ErrorControllerImpl implements ErrorController {

    @Override
    public String getErrorPath() {
        return "/error";
    }

    @RequestMapping("/error")
    public void handleError(HttpServletResponse response) throws Throwable {
        if(response.getStatus()==403){
            throw new AccessDeniedException("权限不足。。。");
        }
        else if(response.getStatus()==404){
            throw new NotFoundException("未找到该页面。。。。");
        }else
        {
            throw new RuntimeException("服务器出错");
        }
    }
}
```

这样之后就可以得到处理对应的异常。

**原因**：

springmvc提供全局异常处理器（一个系统只有一个异常处理器）进行统一异常处理，@ControllerAdvice只能捕获全局异常

全局异常一般由系统的dao、service、controller产生，通过throws Exception向上抛出，最后由springmvc前端控制器交由异常处理器进行异常处理，但是springsecurity产生的并异常并非是全局异常自然无法进行处理，这些都是过滤器里面产生的异常，自然无法进行捕获。



实际上当springboot项目出现异常时，默认会跳转到/error，而/error则是由BasicErrorController进行处理

当我们编写了其实现类之后就会默认转发异常到Controller层，这样返回给前端控制器的时候就会抛出一个全局异常，ControllerAdviced就会获取到该异常。







