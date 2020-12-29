1. 给整体项目创建pom依赖

   ```xml
   <!--    此处可以指定各个dependency的版本-->
       <properties>
           <junit-version>4.12</junit-version> 
       </properties>
   
   
       <dependencyManagement>
           <dependencies>
   <!--            springcloud的依赖-->
               <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-dependencies -->
               <dependency>
                   <groupId>org.springframework.cloud</groupId>
                   <artifactId>spring-cloud-dependencies</artifactId>
                   <version>Hoxton.SR7</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
   <!--            springboot的依赖-->
               <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-dependencies -->
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-dependencies</artifactId>
                   <version>2.3.2.RELEASE</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
   <!--            数据库-->
               <dependency>
                   <groupId>mysql</groupId>
                   <artifactId>mysql-connector-java</artifactId>
                   <version>5.1.47</version>
               </dependency>
   <!--            数据源-->
               <dependency>
                   <groupId>com.alibaba</groupId>
                   <artifactId>druid</artifactId>
                   <version>1.1.10</version>
               </dependency>
   <!--            SpringBoot Mybaits启动器-->
               <!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
               <dependency>
                   <groupId>org.mybatis.spring.boot</groupId>
                   <artifactId>mybatis-spring-boot-starter</artifactId>
                   <version>2.1.3</version>
               </dependency>
               <dependency>
                   <groupId>junit</groupId>
                   <artifactId>junit</artifactId>
                   <version>${junit-version}</version>
               </dependency>
               <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
               <dependency>
                   <groupId>org.projectlombok</groupId>
                   <artifactId>lombok</artifactId>
                   <version>1.18.12</version>
               </dependency>
               <dependency>
                   <groupId>log4j</groupId>
                   <artifactId>log4j</artifactId>
                   <version>1.2.17</version>
               </dependency>
               <!-- https://mvnrepository.com/artifact/ch.qos.logback/logback-core -->
               <dependency>
                   <groupId>ch.qos.logback</groupId>
                   <artifactId>logback-core</artifactId>
                   <version>1.2.3</version>
               </dependency>
           </dependencies>
       </dependencyManagement>
   ```

   ---

2. 创建子项目springcloud-api（该项目中只利用其创建的实体类）

   子项目结构如下

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200813143258.png" alt="image-20200813143211574" style="zoom:50%;" />

   

Dept.class

```java
//这里要继承Serializable接口，否则有可能无法传输实体
@Data
@NoArgsConstructor
@Accessors(chain = true) //链式写法  com.cooper.springcloud.pojo.Dept.setName().setDB_source.
public class Dept implements Serializable {
    private Integer deptno;
    private String dname;

    //添加数据库名称的字段是为了防止多个数据库中有相同的字段，一个服务对应一个数据库，同一个信息可能存在不同的数据库中
    private String db_source;

    public Dept(String dname) {
        this.dname = dname;
    }
}
```

数据库表如下：

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200813143627.png" alt="image-20200813143410530" style="zoom:50%;" />

---

3. 创建子项目provider（提供服务者）

   provider项目结构如下：

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200813143620.png" alt="image-20200813143615664" style="zoom:50%;" />

   1. maven依赖如下（此处因为集成了父级的maven依赖，因此无需再写版本号）：

      **<font color = red >在该依赖中导入了上一个子项目创建的实体类</font>**

      ```xml
       <dependencies>
      <!--        为了拿到实体类(即Dept)，所以需要配置api module-->
              <dependency>
                  <groupId>org.example</groupId>
                  <artifactId>springcloud-api</artifactId>
                  <version>1.0-SNAPSHOT</version>
              </dependency>
              <dependency>
                  <groupId>junit</groupId>
                  <artifactId>junit</artifactId>
                  <scope>test</scope>
              </dependency>
              <dependency>
                  <groupId>com.alibaba</groupId>
                  <artifactId>druid</artifactId>
              </dependency>
              <dependency>
                  <groupId>mysql</groupId>
                  <artifactId>mysql-connector-java</artifactId>
              </dependency>
              <dependency>
                  <groupId>ch.qos.logback</groupId>
                  <artifactId>logback-core</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.mybatis.spring.boot</groupId>
                  <artifactId>mybatis-spring-boot-starter</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-test</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-web</artifactId>
              </dependency>
      <!--        jetty-->
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-jetty</artifactId>
              </dependency>
      <!--        热部署-->
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-devtools</artifactId>
              </dependency>
          </dependencies>
      ```

   2. 配置yaml文件（包括端口号、mybatis、数据库连接等信息）

      ```yaml
      server:
        port: 8081
      mybatis:
        type-aliases-package: com.cooper.springcloud.pojo
        mapper-locations: classpath:mybatis/mapper/*.xml
      
      spring:
        application:
          name: springcloud-provider-dept
        datasource:
          type: com.alibaba.druid.pool.DruidDataSource
          driver-class-name: org.gjt.mm.mysql.Driver
          url: jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncodeing=UTF-8
          username: root
          password: 12345678
      ```

   3. 创建Dao层

      DeptDao接口

      ```java
      @Mapper
      @Repository
      public interface DeptDao {
          public boolean addDept(Dept dept);
      
          public Dept queryByID(Long id);
      
          public List<Dept> queryAll();
      }
      ```

      对应的Mapper.xml

      ```xml
      <?xml version="1.0" encoding="UTF-8" ?>
      <!DOCTYPE mapper
              PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
              "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
      
      <mapper namespace="com.cooper.springcloud.dao.DeptDao">
      
          <insert id="addDept" parameterType="Dept">
             insert into dept (dname,db_source) values (#{dname},DATABASE())
          </insert>
      
          <select id="queryByID" resultType="Dept" parameterType="Long">
             select * from dept where deptno = #{deptno};
          </select>
      
          <select id="queryAll" resultType="Dept">
             select * from dept;
          </select>
      </mapper>
      ```

   4. 创建service层

      ```java
      public interface DeptService {
          public boolean addDept(Dept dept);
      
          public Dept queryByID(Long id);
      
          public List<Dept> queryAll();
      }
      ```

      ```java
      @Service
      public class DeptServiceImpl implements DeptService{
          @Autowired
          DeptDao deptDao;
      
          public boolean addDept(Dept dept) {
              return deptDao.addDept(dept);
          }
      
          public Dept queryByID(Long id) {
              return deptDao.queryByID(id);
          }
      
          public List<Dept> queryAll() {
              return deptDao.queryAll();
          }
      }
      ```

   5. 创建controller层

      ```java
      @RestController
      public class DeptController {
          @Autowired
          DeptServiceImpl service;
      
          /*
              此处为什么要使用PostMapping？
                  因为此处是不面向用户的，所以只对应的开发人员调用，因此用户不能通过该接口添加信息，如果写成GetMapping的话，
                  用户可以直接通过该接口向数据库添加信息，造成不安全
              此处为何要添加@RequestBody？
                  将用户使用接口通过Get方法传递过来的参数包装成一个实体类，如果不添加该方法，则会出现添加的实体类的部分属性为null
           */
          @PostMapping("/dept/add")
          public boolean addDept(@RequestBody  Dept dept)
          {
              return service.addDept(dept);
          }
      
          @GetMapping("/dept/get/{id}")
          public Dept get(@PathVariable("id") Long id)
          {
              return service.queryByID(id);
          }
      
          @RequestMapping("/dept/list")
          public List<Dept> queryAll()
          {
              return service.queryAll();
          }
      }
      ```

   6. 编写启动类

      ```java
      @SpringBootApplication
      public class DeptProvider_8081 {
          public static void main(String[] args) {
              SpringApplication.run(DeptProvider_8081.class,args);
          }
      }
      ```

      ---

4. 创建consumer子项目

   consumer项目结构如下：

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200813150143.png" alt="image-20200813145339509" style="zoom:50%;" />

   1. 添加项目依赖

      ```xml
      <dependencies>
          <dependency>
              <groupId>org.example</groupId>
              <artifactId>springcloud-api</artifactId>
              <version>1.0-SNAPSHOT</version>
          </dependency>
      
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-devtools</artifactId>
          </dependency>
      </dependencies>
      ```

   2. 编写ConfigBean.class 注入 RestTemplate 用来Http通信

      ```java
      @Configuration
      public class ConfigBean {
      
          @Bean
          public RestTemplate getRestTemplate()
          {
              return new RestTemplate();
          }
      }
      ```

   3. 编写Controller层

      ```java
      @RestController
      public class ConsumerController {
          //此处是消费者 不提供服务  因此不需要service层
      
          @Autowired
          RestTemplate restTemplate;//提供多种便捷访问远程Http服务的方法，简单的Restful服务模板
      
          private static final String REST_URL_PREFIX  = "http://localhost:8081";
      
          //此处填写GetMapping是为了让用户通过url传递参数从而将数据添加到数据库中，相当于在原始的基础上多了一层封装，通过该封装传递了信息到provider
          @GetMapping(value = "consumer/dept/add")
          public boolean add(Dept dept)
          {
              return restTemplate.postForObject(REST_URL_PREFIX+"/dept/add",dept,Boolean.class);
          }
      
          @RequestMapping("consumer/dept/get/{id}")
          public Dept get(@PathVariable("id")Long id)
          {
              return restTemplate.getForObject(REST_URL_PREFIX+"/dept/get/"+id, Dept.class);
          }
      
          @RequestMapping("consumer/dept/list")
          public List<Dept> queryAll()
          {
              return restTemplate.getForObject(REST_URL_PREFIX+"/dept/list",List.class);
          }
      }
      ```

   4. 编写启动类

      ```java
      @SpringBootApplication
      public class DeptProvider_8080 {
          public static void main(String[] args) {
              SpringApplication.run(DeptProvider_8080.class,args);
          }
      }
      ```

      ---

5. 测试

   将provider和consumer项目都启动

   1. 添加测试

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200813150137.png" alt="image-20200813145722229" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200813150133.png" alt="image-20200813145751494" style="zoom:50%;" />

​		2. 查询所有信息

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200813150127.png" alt="image-20200813145836250" style="zoom:50%;" />

​        3.查询单个员工

 <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200813150120.png" alt="image-20200813145940913" style="zoom:50%;" /> 

