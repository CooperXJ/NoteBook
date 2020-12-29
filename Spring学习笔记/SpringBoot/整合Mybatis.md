#### 步骤

**1、导入 MyBatis 所需要的依赖**  （当然你自己数据库的依赖也要导入）

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
```



**2、配置数据库连接信息（不变）**

```yaml
spring:
  datasource:
    username: root
    password: 12345678
    #?serverTimezone=UTC解决时区的报错
    url: jdbc:mysql://localhost:3306/springboot?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver

#这个必须要配置  相当于之前Mybatis.xml
mybatis:
  mapper-locations: classpath:mybatis/mapper/*.xml
  type-aliases-package: com.cooper.pojo
```



**3、测试数据库是否连接成功！**



**4、创建实体类，导入 Lombok！ ** [ 这里说明一下为何实体类无需注入](https://blog.csdn.net/STUDENTstudent123/article/details/105078108/)

Department.java

```java
package com.kuang.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Department {

    private Integer id;
    private String departmentName;

}
```



**5、创建mapper目录以及对应的 Mapper 接口**

DepartmentMapper.java

```java
//@Mapper : 表示本类是一个 MyBatis 的 Mapper
@Mapper  //此处的@Mapper如果不加的话，需要在SpringBootApplication启动类中加上@MapperScan（包名）来代替
@Repository
public interface DepartmentMapper {

    // 获取所有部门信息
    List<Department> getDepartments();

    // 通过id获得部门
    Department getDepartment(Integer id);
}
```



**6、对应的Mapper映射文件**

DepartmentMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.kuang.mapper.DepartmentMapper">

    <select id="getDepartments" resultType="Department">
       select * from department;
    </select>

    <select id="getDepartment" resultType="Department" parameterType="int">
       select * from department where id = #{id};
    </select>

</mapper>
```



**7、maven配置资源过滤问题**

```xml
<resources>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.xml</include>
        </includes>
        <filtering>true</filtering>
    </resource>
</resources>
```



**8、编写部门的 DepartmentController 进行测试！**

```java
@RestController
public class DepartmentController {
    
    @Autowired
    DepartmentMapper departmentMapper;
    
    // 查询全部部门
    @GetMapping("/getDepartments")
    public List<Department> getDepartments(){
        return departmentMapper.getDepartments();
    }

    // 查询全部部门
    @GetMapping("/getDepartment/{id}")
    public Department getDepartment(@PathVariable("id") Integer id){
        return departmentMapper.getDepartment(id);
    }
    
}
```

**启动项目访问进行测试！**



[mybatis的坑](https://www.jianshu.com/p/0e466a21185c)

