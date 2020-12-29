在Web中/是一种相对路径

- 如果被浏览器解析 得到的地址是http://ip:prot/

- 如果被服务器解析  得到的地址是http://ip:port/工程路径

  - ```xml
    <url-pattern>/servlet</url-pattern>
    ```

  - ```java
    servletContext.getRealPath("/")
    ```

  - ```java
    request.getRequestDispatcher("/")
    ```

- **<font color = red>特殊情况</font>**

  - response.sendRedirect("/") 把/发送给浏览器解析，得到http://ip:port/

