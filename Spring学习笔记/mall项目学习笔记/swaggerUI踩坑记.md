Swagger2 2.7.x踩坑（与Swagger2 2.9.0比较）

一开始使用的是

```xml
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger2</artifactId>
  <version>2.7.0</version>
</dependency>
<dependency>
  <groupId>io.springfox</groupId>
  <artifactId>springfox-swagger-ui</artifactId>
  <version>2.7.0</version>
</dependency>
```

在此版本中有一次的几个Bug

1. 每次填写一个ApiKey都会发送请求

   ![image-20201110170613111](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201110170619.png)

但是在2.9.0中被解决

2. 所有的ApiKey在任何路径中都被添加了，但是有些没有被主动添加的ApiKey在需要被认证的路径中没有被添加

   ```java
   private List<SecurityReference> defaultAuth() {
           List<SecurityReference> result = new ArrayList<>();
           AuthorizationScope authorizationScope = new AuthorizationScope("global", "accessEverything");
           AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
           authorizationScopes[0] = authorizationScope;
           //将ApiKey添加到每次需要进行认真的请求中  但是在swagger2.7.0 中除了需要认证的路径其他的任何路径都可以使用该APIKey
           result.add(new SecurityReference("Authorization", authorizationScopes));
           result.add(new SecurityReference("sessionId",authorizationScopes));
           return result;
       }
   ```

   

