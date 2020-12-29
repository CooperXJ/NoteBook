#### 整体结构图

​	<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200802220228.png" alt="image-20200802213710048" style="zoom:50%;" />



1. 编写实体类

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class Books {
       private int bookID;
       private String bookName;
       private int bookCounts;
       private String detail;
   }
   ```

2. 连接Mybatis  **<font color=red>（我们没有编写Mapper的实现类，因为在配置文件中直接动态注入到了spring的容器中）</font>**

   1. 创建数据库的相关配置文件

      ```properties
      jdbc.driver=com.mysql.jdbc.Driver
      jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=false&useUnicode=true&characterEncoding=utf8
      jdbc.username=root
      jdbc.password=12345678
      ```

   2. 编写Mybatis的核心配置文件

      ```xml
      <?xml version="1.0" encoding="UTF-8" ?>
      <!DOCTYPE configuration
              PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
              "http://mybatis.org/dtd/mybatis-3-config.dtd">
      <configuration>
      
      <!--    这个必须要加上去，否则 mapper.xml 中的 parameterType="Books"会找不到Books在哪里-->
          <typeAliases>
              <package name="com.cooper.books.pojo"/>
          </typeAliases>
          
      <!--    说明Mapper的位置-->
          <mappers>
              <mapper resource="com/cooper/books/dao/BookMapper.xml"/>
          </mappers>
      
      </configuration>
      ```

   3. 编写Mapper接口

      ```java
      package com.cooper.books.dao;
      
      import com.cooper.books.pojo.Books;
      
      import java.util.List;
      
      public interface BookMapper {
          //增加一个Book
          int addBook(Books book);
      
          //根据id删除一个Book
          int deleteBookById(int id);
      
          //更新Book
          int updateBook(Books books);
      
          //根据id查询,返回一个Book
          Books queryBookById(int id);
      
          //查询全部Book,返回list集合
          List<Books> queryAllBook();
      }
      ```

   4. 编写Mapper接口同名的xml文件

      ```xml
      <?xml version="1.0" encoding="UTF-8" ?>
      <!DOCTYPE mapper
              PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
              "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
      
      <mapper namespace="com.cooper.books.dao.BookMapper">
      
          <!--增加一个Book-->
          <insert id="addBook" parameterType="Books">
            insert into ssmbuild.books(bookName,bookCounts,detail)
            values (#{bookName}, #{bookCounts}, #{detail})
         </insert>
      
          <!--根据id删除一个Book-->
          <delete id="deleteBookById" parameterType="int">
            delete from ssmbuild.books where bookID=#{bookID}
         </delete>
      
          <!--更新Book-->
          <update id="updateBook" parameterType="Books">
            update ssmbuild.books
            set bookName = #{bookName},bookCounts = #{bookCounts},detail = #{detail}
            where bookID = #{bookID}
         </update>
      
          <!--根据id查询,返回一个Book-->
          <select id="queryBookById" resultType="Books">
            select * from ssmbuild.books
            where bookID = #{bookID}
         </select>
      
          <!--查询全部Book-->
          <select id="queryAllBook" resultType="Books">
            SELECT * from ssmbuild.books
         </select>
      
      </mapper>
      ```

   5. 编写spring-dao.xml，整合spring和mybatis 

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:context="http://www.springframework.org/schema/context"
             xsi:schemaLocation="http://www.springframework.org/schema/beans
             http://www.springframework.org/schema/beans/spring-beans.xsd
             http://www.springframework.org/schema/context
             https://www.springframework.org/schema/context/spring-context.xsd">
      
          <!-- 配置整合mybatis -->
          <!-- 1.关联数据库文件 -->
          <context:property-placeholder location="classpath:database.properties"/>
      
          <!-- 2.数据库连接池 -->
          <!--数据库连接池
              dbcp 半自动化操作 不能自动连接
              c3p0 自动化操作（自动的加载配置文件 并且设置到对象里面）
          -->
          <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
              <!-- 配置连接池属性 -->
              <property name="driverClass" value="${jdbc.driver}"/>
              <property name="jdbcUrl" value="${jdbc.url}"/>
              <property name="user" value="${jdbc.username}"/>
              <property name="password" value="${jdbc.password}"/>
      
              <!-- c3p0连接池的私有属性 -->
              <property name="maxPoolSize" value="30"/>
              <property name="minPoolSize" value="10"/>
              <!-- 关闭连接后不自动commit -->
              <property name="autoCommitOnClose" value="false"/>
              <!-- 获取连接超时时间 -->
              <property name="checkoutTimeout" value="10000"/>
              <!-- 当获取连接失败重试次数 -->
              <property name="acquireRetryAttempts" value="2"/>
          </bean>
      
          <!-- 3.配置SqlSessionFactory对象 -->
          <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
              <!-- 注入数据库连接池 -->
              <property name="dataSource" ref="dataSource"/>
              <!-- 配置MyBaties全局配置文件:mybatis-config.xml -->
              <property name="configLocation" value="classpath:mybatis-config.xml"/>
          </bean>
      
          <!-- 4.配置扫描Dao接口包，动态实现Dao接口注入到spring容器中 这样就可以不需要编写BookMapperImpl-->
          <!--解释 ：https://www.cnblogs.com/jpfss/p/7799806.html-->
          <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
              <!-- 注入sqlSessionFactory -->
              <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
              <!-- 给出需要扫描Dao接口包 -->
              <property name="basePackage" value="com.cooper.books.dao"/>
          </bean>
      
      </beans>
      ```

3. 编写Service相关文件

   1. 编写Service接口

      ```java
      package com.cooper.books.service;
      
      import com.cooper.books.pojo.Books;
      
      import java.util.List;
      
      public interface BookService {
          //增加一个Book
          int addBook(Books book);
          //根据id删除一个Book
          int deleteBookById(int id);
          //更新Book
          int updateBook(Books books);
          //根据id查询,返回一个Book
          Books queryBookById(int id);
          //查询全部Book,返回list集合
          List<Books> queryAllBook();
      }
      ```

   2. 编写接口实现类

      ```java
      package com.cooper.books.service;
      
      import com.cooper.books.dao.BookMapper;
      import com.cooper.books.pojo.Books;
      
      import java.util.List;
      
      public class BookServiceImpl implements BookService{
          //调用dao层的操作，设置一个set接口，方便Spring管理
          private BookMapper bookMapper;
      
          public void setBookMapper(BookMapper bookMapper) {
              this.bookMapper = bookMapper;
          }
      
          public int addBook(Books book) {
              return bookMapper.addBook(book);
          }
      
          public int deleteBookById(int id) {
              return bookMapper.deleteBookById(id);
          }
      
          public int updateBook(Books books) {
              return bookMapper.updateBook(books);
          }
      
          public Books queryBookById(int id) {
              return bookMapper.queryBookById(id);
          }
      
          public List<Books> queryAllBook() {
              return bookMapper.queryAllBook();
          }
      }
      ```

   3. 编写spring-service.xml，整合spring和service

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:context="http://www.springframework.org/schema/context"
             xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd">
      
          <!-- 扫描service相关的bean -->
          <context:component-scan base-package="com.cooper.books.service" />
      
          <!--BookServiceImpl注入到IOC容器中-->
          <bean id="BookServiceImpl" class="com.cooper.books.service.BookServiceImpl">
              <property name="bookMapper" ref="bookMapper"/> <!--此处的引用在spring-dao里面-->
          </bean>
      
          <!-- 配置事务管理器 -->
          <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
              <!-- 注入数据库连接池 -->
              <property name="dataSource" ref="dataSource" />
          </bean>
      
      </beans>
      ```

4. 整合SpringMVC

   1. 编写spring-mvc.xml

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:context="http://www.springframework.org/schema/context"
             xmlns:mvc="http://www.springframework.org/schema/mvc"
             xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd
         http://www.springframework.org/schema/mvc
         https://www.springframework.org/schema/mvc/spring-mvc.xsd">
      
          <!-- 配置SpringMVC -->
          <!-- 1.开启SpringMVC注解驱动 -->
          <mvc:annotation-driven />
          <!-- 2.静态资源默认servlet配置-->
          <mvc:default-servlet-handler/>
      
          <!-- 3.配置jsp 显示ViewResolver视图解析器 -->
          <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
              <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
              <property name="prefix" value="/WEB-INF/jsp/" />
              <property name="suffix" value=".jsp" />
          </bean>
      
          <!-- 4.扫描web相关的bean -->
          <context:component-scan base-package="com.cooper.books.controller" />
      
      </beans>
      ```

   2. 在web.xml中注册DispatcherServlet以及乱码过滤器

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
               version="4.0">
          <!--DispatcherServlet-->
          <servlet>
              <servlet-name>DispatcherServlet</servlet-name>
              <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
              <init-param>
                  <param-name>contextConfigLocation</param-name>
                  <!--一定要注意:我们这里加载的是总的配置文件，之前被这里坑了！-->
                  <param-value>classpath:applicationContext.xml</param-value>
              </init-param>
              <load-on-startup>1</load-on-startup>
          </servlet>
          <servlet-mapping>
              <servlet-name>DispatcherServlet</servlet-name>
              <url-pattern>/</url-pattern>
          </servlet-mapping>
      
          <!--encodingFilter-->
          <filter>
              <filter-name>encodingFilter</filter-name>
              <filter-class>
                  org.springframework.web.filter.CharacterEncodingFilter
              </filter-class>
              <init-param>
                  <param-name>encoding</param-name>
                  <param-value>utf-8</param-value>
              </init-param>
          </filter>
          <filter-mapping>
              <filter-name>encodingFilter</filter-name>
              <url-pattern>/*</url-pattern>
          </filter-mapping>
      
          <absolute-ordering />
      
          <!--Session过期时间-->
          <session-config>
              <session-timeout>15</session-timeout>
          </session-config>
      </web-app>
      ```

   3. 编写Controller类

      ```java
      package com.cooper.books.controller;
      
      import com.cooper.books.pojo.Books;
      import com.cooper.books.service.BookService;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.beans.factory.annotation.Qualifier;
      import org.springframework.stereotype.Controller;
      import org.springframework.ui.Model;
      import org.springframework.web.bind.annotation.RequestMapping;
      
      import javax.servlet.http.HttpServlet;
      import javax.servlet.http.HttpServletRequest;
      import javax.servlet.http.HttpServletResponse;
      import java.net.http.HttpResponse;
      import java.util.List;
      
      @Controller
      @RequestMapping("/book")
      public class BookController {
      
          @Autowired
          @Qualifier("BookServiceImpl")
          private BookService bookService;
      
          @RequestMapping("/allBook")
          public String list(Model model, HttpServletRequest  request, HttpServletResponse response) {
              List<Books> list = bookService.queryAllBook();
              model.addAttribute("list", list);
      
              for(Books books:list)
                  System.out.println(books.getBookName());
              return "allBook";
          }
      }
      ```

5. 编写applicationContext.xml  整合所有的配置文件

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <import resource="spring-dao.xml"/>
       <import resource="spring-service.xml"/>
       <import resource="spring-mvc.xml"/>
   
   </beans>
   ```

6. 编写网页  allBook.jsp

   ```jsp
   <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
   <%@ page contentType="text/html;charset=UTF-8" language="java" %>
   <html>
   <head>
       <title>书籍列表</title>
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <!-- 引入 Bootstrap -->
       <link href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
   </head>
   <body>
   
   <div class="container">
   
       <div class="row clearfix">
           <div class="col-md-12 column">
               <div class="page-header">
                   <h1>
                       <small>书籍列表 —— 显示所有书籍</small>
                   </h1>
               </div>
           </div>
       </div>
   
       <div class="row">
           <div class="col-md-4 column">
               <a class="btn btn-primary" href="${pageContext.request.contextPath}/book/toAddBook">新增</a>
           </div>
       </div>
   
       <div class="row clearfix">
           <div class="col-md-12 column">
               <table class="table table-hover table-striped">
                   <thead>
                   <tr>
                       <th>书籍编号</th>
                       <th>书籍名字</th>
                       <th>书籍数量</th>
                       <th>书籍详情</th>
                       <th>操作</th>
                   </tr>
                   </thead>
   
                   <tbody>
                   <c:forEach var="book" items="${requestScope.get('list')}">
                       <tr>
                           <td>${book.getBookID()}</td>
                           <td>${book.getBookName()}</td>
                           <td>${book.getBookCounts()}</td>
                           <td>${book.getDetail()}</td>
                           <td>
                               <a href="${pageContext.request.contextPath}/book/toUpdateBook?id=${book.getBookID()}">更改</a> |
                               <a href="${pageContext.request.contextPath}/book/del/${book.getBookID()}">删除</a>
                           </td>
                       </tr>
                   </c:forEach>
                   </tbody>
               </table>
           </div>
       </div>
   </div>
   ```

   编写index.jsp

   ```jsp
   <%--
     Created by IntelliJ IDEA.
     User: xuejin
     Date: 2020/8/2
     Time: 上午10:18
     To change this template use File | Settings | File Templates.
   --%>
   <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
   <!DOCTYPE HTML>
   <html>
   <head>
     <title>首页</title>
     <style type="text/css">
       a {
         text-decoration: none;
         color: black;
         font-size: 18px;
       }
       h3 {
         width: 180px;
         height: 38px;
         margin: 100px auto;
         text-align: center;
         line-height: 38px;
         background: deepskyblue;
         border-radius: 4px;
       }
     </style>
   </head>
   <body>
   
   <h3>
     <a href="${pageContext.request.contextPath}/book/allBook">点击进入列表页</a>
   </h3>
   </body>
   </html>
   ```

   

