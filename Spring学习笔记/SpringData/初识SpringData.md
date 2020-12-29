#### SpringData定义

​		Spring Data JPA是一种JPA的抽象层，底层依赖Hibernate，也就是给我们直接提供了访问数据库的能力，比如常见的CRUD操作，复杂的操作还是需要我们自己编写。

​		可以访问关系型数据库也可以访问NoSQL类型数据库



**以关系型数据库为例**

---

#### HelloWorld

##### 	1. 导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

##### 	2. 配置yaml文件（因为底层是hibernate实现的，所以需要声明一下）

- ddl-auto:create----每次运行该程序，没有表格会新建表格，表内有数据会清空

- ddl-auto:create-drop----每次程序结束的时候会清空表

- ddl-auto:update----每次运行程序，没有表格会新建表格，表内有数据不会清空，只会更新(此次的选择)

- ddl-auto:validate----运行程序会校验数据与数据库的字段类型是否相同，不同会报错

```yaml
server:
  port: 8081

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai&useSSL=false
    username: root
    password: 12345678

  jpa:
    hibernate:
      ddl-auto: update
      naming:
        physical-strategy: org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy # 遇到大写加_
    show-sql: true
```

#####   	3. 创建实体类

```java
package com.cooper.pojo;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

@Table(name = "user")//如果没有表则会创建表 
@Entity
public class User {
    @GeneratedValue //自动生成
    @Id  //指定主键
    private Integer id;
    private String name;
    private Integer age;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

#####  	4. 继承Repository接口

​		1. Repository接口是一个空的标记接口，我们定义的接口如果继承了该接口，则该接口会被IOC容器识别为一个Repository Bean纳入到IOC容器中，进而可以在该接口中定义**满足一定规范**的方法

​        2. 可以通过注解替代继承接口 	`@RepositoryDefinition(domainClass = User.class,idClass = Integer.class)`

Repository<User,Integer>  其中**User是处理的实体类的类型，Integer为主键类型**

```java
public interface PersonRepository extends Repository<User,Integer> , PersonDao{
    User getUserByName(String name);
}
```

##### 	5. 测试

```java
@Test
void testRepository()
{
    User user = repository.getUserByName("Cooper");
    System.out.println(user);
}
```



#### Repository方法规范

1. 需要符合一定规范

2. 查询方法以find | read | get开头

3. 涉及条件查询时，条件的属性用条件关键词连接

4. 条件属性以首字母大写开头

5. 支持属性的级联查询，若当前类有符合条件的属性，则优先使用，而不使用级联属性。

   如果需要使用级联属性，则属性之间使用 “_” 进行连接

   

   级联属性例子

   User类

   ```java
   package com.cooper.pojo;
   
   import javax.persistence.*;
   
   @Table(name = "user")
   @Entity
   public class User {
       @GeneratedValue
       @Id
       private Integer id;
       private String name;
       private Integer age;
   
       @JoinColumn(name = "ADDRESS_ID")
       @ManyToOne
       private Address address;
       
   
       public int getId() {
           return id;
       }
   
       public void setId(int id) {
           this.id = id;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public int getAge() {
           return age;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
   
       public Address getAddress() {
           return address;
       }
   
       public void setAddress(Address address) {
           this.address = address;
       }
       
   
       @Override
       public String toString() {
           return "User{" +
                   "id=" + id +
                   ", name='" + name + '\'' +
                   ", age=" + age +
                   '}';
       }
   }
   ```

   Address类

   ```java
   package com.cooper.pojo;
   
   import javax.persistence.Entity;
   import javax.persistence.GeneratedValue;
   import javax.persistence.Id;
   import javax.persistence.Table;
   
   @Table(name = "address")
   @Entity
   public class Address {
       @GeneratedValue
       @Id
       private Integer id;
       private String addressName;
   
       public Integer getId() {
           return id;
       }
   
       public void setId(Integer id) {
           this.id = id;
       }
   
       public String getAddressName() {
           return addressName;
       }
   
       public void setAddressName(String addressName) {
           this.addressName = addressName;
       }
   
       @Override
       public String toString() {
           return "Address{" +
                   "id=" + id +
                   ", addressName='" + addressName + '\'' +
                   '}';
       }
   }
   ```



#### @Query注解

使用该注解可以自定义查询,但是需要使用JPQL语句

🌰

```java
@Query("select p from User p where p.id=1 or 1=1")
List<User> getUsers();
```

也可以使用原生SQL语句查询

```java
@Query(value = "select * from user",nativeQuery = true)
List<User> getUsersByNative();
```

带参数查询  参数使用 ？序号  表示  第一个参数就是?1 第二个参数就是?

```java
@Query("select p from User p where p.id=?1")
List<User> getUsersByID(Integer id);
```

模糊查询  （记住查询语句中写了%那么调用时参数就不需要加%，否则需要添加%）

```java
@Query("select p from User p where p.name like %?1%")
List<User> getUsersByName(String name);
```



#### @Modifying 

可以通过自定义JPQL完成update和delete操作，但是JPQL不支持insert操作

涉及到update和delete操作必须使用@modifying注解以通知SpringData这是一个update或delete操作

涉及到update和delete操作还必须添加事务，此时需要定义service层，在service层上添加事务

默认情况下Springdata的每个方法上都有事务，但是只有一个只读事务无法完成修改和删除操作

```java
@Modifying
@Query("update User p set p.name=:name where p.id =:id")
void updateUser(@Param("name")String name,@Param("id")Integer id);
```

service层

```java
@Service
public class PersonService {
    @Autowired
    PersonRepository personRepository;

    @Transactional  //一定要添加事务注解
    public void update(){
        personRepository.updateUser("xyz",1);
    }

}
```



#### CrudRepository

继承该接口之后可以有以下方法

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200903101944.png" alt="image-20200903101937834" style="zoom:50%;" />

涉及到update、save、还有delete的操作都需要定义到servcie中并使用事务进行修饰

🌰

```java
@Service
public class PersonService {
    @Autowired
    PersonRepository personRepository;

    @Transactional
    public void update(){
        personRepository.updateUser("xyz",1);
    }

    @Transactional
    public void save()
    {
        User user = new User();
        user.setId(3);
        user.setAge(10);
        user.setName("xyz");
        Address address = new Address();
        address.setId(2);
        user.setAddress(address);
        user.setAddressId(1);
        personRepository.save(user);
    }
}
```

save就是先检查数据库重视会否含有该数据，如果有则并且有不同的地方则更新该数据，否则添加，相当于merge操作



#### PagingAndSortingRepository

继承该方法后有以下方法

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200903104242.png" alt="image-20200903102426373" style="zoom:50%;" />

```java
@Test
void test6()
{
    int pageNo = 1;
    int pageSize = 1;
    //封装 PageRequest 实现类，其中包含需要分页的信息
    Sort sort = Sort.by(Sort.Direction.DESC,"id");
    Pageable pageable = PageRequest.of(pageNo,pageSize,sort);

    Page<User> page = repository.findAll(pageable);
    
    System.out.println(page.getTotalElements());
    System.out.println(page.getNumber());
    System.out.println(page.getTotalPages());
    System.out.println(page.getContent());
    System.out.println(page.getNumberOfElements());
}
```



#### JpaRepository 

PagingAndSortingRepository的子接口，在其基础上增加了一些方法

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200903104954.png" alt="image-20200903104711154" style="zoom:50%;" />



#### JpaSpecificationExecutor

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20200903111940.png" alt="image-20200903105905362" style="zoom:50%;" />

可以实现**带查询条件**的查询

以带查询条件的分页的为例

```java
@Test
void test7()
{
    int pageNo = 1;
    int pageSize = 1;
    //封装 PageRequest 实现类，其中包含需要分页的信息
    Sort sort = Sort.by(Sort.Direction.DESC,"id");
    Pageable pageable = PageRequest.of(pageNo,pageSize,sort);

    Specification<User> specification = new Specification<User>() {
        /**
         *
         * @param root 代表查询的实体类
         * @param criteriaQuery 从中可以查询到root对象，即告知 JPA criteria查询哪一个实体类，还可以添加c查询条件，还可以结合
         *                      EntityManager对象得到最终查询的typeQuery对象
         * @param criteriaBuilder 创建criteria对象工厂，可以从中获取的奥predicate对象
         * @return
         */
        @Override
        public Predicate toPredicate(Root<User> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
            Path path = root.get("id");//获得具体的查询对象的属性，也就是按照对象的哪个属性进行条件判断
            Predicate predicate = criteriaBuilder.gt(path,1);
            return predicate;
        }
    };

    Page<User> page = repository.findAll(specification,pageable);

    System.out.println(page.getTotalElements());
    System.out.println(page.getNumber());
    System.out.println(page.getTotalPages());
    System.out.println(page.getContent());
    System.out.println(page.getNumberOfElements());
}
```



#### 自定义Repository

##### 为某一个Repository添加自定义的Repository

1. 定义一个接口: 声明要添加的, 并自实现的方法
2. 提供该接口的实现类: 类名需在要声明的 Repository 后添加 Impl, 并实现方法
3. 声明 Repository 接口, 并继承 1) 声明的接口
4. 使用.
5. 注意: 默认情况下, Spring Data 会在 base-package 中查找 "接口名Impl" 作为实现类. 也可以通过　repository-impl-postfix　声明后缀.

🌰

接口

```java
public interface PersonDao {
    void test(Integer id);
}
```



实现类

```java
public class PersonRepositoryImpl implements PersonDao {
    @Autowired
    @PersistenceContext
    EntityManager manager;

    @Override
    public void test(Integer id) {
        User user = manager.find(User.class, id);
        System.out.println("method--->"+user);
    }
}
```



PersonRepository继承该接口即可调用自定义Repository的方法

```java
public interface PersonRepository extends Repository<User,Integer> , PersonDao{

}
```

测试

```java
@Test
void testPersonDao()
{
    repository.test(1);
}
```



<font color=red>**注意点**</font>：

​	命名必须符合声明使用的Repository接口名+Impl

​	在本例中也就是PersonRepository+Impl 即 PersonRepositoryImpl，否则会报错

​	自定义的接口名最好也与声明使用的Repository保持一致也就是此处的PersonDao，当然可以写成其他的名称



​	



##### 为所有的 Repository 都添加自实现的方法

- 步骤：

1. 声明一个接口, 在该接口中声明需要自定义的方法, 且该接口需要继承 Spring Data 的 Repository.
2. 提供 1) 所声明的接口的实现类. 且继承 SimpleJpaRepository, 并提供方法的实现
3. 定义 JpaRepositoryFactoryBean 的实现类, 使其生成 1) 定义的接口实现类的对象
4. 修改 <jpa:repositories /> 节点的 factory-class 属性指向 3) 的全类名
5. 注意: 全局的扩展实现类不要用 Imp 作为后缀名, 或为全局扩展接口添加 @NoRepositoryBean 注解告知 Spring Data: Spring Data 不把其作为 Repository