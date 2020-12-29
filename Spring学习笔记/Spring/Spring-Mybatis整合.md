#### Spring和Mybatis整合步骤

 1.  创建实体类（实体类中的各个属性需要与数据库中表的字段一致，否则会会出现查询为NULL的情况）

 2.  绑定数据库的连接信息并注册为dataSource

     ```xml
      <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
             <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
             <property name="url" value="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8&amp;useSSL=false"/>
             <property name="username" value="root"/>
             <property name="password" value="12345678"/>
      </bean>
     ```

	3. 注册SqlSessionFactory，这样测试时就不需要通过代码获取SqlSessionFactory

    ```xml
        <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
    <!--        配置Mybatis的相关信息-->
    <!--        <property name="configLocation" value="classpath:mapper/*.xml"/>-->
    <!--        配置mapper的位置信息-->
            <property name="mapperLocations" value="classpath:mapper/UserMapper.xml"></property>
        </bean>
    ```

    此处的mapperLocations需要注意一下：最好加上，否则会有概率报找不到mapper

    同时mapperLocations配置如下：

    **如果Mapper.xml与Mapper.class在同一个包下且同名，spring扫描Mapper.class的同时会自动扫描同名的Mapper.xml并装配到Mapper.class。**

    **如果Mapper.xml与Mapper.class不在同一个包下或者不同名，就必须使用配置mapperLocations指定mapper.xml的位置。**

    **此时spring是通过识别mapper.xml中的 <mapper namespace="com.fan.mapper.UserDao"> namespace的值来确定对应的Mapper.class的。**

	4. 注册SqlSession，测试时不需要获取sqlSession，要注意这里只能使用构造函数传入sqlSessionFactory即要填写构造函数的参数 （因为此处SqlSessionTemplate该类没有set方法，因此无法直接注入sqlSessionFactory，需要通过构造函数注入）

    ```xml
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"></constructor-arg>
    </bean>
    ```

	5. 注册mapper，

    ```xml
    <bean id="userMapper" class="mapper.UserMapperImpl">
        <property name="sqlSession" ref="sqlSession"></property>
    </bean>
    ```

6. 测试

   ```java
   UserMapper userMapper = context.getBean("userMapper", UserMapper.class);
   for(User user1 :userMapper.selectUser())
   {
       System.out.println(user1.toString());
   }
   ```

---

#### 完整代码

spring-dao.xml（此处我没有配置Mybatis的额外信息（例如别名之类的），因此没有写Mybatis.xml）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/context/spring-aop.xsd http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">

    <!--  如果想要注解配置的话需要声明这句话-->
    <context:component-scan base-package="pojo"></context:component-scan>
    <context:annotation-config/>

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=UTF-8&amp;useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="12345678"/>
    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
<!--        配置Mybatis的相关信息-->
<!--        <property name="configLocation" value="classpath:mapper/*.xml"/>-->
<!--        配置mapper的位置信息-->
        <property name="mapperLocations" value="classpath:mapper/UserMapper.xml"></property>
    </bean>

    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"></constructor-arg>
    </bean>

    <bean id="userMapper" class="mapper.UserMapperImpl">
        <property name="sqlSession" ref="sqlSession"></property>
    </bean>

</beans>
```



实体类

User

```java
@Component
@Data
public class User {

    private int id;
    @Value("Cooper")
    private String name;
    @Value("18")
    private String password;
    @Value("Cooper@163.com")
    private String email;
}
```

数据库中对应的表

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200728094933.png" alt="image-20200728094917670" style="zoom:50%;" />



UserMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--绑定接口  以及需要对数据库操作的指令-->
<mapper namespace = "mapper.UserMapper">
    <select id="selectUser" resultType="pojo.User">
        select * from test.WebUser;
    </select>
</mapper>
```

如果出现出现 Could not resolve type alias '****'异常。出现这个异常的原因有可能有两个

 1.   没有配置实体类的别名Alias,如果在mybatis的mapper中映射实体类不写包名，需要配置别名

 2.   resultMap和resultType写混了。通常这种情况会出现在select语句中。<select>标签的resultMap应该是mapper中<resultMap>的id，而resultType是一个具体的类型，也就是实体类的类名或者java基本数据类型int、long、string等。

      [resultMap和resultType的差别](https://blog.csdn.net/xushiyu1996818/article/details/89075069)

      

UserMapper

```java
public interface UserMapper {
    public List<User> selectUser();
}
```

UserMapperImpl

```java
public class UserMapperImpl implements UserMapper{

    private SqlSessionTemplate sqlSession;//不能忘记 此处sqlSession在配置文件中注入

    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public List<User> selectUser() {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.selectUser();
    }
}
```



test

```java
    UserMapper userMapper = context.getBean("userMapper", UserMapper.class);
    for(User user1 :userMapper.selectUser())
    {
        System.out.println(user1.toString());
    }
}
```



pom文件（<font color = red>mybatis有两个依赖</font>）

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.2.3.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>javax.annotation</groupId>
        <artifactId>javax.annotation-api</artifactId>
        <version>1.3.1</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.12</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.4</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.4</version>
    </dependency>

    <!-- mysql依赖 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.20</version>
    </dependency>
</dependencies>
```



##### <font color =  red>注意</font>

如果运行编译后报错

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200728095535.png" alt="image-20200728095529742" style="zoom:50%;" />

查看target中有没有UserMapper.xml等其他资源，如果没有，有可能是meaven静态资源被默认过滤掉了

需要在pom.xml中添加如下代码：

```xml
<build>
    <resources>
        <resource>
          <!--目录具体看自己的设置-->
            <directory>src/main/java</directory> 
            <includes>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

