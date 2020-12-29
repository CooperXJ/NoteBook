#### 本质

​	每次添加一个Servlet不需要我们再次注册到配置文件中，而是通过**DispatcherServlet**将用户的 请求分发到不同的处理器。



#### 原理

​	当发起请求时被前置的控制器拦截到请求，根据请求参数生成代理请求，找到请求对应的实际控制器，控制器处理请求，创建数据模型，访问数据库，将模型响应给中心控制器，控制器使用模型与视图渲染视图结果，将结果返回给中心控制器，再将结果返回给请求者。

​	<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200729112547.png" alt="image-20200729112540789" style="zoom:50%;" />



#### 执行原理

​	![image-20200729112619862](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200729112621.png)

**简要分析执行流程**

**<font color=red>controller是控制器，handler是控制器的处理器方法</font>**

1. DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心。用户发出请求，DispatcherServlet接收请求并拦截请求。

   我们假设请求的url为 : http://localhost:8080/SpringMVC/hello

   

   **如上url拆分成三部分：**

   http://localhost:8080服务器域名

   SpringMVC部署在服务器上的web站点

   hello表示控制器

   通过分析，如上url表示为：请求位于服务器localhost:8080上的SpringMVC站点的hello控制器。

2. HandlerMapping为处理器映射。DispatcherServlet调用HandlerMapping,HandlerMapping根据请求url查找Handler。

   ```xml
   <!--    处理器映射器  需要根据bean的名字来查找对于的Controller-->
   <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>
   ```

   通过该处理器映射器找到对应的Handler

   ```xml
   <!--Handler-->
   <bean id="/hello" class="com.cooper.controller.HelloController"></bean>
   ```

3. HandlerExecution表示具体的Handler,其主要作用是根据url查找控制器，如上url被查找控制器为：hello。将该控制器返回给DispatcherServlet

4. HandlerExecution将解析后的信息传递给DispatcherServlet,如解析控制器映射等。

5. HandlerAdapter表示处理器适配器，其按照特定的规则去执行Handler，就是去找Controller。

6. Handler让具体的Controller执行。

7. Controller将具体的执行信息返回给HandlerAdapter,如ModelAndView。

8. HandlerAdapter将视图逻辑名或模型传递给DispatcherServlet。

9. DispatcherServlet调用视图解析器(ViewResolver)来解析HandlerAdapter传递的逻辑视图名。

10. 视图解析器将解析的逻辑视图名传给DispatcherServlet。

11. DispatcherServlet根据视图解析器解析的视图结果，调用具体的视图。

12. 最终视图呈现给用户。





#### 有可能出现了配置正确并且代码没有问题但是却出现404的情况：

有可能是因为编译的时候没有将对应的包没有导入

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200729214731.png" alt="image-20200729214640186" style="zoom:50%;" />

需要在WEB-INF下创建lib包将对应的包导入到其中