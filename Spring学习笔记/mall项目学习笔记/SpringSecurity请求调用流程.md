测试环境

Swagger2 2.7.0 springboot 2.3.x



测试类

```java
JwtAuthenticationTokenFilter extends OncePerRequestFilter
```

该类继承OncePerRequestFilter，表明不管是什么请求都会被该Filter过滤一次，注意是不管什么请求都会被过滤一次！！



1. 经过JwtAuthenticationTokenFilter类
2. 进入到![image-20201110202831683](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201110202833.png)

3. doFilter 方法就是 Spring Security 中过滤器挨个执行的过程，如果 `currentPosition == size`，表示过滤器链已经执行完毕，此时通过调用 originalChain.doFilter 进入到原生过滤链方法中，同时也退出了 Spring Security 过滤器链。否则就从 additionalFilters 取出 Spring Security 过滤器链中的一个个过滤器，挨个调用 doFilter 方法

   ![image-20201110204414460](/Users/cooper/Library/Application Support/typora-user-images/image-20201110204414460.png)

4. 到`currentPosition == size`时会跳转到

   <img src="/Users/cooper/Library/Application Support/typora-user-images/image-20201110223532345.png" alt="image-20201110223532345" style="zoom:50%;" />

   通过递归的方法调用剩下的filters

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201110225158.png" alt="image-20201110225152059" style="zoom:50%;" />

5. 当所有的filters都过滤完之后会调用该方法，也就是需要调用servlet

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201110225343.png" alt="image-20201110225336821" style="zoom:50%;" />

   ![image-20201110225607098](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201110225609.png)



6. 调用dispatch分配url请求

   ![image-20201110225823170](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201110225825.png)

7. 获取handler调用对应的Interceptor

   获取handler

   ![Hello](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201110225925.png)

   调用对应的Interceptor

   ![image-20201110230238535](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201110230240.png)



从中我们看到大致的流程：

请求发生->过滤器1（按照一定的顺序进行过滤并且不同的请求类型过滤的过程也不一样，一般都是11个过滤器）->过滤器2（当过滤器1都过滤完之后会有过滤器2，一般就是originalChain.doFilter ，请求进入到原生过滤链方法中）->经过dispatcher转发请求->请求对应的handler->请求对应的Interceptor



