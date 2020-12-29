1. **如果想从前端传递一个对象到后端，那么name属性必须和对象的属性名称一一对应，**

**这样的话前端会自动将其封装成一个对象传递给后端**

​		**即使前端只传递了一个对象的属性，后端如果说要接受对象的话，那么前端也是可以通过该属性将其自动封装成一个对象的**。

​		举例：

​		Person.class

```java
public class Person {

    //@Value("${name}")
    //@Email()
    private String name;
    //@Value(("${age}"))
    private Integer age;
    private List<Object> list;
    private Map<String,Object> map;
    private Department department;
```

​	Department.class

```java
public class Department {
    private String name;
    private Integer id;
```

那么我在前端页面中想要通过提交一个表单来添加一个人的对象的话，对于部门这个属性来说，如果是通过select组件来选择部门，那么select的name写成department.id的话，系统会通过该级联属性帮你自动封装成Department对象，其中自动封装后的department对象的id属性就是select选择的value，而name属性则为null。



2. Url路径中的信息后端如何获取

   `@PathVariable`

